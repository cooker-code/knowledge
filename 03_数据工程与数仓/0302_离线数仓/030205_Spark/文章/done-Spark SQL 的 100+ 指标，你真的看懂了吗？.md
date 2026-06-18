> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_核心知识点/SparkSQL指标与HistoryServer观测|SparkSQL指标与HistoryServer观测]]
---
title: Spark SQL 的 100+ 指标，你真的看懂了吗？
author: Apache Kyuubi
date: K.Y.K.Y.
url: https://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511978&idx=1&sn=c87c92e12de84fdab0df8b0153a2ff0a&chksm=cf71ce0757b00436a417a3473a2abac9592ab4097336ad5389ca0177b43adfae60d1851fd675&mpshare=1&scene=24&srcid=0528pxRaa0OzQJE6Az72FYjv&sharer_shareinfo=e3fe12998d9c5468832836cb9160cc6f&sharer_shareinfo_first=e3fe12998d9c5468832836cb9160cc6f#rd
---

## 什么是 SQL Metrics？

Spark SQL 的每个物理算子都可以定义 **metrics**——在查询执行过程中跟踪各种计数的指标。当你在 SQL 标签页点击一个查询，看到 "number of output rows: 5,000" 或 "peak memory: 512.0 MiB"，那些就是 SQL Metrics。

它们基于 Spark 的 `AccumulatorV2` 框架：每个任务更新自己的本地副本，任务完成后驱动端进行聚合。

## 五种指标类型

### 1. Sum（createMetric）

最简单的类型。所有任务的值求和为单一总计。

**显示格式：** `1,234,567`

**典型用途：** 行数、文件数、分区数。

### 2. Size（createSizeMetric）

用于字节量度。显示总计加上每任务的分布。

**显示格式：** `total (min, med, max): 512.0 MiB (128.0 MiB, 128.0 MiB, 128.0 MiB)`

`(min, med, max)` 分布对于检测数据倾斜至关重要——如果 `max` 是 `median` 的 10 倍，说明有掉队任务。

### 3. Timing（createTimingMetric）

用于毫秒级耗时。显示总计加上每任务分布。

**显示格式：** `total (min, med, max): 5.0 s (100 ms, 1.2 s, 2.0 s)`

### 4. NsTiming（createNanoTimingMetric）

与 Timing 相同但接受纳秒值，显示时自动转换为毫秒。典型用途：Shuffle 写入时间。

### 5. Average（createAverageMetric）

用于每任务平均值，显示平均值在各任务间的分布。

**显示格式：** `avg (min, med, max): (1.2, 2.5, 6.3)`

## 如何解读 "total (min, med, max)" 格式

```
peak memory
total (min, med, max)
512.0 MiB (128.0 MiB, 128.0 MiB, 128.0 MiB (stage 3.0: task 36))
```

| 字段 | 含义 |
| --- | --- |
| **total** | 所有任务的总和 |
| **min** | 最小的任务值 |
| **med** | 中位数（第 50 百分位） |
| **max** | 最大的任务值，标注 (stage X: task Y) |

**负载均衡时：** min ≈ med ≈ max

**数据倾斜时：** max >> med——检查标注的那个任务

## 完整 SQL Metrics 参考

### 扫描算子

| 指标 | 显示名称 | 类型 |
| --- | --- | --- |
| `numOutputRows` | number of output rows | sum |
| `numFiles` | number of files read | sum |
| `filesSize` | size of files read | size |
| `scanTime` | scan time | timing |
| `metadataTime` | metadata time | timing |
| `pruningTime` | dynamic partition pruning time | timing |

### 聚合算子

| 指标 | 显示名称 | 类型 |
| --- | --- | --- |
| `numOutputRows` | number of output rows | sum |
| `aggTime` | time in aggregation build | timing |
| `peakMemory` | peak memory | size |
| `spillSize` | spill size | size |
| `avgHashProbe` | avg hash probes per key | average |

### Join 算子

| 指标 | 显示名称 | 类型 |
| --- | --- | --- |
| `numOutputRows` | number of output rows | sum |
| `buildDataSize` | data size of build side | size |
| `buildTime` | time to build hash map | timing |
| `spillSize` | spill size | size |

### Shuffle / 广播 / Python UDF

完整的 Shuffle 写入/读取、广播交换、Python UDF、写入算子、MERGE INTO、有状态流处理算子的指标参考，请查看博客原文。这里列出最常用的几个：

| 指标 | 含义 | 类型 |
| --- | --- | --- |
| shuffleBytesWritten | Shuffle 写入字节数 | size |
| shuffleWriteTime | Shuffle 写入时间 | nsTiming |
| fetchWaitTime | Shuffle 读取等待时间 | timing |
| numCoalescedPartitions | AQE 合并的分区数 | sum |
| numSkewedPartitions | AQE 检测到的倾斜分区数 | sum |
| broadcastTime | 广播时间 | timing |
| pythonProcessingTime | Python 代码执行时间 | timing |

## WholeStageCodegen 与指标范围

大多数算子被 WholeStageCodegen 融合成单一 JVM 方法。它们的行数指标（`numOutputRows`）各自准确，但没有各自的计时——因为它们作为一个编译函数执行。

在**代码生成管道之外**有独立执行阶段并具有独立计时的算子：

* `SortExec`

  （排序时间）
* 聚合算子（聚合构建时间）
* `ShuffledHashJoinExec`

  （哈希表构建时间）
* `BroadcastExchangeExec`

  （收集/构建/广播时间）
* `ShuffleExchangeExec`

  （Shuffle 写入时间）
* Python UDF 算子（Python 工作器时间）
* 有状态流处理算子（更新/删除/提交时间）

参考链接

[1] 完整博客原文 https://yaooqinn.github.io/zh/posts/spark/understanding-sql-metrics/

[2] Apache Spark SQL Metrics 源码 https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/metric/SQLMetrics.scala