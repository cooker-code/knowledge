---
title: Flink SQL语法 和 Flink Streaming API 函数的对应关系是怎么样的？
author: 大数据技能圈
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247492014&idx=1&sn=836a0d048e54a2e4aa8436e2ecfb4020&chksm=c1055e6fbd277c323e64f3884678527e4a462f1c0511769aa17318655df35fed5485785af6d6&mpshare=1&scene=24&srcid=11253UTWQbXJwrUH16EpEGhT&sharer_shareinfo=c106de4ac16c3952b0fe722f4e86540a&sharer_shareinfo_first=c106de4ac16c3952b0fe722f4e86540a#rd
---

今天我们聊聊 Apache Flink 的SQL 和 Streaming API的对应关系。

 

 

## 前提：Flink 的两种编程模型

Flink 提供了两种主要的流处理编程方式：

1. 1. **DataStream API（Streaming API）**：这是 Flink 最底层、最灵活的 API，允许你通过 Java/Scala 编写算子（如 map、filter、keyBy、window 等）来构建流处理逻辑。
2. 2. **Flink SQL / Table API**：这是更高层的声明式 API，你可以像写 SQL 一样表达计算逻辑。Flink 内部会将 SQL 转换成 DataStream 程序来执行。

所以，**Flink SQL 是对 DataStream API 的封装和抽象**。当你写一句 SQL，Flink 会将其解析、优化，最终转换成一系列 DataStream 算子来运行。

---

## 核心概念对照

### 1. 表（Table） ↔ DataStream

* • 在 Flink SQL 中，数据源被抽象为“表”（Table），可以是有限的（批）或无限的（流）。
* • 在 DataStream API 中，数据以 `DataStream<T>` 的形式存在，T 是具体的数据类型（如 POJO、Tuple、Row 等）。
* • 两者可以互相转换：

+ • `tableEnv.fromDataStream(dataStream)` → Table
+ • `tableEnv.toDataStream(table)` → DataStream

> 注意：SQL 默认处理的是“动态表”（Dynamic Table），即随时间不断变化的表，这与流的本质一致。

---

### 2. SELECT 字段投影 ↔ map / flatMap

SQL 中的 `SELECT a, b` 表示只保留某些字段，或者对字段做简单计算（如 `a + 1`）。

* • 对应到 DataStream API，就是 `map` 或 `flatMap`。
* • 例如：

  ```
  SELECT id, name, age + 1 AS new_age FROM users
  ```

  相当于：

  ```
  dataStream.map(user -> new User(user.id, user.name, user.age + 1));
  ```

> 如果 SELECT 中包含复杂表达式（如函数调用），Flink 会生成对应的 `map` 函数来执行。

---

### 3. WHERE 条件过滤 ↔ filter

SQL 的 `WHERE` 用于筛选满足条件的行。

* • 对应 DataStream 的 `filter` 算子。
* • 例如：

  ```
  SELECT * FROM orders WHERE amount > 100
  ```

  等价于：

  ```
  orders.filter(order -> order.amount > 100);
  ```

---

### 4. GROUP BY ↔ keyBy + window（或非窗口聚合）

这是最关键也最容易混淆的部分。

#### 情况一：带窗口的 GROUP BY（最常见）

在流处理中，**不能对无限流做全局 GROUP BY**（因为数据永不停止，无法知道“所有”数据是否到达）。所以 Flink SQL 要求 GROUP BY 必须配合 **时间窗口**（或处理时间/事件时间语义）。

例如：

```
SELECT user_id, SUM(amount)  
FROM orders  
GROUP BY user_id, TUMBLE(proctime, INTERVAL '5' MINUTE)
```

这个 SQL 的含义是：每 5 分钟一个滚动窗口，按 user\_id 分组，统计每个窗口内的总金额。

**对应到 DataStream API：**

1. 1. 首先用 `keyBy("user_id")` 按 user\_id 分区；
2. 2. 然后应用一个 **窗口（WindowAssigner）**，这里是 `TumblingProcessingTimeWindows.of(Time.minutes(5))`；
3. 3. 最后用 `reduce` 或 `aggregate` 做聚合。

完整代码类似：

```
orders  
  .keyBy("user_id")  
  .window(TumblingProcessingTimeWindows.of(Time.minutes(5)))  
  .sum("amount");
```

所以，**带窗口的 GROUP BY = keyBy + window + 聚合算子（sum/min/max/agg）**

> 注意：SQL 中的 `TUMBLE`、`HOP`、`SESSION` 等窗口函数，分别对应 DataStream 中的 `TumblingEventTimeWindows`、`SlidingEventTimeWindows`、`EventTimeSessionWindows` 等。

#### 情况二：无窗口的 GROUP BY（仅限于特定场景）

Flink 1.12+ 支持 **“持续查询”（Continuous Query）** 下的无窗口 GROUP BY，但这实际上隐含了 **“累积聚合”** 语义，并且必须配合 **Changelog 流** 输出（即输出的是“当前最新结果”，而不是一次性结果）。

例如：

```
SELECT user_id, COUNT(*)  
FROM orders  
GROUP BY user_id
```

这在流上意味着：每当新订单到来，就更新该 user\_id 的计数，并输出最新的计数值（带 retract 信息）。

**对应到 DataStream API：**

* • 这实际上是 **keyBy + 全局状态 + 持续输出**。
* • 底层使用 `keyBy().process(new KeyedProcessFunction())`，并在状态中维护每个 key 的聚合值。
* • 每次新数据到来，读取旧状态 → 更新 → 写回状态 → 输出新结果。
* • 同时可能输出 retract 消息（旧值撤回，新值插入），以支持下游正确消费。

> 所以，无窗口的 GROUP BY ≈ keyBy + ProcessFunction + ValueState（或 MapState）+ 持续 emit

但请注意：这种模式在 DataStream API 中没有直接的一对一算子，而是需要手动实现状态管理。而 SQL 引擎自动帮你做了这些。

---

### 5. JOIN ↔ connect / join / intervalJoin / temporal join

JOIN 在流处理中比批处理复杂得多，因为涉及时间对齐问题。

#### （1）普通 INNER JOIN（两条流）

```
SELECT a.id, a.name, b.amount  
FROM users AS a  
JOIN orders AS b  
ON a.id = b.user_id
```

这个语句在流上 **必须定义时间范围**，否则无法执行（因为两条流都是无限的，不知道等多久才算“匹配完成”）。

Flink 要求使用 **时间窗口 JOIN** 或 **Temporal Table Join**。

##### 时间窗口 JOIN（Window Join）

```
SELECT *  
FROM orders o  
JOIN shipments s  
ON o.id = s.order_id  
AND o.proc_time BETWEEN s.proc_time - INTERVAL '1' HOUR AND s.proc_time + INTERVAL '1' HOUR
```

对应 DataStream API 的 `intervalJoin`：

```
orders.keyBy(o -> o.id)  
      .intervalJoin(shipments.keyBy(s -> s.order_id))  
      .between(Time.hours(-1), Time.hours(1))  
      .process(new IntervalJoinFunction());
```

所以，**Window JOIN ↔ intervalJoin**

##### Temporal Table Join（维表关联）

用于将流与“版本化历史表”（如 MySQL CDC 流）关联：

```
SELECT o.id, u.name  
FROM orders o  
JOIN users FOR SYSTEM_TIME AS OF o.proc_time AS u  
ON o.user_id = u.id
```

对应 DataStream API 的 **Async I/O + Lookup** 或 **Temporal Join with State**。

Flink 内部会用 `keyBy().process()` 维护一个按时间版本索引的状态表，然后对每条订单查找当时有效的用户快照。

> 所以，Temporal Join ≈ keyed process function + versioned state（或外部系统 lookup）

---

### 6. OVER 窗口 ↔ keyBy + window（但特殊类型）

SQL 中的 OVER 窗口用于计算“每行的滑动聚合”，比如“过去 1 小时的累计销售额”。

```
SELECT id,  
       amount,  
       SUM(amount) OVER (PARTITION BY user_id ORDER BY proc_time RANGE BETWEEN INTERVAL '1' HOUR PRECEDING AND CURRENT ROW) AS sum_last_hour  
FROM orders
```

这在 DataStream 中对应 **OverWindow**，但注意：DataStream API 本身没有直接叫 `overWindow` 的算子。

Flink 内部会将其转换为：

* • `keyBy("user_id")`
* • 使用 `ProcessFunction` 或专用 `OverWindowOperator`，维护一个按时间排序的队列（或状态），动态计算滑动范围内的聚合值。

所以，**OVER 窗口 ≈ keyBy + ProcessFunction + 状态管理（带时间清理）**

---

### 7. INSERT INTO ↔ addSink

SQL 的 `INSERT INTO result_table SELECT ...` 表示将结果写入某个 sink。

对应 DataStream 的 `.addSink(sinkFunction)`。

例如：

```
INSERT INTO kafka_result  
SELECT user_id, SUM(amount)  
FROM orders  
GROUP BY user_id, TUMBLE(proc_time, INTERVAL '5' MINUTE)
```

等价于：

```
resultStream.addSink(new KafkaSink(...));
```

---

### 8. UNION ↔ union

SQL 的 `UNION ALL` 对应 DataStream 的 `union()`。

```
SELECT * FROM table1  
UNION ALL  
SELECT * FROM table2
```

等价于：

```
stream1.union(stream2);
```

> 注意：Flink SQL 不支持 `UNION`（去重），只支持 `UNION ALL`（不去重），因为流上去重成本极高。

---

### 9. DISTINCT ↔ keyBy + deduplication

SQL：

```
SELECT DISTINCT user_id FROM orders
```

在流上，这通常意味着“每个 user\_id 只输出一次”。

对应 DataStream：

* • `keyBy("user_id")`
* • 用 `ProcessFunction` + `ValueState<Boolean>` 记录是否已输出过
* • 第一次出现时输出，之后忽略

所以，**DISTINCT ≈ keyBy + 状态去重**

---

## 总结：核心映射表

| Flink SQL 操作 | 对应的 DataStream API 实现 |
| --- | --- |
| `SELECT` 投影/计算 | `map` / `flatMap` |
| `WHERE` 过滤 | `filter` |
| `GROUP BY` + 窗口 | `keyBy` + `window` + `reduce`/`aggregate` |
| `GROUP BY` （无窗口，持续聚合） | `keyBy` + `ProcessFunction` + 状态（ValueState/MapState） |
| `JOIN` （Window Join） | `intervalJoin` |
| `JOIN` （Temporal Table Join） | `keyed ProcessFunction` + 版本化状态 或 Async I/O |
| `OVER` 窗口 | `keyBy` + `ProcessFunction` + 时间窗口状态管理 |
| `INSERT INTO` | `addSink` |
| `UNION ALL` | `union` |
| `DISTINCT` | `keyBy` + 状态去重 |

---

## 补充说明：时间语义

无论是 SQL 还是 DataStream API，都依赖 **时间属性**（Processing Time 或 Event Time）。

* • 在 SQL 中，你需要在建表时声明时间字段：

  ```
  CREATE TABLE orders (  
    id STRING,  
    amount INT,  
    ts AS PROCTIME()  -- 处理时间  
    -- 或 ts TIMESTAMP(3), WATERMARK FOR ts AS ts - INTERVAL '5' SECOND -- 事件时间  
  )
  ```
* • 在 DataStream 中，你需要调用 `.assignTimestampsAndWatermarks(...)` 来分配时间戳和水位线。

**GROUP BY、JOIN、OVER 等操作的行为，完全取决于你使用的是哪种时间语义。**

---

## 最后提醒

* • Flink SQL 是声明式的，你告诉系统“要什么”；DataStream API 是过程式的，你告诉系统“怎么做”。
* • SQL 更简洁，适合业务逻辑清晰的场景；DataStream 更灵活，适合需要精细控制状态、时间、容错的场景。
* • 所有 Flink SQL 最终都会被翻译成 DataStream 程序（通过 Calcite 解析 + Flink Planner 优化 + CodeGen 生成算子）。

理解这种对应关系，能让你在调试性能、排查问题或混合使用两种 API 时更加得心应手。

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

本学习文档已放入星球，扫描二维码加入星球获取