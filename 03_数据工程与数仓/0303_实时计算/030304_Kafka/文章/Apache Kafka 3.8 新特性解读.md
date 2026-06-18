---
title: Apache Kafka 3.8 新特性解读
author: AutoMQ
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247486086&idx=1&sn=4c4c1393ff6152c14e4e637b70993382&chksm=c1bc2ccff6cba5d9cbd28216ed7a5e70554ff1b99496b2bcb5323d5d7e351fc1193ee7209cb4&mpshare=1&scene=24&srcid=08160fl0NEYSTlOpIIOzIQjJ&sharer_shareinfo=6dec7a52968d182f4a08afa37ab26e01&sharer_shareinfo_first=6dec7a52968d182f4a08afa37ab26e01#rd
---

2024年7.29日 Apache Kafka 3.8 正式发布[1]。该版本包含了诸多新特性与多达456个来自JIRA 的改进与修复[2]。AutoMQ 作为云原生的 Apache Kafka 替代产品，可以保证对 Apache Kafka 的 100% 完全兼容，因此也会尽快合并 Kafka 上游社区3.8的最新改动。这篇文章对 Apache Kafka 3.8 的主要新特性做一个简单总结和解读。

**01**

**Kafka Core**

**1.1****KIP 974 GraalVM Docker Image**

GraavlVM[4] 是一款由 Oracle 推出的高性能、低资源消耗、快速启动的 JDK。利用AOT编译生成 Native Image 使得Java拥有更快的启动速度和更低的内存消耗，从而更好的适应云原生时代。

KIP 974[7] 为 Apache Kafka 提供了一个使用 KRaft 模式，基于 GraalVM 的 Native Docker Image。采用该 Native Image，可以将 Broker 的启动时间缩小到 140ms 以内(原来差不多需要3s)。更快的Broker启动速度可以使得开发者需要运行大批量的 Broker 测试的场景时更有效率。

Tips: 该特性只是当前服务于测试，不要在生产环境下使用。

**1.2 KIP-1028 Official Docker Image**

KIP-1028 引入了 JVM-Based 的 Docker Official Image(DOI)。Docker hub 本身也包含由 Apache 开源社区维护的镜像例如 apache/kafka:3.8.0。但是由 Apache Kafka 社区维护并且通过 Docker 官方发布的DOI因为其构建流程更加严格透明，因此具有更好的安全性。此外由于是 Docker 官方镜像，在 Docker Hub 上也更加容易被用户检索到和使用。

**1.3 KIP-****848 The Next Generation of the Consumer Rebalance Protocol(Preview)**

这是一个比较重要的特性。过去 Kafka 消费者的重平衡协议主要问题是：

* **依赖胖客户端**：必须依赖客户端日志来排查重平衡问题，在云时代比较麻烦。新协议会把客户端的复杂性全部转移到group coordinator。
* **依赖消费者组维度的同步屏障**：单个行为不当的消费者可能会破坏或扰乱整个组，因为每当消费者加入、离开或失败时，都需要对整个组进行重新平衡。此外，在消费者等待重新平衡完成时，无法提交偏移量。新协议不再依赖全局同步屏障，如果消费者的分配没有改变，则消费者不会受到重新平衡的影响。
* **过于复杂有历史包袱**: 原来做了很多改进优化使得重平衡协议变得很复杂。此外还把组协议用于成员之间的一般状态传播，职能的扩散引发了新的复杂度以及歧义。新协议会解决这些历史问题。

**1.4 KIP-****719  Deprecate Log4J Appender**

3.8 以前是 log4j 和 log4j2 共存，一些情况下导致日志无法正确输出。3.8 将彻底弃用 log4j-appender 并将所有省略的模块（即工具、trogdor 和 shell）升级到 log4j2，以从类路径中完全删除 log4j 工件并避免上述问题。3.8在Kafka层面已经不依赖log4j了，但是用户还是可以使用log4j，但是会有warning，按照社区计划，预计是4.0 把log4j从项目真正的删除。

```
log4j-appender is deprecated and will be removed in a future release. For migration, please refer to the latest documentation.
```

**1.5 KIP-****390 Support Compression Level**

开启压缩会影响生产者发送消息的效率。3.8 以前 Apache Kafka 没有提供压缩级别的配置，这使得用户无法自定义调整压缩率和性能之间的权衡。3.8 以后用户可以针对不同的压缩编码方式设置不同的压缩级别。

配置压缩级别方式如下：

```
 compression.type=gzipcompression.gzip.level=4                # NEW: Compression level to be used.
```

不同压缩算法对应可选的压缩级别

**1.6 KIP-993 Allow restricting files accessed by File and Directory ConfigProviders**

安全性的改进。过去 Kafka 对于访问的文件和目录是无限制的，现在可以做一些限制使得其更加安全：

```
 config.providers=directory  
config.providers.directory.class=org.apache.kafka.connect.configs.DirectoryConfigProvider  
config.providers.directory.param.allowed.paths=/var/run,var/configs
```

**1.7 KIP-1****036 Extend RecordDeserializationException exception**

拓展 KIP-334，针对 RecordDeserializationException 这个异常额外增加record的内容和元信息，使得下游接收这个报错后可以更加容易去实现类似死信队列这样的能力。

**1.8 KIP-1028 Official Docker Image**

给 KafkaMetric 类提供一个新的方法用于判断 value provider是否是 measurable 的。相比原来的measurable的好处是，不需要依赖异常来确认是否为 mesurable，并且也避免了去访问私有字段。

```
 /** * The method determines if the metric value provider is of type Measurable. * * @return true if the metric value provider is of type Measurable, false otherwise. */public boolean isMeasurable();
```

**02**

**Kafka Streams**

**2.1 KIP-989  Improved StateStore Iterator metrics for detecting leaks**

是针对 Kafka Streams 的一个优化。通过引入一些新的metric，方便开发者可以排查 Iterator的泄露。

**2.2****KIP-924 customizable task assignment for Streams**

Task Assignor 是 KStream里面负责给节点分配任务的组件[5]。3.8 添加一组新的可配置接口，用于将自定义行为插入 Streams Partition Assignor来使用自定义的 Task Assignor 从而带来更好的灵活性。

**2.3 KIP****-813 Shareable State Stores**

State Stores 在 KStream 里面用来保存流应用的状态数据[5]。State Stores 底层也是通过一个 topic 来实现的。过去这个 topic 不能在不同的 stream task 之间被共享，现在则可以被共享来减少数据冗余改善性能。

**03**

**Kafka Connect**

**3.1 KIP-1004 Enforce tasks.max property in Kafka Connect**

过去通过 tasks.max 来限制 Kafka connect 生成的任务是无效的，在3.8以后可以通过该参数做强置限制。属于修复了一个历史遗留的BUG。

**参考资料**

[1] APACHE KAFKA 3.8.0 RELEASE ANNOUNCEMENT: https://kafka.apache.org/blog#apache\_kafka\_380\_release\_announcement

[2] Apache Kafka 3.8.0 Released: https://aiven.io/blog/apache-kafka-380-released

[3] AutoMQ: https://www.automq.com/

[4] GraalVM: https://www.graalvm.org/

[5]Stanislav Kozlovski Apache Kafka 3.8.0 Released: https://www.linkedin.com/posts/stanislavkozlovski\_kafka-38-ugcPost-7224074556527759360-KX3w/?utm\_source=share&utm\_medium=member\_desktop

[6] GraalVM: The future of JVM languages:https://medium.com/@ahmedeeldeeb/graalvm-the-future-of-jvm-languages-350fa892ae45

[7] KIP-974: https://cwiki.apache.org/confluence/display/KAFKA/KIP-974%3A+Docker+Image+for+GraalVM+based+Native+Kafka+Broker

[8] KIP-1028: https://cwiki.apache.org/confluence/display/KAFKA/KIP-1028%3A+Docker+Official+Image+for+Apache+Kafka

[9] KIP-848:  https://cwiki.apache.org/confluence/display/KAFKA/KIP-848%3A+The+Next+Generation+of+the+Consumer+Rebalance+Protocol

[10] KIP-719:https://cwiki.apache.org/confluence/display/KAFKA/KIP-719%3A+Deprecate+Log4J+Appender

[11] KIP-390: https://cwiki.apache.org/confluence/display/KAFKA/KIP-390%3A+Support+Compression+Level

[12] KIP-993: https://cwiki.apache.org/confluence/display/KAFKA/KIP-993%3A+Allow+restricting+files+accessed+by+File+and+Directory+ConfigProviders

[13] KIP 1036: https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=301795741

[14] KIP-1019: https://cwiki.apache.org/confluence/display/KAFKA/KIP-1019%3A+Expose+method+to+determine+Metric+Measurability

[15] KIP-989:https://cwiki.apache.org/confluence/display/KAFKA/KIP-989%3A+Improved+StateStore+Iterator+metrics+for+detecting+leaks

[16] KIP-924: https://cwiki.apache.org/confluence/display/KAFKA/KIP-924%3A+customizable+task+assignment+for+Streams

[17] KIP-813 https://cwiki.apache.org/confluence/display/KAFKA/KIP-813%3A+Shareable+State+Stores

[18] KIP-1004 https://cwiki.apache.org/confluence/display/KAFKA/KIP-1004%3A+Enforce+tasks.max+property+in+Kafka+Connect

**往期推荐**

[AWS 弹性伸缩特性介绍

2024-07-29](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247485899&idx=1&sn=c5a282d09954d39b2c973cb32795b355&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247485899&idx=1&sn=c5a282d09954d39b2c973cb32795b355&scene=21#wechat_redirect")

[AutoMQ 开源可观测性方案：夜莺 Flashcat

2024-07-26](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247485898&idx=1&sn=0750ff27e8659bbcc72834ecb4c49beb&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247485898&idx=1&sn=0750ff27e8659bbcc72834ecb4c49beb&scene=21#wechat_redirect")

[如何通过 CloudCanal 实现从 Kafka 到 AutoMQ 的数据迁移

2024-07-25](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247485897&idx=1&sn=41812116d65e07b2425fab25f43f08c7&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247485897&idx=1&sn=41812116d65e07b2425fab25f43f08c7&scene=21#wechat_redirect")

[百行代码实现 Kafka 运行在 S3 之上

2024-07-23](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247485847&idx=1&sn=ff79076e35ca0ec840894098b18cfa42&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247485847&idx=1&sn=ff79076e35ca0ec840894098b18cfa42&scene=21#wechat_redirect")

END

**关于我们**

我们是来自 Apache RocketMQ 和 Linux LVS 项目的核心团队，曾经见证并应对过消息队列基础设施在大型互联网公司和云计算公司的挑战。现在我们基于对象存储优先、存算分离、多云原生等技术理念，重新设计并实现了 Apache Kafka 和 Apache RocketMQ，带来高达 10 倍的成本优势和百倍的弹性效率提升。

🌟 GitHub 地址：https://github.com/AutoMQ/automq

💻 官网：https://www.automq.com

👀 B站：AutoMQ官方账号

🔍 视频号：AutoMQ

👉🏻 **扫二维码**

**加入我们的社区群**

关注我们，一起学习更多云原生技术干货！

👇🏻点击下方阅读原文，前往 GitHub 了解体验！