---
title: Apache Kyuubi 1.8 特性解读
author: Apache Kyuubi
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511691&idx=1&sn=1b4f1bf0c1897db08ad382bef75d24da&chksm=ce294b20f95ec2364b9483195bb6459d363164cd97d20f150583cc482ba5c7f8790950461768&mpshare=1&scene=24&srcid=1115NeA0JCP44BaTvI0UNVZa&sharer_shareinfo=fe465755701861d008a697e95de3a5df&sharer_shareinfo_first=fe465755701861d008a697e95de3a5df#rd
---

关注公众号，后台回复【1.8】即可获得本文 PPT 文件

“

本文来自于 Apache Kyuubi PMC Member 潘成的分享。内容围绕 Apache Kyuubi 最新版本 1.8 的特性与改进，主要分为以下部分：

1. 场景扩展 —— 在线分析，离线跑批

2. 流式增强 —— 流批一体，面向未来

3. 企业特性 —— 行业沉淀，持续打磨

4. 开源社区 —— 开放包容，合作共赢

”

**01**

**Apache Kyuubi 简介**

Apache Kyuubi 是一个构建在 Spark、Flink、Trino 等计算引擎之上的，分布式、多租户的**企业级大数据网关**，致力于在 Lakehouse 之上提供 Serverless SQL 服务。

**Kyuubi 支持多种类型的工作负载。**典型的使用场景包括：用户可以使用 JDBC/BeeLine 以及各种 BI 工具，连接 Kyuubi 进行交互式的数据分析；使用 RESRful API 向 Kyuubi 提交 SQL/Python/Scala/Jar 批作业。

Kyuubi 是一个分布式、多租户的企业级大数据网关，Kerberos 作为 Hadoop 生态组件的事实认证标准，Kyuubi 对其做了深度适配，同时 Kyuubi 也支持企业常用的 LDAP 安全认证协议。特别的，Kyuubi 支持同时启用 Kerberos 和 LDAP 认证，客户端可以选择任一种认证方式接入 Kyuubi。

除了作为网关的主体功能外，Kyuubi 还提供一系列可以独立使用 Spark 插件，可提供如小文件治理、Z-Order、SQL 血缘提取、限制查询数据扫描量等企业级功能。

**02**

****场景扩展 —— 在线分析，离线跑批****

使用 Kyuubi Thrift API 搭配 Spark 计算引擎，取代 HiveServer2 或者Spark Thrift Server 是 Kyuubi 最早专注的场景，也是目前最成熟、生产案例最多的使用方式。

Kyuubi Server 是一个轻量级的常驻服务，不参与计算，负责按需拉起计算引擎，并通过 内部RPC 协议与其通信，以 Spark 引擎为例，Kyuubi Spark Engine 即一个 Spark Application。

在交互式会话中，Kyuubi 最重要的两个概念是 Session 和 Operation，分别与 JDBC 中的 Connection 和 Statement，以及 Spark 中 SparkSession 和 QueryExecution 概念对应。

如上是一段使用 JDBC 驱动连接 Kyuubi 执行 SQL 查询的代码，可以清晰地看到客户端 JDBC 调用与 Spark 引擎侧之间的对应关系；特别的，客户端通过 JDBC ResultSet 拉取结果集时，会以小批次的方式从 Spark Driver 经过 Kyuubi Server 回传，确保 Kyuubi Server 不会有较大的内存压力；在 1.7 版本中，Kyuubi 支持了基本 Apache Arrow 的结果集序列化方式，大幅提升了大结果集场景的传输效率，该特性在 1.8 中得到了进一步改进，**优化了与结果集增量拉取组合使用时的性能。**

在企业级的 Spark 应用中，Jar/PySpark 任务是不可回避的场景。自 1.6 版本起，在 eBay 的主导下，Kyuubi 引入了批会话（Batch Session）的概念，支持了 Jar 任务的提交，同时在实现上复用了交互式会话中的 Session、Operation 概念；**在 1.8 版本中，通过引入了基于数据库的多级队列，批任务并发调度能力得到显著增强。**

在批会话中，一个 Spark Application 即对一个一个 Batch Session，当用户发起提交任务请求时，Kyuubi Server 创建一个 Batch Session 并将其放置到 Backend 线程池队列中，此处即将线程池作为一个简易的内存队列，控制提交并发；当 Backend 线程池接收该任务后，调用 spark-submit 提交，并通过 YARN/K8s API 轮训 Spark Application 状态直至作业结束。

批任务会话目前仅提供 RESTful API（以及基于此 API 的 kyuubi-ctl 命令行工具），与交互式会话中常用的基于 TCP 长链接的 Thrift-Binary 协议不同，RESTful API 是基于 HTTP 短链接的协议，在典型的高可用部署中，多台 Kyuubi 服务实例部署在 Load Balancer 之后，客户端发起的请求则可能会被转发到不同的 Kyuubi 服务实例，因此直接类比交互式会话，将 Session、Operation 状态保存在内存里是不可行的行为。

Kyuubi 通过引入 Server 间的请求转发和数据库持久化存储状态解决了上述 HA 场景下的问题。数据库的引入同时支持了 Kyuubi Server 停机重启后的 Batch Session 状态恢复，是滚动升级的必要条件。

在初版的批会话（后续简称 Batch V1）实现中，我们发现一些可以改进的空间：

1. spark-submit会瞬时（持续约30s）消耗大量 CPU 和 IO 资源，突发的高并发任务提交将对 Kyuubi Server 服务所在机器或容器有极高的资源要求；

2. 高并发提交场景下，当单台 Kyuubi Server 出现资源瓶颈时，水平扩容无法快速转移负载到新增节点上；

3. spark-submit 与后续 Application 状态轮训共享线程，spark-submit 仅在初始提交的数十秒内有较大的资源消耗，后续 Application 状态轮训资源消耗极低；使用同一个 Backend 线程池控制线程数从而控制 Batch Session的并发，无法做到精细化的资源控制：较小的线程数造成单台 Kyuubi 节点承接的 Batch 作业并发度较低；较大的线程数在面临瞬间多个 spark-submit 同时执行时可能发生资源耗尽；

4. 无法做到全局 FIFO 调度和全局优先级调度；

在 1.8 中，我们在 Batch V1 的基础上，针对上述问题做了改进，提出Batch V2 设计，其核心变化是，在原有 Backend 线程池队列的基础上，新增两个队列：

* **基于数据库的队列**：用户向 Kyuubi Server 发起 Batch 任务提交后，入队数据库队列即返回；
* **Submitter 线程池队列**：每个 Kyuubi Server 有一个 Submitter 线程池，每个线程专门负责从数据库队列中拾取任务，交由 Backend 线程池做 spark-submit 提交，该线程在 Spark Application 进入 RUNNING 或者退出状态后释放以继续处理数据库队列中的其他任务，从而实现对 spark-submit 进程并发控制；
* 原有的 **Backend 线程池队列**：得益于前置 Submitter 线程池对 spark-submit 进程并发度的控制，此线程池可以设置较大的线程数，用于获取对 Spark Application状态的拉取。

在 Batch V2 模式下，Kyuubi Server 的稳定性大幅提升，尤其在批量调度 ETL 作业场景下，可以从容地应对瞬间大量批任务的提交；当 Kyuubi 提交 Spark 作业的速度跟不上造成数据库队列堆积时，扩容 Kyuubi 集群后，新加入的 Kyuubi 节点启动后立即承接负载；同时，得益于数据库队列的引入，**Batch V2 支持全局****作业 FIFO 和优先级调度。**

在落地 Spark on K8s 场景中，我们还发现频繁地通过查询 Spark Driver Pod 状态来获取 Spark Application 状态会对 API Server 造成巨大压力，随后我们改进方案，通过使用 Informer在 Kyuubi Server 侧维护一份Spark Driver Pod 状态列表而避免频繁的向 API Server 发起查询请求，显著地降低 API Server 的压力。

Kyuubi Batch RESTful API 提供了一个 REST SDK 供用户以编程的方式集成使用；同时也提供了一个 kyuubi-ctl 命令行工具，可以配合 yaml 文件提交和管理 Batch 作业。如上是一个使用 kyuubi-ctl 向 Kyuubi 提交 SparkPi 批作业的案例。

**03**

****流式增强 —— 流批一体，面向未来****

使用 Kyuubi Thrift API 搭配

架构上，Kyuubi 由 Server 和 Engine 两个组件构成。其中 **Server 是一个轻量级常驻服务**，而 Engine 是按需启停的计算引擎。客户端发起连接后，Kyuubi Server 会根据路由规则寻找合适的 Engine，若没有命中，则会主动拉起一个新的计算引擎，当计算引擎闲置一段时段后，会主动退出释放资源。

对于交互会话，Kyuubi创造性地提出引擎共享级别，内置的四个共享级别：CONNECTION、USER、GROUP、SERVER隔离性依次增强，共享程度依次降低，搭配使用可以满足多种负载场景。

Kyuubi 的设计上是天然适配多计算引擎的，我们可以相对容易的扩展接入其他引擎。Flink 是当前最成熟的流式计算引擎，Kyuubi 早在 1.4 就实现了第一版的 Flink Engine，并持续改进，**在最新的 1.8 版本中，来自网易互娱的贡献者对 Flink Engine 做了重大重构，功能得到了显著增强。**

**在 1.8 中，Kyuubi 新增支持了 Flink YARN Application Mode，**该模式与 Spark Cluster 模式类似，将 SQL 解析、执行计划生成等工作从 Kyuubi Server 节点的 Flink Client 进程转移到了 Application Master 中，从而将所有计算资源纳入 YARN 管控，大幅降低了 Kyuubi Server 节点的负载；搭配 Kyuubi 的引擎共享级别使用，可以获得极致的弹性。

Kyuubi 始终遵循 Upstream First 的原则，紧跟 Flink 社区最新动向，**Kyuubi 1.8适配了 Flink 1.16～1.18 最近三个版本。**在具体实现上，Kyuubi 最大程度复用了 Flink SQL Gateway 模块的代码。

此外我们还对 Flink 时间类型做了更完整的覆盖，利好 Flink CDC 场景的用户；通过改进 Kyuubi 的结果集传输框架，**支持了拉取 Flink 无边界流结果集**。

**04**

****企业特性 —— 行业沉淀，持续打磨****

使用 Kyuubi Thrift API 搭配

基于历史原因，Spark 内置的 Hive 连接器没有被实现为一个标准的 Data Source 组件，高度耦合的实现也使得 Spark 无法轻易连接多个 Hive 数仓做联邦查询和跨集群写入。

来自百度的开发者向 Kyuubi 社区贡献了基于 Spark DataSource V2 API 的 Hive 连接器，该连接器可以实现 Spark 对多个 Hive、HDFS 集群的联邦查询、写入。在Kyuubi 1.8 中得到了一系列的改进和错误修复，包括对 Hadoop Delegation Token 的支持、对 Hive Serde 分区表的写入错误修复等。

安全是企业级应用中一个十分重要的话题，同时 Kerberos 是 Hadoop 生态认证机制的事实标准；Kyuubi 作为一个企业级大数据网关，Kyuubi 在 Kerberos 支持上做了深度适配，支持 Kerberos/LDAP 同时启用，客户端可以选择任何一种方式认证；支持 Hadoop 用户代理机制，在保证安全的同时，省去海量用户 keytab 的管理；支持 Hadoop Delegation Token 续期，满足 Spark 常驻任务的认证需求等。

Kyuubi 不乏一些来自一些金融券商和欧美企业的社区用户和贡献者，他们对安全有着更为极致的要求，比如服务组件间的内部通信也需要加密，支持权限控制和 SQL 审计等，Kyuubi 对此类场景亦可胜任。

**Kyuubi 同时也简化了客户端侧的 Kerberos 认证过程**；对于非技术开发人员，接入启用 Kerberos 认证的服务流程繁琐，学习 Kerberos 和 Hadoop 认证机制有陡峭的门槛；例如，用户若期望使用 Hive JDBC驱动，在 Windows 系统上使用 DBeaver 连接启用 Kerberos 认证的 Kyuubi 服务，需要如上图左侧所示的 5 个步骤；而使用 Kyuubi JDBC 驱动，则简化为 2 个步骤，只需配置 krb5.ini，并且在 JDBC URL 中指定客户端 principal 与 keytab，以及服务端 principal 即可，大幅降低了用户学习和接入成本。

**在 Kyuubi 1.8 中，来自 Cisco 的开发者向 Kyuubi 贡献了全新的 Web UI，可以满足对 Engine、Session、Operation 资源的查看和管理。**

**05**

****开源社区 —— 开放包容，合作共赢****

使用 Kyuubi Thrift API 搭配

Kyuubi 项目自诞生以来已经走过了 5 年，经历了网易自主开源、Apache 孵化阶段，社区不断壮大，最终在 2023 年正式毕业，成为了 Apache 顶级项目。

社区始终秉持 Upstream First 的原则，紧跟开源社区上游生态，及时跟进上游组件新版本。**在****Kyuubi 1.8 中新增了对 Java 17、Scala 2.13的适配，并对 Java 21做了初步的验证；主体功能以及所有插件完整兼容 Spark 3.1～3.4，初步兼容 Spark 3.5；基于 Flink 1.16～1.18 做了充分验证；支持 Hive 最新版本 3.1。**

Kyuubi 已被多个公有云、海内外多个行业的头部企业所采纳作为 SQL 网关，我们也看到越来越多的用户在 AWS、GCP、Azure 上 独立部署 Kyuubi 使用。

**如果你也在使用Kyuubi，或者正在调研 Kyuubi，欢迎前往 GitHub 页面分享你的用户场景和案例～**

**https://github.com/apache/kyuubi/discussions/925**

Apache Kyuubi 是一个社区驱动的开源项目。Kyuubi 核心开发者团队是多元化的，他们来自海内外的各个行业的企业，在 Apache Way 的指引下，彼此友好协作，致力于将 Kyuubi 打造成可持续的优秀开源项目。

**如果你也对参与 Kyuubi 开发感兴趣，欢迎参加2023 Kyuubi 开发者活动，与优秀的开发者一起成长！**

**https://github.com/apache/kyuubi/issues/5357**

# 现场问答

**Q**：我们使用共享的 Kyuubi Engine 为用户提供查询服务，在使用过程中，我们发现一些用户的大查询会影响 Engine 的稳定性，对其他查询造成影响甚至导致 Engine OOM，针对这种场景，有什么解决方案吗？

**A**：在用户分享的实践案例中，有两种常用的解决方案：

1. **HBO，**即 History-based Optimization。构建独立的服务，收集并分析任务的特征，对于周期性的调度任务，在任务执行前，基于历史执行信息做出判断，将大查询的会话切换到 CONNECTION 隔离级别，使用独立的计算资源，降低对其他作业的影响。

2. **将选择暴露给用户。**例如在 Bilibili 的分享中提到，在面向用户的 SQL 查询工作台中，给出一个“大查询”的选项，对于有一定经验的数据分析师，可以提前预料到自己的查询可能会对 Engine 稳定性造成影响，则主动使用独立的计算资源以规避。

********END********

**Apache Kyuubi****推特账号****现已开通**

**推特搜索****Apache Kyuubi****或 浏览器****打开下方链接****即可关注~**

**https://twitter.com/KyuubiApache**

**还可以加入****Apache Kyuubi Slack**

**https://join.slack.com/t/apachekyuubi/shared\_invite/zt-1e1qw68g4-yE5HJsVVDin~ABtZISyuxg**

**和海外开发者交流互动哦~**

**最后**

**Kyuubi 在这里提醒大家**

**文明上网 科学上网**

**往期精彩:**

[Kyuubi 1.7 特性解读之高性能 Arrow 结果集传输](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247508948&idx=1&sn=77b90516c0382a2b54dbcec14b4651a5&chksm=ce29467ff95ecf696b916e6b927912f4c7482fc9fd7f032a8272c9fe044a8915829bf46a0572&scene=21#wechat_redirect)

[Kyuubi Code Contribution Program 2023 正式开启](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511533&idx=1&sn=d617dcbeaf129e73e2b7039ff1a4e41f&chksm=ce294846f95ec150dd1723261dfc91ccd5fe0340ef0a0b06ea94e47a72ef7887f6afe5acc31a&scene=21#wechat_redirect)

[基于 Kyuubi 实现分布式 Flink SQL 网关](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511565&idx=1&sn=9bed5dc04b20f30fb42dec9faf2da940&chksm=ce294ba6f95ec2b0c620e5b519ad1ab1b8c97d61df443617b46250e91ebdacce0013af50291e&scene=21#wechat_redirect)

[Apache Kyuubi on CDH 在竞技世界大数据平台实践](https://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247497852&idx=1&sn=8facac617e9c101c653beafdd58a79bb&scene=21#wechat_redirect)

[亲测3分钟！带你从零配置 Kyuubi 查询 Doris](https://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247492762&idx=1&sn=04820177211bef2290d226756fab3d6d&scene=21#wechat_redirect)

看到这里记得多多点赞、评论、收藏

还可以把 Kyuubi 分享给更多朋友~

点击"阅读原文"可跳转 Kyuubi 开发者活动页面