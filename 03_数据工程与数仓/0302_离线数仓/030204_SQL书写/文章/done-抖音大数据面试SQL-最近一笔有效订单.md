---
title: 抖音大数据面试SQL-最近一笔有效订单
author: 大模型数据的朋友
date:
url: http://mp.weixin.qq.com/s?__biz=MzUzMjAxNDA5OQ==&mid=2247484385&idx=1&sn=c042f3fa725e9f5912f94b1b3fe72b81&chksm=fb216664328aeb46e614db86e1fcfb0aaf39b08331e16d0ffca3c4a64be8946a613e68ba718a&mpshare=1&scene=24&srcid=1105ZpLEKM7lpoujrCioEmm6&sharer_shareinfo=ea2c4a9112be296815d6655d345af08a&sharer_shareinfo_first=ea2c4a9112be296815d6655d345af08a#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL窗口滚动与序列问题|SQL窗口滚动与序列问题]]


我们在购物时，有时候要凑单导致下单后没有支付，这个就属于无效单，会记录在订单表order\_detail中，表中有四个字段：

+ order\_id: 订单id
+ order\_time: 下单时间
+ user\_id: 下单的用户id
+ is\_valid: 是否有效订单

需求想要查询每笔订单的上一笔有效订单？

```
举例数据如下:  user_id     order_time          order_id   is_valid      jack      2024-11-11 20:00:00    A101       0  jack      2024-11-11 20:02:05    A102       0  jack      2024-11-11 20:04:05    A103       1  jack      2024-11-11 20:08:08    A104       1          lily      2024-11-11 20:02:05    B105       1   lily      2024-11-11 20:04:05    B106       1   lily      2024-11-11 20:06:05    B107       0   lily      2024-11-11 20:08:09    B108       1  lily      2024-11-11 20:10:09    B109       1  
create temporary view order_detail as select 'jack' as u_name, '2024-11-11 20:00:00' as order_time, 'A101' as order_id, 0 as is_valid union all select 'jack' as u_name, '2024-11-11 20:02:05' as order_time, 'A102' as order_id, 0 as is_valid union all select 'jack' as u_name, '2024-11-11 20:04:05' as order_time, 'A103' as order_id, 1 as is_valid union all select 'jack' as u_name, '2024-11-11 20:08:08' as order_time, 'A104' as order_id, 1 as is_valid union all select 'lily' as u_name, '2024-11-11 20:02:05' as order_time, 'B105' as order_id, 1 as is_valid union all select 'lily' as u_name, '2024-11-11 20:04:05' as order_time, 'B106' as order_id, 1 as is_valid union all select 'lily' as u_name, '2024-11-11 20:06:05' as order_time, 'B107' as order_id, 0 as is_valid union all select 'lily' as u_name, '2024-11-11 20:08:09' as order_time, 'B108' as order_id, 1 as is_valid union all select 'lily' as u_name, '2024-11-11 20:10:09' as order_time, 'B109' as order_id, 1 as is_valid;    
```

**解决思路1：**

+ 利用last\_value函数，跳过null值，返回当前最后一个值；
+ 需要在前一行到往前无穷尽行为窗口获取last\_value的订单；

sql如下所示 

```
select   u_name, order_time, order_id, is_valid, last_value(case when is_valid=1 then order_id else null end, true)      over(partition by u_name order by order_time rows between unbounded preceding and 1 preceding) as last_valid_order_id from order_detail;
```

结果如下所示：

```
+---------+----------------------+-----------+-----------+----------------------+| u_name  |      order_time      | order_id  | is_valid  | last_valid_order_id  |+---------+----------------------+-----------+-----------+----------------------+| jack    | 2024-11-11 20:00:00  | A101      | 0         | NULL                 || jack    | 2024-11-11 20:02:05  | A102      | 0         | NULL                 || jack    | 2024-11-11 20:04:05  | A103      | 1         | NULL                 || jack    | 2024-11-11 20:08:08  | A104      | 1         | A103                 || lily    | 2024-11-11 20:02:05  | B105      | 1         | NULL                 || lily    | 2024-11-11 20:04:05  | B106      | 1         | B105                 || lily    | 2024-11-11 20:06:05  | B107      | 0         | B106                 || lily    | 2024-11-11 20:08:09  | B108      | 1         | B106                 || lily    | 2024-11-11 20:10:09  | B109      | 1         | B108                 |+---------+----------------------+-----------+-----------+----------------------+
```

**PS: last\_value的用法**

在 Hive 中，last\_value()是一个窗口函数，完整语法是

```
last_value(expr ,[ignore_nulls]) OVER ( [PARTITION BY partition_expression] [ORDER BY sort_expression [ASC|DESC]] [frame_clause])
```

+ expr是要获取最后一个值的表达式，通常是列名。
+ ignore\_nulls是一个可选参数, 当设置为true时，表示在计算最后一个值时忽略NULL值。默认不设置即设置为false，NULL值会参与计算。
+ PARTITION BY用于对数据分区，为每个分区内独立计算最后一个值。
+ ORDER BY确定计算最后一个值的顺序。
+ frame\_clause 定义了窗口范围，用于指定计算窗口函数的数据集范围。

```
例如: SELECT     employee_id,     evaluation_date,     performance_score,    last_value(performance_score, true) OVER (PARTITION BY employee_id ORDER BY evaluation_date) AS last_non_null_scoreFROM employee_performance;
```

通过PARTITION BY employee\_id ，我们为每个员工划分一个分区。在每个员工的分区内，按照 evaluation\_date 排序。

使用 last\_value(performance\_score, true)函数，计算每个员工分区内最后一个非NULL的绩效分数，并将其存储在 last\_non\_null\_score 列中。

与默认行为对比（ignore\_nulls为false）

+ 如果不设置 ignore\_nulls 为true（即默认情况下或者设置为false），并且数据中存在NULL值，那么last\_value函数会把NULL值当作普通的值来处理。
+ 例如，数据序列是1, 2, NULL, 3，在默认情况下计算最后一个值会得到3，但如果使用last\_value(expr, true)，在计算过程中会跳过NULL值，最后得到的结果仍然是2。

注意事项

+ 窗口范围（frame\_clause）的定义对last\_value函数的结果有很大影响。如果窗口范围没有正确定义，可能会得到不符合预期的结果。例如，若将窗口范围定义得太窄，可能会错过真正的最后一个值。
+ 当使用PARTITION BY和ORDER BY时，要确保排序顺序符合业务逻辑和数据的实际情况，这样才能准确地计算出每个分区内的最后一个值。

**解题思路二：**

+ 先查询出有效订单；
+ 利用lag函数计算出每笔订单有效订单的上一单的有效订单；
+ 明细数据与新的有效订单表按用户名关联，有效订单表的订单时间大于等于原始订单表；
+ 利用row\_number函数，分组排序，删选rn为1的记录

```
with valid_order as (select    u_name,   order_time,   order_id,   is_valid from order_detail  where is_valid = 1),--每个有效订单的上一个有效订单 lag_valid_order as (select    u_name,  order_time,  order_id,  is_valid,  lag(order_id) over(partition by u_name order by order_time) as last_valid_order_idfrom valid_order ),order_index as (select    t1.u_name,  t1.order_time,  t1.order_id,  t1.is_valid,  t2.last_valid_order_id,  row_number() over(partition by t1.u_name,t1.order_id order by t2.order_time) as rn from order_detail t1 left join lag_valid_order t2 on t1.u_name=t2.u_name where t1.order_time <= t2.order_time)  select     u_name,   order_time,   order_id,   is_valid,  last_valid_order_idfrom order_index where rn = 1
```

**解题思路三：**

订单详情表进行自关联的方式求取，这个方式sql交给你可以试着写下，看看有没有思路，欢迎大家评论区进行讨论。

总结：在处理最近一笔有效订单问题时，我们不知道上一单是有效的还是无效的，我们可以使用lag函数或last\_value函数进行处理，最终获取到我们想要的结果。

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