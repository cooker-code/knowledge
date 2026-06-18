# FlinkCDC版本演进与Pipeline连接器边界

> 验证版本：Flink CDC 3.0-3.6，具体能力以官方 Release 和对应 Flink 版本为准

## 来源
- [Flink CDC 1.0至3.0回忆录](<../文章/done-Flink CDC 1.0至3.0回忆录.md>)
- [Flink CDC 3.0 新一代实时数据集成框架](<../文章/done-Flink CDC 3.0 新一代实时数据集成框架.md>)
- [Flink CDC 3.0 正式发布，详细解读新一代实时数据集成框架](<../文章/done-Flink CDC 3.0 正式发布，详细解读新一代实时数据集成框架.md>)
- [Flink CDC 3.3.0 发布公告](<../文章/done-Flink CDC 3.3.0 发布公告.md>)
- [Flink CDC 3.4 pipeline，哦，蟹特.](<../文章/done-Flink CDC 3.4 pipeline，哦，蟹特..md>)
- [Flink CDC 3.5.0 Source，我特么...](<../文章/done-Flink CDC 3.5.0 Source，我特么....md>)

## 核心问题

Flink CDC 3.x 的核心变化不是单个 Source Connector 升级，而是从“Flink 连接器集合”扩展成带 Pipeline、Schema Evolution、Route、Transform 和多 Sink 的数据集成框架。

## 判断准则

| 判断项 | 准则 |
|---|---|
| 1.x/2.x 文章 | 主要作为 Source Connector、Debezium 集成和全增量同步历史背景，不直接推当前结论 |
| 3.0 文章 | 重点吸收 Pipeline Framework、API/Connect/Composer/Runtime 分层、YAML 作业和整库同步 |
| 3.4/3.5/3.6 文章 | 进入版本记录，重点核对连接器列表、Flink 版本、JDK 版本、Schema Evolution 和 Transform 行为 |
| 选型边界 | 需要多源多目标、整库同步、Schema 变更和路由时看 Pipeline；只在 Flink SQL 内写 CDC Source 时仍按连接器治理 |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| Flink CDC 就是 MySQL Source | 3.x 已经扩展为数据集成框架，Source 只是其中一层 |
| 发布公告可以直接当生产能力 | 连接器能力要看官方 Release、目标 Flink 版本和 Sink 端语义 |
| Pipeline 写成功就代表一致性正确 | 主键、Schema、无主键表、恢复、DDL 和目标端幂等才决定正确性 |

## 待验证缺口

- 以 MySQL -> Paimon/Doris/StarRocks/Kafka 四条链路验证 DDL、无主键表、任务恢复和重放语义。
- 补齐 Flink CDC 3.6 的 JDK、Flink 版本和 Pipeline Connector 官方限制。

## 重新蒸馏补充（2026-06-18）

| 来源 | 认知增量 | 处理 |
|---|---|---|
| [[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/文章/done-Flink CDC 黄金搭档：Debezium 1.9.0.CR1 发布！]] | 补充该主题的生产案例、机制边界或排重样例。 | 重新判断后补入目标知识产物 |
| [[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/文章/done-Flink CDC：新一代实时数据集成框架]] | 补充该主题的生产案例、机制边界或排重样例。 | 重新判断后补入目标知识产物 |
