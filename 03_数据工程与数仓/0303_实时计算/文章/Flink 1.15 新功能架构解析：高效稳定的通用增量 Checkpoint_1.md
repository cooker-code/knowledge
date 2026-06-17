---
title: Flink 1.15 新功能架构解析：高效稳定的通用增量 Checkpoint
author: Apache Flink
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247497497&idx=1&sn=a59b8d8b7eaaae1bf8a892a45e934faf&chksm=fd38795bca4ff04da5e571ebc742aafa0f1d670259476aeaa1912df16fc6f24b53862b953905&mpshare=1&scene=24&srcid=0526BtKQryPhg8UOlUG3i9Ar&sharer_sharetime=1653573667285&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

作者｜梅源（Yuan Mei）&  Roman Khachatryan

流处理系统最重要的特性是端到端的延迟，端到端延迟是指开始处理输入数据到输出该数据产生的结果所需的时间。Flink，作为流式计算的标杆，其端到端延迟包括容错的快慢主要取决于检查点机制（Checkpointing），所以如何将 Checkpoint 做得高效稳定是 Flink 流计算的首要任务。我们在 [“Flink 新一代流计算和容错——阶段总结和展望”[1]](http://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247495906&idx=1&sn=8806069228df05ec3633de7a1e305227&chksm=fd387ea0ca4ff7b61fcde0d8b7b051b2ce91fa3bbda96789d9dc62fe1fb08077eb5a2d16e626&scene=21#wechat_redirect) 一文中介绍了 Flink 从社区 1.12 版本开始所做的提升 Checkpointing 机制的努力，本文将着重介绍其中刚刚在 Flink 1.15 版本发布的 Generic Log-Based Incremental Checkpointing 这个功能。

点击查看 Apache Flink 1.15 发布公告

**一、概述**

## 

Generic Log-Based Incremental Checkpointing 的设计初衷是我们将全量的状态快照和增量的检查点机制分隔开，通过持续上传增量 Changelog 的方法，来确保每次 Checkpointing 可以稳定快速的完成，从而减小 Checkpointing 之间的间隔，提升 Flink系统端到端的延迟。拓展开来说，主要有如下三点提升：

1. **更短的端到端延迟**：尤其是对于 Transactional Sink。Transactional Sink 在 Checkpoint 完成的时候才能完成两阶段提交，因此减小 Checkpointing 的间隔意味着可以更频繁的提交，达到更短的端到端的延迟。
2. **更稳定的 Checkpoint 完成时间**：目前 Checkpoint 完成时间很大程度上取决于在 Checkpointing 时需要持久化的（增量）状态的大小。在新的设计中，我们通过持续上传增量，以达到减少 Checkpoint Flush 时所需要持久化的数据，来保证 Checkpoint 完成的稳定性。
3. **容错恢复需要回滚的数据量更少**：Checkpointing 之间的间隔越短，每次容错恢复后需要重新处理的数据就越少。

那是怎么做到的呢？我们知道影响 Flink Checkpointing 时间的主要因素有以下几点：

1. Checkpoint Barrier 流动和对齐的速度；
2. 将状态快照持久化到非易失性高可用存储（例如 S3）上所需要的时间。

对 Flink Checkpoint 机制不太了解的读者可以参考：

Flink 1.12 版本引入的 Unaligned Checkpoint[2]和 1.14 版本中引入的 Buffer Debloating[3]主要解决了上述第 1 个问题，尤其是在反压的情况下。更早之前引入的 Incremental Checkpoint[4]是为了减少每次 Checkpointing 所需要持久化存储状态的大小，以减小第 2 个影响因素，但在实际中也不完全能做到：现有 Incremental Checkpoint 是基于 RocksDB 来完成的，RocksDB 出于空间放大和读性能的考虑会定期做 Compaction。Compaction 会产生新的、相对较大的文件，会增加上传所需要的时间。每一个执行 Flink 作业的物理节点（Task）至少有一个 RocksDB 实例，所以 Checkpoint 被延迟的概率会随着物理节点增多而变大。这导致在 Flink 的大型作业中，几乎每次完成 Checkpointing 时都有可能会因为某个节点而延迟，如下图所示。

图1: 每次 Checkpoint 都可能因为某个节点上传文件缓慢而延迟

另外值得一提的是在现有的 Checkpointing 机制下，Task 只有在收到至少一个 Checkpoint Barrier 之后，才会做状态快照并且开始持久化状态快照到高可用存储，从而增加了 Checkpoint 完成时间，如下图所示。

图2: 在现有机制下，快照在 Checkpoint Barrier 到达之后才会开始上传

在新的设计中，我们通过持续上传增量 Changelog 的方法，可以避免这个限制，加速 Checkpoint 完成时间。下面我们来看看详细的设计。

**二、设计**

## 

Generic Log-Based Incremental Checkpointing 的核心思想是引入 State Changelog（状态变化日志），这样可以更细粒度地持久化状态：

1. 算子在更新状态的时候写双份，一份更新写入状态表 State Table 中，一份增量写入 State Changelog 中。
2. Checkpoint 变成由两个部分组成，第一个部分是当前已经持久化的存在远端存储上的 State Table，第二个部分是增量的 State Changelog。
3. State Table 的持久化和 Checkpointing 过程独立开来，会定期由 background thread 持久化，我们称为 Materialization（物化）的过程。
4. 在做 Checkpoint 的时候，只要保证新增的 State Changelog 被持久化就可以了。

新的设计中需要在做 Checkpoint 的时候上传的数据量变得很少，不仅可以把 Checkpoint 做得更稳定，还可以做得更高频。整个工作流程如下图所示：

图3: Generic Log-Based Incremental Checkpointing 工作流程

Generic Log-Based Incremental Checkpointing 类似传统数据库系统的 WAL 机制：

1. 数据的增量更改（插入/更新/删除）会被写入到 Transaction Log 中。一旦这部分更改的日志被同步到持久存储中，我们就可以认为 Transaction 已经完成了。这个过程类似于上述方法中的 Checkpointing 的过程。
2. 同时，为了方便数据查询，数据的更改也会异步持久化在数据表（Table）中。 一旦 Transaction Log 中的相关部分也在数据表中被持久化了，Transaction Log 中相关部分就可以删除了。这个过程类似于我们方法中的 State Table 持久化过程。

这种和 WAL 类似的机制可以有效提升 Checkpoint 完成的速度，但也带来一些额外的开销：

1. 额外的网络 IO 和额外的 Changelog 持久存储开销；
2. 缓存 Changelog 带来的额外的内存使用；
3. 容错恢复需要额外的重放 Changelog 带来的潜在的恢复时间的增加。

我们在后面的 Benchmark 对比中，也会对这三方面的影响进行分析。特别对于第 3 点，额外的重放 Changelog 所带来的容错恢复时间增加会在一定程度上因为可以做更频繁的 Checkpoint 所弥补，因为更频繁的 Checkpoint 意味着容错恢复后需要回放的处理数据更少。

**三、Changelog 存储（DSTL）**

## 

Generic Log-Based Incremental Checkpointing 的很重要的一个组件是 State Changelog 存储这个部分，我们称之为 Durable Short-term Log（DSTL，短存 Log）。DSTL 需要满足以下几个特性：

* **短期持久化**

  State Changelog 是组成 Checkpoint 的一个部分，所以也需要能持久化存储。同时，State Changelog 只需要保存从最近一次持久化 State Table 到当前做 Checkpoint 时的 Changelog，因此只需要保存很短时间（几分钟）的数据。

* **写入频率远远大于读取频率**

  只有在 Restore 或者 Rescale 的情况下才需要读取 Changelog，大部分情况下只有 append 操作，并且一旦写入，数据就不能再被修改。

* **很短的写延迟**

  引入 State Changelog 是为了能将 Checkpoint 做得更快（1s 以内）。因此，单次写请求需要至少能在期望的 Checkpoint 时间内完成。

* **保证一致性**

  如果我们有多个 State Changelog 的副本，就会产生多副本之间的一致性问题。一旦某个副本的 State Changelog 被持久化并被 JM 确认，恢复时需要以此副本为基准保证语义一致性。

从上面的特性也可以看出为什么我们将 Changelog 存储命名为 DSTL 短存 Log。

### **3.1 DSTL 方案的选择**

DSTL 可以有多种方式实现，例如分布式日志（Kafka）、分布式文件系统（DFS），甚至是数据库。在 Flink 1.15 发布的 Generic Log-Based Incremental Checkpointing MVP 版本中，我们选择 DFS 来实现 DSTL，基于如下考虑：

1. **没有额外的外部依赖**：目前 Flink Checkpoint 持久化在 DFS 中，所以以 DFS 来实现 DSTL 没有引入额外的外部组件。
2. **没有额外的状态管理**：目前的设计方案中 DSTL 的状态管理是和 Flink Checkpointing 机制整合在一起的，所以也不需要额外的状态管理。
3. **DFS 原生提供持久化和一致性保证**：如果实现多副本分布式日志，这些都是额外需要考虑的成本。

另一方面，使用 DFS 有以下缺点：

1. **更高的延迟**：DFS 相比于写入本地盘的分布式日志系统来讲一般来说有更高的延迟。
2. **网络 I/O 限制**：大部分 DFS 供应商出于成本的考虑都会对单用户 DFS 写入限流限速，极端情况有可能会造成网络过载。

经过一些初步实验，我们认为目前大部分 DFS 实现（例如 S3，HDFS 等）的性能可以满足 80% 的用例，后面的 Benchmark 会提供更多数据。

### **3.2 DSTL 架构**

下图以 RocksDB 为例展示了基于 DFS 的 DSTL 架构图。状态更新通过 Changelog State Backend 双写，一份写到 RocksDB，另一份写到 DSTL。RocksDB 会定期进行 Materialization，也就是将当前的 SST 文件 上传到 DFS；而 DSTL 会将 state change 持续写入 DFS，并在 Checkpointing 的时候完成 flush，这样 Checkpoint 完成时间只取决于所需 flush 的数据量。需要注意的是 Materialization 完全独立于 Checkpointing 的过程，并且 Materialization 也可以比 Checkpointing 的频率慢很多，系统默认值是 10 分钟。

图4: 以 RocksDB 为例基于 DFS 的 DSTL 架构图

这里还有几个问题值得补充讨论一下：

* **状态清理问题**

  前面有提到在新的架构中，一个 Checkpoint 由两部分组成：1）State Table 和 2）State Change Log。这两部分都需要按需清理。1）这个部分的清理复用 Flink 已有的 Checkpoint 机制；2）这个部分的清理相对较复杂，特别是 State Change Log 在当前的设计中为了避免小文件的问题，是以 TM 为粒度的。在当前的设计中，我们分两个部分来清理 State Change Log：一是 Change Log 本身的数据需要在 State Table 物化后删除其相对应的部分；二是 Change Log 中成为 Checkpoint 的部分的清理融合进已有的 Flink Checkpoint 清理机制[4]。

* **DFS 相关问题**

+ 长尾延迟问题

  为了解决 DFS 高长尾延迟问题，DFS 写入请求会在允许超时时间（默认为 1 秒）内无法完成时重试。
+ 小文件问题

  DFS 的一个问题是每个 Checkpoint 会创建很多小文件，并且因为 Changleog State Backend 可以提供更高频的 Checkpoint，小文件问题会成为瓶颈。为了缓解这种情况，我们将同一个 Task Manager 上同一作业的所有 State Change 写到同一个文件中。因此，同一个 Task Manager 会共享同一个 State Change Log。

**四、Benchmark 测试结果分析**

## 

Generic Log-Based Incremental Checkpointing 对于 Checkpoint 速度和稳定性的提升取决于以下几个因素：

1. State Change Log 增量的部分与全量状态大小之比，增量越小越好。
2. 不间断上传状态增量的能力。这个和状态访问模式相关，极端情况下，如果算子只在 Checkpointing 前更新 Flink State Table 的话，Changelog 起不到太大作用。
3. 能够对来自多个 Task 的 changelog 分组批量上传的能力。Changelog 分组批量写 DFS 可以减少需要创建的文件数量并降低 DFS 负载，从而提高稳定性。
4. 底层 State Backend 在刷磁盘前对同一个 key 的 更新的去重能力。因为 state change log 保存的是状态更新，而不是最终值，底层 State Backend 这种能力会增大 Changelog 增量与 State Table 全量状态大小之比。
5. 写持久存储 DFS 的速度，写的速度越快 Changelog 所带来的提升越不明显。

### **4.1 Benchmark 配置**

在 Benchmark 实验中，我们使用如下配置：

* **算子并行度**：50
* **运行时间**：21h
* **State Backend**：RocksDB (Incremental Checkpoint Enabled)
* **持久存储**：S3 (Presto plugin)
* **机器型号**：AWS m5.xlarge（4 slots per TM）
* **Checkpoint 间隔**: 10ms
* **State Table Materialization 间隔**：3m
* **Input Rate**：50K Events /s

### **4.2 ValueState Workload**

我们第一部分的实验，主要针对每次更新的 Key 值都不一样的负载；这种负载因为上述第 2 点和第 4 点的原因，Changelog 的提升是比较明显的：Checkpoint 完成时间缩短了 10 倍（99.9 pct），Checkpoint 大小增加 30%，恢复时间增加 66% - 225%，如下表所示。

表1: 基于 ValueState Workload 的 Changelog 各项指标对比

下面我们来更详细的看一下 Checkpoint Size 这个部分：

表2: 基于 ValueState Workload 的 Changelog（开启/关闭）的 Checkpoint 相关指标对比

* Checkpointed Data Size 是指在收到 Checkpoint Barrier，Checkpointing 过程开始后上传数据的大小。对于 Changelog 来说，大部分数据在 Checkpointing 过程开始前就已经上传了，所以这就是为什么开启 Changelog 时这个指标要比关闭时小得多的原因。
* Full Checkpoint Data Size 是构成 Checkpoint 的所有文件的总大小，也包括与之前 Checkpoint 共享的文件。与通常的 Checkpoint 相比，Changelog 的格式没有被压缩过也不够紧凑，因此占用更多空间。

### **4.3 Window Workload**

这里使用的是 Sliding Window。如下表所示，Changelog 对 checkpoint 完成时间加速 3 倍左右；但存储放大要高得多（消耗的空间接近 45 倍）：

表3: 基于 Window Workload 的 Changelog（开启/关闭）的 Checkpoint 相关指标对比

Full Checkpoint Data 存储空间放大主要原因来自于：

1. 对于 Sliding Window 算子，每条数据会加到多个滑动窗口中，因此为造成多次更新。Changelog 的写放大问题会更大。
2. 前面有提到，如果底层 State Backend（比如 RocksDB）在刷磁盘前对同一个 key 的 更新去重能力越强，则快照的大小相对于 Changelog 会越小。在 Sliding Window 算子的极端情况下，滑动窗口会因为失效被清理。如果更新和清理发生在同一个 Checkpoint 之内，则很可能该窗口中的数据不包含在快照中。这也意味着清除窗口的速度越快，快照的大小就可能越小。

**五、结论**

## 

Flink 1.15 版本实现了 Generic Log-Based Incremental Checkpointing 的 MVP 版本。这个版本基于 DFS 可以提供秒级左右的 Checkpoint 时间，并极大的提升了 Checkpoint 稳定性，但一定程度上也增加了空间的成本，本质上是用空间换时间。1.16 版本将进一步完善使其生产可用，比如我们可以通过 Local Recovery 和文件缓存来加速恢复时间。另一个方面，Changelog State Backend 接口是通用的，我们可以用同样的接口对接更快的存储来实现更短的延迟，例如 Apache Bookkeeper。除此之外，我们正在研究 Changelog 的其他应用，例如将 Changelog 应用于 Sink 来实现通用的端到端的 exactly-once 等。

**附录**

如果您想试用 Generic Log-Based Incremental Checkpointing 的话，可以在 flink-conf.yaml 中进行如下简单的设置：

```
state.backend.changelog.enabled: true  
state.backend.changelog.storage: filesystem   
dstl.dfs.base-path: <location similar to state.checkpoints.dir>
```

完整的设置文档可以参考 [5]

**致谢**

我们感谢 Stephan Ewen 提出了这个功能的最初设想，也感谢 Piotr Nowojski, Yu Li 和 Yun Tang 的讨论和代码 Review。

[1] [https://mp.weixin.qq.com/s/XbcipgrM8v2lr0\_LdnMTtA](https://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247495906&idx=1&sn=8806069228df05ec3633de7a1e305227&scene=21#wechat_redirect)

[2] https://cwiki.apache.org/confluence/display/FLINK/FLIP-76%3A+Unaligned+Checkpoints

[3] https://cwiki.apache.org/confluence/display/FLINK/FLIP-183%3A+Dynamic+buffer+size+adjustment

[4] https://flink.apache.org/features/2018/01/30/incremental-checkpointing.html

[5] https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/state\_backends/#enabling-changelog

---

**Flink CDC Meetup**

**（视频&PPT）**

关注公众号回复 “**0521**”，或点击 「**阅读原文**」，获取 Flink CDC Meetup 视频 & 演讲PDF～

更多 Flink 相关技术问题，可扫码加入社区钉钉交流群～

**戳我，查看**视频 & 演讲PDF～****