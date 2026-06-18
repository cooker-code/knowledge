# Kafka安全认证鉴权边界

## 来源
- [Apache Kafka 中的认证、鉴权原理与应用](../文章/done-Apache Kafka 中的认证、鉴权原理与应用.md)
- [Kafka配置SASL_PLAINTEXT安全认证协议详解](../文章/done-Kafka配置SASL_PLAINTEXT安全认证协议详解.md)

## 核心问题

Kafka 安全治理要同时处理“你是谁”“你能访问什么”“传输是否加密”“审计是否可追踪”。只配置 SASL 或只加 ACL 都不完整。

## 判断准则

| 层次 | 关注点 |
|---|---|
| 认证 | SASL/SSL/Kerberos/OAUTH 等机制确认客户端身份 |
| 鉴权 | ACL 控制 Topic、Group、Cluster、TransactionalId 等资源动作 |
| 传输安全 | PLAINTEXT 只做身份认证不加密；跨网络和敏感数据应评估 SSL |
| 运维治理 | 权限要按生产者、消费者、运维工具和流作业分别授予，避免通配符泛化 |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| SASL_PLAINTEXT 就安全了 | 它解决认证，不解决传输加密 |
| Kafka ACL 只管 Topic | Group、Cluster、TransactionalId 等资源也会影响消费和事务能力 |

## 待验证缺口

- 建一套最小 ACL 样例，覆盖生产、消费、事务写入、运维查看和错误权限回收。
