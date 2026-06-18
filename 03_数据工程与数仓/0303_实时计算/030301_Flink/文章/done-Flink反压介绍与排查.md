> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink反压排查入口|Flink反压排查入口]]
---
title: Flink反压介绍与排查
author: 狗不理咖啡
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5OTA1NDYwNw==&mid=2247483660&idx=1&sn=25995f079fd5db83ce6dc9bb294b7acb&chksm=97d56687aad4576e46e2063789a85a6f817b6e653d5e0e39e299eb98cb6b1f601a51bda5a4d4&mpshare=1&scene=24&srcid=0916zAWfHzwGGhXG2eCkPqG7&sharer_shareinfo=0deda5ebee0e925d8a542dea26091115&sharer_shareinfo_first=0deda5ebee0e925d8a542dea26091115#rd
---

## 一、什么是 Flink 反压（Backpressure）？

**反压** 指的是：**下游消费不过来，上游被迫减慢处理速度的机制**，确保系统整体稳定，不会因为局部阻塞导致 OOM 或任务崩溃。

简单理解：

* **下游处理慢**

  → 缓冲区（Buffer）堆积 → 上游生产不得不减速 → 逐级向源头“反向施压”。

---

## 二、Flink 反压产生的原因

| 典型场景 | 说明 |
| --- | --- |
| **下游算子耗时过长** | 比如 Sink（写外部存储慢）、复杂 Map / Join 操作耗时。 |
| **数据倾斜（数据分布不均匀）** | 某些 Key 数据量巨大，导致部分 Task 负载过高。 |
| **外部系统写入瓶颈** | Kafka Sink、HBase、Elasticsearch 写入速度跟不上。 |
| **Buffer 溢出** | Flink 的网络缓冲区被填满（默认是 TaskManager 内存的一部分），无法继续接受数据。 |
| **反压链传递** | 一个 Task 的反压会逐级向上传递，导致上游 Source 也被拖慢。 |

---

## 三、Flink 反压传播机制

1. Flink 任务之间通过 **Buffer（缓冲区）** 传输数据。
2. 如果下游 Task 无法及时消费，Buffer 就会被填满。
3. 一旦 Buffer 被填满，上游 Task 的数据无法继续发送，**会阻塞 send buffer**，导致上游也慢下来。
4. 这种“阻塞机制”会逐层向上游传播，最终让 Source 降低生产速度。

---

## 四、反压的检测与排查

### 1. **Flink Web UI 排查反压**

* 进入 Flink Dashboard → TaskManager 页面
* 找到 **Backpressure Tab**

+ **Green（绿色）**

  ：没有反压
+ **Yellow（黄色）**

  ：有轻微反压
+ **Red（红色）**

  ：严重反压，Task 已被阻塞

* **累积延迟（Subtask Backpressure Time）**

  显示具体反压比例

### 2. **Metrics 指标监控**

* `task.operator.record.currentOutputWatermark`
* `task.operator.record.currentInputWatermark`
* `task.operator.record.latency`
* `task.numRecordsOutPerSecond`
* `task.numRecordsInPerSecond`

通过指标可以判断：

* 输入速率高，但输出速率低 → 可能存在反压
* Watermark 卡在某一阶段不再推进 → 反压或数据积压

---

## 五、反压带来的影响

| 影响 | 说明 |
| --- | --- |
| **任务吞吐降低** | 整体作业吞吐会跟随最慢的 Task 减速。 |
| **Watermark 推不动** | 下游 Watermark 无法推进，导致事件时间窗口计算被阻塞。 |
| **状态 Checkpoint 频率降低** | Checkpoint Barrier 传递受阻，Checkpoint 延迟拉长。 |
| **严重时导致 OOM 或 Task Fail** | Buffer 溢出，JVM 内存不足导致 Task 崩溃。 |

---

## 六、反压优化手段

| 场景 | 优化策略 |
| --- | --- |
| **下游算子处理耗时** | 并行度调大，异步 IO（Async IO）、批量写入。 |
| **数据倾斜** | keyBy 改进（添加随机前缀）、增加并行度、重分区（rebalance）。 |
| **Buffer 配置** | 增加 TaskManager 内存，调整 `taskmanager.network.memory.fraction`。 |
| **反压链优化** | Operator Chain 拆分，减少链式执行压力。 |
| **Sink 写入优化** | 外部系统（HBase/ES/Kafka）批量写、缓冲写策略优化。 |

---

## 七、反压链路示意图

##

```

```

---

## 重点总结：

1. Flink 反压是 Task 之间的流控保护机制，避免下游处理不过来导致 OOM。
2. 反压检测主要靠 WebUI 的 Backpressure 指标、Metrics 监控。
3. 反压优化的关键在于：“提升下游消费能力”、“打散数据倾斜”、“加大 Buffer & 并行度”。