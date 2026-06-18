---
title: Apache Flink 2.1.0: 面向实时 Data + AI 全面升级，开启智能流处理新纪元
author: Apache Flink
date:
url: https://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247514330&idx=1&sn=90f8b0d282db99fedd88ccafb66add4f&chksm=fcbf94ab2df030d2248e35dae9c983c7e895bd261877ef738f739696c61f3953228fab031e8c&mpshare=1&scene=24&srcid=0731ljrKHNmALhpitFNzlTyv&sharer_shareinfo=be90bcd9a3f9a873fad42c9dfebcbb10&sharer_shareinfo_first=be90bcd9a3f9a873fad42c9dfebcbb10#rd
---

> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_版本记录|版本记录]]


# Apache Flink PMC （项目管理委员会）很高兴地宣布 Apache Flink 2.1.0 版本正式发布，这标志着实时数据处理引擎向统一 Data + AI 平台的里程碑式演进。本次版本汇聚全球 116 位贡献者，完成 16 项改进提案（FLIPs），解决 220 多个问题，重点强化了实时 AI 与智能流处理的深度融合：

* 实时 AI 能力突破：

+ 新增 AI 模型 DDL，支持通过 Flink SQL 与 Table API 创建和修改 AI 模型，实现 AI 模型的灵活管理。
+ 扩展 `ML_PREDICT` 表值函数，支持通过 Flink SQL 实时调用 AI 模型，为构建端到端实时 AI 工作流奠定基础。

* 实时数据处理增强：

+ 通过开放 Flink 核心能力——托管状态、事件时间流处理及表变更日志，`Process Table Functions(PTFs)`使 Flink SQL 解锁了更强大的事件驱动型应用开发能力
+ 引入 `VARIANT`数据类型，高效处理 JSON 等半结构化数据，结合`PARSE_JSON`函数与湖仓格式（如 Paimon），实现动态 Schema 数据分析。
+ 重点优化流式 Join，创新性的引入 `DeltaJoin`与 `MultiJoin`策略，消除状态瓶颈，提升资源利用率与作业稳定性。

Flink 2.1.0 将实时数据处理与 AI 模型无缝集成，推动企业从实时分析迈向实时智能决策，以满足现代数据应用不断变化的需求。感谢各位贡献者的支持！

# Flink SQL 提升

## Model DDLs 支持

自 Flink 2.0 引入 AI 模型相关 SQL 语法后，用户可通过类似创建 Catalog 对象的方式定义模型，并在 Flink SQL 中调用 AI 模型。Flink 2.1进一步扩展支持通过 Table API（Java/Python） 定义模型，通过编程实现模型的灵活管理。

示例：

* 使用 Flink SQL 创建模型

```
CREATE MODEL my_modelINPUT (f0 STRING)OUTPUT (label STRING)WITH (  'task' = 'classification',  'type' = 'remote',  'provider' = 'openai',  'openai.endpoint' = 'remote',  'openai.api_key' = 'abcdefg',);
```

* 使用 Java API 创建模型

```
tEnv.createModel(    "MyModel",     ModelDescriptor.forProvider("OPENAI")      .inputSchema(Schema.newBuilder()        .column("f0", DataTypes.STRING())        .build())      .outputSchema(Schema.newBuilder()        .column("label", DataTypes.STRING())        .build())      .option("task", "classification")      .option("type", "remote")      .option("provider", "openai")      .option("openai.endpoint", "remote")      .option("openai.api_key", "abcdefg")      .build(),true);
```

更多信息：

* FLIP-437: Support ML Models in Flink SQL[1]
* FLIP-507: Add Model DDL methods in TABLE API[2]

## 实时 AI 函数

基于模型 DDL，Flink 2.1扩展了 `ML_PREDICT`表值函数（TVF），支持在 Flink SQL 中实时调用机器学习模型，提供内置兼容 OpenAI API 的模型调用支持，同时开放自定义模型接口，标志着 Flink 从实时数据处理引擎向统一的实时 Data + AI 平台演进。未来将继续扩展 `ML_EVALUATE`、`VECTOR_SEARCH` 等函数，用户可以使用 Flink SQL 更方便地构建端到端实时 AI 工作流。

```
-- Declare a AI modelCREATE MODEL `my_model`INPUT (text STRING)OUTPUT (response STRING)WITH(  'provider' = 'openai',  'endpoint' = 'https://api.openai.com/v1/llm/v1/chat',  'api-key' = 'abcdefg',  'system-prompt' = 'translate to Chinese',  'model' = 'gpt-4o');-- Basic usageSELECT * FROM ML_PREDICT(  TABLE input_table,  MODEL my_model,  DESCRIPTOR(text));-- With configuration optionsSELECT * FROM ML_PREDICT(  TABLE input_table,  MODEL my_model,  DESCRIPTOR(text)  MAP['async', 'true', 'timeout', '100s']);-- Using named parametersSELECT * FROM ML_PREDICT(  INPUT => TABLE input_table,  MODEL => MODEL my_model,  ARGS => DESCRIPTOR(text),  CONFIG => MAP['async', 'true']);
```

更多信息：

* 模型推理[3]
* FLIP-437: Support ML Models in Flink SQL[4]

## Process Table Functions(PTFs)

Apache Flink 现已支持 Process Table Functions（PTFs），这是 Flink SQL 与 Table API 中最强大的函数类型。从概念上讲，PTF 是所有 UDF 的“超集”，能够将零个、一个或多张表映射为零个、一个或多行数据。借助 PTFs，你可以实现与内置算子一样功能丰富的自定义算子，并直接访问 Flink 的托管状态、事件时间、定时器以及表级变更日志。

PTFs 支持如下操作：

* 对表的每一行执行转换。
* 可以把一张表从逻辑上拆分成不同子集，并对每个子集分别转换。
* 缓存已见事件，供后续多次访问。
* 在未来某个时间点继续处理，实现等待、同步或超时机制。
* 利用复杂状态机或基于规则的条件逻辑对事件进行缓存与聚合。

这显著缩小了 Flink SQL 与 DataStream API 之间的差距，同时保留了 Flink SQL 生态的健壮性与易用性。
关于 PTFs 的语法与语义细节，详见：Process Table Functions[5]

示例如下：

```
// Declare a ProcessTableFunction for memorizing your customerspublic static class GreetingWithMemory extends ProcessTableFunction<String> {    public static class CountState {        public long counter = 0L;    }    public void eval(@StateHint CountState state, @ArgumentHint(SET_SEMANTIC_TABLE) Row input) {        state.counter++;        collect("Hello " + input.getFieldAs("name") + ", your " + state.counter + " time?");    }}TableEnvironment env = TableEnvironment.create(...);// Call the PTF in Table APIenv.fromValues("Bob", "Alice", "Bob")   .as("name")   .partitionBy($("name"))   .process(GreetingWithMemory.class)   .execute()   .print();// Call the PTF in SQLenv.executeSql("SELECT * FROM GreetingWithMemory(TABLE Names PARTITION BY name)")
```

更多信息

* FLIP-440: User-defined SQL operators / ProcessTableFunction (PTF)[6]

## Variant 类型

新增 `VARIANT`半结构化数据类型（如 JSON），支持存储任意嵌套结构（包括基本类型、`ARRAY`、`MAP`）并保留原始类型信息。相比 `ROW`、`STRUCTURED` 类型，`VARIANT` 在处理动态 Schema 数据时更灵活。配合 `PARSE_JSON` 函数及 Apache Paimon 等表格式，可实现湖仓中半结构化数据的高效分析。

```
CREATE TABLE t1 (  id INTEGER,  v STRING -- a json string) WITH (  'connector' = 'mysql-cdc',  ...);CREATE TABLE t2 (  id INTEGER,  v VARIANT) WITH (  'connector' = 'paimon'  ...);-- write to t2 with VARIANT typeINSERT INTO t2 SELECT id, PARSE_JSON(v) FROM t1;
```

更多信息：

* Variant Type[7]
* FLIP-521: Integrating Variant Type into Flink: Enabling Efficient Semi-Structured Data Processing[8]

## Structured 类型增强

支持在 `CREATE TABLE`语句中直接声明用户定义结构体类型（STRUCTURED），解决类型兼容性问题并提升易用性。

示例：

```
CREATE TABLE UserTable (    uid BIGINT,    user STRUCTURED<'User', name STRING, age INT>);-- Casts a row type into a structured typeINSERT INTO UserTable SELECT 1, CAST(('Alice', 30) AS STRUCTURED<'User', name STRING, age INT>);
```

更多信息：

* FLIP-520: Simplify StructuredType handling[9]
* Structured Type[10]

## Delta Join

引入一种新的 `DeltaJoin`算子，相比传统的双流 Join 方案，配合 Apache Fluss 等流存储，可以实现 Join 算子无状态化，解决了大状态导致的资源瓶颈、检查点缓慢和恢复延迟等问题。该特性已默认启用，同时需要依赖 Apache Fluss 存储相应版本支持，敬请关注 Support DeltaJoin on Flink 2.1[11]。

更多信息：

* FLIP-486: Introduce A New DeltaJoin[12]

## 级联双流 Join 优化

使用多个级联流式 Join 的 Flink 作业经常会因 State 过大而导致运行不稳定和性能下降。在这个版本，我们引入了一种新的 `MultiJoin`算子， 通过在单个算子内直接进行多流 Join，消除中间结果存储，每条流输入的记录最多存储一份，从而实现实现 "零中间状态" ，显著提升了资源利用率与作业稳定性。目前仅支持 INNER、LEFT JOIN ，并且需要通过参数 `table.optimizer.multi-join.enabled=true`启用。

基准测试：我们进行了一项基准测试，比较了 `MultiJoin`与双流 Join 的优势，更多详情请参见 MultiJoin Benchmark[13]
。

更多信息：

* FLIP-516: Multi-Way Join Operator[14]

#### 异步 Lookup Join 增强

异步 Lookup Join 在之前的版本中，即使用户将 `table.exec.async-lookup.output-mode` 设置为 `ALLOW_UNORDERED`，引擎在处理更新流时仍会强制回退到 `ORDERED` 以保证正确性。从 2.1 版本开始，引擎允许并行处理无关的更新记录，同时保证正确性，从而在处理更新流时实现更高的吞吐。

更多信息：

* FLIP-519: Introduce async lookup key ordered mode[15]

## Sink 节点复用

当单个作业中多个 `INSERT INTO` 语句更新目标表相同列时（下版本将支持不同列），优化器将自动合并 Sink 节点实现复用，该特性可以极大提升 Apache Paimon 等湖仓格式的部分更新场景使用体验。

更多信息：

* FLIP-506: Support Reuse Multiple Table Sinks in Planner[16]

## Compiled Plan 支持 Smile 格式

新增 Smile 二进制格式（兼容 JSON 格式）用于执行计划序列化，较 JSON 更节省内存。默认仍使用 JSON 格式，需通过显示调用 `CompiledPlan#asSmileBytes` 和 `PlanReference#fromSmileBytes` 方法启用。

更多信息：

* FLIP-508: Add support for Smile format for Compiled plans[17]
* Smile 格式规范[18]

# 运行时提升

## 异步 Sink 可插拔的批量处理

支持自定义批量写入策略，用户可根据业务需求灵活扩展异步 Sink 的批量写入逻辑。

更多信息：

* FLIP-509: Add pluggable Batching for Async Sink[19]

## 分片级水位线指标

在 Flink 2.1 版本，我们新增细粒度分片监控指标，涵盖水位线进度与状态统计：

* `currentWatermark`：该分片最新接收到的水位线值
* `activeTimeMsPerSecond`：该分片每秒处于数据处理状态的时间（毫秒）
* `pausedTimeMsPerSecond`：因水位线对齐该分片每秒的暂停时间（毫秒）
* `idleTimeMsPerSecond`：每秒空闲时长（毫秒）
* `accumulatedActiveTimeMs`：累计活跃时长（毫秒）
* `accumulatedPausedTimeMs`：累计暂停时长（毫秒）
* `accumulatedIdleTimeMs`：累计空闲时长（毫秒）

更多信息：

* FLIP-513: Split-level Watermark Metrics[20]

# 连接器提升

## Keyed State 连接器

在 Flink 2.1 中，我们为 Keyed State 引入了一个新的 SQL 连接器。该连接器允许用户使用 Flink SQL 直接查询Checkpoint 和 Savepoint 中的 Keyed State，从而使得更容易探查、调试和验证 Flink 作业状态，无需定制工具，该功能对于分析长期运行的作业和验证状态迁移尤其有用。

```
CREATE TABLE keyed_state (    k INT,    user_id STRING,    balance DOUBLE) WITH (    'connector' = 'savepoint',    'path' = 'file:///savepoint/path');-- 直接查询状态快照SELECT * FROM keyed_state;
```

更多信息：

* FLIP-496: SQL connector for keyed savepoint data[21]

# 其他重要更新

* PyFlink：新增支持 Python 3.12，移除 Python 3.8 支持。
* 依赖升级：flink-shaded 升级至 20.0 以支持 Smile 格式，Parquet 升级至 1.15.3 修复安全漏洞（CVE-2025-30065）。

# 升级说明

Apache Flink 社区努力确保升级过程尽可能平稳, 但是升级到 2.1 版本可能需要用户对现有应用程序做出一些调整。请参考 Release Notes[22]获取更多的升级时需要的改动与可能的问题列表细节。

# 贡献者列表

Apache Flink 社区感谢对此版本做出贡献的每一位贡献者：

主要链接：

[1] https://cwiki.apache.org/confluence/display/FLINK/FLIP-437%3A+Support+ML+Models+in+Flink+SQL

[2] https://cwiki.apache.org/confluence/display/FLINK/FLIP-507%253A+Add+Model+DDL+methods+in+TABLE+API

[3] https://nightlies.apache.org/flink/flink-docs-release-2.1/docs/dev/table/sql/queries/model-inference/

[4] https://cwiki.apache.org/confluence/display/FLINK/FLIP-437%3A+Support+ML+Models+in+Flink+SQL

[5] https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/functions/ptfs/

[6] https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=298781093

[7] https://nightlies.apache.org/flink/flink-docs-release-2.1/docs/dev/table/types/#other-data-types

[8] https://cwiki.apache.org/confluence/display/FLINK/FLIP-521%253A+Integrating+Variant+Type+into+Flink

[9] https://cwiki.apache.org/confluence/display/FLINK/FLIP-520%3A+Simplify+StructuredType+handling

[10] https://nightlies.apache.org/flink/flink-docs-release-2.1/docs/dev/table/types/#user-defined-data-types

[11] https://github.com/apache/fluss/issues/1143

[12] https://cwiki.apache.org/confluence/display/FLINK/FLIP-486%253A+Introduce+A+New+DeltaJoin

[13] https://nightlies.apache.org/flink/flink-docs-release-2.1/docs/dev/table/tuning/#multiple-regular-joins

[14] https://cwiki.apache.org/confluence/display/FLINK/FLIP-516%3A+Multi-Way+Join+Operator

[15] https://cwiki.apache.org/confluence/display/FLINK/FLIP-519%3A++Introduce+async+lookup+key+ordered+mode

[16] https://cwiki.apache.org/confluence/display/FLINK/FLIP-506%3A+Support+Reuse+Multiple+Table+Sinks+in+Planner

[17] https://cwiki.apache.org/confluence/display/FLINK/FLIP-508%3A+Add+support+for+Smile+format+for+Compiled+plans

[18] https://github.com/FasterXML/smile-format-specification

[19] https://cwiki.apache.org/confluence/display/FLINK/FLIP-509+Add+pluggable+Batching+for+Async+Sink

[20] https://cwiki.apache.org/confluence/display/FLINK/FLIP-513%3A+Split-level+Watermark+Metrics

[21] https://cwiki.apache.org/confluence/display/FLINK/FLIP-496%253A+SQL+connector+for+keyed+savepoint+data

[22] https://nightlies.apache.org/flink/flink-docs-master/release-notes/flink-2.1/

**活动推荐**

---

阿里云基于 Apache Flink 构建的企业级产品-实时计算 Flink 版现开启活动：

新用户复制下方链接或者扫描二维码即可0元免费试用 Flink + Paimon

了解活动详情：https://free.aliyun.com/?pipCode=sc

---

---

▼ 关注「**Apache Flink**」 ▼

回复 FFA 2024 获取大会资料

**点击「阅****读原文****」****跳转阿里云实时计算 Flink～**