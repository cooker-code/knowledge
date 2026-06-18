# Fluss流式存储定位与表模型

## 来源
- [Fluss-面向分析的实时流存储初探](../文章/done-Fluss-面向分析的实时流存储初探.md)
- [淘天AB实验分析平台Fluss落地实践：更适合实时OLAP的消息队列](../文章/done-淘天AB实验分析平台Fluss落地实践：更适合实时OLAP的消息队列.md)
- [官宣 _ Apache Fluss (Incubating) 0.8 发布公告](<../文章/done-官宣 _ Apache Fluss (Incubating) 0.8 发布公告.md>)

## 核心问题

Fluss 解决的是“实时流数据既要持续写入，又要可分析、可点查、可更新、可接入湖仓”的问题。它不只是 Kafka 替代品，而是把消息流的一部分职责推进到实时分析存储层。

## 判断准则

| 判断项 | 准则 |
|---|---|
| 是否需要 Fluss | 如果链路只有顺序投递和消费，用 Kafka 足够；如果需要列裁剪、主键点查、更新表、CDC 订阅或 Flink 大状态外置，才进入 Fluss 评估 |
| 表模型选择 | 日志表适合事件流和追加数据；主键表适合最新状态、点查、部分列更新和宽表拼接 |
| 架构位置 | Fluss 更适合作为实时热层，长期历史、审计和多引擎湖表语义仍要交给 Iceberg/Paimon/Hudi/Delta 等湖表格式 |
| 运维前提 | 需要关注 Coordinator、TableServer、Bucket/Tablet、Flink Connector、湖表分层和升级兼容，不应按普通消息队列运维 |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| Fluss 可以直接替代 Kafka | 只在实时分析和 Flink 深度集成场景中有替代空间，通用事件总线仍要看 Kafka 生态 |
| Fluss 是 OLAP 引擎 | Fluss 解决实时流热层可分析和点查问题，复杂 BI 查询仍需要 OLAP 引擎 |
| 有 Fluss 就不需要湖表格式 | Fluss 更偏热数据和流式更新，长期快照、审计和多引擎湖仓仍依赖表格式 |

## 待验证缺口

- 官方文档中 Log Table、Primary Key Table、Bucket、Tablet 的限制和默认分桶策略。
- Fluss 和 Paimon/Iceberg 分层后的查询一致性、补数据和回放边界。
