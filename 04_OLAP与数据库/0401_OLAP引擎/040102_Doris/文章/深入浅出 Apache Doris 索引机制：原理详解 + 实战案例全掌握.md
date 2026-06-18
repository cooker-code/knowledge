---
title: 深入浅出 Apache Doris 索引机制：原理详解 + 实战案例全掌握
author: 架构师知其所以然
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMjg2MzQ5NA==&mid=2247483766&idx=1&sn=e97c93f0d849d1cae2b7520e4813a327&chksm=c3ac0eebf6c68d9d96112ca9b22690e08f06f634baec90b82d10d34b85a8c50f55b01be2ee91&mpshare=1&scene=24&srcid=0806s8bCYVPngl29jMn4uj5F&sharer_shareinfo=b6dcbd39e79e40c128512e6250667f6a&sharer_shareinfo_first=b6dcbd39e79e40c128512e6250667f6a#rd
---

> 索引是加速查询的关键手段，Doris 作为一款高性能 MPP 数据库，支持多种索引机制，合理的索引设计能显著提升查询效率，尤其在大规模数据分析场景下效果突出。

## 索引总览

Doris 的索引体系分为两类：

•

**点查索引（Point Lookup Indexes）**：直接定位满足 WHERE 条件的行，适用于高选择性查询。

•

**跳数索引（Skip Block Indexes）**：快速跳过无关数据块，适用于分析型查询。

| 类型 | 索引名称 | 使用方式示例 | 查询支持类型 | 是否自动创建 |
| --- | --- | --- | --- | --- |
| 点查索引 | 前缀索引（Prefix） | 自动使用 | =, IN, 范围 | 是 |
| 点查索引 | 倒排索引（Inverted） | CREATE INDEX | MATCH, LIKE, = | 否 |
| 跳数索引 | ZoneMap | 自动使用 | =, IN, IS NULL | 是 |
| 跳数索引 | BloomFilter | CREATE INDEX | =, IN | 否 |
| 跳数索引 | NGram BloomFilter | CREATE INDEX | LIKE | 否 |

## 各类索引详解

### 1. 前缀索引（Prefix Index）

•

**原理**：对排序键列的前缀进行稀疏索引，每 1024 行采样一次前缀。

•

**使用方式**：系统自动创建，无需用户干预。

•

**适用场景**：排序键前置、频繁用于等值和范围查询。

### 2. 倒排索引（Inverted Index）

•

**原理**：为字段内容构建关键词 → 行号的倒排表。

•

**使用方式**：

```
CREATE INDEX idx_content ON my_table(content) USING INVERTED;
```

•

**适用场景**：全文检索、LIKE 查询、复杂组合条件。

### 3. ZoneMap 索引

•

**原理**：每个数据块记录列的最小值、最大值和 NULL 状态。

•

**使用方式**：自动创建。

•

**适用场景**：范围过滤、等值过滤、大表扫描优化。

### 4. BloomFilter 索引

•

**原理**：基于哈希构造的位图过滤器，判断值是否可能存在。

•

**使用方式**：

```
CREATE INDEX idx_userid ON my_table(user_id) USING BLOOMFILTER;
```

•

**适用场景**：高基数字段的精确匹配，如 user\_id、订单号等。

### 5. NGram BloomFilter 索引

•

**原理**：对字符串进行 n-gram 分片并建立 BloomFilter。

•

**使用方式**：

```
CREATE INDEX idx_name ON my_table(name) USING NGRAM_BF PROPERTIES("gram_size"="3");
```

•

**适用场景**：LIKE 查询，如 `%abc%` 模糊匹配。

---

## 索引使用建议

1

高频查询字段应尽可能放入排序键以命中前缀索引。

2

需要全文或模糊匹配时使用倒排索引。

3

等值匹配、过滤率高的字段建议启用 BloomFilter。

4

LIKE 查询多的文本字段可启用 NGram BloomFilter。

5

通过 `EXPLAIN` 和 `PROFILE` 命令确认索引是否生效。

## 实际案例 —— 日志搜索系统

### 表设计：日志表 user\_logs

```
CREATE TABLE user_logs (  
    log_id BIGINT,  
    user_id BIGINT,  
    action STRING,  
    log_text STRING,  
    log_time DATETIME  
)  
ENGINE=OLAP  
DUPLICATE KEY(log_id)  
PARTITION BY RANGE(log_time) (  
    PARTITION p202507 VALUES LESS THAN ("2025-08-01"),  
    PARTITION pmax VALUES LESS THAN ("9999-12-31")  
)  
DISTRIBUTED BY HASH(log_id) BUCKETS 8  
PROPERTIES (  
    "replication_num" = "1"  
);
```

### 索引创建

```
-- 倒排索引：全文检索  
CREATE INDEX idx_logtext ON user_logs(log_text) USING INVERTED;  
  
-- BloomFilter：加速 user_id 精确匹配  
CREATE INDEX idx_userid ON user_logs(user_id) USING BLOOMFILTER;  
  
-- NGram BloomFilter：优化 LIKE 查询  
CREATE INDEX idx_logtext_ngram ON user_logs(log_text) USING NGRAM_BF PROPERTIES("gram_size"="3");
```

### 示例数据

```
INSERT INTO user_logs VALUES  
(1001, 20001, 'click', 'user accessed the dashboard page', '2025-07-25 10:00:00'),  
(1002, 20002, 'login', 'login successful from mobile device', '2025-07-25 10:01:00'),  
(1003, 20001, 'logout', 'user logged out normally', '2025-07-25 10:05:00');
```

### 查询示例与索引触发说明

#### 查询 1：user\_id 精确匹配

```
SELECT * FROM user_logs WHERE user_id = 20001;  
-- 命中：BloomFilter、ZoneMap
```

#### 查询 2：全文搜索

```
SELECT * FROM user_logs WHERE MATCH(log_text, 'dashboard');  
-- 命中：倒排索引
```

#### 查询 3：LIKE 模糊查询

```
SELECT * FROM user_logs WHERE log_text LIKE '%mobile%';  
-- 命中：NGram BloomFilter
```

#### 查询 4：时间范围过滤

```
SELECT * FROM user_logs WHERE log_time BETWEEN '2025-07-25 09:00:00' AND '2025-07-25 11:00:00';  
-- 命中：前缀索引、ZoneMap
```

### 场景推荐总结

| 查询场景 | 推荐索引类型 |
| --- | --- |
| 精确匹配高基数字段 | BloomFilter |
| 文本关键词搜索 | 倒排索引（Inverted） |
| 模糊文本匹配 | NGram BloomFilter |
| 范围扫描时间字段 | 前缀索引 + ZoneMap |

## 参考文档

•

官方文档：https://doris.apache.org/zh-CN/docs/table-design/index/index-overview