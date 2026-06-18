---
title: 用AI搭建公司知识库：Knowledge Bank 让每个工程师都站在团队的肩膀上
author: 大数据最佳实践
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5NTI3MTU3NA==&mid=2247483945&idx=1&sn=1885d0055b1bf09eaf0547034bcdbe53&chksm=9122fa43f1be5237f08561d5057d1ba7c06a3a63cc037590c689f0b0789caa1a8aba911d57ee&mpshare=1&scene=24&srcid=04143WoD8piFR7SCBTqwP7Y2&sharer_shareinfo=b90188e9d6aaf05a9481f58c72c83d87&sharer_shareinfo_first=b90188e9d6aaf05a9481f58c72c83d87#rd
---

## 从一条推文说起

最近，前 OpenAI 创始成员、Tesla AI 负责人 Andrej Karpathy 发了一篇帖子，分享他用 LLM 搭建个人知识库的方式：

>  用LLM构建知识库。我把原始资料（文章、论文、代码仓库）索引到 raw/ 目录，然后用 LLM 增量地”编译”出一个 wiki。这个 wiki 只是一堆 .md 文件，包含所有内容的摘要、反向链接，以及按概念分类整理的文章。我用 Obsidian 作为”IDE 前端”来浏览原始数据和编译好的 wiki。重要的是：LLM 负责编写和维护 wiki 里所有的数据，我几乎不会直接去碰它。

这个思路很优雅：人负责摄入原料，AI 负责编译知识。

读完这条推文，我有一个问题：如果一个工程师能这样做，一个工程团队呢？

这就是 Knowledge Bank 要解决的问题。

---

### 个人知识库 vs 公司知识库

Karpathy 解决的是个人问题：如何让自己对某个研究领域的理解持续积累、可检索、有结构。

而工程团队面临的是一个更复杂、也更紧迫的问题：

* 知识不是一个人的：它分散在每个工程师的脑子里、PR 评论里、Slack 消息里、随手写的注释里
* 知识不是静态的：框架升级、业务变化、架构演进，知识每天都在过时
* 知识不是给人查的：真正高频消费知识的，是你的 AI 编程助手

每天都有人问：”我们项目怎么处理错误？”、”认证是怎么做的？”、”为什么当初选了这个库？”

这些答案存在于团队的集体记忆里——但集体记忆最脆弱，随着人员流动而消失，随着时间推移而模糊。

Karpathy 需要一个 Obsidian。工程团队需要的是 Knowledge Bank。

---

### Knowledge Bank 是什么

Knowledge Bank 是专为工程团队设计的 AI 知识载体。

它不是 Wiki，不是文档系统，也不是知识管理平台。它是工程师使用 AI 编程助手（如 Claude Code）开发时，在后台自动运行的知识积累和注入系统——公司和项目内部知识的最重要载体。

借用 Karpathy 的框架来理解：

| Karpathy 个人知识库 | Knowledge Bank（团队知识库） |
| --- | --- |
| 手动把文章放进 raw/ | 开发会话自动成为原料 |
| LLM 编译成 wiki（.md 文件） | AI 自动提取结构化知识条目 |
| Obsidian 浏览 | 知识在编码时自动注入 AI 上下文 |
| 个人使用 | 全团队共享，按仓库隔离 |
| 主动查询 | 主动注入，无需查询 |

最后一行是最关键的区别。

---

### Knowledge Bank 的工作方式

### 第一步：自动捕获（零摩擦数据摄入）

Karpathy 需要手动把文章放进目录。Knowledge Bank 的数据摄入是完全自动的。

知识在开发过程中自然流出：

```
```
工程师使用 Claude Code 开发（正常工作）              ↓     会话结束时，Knowledge Bank 自动触发              ↓     AI 分析整个对话记录              ↓     识别：架构决策、踩过的坑、发现的代码模式              ↓     存入 Knowledge Bank（附带仓库、分支上下文）
```
```

工程师什么都不用做。每一次开发会话，都在悄悄充实团队的知识储备。

### 第二步：结构化存储（不是 Wiki，是有维度的知识体系）

Karpathy 的 wiki 按概念分类。Knowledge Bank 在此基础上增加了团队场景必须的三个维度：

维度1：作用域 — 知识归谁用

```
```
个人知识（Personal）    → 我的编码偏好，不传播给团队 项目知识（Project）     → 这个仓库的约定，自动绑定到 git repo 组织知识（Organization）→ 公司工程规范，所有项目都生效
```
```

维度2：来源 — 知识有多可信

```
```
AI 观察（AI）          → 从实际代码中提取，有源码为证，最可信 架构师决策（Architect） → 深思熟虑的设计决策 开发者经验（Developer） → 实践积累，个人经验
```
```

维度3：类型 — 知识怎么用

```
```
代码模式（Code Pattern） → 直接复用的写法 陷阱警示（Pitfall）      → 不要踩的坑 架构决策（Architecture） → 为什么这么设计 API 用法（API Usage）    → 标准用法和注意事项
```
```

这三个维度让 Knowledge Bank 里的每条知识都清楚地知道自己的边界、可信度和用途——而不是一锅粥的 Wiki。

### 第三步：智能注入（主动送到面前，而不是等待查询）

这是 Knowledge Bank 与个人知识库最本质的不同。

个人知识库是”我打开 Obsidian 去查”。Knowledge Bank 的目标是：工程师不需要想着去查，AI 会在恰当的时机把恰当的知识送过来。

```
```
工程师向 Claude 提问："帮我实现一个新的 API 接口"              ↓     Knowledge Bank 自动检索相关知识（工程师无感知）     ✓ 项目用 Fastify 框架（项目知识）     ✓ 错误处理统一用 handleError()（代码模式）     ✓ 历史 Code Review：API 必须有输入验证（Reviewer 规则）     ✓ 已知陷阱：不要在循环中 throw error（陷阱警示）              ↓     Claude 基于项目知识回答，生成符合规范的代码
```
```

知识不再需要工程师去查，它在需要的时候自动出现。

---

### 效果：新人第一天就能写出符合规范的代码

没有 Knowledge Bank：

```
```
第1天：浏览文档链接，大部分已过时 第1周：到处问"为什么要这么做？" 第3周：Code Review 被打回，"我们不是这样做的" 第1个月：才慢慢摸清楚项目的"潜规则"
```
```

有了 Knowledge Bank：

```
```
第1天：打开 Claude Code        Knowledge Bank 自动加载：        ✓ 认证方案：JWT + token 存 localStorage        ✓ 错误处理：统一使用 handleError()        ✓ 框架约定：Fastify + Zod 验证        ✓ 已知陷阱：并发写入需要事务保护        → 写出完全符合项目规范的代码        → Code Review：LGTM ✅
```
```

Knowledge Bank 里积累的是整个团队的集体记忆，任何人都可以直接站在上面工作。

---

### 回到 Karpathy 那句最关键的话

> LLM 负责写和维护 wiki 里所有的数据，我几乎不会直接去碰它。

这句话揭示了一个根本性的转变：人不再是知识的编辑者，而是知识的使用者。

Knowledge Bank 把这个原则带入了团队场景：

* 工程师不需要写文档：Knowledge Bank 在开发过程中自动提取和整理
* Knowledge Bank 不会过时：每次开发会话都在更新它
* Knowledge Bank 不会因人员流动而消失：它是团队的集体记忆，不依附于任何个人

Karpathy 用 LLM 管理他个人对 AI 研究领域的认知积累。 Knowledge Bank 用同样的方式，管理工程团队对自己项目的集体认知积累。

---

### 结语

个人知识管理有一句话：“你的第二大脑”。

Knowledge Bank 是工程团队的集体第二大脑——

不需要人去写，AI 来整理； 不需要人去查，AI 来注入； 不会随着人员流动消失，不会随着时间推移过时。

你的下一行代码，应该站在整个团队所有过往经验的肩膀上。

这就是 Knowledge Bank 存在的意义。

---

knowledge bank —— 让工程团队的集体智慧，成为每个工程师的超能力  

Knowledge Bankgithub.com/gabrywu-public/knowledge-bank