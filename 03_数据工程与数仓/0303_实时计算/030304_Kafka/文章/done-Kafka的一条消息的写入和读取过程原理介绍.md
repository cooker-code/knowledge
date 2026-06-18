> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030304_Kafka/030304_核心知识点/Kafka副本一致性与ISR边界|Kafka副本一致性与ISR边界]]
---
title: Kafka的一条消息的写入和读取过程原理介绍
author: 跑享网
date:
url: https://mp.weixin.qq.com/s?__biz=MzU2MTg5ODU2NA==&mid=2247483705&idx=1&sn=b9ed28553a0b52fed40ab680eb63c8bc&chksm=fdcce879254db350163085b78a061276dda3675dda13b810faab90d8e5a981146b51707fff68&mpshare=1&scene=24&srcid=09098q3gJaiTZ72IGkVDNR3z&sharer_shareinfo=e90465bb4336714742d4da2255eb6e0f&sharer_shareinfo_first=e90465bb4336714742d4da2255eb6e0f#rd
---

### **一、消息写入过程（Producer → Broker）**

#### **1. 生产者发送消息**

* **分区选择**：

+ Producer 根据 Topic 的分区策略（如轮询、哈希键、自定义）将消息发送到某个 Partition。
+ 若指定了 `key`，则通过哈希算法确定 Partition，保证相同 Key 的消息分配到同一分区。

* **序列化与压缩**：

+ 消息（Key 和 Value）被序列化（如 Avro、JSON），可选压缩（Snappy、GZIP）以减少网络开销。

#### **2. Broker 处理写入请求**

* **写入 Leader 副本**：

+ Producer 向 Partition 的 Leader Broker 发送消息（通过 ZooKeeper 或 Metadata 缓存获取 Leader 信息）。
+ Leader 将消息以 **顺序追加（Append-Only）** 的方式写入本地 Log Segment（物理日志文件）。

* **副本同步（ISR 机制）**：

+ Leader 将消息同步到所有 In-Sync Replicas (ISR) 的 Follower 副本，确保数据冗余。
+ 同步方式：Follower 主动从 Leader 拉取（Pull）消息，保证最终一致性。

* **ACK 确认机制**：

+ `acks=0`：不等待确认（可能丢失数据）。
+ `acks=1`：仅 Leader 确认（默认，平衡可靠性和性能）。
+ `acks=all`：所有 ISR 副本确认（最高可靠性）。

+ Producer 通过 `acks` 参数控制可靠性：

#### **3. 日志存储**

* **分段存储**：

+ 每个 Partition 的日志被拆分为多个 Segment（如 `0000000000.log`），每个 Segment 大小可配置（如 1GB）。
+ 每个 Segment 包含索引文件（`.index` 和 `.timeindex`），加速消息查找。

---

### **二、消息读取过程（Consumer ← Broker）**

#### **1. 消费者订阅与分配**

* **消费者组（Consumer Group）**：

+ 消费者以 Group 为单位订阅 Topic，同一 Group 内的消费者分摊 Partition。
+ 每个 Partition 只能被 Group 内的一个 Consumer 读取（实现并行消费）。

* **分区分配策略**：

+ 通过 `partition.assignment.strategy` 配置策略（如 Range、Round-Robin、Sticky）。

#### **2. 拉取消息（Pull 模型）**

* **Offset 管理**：

+ Consumer 维护当前消费的 Offset（位置指针），记录在 Kafka 内部 Topic `__consumer_offsets` 或外部存储。
+ 消费者从当前 Offset 开始顺序读取消息。

* **零拷贝优化**：

+ Broker 通过 `sendfile` 系统调用直接将磁盘文件发送到网络，避免数据复制到用户空间。

* **批量拉取**：

+ 消费者通过 `fetch.min.bytes` 和 `fetch.max.wait.ms` 配置批量拉取消息，提升吞吐量。

#### **3. 消息投递语义**

* **至少一次（At Least Once）**：

+ 消费者处理完消息后手动提交 Offset（`enable.auto.commit=false`）。

* **至多一次（At Most Once）**：

+ 消费者自动提交 Offset，可能丢失未处理的消息。

* **精确一次（Exactly Once）**：

+ 需事务支持（Producer 和 Consumer 均启用事务）。

---

### **三、关键机制保障**

#### **1. 高可用性**

* **副本机制**：每个 Partition 有多个副本（Replica），Leader 故障时通过 Controller 选举新 Leader。
* **ISR 动态维护**：Broker 定期检查副本同步状态，落后副本会被移出 ISR。

#### **2. 高性能**

* **顺序 I/O**：消息追加写入磁盘，避免随机访问。
* **页缓存（Page Cache）**：利用操作系统缓存加速读写。
* **批处理**：Producer 和 Consumer 均支持批量操作。

#### **3. 水平扩展**

* **分区扩容**：增加 Partition 数可提升 Topic 的并行处理能力。
* **Broker 扩展**：新增 Broker 可分担负载。

---

### **四、流程图解**

text

```
写入过程： Producer → 选择 Partition → 发送到 Leader → 顺序写入 Log → 同步到 Follower → ACK 确认  读取过程： Consumer Group → 分配 Partition → 拉取消息（从 Offset）→ 处理消息 → 提交 Offset
```

---

### **总结**

Kafka 通过分区、副本、顺序 I/O 和批量处理等机制，实现了高吞吐、低延迟的消息传输。写入时依赖 ISR 和 ACK 机制保障可靠性，读取时通过消费者组和 Offset 管理实现灵活消费。理解这些原理有助于优化 Kafka 集群配置（如分区数、副本数、ACK 策略）和排查性能问题。