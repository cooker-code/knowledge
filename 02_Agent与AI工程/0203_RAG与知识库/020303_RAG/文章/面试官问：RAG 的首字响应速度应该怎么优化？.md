---
title: 面试官问：RAG 的首字响应速度应该怎么优化？
author: 吴师兄学大模型
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489138&idx=1&sn=b15f1722d0d19f23fb4649c29659298b&chksm=c34e3c6d5733db7a5ad5d6516bdf5030e43daa49b14749dbfd0cf26d9ac5cecd71700b48dc8f&mpshare=1&scene=24&srcid=1125iOKVJ4wEeIel024TSiSi&sharer_shareinfo=8b32d2b23718e9c05a1ac276c96d7c2e&sharer_shareinfo_first=8b32d2b23718e9c05a1ac276c96d7c2e#rd
---

大家好，我是吴师兄。

在各种 RAG 面试题里，有一个问题非常考验“工程思维”：

> “你们的 RAG 首字延迟（TTFT）怎么优化？”

这个问题的难度在于，它跨越三层内容：

1. 模型接口层
2. 向量检索层
3. 系统架构层

如果只会回答“并发调用”“缓存 embedding”“加 GPU”，这种答法只会让面试官觉得：**“看过几篇文章，但没做过真系统。”**

而能把“哪里慢→为啥慢→怎么拆→怎么优先级”讲得有逻辑、有落地感，才是真正的加分项。

下面我们就按工程链路拆开说。

## unsetunset一、首字延迟到底卡在哪？unsetunset

RAG 的全链路可以拆成四步：

1. Embedding（OpenAI 或自建模型）
2. 向量检索（Milvus / Chroma / Faiss / PgVector）
3. Prompt 拼装
4. 大模型生成（LLM Completion / Streaming）

其中影响 TTFT（Time-to-First-Token）的主要瓶颈是：

* **Embedding API 等待时间**
* **向量检索耗时**
* **系统缺乏并发 / 缓存**

换句话说，**卡的并不在 LLM，而是在 LLM 之前的链路。**

优化 TTFT，本质就是“把 Embedding 和检索变快，把重复计算干掉，把链路做成流水线”。

## unsetunset二、Embedding 阶段：怎么把 OpenAI 的延迟压到最小？unsetunset

Embedding 是行业里“最容易被忽略的延迟来源”。

如果你用最朴素的方式，“来一条算一条”，那必然会慢。

工程落地的优化有三件事：

### 1. 批处理（Batch Embedding）——一次请求算多条

最关键的是：OpenAI 的 Embedding API 支持一次输入多个文本。

例如将 N 个 chunk：

```
["文本1", "文本2", "文本3", ...]
```

一次性扔进去算向量。

好处是：

* **减少网络往返延迟**
* **提高吞吐量**
* **减少 API request 限流风险**

注意 token 限制（8k 左右），按 token 切批即可。

在我们训练营的 RAG 工程项目里，开启批处理能直接把嵌入时间从“几百毫秒”降到“几十毫秒”。

### 2. 异步并发（asyncio）——让 CPU 不再发呆

单线程逻辑：

* 发请求
* 等待
* 发下一个请求
* 再等

CPU 大部分时间在“等”。

异步并发模型：

* 你等 API 的时候，CPU 去安排别的请求
* 整体吞吐可以提升 5~10 倍

但需要控制并发数量： 过高并发（比如 20+）会遇到 429 限流。

经验值：

* **5～10 个并发最稳**

### 3. 缓存（Embedding Cache）——把重复的工作彻底去掉

Embedding 最“浪费钱”的地方就是：重复调用。

现实里你会遇到：

* 用户各种用词相近的提问
* FAQ 类问题
* 编写 RAG 项目时自己不断调试

最佳策略：**把 query → vector 缓存在 Redis / KV 里。**

缓存命中率甚至能达到 30～50%。

对于语料库 embedding，要提前离线算好，这样查询时就不需要临时生成 embedding。

训练营里的实际项目中，把缓存引入后能把首字延迟直接砍掉 40% 以上。

## unsetunset三、向量检索阶段：如何让 Milvus / Faiss 几毫秒就返回？unsetunset

向量检索的速度差异非常大：

* 朴素暴力检索：几十毫秒～几百毫秒
* HNSW / IVF 索引：几毫秒级
* 加副本、分区、过滤：亚毫秒级

RAG 想快，要做到以下几点：

### 1. 建索引（HNSW / IVF）——别用暴力检索

**HNSW 是公认在“速度 + 精度”之间平衡最好的 ANN 索引。**

Milvus HNSW 参数：

* M：控制图连边数量
* efConstruction：控制建索引质量
* efSearch：控制搜索精度与速度

实际经验：

* M=16
* efConstruction=128
* efSearch=64

这是一个 "**稳**" 的组合。

HNSW 是靠增加“预建联结图”的方式减少搜索路径，所以对百万级向量性能非常好。

### 2. 分区 / 分片（Partition + Sharding）——让搜索范围更小

如果你把所有向量丢在同一个集合里，那系统必须“全库搜索”。

更优的做法是：

* 按“主题/时间/来源”分区
* 查询时只查对应分区

例如：

* 只查最近 30 天的文档
* 只查某部门文档
* 只查某业务线的知识库

能直接减少 50%～90% 的检索范围。

### 3. 连接池 + 批量查询——把网络往返次数砍掉

Milvus 支持：

* 一次查多个 query vector
* 多连接并发查询
* 多副本分摊查询负载

做业务时，如果你要查多个 chunk，就批量查：

```
[v1, v2, v3, …]
```

减少网络往返就是最快的优化。

### 4. GPU 加速（可选）

如果你的业务是：

* 高频查询（推荐、广告、电商搜索）
* 向量库千万级以上
* 对延迟要求苛刻

可以考虑 GPU 版本向量数据库。

但 GPU 方案成本高、运维复杂，只适合极端场景。

## unsetunset四、系统层优化：把整个流程做成“流水线”unsetunset

Embedding 变快、检索变快还不够。

真正的大幅降延迟，来自于：

* **异步流水线架构**
* **缓存体系**
* **负载均衡**

下面几件事非常关键：

### 1. 全链路异步化（Async Pipeline）

传统架构：

```
Embedding → 检索 → 拼Prompt → LLM
```

全链路异步后：

* embedding 等待时可以处理检索
* 检索等待时可以准备 prompt
* 多个用户请求不互相阻塞

你的 RAG 服务就变成：

* 更高 QPS
* 更低首字延迟
* 更充分利用 CPU / IO

训练营的 RAG 服务统一采用“嵌入 → 检索 → 生成”的异步流水线，TTFT 能降到“百毫秒级”。

### 2. 三层缓存体系（Embedding / Retrieval / Answer）

这一点是很多在线 RAG 系统一定会做的：

#### 第一层：Embedding 缓存

避免重复算向量。

#### 第二层：检索结果缓存

同样的 query，不需要每次都查向量库。

#### 第三层：答案缓存（FAQ）

如果答案固定，那直接返回，甚至不需要走 RAG。

这三层缓存能把：

* API 调用次数
* Milvus 查询次数
* LLM 调用次数

统统减少至少 30%～60%。

### 3. 多副本 + 多节点（水平扩展）

如果是高并发业务，可以：

* 开多个 Query Node
* 设置多个副本 replica
* LLM 多实例负载均衡

解决 QPS 需求。

## unsetunset五、总结：如何给面试官浓缩回答？unsetunset

你可以总结成下面这个“面试官最爱听”的版本：

> “RAG 的首字延迟主要卡在 embedding 和向量检索。

> embedding 方面通过批处理、异步并发和 KV 缓存减少等待，向量检索通过 HNSW 索引、分区过滤、批量查询缩小范围。

> 系统层面用全链路异步流水线，并辅以 embedding / retrieval / answer 三层缓存，整体能把延迟降低几十到上百毫秒。”

这段话结构清晰、逻辑完整、带工程味，面试官一定会点头。

## unsetunset最后说一句unsetunset

这段时间，我陆续写了二十几篇关于 RAG（检索增强生成）的面试答题文章，阅读量和反馈都非常好。

很多同学说，看完之后不仅知道“怎么答”，还知道“为什么这么答”，甚至能把思路直接用到自己的项目里。

其实，这些文章并不是凭空写出来的，也不是简单整理网络资料，而是**来自我在[大模型训练营](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)里的真实项目沉淀**。

训练营里有多个从零到落地的实战项目。

1、企业培训问答 Agent（含多轮理解与记忆模块）

2、金融研报 RAG 系统（混合检索、重排序、多模态解析）

3、行业深研助手 DeepResearch（实时检索 + 知识沉淀链路）

4、深学 AI 学习助手（上下文结构化与生成链路可解释）

这些实战项目不是“照着文档做一遍”那种，而是会带着同学一步步拆逻辑、跑代码、调权重、对指标，最终能说清楚“为什么这么设计、哪里容易踩坑、怎么迭代优化”。

这些内容最终沉淀成训练营内部的**体系化笔记、方法论文档、Badcase 修复记录和面试表达模板**，而我近期写的那一系列文章，就是从这些文档中衍生出来的。

所以你会看到：

不是只讲概念，而是讲**落地**。

不是只讲方案，而是讲**取舍**。

不是只讲原理，而是告诉你**面试官到底在听什么**。

如果你正在准备[大模型](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)方向的求职，或希望真正把 RAG 从“知道”变成“能做、能讲、能复盘”，那[大模型训练营](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)可能会非常适合你。

真正能拉开差距的，从来不是知识点，而是**体系与思考方式**。

在过去的几个月中，我们已经有超过 **80 个** 同学（战绩真实可查）反馈拿到了**心仪的 offer ，包含腾讯、阿里、字节、华为、快手、智谱、月之暗面、minimax、小红书等各家大厂**以及**传统开发 / 0 基础转行的同学**在短时间内拿到了各类大中小厂的 offer。

如果你近期准备转向[大模型](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)、想拿下一个能讲清楚、能上简历的实战项目，[大模型训练营](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)这可能是你最值得的选择。

往期推荐

[Agent 工程工业级直播课，开营！](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489131&idx=1&sn=ce7cae9ea26b215c375982bd69280bea&scene=21#wechat_redirect)

[面试官问：Agent 的函数调用怎么做到又准又稳？](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489115&idx=1&sn=4d5b70eb8cc5fd9e2f6de0039553f57a&scene=21#wechat_redirect)

[面试官问：Agent 项目经历该怎么写进简历？](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489106&idx=1&sn=ff1f72bcf83ffd0f5c57a9350c26e5e8&scene=21#wechat_redirect)

[面试官问：RAG 的评估体系怎么做？](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489082&idx=1&sn=cd67d77f9b24a2d3a24532cc034998be&scene=21#wechat_redirect)

[面试官问：为什么你的 RAG 能做得比别人好？](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489077&idx=1&sn=64e887a5e0143e7f935f1e4027cf1c0c&scene=21#wechat_redirect)