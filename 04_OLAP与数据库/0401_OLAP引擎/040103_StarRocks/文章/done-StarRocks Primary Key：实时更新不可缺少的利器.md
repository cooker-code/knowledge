> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040103_StarRocks/040103_核心知识点/StarRocksPrimaryKey实时更新模型|StarRocksPrimaryKey实时更新模型]]
---
title: StarRocks Primary Key：实时更新不可缺少的利器
author: StarRocks
date:
url: http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492465&idx=1&sn=cb19f3e4e408290815962d0a8f54d53a&chksm=e9f29c55de851543656d5c0210f7170189c03a0c1dc1e7c9bb7c1762afc6d9bba14034efda7c&mpshare=1&scene=24&srcid=1226loer7OptpKuGCTiEUFPs&sharer_shareinfo=6b5d1c7856c59d67e2c42c73ec59f4b2&sharer_shareinfo_first=6b5d1c7856c59d67e2c42c73ec59f4b2#rd
---

迄今为止，分析型数据库已经成为数据驱动决策的重要组成部分。它们通常用于运行大规模、复杂的分析查询，如数据挖掘和机器学习任务。随着数据科学和人工智能的发展，分析型数据库的重要性在持续增加。

随着实时分析的理念推广，数据分析已经不满足于 T+1 这样的数据延迟。用最新鲜的数据实时分析出结果来指导决策已经成为主流。

**01**

# **分析型数据库实时数据变更的挑战**

由于分析型数据库的设计和优化主要是为了读取大量数据并执行复杂查询，而不是为了处理大量的写入和实时更新，在存储结构、数据分布等方面的特殊设计能够支撑较高的查询性能，却也给实时数据变更带来了不少挑战。

1. **存储结构**：很多分析型数据库采用列式存储，这种方式优化了读取性能，但在插入和更新数据时需要更多的开销。因为当一个新的记录插入或者一个旧的记录更新时，可能需要重写整个数据文件。
2. **索引和预计算**：分析型数据库通常使用复杂的索引结构和预聚合计算，如位图索引和查询改写。这些优化提高了查询性能，但也增加了数据更新的复杂性和成本。
3. **数据分布和****并行****处理**：在 MPP 架构中，数据被分布在多个节点上，查询可以在这些节点上并行执行。这种分布式处理提高了查询性能，但在数据更新时可能需要跨节点同步，这可能会引入延迟。

因此，尽管一些现代的分析型数据库开始提供一些实时或近实时的数据更新能力，与专为实时更新设计的 OLTP 数据库相比还是有很大的差距。

**02**

# **StarRocks 做了哪些改进**

StarRocks 自研的 Primary Key 模型在存储结果，索引等方面进行了改进。

1. **自适应更新模式**：维持列存的结构下，实现了列模式的更新方式。在整列更新的场景下只重写被更新的列，而不需要整行数据全部重写，极大的降低了数据更新时的 I/O 开销。
2. **主键****索引**：基于主键的 bitmap 索引结构支持在查询和更新时可以高效的定位到某行数据所在的位置，配合 delete vector 可以实现基于主键数据的高效检索和更新。
3. **主键****索引落盘**：在一定时间内没有被访问的主键索引会进行落盘操作，降低索引的内存占用。

StarRocks 通过在存储结构，索引方式方面的优化，实现了一个全新的可实时更新的 table format ，**Primary Key 模型可以满足 OLTP 数据库实时同步，Kafka 数据实时接入更新等大多数实时分析场景的需求****。**

**03**

# **Primary Key 更新原理**

Primary Key 模型可以在保证强大的实时更新能力的同时，具备极速的查询性能。那么 StarRocks 的是如何做到的呢？首先，在 AP 数据库系统中通常使用列存作为底层的存储引擎，一个表往往包含多个文件（或 Rowset），每个文件使用列存格式组织（例如 Parquet），并且是不可修改的。在这种组织结构的前提下支持更新，常见的实现方式包括：

1. **Copy on Write.** 当一批更新到来后，需要检查其中每条记录跟原来的文件有无冲突（或者说 Key 相同的记录）。对有冲突的文件，重新写一份新的、包含了更新后数据的。这种方式读取时直接读取最新数据文件即可，无需任何合并或者其他操作，查询性能是最优的，但是写入的代价很大，因此适合 T+1 的不会频繁更新的场景，不适合实时更新场景。
2. **Merge on Read.**当一批更新到来后，直接排序后以列存或者行存的形式写入新的文件。由于数据在写入时没有做去重或者说冲突检查，就需要在读取时通过 Key 的比较进行 Merge 的方式，合并多个版本的数据，仅保留最新版本的数据返回给查询执行层。这种方式写入的性能最好，实现也很简单，但是读取的性能很差。
3. **Delta Store.**当一批更新到来后，通过主键索引，先找到每条记录原来所在的文件和位置（通常是一个整数的行号）。把位置和所做的修改作为一条 Delta 记录，放到跟原文件对应的一个 Delta Store 中。查询时，需要把原始数据和 Delta Store 中的数据进行 Merge。这里牺牲了部分写入性能来换取读取性能。

**StarRocks 采用的是 Delete + Insert 的策略，**当一批更新到来后，通过主键索引，先找到每条记录原来所在的位置，把该条记录标记为删除，然后把最新数据作为新记录写入新文件。读取时，根据删除标记来将旧版本过期数据过滤掉，留下最新更新后的数据。该策略的好处是，因为无需像 Merge-on-Read 和 Delta Store 模式下进行 Merge，过滤算子可以下推到 Scan 层直接利用各类索引进行过滤减少扫描开销，所以查询性能的提升空间更大。

但是由于 Delete-and-Insert 模式需要引入主键索引以及存储和管理删除标记，在大规模实时更新和查询场景下要想高效率低实现还是非常有挑战的，我们采用了多种创新技术来实现这一目标：

* 首先，我们通过 roaring bitmap 来管理和存储每一个列存文件对应的删除标记。比如一个文件中有 10000 行数据，其中 200 行被标记删除，则可以使用一个 bitmap 来标记被删除的行。一般都是很稀疏的，使用 roaring bitmap 可以高效存储和操作。roaring bitmap 在存算一体架构下会被存储到 Rocksdb 里，而在存算分离架构下则会被记录到对象存储文件中。同时，roaring bitmap 也会被缓存在内存中以便能够快速访问。
* 接着，我们引入了主键索引来保存主键到该记录所在位置的映射，用于在数据导入和删除过程中生成列存文件对应的删除标记。我们提供了两种不同的主键索引实现：

+ **全内存索引。**索引数据在数据导入进行时按需构建，一段时间没有持续导入时又会释放以节约内存。比较适合对于更新延迟有更高要求的场景。
+ **持久化索引。**可以允许把部分索引数据持久化到磁盘以节约内存占用。针对当前主键索引的使用特点，比如批量操作，无并发，不需要范围查询等特点，我们设计了一套类似 LSM 的分层存储和 compaction 的机制的存储引擎，数据在文件中按 hash 分布编码。持久化索引做到了在节约内存的同时，查询和更新性能几乎接近全内存索引。

* **除了正常的全列更新和删除操作之外，主键模型还支持了部分列更新的能力。**针对不同的数据更新场景，我们提供了两种不同的部分列更新实现，在不影响查询性能的同时，尽可能地降低部分列更新的开销，从而能够保证更新的实时性。

+ **行模式。**行模式比较适用于小批量的实时更新场景。实现方式是，在数据导入阶段，先生成只包含部分列数据的列存文件，到了事务提交阶段，通过主键索引找到对应行缺失的列数据，并回填到刚才生成的列存文件中，生成完整的列存文件。如下所示，部分列更新想要将<100, John>，<200, Mike>和<300, Casey>更新为<100, Kim>，<200, Kitty>和<300, John>，因此需要从 Column3 中读取对应的缺失列数据，并且将之前的行标记删除。
+ **列模式。**列模式适用于大批量的批处理更新场景。实现方式是，通过主键索引找到被更新的记录所在的源列存文件，读取文件中的原始列数据，在和更新数据合并之后生成和源列存文件一一对应的部分列文件，在元数据中建立文件之间的映射。在执行查询时，由于无需做 Merge 操作，因此不会影响查询性能。如下图所示，我们重新生成了对应 Column2 更新之后的 DeltaColumn2 列文件，后续在构建 Iterator 时，会使用DeltaColumn2 替换 Column2。

在行模式下，由于要补齐缺失的列数据，因此读写放大和更新的列占比有关。假设需要更新的列数占比为 C% 则读写放大的倍数为：`1/C%`

而在列模式下，不需要补齐缺失的列数据，但是需要读取源列存文件中的原始列数据，在和更新数据合并后再写入部分列文件。因此其读写放大和其更新的行占比有关，假设需要更新的行数占比为 R% 则读写放大的倍数为：`1/R%`

**行模式和列模式的部分列更新可以在同一张表中混合使用，因此用户可以根据自己不同的更新场景，灵活选择不同的模式进行更新，**从而在不影响查询性能的前提下，获得最佳的更新效率。

**04**

# **Primary Key 更新方式**

## **01** **upsert**

在数据导入时通过对比主键实现数据的插入和更新，以 insert 作为例子在一次写入中，更新一条数据同时插入新的一行数据。其余导入方式类似。

## **02** **partial update**

在导入时对部分列进行更新操作，比如对 employee 表中需要对 ID 为 2、3、5、7 的员工修改其年龄将其加 1。

**方式一：**

需要更新的数据为一份 csv 文件，内容如下：

```
2,40
3,32
5,32
7,24
```

通过 http put 的方式将上述文件中的数据更新到 employee 表中对应的值。

**方式二：**

通过 SQL 的方式也可以达到上述效果：

```
update employee set age = age+1 where id in (2,3,5,7)
```

## **03** **conditional update**

在时序数据场景，经常会有数据乱序到达的情况。比如一张订单状态在不同的系统中被更新，在这种情况下通常只需要保留最新的一条记录即可。

Primary key 模型支持在导入时通过非 key 列的比较，来实现只保留最新的一行记录。

假设一张订单表如下：

新增的订单记录如下：

```
4，2023/09/15，AIR，4260.0，2023/09/18 00:21:04 
6，2023/09/16，FOB，19.6，2023/09/18 00:15:47
7，2023/09/17，TRUCK，624.0，2023/09/18 00:41:22
8，2023/09/18，REG AIR，22.0，2023/09/18 10:01:27
```

其中 orderid 为 4 和 6 的订单 updatetime 要大于原先 order 表中的记录，新记录会覆盖旧记录。orderid 为 7 的 updatetime 小于之前表内的记录值，说明该条数据是延迟到达状态，故 orderid 为 7 的记录不会被更新。同时 orderid 为 8 的记录之前不存在，会直接写入到表中。

## **04** **update**

Primary key 模型可以通过以下两种方式实现数据更新

1. **在 where 条件中使用 from 从句，实现单表或多表关联的数据更新**

假设有如下的订单表和运费表，自 2023/09/15 日零点起 SF 快递公司的运费上调 10% 。

则可以通过如下的 SQL 通过 orderid 对 orders 表和 ship\_fee 进行 join。关联后通过 shipdate 和 company 两个条件过滤需要更新的行，然后对 fee 字段进行上调操作。

```
UPDATE ship_fee
SET fee = fee * 1.1  --Increase by 10%
FROM orders
WHERE orders.shipdate >= '2023/09/15'
  AND ship_fee.company = 'SF'
  AND ship_fee.orderid = orders.orderid;
```

先通过 orders.shipdate 和 ship\_fee.company 两个条件分别筛选出两张表中符合条件的记录，然后通过 orderid 进行关联，保留关联后的结果，并更新对应的 fee 字段值。

2. **使用 CTE**

使用 CTE 可以改写上面的例子，方便理解

```
WITH increase_fee as (
    SELECT * from orders
    WHERE orders.shipdate >= '2023/09/15'
)
UPDATE ship_fee SET fee = fee * 1.1  --Increase by 10%
FROM increase_fee
WHERE ship_fee.orderid = orders.orderid
AND ship_fee.company = 'SF';
```

**关于 StarRocks**

Linux 基金会项目 StarRocks 是数据分析新范式的开创者、新标准的领导者。面世三年来，StarRocks 一直专注打造世界顶级的新一代极速全场景 MPP 数据库，帮助企业构建极速统一的湖仓分析新范式，是实现数字化转型和降本增效的关键基础设施。

StarRocks 持续突破既有框架，以技术创新全面驱动用户业务发展。当前全球超过 300 家市值 70 亿元以上的头部企业都在基于 StarRocks 构建新一代数据分析能力，包括腾讯、携程、平安银行、中原银行、中信建投、招商证券、大润发、百草味、顺丰、京东物流、TCL、OPPO 等，并与全球云计算领导者亚马逊云、阿里云、腾讯云等达成战略合作伙伴。

拥抱开源，StarRocks 全球开源社区飞速成长。目前，已有超过 300 位贡献者，社群用户近万人，吸引几十家国内外行业头部企业参与共建。项目在 GitHub 星数已超 6500 个，成为年度开源热力值增速第一的项目，市场渗透率跻身中国前十名。

**“极速统一” 数据分析新范式：**

[中信建投](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488978&idx=1&sn=68da6c95be3337048c5900ce70d56c5e&chksm=e9f16af6de86e3e0bc992f098a8d0787a18ff036bc70ea2d11f7d7e08358af16224f52d237a0&scene=21#wechat_redirect) [中原银行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487331&idx=1&sn=e6cdcdf2c46762135b6d95bf58e47664&chksm=e9f17047de86f9519e4d58f47745fe64788f2a5b3117133fc4b39db1a50c50d163a467f20950&scene=21#wechat_redirect)

[芒果TV](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490596&idx=1&sn=9de0a6ce3608875f17287a197b7847b1&chksm=e9f16300de86ea16b83b7f5d8a29b20c137b1bc641d4d190c4d73d717a56d8c104a1f585f4d7&scene=21#wechat_redirect)  [微信](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484928&idx=1&sn=87d2572bba55afef5abd5f7b8571a443&chksm=e9f17924de86f032c3dca0fdc69b9b8ab8b873bc9a5bc4fdb284c06c8539da0e24e6647ded93&scene=21#wechat_redirect) [网易邮箱](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488632&idx=1&sn=0f76b5ce855b3070a138cb58e5fb8112&chksm=e9f16b5cde86e24a853e743df776e25d2358e710442cb419d32c3dd504d8e328e06b5cb473fd&scene=21#wechat_redirect) [美团餐饮SaaS](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488130&idx=1&sn=53c32ae5448505db505872e0d8e8d7e1&chksm=e9f16da6de86e4b03a1295902ac27ff3fe9a554dd76383b6d24c155fb11778bc4d4afa7d9494&scene=21#wechat_redirect)

[理想汽车](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485820&idx=1&sn=7aa47ec35feec64e49ea462fe7506864&chksm=e9f17658de86ff4e1bd61f12d70ee8b478cc0ecd8fa7860df72ce2b5dc4b8ac05c77235809d6&scene=21#wechat_redirect)  [汽车之家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484870&idx=1&sn=74ab3350c6b7e86376c009b3ee6b5b98&chksm=e9f17ae2de86f3f45cc32c4ae153b61c57cdf6e66dd7b10ae9fb8ec689994598130257c43936&scene=21#wechat_redirect)  [滴滴](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490839&idx=1&sn=a007c5207f15129cd8c91da15cc30093&chksm=e9f16233de86eb25f585ef9b911e54ae6dae9547bd240a9554dd52ba449a74289a13d3df99fa&scene=21#wechat_redirect)  [蔚来汽车](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491009&idx=1&sn=d95f01aea67e68b617cb891be823dd9a&chksm=e9f162e5de86ebf310ec56cbe8c950a69ef5a311b8213bb3adaf843069f9f315a5bede006cdd&scene=21#wechat_redirect)

[腾讯游戏](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486696&idx=1&sn=d4bfaf2c2fb84a890922a0dfd4879522&chksm=e9f173ccde86fadaa1313f3e1c82d71f44b3e244113c712a272d3c85543765752431f8896ae6&scene=21#wechat_redirect)  [波克城市](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486150&idx=1&sn=bc272d1174edb74233c584c1b7f970da&chksm=e9f175e2de86fcf4b78f09ce88ead3dc591e11edd11a3efafe74441591928d37c213b9871e7d&scene=21#wechat_redirect)  [欢聚集团](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486170&idx=1&sn=bd5de60aa15f03a6972467c45c58bb9b&chksm=e9f175fede86fce8c2495df1d7f6f3096b70ec8ac69ee94e12f96f3e15c256a85a94c3e3c5f7&scene=21#wechat_redirect)  [37手游](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486749&idx=1&sn=610a64862a91df789faaa2469f2c8116&chksm=e9f17239de86fb2f78abd9f391d3c780f371213c93150d8617bb1c9a1845b837293f203f7d41&scene=21#wechat_redirect)

[顺丰](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484829&idx=1&sn=344fb330aac7d722a57a9591a82ca019&chksm=e9f17ab9de86f3af2e6d995b9ff8493c2a4d8e992d30e205904e3bd0ba88075061ea0158cb82&scene=21#wechat_redirect)  [京东物流](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486458&idx=1&sn=88058c480d5f163155bb885e3d1fbc2a&chksm=e9f174dede86fdc8defec71ca637bc2a5a9c87dd245c6bf9c51e11f2811fcdbcd507776ccd71&scene=21#wechat_redirect)   [58同城](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487038&idx=1&sn=583cf978f57802edd653034d83ac30af&chksm=e9f1711ade86f80cc37de844768228ba7e6a6abc06dd8b4867303334d31334e7b23273d12c39&scene=21#wechat_redirect) [同程旅行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490253&idx=1&sn=93f2a0369f9a9bff2215031efd197530&chksm=e9f165e9de86ecffe737cfebf729668df7efe09e6f9c4c976dc17668dc8042ef7895aef74ea3&scene=21#wechat_redirect)

[360](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485899&idx=1&sn=c8b13a6a6f9d0d59de1721a36f52c4cd&chksm=e9f176efde86fff974e81dfe2fb72a834528141a9b0f4a99f647b11c1a2e9001822b82ddbd1e&scene=21#wechat_redirect)  [得物](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487306&idx=1&sn=46b784eb29c1a4d645946a017deb7f8a&chksm=e9f1706ede86f97861ce9ed2952335312704fe2cc9adc35b3cfa5709f2abdeea9a2779ff18d6&scene=21#wechat_redirect)  [大润发](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488098&idx=1&sn=cae2b42fd8fdb77ed4e67fc83173291f&chksm=e9f16d46de86e450859f842b471de1ee4e57118901f6f1f7f2c545fefdbdfbe355c9dc3e7965&scene=21#wechat_redirect)  [华润万家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487047&idx=1&sn=74201f01931823c9cdc6abc71d3dba68&chksm=e9f17163de86f875439549071e7c79d3c2762cf245b69367be7024ed9d04380816da4348b38d&scene=21#wechat_redirect)  [TCL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=1&sn=cb78e47e420191b2345aff1366709384&chksm=e9f16e7dde86e76b77ecfd0d72d164313e18ef2fad887c3e41d30693bfd5e79629018be53a6a&scene=21#wechat_redirect)

[贝壳](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490822&idx=1&sn=a8bc0b31aa6568f57e919632e3f15877&chksm=e9f16222de86eb34d2a4eabd28c36ff2b29f3d9ad72855e8a9064d3b84b24a1e13970a80d3e3&scene=21#wechat_redirect) [万物新生](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490969&idx=1&sn=22a377677f403c37579b2309686d559b&chksm=e9f162bdde86ebab62ce4bd3905d63870947f5b42076fc4d3d1c60a0303e2565a01cb5e2b644&scene=21#wechat_redirect) [小红书](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484997&idx=1&sn=8644dacbcf467f6946add574b738e5d9&chksm=e9f17961de86f0776f5cbb201601010374c7e6c81b0a3250d93a32a997edbf694f4d471185eb&scene=21#wechat_redirect) [马蜂窝](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485975&idx=1&sn=dc80115074f959e9f389a9f1199d7894&chksm=e9f17533de86fc25650bd4c190f845eaab588acf3d97c4709f41cd2b23370cc3a9d7b9afdef3&scene=21#wechat_redirect)

**StarRocks 技术内幕：**

[极速湖仓神器：物化视图](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490902&idx=1&sn=0bd03778216286548f5d02a2df33466a&chksm=e9f16272de86eb645470a514afb35d6618d805d5358b7afa7b57f897c81ce2902e01d446064d&scene=21#wechat_redirect) 

[存算分离，兼顾降本与增效](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490146&idx=1&sn=771199c6e343612031134c0285fa4881&chksm=e9f16546de86ec50918f2c2ea3cc2eeac8d19655177fab2e9db933f5eb216fe30f2751c108b0&scene=21#wechat_redirect)

[实时更新与极速查询如何兼得](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485927&idx=1&sn=56051046f9eb51a563c6c24eb22c9364&chksm=e9f176c3de86ffd5bab12e9b8657ad58c2b4b4d2741dc0f7de292c800e4ec23cb064955013d0&scene=21#wechat_redirect)

[Query Cache，一招搞定高并发](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490921&idx=1&sn=9fc54c02d5395e112d0021686fb1bf6c&chksm=e9f1624dde86eb5be9a97895e1c92c40977b3943af6ad8b99611706092146e0690a3bf2e083d&scene=21#wechat_redirect)
[资源隔离](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247489066&idx=1&sn=4317418e82af2520ad5c3dd1c3ad5975&chksm=e9f1690ede86e018406d8f5b750a46cafcce21d263ae9794c23c85cc7b35d292053b523cde47&scene=21#wechat_redirect) [大数据自动管理](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485727&idx=1&sn=548ed6a15bd4b938cae8551500e3275d&chksm=e9f1763bde86ff2dcea5aacf2e1d8b9401cee6236b41571e241de74178cd856842e4e62fb48a&scene=21#wechat_redirect) [查询原理浅析](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485848&idx=1&sn=90e47d7a46eb120701d28b5acfbcc401&chksm=e9f176bcde86ffaaa0c4a3b686eea5b053ff286164cb9a27e85daf4e42bec08985f5a41975c3&scene=21#wechat_redirect)