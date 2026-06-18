---
title: 常见大数据面试SQL-物流线路分析SQL
author: 数据仓库技术
date:
url: http://mp.weixin.qq.com/s?__biz=MzAwODgxMjc3OA==&mid=2247484815&idx=1&sn=69dbec313717793d0aa5ded430c0e8fb&chksm=9aa3b270390eb1825e229bdfd52415eb85de5af1fc722d9a186019928bd6db1fe095f2bcc13d&mpshare=1&scene=24&srcid=1106VROy73r8AedTlz85EAND&sharer_shareinfo=242b9b6e30dcce37f3d26459e25ab26e&sharer_shareinfo_first=242b9b6e30dcce37f3d26459e25ab26e#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL窗口滚动与序列问题|SQL窗口滚动与序列问题]]


## 一、题目

有t20\_logistics\_route(物流线路表)

* route\_id（线路ID）: 线路的唯一标识符，BIGINT
* route name(线路名称): 物流线路的名称，STRING

t20\_package\_info(包裹信息)表:

* package id (包裹 ID);包裹的唯一标识符，BIGINT
* route id(线路ID):关联物流线路表的线路 ID，BIGINT
* package\_type (包裹类型):包裹的类型，STRING(大型和小型)
* start\_time(包裹配送时间):包裹开始配送的时间，DATETIMME
* delivery\_time(送达时间): 包裹送达的时间，DATETIMME

t20\_lost\_package(丢失包裹)表:

* lost\_id(丢失 ID);丢失的唯一标识符，BIGINT
* package\_id(包裹ID):关联包裹信息表的包裹 ID，BIGINT

  【要求】:

  查询出每条物流线路在每个季节的平均送达时间（单位小时,分钟部分以小数展示），以及每条物流线路全年包裹丢失率（丢失包裹数量/总包裹数量）,每条物流线路运输包裹类型占比，结果按照线路ID升序

+ route\_id:线路ID
+ route\_name:线路名称
+ spring\_average\_delivery\_time:春季该线路平均送达时间：delivery\_time-start\_time。春季3月、4月、5月（round保留2位小数）
+ summer\_average\_delivery\_time:夏季该线路平均送达时间：delivery\_time-start\_time。夏季6月、7月、8月（round保留2位小数）
+ autumn\_average\_delivery\_time:秋季该线路平均送达时间：delivery\_time-start\_time。秋季9月、10月、11月（round保留2位小数）
+ winter\_average\_delivery\_time:冬季该线路平均送达时间：delivery\_time-start\_time。夏季12月、1月、2月（round保留2位小数）
+ lost\_count:该线路全年包裹丢失总量
+ lost\_rate:该线路全年包裹丢失率（round保留2位小数）
+ small\_packages:该线路运输“小型”包裹类型占比 “小型”包裹数量/总包裹数量（round保留2位小数）
+ big\_packages:该线路运输“大型”包裹类型占比 “大型”包裹数量/总包裹数量（round保留2位小数）

**样例数据**

**t20\_logistics\_route(物流线路)表**

```
+-----------+-------------+
| route_id  | route_name  |
+-----------+-------------+
| 1         | 线路 A        |
| 2         | 线路 B        |
| 3         | 线路 C        |
| 4         | 线路 D        |
+-----------+-------------+
```

**t20\_lost\_package(包裹信息)表**

```
+-------------+-----------+---------------+----------------------+----------------------+
| package_id  | route_id  | package_type  |      start_time      |    delivery_time     |
+-------------+-----------+---------------+----------------------+----------------------+
| 1           | 1         | 小型            | 2024-01-04 09:10:00  | 2024-01-05 10:00:00  |
| 2           | 2         | 大型            | 2024-01-09 11:05:00  | 2024-01-10 12:00:00  |
| 3           | 3         | 小型            | 2024-02-09 11:30:00  | 2024-02-10 12:00:00  |
| 4           | 4         | 大型            | 2024-03-09 11:15:00  | 2024-03-10 12:00:00  |
| 5           | 1         | 小型            | 2024-04-09 10:50:00  | 2024-04-10 12:00:00  |
| 6           | 2         | 小型            | 2024-05-09 11:20:00  | 2024-05-10 12:00:00  |
| 7           | 3         | 大型            | 2024-06-09 14:30:00  | 2024-06-10 15:00:00  |
| 8           | 4         | 小型            | 2024-07-09 11:06:00  | 2024-07-10 12:00:00  |
+-------------+-----------+---------------+----------------------+----------------------+
```

**t20\_logistics\_route(丢失包裹)表**

```
+----------+-------------+
| lost_id  | package_id  |
+----------+-------------+
| 1        | 1           |
| 2        | 6           |
+----------+-------------+
```

\*\* 期望结果\*\*

## 二、分析

该题目难度不高，但是计算起来比较麻烦，但是实际业务中这样的加工相对常见。面试遇到这样的题目，说明团队相对比较务实，但是日常工作可能也是类似繁琐内容较多。在面试过程中，从内容理解和解题上，都属于内容量比较多的。

| 维度 | 评分 |
| --- | --- |
| 题目难度 | ⭐️⭐️⭐️ |
| 题目清晰度 | ⭐️⭐️⭐️⭐️ |
| 业务常见度 | ⭐️⭐️⭐️⭐️ |

## 三、SQL

### 1.明细数据关联

我们根据粒度发现，最细粒度的数据是包裹数据，所以将包裹表作为主表，连接物流线路表（这里join和left join都可以，因为有包裹一定有线路），左连接丢失包裹表数据。

**执行SQL**

```
select *
from t20_package_info t1
         join t20_logistics_route t2
              on t1.route_id = t2.route_id
         left join t20_lost_package t3
                   on t1.package_id = t3.package_id；
```

**SQL结果**

```
+-------------+-----------+---------------+----------------------+----------------------+-----------+-------------+----------+-------------+
| package_id  | route_id  | package_type  |      start_time      |    delivery_time     | route_id  | route_name  | lost_id  | package_id  |
+-------------+-----------+---------------+----------------------+----------------------+-----------+-------------+----------+-------------+
| 1           | 1         | 小型            | 2024-01-04 09:10:00  | 2024-01-05 10:00:00  | 1         | 线路 A        | 1        | 1           |
| 2           | 2         | 大型            | 2024-01-09 11:05:00  | 2024-01-10 12:00:00  | 2         | 线路 B        | NULL     | NULL        |
| 3           | 3         | 小型            | 2024-02-09 11:30:00  | 2024-02-10 12:00:00  | 3         | 线路 C        | NULL     | NULL        |
| 4           | 4         | 大型            | 2024-03-09 11:15:00  | 2024-03-10 12:00:00  | 4         | 线路 D        | NULL     | NULL        |
| 5           | 1         | 小型            | 2024-04-09 10:50:00  | 2024-04-10 12:00:00  | 1         | 线路 A        | NULL     | NULL        |
| 6           | 2         | 小型            | 2024-05-09 11:20:00  | 2024-05-10 12:00:00  | 2         | 线路 B        | 2        | 6           |
| 7           | 3         | 大型            | 2024-06-09 14:30:00  | 2024-06-10 15:00:00  | 3         | 线路 C        | NULL     | NULL        |
| 8           | 4         | 小型            | 2024-07-09 11:06:00  | 2024-07-10 12:00:00  | 4         | 线路 D        | NULL     | NULL        |
+-------------+-----------+---------------+----------------------+----------------------+-----------+-------------+----------+-------------+
```

### 2.计算分季节的送到时间

由于题目中没有明确出季节划分是按照包裹开始配送时间还是送达时间，这里我们就直接使用start\_time来计算。 题目是计算每条线路数据，所以粒度是线路。

**执行SQL**

```
with t as (select t1.package_id,
                  t1.package_type,
                  t1.start_time,
                  t1.delivery_time,
                  t2.route_id,
                  t2.route_name,
                  t3.lost_id
           from t20_package_info t1
                    join t20_logistics_route t2
                         on t1.route_id = t2.route_id
                    left join t20_lost_package t3
                              on t1.package_id = t3.package_id)
select route_id,
       route_name,
       round(avg(case
                     when month(to_date(start_time)) in (3, 4, 5)
                         then (unix_timestamp(delivery_time) - unix_timestamp(start_time)) / 3600 end),
             2)                                                                                            as spring_avg_time,
       round(avg(case
                     when month(to_date(start_time)) in (6, 7, 8)
                         then (unix_timestamp(delivery_time) - unix_timestamp(start_time)) / 3600 end),
             2)                                                                                            as summer_avg_time,
       round(avg(case
                     when month(to_date(start_time)) in (9, 10, 11)
                         then (unix_timestamp(delivery_time) - unix_timestamp(start_time)) / 3600 end),
             2)                                                                                            as autumn_avg_time,
       round(avg(case
                     when month(to_date(start_time)) in (12, 1, 2)
                         then (unix_timestamp(delivery_time) - unix_timestamp(start_time)) / 3600 end),
             2)                                                                                            as winter_avg_time
from t
group by route_id,
         route_name,
         order by route_id asc
```

**SQL结果**

```
+-----------+-------------+------------------+------------------+------------------+------------------+
| route_id  | route_name  | spring_avg_time  | summer_avg_time  | autumn_avg_time  | winter_avg_time  |
+-----------+-------------+------------------+------------------+------------------+------------------+
| 1         | 线路 A        | 25.17            | NULL             | NULL             | 24.83            |
| 2         | 线路 B        | 24.67            | NULL             | NULL             | 24.92            |
| 3         | 线路 C        | NULL             | 24.5             | NULL             | 24.5             |
| 4         | 线路 D        | 24.75            | 24.9             | NULL             | NULL             |
+-----------+-------------+------------------+------------------+------------------+------------------+
```

### 3.最终结果

添加其他特征，得到最终结果

**执行SQL**

```
with t as (select t1.package_id,
                  t1.package_type,
                  t1.start_time,
                  t1.delivery_time,
                  t2.route_id,
                  t2.route_name,
                  t3.lost_id
           from t20_package_info t1
                    join t20_logistics_route t2
                         on t1.route_id = t2.route_id
                    left join t20_lost_package t3
                              on t1.package_id = t3.package_id)
select route_id,
       route_name,
       round(avg(case
                     when month(to_date(start_time)) in (3, 4, 5)
                         then (unix_timestamp(delivery_time) - unix_timestamp(start_time)) / 3600 end),
             2)                                                                        as spring_avg_time,
       round(avg(case
                     when month(to_date(start_time)) in (6, 7, 8)
                         then (unix_timestamp(delivery_time) - unix_timestamp(start_time)) / 3600 end),
             2)                                                                        as summer_avg_time,
       round(avg(case
                     when month(to_date(start_time)) in (9, 10, 11)
                         then (unix_timestamp(delivery_time) - unix_timestamp(start_time)) / 3600 end),
             2)                                                                        as autumn_avg_time,
       round(avg(case
                     when month(to_date(start_time)) in (12, 1, 2)
                         then (unix_timestamp(delivery_time) - unix_timestamp(start_time)) / 3600 end),
             2)                                                                        as winter_avg_time,
       count(lost_id)                                                                  as lost_count,    --丢失包裹数量
       round(count(lost_id) / count(1), 2)                                             as lost_rate,     --包裹丢失率
       round(count(case when package_type = '小型' then package_id end) / count(1), 2) as small_packages,--小型包裹率
       round(count(case when package_type = '大型' then package_id end) / count(1), 2) as big_packages--大型包裹率
from t
group by route_id,
         route_name,
         order by route_id asc
```

**SQL结果**

```
+-----------+-------------+------------------+------------------+------------------+------------------+-------------+------------+-----------------+---------------+
| route_id  | route_name  | spring_avg_time  | summer_avg_time  | autumn_avg_time  | winter_avg_time  | lost_count  | lost_rate  | small_packages  | big_packages  |
+-----------+-------------+------------------+------------------+------------------+------------------+-------------+------------+-----------------+---------------+
| 1         | 线路 A        | 25.17            | NULL             | NULL             | 24.83            | 1           | 0.5        | 1.0             | 0.0           |
| 2         | 线路 B        | 24.67            | NULL             | NULL             | 24.92            | 1           | 0.5        | 0.5             | 0.5           |
| 3         | 线路 C        | NULL             | 24.5             | NULL             | 24.5             | 0           | 0.0        | 0.5             | 0.5           |
| 4         | 线路 D        | 24.75            | 24.9             | NULL             | NULL             | 0           | 0.0        | 0.5             | 0.5           |
+-----------+-------------+------------------+------------------+------------------+------------------+-------------+------------+-----------------+---------------+
4 rows selected (0.588 seconds)
```

## 四、建表语句和数据插入

```
--建表语句
--物流线路表
CREATE TABLE IF NOT EXISTS t20_logistics_route
(
    route_id    bigint comment '线路id',
    route_name  string comment '线路名称'
)
    COMMENT '物流线路表';
--包裹信息表
CREATE TABLE IF NOT EXISTS t20_package_info
(
    package_id      bigint comment  '包裹id',
    route_id        bigint comment  '线路id',
    package_type    string comment  '包裹类型',
    start_time      string comment  '开始配送时间',
    delivery_time   string comment  '包裹送达时间'
)
    COMMENT '包裹信息表';
--丢失包裹表
 CREATE TABLE IF NOT EXISTS t20_lost_package
(
    lost_id      bigint comment  '丢失id',
    package_id        bigint comment  '包裹id'
)
    COMMENT '丢失包裹表';

--插入数据
--物流线路表数据
INSERT INTO t20_logistics_route (route_id, route_name)
VALUES 
(1,'线路 A'),
(2,'线路 B'),
(3,'线路 C'),
(4,'线路 D');

--包裹信息表数据
INSERT INTO t20_package_info (package_id,route_id,package_type,start_time,delivery_time)
VALUES
(1,1,'小型','2024-01-04 09:10:00','2024-01-05 10:00:00'),
(2,2,'大型','2024-01-09 11:05:00','2024-01-10 12:00:00'),
(3,3,'小型','2024-02-09 11:30:00','2024-02-10 12:00:00'),
(4,4,'大型','2024-03-09 11:15:00','2024-03-10 12:00:00'),
(5,1,'小型','2024-04-09 10:50:00','2024-04-10 12:00:00'),
(6,2,'小型','2024-05-09 11:20:00','2024-05-10 12:00:00'),
(7,3,'大型','2024-06-09 14:30:00','2024-06-10 15:00:00'),
(8,4,'小型','2024-07-09 11:06:00','2024-07-10 12:00:00');

--丢失包裹表数据
INSERT INTO t20_lost_package (lost_id,package_id)
VALUES
(1,1),
(2,6);
```

> 本文同步在微信公众号”数据仓库技术“和个人博客”数据仓库技术“发表。原文:www.dwsql.com 同时有“数据仓库技术”社群以及有几十位小伙伴一起讨论数据仓库相关技术，欢迎你的加入，社群免费。