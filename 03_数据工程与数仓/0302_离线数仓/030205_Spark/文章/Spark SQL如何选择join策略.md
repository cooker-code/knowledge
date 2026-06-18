---
title: Spark SQL如何选择join策略
author: 智海观潮
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484354&idx=1&sn=389aab2b4f4339df08d6997b7d33f479&chksm=e8053156a7989cfcd8dbff81c5cd60020eaa402783813181461a3fb8fba27c291bb21b437a69&mpshare=1&scene=24&srcid=0618yWpAmw3D2dkoB0G0N3q5&sharer_shareinfo=4ce4e1f01248615641ce6eef0e5ab7f1&sharer_shareinfo_first=4ce4e1f01248615641ce6eef0e5ab7f1#rd
---

**前言**

众所周知，Catalyst Optimizer是Spark SQL的核心，它主要负责将SQL语句转换成最终的物理执行计划，在一定程度上决定了SQL执行的性能。

Catalyst在由Optimized Logical Plan生成Physical Plan的过程中，会根据：

```
abstract class SparkStrategies extends QueryPlanner[SparkPlan]
```

中的JoinSelection通过一些规则按照顺序进行模式匹配，从而确定join的最终执行策略，并且策略的选择会按照执行效率由高到低的优先级排列。

在了解join策略选择之前，首先看几个先决条件：

**1. build table的选择**

Hash Join的第一步就是根据两表之中较小的那一个构建哈希表，这个小表就叫做build table，大表则称为probe table，因为需要拿小表形成的哈希表来"探测"它。源码如下：

```
/* 左表作为build table的条件，join类型需满足：   1. InnerLike：实现目前包括inner join和cross join   2. RightOuter：right outer join*/      private def canBuildLeft(joinType: JoinType): Boolean = joinType match {  case _: InnerLike | RightOuter => true  case _ => false}  
/* 右表作为build table的条件，join类型需满足（第1种是在业务开发中写的SQL主要适配的）：   1. InnerLike、LeftOuter（left outer join）、LeftSemi（left semi join）、LeftAnti（left anti join）   2. ExistenceJoin：only used in the end of optimizer and physical plans, we will not generate SQL for this join type*/private def canBuildRight(joinType: JoinType): Boolean = joinType match {  case _: InnerLike | LeftOuter | LeftSemi | LeftAnti | _: ExistenceJoin => true  case _ => false}
```

**2. 满足什么条件的表才能被广播**

如果一个表的大小小于或等于参数spark.sql.autoBroadcastJoinThreshold（默认10M）配置的值，那么就可以广播该表。源码如下：

```
private def canBroadcastBySizes(joinType: JoinType, left: LogicalPlan, right: LogicalPlan)  : Boolean = {  val buildLeft = canBuildLeft(joinType) && canBroadcast(left)  val buildRight = canBuildRight(joinType) && canBroadcast(right)  buildLeft || buildRight}  
private def canBroadcast(plan: LogicalPlan): Boolean = {  plan.stats.sizeInBytes >= 0 && plan.stats.sizeInBytes <= conf.autoBroadcastJoinThreshold}  
private def broadcastSideBySizes(joinType: JoinType, left: LogicalPlan, right: LogicalPlan)  : BuildSide = {  val buildLeft = canBuildLeft(joinType) && canBroadcast(left)  val buildRight = canBuildRight(joinType) && canBroadcast(right)    // 最终会调用broadcastSide  broadcastSide(buildLeft, buildRight, left, right)}
```

除了通过上述表的大小满足一定条件之外，我们也可以通过直接在Spark SQL中显示使用hint方式（/\*+ BROADCAST(small\_table) \*/），直接指定要广播的表，源码如下：

```
private def canBroadcastByHints(joinType: JoinType, left: LogicalPlan, right: LogicalPlan)  : Boolean = {  val buildLeft = canBuildLeft(joinType) && left.stats.hints.broadcast  val buildRight = canBuildRight(joinType) && right.stats.hints.broadcast  buildLeft || buildRight}  
private def broadcastSideByHints(joinType: JoinType, left: LogicalPlan, right: LogicalPlan)  : BuildSide = {  val buildLeft = canBuildLeft(joinType) && left.stats.hints.broadcast  val buildRight = canBuildRight(joinType) && right.stats.hints.broadcast    // 最终会调用broadcastSide  broadcastSide(buildLeft, buildRight, left, right)}
```

无论是通过表大小进行广播还是根据是否指定hint进行表广播，最终都会调用broadcastSide，来决定应该广播哪个表：

```
private def broadcastSide(     canBuildLeft: Boolean,     canBuildRight: Boolean,     left: LogicalPlan,     right: LogicalPlan): BuildSide = {  
   def smallerSide =     if (right.stats.sizeInBytes <= left.stats.sizeInBytes) BuildRight else BuildLeft  
  if (canBuildRight && canBuildLeft) {    // 如果左表和右表都能作为build table，则将根据表的统计信息，确定physical size较小的表作为build table（即使两个表都被指定了hint）    smallerSide  } else if (canBuildRight) {     // 上述条件不满足，优先判断右表是否满足build条件，满足则广播右表。否则，接着判断左表是否满足build条件    BuildRight  } else if (canBuildLeft) {    BuildLeft  } else {    // 如果左表和右表都不能作为build table，则将根据表的统计信息，确定physical size较小的表作为build table。目前主要用于broadcast nested loop join    smallerSide  }}
```

从上述源码可知，即使用户指定了广播hint，实际执行时，不一定按照hint的表进行广播。

**3. 是否可构造本地HashMap**

应用于Shuffle Hash Join中，源码如下：

```
// 逻辑计划的单个分区足够小到构建一个hash表// 注意：要求分区数是固定的。如果分区数是动态的，还需满足其他条件private def canBuildLocalHashMap(plan: LogicalPlan): Boolean = {  // 逻辑计划的physical size小于spark.sql.autoBroadcastJoinThreshold * spark.sql.shuffle.partitions（默认200）时，即可构造本地HashMap  plan.stats.sizeInBytes < conf.autoBroadcastJoinThreshold * conf.numShufflePartitions}
```

我们知道，SparkSQL目前主要实现了3种join：Broadcast Hash Join、ShuffledHashJoin、Sort Merge Join。那么Catalyst在处理SQL语句时，是依据什么规则进行join策略选择的呢？

**1. Broadcast Hash Join**

主要根据hint和size进行判断是否满足条件。

```
// broadcast hints were specifiedcase ExtractEquiJoinKeys(joinType, leftKeys, rightKeys, condition, left, right)   if canBroadcastByHints(joinType, left, right) =>   val buildSide = broadcastSideByHints(joinType, left, right)   Seq(joins.BroadcastHashJoinExec(     leftKeys, rightKeys, joinType, buildSide, condition, planLater(left), planLater(right)))  
// broadcast hints were not specified, so need to infer it from size and configuration.case ExtractEquiJoinKeys(joinType, leftKeys, rightKeys, condition, left, right)   if canBroadcastBySizes(joinType, left, right) =>   val buildSide = broadcastSideBySizes(joinType, left, right)   Seq(joins.BroadcastHashJoinExec(     leftKeys, rightKeys, joinType, buildSide, condition, planLater(left), planLater(right)))
```

**2. Shuffle Hash Join**

选择Shuffle Hash Join需要同时满足以下条件：   

1. spark.sql.join.preferSortMergeJoin为false，即Shuffle Hash Join优先于Sort Merge Join
2. 右表或左表是否能够作为build table
3. 是否能构建本地HashMap
4. 以右表为例，它的逻辑计划大小要远小于左表大小（默认3倍）

上述条件优先检查右表。

```
case ExtractEquiJoinKeys(joinType, leftKeys, rightKeys, condition, left, right)    if !conf.preferSortMergeJoin && canBuildRight(joinType) && canBuildLocalHashMap(right)      && muchSmaller(right, left) ||      !RowOrdering.isOrderable(leftKeys) =>   Seq(joins.ShuffledHashJoinExec(     leftKeys, rightKeys, joinType, BuildRight, condition, planLater(left), planLater(right)))  
case ExtractEquiJoinKeys(joinType, leftKeys, rightKeys, condition, left, right)     if !conf.preferSortMergeJoin && canBuildLeft(joinType) && uildLocalHashMap(left)       && muchSmaller(left, right) ||       !RowOrdering.isOrderable(leftKeys) =>    Seq(joins.ShuffledHashJoinExec(      leftKeys, rightKeys, joinType, BuildLeft, condition, planLater(left), planLater(right)))      private def muchSmaller(a: LogicalPlan, b: LogicalPlan): Boolean = {  a.stats.sizeInBytes * 3 <= b.stats.sizeInBytes}
```

如果不满足上述条件，但是如果参与join的表的key无法被排序，即无法使用Sort Merge Join，最终也会选择Shuffle Hash Join。

```
‍‍!RowOrdering.isOrderable(leftKeys)  
def isOrderable(exprs: Seq[Expression]): Boolean = exprs.forall(e => isOrderable(e.dataType))
```

**3. Sort Merge Join**

如果上面两种join策略（Broadcast Hash Join和Shuffle Hash Join）都不符合条件，并且参与join的key是可排序的，就会选择Sort Merge Join。

```
case ExtractEquiJoinKeys(joinType, leftKeys, rightKeys, condition, left, right)   if RowOrdering.isOrderable(leftKeys) =>   joins.SortMergeJoinExec(     leftKeys, rightKeys, joinType, condition, planLater(left), planLater(right)) :: Nil
```

**4. Without joining keys**

Broadcast Hash Join、Shuffle Hash Join和Sort Merge Join都属于经典的ExtractEquiJoinKeys（等值连接条件）。

对于非ExtractEquiJoinKeys，则会优先检查表是否可以被广播（hint或者size）。如果可以，则会使用BroadcastNestedLoopJoin（简称BNLJ），熟悉Nested Loop Join则不难理解BNLJ，主要却别在于BNLJ加上了广播表。

源码如下：

```
‍‍// Pick BroadcastNestedLoopJoin if one side could be broadcastcase j @ logical.Join(left, right, joinType, condition)    if canBroadcastByHints(joinType, left, right) =>  val buildSide = broadcastSideByHints(joinType, left, right)  joins.BroadcastNestedLoopJoinExec(    planLater(left), planLater(right), buildSide, joinType, condition) :: Nil  
case j @ logical.Join(left, right, joinType, condition)    if canBroadcastBySizes(joinType, left, right) =>  val buildSide = broadcastSideBySizes(joinType, left, right)  joins.BroadcastNestedLoopJoinExec(    planLater(left), planLater(right), buildSide, joinType, condition) :: Nil
```

如果表不能被广播，又细分为两种情况： 

1. 若join类型InnerLike（关于InnerLike上面已有介绍）对量表直接进行笛卡尔积处理若
2. 上述情况都不满足，最终方案是选择两个表中physical size较小的表进行广播，join策略仍为BNLJ

源码如下：

```
// Pick CartesianProduct for InnerJoincase logical.Join(left, right, _: InnerLike, condition) =>  joins.CartesianProductExec(planLater(left), planLater(right), condition) :: Nil  
case logical.Join(left, right, joinType, condition) =>  val buildSide = broadcastSide(    left.stats.hints.broadcast, right.stats.hints.broadcast, left, right)  // This join could be very slow or OOM  joins.BroadcastNestedLoopJoinExec(    planLater(left), planLater(right), buildSide, joinType, condition) :: Nil
```

很显然，无论SQL语句最终的join策略选择笛卡尔积还是BNLJ，效率都很低，这一点在实际应用中，要尽量避免。

推荐文章：  
[SparkSQL与Hive metastore Parquet转换](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484333&idx=1&sn=a47a4c52b22e8ee9224c5ae6d7231eef&chksm=e976ff97de017681810f13b919887e3db3cee3684f6408f26bd693fe80b65493d9a93d481296&scene=21#wechat_redirect)  
[通过Spark生成HFile，并以BulkLoad方式将数据导入到HBase](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484317&idx=1&sn=481b1deea59defbf5f4149c0e3fd7286&chksm=e976ffa7de0176b1e12876979ab84241c9fa4dfced058c5b7ae540f46aeeaa64130e3e47b232&scene=21#wechat_redirect)  
[Spark SQL 小文件问题处理](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484206&idx=1&sn=f2b3c0a720e13bc315f478550e5f2406&chksm=e976ff14de017602377983a455305fa312ae62ff6a3cebf3b47c8a6e5e853cdd827a6a102f75&scene=21#wechat_redirect)