---
title: 字节跳动面试SQL-补全日期且补全数据
author: 大模型数据的朋友
date: 
url: http://mp.weixin.qq.com/s?__biz=MzUzMjAxNDA5OQ==&mid=2247484167&idx=1&sn=9aa7fa233594a04467b4a36331892866&chksm=fab8f67bcdcf7f6d9691d57686e4f50d7b71d0e24ed445729ff38bfec43abef90c73812cc158&mpshare=1&scene=24&srcid=0713Rg6lsDIoW1AVu3mumsBN&sharer_shareinfo=7ee77e36ca7233e485cff0c5f36f4d5e&sharer_shareinfo_first=7ee77e36ca7233e485cff0c5f36f4d5e#rd
---

有一个用户交易账户表deal，表中有三个字段：

+ dt: 用户交易的日期
+ uid: 用户id
+ amt: 用户账户金额

```
举例数据如下:  dt         uid  amt   2024-01-01   a   1002024-01-02   a   80  2024-01-04   a   60  2024-01-06   a   40  2024-01-10   a   10
```

求：用户的每天平均金额，如果日期中断当天没有数据, 则取上一天数据的账户金额数据；

最后呈现的结果展示如下: 

```
展示如下:  dt         uid  amt   avg_amt 2024-01-01   a   100     100 2024-01-02   a   80      90 2024-01-03   a   80      86.67 2024-01-04   a   60      802024-01-05   a   60      762024-01-06   a   40      702024-01-07   a   40      65.712024-01-08   a   40      62.52024-01-09   a   40      602024-01-10   a   10      55
```

你可以先思考下，如果被问到的话有没有解决思路了？没有思路再看以下解题思路。

（PS扩展: 如果日期中断取后一天金额数据呢？）  

**解决思路一：**

本题难点在于没有的日期需要给它补全了，没有的数据需要补全了。

* 先求出最早和最晚日期，得到日期天数差
* 利用函数space结合posexplode、 laterval view 获取完整连续的日期
* 拿完整的日期和原表关联，利用last\_value(amt, true)补全金额数据
* 最后求取用户每天的平均金额

```
with deal as (    select '2024-01-01' as dt, 'a' as uid, 100 as amt union all      select '2024-01-02' as dt, 'a' as uid, 80 as amt union all     select '2024-01-04' as dt, 'a' as uid, 60 as amt union all     select '2024-01-06' as dt, 'a' as uid, 40 as amt union all     select '2024-01-10' as dt, 'a' as uid, 10 as amt),-- 求最小和最大日期 deal_min_max_dt as (select     uid,    min(dt) as start_date,     max(dt) as end_date from deal group by uid),--补全缺失的日期date_addition as (select     uid,    date_add(start_date, pos) dtfrom deal_min_max_dt lateral view posexplode(split(space(datediff(end_date, start_date)), '')) t as pos, val),-- 分组内排序 取最后一行的值 补全金额数据  day_amt as (select    t1.dt,    t1.uid,    t2.amt,    coalesce(t2.amt, 0) as amt_nvl, --如果为null即为0      last_value(t2.amt,false) over(partition by t1.uid order by t1.dt) as amo,     last_value(t2.amt,true) over(partition by t1.uid order by t1.dt) as amount,      --如果为null取上面日期数据    last_value(t2.amt,true) over(partition by t1.uid order by t1.dt desc) as amount2 -- 如果为null取下面日期数据 from date_addition t1left join deal t2 on t1.dt = t2.dt)-- 得到用户每天的平均金额 select  uid,   dt,   amt,  amt_nvl,  amount,  amount2,  round(avg(amount) over(partition by uid order by dt), 2) as avg_amount,  round(sum(amount) over(partition by uid order by dt)/count(dt) over(partition by uid order by dt), 2) as avg_amount2,    sum(amount) over(partition by uid order by dt) as sum_amount,  count(dt) over(partition by uid order by dt) as cnt_dt from  day_amt
```

结果如下：

```
+------+-------------+-------+----------+---------+----------+-------------+--------------+-------------+---------+| uid  |     dt      |  amt  | amt_nvl  | amount  | amount2  | avg_amount  | avg_amount2  | sum_amount  | cnt_dt  |+------+-------------+-------+----------+---------+----------+-------------+--------------+-------------+---------+| a    | 2024-01-01  | 100   | 100      | 100     | 100      | 100.0       | 100.0        | 100         | 1       || a    | 2024-01-02  | 80    | 80       | 80      | 80       | 90.0        | 90.0         | 180         | 2       || a    | 2024-01-03  | NULL  | 0        | 80      | 60       | 86.67       | 86.67        | 260         | 3       || a    | 2024-01-04  | 60    | 60       | 60      | 60       | 80.0        | 80.0         | 320         | 4       || a    | 2024-01-05  | NULL  | 0        | 60      | 40       | 76.0        | 76.0         | 380         | 5       || a    | 2024-01-06  | 40    | 40       | 40      | 40       | 70.0        | 70.0         | 420         | 6       || a    | 2024-01-07  | NULL  | 0        | 40      | 10       | 65.71       | 65.71        | 460         | 7       || a    | 2024-01-08  | NULL  | 0        | 40      | 10       | 62.5        | 62.5         | 500         | 8       || a    | 2024-01-09  | NULL  | 0        | 40      | 10       | 60.0        | 60.0         | 540         | 9       || a    | 2024-01-10  | 10    | 10       | 10      | 10       | 55.0        | 55.0         | 550         | 10      |+------+-------------+-------+----------+---------+----------+-------------+--------------+-------------+---------+-
```

在 Hive 中，space函数用于生成指定长度的空格字符串。其语法为：space(int n)，其中n表示要生成的空格字符串的长度。

以下是space函数的一些常见用法：

1. 生成指定长度的空格字符串： select space(10); 上述查询将返回一个长度为10的空格字符串。

2. space与split函数结合使用得到数组：

```
select split(space(10),'') as array; +-----------------------------------------------+|            array                              |+-----------------------------------------------+| [" "," "," "," "," "," "," "," "," "," ",""]  |+-----------------------------------------------+
```

这里会得到一个包含多个空格字符串的数组，其中空格字符串的数量取决于split函数的分割情况。需要注意的是，由于一个空格在分割时会被切割为左右两个空串，所以在生成空格时，仅需要number-1个即可满足需求。若要生成包含3个空格字符串的数组，使用split(space(3-1),'')

3. 结合 split 函数、 posexplode 函数和 lateral view 函数生成连续数字：

```
 select     (id_start + pos) as id  from (select 1 as id_start, 10 as id_end) table  lateral view posexplode(split(space(id_end - id_start), '')) t as pos, val ;  
+-----+| id  |+-----+| 1   || 2   || 3   || 4   || 5   || 6   || 7   || 8   || 9   || 10  |+-----+
```

**解决思路二：**

利用数仓中的日历表与原表deal关联，再开窗排序取第一条，也可以获取每一天的数据，最后再计算平均金额。

```
with deal as (    select '2024-01-01' as dt, 'a' as uid, 100 as amt union all      select '2024-01-02' as dt, 'a' as uid, 80 as amt union all     select '2024-01-04' as dt, 'a' as uid, 60 as amt union all     select '2024-01-06' as dt, 'a' as uid, 40 as amt union all     select '2024-01-10' as dt, 'a' as uid, 10 as amt),-- 日历表  dim_date_tool as (    select       ymd,       from_unixtime(unix_timestamp(cast(ymd as string), 'yyyyMMdd') ,'yyyy-MM-dd') as day    from dim_calendar_tools     where ymd <= from_unixtime(unix_timestamp(),'yyyyMMdd') ),--开窗排序 deal_day_rn as (     select        t1.day,        t2.dt,        t2.uid,        t2.amt,        ROW_NUMBER() over(partition by t1.day order by t2.dt asc) as rn ---取后一天的改为倒序排列       from dim_date_tool t1     left join deal t2 on t1.day <= t2.dt    ---取后一天的改为 >=      order by  1,2,3),--- 获取第一个 deal_day_rn_1 as (select    day,   uid,   amt from deal_day_rn where rn = 1order by day )  -- 得到用户每天的平均金额 select  uid,   day,   amt,  round(avg(amt) over(partition by uid order by day), 2) as avg_amount,  round(sum(amt) over(partition by uid order by day)/count(day) over(partition by uid order by day), 2) as avg_amount2,    sum(amt) over(partition by uid order by day) as sum_amount,  count(day) over(partition by uid order by day) as cnt_dt from  deal_day_rn_1 order by day
```

最后结果如下：

```
+-------+-------------+-------+-------------+--------------+-------------+---------+--+|  uid  |     day     |  amt  | avg_amount  | avg_amount2  | sum_amount  | cnt_dt  |+-------+-------------+-------+-------------+--------------+-------------+---------+--+| a     | 2024-01-01  | 100   | 100.0       | 100.0        | 100         | 1       || a     | 2024-01-02  | 80    | 90.0        | 90.0         | 180         | 2       || a     | 2024-01-03  | 60    | 80.0        | 80.0         | 240         | 3       || a     | 2024-01-04  | 60    | 75.0        | 75.0         | 300         | 4       || a     | 2024-01-05  | 40    | 68.0        | 68.0         | 340         | 5       || a     | 2024-01-06  | 40    | 63.33       | 63.33        | 380         | 6       || a     | 2024-01-07  | 10    | 55.71       | 55.71        | 390         | 7       || a     | 2024-01-08  | 10    | 50.0        | 50.0         | 400         | 8       || a     | 2024-01-09  | 10    | 45.56       | 45.56        | 410         | 9       || a     | 2024-01-10  | 10    | 42.0        | 42.0         | 420         | 10      |
```

总结： 学习并理解函数space结合split、posexplode、laterval view生成自增有序数字。学会利用日历表与日期相关的关联可以解决绝大多数问题。希望这篇文章比你有帮助。

-关注我-

90后北漂男孩

现任互联网大厂大数据工程师

想要影响1000+个人变得更好

一起遇见未知的自己

一起人生长跑

成为自己的光

💛

多点一下**在看**多一条小鱼干