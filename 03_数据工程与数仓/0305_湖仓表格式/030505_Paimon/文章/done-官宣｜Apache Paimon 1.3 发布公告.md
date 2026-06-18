> 已吸收至：[[03_数据工程与数仓/0305_湖仓表格式/030505_Paimon/030505_核心知识点/Paimon版本演进与特性边界|Paimon版本演进与特性边界]]
---
title: 官宣｜Apache Paimon 1.3 发布公告
author: Apache Paimon
date:
url: https://mp.weixin.qq.com/s?__biz=MzkyNjQ1MDI3Mg==&mid=2247484866&idx=1&sn=0b20f78ba7f0c414a740a2ea54b53dbd&chksm=c3f95b97d4376433365977fe77f962103adeb54db519a8d164cd4aed2d7763bd773168d50ec8&mpshare=1&scene=24&srcid=1213oO4IllyjTyhSSU6wzeJs&sharer_shareinfo=431666e259a100f4904a256dbaa7beb3&sharer_shareinfo_first=431666e259a100f4904a256dbaa7beb3#rd
---

Apache Paimon PMC 正式发布 1.3 版本。本次核心版本历经 3 个多月的精心打磨，累计完成 500 余项代码提交。我们谨向所有参与贡献的开发者致以诚挚谢意！ 

**0****1**

版本综述

1. PyPaimon：Python SDK 回炉重造，完全无 JVM 的纯 Python 实现版本，部分场景性能超越 Java SDK。
2. Row Tracking 给表添加全局的 Row ID；Data Evolution 给予了表快速更新列数据的能力，面向大宽表的优化。
3. Incremental Clustering：增量的方式对数据进行排序聚类，以相对低的代价来对数据布局进行优化，为 Append 表带来极速的查询体检。
4. Rest Catalog: Virtual File System，允许以库名表名的方式访问 Rest Catalog 管理的文件系统，统一目录名字、统一权限管理。
5. 性能优化：支持 Spark TopN 下推；limit 下推；引入新的高性能 Range Bitmap。
6. 性能优化：Manifest Cache 按照分区与 Bucket 来组织它的缓存，OLAP 引擎查询时扫描 Manifests 更高效。
7. 提交冲突：解决 MERGE\_INTO 与 COMPACT 同时进行导致文件存储错误的潜在风险。

**02**

多模态数据湖

多模态数据湖的方向，Apache Paimon 聚焦在以下几个方向：

1. 支持文本、图像、音视频等多模态数据存储，并同时支持其结构化 Tags、向量数据的统一存储。Paimon 社区正在开发 Blob 存储以及向量存储的能力。
2. 提供多模态数据的高效检索，包括随机检索及全局索引。Paimon 社区正在开发全局索引能力，提供 Bitmap、B树、向量索引等能力。
3. 深度 AI 集成与应用协同，对接 AI 相关分布式引擎及应用，Paimon 需要一个高性能的 Python SDK，对接无 JVM 的 AI Python 生态。
4. 多模态方向需要支持表的快速加列，以提供给多模态数据对应 Tag 的快速更新，支持工程师的快速打标能力，大幅提升 AI 处理的效率。

Paimon 1.3 版本在 #3 和 #4 都取得了较大进展，并针对 #1 和 #2 在设计与开发中，希望在下个版本能被发布出来。

## PyPaimon

面向 AI 的 Python 生态，我们需要有一个强大的 PyPaimon SDK。PyPaimon 在去年有过一个版本 (0.2)，基于 Py4j 封装了 Java 代码，虽然能满足所有表模式，但是有以下严重问题：

1. 性能太差，Py4j 如果有数据传输，性能倒退十分明显。
2. JVM 依赖，需要客户端的机器安装 JVM 的相关依赖。

为此，Paimon 1.3 中完全重塑了 PyPaimon 的代码，并融入到了 Paimon 主仓库里，完全从 Python 的生态重新实现了 Paimon 的 Python SDK。我们对比了它的相关性能：

可以看到，对比老版本 Python SDK，性能大幅领先；且对比 Java 实现，在某些场景也更快了，这得益于 Python 生态的 Arrow 的 Native 读写的性能优化。

注意，目前的 PyPaimon 在 Append 表中可以基本满足述求，但是对主键表只支持了简单的 Deduplicate 能力，丰富的模式暂不支持。社区在后续版本中：

1. 继续完善 PyPaimon，覆盖更多模式。
2. 后续使用 PyPaimon 与更多生态对接，比如 Ray 以及 Daft 等等引擎。

## Row Tracking

Row Tracking 允许 Paimon 在 Append 表中跟踪行级的变化。一旦在 Paimon 表上启用，表结构中将添加另外两个隐藏列：

1. \_ROW\_ID: BIGINT，这是表中每一行的唯一标识符。它用于跟踪行的更新，并可用于在更新、合并或删除时识别行。
2. \_SEQUENCE\_NUMBER: BIGINT，这是一个字段，指示此记录的版本。它实际上是此行所属快照的快照 id。它用于跟踪行版本的更新。

Row Tracking 的最大好处给表带来了全局 ID 的设计，这给我们后面的 Data Evolution 和全局索引机制打下了基础。

虽然 Paimon 支持完整的 Schema Evolution，允许您自由添加、修改或删除列模式。但是如何更新列的数据，你可以使用 MERGE INTO 语句，但是在执行中会重写所有影响到的行的数据，这个存储成本与计算成本都比较高。

Data Evolution 是 Append 表的一项新功能，它彻底改变了处理数据演化的方式，特别是在添加新列时。此模式允许您更新部分列，而无需重写整个数据文件。相反，它将新的列数据写入单独的文件，并在读取操作期间智能地将它们与原始数据合并。

比如以下的 SQL：

```
CREATE TABLE target_table (id INT, b INT, c INT) TBLPROPERTIES (  'row-tracking.enabled' = 'true',   'data-evolution.enabled' = 'true' );
INSERT INTO target_table VALUES (1, 1, 1), (2, 2, 2);
CREATE TABLE source_table (id INT, b INT);
INSERT INTO source_table VALUES (1, 11), (2, 22), (3, 33);MERGE INTO target_table AS t USING source_table AS s ON t.id = s.id WHEN MATCHED THEN UPDATE SET t.b = s.b  WHEN NOT MATCHED THEN INSERT (id, b, c)   VALUES (id, b, 0);
SELECT * FROM target_table;
+----+----+----+ | id | b  | c  |+----+----+----+ | 1  | 11 | 1  | | 2  | 22 | 2  | | 3  | 33 | 0  |
```

```

```

此语句仅根据源表 source\_table 的匹配记录更新目标表 target\_table 中的 b 列，id 列和 c 列保持不变，并插入具有指定值的新记录。这与未启用 Data Evolution 的表之间的区别在于，只有 b 列数据被写入新文件，非常的轻量。

典型数据测试的性能对比中，经 Data Evolution 后，对比原有 MERGE INTO 的性能：

1. MERGE INTO 耗时从 27 分钟优化到 17 分钟，大幅减少执行耗时，如果更新数据较少，对比更加强烈。
2. MERGE INTO 存储空间从 170 GB 降到 1 GB，大幅减少存储消耗，降低成本。

后续社区的计划：

1. 开发全局索引，包括标量索引与向量索引，加速数据查询。
2. 引入 Blob 存储，让 Paimon 表能轻易存储和分析 KB 到 GB 级的 Blob 数据。

**03**

Incremental Clustering

1.3 版本中为 Append 表提供了一种全新的灵活的数据管理方式— Incremental Clustering（增量式聚类），其不仅负责完成小文件的合并，还将以增量的方式对数据进行排序聚类，以相对低的代价来对数据布局进行优化，为 Append 表带来极速的查询体验。同时用户无需重写数据即可灵活调整聚类键，数据会随着增量 clsuter 的执行而动态演进，逐渐达到最佳效果，显著降低了用户数据布局相关的决策复杂度。

为权衡写放大和排序效果，Paimon 利用了 LSM Tree 的分层概念来对数据文件进行分层和 Universal Compaction 的思路来挑选需要聚类的文件。

通过多层级的设计，控制每次 cluster 的数据量，越高层级的数据聚类越稳定，被重写的概率越小，以此来减缓写放大，同时保证排序仍具有良好的效果。

## 查询性能对比

对比没有Cluster的表，在双聚类键过滤条件下，Incremental Cluster 查询效率可提升超过 150x；

对比没有Cluster的表，在单聚类键过滤条件下，提升效率可超过 17x:

从图中可以看出，对比全量排序的表，查询效率相差不多。

## 排序性能对比

在显著缩短数据布局优化耗时的前提下，Incremental Cluster 的执行时间对比全量排序最快提升超过 20x：

为 Append 表开启 Incremental cluster 后，通过定期调度 Incremental cluster，既能解决小文件问题，又可以让 Append 表保持一个比较优秀的查询效率，同时您还可以在查询模式更改后随时更改聚类键。

**04**

Virtual File System

REST Catalog 提供内置存储，包括 Paimon Table、Format Table 和 Object Table（也称为 Fileset 或 Volume），有些场景需要直接访问文件系统。而我们的 REST Catalog 给表生成的是 UUID 路径，这使得直接访问文件系统变得困难。

因此，PVFS (Paimon Virtual File System) 允许用户通过 "pvfs://catalog/database/table/" 的路径直接访问 Catalog 的所有文件，包括所有内部表。另一个优点是，所有用户访问此文件系统都是通过 Paimon REST Catalog 的权限系统，而不需要维护另一个文件系统权限系统。

比如通过 Spark SQL：

```
val spark = SparkSession.builder()  .appName("PVFS CSV Analysis")  .config("spark.hadoop.fs.pvfs.impl",    "org.apache.paimon.vfs.hadoop.PaimonVirtualFileSystem")  .config("spark.hadoop.fs.pvfs.uri",       "http://localhost:10000")  .config("spark.hadoop.fs.pvfs.token.provider", "bear")  .config("spark.hadoop.fs.pvfs.token", "token")  .getOrCreate()  spark.sql( s"""       |CREATE TEMPORARY VIEW csv_table      |USING csv       |OPTIONS (       |  path 'pvfs://catalog_name/database_name/my_format_table_name/a.csv',       |  header 'true',       |  inferSchema 'true'       |) """.stripMargin )
```

**05**

其它优化

Apache Paimon 社区持续的改进了存储以及读写链路，持续优化性能与易用：

1. 性能方面，支持更多的下推，比如 Spark TopN 下推；limit 下推；引入新的高性能 Range Bitmap；另外针对 OLAP 查询的性能，改进 Manifest Cache 按照分区与 Bucket 来组织它的缓存。
2. 使用方面，解决 MERGE\_INTO 与 COMPACT 同时进行导致文件存储错误的潜在风险，特别是 Deletion Vectors 模式下的冲突问题。