---
title: cmux：专为 Claude Code 而生的终端工作区
author: 智能思辨录
date: 
url: https://mp.weixin.qq.com/s?__biz=MzY5NzIwOTg4MQ==&mid=2247483682&idx=1&sn=c1c93df5e58c393639f4475d19fbab99&chksm=f5dd1bc4e88138573dd663596b154b09446da81a245d2ecc271f179f84a308c5b493ce202b19&mpshare=1&scene=24&srcid=0328W83oVVt5uAuMbgj2wcdi&sharer_shareinfo=d8000f231d10729c63daa3659428e39a&sharer_shareinfo_first=d8000f231d10729c63daa3659428e39a#rd
---

从一个真实的困境说起

如果你已经重度使用 Claude Code 一段时间，应该有过这样的体验：

同时跑着三四个 Agent，每隔几分钟就要逐个切过去确认进度，屏幕上堆满了 iTerm2 的 Tab。某个 Agent 发来一条系统通知，内容是 "Claude is waiting for your input"——你完全不知道它在等什么、遇到了什么问题，只能再切过去看。想让 Agent 帮你顺手查个文档、确认一下页面效果，它没有眼睛，你还得自己截图、复制内容喂给它。

这些问题不是 Claude Code 本身的问题，而是传统终端压根没有为 AI Agent 设计。iTerm2 非常好用，但它是为人类开发者设计的，不是为了管理多个并发运行的 AI Agent 设计的。

cmux 是专门解决这个问题的工具。

 

cmux 是什么

cmux 是由 manaflow-ai 开源（AGPL-3.0，GitHub 4.5k ⭐）的原生 macOS 终端工作区管理器，基于 libghostty 构建，Swift + AppKit 开发，GPU 加速渲染，不是 Electron。

准确说，它不只是"终端模拟器"，而是专门为 AI 编码 Agent 设计的工作区管理器。Claude Code 在里面是"一等公民"，而不是"插件"。

理解 cmux 的定位，先看它和 iTerm2 的本质差异：

 

| 维度 | iTerm2 | cmux |
| --- | --- | --- |
| 定位 | 终端模拟器 | 工作区管理器 |
| 技术栈 | Obj-C + Metal | Swift + AppKit + libghostty（GPU 加速） |
| CLI 控制 | 有限（AppleScript） | 完整 Unix Socket API |
| 内置浏览器 | 无 | 有，且 Agent 可编程控制 DOM |
| 多 Agent 感知 | 无 | 侧边栏实时显示所有工作区状态 |
| 通知系统 | 通用系统通知 | Pane 高亮环 + 未读徽章 + 上下文内容 |
| Claude Code 集成 | 无 | 原生 Hooks、状态栏、进度条、日志 |
| 开源 | 否 | 是（AGPL-3.0） |

 

 

核心概念：工作区层级结构

理解 cmux 先要理解它的四层结构：

Window（macOS 原生窗口）

└── Workspace（工作区 ≈ iTerm2 的 Tab，但功能更强）

    └── Pane（分屏区域，可上下左右任意分割）

        └── Surface（Pane 内的 Tab，可以是终端或浏览器）

 

• Workspace 是核心单元，每个工作区有独立的侧边栏状态卡、日志、通知系统

• Surface 是最小执行单元，支持两种类型：terminal 和 browser

• 每个 cmux 终端内自动注入环境变量 $CMUX\_WORKSPACE\_ID 和 $CMUX\_SURFACE\_ID，命令默认作用于当前工作区，无需硬编码 ID

如果你习惯了 iTerm2：把 Workspace 理解成更强大的 Tab，Pane 理解成分屏，Surface 理解成分屏内还可以再切 Tab。

 

安装

brew tap manaflow-ai/cmux

brew install --cask cmux

 

如果你已经在用 Ghostty，更省事——cmux 直接读取 ~/.config/ghostty/config，主题、字体、颜色开箱即用，零迁移成本。cmux 通过 Sparkle 自动更新，装一次不用管了。

 

核心能力一：让你一眼看清所有 Agent 的状态

打开 cmux 之后，左侧有一个侧边栏。这是 cmux 相比 iTerm2 最直观的差异所在。

侧边栏的每个 Workspace 都有一张信息卡，实时显示：

• 当前 git 分支名称

• 关联 PR 的状态和编号

• 工作目录的监听端口（如 :3000、:8080）

• 工作区的最新通知内容（自定义文本）

 

四个 Agent 同时在跑？你不需要逐个切过去——打开 cmux，扫一眼侧边栏，Frontend Agent 在 feat/auth 分支、监听 3000 端口；Backend Agent 在 main、监听 8080；Test Agent 上次通知说"全部通过"。全局状态一目了然。

这种感知能力在单个 Agent 场景下就很有用，在多 Agent 并行时更是不可替代。

 

核心能力二：真正有上下文的通知系统

传统终端的 AI Agent 通知，信噪比极低。"Claude is waiting for your input" 这条通知什么都没说——它在等什么？遇到了什么问题？你不得不切过去才能知道。

cmux 的通知系统重新设计了这个体验。当某个 Pane 里的 Agent 需要你回应时：

• Pane 外圈亮起蓝色高亮环，视觉上立刻区分"在工作中"和"需要你介入"

• 侧边栏对应的工作区条目出现未读徽章，同时弹出 Popover 显示具体内容

• macOS 桌面通知照常发出，内容是你自定义的文本而不是废话模板

 

这些通知通过标准终端转义序列（OSC 9/99/777）自动触发，无需额外配置。Cmd+Shift+U 一键跳到最新未读的 Pane，同时跑五个 Agent 也不用逐个点击。

你也可以在 Agent 的任意节点主动触发通知：

cmux notify --title "构建完成" --body "PR #42 已合并，prod 部署中"

 

核心能力三：Claude Code 生命周期 Hooks

通知系统解决了"被动感知"的问题，Hooks 解决了"主动汇报"的问题。

cmux 原生支持 Claude Code 的 Hooks 系统，让 Claude 的每个关键动作都能反映在 UI 上。配置一次，永久生效：

// ~/.claude/settings.json

{

  "hooks": {

    "PreToolUse": [{

      "matcher": "",

      "hooks": [{

        "type": "command",

        "command": "cmux set-status claude\_status '⚡ 工作中' --icon activity --color '#f59e0b'"

      }]

    }],

    "Stop": [{

      "hooks": [{

        "type": "command",

        "command": "cmux clear-status claude\_status && cmux notify --title 'Claude 完成' --body '任务已完成'"

      }]

    }]

  }

}

 

效果：Claude 开始调用工具时，侧边栏自动出现"⚡ 工作中"；任务完成后自动清除状态并弹出通知。你不需要盯着屏幕，等通知就够了。

除了状态文本，还支持实时进度条——让 Agent 在处理大任务时主动汇报进度：

# Agent 在处理过程中主动调用

cmux set-progress 0.4 --label "分析文件 2/5"

cmux set-progress 0.8 --label "分析文件 4/5"

cmux clear-progress   # 完成后清除

 

侧边栏会出现一条动态进度条。处理上百个文件的任务时，你能随时知道跑到哪里了，而不是盯着滚动的输出猜测。

 

核心能力四：Agent 可以直接操控内置浏览器

这是 cmux 最让我意外的功能，也是和任何传统终端本质差异最大的地方。

cmux 内置了基于 Chromium 的浏览器面板，关键不在于"内置"，而在于：Agent 可以通过 CLI 直接编程控制它。

# 打开页面

cmux browser open https://localhost:3000

 

# 截图（Agent 可以"看到"当前页面效果）

cmux browser screenshot --out /tmp/page.png

 

# 获取页面文本（直接作为 Agent 的上下文输入）

cmux browser get text

 

# 获取 Accessibility Tree（结构化 DOM 信息）

cmux browser get accessibility-tree

 

# 点击元素、填写表单

cmux browser click --selector "#submit-button"

cmux browser fill --selector "#search" --value "cmux tutorial"

 

# 执行 JavaScript

cmux browser eval "document.querySelector('h1').textContent"

 

这意味着什么？

想象一个前端开发场景：

Agent 修改 CSS → 调用 cmux browser screenshot 截图 → 看到效果不对 → 调用 cmux browser get accessibility-tree 审查 DOM 结构 → 定位到问题 → 修复代码 → 再截图确认。全程闭环，Agent 自主完成。

你以前需要截图、粘贴进去、用自然语言描述问题——现在这个环节消失了。

另外，cmux 还支持 Markdown 实时预览（cmux markdown open ~/project/README.md）。Claude 修改文档时，右侧面板同步渲染最新效果，Agent 通过截图直接验证排版，完全不用切换 App。

 

核心能力五：多 Agent 并行编排

前面说的都是单个 Agent 的体验提升。cmux 最强大的能力，在于它为多个 Agent 之间的协作提供了基础设施。

核心机制很简单，就三个 CLI 命令：

# 列出所有工作区，了解全局状态

cmux list-workspaces

 

# 读取另一个工作区的终端输出

cmux read-screen --workspace workspace:3 --scrollback --lines 50

 

# 向另一个工作区发送指令并执行

cmux send --workspace workspace:3 "请分析 src/auth/ 并输出结果\n"

 

基于这三个原语，可以搭建真正意义上的 Orchestrator 模式：

主 Agent（Orchestrator） — workspace:1

├── Frontend Agent — workspace:2  → 负责 UI 组件重构

├── Backend Agent  — workspace:3  → 负责 API 接口实现

└── Test Agent     — workspace:4  → 负责测试用例编写

 

主 Agent 通过 cmux send 分发任务给各子 Agent，通过 cmux read-screen 定期轮询子 Agent 的输出，汇总结果、协调冲突，任务完成后清理工作区。子 Agent 之间不需要直接通信，通过主 Agent 中转。

启动这样一套环境的脚本

# 创建 Frontend Agent 工作区并分配任务

cmux new-workspace --cwd ~/projects/my-app

cmux rename-workspace "Agent: Frontend"

cmux send --workspace workspace:2 "claude\n"

cmux send --workspace workspace:2 "请重构 src/components/Auth.tsx\n"

 

# 创建 Backend Agent 工作区并分配任务

cmux new-workspace --cwd ~/projects/my-app

cmux rename-workspace "Agent: Backend"

cmux send --workspace workspace:3 "claude\n"

cmux send --workspace workspace:3 "请实现 POST /api/auth/login，返回 JWT token\n"

 

# 30 秒后读取进度

sleep 30

cmux read-screen --workspace workspace:2 --scrollback --lines 30

cmux read-screen --workspace workspace:3 --scrollback --lines 30

 

cmux 还提供了原生多 Agent 协调命令 cmux claude-teams，适合在同一仓库内需要多个 Claude Code 实例高度协作的场景。

关于 Session 持久化：cmux 工作区本身是临时的，但 Claude Code Session 是持久的。推荐将每个工作区映射到一个 Session 文件，配合 claude --resume 恢复，用 Obsidian Bases 等工具建立 Dashboard 追踪所有 Agent 的状态和目标。

 

核心能力六：完整的 CLI 控制 API

cmux 的 CLI 通过 Unix Socket（/tmp/cmux.sock）与 GUI 通信，延迟极低。在 cmux 终端内运行时自动绑定当前工作区，不需要每次都指定 --workspace 参数。这意味着 Claude Code 在某个工作区内运行时，调用 cmux 命令默认就是在操作自己所在的工作区。

除了前面介绍的命令，还有一些实用的：

# 查看当前工作区完整结构（树状）

cmux tree

 

# 把终端输出持久化到文件（类似 tmux pipe-pane）

cmux pipe-pane --command "cat >> /tmp/session-$(date +%Y%m%d).log"

 

# 等待某个信号（tmux 兼容语法）

cmux wait-for --signal build-done --timeout 60

 

# 记录操作日志（可分级、分来源）

cmux log --level info --source claude "开始重构 auth 模块"

cmux log --level warn --source claude "发现潜在的 SQL 注入风险"

cmux list-log --limit 20

 

这套 CLI API 是 cmux 作为"AI Agent 工作区"的基础能力。Agent 可以主动汇报状态、读取环境信息、控制其他工作区——这些在传统终端里完全做不到。

 

进阶：cmuxlayer —— 通过 MCP 控制 cmux

如果你想让 Agent 更流畅地控制 cmux，社区还提供了 cmuxlayer，一个通过 MCP 协议控制 cmux 的 MCP Server。

相比直接调用 CLI，MCP 方式使用持久 socket 连接，速度约快 1423 倍，且对 Agent 更友好——不需要 shell 命令，直接通过结构化的工具调用操作。

git clone https://github.com/EtanHey/cmuxlayer.git && cd cmuxlayer

bun install && bun run build

 

核心能力：

• read\_screen：读取终端内容，自动解析 Claude Code / Codex / Gemini CLI 的结构化状态（无需手动解析输出）

• Agent 生命周期管理：spawn、monitor、teardown

• 支持自主模式和手动控制模式两种策略

 

项目级配置：一键启动标准工作区

对于长期维护的项目，推荐在项目根目录创建 .vscode/terminals.json，定义标准工作区布局：

{

  "cmux": [{

    "name": "My App",

    "tabs": [

      { "name": "Claude Code", "type": "terminal", "commands": ["claude"] },

      { "name": "Dev Server",  "type": "terminal", "commands": ["make dev"] },

      { "name": "Preview",     "type": "browser",  "url": "http://localhost:3000" }

    ]

  }]

}

 

make cmux 即可一键还原完整开发环境：左侧 Claude Code、右侧 Dev Server、底部浏览器预览。换台电脑、换个团队成员，工作区配置完全一致。

同样推荐在项目的 CLAUDE.md 中加入 cmux 使用说明节，明确告知 Agent 可以调用哪些命令、每个工作区对应什么职责：

## cmux Terminal Interaction

 

检查可用性：command -v cmux &>/dev/null

 

重要：cmux 命令只能在启动 cmux 的原始终端窗口中运行，

不要在 cmux 创建的子终端（Claude 会话、应用服务器）中调用。

 

Available surfaces:

- Backend: run `make dev-backend`

- Frontend: run `make dev-frontend`

- Tests: run `make test`

 

使用 cmux read-screen 在重启服务前检查终端状态。

使用 cmux set-progress 和 cmux log 汇报长任务进度。

使用 cmux notify 在任务完成时通知用户。

 

几个需要注意的地方

沙盒兼容问题。 Claude Code 默认在沙盒模式下运行，可能无法访问 /tmp/cmux.sock，导致 cmux 命令无响应。遇到这种情况，需要在受信任项目中禁用沙盒限制，或在 Claude Code 项目设置中明确允许 socket 访问。这是已知问题，cmux 团队正在推进更优雅的解决方案。

cmux 命令只能在原始窗口执行。 cmux CLI 必须在启动 cmux 的那个终端窗口中运行，不能在 cmux 自身创建的子终端中执行。这个约束一定要在 CLAUDE.md 中写清楚，否则 Agent 会尝试从错误的上下文调用命令。

重启不恢复进程状态。 工作区布局重启后会保留，但终端进程（Claude Code session、运行中的服务）需要手动重新启动。claude --resume 可以恢复 Claude Code 的上下文。

Surface ID 是动态的。 surface:3 这样的 ID 每次重启都会变化，脚本里不要硬编码。用 cmux tree 查看当前 ID，或通过名称间接引用。

 

总结

cmux 解决的不是"用什么终端"的问题，而是"怎么把 AI Agent 真正整合进开发工作流"的问题。它提供了五个层面的能力：

 

1.  感知层：侧边栏状态卡让你一眼看清所有 Agent 的状态，不需要逐个切换

2.  通知层：蓝色 Pane 高亮环 + 自定义通知内容，告别无意义的 "Claude is waiting"

3.  控制层：完整的 CLI API 让 Agent 可以主动汇报状态、控制 UI、读取环境

4.  感官层：内置浏览器 + Agent 可编程控制，让 Agent 能"看到"并操作页面

5.  协作层：list、read、send 三个原语，构建真正的多 Agent 并行编排

 

如果你每天用 Claude Code，这五个能力加在一起，体验差异是量级的。