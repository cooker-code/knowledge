> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Nezha 开源：给 Claude Code 和 Codex 准备的 7MB 桌面工作台
author: AI工具教程
date: AI工具教程AI工具教程
url: https://mp.weixin.qq.com/s?__biz=Mzg3NjgxNjkxMA==&mid=2247497449&idx=1&sn=4a3ce1364287d891f6c32025f0a09061&chksm=ce7fde92c66560be9eaff5d029f5066ac81e1d7c5e7852f27d2cc38cc986b325fe3c10acc5ed&mpshare=1&scene=24&srcid=0512CANUeUPfW2pl7QlksKpR&sharer_shareinfo=b5855ee5b49c364ca555124fc4541e81&sharer_shareinfo_first=b5855ee5b49c364ca555124fc4541e81#rd
---

我现在最怕的不是 AI 写不动代码。

是它们都在写。

一个 Claude Code 跑前端，一个 Codex 改脚本，旁边还有老项目在修 bug。终端开三四个，编辑器切来切去，Git 面板再单独挂着。看起来很忙，实际一半时间都在找：刚才那个 Agent 跑到哪了？

Nezha 这个项目，就是冲着这个痛点来的。

它不是再造一个大 IDE，而是把 Claude Code、Codex、终端、任务状态、会话记录、代码浏览和 Git 流程塞进一个桌面应用里。项目介绍里写得很直白：让多个 Claude Code 和 Codex agent 可以在本机、跨项目并行跑。

这个点挺戳人。

以前我们用 AI Coding 工具，很多时候还是“人等 AI”。发一个任务，等它跑完，再开下一个。Nezha 的思路更像是反过来：人负责盯盘，Agent 负责干活。

左边切项目，任务还在后台跑。

某个任务卡住、需要确认，它会把状态标出来。做完之后，还能回看和恢复会话，不用在一堆终端历史里翻半天。

我比较喜欢的是，它没有把自己包装成“下一代万能 IDE”。

它内置了轻量代码编辑器和 Markdown 编辑器，也有文件树、语法高亮、Git 状态标记。够你检查 Agent 改了什么，但不是逼你把整套开发习惯都迁过去。

Git 这块也做得比较实用。

可以看 diff、查 commit log、做分支操作，还能让 AI 帮你生成 commit message。对那种一天让 Agent 改十几处小东西的人来说，这一步确实省心。

还有一个细节挺反常：安装包只有 7MB。

现在很多开发工具，动不动几百 MB 起步，打开以后风扇先替你鼓掌。Nezha 用 Tauri 做桌面端，项目页也把“轻”当成卖点讲，至少方向上很克制。

当然，它也不是无脑替代 VS Code 或 Cursor。

更准确地说，Nezha 适合那些已经开始同时调度多个 AI 助手的人。你不再只是在一个文件里补全代码，而是在几个项目之间分配任务、看进度、审结果、合代码。

这类工作流一旦跑起来，传统编辑器就会显得有点挤。

项目目前开源，许可证是 GPL-3.0

GitHub 地址：hanshuaikang/nezha。