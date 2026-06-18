---
title: Apache Spark目前发展到了什么程度？
author: 大数据技术与架构
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247524409&idx=1&sn=bb34c564833e18313bb2145a8ba6dda1&chksm=fc0f0782a4facbc0f46826552785e4e4f2c234cd5daff14751c76f59fd0832932df70dd1075f&mpshare=1&scene=24&srcid=07048XLJqnFl3btLQi4w1MFC&sharer_shareinfo=d8ad57aa0f93fdc311a628e2ccbf290c&sharer_shareinfo_first=d8ad57aa0f93fdc311a628e2ccbf290c#rd
---

前天发表了一篇关于Hive现状的随笔文章[《Apache Hive还有未来吗？》](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247524398&idx=1&sn=382d2e3076e79d4fb89f9908a1f5a9e3&scene=21#wechat_redirect)，没想到引起很多读者的共鸣，数据时代框架的发展之快远超过你我的预期，各位都是这个过程的亲历者。

今天我们聊一下Spark。

2025年5月23日，Spark4.0版本发布，但是网络上没有太大的反响。

原因很简单，中文社区中的Flink独领风骚，和高速发展的各类湖仓框架已经高频次轰炸了各位的大脑。路边的孩童都听过「实时即未来」这个Slogan。

再加上2025年初DeepSeek的横空出世，AI大模型时代来临的如此之快。

数据领域的王者框架Spark已经到了可以被忽视的地步。

目前Github上Spark的Star数高达41.4K，应该是整个数据开发领域关注度最高的框架，可以说没有之一(如有错误，欢迎评论区指正)。Github的pr数量常年维持在200左右。

事实上，如果说一家成熟的大公司的数据部门只保留一个计算框架的话，Spark应该是最有竞争力的候选者，甚至没有之一。

在大家面试的过程中，如果我们只考察一个框架的掌握程度，那么基本非Spark莫属。

Spark4.0版本可以算是Spark社区的一个巨大的里程碑版本。

在官方的Release Note中甚至用了「significant milestone」这样的词语来形容，足可见Spark社区对这个版本的重视程度。

如果你把Spark4.0版本的Release Note打印出来，那么你会发现这个Release Note高达45页。

我们大概过一下社区的本次发布：

```
- A new lightweight Python client (pyspark-client) at just 1.5 MB.  
- An additional release tarball with Spark Connect enabled by default.  
- Full API compatibility for the Java client.  
- A new spark.api.mode configuration to easily turn on/off Spark Connect for your applications.  
- Greatly expanded API coverage.  
- ML on Spark Connect.  
- A new client implementation for Swift.
```

这其中值得关注的点包括：

1. **PySpark 增强**：引入原生绘图功能、Python数据源API，支持多态用户定义表函数（UDTF），显著提升Python开发者的生产力；
2. **Spark Connect GA**：作为协议层的核心改进，Spark Connect 实现了客户端与驱动程序的解耦，支持 Go、Python 等语言的轻量化客户端开发，用户可通过文本编辑器直接调试远程集群，极大降低了开发门槛；
3. **SQL 与查询优化**：通过SQL脚本支持、ANSI SQL兼容模式扩展，以及动态分区剪枝、自适应查询执行（AQE）等优化，进一步提升Spark SQL的性能和兼容性。

此外在性能提升上，Spark不遗余力的推进向量化优化来提升性能。在此之前，快手把内部的向量化引擎Blaze做了开源，可以做到30%的算力提升，大家可以网上搜索相关文章了解。

关于Spark的向量化执行之前写过的一篇文章，可以参考：[《除了调参/AQE/数据倾斜等，Spark还有什么方式能显著提升性能？》](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247524369&idx=1&sn=d06bd8f291899ab84f64e7b36b9a738a&scene=21#wechat_redirect)。

另外就是跨语言执行优化，例如通过LLVM或JIT编译技术加速表达式计算，以及与Rust生态的深度整合，以进一步降低执行开销。

这里我们不得不感叹一句，性能优化在Spark社区一直以来都是核心命题，从未更改。

天下武功，无坚不破，为快不破。

此外在AI领域，Spark也一直在发力，例如支持与TensorFlow、PyTorch的无缝集成，并探索基于Spark的分布式训练框架。

在Spark RoadMap 2025中，探索基于AI的查询优化（如自动调参、执行计划生成），以及与大语言模型的集成上，也已经被提上了日程。

如果说Hive社区面对各种各样的后来者不得已做出了转型，凭借在稳定性以及基础建设上的先发优势，使得Hive在数据开发领域还能有一席之地。

Spark社区则仍然以一个引领者的姿态站在整个数据开发领域的潮头。

社区也在积极的拥抱新时代数据领域的挑战，你看到的无论是向量化执行、AI融合，还是生态层面的云原生支持、跨框架协作，都体现了社区对未来趋势的前瞻性布局。

当然，未来总有一天，新时代的后来者会取代或者部分取代Spark在数据开发领域的核心地位。

但这不正是大家所期望的：

道德三皇五帝，功名夏后商周。七雄五霸斗春秋，顷刻兴亡过手。 

青史几行名姓，北邙无数荒丘。前人田地后人收，说甚龙争虎斗。

---

最后，欢迎加入我们的知识星球小圈子：  
[《300万字！全网最全大数据学习面试社区等你来》](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247518343&idx=1&sn=eaf44bee8c4d90aedd508201475b57a9&chksm=fd3ece12ca494704cdf02300bbfe39c1d4245ac30f41aba3065efe66eb9aa453a23e0b6dae6e&scene=21&token=1745806505&lang=zh_CN#wechat_redirect)。

如果这个文章对你有帮助，不要忘记 **「在看」** **「点赞」** **「收藏」** 三连啊喂！