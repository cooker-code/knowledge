---
title: Hive 性能调优与优化技巧
author: DATA炼狱
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUyMDkzNjAzMQ==&mid=2247484186&idx=1&sn=0f19e6e98e4c9461e3efe6a9f23402dd&chksm=f82eddc9e39ad28c7d838197fb0da27e1330a89417d292ab644e0506dea40724a0994ba1e259&mpshare=1&scene=24&srcid=04127TmwLKtGjHfnn8Xkg4YJ&sharer_shareinfo=7a20e7c19ae5e1d6505c931998cacc6e&sharer_shareinfo_first=7a20e7c19ae5e1d6505c931998cacc6e#rd
---

# Hive 性能调优与优化技巧

> 摘要：从执行计划分析到参数调优，全面掌握 Hive 性能优化方法，让查询提速 10 倍。

## 性能分析基础

### 查看执行计划

```
-- 基础执行计划
EXPLAIN SELECT * FROM orders WHERE dt = '2024-01-01';

-- 详细执行计划
EXPLAIN EXTENDED SELECT COUNT(*) FROM orders;

-- 格式化输出（更易读）
EXPLAIN FORMATTED 
SELECT user_id, SUM(order_amount) 
FROM orders 
GROUP BY user_id;

-- CBO 成本估算
EXPLAIN CBO SELECT * FROM orders o JOIN users u ON o.user_id = u.user_id;
```

### 执行计划解读

```
STAGE DEPENDENCIES:
  Stage-1 is a root stage
  Stage-0 depends on stages: Stage-1

STAGE PLANS:
  Stage: Stage-1
    Map Reduce
      Map Operator Tree:
          TableScan           -- 表扫描
            alias: orders
            Statistics: Num rows: 1000000 Data size: 190000000
            Filter Operator   -- 过滤操作
              predicate: (dt = '2024-01-01')
              Statistics: Num rows: 10000 Data size: 1900000
              Select Operator -- 列选择
                Statistics: Num rows: 10000
                Group By Operator  -- 分组聚合
                  aggregations: sum(order_amount)
                  keys: user_id
                  Reduce Output Operator
      Reduce Operator Tree:
        Group By Operator
          aggregations: sum(VALUE._col0)
          keys: KEY._col0
          File Output Operator
```

## 数据倾斜优化

### 识别数据倾斜

```
-- 检查 Key 分布
SELECT 
    user_id, 
    COUNT(*) as cnt
FROM orders
GROUP BY user_id
ORDER BY cnt DESC
LIMIT 20;

-- 如果 top 几条数据量远大于其他，说明有倾斜
```

### 倾斜解决方案

**方案 1：随机前缀打散**

```
-- 原始倾斜 JOIN
SELECT * FROM skewed_table a JOIN normal_table b ON a.key = b.key;

-- 优化：给倾斜 key 加随机前缀
SELECT 
    a.*, b.*
FROM (
    SELECT 
        CONCAT(CAST(RAND() * 10 AS INT), '_', key) as new_key,
        key as original_key,
        *
    FROM skewed_table
) a
JOIN (
    SELECT 
        CONCAT(i, '_', key) as new_key,
        *
    FROM normal_table
    LATERAL VIEW EXPLODE(ARRAY(0,1,2,3,4,5,6,7,8,9)) t AS i
) b ON a.new_key = b.new_key;
```

**方案 2：Skew Join 优化（Hive 自动处理）**

```
-- 开启 Skew Join 优化
SET hive.optimize.skewjoin = true;
SET hive.skewjoin.key = 100000;  -- 超过此阈值的 key 视为倾斜

-- 或者使用 hint
SELECT /*+ SKEWJOIN(a) */ *
FROM skewed_table a JOIN normal_table b ON a.key = b.key;
```

**方案 3：两阶段聚合**

```
-- 第一阶段：局部聚合（加随机前缀）
INSERT INTO TABLE tmp_table
SELECT 
    CONCAT(CAST(RAND() * 10 AS INT), '_', user_id) as prefix_user_id,
    SUM(order_amount) as partial_sum
FROM orders
GROUP BY CONCAT(CAST(RAND() * 10 AS INT), '_', user_id);

-- 第二阶段：全局聚合
SELECT 
    SUBSTR(prefix_user_id, 3) as user_id,
    SUM(partial_sum) as total_amount
FROM tmp_table
GROUP BY SUBSTR(prefix_user_id, 3);
```

## JOIN 优化

### MapJoin（小表广播）

```
-- 自动转换 MapJoin
SET hive.auto.convert.join = true;
SET hive.auto.convert.join.noconditionaltask = true;
SET hive.auto.convert.join.noconditionaltask.size = 10000000;  -- 10MB

-- 手动指定 MapJoin
SELECT /*+ MAPJOIN(small_table) */ *
FROM small_table
JOIN big_table ON small_table.id = big_table.id;

-- 多个小表
SELECT /*+ MAPJOIN(a, b) */ *
FROM big_table c
JOIN small_table a ON c.id = a.id
JOIN small_table b ON c.id = b.id;
```

### Bucket MapJoin

```
-- 两个表都按 join key 分桶，且桶数成倍数关系
CREATE TABLE orders_bucketed (
    order_id STRING,
    user_id STRING,
    amount DOUBLE
)
CLUSTERED BY (user_id) INTO 256 BUCKETS;

CREATE TABLE users_bucketed (
    user_id STRING,
    name STRING
)
CLUSTERED BY (user_id) INTO 256 BUCKETS;

-- 开启 Bucket MapJoin
SET hive.optimize.bucketmapjoin = true;

-- 相同桶的数据在同一文件，可直接 MapJoin
SELECT * FROM orders_bucketed o JOIN users_bucketed u ON o.user_id = u.user_id;
```

### SMB Join（Sort-Merge-Bucket）

```
-- 两个表都按相同 key 分桶且排序
CREATE TABLE orders_smb (
    order_id STRING,
    user_id STRING,
    amount DOUBLE
)
CLUSTERED BY (user_id) SORTED BY (user_id) INTO 256 BUCKETS;

-- 开启 SMB Join
SET hive.optimize.bucketmapjoin = true;
SET hive.optimize.bucketmapjoin.sortedmerge = true;

-- 最高效的 JOIN 方式，无需 Shuffle
SELECT * FROM orders_smb o JOIN users_smb u ON o.user_id = u.user_id;
```

## 分区裁剪与列裁剪

### 分区裁剪

```
-- 确保 WHERE 条件能触发分区裁剪
-- ✅ 好的写法：直接比较分区列
SELECT * FROM orders WHERE dt = '2024-01-01';
SELECT * FROM orders WHERE dt BETWEEN '2024-01-01' AND '2024-01-07';

-- ❌ 差的写法：函数作用于分区列
SELECT * FROM orders WHERE SUBSTR(dt, 1, 7) = '2024-01';
SELECT * FROM orders WHERE YEAR(dt) = 2024;  -- 如果 dt 是 STRING

-- ✅ 优化后
SELECT * FROM orders WHERE dt >= '2024-01-01' AND dt <= '2024-01-31';
```

### 列裁剪

```
-- ✅ 只选择需要的列
SELECT user_id, order_amount FROM orders;

-- ❌ 避免 SELECT *
SELECT * FROM orders;

-- 开启列裁剪（默认开启）
SET hive.optimize.cp = true;
```

### 谓词下推

```
-- 开启谓词下推
SET hive.optimize.ppd = true;
SET hive.optimize.ppd.storage = true;

-- 这样写，过滤条件会推到最底层
SELECT * FROM (
    SELECT * FROM orders WHERE dt = '2024-01-01'
) t
WHERE order_amount > 100;
```

## 执行引擎优化

### Tez 引擎调优

```
-- 启用 Tez
SET hive.execution.engine = tez;

-- Tez 内存配置
SET tez.am.resource.memory.mb = 2048;
SET tez.task.resource.memory.mb = 1024;

-- 容器复用
SET tez.session.am.dag.submit.timeout.secs = 300;
SET tez.queue.name = default;

-- 向量化执行
SET hive.vectorized.execution.enabled = true;
SET hive.vectorized.execution.reduce.enabled = true;
```

### Spark 引擎调优

```
-- 启用 Spark
SET hive.execution.engine = spark;

-- Spark 资源配置
SET spark.executor.instances = 10;
SET spark.executor.cores = 4;
SET spark.executor.memory = 8g;
SET spark.executor.memoryOverhead = 2g;
SET spark.driver.memory = 4g;

-- 动态资源分配
SET spark.dynamicAllocation.enabled = true;
SET spark.dynamicAllocation.minExecutors = 5;
SET spark.dynamicAllocation.maxExecutors = 50;

-- 广播阈值（MapJoin 自动转换）
SET spark.sql.autoBroadcastJoinThreshold = 104857600;  -- 100MB
```

## 小文件问题

### 小文件的危害

* NameNode 内存压力
* Map 任务启动开销大
* 影响查询性能

### 小文件解决方案

**方案 1：合并输出**

```
-- 设置输出文件大小
SET hive.merge.mapfiles = true;
SET hive.merge.mapredfiles = true;
SET hive.merge.size.per.task = 256000000;  -- 256MB
SET hive.merge.smallfiles.avgsize = 16000000;  -- 16MB
```

**方案 2：插入时合并**

```
-- 使用 DISTRIBUTE BY 控制 reducer 数量
INSERT OVERWRITE TABLE target_table
SELECT /*+ COALESCE(10) */ * FROM source_table;

-- 或者
INSERT OVERWRITE TABLE target_table
SELECT * FROM source_table
DISTRIBUTE BY CAST(RAND() * 10 AS INT);
```

**方案 3：归档处理**

```
# Hadoop 归档
hadoop archive -archiveName logs.har -p /user/hive/warehouse/logs /user/hive/warehouse/

# 或者压缩
SET hive.exec.compress.output = true;
SET mapred.output.compression.codec = org.apache.hadoop.io.compress.SnappyCodec;
```

## 存储与压缩优化

### 存储格式选择

| 格式 | 压缩比 | 查询性能 | 使用场景 |
| --- | --- | --- | --- |
| TEXTFILE | 1x | 慢 | 调试、临时数据 |
| SEQUENCEFILE | 2-3x | 中等 | MapReduce 兼容 |
| ORC | 4-5x | 最快 | Hive 首选 |
| Parquet | 4-5x | 快 | 多引擎共享 |

### ORC 优化配置

```
CREATE TABLE optimized_table (
    id INT,
    name STRING,
    amount DOUBLE
)
PARTITIONED BY (dt STRING)
STORED AS ORC
TBLPROPERTIES (
    'orc.compress'='ZLIB',              -- 压缩算法: NONE, ZLIB, SNAPPY
    'orc.compress.size'='262144',        -- 压缩块大小 256KB
    'orc.stripe.size'='268435456',       -- Stripe 大小 256MB
    'orc.row.index.stride'='10000',      -- 索引步长
    'orc.create.index'='true',           -- 创建索引
    'orc.bloom.filter.columns'='id,name', -- Bloom Filter 列
    'orc.bloom.filter.fpp'='0.05'        -- 假阳性率
);
```

### 压缩配置

```
-- Map 输出压缩
SET hive.exec.compress.intermediate = true;
SET mapred.map.output.compression.codec = org.apache.hadoop.io.compress.SnappyCodec;

-- 最终输出压缩
SET hive.exec.compress.output = true;
SET mapred.output.compression.codec = org.apache.hadoop.io.compress.SnappyCodec;

-- 任务间压缩
SET hive.exec.compress.intermediate = true;
SET hive.intermediate.compression.codec = org.apache.hadoop.io.compress.SnappyCodec;
```

## CBO（基于成本的优化器）

### 开启 CBO

```
-- 启用 CBO
SET hive.cbo.enable = true;
SET hive.compute.query.using.stats = true;
SET hive.stats.fetch.column.stats = true;
SET hive.stats.fetch.partition.stats = true;

-- 收集表统计信息
ANALYZE TABLE orders COMPUTE STATISTICS;

-- 收集列统计信息
ANALYZE TABLE orders COMPUTE STATISTICS FOR COLUMNS 
    order_id, user_id, order_amount, dt;

-- 查看统计信息
DESCRIBE FORMATTED orders;
```

### 统计信息收集策略

```
-- 新表插入数据后自动收集
SET hive.stats.autogather = true;

-- 手动收集分区统计
ANALYZE TABLE orders PARTITION (dt='2024-01-01') COMPUTE STATISTICS;

-- 增量收集（只收集新增分区）
ANALYZE TABLE orders PARTITION (dt) COMPUTE STATISTICS;
```

## 向量化查询

### 开启向量化执行

```
-- 启用向量化
SET hive.vectorized.execution.enabled = true;
SET hive.vectorized.execution.reduce.enabled = true;

-- 检查是否生效
EXPLAIN VECTORIZATION SELECT SUM(order_amount) FROM orders;

-- 输出包含 "Vectorization: enabled" 表示生效
```

### 向量化适用条件

* 存储格式：ORC、Parquet
* 数据类型：基本类型（INT、BIGINT、DOUBLE、STRING 等）
* 表达式：简单表达式、部分聚合函数

## 并行执行优化

### 并行度设置

```
-- Map 任务数（根据输入数据量自动计算）
-- 或者手动设置切片大小
SET mapred.max.split.size = 256000000;      -- 256MB
SET mapred.min.split.size = 10000000;       -- 10MB

-- Reduce 任务数
SET mapred.reduce.tasks = 100;              -- 手动设置
-- 或者
SET hive.exec.reducers.bytes.per.reducer = 256000000;  -- 每个 reducer 256MB
SET hive.exec.reducers.max = 1000;

-- 并行执行（无依赖的 stage 并行跑）
SET hive.exec.parallel = true;
SET hive.exec.parallel.thread.number = 8;
```

### 推测执行

```
-- 开启推测执行（慢任务启动备份任务）
SET mapred.map.tasks.speculative.execution = true;
SET mapred.reduce.tasks.speculative.execution = true;
SET hive.mapred.reduce.tasks.speculative.execution = true;
```

## 内存与资源优化

### Hive 内存配置

```
-- Map 内存
SET mapreduce.map.memory.mb = 4096;
SET mapreduce.map.java.opts = -Xmx3072m;

-- Reduce 内存
SET mapreduce.reduce.memory.mb = 8192;
SET mapreduce.reduce.java.opts = -Xmx6144m;

-- Hive 特定内存
SET hive.map.aggr = true;                    -- Map 端预聚合
SET hive.groupby.skewindata = true;          -- 数据倾斜时两阶段聚合
```

### JVM 优化

```
-- JVM 重用
SET mapred.job.reuse.jvm.num.tasks = 10;

-- 垃圾回收优化
SET mapred.child.java.opts = "-XX:+UseG1GC -XX:MaxGCPauseMillis=200";
```

## 优化检查清单

| 检查项 | 优化建议 |
| --- | --- |
| 数据倾斜 | 使用随机前缀、Skew Join、两阶段聚合 |
| JOIN 效率 | 小表放左边、使用 MapJoin、Bucket Join |
| 分区裁剪 | WHERE 条件直接比较分区列 |
| 列裁剪 | 避免 SELECT \*，只选需要的列 |
| 存储格式 | 生产环境使用 ORC + ZLIB/SNAPPY |
| 执行引擎 | 优先使用 Tez 或 Spark |
| 小文件 | 设置合并参数，控制 reducer 数量 |
| 统计信息 | 定期执行 ANALYZE TABLE |
| 向量化 | 使用 ORC 格式，开启向量化执行 |

## 总结

Hive 性能优化是一个系统工程，需要从多个层面入手：

2. **数据层面**：合理分区、分桶，选择合适存储格式
4. **SQL 层面**：优化 JOIN 顺序，避免数据倾斜
6. **执行层面**：选择合适引擎，调整并行度
8. **资源层面**：合理配置内存，启用压缩

掌握这些优化技巧，能让你的 Hive 查询从小时级降到分钟级。

---

**参考链接**  
- Hive Performance Tuning  
- Hive Configuration Properties