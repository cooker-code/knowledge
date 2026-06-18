> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030203_Kyuubi/030203_核心知识点/Kyuubi企业级SQL网关1.8边界|Kyuubi企业级SQL网关1.8边界]]
---
title: 坦白局！网易数帆解读 Apache Kyuubi 1.8 特性
author: DataFunSummit
date:
url: http://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247591501&idx=1&sn=4ac48cce509dd9d859b0bd838a58a7e9&chksm=c10d3003f67ab91576f3a0b526de2e6d52f4362337372d5704c98c4f2271a4f47ef712d14e04&mpshare=1&scene=24&srcid=1226mH3AEkP7IH062PElvrTC&sharer_shareinfo=9bdaa7cdcc062d11f9ffca0059a76218&sharer_shareinfo_first=9bdaa7cdcc062d11f9ffca0059a76218#rd
---

**导读** 本文主要介绍 Apache Kyuubi 最新版本 1.8 的特性与改进。

******本次分享主要分为以下部分：******

1. Apache Kyuubi 简介

2. 场景扩展 —— 在线分析，离线跑批

3. 流式增强 —— 流批一体，面向未来

4. 企业特性 —— 行业沉淀，持续打磨

5. 开源社区 —— 开放包容，合作共赢

6. 问答环节

分享嘉宾｜潘成 网易数帆 大数据技术专家

内容校对｜李瑶

出品社区｜DataFun

---

**01**

# **Apache Kyuubi 简介**

Apache
Kyuubi 是一个构建在 Spark、Flink、Trino 等计算引擎之上的，分布式、多租户的企业级大数据网关，致力于在 Lakehouse 之上提供 Serverless SQL 服务。

Kyuubi 支持多种类型的工作负载。典型的使用场景包括：使用 JDBC/BeeLine 以及各种 BI 工具，连接 Kyuubi 进行交互式的数据分析；使用 RESTful API 向 Kyuubi 提交 SQL/Python/Scala/Jar 批作业。

Kyuubi 是一个分布式、多租户的企业级大数据网关，Kerberos 作为 Hadoop 生态组件的事实认证标准，Kyuubi 对其做了深度适配，同时 Kyuubi 也支持企业常用的 LDAP 安全认证协议。特别的，Kyuubi 支持同时启用 Kerberos 和 LDAP 认证，客户端可以选择任一种认证方式接入 Kyuubi。

除了作为网关的主体功能外，Kyuubi 还提供一系列可以独立使用 Spark 插件，可以提供如小文件治理、Z-Order、SQL 血缘提取、限制查询数据扫描量等企业级功能。

# **02**

# **场景扩展 —— 在线分析，离线跑批**

使用 Kyuubi Thrift API 搭配 Spark 计算引擎，取代 HiveServer2 或者 Spark Thrift Server 是 Kyuubi 最早专注的场景，也是目前最成熟、生产案例最多的使用方式。

Kyuubi
Server 是一个轻量级的常驻服务，不参与计算，负责按需拉起计算引擎，并通过
内部 RPC 协议与其通信，以 Spark 引擎为例，Kyuubi Spark Engine 即一个 Spark Application。

在交互式会话中，Kyuubi 最重要的两个概念是 Session 和 Operation，分别与 JDBC 中的 Connection 和 Statement，以及 Spark 中 SparkSession 和 QueryExecution 概念对应。

如上是一段使用 JDBC 驱动连接 Kyuubi 执行 SQL 查询的代码，可以清晰地看到客户端 JDBC 调用与 Spark 引擎侧之间的对应关系；特别的，客户端通过 JDBC ResultSet 拉取结果集时，会以小批次的方式从 Spark Driver 经过 Kyuubi Server 回传，确保 Kyuubi Server 不会有较大的内存压力；在 1.7 版本中，Kyuubi 支持了基于 Apache Arrow 的结果集序列化方式，大幅提升了大结果集场景的传输效率，该特性在 1.8 中得到了进一步改进，优化了与结果集增量拉取组合使用时的性能。

在企业级的 Spark 应用中，Jar/PySpark 任务是不可回避的场景。自 1.6 版本起，在 eBay 的主导下，Kyuubi 引入了批会话（Batch Session）的概念，支持了 Jar 任务的提交，同时在实现上复用了交互式会话中的 Session、Operation 概念；在 1.8 版本中，通过引入基于数据库的多级队列，批任务并发调度能力得到显著增强。

在批会话中，一个 Spark Application 即对应一个 Batch Session，当用户发起提交任务请求时，Kyuubi Server 创建一个 Batch Session 并将其放置到 Backend 线程池队列中，此处即将线程池作为一个简易的内存队列，控制提交并发；当Backend 线程池接收该任务后，调用 spark-submit 提交，并通过 YARN/K8s API 轮训 Spark Application 状态直至作业结束。

批任务会话目前仅提供 RESTful API（以及基于此 API 的 kyuubi-ctl 命令行工具），与交互式会话中常用的基于 TCP 长链接的 Thrift-Binary 协议不同，RESTful API 是基于 HTTP 短链接的协议，在典型的高可用部署中，多台 Kyuubi 服务实例部署在 Load Balancer 之后，客户端发起的请求则可能会被转发到不同的 Kyuubi 服务实例，因此直接类比交互式会话，将 Session、Operation 状态保存在内存里是不可行的行为。Kyuubi 通过引入 Server 间的请求转发和数据库持久化存储状态解决了上述 HA 场景下的问题。数据库的引入同时支持了 Kyuubi Server 停机重启后的 Batch Session 状态恢复，是滚动升级的必要条件。

在初版的批会话（后续简称 Batch V1）实现中，我们发现一些可以改进的空间：

* spark-submit 会瞬时（持续～30s）消耗大量 CPU 和 IO 资源，突发的高并发任务提交将对 Kyuubi Server 服务所在机器或容器有极高的资源要求；
* 高并发提交场景下，当单台 Kyuubi Server 出现资源瓶颈时，水平扩容无法快速转移负载到新增节点上；
* spark-submit 与后续 Application 状态轮训共享线程，spark-submit 仅在初始提交的数十秒内有较大的资源消耗，后续 Application 状态轮训资源消耗极低；使用同一个 Backend 线程池控制线程数从而控制 Batch Session 的并发，无法做到精细化的资源控制：较小的线程数造成单台 Kyuubi 节点承接的 Batch 作业并发度较低；较大的线程数在面临瞬间多个 spark-submit 同时执行时可能发生资源耗尽；
* 无法做到全局 FIFO 调度和全局优先级调度。

在 1.8 中，我们在 Batch V1 的基础上，针对上述问题做了改进，提出了 Batch V2 设计，其核心变化是，在原有 Backend 线程池队列的基础上，新增两个队列：

* 基于数据库的队列：用户向 Kyuubi Server 发起 Batch 任务提交后，入队数据库队列即返回；
* Submitter 线程池队列：每个 Kyuubi Server 有一个 Submitter 线程池，每个线程专门负责从数据库队列中拾取任务，交由 Backend 线程池做 spark-submit 提交，该线程在 Spark
  Application 进入 RUNNING 或者退出状态后释放以继续处理数据库队列中的其他任务，从而实现对 spark-submit 进程并发控制；
* 原有的 Backend 线程池队列：得益于前置 Submitter 线程池对 spark-submit 进程并发度的控制，此线程池可以设置较大的线程数，用于获取对 Spark Application 状态的拉取。

在 Batch V2 模式下，Kyuubi Server 的稳定性大幅提升，尤其在批量调度 ETL 作业场景下，可以从容地应对瞬间大量的批任务提交；当 Kyuubi 提交 Spark 作业的速度跟不上造成数据库队列堆积时，通过扩容 Kyuubi 集群，新加入的 Kyuubi 节点会在启动后立即承接负载；同时，得益于数据库队列的引入，Batch V2 支持全局作业 FIFO 和优先级调度。

在落地 Spark on K8s 场景中，我们还发现频繁地通过查询 Spark Driver Pod 状态来获取 Spark Application 状态会对 API Server 造成巨大压力，随后我们改进方案，通过使用 Informer 在 Kyuubi Server 侧维护一份 Spark Driver Pod 状态列表来避免频繁的向 API Server 发起查询请求，显著地降低了 API Server 的压力。

Kyuubi
Batch RESTful API 提供了一个 REST SDK 供用户以编程的方式集成使用；同时也提供了一个 kyuubi-ctl 命令行工具，可以配合 yaml 文件提交和管理 Batch 作业。上图是一个使用 kyuubi-ctl 向 Kyuubi 提交 SparkPi 批作业的案例。

# **03**

# **流式增强 —— 流批一体，面向未来**

架构上，Kyuubi 由 Server 和 Engine 两个组件构成。其中 Server 是一个轻量级常驻服务，而 Engine 是按需启停的计算引擎。客户端发起连接后，Kyuubi Server 会根据路由规则寻找合适的 Engine，若没有命中，则会主动拉起一个新的计算引擎，当计算引擎闲置一段时间后，会主动退出释放资源。

对于交互会话，Kyuubi 创造性地提出引擎共享级别，内置的四个共享级别：CONNECTION、USER、GROUP、SERVER 隔离性依次增强，共享程度依次降低，搭配使用可以满足多种负载场景。

Kyuubi 设计上是天然适配多计算引擎的，我们可以相对容易的扩展接入其他引擎。Flink 是当前最成熟的流式计算引擎，Kyuubi 早在 1.4 就实现了第一版的 Flink Engine，并持续改进，在最新的 1.8 版本中，来自网易互娱的贡献者对 Flink Engine 做了重大重构，功能得到了显著增强。

在 1.8 中，来自网易互娱的贡献者为 Kyuubi 新增支持了 Flink YARN Application Mode，该模式与 Spark
Cluster 模式类似，将 SQL 解析、执行计划生成等工作从Kyuubi Server 节点的 Flink Client 进程转移到了 Application Master 中，从而将所有计算资源纳入 YARN 管控，大幅降低了 Kyuubi Server 节点的负载；搭配 Kyuubi 的引擎共享级别使用，可以获得极致的弹性。

Kyuubi 始终遵循 Upstream First 的原则，紧跟 Flink 社区最新动向，Kyuubi 1.8适配了 Flink 1.16～1.18 最近三个版本。在具体实现上，Kyuubi 最大程度复用了 Flink SQL Gateway 模块的代码。

此外我们还对 Flink 时间类型做了更完整的覆盖，利好 Flink CDC 场景的用户；通过改进 Kyuubi 的结果集传输框架，支持了拉取 Flink 无边界流结果集。

# **04**

# **企业特性 —— 行业沉淀，持续打磨**

基于历史原因，Spark 内置的 Hive 连接器没有被实现为一个标准的 Data Source 组件，高度耦合的实现也使得 Spark 无法轻易连接多个 Hive 数仓做联邦查询和跨集群写入。

来自百度的开发者向 Kyuubi 社区贡献了基于 Spark DataSource V2 API 的 Hive 连接器，该连接器可以实现 Spark 对多个 Hive、HDFS 集群的联邦查询、写入。在Kyuubi 1.8 中得到了一系列的改进和错误修复，包括对 Hadoop Delegation
Token 的支持、对 Hive Serde 分区表的写入错误修复等。

安全是企业级应用中一个十分重要的话题，同时 Kerberos 是 Hadoop 生态认证机制的事实标准；作为一个企业级大数据网关，Kyuubi 在 Kerberos 支持上做了深度适配，支持 Kerberos/LDAP 同时启用，客户端可以选择任何一种方式认证；支持 Hadoop 用户代理机制，在保证安全的同时，省去海量用户 keytab 的管理；支持 Hadoop Delegation Token 续期，满足 Spark 常驻任务的认证需求等。

Kyuubi 不乏一些来自金融券商和欧美企业的社区用户和贡献者，他们对安全有着更为极致的要求，比如服务组件间的内部通信也需要加密，支持权限控制和 SQL 审计等，Kyuubi 对此类场景亦可胜任。Kyuubi 1.8 中，来自广发证券的贡献者显著增强了 Authz 插件对 Iceberg 等数据湖的支持，这是对湖仓一体安全性上的一个重要补充。

Kyuubi 同时也简化了客户端侧的 Kerberos 认证过程。对于非技术开发人员，接入启用 Kerberos 认证的服务流程繁琐，学习 Kerberos 和 Hadoop 认证机制有陡峭的门槛。例如，用户若期望使用 Hive JDBC 驱动，在 Windows 系统上使用 DBeaver 连接启用 Kerberos 认证的 Kyuubi 服务，需要如上图左侧所示的 5 个步骤。而使用 Kyuubi JDBC 驱动，则简化为两步，进去配置 krb5.ini，并且在 JDBC URL 中指定客户端 principal 与 keytab，以及服务端 principal 即可，大幅降低了用户学习和接入成本。

在 Kyuubi 1.8 中，来自 Cisco 的开发者向 Kyuubi 贡献了全新的 Web UI，可以满足对 Engine、Session、Operation 资源的查看和管理。

# **05**

# **开源社区 —— 开放包容，合作共赢**

Kyuubi 项目自诞生以来已经走过了 5 年，经历了网易自主开源、Apache 孵化阶段，社区不断壮大，最终在 2023 年正式毕业，成为了 Apache 顶级项目。

Kyuubi 社区始终秉持 Upstream First 的原则，紧跟开源社区上游生态，及时跟进上游组件新版本。Kyuubi 1.8 中新增了对 Java 17、Scala 2.13 的适配，并对 Java 21 做了初步的验证；主体功能以及所有插件完整兼容 Spark 3.1～3.4，初步兼容 Spark 3.5；基于 Flink 1.16～1.18 做了充分验证；支持 Hive 最新版本 3.1.

Kyuubi 目前被多个公有云、海内外多个行业的头部企业采纳作为 SQL 网关，我们也看到越来越多的用户在 AWS、GCP、Azure 上独立部署 Kyuubi 使用。

如果你也在使用 Kyuubi，或者正在调研 Kyuubi，欢迎扫码到 GitHub 页面分享你的用户场景和案例～

Apache
Kyuubi 是一个社区驱动的开源项目。Kyuubi 核心开发者团队是多元化的，他们是来自海内外的各个行业的企业，在 Apache
Way 的指引下，彼此友好协作，致力于将 Kyuubi 打造成可持续的优秀开源项目。

如果你也对参与 Kyuubi 开发感兴趣，欢迎参与 Kyuubi 贡献，与优秀的开发者一起成长！

# **06**

# **问答环节**

**Q：我们使用共享的 Kyuubi Engine 为用户提供查询服务，在使用过程中发现一些用户的大查询会影响 Engine 的稳定性，对其他查询造成影响甚至导致 Engine OOM，针对这种场景，有什么解决方案吗？**

A：在用户分享的实践案例中，有两种常用的解决方案：

* HBO，即 History-based Optimization。构建独立的服务，收集并分析任务的特征，对于周期性的调度任务，在任务执行前，基于历史执行信息做出判断，将大查询的会话切换到 CONNECTION 隔离级别，使用独立的计算资源，降低对其他作业的影响。
* 将选择暴露给用户。例如在 Bilibili 的分享中提到，在面向用户的 SQL 查询工作台中，给出一个“大查询”的选项，对于有一定经验的数据分析师，可以提前预料到自己的查询可能会对 Engine 稳定性造成影响，则主动使用独立的计算资源以规避。

**以上就是本次分享的内容，谢谢大家。**

**分享嘉宾**

INTRODUCTION

********潘成********

****网易数帆****

**大数据技术专家**

Apache Kyuubi PMC Member，Apache Celeborn (Incubating) PPMC Member。

往期优质文章推荐

往期推荐

[美团 Doris Bitmap 精确去重优化实践](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247591482&idx=1&sn=bacec2a9cc7c744f6336876230c08d0b&chksm=c10d3074f67ab96271f0e88f758deb14a1b0879ee7966f53b6987711c4ad6ba77439b4e89f9c&scene=21#wechat_redirect)

[华为 当LLM的优势与推荐系统结合后~](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247591423&idx=2&sn=a7c15544390fff2a3c314a032ad86a1f&chksm=c10d33b1f67abaa7e42b6236fbd9103cecae89678f4dc13814b68628fab654760d6df4d86e22&scene=21#wechat_redirect)

[小米如何用数据智能驱动业务增长](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247591310&idx=1&sn=1356d4391ecac77edc5146bc04343df0&chksm=c10d33c0f67abad675ab1018ad48d95653ea344b5122d9003d82bef7aef24be7ea783d89f08c&scene=21#wechat_redirect)

[当电信网络运营遇见知识图谱构建](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247591155&idx=2&sn=04ff67b32682cf8f8de7f572fbe956cd&chksm=c10d32bdf67abbab1c7520f814341b1ee84bd5ccd3620bb71379bfe7dba7a2df35fca8048554&scene=21#wechat_redirect)

[B 站基于 StarRocks 构建大数据元仓和诊断系统](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247590648&idx=1&sn=f4fe6967154f9919bb633e93a6b42cfd&chksm=c10d34b6f67abda0279cfb2d056062762c58d60aec1e9ba04b617fe02e44d8393cd0369ec726&scene=21#wechat_redirect)

[深度强化学习的风吹到了电网](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247590477&idx=1&sn=00692b0adff9facbfa88459d6828c05f&chksm=c10d3403f67abd150ae5462c3e75567edfa04560a6644bf0d5ccc4c2e882df84df088bea1986&scene=21#wechat_redirect)

[两个范式搞定因果与机器学习的前沿初探](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247590474&idx=1&sn=9db1a608d5a883d65ba712af82887390&chksm=c10d3404f67abd122c3343cddac037e024250ed2ade0813e1d592b94e0634fc53bc1bbb9fb65&scene=21#wechat_redirect)

[去冗降本—Doris 高并发实时查询核心技术](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247590307&idx=1&sn=0906ce74d39fa54992b2409c2322d8b2&chksm=c10d37edf67abefb5d16a67253e10792b60e09c15bab34425f8ab7633abce5ec7803ad1c7088&scene=21#wechat_redirect)

[虎牙平台数据驱动业务实践，破局在即！](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247590053&idx=1&sn=71b2b577e78036f373e373279ba1e3b6&chksm=c10d36ebf67abffd1962c1d1b54413ebdf45dae47eee4f7ae110d51a310fad8bf6a07a4df8e2&scene=21#wechat_redirect)

[字节如何打造新用户增长场景下的AB实验体系](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247589968&idx=2&sn=2d6b3dcd8c0b949d12f4e35b9cc56b95&chksm=c10d361ef67abf08dd376a835e34e7b88b14b735ee574779cafec2d8148bb50774b284d094c8&scene=21#wechat_redirect)

[以新能源资产为主体的能源运营决策](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247589919&idx=1&sn=2a1bd9326dedfcf0810c2f3062185ccd&chksm=c10d3651f67abf47e6e282808d6eb4fbc8d5e796e14f0136622c164cc789690d39a95b4e572d&scene=21#wechat_redirect)

[快手指标体系的管理驾驶舱场景应用实践](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247589260&idx=1&sn=234ff03404d9d01cea06c2506b94c6a4&chksm=c10dcbc2f67a42d4593fc9666f564e32553fedb8a84e7f57c0d3d81b1bcfabc41ce0f0d4452a&scene=21#wechat_redirect)

[基于Alink模型流的在线学习](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247589181&idx=1&sn=446d39ea5dbb9a2f5b2893261f58c65c&chksm=c10dcb73f67a4265d720d1ddd2300737807da042b7f3e1f436fc486f54f8de9df8f2a7723e2c&scene=21#wechat_redirect)

[增长的底层逻辑和新增长三大案例](https://mp.weixin.qq.com/s?__biz=MzkxMjM2MDIyNQ==&mid=2247588974&idx=1&sn=0ef4d0fd148f0f7b6674653852d50d45&chksm=c10dca20f67a4336827c2b8949ebbc101b42e820a2a6fd0e355c93441817dae03825724e48a5&scene=21#wechat_redirect)

DataFun

点个在看你最好看