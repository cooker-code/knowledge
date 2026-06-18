> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkKeyGroup状态分配与扩缩容|FlinkKeyGroup状态分配与扩缩容]]、[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkStateBackend与CheckpointStorage解耦|FlinkStateBackend与CheckpointStorage解耦]]
---
title: 面试|深入理解Flink State
author: 大数据技术与数仓
date:
url: http://mp.weixin.qq.com/s?__biz=MzU2ODQ3NjYyMA==&mid=2247488853&idx=1&sn=b3877313bb54b9bce8e26936bbb5343c&chksm=fc8c03f6cbfb8ae0e0bddcafaeee5f2de6c073219a277f8e97208f6b196e2b4ffab2ce450fdc&mpshare=1&scene=24&srcid=0328eHMSwvxoQB8tk05dYZiQ&sharer_shareinfo=cfbeb85cf72071a09177b928188557a6&sharer_shareinfo_first=cfbeb85cf72071a09177b928188557a6#rd
---

## 写在前面

State是指流计算过程中计算节点的中间计算结果或元数据属性，比如 在aggregation过程中要在state中记录中间聚合结果，比如 Apache Kafka 作为数据源时候，我们也要记录已经读取记录的offset，这些State数据在计算过程中会进行持久化(插入或更新)。本文将详细介绍一下Flink State，通过本文，你可以了解到：

* State分类
* 什么是状态后端(state backend)
* State对扩缩容的处理

感谢关注，希望本文对你有所帮助。

## State分类

**Flink 中的状态分为两种主要类型：Keyed State 和 Operator State。**

### Keyed State

* **概念**：Keyed State 是和键（key）相关联的状态。在 Flink 的 Keyed Streams 上进行有状态操作时（例如在使用 keyBy 方法后），每个 key 都会有自己的状态实例，这个状态是独立的，即每个 key 的状态对于其他 keys 不可见。
* **用法**：Keyed State 常用于需要按 key 进行分区处理的情况，如聚合计算（sum、min、max）、窗口操作和其他需要按 key 维护和更新状态的计算。在 SQL 语句中，Keyed State 对应的就是通过 GroupBy 或 PartitionBy 所定义的字段分组。
* **数据结构**：Keyed State 底层通常是基于哈希表的实现，确保每个 key 都能快速地找到对应的状态。这种状态通常存储在 Keyed State 后端中，可以是内存中，也可以是 RocksDB 这种本地存储。

### Operator State

* **概念**：Operator State 与特定的操作符实例（Task）相关联，而不是和特定的 key 关联。每个操作符实例维护自己的状态，所有的 Operator State 实例对于同一操作符是可见的。
* **用法**：Operator State 通常用于记录源（Source）和接收器（Sink）的相关状态，或者用于需要操作符级别聚合的场合。例如，一个 Source Connector 可能会使用 Operator State 来记录已经读取的数据源的 offset。
* **实现**：Flink 提供了几种不同的 Operator State 类型，包括列表状态（ListState）、联合列表状态（UnionListState）、广播状态（BroadcastState）等。这些状态通常存储在 Operator State 后端中，可以是内存中，也可以是持久化存储。

值得注意的是：

> 在 Flink 的 Table API 或 SQL API 中，对于内部的 GroupBy/PartitionBy 操作，Flink 会自动管理 Keyed State。而对于 Source Connector 记录 offset 这样的操作，通常是在底层的 DataStream API 中实现的，可能直接使用 Operator State 来管理。例如，Flink Kafka Consumer 会使用 Operator State 来存储 Kafka 主题的分区 offset，以便在发生故障时能够从上次成功的检查点恢复。

## 什么是状态后端(state backend)

State的具体存储、访问和维护是由\*\*状态后端(state backend)\*\*决定的。状态后端主要负责两件事情：

* 本地状态管理
* 将状态以checkpoint的形式写入远程存储

Flink提供了三种状态后端：

### MemoryStateBackend（内存状态后端）

* **存储**：状态存储在 TaskManager 的 JVM 堆内存上。**生成checkpoint时，\*MemoryStateBackend会将状态发送至JobManager并保存到它的堆内存中。**
* **使用场景**：适用于小规模状态或本地测试，因为它将所有状态作为序列化数据保存在 JVM 堆上。如果 TaskManager 发生故障，状态会丢失。
* **性能**：由于状态是直接存储在内存中的，所以访问速度很快。
* **限制**：状态大小受限于 TaskManager 可用内存。大规模状态可能导致内存溢出错误。

### FsStateBackend（文件系统状态后端）

* **存储**：状态存储在 TaskManager 的 JVM 堆内存中（作为缓存），**但在检查点（checkpoint）时，会持久化到配置的文件系统（如 HDFS）中。**
* **使用场景**：适用于需要持久化状态以避免数据丢失的场景。在发生故障时，Flink 作业可以从文件系统中的检查点恢复状态。
* **性能**：由于状态在内存中进行操作，并在检查点时异步写入文件系统，因此可以提供较快的状态访问速度，但可能受文件系统性能的限制。
* **限制**：内存中的状态大小仍然受限于 TaskManager 可用内存，但由于检查点数据被写入到更稳定的文件系统，因此可以支持更大的状态。

### RocksDBStateBackend（RocksDB 状态后端）

RocksDB是一个嵌入式键值存储(key-value store)，它可以将数据保存到本地磁盘上，为了从RocksDB中读写数据，系统需要对数据进行序列化和反序列化。

* **存储**：状态存储在本地磁盘上的 RocksDB 数据库中，**检查点数据会持久化到配置的文件系统中**。
* **使用场景**：适用于大规模状态管理的场景。由于 RocksDB 是一个优化的键值存储，因此可以有效地管理大量状态数据。
* **性能**：状态访问速度可能比内存状态后端慢(磁盘读写以及序列化和反序列化对象的开销)，但 RocksDB 提供了针对大量状态数据的优化。
* **限制**：对本地磁盘空间有需求，但由于状态是在本地磁盘上操作，因此可以支持非常大的状态。

在选择状态后端时，需要考虑应用的状态大小、恢复速度、持久性和部署环境。对于生产环境，通常推荐使用 RocksDBStateBackend，因为它能够提供良好的扩展性和容错性。

## State对扩缩容的处理

### **Operator State** 的扩容处理

在 Apache Flink 中，对于有状态的流处理作业，当作业进行扩容（scaling out）或缩容（scaling in）时，即增加或减少并行子任务的数量时，Flink 需要重新分配 OperatorState。这个过程称为状态重分配（state redistribution）。

对于 **Operator State** 的扩容处理，Flink 提供了不同的重分配模式来处理状态：

#### ListState

对于 ListState 类型的 Operator State，如果流任务的并行度从 N 增加到 M，Flink 会将每个并行实例的状态分成 M 份，然后将这些分片分配给新的并行实例。如果并行度减少，则相反，状态将会聚合起来。

扩容时：

* 假设原来有 2 个并行实例，每个实例有自己的 ListState。
* 扩容到 3 个并行实例。
* Flink 会将每个原来的 ListState 平均分成 3 份。
* 新的 3 个并行实例每个都会接收一份来自每个原始 ListState 的数据。

缩容时：

* 假设原来有 3个并行实例。
* 缩容到 1 个并行实例。
* 现有的状态将会被聚合，确保新的 1 个实例完整地包含原始状态的全部数据。

#### BroadcastState

BroadcastState 的数据在扩容或缩容时会被复制到所有的并行实例中。由于 BroadcastState 是以广播的方式存储数据，所有并行实例的状态都是相同的。

#### UnionListState

对于 UnionListState 类型的 Operator State，在扩容或缩容时，状态的每个元素将保持不变，原始状态的所有元素将被统一地分发到新的并行实例中。这意味着每个元素仅分配给一个并行实例，但所有并行实例的状态的并集会包括所有原始状态的元素。随后由任务自己决定哪些条目该保留，哪些该丢弃。

思考：**Source的扩容（并发数）是否可以超过Source物理存储的partition数量呢？**

> 在使用像 Apache Kafka 这样的消息队列作为数据源（Source）时，消息队列中的数据被划分为多个分区（partitions）。这种设计主要是为了支持数据的并行处理以及提高吞吐量。在使用 Flink 或类似的流处理框架时，一个常见的做法是将每个分区分配给一个并行的 Source 实例（也称为 Source Task 或 Source Operator）进行处理。
>
> 如果尝试将 Source 的并行度（并发数）设置得比物理存储（比如 Kafka 主题）的分区数量还要高，那么将会有一些并行实例分配不到任何分区，因为分区的数量是固定的，且每个分区只能被一个并行实例消费（至少在 Flink 的默认设置下是这样）。这会导致资源浪费，因为超出分区数量的那部分并行实例不会做任何实际的数据处理工作，但仍然占用系统资源。
>
> 因此，在设置 Source 的并行度时，通常的最佳实践是：
>
> * 确保 Source 的并行度不超过其对应物理存储（如 Kafka 主题）的分区数量。
> * 如果需要增加并行度以提高处理能力，相应地也需要增加物理存储的分区数量。对于 Kafka 来说，可以通过修改主题的分区配置来实现。
>
>   对于 Apache Flink，如果使用的是 Flink Kafka Connector，并且尝试将并行度设置得比 Kafka 主题的分区数量还要高，Flink 会在作业启动时进行检查。如果发现这种配置不匹配的情况，Flink 会抛出异常并终止作业启动，以避免资源浪费和潜在的配置错误。这种设计选择确保了资源的有效利用和处理能力的合理分配，同时也避免了由于配置错误而导致的潜在问题。

### KeyedState对扩容的处理

* 什么是Key-Groups

KeyedState的算子在扩容时会根据新的任务数量对key进行重分区，为了降低状态在不同任务之间迁移的成本，Flink不会单独对key进行在分配，而是会把所有的键值分别存到不同的key-group中，每个key-group都包含了部分键值对。一个key-group是State分配的原子单位。

* 什么决定Key-Groups的个数

key-group的数量在job启动前必须是确定的且运行中不能改变。**由于key-group是state分配的原子单位，而每个operator并行实例至少包含一个key-group**，因此operator的最大并行度不能超过设定的key-group的个数，**那么在Flink的内部实现上key-group的数量就是最大并行度的值**。

* 如何决定key属于哪个Key-Group

为了决定一个key属于哪个Key-Group，通常会采用一种叫做一致性哈希（Consistent Hashing）的算法。一致性哈希算法的基本思想是将所有的Key和所有的Key-Group都映射到同一个哈希环上。对每个Key进行哈希运算得到一个哈希值，然后在哈希环上找到一个顺时针方向最近的Key-Group，这个Key就属于这个Key-Group。即：**Key到指定的key-group的逻辑是利用key的hashCode和maxParallelism取余操作的来分配的。**

如下图当parallelism=2,maxParallelism=10的情况下流上key与key-group的对应关系如下图所示：

img

如上图key(a)的hashCode是97，与最大并发10取余后是7，被分配到了KG-7中，流上每个event都会分配到KG-0至KG-9其中一个Key-Group中。

上面的Stateful Operation节点的最大并行度maxParallelism的值是10，也就是我们一共有10个Key-Group，当我们并发是2的时候和并发是3的时候分配的情况如下图：

img

先计算每个Operator实例至少分配的Key-Group个数，将不能整除的部分N个，平均分给前N个实例。最终每个Operator实例管理的Key-Groups会在GroupRange中表示，本质是一个区间值。比如上图是2->3扩容，那每个task的key-group的数量是：10/3≈3，也即是每个task先分3个key-group，然后把剩余的1个key-group分配给第一task。

**值得注意的是：**

**Key-Group机制的特点就是每个具体的key(event)不关心落到具体的哪个task来处理，只关心会落到哪个Key-Group中：**

* **首先 一个job运行之后，如果要复用state，不允许在修改maxParallelism。**
* **key 值的hash code决定落到哪个KG中，key本身不关系被哪个task处理，也就是说相同的KG在扩容前后可能被不同的task处理。**

## 总结

State是Flink流计算的关键部分。**Flink 中的状态分为两种主要类型：Keyed State 和 Operator State。**Flink提供了三种状态后端：MemoryStateBackend、FsStateBackend、RocksDBStateBackend。对于**Keyed State 和 Operator State应对扩缩容时有不同的分配方式。**