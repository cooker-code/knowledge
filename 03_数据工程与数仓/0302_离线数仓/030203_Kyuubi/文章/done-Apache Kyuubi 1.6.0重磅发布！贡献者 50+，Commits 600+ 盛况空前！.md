---
title: Apache Kyuubi 1.6.0重磅发布！贡献者 50+，Commits 600+ 盛况空前！
author: Flink
date:
url: http://mp.weixin.qq.com/s?__biz=MzI0NTIxNzE1Ng==&mid=2651226118&idx=1&sn=4bbd5fee871f0d751b6ea3c95bdb7200&chksm=f2a3c2edc5d44bfbde19fa1eccbd2ad74b3eb2a8408a8e2b3e8c5562e20150a9d70842387a50&mpshare=1&scene=24&srcid=0908EfrCGVCNkjyWH2kYLaY0&sharer_sharetime=1662643005264&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030203_Kyuubi/030203_版本记录|版本记录]]


**01 版本发布**

2022年09月06日，Apache Kyuubi (Incubating) 社区发布了进入Apache 孵化器之后的第 6 个公开发行的正式版本——**1.6.0-incubating**。非常感谢 Apache Kyuubi (Incubating)  开源社区成员对 Kyuubi 1.6.0-incubating 版本发布做出的贡献。

 

在这个版本中，我们更新了：

**服务端**

* 支持批（Jar）任务提交
* 支持 Thrift HTTP 模式
* 支持 Etcd 作为发现服务
* 支持 JDBC 认证机制
* 统一 Thrift 和 REST v1 API 的认证
* 增强指标系统

**客户端**

* 新增批（Jar）任务提交的 REST 客户端
* 内置 JDBC 驱动程序剥离 Hive 和 Hadoop 依赖
* 内置 JDBC 驱动程序支持使用 keytab 进行 Kerberos 身份验证
* 增强 beeline 以支持 Spark 控制台进度条

**引  擎**

* 使用最新 Spark 3.1/3.2/3.3 充分验证
* 使用最新 Flink 1.14/1.15 充分验证
* 使用 Trino 363 全面验证
* 支持 Hive 后端引擎 (Beta)
* 支持 JDBC 后端引擎 (Beta)

**插  件**

* 新增 Spark TPC-DS 连接器
* 新增 Spark TPC-H 连接器
* 新增 Spark Authz (Apache Ranger) 插件

**文  档**

* 用户文档重构和完善

**详情请见 Apache Kyuubi 官方网站：**

**https://kyuubi.apache.org/zh/release/1.6.0-incubating.html**

**02 社区参与**

Apache Kyuubi (Incubating) 项目依靠社区发展，我们致力于在 Apache Way 的指导下，为用户提供简单易用的大数据产品，我们强调社区协作，互相帮助，共同成长。

首先，如果您在下载和使用 Apache Kyuubi (Incubating) 1.6.0-incubating 中发现任何问题，欢迎使用 Github Issues 功能，将您遇到的问题和社区分享。

https://github.com/apache/incubator-kyuubi/issues

如果您或者您的公司正在使用 Apache Kyuubi(Incubating) ，并乐意与社区和分享，可以在 **Who is using Apache Kyuubi (Incubating) ?**中进行留言。https://github.com/apache/incubator-kyuubi/discussions/925

我们也接受其他任何形式的帮助，详见 https://github.com/apache/incubator-kyuubi#contributing，欢迎通过 **dev-subscribe@kyuubi.apache.org** 订阅我们的邮件列表，获取社区最新动向。

**03 致谢**

在社区驱动的模式下，Apache Kyuubi (Incubating)  得以有今天的发展态势，真诚地感谢每一位项目导师、社区贡献者及用户的信任、支持和帮助。

特别感谢对 1.6.0-incubating 版本有直接贡献的社区小伙伴 ：

**04 项目简介**

Apache Kyuubi （Incubating）是一个 Thrift JDBC/ODBC 服务，目前对接了 Apache Spark/Flink 计算框架以及 Trino(Presto) 查询引擎，支持多租户和分布式等特性，可以满足企业内诸如 ETL、BI 报表等多种大数据场景的应用。Kyuubi 可以为企业级数据湖探索提供标准化的接口，赋予用户调动整个数据湖生态的数据的能力，使得用户能够像处理普通数据一样处理大数据。它的主要方向是依托本身的架构设计，围绕各类主流计算框架，打造一个面向 Serverless SQL on Lakehouse 的服务。Kyuubi 已在国内外数十家企业落地应用，2021年度入选中国科协“科创中国”开源创新榜单。

**社区资源**

* 官方网站：https://kyuubi.apache.org/
* 代码仓库：https://github.com/apache/incubator-kyuubi
* 邮件列表：https://kyuubi.apache.org/mailing\_lists.html
* 问题记录：https://kyuubi.apache.org/issue\_tracking.html
* 成为提交者：[Become A Committer of Apache Kyuubi](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247487924&idx=1&sn=40e868e4deabd789a03ef4d0196f7ce9&chksm=ce2af41ff95d7d096b086eb4949a2fe543356b727da817355e58ca48347c74be12fa03cfc225&scene=21#wechat_redirect)
* # Who is using Apache Kyuubi (Incubating)：https://github.com/apache/incubator-kyuubi/discussions/925

**往期精彩:**

[Apache Kyuubi 官网更新，这个地方需要你的支持](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247492200&idx=1&sn=c60b3c63719c949f79af02aad7a7b44c&chksm=ce2907c3f95e8ed506c085ed2b7a657805a7931652cf2607daf72005f90777107ebe430a9492&scene=21#wechat_redirect)

[Kyuubi 文档贡献活动上线！手把手带你领取周边礼品](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247492584&idx=1&sn=94435cae0037007331fed1786a3c7009&chksm=ce290643f95e8f55ead9190fd770a87bce48453d43ddadc3ed5f91989fe9654472bd9507ef2f&scene=21#wechat_redirect)

[社区的用户老铁们！Apache Kyuubi 叫你来领周边！](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247492364&idx=1&sn=84a4b95d4f6a6d382e5228c79cdda1bc&chksm=ce2906a7f95e8fb1c022023f4f8b65c3beecbc156543e9a443fd8173f7d05015cb16b5677213&scene=21#wechat_redirect)

[Apache Hudi X Apache Kyuubi 中国移动云湖仓一体的探索与实践](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247492644&idx=1&sn=afec463f87fbf76e26a5f5688726c5ee&chksm=ce29018ff95e8899e270e5ba1c958667cf3a2c3cc49515ec3af93f060e4ffaba76fd9f9f9264&scene=21#wechat_redirect)

[Become A Committer of Apache Kyuubi](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247487924&idx=1&sn=40e868e4deabd789a03ef4d0196f7ce9&chksm=ce2af41ff95d7d096b086eb4949a2fe543356b727da817355e58ca48347c74be12fa03cfc225&scene=21#wechat_redirect)

看到这里记得多多点赞、评论、收藏

还可以把 Kyuubi 分享给更多朋友~