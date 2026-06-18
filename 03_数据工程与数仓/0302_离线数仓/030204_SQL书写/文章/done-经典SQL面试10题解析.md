---
title: 经典SQL面试10题解析
author: 智海观潮
date:
url: http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247492370&idx=1&sn=30a2506b9edf404feb7b6c574525dc20&chksm=e9751f28de02963e27a37d09aebf004f844e1c1859e2db61e171ba6292329b85bba592875ee2&mpshare=1&scene=24&srcid=0401VewJsoC7fjTd8DBHrPyx&sharer_sharetime=1648795155222&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL口径验证与对账闭环|SQL口径验证与对账闭环]]


**一、提要**

作为一名数据工作人员，SQL是日常工作中最常用的数据提取&简单预处理语言。因为其使用的广泛性和易学程度也被其他岗位比如产品经理、研发广泛学习使用，本篇文章主要结合经典面试题，给出通过数据开发面试的SQL方法与实战。**以下题目均来与笔者经历&网上分享的中高难度SQL题。**

**二、解题思路**

* **简单**——会考察一些group by & limit之类的用法，或者平时用的不多的函数比如rand()类；会涉及到一些表之间的关联

* **中等**——会考察一些窗口函数的基本用法；会有表之间的关联，相对tricky的地方在于会有一些自关联的使用

* **困难**——会有中位数或者更加复杂的取数概念，可能要求按照某特定要求生成列；一般这种题建中间表会解得清晰些

**三、SQL真题**

**第一题**

* **order订单表，字段为：**goods\_id， amount ；

* **pv 浏览表，字段为：**goods\_id，uid；

* goods按照总销售金额排序，分成top10,top10~top20,其他三组

**求每组商品的浏览用户数（同组内同一用户只能算一次）**

```
create table if not exists test.nil_goods_category asselect goods_id,case when nn<= 10 then 'top10'      when nn<= 20 then 'top10~top20'      else 'other' end as goods_groupfrom(    select goods_id    ,row_number() over(partition by goods_id order by sale_sum desc) as nn    from    (        select goods_id,sum(amount) as sale_sum        from order        group by 1    ) aa) bb;select b.goods_group,count(distinct a.uid) as numfrom pv a left join test.nil_goods_category b on a.goods_id = b.goods_idgroup by 1;
```

**第二题**

商品活动表 goods\_event，g\_id（有可能重复），t1（开始时间）,t2（结束时间）

给定时间段（t3,t4)，**求在时间段内做活动的商品数**

```
1.select count(distinct g_id) as event_goods_numfrom goods_eventwhere (t1<=t4 and t1>=t3) or (t2>=t3 and t2<=t4)

2.select count(distinct g_id) as event_goods_numfrom goods_eventwhere (t1<=t4 and t1>=t3) union all
```

**第三题**

**商品活动流水表，表名为event，字段：**goods\_id， time；

**求参加活动次数最多的商品的最近一次参加活动的时间**

```
select a.goods_id,a.timefrom event a inner join(    select goods_id,count(*)    from event    group by gooods_id    order by count(*) desc    limit 1) bon a.goods_id = b.goods_idorder by a.goods_id,a.time desc
```

**第四题**

用户登录的log数据，划定session，同一个用户一个小时之内的登录算一个session；

**生成session列**

```
drop table if exists koo.nil_temp0222_a2;create table if not exists koo.nil_temp0222_a2 asselect *    ,row_number() over(partition by userid order by inserttime) as nn1from(    select a.*    ,b.inserttime as inserttime_aftr    ,datediff(b.inserttime,a.inserttime) as session_diff  from  (    select userid,inserttime      ,row_number() over(partition by userid order by inserttime asc) nn    from koo.nil_temp0222     where userid = 1900000169  ) a     left join  (     select userid,inserttime      ,row_number() over(partition by userid order by inserttime asc) nn    from koo.nil_temp0222     where userid = 1900000169  ) b  on a.userid =  b.userid and a.nn = b.nn-1) aawhere session_diff >10 or nn = 1order by userid,inserttime;

drop table if exists koo.nil_temp0222_a2_1;create table if not exists koo.nil_temp0222_a2_1 asselect a.*,case when b.nn is null then a.nn+3 else b.nn end as nn_endfrom koo.nil_temp0222_a2 a left join koo.nil_temp0222_a2 b on a.userid = b.userid and a.nn1 = b.nn1 - 1;select a.*,b.nn1 as session_idfrom(  select userid,inserttime    ,row_number() over(partition by userid order by inserttime asc) nn  from koo.nil_temp0222   where userid = 1900000169) aleft join koo.nil_temp0222_a2_1 b on a.userid = b.useridand a.nn>=b.nnand a.nn<b.nn_end
```

**第五题**

订单表，字段有订单编号和时间；



**取每月最后一天的最后三笔订单**

```
select *from(  select *  ,rank() over(partition by mm order by dd desc) as nn1  ,row_number() over(partition by mm,dd order by inserttime desc) as nn2  from  (select cast(right(to_date(inserttime),2) as int) as dd,month(inserttime) as mm,userid,inserttime  from koo.nil_temp0222) aa ) bb where nn1 = 1 and nn2<=3;
```

**第六题**

数据库表Tourists，记录了某个景点7月份每天来访游客的数量如下：

id date visits 1 2017-07-01 100 …… 非常巧，id字段刚好等于日期里面的几号。

**现在请筛选出连续三天都有大于100天的日期。**

**上面例子的输出为：**date 2017-07-01 ……

```
select a.*,b.num as num2,c.num as num3from table  a left join table bon a.userid = b.useridand a.dt = date_add(b.dt,-1)left join table con a.userid = c.useridand a.dt = date_add(c.dt,-2)where b.num>100and a.num>100and c.num>100
```

**第七题**

现有A表，有21个列，第一列id，剩余列为特征字段，列名从d1-d20，共10W条数据！

另外一个表B称为模式表，和A表结构一样，共5W条数据

**请找到A表中的特征符合B表中模式的数据，并记录下相对应的id**

**有两种情况满足要求：**

* 每个特征列都完全匹配的情况下

* 最多有一个特征列不匹配，其他19个特征列都完全匹配，但哪个列不匹配未知

```
1.select aa.*from(  select *,concat(d1,d2,d3……d20) as mmd  from table) aa left join(  select id,concat(d1,d2,d3……d20) as mmd  from table) bb on aa.id = bb.idand aa.mmd = bb.mmd

2.select a.*,sum(d1_jp,d2_jp……,d20_jp) as same_judgefrom(  select a.*  ,case when a.d1 = b.d1 then 1 else 0 end as d1_jp  ,case when a.d2 = b.d2 then 1 else 0 end as d2_jp  ,case when a.d3 = b.d3 then 1 else 0 end as d3_jp  ,case when a.d4 = b.d4 then 1 else 0 end as d4_jp  ,case when a.d5 = b.d5 then 1 else 0 end as d5_jp  ,case when a.d6 = b.d6 then 1 else 0 end as d6_jp  ,case when a.d7 = b.d7 then 1 else 0 end as d7_jp  ,case when a.d8 = b.d8 then 1 else 0 end as d8_jp  ,case when a.d9 = b.d9 then 1 else 0 end as d9_jp  ,case when a.d10 = b.d10 then 1 else 0 end as d10_jp  ,case when a.d20 = b.d20 then 1 else 0 end as d20_jp  ,case when a.d11 = b.d11 then 1 else 0 end as d11_jp  ,case when a.d12 = b.d12 then 1 else 0 end as d12_jp  ,case when a.d13 = b.d13 then 1 else 0 end as d13_jp  ,case when a.d14 = b.d14 then 1 else 0 end as d14_jp  ,case when a.d15 = b.d15 then 1 else 0 end as d15_jp  ,case when a.d16 = b.d16 then 1 else 0 end as d16_jp  ,case when a.d17 = b.d17 then 1 else 0 end as d17_jp  ,case when a.d18 = b.d18 then 1 else 0 end as d18_jp  ,case when a.d19 = b.d19 then 1 else 0 end as d19_jp  from table a   left join table b   on a.id = b.id ) aawhere sum(d1_jp,d2_jp……,d20_jp) = 19
```

**第八题**

**我们把用户对商品的评分用稀疏向量表示，保存在数据库表t里面：**

* **t的字段有：**uid，goods\_id，star。uid是用户id

* goodsid是商品id

* star是用户对该商品的评分，值为1-5

现在我们想要**计算向量两两之间的内积，**内积在这里的语义为：

对于两个不同的用户，如果他们都对同样的一批[商品](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247490996&idx=1&sn=68d16e707e1e0c4f6467d38062b245db&chksm=e976e18ede0168982fff0610734a03a61d72cc7fa083926161148b1a2b9512e94da79f93811e&scene=21#wechat_redirect)打了分，那么对于这里面的每个人的分数乘起来，并对这些乘积求和。

**例子，数据库表里有以下的数据：**

U0 g0 2
U0 g1 4
U1 g0 3
U1 g1 1

**计算后的结果为：**

U0 U1 23+41=10 ……

```
select aa.uid1,aa.uid2,sum(star_multi) as resultfrom(  select a.uid as uid1  ,b.uid as uid2  ,a.goods_id  ,a.star * b.star as star_multi  from t a   left join t b   on a.goods_id = b.goods_id  and a.udi<>b.uid  ) aa group by 1,2
```

```
select uid1,uid2,sum(multiply) as resultfrom(select t.uid as uid1, t.uid as uid2, goods_id,a.star*star as multiplyfrom a left join b on a.goods_id = goods_idand a.uid<>uid) aagroup by goods
```

**第九题**

给出一堆数和频数的表格，**统计这一堆数中位数**

```
select a.*,b.s_mid_n,c.l_mid_n,avg(b.s_mid_n,c.l_mid_n)from(  select  case when mod(count(*),2) = 0 then count(*)/2 else (count(*)+1)/2 end as s_mid  ,case when mod(count(*),2) = 0 then count(*)/2+1 else (count(*)+1)/2 end as l_mid    from table) a left join(  select id,num,row_number() over(partition by id order by num asc) nn  from table) b on a.s_mid = b.nnleft join(  select id,num,row_number() over(partition by id order by num asc) nn  from table) c  on a.l_mid = c.nn
```

**第十题**

表order有三个字段，店铺ID，[订单](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247486633&idx=1&sn=0749c4f7eaac7886fd9a27b351b96538&chksm=e976f093de017985f37fd384c8482cab04b9f9f0e0509eb960844fd3fd8f45872171961342c0&scene=21#wechat_redirect)时间，订单金额



**查询一个月内每周都有销量的店铺**

```
select distinct credit_levelfrom(  select credit_level,count(distinct nn) as number  from  (    select userid,credit_level,inserttime,month(inserttime) as mm    ,weekofyear(inserttime) as week    ,dense_rank() over(partition by credit_level,month(inserttime) order by weekofyear(inserttime) asc) as nn    from koo.nil_temp0222     where substring(inserttime,1,7) = '2019-12'    order by credit_level ,inserttime    ) aa   group by 1  ) bbwhere number = (select count(distinct weekofyear(inserttime))from koo.nil_temp0222 where substring(inserttime,1,7) = '2019-12')
```

**推荐文章：**[经典的SparkSQL/Hive-SQL/MySQL面试-练习题](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484857&idx=1&sn=9b5fe685f0baf1c7f82b77a7ce81d8cf&chksm=e976f983de0170959a0df7ddf415551fe56ac3bdd09e677d02b6c80e3d79d429ec64517a526e&scene=21#wechat_redirect)