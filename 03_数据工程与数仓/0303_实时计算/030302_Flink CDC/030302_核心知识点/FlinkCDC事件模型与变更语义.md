# Flink CDC 事件模型与变更语义

## 原文锚点

- 本地文件：[FlinkCdc中Event最详解，你不知道了吧？](../文章/done-FlinkCdc中Event最详解，你不知道了吧？.md)
- 原文链接：`http://mp.weixin.qq.com/s?__biz=MzA3OTExMjA0MA==&mid=2247485253&idx=1&sn=9745e1473d66641c745584622fb25b4e&chksm=9e2b93335fb0a0cd7c8f6f15700f940ae372b70665300af7853adb5d9f8fbd9ee42bc9b814f0`
- 关键段落：DataChangeEvent、SchemaChangeEvent、before/after/op/source 元数据解释。
- 关键图：无。

## 图片处理

| 图片 | 类型 | 是否保留 | 理由 | 处理方式 |
|---|---|---|---|---|
| 无 | - | 不适用 | 原文以字段解释和 JSON 示例为主 | 不进入知识点 |

## 一句话结论

这篇文章有价值但不完整：它能补 Flink CDC 事件字段的基本认知，尤其是 DataChangeEvent、SchemaChangeEvent、before/after/op/source；但缺少版本边界、事务顺序、下游消费和恢复场景，不能直接当生产事件语义说明。

## 用户相关性判断

| 项 | 内容 |
|---|---|
| 用户当前认知层级 | Flink CDC / 实时计算 L2 draft |
| 认知成熟度 | draft |
| 阅读投入建议 | 精读 |
| 阅读投入理由 | 能补旧总览中 C02 事件语义缺口，但缺少实践验证和版本说明 |
| 对用户的新信息 | CDC 事件要拆成数据变更、Schema 变更、源端位点和处理时间，不能只看业务字段 |
| 问题指纹 | Flink CDC + 事件模型 + DataChangeEvent/SchemaChangeEvent/before/after/op/source + 变更表达 + 下游正确消费边界 |
| 排重判断 | 新建 |
| 置信度 | 中 |

## 认知校准点

| 校准点 | 文章观点/信息 | 与用户认知或价值观的关系 | 处理建议 |
|---|---|---|---|
| CDC 事件不是普通 JSON | 事件包含表标识、前后镜像、操作类型和 source 元数据 | 补充 Flink CDC 下游一致性判断框架 | 写入事件语义节点 |
| SchemaChangeEvent 要和 DataChangeEvent 分开看 | DDL 变化也会作为事件进入链路 | 补齐 Schema 演进入口 | 后续和 Pipeline/下游表演进文章合并验证 |
| source 元数据是恢复和排障锚点 | `file`、`pos`、`gtid`、`server_id`、`snapshot` 等字段描述源端位置 | 补充排障边界 | 后续结合 server_id、WAL、checkpoint 恢复实验验证 |
| 原文缺少版本与事务边界 | 只解释单条事件样例，没有说明 Flink CDC 版本、事务顺序、乱序和 Sink 消费差异 | 证据不足 | 不把它提升为完整事件语义结论 |

## 冲突点

| 冲突类型 | 具体表现 | 影响 | 处理 |
|---|---|---|---|
| 原目录冲突 | 原文原在实时计算根文章池，未进入 Flink CDC 技术节点 | 容易被当成普通 Flink 文章 | 迁入实时计算 / Flink CDC |
| 证据不足 | 没有说明 Flink CDC 版本、内部类版本、事务边界和 Sink 行为 | 不能直接做生产语义准则 | 标记为事件字段认知补充 |
| 图片缺失 | 无架构图或事件流图 | 对链路理解帮助有限 | 后续可用 Mermaid 重建 |

## 待吸收点

| 分级 | 内容 | 为什么值得吸收 | 后续动作 |
|---|---|---|---|
| 理解 | DataChangeEvent 由表标识、before、after、operation type、meta 组成 | 这是判断 insert/update/delete 的基本结构 | 合并到 C02 事件语义 |
| 理解 | SchemaChangeEvent 包含 AddColumn、AlterColumnType、CreateTable、DropColumn、RenameColumn | CDC 链路不仅传数据，也要处理 DDL | 后续补 Schema 演进限制 |
| 记住 | `before=null/after=新值` 表示插入，`before=旧值/after=null` 表示删除，更新要同时看 before 和 after | 下游 Upsert、Delete、审计和对账都依赖这个判断 | 写入下游消费准则 |
| 记住 | `source.file`、`source.pos`、`source.gtid`、`source.snapshot` 是定位快照、binlog 位点和恢复问题的锚点 | 影响延迟、断点续传和重复/丢失排障 | 和故障恢复实验联动 |

## 已知可跳过

| 内容 | 跳过理由 |
|---|---|
| CDC 是 Change Data Capture | 用户大概率已知 |
| 单条 JSON 字段逐个翻译 | 只保留会影响排障和一致性的字段 |
| 文末宣传语 | 无技术沉淀价值 |

## 实践门槛

| 门槛 | 判断 | 证据 |
|---|---|---|
| 可运行 | 否 | 原文只有样例 JSON，没有最小作业 |
| 可验证 | 否 | 没有输入、输出、对账方式 |
| 可排障 | 部分 | source 元数据可作为排障锚点，但没有故障场景 |
| 可迁移 | 是 | 事件字段判断可迁移到 Kafka、Doris、StarRocks、Paimon Sink |
| 结论 | 降为精读 | 适合作为事件语义补充，不是实践 SOP |

## 归类判断

| 项 | 内容 |
|---|---|
| 技术本体 | Flink CDC 事件模型 |
| 文章主问题 | Flink CDC 如何表示数据变更和 Schema 变更 |
| 使用场景 | CDC Source 到下游 Sink 的事件传递和消费 |
| 关键词干扰 | Flink、Event、JSON 容易被看成普通流处理或格式说明 |
| 最终归类 | 数据工程与数仓 / 实时计算 / Flink CDC |
| 归类理由 | 主问题是 CDC 事件语义，属于 Flink CDC 实时接入链路，不是 Flink 通用状态、窗口或 Join |

## 技术定位

| 项 | 内容 |
|---|---|
| 技术类型 | 事件模型 / 变更语义 |
| 所属领域 | 数据工程与数仓 |
| 二级类目 | 实时计算 |
| 全局架构位置 | 源端日志捕获之后、下游写入之前的事件表达层 |
| 涉及模块 | Source Connector、Schema Evolution、Sink、Checkpoint 恢复 |
| 解决问题 | 如何把数据库 insert/update/delete/DDL 变成可被下游正确消费的事件 |
| 原文局限 | 缺少版本、事务、乱序、恢复和不同 Sink 消费语义 |
| 我的结论 | 以后关注；作为 Flink CDC 事件语义入口，后续需要实验补证 |

## 纵向理解

| 维度 | 判断 |
|---|---|
| 全局架构 | 数据库日志 -> Flink CDC Source -> DataChangeEvent / SchemaChangeEvent -> Transform/Route -> Sink |
| 本文位置 | 只讲事件字段，不讲 Source 枚举、Checkpoint、Pipeline 或 Sink 实现 |
| 核心机制 | 通过 before/after/op/source 描述变更内容和源端位点 |
| 使用链路 | 下游消费时要按 op 判断写入、更新、删除，并用 source 元数据辅助恢复和排障 |
| 前置条件 | 源表主键、日志位点、Schema 演进策略和目标端 Upsert/Delete 能力要清楚 |
| 边界 | 不能只凭单条事件样例判断端到端一致性；事务顺序、重复消费、无主键表仍需验证 |

## 横向对标

| 对标技术 | 实现方式 | 优势 | 劣势 | 适合场景 |
|---|---|---|---|---|
| Debezium Event | before/after/source/op 事件格式 | 生态成熟，Kafka Connect 体系清晰 | 与 Flink 作业治理分离 | Kafka 中心化 CDC |
| Flink CDC Event | DataChangeEvent / SchemaChangeEvent + Pipeline Runtime | 贴近 Flink Pipeline、整库同步和下游 Sink | 版本演进和 Sink 语义要细查 | Flink 实时链路和湖仓同步 |
| Canal Event | MySQL binlog 解析事件 | MySQL 场景轻量 | 多源和 Schema 演进弱 | 轻量 MySQL 增量订阅 |

## 后续追查

- 关键词：DataChangeEvent、SchemaChangeEvent、before、after、op、source、snapshot、gtid、binlog file、pos、Schema Evolution。
- 相关技术：Debezium Event、Kafka `debezium-json`、Paimon Upsert、Doris/StarRocks Primary Key Sink。
- 需要补读的文章：Flink CDC 3.x 事件模型官方文档、Schema Evolution 文档、Kafka/Paimon/Doris Sink 对 update/delete 的处理规则。
