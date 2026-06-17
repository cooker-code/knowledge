---
title: B 站基于 StarRocks 构建大数据元仓
author: StarRocks
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492334&idx=1&sn=dd0de6197bc6f3f67ffb0704b72ae02f&chksm=e9f29dcade8514dcfbfb19d06dc75659ff7c5d9d72ec10210eceb5220d9706b56b61d80dd9d8&mpshare=1&scene=24&srcid=1208T3zxYRsa7mm2oXgbvRJl&sharer_shareinfo=230f0ed23926e09612e96e5cb75fac5a&sharer_shareinfo_first=230f0ed23926e09612e96e5cb75fac5a#rd
---

作者：bilibili 大数据高级开发工程师 杨洋

**小编导读：**

B站大数据元仓是一款用来观测大数据引擎运行情况、推动大作业治理的系统诊断产品。经过调研和性能测试，大数据元仓最终以 StarRocks 为技术底座，从实际的应用效果来看，大部分查询都能在几百毫秒内返回结果。

B站大数据元仓是一款用来观测大数据引擎运行情况、推动大作业治理的系统诊断产品。经过调研和性能测试，大数据元仓最终以 StarRocks 为技术底座，从实际的应用效果来看，大部分查询都能在几百毫秒内返回结果。

随着B站业务的高速发展，数据量已达到 EB+ 级，为了适应数据服务需求，B站大数据平台引入了  Presto、Spark、ClickHouse 等多种大数据引擎。

在大数据引擎运行过程中，由于缺乏一些运行时的切面数据，我们难以实时观测引擎的运行情况，另外，由于缺乏作业维度的统计信息，我们也难以推动用户对大作业进行治理。为了满足这些需求，B站构建了大数据元仓系统。

大数据元仓涉及的大数据组件包括 Yarn、Presto、Spark 等，以 Presto 元仓为例，系统主要从集群、队列、query 等三个维度进行分析。

其中集群维度可以细分为包括 CPU、内存、扫描数据量等在内的节点资源汇总信息和包括各种 query 状态统计的集群 query 汇总信息；队列维度主要包括队列的资源、水位信息；query 维度则是对集群 query 汇总信息的一个补充，可以获取更详细的信息，比如可以具体了解导致查询失败的异常情况。

## **技术选型**

## **01** **需求特点**

目前，我们的内部监控架构基于 Prometheus 搭建，Prometheus 存储数据量有限，通常仅为一两个月的数据，不适合存储长时间的历史数据。此外，Prometheus 是基于度量的系统，更多地用于展示趋势性数据，例如集群的 CPU 和内存情况等，但对于像元仓这样需要下钻到具体明细数据的需求，Prometheus 则难以满足。

基于以上问题，我们打算设计一个新的架构来构建大数据元仓。我们的大数据元仓应该满足以下特点：

1. 实时观测：能够实时观测到集群的指标数据，并在多维分析场景中实现秒级或亚秒级的查询返回。
2. 复杂逻辑计算：支持复杂的逻辑计算，不需要将数据落库后打成大宽表的形式。有较高的灵活性，以便后期满足不同的需求，并在现有逻辑的基础上进行处理和分析。
3. 存储及回放：能够存储半年甚至更久的数据，并支持数据的回放。

## **02** **数据湖 or 数仓**

基于以上需求，我们对当前比较热门的数据湖、数仓组件进行了调研。其中，数据湖组件主要包括 Iceberg、Hudi 和 Delta Lake，数仓组件则重点调研了 ClickHouse 和 StarRocks。

最终，由于以下原因，我们选择了数仓技术作为大数据元仓的技术底座：

1. 传统的数据湖技术在实时性方面普遍存在不足，Hudi、Iceberg 虽然可以达到分钟级的实时性，但要实现秒级的实时性可能仍然存在一些困难；
2. 数据湖的远程 I/O 成本可能会较高，而数仓技术更多地采用本地 I/O，可以更有效地减少远程 I/O 的开销。
3. 在数仓技术中，有一些成熟的加速手段，例如通过物化视图和索引等方式来提高查询性能。相对于数据湖技术，数仓技术在这方面更加成熟。

## **03** **组件选型**

在数据湖与数仓之间作出选择后，关于采用 StarRocks 还是 ClickHouse，我们从6个维度进行了比较。

1. 标准 SQL：StarRocks 支持标准 SQL，并兼容 MySQL 协议，这对于应用程序迁移来说是一个优点。而 ClickHouse 在标准 SQL 方面并不完全支持；
2. 性能：StarRocks 的读写性能都较好，而 ClickHouse 在单机性能方面可能更强大；
3. StarRocks 可以很便利地通过多机多核的方式提高并发能力，而 ClickHouse 的并发能力相对较弱，默认的 QPS 大约为100；
4. JOIN 能力：StarRocks 的支持较好，可以建立星型或者雪花模型应对维度数据的变更，而 ClickHouse 的 JOIN 能力相对较弱，通常需要将数据处理成宽表进行查询；
5. 运维：StarRocks 不依赖第三方组件，如果出现资源不足的情况，可以很容易地对 FE 和 BE 进行横向扩展。而 ClickHouse 依赖于第三方组件，如 Zookeeper 来构建集群，运维成本更高；
6. StarRocks 社区在国内活跃度相对较高，在我们对 StarRocks 进行调研和测试时，如果遇到问题，社区往往能够快速给出建议和回复；

根据以上分析，我们更倾向于选择 StarRocks 作为大数据元仓技术的底座。

## **04** **性能测试**

为了进一步了解 StarRocks 在性能方面的表现，我们对 StarRocks 内外表与内部 Presto 集群的性能进行了比较，使用了 TPCH 数据集，并随机选择了一些 SQL 进行性能测试。

图中橙色线表示 StarRocks 外表的查询，灰色线表示 Presto 的查询。可以看出，相对于 Presto，StarRocks 具有更强大的查询性能，外表查询时间相缩短了大约70%至80%。如果采用内表查询，查询时间则会进一步缩短。

除了查询性能，我们还关注计算引擎的资源消耗，因此还比较了 StarRocks 和 Presto 的查询资源消耗。

这里特别说明一下，考虑到我们的元仓场景更倾向于使用内表进行查询，因此采用内表进行了资源、内存和 CPU 方面的比较。总体而言，相对于 Presto，StarRocks 的资源消耗更小。

## **05** **架构方案**

在元仓架构方面，我们最终确定 StarRocks 作为元仓的技术底座，提供存储和查询能力。此外，还构建了一个采集模块，主要功能是收集各个集群的指标，并将其推送到 Kafka。为了实现这一功能，我们在内部实现了一个代理（agent），该代理封装了从采集器（collector）将数据推送到 Kafka 的逻辑。

StarRocks 有两种方式从 Kafka 导入数据：Routine Load 和 Flink。其中，Routine Load 是 StarRocks 自带的一种导入作业方式，可以消费 Kafka 数据并将其写入 StarRocks。

采用 Routine Load 方式比较简单，用户只需要创建一个 Routine Load 作业，并指定列和 Kafka 主题以及一些分区信息即可进行数据消费和写入 StarRocks。在线上环境中，对于新业务来说，Routine Load 是比较容易推广的，因为我们可以与用户规范数据格式，使其以规范的格式写入 Kafka。

对于存量数据，用户可能已经在 Kafka 端采集了一些度量指标，此时让用户按照之前定义的规范重新将数据写入 Kafka 可能并不合适。对于一些特殊的业务逻辑，Routine Load 可能无法满足需求，这时就需要用到 Flink 来处理。

相比 Routine Load，Flink 通过编码的方式更加灵活，特别适用于处理复杂的多表关联查询。然而，由于 Flink 即使是对于简单的表也需要进行编码，这对于一些不常开发代码的用户来说可能会增加上手成本。因此，在内部我们会将 Routine Load 与 Flink 结合使用。

## **应用效果**

根据最终的应用情况，StarRocks 整体的性能表现非常好，在99分位延迟方面表现出色，大部分查询都能在几百毫秒内返回结果。

从元仓的角度来看，大数据元仓（以 Presto 元仓为例）带来的一个效果是对 CPU 使用情况的监控和分析。通过监控 Presto worker 的 CPU 指标，如可用处理器数量和 CPU 负载等，可以根据用户选择的时间范围（如3000分钟）和粒度（分钟、小时或天），对 CPU 使用情况进行分组和聚合，以获取整体 CPU 使用情况的统计数据。这样可以帮助用户了解 CPU 的利用率情况。

上图展示了B站内部 Presto 集群作业的概况。有时用户会反馈 Presto 作业运行较慢或失败较多。在遇到这些问题时，我们可以通过这张图进行量化分析，以确定是否存在排队查询或失败等情况。

图中，排队查询量、正在执行的作业成功量以及失败的作业数量等数据主要来源于 Presto Coordinator 的查询信息。通过这些信息，我们可以更加清楚地了解 Presto 作业的排队情况、执行成功率以及失败数量，以便更好地监控和管理 Presto 集群的性能和稳定性。

## **未来规划**

目前我们已经在内部完成了 StarRocks 的初步落地，将其应用于公司的元仓场景，并构建了一个大数据元仓系统，为用户提供实时的资源观测能力。此外，还通过诊断系统推动用户治理异常作业。

未来，会在如下一些方向开展工作：

1. 由于 StarRocks 在大数据元仓场景中表现非常出色，我们希望将其接入更多的业务场景，例如 BI 和 DQC 等。
2. 解决权限、UDF 等问题，比如接入 Hive UDF，使 StarRocks 与其它引擎对齐。
3. 目前的架构主要是以仓为中心，未来我们计划将半年或者更长时间的数据回流到数据湖中，从而实现湖仓一体化的架构。
4. 开启 StarRocks 的一些加速功能，例如物化视图索引，以提升现有元仓查询的速度。
5. 我们希望能够接入更多的组件，例如将 HDFS、Kyuubi 的大数据元信息纳入元仓体系中。
6. 诊断系统方面，目前主要以 Spark 诊断为主。未来，我们希望能够支持更多类型的作业诊断，如 Presto 和 Flink 作业的智能诊断。此外，我们还希望将诊断系统与公司内部其他平台打通，为用户提供更专业的诊断建议。

**关于 StarRocks**

Linux 基金会项目 StarRocks 是数据分析新范式的开创者、新标准的领导者。面世三年来，StarRocks 一直专注打造世界顶级的新一代极速全场景 MPP 数据库，帮助企业构建极速统一的湖仓分析新范式，是实现数字化转型和降本增效的关键基础设施。

StarRocks 持续突破既有框架，以技术创新全面驱动用户业务发展。当前全球超过 300 家市值 70 亿元以上的头部企业都在基于 StarRocks 构建新一代数据分析能力，包括腾讯、携程、平安银行、中原银行、中信建投、招商证券、大润发、百草味、顺丰、京东物流、TCL、OPPO 等，并与全球云计算领导者亚马逊云、阿里云、腾讯云等达成战略合作伙伴。

拥抱开源，StarRocks 全球开源社区飞速成长。目前，已有超过 300 位贡献者，社群用户近万人，吸引几十家国内外行业头部企业参与共建。项目在 GitHub 星数已超 6300 个，成为年度开源热力值增速第一的项目，市场渗透率跻身中国前十名。

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