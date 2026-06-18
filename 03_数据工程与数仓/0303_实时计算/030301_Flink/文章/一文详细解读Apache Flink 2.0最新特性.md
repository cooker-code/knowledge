---
title: 一文详细解读Apache Flink 2.0最新特性
author: ruby的数据漫谈
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247491488&idx=1&sn=e0458d5417acdb01e815a720fe7e06dd&chksm=913c4ad5c5dcf76ae383d2c7b30060d3d2fd05429a567fd79b076f5d4b490325b7ced702078e&mpshare=1&scene=24&srcid=1218lxvd4y7vbZnZafFc8ofL&sharer_shareinfo=02c34ed8bee12314979904302ef725ca&sharer_shareinfo_first=02c34ed8bee12314979904302ef725ca#rd
---

摘要：文章主要介绍了 Flink 2.0 - preview1 版本。其发布于 2024 年 10 月 23 日，是自 1.0 以来首个重大更新。此版本虽为预览版，不建议用于生产环境，但可让用户提前体验新功能并为社区提供反馈。

该版本有诸多重要特性，包括存算分离状态管理，可解决容器化环境下计算资源受限等问题；物化表功能增强，简化流和批作业数据处理流程且与主流湖格式集成；批作业的自适应执行，能动态优化逻辑和物理计划；以及流式湖仓新进展，与 Apache Paimon 集成带来多方面性能提升。

这些新特性对湖仓一体架构意义重大，它提高了资源利用率、数据处理效率与灵活性，为大数据处理提供更优方案。

* Flink 2.0-preview 版本概述‍‍‍‍‍‍‍‍‍
* 重要新特性解析‍‍‍‍‍‍‍
* 对湖仓一体架构发展的影响‍‍‍‍‍‍‍

01

—

Flink 2.0-preview 版本概述

‍

### （一）发布背景与意义

Apache Flink 社区已于 2024-10-23 发布了 Flink 2.0 版本，这是自 Flink 1.0 发布以来的首个重大更新，距离上一重大版本发布间隔了较长时间，堪称大数据处理领域的一个重要里程碑。

随着大数据技术的不断发展以及应用场景的日益多样化，用户对于数据处理框架的功能、性能、适应性等方面都有了更高的要求。Flink 2.0 的推出，旨在更好地满足当下以及未来大数据处理的各种复杂需求，比如应对海量数据的实时处理、满足不同业务场景下的状态管理挑战、适配云原生环境等。它所引入的多项激动人心的功能和改进，将助力企业和开发者更高效地挖掘数据价值，提升在实时数据分析、复杂事件处理、流数据 ETL 等诸多应用场景下的数据处理效率和质量，推动整个大数据处理领域朝着更先进、更高效的方向迈进。

### （二）版本性质说明

需要着重强调的是，**Flink 2.0-preview1 这个版本属于预览版，并非稳定版本。所以，建议用户不要将其应用于生产环境中。**

Apache Flink 社区之所以提供这个预览版，一方面是为了让广大用户能够提前尝试新功能，提前感受新版本带来的各种变化和优势，例如体验存算分离状态管理、物化表、批作业自适应执行等新特性在实际操作中的效果；另一方面也是希望借此收集用户反馈，以便社区能够进一步完善功能、优化性能、排查可能存在的问题等，为后续正式版本的发布奠定良好基础，确保正式版能够更加稳定、可靠地服务于广大用户的大数据处理需求。

02

—

重要新特性解析

**存算分离状态管理**‍‍‍

Flink 2.0 引入了基于远程存储的存算分离状态管理机制，这一架构解决了容器化环境下计算资源受限和LSM结构下的Compaction导致资源尖峰等问题，为云原生时代的大数据处理奠定了基础。

在说明远程存储的存算分离状态管理机制，我们来详细了解一下什么是RocksDB的LSM结构下的Compaction导致资源尖峰等问题。‍‍‍‍‍‍‍

**RocksDB 是一个基于 LSM（Log-Structured Merge-tree）树的键值存储系统，广泛用于分布式存储和数据库系统，包括 Apache Flink。**LSM 树是一种优化写入操作的数据结构，它通过将所有的写入（包括插入、更新和删除）首先记录到内存和/或磁盘上的日志文件中，然后定期将这些写入操作合并（Compaction）到磁盘上存储的多层结构中，以此来提高写入效率。

### **LSM 树的工作原理**

写入操作：所有的写入操作首先被记录到内存中的结构（例如，跳表）和/或磁盘上的WAL（Write-Ahead Logging）日志文件中。

层级合并（Compaction）：随着时间的推移，内存中的数据会逐渐被刷新到磁盘上的SSTable（Sorted String Table），形成多层结构。较低层级的SSTable会定期与新写入的数据合并，这个过程称为Compaction。

读取操作：读取操作需要检查所有相关层级的SSTable以及WAL日志，以确保读取到最新的数据。

### **Compaction 导致的计算资源尖峰**

Compaction 操作是 LSM 树为了维护数据一致性和优化读取性能而必须进行的后台过程。然而，这个操作可能会带来以下问题：

**I/O 负载：**Compaction 需要读取旧的SSTable，合并数据，然后写入新的SSTable，这个过程会产生大量的I/O操作。

**CPU 负载：**合并过程中需要对大量的键值对进行排序和去重，这可能会占用大量的CPU资源。

**内存使用：**在Compaction过程中，可能会有大量的数据被加载到内存中进行处理，这会增加内存的使用。

**当Compaction操作发生时，尤其是在数据量较大的系统中，这些负载可能会导致计算资源的尖峰，即在短时间内资源使用量急剧上升。这种尖峰可能会影响系统的其他操作，包括正常的用户请求，因为资源被Compaction操作占用，导致系统响应变慢，甚至可能影响系统的稳定性。**

### **对作业性能的影响**

在 Apache Flink 这样的流处理框架中，RocksDB 作为状态后端用于存储状态数据。如果RocksDB中的Compaction操作导致计算资源尖峰，那么：

**延迟增加：**Flink作业可能会因为等待Compaction操作完成而出现延迟。

**吞吐量下降：**由于资源被Compaction占用，Flink作业的吞吐量可能会下降。

**作业稳定性问题：**在极端情况下，Compaction操作可能会导致作业失败或者重启。

因此，Flink 2.0 引入存算分离状态管理和异步执行模型，旨在减少Compaction对计算资源的影响，提高作业的性能和稳定性。

那么我们如何理解Flink 2.0 引入存算分离状态管理和异步执行模型，我们使用一个具体的例子来说明，‍

### **场景：实时股票交易监控**

假设我们正在构建一个实时股票交易监控系统，需要处理大量的交易数据流，并根据交易数据实时更新股票的价格和交易量。

### **传统 Flink 部署（存算耦合）**

**1、状态存储：**在传统的 Flink 部署中，状态数据（如股票价格和交易量）通常存储在本地磁盘上的 RocksDB 中，这限制了状态数据的大小，因为每个计算节点的本地磁盘空间有限。

**2、性能瓶颈：**由于 RocksDB 的 LSM 结构，周期性的 Compaction 操作会导致计算资源的尖峰，影响作业的性能。

**3、扩展性问题：**当状态数据增长到一定规模时，需要手动扩展计算资源，这个过程不够灵活和快速。

### **Flink 2.0 存算分离和异步执行模型**

**1、存算分离状态管理：**在 Flink 2.0 中，我们将状态数据存储在远程存储系统中，如 S3 或 HDFS。这样，计算节点不再受限于本地磁盘大小，可以处理更大规模的状态数据。

**2、异步执行模型：**Flink 2.0 引入了异步执行模型，允许状态访问和检查点操作异步进行，从而提高了作业的性能和吞吐量。

**3、性能优化：**通过异步化执行，我们可以减少因状态访问和检查点操作导致的延迟，提高整体性能。

### **具体实现示例**

假设我们有一个简单的 Flink SQL 作业，用于实时更新股票价格：

sql复制

```
CREATE TABLE stock_trades (  
    stock_symbol STRING,  
    price DOUBLE,  
    volume INT  
) WITH (...);  
  
CREATE TABLE stock_prices (  
    stock_symbol STRING,  
    price DOUBLE,  
    volume INT,  
    WATERMARK FOR rowtime AS rowtime - INTERVAL '5' SECONDS  
) WITH (...);  
  
INSERT INTO stock_prices  
SELECT  
    stock_symbol,  
    SUM(price) AS price,  
    SUM(volume) AS volume  
FROM stock_trades  
GROUP BY stock_symbol, TUMBLE(rowtime, INTERVAL '1' MINUTE);
```

在这个作业中：

stock\_trades 表接收实时的交易数据流。

stock\_prices 表存储每个股票的最新价格和交易量，每分钟更新一次。

### **异步执行和存算分离的优势**

**状态异步访问：**在 Flink 2.0 中，我们可以异步地访问和更新状态数据，这意味着状态更新操作不会阻塞计算流程，提高了吞吐量。

**检查点异步化：**检查点操作也可以异步进行，减少了对计算资源的占用，提高了作业的稳定性。

**混合执行模型：**Flink 2.0 支持同步和异步算子在同一作业中共存，使得我们可以灵活地选择最适合特定操作的执行模型。

通过这个例子，我们可以看到 Flink 2.0 的存算分离状态管理和异步执行模型如何帮助我们构建更高效、更可扩展的实时大数据处理应用。

**物化表****‍‍‍‍‍‍‍‍**

Flink 2.0 增强了物化表功能，包括与主流湖格式的集成和生产就绪的调度器实现。物化表简化了流和批作业的数据处理流程，提供了统一的开发体验，提高了开发效率。

下面通过一个具体的例子来说明这些增强对流和批作业数据处理流程的影响。

### **例子：实时用户行为分析**

假设我们正在构建一个实时用户行为分析系统，需要从 Kafka 中读取用户行为数据流，并实时更新用户的行为统计信息到一个物化表中，以便快速查询。

#### **1. 创建物化表**

在 Flink 1.20 版本中，物化表功能以最简可行产品的形式引入。到了 Flink 2.0，我们增强了物化表的功能，包括与主流湖格式的集成。例如，我们可以创建一个物化表，将数据存储在文件系统中，并指定数据格式为 JSON：

sql复制

```
CREATE MATERIALIZED TABLE user_behavior_stats  
PARTITIONED BY (ds)  
WITH (  
  'format' = 'json',  
  'sink.rolling-policy.rollover-interval' = '10s',  
  'sink.rolling-policy.check-interval' = '10s'  
)  
FRESHNESS = INTERVAL '30' SECOND  
AS SELECT  
  user_id,  
  ds,  
  SUM(payment_amount_cents) AS total_amount,  
  COUNT(*) AS total_count  
FROM user_behaviors  
GROUP BY user_id, ds;
```

在这个例子中，user\_behavior\_stats 是一个物化表，它将根据用户 ID 和日期进行分区，并且每 30 秒刷新一次数据。这个表的数据被存储在文件系统中，格式为 JSON，并且支持滚动策略，每 10 秒创建一个新的文件。

而对于说物化表简化了流和批作业的数据处理流程是指用物化表支持在流处理和批处理中使用相同的 SQL 语法，意味着开发者可以利用相同的查询逻辑来处理流数据和批数据，而不需要为两种数据处理模式编写不同的代码。

### **具体例子**

假设我们有一个电子商务平台，需要分析用户的购买行为。我们可能会遇到以下两种场景：

实时分析：实时监控用户的购买行为，比如实时计算某个商品的购买次数。

历史分析：分析过去一段时间内的用户购买行为，比如计算上个月某个商品的总销售额。

在 Flink 中，我们可以为这两种场景编写相同的 SQL 查询：

sql复制

```
-- 实时分析：计算每个商品的实时购买次数  
SELECT product_id, COUNT(*) AS purchase_count  
FROM user_purchases_stream  
GROUP BY product_id;  
  
-- 历史分析：计算上个月每个商品的总销售额  
SELECT product_id, SUM(amount) AS total_sales  
FROM user_purchases_batch  
WHERE purchase_date BETWEEN DATE_SUB(CURRENT_DATE, INTERVAL 1 MONTH) AND CURRENT_DATE  
GROUP BY product_id;
```

在上述例子中，尽管我们处理的是流数据（user\_purchases\_stream）和批数据（user\_purchases\_batch），但我们使用的是相同的 SQL 语法结构。这意味着我们可以轻松地在流处理作业和批处理作业之间切换，而不需要改变我们的查询逻辑。

通过这种方式，Flink 的物化表功能使得流处理和批处理之间的界限变得模糊，为开发者提供了更大的灵活性和便利性。

Flink 2.0 中的物化表可以与主流湖格式进行集成，这意味着我们可以将物化表的数据存储在如 Delta Lake、Hudi 等湖仓格式中，利用它们提供的数据优化和治理能力。‍

通过这个例子，我们可以看到 Flink 2.0 中物化表功能的增强如何简化流和批作业的数据处理流程，并提供统一的开发体验，从而提高开发效率。同时，与湖格式的集成使得物化表可以更好地融入现有的数据架构中，为数据治理和优化提供了更多的可能性。

**批作业的自适应执行****‍‍‍‍**

Flink 2.0 将具备基于作业已完成的阶段所提供的信息，对逻辑计划和物理计划进行动态优化的能力，包括动态应用Broadcast Join以及对数据倾斜的Join进行优化，提升了批处理任务的效能。

在 Flink 2.0 中，动态优化能力的提升，尤其是在批处理任务中，可以通过一个具体的例子来说明。假设我们有一个订单表和用户表，我们需要执行一个 SQL 查询来找出所有订单金额超过一定数值的用户信息。这个查询涉及到一个 Join 操作，如果数据分布不均匀，可能会导致某些节点负载过高，即数据倾斜问题。

### **场景描述**

我们有两个表：

订单表（Orders）：包含字段 user\_id 和 amount。

用户表（Users）：包含字段 user\_id 和 name。

我们需要执行的 SQL 查询是：

```
sql

```
SELECT U.name, O.amount  
FROM Orders O  
JOIN Users U ON O.user_id = U.user_id  
WHERE O.amount > 1000;
```
```

### **数据倾斜问题**

在传统的批处理系统中，如果 Orders 表中的 user\_id 分布不均匀，某些 user\_id 可能会出现大量的订单记录，而其他 user\_id 的记录很少。这会导致在执行 Join 操作时，某些节点需要处理的数据量远大于其他节点，从而造成性能瓶颈。

### **Flink 2.0 的优化**

Flink 2.0 通过动态优化能力，可以识别这种数据倾斜问题，并动态调整执行计划来优化 Join 操作。以下是具体的优化步骤：

**1、动态识别数据倾斜：**Flink 2.0 可以动态监测到数据分布不均匀的情况，并识别出哪些 user\_id 是热点（即数据倾斜的键）。

**2、动态调整执行计划：**Flink 2.0 可以动态地调整执行计划，比如通过增加更多的资源来处理这些热点键，或者重新分配数据以平衡负载。

**3、Broadcast Join：**对于小表或者热点键的数据量不大的情况，Flink 2.0 可以动态地将小表或热点数据广播到所有节点，以减少数据倾斜的影响。

**4、自适应调整：**Flink 2.0 可以根据作业的执行情况自适应地调整策略，比如在执行过程中动态地调整 Broadcast 的阈值或者重新平衡数据。

### **具体例子**

假设 Users 表中的 user\_id 为 1 的记录特别多，Flink 2.0 可以识别这一点，并采取以下措施：

**识别热点：**在执行 Join 之前，Flink 2.0 通过监控发现 user\_id 为 1 的记录特别多。

**动态优化：**Flink 2.0 决定将 Users 表中 user\_id 为 1 的数据广播到所有节点，以减少对单个节点的压力。

**执行 Join：**通过 Broadcast Join，Flink 2.0 可以在所有节点上并行处理 Join 操作，避免了数据倾斜导致的性能问题。

通过这种方式，Flink 2.0 可以显著提升批处理任务的效能，特别是在处理大规模数据和复杂 Join 操作时。这种动态优化能力使得 Flink 2.0 能够更好地适应不同的数据分布和工作负载，提供更加稳定和高效的数据处理能力。

**流批一体计算范式上的改进****‍‍‍‍‍‍‍‍**‍

在 Flink 2.0 中，流批一体的统一 API 和执行模型意味着开发者可以使用相同的 SQL 语句或者编程 API 来处理流数据和批数据。这种统一性消除了流处理和批处理之间的界限，使得开发者可以更加灵活地处理不同类型的数据。

下面用两个例子来分别说明

### **场景1：实时和历史销售数据分析**

假设我们有一个电子商务平台，需要分析商品的销售数据。我们不仅需要实时监控商品的销售情况，还需要定期分析历史销售数据。

#### **实时销售分析（流处理）**

对于实时销售分析，我们可以使用 Flink 2.0 的流处理能力来监控 Kafka 中的商品销售事件流，并实时计算每个商品的销售额：

```
java

```
// 创建流数据源，从 Kafka 读取商品销售事件  
DataStream<SaleEvent> salesStream = env.addSource(new FlinkKafkaConsumer<>(  
    "sales-topic",  
    schema,  
    properties  
));  
  
// 使用 Flink SQL 客户端来处理流数据  
Table salesTable = tableEnv.fromDataStream(salesStream, "user_id", "product_id", "amount");  
  
// 实时计算每个商品的销售额  
Table result = tableEnv.sqlQuery(  
    "SELECT product_id, SUM(amount) AS total_sales " +  
    "FROM salesTable " +  
    "GROUP BY product_id"  
);  
  
// 将结果输出到下游系统  
tableEnv.toChangelogStream(result).addSink(new SomeSink());
```
```

#### **历史销售分析（批处理）**

对于历史销售分析，我们可以使用相同的 Flink SQL 语句来处理存储在文件系统或数据库中的历史销售数据：

```
java

```
// 创建批数据源，从文件系统读取历史销售数据  
Table historicalSalesTable = tableEnv.sqlQuery(  
    "CREATE TEMPORARY TABLE historical_sales " +  
    "WITH ('connector' = 'filesystem', 'path' = '/path/to/historical-sales-data') " +  
    "LIKE salesTable INCLUDING ALL"  
);  
  
// 使用相同的 SQL 语句来分析历史数据  
Table historicalResult = tableEnv.sqlQuery(  
    "SELECT product_id, SUM(amount) AS total_sales " +  
    "FROM historical_sales " +  
    "GROUP BY product_id"  
);  
  
// 将结果输出到下游系统  
tableEnv.toChangelogStream(historicalResult).addSink(new SomeSink());
```
```

在这个例子中，我们可以看到：

**统一的 API：**无论是流数据还是批数据，我们都使用了相同的 Flink SQL 语句来定义查询逻辑。

**统一的执行模型：**Flink 2.0 的执行引擎能够自动根据数据源的类型（流或批）来优化执行计划，而不需要开发者手动区分处理逻辑。

### **场景2：实时和历史销售数据分析**

假设我们有一个电子商务平台，需要分析商品的销售数据。我们不仅需要实时监控商品的销售情况，还需要定期分析历史销售数据。

#### **Flink 2.0 中的使用方式**

在 Flink 2.0 中，我们可以定义一个统一的表结构，然后通过 SQL 查询来处理流数据和批数据。

定义表结构：

```
sql

```
CREATE TABLE sales_data (  
    product_id INT,  
    sale_amount DECIMAL(10, 2),  
    sale_time TIMESTAMP(3)  
) WITH (  
    'connector' = 'kafka', -- 对于流数据  
    'topic' = 'sales-topic',  
    'properties.bootstrap.servers' = 'localhost:9092',  
    'format' = 'json',  
    'scan.startup.mode' = 'latest-offset' -- 从最新的 offset 开始消费  
);
```
```

**实时销售分析（流处理）：**

```
sql

```
SELECT product_id, SUM(sale_amount) AS total_sales  
FROM sales_data  
GROUP BY product_id;
```
```

**历史销售分析（批处理）：**

```
sql复制

```
SELECT product_id, SUM(sale_amount) AS total_sales  
FROM sales_data  
GROUP BY product_id;
```
```

在 Flink 2.0 中，我们可以使用相同的 SQL 语句来处理流数据和批数据，无需改变查询逻辑。

**流式湖仓**

Flink 2.0 与 Apache Paimon 的集成，将湖仓范式中统一的数据存储、开放格式和成本效益扩展到了实时领域。这种集成使得 Flink 能够利用 Paimon 的优势进行 SQL 执行计划优化、提升 Lookup-Join 的性能、支持 Flink 物化表、以及对自适应批处理和推测执行的支持。这里特别说明一下 paimon什么？**‍‍**

Apache Paimon 是一个开源的流式数据湖平台，它结合了 Flink 和 Spark 的实时计算能力以及 Lakehouse 架构的优势，为用户提供了高速数据摄取、变更日志跟踪和高效的实时分析能力。以下是 Paimon 的一些核心特性和概念：

**1、实时更新：**

Paimon 支持高效的实时更新，基于 LSM Tree 结构，整个流程基于 Append + Compaction 模型。这种结构在业界被许多数据库系统采纳，能够保障写入和更新的吞吐量。

**2、流批一体：**

Paimon 是一个流批一体的存储，支持流读、批读、Time Travel 的方式查询、维表点查、全增量一体消费，也支持流式写入和批式导入。这意味着在流批场景中都有适用的场景。

**3、OLAP 友好：**

Paimon 支持列式存储，默认基于 ORC 的存储格式，并且有丰富的 Statistics 来帮助 Plan 阶段做好 Data Skipping 之类的 Plan 优化。此外，Paimon 支持各种 DataLayout 优化，比如 Zorder 和 Hilbert 排序和数据重分布的优化，加速查询过程。

**4、开放的数据格式：**

Paimon 以湖存储的方式基于分布式文件系统管理元数据，并采用开放的 ORC、Parquet、Avro 文件格式，支持各大主流计算引擎，包括 Flink、Spark、Hive、Trino、Presto。未来会对接更多引擎，包括 Doris 和 Starrocks。

**5、大规模实时更新：**

Paimon 得益于 LSM 数据结构的追加写能力，在大规模的更新数据输入的场景中提供了出色的性能。‍

**6、数据湖功能：**

Paimon 支持存储 Petabyte 规模的数据集和存储大量分区，支持 ACID 事务、时间旅行和模式演化。

**7、统一存储：**

对于像 Apache Flink 这样的流媒体引擎，Paimon 提供了统一的存储解决方案，使得使用方式与传统数据库没有什么不同。‍

Paimon 的设计目标是提供一个能够处理大规模数据更新和实时查询的统一数据湖格式，它通过结合湖存储和 LSM 技术，为数据湖带来了实时流更新以及完整的流处理能力。这使得 Paimon 成为构建实时 Lakehouse 架构的关键技术，特别是在需要实时数据更新和处理的场景中。

以上是Flink 2.0最新的特性，从当前特性来看还是非常吸引人的，不管是从使用的学习成本上，还是从性能，以及稳定性都有很大的改观，以往flink作为流批一体的引擎始终用不起来的可能得到最终的解决，而Flink 2.0的特性对于批处理也有较大的改观，那么对于使用spark 引擎做批处理的小伙伴可能会有部分开始使用flink进行批处理了，这个也是对spark 引擎的一个极大的挑战。未来对Flink 2.0的正式版本的产出特别期待。

03

—

对湖仓一体架构的影响‍‍‍

Flink 2.0 的这些新特性对湖仓一体架构有着深远的影响。存算分离状态管理使得 Flink 能够更好地适应云原生环境，提高了资源利用率和弹性。物化表和批作业的自适应执行则进一步提升了数据处理的效率和灵活性。流式湖仓架构的集成则将湖仓范式的优势扩展到了实时领域，为大数据处理提供了更加统一和高效的解决方案。

综上所述，Flink 2.0-preview1 的发布不仅为 Flink 社区带来了激动人心的新功能，也为整个大数据处理领域，尤其是湖仓一体架构的发展，指明了新的方向。随着 Flink 2.0 的不断完善和正式发布，我们有理由期待它将在实时数据处理和湖仓一体架构中扮演更加重要的角色。

**欢迎加入免费【数据&AIGC交流群】社群，长按以下二维码加入专业微信群****，商务合作加微信备注商务合作，**AIGC应用开发交流入群备注AIGC应用****

###### Ruby数据漫谈

知识星球介绍

在这个数据驱动的时代，您是否渴望成为大数据技术的领航者？是否希望掌握AIGC的前沿应用？是否在寻找数字化转型的秘籍？【数据星河】知识星球，是您理想的知识家园！

**往期数据平台历史热门文章：**

[基于DataOps的数据开发治理：实现数据流程的自动化和规范化](https://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247484720&idx=1&sn=524e990f0e9e52f3fbbd79dc68aab48d&scene=21#wechat_redirect)

[数据平台：湖仓一体、流批一体、存算分离的核心问题及原因解析](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247484725&idx=1&sn=5a0189a88752dfb11ebee050c1f4dbdf&chksm=90313f93a746b685e787cd7d69a287ccbab432fb999fe24c6e1e50667bf3e15655ac5f1e0bc8&scene=21#wechat_redirect)

[数据治理体系该怎么建设？](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247484627&idx=1&sn=676a191fbf1093128426dd6d548fcbc1&chksm=90313e75a746b7632d91776dceb594163089c3863e82e1e03c9b62c297db36377ecaf3bd33a9&scene=21#wechat_redirect)

[实时数仓&流批一体技术发展趋势](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247484603&idx=1&sn=0738f7863b8660681de58901cd873c66&chksm=90313e1da746b70b5780c042f865c2c576e525862e625705f9de75aa8524f5464d9f7b05d616&scene=21#wechat_redirect)

[数据仓库、数据中台、大数据平台的关系？](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247484552&idx=1&sn=952e18c758a8e17080c6661bc4fe8a2c&chksm=90313e2ea746b738136acdad2c74ce7f4df4bcd8e0d9169b1c2410287a64fe896902e26f0762&scene=21#wechat_redirect)

[数字化转型如何促进业务的发展](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247484498&idx=1&sn=28df3618851b3a0976c25bf9f24bd5b8&chksm=90313ef4a746b7e2bb64062a295429357256d8c2a15c9cd01a80c127b38b27c036427188101b&scene=21#wechat_redirect)

[数据中台中的核心概念解析](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247484480&idx=1&sn=0cf8ad6572673ed809e1880e45af6031&chksm=90313ee6a746b7f02ef56ad274d14e2e02367f965b2c387f6502af2c30c6b5e076937017bf63&scene=21#wechat_redirect)

[数据治理中的数据标准的作用？](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247484399&idx=1&sn=00bb5488c69e60c5adb28dcd34966614&chksm=90313949a746b05f4a86d50c94775b07a142f9cbaa5c6c88d2f277881d0a4a0a8745b4b8393a&scene=21#wechat_redirect)

[全面数字化转型：打造全新营销模式](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247484449&idx=1&sn=fcd81245d328f205317702603eb35ba3&chksm=90313e87a746b791f7c216112cd852484cf7054c0e106ed936038572bc9b6364fafbff789934&scene=21#wechat_redirect)

[一图展示数据中台的数据流图](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247487790&idx=1&sn=3f23cdeb226d7868ab71715644ec92cd&chksm=90312b88a746a29ea3b5fc54ae15fc3204ebffad0a00f52618519d27c6dbff910811822e43bd&scene=21#wechat_redirect)

[揭秘数据治理系统的数据流程图](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247487559&idx=1&sn=0233f8a79f1724413334f62173f34e3f&chksm=90312ae1a746a3f724f00fc7905735d01223320250b1846fb1bda6301f5e7ed3d12288a58cf5&scene=21#wechat_redirect)

**往期AIGC历史热门文章：**

**[AIGC系列之一-一文理解什么是Embedding嵌入技术](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247487576&idx=1&sn=5719791e1ad3e54467bb029712d52030&chksm=90312afea746a3e8fd8b32226e81e2dd66cb7bd54158b63f70489073eae9a28a2da55b53b384&scene=21#wechat_redirect)**

**[十大AIGC文生视频产品介绍](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247487479&idx=1&sn=d7d309eace6030973086f2ff465bd95c&chksm=90313551a746bc474140ae957d58c6cd92c405fcc53ef49badcfc22d25c2f38a3bb2209be839&scene=21#wechat_redirect)**

[九大最热门的开源AI Agent框架](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247487293&idx=1&sn=4d02694f321f66168939d1f113009f90&chksm=9031359ba746bc8d20d681cd7b79919aefa94be7270da4c082ce2f40fd847ed178efb12282bb&scene=21#wechat_redirect)

[AutoGen零代码构建⾃⼰的智能助理](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247487204&idx=1&sn=61f217fcd1868b3577d809118baccf0b&chksm=90313442a746bd54fd536520d93c0fe28b118f5e39b55e71d964c16d040cf2bd5c0ff685a1fe&scene=21#wechat_redirect)

**往期数据资产入表历史热门文章：**

**[资产入表](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247489577&idx=1&sn=d6c1b2dd20338e118e1b47f243421272&chksm=9031228fa746ab996b54497aec052ed3bcf276b7775fa5ccc06e779f47aaaf5105fc851887af&scene=21#wechat_redirect)**

**[数据资产入表流程](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247489009&idx=1&sn=17db18c61097d13c1027e7628bab4701&chksm=90312f57a746a64120bcc42a8bdf455890264dd8afdde2a0119537b3e90460de1cb63fd7b2f0&scene=21#wechat_redirect)**

[数据资产管理及入表的关键步骤](http://mp.weixin.qq.com/s?__biz=MzA4ODAyNzA4MQ==&mid=2247489970&idx=1&sn=7d2a3efa30b7f414c0645a807449359f&chksm=90312314a746aa0224afdcc339557aef7457b096426f41aec893466079bd388d116ae41a3e48&scene=21#wechat_redirect)