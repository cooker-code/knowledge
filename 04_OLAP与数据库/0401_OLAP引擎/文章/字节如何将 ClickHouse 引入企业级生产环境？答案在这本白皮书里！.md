---
title: 字节如何将 ClickHouse 引入企业级生产环境？答案在这本白皮书里！
author: DataFunTalk
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247585591&idx=1&sn=11d98a57475b99fe38e25928697aa169&chksm=fbd63d5bcca1b44d0209f61f88fd78dfd0fad5ef3686503f4947d79be05da1f6e11c89849edd&mpshare=1&scene=24&srcid=0706lOEskkefbBRQJgjUfB6b&sharer_sharetime=1657083735654&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**导读****：**最近，火山引擎ByteHouse团队和InfoQ联合发布《从ClickHouse到ByteHouse》白皮书，着重探讨 ClickHouse 引入企业级生产环境过程中存在的问题以及现阶段的解法。

ClickHouse 开源于 2016 年，凭借性能方面的突出优势，在分析型数据库领域发展可谓风生水起。目前，国内外许多头部大厂都在深度使用 ClickHouse 技术。

**在性能方面**，ClickHouse 在 OLAP 场景下的性能超越同类产品数倍不止，**它允许系统以亚秒级的延迟从 PB 级的原始数据生成报告，服务器吞吐量高达每秒数亿行**。

但是将 ClickHouse 引入企业级生产环境中，仍然存在问题。关于落地实践的“坑”，并不是业内所有团队都需要自己踩一遍，也不是所有团队都能负担得起这样的成本，我们要做的是吸取足够的经验，以及选择自研、采购等更加实际的解决方案。

在这一点上，字节跳动无疑是一家非常有代表性的国内企业：字节跳动从 2017 年开始大规模启用 ClickHouse；作为其深度用户，字节跳动拥有国内规模最大的 ClickHouse 集群。

目前，**字节跳动内部的 ClickHouse 节点总数超过 1.8 万个，管理总数据量超过 700PB，最大的单个集群部署规模约为 2400 余个节点。**

当前，字节跳动已将经过五年定制化改造的 ClickHouse，沉淀为 ByteHouse，**正式通过火山引擎对外提供服务**。

从采用并改造开源产品，到上线商业版本对外服务，这是一条非常难走的路，同时也让其中的实践思考和经验更具参考价值。

最近，火山引擎 ByteHouse 联合 InfoQ 发布白皮书《从ClickHouse到ByteHouse》，深度介绍字节跳动万台节点ClickHouse背后的技术实现，**本卷白皮书大致分为四个章节**：

1. ClickHouse 介绍

2. ClickHouse 典型场景

3. 针对生产环境中的 ClickHouse，ByteHouse 的技术优化思考

4. ByteHouse 的设计和演进思路

其中，《从ClickHouse到ByteHouse》**从第三章开始，重点介绍 ByteHouse 的优化思路**。

目前，ByteHouse 对 ClickHouse 做了很多升级和优化，本次挑选了 ByteHouse 对 ClickHouse 优化升级中**非常重要的三个方面**详细展开：

1. 自研表引擎

2. 查询优化器

3. 弹性可扩展

在自研表引擎模块，尽管 ClickHouse 提供 MergeTree Family, Memory, File, Interface 等几十种不同的表引擎，但在字节内部实际使用中，还是明显感觉到表引擎不足以满足业务的使用需求，于是我们进行了相应的优化。

其中，**重点介绍了HaMergeTree、HaUniqueMergeTree、HaKafka三种表引擎**。

图1 白皮书配图摘选：HaMergeTree副本协同原理

**在查询优化器模块**，ByteHouse 对 Optimizer 进行了一年多的改造投入，全面升级产品能力，白皮书详细列举了 ByteHouse 在查询优化器上的改造与优化功能。

**为了追求极致性能**，ClickHouse 采用的是计算和存储节点强耦合的架构，不能根据各自实际需求分开扩容，而且在节点扩展后数据无法自动重新分布的问题给 ClickHouse 扩展带来很多运维的麻烦。

ByteHouse 在改进与优化 ClickHouse 的过程中，也重点基于该架构进行了调整，比如 ByteHouse 在存储和计算上的拆解解耦，实现弹性可扩展的技术优化方案。

图2 白皮书配图摘选：计算存储分离架构

除此之外，《从ClickHouse到ByteHouse》还枚举出广告、金融、工业互联网三大行业的实践案例，这些都属于 OLAP 的典型应用行业，并从技术与企业落地等角度给出了当下企业在 OLAP 数据引擎选型的三个核心关注点。

**点击 “阅读原文” 下载白皮书**