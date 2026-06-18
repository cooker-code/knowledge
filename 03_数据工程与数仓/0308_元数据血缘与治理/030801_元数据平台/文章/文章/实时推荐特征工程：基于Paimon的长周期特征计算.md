---
title: 实时推荐特征工程：基于Paimon的长周期特征计算
author: 熊大数据
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247487400&idx=1&sn=614518556926c9073635a6f9b8ec40c6&chksm=e9debc70ba79b5dc510b9f690d2412fc66f98bc51c36ac44171598f3db0e7f3493df134557f8&mpshare=1&scene=24&srcid=12237jI0N5P34XQlLPomEvRe&sharer_shareinfo=3bc2f2b32b89368fac9f0371d544d8b0&sharer_shareinfo_first=3bc2f2b32b89368fac9f0371d544d8b0#rd
---

我是大熊！某大厂数据负责人

**拒绝AI无脑文**

**每周原创，主要分享大厂实战经验**

《熊大原创》第79篇

关注我，全是百度不到的数据经验

### 在短视频推荐系统中，为了追求极致的 “千人千面” 体验，推荐引擎对用户行为特征的实时性与丰富度提出了严苛要求。系统需要能够**亚秒级**感知用户兴趣的微小迁移，并提供**毫秒级**的在线特征推理服务。

### 

1分钟看图掌握核心观点👇

长周期的特征计算一直存在个核心矛盾：**即时决策需求**与**长周期特征计算延迟**之间的矛盾。

* **即时决策：要求特征服务在推荐召回/粗排阶段，能够在 5ms-10ms 内返回特征向量。**
* **长周期计算：用户的兴趣往往蕴含在长周期的行为序列中（如7天、30天），这涉及海量状态的实时维护与聚合。**

之前团队复用 “Flink + Kafka + HBase/Redis” 架构在处理此类场景时，面临着“状态爆炸”与“架构割裂”的挑战，导致维护特别难受

**Lambda架构的局限性**

状态爆炸：

* 在长窗口（如 7 天）场景下，全量缓存活跃用户的行为明细会导致 Flink State膨胀至 **TB 级**。
* 频繁的 Checkpoint 超时与 GC 停顿，严重影响作业稳定性。

**一致性难以保证：**

* 复杂的 Retract（撤回）流处理逻辑（如用户取消点赞），需要在应用层手动维护状态版本，极易导致特征“脏读”或“漂移”。

**存储与计算耦合：**

* 为了规避大状态，往往需要引入 HBase 等外部存储进行 Point Lookup，这导致了严重的 **I/O 放大**，且外部存储的高并发读写成为系统瓶颈。

我们引入Paimon 的核心思路是 “**状态下沉至存储层**”，通过主键表和聚合表的组合，彻底解决状态膨胀问题

推荐服务的特征体系

我们的核心特征体系主要分为三类

**1.即时行为特征**

**捕捉用户瞬间兴趣强度，****例：近 1min/5min 的点击率 (CTR)、完播率、即时互动度。**

**2.长周期序列表征**

刻画用户长期兴趣演化路径及周期性偏好，例：近 7 天 TopN 偏好类目、近 200 次交互序列。

**3.动态交互特征**

**用户与视频/作者的实时交互状态，例：实时视频完播率水位、作者粉丝互动亲密度。**

特征口径示例

基于Paimin的新架构

“存储即状态”

**数据链路**

用户行为数据接入（观看、点击、滑动、点赞等）→ 关联用户 / 视频维表 → 实时特征聚合（滑动窗口 / 滚动窗口）→ 特征向量格式化 → 写入特征平台 KV 存储 → 推荐模型调用特征进行实时推荐。

**数据流向**

**存算分离的状态管理**

利用 Paimon 的主键表（Primary Key Table）和聚合表（Aggregation Table），将 TB 级状态存储在分布式文件系统（HDFS/S3）上，Flink 仅需维护轻量级索引或缓冲区，彻底解决 State Backend 压力。

**原生ChangeLog支持**

内置的 Merge Engine 能够自动处理 Upsert/Delete 消息，无需 Flink 手动构建 Retract 流，天然支持特征的修正与回溯。

**高并发点查**

结合 Bucket 索引与列式存储（Parquet/ORC），Paimon 支持毫秒级的 Point Query，可直接作为 Feature Store 服务于在线推荐，消除了数据同步到 KV Store 的延迟与成本。

**端到端数据闭环**

Raw Data -> Feature Engineering -> Feature Serving的全链路湖仓一体化，数据流转延迟可控制在 分钟级，同时保证了离线训练与在线推理的数据一致性。

### 

### **实战案例**

### **5分钟特征+长序列特征**

### 

整体架构演进为 **4 层湖仓一体模型**，全链路数据驻留于 Paimon，无需依赖外部 KV，实现了特征计算的极致简化与高效闭环。

**### 

案例1：5分钟用户域特征

**业务需求**

推荐模型需要实时感知用户最近5分钟的行为变化，包括：

* 点击率（CTR）：5分钟内点击次数 / 曝光次数
* 平均观看时长：5分钟内所有视频的平均观看时长
* 互动率：5分钟内点赞、评论、分享等互动行为占比
* 内容偏好分布：5分钟内观看视频的类别分布（Top5类别及占比）
* 行为强度：5分钟内行为总次数，用于判断用户活跃度

**Paimon表设计（核心字段）**

**Flink核心计算逻辑**

****为什么选择5分钟窗口？**

时效性与稳定性平衡

* 1分钟：过于敏感，用户短暂的行为波动会被放大，特征噪声大，容易导致推荐结果抖动
* 5分钟：既能捕捉用户兴趣的实时变化（如从游戏切换到美食），又能平滑短期波动，特征稳定性更好
* 10分钟：响应滞后，用户兴趣已经发生明显转变，但特征还未更新，影响推荐效果****

**状态管理成本**

* **5分钟窗口内，假设活跃用户平均产生20-30次行为，状态大小可控**
* **如果使用1分钟窗口，需要维护5个1分钟窗口的状态，状态管理复杂度提升**
* **5分钟窗口通过Paimon的增量更新机制，只需维护一个窗口的状态，成本更低**

**业务语义匹配**

* 短视频场景下，用户观看一个视频平均15-30秒，5分钟窗口可以包含10-20个视频的交互
* 这个数量足以反映用户的短期兴趣偏好，同时不会因为单个视频的异常行为影响整体特征

**案例2：**用户行为序列特征（长度200）**

**业务需求**

推荐模型需要用户最近200次行为序列，用于：

* Transformer模型输入：序列特征作为模型输入，捕捉用户兴趣演化路径
* 序列模式挖掘：识别用户行为模式（如：游戏→美食→游戏，存在周期性偏好
* 冷启动优化：新用户通过序列特征快速建立兴趣画像

****Paimon表设计****

**Flink核心逻辑**

序列维护策略：滑动窗口 + 增量更新

序列特征格式化输出

### ******常见的Q&A********

**Q1：为什么是200长度序列？**

****模型输入限制：模型通常支持512或1024的序列长度，200是一个平衡点

* 50-100：无法捕捉长期兴趣演化，用户行为模式识别不充分

* 500+：计算成本高，且历史行为对当前推荐的影响衰减严重
* 200长度：覆盖用户约30-60分钟的行为历史，既能反映短期兴趣变化，又能捕捉周期性偏好****

**存储成本考量**

* 假设每个序列项平均200字节（video\_id + category + tags + metadata）：200长度 × 200字节 = 40KB/用户
* 1亿活跃用户 × 40KB = 4TB存储（可接受）
* 如果增加到500长度，存储成本提升到10TB，且查询延迟增加

**Q2：增量更新 VS 全量重算**

**全量重算**：

* 每次新行为到来，重新计算整个200长度的序列
* 问题：计算量大，延迟高，状态管理复杂

****增量更新（**Paimon方案）**：

* 新行为到来：插入序列头部（sequence\_id=1），原有项sequence\_id+1
* 超过200长度：删除尾部项（sequence\_id=200）
* 优势：只需处理增量数据，计算量从O(200)降至O(1)，Paimon的Changelog机制自动处理更新，无需手动维护状态。支持行为修正（如取消点赞），通过UPDATE操作自动修正序列

**Q3：序列特征的时间衰减机制**

**用户行为序列中，不同时间点的行为重要性不同：**

* **0-5分钟：权重1.0，直接反映当前兴趣**
* **5-30分钟：权重0.5-0.8，反映兴趣演化**
* **30-60分钟：权重0.2-0.5，主要用于模式识别**

通过时间衰减权重，模型可以：

* 在Attention机制中，自动关注最近的高权重行为
* 在序列模式挖掘中，平衡短期和长期偏好

**Q4：冷启动与序列填充**

新用户或低活跃用户序列不足200时：

* **策略1：零填充：不足部分用特殊token（如<PAD>）填充**
* **策略2：类别填充：用用户注册时选择的兴趣类别填充**
* **策略3：全局热门：用平台热门内容填充，快速建立初始兴趣**

我们选择**策略2+3混合**：

* 前50%用用户注册兴趣填充（个性化）
* 后50%用全局热门填充（探索性）
* 随着行为积累，逐步替换填充内容

**END**

咨询备注“熊大1V1”，加群备注“熊大群”

让我知道你的来意

每周原创更新，主要分享大厂数据经验 

你永远百度不到的那种！

**最近精彩合集**

[你的DQC检测为什么毫无价值？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486986&idx=1&sn=1fb7e412f453260e56107d6e0ec98e61&scene=21#wechat_redirect)

[深度思考：数仓模型复用的底层逻辑？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247487108&idx=1&sn=7f278d8e2bacdc46eca7101104f52d7b&scene=21#wechat_redirect)

[如何重新定义LLM大模型的数据质量？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247487035&idx=1&sn=b8cf68bb620d39a93b82208932c5ad8e&scene=21#wechat_redirect)

[熊大深度思考：数据清洗到底在洗什么？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247487097&idx=1&sn=98485961a0202b325dd6e81db0b1413a&scene=21#wechat_redirect)

[戒掉灵感后，我稳定周更2篇原创](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486998&idx=1&sn=a1100fd9f3e5e8a6a83cc86e550fbf99&scene=21#wechat_redirect)

[我这样0成本提升Spark性能](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486940&idx=1&sn=9ddf27937d068148c84339577c73310f&scene=21#wechat_redirect)

[数据治理难的不是技术，而是“讲规矩”](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486915&idx=1&sn=a063894370aa9967ecd5f251d1bd8a1a&scene=21#wechat_redirect)

[财务数仓为基础的主数据全生命周期治理](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486893&idx=1&sn=cf50c8c9c655cd2f21ad71e444ac02b0&scene=21#wechat_redirect)

[BroadcastJoin广播了个寂寞](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486842&idx=1&sn=7ff2c26bae8aa27f472f04d8772bbc0e&scene=21#wechat_redirect)

[阿里2面，问到公共模型如何设计？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486712&idx=1&sn=deadce285d50f624a0092c9ca640d0f3&scene=21#wechat_redirect)

[1个月被叫醒20次，字节的欢迎仪式？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486640&idx=1&sn=a979b3d88c7fe379f35dbf8b659a0965&scene=21#wechat_redirect)

[如何高效回填历史的分区数据？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486600&idx=1&sn=c22248c5aef50b718fc05b07a98e0787&scene=21#wechat_redirect)

[Cursor专坑基础薄弱的人](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486556&idx=1&sn=c1d130d4eb63a9237e0168fa46f05c78&scene=21#wechat_redirect)

[大数据无新技术可学](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486519&idx=1&sn=10b4d2806e87feb381e6002ee1ba8bc7&scene=21#wechat_redirect)

[怎么说服不懂技术的领导?](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486496&idx=1&sn=a1ff10cd60a8d5f521731c4d23bf17d7&scene=21#wechat_redirect)

[ABTest数据全链路建设](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486430&idx=1&sn=2355995ba1fa99c4adaeefc4afa2f123&scene=21#wechat_redirect)

[大厂数据优化：Null值的高阶处理](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486357&idx=1&sn=0fda23423ce8097ebe367614e1d4869d&scene=21#wechat_redirect)