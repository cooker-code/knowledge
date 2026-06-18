> 已吸收至：[[03_数据工程与数仓/0305_湖仓表格式/030503_Iceberg/030503_核心知识点/Iceberg v3 删除向量与行级血缘边界|Iceberg v3 删除向量与行级血缘边界]]
---
title: 为什么 Iceberg v3 是数据湖仓的"iPhone 时刻"？
author: 过往记忆大数据
date:
url: https://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=2650742287&idx=1&sn=b37cd515af8f34754ac1674a6601bda8&chksm=8925ebc931e64d1073c40642dfa4352b302fb5919831bf4d296e9e1e68e87ef995cf4691e8e1&mpshare=1&scene=24&srcid=0413MrfSMEtNH8n8sWdcY3JX&sharer_shareinfo=4e0e486ff1e1be47d07aa9699a9b6bc4&sharer_shareinfo_first=4e0e486ff1e1be47d07aa9699a9b6bc4#rd
---

2026年3月4日，snowflake 在其官方博客发表了名为《Announcing Apache Iceberg v3 Support in Public Preview on Snowflake》，一个月后的4月9日，Databricks 也在其官方博客发表了名为《The Next Era of the Open Lakehouse: Apache Iceberg™ v3 in Public Preview on Databricks》的文章。至此，四大云厂商（Databricks、Snowflake、Google、Dremio）湖仓引擎全力押注到 Iceberg v3，**Iceberg 不再是"候选标准之一"，它就是标准本身**。

Apache Iceberg v3 的规范在 2025 年 6 月正式通过，**它不是一次常规的版本升级，而是数据湖仓架构的一次质变。** 其补齐了湖仓一体最后的几块关键拼图，让"用开放格式替代封闭数仓"从愿景变成了现实可行的工程路径。这就是为什么越来越多的云厂商湖仓架构押注到 Iceberg。今天，我们就来详细介绍一下 Iceberg v3 比较重要的新功能。

## 一、删除向量（Deletion Vectors）：行级更新终于不再是性能灾难

### 以前有什么问题？

Iceberg v2 引入了行级删除的能力（通过 positional delete files），这在当时是一个巨大的进步。但它有一个严重的性能瓶颈：**每次删除都会生成独立的小文件，查询时引擎需要将这些小文件与原始数据文件逐一匹配、合并、过滤。**

当你的表有频繁的行级更新操作（比如用户修改订单状态、CDC 实时同步上游数据库变更），这些删除文件会像雪花一样越积越多。一张被频繁更新的大表，可能积累成千上万个小的删除文件。每次查询都要先"扫雪"，然后才能真正读数据。

这样导致的问题就是 **写入越频繁的表，读取越慢。** 恰恰是最需要实时分析的场景，性能最差。

### v3 怎么解的？

Iceberg v3 引入了**二进制删除向量（Binary Deletion Vectors）**，这是一个根本性的架构变革。

核心思路：给每个数据文件附加一个**位图（Bitmap）**，位图中的每一位对应数据文件中的一行。`0` 表示有效行，`1` 表示已删除行。底层数据结构采用 **Roaring Bitmap**——一种专门为稀疏整数集设计的高效压缩算法。

这意味着什么？

* **不再有成千上万的小删除文件。** 所有删除信息被压缩进一个紧凑的位图，附着在对应的数据文件旁边（通常存储在 `.puffin` 文件中）。
* **查询时不再需要"扫雪"。** 引擎读取数据文件时，同时读取对应的删除向量，做一次位运算就能跳过所有被删除的行。开销极小，几乎可以忽略。
* **写入放大大幅降低。** 修改几行数据不再需要重写整个数据文件，只需更新位图即可。

Databricks 在其博客中给出了一个惊人的数据：**删除向量的数据处理速度比传统的 Copy-on-Write 方式快 10 倍。**

更重要的是，v3 还强制要求引擎在写入时必须为每个数据文件维护**单一的删除向量**（而不是允许多个删除文件散落），从根本上杜绝了"小文件地狱"的回归。

## 二、行级血缘（Row Lineage）：CDC 从"手工活"变成"表的固有属性"

### 以前有什么问题？

在 Iceberg 之前格式的表中，"这张表从上次查询以来，哪些行发生了变化？"——这个看似简单的问题，回答起来非常痛苦。

你需要自己维护增量标记：在 ETL 管道里加时间戳字段、维护 watermark、做全表比对或者依赖外部 CDC 工具（Debezium、Maxwell）来追踪变更。这些方案要么重、要么脆、要么贵。

下游要做增量刷新？**对不起，你得自己想办法搞清楚哪些数据变了。** 这是一个困扰了整个数据工程界多年的基础设施级痛点。

### v3 怎么解的？

Iceberg v3 在规范层面引入了**行级血缘**。每一行数据现在天然携带两个元数据字段：

* **行 ID（Row ID）**：全局唯一的行标识符，伴随行的整个生命周期。
* **序列号（Sequence Number）**：记录该行最后一次被添加或修改的提交版本。

这两个字段不是你手动加的业务字段，而是 **Iceberg 表格式规范本身的一部分**。所有写入引擎在写入数据时必须维护它们。

这意味着什么？

**"哪些行变了"这个问题，现在变成了一个简单的元数据查询。** 你只需要比较两次快照的序列号差异，就能精确地找到所有新增、修改和删除的行——不需要全表扫描、不需要外部 CDC 工具、不需要手工维护水位线。

Databricks 在博客中直接说了一句杀伤力极强的话：

> Together, row lineage and deletion vectors make CDC a native property of the table itself.

有了行级血缘，增量刷新物化视图变得异常简单。你知道上次刷新以来哪些行变了，只需要重新计算涉及这些行的聚合结果，而不需要重新跑整张源表。对于数据量在 PB 级别的企业来说，这个差距可能是"跑 30 分钟"和"跑 10 秒"的区别。

在 AI 工程领域，行级血缘同样意义重大——你可以精确追踪某条训练数据的来源和变更历史，这对于数据质量管理、模型可审计性和合规要求都是关键能力。

## 三、VARIANT 类型：半结构化数据终于不再是二等公民

### 以前有什么问题？

现实世界的数据，有一半以上是半结构化的。API 响应、IoT 传感器数据、用户行为日志、第三方 webhook——这些数据的特点是：**字段不固定、层级不确定、模式随时可能变。**

在 Iceberg v2 中处理这类数据，你只有两个选择：

**选择一：把 JSON 展平成固定列。** 结果就是一张几百列的超宽表，大量 NULL 值，每次上游加一个字段就得跑一次 schema evolution。维护成本高到让人崩溃。

**选择二：把 JSON 作为字符串存储。** 列式存储的一切性能优势全部丧失。谓词下推？不存在的。读取时需要全量解析字符串。查询性能惨不忍睹。

无论哪种选择，半结构化数据在数据湖里始终是"二等公民"。

### v3 怎么解的？

Iceberg v3 引入了原生的 **VARIANT 数据类型**。

VARIANT 不是简单地把 JSON 存成 Binary——它使用了一种**高效的二进制编码格式**，能够在保持模式灵活性的同时，支持列式存储引擎的核心优化特性：

* **谓词下推**：查询引擎可以在不解析整个 VARIANT 的情况下，直接对内部字段做过滤。比如 `WHERE payload.event_type = 'purchase'` 可以在文件扫描阶段就完成过滤，而不需要把整个 JSON 读进内存再解析。
* **切碎优化（Shredding）**：对于高频访问的内部字段，引擎可以将其"切碎"为独立的列式存储，实现接近原生列的读取性能。不常访问的字段仍以紧凑的二进制格式存储。
* **零 Schema 迁移**：新字段出现时，不需要任何 ALTER TABLE 操作。直接写入，直接查询。

```
-- 直接在 Iceberg v3 表中存储和查询半结构化数据
CREATETABLEevents (
    event_id BIGINT,
    ts TIMESTAMP_NS,
    payload VARIANT
) USING iceberg
TBLPROPERTIES ('format-version' = '3');

-- 用标准 SQL 查询嵌套字段
SELECT payload:user_id, payload:action, payload:metadata:device_type
FROMevents
WHERE payload:action = 'purchase'
AND ts > current_timestamp() - INTERVAL1HOUR;
```

有了 VARIANT，你的策略变成了：**原始数据直接入湖，查询时再提取你需要的字段。**

对于 AI 工程师来说，这意味着你可以用 SQL 直接查询 LLM 的原始推理日志、Agent 的工具调用记录、模型 A/B 测试的完整 trace——不需要先把它们"规整"成关系型表。**数据分析的速度被提升了一个数量级，因为你消灭了从"数据产生"到"数据可查"之间的等待时间。**

## 四、默认列值：一秒完成十亿行的 Schema 变更

这个功能看起来不起眼，但对日常数据工程的体验改善是巨大的。

以前在 Iceberg 中给一张已有十亿行数据的表加一列并指定默认值，你需要做回填——重写所有数据文件，在每一行中写入新列的值。对于一张 PB 级的表，这可能意味着几个小时的计算资源消耗和服务中断风险。

Iceberg v3 的**默认列值**功能让这个操作变成了纯元数据操作：

```
ALTER TABLE orders ADD COLUMN priority STRING DEFAULT 'normal';
```

执行时间只需要亚秒级。 因为它只修改了表的元数据文件，没有触碰任何数据文件。当查询引擎读取没有 `priority` 列的旧数据文件时，会自动从表模式中查找默认值并在结果中即时填充。

## 五、纳秒级时间戳：精度的最后一公里

v3 新增了 `timestamp_ns` 和 `timestamptz_ns` 数据类型，将时间精度从微秒提升到**纳秒**。

这听起来像是一个细节优化，但对于特定场景意义重大：

* **高频交易**：金融市场的订单簿事件间隔可能在纳秒级，微秒精度会导致事件顺序丢失。
* **IoT 传感器**：工业物联网中的振动传感器、电力监控设备的采样率可能达到百万次/秒。
* **分布式系统调试**：在微服务架构中，纳秒级时间戳是重建因果关系和排查竞态条件的基础。

以前这些场景不得不用 BIGINT 存储纳秒时间戳，丧失了时间类型的语义信息和查询便利性。现在，时间就是时间，精度到纳秒。

## 六、行业变化

Iceberg v3 的技术细节固然重要，但真正让我确信这是一次"质变"的，是**整个行业生态的响应速度**。

**Databricks** 不仅全力支持 Iceberg v3，还在博客中公开宣布：Iceberg v3 的删除向量、行级血缘、VARIANT 类型与 Delta Lake 实现了**同一数据层的互操作**。通过 UniForm 技术，客户只需写入一次 Delta Lake，即可以 Iceberg 格式从 Snowflake、BigQuery、Redshift 等引擎读取。这等于 Databricks 在说："你用 Iceberg 还是 Delta，已经不重要了。"

**Snowflake** 于 2026 年 3 月宣布对 Iceberg v3 进入公开预览支持，覆盖托管表和外部 Iceberg 目录，并且同步推进开源数据目录 Apache Polaris 的治理联邦能力。

**Google Cloud** 的 BigQuery Managed Iceberg（BigLake）全面支持 v3 新特性。Google 工程师还直接参与了 v3 规范的核心贡献。

**Dremio** 在 v3 规范通过当天就发布了详细的技术解读。

当 Databricks、Snowflake、Google、Dremio 四大湖仓引擎同时全力押注一个开放规范时，**格式之战基本上可以宣告结束了。** Iceberg 不再是"候选标准之一"，它就是标准本身。

## 七、结论

Apache Iceberg v3 不是一次"又加了几个功能"的例行升级。**它是数据湖仓架构从"可行"到"成熟"的分水岭。**

删除向量让行级更新快了 10 倍，行级血缘让 CDC 变成表的原生能力，VARIANT 类型让半结构化数据不再是二等公民，默认列值让 Schema 变更从小时级变成秒级，地理空间类型和纳秒时间戳填补了最后的能力空白。

更重要的是，它获得了整个行业生态的集体认可——Databricks、Snowflake、Google、Dremio 同时全力押注，格式之战实质结束。数据湖和数据仓库之间的柏林墙，在 Iceberg v3 这里，正式倒塌了。

*参考资料：*

* *Google Open Source Blog:  What's new in Apache Iceberg v3? (2025.08)*
* *Databricks Blog: Apache Iceberg™ v3: Moving the Ecosystem Towards Unification (2025.06)*
* *Databricks Blog: The Next Era of the Open Lakehouse: Apache Iceberg™ v3 in Public Preview on Databricks (2026.04)*
* *Dremio Blog: What's New in Apache Iceberg Format Version 3? (2025.04)*
* *Snowflake: Announcing Apache Iceberg v3 Support in Public Preview on Snowflake (2026.03)*