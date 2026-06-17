# OLAP 引擎
## 知识点入口

- 本模块先看宏观流程，再看文章：[流程化知识点总览](knowledge/04_OLAP与数据库/0401_OLAP引擎/核心知识点/流程化知识点总览.md)。
- 新文章必须先归入流程节点，再判断是补充、冲突、不同层次还是降权。
- `文章/` 只保留原文锚点，长期知识必须沉淀到 `核心知识点/`。


## 类目定位

| 项 | 内容 |
|---|---|
| 一级类目 | OLAP 与数据库 |
| 二级类目 | OLAP 引擎 |
| 核心问题 | 如何支撑交互式分析、报表查询、聚合分析、明细检索和部分服务化查询 |
| 不解决什么 | 不承担离线数仓建模全链路，不直接替代消息队列、缓存或 OLTP 数据库 |
| 用户当前认知假设 | L1 到 L2：知道 Doris、StarRocks、ClickHouse 等名字，需要强化存储模型、查询路径、索引和场景边界 |

## 用户认知重点

| 认知项 | 当前假设 | 后续整理策略 |
|---|---|---|
| 已知基础 | OLAP 用于查询加速和分析服务 | 不重复讲“OLAP 是什么” |
| 待补边界 | OLAP 与 Hive、湖仓表格式、Redis/KV、Elasticsearch 的边界 | 每篇文章必须写横向对标 |
| 易偏差点 | 性能数字和“极速”表述容易脱离基线 | 标出数据量、并发、冷热、版本和查询条件 |

## 排重准则

问题指纹：

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

## 已覆盖技术

| 技术 | index | 已覆盖问题 | 还缺什么 |
|---|---|---|---|
| Doris | [Doris](Doris/AGENTS.md) | 点查短路径、行存、Row Cache、BE 内存排障、FE 运维 SOP、FE 元数据恢复、Compaction、数据导入、表模型、OLAP 定位 | 索引、物化视图、统计信息与 CBO、FE 元数据故障官方补证 |
| ClickHouse | [ClickHouse](ClickHouse/AGENTS.md) | OLAP 定位、Skip Index 原理、MergeTree 批处理预排序、AggregateFunction 与物化视图、分布式 JOIN | Upsert/实时更新边界、日志留存、MergeTree 官方补证 |
| StarRocks | [StarRocks](StarRocks/AGENTS.md) | Primary Key 实时更新模型、PK 事务与 DelVector、Query Cache 中间聚合缓存、存算分离 Compaction 机制、物化视图建模与透明加速 | 湖仓查询、资源隔离、物化视图命中排查、PK 内存与 Commit 指标 |

## 待补技术和问题

| 技术/问题 | 为什么要补 | 优先级 |
|---|---|---|
| Doris FE 元数据故障恢复 | 已有 FE 运维入口，但还缺 BDB 元数据、选主失败、恢复步骤和验证证据 | 高 |
| ClickHouse MergeTree | ClickHouse 查询和存储的核心模块 | 高 |
| Doris / StarRocks / ClickHouse 横向选型 | 需要按导入、更新、物化视图、并发、运维分别对标 | 高 |

## 相邻类目

| 相邻类目 | 边界 |
|---|---|
| [嵌入式分析](../0403_嵌入式分析/AGENTS.md) | DuckDB 这类进程内本地分析数据库归这里，不和服务化 OLAP 引擎混在同一个二级目录 |

<!-- AUTO:SECONDARY_INIT_START -->
## 全量文章来源初始化

> 自动生成。初始化阶段只使用本地 `本地文章目录`、已有 `knowledge` 和本地 `wiki`，不联网补官网或外部证据。

- 全量文章来源：[文章](文章/)
- 全局明细：`scripts/output/knowledge-secondary-pools.json`

| 指标 | 数量 |
|---|---:|
| 文章数 | 68 |
| 正式沉淀原文数 | 14 |
| 已引用锚点 | 61 |
| 核心知识点数 | 16 |
| 精读候选 | 33 |
| 略读 | 19 |
| 跳过 | 2 |
| 低置信 | 0 |
| 原图缺失 | 36 |

### 主题簇

| 技术/主题 | 文章数 | 正式沉淀 | 精读候选 | 原图缺失 | 处理决策 | 认知校准点 |
|---|---:|---:|---:|---:|---|---|
| Doris | 38 | 6 | 22 | 22 | 以已有核心知识点为排重基线，只补边界、失败场景或实践证据 | 原目录存在误导，按技术本体重路由；有技术图缺失，精修时需回原文或重建 |
| StarRocks | 17 | 4 | 6 | 8 | 以已有核心知识点为排重基线，只补边界、失败场景或实践证据 | 原目录存在误导，按技术本体重路由；有技术图缺失，精修时需回原文或重建 |
| ClickHouse | 13 | 4 | 5 | 6 | 以已有核心知识点为排重基线，只补边界、失败场景或实践证据 | 原目录存在误导，按技术本体重路由；有技术图缺失，精修时需回原文或重建 |

### 精读候选

| 技术对象 | 原文 | 冲突点 | 处理建议 |
|---|---|---|---|
| ClickHouse | [1w字详解 ClickHouse漏斗模型实践方案（收藏）](文章/1w字详解 ClickHouse漏斗模型实践方案（收藏）.md) | 原目录与最终归类不一致；正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| ClickHouse | [ClickHouse 直接读取 MySQL Dump 文件：高效数据迁移与分析的进阶指南](文章/ClickHouse 直接读取 MySQL Dump 文件：高效数据迁移与分析的进阶指南.md) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| ClickHouse | [使用 ClickHouse 实现 Medallion 架构](文章/使用 ClickHouse 实现 Medallion 架构.md) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| ClickHouse | [使用ClickHouse、Grafana和WarpStream规模化的解决可预测成本的日志留存](文章/使用ClickHouse、Grafana和WarpStream规模化的解决可预测成本的日志留存.md) | 正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| ClickHouse | [火山引擎：ClickHouse增强计划之“Upsert”](文章/火山引擎：ClickHouse增强计划之“Upsert”.md) | 正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Doris | [[技术调研]OLAP数据库执行引擎的NUMA优化](文章/[技术调研]OLAP数据库执行引擎的NUMA优化.md) | 正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Doris | [「硬刚Doris系列」Apache Doris的向量化和Roaring BitMap](文章/「硬刚Doris系列」Apache Doris的向量化和Roaring BitMap.md) | 原目录与最终归类不一致；标题/观点需要降权；正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Doris | [「硬刚Doris系列」官方常见问题小汇总](文章/「硬刚Doris系列」官方常见问题小汇总.md) | 标题/观点需要降权 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Doris | [0代码！教会你用Doris+DeepSeek+Dify搭建ChatBI系统（附完整DSL）](文章/0代码！教会你用Doris+DeepSeek+Dify搭建ChatBI系统（附完整DSL）.md) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Doris | [3步！教会你用 Doris MCP + LangChain 搭建AI问数系统（保姆级教程）](文章/3步！教会你用 Doris MCP + LangChain 搭建AI问数系统（保姆级教程）.md) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Doris | [AI+数据分析来了！Doris 让SQL直接调用DeepSeek、ChatGPT、Claude...](文章/AI+数据分析来了！Doris 让SQL直接调用DeepSeek、ChatGPT、Claude....md) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Doris | [Apache Doris Compaction优化百科全书](文章/Apache Doris Compaction优化百科全书.md) | 正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Doris | [Apache Doris 为分析而生开篇：整体架构！](文章/Apache Doris 为分析而生开篇：整体架构！.md) | 正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Doris | [Apache Doris 故障自助排查指南（P0 篇）](文章/Apache Doris 故障自助排查指南（P0 篇）.md) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Doris | [Doris 解析 \| Apache Doris 极速1.0版本解析与未来规划](文章/Doris 解析 _ Apache Doris 极速1.0版本解析与未来规划.md) | 正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |

### 冲突与缺口

- 冲突分布：正文提到技术图但 Markdown 无图(36)、原目录与最终归类不一致(21)、标题/观点需要降权(2)
- 存在原图缺失，精修时只补架构图、流程图、说明图和对比图。
- 精读候选不直接写笔记，先和已有问题指纹对比。
- 初始化阶段不补外部官网/GitHub；需要官方证据时在后续精修阶段补证。

### 下一步

| 优先级 | 动作 |
|---|---|
| P0 | 先处理精读候选，按主题簇合并，不逐篇扩写 |
| P1 | 对已有核心知识点补充排重依据和认知校准点 |
| P2 | 精修阶段再补官网、GitHub、版本状态和官方架构图 |
<!-- AUTO:SECONDARY_INIT_END -->
