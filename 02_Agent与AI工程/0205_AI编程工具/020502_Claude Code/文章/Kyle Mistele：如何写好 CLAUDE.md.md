---
title: Kyle Mistele：如何写好 CLAUDE.md
author: 江鸟阁长
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzOTY0NTc3NA==&mid=2247484759&idx=1&sn=f9de6fa9ea1476afbd9e4d7b5937c130&chksm=c38aeec5c6b1f92009f607c0d591ebae7ef6a4ea871be04d83c40a12680b9dfc89759004ab29&mpshare=1&scene=24&srcid=1202BSRkYacefrFr2KtrQFZD&sharer_shareinfo=f8323710677cca1d41827a7cfb8c17f0&sharer_shareinfo_first=f8323710677cca1d41827a7cfb8c17f0#rd
---

关于如何写好CLAUDE.md，读到一篇非常有启发的文章。以前我总想把要求写得完整一点，看完才明白，真正的高标准其实是「克制」。

原文地址：

https://www.humanlayer.dev/blog/writing-a-good-claude-md

以下是译文。

> 注：本文同样适用于 AGENTS.md，AGENTS.md 是针对 OpenCode、Zed、Cursor 和 Codex 等Agent及其框架 的 CLAUDE.md 开源等价物。

## 原则：LLM 在（大多数情况下）是无状态的

LLM 是无状态函数。它们的参数在用于推理时已经冻结，因此不会随着时间学习。模型对你的代码库所了解的一切，只来自你输入给它的那些 token。

类似地，Claude Code 等编码Agent运行框架通常要求你显式管理Agent的记忆。CLAUDE.md（或 AGENTS.md）是默认会被注入你与Agent每一次对话的唯一文件。

这带来三个重要的含义：

* 编码Agent在每个会话开始时，对你的代码库一无所知。
* 每次开始一个会话时，都必须把关于你的代码库的关键信息告诉Agent。
* CLAUDE.md 是实现这一点的首选方式。

## CLAUDE.md 让 Claude 熟悉你的代码库

由于 Claude 在每个会话开始时都对你的代码库一无所知，你应该使用 CLAUDE.md 来让 Claude 了解你的代码库。高层次来说，它应该涵盖：

* WHAT：告诉 Claude 所用的技术、技术栈、项目结构。给 Claude 一张代码库的地图。对单体仓库（monorepo）来说尤其重要！告诉 Claude 有哪些应用、有哪些共享包，以及它们分别是干什么用的，这样它才知道应该到哪里去找东西。
* WHY：告诉 Claude 项目的目的，以及仓库中每一部分在做什么。项目中不同部分的目标和功能是什么？
* HOW：告诉 Claude 它应该如何在该项目上工作。例如，你是否使用 bun 而不是 node？你需要包含所有让它能在项目上实际做有意义工作的必要信息。Claude 要如何验证自己的修改？它要如何运行测试、类型检查和编译步骤？

但你如何做到这一点也很重要！不要试图把 Claude 可能需要运行的每一个命令都塞进 CLAUDE.md 文件里——这样只会得到次优结果。

## Claude 经常会忽略 CLAUDE.md

不论你使用的是哪个模型，你可能会注意到 Claude 经常忽略 CLAUDE.md 文件中的内容。

你可以通过在 Claude code CLI 和 Anthropic API 之间，利用 ANTHROPIC\_BASE\_URL 放置一个日志代理来自己验证这一点。Claude code 会在发送给Agent的用户消息中，把你的 CLAUDE.md 文件和下面这个 system 提醒一起注入：

```
<system-reminder>      IMPORTANT: this context may or may not be relevant to your tasks.       You should not respond to this context unless it is highly relevant to your task.</system-reminder>
```

> 重要：这些背景信息可能与您的任务相关，也可能无关。除非这些背景信息与您的任务高度相关，否则您不应对此作出回应。

结果就是，如果 Claude 认为 CLAUDE.md 的内容与当前任务无关，它就会忽略这些内容。文件中凡是不对当前要做的任务“普遍适用”的信息越多，Claude 越可能忽略文件中的指令。

Anthropic 为什么要这么做？很难确定，但我们可以稍微推测一下。我们见过的大多数 CLAUDE.md 文件都包含大量并不广泛适用的指令。很多用户把这个文件当作添加“热修复”的方式，通过不断追加许多并不一定具有普适性的指令来修正他们不喜欢的行为。

我们只能假设，Claude Code 团队发现，通过告诉 Claude 可以忽略这些糟糕的指令，实际上能产生更好的结果。

## 创建一个好的 CLAUDE.md 文件

以下部分提供了一些关于如何遵循上下文工程最佳实践来编写一个优秀的 CLAUDE.md 文件的建议。实际效果可能因人而异。并不是所有规则在你的场景里都是最优的。

像其它任何事情一样，只有当你：理解什么时候以及为什么可以打破这些规则，并且有充分理由时，就请自由打破它们...

1. 你明白何时以及为何可以打破它们
2. 你有充分的理由这样做

### 少即是多（Less is more）

试图把 Claude 可能需要运行的每一个命令，以及你的代码规范和风格指南全部塞进 CLAUDE.md 文件的冲动是很强烈的，但我们不建议这么做。

尽管这个话题还没有以极其严谨的方式得到研究，但已经进行了一些研究，表明以下几点：

1. **前沿级的思考型 LLM 可以在大约 150–200 条指令的规模上保持相对稳定的遵循能力。**小模型能关注的指令数比大模型少，而非思考型模型能关注的指令数又比思考型模型少。
2. **小模型在指令数量增加时会变得“糟糕得更快、幅度更大”。**具体来说，随着指令数量增加，小模型的指令遵循表现倾向于呈指数衰减，而更大的前沿思考型模型则表现为线性衰减（见下图）。因此，我们不建议在多步骤任务或复杂实现方案上使用小模型。
3. **LLM 偏向于遵循提示“边缘”上的指令**：最前面的（Claude Code 的 system 消息和 CLAUDE.md），以及最后面的（最近的用户消息）。
4. **随着指令数量的增加，指令遵循质量会整体下降。**这意味着当你给 LLM 更多指令时，它不会简单地忽略较新的（"文件中位置更靠后的"）指令——它会开始均匀地忽略所有指令

我们对 Claude Code  代码框架的分析表明，它的 system prompt 中大约包含 ~50 条独立的指令。根据你使用的模型不同，这几乎已经占用了Agent可可靠遵循指令数量的三分之一——而这还没算上规则、插件、技能或用户消息。

这意味着你的 CLAUDE.md 文件应当尽可能少地包含指令——理想情况下只保留那些对你的任务“普遍适用”的指令。

### CLAUDE.md 的文件长度与应用范围

在其它条件相同的情况下，当 LLM 的上下文窗口被聚焦且相关的内容填满时（包括示例、相关文件、工具调用以及工具结果），它在任务上的表现会比上下文窗口中充斥着大量无关内容时更好。

由于 CLAUDE.md 会被注入每一个会话，你应确保其中的内容尽可能具有普遍适用性。

例如，避免包含诸如“如何设计一个新的数据库 schema”之类的指令——当你在做其它不相关工作时，这些内容既无关紧要，又会干扰模型！

在长度上，“少即是多”的原则同样适用。尽管 Anthropic 没有官方推荐 CLAUDE.md 应该有多长，但普遍共识是少于 300 行为佳，而且越短越好。

在 HumanLayer（指作者所在组织），我们根目录下的 CLAUDE.md 文件不到 60 行。

### 逐步披露（Progressive Disclosure）

在较大的项目中，要写出既简洁又覆盖你希望 Claude 知道的一切的 CLAUDE.md 文件是一个挑战。

为了解决这个问题，我们可以利用“逐步披露（Progressive Disclosure）”原则，来确保 Claude 只有在需要时才会看到与任务或项目相关的特定指令。

与其把所有关于如何构建项目、如何运行测试、代码约定或者其它重要上下文的信息都放进 CLAUDE.md 文件，我们建议将这些与任务相关的指令放在项目中其它位置的独立 markdown 文件中，并使用具有自描述性的文件名。

例如：

```
agent_docs/  |- building_the_project.md  |- running_tests.md   |- code_conventions.md  |- service_architecture.md  |- database_schema.md  |- service_communication_patterns.md
```

然后，在 CLAUDE.md 文件中，你可以列出这些文件并附上对每个文件的简要说明，并指示 Claude 决定哪些（如果有的话）与当前任务相关，并在开始工作之前先阅读它们。或者，你也可以要求 Claude 在阅读之前，先把它想要阅读的文件列表给你审批。

**优先使用“指针”而不是“拷贝”。**如果可以，尽量不要在这些文件中包含代码片段——它们会很快过时。相反，使用“文件名:行号”的引用来把 Claude 指向具有权威性的上下文位置。

从概念上讲，这与 Claude Skills 的设计初衷非常相似，尽管 Skills 更侧重于工具使用而非指令。

### Claude（不是）一个昂贵的 linter

我们最常见到的 CLAUDE.md 内容之一，就是代码风格指南。**永远不要让 LLM 去做 linter 的工作。与**传统 linter 和格式化工具相比，LLM 成本高且非常慢。我们认为，只要有机会，你都应该优先使用确定性的工具。

代码风格指南不可避免地会在你的上下文窗口里添加一堆指令和大多无关的代码片段，从而降低 LLM 的性能和指令遵循能力，并占用你的上下文窗口。

LLM 是上下文学习者！如果你的代码遵循某一套风格或模式，那么只要让它在代码库中做几次搜索（或给它一份好的调研文档），你的Agent通常就能倾向于自发地遵循现有的代码模式和约定，而无需额外告知。

如果你对此非常在意，你甚至可以考虑设置一个 Claude Code Stop hook，用它来运行你的格式化器和 linter，并将错误呈现给 Claude 进行修复。不要让 Claude 自己去找格式问题。

**加分项：**使用可以自动修复问题的 linter（我们喜欢 Biome），并仔细调优那些可以安全自动修复的规则，以实现最大范围的（安全）覆盖。

你也可以创建一个 Slash 命令，其中包含你的代码指南，并指向版本控制中的变更、或者当前的 git status，或类似信息。这样，你就可以把实现和格式化分开处理。你会发现，两者的效果都会更好。

### 不要使用 /init 或自动生成你的 CLAUDE.md

Claude Code 以及其它带有 OpenCode 的运行框架，都提供了自动生成 CLAUDE.md（或 AGENTS.md）文件的方法。由于 CLAUDE.md 会被注入 Claude Code 的每一次会话，它是整个框架中杠杆率最高的点之一——好坏取决于你如何使用它。

一行糟糕的代码就是一行糟糕的代码。一行糟糕的实现计划可能会产生大量糟糕的代码。一行误解系统工作方式的糟糕调研文字，可能会在计划中产生许多糟糕的行，从而进一步导致更多糟糕的代码。

但 CLAUDE.md 文件会影响你的工作流的每一个阶段，以及每一个产出。因此，我们认为你应该花时间非常仔细地思考写入其中的每一行内容：

## 总结

1. CLAUDE.md 用于让 Claude 熟悉你的代码库。它应该定义项目的 WHY、WHAT 和 HOW。
2. 指令越少越好。不要遗漏必要的指令，但应尽可能少地在文件中加入指令。
3. 保持 CLAUDE.md 的内容简洁且具有普遍适用性。
4. 使用逐步披露——不要把所有你希望 Claude 知道的信息都一次性告诉它。而是告诉它如何找到重要信息，这样它就能在需要时去查找并使用这些信息，从而避免膨胀你的上下文窗口或指令数量。
5. Claude 不是 linter。请使用 linter 和代码格式化工具，并按需使用 Hooks 和 Slash 命令等其它特性。
6. CLAUDE.md 是整个运行框架中杠杆率最高的部分，因此避免自动生成它。你应该精心打磨其中的内容，以获得最佳效果。

---

**往期：**

**[当gpt 5.1开始强调「人味」，ai模型真的需要和人好好说话](https://mp.weixin.qq.com/s?__biz=MzkzOTY0NTc3NA==&mid=2247484708&idx=1&sn=4f36af35d664b35d2bf0a2d879f2ae31&scene=21#wechat_redirect)**

[接mcp, 拆subagents，ai agent设计关键仍是：信息要喂足，但别喂堵](https://mp.weixin.qq.com/s?__biz=MzkzOTY0NTc3NA==&mid=2247484685&idx=1&sn=9d6619d21638babb206fef1772bb3089&scene=21#wechat_redirect)

[不再需要额外维护claude.md、gemini.md了，上下文文件统一命名为agents.md](https://mp.weixin.qq.com/s?__biz=MzkzOTY0NTc3NA==&mid=2247484559&idx=1&sn=7f076a505143366d9f03e4b3759f1c6c&scene=21#wechat_redirect)

[对ai生成代码的“过度防御”保持警惕](https://mp.weixin.qq.com/s?__biz=MzkzOTY0NTc3NA==&mid=2247484647&idx=1&sn=257259193545423264105e1aa8fa50e9&scene=21#wechat_redirect)

[精读Anthropic最新的MCP思考，用“软件工程”的老办法解决 AI 的新问题，为你总结10个值得收藏的启发](https://mp.weixin.qq.com/s?__biz=MzkzOTY0NTc3NA==&mid=2247484611&idx=1&sn=8cdc1c2b75e6ffc0c18761fa677f751c&scene=21#wechat_redirect)

---

ai时代，抱团取暖，欢迎链接。

备注「ai」，加交流群。