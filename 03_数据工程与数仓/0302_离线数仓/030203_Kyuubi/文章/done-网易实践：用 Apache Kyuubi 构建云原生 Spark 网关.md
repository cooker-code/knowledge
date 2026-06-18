---
title: 网易实践：用 Apache Kyuubi 构建云原生 Spark 网关
author: Apache Kyuubi
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511917&idx=1&sn=1884d7573ccc6d973b3dfa02b7e3bed9&chksm=cf824ca60a5885ecc70ca656b941302ae13e2a2fc699f257082257dbc8f3e9623e67298dcc3f&mpshare=1&scene=24&srcid=0210bRgqON08RRM31e8cCJBi&sharer_shareinfo=40fd4916477646955f743c3a7d3206cd&sharer_shareinfo_first=40fd4916477646955f743c3a7d3206cd#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030203_Kyuubi/030203_知识地图|知识地图]]


**导读** 本文基于网易数帆软件工程师、Apache Kyuubi/Zeppelin PMC 成员潘成老师的分享整理汇总。通过本文，可以从网易在 Spark 服务化的经验中，学习如何使用 Apache Kyuubi 构建统一的 Spark 网关，既满足业务团队多样的 Spark 使用方式，又可以适配不同基础设施环境中多样的 Spark 部署方式。

****文章主要内容包括以下几大部分：****

1. Apache Kyuubi 简介

2. Kyuubi 支撑多种 Spark 使用场景

3. Kyuubi 对接多样 Spark 部署形态

4. Kyuubi 在 Spark 4 时代的机遇与挑战

5. Q&A

分享嘉宾｜潘成 网易 软件工程师

编辑整理｜张俊光

内容校对｜李瑶

出品社区｜DataFun

---

**01**

**Apache Kyuubi****简介**

首先简单介绍一下 Kyuubi 这个项目。Kyuubi 是一个分布式、多租户的企业级大数据网关，在基础设施组件中，Kyuubi 处在接入层的位置。

Kyuubi 提供多种 API 以供不同类型的工作负载接入，典型的使用场景包括：使用 JDBC/BeeLine 以及各种 BI 工具，连接 Kyuubi 进行交互式的数据分析；使用 RESTful API向Kyuubi 提交 SQL/Python/Scala/Jar 批作业。

作为一个企业级组件，Kyuubi 深度适配了 Hadoop 生态标准的 Kerberos 安全体系，同时也支持企业常用的 LDAP 安全认证协议。特别的，客户端可以选择使用 LDAP 认证接入 Kyuubi，无缝桥接到开启 Kerberos 的 Hadoop 集群，简化用户接入成本。

除了作为网关的主体功能外，Kyuubi 项目还包含一系列可以独立使用的 Spark 插件，比如细粒度权限管理、Hive 表小文件治理、Z-Order、SQL 血缘提取、限制查询数据扫描量等常用功能。

为帮助大家更清晰地了解 Kyuubi，首先回答几个新接触 Kyuubi 同学常问的问题。

首先要明确的是，Kyuubi 不是计算引擎，而是一个网关；Kyuubi 与计算引擎处在不同的生态位，Kyuubi 搭配计算引擎可以构建企业级解决方案：

* Kyuubi 为不同的计算引擎，如 Spark、Trino，提供统一的接入协议、认证方式，简化了企业级场景多引擎的接入和管理成本。
* 对于部分计算引擎，比如 Spark、Flink，Kyuubi 可以按需拉起、销毁计算引擎实例，管理计算引擎的生命周期，实现极致的资源弹性。以 Spark 为例，当客户端发起到 Kyuubi Server 到连接后，Kyuubi Server 会首先到服务发现中心查询合适的计算引擎，若找不到，会调用 spark-submit 启动一个计算引擎；启动后的计算引擎会将自己的连接信息注册到服务发现中心，然后 Kyuubi Server 即可通过内置 RPC 与计算引擎通信。要注意的是，Kyuubi 目前仅支持 spark-submit 而非 Spark Operator，此项选择的考虑将会在后文详述。
* Kyuubi 提供了灵活的路由机制，支持按用户、组等规则将查询路由到指定计算引擎实例，满足多种场景下的多租户需求。

另一个很多用户关心的问题是，Kyuubi 既然支持多种计算引擎，是否也支持 SQL 方言转换呢？其实 Kyuubi 社区有很多用户提过此类需求，经过一系列的讨论，核心开发者总体并不期望涉足此方向：一方面由于不同引擎的固有差异，SQL rewrite 不能尽善尽美；另外这是一个相对独立的方向，我们看到比如由 LinkedIn 开源的 Coral 项目是专注于此领域的项目。因此，Kyuubi 不支持 SQL 方言转换，但会提供开发者 API，允许用户结合其他项目自定义插件去按需改写 SQL。

目前，Kyuubi 已经提供了 Session 级别的插件 API，允许管理员为每个会话注入不同的配置，可以实现 HBO 优化、将计算引擎路由到不同的集群等功能。在 2025 上半年，我们计划开放 Query 级别的插件 API，允许管理员为每个查询注入特定的配置、改写查询语句。

**02**

**Kyuubi****支撑多种 Spark 使用场景**

Spark 是一个灵活的计算引擎，提供多种 API，针对不同的业务场景，用户也有各种各样的使用姿势，接下来介绍一下 Kyuubi 作为 Spark 网关，最典型的几类使用场景。

首先是纯 SQL 场景，尤其是用户已有基于 Hive 的数仓业务，期望迁移到 Spark SQL 以降本增效。

Kyuubi 完全兼容 HiveServer2 的 thrift 通信协议，因此用户无需调整客户端，只需简单修改连接地址即可将 SQL 发送到 Kyuubi 执行；再加上 Spark SQL 高度兼容 Hive 语法，绝大部分 SQL 无需任何改动，少部分 SQL 做微调即可实现 Hive 到 Spark SQL 地平滑迁移。

相较于 Spark 内置的 Thrift Server，Kyuubi 支持 Spark Cluster 模式，将重度逻辑转移到引擎端 ，大幅增强了 Server 端的稳定性；同时 Kyuubi 提供多种引擎共享级别，如默认的 USER 模式，每个用户的所有查询复用一个 Spark App，避免了 ad-hoc 查询的冷启动时间。CONNECTION 模式更适合 ETL 调度作业，每个查询使用单独的 Spark App，确保作业之间不会互相干扰，最大程度提升稳定性。

在网易内部的某业务线实践中，使用 Kyuubi 将 ETL 作业从Hive 迁移到 Spark SQL，大幅降低了资源使用，同时提升了性能，获得综合 7 倍的收益。

第二个 Spark 比较典型的用户场景是 Notebook。

一些流行的发行版，比如 CDH，不提供 spark-sql 命令和 ThriftServer 组件。很多企业的基础设施团队，出于可管控、安全的角度，不希望向业务团队直接提供 Hadoop 客户端机器登录权限，仅通过平台向用户暴露提交 SQL / Jar / PySpark 批作业的接口，也就限制了用户直接使用 spark-sql、pyspark、sparkR 等原生命令的能力。

在网易内部，所有的 Spark 都是托管的，我们和网易云音乐部门合作，基于 Zeppelin 完成了与 Kyuubi 的对接，支持了交互式的 SQL、Scala、PySpark 模式。其中 SQL 功能已经推到了 Zeppelin 社区上游，详见 ZEPPELIN-6027，我们计划在 2025 上半年也将 PySpark 和 Scala 的集成推送到开源社区。

在企业中，Spark Jar 任务必然是一个绕不过的话题，Kyuubi 借鉴 Livy 的 batch 模式，在近期的版本中也支持了 Jar/Python 任务的 batch 作业调度。同时得益于 Kyuubi 使用 spark-submit 提交 Spark App，可以轻易地完成 Spark on YARN 到 Spark on K8s 到切换。

在网易内部的一个业务线中，我们使用 Kyuubi 替换掉了原有的 Livy 服务，帮助业务方实现从单一的 Spark on YARN 到 Spark on YARN/K8s 的混合调度，用更少的硬件支撑了日均十万级的 Spark 调度作业。

**03**

**Kyuubi****对接多样 Spark 部署形态**

无论是网易集团内部不同的业务线，还是网易数帆作为一个 B 端服务提供商对接的外部客户，其提供的基础设施差异总是巨大的，也对 Spark 的部署有不同的要求。比如即使大家都默认将云原生和 Kubernetes 划等号，各自的 Kubernetes 集群总是在存储、网络、调度器等方面存在巨大的差异。Kyuubi 作为一个通用的网关，必须要尽可能适应不同的 Spark 部署形态，尽量抹除底层的差异，为用户提供统一的接入方式。

首先一个常见的部署需求是多版本 Spark 共存，比如用户可能出于兼容性目的，不想升级 Spark 版本；管理员可能希望灰度测试 Spark 新版本。得益于 Kyuubi 的架构设计和完备的兼容性测试，多版本 Spark 支持是非常友好的，具体表现在：

* Kyuubi Server 与 Spark 完全解耦、进程隔离，仅通过拼接 spark-submit 命令提交 Spark App，因此可以通过覆盖 SPARK\_HOME 环境变量切换 Spark 客户端，并自动检测 Spark 客户端的 Scala 版本，使用正确 Scala 版本的 engine Jar 提交 Spark App。
* Spark Engine 使用反射等技术支持多个 Spark 版本，并且经过充分的集成测试确保跨版本 Spark 的二进制兼容性，使得一套 Kyuubi 集群同时支持多个 Spark 版本。

另一个常见的需求是 Spark on YARN / K8s 的混合部署，相信大部分 Spark on YARN 的用户往 Spark on K8s 的迁移不是一蹴而就的，会希望有一个平台可以灵活地控制将 Spark 作业提交到 YARN 或 K8s，实现渐进式的平滑迁移。

前面提到，Kyuubi 已经提供了 Session 级别的插件 API，允许管理员为每个会话注入不同的配置。我们预置了一个基于配置文件的内置实现，允许管理员将一组配置预置到一个 profile 配置文件中，用户只需指定此 profile 即可加载所有配置。如上的例子中，我们预置了两组配置，分别对应 yarn-hz 集群和 k8s-gy 集群，本质上即是一些 spark 相关的配置，这样用户仅需在提交作业时，指定 kyuubi.session.conf.profile 即可控制将此 Spark 作业提交到 YARN 或 K8s 集群。

同样地，借助上述的 profile 特性，我们也可以预置一些比如 gluten、java17 等配置文件，可以灰度地为一些 Spark 作业启用 Native Engine、Java 17 等较新的实验特性，提升 Spark 作业执行效率。

这是一个 Kyuubi 社区用户贡献的案例，他发现阿里云某个产品提供了一个全托管的 Spark 服务，如果能与 Kyuubi 结合，既可以省去 Spark/YARN/K8s 集群的维护成本，又能享受公有云极致弹性和按量付费的服务。

此托管服务开放的入口有限，仅提供了一个与 spark-submit 兼容的命令行工具，通过解压也不难看出，这应该仅是对 API 的一个封装，并非真正的 spark-submit。得益于 Kyuubi 使用 spark-submit 提交 Spark App，该用户轻易地就完成了对接，实现了公有云上按需付费的 Spark SQL 服务。

**04**

**Kyuubi****在 Spark 4 时代的基于与挑战**

Spark 4 正在如火如荼的开发中，Spark 4.0 预计将于 2025 年上半年发布，带来一系列的新特性，其中一项重磅特性即宣布 Spark Connect GA。

Spark Connect 是由 Databricks 主导开发的一项新特性，其全新的架构可能会改变未来 Spark 主流的使用方式。

Kyuubi 项目流行的一个重要因素是兼容 HiveServer2 通信协议，兼容 Hive 生态。Spark 4 提出了自己的 Connect 协议，我们也希望 Kyuubi 未来能兼容 Connect 协议，再次乘得 Spark 生态的东风。

简单而言，Spark Connect 是希望将所有以 DataFrame 为主的 Spark Public API 由方法调用变成 gRPC 远程调用的方式，将客户端与服务端分离，类似于将 Hive 迁移到 BeeLine 和 HiveServer2 的过程。

理想情况下，Spark Connect 可以将 Spark Jar / PySpark 任务也支持服务化，这会进一步简化 Spark 的上手成本，比如大幅简化 PySpark 用户配置本地环境和安装依赖步骤，用户只需安装一个 1MiB 多的客户端，直接连接远程的 Connect Server 即可运行 Spark 作业。

在具体实现细节上，Connect Server 与 Thrift Server 有很多异同之处，如果大家对 Thrift Server 有了解，通过对比可以快速了解 Connect Server。

首先两者的通信协议不同，Connect Server 选择了 gRPC 而非 Thrift，基于 gRPC 目前的生态，我相信这也是当下最正确的选择；在 API 数量上，Thrift 仅需考虑 SQL 场景，而 Connect 覆盖 DataFrame 几乎所有 API，API 和数据结构数量上 Connect 要远多与 Thrift；两者架构无本质却别，都是单点的 RPC Server，也仅支持 Client 模式。

Kyuubi 兼容 Thrift Server 的API，但架构创新取得了成功，这是否意味着 Kyuubi 的架构也能适用于 Connect Server，再次取得成功？

对此我们持乐观态度，并且已经实现了一个 PoC 版本，验证了此想法的可行性，欢迎有兴趣的同学到 GitHub 进一步交流。

**05**

**Q&A**

**Q1****：Kyuubi Server 服务如果崩溃，正在执行的 SQL 会失败吗？**

A1：如果使用 THRIFT\_BINARY 协议，即基于 TCP 长链接，socket 断开后会自动关闭会话并 cancel 所有正在执行的 SQL，可以通过配置 kyuubi.session.close.on.disconnect=true 来改变此行为；如果使用 RESTful Batch API 提交的作业，则不会受到影响。

**Q2****：Spark on K8s 场景下，Kyuubi Server 可以部署在 K8s 外部吗？如果可以，Kyuubi Server 如何与 Spark Driver 通信呢？**

A2：可以，只要外部网络与 K8s 内部网络互通即可。一般情况下，我们使用 Cluster 模式运行 Spark on K8s，Kyuubi Engine（即 Spark App）启动后，会将 Spark Driver Pod 的 IP 注册到 Zookeeper 或 ETCD 以供 Kyuubi Server 发现和连接。默认情况下，K8s 的 Service 和 Pod 运行在与宿主机不同的网段中，其具体实现取决于管理员安装的 CNI 插件， 常用的网络插件如 Calico 可以使用 BGP 协议将 K8s 网络平面与宿主机网络联通，即可正常通信；在实践中，我们也看到一些用户通过使用 Host 网络模式运行 Spark on K8s，直接规避了不同网络平面的通信的问题。这也印证了前面提及的观点：虽然大家都在向云原生靠拢，但对云原生有不同的理解，Spark 云原生部署依然面临着巨大的基础设施差异。

**以上就是本次分享的内容，谢谢大家。**

**分享嘉宾**

INTRODUCTION

****潘成****

****网易****

**软件工程师**

潘成，网易数帆软件工程师，Apache Celeborn/Kyuubi/Zeppelin PMC 成员。主要从事企业级离线计算引擎开发、Apache Kyuubi 开源社区建设等工作。

点个在看你最好看

SPRING HAS ARRIVED