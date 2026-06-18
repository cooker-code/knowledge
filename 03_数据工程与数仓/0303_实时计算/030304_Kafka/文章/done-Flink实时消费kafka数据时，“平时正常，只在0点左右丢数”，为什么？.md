---
title: Flink实时消费kafka数据时，“平时正常，只在0点左右丢数”，为什么？
author: 阿龙大数据
date:
url: https://mp.weixin.qq.com/s?__biz=MzA3OTExMjA0MA==&mid=2247485501&idx=1&sn=489504373219615a18f8333c474d00b9&chksm=9ee0e02d0adc008bb142090cc7d2a3736cc639a09322c6df2559ffd673b533494b6c0a71024a&mpshare=1&scene=24&srcid=11106RsK3cDaxZlutZC5EL5x&sharer_shareinfo=aea5d176a9642da2c7d111551d7bbedf&sharer_shareinfo_first=aea5d176a9642da2c7d111551d7bbedf#rd
---

> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030304_Kafka/030304_核心知识点/Kafka消费滞后定位与治理|Kafka消费滞后定位与治理]]


Flink实时消费kafka数据时，“平时正常，只在0点左右丢数” .

# 一、平时正常，只在0点左右丢数

    这个问题在 Flink + Kafka 的架构中非常典型！0点这个时间点强烈暗示与**时间窗口、状态清理、定时任务或资源调度**相关。让我们聚焦 Flink 应用的特定问题。

**问题**：

    WatermarkStrategy.forMonotonousTimestamps()  这个参数为啥0点左右丢数？

**答案**：

    WatermarkStrategy.forMonotonousTimestamps()在0点左右丢数的根本原因是：**它基于处理时间而不是事件时间**，在0点这个特殊时间点会产生严重的时序混乱。

## 1. **forMonotonousTimestamps() 的工作原理**

```
// 它的本质是： WatermarkStrategy.forMonotonousTimestamps() = 使用处理时间作为事件时间
```

* **Watermark = 当前系统时间 - 1ms**
* **完全忽略数据中的真实时间戳**
* 假设数据是严格按顺序到达的（但CDC数据不可能）

## 2. **0点时的致命问题**

**场景模拟：23:59:59 → 00:00:00**

```
// 23:59:59.999 时刻当前系统时间 = 1704038399999LWatermark = 1704038399998L  // 正常// 00:00:00.001 时刻（跨越日期边界）当前系统时间 = 1704038400001L  // 新的一天开始Watermark = 1704038400000L    // 突然跳到很大值
```

**问题发生：**

```
// 假设在0点前到达的CDC数据包含真实的事件时间DataBean{    executeTime = 1704038399000L,  // 23:59:59    data = "23点的业务数据"}// 在0点后处理这条数据时：当前Watermark = 1704038400000L  // 00:00:00数据事件时间 = 1704038399000L    // 23:59:59// Flink判断：数据事件时间(23:59:59) < 当前Watermark(00:00:00)// ❌ 结论：这是"迟到数据"，默认被丢弃！
```

## 3. **CDC数据的特殊性加剧问题**

CDC数据在0点附近的特点：

```
 // 0点前积压的binlog在0点后集中消费[    {executeTime: 23:58:00, data: "A"},  // 被丢弃    {executeTime: 23:59:00, data: "B"},  // 被丢弃      {executeTime: 23:59:30, data: "C"},  // 被丢弃    {executeTime: 00:00:01, data: "D"},  // 正常处理    {executeTime: 00:00:02, data: "E"}   // 正常处理]
```

**结果**：0点前产生的数据全部被当作"迟到数据"丢弃！

## 4. **对比正确的Watermark策略**

```
// ❌ 你的问题代码WatermarkStrategy.forMonotonousTimestamps()// ✅ 修正后的代码 WatermarkStrategy.<DataBean>forBoundedOutOfOrderness(Duration.ofMinutes(5))    .withTimestampAssigner((event, timestamp) -> event.getExecuteTime())
```

**工作原理对比**：

```
// forMonotonousTimestamps() 在0点时：Watermark时间线: 23:59:59 → 00:00:00 → 00:00:01数据时间线:     23:59:00 → 23:59:30 → 00:00:01// 结果：23:59的数据在00:00被丢弃// forBoundedOutOfOrderness(5分钟) 在0点时：Watermark时间线: 23:55:00 → 23:56:00 → ... → 00:00:00数据时间线:     23:59:00 → 23:59:30 → 00:00:01  // 结果：所有数据正常处理，允许5分钟乱序
```

## 5. **验证这个问题的简单方法**

在你的代码中添加调试：

```
DataStreamSource<DataBean> sourceStream =    env.fromSource(KafkaSource, WatermarkStrategy.forMonotonousTimestamps(), KAFKA_SOURCE);// 添加处理函数观察数据丢弃sourceStream.process(new ProcessFunction<DataBean, DataBean>() {    @Override     public void processElement(DataBeanvalue, Context ctx,                               Collector<DataBean> out) {         long eventTime = value.getExecuteTime();        long currentWatermark = ctx.timerService().currentWatermark();        if (eventTime < currentWatermark) {             // 这里会打印出所有被丢弃的0点前数据            log.error("数据将被丢弃 - 事件时间: {}, Watermark: {}, 时间差: {}ms",                 new Date(eventTime), new Date(currentWatermark),                 currentWatermark - eventTime);        }         out.collect(value);    }});
```

## 6. **根本解决方案**

```
// 替换为适合CDC数据的Watermark策略WatermarkStrategy<DataBean> watermarkStrategy =     WatermarkStrategy.<DataBean>forBoundedOutOfOrderness(Duration.ofMinutes(5))        .withTimestampAssigner((event, timestamp) -> {             // 使用binlog的真实执行时间            return event.getExecuteTime().getTime();        })         .withIdleness(Duration.ofMinutes(1));DataStreamSource<DataBean> sourceStream =    env.fromSource(KafkaSource, watermarkStrategy, KAFKA_SOURCE);
```

## 7.总结

**根本原因**：

    forMonotonousTimestamps()在0点时间边界时，Watermark突然跳跃，导致0点前产生的数据全部被误判为"迟到数据"而丢弃。

**解决方案**：

    使用forMonotonousTimestamps()并设置合理的乱序时间窗口，让Flink能够正确处理跨时间边界的数据。

记录每一份热爱,让美好永远陪伴。