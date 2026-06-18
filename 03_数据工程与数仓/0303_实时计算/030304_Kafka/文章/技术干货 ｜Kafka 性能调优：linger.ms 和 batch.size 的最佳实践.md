---
title: 技术干货 ｜Kafka 性能调优：linger.ms 和 batch.size 的最佳实践
author: AutoMQ
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247489363&idx=1&sn=290f40dc0ca8593c24b051212396d93f&chksm=c010b1dfd691c474dcf01e46567a225054626a8d2777a1b6594bb87ac1f4b739ec2f270d5a00&mpshare=1&scene=24&srcid=1209UHoAVHwQEyfs2e0YBkQI&sharer_shareinfo=b74d5e72db70d755bfd2db953ff97e5f&sharer_shareinfo_first=b74d5e72db70d755bfd2db953ff97e5f#rd
---

**文章导读**

**2025 年 3 月 18 日，Apache Kafka 4.0 正式发布。** 在此次版本更新中，相较于架构层面的升级，开发者们也应关注一个关键的细节变更：**官方将生产者参数 linger.ms 的默认值，从沿用多年的 0ms 正式修改为 5ms。**

这一调整直击传统性能调优的认知盲区，在传统观念中，linger.ms=0 意味着“零等待”和实时发送，通常被视为降低延迟的首选策略。然而，Kafka 4.0 的默认值变更揭示了一个更深层的性能逻辑：在复杂的网络 I/O 模型中，单纯追求发送端的实时性并不等同于全局的低延迟。通过引入微小的“人工延迟”来换取更高的批处理效率，往往能显著降低系统的延迟。

以 Kafka 4.0 的默认值变更为契机，本文将 深入分析 linger.ms 和 batch.size 这两个核心参数背后的协同机制。帮助你在面对复杂的生产环境时，基于原理掌握linger.ms 和 batch.size 的最佳实践。

**linger.ms 和 batch.size 参数**

为了透彻理解这次变更背后的深层逻辑，首先我们需要回归基础，准确理解这两个核心参数的概念。

**linger.ms**

生产者会将两次请求传输之间到达的所有记录组合成单一的批处理请求。这种攒批行为通常在记录到达速率超过发送速率的高负载场景下自然发生，但在负载适中时，客户端也可通过配置 linger.ms 引入少量的“人为延迟”来主动减少请求数量。其行为逻辑类似于 TCP 协议中的 Nagle 算法：生产者不再立即发送每一条到达的记录，而是等待一段指定的时间以聚合更多后续记录。该设置定义了批处理的时间上限，发送行为遵循“先满足者优先”原则——一旦分区积累的数据量达到 batch.size，无论 linger.ms 是否到期，批次都会立即发送；反之，若数据量不足，生产者将“逗留”指定时长以等待更多记录。在 Apache Kafka 4.0 中，该参数的默认值已从 0ms 调整为 5ms，其依据在于更大批次带来的效率增益通常足以抵消引入的等待时间，从而实现持平甚至更低的整体生产者延迟。

**batch.size**

当多条记录需发往同一分区时，生产者会将这些记录聚合为批次（Batch）以减少网络请求频率，从而优化客户端与服务端的 I/O 性能。batch.size 参数定义了该批次的默认容量上限（以字节为单位），超过该阈值的单条记录将不被纳入批处理逻辑。发往 Broker 的单个请求通常包含多个批次，分别对应不同的分区。配置过小的 batch.size 会限制批处理的发生频率并可能降低吞吐量（设置为 0 将完全禁用批处理）；而过大的配置则可能因生产者总是基于此阈值预分配缓冲区而导致内存资源的轻微浪费。该设置确立了发送行为的空间上限：若当前分区积累的数据量未达到此阈值，生产者将依据 linger.ms（默认为 5ms）的设定进行等待；发送触发逻辑遵循“先满足者优先（Whichever happens first）”原则，即一旦数据量填满缓冲区或等待时间耗尽，批次即会被发送。需要注意的是，Broker 端的背压可能导致实际的有效等待时间超过配置值。

通过对两个维度的拆解，我们可以清晰地看到 linger.ms 和 batch.size 的协同工作模式：

* 它们共同决定了 RecordBatch（批次）的大小和 ProduceRequest（请求）的发送时机。
* linger.ms和batch.size参数值较大 -> RecordBatch 和 ProduceRequest 批处理效果越好 -> Kafka 服务器需要处理的 RPC 数量更少 -> Kafka 服务端 CPU 消耗越低。
* 副作用：客户端在批处理上花费的时间增加，从而导致客户端的发送延迟变高。

**这引出了一个关键的性能权衡问题：**

“在服务端 CPU 资源充足的前提下，为了追求极致的低延迟。是否应当尽可能最小化 linger.ms 和 batch.size？”

基于直觉的推断，答案似乎是肯定的。然而，Kafka 4.0 的官方文档指出了相反的结论：

“Apache Kafka 4.0 将默认值从 0 调整为 5。尽管增加了人为的等待时间，但更大批次带来的处理效率提升，通常会导致相似甚至更低的生产者延迟。”

inger.ms=0 代表即时发送，为什么在延迟的表现上反而不如“先等待 5ms”？

**Kafka 服务端与客户端交互的底层规则**

要透彻理解这一反直觉的性能表现，我们不能仅停留在客户端配置的表面，而必须深入 Apache Kafka 网络协议的底层。延迟的产生，本质上源于客户端发送策略与服务端处理模型之间的交互机制。为了探究其根源，我们需要分别从服务端和客户端两个维度，解析这套底层规则的运作逻辑。

**1. 服务端视角：严格按序的“串行”模式**

Kafka 的网络协议在设计上与 HTTP 1.x 颇为相似，它采用的是一种严格的顺序且串行的工作模式。这是理解所有延迟问题的基石：

* **顺序性（Sequential）：**对于来自同一个 TCP 连接的请求，服务端必须严格按照接收到的顺序进行处理，并按同样的顺序返回响应。
* **串行性（Serial）：**服务端只有在完全处理完当前请求并发送响应后，才会开始处理下一个请求。即便客户端并发发送 N 个 ProduceRequest，服务端也会严格执行‘One-by-One’策略：必须等到前一个请求的数据完成所有 ISR 副本同步并返回响应后，才会开始处理下一个请求。

**这意味着：** 哪怕客户端一股脑地并发发送了 N 个 ProduceRequest，服务端也不会并行处理。如果前一个请求因为 ISR 同步卡顿了，后续的所有请求都只能在服务端排队等候。

**2. 客户端视角：化解拥堵的“Batch”原理**

在客户端侧，Producer 的批处理主要包含两个核心模块：RecordAccumulator 和 Sender，分别对应 RecordBatch 和 ProduceRequest。

* **RecordAccumulator：**负责将 RecordBatch 进行批处理。KafkaProducer#send 将记录放入RecordAccumulator 进行批处理。当分区内的ProduceBatch 数据超过 batch.size 时，它会切换到下一个分区并创建一个新的 ProduceBatch 进行批处理。
* **Sender：**负责维护与服务器节点的连接并分批发送数据。它会基于节点从 RecordAccumulator 中排干就绪分区的数据，将它们打包成 ProduceRequest 并发送。排干需要同时满足以下条件：

  a.连接上的在途请求数量小于max.in.flight.requests.per.connection=5

  b.对应节点的任何 ProduceBatch 超过 linger.ms 或超过 batch.size

**场景推演：0ms 与 5ms 的性能对比**

基于上述原理，我们需要进一步评估该机制在实际场景中的表现。当客户端配置 linger.ms=0 以执行即时发送策略，而服务端受限于串行处理模型时，供需两侧的处理节奏将产生错配。为了准确判断这种错配究竟是降低了延迟还是引发了排队积压，仅凭定性分析不足以说明问题。接下来，我们将构建一个模型，通过场景化的定量推演，计算不同配置下的具体延迟数据。

**场景假设：**

* 部署一个单节点集群，创建一个包含 10 个分区的 Topic。
* **客户端：**单客户端，发送速率 1000 条记录/秒，记录大小 1KB。
* **服务端：**处理一个ProduceRequest 耗时 5ms。
* **对比组：**

+ 配置A：linger.ms=0，batch.size=16KB（Apache Kafka 4.0 之前的默认配置）
+ 配置 B：linger.ms=5，其余不变（4.0 新版默认）

**推演 A：当 linger.ms = 0**

1. **1,000 records/s**意味着每 1ms 调用一次 KafkaProducer#send；
2. 由于 linger.ms=0，前 5 条记录会立即转换为 5 个 ProduceRequest，分别在时间戳 T=0ms，T=0.1ms， ...，T=0.4ms 发送。
3. Apache Kafka 顺序且串行地处理这 5 个 ProduceRequest：  
   **a.T=5ms：**Apache Kafka 完成第 1 个 ProduceRequest 的请求，返回响应，并开始处理下一个 ProduceRequest；  
   **b.T=10ms：**第 2 个 ProduceRequest 处理完毕，开始处理下一个；  
   c.以此类推，第 5 个 ProduceRequest 在 T=25ms 时处理完毕。
4. **T=5ms：**客户端收到第 1 个 ProduceRequest 的响应，满足 inflight.request < 5 的条件，从 RecordAccumulator 排干数据。此时，内存中已积累了 （5 - 0.4） / 1 ～= 4K 的数据，这些数据将被放入一个 ProduceRequest 中，Sender 将其打包成第 6 个请求发出。

   **a.T=30ms：**Apache Kafka 在 T=25ms 处理完第 5 个请求后，接着处理第 6 个请求，并在 T=30ms 返回响应。
5. **T=10ms：**同样地，收到第 2 个 ProduceRequest 的响应后，客户端积累了 （10 - 5） / 1 = 5K 的数据并发送给 Broker。Apache Kafka 在 T=35ms 返回响应。
6. 以此类推，后续的 ProduceRequest 都会在 T1 时刻积累 5K 数据并发送给 Broker，Broker 会在 T1 + 25ms 响应请求。**平均生产延迟为 5ms / 2 + 25ms = 27.5ms。**（5ms / 2 是平均批处理时间）。

**推演 B：当 linger.ms = 5**

1. **T=5ms：**由于 linger.ms=5，客户端会先积攒数据直到 5ms，然后发出第一个 ProduceRequest。服务端会在 T=10ms 时对该请求做出响应。
2. **T=10ms：**由于 linger.ms=5，客户端会继续积攒新数据达 5ms，随后发出第二个 ProduceRequest。服务端会在 T=15ms 时做出响应。
3. 以此类推： 后续的请求都会在 T1 时刻攒够 5K 数据后发往 Broker，Broker 会在 T1 + 5ms 时做出响应。此时的**平均生产延迟计算如下：5ms / 2 + 5ms = 7.5ms**（注：5ms / 2 代表平均攒批的时间）。

在这个假设场景中，虽然我们将 linger.ms 从 0 ms 增加到 5 ms，但平均生产延迟反而从 27.5 ms 降到了 7.5 ms。由此可见，“linger.ms 越小，延迟越低”这一说法并不绝对成立。

**linger.ms 与 batch.size 配置最佳实践**

通过对比 linger.ms 为 0ms 和 5ms 的情况，我们可以得出结论：客户端的主动批处理，将 在途请求控制在 1 及以内，要比快速把请求发出然后在网络层排队，更能降低生产延迟。

**那么如何在千变万化的生产环境中，精准设定这两个参数的阈值？**

我们需要一套科学的计算公式，根据服务端的实际处理能力，倒推客户端的最佳配置。以下是针对最小化生产延迟的定向配置建议：

* **linger.ms >= 服务端处理耗时。**

如果 linger.ms 小于网络耗时和服务端的处理时间，根据 Kafka 网络协议的串行处理模式，发出的 ProduceRequests就会在网络层产生积压。这违背了我们前面提到的“将网络在途请求数控制在 1 及以内”的原则。

* **batch.size >= （单个客户端最大写入吞吐量） \* （linger.ms / 1000） / （Broker 数量）。**

如果 batch.size 未设置为大于或等于此值，则意味着在达到 linger.ms 之前，由于 ProduceBatch 超过 batch.size，将会被迫提前发送请求。同样，这些 ProduceRequest 无法及时处理，将在网络中排队，违反了 “将网络在途请求数控制在 1 及以内”的原则。

* **建议将 batch.size 设置得尽可能大（例如 256K）：**

linger.ms 是基于服务端的平均生产延迟来设定的。一旦服务端出现性能抖动（Jitter），更大的 batch.size允许我们在单个 RecordBatch 中积攒更多数据，从而避免因为拆分成多个小请求发送而导致整体延迟升高。

以单节点集群为例，假设服务器处理一个 ProduceRequest需要5ms。那么我们需要将 linger.ms 设置为至少 5ms。如果我们预期单个生产者的发送速度能达到10MBps，那么 batch.size 应设置为至少10 \* 1024 \* （5 / 1000） = 51.2K。

**从“客户端攒批”走向“服务端流水线”**

Apache Kafka 4.0 对默认值的调整，验证了一个核心的技术共识：在处理大规模数据流时，适度的批处理是平衡吞吐与延迟的有效手段。这是一种基于客户端视角的成熟优化策略。

然而，性能优化的路径不止一条。既然瓶颈在于服务端的“串行处理”，那么除了一味调整客户端参数外，我们是否可以从服务端本身寻求突破？正是基于这一思考，作为云原生 Kafka 的探索者，AutoMQ 尝试从服务端视角寻找新的突破：在完全兼容 Kafka 协议语义的前提下，AutoMQ 引入了“Pipeline（流水线）机制”。这一机制并非改变协议本身，而是优化了服务端的模型，使得在保证顺序性的同时，能够充分利用云原生存储的并发能力，将 ProduceRequest 的处理效率提升了 5 倍。

**这意味着什么？让我们回到之前的推演场景：**

即便在 linger.ms=0 导致多个在途请求积压的情况下，AutoMQ 的流水线机制允许服务端同时处理这些请求，显著降低了排队延迟：

* **Apache Kafka：**由于串行排队，平均延迟达**27.5ms**。
* **AutoMQ：**凭借流水线机制，平均延迟降至 **7.5ms**。

因此，当使用 AutoMQ 作为服务端时，你可以享受服务端处理效率的 5 倍提升，客户端不再需要通过长时间的“逗留”来迁就服务端，从而获得更低的延迟体验。你可以将参数配置为原建议值的 1/5，linger.ms 的配置策略会与 Apache Kafka 略有不同：

* linger.ms >= (服务端处理耗时 / 5）
* batch.size >= (单个客户端最大写入吞吐量) \* (linger.ms / 1000) / (Broker 数量)

（注：同样建议在内存允许范围内，将 batch.size 尽可能调大，如 256K）

这种配置上的差异，揭示了性能优化视角的转变：要做到性能调优，不能仅依赖于客户端的适配。

AutoMQ 通过架构层面的创新实践，让用户无需在“低延迟”和“高吞吐”之间做艰难的权衡，而是以更低的门槛实现了两者的兼得。技术总是在不断演进的。从参数调优走向架构演进，不仅是 AutoMQ 的选择，也是云原生时代消息中间件发展的方向。

**结语**

感谢您读到这里。

本文回顾了Apache Kafka 4.0 中 linger.ms 与 batch.size 参数的配置，指出了在传统串行网络模型下，客户端进行性能调优时所面临的“延迟与吞吐”权衡难题。随后，我们深入解析了 AutoMQ 的 Pipeline 机制，它通过服务端 I/O 模型的重构，解除了顺序处理与串行执行的强绑定。

Pipeline 机制是 AutoMQ 云原生架构的核心特性之一，无需依赖繁琐的客户端参数调整，即可在保证数据严格顺序的前提下，实现 5 倍于传统架构的处理效率。结合对云原生存储的深度适配，AutoMQ 致力于通过底层架构的演进，助力企业以更简的运维构建极致性能的流数据平台。

感谢您的阅读，我们下篇文章再见。

💡 开源 AutoMQ 正在 Github Trending 中，欢迎关注：

https://go.automq.com/github?utm\_source=wechat\_automq

👇 AutoMQ 商业版当前提供 2 周免费试用，欢迎试用：

https://go.automq.com/home?utm\_source=wechat\_automq

欢迎底部扫码加入 AutoMQ 社区群，与开发者共同探讨，第一时间获取产品动态。

**往期推荐**

[如何选择合适的 Diskless Kafka](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247489280&idx=1&sn=f44e4430692b5bae95ab6112c4e89c44&scene=21#wechat_redirect)

[2025-11-28](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247489280&idx=1&sn=f44e4430692b5bae95ab6112c4e89c44&scene=21#wechat_redirect)

[当 Kafka 架构显露“疲态”：共享存储领域正迎来创新变革](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247489049&idx=1&sn=dd0be3da18089f58b2548bacaa420500&scene=21#wechat_redirect)

[2025-10-17](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247489049&idx=1&sn=dd0be3da18089f58b2548bacaa420500&scene=21#wechat_redirect)

[技术干货｜Kafka 如何实现零停机迁移](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247488538&idx=1&sn=e5306968630ba3348fcd28835396d4d0&scene=21#wechat_redirect)

[2025-08-07](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247488538&idx=1&sn=e5306968630ba3348fcd28835396d4d0&scene=21#wechat_redirect)

[技术干货｜为什么越来越多企业放弃 Flink/Spark，用 AutoMQ 替代传统 ETL？](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247488409&idx=1&sn=b2bc366bf8d66434880dfa8e43eceda0&scene=21#wechat_redirect)

[2025-07-28](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247488409&idx=1&sn=b2bc366bf8d66434880dfa8e43eceda0&scene=21#wechat_redirect)

END

**关于我们**

我们是来自 Apache RocketMQ 和 Linux LVS 项目的核心团队，曾经见证并应对过消息队列基础设施在大型互联网公司和云计算公司的挑战。现在我们基于对象存储优先、存算分离、多云原生等技术理念，重新设计并实现了 Apache Kafka 和 Apache RocketMQ，带来高达 10 倍的成本优势和百倍的弹性效率提升。

🌟 GitHub 地址：https://github.com/AutoMQ/automq

💻 官网：https://www.automq.com?utm\_source=wechat

👀 B站：AutoMQ官方账号

🔍 视频号：AutoMQ

👉🏻 **扫二维码**

**加入我们的社区群**