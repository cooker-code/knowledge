---
title: Anthropic放出Claude 4.7提示词「规格书」！20万人围观：你升级了模型，Prompt还停在2023年
author: 神经界碑
date: 干饭每日干饭每日
url: https://mp.weixin.qq.com/s?__biz=Mzk0MTIxODg2Mg==&mid=2247492218&idx=1&sn=f6513e72088a57f7b9737d3cf64a2881&chksm=c3a521b49d4052fd92a124c9beaa1baa3c76582aa73fab96ef3810ce90d99808df18b4b3c5c8&mpshare=1&scene=24&srcid=0514uGKTSunhyK9GIuMd3mAQ&sharer_shareinfo=b6d98243ee126d8f575dd6a408b9c233&sharer_shareinfo_first=b6d98243ee126d8f575dd6a408b9c233#rd
---

导读  
Anthropic官方更新了一份系统级提示词工程文档，专门针对Claude Opus 4.7的行为变化给出规范。一位拥有59万订阅者的AI博主Ruben Hassid把文档浓缩成三条规则发到X上，近千人点赞、20万人围观。核心信息只有一个：**模型变了，你的提示词必须跟着变。**

## 一条帖子，把提示词圈炸了锅

5月10日，AI领域博主Ruben Hassid在X上发了一条帖子，开头就是：

> "Anthropic just dropped a 31-page guide."

「Anthropic刚放出了一份31页的指南。」

他花了一个周末读完Anthropic关于Claude 4.7的提示词文档，把核心变化压缩成了三条规则——scope rule、negative rule、length rule。

▲ Ruben Hassid 在 X 上分享 Claude 4.7 提示词规则，获近千赞、20万次浏览

帖子下面的讨论迅速升温。开发者们集体意识到一个问题：**Claude变了，但自己的Prompt一个字都没改。**

## 规则一：Scope——「帮我review一下」等于什么都没说

Ruben把第一条规则叫做scope rule，原话是：

> "Review this contract gets you nothing. Name every output to Claude."

「'帮我审查这份合同'等于什么都没给。你得把每一项输出都写清楚。」

这条在社区里共鸣最强。用户@ls\_brd回复说：

> "Name every output feels like basic negotiation 101 honestly."

「把输出项写清楚——这跟商务谈判的基本功一样。」

Anthropic官方文档里对这一点的表述更系统。文档明确指出，Claude Opus 4.7**对提示词的字面解释更严格**：

> "Claude Opus 4.7 interprets prompts more literally and explicitly than Claude Opus 4.6."

「Claude Opus 4.7 对提示词的字面解读比 4.6 更严格、更明确。」

什么意思？以前你写「帮我review一下这个PR」，Claude会自己脑补你大概想要哪些维度的反馈。现在4.7更倾向于：**你写了什么，它就做什么。你没写的，它默认不做。**

用户@mylifcc已经开始用新的写法了：

> "Flag the 3 highest-risk hunks, ≤2 sentences each, no fixes."

「标出风险最高的3个代码块，每条限两句以内，不要给修复建议。」

从「帮我看看」到把输出格式、数量、边界全部写死——这就是scope rule在开发者手里的真实落地。

## 规则二：Negative——「不要用术语」反而让模型盯着术语看

第二条规则最反直觉。Ruben说：

> "Don't use jargon doesn't work. Use positive instructions."

「'不要用术语'这种写法不管用。用正向指令。」

用户@imhabibx把原因解释得最到位：

> "You're literally telling the model to think about jargon before it decides not to use it."

「你实际上是在强迫模型先想'术语'这个概念，然后再决定不用它。」

Ruben本人的回复更直接：

> "You're making it think about the thing you're trying to avoid. Show it what you want instead."

「你在逼它思考你想避开的东西。直接给它看你想要的样子。」

Anthropic官方文档在这一点上给出了明确的技术建议：

> "Positive examples tend to be more effective than negative examples."

「正向示例通常比负向示例更有效。」

具体怎么改？Ruben在回复里给了一个替换写法：

与其写「不要用术语、不要用营销腔、不要写废话」，不如直接给目标——

**「用一个16岁学生能大声读出来的plain English来写。」**

用户@rugbist\_总结了一句被多人转发的话：

> "Tell it what to do, not what not to do."

「告诉它该做什么，别告诉它不该做什么。」

但这条规则也有边界。用户@XNickDaN补充了一个关键的反方视角：

> Google的官方DESIGN.md规范里就有标准的「Do's and Don'ts」章节。

关键不在于永远不能写「don't」。**当模型越来越按字面执行时，单独的禁令不如「禁令+正向规格」组合来得稳定。**在安全边界和guardrails场景下，明确的「不要做X」仍然有价值，但最好搭配一个正向目标一起给。

## 规则三：Length——「帮我总结一下」换来8段长文

第三条规则直击日常痛点。Ruben说：

> "Summarize this returns 8 paragraphs. Always define the cap."

「'帮我总结一下'会换来8段长文。永远要定义上限。」

用户@shellsgn承认自己踩过这个坑：

> "Never thought about defining length explicitly. I assumed Claude would figure it out."

「我从没想过要明确定义长度。我以为Claude自己会判断。」

Anthropic官方文档对这个变化有更精确的解释：

> "Claude Opus 4.7 calibrates response length to how complex it judges the task to be."

「Claude Opus 4.7 会根据它判断的任务复杂度来校准回答长度。」

▲ Anthropic 官方文档详细说明了 Claude 4.7 在回答长度、effort level、工具调用等方面的行为变化

也就是说，4.7不再像旧模型那样按固定的冗长度输出。它会自己判断任务有多复杂，然后给出它认为「匹配」的长度。如果你不设上限，它可能觉得你的任务需要8段话才能说清——然后就真给你8段。

修复方法很明确：**把格式和上限写进提示词。**

比如「用3个要点总结，每条不超过15个词」，或者「输出一个表格，列标题是风险项/严重等级/建议操作」。

## Anthropic官方文档到底写了什么？

Ruben的帖子是入口，但真正的重头戏在Anthropic的官方文档《Prompting best practices》里。

这份文档开头就声明了自己的定位：

> "This is the single reference for prompt engineering with Claude's latest models, including Claude Opus 4.7."

「这是面向Claude最新模型（包括Claude Opus 4.7）的唯一提示词工程参考文档。」

▲ Anthropic 官方文档明确声明，这是 Claude 最新模型的「单一参考」

文档覆盖的范围远不止Ruben总结的三条规则。它系统地展开了Claude 4.7的几个关键行为变化：

**1. 更少主动调用工具，更多先推理。**Claude Opus 4.7比4.6更倾向于先思考，再决定是否需要调用外部工具。

**2. effort level更敏感。**Anthropic建议coding和agentic场景从xhigh effort起步，多数intelligence-sensitive任务至少用high。在max/xhigh effort下，建议给到64k tokens的输出预算。

**3. 写作风格更直接。**4.7默认减少了validation-forward phrasing（先肯定你再说正事）和emoji使用。

**4. 子agent生成更保守。**除非你在提示词里明确写了触发条件，4.7默认不会自己派生子agent。

最关键的一点在更上层。Anthropic在另一份方法论文档《Prompt engineering overview》里，给出了整个提示词工程的框架：

* 先定义清晰的**成功标准（success criteria）**
* 建立能跑的**评估方式（empirical test）**
* 拿到一个**可以迭代的初版提示词（first draft prompt）**

这套流程跟写代码的思路一致：先定义什么算对，再写测试，再改实现，最后迭代上线。

## 开发者社区在讨论什么？

帖子下面的讨论有两个明显的方向。

**第一个方向是集体自省。**大量开发者承认，自己从来没把提示词当成需要工程化维护的东西。用户@pow2chain说了一句很扎心的话：

> "People will read this, nod, and keep prompting the exact same way because it feels faster than writing spec."

「人们会看完这篇、点点头，然后继续用老方式写Prompt——因为写规格书太慢了。」

用户@modisulak总结得更狠：

> "Half of prompt engineering is just saying what you actually want."

「提示词工程，一半的工作就是把你到底想要什么说出来。」

Ruben回了一句：

> "And the other half is knowing what you actually want."

「另一半是搞清楚你到底想要什么。」

**第二个方向是即时行动。**已经有用户开始拿具体业务场景练手。@mylifcc的PR review模板、@thechaicoder分享的客户项目prompt结构、@iwhaleocean关于约束条件和输出格式的讨论——这些不再是「聊聊感想」，而是能直接复用的生产模板。

▲ Ruben 在 Substack 上发布了更详细的《Prompt 4.7》教程，对照新旧写法逐项拆解

用户@alansalomonx的评价可以当这次讨论的注脚：

> "Prompting is becoming less about clever wording and more about clear specification."

「提示词工程正在从'会说漂亮话'转向'写清楚规格'。」

## 模型升级之后，先暴露出来的是Prompt债务

回看Anthropic这份文档和Ruben引发的讨论，最值得注意的变化发生在更底层。

以前写提示词，靠的是「经验」和「手感」——多试几次，看哪句话效果更好，然后把那句话存下来反复用。这种方式在模型还不够literal的时候能跑通，因为模型会帮你脑补你没写出来的意图。

Claude 4.7把这个安全网抽掉了。

**它更强，但也更严格。**你写了scope它就按scope做，你没写它就不做。你给了正向规格它就执行，你只给了「不要」它可能反而被干扰。你没定义长度它就按自己的理解来。

这意味着什么？**模型升级之后，模型的上限还没碰到，团队积攒的Prompt债务先原形毕露了。**那些以前靠模型脑补也能过关的模糊指令，在更literal的模型面前会原形毕露。

Anthropic的文档把这件事讲到了更深的一层：提示词不只是「一句指令」。它在同时调控模型的行为、成本、延迟、工具调用策略和输出风格。当你写下一个提示词，你实际上是在写一份**接口契约（interface contract）**。

对于还在用「帮我看看这个」「总结一下」「不要废话」这类提示词的人来说，升级到Claude 4.7之后，第一件要做的事可能不是学新功能——

**而是回去重写你的每一条Prompt。**

---

— END —

— END —