> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: Hermes Agent 爱马仕的三级 memory，到底在记什么？
author: One的AI工具箱
date: One 掌柜One 掌柜
url: https://mp.weixin.qq.com/s?__biz=MzI0Mjc0MzA0Mg==&mid=2247486133&idx=1&sn=23f25ba83d20dccdb93e899407b3a3e7&chksm=e80e3d4d54457890915eddb18f3f6339527b561a68a0ec00c313f86edb91152aca8c9d529109&mpshare=1&scene=24&srcid=0520FatvpLVEyzuclPtrSrrB&sharer_shareinfo=20bfa9c22e44a485b08ddb3e033a99a1&sharer_shareinfo_first=20bfa9c22e44a485b08ddb3e033a99a1#rd
---

大家好，我是 One。

我最近看 Hermes，有个感觉越来越强：

很多人一上来先看错地方了。

先看它支持多少模型。 先看它接了多少平台。 先看它能调多少工具。

这些当然都重要。

但如果你真开始把 Agent 往长期工作流里接，你最后最在意的，往往不是“它这轮够不够聪明”。

而是另一件事：

它下轮还记不记得。

这也是为什么我会觉得，Hermes 最近这张图很值得看。

因为它没有在讲一个很虚的“长期记忆故事”，而是直接把 memory 结构拆开给你看了。

先放原图。

这张图最值得看的地方，不是“它也有 memory”。

而是：

Hermes 不是只有一个笼统的 memory。 它把 memory 分成了三层，再配一个周期性的 nudge 机制。

如果你想快速理解 Hermes，这就是最该先看的地方。

## 第一层：两份很小、但每轮都会带上的 Markdown memory

图里第一层写得很直接：

* Fast
* Two tiny Markdown files
* Frozen mid-session
* Always in system prompt

对应的是两份文件：

* `MEMORY.md`
* `USER.md`

这里最关键的点不是“它会记”，而是它只记最值得常驻的那一小部分东西。

图里能看到，`MEMORY.md` 大约是 2200 字符上限，`USER.md` 大约是 1375 字符上限。

也就是说，这一层不是拿来堆资料的。 它更像一个很小、很贵、很克制的常驻 memory 层。

里面放的，是那些最稳定、最值得每轮都带着的信息，比如：

* 项目约定
* 工具 quirks
* lessons learned
* 用户身份
* 沟通风格
* skill level
* 明确的偏好和禁忌

图里还有一个很关键的设计：

这一层在当前 session 里是 frozen 的。

原图明确写了，之所以 frozen mid-session，是为了 preserve the LLM prefix cache。

换句话说，本轮中就算又写入了新的 memory，也不会立刻把 system prompt 前缀打乱，而是等到下一轮再注入。

这个点很工程，也很重要。

因为它说明 Hermes 不是单纯在说“我有长期记忆”，而是在平衡几件事：

* 记忆写入
* prompt 稳定性
* prefix cache 成本

另外，图里还明确写了一个 consolidation 机制。

当 `MEMORY.md` 到 80% 左右时，会触发整理。 agent 会 merge 或 drop 一些内容，把它重新压回高密度状态。

所以第一层不是一个会无限膨胀的记忆池。 它从一开始就是按“小而稳”的思路设计的。

## 第二层：SQLite + FTS5 的历史检索层

第二层图里写得也很清楚：

* Unlimited capacity
* SQLite + FTS5
* Full-text search
* On-demand tool call

这个层解决的，不是“永远带着什么”。

它解决的是：

过去聊过的大量历史，怎么在需要的时候再找回来。

图里甚至把 retrieval pipeline 都画出来了：

1. agent 调用 `session_search(query)`
2. FTS 对历史结果排序
3. 速度大概是 10ms 检索 10,000+ docs
4. 再由 Gemini Flash 总结 top hits
5. 把 concise summary 返回当前上下文

这个设计背后的意思很简单：

不是所有历史都值得常驻。 但真正需要的时候，历史得能低成本召回。

这和第一层是完全不同的逻辑。

第一层负责“每轮都带什么”。 第二层负责“过去聊过什么，临时找回来”。

这也是为什么我会觉得，这张图比单纯讲“长期记忆”更有价值。

因为它不是模糊地说“系统会记住你”，而是把检索层单独拆出来了。

## 第三层：可插拔的 semantic memory provider

第三层是这张图里最容易被误读的地方。

图里明确写的是：

* Semantic
* External providers
* Pluggable
* Opt-in

也就是说，第三层不是简单的“历史记录”，也不是这张图里所说的 skill 层。

它更像一个外部语义 memory provider 层。

图里还能看到几个很关键的信息：

* 支持多个 provider
* 示例里是 `8 supported, 1 active`
* 会做：

+ PREFETCH before turn
+ SYNC after response
+ EXTRACT at session end

图里列了 Honcho 之类的 provider，也列了其他不同风格的实现。

重点不在于你记住每个名字。 重点在于这个层的角色很清楚：

它是可插拔的 semantic memory 层。

而且图里还强调了一句：

就算切换 provider，Tier 1 和 Tier 2 也还在。

这个设计挺重要。

因为它意味着 Hermes 不是把所有记忆都绑死在一个外部服务上，而是把语义层做成了可选增强层。

## 图里还有一个很关键的点：Periodic Nudge

这张图不只是三层 memory。

中间还有一个很值得看的机制：

periodic nudge。

图里写的是：

* every 300s
* configurable
* autonomous curation

它的逻辑是，系统会定期回看最近发生的事，然后问自己：

* 有没有新的偏好值得记？
* 有没有用户纠正值得记？
* 有没有项目约定值得记？

如果有，就调用 memory tool 去 add / replace / remove。 如果没有，就安静返回。

这件事我觉得特别像真正有用的 Agent 系统会去补的一层：

不是等你手动喂记忆，而是系统自己周期性判断“什么值得留下”。

所以 Hermes 这张图真正讲的，不只是三层存储。

它讲的是一套更完整的 memory 运转方式：

* 常驻的小 memory
* 按需搜索的大历史
* 可插拔的语义层
* 再加一个周期性的整理与写回机制

## 这张图真正说明了什么

如果让我用最短的话总结这张图，我会这么说：

### 第一层

把最关键、最稳定、最值得每轮都带上的东西，压进两份很小的 Markdown memory。

### 第二层

把大量历史沉到 SQLite + FTS5 里，需要时再搜回来。

### 第三层

用可插拔的 semantic provider 去做额外的语义记忆增强。

### 额外机制

再用 periodic nudge，定期决定哪些新信息值得写回 memory。

这才是这张图真正有价值的地方。

它不是在说“我们也有 memory”。 而是在认真回答：

不同类型的记忆，到底该放在哪一层。

## 最后一句

我自己现在越来越倾向于一个判断：

未来 Agent 的差距，未必只在模型层。 真正拉开差距的，很可能是它怎么处理连续性。

这张 Hermes memory 图让我觉得它值得看，不是因为它把概念讲得花哨。 恰恰相反，是因为它把 memory 这件事拆得足够具体。

什么该常驻。 什么该检索。 什么该交给外部 semantic layer。 什么该周期性整理。

如果一个 Agent 连这件事都没有认真处理，那它大概率还是更像一次性工具。 而不是一个能长期协作的系统。

以上，

--免费体验 3 天<生财有术>--

生财有术现在可以免费体验 3 天。互联网Top1大社群！

关注 AI、OpenClaw、Agents、互联网项目的，建议先进去白嫖看看。

公开网上的信息很多已经是二手、三手了，但社群里能更早看到一线实战者的项目反馈和机会判断。

不满意还能退款，基本没什么试错成本。

先看 3 天，再决定要不要留下。

有时候差距不是努力，而是你离优质信息源太远。

长按下边👇扫一扫！