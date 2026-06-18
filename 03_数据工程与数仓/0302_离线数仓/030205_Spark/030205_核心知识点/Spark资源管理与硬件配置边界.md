# Spark 资源管理与硬件配置边界

## 来源

- [对Spark硬件配置的建议](<../文章/done-对Spark硬件配置的建议.md>)
- [Pinterest 基于 Apache YuniKorn 对 Apache Spark 进行资源管理实践](<../文章/done-Pinterest 基于 Apache YuniKorn 对 Apache Spark 进行资源管理实践.md>)
- [Spark调优——分布式离线计算实战秘籍](<../文章/done-Spark调优——分布式离线计算实战秘籍.md>)
- [万字详解 Spark Core 开发调优（建议收藏）](<../文章/done-万字详解 Spark Core 开发调优（建议收藏）.md>)
- [三万字长文 _ Spark性能优化实战手册](<../文章/done-三万字长文 _ Spark性能优化实战手册.md>)

## 核心问题

Spark 资源配置没有通用答案。Executor 内存、CPU、本地盘、网络、并行度和资源调度策略必须服务具体作业形态：大 Shuffle、宽表写入、迭代计算、SQL 批处理、并发队列或交互式查询。

## 判断准则

| 资源 | 判断维度 | 典型风险 |
|---|---|---|
| CPU | Task 并行度、算子 CPU 密集度 | core 太多导致单 Executor GC/内存争抢 |
| 内存 | 执行内存、缓存、广播、聚合、排序 | OOM、spill、GC 时间高 |
| 本地盘 | Shuffle、spill、临时文件 | Fetch Failure、磁盘打满、I/O 抖动 |
| 网络 | Shuffle Read/Write、远程存储 | 拉取失败、网络瓶颈 |
| 调度 | 队列、公平性、SLO、YuniKorn/K8s/YARN | 资源饥饿、优先级错配 |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| 给更多 Executor 就更快 | 并行度、Shuffle、IO 和调度开销可能抵消收益 |
| 硬件建议可以固定套用 | 必须绑定数据规模、算子、Shuffle 和存储路径 |
| 资源调度只看集群利用率 | 还要看作业 SLO、公平性、队列隔离和失败恢复 |

## 待验证缺口

- 需要补 Spark 作业资源画像模板：输入量、Shuffle 量、Task 数、spill、GC、输出文件数。
- 需要补 YARN/K8s/YuniKorn 场景下同一作业资源表现对比。
