> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code Agent View：一个界面管所有AI编程会话
author: 石臻说AI
date: 石臻石臻
url: https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500639&idx=1&sn=cb0075103e926d95deb3cb67f6da70ce&chksm=ce50a1d76e04b2f29387f8d7f96f73efe0fb06dc55a891c94f74333854d3fe098600570277b0&mpshare=1&scene=24&srcid=0512I4iBLoxeJBLn4o4o0c1N&sharer_shareinfo=35c4d609002f5284ec1e8e14f718297f&sharer_shareinfo_first=d3a273a962b413f83b150ca37bc8bfeb#rd
---

⭐ 设为星标 · 第一时间收到推送

石臻说AI
编辑：石臻

**导读：** Anthropic 昨天给 Claude Code 加了一个杀手级功能 —— **Agent View**。一句话概括：你现在可以在一个终端界面里同时派发、监控和管理多个 Claude Code 后台会话，不用再开一堆终端 Tab 了。

Claude Code 的开发者 Thariq 在推特上说得很直白："Agent view is the best Claude Code native way to manage multiple sessions, kind of like tmux built for CC."

## 它解决了什么痛点？

用过 Claude Code 并行跑任务的人一定体会过这种崩溃：

* 开了 5-6 个终端 Tab，每个跑一个 `claude` 会话
* tmux 分屏后根本记不清哪个窗格在干嘛
* 某个会话早就在等你确认权限，你却没注意到
* 关掉终端全没了，上下文丢失

Agent View 把这个问题彻底解决了：**一个命令 `claude agents`，所有后台会话的状态、进度一目了然，需要你操作的自动排最前面。**

## 动态演示

先看一段官方演示视频转 GIF，感受下实际操作（38 秒完整视频见文末路径）：

## 快速上手：5 分钟跑起来

### 前置条件

* Claude Code 版本 ≥ **v2.1.139**
* 支持计划：Pro / Max / Team / Enterprise / API
* 检查版本：`claude --version`，低了就 `claude update`

### 第一步：打开 Agent View

```
claude agents
```

终端会被全屏接管，底部是输入框，上方是会话列表（初始为空）。按 `Esc` 退出，后台会话不会中断。

### 第二步：派发你的第一个任务

在底部输入框直接打字，按 `Enter` 发射：

```
重构 src/utils 目录，把重复代码抽成公共函数
```

会话立刻出现在列表中，开始干活。你可以继续派发：

```
审查 PR #42 的代码变更
```

```
给用户模块加上单元测试
```

每个任务独立运行，互不干扰。

### 第三步：Peek 快速回复

用 `↑` `↓` 选中一个会话，按 **Space** 打开预览面板（Peek）：

* 看到它正在做什么
* 如果在等你回答（比如「选方案 A 还是 B？」），直接输入回复按 Enter
* **不需要离开 Agent View**

这就像微信消息弹窗——不用进完整聊天就能快速回复。

### 第四步：深入参与（Attach）

某个任务需要你仔细看？按 **Enter** 或 **→** 进入完整会话，和平时用 `claude` 一模一样。

想回来？在空输入框按 **←** 脱离（detach），会话继续跑。

## 三种后台化姿势

除了在 Agent View 里直接派发，还有几种方式把任务送到后台：

**从已有会话后台化：**

```
/bg
# 或者带个后续指令
/bg 跑完测试套件后自动修复失败的用例
```

**从终端直接启动后台任务：**

```
claude --bg "调查 flaky test 的原因"
```

**用指定 Agent 启动：**

```
claude --agent code-reviewer --bg "审查 PR 1234 的评论"
```

启动后 Claude 会打印 session ID 和管理命令：

```
backgrounded · 7c5dcf5d
  claude agents             list sessions
  claude attach 7c5dcf5d    open in this terminal
  claude logs 7c5dcf5d      show recent output
  claude stop 7c5dcf5d      stop this session
```

## 状态图标：一眼看清所有会话

Agent View 列表里每个会话前面的图标同时传达两个信息：**运行状态**和**进程是否还活着**。

| 图标 | 状态 | 含义 |
| --- | --- | --- |
| ✽ 动画 | Working | Claude 正在执行工具或生成回复 |
| ✻ 黄色 | Needs Input | 等你操作（确认权限/回答问题） |
| ✻ 暗淡 | Idle | 等待输入但没有具体问题 |
| ✻ 绿色 | Completed | 任务完成 |
| ✻ 红色 | Failed | 出错了 |
| ∙ 灰色 | Stopped | 进程已退出但可恢复 |

一个实用细节：**关掉终端不会丢会话**。后台有个 supervisor 进程托管着它们，重新打开 `claude agents` 就能看到。机器休眠后用 `claude respawn --all` 恢复。

## 快捷键速查表

| 快捷键 | 功能 |
| --- | --- |
| `Space` | 打开/关闭预览面板 |
| `Enter` | 进入会话（或派发输入框中的任务） |
| `←`（空输入框） | 脱离当前会话返回列表 |
| `Shift+Enter` | 派发并立即进入 |
| `Alt+1`~`Alt+9` | 直接跳到第 N 个会话 |
| `Ctrl+S` | 切换分组方式（按状态/按目录） |
| `Ctrl+T` | 置顶/取消置顶 |
| `Ctrl+R` | 重命名会话 |
| `Ctrl+X` | 停止会话（2 秒内再按删除） |
| `?` | 显示全部快捷键 |

## 文件编辑隔离：多会话不冲突

后台会话需要改文件时，Claude 会自动创建 **git worktree**（在 `.claude/worktrees/` 下）。每个会话有自己的工作副本，多个会话并行改代码不会互相踩脚。

要保留修改，记得在删除会话前 merge 或 push。

## 从终端直接管理

不想开 Agent View 也能管：

```
claude agents          # 列出所有后台会话
claude attach <id>     # 进入某个会话
claude logs <id>       # 看日志
claude stop <id>       # 停掉
claude respawn --all   # 机器重启后恢复全部
```

## 实际用法参考

Anthropic 的开发者 Thariq 分享了自己的使用心得：

早期用户总结的几种高效模式：

1. 1**批量派发 + PR 集中审核**：一口气派 5 个独立任务，每个配一个 skill/subagent，最后统一审核 PR
2. 2**长期运行 Agent**：PR 监控、仪表盘更新等循环任务，列表里显示下次运行倒计时
3. 3**临时切换上下文**：正在做一个任务，按 `←` 临时开个快速问题，搞定后 `→` 回来继续
4. 4**快速扫成果**：状态指示器 + Peek 标题，一眼扫完哪些会话已经出了 PR

## 注意事项

* 每个后台会话**独立消耗配额**，同时跑太多会很快用完
* 会话摘要由 Haiku 模型生成（约 15 秒刷新），额外消耗少量 token
* 目前是 **Research Preview**，快捷键和界面可能会改
* 管理员可通过 `disableAgentView` 设置禁用

---

Claude Code 终于从「单线程工具」进化成了「多 Agent 调度中心」。说白了就是：以前你是单核处理器，现在你有了一个任务管理器来调度所有核心。

对于日常需要同时处理多个编程任务的开发者来说，这可能是 Claude Code 今年最实用的更新之一。赶紧 `claude update` 升级试试。

**核心要点：** `claude agents` 一个命令打开多会话管理界面。支持 Peek 快速回复、Attach 深入对话、后台派发并行任务。自动 git worktree 隔离写代码不冲突。需要 v2.1.139+，Research Preview 阶段。

参考链接

* Claude Code 官方博客：https://claude.com/blog/agent-view-in-claude-code
* Agent View 官方文档：https://code.claude.com/docs/en/agent-view
* Claude 官方推文：https://x.com/claudeai/status/2053940934736228454

📚 往期精选

[Claude Code 35个实战技巧：每条都有具体命令](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500482&idx=1&sn=929a6424e5c44beec77478184d0b1327&scene=21#wechat_redirect)

[这个开源项目能将任何文件一键转换成MarkDown](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500253&idx=1&sn=5064ed2bb504746f0c6b6c3649aec747&scene=21#wechat_redirect)

[Claude Code 上下文管理，会不会用这个命令就能判定你是不是高手](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500403&idx=1&sn=3eeaeb1a85fdc1509ed26d4adb738b5d&scene=21#wechat_redirect)

[爆火提示词：让AI帮你做色彩和穿搭分析](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500515&idx=1&sn=de4ad1b66f1db62da55d44526d25123d&scene=21#wechat_redirect)

[Claude Code 推送通知上线，编程进入不用管模式](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500585&idx=1&sn=9efb46a496876802a7881f253cbe7929&scene=21#wechat_redirect)

[Claude Code偷偷把缓存缩水被抓包，用户成本飙升， 这个操作能让你少烧一半配额](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500372&idx=1&sn=7dbfe8ceb885d5542c01526e648b6f4f&scene=21#wechat_redirect)

[如何构建正确的架构：瘦Harness，胖Skills](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500389&idx=1&sn=39d3c19a98dbb224ee5df985ea2dc939&scene=21#wechat_redirect)

[你存进去的知识，为什么一条都没用上？Obsidian + Claude 这样让知识库自己变聪明](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500620&idx=1&sn=2e22136768099c11cfa78868b56f7000&scene=21#wechat_redirect)

[别急着卸载 OpenClaw——这套组合拳，让它真正帮你干活](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500611&idx=1&sn=7a213c9f379f01e2cac4160899050bc3&scene=21#wechat_redirect)

[谁能拒绝这么一个Claude桌面宠物？](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500439&idx=1&sn=ee58bef194d0421ef980f55d941d45bb&scene=21#wechat_redirect)

---

— **完** —

**围观朋友圈查看每日最前沿AI资讯**

**一键关注 👇 点亮星标**

**每日科技资讯和提效工具分享**

本文章由快发 api.kuaifa.art 自动排版