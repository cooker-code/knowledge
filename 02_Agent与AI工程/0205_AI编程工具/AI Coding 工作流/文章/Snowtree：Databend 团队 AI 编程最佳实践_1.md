---
title: Snowtree：Databend 团队 AI 编程最佳实践
author: Databend
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg4NzYzMzk1Mw==&mid=2247497011&idx=1&sn=d7f5959652e630b9dc3420c139965d6b&chksm=ce80278d3321596b4f93c7dcbbeddc6d8bc872d0a19f37d0c8b4774c05d2b9c115a76124fb98&mpshare=1&scene=24&srcid=020353Wy8YgYfrrJk83Cz81A&sharer_shareinfo=077ff2b8665a5b3845507f9f37d57914&sharer_shareinfo_first=077ff2b8665a5b3845507f9f37d57914#rd
---

## 为什么开发 Snowtree？

Databend 是一个开源的现代云数仓，集 Analytics、Search、AI 于一体。项目包含 11,216 个 Rust 文件，代码规模超过 190 万行。作为数据库核心系统，它对代码的准确性、性能和稳定性有着非常高的要求。

在 Databend 团队全面转向 AI 辅助开发的过程中，我们尝试了市面上主流的 AI 编程工具，发现它们很难满足这种**严肃场景**的需求：

* **维护性与代码 Review**：系统逻辑错综复杂，AI 容易写出破坏隐式约束的代码。当 AI 一次性修改大量文件时，全量代码 Review 非常耗时且容易遗漏。而在多轮对话中，AI 可能会覆盖掉之前已 Review 通过的代码，导致 Review 工作前功尽弃。缺乏严格的增量代码 Review 机制，大量 AI 代码会迅速积累技术债务。
* **开发工具不统一**：团队成员使用的 AI 模型和 IDE 各异，导致代码风格和开发流程难以统一。
* **工具能力受限**：市面上的 AI Code Agent IDE 大多是基于大模型自行开发 Agent 能力，而非直接集成模型厂商官方的 Agent。这导致官方 Agent 的核心能力（如 Claude 的 MCP/Skills）无法直接使用，且能力更新总是滞后于官方。

我们需要一个这样的工具：既能让 AI 尽情发挥，又能通过严格的 Review 让人类保持掌控；能抹平工具差异，为团队提供一致的开发体验；直接集成大模型厂商官方 AI Code Agent CLI 不丢失能力；且深度融合 GitHub 工作流。

于是我们开发并开源了 Snowtree ——把 Databend 团队的 AI 编程最佳实践做成了工具。

## Snowtree 名称的由来

名字来源于 Git 的 **worktree** 功能。

以前大家很少用 worktree，因为 git branch 就够了——迭代速度没那么快，一个分支一个分支来就行。但有了 AI 之后，情况变了：你可以同时让多个 AI 并行开发不同的功能，迭代速度大幅提升。这时候，worktree 就成了天然的隔离机制——每个 AI 任务一个独立的工作目录，互不干扰。

Snowtree 为每个 AI 会话创建独立的 worktree，就像雪花一样——每一片都独立存在，互不干扰，最终融入主干。

Snow + Worktree = **Snowtree**。

---

## 三步使用

**第一步：让 AI 发挥**

Snowtree 为 AI 创建独立的工作环境。AI 可以自由修改代码，不影响你的主仓库。

**第二步：你来 Review**

AI 完成后，所有变更自动展示。逐行代码 Review，认可的保留，不满意的丢弃让 AI 重写。

**第三步：安全迭代**

每次暂存都是一个快照。后续 AI 怎么改，已暂存的代码都不会被覆盖。全部 OK 后，一键提交、创建 PR。

---

## 原生 CLI 集成

Snowtree 直接调用 Claude Code、Codex、Gemini CLI 的官方命令行，不做二次封装。

这三款 CLI 都支持 MCP，可以连接数据库、API 等外部工具。比如Claude Code 的 Skills 能力——你配置的一切，在 Snowtree 中同样生效。

更重要的是：这些 Agent 推出的任何新能力，你都可以无缝使用，无需等待 Snowtree 更新。

---

## 快速开始

**安装 Snowtree**：

```
curl -fsSL https://raw.githubusercontent.com/databendlabs/snowtree/main/install.sh | sh
```

**安装 AI Agent（任选其一）**：

| Agent | 命令 |
| --- | --- |
| Claude Code | `npm install -g @anthropic-ai/claude-code` |
| Codex | `npm install -g @openai/codex` |
| Gemini CLI | `npm install -g @google/gemini-cli` |

---

## 适合谁？

Snowtree 汇总了 Databend 团队在 AI 编程过程中的工作流经验。如果你有类似的诉求，可以试试：

* 严肃的生产项目，代码质量不能妥协
* 需要多人协作、基于 GitHub 的团队
* 大型代码库，AI 改动量大，需要系统化的审查机制

---

## 感谢

非常感谢 **Zed** 和 **OpenCode** 等优秀产品带来的理念启发。Snowtree 参考了 Zed 优秀的 Diff Review 交互模式，同时也借鉴了 OpenCode TUI 的极简设计理念。这让 Databend 团队在使用 AI 编程时，也能获得纯粹、高效的编码体验。

* GitHub: databendlabs/snowtree

关于 Databend

Databend 是一款 100% Rust 构建、面向对象存储设计的新一代开源云原生数据仓库，统一支持 BI 分析、AI 向量、全文检索及地理空间分析等多模态能力。

期待您的关注，一起打造新一代开源 AI + Data Cloud。

👨‍💻‍ Databend Cloud：https://databend.cn

📖 Databend 文档：https://docs.databend.cn

💻Wechat：Databend

✨GitHub：https://github.com/databendlabs/databend