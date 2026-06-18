> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkSQL流批统一与混合编程|FlinkSQL流批统一与混合编程]]
---
title: Flink高频问题-Flink SQL 与 DataStream API 区别？何时选择哪种？
author: 算法驱动的数据圈
date:
url: https://mp.weixin.qq.com/s?__biz=MzI3ODE4MjczNA==&mid=2651507927&idx=1&sn=a25f457107244bb543602b13d478e9a9&chksm=f1fa62ab23cde30c79c738f8469ea386b3d3fa0fd3551f4570ad20f234b98d434053626fe8b1&mpshare=1&scene=24&srcid=0414fzj7DGAxc3umrwGpUeMp&sharer_shareinfo=7ceeb4c1474375a519d4054f3ceef02e&sharer_shareinfo_first=7ceeb4c1474375a519d4054f3ceef02e#rd
---

Flink 提供两种核心编程模型：**DataStream API**（低阶过程式 API）和 **Flink SQL**（高阶声明式 API）。两者底层共享 Flink 执行引擎（Checkpoint、状态管理、Watermark 机制一致），但抽象层级、开发模式、适用场景差异显著。本文从 “核心区别→维度对比→选型逻辑” 展开，帮你快速判断何时选择哪种 API。

## 一、核心定位与本质区别

### 1. 核心定位

* **DataStream API**

  面向 Java/Scala 开发者的**低阶过程式 API**，需手动定义数据处理的 “每一步流程”（如算子串联、状态管理、窗口配置），灵活性极高，掌控力强；
* **Flink SQL**

  面向 SQL 开发者 / 分析师的**高阶声明式 API**，通过 SQL/Table API 描述 “想要的结果”（如聚合、关联、窗口统计），无需关心底层执行细节（Flink 优化器自动生成执行计划），开发效率极高。

### 2. 本质区别：“怎么做” vs “要什么”

* DataStream API：你需要告诉 Flink“**数据如何处理**”—— 比如 “先 keyBy 分组，再用 10 分钟滚动窗口，最后通过 AggregateFunction 累加销售额”，全程手动控制流程；
* Flink SQL：你只需要告诉 Flink“**最终要什么结果**”—— 比如 “SELECT product\_id, SUM (amount) FROM orders GROUP BY product\_id, TUMBLE (create\_time, INTERVAL '10' MINUTE)”，底层执行计划（如并行度、聚合顺序）由 Flink 自动优化。

## 二、全维度对比表（面试 + 选型必看）

| 对比维度 | DataStream API | Flink SQL / Table API |
| --- | --- | --- |
| **抽象层级** | 低阶 API（过程式） | 高阶 API（声明式） |
| **开发效率** | 低：需手动编写算子、状态、窗口逻辑，代码量大 | 高：SQL 语句简洁，无需关心底层细节，支持快速迭代 |
| **灵活性** | 极高：支持自定义状态（KeyedState/OperatorState）、触发器、侧输出流、CEP 等复杂逻辑 | 中等：支持大部分常见场景（聚合、关联、窗口），复杂逻辑需通过 UDF/UDTF 扩展，受 SQL 语法限制 |
| **性能优化** | 手动优化（需开发者合并算子、预聚合、调整并行度），优化空间大 | 自动优化（Flink 优化器支持谓词下推、聚合重排序、Join 重分区等），普通场景性能稳定，高阶优化需调优参数 |
| **状态管理** | 手动管理：需定义状态描述符、控制状态 TTL、清理策略 | 自动管理：Flink 自动维护状态，支持通过 DDL 配置状态 TTL（如 `TBLPROPERTIES ('state.ttl' = '86400s')`） |
| **时间语义支持** | 手动配置：需显式指定 EventTime/ProcessingTime、Watermark 生成逻辑 | 自动适配：通过 DDL 声明时间字段（如 `WATERMARK FOR create_time AS create_time - INTERVAL '5' SECOND`），简化配置 |
| **适用人群** | Java/Scala 开发工程师、Flink 进阶用户 | 数据分析师、ETL 工程师、熟悉 SQL 的开发人员，无需深入理解 Flink 底层 |
| **调试难度** | 高：需调试算子逻辑、状态流转、Watermark 推进，依赖 Flink UI / 日志 | 低：SQL 语法直观，可通过 `EXPLAIN` 查看执行计划，调试成本低 |
| **生态集成** | 需手动集成外部系统（如 Kafka、MySQL），支持自定义 Source/Sink | 内置丰富 Connector（Kafka、JDBC、Hive 等），可通过 DDL 直接创建表（如 `CREATE TABLE kafka_orders (...) WITH (...)`），集成更便捷 |

## 三、典型适用场景（何时选哪种？）

### 1. 优先选择 Flink SQL 的场景

* **实时数仓建设**

  如构建实时 ODS/DWD/DWS 层，需进行数据过滤、清洗、聚合、关联（如订单表关联商品表），SQL 语法适配数仓开发习惯；
* **简单实时统计**

  如按固定窗口统计销售额、用户活跃度、日志报错数，无需复杂业务逻辑；
* **ETL 数据同步**

  如将 Kafka 中的实时数据清洗后写入 MySQL/Hive，或从 MySQL 同步增量数据到 Kafka，通过 `INSERT INTO ... SELECT` 快速实现；
* **团队技术栈以 SQL 为主**

  数据分析师、ETL 工程师主导开发，无需编写 Java/Scala 代码，降低技术门槛；
* **快速迭代需求**

  如业务需求频繁变更（如调整统计维度、窗口大小），SQL 可快速修改并上线，无需重构大量代码。

#### 示例：Flink SQL 实现 10 分钟订单销售额统计

```
-- 1. 创建 Kafka 订单源表（声明事件时间和 Watermark）CREATE TABLE kafka_orders (    order_id STRING,    product_id STRING,    amount DECIMAL(10,2),    create_time TIMESTAMP(3),    WATERMARK FOR create_time AS create_time - INTERVAL '5' SECOND -- 允许 5 秒延迟) WITH (    'connector' = 'kafka',    'topic' = 'order-topic',    'properties.bootstrap.servers' = 'kafka:9092',    'scan.startup.mode' = 'latest-offset',    'format' = 'json');-- 2. 10 分钟滚动窗口统计商品销售额CREATE TABLE product_sales (    product_id STRING,    window_start TIMESTAMP(3),    window_end TIMESTAMP(3),    total_amount DECIMAL(10,2)) WITH (    'connector' = 'kafka',    'topic' = 'product-sales-topic',    'properties.bootstrap.servers' = 'kafka:9092',    'sink.partitioner' = 'round-robin');-- 3. 执行统计并写入结果表INSERT INTO product_salesSELECT     product_id,    TUMBLE_START(create_time, INTERVAL '10' MINUTE) AS window_start,    TUMBLE_END(create_time, INTERVAL '10' MINUTE) AS window_end,    SUM(amount) AS total_amountFROM kafka_ordersGROUP BY product_id, TUMBLE(create_time, INTERVAL '10' MINUTE);
```

### 2. 优先选择 DataStream API 的场景

* **复杂状态管理**

  如需要自定义状态（如 MapState 存储用户行为序列、ListState 存储历史订单）、精细控制状态 TTL 和清理策略；
* **自定义窗口 / 触发器**

  如非标准窗口（如基于业务规则的动态窗口）、自定义触发逻辑（如收到外部信号后触发计算）；
* **复杂事件处理（CEP）**

  如实时风控场景（检测连续 3 次登录失败、1 分钟内下单超 5 笔），需使用 Flink CEP 库（仅 DataStream API 支持）；
* **异步 IO 处理**

  如算子中需要异步调用外部系统（如 HTTP 接口、数据库），需手动实现 `AsyncFunction` 控制线程池和超时逻辑；
* **特殊 Source/Sink 开发**

  如对接自定义数据源（如物联网设备协议、企业私有消息队列），需实现 `SourceFunction`/`SinkFunction`；
* **极致性能优化**

  如高吞吐场景（每秒千万级数据），需手动合并算子、调整 Shuffle 策略、优化序列化逻辑，压榨系统性能。

#### 示例：DataStream API 实现 CEP 风控检测（连续 2 笔订单金额超 1000 元）

```
// 1. 定义订单流DataStream<Order> orderStream = env.addSource(new KafkaOrderSource())    .assignTimestampsAndWatermarks(        WatermarkStrategy.<Order>forBoundedOutOfOrderness(Duration.ofSeconds(5))            .withTimestampAssigner((order, ts) -> order.getCreateTime())    );// 2. 定义 CEP 模式（连续 2 笔订单金额 > 1000 元，间隔 < 5 分钟）Pattern<Order, ?> highValuePattern = Pattern.<Order>begin("firstOrder")    .where(order -> order.getAmount() > 1000)    .next("secondOrder")    .where(order -> order.getAmount() > 1000)    .within(Time.minutes(5));// 3. 应用模式并检测风险订单PatternStream<Order> patternStream = CEP.pattern(orderStream.keyBy(Order::getUserId), highValuePattern);patternStream.process(new PatternProcessFunction<Order, RiskAlert>() {    @Override    public void processMatch(Map<String, List<Order>> match, Context ctx, Collector<RiskAlert> out) {        Order first = match.get("firstOrder").get(0);        Order second = match.get("secondOrder").get(0);        out.collect(new RiskAlert(            first.getUserId(),            "连续 2 笔高价值订单",            ctx.timestamp()        ));    }}).print("风控告警：");
```

### 3. 混合使用场景（DataStream + Flink SQL）

Flink 支持两者无缝集成，可发挥各自优势：

* 场景 1：DataStream 处理复杂逻辑（如 CEP 检测、自定义状态）后，转换为 Table 用 SQL 做聚合统计；
* 场景 2：Flink SQL 读取 / 清洗数据后，转换为 DataStream 进行特殊处理（如异步调用、自定义 Sink）。

#### 示例：DataStream 与 Flink SQL 混合使用

```
// 1. DataStream 读取 Kafka 并进行 CEP 检测DataStream<Order> orderStream = ...; // 订单流DataStream<RiskAlert> riskStream = CEP.pattern(...) // CEP 检测风险订单// 2. 将 DataStream 转换为 Table，用 SQL 统计风险订单分布StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);tableEnv.createTemporaryView("risk_alerts", riskStream,     $("user_id"), $("alert_type"), $("alert_time").rowtime());// 3. Flink SQL 统计 10 分钟内各用户风险告警次数Table resultTable = tableEnv.sqlQuery("""    SELECT         user_id,        TUMBLE_START(alert_time, INTERVAL '10' MINUTE) AS window_start,        COUNT(*) AS alert_count    FROM risk_alerts    GROUP BY user_id, TUMBLE(alert_time, INTERVAL '10' MINUTE)""");// 4. 转换回 DataStream 写入 KafkaDataStream<Tuple3<String, Long, Long>> resultStream = tableEnv.toAppendStream(    resultTable, DataTypes.TUPLE(        DataTypes.STRING(),        DataTypes.TIMESTAMP(3),        DataTypes.LONG()    ).toRowDataType()).map(row -> Tuple3.of(    row.getFieldAsString(0),    ((Timestamp) row.getField(1)).getTime(),    row.getFieldAsLong(2)));resultStream.addSink(new KafkaSink<>());
```

## 四、选型决策流程（3 步快速判断）

1. **看业务复杂度**

* 简单逻辑（过滤、聚合、关联、固定窗口）→ Flink SQL；
* 复杂逻辑（自定义状态、CEP、动态窗口、异步 IO）→ DataStream API；

2. **看团队技术栈**

* 熟悉 SQL、无 Java/Scala 开发能力 → Flink SQL；
* Java/Scala 工程师、需深度定制 → DataStream API；

3. **看迭代与性能需求**

* 快速迭代、需求频繁变更 → Flink SQL；
* 极致性能、底层优化 → DataStream API。

## 五、面试总结话术（直接套用）

“Flink SQL 是高阶声明式 API，优势是开发效率高、无需关心底层细节，适合实时数仓、ETL、简单统计，适配 SQL 技术栈团队；DataStream API 是低阶过程式 API，优势是灵活性极高，支持自定义状态、CEP、复杂窗口，适合复杂业务逻辑和极致性能优化。选型时先看业务复杂度，简单场景选 Flink SQL，复杂场景选 DataStream；再结合团队技术栈和迭代需求，也可混合使用发挥各自优势。两者底层共享 Flink 执行引擎，核心能力（状态管理、容错）一致。”