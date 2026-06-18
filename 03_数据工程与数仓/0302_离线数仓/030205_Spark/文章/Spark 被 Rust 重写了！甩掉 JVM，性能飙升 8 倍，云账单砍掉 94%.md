---
title: Spark 被 Rust 重写了！甩掉 JVM，性能飙升 8 倍，云账单砍掉 94%
author: 深空信使
date: AGIAGI
url: https://mp.weixin.qq.com/s?__biz=MzY4NTI5NjgzMQ==&mid=2247484118&idx=1&sn=fe57ddb06c4bbc34b3feaff934e09b85&chksm=f2322a6cac54532ab593d245af6b6859943e1c02ba859e646e5c43e7c8d7dfac89cfc7b66271&mpshare=1&scene=24&srcid=05250dYqnHPCRDTbk2xnlYr3&sharer_shareinfo=817f723c6d332fce20c75c4f73dce45a&sharer_shareinfo_first=817f723c6d332fce20c75c4f73dce45a#rd
---

导读  
LakeSail 开源项目 Sail 用纯 Rust 重写了 Spark 的执行引擎，绕开 JVM，在 TPC-H 基准测试中跑出 387 秒 vs 102 秒的成绩，内存峰值从 54GB 降到 22GB，shuffle spill 从 110GB 直接归零。官方称特定 workload 最高提速 8 倍、基础设施成本最高可降 94%。7400 万次浏览的帖子背后，到底有多少含金量？

## 一条帖子，三个数字

2026 年 5 月 9 日，LakeSail 官方 X 账号发了这样一条推文：

> "Spark rebuilt in Rust — no JVM, 8x faster, 94% less cost."

「用 Rust 重做 Spark：没有 JVM、快 8 倍、成本低 94%。」

▲ LakeSail 官方推文，7400 万次浏览，1.2 万点赞

三个数字，一个 GitHub 链接，没有任何多余的话。

这条推文的浏览量超过**7400 万**，转发 1100 次，收藏 1100 次——对于一个数据工程领域的开源项目来说，这个量级相当罕见。

但数字越漂亮，越需要拆开看。

## "快 8 倍"到底什么意思？

先看项目本身。Sail 的 GitHub README 这样定义自己：

> "Sail is a drop-in Apache Spark replacement written in Rust."

「Sail 是用 Rust 编写的 Apache Spark 替代品，目标是无缝替换。」

▲ GitHub lakehq/sail README，Apache-2.0 开源，采集时约 2600 stars

关键来了。README 里对性能的完整表述是：

> "~4× faster (up to 8× in specific workloads) than Spark and 94% cheaper on infrastructure costs."

「平均比 Spark 快约 4 倍（特定 workload 最高可达 8 倍），基础设施成本低 94%。」

注意这里的措辞：**平均约 4 倍，特定场景最高 8 倍**。推文里那个"8x faster"取的是上限值。

## 拆开 Benchmark：387 秒 vs 102 秒

官方 Benchmark 页面给出了完整的测试环境和结果，值得逐项看。

**测试条件：**

* 数据集：TPC-H 22 条查询，Scale Factor 100（100GB 原始数据），Parquet 格式
* 硬件：AWS EC2 r8g.4xlarge（16 vCPU，128GB RAM）
* 存储：独立 EBS 卷

**核心结果：**

| 指标 | Apache Spark | Sail | 差距 |
| --- | --- | --- | --- |
| 总查询时间 | 387.36 秒 | 102.75 秒 | Sail 快约 3.77 倍 |
| 单查询加速幅度 | — | — | 43%–727% |
| 内存峰值 | 54GB（持续占用） | 22GB（峰值仅 1 秒） | 内存减少 59% |
| Shuffle Spill 磁盘写入 | >110GB | **0GB** | Spark 独有的痛 |

▲ 官方 Benchmark Results 页面，TPC-H SF100，AWS r8g.4xlarge

几个值得注意的点：

**总时间看，大约快 3.8 倍**，对应 README 里"约 4 倍"的说法。单条查询差异很大，最快的场景提速 727%（约 8.27 倍），最慢的也有 43%。

**内存占用的差异比速度更惊人。**Spark 全程吃掉 54GB 内存不放手，Sail 峰值只到 22GB 且只持续 1 秒。这意味着在同样的硬件上，Sail 可以处理更大的数据集。

**Shuffle spill 归零可能是最具工程意义的数字。**用过 Spark 的人都知道，shuffle spill 到磁盘是性能杀手——Spark 在这个 benchmark 里产生了超过 110GB 的磁盘写入，而 Sail 是 0。

## 94% 成本下降怎么算出来的？

这个数字的推导链条是这样的：

Sail 比 Spark 快约 4 倍 → Sail 可以跑在**1/4 实例规格**的机器上 → 实例价格按 AWS 阶梯定价折算 → 得出"最高 94% 成本下降"。

换句话说，**94% 来自 TPC-H benchmark 加上实例规格推算，不代表每家公司迁移后账单都能砍掉 94%。**真实生产环境还涉及存储成本、网络费用、多租户开销和运维人力等因素。

但即使打个对折，能省一半基础设施费用，对很多跑大规模 Spark 作业的团队来说也够有吸引力了。

## 技术路线：Spark Connect + DataFusion + Rust Workers

Sail 能做到"不改代码就替换 Spark"，靠的是一个关键设计：**它实现了 Spark Connect 协议的 Rust 版本服务端。**

官方博客《Sail 0.3: Long Live Spark》详细解释了这个架构：

> "Sail is a server implementation of the Spark Connect protocol in Rust."

「Sail 是 Spark Connect 协议在 Rust 中的服务端实现。」

> "Every DataFrame or SQL operation from the PySpark client is translated into a DataFusion logical plan…"

「PySpark 客户端的每一个 DataFrame 或 SQL 操作，都会被转换成 DataFusion 逻辑计划。」

翻译成大白话：

1. 你的 PySpark 代码照写不改 2. 代码通过 Spark Connect 协议发给 Sail 服务端 3. Sail 把请求翻译成 DataFusion 逻辑计划 4. DataFusion 优化后生成物理计划 5. 计划在纯 Rust 的多线程 worker 上执行——没有 JVM，没有 GC pause，没有 JVM 调参的噩梦

▲ LakeSail 官网：Rust-native, Spark compatible, zero JVM overhead

这也是 Sail 和其他 Spark 加速方案的根本区别——很多方案是在 Spark 上面打补丁（比如 Photon、Gluten），Sail 是把整个执行引擎换掉了，只保留了 Spark 的 API 和协议层。

## "Drop-in Replacement"能信几成？

"Drop-in replacement"（无缝替换）是 Sail 最诱人也最危险的承诺。

GitHub README 说得很自信：

> "Compatible with the Spark Connect protocol, supporting the Spark SQL and DataFrame API with no code rewrites required."

「兼容 Spark Connect 协议，支持 Spark SQL 和 DataFrame API，无需重写代码。」

但 Hacker News 上的开发者没有那么客气。

早在 2024 年 9 月，Sail 刚出来的时候，就有人翻出了当时的兼容性测试数据：PySpark 测试通过率 53%（838/1580），Spark SQL 语句解析率 62.6%（1396/2230）。

有人直接问：

> "Kinda early to call this a drop in replacement with those numbers no?"

「以这些兼容性数字就叫 drop-in replacement，是不是太早了？」

▲ Hacker News 社区讨论，开发者对"无缝替换"表示质疑

当然，这些数据来自一年多前的早期版本。到 2026 年 5 月，Sail 已经迭代到 v0.6.2，兼容性肯定有改善。但这个质疑指向的核心问题没有变：

**真实的 Spark 生产环境远比 TPC-H 复杂。**大量边缘 SQL 函数、历史遗留的格式 quirks、自定义 UDF、各种 connector（Hive、Iceberg、Delta）、权限管理、流处理——这些长尾兼容性问题，往往才是企业能不能迁移的决定因素。

跑分上赢 Spark 不难，在真实 workload 上做到"代码不改就能跑"，这才是硬仗。

## JVM 的包袱，Rust 的野心

站远一步看，Sail 的故事其实是**Rust/Arrow/DataFusion 生态正在全面挑战 JVM 大数据栈**的缩影。

Spark 统治了大数据处理近十年，但它的 JVM 底座越来越像历史包袱：GC pause 不可预测、内存管理粗放、冷启动慢、调参复杂、云上资源利用率低。每个跑 Spark 的团队都有一本 JVM 调优血泪史。

而 Rust + Apache Arrow + DataFusion 这条路线，正在从底层重新定义数据处理的性能边界：

* **Polars**

  用 Rust 重写了 DataFrame 引擎，已经在 Python 数据分析领域大量取代 Pandas
* **DataFusion**

  作为 Rust 原生的查询引擎，被越来越多项目集成
* **InfluxDB**

  用 Rust 重写了存储引擎
* 现在**Sail**瞄准的是 Spark 的核心地盘：分布式批处理和 SQL 分析

LakeSail 官网还提到了一个有意思的定位："面向 human + agent 的数据平台"。在 AI agent 越来越多地调用数据工具的趋势下，低延迟、低资源占用、可预测性能的执行引擎确实比 JVM 栈更适合 agent 场景。

## 别急着迁移，先看清楚边界

回到最初那条推文：**"no JVM, 8x faster, 94% less cost"**——每个数字都有真实依据，但每个数字也都有前提条件。

* **8x**

  是特定 workload 的上限，平均值约 4x
* **94%**

  是基于 TPC-H benchmark 和实例缩减的推算，生产中大概率打折
* **no JVM**

  确实消除了一大类性能问题，但没有 JVM 也意味着与 Spark 生态深度绑定的工具链可能不兼容
* **drop-in replacement**

  在核心 SQL 和 DataFrame 场景下可信度较高，但企业级长尾兼容性仍需验证

Sail 是 Apache-2.0 开源的，GitHub 上有完整代码，v0.6.2 版本节奏也说明项目在持续迭代。对于被 Spark 性能和云账单折磨的团队来说，它至少值得关注和试跑。

但如果你的生产线上跑着几百个 Spark 作业，涉及各种 UDF、自定义 connector 和历史 SQL，**先拿一个非核心 workload 试水，远比一步到位切换安全得多。**

毕竟，跑分结果和生产环境之间的距离，比 387 秒和 102 秒之间的距离大得多。

---

— END —

— END —