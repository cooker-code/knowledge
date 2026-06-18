---
title: 还在说 MCP 要被淘汰？Anthropic：这才是 Agent 接入生产环境的正确姿势
author: 张新宇同学
date: 
url: https://mp.weixin.qq.com/s?__biz=MzY5NDExMTI3Mg==&mid=2247483922&idx=1&sn=8e67eab7bf7d7160f8a30f5755e2cbd4&chksm=f5c51134d0d9b38994ba06d49fa6bfe214c810be916876336565573a6bc8b300ea30c3b63e66&mpshare=1&scene=24&srcid=0423fGKBcC2jcTlevXj8F04e&sharer_shareinfo=32aefce7d409201f09d9e452f12f439b&sharer_shareinfo_first=32aefce7d409201f09d9e452f12f439b#rd
---

最近这段时间，"MCP 淘汰论"在开发者社区里传得挺响。有人说用 CLI 就够了，有人说 Skills 出来了 MCP 没必要，还有人说这东西太重、落地太难。

我们之前专门写过一篇文章批了这个观点——MCP、CLI、Skills 根本不在一个维度上，谈淘汰是混淆了层次。[有了食谱，菜刀就淘汰了？——聊聊 MCP、CLI 与 Skills](https://mp.weixin.qq.com/s?__biz=MzY5NDExMTI3Mg==&mid=2247483904&idx=1&sn=93539b77125d93572ac2f3c9349b9f6b&scene=21#wechat_redirect)

4 月 22 日，Anthropic 发了一篇官方博客，把这件事说得更透彻了。不是来辩论的，是直接用生产实践给出了答案：

MCP 不是选项之一，它是 Agent 进入生产环境的关键层。

博客的第一句话直接点出核心：

> **"Agents are only as useful as the systems they can reach."**

翻译成大白话：模型再强、提示词再精，Agent 连不到你的数据库、接不上你的工单系统，它就只是个聪明的聊天窗口。而 MCP，正是解决这个问题的那一层。

下面我们来拆这篇博客。

> 博客原文：https://claude.com/blog/building-agents-that-reach-production-systems-with-mcp

---

# 接入外部系统有三条路，但它们不是竞争关系

博客一开始梳理了三种接入外部系统的路径，顺便也把"淘汰论"的逻辑漏洞说清楚了：

直接 API 调用：最简单的起点。Agent 直接发 HTTP 请求，或者用 function calling 调你的 API。对于单个 Agent 接单个服务，这完全够用。但规模一大，问题就来了——每对 Agent-服务都需要单独维护 Auth、工具描述、边界处理，最终演变成无法管理的 M×N 组合爆炸。

CLI 命令行：让 Agent 在 shell 里运行你的命令行工具。轻量、快速，本地环境里效果拔群，还能复用已有的工具生态。但它的上限也很硬：一旦要连接 mobile、web 或云端托管的平台，CLI 就力不从心了，Auth 也只能靠磁盘上的凭证文件。

MCP 协议：在这两者之上，提供了一个标准化的协议层。一个远程 MCP Server，可以被 Claude、ChatGPT、Cursor、VS Code 等任何兼容客户端使用，Auth、工具发现、语义描述都由协议统一处理。

Anthropic 的原话是：

> 成熟的集成最终会三者并存——API 是基础，CLI 服务本地，MCP 覆盖云端 Agent。

三种方式，各有适合的场景，没有谁淘汰谁。

---

# 为什么生产环境最终会走向 MCP？

一个关键的趋势正在发生：生产环境的 Agent 越来越多地部署在云上。

而它们需要连接的系统——数据、任务追踪、基础设施——也都在云上。这个时候，CLI 的"磁盘凭证文件"方案就彻底跑不通了。

MCP 的价值在这里才真正显现：

无论有多少种 Agent 平台，无论要接多少个后端系统，只需要维护一个 MCP Server，Auth 和接口规范统一搞定。这就是 Anthropic 说的"一个 Server 覆盖所有兼容客户端"。

数据也印证了这个趋势——MCP SDK 的月下载量已经超过 3 亿次，年初还只有 1 亿，三个月翻了三倍。Claude 目录里有超过 200 个MCP Server，每天被数百万用户使用。

---

# 怎么构建一个高质量的 MCP Server？

Anthropic 和大量企业合作后，总结出了五个决定 Server 质量的关键设计模式。

1. 构建远程 Server，而不是本地 Server

这是分发能力的前提。只有远程 Server 才能在 web、mobile、云端 Agent 里通用。本地 Server 只能服务本地环境，价值极为有限。

2. 工具按"意图"分组，而不是按 API 映射

这是最容易踩的坑。很多人会把 API 1:1 包装成 MCP 工具，结果产生了大量细碎的低级工具。

Anthropic 的建议：按用户意图来组合工具。

一个 create\_issue\_from\_thread 工具，远好过：

```
get_thread → parse_messages → create_issue → link_attachment
```

更少的、描述更准确的工具，Agent 用起来更好，出错更少。

3. API 曲面超大时：用"代码编排"模式

AWS、Cloudflare、Kubernetes 这类服务有上千个 API，用意图分组根本覆盖不完。

Anthropic 推荐的解法是：暴露一个"写代码 + 沙箱执行"的薄接口。Agent 写一小段脚本，Server 在沙箱里执行，只把结果返回。

Cloudflare 就是这么做的——用 2 个工具（search+execute）覆盖了约 2500 个 API，仅占约 1K token。

4. 用 MCP Apps 和 Elicitation 提升交互体验

MCP Apps 是首个官方协议扩展，允许工具返回可交互的图表、表单、Dashboard，直接在对话界面里渲染。使用 MCP Apps 的 Server，留存率明显更高。

Elicitation 让 Server 在工具调用中途向用户请求输入：

* Form 模式：渲染原生表单，用来请求缺失参数、确认危险操作
* URL 模式：跳转浏览器完成 OAuth、支付等需要安全凭证的操作

这两个机制都让用户保持在操作流程中，而不是被踢到一个单独的设置页面。

> MCP Apps 和 Elicitation 的细节可参考：
>
> https://modelcontextprotocol.io/extensions/apps/overview
>
> https://modelcontextprotocol.io/specification/2025-11-25/client/elicitation

5. 依托标准化 Auth，别自己造轮子

MCP 最新规范支持 CIMD（Client ID Metadata Documents），让用户首次授权流程更顺畅，大幅减少意外的重复授权弹出。

对于云端 Agent 怎么持有和复用 token，Anthropic 推荐使用 Claude Managed Agents 的 Vaults：注册一次 OAuth token，后续每次会话自动注入并刷新。不需要自己搭 Secret 存储，也不用每次调用都传一遍 token。

---

# Client 侧也有文章：两个技巧省掉大量 token

如果你在构建 MCP Client，Anthropic 给出了两个经过测试的优化模式：

Tool Search（按需加载工具定义）

传统做法是启动时把所有工具定义都塞进 Context。Tool Search 改成运行时检索：用到哪个工具，再把它的定义拉进来。Anthropic 测试下来，这个方式可以把工具定义的 token 消耗削减 85% 以上，同时保持很高的工具选择准确率。

Programmatic Tool Calling（代码沙箱处理工具结果）

不把工具返回的原始数据直接喂给模型，而是让 Agent 在代码沙箱里循环、过滤、聚合多次调用的结果，只把最终输出传给 Context。对于复杂的多步工作流，这可以减少约 37% 的 token 消耗。

两个技巧叠加使用，效果尤其明显。

---

# MCP + Skills：工具能力 × 使用知识，才是完整的 Agent

回到我们之前讨论的核心观点——这里 Anthropic 官方再次印证了它：

MCP 解决的是工具怎么暴露的问题——给 Agent 提供"手"，让它能触达外部系统。

Skills 解决的是工具怎么用的问题——给 Agent 提供"经验"，告诉它走什么流程、避哪些坑。

两者是正交的，不是竞争关系。最有能力的 Agent，往往同时使用了两者。

Anthropic 推荐了两种组合方式：

打包成 Plugin：把 Skills 和 MCP Server 一起放进 Claude Code 的 Plugin 里，工具和操作知识同步分发。

从 Server 下发 Skills：MCP Server 直接附带 Skills，客户端自动继承操作知识。Canva、Notion、Sentry 等平台已经在这样做了。

博客还提到，MCP 社区正在为这种"从 Server 直接下发 Skills"的方式制定标准扩展，一旦落地，任何兼容客户端都能自动继承这套知识。

---

# 最后：MCP 的复利效应

Anthropic 在博客的最后用了一个词："compounding layer"——复利层。

今天构建的一个高质量 Remote MCP Server，随着越来越多的客户端采纳协议、越来越多的扩展落地，它的覆盖范围和能力会持续增长，而你不需要额外发布任何东西。

这不是在给 MCP 打广告，而是在描述一个网络效应的逻辑：每一个接入 MCP 生态的 Server，都在加强整个生态，也在加强自己。

如果你现在正在考虑怎么让 Agent 连接你的系统，Anthropic 的建议是：

> 如果你的目标是让云端生产环境的 Agent 访问你的系统，就认真构建一个 MCP Server——把它做好，用这篇博客里的设计模式。

MCP、CLI、Skills，每一层都有它不可替代的位置。分层理解，组合使用，才是正确答案。