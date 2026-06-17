---
title: Claude Code的上下文管理，我踩过的几个坑
author: 已晨
date: RockBotRockBot
url: https://mp.weixin.qq.com/s?__biz=MzkzNzYzMjQ3Mw==&mid=2247484581&idx=1&sn=809c4a92b0b182c232667d674c2d5ccf&chksm=c3644cc3f4fe43c0762480517c54748ad1b1264158221a1b721f319d76f8ea2da3e82ee5204c&mpshare=1&scene=24&srcid=0428duMQuqCKX4mGMN5gwvNF&sharer_shareinfo=0d729de3c69f2783c31f04803b6435c9&sharer_shareinfo_first=0d729de3c69f2783c31f04803b6435c9#rd
---

### 0、警惕太多信息导致的Context Rot

现在主流模型都有 1M 的 context window，大概相当于一次性装进去半本《红楼梦》，听起来很大，但也不能挥霍。

塞进去的无关信息越多，模型表现反而越差，这就是所谓的 Context Rot（上下文腐烂）

一个常见的误区是觉得Skill是万能的，什么都往一个 Skill 里塞。但 Skill 也受 context window 约束，塞太多也会出现Context Rot。

如上图所示，context window 里装的不只是你的对话，还包括 system prompt、CLAUDE.md、tool calls 和读取的文件内容。看着有 1M，能自由支配的空间其实没那么多。

Claude 4月份发了一篇文章 Using Claude Code: session management and 1M context ，讲述了在不同场景下最佳的上下文的管理策略。

读完之后，我发现自己踩了好几个坑。

### 1、Compact：主动压缩比被动压缩好得多

当接近 context window 大小时，Claude Code 会对上下文做自动压缩（autocompact）。当然你也可以主动通过 `/compact` 指令来触发。

这里有个坑：autocompact 的质量往往不好。 因为自动触发的时候，context 已经快满了，模型本身正处于 context rot 最严重的状态，注意力分散在大量 token 上，判断力下降。

更好的做法是在完成一个阶段性任务之后（比如写完一个模块、通过一轮测试），主动 `/compact` 一次。Compact 虽然是有损压缩，但大部分时候你可以信任Claude的判断，它可能比你更清楚哪些内容是关键的。

如果你明确知道哪些该留哪些该丢，也可以手动引导，比如 `/compact 保留 auth 模块的改动和接口定义，丢掉前面的调试记录`。

### 2、Rewind：与其纠正，不如回退重来

Rewind相当于跳回某个之前的消息，从那个点重新开始。后面的对话会从 context 中丢弃。

一个典型场景：Claude 读了五个文件，尝试了方案 A，没走通。你的第一反应可能是说"方案 A 不行，试试方案 B"。但这样方案 A 的所有 tool call 和输出还留在 context 里，占空间还可能干扰后续判断。

更好的做法是在读完文件之后Rewind（按两下Esc），重新给指令："不要用方案 A，foo 模块没有暴露那个接口，直接走方案 B"。

### 3、Clear：干净的上下文不等于好的上下文

我之前一直有个误区，习惯性地`/clear`，但没意识到 `/clear` 的代价是 AI 要重新读一遍相关文件才能恢复上下文，既慢又费 token。

其实在 1M context window 下，你有足够的空间在同一个 session 里完成一整个需求的开发、测试、文档。

真正需要 `/clear` 的场景只有一个：你要开始一个跟当前任务完全无关的新任务。

文中关于clear的配图有点歧义，`/clear`就是单纯的清空会话，后面不支持带参数。上图的brief是你在新会话里做的总结（如果你愿意总结的话）

### 4、Subagent：把中间过程留在子agent里

之前我一直没有搞明白Subagent的作用，也不理解为什么有了Skill还需要Subagent，但从上下文管理角度就很好理解了。

Subagent适合那些会产生大量中间输出、但你只需要结论的子任务。比如跑一轮编译，中间几十条 warning 和 log 你不关心，只需要知道通没通过；再比如，code review场景，subagent逐文件分析完，只把"3 个需要改的问题"带回来，中间逐行推理的过程我们也不关心。

Subagent 有自己独立的上下文窗口，干完活把结果带回来，中间的信息不会污染主Agent。

### 5、Continue：换Skill可以不换上下文

我在这里踩过坑：需求澄清 Skill 跑完、生成了需求文档 requirement.md。在执行技术方案 Skill之前，执行了/clear 清空上下文。

结果 Claude Code 又得重新读一遍代码仓库才能出方案。不仅浪费时间，还浪费了token。

其实上一轮积累的代码理解还在 context 里，直接 continue 就行。

如果两个Skills之间有强关联，那可以不换上下文。

### 6、总结：上下文管理没有银弹

这几个坑踩下来，对我自己的 long-running agent 设计帮助挺大。

Context 管理的核心策略是判断Context是否还有用，有用就留着，没用就清理，但清理方式别只会 /clear。

当然，你可以把上下文管理委托给claude code，这在大部分场景都是可行的。但如果你正在构建long-running agent，那势必需要对上下文做更精细化的管理。