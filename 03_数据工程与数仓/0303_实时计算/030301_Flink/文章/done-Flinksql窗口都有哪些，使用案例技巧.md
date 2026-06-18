> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink窗口高级特性|Flink窗口高级特性]]
---
title: Flinksql窗口都有哪些，使用案例技巧
author: 大模型数据的朋友
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzMjAxNDA5OQ==&mid=2247484742&idx=1&sn=df4b291f5c89d598545b9b5598774fc3&chksm=fbaea1417bd70dabdc5c357d6a42d6aee6bfc7ac1c02482fb150f9137592e3d65c8d7f58dff0&mpshare=1&scene=24&srcid=0807V6VtvxAuTLKzjkhBXEz1&sharer_shareinfo=8152a79ac0e19c5b3f1dddffb2422661&sharer_shareinfo_first=8152a79ac0e19c5b3f1dddffb2422661#rd
---

⽬前Flink SQL中有两种类型的窗⼝，即Group window和Over window。

Over window是传统数据库中的开窗，每个元素对应⼀个窗⼝；Group window基于时间窗⼝对数据进⾏划分，再将窗⼝内的数据做聚合或其他处理。传统数据库或者Batch SQL中想要做数据聚合，可以直接使⽤Group by即可。但Streaming SQL由于有⼀些流计算的独有概念，使⽤Group by做聚合得到的是不断更新的结果。

Group window

滑动窗⼝

数据进⼊滑动窗⼝后会被分配给多个固定⻓度的窗⼝，窗⼝的⼤⼩可以通过参数进⾏配置，同时可以通过参数控制滑动参数的滑动距离，在滑动距离⼩于窗⼝⼤⼩的情况下，数据会重复。

语法：

* HOP(time\_attr, interval\_1, interval\_2)
* interval\_1: window slide interval;
* interval\_2: window size interval;

滚动窗⼝

滚动窗⼝有特定的⼤⼩，窗⼝内的数据不会重叠。

语法：TUMBLE(time\_attr, interval)

会话窗⼝

会话窗⼝通过session的活跃度来划分窗⼝，窗⼝不会重叠，也没有特定的起⽌时间，当⼀个会话窗⼝在⼀定的时间间隔内没有收到元素时就会关闭。

语法：SESSION(time\_attr, interval)

我们看个案例，假设需要计算某⽹站的PV，现有以下三种场景：

1. 每5分钟统计近10分钟某⽹站的PV；
2. 统计每10分钟某⽹站的PV；
3. 统计某⽹站连续的PV，连续两次访问间隔不⼤于1分钟。

上面三个场景对应着Flink SQL的三个Group window。假设source表和sink表如下：

```
CREATE TABLE source (`timestamp` timestamp(3), -- 此处timestamp的原始格式为STRING，此处隐式转换为timestamp格式，括号内的3代表的是精度-- timestamp(3)格式的标准输⼊为：2020-11-24 12:00:05.000-- 如果输⼊类型是BIGINT，可以使⽤TOTIMESTAMP(XX) function进⾏转换webName VARCHAR,ip VARCHAR,rowtime as `timestamp`,WATERMARK FOR rowtime as rowtime - INTERVAL '60' SECOND) with ('connector.properties.galaxy.talos.service.endpoint'='xx','connector.secret-key'='XXXXXXX','connector.topic'='XXXXX','connector.secret-key-id'='XXXXXXXX','connector.type'='xx','connector.startup-mode'='latest-offset');CREATE TABLE sink ( webName VARCHAR, val BIGINT, `start` timestamp(3), `end` timestamp(3)) with ('connector.type'='print','update-mode'='retract');
```

a. 每5分钟统计近10分钟某⽹站的PV

```
insert into sinkselect webName, count(ip), HOP_START(rowtime, INTERVAL '5' MINUTE, INTERVAL '10' MINUTE), HOP_END(rowtime, INTERVAL '5' MINUTE, INTERVAL '10' MINUTE)from sourcegroup by HOP(rowtime, INTERVAL '5' MINUTE, INTERVAL '10' MINUTE), webName;
```

b. 统计每10分钟某⽹站的PV

```
insert into sinkselect  webName,  count(ip),  TUMBLE_START(rowtime, INTERVAL '10' MINUTE),  TUMBLE_END(rowtime, INTERVAL '10' MINUTE)from sourcegroup by TUMBLE(rowtime, INTERVAL '10' MINUTE), webName;
```

c. 统计某⽹站连续访问PV，连续两次访问间隔不⼤于1分钟

```
insert into sinkselect  webName,  count(ip),  SESSION_START(rowtime, INTERVAL '1' MINUTE),  SESSION_END(rowtime, INTERVAL '1' MINUTE)from sourcegroup by SESSION(rowtime, INTERVAL '1' MINUTE), webName;
```

```
有了上⾯的SQL后，我们来造⼀些数据进⾏验证，假设我们的输⼊为：2024-11-24 12:00:05.000, baidu, test12024-11-24 12:00:07.123, baidu, test22024-11-24 12:05:07.124, baidu, test32024-11-24 12:10:06.124, baidu, test4
三个场景对应的输出为：a:(true,baidu,2,2024-11-24T11:55,2024-11-24T12:05)(true,baidu,3,2024-11-24T12:00,2024-11-24T12:10)(true,baidu,2,2024-11-24T12:05,2024-11-24T12:15)(true,baidu,1,2024-11-24T12:10,2024-11-24T12:20)-------------------------------------------------b:(true,baidu,3,2024-11-24T12:00,2024-11-24T12:10)(true,baidu,1,2024-11-24T12:10,2024-11-24T12:20)-------------------------------------------------c:(true,baidu,2,2024-11-24T12:00:05,2024-11-24T12:01:07.123)(true,baidu,1,2024-11-24T12:05:07.124,2024-11-24T12:06:07.124)(true,baidu,1,2024-11-24T12:10:06.124,2024-11-24T12:11:06.124)
```

Over window

Over window与Group window不同的是，Over window中的每⼀个元素都对应1个窗⼝，每来⼀条数据都会进行⼀次窗⼝计算，Over window可以根据数据的行或者时间戳值来确定窗⼝。

Over window既⽀持event-time，也⽀持processing-time。Over窗⼝分为两类，Rows OVER Window和Range OVER Window。

Rows OVER Window

该窗⼝中每个元素都确定⼀个窗⼝，即使两条数据同时到达，他们也属于两个窗⼝。rows over的模式可以向前追溯bounded和unbounded的数据，在此我们仅介绍bounded的情况。Rows OVER Window语法：

```
SELECT  agg1(col1) OVER([PARTITION BY (value_expression1,..., value_expressionN)]ORDER BY timeCol ROWSBETWEEN (UNBOUNDED | rowCount) PRECEDING AND CURRENT ROW) AS colName, ...FROM Tab1;-- value_expression1 分区元素-- timeCol ⽤于排序的时间列-- rowCount 向前追溯的数据条数
```

Range OVER Window

该窗⼝下，具有相同时间戳的数据会被放在同⼀个窗⼝下，该窗⼝也同时⽀持追溯bounded和unbounded数据。在此也仅介绍unbounded情况。Range OVER Window语法：

```
SELECTagg1(col1) OVER([PARTITION BY (value_expression1,..., value_expressionN)]ORDER BY timeColRANGEBETWEEN (UNBOUNDED | timeInterval) PRECEDING AND CURRENT ROW) AS colName,...FROM Tab1;
```

案例：假设我们有⼀张订单表，有着下面两个场景：

a. 实时更新某商品最近5条订单的总价格

b. 实时更新某商品最近2分钟订单的最⼤价格

```
CREATE TABLE source (createTime timestamp(3),--如果输⼊类型是BIGINT，可以使⽤TOTIMESTAMP(XX) function进⾏转换orderId VARCHAR,goodsType VARCHAR,price DOUBLE,rowtime as createTime,WATERMARK FOR rowtime as rowtime - INTERVAL '60' SECOND) with ('connector.type'='kafka',......'update-mode'='append','format.type'='csv');

CREATE TABLE sink_1 (createTime timestamp(3),orderId VARCHAR,goodsType VARCHAR,price DOUBLE,totalPrice DOUBLE) with ('connector.type'='print','update-mode'='retract');
CREATE TABLE sink_2 (createTime timestamp(3),orderId VARCHAR,goodsType VARCHAR,price DOUBLE,maxPrice DOUBLE) with ('connector.type'='print','update-mode'='retract');

a. 实时更新最近5条订单的总价格insert into sink_1select  createTime,  orderId,  goodsType,  price,sum(price) OVER (PARTITION BY goodsType ORDER BY rowtime Rows BETWEEN 4 preceding AND CURRENT ROW) as totalPricefrom source;


b. 实时更新最近2分钟订单的最⼤价格
insert into sink_2select  createTime,  orderId,  goodsType,  price,  max(price) OVER (PARTITION BY goodsType ORDER BY rowtime RANGE BETWEEN INTERVAL '2' MINUTE preceding AND CURRENT ROW) as maxPricefrom source;
```

注意ORDER BY后⾯接的必须是单个时间列，支持event-time或processing-time。

总结：本文介绍了Flinksql的Group window和Over window两类窗口，滑动、滚动、会话窗口以及over窗口，希望对你有帮助。

Hey!

我是小魔仙

--欢迎关注公众号--

90后北漂男孩

现任互联网大厂大数据工程师

想要影响1000+个人变得更好

一起遇见未知的自己

一起人生长跑

成为自己的光

💛

多点一下**在看**多一条小鱼干