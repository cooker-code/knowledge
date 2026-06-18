> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020301_GraphRAG/020301_核心知识点/GraphRAG构建检索与评估边界|GraphRAG构建检索与评估边界]]
---
title: SeedER：让知识图谱检索从“相似度匹配”走向“结构化探索”
author: KGraph Pattern
date: dddangerdddanger
url: https://mp.weixin.qq.com/s?__biz=MzYzNDE0Mjk1MA==&mid=2247484844&idx=1&sn=044ecb8afb22c1e4148aaab8c18ce45f&chksm=f1f49906e38ebf5ae2f7280b2d8ff8ba1d8fdb478a0bc099ab5ef681abcb8c7cd59ba6f303f9&mpshare=1&scene=24&srcid=0525Nux9CrfoSQ5otBpIT26K&sharer_shareinfo=fdfa0e370e50c35459c22a5709c3b794&sharer_shareinfo_first=fdfa0e370e50c35459c22a5709c3b794#rd
---

## 01｜问题背景：知识图谱检索，难点不在“找相似”，而在“走路径”

知识图谱是一类非常适合承载结构化知识的数据形式。节点可以表示疾病、药物、基因、论文、作者、商品等实体；边则表示实体之间的关系，例如“药物治疗疾病”“基因参与通路”“论文引用论文”“商品属于某品牌”。

但作者指出，知识图谱检索的真正难点在于：

> **答案节点往往并不和查询文本直接相似，而是隐藏在一条关系路径之后。**

例如，一个用户提出问题：

> 哪些药物通过胆碱能通路治疗阿尔茨海默病？

真正的答案可能是 Donepezil、Galantamine 等药物。它们和查询文本未必有明显词面重合，但它们可以通过类似下面的路径被找到：

**Alzheimer’s disease → ACHE gene → cholinergic signaling pathway → Donepezil drug**

这说明，知识图谱检索不是简单地把“查询”和“节点文本”做语义相似度匹配，而是需要沿着图中的关系进行多跳推理。

## 02｜传统方法的瓶颈：Dense Retrieval 为什么不够？

目前常见的检索方法是 dense retrieval，也就是把查询和每个节点都编码成向量，然后按照余弦相似度或内积排序。

这种方式在普通文本检索中很有效，但作者认为它不适合处理知识图谱中的多跳组合查询。

原因很直接：

**Dense Retrieval 试图用一次向量比较，完成一整条关系路径的推理。**

这就会带来两个问题。

第一，语义相似度可能找得到起点，却找不到终点。比如查询中出现“阿尔茨海默病”，dense retriever 很可能能找到疾病节点；但真正的答案药物节点并不一定和查询文本直接相似。

第二，把节点邻域、关系描述、结构信息都塞进节点文本中，也只能缓解一部分问题。因为这种方式本质上还是局部增强，无法真正表示任意长度的多跳关系组合。

作者在理论部分给出一个重要判断：

> 对某些关系追踪型知识图谱查询，如果只依赖固定的查询向量和节点向量来判断相关性，dense retrieval 需要接近图规模的表示容量；而一个局部迭代式策略只需要学习每一步如何沿边走。

换句话说，dense retrieval 的问题不是“模型还不够大”那么简单，而是它的检索范式本身不适合多跳组合推理。

## 03｜一个自然想法：从种子节点开始做 K-hop 扩展

既然 dense retrieval 不擅长直接找到最终答案，那它是否仍然有价值？

作者的答案是：有价值，但应该把它当成“起点发现器”。

也就是说，dense retrieval 可以先找出和查询最相似的一小批节点，例如疾病节点、论文主题节点、品牌节点等。然后系统再从这些种子节点出发，沿着知识图谱的边进行扩展。

但朴素的 K-hop 扩展会立刻遇到一个严重问题：

> 图的邻域增长太快，扩展两三跳之后，候选节点数量可能暴涨到数万甚至接近整个图。

为了解决这个问题，作者先构造了一个简单但有效的基线方法：**K-hop-with-filtering**。

它的做法是：

* 从 dense retrieval 找到的种子节点开始；
* 每一跳只考虑当前 frontier 的一跳邻居；
* 用查询与源节点、关系、候选节点的相似度综合打分；
* 每一跳只保留分数最高的一小批节点；
* 最终形成一个受预算约束的候选集合。

这个方法已经比纯 dense retrieval 更好，尤其能提升 Hit@Any 和 Recall@Any 这类关注“是否把答案捞进候选集”的指标。

## 04｜关键转折：为什么还需要“学习式扩展策略”？

虽然 K-hop-with-filtering 能控制候选规模，但作者指出，它仍然是一个贪心策略。

它的问题在于：每一步都偏向选择当前看起来最相关的节点。

但在知识图谱中，正确路径可能需要先经过一个“看起来不相关”的中间节点，之后才能到达真正有用的答案区域。

这就产生了所谓的 **delayed reward** 问题：

> 有些节点短期看没有收益，但长期看是通往答案的桥梁。

例如，一个中间基因节点或通路节点，单看文本可能和查询不够相似，但它正好连接着最终答案药物。贪心方法可能会跳过它，从而永远到不了答案节点。

作者因此提出：知识图谱检索中的扩展过程应该被看作一个序列决策问题。

这正是强化学习适合处理的场景。

于是，SeedER 的核心思想出现了：

> 不再固定地、贪心地扩展邻居，而是训练一个 query-conditioned 的图感知策略，让模型学习“下一步应该扩展哪些 frontier 节点”。

## 05｜SeedER 方法：Seed-and-Expand 的三段式框架

SeedER 的名字本身就说明了它的主要流程：**Seed + Expand + Rank**。

### 第一步：Seeding the Core Set

作者首先使用轻量的 dense retrieval 找到一小批核心种子节点。

这些节点不一定是最终答案，但它们通常和查询强相关，可以作为后续图搜索的语义锚点。

例如，在医学知识图谱中，查询中提到某种疾病，dense retrieval 往往能找到疾病节点；之后真正的答案可能需要沿着疾病—基因—通路—药物的路径继续扩展。

### 第二步：Bounded Search Space Construction

直接在完整知识图谱上训练强化学习策略成本太高，因为一跳邻居可能很多，多跳后候选空间更大。

因此，作者先用 K-hop-with-filtering 从种子节点出发，构造一个中等规模的 query-specific subgraph，通常包含约 100–200 个节点。

这个子图相当于强化学习策略的“局部环境”。

这样做的好处是：

* 不需要每一步都访问整个知识图谱；
* 不需要在完整图上做大规模 GNN 计算；
* 保留了和当前查询最相关的局部结构；
* 让训练和推理都更可控。

### 第三步：Graph-Aware Expansion Policy

在局部子图中，SeedER 迭代地选择 frontier 节点。

每一步，模型都会构造当前已选节点和候选 frontier 节点组成的诱导子图，并用 GNN 生成 query-conditioned node embedding。随后，一个轻量 policy head 给每个 frontier 节点打分。

训练时，模型从策略分布中采样节点，以鼓励探索；推理时，则选择分数最高的节点。

最后，SeedER 还会用一个 scoring head 对已选候选节点重新排序，让真正答案尽可能排在前面。

## 06｜训练目标：用强化学习负责“找到答案”，用 BPR 负责“排好答案”

SeedER 的训练目标由两部分组成。

第一部分是强化学习策略损失。作者使用一种 group-centered REINFORCE 训练方式。对于同一个查询，模型会采样多条扩展轨迹，在实验中每个查询采样 8 条轨迹。每条轨迹的奖励来自 Recall@Any，也就是看它是否在候选集合中覆盖了更多真实答案。

这里的重点是：

> 强化学习策略不直接优化最终排序，而是优化“能不能把答案节点找进候选集”。

第二部分是监督排序损失。作者使用 BPR loss，让最终 scoring head 学会把正样本答案节点排在负样本节点前面。

因此，SeedER 的分工非常清晰：

* **RL policy**

  ：负责候选发现，尽量扩大答案覆盖率；
* **GNN scoring head**

  ：负责最终排序，提高 Hit@1、Hit@5、MRR 等排序敏感指标；
* **BPR loss**

  ：给 GNN 提供稳定的监督信号；
* **group-centered baseline**

  ：降低强化学习训练方差，提高稳定性。

## 07｜实验设置：三个 STARK 知识图谱检索任务

作者在 STARK benchmark 的三个数据集上评估 SeedER：

**STARK-PRIME**：医学知识图谱检索任务，基于 PrimeKG，包含疾病、药物、基因、通路等实体。这个数据集节点较少，但关系更密集、更复杂，适合检验多跳结构推理能力。

**STARK-MAG**：学术论文检索任务，包含 paper、author、institution、field\_of\_study 等实体，查询往往同时包含文本条件和关系条件，例如某领域、某作者、某机构、引用关系等。

**STARK-AMAZON**：商品检索任务，包含 product 和 brand 两类实体，以及 also\_bought、also\_viewed、has\_brand 等关系。查询更接近真实用户商品搜索，且很多问题有多个正确答案。

作者使用的主要指标包括：

* **Hit@1 / Hit@5**

  ：前 1 或前 5 个结果中是否命中答案；
* **MRR**

  ：第一个正确答案出现得越靠前，分数越高；
* **Recall@20**

  ：前 20 个结果中覆盖了多少真实答案；
* **Hit@Any / Recall@Any**

  ：不考虑排序，只看候选集合里是否包含答案。

## 08｜实验结果：SeedER 在“轻量 first-stage retrieval”中明显领先

主实验使用 MiniLM-L6-v2 作为文本编码器，对比了 dense retrieval、G-Retriever、SubgraphRAG、Beam Search、A\* Search、PPR、PPR+MMR、K-hop-with-filtering 等方法。

结果显示，SeedER 在三个数据集上都取得了最好的整体表现。

在 **STARK-PRIME** 上，SeedER 相比 dense retrieval 提升非常明显：

* Hit@1：从 0.101 提升到 0.199；
* Hit@5：从 0.218 提升到 0.411；
* MRR：从 0.161 提升到 0.293；
* Recall@20：从 0.259 提升到 0.461。

这说明 SeedER 不只是把答案“捞进来”，也能通过 GNN scoring head 改善排序质量。

在 **STARK-MAG** 上，SeedER 同样优于所有一阶段检索基线，Recall@20 达到 0.449。

在 **STARK-AMAZON** 上，dense retrieval 本身已经比较强，因此图扩展带来的提升较小，但 SeedER 仍然取得最好的 Hit@1、Hit@5、MRR 和 Recall@20。

作者还测试了更强的节点编码器。使用 OpenAI text-embedding-ada-002 时，SeedER 在 STARK-PRIME 上将 Recall@20 从 dense retrieval 的 0.360 提升到 0.570。使用 Qwen3-Embedding-4B 时，SeedER 进一步达到 0.310 Hit@1、0.582 Hit@5、0.429 MRR 和 0.647 Recall@20。

这说明 SeedER 并不是替代强 embedding 模型，而是可以和强 embedding 模型互补：embedding 越强，种子节点和初始特征越好；SeedER 的学习式扩展仍然能继续带来结构推理收益。

## 09｜效率与消融：为什么说 SeedER 是一个实用的中间方案？

作者进一步将 SeedER 与 LLM-based agentic graph retrieval 方法进行比较，例如 ToG、SFT、PRM 和 GraphFlow。这些方法通常使用大语言模型逐步探索图结构，有些还加入 LLM reranking。它们表达能力强，但推理成本也高。

SeedER 的定位不同。它并不试图替代完整的 LLM 推理系统，而是作为轻量级 first-stage retriever，先用较低成本产出高覆盖率候选集，再交给后续 reranker 或生成模型处理。

在效率比较中，SeedER 只有约 **1.1M 可训练参数**，而 GraphFlow 使用的是 **8B 参数级别**的 LLM。作者指出，SeedER 参数量约为 GraphFlow 的 1/8000，并且每个查询的推理速度快得多，同时仍能获得很大一部分性能收益。

消融实验也证明了 SeedER 的设计不是简单堆叠模块，而是各部分都有贡献：

* 去掉辅助排序损失后，性能明显下降；
* 只采样单条轨迹不如多轨迹训练稳定；
* 不使用 baseline 或使用 greedy baseline 都不如组内均值 baseline；
* 只用 GNN rerank K-hop-with-filtering 的子图，仍然不如完整的 RL-based SeedER。

这说明 SeedER 的提升来自两个方面的结合：

> **学习式候选发现 + 监督式最终排序。**

作者还在附录中分析了训练稳定性。10 个随机种子的训练曲线显示，BPR loss 稳定下降，训练 reward 和验证/测试 Recall@20 在早期提升后进入平台期。验证集和测试集指标相关性也很高，说明模型选择信号比较可靠。

**总结：**

总体来看，SeedER 的核心贡献可以概括为三点。

第一，作者明确指出了知识图谱检索中的范式问题：多跳组合查询很难仅靠一次 dense embedding 匹配解决。答案节点可能和查询文本并不相似，但它们可以通过图结构中的关系路径被找到。

第二，作者提出了一种轻量、可控的 seed-and-expand 框架。SeedER 先用 dense retrieval 找到语义锚点，再在局部子图中用强化学习策略选择值得扩展的 frontier 节点，最后用 GNN scoring head 重新排序候选节点。

第三，作者通过理论分析、主实验、强编码器对比、LLM agent 对比、消融实验和训练稳定性分析证明：SeedER 是一种介于“便宜但浅层的 dense retrieval”和“强大但昂贵的 LLM 图探索”之间的实用方案。

它最适合扮演的角色，是知识密集型系统中的第一阶段检索器：

> 先用较低成本找出紧凑、高覆盖率的候选节点，再交给更强的 reranker 或 LLM 完成最终推理。

对于知识图谱 RAG、医学知识检索、学术检索、商品图谱搜索等场景，这种思路都具有现实意义。

> SeedER 的价值在于，它把知识图谱检索从“全局相似度排序”改造成“局部结构化探索”：不是问哪个节点最像查询，而是学习下一步该沿哪条图关系走向答案。

更多详细内容可以参考论文原文.