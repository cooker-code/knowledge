> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 新增 Recap 功能，多开党必备神器
author: AGI Hunt
date:
url: https://mp.weixin.qq.com/s?__biz=MzA4NzgzMjA4MQ==&mid=2453483005&idx=1&sn=c9f8f61c028246d79060eac3ae9a94e6&chksm=8692cc7ea46d96df0f136294e6658ad985c4cc3e99ff10da9486f2418668fffb449b3bc15e74&mpshare=1&scene=24&srcid=0422z9TERQKQcwv06ntgyuQu&sharer_shareinfo=3530cfd22ceccf3c3f6c57b63cf65b10&sharer_shareinfo_first=3530cfd22ceccf3c3f6c57b63cf65b10#rd
---

同时开好几个 Claude Code 的人，应该都有过这种体验：切回某个标签页，盯着满屏的操作记录，一脸茫然。

刚才……它干到哪了来着？

Multi-clauding 的烦恼

Anthropic 今天更新了一个小功能，解决的就是这个问题。

**Claude Code 现在会在你切回窗口时，自动显示一段 recap 摘要，告诉你它刚才干了什么、干到哪了、接下来该干什么。**

多标签页场景

01

## 实际效果

官方演示里展示了一个典型的「multi-clauding」场景：同时开着两个 Claude Code 会话，一个在做 auth 中间件拆分，另一个在跑数据库迁移。

auth-refactor 那边已经干完了，它把 `auth.ts` 拆成了 `validateSession` 和 `requireRole` 两个文件，改完了 6 个调用点，跑了一遍 typecheck，没报错。

然后你切到 billing-migration 这边。

Claude 正忙着呢，已经迁移了 4 张账单表到 v2 schema，改了 Prisma 模型，跑了测试，11 个通过但有 1 个失败，是 `invoices.ts` 的外键约束出了问题，正在排查。

这时候，底部自动弹出一行 recap：

> “ recap: Migrated 4 of 7 billing tables to the v2 schema; invoices.ts still fails its FK constraint. Next: fix the foreign key on line 142.

recap 摘要出现

一句话就把进度、问题、下一步全说清楚了。

连那个小牛吉祥物头顶都亮了个灯泡，像是在说：「来，我给你捋捋。」

02

## 多开党的痛

这个功能看起来不大，但它其实戳中了一个真实的痛点。

我自己就是个重度多开党。用的终端是 Ghostty，日常工作的时候会用快捷键疯狂 split pane，一个 window 里开十几个 pane 是常态。每天打开电脑，几十个 pane 铺满屏幕，每个里面跑着不同的 Claude Code agent、不同的服务、不同的日志。

Ghostty 分屏日常

这种工作方式效率确实高，但有个绕不开的问题是……**切回某个 pane 的时候，你得先花时间搞清楚它干到哪了。**

往上翻，看它读了哪些文件、改了什么、测试过没过……然后自己在脑子里拼出一个「当前状态」。

pane 越多，这个恢复上下文的成本就越高。本质上，其实是人类在帮 AI 做上下文恢复。

有了 recap，这事儿反过来了。

**会由 AI 来主动帮你恢复上下文。**

Claude Code 团队的 Thariq 转发了这条更新，称这是他个人最喜欢的体验改进之一：

> “ one of my favorite quality of life features

03

## 像便利贴

如果你用过那种在显示器边上贴满便利贴的工作方式，recap 的感觉就很像。

每个便利贴上写着：「这个任务做到哪了，下一步干什么。」只不过现在贴便利贴的人，变成了 Claude 自己。

Claude 自动贴便利贴

而且它贴的比你自己写的还准。毕竟所有操作记录它都有，它不会漏掉「测试跑了两遍、第二遍还是挂在第 142 行」这种细节。

这个功能目前已经在终端版 Claude Code 中上线了。如果你也是多开党，下次切窗口的时候，应该就能看到了。

**以后「刚才干到哪了」这个问题，**

**终于不用自己翻记录了。**

---

相关链接：

https://x.com/ClaudeDevs/status/2046613932518023259