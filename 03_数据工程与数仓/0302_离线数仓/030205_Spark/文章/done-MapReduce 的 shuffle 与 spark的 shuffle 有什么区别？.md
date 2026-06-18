> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_核心知识点/SparkShuffle与Celeborn远程Shuffle边界|SparkShuffle与Celeborn远程Shuffle边界]]、[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_核心知识点/Spark宽窄依赖与Shuffle执行链路|Spark宽窄依赖与Shuffle执行链路]]
---
title: MapReduce 的 shuffle 与 spark的 shuffle 有什么区别？
author: 数据与模型之美
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247486798&idx=1&sn=3b903dc5360b3aaa949e0e4f17ac55c3&chksm=97842db7235a818ce970ff278703ad8652db7f95c11811ae4ae83a33450b31240847ff1ab1b0&mpshare=1&scene=24&srcid=0908b4vmLMRsZ04Lq2fNPTSI&sharer_shareinfo=679c628a031719b2819dfd82b3abb43f&sharer_shareinfo_first=679c628a031719b2819dfd82b3abb43f#rd
---

```
作者：陈乔怀古，资深数据仓库工程师
```

关注公众号：【数据界的老司机】，回复关键字：【资料】，进社群下载全部 word/ppt/pdf 文件。

添加v：cqhg\_bigdata，备注大数据面试，领取对应资料。

送你一张优惠券👇

---

### 核心总结

* **MapReduce Shuffle**：可以看作是 **“标准操作手册”**。它非常稳定，但步骤固定、机械，每一步操作（Map, Spill, Merge, Fetch, Merge, Reduce）都涉及磁盘I/O，非常**沉重（Heavy-weight）**。
* **Spark Shuffle**：更像一个 **“自适应优化引擎”**。它提供了多种实现方式（Hash, Sort, Tungsten-Sort），并会尝试选择最优策略（如避免排序、使用堆外内存等），其核心目标是**最大限度地减少磁盘I/O和网络I/O**，因此更**轻量（Light-weight）** 和灵活。

---

### 详细对比表

| 特性维度 | MapReduce Shuffle | Spark Shuffle | 解释与影响 |
| --- | --- | --- | --- |
| **1. 设计哲学与模型** | **强制排序 (Sort-Based)** | **灵活可选 (Sort-Based / Hash-Based)** | MapReduce 的 Map 端和 Reduce 端都必须对数据进行排序。Spark 提供了两种主要实现，早期默认是 Hash-Based，后来优化为 Sort-Based 作为默认，并可回退到 Hash-Based。 |
| **2. 磁盘I/O** | **重度依赖磁盘** | **尽力优化，减少磁盘I/O** | MapReduce 的每个环节（Spill, Merge, Fetch）都频繁读写磁盘，I/O开销极大。Spark 会优先使用内存，只有当内存不足时才会溢写（Spill）到磁盘。 |
| **3. 内存使用** | **使用固定大小的环形缓冲区** | **动态使用内存，更高效** | MapReduce 的缓冲区大小固定，一满就溢写。Spark 可以更动态地利用JVM堆内和堆外（Tungsten）内存，内存利用率和效率更高。 |
| **4. 流水线化与物化** | **物化 (Materialization)** | **更流水线化 (Pipelined)** | MapReduce 的 Map 任务必须**完全结束后**，Reduce 任务才能开始拉取数据（物化）。Spark 的 Map 任务**还没结束时**，Reduce 任务就可以开始拉取已经溢写完的磁盘文件（流水线），减少了等待时间。 |
| **5. 容错机制** | **通过重新执行任务** | **通过血统（Lineage）和重计算** | MapReduce 的 Shuffle 数据会写入磁盘，因此Task失败后只需重新执行该Task。Spark 默认不会持久化中间Shuffle数据，如果Stage失败，需要根据血统重新计算整个前一个Stage。但Spark也提供了持久化选项。 |
| **6. 实现与可扩展性** | **单一实现** | **可插拔的架构** | MapReduce 的实现是固定的。Spark 的 ShuffleManager 是**可插拔的**（如 `hash`, `sort`, `tungsten-sort`），并且提供了接口允许用户自定义Shuffle实现（如 `Managed`），社区也因此能不断优化（如推出更好的 `SortShuffleManager`）。 |

---

### 流程详解

#### MapReduce Shuffle 流程（沉重但清晰）

1. **Map 端**：

* Map Task 将输出写入一个**环形内存缓冲区**。
* 缓冲区快满时（默认80%），会**溢写 (Spill)** 到磁盘文件。溢写前会对数据进行**分区(Partition)** 和\*\*排序(Sort)\*\*。
* 所有Map输出完成后，会将多个溢写文件**合并(Merge)** 成一个大的、已排序的分区文件。

2. **Copy/Fetch 阶段**：

* Reduce Task 启动**Fetcher线程**，通过HTTP从各个完成Map任务的节点上**拉取(Fetch)** 属于自己的分区数据。

3. **Reduce 端**：

* 先将拉取到的数据放入内存缓冲区，不够时溢写到磁盘。
* 当所有数据都拉取完毕后，会将所有数据（内存和磁盘的）进行一个\*\*归并排序(Merge Sort)\*\*，形成一个更大的、完全有序的数据文件。
* 最后，Reduce Task 处理这个有序文件。

**关键点**：**每一步都有磁盘操作**，非常“踏实”但也非常慢。

#### Spark Shuffle 流程（灵活且优化）

以默认的 `SortShuffleManager` 为例：

1. **Map 端（称为 Shuffle Write）**：

* 数据不会立即写入磁盘。它使用一种**高效的数据结构**（如类似AppendOnlyMap的结构）在内存中进行聚合（如果需要）和排序。
* 当内存压力大时，会将内存中的数据**溢写(Spill)** 到磁盘文件。可能会产生多个溢写文件。
* Map Task 结束时，会将所有**内存中的数据**和**溢写的文件**进行合并，最终为每个分区生成**两个文件**：一个 `.data` 文件（存储合并后的数据）和一个 `.index` 文件（存储索引，记录每个Reduce分区数据的偏移量）。

2. **Reduce 端（称为 Shuffle Read）**：

* Reduce Task 首先会去读取各个Map Task生成的 `.index` 文件，**定位**到需要拉取的数据位置。
* 然后通过网络**拉取(Fetch)** 属于自己的数据块。
* 拉取到的数据可以放入内存中进行聚合操作（如果定义了聚合函数如 `reduceByKey`），Spark会使用高效的内存数据结构（如HashMap）进行运算，内存不足时也会溢写到磁盘。
* 最后将处理好的数据交给后续的计算操作。

**关键点**：**内存优先，避免不必要的排序和磁盘I/O**。`.index` 文件的使用使得数据拉取非常高效，不再是盲目的全量拷贝。

### 总结与类比

* **MapReduce Shuffle** 就像是一个**老式的图书馆**：每本书（数据）必须先被分类、编号、上架（写磁盘）后，读者（Reduce任务）才能根据索书号来借阅。过程规范但耗时。
* **Spark Shuffle** 就像是一个**现代化的智能仓库**：货物（数据）进来后，机器人会尽量在移动中（内存中）就进行分拣和预处理。同时，它维护了一个精确的电子索引（`.index`文件）。取货员（Reduce任务）根据电子索引直接开车到货架前取走需要的货物，效率极高。

正是因为 Shuffle 阶段的巨大优化，以及内存计算的广泛使用，Spark 在迭代计算（如机器学习算法）和交互式查询上的性能通常远超 MapReduce。

---

据统计，99%的大咖都关注了这个公众号👇

猜你喜欢👇

1. [Doris vs StarRocks vs ClickHouse：新一代MPP引擎的终极对决](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247484607&idx=1&sn=6d27e96ad8a7498833372cc51a79a557&scene=21#wechat_redirect)
2. [性能直接炸了！玩转 Hive 分区与分桶，查询效率轻松翻数十倍！](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247486576&idx=1&sn=1d2120d073e6323fa34de7e363e2d29b&scene=21#wechat_redirect)
3. [Kafka架构深度拆解：从生产者到消费者，一文讲透所有组件](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247486573&idx=2&sn=22f7c6eee7c95b17210be578e2c787fd&scene=21#wechat_redirect)
4. [同样处理大数据，Spark究竟比MapReduce快在哪？](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247486506&idx=1&sn=b160abc27a9c1d50388ca139a2f5acae&scene=21#wechat_redirect)
5. [Hadoop面试逆袭指南：从底层HDFS到调度YARN，硬核详解高频真题，告别一问就懵！(建议收藏)](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247486534&idx=1&sn=7fa89a2435fc2d722c55a5404ece6284&scene=21#wechat_redirect)
6. [面试官逼问Shuffle细节怎么办？这篇终极指南让你对答如流，倒背如流！](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247486452&idx=1&sn=9a141e55d2068db7140997573a2ce5ce&scene=21#wechat_redirect)
7. [一次讲透：MapReduce为什么一定要分成Map和Reduce？](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247486445&idx=1&sn=02a45d142cb704459cdc7a695f0339cd&scene=21#wechat_redirect)

**扫码加入VIP社群🪐 所有资料都可以直接下载**

**⏬**