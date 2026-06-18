> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL窗口滚动与序列问题|SQL窗口滚动与序列问题]]
---
title: 字节面试SQL-去掉最大最小值部门平均薪水【仅一次开窗】
author: 数据仓库技术
date: 数仓技术晨曦数仓技术晨曦
url: https://mp.weixin.qq.com/s?__biz=MzAwODgxMjc3OA==&mid=2247485093&idx=1&sn=5274f1e370ac9d8d3a5ba9f0d4d6ea02&chksm=9ac48a3d3f48efb238f4a70946ec357af6a70a5177c659c94f6a5faadf0c820ebd139ea515a0&mpshare=1&scene=24&srcid=0604wBeVKsqpzlciIM7uSiTz&sharer_shareinfo=219d4591220c95864bd81ac53937fbda&sharer_shareinfo_first=219d4591220c95864bd81ac53937fbda#rd
---

## 一、题目

有员工薪资表t8\_salary,包含员工ID(emp\_id)，部门ID(depart\_id)，薪水(salary),请计算去除最高最低薪资后的平均薪水；（每个部门员工数不少于5人）

**仅用一个开窗函数**

```
+---------+------------+-----------+
| emp_id  | depart_id  |  salary   |
+---------+------------+-----------+
| 1001    | 1          | 5000.00   |
| 1002    | 1          | 10000.00  |
| 1003    | 1          | 20000.00  |
| 1004    | 1          | 30000.00  |
| 1005    | 1          | 6000.00   |
| 1006    | 1          | 10000.00  |
| 1007    | 1          | 11000.00  |
| 1008    | 2          | 3000.00   |
| 1009    | 2          | 7000.00   |
| 1010    | 2          | 9000.00   |
| 1011    | 2          | 30000.00  |
| 1012    | 2          | 31000.00  |
+---------+------------+-----------+
```

## 二、分析

这个题目我们写过很多了，下面列出历史题目，大家可以复习。 但是本次题目要求仅能使用一次开窗函数。在这个要求上，难度就直接提升了。在这个要求下，我不介意给其难度提升到5星。

[HiveSQL-面试题026 去掉最大最小值的部门平均薪水](https://mp.weixin.qq.com/s?__biz=MzAwODgxMjc3OA==&mid=2247484035&idx=1&sn=0d1e72221a8cdf4f7d988a1f56c0cfcb&scene=21#wechat_redirect)

[小红书大数据面试SQL-查询每个用户的第一条和最后一条记录](https://mp.weixin.qq.com/s?__biz=MzAwODgxMjc3OA==&mid=2247484626&idx=1&sn=456cb0bb3fa60031f52bae34c5d60f7c&scene=21#wechat_redirect)

[常见大数据面试SQL-查询前2大和前2小用户并有序拼接](https://mp.weixin.qq.com/s?__biz=MzAwODgxMjc3OA==&mid=2247484647&idx=1&sn=ea92ad673182a910d7092d37861e1ad8&scene=21#wechat_redirect)

| 维度 | 评分 |
| --- | --- |
| 题目难度 | ⭐️⭐️⭐️⭐️⭐️ |
| 题目清晰度 | ⭐️⭐️⭐️⭐️⭐️ |
| 业务常见度 | ⭐️⭐️⭐️⭐️ |

## 三、SQL

### 1.方案一：（最大最小值不能存在重复，否则结果错误）

1. 方案一是群里小伙伴给出的方案，该方案很巧妙，但是仅能处理去掉最大最小的数据，如果把最大最小调整到去掉最高2个和最低2个时就无法使用了。 涉及函数：percent\_rank，percent\_rank() - 计算一个值在一组值中的百分比排名

**执行SQL**

```
select emp_id,
       depart_id,
       salary,
       percent_rank() over (partition by depart_id order by salary) as prn
from t8_salary
```

**查询结果**

```
+---------+------------+-----------+----------------------+
| emp_id  | depart_id  |  salary   |         prn          |
+---------+------------+-----------+----------------------+
| 1001    | 1          | 5000.00   | 0.0                  |
| 1005    | 1          | 6000.00   | 0.16666666666666666  |
| 1002    | 1          | 10000.00  | 0.3333333333333333   |
| 1006    | 1          | 10000.00  | 0.3333333333333333   |
| 1007    | 1          | 11000.00  | 0.6666666666666666   |
| 1003    | 1          | 20000.00  | 0.8333333333333334   |
| 1004    | 1          | 30000.00  | 1.0                  |
| 1008    | 2          | 3000.00   | 0.0                  |
| 1009    | 2          | 7000.00   | 0.25                 |
| 1010    | 2          | 9000.00   | 0.5                  |
| 1011    | 2          | 30000.00  | 0.75                 |
| 1012    | 2          | 31000.00  | 1.0                  |
+---------+------------+-----------+----------------------+
```

2. 在这里限定prn不等于0和1，然后计算部门平均薪水即可

**执行SQL**

```
select depart_id,
       avg(salary) as avg_depart_salary
from (select emp_id,
             depart_id,
             salary,
             percent_rank() over (partition by depart_id order by salary) as prn
      from t8_salary) t
where t.prn not in (0, 1)
group by depart_id
```

**查询结果**

```
+------------+--------------------+
| depart_id  | avg_depart_salary  |
+------------+--------------------+
| 1          | 11400.000000       |
| 2          | 15333.333333       |
+------------+--------------------+
2 rows selected (0.356 seconds)
```

### 2.方案二

该方案通过对窗口函数的大小、位置以及对分组排序后开始和结尾处窗口覆盖范围的应用，来处理。这也是这个题目的亮点，定向严格的考察了对开窗函数的理解；

1. 先通过count开窗，按照部门分组，窗口大小为 当前行的前一行到当前行的后一行

**执行SQL**

```
select emp_id,
       depart_id,
       salary,
       count(1) over (partition by depart_id order by salary rows between 1 preceding and 1 following) as row_cnt
from t8_salary
```

**查询结果**

图一我给圈出了第一行、第二行、第三行数据的窗口范围；

观察图二，我们可以看到首行末行的统计结果；

2. 限定行数row\_cnt = 3 即可得到去掉最大最小记录的数据，计算平均值即可

**执行SQL**

```
select depart_id,
       avg(salary) as avg_depart_salary
from (select emp_id,
             depart_id,
             salary,
             count(1) over (partition by depart_id order by salary rows between 1 preceding and 1 following) as row_cnt
      from t8_salary)
where row_cnt = 3
group by depart_id
```

**查询结果**

```
+------------+--------------------+
| depart_id  | avg_depart_salary  |
+------------+--------------------+
| 1          | 11400.000000       |
| 2          | 15333.333333       |
+------------+--------------------+
2 rows selected (0.387 seconds)
```

### 引伸：去掉薪水最大2人、最小2人的平均薪水

我们将窗口扩大到5，前2行和后两行

**执行SQL**

```
select emp_id,
       depart_id,
       salary,
       count(1) over (partition by depart_id order by salary rows between 2 preceding and 2 following) as row_cnt
from t8_salary;
```

**执行结果** 我们仅需要限制row\_cnt = 5 即可获得目标行

```
+---------+------------+-----------+----------+
| emp_id  | depart_id  |  salary   | row_cnt  |
+---------+------------+-----------+----------+
| 1001    | 1          | 5000.00   | 3        |
| 1005    | 1          | 6000.00   | 4        |
| 1002    | 1          | 10000.00  | 5        |
| 1006    | 1          | 10000.00  | 5        |
| 1007    | 1          | 11000.00  | 5        |
| 1003    | 1          | 20000.00  | 4        |
| 1004    | 1          | 30000.00  | 3        |
| 1008    | 2          | 3000.00   | 3        |
| 1009    | 2          | 7000.00   | 4        |
| 1010    | 2          | 9000.00   | 5        |
| 1011    | 2          | 30000.00  | 4        |
| 1012    | 2          | 31000.00  | 3        |
+---------+------------+-----------+----------+
12 rows selected (0.422 seconds)
```

## 四、建表语句和数据插入

```
--建表语句
CREATE TABLE t8_salary (
  emp_id bigint,
  depart_id bigint,
  salary decimal(16,2)
) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
--插入数据
insert into t8_salary (emp_id,depart_id,salary)
values
(1001,1,5000.00),
(1002,1,10000.00),
(1003,1,20000.00),
(1004,1,30000.00),
(1005,1,6000.00),
(1006,1,10000.00),
(1007,1,11000.00),
(1008,2,3000.00),
(1009,2,7000.00),
(1010,2,9000.00),
(1011,2,30000.00),
(1012,2,31000.00)
;
```

> 本文同步在微信公众号”数据仓库技术“和个人博客”数据仓库技术“发表。原文:www.dwsql.com 同时有“数据仓库技术”社群以及有几十位小伙伴一起讨论数据仓库相关技术，欢迎你的加入，社群免费。