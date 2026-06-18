> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030304_Kafka/030304_核心知识点/Kafka性能压测与批量参数调优|Kafka性能压测与批量参数调优]]
---
title: 超强总结：Kafka可以优化的方方面面
author: 大数据技能圈
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491558&idx=1&sn=e2eab6cc5ce19e324bdaad985d84a026&chksm=c1197078104bb019af1143153aa772effb75f752e64b6a4092b23e98dc563ae747d80602aea9&mpshare=1&scene=24&srcid=0710wBLOVxarB8d9p72U0Ioj&sharer_shareinfo=4b6f5997cde9c57c0d7301ec4bf5acc3&sharer_shareinfo_first=7406f266f34602c4f0684302641600da#rd
---

01

**加入知识星球《随川陪你学大数据》将获得以下权益**

扫描二维码加入星球，随加入人数增加，逐步涨价，现在就是最优惠的

01

**Kafka 优化可以从哪些方面进行？**

## 1. 生产者（Producer）优化

1.1 **批处理优化** - 调整batch.size（建议16KB-64KB）和linger.ms（建议5-100ms）参数，平衡延迟与吞吐量。高吞吐场景可增大batch.size并延长linger.ms（如64KB/50ms），低延迟场景则减小（如16KB/5ms）。注意batch.size过大会导致内存占用增加和延迟上升。
1.2 **压缩配置** - 启用消息压缩（支持snappy、lz4、gzip、zstd），推荐lz4（平衡压缩比与CPU开销）。通过compression.type=lz4配置，压缩可降低网络传输量30%-70%，尤其适合大消息场景。
1.3 **重试机制** - 设置retries（建议3-10次）和retry.backoff.ms（建议100-500ms），配合max.in.flight.requests.per.connection=1避免消息乱序。启用幂等性（enable.idempotence=true）防止重试导致的消息重复。
1.4 **分区策略** - 自定义分区器实现业务键均匀分布，避免热点分区。对时间序列数据可按时间窗口分区，对用户数据可按用户ID哈希分区。避免使用固定分区键导致单一分区过载。

## 2. 消费者（Consumer）优化

2.1 **消费组配置** - session.timeout.ms（建议30000ms）和heartbeat.interval.ms（建议3000ms，约为session.timeout的1/10）。设置max.poll.records控制单次拉取条数（如500-2000条），避免消费超时。
2.2 **批量拉取** - fetch.min.bytes（建议10240B）和fetch.max.wait.ms（建议500ms），当积累到指定大小或超时后返回数据。fetch.max.bytes（默认50MB）需大于单条最大消息尺寸，避免拉取失败。
2.3 **偏移量提交** - 自动提交（enable.auto.commit=true，auto.commit.interval.ms=5000ms）适合非关键场景；手动提交（commitSync()/commitAsync()）适合数据一致性要求高的场景，需在业务处理完成后提交。
2.4 **消费线程模型** - 每个消费线程对应一个分区，线程数不应超过分区数。使用多线程处理时需注意线程安全，可采用每个分区单线程处理模型（如KafkaConsumer多实例+单线程）。

## 3. Broker配置优化

3.1 **JVM参数调优** - 堆大小设置为物理内存的50%-75%（如16GB内存配置-Xms10G -Xmx10G），使用G1GC（-XX:+UseG1GC -XX:MaxGCPauseMillis=20）。新生代比例设为30%-40%，避免频繁YGC。
3.2 **日志刷新策略** - log.flush.interval.messages（默认9223372036854775807，建议不修改）和log.flush.interval.ms（默认null），依赖操作系统页缓存刷新。关键数据可设置log.flush.interval.ms=1000确保数据落盘。
3.3 **分区副本配置** - replication.factor（建议3）和min.insync.replicas（建议2），确保可用性与一致性平衡。acks=all时需min.insync.replicas个副本确认，避免单点故障导致写入失败。
3.4 **网络参数** - socket.send.buffer.bytes和socket.receive.buffer.bytes（建议1MB，即1048576），num.network.threads（CPU核心数+1）处理网络请求，num.io.threads（CPU核心数\*2）处理磁盘I/O。

## 4. 主题（Topic）与分区管理

4.1 **分区数量规划** - 按吞吐量需求计算（单分区写入约1000-2000条/秒，读取约5000条/秒），推荐每个Broker承载2000-4000个分区。分区数过多会增加元数据管理开销，过少则无法充分利用集群资源。
4.2 **日志保留策略** - log.retention.hours（默认168小时）和log.retention.bytes（默认-1无限制），按业务需求设置（如监控数据保留72小时，核心业务保留30天）。定期清理过期日志释放磁盘空间。
4.3 **分区重分配** - 使用kafka-reassign-partitions.sh工具平衡Broker负载，避免某台Broker分区过多。重分配时通过throttle参数限制迁移速率（如50MB/s），避免影响正常业务。
4.4 **压缩策略** - 对需要保留最新值的主题启用log.cleanup.policy=compact，设置log.cleaner.min.compaction.lag.ms（默认0）和log.cleaner.max.compaction.lag.ms（默认86400000ms）控制压缩时机。

## 5. 副本机制优化

5.1 **副本同步** - replica.lag.time.max.ms（默认30000ms），超过此时间未同步的副本将被踢出ISR。对高延迟网络环境可适当增大（如60000ms），避免频繁ISR收缩影响可用性。
5.2 **首领副本选举** - unclean.leader.election.enable=false（默认），禁止非ISR副本成为首领，确保数据一致性。仅在允许数据丢失的场景下可临时开启。
5.3 **跨机架部署** - 副本分布在不同机架（通过broker.rack配置），使用rack-aware分区分配策略，避免单机架故障导致数据不可用。

## 6. 日志存储优化

6.1 **磁盘选择** - 使用SSD（IOPS>10000）或RAID 10（避免单盘故障），挂载点使用noatime选项减少元数据更新。多块磁盘时通过log.dirs配置多个路径，Kafka会自动分散分区存储。
6.2 **日志分段** - log.segment.bytes（默认1GB），控制单个日志段大小，过小会导致文件数过多，过大会增加清理和压缩耗时。推荐设置为512MB-2GB，根据保留策略调整。
6.3 **索引优化** - log.index.size.max.bytes（默认10MB）和index.interval.bytes（默认4096B），索引大小过小将增加查找时间，过大会浪费磁盘空间。高频访问的主题可减小index.interval.bytes提升查找效率。

## 7. 元数据管理（ZooKeeper/KRaft）

7.1 **ZooKeeper连接优化** - zookeeper.session.timeout.ms（默认6000ms），建议设置为10000ms避免网络抖动导致连接断开。zookeeper.connection.timeout.ms（默认6000ms），控制初始连接超时时间。
7.2 **KRaft模式配置** - 迁移至KRaft（Kafka 2.8+支持），配置controller.quorum.voters指定控制器节点，log.dirs存储元数据日志。禁用ZooKeeper减少外部依赖，提高集群稳定性。
7.3 **元数据缓存** - broker.metadata.max.age.ms（默认300000ms），控制元数据缓存过期时间，高频变更的集群可减小（如60000ms），降低元数据不一致风险。

## 8. 网络与I/O优化

8.1 **网络线程配置** - num.network.threads（默认3）处理网络请求，建议设置为CPU核心数+1；num.io.threads（默认8）处理磁盘I/O，建议设置为CPU核心数\*2，充分利用多核资源。
8.2 **TCP参数** - tcp.send.buffer.bytes和tcp.receive.buffer.bytes（建议1MB），通过操作系统sysctl配置net.core.wmem\_max和net.core.rmem\_max确保生效。启用tcp.nodelay=true减少延迟。
8.3 **零拷贝技术** - 启用sendfile（默认开启），通过socket.send.buffer.bytes=0让操作系统直接从磁盘文件发送数据到网络，减少用户态与内核态数据拷贝，提升吞吐量。

## 9. 监控与运维优化

9.1 **指标收集** - 监控关键指标：UnderReplicatedPartitions（应=0）、ISRShrinksPerSec（越低越好）、ConsumerLag（消费者滞后消息数）、BytesIn/BytesOut（吞吐量）、LogFlushRateAndTime（刷盘性能）。
9.2 **告警配置** - 设置分区不可用（UnderReplicatedPartitions>0持续5分钟）、副本不同步（ISR Shrinks>0）、磁盘使用率>85%、网络吞吐量突降>50%等告警，及时发现异常。
9.3 **数据备份策略** - 定期备份日志数据（通过kafka-dump-log.sh导出或直接拷贝log.dirs文件），跨区域备份关键主题，制定灾难恢复演练计划，RTO<4小时，RPO<1小时。

## 10. 安全优化

10.1 认证机制 - 启用SASL/PLAIN（简单用户名密码）或SASL/GSSAPI（Kerberos）认证，配置sasl.mechanism=PLAIN，sasl.jaas.config指定认证文件。生产环境推荐使用Kerberos或OAuth2。  

10.2 授权控制 - 使用ACL限制主题访问权限，通过kafka-acls.sh配置：允许特定用户对主题执行读/写操作，禁止匿名访问（allow.everyone.if.no.acl.found=false）。  

10.3 数据加密 - 传输加密：启用SSL（ssl.enabled.protocols=TLSv1.2,TLSv1.3），配置ssl.keystore.location和ssl.truststore.location；存储加密：配合操作系统或第三方工具（如LUKS）加密磁盘数据。

##

本次优化总结详细文档已放入知识星球《随川陪你学大数据》，可扫码获取

获取更多信息，关注大数据技能圈

**欢迎添加作者交流**

## 推荐阅读系列文章

* [腾讯面试：Spark内存如何优化？包含哪几个方面？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491167&idx=1&sn=d62e305f6803f484aa40469393b1fdb8&scene=21#wechat_redirect)
* [超强总结：Spark可以优化的方方面面](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491543&idx=1&sn=e5f501540f1e4e7937216ee53e0ae8be&scene=21#wechat_redirect)
* [腾讯面试：请详细描述Paimon如何基于LSM树实现高吞吐写入和高效查询？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491536&idx=1&sn=01cb739322d9fd0e5ef4530e6970187c&scene=21#wechat_redirect)
* [腾讯面试：Flink100G大状态如何优化？有哪些参数可以调整？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491532&idx=1&sn=18c1952668c000328b2fa3081fc80906&scene=21#wechat_redirect)
* [阿里面试：Paimon QPS太低怎么优化？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491526&idx=1&sn=d6364e30ac6442c02c7a147337c4d44a&scene=21#wechat_redirect)
* [腾讯面试：Flink出现反压如何排查？有哪些参数可以调整？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491520&idx=1&sn=b8ab27aa17c78653d6598faff0bc85e9&scene=21#wechat_redirect)
* [超强总结：Iceberg可以优化的方方面面](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491505&idx=1&sn=e385a35b8d736b2cc639225b8487563c&scene=21#wechat_redirect)
* [阿里面试：Hudi，Iceberg，Paimon之间的差异有哪些？该如何选择？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491499&idx=1&sn=dc6844323e421ff97157ffe6de5b9bbb&scene=21#wechat_redirect)
* [阿里面试：如果让你负责大数据平台的架构，需要考虑哪些点？如何设计？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491478&idx=1&sn=baf72d7547630119ff4da6c0ba6b1239&scene=21#wechat_redirect)
* [阿里面试：请详细解释一下Flink内存管理，具体有哪些参数可以调整？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491468&idx=1&sn=524bcce4dc664fe78d4442daa6ab21f6&scene=21#wechat_redirect)
* [腾讯面试：介绍一下Doris问题排查思路，有没有总结过相关文档？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491462&idx=1&sn=24b0982030c010dd904b65de15bd17df&scene=21#wechat_redirect)
* [这篇文章把Paimon和Fluss的关系给彻底说清楚了](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491452&idx=1&sn=f6e98294672db035964f347a290f16f4&scene=21#wechat_redirect)
* [腾讯面试：Doris 物化视图的使用场景是怎么样的，有哪几种数据更新方式？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491386&idx=1&sn=d29b14d6912d0efdb941ae5e1dbd1dfb&scene=21#wechat_redirect)
* [腾讯面试：详细介绍Spark的Shuffle阶段数据从输入到输出经历了哪些步骤？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491384&idx=1&sn=5ab414f60fda52a3b9b88e1a7f2cdecd&scene=21#wechat_redirect)
* [腾讯面试：数仓分层架构是怎么样的？为什么要这样设计？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491372&idx=1&sn=97dfb7d58f8b889a8cbaeb4229091e98&scene=21#wechat_redirect)
* [蚂蚁面试：Flink并行度、算子、算子链、Slot、Slot共享组之间的关系是什么？如何设置能够使资源利用最大化？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491363&idx=1&sn=17d02a075e33592aabc517f974e87a02&scene=21#wechat_redirect)
* [网易面试：Hudi、Iceberg、Paimon有什么异同点？如何选型？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491351&idx=1&sn=df26d44b5c9ac16eb3be5a2f3027d5f6&scene=21#wechat_redirect)
* [Flink 反压问题深度剖析与解决方案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491343&idx=1&sn=a898b2172b5a694dfa03e356758ff08a&scene=21#wechat_redirect)
* [小米面试：Paimon Join用法有哪些？大规模数据场景下如何优化 Join 性能？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491218&idx=1&sn=b5aa336a2fb6a37d1093ec6318e8e39d&scene=21#wechat_redirect)
* [蚂蚁面试：Kafka如何做压测？如何保证系统稳定？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491146&idx=1&sn=e2c4a1e298aec2b6d4d42cffb2f331ef&scene=21#wechat_redirect)
* [字节面试：Flink如何做压测？如何保证系统稳定？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491131&idx=1&sn=6bac85b9e17620b49b37ebd3224b3a83&scene=21#wechat_redirect)
* [Flink内存调优指南（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=2&sn=cccaa1289c4581a2dd407d21a8c5ae09&scene=21#wechat_redirect)附500页16万字答案[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=2&sn=cccaa1289c4581a2dd407d21a8c5ae09&scene=21#wechat_redirect)
* [Zookeeper 经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[附1400页21万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)
* [Hbase 经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491195&idx=1&sn=0c93336f246fde472c336ce764623a75&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491195&idx=1&sn=0c93336f246fde472c336ce764623a75&scene=21#wechat_redirect)[附550页12万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491195&idx=1&sn=0c93336f246fde472c336ce764623a75&scene=21#wechat_redirect)
* [Hive经典面试题200道（附8万字420页答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491138&idx=1&sn=f1acb3164d5f0f7e8be7f2037f4a838b&scene=21#wechat_redirect)
* [Kafka 经典面试题200道（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)[附8万字420页答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491138&idx=1&sn=f1acb3164d5f0f7e8be7f2037f4a838b&scene=21&token=1460012704&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)
* [Spark经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)[附500页8万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491106&idx=1&sn=d412a81e6d5742c3e4acf6a680761139&scene=21&token=1460012704&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)
* [ElasticSearch经典面试题200道（附400页12万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491122&idx=1&sn=cb7f3563a8082d248b1a0b768bd4d567&scene=21#wechat_redirect)
* [FlinkCDC经典面试题200道（附500页8万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491106&idx=1&sn=d412a81e6d5742c3e4acf6a680761139&scene=21#wechat_redirect)
* [StarRocks](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect) [经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect)[附550页12万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect)
* [Flink源码分析 经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[附1200页32万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)
* [FlinkSQL 经典面试题200道（附1200页32万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21#wechat_redirect)
* [Paimon经典面试题200道题（附500页16万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491044&idx=1&sn=f11db4583012987748f532d11ebaf1f0&scene=21#wechat_redirect)
* [Hudi经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491340&idx=1&sn=fbfde6a8f265acaa2eb3cf320fc635fc&scene=21#wechat_redirect)[200道（附1050页39万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491037&idx=1&sn=24bcca607e01daae65783775dcc3716e&scene=21#wechat_redirect)
* [Doris经典面试题200道（附1050页39万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491037&idx=1&sn=24bcca607e01daae65783775dcc3716e&scene=21#wechat_redirect)
* [Flink经典面试题200道（附1060页26万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491335&idx=1&sn=97f9e76a359119d51ff26fd177140437&scene=21#wechat_redirect)
* [建议收藏 | Kafka 实战文章总结](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490811&idx=1&sn=8e756190799e98e0e017b5994b6d0c3b&scene=21#wechat_redirect)
* [建议收藏 | Dinky系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489859&idx=1&sn=6b59becc16f653b609d01090e40f9e20&chksm=c02962dcf75eebcabfa43a1e3e4a1b210df6ac0725685a9087984261626223cc339c0c363a24&scene=21#wechat_redirect)
* [建议收藏 | Flink系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489608&idx=1&sn=c055844c7ac20eabbe11e331f0d629c2&chksm=c02963d7f75eeac15892c32660c1e90e000ad72e66ab97ed4051273bb5e3f6299998d6c81097&scene=21#wechat_redirect)
* [建议收藏 | Flink CDC 系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489988&idx=1&sn=c57b867645834257e8567a6e671273ff&chksm=c029625bf75eeb4d29d3f43d1b845aec3a7d176cfbceddd16ae6d195d4d96fa5cf6bfa9c362d&scene=21#wechat_redirect)
* 建议收藏 | [Doris实战文章合集](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490251&idx=1&sn=ae993d282a0a3cd968c3d6a19f79dce8&chksm=c0296154f75ee8428ee77a8e0c86ca08648967e7a68987aaf79d20bdbd481f0ab6d1d2729b53&scene=21#wechat_redirect)
* [建议收藏 | Paimon 实战文章总结](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490466&idx=1&sn=c3bdd1c25d72ba89186d4ed63634f7b3&scene=21#wechat_redirect)
* [建议收藏 | Fluss 实战文章总结](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490616&idx=1&sn=20c60ca77763b23d7c5b3a8785f58007&scene=21#wechat_redirect)
* 建议收藏 | [Seatunnel 实战文章系列合集](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490357&idx=1&sn=61ae7c852b2a41e192c0797cc28fd182&chksm=c02960aaf75ee9bcd422d82ceb80362a0959df74ee7d336e0015c51b3eb6eb1a6d6774e99483&scene=21#wechat_redirect)
* 建议收藏 | [实时离线输数仓（数据湖）总结篇](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488973&idx=1&sn=42d2f8235c822030f187c0d49deb54f5&scene=21#wechat_redirect)
* [建议收藏 | 实时离线数仓实战第一阶段总](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488426&idx=1&sn=6fe3666f5157c651c08a0c8862c1efad&scene=21#wechat_redirect)结
* [超700star！电商项目数据湖建设实战代码 ，拿来即用！](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490491&idx=1&sn=9162eb30018ad3a0c5663afe1d1b0c85&scene=21#wechat_redirect)
* [从0到1建设电商项目数据湖实战教程](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490348&idx=1&sn=e7c6b0d224fea9560218213e7b2e388c&scene=21#wechat_redirect)
* [推荐一套开源电商项目数据湖建设实战代码](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489555&idx=1&sn=e923d58dbabca58d00d76cead2a9580b&scene=21#wechat_redirect)