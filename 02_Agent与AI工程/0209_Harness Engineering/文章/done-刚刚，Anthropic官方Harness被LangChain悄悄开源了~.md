> 已吸收至：[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/上下文与工具加载边界|上下文与工具加载边界]]、[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/多角色编排与权限隔离|多角色编排与权限隔离]]、[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/运行环境与沙箱底座|运行环境与沙箱底座]]
---
title: 刚刚，Anthropic官方Harness被LangChain悄悄开源了~
author: PaperAgent
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247506555&idx=2&sn=d040a7a4aa476ed6b378f887a8147b83&chksm=c33283e145303e796e6a962dda90f092ecf3610d851a95b6fa2ff2e7ab283b084ad96cae1c25&mpshare=1&scene=24&srcid=0415XPyNwKZOmQ1FHpyerHK5&sharer_shareinfo=75719133fbf0e94b702a047d3e94ebe9&sharer_shareinfo_first=75719133fbf0e94b702a047d3e94ebe9#rd
---

大家好，我是PaperAgent，不是Agent！

上周，Anthropic下场，发布了官方Harness： **Claude Managed Agents**，将Harness从概念变成产品。[Agents统一综述：Harness、记忆、Skills和协议](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247506473&idx=1&sn=23d3025038c61e750a31131efac2c6da&scene=21#wechat_redirect)

**核心思路很漂亮**：把**大脑Harness**（Claude 及其控制循环，负责推理和决策）和**手Sandbox**（执行环境，运行代码和编辑文件）解耦，再把会话日志做成一个独立于上下文窗口的持久化存储。这样，容器挂了能恢复，会话不怕丢，沙箱随便换。

这套设计确实先进，但可惜——它是 Anthropic 自用的，闭源，你得用 Claude，你的会话记忆全锁在它的 API 里。

结果 **LangChain** 直接跳出来说：你这套，我也有，而且我开源。[斩获60k星，字节开源Agent Harness，龙虾慌了](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247506466&idx=2&sn=e68d9259d9d72242e4ac86c7225df414&scene=21#wechat_redirect)

LangChain正式发布 **Deep Agents Deploy** 的 Beta 版本。这是部署一个**模型无关**、**开源**的 Agent 框架（Agent Harness）的最快方式，且可直接用于生产环境。

Deep Agents Deploy 为开放世界而生。它基于 **Deep Agents** —— 一个开源、模型无关的 Agent 框架。框架（Harness）与记忆（Memory）紧密绑定，这意味着选择开源框架就是选择拥有你自己的记忆，而不是被锁定在专有框架或单一模型中。

一个闭源，一个开源。一个锁死在 Anthropic 的生态里，一个让你随便换模型、自己管记忆。

有意思了。

## 从框架工程到生产部署

在过去几个月里，\*\*"框架工程"（Harness Engineering）\*\* 已成为将大语言模型转化为 Agent 的学科。这些框架包含编排逻辑、工具、技能，它们构成了 Agent 的基础，同时允许开发者通过自定义指令、工具、技能来针对特定用例进行定制。

但要将 Agent 投入生产，还需要几个步骤：

* 以多租户、可扩展的方式部署 Agent 编排逻辑和记忆系统
* 设置沙盒环境，使其按 Agent 会话自动创建
* 搭建与 Agent 交互的端点，包括 MCP、A2A 以及用于人机协同、记忆管理等端点

今天，我们将所有这些步骤打包成一个命令：`deepagents deploy`

## 你在部署什么？

通过 `deepagents deploy`，你部署的是**自定义 Agent**。需要指定几个参数：

* **model**：使用的大语言模型。Deep Agents 支持任何模型或模型提供商，包括 OpenAI、Google、Anthropic、Azure、Bedrock、Fireworks、Baseten、Open Router 和 Ollama。更多信息请查看我们的模型文档。
* **AGENTS.md**：Agent 的核心指令集，在会话开始时加载的指令。
* **skills**：Agent 技能，允许通过 Markdown 文件提供专业知识和通过脚本执行操作。更多信息请查看技能文档。
* **mcp.json**：Agent 可通过 MCP 协议（HTTPS/SSE）调用的工具。
* **sandbox**：如有需要，可指定 Agent 用于工作和运行技能的沙盒环境。Deep Agents 开箱即用地集成了 Daytona、Runloop、Modal 或 LangSmith Sandboxes。任何沙盒提供商都可以与 Deep Agents 配合使用；请查看我们的实现指南。

## 部署机制

在底层，`deepagents deploy` 将你的 Deep Agent 与其专属的 **LangSmith Deployment 服务器** 打包在一起。这是一个可用于生产环境、支持水平扩展的服务器。

它会启动一个包含 **30+ 端点** 的服务器，包括：

* **MCP**：允许你将部署的 Agent 作为工具调用
* **A2A**：允许你在多 Agent 设置中调用部署的 Agent
* **Agent Protocol**：便于你编写精美的 UI 与部署的 Agent 交互
* **Human-in-the-loop**：允许你设置护栏，控制 Agent 在没有人工干预的情况下可以或不可以做什么
* **Memory 端点**：便于你轻松访问 Agent 的短期或长期记忆

## 开放生态系统

`deepagents deploy` 的一个关键部分是融入开放生态系统。具体来说：

* 我们使用 `deepagents`，一个完全开源、MIT 许可证的框架，支持 Python 和 TypeScript
* 我们使用 `AGENTS.md`，一个开放标准，作为指定 Agent 指令的方式
* 我们使用 Agent Skills，一个开放标准，作为向 Agent 提供专业知识的方式
* 我们集成**每一个**模型提供商，让你完全掌控。没有 Anthropic 锁定，你可以为任务选择最佳的模型组合，甚至开源模型。
* 我们集成**每一个**沙盒提供商，让你完全掌控
* 我们通过 MCP、A2A 和 Agent Protocol 开放标准暴露 Agent
* 你可以自托管 LangSmith Deployments，从而托管并拥有你自己的记忆

## 记忆：为什么开放生态系统至关重要

Agent 框架和 Agent 平台需要开放生态系统的核心原因是**记忆**。

Agent 框架与记忆紧密绑定（Sarah Wooders 写了一篇关于此的精彩文章）。框架的一个关键角色是管理上下文（记忆就是上下文）。随着框架的更多部分被封闭、锁定在 API 之后——你的记忆也被锁定。

实际上，从一个模型切换到另一个模型相当容易（当然，你可能需要稍微调整提示词，但不会太难）。因此，仅模型 API 本身并没有太大的锁定性（正如我们最近看到的从 OpenAI 向 Anthropic 的大规模迁移）。

但当你开始将记忆打包在这些 API 之后——无论是短期记忆还是长期记忆——它就会造成难以置信的锁定。

想象你创建了一个内部的 SDR（销售开发代表）Agent。它开始时很基础，但随着与用户交互，它会实时学习。这些记忆不断积累——但都被锁定在封闭 API 之后。如果你想迁移出那个框架，或那个模型——那就意味着重置你的 Agent 记忆，从头开始。

对于你向客户开放的 Agent 来说，情况更糟。你构建了一个面向客户的销售 Agent，它积累了与客户交互的记忆。所有这些记忆都被锁定在封闭 API 之后。这些记忆是你应该构建的数据飞轮的一部分，用于随时间改善客户体验。但它们不再属于你——它们属于拥有那个封闭 API 的人。

`deepagents deploy` 以标准格式（AGENTS.md、skills 和其他文件）存储记忆，允许你直接通过 API 查询它们，如果你选择自托管，确保记忆始终只保留在你的数据库中。

## 尝试开源框架

在专有框架之上构建 Agent 会造成难以置信的锁定。我们相信，Agent 的构建和部署应该简单，但你仍然拥有模型选择权和记忆所有权。

如果你也相信这一点，今天就尝试 `deepagents deploy`。

```
https://www.anthropic.com/engineering/managed-agents
https://blog.langchain.com/deep-agents-deploy-an-open-alternative-to-claude-managed-agents/
```

[动手设计AI Agents：（编排、记忆、插件、workflow、协作）](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247492838&idx=2&sn=1e25832e7300ef312721325d0def30b4&scene=21#wechat_redirect)

[分享两篇Claude Skills最新论文，有3个核心结论](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247505843&idx=1&sn=9a8a4915606ee747998015395413db87&scene=21#wechat_redirect)

[会学习的龙虾，才是好龙虾：OpenClaw-RL](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247505225&idx=1&sn=4b32282cf6853e29f884bdfb85f89f2e&scene=21#wechat_redirect)  
[2026，做Agentic AI，绕不开这两篇开年综述](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247502666&idx=1&sn=d6a467896c6753c8d8634c7400d8dbb4&scene=21#wechat_redirect)

---

每天一篇大模型Paper来锻炼我们的思维~已经读到这了，不妨点个👍、❤️、↗️三连，加个星标⭐，不迷路哦~