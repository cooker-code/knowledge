> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040103_StarRocks/040103_核心知识点/StarRocks实时数仓与湖仓接入边界|StarRocks实时数仓与湖仓接入边界]]
---
title: Fresha 的实时分析进化：从 Postgres 和 Snowflake 走向 StarRocks
author: StarRocks
date:
url: https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247498689&idx=1&sn=f60dfc5caf4a3771060ec0f47e812090&chksm=e84e02b6f0afd8021a43cdfa110de785da69593e58ddad9395e34a15e00d94cea0d5d5856988&mpshare=1&scene=24&srcid=1218LakPsb2GHqV9buUFS0gU&sharer_shareinfo=3d063d2ca76940e47673ec62ed02d954&sharer_shareinfo_first=3d063d2ca76940e47673ec62ed02d954#rd
---

作者：**Anton Borisov**

**导读：**

**开源无国界，在本期「StarRocks 全球用户精选案例」中，我们走进 Fresha——全球领先的美业、健康与自我护理行业一站式平台，服务于全球数以百万计的消费者与商家。**

随着业务规模的快速增长，Fresha 曾面临典型的架构失配挑战：Postgres 频繁因 OLAP 需求过载，而 Snowflake 在应对高频准实时分析时又面临成本与时效性限制。为此，Fresha 引入了 StarRocks，在保持 Lakehouse 为唯一事实源的前提下，构建了兼具“联邦查询”与“内部表加速”的混合架构。

自 2025 年春季上线以来，Fresha 成为英国最早在生产环境规模化落地 StarRocks 的先行者之一。本文将深度拆解其选型逻辑、落地架构以及性能优化等方面的实战经验。

# 现状与挑战

到 2024 年中期，Fresha 的数据平台呈现出一种极其矛盾的状态：虽然原有的技术栈勉强能跑通，但每个组件都在承担着超出其设计初衷的工作：

* **Postgres****(****OLTP****)**：原本用于支撑面向用户的业务系统，却承担了大量的 Ad-hoc 和产品仪表盘需求。宽表 Join 和重度聚合导致了 Head-of-line blocking 和 Noisy neighbor效应，甚至偶尔会引发“为什么下单接口变慢了？”这种生产事故。
* **Snowflake (****BI****/数据导出)**：虽然能很好地处理传统 BI 看板和大规模数据导出，但在应对高频交互、准实时的产品及运营分析时，无论在成本还是响应速度上都难以为继。

这种架构失配导致高峰期系统响应变慢、仪表盘延迟波动。我们意识到，必须寻找一个能够同时填补两个缺口的分析引擎：

* 在不消耗 Postgres 资源的前提下，能够高效处理海量历史数据。
* 支持标准协议以降低迁移成本，且随着业务增长，性能与扩展性需保持高度可预测。

**核心诉求：填补拼图的缺口**

为此，我们为理想的分析工具划定了几个硬性约束：

* 将历史分析需求从 OLTP 路径中剥离。
* 坚持开放格式优先（基于对象存储的 Iceberg/Paimon），将 Lakehouse 作为唯一事实源，在不增加 Postgres 存储负担的前提下处理历史数据。
* 支持 MySQL 协议、标准驱动，尽量减少改造与工具替换成本。
* 扩展能力可预期：能从容应对流量高峰，而非耗费数天进行容量规划。
* 核心链路达到秒级至分钟级时延，其余链路保持分钟级。
* 低运维复杂度：减少定制化管道与额外系统。

1

### ****为什么选择 StarRocks？****

于上述要求，StarRocks 凭借其混合查询模式脱颖而出：它既能通过外部 Catalog 实现对开放格式数据的联邦查询（保证广度），又支持将时序敏感的指标直接接入内部列存表（保证深度与性能）。

* **原生列式存储：**支持支持明细、聚合及主键模型，并支持高吞吐写入（如 Flink 或 Routine Load）。这是实现核心指标“准实时”可用的最短路径。
* **湖仓加速能力：**通过 Catalog 直接读 Iceberg / Paimon / Hive 等开放表格式，并将 Filter与 Projection 下推以减少对象存储扫描开销——这是处理大规模历史数据的理想方案。
* 物化视图自动查询改写：可定义增量汇总或预关联，优化器会自动将符合条件的查询改写为命中对应的物化视图。
* **存算分离架构：**计算资源可按需弹性扩缩，无需在节点间重新平衡数据，确保了业务高峰期成本与时延的可预测性。
* **MySQL****协议与生态兼容**：与常见 BI 工具及主流客户端库开箱即用，工程师可以快速接入与落地。

（StarRocks 采用存算分离架构：客户端通过 MySQL 协议连接到 FE 节点（Leader、Follower/Observer），由 FE 负责 Catalog 管理与查询协调；CN 节点承担实际查询执行并进行数据缓存。持久化数据存放在分布式存储中，因此扩展算力时只需要新增 CN 节点，无需对存储数据做重分布。）

2

### ****新架构一览****

你可以将整个平台想象成一条统一的数据摄取主干，并延伸出三条**链**路：一条是进入 StarRocks 内部表的**实时链路**，一条是进入 Iceberg/Paimon 的**历史链路**，以及一条进入 Elasticsearch 的**搜索链路**。StarRocks 居中作为**统一的****SQL****入口**。工程师通过标准的 MySQL 协议接入，即可实现跨三条链路的**关联查询**，而无需关注数据的存储位置。

（Fresha 的高层数据流如下：以 Postgres 为主的数据源通过 Debezium + Schema Registry 接入 Kafka；计算层使用 Flink 与 Spark；湖仓层采用 Iceberg + Paimon；下游由多个 Sink 承接，其中 StarRocks 作为统一的 SQL 查询入口。StarRocks 通过外部 Catalog 访问湖仓数据，计算层则分别服务实时与历史链路，对湖仓进行读写。）

1. **写入主干（Ingestion spine）。**它实时捕获 Postgres 的 CDC 变更事件并流向 Kafka，同时配合 Schema Registry 使用 Avro 格式进行序列化。这为我们提供了一个强类型、可平滑演进的事件封装层，既满足了 CDC 需求，也为下游消费者构建了一个单一、可靠的数据主干。

Kafka 在这里承担了扇出点（Fan-out point）的角色：Flink 与 Spark 从同一个事实源获取数据，并根据不同的访问模式，将数据写入到最适合的存储引擎中。

2. **实时链路（StarRocks 内部表）**。针对时效性达“秒级”、且用户体验极度依赖尾部延迟（Tail Latency）稳定性的场景，Flink 会将数据直接写入 StarRocks 的内部列存表。

在表模型选择上，我们针对不同业务场景进行了适配：主键模型（Primary Key）用于承载需要实时保新的变更流；聚合模型（Aggregate Key）用于执行指标预计算（如 Sum/Count/Min/Max）；**而**明细模型（Duplicate Key）则负责接收那些后续需要进行 Compaction 或异步汇总的流式数据。

这种设计刻意压缩了数据路径：即“Kafka → Flink → StarRocks → Dashboard/API”**的**极短链路。通过将对象存储从核心路径中剥离，我们能够依靠 StarRocks 的横向扩展来应对流量峰值，而不必受限于远程存储的 List 或 Get 请求。

在这些内部表之上，我们为常用的聚合与预关联定义了物化视图。StarRocks 的优化器会自动将符合条件的原始查询透明改写，使其直接命中这些物化视图。这使得我们的研发团队只需编写最基础的 SQL 即可。

3. **历史链路(Iceberg/Paimon)。**并非所有查询都具有极高的紧迫性，而且几乎没有哪类查询仅关注“当下”。我们将业务侧 CDC 数据落地到 Paimon；同时，Flink 和 Spark 负责将长期的事实表与缓慢变化维（SCD）写入对象存储上的 Iceberg。其中，Spark 处理更为繁重的工作： backfill、repair、compaction ，以及生成跨大跨度时间范围的一致性快照。

这种模式为我们提供了低成本且持久的历史存储，并支持完善的 Schema 演进和分区机制；同时也确保了 Lakehouse 作为唯一事实源的地位。StarRocks 通过外部 Catalog 直接接入 Iceberg 和 Paimon，使得历史查询能够在不迁移数据的情况下，直接在开放格式上进行联邦查询。当回灌数据落地后，我们可以重建或刷新 StarRocks 内部相关的物化视图，使历史数据的查询体验尽可能接近实时链路。

4. **搜索链路（Elasticsearch）**。部分工作负载并非严格的关系型数据，例如：模糊匹配、前缀/后缀搜索、分词以及相关性评分。我们利用 Flink 或 Spark，从相同的 Kafka/Lakehouse 事实源中将这类数据索引至 Elasticsearch，随后通过 StarRocks 的 experimental Elasticsearch Catalog 将其暴露给开发人员。

这一方案的核心价值不在于引入了 ES，而在于开发人员不再需要直接调用 ES 接口。从他们的视角来看，一个搜索密集型的索引仅仅是另一张可以被 SQL 关联查询的“表”，且使用的仍是原有的分析连接。这种设计降低了认知负荷，同时也实现了基础设施接入层的集中化。

（以 Kafka 为中心的“写入主干”由 Debezium + Schema Registry 提供强类型的 CDC 数据，并向外分为三条链路：实时链路（Flink → StarRocks 内部表 + MV → Dashboard/API）；历史链路（Flink → Paimon；Spark/Flink → Iceberg；StarRocks 通过外部 Catalog 联邦查询并按需刷新 MV）；搜索链路（Spark/Flink → Elasticsearch；通过 ES Catalog 以 SQL 方式进行关联查询）。）

StarRocks 作为统一入口，通过一个 MySQL 端点，实现了热数据、历史数据与搜索链路的统一：时延最敏感的数据切片落在内部列式表；长期事实与维度数据保留在 Iceberg/Paimon；文本密集型数据写入 Elasticsearch。StarRocks 通过外部 Catalog 统一接入这三类存储，因此工程师只需编写标准的 SQL，无需关注数据的具体存放位置。

StarRocks 的优化器与物化视图改写机制会自动规划最优查询路径：优先命中内部表或物化视图，必要时则下推至 Lakehouse 或 Elasticsearch 执行。我们采用**存算分离**模式，实现了计算与存储解耦，在应对业务高峰扩缩容时无需重分布数据，保障了尾部延迟的稳定与运维的极简。数据回灌统一落入 Lakehouse，并通过联邦查询或物化视图刷新实现感知。这种架构确保了底层数据演进的同时，上层查询接口也能保持稳定。

（在生产环境中按数据新鲜度做了分层：Hot（秒级）通过 Kafka → Flink → StarRocks 内部表；Warm（分钟级）由 StarRocks 直接查询 Iceberg/Paimon（联邦查询，必要时配合 MV 加速）；Deep history（深度历史）保留在 Iceberg/Paimon 中，由 Spark 以版本化快照方式进行回灌与补齐。）

# **案例：首页分析查询性能优化**

我们的首页承载着面向客户的分析功能——包括“优秀员工”（双月对比）、“热门服务”以及实时销售动态。起初这些功能由 Postgres 支撑，在小客户场景下表现尚可，但在大客户侧却遭遇了性能瓶颈：页面加载动辄 15-20 秒甚至直接超时，还对 OLTP 业务造成了严重的连带伤害

这是典型的失效模式：一次**冷启动查询**击穿了 buffer cache；首个请求在拖回海量数据页的过程中超时，后续请求虽能“侥幸”成功，却已污染了内存空间，进一步拖慢其他无关的事务。

我们决定将这些视图迁移至 StarRocks，并提出了一个硬性要求：**分钟级的数据时延**。用户不能在完成一笔交易后，因为看不到实时反馈而产生困惑。我们最初尝试使用 Iceberg，功能上没问题但运行层面不稳定——高频写入产生的大量小文件和 Compaction 压力，使分钟级时延难以持续保证。于是，热点链路切换至 StarRocks 内部表，并将 Iceberg/Paimon 继续作为历史数据的长期记录。

关键点在于，我们并未直接使用物化视图，而是基于内部表构建了**分层****SQL****视图**。这样开发者可以复用业务语义，而无需重复定义。整体架构如下：

* 由 Flink 写入的基础表 **rt\_sales**（Debezium → Kafka → Flink → StarRocks）；
* 作为统一语义层的 **vw\_sales\_enriched** 视图，用于补全业务关联并应用状态口径；
* 用于定义“最近成交”的 **vw\_recent\_sales** 视图（包含时间窗口与可计入的状态范围）；
* 在其之上构建的高层视图，例如 **vw\_top\_employees\_2m**、**vw\_top\_services**，均基于前述层级组合计算得到。

（首页分层视图示例：**rt\_sales**（CDC upsert 写入）→ **vw\_sales\_enriched**（业务关联、状态口径、分区过滤条件，以及衍生字段/过滤字段，例如 day、is\_eligible、provider\_bucket）→ **vw\_recent\_sales** → **vw\_top\_\***。）

由于业务语义都封装在视图里，产品团队只需要查询 **vw\_top\_\*** 和 **vw\_recent\_\***；不必记住哪些状态需要计入、“recent”具体怎么定义，或或者销售数据如何关联补全。与此同时，StarRocks 的优化器会将过滤条件与列裁剪下推至整个视图栈，在无需维护物化视图刷新任务的前提下，依然获得高质量的执行计划质量。

**最终成效：**即使在最复杂的过滤与聚合条件下，首页分析查询的响应时间也缩短至 200 毫秒左右，并达到了用户预期的分钟级时效性。Postgres 不再被当作“临时缓存”来透支，确保了 OLTP 事务的响应速度，而首页产生的分析性并发压力则由 StarRocks 承接。

深度历史数据依然保留在 Lakehouse 中（通过 Spark 回灌至 Iceberg/Paimon），而这套分层视图可以根据需要进行跨源联邦查询，从而覆盖更长的时间窗口。这意味着我们无需为不同场景开发多套代码，仅需维护一套可复用的语义定义。

（启用 StarRocks 查询链路（通过 feature flag）前后的延迟分位对比：左图为旧的 Postgres 方案，查询经常出现多秒级峰值；右图为开启 StarRocks 后，p95 降至接近 1 秒以内，并且长尾（p99/p99.9）的峰值基本消失。）

# **实践中的问题与解决方案**

1

### 实现无误的 DDL 迁移

我们构建了一套 ActiveRecord 风格的迁移工具：采用层级命名规范，为每项变更编写显式的 up/down SQL，并在 StarRocks 中维护一个声明式的 Schema 版本号（这是一个原子递增的单一事实源）。

由于 StarRocks 的许多 DDL 操作是异步执行的，该工具会持续轮询变更状态，直到所有后台任务达到最终的 FINISHED 状态后才会更新版本号；一旦失败，它将通过配对的 down SQL 进行回滚。最终效果是：实现了一套与 StarRocks 语义对齐、可逆且支持协作安全的 Schema 演进流程。

2

### ****查询性能分析****

我们统一使用 EXPLAIN ANALYZE 生成的 Profile，并梳理出一套符合常识的核心指标（扫描字节数、命中的分区数量、Join 类型、P50/P95）。这让所有人对“什么变慢了”拥有了一致的判断框架：是分区过多、Join 策略不合适，还是由于过滤条件无法下推。

3

### **分区策略：不向业务代码“泄露”底层细节**

我们按时间进行分区，并按业务键（例如 `provider_id`）进行分桶。为了防止开发人员因疏忽导致全表扫描，我们将过滤谓词封装在视图内部。

例如，`vw_recent_sales` 视图中直接定义了“Recent”的时间范围及合规状态，更高级别的视图则基于此构建。Planner 依然能将过滤条件透传至底层引擎，但调用者无需再记忆复杂的分区计算逻辑。

4

### **维度关联：避免大规模 **Shuffle****

大事实表与小维度表的关联采用 Broadcast 模式；大事实表与大维度表之间的关联则优先使用 Colocate 模式（通过对齐分桶键与分桶数实现），在无法满足 Colocate 条件时则退而求其次使用 Bucket-shuffle。

我们对维度表进行版本化管理，并尽可能精简字段（Narrow Tables）以适配 Broadcast；当某个维度表规模增长到不再适合 Broadcast 时，我们会将其提升至 Colocate Group 中，并调整其分桶策略以匹配主事实表。

5

### **数据跳读与索引取舍**

为了降低范围查询和点查的成本，我们充分利用了 StarRocks 的 Zone Map（每个 Segment 的最大/最小值过滤）以及基于排序列的 prefix/short-key index。此外，我们仅在能产生实质性收益（Move the needle）的场景下，有选择性地添加 Bloom Filter 或 Bitmap 索引。

我们的原则是：在添加索引前，必须通过 Profile 证明其确实减少了扫描字节数；同时，定期清理不再使用的旧索引。

6

### **Schema 演进**

所有 Schema 变更都始于 Avro Schema Registry 的兼容性检查；数据写入方（Writers）最后才进行发布。内部表遵循“仅增量”原则，优先添加新列；视图层则采用版本化定义（如 `vw_sales_enriched_v2`），并配合一个名为 `vw_sales_enriched` 的视图指针，待数据 Backfill完成后再进行原子切换。Flink Sink 均具备幂等性或通过主键（PK）进行数据对齐。此外，CI 环节会拦截任何可能导致下游模型失效的变更。

# **总结**

StarRocks 正逐渐成为我们日常分析中可靠的核心工具：它提供了统一的 SQL 接入层，将实时链路、历史链路与搜索链路有机统一；在存算分离架构下，性能稳定可靠；同时具备开发者友好的易用性，让团队能够通过平实的标准 SQL 快速交付业务，而非陷入复杂的定制化管道中。

通过这一套架构，我们实现了预期的工程目标：内部表上的准实时读取、开放格式上的历史数据联邦查询，以及通过 ES Catalog 实现的搜索关联查询。更重要的是，在实现这一切的同时，我们依然保持了 Lakehouse 作为唯一事实源的架构地位。

本文翻译自：https://medium.com/fresha-data-engineering/how-we-accidentally-became-one-of-uks-first-starrocks-production-pioneers-7db249f10010

如想进一步学习与交流，欢迎添加小助手，加入 StarRocks 社区群，与更多开发者共同探索交流。

**关于 StarRocks**

StarRocks 是隶属于 Linux Foundation 的开源 Lakehouse 引擎 ，采用 Apache License v2.0 许可证。StarRocks 全球社区蓬勃发展，聚集数万活跃用户，GitHub 星标数已突破 11,000，贡献者超过 500 人，并吸引数十家行业领先企业共建开源生态。

StarRocks Lakehouse 架构让企业能基于一份数据，满足 BI 报表、Ad-hoc 查询、Customer-facing 分析等不同场景的数据分析需求，实现 "One Data，All Analytics" 的业务价值。StarRocks 已被全球超过 500 家市值 70 亿元人民币以上的顶尖企业选择，包括中国民生银行、沃尔玛、携程、腾讯、美的、理想汽车、Pinterest、Shopee 等，覆盖金融、零售、在线旅游、游戏、制造等领域。

**行业优秀实践案例**

**泛金融：**[中国民生银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)**[｜](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)**[平安银行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493044&idx=1&sn=580ea0a521f9c9e4099356022536a585&chksm=e9f29a90de851386c04da4c3810b3c1ce0fce8e7f128fcbee0fb1c1a480a17aa5582325f20ee&scene=21#wechat_redirect)｜[中信银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[四川银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[南京银行](https://mp.weixin.qq.com/s?__biz=MzkxOTQyNzU1NQ==&mid=2247484420&idx=1&sn=54b813186be038ad1db9db7957eb5a19&scene=21#wechat_redirect)｜[宁波银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中原银行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487331&idx=1&sn=e6cdcdf2c46762135b6d95bf58e47664&chksm=e9f17047de86f9519e4d58f47745fe64788f2a5b3117133fc4b39db1a50c50d163a467f20950&scene=21#wechat_redirect)｜[中信建投｜](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488978&idx=1&sn=68da6c95be3337048c5900ce70d56c5e&scene=21#wechat_redirect)[苏商银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247487346&idx=2&sn=263c5029751e0ec8ed81df1760deb1d6&scene=21#wechat_redirect)｜[微众银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[杭银消费金融](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[马上消费金融](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中信建投](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[申万宏源](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492791&idx=1&sn=e4eba3e3d3de2fc4e59148a2e52f7201&chksm=e9f29b93de851285bf00f6e94a0f096b24d9a494b0013d43fb42d006eabea27a615e605b5a1b&scene=21#wechat_redirect)[｜](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247485524&idx=1&sn=4a5d0f877d39ed674cbc0b8ba03847c6&scene=21#wechat_redirect)[西南证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中泰证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[国泰君安证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[广发证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[国投证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中欧财富](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247485524&idx=1&sn=4a5d0f877d39ed674cbc0b8ba03847c6&scene=21#wechat_redirect)｜[创金合信基金](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[泰康资产](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[人保财险](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[随行付](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247498100&idx=1&sn=966cd218640936c6830de7f148a4d1e4&scene=21#wechat_redirect)

**互联网：**[微信｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484928&idx=1&sn=87d2572bba55afef5abd5f7b8571a443&chksm=e9f17924de86f032c3dca0fdc69b9b8ab8b873bc9a5bc4fdb284c06c8539da0e24e6647ded93&scene=21#wechat_redirect)[小红书｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484997&idx=1&sn=8644dacbcf467f6946add574b738e5d9&chksm=e9f17961de86f0776f5cbb201601010374c7e6c81b0a3250d93a32a997edbf694f4d471185eb&scene=21#wechat_redirect)[淘宝闪购](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247498373&idx=1&sn=b3f4cd88fd5539514afb0510760cf35a&scene=21#wechat_redirect)｜[滴滴](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490839&idx=1&sn=a007c5207f15129cd8c91da15cc30093&chksm=e9f16233de86eb25f585ef9b911e54ae6dae9547bd240a9554dd52ba449a74289a13d3df99fa&scene=21#wechat_redirect)｜[B站](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492334&idx=1&sn=dd0de6197bc6f3f67ffb0704b72ae02f&scene=21#wechat_redirect)｜[携程](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491637&idx=1&sn=daec09546a550346b65414567dc8db7b&chksm=e9f29f11de851607eb14a9e006803d8f52951d367f19a2477d5abee92f309c8c8145655a959c&scene=21#wechat_redirect)｜[同程旅行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490253&idx=1&sn=93f2a0369f9a9bff2215031efd197530&chksm=e9f165e9de86ecffe737cfebf729668df7efe09e6f9c4c976dc17668dc8042ef7895aef74ea3&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488632&idx=1&sn=0f76b5ce855b3070a138cb58e5fb8112&chksm=e9f16b5cde86e24a853e743df776e25d2358e710442cb419d32c3dd504d8e328e06b5cb473fd&scene=21#wechat_redirect)[芒果TV](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490596&idx=1&sn=9de0a6ce3608875f17287a197b7847b1&chksm=e9f16300de86ea16b83b7f5d8a29b20c137b1bc641d4d190c4d73d717a56d8c104a1f585f4d7&scene=21#wechat_redirect)｜[得物](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487306&idx=1&sn=46b784eb29c1a4d645946a017deb7f8a&chksm=e9f1706ede86f97861ce9ed2952335312704fe2cc9adc35b3cfa5709f2abdeea9a2779ff18d6&scene=21#wechat_redirect)｜[贝壳](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490822&idx=1&sn=a8bc0b31aa6568f57e919632e3f15877&chksm=e9f16222de86eb34d2a4eabd28c36ff2b29f3d9ad72855e8a9064d3b84b24a1e13970a80d3e3&scene=21#wechat_redirect)｜[汽车之家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484870&idx=1&sn=74ab3350c6b7e86376c009b3ee6b5b98&chksm=e9f17ae2de86f3f45cc32c4ae153b61c57cdf6e66dd7b10ae9fb8ec689994598130257c43936&scene=21#wechat_redirect)｜[腾讯大数据](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492960&idx=1&sn=f69f94be1400c0d3260e5c1b0aaa48cb&chksm=e9f29a44de8513520a3d4e6db9abb3936c806ad82ce7194421147c96dac10638cbae5ee817dd&scene=21#wechat_redirect)｜[腾讯音乐](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493768&idx=1&sn=5076f547377aeb56ba351b03fae0730f&chksm=e9f297acde851eba21783c944fe7da28ccafe8eb86063e2100672f7982afd449441f47e73a3e&scene=21#wechat_redirect)｜[饿了么](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247494190&idx=1&sn=89f15ad79ef3aa83db4a6f9cb9ee8bce&chksm=e9f2950ade851c1ce73cb480fcab5f714564ea21f69ef75d565f02750d7760ee3128bc2319da&scene=21#wechat_redirect)｜[七猫](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247494193&idx=1&sn=61b083ca4befbb0fe951131a5efd4687&chksm=e9f29515de851c0383357500c59f07cda799d6a51ea04721326a9af54db0642068dbb6c68645&scene=21#wechat_redirect)｜[金山办公](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493868&idx=1&sn=ad2326665f7aa0b42358af6ceab0fbe9&chksm=e9f297c8de851ede838a7db7a6bbd78d735711a420e26092878f7c334da50fcc0540c648775b&scene=21#wechat_redirect)｜[Pinterest](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493896&idx=1&sn=673579fbfcaf50fe5577b2ec91c5639a&chksm=e9f2962cde851f3a6d60b30eac4c3cabd033307c0456e3cac048a934891a46c83e76dbc75eb9&scene=21#wechat_redirect)｜[欢聚集团](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486170&idx=1&sn=bd5de60aa15f03a6972467c45c58bb9b&chksm=e9f175fede86fce8c2495df1d7f6f3096b70ec8ac69ee94e12f96f3e15c256a85a94c3e3c5f7&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490839&idx=1&sn=a007c5207f15129cd8c91da15cc30093&chksm=e9f16233de86eb25f585ef9b911e54ae6dae9547bd240a9554dd52ba449a74289a13d3df99fa&scene=21#wechat_redirect)[美团餐饮](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488130&idx=1&sn=53c32ae5448505db505872e0d8e8d7e1&chksm=e9f16da6de86e4b03a1295902ac27ff3fe9a554dd76383b6d24c155fb11778bc4d4afa7d9494&scene=21#wechat_redirect)[｜](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492334&idx=1&sn=dd0de6197bc6f3f67ffb0704b72ae02f&scene=21#wechat_redirect)[58同城](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487038&idx=1&sn=583cf978f57802edd653034d83ac30af&chksm=e9f1711ade86f80cc37de844768228ba7e6a6abc06dd8b4867303334d31334e7b23273d12c39&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490253&idx=1&sn=93f2a0369f9a9bff2215031efd197530&chksm=e9f165e9de86ecffe737cfebf729668df7efe09e6f9c4c976dc17668dc8042ef7895aef74ea3&scene=21#wechat_redirect)[网易邮箱](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488632&idx=1&sn=0f76b5ce855b3070a138cb58e5fb8112&chksm=e9f16b5cde86e24a853e743df776e25d2358e710442cb419d32c3dd504d8e328e06b5cb473fd&scene=21#wechat_redirect)｜[360](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485899&idx=1&sn=c8b13a6a6f9d0d59de1721a36f52c4cd&chksm=e9f176efde86fff974e81dfe2fb72a834528141a9b0f4a99f647b11c1a2e9001822b82ddbd1e&scene=21#wechat_redirect)｜[腾讯游戏](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486696&idx=1&sn=d4bfaf2c2fb84a890922a0dfd4879522&chksm=e9f173ccde86fadaa1313f3e1c82d71f44b3e244113c712a272d3c85543765752431f8896ae6&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486696&idx=1&sn=d4bfaf2c2fb84a890922a0dfd4879522&chksm=e9f173ccde86fadaa1313f3e1c82d71f44b3e244113c712a272d3c85543765752431f8896ae6&scene=21#wechat_redirect)[波克城市](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486150&idx=1&sn=bc272d1174edb74233c584c1b7f970da&chksm=e9f175e2de86fcf4b78f09ce88ead3dc591e11edd11a3efafe74441591928d37c213b9871e7d&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486170&idx=1&sn=bd5de60aa15f03a6972467c45c58bb9b&chksm=e9f175fede86fce8c2495df1d7f6f3096b70ec8ac69ee94e12f96f3e15c256a85a94c3e3c5f7&scene=21#wechat_redirect)[37手游](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486749&idx=1&sn=610a64862a91df789faaa2469f2c8116&chksm=e9f17239de86fb2f78abd9f391d3c780f371213c93150d8617bb1c9a1845b837293f203f7d41&scene=21#wechat_redirect)｜[游族网络｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487470&idx=1&sn=e8844ab1037069989e3dc95f5e9092e8&chksm=e9f170cade86f9dcd62d810c6f7132002c4cf7329dbf20f232db82ca6cca955ffd934be04e0c&scene=21#wechat_redirect)[喜马拉雅｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247494895&idx=1&sn=f09d7e5262feb121d12b486d8bd88d29&chksm=e9f293cbde851add02bc6d438118e530d7b569b5d3b788d1abe9237edcb76542f9a62c749b58&scene=21#wechat_redirect)[Shopee](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247495088&idx=1&sn=a42354398d35329e62241705d3e79eec&chksm=e9f29294de851b82d2b55a7c2cf8d5f75a37f1025c6a1f1b07e9fdbddddc865ea55560dffafa&scene=21#wechat_redirect)｜[Demandbase](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497162&idx=1&sn=714a27698ac91c13cd942f06aa6b25d8&scene=21#wechat_redirect)｜[爱奇艺](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497211&idx=1&sn=ea35a147566d6853b9383022340c4022&scene=21#wechat_redirect)｜[阿里集团](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497320&idx=1&sn=dda2e5922343822003105e28044e12b1&scene=21#wechat_redirect)｜[Naver](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497228&idx=1&sn=d380139bc957d8420f2807ef583fc598&scene=21#wechat_redirect)｜[首汽约车](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497432&idx=1&sn=f0ff73acc5820681e20ecf028bab5694&scene=21#wechat_redirect) ｜[Cisco](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247498587&idx=1&sn=bb3ca2af1954e97c9b384d2558f35bad&scene=21#wechat_redirect)

**新经济：**[蔚来汽车｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491009&idx=1&sn=d95f01aea67e68b617cb891be823dd9a&chksm=e9f162e5de86ebf310ec56cbe8c950a69ef5a311b8213bb3adaf843069f9f315a5bede006cdd&scene=21#wechat_redirect)[理想汽车｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485820&idx=1&sn=7aa47ec35feec64e49ea462fe7506864&chksm=e9f17658de86ff4e1bd61f12d70ee8b478cc0ecd8fa7860df72ce2b5dc4b8ac05c77235809d6&scene=21#wechat_redirect)[吉利汽车](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247485625&idx=1&sn=20dabf0908da3aa394f6b2553c363141&scene=21#wechat_redirect)｜[顺丰｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484829&idx=1&sn=344fb330aac7d722a57a9591a82ca019&chksm=e9f17ab9de86f3af2e6d995b9ff8493c2a4d8e992d30e205904e3bd0ba88075061ea0158cb82&scene=21#wechat_redirect)[京东物流｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486458&idx=1&sn=88058c480d5f163155bb885e3d1fbc2a&chksm=e9f174dede86fdc8defec71ca637bc2a5a9c87dd245c6bf9c51e11f2811fcdbcd507776ccd71&scene=21#wechat_redirect)[跨越速运](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=2&sn=7912601d564276718811d53c60116a69&chksm=e9f16e7dde86e76ba4a28a99faf69c33a0553e4d383d4e083c3e0387e1e6d088fd13094c055b&scene=21#wechat_redirect)｜[沃尔玛](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[屈臣氏](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[麦当劳](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[大润发｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488098&idx=1&sn=cae2b42fd8fdb77ed4e67fc83173291f&chksm=e9f16d46de86e450859f842b471de1ee4e57118901f6f1f7f2c545fefdbdfbe355c9dc3e7965&scene=21#wechat_redirect)[华润集团｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487047&idx=1&sn=74201f01931823c9cdc6abc71d3dba68&chksm=e9f17163de86f875439549071e7c79d3c2762cf245b69367be7024ed9d04380816da4348b38d&scene=21#wechat_redirect)[TCL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=1&sn=cb78e47e420191b2345aff1366709384&chksm=e9f16e7dde86e76b77ecfd0d72d164313e18ef2fad887c3e41d30693bfd5e79629018be53a6a&scene=21#wechat_redirect) ｜[万物新生](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490969&idx=1&sn=22a377677f403c37579b2309686d559b&chksm=e9f162bdde86ebab62ce4bd3905d63870947f5b42076fc4d3d1c60a0303e2565a01cb5e2b644&scene=21#wechat_redirect)｜[百草味](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487394&idx=2&sn=242d444b7c02721b503de9383329edcd&chksm=e9f17086de86f990d2e18132c2b9beb3fa3984f5accb03cb34e0d479e8575666426c5508913c&scene=21#wechat_redirect)｜[多点 DMALL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484971&idx=1&sn=8f7e7ec424335b81fb34f1da85efbf8f&chksm=e9f1790fde86f019016e67a8a820f866bcbffe484414fa93b403605f957874bb7450672bfc3d&scene=21#wechat_redirect)｜[酷开科技](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486836&idx=1&sn=399086d79a513d311083c30ec7746589&scene=21#wechat_redirect)[｜vivo](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497338&idx=1&sn=d34ba1738661b7200efbe6fe9950036f&scene=21#wechat_redirect)｜[聚水潭](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492598&idx=1&sn=0d25b7c62770e39a0961acd9501e4ad5&chksm=e9f29cd2de8515c400f7c6076ef91d6f13121d6f0dfd0f2743fb2afc16d07fc8e61d440a330a&scene=21#wechat_redirect)｜[泸州老窖](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中免集团](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[蓝月亮](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[立白](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[美的](https://mp.weixin.qq.com/s?__biz=MjM5NzIzODg0MQ==&mid=2656464503&idx=2&sn=765854a37cc14f3032699266110c51b0&scene=21#wechat_redirect)｜[伊利](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[公牛](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[碧桂园](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497443&idx=1&sn=32057c373f4bc6800f76fe77d5d25be0&scene=21#wechat_redirect)