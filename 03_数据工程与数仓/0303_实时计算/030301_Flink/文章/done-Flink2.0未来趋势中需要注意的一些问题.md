> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink2.0关键变化与选型影响|Flink2.0关键变化与选型影响]]
---
title: Flink2.0未来趋势中需要注意的一些问题
author: 大数据技术与架构
date:
url: http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247523717&idx=1&sn=ba0cae9c803268a7784f9aa873626104&chksm=fc98411466d53c1bf95026bd4ac3a2f7787e02dafe8fc34cdb718007ebd073eeeb87ee8e4912&mpshare=1&scene=24&srcid=1218Yv6v4PAGPy8llzIl4X9E&sharer_shareinfo=1faee84fb8020d416388b184549c9474&sharer_shareinfo_first=1faee84fb8020d416388b184549c9474#rd
---

手机打字，篇幅不长，主要讲一下FFA中关于Flink2.0的未来趋势，直接看重点。

Flink Forward Asia 2024主会场有一场关于Flink2.0的演讲，很精彩，官方也发布了一些关于Flink2.0的展望和要解决的问题。

1.0时代和2.0时代避免不了一些兼容性改动，例如配置文件、状态兼容以及一些常见的API，当然这些问题都不是用户需要考虑的，平台要做好升级。

那么作为普通的开发者应该注意到的未来趋势有哪些？

### 存算分离

存算分离是所有数据领域组件都在解决的一个问题，比如Apache Doris、Apache Pulsar等等，Flink同样面临这样的问题，因为在2.0中一个显著的课题就是**「存算分离云原生化架构升级」**。

Flink官方给出了四个要解决的诉求：

**计算和存储解绑**、**容器化资源的均匀使用**、**利用海量低价云存储**、**带状态的快速扩缩容**。

Flink 2.0 中的存算分离归根结底是存储的问题，因此引入了新开发的ForSt DB来解决这个问题。

如果存算分离能够很好的实现，未来Flink任务的迁移和升级将会十分方便和快捷，尤其是带大状态的任务，目前这个痛点相信困扰了很多很多人。

### 批流一体的解决方案

Flink2.0引入了全新的流批一体 Materialized Table(物化表)的概念来解决Streaming任务和Batch任务在代码层面的不一致性。

除了帮助用户实现只写一份代码、提高开发运维效率之外，Materialized Table 还提供了更多的成本优化空间。Materialized Table 支持流式持续刷新、批式全量刷新以及增量刷新 3 种模式，通过修改数据新鲜度FRESHNESS的定义来实现代码的批和流运行。

关于这一点，本人还是持谨慎怀疑的态度。

从某种意义上来说，代码层面的统一仅仅是解决批流一体中的「代码兼容性问题」，这是批流一体很小的一部分。

Flink社区对批流一体的关注点在于成本的节省，非常低成本的任务时效切换，但是其实这个点其实是批流一体场景中最不重要的一点。

因为能做到这种切换的业务场景其实并不多，大部分场景无法做到完全的批流一体，不过这仍然是一种进度。

### Streaming WareHouse

这个已经是老生常谈的话题了。社区未来会进行Flink和Paimon的深度集成。

但是我还是之前的观点，Paimon并没有给传统的数仓开发模式带来「革命性的进步」，但是的确解决了部分痛点。

Streaming warehouse要解决的是传统的离线/实时数仓中的痛点，而不是为了构建「纯流式的数据仓库」。

Paimon未来作为批流一体存储引擎前途仍然光明。

最后是关于一些AI的话题，这个就不过多介绍了，和大多数读者没关系。

> **[**300万字！全网最全大数据学习面试社区等你来！**](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247518343&idx=1&sn=eaf44bee8c4d90aedd508201475b57a9&chksm=fd3ece12ca494704cdf02300bbfe39c1d4245ac30f41aba3065efe66eb9aa453a23e0b6dae6e&scene=21#wechat_redirect)**

如果这个文章对你有帮助，不要忘记 **「在看」** **「点赞」** **「收藏」** 三连啊喂！

[全网首发|大数据专家级技能模型与学习指南(胜天半子篇)](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247510040&idx=1&sn=aa335f25965975731173916f012d56f4&chksm=fd3eee8dca49679b82f632048fb64d21fac01497d1a0fe33917edb01e194caf0f9a1930a55ce&scene=21#wechat_redirect)

[互联网最坏的时代可能真的来了](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247508317&idx=1&sn=0bcb7fb6997b42306994b890eaa0d47f&chksm=fd3ee7c8ca496ede347a2971002c6ea68dcfd70abeeb90b43c8b02a2a60ebc7cec3a87ef7d52&scene=21#wechat_redirect)

[我在B站读大学，大数据专业](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247507860&idx=1&sn=807ac5003762f29c127bc4071dcebe33&chksm=fd3e9901ca4910178cfc816043ea86c29f881f70cd943d252de9ae2e21cb7e8ba80bff162e1b&scene=21#wechat_redirect)

[我们在学习Flink的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247499604&idx=1&sn=d938dfb30d221774704982d2938b30c1&chksm=fd3eb9c1ca4930d76a391241333de461ca22d2aa27472eab3cffab1564872ae37f1b48fe2d3c&scene=21#wechat_redirect)

[193篇文章暴揍Flink，这个合集你需要关注一下](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504856&idx=2&sn=6f62e2a0c756ce56773eed253d2f41ee&chksm=fd3e954dca491c5b732b7e2aa46db32efbc3f690e528522098659bb3c6e78e1582ab4529b5dc&scene=21#wechat_redirect)

[Flink生产环境TOP难题与优化，阿里巴巴藏经阁YYDS](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504742&idx=1&sn=8765c198d8ad66219a7bcabb221e4a23&chksm=fd3e95f3ca491ce52d0724b9e4154a47af0f5ea349e1e184c8fbc65aa17c0000242aa9b58429&scene=21#wechat_redirect)

[Flink CDC我吃定了耶稣也留不住他！| Flink CDC线上问题小盘点](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504813&idx=1&sn=f5cd6ae2aa2b1e30f87a5ae55971c514&chksm=fd3e9538ca491c2eb191677d070f2c7e4f1098eece00e256d6205b8a5d21b21c1e74f875f16f&scene=21#wechat_redirect)

[我们在学习Spark的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=1&sn=1f693e549f76622725b936041ff8896e&chksm=e9bff2fbdec87bedecbdeed4547d7d612c72fc8d4918614a26a4e31666cf0128fd57b537f347&scene=21#wechat_redirect)

[在所有Spark模块中，我愿称SparkSQL为最强！](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504834&idx=1&sn=46ca1a3924b8fdb89ac1c2acb0319d6c&chksm=fd3e9557ca491c41d837b917639ea62007ea16d3c2cc6c0c02d1f8a8c6e44318f5183b787c46&scene=21#wechat_redirect)

[硬刚Hive | 4万字基础调优面试小总结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247502750&idx=1&sn=bd9a9173d060dc4e4ebd49c8efc6acfe&chksm=fd3e8d0bca49041dea84da93910e5efdc4935e520525c09887c986691377aeb48e5cf7fb5667&scene=21#wechat_redirect)

[数据治理方法论和实践小百科全书](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504382&idx=2&sn=550b78802acfe727e0e77cd9195f8784&chksm=fd3e976bca491e7db2b8b2446d231736df01bbf13653804d680ac4390597ec17fa466ad4ae83&scene=21#wechat_redirect)

[标签体系下的用户画像建设小指南](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503741&idx=1&sn=e5039be93123f2e337013756a818bfc3&chksm=fd3e89e8ca4900fe603b63c5722a6fb8a32bd63d6ba23e0028851948a71b877eb1f742d95087&scene=21#wechat_redirect)

[4万字长文 | ClickHouse基础&实践&调优全视角解析](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503675&idx=1&sn=3ee6af64d0126c78b48cad219308f81e&chksm=fd3e89aeca4900b8b8954e9569ee3c0877881fac8c792bfafc22e7e9d3e8524da8eb860d33d8&scene=21#wechat_redirect)

[【面试&个人成长】社招和校招的经验之谈](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=2&sn=57ecc77718f1b6f2f262d62e8318dcc9&chksm=e9bff2fbdec87bedb1986765bfae7dbabb9aece45b0b2af147bbad1ac5d79ebc934c64df40ea&scene=21#wechat_redirect)

[大数据方向另一个十年开启 |《硬刚系列》第一版完结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504478&idx=1&sn=14efef3868ba42bd044f9618745a7fdc&chksm=fd3e94cbca491dddb961b7b8b93105b2869c5bcf4c03f9f0dc83ad62be0dce4c124b52ed0ba3&scene=21#wechat_redirect)

[我写过的关于成长/面试/职场进阶的文章](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504410&idx=1&sn=7e81ab5a324395eb0f12397c40247ca1&chksm=fd3e948fca491d9946456acbd93b2d651ae4d7a7d0127a211981e08f50f377e60d1b7c0d7b3a&scene=21#wechat_redirect)

[当我们在学习Hive的时候在学习什么？「硬刚Hive续集」](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504783&idx=1&sn=72aed147a459368ed934a007a2df3bc9&chksm=fd3e951aca491c0cade212390011eca7d8a68951fee6aa2844b8f4d14bcbd5b3ed26305d69dc&scene=21#wechat_redirect)