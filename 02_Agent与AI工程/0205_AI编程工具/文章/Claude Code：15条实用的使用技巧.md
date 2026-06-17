---
title: Claude Code：15条实用的使用技巧
author: 神奇犸鲸Magic
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4NTE0NzMzMA==&mid=2660583577&idx=1&sn=c59310a544d68092f5ac2c0975edb8d7&chksm=8552d1eed31ac35ceeff64ed133f5e6bdbab654593a79c3cad3d22f39655f8c24fb028a651ef&mpshare=1&scene=24&srcid=0410DXiF1A6ep8ylYmAU1VXW&sharer_shareinfo=6d057f0d574988dd61886c6a0ffdca21&sharer_shareinfo_first=6d057f0d574988dd61886c6a0ffdca21#rd
---

你好，我是神奇犸鲸Magic

写了10年读书笔记，读书和旅行只是开始，来一起构建属于自己的人生系统。

---

**点击蓝字**

**星标犸(Ma)鲸**

本文由 Claude Code 团队的Boris Cherny撰写：

我一直想和大家分享我在 Claude Code 中最喜爱的一些隐藏功能和尚未被充分挖掘的特性。

我会重点介绍自己日常使用频率最高的那些。

以下是详细内容：

1. 你知道 Claude Code 有移动应用吗？

我个人大量代码都是在 iOS 应用上编写的。它是一种无需打开笔记本电脑就能快速修改代码的便捷方式。

下载 Claude 应用（iOS / Android），在左侧菜单进入“Code”标签页即可使用。

2. 在移动端、网页端、桌面端和终端之间自由迁移会话

运行 claude --teleport 或输入 /teleport，即可将云端会话无缝延续到本地机器。或者输入 /remote-control，从手机或网页远程控制本地运行的会话。我个人已在 /config 中开启了“为所有会话启用远程控制”选项。

3. Claude Code 最强大的两大功能：/loop 和 /schedule

使用这两个指令，可以让 Claude 按设定间隔自动运行，最长可持续一周。  
我本地运行了多个循环任务，例如：  

* /loop 5m /babysit：自动处理代码审查、自动 rebase，并将 PR 推向生产环境；
* /loop 30m /slack-feedback：每 30 分钟自动发起 Slack 反馈 PR；
* /loop /post-merge-sweeper：为我遗漏的代码审查意见自动创建 PR；
* /loop 1h /pr-pruner：关闭陈旧且不再需要的 PR。  
  还有更多！  
  建议大家尝试将工作流转化为技能（skills）+ 循环（loops），威力惊人。

4. 使用 hooks 在 Agent 生命周期中确定性地执行逻辑

例如，通过 hooks 可以实现：  

* 每次启动 Claude 时动态加载上下文（SessionStart）；
* 记录模型运行的每一条 bash 命令（PreToolUse）；
* 将权限请求路由到 WhatsApp 供你批准/拒绝（PermissionRequest）；
* 当 Claude 停止时自动唤醒它继续（Stop）。  
  更多细节可参考官方文档。

5. Cowork Dispatch

我每天都在使用 Dispatch 来跟进 Slack 和邮件、管理文件，以及在不在电脑前时远程操作笔记本。

Dispatch 是 Claude Desktop 应用的 secure 远程控制功能，经你授权后可使用你的 MCPs、浏览器和计算机。

6. 使用 Chrome 扩展进行前端开发

使用 Claude Code 的最重要建议是：给 Claude 提供验证输出的途径。一旦做到这一点，Claude 就会不断迭代，直到结果优秀。

就像任何工程师一样：如果你要求别人建网站却不让他用浏览器，结果会好看吗？几乎不可能。但给他浏览器后，他就会编写代码并迭代到完美。

我每次处理 Web 代码时都使用 Chrome 扩展，它比其他类似 MCP 更稳定可靠。Chrome / Edge 扩展下载地址见官方链接。

7. 使用 Claude Desktop 应用让 Claude 自动启动并测试 Web 服务器

同理，Desktop 应用内置了让 Claude 自动运行 Web 服务器并在内置浏览器中测试的能力。

你也可以在 CLI 或 VS Code 中通过 Chrome 扩展实现类似功能，但 Desktop 应用最便捷。

8. 分支（Fork）你的会话

很多人问如何从现有会话创建分支，有两种方法：  

1. 在会话中运行 /branch；
2. 在 CLI 中运行 claude --resume <session-id> --fork-session。

9. 使用 /btw 处理侧边查询

我在 Agent 工作期间经常用这个指令快速提问。

10. 使用 Git worktrees

Claude Code 对 Git worktrees 提供了深度支持，这是同时处理同一仓库中大量并行工作的关键。我同时运行着数十个 Claude 会话，全靠 worktrees 实现。

使用 claude -w 在 worktree 中启动新会话，或在 Claude Desktop 应用中勾选“worktree”复选框。

对于非 Git VCS 用户，可通过 WorktreeCreate hook 添加自定义逻辑。  
更多信息见官方文档。

11. 使用 /batch 批量处理大规模变更

/batch 会先询问你需求，然后让 Claude 将工作分发到所需数量的 worktree Agent（几十、几百甚至上千个）并行完成。

适用于大型代码迁移等可并行化任务。

12. 使用 --bare 标志将 SDK 启动速度提升高达 10 倍

默认情况下，运行 claude -p（或 TypeScript / Python SDK）时会自动搜索本地 CLAUDE.md、设置和 MCPs。

但对于非交互式场景，通常希望通过 --system-prompt、--mcp-config、--settings 等参数显式指定加载内容。

这是 SDK 早期设计的一个小疏忽，未来版本将默认改为 --bare。目前请手动使用该标志。

13. 使用 --add-dir 让 Claude 访问更多文件夹

跨多个仓库工作时，我通常在一个仓库启动 Claude，然后通过 --add-dir（或 /add-dir）让它看到其他仓库。这不仅告知 Claude 仓库信息，还授予其操作权限。

或者在团队的 settings.json 中添加 “additionalDirectories” 配置，让 Claude Code 启动时自动加载额外文件夹。

14. 使用 --agent 自定义系统提示词和工具

自定义 Agent 是强大却常被忽略的基础功能。  
在 .claude/agents 目录下定义新 Agent，然后运行 claude --agent=<你的 Agent 名称> 即可。更多文档见官方子 Agent 说明。

15. 使用 /voice 开启语音输入

有趣的事实：我大部分代码都是通过和 Claude “说话”而非打字完成的。

在 CLI 中运行 /voice，然后按住空格键；或在 Desktop 应用中按语音按钮；或在 iOS 设置中开启听写功能即可。希望这些内容对你有用！我本来还想继续分享，但还是先打住。以后有机会再发更多。  

你的最爱且被低估的 Claude Code 功能是什么？ 欢迎在评论区分享！

-END-

我是Magic，和你一起慢慢变富。

野生商学院是Magic主导的关于信息筛选的内容库，无论是中药材和生姜，还是羽毛球上下游，其实价格和趋势就存在在信息的伏脉千里之中。

欢迎你一起构建系统~

#Claude #claudecode #人生系统 ##提前退休 ##财务自由 #vibecoding #攒钱 #AI #广州 #深圳 #上海 #人工智能 #AIcoding