> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink反压排查入口|Flink反压排查入口]]
---
title: 美团面试：Flink 反压机制（Backpressure）是如何实现的？出现反压如何解决?
author: 大数据技能圈
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491085&idx=1&sn=cd002b5df8532107fc5a53303bfb4afa&chksm=c10f1565848dde21bd0c64f25574cd33b3b79301061fa21611c8d1f86685db6053e0b4e801b8&mpshare=1&scene=24&srcid=0426UmyoROEVFhuvzYsLHgtn&sharer_shareinfo=eda7b828fadbda202026be84e6cd32e7&sharer_shareinfo_first=eda7b828fadbda202026be84e6cd32e7#rd
---

说在前面

**大数据200道面试题的起源**

我编写"大数据面试200道答案"系列的主要动机，源于对大数据技术全方位、无死角的深入理解与实践需求。

通过系统梳理从Hadoop、Spark、Flink到StarRocks、Doris等主流大数据组件的核心原理，我希望能帮助技术人员建立完整的知识体系，不仅了解"是什么"，更要深入探究"为什么"和"如何做"。

这套面试题覆盖了从理论基础到架构设计，从性能调优到实际应用场景的全面知识点，旨在让学习者能够融会贯通，将理论与实践紧密结合。

特别是在性能优化方面，通过解析各组件内部实现机制和调优方法，帮助开发者应对高并发、大数据量、低延迟等复杂业务挑战，最终实现从入门到精通的技术飞跃，提升在大数据领域的核心竞争力。

这套大数据组件面试题答案，我已上传知识星球，可在文末扫码获取。

01

**Flink 反压机制（Backpressure）是如何实现的？**

反压（Backpressure）是流处理系统中的一种重要机制，用于处理数据生产速率超 过消费速率的情况。Flink的反压机制是完全基于Credit的流量控制实现的，能够 自动适应负载变化，保证系统稳定运行。下面将从源码入手解释反压的运行机制。

01

反压机制概述

Flink 的反压机制基于以下原则：

1. 消费者（下游算子）控制生产者（上游算子）的数据发送速率

2. 使用基于信用（Credit-based）的流量控制

3. 无需配置，自动适应工作负载

核心组件：

* ResultPartition：上游任务生产的数据分区
* InputGate：下游任务消费数据的入口
* LocalBufferPool：管理网络缓冲区的本地缓冲池
* NetworkBufferPool：全局网络缓冲区资源池

02

基于 Credit 的流量控制

Flink 1.5 以后采用基于Credit 的流量控制机制，其核心代码在 org.apache.flink.runtime.io.network.partition.consumer 和 org.apache.flink.runtime.io.network.partition.producer 包中。

核心类：org.apache.flink.runtime.io.network.api.serialization.EventSerializer

```
public class EventSerializer {    // 序列化和反序列化Buffer请求事件和Credit报告    public static ByteBuffer toSerializedEvent(AbstractEvent event) {        // ... 序列化逻辑    }        public static AbstractEvent fromSerializedEvent(ByteBuffer buffer, ClassLoader classLoader) {        // ... 反序列化逻辑    }}
```

```
// Credit报告事件类public class CreditMessage extends AbstractEvent {    private final int targetChannel;    private final int credit;    private final long backlog;        public CreditMessage(int targetChannel, int credit, long backlog) {        this.targetChannel = targetChannel;        this.credit = credit;        this.backlog = backlog;    }    // ...}
```

03

下游算子请求数据的过程

反压机制的核心是下游算子如何向上游算子请求数据，这个过程主要通过 RemoteInputChannel类实现：

```
public class RemoteInputChannel extends InputChannel {    // 向上游请求缓冲区    public void requestSubpartition(int subpartitionIndex) {        if (producerAddress == null) {            throw new IllegalStateException("Producer address unknown.");        }        // 构建请求消息        PartitionRequest request = new PartitionRequest(            partitionId,            subpartitionIndex,            inputChannelId,            credit);        // 将请求发送给生产者        try {            connectionManager.createPartitionRequestClient(producerAddress)                .requestSubpartition(request);        } catch (IOException e) {            // 异常处理...        }    }        // 发送Credit    private void sendCredit() {        // 计算剩余可用缓冲区数量作为信用值        int numCreditsAvailable =             Math.min(desiredNumCredits - numCreditsAndBacklog.getFirst(), Integer.MAX_VALUE);                if (numCreditsAvailable > 0) {            // 构建Credit消息            CreditMessage credit = new CreditMessage(                inputChannelId,                 numCreditsAvailable,                 numCreditsAndBacklog.getSecond());                            // 发送Credit给上游            connectionManager.sendCredit(producerAddress, credit);                        // 更新本地已发送的Credit            numCreditsAndBacklog.setFirst(numCreditsAndBacklog.getFirst() + numCreditsAvailable);        }    }}
```

04

上游算子接收请求并发送数据

上游算子通过ResultSubpartition接收并处理下游的请求：

```
public class PipelinedSubpartition extends ResultSubpartition {    // 添加缓冲区    public void add(BufferConsumer bufferConsumer) {        boolean notifyDataAvailable = false;                synchronized (buffers) {            // 添加到缓冲区队列            buffers.add(bufferConsumer);                        // 通知可能等待数据的消费者            if (buffers.size() == 1) {                notifyDataAvailable = true;            }        }                if (notifyDataAvailable) {            notifyDataAvailable();        }    }        // 通知数据可用    private void notifyDataAvailable() {        if (isAvailable && registeredListener != null) {            registeredListener.notifyDataAvailable();        }    }}
```

数据发送通过 CreditBasedPartitionRequestClientHandler 处理：

```
public class CreditBasedPartitionRequestClientHandler extends ChannelInboundHandlerAdapter {    // 处理Credit消息    private void handleCreditMessage(CreditMessage msg) {        RemoteInputChannel inputChannel = getInputChannel(msg.getReceiverId());                if (inputChannel != null) {            // 更新信用并尝试发送数据            inputChannel.updateCredit(msg.getCredit());            inputChannel.tryFinishPendingRequests();        }    }        // 发送数据    private void writeAndFlushNextMessageIfPossible() {        // 检查是否有足够的Credit发送数据        if (inputChannel.getCreditsAvailable() > 0 && inputChannel.hasDataToSend()) {            // 获取要发送的缓冲区            Buffer buffer = inputChannel.getNextBuffer();                        // 发送数据            NetworkUtils.sendDataWithHeader(                ctx,                 buffer,                 inputChannel.getReceiverId());                            // 减少可用Credit            inputChannel.decreaseCredit();        }    }}
```

05

反压传播机制

Flink的反压机制会从任务链尾部向上游传播：

```
public class StreamTask<OUT, OP extends StreamOperator<OUT>> extends AbstractInvokable {    // 处理输入数据    protected void processInput(OneInputStreamOperator<IN, OUT> operator,                                StreamTaskInput<IN> input) throws Exception {        while (true) {            // 尝试从input读取记录            DataInputStatus status = input.emitNext(operator.getInput());                        switch (status) {                case MORE_AVAILABLE:                    // 继续处理                    continue;                case NOTHING_AVAILABLE:                    // 暂无数据，返回                    return;                case END_OF_INPUT:                    // 输入结束                    endOfInput = true;                    return;                default:                    // 未知状态                    throw new RuntimeException("Unknown input status: " + status);            }        }    }}
```

当下游算子的处理能力不足时，它会减少请求上游数据的Credit，从而限制上游向 它发送数据的速率：

```
public class RemoteInputChannel extends InputChannel {    // 处理数据缓冲区    public void onBuffer(Buffer buffer, int sequenceNumber, int backlog) {        // 添加到缓冲区队列        boolean isEmpty = recycle(buffer, sequenceNumber);                // 更新积压量        updateBacklog(backlog);                // 可能需要发送更多Credit        if (isEmpty) {            sendCredit();        }    }        // 更新积压量    private void updateBacklog(int backlog) {        if (numCreditsAndBacklog.getSecond() != backlog) {            numCreditsAndBacklog.setSecond(backlog);                        // 如果积压量减少，可能需要发送更多Credit            if (backlog == 0) {                sendCredit();            }        }    }}
```

06

缓冲区管理

网络缓冲区管理是反压机制的核心，由NetworkBufferPool和LocalBufferPool 负责：

```
public class LocalBufferPool implements BufferPool {    // 全局缓冲区池    private final NetworkBufferPool networkBufferPool;    // 当前可用缓冲区    private final ArrayDeque<MemorySegment> availableMemorySegments;    // 缓冲区大小    private final int totalNumberOfMemorySegments;    // 最小缓冲区数量    private final int numberOfRequiredMemorySegments;        // 请求缓冲区    public Buffer requestBuffer() {        synchronized (availableMemorySegments) {            if (availableMemorySegments.isEmpty()) {                // 没有可用缓冲区，返回null                return null;            }                        // 从池中获取缓冲区            MemorySegment segment = availableMemorySegments.poll();                        // 创建缓冲区            return new NetworkBuffer(segment, this);        }    }        // 回收缓冲区    public void recycle(MemorySegment segment) {        synchronized (availableMemorySegments) {            // 回收到本地池            availableMemorySegments.add(segment);            // 通知等待者            availableMemorySegments.notifyAll();        }    }}
```

07

反压监控

Flink提供了反压监控功能，可以在Web UI中查看任务的反压状态：

```
public class BackPressureStatsTrackerImpl implements BackPressureStatsTracker {    // 收集反压统计信息    public void handleBackPressureStats(Map<JobVertexID, OperatorBackPressureStats> stats) {        synchronized (lock) {            // 更新统计信息            for (Map.Entry<JobVertexID, OperatorBackPressureStats> entry : stats.entrySet()) {                JobVertexID jobVertexId = entry.getKey();                OperatorBackPressureStats operatorStats = entry.getValue();                                // 存储反压信息                operatorStatsCache.put(jobVertexId, operatorStats);            }        }    }        // 获取某个任务的反压比率    public double getBackPressureRatio(JobVertexID jobVertexId) {        synchronized (lock) {            OperatorBackPressureStats stats = operatorStatsCache.get(jobVertexId);                        if (stats != null) {                return stats.getBackPressureRatio();            } else {                return 0.0;            }        }    }}
```

反压监控的实现在BackPressureRequestCoordinator类中：

```
public class BackPressureRequestCoordinator {    // 触发反压采样    public CompletableFuture<BackPressureStats> triggerBackPressureSampling(ExecutionJobVertex jobVertex) {        // 创建采样请求        BackPressureRequest request = new BackPressureRequest(jobVertex);                // 提交请求        return coordinator.submit(request);    }}
```

02

**反压机制总结**

Flink 反压机制的核心流程：

1. 下游算子根据其处理能力和可用缓冲区数量发送Credit给上游算子
2. 上游算子根据收到的Credit决定可以发送多少数据
3. 当下游处理速度跟不上时，Credit减少，上游发送速率自动降低
4. 反压通过任务链从下游向上游传播，直到Source算子
5. 整个过程无需配置，自动适应负载变化

Flink 的反压机制避免了传统流处理系统中可能出现的缓冲区溢出、数据丢失或 OOM 等问题，同时保证了系统的高吞吐和低延迟。

写在最后

**以上答案收入了《Flink源码分析面试题200道》**

具体目录可查看[Flink源码分析 经典面试题（200道）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)

扫描下方二维码，即可全部面试题答案

获取更多信息，关注大数据技能圈

**欢迎添加作者交流**

## 推荐阅读系列文章

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