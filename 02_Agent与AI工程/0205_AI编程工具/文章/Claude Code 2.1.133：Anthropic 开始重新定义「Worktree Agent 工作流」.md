---
title: Claude Code 2.1.133：Anthropic 开始重新定义「Worktree Agent 工作流」
author: 晓明兄
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzMTE0MTQ5MQ==&mid=2247485611&idx=1&sn=af5aff6abf7a07cc7c46c9a41b088d23&chksm=e9dd18f5d3e7e9477a8839538076e0ea67567f4d47af6af373614f1a7d374fecb21b58e9cf40&mpshare=1&scene=24&srcid=0509fUwAG9mNamgkyFbyvkh0&sharer_shareinfo=8f7834941f39b4cef7448dd7c5256122&sharer_shareinfo_first=8f7834941f39b4cef7448dd7c5256122#rd
---

Claude Code 的更新节奏，已经越来越清晰了。

不是那种“大版本一锤子砸下来，然后长期沉默”的风格，而是**大改动 → 小步快跑 → 回归修复 → 工作流再调整 → 持续推进**的迭代路径。

2.1.133 就是非常典型的一版。

表面看，这次 Changelog 不长，也不像 2.1.132 那样有一堆终端大招。但真正值得重视的是：

**Anthropic 又一次改动了 worktree 的默认行为，而且是“改回去”。**

这通常意味着，他们在真实用户环境中，已经观察到了 Agent workflow 的明显分歧。

### 本次最核心的改动：worktree.baseRef

新增配置项：

```
worktree.baseRef: fresh | head
```

用于控制 `--worktree`、 `EnterWorktree`、agent-isolation worktree 从哪里创建分支。

* • **fresh**（新默认）：从 `origin/<default-branch>` 创建，**干净上下文**，不继承本地未推送 commit。
* • **head**：从本地 HEAD 创建，继承当前所有本地状态（包括未 push commit、脏工作区、实验代码）。

**重点**：从 2.1.128 开始，默认一直是 `head`（继承本地 HEAD），而 2.1.133 又改回了 `fresh`。如果你想保持之前行为，需要手动设置为 `"head"`。

### 表面是 Git 配置，背后是哲学之争

**AI Agent 到底应该继承“人类当前上下文”，还是使用“干净上下文”？**

很多人第一反应是：当然要继承当前状态啊，这样 Agent 才懂我在干什么。

但在真实工程环境中，这往往很危险。

当前工作区通常处于**不稳定状态**：半成品 patch、临时 debug、本地 hack、未提交实验、rebase 残留……

人类知道哪些是临时的，但 Agent 不知道。它会基于“脏上下文”继续推理，导致 patch 越修越歪、分支爆炸、状态不一致。

最终问题不是 Agent 不聪明，而是**上下文被污染了**。

### Anthropic 的选择：把上下文隔离提到优先级

通过把默认改回 `fresh`，Anthropic 明确表达了立场：

**Agent isolation 和 Runtime 一致性，比单纯继承当前状态更重要。**

这已经不是简单的 Git 问题，而是 **Agent Runtime 的边界治理问题**。

### 这和 Cursor 等产品的路线差异越来越明显

* • **Cursor** 更像 IDE 增强层：强绑定当前编辑器状态、人类工作流融合。
* • **Claude Code** 越来越像**独立 Agent Runtime 系统**：强调 isolation、可重复性、deterministic workflow、sandbox。

这很像当年 Docker 取代“直接在宿主机跑”的演进——**环境治理**变得至关重要。

### 为什么现在突然重要？

因为 Claude Code 的使用场景已经彻底变了：

从“问一句、生成一点代码” → **多 Agent 并行、多 worktree、多 session、长时间运行、Multi-Claudeing**。

在多 Agent 时代，如果默认继承 HEAD，上下文污染和互相干扰的问题会指数级放大。

**fresh** 模式带来的优势非常明显：

* • 更高的可重复性（相同基线）
* • 更好的可审计、可 replay、可 rollback
* • 更适合企业级管控

### 小结：Claude Code 正在往哪里走？

2.1.133 表面只是一个 worktree 配置调整，但它清晰地传递了一个信号：

**Anthropic 正在从“更聪明的代码助手”，转向“更稳定的 AI 软件工程系统”。**

真正决定成败的，从来不是模型单次生成能力有多强，而是：

* • 上下文管理能力
* • Runtime 生命周期控制
* • 多 Agent 协调
* • Session 可恢复性
* • Agent 隔离与环境治理

最近几版（2.1.129 插件临时安装 → 2.1.132 终端与 Session ID → 2.1.133 worktree 调整）串起来看，这条路线已经非常明确。

**升级建议**：

1. 1. 执行 `claude --update` 升级到 2.1.133
2. 2. 重启后用 `/version` 确认
3. 3. 根据你的工作流决定是否设置 `worktree.baseRef: "head"`

你更喜欢干净的 fresh 模式，还是继承本地状态的 head 模式？欢迎在评论区分享你的看法和实际使用感受。