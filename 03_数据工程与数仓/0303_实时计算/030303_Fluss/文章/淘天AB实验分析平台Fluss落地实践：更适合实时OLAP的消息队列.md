---
title: 淘天AB实验分析平台Fluss落地实践：更适合实时OLAP的消息队列
author: Apache Flink
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247514211&idx=1&sn=523e12c5c117f40c254839c4040910cf&chksm=fc8d7f58a02538197548410481683d34cf57221093206a94ed160d0beeb5348feee16c5c5752&mpshare=1&scene=24&srcid=0717s6S0Da66T4LFjtBikNo8&sharer_shareinfo=dcd56f1c03c746e075c57d8e892936ca&sharer_shareinfo_first=dcd56f1c03c746e075c57d8e892936ca#rd
---

**摘****要****：**本文投稿自淘天集团数据开发工程师张鑫宇、高级数据开发工程师王利雷老师。内容分为以下六个部分：

1. 业务背景
2. 业务痛点
3. Fluss核心能力
4. 架构演进与能力落地
5. 收益与总结
6. 规划

Tips：****点击**「阅读原文」**跳转阿里云实时计算 Flink～****

**01**

**业务背景**

淘天AB实验分析平台主要场景

淘天 AB 实验分析平台（下述统称为通天塔）主要专注于淘天内C端算法的AB数据，期望能用通用化的AB数据能力建设，促进科学决策活动，从 2015 年诞生以来，已经连续10年有效的支撑淘天算法AB数据的分析，目前接入包括搜索、推荐、内容、用增、营销等业务场域的AB场景 100+。

淘天AB实验分析平台架构

通天塔提供了以下的能力：﻿

* AB数据公共数仓：服务于下游算法各种数据应用，包括：线上分流、分布对齐、通用特征、场景label等各类应用场景。
* 科学实验评估：落地业界成熟的科学评估方案，长周期跟踪AB实验效果，助力业务得到真实、置信的实验评估结果。
* 多维即席OLAP自助分析： 通过成熟的数据方案，支持多维、即席的OLAP查询，服务全端、跨业务、跨场景的数据效果分析。

**02**

**业务痛点**

目前通天塔的实时数仓基于 Flink、消息队列、OLAP 引擎等技术栈，其中消息队列为集团内 TT（类Kafka架构MQ），OLAP 引擎使用阿里云 Hologres。

实时数仓架构

我们消费访问日志采集的消息队列数据后，在 Flink 中进行业务逻辑处理，然而，当 SQL 比较复杂，尤其是存在 Order by、Join 操作时，会导致 Flink 处理的回撤流翻倍，Flink 状态非常庞大，使用大量的计算资源。这种场景在任务的开发和维护方面可能会带来较大的挑战。任务的开发周期相较离线的方案也会长很多。目前消息队列还存在一些局限性，主要遇到的问题如下：

## 2.1 数据消费冗余

在数仓中，一写多读是常见的操作模式，并且每个消费者通常只消费数据的一部分。例如，在通天塔曝光作业中，消费的消息队列提供了 44 个字段，我们仅需要其中 13 个字段。由于消息队列是行存，当消费时，仍然需要读取所有列的数据。这意味着消费 70%的 IO 需要承担 100% 的成本。这种情况导致了网络资源的极大浪费。

在 Flink 中，我们尝试在Source中显式指定消费列Schema 和引入列裁剪 UDF 算子来降低数据源消费成本，但实践下来收效甚微且链路复杂度较高。

通天塔列裁剪算子示意

## 2.2 数据探查困难

### 2.2.1 消息队列不支持数据点查

在数仓建设中，数据探查是基本业务需求，用于问题定位、case排查等，在生产实践中探索出两种消息队列数据探查方式，均有优劣势，无法综合满足业务需求:

消息队列探查图

### 2.2.2 Flink State数据不可见

在电商场景中，统计用户当日首次、末次渠道是衡量用户获取效果和渠道质量的关键指标。为了确保计算结果的准确性，计算引擎必须执行排序去重操作，通过 Flink State 来物化上游的全部数据，但是其中 State 是内置的黑盒，我们看不见摸不着，修改作业或者排查问题难上加难。

Flink 排序任务

## 2.3 大 State 作业运维难

在电商场景中，订单归因是核心业务逻辑，在实时数仓中，常需要通过双流 Join （点击流、成交流）+  Flink State （24H）来实现当天成交归因的业务逻辑。由此带来的问题是超大的Flink State（单任务高达 100TB），这其中包含排序、双流 Join  等操作。

成交归因State

在任务中，Flink 作业需要 State 维护中间结果集，当修改任务后，执行计划校验成功，那么从最新状态启动；只要执行计划校验失败，就要从 0 点重刷 State，耗时耗力；并且每次消费数据，排序、Join State都会更新。其中排序算子 State 可达 90TB，Join State 高达 10TB，庞大的 State ，带来了很多问题，包括成本高、作业稳定性差、Checkpoint超时、重启恢复慢等等。

成交归因Flink任务

**03**

**Fluss 核心能力**

## 3.1 Fluss 是什么

Fluss官方文档：https://fluss.apache.org/

GitHub地址：https://github.com/apache/fluss

Fluss架构

Fluss 是 Flink 团队研发的面向流分析的下一代流存储，是一个为实时分析而构建的流存储。Fluss 创新性地将列存格式和实时更新能力融合进了流存储中，并与 Flink 深度集成，帮助用户构建高吞吐量、低延迟、低成本的流式数仓。其具备如下核心特性：

* 实时读写：支持毫秒级的流式读写能力。
* 列式裁剪：以列存格式存储实时流数据，通过列裁剪可提升 10 倍读取性能并降低网络成本。
* 流式更新：支持大规模数据的实时流式更新。支持部分列更新，实现低成本宽表拼接。
* CDC订阅：更新会生成完整的变更日志（CDC），通过 Flink 流式消费 CDC，可实现数仓全链路数据实时流动。
* 实时点查：支持高性能主键点查，可作为实时加工链路的维表关联。
* 湖流一体：无缝集成 Lakehouse，并为 Lakehouse 提供实时数据层。这不仅为 Lakehouse 分析带来了低延时的数据，更为流存储带来了强大的分析能力。

## 3.2 表类型

Fluss底层架构

类型：分为日志表和主键表，日志表是列存的 MQ，仅支持 insert 操作，主键表可按照主键和指定 Merge 方式进行更新。

分区：按照指定列将数据划分为更小、更方便管理的子集，Fluss 支持更加丰富的分区策略，如 Dynamic create partitions。更多分区的最新文档可以参考下：https://fluss.apache.org/docs/table-design/data-distribution/partitioning/，需要注意的是，分区类型必须是 String 类型，可通过如下 SQL 定义

```
CREATE TABLE temp(  dt STRING) PARTITIONED BY (dt)WITH (    'table.auto-partition.num-precreate' = '2' -- 提前创建2个分区    ,'table.auto-partition.num-retention' = '2' -- 保留前2个分区    ,'table.auto-partition.enabled' = 'true' -- 自动分区);
```

分桶：读写操作的最小单元，对于主键表，根据每条数据主键的哈希值进行确定该条数据属于哪个分桶。对于日志表，可以在创建表时，在 with 参数中进行指定列哈希的配置，否则就会随机打散。

### 3.2.1 日志表（Log Table）

日志表是 Fluss 中常用的一种表，按照写入顺序写入数据，仅支持 insert，不支持 update/delete，类似于 Kafka、TT 等 MQ。对于 Log 表目前大部分的数据都会被上传到远程，在本地只会存一部分数据，例如 128 个 bucket，这样本地就只会放 128 \* 2 （本地保留的 segment 数）\* 1（一个 segment 的大小）\* 3 （副本数） GB = 768 GB。建表语句如下

```
CREATE TABLE `temp` (  second_timestamp  STRING    ,pk               STRING    ,assist           STRING    ,user_name        STRING)with (  'bucket.num' = '256' -- 分桶数    ,'table.log.ttl' = '2d' -- TTL设置，默认7天    ,'table.log.arrow.compression.type' = 'ZSTD' -- 压缩模式，目前默认添加    ,'client.writer.bucket.no-key-assigner' = 'sticky'--分桶模式);
```

在 Fluss 中，Log 表默认以 Apache Arrow 列格式存储。这种格式可以逐列存储数据，而不是逐行存储数据，从而实现了列裁剪。这样可以保证在实时消费过程中，也可以只消费所需的列，从而减少 IO 开销并提高性能减少使用的资源。

### 3.2.2 KV 表（PrimaryKey Table）

相比较日志表，主键表支持 insert、update、delete 操作，通过指定 Merge 引擎来实现不同的合并方式，需要注意的是，bucket.key 、partitioned.key 需要为 primary.key 的子集，如果该 KV 表用于做 DeltaJoin，bucket.key 需要为primary.key 的前缀。建表语句如下：

```
CREATETABLE temp (  ,pk               STRING  ,user_name        STRING  ,item_id          STRING  ,event_time       BIGINT  ,dt               STRING  ,PRIMARY KEY (user_name,item_id,pk,dt) NOT ENFORCED) PARTITIONED BY (dt)WITH ('bucket.num'='512'  ,'bucket.key'='user_name,item_id'  ,'table.auto-partition.enabled'='true'  ,'table.auto-partition.time-unit'='day'  ,'table.log.ttl'='1d'-- binlog保留1天  ,'table.merge-engine'='versioned'-- 拿最后一条数据  ,'table.merge-engine.versioned.ver-column'='event_time'-- 排序字段  ,'table.auto-partition.num-precreate'='2'-- 提前创建2个分区  ,'table.auto-partition.num-retention'='2'-- 保留前2个分区  ,'table.log.arrow.compression.type'='ZSTD');
```

其中，merge-engine 支持 versioned、first\_row；使用versioned 后，需要限制ver-column 进行排序，目前只支持 int、bigint、timestamp 等类型，不支持 string；使用first\_row 后，仅支持保留每个主键的第一条记录，不支持按列排序。

## 3.3  核心功能

### 3.3.1 可裁剪的列式存储

Fluss 是一个基于列的流式存储，底层文件存储采用 Apache Arrow IPC 流式格式。这使得 Fluss 能够实现高效的列裁剪，同时保持毫秒级的流式读写能力。

Fluss 的一个关键优势在于，列裁剪在服务端执行，只有需要的列才会传输到客户端，这种架构不仅提升了性能，还降低了网络成本和资源消耗。

行列存储消费对比

### 3.3.2 KV存储的消息队列

Fluss 的 KV 存储的核心构建于日志表 (Log Table) 之上，并在日志上构建键值 (KV) 索引。KV 索引采用 LSM 树实现，支持大规模实时更新，并支持部分更新，可以高效地构建宽表。此外，KV 生成的变更日志可以直接被 Flink 读取，无需额外的去重成本，从而节省了大量的计算资源。

Fluss 内置的 KV 索引支持高性能主键查找。用户还可以使用 Fluss 进行直接数据探索，包括使用 LIMIT 和 COUNT 等操作的查询，从而轻松调试 Fluss 中的数据。

Fluss KV存储点查

### 3.3.3 双流 Join → Delta Join

在 Flink 中，双流 Join 是一个非常基础的功能，常用于构建宽表。然而，这也是一个常常让开发感到头疼的功能。因为双流 Join 需要在 State 中维护上游全量的数据，这导致其状态通常非常庞大。这带来了很多问题，包括成本高、作业不稳定、Checkpoint超时、重启恢复慢等等。

Fluss 开发了一个名为 Delta Join 的全新 Flink 连接运算符实现，它充分利用了 Fluss 的流式读取和Prefix Lookup 查找功能。Delta Join 可以简单理解为“双边驱动的维表 Join”，左边来了数据，就根据 Join Key 去点查右表，右边来了数据，就根据 Join Key 去点查左表。全程就像维表 Join 一样不需要 State，但是实现了双流 Join 一样的语义，即任何一遍有数据更新，都会触发对关联结果的更新。

双流 Join → Delta Join

### 3.3.4 湖流一体

在 Kappa 架构中，因生产链路差异，数据会在流、湖中分别存储，造成成本浪费，同时需要额外定义数据服务来统一数据消费。湖流一体的目标在于将“湖数据”和“流数据”能够作为一个整体进行统一存储、管理和消费，从而避免数据的冗余和元数据不一致问题。

Fluss 湖流一体

此外，Fluss提供了UnionReads来全托管式实现Kappa架构中依赖的统一数据服务，Fluss 维护了一个 Compaction Service，该服务会自动地将 Fluss 数据转换为湖存储的格式，并确保湖流元数据的一致性。此外，Fluss还提供了分区对齐、Bucket对齐机制，保证湖流数据分布一致。这使得在流转湖的过程中，无需引入网络 Shuffle，只需将 Arrow 文件直接转换为 Parquet 文件即可。

**04**

**架构演进与能力落地**

## 4.1 架构演进

Fluss 创新性地将列存格式和实时更新能力融合进了流存储中，并与 Flink 深度集成。基于 Fluss 的核心能力，我们将实时架构进一步演进，通过 Fluss 构建高吞吐、低延迟、低成本的湖仓，以下以通天塔典型升级场景为例，展开介绍Fluss的落地实践。

### 4.1.1 常规任务演进

通天塔常规任务升级图

以上为作业升级前后的架构，对于Source->ETL清洗->Sink 这类常规任务，由于消息队列是行存，消费时 Flink 会先将整行数据加载到内存中，再进行所需列的过滤，造成 Source IO 的极大浪费。升级Fluss后，由于 Fluss底层的列式存储，Fluss 的列裁剪是在服务端进行的，这意味着发送给客户端的数据已经是裁剪过的，从而节省了大量的网络成本。

### 4.1.2 排序任务演进

通天塔排序任务升级图

在 Flink 中，排序的实现依赖Flink显示计算并使用 State来保存数据中间态，这种模式带来了极大的业务消耗、极低的业务复用能力。Fluss 的引入将这个计算和存储下推至Sink侧， 搭配Fluss Merge引擎实现不同去重方式下的KV表，目前支持 FirstRow Merge Engine（第一条）和 Versioned Merge Engine（最新一条），去重中生成的Changelog 可以直接被 Flink 流读取，节省了大量计算资源，实现了数据业务的快速复用落地。

### 4.1.3 双流 Join 任务演进

通天塔双流Join升级图

通天塔成交归因双流 Join 任务全面落地Fluss改造后升级点如下：

* 列裁剪真正前置至Source消费上，避免无用列的 IO消费。
* 引入Fluss KV表+Merge引擎实现数据排序，剔除Flink排序State依赖。
* 改造双流JOIN为FlussDeltaJoin，使用流读+索引点查，外置Flink双流JoinState。

综合对比传统双流JOIN与基于Fluss新架构，两者优劣如下：

### 4.1.4 湖任务演进

通天塔湖仓架构升级图

在 Fluss 湖流一体架构下，Fluss提供了一种全托管式统一数据服务，Fluss 和 Paimon 各自存储流、湖数据，对计算引擎（如 Flink）输出一个 Catalog，数据以一张统一表的形式对外输出，消费方可以直接以UnionReads的方式访问 Fluss 和湖存储中的数据。

## 4.2 核心能力落地

### 4.2.1 列裁剪能力

在消费消息队列的任务中，消费者通常只消费数据的一部分，但是 Flink 任务仍然需要读取所有列的数据，Flink Source IO 出现了很大的浪费，究其根本，现有的消息队列均是行存，对于需要处理大规模数据的场景来说，行存格式的效率则显得不足。需要底层存储具备强大的Data Skipping 能力，以及支持列裁剪等特性。在这种情况下，拥有列存储的 Fluss 显然更为适合。

Fluss列裁剪演进

在我们实时数仓中，70%的任务均只消费 Source 的部分列，以通天塔推荐点击来说，Source 的 43 个字段我们仅需要 13 个，需要额外的算子资源来对整行数据裁剪，浪费了 20%+ 的 IO 资源。使用 Fluss 后，会直接消费所需列，避免额外的 IO 浪费同时也减少 Flink 列裁剪算子带来的额外资源。截止目前，淘天搜推场域已有多个核心任务上线 Fluss，并在淘天 618 大促中得到验证。

列裁剪作业对比

### 4.2.2 数据实时探查

* 数据点查

无论是排查问题还是进行数据探索，都需要进行数据查询。但消息队列仅支持在界面中抽样查询和在额外存储中查询同步的数据。在抽样查询中，无法查询指定数据，只能拉一批输出展示查看，以同步额外存储这种方式，又会带来分钟级的延迟和额外的存储、计算成本。

Fluss、消息队列 探查对比

在 Fluss KV Table 中，构建了 KV 索引，因此可以支持高性能的主键点查，可以通过点查的 query 语句直接探查 Fluss 数据，还支持 LIMIT、COUNT 等查询功能，以满足日常的数据探查需求，示例如下

Fluss查询示例

* Flink State 点查

Flink 的 State 机制（如 KeyedState 或 OperatorState）提供了高效的状态管理能力，但其内部实现对开发者而言是“黑盒”。开发者无法直接查询、分析或调试 State 中的数据，导致业务逻辑与状态管理强绑定，难以动态调整或扩展。当出现数据异常时，只能通过结果数据倒推状态中的内容，无法直接访问 State 中的具体数据，导致故障排查成本高。

Flink State演进

我们将 Flink State 外置到 Fluss 中，例如双流Join、数据去重等State，基于Fluss的KV索引，提供State探查能力，白盒化State内部数据，高效定位问题。

### 4.2.3 Merge Engine + Delta Join

在目前实时数仓中， 成交归因任务是我们使用 State 非常重的作业， State 高达 100TB ，这其中包含排序、双流 Join  等操作。我们消费 TT 的数据后，首先进行数据排序，再进行双流 Join，用来将订单数据归因。

成交归因任务State

如图，是归因任务中，进行首次归因逻辑实现中的 State。排序算子的 State 高达 90TB ，双流 Join 算子为 10TB，大 State 作业带来了很多问题，包括成本高、作业不稳定、CP 时间长等问题。

* 排序优化，Merge Engine

Fluss 排序优化

在 Merge 引擎实现中，主要是依赖 Fluss 的 KV 表，KV 生成的 Changelog 可以被 Flink 流读取，无需额外的去重操作，节省了 Flink 的计算资源，实现了数据的业务复用。通过 Fluss 的 KV 表，实现我们作业中的排序逻辑，从此在作业中，无需再维护排序算子的 State。

* Join 优化，双边驱动

Join作业对比

Delta Join 的任务如上，原有任务，会消费到数据后，按照业务诉求，进行排序，完成排序后进行双流 Join，并保存 24 小时 State。这样带来的结果是，CP 时间长、任务可维护性差、修改任务 State 需要重跑等问题。而在 Fluss 中，通过 KV 表的 Merge 引擎完成数据的排序，通过 Delta Join，解耦作业与状态，修改作业不需要重跑 State，并将状态数据可查，提高灵活性。

我们使用成交归因作业作为示例，需要注意的是，DeltaJoin 的双边都需要为 KV 表。

```
-- 创建左表CREATETABLE `fluss`.`sr_ds`.`dpv_versioned_merge`(   pk              VARCHAR  ,user_name       VARCHAR  ,item_id         VARCHAR  ,step_no         VARCHAR  ,event_time      BIGINT  ,dt              VARCHAR  ,PRIMARY KEY (user_name,item_id,step_no,dt) NOT ENFORCED)PARTITIONED BY (dt) WITH (      'bucket.num'='512'       ,'bucket.key'='user_name,item_id'      ,'table.auto-partition.enabled'='true'      ,'table.auto-partition.time-unit'='day'      ,'table.merge-engine'='versioned'-- 拿最后一条数据      ,'table.merge-engine.versioned.ver-column'='event_time'-- 排序字段      ,'table.auto-partition.num-precreate'='2'-- 提前创建2个分区      ,'table.auto-partition.num-retention'='2'-- 保留前2个分区      ,'table.log.arrow.compression.type'='ZSTD');-- 创建右表CREATETABLE `fluss`.`sr_ds`.`deal_kv`(  ,pk               VARCHAR  ,user_name        VARCHAR  ,item_id          VARCHAR  ,event_time       bigint  ,dt               VARCHAR  ,PRIMARY KEY (user_name,item_id,pk,dt) NOT ENFORCED) PARTITIONED BY (dt)WITH (  'bucket.num'='48'  ,'bucket.key'='user_name,item_id'  ,'table.auto-partition.enabled'='true'  ,'table.auto-partition.time-unit'='day'  ,'table.merge-engine'='first_row'-- 拿最后一条数据  ,'table.auto-partition.num-precreate'='2'-- 提前创建2个分区  ,'table.auto-partition.num-retention'='2'-- 保留前2个分区  ,'table.log.arrow.compression.type'='ZSTD');-- 部分join逻辑select * from dpv_versioned_merge t1joinselect * from deal_kv t2  on t1.dt = t2.dt   and t1.user_name = t2.user_name   and t1.item_id = t2.item_id
```

部分操作如上代码所示。在从双流 Join 迁移到 Merge + Delta Join 后，成功减免了大状态，使得作业运行更加稳定，CP 也不再超时。同时 CPU 、Memory 实际使用也发生了降低。

除了资源的减少和性能的提升，对于我们的收益还有灵活性的提升。传统的流到流连接的状态与 Flink 作业紧密耦合，如同一个不透明的“黑匣子”。当修改作业后，发现历史资源计划和目前作业不兼容，只能从 0 点开始重跑 State，耗时耗力。在使用 Delta Join 后，相当于状态与作业进行了解耦，修改作业不需要重跑State。并且数据都在 Fluss 里面，变得可查可分析，提升了业务灵活性和开发效率。

Fluss 历史数据消费

同时，Fluss 维护了一个 Compaction Service，将 Fluss 的数据同步到 Paimon 中，最新的数据在 Fluss 中，历史数据在 Paimon 中，Flink 可以支持 Union Read，将Fluss 和 Paimon 中的数据 Union 起来实现秒级新鲜度的分析。

Join任务演进

此外，Data Join 也解决了回追数据的场景，在双流 Join 中，如果修改任务导致执行计划校验失败，只能进行数据回补。而在 Fluss 回追过程中，可以利用湖流一体归档的 Paimon 表加上 Flink Batch Join 进行数据加速回填。

### 4.2.4 湖流演进

Fluss提供了对于数据湖仓的高兼容性，通过底层的 Service，自动将 Fluss 数据转化为 Paimon 格式数据，实时数据一键入湖，同时保证湖、流两侧数据分区一致和分桶一致。

Fluss 湖流演进

拥有湖和流两层数据后，Fluss 具有共享数据的关键特性。Paimon 存储长周期、分钟级延迟的数据；Fluss 存储短周期、毫秒级延迟的数据，两者的数据可以互相共享。

当进行 Fluss 流读取时，Paimon 可以作为历史数据提供高效的回溯能力。在回溯到当前位点后，系统会自动切换到流存储继续读取，并确保不会读取重复数据。在批查询分析中，Fluss 流存储可以为 Paimon 提供实时数据的补充，从而实现秒级新鲜度的分析，这种功能称为 Union Read。

**05**

收益与总结

经过一段时间对 Fluss 的深入调研、全面测试及平稳上线，我们成功完成了基于 Fluss 链路的技术架构演进。其中我们对 Fluss 列裁剪、Delta Join 等核心能力做了全面的测试，确保了链路的稳定性与性能达标。

## 5.1 性能测试

### 5.1.1 读写性能

我们对 Fluss 的读写、列裁剪能力进行了测试。其中，在读写性能上，我们通过相同的流量进行测试，在保持输入 RPS 为 800 w/s，输出 RPS 44 w/s 的情况下，通过对比 Flink 任务实际使用的 CPU、Memory 进行比对。具体如下

### 5.1.2 列裁剪性能

在列裁剪能力上，我们也通过测试 TT、Fluss 以及 Fluss 不同数量的列，来探寻 CPU、Memory、IO 的消耗情况。在保持输入 RPS 为 25 w/s 的情况下，测试的结果具体如下：

此时，我们会发现一个问题，随着列减少（共消费 13 列 of 43 列），IO 并没有出现线性的减少。经过校验，source 中的各个列的存储并不是均匀分布，其中部分所需列的存储占比 72%。由此可见，输入 IO 流量减少 20%，与读取列存储占比符合预期。

### 5.1.3 Delta Join 性能

我们也对上述双流 Join 和 Fluss Delta join 的资源和性能做了测试与比较，在保持左表输入 RPS 为6.8w/s，右表输入 RPS 为0.17w/s 的情况下，所有任务运行 24H。

经过实测，从双流 Join 迁移到 Merge + Delta Join 后，成功减免了 95TB 的大状态，使得作业运行更加稳定，CP 时间也大幅缩短。同时 CPU 、Memory 实际使用也降低了 80%+。

### 5.1.4 数据回追性能

因为 Delta Join 无需维护状态，因此可以使用批模式 Batch Join 来回追，追上以后再切回流模式 Delta Join，中间会重复处理一点数据，通过结果表的幂等更新机制保证了端到端一致性。我们将上述双流 Join 数据回追和 Batch Join 数据回追一天（24H），进行了测试对比

经过实测，批模式 Batch Join 回追一天（24H），相较于双流 Join 回追， 耗时减少 70%+。

## 5.2 总结

Fluss 具有列式裁剪、流式的更新、实时点查、湖流一体等核心特性，同时创新性地将列存格式和实时更新能力融合进了流存储中，并与 Flink 、Paimon深度集成，构建高吞吐、低延迟、低成本的湖仓。正如 Fluss 的初衷：面向分析的实时流存储。

经过近三个月的探索，Fluss 已在淘天核心搜推场景落地，构建湖流一体的AB数仓，并通过淘宝 AB 实验平台（通天塔）持续服务内部业务，在实际应用中，得到如下结果：

* 场景广泛落地：覆盖淘天核心业务搜索、推荐等，同时成功通过淘天 618 大促考验，流量峰值千万级，平均延时 1s 内。
* 列裁剪资源优化：列裁剪功能落地全量Fluss实时作业，消费列存储占比源数据 75%情况下，IO 平均降低 25%，任务资源消耗平均降低 30%。
* 大 State 作业优化：以成交归因作业为例，应用 Delta Join+Merge 引擎重构首次、末次归因。实现State外置， CPU 、Memory 实际使用降低 80%+，将Flink的状态与作业进行了解耦，成功减免 100TB+ 大状态，作业更稳定。
* 数据探查：落地了包括消息队列探查、State探查在内的能力，支持LIMIT、COUNT 等查询算子，实现了排序、双流Join的State 白盒化，灵活度更高、问题定位更高效。

**06**

**规划**

在接下来的工作当中，会沉淀新一代的湖仓数据架构，并在 Data + AI 场景中持续探索。

* 场景普惠：在秒级数仓中，将消费消息队列全链路、全场景切换为 Fluss 链路。基于 Fluss 湖流能力，提供对应的 Paimon 分钟级数据。
* AI 赋能：尝试多模态数据管道、Agent、模型预训练的场景的落地并附加 AI 增强分析、
* 能力探索：探索更多 Fluss 能力并赋能业务，通过 Fluss Aggregate Merge Engine、Partial Update 能力优化更多任务。

# 参考文献

[1] Fluss 官方文档：https://fluss.apache.org/

[2] [Fluss：面向实时分析设计的下一代流存储](https://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247512207&idx=1&sn=e25320a020d7b20e25be8fd614e9f46b&chksm=fd383ecdca4fb7dbf9a37e9c3840699ba8b1b2c57868db028f995c3bc7bcf6a0340e396d87c4&scene=21&cur_album_id=3764887437743669257&search_click_id=#wechat_redirect)

作者介绍

**张鑫宇｜花名：复杂**

淘天集团数据开发工程师，主要负责淘天AB实验分析平台流式架构升级中的Fluss、Paimon架构升级。

**王利雷｜花名：无岂**

淘天集团高级数据开发工程师，主要负责淘天AB实验分析平台的业务迭代、流式架构升级、湖仓一体演进。

**活动推荐**

---

阿里云基于 Apache Flink 构建的企业级产品-实时计算 Flink 版现开启活动：

新用户复制下方链接或者扫描二维码即可0元免费试用 Flink + Paimon

了解活动详情：https://free.aliyun.com/?pipCode=sc

---

---

▼ 关注「**Apache Flink**」 ▼

回复 FFA 2024 获取大会资料

**点击「阅****读原文****」****跳转阿里云实时计算 Flink～**