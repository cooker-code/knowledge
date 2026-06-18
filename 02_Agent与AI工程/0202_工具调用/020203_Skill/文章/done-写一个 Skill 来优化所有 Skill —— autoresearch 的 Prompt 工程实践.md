> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 写一个 Skill 来优化所有 Skill —— autoresearch 的 Prompt 工程实践
author: 慧响
date:
url: https://mp.weixin.qq.com/s?__biz=MjM5NDAwNTQ0MA==&mid=2249485050&idx=1&sn=0eb6f0f381c45a4a23bcce0f22594e48&chksm=a4f27664538a3cce769530d95feeabfd2b40322f46ec19ad8f3d968d8168befb973e1033451a&mpshare=1&scene=24&srcid=0327w2ppPTFWyiOrJsWcx6vy&sharer_shareinfo=e5e64f977a3ee312ece6ead90372e219&sharer_shareinfo_first=e5e64f977a3ee312ece6ead90372e219#rd
---

点击上方“**慧响**” 可以订阅哦！

本文字数: **3240字**

阅读时间: **8分钟**

前几天刷公众号，看到有人用 Karpathy 的 autoresearch 方法论去优化 AI skill 的 prompt，pass rate 从 50% 拉到了 90%。

说实话，看到这个数字的时候我正在手动改 brain-search 这个 skill 的 prompt，已经改了四五版了，效果一直不稳定。手动调 prompt 这个事吧，很像小时候调收音机的天线——你觉得差不多了，手一松它又漂了。看到 autoresearch 这个思路之后我就停下来研究了一下，然后花了一个下午用 Claude Code 写了一个开源的 prompt 优化器。

## Karpathy 和 Lehmann 的 Repo

先说两个已有的项目。

Karpathy 的 autoresearch（53,600+ stars）解决的是 ML 训练的问题：让 AI agent 在一个死循环里不断改训练代码、跑实验、看 loss 降没降——降了保留，没降回滚。说白了就是把"人盯着指标改代码"这个苦力活自动化了。

Ole Lehmann 的 autoresearch-skill（555 stars）把这个模式搬到了 Claude Code 的 skill prompt 上。整个仓库就 2 个文件：`SKILL.md` 和 `eval-guide.md`。让 Claude Code 自己读 prompt、跑 eval、改 prompt、再跑，循环往复。

两者的共性其实很像生物进化——假设、变异、测试、保留或淘汰。只不过一个优化的是训练代码，一个优化的是 prompt 文本。

我想做的方向和 Lehmann 一样，但有几个地方不满足：

1. 我不想盯着。Lehmann 的方案要和 Claude Code 持续对话，人得在旁边看着。我想写完 eval 定义就放着跑。
2. 我日常用 MiniMax，偶尔用 OpenAI 和 Anthropic，不想绑死在某一家。
3. 得兼容 OpenClaw 的 skill 格式，同时也能给 Claude Code、Cursor、Cline 用。

## 怎么做的

最终有两个核心的设计决定。

**脚本驱动，不是对话驱动。** 整个优化器就是一个 Python 脚本 `autoresearch.py`，命令行参数传进去目标 skill 文件和 eval 定义，自己跑完整个循环。agent 只需要调一次脚本就行，不用来回聊天。

```
python autoresearch.py \
--target skills/brain-search/SKILL.md \
--evals eval.json \
--provider minimax \
  --max-experiments 10
```

**混合 eval。** eval 分两类——rule-based 和 LLM-as-judge。rule-based 用正则、关键词匹配、字数检查，结果确定、零成本。LLM eval 负责规则搞不定的语义判断，比如"输出是否准确反映了搜索结果"。

为什么要混合？我觉得这个很像考试出题——选择题（规则）改卷快、分数稳定，但有些东西你只能用主观题（LLM）来测。问题是主观题阅卷有波动，同一份答卷换个老师可能换个分数。所以能出选择题的地方就出选择题，把主观题留给真正需要的场景。

一个 eval.json 的例子，一条 rule 检查是否包含 URL，一条 LLM 判断输出是否有清晰结构：

```
{  "test_inputs": ["search for latest AI agent frameworks"],  "evals": [    {      "name": "has_source_url",      "type": "rule",      "rule": "regex",      "pattern": "https?://[^\\s]+"    },    {      "name": "clear_structure",      "type": "llm",      "question": "Is the output organized with headers, bullets, or numbered lists?",      "pass_description": "Output uses clear visual structure for easy scanning",      "fail_description": "Output is a wall of text without structure"    }  ]}
```

还有一个决定是零外部依赖——整个脚本只用 Python 标准库。HTTP 用 `urllib`，正则用 `re`，JSON 用 `json`。不需要装任何包，拿过来就能跑。不管是 OpenClaw 容器里的 Python 3.11 还是本地的 3.12，直接用。

## 三个 Agent 并行写代码

写代码的过程比较有意思。我没有线性地从头写到尾，而是让 Claude Code 同时起了 3 个 agent 并行干活：

* Agent 1 写 `SKILL.md`（skill 的元信息和使用说明）
* Agent 2 写 `autoresearch.py`（核心脚本，LLM provider、eval runner、实验循环）
* Agent 3 写 `eval-guide.md` 和 `examples/` 目录下的示例 eval 文件

三个 agent 之间有自然的接口约定（eval.json 的 schema、命令行参数格式），并行写问题不大。写完之后又让一个 code-reviewer agent 做了一轮审查。

这轮审查发现了 7 个 bug，印象比较深的几个：

* LLM provider 的自动检测逻辑在找不到任何 API key 时会返回 `None`，但后续代码没有 null check
* `word_count`

  规则没考虑中文字符，直接用空格 split 会把一整段中文算成 1 个 word
* eval.json 既支持 `values`（列表）又支持 `value`（字符串），但解析时只处理了列表

这些在第一次运行之前就被修掉了。从看到那篇公众号文章到 `git push`，整个过程不到 2 小时。

## 拿 brain-search 跑了一下：37.5% -> 54.2%

光写出来不行，得拿真实 skill 验证。

我选了自己知识系统里的 brain-search。它的功能是接收搜索请求，调 Brave Search API，返回结构化的搜索结果摘要。问题是输出一直不太稳定——有时候有来源链接有时候没有，结构有时候清晰有时候一坨，中文查询偶尔还会混入英文回复。

写了 4 条 eval，中英文各半的 test input，跑起来。

**Baseline（实验 0）：37.5% pass rate。**

这个数字比我预期的低…… 仔细看 eval 细节，主要是来源 URL 的引用格式不一致（有时候有、有时候没有），以及 LLM judge 认为结构不够清晰。

**实验 1-2：分数上升。** 优化器分析了 baseline 的失败项，发现两个主要问题——缺少来源 URL 引用和输出缺乏结构。它自动在 SKILL.md 的 Procedure 部分添加了两条指令：一是要求每条搜索结果必须附带原始 URL（不是笼统的"注明来源"，而是给了具体的引用格式模板），二是要求用 markdown 列表组织输出。这两个改动被保留了，pass rate 提升到了 54.2%。

**实验 3-5：全部回滚。** 这是 autoresearch 的经典行为——容易的改进先被找到，后面的变异越来越"大胆"（比如重写整个 Procedure），效果反而变差。实验 3 试图加入"必须用中文回答中文查询"的硬性规则，结果英文查询的质量反而下降了。实验 4 和 5 做了更激进的结构性改动，分数都没超过 54.2%，全部被丢弃。

37.5% 到 54.2% 不算惊艳，但有意思的是那两个被保留的改动（URL 引用格式 + 结构化输出模板）——我之前手动调的时候一直在纠结"语气"和"详细程度"这些模糊的东西，而优化器直接瞄准了 eval 里最具体的失败项。人调 prompt 容易陷入"感觉差不多了"的陷阱，机器不会。

## MiniMax 的 thinking 块，折腾了好一会儿

最折腾的一个 bug 出在 MiniMax M2.7 的 API 上。

MiniMax 支持 extended thinking——在给出最终答案之前先"想"一段。API 返回的 `content` 数组里会有 `{"type": "thinking", "thinking": "..."}` 和 `{"type": "text", "text": "..."}` 两种 block。

问题在于 thinking 块会消耗 `max_tokens` 预算。我最初给 LLM judge 调用设的 `max_tokens=16`（判断结果就是一个 YES 或 NO 嘛），结果 thinking 先把 16 个 token 全用完了，text block 里什么都没有。脚本拿到的返回值是空字符串，既不是 YES 也不是 NO，所有 LLM eval 全部 FAIL。

我当时看着满屏的 FAIL 还以为是 prompt 写得太烂了……

```
# 修复前：max_tokens=16，thinking 占满预算
resp = self.provider.call(system="", user=prompt,
                          temperature=0.0, max_tokens=16)

# 修复后：max_tokens=256，并且增加 thinking 内容的 fallback
resp = self.provider.call(system="", user=prompt,
                          temperature=0.0, max_tokens=256)
```

改了两个地方：一是 max\_tokens 从 16 提到 256（thinking 和 answer 加起来肯定够），二是在 `_extract_text` 方法里加了 fallback 逻辑——如果没有 text block，就从 thinking block 里找 YES/NO。后者是保险措施，提高 max\_tokens 之后实际上不会触发了。

这个 bug 在 OpenAI 和 Anthropic 上都不会出现，因为它们要么不返回 thinking block，要么 thinking 不占 max\_tokens 预算。MiniMax 用的是 Anthropic 兼容协议但行为有区别，不看文档很难发现。

## 做完之后的一些想法

我觉得 autoresearch 最有意思的地方，不是那个"改了再跑"的循环本身——那个循环很朴素。真正有价值的是它逼你去定义"好"的标准。

写 eval.json 的时候你会发现，大部分时候我们调 prompt 是靠感觉的。看着输出"差不多"就行了。但是一旦要把"差不多"翻译成 pass/fail 的规则，就必须想清楚到底什么叫好、什么叫不好。这个过程有点像……你让一个厨师做菜他可能凭感觉就很好吃，但你让他写成食谱教别人，他就得把"适量"翻译成"3 克"。翻译本身就是一种理解的深化。

另外，小步改进比大刀阔斧靠谱。实验 1-2 的小改动生效了，实验 3-5 的大改动全部回滚。这和写代码一样——小 commit、快反馈、及时回滚。autoresearch 只是把这个老道理搬到了 prompt 优化上。

37.5% 到 54.2%，说实话不算高。但我倒是觉得这恰好说明了一件事：eval 写得好不好，直接决定了优化的天花板。我只写了 4 条比较粗糙的 eval。如果能写出更精细的（比如区分"有链接但链接不相关"和"没有链接"），pass rate 应该还能往上走。工具到位了，剩下的是你对"好输出"理解到什么程度。

代码在 GitHub（zning1994/openclaw-autoresearch）和 ClawHub（clawhub.ai/zning1994/openclaw-autoresearch）上，`npx skills add zning1994/openclaw-autoresearch` 或者 `openclaw skills install openclaw-autoresearch` 就能装。不限于 OpenClaw，任何用 SKILL.md 格式管 prompt 的工具都能用——Claude Code、Cursor、Cline 都行。

说到底，prompt engineering 这件事到现在还是挺手艺活的。但至少现在可以让机器帮你做一部分"试错"了，你把精力放在定义"什么是好的"上面就行。我觉得这可能是接下来 AI 工具链演化的一个方向——不是让 AI 替你做决定，而是让 AI 替你做实验，你来定义实验的成功标准。

想想其实挺有意思的，我们用 AI 来优化 AI 的指令，然后用 AI 来评判优化的结果……这个套娃到底能套几层，我也不知道。不过至少目前这一层，是 work 的。

慧响精英荟开张啦~

扫码加入，更有限时免费

进入慧响星球的福利哦~

（如二维码过期，请寻找最新文章底部

或后台留言）

雁过留名 请记得点击

赞、在看

哦~