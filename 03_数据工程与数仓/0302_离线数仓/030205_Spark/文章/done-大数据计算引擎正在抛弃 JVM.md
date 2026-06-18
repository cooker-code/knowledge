---
title: 大数据计算引擎正在抛弃 JVM
author: 种桃者说
date:
url: https://mp.weixin.qq.com/s?__biz=MzU2NzA3OTEwMg==&mid=2247484339&idx=1&sn=64ea5c1f50ff2ece1d9c9cc1ab2d8be6&chksm=fdc4a1d5cabfd8d8a0777854be8cf66ec00c0a5d9f9335118412caf4adb4e273a0b1b958900c&mpshare=1&scene=24&srcid=1103GjYqljYyrDU1Zh674TO4&sharer_shareinfo=01b7fab6c85a03e0adbcf1b47be97ff0&sharer_shareinfo_first=01b7fab6c85a03e0adbcf1b47be97ff0#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_知识地图|知识地图]]


在写这篇文章之前，Java 25正式发布，其中 JEP-508 Vector API 迎来了第10次孵化，旨在提供一种向量计算的接口，从而获得比等效标量计算更高的性能。传统的基于Java虚拟机（JVM）的执行引擎在处理大规模数据时逐渐显露出性能瓶颈 (标量计算) ，特别是在 CPU 密集型任务和内存管理方面。近年来，众多大数据计算引擎开始转向原生（Native）执行模型，采用 C++ 等语言实现向量化执行，以提升性能和适应现代硬件特性。 Databricks 团队于 2022 年在 SIGMOD 会议上发表的论文《Photon: A Fast Query Engine for Lakehouse Systems》中表明向量化查询引擎 Photon 有数量级的性能提升(3~10倍的性能提升)，并在 100TB TPC-DS 基准测试中创下新的经审计性能记录。

本文将从 JVM 在处理大规模数据场景下的局限性说起，探讨 Photon 在此基础上的设计选择，并分析目前业界的基于 Gluten + Velox 和 Native 向量化引擎的两种方式。

---

一. JVM的局限性

在该论文中，Databricks 团队指出，放弃现有基于 JVM 的引擎，是基于观察到当前引擎的工作负载正变得受限于 CPU，改进现有引擎的性能变得越来越困难。几个因素导致了这一点。首先，本地NVMe SSD 缓存和自动优化 shuffle 等低级优化显著减少了 I/O 延迟；其次，Delta Lake 支持的数据聚类等技术通过文件修剪更积极地跳过不需要的数据，进一步减少 I/O 等待时间。最后，湖仓一体引入了需要对非规范化数据、大型字符串和非结构化嵌套数据类型进行繁重处理的新工作负载，这进一步加剧了内存性能的压力。 另一个原因是 JVM 内部即时编译器的限制（例如方法大小限制）导致性能急剧下降。此外，本地代码的性能通常比 JVM 引擎更容易解释，因为内存管理和 SIMD 等特性可被明确控制。

---

二. Photon 

Photon 是 Databricks 为湖仓一体环境开发的新型向量化查询引擎，以下是 Photon 转向原生 Native 执行的关键原因和实现方式：

* 原生 C++ 实现：Photon 选择用 C++ 实现，而不是沿用基于 JVM 的Databricks Runtime（DBR）。原生代码避免了 JVM 的性能瓶颈，如JIT编译器的限制和垃圾回收问题。Photon 通过显式控制内存管理和SIMD指令，显著提升了连接、聚合和表达式评估的性能。
* 向量化执行模型：Photon 采用解释型向量化执行，而非 Spark SQL 的代码生成模型。向量化通过批处理数据分摊函数调用开销，利用 SIMD 提高性能。
* 内存管理优化：Photon 通过内部缓冲池管理内存分配，避免昂贵的操作系统级分配。对于持久性分配（如聚合或连接），Photon 与 Spark 的统一内存管理器集成，支持动态溢出机制。这种灵活性在处理湖仓一体中常见的超大记录时尤为重要。
* 与 Spark 的兼容性：尽管转向原生执行，Photon 通过 JNI 与 Spark 集成，支持部分查询在 Photon 和 Spark SQL 之间切换，确保语义一致性。这种增量部署策略降低了迁移成本，同时保留了 Spark 生态系统的优势。

Photon 的成功验证了原生向量化引擎在湖仓一体环境中的优越性，其在 100TB TPC-DS 基准测试中的世界纪录进一步证明了其性能优势。

---

三. Gluten 和 Velox

Apache Gluten 是由 Intel 和 Kyligence 发起的一个中间层组件，它的主要职责在于将基于 JVM 的SQL 引擎的执行任务卸载到原生 Native 引擎上进行处理，以此显著提升数据处理速度并降低资源消耗。如下图所示，Gluten 作为中间层，上游对接 Spark 或者其他大数据计算框架，下游执行层则对接 Velox，Clickhouse 这类本地高性能计算引擎。

Velox 是 Meta 开源的一款 C++ 实现的向量化执行引擎，简单说就是一个单机/单节点的 C++ 的向量化 runtime 模块的实现，里面包括了数据类型，函数，表达式，aggregate function，operator，I/O等的向量化实现，用于替换 Spark/Presto 的 runtime 部分，使得这类计算引擎从 JVM 切换到 C++ 实现得以提速。

Gluten+Velox 的组合，让Spark/Presto 也可以像等Native引擎一样发挥向量化执行的性能优势。

---

四. Spark/Flink向量化

**Spark向量化**

目前，国内外业界主流各大互联网公司对 Spark 多通过 JNI 的方式直接在大数据量的情况下以 Gluten+Velox 的形式进行 native 算子库加速。比如英特尔公司推出的Gluten，通过 Fallback 机制，即当查询计划不能执行，或者有程序崩溃时，也能保证任务执行。因为项目初期功能表现欠佳，现阶段与 Spark JVM 协作，当有算子或是功能支持失败时，就会回退到 JVM 执行，以保证查询计划的执行成功。

**Flink向量化引擎-Flash**

阿里云也推出了向量化版本 Flink 引擎-- Flash，其中性能数据显示，相较于开源的 Flink 版本，Flash 引擎性能提升了5到10倍。如下图所示，Flash 通过中间一层 Leno 胶水层，它类似于 Spark 中的 Gluten，主要负责将流式 Native Runtime 与 Flink 的分布式框架解耦。这样，在之前的 Java 算子版本上，可以独立发布 Native 算子。Leno 胶水层的任务是生成 Native 的执行计划，即根据用户的 SQL 需求，通过 Flink Planner 判断 SQL 语句中算子是否全部被覆盖。如果全部覆盖，就生成完整的 C++ 向量化执行计划；如果不行，则回退到 Java 的执行计划。

---

五. Native 向量化引擎

业界也有一些成熟的 Native 向量化引擎，如 StarRocks。StarRocks 通过实现全面向量化引擎，充分发挥了 CPU 的处理能力。全面向量化引擎按照列式的方式组织和处理数据。StarRocks 的数据存储、内存中数据的组织方式，以及 SQL 算子的计算方式，都是列式实现的。按列的数据组织也会更加充分的利用 CPU 的 Cache，按列计算会有更少的虚函数调用以及更少的分支判断从而获得更加充分的 CPU 指令流水。

另一方面，StarRocks 的全面向量化引擎通过向量化算法充分的利用 CPU 提供的 SIMD（Single Instruction Multiple Data）指令。这样 StarRocks 可以用更少的指令数目，完成更多的数据操作。经过标准测试集的验证，StarRocks 的全面向量化引擎可以将执行算子的性能，整体提升 3~10 倍。

StarRocks BE 端完全用 C++ 代码实现，只有在涉及到一些外表 Paimon/Hive, 会通过 JNI 的方式进行交互，这也侧面反映 Java 生态的强大。

---

六. 未来和挑战

C++ 的复杂性和 Java 强大的生态：过去十几年的绝大部份大数据框架都是基于 Java 语言，JVM 生态系统在大数据领域根深蒂固，原生引擎需要通过 JNI 与现有周边工具集成，增加了开发复杂性。

大数据计算引擎可能继续向混合模型演进。例如，Spark 和 Flink 可能在保留 JVM 生态的同时，逐步将关键算子迁移到原生实现。而 Native 原生引擎可能成为高性能 OLAP 和湖仓一体场景的主流选择。此外，随着硬件加速器（如GPU和TPU）的普及，引擎可能进一步优化以利用这些新型计算资源得到提速。