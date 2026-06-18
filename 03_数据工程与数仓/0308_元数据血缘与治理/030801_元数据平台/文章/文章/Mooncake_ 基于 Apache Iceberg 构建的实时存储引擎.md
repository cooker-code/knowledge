---
title: Mooncake: 基于 Apache Iceberg 构建的实时存储引擎
author: 过往记忆大数据
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=2650742043&idx=1&sn=089f37429370e6b498baa69424380e83&chksm=89bf661d79398ce79311a28f02a4ff2f6ec0c02fbe31602c1fd1b4eb925dd16535e489903256&mpshare=1&scene=24&srcid=1127dQ9Sy3SSBNHKx4GexJ9C&sharer_shareinfo=22a0c9a924a99cf041b330dc06d3007c&sharer_shareinfo_first=22a0c9a924a99cf041b330dc06d3007c#rd
---

Iceberg 是 “数据湖仓”（Lakehouse）技术趋势的核心组成部分，该架构能让用户实现：

* 以开源文件格式（Parquet 列式存储 + 元数据）在对象存储中存储大型数据集
* 通过 Spark、Trino、Flink、DuckDB 等引擎，直接用 SQL 查询数据
* 获得 “数据库级特性”，如模式演进（Schema Evolution）和时间旅行（Time-Travel）
* 近乎无限的扩展能力，且不被厂商锁定

Iceberg 最初为慢变更数据和批量分析处理而设计。这一领域的应用场景已相当成熟，且随着各类引擎对开源格式的读取能力不断优化，其表现仍在持续提升。

而实时工作负载（如面向用户的分析、流处理）通常需要单独的专用系统 —— 例如 ClickHouse、Druid、SingleStore 等 —— 这导致用户需维护两条数据管道，并手动在系统间迁移数据。

若将数据湖仓的开放性与灵活性延伸至实时场景，需满足两大核心要求：

1. 不牺牲数据湖仓的任何优势：包括生态集成能力、现有工具链兼容性
2. 提供实时数据摄入与快速查询能力，且不影响 Iceberg 的原有性能与数据状态

## Mooncake 介绍

Mooncake 是一款基于 Iceberg 构建的实时存储引擎，专为实时工作负载设计。它在不放弃数据湖仓开放性与灵活性的前提下，为 Iceberg 注入了高吞吐摄入、快速查询和索引能力：

* 高速摄入：支持流式写入，可亚秒级镜像关系型数据库数据
* 快速查询：通过智能缓存与持续表优化，实现低延迟查询响应
* 索引能力：支持低延迟搜索与主键查找

Mooncake 的架构设计、索引层与写感知优化（Write-aware Optimizations），使 Iceberg 升级为能支撑面向用户的分析、搜索、服务及混合事务 / 分析处理（HTAP）工作负载的平台 —— 这些场景此前均需专用实时系统才能实现。

# 架构设计

Mooncake 的核心组件是 Moonlink—— 一款专为实时变更数据捕获（CDC）和流式写入 Iceberg 设计的 Rust 语言库。

Moonlink 为 Iceberg 表扩展了三大核心能力：内存中的 Arrow 缓冲区、位置删除日志（Positional Delete Log），以及内存 / NVMe / 对象存储混合部署的索引，从而能够承接高频更新。查询引擎通过读取 “缓冲区 + Iceberg 数据” 的联合视图获取完全最新的数据 —— 无需等待 Iceberg 元数据提交。

## 数据写入（Inserts）

1. 数据行先写入 Arrow 缓冲区；当积累到足够行数时，高效刷新为 Parquet 格式
2. 插入过程中同步构建内存索引，Parquet 刷新时，索引会以原生对象存储格式持久化

## 数据删除（Deletions）

1. Moonlink 为所有数据行维护主键索引
2. 删除操作通过主键索引定位记录位置，并更新位置删除日志
3. 位置删除日志会定期转换为 Iceberg v3 标准的删除向量（Deletion Vectors）

## 数据读取（Reads）

1. Moonlink 写入的 Iceberg 数据可被任意兼容引擎读取
2. 提供联合读取接口（Union-read Interface），融合内存状态与 Iceberg 文件，确保引擎获取最新表状态
3. （开发中）提供索引查询接口（Index-read Interface），支持基于索引的键值查找与搜索

# 核心技术深度解析

## 3.1 写感知表优化（Write-aware Table Optimization）

### 最新版本联合读取

Iceberg 在数据新鲜度与写放大（Write-amplification）之间存在固有权衡：若要获得更实时的数据，需频繁写入快照，这会产生大量小体积 Parquet 文件和元数据 / 清单文件（Manifest Files）；而压缩合并（Compaction）操作会进一步增加快照数量，且 Iceberg 规范难以高效清理中间版本和回收未使用文件。

Mooncake 的解决方案：定期写入 Iceberg 元数据，同时提供兼容 Iceberg 的 API，支持在元数据提交前 “读取最新版本”—— 通过动态生成最新元数据和数据文件，实现亚秒级表数据新鲜度。

### 支持更新 / 删除的索引 + 删除向量

Iceberg 并非为高吞吐更新 / 删除（CDC 场景）或增量写入（Upserts，流处理场景）优化。传统方案要么依赖昂贵的全局合并批量处理，要么使用等值删除（Equality Deletes），这会影响读取性能并产生大量小文件。

Moonlink 维护覆盖内存缓冲区和 Iceberg 数据的全局索引，用于定位删除记录位置，再通过 Iceberg v3 删除向量执行删除操作，高效支撑高频更新场景。

### 优雅处理写入与压缩冲突

开源表格式对冲突提交的处理较为简单 —— 流式更新常与后台维护操作（排序 / 压缩）发生冲突。Mooncake 在维护过程中构建 “旧文件到新文件” 的映射结构，并对正在执行的更新 / 删除操作进行重新映射，确保事务无失败。

## 3.2 原生 Iceberg 索引

### 索引存储

* 索引文件元数据以 Iceberg Puffin 文件格式存储
* 索引文件与表数据同目录或邻近存储
* 单个索引文件覆盖一个或多个数据文件
* 后台合并操作构建索引的 LSM（日志结构合并树）架构

### 查询接口与集成

* 索引通过（文件 ID，偏移量）指向数据，无需额外主键
* 核心 API：

+ 构建：Build (DataFile) -> IndexFile（从数据文件生成索引文件）
+ 合并：Merge ([IndexFiles]) -> IndexFile (s)（合并多个索引文件）
+ 查询：Query (IndexFile, query) -> [(DataFile, offset, [可选负载])]（通过索引查询数据位置）

### 索引实现

1. 桶式哈希表（Bucket Hash Table）：高基数哈希索引，用于将原始删除请求转换为位置删除。采用不可变、哈希有序的类 SSTable 格式，将 64 位哈希映射到存储位置；通过高位比特分区，空间效率高、缓存友好，支持快速流式合并。
2. 桶式哈希表 + 倒排索引：低基数哈希索引，支撑 GIN（通用倒排索引）/ 全文搜索（每个数据文件对应一个倒排索引；可合并的桶表映射倒排索引偏移量）。
3. 向量索引（Vector Index）：按数据文件构建，支持跨文件合并；提供聚类 / IVF（倒排文件索引）变体，适配原生对象存储性能特性。

## 3.3 托管缓冲区与缓存

Mooncake 是原生对象存储架构，可高效利用本地内存与 NVMe（存储级闪存），设计理念与 SlateDB、TurboPuffer、WarpStream 相似。

### 原生对象存储预写日志（WAL）

对象存储是数据的事实来源（权威存储）。Mooncake 节点大多为无状态设计：直接将预写日志（WAL）写入对象存储，恢复时通过日志重建状态 —— 无需多副本 Raft/Paxos 分布式一致性集群。可将 Mooncake 理解为 “以 Iceberg 快照形式存储自身快照的数据库”。

### 流式写入 Arrow 缓冲区

为实现低延迟写入，最新数据会在写入 Parquet 前存入缓冲区。内存中的 Arrow 缓冲区支持行追加与字段修改，可直接刷新到 Parquet/Iceberg 格式，还能通过网络向实时引擎传输未压缩的 Arrow/Parquet 数据。结合缓冲区与索引，单台机器可处理约 100 万次更新操作。

### 读写穿透缓存

缓存对性能提升与 S3 成本控制至关重要。新生成的 Parquet 文件、删除向量及索引会先存储在 NVMe 中，再异步上传至对象存储，使本地读写延迟控制在 5 毫秒以内。

# 部署与解决方案

## 4.1 pg\_mooncake

在 Postgres 数据库内部实现 HTAP、实时分析及与 Iceberg 的同步。作为 Postgres 扩展插件，它通过后台进程运行 Moonlink，并嵌入 DuckDB 作为执行引擎（采用 MIT 开源许可证）。执行 `mooncake.create_table('T_mooncake', 'T_heap')` 即可创建 Postgres 堆表（T\_heap）的列存版本（T\_mooncake），两者保持亚秒级延迟同步且具备强一致性。

## 4.2 独立版 Moonlink（Alpha 测试版）

实现从 Postgres 到 Iceberg 的快速、轻量数据复制。作为无状态 Rust 引擎，它读取 Postgres 逻辑复制流并直接写入 Iceberg，维持查询就绪状态的表，延迟低于 1 秒且资源开销极低。

## 4.3 Mooncake（托管平台 —— 预计 2025 年底发布预览版）

面向实时与混合工作负载的 Iceberg 解决方案，融合了 Iceberg 的灵活性与搜索 / 流数据库的核心能力。

### 写入方式

* 从动态变化的联机事务处理（OLTP）系统镜像数据
* 通过事件 API / Kafka / OpenTelemetry 流进行流式摄入
* 支持并发批量加载

### 读取方式

* 支持任意引擎读取 Iceberg 快照（数据新鲜度达分钟级）
* 实时读取（数据新鲜度低于 1 秒，具备强一致性）
* 通过 Mooncake 索引查询 API 实现搜索与查找

# 核心总结

Mooncake 支持将传统上需要专用数据库（如搜索场景的 Elasticsearch、分析场景的 ClickHouse、AI 场景的向量数据库）处理的工作负载，直接在 Iceberg 数据湖上运行 —— 从而在单一存储层实现实时工作负载与分析工作负载的统一。