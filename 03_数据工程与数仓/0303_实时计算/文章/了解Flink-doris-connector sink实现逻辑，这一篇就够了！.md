---
title: 了解Flink-doris-connector sink实现逻辑，这一篇就够了！
author: 哈逗噗哈呜
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwODIxMjExMQ==&mid=2247483926&idx=1&sn=3f42d8f5c5d577474d10ff79aa992d59&chksm=c15a7c288d544ddaa250958c0fdc950849acd51ed2b8e89674d41075806c91ce4651460dcc5f&mpshare=1&scene=24&srcid=0823p7CAI508e46wLH2xX3FF&sharer_shareinfo=01c3f7ba9b25af0322fd73e3dcbf2671&sharer_shareinfo_first=01c3f7ba9b25af0322fd73e3dcbf2671#rd
---

# 前言

    大家都知道Flink的三大逻辑结构：source、transform和sink，可以通过自定义实现的方式，来实现不同的sink。

                    

### RichOutputFormat 和RichSinkFunction

    在 Flink 中，RichOutputFormat 和 RichSinkFunction 都是用于处理数据输出的组件，且都属于 “富函数”（Rich Function）家族，但它们的设计背景、应用场景和使用方式存在显著差异。

     两者的核心关联在于都实现了 RichFunction 接口，因此具备以下共同特性：

* 拥有生命周期方法：open()（初始化资源）、close()（释放资源）。
* 可访问运行时上下文（RuntimeContext），支持获取广播变量、累加器、计数器等。
* 本质上都是为了在数据输出过程中提供更灵活的资源管理和上下文访问能力。

 2、区别

| 维度 | RichSinkFunction | RichOutputFormat |
| --- | --- | --- |
| 所属 API | 属于 DataStream API（流处理为主） | 属于 DataSet API（传统批处理） |
| 应用场景 | 主要用于 流处理，支持持续、实时的数据输出 | 主要用于 批处理，适合一次性批量数据输出 |
| 核心方法 | 重写 invoke() 方法，处理单条数据输出 | 重写 writeRecord() 方法，处理单条数据输出 |
| 使用方式 | 通过 DataStream.addSink(new MyRichSinkFunction()) 附加到流中 | 通过 DataSet.output(new MyRichOutputFormat()) 定义输出格式 |
| 设计目标 | 面向流处理的动态输出，支持与外部系统（如 Kafka、数据库）的实时交互 | 面向批处理的静态输出，早期主要用于文件系统、HDFS 等批量存储 |
| 现状 | 目前仍是流处理和批流统一模式（Flink 1.12+）中的主流输出方式 | 随着 DataSet API 逐渐被 DataStream 的批处理模式替代，使用场景减少 |

共性：两者均为 “富函数”，提供生命周期管理和上下文访问能力，用于数据输出。

差异：RichSinkFunction 适用于流处理（DataStream API），支持实时输出；RichOutputFormat 适用于传统批处理（DataSet API），适合批量输出。

    在当前 Flink 推荐的 “批流统一” 模式中，RichSinkFunction 更为常用，而 RichOutputFormat 更多存在于历史代码或特定批处理场景中。

 

### RichSinkFunction框架

    下面简单扫读一下RichSinkFunction相关源码。

    flink sink核心类是RichSinkFunction。Flink中提供了简单的SinkFunction接口，实现SinkFunction接口中的invoke方法即可简单实现自定义sink。

    而RichSinkFunction算是SinkFunction的加强版，它除了实现了SinkFunction接口，还继承了AbstractRichFunction 类。AbstractRichFunction 提供了open和close方法，可以通过重写这两个方法，在open方法中连接资源，在close方法中关闭资源、释放资源。当然还可以在open中做更多的操作，比如初始化一些线程池、定时任务、全局对象等。

这是AbstractRichFunction 类的源码：

 

## Flink sink doris简介

   flink-doris-connector通过定义GenericDorisSinkFunction继承RichSinkFunction并实现CheckpointedFunction来构造sink逻辑。

    值得一提的是flink-doris-connector沿用了DorisDynamicOutputFormat（继承RichOutputFormat）兼容批处理写入。实际上flink后来版本演进中，在批流统一模式下，即使处理批数据，也建议使用 RichSinkFunction了。

### 基本流程

#### 实现open方法:

①　初始化封装Doris StreamLoad导入功能的DorisStreamLoad类

②　定义一个大小为1的线程池执行定时flush的任务，参数sink.batch.interval控制定时任务执行间隔。

#### 实现invoke方法：

1）GenericDorisSinkFunction的invoke()直接调用DorisDynamicOutputFormat的writeRecord()写入数据。DorisDynamicOutputFormat中定义了一个batch的List，用来存放每次从上游来的数据row，在writeRecord被触发时，将row转成json格式存入batch List，并且每次判断batchList的长度是否超过了参数sink.batch.size的大小，超过则主动触发flush流程。

2）Flush时，将batch List转为json格式，调用doris的streamLoad写入doris，最后清空batch List。

如上，该阶段的sink逻辑依赖checkpoint，对于以下场景不太友好：

1）有些任务不太需要启用checkpoint；

2）复杂的Flink ETL任务会导致checkpoint写入很慢，这样会影响写入doris的性能。

因此，在后面的版本中，doris-flink-connector实现了一种不依赖checkpoint的batch写入流程。

### 支持batch-sink

    flink-doris-connector在1.5.0之后的版本中支持了不依赖checkpoint的batch sink，极大地提升了flink 写doris的效率。

    将之前task中的batch ArrayList拆成2个BlockingQueue：write-queue和read-queue。具体数据流向如下图：

#### 基本数据结构

    DorisBatchStreamLoad是doris-flink-connector中包装doris streamload的工具类，定义了writeQueue和readQueue。

    writeQueue用来申请buffer空间，队列元素的数据结构是BatchRecordBuffer，每个BatchRecordBuffer会申请一个大小为sink.buffer-flush.max-bytes的内存空间,该buffer储存从外部数据源读取的若干个字符数组byte[] record。

#### Flink sink doris流程

    DorisBatchWriter是实现Flink connector sink框架的write入口，

对于上游进来的每个IN数据，序列化出byte[]数组，

调用batchStreamLoad.writeRecord(serialize) ；对读取的每个byte[]数组执行写入doris流程：

1）从writeQueue中获取一个buffer空间，存入刚才的一条数据（byte[] IN）；

2）每次判断该buffer空间大小是否超过sink.buffer-flush.max-bytes\*80%，或者buffer内byte[]数组的个数超过sink.buffer-flush.max-rows，才执行flush。

3）Flush操作会将刚才的buffer存到readQueue中，然后会有一个异步线程LoadAsyncExecutor，不断从readQueue中取出buffer，执行Load调用doris的stream load接口完成写入。

4）还有个参数，sink.buffer-flush.interval，用来空batchStreamLoad.flush()的间隔。也就是控制readQueue获取buffer的间隔。

    就是说，除了每次接收上游数据时会check下是否满足flush的条件，后台也会有个定时任务定时触发flush写入doris。所以sink.buffer-flush.interval参数可以适当调大一些，可以降低写入doris的频率从而避免sink任务报Doris tablet version count超出限额的异常。sink.buffer-flush.max-bytes和sink.buffer-flush.max-rows参数也可以根据doris的性能要求调整。

### Batch-sink优化

    在flink-doris-connector后续版本中，社区又对batch sink的逻辑做了一些小优化，比如1.6.0版本的https://github.com/apache/doris-flink-connector/pull/309，统一了Abstract DorisAbstractWriter， 用参数sink.write-mode区分不同的写入模式（后续又增加了一种copy的写入模式）。

 在24.0.0版本的https://github.com/apache/doris-flink-connector/pull/462中提到，将write-queue和read-queue合并一个flush-queue， 减少内存内部数据拷贝的延迟，从而增加数据导入的吞吐。

 

### 支持copy模式

 在flink-doris-connector 1.6.0版本中，https://github.com/apache/doris-flink-connector/pull/312提到，当前sink.write-mode支持stream\_load, 和stream\_load\_batch，新引入一种copy的数据导入方式，用来应对doris存储和计算分离的场景，该导入方式分为两步：

1）将数据上传至对象存储；

2）从对象存储中加载数据写入Doris。

这种数据导入方式可以用于跨vpc数据同步的的场景。

完。