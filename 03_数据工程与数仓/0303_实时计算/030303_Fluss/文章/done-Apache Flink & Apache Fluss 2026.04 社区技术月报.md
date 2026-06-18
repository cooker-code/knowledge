---
title: Apache Flink & Apache Fluss 2026.04 社区技术月报
author: Apache Flink
date: 进藏人进藏人
url: https://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247515461&idx=1&sn=bfc392cb363c577fc6d95dba7320630b&chksm=fc8fb0196a9d274ac4223d5b8ab4967793c9ec66412e839a69f1ae9b680558aea8459fd1d180&mpshare=1&scene=24&srcid=0513m6bKJhMtjFLdG1YOyp16&sharer_shareinfo=c5ce9d035b4ff848315a29b4cd70eee2&sharer_shareinfo_first=c5ce9d035b4ff848315a29b4cd70eee2#rd
---

> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030303_Fluss/030303_版本记录|版本记录]]


4 月，Apache Flink 官方博客没有新增文章。最近一篇仍是 3 月 30 日发布的Flink CDC 3.6.0 版本公告[1]。但从 GitHub 合并记录看，Flink 社区开发并没有放慢，重点仍在基础设施、对象存储、Table / SQL、PyFlink 和 Web UI 这些长期影响生产体验的方向上。

Apache Fluss (Incubating) 这个月的动作更密集。Fluss 官方博客[2]
连续发布 5 篇文章，GitHub 合并 PR 也保持了较高节奏。多语言 SDK、列裁剪、实时多维分析、淘宝即时电商案例，是 4 月 Fluss 最值得关注的几个关键词。

这篇月报主要回答两个问题：

1. 1. Flink 这个成熟项目，4 月主要在补哪些基础能力？
2. 2. Fluss 作为孵化项目，正在把精力投向哪些产品化方向？

---

## 一、Apache Flink：没有高频发声，但底层工作还在推进

### 1. 官方博客：4 月无新增文章

4 月，Apache Flink 官方博客[3]没有新增文章。最近一篇仍是
3 月 30 日发布的 Flink CDC 3.6.0 版本公告[4]。

这并不代表社区停滞。对 Flink 这类成熟基础软件来说，很多关键进展并不总是以 release note 或博客形式出现，而是体现在 PR、文档补齐、依赖升级、backport 和小问题修复里。

### 2. GitHub 合并 PR：4 月累计 153 个

4 月，Apache Flink GitHub 合并 PR 累计 153 个。整体看下来，主要可以分成五类。

#### 基础设施与 CI：继续降低社区协作成本

CI 相关工作是 4 月比较明显的一条线。

社区在 GitHub Actions 上采用了 per-branch 缓存策略，并升级了
`actions/checkout`、`upload-artifact`、`setup-python` 等基础 Action；同时将 `jline` 升级到 `3.30.12`，并处理 Log4j、Jackson 等依赖的CVE 相关升级。相关 PR 包括 #28098[5]、#28097[6]、#28099[7]。

多分支 Maven 仓库目录也做了统一修复，覆盖 2.0、2.1、2.2、2.3 分支，
对应 PR 包括 #28049[8]、#28051[9]、#28055[10]。

此外，社区也支持对非代码变更跳过 GHA 构建，减少无效 CI 开销#28105[11]。

对一个长期活跃的大型项目来说，CI 的稳定性和速度不是“内部事务”。它直接影响贡献者提交 PR 的体验。构建更快、无效构建更少，社区协作的摩擦就会低一些。

#### Flink 2.3 规划：Native S3 FileSystem 继续补齐

Flink 2.3 相关工作中，Native S3 FileSystem 仍然值得关注。3 月社区更新中已经提到，Flink 2.3 仍在规划中，FLIP-555 Native S3已进入 accepted 状态 Flink Community Update for March 2026[12]。

4 月相关 PR 包括：

* • 将 `flink-s3-fs-native` 加入直接支持的文件系统列表
  #28046[13]。
* • 优化 Native S3 `InputStream` 关闭逻辑，剩余字节超过阈值时使用 `abort()`
  #28052[14]。
* • 补充 Native S3 Connector 配置参数文档 #28076[15]。
* • 补充 FLIP-487 用户文档 #28075[16]。

对象存储已经是云上实时计算的常见底座。因此，Native S3 相关能力不是边缘增强，而是 Flink 在云原生环境里继续提升可用性和一致性的基础工作。

#### Table / SQL：围绕语义和稳定性继续细修

Table / SQL 侧，4 月主要看到三类更新：

* • Materialized Table 增加 freshness / refresh mode 相关标志
  #28116[17]。
* • 新增 UTF-8 校验工具和 `StringData.fromUtf8Bytes` API
  #28110[18]。
* • 修复 table-planner 类加载顺序问题，并 backport 到 2.2 / 2.3
  #28057[19]、#28056[20]。

这些不是特别“抓眼球”的功能，但都和 SQL 用户的实际体验有关。Materialized Table 关系到数据新鲜度和刷新模式表达；UTF-8 工具和StringData API 关系到连接器和数据格式处理；planner 类加载顺序问题则直接影响运行稳定性。

从这个角度看，Flink SQL 的演进已经进入比较细的阶段：不只是新增能力，也在不断补齐工程细节。

#### PyFlink 与 Web UI：继续照顾开发和运维体验

PyFlink 方面，Apache Arrow 升级到 19.0.0 #28041[21]。

Web UI 方面，社区修复了 Rescales tab 展示顺序问题
#28044[22]、#28045[23]，以及 rescale history
子页面中 badge 样式未生效的问题 #28039[24]。

这些更新单独看都不大，但放在一起看，能看到 Flink 仍在持续补齐 Python 用户和运维用户的体验。一个成熟系统进入生产后，用户关心的不只是“能不能跑”，还包括“出问题时能不能看清楚”“扩缩容历史能不能解释清楚”
以及“Python 生态依赖是否跟得上”。

### 3. Flink CDC 3.6.0：虽然发布在 3 月，但 4 月仍值得关注

Flink CDC 3.6.0[25] 发布于 3 月 30 日，不属于 4 月新发布，
但它离 4 月很近，也值得在本月回顾中单独提一下。

这一版主要变化包括：

* • 支持 Flink 1.20.x / 2.2.x。
* • JDK 升级到 11。
* • 新增 Oracle Source。
* • 新增 Hudi Sink Pipeline 连接器。
* • Fluss Pipeline 连接器支持 Lenient mode。
* • 新增 PostgreSQL Schema Evolution。
* • Transform 框架增强 VARIANT 类型和 JSON 解析能力。

对实时数据集成场景来说，CDC 仍然是非常关键的一层。它一头连着业务数据库，一头连着 Flink 计算和下游湖仓系统。CDC 的连接器覆盖面、Schema Evolution、Transform 能力，都会直接影响实时链路落地时的复杂度。

### 4. 小结：Flink 4 月在做什么？

如果用一句话概括，Flink 4 月更像是在“修路”。没有特别密集的对外发布，但 CI、依赖、Native S3、Table / SQL、PyFlink、Web UI 都在继续往前走。对生产用户来说，这类进展值得关注。因为真正决定系统长期可用性的，往往就是这些基础能力和细节修复。

---

## 二、Apache Fluss (Incubating)：4 月明显加速

和 Flink 相比，Fluss 4 月的节奏更密集。

Fluss 官方博客[26]发布 5 篇文章，GitHub 合并 PR 累计 95 个。
更重要的是，这些内容不是零散更新，而是围绕一个很清楚的方向展开：
让 Fluss 更容易被开发者使用，也更容易进入真实业务链路。

### 1. 多语言 SDK：Rust 核心 + 薄绑定

4 月 6 日，Fluss 发布 Rust / Python / C++ Client 0.1.0[27]。
这个版本包含 210+ commits，多语言客户端共享一套 Rust 核心，
由 Rust 负责协议协商、批处理、重试、Arrow 数据交换等核心能力，
Python 和 C++ 则通过 thin binding 接入。

4 月 9 日，Fluss 又专门发文解释
为什么选择 Rust 构建多语言 SDK[28]。

这件事值得关注，因为它不是简单地“多支持几门语言”。如果每种语言都各自实现一套客户端，长期维护成本会很高，跨语言行为也容易不一致。

Fluss 现在选择的是一套核心实现、多语言轻量封装。这对后续 SDK 的稳定性、性能一致性和社区贡献门槛，都会有影响。

### 2. 实时多维 UV 去重：把场景讲得更具体

4 月 16 日，Fluss 发布
实时多维 UV 去重实践文章[29]。

文章用 RoaringBitmap 实现多维精确去重，维度包括渠道、城市、日期、小时等7 个维度，组合后可以形成 128 种分组。这类文章的价值在于，它没有停留在“存储性能更好”这种泛泛描述，而是把 Fluss 放进了一个具体分析场景里。

对实时分析系统来说，多维去重一直是典型难题：原始数据直接去重成本高，维度组合多，查询灵活性和资源消耗之间也难平衡。Fluss 用 bitmap 集合运算来处理这类问题，体现了它在流式分析侧的产品思路。

### 3. 淘宝即时电商案例：从多组件架构走向链路收敛

4 月 20 日，Fluss 发布
淘宝即时电商实时决策案例[30]。

这个案例里，淘宝即时电商在高频交易和大促流量洪峰场景下，
使用 Fluss 替换了 Kafka + Flink + Paimon + StarRocks 的多组件架构。

文章提到的关键能力包括：

* • Delta Join
* • Partial Update
* • Streaming-Lakehouse Unification
* • Column Pruning
* • Auto-Increment Columns

我认为这个案例最值得看的地方，不是“替换了哪些组件”，而是它背后的系统诉求：实时链路越长，状态管理、多流 join、湖仓同步、排障和运维成本都会上升。Fluss 希望解决的，正是这些问题叠加后的复杂度。

如果后续有更多类似案例出现，Fluss 的定位会越来越清楚：
它不是只做一个旁路存储，而是在尝试把实时流、更新、查询和湖仓衔接
放到一个更统一的体系里。

### 4. 列裁剪：流存储里的读效率问题

4 月 22 日，Fluss 发布
列裁剪技术文章[31]。

文章重点对比了传统消息系统里的“伪列裁剪”和 Fluss 的原生列裁剪。
在一些系统里，即使查询只需要少数字段，底层仍然需要把整条记录传输出来，所谓裁剪更多发生在消费端。

Fluss 的思路是从存储格式开始处理这个问题：

* • 使用 Arrow IPC 列式存储。
* • 服务端做零拷贝剪枝。
* • 客户端做预 shuffle 批处理。
* • 当裁剪 90% 列时，读吞吐可提升约 10 倍。

这说明 Fluss 并不只是把消息系统包装成分析系统，而是在流式存储层面重新处理列式读取、网络传输和查询效率之间的关系。

### 5. GitHub PR：产品化工作继续补齐

4 月，Fluss GitHub 合并 PR 共 95 个。从内容上看，主要集中在几个方向。

版本发布方面，Fluss 0.9.1-incubating 的发布准备已经开始，Helm chart 版本号也升级到 0.9.1-incubating，对应 PR 包括#3217[32]、#3205[33]。

存储和查询优化方面，社区推进了 lake filter push-down，
支持非分区扫描场景下的过滤下推 #3169[34]、#3181[35]；同时修复了 Partial Update 首次插入时非目标列未置空的问题#3157[36]，以及 Paimon IOManager 泄露导致 `/tmp` 磁盘耗尽的问题#3190[37]、#3193[38]。

Spark 集成方面，Fluss 增加了 `numRowsRead` 自定义指标并优化 Spark UI operator name #3196[39]；同时通过 shade 排除 `slf4j`，
降低日志依赖冲突风险 #3164[40]。

S3 和运维方面，path-style-access 配置可以通过 delegation token 传播#3165[41]、#3167[42]。Helm 也继续补齐部署能力，包括 pod annotations、labels、PDB #3154[43]、#3185[44]，existingSecret 注入 SASL 凭据#3171[45]、#3189[46]，以及外部环境变量和Secret 注入 #3184[47]。

可观测性和开发体验方面，Prometheus Reporter 增加了 filter-label-value选项 #3212[48]，网站上线 AI Doc Agent #3152[49]，Quickstart Image 被重新引入 #3150[50]，DeepFieldGetter 扫描也做了内存释放优化 #3221[51]。

这些 PR 合在一起看，Fluss 4 月不是只在做核心引擎，而是在补部署、监控、Spark 集成、S3、文档助手和快速上手体验。这是一条比较典型的产品化路径。

---

## 三、这个月我建议重点关注什么

从技术布道和社区观察的角度，我会建议大家重点看三条线。

### 1. Flink 的 Native S3 和对象存储能力

云上实时计算越来越依赖对象存储。Native S3 能力的补齐，会影响 checkpoint、文件读写、湖仓衔接等很多生产场景。

### 2. Flink Table / SQL 和 CDC 的持续演进

Flink 的使用门槛，很大程度上取决于 SQL 和 CDC 能不能把复杂实时链路包装得更稳定。Materialized Table、Schema Evolution、Transform、连接器扩展，都和业务开发效率直接相关。

### 3. Fluss 的产品化速度

Fluss 4 月的内容已经不只是项目介绍，而是开始围绕 SDK、案例、列裁剪、Helm、Spark、S3 和可观测性做系统补齐。这说明它正在从“社区孵化项目”向“可被业务团队评估和试用的产品形态”靠近。

---

## 四、结语

Flink 4 月更像是在稳步打磨底座。Fluss 4 月则更像是在加快产品化节奏。

对关注实时计算的人来说，这两个方向都值得看。前者决定成熟计算引擎在生产环境里的稳定性和长期演进；后者则代表流式存储、实时湖仓和多语言访问层的一些新尝试。

接下来，如果你已经在使用 Flink，可以继续关注 Flink 2.3、Native S3、
Flink CDC 和 Table / SQL 相关进展。如果你正在评估实时分析链路的架构简化，也可以把 Fluss 放进观察列表。

---

## 来源

`[1]` Flink CDC 3.6.0 版本公告:*https://flink.apache.org/2026/03/30/apache-flink-cdc-3.6.0-release-announcement/*
`[2]`Fluss 官方博客:*https://fluss.apache.org/blog/*
`[3]`Apache Flink 官方博客:*https://flink.apache.org/blog/*
`[4]`Flink CDC 3.6.0 版本公告:*https://flink.apache.org/2026/03/30/apache-flink-cdc-3.6.0-release-announcement/*
`[5]`#28098:*https://github.com/apache/flink/pull/28098*
`[6]`#28097:*https://github.com/apache/flink/pull/28097*
`[7]`#28099:*https://github.com/apache/flink/pull/28099*
`[8]`#28049:*https://github.com/apache/flink/pull/28049*
`[9]`#28051:*https://github.com/apache/flink/pull/28051*
`[10]`#28055:*https://github.com/apache/flink/pull/28055*
`[11]`#28105:*https://github.com/apache/flink/pull/28105*
`[12]`Flink Community Update for March 2026:*https://flink.apache.org/2026/03/01/flink-community-update-for-march-2026/*
`[13]`#28046:*https://github.com/apache/flink/pull/28046*
`[14]`#28052:*https://github.com/apache/flink/pull/28052*
`[15]`#28076:*https://github.com/apache/flink/pull/28076*
`[16]`#28075:*https://github.com/apache/flink/pull/28075*
`[17]`#28116:*https://github.com/apache/flink/pull/28116*
`[18]`#28110:*https://github.com/apache/flink/pull/28110*
`[19]`#28057:*https://github.com/apache/flink/pull/28057*
`[20]`#28056:*https://github.com/apache/flink/pull/28056*
`[21]`#28041:*https://github.com/apache/flink/pull/28041*
`[22]`#28044:*https://github.com/apache/flink/pull/28044*
`[23]`#28045:*https://github.com/apache/flink/pull/28045*
`[24]`#28039:*https://github.com/apache/flink/pull/28039*
`[25]`Flink CDC 3.6.0:*https://flink.apache.org/2026/03/30/apache-flink-cdc-3.6.0-release-announcement/*
`[26]`Fluss 官方博客:*https://fluss.apache.org/blog/*
`[27]`Rust / Python / C++ Client 0.1.0:*https://fluss.apache.org/blog/fluss\_rust\_client\_release/*
`[28]`为什么选择 Rust 构建多语言 SDK:*https://fluss.apache.org/blog/why-fluss-chose-rust-for-multi-language-sdk/*
`[29]`实时多维 UV 去重实践文章:*https://fluss.apache.org/blog/roaringbitmap-uv-deduplication/*
`[30]`淘宝即时电商实时决策案例:*https://fluss.apache.org/blog/taobao-instant-commerce-real-time-decision/*
`[31]`列裁剪技术文章:*https://fluss.apache.org/blog/column-pruning-streaming-storage/*
`[32]`#3217:*https://github.com/apache/fluss/pull/3217*
`[33]`#3205:*https://github.com/apache/fluss/pull/3205*
`[34]`#3169:*https://github.com/apache/fluss/pull/3169*
`[35]`#3181:*https://github.com/apache/fluss/pull/3181*
`[36]`#3157:*https://github.com/apache/fluss/pull/3157*
`[37]`#3190:*https://github.com/apache/fluss/pull/3190*
`[38]`#3193:*https://github.com/apache/fluss/pull/3193*
`[39]`#3196:*https://github.com/apache/fluss/pull/3196*
`[40]`#3164:*https://github.com/apache/fluss/pull/3164*
`[41]`#3165:*https://github.com/apache/fluss/pull/3165*
`[42]`#3167:*https://github.com/apache/fluss/pull/3167*
`[43]`#3154:*https://github.com/apache/fluss/pull/3154*
`[44]`#3185:*https://github.com/apache/fluss/pull/3185*
`[45]`#3171:*https://github.com/apache/fluss/pull/3171*
`[46]`#3189:*https://github.com/apache/fluss/pull/3189*
`[47]`#3184:*https://github.com/apache/fluss/pull/3184*
`[48]`#3212:*https://github.com/apache/fluss/pull/3212*
`[49]`#3152:*https://github.com/apache/fluss/pull/3152*
`[50]`#3150:*https://github.com/apache/fluss/pull/3150*
`[51]`#3221:*https://github.com/apache/fluss/pull/3221*

 

▼ 「Flink Forward Asia 2026」 ▼

Flink Forward Asia 2026 将于 6 月 26 至 27 日在深圳举行，现面向全球征集议题。活动聚焦实时计算与 AI 的融合，欢迎开发者与 AI 从业者提交创新思路与实践经验。议题将经过专业评选委员会审核，提交截止日期为 5 月 29 日。参会嘉宾可免费报名，获取技术前沿与行业动态。期待您的参与，期待您的参与，共同探索实时 AI 的未来！

* ### PC 端：https://asia.flink-forward.org/shenzhen-2026

打开 FFA 2026 官网，点击「议题征集」或者「参会」

* ### 移动端：扫描下方二维码或点击文末「阅读原文」

|  |  |
| --- | --- |
| （扫描二维码，提交议题） | （扫码即刻抢占席位） |

▼ 「实时计算 Flink 版」 ▼

复制下方链接或者扫描左边二维码

即可免费试用阿里云 **Serverless Flink**，体验新一代实时计算平台的强大能力！

了解试用详情：https://free.aliyun.com/?productCode=sc

---

▼ 关注「**Apache Flink**」 ▼

回复 FFA 2025 获取大会资料