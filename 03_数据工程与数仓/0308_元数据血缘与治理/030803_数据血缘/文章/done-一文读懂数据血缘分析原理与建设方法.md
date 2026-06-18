---
title: 一文读懂数据血缘分析原理与建设方法
author: 老司机聊数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652594385&idx=1&sn=763923d6fd2d056076f9f1f5d8f1d89d&chksm=f7b88204c0cf0b129b9fd8761fa5464573f1ace6a263697e75ed15b70291408ffbe0acaee3da&mpshare=1&scene=24&srcid=0717ChjLOfWijpWZeSBCn2RQ&sharer_shareinfo=7e4c4ab14e88fd797e968bd7cdad1df7&sharer_shareinfo_first=7e4c4ab14e88fd797e968bd7cdad1df7#rd
---

> 已吸收至：[[03_数据工程与数仓/0308_元数据血缘与治理/030803_数据血缘/030803_核心知识点/元数据与数据血缘实现分层|元数据与数据血缘实现分层]]


**前言**

有幸拜读成于念&赛助力老师的**《数据血缘分析原理与实践》**一书，对数据血缘这一概念与分析方法有了更充分的了解，无疑对企业数据建设、分析、数据治理等工作具有颇多指导意义。同时，对数据思维和数据大局观的锻炼，又提供了一种全新的视角，受益良多。

本文结合书中内容，对整本书的前半部分做简要概览，以帮助大家更好了解数据血缘分析及其建设方法。

**01**

**什么是数据血缘分析?**

数据血缘为数据全生命周期过程中的数据关系，包括数据特征的变化，即数据的来龙去脉。主要内容包括数据的来源、数据的加工方式、映射关系以及数据的流出和消费。数据血缘分析就是针对数据分析中的血缘关系做分析，主要包含数据来源分析、数据血缘影响分析和数据全链条分析三个部分。

**02**

**数据血缘的特征有哪些？**

**① 稳定性：**一旦数据血缘关系收集完毕，通常不会再有大的变化。

**② 归属性：**即便数据从生产端流向消费端，数据的归属关系依然存在。

**③ 多源性：**一个数据可以来自一个或者多个数据源，也可以由多个数据源组合而成。

**④ 可追溯性：**数据从产生到消亡的整个生命周期都可以直观地记录和查询，进行追溯。

**⑤ 层次性：**层次性主要体现在数据的分类、归纳和总结过程中，构成层次结构。

**03**

**数据血缘的重要性**

**1、破除数据质疑**

数据血缘分析技术可大大提升数据排查效率，让用户自主对数据来源以及链路进行检查，直观地发现数据生产链路各环节有无异常，快速打消终端用户对报告数据可靠性的怀疑。

**2、快速评估数据变更影响范围**

数据血缘可以对数据对象和数据流与数据图的连接进行可视化，以帮助数据架构师预测移动或更改数据将对数据本身及其下游流程和应用程序产生哪些影响，同时让整个流程的验证和更改也变得更加容易。

**3、度量数据资产价值评估**

数据血缘可以作为数据资产价值评估的一个度量工具，将原始数据、数据资源到数据产品、数据资产的过程进行量化和显现，如数据成本的记录、数据资产的登记、数据资产化进度追踪等。

**4、为数据滥用加上“道德枷锁”**

通过数据血缘的追踪，我们能确认数据的源头、OWNER和数据的流向，同时提供采集、存储、使用、传输、共享、发布、销毁等基于数据生命周期的具体信息，有利于数据确权后避免滥用的情况发生。

**04**

**数据血缘的组成部分**

**1、元数据**

元数据是最基本的数据单元，更多是描述数据的数据，比如身份证号码，数字类型是18位，前两位是省代码，后面几位是出生年月日，这些确定身份证号码是怎么来的数据即是元数据，元数据就像是组成数据血缘的基本元素，也可以说是构成数据血缘的编码规则或体系。

**2、主数据**

主数据是指在整个企业范围内各个系统(操作/事务型应用系统以及分析型系统)间要共享的数据，比如，可以是与客户、供应商、账户以及组织单位相关的数据。主数据的价值之一”统一数据标准、统一口径“对于数据血缘分析至关重要，如果缺乏主数据标准管理，数据血缘的流向以及关联的字段极有可能是错误的。

**3、业务数据**

业务数据是指由企业在业务处理过程中产生的数据，也称交易数据。包括订单合同，营销价格等。数据血缘在业务数据监测与问题定位、数据交圈起到了可追溯可视化的作用，大大提升了业务数据的质量问题。

**4、指标数据**

指标数据是基础数据按照一定业务规则或一系列公式计算加工得出的数据指标，它具有高价值性，更贴近业务场景的特点，代表着数据的最终业务价值呈现。通过数据血缘分析可以满足查看指标数据拆解过程、体现指标数据计算规则、展示指标数据的多源效果。

**05**

**数据血缘的建设**

数据血缘的建设贯穿了数据的全生命周期，通过一个周期、三种实体、五个类型、五个层级进行整体框架的规划和设计，同时，通过选择合适的数据建设方式，按照数据血缘建设六步曲进行建设。

**一个周期：**即数据的全生命周期，包括数据采集生产、数据加工、数据传输、数据使用消费、数据失效。

**三种实体：**即数据的颗粒度结构，它们构成了数据血缘的实体结构，包括数据库血缘、数据表血缘、表字段血缘。

**五个类型：**即数据血缘的五种类型，包括逻辑血缘、物理血缘、时间血缘、操作血缘和业务血缘。

**五个层级：**即数据血缘在全链路实现过程中所贯穿的各层级，包括血缘采集层、血缘处理层、血缘存储层、血缘接口层、血缘应用层。

**数据血缘的建设方法**

当前主流的数据血缘建设方法有采用开源系统建立数据血缘、引进厂商平台建立数据血缘、选择自建方式建立数据血缘三种方式。每家企业对于建设方式的选择各有不同，主要是由于企业资金投入、内部人员技术水平、人力资源投入等的不同等因素决定的。

**数据血缘建设六步曲**

数据血缘根据建设是进行数据血缘管理的前提，数据血缘工具需要具备数据从属谁、在何时、在何地、为什么和如何更改数据的问题。一个完整的数据血缘项目都应包含以下六大步骤：

诚如作者所说，数据血缘不仅仅是一种技术和方法，更是一种数据思维，它能够让我们更深层次理解数据、建设数据、治理数据、运营数据！

**- END -**

**原创不易，喜欢内容就点个赞吧！❤**

**相**

**关**

**推**

**荐**

**01 方法实践**

[> 一文讲透数据治理难点与应对策略（建议收藏）](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652590715&idx=1&sn=9d53dba86f9d0a48ab1d4f069e222bbc&chksm=f7b8b0aec0cf39b8e51f5740b82a267488de887a45ee0cf92f338dda9ebb95a16ef6b296609b&scene=21#wechat_redirect)

> [数据治理项目为什么会失败【深度剖析】](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652592090&idx=1&sn=f083b8c9650275c13c8778027cd9f9df&chksm=f7b8b50fc0cf3c19af61736a045074bc667575c2dfdf187d7e0a7b5ba96f2f9aebbfd32e9185&scene=21#wechat_redirect)

> [数据血缘分析~全网最全原创精华（建议收藏）](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652593318&idx=1&sn=cd081bb797eea44dfcf824e44cd09307&chksm=f7b8be73c0cf376535ed8c7b852bcddf8d666d4e18674dfb42f0e4c52c9ef9137f9d4afc7a4a&scene=21#wechat_redirect)

> [关于SAP-MDG的主数据治理理论概述](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652590415&idx=1&sn=a389f3260f72cf166093c5f98876de70&chksm=f7b8b39ac0cf3a8c6a8a8c72dda37bc2ce811d4232ebd56f27edabf17fdb060a7f8cfcf34d44&scene=21#wechat_redirect)

> [区块链技术对数据治理的一些思考及启发](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652590410&idx=1&sn=b133772b6970b58a074cfc0e2d4548ce&chksm=f7b8b39fc0cf3a89a3ac108597c9ff2d622c55c91433e8df8c5515c59c63009422a9fafbbb42&scene=21#wechat_redirect)

> [主数据治理工作八大难点](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652590354&idx=1&sn=a6129fe2d9cc24537a53d4462e435a0b&chksm=f7b8b3c7c0cf3ad1ea8496203420992436ac01682bb10b3c5ca81f1bdbbd12475a065fe93793&scene=21#wechat_redirect)

> [浅谈数据分析中的数据清洗方法策略](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652590308&idx=1&sn=f5abe2e7a8e9e22a7041987e027a2ecf&chksm=f7b8b231c0cf3b271a5662003ee6d821661ed227ff9cd32d6296bf06212f80c8e7a1fdc4aba7&scene=21#wechat_redirect)

> [数据资产入表难点解析(三)【数据质量提升】](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652591903&idx=1&sn=7186d09cc275e6ab8daace3e0887455b&chksm=f7b8b5cac0cf3cdcbc489bb6798a4c5999affaca39855316d3eec637e581892169b571c6d693&scene=21#wechat_redirect)

> [数据资产入表难点解析(二)【数据确权】](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652591804&idx=1&sn=bb34ee9a5a74159918df54f86f3bd5eb&chksm=f7b8b469c0cf3d7f975219428a70476608962edc8a075f40fa5b860e7f5909c1a44641b8ab31&scene=21#wechat_redirect)

> [数据资产入表难点解析(一)【数据定价】](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652591788&idx=1&sn=709375a880d2a92e2c7e5a329d1d5276&chksm=f7b8b479c0cf3d6f2d1aa8283a0856b1c8f1745f707e6c2d93e3f0e9e151d05a0a734208526e&scene=21#wechat_redirect)

> [全国一体化政务大数据体系建设指南（建议收藏）](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652590861&idx=1&sn=6fe6a331f1a5e9cfd9a54187cf3bd934&chksm=f7b8b1d8c0cf38ce2ddfff810596fd1b71e927b344d6f0b2b60d41c57efdd186d35318ca9ea6&scene=21#wechat_redirect)

**02 观点分析**

> [聊聊国产ERP和国际ERP的差异](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652591184&idx=1&sn=cc7fb20daffdedcc46d1aeaeb79a961a&chksm=f7b8b685c0cf3f93cf36b526a0dbe123d0a41f1854376fda69c986b5768729e04f74f2730c4b&scene=21#wechat_redirect)

> 国家数据局正式亮相，详解三个关键问题

> [怎样才算一个合格的【数据管理部门】？](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652592101&idx=1&sn=13aacaecdd2e6e459473e39782dbbba0&chksm=f7b8b530c0cf3c268169245c62c771f5605b31978d78e4e201416c206f2b3fc1889edfbb5355&scene=21#wechat_redirect)

03 概念解读

[> 一文带你了解什么是数据科学？](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652592130&idx=1&sn=9d43ed4e2276c5d3e91e90c86934427c&chksm=f7b8bad7c0cf33c164b61257a78a60408a556f0c098d111b38be7bf5baa1194c5790eb83b3b3&scene=21#wechat_redirect)

> [大数据是什么 | What's the Big Data?](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652591829&idx=1&sn=6dcb0290acafefbf81ffbbc4992e5256&chksm=f7b8b400c0cf3d16ff2c3282bd1f56dd7ab738fe653d1d1abd7e2e331d93eccd23a03d65ff1e&scene=21#wechat_redirect)

> [大数据处理能力：数据算力到底是什么](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652591992&idx=1&sn=ad38fe12445573d311ecab92472b239c&chksm=f7b8b5adc0cf3cbbdbbaa8eb6f46ee25f49863dbeec079fbf13f83ad539db9b4e3604640ec42&scene=21#wechat_redirect)

> [DCMM《数据管理能力成熟度评估模型》完整解读](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652592023&idx=1&sn=c250d106e892e038351c633c9ac3abd3&chksm=f7b8b542c0cf3c5449fd4e3e91db44e2aa7e647dce70852038832f15172c35a22162f2609903&scene=21#wechat_redirect)

> [什么是数据供应链管理](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652590398&idx=1&sn=63bc7113b55ac0e8aa05ed540eea1b59&chksm=f7b8b3ebc0cf3afdb33d6072248b080b852e7e500f1a69976b2adc02ae6d5151eb0bdf6f93a4&scene=21#wechat_redirect)

> [Web3.0 到底是什么？](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652590852&idx=1&sn=8f6f67aa756b73c4e14b513b89395c3b&chksm=f7b8b1d1c0cf38c714f24e6abad545e0f5eac003424823ccfd6b03251f4d29bc22630d48f915&scene=21#wechat_redirect)

04 职业成长

[> 一文教会如何拥有数据思维](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652593319&idx=1&sn=fde271d42a84d3cd2db48caa113d4ca5&chksm=f7b8be72c0cf3764675298ffe28d3b3b6fdfc4d3352af5295df87edcf1e83b8ddcd6238807ea&scene=21#wechat_redirect)

>[数据治理工程师【考试秘籍】](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652591148&idx=1&sn=2ef13c97f7d468a2c554f45d95ec3132&chksm=f7b8b6f9c0cf3fef122ba06dae70d68ec6d4da466cda2cdcf4301b054629c25683100b2585a0&scene=21#wechat_redirect)

> [数据治理CDGA考试重点70条（吐血整理！建议收藏！）](http://mp.weixin.qq.com/s?__biz=MzI5NTE5NjYwMg==&mid=2652590385&idx=1&sn=632decbffa0b1cab5314eb4625a25fb5&chksm=f7b8b3e4c0cf3af2ea1c8d1f0e9ddf4d138d432de3f681d26c3dba1a96c238fe4c7e8277ae87&scene=21#wechat_redirect)

更多优质内容，持续输出中~

**新书发售👇**

**听说你也是做数据的？****👇**