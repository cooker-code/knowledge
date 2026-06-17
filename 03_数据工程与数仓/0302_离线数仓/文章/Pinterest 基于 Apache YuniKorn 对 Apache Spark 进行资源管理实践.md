---
title: Pinterest 基于 Apache YuniKorn 对 Apache Spark 进行资源管理实践
author: 漫谈大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkzODIzNTQwNA==&mid=2247485085&idx=1&sn=ffc70060f7ed7cd3e1c2ac937c18f02c&chksm=c32dcab298b975270f663ee1806bba4d351aa6948605133866d7f1593f35dead9ac2ac7d5a5f&mpshare=1&scene=24&srcid=1106CMqdzKASrX9S6U35b8bl&sharer_shareinfo=c79add5fbf3552b8e814d3270f85d9f8&sharer_shareinfo_first=c79add5fbf3552b8e814d3270f85d9f8#rd
---

Monarch 是 Pinterest 的批处理平台，最初旨在大规模支持 Pinterest 不断增长的 Apache Spark 和 MapReduce 工作负载。在 Monarch 于 2016 年成立期间，构建该平台的最主要批处理技术是 Apache Hadoop YARN。现在八年过去了，我们决定从 Apache Hadoop 迁移到基于下一代 Kubernetes （K8s） 的平台。以下是我们旨在解决的一些关键问题：

* • 使用容器化进行应用程序隔离：在 Apache Hadoop 2.10 中，YARN 应用程序共享相同的通用环境，没有容器隔离。这通常会导致应用程序之间难以调试依赖项冲突。
* • GPU 支持：Apache Hadoop YARN 的容量调度程序 （YARN-2496[1]） 而不是 Fair Scheduler （YARN-2497[2]） 中添加了节点标记支持，但在 Pinterest，我们在 Fair Scheduler 上投入了大量开发。升级到 Fair Scheduler 中支持节点标记的较新 Apache Hadoop 版本或迁移到 Capacity Scheduler 将需要大量的工程工作。
* • Hadoop 升级工作：2020 年，我们从 Apache Hadoop 2.7 升级到 2.10。此次要版本升级过程大约需要一年时间。升级到 3.x 的主要版本将花费我们更多的时间。
* • Hadoop 社区支持：整个行业一直在转向 K8s，而 Apache Hadoop 在很大程度上处于维护模式。

在过去的几年里，我们观察到行业广泛采用 K8s 来应对这些挑战。经过漫长的内部概念验证和评估，我们决定构建基于 K8s 的下一代平台：Moka（K8s 上的 Monarch）。

在本博客中，我们将介绍挑战和相应的解决方案，这些挑战和解决方案使我们能够将数千个 Spark 应用程序从 Monarch 迁移到 Moka，同时保持资源效率和高质量的服务。您可以查看我们之前的博客[3]，其中介绍了如何有效管理 Monarch 资源，以便在确保工作稳定性的同时节省成本。在撰写本博客时，我们已经将 Monarch 上运行的 Spark 工作负载的一半迁移到 Moka。在 Monarch 上运行的其余 MapReduce 作业正在积极迁移到 Spark，这是在 Pinterest 中弃用 MapReduce 的单独计划的一部分。

# 实现高效的资源管理

我们对 Moka 资源管理的目标是保留 Monarch 资源管理的积极特性并改进其缺点。

首先让我们介绍一下我们想带到 Moka 的 Monarch 中运行良好的功能：

1. 1. 将每个工作流与其所有者的组织和项目相关联
2. 2. 将所有工作流分为三个层：第 1 层（最高优先级）、第 2 层和第 3 层，并为每个工作流定义运行时 SLO
3. 3. 使用分层 org-base-queue （OBQ），格式为  
   根.<组织>.<项目>.

    用于资源分配和工作负载调度
4. 4. 跟踪每个应用程序的运行时数据，包括应用程序的开始和结束时间、内存和 vCore 使用情况等。
5. 5. 高效的基于层的资源分配算法，可根据分配给队列的工作流的历史资源使用情况自动调整 OBQ
6. 6. 可以将工作负载从一个集群路由到另一个集群的服务，我们称之为跨集群路由 （CCR），以达到其相应的 OBQ
7. 7. 使用预留资源载入队列以载入新工作负载
8. 8. 定期资源分配流程根据载入队列运行期间收集的资源使用数据管理 OBQ
9. 9. 用于监控资源利用率和工作流运行时 SLO 性能的仪表板

以编程方式将工作负载从 Monarch 大规模迁移到 Moka 的第一步是资源分配。上面列出的大多数项目都可以按原样重复使用，也可以轻松扩展以支持 Moka。但是，我们还需要一些额外的项目来支持 Moka 上的资源分配：

1. 1. 应用程序感知的调度程序，允许将集群资源作为分层队列进行管理，并支持抢占
2. 2. 一个管道，用于导出 Moka 上运行的所有应用程序的资源使用信息，以及用于生成洞察表的摄取管道
3. 3. 使用洞察表生成 OBQ 资源分配的资源分配作业
4. 4. 能够将工作负载路由到其目标集群和 OBQ 的编排层

现在让我们详细介绍一下我们如何解决这四个缺失的部分。

# 调度

默认的 K8s 调度程序是一个 “万事通，无所不能” 的解决方案，它不是特别擅长调度批处理工作负载。为了满足我们的资源调度需求，我们需要一个支持分层队列的调度程序，并且能够基于每个应用程序和每个用户而不是每个 Pod 进行调度。

Apache YuniKorn[4] 由一群在 Apache Hadoop YARN 和 K8s 方面具有丰富经验的工程师设计。Apache YuniKorn 不仅可以识别用户、应用程序和队列，还包括许多其他因素，例如在做出调度决策时的顺序、优先级和抢占。

鉴于 Apache YuniKorn 具有我们需要的大多数属性，我们决定在 Moka 中使用它。

# 资源使用情况洞察信息

如前所述，应用程序资源使用历史记录对于我们如何进行资源分配至关重要。概括地说，我们使用队列的历史使用情况作为基准来估计在分配过程的每次迭代中应分配多少。但是，当我们决定首先采用 Apache YuniKorn 时，它缺少这一重要功能。Apache YuniKorn 是完全无状态的，只跟踪整个集群的瞬时资源利用率。

我们需要一种解决方案，该解决方案能够可靠地跟踪属于应用程序的所有 Pod 的资源消耗，并且具有容错能力。为此，我们与 Apache YuniKorn 社区密切合作，增加了对记录已完成应用程序的资源使用信息的支持（有关更多信息，请参阅 YUNIKORN-1385[5]）。此功能汇总每个应用程序的 Pod 资源使用情况，并在应用程序完成后报告最终的资源使用情况摘要。

此摘要记录到 Apache YuniKorn 的 stdout 中，我们使用 Fluent Bit 筛选出应用程序摘要日志并将其写入 S3。

根据设计，Apache YuniKorn 应用程序摘要包含与 YARN ResourceManager 生成的 YARN 应用程序摘要类似的记录（请参阅本文档[6]中的更多详细信息），以便它可以无缝地适应 YARN 应用程序摘要的现有使用者。

除了应用程序资源使用情况信息外，以下映射信息还会自动摄取到见解表中：

* • 到项目、层、SLO 的工作流程
* • 项目到所有者
* • 所有者到公司组织
* • 工作流作业到应用程序

此信息用于将工作负载与其目标队列相关联，并估计队列的未来资源需求。

图 1 显示了信息摄取和资源分配流程。

# 资源分配算法改进

要了解有关资源分配算法的更多信息，请参阅我们之前的博客文章：Pinterest 批处理平台的高效资源管理[7]。

我们的算法将资源优先分配给第 1 层和第 2 层队列，直至达到队列所需资源的指定百分位阈值。

这种方法的一个缺点是，它通常需要多次迭代的资源分配才能收敛到一个稳定的迭代，每次迭代都需要对参数进行一些手动调整。

作为 Moka 迁移的一部分，我们设计并实施了一种全新的算法，该算法利用 OR-Tools[8] 开源套件中的 Constraint Programming Using CP-SAT[9]进行优化。此工具通过根据高层（第 1 层和第 2 层）和低层（第 3 层）资源请求之间的使用/容量比率差距构建一组约束来生成模型。这种新的资源分配算法无需人工干预即可更快、更可靠地运行。

# OBQ 和 CCR 路由

我们的作业提交层 Archer 负责处理向 Moka 提交的所有作业。Archer 在平台级别提供了灵活性，以便在提交时分析、修改和路由作业。这包括将作业路由到特定的 Moka 集群和队列。

图 2 显示了如何使用 CCR 为选定作业和部署过程进行资源分配。在 git repo 中所做的资源分配更改会自动提交给 Archer，Archer 与 k8s 通信以部署更改的资源分配配置映射，然后根据 Archer 路由数据库中设置的 CCR 规则在运行时路由作业。

我们计划在以后的博客文章中介绍 Archer 和 Moka。

# 一些关键的 Apache YuniKorn 功能和修复

除了 YUNIKORN-1385[10] 之外，以下是我们回馈给 Apache YuniKorn 社区的其他一些功能和修复。

* • YUNIKORN-790[11]：添加了对 maxApplications 的支持，以限制可在任何队列中并发运行的应用程序数
* • YUNIKORN-2030[12]：修复了检查余量时导致 Apache YuniKorn 停滞的错
* • YUNIKORN-970[13]添加队列指标，将队列名称作为标签，使队列指标更易于跟踪和查看
* • YUNIKORN-1948[14] 引入一个命令来验证给定队列配置文件的内容

Apache YuniKorn 仍然是一个相对年轻的开源项目，我们将继续与社区合作，以丰富其功能并提高其可靠性和效率。

# 工作流 SLO 监控

一旦我们开始将真实的生产工作负载载入 Moka，我们就扩展了现有的 Monarch 工作流 SLO 性能仪表板，以包括在 Moka 上运行的应用程序的每日运行时结果。我们的目标是确保至少 90% 的 1 级工作流在至少 90% 的时间内满足其 SLO。

# 未来的工作

尽管在构建 Moka 平台和从旧平台迁移工作方面取得了巨大进展，但我们仍计划进行许多改进。

我们正在设计一个有状态的服务，该服务能够利用引入事件流支持的 YUNIKORN-1628[15] 和 YUNIKORN-2115[16]。

此外，我们正在开发一个功能齐全的资源管理控制台来管理平台资源。此控制台将使平台管理员能够实时监控集群和队列资源的使用情况，并允许在集群之间进行自定义负载均衡。

原文链接：https://medium.com/pinterest-engineering/resource-management-with-apache-yunikorn-for-apache-spark-on-aws-eks-at-pinterest-0dba3afb4609

#### 引用链接

`[1]` YARN-2496: *https://issues.apache.org/jira/browse/YARN-2496*  
`[2]` YARN-2497: *https://issues.apache.org/jira/browse/YARN-2497*  
`[3]` 博客: *https://medium.com/pinterest-engineering/efficient-resource-management-at-pinterests-batch-processing-platform-61512ad98a95*  
`[4]` Apache YuniKorn: *https://yunikorn.apache.org/docs/get\_started/core\_features/#:~:text=The%20default%20K8s%20scheduler%20simply,etc%2C%20while%20making%20scheduling%20decisions.*  
`[5]` YUNIKORN-1385: *https://issues.apache.org/jira/browse/YUNIKORN-1385*  
`[6]` 本文档: *https://issues.apache.org/jira/secure/attachment/13051969/yunikornResourceUsageVisibility.pdf*  
`[7]` Pinterest 批处理平台的高效资源管理: *https://medium.com/pinterest-engineering/efficient-resource-management-at-pinterests-batch-processing-platform-61512ad98a95*  
`[8]` OR-Tools: *https://developers.google.com/optimization*  
`[9]` 中的 Constraint Programming Using CP-SAT: *https://developers.google.com/optimization/cp/cp\_solver*  
`[10]` YUNIKORN-1385: *https://issues.apache.org/jira/browse/YUNIKORN-1385*  
`[11]` YUNIKORN-790: *https://issues.apache.org/jira/browse/YUNIKORN-790*  
`[12]` YUNIKORN-2030: *https://issues.apache.org/jira/browse/YUNIKORN-2030*  
`[13]` -970: *https://issues.apache.org/jira/browse/YUNIKORN-970*  
`[14]` YUNIKORN-1948: *https://issues.apache.org/jira/browse/YUNIKORN-1948*  
`[15]` YUNIKORN-1628: *https://issues.apache.org/jira/browse/YUNIKORN-1628*  
`[16]` YUNIKORN-2115: *https://issues.apache.org/jira/browse/YUNIKORN-2115*