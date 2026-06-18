> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: Hermes Agent v0.11.0 来了：一次把 CLI、插件、平台与 GPT-5.5 全部重做
author: 码间 AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzYzOTE2NTczMA==&mid=2247485253&idx=1&sn=f92c96951e2f43f4c44b8764e8e13763&chksm=f17f127ca2ea29360858eca2e7f167aac38290a15f795335e2f1a9cd904b5398832f2787e8c2&mpshare=1&scene=24&srcid=0424Uy1ftf6Ag3ddj4gVcNPh&sharer_shareinfo=186e2bcd6ff637fffc565d097fe89d4c&sharer_shareinfo_first=186e2bcd6ff637fffc565d097fe89d4c#rd
---

原帖来源：Nous Research 官方 X
作者：码间AI楠哥

PART 01

## PART 01 这次更新，为什么叫 Interface Release

如果你最近一直在关注 Hermes Agent，这次 `v0.11.0` 可以说是一次真正意义上的“大版本感”更新。

Nous Research 在官方发布里直接把它命名为 **The Interface Release**。表面上看，它像是在升级交互层；但如果你顺着 release note 往下看，会发现这不只是把界面做漂亮了，而是把 **CLI、传输层、插件机制、平台接入、模型路径** 一口气整体抬升了一个台阶。

更关键的是，这个版本还把此前 `v0.10.0` 没有完全展开的更新一并合并进来。官方给出的统计非常夸张：**自 v0.9.0 以来，累计 1556 次提交、761 个合并 PR、1314 个变更文件、22 万+ 新增代码**。

这意味着，Hermes Agent 正在从“一个能跑的 agent”加速变成“一个可扩展、可运营、可协同的 agent 平台”。

楠哥说：很多 AI Agent 项目的问题，不是 demo 不够炫，而是交互层、扩展层、模型层和消息层彼此割裂。Hermes 这次更新最有价值的地方，是它开始把这些层真正接在一起。

PART 02

## PART 02 最值得关注的五个变化

### 1. 交互式 CLI 被彻底重写了

这次最核心的升级，是新的 Ink-based TUI。

官方把 `hermes --tui` 重构成了一个基于 **React/Ink** 的交互式终端界面，底层通过 Python JSON-RPC 后端 `tui_gateway` 驱动。它不是简单换皮，而是把整个终端交互体验往“长期工作台”方向推进了一步。

从 release note 能看到几个很实用的点：

* sticky composer，输入区更稳定
* 流式输出与复制体验更顺手
* 状态栏能看到耗时与 git branch
* `/clear` 这种细节命令也做了确认机制
* 子代理运行时的可观察性也更强

如果你真的把 Hermes 当生产力工具，而不是偶尔跑一下命令，这类升级的价值会非常大。

### 2. 传输层被抽象出来了，架构味道更重了

另一个很关键的变化，是官方把原来很多 provider 适配逻辑，从 `run_agent.py` 中抽离，做成了新的 `agent/transports/` 架构。

这背后意味着两件事：

第一，Hermes 不再只是“兼容几个模型 API”，而是在认真建设一套统一的 transport abstraction；第二，这让后续新增 provider、切换接口形态、适配不同推理协议都更容易。

这一层更新直接带来了 **原生 AWS Bedrock 支持**，并且把 `AnthropicTransport`、`ChatCompletionsTransport`、`ResponsesApiTransport`、`BedrockTransport` 等路径都明确拆了出来。

楠哥说：很多 agent 工具越做越乱，就是因为 provider 接入一开始图快，后面越补越缝。Hermes 这次把 transport 做成正式架构层，说明团队已经在为更长周期的演进做准备。

### 3. 推理路径继续扩容，而且不是小修小补

官方在 release 里提到新增了 5 条 inference path，包括：

* NVIDIA NIM
* Arcee AI
* Step Plan
* Google Gemini CLI OAuth
* Vercel ai-gateway

此外，Gemini 也被进一步接入 native AI Studio API，以提升性能。

这类更新的重点，不只是“又多支持几个模型供应商”，而是 **Hermes 的模型接入方式变得越来越平台化**。你会发现它在逐步具备一个成熟 agent runtime 应有的模型编排能力。

### 4. GPT-5.5 已经接进来了

这次更新里，最容易被转发的一条，应该就是：**GPT-5.5 over Codex OAuth**。

官方说明写得很直白：OpenAI 新的 GPT-5.5 reasoning model，已经可以通过 ChatGPT 的 Codex OAuth 在 Hermes 里直接使用，而且模型选择器还接了 live model discovery，新模型上线后不必等手动更新目录。

这件事的意义不只是“又支持一个新模型”，而是说明 Hermes 正在成为一个可以快速承接新推理模型的统一入口。

对于已经把 Claude、Kimi、Gemini、MiMo、OpenAI 等模型混合使用的人来说，这个价值非常现实：你不需要每次围着新模型重做一套工作流，Hermes 直接把它吸纳进现有体系里。

### 5. Hermes 正在从单体工具走向生态平台

这次 release 里，另一个非常值得重视的方向，是 **插件表面（plugin surface）大幅扩大**。

官方提到，插件现在可以：

* 注册 slash commands
* 直接调度 tools
* 在 `pre_tool_call` 阶段阻断工具执行
* 重写 tool result
* 转换 terminal output
* 增加 image generation backend
* 为 dashboard 增加自定义 tab

这已经不是“小插件系统”了，而是在朝真正的 extension platform 走。

如果一个 agent 框架能让第三方既接命令、又接工具、还能改 UI 和输出，那它的上限就不只是“作者自己继续迭代”，而是社区能不能一起往上堆生态。

PART 03

## PART 03 这次更新还透露出哪些信号

除了上面几个 headline feature，这次 release 还有几个信号同样值得看。

### QQBot 成为第 17 个支持的平台

官方这次新增了 QQBot 适配，说明 Hermes 在消息平台这条线上还在持续扩张。对于国内用户来说，这件事的象征意义会比英文社区更强，因为它说明 Hermes 并没有把自己锁死在 Telegram、Discord、Slack 那一套海外协作工具链里。

### `/steer` 让 agent 运行中途也能纠偏

`/steer <prompt>` 这个能力很值得关注。它允许你在 agent 正在执行时，给它一个中途提示，而且不会直接打断整个 turn。

这其实非常符合真实工作流：很多时候你不是一开始就知道最优任务描述，而是在 agent 执行过程中才发现方向需要微调。Hermes 这次把这种“中途纠偏”做成正式能力，说明它越来越贴近重度使用者的操作习惯。

### Shell hooks、webhook direct-delivery、dashboard plugin system

这些能力放在一起看，会发现 Hermes 已经不仅是一个“聊天式 agent”。

它开始变成：

* 能嵌进脚本生命周期的自动化中间层
* 能作为 webhook 直达消息平台的通知枢纽
* 能被 dashboard 插件体系进一步 UI 化

也就是说，Hermes 正在从 agent 本体，演进为一个可以承接自动化、通知、平台集成和人机交互的基础设施层。

PART 04

## PART 04 为什么这对国内开发者尤其有参考价值

国内很多人看 agent 项目，往往第一眼盯的是模型能力，第二眼看工具数量，但真正决定能不能长期用下去的，通常是下面几个问题：

* 交互界面能不能每天都用
* 模型接入是不是足够灵活
* 平台通知和消息入口是不是顺手
* 插件和自动化能力能不能沉淀成自己的工作流

Hermes v0.11.0 这次最有参考意义的地方，就是它在这些“长期可用性”维度上一起发力了。

你会看到它既在补底层架构，也在补人机界面；既在做 provider 抽象，也在做 dashboard 和平台入口；既在增强工具运行能力，也在增强生态扩展能力。

这恰恰是一个 agent 真正走向成熟时最该补的部分。

楠哥说：如果你只是想试试看 Agent，会被模型 headline 吸引；但如果你真的要把 Agent 接进日常工作流，最后拼的一定是交互、稳定性、扩展性和平台兼容。Hermes 这次更新，已经明显在往这个方向收口。

PART 05

## PART 05 我对这次 Hermes 更新的判断

我的判断很直接：**Hermes Agent v0.11.0，不只是一次功能堆叠，而是一次“产品形态确认”。**

它让人看到了一个越来越完整的轮廓：

* CLI 不再只是命令入口，而是长期交互界面
* 模型接入不再是散装适配，而是统一 transport 架构
* 插件不再只是边角能力，而是生态扩展接口
* Dashboard 不再只是附属页面，而是可演进的前台系统
* Messaging platform 不再只是通知通道，而是 agent 的真实工作入口

再叠加 GPT-5.5、Bedrock、QQBot、Webhook、Shell hooks 这些节点，这个版本的确配得上官方给它起的名字：**The Interface Release**。

PART 06

## PART 06 结尾

如果你最近正好在看 AI Agent 框架，我会建议你认真翻一下这次 Hermes Agent 的 release note。

因为它给出的不是单个 feature 的惊喜，而是一条很清晰的路线：**Agent 接下来真正要竞争的，不只是模型智商，而是整套接口层、交互层、协同层和扩展层的成熟度。**

Hermes v0.11.0，就是一个非常典型的样本。

关注码间AI楠哥，追踪 AI 前沿动态。

THANKS FOR READING

⚡ 爱马仕 · Hermes Agent 技术分享