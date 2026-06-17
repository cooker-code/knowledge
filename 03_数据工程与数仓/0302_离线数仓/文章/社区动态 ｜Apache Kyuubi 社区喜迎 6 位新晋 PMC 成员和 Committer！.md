---
title: 社区动态 ｜Apache Kyuubi 社区喜迎 6 位新晋 PMC 成员和 Committer！
author: Apache Kyuubi
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247510753&idx=1&sn=9a972db5f05e58e3e5669eaba698b767&chksm=ce294f4af95ec65cfe80a59b506955210987dae8891c423e64e5c4bf8a5cbebb0b7d27d04484&mpshare=1&scene=24&srcid=0602i3RDzbj0FViYznBmHjQU&sharer_sharetime=1685658282618&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

近期，通过 Apache Kyuubi 项目管理委员会的推荐与投票，Apache Kyuubi 社区正式迎来了 5 位新晋 Committer 和 1 位 PMC 成员的加入！感谢大家的加入，使得社区的发展后劲十足！

小编分别采访了这 6 位同学，讲述他们与 Apache Kyuubi 的故事。本篇给大家介绍的是来自百度的伊凯飞、广发证券的梁博文以及来自小米的张耀东。（根据时间先后顺序）。

New Committer

伊凯飞（Kaifei Yi）

GitHub 地址：https://github.com/Yikf

目前就职于百度基础架构部，多年来一直从事大数据相关产品的设计与研发工作，先后涉及分布式计算引擎、云原生数仓等领域，主要关注引擎性能优化、湖仓一体等方向。

**01 在 Apache Kyuubi 主要负责什么？**

我主要参与 Apache Kyuubi 的 Spark Engine Hive Connector 模块以及 Trino frontend 模块的开发，同时参与了大量的功能优化和 Bugfix 工作。

**02 加入 Apache Kyuubi 的契机是什么？**

其实我算是 Apache Kyuubi 的业余贡献者，同时也是开源爱好者，一次偶然的机会了解到 Kyuubi 这个项目，被它 Serverless SQL on LakeHouse 的理念深深吸引，经过对整体框架的深入了解，发现它的系统架构、多引擎融合设计、接口设计、代码细节、文档描述都特别优秀，优秀的项目很难让人不爱上它。

此外，本身之前负责过 Apache Spark Auth Plugin 相关的模块，这与 Kyuubi Ranger AuthZ 模块深度契合，于是便有了更进一步的了解。

作为开源爱好者，我在开源领域学习到了很多优秀项目、优秀前辈的经验与精神，这深深影响着我，本着对 Apache Kyuubi 的热爱和 取诸开源，反哺开源 的原则，我参与了 Apache Kyuubi，并见证着 Kyuubi 的毕业，这是一个很令人难忘的经历。

**03 加入社区后有什么体验和收获？**

加入kyuubi 成为 Committer 最大的收获是这段经历，这些美好的经历反哺着我个人的成长。

参与 Kyuubi 社区开源工作的过程中，社区的小伙伴总会及时的 review 提交的 PR 并给出合适的建议，这也促进着我们工程能力的提升。

Kyuubi 社区是一个非常友好的社区，加入社区后，我认识到了一群社区的技术大牛，与他们讨论技术方案，在交流的过程中，也可以学习到他们解决问题的思路、想法和逻辑思维，这些优秀的技术思维无疑是一笔巨大的财富。

**04 能给想参与社区做贡献的小伙伴们一点建议吗？**

Apache Kyuubi 社区是一个非常有活力的社区，对新人非常友好，新同学可以从 README 开始，这通常涵盖了项目背景、系统架构、技术特性等技术相关的介绍，通过 README 可以对 Kyuubi 有个相对全面的认识。可以通过邮件列表、交流群的方式参与社区沟通，了解社区方向。善用官网和文档，了解 Kyuubi 项目的编码规范，在 Issue 中寻找自己感兴趣的点进行切入。

**05 寄语**

首先恭喜 Kyuubi 顺利从 Apache 孵化器毕业成为 Apache 顶级项目。Kyuubi 的开源给了我们一个更广阔的平台，期待能有更多的小伙伴加入 Kyuubi，一起促进社区的发展，也衷心祝愿 Apache Kyuubi 能够在开源舞台中书写新的篇章。

、

New Committer

梁博文（Bowen Liang）

GitHub 地址：https://github.com/bowenliang123

目前就职于广发证券大数据团队，参与负责大数据平台架构工作。曾任职于阿里巴巴及星飞游戏，在大数据、深度学习、中台服务、DevOps、支付安全、运营分析等领域及场景有一线经验。

当前正努力构建具有广发特色的数字化数据中台，秉持“积极评估、审慎落地、持续演进”的思路方法，以自身数据战略为目标建设一体化数据服务体系，以大数据平台为基座，以及万级规模的数据加工作业，提供批处理调度、实时流式链路能力，并配以综合数据治理，构建完整的数据体系。高效统一进行数据采集、数据建模、数据加工、数据输出、数据服务、数据治理等。

**01 在 Apache Kyuubi 主要负责什么？**

我在 Apache Kyuubi 中主要参与了 Ranger 鉴权 Authz 插件、JDBC 认证等关键安全特性的工作，提供最精细粒度的行过滤列权限及脱敏能力，并参与了 Serverless PySpark 等跨越性新质算力的部分开发。同时，我参与开发了 REST API 中的 Batch 上传 Java 特性，令传统 Spark 作业的演进成为可能。另外，近期我也参与完成了对接 ChatGPT 的对接开发来响应 AIGC 时代的新机遇。最近也在社区的帮助下担当 v1.7.1 的发版人如期完成发版。

**02 加入 Apache Kyuubi 的契机是什么？**

对金融企业而言，我们尤其关注到了 Apache Kyuubi 为大数据赋能提供了至关重要的两点，Serverless 服务网关能力以及集中安全管控。

我和广发证券一直立足于自身大数据目标持续评估各种业内技术和开源项目带来的可行性，为平台转型提供机遇。其中在 2021 年-2022 年持续重点关注到正在孵化中的 Apache Kyuubi 项目，认为其基于数据湖的 Serverless SQL 能够为大数据平台升级并完成数据服务层带来巨大的现实价值和潜力，决心作为新一代大数据平台旗舰服务，从 2022 年 8 月起进行重点投入和持续参与，同时为社区在计算、鉴权、审计等领域带来持续社区贡献。

**03 加入社区后有什么体验和收获？**

在公司层面，我们广发证券在 2022 年开始分 4 个阶段逐步落地，取得持续良好的实际效益，查询效益直接触达每一个系统用户，读写作业对已有的 HiveSQL 数据作业迁移提升明显，运行效率提升 100%，同时在权限控制、资源动态调度、技术演进上有了全面飞跃，令 Spark3.x、Iceberg、动态资源管控、传统作业迁移成为可能。

在个人层面，我对 Apache Kyuubi 各位 PMC 的感激之情无以言表。在参与 Kyuubi 社区开发前我的 Github 已经在空白中沉睡了十几年，而现在已经完成了超过 160 次 commit 和上万行的变更，得幸与他们事无巨细的提点、建议、设计重构下。同时在面对兼容 Spark3.x 各个版本变化和 Ranger 版本下，大大拓宽了我在大数据领域的体系化思考能力和具体代码实现能力，填补了自身的空白。

但即使如此，在当下仍然有越来越多未知和我无法解决的问题，未来希望能继续向各位PMC、Committer、Contributor 持续探讨并共同学习。

**04 能给想参与社区做贡献的小伙伴们一点建议吗？**

事实上，我也曾羞涩于提出自己的想法、建议和实现，甚至 Scala 最基本语法我也无法写好。但，让自己的理解和想法成为现实的有效途径就是去参与他。当你想要参与开源项目的这一刻，你就拥有了乐观、自信、爱。

乐观于面对数据领域的千变万化，自信于你的一点一滴让愿景成为可能，爱则是来自于对大数据技术机理和未来的热爱。Apache Kyuubi 在架构上可以提供各种的前端协议和各种后端的灵活组合，同时既有功能已经经过反复的集成测试和场景时间爱你。你的想法可以在稳定架构和成熟项目机制下，落地成为下一代大数据技术新面貌锦上添花。来一起提出issue、PR、文档、测试、周边吧。

**05 寄语**

希望 Apache Kyuubi 社区持续关注和适配更多技术和特性，与各个引擎生态和场景相互赋能成为可能，为所有关注 Apache Kyuubi 的用户和参与者提供更广阔的空间和效益。祝愿 Apache Kyuubi 长足进步和持久发展。

New Committer

张耀东（Yaodong Zhang）

GitHub 地址：https://github.com/iodone

小米研发工程师，目前专注于 OLAP 方向，熟悉 Kyuubi、Trino 等开源项目

**01 在 Apache Kyuubi 主要负责什么？**

主要参与了 Kyuubi Spark Lineage 字段级血缘解析、Trino-JDBC 协议、EventBus 模块开发和相关缺陷修复和优化工作。

**02 加入 Apache Kyuubi 的契机是什么？**

在 2020 年就开始关注到 Kyuubi 这个项目，因为当初公司内部有类似的 SQL 代理服务，基于Spark Thrift Server 独立改造过来的，功能点与 Kyuubi 的目标基本一致。在公司重新打造统一的大数据体系的推动下，构建统一的 SQL 入口服务势在必行。调研了 Kyuubi 之后完全满足我们当时的需求，Kyuubi 与 STS 和 HS2 的完全兼容一致、高可用和资源隔离兼备，加上清晰简洁的架构以及社区打造可测试、可维护、可扩展统一 SQL Gateway 的先进理念，我们从之前的架构上迁移过来非常丝滑，从开发到上线新架构两周时间就完成了平滑迁移。

目前 Kyuubi Server 已经承载了我司业务日均 100 万的 SQL 请求量，可用性仍保持 99.9% 以上，非常稳定。随着业务需要，我们在 Kyuubi 上逐渐增加了一些定制化的能力，同时也反馈到社区将一些通用场景下的能力提交至 Kyuubi，在内部业务和开源社区形成一种良好的促进和协作关系。

**03 加入社区后有什么体验和收获？**

有幸能够成为 Kyuubi 社区的 Committer，非常感谢社区小伙伴们的耐心指导，切实感受到了 Kyuubi 社区的贡献者温暖和自我严格要求，力争在打造一个高质量的开源社区。代码可测、注重细节和优雅实现是体会最为深刻的，需要严格要求自己去努力达到，同时社区伙伴会给出非常详细的 Review 建议和指导意见，帮助我去达成这些。Kyuubi 目前是一个很开放的新社区，在这里任何相关的问题都可以及时得到反馈和解答，能有一个这样的平台与志同道合的小伙伴们一起交流，本身也是一件非常幸福的事情。

**04 能给想参与社区做贡献的小伙伴们一点建议吗？**

Kyuubi 是一个非常优秀的大数据领域的开源项目，它是偏实用型的且开箱即用，累积了很多大数据架构真实场景的解决方案，如果是初建大数据体系，大家一定不要错过 Kyuubi。其优雅的架构实现和可扩展设计模式非常值得我们去学习和借鉴，尤其是较为完备的测试体系，建议大数据相关的小伙伴都可以参与进来，从怎么写单测来入手，逐步加入到 Kyuubi 社区建设中。另外 Kyuubi 是使用 Scala 语言开发的，喜欢 Scala 的小伙伴一定要看一看，Scala 可以帮助我们进行高效优雅的工程化实现落地。

**05 寄语**

未来 Apache Kyuubi 一定会成为大数据的 SQL Gateway 标杆项目，期望更多的小伙伴都能参与进来一起构建 Apache Kyuubi 社区和大数据生态体系。

(未完待续......)

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

[网易 Spark Kyuubi 核心架构设计与源码实现剖析](http://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247510044&idx=1&sn=87bcb554d471ed7a53cc73960a396d72&chksm=ce294db7f95ec4a16e9dfee3cbeb96705ee2082646eaa010fae8440fb8164974345929352eeb&scene=21#wechat_redirect)

[Kyuub‍i 在小米大数据平台的应用实践](https://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247492435&idx=1&sn=fea08603b867f526e0b5dbf9e6fb2482&scene=21#wechat_redirect)

[Apache Kyuubi on CDH 在竞技世界大数据平台实践](https://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247497852&idx=1&sn=8facac617e9c101c653beafdd58a79bb&scene=21#wechat_redirect)

[亲测3分钟！带你从零配置 Kyuubi 查询 Doris](https://mp.weixin.qq.com/s?__biz=Mzg2MDA5MTU4OA==&mid=2247492762&idx=1&sn=04820177211bef2290d226756fab3d6d&scene=21#wechat_redirect)

看到这里记得多多点赞、评论、收藏

还可以把 Kyuubi 分享给更多朋友~

点击"阅读原文"即可跳转到官网链接~