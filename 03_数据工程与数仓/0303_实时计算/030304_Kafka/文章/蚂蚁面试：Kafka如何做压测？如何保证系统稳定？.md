---
title: 蚂蚁面试：Kafka如何做压测？如何保证系统稳定？
author: 大数据技能圈
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491146&idx=1&sn=e2c4a1e298aec2b6d4d42cffb2f331ef&chksm=c1ed92b7ee4fe9fae39da3b2bb64407f176cfb26aaf7e5f11e2f5c5df7d9d89fa38823bd6275&mpshare=1&scene=24&srcid=0508m4uHQQavVfB0rAIytrib&sharer_shareinfo=ac99ba72108e2bafdac8f07379fca63f&sharer_shareinfo_first=ac99ba72108e2bafdac8f07379fca63f#rd
---

Kafka是大数据领域应用非常广泛的消息中间件，如何确定Kafka集群的最大吞吐量和延迟呢？又如何保证Kafka集群的稳定呢？今天我们来介绍Kafka压测方案，来确认Kafka集群的各类指标。

01

**Kafka自带性能测试工具**

Kafka提供了内置的性能测试工具，可以用于生产者和消费者的基准测试：

* 生产者性能测试工具：kafka-producer-perf-test.sh
* 消费者性能测试工具：kafka-consumer-perf-test.sh

第三方压测工具JMeter

* 可以使用JMeter的Kafka插件进行压测Tsung
* 支持Kafka协议的分布式压测工具Gatling
* 可以通过Kafka插件进行压测

02

**压测场景设计**

01

生产者性能测试

测试不同消息大小、批处理设置和压缩算法对生产者性能的影响：

```
# 测试100字节消息，无压缩  /opt/kafka/bin/kafka-producer-perf-test.sh \    --topic test-topic \    --num-records 10000000 \    --record-size 100 \    --throughput -1 \    --producer-props bootstrap.servers=broker1:9092,broker2:9092,broker3:9092 \    acks=1 \    batch.size=16384 \    linger.ms=0 \    compression.type=none    
# 测试1KB消息，使用lz4压缩  /opt/kafka/bin/kafka-producer-perf-test.sh \    --topic test-topic \    --num-records 10000000 \    --record-size 1024 \    --throughput -1 \    --producer-props bootstrap.servers=broker1:9092,broker2:9092,broker3:9092 \    acks=1 \    batch.size=65536 \    linger.ms=10 \    compression.type=lz4
```

02

消费者性能测试

测试不同消费者组配置和分区数对消费性能的影响：

```
# 基本消费者性能测试  /opt/kafka/bin/kafka-consumer-perf-test.sh \    --bootstrap-server broker1:9092,broker2:9092,broker3:9092 \    --topic test-topic \    --messages 10000000 \    --threads 1 \    --print-metrics    
# 多线程消费者测试  /opt/kafka/bin/kafka-consumer-perf-test.sh \    --bootstrap-server broker1:9092,broker2:9092,broker3:9092 \    --topic test-topic \    --messages 10000000 \    --threads 8 \    --print-metrics
```

03

端到端延迟测试

测量从生产到消费的端到端延迟：

```
# 创建一个具有多个分区的测试主题  /opt/kafka/bin/kafka-topics.sh \    --bootstrap-server broker1:9092 \    --create \    --topic latency-test \    --partitions 8 \    --replication-factor 3    
# 使用自定义Java程序测量端到端延迟// 生产者代码示例  Properties props = new Properties();  props.put("bootstrap.servers", "broker1:9092,broker2:9092,broker3:9092");  props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");  props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");  props.put("acks", "all");    
Producer<String, String> producer = new KafkaProducer<>(props);  for (int i = 0; i < 10000; i++) {      long timestamp = System.currentTimeMillis();      ProducerRecord<String, String> record =           new ProducerRecord<>("latency-test", null, timestamp, "key-" + i, "value-" + timestamp);      producer.send(record);      Thread.sleep(100); // 每秒发送10条消息  }  producer.close();    
// 消费者代码示例  Properties props = new Properties();  props.put("bootstrap.servers", "broker1:9092,broker2:9092,broker3:9092");  props.put("group.id", "latency-test-group");  props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");  props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");  props.put("auto.offset.reset", "earliest");    
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);  consumer.subscribe(Collections.singletonList("latency-test"));    
while (true) {      ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));      for (ConsumerRecord<String, String> record : records) {          long latency = System.currentTimeMillis() - record.timestamp();          System.out.printf("Offset = %d, Latency = %d ms%n", record.offset(), latency);      }  }
```

04

吞吐量与延迟权衡测试

测试不同配置下吞吐量与延迟的权衡关系：

```
# 高吞吐量配置测试  /opt/kafka/bin/kafka-producer-perf-test.sh \    --topic throughput-test \    --num-records 5000000 \    --record-size 1024 \    --throughput -1 \    --producer-props bootstrap.servers=broker1:9092,broker2:9092,broker3:9092 \    acks=1 \    batch.size=131072 \    linger.ms=50 \    compression.type=lz4 \    buffer.memory=67108864    
# 低延迟配置测试  /opt/kafka/bin/kafka-producer-perf-test.sh \    --topic latency-test \    --num-records 1000000 \    --record-size 1024 \    --throughput -1 \    --producer-props bootstrap.servers=broker1:9092,broker2:9092,broker3:9092 \    acks=1 \    batch.size=8192 \    linger.ms=0 \    compression.type=none
```

03

**压测指标分析**

01

生产者关键指标

1. 吞吐量（Throughput）：每秒处理的消息数或字节数
2. 延迟（Latency）：消息从发送到确认的时间
3. CPU使用率：生产者进程的CPU使用情况
4. 内存使用率：生产者进程的内存使用情况
5. 批处理率：每批次的平均消息数

02

消费者关键指标

1. 吞吐量：每秒消费的消息数或字节数
2. 延迟：消息从生产到消费的时间
3. 消费者滞后（Consumer Lag）：消费者落后于生产者的消息数
4. 处理时间：消费者处理每条消息的时间
5. 提交率：偏移量提交的频率和成功率

03

Broker关键指标

1. 请求处理率：每秒处理的请求数
2. 请求队列大小：等待处理的请求数
3. 网络吞吐量：进出Broker的网络流量
4. 磁盘使用率：日志文件的增长速率
5. GC暂停时间：垃圾收集对性能的影响

04

**压测结果解读**

01

生产者性能分析

以下是一个典型的生产者性能测试结果示例：

```
100000 records sent, 25000.0 records/sec (24.41 MB/sec), 15.2 ms avg latency, 293.0 ms max latency.  200000 records sent, 26315.8 records/sec (25.67 MB/sec), 12.8 ms avg latency, 128.0 ms max latency.  300000 records sent, 27272.7 records/sec (26.61 MB/sec), 11.5 ms avg latency, 98.0 ms max latency.  
```

结果解读：

* 吞吐量随时间稳定在约26,000条记录/秒（约25MB/秒）
* 平均延迟约为13毫秒，最大延迟为293毫秒
* 随着测试进行，延迟趋于稳定，表明系统性能良好

02

消费者性能分析

以下是一个典型的消费者性能测试结果示例：

```
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec, rebalance.time.ms, fetch.time.ms, fetch.MB.sec, fetch.nMsg.sec  2023-05-01 10:00:00, 2023-05-01 10:01:00, 1024.00, 17.07, 1048576, 17476.27, 20, 60000, 17.07, 17476.27  
```

结果解读：

* 消费速率为17.07MB/秒，约17,476条消息/秒
* 重平衡时间为20毫秒，表明消费者组协调效率高
* 获取时间为60秒，与测试持续时间一致

03

瓶颈识别与解决

常见的性能瓶颈及解决方案：

CPU瓶颈：

* 增加broker数量
* 优化消息压缩算法
* 调整JVM参数

内存瓶颈：

* 增加堆内存大小
* 优化生产者/消费者客户端缓冲区大小
* 减少不必要的对象创建

磁盘I/O瓶颈：

* 使用更快的存储（如SSD）
* 增加数据目录数量，分散I/O负载
* 优化日志段大小和刷盘策略

网络瓶颈：

* 增加网络带宽
* 优化消息批处理大小
* 使用更高效的压缩算法

写在最后

**以上答案收入了《Kafka部署\_压测\_运维方案之48页》**

扫描下方二维码，即可全部面试题答案

获取更多信息，关注大数据技能圈

**欢迎添加作者交流**

## 推荐阅读系列文章

* [字节面试：Flink如何做压测？如何保证系统稳定？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491131&idx=1&sn=6bac85b9e17620b49b37ebd3224b3a83&scene=21#wechat_redirect)
* [ElasticSearch经典面试题200道（附500页7万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491122&idx=1&sn=cb7f3563a8082d248b1a0b768bd4d567&scene=21#wechat_redirect)
* [FlinkCDC经典面试题200道（附600页8万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491106&idx=1&sn=d412a81e6d5742c3e4acf6a680761139&scene=21#wechat_redirect)
* [StarRocks经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect)（附550页12万字答案）
* [Flink源码分析 经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[附1200页32万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)
* [FlinkSQL 经典面试题200道（附1200页32万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21#wechat_redirect)
* Paimon经典面试题200道题（附500页16万字答案）
* Doris经典面试题200道（附1050页39万字答案）
* Flink经典面试题200道（附1060页26万字答案）
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

如果喜欢 请点个在看分享给身边的朋友