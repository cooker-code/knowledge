---
title: SQL进阶技巧：如何计算先进先出库龄问题？
author: 会飞的一十六
date:
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484330&idx=1&sn=ee9b76bde4035d1a8cca8d9a803112a0&chksm=e98f1e2ca9b7f64f7a413bbf275e03a36211cf1aeabf3270f26e799fea77af7e2725a5e7b88e&mpshare=1&scene=24&srcid=0919oL4P43CAUtmJs2JwMJUh&sharer_shareinfo=7019187f3a477a461b9cbc16e7f1b138&sharer_shareinfo_first=7019187f3a477a461b9cbc16e7f1b138#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL窗口滚动与序列问题|SQL窗口滚动与序列问题]]


点击上方「蓝字」关注我们

> 本文讲解了先进先出的库龄计算问题，针对库龄计算首先要理解状态量变化的原理，以及能识别出本题问题点所在。解决本题关键点在于对于先对状态量的判断，找出断点，并基于该断点能够重新分组，然后再计算库存与库龄。

01

# 需求描述

现有一表记录产品进出库

product\_id(产品代码) inoutdate（出入库日期） qty（数量）

001 2004-01-01 10

001 2004-01-03 -8

001 2004-01-04 -1

001 2004-01-05 5

001 2004-01-07 -6

其中数量为正表示入库，数量为负表示出库，现要计算任意日期时库存产品的库存天数。系统默认先进先出。

比如：

查询2004-01-02日库龄，则为10个、库龄为1天

查询2004-01-03日库龄，则为2个、 库龄为2天

查询2004-01-04日库龄，则为1个、 库龄为3天

查询2004-01-05日库龄，则为6个、 库龄为4天

查询2004-01-06日库龄，则为6个、 库龄为5天

查询2004-01-07日库龄，则为0个、 库龄为0

02

# 数据准备

```
create table pro as select '001' pid, '2020-07-01' date_id, 10 qty  union all  select '001' pid, '2020-07-03' date_id, -8 qty  union all  select '001' pid, '2020-07-04' date_id, -1 qty  union all  select '001' pid, '2020-07-05' date_id, 5 qty   union all  select '001' pid, '2020-07-07' date_id, -6 qty  union all  select '001' pid, '2020-07-08' date_id, 7 qty  union all  select '001' pid, '2020-07-09' date_id, 2 qty  union all  select '001' pid, '2020-07-10' date_id, -5 qty  union all  select '001' pid, '2020-07-11' date_id, -2 qty
```

03问题分析

# **分析题意**

分析题意：首先要理解库存量是个状态量，是入库数量与出库数量的差值，数据中为负值时表示出库数量，当为2004-01-02时候由于没有出库量，所以此时库存量依然为10，此时库龄为2，当前时间减去首次入库时间，依次类推，因而得到库存量实际上就是qty随时间的累计值，我们注意到当查询2004-01-07日库龄，则为0个、 库龄为0，此时库存量为0，库龄也清零，那么进入下一个周期的时候开始重新计数，计算库龄与库存量。因此本题的核心关键点在于根据累计量为0时作为分界点，开始重新计数，那么0则为断点，而此断点要归属于上一个周期中，那么问题的难点就在于如何根据断点划分不同周期，且将该断点划分到上一周期中。

另外需求中求的是累计量，数据表中明显时间不齐全，要计算状态量就需要先补全日期维度数据。

**第一步：补全数据，生成产品、时间维度表**

```
select pid, date_add(st, pos) date_id, posfrom (select pid,             st,             ed,             datediff(ed, st) + 1 diff      from (select pid, min(date_id) st, max(date_id) ed            from pro            group by pid) t) t lateral view posexplode(split(space(diff), '(?!$)')) tmp as pos, val
```

步骤2：时间维度表与数据表关联生成完整的数据表

```
select t.pid     , t.date_id     , nvl(p.qty, 0) qtyfrom (select pid, date_add(st, pos) date_id, pos      from (select pid,                   st,                   ed,                   datediff(ed, st) + 1 diff            from (select pid, min(date_id) st, max(date_id) ed                  from pro                  group by pid) t) t lateral view posexplode(split(space(diff), '(?!$)')) tmp as pos, val) t         left join pro p                   on t.pid = p.pid                       and t.date_id = p.date_id
```

步骤3

# 计算累计状态量即库存量

```
with data as (select t.pid,                     t.date_id,                     nvl(p.qty, 0) qty              from (select pid, date_add(st, pos) date_id, pos                    from (select pid,                                 st,                                 ed,                                 datediff(ed, st) + 1 diff                          from (select pid, min(date_id) st, max(date_id) ed                                from pro                                group by pid) t) t lateral view posexplode(split(space(diff), '(?!$)')) tmp as pos, val) t                       left join pro p                                 on t.pid = p.pid                                     and t.date_id = p.date_id)select pid     , date_id     , qty     , sum(qty) over (partition by pid order by date_id) stock_cntfrom data
```

**第四步：根据状态量，对数据进行重分组，断点是库存量为0 的地方**

#

注意此处，我们需要将断点归属到前一个分组中，而如果按照正序排序，断点则会归属到下一个分组中，因此我们调整排序的顺序采用降序的方式，找出分组标记。

**当在断点处时候表示事件的发生，用1表示，非断点处也就是事件发生处用0，采用累计量来反应状态变化，当累计量不变时表示前后状态一致，那么状态一致的则进入同一个分组中，当累计到断点处状态改变说明进入下一个状态，同理将累计量一样的分到一组。**

**理解:累计量其实就是对状态变化的反应，若前后累计量一致说明状态一致属于同一个周期中，当累计量改变说明状态发生变化则进入下一个周期中，我们把这种方法叫断点重分组**

```
with data as (select t.pid,                     t.date_id,                     nvl(p.qty, 0) qty              from (select pid, date_add(st, pos) date_id, pos                    from (select pid,                                 st,                                 ed,                                 datediff(ed, st) + 1 diff                          from (select pid, min(date_id) st, max(date_id) ed                                from pro                                group by pid) t) t lateral view posexplode(split(space(diff), '(?!$)')) tmp as pos, val) t                       left join pro p                                 on t.pid = p.pid                                     and t.date_id = p.date_id)
select pid     , date_id     , qty     , stock_cnt     ---当在断点处时候表示事件的发生，用1表示，非断点处也就是事件发生处用0，采用累计量来反应状态变化，当累计量不变时表示前后     ---状态一致，那么状态一致的则进入同一个分组中，当累计到断点处状态改变说明进入下一个状态，同理将累计量一样的分到一组     ---理解:累计量其实就是对状态变化的反应，若前后累计量一致说明状态一致属于同一个周期中，当累计量改变说明状态发生变化则进入下一个周期中，我们把这种     ---方法叫断点重分组     , sum(case when stock_cnt = 0 then 1 else 0 end) over (partition by pid order by date_id desc) stagefrom (select pid           , date_id           , qty           , sum(qty) over (partition by pid order by date_id) stock_cnt      from data) t
```

**第五步：基于分组标志重新计算库存量和库龄**

```
with data as (select t.pid,                     t.date_id,                     nvl(p.qty, 0) qty              from (select pid, date_add(st, pos) date_id, pos                    from (select pid,                                 st,                                 ed,                                 datediff(ed, st) + 1 diff                          from (select pid, min(date_id) st, max(date_id) ed                                from pro                                group by pid) t) t lateral view posexplode(split(space(diff), '(?!$)')) tmp as pos, val) t                       left join pro p                                 on t.pid = p.pid                                     and t.date_id = p.date_id)select pid     , date_id     , qty     , stock_cnt                                                                       --库存量     , case when stock_cnt = 0 then 0 else datediff(date_id, st_date_id) end stock_age --库龄from (select pid           , date_id           , qty           , sum(qty) over (partition by pid,stage order by date_id) stock_cnt --库存量           --计算当前阶段的开始时间           , min(date_id) over (partition by pid,stage)              st_date_id      from (select pid                 , date_id                 , qty                 , stock_cnt                 ---当在断点处时候表示事件的发生，用1表示，非断点处也就是事件发生处用0，采用累计量来反应状态变化，当累计量不变时表示前后                 ---状态一致，那么状态一致的则进入同一个分组中，当累计到断点处状态改变说明进入下一个状态，同理将累计量一样的分到一组                 ---理解:累计量其实就是对状态变化的反应，若前后累计量一致说明状态一致属于同一个周期中，当累计量改变说明状态发生变化则进入下一个周期中，我们把这种                 ---方法叫断点重分组                 , sum(case when stock_cnt = 0 then 1 else 0 end) over (partition by pid order by date_id desc) stage            from (select pid                       , date_id                       , qty                       , sum(qty) over (partition by pid order by date_id) stock_cnt                  from data) t) t) torder by date_id
```

04

# 小结

本文讲解了先进先出的库龄计算问题，针对库龄计算首先要理解状态量变化的原理，以及能识别出本题问题点所在。解决本题关键点在于对于先对状态量的判断，找出断点，并基于该断点能够重新分组，然后再计算库存与库龄。

# 往期精彩

01

[数仓建模：如何做好业务阶段模型建设？](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484260&idx=1&sn=6a0c1aa64fc0a158df013127822ea516&chksm=e8e21744df959e520b992efa616ee6bf3a898801ca33727ce58f07b892d475c213f626a2fa91&scene=21#wechat_redirect)

02

[SQL进阶技巧：如何使用差值累计计算思想巧解数学中递归计算问题](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484244&idx=1&sn=56c9518aaebf76e1e4b01672ec57a899&chksm=e8e21774df959e627e69e46eee78da52db0d62982ac8f282b2d214105f6185e29945a75d8374&scene=21#wechat_redirect)

03

[数仓建模：如何设计通用的DWS层数据模型？](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484286&idx=1&sn=723c82e264d14a20708252a1a7d7a95f&chksm=e8e2175edf959e48d04247d7c47b528757cfb3e3ca7996a2537bf2b7ac76286f57a60462bb0c&scene=21#wechat_redirect)

###### 会飞的一十六

了解更多公众号内容

扫码二维码关注我们

点一点「在看」支持我们~