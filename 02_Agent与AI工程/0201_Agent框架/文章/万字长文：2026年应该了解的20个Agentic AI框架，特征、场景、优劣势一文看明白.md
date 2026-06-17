---
title: 万字长文：2026年应该了解的20个Agentic AI框架，特征、场景、优劣势一文看明白
author: 王吉伟
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595946&idx=1&sn=5f34bc9ca880d40d82a49e1fa8a60055&chksm=89561269b724e3d5e03134471baaf2039c26b4cb7132f0c6f615d8c503d00a291734b6b7a4a3&mpshare=1&scene=24&srcid=1211gVtOgBZb3PPeIJz2yXp2&sharer_shareinfo=8a73a2eeaee69c2d2a143e5af30d70bd&sharer_shareinfo_first=8a73a2eeaee69c2d2a143e5af30d70bd#rd
---

* 万字长文：2026年应该了解的20个Agentic AI框架，智能体构建选型综合指南
* 【万字长文】2026智能体选型指南：20款Agentic AI框架优缺点全对比，再也不踩坑
* 2026 Agentic AI框架选型终极手册：20款开源工具+场景适配+技术栈匹配，直接抄作业
* 万字长文：2026年20个Agentic AI框架技术解析，架构、特征、适用场景一文打尽
* 万字硬核｜2026年20个Agentic AI框架深度横评：谁是多智能体协作之王？

全文约1.1万字，阅读时间15分钟

文/王吉伟

随着AI Agent的落地热潮带来广大企业对于智能体企业级应用的热情探索，更能体现这一阶段AI特征并代表组织战略规划的Agentic AI也就火了。

2025年初大家可能还在找智能体去体验，现在人们对于Agent已经司空见惯。再看到哪家厂商推出智能体的新闻，已经没有立即想体验的冲动，对邀请码也没有那么敏感了，反正早晚会开放体验，反正还有其他厂商也要推出智能体了。

从大家常用的聊天机器人到各种耳熟能详的软件应用，要么本身就是一个智能体应用，要么就是在原有基础上推出了智能体模式，又或者上线了大量的AI工作流。从传统软件应用到原生AI应用，应用的Agent化已经不再是趋势，已是正在发生且速度极快的事实。

这些我们能看到的事实，代表了传统AI向Agentic AI的快速演进，也代表了软件应用方式从单纯的“用软件”到现在“连做带用”软件的新范式。

从OpenAI推出ChatGPT到AutoGPT横空出世再到现在国产大模型和开源Agent架构璀璨全球，在这短短3年的时间里，现在基于大型语言模型和先进技术作为多功能推理引擎和动态工具组合的智能Agent，得益于多智能体系统（MAS）和专家系统，已经与传统智能体有了显著的区别。

论文《Agentic AI Frameworks:Architectures,Protocols,and Design Challenges》通过对传统AI和现代AI进行比较，为现代智能Agent下了一个新的定义：

一种具备自主性与协作性的实体，拥有推理与沟通能力，能够动态解读结构化语境、编排工具，并通过分布式系统中的记忆与交互调整行为。

论文地址：https://arxiv.org/html/2508.10146v1

图源：论文《Agentic AI Frameworks:Architectures,Protocols,and Design Challenges》

这个定义，在阐述智能体使用工具自主行动的同时，强调了动态性与分布式交互。

在智能体与时俱进情况下，Agentic AI架构也在快速的迭代与重构，全新的框架则更加注重动态与交互。比如全新的OpenHands就进化成了一款完全重构的开源多智能体框架。

那么，目前流行的和具备潜力的框架都有哪些呢？本文，王吉伟频道盘点了20个Agentic AI框架，并梳理了这些框架的特征与优缺点，帮助大家从技术层面更好地理解Agentic AI。

相关论文，公众号主页发消息 251203 ，获取资源，提供中英双语版。

【**PS：赠书福利见文末**】

**1、CrewAI**

一个多智能体框架，专注于智能体团队(crew)的编排，用于协调自主AI Agent。通过角色分工和流程设计简化多智能体协作，提供Crews和Flows两种方法，分别实现自主智能和精确任务编排，让开发者快速构建适用于各种场景的自主智能体系统。

主要特征：

* 角色驱动架构：每个智能体定义特定角色和技能，实现专业化协作；
* 极简API设计：提供高层简洁性同时保持底层控制精确度；
* 动态任务分配：智能体可根据能力自动认领和执行子任务；
* 内置监控与调试：实时追踪智能体状态和任务执行流程；

应用案例：

* 股票分析系统：创建金融分析师、数据收集员、风险评估师等角色智能体协同工作；
* 软件开发虚拟团队：模拟产品经理、开发人员、测试人员协作完成项目；
* 客户服务中心：多智能体分工处理咨询、投诉、技术支持等不同类型请求；

优势：

* 快速原型开发：几行代码即可创建功能完备的多智能体系统；
* 直观的角色定义：通过简单配置文件明确智能体职责和能力；
* 强大的社区支持：拥有活跃开发者社区和丰富的实战应用案例；
* 灵活的扩展性：支持插件系统，可轻松集成新功能；

弱点：

* 文档相对分散：完整学习路径需自行探索多个资源；
* 对复杂场景的适配：高度定制化需求可能需要深入修改框架源码；
* 资源管理：大规模智能体团队可能面临性能瓶颈。

GitHub链接：https://github.com/crew-ai/crewai

官网：https://www.crewai.com/

**2、AutoGen**

微软开发的开源多智能体框架，专为构建可扩展的多智能体系统、智能体间通信与编排设计，能够自主行动或与人类协作。允许LLM智能体相互对话协作解决任务，支持多语言开发和人类无缝参与。

主要特征：

* 多智能体对话机制：通过智能体间自然语言交流解决复杂问题；
* 模块化架构：支持自定义智能体、工具、内存和模型的灵活组合；
* 跨语言支持：兼容Python和.NET，便于企业级应用开发；
* 异步通信：高效处理多智能体间消息传递，提升系统响应性；
* 大型语言模型集成：无缝对接GPT-4、Claude等主流大模型；

应用案例：

* 软件开发加速：通过多个智能体（程序员、架构师、测试员）协作，将开发效率提升4倍；
* 研究协作：学术团队利用其分布式智能体网络进行数据挖掘和论文生成；
* 内容创作：多智能体协同完成从创意构思到内容生成的全流程；

优势：

* 低门槛高产出：仅需少量代码即可构建复杂多智能体应用；
* 智能体自主决策：减少人工干预，提高任务完成效率；
* 灵活的协作模式：支持链式、并行、递归等多种协作方式；
* 可视化界面：AutoGen Studio提供直观UI，简化开发和调试；

弱点：

* 学习曲线陡峭：高级特性需深入理解多智能体通信机制；
* 资源消耗较大：多智能体并行运行对算力和内存要求高；
* 调试复杂度：分布式智能体系统故障排查难度增加。

GitHub链接：https://github.com/microsoft/autogen

**3、LlamaIndex**

**前身为GPT-Index，**专注于连接LLM与私有数据的开源框架，提供数据摄入、索引、查询和检索的完整流水线，支持与多种应用框架集成，让LLM能够基于用户特定数据进行推理，是构建RAG(检索增强生成)应用的首选方案之一。

主要特征：

* 数据连接器：支持多种数据源(文件、数据库、API等)的无缝接入；
* 智能索引：自动构建高效向量索引，优化检索性能；
* 查询增强：智能重写查询，提高检索精度和相关性；
* 上下文窗口优化：有效管理长文本，突破LLM上下文长度限制；
* 插件系统：支持自定义节点和检索策略，满足特定需求；

应用案例：

* 企业知识库：构建公司内部文档智能问答系统；
* 研究助手：帮助学术人员高效检索和分析文献资料；
* 智能客服：结合企业产品文档提供精准解答，减少人工干预；

优势：

* 降低幻觉现象：通过可靠数据源支持，提高LLM输出准确性；
* 增强上下文理解：使LLM能够利用企业特有知识进行推理；
* 简单易用：提供统一接口，隐藏索引和检索的复杂实现；
* 与智能体集成：为智能体提供可靠的知识来源，增强决策能力；

弱点：

* 依赖高质量数据：输出质量高度依赖输入数据的完整性和准确性；
* 索引维护成本：大规模数据需要定期更新索引，消耗计算资源；
* 与纯智能体框架相比：缺乏自主决策和行动能力，需与其他框架结合。

GitHub链接：https://github.com/run-llama/llama\_index

官网：www.llamaindex.ai

**4、LangChain**

一个用于构建智能代理和基于大型语言模型的应用程序的框架，现已演进为全面支持Agentic AI的开发平台。

它通过标准化接口连接模型、嵌入、向量存储等组件，简化LLM与外部工具、数据的集成，提供强大的提示词管理和链式调用功能，支持实时数据增强、模型互操作性、快速原型开发和生产级特性，拥有活跃的社区和生态系统，是构建LLM驱动应用的首选框架之一。

主要特征：

* 模型无关接口：统一API对接OpenAI、Claude、Llama等主流大模型；
* 提示词工程工具：提供模板化、结构化提示词管理；
* 链式调用：支持多步骤推理和工具调用的无缝衔接；
* 记忆管理：内置多种记忆机制，支持上下文保持和历史追踪；
* 工具集成：轻松连接API、数据库、文件系统等外部资源；

应用案例：

* 智能问答系统：构建知识库问答，支持文档检索和自然语言回答；
* 内容创作助手：辅助撰写报告、文章，支持多轮润色和内容扩展；
* API集成网关：将LLM与企业现有系统连接，提供自然语言接口；

优势：

* 生态系统完善：拥有LangSmith(监控)、LangGraph(工作流)等配套工具；
* 社区活跃度高：丰富的文档、教程和活跃的开发者社区；
* 灵活的定制能力：可轻松调整提示词、模型和工具链满足特定需求；
* 企业级支持：已被多家大型企业采用，证明其稳定性和可靠性；

弱点：

* 过度抽象：某些场景下可能需要深入理解底层实现才能优化性能；
* 配置复杂性：完整应用搭建需要配置多个组件和参数；
* 模型依赖：性能高度依赖所集成的LLM模型质量。

GitHub链接：https://github.com/hwchase17/langchain

官网：https://www.langchain.com

**5、LangGraph**

LangChain生态的图结构框架，专为构建状态化多智能体系统设计，通过图结构实现复杂动态工作流，支持循环、条件分支和持久化，用于构建、管理和部署长期运行的有状态代理，提供持久执行、人类参与、全面记忆等核心优势，支持与 LangSmith和LangChain等产品集成，是开发高交互性、自主性智能系统的关键工具。

主要特征：

* 图结构工作流：使用有向图表示智能体协作流程，直观可视化；
* 状态管理：内置状态跟踪，支持复杂流程中的上下文保持；
* 循环与条件：原生支持工作流循环、条件判断和动态路由；
* 人工干预：允许在关键节点插入人工审核或操作；
* 持久化：支持工作流状态的保存和恢复，便于断点续传；

应用案例：

* 客服自动化：构建智能客服，处理从咨询到问题解决的全流程；
* 业务审批流：模拟多级审批流程，支持条件判断和人工介入；
* 内容生产流水线：多智能体协同完成内容策划、创作、审核和发布；

优势：

* 可视化开发：通过LangGraph Studio直观设计和调试工作流；
* 精细控制：提供对智能体执行流程的细粒度控制和监控；
* 复杂流程支持：轻松管理具有循环、分支的复杂业务逻辑；
* 与LangChain无缝集成：可直接利用LangChain的模型和工具生态；

弱点：

* 学习曲线陡峭：掌握图结构工作流设计需要一定时间；
* 性能考虑：复杂图结构可能带来额外计算开销；
* 生态相对较新：与LangChain相比，第三方集成和社区资源较少。

GitHub链接：https://github.com/langchain-ai/langgraph

**6、MetaGPT**

多智能体协作框架，核心设计理念是模拟软件公司组织结构和工作流程，通过多角色智能体(产品经理、架构师、工程师等)协作，将自然语言需求自动转化为可执行代码和完整项目文档，实现"一句话需求，全套解决方案"。

它以一行需求为输入，输出用户故事、竞争分析、需求、数据结构、API、文档等。内部包含产品经理、架构师、项目经理、工程师等角色，提供软件公司的整个流程和精心编排的标准操作程序（SOP）。

主要特征：

* 角色驱动开发：模拟软件团队角色分工，每个智能体负责特定任务；
* 全流程自动化：从需求分析、设计到编码、测试的完整软件开发流程；
* 智能任务分配：系统自动将任务分配给最合适的角色智能体；
* 文档生成：同步输出PRD、设计文档、测试用例等完整项目文档；
* 代码质量保障：通过多智能体评审和测试确保代码质量；

应用案例：

* 原型开发：快速将产品构想转化为可演示的软件原型；
* 小型应用开发：构建简单Web应用、工具脚本等；
* 学习平台：开发者学习软件开发流程和最佳实践的教学工具；

优势：

* 开发效率革命：大幅缩短从需求到实现的时间，提高生产力；
* 降低技术门槛：非技术人员也能通过自然语言创建软件；
* 标准化流程：遵循行业最佳实践，输出规范的代码和文档；
* 学习工具：帮助开发者理解和掌握软件开发的完整流程；

弱点：

* 适用场景有限：主要适合中小型应用，大型复杂系统支持较弱；
* 代码质量：自动生成代码可能需要人工优化才能用于生产环境；
* 模型依赖：输出质量高度依赖于所使用的LLM模型性能。

GitHub链接：https://github.com/geekan/MetaGPT

产品：https://mgx.dev/

**7、Atomic Agents**

新兴的开源框架，专为去中心化多智能体系统设计，采用"原子设计"原则，支持多个具有特定目标的智能体以分布式方式协调任务，用于构建AI应用。

该架构强调原子性，即每个组件（如Agent、工具、上下文提供者）都是单一用途、可重用、可组合且行为可预测的。基于Instructor和Pydantic构建，使开发者能够使用熟悉的软件工程原则来创建AI应用，特别适合模拟、大规模自动化和研究协作等复杂环境。

主要特征：

* 去中心化架构：无中心控制点，提高系统鲁棒性和可扩展性；
* 模块化设计：智能体由独立的"原子"组件构成，可灵活组合；
* 分布式协调：通过消息传递实现智能体间协作，无需共享状态；
* 自治决策：每个智能体拥有独立决策权，能够自主应对环境变化；
* 动态重组：系统可根据任务需求自动调整智能体组合；

应用案例：

大规模模拟：城市交通、物流网络或生态系统的分布式模拟；

科研协作：多个研究智能体并行探索不同解决方案，共享发现；

去中心化自治组织(DAO)管理：构建自动化决策和执行系统；

优势：

* 高容错性：部分智能体失效不影响整体系统运行；
* 无限扩展性：可轻松添加更多智能体处理更大规模任务；
* 创新的协作模式：突破传统中心化架构的性能和可靠性限制；
* 适合前沿研究：为分布式AI和多智能体系统研究提供理想平台；

弱点：

* 技术复杂度高：理解和应用去中心化设计模式需要深厚技术背景；
* 调试困难：分布式系统的问题定位和调试挑战较大；
* 与传统架构集成困难：与中心化系统集成需要额外的适配层。

GitHub链接：https://github.com/BrainBlend-AI/atomic-agents

**8、Semantic Kernel**

微软开发的轻量级SDK，专为企业级Agentic AI系统设计，旨在帮助开发者构建、协调和部署AI代理和多代理系统。

提供简单易用的API连接LLM与外部工具、数据和API，支持函数调用、语义记忆和规划能力，支持多种语言，具有模型灵活性、Agent框架、多Agent系统、插件生态系统、向量数据库支持、多模态支持、本地部署和企业级准备等关键特性，是构建企业级自主工作流的理想选择。

主要特征：

* 模型无关设计：支持OpenAI、Azure OpenAI、Hugging Face等多种LLM；
* 函数调用：无缝集成现有代码和服务，将其作为智能体可调用的技能；
* 语义记忆：提供结构化记忆存储，增强智能体长期知识保持和检索；
* 多语言支持：兼容C#、Python和JavaScript，适应不同开发团队；
* 企业级安全：内置权限管理和数据保护机制，适合敏感业务场景；

应用案例：

* 企业流程自动化：自动处理采购申请、审批、订单生成等流程；
* 智能报表生成：结合数据库和文档生成财务、销售等专业报表；
* 客户服务升级：智能分析客户问题，自动转接给最合适的团队；

优势：

* 与微软生态深度集成：无缝对接Azure服务、Office 365等企业工具；
* 开发效率高：通过简单插件机制快速构建复杂智能体应用；
* 企业级可靠性：经过微软产品验证，适合关键业务系统；
* 性能优化：针对企业场景进行专门优化，减少延迟和资源消耗；

弱点：

* 平台依赖性：与微软技术栈集成最紧密，其他平台体验略差；
* 学习曲线：对不熟悉微软开发体系的团队有一定学习成本；
* 生态相对较新：第三方插件和社区资源不如其他框架丰富。

GitHub链接：https://github.com/microsoft/semantic-kernel

文档：https://learn.microsoft.com/en-us/semantic-kernel/overview/

**9、RASA（智能体对话框架）**

领先的开源对话式AI框架，专注于构建上下文感知的智能对话系统，近年来演进为支持LLM集成的智能体对话框架，通过结合确定性对话管理与LLM驱动的智能，创建更自然、更智能的交互体验。

支持在多个平台（如Facebook Messenger、Slack等）上构建聊天机器人和语音助手，能够实现上下文对话，帮助开发者构建复杂的交互式应用。

主要特征：

* 对话状态管理：精准跟踪多轮对话上下文，保持连贯体验；
* 意图识别：通过NLU模型理解用户意图和实体；
* 对话策略：支持规则和机器学习两种对话流程管理；
* LLM集成：可接入主流大模型增强理解和生成能力；
* 多模态交互：支持文本、语音、按钮等多种交互方式；

应用案例：

* 客户服务机器人：处理产品咨询、故障排除和订单查询；
* 语音助手：构建智能音箱、车载系统的语音交互界面；
* 智能导购：在线商店中提供个性化产品推荐和购买引导；

优势：

* 对话管理能力：在复杂多轮对话中表现优异，保持逻辑一致性；
* 企业级应用验证：被多家大型企业采用，证明其稳定性和可靠性；
* 可视化开发工具：提供Rasa X等界面，简化对话流程设计和调试；
* 社区与生态：拥有丰富的文档、教程和插件生态；

弱点：

* 开发复杂度：构建复杂对话流程需要掌握专门的DSL和设计模式；
* 对非对话场景支持较弱：主要优势：在对话交互，其他智能体类型支持有限；
* 模型依赖：升级到LLM增强版本需要一定的技术迁移成本。

GitHub链接：https://github.com/RasaHQ/rasa

官网：https://rasa.com/

**10、Trigger.dev**

Trigger.dev是一个开源的TypeScript/JavaScript后台任务与工作流框架，专为长时间运行的无超时任务设计，支持使用常规异步代码创建和管理复杂工作流，提供完整的监控、重试和调度能力，使开发者无需担心基础设施管理问题。

主要特征：

* 无超时限制：支持长时间运行的后台任务，不受传统无服务器环境时间限制；
* 事件驱动架构：支持cron定时、Webhook和自定义事件触发工作流；
* 弹性扩展：自动适应不同工作负载，无需手动调整资源；
* 全方位监控：实时状态、详细日志、链路追踪和自定义警报；
* 开发者友好：TypeScript/JavaScript原生支持，无缝集成现有代码库，提供CLI和SDK；
* 灵活部署：支持云端托管(Trigger.dev Cloud)和自托管两种方式；

案例：

* AI客服系统：并行处理客户咨询和内容审核，实现24/7服务，响应时间缩短70%；
* 数据管道构建：连接多个数据源，自动清洗、转换和加载，支持大规模数据处理；
* 电商订单处理：自动化订单确认、库存更新、物流通知和支付对账全流程；

优势：

* 开发效率提升：使用熟悉的TypeScript/JavaScript编写任务，无需学习特定DSL；
* 可靠性增强：内置自动重试、错误处理和任务持久化，确保任务最终完成；
* 成本优化：按使用付费模式，仅为实际执行时间付费，避免闲置资源浪费；
* 无缝集成：与CI/CD工具、数据库和云服务原生集成，简化系统架构；
* 可观测性：实时仪表盘和详细监控，快速诊断和解决问题；

弱点：

* 学习曲线：对于不熟悉异步编程的开发者有一定上手门槛；
* 特定环境问题：在某些自托管或特殊环境中可能出现任务卡顿或状态同步问题；
* 生态相对年轻：与一些成熟的任务调度框架相比，社区插件和集成方案较少。

GitHub链接：https://github.com/triggerdotdev/trigger.dev

官网：https://trigger.dev/

**11、Botpress**

Botpress已从传统聊天机器人构建工具转型为全功能Agentic AI平台，提供无代码/低代码界面创建具有推理、工具调用和工作流编排能力的智能体，支持与API、数据库深度集成，是企业快速部署智能体应用的理想选择。

主要特征：

可视化设计：通过拖放界面设计智能体流程，无需编写代码；

多模态支持：文本、语音、图像等多种交互方式；

AI集成：无缝接入LLM、RAG等先进AI能力；

插件生态：丰富的预建模块，快速添加功能（如支付、通知）；

自定义代码：支持在关键节点插入JavaScript/Python代码；

部署灵活性：支持云部署、自托管和容器化；

应用案例：

客户服务自动化：构建24/7智能客服，处理常见问题和请求；

内部流程助手：简化员工操作，如请假申请、IT支持；

营销自动化：根据用户行为提供个性化推荐和服务；

优势：

零代码门槛：业务人员也能设计和维护智能体；

快速迭代：可视化界面使修改和优化智能体流程变得简单；

成本效益高：减少开发资源投入，加速产品上市；

全面的监控：提供详细分析和日志，便于优化智能体性能；

弱点：

定制化限制：复杂逻辑可能需要编写代码，失去无代码优势；

性能考量：可视化设计可能生成效率较低的执行代码；

生态系统相对较新：与其他成熟框架相比，插件和社区资源较少。

GitHub链接：https://github.com/botpress/botpress

官网：botpress.com

**12、SmolAgents**

Hugging Face开源的轻量级智能体框架，设计理念是"少即是多"，提供极简API和实现，专注于"代码智能体"(Code Agent)的原生支持，让开发者能用最少代码快速构建功能强大的智能体系统。支持多种模型和工具，强调简单性、安全性，并提供丰富的集成选项，包括与模型中心的集成、多模态输入支持以及工具的灵活性。

主要特征：

* 极简架构：核心代码简洁，易于理解和定制，避免不必要的抽象；
* 代码智能体优先：特别优化了执行代码的智能体类型；
* 与Hugging Face生态集成：无缝对接模型、数据集和其他工具；
* 轻量级依赖：对系统资源要求低，适合边缘设备和快速原型开发；
* 三行代码创建智能体：极简API设计，大幅降低入门门槛；

应用案例：

* 自动化脚本执行：构建智能脚本执行器，根据自然语言指令完成任务；
* 简单工具链：连接多个API和工具，形成自动化工作流；
* 教育工具：帮助初学者理解和使用AI智能体，降低学习成本；

优势：

* 入门门槛极低：无需复杂配置，几行代码即可创建可用智能体；
* 执行效率高：轻量级实现减少运行开销，提高响应速度；
* 代码友好：对开发者友好的API设计，便于阅读和维护；
* 与Hugging Face生态协同：可直接利用生态中丰富的模型和工具；

弱点：

* 功能相对有限：与全功能智能体框架相比，高级特性支持不足；
* 扩展性受限：复杂应用可能需要修改框架源码或迁移到其他框架；
* 文档和社区资源较少：作为较新项目，支持资源相对有限。

GitHub链接：https://github.com/huggingface/smolagents

**13、Agno**

一个高性能、注重隐私的多智能体框架、运行时和控制平面，旨在快速、安全和可扩展地构建智能体、多智能体团队和工作流。它提供了丰富的工具集，支持记忆、知识、会话管理和高级功能，如人机交互、防护栏、动态上下文管理等。

提供了完整的智能体解决方案，包括快速开发框架、预构建的FastAPI应用和集成的控制平面，确保数据隐私和安全。

主要特征：

* 性能优化：通过算法和架构优化，实现极高的执行效率；
* 多模态处理：原生支持文本、图像、音频等多种数据类型；
* 智能体团队：支持多智能体协作，有团队领导者控制全局状态；
* 异步执行：支持任务并行处理，提高整体系统吞吐量；
* 极简API：提供简洁接口，降低使用门槛和开发工作量；

应用案例：

* 内容审核系统：快速处理大量多媒体内容，识别违规信息；
* 实时协作：多智能体并行处理任务，如大型数据处理或分析；
* 智能客服：结合视觉和语音识别，提供更全面的客户服务；

优势：

* 卓越性能：高吞吐量和低延迟，适合大规模数据处理；
* 资源高效：轻量级设计，适合边缘设备和资源受限环境；
* 灵活的团队协作：支持多种协作模式，适应不同应用场景；
* 多模态支持：一站式处理多种类型数据，简化应用开发；

弱点：

* 文档有限：详细文档和教程资源相对较少；
* 生态系统较新：第三方集成和社区支持不如其他成熟框架；
* 与特定模型绑定：性能优化可能依赖特定模型和硬件组合。

GitHub链接：https://github.com/agno-agi/agno

Agno 文档：https://docs.agno.com

**14、eliza**

ElizaOS是一个开源的多智能体人工智能开发框架，旨在帮助用户构建、部署和管理自主AI Agent。具有丰富的连接性，支持多种主流模型，提供现代化的Web界面，并且高度可扩展。适用于多种应用场景，如聊天机器人、业务流程自动化和智能游戏NPC等。

主要特征：

* 丰富的连接性：支持Discord、Telegram、Farcaster等多种平台的开箱即用连接器。
* 模型无关性：兼容OpenAI、Gemini、Anthropic、Llama和Grok等主流模型。
* 现代Web UI：提供专业的仪表盘，用于实时管理Agent、团队和对话。
* 多智能体架构：从零开始设计，便于创建和协调专业智能体群。
* 文档摄取：支持轻松导入文档，使Agent能够从用户数据中检索信息并回答问题（RAG）。
* 高度可扩展：通过强大的插件系统构建自定义功能。
* 无缝衔接：提供从第一天起就无缝衔接的设置和开发体验。

案例：

* 聊天机器人：可以快速创建和部署与用户进行自然语言交互的聊天机器人。
* 业务流程自动化：利用智能体自动化复杂的业务流程，提高效率。
* 智能游戏NPC：为游戏开发者提供创建智能NPC的工具，提升游戏体验。

优势：

* 一体化平台：提供从开发到部署的完整工具链，降低开发门槛。
* 高度灵活性：支持多种模型和平台，适应不同用户需求。
* 快速入门：提供详细的入门指南和CLI工具，帮助用户快速上手。
* 社区支持：开源项目，拥有活跃的社区和丰富的文档资源。

弱点：

* 技术门槛：对于初学者来说，可能需要一定时间来熟悉CLI工具和开发流程。
* 依赖外部模型：依赖于外部模型提供商的服务，可能受到其性能和可用性的影响。
* 文档完善度：虽然有详细的文档，但可能需要进一步优化以满足不同用户的需求。

GitHub链接：https://github.com/elizaOS/eliza

官网：https://elizaos.ai/

**15、Motia**

一个多语言后端框架，将API、后台作业、队列、工作流、流式传输、AIAgent等功能集成于一个核心原语“步骤”中。

Motia具备内置的可观测性和状态管理，支持JavaScript、TypeScript、Python等语言，旨在消除后端开发中的运行时碎片化，简化开发流程，提供快速启动、AI辅助开发等特性，并有详细的开发指南和丰富的示例。

主要特征：

* 全栈集成：API、任务、事件和智能体统一管理；
* 多语言混合执行：同一流程中支持TS/JS和Python代码；
* 事件驱动架构：所有操作(HTTP请求、定时任务、Webhook)通过事件触发；
* 内置AI智能体：无需额外集成即可使用智能体功能；
* 模块化设计：可灵活添加或替换组件，适应不同需求；

应用案例：

* 微服务架构：构建分布式后端系统，服务间通过事件通信；
* 实时应用：开发消息推送、实时协作等需要高响应性的应用；
* 智能工作流：结合AI智能体自动处理业务流程，如订单处理、用户管理；

优势：

* 开发效率提升：一站式解决方案减少集成工作量；
* 架构简洁：统一运行时简化系统设计和维护；
* 灵活的语言选择：根据任务特性选择最合适的编程语言；
* 内置AI能力：开箱即用的智能体功能，加速创新应用开发；

弱点：

* 学习曲线陡峭：掌握整个框架需要学习多种技术；
* 生态系统较新：第三方库和社区资源相对有限；
* 与现有系统集成：与传统架构集成可能需要较大调整。

GitHub链接：https://github.com/MotiaDev/motia

官网：https://motia.dev/

**16、Mastra**

一个用于构建AI应用程序和代理的现代TypeScript框架。集成了前端和后端框架，支持多种LLM，包括OpenAI、Anthropic和Gemini等。

Mastra提供模型路由、自主Agent、工作流、人机交互、上下文管理、集成、MCP服务器和生产必备功能，帮助开发者从原型到生产级应用。它还提供文档、社区支持和安全维护。

主要特征：

TypeScript原生：提供类型安全，增强开发体验和代码质量；

前端友好：与React、Vue等主流前端框架无缝集成；

智能体生命周期管理：简化智能体创建、更新和销毁过程；

浏览器优化：针对前端环境优化，减少内存占用和执行开销；

组件化设计：智能体相关功能可作为React组件使用；

应用案例：

智能表单：构建能理解用户意图的智能表单，提供实时验证和建议；

交互式内容：创建能响应用户操作的动态内容，提升用户参与度；

浏览器插件：开发具有智能功能的浏览器扩展，如内容分析、信息提取；

优势：

前端开发者友好：利用现有技能，降低AI开发门槛；

无缝UI集成：智能体逻辑与前端组件紧密结合，简化开发流程；

性能优化：针对浏览器环境优化，确保流畅用户体验；

生态系统优势：：充分利用JavaScript丰富的库和工具链；

弱点：

后端能力有限：主要专注前端，复杂后端逻辑需额外集成；

资源限制：浏览器环境下计算能力和内存受限；

与专用后端智能体框架相比：在复杂推理和大规模数据处理方面较弱。

GitHub链接：https://github.com/mastra-ai/mastra

**17、Pydantic AI**

由Pydantic团队开发的Python Agent框架，旨在帮助开发者快速、自信且轻松地构建生产级别的生成式人工智能应用程序和工作流。

该项目借鉴了FastAPI的设计理念，具有模型无关性、无缝可观察性、完全类型安全、强大的评估功能等优势，还支持MCP、A2A和UI标准，提供人类在循环工具审批、持久执行、流式输出和图支持等功能。

主要特征：

* 数据验证：利用Pydantic自动验证智能体输入输出数据；
* 类型安全：提供强类型保证，减少运行时错误；
* 结构化交互：强制智能体使用预定义的数据模型进行交互；
* 与FastAPI集成：无缝结合FastAPI构建RESTful智能体服务；
* 自定义插件：支持添加自定义验证逻辑和预处理函数；

应用案例：

* API数据处理：构建智能API网关，自动验证和处理请求数据；
* 数据清洗：智能数据处理管道，确保输入数据符合预期格式；
* 表单验证：创建能理解用户意图的智能表单，提供实时验证和纠错；

优势：

* 数据质量保障：通过严格验证减少错误和异常情况；
* 开发效率提升：自动生成文档和验证代码，减少样板工作；
* 与Python生态集成：无缝对接现有的Python工具和库；
* 错误处理简化：清晰的错误信息和数据转换，便于调试和维护；

弱点：

* 灵活性受限：严格的数据模型可能限制智能体的创造力和适应性；
* 学习曲线：掌握Pydantic和框架特性需要一定时间；
* 与专用智能体框架相比：在自主决策和复杂推理方面支持较弱。

GitHub链接：https://github.com/pydantic/pydantic-ai

官网：https://ai.pydantic.dev/

**18、AutoAgent**

香港大学开源的一个全自动化、零代码的大型语言模型（LLM）Agent框架，允许用户仅通过自然语言创建和部署Agent。

其主要功能包括自然语言驱动的Agent构建、零代码框架、自我管理工作流生成、智能资源编排和自玩Agent定制。用户可通过用户模式、Agent编辑器和工作流编辑器三种方式使用。项目开源，支持多种LLM，并提供详细的安装和使用指南。

主要特征：

* 自然语言编程：通过自然语言描述创建和管理智能体；
* 自动工作流生成：根据需求自动设计和执行工作流程；
* 多智能体协作：支持多个专业智能体协同完成复杂任务；
* 预定义智能体类型：内置网络、代码、文件等专用智能体；
* 低代码/无代码：无需编程知识，通过简单配置即可创建智能体；

应用案例：

* 数据处理：用户描述"分析销售数据并生成报告"，系统自动完成数据获取、处理和可视化；
* 内容创作：根据主题描述自动生成文章、演示文稿等内容；
* 任务自动化：将日常重复任务(如文件整理、报告生成)转换为智能工作流；

优势：

* 革命性易用性：无需编程知识，普通用户也能创建智能应用；
* 开发效率：从需求到实现时间大幅缩短，提高生产力；
* 自然交互：符合人类思维习惯的交互方式，降低认知负担；
* 多智能体协同：自动协调多个智能体完成复杂任务，提高成功率；

弱点：

* 理解局限性：对复杂或模糊需求的理解可能不准确；
* 定制化限制：难以实现高度个性化的功能和交互；
* 依赖高质量描述：输出质量高度依赖用户提供的需求清晰度。

GitHub链接：https://github.com/HKUDS/AutoAgent

论文：https://arxiv.org/abs/2502.05957

**19、OpenHands**

OpenHands V1是一款完全重构的开源多智能体框架，专为软件开发和自动化任务设计。

它通过模块化架构和严格的设计原则，解决了V0版本单体架构的沙盒僵化、耦合过高等痛点，以“主Agent+子智能体”分层协作模式，提供从原型到生产部署的完整多智能体解决方案，成为Manus的开源平替标杆。

主要特征：

* 核心设计：可选隔离、默认无状态、关注点分离、双层可组合性，适配多智能体协同；
* 架构亮点：“主Agent+子智能体”分层架构，支持任务自动拆解分配；集成100+LLM、多工具链，内置安全沙盒与浏览器交互能力。

应用案例：

* 软件开发：电商支付模块开发中，自动分配前端、API设计、安全审计等专项智能体，开发周期从7天缩至2天；
* 数据处理：金融数据任务中，子智能体分工完成数据获取、清洗、可视化，高效生成趋势报告。

优势：

* 协作效率跃升：多智能体并行处理，任务完成时间缩短66%，LLM调用成本降低62.5%；
* 部署灵活：Docker一键部署，支持本地/容器化，无缝集成CI/CD；
* 安全可控：可选沙盒隔离高危操作，适配企业级场景。

弱点：

* 高级特性学习曲线陡峭，需理解多智能体任务调度逻辑；
* 大规模多智能体协作对算力要求较高，本地部署需资源优化；
* 生态相对年轻，多智能体专用插件少于成熟框架。

GitHub链接：

* SDK核心：https://github.com/All-Hands-AI/OpenHands
* 官网：https://openhands.dev/

**20、Akka**

Java/Scala平台的高性能分布式计算框架，采用Actor模型简化并发编程，虽然不是专门为AI智能体设计，但因其强大的分布式处理、容错和消息传递能力，被广泛用于构建大规模多智能体系统，特别是需要高可靠性和可扩展性的企业级应用。

主要特征：

* Actor模型：每个Actor是独立的计算单元，通过异步消息通信；
* 分布式计算：支持跨节点部署，轻松构建大规模分布式系统；
* 高容错性：自动处理节点故障，确保系统持续运行；
* 弹性扩展：能够根据负载自动调整资源使用；
* 消息驱动架构：所有交互通过消息传递，避免共享状态问题；

应用案例：

* 大规模实时处理：构建股票交易、物联网设备管理等需要实时处理的系统；
* 分布式智能体系统：多智能体协同工作，如物流调度、智能电网管理；
* 高性能计算：科学计算、数据分析等需要大量并行处理的场景；

优势：

* 卓越的性能：高吞吐量和低延迟，适合大规模数据处理；
* 强大的容错能力：自动处理故障，确保系统可靠性；
* 语言选择：支持Java和Scala两种强大的静态类型语言；
* 成熟度：经过多年发展和大规模生产环境验证；

弱点：

* 学习曲线陡峭：掌握Actor模型和框架概念需要时间；
* 与AI集成：需要额外工作与LLM和其他AI组件集成；
* 开发复杂性：构建分布式系统需要考虑更多因素和边界条件；

GitHub链接：https://github.com/akka/akka-core

官网：https://akka.io/

**后记：框架选择指南**

选择Agentic AI框架的核心，是对齐使用场景、技术栈与需求复杂度。以下按核心场景分类兼顾新手友好度与企业级诉求的简单指南，可以帮你快速选择适配方案。

多智能体协作与编排

* 推荐框架：AutoGen、CrewAI、LangGraph、MetaGPT、OpenHands；
* 适配场景：团队式任务分工（如软件开发、数据分析）、复杂流程拆解；
* 选型建议：新手优先CrewAI（角色定义直观），企业级选AutoGen/OpenHands（可扩展性强），软件开发专项选MetaGPT，复杂流程用LangGraph（图结构工作流）；

开发模式适配

* 低代码/无代码：Botpress、AutoAgent（适合业务人员快速落地）；
* 轻量级快速原型：SmolAgents、Agno（几行代码启动，资源消耗低）；
* 企业级部署：Semantic Kernel（微软生态）、Akka（分布式高可靠）、RASA（对话系统）；

功能侧重匹配

* RAG与私有数据交互：LlamaIndex（专用RAG）、LangChain（灵活集成）；
* 对话式智能体：RASA（复杂多轮对话）、elizaOS（多平台适配）；
* 前端AI应用：Mastra（TypeScript+React/Vue无缝集成）；
* 分布式/去中心化：Atomic Agents、Akka（大规模高容错场景）；
* 任务流自动化：Trigger.dev（无超时后台任务）、LangChain（工具链丰富）；

技术栈适配

* Python生态：CrewAI、AutoGen、LlamaIndex、MetaGPT（生态成熟，资源丰富）；
* JavaScript/TypeScript：Mastra、Trigger.dev、Botpress（前端/全栈优先）；
* 多语言支持：Semantic Kernel（C#/Python/JS）、Motia（TS/JS/Python）
* Java/Scala：Akka（分布式企业级应用）。

最后记住，选择时优先明确核心诉求。新手看入门门槛，企业级看可靠性与集成性，专项场景（如RAG、对话）选垂直优化框架。

全文完

看到这里，如果觉得不错，随手点个赞、在看或者转发吧，也可以给个星标，你的支出就是我的动力。

王吉伟频道新书《一本书读懂AI Agent：技术、应用与商业》已出版，轻松读懂系统掌握AI Agent技术原理、行业应用、商业价值及创业机会，欢迎大家关注。

**【赠书福利】**

**感谢大家长期关注与支持，**小伙伴们随意留言，王吉伟频道会随机选取留言用户，《一本书读懂AI Agent：技术、应用与商业》包邮到家。****

【文末福利1】：后台发消息 *****研报2025*****，获取十份2025年AI Agent研报。

【文末福利2】：后台发消息 ****Workflow****，获取Agentic Workflow相关25篇论文。

【文末福利3】：后台发消息 **agentic**，获取**Agentic AI**相关资源。

【文末福利4】：后台发消息 **RPA Agent**，获取相关论文和研报。

**RECOMMEND**

推荐阅读

**1、[一本书读懂AI Agent技术、应用与商业，我写的新书出版了](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594649&idx=1&sn=7e6841519b4941d8ce622dc38e9bf619&chksm=88b9d48fbfce5d9985d65096667fd6f1c8cdc3ceef1e52f72eef014263c2c0d8549d8e4e7aa5&scene=21#wechat_redirect)**

2、[【万字长文】数字员工、具身智能，AI Agent未来发展十大研究方向](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593643&idx=1&sn=7791eb4566934dba49a92bc8491d9d2b&scene=21#wechat_redirect)

3、[【万字长文】全球AI Agent大盘点，大模型创业要参考的60个AI智能体](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593007&idx=1&sn=22b24b222da839231fbaf16779f658af&scene=21#wechat_redirect)

4、[AI Agent发展简史，从哲学思想启蒙到人工智能实体落地](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593064&idx=1&sn=0f307fb00c95f7160c95ff1a30c54925&scene=21#wechat_redirect)

5、[企业AI Agent战略级规模化落地方法论，Agentic AI Stack for Enterprises](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595877&idx=1&sn=d08b69023bf988b9c7b41251f7c4f6ea&scene=21#wechat_redirect)

6、[五层结构AI Enablement Stack，把真正可用的AI Agent技术栈生态讲透了](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595852&idx=1&sn=d064ef01f17f017810ac68b91d98a2ed&scene=21#wechat_redirect)

7、[智能体主题分享：DeepSeek、Manus与AI Agent行业现状，附51页PPT下载](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595760&idx=1&sn=8d4a682f4ba24ebe2a3efa5014f30ee5&scene=21#wechat_redirect)

8、[十篇AI Agent研报，看懂2025年全球智能体行业全景，附下载](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595400&idx=1&sn=13553e4d39b6ec1bc3f7a08a9ca09181&scene=21#wechat_redirect)

8、[API难以解决AI智能体执行能力问题，AI Agent深度落地锁定RPA](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593544&idx=1&sn=285ec85cc082c34b46131819068f7540&scene=21#wechat_redirect)

10、[RPA终极发展方向瞄准AI Agent，超自动化智能体时代已经开启](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593026&idx=1&sn=a470a32bff49b7de9f4127922dea9559&scene=21#wechat_redirect)

**AIGC研究系列文章**

[2025年智能体应用指南，10个不可错过的海外AI Agent构建平台](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595468&idx=1&sn=d81662e48f60acc38e9b3e29819e2730&scene=21#wechat_redirect)

[聊聊DeepSeek大模型对AI Agent的影响，附相关智能体项目与学习资料包](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595440&idx=1&sn=bcb8de223c395aad96309d762992a364&scene=21#wechat_redirect)

[十篇AI Agent研报，看懂2025年全球智能体行业全景，附下载](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595400&idx=1&sn=13553e4d39b6ec1bc3f7a08a9ca09181&scene=21#wechat_redirect)

[智能体变革软件应用，AI Agent带来的软件行业发展新机会](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595347&idx=1&sn=0ec18e30b91a3a5e108eb1dbb13f849b&scene=21#wechat_redirect)

[智能体加速落地，三份重磅研报多角度了解全球AI Agent行业现状，附下载](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595312&idx=1&sn=d91a2a028c79b201a6ad790ad548144c&scene=21#wechat_redirect)

[智能体主题分享：AI Agent现状、技术进展与发展趋势，附36页PPT下载](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595250&idx=1&sn=b2b50b611c585b318c6de83d36f8fe5b&scene=21#wechat_redirect)

[谷歌发布40页AI Agent白皮书，简单易懂的智能体认知架构，附下载](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595200&idx=1&sn=0aacf99678e8170fbebe60c86ac58765&scene=21#wechat_redirect)

[智能体商用元年开启，2025年AI Agent行业发展十三大趋势](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595174&idx=1&sn=97fe04b7c39ce5b815747763a1db63eb&scene=21#wechat_redirect)

[大模型厂商视角的AI Agent综述，Anthropic教你构建有效智能体](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595138&idx=1&sn=7bce740b7ba6568aa2a3c23a7bba3fa2&scene=21#wechat_redirect)

[【万字长文，建议收藏】AI Agent企业应用场景全解：30个智能体落地案例剖析](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595098&idx=1&sn=2f793c249775a7aa54c9f1cc4f2dd95c&scene=21#wechat_redirect)

[6个认知框架，看懂智能体、智能自动化与自主工作的级别分类](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650595028&idx=1&sn=f8759fd74bbcb512827ac1fa353e4c10&scene=21#wechat_redirect)

[顶级孵化器Y Combinator解读智能体，关于垂直AI Agent未来发展的八个问题](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594968&idx=1&sn=e55a21ceae6cede62c4ae7e2d89f80ea&scene=21#wechat_redirect)

[盘点全球50个AI爬虫项目与产品，聊聊向AI Agent进化的爬虫应用现状](https://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594801&idx=1&sn=1f4cb25d4bc3244e51328dc724ba8f65&scene=21#wechat_redirect)

[重新审视AI Agent，聊聊近期的智能体市场动向、应用实践与发展趋势，附PPT](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594737&idx=1&sn=655b79aac2a98999ec7789a4d00ecbbd&chksm=88b9d467bfce5d717e18629f31e1c855f41220dcb3ca503a6b3d89489d58cdbeebc007ba281e&scene=21#wechat_redirect)

[在Claude 3.5 Sonnet之前，这些AI Agent已能像人类一样操作电脑](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594626&idx=1&sn=4a7a04d7fc836497781288f9d308b354&chksm=88b9d494bfce5d82b008db216a40f7fa5ae2f4566e06af7ce90f807bf4faf9de6ed119c61d07&scene=21#wechat_redirect)

[烧钱、耗费资源、难以盈利，被持续唱衰的大语言模型在艰难中倔强前行](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594563&idx=1&sn=ad6f1f9de9e855c3859cffe289607912&chksm=88b9d4d5bfce5dc3db60f866d5a5cc07c426d6f44bba5441176a69c99f8724566241fdc6a6ea&scene=21#wechat_redirect)

[从思维链到强化学习再到智能体，系统解读o1模型对AI Agent的影响](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594492&idx=1&sn=50fbc625ed6ca9ea95bca949db3f8fe5&chksm=88b9d56abfce5c7c59b0db4c2036b0156a62a81cf18362e544d3dda0318d436d4667fb096080&scene=21#wechat_redirect)

[从国内外10个智能体案例，看AI Agent在教育领域的应用](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594439&idx=1&sn=982513e7006b5337b5fbd17e120d5677&chksm=88b9d551bfce5c4788920295f0b0ad25908e73d231287974a7afeb82a925d6b6e92480582a16&scene=21#wechat_redirect)

[【万字长文】RPA与AI Agent融合产品大盘点：产品形态是怎样的？未来发展如何？](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594335&idx=1&sn=92df6ac54c24637becae77246ea9cc0d&chksm=88b9d5c9bfce5cdfd7a69370f7385da558adb92d9e7d943957da15338a0236d3bae32964a38b&scene=21#wechat_redirect)

[智能体工作流开源项目大盘点，20个项目轻松构建Agentic Workflow](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594236&idx=1&sn=997739fc2004cac5369e00ccc75b0b4c&chksm=88b9d26abfce5b7c3537ba756c44bffd23f0294a6f36a2039a39f57ff0205935e8672aaa6ff1&scene=21#wechat_redirect)

[Agentic Workflow新范式，基于大语言模型的工作流、业务流程、智能体大融合【附十篇相关论文】](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594183&idx=1&sn=62442d045a29569f496c32aff4c5b73a&chksm=88b9d251bfce5b47346e3dda1fb79202dd4a4accecbb1fba13ce0861f7fb5e0c0addf6d745c7&scene=21#wechat_redirect)

[从Workflow到Agentic Workflow，25篇论文全面了解智能体工作流](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594119&idx=1&sn=1b33d8ae1220fa30486c4baf9ecfda92&chksm=88b9d291bfce5b87fa9b3d5b40d94d1b28379ae3dffa33b1823bbd6608fa4b635ba1f5db33fc&scene=21#wechat_redirect)

[Agentic Workflow加速Agentic AI到来，AI Agent成为重要实现方式](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650594022&idx=1&sn=ead770994c1fdf21f5edb85f1fa4263f&chksm=88b9d330bfce5a26cd9cb6fbbb6b6a13978775015cec3cf169264fe3601d7420e2c036ea90bb&scene=21#wechat_redirect)

[【万字长文】AI智能体驱动未来商业，深度剖析11种AI Agent商业模式](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593940&idx=1&sn=9c2d2e728139f5df562b903833ff7240&chksm=88b9d342bfce5a54898afc7149124cd1fee6193708726c5281521af1c0a89cd372ae0be7978e&scene=21#wechat_redirect)

[科技巨头紧锣密鼓布局智能体，你需要了解AI Agent行业未来发展的18个趋势](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593879&idx=1&sn=1cbb73996a9e1fbd05f9427902dd595c&chksm=88b9d381bfce5a97127d45190611ec2488af28a27a17d28a25353cb7e81a50829b69abbfb817&scene=21#wechat_redirect)

[AI智能体构建智能未来，全球80+AI Agent构建平台大盘点](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593838&idx=1&sn=ce5d980dae19a28a10c421ffb99cff10&chksm=88b9d3f8bfce5aee1589c22c5852bb8f8927ff4ac12f180566318e3a1bb41d49fb6640a74173&scene=21#wechat_redirect)

[AI智能体全景式解读，SWOT视角下的AI Agent行业将何去何从？](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593796&idx=1&sn=5e144fbb7d8fa78a74fd2af2879c26c1&chksm=88b9d3d2bfce5ac4cd6eb6da21fb0a22abd7d1c886c391f4880d097a9d757f2f1404dfcf6a0c&scene=21#wechat_redirect)

[【深度盘点】从科技巨头到创业公司，先一步布局的AI Agent加速应用落地](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593768&idx=1&sn=91ddd4067516227c69a8f96dd24a9d8c&chksm=88b9d03ebfce5928c0c6937b37d30314e38b369f003250f9c61c4123aa5140d38bd6e74d04a9&scene=21#wechat_redirect)

[AI Agent涌向移动终端，手机智能体开启跨端跨应用业务连接新场景](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593709&idx=1&sn=fbba53f913c0d8aedc26349584c8dacc&chksm=88b9d07bbfce596dbf7484aaf3316fe49dbff97702ddbbdaade619145dbaf6a29344b6568e95&scene=21#wechat_redirect)

[AI Agent引爆AGI时代，十篇研报透视AI智能体的现在与未来](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593676&idx=1&sn=30cf55872ccf52cd20857432868b30c2&chksm=88b9d05abfce594cf8e2e8cc29ee15bf5cce67be87967a3b17f88cd4494df5b041623ba2990d&scene=21#wechat_redirect)

[【万字长文】数字员工、超级个体、具身智能，AI Agent未来发展十大研究方向](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593643&idx=1&sn=7791eb4566934dba49a92bc8491d9d2b&chksm=88b9d0bdbfce59abf991c9c1e90a82e1416be56cba17d6e202185696c3cfd8aa4db6aa658368&scene=21#wechat_redirect)

[详解AI Agent市场格局、技术路径与未来市场，智能体创业一定不要错过](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593364&idx=1&sn=f6714b5425f720ef39d3e51ed8e5ce26&chksm=88b9d182bfce589440568da59c58119c3f25915a5b959d8ec6904de98203760a5d7b6dc4fdc8&scene=21#wechat_redirect)

[API难以解决AI智能体执行能力问题，AI Agent深度落地锁定RPA](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593544&idx=1&sn=285ec85cc082c34b46131819068f7540&chksm=88b9d0debfce59c8fe276700ccdc3b17921d07b1b53224f7d5a2a8e84726926fd7df23562e5b&scene=21#wechat_redirect)

[热闹的人工智能VS酷寒的资本寒冬，2023年AI Agent项目盘点与融资分析](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593540&idx=1&sn=41d48ed5e5e57927718c2b0ceb54f539&chksm=88b9d0d2bfce59c4d01707ac05f8d4f17aa63c5920e1362dfd70028f989ef3aa1bb450a4e928&scene=21#wechat_redirect)

[正在强烈冲击AI Agent的“准Agent” GPTs，真的会杀死AI智能体吗？](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593284&idx=1&sn=de077ee6efa51aa5ce6d94ed19ecbe1d&chksm=88b9d1d2bfce58c473bd2fc7819b5f8681387c13148f4e8d86ef4ea182891a7897619ee87900&scene=21#wechat_redirect)

[AI Agent发展简史，从哲学思想启蒙到人工智能实体落地](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593064&idx=1&sn=0f307fb00c95f7160c95ff1a30c54925&chksm=88b9defebfce57e8a454559bc71c26205f77236636c908a3f8663c934b4fdcb21cda0456982f&scene=21#wechat_redirect)

[【万字长文】全球AI Agent大盘点，大语言模型创业一定要参考的60个AI智能体](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593007&idx=1&sn=22b24b222da839231fbaf16779f658af&chksm=88b9df39bfce562fa66990f5d90e41473c9d19a3474efd2393cd3e3b8df2cdbbc557c9589665&scene=21#wechat_redirect)

[RPA终极发展方向瞄准AI Agent，超自动化智能体时代已经开启](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593026&idx=1&sn=a470a32bff49b7de9f4127922dea9559&chksm=88b9ded4bfce57c2c926013db9b028ecc824ea2f6cd5b2d6568dc20e6fd37748d13f69bed3be&scene=21#wechat_redirect)

[从大语言模型到大流程模型，生成式AI带来的BPM范式转变](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650593149&idx=1&sn=d4f3db8eaf05b66177d9487336efeed8&chksm=88b9deabbfce57bd4218d528b4939ae17ee66ee8fb4496f2a424c16f68ae596e2f7ca06a7d50&scene=21#wechat_redirect)

[产业上下游齐发力LLM挺进端侧，大语言模型加速落地利好超自动化](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650592770&idx=1&sn=46c634dfc5e5c619deb4471bdaab05b5&chksm=88b9dfd4bfce56c2fee621e295a7bdb00ac0c38a192fe16889939b95b47b4db786fee2e7eda6&scene=21#wechat_redirect)

[从引入并集成多LLM到发布自研模型，RPA与LLM的融合进度怎样了？](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650592604&idx=1&sn=47ce3c036b9bdd9e1e73201409c9df0c&chksm=88b9dc8abfce559c604af1a359a80ec75618dc7f2d5315d54505380d08756a54ad5f05cf4d13&scene=21#wechat_redirect)

C[hatGPT与RPA集成，生成式AI+自动化流程让AIGC价值倍增](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650591119&idx=1&sn=b4c39d8be943c86c6a8e5396126cb785&chksm=88b9c659bfce4f4fa0d9a48ddb3203d47ad9c0f24f72bc80c4486c2bc7023c7a0a7dab27f1b3&scene=21#wechat_redirect)

[产业上下游齐发力LLM挺进端侧，大语言模型加速落地利好超自动化](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650592770&idx=1&sn=46c634dfc5e5c619deb4471bdaab05b5&chksm=88b9dfd4bfce56c2fee621e295a7bdb00ac0c38a192fe16889939b95b47b4db786fee2e7eda6&scene=21#wechat_redirect)

[业务流程将因生成式AI变革，ChatGPT引领的AIGC正在改变组织运营](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650591230&idx=1&sn=826fd3a2d154cd32af4d986d6c39f434&chksm=88b9c628bfce4f3ea6400b0918dc1d4bae0bf4c9398d07bd3625f5a1af7ae72ddaac734eafaf&scene=21#wechat_redirect)

[更多组织接入ChatGPT等生成式AI，生成式自动化或成企业运营新标配](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650591394&idx=1&sn=bf9c8fd25f898f09e3f023c2744fb3a1&chksm=88b9d974bfce50626cd0bbb072496b514535255b3d3203f62aa080d189b836a65495c5b185e8&scene=21#wechat_redirect)

[AIGC模式正在影响更多组织，十个案例助你深度认知生成式AI](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650591473&idx=1&sn=d29d30364cb696959bab94fc9a57f22c&chksm=88b9d927bfce5031fee7b6777c9c886e706ea68275d8be3affbadb6dfdce52831dd4a19746c1&scene=21#wechat_redirect)

[多家厂商引入ChatGPT，集成与融合生成式AI成为RPA技术新趋势](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650591547&idx=1&sn=9365821698a0369af6348edffec7214d&chksm=88b9d8edbfce51fbbf709008652c7bc5f74dc086ceb071822998b182c90557a36965427ed71b&scene=21#wechat_redirect)

[基于AI构建的当代RPA，在生成式AI影响下的生命周期还有多长？](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650591714&idx=1&sn=988175e44a2eb4610572f685aa6d4e57&chksm=88b9d834bfce51224097759dee48da9d8a65b6c9b3203e65bc694d6b185a3ceb0be221d34b71&scene=21#wechat_redirect)

[从ChatGPT数据泄露事件，看组织安全稳定自动化的重要性](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650591739&idx=1&sn=d835a888afe52ec78028b3cd1fc6603b&chksm=88b9d82dbfce513b0836b63270fb8876e486a16b3bdbd3d3fc5bed652f2879eda518fd47d9f2&scene=21#wechat_redirect)

[大模型API上的新商业逻辑，生成式AI变革组织经营](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650591634&idx=1&sn=0f2cf4dd1523616b804bf178fabef9c0&chksm=88b9d844bfce515232f7b04b78a75c5ac9d73b1074acbff6c9ff25be518d7faec407e88893f9&scene=21#wechat_redirect)

[生成式AI与客户体验有什么关系？如何影响客户体验？一文看明白](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650591818&idx=1&sn=7b8e1803a79a405612e81525c80b2a11&chksm=88b9db9cbfce528a077f4e09a13ff0900a55d8a04ee5e8c930b85bc6001183192ed44ff46888&scene=21#wechat_redirect)

[从几个业务场景和实际案例，看生成式AI在金融领域的应用](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650591925&idx=1&sn=343a0f16ff6a93a5132ce4eab2d03c26&chksm=88b9db63bfce5275d05d4aa38a806cc57dc82a7822b6dfca044764a8005d0b4e3a996933c42c&scene=21#wechat_redirect)

[生成式AI席卷PPT制作，办公生产力迎来大变革，附20个正在流行的AI PPT制作工具](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650591948&idx=1&sn=5879d9dceb2229caa53a7adadb137a23&chksm=88b9db1abfce520cf1846fd058f0bb3f22246162bbbba11bf3fafe7eb4b80e1813cb3bb1d74e&scene=21#wechat_redirect)

[LLM时代到来，生成式AI会成为超自动化蓬勃发展的催化剂吗？](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650592016&idx=1&sn=47b90a4e98d8c61c79e3840c4fd34bda&chksm=88b9dac6bfce53d0e5c84ef4e50f44af0d6b29d82f556d00ecd145bc8792343c2d70cb7262a6&scene=21#wechat_redirect)

[AIGC持续火爆大模型争相推出，庞大市场造就算力供应模式演变](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650592048&idx=1&sn=0880d6df47fc088c21b638a7396425dc&chksm=88b9dae6bfce53f0e1d3ee7a56e9ffefe1d18885ab4986bc196259801de07abc3bd9c751524c&scene=21#wechat_redirect)

[从“人+RPA”到“人+生成式AI+RPA”，LLM如何影响RPA人机交互？](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650592122&idx=1&sn=52216fd15426e7381534c78788dab1fd&chksm=88b9daacbfce53babf528606b62839e374909c2a423c7828de1e2600d991e116850d8579c197&scene=21#wechat_redirect)

[生成式AI正在颠覆装饰装修领域【文末附28个AI装饰设计工具】](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650592243&idx=1&sn=d2ca41385bab610c946ee1a3a6197a19&chksm=88b9da25bfce533327e6098f232a05a21cb18bc3f37c6c034d0491aecc1e540a91326595f17b&scene=21#wechat_redirect)

[从LLM特性与数字化转型本质，看大语言模型对数字化转型的影响](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650592359&idx=1&sn=fed5149d38d7f387616483ec4c3479de&chksm=88b9ddb1bfce54a781c8c6a121f424d534293738e5b7cf6d1b9f44660a9cd71be9a9e1005525&scene=21#wechat_redirect)

[2022-2023上半年全球RPA融资盘点：海外项目占比67%，总额165亿元](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650592564&idx=1&sn=03bc8d4db8296e1bdec7963002c33f69&chksm=88b9dce2bfce55f478070684e0e702764d9f136f99445922d4680c286be3427115782ccec07f&scene=21#wechat_redirect)

[从AI模特换装到AIGC赋能运营，生成式AI全方位渗透电商产业链](http://mp.weixin.qq.com/s?__biz=MzA5NjMzODEwNQ==&mid=2650592802&idx=1&sn=cf38b79c8fa186c7ba1fb20f5fe66698&chksm=88b9dff4bfce56e2e3186f79eced4f8fc49d0a60df55deefb68d27fd104a4f75bb304e2584f0&scene=21#wechat_redirect)

* 期待点赞、在看、评论、转发，您的支持就是我的动力。
* 鼓励积极评论，您的留言可以成为选题。
* 欢迎阅读其他文章，或会激发您的更多思考。

点击左下角“阅读原文”查看AIGC研究系列文章，扫码或者后台回复【加群】申请加入AIGC行业应用交流社群。如果你是正在关注AI Agent的创业者、投资人及企业，欢迎带着产品、项目及需求与王吉伟频道交流。

**注：**RPA相关文章，后台回复关键词 RPA 。

【王吉伟频道，关注AIGC与IoT，专注数字化转型、业务流程自动化与AI Agent。公号ID：jiwei1122，欢迎关注与交流。】