# 湖仓表格式

> 分类规则、一二级类目定位、跨域归类边界以根 [目录划分.md](../../目录划分.md) 为准。
> 文章理解的四步法、整理流程以根 [AGENTS.md](../../AGENTS.md) 为准。
> 本文件只维护本领域的认知重点、排重指纹、已覆盖三级节点和待补缺口。

## 三级节点入口

| 节点 | 用途 |
|---|---|
| [030501_Delta Lake](<030501_Delta Lake/AGENTS.md>) | Databricks 系开放表格式 |
| [030502_Hudi](030502_Hudi/AGENTS.md) | Timeline、Index、COW/MOR 与多流拼接 |
| [030503_Iceberg](030503_Iceberg/AGENTS.md) | Snapshot、Branch/Tag、v3 删除向量与查询加速 |
| [030504_LakeSoul](030504_LakeSoul/AGENTS.md) | 国产开放湖仓表格式 |
| [030505_Paimon](030505_Paimon/AGENTS.md) | 实时湖仓主键表、LSM、Changelog、Append |
| [030506_湖仓架构与选型](030506_湖仓架构与选型/AGENTS.md) | 跨表格式、跨引擎、Schema 演进和湖仓架构选型 |

## 用户认知重点

| 项 | 内容 |
|---|---|
| 已知基础 | 用户知道数据湖、数仓、Hive 表的大体区别，无需重复讲概念史 |
| 待补边界 | 表格式与计算引擎、OLAP 引擎、消息队列、CDC 链路之间的边界 |
| 易偏差点 | "湖仓地基""替代数仓""iPhone 时刻"这类标题易夸大 |
| 优先抽取 | 快照/事务/增量机制、引擎兼容、版本边界、生产实践 |

## 排重指纹

```text
表格式 + 表类型/模块 + 快照/事务/增量/更新机制 + 解决问题 + 引擎/延迟/成本边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是湖仓综述 | 没有具体机制和对标，通常略读 |
| 都是 Paimon | 按主键表、Append 表、Changelog、Compaction、Flink 集成拆分 |
| 都是 Iceberg/Hudi/Paimon 对比 | 有明确选型准则才沉淀 |
| 只讲口号 | 跳过或只保留认知校准 |
| 提供实践链路 | 优先沉淀，进入后续实验 |

## 已覆盖问题（按三级节点）

| 节点 | 已覆盖 | 还缺 |
|---|---|---|
| Delta Lake | 仅创建待补入口，未做正式文章沉淀 | 本地原文、官网/GitHub、与 Iceberg/Hudi/Paimon 对比 |
| Hudi | Timeline、Index、COW/MOR、Payload 多流拼接、并发写边界 | 当前版本补证、Flink 写入、Compaction/Clean 实验 |
| Iceberg | Snapshot、Branch/Tag、v3 删除向量、行级血缘、查询加速平台边界 | 官方 v3 支持矩阵、Flink CDC/增量读写、Catalog 选型 |
| LakeSoul | 技术定位与证据缺口 | 官方版本、核心机制、与 Iceberg/Hudi/Paimon 横向对比 |
| Paimon | 实时湖仓表格式定位、CDC 接入、Merge Engine、Changelog、Append 表边界、外部状态、LSM、Bloom、生命周期治理、版本演进、时间旅行、生产治理 | 官方版本限制、查询引擎兼容矩阵、最小实验 |
| 湖仓架构与选型 | 开放表格式选型、Schema 演进、多引擎访问、数据湖/数仓/数据网格边界 | 官方兼容矩阵、统一实验、查询出口边界 |

## 待补缺口

| 优先级 | 项 | 为什么补 |
|---|---|---|
| 高 | Delta Lake 本体 | Iceberg/Hudi/Paimon 的核心横向对标对象 |
| 高 | Iceberg/Hudi/Paimon/Delta Lake 对比 | 形成表格式选型准则，避免被"谁取代谁"标题误导 |
| 高 | Flink CDC -> 表格式链路对比 | 用户关注 CDC、Schema 演进、增量读写和下游一致性 |
| 中 | OLAP 引擎读取湖表边界 | Doris/StarRocks/Trino/ClickHouse 与湖表的职责容易混 |
