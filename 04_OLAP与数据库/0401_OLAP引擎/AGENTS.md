# OLAP 引擎

> 分类规则、一二级类目定位、跨域归类边界以根 [目录划分.md](../../目录划分.md) 为准。
> 文章理解的四步法、整理流程以根 [AGENTS.md](../../AGENTS.md) 为准。
> 本文件只维护本领域的认知重点、排重指纹、已覆盖三级节点和待补缺口。

## 三级节点入口

| 节点 | 用途 |
|---|---|
| [040101_ClickHouse](040101_ClickHouse/AGENTS.md) | 列式分析数据库，预排序 MergeTree 与跳数索引 |
| [040102_Doris](040102_Doris/AGENTS.md) | MPP 分析数据库，FE/BE 架构与多表模型 |
| [040103_StarRocks](040103_StarRocks/AGENTS.md) | MPP 分析数据库，Primary Key 实时更新与物化视图 |

## 用户认知重点

| 项 | 内容 |
|---|---|
| 已知基础 | 用户知道 OLAP 用于查询加速和分析服务，无需重复讲"OLAP 是什么" |
| 待补边界 | OLAP 与 Hive、湖仓表格式、Redis/KV、Elasticsearch 的边界 |
| 易偏差点 | 性能数字和"极速"表述容易脱离基线，需要标出数据量、并发、冷热、版本和查询条件 |
| 优先抽取 | 存储模型、查询路径、索引、导入更新、物化视图与场景边界 |

## 排重指纹

```text
OLAP 引擎 + 查询/存储/导入/索引模块 + 核心机制 + 解决问题 + 性能基线/适用边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是 Doris 性能优化 | 按点查、Join、Compaction、索引、导入、内存拆分 |
| 都是物化视图 | 比较是查询改写、预聚合、湖仓加速还是成本治理 |
| 只有性能数字 | 无基线无场景则降权 |
| 有排障 SOP | 可沉淀为实践型知识点 |
| 与数仓混淆 | 主问题是加工建模归数仓，主问题是查询执行归 OLAP |

## 已覆盖问题（按三级节点）

| 节点 | 已覆盖 | 还缺 |
|---|---|---|
| ClickHouse | OLAP 定位、Skip Index 原理、MergeTree 批处理预排序、AggregateFunction 与物化视图、分布式 JOIN | Upsert/实时更新边界、日志留存、MergeTree 官方补证 |
| Doris | 点查短路径、行存、Row Cache、BE 内存排障、FE 运维 SOP、FE 元数据恢复、Compaction、数据导入、表模型、OLAP 定位 | 索引、物化视图、统计信息与 CBO、FE 元数据故障官方补证 |
| StarRocks | Primary Key 实时更新模型、PK 事务与 DelVector、Query Cache 中间聚合缓存、存算分离 Compaction 机制、物化视图建模与透明加速 | 湖仓查询、资源隔离、物化视图命中排查、PK 内存与 Commit 指标 |

## 待补缺口

| 优先级 | 项 | 为什么补 |
|---|---|---|
| 高 | Doris FE 元数据故障恢复 | 已有 FE 运维入口，但还缺 BDB 元数据、选主失败、恢复步骤和验证证据 |
| 高 | ClickHouse MergeTree | ClickHouse 查询和存储的核心模块 |
| 高 | Doris / StarRocks / ClickHouse 横向选型 | 需要按导入、更新、物化视图、并发、运维分别对标 |
