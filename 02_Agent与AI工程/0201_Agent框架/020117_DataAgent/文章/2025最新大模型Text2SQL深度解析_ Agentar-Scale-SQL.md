---
title: 2025最新大模型Text2SQL深度解析: Agentar-Scale-SQL
author: 不求甚解的AI生活
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5ODg1MzE2NQ==&mid=2247483797&idx=1&sn=0794ca4a71992708596fcee7bfec9452&chksm=97782beba8a4bc8cd3acbdb0fa3eafcb1de75bf5273c824dccc3f768905d9763c91b7cfedc6c&mpshare=1&scene=24&srcid=11154LBUtpasNVhgl95RMQsu&sharer_shareinfo=d8f2a863b27e3c76492383daeea0705c&sharer_shareinfo_first=d8f2a863b27e3c76492383daeea0705c#rd
---

# 

近期蚂蚁发布的 Agentar-Scale-SQL 登顶 BIRD 榜单，凭借 **81.67% Test EX** 超过 GPT-4o + AskData、Gemini 等主流方案。  
如果你也在构建 Text2SQL 系统、Agent 工作流、或想理解 Test-Time Scaling 的工程实践，这篇文章将对你非常有价值。

---

## 一、整体架构：

Agentar-Scale-SQL 的核心思想：

> **不改模型参数（即不做 SFT 或长时间训练），  
> 而是在推理阶段通过三类扩展叠加来提升性能。**

三类扩展对应三步工作流：

| 阶段 | 对应扩展类型 | 主要目标 |
| --- | --- | --- |
| Step 1 Task Understanding | 准备阶段 | 获得上下文与支持性信息 |
| Step 2 SQL Generation Scaling | 内部扩展 + 并行扩展 + 顺序扩展 | 生成高质量 & 多样化 SQL |
| Step 3 SQL Selection Scaling | 内部扩展（强化选择） | 选择最正确的 SQL |

## 二、实现流程

### Step 1：Task Understanding（任务理解）

任务理解是整个系统的“信息地基”，确保模型在生成 SQL 前已经具备：

✔ 需要哪些表？

✔ 哪些字段与问题相关？

✔ 是否包含可识别的 cell 值？

✔ 是否有相似历史 Query 可借鉴？

为此系统做了 **两类向量检索 + 两种 Schema 表示**。

#### **1.1 双 Schema 设计（DDL + Light Schema）**

这是一个非常关键的工程优化。

##### ① DDL Schema

* • 用于推理生成器（Reasoning Generator）
* • 数据量大、标准化强、更适合代码/SQL 模型
* • 作为 RL 训练时的输入结构

##### ② Light Schema（Markdown）

* • 极简表结构（表名/字段名/PK/FK）
* • 更适合 ICL（提示式生成器）
* • 避免大模型处理 DDL 时 token 浪费

👉 结论：  
**同一数据库用两个视角表达，提高了不同生成器的适配度。**

#### **1.2 关键字检索（Keyword Extraction → Cell Vector Store）**

SQL 中的 WHERE 条件常依赖具体 cell 值（如年份、ID）。  
系统做法：

1. 1. 从自然语言问题中抽取关键词
2. 2. 使用向量模型 all-MiniLM-L6-v2
3. 3. 检索数据库中的 cell 内容（存储在 Chroma 向量库）
4. 4. 作为提示插入生成器上下文

这一点大幅提升了实体对齐（schema linking）的准确性。

#### **1.3 示例检索（Skeleton Extraction → Example Vector Store）**

Skeleton extraction 是一个微妙但非常有效的过程：

例如问题：

> “Which city has the highest number of transactions in 2021?”

Skeleton 可能是：

> “{metric} of {entity} by {time}”

好处是：

* • 去掉无关词
* • 保留结构特征
* • 能更准确地检索结构相似的 few-shot 示例

最终把最相关的示例插到 ICL Generator 中。

### Step 2：SQL Generation Scaling（最核心部分）

这一部分是 SOTA 能否诞生的关键。

系统使用了 **两个独立生成器 + 一个迭代修正组件**，实现三类扩展中的：

* • 并行扩展（多生成器）
* • 内部扩展（RL强化推理）
* • 顺序扩展（修正环）

#### **2.1 Reasoning Generator（RL 强化推理模型）**

这是生成器 A  
目标：**逻辑强、结构化、执行正确率高**

##### 基础模型

Omni-SQL-32B  
（来自 OmniSQL 系列，对 SQL 相对更友好）

##### 强化学习方法

GRPO  
Reward 设计：

| 情况 | Reward |
| --- | --- |
| 执行结果等价 | 1.0 |
| 可执行但不等价 | 0.1 |
| 语法错误/不可执行 | 0 |

核心优势：

* • 产生可执行 SQL（可执行 > 正确）
* • 对齐语义
* • 多走推理步骤（内部扩展）

模型会在训练中不断学会“多想一步”。

#### **2.2 ICL Generator（提示式生成器）**

生成器 B 的风格完全不同。

##### 使用模型

Gemini-2.5-Pro、GPT-5  
分别使用不同：

* • Prompt 模板
* • Few-shot 示例顺序
* • Temperature（0.5 vs 1.8）
* • Schema（Light Schema）

这使得生成更加**多样化**：

> 同一个问题可能产生 9 种不同的 SQL。

多样性是并行扩展的核心。

#### **2.3 Iterative Refinement（修正器链）**

即便最好的 LLM 也会：

* • 拼写错误
* • join 错误
* • 多余 group by
* • semantic mismatch

因此加入：

##### ✔ SQL Fixer（修语法）

* • LLM 模型检查语法
* • 自动补括号、修错别字、修 column 名

##### ✔ SQL Reviser（修语义）

* • 根据前一次执行结果
* • 提供逻辑修正建议
* • 修改 join、where、group by

修正器本质上是一种“改写器 + 反思链”，属于顺序扩展。

### Step 3：SQL Selection Scaling（选择扩展）

关键问题：

> 生成很多 SQL，如何挑选最正确的？

传统 self-consistency（执行结果最多的为准）有缺陷：

* • 执行结果相同 ≠ SQL 正确
* • group by、join 错了依旧可能返回类似结果
* • 多数投票不一定能选中 edge case

所以 Agentar-Scale-SQL 使用 **Tournament Selection（锦标赛选择）**。

#### **3.1 执行结果分组 → 代表制**

执行后的 SQL 按结果 hash 分组：

* • 每组挑一个代表
* • 避免重复投票
* • 降低评估开销

---

#### **3.2 SQL 两两 PK（选择器模型）**

选择器模型 M\_selection 输入：

* • question
* • schema（light schema）
* • SQL1 + 执行结果1
* • SQL2 + 执行结果2

任务：

> 输出哪一个 SQL 更可能是正确答案。

#### 训练方式

同样用 GRPO。

#### Reward 设计

非常简单粗暴：

| 选择正确 SQL | 1 |
| --- | --- |
| 选择错误 | 0 |

这让选择器变成一个强大的“判官”，不依赖频率，而依赖推理。

## 三、模型选择与训练数据

### 模型使用总结

| 模块 | 模型 | 原因 |
| --- | --- | --- |
| Task Understanding | Gemini-2.5-Flash | 快、便宜、提取信息准 |
| ICL Generator | Gemini-2.5-Pro / GPT-5 | 多样化生成、ICL 强 |
| Reasoning Generator | Omni-SQL-32B（RL 强化） | SQL 专项能力强 |
| Fixer/Reviser | Gemini-2.5-Pro | 擅长纠错 |
| Selection Model | Qwen2.5-Coder-32B-Instruct（RL 强化） | 强推理能力 + token 便宜 |

### 训练数据情况

#### ① Reasoning Generator (RL)：BIRD train set

数据量：12k+  
每条包含：

* • question
* • evidence
* • schema
* • gold SQL
* • execution result

每条问题生成几十条 SQL roll-outs 再训练 RL。

#### ② Selection Model (RL)：构建 pairwise 数据

从训练集构造多组“胜负样本”：

* • 真实正确 SQL
* • 多个错误 SQL
* • 执行结果对比
* • 让模型学会“判别哪条更准”

总计构建 **8.5k 强化学习样本**。

## 四、优势分析

总结深层逻辑：

### **① 双生成器互补性强**

* • 推理模型：逻辑稳健
* • ICL 模型：模式丰富、多样性高

组合后“正确 SQL 覆盖率”最高。

### **② 修正器链避免低级错误**

执行错误的 SQL 数量显著下降。

### **③ 选择器比 self-consistency 更聪明**

它利用：

* • 执行结果
* • 语义理解
* • 字段匹配
* • join 结构
* • 自身 RL 推理能力

能挑出真正最佳 SQL。

---

## 五、总结

Agentar-Scale-SQL 的真正价值在于：

> **它不是依赖某一个更强的模型，而是构建了一个“可扩展、可协调”的推理架构，让不同模型与不同推理风格互相补足。**

其技术亮点包括：

* • 双 Schema 架构
* • 双生成器并行生成
* • RL 强化推理生成器
* • iter-refine 修正器链
* • 锦标赛选择器 + RL 判别
* • 大量高质量训练数据
* • 完整的 Orchestrated Test-Time Scaling 内部、并行、顺序三重扩展

这套设计为未来所有“高可靠代码生成任务”提供了一个范式。