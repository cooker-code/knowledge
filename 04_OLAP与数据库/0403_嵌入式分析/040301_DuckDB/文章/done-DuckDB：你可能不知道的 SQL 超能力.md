> 已吸收至：[[04_OLAP与数据库/0403_嵌入式分析/040301_DuckDB/040301_核心知识点/DuckDB并行Join与优化器边界|DuckDB并行Join与优化器边界]]
---
title: DuckDB：你可能不知道的 SQL 超能力
author: MaxAiDB
date:
url: https://mp.weixin.qq.com/s?__biz=MzUwOTU4OTU2NQ==&mid=2247483660&idx=1&sn=2c3c3d4eeb695f851c899939c5518296&chksm=f8006fcba1c8de42d6f51aae1960fef354a5d6a32b7508a200a8b24bf14e93eb68acee9c2a83&mpshare=1&scene=24&srcid=0406MYigJ08Fk50pRZwPCnxh&sharer_shareinfo=d1aff675e7537a972243df4222167de8&sharer_shareinfo_first=d1aff675e7537a972243df4222167de8#rd
---

> 本文是《DuckDB：从上手到内核》系列的第 2 篇。上一篇我们跑通了第一个查询，这次来看看 DuckDB 的 SQL 到底有哪些"超能力"。

---

## 标准 SQL 已经够好了，为什么还要"扩展"？

用过标准 SQL 的人都知道几个痛点：

•想查所有列但排除某一列？只能把其他列全写一遍。•想做行列转换？对不起，标准 SQL 的写法能把你绕晕。•窗口函数算完，想过滤结果？再套一层子查询。•想直接查个 CSV 文件？先建表、再导入、再查询——三步走。

DuckDB 对这些痛点逐一开刀。它在兼容 PostgreSQL SQL 方言的基础上，加了一堆实用的语法糖。用过之后，你会觉得"SQL 本来就该是这样的"。

---

## 语法糖 1：SELECT \* EXCLUDE —— 排除列查询

**场景**：你有一张 20 列的表，想看除了 `created_at` 和 `updated_at` 之外的所有列。

传统写法：把其他 18 列全列出来。

DuckDB 写法：

```
SELECT * EXCLUDE (created_at, updated_at)FROM orders;
```

一行搞定。如果你想在查询的同时修改某一列的表达式，可以用 `REPLACE`：

```
-- 把 price 列乘以汇率，其他列不变SELECT * REPLACE (price * 7.2 AS price)FROM products;
```

这两个关键字可以组合使用：

```
SELECT * EXCLUDE (internal_id) REPLACE (name || ' (' || category || ')' AS name)FROM products;
```

根据 DuckDB 官方文档[1]，`EXCLUDE` 和 `REPLACE` 是对 Star Expression（`*`）的扩展，让 `SELECT *` 在生产代码中变得真正可用。

---

## 语法糖 2：PIVOT / UNPIVOT —— 行列转换

行列转换在数据分析中太常见了——做报表、做交叉表、做宽表转长表。在标准 SQL 里，这通常需要一堆 CASE WHEN 嵌套。DuckDB 直接内置了 `PIVOT` 和 `UNPIVOT`。

### PIVOT：把行变成列

**图 1：PIVOT 操作示意图**

```
原始数据（长表）：                    PIVOT 后（宽表）：┌────────┬─────────┬─────────┐      ┌────────┬────────┬────────┐│ region │  year   │  sales  │      │ region │  2023  │  2024  │├────────┼─────────┼─────────┤      ├────────┼────────┼────────┤│ East   │  2023   │  100    │  ──► │ East   │  100   │  150   ││ East   │  2024   │  150    │      │ West   │  200   │  180   ││ West   │  2023   │  200    │      └────────┴────────┴────────┘│ West   │  2024   │  180    │└────────┴─────────┴─────────┘
```

```
-- 先准备数据CREATE TABLE regional_sales ASSELECT * FROM (VALUES    ('East', 2023, 100),    ('East', 2024, 150),    ('West', 2023, 200),    ('West', 2024, 180)) AS t(region, year, sales);
-- PIVOT：按年份展开成列PIVOT regional_salesON yearUSING SUM(sales)GROUP BY region;
```

结果：

```
┌────────┬──────┬──────┐│ region │ 2023 │ 2024 │├────────┼──────┼──────┤│ East   │  100 │  150 ││ West   │  200 │  180 │└────────┴──────┴──────┘
```

DuckDB 的 PIVOT 会**自动检测**年份有哪些值，不需要你手动列出 `IN (2023, 2024)`（当然你也可以手动指定来过滤）。

### UNPIVOT：把列变回行

反过来也一样方便：

```
CREATE TABLE wide_sales(region VARCHAR, q1 INT, q2 INT, q3 INT, q4 INT);INSERT INTO wide_sales VALUES ('East', 100, 120, 90, 150), ('West', 200, 180, 210, 190);
UNPIVOT wide_salesON q1, q2, q3, q4INTO NAME quarter VALUE sales;
```

结果：

```
┌────────┬─────────┬───────┐│ region │ quarter │ sales │├────────┼─────────┼───────┤│ East   │ q1      │   100 ││ East   │ q2      │   120 ││ East   │ q3      │    90 ││ East   │ q4      │   150 ││ West   │ q1      │   200 ││ West   │ q2      │   180 ││ West   │ q3      │   210 ││ West   │ q4      │   190 │└────────┴─────────┴───────┘
```

---

## 语法糖 3：QUALIFY —— 窗口函数过滤

这可能是 DuckDB 最让人惊喜的语法扩展之一。

**场景**：取每个部门薪资最高的员工。

**传统 SQL（需要子查询）：**

```
SELECT * FROM (    SELECT *,           ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn    FROM employees) subWHERE rn = 1;
```

**DuckDB 写法（用 QUALIFY）：**

```
SELECT *FROM employeesQUALIFY ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) = 1;
```

少了一层嵌套，意图更直接。`QUALIFY` 就是窗口函数的 `WHERE`——在窗口函数计算完之后再做过滤。

再来一个实用例子——取每个用户的最近 3 笔订单：

```
SELECT user_id, order_date, amountFROM ordersQUALIFY ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date DESC) <= 3;
```

如果用标准 SQL，又得套子查询。`QUALIFY` 让代码清爽不少。

---

## 语法糖 4：直接查文件，不用建表

这是 DuckDB 的招牌能力。我们在第 1 篇见过，这里展开更多用法。

### 查 CSV

```
-- 基本查询SELECT * FROM 'data/users.csv' LIMIT 10;
-- 自动检测分隔符、编码、类型SELECT * FROM read_csv('data/users.csv', auto_detect=true);
-- 手动指定参数SELECT * FROM read_csv('data/users.csv',    delim=',',    header=true,    columns={'name': 'VARCHAR', 'age': 'INTEGER', 'city': 'VARCHAR'});
```

### 查 Parquet

```
-- 单文件SELECT COUNT(*) FROM 'events.parquet';
-- Glob 模式：查 data 目录下所有 ParquetSELECT * FROM 'data/**/*.parquet' WHERE event_date >= '2024-01-01';
-- 带 Hive 分区的目录结构SELECT * FROM read_parquet('data/year=*/month=*/*.parquet', hive_partitioning=true);
```

### 查 JSON

```
-- 单层 JSONSELECT * FROM 'logs.json';
-- 嵌套 JSON —— DuckDB 会自动展开结构SELECT * FROM read_json('nested_data.json', auto_detect=true);
```

### 混合查询 + 导出

**图 2：跨格式 ETL 数据流**

```
 CSV 文件 ──┐             ├──► DuckDB SQL 处理 ──► Parquet 输出 JSON 文件 ─┘     (过滤、JOIN、聚合)
```

```
-- 读 CSV 和 JSON，JOIN 后导出 ParquetCOPY (    SELECT o.order_id, o.amount, c.name, c.city    FROM 'orders.csv' o    JOIN 'customers.json' c ON o.customer_id = c.id    WHERE o.amount > 100) TO 'high_value_orders.parquet' (FORMAT PARQUET, COMPRESSION ZSTD);
```

一条 SQL，完成了读取两种格式 → JOIN → 过滤 → 导出 Parquet 的完整 ETL 流程。不需要 Pandas，不需要 Spark。

---

## 语法糖 5：SAMPLE —— 采样查询

数据量太大，先看个样本？

```
-- 随机取 1% 的数据SELECT * FROM large_table USING SAMPLE 1%;
-- 取 1000 行样本SELECT * FROM large_table USING SAMPLE 1000 ROWS;
-- 可重复的采样（指定 seed）SELECT * FROM large_table USING SAMPLE 10% (bernoulli, 42);
```

这在数据探索阶段特别有用——先看看数据长什么样，再写正式的查询。

---

## 语法糖 6：COLUMNS 表达式 —— 批量操作列

这是一个非常强大但容易被忽略的功能。`COLUMNS` 让你可以用正则或 Lambda 批量操作列：

```
-- 对所有数值列求 SUMSELECT SUM(COLUMNS(*)) FROM sales;
-- 只对以 'price' 开头的列求平均SELECT AVG(COLUMNS('price.*')) FROM products;
-- 排除某些列后操作SELECT MIN(COLUMNS(* EXCLUDE (id, name))) FROM measurements;
```

配合 `EXCLUDE` 使用，威力倍增。

---

## 完整实战案例：从 CSV 到分析报表

把上面的语法糖串起来，完成一个真实的分析任务。

**场景**：你有一个电商订单 CSV 文件，需要分析各地区、各品类的月度销售趋势，并导出 Parquet。

```
-- 步骤 1：先看看数据长什么样SELECT * FROM 'ecommerce_orders.csv' USING SAMPLE 5 ROWS;
-- 步骤 2：数据概览SELECT COUNT(*) AS total_orders,       COUNT(DISTINCT customer_id) AS unique_customers,       MIN(order_date) AS earliest,       MAX(order_date) AS latestFROM 'ecommerce_orders.csv';
-- 步骤 3：月度 × 地区 × 品类 聚合CREATE TABLE monthly_sales ASSELECT    DATE_TRUNC('month', order_date::DATE) AS month,    region,    category,    SUM(amount) AS total_sales,    COUNT(*) AS order_count,    AVG(amount) AS avg_order_valueFROM 'ecommerce_orders.csv'GROUP BY ALL;  -- DuckDB 支持 GROUP BY ALL，自动按非聚合列分组
-- 步骤 4：找出每个地区销售额最高的品类SELECT region, category, total_salesFROM monthly_salesQUALIFY RANK() OVER (PARTITION BY region ORDER BY total_sales DESC) = 1;
-- 步骤 5：做透视表——地区 × 品类 交叉表PIVOT monthly_salesON categoryUSING SUM(total_sales)GROUP BY region;
-- 步骤 6：导出结果COPY monthly_sales TO 'output/monthly_sales.parquet' (FORMAT PARQUET);
```

这个例子用到了：`read_csv`（隐式）、`SAMPLE`、`GROUP BY ALL`、`QUALIFY`、`PIVOT`、`COPY TO`。6 步完成从原始数据到分析报表的全流程。

---

## 与 Python/Pandas 的无缝集成

DuckDB 不只是 SQL 工具——它和 Python 生态深度融合：

```
import duckdbimport pandas as pd
# 创建一个 Pandas DataFramedf = pd.DataFrame({    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],    'department': ['Eng', 'Mkt', 'Eng', 'Mkt'],    'salary': [95000, 72000, 88000, 76000]})
# 直接用 SQL 查询 DataFrame —— 不需要导入！result = duckdb.sql("""    SELECT department, AVG(salary) AS avg_salary    FROM df    GROUP BY department""").fetchdf()
print(result)
```

输出：

```
  department  avg_salary0        Eng     91500.01        Mkt     74000.0
```

DuckDB 直接识别了 Python 变量 `df`，不需要 `register` 或 `CREATE TABLE`。这种"Python 变量即 SQL 表"的体验，第 3 篇会详细展开。

---

## 小结

DuckDB 的 SQL 扩展不是花架子，每一个都解决了真实的痛点：

1.**EXCLUDE / REPLACE**：让 `SELECT *` 变得实用2.**PIVOT / UNPIVOT**：行列转换一行搞定，告别 CASE WHEN3.**QUALIFY**：窗口函数结果直接过滤，少套一层子查询4.**直接查文件**：CSV / Parquet / JSON 即查即用，不用建表5.**SAMPLE**：大数据探索时先看样本6.**GROUP BY ALL**：自动推断分组列，少写重复代码

下一篇，我们来看 DuckDB 的 Python 集成——如何用它替代（或增强）你的 Pandas 工作流。

---

## 延伸阅读

•DuckDB SQL 语法文档：https://duckdb.org/docs/sql/introduction[2]•PIVOT 语句详解：https://duckdb.org/docs/sql/statements/pivot[3]•QUALIFY 子句：https://duckdb.org/docs/sql/query\_syntax/qualify[4]•SELECT 扩展（EXCLUDE/REPLACE）：https://duckdb.org/docs/sql/query\_syntax/select[5]•CSV 导入指南：https://duckdb.org/docs/data/csv/overview[6]

### References

`[1]` DuckDB 官方文档: *https://duckdb.org/docs/sql/query\_syntax/select*
`[2]`: *https://duckdb.org/docs/sql/introduction*
`[3]`: *https://duckdb.org/docs/sql/statements/pivot*
`[4]`: *https://duckdb.org/docs/sql/query\_syntax/qualify*
`[5]`: *https://duckdb.org/docs/sql/query\_syntax/select*
`[6]`: *https://duckdb.org/docs/data/csv/overview*