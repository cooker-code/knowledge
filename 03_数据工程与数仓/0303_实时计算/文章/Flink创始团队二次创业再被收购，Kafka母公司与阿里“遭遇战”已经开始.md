---
title: Flink创始团队二次创业再被收购，Kafka母公司与阿里“遭遇战”已经开始
author: InfoQ
date: 
url: http://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651153962&idx=1&sn=b7b42ba4d77c288ecbb08d692efad625&chksm=bdb89e798acf176f4afa46525fdc9c394b68d4adbf3356b6c88a17fa9762d87851862374cdc5&mpshare=1&scene=24&srcid=0109Sfhgs5qMRAK3gfij4xVT&sharer_sharetime=1673272838533&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

作者 ｜ 褚杏娟

当地时间 1 月 6 日，Confluent 联合创始人兼 CEO Jay Kreps 发布公告称，Confluent 已经签署了收购 Immerok 的最终协议，但其并未公布收购金额。

Immerok 是一家支持专注云上构建和运行 Apache Flink 的创企，开发了名为 Immerok Cloud 的 Apache Flink 云服务，它是无服务器的，抽象出了处理流数据所需的服务器管理任务。

“他们将加入 Confluent，帮助我们为 Confluent Cloud 添加完全托管的 Flink 产品。对于 Confluent 来说，这是激动人心的一步。” Kreps 说道。

Confluent：Flink 是流处理的未来

提到 Confluent 可能大家都不陌生，作为一家基于 Apache Kafka 构建的动态数据基础平台，Confluent 在 2021 年 6 月在纳斯达克上市，当日股价上涨 25%，公司市值达到 114 亿美元。

Kreps 在公告中表示，Confluent 专注于流处理，使命就是让流数据成为新的默认值，并让数据流平台成为现代数据架构的核心。但为了使流式传输成为默认设置，需要让其变得简单，包括：在操作上容易获得流媒体功能、让使用流媒体的应用程序开发像批处理或任何其他现代应用程序一样容易和自然。

Confluent Cloud 解决了一些问题，但 Confluent 还需要使数据流的开发，即流处理，变得同样容易。“我们相信 Flink 是流处理的未来。”Kreps 说道。

长期以来，Confluent 一直通过 Kafka Streams、KSQL 和 Kafka 中的一些底层事务功能为围绕 Kafka 的新兴流处理生态系统做出贡献，这些功能有助于实现所有流技术的正确性。

Kreps 表示多年来一直在关注 Flink 的发展，并看到它在其许多客户中得到采用。Kreps 对 Flink 评价称，Flink 拥有最好的多语言支持，对 SQL、Java 和 Python 的支持一流；有一个原则性的处理模型，可以泛化批处理和流处理；具有出色的状态管理和容错模型。“也许最重要的是，它拥有一个非常聪明、创新的社区来推动它向前发展。”

在考虑我们的云产品和我们想用流处理做什么时，我们意识到提供 Flink 服务将帮助我们提供客户想要的接口和功能，并且可以作为我们未来流处理战略的核心。

“我们已经从开源、商业产品数量以及客户那里听到的信息中认识到了这一点。就像 Kafka 是整个组织中读写和共享流的明确事实标准一样，Flink 正在成为构建处理、反应和响应这些流的应用程序的明确事实标准。”Kreps 表示。

为什么是去年才成立的 Immerok

“这使我想到了为什么我们对 Immerok 团队如此兴奋。首先，他们建立了一个团队，在帮助建立 Flink 和发展其社区方面做出了令人难以置信的工作。他们利用这些专业知识构建了世界上最先进的 Apache Flink 服务。他们也是一群聪明、谦虚的人，他们有远大的计划，所以他们很适合 Confluent。”Kreps 说道。

虽然去年才成立，但 Immerok 的创始团队不可谓不豪华：至少 6 位 Apache Flink PMC、4 位 Apache Flink Committer。

“我们的设计目标是让任何人都可以利用 Flink API 的强大功能，满足从快速数据管道、面向用户的分析，到实时 ML/AI 和事务处理的一系列实时业务需求。” Immerok 联合创始人 Konstantin Knauf 表示，他也是 Flink PMC 成员，也是 Flink 最初的共同创造者之一。

虽然其在 AWS 上的 Serverless Flink 服务仍处于早期访问模式，但该公司希望在年底前将员工人数从 20 人扩大到 30 人，并已与多家企业合作，包括资产超过 1 万亿美元的荷兰银行 ING，这家银行已经在 Flink 上对流应用程序进行了标准化。

2022 年 10 月，Immerok 宣布获得了 1700 万美元的种子基金，由 CUSP Capital、468 Capital、皮质风投和 Essence VC 领投，还有包括 Apache Flink 联合创始人 Stephan Ewen 在内的天使投资者参与。

值得注意的是，Immerok 一些核心成员背景是来自 Apache Flink 项目背后公司 Ververica。比如 Immerok 联合创始人兼 CEO Holger Temme 此前在 Ververica 担任 COO，2021 年，在接连两位核心高管离职后接任了 CEO 一职。

阿里在 2019 年斥资 9000 万欧元收购了 Ververica，Ververica 被收购后一直保持独立运行。在 2021 年，Ververica 发生了一次“离职潮”。Ververica CEO、Flink 发明人 Kostas Tzoumas 是最早离职的人之一，但为了顺利过渡，他并没有立刻离开。在 1 月份，Apache Flink 联合创始人之一的 Stephan Ewen 宣布辞职离开 Ververica，并减少参与 Apache Flink 项目。2 月底，又有几个核心开发人员宣布离职，包括 Aljoscha Krettek 和 Kostas Kloudas。

显然，Ververica 一些离职的核心人员又创立了与 Ververica 业务高度重合的 Immerok：两者都专注于云上商业化 Apache Flink。

据悉，阿里云已经推出了一款云原生的实时计算 Flink 产品，提供了以 Flink SQL 为核心的开发运维平台。阿里云提供的 Flink 产品也采用了先进的 Serverless 架构，用户只要按需购买计算资源就可以使用 Flink。据悉，未来几个月之内，基于 Flink 的多云 PaaS Serverless 服务也将在全球范围内公测。

由于大量核心成员离开又做了相似的产品，有人评价阿里的收购变成了“竹篮打水”，阿里成为“冤大头”。“他们都没有竞业协议吗？”有人发出了疑问，也有人感叹道：流处理创业似乎都逃不过被收购的命运。

而在 Immerok 加后，Confluent 计划今年晚些时候在 Confluent Cloud 中推出第一个 Flink 产品，其将先从支持 SQL 开始，然后逐渐扩展到整个平台。

结束语

创始团队集体出走另起炉灶的事情也有发生过，比如当年 Facebook 开源大数据引擎 Presto 团队的分裂。2018 年底，Presto 的三位创始人 Martin Traverso，Dain Sundsrom 与企业领导发生了理念差异，三人随后离职创办了 Starburst 并做了一个从 Presto fork 出来的 Trino 项目，Trino 的社区活跃度也已超过了 Presto。Facebook 管理者期间出来表示要服务社区，但做 Presto 企业版的创始人用脚投票，纷纷站队支持三巨头的新 Presto。

不过，Flink 核心团队是否与阿里产生了理念分歧我们目前不得而知。当时，Kostas 在离职的官方声明中表示“下一阶段的发展也需要新的领导”，不过 Holger 接任他 CEO 职位后，现在摇身一变成为了 Immerok 的 CEO，可见这个说法也只是客气一下。如果两者确实存在矛盾，那么对 Flink 社区和阿里来说都不是什么好事。

但总体来说，两者的“分手”还是很体面的。时任阿里巴巴集团副总裁兼阿里云高级研究员贾扬清在 Kostas 离职时还发布声明称，“在过去的几年里，我们很荣幸有机会与 Kostas 一起工作。由于我们中的许多人本身就是开源开发人员，因此这是一个苦乐参半的时刻。我们要感谢 Kostas 建立了如此伟大的基础，我们祝愿他在未来的事业中取得圆满成功。”

**参考链接：**

https://www.confluent.io/blog/cloud-kafka-meets-cloud-flink-with-confluent-and-immerok/

https://www.ververica.com/blog/leadership-changes-to-accelerate-our-next-development-stages

https://techcrunch.com/2022/10/03/with-17m-in-funding-immerok-launches-cloud-service-for-real-time-streaming-data/

https://www.datanami.com/2022/10/04/new-flink-startup-immerok-gets-off-the-ground/

今日好文推荐

[中文编程不如英文香？今年诞生的这些国产编程语言表示不服](http://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651153824&idx=1&sn=1a147161fd0a2b7429f48876a3e2cbb2&chksm=bdb89df38acf14e54dcba8a9565ecea57b1b9cbc5e3208325e4f9718ad35f41e3e6ed5eb83e8&scene=21#wechat_redirect)

[字节回应员工因没年终奖与 HR 互殴；乐视实行 4 天半工作制：不降薪无 996，研发可准点下班；亚马逊发全员信，拟裁员 1.8 万人｜Q 资讯](http://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651153822&idx=1&sn=82c0b4606eac098da2fb3833cf042b1e&chksm=bdb89dcd8acf14db0ba74eee279ae1272e860c92e600ac2fdfa39fef68b95c280391e59c8b54&scene=21#wechat_redirect)

[2022年全球程序员收入报告出炉：首席工程师最高年薪超700万，字节跳动成国内唯一上榜公司](http://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651153745&idx=1&sn=a6f35923cce88d8a97503f6e95f7d417&chksm=bdb89d028acf1414dfa61f8572020462facb2f59ded048d86665087083af4ffbc89ea6180939&scene=21#wechat_redirect)

[为什么谷歌和苹果都要杀死移动Web？资深工程师揭秘大厂从吹捧到扼杀“内幕”](http://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651153502&idx=1&sn=5a3f86b0339e852cf82efe04a49405f6&chksm=bdb89c0d8acf151bb3a78aff004ec0163380ca27d75e0517eddc1d8bbc67b6083e1a997635bd&scene=21#wechat_redirect)

活动推荐

InfoQ 技术大会年底储值活动火热进行中，最低储值 3 万即可享全年购票 7 折，储值金额越高优惠力度越大哦！单张门票最高立省 3240 元。

ArchSummit、QCon、GMTC、PCon 等多个大会品牌均参与活动，活动详情可扫码或咨询小助手：15600537884（微信同电话）