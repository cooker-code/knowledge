> 已吸收至：[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/多角色编排与权限隔离|多角色编排与权限隔离]]、[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/运行环境与沙箱底座|运行环境与沙箱底座]]、[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/长任务状态与恢复闭环|长任务状态与恢复闭环]]
---
title: Anthropic最新harness工程技术：Managed Agents
author: 知识药丸
date:
url: https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649477680&idx=1&sn=63e0bc2cfb3ae9926b35519cb055026d&chksm=8fd766d3357e7a03a435729b828b583f0452f5f44095aa1d5ecafe7cab816b483679d35232ae&mpshare=1&scene=24&srcid=0409mq6X3tJiFhB9jSuqq1NF&sharer_shareinfo=afcd8fb31a9fb0d8e78c5930f2c8b9a4&sharer_shareinfo_first=afcd8fb31a9fb0d8e78c5930f2c8b9a4#rd
---

🌟星标 + 👆关注，第一时间知道最新、最有用的AI编程姿势

[《贾杰的AI编程秘籍》](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649474437&idx=1&sn=4ae1516afc0ec71b7638dd79daaae2cb&scene=21#wechat_redirect)付费合集，共10篇，现已完结。30元交个朋友，学不到真东西找我退钱；）

 

以及[我开发的手机编程App，已经冲到了付费榜第一，欢迎围观](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649477442&idx=1&sn=165edcddec85ffaff9db02890f9d5e55&scene=21#wechat_redirect)

---

### 写在前面

最近在读 Anthropic 工程博客，看到一篇关于 Managed Agents 的文章，标题叫《Scaling Managed Agents: Decoupling the brain from the hands》。

读完之后有点震动——不是因为技术有多新，而是因为它把一个我隐约感觉到但说不清楚的问题，用一句话说透了：**harness 里编码的假设，会随着模型的进步而过时**。

这篇是我的学习笔记，记录一下自己的理解。

---

### 先搞清楚：harness 是什么

在聊 Managed Agents 之前，得先把"harness"这个词搞清楚，因为它是整篇文章的核心概念。

Harness，字面意思是**马具**——就是套在马身上那套缰绳、轭具的组合，用来控制马的方向和力道。

在 AI Agent 的语境里，harness 指的是**包裹在模型外面的那层脚手架代码**：决定暴露哪些工具、怎么格式化 prompt、出错了怎么重试、上下文窗口快满了怎么处理……这些都是 harness 的工作。

换句话说，模型是大脑（brain），harness 是手（hands）。大脑负责思考，手负责执行。

---

### 问题出在哪里

这个比喻很直觉，但问题也藏在这里。

手是为大脑定制的。你给一个两岁小孩设计的辅助手套，套在成年人身上只会碍事。

Harness 也一样。

Anthropic 在文章里举了一个很具体的例子：早期的 Claude 模型需要在 system prompt 里手把手地告诉它"先想想需要哪些工具，然后一次用一个，每次用完之后反思一下学到了什么"。这对 Claude 2 是真实有效的帮助。

但 Claude 3 之后，模型已经会自己规划和反思了。你再把这段话塞进去，不仅是在浪费 token，更糟糕的是——你在
**过度指定一个模型本来能自己处理得更好的过程**
。

这就是所谓的"harness 假设过时"。当初写下这些规则的人是好意，但好意会变成枷锁。

---

### 解耦：把大脑和手分开

Managed Agents 的核心设计原则就一个词：**decoupling（解耦）**。

具体来说，就是把模型（brain）和 harness（hands）分开，让它们能独立演进。

这其实是软件工程里最基本的道理——我们有 API 而不是直接共享源码，就是因为接口应该稳定，实现可以变。但在 AI 系统里，这个道理特别容易被忘掉。

大多数 agent 框架今天的做法是**把 harness 的内部细节全部暴露出来**：你来配置 prompt 模板，你来设置重试次数，你来选择工具调用的格式。这给了你控制权，但也把你和实现细节**耦合**在了一起。一旦模型升级，你的代码就得跟着改。

Managed Agents 反过来。它的接口极度简单：

```
  POST /v1/managed-agents/tasks

{
  “model”： “claude-opus-4-5”，
  “task”： “分析这份 CSV 里的销售数据，写一份摘要报告”，
  “files”： [...]，
  “tools”： [“web_search”， “code_execution”]
}
```

你描述**想要什么**，它负责**怎么做**。重试逻辑、prompt 模板、上下文管理——这些全是实现细节，对你不可见，也不需要你操心。

Anthropic 说他们内部已经改过好几次 harness 了：改过工具结果的格式化方式，改过重试逻辑，改过上下文窗口的处理策略。用户那边，**一行代码都没改**。

### 长任务为什么难

Managed Agents 专门针对的是**长时任务（long-horizon tasks）**——那种需要几十步操作、跑几分钟甚至几小时的任务。

为什么长任务特别难？文章里总结了三点，我觉得说得很准。

**第一是上下文管理。** 一个需要 50 次工具调用的任务，会积累大量上下文。什么该保留、什么该压缩、什么可以丢掉——这是个非平凡的问题。处理不好，要么撑爆上下文窗口，要么丢失关键信息。

**第二是错误恢复。** 步骤越多，出错的机会越多。一个好的 agent 需要知道自己卡住了，能尝试换个思路，还要知道什么时候该放弃并报告失败，而不是无限转圈。

**第三是状态管理。** 有些任务需要在很多步骤之间维护状态——记录哪些做完了、哪些还没做、步骤之间有什么依赖关系。

这三件事，Managed Agents 都帮你处理好了。

### 异步是正确的默认值

Managed Agents 默认是**异步**的。

你提交任务，立刻拿到一个 task ID，然后去做别的事。任务在后台跑，跑完了你再来取结果——或者用 webhook，让它主动通知你。

```
  GET /v1/managed-agents/tasks/{task_id}

{
  “id”： “task_abc123”，
  “status”： “completed”，
  “result”： {
    “type”： “text”，
    “content”： “销售分析报告\n\n...”
  }
}
```

这对长任务来说是唯一合理的设计。一个跑 20 分钟的任务，不应该阻塞你的应用。

### 我觉得最有价值的一句话

整篇文章里，我觉得最值得反复咀嚼的是这句：

> Harnesses encode assumptions. And assumptions go stale.

（harness 里编码了假设，而假设会过时。）

这不只是在说 AI agent 的架构问题。它说的是一个更普遍的工程困境：**今天的最佳实践，明天可能是技术债**。

我们写代码的时候，总是在对当前的环境做假设。假设用户的网速是这样的，假设数据库的规模是那样的，假设模型的能力是这样的。这些假设在当时是合理的，但环境会变。

好的架构，就是把**稳定的接口**和**会变的实现**分开，让变化发生在正确的地方。

### 总结

Managed Agents 本质上是在做一件事：**把"你想要什么"和"怎么做到"彻底分开**。

你只需要描述任务，它负责处理 harness 的所有细节——而且随着模型的进步，这些细节会悄悄变好，你的代码不用动。

这背后的设计哲学，其实就是软件工程里最古老的那条原则：**面向接口编程，而不是面向实现**。只不过现在，这个原则被用在了 AI agent 的架构上。

---

 

 

坚持创作不易，求个一键三连，谢谢你～❤️

以及「AI Coding技术交流群」，联系 ayqywx 我拉你进群，共同交流学习～

订阅链接 https://note.mowen.cn/detail/OLPEp7HzeB0EXJOLe7mM4