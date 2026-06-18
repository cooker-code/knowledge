---
title: Flink CDC：新一代实时数据集成框架
author: Apache Flink
date:
url: http://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247511530&idx=1&sn=c0567366399fce85dd76a81d5d794ccd&chksm=fc0d5c0b2dbbee31472d680fe1ca7ec3aed1a0e589e61f8f1a1421240c376d7085a6b3a2cd57&mpshare=1&scene=24&srcid=1105949oGImH7tyTBrKtCwAd&sharer_shareinfo=8d1e13bb9304fa5683db1e2442a769c4&sharer_shareinfo_first=8d1e13bb9304fa5683db1e2442a769c4#rd
---

> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/030302_核心知识点/FlinkCDC版本演进与Pipeline连接器边界|FlinkCDC版本演进与Pipeline连接器边界]]


**摘****要****：**本文整理自阿里云实时计算团队 Apache Flink Committer 和 PMC Member 任庆盛老师在 Apache Asia CommunityOverCode 2024中的分享。内容主要分为以下四个部分：

1. 什么是 Flink CDC

2 Flink CDC 版本历程

3. Flink CDC 内部实现

4. Flink CDC 社区与未来规划

**Tips：点击**「阅读原文」**跳转阿里云实时计算 Flink～**

**01**

**什么是 Flink CDC**

Flink CDC 是一个数据集成框架，它基于数据库日志的 CDC（变更数据捕获）技术实现了统一的增量和全量数据读取。结合 Flink 出色的管道能力和丰富的上下游生态系统，Flink CDC 可以高效地实现海量数据的实时集成。

#### ****1.1 Flink CDC 使用场景****

Flink CDC 可以应用在多种场景中。比如数据同步，可以将上游数据库中的数据同步至下游数据仓库、数据湖等。用户还可以借助 Flink CDC source 实现实时物化视图，结合下游 Flink 作业处理逻辑实现更丰富的业务场景。此外用户还可以使用 Flink CDC 捕获的变更数据基于业务逻辑进行数据分发。

作为一款数据集成框架，Flink CDC 对接了非常丰富的上下游数据库、数据湖仓和消息队列等外部系统，如 MySQL、PostgreSQL、Kafka、Paimon 等。

#### ****1.2 与传统数据集成流水线比较****

一个传统的数据集成流水线通常由两套系统构成：全量同步和增量同步。其中全量同步会使用 DataX、Sqoop 等系统，增量同步需要使用另外一套系统，如 Debezium、Canal 等等。在全量同步完成后，可能还需要额外的一步合并操作将增量表和全量表进行合并，最终得到与上游一致的快照。这种架构的组件构成较为复杂，为系统维护带来了很多困难。

相比于传统数据集成流水线，Flink CDC 提供了全量和增量一体化同步的解决方案，对于一个同步任务，只需使用一个 Flink 作业即可将上游的全量数据和增量数据一致地同步到下游系统。此外 Flink CDC 使用了增量快照算法，无需任何额外配置即可实现全量和增量数据的无缝切换。

**02**

**Flink CDC 版本历程**

Flink CDC 诞生于 2020 年 7 月，中间经过不断迭代优化，发布了多个大版本。2021 年 8 月，Flink CDC 发布了 2.0 版本，首次为 MySQL CDC source 引入增量快照算法，实现了全增量同步无缝切换。2022 年 11 月，Flink CDC 发布 2.3 版本，将大多数 connector 对接至增量快照框架。2023 年 12 月，Flink CDC 推出 3.0 版本，正式将 Flink CDC 项目升级为实时数据集成框架，提供 YAML API，为数据同步提供端到端解决方案。

**03**

**Flink CDC 内部实现**

#### ****3.1 Flink CDC YAML****

在 Flink CDC 2.x 的时代，Flink CDC 只提供一些 Flink source，用户仍然需要自己开发 Flink DataStream 或 SQL 作业实现数据同步逻辑。如果用户对 Flink 不够熟悉，经常会遇到棘手的数据正确性和乱序问题。此外 Flink CDC 2.x 不支持 schema 变更，而 schema 变更是用户的业务系统中很常见而且很重要的场景。通过对用户使用场景的调研，我们发现绝大多数使用 Flink CDC 的作业都是较为简单的数据 ETL。结合上述问题，我们决定为用户提供一个全新的框架，设计一套全新的 API，专注于数据同步场景。

#### ****3.2 Flink CDC 整体设计****

Flink CDC 基于 Flink runtime 实现，因此可以充分复用 Flink 的资源管理和在不同环境上部署的能力。针对各种数据集成场景，Flink CDC 深度定制了多种自定义算子，如 schema operator、router、transformer 等。为了将不同的算子进行协调和组合，Flink CDC 引入了 composer 组件，可根据用户定义的数据同步逻辑构建 Flink 作业。依托于 Flink 丰富的生态系统，开发者只需简单地封装即可快速将现有的 Flink connector 对接至 Flink CDC。此外 Flink 还提供了 CLI，只需一个脚本即可将用户的 YAML 定义使用 composer 构建成 Flink 作业，并提交至指定 Flink 集群。基于以上架构，Flink CDC 为数据集成用户提供 schema 变更同步、整库同步、分库分表同步等增强能力。

#### ****3.3 Flink CDC API****

Flink CDC API 使用 YAML 语法定义数据同步任务，即易于开发者进行手动开发，又可以高效地使用机器进行处理。YAML API 针对数据集成场景设计，用户只需定义同步数据源和数据目标端即可快速搭建起一个实时同步流水线。此外用户还可以在 YAML 中定义 routing 和 transformation 实现自定义数据分发和变换。用户不再需要熟练掌握 Flink 作业开发与内部实现，即可使用 Flink CDC 搭建实时数据集成流水线。

Flink CDC 提供的 CLI（flink-cdc.sh）进一步简化了用户提交 Flink CDC 任务的流程。用户只需执行一行命令，CDC composer 会将 source、sink、自定义 CDC runtime 构建成 Flink 任务，创建 Flink JobGraph 后提交至 Flink 集群。

#### ****3.4 Flink CDC Pipeline 连接器****

Flink CDC 定义了自己的数据源和目标端连接器的接口，以适配 Flink CDC 内部的数据结构。Flink CDC pipeline connector 基于 Flink connector，只需进行简单的数据转换封装，即可快速复用现有的 Flink connector，将其对接到 Flink CDC 生态系统中。为了实现 schema 变更处理能力，Flink CDC 定义了 MetadataAccessor 和 MetadataApplier，分别对源端和目标端的 schema 等元信息进行获取和处理，实现 schema 变更的实时同步。

#### ****3.5 Flink CDC Source 增量快照算法****

为了实现全量和增量的一体化同步，Flink CDC source 使用增量快照算法，既实现了全增量同步的无缝切换，而且采用了无锁设计，避免全量同步时的锁表动作对上游业务的影响。在增量快照算法中，数据库的全量数据被切分为独立的数据块（chunk），分发给 source 的各个并发进行读取。考虑到在全量读取过程中数据还有可能发生变化，在开始读取前，source 将 binlog 的当前位点记为低水位线（low watermark），在全量读取结束后再次将 binlog 最新位点记录为高水位线（high watermark），随后读取高、低水位线之间的变更数据，将其合并到已读取的全量数据块中，从而构建一个与上游完全一致的数据块。在完成全部数据块的读取之后，source 会根据记录的高水位线确定切换位点，实现全量和增量的无缝切换。

#### ****3.6 Flink CDC 对 Schema 变更的支持****

Flink CDC 通过定制化的 schema operator 以及 schema registry 的协调，实现对上游 schema 变更的实时同步。当 schema operator 感知到上游发生 schema 变更后，会将变更信息同步给 schema registry，并暂停数据流的处理。schema registry 首先插入 Flush 事件将下游数据全部从 sink 推出，在收到全部 sink 的确认后，通过 MetadataApplier 将 schema 变更应用在下游系统中，在完成 schema 变更后，schema registry 通知 schema operator，并恢复数据流处理，完成整个 schema 变更的流程。

#### ****3.7 Flink CDC 对数据分发处理的支持****

Flink CDC 定制了 router 算子，实现对变更数据的分发和合并。用户可以在 YAML 中使用 route 字段修改变更数据的目标数据库和表名，将数据同步至指定目标端，同样也可以通过指定多对一的路由规则，将多个表合并为目标端中的一张表。

#### ****3.8 Flink CDC 对数据变换的支持****

通过在 YAML 中使用 transform 字段，用户可以在数据流上定义投影、过滤、增加元信息列等数据变换操作，调整数据内容后同步至下游。transform 使用类 SQL 语法，既可以让用户简单上手开发，又保留了对更多类型变换支持的可扩展性。

#### ****3.8 Flink CDC 数据结构****

Flink CDC 在数据流中定义了数据和 schema 信息的协议：

* 数据流以 CreateTableEvent 开始来描述起始 schema
* 后续所有的 DataChangeEvent 需要遵循其前方的 schema
* 当 schema 发生变更时，需要在数据流中发送一个新的 SchemaChangeEvent 以描述 schema 变化

这种设计的优势在于实现了数据和 schema 的分离，大大降低了数据的序列化成本。此外 Flink CDC 为数据变更事件使用了压缩的二进制格式，进一步提升了性能。

**04**

**Flink CDC 社区与未来规划**

目前 Flink CDC 已经有超过 160 位贡献者，项目获得 5k+ star，1000+ commit。未来 Flink CDC 将着力于扩展生态，对接至更多的外部系统，如 PostgreSQL、Iceberg 等，并且将支持更多的 schema 变更类型和数据类型。另外 Flink CDC 也会持续提升生产稳定性，包括对异常处理方式进行自定义配置、提升测试覆盖等等。作为 Apache Flink 的子项目，Flink CDC 使用与 Flink 一致的贡献流程。欢迎各位用户和贡献者在 Flink 邮件列表中咨询和讨论，使用 Apache JIRA 创建 issue，在 GitHub 上提交 PR！

---

欢迎大家多多关注 Flink CDC，从钉钉用户交流群[1]、微信公众号[2]、Slack 频道[3]、邮件列表[4]加入 CDC 用户社区，以及在 Flink CDC GitHub 仓库[5]上参与代码贡献！

---

[1] “ Flink CDC 社区 ② 群”群的钉钉群号：80655011780

[2] ” Flink CDC 公众号“的微信号：

[3] https://flink.apache.org/what-is-flink/community/#slack

[4] https://flink.apache.org/what-is-flink/community/#mailing-lists

[5] https://github.com/apache/flink-cdc

**活动推荐**

---

阿里云基于 Apache Flink 构建的企业级产品-实时计算 Flink 版现开启活动：

新用户复制下方链接或者扫描二维码即可0元免费试用 Flink + Paimon

了解活动详情：https://free.aliyun.com/?pipCode=sc

---

▼ 关注「**Apache Flink**」，获取更多技术干货 ▼

**点击「阅****读原文****」跳转阿里云实时计算 Flink** ****～****