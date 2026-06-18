> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030206_数仓建模/030206_核心知识点/数仓平台化与DataAI边界|数仓平台化与DataAI边界]]
---
title: Data + AI 下的新一代智能数仓平台
author: 大数据技术与架构
date:
url: http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247524095&idx=1&sn=51d255d82355a6a56109dd8451544dee&chksm=fc132fc3c699786e46e2758b006734617a37216eeba963367130da52356ca28efad0e94b6179&mpshare=1&scene=24&srcid=05078ETGIIK6vFtxSt1AcwI1&sharer_shareinfo=4ee99ea757b2b6b292c1204ab9917662&sharer_shareinfo_first=4ee99ea757b2b6b292c1204ab9917662#rd
---

本文根据《Data+AI融合趋势下的智能数仓平台建设》线下meetup演讲实录整理而成。

**01**

**MaxCompute 介绍**

MaxCompute 是阿里云自研大数据计算平台，发展至今已历经约15年的演进与优化。MaxCompute 最初命名为 ODPS，在阿里集团内部则被称为“云梯2”。ODPS 自诞生之初便致力于构建一个全面且高效的 SQL 生态系统，并针对 Hive SQL 进行了大量优化。在阿里集团内部应用也十分广泛，支撑了很多核心业务。2014-2015年，ODPS 正式登陆公共云市场，对外提供服务，并更名为 MaxCompute。凭借显著的技术优势和丰富的功能特性，MaxCompute 迅速收获众多国内外企业，用以支撑企业核心业务。

MaxCompute 有几个特点，首先是提供云原生Serverless的服务形态。与许多其他面向单租户或需要预约计算资源的平台不同，无需用户预约资源。这一特性极大地降低了运维成本，使用户能够专注于业务本身。此外，MaxCompute 还支持企业级弹性资源调度（如：弹性计算资源以及分时资源等功能），进一步帮助用户节约成本。MaxCompute 提供强大的企业级安全保障，包括多租户隔离、多层次的安全验证机制以及底层数据加密等，以确保用户数据安全。

作为强大的数仓平台，MaxCompute 不仅支持传统的离线数据分析，还支持增量和近实时场景。此外，MaxCompute 还集成了大数据 AI 一体化能力，支持数据加工、BI 分析、数据探查、数据科学等场景。MaxCompute 通过提供丰富的 SDK、Open API、Console，集成 PAI Studio 等，提供了便捷的接入方式。同时，MaxCompute 也无缝集成 DataWorks 等大数据治理平台，支持进行数据管理、血缘分析、作业提交等。

计算引擎方面，MaxCompute 提供自研 MaxCompute SQL 引擎，同时也支持开源计算引擎，如Apache Spark。此外，我们还支持自研分布式 Python 计算引擎。底层数据则通过统一的元数据管理进行统一管理，确保底层存储的数据能够被高效、一致地访问和处理。在架构设计上，我们采用了存算分离的架构，底层存储依赖于阿里云飞天盘古存储，同时还支持将数据存储在数据湖中，并支持基于数据湖的计算与分析。MaxCompute 保持了高度的开放性和兼容性。用户可以通过 Storage API，轻松地读取 MaxCompute 中的数据，并使用三方计算引擎进行处理与分析。

**02**

**数仓平台在 Data+AI 时代面临的挑战**

随着 AI 技术越来越热门，大模型能力日新月异，在大数据 AI 融合的时代背景下，传统的数仓平台面临着一系列新的挑战。通过接触客户，服务各类业务场景，我总结了几点。

首先，生成式 AI 能力成为很多客户的普遍需求。随着大模型能力不断增强，如何真正用好大模型成为一个难题。尽管开源社区提供了大量高质量模型，如通义千问2.5、DeepSeek 等，但将这些模型成功部署到生产环境中，并高效地应用于实际业务场景中，依然存在较高的技术门槛。

其次，在数据处理即 Data for AI 方面，数仓平台如何更好地支撑大模型预训练的数据处理需求，高效地处理大规模数据，包括量结构化、半结构化及非结构化数据，构成了另一个重要挑战。

此外，如何利用 AI 增强数仓自身能力，如智能查询优化，物化视图，智能诊断等，即 AI for Data 也是一个需要关注的方向。

最后，Data+AI 开发迭代速度非常快，如何提供更友好的开发测试部署环境，帮助开发者更敏捷地进行开发，也是一大难题。

**03**

**MaxCompute 面向 Data+AI 场景的解决方案**

针对上述挑战，MaxCompute 提出了一套全面的解决方案。首先，在数据管理方面，MaxCompute对接OpenLake解决方案。阿里云 OpenLake 解决方案建立在开放可控的 OpenLake 湖仓之上，提供大数据搜索与 AI 一体化服务。基于 OSS 的公共湖仓，结合元数据管理平台 DLF，支持结构化、半结构化及非结构化数据的管理，确保数据表和文件的安全访问，并具备增删改查与 IO 加速能力。该方案支持大数据、搜索和 AI 多引擎对接，实现引擎平权协同计算。

其次，针对高效的数据处理需求，特别是 AI 领域 Python 分布式数据处理的需求，我们提供了python分布式计算框架。统一Python编程接口，兼容 Pandas、XGboost 等数据处理及 ML 算子接口且自动实现分布式处理，同时能直接使用 MaxCompute 的弹性计算资源和数据接口，Python 开发者可以更加高效、便捷的在 MaxCompute 上完成大规模数据处理、可视化探索、科学计算及 ML/AI 开发等工作。

为了提升开发体验，提高开发敏捷度，我们还推出了一套交互式的开发环境，开箱即用，用户可以像开发本地 Notebook 程序一样进行开发，同时还提供诊断分析功能。

最后，我们提供了一个镜像管理平台，用户可以在自定义镜像中运行 UDF。确保开发环境与生产环境尽可能对齐，从而提高开发效率。

### ****分布式计算框架 MaxFrame****

熟悉 MaxCompute 的用户可能了解，MaxCompute 之前有很多 Python 生态产品。比如最早的 PyODPS，它提供了一套 Python API，使用户能够访问 MaxCompute，并提供 Python UDF 功能，支持将用户编写的 Python 脚本嵌入 SQL 中进行调用。然而，这些早期的产品存在一些局限性，主要体现在社区标准兼容性不足、部署灵活性差以及运维效率低下等。具体来说，尽管 PyODPS 支持 Dataframe，但它并不完全兼容 Pandas 设计标准；此外，用户写好 Python 后需要上传 Resource 在 SQL 里面调，Python UDF 里面各种 Python 包依赖，需要手工去打包上传。

针对这些问题，我们推出了新一代 Python 分布式计算框架 MaxCompute MaxFrame。基于底层的多种计算引擎 (SQL, DPE, PAI DLC/EAS)，用户可以在MaxFrame开发环境中使用一套 Python 代码完成数据预处理，模型训推等流程，解决MC中大数据和AI在开发体验和运行中彼此割裂的现状。

### ****Object Table 增强非结构化数据处理能力****

针对 AI 场景，非结构化数据越来越多，尤其是在多模态大模型训练和数据处理领域。这些非结构化数据如图片、音频、视频等，通常存储于对象存储或其他数据湖介质中。如何高效地管理和访问这些数据一直是一个难题，往往需要用户自行编写脚本，还面临数据频繁拉取和数据清洗问题。

为了解决这一问题，MaxCompute 在去年云栖大会上发布了 Object Table：

* 支持 SQL 以表的形式读取 OSS 文件元信息
* 基于 Meta Table 读取并版本化缓存 OSS 文件的多种元信息，便于SQL过滤和下推
* 基于Document Function读取文件内容，支持上传UDF 处理非结构化数据
* MaxCompute SQL 引擎基于元信息切分并发，启动大规模分布式计算能力，加快数据读取和处理的效率
* 支持生成结构化数据并写入数仓内外表
* Python生态的Maxframe框架支持使用Object table

### ****近实时计算+增全量一体化****

在众多AI应用场景中，实时数据处理的需求日益显著。以自动驾驶为例，摄像头捕捉的图像数据及各类驾驶数据需要实时导入数据仓库平台。MaxCompute提供了一套高效的近实时处理能力，以满足这些需求。MCQA 2.0 交互式查询引擎针对单租户环境支持多Quota组资源隔离，通过分时资源分组功能，用户可以以更快的速度进行数据查询，交互式查询性能优化提升1倍。此外，增量计算和增量MV-Pipeline智能编排功能，能够实现实时或自定义增量数据刷新，确保数据的新鲜度。DeltaTable支持近实时写入，Checkpoint 间隔可缩短至分钟级，支持 SQL 近实时查询。数据写入1-5min后即可查询，同时自动进行数据文件管理，包括StorageService、AutoCompaction 和 AutoSorting，以优化存储效率和查询性能。

### ****AI Function****

MaxCompute 还提供了一套对外的 AI Function，为用户提供强大的 GenAI 能力。底层对接阿里云飞天大模型，旨在简化生成式 AI 数据处理流程。如下 Demo，我们在数据湖上有一组驾驶摄像头拍摄的图片。我们封装了一套 AI Function API 接口。用户可以在 AI Function 中输入模型参数，按照模板编写提示词。然后使用 MaxFrame API 从 ObjectTable 生成 DataFrame，并调用AI Function进行图片内容分析。然后将数据存储到MaxCompute表中供后续使用。整个过程设计得非常便捷，用户无需关注复杂的模型部署或编写繁琐的脚本来读取文件数据，只需通过简单的API调用进行语义处理即可。

### ****MaxFrame LLM 算子****

基于 MaxFrame LLM 算子支持大模型预训练的高效文本去重，也有很多用户已经在使用。在大模型预训练领域，为了确保高质量的训练结果，文本去重是一个不可或缺的步骤。如果将未经去重处理的文本数据灌到大模型中，会导致训练质量下降。而文本去重过程涉及大量数据计算。MaxFrame 提供文本去重算子(MinHash+LSH)对相似文档进行高效语义去重。用户通过指定参数，即可计算 MiniHash LSH Band，聚合同 Band 相同 Hash 值的文档，最后生成计算连通图并只保存一个，非常方便高效地实现文本去重。基于公开数据集进行测试，FineWeb-edu 30亿条数据（8TB），使用4000 CU，3小时即可完成文本去重，这是一个非常好的性能指标。

### ****智能数仓能力概览****

如下图是 MaxCompute 智能数仓能力概览。通过智能诊断、智能物化视图、智能调优、数据排布等方面，真正实现利用 AI 进一步增强数仓本身各项指标和能力。

> **[300万字！全网最全大数据学习面试社区等你来！](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247518343&idx=1&sn=eaf44bee8c4d90aedd508201475b57a9&chksm=fd3ece12ca494704cdf02300bbfe39c1d4245ac30f41aba3065efe66eb9aa453a23e0b6dae6e&scene=21#wechat_redirect)**

如果这个文章对你有帮助，不要忘记 **「在看」** **「点赞」** **「收藏」** 三连啊喂！

[全网首发|大数据专家级技能模型与学习指南(胜天半子篇)](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247510040&idx=1&sn=aa335f25965975731173916f012d56f4&chksm=fd3eee8dca49679b82f632048fb64d21fac01497d1a0fe33917edb01e194caf0f9a1930a55ce&scene=21#wechat_redirect)

[互联网最坏的时代可能真的来了](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247508317&idx=1&sn=0bcb7fb6997b42306994b890eaa0d47f&chksm=fd3ee7c8ca496ede347a2971002c6ea68dcfd70abeeb90b43c8b02a2a60ebc7cec3a87ef7d52&scene=21#wechat_redirect)

[我在B站读大学，大数据专业](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247507860&idx=1&sn=807ac5003762f29c127bc4071dcebe33&chksm=fd3e9901ca4910178cfc816043ea86c29f881f70cd943d252de9ae2e21cb7e8ba80bff162e1b&scene=21#wechat_redirect)

[我们在学习Flink的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247499604&idx=1&sn=d938dfb30d221774704982d2938b30c1&chksm=fd3eb9c1ca4930d76a391241333de461ca22d2aa27472eab3cffab1564872ae37f1b48fe2d3c&scene=21#wechat_redirect)

[193篇文章暴揍Flink，这个合集你需要关注一下](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504856&idx=2&sn=6f62e2a0c756ce56773eed253d2f41ee&chksm=fd3e954dca491c5b732b7e2aa46db32efbc3f690e528522098659bb3c6e78e1582ab4529b5dc&scene=21#wechat_redirect)

[Flink生产环境TOP难题与优化，阿里巴巴藏经阁YYDS](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504742&idx=1&sn=8765c198d8ad66219a7bcabb221e4a23&chksm=fd3e95f3ca491ce52d0724b9e4154a47af0f5ea349e1e184c8fbc65aa17c0000242aa9b58429&scene=21#wechat_redirect)

[Flink CDC我吃定了耶稣也留不住他！| Flink CDC线上问题小盘点](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504813&idx=1&sn=f5cd6ae2aa2b1e30f87a5ae55971c514&chksm=fd3e9538ca491c2eb191677d070f2c7e4f1098eece00e256d6205b8a5d21b21c1e74f875f16f&scene=21#wechat_redirect)

[我们在学习Spark的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=1&sn=1f693e549f76622725b936041ff8896e&chksm=e9bff2fbdec87bedecbdeed4547d7d612c72fc8d4918614a26a4e31666cf0128fd57b537f347&scene=21#wechat_redirect)

[在所有Spark模块中，我愿称SparkSQL为最强！](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504834&idx=1&sn=46ca1a3924b8fdb89ac1c2acb0319d6c&chksm=fd3e9557ca491c41d837b917639ea62007ea16d3c2cc6c0c02d1f8a8c6e44318f5183b787c46&scene=21#wechat_redirect)

[硬刚Hive | 4万字基础调优面试小总结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247502750&idx=1&sn=bd9a9173d060dc4e4ebd49c8efc6acfe&chksm=fd3e8d0bca49041dea84da93910e5efdc4935e520525c09887c986691377aeb48e5cf7fb5667&scene=21#wechat_redirect)

[数据治理方法论和实践小百科全书](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504382&idx=2&sn=550b78802acfe727e0e77cd9195f8784&chksm=fd3e976bca491e7db2b8b2446d231736df01bbf13653804d680ac4390597ec17fa466ad4ae83&scene=21#wechat_redirect)

[标签体系下的用户画像建设小指南](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503741&idx=1&sn=e5039be93123f2e337013756a818bfc3&chksm=fd3e89e8ca4900fe603b63c5722a6fb8a32bd63d6ba23e0028851948a71b877eb1f742d95087&scene=21#wechat_redirect)

[4万字长文 | ClickHouse基础&实践&调优全视角解析](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503675&idx=1&sn=3ee6af64d0126c78b48cad219308f81e&chksm=fd3e89aeca4900b8b8954e9569ee3c0877881fac8c792bfafc22e7e9d3e8524da8eb860d33d8&scene=21#wechat_redirect)

[【面试&个人成长】社招和校招的经验之谈](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=2&sn=57ecc77718f1b6f2f262d62e8318dcc9&chksm=e9bff2fbdec87bedb1986765bfae7dbabb9aece45b0b2af147bbad1ac5d79ebc934c64df40ea&scene=21#wechat_redirect)

[大数据方向另一个十年开启 |《硬刚系列》第一版完结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504478&idx=1&sn=14efef3868ba42bd044f9618745a7fdc&chksm=fd3e94cbca491dddb961b7b8b93105b2869c5bcf4c03f9f0dc83ad62be0dce4c124b52ed0ba3&scene=21#wechat_redirect)

[我写过的关于成长/面试/职场进阶的文章](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504410&idx=1&sn=7e81ab5a324395eb0f12397c40247ca1&chksm=fd3e948fca491d9946456acbd93b2d651ae4d7a7d0127a211981e08f50f377e60d1b7c0d7b3a&scene=21#wechat_redirect)

[当我们在学习Hive的时候在学习什么？「硬刚Hive续集」](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504783&idx=1&sn=72aed147a459368ed934a007a2df3bc9&chksm=fd3e951aca491c0cade212390011eca7d8a68951fee6aa2844b8f4d14bcbd5b3ed26305d69dc&scene=21#wechat_redirect)