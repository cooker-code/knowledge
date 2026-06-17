---
title: DuckDB：并行执行与 Morsel-Driven 调度-多核 CPU 是怎么被充分利用的？
author: MaxAiDB
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUwOTU4OTU2NQ==&mid=2247483714&idx=1&sn=9d5e1ceaa28584080f58101b47943f61&chksm=f83cb596d2a216fd87a5e90df58397af1fee3c5fcd59abbafc4749d677cd11c3bfbe81b4d10f&mpshare=1&scene=24&srcid=0410BQ7ypVPOavBTLuCiyQr2&sharer_shareinfo=d79d36309a2868a1325257cc985ce9b3&sharer_shareinfo_first=d79d36309a2868a1325257cc985ce9b3#rd
---

—— 多核 CPU 是怎么被充分利用的？

> 本文是《DuckDB：从上手到内核》系列的第 10 篇。前面我们讲了 DuckDB 怎么存数据（列式存储）、怎么算数据（向量化执行）、怎么优化查询（优化器）。但还有一个关键问题没回答：**你的 Mac 有 10 个核，DuckDB 是怎么把它们全部用上的？**

---

## 为什么数据库并行化很难？

你可能觉得：并行化不就是"多开几个线程"吗？没那么简单。

假设你有一条 `GROUP BY` 查询。4 个线程同时扫描数据，然后同时往一个哈希表里插数据——如果两个线程恰好要往同一个位置写，数据就乱了。加锁？锁太多线程全在等锁，还不如单线程快。

数据库并行化有三个经典难题：

1.**数据依赖**：聚合、排序需要看到全部数据才能出结果——不能简单拆分2.**同步开销**：线程之间的协调（锁、原子操作、屏障）吃掉了并行带来的收益3.**负载不均**：某些线程分到的数据多、某些少，快的线程干完了只能等慢的

DuckDB 用 **Morsel-Driven Parallelism**（基于 Morsel 的并行机制）解决了这些问题。

---

## Morsel-Driven：工厂流水线的思路

### 什么是 Morsel？

Morsel 这个词本意是"一小口食物"。在 DuckDB 中，**一个 Morsel 就是一个 DataChunk——2048 行数据**。

打个比方。想象一个巧克力工厂的流水线：

•原料仓库（Source）里有几百万颗可可豆•工人（线程）每次从仓库领一小筐（Morsel = 2048 颗）•拿着这筐经过碾碎机（Filter）→ 搅拌机（Projection）→ 包装机（Aggregate）•处理完放到成品区（Sink），然后回去再领一筐•所有工人都在同一条流水线上，各自独立，互不干扰

```
    ┌──────────────────────────────────────────┐    │              数据源 (Source)               │    │        [RowGroup₁] [RowGroup₂] ...        │    │         122880行     122880行              │    └───┬──────────┬──────────┬────────────────┘        │          │          │   ┌────▼────┐┌───▼────┐┌───▼────┐   │ 线程 1  ││ 线程 2 ││ 线程 3 │  ← 每次取 2048 行   │ morsel  ││ morsel ││ morsel │   └────┬────┘└───┬────┘└───┬────┘        │          │          │        ▼          ▼          ▼    ┌───────────────────────────────┐    │   算子链: Filter → Proj → ...  │  ← 各线程独立处理    └───────────────────────────────┘        │          │          │        ▼          ▼          ▼    ┌──────────────────────────────┐    │        Sink (汇聚结果)        │  ← 最后合并    └──────────────────────────────┘  
    图 1：Morsel-Driven 并行执行示意图
```

关键设计：

•**每个线程独立工作**：领一个 Morsel，处理完再领下一个，不需要和其他线程商量•**动态分配**：不是预先把数据平均分给每个线程，而是"谁干完了谁再取"——自动负载均衡•**同步点极少**：只在"领取 Morsel"和"合并结果"时需要同步

### 和 MapReduce 有什么不同？

| 维度 | MapReduce | Morsel-Driven |
| --- | --- | --- |
| 粒度 | 按文件/分区分任务（粗粒度） | 按 2048 行分任务（细粒度） |
| Shuffle | 需要跨节点 Shuffle | 单进程内，不需要 Shuffle |
| 负载均衡 | 静态分配，可能不均 | 动态领取，自动均衡 |
| 适用场景 | 分布式集群 | 单机多核 |

---

## DuckDB 的并行架构

### Pipeline：并行的基本单位

在第 6 篇我们讲过，DuckDB 的查询执行是以 **Pipeline** 为单位的。一个 Pipeline 就是一段可以不中断地流式处理数据的算子链：

```
Source → Operator₁ → Operator₂ → ... → Sink
```

比如一个带聚合的查询：

```
SELECT category, SUM(amount) FROM orders WHERE year = 2024 GROUP BY category;
```

会被切分成两个 Pipeline：

```
Pipeline 1: TableScan(orders) → Filter(year=2024) → HashAggregate(Build)                                                          ↓ BreakerPipeline 2: HashAggregate(Scan) → ResultCollector  
图 2：Pipeline 切分示意图
```

**Pipeline Breaker**（HashAggregate 的 Build 阶段）把查询切分成了两段。Pipeline 1 必须完全执行完（所有数据都聚合好），Pipeline 2 才能开始扫描聚合结果。

### 两种并行维度

DuckDB 同时利用了两种并行方式：

```
  Intra-operator 并行                Inter-operator 并行  (同一算子内并行)                   (不同 Pipeline 间并行)  
  ┌────────────────────┐            Pipeline A ──→ Pipeline C  │    TableScan       │            Pipeline B ──→ Pipeline C  │  T1  T2  T3  T4   │  │  │   │   │   │    │            A 和 B 没有依赖关系  │  ▼   ▼   ▼   ▼    │            → 可以同时执行  │ 不同 morsel 并行处理 │  └────────────────────┘            C 依赖 A 和 B                                    → 必须等 A、B 都完成  
  图 3：Intra-operator vs Inter-operator 并行对比
```

•**Intra-operator 并行**：同一个 Pipeline 内，多个线程处理不同的 Morsel。这是最常见的并行方式。•**Inter-operator 并行**：不同 Pipeline 没有依赖关系时，可以同时执行。比如两个独立子查询可以并行跑。

### TaskScheduler：全局任务调度器

DuckDB 的所有并行任务都通过一个全局的 `TaskScheduler`（`src/parallel/task_scheduler.cpp`）来协调。它的核心组件：

```
TaskScheduler├── ConcurrentQueue     ← 无锁并发任务队列（基于 moodycamel::ConcurrentQueue）├── LightweightSemaphore ← 轻量级信号量，用于唤醒空闲线程├── WorkerThreads[]     ← 后台工作线程池└── 线程 Pin 机制       ← CPU 数 > 64 时绑定线程到核心
```

工作线程的主循环非常简洁：

```
while (running) {    1. 等信号量（超时 5ms）    2. 从队列取一个 Task    3. 执行 Task    4. 如果 Task 没完成 → 放回队列    5. 如果 Task 完成 → 释放    6. 如果 Task 阻塞 → 暂时移出调度}
```

注意第 4 步——**Task 可以被中途打断**。当启用了 `PROCESS_PARTIAL` 模式时，每个 Task 处理 50 个 Chunk 后就会主动让出，重新放回队列。这实现了多个查询之间的公平调度——不会有一个大查询把所有线程占满，让小查询一直等。

### 并行度怎么决定？

一个 Pipeline 到底用几个线程？DuckDB 会综合考虑三个因素：

```
实际并行度 = min(    Source 能支持的最大线程数,     -- 比如：总行数 / 122880    所有中间算子限制的最大线程数,  -- 某些算子可能要求单线程    Sink 限制的最大线程数,        -- Sink 也有并行约束    系统配置的总线程数            -- PRAGMA threads = N)
```

以表扫描为例：如果一张表有 1000 万行，Row Group 大小是 122880 行，那 Source 的 `MaxThreads()` = 1000 万 / 122880 ≈ 82。如果你的 CPU 有 10 个核、配置了 `PRAGMA threads = 10`，那实际并行度就是 min(82, 10) = 10。

---

## 事件驱动：Pipeline 之间的协调

多个 Pipeline 之间有依赖关系——Pipeline 2 必须等 Pipeline 1 完成。DuckDB 用 **事件（Event）** 机制来管理这些依赖：

```
Pipeline 1:  InitializeEvent → PipelineEvent → FinishEvent → CompleteEvent                                                        │                                                        ▼ 触发Pipeline 2:  InitializeEvent → PipelineEvent → FinishEvent → CompleteEvent
```

每个 Pipeline 的执行分为五个事件阶段：

1.**Initialize**：初始化全局状态2.**Pipeline**：实际执行（多线程并行处理 Morsel）3.**PrepareFinish**：准备合并4.**Finish**：各线程的本地状态合并到全局状态5.**Complete**：清理资源，触发下游 Pipeline

这些事件形成一个 **DAG（有向无环图）**——`Executor` 会找出所有没有前驱依赖的事件先执行，完成后自动触发后续事件。

---

## 动手实验：看线程数对性能的影响

### 实验 1：控制线程数

```
-- 建一张大表CREATE TABLE big_data AS SELECT range AS id, random() * 1000 AS value, range % 100 AS group_id FROM range(50000000);  
-- 单线程SET threads = 1;.timer on  
SELECT group_id, SUM(value), AVG(value), COUNT(*) FROM big_data GROUP BY group_id;-- 记录时间  
-- 多线程SET threads = 8;SELECT group_id, SUM(value), AVG(value), COUNT(*) FROM big_data GROUP BY group_id;-- 记录时间  
SET threads = 1;SELECT group_id, SUM(value), AVG(value), COUNT(*) FROM big_data GROUP BY group_id;
```

在我的测试中（M2 MacBook Pro, 16GB RAM, 5000 万行）：

| 线程数 | 执行时间 | 相对加速比 |
| --- | --- | --- |
| 1 | ~2.1s | 1.0x |
| 2 | ~1.2s | 1.75x |
| 4 | ~0.7s | 3.0x |
| 8 | ~0.45s | 4.7x |

加速比不是线性的——因为存在同步开销和内存带宽限制。但 8 线程仍然比单线程快了近 5 倍。

### 实验 2：观察 Pipeline 切分

```
SET threads = 4;  
EXPLAIN ANALYZESELECT p.name, SUM(o.amount)FROM orders o JOIN products p ON o.product_id = p.idWHERE o.year = 2024GROUP BY p.nameORDER BY SUM(o.amount) DESCLIMIT 10;
```

这个查询会被切分成多个 Pipeline：

1.**Pipeline 1**：扫描 products → 建 Hash Join 的 Build 端（Pipeline Breaker）2.**Pipeline 2**：扫描 orders → Filter → Hash Join Probe → Hash Aggregate Build（Pipeline Breaker）3.**Pipeline 3**：Aggregate Scan → Sort Build（Pipeline Breaker）4.**Pipeline 4**：Sort Scan → TopN → Result

在 `EXPLAIN ANALYZE` 的输出中，你可以看到每个算子的线程数和处理行数。

---

## 几个有意思的实现细节

### 为什么 Morsel 是 2048 行？

DuckDB 的 `STANDARD_VECTOR_SIZE` 固定为 2048（`src/include/duckdb/common/vector_size.hpp`）。这个数字的选择平衡了三个因素：

1.**CPU L1 缓存**：2048 个 8 字节的值 = 16KB，正好能放进 32KB 的 L1 数据缓存2.**SIMD 友好**：2048 是 2 的幂次，方便 SIMD 指令对齐3.**调度粒度**：太大会导致负载不均，太小会增加调度开销

### Row Group 和 Morsel 的关系

一个 Row Group 有 122880 行 = 60 个 Morsel（122880 / 2048 = 60）。表扫描时，一个线程通常一次领取一个 Row Group 的所有 Morsel，这样可以充分利用局部性。

### 线程 Pinning

当 CPU 核心数超过 64 时（大型服务器），DuckDB 会把线程绑定（Pin）到特定的 CPU 核心。这避免了线程在不同核心之间迁移导致的缓存失效——在 NUMA 架构上尤其重要。

---

## 小结

1.**Morsel-Driven Parallelism** 是 DuckDB 的并行执行模型——每个线程独立领取 2048 行的 Morsel，处理完再取下一个。2.**动态任务分配**实现了自动负载均衡——干得快的线程多处理、干得慢的少处理。3.**Pipeline** 是并行的基本单位，被 Pipeline Breaker（聚合、排序、Hash Join Build）切分。4.**TaskScheduler** 使用无锁并发队列分发任务，5ms 超时信号量唤醒空闲线程。5.**并行度** = min(Source 限制, 算子限制, Sink 限制, 配置线程数)。6.用 `SET threads = N` 可以控制线程数，8 核 CPU 上通常能获得 4-5 倍的加速比。

---

**Q1：什么是 Pipeline Breaker？为什么聚合和排序会"打断" Pipeline？**

因为聚合和排序需要**看到全部数据**才能给出结果。想象你要对一百万个数字求平均——你必须把所有数字加完再除，不能"流式地"边加边输出。这就是"打断"的含义：必须等上游全部数据到齐，才能开始处理。

和它相反的是 Filter（过滤）——每来一行就可以判断要不要保留，不需要等全部数据。所以 Filter 不会打断 Pipeline。

**Q2：`SET threads = 1` 和不设置有什么区别？DuckDB 默认用几个线程？**

DuckDB 默认使用你 CPU 的所有核心数。比如 M2 MacBook Pro 有 8 个核（实际是 4 性能核 + 4 能效核），DuckDB 默认就用 8 个线程。

`SET threads = 1` 强制单线程执行——这在调试、对比性能、或者想减少 CPU 占用时有用。

**Q3：为什么 8 线程只快了 4.7 倍，不是 8 倍？**

理论上的"线性加速"在现实中几乎不可能实现。原因有几个：

•线程之间有同步开销（取 Morsel、合并结果）•内存带宽是共享的——8 个线程同时读数据，总带宽没有变成 8 倍•聚合的 Combine 阶段是串行的•M2 的 4 个能效核比 4 个性能核慢

一般来说，能达到线程数 60-70% 的加速比就已经很好了。

**Q4：DuckDB 能跨多台机器并行吗？**

不能。DuckDB 是单进程、单机数据库——所有并行都在一台机器的多个 CPU 核心上。如果你的数据大到一台机器放不下，应该考虑 Spark、ClickHouse、DuckDB-Wasm + 分片等方案。

DuckDB 的定位就是"单机分析的极致"——把一台机器的能力榨干。

---

## 延伸阅读

**源码路径：**

•TaskScheduler：`src/parallel/task_scheduler.cpp`•Pipeline：`src/parallel/pipeline.cpp`•PipelineExecutor：`src/parallel/pipeline_executor.cpp`•Executor（顶层编排）：`src/parallel/executor.cpp`•MetaPipeline：`src/parallel/meta_pipeline.cpp`•向量大小定义：`src/include/duckdb/common/vector_size.hpp`

**参考资料：**

•论文原文：「Morsel-Driven Parallelism: A NUMA-Aware Query Evaluation Framework for the Many-Core Age」, Viktor Leis et al., SIGMOD 2014•DuckDB 官方文档 - 配置：https://duckdb.org/docs/configuration/overview[1]•moodycamel::ConcurrentQueue：https://github.com/cameron314/concurrentqueue[2]

### References

`[1]`: *https://duckdb.org/docs/configuration/overview*  
`[2]`: *https://github.com/cameron314/concurrentqueue*