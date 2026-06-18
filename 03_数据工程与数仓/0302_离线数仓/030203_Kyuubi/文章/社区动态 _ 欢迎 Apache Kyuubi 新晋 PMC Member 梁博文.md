---
title: 社区动态 | 欢迎 Apache Kyuubi 新晋 PMC Member 梁博文
author: Apache Kyuubi
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511857&idx=1&sn=080c77524be623c37391d849925ecb7e&chksm=ce294a9af95ec38c1002f8f1e0bdfba8d657aa9f41fc92c95e28470aa496544bfd99549e10f6&mpshare=1&scene=24&srcid=1226FLh4HFn2oNJgpTE3Jnun&sharer_shareinfo=1e5ee8958a582836484d74952c4ae706&sharer_shareinfo_first=1e5ee8958a582836484d74952c4ae706#rd
---

Apache Kyuubi 社区再添一位 PMC 成员 —— 广发证券的大数据平台架构师梁博文。

在 Kyuubi 社区其实能常见到这位新晋 PMC 成员的身影，新版本贡献名单、干货文章作者、meetup 讲师等等，两年来他持续助力着 Kyuubi 的发展，接下来让我们一起再次了解下他吧。

**# 新晋 PMC Member** 

**梁博文 （Bowen Liang）  
Github: bowenliang123  
Apache Kyuubi PMC Member  
广发证券大数据平台架构师**

**# 主要贡献**

* **Authz 鉴权插件**

为 Spark 生态实现对接 Ranger 统一精细化行列数据权限控制策略，适配了 DataSource V2 的命令，以开放式可插拔执行计划解析，兼顾 Hive 内表、Iceberg 数据湖等。满足 SQL、Jar、PySpark 等场景权限控制需要，并充分验证适配 Spark 3.0-3.5 所有版本，并为兼容更多新兴数据湖格式提供可能。

* **JDBC 认证插件**

支持通过 JDBC 方式完成用户身份认证，灵活组合 JDBC 数据源作为认证方式并快速按场景需要投射可控的用户管控能力。以 SQL 方式灵活定义鉴权细节，并兼顾异构 JDBC 数据源具有良好的适用性，同时 JDBC 易于实现可与 Hive、Trino 等快速联动使用统一认证方式。

* **热刷新用户配置**

支持对 User defaults 用户配置支持热刷新，避免对 Server 服务进行重启造成服务中断，敏捷因应场景需要，调整用户后续作业所用的运行资源配额、调优、引擎细节等。

* **新增上传 Jar 方式提交 Batch 作业**

对 REST 提交 Batch 作业接口进行扩展，支持通过 POST form-data 方式上传 Jar 包来提交 Spark 的 Batch 作业，以此完成一站式作业迁移，同时落实了认证、鉴权等的统一管理。

* **适配 Scala 2.13**

对 Server 和所有引擎及插件（除 Flink），在充分保持兼顾 Scala 2.12 为基线的前提下，兼容适配了 Scala 2.13 对其上下游可能涉及的问题补充了针对性的改造适配及验证适配。

* **Kafka 日志输出**

在 Server 端实现了 Kafka 日志插件，支持输出 Server 端日志到 Kafka 2.X 及 3.x。

* **Serverless Python 用户签名**

Kyuubi 支持以 JDBC 连接的 Serverless 方式执行 Python 及 Scala，新增用户签名，解决脚本在运行时篡改身份信息的问题，满足权限控制的需要。

* **其他持续稳定性优化**

增加 Java 8 的 bytecode 检查强化 JVM 兼容性，改造 TRowSet 结果集生成的通用抽象提升性能，参与修正 Spark 等引擎的运行时，更新依赖包版本及解决部分构建问题。

**# 入选感受**

**梁博文：**非常荣幸能够成为 Apache Kyuubi 的 PMC 成员，回顾过去两年中持续的参与，感谢 Kyuubi 社区对我和广发证券的认可和支持。

以“正向构建”的方式驱动和实践大数据生态能力，广发证券和 Kyuubi 项目在这一点上不谋而合。过去三年从技术跟踪到演进落地，Apache Kyuubi 成功在广发证券落地和持续实践，并成为新一代大数据中台的平台级旗舰服务，目前已超万个任务并迅猛增长。这是所有广发证券大数据智能团队全体系全要素全角色的共同实践，也藉此机会特别感谢广发证券领导同事，尤其是刘宇、区健灵（GitHub: @jeanlyn）及大数据智能团队的所有同仁的帮助。

一年半前我个人的 Scala 能力还处于原始起步阶段，开源项目参与经历也几乎为零。这一路从零开始的代码贡献，多亏 Kyuubi 社区的 Committer、Contributor 事无巨细地对我进行指导和鞭策。在这里，从最小的换行与命名，到设计建议与代码逻辑，再到多层次跨体系的运用和实践，我的大部分大数据实践都是各位一点一滴的帮助下进行的。

是大家的帮助才让共同的愿景能发生变成价值交付，我也希望能够持续以社区参与者的身份传递这份传统。

****◆  ◆  ◆  ◆  ◆****

最后再次欢迎 Apache Kyuubi 新晋 PMC Member 梁博文。也期待越来越多的企业和开发者能够积极拥抱开源、贡献开源。

Apache Kyuubi 将持续秉承 Apache Way，本着开放、友好的文化，欢迎更多开发者加入。

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

[深度剖析！广发证券 Apache Kyuubi 构建“提效可控”大数据赋能层](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511806&idx=1&sn=2267ae27c2c2003709c7119853bb54c2&chksm=ce294b55f95ec24392a66a56e10b8ea1ede1a46956459ec8a65b21bef72a111603ba4ce47223&scene=21#wechat_redirect)

[Apache Kyuubi 1.8 特性解读](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511691&idx=1&sn=1b4f1bf0c1897db08ad382bef75d24da&chksm=ce294b20f95ec2364b9483195bb6459d363164cd97d20f150583cc482ba5c7f8790950461768&scene=21#wechat_redirect)

[Apache Spark Native Engine](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511630&idx=1&sn=67055fc3989367e54a4155aad6acf95b&chksm=ce294be5f95ec2f3d22224745b224b7d78a5d94f0aaca013480ee3c7b7a40bc9ebcade39d16d&scene=21#wechat_redirect)

[基于 Kyuubi 实现分布式 Flink SQL 网关](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511565&idx=1&sn=9bed5dc04b20f30fb42dec9faf2da940&chksm=ce294ba6f95ec2b0c620e5b519ad1ab1b8c97d61df443617b46250e91ebdacce0013af50291e&scene=21#wechat_redirect)

[Apache Kyuubi & Celeborn（Incubating）助力 Spark 拥抱云原生](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247511319&idx=1&sn=d263a27cd7f47ad536f7a061a7c41075&chksm=ce2948bcf95ec1aadd7e471c5c54756313b1ece713176c7efdc6ca6fffd9c4632a252dcf1214&scene=21#wechat_redirect)

看到这里记得多多点赞、评论、收藏

还可以把 Kyuubi 分享给更多朋友~

点击"阅读原文"即可跳转官网链接~