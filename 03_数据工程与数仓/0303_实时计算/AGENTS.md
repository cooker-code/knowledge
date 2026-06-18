# 实时计算

> 分类规则、一二级类目定位、跨域归类边界以根 [目录划分.md](../../目录划分.md) 为准。
> 文章理解的四步法、整理流程以根 [AGENTS.md](../../AGENTS.md) 为准。
> 本文件只维护本领域的认知重点、排重指纹、已覆盖三级节点和待补缺口。

## 三级节点入口

| 节点 | 用途 |
|---|---|
| [030301_Flink](030301_Flink/AGENTS.md) | 有状态流处理与流批一体计算引擎 |
| [030302_Flink CDC](<030302_Flink CDC/AGENTS.md>) | 数据库变更捕获 Pipeline |
| [030303_Fluss](030303_Fluss/AGENTS.md) | 流式存储 |
| [030304_Kafka](030304_Kafka/AGENTS.md) | 消息系统与流式数据底座 |

## 用户认知重点

| 项 | 内容 |
|---|---|
| 已知基础 | 用户知道实时计算服务实时数仓、CDC、流式加工，无需重复讲流批区别 |
| 待补边界 | Flink、Flink CDC、Kafka、Paimon、Doris、Spark Streaming 的分工 |
| 易偏差点 | 调优文章易只给参数；CDC 文章易只讲跑通链路不讲事件语义和恢复边界 |
| 优先抽取 | 状态、时间、容错、事件语义、下游一致性 |

## 排重指纹

```text
实时技术 + 状态/时间/窗口/Join/Checkpoint/Connector 模块 + 核心机制 + 解决问题 + 延迟/一致性/成本边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是 Flink Join | 按 Regular、Interval、Temporal、Lookup、维表、状态治理拆分 |
| 都是性能优化 | 按反压、状态、Checkpoint、数据倾斜、序列化、资源拆分 |
| 都是 CDC | 区分源端采集、Exactly Once、Schema 演进、下游写入 |
| 只给参数清单 | 没有机制和验收指标时不新建 |
| 有生产故障路径 | 优先沉淀为实践或排障知识 |

## 已覆盖问题（按三级节点）

| 节点 | 已覆盖 | 还缺 |
|---|---|---|
| Flink | 双流 Join 状态膨胀、KeyGroup 与扩缩容、状态后端选型、State TTL、窗口触发与状态淘汰、SQL 大状态调优、通用增量 Checkpoint、Checkpoint/Savepoint 边界、反压排查入口 | Connector、端到端 Exactly Once、窗口/Checkpoint/State TTL/SQL 状态调优的生产指标验证 |
| Flink CDC | 3.x Pipeline 架构、全增量切换、MySQL Source Enumerator、事件模型、`server_id` 冲突、PostgreSQL WAL 膨胀、Doris/StarRocks/Kafka 下游一致性、生产切换双跑对数 | Schema 演进生产限制、DDL 冲突恢复、端到端恢复实验 |
| Kafka | 消费滞后定位、分区策略与 Flink 写入边界、Exactly Once 语义、Consumer Rebalance 机制 | 存储日志、KRaft、Kafka 与 Flink 端到端一致性实测 |

## 待补缺口

| 优先级 | 项 | 为什么补 |
|---|---|---|
| 高 | Flink + Kafka 端到端 Exactly Once | 一致性核心，需与 Kafka 事务区分 |
| 高 | Flink CDC Schema 演进与 DDL 恢复 | 整库同步与下游表结构正确性 |
| 高 | 反压与 Checkpoint 排障 | 生产高频问题 |
| 高 | 窗口触发与迟到数据验证 | 机制已沉淀，缺固定输入序列、迟到侧输出和状态清理实验 |
| 高 | State TTL 生产验证 | 缺指标、版本和 savepoint 兼容性验证 |
| 高 | Flink SQL 状态算子排查实验 | 缺真实执行计划和 Web UI 指标 |
