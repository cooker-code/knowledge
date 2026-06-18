---
title: 近期，不错的LLM Agent统一记忆框架综述~
author: PaperAgent
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247506893&idx=1&sn=285140feb60dcfb25244304258991bd5&chksm=c3d8ab0aa40a394934c3f0f4e02ed4807e8f9ff8afb50a6ed18bd116445b5705da8d1fe3dee2&mpshare=1&scene=24&srcid=0427Faoaie9iFl0lYOiKVGg1&sharer_shareinfo=976737d44e14f5bf71e423b94525af27&sharer_shareinfo_first=976737d44e14f5bf71e423b94525af27#rd
---

随着**GPT、Qwen、Claude** 等大模型能力持续提升，LLM-based Agent 正在从单轮问答走向更复杂的**长期任务**：多轮对话、个人助手、游戏智能体等。在这些场景中，Agent 不仅要理解当前输入，还要持续积累过去的交互、偏好、事实变化和任务状态。

一个直接方案是把历史消息全部放进 prompt，也就是 **naive long-context prompting**。但这种方式会带来几个**明显问题**：上下文窗口可能溢出，token 成本高，推理延迟增加，并且模型也不一定能找到真正相关的证据。[ICLR 2026杰出论文开奖：我找到了LLM真正的考场](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247506892&idx=1&sn=030fcb0a076e1f3d130f3ff7a8249eb7&scene=21#wechat_redirect)

因此，**Agent Memory 的核心目标**是：不要让模型“每次重读全部历史”，而是让系统主动维护一套记忆机制，在需要时取回相关信息，帮助 LLM 进行更可靠的长期推理。

Memory in the LLM Era: Modular Architectures and Strategies in a Unified Framework

这篇论文面向 **LLM Agent 长期记忆机制**的系统化理解与评测，从模块化视角统一抽象现有方法，并在一致实验设置下分析不同设计在效果、成本与鲁棒性上的表现，为后续记忆系统设计提供经验和参考。

## 1. 统一框架：把 Agent Memory 放进同一张图里

论文提出的统一框架将 Agent Memory 拆解为四个核心组件：**Information Extraction**、**Memory Management**、**Memory Storage** 和 **Information Retrieval**。[ICLR 2026杰出论文开奖：我找到了LLM真正的考场](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247506892&idx=1&sn=030fcb0a076e1f3d130f3ff7a8249eb7&scene=21#wechat_redirect)

该框架可以统一刻画各种 Agent Memory 代表性方法，将它们拆解到同一组组件中进行系统分析。

## 2. 四个核心组件：Agent Memory 到底由什么组成？

### Information Extraction：记什么？

信息提取决定哪些内容会进入记忆系统，现有方法主要包括三类：**直接归档** 、**总结式提取** 和 **基于图的提取**。

### Memory Management：怎么维护记忆？

记忆管理决定新旧记忆如何融合、演化和遗忘。论文将该过程总结为五类操作：连接相关经验、整合碎片记忆、在不同记忆层级之间迁移、更新已有记忆、以及过滤无用信息。

### Memory Storage：存在哪里、用什么结构存？

记忆存储可以从两个维度理解：组织结构和表示方式。在组织结构上，分为 **扁平式存储** （JSON、队列）和 **层级式存储** （长短期、树结构中的不同层级）。在表示方式上，分为 **基于向量的存储** 和 **基于图的存储** 。

### Information Retrieval：如何取回相关记忆？

信息检索决定当前 query 到来时，系统如何从记忆中找到最有用的信息。论文将其分为四类。

**词汇匹配检索** 例如 BM25 或 Jaccard，精确匹配实体、名称和关键词。

**向量检索** 依赖向量余弦相似度、ANN 算法，是许多方法的基础检索方式。

**结构检索** 利用图或树中的显式连接，通过邻居扩展、图遍历找到相关信息。

**LLM辅助检索** 让 LLM 参与检索过程，识别关键信息，或直接判断记忆相关性。

## 3. 实验：统一复现、系统比较

### 3.1 做了哪些实验？

**LOCOMO** 是一个人类长期对话记忆数据集，问题覆盖单跳、多跳、时间推理和开放域知识等类型。

**LONGMEMEVAL** 是一个用户与 AI 长期交互记忆数据集，用于评估信息提取、多会话推理、知识更新和时间推理等能力。

围绕这两个数据集，我们统一复现并比较 10 个代表性 Agent Memory 方法，实验包括：整体性能对比、token 消耗情况及性能-成本权衡、上下文扩展性分析、证据位置敏感性分析，以及不同底层 LLM 下的表现对比。

### 3.2 主要结果与发现

* 在整体性能表现中，MemTree、MemoryOS、MemOS 等层次化或树状方法表现突出，说明多层结构能够同时保留高层摘要和底层证据，更适合复杂长期任务。
* 将多轮对话作为一个整体进行处理可以显著降低 token 消耗，适当的粗粒度处理反而可能提升记忆效果。
* 当上下文规模扩展到 200% 时，几乎所有方法都会出现性能下降。相比之下，采用更明确层次管理的方法通常更稳定。

* 多数方法存在证据位置敏感性：当关键证据位于更早会话时，很多方法更容易被后续信息干扰而检索失败。
* 现有记忆架构仍然依赖底层 LLM 的推理能力。从 Qwen2.5-7B 扩展到 72B 后，多数方法都有明显提升。

### 3.3 新SOTA算法

基于上述发现，我们进一步组合 MemTree/MemOS 的树状组织能力与 MemoryOS 的分层存储架构，设计出一个新的低 token 开销 Agent Memory 框架。

lme-sota

cost sota

```
论文链接: https://arxiv.org/abs/2604.01707  
代码仓库: https://github.com/Yanchen398/Memory-in-the-LLM-Era
```

[动手设计AI Agents：（编排、记忆、插件、workflow、协作）](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247492838&idx=2&sn=1e25832e7300ef312721325d0def30b4&scene=21#wechat_redirect)

[分享两篇Claude Skills最新论文，有3个核心结论](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247505843&idx=1&sn=9a8a4915606ee747998015395413db87&scene=21#wechat_redirect)

[会学习的龙虾，才是好龙虾：OpenClaw-RL](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247505225&idx=1&sn=4b32282cf6853e29f884bdfb85f89f2e&scene=21#wechat_redirect)    
[2026，做Agentic AI，绕不开这两篇开年综述](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247502666&idx=1&sn=d6a467896c6753c8d8634c7400d8dbb4&scene=21#wechat_redirect)

---

每天一篇大模型Paper来锻炼我们的思维~已经读到这了，不妨点个👍、❤️、↗️三连，加个星标⭐，不迷路哦~