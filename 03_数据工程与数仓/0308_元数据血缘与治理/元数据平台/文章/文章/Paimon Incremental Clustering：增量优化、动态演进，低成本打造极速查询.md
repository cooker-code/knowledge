---
title: Paimon Incremental Clustering：增量优化、动态演进，低成本打造极速查询
author: Apache Paimon
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkyNjQ1MDI3Mg==&mid=2247484847&idx=1&sn=20b64bfa4c804f15be2e5c5275ade364&chksm=c3f40a1765eead0ee9785f381c49a89344a0626f38f70e5efd07dc2e652265af3d2b23ca9ca4&mpshare=1&scene=24&srcid=11122rdSbEgtDw5XgVjG7hML&sharer_shareinfo=2a9715c026d8fb9dd5b7a4e5dd152b6c&sharer_shareinfo_first=2a9715c026d8fb9dd5b7a4e5dd152b6c#rd
---

**00**

摘要

Paimon 为 Append Scalable 表提供了一种全新的灵活的数据管理方式— Incremental Clustering（增量式聚类），其不仅负责完成小文件的合并，还将以增量的方式对数据进行排序聚类，以相对低的代价来对数据布局进行优化，为 Append 表带来极速的查询体验。同时用户无需重写数据即可灵活调整聚类键，数据会随着增量 clsuter 的执行而动态演进，逐渐达到最佳效果，显著降低了用户数据布局相关的决策复杂度。

**01**

技术背景

创建 Paimon Append 表时，在 WITH 参数中指定 `'bucket' = '-1'`（默认值），将会创建 Append Scalable 表，也称之为 Append Unaware-bucket 表。Append Scalable 表具有以下几个特点：

* 写入效率高

Append Scalable 表由于无视桶的概念，在写入时，并发设置更加随意和便捷，不需要考虑桶对于并发数量的限制约束，并且直接由上游将数据推往 writer 节点。由此，在其他条件相同的情况下，此类型的表写入速度通常更快。

* 导入/迁移方便

Append Scalable 表可以直接吸收或者转换现存的 hive 表、iceberg 表、hudi 表等，不需要特定的 etl 读写过程，直接从文件级别将 hive 表转换为 paimon 表，效率非常高。详情参考官网文档：https://paimon.apache.org/docs/master/migration/clone-to-paimon/

* 数据可排序

Append Scalable 表的数据可以分区排序，利用 order、zorder 和 hilbert 对表进行排序后，可很大程度上增大查询效率。排序后的此种类型的表，通常查询速度远高于排序前。详情参考官方文档：https://paimon.apache.org/docs/master/maintenance/dedicated-compaction/#sort-compact

我们详细讨论下第三点「数据可排序」。Paimon 默认会在文件元数据中记录每个字段的最大最小值等统计信息，在查询时，根据查询条件和文件元数据的统计信息，能够做到文件级的过滤。如果文件级的过滤效果良好，那么需要读取的文件数将会大大减少，毋庸置疑会极大提高查询的效率。

但表的数据布局并不总是完美契合查询过滤条件的，不同文件间关于查询字段的统计信息可能存在大幅度的重叠，导致过滤效果微乎其微。Paimon 则支持对 Append Scalable 表按照某些特定字段进行排序，让数据按照排序字段重新聚类，重新聚类过的数据文件布局在面向排序字段进行查询时，效率会有极大的提升，且数据规模越大，提升效果越明显。这个排序的过程即为 Paimon 的 SortCompact。

#### ****1.1 SortCompact****

早在 23 年 10 月（Paimon release-0.5），Paimon 就支持对 Append Scalable 表进行排序来获取更佳的查询效率，文章指路👉[基于 Apache Paimon 的 Append 表处理](https://mp.weixin.qq.com/s?__biz=MzkyNjQ1MDI3Mg==&mid=2247484078&idx=1&sn=b553af3a1564066460f5197ce6c2c63d&scene=21#wechat_redirect)。

用户可以自定义排序字段、排序策略以及指定分区来完成对 Append Scalable 表的数据布局重组。

* 排序字段：一般定义为常用的查询字段
* 排序策略：支持 order、zorder、hilbert 三种排序策略。默认一个排序字段用 order，二至四个使用 zorder，四个以上使用 hilbert
* 指定分区：支持限定只优化特定分区的数据布局

然而，SortCompact 也存在一些劣势。只要执行 SortCompact，就会重写全表/全分区的全量数据，即使可以通过限定分区来控制重写的数据量，每次的重写代价依然十分高昂。并且，SortCompact 是无记忆性的，也就是说即使数据已经处于排序完毕的状态，执行 SortCompact 依然会重写全量数据。除此之外，不能直接通过查询表的元数据信息来查看表当前的排序状态，排序字段需要用户自己来维护并记录。

#### 

#### ****1.2 痛点****

如上所述，用户在使用 SortCompact 来加速查询时仍然存在一些痛点，具体面临以下几个痛点：

痛点 1:全量 SortCompact 执行代价高昂，写放大尤为严重

SortCompact 在执行时并不会感知数据当前的布局情况，这就导致了只要存在数据更新，为了让新数据得到最优的查询体验，即使存量数据已经排序好，也必须要重写全量的数据，执行代价高，写放大非常严重。

痛点 2:最优排序字段决策困难

排序字段的选择和查询模式息息相关，而业务的查询模式可能会随着发展而变化，对数据管理者而言，很难在一开始就设计出完美的排序字段。在现有方案中，一旦查询字段发生变化，与排序字段不再一致，那么排序带来的查询加速将不再适用，用户只能用新的查询字段作为排序字段重新对数据进行全量 SortCompact。

痛点 3:无法通过表的元数据信息感知表的排序状态，用户需要自己维护表的排序状态

SortCompact 并未将排序信息持久化到表的元数据中，因此无法仅从表的元数据判断数据是否经历过排序，用户需要自己来维护表的排序状态，记录表当前的排序字段。

**02**

Incremental Clustering

Incremental Clustering 是 Paimon 新引入的一种针对 Append Scalable 表的数据布局优化手段，它可以解决上述所有痛点，在控制写放大的前提下，提高数据布局优化的执行效率，为 Append Scalable 表带来极速的查询体验。开启 Incremental Clustering 后，每次执行聚类操作（即上文提及到的排序操作）时，仅选取特定的文件集来做 cluster，避免重写全量数据，以相对低的代价来对数据布局进行排序优化，提升数据查询能力。除此之外，用户可随时灵活调整聚类键，无需重写全量数据，数据会随着 Incremental Clustering 的执行而动态演进，逐渐达到最佳效果，显著降低了用户数据布局相关的决策复杂度。

#### ****2.1 功能定位****

####

Incremental Clustering 提供了以下功能：

1. 支持小文件合并，在重写数据文件时会遵循 target-file-size

Incremental Clustering 不仅会对数据进行排序，在对排序后的数据进行重写时也会遵循设置的 target-file-size。对 Append 表而言，在功能上，Incremental Clustering 其实覆盖了 Compact 的功能，并且由于执行 Incremental Clustering 后，表的数据布局状态是有序的，而 Append 表在做 Compact 时会挑选某些文件来做合并与重写，会破坏这种有序性，所以 Incremental Clustering 与 Compact 是相斥的。目前开启了 Incremental Clustering 的 Append Scalable 表会禁用 Compact，Incremental Clustering 本身已经可以承担 Compact 的职责了。

2. 支持增量式聚类（排序），尽可能减少写放大

相较于 SortCompact，Incremental Clustering 在 cluster 时只会挑选特定的文件集来进行排序重写，避免每次执行都重写全量数据。开启 Incremental Clustering 后，表的聚类状态可以通过元数据查询得知，包括当前排序字段以及当前数据文件的排序状态。

3. 支持灵活更改聚类键

用户可根据查询模式的变化随时更改表的聚类键，新写入的数据将会按照新的聚类键进行排序。更改聚类键并不会立马引起全部存量数据的重写，而是会随着 Incremental Clustering 的执行而动态优化数据布局，逐渐达到最佳效果。

4. 提供 full 模式的执行模式

更改聚类键后，如果用户希望旧的存量数据也能立马享受到新的聚类键带来的查询效率提升，可以选择对表执行 full 模式的 Incremental Clustering，full 模式会对全量数据进行排序与重写。

#### ****2.2 用户使用****

####

1. 开启 Incremental Clustering 功能

首先您需要为 Append Scalable 表开启 Incremental Clustering 功能。可以在建表时设置以下配置项来开启 Incremental Clustering 功能，开启后，表的 Compact 功能会被禁掉。

|  |  |  |  |  |
| --- | --- | --- | --- | --- |
| 配置项 | 值 | 是否必须 | 类型 | 描述 |
| clustering.incremental | true | 是 | Boolean | 默认为 false，开启 Incremental Clustering 必须设为 true |
| clustering.columns | 'column1,column2' | 是 | String | 聚类列，也称为排序列。格式是'column1,column2'，多个字段用","分割。不推荐将分区字段设置为聚类列 |
| clustering.strategy | 'zorder' 或 'hilbert' 或 'order' | 否 | String | 排序算法。如果没设置，默认一个排序字段用 order，二至四个使用 zorder，四个以上使用 hilbert |

2. 执行 Incremental Clustering

开启 Incremental Clustering 功能后，您可以在批模式下定期调度执行 Incremental Clustering，持续优化表的数据布局，从而提升表的查询性能。您可以通过 Spark SQL 或 Flink Action 来执行 Incremental Clustering。

Spark SQL：

```
-- 设置写入并发数，如果并发设置的过大，可能会产生大量的小文件SET spark.sql.shuffle.partitions=10;-- 执行 Incremental Clustering，默认以增量模式执行CALL sys.compact(table => 'T')-- 执行 full 模式的 Incremental Clustering，这会重写全量的数据文件CALL sys.compact(table => 'T', compact_strategy => 'full')
```

Flink Action：

```
# 执行以下命令来提交一个 Incremental Clustering Job<FLINK_HOME>/bin/flink run \    /path/to/paimon-flink-action-1.4-SNAPSHOT.jar \        compact \        --warehouse <warehouse-path> \        --database <database-name> \        --table <table-name> \        [--compact_strategy <minor / full>] \        [--table_conf <table_conf>] \        [--catalog_conf <paimon-catalog-conf>     [--catalog_conf <paimon-catalog-conf> ...]]# 示例, 注意：sink.parallelism 控制写入并发度，过大可能会导致小文件过多<FLINK_HOME>/bin/flink run \        /path/to/paimon-flink-action-1.4-SNAPSHOT.jar \        compact \    --warehouse s3:///path/to/warehouse \        --database test_db \        --table test_table \        --table_conf sink.parallelism=2 \        --compact_strategy minor \        --catalog_conf s3.endpoint=https://****.com \        --catalog_conf s3.access-key=***** \        --catalog_conf s3.secret-key=*****
```

#### 

#### ****2.3 实现****

####

需要解决的核心问题是如何做增量更新，即如何挑选部分文件进行排序和重写。这其中需要考虑排序效果和写放大。如果每次参与排序重写的数据量太大，写放大会比较高；如果参与排序的数据量太小，排序效果又会受到影响。

如何挑选文件？

一个简单的思路是确定一个阈值，当某一批排序文件的大小超过该阈值时，我们就认为这批排序文件已经趋于稳定了，下次再挑选需要聚类的文件时就跳过这些文件。但这个阈值很难确定，如果定小了，排序效果不佳；如果定大了，那么排序集在达到这个阈值前会不断的被重写，写放大很严重。

因此，为权衡写放大和排序效果，Paimon 利用了 LSM Tree 的分层概念来对数据文件进行分层和 Universal Compaction 的思路来挑选需要聚类的文件。

* 新写的数据都会存在于 level-0，level-0 的文件都是未经聚类的文件。
* level-i 中的所有文件都是在同一个排序集内排序后生成的文件。
* 类比于 Universal Compaction 的概念，level-0 中一个文件即为一个 sortedRun，level-i 中的所有文件是一个 sortedRun，cluster 时，以 sortedRun 为基本单位。

通过多层级的设计，控制每次 cluster 的数据量，越高层级的数据聚类越稳定，被重写的概率越小，以此来减缓写放大，同时保证排序仍具有良好的效果。

Incremental Clustering 的总体流程如下：

**03**

性能测试

#### 

#### ****3.1 测试准备****

#### 

#### **■ 测试数据**

测试数据使用 flink 的 datagen 在批模式下生成。总数据大小约为100G，数据条数为 12.6 亿条。数据将会分成 18 批写入到测试表中，每批生成的数据条数为 7000w，大小约为 5.6G。

准备三张测试表分别命名为 cluster、sort、uncluster。三张表每次都会被导入相同的测试数据，每批数据写入结束后，三张表将会执行不同的操作来优化数据布局：

* 增量 cluster 表：执行 incremental cluster（增量 cluster）
* 全量 sort 表：执行全量 sort compact
* uncluster 表：由于每次批写后生成的数据文件均为大文件（超过 250MB），故无需再额外执行小文件合并

增量 cluster 表的定义如下，其余两张表除了未设置 Incremental Cluster 相关参数外，其余与增量 cluster 表完全一致。

```
CREATE TABLE `paimon_cluster`.`cluster_test`.`cluster`(    pt   STRING        ,f1  INT        ,f2  INT        ,f3  STRING        ,f4  STRING        ,f5  STRING        ,f6  BIGINT        ,f7  BIGINT        ,f8  DECIMAL(10, 3)        ,f9  DOUBLE        ,f10 TIMESTAMP        ,f11 STRING        ,f12 STRING) WITH (    'clustering.incremental' = 'true',        'clustering.columns' = 'f1,f2',        'clustering.strategy' = 'zorder');
```

#### 

#### **■ 测试环境**

测试在阿里云 severless spark 上进行，spark 集群的主要配置如下，存在 2 个 executors。

```
spark.driver.cores                      2spark.driver.memory                     8gspark.executor.cores                    4spark.executor.memory                   16g
```

#### 

#### ****3.1 测试结果****

1. 在双聚类键过滤条件下，Incremental Cluster 查询效率可提升超过 150x；在单聚类键过滤条件下，提升效率可超过 17x

图例说明：figure-2 中，sort:f1 表示执行全量 Sort Compact 后，以 f1 为查询条件时的查询时间；cluster:f1 表示执行 Incremental Cluster 后，以 f1 为查询条件时的查询时间；uncluster:f1 表示对表不做多余操作，以 f1 为查询条件时的查询时间。figure-1 与 figure-3 中的图例意义与 figure-2 类似，查询条件不同。

figure-1 显示，在用两个聚类键（f1+f2）作为查询条件时，相较于数据布局未优化过的表（uncluster 表），incremnetal cluster 后的查询时间几乎都在 1s 左右，查询效率提升极其明显，随着数据量的增大，效率提升超过了 150 倍。

figure-2 与 figure-3 显示，在用单聚类键（f1或f2）作为查询条件时，incremnetal cluster 带来的查询效率提升依然十分明显，平均下来有 10 倍左右的效率提升，最大达到了 17 倍。且对比全量 Sort Compact 的查询性能相差不大。

这是因为 Paimon 会记录文件中各个字段的统计信息作为元数据，在 scan 阶段会根据查询条件进行文件级的过滤，经过 incremental cluster 后的文件会根据聚类键进行排序，数据文件中聚类键的统计信息更加“紧密”，能过滤掉更多无关文件，从而带来查询效率的提升。而 uncluster 表虽然不存在小文件问题，但由于各个字段的值都是随机的，文件中字段的 min/max 统计信息范围过大，失去了过滤能力，所以查询效率随着数据量的增加会越来越慢。

2. 在显著缩短数据布局优化耗时的前提下，Incremental Cluster 仍能有效保障查询性能。实测显示，优化后执行时间最快提升超过 20x

从 figure-4 可以看出，大部分情况下 Incremental Cluster 的执行时间都显著低于全量 Sort Compact，最快提升超过 20 倍。这是因为全量 Sort Compact 每次执行时都会对全局的数据文件进行排序重写，这样确实可以达到最优数据布局，查询速度最快，但全量写消耗过大，并且随着数据量的增长代价会越来越高。而 Incremental Cluster 则是增量式的排序重写，保证优化新数据的数据布局，同时选择性的重写老文件，逐步达到最佳数据布局。figure-4 中，Incremental Cluster 的执行时间大部分都在 5min 以内，两次波峰对应着两次 full cluster，消耗的时间会比较久。但每经过一次 full cluster，最高层的文件就会越稳定，发生 full cluster 的概率也就会越低。

从 figure-5 可以看出，两次 full cluster后，增量 cluster 表的查询性能都有一定程度的提升，查询性能与全量 sort 表趋于一致。此外，对于双聚类键的查询，Incremental Cluster 与全量 Sort Compact 的效果相差不大；而对于单聚类键的查询，Incremental Cluster 会稍慢于全量 Sort Compact，最大差距在 2 倍左右。这是因为单聚类键的过滤效果没有双聚类键强，会筛选出更多的文件，而增量 cluster 表中可能存在多个层级，每个层级为一个排序单元，也就是说每个层级都可能都会筛选出一部分候选文件，所以查询效率相较 全量 sort 表会慢一些。但从 figure-2 与 figure-3 中可以看出，相较于 uncluster 表，查询效率的提升还是非常显著的。

总之，为 Append 表开启 Incremental cluster 后，通过定期调度 Incremental cluster，既能解决小文件问题，又可以让 Append 表保持一个比较优秀的查询效率，同时您还可以在查询模式更改后随时更改聚类键。

Incremental Clustering 已在 Paimon-1.3.0 版本中正式发布，如果您需要改进现有 Append 表的查询效率，非常推荐尝试！更多精彩请：

1. 关注 Apache Paimon 微信公众号，了解更多咨询。

2. 点赞项目：https://github.com/apache/paimon

   **点击「阅读原文」**