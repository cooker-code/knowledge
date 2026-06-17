---
title: 这个流场景，Spark 跟 Flink 的表现，谁更聪明？
author: 安瑞哥是码农
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI0OTEwNzQyNA==&mid=2247488849&idx=1&sn=21456fa8b8d34998c0e6c721cc4c29cc&chksm=e8d0d5e5e4fd10c8f65db94a61430ba077f4e9421d3430c0fd94faf029250e4d52bae339c3d6&mpshare=1&scene=24&srcid=1106ZVrFkRHqtjbPCFIPSC3s&sharer_shareinfo=462727cab963a188a0c2b8a1f5e3434c&sharer_shareinfo_first=462727cab963a188a0c2b8a1f5e3434c#rd
---

前两天把 Flink 升级到了次新版的 1.19.1，就想着用新版本的 Flink 来解决我的一些数据问题，是不是应该更加的得心应手。

面对的具体场景呢，是我有一批文本数据文件，总数大概 1k+，它从源端以串行的方式，挨个生成，每生成一个就传过来一个，因为文件大小不一，文件生成的时间也就不固定，所以传过来的频率也是不固定的。

这个时候，就想着如果程序能持续监听一个目录，把源端生成的数据文件，都通通放到这个固定目录下，每放一个，程序就处理一个，类似流的数据处理方式。

而这种处理方式，刚好 Spark 跟 Flink ，理论上也都是支持的，那么这一次，我们就针对这个应用场景，来看看，Spark 跟 Flink，谁支持的更好？

***0. 场景设计***

从 Spark 跟 Flink 各自的官网可以知道，它们都支持以「流」的方式来读取某个文件夹下面的文件。

*Flink官网的支持说明*

*Spark官网的支持说明*

理论上的支持，并不代表就好用，能不能解决我们的实际问题，还真不好说，毕竟，类似打脸的案例多了去了。

这一次呢，我们就基于这样一个，以流方式读取 HDFS 文件夹下的数据文件，然后对文件中数据内容做简单的聚合计算，最后将聚合结果写 Clickhouse 的场景。

看 Spark 跟 Flink，谁能更好地满足这个场景的需求，具体的数据处理流程如下图：

经过了 4 天 5 夜的对比测试，结果表明：一个表现拉胯、一个表现尚可！

你能猜出它们分别是谁不？

***1. 核心代码对比***

***1.1 数据源读取方式对比***

先看 Spark 以流的方式读取文件夹：

```
val schema = new StructType()  
            .add("line", StringType) //将整行做为一个字段  
  
val rawDF = spark.readStream  
            .option("header", false)  
            .schema(schema)  
            .text("hdfs://namenode_ip:8020/the/files_dir/")
```

再看 Flink 以流的方式读取文件夹：

```
val rawDS = FileSource.forRecordStreamFormat(new TextLineInputFormat,new Path("hdfs://namenode_ip:8020/the/files_dir"))  
                      .monitorContinuously(Duration.ofSeconds(10))  
                      .build()
```

各有风格，各有特点。

***1.2 聚合逻辑对比***

因为对于这次数据的处理，是一个类似 world count 的简单聚合，但 Spark 跟 Flink 的聚合算子有些不一样，所以它们各自的聚合逻辑写法，也会有略微差别。

先看 Spark 的聚合逻辑：

```
.groupByKey(_._1)  
.reduceGroups((kv1, kv2) => {  
    val value1 = kv1._2  
    val value2 = kv2._2  
    (kv1._1, value1 + "," + value2)  
})
```

再看 Flink 的聚合逻辑：

```
.keyBy(_._1)  
.reduce((kv1, kv2) => {  
    val value1 = kv1._2  
    val value2 = kv2._2  
    (kv1._1, value1 + "," + value2)  
})
```

可以看到，除了算子名字不一样外，逻辑是一毛一样的，其他非聚合部分的数据处理逻辑也是一样的，这里就不贴出来了。

***1.3 结果 sink 逻辑对比***

这里我特意让它们都用了非常类似的逻辑。

Spark 的实现逻辑是这样子的：

```
.foreach(new ForeachWriter[Row] {  
                ......  
                override def open(partitionId: Long, epochId: Long): Boolean = {  
                  ......  
                }  
                override def process(value: Row): Unit = {  
                  ......  
                }  
                override def close(errorOrNull: Throwable): Unit = {}  
        })
```

Flink 是这样子的：

```
.addSink(new RichSinkFunction[(String, String, String, String)] {  
            ......  
            override def open(parameters: Configuration): Unit = {  
                ......  
            }  
            override def invoke(value: (String, String, String, String), context: SinkFunction.Context): Unit = {  
                ......  
            }  
            override def close(): Unit = {}  
        })
```

是不是贼像？

***2. 运行方式可行性对比***

这里的可行性，指的是 Spark 跟 Flink 是不是都可以非常畅快的用「流」的方式给跑起来。

对 Spark 来说：***可以，能够很正常的，跟其他流任务一样(比如当数据源为 Kafka 时)跑起来***。

对 Flink 来说：***不行，跑一会就会挂掉***。

Flink 之所以挂掉，原因在这里：

怎么说呢，就很不聪明。

对于 HDFS 这个文件系统来说，当有文件要写入(或者 copy)到某个目录时，就回先生成一个以 *\_COPYING\_* 的临时文件，当数据全部写入(copy完)，才会重命名为正式的文件名。

***在识别文件这一点上，显然 Spark 的经验更加老道，当文件还处于「生成中」状态时，它是不会去读的***。

但 Flink 则不一样，它傻不拉几的才不管文件是什么状态呢，都照单全收，比如这个以 *\_COPYING\_ 结尾*的临时文件，然后读着读着，当文件生成完毕，这个后缀就没了，然后它就因为找不到这个文件名报错了。

 

***3. 计算结果对比***

对于 Flink，想要这个测试进行下去，就只能先把文件 copy 到目录里，然后再启动 Flink 进程。

 先看一条 Spark 写入到 CK 的聚合结果：

同样的 domain(key)，用 Flink 聚合后，写入到 CK 就是这样的：

(其他数据的聚合结果都类似，这里不一一举例)

***很明显，Spark 写入到 CK 的聚合结果，才是我们想要的(以文件为单位聚合的批思想)***。

而 Flink，很显然***把整个计算过程的中间结果，也给写了进来(典型的，来一条聚合一条的流处理思想)***。

但有意思的是，即便我把 Flink 的运行模式从 「streaming mode」改为 「batch mode」，这个计算结果，也没有得到丝毫改变。

我就想知道，Flink 这个所谓的 「batch mode」意义是啥，是设计出来骗三岁小孩的吗？(想最终解决这个问题，只能通过表引擎的设置来达到目的)

***最后***

从这次以流的方式处理 HDFS 目录下数据文件的场景来看，虽然 Spark 跟 Flink 的官方文档都宣称支持，代码逻辑也都非常类似。

然而，跑起来后的实测结果表明，Spark 的表现要更加智能，能满足我当前的业务需求。

而 Flink 呢，就显得不够灵活聪明，不能满足我当前的业务需求。

---

你可以添加我的私人微信，拉你入技术讨论群，跟一群热爱技术的小伙伴一起成长...