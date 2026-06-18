---
title: SQL进阶技巧：如何查找相邻座位员？| 员工座位安排问题
author: 会飞的一十六
date:
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247488568&idx=1&sn=0f9411556fe1c2cee11426526cf45913&chksm=e9763047cc084238634580b5bdd52dac847e06de7b367e434346deeb40ef56d6126b932da6a9&mpshare=1&scene=24&srcid=0108rcBs54AXAHhhpL5K8XiH&sharer_shareinfo=2e136a5456f43602942c4c00d47cd00c&sharer_shareinfo_first=2e136a5456f43602942c4c00d47cd00c#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL窗口滚动与序列问题|SQL窗口滚动与序列问题]]


点击上方【蓝色】字体   关注我们

# 01 场景描述

座位表 employees，用于记录员工座位信息，包含 employee\_id（员工 ID）、row\_number（座位行号）、column\_number（座位列号）这几个字段。

现在要找出每个员工相邻座位的员工。这里相邻座位定义为行号相差 1 或者列号相差 1 的座位对应的员工（暂不考虑斜对角相邻情况）。

# 02 数据准备

```
-- 创建表CREATE TABLE employees (    employee_id INT,    row_number INT,    column_number INT);
-- 插入示例数据INSERT INTO employees VALUES(1, 1, 1),(2, 1, 2),(3, 1, 3),(4, 2, 1),(5, 2, 2),(6, 2, 3),(7, 3, 1),(8, 3, 2),(9, 3, 3);
```

# 03 问题分析

**步骤一：使用分析函数计算每个员工与其他员工在行号和列号上的差值**
我们要找出相邻座位的员工，首先需要计算每个员工与其他员工在座位行号和列号上的差值，通过分析函数来实现这一操作，方便后续判断相邻关系。

```
-- 使用分析函数计算行号和列号差值SELECT     e1.employee_id AS employee_id_1,    e2.employee_id AS employee_id_2,    e1.row_number - e2.row_number AS row_diff,    e1.column_number - e2.column_number AS col_diffFROM employees e1JOIN employees e2 ON e1.employee_id!= e2.employee_idORDER BY e1.employee_id, e2.employee_id;
```

在这个步骤中，通过自连接 employees 表（e1 和 e2 分别代表同一张表的不同别名，用于关联比较不同员工之间的情况），并要求 employee\_id 不相等（避免员工与自己比较），然后使用减法运算计算出两个员工之间在座位行号（row\_diff 字段）和列号（col\_diff 字段）上的差值，为下一步判断相邻关系做准备。按照 employee\_id 排序只是为了让结果展示更有条理，方便查看。

步骤二：基于行号和列号差值判断相邻关系并筛选出相邻座位的员工

相邻座位定义为行号相差 1 或者列号相差 1 的座位对应的员工（暂不考虑斜对角相邻情况），我们基于第一步计算出的差值来判断这种相邻关系，并筛选出符合条件的员工组合。

```
-- 判断相邻关系并筛选出相邻座位的员工SELECT     employee_id_1,    employee_id_2FROM (    SELECT         e1.employee_id AS employee_id_1,        e2.employee_id AS employee_id_2,        e1.row_number - e2.row_number AS row_diff,        e1.column_number - e2.column_number AS col_diff    FROM employees e1    JOIN employees e2 ON e1.employee_id!= e2.employee_id) subqueryWHERE (ABS(row_diff) = 1 AND col_diff = 0) OR (row_diff = 0 AND ABS(col_diff) = 1);
```

在这个步骤中，先在子查询里复用了第一步计算行号和列号差值的结果，然后在外层查询中，通过 WHERE 子句设定相邻座位的判断条件。使用 ABS 函数取差值的绝对值，判断当行号差值的绝对值等于 1 且列号差值为 0 （表示同行相邻列），或者行号差值为 0 且列号差值的绝对值等于 1 （表示同列相邻行）时，就认定这两个员工的座位是相邻的，筛选出符合这些条件的员工组合，也就是找到了相邻座位的员工。

# 04  小 结

文章分析了一种员工相邻座位查找问题的解法。

窗口函数分析法（另一种解法）：

```

```

```
 **利用窗口函数和条件判断（另一种窗口函数用法）**   - **思路**：     - 可以先使用窗口函数来计算每个员工的上下左右四个方向的员工ID（如果存在）。然后，通过筛选这些计算结果来找出相邻座位的员工。   - **示例代码**：     ```sql    -- 使用窗口函数找出相邻座位的员工（同行相邻列和同列相邻行情况）select employee_id_1     ,  value employe_id_2from (SELECT employee_id_1           , concat_ws(',', cast(left_adjacent as string), cast(right_adjacent as string), cast(up_adjacent as string),                       cast(down_adjacent as string)) employee_id_2      FROM (               -- 利用窗口函数计算相邻座位相关信息               SELECT e.employee_id                                                  AS employee_id_1,                      -- 同行左边相邻员工ID（如果有）                      LAG(e.employee_id, 1)                          OVER (PARTITION BY e.row_number ORDER BY e.column_number)  AS left_adjacent,                      -- 同行右边相邻员工ID（如果有）                      LEAD(e.employee_id, 1)                           OVER (PARTITION BY e.row_number ORDER BY e.column_number) AS right_adjacent,                      -- 同列上边相邻员工ID（如果有）                      LAG(e.employee_id, 1)                          OVER (PARTITION BY e.column_number ORDER BY e.row_number)  AS up_adjacent,                      -- 同列下边相邻员工ID（如果有）                      LEAD(e.employee_id, 1)                           OVER (PARTITION BY e.column_number ORDER BY e.row_number) AS down_adjacent               FROM employees e) subquery      WHERE coalesce(left_adjacent, right_adjacent, up_adjacent, down_adjacent) is not null) t         lateral view explode(split(employee_id_2,',')) tmp as value
```

中间步骤结果如下：

**第二步：行列转换**

```
      SELECT employee_id_1           , concat_ws(',', cast(left_adjacent as string), cast(right_adjacent as string), cast(up_adjacent as string),                       cast(down_adjacent as string)) employee_id_2      FROM (               -- 利用窗口函数计算相邻座位相关信息               SELECT e.employee_id                                                  AS employee_id_1,                      -- 同行左边相邻员工ID（如果有）                      LAG(e.employee_id, 1)                          OVER (PARTITION BY e.row_number ORDER BY e.column_number)  AS left_adjacent,                      -- 同行右边相邻员工ID（如果有）                      LEAD(e.employee_id, 1)                           OVER (PARTITION BY e.row_number ORDER BY e.column_number) AS right_adjacent,                      -- 同列上边相邻员工ID（如果有）                      LAG(e.employee_id, 1)                          OVER (PARTITION BY e.column_number ORDER BY e.row_number)  AS up_adjacent,                      -- 同列下边相邻员工ID（如果有）                      LEAD(e.employee_id, 1)                           OVER (PARTITION BY e.column_number ORDER BY e.row_number) AS down_adjacent               FROM employees e) subquery      WHERE coalesce(left_adjacent, right_adjacent, up_adjacent, down_adjacent) is not null
```

**第一步： 核心逻辑**

```
-- 利用窗口函数计算相邻座位相关信息SELECT e.employee_id                                                  AS employee_id_1,       -- 同行左边相邻员工ID（如果有）       LAG(e.employee_id, 1)           OVER (PARTITION BY e.row_number ORDER BY e.column_number)  AS left_adjacent,       -- 同行右边相邻员工ID（如果有）       LEAD(e.employee_id, 1)            OVER (PARTITION BY e.row_number ORDER BY e.column_number) AS right_adjacent,       -- 同列上边相邻员工ID（如果有）       LAG(e.employee_id, 1)           OVER (PARTITION BY e.column_number ORDER BY e.row_number)  AS up_adjacent,       -- 同列下边相邻员工ID（如果有）       LEAD(e.employee_id, 1)            OVER (PARTITION BY e.column_number ORDER BY e.row_number) AS down_adjacentFROM employees e;
```

员工座位查找问题总结

（1）按部门角落座位查找：

问题特点：在角落座位查找的基础上增加了部门维度的限制，要求分别找出每个部门内部的角落座位员工。

解题关键：首先按部门分组计算每个部门各自的最大行号和最大列号，可通过分组聚合操作实现。然后将这些信息与员工表进行关联，结合角落座位的判断条件进行筛选，重点在于处理好部门分组与整体座位条件判断之间的逻辑关系，常借助子查询或连接操作来整合数据。

（2）相邻座位员工查找

问题特点：关注员工座位之间的相邻关系，定义相邻为行号相差特定值（如 1）或列号相差特定值（如 1）的座位对应的员工，不考虑斜对角相邻等复杂情况。

解题关键：

一种方法是通过自连接员工表，计算不同员工之间的行号差值和列号差值，然后根据相邻条件进行筛选。例如使用简单的减法运算得到差值，并在 WHERE 子句中设定相邻的差值判断条件。

另一种是利用窗口函数（如 LAG 和 LEAD），按照行号或列号分区排序，获取每个员工在同行或同列的相邻员工 ID 信息，再通过判断这些 ID 是否为空来确定相邻关系。这种方法在处理大规模数据且数据库支持窗口函数时，效率较高且代码逻辑相对简洁。

（3）根据座位距离查找员工

问题特点：基于员工座位的坐标信息（如 x 坐标和 y 坐标），按照给定的距离范围查找与特定员工距离在该范围内的其他员工，涉及到坐标系统中的距离计算。

解题关键：首先要明确距离计算的公式，如平面直角坐标系下的欧几里得距离公式。在 SQL 中，通过相应的数学函数（如 POWER 用于计算平方，SQRT 用于计算平方根）结合坐标差值计算来实现距离计算。通常会采用嵌套查询的方式，先计算坐标差值，再基于差值计算距离，最后根据距离范围和特定员工条件进行筛选，确保数据的层层处理和结果的准确性。

往期精彩

[SQL进阶技巧：如何根据座位距离查找员工？| 员工座位安排问题](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247488539&idx=1&sn=93a94a9955d33838fbd859b84eceae0a&scene=21#wechat_redirect)

[SQL进阶技巧：如何查找每个部门里坐在角落位置的员工？| 员工座位安排问题](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247488513&idx=1&sn=319c0c3420eba62fb12ee300945dad8f&scene=21#wechat_redirect)

[解锁SQL无限可能：如何利用HiveSQL实现0-1背包问题？](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247488479&idx=1&sn=6ec008a44f99dc1d95655cb515f276da&scene=21#wechat_redirect)

[数仓建模：一种动态字段表模型设计方法与应用](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247488311&idx=1&sn=846fcb37263ecbed9cf078e685828249&scene=21#wechat_redirect)

[SQL进阶技巧：如何根据工业制程参数计算良品率？](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247488298&idx=1&sn=8dbf4c395e64716a481eca63cfd9c796&scene=21#wechat_redirect)

[数据科学与SQL：如何利用SQL计算线性回归系数？](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247488265&idx=1&sn=5f8813ad8e749d65632cd60dd4efb260&scene=21#wechat_redirect)

######

###### 会飞的一十六

扫描右侧二维码关注我们

点个【在看】 你最好看

点击“阅读原文”查看更多~~