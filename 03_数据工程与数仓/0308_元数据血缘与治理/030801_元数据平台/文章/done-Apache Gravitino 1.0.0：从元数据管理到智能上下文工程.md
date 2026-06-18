> 已吸收至：[[03_数据工程与数仓/0308_元数据血缘与治理/030801_元数据平台/030801_核心知识点/Gravitino AI元数据与上下文工程边界|Gravitino AI元数据与上下文工程边界]]
---
title: Apache Gravitino 1.0.0：从元数据管理到智能上下文工程
author: Apache Gravitino
date:
url: https://mp.weixin.qq.com/s?__biz=MzAxODY2MTM3MA==&mid=2247484263&idx=1&sn=2030d11a1847d796bfdf23e9c064a7ed&chksm=9ac31c2ab401e670b6578750cdf6b98b97a9fdcac7bf8a6c3e87120a09b13f71cdd283bb7fe7&mpshare=1&scene=24&srcid=1011nDHlKCwb7WmpZJfidlfs&sharer_shareinfo=f9b62d686dbf8f41404975368afce4b9&sharer_shareinfo_first=f9b62d686dbf8f41404975368afce4b9#rd
---

**2025年9月30日 — Apache 软件基金会（ASF）顶级项目 Apache Gravitino™ 今日正式发布其 1.0.0 版本。这一里程碑式的发布以全新的“元数据驱动的行动系统”为核心，标志着 Gravitino 从一个强大的统一元数据目录，正式演进为一个智能、可行动的统一数据治理平台。新版本旨在帮助企业构建其中枢“数据大脑”，将数据管理提升至前所未有的自动化与智能化水平。

  Apache Gravitino 旨在为企业跨异构数据源、跨云环境、跨区域的复杂数据生态提供统一的元数据管理解决方案，即“元数据湖（Metalake）”。它致力于解决现代数据平台中元数据碎片化和治理困难的核心挑战。

  “Gravitino 1.0 的发布不仅是版本号的更迭，它更代表了一种数据治理范式的转变，” Apache Gravitino 项目管理委员会（PMC）主席 Jerry Shao 表示，“我们不再仅仅满足于让用户‘看到’他们的全部数据资产，而是要赋予他们‘智能且自动地处理’这些资产的能力。全新的‘元数据驱动的行动系统’正是这一愿景的核心，它将数据从被动管理转变为主动治理。”

Apache Gravitino 1.0.0 版本的核心亮点包括：

* 元数据驱动的行动系统：

  首次引入统计、策略和作业三大核心组件。用户现在可以基于精准的元数据洞察，定义自动化规则（例如 Iceberg 表文件合并、数据生命周期（TTL）管理、个人敏感信息（PII）识别等），并自动触发相应作业来执行。这将繁琐的数据维护工作转变为自动化流程，极大提升了数据治理的效率与准确性。

* 面向 AI 的自然语言交互能力：

  通过全新的 MCP（Model Context Protocol）server，Gravitino 成功打通了与大语言模型（LLM）的桥梁。用户可以使用自然语言，通过 Cursor、Claude Desktop 等对话式 AI 工具直接与 Gravitino 交互，以“聊天”的方式实现复杂的元数据管理和治理任务，极大地降低了使用门槛。

* 全面的企业级统一访问控制：

  新版本完善并正式启用了基于 RBAC（基于角色的访问控制）的统一访问控制框架。企业可以在 Gravitino 中为所有数据资产定义统一、细粒度的安全策略，并在访问时强制执行，确保了从元数据到数据实体的端到端安全合规。

* 更广泛的生态系统支持与核心增强：

  ·新增数据源支持：正式支持 StarRocks 数据目录的管理；

  ·核心版本升级：全面兼容最新版的 Apache Iceberg (1.9.0) 和 Apache Paimon (1.2.0)；

  ·内核增强：内核增加了缓存系统以提升性能。

版本变更注意事项:

  为支持更现代化的技术栈，Gravitino 1.0.0 服务器端现要求 JDK 17 环境，同时 Python 客户端不再支持 Python 3.8。

  Apache Gravitino 1.0.0 现已正式上线。如需了解更多信息、查看完整的发布说明并开始使用，请访问 GitHub 仓库：https://github.com/apache/gravitino , 或项目官网 https://gravitino.apache.org/ 。

关于 Apache Gravitino

  Apache Gravitino™ 是一个开源的、高性能、可扩展的统一元数据湖，旨在集中管理海量、异构的数据、计算及 AI 资产。它为数据工程师和 AI 工程师提供了一个统一的元数据接口，极大地简化了数据治理与管理，从而加速了数据驱动应用和 AI 模型的发展。

Apache, Apache Gravitino, Gravitino, Apache Iceberg, Apache Paimon, Apache Hive, Apache Spark, Apache Kafka 以及 Apache Flink 是 Apache 软件基金会在美国和/或其他国家的注册商标或商标。所有其他品牌和产品名称可能是其各自所有者的商标。

每一个 Star⭐️
都是对开源社区最有力的支持**