---
title: Doris建表参数全解析：性能调优与场景适配指南
author: 架构师知其所以然
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMjg2MzQ5NA==&mid=2247483787&idx=1&sn=9022c852dfcbd0a59159e2b620a94179&chksm=c35034a8039b1d7208f06201dd2a92b4efcf8472d89fbd93487083639b5e5b55e7c22f098a31&mpshare=1&scene=24&srcid=0806W1YU3fanDU7K7ObejjCl&sharer_shareinfo=5ca4044c4328a9d4bf3bad504a15d585&sharer_shareinfo_first=5ca4044c4328a9d4bf3bad504a15d585#rd
---

> 在用 Doris 2.1 搭建数据系统时，很多人都会遇到一个问题：建表语句到底该怎么写？参数一大堆，从UNIQUE KEY 到分桶策略，再到各种 PROPERTIES 设置，看得人眼花缭乱。不少小伙伴刚开始随便设一设，结果上线后查询慢、存储飙、写入卡，才发现建表阶段出了问题。

## 建表基本结构

```
CREATE TABLE db_name.table_name (  
    column1 data_type,  
    column2 data_type,  
    ...  
)  
ENGINE=OLAP  
UNIQUE KEY (key_column)  
DISTRIBUTED BY HASH(key_column) BUCKETS 10  
PROPERTIES (  
    "replication_num" = "3",  
    "enable_persistent_index" = "true"  
);
```

## 核心参数及适用场景

### 1. `ENGINE`

•

**取值：**`OLAP`

•

**说明：** Doris 统一采用 `OLAP` 引擎，无需更改。

•

**适用场景：** 分析型查询、大规模数据分析。

---

### 2. `KEY` 类型（主键模型）

| KEY 类型 | 功能说明 | 适用场景 |
| --- | --- | --- |
| `UNIQUE KEY` | 唯一键，支持更新操作 | 实时数据更新、CDC、维表场景 |
| `AGGREGATE KEY` | 聚合键，支持聚合函数 | 报表汇总、统计分析 |
| `DUPLICATE KEY` | 全字段保留，重复数据不过滤 | 明细日志、事件数据、原始数据保留场景 |

---

### 3. 分布策略（`DISTRIBUTED BY`）

| 分布方式 | 示例 | 适用场景 |
| --- | --- | --- |
| 哈希分布（HASH） | `DISTRIBUTED BY HASH(user_id) BUCKETS 48` | 大数据量、字段基数高 |
| 随机分布（RANDOM） | `DISTRIBUTED BY RANDOM BUCKETS 10` | 小表、测试表、无热点字段 |

---

### 4. `BUCKETS`（桶数量）

•

**说明：** 决定分区粒度与并发度

•

**建议设置：**

•

小表：3~10

•

中表：10~50

•

大表：50~200（视节点/核数而定）

> **判断依据：**

•

小表： 查询非常频繁但数据量小，一次性能全查出来，比如用户等级表、配置表等。

•

中表： 数据不算大，但也需要一定聚合处理或分桶设计，如日常报表、商品明细等。

•

大表： 通常是日志类、事件类数据，存储周期长、查询复杂、写入频繁，数据量呈爆炸增长。

### 5. 表属性（`PROPERTIES`）

| 属性名 | 功能说明 | 建议场景/备注 |
| --- | --- | --- |
| `"replication_num"` | 副本数量，提升容灾能力 | 小集群可调低为 1 或 2；大集群建议保持 3 |
| `"enable_persistent_index"` | 启用持久化索引，降低内存压力 | 大表、主键高基数表推荐开启 |
| `"storage_format"` | 存储格式，默认 `V2` | 推荐使用 V2，功能更丰富 |
| `"compression"` | 数据压缩，支持 `lz4`, `zstd`, `snappy` | 大表推荐 `zstd`，兼顾压缩率与性能 |
| `"enable_unique_key_merge_on_write"` | 唯一键表写入时自动合并（MoW） | 实时数据写入、去重更新类场景 |
| `"in_memory"` | 是否常驻内存 | 高频访问小表可设置为 `true` |

### 6. 分区策略（可选）

```
PARTITION BY RANGE (dt) (  
    PARTITION p202508 VALUES LESS THAN ("2025-09-01"),  
    PARTITION pMAX VALUES LESS THAN MAXVALUE  
)
```

•

**适用场景：** 时序类、分区清理、冷热数据管理

•

**优势：** 提升查询效率，便于数据管理

---

## 场景配置推荐表

| 应用场景 | KEY 类型 | 是否分区 | 分桶字段 | 其他建议 |
| --- | --- | --- | --- | --- |
| 用户行为日志 | DUPLICATE KEY | 是 | user\_id | 压缩用 zstd，BUCKETS ≥ 48 |
| 实时更新维度表 | UNIQUE KEY | 否/是 | 主键字段 | MoW 模式开启，persistent\_index 开启 |
| 聚合报表数据 | AGGREGATE KEY | 是 | 地区、产品ID等 | 设置合适聚合函数，zstd 压缩 |
| 小型配置维度表 | UNIQUE KEY | 否 | 随机或主键 | BUCKETS 设置为 3~10，in\_memory 可设置为 true |

## 案例

```
-- DROP TABLE IF EXISTS dwd.dwd_trade_order_detail_inc;  
CREATE TABLE dwd.dwd_trade_order_detail_inc  
(  
    `id`                    VARCHAR(255) COMMENT '编号',  
    `k1`                    DATE NOT NULL COMMENT '分区字段',  
    `order_id`              STRING COMMENT '订单编号',  
    `user_id`               STRING COMMENT '用户ID',  
    `sku_id`                STRING COMMENT '商品ID',  
    `sku_name`              STRING COMMENT '商品名称',  
    `province_id`           STRING COMMENT '省份ID',  
    `activity_id`           STRING COMMENT '活动ID',  
    `activity_rule_id`      STRING COMMENT '活动规则ID',  
    `coupon_id`             STRING COMMENT '优惠券ID',  
    `date_id`               STRING COMMENT '日期ID',  
    `create_time`           STRING COMMENT '创建时间',  
    `source_id`             STRING COMMENT '来源编号',  
    `source_type`           STRING COMMENT '来源类型',  
    `source_type_name`      STRING COMMENT '来源类型名称',  
    `sku_num`               BIGINT COMMENT '商品数量',  
    `split_original_amount` DECIMAL(16, 2) COMMENT '原始价格',  
    `split_activity_amount` DECIMAL(16, 2) COMMENT '活动优惠分摊',  
    `split_coupon_amount`   DECIMAL(16, 2) COMMENT '优惠券优惠分摊',  
    `split_total_amount`    DECIMAL(16, 2) COMMENT '最终价格分摊'  
)  
    ENGINE=OLAP  -- 使用Doris的OLAP引擎，适用于高并发分析场景  
    UNIQUE KEY(`id`,`k1`)  -- 唯一键约束，保证(id, k1)组合的唯一性（Doris聚合模型特性）  
COMMENT '交易域订单明细事实表'  
PARTITION BY RANGE(`k1`) ()  -- 按日期范围分区（具体分区规则由动态分区配置决定）  
DISTRIBUTED BY HASH(`id`)  -- 按id哈希分桶，保证相同id的数据分布在同一节点  
PROPERTIES  
(  
    "replication_allocation" = "tag.location.default: 1",  -- 副本分配策略：默认标签分配1个副本  
    "is_being_synced" = "false",          -- 是否处于同步状态（通常保持false）  
    "storage_format" = "V2",             -- 存储格式版本（V2支持更高效压缩和索引）  
    "light_schema_change" = "true",      -- 启用轻量级schema变更（仅修改元数据，无需数据重写）  
    "disable_auto_compaction" = "false", -- 启用自动压缩（合并小文件提升查询性能）  
    "enable_single_replica_compaction" = "false", -- 禁用单副本压缩（多副本时保持数据一致性）  
  
    "dynamic_partition.enable" = "true",            -- 启用动态分区  
    "dynamic_partition.time_unit" = "DAY",          -- 按天创建分区  
    "dynamic_partition.start" = "-60",             -- 保留最近60天的历史分区  
    "dynamic_partition.end" = "3",                 -- 预先创建未来3天的分区  
    "dynamic_partition.prefix" = "p",              -- 分区名前缀（如p20240101）  
    "dynamic_partition.buckets" = "32",            -- 每个分区的分桶数（影响并行度）  
    "dynamic_partition.create_history_partition" = "true", -- 自动创建缺失的历史分区  
  
    "bloom_filter_columns" = "id,order_id,user_id,sku_id,activity_id,coupon_id",  -- 为高频过滤字段创建布隆过滤器，加速WHERE查询  
    "compaction_policy" = "time_series",          -- 按时间序合并策略优化时序数据  
    "enable_unique_key_merge_on_write" = "true",  -- 唯一键写时合并（实时更新场景减少读放大）  
    "in_memory" = "false"                        -- 关闭全内存存储（仅小表可开启）  
);
```

该表为典型的大表场景，具备以下特征：

•

明细数据量大（亿级）

•

实时写入和更新（UNIQUE KEY + MoW）

•

按天分区管理（动态分区）

•

多字段查询优化（布隆过滤器）

•

存储节省和压缩优化（V2 + time\_series）  
非常适合用于实时订单处理、日志数据存储、行为数据分析等中高频查询场景。

## 总结

•

大表建议开启持久化索引、压缩、合理分桶和分区。

•

唯一键表多用于更新类数据，结合 MoW 提高效率。

•

日志类建议使用 DUPLICATE KEY，保留原始明细。

•

表设计前尽量明确查询模式和更新模式，做到精细建模。