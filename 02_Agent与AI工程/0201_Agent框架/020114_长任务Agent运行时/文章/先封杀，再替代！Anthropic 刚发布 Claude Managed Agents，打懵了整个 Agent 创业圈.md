---
title: 先封杀，再替代！Anthropic 刚发布 Claude Managed Agents，打懵了整个 Agent 创业圈
author: AI智见录
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU3NTg5MjU1Mw==&mid=2247496965&idx=1&sn=dcb93c04d48101a4b9b1cfaa23e8ac35&chksm=fc8ea700f9ba5ec44e87b325204996f971ae0e13577db8903783f2228a463167ad3ef51111a5&mpshare=1&scene=24&srcid=0409OSasrRyLjQZrhMbFVp0G&sharer_shareinfo=489b9e0e75f04d8c5c1fa450779732f5&sharer_shareinfo_first=489b9e0e75f04d8c5c1fa450779732f5#rd
---

大家好，我是智见君！

Anthropic 又放了一个大招。

Claude Managed Agents 正式进入公开测试。简单来说，这是一个**由 Anthropic 官方托管的 Agent 部署平台**，你只需要定义 Agent 的任务、工具和规则，Anthropic 帮你跑起来。沙箱、状态管理、凭证隔离、多 Agent 协调，全部打包好了，开箱即用。

消息一出，社交媒体上就炸开了锅。Twitter 上一位叫 Aakash Gupta 的博主写了一段话，被疯狂转发：

> "Anthropic just mass-obsoleted every agent orchestration startup in a single launch."
>
> （Anthropic 一次发布就让所有 Agent 编排创业公司集体过时了。）

说得有点狠，但确实戳到了痛处。

**为什么说这是"降维打击"？**

过去一年，市面上涌现了大量围绕 Agent 编排的创业公司和开源框架。LangChain、CrewAI、AutoGen……它们做的核心事情就是：帮你管理 Agent 的生命周期，处理工具调用，协调多个 Agent 之间的协作。

问题在于，这些框架本质上都是**在模型供应商的 API 之上搭的一层"胶水"**。而现在，模型供应商自己下场了。

Anthropic 的 Managed Agents 提供了什么？

* • **安全沙箱执行**：Agent 在隔离环境中运行，凭证存储在沙箱之外的保险库里，连提示注入攻击都拿不到环境变量
* • **长时间会话**：Agent 可以自主运行数小时，断开连接也不会丢失进度
* • **多 Agent 协调**（研究预览版）：一个 Agent 可以生成并指挥其他 Agent 并行处理任务
* • **MCP 原生集成**：OAuth 令牌安全管理，Claude 通过专用代理调用 MCP 工具
* • **Session 持久化**：不可变的事件流记录，支持上下文恢复和历史回溯

最关键的是，这些能力是 **为 Claude 量身定制的**。第三方框架在通用性上做文章，但 Anthropic 比谁都了解自家模型的脾气。

**架构设计：操作系统级的思维**

Anthropic 同步发布了一篇工程博客，详细拆解了 Managed Agents 的架构。读完之后，说实话，设计思路确实高级。

他们把 Agent 拆成了三个解耦的组件：

**Brain（大脑）**：Claude 模型 + Harness（执行框架），负责推理和决策。所有工具调用统一为一个简洁的接口：`execute(name, input) → string`。

**Hands（双手）**：执行环境，包括沙箱容器、代码执行器、外部工具、MCP 服务器。Brain 和 Hands 之间通过统一接口通信，不再绑定在同一个容器里。

**Session（会话）**：独立于 Harness 的不可变事件流，支持切片查询、上下文恢复。传统 Agent 框架最头疼的"长对话超出上下文窗口"问题，在这里被干净利落地解决了。

这个解耦带来了实打实的性能提升。Anthropic 给出的数据：**p50 首 Token 延迟下降约 60%，p95 下降超过 90%**。

原因很直观，Harness 作为无状态服务部署，只在需要时才分配容器资源，避免了传统方案中"一个 Agent 独占一个容器"的浪费。

**容错设计：Agent 挂了不怕**

生产环境最怕什么？Agent 跑到一半挂了，所有上下文全丢。

Managed Agents 的容错逻辑也值得说一下：

* • **容器故障**：Harness 捕获工具调用错误，交给 Claude 决策是否重试。重试时会启动新容器，通过标准配方重新初始化
* • **Harness 故障**：重启后调用 `wake(sessionId)`，从 Session 事件流中恢复上下文，继续执行

因为 Session 是外部持久化的，不依赖任何单一组件的存活，所以系统的韧性比传统单容器方案强得多。

**谁已经在用了？**

Anthropic 在博客里列了一串早期客户，个个来头不小：

* • **Notion**：把 Managed Agents 内置到自定义 Agent 功能里，支持代码交付和内容生成，可以并行跑几十个任务
* • **Rakuten（乐天）**：一周内部署了企业级 Agent，覆盖产品、销售、营销、财务和人力资源部门，接入了 Slack 和 Teams
* • **Asana**：打造了"AI 队友"，在项目中跟人类协作，处理任务和起草交付物
* • **Sentry**：组合调试 Agent 和补丁编写 Agent，从 bug 标记到可审查 PR 的完整流程，几周内就上线了

这些不是 Demo，是真实的生产部署。Anthropic 宣称"从原型到上线快 10 倍"，从这些案例来看，并非虚言。

---

这个发布时间点太有意思了。

就在四天前，Anthropic 刚刚封杀了 OpenClaw 和其他所有通过消费者订阅凭证调用 Claude 的第三方 Harness。当时很多人骂 Anthropic 霸道，"凭什么不让我用自己的账号接别的工具？"

现在答案揭晓了：**你不需要别的工具了，我自己做了一个更好的。**

先堵后疏，这招棋走得很清楚。先切断第三方对消费者 auth 层的依赖，再推出官方的生产级替代方案。不管你是否认同这种手法，从商业策略上看，确实是一套连贯的组合拳。

---

Managed Agents 目前还是公开测试阶段，多 Agent 协调和自评估功能需要申请研究预览版权限。距离全面成熟还有一段路。

但方向已经很明确了：**模型供应商正在把 Agent 基础设施收归平台层**。OpenAI 有 Assistants API，Google 有 Vertex AI Agent Builder，现在 Anthropic 也有了 Managed Agents。

留给第三方 Agent 编排框架的空间，正在一点点被挤压。未来能活下来的，要么是做出了模型供应商不愿意做的垂直场景深度，要么是在多模型编排上建立了不可替代的价值。

至于那些只是简单包装 API 调用的"Agent 平台"？趁还来得及，该转型了。