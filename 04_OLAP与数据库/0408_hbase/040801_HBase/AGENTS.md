# HBase

## 技术定位

| 项 | 内容 |
|---|---|
| 技术名 | HBase |
| 一级类目 | OLAP 与数据库 |
| 二级类目 | 宽列数据库 |
| 技术本体 | 基于 HDFS 的分布式宽列数据库，面向大规模稀疏数据的随机读写和列族存储 |
| 全局架构位置 | 位于存储服务层，常作为业务或画像/标签系统的在线存储，也可能作为数仓入仓数据源 |
| 主要使用者 | 后端工程师、数据平台工程师、数仓工程师 |
| 主要产出 | HBase 表、列族、Region、HFile、WAL、增量入仓数据 |

## 归类边界

| 相邻对象 | 不归入这里的情况 |
|---|---|
| 离线数仓 | 文章主问题是 Hive/Spark 建模和分层加工，HBase 只是上游来源时，归离线数仓 |
| 数据集成 | 文章主问题是同步工具或 CDC 管道本身时，归数据集成 |
| 存储引擎 | 文章抽象讲 LSM/WAL/MemTable/SSTable，不围绕 HBase 系统时，归存储引擎 |
| OLAP 引擎 | 文章主问题是 ClickHouse/Doris/StarRocks 查询服务时，不因提到 HBase 数据源而归 HBase |

## 排重准则

问题指纹：

```text
HBase + 写入/列族/Region/MemStore/HFile/WAL/入仓模块 + 核心机制 + 解决问题 + 随机读写/增量抽取/业务库压力边界 + 对用户的认知增量
```

## 已沉淀核心知识点

| 主题 | 文件 | 问题指纹 | 解决什么问题 | 认知增量 |
|---|---|---|---|---|
| 写入链路、MemStore Flush 与 HFile 边界 | [HBase写入链路MemStoreFlush与HFile边界](040801_核心知识点/HBase写入链路MemStoreFlush与HFile边界.md) | HBase + 写入链路、MemStore Flush 与 HFile 边界 + 机制/边界/验证 | HBase 写入链路从 Region 定位、WAL、MemStore 到 Flush 生成 HFile，再由 Compaction 合并 | 形成可复用判断，不保留文章池 |
| Compaction 与低延时调优边界 | [HBaseCompaction与低延时调优边界](040801_核心知识点/HBaseCompaction与低延时调优边界.md) | HBase + Compaction 与低延时调优边界 + 机制/边界/验证 | HBase 低延时问题常来自 Region 分布、HFile 数量、BlockCache、Compaction、数据异常和 Flush 抖动 | 形成可复用判断，不保留文章池 |
| Meta 与 Replication 运维排障边界 | [HBaseMeta与Replication运维排障边界](040801_核心知识点/HBaseMeta与Replication运维排障边界.md) | HBase + Meta 与 Replication 运维排障边界 + 机制/边界/验证 | HBase 运维高风险点集中在 meta 表、Region 分配、协处理器异常和 Replication 线程阻塞 | 形成可复用判断，不保留文章池 |
| HBase 入仓边界 | [HBase入仓边界](040801_核心知识点/HBase入仓边界.md) | HBase + HBase 入仓边界 + 机制/边界/验证 | HBase 入仓的主问题不是“能不能扫 HBase”，而是如何减少业务库压力、处理字段变更、增量时间戳、回溯和权限治理 | 形成可复用判断，不保留文章池 |

## 后续追查

- 写入链路：客户端、Region 定位、WAL、MemStore、Flush、HFile、Compaction。
- 列族设计：列族数量、数据局部性、Flush/Compaction 放大。
- HBase 入仓：TimeRange、rowKey get、Phoenix 二级索引和 Hive 映射表的边界。
