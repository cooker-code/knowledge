> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/030302_核心知识点/FlinkCDC3数据集成架构|FlinkCDC3数据集成架构]]、[[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/030302_核心知识点/FlinkCDC版本演进与Pipeline连接器边界|FlinkCDC版本演进与Pipeline连接器边界]]
---
title: Flink CDC 3.0  新一代实时数据集成框架
author: 3分钟秒懂大数据
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247512113&idx=1&sn=6998255d45d478e38ef1353d76854e2a&chksm=c019098ef76e80985d4574cb3595dfc9c07ace8ce67e82daa9a5a3d20d57fcb12483cd1237f4&mpshare=1&scene=24&srcid=0102a5e1YA1i6Prmfa31zB4r&sharer_shareinfo=bf004282e4b16ac37ae42b5aa307fdc9&sharer_shareinfo_first=bf004282e4b16ac37ae42b5aa307fdc9#rd
---

```
点击上方公众号进入 3分钟秒懂大数据 主页

然后点击右上角 “设为标星” 

比别人更快接收硬核文章
```

大家好，我是土哥。

前几天 Flink CDC 3.0 数据集成框架已经发布，下面给大家分析一下 3.0 的功能用处都有哪些

## 1 概述

首先，Flink CDC 是一个实时的数据集成框架，支持了全增量一体化、无锁读取、并行读取、表结构变更自动同步、分布式架构等高级特性。FlinkCDC 支持多种常见的数据库，如 MySQL、PostgreSQL、Oracle 等，并提供了对应的连接器，使其能够方便地与这些数据库进行集成。

## 2 Flink CDC 3.0 新架构

如下图所示：

Flink CDC 3.0 的整体架构自顶而下分为 4 层：

* Flink CDC API：面向终端用户的 API 层，用户使用 YAML 格式配置数据同步流水线，使用 Flink CDC CLI 提交任务
* Flink CDC Connect：对接外部系统的连接器层，通过对 Flink 与现有 Flink CDC source 进行封装实现对外部系统同步数据的读取和写入
* Flink CDC Composer：同步任务的构建层，将用户的同步任务翻译为 Flink DataStream 作业
* Flink CDC Runtime：运行时层，根据数据同步场景高度定制 Flink 算子，实现 schema 变更、路由、变换等高级功能

现在针对上面的几点简单介绍一下：

## 3 核心点讲解

### 3.1 面向终端用户的 API 层

在FlinkCDC 3.0 API 设计中，用户无需关心框架实现，因为这些底层已经全部封装好，用户在使用时，只需要在 YAML 格式的配置文件中描述输入源和输出源，即可快速构建一个数据同步任务。

我们以从MYSQL 到 Doris 为例写一个 YAML 文件，如下图所示：在配置文件中，配置Mysql 和Doris 的相关信息，就可以完成构建。

### 3.2 Pipeline Connector API 设计

如下图所示：

根据上图可以简单理解成，Flink CDC 3.0 的 DataSource 为从外部上系统中读取变更事件的Event ，并将其传递给下游算子。DataSink 是将上游算子传递来的变更事件 Event 写出至外部系统。

同时 DataSource 和 DataSink 在设计上复用 Flink Source 和 Sink，开发者可以快速基于 Flink connector 对接 Flink CDC 3.0 框架，将外部系统高效地接入 Flink CDC 的上下游生态。

### 3.3 整库同步设计

用户可以在 Flink CDC 3.0 的配置文件中指定 DataSource 同步任务捕获上游多表或整库变更，结合 Schema Evolution 的设计，SchemaRegistry 会在读取到新表的数据后，自动在目标端外部系统建表，实现自动化的数据整库同步。

### 3.4 分库分表同步设计

在数据同步中，一个常见的使用场景是将上游由于业务或数据库性能问题而拆分的多表在下游系统合并为一张表。Flink CDC 3.0 使用路由（Route）机制实现分库分表合并的能力。用户可以在配置文件中定义 route 规则使用正则表达式匹配多张上游表，并将其指向同一张目标表，实现分库分表数据的归并。

总体来说，Flink CDC 3.0 不仅提供基础的数据同步能力，schema 变更自动同步、整库同步、分库分表等增强功能使 Flink CDC 3.0 在更复杂的数据集成与用户业务场景中发挥作用：用户无需在数据源发生 schema 变更时手动介入，大大降低用户的运维成本；只需对同步任务进行简单配置即可将多表、多库同步至下游，并进行合并等逻辑，显著降低用户的开发难度与入门门槛。

以上就是 Flink CDC 3.0 的一些基本介绍，想了解更多的，可以通过 Flink 中文社区了解更多内容~

End

互联网行业竞争越来越激烈，如果你因为找工作而烦恼，不会改简历，不会备战复习，不会面试技巧，不会 HR 面，不会谈薪，不用怕，有土哥。

[土哥社招参加 28 场面试，100% 通过率，拿到全部 offer！](http://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247511408&idx=1&sn=beb292ab97ada3ee486511bfe503117d&chksm=c01914cff76e9dd90fd81857805a57aadcf4fa0a3ce731e5939d8651ed9bac561dba6bb7e03a&scene=21#wechat_redirect)

[土哥这半年的悲惨人生，经历过被鸽 offer,最终触底反弹~](http://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247510455&idx=1&sn=9cccfbebca3d2ee9d72538d73dd6fe74&chksm=c0191008f76e991e1760857f7c8a75deb1e0231ce8ba189eaf280c84e8a5e65f7f0197988ff1&scene=21#wechat_redirect)

**可以找土哥修改简历，1 对 1 辅导项目、面试技巧，HR 面以及谈薪等，同时发你总结的最新面经试题（有偿哈），具体价格私信土哥。**

毕竟免费的东西，不仅你不会上心，土哥也没有多大精力认真去修改和辅导。

如果有任何意愿，可以加土哥微信，备注：简历修改+面试辅导，当然，想进群的，也可以扫码，备注：加群~