# 宽列数据库

> 分类规则、一二级类目定位、跨域归类边界以根 [目录划分.md](../../目录划分.md) 为准。
> 文章理解的四步法、整理流程以根 [AGENTS.md](../../AGENTS.md) 为准。
> 本文件只维护本领域的认知重点、排重指纹、已覆盖三级节点和待补缺口。

## 三级节点入口

| 节点 | 用途 |
|---|---|
| [040801_HBase](040801_HBase/AGENTS.md) | 宽列/列族数据库，RowKey、Region、HFile、随机读写 |

## 用户认知重点

| 项 | 内容 |
|---|---|
| 已知基础 | 用户知道 HBase 是宽列数据库，常配合 Hadoop/数仓使用 |
| 待补边界 | HBase 与离线数仓、KV、关系数据库、湖仓表的差异 |
| 易偏差点 | 把 HBase 当离线数仓或普通 KV；忽视 RowKey 设计与 Region 热点 |
| 优先抽取 | RowKey、列族、Region、MemStore、HFile、WAL、Compaction |

## 排重指纹

```text
宽列数据库 + RowKey/列族/Region/MemStore/HFile/WAL/Compaction 模块 + 核心机制 + 解决问题 + 随机读写/入仓/存储边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都讲 HBase 写入 | 按 WAL、MemStore、HFile、Flush、Compaction 拆分 |
| 都讲 RowKey | 比较是设计原则、热点排查还是预分区方案 |
| 与数仓混淆 | 加工建模归数仓，HBase 本体留本类 |
| 与 KV 混淆 | 列族与宽列模型留本类，纯 KV 协议归缓存与 KV |

## 已覆盖问题（按三级节点）

| 节点 | 已覆盖 | 还缺 |
|---|---|---|
| HBase | 初始承接 HBase 写入流程、列族和入仓方案文章 | RegionServer、Compaction、RowKey 设计、Phoenix、与 Hive/湖仓同步边界 |

## 待补缺口

| 优先级 | 项 | 为什么补 |
|---|---|---|
| 高 | RowKey 设计与热点排查 | HBase 性能与稳定性的核心 |
| 高 | RegionServer 与 Compaction | 写入与运维的关键模块 |
| 中 | HBase 与 Hive/湖仓同步 | 入仓与下游消费的边界 |
| 中 | Phoenix SQL 层 | 兼容 SQL 接入的常见方案 |
