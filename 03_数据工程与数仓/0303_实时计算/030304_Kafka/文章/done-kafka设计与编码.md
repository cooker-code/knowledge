> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030304_Kafka/030304_核心知识点/Kafka副本一致性与ISR边界|Kafka副本一致性与ISR边界]]
---
title: kafka设计与编码
author: 程序猿架构之路
date: evanevan
url: https://mp.weixin.qq.com/s?__biz=MzIwNjkwNDc1OQ==&mid=2247483816&idx=1&sn=a02e25e774431254cd3fe309417ebe61&chksm=96d7d9f63061f89ab73e0d9a6435f7a7ec27f313742c2ce0df71cf84cb32a367aa445507150e&mpshare=1&scene=24&srcid=05104hdbILUocOiA8txuNodn&sharer_shareinfo=1518ba808501afb97fddbd82356d00be&sharer_shareinfo_first=1518ba808501afb97fddbd82356d00be#rd
---

# kafka系统设计与编码

🏠 README · ⬅ 02 调优与运维 · ➡ 04 源码与OS底层

* **包含章节**

  ：Ch13 业务场景设计 / Ch14 手撕题 / Ch15 加分项 & 反问
* **难度**

  ：⭐⭐ 资深岗分水岭
* **建议**

  ：**第 3 周**完成

+ Ch13 准备 4~5 个白板可画的业务场景
+ Ch14 手撕题**必须动手敲熟**，肌肉记忆
+ Ch15 加分项 + 反问，面试前 1 天回顾

---

## 📑 本模块目录

* 13. 业务场景设计题 ⭐⭐
* 14. 手撕题 / 编码题 ⭐⭐
* 15. 面试加分项 & 反问环节 ⭐

---

### 模块导引

**目标**：从"会用 Kafka"上升到"能用 Kafka 解决业务难题"。资深岗 30~60 分钟系统设计题就考这一模块。

**检验标准**：

* 能在白板上画出"秒杀 / CDC / 延迟队列 / DLQ"四种典型场景的 Kafka 用法
* 现场写出可靠的 Producer/Consumer 包装类（含幂等、优雅停机、Rebalance Listener）
* 能针对"基于 Kafka 实现延迟队列"提出至少 2 种方案并讲清取舍

> 共 3 章（Ch13~Ch15）。重点是 **Ch14 手撕题**——很多公司会让你现场敲，必须熟练到肌肉记忆。

---

## 13. 业务场景设计题

> **本章定位**：资深岗、架构师面试的「白板题」核心。每题按 **场景 → 架构图 → 关键决策 → 陷阱 → 完整代码** 结构展开。
>
> **答题套路**：先讲清需求（QPS / 一致性 / 延迟 SLA） → 画架构图 → 解释每个组件的选型理由 → 主动暴露 2~3 个陷阱并给出方案。

### 13.1 设计一个秒杀系统中的 Kafka 用法

#### 13.1.1 业务需求

* **峰值 QPS**

  ：100 万 / 秒（如双 11 / 演唱会抢票）
* **库存数量**

  ：1 万件
* **订单不可超卖**

  + **不可重复下单** + **同一用户多次点击不重复扣款**
* **响应时间**

  ：< 200ms

#### 13.1.2 端到端架构

#### 13.1.3 关键决策点

**层层削减流量**：

```
1M QPS（用户）  ↓ CDN + WAF：拦截爬虫和明显作弊（-50%）500K QPS（合法用户）  ↓ 网关令牌桶限流 + 验证码（-94%）30K QPS（前置校验通过）  ↓ Redis 预扣库存（仅 1 万件 → 99% 拒绝）10K QPS（实际成功）  ↓ Kafka 异步削峰100~500 QPS（数据库写入）
```

**Kafka 配置**：

```
# Produceracks=all                                # 不丢enable.idempotence=true                 # 不重linger.ms=10                            # 攒批batch.size=131072                       # 128KBcompression.type=lz4max.in.flight.requests.per.connection=5# Topicseckill-topicpartitions=64                            # 64 个 partition 支撑高并发replication.factor=3min.insync.replicas=2# Producer 路由：key=skuId# → 同一 SKU 顺序进同一 Partition# → 避免库存争抢，便于业务侧顺序消费
```

**消费者幂等**：

```
CREATE TABLE seckill_order (    id BIGINT PRIMARY KEY,    user_id BIGINT NOT NULL,    sku_id BIGINT NOT NULL,    -- 业务幂等键：用户 + 活动 + 商品    UNIQUE KEY uniq_user_activity_sku (user_id, activity_id, sku_id),    created_at DATETIME) ENGINE=InnoDB;
```

```
try {    orderMapper.insert(order);} catch (DuplicateKeyException e) {    // 幂等命中，不算失败    log.info("Order already exists: {}", order.getId());}
```

#### 13.1.4 陷阱与方案

**陷阱 1：Redis 预扣后服务崩溃 → 库存"假性占用"**

→ 方案：Redis 设置 5 分钟 TTL；MySQL 超时未支付订单定时回补 Redis 库存。

**陷阱 2：Kafka 写入失败 → Redis 已扣库存丢失**

→ 方案：Producer 同步等 ack；失败则 Lua 脚本回滚 Redis 库存（**两阶段补偿**）。

**陷阱 3：Hot Partition（爆款 SKU）**

→ 方案：单个爆款 SKU 单独一个 Topic + 加大 Partition 数；或用 `skuId + (userId % 10)` 散列到 10 个 Partition，业务侧汇总。

---

### 13.2 基于 Kafka 实现延迟队列

#### 13.2.1 业务需求

* 订单 30 分钟未支付自动取消
* 优惠券到期前 1 小时提醒
* **精度要求**

  ：秒级
* **支持任意延迟时长**

  ：从 1 秒到 30 天

#### 13.2.2 三种方案对比

| 方案 | 实现 | 精度 | 复杂度 | 适用 |
| --- | --- | --- | --- | --- |
| **A. 多级 Topic 分桶** | 5s / 1min / 5min / 30min / 1h / 1d 多个 Topic | 桶级（最坏到 1h） | 低 | 业务可容忍粗粒度 |
| **B. 单 Topic + 时间轮服务** | 自研 KafkaDelayService 中转 | 秒级 | 中 | 通用场景 |
| **C. 换 RocketMQ 5.0** | 内置任意精度延迟 | 秒级 | 低 | 允许换技术栈 |

#### 13.2.3 方案 B 详细设计（推荐）

**消息格式**：

```
public class DelayMessage {    String targetTopic;       // 到期后投递的目标 Topic    long deliverAtMs;         // 到期时间戳（绝对时间）    String key;               // 业务 key（路由用）    byte[] payload;           // 业务消息体    Map<String, String> headers;  // 透传 header}
```

**时间轮服务核心逻辑**：

```
public class KafkaDelayService {    private final HashedWheelTimer timer = new HashedWheelTimer(1, TimeUnit.SECONDS, 512);    private final KafkaProducer<String, byte[]> producer;    private final KafkaConsumer<String, DelayMessage> consumer;    public void run() {        consumer.subscribe(List.of("delay-topic"));        while (running) {            ConsumerRecords<String, DelayMessage> records = consumer.poll(Duration.ofMillis(500));            for (ConsumerRecord<String, DelayMessage> r : records) {                DelayMessage msg = r.value();                long delay = msg.deliverAtMs - System.currentTimeMillis();                if (delay <= 0) {                    // 已过期，立即投递                    forward(msg);                } else if (delay > MAX_DELAY) {                    // 超长延迟分级处理（避免内存堆积）                    putToColdStorage(msg);                } else {                    timer.newTimeout(t -> forward(msg), delay, TimeUnit.MILLISECONDS);                }            }            consumer.commitSync();        }    }    private void forward(DelayMessage msg) {        producer.send(new ProducerRecord<>(msg.targetTopic, msg.key, msg.payload));    }}
```

#### 13.2.4 关键陷阱

**陷阱 1：服务重启 → 时间轮内存丢失**

→ 方案：消息**先 commit 到 delay-topic**（持久化），重启后从 earliest 重新消费 + 重建时间轮。

**陷阱 2：超长延迟（30 天）占内存**

→ 方案：**分级时间轮** + 冷热分离：

* < 1h：内存时间轮
* 1h ~ 24h：RocksDB 落盘
* > 24h：写到独立的 long-delay-topic + 定时调度器

**陷阱 3：服务多实例水平扩展，消息分配不均**

→ 方案：delay-topic 多分区 + 消费者按 partition 分配。每个实例只处理自己负责的 partition。

**陷阱 4：高峰期投递放大流量**

→ 方案：消费 delay-topic 时主动限流，避免到期消息瞬间打爆下游。

---

### 13.3 数据库 CDC 接 Kafka 实时数仓（高频）

#### 13.3.1 业务需求

* MySQL 业务库变更**实时同步到数据湖 / OLAP**
* 端到端延迟 < 30 秒
* **保证 update / delete 顺序**
* 支持 Schema 演进
* 不丢不重

#### 13.3.2 完整架构

#### 13.3.3 核心配置

**Topic 设计**：

```
按表分 Topic（推荐）：  cdc.order_db.orders          ← 主键 key 路由  cdc.order_db.users  cdc.order_db.order_items按库分 Topic（备选）：  cdc.order_db                 ← table 在 header 里  → 优势：Topic 少；劣势：Schema 多样难处理
```

**Producer 配置**（Debezium）：

```
# 同主键路由同分区 → 保证 UPDATE/DELETE 顺序key.converter=io.confluent.connect.avro.AvroConverterkey.converter.schema.registry.url=http://schema-registry:8081# 用 Compact Topic 保留每行最新状态（变成流表）log.cleanup.policy=compact# Tombstone 处理（删除标记）delete.handling.mode=rewrite# 大事务保护max.batch.size=2048producer.delivery.timeout.ms=300000
```

#### 13.3.4 端到端 EOS 实现

#### 13.3.5 陷阱与方案

**陷阱 1：Schema 演进破坏下游**

→ Schema Registry + 兼容性策略（推荐 BACKWARD）：

* **BACKWARD**

  ：新 schema 能读老消息（**只允许加可选字段 / 删字段**）。
* **FORWARD**

  ：老 schema 能读新消息（只允许加字段 / 删可选字段）。
* **FULL**

  ：双向兼容。

**陷阱 2：DDL 变更（加列、改类型）后下游 Flink 报错**

→ 方案：DDL 变更前先升级 Sink Schema → 数据库变更 → Source 自动感知。或者用 **Schema-On-Read** 的格式（Iceberg / Delta Lake）。

**陷阱 3：MySQL 大事务（10 万行）→ Kafka 单分区瞬时压力**

→ 方案：Debezium 配置 `max.batch.size` 拆批 + 分区数充足。

**陷阱 4：DELETE 后又 INSERT 同主键 → 顺序错乱**

→ 用主键 hash 路由保证同 partition；Compact Topic 自动处理。

---

### 13.4 死信与重试机制完整设计

#### 13.4.1 业务需求

* 消费失败时**有限次重试**，避免永远卡住
* 不同失败类型采用不同重试策略
* 反复失败的消息进入死信队列，人工介入
* 死信可视化 + 重投能力

#### 13.4.2 重试 Topic 链架构

#### 13.4.3 实现关键

**消息 Header 标记**：

```
Header keys:  retry.count       = 3                              # 当前重试次数  retry.original-topic = order-event                  # 原始 Topic  retry.first-failed-at = 2026-05-07T10:00:00.000Z   # 首次失败时间  retry.last-error  = "DB connection timeout"        # 最近一次错误
```

**消费者处理逻辑**：

```
@KafkaListener(topics = "order-event", groupId = "order-svc")public void handleOrder(ConsumerRecord<String, Order> record, Acknowledgment ack) {    try {        orderService.process(record.value());        ack.acknowledge();    } catch (Exception e) {        Headers headers = record.headers();        int retryCount = getRetryCount(headers);        if (retryCount >= MAX_RETRIES) {            sendToDLQ(record, e);        } else {            sendToRetryTopic(record, retryCount + 1, e);        }        ack.acknowledge();   // 主 Topic 消费完毕    }}private void sendToRetryTopic(ConsumerRecord<String, Order> record, int newCount, Exception e) {    String retryTopic = pickRetryTopic(newCount);  // 30s / 5m / 30m    Headers newHeaders = record.headers().add("retry.count", String.valueOf(newCount).getBytes())                                          .add("retry.last-error", e.getMessage().getBytes());    producer.send(new ProducerRecord<>(retryTopic, null, record.timestamp(), record.key(), record.value(), newHeaders));}
```

**重试 Topic 消费者（带延迟）**：

```
@KafkaListener(topics = {"order-event.retry.30s", "order-event.retry.5min", "order-event.retry.30min"})public void handleRetry(ConsumerRecord<String, Order> record, Acknowledgment ack) {    // 延迟到 deliverAt 才处理    long deliverAt = record.timestamp() + getDelayMs(record.topic());    long sleepMs = deliverAt - System.currentTimeMillis();    if (sleepMs > 0) {        Thread.sleep(sleepMs);    }    // 重新触发主消费者逻辑    handleOrder(record, ack);}
```

#### 13.4.4 DLQ 运营平台

最低实现：

```
CREATE TABLE dead_letter (    id BIGSERIAL PRIMARY KEY,    topic VARCHAR(128),    partition INT,    offset BIGINT,    key VARCHAR(256),    payload TEXT,    headers JSONB,    last_error TEXT,    created_at TIMESTAMP,    handled_at TIMESTAMP NULL,    handled_by VARCHAR(64) NULL,    handle_action VARCHAR(32) NULL  -- REPLAY / DELETE / IGNORE);
```

DLQ 消费者把消息写入 DB → 运营后台 GUI 操作 → REPLAY 时重新投回主 Topic。

#### 13.4.5 陷阱

1. **重试导致顺序错乱**

   ：失败消息延迟后投递，新消息已处理 → 业务**必须幂等**或接受最终一致。
2. **重试 Topic 消费 sleep 占用 Consumer 线程**

   ：用 Spring Kafka 的 `RetryableTopic` 或自研异步定时器。
3. **DLQ 黑洞**

   ：无人值守的 DLQ 等于丢消息，**必须告警 + 自动监控 DLQ Lag**。

---

### 13.5 跨地域数据同步（异地多活前置）

#### 13.5.1 业务需求

* 业务部署在两个机房（杭州 + 上海）
* 任一机房故障，业务不中断
* 数据双向同步，最大可容忍 RPO < 1 分钟

#### 13.5.2 方案对比

| 方案 | 同步方向 | 延迟 | 冲突处理 | 适用 |
| --- | --- | --- | --- | --- |
| **MM2 单向** | 主 → 备 | 秒级 | 无（备只读） | 灾备 |
| **MM2 双向 + 防回环** | 双向 | 秒级 | 业务侧幂等 | 双活 |
| **Stretched Cluster** | 同集群跨机房 | RPO=0 | 无（强一致） | 同城双活 |
| **业务侧路由 + Kafka 单元化** | 用户分流 | 秒级 | 单元内闭环 | 蚂蚁体系 |

#### 13.5.3 MM2 双向同步详细配置

**MM2 配置要点**：

```
# MM2 connectorclusters=HZ,SHHZ.bootstrap.servers=kafka-hz:9092SH.bootstrap.servers=kafka-sh:9092# 双向同步HZ->SH.enabled=trueSH->HZ.enabled=true# 防回环：同步过来的 Topic 加前缀，本地 Consumer 同时订阅 order-event 和 SH.order-eventreplication.policy.class=org.apache.kafka.connect.mirror.DefaultReplicationPolicy# 同步 ACL / Quota / __consumer_offsetsemit.heartbeats.enabled=truesync.topic.acls.enabled=truesync.group.offsets.enabled=true# 限速HZ->SH.replication.factor=3HZ->SH.tasks.max=8HZ->SH.replication.policy.separator=.
```

#### 13.5.4 陷阱

1. **跨机房 RTT 高 → acks=all 性能下降**

   ：本地写本地集群，跨机房异步 MM2，**业务不要直连远端 Kafka**。
2. **回环检测**

   ：MM2 用 cluster alias 前缀（HZ.xxx）防回环；自研同步必须维护源 cluster ID 的 header。
3. **Consumer offset 不一致**

   ：MM2 维护映射表（每条消息源 offset → 目标 offset），切换时按映射表 reset。
4. **Schema Registry 也要双活**

   ：Confluent 的 Schema Linking 或自研同步。

---

### 13.6 消息广播（Pub/Sub Fanout）的多种实现

#### 13.6.1 场景

业务变更需要通知**多个不同业务系统**，比如订单创建后：

* 财务系统：记账
* 风控系统：实时分析
* 推送系统：消息通知用户
* 数据仓库：实时入仓

#### 13.6.2 三种实现对比

| 方案 | 实现 | 优点 | 缺点 |
| --- | --- | --- | --- |
| **A. 单 Topic + 多 Consumer Group** | 每个下游一个独立 group | 简单，Kafka 原生支持 | Topic 写流量与读流量不对称 |
| **B. 一个主 Topic + Fanout 服务转发** | 主 Topic + 业务 Topic | 业务下游解耦 | 多一次转发延迟 |
| **C. 主 Topic + Connect Sink** | Kafka Connect 推到 ES/HDFS/MySQL | 不需要业务消费者 | 灵活性差 |

#### 13.6.3 推荐方案：单 Topic + 多 Consumer Group

**关键点**：

* Topic 设计：业务**事件**型，不是命令型（`OrderCreated` 而不是 `SendNotification`）。
* 每个下游有独立 Group → offset 独立 → 互不影响。
* 部分下游有"过滤"需求（只关心 VIP 订单）→ 业务侧消费时过滤即可。

#### 13.6.4 容量估算

```
入流量：1 GB/s出流量：1 GB/s × 4 (4 个下游) = 4 GB/s+ 副本同步：1 GB/s × (R-1) = 2 GB/s总出流量：6 GB/s = 48 Gbps→ 必须 25Gbps × 2 网卡 bond→ 多个下游消费时网络是瓶颈，不是磁盘
```

---

### 13.7 高频追问

**Q：Kafka 能不能做请求-响应模式（RPC）？**

A：理论可以但**不推荐**。常见做法：

1. Producer 发请求消息，Header 带 correlationId + replyTopic。
2. Consumer 处理后，把响应发到 replyTopic（或 caller 订阅的临时 Topic）。
3. Producer 端订阅 replyTopic，按 correlationId 匹配响应。

缺点：

* 延迟高（一次往返 + Kafka 写盘）。
* 资源浪费（Topic / Consumer 持续运行）。
* 失败处理复杂（超时、重试、幂等）。

**正确选择**：

* 同步 RPC → gRPC / Dubbo / HSF。
* 异步通知 → Kafka 单向消息 + Webhook。

**Q：Kafka 适合做"任务队列"吗？**

A：**不适合**做精细任务调度。原因：

* 消费并发受 Partition 数限制，无法动态扩缩。
* 失败重试机制简陋，需要业务自己造轮子。
* 没有任务优先级。

**应该用**：

* Celery（Python）/ XXL-Job / ElasticJob → 任务调度。
* RocketMQ 顺序消息 + 重试机制 → 业务消息任务。

Kafka 适合的是 **“事件流”** 场景：日志、CDC、流处理上游。

**Q：你们公司 Kafka Topic 命名规范是怎样的？**

A（参考阿里 / 美团 / 字节通用规范）：

```
{environment}.{domain}.{event-type}.{version}例：prod.order.created.v1prod.payment.refunded.v1prod.user.profile-updated.v2test.cdc.mysql.order_db.orders
```

要点：

1. 环境前缀（prod/test/staging）— 物理集群往往不同。
2. 业务域分组（order / payment / user）— 便于权限管理。
3. 事件名（过去式）— 表示已发生事实。
4. 版本号（v1/v2）— Schema 演进时切新 Topic（避免 in-place 兼容性破坏）。

---

## 14. 手撕题 / 编码题

### 14.1 写一个可靠的 Producer 包装类

```
public class ReliableKafkaProducer<K, V> implements Closeable {    private final KafkaProducer<K, V> producer;    public ReliableKafkaProducer(Map<String, Object> userProps) {        Map<String, Object> props = new HashMap<>(userProps);        props.put(ProducerConfig.ACKS_CONFIG, "all");        props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);        props.put(ProducerConfig.RETRIES_CONFIG, Integer.MAX_VALUE);        props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 5);        props.put(ProducerConfig.DELIVERY_TIMEOUT_MS_CONFIG, 120_000);        props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "lz4");        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 128 * 1024);        props.put(ProducerConfig.LINGER_MS_CONFIG, 20);        this.producer = new KafkaProducer<>(props);    }    public CompletableFuture<RecordMetadata> sendAsync(ProducerRecord<K, V> record) {        CompletableFuture<RecordMetadata> future = new CompletableFuture<>();        producer.send(record, (md, ex) -> {            if (ex != null) future.completeExceptionally(ex);            else future.complete(md);        });        return future;    }    @Override public void close() { producer.flush(); producer.close(Duration.ofSeconds(30)); }}
```

### 14.2 写一个支持优雅停机 + 手动提交 + 业务线程池的 Consumer

```
public class ManagedConsumer implements Runnable, Closeable {    private final KafkaConsumer<String, String> consumer;    private final ExecutorService workerPool = Executors.newFixedThreadPool(16);    private final Map<TopicPartition, OffsetAndMetadata> pendingOffsets = new ConcurrentHashMap<>();    private volatile boolean running = true;    public ManagedConsumer(String groupId, List<String> topics) {        Properties p = new Properties();        p.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka:9092");        p.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);        p.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);        p.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 200);        p.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 600_000);        p.put(ConsumerConfig.PARTITION_ASSIGNMENT_STRATEGY_CONFIG,                CooperativeStickyAssignor.class.getName());        p.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);        p.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);        this.consumer = new KafkaConsumer<>(p);        consumer.subscribe(topics, new RebalanceListener());    }    @Override    public void run() {        try {            while (running) {                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(500));                if (records.isEmpty()) { commitIfNeeded(); continue; }                CountDownLatch latch = new CountDownLatch(records.count());                for (ConsumerRecord<String, String> r : records) {                    workerPool.submit(() -> {                        try { handle(r); }                        finally {                            pendingOffsets.merge(                                new TopicPartition(r.topic(), r.partition()),                                new OffsetAndMetadata(r.offset() + 1),                                (oldV, newV) -> newV.offset() > oldV.offset() ? newV : oldV);                            latch.countDown();                        }                    });                }                latch.await();                commitIfNeeded();            }        } catch (WakeupException ignored) {        } catch (Exception e) {            // log & alarm        } finally {            try { consumer.commitSync(pendingOffsets); } catch (Exception ignored) {}            consumer.close();        }    }    private void commitIfNeeded() {        if (pendingOffsets.isEmpty()) return;        Map<TopicPartition, OffsetAndMetadata> snapshot = new HashMap<>(pendingOffsets);        consumer.commitAsync(snapshot, (m, e) -> {            if (e == null) snapshot.forEach((k, v) -> pendingOffsets.remove(k, v));        });    }    private void handle(ConsumerRecord<String, String> r) { /* 业务处理（必须幂等） */ }    private class RebalanceListener implements ConsumerRebalanceListener {        @Override public void onPartitionsRevoked(Collection<TopicPartition> parts) {            try { consumer.commitSync(pendingOffsets); } catch (Exception ignored) {}            parts.forEach(pendingOffsets::remove);        }        @Override public void onPartitionsAssigned(Collection<TopicPartition> parts) { }    }    @Override public void close() {        running = false;        consumer.wakeup();        workerPool.shutdown();    }}
```

加分点：

* 引入 **per-partition 单线程**（按 partition 路由到固定 worker）保证局部顺序。
* onPartitionsRevoked 中 commit 已完成 offset 防止丢失/重复。

### 14.3 写一个本地内存的批量幂等组件

```
public class IdempotentExecutor {    private final Cache<String, Boolean> seen = Caffeine.newBuilder()            .expireAfterWrite(10, TimeUnit.MINUTES)            .maximumSize(1_000_000).build();    public boolean executeOnce(String bizId, Runnable task) {        return seen.get(bizId, k -> { task.run(); return true; });    }}
```

注意：本地内存只适合**短时窗内防重**，最终幂等仍要靠 DB 唯一索引。

---

## 15. 面试加分项 & 反问环节

> **本章定位**：给同样答得不错的候选人之间制造**差异化优势**。资深岗终面或交叉面，**会不会反问 / 会不会延伸是否有体系**，往往是定级的依据。

### 15.1 技术加分项 ── 拉开普通候选人的差距

#### 15.1.1 体系性表达（必备）

| 能力 | 具体表现 | 含金量 |
| --- | --- | --- |
| **能徒手画架构图** | 在白板上 5 分钟画出 Producer → 3 副本 → Consumer Group → Coordinator 全链路 | ⭐⭐⭐ |
| **能源码定位** | “这个错误对应 `Sender#completeBatch` 中的 `OutOfOrderSequenceException`” | ⭐⭐⭐ |
| **能讲清协议流程** | “事务 commit 是 2PC：先 PREPARE\_COMMIT 写 `__transaction_state`，再 WriteTxnMarker 写各分区” | ⭐⭐⭐ |
| **能给出量化数据** | “我们集群 8 broker，日均 100 亿消息，P99 写入 25ms，PageCache 命中率 96%” | ⭐⭐⭐ |
| **能对比技术方案** | “ISR 是 PacificA 协议变种，与 Raft Quorum 的核心区别是 N 副本全同步 vs 多数派” | ⭐⭐⭐ |

#### 15.1.2 深度技术点（高频被追问）

#### 15.1.3 数据驱动的项目讲法（金句模板）

不要说：“我们用 Kafka 做了订单消息处理”

推荐说法（先数据、再三条可验证动作）：

> **我们订单中心每天处理约 12 亿消息，峰值约 5 万 QPS；Kafka 集群 8 个 broker，单集群日均约 200TB 流量。** 核心优化我做了三块：一是把 `acks=1` 改成 `all` 并配上 `min.insync.replicas=2`、Producer 幂等与业务唯一键，把丢失率从约 **1/10 万压到 0**；二是引入 **Cooperative Sticky Assignor**，把日均 **Rebalance 从 50+ 降到 5 次以内**；三是完成 **KRaft** 后，**Controller 切换从分钟级降到秒级**。

**讲述模板（STAR + 数据）**：

```
S (Situation):  数据规模 + 业务背景T (Task):       具体责任 / 难点A (Action):     做了什么（参数 / 架构 / 工具）R (Result):     量化指标提升（百分比、绝对值、SLA）强化：+ 失败案例（曾经踩坑后修复）+ 业务影响（带来什么业务价值）+ 持续改进（后续还有什么计划）
```

#### 15.1.4 跨领域知识（资深岗 +）

* **JVM**

  ：能讲 G1 vs ZGC vs Shenandoah 的取舍（参考 Ch9.4），关键是"Kafka 不需要 ZGC"。
* **Linux 内核**

  ：PageCache、epoll、TCP 调优。
* **网络**

  ：ENI / RDMA / kTLS / TCP\_NODELAY 与 Kafka 的关系。
* **数据库 / 数据湖**

  ：CDC / Iceberg / Hudi / Doris / StarRocks 与 Kafka 的协同。
* **流处理**

  ：Flink Checkpoint / Watermark / 反压如何影响 Kafka 消费。
* **可观测性**

  ：OpenTelemetry / 链路追踪 traceparent header 透传到 Kafka。

#### 15.1.5 行业前沿（紧跟社区）

| 主题 | 关键词 |
| --- | --- |
| **KIP-405** Tiered Storage | 冷热分层、S3 卸载 |
| **KIP-848** Next-Gen Consumer | 服务端 rebalance、弃用 Eager 协议 |
| **KIP-500** KRaft 模式 GA | 4.0 完全去 ZK |
| **KIP-996** Pre-vote | 减少 KRaft 选举抖动 |
| **WarpStream / Bufstream** | 基于 S3 的 0 本地盘 Kafka 替代品 |
| **Confluent KORA** | Kafka 云原生重写引擎 |
| **AutoMQ** | 国内云原生 Kafka 替代 |

> 主动谈一两个最新动向（比如 WarpStream 基于 S3 的 0 本地盘架构、AutoMQ 在阿里云的实践），能立刻让面试官感受到候选人**还在持续学习**。

### 15.2 软实力加分项

#### 15.2.1 面试官评判候选人的隐性维度

| 维度 | 优秀表现 | 反例 |
| --- | --- | --- |
| **结构化思维** | 答题先讲背景 → 选项 → 决策 → 结果 | 想到啥讲啥，跳跃 |
| **诚实** | 不会的就说不会，承认局限 | 编造或瞎猜 |
| **沟通** | 5 分钟内讲清复杂概念 | 啰嗦或不抓重点 |
| **学习能力** | 主动谈最近读的 KIP / 论文 | 一年没看技术 |
| **业务理解** | 能讲技术决策对业务的影响 | 只关心技术指标 |
| **协作** | 提到与产品 / 测试 / 运维的协作 | 只讲个人英雄主义 |

#### 15.2.2 高情商应答模板

**面对不会的题**：

❌ “这个我不太了解……”

✅ “**这个具体细节我不确定，但根据 XXX 原理推测应该是 YYY，方便的话我面试完查证后回复邮件给您。**”

**面对失败案例**：

❌ “其实那次故障不是我的锅。”

✅ “**那次故障我作为主要排查人，第一时间确认了影响范围（XXX 业务受影响 XXX 时长），止血后做了根因分析（XXX），最终我们的改进是 XXX，现在线上运行 XXX 没再复发。**”

**面对压力问**：

❌ “我也是这么想的，但我们老大说这样做……”

✅ “**这个权衡当时确实有讨论，从 ABC 三个维度看 XXX 方案更合适，但我也很理解您说的考虑点。如果让我重新设计，我会再加 XXX 来兼顾。**”

### 15.3 反问环节 ── 你问面试官的问题

#### 15.3.1 反问的战略意义

**反问的目的**：

1. 判断公司 / 团队是否值得加入；
2. 体现技术深度（**好问题胜过好答案**）；
3. 转换攻防（让面试官回答你，建立对话感）；
4. 收集信息为后续面试 / 谈薪做准备。

**避坑**：

* ❌ 不要问公开资料能查到的东西（“贵公司主营业务是什么”）。
* ❌ 不要问很功利的（“几号发工资？”）— 留到 HR 面或 offer 谈判。
* ❌ 不要问你已经清楚不会满意的（除非测试面试官真诚度）。

#### 15.3.2 推荐反问清单（按面试阶段分）

##### 一面（技术/直接经理）

具体问题示例：

1. “贵团队的 Kafka 集群规模大概是多少？日均消息量在什么量级？”
2. “目前在用 ZK 还是 KRaft？如果还在 ZK，是否有升级计划？”
3. “团队最近半年最大的一次稳定性挑战是什么？怎么解决的？”
4. “Kafka 平台化做到什么程度了？是否有自助申请、容量评估、自动监控？”
5. “团队对 Kafka 的二次开发深度如何？是直接用社区版还是有内部 fork？”

##### 二面（部门 leader / 架构师）

具体问题示例：

1. “团队对 Kafka 的未来规划，是更倾向跟随社区，还是有内部演进路线？”
2. “整个数据链路（Kafka → Flink → 数据湖）的端到端 SLA 是怎么定的？”
3. “您觉得团队当前最缺哪种能力？我加入后如何补足？”
4. “未来 1~2 年团队最大的技术挑战是什么？”

##### 三面（HRBP / 大老板 / 跨部门）

1. “公司对 Kafka 这类基础设施团队的投入和定位是什么？”
2. “如果我加入，前 3 个月、6 个月、1 年的合理预期是什么？”
3. “您对一个资深工程师区别于普通工程师的核心要求是什么？”

##### HR 面

1. “贵司晋升周期一般多久？跨部门转岗是否容易？”
2. “团队的工作节奏 / 加班情况 / 弹性工作？”
3. “员工的成长支持（培训、外部会议、开源贡献）？”

#### 15.3.3 高质量反问示例（按场景）

**场景 A：你想去做技术深度 → 问反映技术含量的题**

> “我看您们公司在用 KRaft 模式了。能聊聊从 ZK 迁移到 KRaft 的实践吗？特别是迁移过程中遇到的最大坑？”

效果：体现你了解技术演进 + 关心实战经验。

**场景 B：你想去做业务影响 → 问业务相关题**

> “我注意到您们最近在做实时风控。从消息系统的视角看，这个业务对 Kafka 的延迟、丢失、有序性的诉求是什么？团队是怎么权衡的？”

效果：体现你能从业务反推技术。

**场景 C：你担心团队稳定性 → 问"故障"**

> “团队最近半年最严重的一次 Kafka 故障是什么？事后是怎么复盘的？有没有沉淀文档？”

效果：好的团队会大方分享 + 体现"复盘文化"。

**场景 D：你担心是否被压榨 → 问值班 / 节奏**

> “团队的值班机制是怎样的？月度 / 季度的节奏是什么样？”

效果：直接但合理，HR / 直接经理都该回答。

---

### 15.4 一句话压箱底（资深面试金句库）

把这些金句**背熟、改成自己的口吻**，面试时随手丢出立刻提级：

#### 关于设计

1. **“Kafka 是为吞吐而生，RocketMQ 是为业务消息而生。”**
2. **“PageCache 是 Kafka 高性能的灵魂，堆内存只是配菜。”**
3. **“ISR 不是 Raft，是 PacificA 协议的变种 ── 全副本同步换更少的副本数。”**
4. **“acks=all + min.insync.replicas=2 + 副本 3，是金融级不丢的最低标配。”**
5. **“Kafka 的 EOS = 至少一次 + 业务幂等，所有 MQ 本质都这样。”**

#### 关于权衡

6. **“Partition 数不是越多越好 ── 单 Broker 超过 4000 就开始受 Controller 选举与 PageCache 命中率影响。”**
7. **“`acks=all + min.insync.replicas=1` 等价于 `acks=1`，这是最隐蔽的丢消息陷阱。”**
8. **“为什么不用 ZGC？因为 Kafka 堆只有 6~8GB，G1 已经够用，ZGC 反而要付读屏障开销。”**
9. **“Kafka 不主动 fsync 不是 bug 是 feature，依赖多副本而非单机磁盘是它快的根本。”**
10. **“分区数能加不能减，是 Kafka 设计的原罪之一。”**

#### 关于运维

11. **“网络往往先于磁盘成为瓶颈，3 副本入 1GB 出可能 5GB。”**
12. **“reassign-partitions 不带 throttle 等于自杀，限速是必备而不是可选。”**
13. **“监控 Lag 一个指标是不够的，要监控端到端时延 + 业务正确性。”**
14. **“Rebalance 风暴的根因往往是 GC，不是 Kafka 本身。”**

#### 关于业务

15. **“幂等是消息系统的最后一道防线，业务写入必须有唯一键。”**
16. **“消息回溯是 Kafka 相比传统 MQ 最强的能力，要善用而不是怕。”**
17. **“Outbox 模式是解决’DB 提交了消息没发’的标准答案。”**

---

---

## 🧭 章节导航

> 🏠 返回 README
>
> ⬅️ 上一模块：02-调优与运维.md ｜ ➡️ 下一模块：04-源码与OS底层.md

| 模块 | 文件 |
| --- | --- |
| 🔵 模块一·基础原理 | 01-基础与原理.md |
| 🟢 模块二·调优运维 | 02-调优与运维.md |
| 🟡 **模块三·设计编码（当前）** | `03-设计与编码.md` |
| 🔴 模块四·源码 OS | 04-源码与OS底层.md |
| 🟣 模块五·分布式理论 | 05-分布式理论与大厂设计.md |
| 📎 附录 | 99-附录.md |