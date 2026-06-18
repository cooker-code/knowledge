# Fluss

## 快速入口

| 入口 | 用途 |
|---|---|
| [知识地图](030303_知识地图.md) | Fluss 的定位、表模型、Delta Join、湖流一体和待补缺口 |
| [版本记录](030303_版本记录.md) | Apache Fluss 版本、孵化状态和版本敏感边界 |
| [030303_核心知识点/](030303_核心知识点/) | 已蒸馏的长期知识点 |
| [文章/](文章/) | 原文锚点，只保留已经吸收到长期知识结构的 `done-` 文件 |

## 技术定位

| 项 | 内容 |
|---|---|
| 技术本体 | 面向实时分析的流式存储，提供 Log Table、Primary Key Table、列式读取、点查、变更日志和湖流一体能力 |
| 架构位置 | 位于 Flink 等实时计算引擎和湖仓表格式之间，承接 Kafka 类消息流、Flink State 外置、实时 OLAP 探查和湖仓热层 |
| 主要问题 | 解决传统消息队列不便点查、不便列裁剪、不承担更新表语义，以及 Flink 大状态作业难运维的问题 |
| 不是 | 不是通用 OLAP 引擎；不是完全替代 Kafka 的消息系统；也不是 Iceberg/Paimon/Hudi/Delta Lake 这类长期湖表格式 |

## 官方锚点

- 项目主页：[Apache Fluss](https://fluss.apache.org/)
- 代码与版本：[apache/fluss releases](https://github.com/apache/fluss/releases)
- 版本记录只引用官方发布页、GitHub Release 或官方文档，不凭社区文章猜版本。

## 横向对标

| 对象 | 相同点 | Fluss 差异 | 选型边界 |
|---|---|---|---|
| Kafka | 都承接流式写入和订阅 | Fluss 强调列式读取、主键点查、更新流和 Flink 深度集成 | 需要消息生态、跨语言客户端和通用事件总线时 Kafka 仍是基线 |
| Paimon | 都可服务实时湖仓状态表 | Fluss 更偏实时流存储热层；Paimon 更偏湖仓表格式和离线/实时统一表 | 需要长期湖表、快照和多引擎读写时优先看 Paimon |
| Flink State | 都能保存中间状态 | Fluss 将部分状态外置为可查询表，降低大状态 Join 的恢复和 checkpoint 压力 | 低延迟算子内状态仍应留在 Flink State |
| OLAP 引擎 | 都可用于查询分析 | Fluss 主要解决实时流探查和热数据访问，不替代 Doris/StarRocks/Hologres 的复杂分析服务 | 面向报表和高并发 BI 查询仍需 OLAP 出口 |

## 文章处理 SOP

1. 先判断文章主问题：流式存储本体、Delta Join、大状态治理、实时 OLAP 探查、湖流一体、还是应用场景。
2. 只讲“下一代”“更快”“更省”的宣传文不单独沉淀；必须抽出表模型、状态边界、读写链路、指标或失败模式。
3. 版本发布、Flink 兼容、孵化状态、升级注意事项进入 [版本记录](030303_版本记录.md)。
4. Agent 风控、AB 实验等场景文，只吸收 Fluss 在事件采集、点查、列裁剪、状态外置中的工程角色，不沉淀业务叙事。
5. 有贡献文章改为 `done-` 前缀；无贡献文章直接删除。

## 排重判断

| 指纹项 | 排重规则 |
|---|---|
| 技术本体 | 是否讨论 Fluss 自身，而不是借 Fluss 讲 Flink、Agent 或业务平台 |
| 模块 | Log Table、Primary Key Table、Bucket/Tablet、Delta Join、Lakehouse Tiering、Flink Connector |
| 核心机制 | 列式流、点查、更新流、状态外置、湖流一体、元数据/Coordinator/TableServer |
| 解决问题 | 大状态、数据探查、列裁剪、实时宽表、热冷分层、事件流可观测 |
| 边界 | 与 Kafka、Paimon、Flink State、OLAP 引擎的职责差异 |

## 已沉淀核心知识点

| 知识点 | 文件 | 认知校准 |
|---|---|---|
| 流式存储定位与表模型 | [Fluss流式存储定位与表模型](030303_核心知识点/Fluss流式存储定位与表模型.md) | Fluss 的价值不只是“替代消息队列”，而是让实时流具备列裁剪、点查和更新表语义 |
| Delta Join 与外部化状态 | [Fluss Delta Join 与外部化状态边界](<030303_核心知识点/Fluss Delta Join 与外部化状态边界.md>) | Delta Join 是把部分 Join 历史外置到 Fluss，而不是让 Flink 不再需要状态 |
| 实时 OLAP 与事件流场景 | [Fluss实时OLAP与事件流场景边界](030303_核心知识点/Fluss实时OLAP与事件流场景边界.md) | AB 实验和 Agent 风控文章只作为场景证据，长期结论仍落在流式存储能力边界 |

## 后续追查

- 用官方文档补齐 Fluss 0.9.x 的 Log Table、Primary Key Table、Lakehouse Tiering 和 Flink Connector 限制。
- 实验验证：同一实时 Join 链路分别用 Flink State、Fluss Delta Join、Paimon Lookup，比较状态大小、checkpoint、恢复时间和一致性。
