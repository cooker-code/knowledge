> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 现在能操作你的电脑了
author: 嗨清单
date:
url: https://mp.weixin.qq.com/s?__biz=MzY5OTExMTQyNQ==&mid=2247484548&idx=1&sn=d1fdd712c21914addb412d7d3100f81e&chksm=f54fce4e93cb3bdde184e3fe650248f66529cbb8a773d808ad1e28dd7c5f497724a3dbeba3c5&mpshare=1&scene=24&srcid=0401ORTJYjBreFQTLlY939TX&sharer_shareinfo=8331d6c7d20f33e50b0b676035e0d1b8&sharer_shareinfo_first=8331d6c7d20f33e50b0b676035e0d1b8#rd
---

Claude Code 更新了一个让我兴奋的功能：**Computer Use** —— Claude 可以直接操作你的电脑了。打开应用、点击按钮、输入文字、截图验证，全部在终端会话里完成。

这不是概念验证，不是 demo。这是一个可以在日常工作中直接用的功能。我实际体验了一段时间，说说感受。

PART 01它能做什么

一句话：**Claude 能看到你的屏幕，能操作你的鼠标和键盘。**

听起来简单，但这意味着 Claude 不再被困在终端里。过去，遇到需要 GUI 操作的任务，你得自己切出去手动完成。现在不用了。

具体能干什么：

* **构建并验证原生应用** —— 让 Claude 写一个 macOS 菜单栏应用，它会自己编译、启动、逐个点击每个控件，验证没有崩溃，然后截图给你看结果
* **端到端 UI 测试** —— 指着你的 Electron 应用说"测一下注册流程"，Claude 打开应用、点击注册、填写表单、截图每一步。不需要配置 Playwright，不需要写测试
* **调试视觉问题** —— "小窗口下弹窗被截断了"，Claude 自己调整窗口大小复现 bug、截图、改 CSS、验证修复
* **操作纯 GUI 工具** —— 设计软件、硬件控制面板、iOS 模拟器，那些没有命令行也没有 API 的工具，Claude 都能直接操作

关键变化

以前 Claude Code 的边界是终端。现在它的边界是你的整个桌面。写代码 → 编译 → 启动 → 交互验证，一个会话完成闭环。

PART 02三步开启

设置非常简单，不需要安装额外的东西。

第一步：打开 MCP 菜单

在 Claude Code 会话中输入：

/mcp

找到 `computer-use`，它默认是关闭的。选中并启用。

第二步：授权 macOS 权限

Claude 第一次尝试操作屏幕时，会请求两个系统权限：

| 权限 | 用途 |
| --- | --- |
| 辅助功能（Accessibility） | 让 Claude 能点击、输入、滚动 |
| 屏幕录制（Screen Recording） | 让 Claude 能看到屏幕内容 |

授权后可能需要重启 Claude Code。这是 macOS 的常规操作，不是 bug。

第三步：直接开始用

# 比如让它验证你刚写的应用

Build the app, launch it, and click through each tab to make sure nothing crashes. Screenshot any error states.

每个项目只需设置一次。下次打开同一项目，computer-use 自动就是启用状态。

PART 03它是怎么工作的

理解工作机制，才能用好它。

智能工具选择

Claude 不会上来就抢你的屏幕。它有一套优先级：

1**MCP 服务器** —— 如果任务对应的服务有 MCP，优先用 MCP

2**Shell 命令** —— 能用命令行解决的，用命令行

3**Chrome 集成** —— 浏览器任务用 Claude in Chrome

4**Computer Use** —— 以上都不行时，才用屏幕控制

这个设计很聪明。屏幕控制是最慢但覆盖面最广的方式，只用在真正需要的地方。

工作时会发生什么

当 Claude 开始操作屏幕：

* **其他窗口自动隐藏**，Claude 只和你批准的应用交互
* **终端窗口始终可见**，但被排除在截图之外 —— Claude 永远看不到自己的输出
* **macOS 通知提示** "Claude is using your computer"
* **操作完成后**，隐藏的窗口自动恢复

全程你可以在终端里观察 Claude 在做什么。如果觉得不对，随时按 **Esc** 中止，或者在终端里按 **Ctrl+C**。

PART 04安全机制设计得很细

让 AI 控制电脑，安全是第一反应。Anthropic 在这方面想了很多。

逐应用授权

启用 computer-use **不等于** Claude 能操作你电脑上所有应用。每次 Claude 需要某个应用时，都会先弹出确认：

* Claude 想控制哪些应用
* 是否需要额外权限（比如剪贴板）
* 工作期间哪些应用会被隐藏

你选 "Allow for this session" 或 "Deny"。授权只在当前会话有效，关了再开需要重新确认。

高风险应用会额外警告

有些应用权限很大，Claude 会在确认对话框里加一行警告：

| 警告 | 适用应用 |
| --- | --- |
| 等同于 Shell 权限 | 终端、iTerm、VS Code、Warp 等 |
| 可读写任意文件 | Finder |
| 可修改系统设置 | 系统设置 |

不是阻止你用，而是确保你知道自己在授权什么。此外，浏览器和交易平台只有只读权限，终端和 IDE 只有点击权限，其他应用才有完整控制权限。分级很清晰。

安全底线

终端窗口永远被排除在截图外（Claude 看不到自己的输出），全局 Esc 键随时中止，同一时间只有一个会话能操作屏幕。这些都是不可配置的硬性限制。

PART 05三个让我真正觉得有用的场景

场景一：写完代码直接验证

这是最自然的用法。以前的工作流是：让 Claude 写代码 → 自己编译 → 自己打开应用 → 自己点击测试 → 回到终端描述问题。

现在直接说：

Build the MenuBarStats target, launch it, open the preferences window, and verify the interval slider updates the label. Screenshot when done.

Claude 自己 `xcodebuild`、启动应用、打开偏好设置、拖动滑块、验证标签变了、截图。**整个闭环在一个会话里完成**，你甚至不需要切出终端。

场景二：复现只能用眼看的 Bug

有些 Bug 只在特定条件下出现，比如窗口很小的时候弹窗被截断。以前你得自己复现、截图、然后在终端里用文字描述给 Claude。

现在：

The settings modal clips its footer on narrow windows. Resize the app window to reproduce it, screenshot, then fix the CSS.

Claude 自己缩小窗口、复现 Bug、截图留证、检查 CSS、改代码、再验证。**它看到的和你看到的是同一个东西**，不需要来回翻译。

场景三：驱动 iOS 模拟器

不用写 XCTest，不用配置任何测试框架：

Open the iOS Simulator, launch the app, tap through the onboarding screens, and tell me if any screen takes more than a second to load.

Claude 操作模拟器的方式和你用鼠标点一模一样。对于快速验证来说，这比写自动化测试效率高得多。

PART 06当前限制

作为 research preview，有几个限制需要知道：

| 限制 | 说明 |
| --- | --- |
| 仅 macOS | Windows 和 Linux 暂不支持 |
| 需要 Pro 或 Max 订阅 | Team 和 Enterprise 暂不可用 |
| 需要 v2.1.85+ | 用 `claude --version` 检查版本 |
| 仅交互模式 | 不能用 `-p` 非交互模式 |
| 仅 claude.ai 账户 | 通过 Bedrock、Vertex 等第三方接入的不支持 |

Computer Use 对 Claude Code 的意义，不在于它能做多少花哨的事。而在于它补上了最后一块拼图：**视觉验证**。

以前，Claude 写代码很强，但它是「盲」的 —— 它看不到代码运行后长什么样。这导致 UI 相关的工作始终需要人来回切换、截图描述、反复沟通。

现在，Claude 能看了。它写完代码可以自己编译、自己打开、自己看效果、自己改。**从「写代码」变成了「做产品」**。

如果你是 Claude Code 用户，而且在 Mac 上，建议现在就去 `/mcp` 里打开它试试。