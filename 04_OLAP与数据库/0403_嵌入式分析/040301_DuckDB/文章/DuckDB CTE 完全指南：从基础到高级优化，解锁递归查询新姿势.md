---
title: DuckDB CTE 完全指南：从基础到高级优化，解锁递归查询新姿势
author: Hello大数据
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5MDE0MDE1OQ==&mid=2247484299&idx=1&sn=21fdb81d39004d9436d2e5e4d87e990a&chksm=919c4f03d1cf240f49ccb847125b17c09943ea2afdf5008b4cfc50855c40b7480fcc898b0dcb&mpshare=1&scene=24&srcid=0411U8N4kMeccKjEZSWhnSpb&sharer_shareinfo=6624edbab8a37132f22118fe59284351&sharer_shareinfo_first=6624edbab8a37132f22118fe59284351#rd
---

在数据查询与分析中，复杂逻辑的拆解与性能优化始终是核心需求。DuckDB 作为一款轻量、高性能的嵌入式分析型数据库，其对 CTE（公共表表达式）的支持不仅遵循 SQL 标准，更增加了 USING KEY 等独特优化特性，让递归查询、层次数据处理变得高效又简洁。本文将从基础概念出发，逐步深入 DuckDB CTE 的核心用法、高级特性与底层实现，帮你彻底掌握这一强大工具。

## 一、什么是 DuckDB CTE？

CTE（Common Table Expression，公共表表达式）是一种临时命名的结果集，通过 WITH 子句定义，可在后续查询中多次引用，核心作用是将复杂查询拆解为模块化、可读性更强的步骤。与传统子查询相比，CTE 不仅结构清晰，还能避免重复计算，同时支持递归查询，适配层次数据、图结构等复杂场景。

DuckDB 中的 CTE 分为两类：

* • 普通 CTE：非递归，适用于简单的查询拆解，比如将多步过滤、聚合逻辑拆分，提升代码可读性。
* • 递归 CTE：通过 WITH RECURSIVE 定义，包含锚点查询和递归查询两部分，专门用于处理树形结构、路径查找等递归场景。

基础语法示例（普通 CTE）：

```
-- 普通 CTE：拆解复杂查询  
WITH cte_name AS (  
    SELECT id, name, score FROM students WHERE score > 80  
)  
SELECT * FROM cte_name ORDER BY score DESC;
```

## 二、递归 CTE：核心用法与锚点查询

递归 CTE 是 DuckDB 处理层次数据的核心工具，比如组织架构遍历、目录结构查询、图路径查找等场景，都能通过它简洁实现。其核心结构由两部分组成：锚点查询（非递归部分）和递归查询（引用自身的部分）。

### 2.1 锚点查询：递归的“起点”

锚点查询是递归 CTE 的非递归部分，位于 UNION 或 UNION ALL 的左侧，作用是为整个递归过程提供初始数据集，同时定义 CTE 的列名和数据类型。简单来说，锚点查询就是递归的“起点”，没有它，递归就无从谈起。

从底层实现来看，DuckDB 在解析递归 CTE 时，会将锚点查询识别为左侧查询（left），并优先处理它，以此确定整个 CTE 的数据结构。例如：

```
WITH RECURSIVE employee_hierarchy AS (  
    -- 锚点查询：找到顶层管理者（无上级的员工）  
    SELECT employee_id, name, manager_id, 1 AS level  
    FROM employees   
    WHERE manager_id IS NULL  
      
    UNION ALL  
      
    -- 递归查询：找到所有下属，递归引用自身  
    SELECT e.employee_id, e.name, e.manager_id, h.level + 1  
    FROM employees e  
    JOIN employee_hierarchy h ON e.manager_id = h.employee_id  
)  
SELECT * FROM employee_hierarchy;
```

在这个例子中，锚点查询找到所有没有管理者的顶级员工，为后续递归查找下属关系提供了初始数据。DuckDB 会先执行锚点查询，再将其结果作为递归查询的输入，循环执行直到满足终止条件。

### 2.2 递归查询的终止条件

递归查询不会无限执行，其终止依赖两个核心机制：自然终止和显式条件控制，二者结合确保查询的安全性和高效性。

* • 自然终止：当递归查询的一次迭代没有产生任何新行时，自动结束。DuckDB 会在执行过程中检查每次迭代的结果行数，若为 0，则终止递归。
* • 显式条件控制：用户必须在递归查询的 WHERE 子句中添加限制条件，防止无限递归。这是最常用、最可控的终止方式，比如限制递归层次、数值范围等。

示例（显式条件控制终止）：

```
WITH RECURSIVE number_sequence AS (  
    SELECT 1 AS n  -- 锚点查询：初始值  
    UNION ALL  
    SELECT n + 1 FROM number_sequence WHERE n < 10  -- 显式条件：n<10，避免无限递归  
)  
SELECT * FROM number_sequence;
```

需要注意的是，递归查询部分不能使用 ORDER BY、LIMIT、OFFSET 等关键字，否则会抛出语法错误——这是 DuckDB 为了避免递归逻辑混乱而做的限制。

## 三、物化控制：平衡可读性与性能

CTE 的物化（Materialization）是指将 CTE 的结果集存储在临时表中，后续查询直接引用该临时表；而非物化（内联）则是将 CTE 逻辑直接替换到主查询中，相当于子查询的优化版本。DuckDB 提供了灵活的物化控制机制，可在语法层面精确配置，平衡查询可读性与性能。

### 3.1 物化控制的三种模式

DuckDB 支持三种物化模式，可通过 WITH 子句中的关键字显式配置：

1. 1. MATERIALIZED（强制物化）：无论 CTE 被引用多少次，都只计算一次并存储在临时表中，适合 CTE 逻辑复杂、被多次引用的场景，可避免重复计算。
2. 2. NOT MATERIALIZED（强制不物化）：将 CTE 逻辑内联到主查询中，适合 CTE 被单次引用、逻辑简单的场景，可减少临时表的开销。
3. 3. 默认模式（自动优化）：DuckDB 会根据 CTE 的引用次数、查询复杂度自动决定是否物化——单次引用通常内联，多次引用通常物化。

语法示例：

```
-- 强制物化  
WITH cte_name AS MATERIALIZED (SELECT * FROM large_table WHERE category = 'A')  
SELECT * FROM cte_name JOIN other_table ON cte_name.id = other_table.cte_id;  
  
-- 强制不物化  
WITH cte_name AS NOT MATERIALIZED (SELECT id, name FROM small_table)  
SELECT * FROM cte_name WHERE name LIKE '张%';
```

### 3.2 物化控制的配置位置

DuckDB 的物化控制主要通过 SQL 语法层面配置，没有全局系统设置——这种设计让用户可以为每个 CTE 单独配置物化行为，更具灵活性。其底层处理流程贯穿查询全阶段：

* • 语法解析：识别 MATERIALIZED/NOT MATERIALIZED 关键字，转换为内部枚举值。
* • 转换阶段：将语法树中的物化设置存储到 CommonTableExpressionInfo 结构中。
* • 优化阶段：CTE 内联优化器根据物化设置，决定是否将 CTE 内联到主查询。
* • 执行阶段：根据物化设置生成对应的物理执行计划（临时表存储或内联执行）。

## 四、USING KEY 优化：递归查询的性能利器

USING KEY 是 DuckDB 递归 CTE 的独特优化特性，专门用于解决递归查询中重复数据过多、内存开销大的问题。通过键列去重和负载聚合，USING KEY 能显著提升递归查询的性能，尤其适合图遍历、路径查找等场景。

### 4.1 核心原理：键列去重 + 负载聚合

USING KEY 的核心逻辑的是：指定一组“键列”，DuckDB 会通过哈希表对键列进行去重，同时对非键列（负载列）执行指定的聚合函数（如 max、min、avg 等），避免存储重复的中间结果，从而减少内存占用和计算开销。

语法示例（USING KEY 优化）：

```
WITH RECURSIVE tbl(a, b) USING KEY (a, max(b)) AS (  
    SELECT 1, 5  -- 锚点查询  
    UNION ALL  
    SELECT a, b - 1 FROM tbl WHERE b &gt; 0  -- 递归查询  
)  
TABLE tbl;
```

在这个例子中，键列是 a，负载列是 b，聚合函数是 max(b)。递归过程中，DuckDB 会对键列 a 去重，同时保留 b 的最大值，最终只返回一行结果（1, 5），避免了重复存储中间值（1,4）、（1,3）等。

### 4.2 关键特性与使用注意事项

* • 键列要求：USING KEY 子句必须包含至少一个键列，键列可以是普通列引用，不能是聚合函数。
* • 聚合支持：负载列支持多种聚合函数，如 max、min、avg、list、count 等，但不支持 FILTER、ORDER BY、DISTINCT 等修饰符。
* • 语法兼容性：当前版本中，USING KEY 推荐使用 UNION ALL，UNION 语法已被废弃，若使用会触发警告，可通过设置 deprecated\_using\_key\_syntax=&#39;UNION\_AS\_UNION\_ALL&#39; 临时兼容。
* • 适用场景：特别适合图遍历、路径查找、层次数据聚合等场景，能有效减少中间结果的存储和计算开销。

## 五、底层实现简析

了解 DuckDB CTE 的底层实现，能帮助我们更好地理解其性能特性和使用限制。其核心处理流程分为三个阶段：

1. 1. 解析阶段：将 SQL 中的 CTE 语法解析为内部数据结构（如 CommonTableExpressionInfo、RecursiveCTENode），识别锚点查询、递归查询、USING KEY、物化设置等信息。
2. 2. 绑定阶段：处理 CTE 的列类型、名称绑定，区分键列和负载列（USING KEY 场景），将递归 CTE 转换为逻辑执行计划。
3. 3. 执行阶段：生成物理执行计划，递归 CTE 通过迭代方式执行，USING KEY 场景下会创建 GroupedAggregateHashTable 进行去重和聚合，直到满足终止条件。

需要注意的是，DuckDB 不支持相互递归 CTE（即两个 CTE 互相引用），这是当前的一个限制，后续版本可能会优化。

## 六、总结与最佳实践

DuckDB 的 CTE 功能兼具灵活性和高性能，从普通查询拆接到复杂递归查询，从手动物化控制到 USING KEY 自动优化，覆盖了多种数据处理场景。结合实际使用场景，给出以下最佳实践：

1. 1. 复杂查询拆解：将多步过滤、聚合、关联逻辑拆分为多个 CTE，提升代码可读性和可维护性。
2. 2. 递归场景适配：处理树形结构、图路径等场景时，优先使用递归 CTE，通过锚点查询定义起点，显式条件控制终止。
3. 3. 物化策略选择：单次引用的简单 CTE 用 NOT MATERIALIZED 内联；多次引用、逻辑复杂的 CTE 用 MATERIALIZED 避免重复计算。
4. 4. 性能优化：递归查询中存在大量重复数据时，使用 USING KEY 进行去重和聚合，减少内存开销和计算时间。

总的来说，DuckDB 的 CTE 不仅遵循 SQL 标准，更通过 USING KEY 等独特优化，解决了传统递归查询的性能痛点。无论是日常数据分析还是复杂数据处理，掌握 CTE 的用法都能让你的查询更高效、更简洁。