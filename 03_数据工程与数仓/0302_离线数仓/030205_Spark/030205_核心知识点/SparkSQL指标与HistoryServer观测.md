# Spark SQL 指标与 History Server 观测

## 来源

- [Spark SQL 的 100+ 指标，你真的看懂了吗？](<../文章/done-Spark SQL 的 100+ 指标，你真的看懂了吗？.md>)
- [SQL Metrics 和 AQE 用的竟然不是同一套数据](<../文章/done-SQL Metrics 和 AQE 用的竟然不是同一套数据.md>)
- [一站式解析Spark 日志 ，助力开发Spark任务调优诊断](<../文章/done-一站式解析Spark 日志 ，助力开发Spark任务调优诊断.md>)
- [让 Spark History Server 对 AI Agent 友好](<../文章/done-让 Spark History Server 对 AI Agent 友好.md>)

## 核心问题

Spark 观测不是“看 UI”。SQL Metrics、Stage/Task 指标、EventLog、History Server REST API 和 AQE 运行时统计各自来源不同。要做自动诊断或 Agent 工具，必须先统一指标语义。

## 判断准则

| 指标来源 | 能回答什么 | 注意点 |
|---|---|---|
| SQL Metrics | 算子行数、数据量、耗时、spill 等 | 显示格式和聚合方式要理解 |
| Stage/Task | 任务耗时分布、Shuffle、GC、失败重试 | 适合定位长尾和资源瓶颈 |
| EventLog | 历史作业完整事件 | 数据量大，解析成本和字段语义要治理 |
| History Server REST | 应用、Job、Stage、SQL、Executor 查询 | 适合工具化和 Agent 查询 |
| AQE 统计 | 运行时改写依据 | 不等于 UI 上所有 SQL Metrics |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| UI 看起来慢就能判断原因 | 需要把 SQL、Stage、Task 和 Executor 指标串起来 |
| Agent 读取 History Server 就能自动调优 | 先要定义指标语义、阈值和诊断规则 |
| SQL Metrics 和 AQE 统计是一回事 | 两者生命周期和用途不同 |

## 待验证缺口

- 需要补 Spark History Server REST API 到诊断字段的映射表。
- 需要补慢 SQL 诊断规则：扫描大、Shuffle 大、倾斜、spill、广播失败、OOM。
