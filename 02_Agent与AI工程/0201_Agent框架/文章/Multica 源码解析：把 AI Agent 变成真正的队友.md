---
title: Multica 源码解析：把 AI Agent 变成真正的队友
author: Sunward Rivers
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYyNDM4NTk3Mg==&mid=2247484463&idx=1&sn=ef3e5750afba646c065d81682b6c37e4&chksm=f1b5253576a4b43cbe0421019318b2d40d49941acaf5226287734fa9ccca7f4c54025ac97560&mpshare=1&scene=24&srcid=0424OonirpDp0IAJJ4gX4hQy&sharer_shareinfo=27c7d0d29d559ad0fa44219f7df36430&sharer_shareinfo_first=27c7d0d29d559ad0fa44219f7df36430#rd
---

你现在大概已经习惯了用 Claude Code 或 Codex 写代码。输入一个 prompt，Agent 开始干活，你盯着终端看输出。

但如果你有 3 个 Agent 同时在跑呢？5 个呢？谁在做什么？做到哪了？卡住了吗？需要人工介入吗？

这就是 Multica 要解决的问题。

Multica 不是又一个 AI 编程 Agent。它是 Agent 的管理平台——一个开源的、AI 原生的项目管理工具，把 coding agent 当作真正的团队成员来管理。你给 Agent 分配 Issue，就像给同事分配任务一样。Agent 自己认领、执行、报告进度、遇到问题主动说。

这篇文章从源码层面拆解 Multica 的架构，搞清楚它到底怎么工作的，以及它和直接用 Claude Code / Codex 有什么本质区别。

## Multica 是什么

一句话：Multica 是一个开源的 managed agents 平台，让你像管理人类团队一样管理 AI coding agent。

它的名字来自 Multics（Multiplexed Information and Computing Service），1964 年诞生的分时操作系统先驱。Multics 解决了多用户共享计算资源的问题，Multica 要解决的是多 Agent 协作的问题。

核心信息：

* **定位**：AI 原生项目管理平台，不是 Agent 框架
* **开源协议**：开源，可自托管
* **技术栈**：Next.js 16 前端 + Go 后端（Chi router + sqlc）+ PostgreSQL 17（pgvector）+ Go CLI Daemon
* **支持的 Agent**：Claude Code、Codex、OpenClaw、OpenCode
* **部署方式**：Cloud 托管 或 Docker 自托管
* **目标用户**：2-10 人的 AI 原生小团队

关键区分：Multica 不执行代码。它管理执行代码的 Agent。Claude Code 是干活的人，Multica 是分配活的经理。

## 源码架构拆解

Multica 的代码库是一个 monorepo，核心分为四层。

前端层 apps/web/

### 前端：Next.js 16 + Zustand

前端在 `apps/web/` 目录下，用 Next.js 16 App Router 构建。状态管理用 Zustand（轻量级，比 Redux 简单得多），数据获取用 TanStack Query。

有意思的是前端的组织方式。`packages/core/` 提供共享的 API 客户端和认证逻辑，`packages/ui/` 提供原子级 UI 组件，`apps/web/features/` 按功能模块组织页面。这个结构说明团队从一开始就考虑了多端复用——除了 Web，还有 `apps/desktop/`（Electron 桌面端）。

实时更新通过 WebSocket 实现。当 Agent 在执行任务时，进度信息通过 WebSocket 实时推送到前端，你能在看板上看到 Agent 的工作状态实时变化。

### 后端：Go + Chi + sqlc

后端在 `server/` 目录下，用 Go 写。路由用 Chi（轻量级 HTTP router），数据库查询用 sqlc（从 SQL 生成类型安全的 Go 代码），WebSocket 用 gorilla/websocket。

后端的核心是 `realtime.Hub`——一个 WebSocket 广播中心。所有状态变更（任务创建、Agent 认领、进度更新、完成/失败）都通过 Hub 广播给所有连接的客户端。这是 Multica 实时性的基础。

数据库用 PostgreSQL 17，带 pgvector 扩展。pgvector 的存在暗示了未来可能的语义搜索能力——比如根据 Issue 描述自动匹配最合适的 Agent 或 Skill。

### Agent Daemon：本地执行引擎

这是 Multica 架构中最独特的部分。

Daemon 是一个运行在你本地机器上的 Go 进程（`cmd/multica/daemon.go`）。它做三件事：

1. **自动检测**：扫描系统 PATH，发现可用的 Agent CLI（claude、codex、openclaw、opencode）
2. **任务轮询**：定期向后端 API 轮询 `/api/daemon/tasks`，获取分配给本机 Runtime 的任务
3. **隔离执行**：在隔离环境中启动对应的 Agent CLI，传入任务上下文，收集输出

Daemon 启动后，你的机器就变成了一个 "Runtime"——一个可以执行 Agent 任务的计算环境。在 Multica 的 Web UI 里，你能看到所有 Runtime 的状态、可用的 Agent CLI、当前负载。

这个设计的巧妙之处在于：Agent 的执行发生在你自己的机器上，代码不需要上传到任何第三方服务器。对安全敏感的团队来说，这很重要。

### Task Lifecycle：任务的一生

每个任务在 Multica 里都有明确的生命周期：

图表

什么意思呢？当你在看板上把一个 Issue 分配给 Agent 时：

1. 后端创建一个 Task，状态为 `enqueued`
2. 对应 Runtime 上的 Daemon 轮询到这个 Task，状态变为 `claimed`
3. Daemon 启动 Agent CLI（比如 Claude Code），传入 Issue 描述和上下文，状态变为 `running`
4. Agent 执行过程中，进度通过 WebSocket 实时回传
5. 执行完成后状态变为 `completed`，或者失败变为 `failed`
6. 如果 Agent 遇到无法自行解决的问题，它会主动报告 `blocked`，等待人工介入

这个状态机是 Multica 和"直接在终端跑 Claude Code"的核心区别。直接跑 CLI，你只能盯着一个终端窗口。Multica 给了你一个看板视图，所有 Agent 的所有任务一目了然。

### 实时通信：WebSocket Hub

图表

`realtime.Hub`（`server/internal/realtime/hub.go`）是整个实时系统的核心。它维护所有 WebSocket 连接，当后端状态发生变化时，Hub 负责把事件广播给所有相关的客户端。

这意味着：当 Agent A 在你的机器上执行任务时，你的同事在另一台电脑上也能实时看到进度。这是团队协作的基础。

### Skills 系统：可复用的 Agent 能力

Multica 的 Skills 系统让你把 Agent 学到的能力打包成可复用的模块。

一个 Skill 包含代码、配置和上下文——比如"部署到 staging 环境"、"写 SQL 迁移脚本"、"审查 PR"。一旦定义好，工作区里的任何 Agent 都可以执行这个 Skill。

这和 Hermes Agent 的自动技能生成不同。Multica 的 Skills 更像是团队知识库——人工定义、Agent 执行。好处是可控性强，坏处是需要人工维护。

## 与现有 Agent 的本质区别

这是理解 Multica 最重要的一点：它不在执行层竞争。

分层对比图：三个水平层级，顶层管理层Multica(蓝色)负责任务分配和进度追踪，中层编排层Kiro/Cursor(橙色)负责代码规划和Agent模式，底层执行层Claude Code/Codex/OpenClaw(绿色)负责实际代码编写，层间有向下箭头表示调用关系

现有的 AI 编程工具大致分三层：

**执行层**：Claude Code、Codex、OpenClaw、OpenCode。它们是真正写代码的 Agent——接收指令，操作文件，执行命令。

**编排层**：Kiro、Cursor、Windsurf。它们是 Agent 的工作环境——提供 IDE 界面、代码补全、Agent 模式、Spec 工作流。

**管理层**：Multica。它是 Agent 的项目经理——分配任务、追踪进度、协调多个 Agent、积累团队知识。

Multica 不替代 Claude Code，它管理 Claude Code。你可以同时有 3 个 Agent（一个跑 Claude Code，一个跑 Codex，一个跑 OpenClaw），Multica 统一管理它们的任务队列、执行状态和输出结果。

这个定位解释了为什么 Multica 的架构里有 Daemon 但没有 LLM 调用——它不需要自己调模型，它调度别人调模型。

## 对比：三种 Agent 协作模式

雷达图：五个维度对比Multica、Claude Code Agent Teams、直接CLI三种模式，Multica在任务管理(9.5)和进度可视化(9.5)维度接近满分，Claude Code Teams在多Agent协调(7)居中，直接CLI在上手门槛(9)最优但其他维度偏低，蓝色Multica橙色Teams绿色CLI

### 模式 1：直接用 CLI

最简单的方式。打开终端，跑 `claude` 或 `codex`，给它一个任务。

优点：零配置，即开即用。
缺点：一次只能盯一个 Agent，没有任务管理，没有进度追踪，没有历史记录。适合个人快速任务。

### 模式 2：Claude Code Agent Teams

Claude Code 自带的多 Agent 模式。一个 lead session 协调多个 teammate session，在同一个项目上并行工作。

优点：原生集成，不需要额外工具。
缺点：只支持 Claude Code（不能混用 Codex 或 OpenClaw），没有持久化的任务看板，session 结束后上下文丢失。适合单次复杂任务的并行拆分。

### 模式 3：Multica

完整的管理平台。看板视图、任务生命周期、多 Agent 多 Runtime、实时进度、Skills 复用。

优点：真正的团队级 Agent 管理，vendor-neutral（不绑定特定 Agent），持久化所有历史。
缺点：需要部署和配置，有学习成本。适合持续运行多个 Agent 的团队。

| 维度 | 直接 CLI | Claude Code Teams | Multica |
| --- | --- | --- | --- |
| 多 Agent 支持 | 手动开多终端 | 原生 lead-teammate | 统一看板管理 |
| Agent 类型 | 单一 | 仅 Claude Code | Claude/Codex/OpenClaw/OpenCode |
| 任务持久化 | 无 | Session 内 | 完整生命周期 |
| 进度可视化 | 终端输出 | 终端输出 | Web UI 实时看板 |
| 技能复用 | 无 | 无 | Skills 系统 |
| 部署成本 | 零 | 零 | Docker 或 Cloud |

## 使用场景与工作流

### 场景 1：夜间批量执行

下班前在 Multica 看板上创建 5 个 Issue，分配给不同的 Agent。设置好优先级和依赖关系。回家。

第二天早上打开 Multica，看到：3 个完成、1 个失败（附带错误日志）、1 个 blocked（Agent 报告需要数据库权限）。处理失败和 blocked 的任务，继续。

这个场景用直接 CLI 是做不到的——你不可能开 5 个终端窗口然后去睡觉。

### 场景 2：混合团队协作

团队有 3 个人类开发者和 2 个 Agent。在 Multica 的看板上，人类和 Agent 的任务在同一个视图里。

Sprint 规划时，把简单的重构任务分给 Agent，复杂的架构设计留给人类。Agent 完成后自动更新状态，人类在同一个 Activity Feed 里看到所有变更。

看板 Board

### 场景 3：Agent 技能积累

第一次让 Agent 做 staging 部署，你需要详细描述步骤。Agent 完成后，把这个流程打包成一个 Skill。

下次任何 Agent 需要做 staging 部署，直接调用这个 Skill。不需要重新描述，不需要重新学习。团队的 Agent 能力随时间积累。

## 局限与展望

**还很早期。** Multica 是一个新项目，社区规模和文档丰富度都还在起步阶段。生产环境使用需要评估风险。

**Agent 质量依赖底层工具。** Multica 管理 Agent，但 Agent 写出的代码质量取决于 Claude Code / Codex 本身。Multica 不能让一个差的 Agent 变好，它只能让好的 Agent 更有组织。

**自托管有门槛。** 需要 Docker、PostgreSQL、Go 环境。对于小团队来说，Cloud 托管版可能更实际。

**Skills 系统还比较基础。** 目前的 Skills 是人工定义的，没有 Hermes Agent 那样的自动学习能力。未来如果能结合 Agent 的执行历史自动生成 Skills，价值会大很多。

**值得期待的方向：**

pgvector 的存在暗示了语义搜索的可能——根据 Issue 描述自动匹配最合适的 Agent 和 Skill。多 Workspace 隔离已经实现，企业级的权限管理和审计日志是自然的下一步。

Multica 的核心洞察是对的：当 AI Agent 从"偶尔用一下的工具"变成"每天都在干活的队友"时，你需要一个系统来管理它们。这个需求只会越来越强。

## 结语

Multica 解决的不是"怎么让 AI 写代码"的问题，而是"怎么管理一群会写代码的 AI"的问题。

它的定位很清晰：不做执行层（那是 Claude Code / Codex 的事），不做编排层（那是 Kiro / Cursor 的事），专注做管理层。任务分配、进度追踪、多 Agent 协调、技能积累——这些是当你有多个 Agent 同时工作时真正需要的能力。

从源码看，架构设计是干净的：Go 后端 + WebSocket 实时通信 + 本地 Daemon 执行，每一层职责明确。vendor-neutral 的设计（支持四种 Agent CLI）避免了绑定特定供应商的风险。

如果你的团队已经在用 Claude Code 或 Codex，而且经常同时跑多个 Agent 任务，Multica 值得试试。它不会改变 Agent 写代码的方式，但会改变你管理 Agent 的方式。

你的下 10 个 hire，可能不是人类。但你仍然需要管理他们。

---

## 参考链接

* Multica GitHub 仓库 - multica-ai/multica[1]
* Multica 官方网站[2]
* Multica 产品介绍 - MOGE[3]
* Multica 架构深度分析 - DeepWiki[4]
* Multica: A Visual Desktop Client for Coding Agents - Jimmy Song[5]
* How to Run a Multi-Agent Coding Workspace (2026) - Augment Code[6]
* Claude Code Agent Teams: Complete Guide - AIFreeAPI[7]
* What makes multi-agent coding work - Addy Osmani[8]

---

**参考链接**

[1] https://github.com/multica-ai/multica

[2] https://multica.ai

[3] https://moge.ai/product/multica

[4] https://deepwiki.com/multica-ai/multica

[5] https://jimmysong.io/ai/multica/

[6] https://www.augmentcode.com/guides/how-to-run-a-multi-agent-coding-workspace

[7] https://www.aifreeapi.com/en/posts/claude-code-agent-teams

[8] https://addyosmani.com/blog/code-agent-orchestra/