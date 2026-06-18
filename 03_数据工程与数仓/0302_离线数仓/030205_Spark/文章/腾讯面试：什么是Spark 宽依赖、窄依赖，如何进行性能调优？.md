---
title: 腾讯面试：什么是Spark 宽依赖、窄依赖，如何进行性能调优？
author: 大数据技能圈
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491564&idx=1&sn=4c803bdaf8cff6eff5625e731aa9c8c5&chksm=c16b487800936d0f5a8b9a324f737137b99e5b4eac01bf0763a94b7f1cefb11c17a61106bfbe&mpshare=1&scene=24&srcid=0711WUWU1QXbQh5dqMuWs9XN&sharer_shareinfo=d9c29358a0fff2ed8ca2d66fddd47688&sharer_shareinfo_first=d9c29358a0fff2ed8ca2d66fddd47688#rd
---

01

**加入知识星球《随川陪你学大数据》将获得以下权益**

扫描二维码加入星球，随加入人数增加，逐步涨价，现在就是最优惠的

01

**宽依赖、窄依赖与性能调优实践**

## 引言：Spark核心优化的基石

Apache Spark作为分布式计算框架的标杆，其性能优势很大程度上源于对计算流程的精细化控制。**RDD（弹性分布式数据集）** 作为Spark的核心抽象，不仅提供了分布式数据存储能力，更通过**依赖关系（Dependency）** 描述了数据转换的血缘关系（Lineage）。而**DAG（有向无环图）** 作为任务执行的逻辑表示，其划分与优化直接决定了Spark作业的并行效率。在实际生产中，开发者常面临任务延迟、资源利用率低、Shuffle开销过大等问题，其根源往往在于对依赖关系的理解不足或DAG优化策略的缺失。

本文将从RDD依赖关系的本质出发，深入剖析窄依赖与宽依赖的区别，详解Spark如何基于依赖关系进行DAG划分和Stage优化，并结合Spark 3.x的最新特性（如自适应查询执行AQE、动态分区裁剪DPP），提供一套完整的依赖关系调整与性能调优实践方案。

## 一、RDD依赖关系：从数据血缘到计算效率的关键

### 1.1 依赖关系的本质与分类

RDD的依赖关系描述了子RDD如何从父RDD转换而来，是Spark实现容错和并行计算的基础。根据父RDD分区被子RDD分区引用的方式，依赖关系分为**窄依赖（Narrow Dependency）** 和**宽依赖（Wide Dependency）**，二者的核心区别在于是否触发**Shuffle操作**。

#### 1.1.1 窄依赖：无Shuffle的高效并行

**窄依赖**指父RDD的每个分区最多被子RDD的一个分区引用，数据无需跨节点传输，可在单个Executor内完成转换。Spark将窄依赖操作合并为**流水线（Pipeline）** 执行，显著提升效率。窄依赖主要包括以下三种形式：

* **一对一依赖（OneToOneDependency）**：子RDD分区与父RDD分区一一对应，如`map`、`filter`算子。例如，对RDD执行`map(x => x*2)`时，子RDD的每个分区仅依赖父RDD相同索引的分区。
* **范围依赖（RangeDependency）**：父RDD的连续分区被子RDD的一个分区引用，典型场景为`Union`操作。例如，两个RDD通过`union`合并时，子RDD的分区0依赖第一个父RDD的分区0，分区1依赖第一个父RDD的分区1，以此类推。
* **协同分区Join（Co-partitioned Join）**：若两个RDD具有相同的分区器（Partitioner）和分区数，其Join操作可通过窄依赖实现。例如，RDD A和RDD B均按`user_id`分区，且分区数均为100，则Join时子RDD的分区i仅需关联A和B的分区i，无需Shuffle。

**代码示例：窄依赖转换**

```
```
// 一对一依赖：map操作  
val rdd1 = sc.parallelize(Array(1,2,3,4),2)// 2个分区  
val rdd2 = rdd1.map(_ *2)// rdd2的每个分区依赖rdd1的对应分区（窄依赖）  
  
// 范围依赖：union操作  
val rdd3 = sc.parallelize(Array(5,6),1)  
val rdd4 = rdd2.union(rdd3)// rdd4的前2个分区依赖rdd2，第3个分区依赖rdd3（窄依赖）  
  
// 协同分区Join  
val rddA = sc.parallelize(Seq(("a",1),("b",2))).partitionBy(new HashPartitioner(2))  
val rddB = sc.parallelize(Seq(("a",3),("b",4))).partitionBy(new HashPartitioner(2))  
val joinedRDD = rddA.join(rddB)// 分区器相同，为窄依赖Join
```
```

#### 1.1.2 宽依赖：Shuffle的性能瓶颈

**宽依赖**指父RDD的一个分区被多个子RDD分区引用，此时需通过**Shuffle**将数据跨节点重新分区。Shuffle过程涉及磁盘I/O、网络传输和序列化/反序列化，是Spark作业的主要性能瓶颈。常见触发宽依赖的算子包括`groupByKey`、`reduceByKey`（需Shuffle）、`join`（非协同分区时）等。

**宽依赖的底层机制**：以`groupByKey`为例，父RDD的每个分区数据会按Key哈希分配到不同子分区，过程中需将中间结果写入本地磁盘（Shuffle Write），再由子RDD的对应分区通过网络拉取（Shuffle Read）。此过程中，数据倾斜（某Key对应数据量过大）会导致部分Task耗时激增，进一步加剧性能问题。

**代码示例：宽依赖转换**

```
```
// groupByKey触发宽依赖  
val rdd = sc.parallelize(Seq(("a",1),("a",2),("b",3)),2)  
val groupedRDD = rdd.groupByKey()// 宽依赖：父RDD分区数据按Key重新分布  
  
// 非协同分区Join触发宽依赖  
val rddA = sc.parallelize(Seq(("a",1))).partitionBy(new HashPartitioner(2))  
val rddB = sc.parallelize(Seq(("a",2))).partitionBy(new HashPartitioner(3))// 分区数不同  
val shuffledJoinRDD = rddA.join(rddB)// 宽依赖：需Shuffle对齐分区
```
```

### 1.2 依赖关系对Spark执行的影响

依赖关系直接决定了Spark的**容错机制**和**任务并行度**：

* **容错效率**：窄依赖下，单个分区丢失仅需重算对应父分区；宽依赖则需重算所有父分区，恢复成本高。因此，Spark优先对宽依赖结果进行Checkpoint。
* **并行度**：窄依赖支持流水线执行（如`map -> filter -> map`可在单个Task内完成），而宽依赖会阻断流水线，需等待所有父分区完成才能开始子分区计算。
* **资源利用率**：宽依赖的Shuffle过程会导致大量网络传输和磁盘I/O，若配置不当（如分区数过少），易造成Executor资源空闲或过载。

## 二、DAG生成与Stage划分：从逻辑计划到物理执行

Spark将用户代码转换为DAG后，需通过**Stage划分**将逻辑计划转换为可执行的物理计划。Stage是Task的集合，每个Stage包含一组可并行执行的Task，其划分的核心依据是**宽依赖**。

### 2.1 DAG的构建过程

DAG的构建始于**RDD转换链**，终于**Action算子**（如`collect`、`count`）。当触发Action时，SparkContext会将RDD依赖链提交给**DAGScheduler**，由其构建DAG并划分Stage。例如，以下代码对应的DAG包含3个RDD转换：

```
```
val result = sc.textFile("data.txt")// RDD1：HadoopRDD  
.flatMap(_.split(" "))// RDD2：MapPartitionsRDD（窄依赖）  
.map((_,1))// RDD3：MapPartitionsRDD（窄依赖）  
.reduceByKey(_ + _)// RDD4：ShuffledRDD（宽依赖）  
.collect()// Action算子，触发DAG构建
```
```

### 2.2 Stage划分的核心算法：回溯与宽依赖检测

DAGScheduler采用**从后往前回溯**的算法划分Stage：

1. **起点**

   ：以Action算子对应的ResultStage（最终输出Stage）为起点。
2. **回溯依赖**

   ：遍历当前RDD的依赖关系，若为**窄依赖**，则将其父RDD合并到当前Stage；若为**宽依赖**，则以宽依赖为边界拆分Stage，父RDD作为新的ShuffleMapStage（需输出Shuffle结果）。
3. **递归处理**

   ：对新拆分的ShuffleMapStage重复上述过程，直至所有RDD均被划分到Stage中。

**示例**：上述WordCount代码的Stage划分如下：

* **Stage 1（ResultStage）**

  ：包含`reduceByKey`操作，依赖宽依赖，需等待Shuffle完成。
* **Stage 0（ShuffleMapStage）**

  ：包含`textFile -> flatMap -> map`操作，均为窄依赖，输出结果用于Shuffle。

**源码逻辑简化**：

```
```
// DAGScheduler核心划分逻辑（简化版）  
privatedef getParentStages(rdd: RDD[_]): List[Stage]={  
val parents = mutable.HashSet[Stage]()  
val visited = mutable.HashSet[RDD[_]]()  
val stack = mutable.Stack[RDD[_]](rdd)  
  
while(stack.nonEmpty){  
val currentRDD = stack.pop()  
if(!visited(currentRDD)){  
      visited.add(currentRDD)  
      currentRDD.dependencies.foreach {  
case narrowDep: NarrowDependency[_]=>  
          stack.push(narrowDep.rdd)// 窄依赖：合并到当前Stage  
case shuffleDep: ShuffleDependency[_, _, _]=>  
// 宽依赖：创建新的ShuffleMapStage  
val stage = getOrCreateShuffleMapStage(shuffleDep)  
          parents.add(stage)  
}  
}  
}  
  parents.toList  
}
```
```

### 2.3 Stage的任务类型与执行顺序

划分后的Stage分为两类：

* **ShuffleMapStage**

  ：输出结果用于Shuffle，对应`ShuffleMapTask`，任务数等于RDD分区数。
* **ResultStage**

  ：生成最终结果，对应`ResultTask`，任务数由Action算子决定（如`collect`对应1个任务）。

Stage的执行顺序遵循**依赖关系**：若Stage B依赖Stage A的Shuffle结果，则需等待Stage A完成后才能执行Stage B。Spark通过**广度优先调度**提交Stage，以最大化并行度。

## 三、Spark Stage优化策略：从静态配置到动态自适应

Stage优化是Spark性能调优的核心，涵盖Shuffle优化、内存管理、数据本地化等多个维度。Spark 3.x引入的**自适应查询执行（AQE）** 进一步实现了运行时动态优化，显著降低了人工调参成本。

### 3.1 基于依赖关系的静态优化

#### 3.1.1 减少宽依赖：算子选择与逻辑调整

宽依赖是性能瓶颈的主要来源，优化的核心是**避免不必要的Shuffle**或**减少Shuffle数据量**：

* **用`reduceByKey`替代`groupByKey`**：`reduceByKey`支持Map端预聚合（Combiner），减少Shuffle数据量。例如，对`(key, value)`按Key求和时，`reduceByKey(_ + _)`会先在每个分区内局部聚合，再Shuffle全局聚合；而`groupByKey().mapValues(_.sum)`需将所有Value传输到目标节点后聚合，数据量更大。

  **代码对比**：

  ```
  ```
  // 低效：groupByKey无预聚合  
  val groupResult = rdd.groupByKey().mapValues(_.sum)  
    
  // 高效：reduceByKey预聚合  
  val reduceResult = rdd.reduceByKey(_ + _)// 减少Shuffle数据量约80%
  ```
  ```
* **广播小表优化Join**：当Join的一个表较小时（如<100MB），使用`broadcast`将其广播到所有Executor，转为**Map端Join**，避免Shuffle。Spark 3.x可通过`spark.sql.autoBroadcastJoinThreshold`自动触发，也可手动指定：

  **代码示例**：

  ```
  ```
  importorg.apache.spark.sql.functions.broadcast  
    
  val smallDF = spark.read.parquet("small_table")  
  val largeDF = spark.read.parquet("large_table")  
  val joinedDF = largeDF.join(broadcast(smallDF),"id")// 广播小表，避免Shuffle
  ```
  ```

#### 3.1.2 分区策略优化：并行度与数据均衡

合理的分区数是提升并行度的关键。Spark推荐**每个Task处理128MB~256MB数据**，分区数设置为`Executor数量 × 核心数 × 2~3`。例如，50个Executor、每个4核的集群，推荐分区数为`50×4×2=400`。

* **`repartition`与`coalesce`**：`repartition`会触发Shuffle，用于增加分区或彻底重分区；`coalesce`不触发Shuffle，仅合并分区（适用于减少小文件）。

  **代码示例**：

  ```
  ```
  // 增加分区（触发Shuffle）  
  val repartitionedRDD = rdd.repartition(400)  
    
  // 合并分区（不触发Shuffle）  
  val coalescedRDD = rdd.coalesce(50)// 将100个小分区合并为50个
  ```
  ```
* **动态分区裁剪（DPP）**：Spark 3.0引入的DPP可在Join时基于运行时条件过滤无关分区。例如，`fact_table`按`date`分区，与`dim_table` Join时，若`dim_table`过滤出`date='2023-01-01'`，DPP会仅扫描`fact_table`的对应分区，减少I/O。

### 3.2 Spark 3.x自适应查询执行（AQE）：动态优化的革命

AQE（Adaptive Query Execution）是Spark 3.x的核心优化特性，通过**运行时统计信息**动态调整执行计划，解决静态优化的局限性。其三大核心功能如下：

#### 3.2.1 动态合并Shuffle分区

传统Shuffle分区数固定（默认200），易导致小分区过多（调度开销大）或大分区（数据倾斜）。AQE在Shuffle后根据实际数据量合并小分区，目标分区大小由`spark.sql.adaptive.advisoryPartitionSizeInBytes`控制（默认64MB）。

**案例**：某电商日志分析任务初始设置`spark.sql.shuffle.partitions=2000`，AQE根据实际数据量合并为420个分区，Shuffle耗时从58分钟降至12分钟（提升79%）。

#### 3.2.2 动态调整Join策略

AQE在运行时根据表大小动态选择Join策略：

* 若小表大小<广播阈值（`spark.sql.autoBroadcastJoinThreshold`），自动转为Broadcast Join。
* 若表大小适中，转为Shuffled Hash Join。
* 若表极大，保持Sort Merge Join。

**案例**：某金融Join任务中，静态优化误判小表大小选择Sort Merge Join（耗时2.1小时），AQE检测到小表实际仅1GB，动态转为Broadcast Join，耗时降至18分钟（提升7倍）。

#### 3.2.3 动态优化倾斜Join

AQE自动检测倾斜Key（默认分区大小>中位数5倍且>256MB），将倾斜分区分拆为多个子分区，并行处理。例如，某支付数据中“热门商品”Key占比80%，AQE将其拆分为20个子分区，总耗时从6小时降至1.5小时（提升75%）。

**AQE启用配置**：

```
```
spark.conf.set("spark.sql.adaptive.enabled","true")  
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled","true")// 合并小分区  
spark.conf.set("spark.sql.adaptive.skewJoin.enabled","true")// 倾斜Join优化
```
```

### 3.3 Tungsten引擎：内存与CPU的极致优化

Tungsten是Spark的底层优化引擎，通过**内存管理**和**代码生成**提升性能，尤其对宽依赖Shuffle场景效果显著：

* **堆外内存（Off-Heap）**：通过`sun.misc.Unsafe` API直接操作内存，避免JVM对象开销和GC。例如，Java字符串“abcd”在JVM中占48字节（含对象头），Tungsten二进制存储仅需4字节。
* **向量化执行**：以Batch为单位处理数据（而非单行），利用CPU SIMD指令提升计算效率，TPC-DS基准测试性能提升40%。
* **代码生成（Whole-Stage CodeGen）**：将多个算子逻辑合并为单一Java方法，消除虚函数调用和中间对象，Shuffle聚合吞吐量提升10倍以上。

**Tungsten启用配置**：

```
```
spark.conf.set("spark.memory.offHeap.enabled","true")  
spark.conf.set("spark.memory.offHeap.size","4g")// 堆外内存大小  
spark.conf.set("spark.sql.codegen.wholeStage","true")// 启用全阶段代码生成
```
```

## 四、实际应用中的依赖关系调整与性能调优案例

### 4.1 案例一：电商用户画像计算优化（宽依赖→窄依赖）

**背景**：某电商平台用户画像任务需关联用户行为表（100亿行）与用户标签表（1亿行），原始代码使用`join`算子（宽依赖），Shuffle耗时4.2小时。

**问题分析**：用户标签表虽大但可全量加载到内存，且用户行为表按`user_id`分区，可通过广播+协同分区转为窄依赖。

**优化步骤**：

1. **广播标签表**

   ：`broadcast(userTagsDF)`避免Shuffle。
2. **预分区行为表**

   ：按`user_id`重分区，确保与标签表协同。

**优化后代码**：

```
```
val userTagsDF = spark.read.parquet("user_tags").repartition("user_id")// 预分区  
val userBehaviorDF = spark.read.parquet("user_behavior").repartition("user_id")// 协同分区  
val profileDF = userBehaviorDF.join(broadcast(userTagsDF),"user_id")// 窄依赖Join
```
```

**效果**：任务耗时从4.2小时降至23分钟（提升11倍），Shuffle数据量减少98%。

### 4.2 案例二：金融支付数据倾斜处理（宽依赖倾斜→拆分优化）

**背景**：某银行支付数据聚合任务中，5%的商户占85%交易数据，`groupByKey(merchant_id)`导致单个Task处理20GB数据，耗时6小时。

**问题分析**：数据倾斜导致宽依赖Shuffle中个别Task过载。

**优化步骤**：

1. **检测倾斜Key**

   ：通过`df.groupBy("merchant_id").count().orderBy(desc("count"))`定位倾斜Key。
2. **拆分倾斜数据**

   ：将倾斜Key单独处理，添加随机前缀打散分区。
3. **二次聚合**

   ：先按“Key+随机前缀”聚合，再去掉前缀全局聚合。

**优化后代码**：

```
```
val skewedKeys = List("merchant_001","merchant_002")// 倾斜Key列表  
val saltedDF = df.withColumn("salt", when(col("merchant_id").isin(skewedKeys: _*),  
  concat(col("merchant_id"), lit("_"),(rand *10).cast("int"))// 加盐打散  
).otherwise(col("merchant_id")))  
  
// 二次聚合  
val resultDF = saltedDF.groupBy("salt").agg(sum("amount").alias("sum_amount"))  
.withColumn("merchant_id", split(col("salt"),"_").getItem(0))  
.groupBy("merchant_id").agg(sum("sum_amount").alias("total_amount"))
```
```

**效果**：单个Task数据量从20GB降至2GB，总耗时从6小时降至1.5小时（提升75%）。

### 4.3 案例三：机器学习特征工程优化（窄依赖流水线）

**背景**：某推荐系统特征工程需对1亿用户样本执行“清洗→特征提取→归一化”三步转换，原始代码分三次触发Action，导致重复计算。

**问题分析**：未充分利用窄依赖的流水线特性，多次Action触发多次DAG执行。

**优化步骤**：

1. **合并转换算子**

   ：将窄依赖操作串联，形成流水线。
2. **持久化中间结果**

   ：对复用的RDD使用`persist`缓存至内存。

**优化后代码**：

```
```
val featureRDD = rawDataRDD  
.filter(_.isValid)// 清洗（窄依赖）  
.map(extractFeatures)// 特征提取（窄依赖）  
.map(normalize)// 归一化（窄依赖）  
.persist(StorageLevel.MEMORY_AND_DISK)// 缓存中间结果  
  
// 单次Action触发所有转换  
val trainData = featureRDD.filter(_.label.isDefined)  
val testData = featureRDD.filter(_.label.isEmpty)
```
```

**效果**：计算次数从3次降至1次，总耗时从90分钟降至25分钟（提升72%）。

RDD依赖关系与DAG优化是Spark性能调优的核心，其本质是通过**减少数据移动**和**提升并行效率**实现计算加速。窄依赖的流水线执行和宽依赖的Shuffle优化构成了Spark任务调度的基础，而Spark 3.x的AQE和Tungsten引擎进一步降低了调优门槛，实现了“静态配置+动态自适应”的双重优化。

## 附录：关键配置参数参考

| 配置项 | 作用 | 推荐值 |
| --- | --- | --- |
| `spark.default.parallelism` | RDD默认并行度 | `Executor数 × 核心数 × 2` |
| `spark.sql.shuffle.partitions` | SQL Shuffle分区数 | 400~1000（根据数据量调整） |
| `spark.sql.adaptive.enabled` | 启用AQE | `true` |
| `spark.sql.autoBroadcastJoinThreshold` | 广播Join阈值 | 100MB（大内存集群可设200MB） |
| `spark.memory.offHeap.enabled` | 启用堆外内存 | `true` |
| `spark.shuffle.file.buffer` | Shuffle写缓冲区 | 64KB→256KB（减少磁盘I/O） |
| `spark.reducer.maxSizeInFlight` | Shuffle读缓冲区 | 48MB→96MB（减少网络请求） |

通过合理配置这些参数，并结合依赖关系调整策略，可显著提升Spark作业性能，充分发挥分布式计算的威力。

### 以上。

本次优化总结详细文档已放入知识星球《随川陪你学大数据》，可扫码获取

获取更多信息，关注大数据技能圈

**欢迎添加作者交流**

## 推荐阅读系列文章

* [腾讯面试：Spark内存如何优化？包含哪几个方面？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491167&idx=1&sn=d62e305f6803f484aa40469393b1fdb8&scene=21#wechat_redirect)
* [腾讯面试：请详细描述Paimon如何基于LSM树实现高吞吐写入和高效查询？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491536&idx=1&sn=01cb739322d9fd0e5ef4530e6970187c&scene=21#wechat_redirect)
* [腾讯面试：Flink100G大状态如何优化？有哪些参数可以调整？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491532&idx=1&sn=18c1952668c000328b2fa3081fc80906&scene=21#wechat_redirect)
* [阿里面试：Paimon QPS太低怎么优化？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491526&idx=1&sn=d6364e30ac6442c02c7a147337c4d44a&scene=21#wechat_redirect)
* [腾讯面试：Flink出现反压如何排查？有哪些参数可以调整？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491520&idx=1&sn=b8ab27aa17c78653d6598faff0bc85e9&scene=21#wechat_redirect)
* [超强总结：Iceberg可以优化的方方面面](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491505&idx=1&sn=e385a35b8d736b2cc639225b8487563c&scene=21#wechat_redirect)
* [阿里面试：Hudi，Iceberg，Paimon之间的差异有哪些？该如何选择？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491499&idx=1&sn=dc6844323e421ff97157ffe6de5b9bbb&scene=21#wechat_redirect)
* [阿里面试：如果让你负责大数据平台的架构，需要考虑哪些点？如何设计？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491478&idx=1&sn=baf72d7547630119ff4da6c0ba6b1239&scene=21#wechat_redirect)
* [阿里面试：请详细解释一下Flink内存管理，具体有哪些参数可以调整？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491468&idx=1&sn=524bcce4dc664fe78d4442daa6ab21f6&scene=21#wechat_redirect)
* [腾讯面试：介绍一下Doris问题排查思路，有没有总结过相关文档？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491462&idx=1&sn=24b0982030c010dd904b65de15bd17df&scene=21#wechat_redirect)
* [这篇文章把Paimon和Fluss的关系给彻底说清楚了](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491452&idx=1&sn=f6e98294672db035964f347a290f16f4&scene=21#wechat_redirect)
* [腾讯面试：Doris 物化视图的使用场景是怎么样的，有哪几种数据更新方式？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491386&idx=1&sn=d29b14d6912d0efdb941ae5e1dbd1dfb&scene=21#wechat_redirect)
* [腾讯面试：详细介绍Spark的Shuffle阶段数据从输入到输出经历了哪些步骤？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491384&idx=1&sn=5ab414f60fda52a3b9b88e1a7f2cdecd&scene=21#wechat_redirect)
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