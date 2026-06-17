---
title: Agent效率X10，ClaudeCode/Codex 专属的文件管理Agent：Agentfile
author: AI产品自由
date: 成峰成峰
url: https://mp.weixin.qq.com/s?__biz=MzU3MjU5Mzc2Nw==&mid=2247489575&idx=1&sn=c9043a57893a4c5889d0c52e1deca627&chksm=fdf945da8922c06a2abd155cbc515eb1f604a14ff8e330b2970025be889af48bd65d3f3aa5b1&mpshare=1&scene=24&srcid=0512WcHeuEqKXdBN0l9tFi2I&sharer_shareinfo=aafabd4269233e9a77e7f0d31dc69d53&sharer_shareinfo_first=aafabd4269233e9a77e7f0d31dc69d53#rd
---

Claude Code 创始人 Boris Cherny，发过一张自己的终端截图

他在终端里同时跑 5 个 Claude，把标签页编号为 1-5。哪个 Claude 需要输入，就让系统通知提醒他

一个 Claude 改前端，一个 Claude 修 bug，一个 Claude 跑测试，人更多是在拆任务、看结果、最后收口

 

再看 Claude Code 团队自己的二月更新日历。

非常夸张。二月几乎每个工作日都有新功能上线……一整个月都没停

一个 AI 编程工具，自己也在用 AI 加速开发

# Agent 跑得快，怎么看文件

我也开始把这种方式搬到自己的项目里

编程项目 A、编程项目 B、写作项目、编程项目 C，几件事一起往前推。原来要排队处理的事，开始同时滚动

 

Agent 同时往前跑，结果却开始四处散。

我需要一个地方，把这些项目收进同一个窗口里，像浏览器标签页一样来回切。

所以我做了 Agentfile：https://github.com/Ceeon/agentfile

# 项目像网页一样切换

打开 Agentfile，几个项目文件夹先变成了顶部的一排标签页。

编程项目 A、编程项目 B、写作项目、编程项目 C，都在这个窗口里。

我不用再开一堆 Finder、终端和编辑器窗口。要看哪个项目，就切到哪个标签页

 

哪个 Agent 跑完，我就用快捷键切回哪个项目。不用先想“这个项目在哪个窗口”。回到对应的项目标签页，继续看文件、看日志、接着处理

# 少背一层终端，才能更小更快

Agentfile 是从 Wave 改出来的。

Wave 原来是一个终端工作台，终端、连接、面板、窗口都可以往里放。

能力很完整，但项目一多，内存很快就顶上来。

如果继续这样做，我可能只能并发四五个项目。再多开几个终端，切换和响应都会开始吃力。

 

所以我把终端拆了出去。

Agentfile 只做文件管理和项目切换。真正要跑命令，就交给 Ghostty 或 cmux（都是终端工具）。

这类终端本来就更适合干这件事：内存占用小，渲染快，Bug 也少。

 

拆开以后，Agentfile 自己轻了很多。我现在可以同时挂着多个项目标签页，再并发十几个外部终端。

# Html 比 Markdown 更适合做展现

最近 Claude Code 团队的 Thariq 写了一篇文章，标题叫《The Unreasonable Effectiveness of HTML》。

它说的不是“Markdown 没用了”。

 

Agent 现在生成的东西，早就不只是几段文字。它会生成代码审查报告、数据图表、设计方案、流程图、教程页，甚至一个可以交互的小工具。

 

这些东西放在 Markdown 里，很容易变成一堵文字墙。换成 HTML，表格、颜色、图表、布局和交互可以直接呈现出来。

Agentfile 里可以承接这类 HTML 结果。我切回这个项目标签页，就能直接打开看。

我也内置了一版 Markdown 样式。

# 工具也可以随叫随改

这几天加 HTML 预览，我没有重新写一份需求文档。

我直接把需求丢给 Agent。Agent 按 Skill 启动测试版、看日志、改代码、跑验证。

 

我看一眼测试版，能用，就继续拿测试版跑。

这套流程跑通以后，Agentfile 的角色也变了。

 

文件管理变成入口，后面接着一套可被 Agent 接管的开发流程。它更像一种新的软件形态。

 

以前的软件，是开发者交付给用户。

以后有一类软件，会变成用户和 Agent 一起持续改。哪里不顺手，Agent 可以按我的使用习惯改。

 

出了 Bug，Agent 可以先看日志、复现问题、尝试修复。如果本地解决不了，它也可以把上下文整理好，发到 GitHub，让上游开发者继续接。

 

所以我说 Agentfile 是 Skill 驱动的 Agent。

Skill 在这里更像一份操作协议，让 Agent 可以接管软件。

它告诉 Agent 这几件事：

后面 Agentfile 继续更新，我也会继续更新这些 Skills。软件本身会变，改软件的方法也会一起沉淀下来。

 

这才是我说的“随叫随改”。

Agentfile 本来就是从 Wave 魔改出来的，也欢迎别人继续魔改。

# Agentfile 做的是收口

Agentfile 的定位很明确：收口。

终端交给 Ghostty 和 cmux。Agentfile 做的是文件管理那一层。

 

几个项目在哪里，Agent 跑出了什么，下一步要回到哪个文件夹，哪个结果应该直接预览。

 

这些东西原来散在 Finder、终端、编辑器和浏览器之间。

Agentfile 把它们收成一个项目入口。对我来说，它就是多 Agent 工作流里的收口台。