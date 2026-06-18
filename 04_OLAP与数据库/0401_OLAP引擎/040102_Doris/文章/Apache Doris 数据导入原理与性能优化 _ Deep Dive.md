---
title: Apache Doris 数据导入原理与性能优化 | Deep Dive
author: SelectDB
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537979&idx=1&sn=045fc0d552afe9e628e4b7bd03b542be&chksm=ce3ae2ef4586b92c2492ed01220ce33165b54355750dfc2b887c8dbaa116d75cf9a9380cbf09&mpshare=1&scene=24&srcid=103159tfM1Cse41Pm6EFr9Yc&sharer_shareinfo=e2a91087244ecfc76ed7aceec6f49f75&sharer_shareinfo_first=e2a91087244ecfc76ed7aceec6f49f75#rd
---

对于 Apache Doris 这样的高性能分析型数据库而言，高效、稳定的数据导入是保障实时分析能力的生命线。然而，在海量数据持续写入的场景下，如何平衡导入延迟与吞吐、如何避免性能瓶颈，是开发者面临的核心挑战。 本文将深入剖析 Apache Doris 数据导入的核心原理，涵盖关键流程、组件、事务管理等，探讨影响导入性能的因素，并提供实用的优化方法和最佳实践，有助于用户选择合适的导入策略，优化导入性能。

数据导入原理

---

导入原理概述

Doris 的数据导入原理建立在其分布式架构之上，主要涉及前端节点（Frontend， FE）和后端节点（Backend， BE）。FE 负责元数据管理、查询解析、任务调度和事务协调，而 BE 则处理实际的数据存储、计算和写入操作。Doris 的数据导入设计旨在满足多样化的业务需求，包括实时写入、流式同步、批量加载和外部数据源集成。其核心理念包括：

* 一致性与原子性：每个导入任务作为一个事务，确保数据原子写入，避免部分写入。通过 Label 机制保证导入数据的不丢不重。

* 灵活性：支持多种数据源（如本地文件、HDFS、S3、Kafka 等）和格式（如 CSV、JSON、Parquet、ORC 等），能满足不同场景。

* 高效性：利用分布式架构并行处理数据，多 BE 节点并行处理数据，提高吞吐量。
* 简易性：提供轻量级 ETL 功能，用户可在导入时直接进行数据清洗和转换，减少外部工具依赖。
* 灵活建模：支持明细模型（Duplicate Key）、主键模型（Unique Key）和聚合模型（Aggregate Key），允许在导入时进行数据聚合或去重。

导入通用流程

Doris 的数据导入遵循一个标准化的核心流程，主要包括以下几个阶段：

1.提交导入任务时间

* 用户通过客户端（如 HTTP、JDBC、MySQL 客户端）提交导入请求，指定数据源（如本地文件、Kafka Topic、HDFS 文件路径）、目标表、文件格式和导入参数（如分隔符、错误容忍度）。
* 每个任务可以指定一个唯一的 Label，用于标识任务并支持幂等性（防止重复导入）。例如，用户在 Stream Load 中通过 HTTP header 指定 Label。

* Doris 的前端节点（FE）接收请求，验证权限、检查目标表是否存在，并解析导入参数。

2.任务分配与协调

* FE 分析数据分布（基于表的分区和桶分规则），生成导入计划，并选择一个后端节点（BE）作为 Coordinator，负责协调整个任务。

* 如果用户直接向 BE 提交（如 Stream Load），BE 可直接担任 Coordinator，但仍需从 FE 获取元数据（如表 Schema）。

* 导入计划会将数据分配到多个 BE 节点，确保并行处理以提高效率。

3.数据读取与分发

* Coordinator BE 从数据源读取数据（例如，从 Kafka 拉取消息、从 S3 读取文件，或直接接收 HTTP 数据流）。

* Doris 解析数据格式（如对 CSV 分割、JSON 解析），并支持用户定义的 轻量 ETL 操作，包括：

+ 前置过滤：对原始数据进行过滤，减少处理开销。

+ 列映射：调整数据列与目标表列的对应关系。

+ 数据转换：通过表达式处理数据。

+ 后置过滤：对转换后的数据进行过滤。

* Coordinator BE 解析完数据后按分区和桶分规则分发到多个下游的 Executor BE。

4.数据写入

Doris 的高吞吐写入得益于其独特的数据模型与 LSM Tree（Log-Structured Merge-Tree）存储结构的结合。LSM Tree 是一种高效的磁盘写入优化结构，通过将写操作分为内存和磁盘两个阶段，显著提升了写入性能。其核心思想是将随机写转换为顺序写，减少磁盘 I/O 开销，同时通过多级合并（Compaction）维护数据的有序性和查询效率。

* 数据首先分发到多个 BE（Backend）节点，写入内存表（MemTable），并按 Key 列进行排序。对于 Aggregate 或 Unique Key 数据模型，Doris 会根据 Key 执行聚合或去重操作（如 SUM、REPLACE），减少数据冗余，提升查询性能。

* 当 MemTable 写满（默认 200MB）或任务结束时，数据会异步写入磁盘，形成列式存储的 Segment 文件，并组成 Rowset。LSM Tree 的内存写入和异步刷盘机制确保了高吞吐量，同时通过后台的 Compaction 过程定期合并 Segment 文件，优化存储结构和查询效率。

* 每个 BE 节点独立处理分配的数据，写入完成后向 Coordinator 报告状态，确保分布式环境下写入操作的可靠性和一致性。

5.事务提交与发布

* Coordinator 向 FE 发起事务提交（Commit）。FE 确保多数副本成功写入后，并通知 BE 发布数据版本（Publish Version），待 BE Publish 成功后，FE 标记事务为 VISIBLE，此时数据可以查询。

* 如果失败，FE 触发回滚（Rollback），则删除临时数据，以确保数据一致性。

6.结果返回

* 同步方式（如 Stream Load、Insert Into）直接返回导入结果，包含成功/失败状态和错误详情（如 ErrorURL）。

* 异步方式（如 Broker Load）提供任务 ID 和 Label，用户可通过 SHOW LOAD 查看进度、错误行数和详细信息。

* 操作记录到审计日志，支持后续追溯。

导入冲突解决

在冲突解决方面， 经典的写写冲突会导致写入无法并行，从而显著降低写入吞吐量。Doris 提供了基于业务语义的冲突机制，可很好避免该问题（详见官网文档）。而 Redshift、Snowflake、Iceberg 和 Hudi 等则采用了文件级别的冲突处理，因而不具备实时更新的能力。

MemTable 前移

MemTable 前移是 Apache Doris 2.1.0 版本引入的优化机制，针对 INSERT INTO…SELECT 导入方式显著提升性能。官方测试显示该优化使得单副本导入耗时缩短约 64%（为原先的 36%），三副本导入耗时缩短约 46%（为原先的 54%），传统流程中，Sink 节点需将数据编码为 Block 格式，通过 Ping-pong RPC 传输到下游节点，涉及多次编码和解码，增加开销。Memtable 前移优化了这一过程：Sink 节点直接处理 MemTable，生成 Segment 数据后通过 Streaming RPC 传输，减少编码解码和传输等待，同时提供更准确的内存反压。目前该功能只支持存算一体部署模式。

存算分离导入

在存算分离架构下，导入优化聚焦数据存储和事务管理解耦：

* 数据存储：BE 不持久化数据，MemTable flush 后生成 Segment 文件直接上传至共享存储（如 S3、HDFS），利用对象存储的高可用性和低成本支持弹性扩展。BE 本地 File Cache 异步缓存热点数据，通过 TTL 和 Warmup 策略提升查询命中率。元数据（如 Tablet、Rowset 元数据）由 Meta Service 存储于 FoundationDB，而非 BE 本地 RocksDB。

* 事务处理：事务管理从 FE 迁移至 Meta Service，消除了 FE Edit Log 写入瓶颈。Meta Service 通过标准接口（beginTransaction、commitTransaction）管理事务，依赖 FoundationDB 的全局事务能力确保一致性。BE 协调者直接与 Meta Service 交互，记录事务状态，通过原子操作处理冲突和超时回收，简化同步逻辑，提升高并发小批量导入吞吐量。

导入方式

Doris 提供多种导入方式，共享上述原理，但针对不同场景优化。用户可根据数据源和业务需求选择：

* Stream Load: 通过 HTTP 导入本地文件或数据流，同步返回结果，适合实时写入（如应用程序推送数据）。

* Broker Load: 通过 SQL 导入 HDFS、S3 等外部存储，异步执行，适合大规模批量导入。

* Routine Load: 从 Kafka 持续消费数据，异步流式导入，支持 Exactly-Once，适合实时同步消息队列数据。

* Insert Into/Select: 通过 SQL 从 Doris 表或外部源（如 Hive、MySQL、S3 TVF）导入，适合 ETL 作业、外部数据集成。

* MySQL Load: 兼容 MySQL LOAD DATA 语法，导入本地 CSV 文件，数据经 FE 转发为 Stream Load，适合小规模测试或 MySQL 用户迁移。

如何提升 Doris 的导入性能

---

Doris 的导入性能受其分布式架构与存储机制影响，核心涉及 FE 元数据管理、BE 并行处理、MemTable 缓存刷盘及事务管理等环节。以下从表结构设计、攒批策略、分桶配置、内存管理和并发控制等维度，结合导入原理说明优化策略及其有效性。

表结构设计优化：降低分发开销与内存压力

Doris 的导入流程中，数据需经 FE 解析后，按表的分区和分桶规则分发至 BE 节点的 Tablet（数据分片），并在 BE 内存中通过 MemTable 缓存、排序后刷盘生成 Segment 文件。表结构（分区、模型、索引）直接影响数据分发效率、计算负载和存储碎片。

* 分区设计：隔离数据范围，减少分发与内存压力

通过按业务查询模式（如时间、区域）划分分区，导入时数据仅分发至目标分区，避免处理无关分区的元数据和文件。同时写入多个分区会导致大量 Tablet 活跃，每个 Tablet 占用独立的 MemTable，显著增加 BE 内存压力，可能触发提前 Flush，生成大量小 Segment 文件。这不仅增加磁盘或对象存储的 I/O 开销，还因小文件引发频繁 Compaction 和写放大，降低性能。通过限制活跃分区数量（如逐天导入），可减少同时活跃的 Tablet 数，缓解内存紧张，生成更大的 Segment 文件，降低 Compaction 负担，从而提升并行写入效率和后续查询性能。

* 模型选择：减少计算负载，加速写入

明细模型（Duplicate Key）仅存储原始数据，无需聚合或去重计算；而 Aggregate 模型需按 Key 列聚合，Unique Key 模型需去重，均会增加 CPU 和内存消耗。对于无需去重或聚合的场景，优先使用明细模型，可避免 BE 节点在 MemTable 阶段的额外计算（如排序、去重），降低内存占用和 CPU 压力，进而加速数据写入流程。

* **索引控制：平衡查询与写入开销**

索引（如位图索引、倒排索引）需在导入时同步更新，否则会增加写入时的维护成本。仅为高频查询字段创建索引，避免冗余索引，可减少 BE 写入时的索引更新操作（如索引构建、校验），降低 CPU 和内存占用，来提升导入吞吐量。

攒批优化：减少事务与存储碎片

Doris 的每个导入任务为独立事务，涉及 FE 的 Edit Log 写入（记录元数据变更）和 BE 的 MemTable 刷盘（生成 Segment 文件）。高频小批量导入（如 KB 级别）会导致 Edit Log 频繁写入（增加 FE 磁盘 I/O）、MemTable 频繁刷盘（生成大量小 Segment 文件，触发 Compaction 写放大），显著降低性能。

* 客户端攒批：减少事务次数，降低元数据开销

客户端将数据攒至数百 MB 到数 GB 后一次性导入，减少事务次数。单次大事务替代多次小事务，可降低 FE 的 Edit Log 写入频率（减少元数据操作）及 BE 的 MemTable 刷盘次数（减少小文件生成），避免存储碎片和后续 Compaction 的资源消耗。

* 服务端攒批（Group Commit）：合并小事务，优化存储效率

开启 Group Commit 后，服务端将短时间内的多个小批量导入合并为单一事务，减少 Edit Log 写入次数和 MemTable 刷盘频率。合并后的大事务生成更大的 Segment 文件（减少小文件），减轻后台 Compaction 压力，特别适用于高频小批量场景（如日志、IoT 数据写入）。

分桶数优化：平衡负载与分发效率

分桶数决定 Tablet 数量（每个桶对应一个 Tablet），直接影响数据在 BE 节点的分布。过少分桶易导致数据倾斜（单 BE 负载过高），过多分桶会增加元数据管理和分发开销（BE 需处理更多 Tablet 的 MemTable 和 Segment 文件）。

* 合理配置分桶数：确保 Tablet 大小均衡

分桶数需根据 BE 节点数量和数据量设置，推荐单 Tablet 压缩后的数据大小为 1-10GB（计算公式：分桶数=总数据量/（1-10GB））。同时，调整分桶键（如随机数列）避免数据倾斜。合理分桶可平衡 BE 节点负载，避免单节点过载或多节点资源浪费，提升并行写入效率。

* 随机分桶优化：减少 RPC 开销与 Compaction 压力

在随机分桶场景中，启用load\_to\_single\_tablet=true，可将数据直接写入单一 Tablet，绕过分发到多个 Tablet 的过程。这消除了计算 Tablet 分布的 CPU 开销和 BE 间的 RPC 传输开销，显著提升写入速度。同时，集中写入单一 Tablet 减少了小 Segment 文件的生成，避免频繁 Compaction 带来的写放大，降低减少 BE 的资源消耗和存储碎片，提升导入和查询效率。

内存优化：减少刷盘与资源冲击

数据导入时，BE 先将数据写入内存的 MemTable（默认 200MB），写满后异步刷盘生成 Segment 文件（触发磁盘 I/O）。高频刷盘会增加磁盘或对象存储（存算分离场景）的 I/O 压力；内存不足则导致 MemTable 分散（多分区/分桶时），易触发频繁刷盘或 OOM。

* 按分区顺序导入：集中内存使用

按分区顺序（如逐天）导入，集中数据写入单一分区，减少 MemTable 分散（多分区需为每个分区分配 MemTable）和刷盘次数，降低内存碎片和 I/O 压力。

* 大规模数据分批导入：降低资源冲击

对大文件或多文件导入（如 Broker Load），建议分批（每批≤100GB），避免导入出错后带来过大的重试代价过大，同时减少对 BE 内存和磁盘的集中占用。本地大文件可使用 streamloader 工具自动分批导入。

并发优化：平衡吞吐量与资源竞争

Doris 的分布式架构支持多 BE 并行写入，增加并发可提升吞吐量，但过高并发会导致 CPU、内存或对象存储 QPS 争抢（存算分离场景需考虑 S3 等 API 的 QPS 限制），会增加事务冲突和延迟。

* 合理控制并发：匹配硬件资源

结合 BE 节点数和硬件资源（CPU、内存、磁盘 I/O）设置并发线程。适度并发可充分利用 BE 并行处理能力，提升吞吐量；过高并发则因资源争抢降低效率。

* 低时延场景：降低并发与异步提交

对低时延要求场景（如实时监控），需降低并发数（避免资源竞争），并结合 Group Commit 的异步模式（async\_mode）合并小事务，减少事务提交延迟。

Doris 数据导入的延迟与吞吐取舍

---

在使用 Apache Doris 时，数据导入的 延迟（Latency） 与 吞吐量（Throughput） 往往需要在实际业务场景中进行平衡：

* 更低延迟：意味着用户能更快看到最新数据，但写入批次更小，写入频率更高，会导致后台 Compaction 更频繁，占用更多 CPU、IO 和内存资源，同时增加元数据管理的压力。

* 更高吞吐：则通过增大单次导入数据量来减少导入次数，可以显著降低元数据压力和后台 Compaction 开销，从而提升系统整体性能。但数据写入到可见之间的延迟会有所增加。

因此，建议用户在满足业务时延要求的前提下，尽量增大单次导入写入的数据量，以提升吞吐并减少系统开销。

测试数据

Flink 端到端时延

采用 Flink Connector 使用攒批模式进行写入，主要关注数据端到端的时延和导入吞吐。攒批时间通过 flink Connector 的 sink.buffer-flush.interval 参数来控制的，Flink Connector 的详细使用参考：https://doris.apache.org/docs/3.0/ecosystem/flink-doris-connector

机器配置：

* 1 台 FE： 8 核 CPU、16GB 内存

* 3 台 BE：16 核 CPU、64GB 内存

数据集：

* TPCH lineitem 数据

不同攒批时间和不同并发下的导入性能，测试结果如下：

不同 bucket 数对导入性能的影响，测试结果如下：

Group Commit 测试

性能测试数据参考: https://doris.apache.org/zh-CN/docs/3.0/data-operate/import/group-commit-manual

总结

---

Apache Doris 的数据导入优化并非单一参数的调整，而是一个涉及表结构设计、写入策略、资源配置与业务场景的系统性工程。Apache Doris 的数据导入机制依托 FE 和 BE 的分布式协作，结合事务管理和轻量 ETL 功能，来确保高效、可靠的数据写入。频繁小批量导入会增加事务开销、存储碎片和 Compaction 压力，可以通过以下优化策略来有效缓解：

* 表结构设计：合理分区和明细模型减少扫描和计算开销，精简索引降低写入负担。

* 攒批优化：客户端和服务端攒批减少事务和 flush 频率，生成大文件，优化存储和查询。

* 分桶数优化：适量分桶平衡负载，避免热点或管理开销。

* 内存优化：控制 MemTable 大小、按分区导入。

* 并发优化：适度并发提升吞吐量，结合分批和资源监控控制延迟。

用户可根据业务场景（如实时日志、批量 ETL）结合这些策略，优化表设计、参数配置和资源分配，可以显著提升导入性能。

---

欢迎扫码添加小助手，加入 Doris社区交流群，还可以****免费领取 Doris x****AI****和 100+ 企业实践案例集****，获取技术帮助、了解最新动态，并与更多开发者和用户互动。

- END -  

**更多标杆企业信赖**

智慧金融与政企：[东北证券](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247534484&idx=1&sn=be3c0e26da40a1da03dd3bc487961880&scene=21#wechat_redirect)｜[国金证券](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535362&idx=1&sn=745fd6aa178aae78e807f59932c7e5eb&scene=21#wechat_redirect)|[国信证券](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537026&idx=1&sn=19a88be7610378cb0283f43e0563cb61&scene=21#wechat_redirect)｜[杭银消金](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247517678&idx=1&sn=2fa963e0cf8194ad8a027f2c108d5459&chksm=cf2f8be9f85802ffb84237297a0c2ec6efdf1266f9213ee028f46e7e6aad43e7f10042015095&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[河北幸福消费金融](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522589&idx=1&sn=2c8e14756aa6727ef51da608dfb074f7&chksm=cf2f971af8581e0c2fdd887636a9844eef3b16c3c9832d9e62e1457cff641935db69832f885e&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[汇添富基金](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532266&idx=1&sn=12b08c27cc55dd9e6074f565ddeb9378&chksm=cf2f70edf858f9fb7d19675a068a77b92bba293fba079e267cefcb46ae5ec452b646d7b69d93&scene=21#wechat_redirect)｜[金融壹账通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522120&idx=1&sn=32bfd0bec1a56c7ecea05c088e566cd1&chksm=cf2f994ff858105979a27bd768133ff2c663e3f7ed5f894e373d834a1e35e67b15ca15c2de6d&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[陆金所控股](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247534403&idx=1&sn=a07c42af38ce146f75fbf3fc1a96f4ba&scene=21#wechat_redirect)｜[霖梓控股](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535962&idx=1&sn=1d348942d631ebc15a1a4cb5b73b2177&scene=21#wechat_redirect)｜[拉卡拉](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536412&idx=1&sn=099a0e58cfb6a49a444a6afa4268438b&scene=21#wechat_redirect)｜[平安人寿](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526035&idx=1&sn=ce723ff107a98a6d8887d45455c9e4c0&chksm=cf2f6894f858e182ab243f4e96ce9d675831802272f5e40915bf9c7edd8d36b0b022d9ec3ca1&scene=21&cur_album_id=2524165801138995201#wechat_redirect)[｜](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526035&idx=1&sn=ce723ff107a98a6d8887d45455c9e4c0&chksm=cf2f6894f858e182ab243f4e96ce9d675831802272f5e40915bf9c7edd8d36b0b022d9ec3ca1&scene=21&cur_album_id=2524165801138995201#wechat_redirect)[奇富科技](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526469&idx=1&sn=4b2d7748da1a3b601499d7d2d80748b2&chksm=cf2f6642f858ef54b7dc5b307ec4eaba7d867ee5ab5862b9e60274cc56940787686da30873fa&scene=21#wechat_redirect)｜[上海证券](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537933&idx=1&sn=18d6800c56fdda9df28bcb17c82ff50b&scene=21#wechat_redirect) | [同程数科](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247521341&idx=1&sn=3e2b5ee81ebe6ba8b2a238e91ea7f7b7&chksm=cf2f9a3af858132c380332c123813f1df24169e30040363428db78cc0fef541f6ee802f2262d&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[通联支付](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532843&idx=1&sn=ec7c674e29a66dcb4a78ad24e9507bc2&chksm=cf2f4f2cf858c63a7953dbbe8893c2335bee15b0abe3d97d45725cfe80d9b35aa465f43d6264&scene=21#wechat_redirect)｜[无锡锡商银行](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529958&idx=1&sn=e720bfefc8cc63148c3aa6ceeee69be6&chksm=cf2f7be1f858f2f75dfab49839eafeffeddb93c49851fc8ebdcbcfd788230f6d06b1be6ad9fb&scene=21#wechat_redirect)｜[星云零售信贷](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522161&idx=1&sn=40862998ddead3c398c7765e1e8b57d8&chksm=cf2f9976f8581060f9cbf6e402f3a39d35e86a559f7d59c80b680d41977c6165e0f42cce8422&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[星火保](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532661&idx=1&sn=adba43453acddfdcf2f294f2e46727fc&chksm=cf2f4e72f858c76437d3dc998b712deb7841fb2d65c45e017aeb4f9e61e74e4ad710a5cd0df1&scene=21#wechat_redirect) | [宇信科技](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537885&idx=1&sn=692aa93fd32b98a0c226b794eaaee4de&scene=21#wechat_redirect)｜[银联商务](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526954&idx=1&sn=4dae0891ccbaae99146b5a6ba783f392&chksm=cf2f642df858ed3b939be0b04d264ddb0529556fe15ad70c3501af44d0026972cd1bef49a7b7&scene=21#wechat_redirect)｜[易生支付](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532494&idx=1&sn=aec3c408890ce0b6fd802a7f7d7e2bd1&chksm=cf2f71c9f858f8dfff7ae63f7bd3b4b91bc2717f83824d3245afb8ebbc44b1be96cbe9403354&scene=21#wechat_redirect)｜[招商信诺人寿](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524176&idx=1&sn=19138ac93dd44f395d68745cf94d9a6b&chksm=cf2f9157f858184176169417464dce040cd89fab6b35099d8b76086342f9682c49fd8cc87582&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[招联金融](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533296&idx=1&sn=b94ffa477e3398afe94e38b66479bf57&chksm=cf2f4cf7f858c5e10f996f0634f967de6478eaa573a45f102502602e16e142397935d58205d5&scene=21#wechat_redirect)｜[中信银行信用卡中心](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535858&idx=1&sn=8edeb10d97936a88b52543c2ac108288&scene=21#wechat_redirect)｜[360 数科](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247509531&idx=1&sn=60f3b5160acf1f2e6df2f30242dc4306&chksm=cf2fa81cf858210abd2b7158c19b39dc1431cc9114b1f463d25586dafe8c60673992deda574a&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[360 企业安全浏览器](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247527814&idx=1&sn=f4b56af0fac8b465b40819cd080f7563&chksm=cf2f6381f858ea97d8121642b03ae99fe48cd8c5a514c18e571be235405a90397d25f243b624&scene=21#wechat_redirect)

**互联网与文娱**：[菜鸟](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537342&idx=1&sn=118685b9bfe9c0442e0bb1558718431f&scene=21#wechat_redirect)｜[抖音集团](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247530668&idx=1&sn=3ecfbb295fc4e3fd7617044ac3575543&chksm=cf2f76abf858ffbd0718c2102777dfbb914f07deb05bf6e07a663107f1d0f1897257ba02a131&scene=21#wechat_redirect)｜[斗鱼](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247517967&idx=1&sn=0896767aa1b7d1f0b314c22023e4de3c&chksm=cf2f8908f858001e935f7f0c1c8082847c31851dc27377652b57a8c6b3c4883890e0448eff27&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[叮咚买菜](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247518311&idx=1&sn=d18e9d2be16c26833d4d3e03f5c2836b&chksm=cf2f8660f8580f7632c6211ede6d56a4a79531c95981c72dccc76948dd9f7afdb2c419db10db&scene=21&cur_album_id=2524165801138995201#wechat_redirect)|[浩瀚深度](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537072&idx=1&sn=6b5f4fae6224b2535badfd29ba3c8ecb&scene=21#wechat_redirect)｜[京东](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536067&idx=1&sn=6325bd6379241c56925cdfd0b748212e&scene=21#wechat_redirect)｜[工商信息查询平台](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526281&idx=1&sn=a586ef0e7b8d1cc631cc08da63331dbc&chksm=cf2f698ef858e098407b107df2511260590c09f2629071fcc40f24632c92582304230cb7bf7f&scene=21#wechat_redirect)｜[货拉拉](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247496431&idx=1&sn=65b8ff94a0c2de9e60b42ebfe454a4dd&scene=21#wechat_redirect)｜[快手](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533822&idx=1&sn=36a3db48f30db59f05ff4313d205d146&chksm=cf2f4af9f858c3ef54b109d4fc465d6883a541b8b18adc23b0dcb15ca5e876201af2498cc811&scene=21#wechat_redirect)｜[荔枝微课](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247519304&idx=1&sn=6c59fae3838e1f02a29eebca1b5799e9&chksm=cf2f824ff8580b593720ebed5234490c9b4a00fa616218b2ee3f8e89beb95eab11c720022e91&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[票务平台](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247521422&idx=1&sn=e42a6e93170f14bd17a48812f1e8ece4&chksm=cf2f9a89f858139f1ffa47678d3c4d068f44788ac18f288b1e31243bad0bfff1745b5763794c&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[墨迹天气](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247530525&idx=1&sn=33ac4ce5192c12a8b19b6b18a344f718&chksm=cf2f761af858ff0c0c69f833e501ca52e0a5818a17e0fcc8c1d7f86d6b8d364b83b5ad6d2244&scene=21#wechat_redirect)｜[MiniMax](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536310&idx=1&sn=8c2d8980a4c7b3f88421f1ed5f1d0eba&scene=21#wechat_redirect)｜[奇安信](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524942&idx=1&sn=331f494035518da1875fca0fc9c5437a&chksm=cf2f6c49f858e55fd2ad0aa4a2f958b9b1b0f790441a519471debe217c38b6a5bde06ae46e87&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[趣丸科技](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247530649&idx=1&sn=da6e4444e1e50b54ba640cc40385b33c&chksm=cf2f769ef858ff88bc36e0ef5e894b3898348123db2cc3d9d4d55b7b56e17e65402f09bc32a0&scene=21#wechat_redirect)｜[顺丰科技](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536639&idx=1&sn=2f0cb56207b97a63aa5857dffac9ea83&scene=21#wechat_redirect)｜[腾讯音乐](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533968&idx=1&sn=bd1f32f608c3ce648c12f5cf3365db8c&chksm=cf2f4b97f858c2811330a4247d40bfd404ea031cc0c4f2c7d5ae79309b204c13afe8d81ff1a4&scene=21#wechat_redirect)｜[天眼查](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247519778&idx=1&sn=e32c89396fc87a09965cc89b5e9e1b4b&chksm=cf2f8025f858093392bffd5619ba53be539a865128bcedd6e6094cf6e49082282cd7c3bc5298&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[网易](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529891&idx=1&sn=fbdc2154807b766bd4534522dc48295d&chksm=cf2f7ba4f858f2b26996ef3fc3564da96a632a81d4ae01500a77e58ef925cbdb14d6e22efdad&scene=21#wechat_redirect)｜[网易游戏](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247531491&idx=1&sn=bbff8938a6d08b6ead0a08e203709f01&chksm=cf2f75e4f858fcf2a6e4023e93aec4fd165af096388a6fa61906fa640b82ff4e60cd3d8a2641&scene=21#wechat_redirect)｜[网易严选](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247503369&idx=1&sn=29fab18c22f5778b0bb19409cb68f567&chksm=cf2fc00ef8584918fc809d661066f139d002f0202f2f6477a02286527409296d6212a373e58a&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[网易云信](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536880&idx=1&sn=710229bd13e19f06216eb7211220a0ce&scene=21#wechat_redirect)｜[网易云音乐](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535994&idx=1&sn=88d69524871d7b34191d2f2d778e78e6&scene=21#wechat_redirect)｜[小米](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537383&idx=1&sn=47a4bc984bf6097a74b343e77ec4a86b&scene=21#wechat_redirect)｜[小鹅通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524553&idx=1&sn=63af19e32467049aa3b2fff79898c5ce&chksm=cf2f6ecef858e7d8dc3939f71ebef3bf17962e50889707b8393080a7e2ffca7520c814232bd7&scene=21&cur_album_id=2524165801138995201&poc_token=HCXDWmWjWHZ5_WrVJMWvScoFjGBJjOj_ZV5DcLPW#wechat_redirect)｜[迅雷](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247531772&idx=1&sn=3db2e0c4f8dec01085bd6e24d2c857b3&chksm=cf2f72fbf858fbed95f9aa74e103b76c828dd5433a0338e97354db4781974e9444a957e9adde&scene=21#wechat_redirect)｜[约苗](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247521201&idx=1&sn=dc8f3d55343cd054974e8c5f4bfe50a2&chksm=cf2f9db6f85814a055a245c88c5c5c138fb7b9a9bbacedfa39dae563726ae43ad237993a5872&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[字节跳动](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247516005&idx=1&sn=84d069e950cbb178a45cd569b244d6a4&chksm=cf2fb162f8583874b83a2a591a92e75cc689f56f4ff2324ccbb583494daf76d9a96e3d6feb42&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[知乎](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247520797&idx=1&sn=1fd8394aafaddbd0ab0fbfc322e5fb25&chksm=cf2f9c1af858150c4ef3e2d9b5762477584d3e0908d71ab8871b81061253343e689c6aa18ac0&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[360 商业化](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247518769&idx=1&sn=60b5ac3a78930baddc845f1e6c66a29c&chksm=cf2f8436f8580d201bacec5b3434ee62b2fea52718c4b82355c8e8c698fc2d4228a2b0cc0bef&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)

**企业服务与新经济**：[宝尊科技](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536183&idx=1&sn=d4ddeaf5a8ebbb284b5207c62ea50c40&scene=21#wechat_redirect)| [波司登](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537544&idx=1&sn=049172879eeb986d6c67a0a2472bbdd2&scene=21#wechat_redirect)｜[Cisco](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535303&idx=1&sn=3a57e0de6153d6715e5d19f1f3ff0717&scene=21#wechat_redirect)｜[橙联](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247507345&idx=1&sn=4dc83cae32333c2aa2355c9447ecfb81&chksm=cf2fd396f8585a8018f753f50d89f6bbbb607c86350bf49957bbc07894477b88a61f3eba9342&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[度言](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247512880&idx=1&sn=9b3adbddb44b7287835adf560a56e65b&chksm=cf2fbd37f8583421316ef89513c84654956b8d9e2231868613ce4c9ad439127bb5a8a4077b73&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[观测云](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526244&idx=1&sn=af3b3bc7a5d5419d003ada6b9ea9c376&chksm=cf2f6963f858e0758e623e16612bceae95457c6d7aeed918be6d0d5bd53e8962d14549b64de0&scene=21#wechat_redirect)｜[慧策](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247515310&idx=1&sn=abfa22039b546171dd6cb1ead692de0f&chksm=cf2fb2a9f8583bbff034fc5824f89bf6a1d960a1f7997f3c3fc42af27b271c76d28aecf4b522&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[快成物流](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247531733&idx=1&sn=53e4df3ca765685dcc2b429db833a64d&chksm=cf2f72d2f858fbc4e5ccc99ea75262b6903467c9c3ea3e66919a587545fe1ea01cd7b2fd3b62&scene=21#wechat_redirect)｜[领健](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247512380&idx=1&sn=e7bb6b39a71b2a350acd7ac8d2f3b202&chksm=cf2fbf3bf858362d4e4326712e9d360493c2660e5d619e0c7ec23383b560bea2f0644b10c84f&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[领创](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247495090&idx=1&sn=314d0a12cab7985f822876ba99e29e39&chksm=cf2fe3b5f8586aa36938cd8493f98caba5fb821ca897ecf06412e6f90cb1c3deed02c4e7ed89&token=2145458351&lang=zh_CN&scene=21#wechat_redirect)｜[灵犀科技](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535808&idx=1&sn=142386b7a74856d424676f79b281d07a&scene=21#wechat_redirect)｜[名创优品](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529981&idx=1&sn=85ba7efaa7c947573d59d87ae4897cc4&chksm=cf2f7bfaf858f2ec757fe383ec0bd4053bdfb09c09197fa1587c36d8c3947394d6621116341a&scene=21#wechat_redirect)｜[Moka BI](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247518538&idx=1&sn=d9a39d040c9f430a841a2045a2bafb90&chksm=cf2f874df8580e5b0d77c6235aadd8610c80b4afd1c2781172ba6e1fb734dbb6e463d6f8794e&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[美联物业](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247517792&idx=1&sn=455c085a102c7af1fcac44e62bc06ca1&chksm=cf2f8867f8580171c05af38785d7c7a32766308df46f482f7731cdc0d3dc76d9ed0a69b864f2&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[钱大妈](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529314&idx=1&sn=e1564a7bc1184ba73c353c465eae503d&chksm=cf2f7d65f858f473f71cad992eb80cdf095cb873fe2394c6a2deab07ecbd12aae6eb5de72a60&scene=21#wechat_redirect)｜[拈花云科](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522076&idx=1&sn=c2b2b0f72e9bc25b9020238048a3c6ae&chksm=cf2f991bf858100dfdacf18748ef75a4e76e6cbf34e8b1e52ffc55232e6f6db0ae6c4d9b1b69&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[森马](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536942&idx=1&sn=aeb684eaa379dbb4d910d9fefc678ccd&scene=21#wechat_redirect) |[思必驰](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247506825&idx=1&sn=b6d002ddef220acff8963b0af04e3676&chksm=cf2fd58ef8585c9829f474fe4f75a8b6fa9e2a923b474043d0f92582318197fd737933d4cc59&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[顺丰科技](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536639&idx=1&sn=2f0cb56207b97a63aa5857dffac9ea83&scene=21#wechat_redirect)｜[上海家化](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529885&idx=1&sn=b7c4ae401c29dfa64d10097a2ae3c678&chksm=cf2f7b9af858f28c309136b6a68ede29631849dceba2b84d180b07cd04d02937c940c74975ae&scene=21#wechat_redirect) | [物易云通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247490382&idx=1&sn=531f8d388526ee57aff6e86c8befe66c&scene=21#wechat_redirect)｜[云积互动](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247508739&idx=1&sn=52d028e91f9fa9cf4e93036970711eed&chksm=cf2fad04f8582412ec4cfd3c7a4a672326652c0062a481eda98a704ec245a878f5246ad236bc&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[有赞](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524089&idx=1&sn=95b479a8648809456100e121677fb29d&chksm=cf2f90fef85819e8e73021e841d396ad1cc44e002bff0002c1b431741b3790a775415b829bf8&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[雨润集团](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532231&idx=1&sn=ff3913472ee157907f40e174ddf370bd&chksm=cf2f70c0f858f9d69c4f195e01df934a28149797aacaffb8382d0883c7fb07dcf5a861aa239c&scene=21#wechat_redirect)｜[纵腾集团](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247513813&idx=1&sn=348def05d3f82fc3e7b119a78b3c2b84&chksm=cf2fb8d2f85831c45349ea5a3e4b6c27503aa41f783903a856b915a6556b3d4c81691fb24a7c&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[中通快递](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533447&idx=1&sn=68dcc5af518f418d77536c711c3a4639&scene=21#wechat_redirect)

**先进智造与电信**：[爱玛](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529522&idx=1&sn=95d61e4fea493b667612a946ed126b2d&chksm=cf2f7a35f858f323918bfa574f60d3c8af817c60c6e1680f0b639533528fce3183e60ac90e95&scene=21#wechat_redirect)｜[长安汽车](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526078&idx=1&sn=55ac5982a5a81eb98d380cdca2fe4a1f&chksm=cf2f68b9f858e1af4f53a4eeb489516d2508f513435f9c82b50896acce8cbe522f165034ce9f&scene=21&cur_album_id=2524165801138995201#wechat_redirect)[｜](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526078&idx=1&sn=55ac5982a5a81eb98d380cdca2fe4a1f&chksm=cf2f68b9f858e1af4f53a4eeb489516d2508f513435f9c82b50896acce8cbe522f165034ce9f&scene=21&cur_album_id=2524165801138995201#wechat_redirect)[极越汽车](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247530634&idx=1&sn=b02e765afeefbe7eb709260ec5127031&chksm=cf2f768df858ff9b78f0e61907bf86ef319c7d03e6eddc116c0da70713bb853e3301da160148&scene=21#wechat_redirect)｜[金风科技](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533704&idx=1&sn=f1c619e15f318e58ad70befd0e70fc91&scene=21#wechat_redirect)｜[科大讯飞](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247534327&idx=1&sn=60c35720f3f412db6a78079815acc592&chksm=cf2f48f0f858c1e6f7ca4b881ed20e60c0aba8554972514674e291e26face6586fc96413e2e9&scene=21#wechat_redirect)｜[岚图汽车](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537812&idx=1&sn=dba5ebde479749f8ba74409e6a93c2e4&scene=21#wechat_redirect)｜[Lifewit](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247487975&idx=1&sn=0cfd5f9d748cb982e1ff5abc35fddb9a&chksm=cf2c1fe0f85b96f652975a5f88a9ca85d6ef75e55427dda02126d34f06d2e2a3ac15bda56413&token=2145458351&lang=zh_CN&scene=21#wechat_redirect)｜[哪吒科技](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247531497&idx=1&sn=473ea00ef53c0bd902968446e8b0e6e4&chksm=cf2f75eef858fcf850ab57e1c28d5fbfc81aa369c865c1153105b9d8e4a61d910a739634686d&scene=21#wechat_redirect)｜[四川航空](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536110&idx=1&sn=86a18fe553ddc620be30173c663101e4&scene=21#wechat_redirect)｜[上海通用五菱](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247530723&idx=1&sn=47c5c9d104566b638cb09e9d0ab3f448&chksm=cf2f76e4f858fff26b1aac2bb8d6f2c718ac8025a33aeed62bb319a43863f8a50eb2b79a21d7&scene=21#wechat_redirect)｜[三星电子](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533303&idx=1&sn=5b8a674e5ab8fbc2a9d2d462e1527a32&chksm=cf2f4cf0f858c5e689ecebf5c2f2a60681926946038f3be3ffddc11bfb29b256c158870b1ee5&scene=21#wechat_redirect)｜[蜀海供应链](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247496660&idx=1&sn=1c0738a95564fc4cebb8a56cb539f703&scene=21#wechat_redirect)｜[特步](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535841&idx=1&sn=7a0a2851555bdd5f340bb3aa640fc61c&scene=21#wechat_redirect)｜[天翼云](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536234&idx=1&sn=957e6d589138555c5c7c94e62a793126&scene=21#wechat_redirect)｜[雅迪](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532027&idx=1&sn=d4de3a3326c1450b063d017aaaa78506&chksm=cf2f73fcf858faea620ba48c0b78b3746485a74fd5f1d29482b0dbed3cbd6d374ecc17cd2453&scene=21#wechat_redirect)｜[中国联通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247523168&idx=1&sn=f6a8195d485b56438a4413ca05c9a5e7&chksm=cf2f9567f8581c71f476c52f89de7e4e6769d17e232c8930e257d2db71a45b9ab83a50382655&scene=21&cur_album_id=2524165801138995201#wechat_redirect)

作为基于 Apache Doris 的商业化公司，飞轮科技秉承着 “开源技术创新”和“实时数仓服务”双轮驱动的战略，在投入资源大力参与 Apache Doris 社区研发和推广的同时，基于 Apache Doris 内核打造了聚焦于企业大数据实时分析需求的企业级产品 SelectDB ，面向新一代需求打造世界领先的实时分析能力。自 2022 年成立以来，获得 IDG 资本、红杉中国、襄禾资本等顶级 VC 的近 10 亿元融资，创下了近年来开源基础软件领域的新纪录。

#### 

#### ▼ 点击阅读原文，下载使用 Apache Doris！