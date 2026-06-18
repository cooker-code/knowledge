> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_核心知识点/SparkAQE与执行计划诊断|SparkAQE与执行计划诊断]]
---
title: 【组件学习】Spark AQE-Join优化规则非深入研究
author: 傻文和靓燕
date:
url: https://mp.weixin.qq.com/s?__biz=MzAwOTkwNzI4Ng==&mid=2247483876&idx=1&sn=30c87cc9f28a0b99bea87b3d607a07ce&chksm=9ae50bfb53a81aee4121c03d357640897f99e2cd0066516686926c1063fe3afd08ade37a9f00&mpshare=1&scene=24&srcid=0808HWVx7HlIa3OIEMszcv8f&sharer_shareinfo=4b304aae2baa5ff1197a6443d76a0c13&sharer_shareinfo_first=4b304aae2baa5ff1197a6443d76a0c13#rd
---

重新学习`Spark sql`的`AQE`之后，对`Join`优化规则中的非空分区比例参数产生了疑问，我看的文章中结论是小于非空比例参数才可以广播，但是没有提及是`join`中的哪一个表需要满足非空分区比例和为什么需要满足非空分区比例？以及满足非空分区比例的好处是什么？我问了万（分）能（裂）的`DeepSeek`，给出的结论居然和我看到的文章结论完全相反，这个时候我就开始头大了😰，我到底该信谁？？？
然后就是网上各种查，查到的内容和我原始看的文章内容几乎一样😢，只有知乎上的一篇文章的评论中提到了对`Spark sql`的`AQE`中`DemoteBroadcastHashJoin`规则的解释，并且提到了该规则后续的变化，还暖心的提供了该规则的源码地址，于是我就顺着这个线索一路摸索，遂成此文。
感谢这条评论，截图致敬！

**以下内容如有疑问或者错误，请帮忙指出，谢谢！**

接下来，正文开始。

`Spark 3.3`版本之后，`AQE`中的`Join`优化规则做了调整，引入了`Shuffle Hash Join`。并且原DemoteBroadcastHashJoin 规则改名为`DynamicJoinSelection`  。对应源码地址为：DynamicJoinSelection.scala

> 原Join优化`DemoteBroadcastHashJoin` 中的非空分区比例参数`nonEmptyPartitionRatioForBroadcastJoin`的作用是：
> 如果`Join`的一侧表（没有预设值hint）非空分区比例小于该参数值（比如该表原有100分区，经过过滤之后只剩10个分区，那么非空分区比例为0.1，但是该参数值默认是0.2），则避免将其广播，因为广播一个有很多空分区的表可能不高效，而使用`Shuffle Join`（`Sort Merge Join`）可以跳过这些空分区，从而更快。（该结论与以前文章内容有差异，以该文中结论为准！抱歉！！后续会对以前文章做修改。）

## 代码解析：

`DynamicJoinSelection`源码如下（内含个人注释）：

```
package org.apache.spark.sql.execution.adaptive

import org.apache.spark.MapOutputStatistics
import org.apache.spark.sql.catalyst.optimizer.JoinSelectionHelper
import org.apache.spark.sql.catalyst.planning.ExtractEquiJoinKeys
import org.apache.spark.sql.catalyst.plans.{LeftAnti, LeftOuter, RightOuter}
import org.apache.spark.sql.catalyst.plans.logical.{HintInfo, Join, JoinStrategyHint, LogicalPlan, NO_BROADCAST_HASH, PREFER_SHUFFLE_HASH, SHUFFLE_HASH}
import org.apache.spark.sql.catalyst.rules.Rule
import org.apache.spark.sql.internal.SQLConf

/**
 * This optimization rule includes three join selection:
 *   1. detects a join child that has a high ratio of empty partitions and adds a
 *      NO_BROADCAST_HASH hint to avoid it being broadcast, as shuffle join is faster in this case:
 *      many tasks complete immediately since one join side is empty.
 *   2. detects a join child that every partition size is less than local map threshold and adds a
 *      PREFER_SHUFFLE_HASH hint to encourage being shuffle hash join instead of sort merge join.
 *   3. if a join satisfies both NO_BROADCAST_HASH and PREFER_SHUFFLE_HASH,
 *      then add a SHUFFLE_HASH hint.
 */
object DynamicJoinSelection extends Rule[LogicalPlan] with JoinSelectionHelper {
    //非空分区比例是否小于nonEmptyPartitionRatioForBroadcastJoin（空分区比例参数）参数设定
privatedef hasManyEmptyPartitions(mapStats: MapOutputStatistics): Boolean = {
    val partitionCnt = mapStats.bytesByPartitionId.length
    val nonZeroCnt = mapStats.bytesByPartitionId.count(_ > 0)
    partitionCnt > 0 && nonZeroCnt > 0 &&
      (nonZeroCnt * 1.0 / partitionCnt) < conf.nonEmptyPartitionRatioForBroadcastJoin
  }
//是否优先使用Shuffle Hash Join，判断依据为所有分区尺寸都小于阈值maxShuffledHashJoinLocalMapThreshold
//  并且建议分区尺寸也小于阈值
//  maxShuffledHashJoinLocalMapThreshold：Shuffle Hash Join 的最大分区阈值
privatedef preferShuffledHashJoin(mapStats: MapOutputStatistics): Boolean = {
    val maxShuffledHashJoinLocalMapThreshold =
      conf.getConf(SQLConf.ADAPTIVE_MAX_SHUFFLE_HASH_JOIN_LOCAL_MAP_THRESHOLD)
    val advisoryPartitionSize = conf.getConf(SQLConf.ADVISORY_PARTITION_SIZE_IN_BYTES)
    advisoryPartitionSize <= maxShuffledHashJoinLocalMapThreshold &&
      mapStats.bytesByPartitionId.forall(_ <= maxShuffledHashJoinLocalMapThreshold)
  }
//Join策略选择方法
privatedef selectJoinStrategy(
      join: Join,
      isLeft: Boolean): Option[JoinStrategyHint] = {
    //以Left Join为例，isLeft代表当前节点为左侧表
    val plan = if (isLeft) join.left else join.right
    plan match {
      caseLogicalQueryStage(_, stage: ShuffleQueryStageExec) if stage.isMaterialized
        && stage.mapStats.isDefined =>
                //join的当前节点空分区比例是否小于参数=>manyEmptyInPlan 
        val manyEmptyInPlan = hasManyEmptyPartitions(stage.mapStats.get)
        //当前节点是否可进行广播，Left Join中右侧表可以被广播，左侧表不可以被广播
        //left/right时，当前只有是非保留表，canBroadcastPlan 才会为true
        //canBuildBroadcastLeft方法下文中有提到
        val canBroadcastPlan = (isLeft && canBuildBroadcastLeft(join.joinType)) ||
          (!isLeft && canBuildBroadcastRight(join.joinType))
        //join的另一侧节点表空分区比例是否小于参数=>manyEmptyInOther
        val manyEmptyInOther = (if (isLeft) join.right else join.left) match {
          caseLogicalQueryStage(_, stage: ShuffleQueryStageExec) if stage.isMaterialized
            && stage.mapStats.isDefined => hasManyEmptyPartitions(stage.mapStats.get)
          case _ => false
        }
                //降级广播，也就是是否避免使用广播，Left Join中，只有右表可以被广播
        val demoteBroadcastHash = if (manyEmptyInPlan && canBroadcastPlan) {
            //left join中，右侧表空分区较多，并且可以被广播，才可以进入当前逻辑
          join.joinType match {
            // don't demote BHJ since you cannot short circuit local join if inner (null-filled)
            // side is empty
            //当前为右侧表空分区较多，并且可广播，但是当前是LeftOuter ，结果为false（不降级广播）
            //  也就是继续使用广播
            caseLeftOuter | RightOuter | LeftAnti => false
            //inner join，直接降级广播，不使用广播
            case _ => true
          }
          // 总结：在Left/Right中，被广播表的空分区较多，会保留广播。
          //  inner 中，由于左右两边都可以被广播，那么只要某一边表空分区较多，则不使用广播
        } elseif (manyEmptyInOther && canBroadcastPlan) {
          // for example, LOJ, !isLeft but it's the LHS that has many empty partitions if we
          // proceed with shuffle.  But if we proceed with BHJ, the OptimizeShuffleWithLocalRead
          // will assemble partitions as they were before the shuffle and that may no longer have
          // many empty partitions and thus cannot short-circuit local join
          join.joinType match {
              //当左表空分区较多的时候，可降级广播
            caseLeftOuter | RightOuter | LeftAnti => true
            case _ => false
          }
        } else {
          false
        }
        //当前节点是否适合Shuffle Hash Join
        val preferShuffleHash = preferShuffledHashJoin(stage.mapStats.get)
        //如果可以降级广播，并且适合使用Shuffle Hash Join，则直接降级为Shuffle Hash Join
        if (demoteBroadcastHash && preferShuffleHash) {
          Some(SHUFFLE_HASH)
        } elseif (demoteBroadcastHash) {
        //如果可以降级广播，则建议降级广播，避免无效广播
          Some(NO_BROADCAST_HASH)
        } elseif (preferShuffleHash) {
        //如果适合Shuffle Join，则建议Shuffle Hash Join
          Some(PREFER_SHUFFLE_HASH)
        } else {
          None
        }

      case _ => None
    }
  }

def apply(plan: LogicalPlan): LogicalPlan = plan.transformDown {
    case j @ ExtractEquiJoinKeys(_, _, _, _, _, _, _, hint) =>
      var newHint = hint
      //检查左侧策略，并将得到的hint赋值给newHint
      if (!hint.leftHint.exists(_.strategy.isDefined)) {
        selectJoinStrategy(j, true).foreach { strategy =>
          newHint = newHint.copy(leftHint =
            Some(hint.leftHint.getOrElse(HintInfo()).copy(strategy = Some(strategy))))
        }
      }
      //检查右侧策略，并将得到的hint赋值给newHint
      if (!hint.rightHint.exists(_.strategy.isDefined)) {
        selectJoinStrategy(j, false).foreach { strategy =>
          newHint = newHint.copy(rightHint =
            Some(hint.rightHint.getOrElse(HintInfo()).copy(strategy = Some(strategy))))
        }
      }
      //更新Join节点的hint
      if (newHint.ne(hint)) {
        j.copy(hint = newHint)
      } else {
        j
      }
  }
}
```

`canBuildBroadcastLeft`方法来源于：org.apache.spark.sql.catalyst.optimizer.JoinSelectionHelper

```
  def canBuildBroadcastLeft(joinType: JoinType): Boolean = {
    joinType match {
      case _: InnerLike | RightOuter => true//join类型为inner和right才能将左表广播出去
      case _ => false
    }
  }

def canBuildBroadcastRight(joinType: JoinType): Boolean = {
    joinType match {
      //join类型为inner、left等才能将右表广播出去
      case _: InnerLike | LeftOuter | LeftSingle | LeftSemi | LeftAnti | _: ExistenceJoin => true
      case _ => false
    }
  }
```

## 变动总结

> 名词解释
> - 保留表：`A left join B`中`A`为保留表，`B`为非保留表；`Inner Join`中两个表都可视为保留表。
> - 降级广播：不再使用广播`join`。
> - `BHJ`：`Broadcast Hash Join`，即广播`join`。
> - 空分区较多or空分区较多：在本文中指非空分区比例小于参数`nonEmptyPartitionRatioForBroadcastJoin`。

原`Join`优化策略中，是将`Sort Merge Join`优化为`Broadcast Hash Join`。
`Spark 3.3`之后的`AQE`对`Join`优化策略进行了调整，不再是一刀切的将小表就直接广播出去，并且引入了`Shuffle Hash Join`。
**一句话总结**：如果主表空分区较多，并且存在可广播小表，这个时候可进行降级广播；如果可广播的小表对应分区都很小（满足`Shuffle Hash Join`阈值），这个时候可直接降级为`Shuffle Hash Join`；如果可广播小表对应分区不满足`Shuffle Hash Join`阈值，则可降级为`Sort Merge Join`；如果不推荐降级广播，但是可广播的小表满足`Shuffle Hash Join`阈值，则优先使用`Shuffle Hash Join`。

### 降级广播规则

**`Left/Right Join`类型：**
如果保留表空分区多，非保留表可被广播的情况下，可降级广播；
如果保留表空分区不多，非保留表可被广播，即使非保留表空分区较多，可保持广播。

**`Inner join`类型：**
任一表空分区较多，则可以降级广播。

例如在`A Left Join B`中

* • 如果`A`表空分区多，`B`表经过计算之后是可以广播的小表，这种情况可以**降级广播**；
* • 如果`A`表空分区不多，`B`表经过计算之后是可以广播的小表，即使`B`表空分区较多，这种情况也可保持`BHJ`，不降级广播。

### 是否优先`Shuffle Hash Join`规则：

具体可参考`DynamicJoinSelection`类中的`preferShuffledHashJoin`方法，主要是判断可广播节点的分区是否都小于`Shuffle Hash Join`的分区阈值（`conf.getConf(SQLConf.ADAPTIVE_MAX_SHUFFLE_HASH_JOIN_LOCAL_MAP_THRESHOLD)`），并且推荐分区阈值是否小于`Shuffle Hash Join`的分区阈值，如果都满足，则优先`Shuffle Hash Join`。

### 总结

1. 1. 如果广播可以降级，并且可广播表（小表）的每个分区都小于`Shuffle Hash Join`的阈值，则强制转为`Shuffle Hash Join`；
2. 2. 如果广播可以降级，但是不适合`Shuffle hash join`，则降级广播（最终可能选择`Sort merge join`，因为已经判断`Shuffle Hash Join`不满足）；
3. 3. 如果广播不可以降级，但是可广播表（小表）满足`Shuffle Hash Join`条件，则优先使用`Shuffle Hash Join`（可能会选择`Shuffle Hash Join`，但还需要进行其他条件判断，比如数据量、分区、数据倾斜等）。

`Join`策略修改后，可以注意到，如果保留表空分区较多，则不太建议使用`BHJ`。
并且如果可广播表（小表）的分区都比较小的话，则直接修改为`Shuffle Hash Join`。

## 优化基础

**短路机制（`OptimizeShuffleWithLocalRead`）**

短路机制，`Join`过程中，检测到空分区可直接跳过，不进行计算。
但是`Broadcast Hash Join`会重组分区，破坏空分区的连续性，进而丧失短路优化的好处。

如果保留表（通常为大表）的空分区非常多，也就是说很多分区可以被跳过，不用计算，那么短路机制的好处可以被最大化利用。
如果保留表空分区不是很多的话，还是继续进行`BHJ`为最优选择。