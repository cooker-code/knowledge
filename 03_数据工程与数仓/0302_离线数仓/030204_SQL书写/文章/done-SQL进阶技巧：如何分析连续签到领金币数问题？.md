---
title: SQL进阶技巧：如何分析连续签到领金币数问题？
author: 会飞的一十六
date:
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247489061&idx=1&sn=e972f29f2e87e3c8a4765365a0bb74ec&chksm=e924bb0f08057950238032ecad2ff18aaa75f517619a55e7485937033054cbed99ed4cdaaa77&mpshare=1&scene=24&srcid=0312MRCxVzTIUaG2eaQ68mIj&sharer_shareinfo=c64967cba40c7f31085da90e5b3c228e&sharer_shareinfo_first=c64967cba40c7f31085da90e5b3c228e#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL窗口滚动与序列问题|SQL窗口滚动与序列问题]]


点击上方蓝字关注我们

01

# 问题描述

用户每天签到可以领1金币，并可以累计签到天数，连续签到的第3、7天分别可以额外领2和6金币。

每连续签到7天重新累积签到天数。

从用户登录明细表中求出每个用户金币总数，并按照金币总数倒序排序

结果如下：

|  |  |
| --- | --- |
| ****User\_id（用户id）**** | ****Sum\_coin\_cn（金币总数)**** |
| ****101**** | 7 |
| ****109**** | 3 |
| ****107**** | 3 |
| ****102**** | 3 |
| ****106**** | 2 |
| ****104**** | 2 |
| ****103**** | 2 |
| ****1010**** | 2 |
| ****108**** | 1 |
| ****105**** | 1 |

02

# 数据准备

****1）表结构****

********2********）建表语句********

```
DROP TABLE IF EXISTS user_login_detail;
CREATE TABLE user_login_detail
(
`user_id` string comment '用户id',
`ip_address` string comment 'ip地址',
`login_ts` string comment '登录时间',
`logout_ts` string comment '登出时间'
) COMMENT '用户登录明细表'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
```

**********3********）数据装载**********

```
INSERT overwrite table user_login_detail
VALUES ('101', '180.149.130.161', '2021-09-21 08:00:00', '2021-09-27 08:30:00'),
       ('101', '180.149.130.161', '2021-09-27 08:00:00', '2021-09-27 08:30:00'),
       ('101', '180.149.130.161', '2021-09-28 09:00:00', '2021-09-28 09:10:00'),
       ('101', '180.149.130.161', '2021-09-29 13:30:00', '2021-09-29 13:50:00'),
       ('101', '180.149.130.161', '2021-09-30 20:00:00', '2021-09-30 20:10:00'),
       ('102', '120.245.11.2', '2021-09-22 09:00:00', '2021-09-27 09:30:00'),
       ('102', '120.245.11.2', '2021-10-01 08:00:00', '2021-10-01 08:30:00'),
       ('102', '180.149.130.174', '2021-10-01 07:50:00', '2021-10-01 08:20:00'),
       ('102', '120.245.11.2', '2021-10-02 08:00:00', '2021-10-02 08:30:00'),
       ('103', '27.184.97.3', '2021-09-23 10:00:00', '2021-09-27 10:30:00'),
       ('103', '27.184.97.3', '2021-10-03 07:50:00', '2021-10-03 09:20:00'),
       ('104', '27.184.97.34', '2021-09-24 11:00:00', '2021-09-27 11:30:00'),
       ('104', '27.184.97.34', '2021-10-03 07:50:00', '2021-10-03 08:20:00'),
       ('104', '27.184.97.34', '2021-10-03 08:50:00', '2021-10-03 10:20:00'),
       ('104', '120.245.11.89', '2021-10-03 08:40:00', '2021-10-03 10:30:00'),
       ('105', '119.180.192.212', '2021-10-04 09:10:00', '2021-10-04 09:30:00'),
       ('106', '119.180.192.66', '2021-10-04 08:40:00', '2021-10-04 10:30:00'),
       ('106', '119.180.192.66', '2021-10-05 21:50:00', '2021-10-05 22:40:00'),
       ('107', '219.134.104.7', '2021-09-25 12:00:00', '2021-09-27 12:30:00'),
       ('107', '219.134.104.7', '2021-10-05 22:00:00', '2021-10-05 23:00:00'),
       ('107', '219.134.104.7', '2021-10-06 09:10:00', '2021-10-06 10:20:00'),
       ('107', '27.184.97.46', '2021-10-06 09:00:00', '2021-10-06 10:00:00'),
       ('108', '101.227.131.22', '2021-10-06 09:00:00', '2021-10-06 10:00:00'),
       ('108', '101.227.131.22', '2021-10-06 22:00:00', '2021-10-06 23:00:00'),
       ('109', '101.227.131.29', '2021-09-26 13:00:00', '2021-09-27 13:30:00'),
       ('109', '101.227.131.29', '2021-10-06 08:50:00', '2021-10-06 10:20:00'),
       ('109', '101.227.131.29', '2021-10-08 09:00:00', '2021-10-08 09:10:00'),
       ('1010', '119.180.192.10', '2021-09-27 14:00:00', '2021-09-27 14:30:00'),
       ('1010', '119.180.192.10', '2021-10-09 08:50:00', '2021-10-09 10:20:00');
```

03

# 问题分析

01.

# 代码实现

```
-- 求连续并标志是连续的第几天select  t1.user_id,  t1.login_date,  date_sub(t1.login_date,t1.rk) login_date_rk,  count(*)over(partition by t1.user_id, date_sub(t1.login_date,t1.rk) order by t1.login_date) counti_cnfrom  (   select     user_id,     date_format(login_ts,'yyyy-MM-dd') login_date,     rank()over(partition by user_id order by date_format(login_ts,'yyyy-MM-dd')) rk   from     user_login_detail   group by     user_id,date_format(login_ts,'yyyy-MM-dd'))t1
--求出金币数量，以及签到奖励的金币数量select  t2.user_id,  max(t2.counti_cn)+sum(if(t2.counti_cn%3=0,2,0))+sum(if(t2.counti_cn%7=0,6,0)) coin_cnfrom  (select  t1.user_id,     t1.login_date,     date_sub(t1.login_date,t1.rk) login_date_rk,     count(*)over(partition by t1.user_id, date_sub(t1.login_date,t1.rk) order by t1.login_date) counti_cn   from     (      select        user_id,        date_format(login_ts,'yyyy-MM-dd') login_date,        rank()over(partition by user_id order by date_format(login_ts,'yyyy-MM-dd')) rk      from        user_login_detail      group by        user_id,date_format(login_ts,'yyyy-MM-dd')   )t1)t2group by  t2.user_id,t2.login_date_rk
-- 求出每个用户的金币总数
select  t3.user_id,  sum(t3.coin_cn) sum_coin_cnfrom   (    select      t2.user_id,      max(t2.counti_cn)+sum(if(t2.counti_cn%3=0,2,0))+sum(if(t2.counti_cn%7=0,6,0)) coin_cn    from      (    select      t1.user_id,         t1.login_date,         date_sub(t1.login_date,t1.rk) login_date_rk,         count(*)over(partition by t1.user_id, date_sub(t1.login_date,t1.rk) order by t1.login_date) counti_cn       from         (          select            user_id,            date_format(login_ts,'yyyy-MM-dd') login_date,            rank()over(partition by user_id order by date_format(login_ts,'yyyy-MM-dd')) rk          from            user_login_detail          group by            user_id,date_format(login_ts,'yyyy-MM-dd')       )t1    )t2    group by      t2.user_id,t2.login_date_rk    )t3group by  t3.user_idorder by  sum_coin_cn desc
```

02.

# 代码功能分析

第一段SQL功能：

找出用户连续登录的信息，包括用户 ID、登录日期、经过调整的登录日期（通过 date\_sub(t1.login\_date,t1.rk)）以及连续登录的天数。

实现方式：

首先在子查询 t1 中：

使用 date\_format(login\_ts,'yyyy-MM-dd') 将 login\_ts 转换为 yyyy-MM-dd 格式的日期，命名为 login\_date。

使用 rank() 窗口函数，按照 user\_id 分区，按照 login\_date 排序，为每个用户的登录日期分配一个排名 rk。

对 user\_id 和 login\_date 进行分组，以确保数据唯一。

主查询中：

使用 date\_sub(t1.login\_date,t1.rk) 计算出一个新的日期 login\_date\_rk，用于判断用户登录是否连续。

使用 count(\*) over(partition by t1.user\_id, date\_sub(t1.login\_date,t1.rk) order by t1.login\_date) 窗口函数，按照 user\_id 和 login\_date\_rk 分区，按照 login\_date 排序，计算连续登录的天数 counti\_cn。

第二段 SQL功能：

功能：

计算用户可以获得的金币数量，根据连续登录天数计算签到奖励的金币数量。

实现方式：

将第一段 SQL 的查询结果作为子查询 t2。

主查询中：

使用 max(t2.counti\_cn) 取每个用户的最大连续登录天数。

使用 sum(if(t2.counti\_cn%3=0,2,0)) 当连续登录天数是 3 的倍数时，给予 2 个金币奖励。

使用 sum(if(t2.counti\_cn%7=0,6,0)) 当连续登录天数是 7 的倍数时，给予 6 个金币奖励。

计算 coin\_cn 作为用户的金币数量。

第三段 SQL功能：

* 功能：

  计算每个用户的金币总数。

  实现方式：

  将第二段 SQL 的查询结果作为子查询 t3。

  主查询中：

  使用 sum(t3.coin\_cn) 计算每个用户的金币总数 sum\_coin\_cn。

  按照 user\_id 分组并按照 sum\_coin\_cn 降序排序。

  整个 SQL 代码通过窗口函数和日期计算，将用户登录数据进行了复杂的分析和处理。

  首先使用 RANK() 函数对用户的登录日期进行排序和排名，为后续计算连续登录提供基础。

  通过 DATE\_SUB() 函数结合排名，找出连续登录的序列，并计算连续登录天数。

  然后根据连续登录天数的规则，使用 IF() 函数和 SUM() 函数添加相应的金币奖励。

  最后对用户的金币数进行求和并排序，以展示用户的金币总数排名。

04

# 小结

3.1. 窗口函数

RANK () 函数：

在 login\_rank 子查询中使用 RANK() OVER (PARTITION BY user\_id ORDER BY DATE\_FORMAT(login\_ts,'yyyy-MM-dd')) AS rk。

核心功能：为每个用户的登录日期生成排名，按用户 ID 进行分区，按登录日期进行排序。这有助于将用户的登录记录按照时间顺序编号，为后续判断登录是否连续提供依据。

重要性：窗口函数是 SQL 中的高级功能，能方便地进行数据分组内的排序和排名，在处理复杂的分析需求时非常有用，比如对用户的连续操作进行排名，以便后续计算连续操作的天数或次数。

3.2. 日期处理

DATE\_FORMAT 函数：

在 login\_rank 子查询中使用 DATE\_FORMAT(login\_ts,'yyyy-MM-dd') AS login\_date。

核心功能：将日期时间类型的数据（如 login\_ts）转换为特定的日期格式（如 yyyy-MM-dd），以便进行按天的分组和排序操作。

重要性：在处理时间序列数据时，将日期时间精确到所需的粒度，方便进行时间范围的筛选、分组和比较，确保日期比较的一致性和准确性。

3. 3 DATE\_SUB 函数与连续数据的判断

在子查询中使用 DATE\_SUB(lr.login\_date, lr.rk) AS login\_date\_rk。

核心功能：通过将日期减去排名，找出连续登录的日期序列。对于连续的登录记录，这个操作会产生相同的 login\_date\_rk 值，因为连续的日期减去递增的排名会得到一个相同的结果序列。

重要性：这是一种判断连续数据的巧妙方法，可用于分析用户的连续行为（如连续登录、连续操作等），适用于多种业务场景，如用户行为分析、设备连续运行时间统计等。

3.4. 条件求和

SUM 和 IF 函数的组合：

在 coin\_reward 子查询中使用 SUM(IF(cl.counti\_cn % 3 = 0, 2, 0)) 和 SUM(IF(cl.counti\_cn % 7 = 0, 6, 0))。

核心功能：根据条件对满足条件的数据进行求和，如根据连续登录天数是否为 3 或 7 的倍数添加不同的奖励金币。

重要性：在需要根据数据的不同情况添加不同权重或奖励时，这种组合可以灵活地根据数据条件计算出所需的结果，是实现复杂业务逻辑计算的常用技巧，如根据不同销售额计算不同提成、根据不同的用户行为给予不同积分等。

这些核心技巧相互配合，实现了从用户登录信息的分析到用户金币奖励的计算。通过使用窗口函数对用户登录记录进行排名，结合日期处理函数和 DATE\_SUB 函数判断连续登录行为，使用条件求和根据不同的连续登录天数计算金币奖励，再通过分组和聚合函数计算用户的总金币数。这些技巧在不同的业务场景中可以灵活应用，通过调整函数的使用方式和条件判断，可以处理如用户行为分析、业务指标计算、用户激励机制等各种业务需求。

# 往期回顾

SQL进阶技巧：如何计算商品需求与到货队列表进出计划？

[SQL进阶技巧：如何实现分钟级的趋势图？](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247488062&idx=1&sn=ad80e8e01c048077a6d43d75080ec7aa&scene=21#wechat_redirect)

[SQL进阶技巧：非等值连接--单向近距离匹配](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247487955&idx=1&sn=1f551653cf9d25490310b5b14edd7de9&scene=21#wechat_redirect)

END

###### 公众号：会飞一十六

扫码关注 了解更多内容

点个在看，你最好看

点击“阅读原文”获取更多精彩内容~~