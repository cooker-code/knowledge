---
title: [翻译] Anthropic：用 Agent Skills 赋能现实世界中的智能体
author: 进化中的程序员
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5ODAwOTc5NQ==&mid=2247485166&idx=1&sn=31725d0adc43b6f5c57a6e62ce93d628&chksm=91ecff48c14f65872fcf234c69e69544f663beb0b33fb4c6af77d9cda3ccbe289ee14a064b31&mpshare=1&scene=24&srcid=0105ewMqy9KPGg0l9JgcYOZz&sharer_shareinfo=e97c4a7517b25b06f606e3b80b90a5cc&sharer_shareinfo_first=e97c4a7517b25b06f606e3b80b90a5cc#rd
---

> Kafka 说，我是比较赞同，大家更多应该开发 skill 而不是 Agent。原文链接：https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills

*更新：我们已将**Agent Skills**作为跨平台可移植性的开放标准发布。（2025年12月18日）*

随着模型能力的提升，我们现在可以构建能够与完整计算环境交互的通用智能体。例如，Claude Code 可以通过本地代码执行和文件系统完成跨领域的复杂任务。但随着这些智能体变得更加强大，我们需要更加可组合、可扩展且可移植的方式来为它们配备特定领域的专业知识。

这促使我们创建了 **Agent Skills**：包含指令、脚本和资源的有组织文件夹，智能体可以动态发现和加载这些内容，以在特定任务中表现更出色。Skills 通过将你的专业知识打包成可组合的资源来扩展 Claude 的能力，将通用智能体转化为满足你需求的专业智能体。

为智能体构建 Skill 就像为新员工准备入职指南一样。现在，任何人不再需要为每个用例构建分散的、定制设计的智能体，而是可以通过捕获和共享程序性知识，用可组合的能力来专业化他们的智能体。在本文中，我们将解释什么是 Skills，展示它们如何工作，并分享构建自己 Skills 的最佳实践。

要激活 skills，你需要编写一个 SKILL.md 文件，其中包含针对智能体的自定义指导。

Skill 是一个包含 SKILL.md 文件的目录，该文件包含有组织的指令、脚本和资源文件夹，为智能体提供额外能力。

## Skill 的剖析

为了了解 Skills 的实际运作，让我们来看一个真实的例子： powering Claude 最近推出的文档编辑功能 的 skills 之一。Claude 已经对理解 PDF 有了很多了解，但在直接操作它们（例如填写表单）方面受到限制。这个 PDF skill 让我们能够赋予 Claude 这些新能力。

最简单来说，skill 是一个包含 `SKILL.md` 文件的目录。该文件必须以包含某些必需元数据的 YAML frontmatter 开头：`name` 和 `description`。在启动时，智能体会将每个已安装 skill 的 `name` 和 `description` 预加载到其系统提示中。

这个元数据是*渐进式披露*的**第一级**：它为 Claude 提供了刚好足够的信息，以了解何时应该使用每个 skill，而无需将所有内容加载到上下文中。该文件的实际主体是**第二级**细节。如果 Claude 认为 skill 与当前任务相关，它将通过读取完整的 `SKILL.md` 来加载 skill。

  

SKILL.md 文件的结构，包括相关元数据：名称、描述以及与 skill 应采取的具体操作相关的上下文。

SKILL.md 文件必须以包含文件名和描述的 YAML Frontmatter 开头，这些内容在启动时会被加载到系统提示中。

随着 skills 复杂性的增长，它们可能包含太多上下文而无法放入单个 `SKILL.md` 中，或者包含仅在特定场景中相关的上下文。在这些情况下，skills 可以在 skill 目录中捆绑其他文件，并从 `SKILL.md` 中按名称引用它们。这些额外的链接文件是**第三级**（及更多级）细节，Claude 可以根据需要选择导航和发现这些细节。

在下面显示的 PDF skill 中，`SKILL.md` 引用了 skill 作者选择与核心 `SKILL.md` 一起捆绑的两个额外文件（`reference.md` 和 `forms.md`）。通过将表单填写说明移到单独的文件（`forms.md`）中，skill 作者能够保持 skill 核心的精简，相信 Claude 只会在填写表单时才阅读 `forms.md`。

  

如何将额外内容捆绑到 SKILL.md 文件中.alt text

你可以在 skill 中包含更多上下文（通过额外文件），Claude 可以根据系统提示触发这些上下文。

渐进式披露是使 Agent Skills 灵活且可扩展的核心设计原则。就像一本组织良好的手册，从目录开始，然后是特定章节，最后是详细的附录，skills 让 Claude 仅根据需要加载信息：

  

此图像描述了 Skills 中上下文的渐进式披露。

拥有文件系统和代码执行工具的智能体不需要在处理特定任务时将整个 skill 读取到上下文窗口中。这意味着可以捆绑到 skill 中的上下文量实际上是无限的。

### Skills 和上下文窗口

下图显示了当 skill 被用户消息触发时上下文窗口如何变化。

  

此图像描述了 skills 如何在上下文窗口中触发。

Skills 通过系统提示在上下文窗口中触发。

显示的操作序列：

1. 首先，上下文窗口包含核心系统提示和每个已安装 skill 的元数据，以及用户的初始消息；
2. Claude 通过调用 Bash 工具读取 `pdf/SKILL.md` 的内容来触发 PDF skill；
3. Claude 选择读取与 skill 捆绑的 `forms.md` 文件；
4. 最后，Claude 继续执行用户的任务，因为它已经从 PDF skill 加载了相关指令。

### Skills 和代码执行

Skills 还可以包含代码，供 Claude 根据需要自行决定作为工具执行。

大型语言模型在许多任务上表现出色，但某些操作更适合传统的代码执行。例如，通过令牌生成排序列表比简单地运行排序算法要昂贵得多。除了效率方面的考虑，许多应用程序需要只有代码才能提供的确定性可靠性。

在我们的例子中，PDF skill 包含一个预先编写的 Python 脚本，用于读取 PDF 并提取所有表单字段。Claude 可以运行此脚本，而无需将脚本或 PDF 加载到上下文中。而且因为代码是确定性的，所以这个工作流程是一致且可重复的。

  

此图像描述了如何通过 Skills 执行代码。

Skills 还可以包含代码，供 Claude 根据任务性质自行决定作为工具执行。

## 开发和评估 skills

以下是一些有助于开始编写和测试 skills 的指导原则：

* **从评估开始：** 通过在代表性任务上运行智能体并观察它们在哪些方面遇到困难或需要额外上下文，来识别智能体能力中的具体差距。然后增量式地构建 skills 来解决这些不足。
* **为扩展而构建结构：** 当 `SKILL.md` 文件变得难以处理时，将其内容拆分为单独的文件并引用它们。如果某些上下文互斥或很少一起使用，保持路径分离将减少令牌使用。最后，代码既可以作为可执行工具，也可以作为文档。应该清楚 Claude 是应该直接运行脚本还是将它们作为参考读取到上下文中。
* **从 Claude 的角度思考：** 监控 Claude 在实际场景中如何使用你的 skill，并根据观察进行迭代：注意意外的轨迹或对某些上下文的过度依赖。特别注意 skill 的 `name` 和 `description`。Claude 在决定是否响应当前任务触发 skill 时会使用这些信息。
* **与 Claude 一起迭代：** 当你与 Claude 一起处理任务时，要求 Claude 将其成功的方法和常见错误捕获到 skill 内的可重用上下文和代码中。如果它在使用 skill 完成任务时偏离轨道，请它自我反思哪里出了问题。这个过程将帮助你发现 Claude 实际需要什么上下文，而不是试图预先预测它。

### 使用 Skills 时的安全考虑

Skills 通过指令和代码为 Claude 提供新功能。虽然这使它们变得强大，但也意味着恶意 skills 可能会在使用它们的环境中引入漏洞，或者指示 Claude 渗漏数据并采取意外操作。

我们建议仅从受信任的来源安装 skills。当从不太受信任的来源安装 skill 时，请在使用前彻底审计它。首先阅读 skill 中捆绑的文件内容，了解它的作用，特别关注代码依赖项和捆绑的资源（如图像或脚本）。同样，要注意 skill 中的指令或代码指示 Claude 连接到潜在不受信任的外部网络源。

## Skills 的未来

Agent Skills 目前在 Claude.ai、Claude Code、Claude Agent SDK 和 Claude Developer Platform 中得到支持。

在接下来的几周里，我们将继续添加支持创建、编辑、发现、共享和使用 Skills 的完整生命周期的功能。我们特别期待 Skills 能够帮助组织和个人与 Claude 共享他们的上下文和工作流程。我们还将探索 Skills 如何通过教授智能体涉及外部工具和软件的更复杂工作流来补充模型上下文协议 (MCP) 服务器。

展望未来，我们希望使智能体能够自主创建、编辑和评估 Skills，让它们将自己的行为模式编码为可重用的能力。

Skills 是一个概念简单、格式也简单的东西。这种简单性使组织、开发者和终端用户更容易构建定制化智能体并赋予它们新能力。

我们很高兴看到人们用 Skills 构建什么。今天就开始吧，查看我们的 Skills 文档和cookbook。

## 致谢

由 Barry Zhang、Keith Lazuka 和 Mahesh Murag 撰写，他们都非常喜欢文件夹。特别感谢 Anthropic 的许多其他人，他们倡导、支持和构建了 Skills。