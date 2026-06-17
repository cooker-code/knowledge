---
title: 字节跳动大数据面试SQL-交叉日期问题
author: 大模型数据的朋友
date: 
url: http://mp.weixin.qq.com/s?__biz=MzUzMjAxNDA5OQ==&mid=2247484515&idx=1&sn=c66dfa7075d36330fd5c1b06e8fe6ac8&chksm=fbea3751399938b576c38e1d4a102f9954bb883305d3c875092d7cd2ffa3c3c21a89e2afa6a2&mpshare=1&scene=24&srcid=0106crJ6cMPjpJlJr7SZgbWr&sharer_shareinfo=222ae0ed4248dbaba12e58c20e1fdd15&sharer_shareinfo_first=222ae0ed4248dbaba12e58c20e1fdd15#rd
---

已知有活动表hall\_table，表中有3个字段：

* hall\_id: 大厅id
* start\_date: 活动开始日期
* end\_date: 活动结束日期

每个大厅可以有多个活动， 

* 请合并在同一个大厅举行的所有重叠的活动，如果两个活动至少有一天相同，那他们就是重叠的。
* 统计每个大厅开展的营销天数，日期如果有重叠需要去重。

测试数据如下所示： 

```
hall_id  start_date  end_date1        2025-01-13  2025-01-201        2025-01-14  2025-01-171        2025-01-14  2025-01-161        2025-01-18  2025-01-251        2025-01-20  2025-01-282        2024-12-09  2024-12-17 2        2024-12-20  2024-12-24  2        2024-12-25  2024-12-30 3        2024-12-01  2025-01-30
```

解决思路：

* 使用开窗函数，获取截止当前行活动的最后日期max\_end\_date；
* 对当前行start\_date和max\_end\_date进行比较，如果

  start\_date <= max\_end\_date，代表有交叉，可以合并；否则不合并；
* 连续问题，可以使用sum() over() 进行分组；
* 对hall\_id+group\_id分组，取每个组内的start\_date的最小值作为活动开始日期，end\_date的最大值作为活动结束日期；

sql如下所示：

```
with  hall_table as (select 1 as hall_id, '2025-01-13' as start_date, '2025-01-20' as end_date union all select 1 as hall_id, '2025-01-14' as start_date, '2025-01-17' as end_date union all select 1 as hall_id, '2025-01-14' as start_date, '2025-01-16' as end_date union all select 1 as hall_id, '2025-01-18' as start_date, '2025-01-25' as end_date union all select 1 as hall_id, '2025-01-20' as start_date, '2025-01-28' as end_date union all select 2 as hall_id, '2024-12-09' as start_date, '2024-12-17' as end_date union all select 2 as hall_id, '2024-12-20' as start_date, '2024-12-24' as end_date union all select 2 as hall_id, '2024-12-25' as start_date, '2024-12-30' as end_date union all select 3 as hall_id, '2024-12-01' as start_date, '2025-01-30' as end_date), --使用max()函数开窗，获得截止到当前行之前的活动最后日期hall_max_dt as (select    hall_id,    start_date,    end_date,    max(end_date) over (partition by hall_id order by start_date asc,end_date asc rows between unbounded preceding and 1 preceding) as  max_end_date --不包含当前行from hall_table), --对当前行的start_date 和截止到上一行的最大end_date即作max_end_date进行比较--如果当前行的start_date 小于等于max_end_date 代表有交叉，可以合并,否则不合并merge_table as (select    hall_id,   start_date,   end_date,   max_end_date,   if(start_date <= max_end_date, 0, 1) as is_merge, --0:合并，1:不合并   case when max_end_date is null then start_date         when (start_date > max_end_date) then start_date         else date_add(max_end_date, 1) end as new_start_datefrom  hall_max_dt),--连续问题max_merge as (select      hall_id,     start_date,     end_date,     max_end_date,     new_start_date,     is_merge,     sum(is_merge) over (partition by hall_id order by start_date asc,end_date asc) as group_idfrom merge_table)select hall_id,       min(start_date) as start_date,       max(end_date)  as end_datefrom  max_mergegroup by hall_id, group_id;
```

中间结果：

```
select * from max_merge;hall_id start_date  new_start_date  end_date    max_end_date    is_merge1       2025-01-13  2025-01-13      2025-01-20  null              11       2025-01-14  2025-01-21      2025-01-16  2025-01-20        01       2025-01-14  2025-01-21      2025-01-17  2025-01-20        01       2025-01-18  2025-01-21      2025-01-25  2025-01-20        01       2025-01-20  2025-01-26      2025-01-28  2025-01-25        02       2024-12-09  2024-12-09      2024-12-17  null              12       2024-12-20  2024-12-20      2024-12-24  2024-12-17        12       2024-12-25  2024-12-25      2024-12-30  2024-12-24        13       2024-12-01  2024-12-01      2025-01-30  null              1
```

结果如下所示：

```
hall_id start_date  end_date  1     2025-01-13  2025-01-28  2     2024-12-09  2024-12-17  2     2024-12-20  2024-12-24  2     2024-12-25  2024-12-30  3     2024-12-01  2025-01-30
```

每个大厅的活动天数：

活动上一个日期区间A与当前活动日期区间B出行重叠，日期交叉有重复数据，需要将区间B的起始时间改为区间A的结束时间，需要保证区间B的结束日期 > 开始时间。具体sql如下：

```
select     hall_id,    sum((datediff(end_date,new_start_date)+ 1)) as day_cnt from  max_mergewhere new_start_date <= end_dategroup by hall_id;
```

结果如下：

```
hall_id day_cnt  1       16  2       20  3       61
```

总结：

可以看之前发表的这篇对于交叉问题的通用解法，大家可以多总结其中的规律；在新的一年里，希望你每周都有进步，每周可以对自己来个复盘，加油！

[大数据经典面试SQL-日期交叉重叠问题](https://mp.weixin.qq.com/s?__biz=MzUzMjAxNDA5OQ==&mid=2247484257&idx=1&sn=ec2153d30d8db66100663db4189293d7&scene=21#wechat_redirect)

附群交流二维码：

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