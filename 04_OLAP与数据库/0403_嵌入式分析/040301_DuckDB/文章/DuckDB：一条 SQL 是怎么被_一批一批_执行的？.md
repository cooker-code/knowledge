---
title: DuckDB：一条 SQL 是怎么被"一批一批"执行的？
author: MaxAiDB
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUwOTU4OTU2NQ==&mid=2247483679&idx=1&sn=8f6d7a6e8250950bc4342b1e857f299b&chksm=f8f9186d1aac9374b6c8c06149ffdaa1f553a35288c747e90ed2c7a642c3929f19270366dd97&mpshare=1&scene=24&srcid=0406Du2PXRGb7EV07CoC22gT&sharer_shareinfo=de4aa629413a75f642ae590dfedff8d9&sharer_shareinfo_first=de4aa629413a75f642ae590dfedff8d9#rd
---

**—— 一条 SQL 是怎么被"一批一批"执行的？**

> 本文是《DuckDB：从上手到内核》系列的第 6 篇。上一篇讲了"读数据快"（列式存储），这篇讲"算数据快"——向量化执行引擎。

---

## 搬砖的故事

假设你要把 10000 块砖从 A 搬到 B。

**方案一（逐行处理/火山模型）**：每次拿起一块砖，走到 B，放下，走回 A，再拿一块。来回 10000 趟。每趟的"走路时间"远大于"搬砖时间"。

**方案二（向量化执行）**：推一辆小车过去，一次装 2048 块砖，推到 B，卸完，回来再装。只需要 5 趟。

**方案三（编译执行/代码生成）**：直接修公路，用传送带从 A 连到 B。

DuckDB 选的是方案二——**向量化执行**。这是 DuckDB 快的第二层原因。

---

## 火山模型：传统数据库的做法

大部分传统数据库（PostgreSQL、MySQL、SQLite）用的是**火山模型（Volcano Model）**，也叫迭代器模型。它的核心接口是 `next()`——每个算子调用下游的 `next()` 获取一行数据，处理完再交给上游。

```
SELECT region, SUM(amount) FROM orders WHERE year = 2024 GROUP BY region;  
执行过程（火山模型）：  Aggregate.next()    → Filter.next()        → Scan.next()  → 返回第 1 行: {Alice, East, 2024, 500}      ← 判断 year=2024? 是 → 返回给 Aggregate    → Filter.next()        → Scan.next()  → 返回第 2 行: {Bob, West, 2023, 300}      ← 判断 year=2024? 否 → 丢弃，继续调 next()    → Filter.next()        → Scan.next()  → 返回第 3 行: ...    ... 重复 N 次
```

**问题在哪？**

1.**虚函数调用开销巨大**：每一行数据都要经历一次 `next()` 调用链。100 万行数据 = 100 万次虚函数调用。CPU 的分支预测器会疯掉。2.**CPU 缓存效率低**：每次只处理一行，数据在缓存中一闪而过，还没来得及复用就被驱逐了。3.**无法利用 SIMD**：SIMD（单指令多数据）指令可以一次对多个数据做同样的操作，但火山模型每次只给你一个数据，用不上。

---

## 向量化执行：一次处理一批

DuckDB 的做法不同——不是一次处理一行，而是一次处理 **2048 行**（一个"向量"）。

**图 1：火山模型 vs 向量化模型**

```
火山模型（一次一行）：                 向量化模型（一次 2048 行）：  
Aggregate                            Aggregate  │ next() → 1 行                      │ Execute() → 2048 行  ▼                                    ▼Filter                               Filter  │ next() → 1 行                      │ Execute() → 2048 行  ▼                                    ▼Scan                                 Scan  │ next() → 1 行                      │ GetData() → 2048 行  
调用次数：N 次                        调用次数：N/2048 次
```

函数调用次数从 N 降到了 N/2048——减少了三个数量级。

---

## DataChunk：DuckDB 的基本数据单元

DuckDB 的向量化数据结构叫 **DataChunk**（源码在 `src/include/duckdb/common/types/data_chunk.hpp`）。

**图 2：DataChunk 内存结构**

```
DataChunk (最多 2048 行)┌───────────────────────────────────────────────┐│ Vector 0 (region列):  [East, West, East, ...]  │  ← 连续内存│ Vector 1 (year列):    [2024, 2024, 2023, ...]  │  ← 连续内存│ Vector 2 (amount列):  [500, 300, 800, ...]     │  ← 连续内存│ count: 2048                                    │└───────────────────────────────────────────────┘
```

几个要点：

•每一列是一个 `Vector`，在内存中连续存放•一个 DataChunk 包含多个 Vector（有多少列就有多少个 Vector）•`STANDARD_VECTOR_SIZE = 2048`（编译时可配置，必须是 2 的幂）

**为什么是 2048？** 拿 `int32` 类型举例：2048 × 4 字节 = 8KB。加上其他几列的 Vector，一个 DataChunk 的总大小在 32KB~64KB 范围——刚好放进大多数 CPU 的 L1 数据缓存。缓存命中 = 快。

---

## Push-Based Pipeline

DuckDB 不仅是向量化的，还是 **Push-Based（推进式）** 的——这跟火山模型的 Pull-Based（拉取式）正好相反。

火山模型是"上游问下游要数据"（Pull）。DuckDB 是"下游主动把数据推给上游"（Push）。

**图 3：Pipeline 执行模型**

```
Pipeline 1:┌──────────┐    DataChunk     ┌──────────┐    DataChunk     ┌──────────┐│  Source   │ ──────────────► │ Operator │ ──────────────► │   Sink   ││(Parquet   │    (2048行)     │ (Filter) │    (2048行)     │ (HashAgg ││ Scan)     │                 │          │                 │  Build)  │└──────────┘                  └──────────┘                  └──────────┘  
数据流方向 →→→→→→→→→→→ (Push)
```

三种角色：

•**Source**：数据的生产者。调用 `GetData()` 产生 DataChunk。比如扫描表、读 Parquet。•**Operator**：数据的加工者。接收一个 DataChunk，处理后输出一个 DataChunk。比如 Filter、Projection。中间不做物化（不存中间结果）。•**Sink**：数据的收集者。接收 DataChunk 并积累结果。比如 Hash Aggregate 的"建哈希表"阶段、Sort 的"收集所有数据"阶段。

### 什么是 Pipeline Breaker？

不是所有算子都能直接"传递"DataChunk。有些算子必须先"收集完所有数据"才能开始输出——这些算子叫 **Pipeline Breaker**。

**图 4：Pipeline Breaker 导致 Pipeline 切分**

```
原始查询:  SELECT region, SUM(amount) FROM orders WHERE year=2024 GROUP BY region ORDER BY SUM(amount)  
Pipeline 1:                              Pipeline 2:  Scan → Filter → HashAgg(Build)          HashAgg(Scan) → Sort(Build)           ↓ Pipeline Breaker ↓                    ↓ Pipeline Breaker ↓         (聚合必须收集完所有数据                  Pipeline 3:          才能输出结果)                           Sort(Scan) → Output
```

常见的 Pipeline Breaker：

•**Hash Aggregate**：必须扫完所有数据才知道每个 group 的最终结果•**Sort**：必须拿到所有数据才能排序•**Hash Join 的 Build 端**：必须先建完整个哈希表，Probe 端才能开始

---

## 向量化 Filter 长什么样？

我们用一个简单例子看看向量化的 Filter 是怎么工作的。

`WHERE year = 2024`，传入一个 2048 行的 DataChunk：

```
输入 DataChunk:  year Vector:   [2024, 2023, 2024, 2025, 2024, 2023, ...]  (2048 个值)  region Vector: [East, West, East, North, West, East, ...]  amount Vector: [500,  300,  800,  200,  600,  100,  ...]  
向量化 Filter 操作:  1. 对 year Vector 做批量比较: year == 2024     → Selection Vector: [0, 2, 4, ...]  (记录哪些位置满足条件)  
  2. 用 Selection Vector 过滤所有 Vector     → 输出 DataChunk (只包含满足条件的行)  
输出 DataChunk:  year Vector:   [2024, 2024, 2024, ...]  region Vector: [East, East, West, ...]  amount Vector: [500,  800,  600,  ...]  count: 1200  (2048行中有1200行满足条件)
```

关键：**不是逐行判断，而是一次性对整个 Vector 做批量比较**。这种"对一整列数据做同一个操作"的模式，天然适合 SIMD 指令加速——CPU 可以一条指令同时比较 4 个（SSE）或 8 个（AVX2）整数。

---

## 动手验证：看 Pipeline 切分

```
-- 创建测试数据CREATE TABLE orders ASSELECT (i % 5)::VARCHAR AS region,        2020 + (i % 5) AS year,        (random()*1000)::INT AS amountFROM generate_series(1, 2000000) t(i);  
-- 查看带聚合+排序的执行计划EXPLAIN ANALYZESELECT region, SUM(amount) AS totalFROM ordersWHERE year = 2024GROUP BY regionORDER BY total DESC;
```

你会看到执行计划中有多个 Pipeline，被 Hash Aggregate 和 Sort 切分。每个 Pipeline 内部是连续的 DataChunk 传递，没有中间物化。

---

## 与 MonetDB/X100 的对比

DuckDB 的向量化引擎受 **MonetDB/X100**（后来叫 Vectorwise）论文启发，但做了重要的改变：

| 维度 | MonetDB/X100 | DuckDB |
| --- | --- | --- |
| 执行模型 | **Pull-Based** （拉取式） | **Push-Based** （推进式） |
| 向量大小 | 通常 1000 | **2048** （可配置） |
| 并行模型 | 向量级并行 | **Morsel-Driven** （数据块级） |
| 存储集成 | 独立存储层 | 深度集成列式存储 |
| 事务支持 | 有限 | **完整 ACID + MVCC** |

参考论文：**MonetDB/X100: Hyper-Pipelining Query Execution** (CIDR 2005)

Push-Based 的优势在于：编译器和 CPU 更容易优化紧凑的循环（`for (int i = 0; i < 2048; i++) { ... }`），而 Pull-Based 的 `next()` 虚函数调用会打断循环优化。

---

## 小结

1.**火山模型的问题**：每行一次虚函数调用，CPU 缓存低效，无法用 SIMD2.**向量化的核心**：一次处理 2048 行（一个 DataChunk），减少函数调用，利用缓存和 SIMD3.**DataChunk 结构**：多个 Vector（每列一个），连续内存布局4.**Push-Based Pipeline**：Source → Operator → Sink，数据主动推送5.**Pipeline Breaker**：Hash Aggregate、Sort 等必须收集完所有数据的算子，会切分 Pipeline

下一篇进入查询优化器——看看 DuckDB 的 35 个优化 Pass 是怎么让你的 SQL 自动变快的。

---

## 延伸阅读

•DuckDB 执行引擎源码可参阅 `src/execution/` 目录•MonetDB/X100 论文：**Hyper-Pipelining Query Execution** (CIDR 2005)•Morsel-Driven Parallelism 论文：(SIGMOD 2014)•DuckDB 向量大小常量：`src/include/duckdb/common/vector_size.hpp`