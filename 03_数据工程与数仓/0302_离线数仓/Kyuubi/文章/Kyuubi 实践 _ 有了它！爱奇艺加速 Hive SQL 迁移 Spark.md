---
title: Kyuubi 实践 | 有了它！爱奇艺加速 Hive SQL 迁移 Spark
author: HBase技术社区
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU5OTQ1MDEzMA==&mid=2247490925&idx=1&sn=b61ccf4d25a099dd5663952c8578777e&chksm=feb5ea10c9c26306bb93d8fa1a3b1d1b630e31cd9dc1a729072e0af241449be22dfae8509670&mpshare=1&scene=24&srcid=0628BoasahuPAh1JTtjroIer&sharer_sharetime=1656422827077&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

“

Hive 作为爱奇艺数仓的基础，Hive SQL 是爱奇艺大数据平台目前主要的数据处理工具，各个业务积累大量的 Hive ETL 任务。Spark 相对于 MapReduce 有着更为灵活的的计算模型，这使得 Spark 相对于
Hive (on MapReduce) 有更好的性能。 

经过测试对比，我们发现迁移 Hive SQL 到 Spark 将会带来很大的性能提升和资源节省。

Apache Kyuubi (Incubating) 项目提供一个分布式多租户的 Spark Thrift Server，相对于 Spark 原生的
Spark Thrift Server 有更好的架构优势和更多优秀的特性，具体对比可参考：Kyuubi v.s. Spark Thrift
JDBC/ODBC Server (STS)。

”

**0****1**

****Hive****S****QL**** ****迁移 Spark****

**1.2****TPCDS 数据集**

**1.1****双****跑****对比**

大数据平台中已有大量稳定运行的 Hive SQL 任务，为了在迁移的过程中提高用户迁移意愿，降低与用户的沟通成本，我们需要保证迁移后 Spark SQL 的稳定性以及数据准确性，并尽量减少用户操作。

我们在大数据平台中新增了 Hive SQL 迁移服务，提供一键批量双跑测试和一键迁移功能。在迁移前， 我们会对业务的 Hive SQL 任务进行批量双跑测试。改写业务 SQL 将输出替换成临时库表，并分别通过 Hive SQL 和 Spark SQL 执行。

执行完成后，对 Hive SQL 和 Spark SQL 的执行结果进行校验，确保迁移后数据一致。对每行数据进行 concat 后计算 crc32 并求和得到 checksum 值，通过对比两个结果表的 checksum 值和 count 数，来确定数据是否一致。

> select sum(cast(CRC32(concat(\*)) as decimal(19,0))) as checksum, count(\*) as count
>
> from mock\_db.mock\_table

通过双跑测试，我们能够提前发现 Hive SQL 与 Spark SQL 的一些兼容性问题，以及配置不合理导致任 务失败的问题。及时优化平台配置，指导用户优化 SQL 并为任务提供合适的配置。

**1.2****兼容性适配**

Hive SQL 迁移到 Spark 的过程中，我们遇到了一些语法兼容性的问题，例如：

> * Spark 删除不存在分区报错，而 Hive 中允许删除不存在分区
> * Spark 执行 set mapreduce.job.reduces=-1 报错
> * 数字类型与字符串比较，结果与 Hive 不一致
> * .....

为了加快迁移的进度，减少用户参与，我们需要兼容部分 Hive 语法。

在 Kyuubi 中提供 kyuubi-extension-spark 模块，维护Spark SQL的 Extensions，用于优化Spark SQL的执行计划，社区已经提供了很多优化，详细文档：Auxiliary
SQL extension for Spark SQL。我们基于此模块，为不兼容的语法修改其执行计划，使得与 Hive 保持一致的语义。 

例如，在 Hive 中由于默认的 hive.exec.drop.ignorenonexistent=true 配置，在删除不存在的表或分区时不会报错，但是迁移到 Spark 运行后，出现大量删除不存在分区的错误。我们在Kyuubi中实现了 DropIgnoreNonexistent 优化，改写 AlterTableDropPartitionCommand/DropTableCommand 等命令的执行计划，将 ifExists 属性设置为 true ，从而在删除不存在分区时不报错。

Kyuubi提供了只解释SQL执行计划的运行模式，通过kyuubi.operation.plan.only.mode配置，可以以 PARSE/ANALYZE/OPTIMIZE/PHYSICAL/EXECUTION 的一些模式解释 SQL 的执行计划而不执行它，这样我们可以快速扫描出有语法兼容问题的一些 SQL。

**1.3****小文件优化**

Hive SQL 迁移 Spark 运行不可避免地会出现小文件的问题。我们使用 Kyuubi 的 Spark Extensions 结合 Spark AQE 可以有效地控制小文件问题。

Kyuubi Spark Extensions 中提供了 RepartitionBeforeWrite 的优化，在写入前插入 repartition 操作，再结合 AQE 合并小分区，从而实现控制小文件。

对于写入动态分区的情况，Kyuubi 中 Repartition 操作指定了动态分区字段 ，这样可以减小动态分区导致的小文件。如果动态分区数据分布不均匀，可能导致部分分区数据倾斜，所以对于动态分区写入 Kyuubi 在 Repartition 时，同时加入了一个随机数字段来避免数据倾斜。不过这样由于扩展了分区字段 的基数，也可能会导致小文件问题。

在 spark-defaults.conf 中添加相关的配置：

对于 Spark3.2，Kyuubi 通过在写入前插入 Rebalance 算子，更好地解决小文件的问题。

**0****2**

****Kyuubi 服务增强****

**2.1****标签化配置**

对于不同用户和业务的 SQL，由于数据量和业务处理逻辑存在差异，可能需要对不同的任务进行单独的 配置优化。我们在数据开发平台上提供了用户配置参数的能力，允许用户对 SQL 任务添加配置。

对于同一用户或者业务会存在一些具有相同的特效的一些 SQL，我们提供标签化配置，定义一些标签绑定特有的配置，并在任务提交时带上相关的标签。

例如，我们定义的部分标签如下：

> + **Adhoc**：用于即席查询平台任务，数据量小，要求快速响应。配置 USER 共享级别的引擎，较大 Driver 内存，以及 Spark 任务抢占策略等。
> + **Batch**：用于数据开发平台任务，定时调度运行，稳定要求较高。配置 CONNECTION 级别独立引 擎，使得任务完全资源隔离，并添加小文件、AQE 等优化配置。
> + **User/Business/Custom**：允许定义用户、业务或其他自定义标签，绑定特有的一些配置，如：队列、资源、兼容性适配相关配置等，降低用户使用门槛。

Kyuubi 中已经支持了对用户添加默认配置，例如在配置文件中添加 "\_\_\_bob\_\_\_.spark.executor.memory=8g" 配置，为 bob 用户的任务默认指定 8G 的 executor 内存，具体使用参考：User Default。

同时，在Kyuubi 的 kyuubi-server-plugin 中，提供了 SessionConfAdvisor 接口，用于为 Session 注入配置。可以将标签配置保存在数据库中，并实现 SessionConfAdvisor 接口，根据 Session 配置 的标签从数据库中获取对应的配置。

**2.2****SQL 审计**

Kyuubi 服务中定义了一些事件，可用于 SQL 审计操作。

Kyuubi Server 中定义了如下事件：

> + **KyuubiServerInfoEvent**：Kyuubi Server 启动、停止事件信息
> + **KyuubiSessionEvent**：Session 开启、关闭事件，以及连接相关信息
> + **KyuubiOperationEvent**：Kyuubi Operation 执行事件，包括了 SQL 执行相关信息

目前 Kyuubi 仅实现了 JSON 写入本地文件的方式采集事件，相关配置：

将事件写入 JSON 文件后，可以借助日志收集插件采集到 Kakfa 中并落地到 ElasticSearch 中，方便后续 对 SQL 执行事件进行审计分析。

我们在 Kyuubi Server 中通过实现自定义的 EventHandler ，处理 Kyuubi Server 的事件，直接将事件 写入到 ElasticSearch 中。分析失败 SQL 执行事件的错误信息，完善 Spark SQL 运维知识库，并添加正则匹配分析规则，匹配执行事件的错误信息给出对应的解决方案，方便用户自行排障。

**文中文档可参考：**

* **Kyuubi
  v.s. Spark Thrift JDBC/ODBC Server (STS)** ——https://kyuubi.apache.org/docs/latest/overview/kyuubi\_vs\_thriftserver.html
* **Auxiliary
  SQL extension for Spark SQL**—— https://kyuubi.apache.org/docs/latest/sql/rules.html#
* **DropIgnoreNonexistent**——https://github.com/apache/incubator-kyuubi/blob/master/extensions/spark/kyuubi-extension-spark-3-1/src/main/scala/org/apache/kyuubi/sql/DropIgnoreNonexistent.scala
* **User
  Defaults**—— https://kyuubi.apache.org/docs/latest/deployment/settings.html#user-defaults

‍

********END********

**Apache Kyuubi** **推特账号****现已开通**

**推特搜索****Apache Kyuubi****或 浏览器** **打开下方链接**

**即可关注~**

**https://twitter.com/KyuubiApache**

**还可以加入****Apache Kyuubi Slack**

**https://join.slack.com/t/apachekyuubi/shared\_invite/zt-1bhswm1n6-n~0wMbkvhsp0WZX0FZXvPA**

**和海外开发者交流互动哦~**

**最后**

**Kyuubi 在这里提醒大家**

**文明上网 科学上网**

**往期精彩**

[**· 如何优化 Spark 小文件，Kyuubi 一步搞定！**](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247491506&idx=1&sn=ec070e4d00fd3cc91d95b5e579a05d99&chksm=ce2afa19f95d730fbef721003505bcf06e03eeece8e6a4a5421b7acaf27de0279c0e94843764&scene=21#wechat_redirect)

**·**[**Kyuubi on Kubernetes 模式从 0 到 1 部署**](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247488092&idx=1&sn=24b40dc0457c09b83518b1d2ed08ab22&chksm=ce2af7f7f95d7ee1f55a47808ad87905383913e76ef2fa886eb149cbce00b05c6d8d4fb14b5a&scene=21#wechat_redirect)

[**·**](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247488088&idx=1&sn=6f6104175b70d5edcd7c98784be5b27f&chksm=ce2af7f3f95d7ee51b521a631ce6b3d681e010d2267f97e9960ec2eca4ac9a1a8e488bacf984&scene=21#wechat_redirect)[**Become A Committer of Apache Kyuubi**](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247487924&idx=1&sn=40e868e4deabd789a03ef4d0196f7ce9&chksm=ce2af41ff95d7d096b086eb4949a2fe543356b727da817355e58ca48347c74be12fa03cfc225&scene=21#wechat_redirect)

丨记得点关注

看到这里记得多多点赞、评论、收藏

还可以把 Kyuubi 分享给更多朋友~