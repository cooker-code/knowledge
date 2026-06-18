---
title: Google 广告核心基础之学习阶段&归因
author: 男寝 508
date:
url: http://mp.weixin.qq.com/s?__biz=MzA5NTI5NDE0NQ==&mid=2247484397&idx=1&sn=bedd64dba08893a0529313c78696a22a&chksm=9040c443a7374d55cd9f057b0775e58098765bc1974556c6c6017dee235b298824a94d17d6cc&mpshare=1&scene=24&srcid=070356Iv5RTawdgdUrv4YhBu&sharer_shareinfo=c1b6bd6c5e2cc131359521c36ef0300b&sharer_shareinfo_first=c1b6bd6c5e2cc131359521c36ef0300b#rd
---


> 已吸收至：[[05_数据分析与BI/0505_归因分析/0505_核心知识点/广告归因与投放链路|广告归因与投放链路]]

这是 Leon 的第 33 篇原创文章。

停更了一段时间，这段时间睡眠得到了充分的保障，好像把上半年的睡眠都补回来了。

业余时间在看两本业余的书，其中一本是《人类简史》。某站被我用成了听书软件，白嫖得美滋滋。

话不多说，今天想谈的是谷歌广告的学习阶段和广告归因。

其中理解归因尤其重要。因为归因涉及到广告后台，怎么统计广告效果的数据。而谷歌广告的算法，最终会根据这个效果数据来优化广告。

**所以，这篇文章，多数篇幅会放在归因相关的问题上。**

**学习阶段**

广告的学习阶段，就是我们设定好了谷歌广告，谷歌广告的系统，需要不断地去测试投放，最后经过算法调试，得到满足我们广告投放目标最佳策略。

其实即使退出了学习阶段，谷歌广告也一直在学习，新的转化，对于算法的影响，大于旧的转化。**只是说，当后台提示广告进入了学习阶段，意味着转化效果会变得不稳定。**

为了稳定我们的广告表现，非必要情况下，应该尽量减少广告进入学习阶段。

以下这些操作，是引起广告重新进入学习阶段因素。

*更改竞价策略（含 tCPA or tROAS 目标值）、修改广告目标&转化事件、修改系列预算（20% 以上）、新增广告组、大量增加/删除关键词、大量增加否定关键词、改动受众&地区、大量添加素材、广告暂停 7 天以上。*

**除了更改竞价策略，和大幅度修改关键词，一定会让广告进入学习阶段。**其他操作，需要看都改了什么因素组合，改动相对于原来是不是差别很大，改动的频率，三个方面来判断广告重新进入学习阶段的概率。

**谷歌广告系统，更偏向于小步慢跑。不要一次性做太大的改动，或者是改动太频繁。调整完广告以后，一般观察一两周。**

**归因窗口**

我第一次看到归因窗口这个名词，就觉得很洋气和别扭。中国人看到窗口这个词，就是一个铁栅栏。后面新冠来了，我觉得用大白话解释这个名词，变得很简单了。

**从病毒感染，到最后出现症状，这段时间就叫窗口期。不同的人，窗口期不一样。**

广告也一样，从用户接触广告，比如点击广告进入的网站，但是没购买。过了几天才购买。那这段时间就是转化的窗口期。

创建 conversion action 的时候，我们需要设置归因窗口。

归因窗口，就是我们告诉广告后台，用户对某个广告产生了某种互动后，在多少天以内，只要用户产生了转化行为，那么就把效果归因到这个广告。**这个多少天，就是归因窗口。**

实际投放过程中，不同用户的转化窗口期不一样。**一般先设置得长一些，这样可以更准确地统计广告实际效果，避免遗漏。**

进入 Google Ad 后台，在 Goals>Measurement>Attribution>Path Metrics 里面，看到最后一次点击广告或者是第一次点击广告后，几天内产生了转化的概率分布。

通过这个数据，就可以知道，目前广告的销售转化周期概率分布情况。如果转化周期很短，那么归因窗口，没必要设置太长。

**归因模型**

一个用户在产生购买前，可能会多次接触我们的广告。**归因模型，决定了怎么把这个购买的功劳，分配给这几次广告接触。**不同归因模型，算这个功劳的方法不一样。

谷歌的归因模型，已经简化了很多。**目前剩下 last click 模型和 data-driven 模型。**

last click 模型，顾名思义，就是只把广告效果，归因给最后一次广告点击。如果多数用户，是多次接触后才购买，**这个归因模型，可能会忽略了一些广告系列的作用。**

data-driven 模型，会把广告转化效果，根据情况，智能分配给对应的广告。**至于这个智能的算法是怎么样的，是个黑盒。**

可以进入 Google Ad 后台，在 Goals>Measurement>Attribution>Model comparison 里面，选择对比不同归因模型下的模型的转化数据。通过计算 cpa 和 roas， **我们可能找到采用 last click 模型时，被低估的广告系列或者是关键词。**

**归因的数据源**

在 Google Ad 后台创建 conversion action 时，可以选择两种不同的数据源。一个是 google ad conversion tracking，另一个是导入 GA4(google analytics 4)的转化事件。

由于这两个数据源，监测代码的技术原理不同，返回的数据，可能也不一样。所以选择数据源很重要。

GA4 默认采用的是 last click 的归因模型。如果用户先点击了搜索广告进入网站。然后再通过自然搜索进入网站，并产生购买。**GA4 这个时候，会把这个购买来源统计成 organic，造成数据遗漏。广告算法会认定有作用的广告为没效果。**

还有一些技术因素，**比如用户端使用了隐私政策，或者是广告屏蔽插件，可能会阻止GA4 的发送监测的 tag，造成数据遗漏。**

**GA4 还存在数据延迟问题。**GA4 数据处理数据慢，一般需要 24-48 小时，有时甚至是 72 小时。对比之下，google conversion tracking 数据回传的速度更快。

因此， google conversion tracking 的数据，相对于 GA4 来说，更快更准，就是设置操作工作量大。

所以，**对于 purchase 之类的重要转化事件，要使用 google ad conversion tracking，设置成为 primary conversion action。**primary conversion action 是广告学习优化的依据，需要准确的转化数据喂算法。

**从 GA4 导入的数据，我们可以设置成 secondary conversion action。**这样我们可以得到更加详细，全面的流量转化数据。设置操作也比较很简单，只需要导入就行。

详细的 GA4 和 google ad conversion tracking 的差别，可以去看一下官方文档解释，这里我仅贴一张图。

**转化设置排错**

**广告后台的数据 vs 独立站后台数据**

同样的时间段内，如果广告后台的转化数据，大于独立站后台的销售数据，说明 conversion tracking 设置存在问题。

**因为独立站会有一些 organic 或者是其他广告平台转化数据，正常情况下，会大于广告后台的数据。**

**非转化事件统计了转化价值**

进入 Google Ad 后台，Campaigns>Campaigns, 选择所有 campaigns，然后点击 segment，选择 conversion action。

这里可以排查，有没有非转化事件，统计了转化价值。比如 page view 事件，带来了转化价值之类的。

**有时候使用插件，或者是错误的设置，会导致这些问题。**

好的，今天的文章就写到这里。如果对你有帮助，欢迎一键三连❤️❤️❤️

---

***我的往期文章***

**① 市场洞察**

*适合自研做品牌的用，实战起来比较容易*

*产品概念线上调研设计**案例研究*

*ODI (outcome based innovation) 相关，实战起来比较难*

# [*比用户画像厉害 100 倍的用户任务*](http://mp.weixin.qq.com/s?__biz=MzA5NTI5NDE0NQ==&mid=2247483946&idx=1&sn=8718f8ea59a114be5cc5e4f59a329ca8&chksm=9040c584a7374c92686638be66c082bd43c98bdee9ca09fa837badbdfc60e5d42cef60685901&scene=21#wechat_redirect)

[*python 分析问卷调研结果，探索市场细分*](http://mp.weixin.qq.com/s?__biz=MzA5NTI5NDE0NQ==&mid=2247484277&idx=1&sn=f8d76d0bab3700600bfd61d55d53f670&chksm=9040c4dba7374dcdf096f77c6d810d406ec98df1190df40853f2d4baa6b4f717419e71501343&scene=21#wechat_redirect)

****② 产品规划****

*本少爷正在忙乱准备中。。。*

****③ 产品营销****

# [*3500 字卖点提炼真经*](http://mp.weixin.qq.com/s?__biz=MzA5NTI5NDE0NQ==&mid=2247484089&idx=1&sn=d87755dc0c3e298b34e0003a02b0edbe&chksm=9040c517a7374c01e2d6b280ded404b21713b220408df4d1b932a08382e0957c912e579a65ef&scene=21#wechat_redirect)

#

# **④ 市场推广**

[*Facebook 脸书广告核心基础*](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5NTI5NDE0NQ==&action=getalbum&album_id=2822516039514243078&subscene=159&subscene=&scenenote=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzA5NTI5NDE0NQ%3D%3D%26mid%3D2247483900%26idx%3D1%26sn%3Dcae78784b9171786e9239228346c6c75%26chksm%3D9040c652a7374f4402b16066571d90cf10147ecbf01cffa4793c7ee2398f2e645f6830b3b4ca%26token%3D129622874%26lang%3Dzh_CN%23rd&nolastread=1#wechat_redirect)

[*Google 谷歌广告核心基础*](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5NTI5NDE0NQ==&action=getalbum&album_id=3451727869737500679#wechat_redirect)

**⑤ 案例研究**

# [*从 blendjet 看中美打造品牌的差异*](http://mp.weixin.qq.com/s?__biz=MzA5NTI5NDE0NQ==&mid=2247484141&idx=1&sn=f38443092ebf1c6c4183ed13ca52680e&chksm=9040c543a7374c556d3ce3195f4f683e70a97036937d9f38a03e206c29223e9c13f95c599973&scene=21#wechat_redirect)

# **⑥ 杂谈&好书推荐**

[*《非传统营销》之科特勒说的都是对的吗？*](http://mp.weixin.qq.com/s?__biz=MzA5NTI5NDE0NQ==&mid=2247483880&idx=1&sn=15a7587c5895e43c68c443b1e35a75cd&chksm=9040c646a7374f50fc9b5e1b8fe1adcfe1acf0cc0cd8f218bf3f94a0a905ce0989d1c8af3aef&scene=21#wechat_redirect)

[*行为的误偏，本质上是认知上概念的误偏*](http://mp.weixin.qq.com/s?__biz=MzA5NTI5NDE0NQ==&mid=2247483848&idx=1&sn=26642ec0b69e54e381f03649ea94fc60&chksm=9040c666a7374f708a392ce36db4018df8c56e88d66d5a0df63b22d50ee4dabab0d07627e6e6&scene=21#wechat_redirect)