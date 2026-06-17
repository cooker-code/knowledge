---
title: SQL实战：如何写SQL求出平均数、中位数和众数？
author: 数据分析师的日常
date: 
url: http://mp.weixin.qq.com/s?__biz=MzUyMTUwNDM4NQ==&mid=2247484739&idx=1&sn=1c8c57139253d748a7a38c94258c09ec&chksm=f9db56adceacdfbb9f4eb135cdccbf0d65a644e790cd64998851238e57dec3afb8400aed41cf&mpshare=1&scene=24&srcid=0425TthgsrHVANNGqrQVHWlI&sharer_shareinfo=9456200988729d49f9cb986de72d4f8c&sharer_shareinfo_first=9456200988729d49f9cb986de72d4f8c#rd
---

点击上方的蓝字和下方的卡片关注我，一起踏上数据分析进阶之路！

你好，我是林赛~

今天分享如何写SQL求出中位数、平均数和众数。

**01**

**数据准备**

我们用前面文章：[SQL常见笔试面试题：计算除去部门最高工资和最低工资的平均工资（4种解题思路）](http://mp.weixin.qq.com/s?__biz=MzUyMTUwNDM4NQ==&mid=2247484684&idx=1&sn=bdd3d2e34b1498014d0d452c4e840839&chksm=f9db56e2ceacdff4d201567fb5df750b8c9740de523158c398b07221e37efe7cfd91f5e67152&scene=21#wechat_redirect)中的数据表data\_learning.employee\_salaries来操作，具体建表语句见文末。

表内容如下：

    

### **02**

### **SQL求平均数**

平均数（mean）也称为均值，它是一组数据相加后除以数据的个数得到的结果。

平均数在统计学中具有重要地位，是集中趋势的最主要测度值，它主要适用于数值型数据，而不适用于分类数据和顺序数据。

平均数包括简单平均数、加权平均数、几何平均数，这里仅分享简单平均数如何用SQL实现。

平均数的计算相对简单。

方式一：使用SQL的AVG函数

```
SELECT              AVG(salary) as avg_salary            FROM              data_learning.employee_salaries            ;
```

方式二：使用SUM和COUNT函数

```
SELECT              SUM(salary)/COUNT(salary) as avg_salary            FROM              data_learning.employee_salaries            ;           
```

结果均如下：    

### **03**

### **SQL求中位数**

中位数（median）是一组数据排序后处于中间位置上的变量值。当该组数据的个数为奇数时，中位数刚好是中间位置的数据值，当该组数据为偶数时，中位数为中间两位数据的均值。

中位数主要测度顺序数据的集中趋势，当然也适用于测度数值型数据的集中趋势，但不适用于分类数据。

 为方便理解，我们先对数据排序，了解数据情况。

SELECT              
    \*              
FROM              
    data\_learning.employee\_salaries              
ORDER BY              
    salary              
;            

排序后的数据表如下，data\_learning.employee\_salaries 准备的数据共20条，排序以后中位数是红色方框中两个数据的平均数。    

方式一：使用窗口函数

备注：适用于支持窗口函数的数据库系统，如 PostgreSQL, SQL Server, Oracle，MySQL 8.0以上

```
WITH RankedValues AS (                 -- 第1步：对salary列从小到大进行排列，并计算数据的个数。               SELECT                    salary                    ,ROW_NUMBER() OVER (ORDER BY salary) AS rn                    ,COUNT(*) OVER () AS cnt                 FROM                     data_learning.employee_salaries            )             -- 第2步：求中位数，利用FLOOR函数、CEIL函数找到求中位数数据的位置，在使用AVG函数对该位置的数据求均值            SELECT                AVG(1.0 * salary) AS median_salary             FROM                RankedValues             WHERE                rn IN (FLOOR(cnt / 2) + 1, CEIL(cnt / 2) + 1);               
```

结果如下：

*上面的SQL语句中可能会有疑惑，中位数的计算会根据数据个数是奇数、还是偶数确定选中间位置的1位、还是2位数求平均呀？为什么上面的SQL看起来像是直接选取了2个数据求平均。*

一组数据的个数n如果是偶数，中间位置就是（n+1）/2的前后两位了，比如这里20条数据，中间位置是(20+1)/2的前后两位，刚好是10、11，如果是奇数21条数据，那么中间位置是（(21+1)/2）= 11。

SQL中的FLOOR函数和CEIL函数刚好可以实现我们要的结果：

```
-- FLOOR()函数可以将一个数值向下取整为最接近的整数。             -- CEIL()函数可以将一个数值向上取整为最接近的整数。               SELECT                FLOOR(10.5)                ,CEIL(10.5)                ,FLOOR(11)                ,CEIL(11)            ;              
```

结果如下：

方式二：使用子查询和聚合函数

```
SELECT                AVG(t1.salary) AS median_salary             FROM (                 SELECT salary,                        @rownum := @rownum + 1 AS rn ,                        @total_rows := (SELECT COUNT(*) FROM data_learning.employee_salaries) as cnt                FROM                      data_learning.employee_salaries , (SELECT @rownum := 0) as r                 ORDER BY salary            ) AS t1,             (                 SELECT @mid_row := CEIL(@total_rows / 2.0)             ) AS t2             -- 数据个数如果能被2整除，则CEIL(total_rows/2)+1,不能被2整除说明是奇数，则为CEIL(total_rows/2)+1            WHERE t1.rn  IN (@mid_row, IF(@total_rows % 2 = 0, @mid_row + 1, @mid_row));           
```

方式一中where条件也可以同方式二的where条件类似写，查询得到的中位数结果也一样：    

```
WITH RankedValues AS (                 -- 第1步：对salary列从小到大进行排列，并计算数据的个数。               SELECT                    salary                    ,ROW_NUMBER() OVER (ORDER BY salary) AS rn                    ,COUNT(*) OVER () AS cnt                 FROM                     data_learning.employee_salaries            )             -- 第2步：求中位数，利用FLOOR函数、CEIL函数找到求中位数数据的位置，再使用AVG函数对该位置的数据求均值            SELECT                AVG(1.0 * salary) AS median_salary             FROM                RankedValues             WHERE                rn IN (CEIL(cnt/2), IF(CEIL(cnt/2) % 2 = 0 ,CEIL(cnt/2) +1 ,CEIL(cnt/2)))           
```

结果均如下：

方式三：使用变量和条件语句

类似于方式二，只是写法略有差异。

```
SET @row_number = 0;             SET @total_rows = (SELECT COUNT(*) FROM data_learning.employee_salaries);             SET @half_rows = CEIL(@total_rows / 2);                          SELECT                AVG(salary) AS median_salary             FROM (                 SELECT salary,                        @row_number := @row_number + 1 AS rn                 FROM                    data_learning.employee_salaries                ORDER BY salary             ) AS ranked_values             WHERE rn IN (@half_rows, IF(@total_rows % 2 = 0, @half_rows + 1, @half_rows));                
```

结果如下：

方式二和方式三都是在MySQL客户端会话中定义的变量，并且只在当前会话中有效，在会话结束时消失。以下是关于用户定义自定义变量的说明：

```
-- 设置用户定义变量            SET@my_variable='some value';            -- 查询用户定义变量            SELECT@my_variable;
```

### **04**

### **SQL求众数**

众数（mode）是一组数据中出现次数最多的变量值。    

众数用于测度分类数据的集中趋势，也适用于顺序数据及数值型数据。

方式一：使用ORDER BY结合LIMIT

```
SELECT                salary as mode_salary,                COUNT(*) AS cnt            FROM                data_learning.employee_salaries            GROUP BY salary            ORDER BY cnt DESC            LIMIT 1            ;
```

结果如下：

方式一有一个问题：当数据具有多个众数时，无法实现查询出全部众数的情况。下面的几种方式可以避免这个问题。

方式二：使用窗口函数和子查询

**解题思路：**

* 第1步：对每个salary计数；
* 第2步：使用窗口函数DENSE\_RANK对计数结果按照从大到小降序排列（这里选DENSE\_RANK更合适，不同窗口函数的区别可以看我之前的文章）；
* 第3步：选取排序序号为1，即计数量最多的salary

**SQL答案：**

```
-- 第3步：选取排序序号为1，即计数量最多的salary            SELECT                salary                ,dense_rank_no            FROM                (                -- 第2步：使用窗口函数对计数结果按照从大到小降序排列                SELECT                    salary                    ,DENSE_RANK() OVER(ORDER BY cnt DESC) AS dense_rank_no                FROM                    (                    -- 第1步：对每个salary计数                    SELECT                        salary,                         COUNT(*) AS cnt                     FROM                        data_learning.employee_salaries                    GROUP BY                        salary                    )t0                )t            WHERE                dense_rank_no = 1            ;         
```

结果如下，可以看到这组数据是有三个众数的：

方式三：使用GROUP BY、HAVING语句和子查询

```
SELECT            -- 第1步：求salary计数                salary                , COUNT(*) AS cnt             FROM                data_learning.employee_salaries             GROUP BY                salary            -- 第2步：筛选计数值大于等于最大的计数值结果            HAVING                COUNT(*) >= (                 SELECT MAX(cnt)                 FROM                    (                    SELECT                        salary                        , COUNT(*) AS cnt                     FROM                    data_learning.employee_salaries                    GROUP BY salary                       )a0            )            ;          
```

结果如下：

方式四：使用GROUP BY、HAVING语句和子查询2

类似于方式三，对HAVING后面的子查询做了简化。用了ALL，表示COUNT(\*)要大于等于所有子查询中的COUNT(\*)才返回满足条件。使用方式三中的MAX更加直观。

```
SELECT                salary                , COUNT(*) AS cnt             FROM                data_learning.employee_salaries             GROUP BY                salary            HAVING                COUNT(*) >= ALL (                 SELECT COUNT(*)                 FROM data_learning.employee_salaries                GROUP BY salary            )            ;             
```

结果如下：

### **附录**

薪资表建表语句如下：

```
CREATE TABLE data_learning.employee_salaries (              id INT PRIMARY KEY,              department varchar(255),              salary INT NOT NULL            );                       INSERT INTO data_learning.employee_salaries VALUES            (10001,'IT部',19800),            (10002,'IT部',29800),            (10003,'IT部',9800),            (10004,'IT部',12000),            (10005,'IT部',6000),            (10006,'市场部',9900),            (10007,'市场部',10000),            (10008,'市场部',3000),            (10009,'市场部',46000),            (10010,'市场部',7000),            (10011,'财务部',8000),            (10012,'财务部',16000),            (10013,'财务部',5800),            (10014,'财务部',14000),            (10015,'财务部',25000),            (10016,'人力资源部',4500),            (10017,'人力资源部',6500),            (10018,'人力资源部',10000),            (10019,'人力资源部',6000),            (10020,'人力资源部',8000);
```

以上就是今天的分享，感谢观看！

点击【赞】和【在看】，这将是我前进的动力和鼓励！谢谢你的支持！

---

欢迎关注我，一起学习数据知识，一起成长~

👇👇👇

- END -