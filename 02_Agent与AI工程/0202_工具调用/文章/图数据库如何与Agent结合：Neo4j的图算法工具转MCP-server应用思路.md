---
title: 图数据库如何与Agent结合：Neo4j的图算法工具转MCP-server应用思路
author: 老刘说NLP
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxMjc3MjkyMg==&mid=2648422821&idx=2&sn=972c87c70b1eba6c622e42d43229fe2b&chksm=825e6e4c5c229d636f5843abe5a4d39d263632236e1df833baeb2be46c0557e3d60b1b83d82c&mpshare=1&scene=24&srcid=09125qFencUMOTlTxSvcad1Q&sharer_shareinfo=f787260ca22680c93746ee4839f05694&sharer_shareinfo_first=f787260ca22680c93746ee4839f05694#rd
---

今天是2025年9月4日，星期四，北京，晴

来看**知识图谱**进展，来看图数据库Neo4j的图算法应用智能体思路，既然内部有一些现成的图算法库，那么，**就把这些算法搞成mcp-server，然后接一个llm，借助llm的function-call能力，来做一个agent，实现一个复杂些的任务，这是个不错的故事**。

多总结，多归纳，多从底层实现分析，会有收获。

## 图数据库Neo4j的图算法应用智能体思路

图数据库厂商也在谋出路，也需要贴近Agent讲故事，例如 **Neo4j搞的GDS Agent（图数据科学智能体）** ，GDS Agent: A Graph Algorithmic Reasoning Agent，https://arxiv.org/pdf/2508.20637，https://github.com/neo4j-contrib/gds-agent，想解决的问题是大模型在大规模图结构数据处理与算法推理上的不足，**然后既然都内置了一些工具，那么何不将这些工具都用起来**。

**来看能够得到什么启发**，看看他是怎么做的？

**第一步：包装成mcp-server**

思路很简单，很工程化，**就是把neo4j内置的一些算法工具搞成mcpserver**，共46种工具（v0.3.0版本），包括辅助工具，节点计数、获取节点/关系属性键（如get\_node\_properties\_keys）、图算法工具，中心性（如PageRank、介数中心性）、社区检测（如Louvain、Leiden）、路径查找（如Dijkstra、Yen算法）、相似度计算（如余弦相似度）等。

例如，将**yens\_shortest\_paths**算法的转换：

**第二步，接入llm，依靠llm的function-call能力进行调用**，这个本质上就是上Agen t了。

为了验证这个逻辑是通的，所以搞了个Benchmark，https://github.com/brs96/gds-agent-benchmarks，从中可以看到对应的tool-call，tool parameters等。

最后，**看一个具体例子，具像化理解**：

对应的执行步骤如下：

拆解看，也很好理解，用户如何询问从帕丁顿（Paddington）到伦敦桥（LondonBridge）的最快路线，GDSAgent通过调用不同的工具来获取最终答案的过程。

首先，用户询问从帕丁顿到伦敦桥的几种最快路线。

其次，**工具调用**，先后执行**get\_node\_properties\_keys**获取节点属性的键，如名称、ID、总线路数、区域等；**get\_relationship\_properties\_keys**，获取关系属性的键，如线路、距离、时间等；**yens\_shortest\_paths**调用Yen算法来计算最短路径，指定起点（Paddington）、终点（LondonBridge）和路径数量（k=3），工具调用中返回的原始数据，**包括路径的详细信息，如成本、节点ID和路径本身**；

最后，**大模型根据用户问题和所有工具返回的上下文生成最终答案**，从帕丁顿到伦敦桥的前三条最快路径，其中最快的路径是帕丁顿→贝克街→伦敦桥，总耗时25分钟，共9个站点。

所以，**其中的根本，还是要看大模型调用工具的能力，很多产品找出路，都在往前沿热点上靠拢**。

## 参考文献

1、https://github.com/neo4j-contrib/gds-agent

## 关于我们

老刘，NLP开源爱好者与践行者，主页：https://liuhuanyong.github.io。

**对大模型&知识图谱&RAG&文档理解感兴趣，并对每日早报、老刘说NLP历史线上分享、心得交流等感兴趣的，欢迎加入社区，社区持续纳新。**

加入社区方式：关注公众号，在后台菜单栏中点击会员社区加入。