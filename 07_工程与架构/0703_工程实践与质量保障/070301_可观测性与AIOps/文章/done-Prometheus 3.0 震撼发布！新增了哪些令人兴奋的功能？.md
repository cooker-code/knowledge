> 已吸收至：[[07_工程与架构/0703_工程实践与质量保障/070301_可观测性与AIOps/070301_核心知识点/监控告警与稳定性来源校准|监控告警与稳定性来源校准]]、[[07_工程与架构/0703_工程实践与质量保障/070301_可观测性与AIOps/070301_知识地图|070301_可观测性与AIOps知识地图]]

---
title: Prometheus 3.0 震撼发布！新增了哪些令人兴奋的功能？
author: dbaplus社群
date:
url: http://mp.weixin.qq.com/s?__biz=MzkzMjYzNjkzNw==&mid=2247628527&idx=1&sn=b85b4ef9180b7598c4676b2bd93f4816&chksm=c360160b127adf1c2dccdbbfe8c4ef7449136d143f62e0dc806ce8c59851a901193f67cfb12d&mpshare=1&scene=24&srcid=0116wZ227USc3shd7cSKk9ux&sharer_shareinfo=839670b92d319f21d3cf7e837b2e63f4&sharer_shareinfo_first=839670b92d319f21d3cf7e837b2e63f4#rd
---

在柏林的 PromCon 上发布 Prometheus 3.0 beta 之后，Prometheus 团队很高兴地宣布 Prometheus 3.0 现已正式发布！

这一版本是 7 年来的首次重大更新，标志着一个重要的里程碑。Prometheus 在这段时间内经历了巨大的变化，从最初的早期采用者项目发展为云原生监控的标准组成部分。Prometheus 3.0 继续这一进程，新增了一些令人兴奋的功能，同时保持与之前版本的稳定性和兼容性。

**新特性**

以下是 beta 版本中发布的一些令人兴奋的变化，以及自那时以来的新增内容：

**新的用户界面**

Prometheus 3.0 的一大亮点是全新的用户界面，默认启用：

用户界面经过完全重写，界面更简洁，外观更现代。新增了 PromLens 风格的树形视图，并使用更现代的技术栈，未来的维护将更为简便。

有关新用户界面的更多信息，请查看 Julius 在 PromLabs 博客上的详细文章[1]。用户可以通过 old-ui 功能标志暂时启用旧用户界面。

由于新用户界面尚未经过充分测试，可能仍存在一些漏洞。如发现问题，请在 GitHub 上报告。

自 beta 版本以来，用户界面已更新，以支持 UTF-8 指标和标签名称。

**远程写入 2.0**

远程写入 2.0 在之前协议版本的基础上，新增了对多种新元素的原生支持，包括元数据、示例、创建时间戳和原生直方图。它还使用字符串驻留来减少压缩和解压缩时的有效负载大小和 CPU 使用率。对部分写入有更好的处理，以便在发生这种情况时向客户端提供更多详细信息。更多详细信息可在此处[2]找到。

**UTF-8 支持**

Prometheus 现在允许默认使用所有有效的 UTF-8 字符作为指标和标签名称，以及标签值，和 2.x 版本一致。

用户需确保其指标生成器配置为传递 UTF-8 名称。如果任一方不支持 UTF-8，指标名称将使用传统下划线替换方法进行转义。PromQL 查询可以使用新的引号语法来检索 UTF-8 指标，用户也可以手动指定 \_\_name\_\_ 标签名称。

目前，只有 Go 客户端库已更新以支持 UTF-8，其他语言的支持将很快添加。

**OTLP 支持**

与我们对 OpenTelemetry 的承诺一致，Prometheus 3.0 引入了多个新功能，以提高与 OpenTelemetry 的互操作性。

**OTLP 数据接收**

Prometheus 可以配置为 OTLP 指标协议的原生接收器，在 /api/v1/otlp/v1/metrics 端点接收 OTLP 指标。

有关将 OTLP 指标流量引入 Prometheus 的最佳实践，请参阅我们的**指南[3]。**

**UTF-8 规范化**

借助 UTF-8 支持，用户可以存储和查询 OpenTelemetry 指标，而无需烦人的名称更改，如将点号改为下划线。

这显著减少了用户和工具之间因 OpenTelemetry 语义约定或 SDK 定义与实际可查询内容不一致而产生的混淆。

为支持 OTLP 数据接收，Prometheus 3.0 实验性支持不同的翻译策略。有关详细信息，请查看 **Prometheus 配置中的 OTLP 部分[4]。**

注意：虽然 “NoUTF8EscapingWithSuffixes” 策略允许特殊字符，但仍会添加必要的后缀以获得最佳体验。有关未来工作的提案，请查看 **Prometheus 的无后缀实现[5]。**

**原生直方图**

原生直方图是一种 Prometheus 指标类型，提供比经典直方图更高效、成本更低的替代方案。与需要根据数据集选择（并可能需要更新）桶边界不同，原生直方图根据指数增长预设了桶边界。

原生直方图仍处于实验阶段，默认未启用，可以通过传递 --enable-feature=native-histograms 来开启。某些方面（如文本格式和访问函数/操作符）仍在积极设计中。

**破坏性变更**

Prometheus 社区致力于在重大版本中不破坏现有功能。借助新的重大版本，我们抓住机会清理了一些小的长期问题。换句话说，Prometheus 3.0 包含了一些破坏性变更，包括功能标志、配置文件、PromQL 和抓取协议的变更。

请阅读**迁移指南[6]**，以了解你的设置是否受到影响以及需要采取的措施。

**性能**

自 Prometheus 2.0 以来，我们在社区取得了令人印象深刻的成就。我们都喜欢数字，因此让我们来庆祝在 TSDB 模式下 CPU 和内存使用效率的提升。以下是三版本 Prometheus 在 8 核心和 49 GB 可分配内存的节点上的性能数据。

* 2.0.0（7 年前）
* 2.18.0（4 年前）
* 3.0.0（现在）

更令人印象深刻的是，这些数据是使用我们的 **prombench 宏基准测试[7]**得到的，使用相同的 PromQL 查询、配置和环境，强调了核心功能的向后兼容性和稳定性，即使在 3.0 中也是如此。

**接下来的计划**

在 Prometheus 及其生态系统中，仍有许多令人兴奋的功能和改进。以下是一些非详尽列表，希望能激励你参与并加入我们！

* 新的、更具包容性的治理
* 更多 OpenTelemetry 兼容性和功能
* OpenMetrics 2.0，现已由 Prometheus 治理！
* 原生直方图稳定性（以及自定义桶！）
* 更多优化！
* 在更多 SDK 和工具中覆盖 UTF-8 支持

**尝试一下！**

你可以通过从我们的官方网站下载**二进制文件[8]**和**容器镜像[9]**来试用 Prometheus 3.0。

如果你是从 Prometheus 2.x 升级，请查看迁移指南以获取有关你需要进行的调整的更多信息。请注意，我们强烈建议在升级到 3.0 之前先升级到 2.55。可以从 3.0 回滚到 2.55，但无法回滚到更早版本。

****>****>****>****>****

**参考资料**

* **[1]详细文章:** https://promlabs.com/blog/2024/09/11/a-look-at-the-new-prometheus-3-0-ui/
* **[2]此处:**https://prometheus.io/docs/specs/remote\_write\_spec\_2\_0/
* **[3]指南:** https://prometheus.io/docs/guides/opentelemetry
* **[4]Prometheus 配置中的 OTLP 部分:**https://prometheus.io/docs/prometheus/latest/configuration/configuration/#:%7E:text=Settings%20related%20to%20the%20OTLP%20receiver%20feature
* **[5]Prometheus 的无后缀实现:**https://github.com/prometheus/proposals/pull/39
* **[6]迁移指南:**https://prometheus.io/docs/prometheus/3.0/migration/
* **[7]prombench 宏基准测试:**https://github.com/prometheus/prometheus/pull/15366
* **[8]二进制文件:** https://prometheus.io/download/#prometheus
* **[9]容器镜像:**https://quay.io/repository/prometheus/prometheus?tab=tags

来源丨公众号：云原生前沿笔记（ID：gh\_12fe4ff61d71）

dbaplus社群欢迎广大技术人员投稿，投稿邮箱：editor@dbaplus.cn