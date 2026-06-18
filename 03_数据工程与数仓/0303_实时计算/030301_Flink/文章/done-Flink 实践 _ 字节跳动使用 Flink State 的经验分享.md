> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkRocksDB深度调优|FlinkRocksDB深度调优]]、[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink大状态优化实战|Flink大状态优化实战]]
---
title: Flink 实践 | 字节跳动使用 Flink State 的经验分享
author: HBase技术社区
date:
url: http://mp.weixin.qq.com/s?__biz=MzU5OTQ1MDEzMA==&mid=2247490911&idx=1&sn=938dd48d22239aa39681b3d75b63a0c3&chksm=feb5ea22c9c263349ae1184b7f6a01884a5061dc1368d035a0d704c65bf761227ca17ed955de&mpshare=1&scene=24&srcid=0608yEnggNb69OdlRXRuYbvh&sharer_sharetime=1654672725113&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**动手点关注 干货不迷路** 👆

字节跳动流式计算团队持续招人，欢迎对 state 感兴趣的同学直接投递简历 https://job.toutiao.com/s/FoRn9Th 或添加微信

> 本文主要分享字节跳动在使用 Flink State 上的实践经验，内容包括 Flink State 相关实践以及部分字节内部在引擎上的优化，希望可以给 Flink 用户的开发及调优提供一些借鉴意义。

## 前言

Flink 作业需要借助 State 来完成聚合、Join 等有状态的计算任务，而 State 也一直都是作业调优的一个重点。目前 State 和 Checkpoint 已经在字节跳动内部被广泛使用，业务层面上 State 支持了数据集成、实时数仓、特征计算、样本拼接等典型场景；作业类型上支持了 Map-Only 类型的通道任务、ETL 任务，窗口聚合计算的指标统计任务，多流 Join 等存储数据明细的数据拼接任务。

以 WordCount 为例，假设我们需要统计 60 秒窗口内 Word 出现的次数：

```
select
    word,
    TUMBLE_START(eventtime, INTERVAL '60' SECOND) as t,
    count(1)
from
    words_stream
group by
    TUMBLE(eventtime, INTERVAL '60' SECOND), word
```

每个还未触发的 60s 窗口内，每个 Word 对应的出现次数就是 Flink State，窗口每收到新的数据就会更新这个状态直到最后输出。为了防止作业失败，状态丢失，Flink 引入了分布式快照 Checkpoint 的概念，定期将 State 持久化到 Hdfs 上，如果作业 Failover，会从上一次成功的 checkpoint 恢复作业的状态（比如 kafka 的 offset，窗口内的统计数据等）。

在不同的业务场景下，用户往往需要对 State 和 Checkpoint 机制进行调优，来保证任务执行的性能和 Checkpoint 的稳定性。阅读下方内容之前，我们可以回忆一下，在使用 Flink State 时是否经常会面临以下问题：

* 某个状态算子出现处理瓶颈时，加资源也没法提高性能，不知该如何排查性能瓶颈
* Checkpoint 经常出现执行效率慢，barrier 对齐时间长，频繁超时的现象
* 大作业的 Checkpoint 产生过多小文件，对线上 HDFS 产生小文件压力
* RocksDB 的参数过多，使用的时候不知该怎么选择
* 作业扩缩容恢复时，恢复时间过长导致线上断流

## State 及 RocksDB 相关概念介绍

### State 分类

由于 OperatorState 背后的 StateBackend 只有 DefaultOperatorStateBackend，所以用户使用时通常指定的 FsStateBackend 和 RocksDBStateBackend 两种，实际上指定的是 KeyedState 对应的 StateBackend 类型：

* FsStateBackend：DefaultOperatorStateBackend 和 HeapKeyedStateBackend 的组合
* RocksDBStateBackend：DefaultOperatorStateBackend 和 RocksDBKeyedStateBackend 的组合

### RocksDB 介绍

RocksDB 是嵌入式的 Key-Value 数据库，在 Flink 中被用作 RocksDBStateBackend 的底层存储。如下图所示，RocksDB 持久化的 SST 文件在本地文件系统上通过多个层级进行组织，不同层级之间会通过异步 Compaction 合并重复、过期和已删除的数据。在 RocksDB 的写入过程中，数据经过序列化后写入到 WriteBuffer，WriteBuffer 写满后转换为 Immutable Memtable 结构，再通过 RocksDB 的 flush 线程从内存 flush 到磁盘上；读取过程中，会先尝试从 WriteBuffer 和 Immutable Memtable 中读取数据，如果没有找到，则会查询 Block Cache，如果内存中都没有的话，则会按层级查找底层的 SST 文件，并将返回的结果所在的 Data Block 加载到 Block Cache，返回给上层应用。

### RocksDBKeyedStateBackend 增量快照介绍

这里介绍一下大家在大状态场景下经常需要调优的 RocksDBKeyedStateBackend 增量快照。RocksDB 具有 append-only 特性，Flink 利用这一特性将两次 checkpoint 之间 SST 文件列表的差异作为状态增量上传到分布式文件系统上，并通过 JobMaster 中的 SharedStateRegistry 进行状态的注册和过期。

如上图所示，Task 进行了 3 次快照（假设作业设置保留最近 2 次 Checkpoint）：

* CP-1：RocksDB 产生 sst-1 和 sst-2 两个文件，Task 将文件上传至 DFS，JM 记录 sst 文件对应的引用计数
* CP-2：RocksDB 中的 sst-1 和 sst-2 通过 compaction 生成了 sst-1,2，并且新生成了 sst-3 文件，Task 将两个新增的文件上传至 DFS，JM 记录 sst 文件对应的引用计数
* CP-3：RocksDB 中新生成 sst-4 文件，Task 将增量的 sst-4 文件上传至 DFS，且在 CP-3 完成后，由于只保留最近 2 次 CP，JobMaster 将 CP-1 过期，同时将 CP-1 中的 sst 文件对应的引用计数减 1，并删除引用计数归 0 的 sst 文件（sst-1 和 sst-2）

增量快照涉及到 Task 多线程上传/下载增量文件，JobMaster 引用计数统计，以及大量与分布式文件系统的交互等过程，相对其他的 StateBackend 要更为复杂，在 100+GB 甚至 TB 级别状态下，作业比较容易出现性能和稳定性瓶颈的问题。

## State 实践经验

### 提升 State 操作性能

用户在使用 State 时，会发现操作 State 并不是一件很"容易"的事情，如果使用 FsStateBackend，会经常遇到 GC 问题、频繁调参等问题；如果使用 RocksDBStateBackend，涉及到磁盘读写，对象序列化，在缺乏相关 Metrics 的情况下又不是很容易进行性能问题的定位，或者面对 RocksDB 的大量参数不知道如何调整到最优。

目前字节跳动内有 140+ 作业的状态大小达到了 TB 级别，单作业的最大状态为 60TB，在逐步支持大状态作业的实践中，我们积累了一些 State 的调优经验，也做了一些引擎侧的改造以支持更好的性能和降低作业调优成本。

#### **选择合适的 StateBackend**

我们都知道 FsStateBackend 适合小状态的作业，而 RocksDBStateBackend 适合大状态的作业，但在实际选择 FsStateBackend 时会遇到以下问题：

* 进行开发之前，对状态大小无法做一个准确的预估，或者做状态大小预估的复杂度较高
* 随着业务增长，所谓的 "小状态" 很快就变成了 "大状态"，需要人工介入做调整
* 同样的状态大小，由于状态过期时间不同，使用 FsStateBackend 产生 GC 压力也不同

针对上面 FsStateBackend 中存在的若干个问题，可以看出 FsStateBackend 的维护成本还是相对较高的。在字节内部，我们暂时只推荐部分作业总状态小于 1GB 的作业使用 FsStateBackend，而对于大流量业务如短视频、直播、电商等，我们更倾向于推荐用户使用 RocksDBStateBackend 以减少未来的 GC 风险，获得更好的稳定性。

随着内部硬件的更新迭代，ssd 的推广，长远来看我们更希望将 StateBackend 收敛到 RocksDBStateBackend 来提高作业稳定性和减少用户运维成本；性能上期望在小状态场景下，RocksDBStateBackend 可以和 FsStateBackend 做到比较接近或者打平。

#### **观测性能指标，使用火焰图分析瓶颈**

社区版本的 Flink 使用 RocksDBStateBackend 时，如果遇到性能问题，基本上是很难判断出问题原因，此时建议打开相关指标进行排查[1]。另外，在字节跳动内部，造成 RocksDBStateBackend 性能瓶颈的原因较多，我们构建了一套较为完整的 RocksDB 指标体系，并在 Flink 层面上默认透出了部分关键的 RocksDB 指标，并新增了 State 相关指标，部分指标的示意图如下：

造成 RocksDB 性能瓶颈的常见如下：

* 单条记录的 State Size 过大，由于 RocksDB 的 append-only 的特性，write buffer 很容易打满，造成数据频繁刷盘和 Compaction，抢占作业 CPU
* Operator 内部的 RocksDB 容量过大，如 Operator 所在的 RocksDB 实例大小超过 15GB 我们就会比较明显地看到 Compaction 更加频繁，并且造成 RocksDB 频繁的 Write Stall
* 硬件问题，如磁盘 IO 打满，从 State 操作的 Latency 指标可以看出来，如果长时间停留在秒级别，说明硬件或者机器负载偏高

除了以上指标外，另外一个可以相配合的方法是火焰图，常见方法比如使用阿里的 arthas[2]。火焰图内部会展示 Flink 和 RocksDB 的 CPU 开销，示意图如下：

如上所示，可以看出火焰图中 Compaction 开销是占比非常大的，定位到 Compaction 问题后，我们可以再根据 Value Size、RocksDB 容量大小、作业并行度和资源等进行进一步的分析。

#### **使用合理的 RocksDB 参数**

除了 Flink 中提供的 RocksDB 参数[3]之外，RocksDB 还有很多调优参数可供用户使用。用户可以通过自定义 RocksDBOptionsFactory 来做 RocksDB 的调优[4]。经过内部的一些实践，我们列举两个比较有效的参数：

* 关闭 RocksDB 的 compression（需要自定义 RocksDBOptionsFactory）：RocksDB 默认使用 snappy 算法对数据进行压缩，由于 RocksDB 的读写、Compaction 都存在压缩的相关操作，所以在对 CPU 敏感的作业中，可以通过ColumnFamilyOptions.setCompressionType(CompressionType.NO\_COMPRESSION) 将压缩关闭，采用磁盘空间容量换 CPU 的方式来减少 CPU 的损耗
* 开启 RocksDB 的 bloom-filter（需要自定义 RocksDBOptionsFactory）：RocksDB 默认不使用 bloom-filter[5]，开启 bloom-filter 后可以节省一部分 RocksDB 的读开销
* 其他 cache、writebuffer 和 flush/compaction 线程数的调整，同样可以在不同场景下获得不同的收益，比如在写少多读的场景下，我们可以通过调大 Cache 来减少磁盘 IO

这里要注意一点，由于很多参数都以内存或磁盘来换取性能上的提高，所以以上参数的使用需要结合具体的性能瓶颈分析才能达到最好的效果，比如在上方的火焰图中可以明显地看到 snappy 的压缩占了较大的 CPU 开销，此时可以尝试 compression 相关的参数。

#### **关注 RocksDBStateBackend 的序列化开销**

使用 RocksDB State 的相关 API，Key 和 Value 都是需要经过序列化和反序列化，如果 Java 对象较复杂，并且用户没有自定义 Serializer，那么它的序列化开销也会相对较大。比如去重操作中常用的 RoaringBitmap，在序列化和反序列化时，MB 级别的对象的序列化开销达到秒级别，这对于作业性能是非常大的损耗。因此对于复杂对象，我们建议：

* 业务上尝试在 State 中使用更精简的数据结构，去除不需要存储的字段
* StateDescriptor 中通过自定义 Serializer 来减小序列化开销
* 在 KryoSerializer 显式注册 PB/Thrift Serializer[6]
* 减小 State 的操作次数，比如下方的示例代码，如果是使用 FsStateBackend ，则没有太多性能损耗；但是在 RocksDBStateBackend 上因为两次 State 的操作导致 userKey 产生了额外一次序列化的开销，如果 userKey 本身是个相对复杂的对象就要注意了

```
if (mapState.contains(userKey)) {
    UV userValue = mapState.get(userKey);
}
```

更多关于序列化的性能和指导可以参考社区的调优文档[7]。

#### **构建 RocksDB State 的缓存**

上面提到 RocksDB 的序列化开销可能会比较大，字节跳动内部在 StateBackend 和 Operator 中间构建了 StateBackend Cache Layer，负责缓存算子内部的热点数据，并且根据 GC 情况进行动态扩缩容，对于有热点的作业收益明显。

同样，对于用户而言，如果作业热点明显的话，可以尝试在内存中构建一个简单的 Java 对象的缓存，但是需要注意以下几点：

* 控制缓存的阈值，防止缓存对象过多造成 GC 压力过大
* 注意缓存中 State TTL 逻辑处理，防止出现脏读的情况

### 降低 Checkpoint 耗时

Checkpoint 持续时间和很多因素相关，比如作业反压、资源是否足够等，在这里我们从 StateBackend 的角度来看看如何提高 Checkpoint 的成功率。一次 Task 级别的快照可以划分为以下几个步骤：

* 等待 checkpointLock：Source Task 中，触发 Checkpoint 的 Rpc 线程需要等待 Task 线程完成当前数据处理后，释放 checkpointLock 后才能触发 checkpoint，这一步的耗时主要取决于用户的处理逻辑及每条数据的处理时延
* 收集 Barrier: 非 Source 的 Task 中，这一步是将上游所有 Task 发送的 checkpoint barrier 收集齐，这一步的耗时主要在 barrier 在 buffer 队列中的排队时间
* 同步阶段：执行用户自定义的 snapshot 方法以及 StateBackend 上的元信息快照，比如 FsStateBackend 在同步阶段会对内存中的状态结构做浅拷贝
* 异步阶段：将状态数据或文件上传到 DFS

字节跳动内部，我们也针对这四个步骤构建了相关的监控看板：

生产环境中，「等待 checkpointLock」和「同步阶段」更多是在业务逻辑上的耗时，通常耗时也会相对较短；从 StateBackend 的层面上，我们可以对「收集 Barrier」和「异步阶段」这两个阶段进行优化来降低 Checkpoint 的时长。

#### **减少 Barrier 对齐时间**

减少 Barrier 对齐时间的核心是降低 in-flight 的 Buffer 总大小，即使是使用社区的 Unaligned Checkpoint 特性，如果 in-flight 的 Buffer 数量过多，会导致最后写入到分布式存储的状态过大，有时候 in-flight 的 Buffer 大小甚至可能超过 State 本身的大小，反而会对异步阶段的耗时产生负面影响。

* 降低 channel 中 Buffer 的数量：Flink 1.11 版本支持在数据倾斜的环境下限制单个 channel 的最大 Buffer 数量，可以通过 taskmanager.network.memory.max-buffers-per-channel 参数进行调整
* 降低单个 Buffer 的大小：如果单条数据 Size 在 KB 级别以下，我们可以通过降低 taskmanager.memory.segment-size 来减少单个 Buffer 的大小，从而减少 Barrier 的排队时间

#### **结合业务场景降低 DFS 压力**

如果在你的集群中，所有 Flink 作业都使用同一个 DFS 集群，那么业务增长到一定量级后，DFS 的 IO 压力和吞吐量会成为「异步阶段」中非常重要的一个参考指标。尤其是在 RocksDBStateBackend 的增量快照中，每个 Operator 产生的状态文件会上传到 DFS中，上传文件的数量和作业并行度、作业状态大小呈正比。而在 Flink 并行度较高的作业中，由于各个 Task 的快照基本都在同一时间发生，所以几分钟内，对 DFS 的写请求数往往能够达到几千甚至上万。

* 合理设置 state.backend.fs.memory-threshold 减小 DFS 文件数量：此参数表示生成 DFS 文件的最小阈值，小于此阈值的状态会以 byte[] 的形式封装在 RPC 请求内传给 JobMaster 并持久化在 \_metadata 里)。

+ 对于 Map-Only 类型的任务，通常状态中存储的是元信息相关的内容（如 Kafka 的消费位移），状态相对较小，我们可以通过调大此参数避免将这些状态落盘。Flink 1.11 版本之前，state.backend.fs.memory-threshold 默认的 1kb 阈值较小，比较容易地导致每个并行度都需要上传自己的状态文件，上传文件个数和并行度成正比。我们可以结合业务场景调整此参数，将 DFS 的请求数从 N(N=并行度) 次优化到 1 次
+ 这里需要注意，如果阈值设置过高(MB级别)，可能会导致 \_metadata 过大，从而增大 JobMaster 恢复 Checkpoint 元信息和部署 Task 时的 GC 压力，导致 JobMaster 频繁 Full GC

* 合理设置 state.backend.rocksdb.checkpoint.transfer.thread.num 线程数减少 DFS 压力：此参数表示制作快照时上传和恢复快照时下载 RocksDB 状态文件的线程数。

+ 在状态较大的情况下，用户为了提高 Checkpoint 效率，可能会将此线程数设置的比较大，比如超过 10，在这种情况下快照制作和快照恢复都会给 DFS 带来非常大的瞬时压力，尤其是对 HDFS NameNode，很有可能瞬间占满 NameNode 的请求资源，影响其他正在执行的作业

* 调大 state.backend.rocksdb.writebuffer.size：此参数表示 RocksDB flush 到磁盘之前，在内存中存储的数据大小。

+ 如果作业的吞吐比较高，Update 比较频繁，造成了 RocksDB 目录下的文件过多，通过调大此参数可以一定程度上通过加大文件大小来减少上传的文件数量，减少 DFS IO 次数。

#### **合并 RocksDBKeyedStateBackend 上传的文件（FLINK-11937）**

在社区版本的增量快照中，RocksDB 新生成的每个 SST 文件都需要上传到 DFS，以 HDFS 为例，HDFS 的默认 Block 大小通常在 100+MB（字节跳动内部是 512MB），而 RocksDB 生成的文件通常为 100MB 以下，对于小数据量的任务甚至是 KB 级别的文件大小，Checkpoint 产生的大量且频繁的小文件请求，对于 HDFS 的元数据管理和 NameNode 访问都会产生比较大的压力。

社区在 FLINK-11937 中提出了将小文件合并上传的思路，类似的，在字节内部的实现中，我们将小文件合并的逻辑抽象成 Strategy，这样我们可以根据 SST 文件数量、大小、存活时长等因素实现符合我们自己业务场景的上传策略。

### 提高 StateBackend 恢复速度

除了 State 性能以及 DFS 瓶颈之外，StateBackend 的恢复速度也是实际生产过程中考虑的一个很重要的点，我们在生产过程中会发现，由于某些参数的设置不合理，改变作业配置和并发度会导致作业在重启时，从快照恢复时性能特别差，恢复时间长达十分钟以上。

#### **谨慎使用 Union State**

Union State 的特点是在作业恢复时，每个并行度恢复的状态是所有并行度状态的并集，这种特性导致 Union State 在 JobMaster 状态分配和 TaskManager 状态恢复上都比较重：

* JobMaster 需要完成一个 NN 的遍历，将每个并行度的状态都赋值成所有并行度状态的并集。（这里实际上可以使用 HashMap 将遍历优化成 N1 的复杂度[8]）
* TaskManager 需要读取全量 Union State 的状态文件，比如 1000 并行度的作业在恢复时，每个并行度中的 Union State 在恢复状态时都需要读取 1000 个并行度 Operator 所产生的状态文件，这个操作是非常低效的。（我们内部的优化是将 Union State 状态在 JobMaster 端聚合成 1 个文件，这样 TaskManager 在恢复时只需要读取一个文件即可）

Union State 在实际使用中，除恢复速度慢的问题外，如果使用不当，对于 DFS 也会产生大量的压力，所以建议在高并行度的作业中，尽量避免使用 Union State 以降低额外的运维负担。

#### **增量快照 vs 全量快照恢复**

RocksDBStateBackend 中支持的增量快照和全量快照（或 Savepoint），这两种快照的差异导致了它们在不同场景下的恢复速度也不同。其中增量快照是将 RocksDB 底层的增量 SST 文件上传到 DFS；而全量快照是遍历 RocksDB 实例的 Key-Value 并写入到 DFS。

以是否扩缩容来界定场景，这两种快照下的恢复速度如下：

* 非扩缩容场景：

+ 增量快照的恢复只需将 SST 文件拉到本地即可完成 RocksDB 的初始化\***（多线程）**
+ 全量快照的恢复需要遍历属于当前 Subtask 的 KeyGroup Range 下的所有键值对，写入到本地磁盘并完成 RocksDB 初始化**（单线程）**

* 扩缩容场景：

+ 增量快照的恢复涉及到多组 RocksDB 的数据合并，涉及到多组 RocksDB 文件的下载以及写入到同一个 RocksDB 中产生的大量 Compaction，**Compaction 过程中会产生严重的写放大**
+ 全量快照的恢复和上面的非扩缩容场景一致**（单线程）**

这里比较麻烦的一点是扩缩容恢复时比较容易遇到长尾问题，由于单个并行度状态过大而导致整体恢复时间被拉长，目前在社区版本下还没有比较彻底的解决办法，我们也在针对大状态的作业进行恢复速度的优化，在这里基于社区已支持的功能，在扩缩容场景下给出一些加快恢复速度的建议：

* 扩缩容恢复时尽量选择从 Savepoint 进行恢复，可以避免增量快照下多组 Task 的 RocksDB 实例合并产生的 Compaction 开销
* 调整 RocksDB 相关参数，调大 WriteBuffer 大小和 Flush/Compaction 线程数，增强 RocksDB 批量将数据刷盘的能力

## 总结

本篇文章中，我们介绍了 State 和 RocksDB 的相关概念，并针对字节跳动内部在 State 应用上遇到的问题，给出了相关实践的建议，希望大家在阅读本篇文章之后，对于 Flink State 在日常开发工作中的应用，会有更加深入的认识和了解。

目前，字节跳动流式计算团队同步支持的火山引擎流式计算 Flink 版正在公测中，支持云中立模式，支持**公共云、混合云及多云部署，**全面贴合企业上云策略，欢迎申请试用：

## 引用

1. https://nightlies.apache.org/flink/flink-docs-master/docs/deployment/config/#rocksdb-native-metrics
2. https://arthas.aliyun.com/doc/profiler.html
3. https://ci.apache.org/projects/flink/flink-docs-master/docs/deployment/config/#advanced-rocksdb-state-backends-options
4. https://ci.apache.org/projects/flink/flink-docs-master/docs/ops/state/state\_backends/#passing-options-factory-to-rocksdb
5. https://github.com/facebook/rocksdb/wiki/RocksDB-Bloom-Filter
6. https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/serialization/custom\_serializers/
7. https://flink.apache.org/news/2020/04/15/flink-serialization-tuning-vol-1.html#performance-comparison
8. https://issues.apache.org/jira/browse/FLINK-18203