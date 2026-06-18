# Spark AQE 与执行计划诊断

## 来源

- [自适应查询执行：在运行时提升Spark SQL执行性能](<../文章/done-自适应查询执行：在运行时提升Spark SQL执行性能.md>)
- [【组件学习】Spark AQE--Spark为了优化你的代码有多努力？](<../文章/done-【组件学习】Spark AQE--Spark为了优化你的代码有多努力？.md>)
- [【组件学习】Spark AQE-Join优化规则非深入研究](<../文章/done-【组件学习】Spark AQE-Join优化规则非深入研究.md>)
- [【组件学习】Spark AQE相关参数使用介绍](<../文章/done-【组件学习】Spark AQE相关参数使用介绍.md>)
- [【极速学习spark调优】Spark SQL 执行计划怎么看？一眼定位慢 SQL](<../文章/done-【极速学习spark调优】Spark SQL 执行计划怎么看？一眼定位慢 SQL.md>)

## 核心问题

AQE 的价值在于运行时根据真实统计信息调整计划，但它不是万能优化器。诊断慢 SQL 时要同时看初始计划、AdaptiveSparkPlan、运行时分区统计、Join 策略变化和最终指标。

## 判断准则

| AQE 能力 | 判断证据 | 边界 |
|---|---|---|
| 合并小分区 | Shuffle 分区大小、coalesce 后分区数 | 分区过少也可能降低并行度 |
| Join 策略调整 | 初始 Join 与最终 Join 对比 | 广播阈值、统计信息和 Join 类型限制 |
| 倾斜处理 | 热点分区识别、拆分策略、任务耗时改善 | 不能替代表关系和热点 Key 治理 |
| 计划诊断 | `EXPLAIN`、SQL UI、最终物理计划 | 只看静态计划会漏掉运行时变化 |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| 开 AQE 就不用管 SQL | AQE 只能在可识别的运行时统计上调计划 |
| 参数背熟就能调优 | 必须看实际计划是否变化和任务指标是否改善 |
| SQL Metrics 就是 AQE 决策数据 | UI 指标、Accumulator 和 AQE 统计来源不同 |

## 架构/流程图

```mermaid
flowchart LR
  Logical["逻辑计划"] --> Initial["初始物理计划"]
  Initial --> Runtime["运行时统计"]
  Runtime --> AQE["AQE 改写计划"]
  AQE --> Final["最终执行计划"]
  Final --> Metrics["SQL Metrics / Stage 指标"]
```

## 待验证缺口

- 需要补 AQE 开启前后计划 diff 模板。
- 需要补 AQE 误优化或结果风险的生产反例。
