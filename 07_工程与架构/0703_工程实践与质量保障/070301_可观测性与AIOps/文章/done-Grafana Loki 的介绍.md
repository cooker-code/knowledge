> 已吸收至：[[07_工程与架构/0703_工程实践与质量保障/070301_可观测性与AIOps/070301_核心知识点/结构化日志与LLM_RCA证据边界|结构化日志与LLM_RCA证据边界]]
---
title: Grafana Loki 的介绍
author: AC技术与生活
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg4MDg2OTE0MQ==&mid=2247484376&idx=1&sn=26a8504fe957b1cb0520c7382e9133f1&chksm=cf6fe9a2f81860b47d050e4645275f5263456c58bde4c25e40f9630d886117cdaa8d35380a30&mpshare=1&scene=24&srcid=0102j2kIRMo2nfUoKe6hyiZ2&sharer_shareinfo=7426f83b2555a15edb886e8e3150f277&sharer_shareinfo_first=7426f83b2555a15edb886e8e3150f277#rd
---

# Grafana Loki 简明指南

> Loki 是一个可水平扩展、高度可用、多租户的日志聚合系统，其设计灵感来自 Prometheus。它设计得非常经济高效，而且易于操作。它不索引日志内容，而是为每个日志流设置一组标签。

## 关于标签您需要了解的一切 （译文）

标签既是 Loki 对传入数据进行分片的键空间，也是用于在查询时查找日志的索引。正确使用标签对应用程序的成功至关重要，因此我很自然地选择在本系列文章中专门讨论标签的工作原理和最佳实践。

从概念上讲，标签很简单；它们是一组任意的键值对，您可以在摄取时将其分配给日志。但在实践中，它们会给新用户带来一些问题。人们可能会根据使用其他数据库的经验，先入为主地认为 Loki 的索引应该如何工作，或者他们可能会试图在 Loki 标签中使用与在 Prometheus 中使用度量标准相同的卡入度。

那么，一个很好的起点或许就是举例/演示标签在洛基中是如何工作的。想象一下日志数据的来源--生成日志的应用程序或系统。多年前，这可能是一台机器，它可能有一个类似 "Rook "的名字，是一个在 SPARC 硬件上运行 Solaris 的 Unix 系统。事实上，它可能至今还在你的梦中萦绕。

让我们把这台服务器上的日志发送给洛基。为此，我们将指定一个非常基本的单一标签：

```
{host="rook"}
```

Loki 会接收这些数据，并开始为该数据流构建数据块。

但这里有一个问题。Rook 运行着大量应用程序，我们运行的每个查询都必须查询所有这些应用程序。因此，用户在试图了解哪些日志来自哪个系统时会感到非常沮丧，查询速度也会变慢。

让我们使用标签来改善这种情况。

```
{host=”rook”, app=”webserver”}
{host=”rook”, app=”ftp”}
{host=”rook”, app=”middleware”}
```

**由于内容过长，欢迎大家看原文：**

https://grafana.com/blog/2023/12/20/the-concise-guide-to-grafana-loki-everything-you-need-to-know-about-labels/

**核心就是定义好需要指标，定义好标签，能大大提高Loki 查询性能。**

# Grafana labs

### 它还有其它更多的开源项目，更好的项目。

**比如：**

* • `Grafana Mimir` 可代替 `prometheus`
* • `Grafana Agent` All In One 的 `exporter`
* • `Grafana k6` 新一代测试神器，数据直接导入库中配合`Grafana`直接展示。
* • `OpenTelemetry` 链路跟踪，没有说哪个更好，只有哪个更合适。

**参考：** https://grafana.com/

**记得按时休息**

​