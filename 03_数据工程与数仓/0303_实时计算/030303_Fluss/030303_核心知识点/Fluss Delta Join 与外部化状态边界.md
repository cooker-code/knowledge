# Fluss Delta Join 与外部化状态边界

> 验证版本：需结合 Flink 2.1+ 与 Fluss 官方文档继续补证

## 来源
- [Fluss 到底有何特别？Flink 的 Delta Join 来了！](<../文章/done-Fluss 到底有何特别？Flink 的 Delta Join 来了！.md>)
- [官宣 _ Apache Fluss (Incubating) 0.8 发布公告](<../文章/done-官宣 _ Apache Fluss (Incubating) 0.8 发布公告.md>)

## 核心问题

传统双流 Join 会把大量历史数据压进 Flink State，导致 checkpoint、恢复和扩缩容成本暴涨。Delta Join 的核心思路是把 Join 依赖的历史状态外置到 Fluss 表，Flink 只处理增量和异步探查。

## 判断准则

| 判断项 | 准则 |
|---|---|
| 适用场景 | 长窗口、宽表拼接、状态过大、重启恢复慢、需要实时点查的 Join 链路 |
| 前置约束 | 源表到 Join 之间不能随意引入改变 Key 语义的有状态算子；异步探查要保证同 Key 顺序 |
| 一致性理解 | Delta Join 更接近“基于外部最新状态的最终一致探查”，不是把传统 Join 的每个历史组合都完全保留 |
| 收益验证 | 不能只看宣传收益，要对比状态大小、checkpoint 时长、恢复耗时、补数据方式和错误重放成本 |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| Delta Join 让 Flink 不再需要状态 | 它只是把特定 Join 历史外置，Flink 仍要维护算子、窗口、异步请求和一致性控制 |
| 状态外置一定更快 | 外部点查会引入网络、索引、热点和可用性问题，收益依赖访问模式 |
| 所有 Join 都适合 Delta Join | 只有能接受外部最新状态探查语义、且状态成本成为主要瓶颈时才适合 |

## 待验证缺口

- 官方 FLIP、Flink 版本、Fluss Connector 配置项和失败恢复语义。
- 与 Paimon Lookup Join、传统 Flink State Join 的同数据集对比。
