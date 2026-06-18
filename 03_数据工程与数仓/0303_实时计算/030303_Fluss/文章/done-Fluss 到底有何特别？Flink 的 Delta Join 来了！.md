> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030303_Fluss/030303_核心知识点/Fluss Delta Join 与外部化状态边界|Fluss Delta Join 与外部化状态边界]]
---
title: Fluss 到底有何特别？Flink 的 Delta Join 来了！
author: Apache Fluss
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk1NzE1MjQxMg==&mid=2247483856&idx=1&sn=2b22ee39fb02c3a68f08d598569a3864&chksm=c207caf17acb6b9608a00896249c566c9c8b0085cfc57a3e461f88ff8786b9b0d7543dc1ab61&mpshare=1&scene=24&srcid=1119QUGCTV6p8Ja1WBYAJMZq&sharer_shareinfo=2382afc7a878bbb43227ab712686a9da&sharer_shareinfo_first=2382afc7a878bbb43227ab712686a9da#rd
---

**原文：https://medium.com/fresha-data-engineering/what-the-fuss-with-fluss-flink-delta-force-1ab3d6be5c98**

作者：Anton Borisov @ Fresha

2025年7月31日发布的 Flink 2.1 在流处理世界投下了一枚重磅炸弹：DeltaJoin 与 MultiJoin。它们可不是普通的 Join 算子，而是专门用于解决生产环境中长期存在的状态膨胀问题的。

用过 Flink/Fluss 后，你会爱上水獭（Fluss）

每个 Flink 团队都心知肚明那个“秘密”：Join 状态膨胀的速度，比伦敦房租涨得还快。你从 GB 级起步，很快升到 TB 级，眼睁睁看着 Checkpoint 从几秒拖到几分钟，最后只能无奈地“只好直接重启作业算了”。传统流式 Join 会囤积一切（每条用户记录、每个订单、每次更新）因为“说不定哪天要用”。这本质上是在数据中心规模上演的数字囤积症。

数据触目惊心：对许多团队而言，一个中等规模的电商场景（订单关联用户）每处理十亿事件就会累积数十 GB 状态。再叠加商品目录、库存、物流……整个拓扑的状态轻松突破 TB。Checkpoint 从 30 秒拉长到 5 分钟、20 分钟，恢复时间也同步恶化。值班轮转变成俄罗斯轮盘：“下次 OOM 崩掉生产环境时，轮到谁了？”

当然，你可以反击：用 State TTL 清理旧数据、用 Interval Join 加时间窗口、广播小维表，甚至回退到批处理重跑。有些团队还会预分区以减少单算子状态，或尽可能改用 Temporal Join。但这些方案要么牺牲功能，要么增加运维复杂度。

正因如此，业界正竞相寻找根本解法：

* DeltaJoin：彻底外置状态
* RisingWave：状态基于 S3 存算分离
* Feldera：全链路预物化

不同哲学，同一共识：解决传统 Join 状态痛点。

**01**

DeltaJoin：既然能查，何必存？

Delta Join 彻底颠覆思路：既然能查，何必存？

它不在算子内部维护伙伴历史，而是在输出时实时查询外部索引存储。Flink 的 DeltaJoin 采用“最终一致性”模型 —— 每次 emit 时都探查最新版本（不锁定快照）。效果立竿见影：

* Checkpoint 开销极小
* 恢复速度飞快
* 作业真正实现弹性伸缩

流处理正在我们眼前被重新定义。

Flink&Fluss、RisingWave、Feldera 并非对手，而是对“状态管理”的三种哲学探索。关注它们，就是提前看清未来十年的技术走向。

本文聚焦 Flink 与 Fluss 的 Delta Join 解决方案，也会简要对比 RisingWave 和 Feldera 如何从不同角度破解同一难题。

---

**02**

Fluss：为这种模式而生的存储层

这种“按需查询”的革命，需要一个高度协同的存储层。Apache Fluss (incubating) 不是又一个流存储，而是专为此模式量身打造。

当其他系统让你在“流式更新”和“高效查询”之间二选一时，Fluss 凭借前缀查询（prefix lookup） 能力两者兼得。

### 前缀查询，才是关键

多数查询存储要求完整主键，否则无能为力。但 Fluss 允许你只用前缀发起查询。

例如，你的复合主键是 (customer\_id, order\_id, item\_id)，但只知道 `customer_id`？传统存储要么强制全表扫描，要么依赖二级索引（如果支持）。而 Fluss 可直接在单个 Tablet 上执行 RocksDB 范围扫描，精准命中。

Paimon 的主键查询在 Join Key 完全匹配主键时表现优异；但现实中，维表关联常只有部分 Key（如仅 `customer_id`）。Fluss 的前缀查询让这类复合键 Join 在真实数据开发过程中变得可行。

Paimon 要求完整主键匹配（或绕道聚合），Fluss 支持前缀查询，使复合键 Join 实用化。

底层架构上，每个 Fluss 表分桶对应一个 RocksDB KV Tablet + 一个 Changelog 日志 Tablet。查询请求直达 Tablet Leader，直接转化为 RocksDB 操作。KV 存储 + Changelog 双结构，同时支持“任意时间点查询”和“CDC 流输出”——这不是巧合，而是为 DeltaJoin 等模式精心设计。

传统 Join 在算子内缓存双侧状态，导致 RocksDB 膨胀；DeltaJoin 将历史外置到 Fluss，按需索引查询

---

## DeltaJoin 的两大核心技术

1. FLIP-486 [1]：引入核心算子 `StreamingDeltaJoinOperator`，配备双侧 LRU 缓存，并具备优化器智能——知道何时启用缓存。记录到达时先查缓存，未命中则触发异步查询。两个 `AsyncDeltaJoinRunner` 实例分别处理两侧查询，各自维护缓存以减少外部调用。

2. FLIP-519 [2]：解决更棘手的问题——异步乱序。`KeyedAsyncWaitOperator` 确保同一 Key 的更新严格串行处理，不同 Key 则并发执行。没有它，你的 Upsert 结果可能会错乱。这是“最终一致”与“最终灾难”的分水岭。

同一 Key 的更新（A1→A2→A3）即使异步任务重叠，输出仍保持顺序；不同 Key 并发执行

---

## 配置：理想与现实的交汇点

起步很简单：

```
SET 'table.optimizer.delta-join.strategy' = 'AUTO';SET 'table.exec.async-lookup.key-ordered-enabled' = 'true';SET 'table.exec.async-lookup.output-mode' = 'ALLOW_UNORDERED';-- 每侧缓存可单独调优
```

```

```

但优化器很挑剔：

* 源表到 Join 之间只允许无状态算子：支持下推的 Scan、保 Key 的计算、Watermark、Exchange。
* 一旦出现有状态算子，立即回退到传统 Join。
* 暂不支持级联 Join，优化器保持保守。

Changelog 流也面临严格审查：

* 含`UPDATE_BEFORE` 的流会被拒绝（已在 Flink 2.2 中支持）。
* 删除语义有严格规则：INNER Join 仅允许单侧流包含删除事件；LEFT/RIGHT Join 仅允许外侧流包含删除事件。
* 优化器宁可放弃优化，也要保证正确性。

**译注：在即将发布的 Flink 2.2 中，Delta Join 得到了诸多增强，可支持更多的场景。**

你的 Fluss 表设计决定性能：

```
CREATE TABLE fluss_customers (    customer_id BIGINT,    region_id INT,    order_count BIGINT,    customer_data STRING,    PRIMARY KEY (customer_id, region_id) NOT ENFORCED) WITH (    'bucket.num' = '16',    'bucket.key' = 'customer_id',  -- 按 customer_id 分桶，确保单 Tablet 查询路径    'table.log.ttl' = '7d'         -- 日志保留策略);
```

```
PK 表默认按主键哈希分桶（不含分区字段），除非显式设置 bucket.key。这一步至关重要：设 bucket.key = 'customer_id'，所有查询命中单个 Tablet；跳过它，查询将分散至整个集群。
```

---

##

**03**

架构之争：不同哲学，同一问题

流处理世界正分化为三大阵营：

### 🟢 Flink + Fluss（DeltaJoin）：极致轻量

* 外置一切：无内部状态、无 MVCC 开销、无增量索引维护
* 核心哲学：按需探查

### 🔴 RisingWave：流式数据库路线

* PostgreSQL 协议兼容，所有 Join 状态存于共享存储（Hummock，通常为 S3）
* Join 本质是增量物化视图，内置 MVCC，默认提供快照一致性
* 计算节点无状态，热数据靠多级缓存（Block/Meta/Disk）
* 优势：BI 工具开箱即用，报表永远平衡
* 代价：Checkpoint 需协调所有 Actor，延迟受最慢节点制约

### 🔵 Feldera：全链路增量计算

* 基于 Differential Dataflow（DBSP），不仅存状态，更存整个计算图
* 每个 Join 是 Z-set 变换，追踪带权重的增删操作
* 查询结果来自预计算、持续更新的索引
* 近期新增一级 Backfill 编排、标记连接器、分阶段历史+实时摄入
* 多跳 Join 或递归查询（如供应链追踪、图算法）性能碾压查询型方案——因每次迭代复用增量索引，而非重复查询或重算

Flink+Fluss（外置 KV+日志）、RisingWave（状态 S3 存算分离）、Feldera（增量索引+预物化）。三角定位清晰：

+ Flink+Fluss：按需查询，最小作业状态，最终一致性
+ RisingWave：S3 共享存储 + MVCC，快照一致
+ Feldera：全链路预物化，多跳/递归场景王者

##

**04**

性能现实检验

* Flink+Fluss (DeltaJoin)：TB 级维表存于 Fluss，作业状态仅缓存；未命中时增加个位数毫秒延迟，流量高峰可能引发背压。合理配置缓存 + 热点识别，可维持高命中率。

* RisingWave：Checkpoint 吞吐可达 100–200MB/s，但十亿行维表仍需 GB 级 Checkpoint；恢复需分钟级，但支持并行恢复（新节点可边预热缓存边处理）。

* Feldera：Checkpoint 更小，但存储开销大；单节点可处理百万级更新/秒，但每增加一个 Join，存储需求成倍增长。

Flink+Fluss、RisingWave、Feldera 与  的选型指南：适用场景与应避免的情况。

✅ 选 Flink + Fluss (Delta Join)：

* 高吞吐关联（用户画像、特征组装、实体解析）
* 维度表规模远小于事实流（如淘宝每秒百万事件关联用户配置）
* ⚠️ 避免：高频变更维表 → 缓存瞬间失效，每 Join 都变查询，网络打满。

✅ 选 RisingWave：

* 需要 SQL 兼容性 + 快照一致性
* 场景：金融看板、运营分析、业务人员直连 BI
* ⚠️ 避免：不愿承担“分布式数据库”级运维复杂度

✅ 选 Feldera：

* 复杂多跳/递归增量查询（如物流路径追踪：包裹→卡车→路线→天气→GPS）
* ⚠️ 避免：存储/I/O 预算紧张

**05**

一致性：无法回避的代价

官方文档明确：DeltaJoin 提供最终一致性，非快照一致性。

每次查询看到的是当时最新版本。若双侧并发更新，输出可能出现短暂不一致，最终收敛。

示例：T1 时刻订单使用客户地址 V1；T2 客户更新为 V2；T3 新订单使用 V2。输出流中同一客户的两笔订单地址不同——尽管下单仅隔数秒。

> RisingWave/Feldera 用户或许会嗤之以鼻，但他们的强一致性是以 Checkpoint 体积、恢复时间和运维成本为代价的。

下游必须适配此现实：

* 使用可幂等的 Sink（如 PostgreSQL）
* 业务逻辑容忍临时不一致
* 监控收敛时间，而非仅吞吐

对大多数实时分析场景，这是明智权衡：用完美一致性换运维 sanity。但若需金融级 ACID？请另寻他路。

最终一致性 vs. 快照一致性：DeltaJoin 在检查点收敛前会输出包含不同客户版本的订单，而 Feldera 等系统则在快照边界处对齐各版本。

##

**06**

未来已来

##

* Fluss 0.8 将深度集成 Flink 2.1/DeltaJoin，规划自动分桶推断、自适应缓存等优化。
* RisingWave 加码云原生，Elastic Disk Cache 显著降低 S3 I/O，并大力投入流式 Iceberg。
* Feldera 持续拓展增量计算边界，Backfill 编排让复杂状态管道启动更简单。

流处理世界不会收敛于单一方案，而是分化为专用工具解决专用问题。

DeltaJoin + Fluss 并非要取代 RisingWave 或 Feldera，而是为深陷状态膨胀困境的 Flink 用户提供一条务实逃生通道——无需更换引擎，即可让 Join 真正 scale。

约束真实存在：最终一致性、严格的拓扑限制、依赖协同存储。但对愿意接受这些权衡的团队，回报巨大：

* 真正可扩展的 Join
* 秒级 Checkpoint
* 工程师终于能睡整夜觉 💤

引用链接

[1] https://cwiki.apache.org/confluence/display/FLINK/FLIP-486%3A+Introduce+A+New+DeltaJoin

[2] https://cwiki.apache.org/confluence/display/FLINK/FLIP-519%3A++Introduce+async+lookup+key+ordered+mode

**关于 Apache Fluss**

---

## Apache Fluss (Incubating) 是面向实时分析设计的下一代流存储，正在持续迭代中，欢迎关注项目动态，体验试用。如果你喜欢这个项目，欢迎在 GitHub 上点赞支持 ❤️ ⭐。

GitHub：https://github.com/apache/fluss