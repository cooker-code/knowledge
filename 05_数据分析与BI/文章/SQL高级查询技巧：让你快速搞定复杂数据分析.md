---
title: SQL高级查询技巧：让你快速搞定复杂数据分析
author: AI源来如此
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkwMDY1Mjc1Mg==&mid=2247485339&idx=1&sn=79fa0a0202608aa92b756c5d1c7038d4&chksm=c1a85e63a99132fef11cd605a9f1adc210597e11f8667ed12ae623845a4bcfc9e84b2782ede1&mpshare=1&scene=24&srcid=0807XX1Umg4nA11DqNhCsYBR&sharer_shareinfo=11cae2bda6085fbabd342b32a66321a3&sharer_shareinfo_first=11cae2bda6085fbabd342b32a66321a3#rd
---

👋 热爱编程的小伙伴们，欢迎来到我的编程技术分享公众号！在这里，我会分享编程技巧、实战经验、技术干货，还有各种有趣的编程话题！

> ❝
>
> SQL（Structured Query Language）是与数据库交互的主要语言，无论是数据检索、插入、更新还是删除操作都离不开 SQL 查询。掌握基本的 SQL 查询语法后，深入理解并应用高级查询技巧可以显著提高数据处理效率和查询性能。本篇文章将介绍几种常用的 SQL 高级查询技巧，帮助你在实际工作中提升查询效率和数据处理能力。

## 窗口函数（Window Functions）

### 窗口函数概述

窗口函数是一种特殊的 SQL 函数，它能够在一组行（称为窗口）上执行计算，但不会将结果合并到单个行中，这与聚合函数有所不同。窗口函数在处理排名、累计和运行总和等场景中非常有用。

### 窗口函数的语法结构

**窗口函数的基本语法如下：**

```
<窗口函数> OVER (  
    [PARTITION BY <分区列>]  
    [ORDER BY <排序列>]  
)
```

* **PARTITION BY** 用于将数据分成不同的组。
* **ORDER BY** 用于定义计算的顺序。

### 常见窗口函数

1. **ROW\_NUMBER()** ：为每一行分配一个唯一的序号。
2. **RANK()** ：为每一行分配一个序号，序号间可能有跳跃。
3. **DENSE\_RANK()** ：为每一行分配一个序号，序号间无跳跃。
4. **LEAD()** 和 **LAG()** ：访问同一组中前一行或后一行的数据。

**示例：**

```
SELECT  
    employee_id,  
    department_id,  
    salary,  
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS row_num,  
    RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank,  
    DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dense_rank,  
    LAG(salary, 1) OVER (PARTITION BY department_id ORDER BY salary DESC) AS previous_salary  
FROM  
    employees;
```

**小结**：窗口函数通过在行之间进行计算，提供了强大的数据分析功能。

## 递归查询（Recursive Queries）

### 递归查询概述

递归查询是一种自引用的查询方式，常用于处理树形结构的数据，如组织架构、目录结构等。

### 递归查询的语法结构

递归查询使用`WITH RECURSIVE`子句，其基本语法如下：

```
WITH RECURSIVE cte_name AS (  
    初始查询  
    UNION ALL  
    递归查询  
)  
SELECT * FROM cte_name;
```

### 实际应用示例

```
WITH RECURSIVE EmployeeCTE AS (  
    SELECT  
        employee_id,  
        manager_id,  
        1 AS level  
    FROM  
        employees  
    WHERE  
        manager_id IS NULL  
    UNION ALL  
    SELECT  
        e.employee_id,  
        e.manager_id,  
        ecte.level + 1  
    FROM  
        employees e  
    INNER JOIN EmployeeCTE ecte ON e.manager_id = ecte.employee_id  
)  
SELECT * FROM EmployeeCTE;
```

**小结**：递归查询在处理层级结构数据时非常有用，能够方便地展现数据之间的层级关系。

## 公共表表达式（CTE, Common Table Expressions）

### CTE 概述

CTE 是一种临时的结果集，其定义只在单个查询的执行周期内有效。CTE 能使复杂查询更易读、易维护。

### CTE 的语法结构

CTE 的基本语法如下：

```
WITH cte_name AS (  
    查询语句  
)  
SELECT * FROM cte_name;
```

### CTE 的实际应用

```
WITH SalesCTE AS (  
    SELECT  
        sales_person,  
        SUM(amount) AS total_sales  
    FROM  
        sales  
    GROUP BY  
        sales_person  
)  
SELECT  
    sales_person,  
    total_sales  
FROM  
    SalesCTE  
WHERE  
    total_sales > 10000;
```

**小结**：CTE 能够将复杂查询分解成多个部分，使得 SQL 查询更加清晰和易于维护。

## 子查询（Subqueries）

### 子查询概述

子查询是嵌套在另一个查询中的查询，可以在`SELECT`、`WHERE`、`FROM`、`HAVING`子句中使用。子查询可以分为相关子查询和非相关子查询。

### 子查询的使用场景

在实际应用中，子查询常用于筛选条件、数据过滤等场景。

### 实际应用示例

```
SELECT  
    employee_id,  
    salary  
FROM  
    employees  
WHERE  
    salary > (SELECT AVG(salary) FROM employees);
```

**小结**：子查询可以将复杂的筛选条件嵌套在查询中，使得查询更加灵活和强大。

## 集合操作（Set Operations）

### 集合操作概述

集合操作用于将两个或多个查询结果集进行合并或比较。常见的集合操作符包括`UNION`、`INTERSECT`、`EXCEPT`。

### 集合操作的语法结构

**基本语法如下：**

```
SELECT column_list FROM table1  
UNION [ALL]  
SELECT column_list FROM table2;  
  
SELECT column_list FROM table1  
INTERSECT  
SELECT column_list FROM table2;  
  
SELECT column_list FROM table1  
EXCEPT  
SELECT column_list FROM table2;
```

### 实际应用示例

```
-- 合并两个查询结果集  
SELECT name FROM customers  
UNION  
SELECT name FROM suppliers;  
  
-- 找出两个查询结果集的交集  
SELECT name FROM customers  
INTERSECT  
SELECT name FROM suppliers;  
  
-- 找出只在第一个查询结果集中存在的记录  
SELECT name FROM customers  
EXCEPT  
SELECT name FROM suppliers;
```

**小结**：集合操作可以方便地进行数据集之间的合并、比较和差异分析。

## 高级过滤与排序技巧

### 高级过滤技巧

**使用正则表达式进行过滤：**

```
SELECT  
    email  
FROM  
    users  
WHERE  
    email REGEXP '^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$';
```

### 高级排序技巧

**多条件排序：**

```
SELECT  
    employee_id,  
    department_id,  
    salary  
FROM  
    employees  
ORDER BY  
    department_id,  
    salary DESC;
```

**小结**：通过使用正则表达式和多条件排序，可以更加灵活地进行数据过滤和排序。

### SQL 查询优化的建议

* **使用索引**：创建适当的索引可以显著提高查询性能。
* **避免全表扫描**：使用 WHERE 子句进行筛选，避免不必要的全表扫描。
* **简化复杂查询**：使用 CTE、子查询等手段将复杂查询简化，提高可读性和维护性。
* **合理使用连接**：选择合适的连接方式（如内连接、外连接）来优化查询性能。

## 结语

本文介绍了窗口函数、递归查询、公共表表达式、子查询、集合操作、高级过滤与排序技巧等高级 SQL 查询技巧。通过掌握这些高级查询技巧，你可以更加高效地处理复杂数据查询，提高数据库操作的性能和效率。

---

个人观点，仅供参考，非常感谢各位朋友们的支持与关注！

如果你觉得这个作品对你有帮助，请不吝**点赞**、**在看**，分享给身边更多的朋友。

同时，如果你有任何疑问或建议，欢迎在评论区留言交流。