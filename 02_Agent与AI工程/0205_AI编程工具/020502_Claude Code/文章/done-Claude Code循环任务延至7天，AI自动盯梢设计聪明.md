> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code循环任务延至7天，AI自动盯梢设计聪明
author: 知识药丸
date:
url: https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649477475&idx=1&sn=667482a414432a2fe943fb24989153f0&chksm=8f2e92a3e19e9859b2a2d8bd8de9e9e7b752904aaee00bb90e8fb6a25e7f8b1519b9905f211e&mpshare=1&scene=24&srcid=0323FBIaW5QOBqFAlC9KY8ds&sharer_shareinfo=bf40769322d28fd4ee1ed05208bd04ef&sharer_shareinfo_first=bf40769322d28fd4ee1ed05208bd04ef#rd
---

👀 最新、最有用的AI编程姿势，总来自「知识药丸」

[《贾杰的AI编程秘籍》](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649474437&idx=1&sn=4ae1516afc0ec71b7638dd79daaae2cb&scene=21#wechat_redirect)付费合集，共10篇，现已完结。30元交个朋友，学不到真东西找我退钱；）

 

我开发的手机编程app，已经上架AppStore，搜索 enjoycoding 欢迎围观

---

### 写在前面

前两天看到 Noah Zweben 在推特上说，Claude Code 的 loop 功能从 3 天延长到 7 天了。

听起来挺普通的一个更新，但我翻了下文档，发现这玩意儿设计得**挺漂亮**。

想想你提了个 PR，等 CI 跑完。刷网页？太蠢。轮询脚本？太重。

最好的办法是什么？让 AI 帮你盯着。

这就是 `/loop` 干的事。

### 五分钟循环

最简单的用法就一行：

```
    /loop 5m check if the deployment finished
```

每 5 分钟，Claude 自动跑一次，然后告诉你结果。

时间怎么写都行。放前面、放后面、不写（默认 10 分钟）都可以：

```
    /loop 30m check the build
/loop check the build every 2h
/loop check the build
```

支持 `s`、`m`、`h`、`d`。写了 `7m` 这种怪数字？Claude 会帮你圆整，然后告诉你实际设了啥。

秒会向上取整到分钟，因为底层是 cron，粒度就这样。

### 套娃调用

循环任务可以调其他命令。

假设你封装了个 `/review-pr` 技能：

```
    /loop 20m /review-pr 1234
```

每 20 分钟，就像你手敲一样，执行一次。

这个设计**很聪明**——它不管你循环的是啥，只管定时触发。复杂逻辑封装成技能，循环只是个壳。

### 一次性闹钟

不想循环，只想提醒一次？

自然语言就够了：

```
    remind me at 3pm to push the release branch
```

Claude 会创建一个**一次性任务**，跑完自动删。

你不需要知道 cron 怎么写，说人话就行。

### 管理任务

创建多了，可能忘了都设了啥。

问就完了：

```
    what scheduled tasks do I have?
```

它会列出 ID、计划、提示词。

想删？

```
    cancel the deploy check job
```

每个任务有 8 位 ID，一个会话最多 50 个。

### 它怎么跑的

调度器每秒检查一次，到期的任务扔进**低优先级队列**。

注意这个“低优先级”——定时任务**不会打断** Claude 正在干的事，而是等它忙完了再插进来。

想象一下，你跟 Claude 聊到一半，突然蹦出个定时任务的结果，多**诡异**。

所有时间都是**本地时区**。`0 9 * * *` 就是你那儿的早上 9 点，不是 UTC。

为了避免所有会话同时触发（想想整点流量洪峰），系统加了点**抖动**：

循环任务最多晚 10% 周期，上限 15 分钟。整点任务可能在 ：00 到 ：06 之间跑。

一次性任务如果设在整点，会提前最多 90 秒。

抖动是固定的，同一个任务 ID 每次偏移量一样。想精确控制？别用整点，用 `3 9 * * *`。

### 七天寿命

这次更新的重点：循环任务现在活 **7 天**，之前是 3 天。

为啥要过期？

因为任务是**会话级别**的，只要你终端开着就一直跑。没有过期机制，忘了的循环会一直占着资源。

7 天后，任务跑最后一次，然后自杀。

需要更长？重建，或者用桌面端的持久化任务。

### 它的边界

会话级别的调度有三个**硬伤**：

**关终端就没了。** 任务活在进程里，退出会话全清空。

**错过不补。** Claude 忙的时候到期了，它会等空闲跑一次，但不会把错过的每次都补上。这不是队列，是定时器。

**不跨重启。** 重启 Claude Code，所有任务归零。

想要无人值守？GitHub Actions 的 `schedule` 触发器，或者桌面端的定时任务。

### 总结

`/loop` 本质上是把 cron 包了一层自然语言接口，然后塞进 AI 会话里。

它不是万能的后台任务系统，但对于“盯着 CI”、“等部署”、“查构建”这种**短期轮询**场景，**够用且优雅**。

7 天够干完大部分活了。再长？那不是会话该干的事。

P.S. 文档里提到可以设 `CLAUDE_CODE_DISABLE_CRON=1` 关掉整个调度器。估计是给不想要这功能的用户留的开关。

### 参考资料

* • Claude Code 官方文档 - Scheduled Tasks
* • Noah Zweben 的推文更新

---

 

 

坚持创作不易，求个一键三连，谢谢你～❤️

以及「AI Coding技术交流群」，联系 ayqywx 我拉你进群，共同交流学习～

订阅链接 https://note.mowen.cn/detail/OLPEp7HzeB0EXJOLe7mM4