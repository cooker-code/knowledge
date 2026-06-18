---
title: 万字长文：基于Apache Hudi + Flink多流拼接(大宽表)最佳实践
author: Apache Hudi
date:
url: http://mp.weixin.qq.com/s?__biz=MzIyMzQ0NjA0MQ==&mid=2247490026&idx=1&sn=15b15480af127977422873492554198e&chksm=e81f4c9cdf68c58a4f10392746913b7d1131c8e08130a269b1a1facbb8ff768123fd115df5bd&mpshare=1&scene=24&srcid=09269yMKcs1drZb3lZK9ASvO&sharer_sharetime=1664179345589&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

> 已吸收至：[[03_数据工程与数仓/0305_湖仓表格式/030502_Hudi/030502_核心知识点/Hudi Payload 多流拼接与并发写边界|Hudi Payload 多流拼接与并发写边界]]


## 1. 背景

经典场景

Flink 侧实现

业务侧通常会基于实时计算引擎在流上做多个数据源的 JOIN 产出这个宽表，但这种解决方案在实践中面临较多挑战，主要可分为以下两种情况：

1. 1. 维表 JOIN

* • 场景挑战：指标数据与维度数据进行关联，其中维度数据量比较大，指标数据 QPS 比较高，导致数据可能会产出延迟。
* • 当前方案：将部分维度数据缓存起起来，缓解高 QPS 下访问维度数据存储引擎产生的任务背压问题。
* • 存在问题：`由于业务方的维度数据和指标数据时间差比较大，所以指标数据流无法设置合理的 TTL；而且存在 Cache 中维度数据没有及时更新，导致下游数据不准确的问题`。

2. 2. 多流 JOIN

* • 场景挑战：多个指标数据进行关联，不同指标数据可能会出现时间差比较大的异常情况。
* • 当前方案：使用基于窗口的 JOIN，并且维持一个比较大的状态。
* • 存在问题：`维持大的状态不仅会给内存带来的一定的压力，同时 Checkpoint 和 Restore 的时间会变得更长，可能会导致任务背压`。

我们基于**Hudi Payload**的合并机制，开发出了一种全新的多流join的解决方案:

* • 多流数据完全在存储层进行拼接，与计算引擎无关，因此不需要保留状态及其 TTL 的设置。
* • 维度数据和指标数据作为不同的流独立更新，更新过程中不需要做多流数据合并，下游读取时再 Merge 多流数据，因此不需要缓存维度数据，同时可以在执行 Compact 时进行 Merge，加速下游查询。

该方案在存储层提供对多流数据的关联能力，旨在解决实时场景下多流 join遇到的一系列问题。

## 2.核心能力

### 2.1 Timeline (时间线)

在所有的表中维护了一个包含在不同的即时(Instant)时间对数据集操作(比如新增、修改或删除)的时间轴(Timeline) 在每一次对hudi表的数据集操作时都会在该表的Timeline上生成一个Instant，从而可以实现在仅查询某个时间点之后成功提交的数据，或是仅查询某个时间点之前的数据，有效避免了扫描更大时间范围的数据。同时，可以高效的只查询更改前的文件（如在某个Instant提交了更改操作后，仅query某个时间点之前的数据，则仍可以query修改前的数据）

Action（操作行为）：

* • COMMITS：数据提交
* • CLEANS：数据删除
* • DELTA\_COMMIT：
* • COMPACTION：小文件合并
* • ROLLBACK：回滚
* • SAVEPOINT：保存点

Timeline是hudi用来管理提交(commit)的抽象，每个commit都绑定一个固定时间戳，分散到时间线上。在Timeline上，每个commit被抽象为一个HoodieInstant，一个instant记录了一次提交（commit）的行为、时间戳和状态。

上图的例子展示了10:00至10:20，每5分钟在 Hudi 表的 upsert 操作，时间线有 commit，clean 和 compact。同时还可以观察到 commit time 记录的是数据到达时间（如，10:20AM）,而实际是按 event time (事件时间) 从7:00每小时一个分区来组织数据的。到达时间和事件事件是平衡数据延迟及完整性的两个主要概念。

迟到的数据到来（如，事件时间是9:00，在>1小时之后的10:20到达），会根据事件数据写入到对应的分区。在时间线的帮助下，增量查询只需要读取所有在某一瞬间（instant time）以来 commit 成功的变更文件就可以获取到新数据，而不通过扫描所有的文件。

### 2.2. 并发控制

#### 2.2.1. 概述

如今数据湖上的事务被认为是 Lakehouse 的一个关键特征。但到目前为止，实际完成了什么？目前有哪些方法？它们在现实世界中的表现如何？这些问题是本文的重点。

有幸从事过各种数据库项目——RDBMS (Oracle[1])、NoSQL 键值存储 (Voldemort[2])、流数据库 (ksqlDB[3])、闭源实时数据存储，当然还有 Apache Hudi， 我可以肯定地说，工作负载的不同深刻地影响了不同数据库中采用的并发控制机制。本文还将介绍我们如何重新思考 Apache Hudi 数据湖的并发控制机制。

首先，我们直截了当点，RDBMS 数据库提供了最丰富的事务功能集和最广泛的并发控制机制[4]，不同的隔离级别、细粒度锁、死锁检测/避免等其他更多机制，因为它们必须支持行级变更和跨多个表的读取，同时强制执行键约束[5]并维护索引[6]。

而NoSQL 存储提供了非常弱的保证，例如仅仅提供最终一致性和简单的行级原子性，以换取更简单的工作负载的更好的扩展性。传统数据仓库基于列存或多或少提供了您在 RDBMS 中可以找到的全套功能，强制[7]执行锁定和键约束，而云数据仓库似乎更多地关注存算分离架构，同时提供更少的隔离级别。作为一个令人惊讶的例子，没有强制执行[8]键约束。

#### 2.2.2. 数据湖并发控制中的陷阱

从历史看来，数据湖一直被视为在云存储上读取/写入文件的批处理作业，有趣的是看到大多数新工作如何扩展此视图并使用某种形式的“乐观并发控制[9]”（OCC）来实现文件版本控制。

OCC 作业采用表级锁来检查它们是否影响了重叠文件，如果存在冲突则中止操作，锁有时甚至只是在单个 Apache Spark Driver节点上持有的 JVM 级锁，这对于主要将文件附加到表的旧式批处理作业的轻量级协调来说可能没问题，但不能广泛应用于现代数据湖工作负载。此类方法是在考虑不可变/仅附加数据模型的情况下构建的，这些模型不适用于增量数据处理或键控更新/删除。

OCC 非常乐观地认为真正的冲突永远不会发生。将 OCC 与 RDBMS 或传统数据仓库的完全成熟的事务功能进行比较的开发人员布道是完全错误的，直接引用维基百科——“如果频繁地争用数据资源，重复重启事务的成本会显着损害性能，在这种情况下，其他并发控制方法[10]可能更适合。” 当冲突确实发生时，它们会导致大量资源浪费，因为你有每次尝试运行几个小时后都失败的批处理作业！

想象一下两个写入进程的真实场景：一个每 30 分钟生成一次新数据的摄取写入作业和一个执行 GDPR 的删除作业，需要 2 小时才能完成删除。这些很可能与随机删除重叠文件，并且删除作业几乎可以保证每次都饿死并且无法提交。在数据库方面，将长期运行的事务与乐观混合会导致失望，因为事务越长，它们重叠的可能性就越高。

那么有什么替代方案呢？锁？维基百科还说 - “但是，基于锁（“悲观”）的方法也可能提供较差的性能，因为即使避免了死锁，锁也会极大地限制有效的并发性。”。这就是 Hudi 采用不同方法的地方，我们认为这种方法更适合现代数据湖事务，这些事务通常是长期运行的，甚至是连续的。与数据库的标准读/写相比，数据湖工作负载与高吞吐量流处理作业共享更多特征，这就是我们借鉴的地方。在流处理中，事件被序列化为单个有序日志，避免任何锁/并发瓶颈，用户可以每秒连续处理数百万个事件。Hudi 在 Hudi 时间线[11]上实现了一个文件级、基于日志的并发控制协议，而该协议又依赖于对云存储的最低限度的原子写入。通过将事件日志构建为进程间协调的核心部分，Hudi 能够提供一些灵活的部署模型，与仅跟踪表快照的纯 OCC 方法相比，这些模型提供更高的并发性。

#### 2.2.3. 模型 1：单写入，内联表服务

并发控制的最简单形式就是完全没有并发。数据湖表通常在其上运行公共服务以确保效率，从旧版本和日志中回收存储空间、合并文件（Hudi 中的Clustering）、合并增量（Hudi 中的Compaction）等等。Hudi 可以简单地消除对并发控制的需求，并通过支持这些开箱即用的表服务并在每次写入表后内联运行来最大化吞吐量。执行计划是幂等的，持久化至时间线并从故障中自动恢复。对于大多数简单的用例，这意味着只需写入就足以获得一个不需要并发控制的管理良好的表。

#### 2.2.4. 模型2：单写入，异步表服务

我们上面的删除/摄取示例并不是那么简单。虽然摄取/写入可能只是更新表上的最后 N 个分区，但删除甚至可能跨越整个表，将它们混合在同一个工作负载中可能会大大影响摄取延迟，因此Hudi 提供了以异步方式运行表服务的选项，其中大部分繁重的工作（例如通过压缩服务实际重写列数据）是异步完成的，消除了任何重复的浪费重试，同时还使用Clustering技术。因此单个写入可以同时使用常规更新和 GDPR 删除并将它们序列化到日志中。鉴于 Hudi 具有记录级索引并且 avro 日志写入要便宜得多（与写入 parquet 相比，后者可能要贵 10 倍或更高），摄取延迟可以持续，同时享受出色的可回溯性。事实上我们能够在 Uber[12] 将这个模型扩展到 100 PB数据规模，通过将所有删除和更新排序到同一个源 Apache Kafka 主题中，并发控制不仅仅是锁，Hudi 无需任何外部锁即可完成所有这一切。

#### 2.2.5. 模型3：多写入

但是并不总是可以将删除序列化到相同的写入流中，或者需要基于 sql 的删除。对于多个分布式进程，某种形式的锁是不可避免的，但就像真正的数据库一样，Hudi 的并发模型足够智能，可以将实际写入表的内容与管理或优化表的表服务区分开来。Hudi 提供了类似的跨多个写入器的乐观并发控制，但表服务仍然可以完全无锁和异步地执行。这意味着删除作业只能对删除进行编码，摄取作业可以记录更新，而压缩服务再次将更新/删除应用于基本文件。尽管删除作业和摄取作业可以像我们上面提到的那样相互竞争和饿死，但它们的运行时间要低得多，浪费也大大降低，因为压缩完成了parquet/列数据写入的繁重工作。

综上所述，在这个基础上我们还有很多方法可以改进。

* • 首先，Hudi 已经实现了一种标记机制[13]，可以跟踪作为活动写入事务一部分的所有文件，以及一种可以跟踪表的活动写入者的心跳机制。这可以由其他活动事务/写入器直接使用来检测其他写入器正在做什么，如果检测到冲突，则尽早中止[14]，从而更快地将集群资源返回给其他作业。
* • 虽然在需要可序列化快照隔离时乐观并发控制很有吸引力，但它既不是最佳方法，也不是处理写入者之间并发性的唯一方法。我们计划使用 CRDT 和广泛采用的流处理概念，通过我们的日志合并 API 实现完全无锁的并发控制，这已经被证明[15]可以为数据湖维持巨大的连续写入量。
* • 谈到键约束，Hudi 是当今唯一确保唯一键约束[16]的湖事务层，但仅限于表的记录键。我们将寻求以更通用的形式将此功能扩展到非主键字段，并使用上述较新的并发模型。

### 2.3. marker机制

#### 2.3.1. 概述

Hudi 支持在写入时自动清理未成功提交的数据。Apache Hudi 在写入时引入标记机制来有效跟踪写入存储的数据文件。在本文中，我们将深入探讨现有直接标记文件机制的设计，并解释了其在云存储（如 AWS S3、Aliyun OSS）上针对非常大批量写入的性能问题。并且演示如何通过引入基于时间轴服务器的标记来提高写入性能。

#### 2.3.2. 为何引入Markers机制

Hudi中的 `marker` 是一个表示存储中存在对应的数据文件的标签，Hudi使用它在故障和回滚场景中自动清理未提交的数据。每个标记条目由三部分组成

* • 数据文件名
* • 标记扩展名 (.marker)
* • 创建文件的 I/O 操作（CREATE - 插入、MERGE - 更新/删除或 APPEND - 两者之一）。

例如标记 `91245ce3-bb82-4f9f-969e-343364159174-0_140-579-0_20210820173605.parquet.marker.CREATE` 指示相应的数据文件是 `91245ce3-bb82-4f9f-969e-343364159174-0_140-579-0_20210820173605.parquet` 并且 I/O 类型是 CREATE。在写入每个数据文件之前，Hudi 写入客户端首先在存储中创建一个标记，该标记会被持久化，在提交成功后会被写入客户端显式删除。标记对于写客户端有效地执行不同的操作很有用，标记主要有如下两个作用

* • **删除重复/部分数据文件**：通过 Spark 写入 Hudi 时会有多个 Executor 进行并发写入。一个 Executor 可能失败，留下部分数据文件写入，在这种情况下 Spark 会重试 Task ，当启用 `speculative execution` 时，可以有多次 `attempts` 成功将相同的数据写入不同的文件，但最终只有一次 `attempt` 会交给 Spark Driver程序进程进行提交。标记有助于有效识别写入的部分数据文件，其中包含与后来成功写入的数据文件相比的重复数据，并在写入和提交完成之前清理这些重复的数据文件。
* • **回滚失败的提交**：写入时可能在中间失败，留下部分写入的数据文件。在这种情况下，标记条目会在提交失败时保留在存储中。在接下来的写操作中，写客户端首先回滚失败的提交，通过标记识别这些提交中写入的数据文件并删除它们。接下来我们将深入研究现有的标记机制，阐述其性能问题，并演示新的基于时间轴服务器的标记机制来解决该问题。

#### 2.3.3. 基于Timeline (时间线)服务器的标记机制提高写入性能

这里主要描述**基于时间线服务器的标记机制**，该机制优化了存储标记的相关延迟。Hudi 中的时间线服务器用作提供文件系统和时间线视图。如下图所示，新的基于时间线服务器的标记机制将标记创建和其他标记相关操作从各个执行器委托给时间线服务器进行集中处理。时间线服务器在内存中为相应的标记请求维护创建的标记，时间线服务器通过定期将内存标记刷新到存储中有限数量的底层文件来实现一致性。通过这种方式，即使数据文件数量庞大，也可以显着减少与标记相关的实际文件操作次数和延迟，从而提高写入性能。

为了提高处理标记创建请求的效率，我们设计了在时间线服务器上批量处理标记请求。每个标记创建请求在 Javalin 时间线服务器中异步处理，并在处理前排队。对于每个批处理间隔，例如 20 毫秒，调度线程从队列中拉出待处理的请求并将它们发送到工作线程进行处理。每个工作线程处理标记创建请求，并通过重写存储标记的底层文件。有多个工作线程并发运行，考虑到文件覆盖的时间比批处理时间长，每个工作线程写入一个不被其他线程触及的独占文件以保证一致性和正确性。批处理间隔和工作线程数都可以通过写入选项进行配置。

请注意工作线程始终通过将请求中的标记名称与时间线服务器上维护的所有标记的内存副本进行比较来检查标记是否已经创建。存储标记的底层文件仅在第一个标记请求（延迟加载）时读取。请求的响应只有在新标记刷新到文件后才会返回，以便在时间线服务器故障的情况下，时间线服务器可以恢复已经创建的标记。这些确保存储和内存中副本之间的一致性，并提高处理标记请求的性能。

### 2.4. 早期冲突检测

#### 2.4.1. 概述

目前Hudi实现了一个基于时间轴的OCC（Optimistic Concurrency Control）来保证数据多写入之间的一致性、完整性和正确性。但是，相关的冲突检测是在提交元数据之前和数据写入完成之后。如果检测到任何冲突，则会造成集群资源的浪费，因为计算和写入已经完成。为了解决这个问题，这个 RFC[17] 提出了一个基于现有 Hudi 标记机制的早期冲突检测机制。不同类型的标记维护者之间的早期冲突检测工作流程存在一些细微的差异：

* • 对于直接标记，hoodie 直接列出必要的标记文件，并在writers创建标记之前和开始写入相应的数据文件之前进行冲突检查。
* • 对于基于时间线服务器的标记，hoodie 只是在writers创建标记之前和开始写入相应的数据文件之前获取标记冲突检查的结果。对冲突进行异步和定期检查，以便尽早检测到写入冲突。两个 writer 仍然可以写入同一个 file slice 的数据文件，直到在下一轮检查中检测到冲突。

更重要的是， Hoodie 可以提前停止写入，因为早期的冲突检测，可以将资源释放到集群，提高资源利用率。

#### 2.4.2.为什么需要早期冲突检测

数据湖的事务和 multi-writers 正在成为如今构建 Lakehouse 的关键特征。直接引用**Lakehouse 并发控制：我们是否过于乐观？**[18]

> “Hudi 在 Hudi 时间轴上实现了一个文件级、基于日志的并发控制协议，该协议又依赖于对云存储的最低限度的原子放置。通过将事件日志构建为进程间协调的核心部分，Hudi 能够提供一些灵活的部署模型，与仅跟踪表快照的纯 OCC 方法相比，这些模型提供了更高的并发性。”

在multi-writer场景下，Hudi 现有的冲突检测发生在 writer 写完数据之后和提交元数据之前。也就是说，虽然所有的计算和数据写入都已经完成，但是writer在开始commit的时候才检测到冲突的发生，这就造成了资源的浪费。例如：现在有两个写作业：job1会写10M的数据到Hudi表，包括更新文件组1。另一个job2会写100G到Hudi表，也会更新同一个文件组1。Job1 成功完成并提交给 Hudi。几个小时后，job2 写完数据文件（100G），开始提交元数据。这时候发现和job1比较有冲突，job2失败后不得不中止重新运行。显然，大量的计算资源和时间都浪费在了job2上。

Hudi 目前有两个重要的机制，标记机制和心跳机制：

1. 1. 标记机制可以跟踪作为主动写入一部分的所有文件。
2. 2. 心跳机制，可以跟踪所有活跃的writers到一个Hudi表。

基于标记机制和心跳机制，本RFC提出了一种新的冲突检测：Early Conflict Detection。在writer创建marker和开始写入文件之前，Hudi会执行这个新的冲突检测，尝试直接检测写入冲突或者尽早获取异步冲突检查结果（Timeline-Based）并中止writer当冲突发生时，这样我们就可以尽快释放资源，提高资源利用率。

#### 2.4.3.实现

这是早期冲突检测的工作流程，如图 1 所示。正如我们所见，当 `supportsOptimisticConcurrencyControl` 和 `isEarlyConflictDetectionEnable` 都为真时，我们可以使用这种早期冲突检测功能。否则，我们跳过此检查并直接创建标记。

### 2.5. 事务写（ACID能力）

传统数据湖在数据写入时的事务性方面做得不太好，但随着越来越多的业务关键处理流程移至数据湖，情况也在发生变化，我们需要一种机制来原子地发布一批数据，即仅保存有效数据，部分失败必须**回滚**而不会损坏已有数据集。同时查询的结果必须是可重复的，查询端看不到任何部分提取的数据，任何提交的数据都必须可靠地写入。Hudi提供了强大的ACID能力。高效的回滚机制能够保证数据一致性和避免“孤儿文件”或中间状态数据文件残留和产生。

### 2.6. 灵活的Payload机制

#### 2.6.1.摘要

Apache Hudi 的Payload是一种可扩展的数据处理机制，通过不同的Payload我们可以实现复杂场景的定制化数据写入方式，大大增加了数据处理的灵活性。Hudi Payload在写入和读取Hudi表时对数据进行去重、过滤、合并等操作的工具类，通过使用参数 "hoodie.datasource.write.payload.class"指定我们需要使用的Payload class。本文我们会深入探讨Hudi Payload的机制和不同Payload的区别及使用场景。

#### 2.6.2.为何需要Payload

在数据写入的时候，现有整行插入、整行覆盖的方式无法满足所有场景要求，写入的数据也会有一些定制化处理需求，因此需要有更加灵活的写入方式以及对写入数据进行一定的处理，Hudi提供的playload方式可以很好的解决该问题，例如可以解决写入时数据`去重`问题，针对`部分字段进行更新`等等。

#### 2.6.3.Payload的作用机制

写入Hudi表时需要指定一个参数 `hoodie.datasource.write.precombine.field` ，这个字段也称为Precombine Key，Hudi Payload就是根据这个指定的字段来处理数据，它将每条数据都构建成一个Payload，因此数据间的比较就变成了Payload之间的比较。只需要根据业务需求实现Payload的比较方法，即可实现对数据的处理。Hudi所有Payload都实现HoodieRecordPayload接口，下面列出了所有实现该接口的预置Payload类。

下图列举了HoodieRecordPayload接口需要实现的方法，这里有两个重要的方法preCombine和combineAndGetUpdateValue，下面我们对这两个方法进行分析。

#### 2.6.3.1 preCombine分析

从下图可以看出，该方法比较当前数据和oldValue，然后返回一条记录。

从preCombine方法的注释描述也可以知道首先它在多条相同主键的数据同时写入Hudi时，用来进行数据去重。调用位置

其实该方法还有另一个调用的地方，即在MOR表读取时会对Log file中的相同主键的数据进行处理。如果同一条数据多次修改并写入了MOR表的Log文件，在读取时也会进行preCombine。

#### 2.6.3.2 combineAndGetUpdateValue分析

该方法将currentValue（即现有parquet文件中的数据）与新数据进行对比，判断是否需要持久化新数据。

由于COW表和MOR表的读写原理差异，因此combineAndGetUpdateValue的调用在COW和MOR中也有所不同：

* • 在COW**写入时**会将**新写入的数据与Hudi表中存的currentValue**进行比较，返回需要持久化的数据
* • 在MOR**读取时**会将经过preCombine处理的**Log中的数据与Parquet文件中的数据**进行比较，返回需要持久化的数据

### 2.7. 跨任务并发写支持

内部Hudi版本支持了基于文件锁及OCC机制实现了Flink 多重writer并发写入的场景。

### 2.8.异步compaction和clean

Hudi支持job内inline compation和clean，可以及时的合并小文件和清理，从而避免了小文件问题。当然也可以通过参数关闭inline compaction，hudi在spark/flink都提供了offline compaction和clean。

## 3. 多流拼接过程

接下来，介绍多流拼接场景下 Snapshot Query 的核心过程，即先对 LogFile 进行去重合并，然后再合并 BaseFile 和 去重后的 LogFile 中的数据。下图显示了整个数据合并的过程，具体可以拆分成以下 两个过程：

• Merge LogFile

Hudi 现有逻辑是将 LogFile 中的数据读出来存放在 Map 中，对于 LogFile 中每条Record，如果 Key 不存在 Map 中，则直接放入 Map，如果 Key 已经存在于 Map 中，则需要更新操作。

在多流拼接中，因为 LogFile 中存在不同数据流写入的数据，即每条数据的列可能不相同，所以在更新的时候需要判断相同 Key 的两个 Record 是否来自同一个流，是则做更新，不是则做拼接。如图 3 所示，读到 LogFile2 中的主键是 key1 的 Record 时，key1 对应的 Record 在 Map 中已经存在，但这两个 Record 来自不同流，则需要拼接形成一条新的 Record (key1，b0\_new，c0\_new，d0\_new) 放入 Map 中。

* • Merge BaseFile and LogFile

Hudi 现有默认逻辑是对于每一条存在于 BaseFile 中的 Record，查看 Map 中是否存在 key 相同的 Record，如果存在，则用 Map 中的 Record 覆盖 BaseFile 中的 Record。在多流拼接中，Map 中的 Record 不会完整覆盖 BaseFile 中对应的 Record，可能只会更新部分列的值，即 Map 中的 Record 对应的列。

如下图所示，以最简单的覆盖逻辑为例，当读到 BaseFile 中的主键是 key1 的 Record 时，发现 key1 在 Map 中已经存在并且对应的 Record 有 BCD 三列的值，则更新 BaseFile 中的 BCD 列，得到新的 Record(key1，b0\_new，c0\_new，d0\_new，e0)，注意 E 列没有被更新，所以保持原来的值 e0。对于新增的 Key 如 Key3 对应的 Record，则需要将 BCE 三列补上默认值形成一条完整的 Record。

## 4. 实现原理图

实现的原理基本上就是通过自定义的 Payload class 来实现相同 key 不同源数据的合并逻辑，写端会在批次内做多源的合并并写入 log，读端在读时合并时也会调用相同的逻辑来处理跨批次的情况。

这里需要注意的是乱序和迟到数据（out-of-order and late events）的问题。如果不做处理，在下游经常会导致旧数据覆盖新数据，或者列更新不完整的情况。

针对乱序和迟到数据，我们对 Hudi 做了 Multiple ordering value 的增强，保证每个源只能更新属于自己那部分列的数据，并且可以根据设置的 event time (ordering value) 列，确保只会让新数据覆盖旧数据。最后结合 lock less multiple writers 来实现多 Job 多源的并发写入。

## 5.如何使用

### 5.1.Maven pom 依赖

针对此功能特性发了基于0.12.0-1-tencent的快照版本

```
<dependencies>
  <dependency>
    <groupId>org.apache.hudi</groupId>
    <artifactId>hudi-flink1.13-bundle</artifactId>
    <version>0.12.0</version>
  </dependency>
</dependencies>
```

### 5.2.多Flink Job写入同一张目标表

Job1

* • 源表A

```
CREATE TABLE sourceA (\n" +
  uuid STRING,\n" +
  name STRING,\n" +
  _ts1 timestamp(3)\n" +
) WITH (\n" +
.....
)
```

* • 目标表

```
    public static String sinkTableDDL1() {
        return String.format("create table %s(\n"
            + "  uuid STRING,\n"
            + "  name STRING,\n"
            + "  age int,\n"
            + "  _ts1 bigint,\n"
            + "  _ts2 bigint,\n"
            + "  PRIMARY KEY(uuid) NOT ENFORCED"
            + ")\n"
            + " PARTITIONED BY (_ts1)\n"
            + " with (\n"
            + "  'connector' = 'hudi',\n"
            + "  'path' = '%s', -- 替换成的绝对路径\n"
            + "  'table.type' = 'MERGE_ON_READ',\n"
            + "  'write.bucket_assign.tasks' = '5',\n"
            + "  'write.tasks' = '5',\n"
            + "  'write.partition.format' = 'yyyyMMdd',\n"
            + "  'write.partition.timestamp.type' = 'EPOCHMILLISECONDS',\n"
            + "  'hoodie.bucket.index.num.buckets' = '5',\n"
            + "  'changelog.enabled' = 'true',\n"
            + "  'index.type' = 'BUCKET',\n"
            + "  'hoodie.bucket.index.num.buckets' = '5',\n"
            + String.format("  '%s' = '%s',\n", FlinkOptions.PRECOMBINE_FIELD.key(), "_ts1:name;_ts2:age")
            + "  'write.payload.class' = '" + PartialUpdateAvroPayload.class.getName() + "',\n"
            + "  'hoodie.write.log.suffix' = 'job1',\n"
            + "  'hoodie.write.concurrency.mode' = 'optimistic_concurrency_control',\n"
            + "  'hoodie.write.lock.provider' = 'org.apache.hudi.client.transaction.lock.FileSystemBasedLockProvider',\n"
            + "  'hoodie.cleaner.policy.failed.writes' = 'LAZY',\n"
            + "  'hoodie.cleaner.policy' = 'KEEP_LATEST_BY_HOURS',\n"
            + "  'hoodie.consistency.check.enabled' = 'false',\n"
            + "  'hoodie.write.lock.early.conflict.detection.enable' = 'true',\n"
            + "  'hoodie.write.lock.early.conflict.detection.strategy' = '"
            + SimpleTransactionDirectMarkerBasedEarlyConflictDetectionStrategy.class.getName() + "',\n"
            + "  'hoodie.keep.min.commits' = '1440',\n"
            + "  'hoodie.keep.max.commits' = '2880',\n"
            + "  'compaction.schedule.enabled'='false',\n"
            + "  'compaction.async.enabled'='false',\n"
            + "  'compaction.trigger.strategy'='num_or_time',\n"
            + "  'compaction.delta_commits' ='5',\n"
            + "  'compaction.delta_seconds' ='180',\n"
            + "  'compaction.max_memory' = '3096',\n"
            + "  'clean.async.enabled' ='false',\n"
            + "  'hive_sync.enable' = 'true',\n"
            + "  'hive_sync.mode' = 'hms',\n"
            + "  'hive_sync.db' = '%s',\n"
            + "  'hive_sync.table' = '%s',\n"
            + "  'hive_sync.metastore.uris' = '%s'\n"
            + ")", sinkAliasTable1, basePath, dbName, targetTable, metastoreUrl);
    }
```

A流数据写入

```
insert into %s(uuid, name, _ts1) select uuid, name, ts as _ts1 from sourceA
```

Job2

* • 源表B

```
CREATE TABLE sourceB (\n" +
  uuid varchar(20),\n" +
  age int,\n" +
  _ts2 timestamp(3)\n" +
) WITH (\n" +
.....
)
```

* • 目标表

```
   public static String sinkTableDDL2() {
        return String.format("create table %s(\n"
            + "  uuid STRING,\n"
            + "  name STRING,\n"
            + "  age int,\n"
            + "  _ts1 bigint,\n"
            + "  _ts2 bigint,\n"
            + "  PRIMARY KEY(uuid) NOT ENFORCED"
            + ")\n"
            + " PARTITIONED BY (_ts2)\n"
            + " with (\n"
            + "  'connector' = 'hudi',\n"
            + "  'path' = '%s', -- 替换成的绝对路径\n"
            + "  'table.type' = 'MERGE_ON_READ',\n"
            + "  'write.bucket_assign.tasks' = '5',\n"
            + "  'write.tasks' = '5',\n"
            + "  'write.partition.format' = 'yyyyMMdd',\n"
            + "  'write.partition.timestamp.type' = 'EPOCHMILLISECONDS',\n"
            + "  'changelog.enabled' = 'true',\n"
            + "  'index.type' = 'BUCKET',\n"
            + "  'hoodie.bucket.index.num.buckets' = '5',\n"
            + String.format("  '%s' = '%s',\n", FlinkOptions.PRECOMBINE_FIELD.key(), "_ts1:name;_ts2:age")
            + "  'write.payload.class' = '" + PartialUpdateAvroPayload.class.getName() + "',\n"
            + "  'hoodie.write.log.suffix' = 'job2',\n"
            + "  'hoodie.write.concurrency.mode' = 'optimistic_concurrency_control',\n"
            + "  'hoodie.write.lock.provider' = 'org.apache.hudi.client.transaction.lock.FileSystemBasedLockProvider',\n"
            + "  'hoodie.cleaner.policy.failed.writes' = 'LAZY',\n"
            + "  'hoodie.cleaner.policy' = 'KEEP_LATEST_BY_HOURS',\n"
            + "  'hoodie.consistency.check.enabled' = 'false',\n"
            + "  'hoodie.write.lock.early.conflict.detection.enable' = 'true',\n"
            + "  'hoodie.write.lock.early.conflict.detection.strategy' = '"
            + SimpleTransactionDirectMarkerBasedEarlyConflictDetectionStrategy.class.getName() + "',\n"
            + "  'hoodie.keep.min.commits' = '1440',\n"
            + "  'hoodie.keep.max.commits' = '2880',\n"
            + "  'compaction.schedule.enabled'='true',\n"
            + "  'compaction.async.enabled'='true',\n"
            + "  'compaction.trigger.strategy'='num_or_time',\n"
            + "  'compaction.delta_commits' ='5',\n"
            + "  'compaction.delta_seconds' ='180',\n"
            + "  'compaction.max_memory' = '3096',\n"
            + "  'clean.async.enabled' ='false',\n"
            + "  'hive_sync.enable' = 'true',\n"
            + "  'hive_sync.mode' = 'hms',\n"
            + "  'hive_sync.db' = '%s',\n"
            + "  'hive_sync.table' = '%s',\n"
            + "  'hive_sync.metastore.uris' = '%s'\n"
            + ")", sinkAliasTable2, basePath, dbName, targetTable, metastoreUrl);
    }
```

B流数据写入

```
insert into %s(uuid, age, _ts2) select uuid, age, ts as _ts2 from sourceB
```

### 5.3.单个flink job多pipline写入同一张表

* • 创建源表A、源表B 同5.2中创建表ddl
* • A流、B流数据写入 同5.2中insert 写入

### 5.4.设置参数说明

| 参数名 | Required | 默认值 | 备注 |
| --- | --- | --- | --- |
| path | Required | N/A | 目标表的路径 |
| table.type | Optional | MERGE\_ON\_READ | 表的类型，COPY\_ON\_WRITE or MERGE\_ON\_READ |
| write.operation | Optional | upsert | 写入类型，UPSERT or INSERT |
| **write.payload.class** | Required | PartialUpdateAvroPayload | 指定处理数据的Payload，PartialUpdateAvroPayload.class.getName() |
| write.partition.format | Optional | N/A | 分区格式，yyyyMMdd按天分区，yyyyMMddHH按小时分区 |
| write.partition.timestamp.type | Optional | N/A | 分区类型，当分区字段为bigint(long)时使用, 值为: `EPOCHMILLISECONDS` |
| **write.precombine** | Required | ts | \_ts1:name;\_ts2:age 说明\_ts1、\_ts2都是排序字段，**冒号**后是需要**更新**的字段，**分号**用来表示不同流 |
| hoodie.write.log.suffix | Required | 无 | log文件后缀，用来区分不同job |
| index.type | Required | FLINK\_STATE | 这里设置为`BUCKET` |
| hoodie.bucket.index.num.buckets | Required | 256 | 需要根据数据量预估 |
| hoodie.write.concurrency.mode | Required | SINGLE\_WRITER | 设置为`optimistic_concurrency_control` |
| hoodie.cleaner.policy.failed.writes | Required | LAZY | 设置为`LAZY` |
| hoodie.write.lock.early.conflict.detection.enable | Required | true |  |
| hoodie.write.lock.early.conflict.detection.strategy | Required | SimpleTransactionDirectMarkerBasedEarlyConflictDetectionStrategy.class.getName() |  |

**说明：** 1.根据precombine key比较是否要更新数据，适合实时入湖且入湖顺序乱序 2.如果用户原始表中时间字段数值相同没法比较则会按照`FIFO`的顺序来拼接合并。

### 5.5.查询数据

#### 5.5.1.使用spark查询

```
select * from hudi_tauth_test.hudi_partial_01_rt limit 10;
```

#### 5.5.2.使用presto查询

**Note：presto待更新版本**

## 6.效果收益

最终，基于 Hudi 多流拼接的方案，除了解决背景中问题外，在实时数仓的 DWS 层落地，单表支持了 3+ 数据流的并发导入，覆盖了数百 TB 的数据。此外，在使用 Spark 对宽表数据进行查询时，由于数据已经去重压缩拼接成大宽表了，在单次扫描量几十 TB 的查询中，性能相比于直接使用多表关联性能提升在 200% 以上，在一些更加复杂的查询下，也有 40-140% 的性能提升。

## 7.下一步规划

* • 进一步提高Hudi 多流拼接方案的易用性，减少参数配置，后续会做部分列插入和更新的 SQL 的语法支持以及参数的收敛。
* • 利用payload机制实现Flink left Join、right join、TopN等功能。
* • 将multi writer这一块功能回推到社区。

**推荐阅读**

[字节跳动基于 Apache Hudi 构建实时数仓的实践](http://mp.weixin.qq.com/s?__biz=MzIyMzQ0NjA0MQ==&mid=2247489986&idx=1&sn=7772652fd22520a9f206ae4c82922e77&chksm=e81f4cb4df68c5a200c37e6699596b57dda527b676de67205bd2087f598ce0e4b158f2b057d9&scene=21#wechat_redirect)

[华为云 MRS 基于 Apache Hudi 极致查询优化的探索实践](http://mp.weixin.qq.com/s?__biz=MzIyMzQ0NjA0MQ==&mid=2247489954&idx=1&sn=cf55aebfcd58657302dbc9c4cd1b0830&chksm=e81f4cd4df68c5c2557e434a8d295cea40a0bbf0d9e20a30ba111794df37f9e44bc10c5cdb02&scene=21#wechat_redirect)

[基于 Apache Hudi 的湖仓一体技术在 Shopee 的实践](http://mp.weixin.qq.com/s?__biz=MzIyMzQ0NjA0MQ==&mid=2247489927&idx=2&sn=25d6488702f6397267ba8a26285f9d1b&chksm=e81f4cf1df68c5e7fbd0c484eb8f2476350c2571e7fd83feff6950f98ff107a3193f7eb7538c&scene=21#wechat_redirect)

[字节跳动基于Apache Doris + Hudi的湖仓分析探索实践](http://mp.weixin.qq.com/s?__biz=MzIyMzQ0NjA0MQ==&mid=2247489757&idx=2&sn=ba99265812c3cb71971c8dc3d9306cce&chksm=e81f4dabdf68c4bddf5a7d9069bcc9c3f160ed220644ad83d383cb21ce9edacc92198f610caa&scene=21#wechat_redirect)

[构建端到端的开源现代数据平台](http://mp.weixin.qq.com/s?__biz=MzIyMzQ0NjA0MQ==&mid=2247489714&idx=1&sn=9adc2e57ab6a1133ff87ccc7b8fd5d77&chksm=e81f4dc4df68c4d2441e59296d529b718bd874cfacdc2f3a47d16b694e707b31c3ebce9cfb7d&scene=21#wechat_redirect)

## 参考

* • Apache Hudi灵活的Payload机制[19]
* • Snapshot Isolation using Optimistic Concurrency Control for multi-writers[20]
* • [https://mp.weixin.qq.com/s/3nsYTVu9nZCIFaaXP09hiQ](https://mp.weixin.qq.com/s?__biz=MzkzMDE5MDgwMQ==&mid=2247489737&idx=1&sn=a6d94fa72cd9468414d651cf28e31021&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkzMDE5MDgwMQ==&mid=2247489737&idx=1&sn=a6d94fa72cd9468414d651cf28e31021&scene=21#wechat_redirect")
* • https://segmentfault.com/a/1190000041630798

#### 引用链接

`[1]` Oracle: *https://www.oracle.com/database/*
`[2]` Voldemort: *https://www.slideshare.net/vinothchandar/voldemort-prototype-to-production-nectar-edits*
`[3]` ksqlDB: *https://www.confluent.io/blog/ksqldb-pull-queries-high-availability/*
`[4]` 机制: *https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-transaction-model.html*
`[5]` 键约束: *https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html*
`[6]` 索引: *https://dev.mysql.com/doc/refman/8.0/en/create-table-secondary-indexes.html*
`[7]` 强制: *https://docs.teradata.com/r/a8IdS6iVHR77Z9RrIkmMGg/wFPZS4jwZgSG21GnOIpEsw*
`[8]` 强制执行: *https://docs.snowflake.com/en/sql-reference/constraints-overview.html#supported-constraint-types*
`[9]` 乐观并发控制: *https://en.wikipedia.org/wiki/Optimistic\_concurrency\_control*
`[10]` 并发控制方法: *https://en.wikipedia.org/wiki/Concurrency\_control*
`[11]` 时间线: *https://hudi.apache.org/docs/timeline*
`[12]` Uber: *https://eng.uber.com/uber-big-data-platform/*
`[13]` 标记机制: *https://hudi.apache.org/blog/2021/08/18/improving-marker-mechanism/*
`[14]` 尽早中止: *https://issues.apache.org/jira/browse/HUDI-1575*
`[15]` 被证明: *https://hudi.apache.org/blog/2021/09/01/building-eb-level-data-lake-using-hudi-at-bytedance/#functionality-support*
`[16]` 唯一键约束: *https://hudi.apache.org/docs/key\_generation*
`[17]` 这个 RFC: *https://cwiki.apache.org/confluence/display/HUDI/RFC+-+22+%3A+Snapshot+Isolation+using+Optimistic+Concurrency+Control+for+multi-writers*
`[18]` **Lakehouse 并发控制：我们是否过于乐观？**: *https://hudi.apache.org/blog/2021/12/16/lakehouse-concurrency-control-are-we-too-optimistic/*
`[19]` Apache Hudi灵活的Payload机制: *https://developer.aliyun.com/article/909719*
`[20]` Snapshot Isolation using Optimistic Concurrency Control for multi-writers: *https://cwiki.apache.org/confluence/display/HUDI/RFC+-+22+%3A+Snapshot+Isolation+using+Optimistic+Concurrency+Control+for+multi-writers*