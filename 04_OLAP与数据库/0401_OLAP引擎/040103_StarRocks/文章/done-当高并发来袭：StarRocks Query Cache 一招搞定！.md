> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040103_StarRocks/040103_核心知识点/StarRocksQueryCache中间聚合缓存|StarRocksQueryCache中间聚合缓存]]
---
title: 当高并发来袭：StarRocks Query Cache 一招搞定！
author: StarRocks
date:
url: http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490921&idx=1&sn=9fc54c02d5395e112d0021686fb1bf6c&chksm=e9f1624dde86eb5be9a97895e1c92c40977b3943af6ad8b99611706092146e0690a3bf2e083d&mpshare=1&scene=24&srcid=1227pcyIVPnxv7BSk3CMvYzg&sharer_shareinfo=23b17946a14a8adfdf74dc7fcbd56abb&sharer_shareinfo_first=23b17946a14a8adfdf74dc7fcbd56abb#rd
---

您是否曾经遇到这样的情况？每天早上或业务活动高峰期，大量用户涌入报表平台或数据应用，希望查看特定业务领域的最新指标或趋势。这些用户可能会基于庞大的数据集进行大量类似的聚合查询，造成集群的 CPU 负载持续攀升，从而导致查询性能不断下滑。针对这种高并发且呈现一定规律的查询，是否存在一种方法可以让集群在处理时智能地“精简计算量”呢？

**#01****StarRocks Query Cache（查询缓存）**

## 为了解决这一问题， StarRocks 研发了 Query Cache（查询缓存）。**它的作用是将本地聚合的中间结果缓存在内存中，以供后续复用。**当执行查询时，StarRocks 会优先检查 Query Cache。如果发现相同查询语义的结果已经存在于缓存中，就可以直接复用这些中间结果，避免重复计算，从而节省了磁盘访问和部分计算开销，有效提升查询性能。

值得注意的是，**Query** **Cache 并不是 Result Cache****，它缓存的是查询过程中的聚合中间结果而不是最终结果**，因此大大提升了缓存的命中率。即便对于不完全一致的查询，也能起到加速作用。据测试结果显示，**在高并发场景下，Query Cache 可以将查询效率提高****3****至****17****倍****，**从而有效减轻集群的负载压力，提供更快速的查询响应时间，使得整个系统在高峰期依然能够保持高性能运行。

（Query Cache 机制）

**#02****面向更多场景设计，最大限度提升缓存复用率**

StarRocks 的 Query Cache 在设计时就考虑了如何能够让缓存的信息最大程度得到复用。整体来讲，下列三个场景均可以利用到 Query Cache：

* 语义等价的查询
* 扫描分区重合的查询：基于谓词的查询拆分
* 仅涉及追加写入（无删除及更新）数据的查询：多版本缓存能力

###

## **01** **语义等价的查询**

类似上图的例子，第一张图的子查询与第二张图在语义上是等价的，因此在执行了其中一个后，另一个查询就可以复用缓存中的结果加速查询。

> 语义等价还包含非常多的场景，更多例子请见： https://docs.starrocks.io/zh-cn/latest/using\_starrocks/query\_cache#语义等价的查询

## **02** **扫描分区重合的查询：基于谓词的查询拆分**

在上图的两个查询中，ts 是分区列，查询仅在分区列的筛选区间上有区别，并且其中一部分区间是重叠的。在执行任意查询时，StarRocks 会将谓词中的区间按照分区来切割，并按照分区级别缓存聚合中间结果。在下次执行时，就可以复用有重叠的分区结果，达到查询加速的效果。

> 更多例子请见：https://docs.starrocks.io/zh-cn/latest/using\_starrocks/query\_cache#扫描分区重合的查询

## **03** **仅涉及追加写入数据的查询：多版本缓存能力**

除了上述在不同查询中尽可能复用 Cache，还有一类场景需要考虑：如果数据变化了该如何应对？**Query** **Cache 可以在只有追加写入（append）的场景下被复用****。**

总体来说，随着数据导入，Tablet 会产生新的版本，进而导致 Query Cache 中缓存结果的 Tablet 版本落后于实际的 Tablet 版本。这时候，多版本 Cache 机制会尝试把 Query Cache 中缓存的结果与磁盘上存储的增量数据合并，确保新查询能够获取到最新版本的 Tablet 数据。

> 更多例子请见：https://docs.starrocks.io/zh-cn/latest/using\_starrocks/query\_cache#仅涉及追加写入数据的查询

**#03****在这些场景上，Query Cache 能事半功倍**

根据上面的讲解，可以看出相比基于结果的 Result Cache，基于聚合中间结果的 Query Cache 能够被更大程度地利用。**因而 Query Cache 就更适用于以下的查询场景：**

* 聚合类查询执行比较频繁，包含针对宽表的聚合查询与星型模型 JOIN 后的聚合查询
* 两个语言相似的查询，但允许不完全相同
* 数据只会追加写入，没有更新操作

这样的查询特征在很多场景中都很常见，例如：

* **监控或报表平****台****：**数据集会随着时间的推移逐渐增加，而且用户会对不同时间段的数据汇总结果感兴趣
* **面向用户的高并发分****析****：**包含按照特定维度指标进行汇总的查询

**在这些场景中，****Query** **Cache 可以通过重复使用查询的中间结果，避免重复计算，加快查询的响应速度。同时，由于缓存机制的使用，系统的可扩展性也得到了提升，从而可以更好地处理高并发的查询请求，为用户提供更好的体****验****。**

**#04****最佳实践**

**为了更高效地利用** **Query** **Cache，建表时需要根据查询设置合理的分区策略，并选择合适的数据分布方式**，包括：

* **选择一个单独的 date/datetime 类型的列为分区列****。**这个列的数据最好随着导入单调增加，并且查询会基于该列进行区间筛选。
* **选择合适的分区大小****。**因为随着导入，最近的分区数据很有可能会经常变动，从而导致缓存失效。因此过大或过小的分区都会影响缓存的命中率。
* **确保分桶数量在数十个左****右。**如果分桶数量过小，那么当 BE 需要处理的 Tablet 数量小于 `pipeline_dop` 参数的取值时，Query Cache 无法生效。

并且，因为每个 BE 节点在内存中维护自己的本地缓存，只要 BE 节点上有查询所需的副本数据，查询就可以被分配给该 BE。所以，为了能够最大程度地应用到 Cache，查询应该至少执行与副本数（replication\_num）相同的次数。不过，Query Cache 也并不是只有在完全加载后才起作用。

**#05****如何使用 Query Cache**

在这一节我们将用通过一个简单的例子来展示 StarRocks Query Cache 是如何工作的。

## **01** **准备工作**

**Query** **Cache 默认情况下是禁用的****。您可以通过会话变量来启用它****。**在例子中我们在单个 BE 集群中创建一副本的表，为了使 Query Cache 能够发挥作用，也需要对 pipeline\_dop 进行调整。为了确认 Query Cache 的使用情况，还需要打开 Profile。

```
--打开 Query Cacheset enable_query_cache=true;--因为只有1副本，因此调整 pipeline dop 为1set pipeline_dop=1;--打开 Query Profileset is_report_success = true;set enable_profile = true;
```

> 建表和导入语句请见：https://docs.starrocks.io/zh-cn/3.1/using\_starrocks/query\_cache#数据集

###

## **02** **示例查询**

### 接下来我们将用一个简单的例子来激活 Query Cache 并查看它的使用情况。

首先我们执行第一个基础查询：

```
-- Q1: 基础查询SELECT    date_trunc('hour', ts) AS hour,    k0,    sum(v1) AS __c_0FROM    t0WHERE    ts between '2022-01-03 00:00:00'    and '2022-01-03 23:59:59'GROUP BY    date_trunc('hour', ts),    k0;
```

执行后我们查看 BE 的缓存情况。其 `usage`相关指标被填充，说明 Query Cache 已经被填充：

```
curl http://127.0.0.1:8040/api/query_cache/stat{    "capacity": 536870912,    "usage": 3889,    "usage_ratio": 0.000007243826985359192,    "lookup_count": 42,    "hit_count": 0,    "hit_ratio": 0.0
```

接下来执行语义等价的查询：

```
-- Q2: 语义等价的查询SELECT    (        ifnull(sum(murmur_hash3_32(hour)), 0) + ifnull(sum(murmur_hash3_32(k0)), 0) + ifnull(sum(murmur_hash3_32(__c_0)), 0)    ) AS fingerprintFROM    (        SELECT            date_trunc('hour', ts) AS hour,            k0,            sum(v1) AS __c_0        FROM            t0        WHERE            ts between '2022-01-03 00:00:00'            and '2022-01-03 23:59:59'        GROUP BY            date_trunc('hour', ts),            k0    ) AS t;
```

执行后我们继续查看 BE 的缓存情况。这里可以看到，Query Cache 被命中：

```
curl http://127.0.0.1:8040/api/query_cache/stat{    "capacity": 536870912,    "usage": 3889,    "usage_ratio": 0.000007243826985359192,    "lookup_count": 44,    "hit_count": 2,    "hit_ratio": 0.045454545454545459
```

进一步分析此次查询的 Profile。可以发现 Profile 中出现 Cache 节点，Populate 相关指标均为 0，说明没有新的聚合结果被缓存；并且 Scan 节点的 RawRowsRead 指标为 0，说明实际并没有读取数据：

```
Cache节点populate相关指标均为0CachePopulateBytes: 0.00 CachePopulateChunkNum: 0CachePopulateRowNum: 0CachePopulateTabletNum: 0
Scan节点RawRowsRead为0RawRowsRead: 0M (0)
```

**#06****性能报告**

尽管 StarRocks 的 Query Cache 不是 Result Cache，但是重复利用中间计算结果仍然可以带来很大的性能提升。这不仅适用于聚合查询，还能加速 JOIN 操作。现在让我们来看一些性能数据。为简单起见，我们将结果表示为 RT 比率，即查询延迟的 no\_cache/cache\_hit 比率。

> 注意，下方所有测试中 Query Cache 均已经被充分加载。

###

## **01** **宽表测试**

|  |  |
| --- | --- |
| 硬件 | 3 x (CPU: 16cores, 内存: 64GB) |
| 数据集 | SSB 100GB 宽表 |
| 并发量 | 10 |

（Query Cache vs. 冷查询）

可以看到，**在****10****并发的单表聚合查询中，****Query** **Cache 命中可以带来高达 10 倍的性能提升****。**

## **02** **星型模型测试**

|  |  |
| --- | --- |
| 硬件 | 3 x (CPU: 16cores, 内存: 64GB) |
| 数据集 | SSB 100GB 多表 |
| 并发量 | 10 |

可以看到，**在 10 并发的多表聚合查询中，****Query** **Cache 命中可以带来高达****17****倍的性能提****升****。**

**#07****总结**

Query Cache 可以极大地提升聚合查询的性能。通过将本地聚合的中间结果存储在内存中，Query Cache 可以避免对类似于先前查询的新查询进行不必要的磁盘访问和计算。Query Cache 还可以处理不完全相同的查询和数据，这使得它比 Result Cache 更加灵活。在高并发场景中，许多用户在大型复杂数据集上运行类似的查询时，Query Cache 尤其有用。借助 Query Cache，StarRocks 可以为聚合查询提供快速而准确的结果，节省时间和资源，并实现更好的可扩展性。

**💬** **Query** **Cache 讨论专区**

为了帮助用户更好的使用 query cache，社区在论坛上开了一个相关的讨论帖，欢迎来分享你们都是怎么使用 query cache，或是使用过程中遇到了什么问题。性能提升的招式，大家都学起来！👉🏻 https://forum.mirrorship.cn/t/topic/8468

**参考资料**

https://docs.starrocks.io/zh-cn/latest/using\_starrocks/query\_cache

**关于 StarRocks**

Linux 基金会项目 StarRocks 是数据分析新范式的开创者、新标准的领导者。面世三年来，StarRocks 一直专注打造世界顶级的新一代极速全场景 MPP 数据库，帮助企业构建极速统一的湖仓分析新范式，是实现数字化转型和降本增效的关键基础设施。

StarRocks 持续突破既有框架，以技术创新全面驱动用户业务发展。当前全球超过 260 家市值 70 亿元以上的头部企业都在基于 StarRocks 构建新一代数据分析能力，包括腾讯、携程、平安银行、中原银行、中信建投、招商证券、众安保险、大润发、百草味、顺丰、京东物流、TCL、OPPO 等，并与全球云计算领导者亚马逊云、阿里云、腾讯云等达成战略合作伙伴。

拥抱开源，StarRocks 全球开源社区飞速成长。截至 2022 年底，已有超过 200 位贡献者，社群用户近万人，吸引几十家国内外行业头部企业参与共建。项目在 GitHub 星数已超 5200 个，成为年度开源热力值增速第一的项目，市场渗透率跻身中国前十名。

**“极速统一” 数据分析新范式：**

[中信建投](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488978&idx=1&sn=68da6c95be3337048c5900ce70d56c5e&chksm=e9f16af6de86e3e0bc992f098a8d0787a18ff036bc70ea2d11f7d7e08358af16224f52d237a0&scene=21#wechat_redirect) [众安保险](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485788&idx=1&sn=fd7f8f86d1a7ee02063a8ca7cba295cf&chksm=e9f17678de86ff6ed2af617780d227bf2b04df8f99c102df0acb507ae7373ec6ec6e8a31c200&scene=21#wechat_redirect) [中原银行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487331&idx=1&sn=e6cdcdf2c46762135b6d95bf58e47664&chksm=e9f17047de86f9519e4d58f47745fe64788f2a5b3117133fc4b39db1a50c50d163a467f20950&scene=21#wechat_redirect) [信也科技](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484926&idx=1&sn=09ceb7940091b9356609376747a36ac7&chksm=e9f17adade86f3cc3afa02f5b0ecc9bbbbc4072b56c27296eff80f6af815c8fd6c1af442a5be&scene=21#wechat_redirect)

[微信](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484928&idx=1&sn=87d2572bba55afef5abd5f7b8571a443&chksm=e9f17924de86f032c3dca0fdc69b9b8ab8b873bc9a5bc4fdb284c06c8539da0e24e6647ded93&scene=21#wechat_redirect) [网易邮箱](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488632&idx=1&sn=0f76b5ce855b3070a138cb58e5fb8112&chksm=e9f16b5cde86e24a853e743df776e25d2358e710442cb419d32c3dd504d8e328e06b5cb473fd&scene=21#wechat_redirect) [美团餐饮SaaS](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488130&idx=1&sn=53c32ae5448505db505872e0d8e8d7e1&chksm=e9f16da6de86e4b03a1295902ac27ff3fe9a554dd76383b6d24c155fb11778bc4d4afa7d9494&scene=21#wechat_redirect) [易点天下](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247489201&idx=1&sn=a8650b65040bee2a1faaba2af7a58244&chksm=e9f16995de86e083223aab0daa1555944aefc1af9058bed463f646eee8a5c5912c33084af34f&scene=21#wechat_redirect)

[理想汽车](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485820&idx=1&sn=7aa47ec35feec64e49ea462fe7506864&chksm=e9f17658de86ff4e1bd61f12d70ee8b478cc0ecd8fa7860df72ce2b5dc4b8ac05c77235809d6&scene=21#wechat_redirect)  [汽车之家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484870&idx=1&sn=74ab3350c6b7e86376c009b3ee6b5b98&chksm=e9f17ae2de86f3f45cc32c4ae153b61c57cdf6e66dd7b10ae9fb8ec689994598130257c43936&scene=21#wechat_redirect)   [滴滴](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484326&idx=1&sn=3e038503ce8a81a74728b50f676a35ec&chksm=e9f17c82de86f5949d4f5c0622efe4b69d65edf6b7faeddbe30d410dcd2e84a6c95249ad4ff7&scene=21#wechat_redirect)     [首约汽车](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488208&idx=1&sn=12eae8447ac459bcb3fffdf12fad379e&chksm=e9f16df4de86e4e207748dd12102849f29addf878bf382d4c25f01e47934437ab49aba0add4b&scene=21#wechat_redirect)

[腾讯游戏](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486696&idx=1&sn=d4bfaf2c2fb84a890922a0dfd4879522&chksm=e9f173ccde86fadaa1313f3e1c82d71f44b3e244113c712a272d3c85543765752431f8896ae6&scene=21#wechat_redirect)  [波克城市](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486150&idx=1&sn=bc272d1174edb74233c584c1b7f970da&chksm=e9f175e2de86fcf4b78f09ce88ead3dc591e11edd11a3efafe74441591928d37c213b9871e7d&scene=21#wechat_redirect)  [欢聚集团](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486170&idx=1&sn=bd5de60aa15f03a6972467c45c58bb9b&chksm=e9f175fede86fce8c2495df1d7f6f3096b70ec8ac69ee94e12f96f3e15c256a85a94c3e3c5f7&scene=21#wechat_redirect)  [37手游](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486749&idx=1&sn=610a64862a91df789faaa2469f2c8116&chksm=e9f17239de86fb2f78abd9f391d3c780f371213c93150d8617bb1c9a1845b837293f203f7d41&scene=21#wechat_redirect)  [游族](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487470&idx=1&sn=e8844ab1037069989e3dc95f5e9092e8&chksm=e9f170cade86f9dcd62d810c6f7132002c4cf7329dbf20f232db82ca6cca955ffd934be04e0c&scene=21#wechat_redirect)

[顺丰](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484829&idx=1&sn=344fb330aac7d722a57a9591a82ca019&chksm=e9f17ab9de86f3af2e6d995b9ff8493c2a4d8e992d30e205904e3bd0ba88075061ea0158cb82&scene=21#wechat_redirect)  [京东物流](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486458&idx=1&sn=88058c480d5f163155bb885e3d1fbc2a&chksm=e9f174dede86fdc8defec71ca637bc2a5a9c87dd245c6bf9c51e11f2811fcdbcd507776ccd71&scene=21#wechat_redirect)   [跨越速运](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=2&sn=7912601d564276718811d53c60116a69&chksm=e9f16e7dde86e76ba4a28a99faf69c33a0553e4d383d4e083c3e0387e1e6d088fd13094c055b&scene=21#wechat_redirect)  [京东到家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485053&idx=1&sn=b9dfda8d35c27621c8012f3df54d9728&chksm=e9f17959de86f04f15f2d86a7df86e66192ee2cc38bdcafbc790c2a5149f96629fb79f511c76&scene=21#wechat_redirect)   [58同城](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487038&idx=1&sn=583cf978f57802edd653034d83ac30af&chksm=e9f1711ade86f80cc37de844768228ba7e6a6abc06dd8b4867303334d31334e7b23273d12c39&scene=21#wechat_redirect)

[小米](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484613&idx=1&sn=7704c22b16a36851b2392f14ef7cda70&chksm=e9f17be1de86f2f765a2352b67a2ff49ea2c89d0940a3b27529cb18b78e5a51bcf8308a182dd&scene=21#wechat_redirect)   [搜狐](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485710&idx=1&sn=7438bc119f8e0cac3f7535a7c34539d8&chksm=e9f1762ade86ff3cbd4f4bf45b4d422cee972f4cf646e6822591e9213fb064a3e3ed8a3912bb&scene=21#wechat_redirect)   [小红书](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484997&idx=1&sn=8644dacbcf467f6946add574b738e5d9&chksm=e9f17961de86f0776f5cbb201601010374c7e6c81b0a3250d93a32a997edbf694f4d471185eb&scene=21#wechat_redirect)   [华米](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485647&idx=1&sn=b9eddd4b269eb1545c144e8ad8ccfc27&chksm=e9f177ebde86fefdfb2a9d25729d5b7c4a3bd7ffe4da1d51297c4e5bb4a6a8bae93a6395880f&scene=21#wechat_redirect) [360](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485899&idx=1&sn=c8b13a6a6f9d0d59de1721a36f52c4cd&chksm=e9f176efde86fff974e81dfe2fb72a834528141a9b0f4a99f647b11c1a2e9001822b82ddbd1e&scene=21#wechat_redirect)  [得物](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487306&idx=1&sn=46b784eb29c1a4d645946a017deb7f8a&chksm=e9f1706ede86f97861ce9ed2952335312704fe2cc9adc35b3cfa5709f2abdeea9a2779ff18d6&scene=21#wechat_redirect)  [大润发](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488098&idx=1&sn=cae2b42fd8fdb77ed4e67fc83173291f&chksm=e9f16d46de86e450859f842b471de1ee4e57118901f6f1f7f2c545fefdbdfbe355c9dc3e7965&scene=21#wechat_redirect)

[酷家乐](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486188&idx=1&sn=f495dcff3c1564d56afd206f1c387d18&chksm=e9f175c8de86fcde90a2c1e0e3d4a35150a7b0f68aedfa9b3715b2adc6b9069786d1b6253c99&scene=21#wechat_redirect)   [DMALL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484971&idx=1&sn=8f7e7ec424335b81fb34f1da85efbf8f&chksm=e9f1790fde86f019016e67a8a820f866bcbffe484414fa93b403605f957874bb7450672bfc3d&scene=21#wechat_redirect)   [华润万家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487047&idx=1&sn=74201f01931823c9cdc6abc71d3dba68&chksm=e9f17163de86f875439549071e7c79d3c2762cf245b69367be7024ed9d04380816da4348b38d&scene=21#wechat_redirect)   [百草味](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487394&idx=2&sn=242d444b7c02721b503de9383329edcd&chksm=e9f17086de86f990d2e18132c2b9beb3fa3984f5accb03cb34e0d479e8575666426c5508913c&scene=21#wechat_redirect)  [中纺](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487832&idx=1&sn=54c8c68f8e0c9bb2c57fe00cb363750c&chksm=e9f16e7cde86e76a41134c7abe5855e4034bf4897bf5c26a9de05d6b6dacad758a4672c828d1&scene=21#wechat_redirect)

[马蜂窝](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485975&idx=1&sn=dc80115074f959e9f389a9f1199d7894&chksm=e9f17533de86fc25650bd4c190f845eaab588acf3d97c4709f41cd2b23370cc3a9d7b9afdef3&scene=21#wechat_redirect)   [松果出行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486673&idx=1&sn=684ecac7270c81c2daade5bbe826ab80&chksm=e9f173f5de86fae3427c28f4de0e1fb8698aed9eb7f42fc59ece4cae5549faeb4645ffead2ac&scene=21#wechat_redirect)   [酷开](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486836&idx=1&sn=399086d79a513d311083c30ec7746589&chksm=e9f17250de86fb4630b4c3f6d3463c06daa9941d42b8c8cf311e91c0817f403de2d138d3884e&scene=21#wechat_redirect)   [TCL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=1&sn=cb78e47e420191b2345aff1366709384&chksm=e9f16e7dde86e76b77ecfd0d72d164313e18ef2fad887c3e41d30693bfd5e79629018be53a6a&scene=21#wechat_redirect) [零洞科技](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247489208&idx=1&sn=aee3d681595f1036955298b9ee02b0e5&chksm=e9f1699cde86e08a6512b7b75dc7afbbe9f5370d5469b2a5f36cbe2dacaabf8061a7035a648e&scene=21#wechat_redirect)

**StarRocks 技术内幕：**

[资源隔离](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247489066&idx=1&sn=4317418e82af2520ad5c3dd1c3ad5975&chksm=e9f1690ede86e018406d8f5b750a46cafcce21d263ae9794c23c85cc7b35d292053b523cde47&scene=21#wechat_redirect) [大数据自动管理](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485727&idx=1&sn=548ed6a15bd4b938cae8551500e3275d&chksm=e9f1763bde86ff2dcea5aacf2e1d8b9401cee6236b41571e241de74178cd856842e4e62fb48a&scene=21#wechat_redirect) [查询原理浅析](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485848&idx=1&sn=90e47d7a46eb120701d28b5acfbcc401&chksm=e9f176bcde86ffaaa0c4a3b686eea5b053ff286164cb9a27e85daf4e42bec08985f5a41975c3&scene=21#wechat_redirect)

[实时更新与极速查询如何兼得](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485927&idx=1&sn=56051046f9eb51a563c6c24eb22c9364&chksm=e9f176c3de86ffd5bab12e9b8657ad58c2b4b4d2741dc0f7de292c800e4ec23cb064955013d0&scene=21#wechat_redirect)

[基于全局字典的极速字符串查询](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486910&idx=1&sn=69ff907f3913c5c469bace781e6e0ad2&chksm=e9f1729ade86fb8ceb7a67f9cb68b771ac67fabbbddb6855fcc5f5deac5101f77279a3edb999&scene=21#wechat_redirect)

[向量化编程精髓](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487094&idx=1&sn=78582f75527db7e95f7fc26ad3462c43&chksm=e9f17152de86f844e4eff1042dec6444f4e02719bc252b4cc9e25834eabed346c3d5e5802fcf&scene=21#wechat_redirect) [Pipeline 执行框架](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487394&idx=1&sn=1f09310bda58777f970958731bd368ec&chksm=e9f17086de86f9904bdd6054b3f91196431df7c463d0bdc0812d48b74404d75510b7a8c3af71&scene=21#wechat_redirect)

[Join 查询优化](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487563&idx=1&sn=8d252c17ec64af465174a7d69ba20200&chksm=e9f16f6fde86e6792dca6b8212b99468adced26cfc442bd004df97019386ba1dad01cd02f3cf&scene=21#wechat_redirect)   [多表物化视图的设计与实现](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487666&idx=1&sn=6d079a5bea8c70e9ee45ce99c1591c78&chksm=e9f16f96de86e6809f3444b0ae15a10d350b0273f8fb88bac4d921c3cd8ffec722c1892b89ad&scene=21#wechat_redirect)

[兼顾降本与增效，我们对存算分离的设计与思考](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490146&idx=1&sn=771199c6e343612031134c0285fa4881&chksm=e9f16546de86ec50918f2c2ea3cc2eeac8d19655177fab2e9db933f5eb216fe30f2751c108b0&scene=21#wechat_redirect)

****👇****阅读原文****了解   产品详细信息****