> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: Learning Loop：Hermes 沉淀行为SKILL的法宝
author: AI步步通
date:
url: https://mp.weixin.qq.com/s?__biz=MzY4NTE4OTYzNg==&mid=2247483836&idx=1&sn=644bba6cb4b8884e02f08906812fbbed&chksm=f2db15da2c48deceba2370b439d566826646f3408e7bc5f693295e12dd734572285b88c0c562&mpshare=1&scene=24&srcid=0423ALits6Rx1dvnsVRVwO3U&sharer_shareinfo=671a918b4785e976c5203ef92cf91a46&sharer_shareinfo_first=671a918b4785e976c5203ef92cf91a46#rd
---

很多持续运行的 Agent 都会遇到同一种场景：同一个用户每周五都要拉一遍五家竞品的新动态。第一次，Agent 还能边查边试；第三次，用户已经把截图顺序、分析模板、归档目录都校正过了。这个时候，真正有价值的，是这套流程能不能变成下次开工就能直接调用的方法。

Hermes Agent中的Learning Loop 处理的正是这件事。它负责把一次复杂任务中摸索出来的有效做法，从“本轮对话里的临场发挥”推进成“以后还能直接调用的程序性能力”。Hermes 的经验能不能真正累积，Agent 会不会随着使用次数增长而越来越顺手，关键就看这条链路能不能成立。

Hermes 为这件事补上了完整工程路径：偏好进入 `MEMORY.md`，历史对话留在 `state.db`，成功路径被整理成 `SKILL.md`，下一轮任务启动时再重新进入提示词组装链路。沿着这条任务流拆开来看，Hermes 的“越用越聪明”就会落到具体源码文件和运行路径上。

Learning Loop 给 Hermes 带来的增量，是把复杂任务的成功做法保存成程序性记忆。结果会过期，方法会复用，下一轮任务因此可以直接复用上一次沉淀的方法。

## 一、Learning Loop 保存的是可复用方法

在 Hermes 的整个长期记忆体系里，至少有三种完全不同的东西。第一种是长期事实，比如项目约定、环境信息、用户偏好，这部分进入 `MEMORY.md` 和 `USER.md`；第二种是历史过程，比如上个月某次排障到底聊过什么，这部分进入 SQLite 和全文检索；第三种是做事路径，比如“竞品研究按什么顺序抓取、截图、归档、总结、校验”，这部分才属于 skill。

Learning Loop 的任务，就是把第三种东西稳定留下来。它保存的是触发条件、执行步骤、容易踩坑的位置、最后如何验证成功。整条链路可以压成四步：观察、提炼、复用、修补。源码层的实现也基本沿着这四步展开，只是每一步分散在不同文件里。

Hermes 三类长期沉淀的分工

| 沉淀对象 | 承载位置 | 解决的问题 |
| --- | --- | --- |
| 稳定事实与偏好 | MEMORY.md / USER.md | 下次开局就默认带上哪些事实和约束 |
| 历史对话与轨迹 | state.db + FTS5 + session\_search | 需要追溯旧过程时去哪里找证据 |
| 可复用流程与协议 | SKILL.md + references / templates / scripts | 以后遇到同类任务时如何直接复用做法 |

## 二、Hermes 为什么会决定生成一个 Skill

假设用户连续几周都在让 Hermes 做同一类工作：每周五整理五家竞品的新动态。真正的任务是一条多步骤流水线：打开官网和社媒、截取关键页面、抽取变化点、保存到固定目录、按照统一模板生成报告、最后检查截图和结论是否一致。

第一次做这件事时，Hermes 只能边跑边摸索。第二次做时，用户会补充修正，比如截图顺序要固定、同类产品最多分析三家、报告一定要带时间戳。等到第三次、第四次，这件事已经具备了典型特征：步骤多、依赖工具多、反复发生、修正过多次、而且一旦跑顺就高度可复用。Learning Loop 的价值，正好在这类任务上释放出来。

Hermes 把“反复跑通的复杂任务可以进入提炼阶段”放进了这套设计里。任务一旦积累到足够高的复用价值，就会开始朝 skill 的方向收束。系统提示也在推动这件事发生：当任务包含五次以上工具调用、修过棘手错误、或者发现了一条非平凡工作流时，Agent 应该把做法保存成 skill。

这里需要把两件事分开看。单次会话里的复杂度触发，已经有明确实现线索：主循环会定期回看最近的工具调用，`prompt_builder.py` 里也明确列着 `5+ tool calls`、`tricky error`、`non-trivial workflow` 这些条件。跨会话的“第三次重复”则还没有展开到一套可直接指出的重复计数器或后台聚类机制，至少在当前能看到的实现边界里，这一层还没有被写成一条同样具体的工程路径。

Learning Loop 在复杂任务上的演进路径同类任务重复出现时，经验会从临场操作变成可调用 skill第一次任务浏览器、搜索、截图、汇总边执行边试错成功路径还停留在当前会话里第二到第三次任务用户修正顺序、模板、边界Agent 识别出可复用模式开始值得提炼成程序性记忆后续同类任务加载 skill 直接开工少走弯路，步骤更短新边界出现时继续patch skillLearning Loop 把重复成功的复杂任务压缩成一份可重复调用的操作协议

## 三、AIAgent 在主循环里先把任务跑完，再决定哪些经验值得沉淀

从架构图往下看，Hermes 的一切都先经过 `run_agent.py` 里的 `AIAgent` 主循环。主要逻辑是：先组装 system prompt 和工具 schema，再向模型发起调用，拿到 tool calls 后执行工具，把结果回填进消息历史，再继续下一轮。Learning Loop 的第一步“观察”，就发生在这条循环里。

关键点在于它能观察到一整条轨迹：用了哪些工具、用户补过哪些纠正、哪一步失败过、最后哪条路径跑通。系统还会定期进入自评检查点，回看最近若干次工具调用，判断哪些东西应该写入 memory，哪些东西已经值得提炼成 skill。也正因为如此，“学习”才不是任务结束后的附加动作，而是运行时循环中的固定判断动作。

这一步非常重要，因为 skill 建立在真实执行、真实修正、真实验证之上。前面的轨迹越完整，后面的提炼就越有工程价值。Learning Loop 先观察轨迹，再抽象协议，所以生成出的 skill 更接近“跑通过的流程”和“验证过的做法”。

这套机制里更容易触发沉淀的信号

1. 任务已经包含较多工具调用，属于复杂工作流。

2. 中途经历过错误、回退、修正，最终找到可行路径。

3. 同类任务反复出现，流程可复用价值高。

4. 成功路径已经能被写成触发条件、步骤、坑点和验证标准。

## 四、skill\_manage 是真正把经验写成 Skill 的核心工具

Learning Loop 真正落盘的地方，在 `tools/skill_manager_tool.py`。这个文件把 skill 定义成 agent 的 procedural memory，也就是程序性记忆，专门保存“如何做某类任务”。到了这里，前面跑通的竞品研究流程才真正有机会从一次性的任务结果，变成以后还能反复调用的方法。

skill\_manage 负责两类事情。第一类是生成和更新能力包：agent 可以在运行时自己写一份新的 `SKILL.md`，必要时再补 `references`、`templates`、`scripts` 这些 supporting files。第二类是维护这份能力包的生命周期。源码里给了 `create`、`patch`、`edit`、`delete`、`write_file`、`remove_file` 六种动作，其中 `patch` 还用了 fuzzy match，方便在格式轻微变化时继续更新。

Hermes 同时给这条路径加了三层护栏。写文件时走原子写入，避免中途中断写坏内容；落盘前做 frontmatter、字符数、目录结构等硬校验，保证 skill 还能被系统加载；最后再做安全扫描，必要时直接回滚。Learning Loop 交付的因此是一条受控的能力生成通道，一份 skill 也因此具备了进入系统主链路的工程质量。

从工程判断上看，Hermes 对“运行时工具创造”给出了一条更稳的落地方式：运行时生成 skill。它不改底层工具 schema，不污染核心注册表，却能在任务结束后立刻长出一个新能力入口。这种能力入口足够轻，复用成本低，也更适合被持续修补。

SKILL.md 的公共骨架，大致长这样

```
---
name: competitor-analysis-workflow
description: Weekly competitor tracking workflow with screenshots and timestamped summaries
version: 1.0.0
platforms: [macos, linux]
metadata:
  hermes:
    tags: [research, competitor, weekly]
    requires_toolsets: [web, terminal]
---
# Competitor Analysis Workflow

## When to Use
When the user asks for repeated competitor monitoring with a stable output format.

## Procedure
1. Capture key pages with browser and save screenshots.
2. Extract visible changes and compare with the prior baseline.
3. Write findings into the fixed report template.
4. Save outputs to the agreed directory with a timestamp.

## Pitfalls
- Do not analyze too many competitors in one pass.
- Keep screenshot order stable across weeks.
- Preserve raw evidence before summarizing.

## Verification
Confirm screenshots exist, output files are written, and the final report matches the requested format.
```

这段示例抓的是这类 skill 的公共骨架。读者真正要抓住的是：Hermes 最后交付的，是一份带 YAML frontmatter 的可执行工作说明书，里面直接写清什么时候用、怎么做、哪里容易出错、怎样算完成。

skill\_manage 落盘后的目录形态

`~/.hermes/skills/competitor-analysis-workflow/SKILL.md`

`~/.hermes/skills/competitor-analysis-workflow/references/...`

`~/.hermes/skills/competitor-analysis-workflow/templates/...`

`~/.hermes/skills/competitor-analysis-workflow/scripts/...`

skill 从一开始就被设计成可继续扩展的能力包。

## 五、prompt\_builder 还要把 Skill 重新接回主执行链路

写进磁盘以后，skill 还要重新回到主执行面，Learning Loop 的闭环才算完整。下一轮遇到同类任务时，Hermes 会先看到这份能力索引，再决定要不要把全文调进来。这个动作发生在 `agent/prompt_builder.py`：它位于冻结 memory snapshot 之后，也就是 system prompt 里专门留了一层给 skills index。

这里的顺序可以概括成三步：先筛掉不相关的 skill，再给模型看技能目录，真正相关时才加载全文。`prompt_builder.py` 会先扫描所有 `SKILL.md`，抽出名字、描述、分类、平台约束和条件约束，生成一份 snapshot，并做两层缓存：进程内 LRU 和磁盘上的 `.skills_prompt_snapshot.json`。随后它再根据 frontmatter 里的 `requires_toolsets`、`requires_tools`、`fallback_for_toolsets`、`fallback_for_tools` 做一轮确定性过滤，把不该出现的 skill 先挡在索引之外。

过滤完成后，Hermes 再把轻量索引放进 system prompt，并明确要求模型在相关时调用 `skill_view(name)`。至少从现在这条实现路径来看，Hermes 走的是“元数据索引 + 条件过滤 + 模型决策加载全文”这条路；文章里没有继续把它上升成向量召回器或单独的 skill router tool。这条实现更像一条轻量技能路由路径，而不是一套 embedding 检索系统。

此外，`prompt_builder.py` 里不仅有 skills index，还有一段明确的系统级 guidance：复杂任务完成后要考虑保存 skill，使用 skill 时发现过期或缺步骤要立即 patch。Learning Loop 因此已经进入 Agent 的默认行为规范。

再往旁边看，`agent/skill_commands.py` 会扫描 `~/.hermes/skills/`，把 skill 暴露成 `/skill-name` 形式的命令。于是同一份 skill 既能被模型自动加载，也能被用户显式调用。经验沉淀、能力发现、运行时调用，这三步就连成了一条完整链路。

## 六、Patch 的真实护栏也要讲清楚

Hermes 对 skill 更新设置了明确限制。现在能看到的护栏至少有四层。第一层是原子写入：skill 文件先写到同目录临时文件，再用 `os.replace()` 替换目标文件，防止中途留下半个坏文件。第二层是结构校验：patch 完 `SKILL.md` 之后，frontmatter 和正文结构还要重新过校验。第三层是安全扫描：一旦命中危险模式，工具会直接恢复原始内容。第四层是 prompt cache 失效：更新成功以后，skills system prompt cache 会被清掉，下一轮重新生成索引。

目前这套机制已经能防住半写坏文件、结构损坏、危险内容落盘。更高一层的语义保证，比如 skill 改完以后是否还完整保留了原工作流，当前这套实现里还没有展开成 dry-run 验证器或 skill 级版本树。

生产环境里还有一层更宽的保护机制。Hermes 提供全局的 Filesystem Checkpoints，默认会在文件改写前做工作目录快照，用户可以通过 `/rollback` 恢复。但这层保护服务的是整个工作目录，它提供的是通用文件回退能力；单个 skill 的独立语义版本树，当前还没有展开成一条单独能力线。

## 七、为什么 Hermes 要把 memory、session\_search、skill 分成三条线

Hermes 把三种信息放进了三种不同容器。memory 负责常驻事实，适合保存偏好、环境和长期约定；session\_search 负责回忆旧过程，适合追溯某次任务当时到底怎样跑通；skill 负责复用方法，适合保存以后还会重复发生的工作流。

Learning Loop 位于第三层。它处理的是那些已经足够稳定、足够具体、以后还会反复出现的复杂任务。这样一来，竞品研究这种周周重复的流程就不必每次都从历史对话里重新拼装，系统路径本身已经先把“事实、过程、方法”分好了家。

## 八、对做 Agent 的团队来说，Learning Loop 最具有借鉴意义的有四件事

同一类复杂任务一旦需要反复教，团队就在重复消耗；一旦能沉淀成 skill，团队开始累计方法资产。这里最具有借鉴意义的地方有四件事。第一，把程序性记忆单独做成一种对象，不要把事实、历史、方法混在一个存储层里。第二，能力沉淀优先选择轻量能力包，让系统先学会长 skill，再考虑更重的内建工具路径。第三，生成后的能力必须立即进入下一轮 prompt 体系，经验才能真正回到主执行面。第四，生成和修补都要带工程护栏，原子写、结构校验、安全扫描、缓存刷新，一个都不能少。

回到前面的竞品研究任务来看：同一份周报如果每周都要重新教一遍截图顺序、分析模板和归档方式，团队得到的只是一次次重复执行；这些做法一旦沉淀成 skill，团队累积的就是长期方法资产。Learning Loop 的价值，最终就落在这里。

Hermes Learning Loop 综合流程图AIAgent 主循环执行复杂任务模型调用工具、接收结果、继续推理，形成一整条可观察任务轨迹任务轨迹评估复杂任务、重复任务、错误修复系统 guidance 推动 skill 沉淀observe / distill 发生在这里skill\_manage 落盘create / patch / edit / write\_file原子写、结构校验、安全扫描写入 ~/.hermes/skills/MEMORY.md稳定事实偏好、环境、约定下次开局直接带上state.db + FTS5历史对话session\_search 按需回忆保留旧证据链skills indexprompt\_builder 注入第七层skills\_list 轻量索引skill\_view 按需加载全文下一轮同类任务自动匹配 skill 或通过 /skill-name 调起，复用后继续 patch，形成 refine 闭环

Hermes 的 Learning Loop 已经把“持续学习”做成了一条完整工程链路：主循环里观察轨迹，skill\_manage 把流程写成能力包，prompt\_builder 再把能力包送回下一轮 system prompt。Agent 因而开始沉淀工作方法，团队也能把成功路径累计成长期资产。对于任何想做 Stateful Agent 的团队来说，这一层都值得认真研究。