> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkRocksDB深度调优|FlinkRocksDB深度调优]]、[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink内存模型|Flink内存模型]]、[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink大状态优化实战|Flink大状态优化实战]]
---
title: 腾讯面试：Flink100G大状态如何优化？有哪些参数可以调整？
author: 大数据技能圈
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491532&idx=1&sn=18c1952668c000328b2fa3081fc80906&chksm=c1eb64beb2aada7bc3d1a76b7c28b80376e6f2cde17ac08290574c3c184d7721d27b8507602f&mpshare=1&scene=24&srcid=07022PklxW9uizsDjLkorcp9&sharer_shareinfo=fe0dde88eab4cfb2ca9be1f7c4d22196&sharer_shareinfo_first=23efdaf6fa81f4f0f46ce5dd961b62ea#rd
---

01

**加入知识星球《随川陪你学大数据》将获得以下权益**

扫描二维码加入星球，随加入人数增加，逐步涨价，现在就是最优惠的

01

**Flink100G级别大状态调优指南**

## 一、引言：100G状态的挑战与优化框架

在实时数据处理领域，当Flink作业状态达到100G级别时，将面临一系列独特的技术挑战。这类作业通常出现在用户行为分析、实时推荐、会话窗口聚合等场景，其共同特征是需要维护大量历史数据或复杂数据结构。本文将系统阐述100G级状态作业的完整调优方案，从状态后端选型、RocksDB深度配置、Checkpoint策略优化到内存管理，构建一套可落地的性能优化体系。

### 1.1 100G状态的典型特征

100G级状态作业通常具有以下特点：

* **状态增长稳定**

  ：日均增量5-10G，需关注长期存储成本
* **读写混合负载**

  ：既有高频状态更新（如计数器），也有复杂查询（如TopN聚合）
* **亚秒级延迟要求**

  ：端到端延迟通常要求在500ms以内
* **高可用性需求**

  ：Checkpoint成功率需保持99.9%以上

### 1.2 优化框架概览

针对100G级状态作业，我们采用"四维优化框架"：

1. **存储层优化**

   ：RocksDB参数调优与磁盘I/O配置
2. **一致性层优化**

   ：Checkpoint策略与状态持久化方案
3. **计算层优化**

   ：并行度设计与数据倾斜处理
4. **资源层优化**

   ：内存分配与JVM参数调优

## 二、存储层优化：RocksDB深度配置

### 2.1 状态后端选型决策

对于100G级状态，RocksDBStateBackend是唯一可行选择，其核心优势在于：

* **增量Checkpoint**

  ：仅上传变更数据，减少网络传输
* **磁盘友好的存储结构**

  ：基于LSM树的分层存储，适合顺序写入
* **内存与磁盘平衡**

  ：通过Block Cache缓存热点数据，平衡内存占用与I/O效率

基础配置示例：

```
```
state.backend: rocksdb
state.backend.incremental:true
state.checkpoints.dir: hdfs:///flink/checkpoints
```
```

### 2.2 RocksDB内存配置精要

RocksDB的内存配置直接决定了100G状态的访问性能，需要精细平衡以下参数：

#### 2.2.1 Block Cache优化

Block Cache用于缓存从磁盘读取的数据块，推荐配置为TaskManager内存的15-20%：

```
```
state.backend.rocksdb.block.cache-size:134217728# 128MB，当TaskManager总内存为8GB时
```
```

**优化原理**：100G状态下，Block Cache设置过小会导致频繁磁盘I/O，设置过大则会挤占JVM内存。通过监控`rocksdb.block.cache.hit.rate`指标（目标>0.85）动态调整，确保热点数据缓存命中率。

#### 2.2.2 Write Buffer配置

Write Buffer（MemTable）是内存中的写入缓冲区，合理配置可减少刷盘次数：

```
```
state.backend.rocksdb.writebuffer.size:67108864# 64MB
state.backend.rocksdb.writebuffer.count:4# 最大4个memtable
state.backend.rocksdb.writebuffer.number-to-merge:2# 合并2个memtable后刷盘
```
```

**实践经验**：对于100G状态，单个memtable设置为64MB，配合4个memtable（总256MB）可有效平衡写入性能与恢复速度。合并2个memtable刷盘可减少小文件数量，降低后续Compaction压力。

### 2.3 Compaction策略调优

Compaction是RocksDB的核心机制，直接影响读性能和磁盘空间利用率。100G状态推荐使用LEVEL Compaction策略：

```
```
state.backend.rocksdb.compaction.style: LEVEL
state.backend.rocksdb.compaction.level.target-file-size-base:67108864# 64MB
state.backend.rocksdb.compaction.level.max-size-level-base:536870912# 512MB（L1层总大小）
state.backend.rocksdb.thread.num.compaction:4# Compaction线程数
```
```

**调优要点**：

* L1层单个文件64MB，总大小512MB，使各层级数据量呈指数增长
* Compaction线程数设置为CPU核心数的50%，避免资源竞争
* 监控`rocksdb.compaction.bytes.per.second`，确保Compaction速度大于写入速度

### 2.4 多磁盘I/O优化

100G状态下，磁盘I/O容易成为瓶颈，通过多磁盘配置分散压力：

```
```
state.backend.rocksdb.localdir: /data1/rocksdb,/data2/rocksdb,/data3/rocksdb,/data4/rocksdb
```
```

**实施建议**：

* 使用4块独立SSD磁盘，每块磁盘对应一个RocksDB实例目录
* 避免使用RAID，直接让Flink管理多磁盘分布
* 监控各磁盘I/O利用率，确保负载均衡（差异<20%）

## 三、一致性层优化：Checkpoint策略设计

### 3.1 Checkpoint基础参数配置

100G状态下，Checkpoint策略需要在数据安全性与性能之间取得平衡：

```
```
execution.checkpointing.interval:600000# 10分钟
execution.checkpointing.timeout:1200000# 20分钟超时
execution.checkpointing.min-pause-between-checkpoints:300000# 5分钟最小间隔
execution.checkpointing.max-concurrent-checkpoints:1# 禁止并发Checkpoint
```
```

**参数协同关系**：

* 间隔设置为10分钟，确保每天仅144次Checkpoint，减少资源消耗
* 超时时间为间隔的2倍，给予足够时间完成状态上传
* 最小间隔设置为间隔的50%，避免Checkpoint过于密集

### 3.2 非对齐Checkpoint应用

在存在反压的场景下，启用非对齐Checkpoint可显著降低Checkpoint耗时：

```
```
execution.checkpointing.unaligned:true
execution.checkpointing.buffer-debloating.enabled:true# 启用Buffer去膨胀
```
```

**适用场景**：

* 当Checkpoint对齐时间超过总耗时的30%时启用
* 配合`buffer-debloating`可减少缓冲数据量，避免状态膨胀
* 实测在100G状态、中度反压场景下，可将Checkpoint耗时从15分钟降至8分钟

### 3.3 本地恢复配置

启用本地恢复可大幅提升故障恢复速度：

```
```
state.backend.local-recovery:true
state.backend.rocksdb.localdir: /data/rocksdb/local  # 本地恢复目录
```
```

**恢复流程优化**：

1. 优先从本地磁盘恢复状态元数据
2. 仅从远端下载缺失的SST文件
3. 恢复速度提升约60%，100G状态恢复时间从25分钟缩短至10分钟

## 四、计算层优化：并行度与数据分布

### 4.1 并行度设计原则

100G状态作业的并行度设计需遵循"状态均分"原则：

```
```
parallelism.default:32# 总并行度
state.backend.max-parallelism:1024# KeyGroup数量
```
```

**计算资源配置**：

* 每并行实例处理约3-4G状态（100G/32≈3.125G）
* KeyGroup数量设置为并行度的32倍，确保重分区时负载均衡
* 每个TaskManager配置4-8个slot，避免单个节点状态过大

### 4.2 数据倾斜治理

即使在100G中等规模状态下，数据倾斜仍可能导致局部节点过载：

#### 4.2.1 倾斜检测

通过Flink Web UI监控以下指标识别倾斜：

* Subtask级别的`numRecordsInPerSecond`差异超过3倍
* 特定Subtask的`stateSize`显著大于其他节点
* 倾斜节点的`backpressure`指标持续为HIGH

#### 4.2.2 两阶段聚合解决方案

实施两阶段聚合打散热点Key：

```
```
// 第一阶段：随机加盐
DataStream<Tuple2<String,Long>> saltedStream = input
.map(record->{
String key =record.f0;
// 随机添加1-16的盐值
String saltedKey = key +"#"+newRandom().nextInt(16);
returnTuple2.of(saltedKey,record.f1);
})
.keyBy(0)
.window(TumblingEventTimeWindows.of(Time.minutes(5)))
.aggregate(newCountAggregator());

// 第二阶段：去盐聚合
DataStream<Tuple2<String,Long>> result = saltedStream
.map(tuple ->{
String originalKey = tuple.f0.split("#")[0];
returnTuple2.of(originalKey, tuple.f1);
})
.keyBy(0)
.reduce((a, b)->Tuple2.of(a.f0, a.f1 + b.f1));
```
```

## 五、资源层优化：内存与JVM配置

### 5.1 内存分配策略

100G状态作业的内存配置需要精细规划各区域占比：

```
```
taskmanager.memory.process.size: 16g  # 总进程内存
taskmanager.memory.heap.size: 6g  # JVM堆内存
taskmanager.memory.managed.fraction:0.4# 托管内存占比
```
```

**内存分配明细**：

* **JVM堆内存**

  ：6G，用于用户代码和Flink框架
* **托管内存**

  ：6.4G（16G×0.4），分配给RocksDB
* **网络内存**

  ：1.6G（10%），用于数据传输
* **JVM元空间**

  ：512M，用于类加载
* **剩余内存**

  ：1.48G，用于操作系统和其他开销

### 5.2 JVM参数调优

针对100G状态作业，优化JVM参数避免GC问题：

```
```
env.java.opts:>-
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:ParallelGCThreads=4
-XX:ConcGCThreads=2
-XX:NewRatio=3
-XX:MetaspaceSize=512m
-XX:MaxMetaspaceSize=512m
-Xloggc:/opt/flink/logs/gc.log
```
```

**G1GC调优要点**：

* 设置最大停顿时间200ms，平衡延迟与吞吐量
* NewRatio=3表示老年代:新生代=3:1，减少年轻代GC次数
* 禁用显式GC，避免Checkpoint时触发Full GC
* 监控GC日志，确保Full GC间隔>1小时，单次Full GC时间<1秒

## 六、调优决策指南与最佳实践

### 6.1 参数调优决策树

针对100G级状态作业，建议按照以下优先级进行调优：

1. **磁盘I/O优化**

* 确认使用SSD磁盘
* 配置多磁盘目录分散I/O
* 监控磁盘利用率，目标<70%

2. **内存配置**

* 托管内存占比40-50%
* Block Cache设置为总内存的15-20%
* 监控RocksDB内存使用，避免OOM

3. **Checkpoint策略**

* 启用增量Checkpoint和本地恢复
* 非对齐Checkpoint用于反压场景
* 间隔设置为5-15分钟，根据SLA调整

4. **Compaction优化**

* LEVEL策略用于读密集型，UNIVERSAL用于写密集型
* Compaction线程数=CPU核心数/2
* 监控Compaction速度，确保不滞后于写入速度

### 6.2 关键监控指标体系

建立以下监控看板，实时跟踪100G状态作业健康度：

#### 状态健康度看板

* 总状态大小及增长率
* 各Subtask状态分布均匀性
* 状态TTL命中率

#### RocksDB性能看板

* Block Cache命中率（目标>85%）
* Compaction吞吐量（MB/秒）
* Memtable刷写频率

#### Checkpoint看板

* Checkpoint成功率
* 同步/异步阶段耗时占比
* Checkpoint数据量（全量/增量）

### 6.3 常见问题诊断与解决方案

| 问题现象 | 可能原因 | 解决方案 |
| --- | --- | --- |
| Checkpoint超时 | 状态数据量大、网络带宽不足 | 启用增量Checkpoint、增加网络缓冲 |
| 读延迟高 | Block Cache命中率低 | 增大block\_cache\_size、优化block\_size |
| Compaction滞后 | 写入速度超过Compaction速度 | 增加Compaction线程、调整Compaction策略 |
| 数据倾斜 | Key分布不均 | 两阶段聚合、动态负载均衡 |
| GC频繁 | 堆内存设置不合理 | 调整NewRatio、增大堆内存 |

##

以上。

本次优化总结详细文档已放入知识星球《随川陪你学大数据》，可扫码获取

获取更多信息，关注大数据技能圈

**欢迎添加作者交流**

## 推荐阅读系列文章

* [腾讯面试：Spark内存如何优化？包含哪几个方面？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491167&idx=1&sn=d62e305f6803f484aa40469393b1fdb8&scene=21#wechat_redirect)
* [超强总结：Iceberg可以优化的方方面面](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491505&idx=1&sn=e385a35b8d736b2cc639225b8487563c&scene=21#wechat_redirect)
* [超强总结：StarRocks可以优化的方方面面](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491472&idx=1&sn=a541fb0aedd682f0a9b8b38b082655f5&scene=21#wechat_redirect)
* [超强总结：Doris可以优化的方方面面](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491458&idx=1&sn=74f33d808ce0ce732fcc59704dc786af&scene=21#wechat_redirect)
* [超强总结：Paimon可以优化的方方面面](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491418&idx=1&sn=ebb818ede7585037fce758680d4bc5bf&scene=21#wechat_redirect)
* [大数据计算引擎（Hadoop,Spark,Flink）发展史](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491412&idx=1&sn=0db91ec54d5760b004a4f877f83a133f&scene=21#wechat_redirect)
* [腾讯面试：Doris 物化视图的使用场景是怎么样的，有哪几种数据更新方式？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491386&idx=1&sn=d29b14d6912d0efdb941ae5e1dbd1dfb&scene=21#wechat_redirect)
* [腾讯面试：详细介绍Spark的Shuffle阶段数据从输入到输出经历了哪些步骤？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491384&idx=1&sn=5ab414f60fda52a3b9b88e1a7f2cdecd&scene=21#wechat_redirect)
* [腾讯面试：数仓分层架构是怎么样的？为什么要这样设计？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491372&idx=1&sn=97dfb7d58f8b889a8cbaeb4229091e98&scene=21#wechat_redirect)
* [蚂蚁面试：Flink并行度、算子、算子链、Slot、Slot共享组之间的关系是什么？如何设置能够使资源利用最大化？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491363&idx=1&sn=17d02a075e33592aabc517f974e87a02&scene=21#wechat_redirect)
* [网易面试：Hudi、Iceberg、Paimon有什么异同点？如何选型？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491351&idx=1&sn=df26d44b5c9ac16eb3be5a2f3027d5f6&scene=21#wechat_redirect)
* [Flink 反压问题深度剖析与解决方案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491343&idx=1&sn=a898b2172b5a694dfa03e356758ff08a&scene=21#wechat_redirect)
* [小米面试：Paimon Join用法有哪些？大规模数据场景下如何优化 Join 性能？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491218&idx=1&sn=b5aa336a2fb6a37d1093ec6318e8e39d&scene=21#wechat_redirect)
* [蚂蚁面试：Kafka如何做压测？如何保证系统稳定？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491146&idx=1&sn=e2c4a1e298aec2b6d4d42cffb2f331ef&scene=21#wechat_redirect)
* [字节面试：Flink如何做压测？如何保证系统稳定？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491131&idx=1&sn=6bac85b9e17620b49b37ebd3224b3a83&scene=21#wechat_redirect)
* [Flink内存调优指南（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=2&sn=cccaa1289c4581a2dd407d21a8c5ae09&scene=21#wechat_redirect)附500页16万字答案[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=2&sn=cccaa1289c4581a2dd407d21a8c5ae09&scene=21#wechat_redirect)
* [Zookeeper 经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[附1400页21万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)
* [Hbase 经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491195&idx=1&sn=0c93336f246fde472c336ce764623a75&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491195&idx=1&sn=0c93336f246fde472c336ce764623a75&scene=21#wechat_redirect)[附550页12万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491195&idx=1&sn=0c93336f246fde472c336ce764623a75&scene=21#wechat_redirect)
* [Hive经典面试题200道（附8万字420页答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491138&idx=1&sn=f1acb3164d5f0f7e8be7f2037f4a838b&scene=21#wechat_redirect)
* [Kafka 经典面试题200道（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)[附8万字420页答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491138&idx=1&sn=f1acb3164d5f0f7e8be7f2037f4a838b&scene=21&token=1460012704&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)
* [Spark经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)[附500页8万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491106&idx=1&sn=d412a81e6d5742c3e4acf6a680761139&scene=21&token=1460012704&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)
* [ElasticSearch经典面试题200道（附400页12万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491122&idx=1&sn=cb7f3563a8082d248b1a0b768bd4d567&scene=21#wechat_redirect)
* [FlinkCDC经典面试题200道（附500页8万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491106&idx=1&sn=d412a81e6d5742c3e4acf6a680761139&scene=21#wechat_redirect)
* [StarRocks](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect) [经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect)[附550页12万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect)
* [Flink源码分析 经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[附1200页32万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)
* [FlinkSQL 经典面试题200道（附1200页32万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21#wechat_redirect)
* [Paimon经典面试题200道题（附500页16万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491044&idx=1&sn=f11db4583012987748f532d11ebaf1f0&scene=21#wechat_redirect)
* [Hudi经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491340&idx=1&sn=fbfde6a8f265acaa2eb3cf320fc635fc&scene=21#wechat_redirect)[200道（附1050页39万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491037&idx=1&sn=24bcca607e01daae65783775dcc3716e&scene=21#wechat_redirect)
* [Doris经典面试题200道（附1050页39万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491037&idx=1&sn=24bcca607e01daae65783775dcc3716e&scene=21#wechat_redirect)
* [Flink经典面试题200道（附1060页26万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491335&idx=1&sn=97f9e76a359119d51ff26fd177140437&scene=21#wechat_redirect)
* [建议收藏 | Kafka 实战文章总结](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490811&idx=1&sn=8e756190799e98e0e017b5994b6d0c3b&scene=21#wechat_redirect)
* [建议收藏 | Dinky系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489859&idx=1&sn=6b59becc16f653b609d01090e40f9e20&chksm=c02962dcf75eebcabfa43a1e3e4a1b210df6ac0725685a9087984261626223cc339c0c363a24&scene=21#wechat_redirect)
* [建议收藏 | Flink系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489608&idx=1&sn=c055844c7ac20eabbe11e331f0d629c2&chksm=c02963d7f75eeac15892c32660c1e90e000ad72e66ab97ed4051273bb5e3f6299998d6c81097&scene=21#wechat_redirect)
* [建议收藏 | Flink CDC 系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489988&idx=1&sn=c57b867645834257e8567a6e671273ff&chksm=c029625bf75eeb4d29d3f43d1b845aec3a7d176cfbceddd16ae6d195d4d96fa5cf6bfa9c362d&scene=21#wechat_redirect)
* 建议收藏 | [Doris实战文章合集](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490251&idx=1&sn=ae993d282a0a3cd968c3d6a19f79dce8&chksm=c0296154f75ee8428ee77a8e0c86ca08648967e7a68987aaf79d20bdbd481f0ab6d1d2729b53&scene=21#wechat_redirect)
* [建议收藏 | Paimon 实战文章总结](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490466&idx=1&sn=c3bdd1c25d72ba89186d4ed63634f7b3&scene=21#wechat_redirect)
* [建议收藏 | Fluss 实战文章总结](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490616&idx=1&sn=20c60ca77763b23d7c5b3a8785f58007&scene=21#wechat_redirect)
* 建议收藏 | [Seatunnel 实战文章系列合集](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490357&idx=1&sn=61ae7c852b2a41e192c0797cc28fd182&chksm=c02960aaf75ee9bcd422d82ceb80362a0959df74ee7d336e0015c51b3eb6eb1a6d6774e99483&scene=21#wechat_redirect)
* 建议收藏 | [实时离线输数仓（数据湖）总结篇](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488973&idx=1&sn=42d2f8235c822030f187c0d49deb54f5&scene=21#wechat_redirect)
* [建议收藏 | 实时离线数仓实战第一阶段总](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488426&idx=1&sn=6fe3666f5157c651c08a0c8862c1efad&scene=21#wechat_redirect)结
* [超700star！电商项目数据湖建设实战代码 ，拿来即用！](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490491&idx=1&sn=9162eb30018ad3a0c5663afe1d1b0c85&scene=21#wechat_redirect)
* [从0到1建设电商项目数据湖实战教程](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490348&idx=1&sn=e7c6b0d224fea9560218213e7b2e388c&scene=21#wechat_redirect)
* [推荐一套开源电商项目数据湖建设实战代码](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489555&idx=1&sn=e923d58dbabca58d00d76cead2a9580b&scene=21#wechat_redirect)