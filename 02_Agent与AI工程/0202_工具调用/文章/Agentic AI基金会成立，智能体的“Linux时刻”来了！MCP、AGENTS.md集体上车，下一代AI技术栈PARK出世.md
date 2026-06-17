---
title: Agentic AI基金会成立，智能体的“Linux时刻”来了！MCP、AGENTS.md集体上车，下一代AI技术栈PARK出世
author: InfoQ
date: 
url: https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651266240&idx=1&sn=a1cc0433c22138f7c2bdc31a3af41bc9&chksm=bc11fced2eba831f2636bf0a6ce2b845f3c2883524c17ab83afaff253a91e3c681cd7e712784&mpshare=1&scene=24&srcid=1211Ckt1IjoA3oSkiIS0RdcE&sharer_shareinfo=a231daccb01912c37b43de3fceeb56b2&sharer_shareinfo_first=a231daccb01912c37b43de3fceeb56b2#rd
---

整理 | 华卫

刚刚，Linux 基金会正式宣布推出智能体 AI 基金会（Agentic AI Foundation，简称 AAIF）。据公告披露，AAIF 定位为 AI 智能体（AI agents）相关开源项目的中立托管平台，全球几乎所有科技巨头均已签约成为该基金会成员。Anthropic、OpenAI 与 Block 三家公司作为联合创始成员，将贡献三大开源项目，构成基金会启动初期的支柱。

目前，AAIF 基金会的成员名单包括亚马逊云科技、Anthropic、Block、Cloudflare、谷歌、微软、OpenAI、思科（Cisco）、IBM、甲骨文（Oracle）、Salesforce、SAP、Snowflake、Hugging Face 等。他们将首次携手，共同制定 AI 智能体的开放标准。

1 初代核心标准已定，无单一成员独占话语权

当前，AAIF 基金会围绕三大开源项目构建：Anthropic 的模型上下文协议（Model Context Protocol，简称 MCP）、Block 的 goose 项目，以及 OpenAI 的 AGENTS.md 规范。三者将协同实现 AI 智能体与外部工具的交互标准化，推动跨系统运行能力的统一。

在启动仪式相关访谈中，Linux 基金会执行董事 Jim Zemlin 直言不讳："我们的目标是避免未来出现 ' 围墙花园 ' 式的专有技术栈，工具连接、智能体行为与协同调度均被少数平台锁定。通过将这些项目纳入 AAIF 统筹，我们得以针对 AI 智能体制定专属的互操作性标准、安全模式与最佳实践。"同时，Zemlin 表示，“AI 正迈入全新发展阶段，对话式系统正向可协同工作的自主智能体演进。短短一年内，MCP、AGENTS.md 与 goose 已成为开发者构建新一代智能体技术的核心工具。”

这三项技术中，MCP 的知名度和成熟度最高，Anthropic 于一年前将其开源。该协议的核心目标是通过标准化方式连接 AI 智能体与数据源，Anthropic 形象地将其称为 “AI 领域的 USB-C 接口”。借助 MCP，开发者无需为不同数据库或云存储平台单独搭建定制化集成方案，即可快速对接所有兼容 MCP 协议的服务器。高通 AI 产品负责人 Vinesh Sukumar 透露，“许多生产力与内容创作类任务在边缘设备上完全可以完成，而借助 MCP 协议，用户能够对接多家云服务提供商，高效处理各类复杂任务。”

自发布以来，MCP 已获得广泛应用。据 Linux 基金会数据，目前已部署超过 1 万台 MCP 服务器，Claude、Cursor 编辑器、微软 Copilot、Gemini、VS Code 及 ChatGPT 等主流产品均已支持该协议。谷歌在 2025 年 I/O 开发者大会上宣布，将在其开发工具中新增 MCP 支持；此后，谷歌多款产品陆续部署 MCP 服务器，以提升智能体对数据的访问效率。OpenAI 也在 MCP 发布仅数月后便完成了适配接入。

对此，有网友调侃道，“他们可能已经意识到这类技术的商业化变现一团糟，而且由于自身是唯一没有开源模型的实验室，便向开源社区抛出了象征性的好处。”

OpenAI 则贡献了 AGENTS.md 规范，这一标准于 2025 年 8 月推出，旨在为 AI 编程智能体提供项目专属指令支持。据披露，该 Markdown 格式标准已被超 6 万个开源项目采用，Cursor、Devin、GitHub Copilot 及 Gemini CLI 等主流开发框架均已实现兼容。值得注意的是，OpenAI 亦是 MCP 协议的早期支持者。OpenAI 工程师 Nick Cooper 表示，"协议本质上是一种共享语言，能让不同智能体与系统协同工作，无需开发者重复搭建集成方案。我们需要多种协议通过协商、沟通实现价值共创，而这种开放性与互通性决定了行业绝不会由单一供应商、单一平台或单一企业垄断。"

作为 Square 支付平台与 Cash App 的母公司，金融科技企业 Block Block 将开源 AI 智能体框架 Goose 纳入基金会贡献项目。该框架于 2025 年初发布，通过融合语言模型、可扩展工具及基于 MCP 协议的集成能力，为开发者提供结构化的智能体工作流构建方案。该公司 AI 技术负责人 Brad Axen 称，goose 的成功证明开源方案完全能与专有智能体在规模化应用中抗衡，目前每周已有数千名工程师使用该框架进行编码、数据分析与文档编写。

Axen 透露，对 Block 而言，开源 goose 具备双重战略意义，"推向社区能吸引全球开发者参与优化，我们已有大量开源贡献者，他们的每一项改进最终都会反哺公司业务" 。同时，将 Goose 捐赠给 Linux 基金会，既能让 Block 获得社区层面的压力测试反馈，又能使其成为 AAIF 愿景的实践范例，一个可对接 MCP、AGENTS.md 等共享组件的智能体框架。

据介绍，AAIF 的资金来源是“定向基金”，企业可以通过缴纳会员费来捐款，最高的白金会员资格需每年捐35万美元，黄金会员20万美元/年。但 Zemlin 也强调，资金投入不等于控制权，"项目路线图由技术指导委员会制定，任何单一成员都无权单方面决定发展方向。"

2 为何共享标准对智能体至关重要？

这一年，各大企业争相将生成式 AI 融入各类产品与业务流程，同时科技巨头们不断宣称已迈入 AI 智能体时代。尽管 AI 智能体模型的发展路径尚不明朗，但企业在该领域的巨额投入已催生出一批标杆性工具。

UiPath 发布的报告显示，企业对 AI 智能体的采纳率正快速攀升：截至 2025 年年中，约 65% 的组织已启动智能体系统的试点或部署工作，近九成高管计划在 2026 年全年增加相关投入。数据表明，多智能体系统（multi-agent systems）能显著提升业务表现，与传统流程相比，错误率最多可降低 60%，执行效率提升 40%。不过，该报告援引麻省理工学院（MIT）近期研究指出，仅有 5% 的企业从 AI 项目中获得了实质性财务回报。同时，UiPath 警告称，96% 的信息技术专家与安全负责人对 AI 智能体带来的风险升级表示担忧，呼吁行业尽快采取应对措施。

在此背景下，科技巨头们似乎达成了共识：推动行业标准化。即便对于支持度最广的 MCP 协议，其在 OAuth 等基础技术的适配方式上仍存在较大不确定性。若缺乏行业共识，智能体生态可能陷入 "碎片化" 困境，各系统孤立运行、难以互联互通，重蹈早期互联网在开放协议普及前的分散格局。

而 AAIF  基金会的核心使命正在于规避这一风险：通过将 MCP 等协议、goose 等框架及 AGENTS.md 等规范纳入中立平台统筹管理，推动智能体开发框架、云服务提供商与开发者工具的兼容性协同。其目标清晰明确：让下一代 AI 智能体运行于开放互通的标准之上，且这些标准的管理独立于任何单一企业。

Linux 基金会也表示，将以开放之名推动这些关键技术发展，但按照目前的节奏，该组织或许最终会收纳大量尚处萌芽阶段的 AI 工具。

此前 Linux 基金会已发起多个项目，旨在支持关键技术的中立化与互操作性开发。例如，该组织于 2015 年成立云原生计算基金会（CNCF），最初旨在支持谷歌开源的 Kubernetes 容器编排系统，如今已整合数十款云计算工具。这些工具的认证与培训服务为基金会提供了稳定的资金来源，但值得注意的是，Kubernetes 在谷歌广泛推广时已是一项成熟技术。反观当前的 AI 领域，MCP、AGENTS.md 等技术虽热度高涨，但能否在长期发展中保持重要地位，仍有待观察。

3 大家怎么看？

现在行业的一个核心疑问是：AAIF 究竟能成为真正的基础设施，还是仅仅沦为又一个 "品牌联盟"？

对此，Zemlin 表态称，"除了标准的采纳率，全球厂商的智能体产品是否开发并落地这些共享标准，将是衡量其早期成功的关键指标。"在 Cooper 看来，标准的持续演进才是成功的核心："我不希望这些协议成为僵化的存在，被纳入基金会后就停滞两年无进展。它们需要不断迭代，持续吸收新的行业反馈。"

另一个潜在影响更为微妙：即便采用开放治理模式，某家企业的技术实现方案仍可能因发布速度快或市场占有率高而成为默认标准。不过 Zemlin 认为这未必是坏事，他以开源历史为例解释道，"正如 Kubernetes 在容器领域的胜出，这种主导地位源于技术本身的优势，而非厂商的强制控制。"

在前不久的一场开源峰会上，Zemlin 提出，尽管目前只有少数组织在生产中使用 MCP，但 2026 年将迎来真正的企业自动化浪潮：多智能体工作流、学习型编排、验证框架以及确定性和非确定性系统的新融合。他强调，“智能体人工智能的规模并不取决于模型的大小，关键在于你如何构建解决方案。”

当时，Zemlin 亦针对当前 AI 行业的发展发表了看法：“我认为我们并未处于人工智能泡沫之中，但我们可能正处于 LLM 的泡沫之中。”他表示，尽管开源模型成本更低，功能也几乎相同，但封闭模型仍然占据了 95% 的收入，导致每年在专有系统上的支出估计高达 248 亿美元。

Zemlin 还重点介绍了 PARK 技术栈的兴起：PyTorch、AI、Ray 和 Kubernetes。（Ray 是一个开源分布式计算框架，旨在简化 AI 和机器学习 [ML] 工作负载的扩展。）他认为，正如 LAMP 技术栈定义了早期 Web 时代一样，这一代 AI 技术栈也将定义未来的技术栈。他声称，PARK 正在迅速成为大规模 AI 部署的默认平台。他将这一时刻比作 Linux 内核的演进，后者源于全球开发者社区的集体压力，不断推动各种硬件的效率提升。

同时，有网友担忧道，“维护协议恐怕是一项吃力不讨好的工作。”也有网友对 AAIF 当前的三大核心标准工具提出质疑，“MCP 已经在过时了，效率也太低了。目前 MCP 是行业标准，但这并非永远如此。”“试过 goose，但什么都做不了。”

社区内还有许多人已开始期待 AAIF 能做出一些贡献：“AAIF 是否有可能制定一套类似聊天补全 JSON API 的社区共享标准？既然大家似乎都已经在克隆它，如果能有一个公认的标准规范及相应的一致性测试套件就太好了。”

无论如何，对开发者与企业而言，AAIF 基金会的短期价值显而易见：减少定制化连接器的开发时间，提升跨代码库的智能体行为可预测性，同时简化在高安全需求环境中的部署流程。其长远愿景似乎也值得期待：若 MCP、AGENTS.md、Goose 等工具成为行业标准基础设施，AI 智能体领域或将从封闭平台模式，转向开放兼容、自由组合的软件生态。

**参考链接：**

https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation

https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation

https://techcrunch.com/2025/12/09/openai-anthropic-and-block-join-new-linux-foundation-effort-to-standardize-the-ai-agent-era/

https://thenewstack.io/linux-foundation-leader-were-not-in-an-ai-bubble/

**声明：本文为 AI 前线整理，不代表平台观点，未经许可禁止转载。**

今日好文推荐

[“人人都是程序员”的梦该醒了！AI 编程“大逃杀”：Cursor 或成创业公司唯一“幸存者”，“60 分开发者”撑起最后防线](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651265198&idx=1&sn=b9ff2885d2b23b74d2c8596a48ef4e18&scene=21#wechat_redirect)

[JetBrains放弃Fleet：急刹变道打造全新Agentic IDE，与VS Code、Cursor争夺下一代AI编程王座](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651266134&idx=1&sn=e970e2cac9d0d9e5a99b76a486afbab6&scene=21#wechat_redirect)

[谷歌突砍Gemini免费版炸锅，数据养模遭背刺？GPT-5.2突袭Gemini 3，Demis Hassabis：谷歌须占最强位](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651266048&idx=1&sn=bfbaf64fe71ae574d73810b8656ceaf7&scene=21#wechat_redirect)

[从被吐槽“套壳中国大模型”到自研封神！首次揭秘Cursor背后极速代码Agent如何对标GPT5.1 Codex，生成效率提4倍](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651266012&idx=1&sn=f988df738ae56e7165c45dbb349193df&scene=21#wechat_redirect)

活动推荐

AI 重塑组织的浪潮已至，Agentic 企业时代正式开启！当 AI 不再是单纯的辅助工具，而是深度融入业务核心、驱动组织形态与运作逻辑全面革新的核心力量。

把握行业变革关键节点，12 月 19 日 - 20 日，AICon 全球人工智能开发与应用大会（北京站） 即将重磅启幕！本届大会精准锚定行业前沿，聚焦大模型训练与推理、AI Agent、研发新范式与组织革新，邀您共同深入探讨：如何构建起可信赖、可规模化、可商业化的 Agentic 操作系统，让 AI 真正成为企业降本增效、突破增长天花板的核心引擎。