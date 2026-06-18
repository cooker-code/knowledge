---
title: 美团面试：Paimon LSM-Tree 分层机制是怎么样的？
author: 大数据技能圈
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247492183&idx=1&sn=9262dd3bbeaa1cc5d6c31bcd3f0e55e2&chksm=c1eaaf081ff4f8dd445125734388674193cf4ef7254f00a4af1fe2ec4f12498573d5bed19cfc&mpshare=1&scene=24&srcid=1216oifkU4VMs0MJ5JEuLmMp&sharer_shareinfo=c4f6377629b9709ef4fbfe0b65ec1948&sharer_shareinfo_first=c4f6377629b9709ef4fbfe0b65ec1948#rd
---

## 一、引言

Paimon是专为流批一体化设计的数据湖存储系统，它基于 LSM-Tree（Log-Structured Merge-Tree）架构，提供了高效的读写性能和强大的数据管理能力。Paimon 文件系统不仅支持高吞吐量的数据写入，还能保证数据的强一致性和事务特性，这使其成为构建实时数据湖的理想选择。

## 二、Paimon 文件系统整体架构

Paimon 文件系统采用分层设计，从上到下主要包含四个核心层次：Snapshot 快照层、Manifest 清单层、LSM-Tree 索引层和 Data File 数据文件层。这种分层架构实现了元数据与数据的分离管理，提供了高效的版本控制和数据组织能力。

如上图所示，整个架构呈现清晰的层次关系：

* **Snapshot 层**：负责版本管理和时间旅行功能，每个快照记录某一时刻的完整表状态
* **Manifest 层**：管理文件变更清单，追踪数据文件的增删改操作
* **LSM-Tree 索引层**：组织数据文件的逻辑结构，通过多层分级优化读写性能
* **Data File 层**：实际的 SST（Sorted String Table）数据文件物理存储

## 三、LSM-Tree 分层机制深度解析

LSM-Tree 是 Paimon 文件系统的核心数据结构，它通过多层级的文件组织方式，在写入性能和读取效率之间找到了最佳平衡点。下面我们详细剖析每一层的特性和作用。

### 3.1 Level 0：未排序层（写入缓冲层）

**核心特征：**

* **数据组织方式**：Level 0 是新数据的直接落地层，文件之间允许键范围重叠
* **写入机制**：数据批次（batch）写入时，直接生成新的 SST 文件，无需考虑与已有文件的键范围关系
* **文件数量限制**：通常配置为 5-10 个文件，通过参数 `num-sorted-run.compaction-trigger` 控制
* **文件大小**：相对较小，通常在几 MB 到几十 MB 之间

**为什么允许重叠？**

Level 0 设计为未排序层的主要原因是优化写入性能。如果要求新写入的数据立即按键范围排序并与现有文件合并，会导致每次写入都需要大量的排序和合并操作，严重影响写入吞吐量。通过允许文件重叠，Paimon 实现了近乎追加式的快速写入。

**配置示例：**

```
CREATE TABLE my_table ( 

    id BIGINT, 

    name STRING, 

    PRIMARY KEY (id) NOT ENFORCED 

) WITH ( 

    'num-sorted-run.compaction-trigger' = '5',  -- Level 0 文件数达到 5 个时触发合并 

    'write-buffer-size' = '128mb'               -- 写入缓冲区大小 

);
```

### 3.2 Level 1：第一排序层

**核心特征：**

* **数据组织方式**：Level 1 开始实行严格的键范围分区，文件之间键范围完全不重叠
* **文件大小**：约 10MB（可通过 `target-file-size` 配置）
* **排序保证**：每个文件内部键值有序，文件之间也按键范围排序
* **合并来源**：主要由 Level 0 的文件通过 compaction 合并而来

**从 Level 0 到 Level 1 的转换：**

如图所示，Level 0 中三个存在键范围重叠的文件（文件 A: key 1-100，文件 B: key 50-150，文件 C: key 80-200），经过合并排序后，生成 Level 1 中两个键范围不重叠的有序文件（文件 1: key 1-100，文件 2: key 101-200）。这个过程不仅消除了重叠，还实现了数据的全局排序。

### 3.3 Level 2-5：递增存储层

**层级递增规律：**

从 Level 2 开始，每一层的文件大小呈指数级增长（默认 10 倍增长因子）：

* **Level 2**：约 100MB（10 × Level 1）
* **Level 3**：约 1GB（10 × Level 2）
* **Level 4**：约 10GB（10 × Level 3）
* **Level 5**：约 100GB（10 × Level 4）

**设计原理：**

这种指数增长的设计基于以下考虑：

1. **写放大控制**：较低层级的小文件可以更频繁地合并，而高层级的大文件合并频率较低，降低整体写放大
2. **空间利用率**：随着数据沉淀到更深层级，文件规模增大，减少文件数量，降低元数据开销
3. **读取优化**：查询时通过布隆过滤器和索引，可以快速定位目标文件，减少扫描范围

**层级间的容量关系：**

每一层的总容量限制通常遵循以下规则：

```
Level N 总容量限制 = Level (N-1) 总容量 × 放大因子
```

例如，如果 Level 1 总容量限制为 100MB，放大因子为 10：

* Level 2 容量限制：1GB
* Level 3 容量限制：10GB
* Level 4 容量限制：100GB
* Level 5 容量限制：1TB

### 3.4 最大层级配置

**默认层级设置：**

Paimon 默认支持 **Level 0 到 Level 5**，共 6 个层级。这个配置适合大多数应用场景，能够处理 TB 级别的数据量。

**可扩展性：**

通过配置参数 `num-levels`，Paimon 支持扩展到更多层级，理论上可以支持到 Level 7 或更高。但需要注意：

* **优点**：更多层级可以存储更大规模的数据
* **缺点**：读取时可能需要查询更多层级，增加读放大；合并操作链路更长，增加维护复杂度

**配置示例：**

```
CREATE TABLE large_table ( 

    id BIGINT, 

    data STRING, 

    PRIMARY KEY (id) NOT ENFORCED 

) WITH ( 

    'num-levels' = '7',                    -- 扩展到 7 层 

    'compaction.max-file-num' = '50',      -- 每层最大文件数 

    'target-file-size' = '128mb'           -- 目标文件大小 

);
```

**选择合适的层级数：**

* 数据量 < 1TB：Level 0-5（默认）足够
* 数据量 1TB-10TB：可考虑扩展到 Level 6
* 数据量 > 10TB：可扩展到 Level 7，但需评估读写性能

## 四、Compaction 合并机制详解

Compaction 是 LSM-Tree 维护数据组织和性能的核心机制。它负责将上层的小文件合并成下层的大文件，同时删除过期数据和冗余版本。

### 4.1 触发条件

不同层级的合并触发条件不同：

**Level 0 → Level 1 合并：**

* 触发条件：Level 0 文件数 ≥ `num-sorted-run.compaction-trigger`（默认 5）
* 合并策略：将所有 Level 0 文件与 Level 1 重叠范围的文件一起合并
* 处理逻辑：排序、去重、删除墓碑标记

**Level N → Level N+1 合并：**

* 触发条件：Level N 总大小超过容量限制
* 选择策略：选择 Level N 中一个或多个文件，与 Level N+1 中键范围重叠的文件合并
* 优化目标：最小化写放大和空间放大

### 4.2 合并类型

**Minor Compaction（小合并）：**

* 作用于层内：将同一层内的小文件合并成大文件
* 目的：减少文件碎片，优化文件大小分布
* 频率：较高，后台持续执行

**Major Compaction（大合并）：**

* 作用于跨层：将上层文件合并到下层
* 目的：彻底清理过期数据，优化存储结构
* 频率：较低，可配置定期执行

### 4.3 合并策略优化

Paimon 提供多种合并策略：

1. **Universal Compaction**：适合写多读少场景，优先控制空间放大
2. **Level Compaction**：默认策略，平衡读写性能
3. **Sorted Compaction**：针对有序写入优化

**配置示例：**

```
CREATE TABLE optimized_table ( 

    ts BIGINT, 

    value DOUBLE, 

    PRIMARY KEY (ts) NOT ENFORCED 

) WITH ( 

    'compaction.optimization-strategy' = 'level',      -- 合并策略 

    'compaction.min-file-num' = '3',                   -- 最小合并文件数 

    'compaction.max-file-num' = '10',                  -- 最大合并文件数 

    'full-compaction.delta-commits' = '3'              -- 全量合并触发间隔 

);
```

## 五、Snapshot 快照层：版本管理的基石

Snapshot 层是 Paimon 实现 ACID 特性和时间旅行功能的关键组件。每个快照包含：

* **Snapshot ID**：唯一标识，递增生成
* **Timestamp**：快照创建时间戳
* **Manifest List**：指向该版本的清单列表
* **Schema Version**：表结构版本信息

**核心功能：**

1. **MVCC（多版本并发控制）**：读操作访问特定快照，写操作创建新快照，实现读写隔离
2. **时间旅行**：查询历史版本数据，支持增量读取和回溯分析
3. **数据恢复**：快照保留策略支持数据回滚

**快照保留策略：**

```
CREATE TABLE versioned_table ( 

    id BIGINT, 

    data STRING, 

    PRIMARY KEY (id) NOT ENFORCED 

) WITH ( 

    'snapshot.time-retained' = '24h',          -- 保留 24 小时的快照 

    'snapshot.num-retained.min' = '10',        -- 最少保留 10 个快照 

    'snapshot.num-retained.max' = '100'        -- 最多保留 100 个快照 

);
```

## 六、Manifest 清单层：元数据管理中枢

Manifest 层记录每次数据变更的文件级别信息，包括：

* **新增文件**：新写入或合并产生的文件
* **删除文件**：被合并或清理的文件
* **文件统计**：行数、文件大小、键范围、最小/最大值等

**Manifest List**：

* 聚合多个 manifest 文件，减少元数据读取开销
* 支持增量更新，避免每次都扫描全量文件

**优化机制：**

* Manifest 文件定期合并，防止元数据膨胀
* 布隆过滤器加速文件过滤，减少 I/O

## 七、数据文件层：物理存储实现

数据文件层采用 SST（Sorted String Table）格式存储，特点包括：

**存储格式：**

* **Parquet/ORC**：列式存储格式，支持高效压缩和列裁剪
* **数据块**：按块组织，每块包含索引和数据
* **压缩算法**：支持 SNAPPY、ZSTD、LZ4 等

**文件组织：**

* **Bucket 分桶**：按哈希或范围分区，支持并行读写
* **文件命名**：包含层级、序列号等信息，便于管理

**配置示例：**

```
CREATE TABLE storage_optimized_table ( 

    id BIGINT, 

    name STRING, 

    value DOUBLE, 

    PRIMARY KEY (id) NOT ENFORCED 

) WITH ( 

    'file.format' = 'parquet',                    -- 文件格式 

    'file.compression' = 'zstd',                  -- 压缩算法 

    'file.compression.zstd-level' = '3',          -- 压缩级别 

    'bucket' = '16'                               -- 分桶数量 

);
```

## 八、总结：Paimon 文件系统的技术优势

Paimon 通过精心设计的 LSM-Tree 分层架构，实现了以下核心优势：

1. **高写入吞吐量**：Level 0 未排序层设计，支持快速追加写入，适合流式数据场景
2. **优秀的读性能**：多层索引、布隆过滤器、统计信息加速查询过滤
3. **可控的空间放大**：通过 compaction 机制定期清理冗余数据，维护存储效率
4. **灵活的层级配置**：默认 6 层（Level 0-5）适配多数场景，支持扩展到更多层级处理海量数据
5. **完整的 ACID 支持**：基于快照的 MVCC 实现强一致性和事务隔离
6. **流批一体化**：统一的存储格式支持实时写入和批量查询

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

本学习文档已放入星球，扫描二维码加入星球获取