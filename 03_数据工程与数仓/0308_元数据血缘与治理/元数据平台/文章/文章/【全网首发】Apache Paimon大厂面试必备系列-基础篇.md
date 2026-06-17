---
title: 【全网首发】Apache Paimon大厂面试必备系列-基础篇
author: 大数据技术与架构
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247523826&idx=1&sn=c8ff85df70c01629f08960ffb845d98d&chksm=fcd6a7a77b4962b44c3ddc44dac6b496dd512523203fae57a9db74ff5b429fa2f4cf1f353dbb&mpshare=1&scene=24&srcid=0102cbqBJPgJQ2Ritb3sd1eV&sharer_shareinfo=57bced91e3952de1694ac8eca41d4658&sharer_shareinfo_first=57bced91e3952de1694ac8eca41d4658#rd
---

这是一个系列文章，包含基础篇、原理篇、进阶篇、实践篇等至少4+个系列。欢迎收藏、追更。

本系列的文章非常「功利」，完全着眼于面试，当然读者完全可以把它当成学习完Paimon后的自我检验也是可以的。文章较长，推荐收藏。

本篇文章是基础篇。基础篇是入门Paimon必须要掌握的部分。本系列内容在知识星球同步更新，[**知识星球**](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247518343&idx=1&sn=eaf44bee8c4d90aedd508201475b57a9&chksm=fd3ece12ca494704cdf02300bbfe39c1d4245ac30f41aba3065efe66eb9aa453a23e0b6dae6e&scene=21#wechat_redirect)内同步答疑，冲刺中大公司、高阶岗位的同学随时在知识星球提问。

本文部分参考了Paimon官网、社区、网络分享的内容，内容较长难免有笔误，大家可自行对比官网纠错。

### Paimon作为湖表有哪些核心能力？

Apache Paimon是一种流批统一的数据湖存储格式，结合Flink及Spark构建流批处理的实时湖仓一体架构。Paimon创新地将湖格式与LSM技术结合起来，给数据湖带来了实时流更新以及完整的流处理能力：

**实时入湖**：Paimon支持包括MySQL在内的多种数据库系统的实时变化同步写入，在千万级数据规模下也能保持高效率与低延迟。

**湖上流批一体处理**：Paimon结合Flink提供了完整的流处理能力，结合Spark提供了完整的批处理能力。基于统一的数据湖存储，提供数据口径一致的流批一体处理，提高易用性并降低成本。

**全面生态集成拓展**：Paimon与众多计算组件紧密集成，Flink、Spark等都与Paimon有着较为完善的集成度，统一存储，计算无边界。

**高效查询能力**：Paimon在流批技术处理的基础上，提出Deletion Vectors和索引来增强查询性能，在分钟级时效性基础上满足流、批、OLAP等场景的全方位支持。

### Paimon有哪些表类型可以选择？

上图描述了大致所有表模式的配置及能力。从类型上来分，Paimon分为主键表和非主键表。

对于**主键表**：定义了主键的表可以进行插入、更新和删除操作。主键由一组列组成，这些列对每条记录具有唯一性。Paimon通过对每个桶（bucket）内的主键进行排序来确保数据有序，从而在应用主键过滤条件时实现高性能。

主键表强制绑定 Bucket (桶) 概念，数据根据主键会被分配不同的 Bucket 中，在同一时间，同一个主键只会落到同一个 Bucket 中，每个 bucket 里面都包含一个单独的 LSM Tree 及其变更日志文件。Bucket 是最小的读写存储单元，因此 Bucket 的数量限制了最大的处理并行度。不过，这个数字不应该太大，因为这会导致大量小文件和低读取性能。一般情况下，建议每个bucket中的数据大小约为 200MB-1GB。

对于**非主键表**：如果在创建Paimon表时没有指定主键（Primary Key），则该表就是Paimon Append Only表。您只能以流式方式将完整记录插入到表中，适合不需要流式更新的场景（例如日志数据同步）。

Append 表的 Compaction 只是用来合并小文件。目前 Append 表有两种模式：Scalable 表 与 Queue 表。整体上我们推荐使用 Scalable 表，它更简单易懂。

在Paimon的0.9版本中，Paimon强制规定，如果你设置了bucket那么必须设置bucket-key。

### Paimon中的LSM树是什么？

‌LSM树（Log-Structured Merge Tree）是一种专为海量数据读写而设计的数据结构，特别适用于写密集型应用程序，如数据库系统、日志记录系统和键值存储系统。‌‌

LSM树的核心思想是将数据分散存储在多个结构中，这些结构包括内存中的临时存储区（如Memtable）和磁盘上的持久存储区（如SSTable）。数据首先写入内存中的Memtable，当达到一定阈值后，Memtable会被刷新到磁盘上形成SSTable。读取操作从最新的层次开始，逐级向上查找。

关于LSM树你可以参考：[【LSM树】](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247513135&idx=1&sn=fb2263c7d22e7d488f3e015916a1a93e&scene=21#wechat_redirect)，或者可以从网上搜一搜关于LSM的详细解释。

Paimon 采用 LSM Tree 作为文件存储的数据结构。LSM Tree 将文件组织为几个排序的 Sort Runs。Sort Runs 由一个或多个数据文件组成，数据文件中的记录按其主键进行排序。在 Sort Run 中，数据文件的主键范围从不重叠。

Paimon通过主键排序、分桶结构和LSM树结构实现高效的数据存储和查询。在每个桶内，主键进行排序以确保数据有序；通过分桶管理数据，提升查询和写入的效率；同时，利用LSM树有效组织和存储新数据，并通过有序运行管理确保查询性能。

### 解释一下Paimon的Bucket？

#### Bucket（桶）

* 概念：无分区表或分区表中的分区被进一步细分为多个桶，以提供更高效的查询结构。
* 结构：每个桶目录包含一个 LSM 树及其变更日志文件。
* 分桶方式：桶的范围由记录中一个或多个列的哈希值决定。用户可以通过 bucket-key 选项指定分桶列。如果未指定，将使用主键（如定义）或完整记录作为分桶键。
* 处理并行度：桶是读写的最小存储单元，桶的数量限制了最大处理并行度。桶的数量不宜过大，否则会导致许多小文件并降低读性能。一般推荐每个桶的数据大小在 200MB 到 1GB 之间。

##### 固定桶（Fixed Bucket）

* 机制：配置一个大于 0 的桶数，使用公式 Math.abs(key\_hashcode % numBuckets) 计算记录所归属的桶。
* 可扩展性：只能通过离线过程重新调整桶数。桶数过多会导致小文件过多，过少则会导致写入性能下降。

##### 动态桶（Dynamic Bucket）

* 默认模式：主键表的默认模式，或通过配置 bucket = '-1' 启用。
* 分配策略：1. 初始数据分配到旧桶，新数据分配到新桶，依据数据到达顺序。2. Paimon 通过索引来确定键-桶的映射关系，并自动扩展桶的数量。
* 配置选项：

```
dynamic-bucket.target-row-num：控制单个桶的目标行数  
dynamic-bucket.initial-buckets：控制初始桶的数量
```

* 限制：动态桶支持单写作业。不要启动多个作业写入同一分区，否则会导致数据重复问题。

### 解释一下动态分桶？以及跨分区更新的问题

对于**正常动态桶模式（Normal Dynamic Bucket Mode）**，更新不跨分区（无分区或主键包含所有分区字段）时，使用 HASH 索引维护键-桶映射。需要更多内存，约1亿条目占用1GB内存，但不再活动的分区不占用内存。如果出现跨分区更新，一般无性能损失，但消耗更多内存。适用于更新频率低的表，可显著提升性能。

对于**跨分区 Upsert 动态桶模式（Cross Partitions Upsert Dynamic Bucket Mode）**

需要跨分区 Upsert 操作（主键不包含所有分区字段）。一般需要配置 Index TTL（cross-partition-upsert.index-ttl）以减少索引和初始化时间，但可能导致数据重复。

跨分区 Upsert 动态桶模式有三种模式可以选择：

```
Deduplicate：删除旧分区的数据，插入新分区的数据。  
PartialUpdate & Aggregation：在旧分区插入新数据。  
FirstRow：忽略新数据，如果存在旧值。
```

Paimon会维护维护键-分区-桶的映射，使用本地磁盘，通过读取所有现有键初始化索引。性能上对于大量数据的表，性能会显著下降，且初始化时间较长。

### 生产环境分区键的最佳选择有哪些？

1. **创建时间（推荐）**：通常是不可变的，可以作为分区字段并添加到主键中。
2. **事件时间**：原表中的字段，适用于 CDC 数据（如从 MySQL CDC 同步的表），声明主键包含分区字段可以实现唯一效果。
3. **CDC 操作时间戳（op\_ts）**：不能定义为分区字段，因为无法知道之前记录的时间戳，需要跨分区 Upsert，消耗更多资源。

核心原则：不变且唯一。

### Paimon的表模式有哪些？他们分别有什么特点？

Paimon 的主键表采用 LSM 树结构，每个分区包含多个桶，每个桶是一个独立的 LSM 树，包含多个文件。根据写入过程中不同的处理方式，Paimon 提供了三种模式：

1. MOR（Merge On Read）：默认模式，仅进行小范围合并，读取时需要进行数据合并。
2. COW（Copy On Write）：启用全量合并，每次写入后立即完成合并，读取时不需再合并数据。
3. MOW（Merge On Write）：启用删除向量文件，写入时生成删除向量文件，读取时直接过滤不必要的行。

* Merge On Read（MOR）

1. **特点**：读取时合并所有文件，因为文件已排序，需要多路合并，包含主键比较。
2. **限制**：单一 LSM 树只能单线程读取，读取并行度有限。如果桶中数据量过大，读取性能会下降。
3. **写性能**：非常好。
4. **读性能**：不太好。

备注：如果不使用删除向量模式，可以通过配置 compaction.optimization-interval 来优化 MOR 模式的读取性能。查询优化系统表中的结果可以避免合并同一主键的记录，从而提高读取性能。

* Copy On Write（COW）

1. **特点**：每次写入都会进行全量合并，不再需要读取时合并，读取性能最高。
2. **缺点**：写入时需要完全合并，写入放大效应严重。
3. **写性能**：非常差。
4. **读性能**：非常好。

* Merge On Write（MOW）

1. **特点**：利用 LSM 结构的主键查询能力，写入时生成删除向量文件，读取时直接过滤不必要的行，相当于合并。
2. **优点**：读取性能好，写入性能好。
3. **可见性保证**：L0 级别的文件在压缩后才可见，默认情况下压缩是同步进行的，如果开启异步模式，可能会有数据延迟。

表模式选择：

* MOR：适用于写入频繁、读取性能不敏感的场景。
* COW：适用于读取需求极高、写入频率较低的场景。
* MOW：适用于均衡读写性能的场景。

### 谈谈Paimon表的文件构成

一张表的所有文件都存储在一个基本目录下，Paimon文件以分层方式组织。下图说明了文件布局：

```
warehouse  
└── default.db  
    └── my_table  
        ├── bucket-0  
        │   └── data-59f60cb9-44af-48cc-b5ad-59e85c663c8f-0.orc  
        ├── index  
        │   └── index-5625e6d9-dd44-403b-a738-2b6ea92e20f1-0  
        ├── manifest  
        │   ├── index-manifest-5d670043-da25-4265-9a26-e31affc98039-0  
        │   ├── manifest-6758823b-2010-4d06-aef0-3b1b597723d6-0  
        │   ├── manifest-list-9f856d52-5b33-4c10-8933-a0eddfaa25bf-0  
        │   └── manifest-list-9f856d52-5b33-4c10-8933-a0eddfaa25bf-1  
        ├── schema  
        │   └── schema-0  
        └── snapshot  
            ├── EARLIEST  
            ├── LATEST  
            └── snapshot-1
```

* **Snapshot Files**：包含表在某个时间点的状态信息。
* **Manifest Files**：所有清单列表（manifestlist）和清单文件（manifestfile）都存储在 Manifest Files 目录中。清单列表是清单文件名的列表，而清单文件则包含有关 LSM 数据文件和变更日志文件的信息。
* **Data Files**：数据文件按分区和存储桶分组，每个存储桶目录都包含一个 LSM 树及其变更日志文件。目前，Paimon 支持使用 orc（默认）、parquet 和 avro 作为数据文件格式。
* **Partition**：Paimon采用与 Apache Hive 相同的分区概念来分离数据，这是一种可选方法。
* **LSM Trees**：Paimon采用 LSM Tree（日志结构合并树）作为其底层的数据结构来组织和管理数据的存储和更新操作.

### 谈谈Paimon的并发控制

Apache Paimon 支持对多个并发写入作业的乐观并发控制（Optimistic Concurrency Control）。以下是对 Paimon 中乐观并发控制机制的详细解释：

1. 并发写入作业：Paimon 允许多个写入作业并发执行。每个作业按照自己的节奏写入数据，并在提交时基于当前快照生成一个新的快照。
2. 快照和增量文件：作业在提交时通过应用增量文件（删除或添加文件）来生成新快照。增量文件代表了自上次快照以来数据的变化。

乐观并发控制的优点

1. 提高并发性：乐观并发控制允许多个作业并发执行，提高了系统的并发性和吞吐量。
2. 减少锁争用：与悲观并发控制相比，乐观并发控制减少了锁的使用，从而减少了锁争用和死锁的风险。
3. 提高写入性能：作业可以按照自己的节奏写入数据，而不需要等待其他作业完成，从而提高了写入性能。

### Paimon主键表的数据合并机制有哪些？

如果将多条具有相同主键的数据写入Paimon主键表，Paimon将会根据WITH参数中设置的merge-engine参数对数据进行合并。参数取值如下：

* deduplicate（默认值）

设置'merge-engine' = 'deduplicate' 后，对于多条具有相同主键的数据，Paimon主键表仅会保留最新一条数据，并丢弃其它具有相同主键的数据。如果最新数据是一条delete消息，所有具有该主键的数据都会被丢弃。

* first-row

设置'merge-engine' = 'first-row'后，Paimon只会保留相同主键数据中的第一条。与deduplicate合并机制相比，first-row只会产生insert类型的变更数据，且变更数据的产出效率更高。

* aggregation

对于多条具有相同主键的数据，Paimon主键表将会根据您指定的聚合函数进行聚合。对于不属于主键的每一列，都需要通过`fields.<field-name>.aggregate-function`指定一个聚合函数，否则该列将默认使用last\_non\_null\_value聚合函数。

* partial-update

设置'merge-engine' = 'partial-update'后，您可以通过多条消息对数据进行逐步更新，并最终得到完整的数据。即具有相同主键的新数据将会覆盖原来的数据，但值为null的列不会进行覆盖。

### Paimon是如何处理乱序数据的？

默认情况下，Paimon会按照数据的输入顺序确定合并的顺序，最后写入Paimon的数据会被认为是最新数据。如果您的输入数据流存在乱序数据，可以通过在WITH参数中指定`'sequence.field' = '<column-name>`，具有相同主键的数据将按`<column-name>`这一列的值从小到大进行合并。可以作为sequence field的数据类型有:`TINYINT、SMALLINT、INTEGER、BIGINT、TIMESTAMP和TIMESTAMP_LTZ`。

### 谈谈Paimon表的数据压缩方式

在 LSM 树中，随着更多记录的写入，排序运行的数量会增加。查询 LSM 树需要合并所有排序运行，过多的排序运行会导致查询性能下降甚至内存不足。因此，必须定期将多个排序运行合并成一个大的排序运行，称为压缩（Compaction）。

* 异步压缩（Asynchronous Compaction）

压缩本质上是异步的，但如果希望它完全异步且不阻塞写入，可以配置最大写入吞吐量，并让压缩以较慢的速度进行。以下策略可用：

```
num-sorted-run.stop-trigger = 2147483647  
sort-spill-threshold = 10  
lookup-wait = false
```

这种配置在写入高峰期间生成更多文件，并在写入较少的期间逐步合并以优化读取性能。如果多个任务写入同一个表，需要分离压缩任务，可以使用专用的压缩任务。可以在压缩过程中配置记录级过期时间以自动过期记录。相关配置包括：

```
record-level.expire-time：记录保留时间。  
record-level.time-field：记录级过期的时间字段。  
record-level.time-field-type：记录级过期时间字段的类型，如 seconds-int 或 millis-long。
```

* 全量压缩（Full Compaction）

Paimon文件合并使用通用合并（Universal-Compaction）策略。当增量数据过多时，会自动执行全量压缩，以确保读取性能。可以通过配置定期执行全量压缩：

```
compaction.optimization-interval：定期执行全量压缩的时间间隔。  
full-compaction.delta-commits：在 delta 提交后不断触发全量压缩。
```

默认情况下，当Paimon向LSM树追加记录时，它也会根据需要执行压缩。我们还可以选择在一个专门的压缩作业来执行全量压缩操作。

主键表的 Compaction 默认在 Flink Sink 中自动完成，不用关心它的具体过程,它会在 LSM 中做到写放大与读放大的基本平衡：

* 如果压缩合并非常频繁，那么读不会放大(读的都是合并完成的数据)，写会放大(每次写都要跟所有文件合并)。
* 如果压缩合并很不频繁，那么写不会放大(每次直接写文件落盘)，读会放大(要做大量重复数据的meger)。

Paimon 支持多作业同时写入，但是并不支持多作业同时 Compaction，所以你可以给写入作业配置 write-only 为 true，避免写入作业进行后台 compaction，然后启动单独的 Dedicated Compaction Job。

### Paimon表的消费方式有哪些？

* 通过流作业消费Paimon表

又可以分为从指定位点消费Paimon表、指定Consumer ID消费。

* 通过批作业消费Paimon表

又可以分为Batch Time Travel，通过SQL Hint设置scan.timestamp-millis参数，即可查询Paimon表在该时间点的状态。

查询两次快照之间的数据变化，想要查询两次快照间Paimon表中数据发生的变化，可以通过SQL Hint设置incremental-between参数。

### Paimon的Tag有什么作用？

‌Tag‌是Apache Paimon中的一个重要概念，主要用于数据版本管理和操作。Tag是数据的"标签"，用于标记数据的某个特定版本或状态，适用于需要记录版本的场景。

Paimon 支持在写入作业中自动创建标签。

* 选择创建模式

可以通过表选项 'tag.automatic-creation' 来设置创建模式。支持的值有：

```
process-time: 根据机器的时间创建标签。  
watermark: 根据 Sink 输入的水印时间创建标签。  
batch: 在批量处理场景中，当前任务完成后生成一个标签。
```

如果选择了 Watermark，并且水印不在 UTC 时区，请配置 'sink.watermark-time-zone' 以指定水印的时区。

* 选择创建周期

用于生成标签的频率。可以为 'tag.creation-period' 选择 'daily'（每天）/'hourly'（每小时）和 'two-hours'（每两小时）。

如果需要等待迟到的数据，可以配置延迟时间：'tag.creation-delay'。

* 标签的自动删除

可以配置 'tag.num-retained-max' 或 'tag.default-time-retained' 来实现标签的自动删除。

核心的配置和参数参考：

### Paimon清理过期数据的方式有哪些？

* 调整快照文件过期时间

随着快照文件不断产生，历史数据占用的存储空间也将逐渐增加。因此，我们需要清理不再使用的快照文件，以释放该快照文件指向的历史数据文件，从而释放存储空间。

* 设置分区过期时间

如果您的业务只关心最近一段时间内的数据，您可以按时间对数据进行分区，并设置分区过期时间以自动删除过于久远的历史分区，从而释放存储空间。

* 清理废弃文件

由于作业报错重启等原因，Paimon表目录下可能会遗留一些未被提交的临时文件。这些废弃文件无法通过快照过期删除，需要手动执行以下步骤进行清理。

### Paimon的系统表包含哪些？

Paimon系统表用于存储Paimon表的元数据和特定的数据消费行为。

#### 元数据系统表

* Snapshots系统表

Snapshots系统表可以查询每个快照文件的具体信息，例如快照文件的编号，快照文件的创建时间等。

* Schemas系统表

Schemas系统表可以查询表的当前以及历史结构信息。

* Options系统表

Options系统表可以查询当前配置的表参数。

* Partitions系统表

Partitions系统表可以查询Paimon表里有哪些分区、每个分区的数据总数，以及文件总量。

* Files系统表

Files系统表可以查询某个快照文件指向的所有数据文件，包括数据文件的格式、文件内的数据条数和文件大小等。

* Tags系统表

Tags系统表可以查询每个Tag文件的具体信息，例如Tag的名称、创建Tag时使用的快照编号等。

#### 特定消费行为的系统表

* Read-optimized系统表

Paimon表在消费过程中，需要将数据文件在内存中归并之后，才能产出真正的数据，一定程度上会影响查询效率。如果您对Paimon主键表的批作业读取效率，或即席（OLAP）查询效率有较高的要求，可以消费该Paimon表对应的Read-optimized系统表。Read-optimized系统表只读取不需要合并的文件（主要是小文件全量合并后的数据文件），因此省去了在内存中归并排序的流程，提高了查询效率。然而，小文件的全量合并频率较低，因此read-optimized系统表产出的数据时效性偏低。您可以配置以下参数，使得小文件全量合并在一段时间后强制进行，在写入效率、消费效率与数据时效性之间进行平衡。

* Audit Log系统表

Paimon表对应的Audit Log系统表记录了每一条数据的操作类型是插入还是删除。

> **[**300万字！全网最全大数据学习面试社区等你来！**](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247518343&idx=1&sn=eaf44bee8c4d90aedd508201475b57a9&chksm=fd3ece12ca494704cdf02300bbfe39c1d4245ac30f41aba3065efe66eb9aa453a23e0b6dae6e&scene=21#wechat_redirect)**

如果这个文章对你有帮助，不要忘记 **「在看」** **「点赞」** **「收藏」** 三连啊喂！

[全网首发|大数据专家级技能模型与学习指南(胜天半子篇)](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247510040&idx=1&sn=aa335f25965975731173916f012d56f4&chksm=fd3eee8dca49679b82f632048fb64d21fac01497d1a0fe33917edb01e194caf0f9a1930a55ce&scene=21#wechat_redirect)

[互联网最坏的时代可能真的来了](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247508317&idx=1&sn=0bcb7fb6997b42306994b890eaa0d47f&chksm=fd3ee7c8ca496ede347a2971002c6ea68dcfd70abeeb90b43c8b02a2a60ebc7cec3a87ef7d52&scene=21#wechat_redirect)

[我在B站读大学，大数据专业](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247507860&idx=1&sn=807ac5003762f29c127bc4071dcebe33&chksm=fd3e9901ca4910178cfc816043ea86c29f881f70cd943d252de9ae2e21cb7e8ba80bff162e1b&scene=21#wechat_redirect)

[我们在学习Flink的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247499604&idx=1&sn=d938dfb30d221774704982d2938b30c1&chksm=fd3eb9c1ca4930d76a391241333de461ca22d2aa27472eab3cffab1564872ae37f1b48fe2d3c&scene=21#wechat_redirect)

[193篇文章暴揍Flink，这个合集你需要关注一下](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504856&idx=2&sn=6f62e2a0c756ce56773eed253d2f41ee&chksm=fd3e954dca491c5b732b7e2aa46db32efbc3f690e528522098659bb3c6e78e1582ab4529b5dc&scene=21#wechat_redirect)

[Flink生产环境TOP难题与优化，阿里巴巴藏经阁YYDS](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504742&idx=1&sn=8765c198d8ad66219a7bcabb221e4a23&chksm=fd3e95f3ca491ce52d0724b9e4154a47af0f5ea349e1e184c8fbc65aa17c0000242aa9b58429&scene=21#wechat_redirect)

[Flink CDC我吃定了耶稣也留不住他！| Flink CDC线上问题小盘点](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504813&idx=1&sn=f5cd6ae2aa2b1e30f87a5ae55971c514&chksm=fd3e9538ca491c2eb191677d070f2c7e4f1098eece00e256d6205b8a5d21b21c1e74f875f16f&scene=21#wechat_redirect)

[我们在学习Spark的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=1&sn=1f693e549f76622725b936041ff8896e&chksm=e9bff2fbdec87bedecbdeed4547d7d612c72fc8d4918614a26a4e31666cf0128fd57b537f347&scene=21#wechat_redirect)

[在所有Spark模块中，我愿称SparkSQL为最强！](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504834&idx=1&sn=46ca1a3924b8fdb89ac1c2acb0319d6c&chksm=fd3e9557ca491c41d837b917639ea62007ea16d3c2cc6c0c02d1f8a8c6e44318f5183b787c46&scene=21#wechat_redirect)

[硬刚Hive | 4万字基础调优面试小总结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247502750&idx=1&sn=bd9a9173d060dc4e4ebd49c8efc6acfe&chksm=fd3e8d0bca49041dea84da93910e5efdc4935e520525c09887c986691377aeb48e5cf7fb5667&scene=21#wechat_redirect)

[数据治理方法论和实践小百科全书](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504382&idx=2&sn=550b78802acfe727e0e77cd9195f8784&chksm=fd3e976bca491e7db2b8b2446d231736df01bbf13653804d680ac4390597ec17fa466ad4ae83&scene=21#wechat_redirect)

[标签体系下的用户画像建设小指南](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503741&idx=1&sn=e5039be93123f2e337013756a818bfc3&chksm=fd3e89e8ca4900fe603b63c5722a6fb8a32bd63d6ba23e0028851948a71b877eb1f742d95087&scene=21#wechat_redirect)

[4万字长文 | ClickHouse基础&实践&调优全视角解析](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503675&idx=1&sn=3ee6af64d0126c78b48cad219308f81e&chksm=fd3e89aeca4900b8b8954e9569ee3c0877881fac8c792bfafc22e7e9d3e8524da8eb860d33d8&scene=21#wechat_redirect)

[【面试&个人成长】社招和校招的经验之谈](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=2&sn=57ecc77718f1b6f2f262d62e8318dcc9&chksm=e9bff2fbdec87bedb1986765bfae7dbabb9aece45b0b2af147bbad1ac5d79ebc934c64df40ea&scene=21#wechat_redirect)

[大数据方向另一个十年开启 |《硬刚系列》第一版完结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504478&idx=1&sn=14efef3868ba42bd044f9618745a7fdc&chksm=fd3e94cbca491dddb961b7b8b93105b2869c5bcf4c03f9f0dc83ad62be0dce4c124b52ed0ba3&scene=21#wechat_redirect)

[我写过的关于成长/面试/职场进阶的文章](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504410&idx=1&sn=7e81ab5a324395eb0f12397c40247ca1&chksm=fd3e948fca491d9946456acbd93b2d651ae4d7a7d0127a211981e08f50f377e60d1b7c0d7b3a&scene=21#wechat_redirect)

[当我们在学习Hive的时候在学习什么？「硬刚Hive续集」](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504783&idx=1&sn=72aed147a459368ed934a007a2df3bc9&chksm=fd3e951aca491c0cade212390011eca7d8a68951fee6aa2844b8f4d14bcbd5b3ed26305d69dc&scene=21#wechat_redirect)