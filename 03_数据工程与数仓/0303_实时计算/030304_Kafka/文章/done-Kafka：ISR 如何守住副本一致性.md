> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030304_Kafka/030304_核心知识点/Kafka副本一致性与ISR边界|Kafka副本一致性与ISR边界]]
---
title: Kafka：ISR 如何守住副本一致性
author: 编程宇航员
date: 干货干货
url: https://mp.weixin.qq.com/s?__biz=MzAwNjExNTU1MQ==&mid=2247489664&idx=1&sn=18514d22068e94cb44ab1c3976c4a3e3&chksm=9ad2a5de34e35bd7c60afc2191a8d9ebe596104cc6275a8c8acbf93cced5e439cbe79b402f48&mpshare=1&scene=24&srcid=05308n69UjtaFYr9A53JpjDe&sharer_shareinfo=33381e99a05bbede0a9c74b300ef5a87&sharer_shareinfo_first=33381e99a05bbede0a9c74b300ef5a87#rd
---

一条消息写进 Kafka 后，生产者拿到成功响应，并不代表所有副本都已经落盘。更准确地说，Kafka 只承诺这条消息已经到达某个「足够安全」的位置。这个位置由 Leader、Follower、ISR 和 High Watermark 一起决定。

线上排查 Kafka 丢消息、重复消费、切主后数据回退时，很多人会先看副本数和 `acks=all`。这两个参数重要，但还不够。真正决定一条消息能不能对消费者可见、能不能在 Leader 故障后保留下来的是另一组状态：ISR 里还有哪些副本，High Watermark 推到了哪里，新的 Leader 是否能用 Leader Epoch 判断旧日志的分叉点。



这篇只看一个 Partition 内部的复制链路。Kafka 的副本一致性没有走多数派日志复制那套路径，它让 Leader 处理所有读写，Follower 像普通消费者一样从 Leader 拉取日志。ISR 是 Leader 认为「跟得上」的一组副本，High Watermark 是消费者最多能读到的位置，Leader Epoch 则在切主后帮助副本判断哪些尾部日志要截掉。

## Leader 为什么要收住可见位置

先看一次最常见的写入：

```
topic: orderspartition: 3replication.factor: 3min.insync.replicas: 2producer.acks: all
```

Partition 3 有 3 个副本，当前 Leader 在 Broker 1，Follower 在 Broker 2 和 Broker 3。生产者把消息发给 Broker 1。Leader 把消息 append 到本地 log，然后等待 ISR 里的副本复制到足够位置，再给生产者返回成功。

这里容易误判的是 `replication.factor=3`。副本数是存储冗余的上限，每次写入需要同步到多少副本，主要看 `acks`、`min.insync.replicas` 和 ISR 当前成员。

如果 `acks=1`，Leader 本地 append 成功就可以回复。Follower 还没拉到这条消息时，Leader 宕机，且新 Leader 没有这段日志，生产者已经收到成功的消息可能丢失。

如果 `acks=all`，Leader 要等 ISR 中足够多副本确认。`min.insync.replicas=2` 时，至少 Leader 加一个 ISR Follower 到位，生产者才能拿到成功。这个组合常被用来提高耐故障能力，但前提是禁用不干净选主，且 ISR 至少还能保留 2 个副本。

如果 ISR 只剩 Leader 一个副本，`min.insync.replicas=2` 会让写入失败，生产者看到 `NOT_ENOUGH_REPLICAS` 或 `NOT_ENOUGH_REPLICAS_AFTER_APPEND`。Kafka 在这里拒绝把一条只有单副本保护的消息包装成强成功。

## Follower 拉取为什么能做复制

Kafka 的复制方向看起来有点反直觉：Leader 不主动把日志推给 Follower。Follower 持续向 Leader 发 Fetch 请求，携带自己下一条要拉的 offset。Leader 返回从这个 offset 开始的一批消息，Follower append 到本地 log 后，再发下一次 Fetch。

这个模型让复制路径复用消费路径的很多机制。Follower 不需要 Leader 为每个副本维护一套推送通道，Leader 只处理 Fetch 请求，并根据每个 Follower 最近一次拉取的位置更新副本状态。

可以把 Follower 的状态粗略看成几项：

* 它最近一次向 Leader 发起 Fetch 的时间。
* 它已经复制到本地的 log end offset。
* 它是否还在 ISR 里。
* 它认为当前 Leader 的 epoch 是多少。

Leader 判断一个 Follower 是否「跟得上」，主要看它有没有持续拉取、有没有在 `replica.lag.time.max.ms` 这类阈值内保持活跃。长时间没有 Fetch，或者拉取进度停住，Follower 会被踢出 ISR。被踢出去的副本仍然可以继续追数据，但写入确认不再等它。

这个设计把延迟副本和正常写入隔开了。慢副本不会拖住所有写入，代价是 ISR 变小后容错空间也变小。`min.insync.replicas=2` 的集群里，ISR 从 3 缩到 2 还能写；缩到 1 后，`acks=all` 写入会被拒绝。

## High Watermark 到底挡住了什么

Leader 的 log end offset 代表本地已经 append 到哪里。High Watermark 表示一段日志中「已提交」的位置。消费者只能读到 High Watermark 之前的数据，后面的尾部日志即使已经在 Leader 本地，也暂时不可见。



假设当前 ISR 有 3 个副本：

```
Leader LEO      = 108Follower A LEO  = 108Follower B LEO  = 105High Watermark  = 105
```

Leader 和 Follower A 已经到 108，Follower B 只到 105。只要 B 仍在 ISR 里，High Watermark 就不能越过 105。消费者最多读到 offset 104。offset 105、106、107 还在 Leader 日志尾部，但它们没有被所有 ISR 副本覆盖。

等 Follower B 拉到 108 后，Leader 才能把 High Watermark 推到 108。生产者端的 `acks=all` 和消费者端的可见性并不完全等价，但它们都围绕同一件事工作：不要让还没获得足够副本保护的尾部日志提前暴露。

这里有个细节很影响排查。High Watermark 推进依赖 Follower 的 Fetch 请求。Follower 拉到某个 offset 后，Leader 要从下一次 Fetch 中知道它的进度，才能更新本地对这个 Follower 的复制位置。低流量 Partition 上，HW 推进看起来可能有一点滞后，不一定是磁盘慢，也可能是复制状态还没被下一轮 Fetch 刷新。

High Watermark 保护的是消费者读到的数据。Leader 本地比 HW 更靠后的日志，属于未提交尾部。切主时这些尾部可能保留，也可能被截断，取决于新 Leader 和旧 Leader 的日志关系。

## ISR 为什么会收缩和扩张

ISR 不是固定副本列表。它会随着复制进度变化。

Follower 正常拉取时，Leader 会把它视为 in-sync。Follower 卡住太久，Leader 会把它从 ISR 移除。Follower 追上 Leader 的已提交位置后，可以重新加入 ISR。

这套机制解决了一个直接问题：如果 Kafka 永远等待所有副本，任意一个慢副本都会把 Partition 写入拖慢；如果 Kafka 完全不管慢副本，Leader 单副本成功又会扩大故障丢数据窗口。ISR 给了一个中间层：只等待当前健康的同步副本，同时用 `min.insync.replicas` 给健康副本数量设置下限。

生产上更麻烦的状态是 ISR 频繁抖动。比如磁盘偶发长尾、网络微抖、GC 停顿，都会让 Follower 周期性退出 ISR。生产者看到的可能是一阵阵写入失败或延迟尖刺，消费者看到的可能是高水位推进变慢。

可以盯几类信号：

* `UnderReplicatedPartitions` 持续大于 0，说明有副本不在 ISR 或追不上。
* `IsrShrinksPerSec` 和 `IsrExpandsPerSec` 同时有波动，说明 ISR 在来回变化。
* Follower Fetch 延迟升高，通常会传导到 HW 推进。
* `min.insync.replicas` 接近当前 ISR 大小时，写入可用性会变差。

这些指标要结合业务写入语义看。日志采集类业务可能能接受短时间 `acks=1`，交易、订单、账务事件通常会选择 `acks=all`、`min.insync.replicas>=2` 和禁用不干净选主。

## Leader Epoch 解决哪种切主麻烦

Kafka 早期副本恢复主要依赖 offset 和 High Watermark。问题在切主后会变复杂：旧 Leader 可能写入过一些未提交尾部，新 Leader 没有这些记录；旧 Leader 恢复后，如果只看 offset，很难判断自己的日志从哪里开始和新 Leader 分叉。

Leader Epoch 给每一任 Leader 一个递增编号。每个副本会记录「从哪个 offset 开始由哪个 epoch 的 Leader 写入」。切主后，Follower 可以向当前 Leader 查询 epoch 对应的末端位置，然后把本地不属于当前合法历史的尾部截掉。



看一个简化过程：

```
epoch 5: Broker 1 是 Leader，写到 offset 120epoch 6: Broker 2 被选为 Leader，它只有 offset 118Broker 1 恢复后，发现当前 Leader epoch 是 6Broker 1 查询 epoch 边界，把本地 118 之后的旧尾部截掉
```

offset 118 之后那段日志不一定对消费者可见。它可能只是旧 Leader 本地 append 成功，还没被 ISR 复制完成。Leader Epoch 的价值在这里：副本恢复时不再只凭「我本地有更大的 offset」判断自己更新，而是看这段日志属于哪一任 Leader，以及当前集群承认的历史到哪里。

这也是 Kafka 复制和共识系统容易混淆的地方。Kafka 没有让所有副本通过投票提交每条日志。它通过 Leader 串行写入、ISR 约束确认、HW 控制可见性、Leader Epoch 修复切主分叉，把一致性问题拆成几段处理。

## 故障切换时 Kafka 会丢哪一段

Leader 故障后，控制器会在合适副本里选择新的 Leader。正常情况下，新 Leader 从 ISR 中选出。因为 ISR 副本已经跟上已提交日志，消费者读到的 HW 之前数据应该能保住。

真正有风险的是两个地方。

一类风险来自未提交尾部。旧 Leader 本地已经写入、生产者还没拿到 `acks=all` 成功，或者只用了 `acks=1` 提前拿到成功。切主后，新 Leader 没有这段尾部，旧 Leader 恢复时会按 Leader Epoch 截断。这些记录会消失。

另一类风险来自不干净选主。如果 `unclean.leader.election.enable=true`，ISR 全部不可用时，Kafka 可以从非 ISR 副本中选 Leader。这个副本可能缺少已经提交过的数据。换来的是 Partition 尽快恢复服务，代价是数据丢失风险明显上升。



`acks=all`、`min.insync.replicas=2`、禁用不干净选主这组配置的目标，是让生产者成功返回的数据落在「新 Leader 大概率能继承」的范围里。它不能消除所有重复消费和重试问题，也不能替代业务幂等，但能显著缩小已确认消息丢失的窗口。

如果业务还启用了幂等生产者或事务，Kafka 会进一步处理生产者重试带来的重复写入和跨 Partition 原子性。那是生产者协议和事务协调器的范围，ISR 主要负责单个 Partition 的副本复制边界。

## 为什么 ISR 不能只看副本数量

很多事故复盘会写「副本数是 3，所以应该安全」。这个判断少了两个条件。

第一，副本是否在 ISR 里。副本存在不等于副本可用。落后很久的副本只是磁盘上有一份旧数据，它不能为当前写入提供确认，也不能成为安全的新 Leader。

第二，生产者是否等待了 ISR 确认。`replication.factor=3` 加 `acks=1`，仍然可能让生产者在 Follower 没复制到消息时收到成功。这个配置偏向吞吐和可用性，不适合对确认语义敏感的事件。

实际配置要先问写入成功代表什么：

* 如果成功只表示 Leader 收到了，`acks=1` 能降低等待时间，但 Leader 故障时可能丢已确认消息。
* 如果成功表示至少两个同步副本有保护，常见组合是 `acks=all` 和 `min.insync.replicas=2`。
* 如果系统宁愿停写也不能承认有风险的数据，禁用不干净选主，并监控 ISR 缩小后的写入失败。
* 如果业务能重放、去重、补偿，可以把 Kafka 的确认语义和业务幂等放在一起评估。

参数没有脱离业务的最佳值。Kafka 把选择权暴露出来，是因为日志采集、用户行为、订单状态变更对「成功」的理解差异很大。

## 一条消息从写入到可见的完整路径

把前面的机制合起来，一条 `acks=all` 消息大致经历这些状态。

生产者把消息发送给当前 Leader。Leader append 到本地 log，更新本地 log end offset。ISR 里的 Follower 通过 Fetch 拉取这条消息，append 到自己的本地 log。Leader 根据 Follower 后续 Fetch 请求更新复制进度。当 ISR 中足够副本到达该 offset，Leader 可以满足生产者确认条件；当所有 ISR 副本的复制位置推进后，High Watermark 随之推进。消费者 Fetch 时，Leader 只返回 HW 之前的数据。

如果 Leader 在中途故障，控制器选出新的 Leader。新 Leader 继承已提交位置。旧 Leader 恢复后，根据 Leader Epoch 和当前 Leader 校准日志，截掉不属于当前历史的尾部，然后作为 Follower 继续拉取。



排查 Kafka 副本一致性问题时，可以按这个顺序看：

```
生产者确认语义：acks、retries、幂等、事务Partition 复制状态：Leader、ISR、UnderReplicatedPartitions可见性边界：High Watermark、Log End Offset、Follower lag切主历史：Leader Epoch、unclean leader election、日志截断业务结果：重复、丢失、乱序、消费位点回退
```

这个顺序能避免一个常见误区：直接拿消费者是否读到消息反推生产者是否成功。消费者读不到，可能是 HW 没推进；生产者成功了，也要看成功是 `acks=1` 还是 `acks=all`；切主后日志回退，还要看新 Leader 是否来自 ISR，以及旧 Leader 恢复时截掉了哪段尾部。

Kafka 的副本一致性靠几条边界共同工作。ISR 决定哪些副本参与写入确认，High Watermark 决定消费者能读到哪里，Leader Epoch 决定切主后日志怎样收敛。副本数只是容量和冗余的起点，真正影响一致性的是这些边界在故障和延迟下怎么移动。

## 参考资料

* Apache Kafka Documentation: Design，https://kafka.apache.org/documentation/#design
* Apache Kafka Documentation: Topic Configs，https://kafka.apache.org/documentation/#topicconfigs
* Apache Kafka Documentation: Broker Configs，https://kafka.apache.org/documentation/#brokerconfigs
* Apache Kafka Protocol: Fetch API，https://kafka.apache.org/protocol#The\_Messages\_Fetch
* Apache Kafka Source: Partition.scala，https://github.com/apache/kafka/blob/trunk/core/src/main/scala/kafka/cluster/Partition.scala
* Apache Kafka Source: LeaderEpochFileCache.scala，https://github.com/apache/kafka/blob/trunk/storage/src/main/java/org/apache/kafka/storage/internals/epoch/LeaderEpochFileCache.java

- END -