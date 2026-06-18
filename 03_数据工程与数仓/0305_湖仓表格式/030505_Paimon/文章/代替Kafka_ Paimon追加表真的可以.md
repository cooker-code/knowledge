---
title: 代替Kafka? Paimon追加表真的可以
author: 大数据技能圈
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490431&idx=1&sn=d54ddce6542da78253911ba352e84516&chksm=c1b339b8571cfafa7b6b894c9906079a628570b96ef28db01a88611a1f1fa91af656c9cdf273&mpshare=1&scene=24&srcid=0117R8wDay52MjFobPqzQuq8&sharer_shareinfo=c7e3e4d77f246c19539fb3ff58da8228&sharer_shareinfo_first=c7e3e4d77f246c19539fb3ff58da8228#rd
---

Paimon中如果一张表没有定义主键，那么它是一张追加表（append table）。与具有主键的表相比，它不具备直接接收变更日志的能力。它不能通过更新插入（upsert）的方式直接更新数据。它只能接收以追加方式 incoming 的数据。

创建语句如下：

```
CREATE TABLE my_table (    product_id BIGINT,    price DOUBLE,    sales BIGINT) WITH (    -- 'target-file-size' = '256 MB',    -- 'file.format' = 'parquet',    -- 'file.compression' = 'zstd',    -- 'file.compression.zstd-level' = '3');
```

01

**Paimon非主键表的应用场景**

paimon非主键表典型的应用场景中是批量写入和批量读取，类似于常规的Hive分区表，但与Hive表相比，它能够带来以下优势：

* **对象存储友好（S3, OSS）**：更适用于对象存储服务，如亚马逊的S3或阿里云的对象存储服务（OSS），优化了数据读写的效率和成本。
* **时间旅行和回滚（Time Travel and Rollback）**：支持数据的时间点查询和版本回滚，允许用户访问历史数据状态或撤销不想要的数据变更。
* **流式接收端的小文件自动合并**：当作为流处理的接收端时，能够自动合并小文件，从而减少大量小文件对存储系统性能的影响。
* **队列式的流式读写**：支持类似队列的流式读写操作，使得数据可以像消息队列一样被连续地消费和生产，非常适合实时数据处理场景。
* **有序和索引支持的高性能查询**：通过保持数据的有序性并利用索引结构，能够显著提高查询性能，特别是对于大规模数据分析和复杂查询任务来说非常重要。

这些特性共同使得这种表格类型特别适合用于现代大数据处理和分析环境，尤其是那些需要频繁进行数据更新、要求高查询性能以及对存储成本敏感的应用场景。

> **这或许是一个对你有用的开源项目**，**data-warehouse-learning**项目是一套基于 MySQL + Kafka + Hadoop + Hive + Dolphinscheduler + Doris + Seatunnel + Paimon + Hudi + Iceberg + Flink + Dinky + DataRT + SuperSet 实现的实时离线数仓（数据湖）系统，以大家最熟悉的电商业务为切入点，详细讲述并实现了数据产生、同步、数据建模、数仓（数据湖）建设、数据服务、BI报表展示等数据全链路处理流程。
>
> **https://gitee.com/wzylzjtn/data-warehouse-learning**
>
> **https://github.com/Mrkuhuo/data-warehouse-learning**
>
> **https://bigdatacircle.top/**
>
> 项目演示：

02

**如何替代消息队列？**

在paimon中可以通过Flink以非常灵活的方式向非主键表（追加表（Append table））进行流式写入，或者通过Flink读取追加表，将其用作队列。唯一的区别在于其延迟是以分钟为单位的。它的优点是成本非常低，并且能够下推过滤和投影操作。

01

自动小文件合并

在流式写入作业中，如果没有定义桶（bucket），则写入过程中不会进行文件合并（compaction），而是使用合并协调器（Compact Coordinator）来扫描小文件，并将合并任务发送给合并工作器（Compact Worker）。在流模式下，如果你在Flink中运行的是插入SQL，其拓扑结构将会是这样的：

压缩过程不必担心反压（backpressure），因为合并操作永远不会产生反压。

如果将写入模式设置为仅写（write-only），那么合并协调器（Compact Coordinator）和合并工作器（Compact Worker）将会从拓扑结构中移除。

自动合并功能仅在Flink引擎的流模式下得到支持。也可以通过Paimon中的Flink动作启动一个合并作业，并通过设置写入模式为仅写（write-only）来禁用所有的其他紧凑操作。

02

流式查询

可以将追加表（Append table）当作消息队列一样进行流式处理。与主键表类似，对于流式读取有两种选项：

* 默认情况下，流式读取会在首次启动时生成表的最新快照，并继续读取最新的增量记录。
* 可以通过指定 scan.mode、scan.snapshot-id、scan.timestamp-millis 或 scan.file-creation-time-millis 来仅流式读取增量数据。

类似于Kafka，顺序默认是不被保证的；如果数据有一定的顺序要求，需要考虑定义一个桶键（bucket-key）。

03

****分桶追加表（Bucketed Append）****

可以定义桶（bucket）和桶键（bucket-key）来创建一个分桶追加表（bucketed append table）。

以下是创建分桶追加表的示例：

```
CREATE TABLE my_table (    product_id BIGINT,    price DOUBLE,    sales BIGINT) WITH (    'bucket' = '8',    'bucket-key' = 'product_id');
```

01

流式查询

一个普通的追加表（Append table）对其流式写入和读取没有严格的顺序保证，但对于同一桶（bucket）中的每条记录，它们是严格按照写入顺序排列的，流式读取会按照写入的确切顺序将记录传递给下游。要使用这种模式，不需要配置特殊的设置，所有数据将会进入同一个桶中，作为队列处理。

**1.1. 桶内合并**

默认情况下，sink节点会自动执行合并操作以控制文件数量。以下选项用于控制合并策略：

|  |  |  |  |
| --- | --- | --- | --- |
| **键** | **默认值** | **类型** | **描述** |
| write-only | false | 布尔值 | 如果设置为 true，则会跳过合并操作和快照过期处理。此选项通常与专门的合并作业一起使用。 |
| compaction.min.file-num | 5 | 整数 | 对于文件集 [f\_0,...,f\_N]，触发追加表合并操作所需的最小文件数量，需满足 sum(size(f\_i)) >= targetFileSize。该值避免几乎满的文件被合并，因为这样做成本效益不高。 |
| compaction.max.file-num | 5 | 整数 | 对于文件集 [f\_0,...,f\_N]，即使 sum(size(f\_i)) < targetFileSize，触发追加表合并操作的最大文件数量。该值避免累积过多小文件，这会降低性能。 |
| full-compaction.delta-commits | (无) | 整数 | 在增量提交后，将不断触发完全合并操作。 |

**1.2. 流式读取顺序**

1. 来自两个不同分区的任意两条记录：

+ 如果设置了 scan.plan-sort-partition = true，则分区值较小的记录将首先被生成。
+ 否则，较早创建的分区中的记录将首先被生成。

2. 来自同一分区和同一桶的任意两条记录：

+ 先写入的记录将首先被生成，确保了同一桶内的记录按照写入顺序处理。

3. 来自同一分区但不同桶的任意两条记录：

+ 不同的桶由不同的任务处理，因此它们之间没有顺序保证。

**1.3. 水印设置**

可以为读取 Paimon 表定义水印：

```
CREATE TABLE t (    `user` BIGINT,    product STRING,    order_time TIMESTAMP(3),    WATERMARK FOR order_time AS order_time - INTERVAL '5' SECOND) WITH (...);  
-- launch a bounded streaming job to read paimon_tableSELECT window_start, window_end, COUNT(`user`) FROM TABLE( TUMBLE(TABLE t, DESCRIPTOR(order_time), INTERVAL '10' MINUTES)) GROUP BY window_start, window_end;
```

还可以启用 Flink 的水印对齐（Watermark Alignment）功能，这将确保没有数据源/分片/分片分区/分区的水印进度过快地领先于其他部分：

|  |  |  |  |
| --- | --- | --- | --- |
| 键 | 默认值 | 类型 | 描述 |
| scan.watermark.alignment.group | (无) | 字符串 | 一组需要对齐水印的数据源。通过指定这个参数，可以将多个数据源归为同一组，以确保它们的水印进度保持一致。 |
| scan.watermark.alignment.max-drift | (无) | 持续时间 | 在我们暂停从某个数据源/任务/分区消费之前，允许的最大水印偏差。该参数定义了在不同数据源之间水印对齐时可容忍的最大时间差。 |

**1.4. 有界流**

流式数据源也可以是有界的（Bounded），可以通过指定 scan.bounded.watermark 来定义有界流模式的开始条件。有界流读取操作会在遇到更大的水印快照时开始。

快照中的水印是由写入者生成的，例如，可以指定一个 Kafka 源并声明水印的定义。当使用这个 Kafka 源向 Paimon 表写入数据时，Paimon 表的快照会生成相应的水印，从而在对该 Paimon 表进行流式读取时可以利用有界水印的功能。

```
CREATE TABLE kafka_table (    `user` BIGINT,    product STRING,    order_time TIMESTAMP(3),    WATERMARK FOR order_time AS order_time - INTERVAL '5' SECOND) WITH ('connector' = 'kafka'...);  
-- launch a streaming insert jobINSERT INTO paimon_table SELECT * FROM kakfa_table;  
-- launch a bounded streaming job to read paimon_tableSELECT * FROM paimon_table /*+ OPTIONS('scan.bounded.watermark'='...') */;
```

04

**批量处理模式**

分桶表可以在批查询中避免不必要的洗牌（shuffle），例如，可以使用以下 Spark SQL 来读取 Paimon 表：

```
SET spark.sql.sources.v2.bucketing.enabled = true;  
CREATE TABLE FACT_TABLE (order_id INT, f1 STRING) TBLPROPERTIES ('bucket'='10', 'bucket-key' = 'order_id');  
CREATE TABLE DIM_TABLE (order_id INT, f2 STRING) TBLPROPERTIES ('bucket'='10', 'primary-key' = 'order_id');  
SELECT * FROM FACT_TABLE JOIN DIM_TABLE on t1.order_id = t4.order_id;
```

spark.sql.sources.v2.bucketing.enabled 配置项用于启用 V2 数据源的分桶功能。当启用时，Spark 将识别 V2 数据源通过 SupportsReportPartitioning 报告的具体分布情况，并在必要时尝试避免洗牌（shuffle）。

如果两个表具有相同的分桶策略和相同数量的桶，则可以避免代价高昂的连接洗牌操作。

05

**加群请添加作者**

## 推荐阅读系列文章

* [建议收藏 | Dinky系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489859&idx=1&sn=6b59becc16f653b609d01090e40f9e20&chksm=c02962dcf75eebcabfa43a1e3e4a1b210df6ac0725685a9087984261626223cc339c0c363a24&scene=21#wechat_redirect)
* [建议收藏 | Flink系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489608&idx=1&sn=c055844c7ac20eabbe11e331f0d629c2&chksm=c02963d7f75eeac15892c32660c1e90e000ad72e66ab97ed4051273bb5e3f6299998d6c81097&scene=21#wechat_redirect)
* [建议收藏 | Flink CDC 系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489988&idx=1&sn=c57b867645834257e8567a6e671273ff&chksm=c029625bf75eeb4d29d3f43d1b845aec3a7d176cfbceddd16ae6d195d4d96fa5cf6bfa9c362d&scene=21#wechat_redirect)
* 建议收藏 | [Doris实战文章合集](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490251&idx=1&sn=ae993d282a0a3cd968c3d6a19f79dce8&chksm=c0296154f75ee8428ee77a8e0c86ca08648967e7a68987aaf79d20bdbd481f0ab6d1d2729b53&scene=21#wechat_redirect)
* 建议收藏 | [Seatunnel 实战文章系列合集](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490357&idx=1&sn=61ae7c852b2a41e192c0797cc28fd182&chksm=c02960aaf75ee9bcd422d82ceb80362a0959df74ee7d336e0015c51b3eb6eb1a6d6774e99483&scene=21#wechat_redirect)
* 建议收藏 | [实时离线输数仓（数据湖）总结篇](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488973&idx=1&sn=42d2f8235c822030f187c0d49deb54f5&scene=21#wechat_redirect)
* [建议收藏 | 实时离线数仓实战第一阶段总](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488426&idx=1&sn=6fe3666f5157c651c08a0c8862c1efad&scene=21#wechat_redirect)结

如果喜欢 请点个在看分享给身边的朋友