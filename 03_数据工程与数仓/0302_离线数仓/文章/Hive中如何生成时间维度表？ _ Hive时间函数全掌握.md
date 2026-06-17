---
title: Hive中如何生成时间维度表？ |   Hive时间函数全掌握
author: 会飞的一十六
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247485068&idx=1&sn=f75a146fe54e8e4e936b33032692f8ee&chksm=e99be2f03743f7178bc6f65cfa21a2bcdbc20d974f63570696bbffff375d85040b8f4340eab9&mpshare=1&scene=24&srcid=09194tWxlldsIHDcBFhEdKv7&sharer_shareinfo=087a0bb28055d5ef7797b5c1401d87c3&sharer_shareinfo_first=087a0bb28055d5ef7797b5c1401d87c3#rd
---

01

—

问题描述

使用SQL 实现一张日期维度表，包含以下字段：

```
drop table if exists dim_date;create table if not exists dim_date(`date` string comment '日期',d_week string comment '年内第几周',weeks string comment '周几',w_start string comment '周开始日',w_end string comment '周结束日',d_month string comment '第几月',m_start string comment '月开始日',m_end string comment '月结束日',d_quarter int comment '第几季',q_start string comment '季开始日',q_end string comment '季结束日',d_year int comment '年份',y_start string comment '年开始日',y_end string comment '年结束日');  
--自然月: 指每月的1 号到那个月的月底，它是按照阳历来计算的。就是从每月1 号到月底，不管这个月有30 天，31 天，29 天或者28 天，都算是一个自然月。
```

02

—

具体实现

Hive低版本使用

```
select `date`     , d_week                                                                                         --年内第几周     , case weekid           when 0 then '周日'           when 1 then '周一'           when 2 then '周二'           when 3 then '周三'           when 4 then '周四'           when 5 then '周五'           when 6 then '周六'    end                                                                                    as weeks   -- 周     , date_sub(next_day(`date`, 'MO'), 7)                                                 as w_start --周一     , date_sub(next_day(`date`, 'MO'), 1)                                                 as w_end   -- 周日_end-- 月份日期     , monthid                                                                             as d_month     , m_start     , m_end-- 季节     , quarterid                                                                           as d_quart     , concat_ws('-', cast(d_year as string), cast(quarterid * 3 - 2 as string), '1')              as q_start --季开始日     , date_sub(concat_ws('-', cast(d_year as string), cast(quarterid * 3 + 1 as string), '1'), 1) as q_end   --季结束日     , d_year     , y_start     , y_endfrom (         select `date`  
              , pmod(datediff(`date`, '2012-01-01'), 7)            as weekid--获取周几              , month(`date`)                                      as monthid--获取月份              , ceil(month(`date`) / 3)                            as quarterid--获取季节id              , year(`date`)                                       as d_year-- 获取年份              , trunc(`date`, 'YYYY')                              as y_start--年开始日              , date_sub(trunc(add_months(`date`, 12), 'YYYY'), 1) as y_end --年结束日  
              , trunc(`date`, 'MM')                                as m_start--当月第一天              , last_day(`date`)                                   as m_end--当月最后一天              , weekofyear(`date`)                                 as d_week--年内第几周         from (                  -- '2021-01-01'是开始日期, '2022-05-31'是截止日期                  select date_add('2021-01-01', t0.pos) as `date`                  from (                           select posexplode(split(repeat('o', datediff('2022-05-31', '2021-01-01')), 'o'))                       ) t0              ) t1     ) t2;
```

或Hive高版本使用

```
select `date`     , d_week                                                                                         --年内第几周     , case weekid           when 0 then '周日'           when 1 then '周一'           when 2 then '周二'           when 3 then '周三'           when 4 then '周四'           when 5 then '周五'           when 6 then '周六'    end                                                                                    as weeks   -- 周     , date_sub(next_day(`date`, 'MO'), 7)                                                 as w_start --周一     , date_sub(next_day(`date`, 'MO'), 1)                                                 as w_end   -- 周日_end-- 月份日期     , monthid                                                                             as d_month     , m_start     , m_end-- 季节     , quarterid                                                                           as d_quart     , concat_ws('-', cast(d_year as string), lpad(cast(quarterid * 3 - 2 as string),2,0), '1')              as q_start --季开始日     , date_sub(concat_ws('-', cast(d_year as string), cast(quarterid * 3 + 1 as string), '1'), 1) as q_end   --季结束日     , d_year     , y_start     , y_endfrom (         select `date`  
              , dayofweek(`date`) -1                              as weekid--获取周几              , month(`date`)                                      as monthid--获取月份              , quarter(`date`)                                    as quarterid--获取季节id              , year(`date`)                                       as d_year-- 获取年份              , trunc(`date`, 'YYYY')                              as y_start--年开始日              , date_sub(trunc(add_months(`date`, 12), 'YYYY'), 1) as y_end --年结束日  
              , trunc(`date`, 'MM')                                as m_start--当月第一天              , last_day(`date`)                                   as m_end--当月最后一天              , weekofyear(`date`)                                 as d_week--年内第几周         from (                  -- '2021-01-01'是开始日期, '2022-05-31'是截止日期                  select date_add('2021-01-01', t0.pos) as `date`                  from (                           select posexplode(split(repeat('o', datediff('2022-05-31', '2021-01-01')), 'o'))                       ) t0              ) t1     ) t2;
```

03

—

Hive时间函数回顾

## 0 基础函数

### trunc()

trunc(string date,string format) — 返回日期的最开始日期

目前格式支持：MM（月）YYYY（年）Hive3.0后支持Q表示季度

```
select trunc(current_date,'MM') --月初select trunc(current_date,'YY') --年初
```

### last\_day()

last\_day(string date) — 返回该月最后一天的日期

```
select last_day(current_date());
```

### add\_months()

add\_months(string date,int months) — 返回months月之后的date,months可以是负数

```
select add_months(current_date(),1);
```

### current\_date()/current\_date

当前日期（天）

```
select current_date
```

### next\_day

next\_day(string date,string dayOfWeek) — 返回date之后的下一个周的dayOfWeek的天

求当前日期的下周一时间

```
select next_day(current_date,'MO')
```

### pmod()

pmod(int a, int b)

pmod(double a, double b)

返回a除以b的余数的绝对值

```
select pmod(10,4)
```

### to\_date

语法：to\_date(string timestamp)  
返回值: 2.1.0版本前返回string，2.1.0版本后返回date  
描述: 将一个字符串date按照"yyyy-MM-dd"格式转成日期值

```
select to_date(current_timestamp)
```

### year

语法：year(string date)  
返回值：int  
描述：提取日期中的年份部分

```
select year(current_timestamp)
```

### month

语法：month(string date)  
返回值: int  
描述：提取日期中的月份部分

```
select month(current_timestamp)
```

### hour

语法：hour(string date)  
返回值：int  
描述：提取日期中的小时部分

```
select hour(current_timestamp)
```

### dayofweek

获取当前日期的星期几。这里本周的第一天是周日，使用时候注意转换。Hive低版本没有该函数，Hive2.1版本之后才有，如果版本较低建议使用pmod方法获取当前日期的星期几，参考下文。

```
select dayofweek('2022-05-29 20:21:22')
```

### weekofyear

语法：weekofyear(string date)  
返回值：int  
描述: 返回指定日期处于当年的第几周

```
select weekofyear('2022-05-29') as weekofyear;
```

### quarter

获取当前日期所处的季度

```
select quarter('2022-05-29 20:21:22')
```

低版本没有该函数，可以用ceil(month(date) / 3)来代替

```
select CEIL(month('2022-05-29 20:21:22') / 3)
```

### datediff

语法：datediff(string enddate, string startdate)  
返回值: int  
描述: 返回两指定日期之间的天数

```
select datediff('2022-05-30', '2022-05-15')
```

**date\_add**

语法：

```
date_add(date/timestamp/string startdate, tinyint/smallint/int days)
```

返回值: 2.1.0版本前返回string，2.1.0版本后返回date

描述：返回开始日期startdate增加天数days后的日期

```
select date_add('2022-05-15', 5);
```

**date\_sub**

语法: date\_sub(date/timestamp/string startdate, tinyint/smallint/int days)

返回值: 2.1.0版本前返回string，2.1.0版本后返回date

描述: 返回开始日期startdate减少天数days后的日期

```
select date_sub('2022-05-15', 5)
```

**add\_months**

语法:

```
add_months(string start_date, int num_months, output_date_format)
```

返回值: string

描述: 返回开始日期start\_date增加num\_months月后的日期，output\_date\_format做为可选参数只在Hive 4.0.0版本后可用。

```
select add_months('2022-05-15', 5)
```

**months\_between**

语法：months\_between(date1, date2)

返回值: double

描述: 返回两日期的间隔月份

```
select months_between('2022-05-15', '2022-01-15')
```

**date\_format**

格式化日期：date\_format函数将字符串或者日期转化为指定格式的日期

```
select date_format('2022-05-29 20:21:22', 'yyyy-MM-dd')
```

## 1 关于月的计算

关于月的计算主要使用trunc()函数的计算

### 1.1上月末

```
select date_sub(trunc(current_date,'MM'),1);
```

### 1.2上月初

```
select trunc(add_months(current_date,-1),'MM')
```

### 1.3本月初

```
select trunc(current_date,'MM')
```

### 1.4 本月末

```
select last_day(current_date());
```

**2 关于周计算**

关于周的计算主要是使用next\_day()函数的计算

使用函数next\_day获取日期下个星期几的日期，参数周一：MO；周二：TU；周三：WE ；周四：TH ；周五：FR ；周六：SA；周日SU

**2.1 本周一**

```
select date_sub(next_day(current_date,'MO'),7) ;
```

### 2.2 本周末

```
select date_sub(next_day(current_date,'MO'),1);
```

### 2.3 上周一

```
select date_sub(next_day(current_date,'MO'),14) ;
```

### 2.4上周末

```
select date_sub(next_day(current_date,'MO'),8) ;
```

### 2.5 根据当前日期得出星期几

主要使用pmod()函数

```
select pmod(datediff(current_date, '2012-01-01'), 7) = 0 --当等于0是当前天为星期日select pmod(datediff(current_date, '2012-01-01'), 7) = 1 --当等于1是当前天为星期1select pmod(datediff(current_date, '2012-01-01'), 7) = 2 --当等于2是当前天为星期2select pmod(datediff(current_date, '2012-01-01'), 7) = 3 --当等于3是当前天为星期3select pmod(datediff(current_date, '2012-01-01'), 7) = 4 --当等于4是当前天为星期4select pmod(datediff(current_date, '2012-01-01'), 7) = 5 --当等于5是当前天为星期5select pmod(datediff(current_date, '2012-01-01'), 7) = 6 --当等于6是当前天为星期6
```

## 3 关于季度计算

### 3.1季度初方法一

利用季度的小公式计算

```
select concat_ws('-', cast(year(current_date) as string), cast(ceil(month(current_date()) / 3) * 3 - 2 as string), '1');
```

**3.2 季度初方法二**

利用quater()函数+case when转换

```
select case quarter(current_date)            when 1 then concat(year(current_date), '-01-01')            when 2 then concat(year(current_date), '-04-01')            when 3 then concat(year(current_date), '-07-01')            when 4 then concat(year(current_date), '-10-01')               end as q_first_day
```

**3.3 获取季度末的方法**

季度算法小公式：

```
ceil(当前月份 / 3 )：获取当前时间所处的季度（Hive高版本可用quater()函数代替）
```

一个季度有3个月：

```
ceil(当前月份 / 3 ) *3：得到当前月份所处的季度末月份ceil(当前月份 / 3 ) *3 -2 :获取当前月份所处的季度初的月份ceil(当前月份 / 3 ) *3  + 1 :获取当前月份所处的季度的下一个季度初的月份
```

当前日期所处的季度末就是:下一个季度月初的时间减去1

```
select date_sub(concat_ws('-', cast(year(current_date) as string), cast(ceil(month(current_date()) / 3) * 3 + 1 as string), '1'),1)
```

## 4 关于年计算

### 4.1 年开始日期

```
select trunc(current_date, 'YYYY')
```

### 4.2 年结束的日期

明年的第一天减去1就是今年的结束日期

```
select date_sub(trunc(add_months(current_date, 12), 'YYYY'), 1)
```

**04**

小结

本文总结了关于Hive中时间函数的使用及时间维度表的生成方法，时间维度表及时间函数在数据开发中经常被用到，这块需要切实掌

**往期精彩**

[SQL进阶技巧：如何取时间序列最新完成状态的前一个状态并将完成状态的过程进行合并？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484979&idx=1&sn=f5a50d0f07d5636ae0f0da3a5a9b2460&chksm=e8e21213df959b0569675b00a9479e0b2ed1a45f642edb6a40939c0aa2a9c8ad634f01ea471f&scene=21#wechat_redirect)

[SQL进阶技巧：如何获取稀疏表字段中最新的值所对应的其他字段值](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484904&idx=1&sn=cac41f19f0a0a55dd14131eda8dcbbf5&chksm=e8e211c8df9598deb04913acf5af1d0b68cd1c17fc0a5b7f8f1b5dc7fe30022c210e6e621615&scene=21#wechat_redirect)

[SQL进阶技巧：如何不使用union all进行行转列？【三种方法实现】](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484545&idx=1&sn=83179a23f1ffea78a17066e91394a66b&chksm=e8e210a1df9599b7fcdec2d96ee3fc0f657921dde2a2ae06a2d7fec395eaf41a804ac6637bfa&scene=21#wechat_redirect)