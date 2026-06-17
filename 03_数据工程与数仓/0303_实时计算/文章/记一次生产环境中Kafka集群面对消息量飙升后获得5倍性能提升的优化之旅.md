---
title: 记一次生产环境中Kafka集群面对消息量飙升后获得5倍性能提升的优化之旅
author: 大数据技能圈
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491189&idx=1&sn=e61a090bce54713fdfbfe891e02331bf&chksm=c173dc9dd2e2bf11846a15917a616a05f5d2e4cbc1d00291368c57cfca9110cfb236a16fbbdd&mpshare=1&scene=24&srcid=0513cIcFrq9UtyZ2EEtZizq2&sharer_shareinfo=c4bb9b5d9f6ac5e2851621079ab7b2e8&sharer_shareinfo_first=c4bb9b5d9f6ac5e2851621079ab7b2e8#rd
---

说在前面

# 🌟《苦获陪你学大数据》知识星球简介

# 

如何系统学习大数据？如何高效备战面试？也许知识星球《苦获陪你学大数据》可以帮你。

## 🔥 热门面试题合集

## 

覆盖Flink、Spark、Hive、Kafka、Doris、StarRocks等主流大数据组件，题型丰富，答案详实，助你轻松应对各类面试场景。

## 📖 核心技术文档

## 

持续整理和更新大数据领域的核心技术资料，涵盖数据仓库、实时计算、数据湖等热门方向，理论与实战并重。

## 🛠️ 组件部署运维文档

## 

提供Flink、Spark、Kafka、Paimon、Iceberg、Hudi、Seatunnel等组件的详细部署、压测与运维方案，手把手教你搭建生产级环境。

## 🚀 性能优化文档

## 

专设性能调优板块，内容涵盖内存优化、JVM调优、数据结构优化、集群资源配置等，助你解决实际工作中的性能瓶颈。

## 👥 专属群聊答疑

## 

加入专属星球群聊，遇到问题随时提问，星主和大佬们在线答疑，学习路上不再孤单。

## ⏰ 持续高频更新

## 

每周持续更新3-4篇十万字左右的高质量文档，内容紧跟行业前沿，保证你学到的都是最新、最实用的干货！

🌈 欢迎加入《苦获陪你学大数据》知识星球，和一群志同道合的小伙伴一起进步，收获成长！

前几天我碰到了一个Kafka集群在消息量突然飙升时遇到的性能瓶颈问题，经过几个小时的苦战后，集群恢复了平稳运行，性能提升了5倍，本文将详细阐述从问题发现、分析到最终解决的完整过程。

本文详细内容已收入咱们的 《生产案例之线上Kafka集群面对消息风暴的5倍性能提升之旅PDF》，供后面的小伙伴参考，提升大家的大数据组件优化及架构能力。

01

**第一部分：问题发现**

1.1 告警触发

周一早上9:00，监控系统突然触发了多项Kafka集群告警：

* 消息积压量急剧上升，部分主题的消费延迟从毫秒级增加到分钟级
* 生产者客户端报告大量请求超时错误
* Broker的CPU使用率飙升至90%以上
* 网络流量激增，接近物理网卡限制
* 磁盘I/O使用率达到峰值

### 1.2 初步情况评估

运维团队立即展开初步调查，发现：

* 集群配置：5个broker节点，每个节点16核CPU，64GB内存，10Gbps网络，RAID10磁盘阵列
* 主要业务主题的分区数：30个，副本因子为3
* 正常情况下，集群处理能力约为每秒50,000条消息
* 当前情况：消息生产速率突然飙升至每秒200,000条，是平时的4倍
* 消费者组处理速度明显滞后，无法跟上生产速度

### 1.3 业务影响评估

* 多个关键业务系统报告数据处理延迟
* 用户交易确认时间延长，部分交易出现超时
* 数据分析平台无法及时获取最新数据
* 实时监控系统数据更新滞后

02

**第二部分：问题分析**

2.1 日志分析

首先检查Kafka服务器日志，发现大量警告和错误信息：

```
WARN [ReplicaManager brokerId=1] Produce request with correlation id 12345 from client client1 on partition topic1-15 failed due to request timeout  ERROR [KafkaRequestHandlerPool-0] Error when handling request: clientId=client2, correlationId=23456, api=PRODUCE  java.lang.OutOfMemoryError: Java heap space  WARN [ReplicaFetcherThread-0-3] Error in fetch from broker 3 for partition topic2-5  
```

这些日志表明集群正面临严重的资源压力，包括请求处理超时、内存不足和副本同步问题。

### 2.2 JVM监控分析

通过JVM监控工具分析broker进程：

* 堆内存使用率接近90%，频繁触发Full GC
* GC暂停时间从正常的几十毫秒增加到几百毫秒
* 年轻代内存分配速率异常高，表明有大量对象创建

### 2.3 系统资源分析

使用系统监控工具分析各节点资源使用情况：

* CPU：用户空间使用率85%，系统空间15%，几乎没有空闲
* 内存：物理内存使用率95%，部分节点开始使用swap
* 磁盘I/O：写入速度接近物理极限，读取队列长度持续增加
* 网络：入站流量8.5Gbps，接近10Gbps的物理限制

### 2.4 Kafka指标分析

通过Kafka内置指标和JMX监控，发现以下关键问题：

1. **请求队列积压**

   ：

* 请求队列大小(`request-queue-size`)持续增长，表明网络线程无法及时处理请求
* 网络处理线程CPU使用率接近100%

2. **磁盘I/O瓶颈**

   ：

* 日志刷盘延迟(`log-flush-rate-and-time-ms`)显著增加
* 磁盘写入操作频繁阻塞

3. **副本同步问题**

   ：

* ISR收缩事件频繁发生，表明副本无法跟上leader的写入速度
* 副本同步延迟(`replica-lag`)持续增加

4. **消息批处理效率低下**

   ：

* 平均批次大小(`batch-size-avg`)远低于配置的最大值
* 生产者请求频率过高，导致网络和处理开销增加

### 2.5 客户端配置分析

检查生产者客户端配置，发现以下问题：

* 批处理大小(`batch.size`)设置过小，默认为16KB
* linger.ms设置为0，导致消息立即发送而不等待批处理
* 压缩类型设置为`none`，未启用任何压缩
* 生产者缓冲区(`buffer.memory`)设置不足，仅为32MB

检查消费者客户端配置：

* 消费者拉取大小(`fetch.max.bytes`)设置过小，限制了单次拉取的数据量
* 消费者线程数不足，无法充分利用多核处理能力
* 消费者组重平衡策略不合理，导致频繁的分区重分配

### 2.6 根因分析

综合以上分析，确定了以下核心问题：

1. **1. 资源配置不足：**

* Broker的JVM堆内存配置不足以应对突发流量
* 网络线程和I/O线程数量不足，无法充分利用多核CPU

2. **2. 参数配置不合理：**

* 生产者批处理和压缩配置不合理，导致网络和处理效率低下
* 消费者拉取配置限制了消费速度
* Broker端队列大小限制不足以应对突发流量

3. **3. 主题分区设计不合理：**

* 热点主题分区数量不足，无法充分利用并行处理能力
* 分区分布不均衡，导致部分broker负载过重

4. **4. 监控预警不及时：**

* 缺乏对生产速率变化的有效监控和预警机制
* 没有针对突发流量的自动扩容策略

03

**第三部分：紧急优化方案**

基于上述分析，制定了分阶段的紧急优化方案：

### 3.1 第一阶段：立即缓解压力（0-2小时）

#### 3.1.1 增加关键资源配置

1. **增加JVM堆内存**

```
# 修改Kafka启动脚本中的KAFKA_HEAP_OPTS  export KAFKA_HEAP_OPTS="-Xms16G -Xmx16G"
```

1. **优化GC配置**

```
# 添加以下GC参数  export KAFKA_JVM_PERFORMANCE_OPTS="-XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent"
```

1. **增加网络和I/O线程数**
2. 修改`server.properties`配置：

```
# 增加网络线程数，从默认的3增加到16  num.network.threads=16    
# 增加I/O线程数，从默认的8增加到32  num.io.threads=32    
# 增加请求队列大小  queued.max.requests=1000
```

1. **调整请求处理参数：**

```
# 增加socket接收缓冲区  socket.receive.buffer.bytes=1048576    
# 增加socket发送缓冲区  socket.send.buffer.bytes=1048576    
# 增加最大请求大小  max.request.size=10485760
```

#### 3.1.2 优化生产者客户端配置

为主要生产者应用紧急推送配置更新：

```
# 增加批处理大小  batch.size=131072    
# 设置linger.ms使消息有时间进行批处理  linger.ms=50    
# 启用压缩  compression.type=lz4    
# 增加生产者缓冲区  buffer.memory=536870912    
# 增加重试次数和重试间隔  retries=10  retry.backoff.ms=100
```

#### 3.1.3 优化消费者客户端配置

为主要消费者应用紧急推送配置更新：

```
# 增加单次拉取大小  fetch.max.bytes=52428800    
# 增加最大拉取等待时间  fetch.max.wait.ms=500    
# 增加消费者缓冲区大小  max.partition.fetch.bytes=10485760
```

#### 3.1.4 临时限流措施

为非关键业务实施临时限流：

```
// 在生产者客户端实施限流  Properties props = new Properties();  props.put("throttle.rate", "10000"); // 每秒限制10000条消息
```

### 3.2 第二阶段：系统优化（2-6小时）

#### 3.2.1 增加热点主题分区数

为热点主题增加分区数，提高并行处理能力：

```
# 使用kafka-topics.sh工具增加分区数  bin/kafka-topics.sh --bootstrap-server broker1:9092 --alter --topic hot-topic --partitions 60
```

#### 3.2.2 重平衡分区分配

执行分区重分配，优化分区在broker间的分布：

```
# 生成分区重分配计划  bin/kafka-reassign-partitions.sh --bootstrap-server broker1:9092 --generate --topics-to-move-json-file topics.json --broker-list "1,2,3,4,5"    
# 执行分区重分配  bin/kafka-reassign-partitions.sh --bootstrap-server broker1:9092 --execute --reassignment-json-file reassignment.json
```

#### 3.2.3 优化日志配置

修改`server.properties`配置，优化日志处理：

```
# 增加日志段大小，减少小文件数量  log.segment.bytes=1073741824    
# 优化日志刷盘策略，避免频繁刷盘  log.flush.interval.messages=50000  log.flush.interval.ms=10000    
# 优化日志清理线程数  log.cleaner.threads=4
```

#### 3.2.4 调整副本同步参数

```
# 增加副本拉取线程数  num.replica.fetchers=8    
# 增加副本拉取大小  replica.fetch.max.bytes=10485760    
# 调整ISR收缩时间，避免频繁的ISR变化  replica.lag.time.max.ms=30000
```

#### 3.2.5 实施动态配置调整

利用Kafka的动态配置功能，无需重启即可调整关键参数：

```
# 使用kafka-configs.sh工具动态调整配置  bin/kafka-configs.sh --bootstrap-server broker1:9092 --entity-type brokers --entity-name 1 --alter --add-config "num.io.threads=32,num.network.threads=16"
```

### 3.3 第三阶段：架构优化（6-24小时）

#### 3.3.1 扩展集群规模

紧急添加新的broker节点，扩展集群处理能力：

```
# 配置新的broker节点  # server.properties for new brokers  broker.id=6  listeners=PLAINTEXT://new-broker:9092  ...    
# 启动新节点  bin/kafka-server-start.sh config/server.properties
```

#### 3.3.2 实施主题分区迁移

将部分热点主题的分区迁移到新节点：

```
# 生成迁移计划，将部分分区迁移到新节点  bin/kafka-reassign-partitions.sh --bootstrap-server broker1:9092 --generate --topics-to-move-json-file hot-topics.json --broker-list "1,2,3,4,5,6"    
# 执行迁移计划  bin/kafka-reassign-partitions.sh --bootstrap-server broker1:9092 --execute --reassignment-json-file migration.json
```

#### 3.3.3 优化存储架构

1. 将日志目录分散到多个物理磁盘：

```
# 配置多个日志目录，分散I/O压力  log.dirs=/data1/kafka-logs,/data2/kafka-logs,/data3/kafka-logs,/data4/kafka-logs
```

为不同类型的主题配置不同的存储策略：

```
# 为高吞吐量主题配置专用存储策略  bin/kafka-configs.sh --bootstrap-server broker1:9092 --entity-type topics --entity-name high-throughput-topic --alter --add-config "retention.ms=86400000,cleanup.policy=delete"    
# 为需要长期保存的主题配置压缩策略  bin/kafka-configs.sh --bootstrap-server broker1:9092 --entity-type topics --entity-name archive-topic --alter --add-config "cleanup.policy=compact"
```

#### 3.3.4 实施客户端架构优化

1. 1. 改进生产者设计

```
// 实现异步批量发送模式  public class OptimizedProducer {      private final KafkaProducer<String, String> producer;      private final ScheduledExecutorService scheduler;      private final ConcurrentLinkedQueue<ProducerRecord<String, String>> messageQueue;      private final int batchSize;    
    public OptimizedProducer(Properties props, int batchSize) {          this.producer = new KafkaProducer<>(props);          this.scheduler = Executors.newScheduledThreadPool(1);          this.messageQueue = new ConcurrentLinkedQueue<>();          this.batchSize = batchSize;    
        // 定时批量发送          this.scheduler.scheduleAtFixedRate(this::sendBatch, 100, 100, TimeUnit.MILLISECONDS);      }    
    public void send(String topic, String key, String value) {          messageQueue.add(new ProducerRecord<>(topic, key, value));          if (messageQueue.size() >= batchSize) {              sendBatch();          }      }    
    private void sendBatch() {          List<ProducerRecord<String, String>> batch = new ArrayList<>();          ProducerRecord<String, String> record;          while ((record = messageQueue.poll()) != null && batch.size() < batchSize) {              batch.add(record);          }    
        for (ProducerRecord<String, String> r : batch) {              producer.send(r, (metadata, exception) -> {                  if (exception != null) {                      // 处理失败，重新入队或记录日志                      messageQueue.add(r);                  }              });          }          producer.flush();      }  }
```

1. 2. 优化消费者处理模型

```
// 实现多线程消费处理模型  public class ParallelConsumer {      private final Consumer<String, String> consumer;      private final ExecutorService executor;      private final int numWorkers;    
    public ParallelConsumer(Properties props, int numWorkers) {          this.consumer = new KafkaConsumer<>(props);          this.numWorkers = numWorkers;          this.executor = Executors.newFixedThreadPool(numWorkers);      }    
    public void start(String topic, MessageProcessor processor) {          consumer.subscribe(Collections.singleton(topic));    
        while (true) {              ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));              if (!records.isEmpty()) {                  // 将记录分组，并行处理                  List<Future<?>> futures = new ArrayList<>();                  for (TopicPartition partition : records.partitions()) {                      List<ConsumerRecord<String, String>> partitionRecords = records.records(partition);                      futures.add(executor.submit(() -> {                          for (ConsumerRecord<String, String> record : partitionRecords) {                              processor.process(record);                          }                      }));                  }    
                // 等待所有处理完成                  for (Future<?> future : futures) {                      try {                          future.get();                      } catch (Exception e) {                          // 处理异常                      }                  }    
                // 手动提交偏移量                  consumer.commitSync();              }          }      }    
    public interface MessageProcessor {          void process(ConsumerRecord<String, String> record);      }  }
```

#### 3.3.5 实施主题分区策略优化

1. 根据消息流量特征重新设计分区策略：

```
# 为高流量主题增加分区数  bin/kafka-topics.sh --bootstrap-server broker1:9092 --alter --topic high-traffic-topic --partitions 100    
# 为关键业务主题增加副本因子  bin/kafka-topics.sh --bootstrap-server broker1:9092 --alter --topic critical-topic --replica-assignment "1:2:3:4:5,2:3:4:5:1,..."
```

1. 实施自定义分区器，避免热点分区：

```
public class BalancedPartitioner implements Partitioner {      private Random random = new Random();    
    @Override      public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {          List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);          int numPartitions = partitions.size();    
        if (keyBytes == null) {              // 随机分区              return random.nextInt(numPartitions);          } else {              // 使用一致性哈希算法，但避免热点              int hashCode = Arrays.hashCode(keyBytes);              return Math.abs(hashCode) % numPartitions;          }      }    
    @Override      public void close() {}    
    @Override      public void configure(Map<String, ?> configs) {}  }
```

以上。

文章完整内容已收录知识星球《苦获陪你学大数据》“生产案例之Kafka集群面对消息风暴的5倍性能提升之旅pdf”，目录如下：

写在最后

**以上答案收入了《生产案例之Kafka集群面对消息风暴的5倍性能提升之旅pdf》**

扫描下方二维码，即可获取

全部文档及专业群聊答疑

获取更多信息，关注大数据技能圈

**欢迎添加作者交流**

## 推荐阅读系列文章

* [腾讯面试：Spark内存如何优化？包含哪几个方面？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491167&idx=1&sn=d62e305f6803f484aa40469393b1fdb8&scene=21#wechat_redirect)
* [蚂蚁面试：Kafka如何做压测？如何保证系统稳定？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491146&idx=1&sn=e2c4a1e298aec2b6d4d42cffb2f331ef&scene=21#wechat_redirect)
* [字节面试：Flink如何做压测？如何保证系统稳定？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491131&idx=1&sn=6bac85b9e17620b49b37ebd3224b3a83&scene=21#wechat_redirect)
* [Flink内存调优指南（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=2&sn=cccaa1289c4581a2dd407d21a8c5ae09&scene=21#wechat_redirect)附500页16万字答案[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=2&sn=cccaa1289c4581a2dd407d21a8c5ae09&scene=21#wechat_redirect)
* [Hive经典面试题200道（附8万字420页答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491138&idx=1&sn=f1acb3164d5f0f7e8be7f2037f4a838b&scene=21#wechat_redirect)
* [Kafka 经典面试题200道（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)[附8万字420页答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491138&idx=1&sn=f1acb3164d5f0f7e8be7f2037f4a838b&scene=21&token=1460012704&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)
* [Spark经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)[附500页8万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491106&idx=1&sn=d412a81e6d5742c3e4acf6a680761139&scene=21&token=1460012704&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)
* [ElasticSearch经典面试题200道（附400页12万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491122&idx=1&sn=cb7f3563a8082d248b1a0b768bd4d567&scene=21#wechat_redirect)
* [FlinkCDC经典面试题200道（附500页8万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491106&idx=1&sn=d412a81e6d5742c3e4acf6a680761139&scene=21#wechat_redirect)
* [StarRocks](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect) [经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect)[附550页12万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect)
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