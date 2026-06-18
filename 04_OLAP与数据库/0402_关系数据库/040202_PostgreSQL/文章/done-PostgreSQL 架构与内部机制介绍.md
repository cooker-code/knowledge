> 已吸收至：[[04_OLAP与数据库/0402_关系数据库/040202_PostgreSQL/040202_核心知识点/PostgreSQL架构WAL与高可用边界|PostgreSQL架构WAL与高可用边界]]
---
title: PostgreSQL 架构与内部机制介绍
author: DB小匠
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzOTM5NTQwOQ==&mid=2247484202&idx=1&sn=af2dd55a4f26c957cc9201dda9ce65ba&chksm=c35d18df33d8fcb65278b817cce7c52472d8e470dd43d7e77b2c8d0145f29f9724d54fe34fde&mpshare=1&scene=24&srcid=1206pw0v81MNdQpgP4pM4mhd&sharer_shareinfo=64db9550caf6bc46e14842fb80061687&sharer_shareinfo_first=64db9550caf6bc46e14842fb80061687#rd
---

## 引言

PostgreSQL 是一款功能强大的开源关系型数据库管理系统，其架构设计体现了关系理论与实际工程的完美结合。本文将深入探讨 PostgreSQL 的数据组织、进程模型、内存管理以及客户端-服务端交互机制，帮助您全面理解 PostgreSQL 的内部工作原理。

---

## 一、数据组织架构

### 1.1 数据库与集簇

**核心概念**:

* **PostgreSQL 实例**: 运行中的 PostgreSQL 服务进程
* **数据库集簇**: 单个实例管理的所有数据库的集合
* **PGDATA**: 存储所有集簇相关文件的目录

**集簇初始化后的默认数据库**:

```
-- 初始化集簇后会创建三个数据库
```

| 数据库 | 用途 | 是否可修改 |
| --- | --- | --- |
| **template0** | 用于备份恢复、创建特殊编码数据库 | ❌ 禁止修改 |
| **template1** | 用户创建新数据库的模板 | ✅ 可以修改 |
| **postgres** | 常规数据库，供日常使用 | ✅ 可以修改 |

**架构示意**:

```
PostgreSQL 实例
    ├── template0 (只读模板)
    ├── template1 (可修改模板)
    └── postgres (常规数据库)
```

**创建新数据库示例**:

```
-- 基于 template1 创建新数据库
CREATE DATABASE mydb;

-- 基于 template0 创建指定编码的数据库
CREATE DATABASE mydb_utf8
    ENCODING 'UTF8'
    TEMPLATE template0;
```

### 1.2 系统目录

**定义**: 系统目录是一组特殊的表和视图，存储所有数据库对象的元数据。

**系统目录特点**:

* 每个数据库有自己的系统目录表集合
* 某些系统目录表在整个集簇共享（使用虚拟数据库 ID 0）
* 所有系统表名以 `pg_` 开头

**命名规范**:

```
-- 表名: pg_database (系统目录表)
-- 列名: datname (dat 前缀对应表名)
-- 主键: oid (对象标识符, 32位整数)
```

**查看数据库信息示例**:

```
-- 查询所有数据库
SELECT oid, datname, datdba, encoding 
FROM pg_database;
```

**示例输出**:

```
  oid  |  datname  | datdba | encoding
-------+-----------+--------+----------
 13442 | template0 |     10 |        6
 13443 | template1 |     10 |        6
 16384 | postgres  |     10 |        6
```

**OID 分配机制**:

* 由公共计数器生成唯一 ID
* 类似序列但更早出现
* 达到最大值后重置，通过唯一索引确保唯一性

**查看系统目录表**:

```
-- 查看所有系统目录表
SELECT tablename 
FROM pg_tables 
WHERE schemaname = 'pg_catalog'
ORDER BY tablename;

-- 常用系统目录表
-- pg_database: 数据库信息
-- pg_class: 表、索引、视图等关系信息
-- pg_attribute: 列信息
-- pg_index: 索引信息
-- pg_proc: 函数和存储过程
```

### 1.3 模式(Schema)

**定义**: 模式是数据库内对象的命名空间，用于组织和隔离数据库对象。

**预定义模式**:

| 模式名 | 用途 |
| --- | --- |
| **public** | 用户对象的默认模式 |
| **pg\_catalog** | 系统目录表 |
| **information\_schema** | SQL 标准定义的系统视图 |
| **pg\_toast** | TOAST 相关对象 |
| **pg\_temp** | 临时表（每个用户有 pg\_temp\_N） |

**搜索路径机制**:

```
-- 查看当前搜索路径
SHOW search_path;
-- 结果: "$user", public

-- 设置搜索路径
SET search_path TO myschema, public;

-- search_path 隐式包含 pg_catalog 和 pg_temp
```

**模式使用示例**:

```
-- 创建模式
CREATESCHEMA sales;
CREATESCHEMA hr;

-- 在不同模式中创建同名表
CREATETABLE sales.orders (idint, amount numeric);
CREATETABLE hr.orders (idint, description text);

-- 不指定模式时使用搜索路径
SELECT * FROM orders;  -- 根据 search_path 查找

-- 完全限定名
SELECT * FROM sales.orders;
SELECT * FROM hr.orders;
```

**模式的优势**:

* 避免命名冲突
* 组织相关对象
* 简化权限管理
* 支持多租户架构

### 1.4 表空间(Tablespace)

**定义**: 表空间定义数据的物理存储位置，是文件系统中的目录。

**逻辑与物理分离**:

```
逻辑层: 数据库 → 模式 → 表
物理层: 表空间 → 目录 → 文件
```

**默认表空间**:

| 表空间名 | 物理位置 | 用途 |
| --- | --- | --- |
| **pg\_default** | `PGDATA/base/` | 默认用户对象存储 |
| **pg\_global** | `PGDATA/global/` | 集簇共享的系统目录 |

**表空间架构图**:

```
┌─────────────────────────────────────────────┐
│              数据库集簇                      │
├─────────────────────────────────────────────┤
│                                             │
│  ┌──────────────┐      ┌──────────────┐   │
│  │ postgres DB  │      │ mydb DB      │   │
│  │              │      │              │   │
│  │ public schema│      │ public schema│   │
│  │   ├─ orders  │      │   ├─ users   │   │
│  │   └─ products│      │   └─ logs    │   │
│  └──────┬───────┘      └──────┬───────┘   │
│         │                     │            │
│         └──────┬──────────────┘            │
│                ↓                           │
│  ┌─────────────────────────────┐          │
│  │     表空间映射               │          │
│  │  pg_default → PGDATA/base/  │          │
│  │  xyzzy → /mnt/ssd/pgdata/   │          │
│  └─────────────────────────────┘          │
└─────────────────────────────────────────────┘
```

**创建和使用表空间**:

```
-- 创建自定义表空间
CREATETABLESPACE fast_storage 
    LOCATION '/mnt/ssd/pgdata';

CREATETABLESPACE archive_storage 
    LOCATION '/mnt/hdd/pgarchive';

-- 在表空间中创建数据库
CREATEDATABASE analytics
    TABLESPACE fast_storage;

-- 在表空间中创建表
CREATETABLE hot_data (idint, datatext)
    TABLESPACE fast_storage;

CREATETABLE cold_data (idint, datatext)
    TABLESPACE archive_storage;

-- 查看表空间
SELECT spcname, pg_tablespace_location(oid) 
FROM pg_tablespace;
```

**表空间的应用场景**:

* 将热数据放在 SSD，冷数据放在 HDD
* 分散 I/O 负载到不同磁盘
* 隔离不同租户的数据
* 实现分级存储策略

**符号链接机制**:

```
# PostgreSQL 在 PGDATA/pg_tblspc/ 创建符号链接
ls -l $PGDATA/pg_tblspc/
# 16385 -> /mnt/ssd/pgdata
# 16386 -> /mnt/hdd/pgarchive
```

### 1.5 关系(Relations)

**定义**: PostgreSQL 中将表、索引、序列、物化视图等统称为"关系"。

**关系的类型**:

| 类型 | 存储数据 | 行结构 | relkind 值 |
| --- | --- | --- | --- |
| **表** | ✅ | 是 | r |
| **索引** | ✅ | 是 | i |
| **序列** | ✅ | 是（单行） | S |
| **物化视图** | ✅ | 是 | m |
| **视图** | ❌ | - | v |
| **外部表** | ❌ | - | f |

**系统目录表命名历史**:

* 原名: `pg_relation`
* 现名: `pg_class` (面向对象趋势影响)
* 列前缀: `rel` (保留历史遗产)

**查看关系信息**:

```
-- 查看数据库中的所有关系
SELECT relname, relkind, 
    CASE relkind
        WHEN'r'THEN'table'
        WHEN'i'THEN'index'
        WHEN'S'THEN'sequence'
        WHEN'v'THEN'view'
        WHEN'm'THEN'materialized view'
        WHEN'f'THEN'foreign table'
    ENDAStype
FROM pg_class
WHERE relnamespace = 'public'::regnamespace;
```

**关系的共同点**:

* 都由行组成（即使是 B 树节点）
* 使用统一的页面结构
* 通过 OID 唯一标识

### 1.6 文件和分支

**分支的概念**: 每个关系包含多个分支(fork)，每个分支存储特定类型的数据。

**文件命名规则**:

```
基本格式: <relfilenode>[.<fork_suffix>][.<segment_number>]

示例:
16385           # 主分支第一个段
16385.1         # 主分支第二个段
16385_fsm       # 空闲空间映射
16385_vm        # 可见性映射
16385_init      # 初始分支（无日志表）
```

**文件分段机制**:

* 单个文件最大 1GB
* 超过后创建新段，文件名后添加序号
* 可在编译时修改：`./configure --with-segsize`

**分支类型详解**:

#### 1.6.1 主分支(Main Fork)

**用途**: 存储实际数据（表行或索引行）

**查看文件路径示例**:

```
-- 创建测试表
CREATE UNLOGGED TABLE t(
    a integer,
    b numeric,
    c text,
    d json
);

INSERT INTO t VALUES (1, 2.0, 'foo', '{}');

-- 获取文件路径
SELECT pg_relation_filepath('t');
```

**输出**:

```
 pg_relation_filepath
----------------------
 base/16384/16385
```

**路径解析**:

```
base/         → pg_default 表空间
     16384/   → 数据库 OID
          16385 → 表的 relfilenode
```

**验证路径**:

```
-- 确认数据库 OID
SELECT oid FROM pg_database WHERE datname = 'internals';
-- 结果: 16384

-- 确认表的 relfilenode
SELECT relfilenode FROM pg_class WHERE relname = 't';
-- 结果: 16385

-- 查看文件大小
SELECT size
FROM pg_stat_file('/usr/local/pgsql/data/base/16384/16385');
-- 结果: 8192
```

#### 1.6.2 初始分支(Init Fork)

**用途**: 仅用于无日志表(UNLOGGED)和索引

**特点**:

* 无日志表不写 WAL，速度快但崩溃后数据丢失
* 恢复时用初始分支覆盖主分支，创建空表

**示例**:

```
-- 查看初始分支（上面的表 t 是 UNLOGGED）
SELECT size
FROM pg_stat_file('/usr/local/pgsql/data/base/16384/16385_init');
-- 结果: 0 (空文件)
```

**无日志表的优缺点**:

| 优点 | 缺点 |
| --- | --- |
| 写入速度快 | 崩溃后数据丢失 |
| 不产生 WAL | 不支持流复制 |
| 减少 I/O 压力 | 不适合重要数据 |

**适用场景**:

* 临时数据处理
* ETL 中间结果
* 可重新生成的缓存数据

#### 1.6.3 空闲空间映射(FSM)

**用途**: 跟踪页面内的可用空间

**工作机制**:

* 快速找到可容纳新数据的页面
* 容量动态变化（VACUUM 后变大，INSERT 后变小）
* 以树形结构组织，至少三个数据页

**创建示例**:

```
-- FSM 初始不存在，VACUUM 后创建
VACUUM t;

SELECT size
FROM pg_stat_file('/usr/local/pgsql/data/base/16384/16385_fsm');
-- 结果: 24576 (3页 × 8KB)
```

**FSM 在表和索引中的差异**:

| 对象类型 | FSM 跟踪内容 |
| --- | --- |
| **表** | 每个页面的可用空间量 |
| **索引** | 仅跟踪完全清空可重用的页面 |

**索引的特殊性**: B 树索引必须按排序顺序插入，不能任意选择页面，因此 FSM 只标记完全空的页面。

#### 1.6.4 可见性映射(VM)

**用途**: 优化 VACUUM 和支持仅索引扫描

**两个比特位**:

| 比特位 | 含义 | 作用 |
| --- | --- | --- |
| **第一位** | 页面仅包含最新行版本 | VACUUM 跳过，支持仅索引扫描 |
| **第二位** | 页面所有行版本已冻结 | 避免事务 ID 回卷 |

**示例**:

```
-- 查看 VM 文件
SELECT size
FROM pg_stat_file('/usr/local/pgsql/data/base/16384/16385_vm');
-- 结果: 8192 (单页)
```

**VM 的性能影响**:

```
-- 创建测试表
CREATETABLE test_vm (idint, datatext);
INSERTINTO test_vm SELECT generate_series(1, 100000), 'data';
CREATEINDEXON test_vm(id);

-- VACUUM 设置 VM 位
VACUUM test_vm;

-- 仅索引扫描（不访问表）
EXPLAIN (ANALYZE, BUFFERS) 
SELECTidFROM test_vm WHEREid < 1000;

-- 如果 VM 位已设置，不会读取表页面
```

**仅适用于表**: 索引没有 VM 分支，因为索引不使用 MVCC。

### 1.7 页面(Pages/Blocks)

**定义**: 页面是 PostgreSQL 读写数据的最小单位。

**页面特性**:

* **默认大小**: 8 KB
* **可配置范围**: 最大 32 KB
* **配置时机**: 仅编译时可配置（`./configure --with-blocksize`）
* **一致性要求**: 实例只能处理相同大小的页面

**页面的重要性**:

* 所有内部算法针对页面处理优化
* 影响性能和存储效率
* 决定单行最大大小

**页面处理流程**:

```
磁盘文件
    ↓ 读取
缓冲区缓存 (共享内存)
    ↓ 进程访问和修改
    ↓ 定期刷新
磁盘文件
```

**页面大小的权衡**:

| 页面大小 | 优点 | 缺点 |
| --- | --- | --- |
| **小 (4KB)** | 更细粒度锁定，减少无效 I/O | 单行大小受限，索引层次深 |
| **大 (32KB)** | 单行可更大，减少索引层次 | 更多无效 I/O，锁竞争增加 |
| **默认 (8KB)** | 平衡性能和灵活性 | - |

**查看页面信息**:

```
-- 查看当前页面大小
SHOW block_size;
-- 结果: 8192

-- 查看表占用的页面数
SELECT relpages FROM pg_class WHERE relname = 't';

-- 查看表大小（字节）
SELECT pg_relation_size('t');

-- 查看包含索引和 TOAST 的总大小
SELECT pg_total_relation_size('t');
```

### 1.8 TOAST 机制

#### 1.8.1 TOAST 概述

**TOAST**: The Oversized-Attribute Storage Technique（超长属性存储技术）

**核心问题**: 每行必须适合单个页面（约 8KB），如何存储超长数据？

**解决方案**:

1. **压缩**: 压缩长值以适合页面
2. **切分**: 将长值切成小块存储在单独的 TOAST 表中
3. **混合**: 先压缩再切分

#### 1.8.2 TOAST 策略

**四种存储策略**:

| 策略 | 压缩 | 外部存储 | 适用类型 | 说明 |
| --- | --- | --- | --- | --- |
| **plain** | ❌ | ❌ | integer, smallint | 不使用 TOAST |
| **extended** | ✅ | ✅ | text, json, bytea | 压缩并可外部存储 |
| **external** | ❌ | ✅ | - | 不压缩但可外部存储 |
| **main** | ✅ | ✅ | numeric | 优先压缩，压缩无效才外部存储 |

**查看列的存储策略**:

```
CREATE TABLE t(
    a integer,
    b numeric,
    c text,
    d json
);

SELECT attname, atttypid::regtype,
    CASE attstorage
        WHEN'p'THEN'plain'
        WHEN'e'THEN'external'
        WHEN'm'THEN'main'
        WHEN'x'THEN'extended'
    ENDASstorage
FROM pg_attribute
WHERE attrelid = 't'::regclass AND attnum > 0;
```

**输出**:

```
 attname | atttypid | storage
---------+----------+----------
 a       | integer  | plain
 b       | numeric  | main
 c       | text     | extended
 d       | json     | extended
```

**修改存储策略**:

```
-- 对于 JPEG 等不可压缩数据，使用 external 策略
ALTERTABLE t ALTERCOLUMN d SETSTORAGEexternal;

-- 验证修改
SELECT attname, 
    CASE attstorage
        WHEN'p'THEN'plain'
        WHEN'e'THEN'external'
        WHEN'm'THEN'main'
        WHEN'x'THEN'extended'
    ENDASstorage
FROM pg_attribute
WHERE attrelid = 't'::regclass AND attnum > 0;
```

**输出**:

```
 attname | storage
---------+----------
 a       | plain
 b       | main
 c       | extended
 d       | external
```

#### 1.8.3 TOAST 触发阈值

**触发条件**: 行大小超过页面的 1/4（约 2000 字节）

**TOAST 算法流程**:

```
1. 遍历 extended 和 external 列（从最长开始）
   ├─ extended: 压缩，如果压缩后 > 2000 字节，移到 TOAST 表
   └─ external: 直接移到 TOAST 表（不压缩）
   
2. 如果行仍超过阈值
   └─ 将剩余 extended/external 列逐个移到 TOAST 表

3. 如果行仍超过阈值
   └─ 压缩 main 列（保留在主表）

4. 如果行仍超过阈值
   └─ 将 main 列移到 TOAST 表
```

**自定义阈值**:

```
-- 调整表级 TOAST 阈值
ALTER TABLE t SET (toast_tuple_target = 4000);

-- 查看设置
SELECT reloptions FROM pg_class WHERE relname = 't';
```

#### 1.8.4 TOAST 表结构

**TOAST 表的特点**:

* 位于 `pg_toast` 模式（不在搜索路径中）
* 临时表的 TOAST 在 `pg_toast_temp_N` 模式
* 自动创建，对用户透明

**查看 TOAST 表**:

```
-- 查找表的 TOAST 表
SELECT relnamespace::regnamespace, relname
FROM pg_class
WHERE oid = (
    SELECT reltoastrelid
    FROM pg_class WHERE relname = 't'
);
```

**输出**:

```
 relnamespace |     relname
--------------+-----------------
 pg_toast     | pg_toast_16385
```

**TOAST 表结构**:

```
\d+ pg_toast.pg_toast_16385
```

**输出**:

```
        TOAST table "pg_toast.pg_toast_16385"
   Column   |  Type   | Storage
------------+---------+---------
 chunk_id   | oid     | plain
 chunk_seq  | integer | plain
 chunk_data | bytea   | plain
Owning table: "public.t"
Indexes:
    "pg_toast_16385_index" PRIMARY KEY, btree (chunk_id, chunk_seq)
```

**字段说明**:

* **chunk\_id**: 标识属于哪个原始值
* **chunk\_seq**: 块的序号
* **chunk\_data**: 实际数据块

#### 1.8.5 TOAST 实战示例

**示例 1: 压缩场景**

```
-- 插入重复字符（可压缩）
INSERT INTO t VALUES (1, 2.0, 'foo', '{}');
UPDATE t SET c = repeat('A', 5000);

-- 检查 TOAST 表
SELECT * FROM pg_toast.pg_toast_16385;
-- 结果: 0 rows (数据被压缩后适合主表页面)
```

**示例 2: 外部存储场景**

```
-- 插入随机字符（不可压缩）
UPDATE t SET c = (
    SELECT string_agg(chr(trunc(65+random()*26)::integer), '')
    FROM generate_series(1, 5000)
)
RETURNING left(c, 10) || '...' || right(c, 10);
```

**输出**:

```
        ?column?
-------------------------
 YEYNNDTSZR...JPKYUGMLDX
```

**查看 TOAST 表内容**:

```
SELECT chunk_id,
    chunk_seq,
    length(chunk_data),
    left(encode(chunk_data, 'escape')::text, 10) || '...' ||
    right(encode(chunk_data, 'escape')::text, 10)
FROM pg_toast.pg_toast_16385;
```

**输出**:

```
 chunk_id | chunk_seq | length |        ?column?
----------+-----------+--------+-------------------------
    16390 |         0 |   1996 | YEYNNDTSZR...TXLNDZOXMY
    16390 |         1 |   1996 | EWEACUJGZD...GDBWMUWTJY
    16390 |         2 |   1008 | GSGDYSWTKF...JPKYUGMLDX
```

**块大小说明**: 块大小选择使 TOAST 表每页容纳 4 行（具体值因版本而异）。

#### 1.8.6 TOAST 性能优化

**自动值恢复**:

```
-- PostgreSQL 自动恢复原始值
SELECT c FROM t;
-- 对应用透明，无需特殊处理
```

**部分读取优化**:

```
-- 只读取前面的块
SELECT left(c, 100) FROM t;
-- 即使数据被压缩，也只读取需要的块
```

\*\*避免使用 SELECT \*\*\*:

```
-- ❌ 不好：读取所有列包括 TOAST 列
SELECT * FROM t;

-- ✅ 好：只读取需要的列
SELECT id, a, b FROM t;
```

**TOAST 表文件数量**:

```
单个表的最少文件数（有 TOAST 列时）:
- 主表: 3 个（主分支 + FSM + VM）
- TOAST 表: 3 个（主分支 + FSM + VM）
- TOAST 索引: 2 个（主分支 + FSM）
总计: 8 个文件
```

#### 1.8.7 TOAST 使用建议

**何时使用 TOAST**: ✅ 适用场景：

* 存储文档、日志、JSON 数据
* 数据访问频率低
* 需要数据库事务保护

❌ 不适用场景：

* 频繁访问的大对象
* 不需要事务的文件（如图片、视频）
* 对性能要求极高的场景

**替代方案**:

```
-- 方案 1: 存储文件路径
CREATETABLE documents (
    idint,
    file_path text,  -- 存储文件系统路径
    metadata jsonb
);

-- 方案 2: 使用外部存储（如 S3）
CREATETABLE images (
    idint,
    s3_url text,
    thumbnail bytea  -- 只存缩略图
);
```

**性能考虑**:

* 压缩和解压需要 CPU 资源
* 访问 TOAST 表需要额外 I/O
* 频繁更新长列会产生碎片

---

## 二、进程与内存架构

### 2.1 进程模型

**PostgreSQL 采用多进程模型**，而非多线程模型。

#### 2.1.1 Postmaster 进程

**角色**: 主进程，负责管理所有其他进程

**职责**:

* 启动和管理后台进程
* 监听客户端连接
* 为每个连接创建后端进程
* 监控进程健康状态
* 发生故障时重启进程或整个服务

**进程树示意**:

```
postmaster (主进程)
    ├── startup (恢复进程)
    ├── checkpointer (检查点进程)
    ├── writer (后台写进程)
    ├── wal writer (WAL 写进程)
    ├── autovacuum launcher (自动清理启动器)
    │   └── autovacuum worker (清理工作进程)
    ├── stats collector (统计收集器)
    ├── wal sender (WAL 发送进程，用于复制)
    ├── wal receiver (WAL 接收进程，在备库上)
    └── backend processes (后端进程，每个客户端一个)
        ├── backend 1 (客户端 1)
        ├── backend 2 (客户端 2)
        └── backend N (客户端 N)
```

#### 2.1.2 后台进程详解

| 进程名 | 作用 | 运行模式 |
| --- | --- | --- |
| **startup** | 故障后恢复系统，重放 WAL 日志 | 启动时运行 |
| **autovacuum** | 自动清理过期数据和更新统计信息 | 持续运行 |
| **wal writer** | 将 WAL 缓冲区写入磁盘 | 持续运行 |
| **checkpointer** | 执行检查点，将脏页刷盘 | 持续运行 |
| **writer** | 后台写进程，刷新脏页到磁盘 | 持续运行 |
| **stats collector** | 收集数据库使用统计信息 | 持续运行 |
| **wal sender** | 向备库发送 WAL 日志 | 有复制时运行 |
| **wal receiver** | 在备库接收 WAL 日志 | 仅在备库运行 |

**查看进程**:

```
# 查看 PostgreSQL 相关进程
ps aux | grep postgres

# 示例输出
postgres  1234  postmaster -D /var/lib/pgsql/data
postgres  1235  postgres: checkpointer
postgres  1236  postgres: background writer
postgres  1237  postgres: walwriter
postgres  1238  postgres: autovacuum launcher
postgres  1239  postgres: stats collector
postgres  1240  postgres: logical replication launcher
postgres  1241  postgres: user mydb [local] idle
```

#### 2.1.3 多进程 vs 多线程

**当前模型的缺点**:

| 缺点 | 影响 |
| --- | --- |
| **静态共享内存** | 无法动态调整缓冲区大小 |
| **进程创建开销** | 连接建立较慢 |
| **并行效率** | 并行查询效率不如线程 |
| **会话绑定** | 一个会话必须对应一个进程 |

**为什么不改用多线程?**

挑战：

* 需要彻底重构代码
* 隔离性问题（线程共享地址空间）
* 操作系统兼容性
* 资源管理复杂性
* 需要多年开发工作

**当前态度**: 保守为主，短期内不会改变。

### 2.2 内存架构

#### 2.2.1 共享内存

**定义**: 由 postmaster 分配，所有进程可访问的内存区域。

**主要组成**:

```
共享内存
├── 缓冲区缓存 (Buffer Cache) ←  占大部分
│   └── 数据页缓存
├── WAL 缓冲区 (WAL Buffers)
├── CLOG 缓冲区
├── 锁管理结构
├── 共享哈希表
└── 其他共享数据结构
```

**缓冲区缓存**:

```
-- 查看缓冲区缓存大小
SHOW shared_buffers;
-- 默认: 128MB (通常设置为系统内存的 25%)

-- 设置缓冲区大小（需重启）
ALTER SYSTEM SET shared_buffers = '8GB';
```

**缓冲区缓存的作用**:

* 减少磁盘 I/O
* 提高数据访问速度
* 缓存热数据

**查看缓冲区使用情况**:

```
-- 安装 pg_buffercache 扩展
CREATE EXTENSION pg_buffercache;

-- 查看缓冲区统计
SELECT
    c.relname,
    count(*) AS buffers,
    pg_size_pretty(count(*) * 8192) ASsize
FROM pg_buffercache b
JOIN pg_class c ON b.relfilenode = pg_relation_filenode(c.oid)
WHERE b.reldatabase = (SELECToidFROM pg_database WHERE datname = current_database())
GROUPBY c.relname
ORDERBYcount(*) DESC
LIMIT10;
```

#### 2.2.2 进程本地内存

**每个后端进程的私有内存**:

| 内存区域 | 用途 | 配置参数 |
| --- | --- | --- |
| **work\_mem** | 排序、哈希操作 | work\_mem (默认 4MB) |
| **maintenance\_work\_mem** | VACUUM、CREATE INDEX | maintenance\_work\_mem (默认 64MB) |
| **temp\_buffers** | 临时表缓存 | temp\_buffers (默认 8MB) |
| **连接缓存** | 系统目录缓存、预备语句 | - |

**配置示例**:

```
-- 设置排序内存
SET work_mem = '256MB';

-- 设置维护操作内存
SET maintenance_work_mem = '1GB';

-- 仅对当前会话有效，事务结束后恢复
```

**内存使用示例**:

```
-- 查看查询内存使用
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM large_table ORDER BY column1;

-- 输出包含
-- Sort Method: external merge  Disk: 12345kB
-- 或
-- Sort Method: quicksort  Memory: 25kB
```

#### 2.2.3 双缓存机制

**PostgreSQL 与操作系统都有缓存**:

```
应用程序
    ↓
PostgreSQL 进程
    ↓
共享内存缓冲区 (shared_buffers)
    ↓
操作系统页面缓存 (Page Cache)
    ↓
磁盘
```

**双缓存的影响**:

* ✅ 提高命中率
* ❌ 内存使用重复
* ❌ 管理复杂性

**PostgreSQL 不使用 Direct I/O 的原因**:

* 操作系统缓存提供额外保护
* 简化实现
* 依赖操作系统的 I/O 调度

**内存配置建议**:

```
总内存 = 64GB

建议分配：
- shared_buffers: 16GB (25%)
- 操作系统缓存: 32GB (50%)
- 其他进程: 16GB (25%)
```

### 2.3 预写式日志(WAL)

#### 2.3.1 WAL 的作用

**问题**: 缓冲区中的脏页可能因故障丢失，导致数据不一致。

**解决方案**: Write-Ahead Logging (预写式日志)

**原理**:

```
1. 修改数据前，先写 WAL 日志
2. WAL 日志持久化到磁盘
3. 再修改缓冲区中的数据页
4. 稍后将脏页刷新到数据文件
```

**故障恢复流程**:

```
系统崩溃
    ↓
重启数据库
    ↓
读取 WAL 日志
    ↓
重放 (redo) 已提交事务的操作
    ↓
恢复到一致状态
```

#### 2.3.2 WAL 配置

**WAL 位置**:

```
# WAL 文件位置
ls -lh $PGDATA/pg_wal/

# 示例输出
-rw------- 1 postgres postgres 16M Nov 17 10:30 000000010000000000000001
-rw------- 1 postgres postgres 16M Nov 17 10:45 000000010000000000000002
```

**WAL 相关参数**:

```
-- 查看 WAL 配置
SHOW wal_level;          -- minimal, replica, logical
SHOW max_wal_size;       -- WAL 最大大小
SHOW min_wal_size;       -- WAL 最小大小
SHOW wal_buffers;        -- WAL 缓冲区大小

-- WAL 写入策略
SHOW fsync;              -- 是否强制同步
SHOW synchronous_commit; -- 同步提交级别
SHOW wal_writer_delay;   -- WAL writer 休眠时间
```

**同步提交级别**:

| 级别 | 保证 | 性能 | 风险 |
| --- | --- | --- | --- |
| **off** | 不等待 WAL 写入 | 最快 | 可能丢失最近事务 |
| **local** | 等待本地 WAL 写入 | 快 | 备库可能落后 |
| **remote\_write** | 等待备库接收 WAL | 中 | 备库可能未持久化 |
| **on** | 等待备库持久化 WAL | 慢 | 最安全 |
| **remote\_apply** | 等待备库应用 WAL | 最慢 | 最强一致性 |

---

## 三、客户端-服务端交互

### 3.1 连接建立过程

**连接流程**:

```
1. 客户端发起连接请求
    ↓
2. postmaster 监听到连接
    ↓
3. postmaster 创建新的后端进程 (fork)
    ↓
4. 后端进程与客户端建立连接
    ↓
5. 执行身份验证
    ↓
6. 开始会话
    ↓
7. 客户端断开或连接丢失时，会话结束
```

**连接参数**:

```
# 使用 psql 连接
psql -h localhost \      # 主机
     -p 5432 \          # 端口
     -U myuser \        # 用户
     -d mydb            # 数据库

# 连接字符串
postgresql://myuser:password@localhost:5432/mydb
```

**查看当前连接**:

```
-- 查看所有连接
SELECT pid, usename, datname, client_addr, state, query
FROM pg_stat_activity;
```

**示例输出**:

```
  pid  | usename  | datname | client_addr |  state  | query
-------+----------+---------+-------------+---------+--------
 12345 | postgres | mydb    | 127.0.0.1   | active  | SELECT...
 12346 | app_user | appdb   | 10.0.1.5    | idle    | 
 12347 | readonly | reports | 10.0.1.8    | idle in transaction |
```

### 3.2 连接限制

**配置参数**:

```
-- 查看最大连接数
SHOW max_connections;
-- 默认: 100

-- 查看为超级用户预留的连接
SHOW superuser_reserved_connections;
-- 默认: 3

-- 查看当前连接数
SELECT count(*) FROM pg_stat_activity;
```

**设置连接限制**:

```
-- 修改最大连接数（需重启）
ALTERSYSTEMSET max_connections = 200;

-- 为特定数据库设置连接限制
ALTERDATABASE mydb CONNECTIONLIMIT50;

-- 为特定用户设置连接限制
ALTERUSER app_user CONNECTIONLIMIT10;
```

### 3.3 连接池问题

**多连接的问题**:

| 问题 | 影响 |
| --- | --- |
| **内存消耗** | 每个进程占用内存（系统目录缓存、预备语句等） |
| **进程创建开销** | 短连接频繁创建/销毁进程 |
| **性能下降** | 扫描进程列表开销增加 |

**示例场景**:

```
-- 每个连接可能占用 5-10MB 内存
-- 1000 个连接 = 5-10GB 内存

-- 查看单个进程内存使用
SELECT pid, 
       pg_size_pretty(resident_memory_size) as memory
FROM pg_backend_memory_contexts
WHERE pid = pg_backend_pid();
```

**连接池解决方案**:

#### 3.3.1 外部连接池

**PgBouncer**:

```
# pgbouncer.ini 配置
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = md5
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
```

**池模式对比**:

| 模式 | 说明 | 限制 |
| --- | --- | --- |
| **session** | 客户端断开后才释放连接 | 与直连相同 |
| **transaction** | 事务结束后释放连接 | 不能使用会话级特性 |
| **statement** | 语句执行后释放连接 | 不能使用事务 |

**事务池模式限制**:

```
-- ❌ 不能使用的特性
PREPARE stmt ASSELECT * FROM t WHEREid = $1;
SET work_mem = '256MB';
CREATE TEMP TABLE tmp (idint);
DECLAREcursorFORSELECT * FROM t;

-- ✅ 可以使用
BEGIN;
INSERTINTO t VALUES (1, 'data');
COMMIT;
```

#### 3.3.2 应用层连接池

**示例 (Python)**:

```
from psycopg2 import pool

# 创建连接池
connection_pool = pool.SimpleConnectionPool(
    minconn=5,
    maxconn=20,
    host='localhost',
    database='mydb',
    user='myuser'
)

# 获取连接
conn = connection_pool.getconn()
try:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM t")
    results = cursor.fetchall()
finally:
    # 归还连接到池
    connection_pool.putconn(conn)
```

### 3.4 协议通信

#### 3.4.1 libpq 协议

**标准客户端库**: libpq 是 PostgreSQL 官方的 C 语言客户端库。

**协议特点**:

* 基于 TCP/IP
* 消息驱动
* 支持扩展查询协议
* 支持准备语句

**消息类型示例**:

```
客户端 → 服务器:
- Query (Q): 简单查询
- Parse (P): 解析语句
- Bind (B): 绑定参数
- Execute (E): 执行语句

服务器 → 客户端:
- RowDescription (T): 行描述
- DataRow (D): 数据行
- CommandComplete (C): 命令完成
- ReadyForQuery (Z): 准备接收新查询
```

#### 3.4.2 查询执行流程

**简单查询协议**:

```
客户端: SELECT * FROM t WHERE id = 1;
    ↓
服务器: 
    1. 解析 SQL
    2. 重写查询
    3. 规划执行计划
    4. 执行查询
    5. 返回结果
```

**扩展查询协议**:

```
-- 1. 准备语句
PREPARE get_user AS 
    SELECT * FROM users WHERE id = $1;

-- 2. 绑定参数并执行
EXECUTE get_user(123);

-- 3. 释放语句
DEALLOCATE get_user;
```

**协议优势**:

* 避免重复解析
* 支持参数化查询
* 防止 SQL 注入
* 减少网络传输

**查看预备语句**:

```
-- 查看当前会话的预备语句
SELECT name, statement, parameter_types
FROM pg_prepared_statements;
```

### 3.5 身份验证

**认证方法**:

| 方法 | 说明 | 安全性 | 使用场景 |
| --- | --- | --- | --- |
| **trust** | 无需密码 | ⚠️ 低 | 本地开发 |
| **password** | 明文密码 | ⚠️ 低 | 不推荐 |
| **md5** | MD5 哈希 | ⚠️ 中 | 传统方式 |
| **scram-sha-256** | SHA-256 哈希 | ✅ 高 | 推荐 |
| **peer** | 操作系统用户 | ✅ 高 | 本地连接 |
| **cert** | SSL 证书 | ✅ 高 | 企业环境 |

**配置文件**: `pg_hba.conf`

```
# TYPE  DATABASE    USER        ADDRESS         METHOD

# 本地连接使用 peer
local   all         all                         peer

# 本地 TCP 使用 scram-sha-256
host    all         all         127.0.0.1/32    scram-sha-256

# 内网使用 scram-sha-256
host    all         all         10.0.0.0/8      scram-sha-256

# 拒绝其他所有连接
host    all         all         0.0.0.0/0       reject
```

**创建用户**:

```
-- 创建用户并设置密码
CREATEUSER app_user WITHPASSWORD'secure_password';

-- 授予数据库权限
GRANTCONNECTONDATABASE mydb TO app_user;

-- 授予表权限
GRANTSELECT, INSERT, UPDATEONALLTABLESINSCHEMApublicTO app_user;

-- 设置默认权限
ALTERDEFAULTPRIVILEGESINSCHEMApublic
    GRANTSELECT, INSERT, UPDATEONTABLESTO app_user;
```

---

## 四、性能优化最佳实践

### 4.1 连接管理

**推荐配置**:

```
-- 根据硬件调整最大连接数
-- 公式: max_connections = (CPU 核心数 × 2) + 磁盘数
ALTER SYSTEM SET max_connections = 100;

-- 使用连接池
-- 应用 → PgBouncer → PostgreSQL
-- 1000 客户端 → 50 数据库连接
```

### 4.2 内存配置

**推荐比例**:

```
-- 对于专用数据库服务器
-- 系统总内存 = 64GB

-- shared_buffers: 25% = 16GB
ALTERSYSTEMSET shared_buffers = '16GB';

-- effective_cache_size: 75% = 48GB (提示优化器)
ALTERSYSTEMSET effective_cache_size = '48GB';

-- work_mem: 根据并发调整
-- 公式: (总内存 × 0.25) / max_connections
-- (64GB × 0.25) / 100 = 160MB
ALTERSYSTEMSET work_mem = '160MB';

-- maintenance_work_mem: 5% = 3.2GB
ALTERSYSTEMSET maintenance_work_mem = '3GB';
```

### 4.3 表空间策略

**示例配置**:

```
-- SSD 用于热数据
CREATETABLESPACE hot_data 
    LOCATION '/mnt/ssd/pgdata';

-- HDD 用于归档数据
CREATETABLESPACE archive_data 
    LOCATION '/mnt/hdd/pgarchive';

-- 分离索引和表
CREATETABLESPACE index_space 
    LOCATION '/mnt/nvme/pgindex';

-- 应用
CREATETABLE orders (...) TABLESPACE hot_data;
CREATEINDEX orders_idx ON orders(date) TABLESPACE index_space;
```

### 4.4 TOAST 优化

**优化建议**:

```
-- 1. 对不可压缩数据使用 external
ALTERTABLE images 
    ALTERCOLUMN image_data SETSTORAGEexternal;

-- 2. 避免频繁访问长列
-- ❌ 不好
SELECT * FROM documents;

-- ✅ 好
SELECTid, title, summary FROM documents;

-- 3. 考虑使用外部存储
CREATETABLE files (
    idserial,
    filename text,
    s3_url text,      -- 文件在 S3 上
    thumbnail bytea   -- 只存缩略图
);
```

### 4.5 监控指标

**关键指标**:

```
-- 1. 连接数监控
SELECTcount(*) as connections,
       count(*) FILTER (WHERE state = 'active') as active,
       count(*) FILTER (WHERE state = 'idle') as idle
FROM pg_stat_activity;

-- 2. 缓冲区命中率
SELECT
    sum(blks_hit) * 100.0 / (sum(blks_hit) + sum(blks_read)) as cache_hit_ratio
FROM pg_stat_database;
-- 目标: > 99%

-- 3. 表膨胀监控
SELECT schemaname, tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) assize,
    n_dead_tup,
    n_live_tup,
    round(n_dead_tup * 100.0 / nullif(n_live_tup, 0), 2) as dead_ratio
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDERBY n_dead_tup DESC;

-- 4. WAL 生成速率
SELECT
    pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), '0/0')) as wal_size;
```

---

## 五、总结

### 5.1 核心概念回顾

**数据组织层次**:

```
实例 (Instance)
  └── 集簇 (Cluster)
      ├── 数据库 (Database)
      │   ├── 模式 (Schema)
      │   │   ├── 表 (Table)
      │   │   ├── 索引 (Index)
      │   │   └── 其他对象
      │   └── 系统目录
      └── 表空间 (Tablespace)
```

**物理存储**:

```
PGDATA/
  ├── base/           # pg_default 表空间
  ├── global/         # pg_global 表空间
  ├── pg_wal/         # WAL 日志
  ├── pg_tblspc/      # 表空间符号链接
  └── pg_xact/        # 事务状态 (CLOG)
```

**进程架构**:

```
postmaster (主进程)
  ├── 后台进程 (持续运行)
  └── 后端进程 (每个客户端一个)
```

**内存架构**:

```
共享内存
  ├── 缓冲区缓存 (最大)
  ├── WAL 缓冲区
  └── 其他共享结构

进程本地内存
  ├── work_mem
  ├── maintenance_work_mem
  └── 连接缓存
```

### 5.2 设计哲学

**PostgreSQL 的设计特点**:

1. **学术遗产**: 源自关系理论（关系、元组等术语）
2. **简单优先**: 多进程模型虽有局限但稳定可靠
3. **透明机制**: TOAST 对应用透明
4. **灵活配置**: 表空间、模式提供灵活的组织方式
5. **性能平衡**: 双缓存、延迟写入等折中方案

### 5.3 学习路线

**建议学习顺序**:

1. ✅ 理解数据组织（本章）
2. → 掌握 MVCC 和事务
3. → 学习查询执行和优化
4. → 深入索引原理
5. → 理解 WAL 和恢复机制
6. → 掌握复制和高可用

### 5.4 实践建议

**日常运维**:

```
-- 1. 定期检查连接数
SELECTcount(*) FROM pg_stat_activity;

-- 2. 监控缓冲区命中率
SELECTsum(blks_hit) * 100.0 / (sum(blks_hit) + sum(blks_read)) 
FROM pg_stat_database;

-- 3. 检查表膨胀
SELECT schemaname, tablename, n_dead_tup
FROM pg_stat_user_tables
WHERE n_dead_tup > 10000;

-- 4. 查看长事务
SELECT pid, usename, state, 
       now() - xact_start asduration
FROM pg_stat_activity
WHERE state != 'idle'
ORDERBYdurationDESC;
```

---

## 参考资源

**官方文档**:

* Managing Databases
* System Catalogs
* Database File Layout
* TOAST Storage
* Client/Server Protocol

**扩展工具**:

* PgBouncer - 连接池
* Odyssey - 高级连接池
* pg\_buffercache - 缓冲区分析
* pageinspect - 页面内容检查

**源码参考**:

* `backend/catalog/catalog.c` - OID 分配
* `backend/tcop/postgres.c` - 查询处理
* `backend/storage/` - 存储管理
* `backend/access/heap/heaptoast.c` - TOAST 实现

通过理解 PostgreSQL 的架构和内部机制，您可以更好地设计数据库方案、优化性能，并有效地进行故障诊断和系统调优。