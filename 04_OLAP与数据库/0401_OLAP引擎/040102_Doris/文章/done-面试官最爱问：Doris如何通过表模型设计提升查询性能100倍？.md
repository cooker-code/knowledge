> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040102_Doris/040102_核心知识点/Doris表模型选择边界|Doris表模型选择边界]]
---
title: 面试官最爱问：Doris如何通过表模型设计提升查询性能100倍？
author: 一臻数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650485167&idx=1&sn=a676efe1848ceab7c6650702010ca7b7&chksm=f2bf9477fb32308dc33c649e7bd4ae2f6efe07f6c5ff6ef9601cecd27602014fe0f2d6067df9&mpshare=1&scene=24&srcid=1215DHNiSoGmadHjNnMVZmKH&sharer_shareinfo=e990cf0705947a5f8884070e9785ed62&sharer_shareinfo_first=e990cf0705947a5f8884070e9785ed62#rd
---

更多趣文请关注一臻数据

> “
>
> 面对每天几十上百亿条的数据流入，Doris如何合理地设计存储模型？这是摆在每位数据工程师面前的一道必答题。
>
> 合适的表模型设计就像建筑的地基，一旦根基不稳，上层再华丽的架构都会成为空中楼阁。明细模型能留存全量数据但查询吃力，主键模型擅长实时更新但不适合预聚合，聚合模型查询飞快但失去了灵活性...每种模型都像是把双刃剑，用好了锦上添花，用错了徒增烦恼。 
>
> 今天，一起来学习Doris不同表模型的应用场景，让你在处理海量数据时胸有成竹，不再迷茫。

# 玩转Doris表模型，让数据如鱼得水

不知道你有没有遇到过这样的场景：

刚接手一个数据分析项目，面对纷繁复杂的业务需求，一筹莫展 - 是选择明细模型保留所有原始数据，还是用聚合模型提升查询性能？数据需要更新时，又该如何选择合适的主键模型？

说实话，在Doris表模型的选择上，我也曾走过不少弯路。曾经因为盲目追求查询性能而处处使用聚合模型，结果遇到临时性的多维分析需求时，才发现数据的灵活性被极大地限制了。

数据就像流水，需要一个恰当的容器来承载。Doris提供了三种表模型，就像三种不同的容器，各自有着独特的特点和适用场景。今天，我们就一起深入了解这三种表模型，掌握它们的使用艺术。

## 明细模型：原汁原味的数据记录

明细模型是最简单直观的表模型，它保留了数据的原始面貌。就像一部高清纪录片，每一帧画面都清晰完整地记录下来。

在日志分析系统中，我们会记录用户的每次点击、每次操作、每条错误信息。这些数据没有聚合的需求，也不需要保证唯一性，使用明细模型再合适不过。它不仅能够保存全量数据，还能按照指定的列进行排序，方便后续的查询分析。

```
CREATE TABLE user_logs
(
    timestamp  DATETIME,
    user_id    BIGINT,
    action     VARCHAR(32),
    device     VARCHAR(64),
    location   VARCHAR(128)
)
DUPLICATE KEY(timestamp, user_id)
DISTRIBUTED BY HASH(user_id);
```

## 主键模型：数据更新的绝佳选择

在用户画像系统中，我们需要存储用户的基础信息、标签属性等数据。这些数据会随着用户行为不断更新，需要保证每个用户ID只对应一条最新记录。主键模型通过保证key列的唯一性，完美地满足了这一需求。

新版本的Doris（2.1以后）默认采用写时合并实现，极大地提升了查询性能。它就像一个智能管家，在数据写入时就完成了整理工作，确保查询时能够快速找到需要的信息。

## 聚合模型：报表分析的核心引擎

聚合模型是Doris中最富特色的表模型,它通过预聚合机制大幅提升了查询性能。让我们通过一个电商分析场景来深入理解它的魅力。

假设我们需要统计各个商家的每日销售额。在传统模式下,每次查询都需要扫描所有订单数据再做聚合,面对海量数据时性能往往难以满足需求。而使用聚合模型,我们可以这样设计:

```
CREATE TABLE shop_sales
(
    shop_id     BIGINT,
    sale_date   DATE,
    province    VARCHAR(32),
    total_amount DECIMAL(16,2) SUM,
    order_count BIGINT SUM,
    user_count  BIGINT COUNT_DISTINCT
)
AGGREGATE KEY(shop_id, sale_date, province);
```

这个设计有几个精妙之处:

* Key列(shop\_id, sale\_date, province)定义了数据聚合的粒度
* Value列使用不同的聚合函数,满足多样化的分析需求
* 数据在导入时就开始预聚合,大幅减少存储空间和查询时间

# 模型选择实战指南

## 模型选择思路

经过多年的实践,X总结出一套模型选择的思路:

1.数据特征分析

* 评估数据的规模、更新频率、查询模式等特征。比如对于物联网设备上报的传感器数据,由于数据量巨大且无需更新,明细模型是不错的选择。

2.性能目标权衡

* 写入速度: 明细模型 > 主键模型 > 聚合模型
* 查询性能: 主键模型(写时合并) > 聚合模型 > 明细模型
* 灵活性: 明细模型 > 主键模型 > 聚合模型

3.场景匹配度

* 日志分析、审计追踪 -> 明细模型
* 用户画像、配置中心 -> 主键模型(写时合并)
* 指标分析、报表统计 -> 聚合模型

## 模型应用技巧

1.聚合模型性能优化

使用agg\_state类型处理复杂聚合,它能在保持灵活性的同时提供优秀的性能:

```
CREATE TABLE user_metrics
(
    user_id BIGINT,
    metric_date DATE,
    visit_stats agg_state<group_concat(string)> generic
)
AGGREGATE KEY(user_id, metric_date);
```

2.主键模型更新优化

对于频繁更新的场景,启用部分列更新能显著提升性能:

```
CREATE TABLE user_profile
(
    user_id BIGINT,
    nickname VARCHAR(32),
    avatar VARCHAR(256),
    tags VARCHAR(1024)
)
UNIQUE KEY(user_id)
PROPERTIES(
    "enable_unique_key_merge_on_write" = "true",
    "enable_unique_key_partial_update" = "true"
);
```

掌握了这些技巧,相信你已经能够从容基于Doris应对各种数据分析场景。

欢迎在评论区分享你在使用Doris表模型时的心得体会。下一篇,我们将深入探讨Doris的其它特性,敬请期待！

---

 一臻数据致力于大数据AI时代的前沿内容分享，会持续分享更多有趣有用有态度的知识。同时也欢迎大家**投稿，共建共进**，帮助圈友们冲破认知壁垒，实现自我提升！

另外，一臻整理了一份《**Apache Doris知识库**》，其中包含 Apache Doris**学习资料、方案中心、企业实践**和**问题指南** 等内容，会持续更新，欢迎**关注公众号，免费领取**。

**资料获取** 🔗 欢迎扫描下方二维码图片 加入【**Apache Doris社区**】免费领取❗️

---

往期推荐

[*走进*开源，拥抱开源](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483656&idx=1&sn=300e90f5017ebb3d97d3e98d26d52ff7&chksm=f374e9aec40360b85c87a26d9d1af93b2807ad1c54340676ce3173fb0508b3be7ca9595182f0&scene=21#wechat_redirect)

[*大数据*平台开发规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482778&idx=1&sn=6a4a6b3bf16ab818aa38d222ce46fed6&chksm=f374ea3cc403632a0f6ef1a9728393b459c3d19926f8e9672467f278e9f56abb010b198d2b34&scene=21#wechat_redirect)

[*大数据*仓库开发规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483237&idx=1&sn=824d2125280a009dddeec3f0aa60c4f6&chksm=f374e843c40361551cbbf48c7ad58fb054246a1abc5a60166a29aa7c7a547903848873149902&scene=21#wechat_redirect)

[*大数据*质量管制规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483244&idx=1&sn=1f87c073c411c338ff73f095d250f2e5&chksm=f374e84ac403615c9006d4bcfa49374d5f652b742a7851f9848cb9ea8c0909d53c70dee36f40&scene=21#wechat_redirect)

[*Flink* CDC 1.0至3.0回忆录](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483092&idx=1&sn=396c5873c5b2bf6d66532daeae4b445f&chksm=f374ebf2c40362e4ebac29580dcb7add14c9980290769de9366924d98ad27b3e5dc3f1095613&scene=21#wechat_redirect)

[【Apache *Doris*】Manager 极致丝滑地运维管理](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482971&idx=1&sn=b0953da7f28e016edd032a5be9bd34e5&chksm=f374eb7dc403626b97e57c4e2ec4922e3c17fa71fd2714623cd84328716a53b839976e59f0c3&scene=21#wechat_redirect)

[【Apache *Doris*】如何一键实现MySQL万表整库同步？](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482994&idx=1&sn=1e10161f6f7e9777a8fd572e7bd91482&chksm=f374eb54c4036242d7b83a58bd0533e6187444adae3c6defb0e0dc6ecfe0f314c3965c2d4872&scene=21#wechat_redirect)

[【Apache *Doris*】如何实现高并发点查？（原理+实践全析）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483049&idx=1&sn=e60a95fe683a663f249af230e6606bce&chksm=f374eb0fc403621998ca50c01bad8e093f788a982ca33463d28863840e3bfd3c94791f38efb9&scene=21#wechat_redirect)

[为什么Apache *Doris*适合做大数据的复杂计算，MySQL不适合？](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483064&idx=1&sn=330709a47b85d247af7510b1557fe868&chksm=f374eb1ec40362088903ac541bd4ea3d1d4ea7962b95214405132d4a0a96fae6e23c671787d6&scene=21#wechat_redirect)

[如何正确地使用ChatGPT（角色扮演+提示工程）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482824&idx=1&sn=429e15f252a79bf18f72161c9ccc04af&chksm=f374eaeec40363f8aff13083a2ee0ef60b07ab56ac6c7aaff5ae282a5092aae82d00455c1633&scene=21#wechat_redirect)

[超强满血不收费的AI绘图教程来了（在线Stable Diffusion一键即用）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482860&idx=1&sn=ad75b434b7eae3511017c67d14dad143&chksm=f374eacac40363dc192372fbdd36ff76ad8cbdd866df5a3287c3e48476f4c9c88f2ef3e5f3db&scene=21#wechat_redirect)

点击下方蓝字关注一臻数据