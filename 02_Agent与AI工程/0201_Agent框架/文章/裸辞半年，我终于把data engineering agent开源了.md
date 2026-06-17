---
title: 裸辞半年，我终于把data engineering agent开源了
author: 数据杂货铺
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwMjYxNTg1OQ==&mid=2247483849&idx=1&sn=c4aae127da2f2933c98a16ed031c4f26&chksm=c1682fbccb28ac9206d90d317d6cd214b1699c00267bab1f747055d4a2913389443689baef1a&mpshare=1&scene=24&srcid=1028ohi5lAVAstW65ADuiJxY&sharer_shareinfo=e574ecccae916cbbf59625e0047a23ea&sharer_shareinfo_first=e574ecccae916cbbf59625e0047a23ea#rd
---

# 缘起

半年前，我做了个决定，从数据库一头跳进了AI领域，开始了我的 data engineering agent的全新旅程。故事的起点可以追溯到我写的一篇 [DBT 反思文章](https://mp.weixin.qq.com/s?__biz=MzkwMjYxNTg1OQ==&mid=2247483818&idx=1&sn=e398ef43cf946ce448443595257228f6&scene=21#wechat_redirect)。很多人在谈 data agent 时，容易把它等同于 ChatBI。但我的切入点更靠近数据工程侧而非上层报表：我希望先让工程侧“做对”，再让分析与业务“看得对”。虽然距离“开箱即用”的高质量产品还有差距，但我依旧认为现在是开源的时刻：把代码、方法论与这半年来的体感、成功的经验和 Bitter Lesson 摆到台面上，做个更广泛的交流。最初的想法很简单：

1. 数据工程是数据分析的基石。没有高质量的数据工程与语义口径，后续所有分析都容易变成错误率的叠加。
2. 数据工程与数据分析的诉求并不相同：工程更关注元数据、血缘、口径、可维护性与可复用性；分析更关注可解释性、业务闭环与决策速度。
3. 我选择从 “Claude Code for Data Engineers” 的产品形态出发：CLI + Agent + 可编程上下文，把“上下文构建与评测”变成工程流程，而不是零散附件，最终完成了一个带有feedback的可迭代的命令行开发工具

# Contextual data engineering

传统的data engineering pipeline就是数仓分层以后，从表到表的加工，最终转化成BI的dashboard，但是随着AI的数据需求增长，数据工程团队的交付转变为：面向业务的 AI Chatbot、面向系统的 Agent API，以及面向分析的 BI Dashboard，我认为这三者不应分成不同的系统，而是作为一个统一的 Agent 被交付，同时它也内置了semantic layer的指标定义，表的元数据信息，以及其他可以参考的知识。

> From building tables to deliver agents

过去很多 ChatBI 的做法，是构建一个全局 RAG，或是一个可控性不足的对话层；仅依赖“消费层数据”很难支撑持续的数据与语义迭代。真正可持续的路径，是将上下文（context）作为一等公民：在 Agent 中统一管理与进化。而**Datus[1]**就提供给了data engineer持续更新的开发工具来实现**context engineering[2]**

## Datus architecture

Datus 的核心是一个 Context Engine，用于组织 metadata、reference SQL、metrics、success stories 等关键上下文。它既支持由模型自动生成，也支持工程师人工编辑；在服务形态上既可通过 agentic mode（subagent）这种完全模型规划调度的模式直接响应，也可以tools 和llm编排成workflow提供一些稳定的API服务。

对外我们提供 Datus CLI，让数据工程师以类似 Claude Code 的体验开发与维护上下文，并生成 subagent 供分析师或业务侧以 Datus-chat（一个web chatbot） 方式使用；用户反馈会回流到上下文中，驱动持续优化。当 subagent 在某一场景达到稳定与可复用，便可进一步对外导出 API，供其它 agent 或微服务调用。

## How data context organized

我们的 Context Engine 从两个维度组织信息：

* 物理维度（Physical/catalog）：从 catalog service 或数据库获取真实的表结构，按照 catalog → database → schema → table 的层级构建；在此之上可将 semantic model 贴附到 table/view，补充维度、度量与口径说明。
* 语义维度（Logical/subject）：围绕业务域 →一级主题 → 二级主题 的层次；在主题树中承载细粒度指标、可复用的 reference SQL 与外部文本知识等。

在 Datus CLI 中，你可以通过@catalog 与 @subject 浏览和编辑上述两类上下文，通过/gen\_semantic\_model /gen\_metrics /gen\_sql\_summary等subagent命令自动生成context， 也可以使用 datus-agent bootstrap-kb 从历史SQL文件和 success story对中可批量初始化、冷启动知识库。借助 subagent，可以为具体场景定义 scoped context——它从全局上下文中精选与该域强相关的子集，从而实现更精确、 domain-aware 的交付。

## Subagent and feedback loop

Subagent 建立在“工具集合 + scoped context”之上，通常面向一个明确的使用场景（例如某条业务线下某个二级主题的专用问答）。一个subagent 的上下文，可能包含约 10 张关键表、20 个核心指标、30 条高价值 reference SQL 以及若干规则；这些指标与 reference SQL 多从历史 SQL与success stories 自动抽取生成，工程师可在此基础上继续人工校准。我们采用经过增强的 **MetricFlow[3]** 格式存储指标，既有利于上下游解析，也提升了与其他 BI 工具协作的可扩展性。

subagent划分了更小的状态空间，从而让tools的调用准确提升，也可以在外层更容易的增加意图识别进行调度，而每一个subagent的迭代都是可以按照下面流程：Ad-hoc 探索与开发 → 生成场景相关的指标与 reference SQL → 配置 subagent 并交付 chatbot → 用户对话与反馈回流 → 优化与固化 data context。随着这个闭环不断运转，subagent 的准确性与覆盖度会稳步提升，直至可以作为 API 在更大范围内复用与编排。

更详细的步骤请参考**官方文档 [4]**

# Bitter lessons

### Bitter lesson 1：相信模型，workflow是agentic的补充，而非反过来

Datus 的最初版本是围绕 workflow 构建的：我专门设计了一个 reflection 节点来约束模型遵循 ReAct/**reflexion[5]** 的方法，在早期确实起到过“强制规范”的作用。但随着基础模型的 agentic 能力持续增强，从测试benchmark中看，与其在复杂编排里兜转，不如直接为模型提供恰当的 tools 与高质量的 context，效果更直接、误差更少。自此，workflow 在体系中的角色被主动下调为“兜底方案”：在 RL 训练尚未充分、样本稀疏或需要快速稳态的场景下，通过 parallel + selection 的流程提高准确率是务实选择，但它不再是系统设计的中心。更重要的是，RL 的训练单元也从workflow trajactory 回归为更自然的 agentic loop，也就是最终以 subagent 作为训练与评测的基本单位；在基座模型强度足够的前提下，首轮 SFT 甚至可以省略，直接进入 RFT 与回归评测的闭环，之前人工写CoT的路径也被完全抛弃了，因为大部分写的CoT质量可能还真的不如用SOTA的Claude跑出来的结果。

---

### Bitter lesson 2: Native tools的重要性和 MCP的定位

数据库周边的 MCP 生态十分活跃，我们曾经接入过 Snowflake、StarRocks、DuckDB 等多种 MCP 实现，但不同实现的接口风格与语义差异，给统一的 CLI 体验与长期维护带来了显著成本，也让 RL 的泛化学习承受额外麻烦。回到工程本质，采用原生数据库工具（Native tools）作为主路径，更有利于稳定语义边界、降低工具类型的复杂度，并为后续的 RL 与评测提供一致的行为基础。在新框架下，MCP 被明确为“扩展层”：用于挂接第三方 API、生态组件与非关键路径操作（比如metricflow的指标层，还有未来的**airflow[6]**，**数据质量工具[7]**），既保留开放性，又不侵蚀核心链路的确定性。一个 agent 的独特性，更多应体现在工具（及其spec）与 system prompt 策略的稳健设计上，而非盲目扩展一大堆mcp，mcp是一个特别好的quickstart，而针对场景精心裁剪/整理的工具才可以让你持续优化。

---

### Bitter lesson 3: SQL开发效率并不是数据工程效率瓶颈

还有一个最初的误解是在跟一线数据工程师的交流中感受到的，写 SQL 的效率并非主要痛点；多数工程师并不认为 AI 在通用 SQL 生成上比他们更快、更准。真正的瓶颈在于：不断变化的新需求沟通、严谨而繁琐的数据验证流程，以及面对不熟悉表结构与血缘关系时的理解成本。比如一天接了5个需求，每个需求写了SQL提交以后，等结果回来再分析修改，然后最终交付出去以后业务还需要做一些字段变更或者数据补全。因而更有效的路径，是让系统能够快速召回与复用历史 SQL 与口径沉淀，在此基础上进行小幅改写，并把“检索—执行—观测—反思”的验证链路机制化、持续化。进一步地，把这些能力封装为特定场景的 subagent，让工程师能够交付一个在 scoped context 内可维护、可持续提升的“SQL chatbot”，以此降低跨角色沟通成本；随后再通过用户反馈不断补充指标、调整 reference SQL 分类与引入规则，形成可回归评测的稳态改进。最终交付一个稳定的chatbot 或者 API服务。

---

### Bitter lesson 4: ChatBI依然困难，但无法持续提升的Agent是必定失败的

SQL 准确性仍然是难点，将agent 直接交给缺乏数据判断力的终端用户往往会被模型的随机性误导，Next token prediction的机制暂时限制了这个场景的泛用性，过度推广反而会导致信任快速流失产品失败。因此，实际可行的落地场景首先集中在分析师自助：他们具备口径判断能力，能够在多轮对话中校正语义与意图，模型则承担重复性的检索、执行、观测与反思工作。许多 ChatBI 项目停留在第一个业务场景的核心原因，不在“问不出来”，而在“上下文无法在互动中持续提升”，或者用户在尝鲜之后放弃改进。Datus 的应对策略，是把数据工程师与分析师的真实协作过程转化为上下文建设与修复的流水线，用可回归的评测集固化“正确”、把“错误”转化为训练样本，让模型始终工作在可理解、可演化的上下文中；只有这样，Agent 才能在准确性与覆盖度上实现可持续提升，并真正走向规模化。

# Our Vision

## 为什么要开源？

我被好几个朋友问过同样的问题：“这个项目有必要开源吗？”

我也认真考虑过，直接做 SaaS 也许更快产生收入、更容易控节奏、也少很多代码质量和重构额外压力。

但是在 AI 时代，代码已经很难成为长期门槛，即使是闭源的产品，也能够被快速的复制。真正稀缺的是清晰可复用的架构、产品品味与社区形成的合力。开源的意义不只在代码，而在于聚集同路人：大家在分歧中迭代、在争论中收敛，才能推动产品的落地和一个通用的产品形态的构建(这里感谢一下某头部游戏公司&连锁餐饮公司的早期反馈和支持)。

> Talk is cheap, code is also cheap, but community values.

这大概是我第三段或者第四段开源项目的经历，但我依旧记得我大学校园里面作为linux协会的会长每年给新生发我们自己刻的ubuntu光盘的日子，我的第一份工作也开源的key value database Tair（应该是国内最早一批 data infra 的开源项目），再到一步步伴随着StarRocks社区的成长壮大，可能开源对我只是一个自然选择。

## The future of open source data engineering agent

严格来说，我还没有在市场上找到专注“面向工程侧”的开源数据智能体。大家更关注面向报表与业务分析输出的 Agent（这很合理），但数据工程侧的上下文治理、evaluation 与 continus learning 形态的依然是Data agent发展的瓶颈。我希望Datus能够补上这一块板。

相信所有data团队都在思考如何AI化的问题，在我心中的top3的方向：

* 所有的Agent都在尝试更加直接的交付价值，而非交付工具，所谓的**Service as software[8]** 本质是在encode human expertise in process of doing work， 对于数据工程而言，有很多留在人脑子里面的hidden knowledge，他们有的是数据的潜规则，指标的黑话，SQL开发的标准，数据质量的要求，这些东西只有被系统化到agent内部才能实现scale，我们做的两棵context tree就是第一步。
* 随着基础的开源模型的agent能力越来越强，同时RL infra也越来越成熟（还有Thinkingmachine发布了**Tinker[9]**这中产品会提供Serverless training的服务），而NL2SQL其实是天生就有Ground truth的极佳RL场景，但是问题在环境的构建和问题的收集，构建benchmark（内部的评估标准）和feedback机制，是持续进行RL的基础，也是真正让模型理解公司内部数据的基石。
* 将数据治理的规则从事后转向事中是agent的另一个重要意义，不是在数据已经变成一片泥沼以后再去做一次又一次突击项目进行治标不治本的运动，而是把很多规则内化到数据开发中，数据的工程化（版本管理，质量检测），大模型不能保证准确，但是只要给他规则和可验证的工具，他一定是一个好的 problem detector。 这也是我理解的从面向人的数据工程转变向面向模型的数据工程(治理)

That‘s the future of the (Contextual) data engineering.

最后，Datus已经采用Apache2.0协议开源，地址: **https://github.com/Datus-ai/Datus-agent**，欢迎star watch fork三连，文档地址**https://docs.datus.ai**

### 参考资料

[1]

Datus: *https://datus.ai/*

[2]

context engineering: *https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents*

[3]

MetricFlow: *https://github.com/Datus-ai/metricflow*

[4]

官方文档 : *https://docs.datus.ai/getting\_started/contextual\_data\_engineering/*

[5]

reflexion: *https://arxiv.org/abs/2303.11366*

[6]

airflow: *https://airflow.apache.org/*

[7]

数据质量工具: *https://github.com/sodadata/soda-core*

[8]

Service as software: *https://www.youtube.com/watch?v=hxNqhKbYoHE*

[9]

Tinker: *https://thinkingmachines.ai/blog/announcing-tinker/*