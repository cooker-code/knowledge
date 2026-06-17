---
title: Flink + Paimon 数据 CDC 入湖最佳实践
author: Apache Paimon
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkyNjQ1MDI3Mg==&mid=2247484168&idx=1&sn=2d08ba19507cea8e1ee1bc8c4393e419&chksm=c2365407f541dd11279e279794ae4287ccdd74c4a35403c21d232855227fcf3bc60f674fae50&mpshare=1&scene=24&srcid=1124SQkoPOGgFBDmEddhRwbM&sharer_shareinfo=a5b8d7ea2a94caf445e15c569428fcdb&sharer_shareinfo_first=a5b8d7ea2a94caf445e15c569428fcdb#rd
---

**前言**

Apache Paimon 最典型的场景是解决了 CDC (Change Data Capture) 数据的入湖，看完这篇文章，你可以了解到：

1. 为什么从 CDC 入 Hive 迁移到 Paimon？
2. CDC 入 Paimon 怎么样做到成本最低？
3. Paimon 对比 Hudi 有什么性能优势？

Paimon 从 CDC 入湖场景出发，希望提供给你 简单、低成本、低延时 的一键入湖。本文基于 Paimon 0.6，0.6 正在发布中，可提前在此处下载：https://paimon.apache.org/docs/master/project/download/

**CDC 入 Hive**

CDC 数据来自数据库。一般来说，分析需求是不会直接查询数据库的。

1. 容易对业务造成影响，一般分析需求会查询全表，这可能导致数据库负载过高，影响业务
2. 分析性能不太好，业务数据库一般不是列存，查询部分列 Projection 性能太差
3. 没有 Immutable 的视图，离线数仓里面需要根据 Immutable 的一个分区来计算

所以需要通过 CDC 的方式同步数据库的数据到数据仓库或数据湖里。目前典型的同步方式依然是 Hive 的全量与增量的离线合并同步方式。

在 Hive 数仓里维护两张表：增量分区表和全量分区表，通过：

1. (按需) 初始化时使用 DataX 或 Sqoop 等工具同步整张数据库表到 Hive 全量表的分区中。
2. 每天定时 (比如凌晨0点30分) 同步增量数据 (通过 Kafka) 到 Hive 增量分区表，形成一个增量分区 T。
3. 将 增量分区 T 与 全量分区 T-1 进行合并，产出今天的 全量表 分区 T。

这个流程在今天也是主流的同步方式，离线数据提供一个 Immutable 的视图，让数据的可靠性大大增加。

但是它的问题不少：

1. 架构链路复杂度高：由于链路复杂，每天产出全量分区容易有问题导致不能按时产出，新增业务也比较复杂，全量和增量割裂。
2. 时延高：至少 T + 1 延时，而且需要等全量和增量合并完成。
3. 存储成本高：每天全量表一个分区存储所有数据，意味着 100 天就需要 100 倍的存储成本。
4. 计算成本高：每天需要读取全量数据，与增量数据进行全量合并，在增量数据不多时浪费严重。

是时候该做一些改变了。

**CDC 入 Paimon**

Apache Paimon (incubating) 是一项流式数据湖存储技术，可以为用户提供高吞吐、低延迟的数据摄入、流式订阅以及实时查询能力。Paimon 采用开放的数据格式和技术理念，可以与 ApacheFlink / Spark / Trino 等诸多业界主流计算引擎进行对接，共同推进 Streaming Lakehouse 架构的普及和发展。

和其它数据湖不同的是，Paimon 是从流世界里面诞生的数据湖，所以它在对接流写流读、对接 Flink 方面都要比其它数据湖做得更好，详见后续的功能和性能对比。

Flink 结合 Paimon 打造的入湖架构如下：

步骤如下：

1. 通过 Flink CDC 一键全增量一体入湖到 Paimon，此任务可以配置 Tag 的自动创建，然后通过 Paimon 的能力，将 Tag 映射为 Hive 的分区，完全兼容原有 Hive SQL 的用法。

完，只需一步。

流式入湖方式可以有如下多种方式：

1. Flink SQL 入湖，SQL 处理，可以有函数等 Streaming SQL 的处理
2. Paimon 一键 Schema Evolution 入湖，好处是 Schema 也会同步到下游 Paimon 表里：详见 https://paimon.apache.org/docs/master/cdc-ingestion/overview/

它的好处是：

1. 架构链路复杂度低，不再因为各种组件的问题导致链路延时，你只用运维这一个流作业，而且可以完全兼容原有 Hive SQL 用法。
2. 时延低：延时取决于流作业的 Checkpoint Interval，数据最低1分钟实时可见 (建议1-5分钟)。不但如此，Paimon 也提供了流读的能力，让你完成分钟级的 Streaming 计算，也可以写到下游别的存储。
3. 存储成本低：得益于湖格式的 Snapshot 管理，加上 LSM 的文件复用，比如同样是存储 100天的快照，原有 Hive 数仓 100 天需要 100 份的存储，Paimon 在某些增量数据不多的场景只需要 2 份的存储，大幅节省存储资源。
4. 计算成本低：得益于 LSM 的增量合并能力，此条链路只有增量数据的处理，没有全量的合并。可能有用户会担心，常驻的流作业会消耗更多的资源，对 Paimon 来说，你可以打开纯异步 Compaction 的机制，以 Paimon 优异的性能表现，只用少量的资源即可完成同步，Paimon 另有整库同步等能力帮助你节省资源。

**Tag 与 Hive 兼容**

什么是 Tag？Paimon 的每一次写都会生成一个 Immutable 的快照，快照可以被 Time Travel 的读取。但在大多数情况下，作业会生成过多的快照，所以根据表配置，快照会在合适的时间点被过期。快照过期还会删除旧的数据文件，过期快照的历史数据将无法再查询。

要解决此问题，可以基于快照创建 Tag。Tag 将维护快照的清单和数据文件。典型的用法是每天创建 Tag，然后您可以维护每天的历史数据以进行批式查询。

Tag 是 immuatable 的，它不能被增删改查的，一般来说，数据库映射的表是不可变的，我们推荐在 ODS 层使用 Tag 来替代 Hive 的分区，但是后续的 DWD 和 DWS 不建议。

Paimon 提供了 Tag 的自动创建：

此 DDL 会让 Flink 流写作业时，自动周期的创建 Tag，此配置表明每天0点10分钟创建一个 Tag，最大保留3个月的 Tag，Flink 流式写入，自动创建 Tags，自动清理 Tags。

有了 Tag 后，你需要在 Flink SQL 或者 Spark SQL 里使用 Time Travel 来查询 Tags，这给业务带来了一个问题，老的 Hive SQL 如何兼容？老的 Hive 可是一个全量分区表，而 Paimon 表是一个非分区主键表，Hive 数据仓库的传统使用更习惯于使用分区来指定查询的 Tag。

因此，我们引入了 'metastore.tag-to-partition' 和 'metastore.tag-to-partition.preview' (配置此参数可以让 Hive SQL 查询到未 Tag 的分区，比如当前最新数据) 来将未分区的主键表映射到 Hive metastore 中的分区表，并映射分区字段为 Tag 查询。

使用此功能，可以让业务使用方完全不感知 Paimon 主键表的玩法，完全兼容老 Hive SQL 的用法，做到无感知的升级！(如果你使用 Spark 或者 Flink 来查询，需要使用 Time Travel 的语法)

我们再来看看成本的降低。

**存储成本大幅节省**

什么是 LSM 的文件复用，为何能大幅节省存储成本？

LSM 结构如下：

LSM 典型的 Minor Compaction 是指：增量数据只会让前面几层的文件进行合并，只要增量数据不够多，最底层的文件是不会参与 Compaction 的，这就意味着多个 Tag 之间的最底层是完全一样，完全复用的，结合湖格式的文件管理，多个 Tag 并不会带来冗余的文件存储。

针对增量数据不多的情况，最底层的文件，也是最大的数据量的文件，是可以被多个 Tag 复用的，你不用做任何事情，Paimon 的 Snapshot 管理会自动完成文件的复用。

**计算成本**

接下来，让我们看看计算的成本，如何打造低成本的计算。也许你担心，将架构从离线合并切换到实时，作业常驻24小时运行，资源上会有上升。

Paimon 默认情况下会在写入后台线程自动运行 Compaction，当新增 CDC 数据太多，Paimon 可能会反压 Writer，等待 Compaction 完成，这是因为 Paimon 在默认情况下希望提供一个写放大和读放大适中的环境，保证你的实时读取性能。

当面向计算成本优先时，你可以考虑开启全异步 Compaction，解放写入资源：

此表配置将在写入的峰值期间生成更多文件，并在写入的低谷期间逐渐合并为最佳读取性能。

另外，Paimon 也提供了其它丰富的方式来让你的写入应用在各种场景：比如独立的 Compaction 作业，帮助你分离 Compaction 的资源且支持多作业同时写入一张表中；比如整库 Compaction，作业资源取决于你的配置：资源多，Compaction 快；资源少，慢慢 Compaction。

最小化 Compaction 后，你完全可以使用 Paimon 的整库同步，单个作业同步上千个表，较小的资源使用。

**最佳实践**

此节提供 CDC 入湖的一个参考配置，最好理解每个配置的作用，根据自己的业务按需选择。

小表整库同步

参数说明：

1. 使用 Mysql 整库同步
2. 使用 combined 模式，Paimon 只会用一个 Sink 同步所有的表
3. 配置 Mysql 参数
4. 配置 Hive metastore 参数
5. 排除大表：excluding-tables
6. 表参数 changelog-producer = input，如若下游不流读，没必要配置此参数
7. 配置整库同步作业并发为 8，可根据你资源情况配置
8. 配置 Tag 自动创建
9. 配置 Tag 映射为 Hive 分区，如果不使用 Hive SQL，请不要配置
10. 配置全异步 Compaction，如若表很小，可配置 num-sorted-run.compaction-trigger 为 3，减少小文件。如果资源足够，请不用配置。

单表同步

参数说明：

1. 大表适合单独作业来写入，可以用 Paimon CDC 来进行 Schema Evolution 的同步，也可以用 Flink SQL 写入。
2. 通过 Catalog 配置 Hive metastore。
3. 大表推荐使用动态 Bucket 模式：bukcet = -1，自动调整 bucket 个数，根据你对查询速度的要求，可以定义你期望单个 bucket 内包含多少条数据。
4. 同样配置好 Tag 自动创建。
5. 同样配置 Tag 映射为 Hive 分区，如果不使用 Hive SQL，请不要配置。
6. 同样配置全异步 Compaction，如果资源足够，请不用配置。

写入性能

下图也提供一个流程图来说明 Paimon 对于多个方面的权衡的性能调优：

更多参数可以看看官网，官网提供了详细的调优及参数说明。

**性能对比**

入湖更新的资源消耗非常重要，否者计算成本大幅增加得不偿失，而当前降本也是企业的核心需求之一。

这一节将评估 Paimon 与 Hudi 的 Flink 写入性能，相关测试环境在阿里云的 EMR 5.14.0 集群上，组件及版本如下：Paimon: 0.6、Hudi: 0.13.1、Flink: 1.15，文件系统使用 OSS。

本节使用 https://github.com/apache/incubator-paimon/tree/master/paimon-benchmark/paimon-cluster-benchmark 此测试，统计写入5亿条数据的总耗时，非常简单的随机数据入湖性能测试。

Flink 集群配置：

```
parallelism.default: 16jobmanager.memory.process.size: 4gtaskmanager.numberOfTaskSlots: 1taskmanager.memory.process.size: 8gexecution.checkpointing.interval: 2minexecution.checkpointing.max-concurrent-checkpoints: 3taskmanager.memory.managed.size: 1mstate.backend: rocksdbstate.backend.incremental: truetable.exec.sink.upsert-materialize: NONE
```

### 

### 测试1：MOR

Paimon 表配置：

```
'bucket' = '16','file.format' = 'parquet','file.compression' = 'snappy'
```

使用 Parquet 与 Hudi 对齐，Paimon 默认 ORC 会性能稍高一些。

Hudi 表配置：

```
'table.type' = 'MERGE_ON_READ','index.type' = 'BUCKET','hoodie.bucket.index.num.buckets' = '16','write.tasks' = '16','hoodie.parquet.compression.codec' = 'snappy','read.tasks' = '16','compaction.async.enabled' = 'true','compaction.tasks' = '16','compaction.delta_commits' = '2''compaction.max_memory' = '4096'
```

由于测试所需的总耗时不多（checkpoint 个数也相应较少），配置 compaction.delta\_commits 为 2来保证在写入期间有 compaction 执行。

测试结果：

我们也测试了查询性能 (Merge On Read)，发现 Hudi 的查询性能非常差，所以分析了 Hudi 表的文件状态，发现大部分 Log 都没有被合并，分析原因是：

* Hudi MOR 的 Compaction 完全异步，导致太多数据没有合并，读取性能极差。
* Paimon 默认会在写入和读取性能取一个平衡，Compaction太慢会等待其完成。

此 MOR 场景不能测试到 Compaction 的性能，所以下面也测试了 COW 表，来测试对比 Compaction 的性能。

### 测试2：COW

Paimon 表配置：

```
'bucket' = '16','file.format' = 'parquet','file.compression' = 'snappy','full-compaction.delta-commits' = '1'
```

使用 'full-compaction.delta-commits'，配置每个 Checkpoint 都完成全量合并，达到 COW 的效果。(生产状态不建议全部作业使用，资源消耗较大)

Hudi 表配置：

```
'table.type' = 'COPY_ON_WRITE','index.type' = 'BUCKET','hoodie.bucket.index.num.buckets' = '16','write.tasks' = '16','hoodie.parquet.compression.codec' = 'snappy','read.tasks' = '16','compaction.max_memory' = '4096'
```

只测试1亿数据入湖，因为 COW 吞吐较差，耗时太久。

测试结果：

### 

### 测试结论

* **数据 MOR Flink 写入性能，Paimon 是 Hudi 的 4 倍，Hudi 遗留大量未合并数据导致读取性能很差。**
* **数据 COW Flink 写入性能，Paimon 是 Hudi 的 10 倍以上，Paimon 的合并性能大幅领先 Hudi。**
* **如果已有 Hudi 作业，替换成 Paimon 建议只用 1/3 的资源。**

以上测试环境在阿里云 EMR，可以参考 paimon-cluster-benchmark 里面步骤在你的集群复现测试。

**关于 Paimon**

1. 微信公众号：Apache Paimon ，了解行业实践与最新动态
2. 官网：https://paimon.apache.org/ 查询文档和关注项目