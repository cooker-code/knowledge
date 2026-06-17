# 存储引擎

## 知识点入口

- 本模块先看宏观流程，再看文章：[流程化知识点总览](knowledge/04_OLAP与数据库/0406_存储引擎/核心知识点/流程化知识点总览.md)。
- 新文章必须先判断它讲的是“存储引擎机制”，还是上层数据库、缓存或业务系统的普通使用。
- `文章/` 只作为临时入口；正式归档必须落到 `LSM-Tree/`、`RocksDB/`、`LevelDB/`、`FlowDB/` 或后续新增技术目录。

## 类目定位

| 项 | 内容 |
|---|---|
| 一级类目 | OLAP 与数据库 |
| 二级类目 | 存储引擎 |
| 核心问题 | 数据库或 KV 系统底层如何组织写入、内存表、WAL、SSTable、索引、Compaction、缓存和恢复 |
| 不解决什么 | MySQL/PostgreSQL 普通 SQL 调优、Redis 应用缓存策略、OLAP 引擎完整架构，除非文章主问题是底层存储机制 |
| 用户当前认知假设 | 用户需要把 LSM、RocksDB、LevelDB 和具体数据库的存储层关系理清，重点补写放大、读放大、空间放大和 Compaction 代价 |

## 划分准则

| 保留条件 | 归入位置 | 示例判断 |
|---|---|---|
| 讲 LSM-Tree 抽象结构、WAL、MemTable、SSTable、Compaction 策略 | `LSM-Tree/` | LSM 数据结构、论文学习、数据库筑基课 |
| 讲 RocksDB 的具体模块、Compaction、参数、放大问题 | `RocksDB/` | RocksDB Compaction |
| 讲 LevelDB 作为最小 LSM 样本的文件、Manifest、Flush、Compaction | `LevelDB/` | LevelDB 如何组织数据 |
| 讲专用存储引擎完整设计，并以读写路径、索引和 Compaction 为主问题 | 对应技术目录，如 `FlowDB/` | FlowDB 时序存储引擎 |

## 技术子目录

| 技术/主题 | 目录 | 文章数 | 定位 |
|---|---|---:|---|
| LSM-Tree | [LSM-Tree/](LSM-Tree/) | 3 | 写优化存储结构与 Compaction 抽象 |
| RocksDB | [RocksDB/](RocksDB/) | 1 | 嵌入式高性能 LSM 存储引擎 |
| LevelDB | [LevelDB/](LevelDB/) | 1 | 最小 LSM 样本，适合理解文件与版本组织 |
| FlowDB | [FlowDB/](FlowDB/) | 1 | 面向时序场景的专用存储引擎案例 |

## 排重准则

问题指纹：

```text
存储引擎本体 + WAL/MemTable/SSTable/索引/Compaction/缓存模块 + 核心机制 + 解决问题 + 放大代价/适用边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都讲 LSM | 按写路径、读路径、Compaction、版本管理、恢复、放大问题拆分 |
| 都讲 Compaction | 比较是 size-tiered、leveled、universal，还是具体 RocksDB 参数和故障 |
| 只讲某数据库用了 LSM | 如果不展开存储机制，只作为该数据库技术目录的背景，不进本目录 |
| 只给性能结论 | 没有数据规模、基线、写入模式、压缩/缓存参数时降权 |

## 后续追查

- LSM：WAL、MemTable、SSTable、Bloom Filter、Manifest、Compaction、读写空间放大。
- RocksDB：Column Family、Block Cache、Compaction Filter、Write Stall、参数调优。
- LevelDB：文件组织、版本编辑、恢复路径。
- FlowDB：时序场景约束、索引结构、性能证据和适用边界。
