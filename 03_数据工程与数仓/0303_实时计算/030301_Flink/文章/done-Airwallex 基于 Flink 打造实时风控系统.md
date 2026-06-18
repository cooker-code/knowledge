> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink实时特征与变量计算架构模式|Flink实时特征与变量计算架构模式]]、[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink生产最佳实践|Flink生产最佳实践]]
---
title: Airwallex 基于 Flink 打造实时风控系统
author: Apache Flink
date:
url: http://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247504584&idx=1&sn=354fb021342419eb310eb578dbf3f7ad&chksm=fd385c8aca4fd59c8f0d3a861dea79ed2302b651e685f3276e27d5109a614a0edb6ca8f06609&mpshare=1&scene=24&srcid=0322OoUbOqXY5L01rWjZVw5h&sharer_sharetime=1679488414326&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**摘要：**本文整理自 Airwallex Risk ML Platform Team 董大凡，在 Flink Forward Asia 2022 实时风控专场的分享。本篇内容主要分为五个部分：

1. 背景介绍
2. 应对方案
3. 技术挑战与亮点
4. 可用性保证
5. 线上表现

**Tips：**点击**「阅读原文」**查看原文视频&演讲 ppt

##

**01**

**背景介绍**

##

Airwallex 是一个以“创建一个让企业跨境运营的环境，以此助推全球的数字经济发展。”为目标的金融科技公司。服务有 GTPN、PA、Scale。目前 Airwallex 总用户数超过 6 万，遍布 20+个国家/地区，且用户数保持稳步增长，过去一年新增用户超过 100%；日平均交易数超过 12 万笔；峰值交易流量超过 100 笔/秒；单个交易请求的总时延不能超过 800 毫秒（P99）。

以上就是 Airwallex 大体的服务。那么对于我们来说，风险在哪里呢？

平台必须降低的一些风险：

•   对于 GTPN 这种转账服务，会有一些人想利用我们这种国际转账的能力去做洗钱。

•   对于 PA 有一些收款服务，就会涉及到欺诈和纠纷事件。比如我们的客户是一个电商客户，如果他卖假货，然后利用我们对收货时延的宽容，把钱从客户那里通过我们的渠道转移到自己的账户上。当客户发现收到的货有问题提出申诉的时候，责任就会由我们来承担。

•   对于 Scale 会涉及到很多个子用户，他们之间会有很多内部的转账操作，也会有一些欺诈行为。

针对以上风险，我们公司成立了 RISK 团队。公司对这个团队的要求是，希望能够高效识别并拦截恶意转账；并且决策过程可查询，决策结果可解释；同时确保用户体验不受影响。

作为 ML Platform 团队，我们提供了一个稳定高效的 Feature 计算平台，并对机器学习工程师提供了友好的编程接口，还提供了一站式模型训练和部署解决方案。

> ML Platform 团队成员：Xin Hao、Yusup Ashrap、Michael Liu、Tim Zhu。

##

**02**

**应对方案**

##

### **2.1 Airwallex 业务与产品介绍**

对于应对风险，业内已经有了很多探讨，可以总结为以下几点。

•   传统规则引擎，可扩展性有限，无法处理如此复杂多变的场景。如果想处理这种场景，会让规则或者执行逻辑变得非常难以维护。

•   引入机器学习技术对风险进行探测已经成为行业发展的主流方向。

•   在风控领域，模型需要大量频繁回看不同时间周期内特定账户行为特征，使用传统离线数据处理方式，无法及时产生所需数据（Feature）。

•   所以综上所述，我们决定拥抱 Flink 原生流计算能力，构建流式风控平台。

从上图左侧部分可以看到，Scale/GTPN/PA 是我们正在执行转账的服务。每一笔服务都会给我们的 Risk Decision Service 发一个通知，并告诉它是 Approve 还是 Reject。所以现在我们把 Risk Decision Service 当成一个黑盒看，就是这样一个架构。

因为 Risk Decision Service 需要一些机器学习的技术或者其他技术去做计算，所以它需要相对丰富的数据。我们现在有两个数据源，一个是流的 Kafka 数据源，它有个 Account Transaction Log。即每发生一笔交易，都会有一条数据实时汇总到服务里去。另一个是批的数据源，它是一个相对比较稳定不会变化的数据。因为我们是借助 Google 的云存储，所以它会放在 Google Cloud 上。

然后将这两个数据 Feed 给 Risk Decision Service 供它进行计算。

接下来需要把数据计算出来。Flink Feature Calculation Engine 会同时接入流数据和批数据进行 Join，提供一个更丰富的流数据，供我们计算 Feature。同时，我们还有一个基于 Redis 的 Feature Cache System，它可以在 Inference 的时候，实时从里面读取数据，然后把生成的数据实时的写入到 Redis 里。

然后就引入了 Risk Model Inference Engine，它会调用一些机器学习模型或规则模型，读取一些 Feature。然后对每一笔进入的 Transaction 进行风险评估，并返回一个结果，来告诉我们的客户允许还是拒绝这笔 Transaction。

上面提到需要对结果做到可查，可解释，那么就需要把所有的结果和结果在计算中用到的 Feature 都存储下来。所以如上图右侧所示，我们引入了 Data Persist 并在外边接上了 Google Cloud Storage，实时把 Model Inference Result 和生成的 Feature 汇总，然后在 Google Cloud 上落盘存储。

上图是我们的整体架构，下层借助了 Google Cloud 存储和 Kafka 的能力，给上层提供一些数据。上层是 Feature Generating Job，包含 Feature Serving、Feature Persist、Flink Feature Generation Job。再往上层是 Model Inference，它跟外层进行通讯，调用 Model 做一些判断，同时它也会调用右侧的 Persistlayer 存储这些数据。

最上层主要负责 Management 和 Control，它有 Flink Operator 用来调度下层 Flink Job 运行；Model Management 管理每一个上线 Model 当前版本和版本所需要的 Feature 列表；DSL Management Management 用来实现 Flink Feature Generation Job 的统一接口。

**03**

**技术挑战与亮点**

##

### **3.1 挑战与应对**

我们面对的挑战主要有左侧的 3 点。

第一个是 Feature 计算需要频繁重算历史数据。每一个 Incoming 的 Event 都可能会触发滑动窗口的滑动，然后就会触发一个 Feature 重新计算，这个计算量还是比较大的。

另外，仔细想想其实还有一个场景，比如现在必须的 Feature 里有过去一个月的总交易量，如果想重新上线一版 Feature 计算逻辑，就需要把这一个月的数据都回溯一遍才能计算出来。所以频繁重算并不只是包含 Feature 正常运行时，一个跟随滑动窗口的更新，也包含 Feature 计算逻辑更新时，对历史数据的重算。

为了解决上述问题，我们直接把流本身做成一个 Process 的介质，然后把流的窗口扩大，把它当成一个批的计算，就可以做历史数据的回溯了。

第二个是 Flink 编程逻辑过于复杂。因为 Feature 生成逻辑是由写 Decision Model 的科学家决定的，他们比较擅长做一些 ML 相关的模型开发，如果让他们学习 Flink Native 开发会比较复杂。另外我们也不用 Flink SQL，因为我们认为 Flink SQL 语言的表现力有限，一些相对复杂的跨行操作，用 SQL 开发起来会比较困难，不是非常直观。

为了解决这个问题，我们抽象了一个 DSL for ML Engineer。让工程师不需要接触 Flink 那些复杂的概念，就可以写出自己需要的 Job。

第三个是模型对 Feature 依赖关系复杂多变。在风控领域，会遇到不同的风险。每个人都会有自己 Feature 要求，而这些不同的 Feature 会有相对比较复杂的生成逻辑，这些生成逻辑都还会对应一个单独的 Flink Job。我们该如何管理这些 Flink Job 呢？

我们有一个一体化的模型和 Feature 管理。因为我们线上做 Inference 的时候，是基于某个模型的特定版本，而每个模型会有自己的 Feature 依赖关系，每个 Feature 会有自己对应的 Feature Generating Job。如果反过来，我们只要抓住模型的核心点，反推它的 Dependency，就可以保证在线上跑的只有我们必须的 Feature Generating Job。所以只要打通模型和 Feature 的 Metadata，就可以比较高效的去做 Job 的 Schedule。

### **3.2 挑战的应对细节**

接下来为大家介绍刚才三个挑战的应对细节。

第一个是 Kappa Architecture。上图左侧是一个时间轴，大概描述了我们程序的执行逻辑，时间轴上面是一个大的滑动窗口。

在新的 Feature Generating Job 上线时，需要回看两周的数据，就会使用长时间窗口，加上 event time, 用于 Job 初始化。如果当前处理的时间和 Current Time 已经小于我们的判断标准，就会自动切换为短时间窗口，加上 processing time 降低 latency。

基于以上两种机制，可以让我们的程序自动在这两个模型之间切换。我们会通过一些标志文件或者共享存储的方式把当前程序的状态暴露出来，然后由 Flink Operator 调度完成在这两种模式之间的自由切换。

第二个是 DSL for ML Engineer。上图左侧是 DSL 提供的所有 API，有 FlatMap、Map、Keyby、Sum、Value。

然后看地下的两行，两个模式都可以用上面完全同样的接口去做开发，所以 DSL 屏蔽 Feature 生成过程中流批数据的差异，也简化接口，隐藏下层细节逻辑。因为 Feature Generation 根本不需要相对比较复杂的概念，比如像 Watermark、Point-in-time Joins 都不需要暴露给科学家，他们的行为可以完全由 DSL 来表达。这样科学家就能通过一些简单的培训，独立开发 Feature Generation Job 了。

第三个是一体化的模型和 Feature 管理。如果想高效调度 Feature Generating Job，就必须要打通 Model 和 Filter 的 Metadata。那么 Model Management Service 作为整个系统的管理数据中心维护如下信息，它维护着如下几点。

•   Model 对 Feature 的依赖关系。

•   Model 的生命周期。

•   Feature 的生成逻辑。

在系统运行时，通过 Model Management Service 遍历当前使用的全部 Model，并汇总全部依赖 Feature，然后调度每一个 Feature 的生成 Job。

**04**

**可用性保证**

##

上图左侧是我们简化的系统架构。像 Inference Engine 或者 Model Management 这些服务，虽然提供的内容比较新颖，但本质上就是线上服务。对线上服务做高考用，无非就是冗余、备份、分层切换流量的操作，那么我们如何做到 Feature Generation Job 的高考用性呢？

我们想做的就是冗余。为了支持冗余，我们必须实现幂等的 Feature 生成，从而实现 Feature Generation Job 的冗余能力。在这样一个需求下，我们引入了以下 Convention。

•   使用 Feature Name 作为 Feature 的唯一标识。

•   Feature Version 单增，并在写入的时候自动 Merge。

•   Model Inference 根据注册的 Feature 名字始终消费最新版本 Data。

这里我们提供了一种额外要求，就是写 Feature 的人必须保证 Feature 向后兼容，如果不能向后兼容，就换一个名字。在这种情况下，就可以实现冗余的存在。

举个例子，现在线上出现了一个 Feature Generation 的 bug，一些 Feature 无法正常生成。我们首先会修复 bug，然后 push 一个新的 DSL，然后把 Version 值加 1，重新部署新的 Feature Generation。

新的 Job 会处于 Catchup Mode，读取历史数据，并重新计算 Feature。它计算出来的每一个 Feature 都会实时写入到，已经准备好的 Feature Catch 里。

因为旧的系统并不是完全不可用，所以我们还会保证旧的版本的生成 Job 同时在这儿，提供最大的能力。直到新的 Feature Generation Mode 已经追上最新的数据，我们就可以把旧的版本下线，由新的版本去 serve。通过这种方式，我们可以确保完成我们的高可用性。

同时，Flink Operator 同步监控系统负载，动态调整系统资源。

举了一个例子，在某一个集群里对应一些 Job 的数量，可以看到一些 Job 都可以在这被正确的 Schedule。我们做了这样一个 UI，让运维工程师可以更方便的去观察系统的运行情况。

**05**

**线上表现**

##

线上表现：PA Fraud Detection Metric

上图是我们的 PA Fraud Detection Metric，可以看到 P99 的 latency 大部分都在 800 毫秒以下，右侧有一个 spike，是由一个 deployment 造成的。

往期精选

▼ 关注「**Apache Flink**」，获取更多技术干货 ▼

**点击「阅****读原文****」，查看原文视频&演讲** **PPT**