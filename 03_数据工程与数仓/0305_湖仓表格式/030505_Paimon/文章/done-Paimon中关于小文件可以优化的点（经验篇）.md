> 已吸收至：[[03_数据工程与数仓/0305_湖仓表格式/030505_Paimon/030505_核心知识点/Paimon生产实践与小文件治理|Paimon生产实践与小文件治理]]
---
title: Paimon中关于小文件可以优化的点（经验篇）
author: 大数据技能圈
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490454&idx=1&sn=16596fbede7cec524de7fc0b3b88246c&chksm=c147791e6f006b94ea6efaa223f1a2b7cb62205146838de616802339471bed4f281d8fdd421d&mpshare=1&scene=24&srcid=0125SChZ8mUkGHGXBSyJmSL2&sharer_shareinfo=5e0ff1b01389b2ba1717f76d790becd5&sharer_shareinfo_first=5e0ff1b01389b2ba1717f76d790becd5#rd
---

Paimon在运行的过程中，因为配置等各种原因，会产生小文件，小文件的多少又会影响到系统西能，如果你的小文件是存储在HDFS上面，那非常有可能会遇到以下问题：

* 稳定性问题：HDFS 中存在过多的小文件会使 NameNode 过载。
* 成本问题：HDFS 中的小文件会暂时占用至少一个块（Block）的大小，例如 128 MB。
* 查询效率：查询过多的小文件时，效率会受到影响。

这篇文章我们来看可以通过哪些方式可以处理小文件问题。

01

**检查点（Checkpoints）**

如果使用 Flink Writer，每个检查点会生成1到2个快照，并且检查点会强制文件在分布式文件系统（DFS）上生成，因此检查点间隔越小，生成的小文件就越多。

所以，首先要做的是**增加****检查点间隔**。默认情况下，不仅检查点会导致文件生成，Writer 的内存（write-buffer-size）耗尽也会将数据刷新到 DFS 并生成相应的文件。可以启用 write-buffer-spillable 选项，使得 Writer 在内存不足时溢出到 DFS 上生成更大的文件。

因此，第二件事是**增加** write-buffer-size 或 **启用** write-buffer-spillable。

02

**快照（Snapshots）**

Paimon 维护着文件的多个版本，文件的压缩和删除是逻辑上的，并不会真正地删除文件。文件只有在快照过期时才会被实际删除，因此减少文件数量的方法是**缩短快照过期的时间**。Flink Writer 会自动处理快照的过期。

> **这或许是一个对你有用的开源项目**，**data-warehouse-learning**项目是一套基于 MySQL + Kafka + Hadoop + Hive + Dolphinscheduler + Doris + Seatunnel + Paimon + Hudi + Iceberg + Flink + Dinky + DataRT + SuperSet 实现的实时离线数仓（数据湖）系统，以大家最熟悉的电商业务为切入点，详细讲述并实现了数据产生、同步、数据建模、数仓（数据湖）建设、数据服务、BI报表展示等数据全链路处理流程。
>
> **https://gitee.com/wzylzjtn/data-warehouse-learning**
>
> **https://github.com/Mrkuhuo/data-warehouse-learning**
>
> **https://bigdatacircle.top/**
>
> 项目演示：

03

**分区和分桶（Partitions and Buckets）**

Paimon 文件以分层的方式组织。下图展示了文件布局。从一个快照文件开始，Paimon 读取器可以递归地访问表中的所有记录。

例如，以下表格：

```
CREATE TABLE MyTable (    user_id BIGINT,    item_id BIGINT,    behavior STRING,    dt STRING,    hh STRING,    PRIMARY KEY (dt, hh, user_id) NOT ENFORCED) PARTITIONED BY (dt, hh) WITH (    'bucket' = '10');
```

表数据将被物理地切分为不同的分区，并且在分区内部进一步划分到不同的桶中，所以如果总体数据量太小，每个桶中至少会有一个文件。建议**配置较少的桶数量**，否则也会产生相当多的小文件。

04

**主键表中的LSM数据**

LSM 树将文件组织成若干个已排序的片段（sorted runs）。一个已排序的片段由一个或多个数据文件组成，并且每个数据文件仅属于一个已排序的片段（sorted runs）。

默认情况下，sorted runs 的数量由 num-sorted-run.compaction-trigger 决定，如果减少这个数量，就能够实现保留较少文件的目标，但需要注意的是，这可能会导致写入性能受到影响。

05

**追加表中的文件**

默认情况下，Append表 会自动进行压缩以减少小文件的数量。

然而，对于分桶的 Append 表，它仅会压缩桶内的文件以保持顺序性，这可能会导致保留更多的小文件。可以按照下面一步全量压缩的方式解决。

06

**全量压缩**

Append 表（分桶），单个桶中可能会存在几十个小文件，这是很难接受的。更糟的是，不再活跃的分区也会保留大量小文件。

通过**配置** full-compaction.delta-commits，可以在 Flink 写入时定期执行完整压缩。这可以确保在写入结束之前，分区已经被完全压缩。

07

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