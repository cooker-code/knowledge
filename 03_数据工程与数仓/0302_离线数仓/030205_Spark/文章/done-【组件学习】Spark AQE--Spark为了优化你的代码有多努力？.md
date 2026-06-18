> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_核心知识点/SparkAQE与执行计划诊断|SparkAQE与执行计划诊断]]
---
title: 【组件学习】Spark AQE--Spark为了优化你的代码有多努力？
author: 傻文和靓燕
date:
url: https://mp.weixin.qq.com/s?__biz=MzAwOTkwNzI4Ng==&mid=2247483858&idx=1&sn=7d045baa55ce83ca80bf7f03a0b0d0d1&chksm=9a070e5a1fcbe3e8abf38762792751ed9e11df83f9e5f92008691da014cc96e6539a891116a5&mpshare=1&scene=24&srcid=0808vz6ZVKwa50hh7Q26EcUn&sharer_shareinfo=91aa27e9cb16b54029f3955d5ac4396f&sharer_shareinfo_first=91aa27e9cb16b54029f3955d5ac4396f#rd
---

本篇文章内容来自极客时间-Spark性能调优实践
想学习Spark的内部优化AQE（自适应查询执行），首先要知道Spark SQL转化为执行任务的过程。因为Spark的相关优化规则和策略是在这个过程中被转化为执行计划的。
然后再去学习AQE的触发实际，优化内容等。

### Q&A：Spark SQL的优化过程：

一段Spark SQL转化为机器内部执行的作业，需要经历以下步骤：

SQL解析→分析→逻辑优化→物理计划生成、优化→字节码生成，转化为DAG→执行

### Q&A：AQE是什么？

本质是Spark3.0版本之后内部提供的普适性的优化手段、措施，目的是让Spark作业运行起来更节省资源，更高效。作用于逻辑优化和物理优化期间。

### Q&A：AQE优化出现之前，Spark SQL的优化措施是什么？

RBO：基于规则的优化，比如谓词下推，列剪枝等
CBO：基于成本的优化，基于数据表的统计信息来选择优化策略，但是适用面较窄，仅支持注册到hive元数据的数据表，并且需要调用ANALYZE TABLE COMPUTE STATISTICS 语句收集统计信息，而各类信息的收集会消耗大量时间。
这两种优化措施都是静态的，一旦按照这些优化措施制定好了执行计划，后续便一直执行到作业结束。即使中间在运行中出现了明显的，可以优化的点，也无法回头重新修改计划。

### Q&A： AQE的特点是什么？

动态的，作业运行时执行计划可修改的。
完整的说就是AQE是Spark SQL的一种动态优化机制，在运行时，每当shuffle Map阶段执行完毕，AQE都会结合这个阶段的统计信息，基于既定的规则动态的调整、修正尚未执行的逻辑计划和物理计划，来完成对原始查询语句运行时的优化。
AQE通过Shuffle Map阶段结束后统计基于shuffle中间文件的数据（中间文件中是由索引文件的，索引文件中包含每个data文件的大小，空文件数量与占比等信息）。

### Q&A：AQE可以做哪些优化？

AQE优化规则和策略主要有四个，一个逻辑优化规则（作用于逻辑优化阶段）和三个物理优化策略（作用域物理优化阶段）。

优化特性有三个：
1、大表关联小表（join策略调整）；
2、运行过程中小分区过多（自动分区合并）；
3、数据倾斜问题（自动倾斜处理）。

```
spark.sql.adaptive.enabled    是否启用AQE，默认禁用
```

### Join策略调整

Join策略调整涉及到AQE的一个逻辑优化规则（DemoteBroadcastHashJoin）和一个物理优化策略（OptimizeLocalShuffleReader）。
简单说就是AQE通过Shuffle Map的中间文件，判断存在一个合格的小表，那么就可以触发逻辑优化规则，将该小表通过广播的方式分发到各个Executor。优化触发节点是在Shuffle Map的数据落盘之后。
以上都是DemoteBroadcastHashJoin优化规则的起到的作用，将Shuffle Join（必须是sort merge join）降级为Broadcast Join。

常规的Spark shuffle过程中，reduce阶段的计算需要跨节点访问中间文件拉取数据分片。当前优化策略中，虽然AQE在运行时将shuffle sort metge join降级为了Broadcast Join，但是大表的中间文件还是需要通过网络分发到不同的reduce节点，这样的化，当前这个Join优化策略就是没啥实际作用的，因为大表实际还是要做完整shuffle流程，再转为Broadcast Join已经没有意义。
Join优化策略中的OptimizeLocalShuffleReader物理优化策略此时就起到了作用，它可以让大表中间数据存在节点的reduce task读取本地的大表中间文件，避免大表中间文件进行网络传输，大大降低了资源消耗。
整个过程看下来，个人感觉AQE的Join优化策略有用，但是不是100%的Broadcast Join。与其这样，我更愿意在代码开发过程中使用广播join的hint，强制Broadcast Join（当然要有十足把握），这样才是100%的Broadcast Join。
但是作为一个普适性的优化措施，还是不错的。

涉及到的参数：

```
spark.sql.autoBroadcastJoinThreshold    中间文件尺寸中和小于广播阈值
spark.sql.adaptive.nonEmptyPartitionRatioForBroadcastJoin 空文件占比小于配置项
spark.sql.adaptive.localShuffleReader.enabled OptimizeLocalShuffleReader物理策略是否生效，默认为true。
```

### 自动分区合并

Reduce阶段Reduce task从全网把数据分片拉回，AQE按照分区编号的顺序，依次将小于目标表尺寸的分区合并在一起。
shuffle map阶段完成后，AQE自动触发，并将CoalesceShufflePartitions策略无条件的添加到新的物理计划中。
涉及到的参数：

```
spark.sql.adaptive.coalescePartitions.enabled，是否启用AQE中自动分区合并，默认开启
spark.sql.adaptive.advisoryPartitionSizeInBytes，由开发者指定分区合并后的推荐尺寸
spark.sql.adaptive.coalescePartitions.minPartitionNum，分区合并后，分区数不能低于该值
```

### 自动倾斜处理

与自动分区合并相反，自动倾斜处理是将大分区拆分成小分区。
在Reduce阶段，当Reduce task所需处理的分区尺寸大于一定阈值的事后，利用OptimizeSkewedJoin策略，AQE会把大分区拆分为多个小分区。
拆分操作是在reduce阶段进行的，同一个Executor中，本该由一个task处理的大分区，被AQE拆分成多个分区由多个task完成，这样task的计算负载得到了均衡。

但是，这只是在Executor中得到了均衡，Executor之间还是老样子，比如2T的数据，分发到100个Executor中处理，其中A-Executor分配了900G，其他99个Executor平均分了100G，那么经过自动倾斜处理，A-Executor内部task之间得到了负载均衡，但是100个Executor之间还是A-Executor承担了更多。
并且在Join场景中，A表进行了数据拆分，一个分区拆分成了两个，B表对应的分区需要进行复制，保证关联关系不被破坏。
如果两个表都出现了倾斜呢？事情就变得复杂起来了。

总的来说，如果场景比较简单，比如有倾斜，但是数据分布相对均匀，或者关联中只有一方倾斜，可以使用AQE的自动倾斜处理机制。但是，如果数据中key分布非常悬殊，或者关联的两个表都存在倾斜，就要谨慎使用该优化机制了。

涉及到的参数：

```
spark.sql.adaptive.skewJoin.enabled，是否启用AQE中自动分区合并，默认开启
spark.sql.adaptive.skewJoin.skewedPartitionFactor，判定倾斜的膨胀系数
spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes，判定倾斜的最低阈值
spark.sql.adaptive.advisoryPartitionSizeInBytes，以字节为单位，定义拆分粒度
```

### 后序

Spark AQE不是万能的，请合理使用Spark AQE，不要把它当作万金油。
下一篇文章，会讲解（搬运）每一个优化项中涉及到的参数的作用。