> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040103_StarRocks/040103_核心知识点/StarRocksPrimaryKey事务与DelVector读写边界|StarRocksPrimaryKey事务与DelVector读写边界]]
---
title: StarRocks 技术内幕：实时更新与极速查询如何兼得
author: StarRocks
date:
url: http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485927&idx=1&sn=56051046f9eb51a563c6c24eb22c9364&chksm=e9f176c3de86ffd5bab12e9b8657ad58c2b4b4d2741dc0f7de292c800e4ec23cb064955013d0&mpshare=1&scene=24&srcid=1226HDhzxOeUTvpFSwFj34re&sharer_shareinfo=9fdcf9865b66a8eab5b58915bdded420&sharer_shareinfo_first=9fdcf9865b66a8eab5b58915bdded420#rd
---

实时数据的分析对企业数字化运营和决策已然至关重要，因此很多用户构建了实时数据分析平台。为了对业务各类“变更”进行实时分析、快速响应业务变化，**实时数据更新**成了实时分析的核心要求。很多用户在进行实时数据更新时，查询性能不够理想，大大降低了业务分析效率。

和其他行业领先 OLAP 数据库不同，**StarRocks 设计和实现了 Primary Key 模型**，让数据可以更好地实时更新，并且具备极速的查询能力。当前 Primary Key 模型已经打磨了一年多，基于全新的设计和实现，**在大规模实时数据进行写入时，查询性能可以做到其他行业领先 OLAP 数据库的 3-5 倍**。

当前 StarRocks 已经在**超过 110 家大型用户的核心业务场景下进行大规模使用**，其中很多用户已成功采用 Primary Key 模型提升了实时分析能力，如腾讯、Airbnb、顺丰、众安保险等。

本文就将介绍 StarRocks 2.x 版本中实现的 Primary Key 模型的技术内幕，包括实时数据更新机制，以及针对实时更新数据的极速查询的实现原理。

**#01**

**背景**

**—**

随着实时数据分析场景的发展，对实时可更新数据的分析需求也越来越多，例如：

* TP 系统通过 CDC 实时同步数据 Binlog 到 AP 系统，可以非常方便地实时对业务数据“变更”进行分析，这是最常见的场景。
* 传统的 OLAP 数据流采用 ETL (Extract-Transform-Load) 的形式，数据充分加工后再导入 AP 系统，而随着现代 AP 系统的能力提升，ELT (Extract-Load-Transform) 形式的数据流逐渐流行。

  即把原始或者粗加工的数据先导入 AP 系统，在 AP 系统内部使用 SQL 对数据进行各种加工转换后再查询。

  这种方式有很多好处：统一使用一套系统开发维护成本更低；相比使用其他计算框架处理 ETL，SQL 易用性更好；最后在同一个 AP 数据库系统内通过事务机制可以更加简单便利地保证数据的一致性。

  而要支持实时 ELT 中的各种数据加工处理逻辑，数据库对于实时更新的支持也变成了强需求。

**#02**

**技术方案**

**—**

AP 数据库系统中通常使用列存作为底层的存储引擎，通常一个表包含多个文件（或 Rowset），每个文件使用列存格式组织（例如 Parquet），并且是不可修改的。

在这种组织结构的前提下支持更新，常见的技术方案有下面几种：

**Copy-on-Write**

当一批更新到来后，需要检查其中每条记录跟原来的文件有无冲突（或者说 Key 相同的记录）。对有冲突的文件，重新写一份新的、包含了更新后数据的。

这种方式读取时直接读取最新数据文件即可，无需任何合并或者其他操作，查询性能是最优的，但是写入的代价很大，因此适合 T+1 的不会频繁更新的场景，不适合实时更新场景。

很多数据湖都使用了这种方式实现更新，例如 Delta Lake、Hudi 的 Copy-on-Write 表等。

**Merge-on-Read**

当一批更新到来后，直接排序后以列存或者行存的形式写入新的文件。由于数据在写入时没有做去重或者说冲突检查，就需要在读取时通过 Key 的比较进行 Merge 的方式，合并多个版本的数据，仅保留最新版本的数据返回给查询执行层。

这种方式写入的性能最好，实现也很简单，但是读取的性能很差，不适合对查询性能要求很高的场景。采用这种方案的包括 Hudi 的 Merge-on-Read 表，StarRocks 的 Unique 模型表，ClickHouse 的实时更新模型等。

**Delta Store**

当一批更新到来后，通过主键索引，先找到每条记录原来所在的文件和位置（通常是一个整数的行号）。把位置和所做的修改作为一条 Delta 记录，放到跟原文件对应的一个 Delta Store 中。查询时，需要把原始数据和 Delta Store 中的数据合并（或者说应用 Delta 数据）。

由于 Delta 数据是按照行号组织的，合并过程直接按照行号进行合并即可，和 Merge-on-Read 的按照 Key 进行合并相比，查询性能好很多。这种方式由于写入时要查询索引，写入性能稍差，但是读取的性能要好很多。相当于牺牲了一些写入性能换来读性能。另外由于引入了主键索引和 Delta Store，复杂性提高。采用这种方案的典型系统就是 Kudu，当然还有一些内存数据库 TP 系统也常用这种方案。

**Delete-and-Insert**

当一批更新到来后，通过主键索引，先找到每条记录原来所在的位置，把该条记录标记为删除，然后把最新数据作为新记录写入新文件。读取时，根据删除标记来将旧版本过期数据过滤掉，留下最新更新后的数据。

因为无需像 Merge-on-Read 和 Delta Store 模式下进行 Merge，另外过滤算子可以下推到 Scan 层直接利用各类索引进行过滤减少扫描开销，所以查询性能的提升空间更大。

和 Delta Store 类似，此模式也牺牲了一些写入性能换取读性能，当前业界也有一些应用：比如 SQL Server 的 In-memory 列存等。Delete-and-Insert 模式需要引入主键索引以及存储和管理删除标记，在大规模实时更新和查询场景下实现难度非常大。

当前 StarRocks 的 Primary Key  模型主要采用了 Delete-and-Insert 模式，并且进行了很多新的设计，以支持在大规模实时数据更新时提供极速的查询性能。

**#03**

**StarRocks 实时更新实现**

**—**

StarRocks 全新添加了一种新的表模型: Primary Key，该模型的表支持按照主键对一行或多行进行更新(Upsert)或者删除(Delete)操作。

StarRocks 对表的操作通过导入(Load)的形式载入系统。每个导入任务可以看作一个单独的写入事务，StarRocks 保证导入事务的 ACID 属性。也就是说，对于一个跨多个 Tablet 的大批量导入事务，导入也是原子生效的，并且保证并发导入事务之间的隔离性。

**事务整体流程**

如下图所示， 一个导入事务可以分为两个阶段：Write , Commit.

* Write 阶段，客户端把该导入事物的数据按照分区分桶规则分发到对应的 Tablet，每个 Tablet 收到数据后把数据写成内部的列存格式，形成一个 Rowset；
* Commit 阶段，所有数据都写入成功后，FE 向所有参与的 Tablet 发起事务、提交 Commit，每个 Commit 会携带一个 Version 版本号，代表 Tablet 数据的最新版本。

**Tablet 内部结构**

Tablet 是 StarRocks 底层存储引擎的基本单元，每个 Tablet 内部可以看作一个单机列存引擎，负责对数据的读写操作。

为了支持实时更新，Primary 模型的单机存储引擎重新设计，与其他三个模型 (Duplicate, Aggregate, Unique) 原有的存储引擎已经有了明显的区别。如下图所示，其主要包含以下 4 个组件：

* Meta: 元数据，保存 Tablet 的版本历史以及每个版本的信息，比如包含哪些 Rowset 等。序列化为 Protobuf 后存储在 RocksDB 中，为了快速访问，也会缓存在内存中。
* Rowset: 一个 Tablet 的所有数据被切分成多个 Rowset，每个 Rowset 以列存文件的形式存储，其内部组织编码方式类似 Parquet 文件，但是是 StarRocks 特有的格式。
* Primary Index: 主键索引，保存主键到该记录所在位置的映射。目前完整维护在内存中，不持久化到磁盘，在导入进行时按需构建，一段时间没有持续导入时又会释放以节约内存。
* DelVector: 记录每个 Rowset 中被标记为删除的行，比如一个文件中有 10000 行数据，其中 200 行被标记删除，则可以使用一个 Bitmap 来标记被删除的行。一般都是很稀疏的，使用 RoaringBitmap 可以高效存储和操作，同样保存在 RocksDB 中，但也会缓存在内存中以便能够快速访问。

**元数据组织**

元数据的组织会有一个版本历史，每次 Commit 或者 Compaction 都会生成一个新版本，记录了该版本包含了那些数据，相比上一个版本做了那些改动。下图是一个元数据版本历史的例子:

可以看到每次 Rowset Commit 或者 Compaction Commit，都会生成一个新的版本，每个版本会记录该版本包含哪些 Rowset。

当然随着数据导入，版本历史会逐渐积累，而较老的版本不太会再被使用到，所以后台 GC 线程会定期删除老版本。随着老版本的删除，一些 Rowset 可能不会被任何版本引用，也可以一并删除，如下图所示：

**写入流程**

上文提到，一个写入事务的数据会根据分区分桶信息分发到相关的 Tablet，每个 Tablet 负责写入一部分数据，每个 Tablet 接收到这部分数据后，会执行一个写入流程，最终把这部分数据加入自己管理的数据集。下图展示了写入流程的各个阶段:

具体流程如下:

* 收到的各个批次数据在 MemTable 中积累，MemTable 满了之后，需要 Flush 到磁盘
* Flush 包含 Sort、Merge、Split 三个操作，Sort 先按主键对数据排序，Merge 对主键相同的多条记录进行合并，只保留最新版本的数据。Split 把这批操作中的 Upsert 和 Delete 操作拆分开来，写入不同的文件
* 该导入的所有数据发送完毕并且 Flush 后，会生成一个包含多个文件的新 Rowset
* FE 发起 Commit 流程，针对 Tablet 执行 Commit 流程，BE 包含 3 步:

1. 查找并更新 Primary Index，记录所有被更新的记录，标记为删除
2. 根据标记为删除的记录，生成 DelVector
3. 生成新版本的 Meta，与 DelVector 一起写入 RocksDB

**写入事务并发控制**

StarRocks 支持并发导入，多个导入事务间可能产成冲突，需要在并发控制解决冲突，每个导入事务分为两个阶段：Write 和 Commit.

* Write: 写入数据生成 Rowset 文件阶段，可以并行执行，占据事务的绝大部分时间
* Commit: FE 发起 Commit 后，查找主键索引，生成 DelVector 和元数据的操作，需要序列化执行，但只占整个导入的很小一部分时间，所以序列化执行对导入性能影响较小

导入遵循 Last Write Wins 原则，对于并发导入事务中相同 Key 的记录冲突，版本大的记录覆盖版本小的记录。对于并发事务中写写冲突的处理方式， 一般有两种：

* 一种是悲观的加锁方式，缺点是有加锁的开销，并且行级别的锁对超大写事物不太现实，另外加锁也会影响并发写入的性能。
* 一种是乐观方式，写入时不检查冲突，在 Commit 时检查冲突，如果冲突再 Abort，AP 系统中的导入通常是超大事务，Abort 重新导入的代价太大也不理想。

StarRocks 并没有采用这两种经典的方式，其在事务冲突处理上是独特的。Commit 阶段序列执行，使用 DelVector 来解决冲突，避免了加锁的开销和性能影响。

写入冲突后也不需要回滚，只需要将冲突中老的记录标记为删除，非常适合导入数据这种导入任务可以按照纯写大事务进行处理的场景。

**主键索引**

Commit 阶段的最主要工作是查找和更新主键索引，约占整个 Commit 过程 90% 以上的时间。Commit 的速度主要取决于主键索引的性能，因此主键索引目前使用内存 HashMap 来实现，和计算层一样使用高性能 HashMap 库 phmap。HashMap 的 key 是多列编码后的 Binary 串，Value 是由(rowset\_id, rowid)组成的 64bit 整数，可以根据主键查找主键对应的行在哪个 Rowset 的第几行。

内存 HashMap 相比 RocksDB 等通用 KV 存储在性能上有很大优势，通常在现代 CPU 上每个操作耗时仅 20ns~200ns，也就是单核可以达到 5M-50Mop/s，能够保证批量导入的大事务快速完成 Commit。但是其最大的劣势是需要消耗内存。我们针对定长和变长类型的主键分别做了一些优化，以降低内存占用，未来会持续优化 HashMap 的内存占用，另外会探索实现基于 SSD/PMEM 的可持久化的主键索引。

主键索引目前是按需加载的，在有写入时会读取主键列构建，在长时间没有写入后会释放以节约内存。这非常适合有冷热数据模式的场景，即一条记录在创建后的一段时间内是热数据经常被更新，过一段时间后逐渐变冷不再更新。数据按天分区后，只有最近几天的分区才会有导入存在，这样最近几天的 Tablet 的主键索引才会被加载，整体内存使用量可控。

典型的场景比如各种订单场景，包括电商的订单、打车/共享单车订单等，或者应用或者设备的 Session 等。

**Compaction**

数据持续实时导入会导致存在大量小的 Rowset 文件，另外对数据的更新和删除也会使得 Rowset 中的大量记录被标记为删除，这些都会影响查询的读取性能，并且会浪费存储空间。

和 LSM 等存储引擎类似，StarRocks 也会在后台利用 Compaction 来优化数据的组织，但其具体机制和 LSM 也有很大不同：

* 首先在选择 Rowset 进行 Compaction 的策略上，会根据 Cost，优先选择小文件和大量记录被删除的文件。
* 由于在写入时已经去除了重复数据，存储中已经不存在重复的记录，所以 Compaction 不需要考虑重复记录的合并。
* 由于原生支持 Delete，Rowset 文件中也不存在 Delete 或者 RangeDelete 的 Tombtone 标记，所以处理上也更加简单。
* 但是由于引入了主键索引，Compaction 会使得记录的实际位置发生变化，需要额外维护主键索引的一致性，增加了复杂度。

Compaction 的流程和正常写入流程类似：

1. 根据选择策略，挑选 Compaction 的输入 Rowsets
2. 合并 Rowsets，生成新的 Rowset
3. Commit 该 Rowset，更新主键索引，生成 DelVector，在元数据中原子替换掉输入的 Rowsets

在 Compaction 的过程中，同样可能有并发的导入在执行，可能会与 Compaction 发生冲突。为了检测并解决冲突，我们会记录 Compaction 的原始 Rowset 的 ID，在 Commit 更新主键索引时，检查每一行对应 Rowset ID 是否发生过改变。如果发生过改变，意味着这一样已经被更新过，则忽略对这一行的更新，并把 Rowset 中对应的行标记为删除。

**读取流程**

相比其他设计和实现，Primary Key 表在读取性能上有很大优势，**多个因素综合下来，查询性能会有 3-5 倍以上的提升**，其中主要原因有:

* 去除 Merge 操作：由于历史重复记录已经在写入时标记“删除”，读取时不再需要 Merge 过程来去重以找到最新版本数据，对于很多查询来说，Merge 过程是整个查询流程的瓶颈，无需 Merge 可以很大提高性能。另外，去除 Merge 操作后整个执行树都可以向量化计算，查询性能也有极大提升。
* 谓词下推：其他很多系统里谓词下推无法下推到 Merge 操作以下。我们在去除 Merge 操作后，谓词下推可以直达低层 Rowset 的 Scan 操作。这样 Rowset 文件中的二级索引 (zonemap, bitmap, bloom) 才可以发挥作用，加速查询实现数倍的提升。
* 并行扫描：有多个 Rowset 文件，多个文件的 Scan 可以并行操作。

下图是一个查询使用 Merge-on-Read 模式 (比如 StarRocks 的 Unique key 模型和 ClickHouse 的实时更新模型) 和 Delete+Insert 模式 (StarRocks 的 Primary Key 模型) 的对比:

可见，使用 Primary Key 模式，省去了 Merge 操作的开销，省去了读取 order\_id 的 io, state==2 的谓词可以利用 Bitmap 索引加速，整个操作树传输的数据量也少了很多。

我们模拟了一个订单表的场景，测试对比 Unique 模型表和 Primary 模型表的性能。该表每天 1000 万订单，每个订单会被更新多次，我们持续导入了 20 天的数据。在边实时导入边查询的情况下，简单查询最好情况下会有 10 倍以上的性能提升；在暂停导入一段时间后单独查询的情况下，也会有 3 倍左右的性能提升。

**#04**

**未来工作**

**—**

当前支持的操作类型包括 Upsert 和 Delete，可以满足最常用的场景，包括 TP 系统 CDC 同步数据到 StarRocks 需求，正常的批量导入去重需求等。随着应用场景的复杂性提高，对 StarRocks 提出了各种新的需求，这些是我们下一步要大力投入的特性：

**Partial-Update（部分列更新）**

支持部分列更新，典型的场景是上游几个模块实时写入一个大宽表，每个模块仅更新表中的一部分列，在大宽表里自动完成 Join 的操作。

目前实现这种 Join 操作，一个方案是使用 T+1 的方式，在外部使用 Hive/Spark 等计算框架批量完成，无法做到实时；另一个方案是在 Flink 等流式系统中实时 Join 后再导入 AP 系统，限制较大，实现也非常复杂。

**Conditional-Update（条件更新）**

支持根据条件更新，导入数据满足一定条件才更新。典型的一个场景是：上游数据乱续到达，需要通过一个时间戳来判断是否需要更新。

**Read-Write Update（通用读写事务）**

部分列更新和条件更新操作本质上是一种读写事务，相比 Upsert / Delete 这种纯写事务，实现上要复杂一些。发生冲突后，简单的 Last-Write-Wins 策略不再适用，需要进一步处理。但是相比通用的读写事务，还是有一些特性可以利用，来优化冲突的检测和解决。

**ELT 更新**

ELT(Extract-Load-Transform)逐渐流行。先把原始数据导入 AP 系统，在系统内部使用各种 SQL 语句对数据进行进一步加工。这样不但可以加速查询，简化了开发和运维难度，还实现了在一个系统内管理数据的依赖关系。

在复杂的 ELT 数据流中，常会涉及对数据的批量更新。这通常是批量数据的大规模读写事务，传统 TP 系统的并发控制机制并不能很好解决大事务带来的挑战，需要有创新性的解决方案。

**作者**

**常冰琳** | StarRocks 核心研发、Apache Kudu PMC

**“极速统一” 数据分析新范式：**

[阿里云](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485289&idx=1&sn=6f65b0361f79aa9ef22b7cd38eaa7af2&chksm=e9f1784dde86f15b5ae2a593a43945b003c61f2d60e762eb4341fe5e7a6146f9df6192101453&scene=21#wechat_redirect)   [京东到家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485053&idx=1&sn=b9dfda8d35c27621c8012f3df54d9728&chksm=e9f17959de86f04f15f2d86a7df86e66192ee2cc38bdcafbc790c2a5149f96629fb79f511c76&scene=21#wechat_redirect)   [携程](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484378&idx=1&sn=53ba8ffc20d185932c6eaac9cfa5aa62&chksm=e9f17cfede86f5e8c60d02e7c494208b0fc9aa15b0449172e93b1fc09224ea8616703b1620b1&scene=21#wechat_redirect)   [小米](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484613&idx=1&sn=7704c22b16a36851b2392f14ef7cda70&chksm=e9f17be1de86f2f765a2352b67a2ff49ea2c89d0940a3b27529cb18b78e5a51bcf8308a182dd&scene=21#wechat_redirect)[搜狐](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485710&idx=1&sn=7438bc119f8e0cac3f7535a7c34539d8&chksm=e9f1762ade86ff3cbd4f4bf45b4d422cee972f4cf646e6822591e9213fb064a3e3ed8a3912bb&scene=21#wechat_redirect)

[DMALL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484971&idx=1&sn=8f7e7ec424335b81fb34f1da85efbf8f&chksm=e9f1790fde86f019016e67a8a820f866bcbffe484414fa93b403605f957874bb7450672bfc3d&scene=21#wechat_redirect)    [海尔云链](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485307&idx=1&sn=a64d53c06cc57b1367fafe4fdf936e3a&chksm=e9f1785fde86f1496291fb4ad54c662b63a4dca70eec2c173a35d928c88e41d3ac5475cd5742&scene=21#wechat_redirect)    [信也科技](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484926&idx=1&sn=09ceb7940091b9356609376747a36ac7&chksm=e9f17adade86f3cc3afa02f5b0ecc9bbbbc4072b56c27296eff80f6af815c8fd6c1af442a5be&scene=21#wechat_redirect)[众安](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485788&idx=1&sn=fd7f8f86d1a7ee02063a8ca7cba295cf&chksm=e9f17678de86ff6ed2af617780d227bf2b04df8f99c102df0acb507ae7373ec6ec6e8a31c200&scene=21#wechat_redirect)

[理想汽车](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485820&idx=1&sn=7aa47ec35feec64e49ea462fe7506864&chksm=e9f17658de86ff4e1bd61f12d70ee8b478cc0ecd8fa7860df72ce2b5dc4b8ac05c77235809d6&scene=21#wechat_redirect)    [汽车之家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484870&idx=1&sn=74ab3350c6b7e86376c009b3ee6b5b98&chksm=e9f17ae2de86f3f45cc32c4ae153b61c57cdf6e66dd7b10ae9fb8ec689994598130257c43936&scene=21#wechat_redirect)    [滴滴](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484326&idx=1&sn=3e038503ce8a81a74728b50f676a35ec&chksm=e9f17c82de86f5949d4f5c0622efe4b69d65edf6b7faeddbe30d410dcd2e84a6c95249ad4ff7&scene=21#wechat_redirect)   [华米](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485647&idx=1&sn=b9eddd4b269eb1545c144e8ad8ccfc27&chksm=e9f177ebde86fefdfb2a9d25729d5b7c4a3bd7ffe4da1d51297c4e5bb4a6a8bae93a6395880f&scene=21#wechat_redirect)   [360](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485899&idx=1&sn=c8b13a6a6f9d0d59de1721a36f52c4cd&chksm=e9f176efde86fff974e81dfe2fb72a834528141a9b0f4a99f647b11c1a2e9001822b82ddbd1e&scene=21#wechat_redirect)

**StarRocks 技术内幕：**

[大数据自动管理](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485727&idx=1&sn=548ed6a15bd4b938cae8551500e3275d&chksm=e9f1763bde86ff2dcea5aacf2e1d8b9401cee6236b41571e241de74178cd856842e4e62fb48a&scene=21#wechat_redirect)   [查询原理浅析](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485848&idx=1&sn=90e47d7a46eb120701d28b5acfbcc401&chksm=e9f176bcde86ffaaa0c4a3b686eea5b053ff286164cb9a27e85daf4e42bec08985f5a41975c3&scene=21#wechat_redirect)