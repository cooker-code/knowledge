---
title: SkillsBench：首个 Agent Skills 效果基准
author: 符智汇
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA3NjE4MDIzOQ==&mid=2650369975&idx=1&sn=b2077d04b5c407e094c7896f7407c82f&chksm=8640446ac56a36640a4a44d779815791134fde4df7bdff783e926d87b567e9fb8866bc595858&mpshare=1&scene=24&srcid=0408asmD55xkY3qY9KMO2ckz&sharer_shareinfo=7e91853e4ef3b737d165df76d1d19b53&sharer_shareinfo_first=7e91853e4ef3b737d165df76d1d19b53#rd
---

**来源：**arXiv 2602.12670 | **规模：**86 任务 × 11 领域 × 7,308 轨迹  
  

**编者按：**SkillsBench 是首个 Agent Skills 效果基准，86 任务 × 11 领域，7,308 条轨迹。核心发现：Curated Skills 提升 16.2pp，但自生成 Skills 平均无提升。关键洞察：模型无法可靠地自写它们受益的程序化知识；2-3 模块的聚焦 Skills 优于完整文档。这揭示了 Skills 生态的核心矛盾。

  

## 一、基准规模：86 任务 × 11 领域

SkillsBench 是首个系统性评估 Agent Skills 效果的基准：

86

任务

11

领域

7,308

轨迹

**领域覆盖：**代码生成、数据分析、Web 操作、文件管理、API 调用等多种场景。

  

## 二、核心发现：三种 Skills 的对比

### 发现 1：Curated Skills 提升 16.2pp

精心设计的 Skills 显著提升任务表现，平均提升 16.2 个百分点。

### 发现 2：自生成 Skills 平均无提升

模型自己生成的 Skills，平均来看对任务表现没有帮助。

**关键洞察：**模型无法可靠地自写它们受益的程序化知识——Skills 需要外部支持。

### 发现 3：2-3 模块的聚焦 Skills 优于完整文档

小而精的 Skills 比大而全的文档更有效。聚焦的 2-3 个模块比完整文档带来更好的表现。

  

## 三、实验设计

SkillsBench 的实验设计包含三个关键维度：

| 维度 | 描述 |
| --- | --- |
| **Skills 类型** | Curated vs 自生成 vs 无 Skills |
| **Skills 粒度** | 完整文档 vs 聚焦模块（2-3 个） |
| **评估指标** | 任务成功率 + 步骤效率 + 资源消耗 |

  

## 四、对 Skills 生态的启示

* **Skills 不能依赖模型自生成：**需要人工设计或专门的技能发现系统
* **聚焦比全面更重要：**小而精的 Skills 更有效
* **需要标准化的评估：**SkillsBench 提供了评估框架
* **与 EvoSkill/SkillNet 互补：**EvoSkill 负责发现，SkillNet 负责组织，SkillsBench 负责评估

「SkillsBench 揭示了 Skills 生态的核心矛盾：模型需要 Skills，但无法自己创造有效的 Skills。」

  

## 写在最后

SkillsBench 的意义在于：它首次系统性评估了 Agent Skills 的效果，并揭示了关键问题——模型无法可靠地自写它们受益的程序化知识。

**「Skills 需要精心设计，不能依赖模型自生成——这是 Skills 生态发展的核心约束。」**

  

参考文献：SkillsBench: Benchmarking How Well Agent Skills Work — arXiv 2602.12670