---
title: Teradata AnalyticOps——消除数据分析大规模应用的障碍
author: Teradata天睿大数据分析
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA3NjI4MDkzNA==&mid=2651034769&idx=1&sn=672cf2c786f4ef44c1bf918caee96439&chksm=84949198b3e3188e8c2894116fc19b85bc6dc6220a9b482e3fe67b960967b6014aeafa864590&mpshare=1&scene=24&srcid=0712ZXLhcxyxaxrOCJ0Vq13w&sharer_sharetime=1657599257665&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**作者**

李明军, Teradata天睿公司资深解决方案顾问

现在企业的决策和运营，越来越多的依赖数据分析的结果。可以说，数据分析能力是现在企业成功的关键能力。然而在数据分析的大规模应用方面，仍然存在诸多障碍，比如生产模型的开发、部署、运营监控以及生命周期管理等诸多环节。

**数据分析大规模应用存在的障碍**

数据分析大规模应用存在的障碍，主要存在于以下几个环节：

* 生成模型部署：数据分析模型的开发与部署之间存在的缺口。
* 生产模型运行监控：生产模型的运行效果，往往缺少监控，无法及时得知模型是否劣化、是否需要重新训练和评估。
* 生产模型的生命周期管理：模型的开发、部署、退役等需要相关的审批流程；模型的运行效果，需要定期评估，以确定模型是否需要重新训练或退役；模型的整个生命周期管理的相关知识和经验，缺少积累无法复用。
* 生产模型治理：机器学习需要很强的可审计性、合规报告以及可追溯性，才能确保数据分析结果的可管可用。

麦肯锡的一项调查表明，有65%的预测模型从来没有在生产环境中部署过；开发一个模型的平均周期是5个月；数据科学家的时间，有25%浪费在于开发测试团队的互动过程中。由此可见，企业在大规模应用数据分析方面，存在着巨大的障碍。对于任何企业来说，数据分析模型的开发部署、以及生命周期内的运行监控，始终是企业AI战略的核心。

图1：企业大规模应用数据存在的障碍

在现实世界中，数据分析模型的代码只占机器学习系统很小一部分，机器学习系统所需的周边基础设施庞大而复杂，包括分析工具、数据分析流程管理、计算分资源与服务设施的管理、以及模型的运行监控。

图2：数据分析系统所需的生态庞大切复杂

**Teradata AnalyticOps消除数据分析大规模应用障碍**

Teradata AnalyticOps是什么？Teradata AnalyticOps是企业管理数据分析模型的全生命周期的方法论和工具箱。Teradata AnalyticOps与Teradata Vantage相结合，涵盖了数据分析的整个生命周期——从业务理解到模型消费各个环节所需的方法和工具组件。

AnalyticOps能够支撑数据分析建模流程包括业务理解、数据获取与特征工程、数据探索、模型建模、生产环境部署、模型消费等各个环节。在模型开发方面，为数据科学家提供灵活性、探索、发现、建模、蓝图构想、外部数据、迭代、数据挖掘、统计数据、价值驱动能力；在模型运营阶段，提供安全、治理、合规、推广、部署、维护、一体化、测试、工程、流程驱动、等能力。

图3：Teradata AnalyticsOps涵盖数据分析的全生命周期

数据分析需要特征存储、模型训练和运行的生产环境。Teradata AnalyticOps专注于模型训练和生产环境部署环节。为数据科学家提供特征整合、特征工程和复用、供数据科学家使用的各种建模语言和工具、提供生产环境的数据、模型的训练与评估、模型运行、以及分析结果开放服务。

图4：AnalyticOps专注于模型训练和生产环境部署等环节

**BYOM，Bring Your Own Model**

数据科学家和分析师经常使用许多不同的开发平台来创建机器学习模型，针对这种情况，除了直接进行模型训练，AnalyticOps还允许数据科学家在自己最熟悉、最擅长的建模工具中完成模型训练，然后，把模型导入到Vantage中来运行。Vantage支持多种标准的模型交换格式（如 PMML、mojo、Onyx 或 M leap）导出模型。

通过BYOM，数据科学家可以使用自己最擅长的工具，在数据集外部完成模型训练，最大限度的发挥数据科学家和数据分析师的生产力。在Vantage平台上运行训练好的模型，又能够充分利用Vantage多节点的并行能力。

图5：Bring Your Own Model

**系统展示与优势总结**

AnalyticOps涵盖了数据分析建模的全流程，包括项目创建、数据探索、建模、模型训练和评估、审批与发布、运行和监控。如图所示：

图6：AnalyticOps系统能展示

总结一下，AnalyticOps优势主要以下几个方面的优势：

* 生产模型部署：将端到端的AI操作运营标准化和自动化
* 生产模型监控：监控模型是否劣化，是否需要重新训练
* 生产模型生命周期管理：指导数据科学家和部署工程师实施AI；通过自动化和标准化提高复用率
* 生产模型治理：完整血缘分析和审计跟踪

图7：AnalyticOps优势总结

**END**

**更多推荐**

## 2022年 Teradata 继续加强生态建设，进一步深化与合作伙伴的联合营销，自2022年6月起面向合作伙伴推出两项重要激励政策，所有经过 Teradata 合规审核的合作伙伴均有机会获得以下奖励：

**上期文章**

## [**********Teradata 2022下半年奖励更新，携手合作伙伴推动企业数字化转型！**********](http://mp.weixin.qq.com/s?__biz=MzA3NjI4MDkzNA==&mid=2651034608&idx=1&sn=20cc5834398d22a57c5f4ddc13fac17a&chksm=849496f9b3e31fef586fc23ee36070951d774e3d37492c136adde6876d7b8f0b1bf03f507f3c&scene=21#wechat_redirect)

Teradata是领先的多云互联的企业级智能数据平台公司，可大规模解决全球最复杂的数据挑战。我们通过将数据转化为最大资产来帮助企业释放价值。欢迎您随时通过如下任一渠道联络我们。

**网站：**https://www.teradata.com.cn

**电子邮箱：**GCA.Marketing@Teradata.com

**微信公众号：**Teradata天睿大数据分析

**微博：**@Teradata天睿

**知乎****：**Teradata天睿大数据分析

**销售热线：**400 661 2025

**往期推荐****:**

[智能工厂：数据驱动的点焊分析——来自Teradata的成功实践](http://mp.weixin.qq.com/s?__biz=MzA3NjI4MDkzNA==&mid=2651034649&idx=1&sn=33628a125634bb54c85dc732d8afde49&chksm=84949110b3e3180663186a2ac138c311eca6672599305e16ebabe042bb422cde88e7b4b53b08&scene=21#wechat_redirect)

[Teradata天睿公司基于阿里云VMware发布Vantage on Alibaba Cloud解决方案](http://mp.weixin.qq.com/s?__biz=MzA3NjI4MDkzNA==&mid=2651034584&idx=1&sn=96eb1052e87fd44013f4e6f75d29d4e0&chksm=849496d1b3e31fc7ea8c5295cad2111aae00f09630f11ee101e19f8e595a2b64691e1199cb61&scene=21#wechat_redirect)

[Teradata天睿公司被2022年IDC MarketScape评为数据和营销运营用户的全球客户数据平台领导者](http://mp.weixin.qq.com/s?__biz=MzA3NjI4MDkzNA==&mid=2651034556&idx=1&sn=f5a3dc7fc61625fe4c0bb38541c311dc&chksm=849496b5b3e31fa38ca87f454f2d1ed3aa7608d99070c4c41c2505e7dcbcfd21d0861d2b8243&scene=21#wechat_redirect)

[618购物节来临，某知名快递公司加强“跑冒滴漏”治理，赋能物流体系增收降本](http://mp.weixin.qq.com/s?__biz=MzA3NjI4MDkzNA==&mid=2651034533&idx=1&sn=e83abbba50d9bde8a321222bad2496d2&chksm=849496acb3e31fba0a165c06f8fa8e12b2d5f32aeb52bbf89a57a67c55923de2ffa46eeb9b17&scene=21#wechat_redirect)

[可持续商业实践为何能成为一种竞争优势](http://mp.weixin.qq.com/s?__biz=MzA3NjI4MDkzNA==&mid=2651034462&idx=1&sn=2434ac093a108f393a170837a0ba2627&chksm=84949657b3e31f41833422d7de2446b71c7427ac4e924052a1f8f5314898c03c84957250ace2&scene=21#wechat_redirect)

1.关注“ Teradata天睿大数据分析”微信公众号

2.在菜单栏点击您感兴趣的话题

转载是一种动力，分享是一种美德