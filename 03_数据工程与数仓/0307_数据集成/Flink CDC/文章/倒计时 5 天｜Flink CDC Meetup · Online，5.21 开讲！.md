---
title: 倒计时 5 天｜Flink CDC Meetup · Online，5.21 开讲！
author: Apache Flink
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247497346&idx=1&sn=e50775f8932ec46efdfb504624389f98&chksm=fd3878c0ca4ff1d6bd858c51c2c44b35c7e021e0e5cc370c342eebccbd39092bb2f66b0ef965&mpshare=1&scene=24&srcid=0518HDMc7ailhD31lQy0ppkN&sharer_sharetime=1652833515817&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

当下数据规模正在以惊人的速度增长，越来越多的应用场景也对数据处理的时效性有了更高的要求。随着近几年实时计算技术的迅猛发展，涌现了实时 OLAP、实时数据湖、实时数仓等架构，较好地解决了湖仓实时化问题。然而实时化需要的是端到端的解决方案，除了湖仓实时化之外，我们还急需数据集成的实时化。

实时数据集成是指将各个数据孤岛中的数据实时地同步、集中到数据仓库中，便于后续进行统一的实时分析。实时数据集成是数据技术栈实时化的重要组成部分，也是目前业界的主流发展趋势。与离线数据集成不同，实时数据集成需要面对随时都可能发生变化的数据与结构，除了需要保证低延迟地同步到目标存储中，还需要保证在各种场景下的数据一致性、正确性等问题。

Flink CDC 是实时数据集成框架的开源代表，具有全增量一体化、无锁读取、并发读取、分布式架构等技术优势，在开源社区中非常受欢迎。除了具备实时入湖入仓能力，Flink CDC 还支持强大的数据加工能力，可以通过 SQL 对数据库数据做实时关联、聚合、打宽等。

**Flink CDC Meetup · Online**

**5月21日 | 线上**

为了促进 Flink CDC 技术的交流和发展，我们将于 5 月 21 日在线举办 Flink CDC Meetup。本次 Meetup 由阿里巴巴技术专家，Apache Flink PMC Member & Committer 伍翀 (云邪) 作为出品人，邀请了来自阿里巴巴、XTransfer、顺丰、OceanBase、大健云仓的大咖分享 Flink CDC 在各场景中的最佳实践、生产经验、技术原理等。

【**活动亮点】**

* 超多实用干货，如 Flink CDC 实现海量数据的实时同步和转换的技术原理，以及各业务场景下的实践优化。
* 每位讲师均留有 Q&A 环节，通过社区钉群、微信群、视频号直播提出问题，均有机会得到讲师线上答复～
* 通过 ApacheFlink 视频号观看直播，将有机会获得 Flink CDC 定制 T恤！

【**活动议程】**

**嘉宾及议题介绍**

**伍翀**

**阿里巴巴技术专家**

**Apache Flink PMC Member & Committer**

**出品人简介：**

伍翀，花名云邪，Apache Flink PMC member & Committer。就职于阿里云开源大数据平台，主要负责 Flink CDC、Flink SQL 相关的研发工作，长期以来一直专注于流处理、批处理领域。

**《基于 Flink CDC 实现**

**海量数据的实时同步和转换》**

**徐榜江**

**阿里巴巴高级开发工程师**

**Apache Flink Committer & Flink CDC Maintainer**

**【嘉宾简介】**

徐榜江，阿里花名雪尽，目前专注数据集成领域。

****【演讲简介】****

1. 海量数据集成的痛点；
2. 基于 Flink CDC 实现海量数据的实时同步和转换；
3. Demo 演示：实时大屏；
4. 总结与展望。

****【听众受益】****

了解 Flink CDC 实现海量数据的实时同步和转换的技术原理，为业务提供更新鲜的数据。

**《Flink CDC MongoDB Connector**

**的实现原理和使用实践》**

**孙家宝**

**XTransfer 资深 Java 开发工程师**

****Flink CDC Maintainer****

**【嘉宾简介】**

孙家宝，任职于 XTransfer 基础架构部，负责大数据平台基础设施建设。 是 Flink CDC 项目 Maintainer 成员，Debezium、Zeppelin 等开源项目贡献者。

****【演讲简介】****

1. MongoDB ChangeStream 技术简介；
2. MongoDB CDC Connector 使用实践；
3. MongoDB CDC Connector 并行化 Snapshot 改进。

****【听众受益】****

受益对象：Flink CDC MongoDB 的用户和技术开发。

**《Flink CDC + Hudi**

**海量入湖在顺丰的实践》**

**覃立辉**

**顺丰大数据研发工程师**

**【嘉宾简介】**

覃立辉，任职于顺丰科技大数据底盘团队，主要从事数据入湖入仓相关的研发工作。

****【演讲简介】****

1. 顺丰数据集成背景
2. Flink CDC 实践问题与优化
3. 未来规划

**【听众受益】**

听众可以了解到在 Flink CDC 生产实践过程中遇到哪些问题与挑战，以及我们为解决这些问题对 Flink CDC 进行优化，支持全量与增量日志流并行读取、支持全量混合拆分解决数据倾斜，支持多 DB 实例的分库分表同步等功能。

**《Flink CDC + OceanBase**

**全增量一体化数据集成方案》**

**王赫**

**OceanBase 技术专家**

**【嘉宾简介】**

王赫 (川粉)，OceanBase 技术专家。

**【演讲简介】**

本次分享将从以下四部分带来 Flink CDC + OceanBase 全增量一体化数据集成方案： 

1. CDC 技术简介
2. OceanBase CDC 组件介绍
3. Flink CDC 简介
4. Flink CDC OceanBase Connector 简介

**【听众受益】**

了解 Flink CDC 和 OceanBase 社区版数据迁移相关的工具，了解 Flink CDC OceanBase Connector 的原理和使用，掌握分布式数据库 OceanBase 社区版与大数据处理引擎 Flink 的集成方案。

**《Flink CDC 在大健云仓的实践》**

**龚中强**

**大健云仓基础架构部负责人**

**【嘉宾简介】**

任职于大健云仓基础架构部，主要负责公司系统架构设计与开发。目前专注于大数据、云原生领域，有一定的实践经验和个人见解。

**【演讲简介】**

1. 公司引入 Flink CDC 的背景；
2. 现今 Flink CDC 内部落地的业务场景；
3. 未来 Flink CDC 内部推广以及平台化建设。

**【听众受益】**

1. 了解 Flink CDC 在公司内落地的业务场景和生产实践的经验；
2. 开拓应用 Flink CDC 业务场景的视野。

**活动详情**

**时间：**5 月 21 日 9:00-12:25

**PC 端直播观看：**https://developer.aliyun.com/live/248997

**移动端**建议关注 ApacheFlink 视频号预约观看

更多 Flink CDC 相关技术问题，可扫码加入钉钉交流群

▼ 关注「**Apache Flink**」，获取更多技术干货 ▼

更多 Flink 相关技术问题，可扫码加入社区钉钉交流群～

   **点击预约直播～**