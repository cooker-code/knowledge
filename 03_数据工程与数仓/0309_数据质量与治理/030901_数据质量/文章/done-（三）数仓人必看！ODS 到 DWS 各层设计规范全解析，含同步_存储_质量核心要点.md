> 已吸收至：[[03_数据工程与数仓/0309_数据质量与治理/030901_数据质量/030901_核心知识点/数仓SLA与发布准入闭环|数仓SLA与发布准入闭环]]
---
title: （三）数仓人必看！ODS 到 DWS 各层设计规范全解析，含同步/存储/质量核心要点
author: 白鲸开源
date:
url: https://mp.weixin.qq.com/s?__biz=MzkyODg0MzA1MQ==&mid=2247490846&idx=1&sn=c7e0d2f239da02309bdef37cf67d68d2&chksm=c33ec72192bebc5e8fbd9676861a66fc1b24e0547c8495690915bf87f3fd06f87b9a51cb87a5&mpshare=1&scene=24&srcid=0925OIQblCI4ZwePzIPwWTeY&sharer_shareinfo=ba7ebf0e4ef2b4dc6826378aa6965557&sharer_shareinfo_first=ba7ebf0e4ef2b4dc6826378aa6965557#rd
---

点击蓝字，关注我们

**《新兴数据湖仓设计与实践手册·数据湖仓建模及模型命名规范（2025年）》** 由四篇递进式指南组成，以“模型架构—公共规范—分层规范—命名规范”为主线，系统构建可演进、可治理、可共享的现代数据湖仓。

本文为系列文章第三篇，详细剖析了**数仓各层的设计规范**，包含同步、存储、质量等核心要点。

最后一篇将在此框架内，依次剖析数仓各层的命名规范，帮助企业用一套方法论完成从数据入湖到价值变现的全链路建设，敬请期待完整版。

1

**ODS层设计规范**

### 同步规范：

1. 一个系统源表只允许同步一次；
2. 全量初始化同步和增量同步处理逻辑要清晰；
3. 以统计日期和时间进行分区存储；
4. 目标表字段在源表不存在时要自动填充处理。

### 表分类与生命周期：

**1. ods流水全量表：**

* 不可再生的永久保存；
* 日志可按留存要求；
* 按需设置保留特殊日期数据；
* 按需设置保留特殊月份数据；

**2. ods镜像型全量表：**

* 推荐按天存储；
* 对历史变化进行保留；
* 最新数据仓储在最大分区；
* 历史数据按需保留；

**3. ods增量数据：**

* 推荐按天存储；
* 有对应全量表的，建议只保留14天数据；
* 无对应全量表的，永久保留；

**4. ods的etl过程中的临时表：**

* 推荐按需保留；
* 最多保留7天；
* 建议用完即删，下次使用再生成；

**5. BDSync非去重数据：**

通过中间层保留，默认用完即删，不建议保留。

### 数据质量：

1. 全量表必须配置唯一性字段标识；
2. 对分区空数据进行监控；
3. 对枚举类型字段，进行枚举值变化和分布监控；
4. ods表数据量级和记录数做环比监控；
5. ods全表都必须要有注释；

2

**公共维度层设计规范**

### 1) 设计准则

1. 一致性
   共维度在不同的物理表中的字段名称、数据类型、数据内容必须保持一致（历史原因不一致，要做好版本控制）
2. 维度的组合与拆分

* **组合原则：**

**将维度与关联性强的字段进行组合，一起查询，一起展示，两个维度必须具有天然的关系，如：商品的基本属性和所属品牌。**

**无相关性：如一些使用频率较小的杂项维度，可以构建一个集合杂项维度的特殊属性。**

**行为维度：经过计算的度量，但下游当维度处理，例：点击量 0-1000,100-1000等，可以做聚合分类。**

* **拆分与冗余：**

**针对重要性，业务相关性、源、使用频率等可分为核心表、扩展表。**

**数据记录较大的维度，可以适当冗余一些子集。**

### 2) 存储及生命周期管理

建议按天分区。

1. 3个月内最大访问跨度<=4天时，建议保留最近7天分区；
2. 3个月内最大访问跨度<=12天时，建议保留最近15天分区；
3. 3个月内最大访问跨度<=30天时，建议保留最近33天分区；
4. 3个月内最大访问跨度<=90天时，建议保留最近120天分区；
5. 3个月内最大访问跨度<=180天时，建议保留最近240天分区；
6. 3个月内最大访问跨度<=300天时，建议保留最近400天分区；

## 3 **DWD明细层设计规范**

### 1) 存储及生命周期管理

建议按天分区。

1. 3个月内最大访问跨度<=4天时，建议保留最近7天分区；
2. 3个月内最大访问跨度<=12天时，建议保留最近15天分区；
3. 3个月内最大访问跨度<=30天时，建议保留最近33天分区；
4. 3个月内最大访问跨度<=90天时，建议保留最近120天分区；
5. 3个月内最大访问跨度<=180天时，建议保留最近240天分区；
6. 3个月内最大访问跨度<=300天时，建议保留最近400天分区；

### 2) 事务型事实表设计准则

* 基于数据应用需求的分析设计事务型事实表，结合下游较大的针对某个业务过程和分析指标需求，可考虑基于某个事件过程构建事务型实时表；
* 一般选用事件的发生日期或时间作为分区字段，便于扫描和裁剪；
* 冗余子集原则，有利于降低后续IO开销；
* 明细层事实表维度退化，减少后续使用join成本。

### 3) 周期快照事实表

* 周期快照事实表中的每行汇总了发生在某一标准周期，如某一天、某周、某月的多个度量事件。
* 粒度是周期性的，不是个体的事务。
* 通常包含许多事实，因为任何与事实表粒度一致的度量事件都是被允许的。

### 4) 累积快照事实表

* 多个业务过程联合分析而构建的事实表，如采购单的流转环节。
* 用于分析事件时间和时间之间的间隔周期。
* 少量的且当前事务型不支持的，如关闭、发货等相关的统计。

4

**DWS公共汇总层设计规范**

数据仓库的性能是数据仓库建设是否成功的重要标准之一。**聚集主要是通过汇总明细粒度数据** 来获得改进查询性能的效果。通过访问聚集数据，可以减少数据库在响应查询时必须执行的工作量，能够快速响应用户的查询，同时有利于减少不同用访问明细数据带来的结果不一致问题。

### 1) 聚集的基本原则

* 一致性。聚集表必须提供与查询明细粒度数据一致的查询结果。
* 避免单一表设计。不要在同一个表中存储不同层次的聚集数据。
* 聚集粒度可不同。聚集并不需要保持与原始明细粒度数据一样的粒度，聚集只关心所需要查询的维度。

### 2) 聚集的基本步骤

**第一步：确定聚集维度**

在原始明细模型中会存在多个描述事实的维度，如日期、商品类别、卖家等，这时候需要确定根据什么维度聚集，如果只关心商品的交易额情况，那么就可以根据商品维度聚集数据。

**第二步：确定一致性上钻**

这时候要关心是按月汇总还是按天汇总，是按照商品汇总还是按照类目汇总，如果按照类目汇总，还需要关心是按照大类汇总还是小类汇总。当然，我们要做的只是了解用户需要什么，然后按照他们想要的进行聚集。

**第三步：确定聚集事实**

在原始明细模型中可能会有多个事实的度量，比如在交易中有交易额、交易数量等，这时候要明确是按照交易额汇总还是按照成交数量汇总。

### 3) 公共汇总层设计原则

除了聚集基本的原则外，公共汇总层还必须遵循以下原则：

* 数据公用性。汇总的聚集会有第三者使用吗？基于某个维度的聚集是不是经常用于数据分析中？如果答案是肯定的，那么就有必要把明细数据经过汇总沉淀到聚集表中。
* 不跨数据域。数据域是在较高层次上对数据进行分类聚集的抽象。如以业务
* 区分统计周期。在表的命名上要能说明数据的统计周期，如 \_Id表示最近1天，\_td 表示截至当天，\_nd 表示最近N天。

🫱 前文回顾：
[（一）数据模型架构原则：四层七阶，数据湖仓建模的“第一块基石”](https://mp.weixin.qq.com/s?__biz=MzkyODg0MzA1MQ==&mid=2247490765&idx=1&sn=85710eacea713faf704ba09eafcd2c30&scene=21#wechat_redirect)
[（二）一文读懂数仓设计的核心规范：从层次、类型到生命周期](https://mp.weixin.qq.com/s?__biz=MzkyODg0MzA1MQ==&mid=2247490798&idx=1&sn=cdb43fbdb9ff5f5963cebfc61bbc9fa7&scene=21#wechat_redirect)

🫱 下文预告：数仓命名规范

**用户案例**

[天翼云](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247523837&idx=1&sn=e86291d08c937f0532eb32cdd89d62cf&scene=21#wechat_redirect)[Zoom](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247523741&idx=1&sn=358e27a66d278b8121ad6488d0496e86&scene=21#wechat_redirect)[网易邮箱](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247523424&idx=1&sn=897add93a8a4f2522dc283f4f29383ef&scene=21#wechat_redirect)

[每日互动](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522464&idx=1&sn=c2be472e02a5efdd8f34bf9fec85b206&scene=21#wechat_redirect)[惠生工程](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522888&idx=1&sn=7268afede2cb6e39585192487dd8c42b&scene=21#wechat_redirect)[作业帮](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522811&idx=1&sn=fd9815e0ec661bcb02c8f23d9f5a4693&scene=21#wechat_redirect)

[博世智驾](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522746&idx=1&sn=123081e566642559acdff0dcdb0d4a2f&scene=21#wechat_redirect) [蔚来汽车](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522480&idx=1&sn=b3bf2d7397cddb239d7e2187d3e6509b&chksm=c0f3c8d7f78441c13a08c9059d6d5511b95db0ad8ad1b1307e2ff6bf1259cc4653d022082796&scene=21#wechat_redirect)[长城汽车](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522240&idx=1&sn=ec3816d5967b51d03503c045ce096f34&chksm=c0f3c7a7f7844eb1df46c560f73ab74bf2cb63d8a1ad3e6c91b730287b49c2ee9acd0c8b7b0a&scene=21#wechat_redirect)

[集度](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522205&idx=1&sn=5bb295374a090a39059ff3f9ceb99d48&chksm=c0f3c7faf7844eecb800cafba9a9fc71e5a702e017cdaa3867002aa7daddb44953a535af5e12&scene=21#wechat_redirect)[长安汽车](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522172&idx=1&sn=a1087e48b6d6bc3cd744e599ca73f0a9&chksm=c0f3c71bf7844e0de2131ac25848d0527201ff8b3c8ca2b618cce9fbef15faf9571572682f88&scene=21#wechat_redirect)[思科网讯](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522089&idx=1&sn=7dbf20157a813882ac6079225372bf94&chksm=c0f3c74ef7844e580eeac890a69fddf7e6920207c123a3ebe28253fc1f97632d985e4835a245&scene=21#wechat_redirect)

食行[生鲜](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247521940&idx=1&sn=4c630d6b098a116d4ef46800309207fb&chksm=c0f3c6f3f7844fe52dbed6b907300fe8b3fa577b3714c2c0a49be1387a10377c92b5dd0509f5&scene=21#wechat_redirect)[联通医疗](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522050&idx=1&sn=f453aaf00d16a7d2dbeb6beaa8292597&chksm=c0f3c765f7844e731d33714059b2267aed730052b0f1462a2f97c83868894e9714c78a425320&scene=21#wechat_redirect)[联想](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522084&idx=1&sn=5579a2b3ce1238aa68735517fad6d939&chksm=c0f3c743f7844e55bda58f61c48378141cd87fed5127525fb707776addad1b5d4e46da3ffb54&scene=21#wechat_redirect)

[新网银行](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522033&idx=1&sn=a02f1db461bf2ee7df271b34e484a811&chksm=c0f3c696f7844f80f98793aee9f6efcd825ff998a43b6b3b777ab3711814e1b9601f7498987f&scene=21#wechat_redirect)唯品富邦[消费金融](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522218&idx=1&sn=ef94406e939a9a745b55cc738837f962&chksm=c0f3c7cdf7844edb18500ef088173f380f9d54b87edf3c4778329a4107e87e5c8cfdab315ea4&scene=21#wechat_redirect)

[自如](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522014&idx=1&sn=3308ce7bca93b2035d52d3640a38834d&chksm=c0f3c6b9f7844faf3dc774585ca16b89e8b52bf1dff20c6954edd2601437550a8b042465ee0a&scene=21#wechat_redirect)[有赞](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247521946&idx=1&sn=c205f5c8adecadada22c09abac0841c1&chksm=c0f3c6fdf7844febf7e3962c482c44dc4570e225fa9d0db4da589a4464ccd09d7d1bd29f9c9b&scene=21#wechat_redirect)[伊利](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522088&idx=1&sn=986c7050c428a7bc9717bbdab5a4f63f&chksm=c0f3c74ff7844e590a1eba962b504ff44dfebdc85441c6cca3566b5ee88f3ea28cd39b39c393&scene=21#wechat_redirect)[当贝大数据](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522042&idx=1&sn=e2e6a89387d58781893d83fbc6181707&chksm=c0f3c69df7844f8bd8c8b2ceb1ed4678f5b3f90204846f36d738107e49e9cdec2a8c9b1c6225&scene=21#wechat_redirect)

[珍岛集团](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522317&idx=1&sn=2090fab5cd17248ea555c97d650160f7&chksm=c0f3c86af784417ca9fb1742ca9210dd6610b640f08c8bacb5e1f88dc7513255065e31a38d60&scene=21#wechat_redirect)[传智教育](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522077&idx=1&sn=2590dcbf4ddffbb04801616d79b1b647&chksm=c0f3c77af7844e6c4c90fd8f7af997b4f2ee015d209b599043472cbaa0b6e9444b631bc203b5&scene=21#wechat_redirect)[Bigo](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247521942&idx=1&sn=bf16ca2c69ea4663e03779a145ffc9fe&chksm=c0f3c6f1f7844fe700ed91c4a4218a0232e31206ae74f748852dde32db2c08cd9bf68ee440e4&scene=21#wechat_redirect)

[YY直播](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522016&idx=1&sn=7850b746a21139d2d1b1e2b8d4be967c&chksm=c0f3c687f7844f914f18a6c41dbc5750e79326bddf006e79ff94221ba96974e121cac0564d69&scene=21#wechat_redirect)  [拈花云科](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522248&idx=1&sn=1bc58df2ca858b03c7be054bcd77b374&scene=21#wechat_redirect)[太美医疗](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522362&idx=1&sn=2af194c3bc2dfc03f17b08ea99e8cdea&chksm=c0f3c85df784414b5015a37346546d841138434d75252467c629009737b0eb0a2483aeb610e3&scene=21#wechat_redirect)

[Cisco Webex](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522442&idx=1&sn=c6fc64973008fd84898496b1e9a7a3eb&scene=21#wechat_redirect)[兴业证券](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522438&idx=1&sn=3c3d71e4788c1fdf75b8d903a8e36774&scene=21#wechat_redirect)

**迁移实战**

[Azkaban](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522046&idx=1&sn=e48aa66bab5e784958b4fa1e716e0f13&chksm=c0f3c699f7844f8f253b69da14be80d68c9feccfe927fcfd0e6a11c756669186b73aa8555df4&scene=21#wechat_redirect)   [Ooize](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522042&idx=1&sn=e2e6a89387d58781893d83fbc6181707&chksm=c0f3c69df7844f8bd8c8b2ceb1ed4678f5b3f90204846f36d738107e49e9cdec2a8c9b1c6225&scene=21#wechat_redirect)（当贝迁移案例）

[Airflow （有赞迁移案例）](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522478&idx=1&sn=d519310eecff8c78d3ffb9e67b5cb20b&chksm=c0f3c8c9f78441dfe238fd70560184ddbdc5b1bb1296935a44d655d9d4f2491442fd16609190&scene=21#wechat_redirect)

[Air2phin（迁移工具）](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522168&idx=1&sn=82a260336451e9c633b60f0f5fa77b79&chksm=c0f3c71ff7844e0951efa6116ce9dc9a5cd288a7d9046643642e6446e4bc0d9065cd56c0e592&scene=21#wechat_redirect)

[Airflow迁移](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522321&idx=1&sn=4f1cb9104c7c7247376a45fb319abced&source=41&scene=21#wechat_redirect)[实践](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522321&idx=1&sn=4f1cb9104c7c7247376a45fb319abced&source=41&scene=21#wechat_redirect)

**发版消息**

[Apache DolphinScheduler 3.2.2版本正式发布！](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522421&idx=1&sn=2cf5d31d5182198f872c4c1e70f48bc2&scene=21#wechat_redirect)

[Apache DolphinScheduler 3.2.1 版本发布：增强功能与安全性的全面升级](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522347&idx=1&sn=89c72d5c7510824674154a47480075fe&scene=21#wechat_redirect)

[Apache DolphinScheduler 3.3.0 Alpha发布，功能增强与性能优化大升级！](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247523457&idx=1&sn=17c3bf447bd6ff40fefb93bf9d6abddf&scene=21#wechat_redirect)

**加入社区**

关注社区的方式有很多：

* **GitHub:** https://github.com/apache/dolphinscheduler
* **官网**：https://dolphinscheduler.apache.org/en-us
* **订阅开发者邮件**：dev@dolphinscheduler@apache.org（向邮箱发送任意内容，收到邮件后回复同意订阅即可）
* **X.com**：@DolphinSchedule
* **YouTube**：https://www.youtube.com/@apachedolphinscheduler
* **Slack**：https://join.slack.com/t/asf-dolphinscheduler/shared\_invite/zt-1cmrxsio1-nJHxRJa44jfkrNL\_Nsy9Qg

同样地，参与Apache DolphinScheduler 有非常多的参与贡献的方式，主要分为代码方式和非代码方式两种。

📂非代码方式包括：

完善文档、翻译文档；翻译技术性、实践性文章；投稿实践性、原理性文章；成为布道师；社区管理、答疑；会议分享；测试反馈；用户反馈等。

👩‍💻代码方式包括：

查找Bug；编写修复代码；开发新功能；提交代码贡献；参与代码审查等。

贡献第一个PR(文档、代码) 我们也希望是简单的，第一个PR用于熟悉提交的流程和社区协作以及感受社区的友好度。

**社区汇总了以下适合新手的问题列表**：https://github.com/apache/dolphinscheduler/pulls?q=is%3Apr+is%3Aopen+label%3A%22first+time+contributor%22

**优先级问题列表**：https://github.com/apache/dolphinscheduler/pulls?q=is%3Apr+is%3Aopen+label%3Apriority%3Ahigh

**如何参与贡献链接**：https://dolphinscheduler.apache.org/zh-cn/docs/3.2.2/%E8%B4%A1%E7%8C%AE%E6%8C%87%E5%8D%97\_menu/%E5%A6%82%E4%BD%95%E5%8F%82%E4%B8%8E\_menu

如果你❤️小海豚，就来为我点亮Star吧！

https://github.com/apache/dolphinscheduler

**你的好友秀秀子拍了拍你**

**并请你帮她点一下“分享”**