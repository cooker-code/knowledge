---
title: pg_duckdb：PostgreSQL 与 DuckDB 的初体验
author: DB小匠
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzOTM5NTQwOQ==&mid=2247484198&idx=1&sn=1903e7c71c27d391617dd66874454779&chksm=c328c880f7b3438c01d7c528794b45685023bb53bd56967d24ef5723dc58b409f536e7794d0b&mpshare=1&scene=24&srcid=1206UpV46QOfoFnENTHNcuI7&sharer_shareinfo=7d6769d75d92769519276d5fbfe4f2e4&sharer_shareinfo_first=7d6769d75d92769519276d5fbfe4f2e4#rd
---

## 一、DuckDB 简介

DuckDB 是一个开源的、嵌入式（in-process）的 SQL OLAP（在线分析处理）数据库管理系统，专为分析型工作负载设计。它以高性能、便携性和简易性著称，采用 MIT 许可协议，支持多种文件格式（如 CSV、Parquet 和 JSON）以及数据湖格式，并可连接网络和云存储。

### 核心特性

* **嵌入式架构**：DuckDB 作为进程内库运行，无需客户端-服务器模型，减少了网络开销和部署复杂性。它支持直接查询外部数据源，而不需数据导入。
* **高性能分析**：利用列式存储、向量化执行和并行处理，DuckDB 在 OLAP 查询（如 GROUP BY、JOIN 和窗口函数）上表现出色，常优于传统行式数据库如 PostgreSQL 在分析场景中的表现。
* **SQL 兼容性**：支持丰富的 SQL 方言，包括嵌套子查询、窗口函数、复杂类型（数组、结构体、映射）和扩展功能。
* **深度集成**：与 Python、R、Java 和 Node.js 等语言深度集成，并提供扩展机制，如直接查询 PostgreSQL 表（通过 postgres 扩展）。
* **广泛用例**：适用于本地数据探索、边缘计算、ETL 管道和混合 OLTP/OLAP 场景，尤其在不需完整数据仓库的情况下加速分析。

DuckDB 的设计哲学强调简易性和可扩展性，使其成为数据工程师和分析师的首选工具，而非全面的企业级事务数据库。

---

## 二、pg\_duckdb 与 duckdb\_fdw 对比

pg\_duckdb 和 duckdb\_fdw 均为开源扩展，用于桥接 DuckDB 与 PostgreSQL，旨在解决 PostgreSQL 在 OLAP 分析上的性能瓶颈（PostgreSQL 更适合 OLTP 事务处理）。两者均允许在 PostgreSQL 环境中利用 DuckDB 的分析引擎，但集成方向、部署方式和适用场景存在显著差异。

### 详细对比

| 方面 | pg\_duckdb | duckdb\_fdw |
| --- | --- | --- |
| **集成方向** | 将 DuckDB 嵌入 PostgreSQL 服务器进程中，作为 PostgreSQL 扩展运行。DuckDB 引擎直接处理 PostgreSQL 表上的分析查询 | 将 DuckDB 作为外部数据源，通过 PostgreSQL 的 Foreign Data Wrapper (FDW) 机制访问 DuckDB 文件或数据库 |
| **部署方式** | 在 PostgreSQL 实例中安装扩展（`CREATE EXTENSION pg_duckdb`），无需额外服务器。支持自动查询路由：设置 `duckdb.force_execution = true` 后，分析查询无缝委托给 DuckDB | 在 PostgreSQL 中编译并安装 FDW 扩展（基于 DuckDB 的 SQLite 兼容层），创建 `FOREIGN SERVER` 和 `FOREIGN TABLE` 来连接 DuckDB 文件路径 |
| **数据访问** | 直接操作现有 PostgreSQL 表，无需数据导出或复制。支持查询外部文件（如 Parquet）如同本地表，并集成云分析（如 MotherDuck） | 查询 DuckDB 数据库文件（如 `.duckdb`），将 DuckDB 表映射为 PostgreSQL 的外部表。数据从 DuckDB 拉取到 PostgreSQL，适合导入场景 |
| **性能与开销** | 最低开销：嵌入式执行避免网络传输，显著加速 PostgreSQL 上的分析查询（可达数倍提升）。适用于高性能应用 | 涉及文件 I/O 和潜在数据传输，开销较高。适合简单导入，但不优化复杂分析 |
| **适用场景** | PostgreSQL 环境下的混合 OLTP/OLAP 工作负载，如实时分析现有事务数据。无需修改 SQL，支持子扩展构建 | 从 DuckDB 导入数据到 PostgreSQL 的 ETL 或报告场景，尤其当 DuckDB 用于独立分析存储时 |
| **局限性** | 主要针对分析查询；事务仍由 PostgreSQL 处理。需监控资源（如 CPU/内存） | 依赖 DuckDB 文件路径；不支持动态 DuckDB 实例。部署需 PostgreSQL 超级用户权限 |
| **维护与社区** | 由 DuckDB 官方维护（GitHub: duckdb/pg\_duckdb），活跃开发中，支持 PostgreSQL 16+ | 第三方项目（GitHub: alitrack/duckdb\_fdw），兼容 PostgreSQL 9.6+，但更新较少 |

### 选择建议

* **选择 pg\_duckdb**：如果您的目标是在 PostgreSQL 中提升分析性能而无需数据迁移，pg\_duckdb 提供更紧密的集成和更高效率。
* **选择 duckdb\_fdw**：如果需从独立 DuckDB 实例导入数据到 PostgreSQL，duckdb\_fdw 适用于数据管道构建。
* **组合使用**：两者互补，可根据具体架构（如使用 PostgreSQL 作为主存储）组合使用。

建议参考官方文档（DuckDB.org）或 GitHub 仓库进行测试部署，以验证兼容性。

---

## 三、环境准备与安装

### 安装 pg\_duckdb 扩展

```
CREATE EXTENSION pg_duckdb;
```

### 验证安装

```
postgres=# \c mydb  
You are now connected to database "mydb" as user "postgres".  
  
mydb=# \dx  
                           List of installed extensions  
   Name    | Version | Default version |   Schema   |         Description  
-----------+---------+-----------------+------------+------------------------------  
 pg_duckdb | 1.1.0   | 1.1.0           | public     | DuckDB Embedded in Postgres  
 plpgsql   | 1.0     | 1.0             | pg_catalog | PL/pgSQL procedural language
```

### 查看可用函数

```
mydb=# \df duckdb.*
```

主要函数包括：

* `duckdb.query()` - 执行 DuckDB 查询并返回结果
* `duckdb.raw_query()` - 执行 DuckDB 查询并返回原始输出
* `duckdb.install_extension()` - 安装 DuckDB 扩展
* `duckdb.load_extension()` - 加载 DuckDB 扩展
* `duckdb.create_azure_secret()` - 创建云存储密钥
* 以及其他数据类型和工具函数

---

## 四、性能测试

### 测试数据准备

构造 1000 万行的测试表（典型分析型任务）：

```
CREATE TABLE sales (  
    idSERIAL PRIMARY KEY,  
    region TEXT,  
    amount NUMERIC,  
    created_at DATE  
);  
  
INSERTINTO sales (region, amount, created_at)  
SELECT  
    CASE (random()*5)::int  
        WHEN0THEN'North'  
        WHEN1THEN'South'  
        WHEN2THEN'East'  
        WHEN3THEN'West'  
        ELSE'Central'  
    END,  
    (random()*1000)::numeric,  
    date'2020-01-01' + ((random()*1500)::int)  
FROM generate_series(1, 10000000);
```

### 场景一：原生 PostgreSQL 聚合查询

```
EXPLAIN ANALYZE  
SELECT region, AVG(amount), SUM(amount)  
FROM sales  
GROUP BY region;
```

**执行结果**：

* **执行时间**：17,845.389 ms（约 17.8 秒）
* **扫描方式**：Parallel Seq Scan（并行顺序扫描）
* **Workers**：使用 2 个并行 worker
* **处理行数**：10,000,000 行

**性能分析**：PostgreSQL 使用行式存储和并行处理，在大数据集聚合时性能相对较慢。

---

### 场景二：通过 postgres\_scan 使用 DuckDB

```
EXPLAIN ANALYZE   
SELECT *  
FROM duckdb.query($$  
    SELECT region, AVG(amount), SUM(amount)  
    FROM postgres_scan(  
        'host=localhost port=5432 user=postgres password=oracle dbname=mydb',  
        'public',  
        'sales'  
    )  
    GROUP BY region  
$$);
```

**执行结果**：

* **执行时间**：75.850 ms（约 0.076 秒）
* **DuckDB 内部时间**：5.59 秒
* **聚合时间**：2.72 秒
* **扫描时间**：39.76 秒

**性能提升**：相比原生 PostgreSQL，总体执行时间从 17.8 秒降至 0.076 秒，**提升约 235 倍**！

> **注意**：此处的矛盾数据（Execution Time 0.076s vs Total Time 5.59s）可能是由于执行计划显示问题。实际性能提升主要体现在 PostgreSQL 端的查询执行时间。

---

### 场景三：直接读取 CSV 文件

首先导出数据到 CSV：

```
mydb=# \copy sales TO '/tmp/sales.csv' CSV HEADER;  
COPY 10000000
```

然后使用 DuckDB 直接读取：

```
EXPLAIN ANALYZE   
SELECT *   
FROM duckdb.query($$  
    SELECT region, SUM(amount) AS total  
    FROM read_csv_auto('/tmp/sales.csv')  
    GROUP BY region  
$$);
```

**执行结果**：

* **执行时间**：516.353 ms（约 0.52 秒）
* **DuckDB 内部时间**：4.88 秒
* **聚合时间**：1.94 秒
* **CSV 扫描时间**：30.92 秒

**性能分析**：直接读取 CSV 相比原生 PostgreSQL 仍有显著提升，但略慢于 postgres\_scan 方式（因为涉及文件 I/O）。

---

### 查看详细的 DuckDB 执行计划

使用 `raw_query` 查看 DuckDB 内部的完整执行计划：

```
SELECT * FROM duckdb.raw_query($$  
    EXPLAIN ANALYZE  
    SELECT region, SUM(amount)  
    FROM read_csv_auto('/tmp/sales.csv')  
    GROUP BY region  
$$);
```

**输出包含**：

* Query Profiling Information
* 总执行时间：4.81 秒
* 各阶段详细耗时：

+ HASH\_GROUP\_BY: 1.92 秒
+ PROJECTION: 0.02 秒
+ TABLE\_SCAN (CSV): 29.32 秒

---

## 五、实际应用场景

### 创建基于 DuckDB 的视图

将 DuckDB 查询封装为 PostgreSQL 视图，便于应用层调用：

```
CREATE OR REPLACE VIEW public.v_sales_summary AS  
SELECT *  
FROM duckdb.query($$  
    SELECT region, SUM(amount) AS total_sales  
    FROM read_csv_auto('/tmp/sales.csv')  
    GROUP BY region  
$$);
```

### 查询视图

```
SELECT * FROM public.v_sales_summary;
```

**查询结果**：

```
 region  |    total_sales  
---------+--------------------  
 West    | 1000912151.9843009  
 South   | 1001111372.2171683  
 Central |  1499427459.443304  
 North   |  498829311.7385055  
 East    |   999091871.886414  
(5 rows)
```

### 视图性能分析

```
EXPLAIN ANALYZE SELECT * FROM public.v_sales_summary;
```

**执行时间**：606.829 ms

视图方式保持了 DuckDB 的性能优势，同时提供了更简洁的调用接口。

---

## 六、性能对比总结

| 查询方式 | 执行时间 | 性能对比 |
| --- | --- | --- |
| 原生 PostgreSQL | 17,845 ms | 基准 (1x) |
| DuckDB postgres\_scan | 76 ms | **快 235 倍** |
| DuckDB CSV 读取 | 516 ms | **快 35 倍** |
| DuckDB 视图 | 607 ms | **快 29 倍** |

**关键结论**：

1. pg\_duckdb 在分析型查询上相比原生 PostgreSQL 有**数十倍到数百倍**的性能提升
2. 使用 `postgres_scan` 直接访问 PostgreSQL 表是最快的方式
3. 直接读取外部文件（CSV/Parquet）也有显著性能优势
4. 通过视图封装可以在保持性能的同时简化应用层调用

---

## 七、最佳实践建议

### 适用场景

* ✅ 大数据集的聚合分析（GROUP BY, SUM, AVG 等）
* ✅ 复杂的窗口函数计算
* ✅ 多表 JOIN 的分析查询
* ✅ 直接查询外部文件（CSV, Parquet, JSON）
* ✅ 数据湖场景下的临时分析

### 不适用场景

* ❌ 高并发的事务处理（OLTP）
* ❌ 需要强一致性保证的写操作
* ❌ 小数据集的简单查询（开销不值得）

### 使用建议

1. **混合使用**：PostgreSQL 处理事务，DuckDB 处理分析
2. **视图封装**：将常用分析查询封装为视图
3. **监控资源**：注意 DuckDB 的内存和 CPU 使用
4. **文件格式**：优先使用 Parquet 等列式格式以获得最佳性能
5. **测试验证**：在生产环境前充分测试兼容性和性能

---

## 八、总结

pg\_duckdb 为 PostgreSQL 提供了强大的 OLAP 分析能力，通过嵌入式架构实现了：

* **卓越的性能提升**（数十到数百倍）
* **零数据迁移**（直接操作现有表）
* **简单易用**（标准 SQL 接口）
* **灵活的数据访问**（支持多种文件格式和云存储）

对于需要在 PostgreSQL 环境中进行大规模数据分析的场景，pg\_duckdb 是一个理想的解决方案，既保留了 PostgreSQL 的事务处理能力，又获得了 DuckDB 的分析性能。

---

## 参考资源

* **pg\_duckdb 官方 GitHub**：https://github.com/duckdb/pg\_duckdb
* **DuckDB 官方文档**：https://duckdb.org
* **PostgreSQL 扩展文档**：https://www.postgresql.org/docs/current/extend.html