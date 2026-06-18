# 存储引擎

> 分类规则、一二级类目定位、跨域归类边界以根 [目录划分.md](../../目录划分.md) 为准。
> 文章理解的四步法、整理流程以根 [AGENTS.md](../../AGENTS.md) 为准。
> 本文件只维护本领域的认知重点、排重指纹、已覆盖三级节点和待补缺口。

## 三级节点入口

| 节点 | 用途 |
|---|---|
| [040601_FlowDB](040601_FlowDB/AGENTS.md) | 面向时序场景的专用存储引擎案例 |
| [040602_LSM-Tree](040602_LSM-Tree/AGENTS.md) | 写优化存储结构与 Compaction 抽象 |
| [040603_LevelDB](040603_LevelDB/AGENTS.md) | 最小 LSM 样本，文件与版本组织 |
| [040604_RocksDB](040604_RocksDB/AGENTS.md) | 嵌入式高性能 LSM 存储引擎 |

## 用户认知重点

| 项 | 内容 |
|---|---|
| 已知基础 | 用户知道很多数据库底层用 LSM 或 B+Tree，但具体存储层关系模糊 |
| 待补边界 | LSM、RocksDB、LevelDB 与上层数据库（Doris/StarRocks/Kvrocks/HBase 等）的承载关系 |
| 易偏差点 | 只关注性能结论，忽略写放大、读放大、空间放大与 Compaction 代价 |
| 优先抽取 | WAL、MemTable、SSTable、Bloom、Manifest、Compaction、Block Cache |

## 排重指纹

```text
存储引擎本体 + WAL/MemTable/SSTable/索引/Compaction/缓存模块 + 核心机制 + 解决问题 + 放大代价/适用边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都讲 LSM | 按写路径、读路径、Compaction、版本管理、恢复、放大问题拆分 |
| 都讲 Compaction | 比较是 size-tiered、leveled、universal，还是具体 RocksDB 参数和故障 |
| 只讲某数据库用了 LSM | 不展开存储机制只作为该数据库技术目录的背景，不进本目录 |
| 只给性能结论 | 没有数据规模、基线、写入模式、压缩/缓存参数时降权 |

## 已覆盖问题（按三级节点）

| 节点 | 已覆盖 | 还缺 |
|---|---|---|
| LSM-Tree | 抽象结构、WAL/MemTable/SSTable、Compaction 策略候选承接 | Bloom Filter、Manifest、读写空间放大系统化沉淀 |
| RocksDB | 嵌入式高性能 LSM 引擎候选承接 | Column Family、Block Cache、Compaction Filter、Write Stall、参数调优 |
| LevelDB | 最小 LSM 样本候选承接 | 文件组织、版本编辑、Manifest、Flush、恢复路径完整沉淀 |
| FlowDB | 时序专用存储引擎案例承接 | 时序场景约束、索引结构、性能证据和适用边界 |

## 待补缺口

| 优先级 | 项 | 为什么补 |
|---|---|---|
| 高 | LSM 写/读/空间放大系统对比 | 选型与参数调优的根本依据 |
| 高 | RocksDB Compaction 与 Write Stall | 生产高频卡点 |
| 中 | LevelDB Manifest 与版本编辑 | 理解 LSM 恢复路径的最小样本 |
| 中 | FlowDB 时序场景适用边界 | 区分通用 LSM 与时序专用引擎 |
