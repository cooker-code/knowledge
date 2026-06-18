> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: 给 Hermes Agent 装一个指挥中心：Hermes Workspace 项目实测
author: 极客BIM设计工坊
date: 朗朗晴空朗朗晴空
url: https://mp.weixin.qq.com/s?__biz=MzI2MjA3ODk0OQ==&mid=2648116613&idx=1&sn=4210b103be94428ab54e5fa84c7bfbb9&chksm=f3e6877c2537153d52af6da2867a366f5faf179e054a0b2d5d4fd90aa64b89dbcd77ff50f7fd&mpshare=1&scene=24&srcid=0504D1TY61UKu1UrzrePRyXF&sharer_shareinfo=a4020c5942dc39de36420f517c66e864&sharer_shareinfo_first=a4020c5942dc39de36420f517c66e864#rd
---

KEY TAKEAWAY

Hermes Workspace 是 NousResearch 出品的 Hermes Agent 的官方配套工作区——把聊天、终端、内存浏览、技能管理、多代理调度全部塞进一个 Web 界面。不是聊天壳子，是一个完整的代理控制中心。

2.8k

GitHub Stars

v2.1.3

最新版本

128k

母项目 Stars

小白先看懂：这个项目是干什么的

Hermes Agent 是 NousResearch 做的开源个人代理框架——你可以把它理解成一个"住在你服务器上的助手"，能读文件、跑命令、记记忆、用技能。但 Hermes Agent 本身只有命令行界面，没有图形化操作面板。

Hermes Workspace 就是给它装的"指挥中心"——一个 Web 界面，让你能在一个页面里看到代理在干什么、它的记忆里有什么、它装了哪些技能、它的终端在跑什么命令。还支持多个代理同时调度（Swarm Mode），像一个作战室。

●　**适合谁** — 已经在用或准备用 Hermes Agent 的开发者、需要多代理协作的团队、想要可视化管理代理状态的技术用户

●　**不适合谁** — 不用 Hermes Agent 的人（它不是独立产品）、只想在网页上聊天的普通用户、没有 Node.js 环境的非技术用户

●　**核心问题** — 解决的是"Hermes Agent 没有可视化界面"的问题，让代理的状态、记忆、技能、任务全部透明化

项目基本面

仓库信息

语言：TypeScript

协议：MIT

创建：2026-03-16

最近推送：2026-05-01

社区数据

Stars：2,823

Forks：329

Open Issues：6

贡献者：5 人（主导 outsourc-e）

母项目 Hermes Agent 在 GitHub 上有 128,591 颗 Star，是 2026 年最受关注的开源代理框架之一。Hermes Workspace 作为其官方配套 UI，从 3 月上线至今不到两个月，增长速度很快。MIT 协议意味着你可以自由使用、修改、商用。

核心功能

▸　**实时聊天** — 通过 SSE 流式连接 Hermes Agent，实时看到代理的输出和工具调用过程

▸　**终端集成** — 内嵌终端，可以直接在界面里看代理跑的命令和输出

▸　**内存浏览器** — 浏览、搜索、编辑代理的三层记忆系统，不用翻文件

▸　**技能管理** — 查看和管理代理的 2000+ 技能，支持安装、编辑、搜索

▸　**Swarm Mode** — 多代理调度：一个编排器 + 无限工作代理，支持 Kanban 任务板、角色路由、检查点审查

▸　**8 套主题** — Official、Classic、Slate、Mono 等，每套有亮色和暗色变体

▸　**移动端 PWA** — 通过 Tailscale 在手机上也能用，功能完整

▸　**桌面应用** — 提供 macOS 原生桌面版（约 529 MB），拖入 Applications 即可安装

安装：两条路径

**路径一：一键安装（推荐）**
一条命令搞定 Hermes Agent + Workspace + 依赖：

```
curl -fsSL https://raw.githubusercontent.com/outsourc-e/hermes-workspace/main/install.sh | bash
```

**路径二：已有 Hermes Agent，只装 Workspace**
如果你已经通过 pip 或 Docker 装了 Hermes Agent，只需要克隆仓库、配置 .env 指向现有服务：

```
git clone https://github.com/outsourc-e/hermes-workspace.git
cd hermes-workspace && pnpm install
cp .env.example .env
# 填入 HERMES_API_URL 和 HERMES_DASHBOARD_URL
pnpm dev
```

前置要求：Node.js ≥ 22、pnpm。桌面版直接下载 dmg 安装包，不需要 Node 环境。

开发阶段与活跃度

项目处于**快速迭代期**。v2.0.0 于 2026-04-20 发布（Zero-fork 架构重构），v2.1.3 于 2026-05-01 发布，11 天内迭代了 3 个版本。最近一周有 5 个 commit 被合并，包括安全更新系统、MCP 服务器管理页面、桌面更新流程等。Open Issues 仅 6 个，维护者响应速度快。

▸　**v2.0 架构重构** — 从 fork 模式改为 zero-fork，直接对接原版 Hermes Agent，不再需要自定义网关

▸　**Swarm Mode 上线** — 多代理编排、Kanban 任务板、角色路由、检查点审查

▸　**桌面版发布** — macOS 原生应用已可用（截图证实：529.3 MB，标准拖拽安装）

截图核实

**已核实**　用户提供的截图显示 macOS 系统正在将「Hermes Workspace」应用复制到 Applications 文件夹，进度 529.3 MB 中的 343.7 MB。这是标准的 macOS 桌面应用安装流程，与 GitHub 仓库中的桌面版发布信息一致。桌面版真实存在，非虚构。

竞品位置

Hermes Workspace 不是通用的 AI 聊天界面，它是 Hermes Agent 的专属工作区。如果把 Hermes Agent 比作发动机，Workspace 就是仪表盘。它的竞品不是 ChatGPT 网页版，而是直接跑 CLI、用 Hermes 自带的 `hermes web` 命令、或者自己搭前端调 API 这三种替代方案。

▸　**vs 直接跑 CLI** — Workspace 的优势是可视化：能看到内存、技能、任务状态，不用翻文件

▸　**vs Hermes 自带 web UI** — Workspace 功能更完整：Swarm Mode、Kanban、主题系统、桌面版

▸　**vs 自己搭前端** — Workspace 开箱即用，zero-fork 架构不需要维护自定义网关

试用建议

●　**已经是 Hermes Agent 用户** — 直接装，zero-fork 架构不侵入现有部署，5 分钟能跑起来

●　**准备入门 Hermes Agent** — 用一键安装脚本，Agent + Workspace 一起装，省去分别配置的麻烦

●　**想试多代理协作** — Swarm Mode 是亮点，但需要先熟悉单代理工作流，建议先跑通基础功能再上 Swarm

●　**非技术用户** — 暂时不建议折腾。项目依赖 Node.js 环境，配置需要改 .env 文件，虽然有桌面版但仍需先理解 Hermes Agent 的基本概念

SOURCES

GitHub 仓库 — github.com/outsourc-e/hermes-workspace

Hermes Agent — github.com/NousResearch/hermes-agent (128k Stars)

Release v2.1.3 — 2026-05-01

用户截图核实：macOS 桌面版安装进程（529.3 MB）