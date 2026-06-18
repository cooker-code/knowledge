---
title: Doris/StarRocks 高频面试题通关指南
author: 涤生大数据
date: 涤生-阳哥涤生-阳哥
url: https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247494954&idx=1&sn=4523ac15ce224eaafc204913f6ee6d53&chksm=ce12c4b0a02d61a34372eb311188d132d288b925f0361b8228681afbc16cc2e8a14860990759&mpshare=1&scene=24&srcid=0519CBjwD3LDGjEIpkMv3MmM&sharer_shareinfo=dc214ad74282f5a45b4bbbf2a00b7067&sharer_shareinfo_first=dc214ad74282f5a45b4bbbf2a00b7067#rd
---

点击上方蓝字关注我们

对于大数据开发和数据仓库工程师来说（尤其是实时方向来说），Apache Doris 和 StarRocks 已经成为面试中无法绕开的高频重头戏。

本文基于**真实的面试反馈**，为你疯狂输出一波 Doris/StarRocks 最高频的面试题集锦（100%真题，公司名字暂不方便透漏了）。文章梳理了架构原理、核心特性、表设计与优化等方向内容，赶紧收藏，助你斩获心仪的 Offer！

## 1.架构与底层原理相关

### 1.1 Doris/StarRocks 的整体架构介绍

（1）存算一体架构

采用 “FE（前端）+ BE（后端）” 分布式架构：

* FE：负责元数据管理、SQL 解析优化、查询规划及集群调度。
* BE：承担数据存储、查询执行（计算）及数据副本管理。

> ❝
>
> 整体介绍

* 设计：计算与存储在同一 BE 节点，数据本地磁盘存储 + 本地并行计算。
* 优势：数据本地性强，无网络 IO 开销，查询性能极致；架构简洁，运维成本低。
* 技术细节：数据按 “分片（Tablet）” 分布式存储，多副本保障高可用；BE 通过 MPP 并行框架 + 向量化引擎执行计算。
* 适用：数据规模稳定、对亚秒级查询要求高的场景（如实时业务监控、核心报表）。

（2）存算分离架构

* 元数据层： 负责请求规划、查询解析以及元数据的存储和管理。
* 计算层： 由多个计算组组成。每个计算组可以作为一个独立的租户承担业务计算。每个计算组包含多个无状态的 BE 节点，可以随时弹性伸缩 BE 节点。
* 存储层： 可以使用 S3、HDFS、OSS、COS、OBS、Minio、Ceph 等共享存储来存放 Doris 的数据文件，包括 Segment 文件和反向索引文件等。

> ❝
>
> 整体介绍

* 优势：存储 / 计算可独立扩缩，云原生弹性好；利用对象存储实现海量数据低成本存储。
* 技术细节：BE 查询时从对象存储拉取数据块，部分场景引入本地缓存层优化 IO 延迟。
* 适用：PB 级超大规模数据、突发计算需求的云原生场景（如互联网用户行为分析）。

### 1.2 存算一体架构与存算分离架构该如何选择？

在选择 Doris/StarRocks 的存算一体或存算分离架构时，可从业务需求、资源弹性、部署复杂度三个维度决策：

（1）存算一体架构

* 特点：计算与存储在同一节点，部署简单，无外部存储依赖，数据本地性强，查询性能极致。
* 适用场景：业务数据规模稳定、对弹性扩缩容要求低，追求极致查询性能的场景（如企业内部核心报表分析、数据量增长平缓的业务）。

（2）存算分离架构

* 特点：计算与存储解耦，存储依赖共享存储（如 S3、OSS），计算资源可弹性伸缩，云原生适配性强。
* 适用场景：数据量爆发式增长、需动态调整计算资源（如互联网用户行为分析、流量波动大的业务），或追求存储与计算独立扩缩的云原生场景。

简言之，若业务追求性能稳定、部署简单，选存算一体；若业务需弹性扩缩、适配云原生，选存算分离。

### 1.3 Doris/StarRocks 在数据湖架构中的应用和优势是什么？

Doris/StarRocks 在数据湖架构中主要作为湖仓一体的分析引擎，实现对数据湖的高效查询分析，核心应用与优势如下：

（1）直接对接数据湖的存储系统（如 HDFS、S3、OSS 等），支持 Parquet、ORC 等主流数据湖格式，无需将数据从湖中迁移，即可对海量数据进行多维度分析查询，实现 “湖仓一体” 的高效分析能力。

（2）关键优势

* 无缝整合：兼容数据湖多格式（Parquet、ORC 等）与存储系统（HDFS、对象存储等），直接查询湖中数据，无数据迁移成本。
* 实时分析：依托 MPP 架构和向量化执行引擎，提供亚秒级至秒级查询响应，支持数据湖场景下的实时决策分析（如用户行为实时洞察）。
* 简化 ETL：直接查询湖中原始数据，减少传统 ETL 的 “抽取 - 转换 - 加载” 步骤，降低数据处理复杂度与时间成本。
* 成本优化：结合数据湖的低成本存储（如对象存储）与 Doris/StarRocks 的高效查询能力，实现 PB 级数据的经济化分析。
* 弹性扩展：支持水平扩缩容计算节点，可根据数据湖数据量的增长灵活调整计算资源，适配业务的爆发式数据需求。

### 1.4 介绍下 Doris 的数据分布、数据存储

Doris 采用 “分区（Partition）+ 分桶（Bucket）” 的两级数据分布策略：

两级分布： 第一级通常基于时间或地域进行 Partition（逻辑划分）；第二级基于高基数列（如 user\_id）进行 Hash 分桶，将数据打散成多个 Tablet（数据分片）。Tablet 是分布式存储和多副本复制的基本物理单元。

存储文件关系： 数据导入成功后，在物理磁盘上会产生一个带有版本号的 Rowset。Rowset 内部由多个连续存放的 Segment 文件 组成（由 Segment Writer 生成）。

合并逻辑： 后台的 Compaction 线程会定期将多个小 Rowset 按照版本号进行物理合并，最终融合成大 Tablet，以减少文件句柄并提升查询性能。

### 1.5 介绍一下 BE 的半数写入机制与 Version

在 Doris 和 StarRocks 里，Version 就是数据的版本号，由 FE 统一分配、全局单调递增，每次导入或更新事务提交都会生成一个新版本。

它主要用来实现 MVCC 多版本并发控制，保证读写互不阻塞、查询能拿到一致的快照数据，不会读到未提交或中间状态的数据。同一主键的多条数据会靠 Version 区分新旧，Version 更大的就是最新数据，在 Unique 表和主键表做数据合并、去重覆盖时，就是以 Version 作为判断依据。

同时 Version 也用于分布式事务和多副本一致性，只有所有副本都同步到对应版本，数据才对外可见，从而保证整个集群的数据一致。后台 Compaction 也是按 Version 来合并小文件、清理旧版本，既减少文件数量提升查询性能，又能回收存储空间。

### 1.6 为什么不用 Doris 做计算引擎，而使用 Doris 做数仓？

Doris 本质上是一个集成了 “一体化存储 + 分布式计算 + 高性能查询” 的 MPP 分析型数据库，而非 Spark 这种纯通用计算引擎。

功能全链路覆盖： Doris 自带高效的列式存储引擎，支持秒级实时摄入、Upsert 强一致更新、多表流式 Join，可以直接完整覆盖数仓的全链路。

极简架构降低运维： 如果只把 Doris 当计算引擎用，意味着前面还要挂一套复杂的存储和元数据组件，不仅浪费了 Doris 优秀的底层列存和内置索引优势，还增加了不必要的架构复杂度和运维麻烦。

目前，市面上有非常多的公司使用Doris做数仓案例，非常成熟。

## 2.核心特性与性能对比

### 2.1 Doris/StarRocks 为什么查询这么快？

Doris/StarRocks 查询快的核心原因，是它们围绕 OLAP 场景（多维度分析、大表聚合、复杂查询） 做了全链路的优化—— 从存储层减少 IO 开销、计算层提升处理效率，到查询层智能规划，每一步都是为了尽可能“减少无效数据、最大化硬件利用率”。

| 优化维度 | 核心加速原理 |
| --- | --- |
| 整体架构 | 典型的 MPP（大规模并行处理） 架构，将查询拆分成多个 Fragment，在多节点并行执行。 |
| 存储层 | 列式存储 + 极致裁剪。同类数据连续存放，只读所需列（列裁剪）；支持谓词下推；利用前缀索引、ZoneMap（Min/Max）、BloomFilter、倒排索引快速跳过无关数据块。 |
| 执行层 | 全向量化执行引擎。摒弃火山模型逐行处理模式，一次处理一批数据（Block），CPU Cache 命中率极高，并完美适配 SIMD（单指令多数据） 指令集加速。 |
| 调度模型 | 引入全新的 Pipeline 执行模型（异步数据驱动，由 Push 模式替代 Pull 模式），摆脱了传统线程池与线程切换的巨大开销，多核利用率拉满。 |
| 优化器 | 具备先进的 CBO（Cost-Based Optimizer），Doris还有RBO （Rule-Based Optimization）优化器，能够充分利用了丰富的数据统计信息，自动的尽可能的采用最优的策略来优化查询。 |

### 2.2 ClickHouse 的 Join 性能为什么比 StarRocks/Doris 差？

分布式架构缺陷： ClickHouse 采用的是两层汇聚的分布式架构，缺乏完善的 MPP 分布式查询层。在大表与大表需要跨节点进行 Shuffle Join 的场景下表现极为鸡肋。

Join 策略单一： ClickHouse 核心仅支持 Global Join（类似 Broadcast）和 Local Join。当右表内存放不下时，数据不得不溢写到磁盘，性能暴跌。而 Doris/StarRocks 完美支持 Broadcast、Shuffle、Bucket Shuffle、Colocate Join 四种策略，能根据 CBO 自动选择最优解。

优化器不足： ClickHouse 缺乏基于成本的优化器（CBO），无法对 Join 操作进行有效的优化。其聚合能力采用 Scatter Gather 模式的 2 阶段聚合，第二阶段为单节点，对内存和 CPU 的消耗较大 。Doris/starrocks 拥有较完善的 CBO，能够根据数据分布和查询计划选择最优的执行方式。其聚合能力采用多节点并行聚合计算，结合 Runtime Filter 技术，进一步提高 Join 性能。

### 2.3 Doris/StarRocks 有哪些机制来保证数据的高可用？

主要通过以下机制保障数据高可用：

* 数据分片与多副本：将数据划分为多个分片（tablet），每个分片存储多个副本（replica，建表时候指定副本数）并分布在不同物理节点。即使单个节点故障，系统可从其他节点的副本读取数据，保障数据可访问性。
* 元数据冗余存储：元数据在多个节点间冗余存储，避免元数据单点故障，保障元数据的高可用与一致性。
* 多种数据恢复机制：支持备份/跨集群数据同步等方式，可在实际生产故障后，通过备份恢复数据，降低业务中断风险。

### 2.4 Doris 和 Hive 对比有什么区别？

Hive 是构建在 Hadoop 之上的离线数仓引擎，采用存算分离架构，数据存储在hdfs或者s3上，依赖 MapReduce 或 Spark 执行计算，查询延迟高，以批量 ETL、数据清洗和离线统计为主，事务与实时更新能力较弱。

Doris 是MPP 架构的实时分析数仓，存算一体、自带分布式存储（Doris3.0之后支持存算分离，数据可以存储在hdfs或者s3上，主要是为了节省成本），采用向量化执行引擎，查询延迟低、并发能力强，支持实时写入与 Upsert 更新，更适合交互式分析和高并发报表服务。

### 3.表设计与高级索引

### 3.1 Doris/Starrocks 的表模型有哪些？分别适用于哪些场景？

Doris 有Duplicate/Aggregate/Unique三种模型，Starrocks 比它多一个Primary Key，下面介绍这些表模型的机制和适用场景

（1）明细表（Duplicate Key）

机制：按指定列排序存储，数据完全保留（不聚合、不去重），支持任意维度查询。

适用场景：原始数据存储（如用户行为日志、设备埋点数据），需完整保留明细且查询维度灵活的场景。

（2）聚合表（Aggregate Key）

机制：按「聚合键」预聚合数据（如 sum、count、max 等），相同聚合键的新数据会合并旧数据。

适用场景：统计分析场景（如 PV/UV 统计、销售汇总），需高频查询汇总结果，且能容忍数据预聚合的场景。

（3）更新表（Unique Key）

机制：按「唯一键」去重，新数据覆盖旧数据（支持部分列更新），保证唯一键对应的数据唯一。

适用场景：需高频更新部分字段的场景（如订单状态更新、用户信息修改），更新维度固定且无需保留历史版本。

（4）主键表（Primary Key）（starrocks特有）

机制：支持主键级别的插入、更新、删除，提供行级事务一致性，数据按主键有序存储。

适用场景：强事务性需求（如实时业务库同步）、需保留最新状态且高频更新的场景（如用户画像、库存数据）。

核心差异：明细表强调 “完整保留”，聚合表强调 “预计算”，更新表 / 主键表强调 “数据更新”（主键表事务性更强）。

### 3.2 Doris/StarRocks 的分区和分桶有什么区别，怎么去选择？

分区是更高层级的逻辑划分，分桶是分区内部的物理存储划分（即分桶是分区内的细分）。分桶会将分区后的每段数据打散为逻辑分片Tablet。

> ❝
>
> 划分依据

* 分区：基于业务逻辑字段（如时间、地域）。
* 分桶：基于高基数字段（如用户 ID）+ 哈希算法。

> ❝
>
> 核心作用

* 分区：快速定位数据区间，减少全局数据扫描量。
* 分桶：让数据在分区内均匀分散，提升并行查询效率。

> ❝
>
> 选择与结合策略

* 选分区：数据量大，需频繁按时间、地域等字段快速过滤数据（如实时时序数据查询）。
* 选分桶：数据含高基数字段，需并行处理多维度查询（如用户行为的并发分析）。

注意：单个 Tablet（分桶）（Tablet 数 = 分区数 \* 桶数 \* 副本数）的数据量理论上没有上下界，除小表（百兆维表）外需确保在 1G - 10G 的范围内：

如果单个 Tablet 数据量过小，则数据的聚合效果不佳，且元数据管理压力大。

如果数据量过大，则不利于副本的迁移、补齐，且会增加 Schema Change 或者 物化 操作失败重试的代价（这些操作失败重试的粒度是 Tablet）。

### 3.3 Doris有哪些索引，该如何选择

为了彻底规避全表扫描，Doris/StarRocks 提供了极其强悍的索引机制，Doris 的索引分两大类：

> ❝
>
> 内置索引（自动维护，无需手动创建）

* 前缀索引：基于有序存储，自动取 Key 列前 36 字节，适合最高频的等值/范围过滤
* ZoneMap 索引：自动维护每列的 Min/Max/Null 统计，用于跳过不满足条件的数据块

> ❝
>
> 二级索引（手动创建）

* 倒排索引：适用最广，支持等值、范围、全文检索（MATCH），支持多条件 AND/OR/NOT 组合，首选推荐
* BloomFilter 索引：适合高基数列（>5000）的等值/IN 查询，存储空间中等，不支持范围查询
* NGram BloomFilter 索引：专门加速字符串 LIKE 查询
* Bitmap 索引：适合中等基数（100~100,000）列的等值查询，支持正交多条件位运算

> ❝
>
> 选择建议（三步走）

* 最高频过滤列 → 放 Key 最前面，利用前缀索引
* 非 Key 列需要加速 → 优先建倒排索引（通用性最强）
* 特殊场景补充：LIKE 查询加 NGram BloomFilter；存储空间敏感时用 BloomFilter 替代倒排索引

## 4.数据操作与性能调优

### 4.1 Flink 同步数据到 Doris，如何保证 Exactly-Once 与数据有序性？

不丢不重（Exactly-Once）： 依赖 Flink Doris Connector。底层利用了 Flink 的两阶段提交（2PC）机制与 Doris 的 Stream Load 2PC。在 Flink Checkpoint 预提交阶段，数据通过 HTTP Chunked 持续写入 BE 但不可见（状态为 Precommitted）；当 Checkpoint 成功完成后，Flink 向 Doris 发起 Commit 信号（状态变为 Committed），数据正式对外可见。如果中间任务挂掉，未提交的事务会自动 Abort 回滚。 数据有序性保障：

上游控制：上游数据源能够保证 At-Least-Once 语义，则配合 Doris 的 Label 机制，能够保证 Exactly-Once 语义。

Doris 侧 Sequence 列： 在 Doris Unique 表中启用 Sequence 列，导入时指定业务时间戳或递增序列。即使 Flink 传输由于重试出现微弱的乱序，Doris 底层也会根据 Sequence 值进行版本比较，确保 “大值覆盖小值”，绝不会出现旧数据覆盖新数据的现象。

### 4.2 讲下 Doris 表物理关联（JOIN）的几种常见方式

Doris 在处理多表关联时，底层主要有以下四种数据 Shuffle 策略：

Doris 的 JOIN 物理执行有以下几种数据 Shuffle 方式：

* Broadcast Join：将右表全量数据广播到所有参与 JOIN 的节点，左表数据不动。网络开销 = N × T(右表)，适用范围广但右表不能太大。
* Partition Shuffle Join：左右表数据都按 JOIN 条件列的 Hash 值重新分发到对应节点。网络开销 = T(左表) + T(右表)，适用于通用场景。
* Bucket Shuffle Join：左表数据不动，只将右表数据按左表的分桶分布进行 Shuffle。网络开销 = T(右表)，要求 JOIN 条件包含左表的分桶列，且左表为单分区。
* Colocate Join：左右表数据都不移动，直接在本地做 JOIN。网络开销 = 0，要求两表属于同一 Colocation Group，分桶列类型和桶数完全一致。

灵活性依次降低，但性能依次提升（前提是桶数足够多，并行度不受限）。此外，底层实现上支持 Hash Join（等值条件）和 Nested Loop Join（非等值/笛卡尔积）两种算子。

### 4.3 线上出现慢查询（Slow Query）如何排查？

Doris 慢查询排查分以下几步：

* 定位慢 SQL：通过 Doris Manager审计日志页面搜索 slow\_query，或直接查看 FE 节点的 fe.audit.log 文件，或者直接看Doris内置的审计日志表audit\_log，过滤出执行时间超过阈值（默认 5 秒）的 SQL。
* 聚合分析模式：利用日志中的 SqlDigest 字段（SQL 结构的 hash 值）识别高频或高耗时的 SQL 模式，优先优化出现最多的那类。
* 分析执行计划：用 EXPLAIN  检查执行计划，确认分区裁剪是否生效、Join 顺序是否合理、过滤条件是否下推等。
* 查看 Profile：用日志中的 QueryId 拉取对应的 Query Profile，重点看 MergedProfile 中各算子的 DependencyWaitTime（定位瓶颈算子），再通过 DetailProfile 深入分析，同时关注 InputRows 是否存在数据倾斜。

针对性调优：根据分析结果，从 Schema 设计（分桶、索引）、执行计划（Join Hint、物化视图）、执行层（并行度、Runtime Filter）逐层优化。

> ❝
>
> 面试通关寄语

回答 Doris/StarRocks 相关问题时，切忌死记硬背。一定要结合自己项目中的真实应用场景（如：“我们在项目中将 ADS 层放到了 Doris，解决了以前 ClickHouse 频繁 OOM 以及多表 Join 性能差的痛点……”），同时把 **“分区裁剪、列式存储、向量化、MPP Shuffle、Runtime Filter”** 这几个核心高频词汇自然地穿插到你的架构叙述中，体现Doris/StarRocks的极致性能。祝大家顺利通关，拿下心仪的 Offer！

Doris/Starrocks/SelectDB现在企业用的比较多了，小公司会考虑去Hadoop化，直接基于SR/doris构建数仓，大公司使用Doris构建实时数仓或者加速hive查询Olap分析，面试也越来越多。

更多精彩关注：

往期推荐

[大模型时代，Apache Doris不止是实时数仓！](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247494481&idx=1&sn=9bfbbe37dd0b124e9d030ac68e8426e2&scene=21#wechat_redirect)

[Apache Doris 进化：SQL + AI 打造大模型时代的“超级外挂”](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247494311&idx=1&sn=47d520b0af375e443bba7ed0568e8429&scene=21#wechat_redirect)

[Doris vs StarRocks：一文看懂两大国产 OLAP 引擎的异同与选择指南](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247493265&idx=1&sn=949db2862e5be5d8ea857ac23d1b8154&scene=21#wechat_redirect)

[日均亿级数据的实时分析：Doris如何接过Spark的接力棒？](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247493010&idx=1&sn=325b43b8d6f7879a6aea7bedd9845a61&scene=21#wechat_redirect)

[Doris Summit 2025 报名启动！议程全面公开，11 月 5-6 日敬请关注](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247492951&idx=1&sn=bf17c7e43d28dc06a1f2f2cd6a0314a3&scene=21#wechat_redirect)

[Apache Doris性能优化全解析：慢查询定位与引擎深度调优](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247492570&idx=1&sn=a6a77fc247f3fd7b9718cf9ed0c0b36a&scene=21#wechat_redirect)

[如何突破数据传输瓶颈？且看Doris Arrow Flight SQL 的技术实践！](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247492477&idx=1&sn=904513ebd52139647987f89a9a33fcc4&scene=21#wechat_redirect)

[揭秘 Doris 高并发点查询：原理与优化](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247492317&idx=1&sn=9bbaaf5bcee58083dd4db36dc11b6e4a&scene=21#wechat_redirect)

[Doris 物化视图：原理、使用及常见问题处理](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247492228&idx=1&sn=964cb705bf9b07e7e6b0826e39b908e6&scene=21#wechat_redirect)

[Apache Doris 在数据仓库中的作用与应用实践](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491890&idx=1&sn=4f8b503b1320d2c6aa51dd56b322e4a0&scene=21#wechat_redirect)

[一文吃透！Doris 冷热分层技术全解析](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491732&idx=1&sn=83e420e2f61d9111c1e7d60c469c42e7&scene=21#wechat_redirect)

[海量数据存储与分析：HBase vs ClickHouse vs Doris 三大数据库优劣对比指南](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491731&idx=1&sn=36a66d99ccce371ce98ce912f72fef88&scene=21#wechat_redirect)

[探索Doris：日志分析的新宠，是否能取代老牌ES？](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491542&idx=1&sn=5a7dcfbb575ebd1479bf781cabd629c3&scene=21#wechat_redirect)

[Doris 湖仓一体：数据分析新范式](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491413&idx=1&sn=2381496db7087dd30b98d9922a7e6dfe&scene=21#wechat_redirect)

[Doris 数据库性能优化深度剖析（下篇）](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247490397&idx=1&sn=74b8805c3abf692fce48060cd7fc2fbd&scene=21#wechat_redirect)

[Doris 企业性能优化深度剖析（上）](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247490223&idx=1&sn=cad6090ece969b901d31880c17d00666&scene=21#wechat_redirect)

[Doris凭什么这么强？](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247490026&idx=1&sn=af2c658dcf96847e70177244301ec5a6&scene=21#wechat_redirect)

[企业Apache Doris 建表最佳实践，都是干货](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489896&idx=1&sn=4c563b3ff8ebb93baa9c9fb0702a7971&scene=21#wechat_redirect)