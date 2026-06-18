# DuckDB
## 知识点入口

- 本模块先看宏观流程，再看文章：[知识地图](040301_核心知识点/知识地图.md)。
- 新文章必须先归入流程节点，再判断是补充、冲突、不同层次还是降权。
- `文章/` 只保留原文锚点，长期知识必须沉淀到 `040301_核心知识点/`。


## 技术定位

| 项 | 内容 |
|---|---|
| 技术名 | DuckDB |
| 一级类目 | OLAP 与数据库 |
| 二级类目 | 嵌入式分析 |
| 技术本体 | 面向本地分析、嵌入式分析和列式 OLAP 的进程内数据库 |
| 全局架构位置 | 位于应用进程、Notebook、本地文件和轻量分析任务之间，承担本地列式查询和分析计算 |
| 主要使用者 | 数据分析师、数据工程师、后端工程师、分析应用工程师 |
| 主要产出 | 本地查询结果、Parquet/CSV 分析、嵌入式分析能力 |

## 官方锚点

- 官网：[DuckDB](https://duckdb.org/)
- GitHub：[duckdb/duckdb](https://github.com/duckdb/duckdb)
- 官方文档：[DuckDB Docs](https://duckdb.org/docs/)

## 架构图

```mermaid
flowchart LR
  App["Python / R / CLI / App"] --> Duck["DuckDB 引擎"]
  Files["CSV / Parquet / Arrow / 本地文件"] --> Duck
  Duck --> Exec["向量化执行\nDataChunk / Pipeline"]
  Duck --> Txn["ACID / MVCC"]
  Exec --> Result["本地分析结果"]
```

## 核心模块

| 模块 | 职责 | 重点问题 |
|---|---|---|
| 列式存储 | 提高分析扫描效率 | 压缩、文件格式、Parquet 互操作 |
| 向量化执行 | 批量处理 DataChunk | CPU 缓存、SIMD、pipeline breaker |
| 优化器 | 改写和优化 SQL | 统计信息、join order、谓词下推 |
| 扩展机制 | 扩展数据源和功能 | 安全、版本、生态质量 |
| ACID/MVCC | 嵌入式事务能力 | 并发边界、写入模式 |

## 横向对标

| 对标技术 | 对标点 | DuckDB 优势 | DuckDB 劣势 | 使用判断 |
|---|---|---|---|---|
| SQLite | 嵌入式数据库 | DuckDB 更适合 OLAP 和列式扫描 | SQLite OLTP 生态更强 | 本地分析用 DuckDB，轻事务应用用 SQLite |
| PostgreSQL | 通用关系数据库 | DuckDB 本地分析轻量 | 不适合多用户服务型 OLTP | 服务端业务库用 PostgreSQL |
| ClickHouse | 列式 OLAP | DuckDB 部署轻、嵌入式 | 不适合大规模服务化集群 | 单机/本地分析用 DuckDB，集群查询用 ClickHouse |
| Pandas | 本地数据分析 | DuckDB SQL 和文件扫描强 | DataFrame 生态交互不同 | 大文件 SQL 分析可优先 DuckDB |

## 已沉淀核心知识点

| 主题 | 文件 | 问题指纹 | 解决什么问题 | 认知增量 |
|---|---|---|---|---|
| 向量化执行与 Pipeline | [DuckDB向量化执行与Pipeline](040301_核心知识点/DuckDB向量化执行与Pipeline.md) | DuckDB + 执行引擎 + DataChunk/Vector/Push-Based Pipeline + 本地分析性能 + 不等同传统行式执行 | DuckDB 为什么能在本地分析里快 | 把 DuckDB 定位从“轻量数据库”校准为“嵌入式列式分析引擎” |

## 后续追查

- 关键词：DataChunk、Vectorized Execution、Push-Based Pipeline、Pipeline Breaker、Morsel-Driven Parallelism。
- 待读资料：DuckDB 优化器、MVCC、扩展机制、pg_duckdb。
- 待补实验：用 CSV/Parquet 和 Pandas/PostgreSQL 对比本地聚合、过滤和 Join。
