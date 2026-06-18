> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_核心知识点/SparkSQL业务逻辑优化方法|SparkSQL业务逻辑优化方法]]
---
title: SparkSQL中产生笛卡尔积的几种典型场景以及处理策略
author: 智海观潮
date:
url: https://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484625&idx=1&sn=a01492275033dfd05d288f0ff1f8346c&chksm=e8d17b03e6ef1206a4f1d365b611b0f6ab87c3a1e10ef2ee903c121abdefe94212e33c5bb3d0&mpshare=1&scene=24&srcid=0612VGewJEBOm3qIDP3DLH1k&sharer_shareinfo=1e29125536ac55d526779b54130ca459&sharer_shareinfo_first=1e29125536ac55d526779b54130ca459#rd
---

【前言：如果你经常使用Spark SQL进行数据的处理分析，那么对笛卡尔积的危害性一定不陌生，比如大量占用集群资源导致其他任务无法正常执行，甚至导致节点宕机。那么都有哪些情况会产生笛卡尔积，以及如何事前"预测"写的SQL会产生笛卡尔积从而避免呢？（以下不考虑业务需求确实需要笛卡尔积的场景）】

**Spark SQL几种产生笛卡尔积的典型场景**

---

首先来看一下在Spark SQL中产生笛卡尔积的几种典型SQL：

1. join语句中不指定on条件

```
select * from test_partition1 join test_partition2;
```

2. join语句中指定不等值连接

```
select * from test_partition1 t1 inner join test_partition2 t2 on t1.name <> t2.name;
```

3. join语句on中用or指定连接条件

```
select * from test_partition1 t1 join test_partition2 t2 on t1.id = t2.id or t1.name = t2.name;
```

4. join语句on中用||指定连接条件

```
select * from test_partition1 t1 join test_partition2 t2 on t1.id = t2.id || t1.name = t2.name;
```

除了上述举的几个典型例子，实际业务开发中产生笛卡尔积的原因多种多样。

同时需要注意，在一些SQL中即使满足了上述4种规则之一也不一定产生笛卡尔积。比如，对于join语句中指定不等值连接条件的下述SQL不会产生笛卡尔积:

```
--在Spark SQL内部优化过程中针对join策略的选择，最终会通过SortMergeJoin进行处理。select * from test_partition1 t1 join test_partition2 t2 on t1.id = t2.id and t1.name<>t2.name;
```

此外，对于直接在SQL中使用cross join的方式，也不一定产生笛卡尔积。比如下述SQL：

```
-- Spark SQL内部优化过程中选择了SortMergeJoin方式进行处理select * from test_partition1 t1 cross  join test_partition2 t2 on t1.id = t2.id;
```

但是如果cross join没有指定on条件同样会产生笛卡尔积。

那么如何判断一个SQL是否产生了笛卡尔积呢？

**Spark SQL是否产生了笛卡尔积**

---

以join语句不指定on条件产生笛卡尔积的SQL为例:

```
-- test_partition1和test_partition2是Hive分区表select * from test_partition1 join test_partition2;
```

通过Spark UI上SQL一栏查看上述SQL执行图，如下：

可以看出，因为该join语句中没有指定on连接查询条件，导致了CartesianProduct即笛卡尔积。

再来看一下该join语句的逻辑计划和物理计划：

```
== Parsed Logical Plan =='GlobalLimit 1000+- 'LocalLimit 1000   +- 'Project [*]      +- 'UnresolvedRelation `t`
== Analyzed Logical Plan ==id: string, name: string, dt: string, id: string, name: string, dt: stringGlobalLimit 1000+- LocalLimit 1000   +- Project [id#84, name#85, dt#86, id#87, name#88, dt#89]      +- SubqueryAlias `t`         +- Project [id#84, name#85, dt#86, id#87, name#88, dt#89]            +- Join Inner               :- SubqueryAlias `default`.`test_partition1`               :  +- HiveTableRelation `default`.`test_partition1`, org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe, [id#84, name#85], [dt#86]               +- SubqueryAlias `default`.`test_partition2`                  +- HiveTableRelation `default`.`test_partition2`, org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe, [id#87, name#88], [dt#89]
== Optimized Logical Plan ==GlobalLimit 1000+- LocalLimit 1000   +- Join Inner      :- HiveTableRelation `default`.`test_partition1`, org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe, [id#84, name#85], [dt#86]      +- HiveTableRelation `default`.`test_partition2`, org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe, [id#87, name#88], [dt#89]
== Physical Plan ==CollectLimit 1000+- CartesianProduct   :- Scan hive default.test_partition1 [id#84, name#85, dt#86], HiveTableRelation `default`.`test_partition1`, org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe, [id#84, name#85], [dt#86]   +- Scan hive default.test_partition2 [id#87, name#88, dt#89], HiveTableRelation `default`.`test_partition2`, org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe, [id#87, name#88], [dt#89]
```

通过逻辑计划到物理计划，以及最终的物理计划选择CartesianProduct，可以分析得出该SQL最终确实产生了笛卡尔积。

**Spark SQL中产生笛卡尔积的处理策略**

---

在之前的文章中[《Spark SQL如何选择join策略》](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484354&idx=1&sn=389aab2b4f4339df08d6997b7d33f479&chksm=e976fff8de0176eea1e5aaa765392b5471f8eec9a67fd834483fb585ff4ed284cf05cc7435bb&scene=21#wechat_redirect)已经介绍过，Spark SQL中主要有ExtractEquiJoinKeys（Broadcast Hash Join、Shuffle Hash Join、Sort Merge Join，这3种是我们比较熟知的Spark SQL join）和Without joining keys（CartesianProduct、BroadcastNestedLoopJoin）join策略。

那么，如何判断SQL是否产生了笛卡尔积就迎刃而解。

1. 在利用Spark SQL执行SQL任务时，通过查看SQL的执行图来分析是否产生了笛卡尔积。如果产生笛卡尔积，则将任务杀死，进行任务优化避免笛卡尔积。【不推荐。用户需要到Spark UI上查看执行图，并且需要对Spark UI界面功能等要了解，需要一定的专业性。（注意：这里之所以这样说，是因为Spark SQL是计算引擎，面向的用户角色不同，用户不一定对Spark本身了解透彻，但熟悉SQL。对于做平台的小伙伴儿，想必深有感触）】
2. 分析Spark SQL的逻辑计划和物理计划，通过程序解析计划推断SQL最终是否选择了笛卡尔积执行策略。如果是，及时提示风险。

   具体可以参考Spark SQL join策略选择的源码:

   ```
   def apply(plan: LogicalPlan): Seq[SparkPlan] = plan match {// --- BroadcastHashJoin --------------------------------------------------------------------// broadcast hints were specifiedcase ExtractEquiJoinKeys(joinType, leftKeys, rightKeys, condition, left, right)if canBroadcastByHints(joinType, left, right) =>        val buildSide = broadcastSideByHints(joinType, left, right)Seq(joins.BroadcastHashJoinExec(          leftKeys, rightKeys, joinType, buildSide, condition, planLater(left), planLater(right)))// broadcast hints were not specified, so need to infer it from size and configuration.case ExtractEquiJoinKeys(joinType, leftKeys, rightKeys, condition, left, right)if canBroadcastBySizes(joinType, left, right) =>        val buildSide = broadcastSideBySizes(joinType, left, right)Seq(joins.BroadcastHashJoinExec(          leftKeys, rightKeys, joinType, buildSide, condition, planLater(left), planLater(right)))// --- ShuffledHashJoin ---------------------------------------------------------------------case ExtractEquiJoinKeys(joinType, leftKeys, rightKeys, condition, left, right)if !conf.preferSortMergeJoin && canBuildRight(joinType) && canBuildLocalHashMap(right)           && muchSmaller(right, left) ||           !RowOrdering.isOrderable(leftKeys) =>Seq(joins.ShuffledHashJoinExec(          leftKeys, rightKeys, joinType, BuildRight, condition, planLater(left), planLater(right)))case ExtractEquiJoinKeys(joinType, leftKeys, rightKeys, condition, left, right)if !conf.preferSortMergeJoin && canBuildLeft(joinType) && canBuildLocalHashMap(left)           && muchSmaller(left, right) ||           !RowOrdering.isOrderable(leftKeys) =>Seq(joins.ShuffledHashJoinExec(          leftKeys, rightKeys, joinType, BuildLeft, condition, planLater(left), planLater(right)))// --- SortMergeJoin ------------------------------------------------------------case ExtractEquiJoinKeys(joinType, leftKeys, rightKeys, condition, left, right)if RowOrdering.isOrderable(leftKeys) =>        joins.SortMergeJoinExec(          leftKeys, rightKeys, joinType, condition, planLater(left), planLater(right)) :: Nil// --- Without joining keys ------------------------------------------------------------// Pick BroadcastNestedLoopJoin if one side could be broadcastcase j @ logical.Join(left, right, joinType, condition)if canBroadcastByHints(joinType, left, right) =>        val buildSide = broadcastSideByHints(joinType, left, right)        joins.BroadcastNestedLoopJoinExec(          planLater(left), planLater(right), buildSide, joinType, condition) :: Nilcase j @ logical.Join(left, right, joinType, condition)if canBroadcastBySizes(joinType, left, right) =>        val buildSide = broadcastSideBySizes(joinType, left, right)        joins.BroadcastNestedLoopJoinExec(          planLater(left), planLater(right), buildSide, joinType, condition) :: Nil// Pick CartesianProduct for InnerJoincase logical.Join(left, right, _: InnerLike, condition) =>        joins.CartesianProductExec(planLater(left), planLater(right), condition) :: Nilcase logical.Join(left, right, joinType, condition) =>        val buildSide = broadcastSide(left.stats.hints.broadcast, right.stats.hints.broadcast, left, right)// This join could be very slow or OOM        joins.BroadcastNestedLoopJoinExec(          planLater(left), planLater(right), buildSide, joinType, condition) :: Nil// --- Cases where this strategy does not apply ---------------------------------------------case _ => Nil    }
   ```

‍

此外，在业务开发中，要不断总结归纳产生笛卡尔积的情况，形成知识文档，以便在后续业务开发中避免类似的情况出现。

除了笛卡尔积效率比较低，BroadcastNestedLoopJoin效率也相对低效，尤其是当数据量大的时候还很容易造成driver端的OOM，这种情况也是需要极力避免的。

推荐文章：

[Spark SQL中Not in Subquery为何低效以及如何规避](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484376&idx=1&sn=111c979b86cae4e1c0d1d64315d779c9&chksm=e976ffe2de0176f4d696598dbbe7249f7b63a98ed8bccb9c3d4c76cf3dd6bbcc786e06df65bd&scene=21#wechat_redirect)

[Spark SQL如何选择join策略](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484354&idx=1&sn=389aab2b4f4339df08d6997b7d33f479&chksm=e976fff8de0176eea1e5aaa765392b5471f8eec9a67fd834483fb585ff4ed284cf05cc7435bb&scene=21#wechat_redirect)

[SparkSQL与Hive metastore Parquet转换](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484333&idx=1&sn=a47a4c52b22e8ee9224c5ae6d7231eef&chksm=e976ff97de017681810f13b919887e3db3cee3684f6408f26bd693fe80b65493d9a93d481296&scene=21#wechat_redirect)

[Spark SQL 小文件问题处理](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484206&idx=1&sn=f2b3c0a720e13bc315f478550e5f2406&chksm=e976ff14de017602377983a455305fa312ae62ff6a3cebf3b47c8a6e5e853cdd827a6a102f75&scene=21#wechat_redirect)

[Spark SQL解析查询parquet格式Hive表获取分区字段和查询条件](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247483970&idx=1&sn=99077360915202740f7170cc6fb4a9a1&chksm=e976fe78de01776ea5e830eb150f0e5bfa261137440315902bcdae99f51f0b694566b420a86f&scene=21#wechat_redirect)

[Spark SQL | 目前Spark社区最活跃的组件之一](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247483909&idx=1&sn=768cf5bfe4eaf7b0b57b69b9f2aa2401&chksm=e976fe3fde017729766abd9ebd86503e047489e4f114e3f95d9bd4872787d9a8f210085e96cc&scene=21#wechat_redirect)