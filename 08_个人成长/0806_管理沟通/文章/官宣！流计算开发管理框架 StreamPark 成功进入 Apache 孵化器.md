---
title: 官宣！流计算开发管理框架 StreamPark 成功进入 Apache 孵化器
author: Apache StreamPark
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg5OTcwNTg1MQ==&mid=2247530379&idx=1&sn=c1774bb1be52a062e1f73e68a966eacb&chksm=c04d091ff73a80094989f87f68e8a305c3ff2f74d61eff299d19cc111d911fe187df8ec90ffd&mpshare=1&scene=24&srcid=0909QQtUuvKrbLWh7KpFiYQc&sharer_sharetime=1662703509601&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

StreamPark[1] 在 9 月 1 号顺利通过投票，正式成为全球最大的开源基金会 Apache 软件基金会 (ASF) 的孵化项目。这是 StreamPark 项目的新起点，意味着开源社区化协作将会变得更加规范以及国际化。

什么是 StreamPark

StreamPark 原名 StreamX，是一个简单易用的流处理应用开发框架和操作管理平台。于 2019 年由个人组织 streamxhub 创建，并于 2021年 4月在 GitHub 上开源，2022 年 8 月改名为 StreamPark。

StreamPark 初衷是让流处理更简单，在实时处理领域 Apache Spark 和 Apache Flink 是一个伟大的进步,尤其是 Apache Flink 被普遍认为是下一代大数据流计算引擎, 我们在使用 Flink & Spark 时发现从编程模型, 参数配置到运维管理都有很多可以抽象共用的地方, 我们将一些好的经验固化下来并结合业内的最佳实践, 通过不断努力终于诞生了今天的框架 — StreamPark , 其规范了项目的配置, 鼓励函数式编程, 定义了最佳的编程方式, 提供了一系列开箱即用的 Connectors 和一套快速开发的脚手架, 使用 StreamPark 开发,可以极大降低学习成本和开发门槛, 让开发者只用关心最核心的业务。

另一方面，在实时作业部署管理方面, 没有针对 Flink & Spark 作业的专业管理平台，这是企业在实践中会遇到的一道坎。StreamPark 提供专业的作业管理平台，包括但不限于 作业开发、调试、交互查询、部署、操作、运维等。

目前 StreamPark 只支持 Apache Flink 和 Apache Spark, 后续计划支持更多引擎。

StreamPark 发展现状

目前 StreamPark 已初步建立起了一个小型社区, 自开源以来累计发版 10 余次, Github Star 2K, 累计下载次数 5.6 K, 累计开发者共计 66 位, 项目一直处于活跃更新状态, 由衷感谢每位贡献者的努力和付出。  
  
目前公开登记使用的用户[2] 共计 30 余家, 有: inmobi, 自如, 永辉超市, 圆通速递, 天翼云, 联通, 腾讯 等, 不少公司已经大规模投入生产使用, 并写了生产实践的文章, 详情可查看往期生产实践相关文章。

开发者墙

# Contributor Over Time

为什么加入 ASF 孵化器

StreamPark 加入 ASF 孵化器主要是基于以下几个原因:

* 本身就是 ASF 大数据开源项目的生态项目，期待成为 ASF 正式一员。
* 在成熟开源基金会的指导下, 让 StreamPark 开源项目协作和运营都更加规范。
* 建立更加繁荣和多样化的开发者社区, 我们希望可以吸引更多优秀的海内外开发者加入, 让开发者社区更加多样化。
* 通过参加 ASF 相关的技术会议, 吸引更多的开源开发者加入 StreamPark 社区。

接下来社区会在 ASF 孵化器导师的引导下, 遵从 “Community over Code” 的理念来管理和运营社区, 也让每个优秀贡献者都能够被看见。

**导 师 介 绍**

* **@tison (Champion) :**ASF Member, Apache Curator PMC Member, Apache Flink Committer, ASF 孵化器导师, 公众号《夜天之书》[3] 作者
* **@姜宁 :**ASF Member, ASF 董事, ASF 孵化器导师, Apache Beijing Local Community[4] 发起人
* **@张铎 :** ASF Member, Apache HBase PMC Chair, ASF 孵化器导师
* **@Stephan Ewen :**ASF Member, Apache Flink 原始核心作者, Apache Flink 原 PMC Chair, ASF 孵化器导师
* **@Thomas Weise :**ASF Member, Apache Flink/Beam/Hudi PMC Member, ASF 孵化器导师

**特 别 感 谢**

## 

## 

## 感谢项目的 Champion @tison, 在项目进入 ASF 孵化器的过程中给予了无私的帮助和指导, 主导了项目从 Proposal 起草阶段到讨论再到发起投票整个过程, 给了很多专业的建议和指导。

## 感谢导师 @姜宁 @张铎 给予了项目在合规和流程推进上的专业指导和大力帮助。

感谢导师@tison、@姜宁、@张铎、@Stephan Ewen、@Thomas Weise 有了各位导师无私的帮助, StreamPark 在进入孵化器的过程更加顺利。未来在各位导师的指导下社区一定逐步变得更加规范和国际化。

感谢两位 Apache IPMC 导师 @吴晟 @柯振旭 和 Apache Doris Chair @陈明雨 在中间过程中也给到不少帮助和支持, 感谢 @王志鹏 在此过程中给予的帮助和支持。

**加 入 我 们**

进入 Apache 孵化器意味着 StreamPark 距离成为顶级的开源社区产品更近一步, 也是万里长征的第一步, 我们必须时刻保持开发者谦逊朴素的本质, 认真学习和遵循「The Apache Way」, 秉承更加兼容并包的心态, 迎接更多的机遇与挑战。我们诚挚欢迎更多的贡献者参与到社区建设中来。

**项目地址**

https://github.com/apache/incubator-streampark

**提交问题和建议：**

https://github.com/apache/incubator-streampark/issues

**贡献代码：**

https://github.com/apache/incubator-streampark/pulls

**订阅社区开发邮件列表 :**

dev@streampark.apache.org [5] 

社区沟通：

### 参考资料

*[1] StreamPark: *https://github.com/apache/incubator-streampark**

**[2] 登记用户: https://github.com/apache/incubator-streampark/issues/163**

*[3] 夜天之书: 公众号搜索关键词: 夜天之书, *传播开源之*道,开源治理,社区建设*

*[4] ALC Beijing: 公*众号*搜索关键词: ALC Beijing, 旨在中国本土传播 Apache 之道***

***[5] dev@streampark.apache.org: *mailto:dev@***streampark***.apache.org****