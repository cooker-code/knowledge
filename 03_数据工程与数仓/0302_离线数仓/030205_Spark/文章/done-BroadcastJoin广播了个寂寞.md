---
title: BroadcastJoin广播了个寂寞
author: 熊大数据
date:
url: https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486842&idx=1&sn=7ff2c26bae8aa27f472f04d8772bbc0e&chksm=e955640cf38664adc5b257c05a522e34544b177d22e973d350ab8de673a3d39e23e2c7986476&mpshare=1&scene=24&srcid=0909MUNkKS4pN6h0be3YNA4F&sharer_shareinfo=f3c864042cf44f36203b845d503e1af0&sharer_shareinfo_first=f3c864042cf44f36203b845d503e1af0#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_知识地图|知识地图]]


我是大熊！某大厂数据负责人
将公众号设为“星标⭐”，第一时间推送

**1个需求+1个改动=1点下班**

大熊

平时写SQL都习惯了，对于小表<10M

我闭着眼都不用看执行计划

妥妥的Broadcast Join无疑

```
-- 计算有登入的等级用户每日下单数SELECT user_type,            COUNT(order_id) as order_countFROM dwd_login_users a JOIN dwd_order_di b    ON a.user_id = b.user_idWHERE b.dt = '2025-08-25'GROUP BY user_type;
```

提交-运行-走你！

内心默认帮优化器选择了广播

但运行下来

Execute Me，我的广播呢？

避免大量数据跨节点Shuffle，用10MB小表广播到每个Executor

时间复杂度从O(n log n) 降到 O(n)

心法口中念，没道理不成功

百思不得其姐

# ******BroadCast Hash Join = BHJ（简写）******

# **一、广播的前提**

01 等值连接

必须存在等值连接条件

比如：ON t1.id = t2.id

02 仅支持5种连接类型

分别是：Inner Join、Left Outer、Right Outer、Left Semi、Left Anti

从源码来看，Broadcast Join也不支持Full Outer Join

不支持Full Outer Join我理解是为了性能考虑

如果要为未匹配的行生成Null填充

就需要在多个阶段之间传递匹配状态信息

这又会是一笔内存开销和网络传输

# **二、Join策略选择的优先级**

BHJ相比其他Join机制而言，效率更高

但是BHJ属于网络密集型操作

需要在Driver段缓存数据，所以容易出现OOM

# **三、BHJ开启参数**

BHJ开启的静态参数如下：

```
# 静态参数默认10MBspark.sql.autoBroadcastJoinThreshold# 如果是AQE框架中，自适应阈值spark.sql.adaptive.autoBroadcastJoinThreshold
```

自动广播是基于metastore统计信息的**预判**

必须有准确的统计信息（stats.sizeInBytes）

对于Hive表，需要运行

```
 ANALYZE TABLE <tableName> COMPUTE STATISTICS noscan
```

还有因为数据有经过Snappy的压缩，导致数据进入到内存后展开变大

```
# Parquet文件包含详细的统计信息case class ParquetFileStats(  numRows: Long,  compressedSize: Long,  # 解压后的大小  uncompressedSize: Long,    columnStats: Map[String, ColumnStats],  compressionCodec: String)
```

使我们对小表的数据大小判断不准确，这时：

* Spark会先用静态统计信息做初步判断
* 在AQE中，收集实际数据大小
* 如果判断错误，动态跳转Join策略

这就是为什么实际运行时表可能更大（虚胖），也会导致BHJ失效

对于文件数据源

统计信息可以直接从文件计算

# **四、BHJ失效原因**

# **源码位置**

```
org.apache.spark.sql.catalyst.optimizer.JoinSelectionHelperorg.apache.spark.sql.execution.adaptive.DynamicJoinSelection
```

01 基表不能被广播

A Left Join B

左连接时，能且只能广播右表

LJ要求：左表的所有行都必须出现在结果中

左表决定结果集的行数和顺序

如果广播B表（订单表），而订单表是大表

自然没办法广播，这是原因之一！

02 动态Join选择失效

在自适应执行框架中

BroadcastJoin可能被动态降级

当存在大量空分区时

Spark为了避免使用BroadcastJoin

就会添加 NO\_BROADCAST\_HASH 提示

然而 shuffle join 在这种情况下更快，许多任务可以立即完成

源码部分也解释很清楚

引起空分区最主要原因：倾斜

还是最开始的案例：计算有登入的等级用户每日下单数，普通用户的订单肯定占大头

导致数据集中在一个分区，出现大量空分区

03 超过阈值范围

* 表大小超过配置的阈值时
* 统计信息不准确或缺失时
* 运行时发现实际数据量远大于预估时

当然这样还有另一种极端情况

* driver内存不足，无法创建broadcast变量
* executor内存不足时，无法接收broadcast数据

实际基本没遇到过，只不过存在可能性

它广播了个寂寞，我说了个寂寞

---

参考

1.https://medium.com/@kerrache.massipssa/apache-spark-join-strategies-in-depth-171bf7fef4b0

2.https://docherish.com/post/spark-joinselection-ce-lue/

**END**

加群/领资料备注“熊大群/面试题库”

每周原创更新，主要分享大厂数据经验 

你永远百度不到的那种！

**往期精彩文章合集**

【数据建模】

 [深度调研：外卖数仓的建设](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486319&idx=1&sn=899d4638bcff5c6bddc207f24e0b9100&scene=21#wechat_redirect)

 [数仓专家如何进行数据调研？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485454&idx=1&sn=0cb476d43665eea03750ab2562ba9ad9&scene=21#wechat_redirect)

[从业务到数仓-网约车平台Gra建模设计](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485035&idx=1&sn=9ec02657a9d224e8ecbbb94ae5750918&scene=21#wechat_redirect)

【数据性能优化】

 [宇宙厂Flink优化：实时榜单最佳实践](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486294&idx=1&sn=4ab511381ff73b31107ada027281e2bd&scene=21#wechat_redirect)

 [SQL千亿数据膨胀OOM优化经验](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247484805&idx=1&sn=7082c25af5d666a54756007770303ee7&scene=21#wechat_redirect)

 [ABTest数据全链路建设](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486430&idx=1&sn=2355995ba1fa99c4adaeefc4afa2f123&scene=21#wechat_redirect)

 [1个月被叫醒20次，字节的欢迎仪式？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486640&idx=1&sn=a979b3d88c7fe379f35dbf8b659a0965&scene=21#wechat_redirect)

【面试经验】
 [明知我只写SQL，为何面试考动态规划](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485491&idx=1&sn=0e13cf91fc90fd2a58651b973367b689&scene=21#wechat_redirect)
 [宇宙厂数据岗 3-1 初面，等消息...](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486036&idx=1&sn=afc7122b4da23ca0d1ff1ebf97e066eb&scene=21#wechat_redirect)

【工作十年】

[做数开，别把路走窄了](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485682&idx=1&sn=4c77735ce24319509c116cd45417702e&scene=21#wechat_redirect)

[入职阿里，跟不上节奏](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486072&idx=1&sn=227bd9fbc857eebc51f43df75c80e6d4&scene=21#wechat_redirect)

[技术人如何体面离职？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486417&idx=1&sn=aa79f38420aa196fad34b2e4953dca41&scene=21#wechat_redirect)