> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL执行前置优化|SQL执行前置优化]]
---
title: SQL任务优化思路之map端长尾
author: 大数据技术与数仓
date:
url: http://mp.weixin.qq.com/s?__biz=MzU2ODQ3NjYyMA==&mid=2247488801&idx=1&sn=0af1052b4e0a6c73b3c5b6b05b70cc09&chksm=fc8c0382cbfb8a941a7820b8c629d10bc1ec6a1678dc4ebc0f24f6f542b2b6b09da89f5a4284&mpshare=1&scene=24&srcid=0318br7B6LASfiqCWiNE6ViT&sharer_shareinfo=09e8500d091bed2943b07b1435de80c8&sharer_shareinfo_first=09e8500d091bed2943b07b1435de80c8#rd
---

MapReduce将复杂的、运行于大规模集群上的并行计算过程高度地抽象到了两个函数：Map和Reduce。MapReduce采用“**分而治之**”策略，一个存储在分布式文件系统中的大规模数据集，会被切分成许多独立的分片（split），这些分片可以被多个Map任务并行处理。本文将介绍map端长尾的优化思路，通过本文你可以了解到：

* Mapper 是如何工作的
* Map端长尾产生的原因
* Map端长尾的优化思路

感谢关注，希望本文对你有所帮助。

## Mapper 是如何工作的

### MapReduce的工作流程

img

* 1）MapTask收集我们的map()方法输出的kv对，放到内存缓冲区中
* 2）从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件
* 3）多个溢出文件会被合并成大的溢出文件
* 4）在溢出过程及合并的过程中，都要调用Partitioner进行分区和针对key进行排序
* 5）ReduceTask根据自己的分区号，去各个MapTask机器上取相应的结果分区数据
* 6）ReduceTask会取到同一个分区的来自不同MapTask的结果文件，ReduceTask会将这些文件再进行合并（归并排序）
* 7）合并成大文件后，Shuffle的过程也就结束了，后面进入ReduceTask的逻辑运算过程（从文件中取出一个一个的键值对Group，调用用户自定义的reduce()方法）

注意:Shuffle中的缓冲区大小会影响到MapReduce程序的执行效率，原则上说，缓冲区越大，磁盘io的次数越少，执行速度就越快。缓冲区的大小可以通过参数调整，参数：mapreduce.task.io.sort.mb默认100M。

### Mapper端shuffle

img

* 每个Map任务分配一个缓存(MapReduce默认100MB缓存,设置溢写比例默认0.8)
* 分区默认采用哈希函数
* 排序是默认的操作
* 排序后可以合并（Combine）
* 合并不能改变最终结果
* 在Map任务全部结束之前进行归并
* 归并得到一个大的文件，放在本地磁盘
* 文件归并时，如果溢写文件数量大于预定值（默认是3）则可以再次启动Combiner，少于3不需要

> 合并（Combine）和归并（Merge）的区别：两个键值对<“a”,1>和<“a”,1>，如果合并，会得到<“a”,2>，如果归并，会得到<“a”,<1,1>>

#### 关于Split（分片）

img

HDFS 以固定大小的block 为基本单位存储数据，而对于MapReduce 而言，其处理单位是split。split 是一个逻辑概念，它只包含一些元数据信息，比如数据起始位置、数据长度、数据所在节点等。它的划分方法完全由用户自己决定。

#### **Map任务的数量**

Hadoop为每个split创建一个Map任务，split 的多少决定了Map任务的数目。大多数情况下，理想的分片大小是一个HDFS块。

#### 环形内存缓冲区

img

每个 maptask 的写出一般会导致环形缓冲区多次溢写，从而产生多个溢写文件，这些溢写文件均已经分好区排好序了，由于 Reduce 阶段是接收一整个文件作为输入，所以这多个溢写文件还要再进行一次合并，这次的合并采用的是归并排序的算法。

img

这个机制是为了优化Map任务的输出过程，减少磁盘I/O操作，从而提高MapReduce作业的效率。

环形缓冲区（也称为spill缓冲区）是Map任务中的内存结构，用于临时存储Map输出的键值对（intermediate data）。当环形缓冲区达到一定的阈值（默认为80%），MapReduce框架会触发溢写（spill）操作，将缓冲区中的数据写到磁盘上的临时文件中。在溢写前，数据会先被排序（如果设置了Combiner，也会在这一步进行局部聚合），然后才写入磁盘。

> 环形缓冲区的大小和溢写阈值可以通过MapReduce作业的配置参数进行调整，例如：
>
> * mapreduce.task.io.sort.mb：设置环形缓冲区的大小，默认是100MB。
> * mapreduce.map.sort.spill.percent：设置触发溢写的阈值，默认是0.80（即80%）。

**此外，Map任务结束后，磁盘上的所有溢写文件会被合并，以准备进行后续的Reduce操作。对于较小的作业，或者当Map输出的数据量不大时，可能整个Map输出都可以在内存中处理，不需要溢写到磁盘。**

正确地配置这些参数对于整个MapReduce作业的性能至关重要。如果环形缓冲区太小，将导致频繁的溢写操作。如果太大，可能会导致内存资源浪费，甚至影响到其他并行运行的任务。性能调优通常需要依据作业的特点和集群的资源状况来决定合适的参数设置。

## Map端长尾产生的原因及优化思路

Map端是MR任务的起始阶段，Map端的主要功能是从磁盘中将数据读入内存，由于读入数据的文件大小分布不均匀，会导致一些Map Instance读取并且处理的数据特别多，而有些Map Instance处理的数据特别少，造成Map端长尾。一般情况下，**Map端很少遇到长尾**，但以下两种情况可能会导致Map端长尾，以ODPS为例：

* **原因一：**上游表文件的大小特别不均匀，并且小文件特别多，导致当前表Map端读取的数据分布不均匀，引起长尾

  > 针对该情况导致的Map端长尾，可通过对上游进行合并小文件，同时**调节本节点的小文件的参数**来进行优化，即通过设置`set odps.sql.mapper.merge.limit.size=64`和`set odps.sql.mapper.split.size=256`两个参数来调节。
* **原因二：** Map端做聚合时由于某些Map Instance读取文件的某个值特别多而引起长尾，主要是指Count Distinct操作

  > 解决办法：数据分布不均匀造成的Map端长尾时，是通过distribute by rand()的方式来打乱数据分布，从而平衡Map端每个Instance处理的数据量。但是，Map端的长尾问题是解决了，与此同时带来了Reduce端的资源紧张。

## 总结

在Hadoop MapReduce中，Map任务从磁盘读取数据并执行转换，结果存储在环形缓冲区内。当缓冲区满时，数据溢写到磁盘。Map端长尾是指部分Map任务因数据倾斜或资源限制而执行缓慢，延长整体作业时间。优化策略包括合理分配数据、调整缓冲区大小、合并小文件、优化硬件资源。通过设置参数mapreduce.task.io.sort.mb和mapreduce.map.sort.spill.percent可以调整缓冲区行为。解决数据倾斜的一种方法是使用distribute by rand()使数据分布更均匀，但可能会导致Reduce端压力增大。