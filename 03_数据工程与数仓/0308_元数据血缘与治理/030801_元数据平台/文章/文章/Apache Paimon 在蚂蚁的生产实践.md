---
title: Apache Paimon 在蚂蚁的生产实践
author: Apache Paimon
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkyNjQ1MDI3Mg==&mid=2247484232&idx=1&sn=d6505dd9759c399669ad8fb6f147e2ee&chksm=c2365447f541dd51c1cd4652d585739a64a956d772fa11cd7949bf059179e5b72a371123caf7&mpshare=1&scene=24&srcid=0111sv7vMMTQbfBcDZOaTGfB&sharer_shareinfo=644f4b1ce3483e768877028b503e0113&sharer_shareinfo_first=644f4b1ce3483e768877028b503e0113#rd
---

**01**

**背景**

在蚂蚁数据研发的体系中, 我们探索落地了一套基于 Flink 流批统一的计算架构, 但是在使用的过程中发现, 由于存储系统的割裂, 需要通过平台来做各种离线表, 实时表的源表映射来统一源表的处理, 用户理解成本和使用成本还是比较高, 难以推广和真正为用户减负。

因此, 我们开始探索基于开源数据湖的方案来补足存储上的这一块拼图. 我们调研了 iceberg/hudi/paimon 数据湖格式, 综合比较下来, 我们选择了基于 Paimon 的数据湖方案。

Paimon 数据湖格式主要有以下的几大特点:

* 流批统一, 支持批读批写, 流读流写
* 支持非常灵活的 Merge engine, 如 deduplicate, first\_row, aggregation, partial-update
* 支持灵活的change log producer, 能够提供完整的流读changelog
* 对 AppendOnly 表也有很好的小文件合并和索引优化支持
* 基于分布式文件系统存储, 低成本和可扩展的元数据

综合来看, Paimon 在流场景支持中做了很多工作, 同时对传统的批处理场景也对标现有其他湖格式的能力。

**0****2**

**业务场景**

总体架构如上, 主要包含五个部分, 数据入湖, 元数据管理, 离线计算入口, Flink SqlGateway 查询, 流读入口。 以下主要介绍两个实际使用数据湖优化开发体验和效率的场景。

#### 

#### ****直播用户特征序列****

在支付宝的直播场景中常见的一个需求是计算直播用户行为的序列特征,  用户序列特征是指用户对于端上推荐内容做出的互动行为，包括曝光、点击、点赞等的汇总序列特征。

拆解到数据侧，需要产出一份 user x item 粒度的汇总宽表数据，在引入 Paimon 之前，采用的计算方案是所有聚合、打宽逻辑完全在 Flink 作业中实现，如下图所示：

这个链路中主要有以下痛点：

* 超大状态：由于聚合粒度为 user x item，所以状态中维护Key的数据量接近原始明细数据量级别，导致Agg、Join等算子的状态非常大
* 性能较差：经常受上游流量冲高影响，使任务中超大状态算子成为性能瓶颈，导致任务反压延迟、吞吐下降，还会引发检查点失败等一系列问题
* 探查成本高：中间结果（状态）维护在状态后端中，对外不可查，排查数据需另起链路，人力成本投入过高

因此，希望充分利用 Paimon 的 LSM Tree 机制，用户可以通过指定 Merge Engine，将原先任务中实现的聚合、打宽逻辑下沉到 Compaction 过程中，为实时链路降本增效。基于Paimon优化后的链路，如下图所示：

可以看到在新链路中，Flink 任务变得更加轻量，原始链路中 Agg、Join 等超大状态算子都可以被省略。综合来看取得了以下收益：

* 业务逻辑复杂度降低：无需在任务中手动声明聚合、打宽逻辑，因此可大幅简化业务逻辑、提升开发效率
* 计算资源消耗降低：利用存储端合并期间的聚合能力，任务中状态存储与计算的开销大幅缩减，并且在流量高峰期, 可牺牲一定的时效性来换取较高的吞吐
* 探查成本降低：通过 Paimon 所提供的 audit\_log 系统表，业务同学可以快速查询某一主键的中间变更过程，当出现数据问题时，可快速回溯，使排查效率提升80%左右

## 

#### ****多表外键打宽****

另一个在内部使用的场景是基于外键打宽的方案尝试, 目前社区支持的还是两表基于同一主键的, 通过 LSM tree 合并的方式来实现的两表 join. 但是多表外键打宽在业务上实际上是很普遍的诉求。

传统离线场景中一般通过定期合并及多表关联实现大宽表, 计算代价很高, 时效性一般也是T+1. 同时我们在Paimon社区看到其他业务也在探索多表外键打宽的能力, 因此我们基于社区讨论的一个方案, 在 Paimon 上尝试解决这个问题。

以 a b c 三张表为例, 其中主键分别为 pk1, pk2, pk3, 每张表包含另一张表的外键, 如何高效的获取这三张表的宽表呢?

* a: pk1 pk2
* b: pk2 pk3
* c: pk3 xx

首先我们可以通过Flink流任务来实现这个需求, 可以通过多流 Join 的方式, 理想的情况下, 这个流就可以产出预期的打宽数据了, 但是 Flink Join 依赖本地状态存储, 而且每两张表的join都会重复存储本地状态, 表越多, 状态存储大, 性能越差。

```
SELECT *FROM aINNER JOIN b ON a.pk2 = b.pk2INNER JOIN c ON b.pk3 = c.pk3
```

在这个基础上, 我们可以通过多流join只保留主键的部分, 这样状态开销就会节省很多, 性能也会提升. 这样通过Flink流任务就可以以较低代价产出一张多表的主键变更关系流。

```
INSERT INTO pk_relation_tableSELECT a.pk1,b.pk2,c.pk3FROM aINNER JOIN b ON a.pk2 = b.pk2INNER JOIN c ON b.pk3 = c.pk3
```

然后下游流读这张 pk\_relation\_table, 和原始的单表进行基于主键的维表关联, 来补齐其他表的字段形成大宽表。

```
SELECT *FROM pk_relation_table AS xJOIN a ON x.pk1 = a.pk1JOIN b ON x.pk2 = b.pk2JOIN c ON x.pk3 = c.pk3
```

### 

整体链路如上, 这样做的好处是:

* 整个链路基于变更数据关联, 因此只有增量数据触发变更, 避免了周期性的全量关联
* 流式触发关联, 整体的时效性可以从 T+1降低到分钟级
* 原始数据只保留一份, 不需要在关联的过程中在状态中重复存储

不过这里还有个缺陷, 就是如果中间这个生成pk关系的 join 流任务重置, 就会重新下发历史数据, 那么消费这张pk relation table的下游表, 又会回放之前的变更记录, 导致下游全量重新 lookup 关联, 这部分计算开销是完全无用的, 而且会导致额外的开销

这里的一个解决方法是用户可以在下游的任务中, 通过过滤条件将历史已经完成关联的数据过滤, 不过可能不是那么优雅。

针对这个问题, 我们设计了另一种方案, 来优化上面 pk relation table 的重置的情况. 就是将中间的pk relation table也保存为一张 pk 表, 并指定其 sequence 键。

```
INSERT INTO pk_relation_tableSELECT pk1,pk2,pk3,sequence_1,sequence_2,sequence_3FROM aLEFT JOIN b ON a.pk2 = b.pk2LEFT JOIN c ON b.pk3 = c.pk3pk-relation-table 的 schemaCREATE TABLE PK_RELATION_TBALE (  `pk1` varchar,  `pk2` varchar,  `pk3` varchar,  `g_1` bigint,  `g_2` bigint,  `g_3` bigint,   primary key (`pk1`, `pk2`, `pk3`) NOT ENFORCED) with (    'merge-engine' = 'partial-update',    'fields.g_1.sequence_group' = 'g_1',    'fields.g_2.sequence_group' = 'g_2',    'fields.g_3.sequence_group' = 'g_3',    'chaneglog-producer' = 'lookup')
```

其中 g\_1, g\_2, g\_3 分别对应上游3张表的gmt\_modified时间, 即数据的版本. 对于上游的写入, 当任意一个流的版本变大时, 才会发送 changelog 的数据, 那么重置时也不会导致下游全部回刷重查了. 相当于利用Paimon sequence防降的能力, 当只有上游数据版本变大时才下发变更数据。

基于这个方案, 可以实现多表的流式增量关联打宽, 整体代价会远小于离线定期全量的打宽, 数据新鲜度也高于离线的方案. 目前还只是基于小数据量的表进行验证, 因为整体的方案最终还依赖于 Paimon 的维表关联的性能, 这块社区还在持续优化, 我们也共同期待。

**0****3**

**功能增强**

在内部场景的支持中, 我们也不断完善了 Paimon 了很多功能. 主要有：

**支持 session 级别动态参数设置**

支持通过 set 语句设置表级别的动态参数覆盖 catalog 中的参数, 实现在即席查询中参数动态生效的能力

**优化查询分区下推能力**

优化原始的 query filter 下推, 支持分区条件的完全下推, 避免分区limit query 扫描全分区的问题

**支持 Partition lookup join**  
支持 hash 分区的 Partition join 策略, 可以大大减小 Paimon 维表关联时需要缓存的数据量, 降低存储开销

**支持 partial insert union sink**

优化用户多表写入任务的体验, 只需要通过多个 partial insert 语句, 然后由 planner 自动优化插入 Union 节点,合并多个 sink 算子. 避免用户手动 union 以及 Cast null 的操作

**catalog 支持多存储后端设计**

内部使用的 catalog 会支持多个存储后端, 如基于 hdd 的 dfs, 基于 ssd 的 dfs, 以及 oss 等等, 因此我们设计了一个可以支持多存储后端的的 catalog, 通过 database 级别的参数动态获取存储后端的配置, 提升了管理上的灵活性

**支持 first\_row merge engine**

支持 first\_row 模式的 merge engine, 可以对 append 流进行去重处理, 类似于 Flink 中的 first row 去重, 其好处是通过一个去重表之后, 下游可以多个任务复用, 避免重复的去重开销

**支持 partial-update with aggregation**

支持多表合并的时候指定字段的聚合策略, 在1对多合并的场景中, 通过指定聚合策略, 能更灵活的满足业务上的诉求

以上绝大部分功能已贡献给社区, 也非常感谢 Paimon 社区对这些功能的接纳与建议。

**0****4**

**总结与展望**

经过半年的探索, 内部基于 Paimon 的数据湖产品基本完成落地, 开始服务部分内部业务, 在此也非常感谢 Paimon 社区的支持, 后续我们将继续针对内部业务场景进行优化, 主要方向包括:

* 跟进社区维表关联的优化, 提升外键打宽的适用范围
* 搭建数据湖管理平台和表管理服务
* 探索数据湖上索引的应用, 优化查询性能