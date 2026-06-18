> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: claude code 发布新版本 2.1.100，推出更省 token 的功能
author: 渔夫 AIDaily
date:
url: https://mp.weixin.qq.com/s?__biz=MzUyODgxNzM0Nw==&mid=2247489798&idx=1&sn=4916122a83fb0628695cf65a42a77f44&chksm=fb596002343a719524230fa94cccfb20a27593b0af291df2ac1a31bb1bc4904ce695f45013fb&mpshare=1&scene=24&srcid=0411LFkKJONA0uICdwc27PE7&sharer_shareinfo=e75c51e9d0c3f2d50aa43fc752902637&sharer_shareinfo_first=e75c51e9d0c3f2d50aa43fc752902637#rd
---

hi，我是渔夫。

Claude code今天发布新的功能 **Monitor 工具， 据说是能大幅度省token。**

其实是一个新的内置工具，让 Claude 自己写一个**持久化后台脚本**来监控某个条件，条件触发时主动"唤醒" agent 继续干活。

说它可以省 token，做法是 Claude 可以派一个 Monitor 出去"守夜"，自己继续干别的，有事再叫它。

传统方式 Agent loop 里不断 Polling（轮询检查），这样会消耗大量 toekn。

另外，在新版本 Claude Code 2.1.100 发布中，一个比较有意思的，就是支持 /advisor 命令了。

```
/advisor [ opus | sonnet | off ]
```

现在，你通过开启 /advisor 的模型，然后 Sonnet 作为 executor 全程跑任务，调工具，迭代，当发生错误，遇到搞不定决策情况下。才会去咨询 Opus，它以第二意见的形式审阅主代理的工作。

这里的设计真的很巧妙，不仅节省 token，可以让你跑长任务时，更加放心。Opus 拿到的是共享上下文，不是重新起一个对话，这样能看到完整的执行过程，给出的建议也不是空中楼阁。

而且每个 agent 孤立运行，但上下文打通，这是 harness 工程的核心哲学之一。

什么场景使用：如当在长 session，不想全程烧 Opus 的钱但又怕 Sonnet 在关键节点掉链子，可以开启。

还有重构，多文件修改这类，动脑子少的场景，可以开启，起码 opus 可以给你兜底指挥。

总结一下，这两个小功能还蛮有意思，Monitor 让 LLM 不去轮询，脚本守夜。然后/advisor 让 Opus 不去打螺丝，只管决策，都能省 toekn。

如果你正在用 Claude Code 跑长任务，那推荐可以试了。

另外，我也上线了一本手册《Claude code harness engineering：入门到实战》，仅供参考学习。

推荐阅读：

[Harness 是个过渡，Environment Engineering 才是未来](https://mp.weixin.qq.com/s?__biz=MzUyODgxNzM0Nw==&mid=2247489755&idx=1&sn=9a6f4bc806f70d2a633a64e70f575bf0&scene=21#wechat_redirect)

[我用 Claude Code 接入 Figma，1 小时重构整个小程序 UI](https://mp.weixin.qq.com/s?__biz=MzUyODgxNzM0Nw==&mid=2247489729&idx=1&sn=8d8f6a0e91d54bdd60c14bd54759c453&scene=21#wechat_redirect)

[Claude Code 核心负责人亲自分享！15 个隐藏高效用法](https://mp.weixin.qq.com/s?__biz=MzUyODgxNzM0Nw==&mid=2247489694&idx=1&sn=bb8519ca9545798b6e1fd64ef06cd8d0&scene=21#wechat_redirect)

[Anthropic官方上线Claude硬核课程，共 13 门，学完获结业证书，简历加分！](https://mp.weixin.qq.com/s?__biz=MzUyODgxNzM0Nw==&mid=2247489471&idx=1&sn=1e0b23a7b1f4d08bb3d95b144661c6f3&scene=21#wechat_redirect)