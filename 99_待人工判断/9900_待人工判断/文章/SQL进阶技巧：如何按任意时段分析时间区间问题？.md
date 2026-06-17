---
title: SQL进阶技巧：如何按任意时段分析时间区间问题？
author: 会飞的一十六
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484761&idx=1&sn=487f8abe2156db46aa47464f3456b896&chksm=e9bf3ac54e0efa8fdcac167f3fb718d5456b0a4db344e625f97034c4a2f0e7b93aec0cf14784&mpshare=1&scene=24&srcid=0828PFHtV5rqlSCzCl6Zt4mW&sharer_shareinfo=bce3ca34f5e6562da4c37f37a3c573cb&sharer_shareinfo_first=bce3ca34f5e6562da4c37f37a3c573cb#rd
---

01

—

场景描述

现有用户还款计划表 user\_repayment ，该表内的一条数据，表示用户在指定日期区间内 [date\_start, date\_end] ，每天还款 repayment 元。

如何统计任意时段内（如：2024-01-15至2024-01-16）每天所有用户的应还款总额？

02

—

 数据准备

```
with user_repayment as (    select stack(        3,        '101', '2024-01-01', '2024-01-15', 10,        '102', '2024-01-05', '2024-01-20', 20,        '103', '2024-01-10', '2024-01-25', 30    )     -- 字段：用户，开始日期，结束日期，每日还款金额    as (user_id, date_start, date_end, repayment))select * from user_repayment;
```

03

—

问题分析

我们将需求转换为时序图，如下图所示：

时间点对应下图离散的点，图中标注了每个用户对应的起始点和结束点

通过上图可以看出，需求时间段对应的为15，和16两个点，15处此时有三个用户还款，因此当天所有用户还款总额为10+20+30 = 60,16点处当天有2个用户还款，还款应为20+30=50.上述的分析我们通过SQL怎么实现呢？

### 方法1：分情况讨论，找出重叠区间

我们先来了解一下，判断一个区间是否与已知的区间存在包含（重叠）关系的几种情况。

假设 已知的区间起点为st,结束点为et，当前需要判断的起点为ct,结束点为cet，如下图所示：

**情况1：区间在右**

```
判断条件 cet >= et and ct <= et          重叠区间为【ct,et]】
```

**情况2：区间在内**

```
判断条件为 ct>= st  and cet <= et      重叠区间为 【ct,cet】
```

**情况3：区间在左**

```
判断条件 ct <= st  and cet >= st        重叠区间为【st,cet】
```

基于上述情况的讨论，我们用SQL如下：

**步骤1：计算重叠区间**

注意需要需要判断区间2个断点重合的情况

```
select user_id     , date_start     , date_end     , repayment     , case           when '2024-01-15' <= date_end and '2024-01-16' >= date_end then               if('2024-01-15' = date_end, date_end, concat_ws(',', '2024-01-15', date_end))           when '2024-01-15' >= date_start and '2024-01-16' <= date_end               then concat_ws(',', '2024-01-15', '2024-01-16')           when '2024-01-15' <= date_start and '2024-01-16' >= date_start then               if('2024-01-15' = date_start, date_start, concat_ws('-,', date_start, '2024-01-16'))    end overlap_datefrom user_repayment
```

**步骤2：将重叠区间展开**

利用如下方法展开

```
lateral view explode(split(overlap_date, ','))
```

SQL如下：

```
select user_id      , date_start      , date_end      , repayment      , dt from (select user_id            , date_start            , date_end            , repayment            , case                  when '2024-01-15' <= date_end and '2024-01-16' > date_end then                      if('2024-01-15' = date_end, date_end, concat_ws(',', '2024-01-15', date_end))                  when '2024-01-15' >= date_start and '2024-01-16' <= date_end                      then concat_ws(',', '2024-01-15', '2024-01-16')                  when '2024-01-15' < date_start and '2024-01-16' >= date_start then                      if('2024-01-15' = date_start, date_start, concat_ws('-,', date_start, '2024-01-16'))         end overlap_date       from user_repayment) t          lateral view explode(split(overlap_date, ',')) tmp as dt
```

**步骤3：按照dt进行分组，计算每天所有用户的还款金额**

```
select dt     , sum(repayment) repaymentfrom (select user_id           , date_start           , date_end           , repayment           , case                 when '2024-01-15' <= date_end and '2024-01-16' > date_end then                     if('2024-01-15' = date_end, date_end, concat_ws(',', '2024-01-15', date_end))                 when '2024-01-15' >= date_start and '2024-01-16' <= date_end                     then concat_ws(',', '2024-01-15', '2024-01-16')                 when '2024-01-15' < date_start and '2024-01-16' >= date_start then                     if('2024-01-15' = date_start, date_start, concat_ws('-,', date_start, '2024-01-16'))        end overlap_date      from user_repayment) t         lateral view explode(split(overlap_date, ',')) tmp as dtgroup by dt;
```

### 方法2：暴力美学法。按区间展开成日期明细表

步骤1：区间日期展开，形成明细数据

```
select date_add(date_start, pos) as dt,        repaymentfrom user_repayment         lateral view posexplode(split(space(datediff(date_end, date_start)), '(?!&)')) t as pos, val;
```

步骤2：基于展开的明细数据，进行输入日期的过滤筛选，并分组求和

```
select dt     , sum(repayment) as repaymentfrom (select date_add(date_start, pos) as dt,             repayment      from user_repayment               lateral view posexplode(split(space(datediff(date_end, date_start)), '(?!&)')) t as pos, val) twhere dt >= '2024-01-15'  and dt <= '2024-01-16'group by dtorder by dt;
```

04

—

拓展案例

场景描述

```
商品活动表 goods_event，g_id（有可能重复），t1（开始时间）,t2（结束时间）  
给定时间段（t3,t4)，求在时间段内做活动的商品数
```

问题分析

利用前面分情况讨论总结的结论，直接给出答案

```
select count(distinct g_id) as event_goods_numfrom goods_eventwhere (t1<=t4 and t1>=t3) --区间在左or   (t1  <= t3 and t2>=t4) --区间在内or (t2>=t3 and t2<=t4) --区间在右
```

05

—

小结

本文分析了一种按任意时段计算时间区间问题的方法和思路，其本质是区间重叠问题的应用，文中给出了两种方法，一种按照区间重叠的方式进行分情况讨论划分区间，找出重叠区间，并计算问题，一种按照区间展开的形式形成全量的明细表，基于明细表求解任意给定的时段统计问题。