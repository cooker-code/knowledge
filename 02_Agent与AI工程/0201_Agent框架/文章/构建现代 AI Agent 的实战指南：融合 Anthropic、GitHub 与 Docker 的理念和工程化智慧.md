---
title: 构建现代 AI Agent 的实战指南：融合 Anthropic、GitHub 与 Docker 的理念和工程化智慧
author: AI 启蒙小伙伴
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwNDExODE4Nw==&mid=2247492038&idx=1&sn=ea9a77e441f455ecdcd3f7f630555da7&chksm=c1f546bca76caf1bdf19ca5fc1b4478589fc364dca5e4aace45fb239cf3f4caf67c8cc9f0d7f&mpshare=1&scene=24&srcid=12042jeveqJ2koAPm1orKdjP&sharer_shareinfo=7a8e15bb1590ede879497356eae35eb0&sharer_shareinfo_first=7a8e15bb1590ede879497356eae35eb0#rd
---

在深度理解了 Anthropic、Github 和 Docker 的三篇 AI Agent 文章后，对于如何构建 AI Agent，我们学到了这些理念和工程化实战智慧。

来自博客「How to Build an AI Agent: Lessons from Anthrophic, Github and Docker」

https://interviewready.io/blog/how-to-build-an-ai-agent-lessons-from-anthrophic-github-and-docker

核心观点一：设计理念（来自 Anthropic）

——“拒绝过度设计，从简单工作流开始”

Effective harnesses for long-running agents

https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents

> Agents still face challenges working across many context windows. We looked to human engineers for inspiration in creating a more effective harness for long-running agents.

1. 先有“工作流”，再有“智能体”：

· 不要一上来就试图构建一个全知全能、完全自主的 AI。

· Lesson：大多数业务需求只需要确定性的工作流。比如“先搜索，再总结，最后发邮件”，这是一条直线。只有当路径不确定，需要 AI 自己做决定时（“我该搜索还是直接回答？”），才称之为 Agent。

2. 简单的力量：

· 文章强调使用可组合的模式而不是复杂的框架。

· 推荐模式：

· Prompt Chaining：把任务切碎，一步步喂给 AI。

· Orchestrator-Workers：一个 AI 负责分配任务，几个 AI 负责具体干活。

· Evaluator-Optimizer：一个 AI 写，另一个 AI 负责挑剔和修改。

核心观点二：基础设施（来自 Docker）

——“给 AI 一个安全的‘家’和统一的‘手’”

Docker + E2B: Building the Future of Trusted AI

https://www.docker.com/blog/docker-e2b-building-the-future-of-trusted-ai/

> Docker + E2B: Secure Access to Hundreds of MCP Tools
>
> Starting today, every E2B sandbox includes direct access to Docker’s MCP Catalog, a collection of 200+ tools such as GitHub, Perplexity, Browserbase, and ElevenLabs, all enabled by the Docker MCP Gateway.

如果说 LLM 是大脑，Docker 正在成为 AI Agent 的躯干和手脚。文章强调了 Docker 在 Agent 时代的两个新角色：

1. 标准化的工具接口（MCP）：

· 以往 AI 要连接数据库或 Google 日历，每家公司都有不同的写法。

· Lesson：Docker 正在大力推广 MCP (Model Context Protocol)。这是一种通用标准，让 AI 像插 USB 一样即插即用地连接外部工具。你不需要为每个工具重写代码，只需使用标准化的 MCP 服务。

2. 安全沙箱（Sandboxing）：

· Agent 需要执行代码、读写文件，直接在你的电脑上跑太危险（可能会误删文件）。

· Lesson：利用 Docker 容器为 Agent 提供一个隔离环境。Agent 可以在里面随意折腾、安装软件、运行代码，即便搞砸了，把容器删掉即可，不会影响主机。

核心观点三：交互与落地（来自 GitHub）

——“上下文就是一切”

How to write a great agents. md: Lessons from over 2,500 repositories

https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/

> Key takeaways
>
> Building an effective custom agent isn’t about writing a vague prompt; it’s about providing a specific persona and clear instructions.
>
> My analysis of over 2,500 agents.md files shows that the best agents are given a clear persona and, most importantly, a detailed operating manual.

通过 Copilot Workspace 展示了 Agent 如何真正融入工作。

1. 感知环境：

· 一个好的 Agent 不能只看你的一句话，它必须看懂你的“整个世界”。

· Lesson：就像 GitHub Copilot 的 @ workspace 功能一样，Agent 需要能够理解整个项目仓库、文件结构和依赖关系。构建 Agent 时，关键在于如何高效地把这些背景信息喂给 AI。

2. 人机协作而非替代：

· GitHub 的经验表明，Agent 不应该是一个黑盒，它应该是一个透明的合作伙伴。

· Lesson：让用户看到 Agent 的计划，并允许用户在 Agent 执行任务的过程中进行干预和纠正。

总结：如何构建现代 AI Agent？

· 大脑：使用 Anthropic 提倡的简单模式（如指挥官模式），不要迷信复杂的 Agent 框架。

· 身体：使用 Docker 容器来运行 Agent，确保安全；使用 MCP 协议来连接工具，确保通用性。

· 灵魂：像 GitHub 一样重视上下文，让 Agent 理解业务全貌，而不仅仅是回答问题。

一句话总结：现在的 AI Agent 开发已经过了“甚至不知道怎么写 Prompt”的草莽阶段，正在进入 标准化（Docker/MCP）、模式化（Anthropic Patterns）和工程化（GitHub Context）的成熟期。

信息卡提示词

[醒目杂志风信息/知识卡提示词（手机友好）](https://mp.weixin.qq.com/s?__biz=MzkwNDExODE4Nw==&mid=2247492026&idx=1&sn=35eb0352436355352c183530422e93ff&scene=21#wechat_redirect)

[提示词分享：用 AI 模型给文章生成信息卡片（Gemini 3 效果最佳、Kimi K2 最听话稳定）](https://mp.weixin.qq.com/s?__biz=MzkwNDExODE4Nw==&mid=2247492001&idx=1&sn=30a1b40555c6a5c8f55c69a6d5e1ca5a&scene=21#wechat_redirect)

.cls-1{fill:#001e36;}.cls-2{fill:#31a8ff;}

.cls-1{fill:#001e36;}.cls-2{fill:#31a8ff;}

.cls-1{fill:#001e36;}.cls-2{fill:#31a8ff;}

.cls-1{fill:#001e36;}.cls-2{fill:#31a8ff;}

.cls-1{fill:#001e36;}.cls-2{fill:#31a8ff;}

.cls-1{fill:#001e36;}.cls-2{fill:#31a8ff;}

.cls-1{fill:#001e36;}.cls-2{fill:#31a8ff;}

.cls-1{fill:#001e36;}.cls-2{fill:#31a8ff;}