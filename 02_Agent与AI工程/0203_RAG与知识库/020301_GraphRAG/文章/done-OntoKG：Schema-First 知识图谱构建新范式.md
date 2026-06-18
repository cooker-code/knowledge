> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020301_GraphRAG/020301_核心知识点/GraphRAG构建检索与评估边界|GraphRAG构建检索与评估边界]]
---
title: OntoKG：Schema-First 知识图谱构建新范式
author: MindChain.AI
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk3NTQxNTEwNQ==&mid=2247484752&idx=1&sn=907510f55299de9af5e74190afdc4539&chksm=c512c132b83e46880934d01f9f5a7c26a2a151be2489fa6f1beaf54536f6a9e2af8460e2338f&mpshare=1&scene=24&srcid=0416N1igcT8ybxTXxZK8UmHV&sharer_shareinfo=4abc259018098aa56700853b959c8687&sharer_shareinfo_first=4abc259018098aa56700853b959c8687#rd
---

📌 一句话总结：

本工作提出 OntoKG，一种以本体（ontology）为核心的知识图谱构建框架，通过 intrinsic-relational routing 实现 schema-first 的结构化建模与下游可复用性。

🔍 背景问题：

当前知识图谱构建方法存在两方面关键问题：

1️⃣ schema 通常隐式耦合在构建 pipeline 中（如 DBpedia、YAGO），难以复用与迁移；

2️⃣ LLM-based 构图方法多为 ad hoc extraction，缺乏统一 schema 约束，导致结构混乱且难以支持下游 ontology-level 任务。

💡 方法简介：

提出 ontology-oriented 的构图范式，将 schema 作为核心产物而非副产物；

设计 intrinsic-relational routing，将属性划分为 intrinsic（节点属性）与 relational（图边），实现结构决策显式化；

构建 declarative schema（YAML-based），支持跨存储后端迁移与模块化复用；

引入 agentic LLM workflow，让 LLM 作为“schema designer”，通过工具调用进行可验证的 schema 迭代优化；

提出迭代 refinement 机制（类似闭环系统），通过未分类实体与未匹配模块驱动 schema 自动扩展与修正。

📊 实验结果：

在 Wikidata（~100M 实体）上构建 34.0M 节点、61.2M 边的大规模 property graph；

schema 覆盖率达到 93.3%，模块匹配率达到 98.0%；

在实体消歧任务上，相比 YAGO 4.5 提升 +2.4 macro score；

支持 ontology analysis、domain customization、LLM-guided extraction 等多种下游任务，验证 schema 的独立价值。

📂 开源链接：

https://github.com/Prorata-ai/OntoKG

📄 论文原文：

https://arxiv.org/abs/2604.02618

✨ 一句话点评：

OntoKG 用“schema-first + intrinsic-relational routing”的设计首次把知识图谱构建从 pipeline engineering 提升到 ontology engineering，本质上是把 GraphRAG 的“结构”变成可学习、可复用的核心资产。