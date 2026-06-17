---
title: DuckDB：扩展机制与生态系统—— DuckDB 的"插件商店"是怎么实现的？
author: MaxAiDB
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUwOTU4OTU2NQ==&mid=2247483716&idx=1&sn=e4cd99eefc2893302f879bb9d2b1ee55&chksm=f898113b9e5a11136a2134b8055fbd9ee792cd8b3a0f57f7f776c2e5c63cdb0b1530ce89e047&mpshare=1&scene=24&srcid=0412yOZPBr5C0vVc7MJCl4fd&sharer_shareinfo=05e1f911990d657662ce9ff77c53a5c0&sharer_shareinfo_first=05e1f911990d657662ce9ff77c53a5c0#rd
---

—— DuckDB 的"插件商店"是怎么实现的？

> 本文是《DuckDB：从上手到内核》系列的第 12 篇。前面的文章聚焦在 DuckDB 的核心引擎——存储、执行、优化、事务。从这篇开始，我们往外看：DuckDB 是怎么和外部世界连接的？

---

## 一行命令，能力翻倍

DuckDB 本体是一个精简的分析引擎。但通过扩展（Extension），它可以变得无所不能：

```
-- 读远程 S3 上的 Parquet 文件INSTALL httpfs; LOAD httpfs;SELECT * FROM read_parquet('s3://my-bucket/data.parquet');  
-- 读 PostgreSQL 的表INSTALL postgres_scanner; LOAD postgres_scanner;SELECT * FROM postgres_scan('host=localhost dbname=mydb', 'public', 'users');  
-- 全文搜索INSTALL fts; LOAD fts;PRAGMA create_fts_index('docs', 'id', 'content');
```

**两行命令（INSTALL + LOAD），就给 DuckDB 添加了一个全新的能力。** 这种设计让 DuckDB 的核心保持小巧（几 MB 的二进制文件），同时通过扩展生态覆盖几乎所有数据场景。

---

## 扩展系统的架构

```
  ┌──────────────────────────────────────────────────────┐  │                   DuckDB 核心引擎                      │  │  Parser → Binder → Optimizer → Executor → Storage    │  └───────────────────────┬──────────────────────────────┘                          │ ExtensionLoader API            ┌─────────────┼─────────────┐            │             │             │     ┌──────▼─────┐ ┌────▼─────┐ ┌────▼─────┐     │  内置扩展   │ │ 官方扩展  │ │ 社区扩展  │     │ (编译链接)  │ │(INSTALL)  │ │(自定义)   │     ├────────────┤ ├──────────┤ ├──────────┤     │ parquet    │ │ httpfs   │ │ 你的扩展  │     │ json       │ │ postgres │ │          │     │ icu        │ │ spatial  │ │          │     │ core_funcs │ │ delta    │ │          │     └────────────┘ └──────────┘ └──────────┘  
     图 1：DuckDB 扩展系统架构图
```

### Extension 基类

每个扩展都是一个实现了 `Extension` 接口的动态库（`src/include/duckdb/main/extension.hpp`）：

```
class Extension {public:    virtual void Load(ExtensionLoader &loader) = 0;  // 加载入口    virtual std::string Name() = 0;                   // 扩展名称    virtual std::string Version() const;              // 版本号};
```

`ExtensionLoader` 是扩展注册各种能力的 API，支持注册：

| 注册方法 | 能力 |
| --- | --- |
| `RegisterFunction(ScalarFunction)` | 标量函数（如 `json_extract`） |
| `RegisterFunction(AggregateFunction)` | 聚合函数（如 `approx_count_distinct`） |
| `RegisterFunction(TableFunction)` | 表函数（如 `read_parquet`） |
| `RegisterType(name, LogicalType)` | 自定义类型（如 `JSON`） |
| `RegisterCastFunction(...)` | 类型转换 |
| `RegisterFunction(CopyFunction)` | COPY 导入导出 |

### INSTALL / LOAD 工作流程

```
INSTALL httpfs;  → 从 DuckDB 扩展仓库下载 httpfs.duckdb_extension  → 验证签名和元数据（平台、版本兼容性）  → 存放到 ~/.duckdb/extensions/<version>/<platform>/  
LOAD httpfs;  → dlopen 加载动态库  → 查找入口函数: httpfs_duckdb_cpp_init (C++ ABI)                 或 httpfs_init_c_api (C ABI)  → 调用入口函数，传入 ExtensionLoader  → 扩展通过 ExtensionLoader 注册函数/类型  → 完成！新能力立即可用
```

DuckDB 支持两种扩展 ABI：

•**C++ ABI**：入口函数签名为 `void <name>_duckdb_cpp_init(ExtensionLoader &)`，需要编译器版本精确匹配•**C ABI**：入口函数签名为 `bool <name>_init_c_api(...)`，通过函数指针表交互，跨编译器兼容

---

## 内置扩展一览

DuckDB v1.5.0 源码中包含 11 个内置扩展目录：

| 扩展 | 功能 | 是否默认加载 |
| --- | --- | --- |
| **core\_functions** | 核心函数库（数学、字符串、日期等） | ✅ 自动 |
| **parquet** | Parquet 文件读写 | ✅ 自动 |
| **json** | JSON 文件读写 + JSON 函数 | ✅ 自动 |
| **icu** | 国际化排序和时区支持 | ✅ 自动 |
| **autocomplete** | SQL 自动补全 | ✅ 自动 |
| **delta** | Delta Lake 格式读取 | 需手动加载 |
| **jemalloc** | 高性能内存分配器 | 平台相关 |
| **tpch** | TPC-H 基准测试数据生成 | 需手动加载 |
| **tpcds** | TPC-DS 基准测试数据生成 | 需手动加载 |
| **demo\_capi** | C API 扩展示例 | 不加载 |
| **loader** | 扩展加载基础设施 | 内部使用 |

## 常用外部扩展

除了内置扩展，还有一大批通过 `INSTALL` 安装的官方和社区扩展：

| 扩展 | 功能 | 典型用途 |
| --- | --- | --- |
| **httpfs** | HTTP/S3/GCS 远程文件读取 | 查询云存储上的数据 |
| **postgres\_scanner** | 直接查询 PostgreSQL | 不用导数据就能分析 |
| **mysql\_scanner** | 直接查询 MySQL | 同上 |
| **sqlite\_scanner** | 直接查询 SQLite | 同上 |
| **spatial** | 地理空间函数（ST\_Point, ST\_Distance 等） | GIS 分析 |
| **fts** | 全文搜索（倒排索引） | 文本检索 |
| **iceberg** | Apache Iceberg 格式 | 数据湖查询 |
| **aws** | AWS 认证（配合 httpfs） | S3 数据访问 |
| **azure** | Azure Blob 认证 | Azure 数据访问 |
| **vss** | 向量相似性搜索 | AI/ML 向量检索 |
| **ducklake** | DuckLake 数据湖格式 | 数据湖管理 |

---

## Arrow 集成：零拷贝的秘密

DuckDB 和 Python/Pandas/Polars 之间传数据为什么这么快？因为它用了 **Apache Arrow C Data Interface** 实现零拷贝。

### nanoarrow：轻量级选择

DuckDB 没有依赖完整的 Apache Arrow C++ 库（那个库有几百 MB），而是用了 **nanoarrow**——一个轻量级的纯 C 实现。这和 DuckDB "零依赖"的设计哲学一致。

关键组件：

•**ArrowConverter**：DuckDB → Arrow 转换（`ToArrowSchema()` + `ToArrowArray()`）•**ArrowAppender**：增量构建 Arrow 数组•**Arrow C Data Interface**：标准化的内存布局约定，不同系统间直接共享内存指针

```
import duckdbimport pyarrow as pa  
# DuckDB → Arrow → Pandas（零拷贝链路）result = duckdb.sql("SELECT * FROM range(1000000) t(id)")arrow_table = result.fetch_arrow_table()  # 零拷贝！df = arrow_table.to_pandas()              # 零拷贝！
```

"零拷贝"的含义是：**数据在内存中的位置没有变**——DuckDB 把自己内存中数据的指针直接传给了 Arrow，Arrow 再把指针传给 Pandas。没有 `memcpy`，没有序列化。

---

## DuckDB 生态全景

```
  ┌─────────────────────────────────────────────────────────────┐  │                    DuckDB 核心引擎                            │  └───────┬─────────────┬──────────────┬───────────────────────┘          │             │              │   ┌──────▼──────┐  ┌──▼──────┐  ┌───▼────────┐   │ 多语言绑定   │  │数据格式  │  │ 云存储/DB   │   ├─────────────┤  ├─────────┤  ├────────────┤   │ Python      │  │ Parquet │  │ S3/GCS     │   │ R           │  │ JSON    │  │ PostgreSQL │   │ Java/JDBC   │  │ CSV     │  │ MySQL      │   │ Node.js     │  │ Delta   │  │ SQLite     │   │ Go          │  │ Iceberg │  │ Azure Blob │   │ Rust        │  │ Arrow   │  │ HTTP(S)    │   │ WASM        │  │ Excel   │  └────────────┘   │ Swift       │  └─────────┘   │ C/C++       │   └─────────────┘  
   ┌──────────────────────────────────────────┐   │             工具集成                       │   ├──────────────────────────────────────────┤   │ dbt-duckdb │ SQLAlchemy │ Jupyter Magic  │   │ Grafana    │ Metabase   │ DBeaver        │   │ Superset   │ Observable │ MotherDuck     │   └──────────────────────────────────────────┘  
   图 2：DuckDB 生态全景图
```

---

## 动手实验

### 实验 1：查看已安装的扩展

```
-- 查看所有已加载的扩展SELECT * FROM duckdb_extensions() WHERE loaded = true;  
-- 查看所有可用的扩展SELECT extension_name, installed, loaded, descriptionFROM duckdb_extensions()ORDER BY extension_name;
```

### 实验 2：使用 httpfs 查询远程数据

```
-- 安装并加载 httpfsINSTALL httpfs;LOAD httpfs;  
-- 直接查询 DuckDB 官方提供的远程 Parquet 示例SELECT COUNT(*), AVG(total_amount)FROM read_parquet('https://shell.duckdb.org/data/tpch/0_01/parquet/lineitem.parquet');
```

### 实验 3：跨数据库查询

```
-- 附加一个 SQLite 数据库INSTALL sqlite_scanner;LOAD sqlite_scanner;ATTACH 'my_sqlite.db' AS sqlite_db (TYPE sqlite);  
-- 直接查询 SQLite 中的表SELECT * FROM sqlite_db.main.users LIMIT 10;  
-- 跨数据库 JOIN：DuckDB 表 JOIN SQLite 表SELECT d.*, s.nameFROM my_duckdb_table dJOIN sqlite_db.main.users s ON d.user_id = s.id;
```

---

## 小结

1.**DuckDB 的扩展系统基于动态加载**——`INSTALL` 下载，`LOAD` 加载，两行命令添加新能力。2.**Extension 接口简洁**：实现 `Load()` 和 `Name()` 两个方法，通过 `ExtensionLoader` 注册函数、类型、表函数等。3.**支持 C++ ABI 和 C ABI 两种扩展接口**——C ABI 跨编译器兼容。4.**内置 11 个源码扩展**，另有 15+ 个官方外部扩展通过 INSTALL 安装。5.**nanoarrow 实现零拷贝**——DuckDB ↔ Arrow ↔ Pandas/Polars 之间不复制数据。6.**多语言绑定覆盖 Python、R、Java、Node.js、Go、Rust、WASM、Swift、C/C++**——几乎所有主流语言。

---

**Q1：INSTALL 和 LOAD 有什么区别？为什么要两步？**

`INSTALL` 是下载——把扩展文件从网上下载到本地。`LOAD` 是加载——把已下载的扩展载入当前会话。分开的好处是：下载一次，以后每次只需要 LOAD 就行了。而且 DuckDB 现在支持自动加载（`autoload_known_extensions = true`），常用扩展会在需要时自动 LOAD。

**Q2：所有扩展都是免费的吗？**

•DuckDB 本身和所有官方扩展都是免费开源的（MIT 协议）。MotherDuck 是商业化的云服务。

---

## 延伸阅读

**源码路径：**

•Extension 基类：`src/include/duckdb/main/extension.hpp`•ExtensionLoader：`src/main/extension/extension_loader.cpp`•LOAD 实现：`src/main/extension/extension_load.cpp`•INSTALL 实现：`src/main/extension/extension_install.cpp`•nanoarrow：`src/include/duckdb/common/arrow/nanoarrow/`•Arrow 转换：`src/include/duckdb/common/arrow/arrow_converter.hpp`

**参考资料：**

•DuckDB 官方文档 - 扩展：https://duckdb.org/docs/extensions/overview[1]•DuckDB 官方博客 - Extension System：https://duckdb.org/2024/07/05/community-extensions.html[2]•Apache Arrow C Data Interface：https://arrow.apache.org/docs/format/CDataInterface.html[3]•nanoarrow：https://github.com/apache/arrow-nanoarrow[4]

### References

`[1]`: *https://duckdb.org/docs/extensions/overview*  
`[2]`: *https://duckdb.org/2024/07/05/community-extensions.html*  
`[3]`: *https://arrow.apache.org/docs/format/CDataInterface.html*  
`[4]`: *https://github.com/apache/arrow-nanoarrow*