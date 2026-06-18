---
title: GitButler：Git 的革命性开源神器！并行分支、无限撤销、AI 智能代理、零 rebase 烦恼，一文吃透全部功能+安装+源码编译
author: 如此才是
date: 如此才是如此才是
url: https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484946&idx=1&sn=1b7bd38fc2f4b0faccc30d6a837a654c&chksm=f5062fca852fac712583937c2682e338e5881fdd3f4cb135275ab49327f7d137c3a3e6645350&mpshare=1&scene=24&srcid=0426s4GKILeJmJwWGqmz8Y7N&sharer_shareinfo=56d3fc6198a2ee438609ac861975718c&sharer_shareinfo_first=56d3fc6198a2ee438609ac861975718c#rd
---

Git 的分支切换、rebase 地狱、无限 stash 和 AI 代理“乱改代码”着实让人头疼。

GitButler以开源姿态杀出重围，不只是简单的 Git GUI，而是**从零重构的 Git-backed 变更管理系统**，专为现代 AI 编码工作流设计。

支持**真正并行分支**、**堆叠分支**、**无限撤销时间线**、**拖拽式 Commit 编辑**、**Forge 集成**、**AI 自动生成**以及**代理安全集成**，让 Git 真正“但更好”（Git, but better）。

### 一、GitButler ：虚拟分支 + 并行工作流，彻底告别传统 Git 局限

传统 Git 一次只能有一个 HEAD 和 index，切换分支需 stash、rebase 风险高。GitButler 引入\*\*虚拟分支（Virtual Branches）概念：

●**并行分支（Parallel / Virtual Branches）**：在**单个工作目录**中同时应用多个独立分支。每条分支是一个垂直“lane”（泳道），文件变更像卡片一样**拖拽分配**到不同 lane，各 lane 有独立 staging area。

●**堆叠分支（Stacked Branches）**：依赖关系分支，形成 PR 链（bottom-up 审查）。GitButler 自动处理 restacking 和 stacked PRs。

GitButler 不依赖真实 Git branch 切换，而是维护一个集成分支（Integration Branch）来协调多分支。所有变更先在工作目录累积，再按 lane 计算“如果只有本分支变更”的文件树并 commit。确保分支最终干净合并（因为起始于同一工作目录的 merge product）。

如果用 `git switch` 等原生命令，GitButler 会暂停并提示返回其管理模式。

**并行分支视觉示例**：

**堆叠分支**：默认 parallel 为单分支 stack，点击 + 创建依赖分支。新 commit 落在栈顶，push 时自动创建 stacked PRs（每个 PR target 上一个分支）。GitHub 需开启“自动删除分支”以正确显示 diff。

**与并行分支对比**：并行=独立，堆叠=依赖，可混合使用（如无关 bugfix 移到独立 lane）。

### 二、Commit 管理：拖拽胜过 rebase -i

GitButler 让 Commit 操作变成**拖拽式**，无需 `git rebase -i`：

●**创建 Commit**：点击 “Start a Commit”，浮动编辑器支持 AI 生成消息（%{diff}、%{branch\_name} 等变量）。

●**吸收变更（Absorb）**：新修改拖进已有 Commit，自动 restack 上游。

●**Undo Commit**：展开 Commit 点击 Undo，变更变回 uncommitted（不丢弃）。

●**Uncommit 单文件**：展开后点击文件 “Uncommit”。

●**Squash**：拖一个 Commit 到另一个上方。

●**Split**：插入空 Commit，拖变更进去。

●**Reorder**：拖拽调整顺序，自动 rebase。

●**Edit Mode**：点击 “Edit commit” 临时 checkout 该 Commit，修改后 Save & Exit 自动 amend + restack。

**AI Commit 设置**：支持 OpenAI/Anthropic（直连或代理）、Ollama/LM Studio 本地模型。可自定义 prompt，支持 GitMoji、简短风格等。

**Commit 编辑视觉**：

### 三、无限 Undo 时间线：Git 对象数据库快照机制

每次重大操作前，GitButler 自动将**虚拟分支状态、uncommitted 工作、冲突状态**等完整快照存入**Git 对象数据库**。

●GUI：Operations History 标签页，hover 任意条目点击 “Revert” 恢复到操作前状态。

●可预览最近变更文件 diff（即使未 commit）。

●CLI 对应 operations-log。

这实现**真正无限撤销**，远超 Git reflog。

**时间线架构**：

### 四、冲突处理：Rebase 永不失败，先标记后解决

传统 Git rebase 一冲突就停。GitButler **总是成功 rebase**，将冲突 Commit 标记为 conflicted 状态。

●可随时 checkout 单个 conflicted Commit 解决。

●解决后自动 rebase 上游。

●支持任意顺序解决。

**冲突分支示例**：

### 五、Forge 集成：GitHub / GitLab 原生 PR 管理

支持多账号（Device Flow、PAT、Enterprise）。

●自动检测/创建/更新 PR。

●显示 CI 状态、mergeability。

●Stacked PRs 自动处理 target branch。

●GUI/CLI 均支持（`but` 命令）。

GitLab 支持类似（MR draft 等）。

### 六、AI 工具编排：代理安全 + 并行分支 Agents

GitButler 专为 AI 编码设计：

●**内置 Claude Code GUI**：Agents Tab 图形化运行 Claude。

●**并行分支-based Agents**：多个 Agent 工作在独立分支，上下文隔离。

●**任意 Agent 集成**：Cursor/Claude hooks，或 **Model Context Protocol (MCP) Server** 实现自动版本控制。

●**AI 生成**：Commit 消息、分支名、PR 描述。

●**Hooks / Skills**：`but skill install` 为 Agent 安装 Git 管理技能。

实验特性需开启 Global Settings → Experimental → GitButler Actions。

**AI 架构**：

### 七、安装方法（Prebuilt）

1.**GUI**：访问 https://gitbutler.com/downloads 下载最新安装包（0.19.9+ 支持 macOS/Windows/Linux）。

2.**CLI (`but`)**：从 GitHub Releases 下载，或使用 `but skill install`（Linux 支持已完整）。

3.安装后打开即在任意 Git repo 中工作（drop-in 替换）。

### 八、从源码安装 & 编译

GitButler 是 **Tauri 应用**：Rust 后端 + Svelte/TypeScript 前端。CLI 与 GUI 共享同一 Rust 引擎（crates/but）。

**前提**（macOS/Linux/Windows 通用）：

●Rust（rustup）

●Node 20+ + pnpm（corepack enable）

●Tauri 系统依赖（Linux: libwebkit2gtk 等，详见 Tauri 官网）

●CMake、XCode（macOS）

**步骤**（master 分支）：

```
●●●bash

git clone https://github.com/gitbutlerapp/gitbutler.git  
cd gitbutler  
corepack enable  
pnpm install  
cargo build          # 构建辅助 binary（如 gitbutler-git-askpass）  
pnpm dev:desktop     # 开发模式运行 GUI（LOG_LEVEL=debug 开启日志）
```

**CLI 单独开发**：

```
●●●bash

cargo run -p but -- --help  
cargo test -p but
```

**调试**：

●日志：stdout 或 Tauri logs 目录

●性能：`GITBUTLER_PERFORMANCE_LOG=1`

●Repo 可视化：在 GUI 输入 `dot`（需 graphviz）生成 commit-graph SVG

●Tokio console：`tokio-console`

**构建发布**：`pnpm tauri build`。

**架构概览**：

●Rust 后端：libgit2 操作 + 虚拟分支计算 + 操作日志

●前端：Svelte + TS（apps/）

●共享引擎：crates/ 下多个 crate（but 为 CLI 核心）

**架构图**：

### 九、使用方法快速上手（GUI + CLI）

**GUI**：

●打开 repo → 自动识别 target branch（通常 origin/main）

●开始修改 → 拖拽到 lanes → Commit / AI 生成

●右键 Commit 操作一切

**CLI 示例**（`but`）：

●`but status`：查看并行分支状态

●`but branch create <name>`：创建虚拟分支

●`but commit`：提交当前 lane

●`but push`：推送 + 创建 PR

●`but tui`：文本界面（0.19+ 大幅优化）

**Git 的进化**：保留 Git 兼容性，同时用虚拟分支、快照、AI 彻底解放生产力。

**—— 如此才是**

**把复杂的技术，讲成你真正能用上的生产力**

**[零基础也能玩转卫星！开源Ground Station + SDR 打造个人地面站全攻略](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484408&idx=1&sn=fa96368ff3647cd53bad3ee9391103ee&scene=21#wechat_redirect)**

**[OpenClaw & Hermes刷屏后，GitHub  Mercury Agent如何打动用户？ 灵魂驱动+权限铁闸+24/7永动 vs 两大竞品](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484903&idx=1&sn=8ea193b342d5f61ed5ed30de9ebb32b9&scene=21#wechat_redirect)**

**[苹果M系列芯片的福音！无需H100、无需云GPU，本地MacBook就能微调Gemma 4多模态模型](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484529&idx=1&sn=63e2f7f4ac65540fd05ef9d05f0d28a8&scene=21#wechat_redirect)**

**[163个AI工具塞进Godot，solo游戏开发者效率直接起飞！15刀搞定爆款游戏](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484384&idx=1&sn=0cb741021a027dbacf774fcbfd8096aa&scene=21#wechat_redirect)**

**[开源Minecraft终极杀手！12.7K星GitHub神器Luanti（原Minetest）完整中文攻略：零基础安装、2800+模组随便玩、服务器+源码编译](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484515&idx=1&sn=f7cac21fab871c06cee81d780a1e37af&scene=21#wechat_redirect)**

**[AI 直接操控 Unity/Godot/Unreal 编辑器！用 OpenClaw + TomLeeLive 插件，聊天就能把你的游戏梦想变成现实](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484263&idx=1&sn=49474c84d4c0c6a1dd7925d821680aca&scene=21#wechat_redirect)**

[Rust开源AI Agent安全基座LoongClaw正式开源：7-crate严格DAG内核+L0-L9分层治理，团队垂域智能体终于有生产级“底座”了！](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484610&idx=1&sn=f2f6278fd6f6dd280a62038fb620c512&scene=21#wechat_redirect)

[开源项目Paseo，AI编码代理跨设备统一指挥中心：统管Claude Code、Codex、OpenCode（以及Copilot、Pi等）](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484927&idx=1&sn=b4d7d4aed5a5ad7263bff54b50c395a5&scene=21#wechat_redirect)

[Notebook LM平替，开源Open Notebook：隐私零泄露、18+AI模型随意切、1-4人定制播客秒生成](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484913&idx=1&sn=a3307c1fb6b981881b22ca1c1ca407e2&scene=21#wechat_redirect)