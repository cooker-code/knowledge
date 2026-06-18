# Kafka副本一致性与ISR边界

## 来源
- [Kafka：ISR 如何守住副本一致性](<../文章/done-Kafka：ISR 如何守住副本一致性.md>)
- [Kafka的一条消息的写入和读取过程原理介绍](../文章/done-Kafka的一条消息的写入和读取过程原理介绍.md)
- [kafka设计与编码](../文章/done-kafka设计与编码.md)

## 核心问题

Kafka 的可靠性不是“多副本就安全”，而是由 Leader/Follower、ISR、acks、HW/LEO、选主策略和副本落后判定共同决定。

## 判断准则

| 判断项 | 准则 |
|---|---|
| 写入可靠性 | `acks=all` 只有在 ISR 管理、最小 ISR 和副本同步健康时才有意义 |
| 消费可见性 | 消费者只能看到高水位线之前的数据，避免读到未被足够副本确认的消息 |
| 故障风险 | ISR 缩小、Leader 切换、磁盘/网络抖动会改变可用性和一致性权衡 |
| 运维指标 | 关注 under replicated partitions、ISR shrink/expand、request latency、磁盘和网络瓶颈 |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| 副本数越高越安全 | 副本数还要配合 ISR、机架分布、磁盘和网络，否则只是增加成本 |
| `acks=all` 保证业务不丢 | 它只保证 Kafka 复制层确认，不保证下游处理和业务幂等 |

## 待验证缺口

- 构造三副本 Topic，模拟 Follower 延迟、Leader 宕机和 ISR 收缩，观察生产者延迟与消费者可见性。
