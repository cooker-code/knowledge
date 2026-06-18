> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkUDF自定义函数|FlinkUDF自定义函数]]
---
title: Flink SQL 实战
author: DATA炼狱
date:
url: https://mp.weixin.qq.com/s?__biz=MzUyMDkzNjAzMQ==&mid=2247484365&idx=1&sn=c4262705c56a03a6efc3d9ddf5e1604d&chksm=f878f03f6bb255421b213086bac99ddad19c3f39d3a3a7b7528c74d76a82df2a66d053e13568&mpshare=1&scene=24&srcid=0411VdoCNYx9pUtnI9iJk0vJ&sharer_shareinfo=5796b1273cf6afd24c34b4eb2d4f4c2f&sharer_shareinfo_first=5796b1273cf6afd24c34b4eb2d4f4c2f#rd
---

# Flink SQL 实战

## Flink SQL 简介

Flink SQL 是 Flink 提供的高层次流批统一 SQL 接口，让不熟悉 Java/Scala 的数据工程师也能轻松进行流式计算。

```
Flink 层次架构：

  SQL / Table API（高层，声明式）
          │ 翻译
          ▼
    DataStream API（中层，程序化）
          │ 翻译
          ▼
    Runtime（低层，执行引擎）
```

**Flink SQL 的核心能力：**

| 特性 | 说明 |
| --- | --- |
| 流批统一 | 同一条 SQL 既可以处理实时流，也可以处理有界批数据 |
| 动态表 | 将无界流抽象为持续更新的"动态表" |
| 时间语义 | SQL 中支持 PROCTIME() 和 ROWTIME 两种时间属性 |
| CDC 支持 | 原生支持 Debezium / Canal 格式的 CDC 数据 |
| 连接器丰富 | Kafka、JDBC、Hive、Iceberg、HBase 等开箱即用 |

---

## 基础用法

### 创建 TableEnvironment

```
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;
import org.apache.flink.table.api.*;

// 流处理场景
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tEnv = StreamTableEnvironment.create(env);

// 纯批处理场景
TableEnvironment batchEnv = TableEnvironment.create(
    EnvironmentSettings.inBatchMode()
);
```

### DDL 建表

```
-- 创建 Kafka Source 表
CREATE TABLE orders (
    order_id     STRING,
    user_id      BIGINT,
    product_id   STRING,
    category     STRING,
    amount       DOUBLE,
    order_status STRING,
    event_time   TIMESTAMP(3),
    -- 定义 Watermark：允许 5 秒乱序
    WATERMARK FOR event_time AS event_time - INTERVAL '5' SECOND
) WITH (
    'connector' = 'kafka',
    'topic'     = 'orders',
    'properties.bootstrap.servers' = 'kafka:9092',
    'properties.group.id'          = 'flink-sql-group',
    'scan.startup.mode'            = 'latest-offset',
    'format'    = 'json',
    'json.timestamp-format.standard' = 'ISO-8601'
);

-- 创建 MySQL Sink 表
CREATE TABLE order_stats (
    category      STRING,
    window_start  TIMESTAMP(3),
    window_end    TIMESTAMP(3),
    total_amount  DOUBLE,
    order_count   BIGINT,
    PRIMARY KEY (category, window_start) NOT ENFORCED
) WITH (
    'connector' = 'jdbc',
    'url'       = 'jdbc:mysql://mysql:3306/realtime',
    'table-name'= 'order_stats',
    'username'  = 'root',
    'password'  = '123456'
);

-- 创建 Print Sink（调试用）
CREATE TABLE print_sink WITH ('connector' = 'print') LIKE orders (EXCLUDING ALL);
```

### 基础查询

```
-- 普通过滤查询（在流上持续执行）
SELECT user_id, order_id, amount
FROM orders
WHERE amount > 100 AND order_status = 'paid';

-- 聚合（按 category 统计总金额，会持续更新）
SELECT
    category,
    SUM(amount)  AS total_amount,
    COUNT(*)     AS order_count
FROM orders
GROUP BY category;
```

---

## 时间属性

### 处理时间（Processing Time）

```
CREATE TABLE user_clicks (
    user_id    STRING,
    page_id    STRING,
    -- 处理时间：系统自动维护，无需从数据中提取
    proc_time  AS PROCTIME()
) WITH (
    'connector' = 'kafka',
    'topic'     = 'clicks',
    ...
);
```

### 事件时间（Event Time）

```
CREATE TABLE orders (
    order_id    STRING,
    amount      DOUBLE,
    -- 方式1：数据中直接包含 TIMESTAMP 字段
    event_time  TIMESTAMP(3),
    WATERMARK FOR event_time AS event_time - INTERVAL '5' SECOND

    -- 方式2：数据中是 BIGINT 毫秒时间戳，需要转换
    -- ts_ms      BIGINT,
    -- event_time AS TO_TIMESTAMP_LTZ(ts_ms, 3),
    -- WATERMARK FOR event_time AS event_time - INTERVAL '5' SECOND
) WITH (...);
```

---

## 窗口聚合

### TVF 窗口函数（Flink 1.13+ 推荐写法）

```
-- 滚动窗口：每1分钟统计各品类销售额
SELECT
    category,
    window_start,
    window_end,
    SUM(amount)  AS total_amount,
    COUNT(*)     AS order_count
FROM TABLE(
    TUMBLE(TABLE orders, DESCRIPTOR(event_time), INTERVAL '1' MINUTE)
)
GROUP BY category, window_start, window_end;

-- 滑动窗口：5分钟内，每1分钟统计一次
SELECT
    category,
    window_start,
    window_end,
    SUM(amount) AS total_amount
FROM TABLE(
    HOP(TABLE orders, DESCRIPTOR(event_time), INTERVAL '1' MINUTE, INTERVAL '5' MINUTE)
)
GROUP BY category, window_start, window_end;

-- 会话窗口：超过 5 分钟无活动则关闭
SELECT
    user_id,
    window_start,
    window_end,
    COUNT(*) AS click_count
FROM TABLE(
    SESSION(TABLE user_clicks, DESCRIPTOR(event_time), INTERVAL '5' MINUTE)
)
GROUP BY user_id, window_start, window_end;
```

### 累积窗口（Cumulate Window）

累积窗口从固定起点开始，按步长累积，最终到达最大窗口大小。适合"每小时统计今日累计销售额"：

```
-- 每1分钟输出一次，但汇总从每天0点开始的累计数据
SELECT
    category,
    window_start,
    window_end,
    SUM(amount) AS cumulative_amount
FROM TABLE(
    CUMULATE(TABLE orders, DESCRIPTOR(event_time),
             INTERVAL '1' MINUTE,   -- 步长
             INTERVAL '1' DAY)      -- 最大窗口
)
GROUP BY category, window_start, window_end;
```

---

## Over 窗口（行级滑动聚合）

Over 窗口对每条记录都计算其前后若干行或时间范围内的聚合值，适合移动平均、排名等：

```
-- 每个用户过去 1 小时内的累计消费（时间范围 Over 窗口）
SELECT
    user_id,
    order_id,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY event_time
        RANGE BETWEEN INTERVAL '1' HOUR PRECEDING AND CURRENT ROW
    ) AS amount_1h
FROM orders;

-- 每个用户最近 5 笔订单的平均金额（行数 Over 窗口）
SELECT
    user_id,
    order_id,
    amount,
    AVG(amount) OVER (
        PARTITION BY user_id
        ORDER BY event_time
        ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
    ) AS avg_amount_5
FROM orders;
```

---

## 流表 Join

### Regular Join（双流 Join，保留历史状态）

```
-- 订单与用户信息 Join（两个都是流）
SELECT
    o.order_id,
    o.amount,
    u.username,
    u.city
FROM orders AS o
JOIN users AS u ON o.user_id = u.user_id;
-- 注意：Regular Join 会保留两侧全部历史状态，状态会无限增长，需设置 TTL
```

### Interval Join（时间区间 Join）

```
-- 订单在支付后 30 分钟内发货
SELECT
    o.order_id,
    o.amount,
    s.ship_time
FROM orders AS o, shipments AS s
WHERE o.order_id = s.order_id
  AND s.ship_time BETWEEN o.event_time
                      AND o.event_time + INTERVAL '30' MINUTE;
```

### Temporal Join（维表 Join，查询维表当时的版本）

```
-- 定义带版本的汇率维表（Kafka + upsert 格式）
CREATE TABLE exchange_rates (
    currency    STRING,
    rate        DOUBLE,
    updated_at  TIMESTAMP(3),
    PRIMARY KEY (currency) NOT ENFORCED,
    WATERMARK FOR updated_at AS updated_at - INTERVAL '5' SECOND
) WITH (
    'connector' = 'upsert-kafka',
    'topic'     = 'exchange_rates',
    ...
);

-- 用订单发生时的汇率转换金额
SELECT
    o.order_id,
    o.amount,
    o.amount * r.rate AS amount_usd
FROM orders AS o
LEFT JOIN exchange_rates FOR SYSTEM_TIME AS OF o.event_time AS r
    ON o.currency = r.currency;
```

### Lookup Join（查询外部系统维表）

```
-- 查询 MySQL 中的用户维表（会自动缓存）
CREATE TABLE dim_users (
    user_id   BIGINT,
    username  STRING,
    city      STRING,
    level     INT,
    PRIMARY KEY (user_id) NOT ENFORCED
) WITH (
    'connector'        = 'jdbc',
    'url'              = 'jdbc:mysql://mysql:3306/dim',
    'table-name'       = 'users',
    'lookup.cache.max-rows' = '10000',
    'lookup.cache.ttl'      = '10min'
);

-- Lookup Join
SELECT
    o.order_id,
    o.amount,
    u.username,
    u.city
FROM orders AS o
JOIN dim_users FOR SYSTEM_TIME AS OF o.proc_time AS u
    ON o.user_id = u.user_id;
```

---

## Flink SQL CDC：实时数仓建设

### 读取 MySQL CDC 数据

```
-- 读取 MySQL Binlog（需要 flink-cdc-connectors）
CREATE TABLE mysql_orders (
    order_id     BIGINT,
    user_id      BIGINT,
    amount       DOUBLE,
    status       STRING,
    created_at   TIMESTAMP(3),
    PRIMARY KEY (order_id) NOT ENFORCED
) WITH (
    'connector' = 'mysql-cdc',
    'hostname'  = 'mysql',
    'port'      = '3306',
    'username'  = 'root',
    'password'  = '123456',
    'database-name' = 'shop',
    'table-name'    = 'orders'
);

-- CDC 数据包含 +I（插入）、-U（更新前）、+U（更新后）、-D（删除）四种变更类型
-- 可以用于实时同步到 Kafka、Iceberg、Hudi 等

-- 实时同步到 Kafka（upsert-kafka）
INSERT INTO kafka_orders
SELECT order_id, user_id, amount, status
FROM mysql_orders;
```

### 实时数仓 ODS → DWD

```
-- ODS 层：原始 Kafka 数据
CREATE TABLE ods_orders (...) WITH ('connector' = 'kafka', ...);

-- DWD 层：关联维表，写入 Iceberg
CREATE TABLE dwd_order_detail (...) WITH ('connector' = 'iceberg', ...);

-- ETL：过滤无效数据 + 关联用户维表
INSERT INTO dwd_order_detail
SELECT
    o.order_id,
    o.user_id,
    u.username,
    u.city,
    o.amount,
    o.status,
    o.event_time
FROM ods_orders AS o
JOIN dim_users FOR SYSTEM_TIME AS OF o.proc_time AS u
    ON o.user_id = u.user_id
WHERE o.amount > 0 AND o.status IS NOT NULL;
```

---

## 常用内置函数

```
-- 时间函数
CURRENT_TIMESTAMP                    -- 当前处理时间
TO_TIMESTAMP('2024-01-01 10:00:00')  -- 字符串转 TIMESTAMP
DATE_FORMAT(event_time, 'yyyy-MM-dd') -- 格式化时间
TIMESTAMPDIFF(MINUTE, t1, t2)        -- 时间差（分钟）
DATE_ADD(event_time, 1)              -- 加1天

-- 字符串函数
CONCAT(a, '-', b)                    -- 拼接
SUBSTRING(s, 1, 10)                  -- 截取
REGEXP_EXTRACT(s, '(\d+)', 1)        -- 正则提取
SPLIT_INDEX(s, ',', 0)               -- 按分隔符拆分

-- JSON 函数
JSON_VALUE(json_str, '$.order_id')   -- 提取 JSON 字段
JSON_OBJECT('id' VALUE id, 'name' VALUE name) -- 构造 JSON

-- 条件函数
CASE WHEN amount > 1000 THEN '大额' ELSE '普通' END
COALESCE(field1, field2, 'default')  -- 取第一个非 NULL 值
NULLIF(a, b)                         -- 时返回 NULL

-- 聚合函数
COUNT(DISTINCT user_id)              -- 去重计数
FIRST_VALUE(order_id)                -- 窗口内第一个值
LAST_VALUE(order_id)                 -- 窗口内最后一个值
LISTAGG(product_id, ',')            -- 拼接字符串
```

---

## SQL 与 DataStream API 互转

```
// DataStream → Table
DataStream<Order> orderStream = ...;
Table orderTable = tEnv.fromDataStream(orderStream,
    Schema.newBuilder()
        .columnByExpression("event_time", "CAST(ts AS TIMESTAMP_LTZ(3))")
        .watermark("event_time", "event_time - INTERVAL '5' SECOND")
        .build());
tEnv.createTemporaryView("orders", orderTable);

// Table → DataStream
Table resultTable = tEnv.sqlQuery("SELECT * FROM orders WHERE amount > 100");
DataStream<Row> resultStream = tEnv.toDataStream(resultTable);
resultStream.print();

// 在 SQL 中调用自定义函数
tEnv.createTemporaryFunction("myFunc", MyUDF.class);
tEnv.executeSql("SELECT myFunc(amount) FROM orders");
```

---

## 自定义函数（UDF）

```
// 标量函数（一行输入 → 一行输出）
public class AmountFormatter extends ScalarFunction {
    public String eval(Double amount) {
        if (amount == null) return "N/A";
        return String.format("¥%.2f", amount);
    }
}

// 表函数（一行输入 → 多行输出）
public class TagSplitter extends TableFunction<String> {
    public void eval(String tags) {
        if (tags != null) {
            for (String tag : tags.split(",")) {
                collect(tag.trim());
            }
        }
    }
}

// 聚合函数（多行输入 → 一行输出）
public class WeightedAvg extends AggregateFunction<Double, WeightedAvgAccumulator> {
    @Override
    public WeightedAvgAccumulator createAccumulator() {
        return new WeightedAvgAccumulator();
    }

    public void accumulate(WeightedAvgAccumulator acc, Double value, Integer weight) {
        acc.sum += value * weight;
        acc.count += weight;
    }

    @Override
    public Double getValue(WeightedAvgAccumulator acc) {
        return acc.count == 0 ? 0.0 : acc.sum / acc.count;
    }
}

// 注册并使用
tEnv.createTemporaryFunction("format_amount", AmountFormatter.class);
tEnv.executeSql("SELECT format_amount(amount) FROM orders");
```