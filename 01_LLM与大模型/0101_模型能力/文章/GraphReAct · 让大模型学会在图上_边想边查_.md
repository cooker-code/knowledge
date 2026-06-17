---
title: GraphReAct · 让大模型学会在图上"边想边查"
author: fly的AI学习
date: fly的AI学习fly的AI学习
url: https://mp.weixin.qq.com/s?__biz=MzYzNTg2MzYwNw==&mid=2247484090&idx=1&sn=031669eb69dc4b030df3dd7692aff899&chksm=f186420107fafa5f23f9c12ccd9c22b08f3c91c75bbb8fc91f189f46953c75c535fed09a033b&mpshare=1&scene=24&srcid=05190KglAkqVW3wBwVoeXiVK&sharer_shareinfo=6b07e03487064860dced8d508585c650&sharer_shareinfo_first=6b07e03487064860dced8d508585c650#rd
---

| 项目 | 内容 |
| --- | --- |
| 论文标题 | GraphReAct: Reasoning and Acting for Multi-step Graph Inference |
| 作者 | Xingtong Yu, Zhongwei Kuai, Chang Zhou 等 |
| 机构 | 香港中文大学 · 中科大 · 电子科大 · 东京大学 · RMIT · 新加坡管理大学 |
| 发表日期 | 2026-05-08 |
| 来源 | https://arxiv.org/abs/2605.07357 |
| 标签 | Agent · 图神经网络 · 零样本学习 · ReAct · 节点分类 |

## 这篇论文在说什么？

GraphReAct 做了一件事：让大模型在图结构数据上也能像人一样"边想边查"。

具体来说，它把 NLP 里已经很成熟的 ReAct（Reasoning + Acting）范式搬到了图学习领域。大模型不再被动接收"整理好的图信息"，而是主动决定去图的哪个角落找线索——查查邻居节点是谁、语义相似的有谁，然后整理已有线索再想一步。6 个数据集上的 zero-shot 节点分类实验，在 Cora 上比之前最强方法高了 35%，History 上高了 22%。

## 为什么要关注？

大模型和图数据的结合，是这两年非常火的方向。原因很简单——现实世界里的数据天然就是图结构：

* **社交网络**：人和人的关注、互动关系
* **电商**：用户买了什么、浏览了什么
* **学术**：论文引用谁、谁和谁合作
* **知识图谱**：实体之间的关系

传统的图神经网络（GNN）擅长捕捉局部结构，但它的"视野"是固定的——几层聚合之后就看不了太远。而且 GNN 只能做一次前向推理，不能像人一样"想想再查，查了再想"。

另一方面，大模型擅长推理，但不认识图结构。怎么让大模型既能利用图上的结构信息，又能保持多步推理能力？

之前也有过尝试。GraphGPT 把图信息压缩成 token 喂给大模型；TEA-GLM 做了图-语言对齐。但它们都是**静态上下文**——信息提前整理好，大模型被动接收。

> "While LLM-based and graph–LLM methods, such as LLaGA and TEA-GLM, incorporate textual or graph representations into LLM inference, they typically rely on a fixed and static context, limiting their ability to adaptively exploit available evidence."
>
> 说人话就是：之前的方法相当于给大模型一份整理好的资料，看完给答案。GraphReAct 让大模型像侦探一样，觉得线索不够就主动去查，查完整理一下，再想想，一步步逼近答案。

图源：原文 Fig.1 — 静态推理 vs GraphReAct 的多步推理对比。

## 核心方法

### 关键思路

GraphReAct 给大模型设计了三种"行动"，让它在图上"能动起来"：

图源：原文 Fig.2 — GraphReAct 整体框架，展示了拓扑检索、语义检索、上下文精炼三步循环

**① 拓扑检索（Topological Retrieval）**

沿着图的连线，用 BFS（广度优先搜索）找目标节点最近的 N=4 个邻居。就像你在社交网络里查一个人，顺便看看他的朋友都是谁、关注了什么话题。找到邻居后，让大模型生成一段"拓扑摘要"来描述这些邻居的信息。

> "To capture such structural dependencies, we retrieve nodes that are topologically related to the target node v via a breadth-first traversal, collecting the first N visited neighbors."
>
> 这一招抓的是"局部结构"——一个节点跟谁直接相连，这些邻居大概率能提供有用的分类线索。

**② 语义检索（Semantic Retrieval）**

不看连线，用余弦相似度在整个图里找跟目标节点最像的 M=4 个节点。相当于你不仅看身边的人，还在整个公司里找跟你做类似项目的人。

> "We identify nodes that are semantically similar to the target node v based on cosine similarity between node embeddings, and select the top-M most similar nodes."
>
> 这一招突破图的连通性限制——即使两个节点在图上隔得很远，只要语义接近就能关联。和拓扑检索形成互补：一个管"近邻结构"，一个管"全局语义"。

**③ 上下文精炼（Context Refinement）**

前两步在不断"扩展"信息，但信息多了也有问题——大模型的上下文窗口是有限的，塞太多反而会干扰判断。

> "Context refinement is designed to support multi-step reasoning by progressively updating the reasoning context under the guidance of intermediate thoughts. Unlike graph-based retrieval, which acquires external evidence from the graph, refinement is formulated as another type of action within the same action space."

精炼动作就是每推理一步，把之前积累的信息压缩整理一遍，去掉噪音，保留关键线索。论文里把整个推理过程设计为 **4 步迭代**：先检索（扩展），再精炼（压缩），循环往复。

### 技术细节

整个流程分两个阶段：

**预训练阶段**：用 GraphSAGE（3 层）做图编码器，训练一个投影层把图表示对齐到 LLM（Vicuna-7B-v1.5）的 embedding 空间。LLM 部分是**冻结的**，不微调。

**推理阶段**（4 步循环）：

```
Step 1: 编码目标节点 → 生成初始 Thought → 触发拓扑检索 + 语义检索  
Step 2: 基于 Thought 和检索结果生成新 Thought → 精炼上下文  
Step 3-4: 继续精炼，逐步压缩信息  
最终: 基于精炼后的上下文给出预测
```

从论文的参数分析图来看，4 步推理是最优的平衡点——再多步收益递减，因为小模型（Vicuna-7B）的推理能力有限。

图源：原文 Fig.3 — 推理步数对准确率的影响，4 步达到最佳平衡

## 效果怎么样？

### 主实验：Zero-shot 节点分类

实验设置是：在 Arxiv 和 Computer 两个大图上预训练，然后在 **6 个完全不同的数据集**上直接测试（zero-shot），不接触目标数据集的任何标签。

**引用网络（论文分类）：**

| 模型 | Cora | Pubmed |
| --- | --- | --- |
| GAT | 0.016 | 0.343 |
| DIFFormer | 0.029 | 0.361 |
| GKD | 0.042 | 0.399 |
| Vicuna-7B-v1.5 | 0.156 | 0.719 |
| GraphGPT-std | 0.126 | 0.701 |
| GraphGPT-cot | 0.181 | 0.521 |
| LLaGA | 0.168 | 0.793 |
| TEA-GLM | 0.202 | **0.848** |
| **GraphReAct** | **0.273** | 0.819 |

**电商网络（商品/用户分类）：**

| 模型 | Children | History | Photo | Sports |
| --- | --- | --- | --- | --- |
| Vicuna-7B-v1.5 | 0.270 | 0.363 | 0.378 | 0.370 |
| LLaGA | 0.199 | 0.146 | 0.276 | 0.352 |
| TEA-GLM | 0.271 | 0.528 | 0.497 | 0.404 |
| **GraphReAct** | **0.294** | **0.645** | **0.523** | **0.483** |

几个关键观察：

* Cora 上提升最大，从 20.2% 到 27.3%，+35%。Cora 有 70 个细粒度类别，分类难度高，多步推理的优势明显
* History 上从 52.8% 到 64.5%，+22%。History 是历史类图书，语义关系比较隐含，检索+精炼帮了大忙
* PubMed 上 GraphReAct 略低（0.819 vs 0.848）。论文解释：PubMed 只有 3 个粗粒度类别，邻居节点可能引入歧义信号
* 电商场景全面提升，尤其是 Sports（+19.6%）和 History（+22.2%）

### 消融实验：三个组件分别贡献多少？

| 配置 | 拓扑检索 | 语义检索 | 上下文精炼 | Cora | Children | History | Sports |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 基线 | ✗ | ✗ | ✗ | 0.248 | 0.270 | 0.568 | 0.404 |
| +拓扑 | ✓ | ✗ | ✗ | 0.274 | 0.290 | 0.603 | 0.445 |
| +语义 | ✗ | ✓ | ✗ | 0.256 | 0.283 | 0.599 | 0.418 |
| +两者 | ✓ | ✓ | ✗ | 0.285 | 0.293 | 0.631 | 0.421 |
| +精炼 | ✗ | ✗ | ✓ | 0.255 | 0.272 | 0.578 | 0.410 |
| **完整版** | ✓ | ✓ | ✓ | **0.273** | **0.294** | **0.645** | **0.483** |

图源：原文 Table 2

几个有意思的点：

* 拓扑检索单独用，提升比语义检索大——图结构信息确实很关键
* 两个检索一起用（+两者），History 上跳了 6.3%，说明拓扑+语义的组合在复杂关系图上特别有效
* 上下文精炼单独看提升不大（+0.7%），但在完整模型里它让 History 从 0.631 跳到 0.645——精炼的价值在于"画龙点睛"
* 完整模型在 History 和 Sports 上远超任何单一组件的消融组合，说明三者之间有**非线性协同效应**

### 文本搜索 vs 图检索

论文还做了一组有意思的对比：直接用维基百科文本搜索，和图检索比哪个好？

| 方法 | Cora | Children | History | Sports |
| --- | --- | --- | --- | --- |
| 纯文本搜索 | 0.253 | 0.225 | 0.535 | 0.427 |
| **GraphReAct** | **0.273** | **0.294** | **0.645** | **0.483** |

图源：原文 Table 3

> "These results highlight that, unlike in NLP tasks, effective evidence for graph reasoning is primarily encoded within the graph itself, and directly applying text-based search actions is neither necessary nor beneficial."
>
> 翻译一下：图上的推理，关键信息就在图里。直接套 NLP 那套"去搜索引擎查资料"的思路，在图学习里不管用。这个发现挺重要的——说明图数据和文本数据需要不同的推理策略。

## 和其他方案比有什么不同？

| 对比维度 | GraphGPT / TEA-GLM | LLaGA | **GraphReAct** |
| --- | --- | --- | --- |
| 信息获取 | 被动，预处理后喂模型 | 被动，线性化邻居序列 | **主动** ，模型决定去哪查 |
| 推理轮数 | 单次 | 单次 | **多步** （4步） |
| 检索方式 | 聚合邻居信息 | 序列化邻居+路径 | **拓扑 + 语义** ，互补检索 |
| 上下文管理 | 无 | 无 | **精炼机制** ，扩展+压缩循环 |
| 适用场景 | 小图、简单任务 | 中等规模图 | **复杂图、多类别** |

一句话总结区别：之前的方法是"资料准备好给你看"，GraphReAct 是"你自己去查，查完自己整理"。

这也解释了为什么 GraphReAct 在类别多（Cora 70 类）、结构复杂（History）的数据集上优势大——信息越分散、类别越细，多步检索+精炼的价值就越突出。反之在 PubMed（只有 3 类）上优势不明显，因为问题本身就不够复杂。

## 对开发者意味着什么？

**什么场景值得尝试？**

* **社交网络分析**：用户画像、社区发现、影响力分析。图结构天然适合 GraphReAct 的多步检索
* **推荐系统**：基于用户-商品二部图的推荐。电商数据集上的实验结果已经验证了可行性
* **知识图谱问答**：在实体关系图上推理，"边查边想"比一次性塞入所有三元组更高效
* **金融风控**：交易网络中的异常检测。交易图通常很复杂，多步推理有助于发现隐蔽的异常模式

**能直接用吗？**

* 目前论文**没有开源代码**，但思路可以自己复现
* 核心组件拆开来看，技术门槛不高：BFS 拓扑检索 + 向量相似度检索 + LLM 总结精炼
* 图编码器用的 GraphSAGE（3层），LLM 部分冻结不微调，部署成本可控

**实用建议：**

1. **大图场景先做粗筛**：如果你的图有百万级节点，每步都做全图相似度检索会很慢。建议先用聚类或 ANN 索引缩小候选范围
2. **步数不必贪多**：论文实验显示 4 步最优。更多步数在小模型上反而递减，如果你的 LLM 比 Vicuna-7B 强，可以试试 5-6 步
3. **类别少的情况别强用**：如果你的分类任务只有 3-5 个类别，简单方法可能就够了，多步推理的收益有限
4. **关注后续代码**：作者团队来自港中文+中科大，在图学习领域持续产出，值得持续关注

---

👤 fly的AI学习