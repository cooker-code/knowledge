> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL聚合去重与膨胀治理|SQL聚合去重与膨胀治理]]
---
title: SQL成神之路｜求求你别用with cube了
author: 胡说大数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzkyMDMxMzY0MQ==&mid=2247484481&idx=1&sn=a0b3890324adebad6980911cb15ba727&chksm=c00c59ac977a43ad2659537b0809d03c2fc20b51eb772d92f255ff0930a8c52d61e86accb6c2&mpshare=1&scene=24&srcid=1104V7NBPbRuhcrbW2v1bwKi&sharer_shareinfo=ee6aae621270d99bc24b55c42d77625b&sharer_shareinfo_first=ee6aae621270d99bc24b55c42d77625b#rd
---

# SQL成神之路｜求求你别用with cube了

> CUBE 是一种多维聚合操作，用于生成所有可能维度组合的聚合结果。它扩展了 GROUP BY 的功能，`允许用户在多个维度上进行数据聚合，并生成每个维度组合的汇总数据`。CUBE 操作特别适用于需要查看所有可能维度组合的汇总数据的场景，本文讲下CUBE操作的原理及实践经验。

## 1. 背景

在实际生产中针对大数据场景的报表数据，对应BI系统无法支持相对细粒度的数据，我们一般会直接加工结果，这里就会用到CUBE操作。而如果关注的维度较多时，`数据加工任务大概率会出现性能问题`。

## 2. CUBE的原理

### a.with cube

假如维度字段为n个，with cube操作会生成`2的n次方`的维度组合数，数据会膨胀`2的n次方`倍去计算

```
select
  a
  ,b
  ,count(1) as cnt
from table
group by 
  a
  ,b
with cube
```

从下面执行计划可以看出，共有a和b两个维度属性字段，`[List(a#22L, b#23L, 0), List(a#22L, null, 1), List(null, b#23L, 2), List(null, null, 3)]`，总共数据膨胀了4倍。

```
== Physical Plan ==
*CollectLimit 100000
+- *(1) HashAggregate(keys=[a#24L, b#25L, spark_grouping_id#21], functions=[count(1)], output=[a#24L, b#25L, count(1)#20L])
   +- *Exchange hashpartitioning(a#24L, b#25L, spark_grouping_id#21, 3000)
      +- *(1) HashAggregate(keys=[a#24L, b#25L, spark_grouping_id#21], functions=[partial_count(1)], output=[a#24L, b#25L, spark_grouping_id#21, count#49L])
         +- *(1) Expand [List(a#22L, b#23L, 0), List(a#22L, null, 1), List(null, b#23L, 2), List(null, null, 3)], [a#24L, b#25L, spark_grouping_id#21]
            +- *(1) Project [a#1L AS a#22L, b#2L AS b#23L]
               +- *(1) FileScan parquet table[a#1L,b#2L] 
```

### b.grouping sets

with cube会暴力的生成所有维度组合，`假如关注10个维度属性，如果维度全组合计算数据，数据会膨胀2的10次方1024倍，极容易出现计算性能问题`。而往往日常看数并不会所有维度组合都看，`grouping sets让开发者可以自定义维度组合，免去不必要的计算。所有推荐大家使用grouping sets`。

```
select
  a
  ,b
  ,count(1) as cnt
from table
group by 
  a
  ,b
with cube

--等价于

select
  a
  ,b
  ,count(1) as cnt
from table
group by 
  a
  ,b
grouping sets(
  ()  --不区分维度的组合
  ,a
  ,b
  ,(a,b)
)
```

## 3. 总结

日常工作中，对于小数据量的场景，维度随意交叉其实还好。但是对于大数据量的场景，每加一种维度组合都会引起数据的翻倍，而业务方不可能所有维度组合都看，日常一定要做好管控。

**私人微信👇👇👇**

****欢迎加入星球，获取海量数据资料👇👇👇****