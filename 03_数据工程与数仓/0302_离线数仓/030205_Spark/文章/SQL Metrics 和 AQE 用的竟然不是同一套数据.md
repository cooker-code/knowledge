---
title: SQL Metrics 和 AQE 用的竟然不是同一套数据
author: Apache Kyuubi
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511982&idx=1&sn=dec0a6fbe9ca3019d7e75e04c1bf1fd2&chksm=cf068fd2b21f95d34f46c8b45e2f4f16fa302e80c6e8d32eaff8a91f16b8571cc0fa63c13ed6&mpshare=1&scene=24&srcid=0403Onzd1fCUQOkLxCT7ql2U&sharer_shareinfo=7fba2ff442517036be1f652af0b6b900&sharer_shareinfo_first=7fba2ff442517036be1f652af0b6b900#rd
---

SQL Metrics 和 AQE 用的竟然不是同一套数据？先来看看`SQLMetric！`

## AccumulatorV2 生命周期

每个 SQL 指标都是一个 `SQLMetric`，它继承自 `AccumulatorV2[Long, Long]`。任务在 Executor 上运行时，使用的是累加器的**本地副本**。算子代码通过 `metric += value` 来更新指标——执行过程中不产生任何网络通信。

### 从任务到驱动端

```
Task (Executor)           Driver  
─────────────           ──────  
metric.add(value)          
    ↓                      
Task completes →────→ onTaskEnd()  
                         ↓  
                    Store in LiveStageMetrics  
                         ↓  
                    aggregateMetrics()  
                         ↓  
                    MetricUtils.stringValue()  
                         ↓  
                    Map[accId → "512.0 MiB (min, med, max)"]  
                         ↓  
                    Persist to KVStore
```

任务完成后，驱动端通过 `SparkListener` 事件接收累加器更新。`SQLAppStatusListener` 处理 `onTaskEnd()`——提取指标值并存储到 `LiveStageMetrics` 中。

### 聚合与存储

对于**已完成**的执行，指标经过 `aggregateMetrics()` 计算 `total (min, med, max)` 分布，格式化为字符串后持久化到 KVStore。原始每任务值随后被丢弃。

对于**正在运行**的执行，聚合是实时计算的。每次刷新 SQL 标签页，都会从内存中的任务值重新计算。这就是查询运行时指标能近实时更新的原因。

### 驱动端指标

并非所有指标都来自任务。子查询执行时间和广播时间直接在驱动端产生，通过 `SQLMetrics.postDriverMetricUpdates()` 上报，完全绕过任务生命周期。

## AQE 如何利用统计信息做出运行时决策

这里很关键：**AQE 不使用 SQL Metrics**。它使用完全独立的数据源——**MapOutputStatistics**。

### 数据流转过程

1. `ShuffleExchangeExec`

   提交 Shuffle Map Stage
2. Map 任务运行，将 Shuffle 数据写入本地磁盘
3. 所有 Map 任务完成后，`MapOutputTracker` 精确知道每个 Reducer 分区的字节数
4. 信息封装为 `MapOutputStatistics`，包含 `bytesByPartitionId: Array[Long]`
5. `AdaptiveSparkPlanExec`

   在 Stage 物化**之后**运行优化规则

核心要点：AQE 等待 Shuffle Stage 完成，然后利用**实际输出大小**决定下一步操作。

### CoalesceShufflePartitions——合并小分区

读取 `bytesByPartitionId`，将相邻小分区合并至目标大小（默认 64 MB）。

| 配置项 | 默认值 | 用途 |
| --- | --- | --- |
| advisoryPartitionSizeInBytes | 64 MB | 合并后分区目标大小 |
| coalescePartitions.minPartitionNum | （无） | 最小分区数 |
| coalescePartitions.minPartitionSize | 1 MB | 不会创建小于此值的分区 |

### OptimizeSkewedJoin——拆分倾斜分区

读取 Join 两侧的 `bytesByPartitionId`，计算中位数，满足以下条件的分区被标记为倾斜：

```
size > max(skewThreshold, median × skewFactor)
```

| 配置项 | 默认值 | 用途 |
| --- | --- | --- |
| skewedPartitionThresholdInBytes | 256 MB | 绝对最小阈值 |
| skewedPartitionFactor | 5.0 | 必须达到中位数的倍数 |

两个条件必须**同时满足**。倾斜分区被拆分为目标大小（64 MB）的子分区，Join 的非倾斜侧被复制以匹配。

## 核心区别：SQL Metrics vs AQE 统计信息

|  | SQL Metrics | AQE 统计信息 |
| --- | --- | --- |
| **是什么** | SQLMetric 累加器 | MapOutputStatistics |
| **目的** | 可观测性（UI 显示） | 运行时计划优化 |
| **数据格式** | 格式化字符串 | 原始 Long[] 数组 |
| **计算时机** | 每个任务完成后 | Stage 所有 Map 任务完成后 |
| **消费者** | Spark UI、REST API、用户 | AQE 优化器规则 |

**SQL Metrics 告诉你发生了什么，AQE 统计信息决定接下来会发生什么。**

## 从指标中读取 AQE 的决策

`AQEShuffleReadExec` 是你了解 AQE 决策的窗口：

| 指标 | 含义 |
| --- | --- |
| numCoalescedPartitions > 0 | AQE 合并了小分区 |
| numSkewedPartitions > 0 | AQE 检测到了倾斜分区 |
| numSkewedSplits | 从倾斜分区创建了多少子分区 |
| numEmptyPartitions | 检测到的空分区数 |

> **实际示例：**如果你看到 numSkewedPartitions: 3、numSkewedSplits: 12，意味着 AQE 发现 3 个倾斜分区并拆分为 12 个子分区。3 个瓶颈任务变成 12 个并行任务。

> 如果 numCoalescedPartitions 和 numSkewedPartitions 都为零，说明 AQE 启用了但没有找到需要优化的内容。

## 通过 SQL 计划理解 AQE

对比初始计划和最终计划，可以看到 AQE 在哪里进行了干预：

```
# 查看初始计划（AQE 之前）  
spark-history-cli -a <app-id> sql-plan <id> --view initial  
  
# 查看最终计划（AQE 之后）  
spark-history-cli -a <app-id> sql-plan <id> --view final
```

* `ShuffleExchangeExec`

  → `AQEShuffleReadExec`：应用了 Shuffle 优化
* Join 策略改变：Sort Merge Join → Broadcast Hash Join
* 分区数变化：发生了合并或拆分

参考链接

[1] 博客原文 https://yaooqinn.github.io/zh/posts/spark/sql-metrics-part2-internals/

[2] 第一部分 https://yaooqinn.github.io/zh/posts/spark/understanding-sql-metrics/

[4] spark-history-cli https://github.com/yaooqinn/spark-history-cli