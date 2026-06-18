---
title: Paimon系列：主键表之合并引擎merge-engine
author: 种桃者说
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU2NzA3OTEwMg==&mid=2247484272&idx=1&sn=db7443da3dbed29229824cad3c733c88&chksm=fdab2edaaa80aadf79e18c482b11f7cede5431788a858ad356dffde38294fa1c3a381203164f&mpshare=1&scene=24&srcid=0829sKgmUx1QjJordMUzTAlJ&sharer_shareinfo=3dc8b0bd9246780a4a9b30d4fc484c8a&sharer_shareinfo_first=3dc8b0bd9246780a4a9b30d4fc484c8a#rd
---

### 前言

在《[Paimon系列：IDEA环境读写Paimon表](https://mp.weixin.qq.com/s?__biz=MzU2NzA3OTEwMg==&mid=2247484241&idx=1&sn=f148131baf686fefac3243bde5a6b600&scene=21#wechat_redirect)》一文中，详细介绍了如何创建和使用追加表与主键表，其中主键表是核心表类型，通过主键保证数据唯一性，支持高效的插入、更新和删除操作。主键表通过 Merge Engine（合并引擎） 来处理具有相同主键的多条记录，合并成一条记录以保持主键唯一性，本文将主键表 Merge Engine 的四种合并引擎：deduplicate, first-row, aggregation, partial-update。

本文依旧使用 Flink-1.16 + Paimon-1.0.1 版本，通过 Flink SQL 进行读写演示 (需要把 paimon-flink-1.16-1.2.0.jar 放到 Flink lib目录下)。

### 主键表和 merge-engine

当 Paimon 的 Sink 接收到多条具有相同主键的记录时，它会根据配置的 merge-engine 属性将这些记录合并为一条，以确保主键的唯一性。合并引擎定义了如何处理这些记录，适用于不同的业务场景。Paimon 提供了以下几种主要的合并引擎：

---

一. deduplicate (去重，默认)

**功能**：保留同一主键下最新的记录，丢弃其他具有相同主键的记录; 如果最新的记录是 DELETE 记录，则所有具有相同主键的记录都会被删除。

**适用场景**：需要去重并只保留最新数据的场景，例如日志去重或覆盖式更新。

**写入：**

创建一张商品表 products\_dedup

```
SET 'execution.runtime-mode' = 'streaming';SET 'table.exec.sink.upsert-materialize'='NONE';SET 'execution.checkpointing.interval'='60 s';  
//创建本地文件 catalogCREATE CATALOG paimon WITH (           'type' = 'paimon',           'warehouse' = 'file:///tmp/paimon');USE CATALOG paimon;  
create database if not exists merge_engine_db;USE merge_engine_db;  
//创建 去重表CREATE TABLE products_dedup (       product_id STRING,           product_name STRING,           price DECIMAL(10, 2),           PRIMARY KEY (product_id) NOT ENFORCED) WITH (          'merge-engine' = 'deduplicate');  
INSERT INTO products_dedup (product_id, product_name, price) VALUES ('P001', 'iPhone', 1000.00);INSERT INTO products_dedup (product_id, product_name, price) VALUES ('P001', 'Mac', 20000.00);
```

**读取**

另起一个客户端进行批读，可以看到同一主键插入的两条数据，只会保留最新的一条数据：

---

二. first\_row

**功能**：保留同一主键下的第一条记录，忽略后续记录。

**适用场景**：流式计算中需要保留首次记录的场景，如去重日志处理。

**写入**

```
CREATE TABLE products_first (         product_id STRING,         product_name STRING,         price DECIMAL(10, 2),         PRIMARY KEY (product_id) NOT ENFORCED) WITH (        'merge-engine' = 'first-row');  
INSERT INTO products_first (product_id, product_name, price) VALUES ('P001', 'iPhone', 1000.00);INSERT INTO products_first (product_id, product_name, price) VALUES ('P001', 'Mac', 20000.00);
```

**读取**

同样是进行批读，可以看到同一主键插入的两条数据，只会保留最早的一条数据：

---

三. aggregation

**功能**：根据指定的聚合函数（如 sum, max, last\_value 等）对每个非主键字段进行聚合。

**适用场景**：需要对数据进行聚合的场景，如统计销售额、最大价格等。

```
CREATE TABLE sales_agg (          city_id STRING,          city_name STRING,          sales DECIMAL(10, 2),          sales_sum DECIMAL(10, 2),          sales_max DECIMAL(10, 2),          PRIMARY KEY (city_id) NOT ENFORCED) WITH (         'merge-engine' = 'aggregation',         'fields.sales_sum.aggregate-function' = 'sum',         'fields.sales_max.aggregate-function' = 'max');     INSERT INTO sales_agg (city_id, city_name, sales, sales_sum, sales_max) VALUES ('P001', 'beijing', 1000.00, 1000.00, 1000.00);INSERT INTO sales_agg (city_id, city_name, sales, sales_sum, sales_max) VALUES ('P001', 'beijing', 25000.00, 25000.00, 25000.00);
```

**读取**

同样是进行批读，插入的两条数据会按主键求 sum, max 做聚合

---

四. partial-update

**功能**：允许通过多次更新逐步完善记录的列，逐一更新值字段，使用同一主键下的最新非空数据，空值（NULL）不会覆盖已有值。

**适用场景**：多流数据打宽（如信息流曝光/转发/评论/点赞 多流 Join）、实时数据合并。

**写入**：

```
CREATE TABLE article_partial (           article_id STRING,           is_view BOOLEAN,           is_transmit BOOLEAN,           is_cmt BOOLEAN,           is_like BOOLEAN,           PRIMARY KEY (article_id) NOT ENFORCED) WITH (          'merge-engine' = 'partial-update');  
INSERT INTO article_partial (article_id, is_view, is_transmit, is_cmt, is_like) VALUES ('A001', true, CAST(NULL AS BOOLEAN), CAST(NULL AS BOOLEAN), CAST(NULL AS BOOLEAN));INSERT INTO article_partial (article_id, is_view, is_transmit, is_cmt, is_like) VALUES ('A001', CAST(NULL AS BOOLEAN), true, CAST(NULL AS BOOLEAN), CAST(NULL AS BOOLEAN));INSERT INTO article_partial (article_id, is_view, is_transmit, is_cmt, is_like) VALUES ('A001', CAST(NULL AS BOOLEAN), CAST(NULL AS BOOLEAN), true, CAST(NULL AS BOOLEAN));INSERT INTO article_partial (article_id, is_view, is_transmit, is_cmt, is_like) VALUES ('A001', CAST(NULL AS BOOLEAN), CAST(NULL AS BOOLEAN), CAST(NULL AS BOOLEAN), true);
```

**读取**

使用 Paimon 的 partial-update 合并引擎可以有效替代实时 JOIN 操作，从而降低状态开销无需维护复杂的JOIN状态。相比实时 JOIN 的高内存（状态数据）和计算开销，partial-update 通过主键表直接合并（union）多流数据（如曝光、转发、评论等），简化了数据处理流程，减少了状态存储需求。如果你看到这里意味着什么呢？意味着这篇文章需要"转发"，"评论"，"点赞"走一波。

---

以上例子都是指定"批模式"下进行查询，如果把查询模式切换成流读呢？如下图所示，报错提示需要设置对应的changelog producer，有关changelog-producer的详细配置和流读实现，将在后续文章中深入探讨。