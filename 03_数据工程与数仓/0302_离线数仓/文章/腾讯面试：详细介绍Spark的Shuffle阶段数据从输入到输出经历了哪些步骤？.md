---
title: 腾讯面试：详细介绍Spark的Shuffle阶段数据从输入到输出经历了哪些步骤？
author: 大数据技能圈
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491384&idx=1&sn=5ab414f60fda52a3b9b88e1a7f2cdecd&chksm=c113a10aa96a19fca5a4bfc237c183f2bf19cf7b427a036a38eee98d207bed19316ce3fb2818&mpshare=1&scene=24&srcid=06118IboMSODm6P4U8lvFD21&sharer_shareinfo=b14deedc877396993fd7015a6f2f4901&sharer_shareinfo_first=b14deedc877396993fd7015a6f2f4901#rd
---

01

**加入知识星球《苦获陪你学大数据》将获得以下权益**

扫描二维码加入星球，每增加100人就会涨价，现在就是最优惠的

今天群里有同学问了这样一个问题：

群里面进行了非常热烈的讨论，我想有必要对Spark shuffle阶段进行一次详细的总结。

01

**Spark的Shuffle阶段：从数据输入到输出的详细解析**

## 一、Shuffle概述

### 1.1 Shuffle的定义与作用

Shuffle，中文可理解为“洗牌操作”，在分布式计算框架如Spark中，它是一个关键且复杂的机制，用于在某些操作期间对集群中的数据进行重新分配和组织。在Spark里，数据以分布式方式存储在集群的多个节点上，每个节点处理数据的一个子集，即分区（Partition）。Spark的操作可分为窄变换（Narrow Transformations）和宽变换（Wide Transformations）。窄变换如map、filter等，这些操作在单个分区上执行，无需数据在节点之间移动；而宽变换如groupBy、join、reduceByKey等，需要跨分区重新分配数据，因为一个分区的输出可能依赖于其他分区的数据。Shuffle就是在宽变换期间重新分配数据的过程，它确保相关数据（例如，groupBy中具有相同键的所有记录）被分组到同一节点上，以便进一步处理。

### 1.2 Shuffle的重要性

Shuffle在Spark中扮演着至关重要的角色，大多数需要分区键（partition key）重新组织数据的场景，如统计、汇总、分组、连接等操作都离不开它。然而，Shuffle也是Spark中最昂贵的操作之一，它不仅会带来网络IO，还会引发磁盘IO、数据序列化/反序列化、JVM垃圾回收等操作，因此其性能往往决定着Spark作业的整体性能。

### 1.3 Shuffle的演进

Spark的Shuffle机制经历了不断的演进和优化，以提高性能和处理大规模数据的能力。

* **Hash Shuffle（早期）**

  ：基于哈希分区，容易在大数据量下产生大量小文件，后续逐步弃用。在Spark 1.2之前，HashShuffleManager是默认的Shuffle管理模式。在未经优化的情况下，每个执行Shuffle Write的任务，会为下一个阶段的每个任务创建一个磁盘文件，导致Shuffle Write阶段产生大量的磁盘文件，增加了磁盘IO开销和文件系统压力。例如，若下一个阶段有100个任务，当前阶段的每个任务就要创建100份磁盘文件。为了优化这一问题，引入了合并机制，即通过设置`spark.shuffle.consolidateFiles`参数为`true`，可以复用磁盘文件，减少文件数量。
* **Sort Shuffle（Spark 1.2+ 默认）**

  ：以排序和聚合的方式减少小文件数，通过排序将相同分区数据集中到一起，写到更少的文件中。SortShuffleManager的运行机制主要分为普通运行机制和bypass运行机制。当shuffle read task的数量小于等于`spark.shuffle.sort.bypassMergeThreshold`参数的值（默认为200），且不是聚合类的shuffle算子时，会启用bypass机制。普通运行机制下，数据会先写入内存数据结构，根据不同的shuffle算子选用不同的数据结构，如reduceByKey会选用Map数据结构进行局部聚合，join会选用Array数据结构直接写入内存。当达到临界阈值时，会将内存数据结构中的数据溢写到磁盘，并在溢写前对数据进行排序，分批写入磁盘文件，最后将所有临时文件合并成一个磁盘文件，并创建一个单独的索引文件。bypass机制下，任务会为每个下游任务创建一个临时磁盘文件，将数据按key进行hash后写入对应的磁盘文件，最后将所有临时磁盘文件合并成一个磁盘文件和一个索引文件，该机制不会进行排序操作，节省了排序的性能开销。
* **Tungsten Sort Shuffle**

  ：利用Tungsten引擎的内存管理和二进制处理进一步优化Shuffle。Tungsten Sort Shuffle是Sort Shuffle的优化版，它利用Tungsten的内存管理技术以及二进制处理加速Shuffle，减少内存占用和GC压力。一般在Spark 2.0+中默认已使用Tungsten - level优化（无需额外配置）。
* **Push - Based Shuffle（Spark 3.0+ 新特性）**

  ：在Map端预先将部分shuffle数据推给Reduce端，提升大规模Shuffle的性能并减少数据倾斜。该特性通过在Map阶段就将部分shuffle数据推送到下游节点或第三方存储，加速数据可用性并减少reduce端的拉取压力，适合大规模、长tail任务场景，减少数据倾斜问题。

## 二、Shuffle的触发

### 2.1 触发Shuffle的操作

在Spark中，以下几类操作通常会触发Shuffle：

* **重新调整分区操作**

  ：如`repartition`、`coalesce`（当`shuffle=true`时）。`repartition`会增加或减少RDD的分区数，必然会涉及数据的重新分配；`coalesce`在`shuffle=true`时，也会进行数据的重新分区。
* **基于Key的操作**

  ：如`groupByKey`、`reduceByKey`、`aggregateByKey`、`foldByKey`等。这些操作需要将具有相同键的数据聚集到一起进行处理，因此会触发Shuffle。例如，`groupByKey`会将键相同的所有值分组到一个迭代器中，`reduceByKey`会在每个分区内先进行局部聚合，然后再将相同键的数据聚集到一起进行最终的聚合。
* **关联操作**

  ：如`join`、`cogroup`等。`join`操作会将两个RDD中具有相同键的元素进行连接，`cogroup`会将两个RDD中具有相同键的元素分组到一起，这些操作都需要跨分区重新组织数据，从而触发Shuffle。

### 2.2 源码分析触发Shuffle的过程

在Spark的`DAGScheduler`（类`org.apache.spark.scheduler.DAGScheduler`）中，`submitJob`方法会分析RDD血缘关系并识别Shuffle依赖。当检测到Shuffle依赖（通过`ShuffleDependency`）时，会创建一个新的`ShuffleMapStage`。例如，以下是`submitJob`方法的部分伪代码：

```
```
def submitJob[T](  
    rdd: RDD[T],  
    func:(TaskContext, Iterator[T])=> _,  
    partitions: Seq[Int],  
    callSite: CallSite,  
    resultHandler:(Int, U)=>Unit,  
    properties: Properties  
): JobId ={  
// 检测Shuffle依赖并在需要时创建新阶段  
if(hasShuffleDependency){  
      createNewShuffleMapStage()  
}  
}
```
```

## 三、Shuffle的详细步骤

### 3.1 数据输入与转换操作

#### 3.1.1 数据输入

首先，需要创建一个Spark上下文并输入数据。以下是一个示例代码：

```
```
from pyspark import SparkConf, SparkContext  
# 创建Spark配置和上下文  
conf = SparkConf().setAppName("Shuffle Example").setMaster("local")  
sc = SparkContext(conf=conf)  
# 读入数据  
data =[("Alice",1),("Bob",2),("Alice",3),("Bob",4)]  
rdd = sc.parallelize(data)  
# 输出初始RDD   
print("初始RDD:", rdd.collect())
```
```

在上述代码中，`SparkConf().setAppName("Shuffle Example").setMaster("local")`设置了Spark应用的名称和运行模式，`sc.parallelize(data)`将数据转换为RDD（弹性分布式数据集），`rdd.collect()`收集并打印初始RDD的内容。

#### 3.1.2 执行转换操作

接下来，可以对RDD进行基本的转换操作，例如`map`，对数据进行处理。示例代码如下：

```
```
# 使用map转换  
mapped_rdd = rdd.map(lambda x:(x[0], x[1]*2))  
# 输出转换后的RDD   
print("转换后的RDD:", mapped_rdd.collect())
```
```

在这个例子中，`rdd.map(lambda x: (x[0], x[1] * 2))`将每个值乘以2，形成新的RDD。

### 3.2 Shuffle Write阶段

#### 3.2.1 整体流程

Shuffle Write阶段是由当前阶段的ShuffleMapTask执行的，主要任务是将每个任务处理的数据按照下游分区的规则进行划分，并将结果写入磁盘。上游任务（Map端）按照下游分区的规则将数据切分成多个bucket，每一个bucket对应下游需要读取的一个分区。默认使用Sort Shuffle时，数据会先写入到内存中的缓存区（sorter），再通过内存溢写（spill）到磁盘，同时进行排序和聚合压缩。如果数据足够大，可能会多次溢写到磁盘。最后，Map端会写出若干个文件，文件中按照分区将数据分块组织。当Map端任务完成后，会向Driver汇报该map任务输出的Shuffle数据位置信息（即ShuffleMapStatus）。

#### 3.2.2 HashShuffleManager的Shuffle Write

##### 3.2.2.1 未经优化的HashShuffleManager

假设每个Executor只有1个CPU core，在Shuffle Write阶段，将每个task处理的数据按照key进行hash算法，从而将相同key都写入同一个磁盘文件，而每一个磁盘文件都只属于下游stage的一个task。在数据写入磁盘之前，会先将数据写入内存缓冲区中，当内存缓冲区填满以后，才会溢写到磁盘文件中去。下一个stage的task有多少个，当前stage的每个task就要创建多少份磁盘文件。例如，当前stage有20个task，总共有4个Executor，每个Executor执行5个task，下一个stage总共有40个task，那么每个Executor上就要创建200个磁盘文件，所有Executor会创建800个磁盘文件。这种方式会产生大量的磁盘文件，增加磁盘IO开销和文件系统压力。

##### 3.2.2.2 优化后的HashShuffleManager

为了优化HashShuffleManager，可以启用参数`spark.shuffle.consolidateFiles`，该参数的默认值为`false`，将其设置为`true`即可开启优化机制。开启优化机制后，在shuffle write过程中，task不是为下游stage的每个task创建一个磁盘文件，而是会出现shuffleFileGroup的概念，每个shuffleFileGroup会对应一批磁盘文件，磁盘文件的数量与下游stage的task数量相同。一个Executor上有多少个CPU core，就可以并行执行多少个task。第一批并行执行的每个task都会创建一个shuffleFileGroup，并将数据写入对应的磁盘文件内。当执行下一批task时，下一批task会复用之前已有的shuffleFileGroup，包括其中的磁盘文件。这样可以有效将多个task的磁盘文件进行一定程度上的合并，从而大幅度减少磁盘文件的数量，进而提升shuffle write的性能。例如，上述例子中，优化后每个Executor上只需创建40个磁盘文件，所有Executor会创建160个磁盘文件。

#### 3.2.3 SortShuffleManager的Shuffle Write

##### 3.2.3.1 普通运行机制

在普通运行机制下，数据会先写入一个内存数据结构中。如果是`reduceByKey`这种聚合类的shuffle算子，会选用Map数据结构，一边通过Map进行局部聚合，一边写入内存；如果是`join`这种普通的shuffle算子，会选用Array数据结构，直接写入内存。接着，每写一条数据进入内存数据结构之后，就会判断是否达到某个临界阈值。如果达到临界阈值，就会尝试将内存数据结构中的数据溢写到磁盘，然后清空内存数据结构。在溢写到磁盘文件之前，会先根据key对内存数据结构中已有的数据进行排序。排序过后，会分批将数据写入磁盘文件，默认的batch数量是10000条。一个task将所有数据写入内存数据结构的过程中，会发生多次磁盘溢写操作，产生多个临时文件。最后，会将之前所有的临时磁盘文件都进行合并，同时单独写一份索引文件，标识下游各个task的数据在文件中的起始偏移量和长度。

##### 3.2.3.2 bypass运行机制

bypass运行机制的触发条件为：shuffle map task数量小于`spark.shuffle.sort.bypassMergeThreshold`参数的值，且不是聚合类的shuffle算子（比如`reduceByKey`）。在这种机制下，任务会为每个下游task都创建一个临时磁盘文件，将数据按key进行hash，将key的hash值写入对应的磁盘文件之中，写入磁盘也是先写入内存缓冲，再溢写到磁盘的方式。最后，会将所有临时磁盘文件都合并成一个磁盘文件，并创建一个单独的索引文件。与普通SortShuffleManager运行机制不同的是，bypass机制不会进行排序操作，节省了排序的性能开销。

### 3.3 Shuffle数据传输

在Shuffle Write阶段完成后，数据会被存储在各个节点的磁盘上。当下游阶段的任务需要这些数据时，就会触发Shuffle数据传输。下游任务（Reduce端）会根据ShuffleMapStatus中的文件位置，从相应的节点拉取（Fetch）自己需要的分区数据。每个Reduce任务只会取自己相应的分区数据，然后进行后续的处理（如聚合、join等）。如果开启了External Shuffle Service，则数据是从External Shuffle Service进程中取，这样可以在Executor退出时保留Shuffle文件，避免因为Executor重启导致无法读取Shuffle文件的情况。

### 3.4 Shuffle Read阶段

#### 3.4.1 数据拉取

下游阶段的ReduceTask会根据Driver中的MapOutputTrackerMaster提供的ShuffleMapStatus信息，知道每个ShuffleMapTask的输出数据所在的位置。然后，ReduceTask会从各个节点拉取自己需要的数据。在拉取数据时，会使用`BlockStoreShuffleFetcher`等工具，通过网络将数据从远程节点传输到本地。拉取过程是一边拉取一边进行聚合的，每个shuffle read task都有一个自己的buffer缓冲，每次只能拉取与buffer缓冲相同大小的数据，然后在内存中进行聚合等操作，聚合完一批数据，再拉取下一批，以此类推，直到所有数据拉取完，并得到最终结果。

#### 3.4.2 数据处理

拉取过来的数据会组成一个内部的ShuffleRDD，优先放入内存，内存不够用则放入磁盘。在数据处理过程中，如果在Shuffle Write阶段开启了`mapSideCombine`（如`reduceByKey`默认开启），在Shuffle Read阶段也会进行合并操作。例如，在shuffle write阶段，某个分区的数据为`[(hello, 1), (hello, 1)]`，经过`mapSideCombine`后变为`[(hello, 2)]`。在shuffle read阶段，会将多个分区中相同键的数据进行合并，最终得到聚合结果。排序操作通常发生在shuffle read阶段，在进行完`mapSideCombine`之后，就开始进行排序了。

### 3.5 执行Shuffle后的操作与数据输出

#### 3.5.1 执行Shuffle后的操作

在完成Shuffle Read阶段后，可以继续对Shuffle结果进行操作，例如进一步的转换或过滤。以下是一个示例代码：

```
```
# 使用filter操作  
filtered_rdd = result_rdd.filter(lambda x: x[1]>2)  
# 输出过滤后的RDD   
print("过滤后的RDD:", filtered_rdd.collect())
```
```

在这个例子中，`result_rdd.filter(lambda x: x[1] > 2)`过滤出聚合值大于2的数据。

#### 3.5.2 数据输出

最后，可以将结果保存到文件中或者进行其他操作。示例代码如下：

```
```
# 将结果保存为文本文件  
filtered_rdd.saveAsTextFile("output/result.txt")  
# 停止Spark上下文  
sc.stop()
```
```

在上述代码中，`filtered_rdd.saveAsTextFile("output/result.txt")`将结果保存到指定的文本文件，`sc.stop()`停止Spark上下文，释放资源。

## 四、Shuffle的性能问题与优化策略

### 4.1 Shuffle的性能问题

#### 4.1.1 增加网络I/O

Shuffle操作涉及跨网络的数据交换和传输，导致较高的网络输入/输出（I/O）开销。shuffle数据量的增加会使网络资源紧张，从而导致执行时间变慢并降低总体吞吐量。例如，在大规模数据处理时，大量的数据需要在节点之间传输，会占用大量的网络带宽，影响作业的执行效率。

#### 4.1.2 资源密集型

Shuffle需要额外的计算资源，包括CPU、内存和磁盘I/O。shuffle期间资源利用率的增加会导致资源争用、作业执行时间延长和效率降低。例如，在Shuffle过程中，如果内存不足，可能会导致GC（垃圾回收）频率提高，从而影响性能；同时，大量的磁盘读写操作也会增加磁盘I/O的压力。

#### 4.1.3 产生大量小文件

在早期的HashShuffleManager中，会产生大量的小文件，这会增加磁盘IO开销和文件系统压力。例如，每个MapTask为每个下游分区都生成一个文件，如果下游有很多分区，就会生成大量小文件，导致文件系统崩溃的风险增加。

### 4.2 Shuffle的优化策略

#### 4.2.1 减少Shuffle的次数

在某些情况下，可以通过调整计算逻辑，减少Shuffle的次数。例如，使用`reduceByKey`替代`groupByKey`可以减少数据的Shuffle，因为前者在Map阶段就进行数据合并。示例代码如下：

```
```
val result = rdd.reduceByKey((a, b)=> a + b)
```
```

#### 4.2.2 增加并行度

通过增加分区数来提高并行处理能力，可以使用`repartition`方法来增加分区数。示例代码如下：

```
```
val repartitionedRDD = rdd.repartition(numPartitions)
```
```

#### 4.2.3 使用Tungsten执行引擎

Spark的Tungsten项目通过物理计划优化、代码生成和内存管理等技术，显著提高了Shuffle的性能。开启Tungsten，通常在使用DataFrame和Dataset时自动生效。

#### 4.2.4 避免长链Shuffle

将复杂的操作链简化为较少的Shuffle任务。例如，避免多次`groupBy`操作，可以先合并数据，再进行分组。

#### 4.2.5 优化内存管理

调节Spark配置，如`spark.memory.fraction`和`spark.memory.storageFraction`，以有效利用内存。

#### 4.2.6 其他优化策略

* **减少列并过滤行**

  ：减少混洗的列数并在混洗之前过滤掉不必要的行可以显著减少传输的数据量。通过在管道中尽早消除不相关的数据，最大限度地减少shuffle的影响并提高整体性能。
* **使用广播哈希连接**

  ：广播哈希连接是一种将连接操作的较小数据集广播到所有工作节点的技术，从而减少shuffle的需要。这种方法利用内存复制并消除与shuffle相关的网络开销，从而提高连接性能。
* **使用分桶技术**

  ：Bucketing是一种基于哈希函数将数据组织到桶中的技术。通过预先分区并将数据存储在桶中，Spark可以避免在连接和聚合等操作期间进行shuffle。这种优化技术减少了跨分区的数据移动，从而缩短了执行时间。

Spark的Shuffle阶段是一个复杂且关键的过程，它在分布式计算中起着重要的作用，确保了数据的重新分配和组织，使得各种复杂的数据处理操作得以实现。然而，Shuffle也是Spark作业性能的瓶颈之一，涉及大量的网络IO、磁盘IO和资源消耗。通过了解Shuffle的触发条件、详细步骤以及性能优化策略，可以更好地优化Spark作业，提高数据处理效率。在实际应用中，需要根据具体的业务场景和数据特点，选择合适的Shuffle管理模式和优化策略，以充分发挥Spark的性能优势。

以上。

获取更多信息，关注大数据技能圈

**欢迎添加作者交流**

## 推荐阅读系列文章

* [腾讯面试：Spark内存如何优化？包含哪几个方面？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491167&idx=1&sn=d62e305f6803f484aa40469393b1fdb8&scene=21#wechat_redirect)
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