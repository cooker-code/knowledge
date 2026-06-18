# ClickHouse Skip Index 原理

## 原文锚点

- 本地文件：[ClickHouse 原理 | ClickHouse 的索引原理](<../文章/ClickHouse 原理 _ ClickHouse 的索引原理.md>)
- 原文链接：`http://mp.weixin.qq.com/s?__biz=MzU5OTQ1MDEzMA==&mid=2247490917&idx=2&sn=8c3f9268234c601929bd46537c825f2f`
- 关键段落：ClickHouse 索引与 RDBMS B-tree 的本质差异、skip index、granule、minmax/set/bloomfilter、GRANULARITY、读路径。
- 关键图：原文图未保留。

## 图片处理

| 图片 | 类型 | 是否保留 | 理由 | 处理方式 |
|---|---|---|---|---|
| B-tree 索引示意图、Query 处理流程 | 说明图 | 原图缺失 | 帮助理解索引差异和读路径 | Mermaid 重建读路径 |

```mermaid
flowchart LR
  Query["查询条件"] --> Part["过滤 Part"]
  Part --> PK["Primary Key\n缩小 granule 范围"]
  PK --> Skip["Skip Index\nminmax/set/bloomfilter"]
  Skip --> Read["读取剩余 granule"]
  Read --> Result["返回结果"]
```

## 一句话结论

这篇文章值得精读：ClickHouse 的 skip index 不是像 B-tree 一样快速定位目标行，而是按 granule 快速排除不需要读取的数据。

## 用户相关性判断

| 项 | 内容 |
|---|---|
| 用户当前认知层级 | ClickHouse / OLAP 引擎：L2 |
| 认知成熟度 | draft |
| 阅读投入建议 | 精读 |
| 阅读投入理由 | 能补 ClickHouse 与传统数据库索引的关键差异，但缺实际数据分布实验 |
| 对用户的新信息 | skip index 的效果取决于 granule、数据分布和索引类型，不是“加索引必快” |
| 问题指纹 | ClickHouse + 索引 + skip index/minmax/set/bloomfilter/granularity + 数据排除 + 不等同 B-tree |
| 排重判断 | 新建 |
| 置信度 | 高 |

## 认知校准点

| 校准点 | 文章观点/信息 | 与用户认知或价值观的关系 | 处理建议 |
|---|---|---|---|
| ClickHouse 索引不是传统二级索引 | 原文强调作用是排除不想要的数据 | 关键认知纠偏 | 写入 ClickHouse index |
| 索引效果依赖数据分布 | 离散数据可能效果差甚至拖累性能 | 符合用户反“最佳实践”偏好 | 建索引前先看分布和过滤条件 |
| GRANULARITY 是体积与效率取舍 | 每几个 granule 保存一份索引信息 | 重要配置边界 | 不照抄参数 |
| set 和 bloomfilter 都有成本 | set 大小不可控，bloomfilter 有假阳性 | 横向对标价值高 | 按列基数和查询条件选 |

## 冲突点

| 冲突类型 | 具体表现 | 影响 | 处理 |
|---|---|---|---|
| 图片缺失 | 索引和查询流程图缺失 | 影响机制理解 | Mermaid 重建 |
| 实践门槛不足 | 有建索引 SQL，但无数据分布和查询对照 | 不能判实践 | 降为精读 |
| 版本风险 | ClickHouse 索引能力随版本演进 | 生产前需查官方文档 | 标记后续追查 |

## 待吸收点

| 分级 | 内容 | 为什么值得吸收 | 后续动作 |
|---|---|---|---|
| 理解 | RDBMS B-tree 是定位目标行，ClickHouse skip index 是排除 granule | 这是核心差异 | 与 MySQL/PostgreSQL 索引对比 |
| 理解 | granule 是 skip index 作用单位 | 解释为什么不是行级精确索引 | 结合 MergeTree 继续追查 |
| 记住 | minmax 轻量，适合有范围聚集的数据 | 常见时间、数值范围过滤 | 实验验证 |
| 记住 | set 无假阳性但大小受唯一值数量影响 | 适合低基数或 granule 内唯一值少 | 建索引前看基数 |
| 记住 | bloomfilter 大小固定但有假阳性 | 适合包含/等值类过滤 | 需要验证误判率 |

## 已知可跳过

| 内容 | 跳过理由 |
|---|---|
| B-tree 基础介绍 | 用户大概率知道 |
| SQL 解析到执行计划通用流程 | 背景信息 |
| 文末引导 | 不进入知识点 |

## 实践门槛

| 门槛 | 判断 | 证据 |
|---|---|---|
| 可运行 | 部分 | 有 `ALTER TABLE ADD INDEX` 示例 |
| 可验证 | 否 | 没有数据分布和查询扫描量对照 |
| 可排障 | 部分 | 能解释索引为什么没命中 |
| 可迁移 | 是 | 可迁移到 OLAP 索引设计 |
| 结论 | 降为精读 | 后续补实验 |

## 归类判断

| 项 | 内容 |
|---|---|
| 技术本体 | ClickHouse MergeTree 索引 |
| 文章主问题 | skip index 如何过滤 granule |
| 使用场景 | OLAP 查询过滤、明细查询加速 |
| 关键词干扰 | B-tree、RDBMS 是对标，不是主类目 |
| 最终归类 | OLAP 与数据库 / OLAP 引擎 / ClickHouse |
| 归类理由 | 主问题是 ClickHouse 查询和存储引擎机制 |

## 技术定位

| 项 | 内容 |
|---|---|
| 技术类型 | OLAP 存储与查询优化机制 |
| 所属领域 | OLAP 与数据库 |
| 二级类目 | OLAP 引擎 |
| 全局架构位置 | ClickHouse MergeTree 读路径中的数据裁剪层 |
| 涉及模块 | Part、granule、primary key、skip index、MergeTreeSelectProcessor |
| 解决问题 | 减少不必要的数据读取 |
| 原文局限 | 无 benchmark、无版本说明 |
| 我的结论 | 以后关注，作为 ClickHouse 索引入口 |

## 纵向理解

| 维度 | 判断 |
|---|---|
| 全局架构 | SQL -> QueryPlan -> ReadFromMergeTree -> part/granule/index 过滤 -> 读取数据 |
| 本文位置 | MergeTree 读路径中的 skip index |
| 核心机制 | 对每个 granule 保存统计或集合信息，查询时排除不可能命中的 granule |
| 使用链路 | 看过滤条件 -> 看数据分布 -> 选索引类型 -> 调 granularity -> 验证扫描量 |
| 前置条件 | MergeTree 表、合适排序键、数据分布可利用 |
| 边界 | 离散数据或低选择性条件下可能收益低 |

## 横向对标

| 对标技术 | 实现方式 | 优势 | 劣势 | 适合场景 |
|---|---|---|---|---|
| B-tree 索引 | 树结构定位行 | 精确定位 | 对列式批量分析不一定合适 | OLTP 点查 |
| minmax | 保存范围 | 最轻量 | 数据无序时效果差 | 时间、范围聚集列 |
| set | 保存唯一值集合 | 无假阳性 | 唯一值多时索引失效或膨胀 | 低基数等值过滤 |
| bloomfilter | 布隆过滤 | 大小可控 | 有假阳性 | 高基数等值/包含判断 |

## 后续追查

- 关键词：ClickHouse skip index、MergeTree、granule、minmax、set、bloomfilter、GRANULARITY。
- 相关技术：Primary Key、Part、Mark、Materialized View。
- 需要补读的文章：ClickHouse MergeTree、物化视图、Block + LSM、Upsert。
