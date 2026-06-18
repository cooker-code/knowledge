> 已吸收至：[[04_OLAP与数据库/0408_hbase/040801_HBase/040801_核心知识点/HBaseCompaction与低延时调优边界|HBaseCompaction与低延时调优边界]]
---
title: HBase 实践 | 货拉拉 HBase Compaction 调优实践
author: HBase技术社区
date:
url: http://mp.weixin.qq.com/s?__biz=MzU5OTQ1MDEzMA==&mid=2247492513&idx=1&sn=a7eb9a7193ae47e1eb363ff1a7325486&chksm=feb614dcc9c19dcac7be7b3c13489e018954937409262eb08b198710e83665e735120ed71ed9&mpshare=1&scene=24&srcid=12274G2BhioKCVsqn59DHfoL&sharer_shareinfo=f95a9982d338b62ef072deba317d5420&sharer_shareinfo_first=f95a9982d338b62ef072deba317d5420#rd
---

## |背景

> LSM-Tree（Log-Structured Merge Tree）架构通过Log Append的方式带来了高吞吐的写，但随着内存中数据不断的刷盘，不同文件之间的数据范围通常是交叠的，这会引起读性能的下降和存储空间的膨胀。而Compaction机制，就是通过周期性的任务不断地合并交叠的文件来优化读性能和空间问题。Compaction通常会大量占用集群的I/O、CPU资源，因此如何权衡存储空间、I/O、CPU资源成为日常运维中的一个难题。本文就从HBase （version=2.0.2）的生产实践出发，介绍一下货拉拉在HBase Compaction上的调优实践。

## 概念解释

在日常的HBase Compaction运维中，经常会提到几组概念：Minor/Major、Short/Long、Small/Large。其对应的理念经常被混淆，这里详细地解析一下

* • Minor/Major

| 概念 | 定义 | 特点 |
| --- | --- | --- |
| Minor Compaction | 选取HStore部分相邻的HFile进行合并 | 1. 按照HFile删除整个HFile都过期的文件 |
| Major Compaction | 选取HStore所有的HFile进行合并 | 1. TTL过期数据 2. 被删除的数据 3. 超过设定版本号的数据 |

* • Short/Long、Small/Large

| 概念 | 定义 | 特点 |
| --- | --- | --- |
| Short/Long | Compaction线程池的名称，分别叫做ShortCompactions和LongCompactions | 1. 通过hbase.regionserver.thread.compaction.small和hbase.regionserver.thread.compaction.large来控制对应线程池大小 |
| Small/Large | 是Short/Long两个线程池的队列，分别叫做SmallCompactionQueue和LargeCompactionQueue | 1. 通过hbase.regionserver.thread.compaction.throttle控制Compaction任务存放于哪个队列 2. 当LargeCompactionQueue队列为空时，longCompactions线程池会去执行SmallCompactionQueue中的Compaction任务 |

四者的关系如下图所示：

## 生产实践

Compaction是资源消耗型任务，过多或过少的压缩都会影响业务的读/写性能。Compaction的调优就是在业务rt性能能容忍的前提下，利用最少的资源最高效率的运行。所以无法做到一份配置应对所有的业务场景，但是整体的思路是一致的。

### 控制最大合并

通常我们会认为Major Compaction对集群稳定性有较大影响，所以大家都会将Major Compaction由自动触发改手动触发。其实这种认知不完全对，影响集群稳定性的是Large Compaction（即单次合并的HFile文件总大小过大，我们称之为：大合并），Major是对整个HStore进行Compaction，这通常是一个大合并，但默认情况下Minor也会触发大合并。所以控制大合并，主要是两步：

1. 1. 关闭自动Major Compaction

```
hbase.hregion.majorcompaction=0 
```

2. 定义大合并

结合业务RT要求、集群性能，我们发现不同的业务能容忍的最大合并不一样。我们集群资源配置下，较多的业务高峰期对单次合并的HFile总大小超过1G时（snappy压缩后的HFile总大小），业务就出现rt抖动超过容忍范围，所以我们将超过1G的合并定位为大合并，并禁止了高峰期执行大合并。

```
hbase.hstore.compaction.max.size=1073741824
```

### 提升合并效率

对于一个持续稳定的业务而言，每次Flush出来的HFile文件大小基本上相似，在以默认的Compaction配置运行时，我们发现一个现象：

> 如果hbase.hstore.compaction.max.size=2G，HStore中存在这样大小的两个HFile：1G、900M，此时无论Flush出来的文件有多大（假设15M），一定会触发Compaction。

这种Compaction的文件差异过大的情况，其Compaction的效率不高。对此我们联想到了“2048游戏”：每次都合并相同大小的数字。并依据这个特点对Compaction进行一版优化，为了方便展示，以单次最少合并4个HFile，每次Flush的HFile为15M为例，详细分析一下业务生成30个HFile（30s生成一个）的合并过程：

> 优化效果
>
> 1. 1. Compaction次数减少1次，减少了27.8%的Compaction IO（从Compaction 1080M HFile减少到780M)
> 2. 2. 第一个flush的HFile文件减少了50%（2次）的Compaction次数

我们在一个TTL表，无数据更新删除，查询只有Get，要求rt P99<200ms的业务场景下上线了该优化，运行一段时间后效果：

> 1. 1. HStore 的最大文件数的监控较之前增加2个，高峰期集群CPU抖动峰值进一步降低
> 2. 2. 业务Get请求rt抖动降低，整体RT水位无影响

相关配置设置如下：

```
hbase.hstore.compaction.min=4
hbase.hstore.compaction.min.size=20971520  ## 20M
hbase.hstore.compaction.ratio=0.4          ## HBase官网也有解释到，如果可以用BloomFilter过滤HFile，可降低该值来降低写入成本
```

### 确定业务高峰

在货拉拉大部分的业务有着明显的高峰和低谷，对应集群的资源水位也有着非常明显的高峰低谷：

结合流量监控和各个业务特点确定业务的流量低谷时间范围，大部分业务将[20:00 8:00]定义为低峰期，所以对应HBase Offpeak的时间段设置为：

```
hbase.offpeak.start.hour=20
hbase.offpeak.end.hour=6
```

这里需要注意2点：

1. 1. 不同的业务高峰低谷存在差异，可以按照RsGroup级别去控制，这里以公司较为典型的场景为例。
2. 2. Offpeak的结束时间必须在业务低峰结束时间之前，这是因为Offpeak Compaction如果在7:50分触发较大的Compaction任务，执行时间可能达到小时级别，那么就存在较大的可能影响到业务高峰期的稳定。

### 充分利用低峰期

通过上面的优化，我们将HFile的大小控制在1G以内，虽然减少了Compaction次数，但这会导致单表HStore的平均HFile数量会不断增长。前面提到业务有着非常明显的高低峰，对此我们想到充分利用低峰的资源来将白天生成的HFile合并成一个更大的HFile。

调整相关配置后，HStore中HFile大小的效果如下：

* • 配置调整

max.size需要按照业务配置：确定希望控制HFile在多大的一个范围和单个HStore日常的数据量

```
hbase.hstore.compaction.max.size.offpeak=5368709120
hbase.hstore.compaction.ratio.offpeak=5.0
```

* • HFile大小表现

利用低峰期资源合并成更大的HFile，从而控制单个HStore的HFile数量<16个

线上运行一段时间，我们预期通过hbase.regionserver.thread.compaction.large 来控制Offpeak Compaction的并行度，从而让低峰期集群资源（CPU、I/O）能达到高峰期的水平，并且不影响业务的RT，充分压榨集群资源。但是运行了一段时间，我们发现并不符合预期，通过日志和代码分析，最终确认Offpeak Compaction在一台RS上只能以一个并行度运行。我们修改对应逻辑，提高了Offpeak Compaction的并行度，从而达到更多的利用低峰期集群资源的目的，见：HBASE-27861。

### 选择性禁用Major

关闭了自动的Major Compaction，但手动模型下，也并非所有的表都会触发Major。

|  | 业务场景 | 为什么可禁用 |
| --- | --- | --- |
| TTL表 | 1. 业务不存在大量更新或删除操作 2. 不会写入一些未来的timestamp | 1. Minor Compact特性：会清理整个HFile都过期的文 2. 磁盘容量充足：按照上文的例子，单个HStore会多存储7天的过期数据 |
| 非TTL表 | 1. 小表（单Region <5G） 2. 无大量数据更新、删除操作 | 1. HFile数量和大小关系满足Offpeak Compaction的要求（max.size.offpeak=5G，ratio.offpeak=5.0）时，Offpeak Compaction会选中所有的HFile进行合并（ALL\_FILES） 2. DisplayCompactionType=ALL\_FILES时也会像Major一样进行数据清理 |

对于其他必须手动执行Major的场景，我们也通过外挂的Major Compaction调度器来实现了多种策略，见：《[货拉拉HBase Bulkload实践](http://mp.weixin.qq.com/s?__biz=MzI0MjIxNjE0OQ==&mid=2247486761&idx=1&sn=30c955adb9ef7126d5166ead87d08fa8&chksm=e97ef093de097985082bbbbb1efcd5b3b6ebbec8362ff6713292bf276c10e816a98ead8a03a8&scene=21#wechat_redirect)》一文。

### 深入业务

在我们一个大表（单副本580TB）场景，以往在低峰期会手动触发Major去合并HFile清理过期数据。通过观察发现，无论Region设置为50G还是100G，一台RS上所有的Region执行一轮Major需要15天左右，在这个时间内，最早执行Major的HStore中HFile数量可能达到70+。我们将上述所有的操作应用到该业务场景后，RS级别HStore最多的HFile数量确实得到了很好的控制（<20），但业务反馈Scan rt P99增长了30%，超过了业务可接受范围。对此，我们结合HBase 2.0.2和业务场景，得到如下信息和对应改进措施：

|  | 存在问题 | 如何改进 |
| --- | --- | --- |
| 业务侧 | 1. rowkey设计为：userId\_数据时间戳 2. 查询方式为scan，设置startKey（userId\_开始时间）和endKey（userId\_结束时间） | 1. 数据写入时将数据时间戳替换HBase version timestamp 2. Scan查询时addTimeRange |
| 服务侧 | 1. bloom filter只有row和rowcol，均不支持Scan请求 2. scan请求默认timeRange为ALL\_TIME | 1. 合入HBASE-20636中的bloom filter类型：ROWPREFIX\_FIXED\_LENGTH |

在优化前，Scan请求无法通过BloomFilter和TimeRange去过滤HFile，所以如果表的平均HFile数量增加后，对Scan的延时影响是比较大的。但在Scan中设置TimeRange后，查询T-1的数据时通过通过Timestamp确定到最多两个HFile中，再结合新的BloomFilter可进一步过滤HFile，从而提升业务Scan性能。

## 总结

Compaction的调优，并不是一个简单的服务侧的优化，也不能一套参数搞定所有的业务场景和集群。Compaction相关的配置几乎都是支持动态修改的，只有深入理解业务，结合自身集群的资源调整相关的配置，最终找到一个适合的参数组合。本文从真实的生产实践出发，有着我们对每一个参数的思考和理解，希望能给读者带来一些启发，同时也欢迎和我们沟通。

> 笔者介绍：
>
> 章啸｜大数据基础架构组，负责大数据存储方向，Apache HBase Contributor