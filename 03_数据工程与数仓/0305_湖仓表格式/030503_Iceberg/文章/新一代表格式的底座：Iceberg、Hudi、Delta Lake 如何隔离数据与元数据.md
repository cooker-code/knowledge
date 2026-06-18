---
title: 新一代表格式的底座：Iceberg、Hudi、Delta Lake 如何隔离数据与元数据
author: 编程宇航员
date: 干货干货
url: https://mp.weixin.qq.com/s?__biz=MzAwNjExNTU1MQ==&mid=2247489483&idx=1&sn=1dd0b8c465d11b0c5304fc3ed7ff77a4&chksm=9a1d3d3d39ccc3b1e7d25bd3d84fc82efd6ab6c796497f6e4872849a10b909d057d6e15d4f23&mpshare=1&scene=24&srcid=0530UTRRGrxyrm7sAaAQKSS9&sharer_shareinfo=5025537999778b574cf45cf436df34e0&sharer_shareinfo_first=5025537999778b574cf45cf436df34e0#rd
---

数据湖最早给人的感觉很简单：把 Parquet、ORC、Avro 文件放到 HDFS 或对象存储里，再让 Spark、Presto、Trino、Flink 去读。存储便宜，计算弹性，格式开放，似乎只要有一堆文件和一份分区目录，就能搭出一个湖。

真正跑起来后，问题很快出现。一个分区里有几万个小文件，查询前光列文件就要很久；写任务失败后留下半成品文件，读任务不知道哪些文件该读；上游更新一批数据，下游正在读同一张表，看到的是旧文件、新文件还是混合状态；表结构改了，历史文件怎么解释；想回看昨天 10 点的表，目录本身却只保留了当前样子。

Apache Iceberg、Apache Hudi、Delta Lake 这类新一代表格式，解决的不是“再发明一种列式文件”。它们的核心，是在数据文件之上增加一层表级元数据和提交协议：数据文件仍然放在对象存储或 HDFS，表的当前版本、历史版本、文件增删、schema、分区、统计信息、删除语义，由表格式维护。计算引擎不再直接猜目录，而是通过表元数据理解“这张表此刻由哪些文件组成”。

  

## 只靠目录和分区，湖表会卡在哪里

传统 Hive 表把表结构放在 metastore，把数据文件按目录和分区组织。这个模型简单、直观，也足够支撑早期批处理。但它对“表的一致版本”表达得很弱。一次写入可能先生成临时文件，再移动到目标目录；对象存储上的 rename 成本高且语义不完全等同本地文件系统；读任务如果正好遇到写任务提交中间状态，就可能看到不完整文件集合。

另一个问题是元数据过于粗糙。目录告诉你有哪些分区，文件名告诉你文件存在，但不知道每个文件的记录数、字段统计、删除文件关系、所属快照、是否已经被逻辑删除。查询引擎要做分区裁剪、文件裁剪、时间旅行、并发提交，就必须补上这些信息。

所以新一代表格式的第一层价值，是把“文件集合”升级成“表版本”。一次写入不是简单把文件丢到目录里，而是生成新数据文件、更新表元数据、提交一个新的表快照。读者只读取某个已提交快照，就不会被写入中间态干扰。

## 表格式真正管理的是可见性

在湖表里，数据文件通常是不可变的。写入、更新、删除并不是原地改某个 Parquet 文件，而是生成新文件、删除文件或日志文件，再通过元数据提交让它们变成表的一部分。旧文件是否仍可见，取决于当前快照或事务日志，而不是文件还在不在对象存储目录里。

这个设计把数据层和元数据层分开了。数据层负责保存大量列式文件，适合对象存储和分布式扫描；元数据层负责保存表的版本、文件清单、统计信息、删除语义和提交历史。Catalog 或 metastore 通常只保存“当前表元数据指针”，真正的表状态由格式自己的元数据文件或日志描述。

  

这也是多引擎读写能成立的前提。Spark、Flink、Trino、Presto、Hive 不必约定同一套目录扫描细节，只要它们实现同一种表格式协议，就能理解同一份表元数据。计算引擎可以换，底层数据文件可以继续放在对象存储里，表的可见状态由元数据协议维护。

## Iceberg 用快照和 Manifest 描述整张表

Iceberg 的元数据结构很清晰。Catalog 指向当前 metadata file；metadata file 记录当前 snapshot、schema、partition spec、sort order 等信息；snapshot 指向 manifest list；manifest list 再指向一组 manifest file；manifest file 记录具体 data file 和 delete file，以及文件级统计信息。

这个层级让 Iceberg 可以在不列举全表目录的情况下，知道某个快照由哪些文件组成。查询引擎先拿到当前快照，再根据 manifest 里的分区、列统计、文件大小、记录数做裁剪。写入时，Iceberg 生成新数据文件和新 metadata file，通过原子方式把 catalog 指针切到新 metadata file，形成新的表版本。

  

Iceberg 的一个重要特点，是把表状态表达成快照链。每次提交都会产生新的 snapshot，时间旅行、回滚、并发提交检测都围绕快照完成。旧快照保留多久，决定了能回看多久；旧 metadata、manifest 和数据文件什么时候清理，则依赖 expire snapshots、remove orphan files 等维护动作。

所以 Iceberg 的核心不是 metadata 文件多，而是这些文件让表从“目录集合”变成“可提交、可回滚、可裁剪的版本集合”。

## Delta Lake 用事务日志记录表的增删动作

Delta Lake 的入口是表目录下的 `_delta_log`。每次提交会向事务日志写入一组 action，比如 protocol、metaData、add、remove、txn 等。add 表示新增数据文件，remove 表示逻辑删除某个文件，metaData 描述 schema 和分区信息，protocol 描述读写协议版本。随着提交增多，Delta 还会生成 checkpoint，把多次 JSON log 压缩成更快加载的状态文件。

读 Delta 表时，计算引擎不是直接扫描目录，而是读取 `_delta_log`，重建某个版本下的有效文件集合。写入时，先写数据文件，再以事务方式追加一条新的 log commit。只要事务日志提交成功，表就进入新版本；失败的中间文件不会自动成为表的可见数据。

  

这个模型特别容易理解：表状态是日志回放的结果。文件本身可以存在很久，但是否属于当前表版本，由 log 里的 add/remove 决定。时间旅行就是读取某个历史版本的日志状态；vacuum 则负责在保留窗口之后清理不再需要的旧文件。

Delta 的优势在于提交日志直观、生态与 Spark 深度结合，CDC、merge、优化小文件、Z-Ordering 等能力也常围绕日志和文件重写展开。它的维护重点，是日志膨胀、checkpoint 周期、vacuum 保留窗口和多引擎协议兼容。

## Hudi 用 Timeline 管住写入和更新

Hudi 从设计上就很关注 upsert、增量拉取和近实时写入。它用 timeline 记录表上的一系列 instant，每个 instant 代表一次 commit、delta commit、compaction、clean、rollback 等动作。通过 timeline，Hudi 能知道哪些写入已完成，哪些写入还在进行，哪些动作需要回滚或清理。

Hudi 的数据组织围绕 file group 和 file slice。Copy-on-Write 表会把更新合并进新的 base file；Merge-on-Read 表则把新变化先写入 log file，后续通过 compaction 合并到 base file。这样它可以在更新吞吐、读取延迟和文件维护成本之间做取舍。

  

Hudi 还提供 metadata table，用于加速文件列表、列统计、bloom filter 等元数据查询。对大量分区和文件的表来说，避免频繁 list 对象存储目录非常重要。Hudi 的强项不是只做“快照表”，而是把写入、更新、增量消费、compaction、cleaning 放在同一套 timeline 里管理。

因此 Hudi 常出现在 CDC 入湖、近实时 upsert、大量更新删除的链路里。代价是表服务更多：compaction、clustering、clean、metadata table 同步，都需要被当成湖表运行的一部分，而不是事后可有可无的维护任务。

## 隔离元数据之后，多引擎才能共享同一张表

“存算分离”常被理解成计算集群和对象存储分开扩缩容，但湖表真正能被多个引擎共享，还需要“计算和表状态分离”。如果表状态只藏在某个计算引擎的任务临时目录或内部 catalog 里，其他引擎就无法安全读取。表格式把状态写成开放元数据，才让不同引擎有共同语言。

比如一条链路里，Flink 从 CDC 事件实时写入湖表，Spark 做小时级合并和重写，Trino 提供交互式查询，批处理任务做历史回补。它们访问的是同一批数据文件，但更重要的是访问同一份表元数据。谁提交了新快照，谁移除了旧文件，谁改变了 schema，其他引擎都必须按表格式协议理解。

  

这里的难点是并发提交。两个 writer 同时基于旧快照写入，最后谁能提交，谁需要重试，取决于表格式的乐观并发控制和冲突检测。元数据隔离不是让所有人随便写同一目录，而是让大家通过同一套提交协议修改表状态。

## 表格式不是免维护层

新一代表格式解决了很多老湖表问题，但它们不是免维护层。表元数据会增长，快照会堆积，小文件会增多，删除文件会影响读取性能，compaction 和 clustering 会消耗计算资源。数据文件和元数据分离之后，维护动作反而更明确：清理旧快照、压缩小文件、合并删除文件、重写 manifest、生成 checkpoint、清理孤儿文件。

这些维护动作如果长期缺失，表会逐渐变慢。查询计划阶段要读更多元数据；扫描阶段要处理更多小文件；更新删除场景要合并更多 delete/log 文件；对象存储 list、GET 请求和元数据加载都会变成成本。

  

工程里可以把湖表维护写成几条固定规则：

* 快照和日志：保留多久的时间旅行能力，旧快照、旧日志、旧 checkpoint 如何清理。
* 文件大小：写入侧如何控制文件大小，小文件多久合并一次。
* 删除语义：position delete、equality delete、log file、tombstone 是否需要定期合并。
* 元数据加速：manifest、metadata table、transaction log checkpoint 是否健康。
* 并发写入：多个 writer 的冲突检测、重试策略和 catalog 提交是否可观测。

Iceberg、Hudi、Delta Lake 的共同方向，是把数据湖从“文件目录”升级成“带事务语义的表”。Iceberg 更强调快照、manifest 和开放表规范；Delta Lake 更强调事务日志和版本回放；Hudi 更强调 timeline、upsert 和增量处理。它们的共同基石，是把数据文件的存储和表状态的元数据隔离开，让湖表可以被多引擎安全读写、被版本化管理，也能在持续维护中保持可查询。

- END -