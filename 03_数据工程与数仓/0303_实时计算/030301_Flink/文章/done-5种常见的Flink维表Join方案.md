> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkJoin技术全景|FlinkJoin技术全景]]、[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkLookupJoin维表关联实践|FlinkLookupJoin维表关联实践]]
---
title: 5种常见的Flink维表Join方案
author: 大数据实践录
date:
url: http://mp.weixin.qq.com/s?__biz=MzU1ODAyNDY2Nw==&mid=2247484420&idx=1&sn=fce85def21e50c6667c036778595c759&chksm=fc2d96b0cb5a1fa62c74e16c37504a7602e434da2f642bfcc40001bfa286be7960a4c89fb905&mpshare=1&scene=24&srcid=0115To4OYjpkFBlMiGiJd32Y&sharer_shareinfo=bd26174401a4b0126ecde28b55a9f179&sharer_shareinfo_first=bd26174401a4b0126ecde28b55a9f179#rd
---

> 一般来说维表分为静态维表和变化维表。对于静态维表比较好处理，初始化一次加载到内存即可。但现实情况一般维表是不定期更新的，那如何 JOIN 一张不断变化的表呢？所以本文就目前几种常见的维表join方案做一个总结。

# 写在前面

这里总结一下几种维表join的思路，主要包括以下几种：

① 热存储关联

② 预加载维表

③ 广播维表

④ Temporal Table Join

⑤ Lookup Table Join

## 热存储关联

查找关联就是在主数据流中直接访问外部数据(mysql，redis，presto ...)根据关键条件去关联。

### 1. 使用同步join

访问数据库时同步调用，这会导致子线程会被阻塞，影响吞吐量。

### 2. 使用异步join

既然同步方式会造成资源阻塞，所以为了提升吞吐量，我们可以通过异步 I / O ，可以并发处理多个请求，很大程度上减少了对子线程的阻塞。具体使用方法可以参考 Flink 的 AsyncDataStream 使用说明。

### 3. 使用缓存

不管是同步还是异步访问数据库，都会对外部数据库造成一定的压力。这时我们不妨考虑一下使用缓存来减轻外部存储的压力，比如 guava Cache 或者 Redis 等。

## 预加载维表

通过定义一个类实现RichMapFunction，在open()中读取维表数据加载到内存中，在主数据流map()方法中与维表数据进行关联。

RichMapFunction中open方法里加载维表数据到内存的方式特点如下：

**优点**：实现简单

**缺点**：因为数据存于内存，所以只适合小数据量并且维表数据更新频率不高的情况下。虽然可以在open中定义一个定时器定时更新维表，但是还是存在维表更新不及时的情况。

## 广播维表

利用Flink的Broadcast State将维度数据流广播到下游做join操作。

**优点**：维度数据变更后可以即时更新到结果中。这种场景要求维表数据存在于实时流中。

**缺点**：数据保存在内存中，支持的维度数据量比较小。

## Temporal Table Join

时态表，是Flink针对维表关联的一个解决方案。Temporal table是持续变化表上某一时刻的视图，Temporal table function是一个表函数，传递一个时间参数，返回Temporal table这一指定时刻的视图。

**优点**：维度数据量可以很大，维度数据更新及时，要求维表数据存在于实时流中。 **缺点**：只支持在Flink SQL API中使用。

## Lookup Table Join

lookup join 其实简单理解来，就是每来一条数据去数据库中搂一次数据。然后把关联到的维度数据给拼接到当前数据中。

JDBC 连接器可以用在时态表关联中作为一个可 lookup 的 source (又称为维表)。用到的语法是 Temporal Joins 的语法。

**优点**：使用简单。

**缺点**：只支持在Flink SQL API中使用，且flink官方目前只支持Jdbc和Hbase的维表join。

# 写在最后

选择哪种Join方案，取决于业务实际需求以及数据量的大小。当然了，如果有更好的想法，欢迎联系我一起探讨，在代码实现过程中有任何问题也可以随时联系作者。之后也会整理一份代码版的分享出来。

往期推荐

[大数据学习指南（第七节）：Kafka技术（上）](http://mp.weixin.qq.com/s?__biz=MzU1ODAyNDY2Nw==&mid=2247484415&idx=1&sn=66c31f1857c789ce70ec28864ee8f3b3&chksm=fc2d914bcb5a185d181605276b8bc83937dd8cd3e711e287c7ca969d82c125cb3268d68766f3&scene=21#wechat_redirect)

[大数据学习指南（第六节）：Zookeeper技术](http://mp.weixin.qq.com/s?__biz=MzU1ODAyNDY2Nw==&mid=2247484406&idx=1&sn=3083b391b94e09ca2815ea50228460b9&chksm=fc2d9142cb5a185478f418e6bd7531b3870e3182bce39f17d6f34d10f58f7ec3dafb23d01e99&scene=21#wechat_redirect)

[大数据学习指南（第五节）：Hadoop技术之YARN（续）](http://mp.weixin.qq.com/s?__biz=MzU1ODAyNDY2Nw==&mid=2247484363&idx=1&sn=85fbfe4ec552a659f014d0f1fd5ecdd6&chksm=fc2d917fcb5a1869158af57c992682f84fc11e8649aa9c9b85a1b8e2421dc10bc226958dbe25&scene=21#wechat_redirect)

[大数据学习指南（第四节）：Hadoop技术之YARN](http://mp.weixin.qq.com/s?__biz=MzU1ODAyNDY2Nw==&mid=2247484349&idx=1&sn=9fc4cc4b5d06caeb713612b692a65a2b&chksm=fc2d9109cb5a181f31ba586a5d43e9eafd18931e90fba88ec0023e97f400b1da6d29acdb8163&scene=21#wechat_redirect)

[大数据学习指南（第三节）：Hadoop技术之HDFS](http://mp.weixin.qq.com/s?__biz=MzU1ODAyNDY2Nw==&mid=2247484337&idx=1&sn=32d514d9247a7fc4c3dca03df9ce60df&chksm=fc2d9105cb5a1813d2c2a3bcfec41a5bff741a2dfd58d4f3eea1c92e66f01051e280b7685d59&scene=21#wechat_redirect)

[大数据学习指南（第二节）：Hadoop介绍及环境搭建](http://mp.weixin.qq.com/s?__biz=MzU1ODAyNDY2Nw==&mid=2247484324&idx=1&sn=d06c3e8c6d227ead9e6253667451bf77&chksm=fc2d9110cb5a1806772ae7951f88725a0157fe75ad61e6d7aa3deb102ceb8b421a5a66209e66&scene=21#wechat_redirect)

[大数据学习指南（第一节）：大数据概论](http://mp.weixin.qq.com/s?__biz=MzU1ODAyNDY2Nw==&mid=2247484317&idx=1&sn=a69f9ba92077d74a78892055c64bceb7&chksm=fc2d9129cb5a183f261b8a0bb204106882fb9351bed28b3908a0758c14e74d60a5760a22e49d&scene=21#wechat_redirect)