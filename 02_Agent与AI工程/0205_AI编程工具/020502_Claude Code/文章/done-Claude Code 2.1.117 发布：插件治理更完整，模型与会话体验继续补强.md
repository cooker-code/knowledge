> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 2.1.117 发布：插件治理更完整，模型与会话体验继续补强
author: 阿俊聊AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzYzODc1NzU3Ng==&mid=2247483950&idx=1&sn=e08eb18ee071e355de31d3a27dbd8064&chksm=f150f987fac5c4e818ce4d816ae8b1619ac23d19fb299e31f4dd31e16d4c3eaae05492dd4231&mpshare=1&scene=24&srcid=0422V1X1j9w9KIOaqVl0nRBM&sharer_shareinfo=e3c7798a7b3199ed71da26cb5211619e&sharer_shareinfo_first=e3c7798a7b3199ed71da26cb5211619e#rd
---

一句话看这版：**这次更新覆盖面很广，但主线很清楚：一边继续打磨模型选择、会话恢复和启动速度这些高频环节，一边把插件依赖治理、受管设置约束，以及一批长期存在的细节问题补得更完整。**

---

## 这次更新最值得关注的几个点

### 1. 模型选择和大会话恢复更顺手了

2.1.117 继续补强了两个高频场景：模型切换和大会话恢复。

官方确认的变化包括：

* `/model` 的选择现在会跨重启保留；即使项目 pin 了不同模型，用户自己的选择也能持续保留
* 启动头部现在会显示当前激活模型是否来自 project pin 或 managed-settings pin
* `/resume` 现在会在重新读取陈旧且体量较大的 session 之前，先提供摘要选项，这与现有 `--resume` 行为保持一致

**价值：** 这些变化都在减少“为什么这次又不一样”的不确定感。模型来源更清楚了，大 session 恢复时也多了一条更省时间的处理路径。

---

### 2. 插件安装、依赖补全和 Marketplace 限制更完整了

插件相关是这次更新里很集中的一块。

官方确认：

* 对已经安装过的插件再次执行 `plugin install` 时，不会停在“already installed”，而是会继续安装缺失依赖
* 插件依赖报错现在会明确显示 “not installed”，并给出安装提示
* `claude plugin marketplace add` 现在会从已配置的 marketplaces 中自动解析缺失依赖
* managed-settings 的 `blockedMarketplaces` 和 `strictKnownMarketplaces`，现在会在插件安装、更新、刷新和自动更新时统一生效

**价值：** 这意味着插件体系的行为更完整，也更一致。对个人用户来说，少了一些“插件装上了但依赖没补全”的中断；对团队和受管环境来说，Marketplace 约束终于覆盖到了更多实际入口。

---

### 3. Agent、MCP 和启动速度继续优化

这版也继续在 Agent 和 MCP 路径上补细节、提速度。

官方确认：

* 外部构建现在可以通过设置 `CLAUDE_CODE_FORK_SUBAGENT=1` 启用 forked subagents
* 通过 `--agent` 启动主线程 agent session 时，现在也会加载 Agent frontmatter 里的 `mcpServers`
* 当同时配置本地和 claude.ai MCP server 时，启动更快了；并发连接现在默认开启
* `cleanupPeriodDays` 的保留清理范围，现已覆盖 `~/.claude/tasks/`、`~/.claude/shell-snapshots/` 和 `~/.claude/backups/`

**价值：** 一部分提升的是能力覆盖范围，另一部分提升的是进入可用状态的速度。对 Agent 和 MCP 配置比较重的用户来说，这类更新会直接影响日常使用的顺滑程度。

---

### 4. 可观测性、本地构建和默认配置也有明确调整

除了交互和插件，这版还更新了一些底层能力与默认行为。

官方确认：

* OpenTelemetry 中，`user_prompt` 事件现在会为 slash commands 带上 `command_name` 和 `command_source`
* `cost.usage`、`token.usage`、`api_request`、`api_error` 在模型支持 effort levels 时，现会带上 `effort` 属性
* 除非设置 `OTEL_LOG_TOOL_DETAILS=1`，否则 Custom/MCP command names 会被脱敏
* 在 macOS 和 Linux 的原生构建中，Glob 和 Grep 工具被替换为可通过 Bash 工具使用的嵌入式 `bfs` 与 `ugrep`
* Windows 现在会按进程缓存 `where.exe` 的可执行文件查找结果，以加快子进程启动
* Pro/Max 订阅用户在 Opus 4.6 和 Sonnet 4.6 上的默认 effort 现在是 high，此前为 medium

**价值：** 这些变化虽然不都直接体现在界面上，但会影响诊断信息、默认推理强度，以及部分平台上的启动和搜索性能。

---

### 5. Advisor Tool 的实验性标识和报错问题被补齐了

官方把 Advisor Tool（experimental）这一块也补得更完整了：

* 对话里现在会明确显示 “experimental” 标签
* 启用时会显示 learn-more 链接和启动通知
* session 不会再在每次 prompt 和 `/compact` 时都卡在 “Advisor tool result content could not be processed” 错误上

**价值：** 实验功能最怕状态不清和报错打断。这次更新至少把标识和一类高频错误处理补齐了。

---

## 这次还修了哪些值得注意的问题？

### 认证、登录与网络请求

* 修复 Plain-CLI OAuth session 在 access token 中途过期后报 “Please run /login” 并失效的问题；现在会在收到 401 后主动刷新 token
* 修复使用 `CLAUDE_CODE_OAUTH_TOKEN` 环境变量启动时，若 token 过期，`/login` 不生效的问题
* 修复在 Bun 环境下，远程 API 请求不遵守 `NO_PROXY` 的问题
* 修复代理返回 HTTP 204 No Content 时触发崩溃的问题；现在会给出明确错误，而不是 TypeError

---

### 输入、终端与交互细节

* 修复 prompt 输入框里 `Ctrl+_` 撤销在刚输入后无效、且每次撤销会跳过一个状态的问题
* 修复在慢连接下，按键名以合并文本形式到达时，偶发误触发 escape/return 的问题
* 修复后台任务存在时的空闲重渲染循环，降低 Linux 上的内存增长

---

### 插件、MCP 与 SDK 行为

* 修复 SDK `reload_plugins` 串行重连所有用户 MCP servers 的问题
* 修复 MCP `elicitation/create` 请求在 print/SDK 模式下，当 server 在本轮中途完成连接时被自动取消的问题
* 修复 subagents 使用与主 agent 不同模型时，文件读取被错误标记为 malware warning 的问题
* 修复 VS Code 中在配置多个大型 marketplaces 时，“Manage Plugins” 面板异常的问题

---

### 模型与上下文计算

* 修复 Bedrock application-inference-profile 请求在底层使用 Opus 4.7 且关闭 thinking 时返回 400 的问题
* 修复 Opus 4.7 session 的 `/context` 百分比被高估、并因此过早触发 autocompact 的问题；此前 Claude Code 错按 200K context window 计算，而不是 Opus 4.7 原生 1M

---

### 抓取与页面处理

* 修复 WebFetch 在超大 HTML 页面上卡住的问题；现在会在 HTML 转 Markdown 前先截断输入

---

## 这版更新最大的用户价值是什么？

### 1. 高频工作流又少了一些阻塞点

无论是 `/model` 持久化、`/resume` 的摘要选项，还是 MCP 并发连接默认开启，核心都是让日常高频动作更少卡顿。

### 2. 插件体系终于更像一个完整系统

安装、依赖、报错提示、Marketplace 约束，这次都被拉到了同一条线上。

### 3. 团队和受管环境更容易控边界

`blockedMarketplaces` 和 `strictKnownMarketplaces` 的执行范围扩大后，插件治理不再只停留在部分入口。

### 4. 底层诊断信息更完整，平台行为更清楚

OpenTelemetry 字段补充、默认 effort 调整，以及不同平台上的性能改进，都让系统行为更可见，也更稳定。

### 5. 一批细碎但真实影响体验的问题被继续清掉了

OAuth 过期、WebFetch 卡住、输入撤销异常、上下文计算偏差，这些都不是新功能，但都会直接影响日常使用质量。

---

## 怎么看 Claude Code 2.1.117？

**2.1.117 不是那种只靠一两个新入口吸引注意力的版本。**

它更像是一次面向日常使用质量的系统性补强：

* 模型选择和会话恢复更清楚、更顺手
* 插件安装、依赖处理和 Marketplace 治理更完整
* Agent 与 MCP 的接入和启动路径继续优化
* 认证、输入、抓取、上下文计算等基础问题继续收口

如果你已经把 Claude Code 用在高频工作流里，这一版的价值会比较直接：不是“花样更多了”，而是很多真实会打断使用的小问题，又少了一批。