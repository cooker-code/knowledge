---
title: SQL实战：三种方法计算用户最近两次交易时间差
author: 数据分析师的日常
date: 
url: http://mp.weixin.qq.com/s?__biz=MzUyMTUwNDM4NQ==&mid=2247484701&idx=1&sn=3ba8d8af6529ddc5ea80808fcfd6d7f9&chksm=f9db56f3ceacdfe5ed1eb02253a9b6d6321faf4e0242ea1e17a46a3d29b8611d1e542d60795a&mpshare=1&scene=24&srcid=0418dITZxJM4qukuEeen3gUj&sharer_shareinfo=a18145bf46ce4fc4305a45ae1a7f6cb3&sharer_shareinfo_first=a18145bf46ce4fc4305a45ae1a7f6cb3#rd
---

点击上方的蓝字和下方的卡片关注我，一起踏上数据分析进阶之路！

你好，我是林赛~

今天分享使用三种方法计算用户最近两次交易时间差。

### **01 数据准备**

假设有一张表transactions，一列是用户id（user\_id），一列是交易时间（transaction\_time），问用户最近一次交易时间和倒数第二次交易时间之差，单位：秒。

示例表数据如下，具体建表及插入数据语句见文末附录：

    

### **02 解题思路及SQL答案**

#### **解题思路一：使用窗口函数 — row\_number() over()函数**

* 在公共表达式中使用窗口函数ROW\_NUMBER()为每个用户的交易时间降序排列；
* 使用自连接，将每个用户的每次交易时间与其倒数第二次交易时间关联；
* 筛选最近一次交易时间的记录并计算最近一次交易时间与倒数第二次交易时间相差秒数。

SQL如下：

```
WITH RankedTransactions AS (                 SELECT                     user_id,                     transaction_time,                     ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY transaction_time DESC) AS rn                 FROM                     transactions             )                          SELECT                 t1.user_id,                     t1.transaction_time as latest_time,                    t2.transaction_time as second_latest_time,                TIMESTAMPDIFF(SECOND, t2.transaction_time, t1.transaction_time) AS time_difference_s             FROM                 RankedTransactions t1             JOIN                 RankedTransactions t2 ON t1.user_id = t2.user_id AND t2.rn = 2             WHERE                 t1.rn = 1;
```

          

结果如下：

#### **解题思路二：使用公共表达式与表连接**

* 第一步：找出每个用户的最近一次交易时间 ；
* 第二步：剔除最近一次交易时间后，找出每个用户的最近一次交易时间（即倒数第二次）；
* 第三步：将两个子查询结果关联起来，并计算时间差。

SQL如下：

```
-- 第一步：找出每个用户的最近一次交易时间             WITH LatestTransactions AS (                 SELECT                     user_id,                     MAX(transaction_time) AS latest_time                 FROM                     transactions                 GROUP BY                     user_id             ),                          -- 第二步：剔除最近一次交易时间后，找出每个用户的最近一次交易时间（即倒数第二次）             SecondLatestTransactions AS (                 SELECT                     t.user_id,                     MAX(t.transaction_time) AS second_latest_time                 FROM                     transactions t                 LEFT JOIN                     LatestTransactions lt ON t.user_id = lt.user_id                    AND t.transaction_time = lt.latest_time                 WHERE                     lt.latest_time IS NULL OR t.transaction_time < lt.latest_time                 GROUP BY                     t.user_id             )                          -- 第三步：将两个子查询结果关联起来，并计算时间差             SELECT                 lt.user_id,                 latest_time,                second_latest_time,                TIMESTAMPDIFF(SECOND, slt.second_latest_time, lt.latest_time) AS time_difference             FROM                 LatestTransactions lt             JOIN                 SecondLatestTransactions slt ON lt.user_id = slt.user_id;
```

         
结果如下：

#### **解题思路三：窗口函数、子查询结合法**

* 第一步：对每个用户的交易时间降序并位移获取下一个交易时间，这时候最近一次交易时间和倒数第二次交易时间就在每一个用户id的第一行；
* 第二步：计算每个用户的交易时间和其降序排列的下一个交易时间之差，并使用窗口函数对每个用户的交易时间降序排列；
* 第三步：筛选最近一次交易时间，即可得到计算后的结果。

SQL如下：

```
-- 第三步：筛选最近一次交易时间，即可得到计算后的结果            SELECT                user_id,                transaction_time,                       next_time,                time_difference            FROM                 (                -- 第二步：计算每个用户的交易时间和其降序排列的下一个交易时间之差，并使用窗口函数对每个用户的交易时间降序排列                SELECT                     user_id,                    transaction_time,                           next_time,                    TIMESTAMPDIFF(SECOND, next_time, transaction_time) AS time_difference,                    ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY transaction_time DESC) AS rn                FROM                    -- 第一步：对每个用户的交易时间降序并位移获取下一个交易时间，这时候最近一次交易时间和倒数第二次交易时间就在每一个用户id的第一行                    (                     SELECT                         user_id,                         transaction_time,                           LEAD(transaction_time) OVER (PARTITION BY user_id ORDER BY transaction_time DESC) AS next_time                     FROM                         transactions                     )a                )a            WHERE rn = 1            ；
```

         

结果如下：

### **附录**

建表及插入数据语句：

```
CREATE TABLE transactions (              user_id INT,              transaction_time datetime            );                       insert into transactions values            ('123',    '2024-01-01 15:30:30'),            ('123',    '2024-01-02 16:50:00'),            ('123',    '2024-01-03 16:51:38'),            ('123',    '2024-01-04 15:30:30'),            ('234',    '2024-01-01    16:51:38'),            ('234',    '2024-01-04 12:35:00'),            ('234',    '2024-01-07 09:58:00'),            ('345',    '2024-01-01 16:51:38'),            ('345',    '2024-01-02 15:30:30'),            ('345',    '2024-03-16 16:51:38'),            ('345',    '2024-03-17 12:35:00'),            ('345',    '2024-03-18 16:51:38'),            ('456',    '2024-03-04 16:51:38'),            ('456',    '2024-03-08 09:58:00'),            ('456',    '2024-03-09 12:35:00'),            ('456',    '2024-03-13 15:30:30');
```

以上就是今天的分享，感谢观看！

点击【赞】和【在看】，这将是我前进的动力和鼓励！谢谢你的支持！

---

欢迎关注我，一起学习数据知识，一起成长~

👇👇👇

- END -