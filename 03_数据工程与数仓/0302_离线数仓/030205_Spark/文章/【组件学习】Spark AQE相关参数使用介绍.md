---
title: 【组件学习】Spark AQE相关参数使用介绍
author: 傻文和靓燕
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAwOTkwNzI4Ng==&mid=2247483863&idx=1&sn=5153287957639a5f8ae4cd3f51895a21&chksm=9aa214566b8c6d393e306e17f0b8b37362aea4cd0df2821b3af6fb023d881db08bd5489c123c&mpshare=1&scene=24&srcid=0807FCTtAcPPqZFmxR2buCmv&sharer_shareinfo=0ee801a843cf337420b634f26ef691d0&sharer_shareinfo_first=0ee801a843cf337420b634f26ef691d0#rd
---

本文为Spark AQE相关参数如何使用的内容。

Spark AQE开关配置参数。

```
spark.sql.adaptive.enabled=true 是否启用Spark AQE
```

### 特性一、Join策略调整

```
spark.sql.autoBroadcastJoinThreshold 静态配置：中间文件尺寸中小于广播阈值  
spark.sql.adaptive.autoBroadcastJoinThreshold 动态配置项，默认值同spark.sql.autoBroadcastJoinThreshold  
spark.sql.adaptive.nonEmptyPartitionRatioForBroadcastJoin  该配置项为内部配置项，Spark AQE内部可以处理该  
spark.sql.adaptive.localShuffleReader.enabled  默认为true
```

spark.sql.autoBroadcastJoinThreshold 参数在AQE发布之前，就可以使用。开发者可以通过配置这个参数，对数据关联操作进行主动降级。这个参数默认值是10MB，参与Join的两张表只要有一张数据表尺寸小于10MB，二者的关联操作就可以降级为Broadcast Join。

这个参数虽然好用，但是有两个短板：  
1、可靠性差，尽管小表数据量在阈值之内，但是Spark对小表尺寸可能发生误判，导致降级失败；  
2、预先设置的广播阈值是一种静态的优化机制，没办法在运行时动态对数据关联进行降级调整。比如两张表都不满足参数配置的阈值，但是运行过程中，有一个表通过过滤等操作数据量降低到了阈值之内，完全满足Broadcast join，但是由于是静态优化机制，无法降级。

Spark AQE可动态广播 Join 的阈值（spark.sql.adaptive.autoBroadcastJoinThreshold），运行时根据 Shuffle Map 阶段输出的实际数据量判断是否降级为 Broadcast Join。  
localShuffleReader.enabled 参数启用后（默认开启），当 Join 降级为Broadcast Join 时，允许 Reduce Task 本地读取大表的 Shuffle 中间文件，避免网络传输。

### 特性二、自动分区合并

```
spark.sql.adaptive.coalescePartitions.enabled  是否启用AQE中自动分区合并，默认开启  
spark.sql.adaptive.advisoryPartitionSizeInBytes 由开发者指定分区合并后的推荐尺寸  
spark.sql.adaptive.coalescePartitions.minPartitionNum  分区合并后，分区数不能低于该值
```

AQE的自动分区合并，是按照分区编号，从左到右的进行扫描，边扫描边记录分区尺寸，当相邻的尺寸之和大于目标尺寸是，AQE就把这些扫描过的分区进行合并，然后继续扫描，采用同样的算法，按照目标尺寸合并剩余分区，知道所有分区都处理完成。

**那么AQE怎么判断目标尺寸？通过参数配置，开发这个设置目标尺寸。**

advisoryPartitionSizeInBytes为开发者建议的目标尺寸（这个参数在自动数据倾斜处理中也会被用到），并不是最终的目标尺寸；minPartitionNum合并之后的最小分区数，如果配置成200，说明合并之后的分区数量不能小于200，这个参数的目的是避免并行度过低导致CPU资源利用不充分。

举个例子，如果shuffle之后的数据是20G，配置的minPartitionNum最小分区数是200，那么每个分区的目标尺寸就是20GB/200=100MB，如果advisoryPartitionSizeInBytes设置的是200MB，那么最终的目标分区大小就是取(100M,200M)中的最小值，也就是100M。

所以，自动分区合并中目标分区的大小是由以上两个参数决定的。

### 特性三、自动数据倾斜处理

```
spark.sql.adaptive.skewJoin.enabled 是否启用AQE中自动分区合并，默认开启  
spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes 判定倾斜的最低阈值  
spark.sql.adaptive.skewJoin.skewedPartitionFactor   判定倾斜的膨胀系数  
spark.sql.adaptive.advisoryPartitionSizeInBytes  以字节为单位，定义拆分粒度
```

自动数据倾斜处理中，Spark AQE是检测倾斜的数据分区，然后对其进行拆分操作，把大分区拆分成小分区，从而避免单个任务的数据量过大。

**AQE如何判断检测到倾斜的数据分区？又按照什么样的策略对数据进行拆分，拆分成多大？**

1、分区尺寸必须大于skewedPartitionThresholdInBytes参数的设定值，才有可能被判定为倾斜分区；  
2、AQE统计所有数据分区大小然后排序，取中位数作为放大的基数，大于这个基数一定倍数的分区才会被判定为倾斜分区，这个放大倍数由skewedPartitionFactor参数设置；  
3、接下来就是对倾斜分区进行拆分，AQE会参考advisoryPartitionSizeInBytes参数控制拆分的粒度，比如倾斜分区大小为512MB，advisoryPartitionSizeInBytes为256M，那么倾斜分区就会按照Math.max((80+100)/2,256)=256M的粒度进行拆分，拆分为两个小分区。AQE并不是完全按照advisoryPartitionSizeInBytes参数进行拆分，而是通过一定的算法得到最终拆分粒度。

例如A表有三个分区，80M、100M、512M，这三个分区的中位数是100M，skewedPartitionFactor设定的倍数为5，那么512M的分区刚好比100M大5倍，符合这个条件。skewedPartitionThresholdInBytes参数默认值是256M，也符合条件。如果skewedPartitionThresholdInBytes参数修改为1G，那么即使比中位数大5倍，也无法被判定为倾斜分区。  
被判定为倾斜分区之后，就要进行拆分，advisoryPartitionSizeInBytes被设置为200M，那么512M的分区就会被拆分成3个分区，200M、200M、112M，拆分之后，总体数据由原来的3个分区被拆分为5个小分区。

### 完整示范

```
-- 开启AQE（必须）  
SET spark.sql.adaptive.enabled=true;  
  
-- Join策略调整  
set spark.sql.adaptive.autoBroadcastJoinThreshold=20971520; -- 20MB  
  
-- 分区合并配置  
spark.sql.adaptive.coalescePartitions.enabled =true;  
SET spark.sql.adaptive.advisoryPartitionSizeInBytes=134217728; -- 128MB  
SET spark.sql.adaptive.coalescePartitions.minPartitionNum=100;   
  
-- 数据倾斜处理  
SET spark.sql.adaptive.skewJoin.enabled=true;  
SET spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes=268435456; -- 256MB  
SET spark.sql.adaptive.skewJoin.skewedPartitionFactor=3; 
```

### 后记

1、关于nonEmptyPartitionRatioForBroadcastJoin有一些疑问，从网上给出的信息看，有矛盾之处，无法判断这个参数的作用？为什么非空文件必须小于该值的原因是什么？

2、有网友提到，“DemoteBroadcastHashJoin 规则的作用并不是将 SMJ转换成 BHJ，而是在空文件较多的情况下，避免将 SMJ转换成 BHJ，防止性能回退。因为这种情况，普通的shuffle hash join更快。  
在spark3.3以后，更多规则和算子也改名字了。比如 DemoteBroadcastHashJoin 改成了 DynamicJoinSelection”需要探查该言论是否正确。