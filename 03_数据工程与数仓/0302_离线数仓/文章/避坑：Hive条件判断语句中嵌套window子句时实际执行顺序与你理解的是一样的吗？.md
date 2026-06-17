---
title: 避坑：Hive条件判断语句中嵌套window子句时实际执行顺序与你理解的是一样的吗？
author: 会飞的一十六
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247485019&idx=1&sn=07cca4d12fc21e5510a6debe93bcccf3&chksm=e94161e01d80f2787cfe127ef798f44d1ca4871b8559a3a16692218ac9f4dc41526a3662b7d2&mpshare=1&scene=24&srcid=0906lkosmHQsT11xE9QE7LB7&sharer_shareinfo=e87623429de059dae4b353a40a59804d&sharer_shareinfo_first=e87623429de059dae4b353a40a59804d#rd
---

**“**

本文通过一个实际需求分析，探讨了在Hive SQL中，当条件判断语句（如CASE WHEN）嵌套在窗口函数（Window Function）中时，可能出现的错误结果和执行顺序问题。文章详细解释了错误的原因，并提供了两种解决方案，强调了条件判断在窗口函数执行顺序上的重要性。同时，总结了此类问题的关键知识点，提醒读者注意条件判断与窗口函数的正确使用方式。

**”**

01

—

需求描述

需求：表如下

以上数据中，goods\_type列，假设26代表是广告，现在有个需求，想获取每个用户每次搜索下非广告类型的商品位置自然排序，如果下效果：

02

—

数据准备

```
create table window_goods_test (user_id int,    --用户idgoods_name string,  --商品名称goods_type int, --标识每个商品的类型，比如广告，非广告rk int  --这次搜索下商品的位置，比如第一个广告商品就是1，后面的依次2，3，4...)ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';  
1  hadoop  10  11  hive  12  21  sqoop  26  31  hbase  10  41  spark  13  51  flink  26  61  kafka  14  71  oozie  10  8  
load data local inpath "/home/centos/dan_test/window_goods_test.txt" into table window_goods_test;
```

03

—

问题分析

从结果表来看只需要增加一列为排序列，只不过是将goods\_type列去除掉重新排序，因此我们很容易想到用窗口函数解决排序问题.row\_number()便可顺利解决。于是很容易写出如下SQL：

```
select     user_id,    goods_name,    goods_type,    rk,    case when goods_type!=26 then row_number() over(partition by user_id  order by rk) else null end as naturl_rank  from window_goods_test
```

```
1  hadoop  10  1  11  hive  12  2  21  sqoop  26  3  NULL1  hbase  10  4  41  spark  13  5  51  flink  26  6  NULL1  kafka  14  7  71  oozie  10  8  8Time taken: 2.858 seconds, Fetched 8 row(s)
```

从结果可以看出并非自然排序，不是我们最终想要的目标结果，从实现上看逻辑也清除没什么问题，问题出哪了呢？其原因在于对窗口函数的执行原理及顺序不了解。下面我们进一步通过执行计划来看此SQL的执行过程。SQL如下：

```
explain select     user_id,    goods_name,    goods_type,    rk,    case when goods_type!=26 then row_number() over(partition by user_id  order by rk) else null end as naturl_rank  from window_goods_test
```

具体执行计划如下：

```
== Physical Plan ==*Project [user_id#67, goods_name#68, goods_type#69, rk#70, CASE WHEN NOT (goods_type#69 = 26) THEN _we0#72 ELSE null END AS naturl_rank#64]+- Window [row_number() windowspecdefinition(user_id#67, rk#70 ASC NULLS FIRST, ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS _we0#72], [user_id#67], [rk#70 ASC NULLS FIRST]   +- *Sort [user_id#67 ASC NULLS FIRST, rk#70 ASC NULLS FIRST], false, 0      +- Exchange hashpartitioning(user_id#67, 200)         +- HiveTableScan [user_id#67, goods_name#68, goods_type#69, rk#70], MetastoreRelation default, window_goods_testTime taken: 0.238 seconds, Fetched 1 row(s)21/06/25 13:06:24 INFO CliDriver: Time taken: 0.238 seconds, Fetched 1 row(s)
```

具体执行步骤如下：

（1）扫描表

（2）按照use\_id分组

（3）按照user\_id和rk进行升序排序

（4）执行row\_number()函数进行分析

  (5) 使用case when进行判断

由执行计划可以看出case when是在窗口函数之后执行的，与我们同常理解先进行条件判断，满足条件后再执行窗口函数的认知是有差异的。

也就是说case when中使用窗口函数时候，先执行窗口函数再执行条件判断。

上述代码中数据运转流程图如下：

实际上上述SQL被拆分成三部分执行：

第一步：扫描表，获取select的结果集

```
select     user_id,    goods_name,    goods_type,    rkfrom window_goods_test
```

```
21/06/25 13:49:49 INFO DAGScheduler: Job 9 finished: processCmd at CliDriver.java:376, took 0.039341 s1  hadoop  10  11  hive  12  21  sqoop  26  31  hbase  10  41  spark  13  51  flink  26  61  kafka  14  71  oozie  10  8Time taken: 0.153 seconds, Fetched 8 row(s)21/06/25 13:49:49 INFO CliDriver: Time taken: 0.153 seconds, Fetched 8 row(s)
```

第二步：执行窗口函数

```
select     user_id,    goods_name,    goods_type,    rk,   row_number() over(partition by user_id order by rk) as naturl_rank  from window_goods_test
```

```
21/06/25 13:47:33 INFO DAGScheduler: Job 8 finished: processCmd at CliDriver.java:376, took 0.754508 s1  hadoop  10  1  11  hive  12  2  21  sqoop  26  3  31  hbase  10  4  41  spark  13  5  51  flink  26  6  61  kafka  14  7  71  oozie  10  8  8Time taken: 1.016 seconds, Fetched 8 row(s)21/06/25 13:47:33 INFO CliDriver: Time taken: 1.016 seconds, Fetched 8 row(s)
```

 第三步：在第二步的结果集中执行case when

```
select     user_id,    goods_name,    goods_type,    rk,    case when goods_type!=26 then row_number() over(partition by user_id  order by rk) else null end as naturl_rank  from window_goods_test
```

 其变换结果集如下：

由以上分析我们我们可以看出要想得到正确的SQL需要在窗口函数执行前就需要将数据先过滤掉而不是窗口函数执行后。因此可以想到在where语句里面先过滤，但是根据结果商品类型为26的排序需要置为NULL，因此我门采用union,具体SQL如下：

```
select user_id  ,goods_name  ,goods_type  ,rk  ,row_number() over(partition by user_id  order by rk) as naturl_rank from window_goods_testwhere goods_type!=26union allselect user_id  ,goods_name  ,goods_type  ,rk  ,null as naturl_rank from window_goods_testwhere goods_type=26
```

```
1  hadoop  10  1  11  hive  12  2  21  hbase  10  4  31  spark  13  5  41  kafka  14  7  51  oozie  10  8  61  sqoop  26  3  NULL1  flink  26  6  NULLTime taken: 1.482 seconds, Fetched 8 row(s)21/06/25 14:08:14 INFO CliDriver: Time taken: 1.482 seconds, Fetched 8 row(s)
```

但此处的缺点是需要扫描表 window\_goods\_test两次，显然对于此题不是最好的解法.

下面我们给出其他解法。为了能够先过滤掉商品类型为26的商品，我们可以先在partition by分组中先进行if 语句过滤，如果goods\_type!=26则取对应的id进行分组排序，如果goods\_type=26则置为随机数再按照随机数分组排序，最后外层再通过goods\_type!=26将其过滤掉。此处partition by分组中if 语句中的else后不置为NULL而是随机数，是因为如果置为NULL，goods\_type=26的数较多的情况下会被分到一组造成数据倾斜，因此采用了rand()函数。具体SQL如下：

```
select user_id  ,goods_name  ,goods_type  ,rk  ,if(goods_type!=26,row_number() over(partition by if(goods_type!=26,user_id,rand()) order by rk),null) naturl_rank from window_goods_testorder by rk------------------------------
```

此处为了得到最终的结果对rk进行了order by排序，order by执行是在窗口函数之后。

获取的中间结果如下：

```
select user_id  ,goods_name  ,goods_type  ,rk  ,row_number() over(partition by if(goods_type!=26,user_id,rand()) order by rk) naturl_rank from window_goods_testorder by rk--------------------------------- 1  hadoop  10  1  11  hive  12  2  21  sqoop  26  3  11  hbase  10  4  31  spark  13  5  41  flink  26  6  11  kafka  14  7  51  oozie  10  8  6Time taken: 1.069 seconds, Fetched 8 row(s) == Physical Plan ==*Sort [rk#217 ASC NULLS FIRST], true, 0+- Exchange rangepartitioning(rk#217 ASC NULLS FIRST, 200)   +- *Project [user_id#214, goods_name#215, goods_type#216, rk#217, naturl_rank#211]      +- Window [row_number() windowspecdefinition(_w0#219, rk#217 ASC NULLS FIRST, ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS naturl_rank#211], [_w0#219], [rk#217 ASC NULLS FIRST]         +- *Sort [_w0#219 ASC NULLS FIRST, rk#217 ASC NULLS FIRST], false, 0            +- Exchange hashpartitioning(_w0#219, 200)               +- *Project [user_id#214, goods_name#215, goods_type#216, rk#217, if (NOT (goods_type#216 = 26)) cast(user_id#214 as double) else rand(-4528861892788372701) AS _w0#219]                  +- HiveTableScan [user_id#214, goods_name#215, goods_type#216, rk#217], MetastoreRelation default, window_goods_testTime taken: 0.187 seconds, Fetched 1 row(s)21/06/25 14:37:27 INFO CliDriver: Time taken: 0.187 seconds, Fetched 1 row(s)
```

最终执行结果如下：

```
1  hadoop  10  1  11  hive  12  2  21  sqoop  26  3  NULL1  hbase  10  4  31  spark  13  5  41  flink  26  6  NULL1  kafka  14  7  51  oozie  10  8  6Time taken: 1.255 seconds, Fetched 8 row(s)21/06/25 14:14:21 INFO CliDriver: Time taken: 1.255 seconds, Fetched 8 row(s)  explain select user_id  ,goods_name  ,goods_type  ,rk  ,if(goods_type!=26,row_number() over(partition by if(goods_type!=26,user_id,rand()) order by rk),null) naturl_rank from window_goods_testorder by rk == Physical Plan ==*Sort [rk#227 ASC NULLS FIRST], true, 0+- Exchange rangepartitioning(rk#227 ASC NULLS FIRST, 200)   +- *Project [user_id#224, goods_name#225, goods_type#226, rk#227, if (NOT (goods_type#226 = 26)) _we0#230 else null AS naturl_rank#221]      +- Window [row_number() windowspecdefinition(_w0#229, rk#227 ASC NULLS FIRST, ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS _we0#230], [_w0#229], [rk#227 ASC NULLS FIRST]         +- *Sort [_w0#229 ASC NULLS FIRST, rk#227 ASC NULLS FIRST], false, 0            +- Exchange hashpartitioning(_w0#229, 200)               +- *Project [user_id#224, goods_name#225, goods_type#226, rk#227, if (NOT (goods_type#226 = 26)) cast(user_id#224 as double) else rand(1495282467312192326) AS _w0#229]                  +- HiveTableScan [user_id#224, goods_name#225, goods_type#226, rk#227], MetastoreRelation default, window_goods_testTime taken: 0.186 seconds, Fetched 1 row(s)21/06/25 14:39:55 INFO CliDriver: Time taken: 0.186 seconds, Fetched 1 row(s)
```

04

—

小结

此题给我们的启示：

* （1）case when(或if)语句中嵌套窗口函数时，条件判断语句的执行顺序在窗口函数之后
* （2）窗口函数partition by子句中可以嵌套条件判断语句

往期精彩

[SQL进阶技巧：如何分区间或分时段统计数据？| 等距分桶问题](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484711&idx=1&sn=479f57e4e2c64324544f35dfeabd69bf&chksm=e8e21107df959811e91633fca516d5a435dfd0fd70c2e222b325625f55d6b6e1b612ccaa6b48&scene=21#wechat_redirect)

[SQL进阶技巧：如何将数值数据扩充完整？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484557&idx=1&sn=77cd222a5511bd15c902a693e06af591&chksm=e8e210addf9599bbd426d3b055891079838acb019ad5617cf6b789287fea6775d5eff6f4a808&scene=21#wechat_redirect)

[SQL进阶技巧：如何不使用union all进行行转列？【三种方法实现】](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484545&idx=1&sn=83179a23f1ffea78a17066e91394a66b&chksm=e8e210a1df9599b7fcdec2d96ee3fc0f657921dde2a2ae06a2d7fec395eaf41a804ac6637bfa&scene=21#wechat_redirect)