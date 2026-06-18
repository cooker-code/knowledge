> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020113_Agent协议/020113_核心知识点/Agent协议连接层与生态边界|Agent协议连接层与生态边界]]
---
title: Anthropic 买下 Stainless，真正想补的是 Agent 的连接层
author: Vibe编码
date: VibeCoderVibeCoder
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODkzOTY3MA==&mid=2247485381&idx=1&sn=a23869f07ebc22f0447ab435b3252251&chksm=c4dd84080afaeea7fabda80e27901c94421fc4e6ba62919963f71ec2f4eaca9e08eb8e4b2b5f&mpshare=1&scene=24&srcid=0520QEhjfcecbAgxW6QDHEKN&sharer_shareinfo=01a3e8892d55bda1036e1e0db64f0889&sharer_shareinfo_first=01a3e8892d55bda1036e1e0db64f0889#rd
---

Anthropic 在美国时间 2026 年 5 月 18 日宣布收购 Stainless。这个消息乍看没那么劲爆：Stainless 做 SDK、文档、CLI、MCP Server 生成，听起来像开发者工具链里的一个细分环节。

但如果把它放到 Claude Platform、Claude Code、MCP 和企业 Agent 的方向里看，这笔收购的信号很明确：Agent 要进入真实工作流，模型本身只是一层。它还需要稳定地连接 API、读文档、调工具、处理权限和更新 SDK。

## Stainless 是什么

Stainless 成立于 2022 年，创始人 Alex Rattray 之前在 Stripe 做过工程相关工作。它的主线产品是从 OpenAPI spec 生成生产级开发者接口。

它生成的不只是一套 SDK。Stainless 文档里写得很直接：从 OpenAPI spec 生成 SDK、docs、CLI、MCP server，以及 Terraform provider。SDK 语言覆盖 TypeScript、Python、Go、Java、Kotlin 等常见栈。

这件事对 API 公司很重要。API 一旦变动，SDK、文档、示例、测试、发布包都要同步。手工维护多语言 SDK 很容易失控，尤其 AI 平台和云服务 API 变化频繁。Stainless 把这件事收敛到一个以 API spec 为中心的生成和发布流程里。

Stainless 客户页给了一个很直观的量级：1K+ live SDKs，每周 110 million+ package downloads。客户名单里有 Anthropic、OpenAI、Cloudflare、Replicate、Modern Treasury、MUX、Weights & Biases 等公司。这说明它已经在高频生产链路里跑起来，远不只是一个玩具生成器。

## 这次收购最硬的信息

Anthropic 官方公告说，Stainless 从早期开始就生成了每一个官方 Anthropic SDK。这个细节很重要。Anthropic 买的是自己 API 开发者体验里已经在用的一块底座。

Stainless 自己的公告更直白。加入 Anthropic 后，团队会聚焦 Claude Platform capabilities 和 connecting agents to APIs，同时关闭所有 hosted Stainless 产品，包括 SDK generator。新注册、新项目、新 SDK 都不再开放。已有客户仍然拥有已经生成的 SDK，可以继续修改和扩展。

TechCrunch 报道里还有一个竞争维度：Stainless 被 OpenAI、Google、Cloudflare 等使用；交易金额官方没披露，但 The Information 此前报道谈判价格超过 3 亿美元。这个数字来自媒体报道，足够说明市场怎么看这层基础设施。

## 为什么 Anthropic 要买

Claude 这类产品正在从聊天框走向 Agent。聊天框里，模型回答得好就够了。Agent 要做事，就得进 GitHub、Slack、数据库、内部系统、支付、CRM、工单、云平台。

问题来了：这些系统怎么接？

Anthropic 2024 年推出 MCP，就是想把连接方式标准化。MCP 官方公告里说得很清楚：每个新数据源都单独集成，会导致碎片化，规模化很困难。MCP 提供一个开放标准，让 AI 系统能连接数据源和工具。

协议解决的是共同语言，Stainless 解决的是生产问题。API 公司不可能手写几百个高质量 MCP Server，也不可能在 API 每次变化后手动修所有连接器。Stainless 的位置，就是把 API spec 变成 Agent 能用的接口。

## 真正有意思的是 MCP 生成方式

Stainless 的 MCP 文档里有个设计点值得写。它没有把每个 API endpoint 都暴露成一个独立工具。那种做法看起来直接，但 endpoint 一多，上下文会膨胀，模型也更难选对工具。

Stainless 生成的 MCP Server 采用 code tool architecture，主要包括代码执行工具和文档搜索工具。Agent 可以先查文档，再写 SDK 调用代码，然后执行。复杂任务也能放到一次代码工具调用里完成。

这其实更接近开发者真实使用 API 的方式。人类工程师通常会先读文档、理解对象模型、写几行代码跑一下，避免面对几百个按钮逐个试错。Stainless 把这个流程封装给 Agent 用。

## 对 Claude Code 意味着什么

这笔收购和 Claude Code 是互补关系。

Claude Code 负责理解代码库、执行命令、修改文件。MCP 负责统一工具协议。Stainless 负责把外部 API 变成 SDK、文档和 MCP Server。

三者合起来之后，Claude Code 做集成开发时就更顺：它可以在项目里写业务代码，也可以通过 MCP 接外部服务，还能用 SDK 文档和代码工具完成复杂调用。

更大的想象空间在企业里。企业内部有很多 API，本来只给人类开发者看文档。以后这些 API 也要给 Agent 用。谁能把内部 API 快速变成可审计、可安装、可更新的 Agent 工具，谁就能让 Claude 进入更多真实工作流。

## 竞争面也很现实

Stainless 原本是一个公共供应商。它服务的不只是 Anthropic。客户页和媒体报道都显示，OpenAI、Google、Cloudflare 等公司也在相关链路里使用 Stainless。

收购后 hosted 产品关闭。这会带来一个直接结果：Anthropic 把一块共享基础设施收进了自己平台。其他团队要迁移、替换，或者重建类似能力。

这个动作并不罕见。模型公司越往应用和企业落地走，越会发现很多关键能力不在模型参数里，而在开发者体验、工具协议、权限、连接器、SDK 和平台分发里。Anthropic 买 Stainless，就是把其中一块补上。

## API 文档会变成 Agent 文档

过去 API 文档主要给人看。写得清楚，示例能跑，SDK 好用，就算不错。

Agent 时代会多一层要求：文档要能被模型检索，接口要能被工具描述，错误要能被自动修复，权限要能被安全约束，调用过程要能审计。

这会改变 API 公司做开发者体验的方式。未来发布 API 时，不只要问开发者能不能接入，还要问 Agent 能不能可靠调用。

Stainless 这种工具的价值就在这里。它把 API 产品化，从人类 SDK 推到 Agent 工具。Anthropic 收购它，是在为 Claude Platform 补一段长期会越来越重要的管道。

## 总结

这次收购最值得看的，是 Anthropic 把 Agent 落地需要的连接层往自己手里收。

模型会回答问题，只是第一步。Agent 要完成任务，需要进入真实系统。真实系统靠 API 暴露能力，API 又需要 SDK、文档、MCP Server 和工具协议包装出来。

Stainless 正好在这条链路上。

我的判断是，接下来 AI 公司会继续补开发者基础设施。模型、协议、SDK、连接器、权限和审计会绑得越来越紧。未来的 API 文档，也要给 Agent 用。