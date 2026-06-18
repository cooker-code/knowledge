---
title: AutoMQ 1.1.0-RC0 重磅更新：内核升级到 Apache Kafka 3.7.0
author: AutoMQ
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247484504&idx=1&sn=09c453ea445fe791e4576510afcbdda4&chksm=c1bc2211f6cbab07f07a5eab0d9b8af27ac608b25b8074a1c289041ddf0f872b1c4030eaf943&mpshare=1&scene=24&srcid=0402AlHx4Ucsslxxpm0X5Qek&sharer_shareinfo=12224b4c1ac186643210602e97e5cef1&sharer_shareinfo_first=12224b4c1ac186643210602e97e5cef1#rd
---

AutoMQ 在 2024.02 正式发布了基于 Apache Kafka 3.4.0 的云原生重构版本 1.0.0，AutoMQ 1.0.0 版本相比原版提供了 Serverless、自动负载均衡、秒级分区迁移和 All in 对象存储能力，让 Kafka 用户能充分利用云的弹性能力和廉价存储，实现十倍成本优势。

AutoMQ 基于 Apache Kafka 3.4.0 版本开始进行 Kafka 云原生重构，在此期间 Apache Kafka 社区在 trunk 分支上提交了 1700+ 个 Commits，版本也从 3.4.0 演进到 3.7.0，**新增了大量的特性、优化和修复**。

AutoMQ 为了保障 100% 与 Apache Kafka 兼容，在 AutoMQ 1.0.0 版本稳定后，就立刻开始了对 Apache Kafka 最新代码的合并工作。得益于 AutoMQ 存算分离的架构，对 Apache Kafka 的修改主要限制在存储切面，存储代码仅占整个项目不到 2%，因此 AutoMQ 合并跟进 Apache Kafka 的代码成本很低。

3.7.0 代码合并后，经过评审、Chaos 测试和 500+ E2E 测试，我们很高兴宣布 AutoMQ 1.1.0-RC0 版本正式发布，内核版本从 3.4.0 升级到 3.7.0。

https://github.com/AutoMQ/automq/releases/tag/1.1.0-rc0

**新特性&优化**

Apache Kafka 内核从 3.4.0 升级到 3.7.0 新增重要的特性和优化如下：

**Kafka Broker, Controller, Producer, Consumer**

ꔷ KIP-881：消费者分区分配支持 Rack-aware，节省跨 AZ 流量费用（3.5.0）

ꔷ KIP-887：增加 EnvVarConfigProvider 支持从环境变量加载配置（3.5.0）

ꔷ KIP-900：kafka-storage.sh API 增加对 Kraft SCRAM 验证支持（3.5.0）

ꔷ KIP-890：事务消息服务端防御，避免特殊场景下的事务悬挂问题（3.6.0 / 3.7.0）

ꔷ KIP-797：IPv4/IPv6 支持绑定到相同端口（3.6.0）

ꔷ KIP-863：CompletedFetch#parseRecord 内存拷贝优化（3.6.0）

ꔷ KIP-868：KRaft 支持以事务模式提交一系列 Records（3.6.0）

ꔷ KIP-714：新增多语言统一的客户端 Metrics，提供从服务端查询客户端 Metrics 的能力（3.7.0）

ꔷ KIP-848：（Early Access）下一代消费者重平衡协议（3.7.0）

ꔷ KIP-951：客户端分区 Leader 发现优化，降低发送/消费的时间（3.7.0）

ꔷ KIP-580：客户端指数退避重试（3.7.0）

**Kafka Streams**

ꔷ KIP-889：新增 Versioned State Stores 提升在乱序场景下的 join 准确性（3.5.0）

ꔷ KIP-923：为流表 Join 添加宽限期提升乱序处理能力（3.6.0）

ꔷ KIP-941：Range 查询上下界支持 null（3.6.0）

ꔷ KIP-925：Kafka Streams 支持 Rack aware 任务分配（3.7.0）

ꔷ KIP-954：扩展默认的 DSL 存储配置支持自定义类型（3.7.0）

ꔷ KIP-968：Versioned State Stores 新增 IQ 支持（3.7.0）

**Kafka Connect**

ꔷ KIP-710：MirrorMaker 2.0 支持分布式模式（3.5.0）

ꔷ KIP-898：Connect 插件服务发现能力增强（3.6.0）

ꔷ KIP-976：新增集群维度动态日志等级调节（3.7.0）

ꔷ KIP-980：支持创建停止状态的 Connectors（3.7.0）

**参考资料**

[1] https://archive.apache.org/dist/kafka/3.5.0/RELEASE\_NOTES.html

[2] https://archive.apache.org/dist/kafka/3.6.0/RELEASE\_NOTES.html

[3] https://downloads.apache.org/kafka/3.7.0/RELEASE\_NOTES.html

[4] https://www.confluent.io/blog/introducing-apache-kafka-3-7/

**往期推荐**

[原理剖析: 一文搞懂 Kafka consumer 与 broker 交互机制与原理

2024-03-29](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247484467&idx=1&sn=0628fd9be91b47435ef73f8f8d849b37&chksm=c1bc227af6cbab6cebedb43a5ec07d7549b7fb4fe179f0e3f4254c30aa95545b3bfdfca1a3a5&token=201608804&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247484467&idx=1&sn=0628fd9be91b47435ef73f8f8d849b37&chksm=c1bc227af6cbab6cebedb43a5ec07d7549b7fb4fe179f0e3f4254c30aa95545b3bfdfca1a3a5&token=201608804&lang=zh_CN&scene=21#wechat_redirect")[从 Redis 开源协议变更看开源软件与云计算巨头之间的竞争博弈

2024-03-27](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247484438&idx=1&sn=68f7ed3cc78670b95216789ca399112c&chksm=c1bc225ff6cbab4961b3ffb15262943e5f3d559186778301b8126b8934da83b717b0ea8b3ab8&token=1244003501&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247484438&idx=1&sn=68f7ed3cc78670b95216789ca399112c&chksm=c1bc225ff6cbab4961b3ffb15262943e5f3d559186778301b8126b8934da83b717b0ea8b3ab8&token=1244003501&lang=zh_CN&scene=21#wechat_redirect")[原理剖析：AutoMQ 如何基于裸设备实现高性能的 WAL

2024-03-22](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247484374&idx=1&sn=5fbe4f7f292225c29a7b6b137852cd81&chksm=c1bc259ff6cbac890dfb04c5ff61775c68c62918db16f950f409b778d7148a7f5aa7bc55caf9&token=201608804&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247484374&idx=1&sn=5fbe4f7f292225c29a7b6b137852cd81&chksm=c1bc259ff6cbac890dfb04c5ff61775c68c62918db16f950f409b778d7148a7f5aa7bc55caf9&token=201608804&lang=zh_CN&scene=21#wechat_redirect")[Kafka 痛点专题｜AutoMQ 如何解决 Kafka 冷读副作用

2024-03-15](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247484316&idx=1&sn=ea660e4126f06b5dc713d6d65397c9ff&chksm=c1bc25d5f6cbacc380e2afef4d381cbe96116c0b7fe8cbbeab564f234eac48940031eda03071&token=1244003501&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247484316&idx=1&sn=ea660e4126f06b5dc713d6d65397c9ff&chksm=c1bc25d5f6cbacc380e2afef4d381cbe96116c0b7fe8cbbeab564f234eac48940031eda03071&token=1244003501&lang=zh_CN&scene=21#wechat_redirect")

END

**关于我们**

我们是来自 Apache RocketMQ 和 Linux LVS 项目的核心团队，曾经见证并应对过消息队列基础设施在大型互联网公司和云计算公司的挑战。现在我们基于对象存储优先、存算分离、多云原生等技术理念，重新设计并实现了 Apache Kafka 和 Apache RocketMQ，带来高达 10 倍的成本优势和百倍的弹性效率提升。

🌟 GitHub 地址：https://github.com/AutoMQ/automq

💻 官网：https://www.automq.com

👀 B站：AutoMQ官方账号

🔍 视频号：AutoMQ

👉🏻 **扫描二维码**

**加入我们的社区群**

关注我们，一起学习更多云原生技术干货！

👇🏻点击下方阅读原文，前往 GitHub 了解体验！