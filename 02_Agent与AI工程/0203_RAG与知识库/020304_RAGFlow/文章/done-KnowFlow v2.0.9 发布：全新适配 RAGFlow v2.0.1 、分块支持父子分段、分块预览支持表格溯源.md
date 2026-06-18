> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020304_RAGFlow/020304_核心知识点/RAGFlow工程化边界与知识库治理|RAGFlow工程化边界与知识库治理]]
---
title: KnowFlow v2.0.9 发布：全新适配 RAGFlow v2.0.1 、分块支持父子分段、分块预览支持表格溯源
author: KnowFlow 企业知识库
date:
url: https://mp.weixin.qq.com/s?__biz=MzAxNDg0NjEzNw==&mid=2651489737&idx=1&sn=a4fe3707d40dd5213b1ccfef2aa8a810&chksm=817cb78d605343c95b0ea04cc1f82d5f8d37b83b66825ae5a77cec912c4a2b2e79b65eeb7ded&mpshare=1&scene=24&srcid=0912ah1HjqX40KpVQX56rXKE&sharer_shareinfo=359bbd824cbd11b6179d5a1b6c0491ce&sharer_shareinfo_first=359bbd824cbd11b6179d5a1b6c0491ce#rd
---

# KnowFlow v2.0.9 发布：全新适配 RAGFlow v2.0.1 、分块支持父子分段、分块预览支持表格溯源

继 KnowFlow v2.0.4 支持 RBAC 用户权限管理后，我们修复了一些小问题。v2.0.9 正式发布，**本版本全新适配了 RAGFlow v2.0.1 ，将新版本 Agent 移植进来。支持类似于 Dify 的父子分段检索，同时优化了分块预览时对于表格的支持。**

# 新版预览

## 适配 RAGFlow v2.0.1

得益于 KnowFlow 微服务设计，RAGFlow 新版兼容对于后端代码可以一键合入；由于前端做了深度定制，前端代码稍显复杂，我们通过 AI 编程工具 Claude Code 自动进行合入, 准确率达到 70%，省下手动进行修补。

## 父子分块

在使用 RAGFlow 官方分块算法时，经常遇到一个问题，当答案分布在多个子块之间时，想要召回完整只能看运气；或者通过自动问题和自动关键词勉强实现。

**为了解决分块的上下文关联问题，新版本我们引入了父子分块。** 可以看看效果：

上述规则条文分布在多个上下相关的子块中，普通的分块策略导致在回答时只能命中部分。

应用最新版本的父子分块后，看下最终的检索效果：

所有相关的条例完美召回。

## 分块预览表格溯源

原先版本对于表格溯源一直没做处理，导致只能坐标溯源普通文本内容，新版本我们添加了对表格支持。

# 父子分块技术架构

## 整体架构流程

```
检索阶段

原始文档

Markdown AST解析

增强节点创建

智能分块处理

子分块生成Child Chunks

父分块生成Parent Chunks

子分块向量化Embedding

父分块存储Context Storage

向量索引Elasticsearch

数据库存储MySQL

关联关系映射Relationship Mapping

ParentChildMapping表

用户查询

向量检索子分块

根据关联获取父分块

返回上下文丰富的结果
```

## 原理剖析

KnowFlow 是基于 MinerU 解析产物 MarkDown  进行分块的，为了更好的支持 MarkDown  文档结构解析，我们采用了 AST 进行结构解析。正好今天我们发现了 RAGFlow 官方也开始采用 AST 增强 MarkDown 文档解析了，也算是殊途同归。

1. 1. 对于**子块我们通过 AST 根据语义进行分块，确保语义完整。**

```
关联策略

Markdown文本

MarkdownIt解析

AST语法树

增强节点创建

节点信息提取

标题节点Header Nodes

段落节点Paragraph Nodes

表格节点Table Nodes

代码块节点Code Blocks

列表节点List Nodes

标题层级分析H1/H2/H3

父分块边界确定

内容分块处理

Token计数与大小控制

智能分割点选择

父分块生成Parent Chunk Creation

子分块生成Child Chunk Creation

语义关联分析Semantic Relationship Analysis

关联关系建立Relationship Creation

语义包含semantic_containment

层级关系hierarchical_child

上下文相关contextual_related
```

1. 2. 对于父块，**我们通过 AST 按照标题进行分父块，前端界面提供标题层级：H1/H2/H3 等，通过这种方式能快速对父块进行按照标题进行不同颗粒度划分。**

1. 3. 通过 mysql 存储父子分块的映射关系
2. 4. **子块参与向量化存储，检索时查找子块关联的父块，送给 LLM chat 进行回复。**

## 架构核心优势

### 1. 智能关联机制

* • **AST语义分析**: 基于Markdown语法树建立精确的语义关联
* • **层级结构映射**: 按标题层级(H1/H2/H3)建立父子关系
* • **位置感知关联**: 通过行号范围确定包含关系

### 2. 可配置分块策略

* • **文档级配置**: 支持单个文档的个性化分块策略
* • **知识库级配置**: 提供全局默认配置和批量管理
* • **动态策略调整**: 运行时可调整分块参数

### 3. 多模式检索

* • **子分块模式**: 精确匹配，快速检索
* • **父分块模式**: 上下文丰富，语义完整
* • **混合模式**: 兼顾精度和上下文

### 4. 系统集成设计

* • **非侵入式**: 不修改 RAGFlow 核心代码
* • **向后兼容**: 现有功能完全保持

## Roadmap

* • 官方文档更新和 B 站视频录制：近期频繁发布了多个版本，但使用说明文档尚未来得及更新，需体系化整合和输出。
* • **接入小红书 dots ocr** 和 MinerU 进行比较。

## **交流联系**

欢迎关注「KnowFlow 企业知识库」，加入交流群，一起沟通交流。**KnowFlow 旨在成为基于 RAGFlow 的企业级知识库解决方案**。欢迎提出更多有价值的需求和问题，一起探讨和解决。