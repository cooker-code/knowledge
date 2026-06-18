> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink通用增量Checkpoint|Flink通用增量Checkpoint]]
---
title: 基于 Log 的通用增量 Checkpoint
author: Apache Flink
date:
url: http://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247506107&idx=1&sn=9c9d9a626b75014ff1a206db7d38f10e&chksm=fd3856f9ca4fdfefb3f1962766c92ec697569f32973d3d3977f4e1286efed2c8a157af5d2ded&mpshare=1&scene=24&srcid=0530J42pvxSEvsDeE380AwDc&sharer_sharetime=1685420729933&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**摘要：**本文整理自阿里巴巴开发工程师，Apache Flink Contributor 俞航翔，在 Flink Forward Asia 2022 核心技术专场的分享。本篇内容主要分为四个部分：

1. Checkpoint 性能优化之路

2. Changelog 机制解析

3. Changelog 性能测试

4. 总结与规划

**Tips：**点击**「阅读原文」**查看原文视频&演讲 ppt

**01**

**Checkpoint 性能优化之路**

##

### **1.1 Checkpoint 总览**

众所周知，Flink 是有状态的分布式计算引擎，状态是 Flink 中非常重要的概念，而在 Flink 中状态和 Checkpoint 机制是密不可分的，因此在讨论 Flink 在 Checkpoint 上优化历程之前，先来看下为什么 Checkpoint 这么重要， Checkpoint 到底做了些什么呢？

Checkpoint 概念并不陌生，它在各种系统中都出现过，其主要目的就是容错及保证应用在发生故障后依然能够正常运行。故障对长期应用的系统是无法避免的，而数据处理延迟是流计算系统中非常重要的指标。如何在发生故障后保证应用尽快恢复并追上最新的数据是流计算系统需要重点解决的问题。而相比基于容机制的故障恢复，Checkpoint 机制会更轻量、更易用。

进一步，很多业务对故障恢复后的数据一致性提出了更高的要求。Flink 的 Checkpoint 机制支持了 Exactly-once 语义，在 Source 支持回放和 Sink 支持事务后，可以做到端到端的 Exactly-once 语义。在 Checkpoint 和恢复性能优化到一定程度后，应用可以做到真正的、仿佛没有出现故障似的长期运行。

Flink 是如何基于 Checkpoin 机制做到的呢？

在作业的运行过程中，Flink 的 Stateful 算子会通过 State 记录多个 events 之间的信息，Flink 会定期执行 Checkpoint 把这些状态持久化，将全局一致性快照上传到远端存储中，而在发生故障后，Flink 的每个 Task 将会下载持久化的状态数据到本地并重新构建本地的状态数据结构。如果 Source 支持重放，整个 Pipeline 会从记录的上一个位点开始重放，作业开始正常运行。

基于这两个部分的讨论，可以了解到 Checkpoint 需要围绕两个重要目标设计：轻量级和快速的 Failover。

### **1.2 更轻量、更快速的 Checkpoint**

基于以上两个设计目标，Flink 在 Checkpoint 做了诸多优化，我们可以结合 Checkpoint Metrics 来一览这些优化的作用。

在 0.9 这个版本中，Flink 引入了轻量的异步快照算法。这个算法有两个核心点：

* 一是在作业粒度，将 Barrier 作为特殊的 Record 在 Graph 中传递，收到 Barrier 算子将执行 Checkpoint。
* 二是在算子粒度，Checkpoint 的执行步骤分成了同步阶段和异步阶段，并将较重的操作，如文件上传等，放到了异步阶段。

在 Metrics 中我们可以看到 Checkpoint 端到端的耗时被分成了多个时间段，其中我们在遇到 Checkpoint 性能问题时，首先会查看的就是同步阶段的耗时和异步阶段的耗时。之后出现的各项技术就是主要针对这两个阶段的优化。

在 1.0 这个版本中，Flink 支持了 RocksDB StateBackend，让大状态作业拥有了更高的稳定性，然而大状态的 Checkpoint 却成为了瓶颈。随着状态增大，我们可以看到 Full Checkpoint Data Size，即全量 Checkpoint 的数据量会有明显增大，进而导致 Checkpoint 异步部分的耗时增加明显。

因此在 1.3 版本中，Flink 支持了基于 RockDB 的 Incremental Checkpoint。在这种机制下，State Backend 在异步阶段只需要上传增量文件即可，大大减少了 Checkpoint 在异步阶段上传的文件量，从而缩短了 Checkpoint 的异步耗时。

通常，在 Metrics 中看到异步阶段耗时过长，同时 Full Checkpoint Data Size 较大时，可以首先考虑开启该配置。开启之后，可以通过 Checkpointed Data Size 看到增量部分的大小。

那么同步阶段的耗时可以进一步缩小吗？

在生产实践过程中，在刚才的算法下，Alignment Duration，即对齐时间，通常是同步阶段耗时较长的部分。例如，作业中间的一个算子可能会收到多个上游算子的输入，而在刚刚提及的算法中，为了保证 Exactly-once 语义，需要等到多个 Barries 对齐后，算子才会触发整个 Checkpoint，这会导致整体 Checkpoint 可能会因为链路的算子处理过慢而让整个 Checkpoint 做不出来。这个时候对齐时间就会变长。因此在 1.11 中，Flink 支持了 Unaligned Checkpoint，并在 1.13 中该功能达到 Production Ready 状态。

该功能开启后，在单个 Barrier 到达某个算子时，可以直接被传递到 Output Buffer 最后，同时直接触发 State Backend 层的 Checkpoint，并把接受最后一个 Barrier 前的 Input Buffer 和 Output Buffer 中的 inflate data 存储到远端。而在恢复时，额外恢复这一部分的数据，并回放 Input Buffer 中的数据到算子上。这种方式在减少同步对齐时间的同时，还提供了 Exactly-once 的语义保障。

这个时候我们可以在 Flink UI 上看到 Unaligned Checkpoint 已被置为 true，同时可以看到 persisted in-flight data 有明显的上升。

但 Unaligned Checkpoint 的打开会因为要存储额外的 in-flight data 导致空间放大问题，有没有办法可以减少这部分空间的开销呢？同时 Checkpoint Metrics 中还有个指标——Start Delay。这个指标是 Barrier 从创建到到达该算子的时间，有没有办法可以进一步加速整个作业的 Barrier 流动，让 Barrier 更快能够到达某个算子，从而更快触发 Checkpoint 呢？

为了解决这个问题，1.14 中引入了 Buffer Debloating 机制。该功能可以通过检测网络流量情况来动态调节 Network Buffer 的大小，从而加快 Aligned Checkpoint 触发，减少 Unaligned Checkpoint 的额外空间开销。

以上各项技术可以优化 Checkpoint 的同步耗时和异步耗时，其中 Unaligned Checkpoint 和 Buffer Debloating 机制的结合可以相对较小的代价来大大减少同步阶段的耗时。

那么异步阶段的耗时是否有进一步改进的空间呢？1.15 中引入了通用增量 Checkpoint，即 Changelog StateBackend，该功能在 1.16 中达到 Production Ready 的状态，也就是这次分享的主角。

**02**

**Changelog 机制解析**

##

### **2.1 RocksDB Incremental Checkpoint 机制**

首先，正如我们刚才提到，基于 RocksDB 的 Incremental Checkpoint 机制可以减少异步阶段需要上传的文件量，但是他没法从根本上保证 Checkpoint 过程的快速和稳定。为什么呢？

我们先来回顾下它的机制。在写入流程中，Record 会先被写入到 RocksDB 的内存数据结构，即 MemTable 中。然后 MemTable 在满了之后会变成 ImmuTable MemTable。在达到阈值时，ImmuTable MemTable 会被 Flush 到磁盘中形成 SST Files。

这个过程可能会触发 SST 的 Compaction，在 Level Compaction 机制下，可能会触发多层的级联 Compaction，进一步产生大量的新文件。如图所示，黄色标注的新文件和旧文件就形成了全量的 SST。

在 Checkpoint 的同步流程中，会先强制触发 MemTable 的 Flush，类似于刚才的过程是可能会触发 SST 的 Compaction 的。然后会执行一个本地的 Checkpoint。这个其实是一个 Hard Link 的过程，是相对轻量的。而在 Checkpoint 的异步流程中，则会把增量部分的文件上传到远端存储中，并写入一些 Meta 信息。

这个过程中存在的问题：

首先，Flush 可能会触发多层 Level Compaction，进一步导致大量文件需要重新上传。

其次，除了数据量达到阈值时，Checkpoint 的同步阶段也是会强制 Flush MemTable。在大规模作业下，会导致多个 Task 同时触发 Compaction 和文件上传，进一步导致资源和 Checkpoint 耗时随 Checkpoint 周期抖动。

最后，Checkpoint 的端到端耗时取决于最长的链路。而对于大规模作业，每次 Checkpoint 都可能因为某个 Task 的异步时间过长而导致整体耗时变长，最终导致 Checkpoint 耗时长且不稳定。

因此我们引入了 Changelog 机制，其目标就是为了进一步提供稳定且快速的 Checkpoint。 

首先，RocksDB 由于 Compaction 导致上传文件量的不稳定，进一步会导致 Checkpoint 耗时突增，而开启 Changelog 就可以大幅减少 Checkpoint 耗时突增的情况。同时在生产实践中，我们有时会发现 CPU 和网络带宽随着 Checkpoint 周期性抖动。这通常是因为 Checkpoint 触发了多个 Task 的 Compaction，同时导致大量文件需要重新上传，这是可能会影响到作业本身的稳定性甚至其他作业的稳定性的。而 Changelog 是可以改善这个情况的。

其次，Changelog 通过上传相对固定增量的方式，极大减少异步部分耗时，保证在状态量较大时依然能在秒级/亚秒级完成 Checkpoint。

而更快速的 Chekpoint 可以带来什么好处呢？

一个是更小的端到端延迟。Transactional Sink 的提交依赖于 Checkpoint 的完成，而 Checkpoint 完成得越快，可以让 Sink 的数据越新鲜。另一个是更少的数据回追，我们知道在 Failover 时会从最近的一个 Checkpoint 恢复并回追位点后的数据，而更快的 Checkpoint 也就意味着更少的数据回追。

当然，Changelog 会带来额外的开销，如空间开销、恢复耗时开销等，但总体上这些开销是可控的，且相比其带来的收益，开销是相对较小的，后面我们会通过实验来进一步说明。

那 Changelog 是如何做到这些提升的呢？

Changelog 的机制其实很像 DB 中的 Checkpoint + WAL 机制，因此在介绍 Changelog 机制之前，我们先看一下 DB 中的类似机制。

为了能快速从故障中恢复，DB 中通常会开启 WAL 功能。当开启该功能并写入数据到 DB 中时，会将 Record 以操作日志的形式先写入到磁盘的 WAL 结构中，然后再更新内存的数据结构，同时在内存和磁盘间同步。由于 WAL 是顺序写入，因此该过程是非常快的。而在触发 Checkpoint 时，DB 会在本地做一个全量快照，同时在完成后将历史的 WAL Truncate 掉。而在故障恢复时，会首先从 Checkpoint 中重新构建 DB，并将 WAL 回放到 DB 中，从而恢复到故障前的状态。

为什么会需要这两种机制呢？首先，因为相比 WAL，Checkpoint 的操作相对较重，无法做到频繁且细力度的快照，而顺序写的 WAL 可以做到。其次，WAL 有较大的空间放大问题，这在恢复时需要额外的回放开销，因此定期的 Checkpoint 可以减少这部分开销。

### **2.2 Changelog Incremental Checkpoint 术语**

我们可以基于 DB 中的机制引申到 Changelog 中的一些概念。State Table 是算子本地状态数据的读写结构，比如 RocksDB，其类似于上图左侧的 DB 数据结构。

State Changelog 是以 Append-only Log 形式存储的状态记录。DSTL 则是 Changelog 部分的存储组件，这两个部分类似于左图中的 WAL 结构。Materialization 是 State Table 的持久化过程，会定时触发，并且在成功之后会穿 Turncate 掉 Changelog，类似于上图左侧中的 Checkpoint 的过程。

因此，简而言之，Changelog 就是通过额外的 DSTL 持续上传增量，并利用 Materialization 来完成定期刷新 State Table 全量快照的过程。

### **2.3 Changelog Incremental Checkpoint 核心机制**

我们可以进一步来看它的一些核心机制。首先，在作业粒度，端到端的整个过程是在 Flink Checkpoint 的机制下完成的，是与之前类似的。而在算子粒度，我们可以同样通过三条链路来看整体过程：

1. 在读写链路上，新写入的 Record 会以 State Changelog 的形式写入 DSTL 和 State Table 中，而读取时只会从 State Table 中读取。

2. 在 Checkpoint 链路上，包含四个主要部分：

* 一是在作业运行时，Changelog 部分会被 DSTL 定期连续上传到远端存储中。
* 二是在 Flink Checkpoint 触发时，Changelog 部分会被直接上传。
* 三是 Materialization 会被定时触发，同时在完成后会 Truncate 掉历史 Changelog。
* 四是基于上述三个过程，JM 将获取到由 Materialization 部分和 Changelog 部分组成的 Handle 保存到 Meta 信息中。

3. 在恢复链路上，首先会恢复 State Table，包括从远端下载文件和重新构建 State Table，然后再从远端下载并重新回放 Changelog 部分。

在该机制下，Checkpoint 的过程会变得非常平滑。每个 Task 会定时触发 Materialization 去持久化全量数据，而在作业运行时和 Flink Checkpoint 触发时，都会上传相对较小的增量数据。

在这个机制下，每个 Task 的 Materialization 过程是相对独立且不会影响到 Checkpoint 上传的数据量，因此 State Table 的具体机制和 Incremental Checkpoint 过程是解耦开的，Checkpoint 的过程可以变得非常稳定且快速。

同样在恢复时，每个 Task 将基于最近的 Materialization 构建 State Table，并在其上回放 Materialization 和最近 Checkpoint 之间的 Changelog 历史即可。

伴随着 Checkpoint 变得快速且稳定，Changelog 是会带来一些开销的，主要分为三个方面：

1. 额外的存储空间开销。我们知道 Changelog 部分是以操作日志的形式写入的，目前没有 Merge 机制，在 Truncate 之前是会持续增长的。因此相对原生的如 Rocksdb 的机制，是会有空间放大的问题的。

值得一提的是，远端存储的费用是相对廉价的，比如 Aliyun OSS 标准存储大约在 0.12元/GB/月，我们可以考虑用少量增长的远端存储的费用来换取更稳定且快速的 Checkpoint。

2. 额外的恢复开销。在恢复时除了 State Table 部分的恢复，我们还需要下载并回放 Changelog 部分，这部分耗时就是额外的下载和回放带来的。

3. 额外的性能开销。Changelog 机制引入了额外的双写步骤，是会对作业 TPS 上限产生影响的。值得注意的是，这里影响的是作业的 TPS 上限，而从日常运行的 TPS per core 来看，Changelog 是可能达到更高的。

在后面的实验中，我们可以看到整体的开销是可控的。

##

**03**

**Changelog 性能测试**

##

这里将通过三个实验来验证 Changelog 机制的稳定性和执行性能，并观察它在空间放大、恢复性能和极限 TPS 上的开销。

###

### **3.1 Changelog Incremental Checkpoint 的使用**

首先介绍一下 Changelog 的使用方式，以及结合事例介绍在实验中的相关基础配置。

* 第一个参数是启用 Changelog 的参数，我们只要把它设置成 true 即可开启 Changelog，1.16 中我们也支持了该参数的兼容性。
* 第二个参数是 Materialization 的间隔，它可以一定程度上控制空间放大，在实验中，我们把它设置为 3 分钟。
* 第三个参数是 Changelog 部分的存储介质，生产实践中我们可以设置成 File System 来将 Changelog 存储在 DFS 中；DFS 也是目前唯一支持的 Changelog 存储，之后我们也会考虑推出其他存储形式。
* 第四个参数是 Changelog 在 DFS 上的存储路径，在第三个参数设置为 Filesystem 时需要设置，配置方式类似 Checkpoint 路径。

###

### **3.2 Benchmark 实验设计**

整个测试分为三个实验来对比测试 RocksDB 和开启 Changelog 时的作业指标，来观察 Changelog 为作业带来的好处和额外的开销，同时说明开销的可控性。

实验 A：对比测试在日常流量下，不同状态大小下的CP稳定性、速度和空间放大率。其中我们设置了单并发 100MB 和 1.2GB 两种设置，分别对应状态全内存和落盘后的场景。

实验 B：在实验 A 的基础上，通过手动制造异常来触发 Failover，以对比测试在日常流量下作业恢复的性能。

实验 C：让作业反压，对比测试状态算子在反压时的 TPS，以观察 Changelog 双写对 TPS 的影响。

这里我们主要记录了 Value State 的测试结果，我们也测试了不同类型 State 和不同 Operator 的性能，如 Window State，这部分数据将在后续的 Blog Post 中进一步分享给大家。

首先我们可以从 Checkpoint 耗时图来看 Checkpoint 的稳定性。

由上图可以看到，RocksDB 的 Checkpoint duration 是相对不稳定的，如果打开 SubTask 详情和 Rocksdb Metric 可以进一步看到，这个时间的长短是和 Compaction 相关的。而 Changelog 是相对稳定的，持续上传增量的方式可以让 Changelog 的 Checkpoint 执行得非常稳定。

另外，我们可以看到 Checkpoint Duration 的变化是和 Checkpoint 周期有相关性的，比如每四个会久一些，这个是因为前面说的 Checkpoint 同步阶段时会触发 MemTable Flush，而 RocksDB 的 LO 层默认每 4 个会触发一次 Compaction 从而导致更多文件更新，这会进一步加剧 Rocksdb Checkpoint 的不稳定性。

关于 Checkpoint 的执行性能，从途中可以看到，随着 State Size 增大，打开 Changelog 后 Checkpoint 的 p99 耗时可以稳定在 1s 左右，而 Rocksdb 则从 8 秒增加到近 20 秒。这是因为 Changelog 持续上传固定增量的方式，可以让触发 Checkpoint 时的增量大小变得非常稳定且快速。

在实验 A 中可以看到，开启 Changelog 是会有额外空间开销的，相比 Rocksdb 的空间放大大约在 1.2-2 倍，其中状态较小，比如在内存中时会更大。这是因为 RocksDB 的合并机制相比 Changelog 记录操作日志的方式能更好地减少总数据量。特别地，当状态在内存中时，RocksDB 能有更好的合并效果。

针对这个情况，我们可以通过支持 Changelog 部分的 Merge 来进一步减小空间上的开销。但就目前现状而言，如上文所说，远端存储的费用是相对廉价的，我们完全可以考虑用少量增长的远端存储的的费用来换取更稳定且快速的 Checkpoint。

可以通过支持 Changelog 部分的 Merge 来减小空间上的开销。但就目前现状而言，远端存储的费用是相对廉价的。如上文所说，可以考虑用少量增长的远端存储费用来换取更稳定且快速的 Checkpoint。

实验 B 主要测试额外恢复开销。可以看到，在不打开 Local Recovery 时，打开 Changelog 相比 Rocksdb 约有 40% 的额外耗时开销，打开 Local Recovery 后，两者的恢复开销都可以接近忽略。同时，结合 Checkpoint 端到端耗时的提升，考虑作业整体的 Failover 流程，打开 Changelog 后还是会更快的。

Changelog 恢复时的开销主要来自哪里呢？由下面的公式可以看到，额外开销主要来自 Changelog 部分的下载和回放。而在开启 local recovery 后下载时间被抹去，只有回放上还有额外较小的开销。这个部分可以进一步优化，比如通过 Remote Apply，即远端预先 Merge 的方式，运行时将 Changelog 部分持续 Apply 到 State Table 的文件中，进而缩小需要回放的数据。同时这种方式也能减少远端存储开销。

但目前来看开销上影响不是那么大，也欢迎大家使用反馈，以便我们能关注并开发最核心的优化点。

实验 C 令作业反压，同时通过分步启用 Checkpoint 和 Local Recovery 来观察 Changelog 和开启 Local Recovery 后对极限 TPS 的影响。下图中可以看出，由于双写的额外开销相比于 RocksDB，Changelog 的极限 TPS 下降约 10%~20%。而由于三写的额外开销，打开 Local Recovery 后极限 TPS 约下降 5% 左右。这个部分的性能改善已经在 FLINK-30345 中完成，进一步的测试详情可以参考后续的 Blog Post。

**04**

**总结与规划**

##

本篇分享首先从 Checkpoint 的基本概念出发，介绍了多项 Checkpoint 的优化技术和他们在 Flink UI 中的体现，然后从 RocksDB Incremental Checkpoint 机制存在的问题出发，引出了 Changelog 的设计目标，并解析了 Changelog 的 Checkpoint 机制。最后通过三组实验直观的感受了 Changelog 的优势和开销，同时也介绍了 Changelog 的使用。

未来我们会围绕三个方向进行优化：

* 针对上面三个问题进行进一步的性能优化。
* 作为 Fault Tolerance 2.0 中的一员，让整个容错过程变得更轻量、更易用。
* 结合 Table Store，为 Table Store 提供更高的数据新鲜度。

往期精选

---

▼ 活动推荐▼

▼ 关注「**Apache Flink**」，获取更多技术干货 ▼

   **点击「阅****读原文****」，**查看原文视频&演讲** **PPT****