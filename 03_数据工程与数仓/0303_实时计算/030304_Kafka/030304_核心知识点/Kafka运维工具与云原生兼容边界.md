# Kafka运维工具与云原生兼容边界

## 来源
- [5分钟部署自己的Kafka可视化平台](../文章/done-5分钟部署自己的Kafka可视化平台.md)
- [Kafbat UI：Apache Kafka 集群可视化管理利器](../文章/done-Kafbat UI：Apache Kafka 集群可视化管理利器.md)
- [如何用Know Streaming对Kafka Topic完成快速扩缩副本操作](../文章/done-如何用Know Streaming对Kafka Topic完成快速扩缩副本操作.md)
- [Kafka 4.0 集群部署文档（K8S模式）](<../文章/done-Kafka 4.0 集群部署文档（K8S模式）.md>)
- [AutoMQ 1.1.0-RC0 重磅更新：内核升级到 Apache Kafka 3.7.0](<../文章/done-AutoMQ 1.1.0-RC0 重磅更新：内核升级到 Apache Kafka 3.7.0.md>)
- [AutoMQ 携手阿里云共同发布新一代云原生 Kafka，帮助得物有效压缩 85% Kafka 云支出！](<../文章/done-AutoMQ 携手阿里云共同发布新一代云原生 Kafka，帮助得物有效压缩 85% Kafka 云支出！.md>)

## 核心问题

Kafka 运维工具和云原生兼容方案解决的是“可视化、扩缩容、副本迁移、K8s 部署、存储成本”问题，但它们不能改变 Kafka 协议和业务一致性的基本约束。

## 判断准则

| 场景 | 判断 |
|---|---|
| 可视化平台 | 适合查看 Topic、Group、Lag、消息样例和权限，不能替代告警、容量规划和变更审批 |
| 副本扩缩 | 必须关注副本迁移流量、ISR、Leader 分布和磁盘水位 |
| K8s 部署 | StatefulSet、持久卷、网络标识和滚动升级要和 broker 身份绑定 |
| Kafka 兼容云原生 | AutoMQ 等方案要单独验证存储语义、延迟、冷读、副本模型和 Kafka 客户端兼容 |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| 有 UI 就等于可治理 | UI 是入口，治理还要靠权限、审计、告警、容量和流程 |
| Kafka 兼容就等于无迁移风险 | 协议兼容不代表存储、延迟、运维、故障恢复完全一致 |

## 待验证缺口

- 对比原生 Kafka、Strimzi、AutoMQ 的部署、扩容、故障恢复和冷读行为。
