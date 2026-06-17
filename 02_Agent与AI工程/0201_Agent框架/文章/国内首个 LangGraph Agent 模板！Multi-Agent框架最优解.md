---
title: 国内首个 LangGraph Agent 模板！Multi-Agent框架最优解
author: Zilliz
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510194&idx=1&sn=0d5288d7d900785b489736dc7631cb8c&chksm=fbc994f2911b644f43956da2c296df39b5accd15e22bf9538d4dbac3c518e1a0ed01bb13b6f5&mpshare=1&scene=24&srcid=0912rEoTuytdrPOXWpWem3Cn&sharer_shareinfo=91d08711ed84453155fa621557629330&sharer_shareinfo_first=91d08711ed84453155fa621557629330#rd
---

本文是LangGraph 系列第二篇，首篇请见[LangChain vs LangGraph：谁是agent落地最优解](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510179&idx=1&sn=dcdddcffd80931259f7087377438f232&scene=21#wechat_redirect)

后续基于langgraph-up-react做Multi-Agent的教程，将在本系列第三篇推出

大模型落地，要么做workflow、要么做agent，而无论哪一种，全都离不开框架。

而毫无疑问，LangGraph 是构建Agent 系统的最主流框架之一。

它不仅支持循环、条件分支，能处理复杂流程；对应用流程和状态有细粒度控制；有内置持久性功能，跨交互保持上下文；支持人工干预；还能实时流输出数据……**堪称Multi-Agent系统的框架最优解。**

但是，作为新事物，LangGraph也有不小问题，比如：官方的文档变更频繁，学习曲线较高，对新手并不算友好。

**而对大部分开发者来说，做agent没那么难，难的是，如何从0到1做个细分领域能用的小应用，解决启动问题。**

那不妨试试 langgraph-up-react，这一套 LangGraph 模板，不仅完美解决以上问题，而且更适合中国用户的开发环境。

以下是基于LangGraph 做一个ReAct Agent的手把手教程。

# 01 ReAct Agent 解读

ReAct（Reason + Act） 框架诞生于谷歌 2022 年的论文《ReAct: Synergizing Reasoning and Acting in Language Models》。

其意义在于，结合了 Chain of Thought（CoT，思维链，擅长推理，但无法与外部环境交互）以及传统强化学习 Agent（擅长环境交互，但缺乏逻辑规划）两者之所长，让 agent 像人类一样，通过 **“分析问题（思考）→ 与环境交互（行动）→ 获取反馈（观察）” 的动态循环解决问题。**

其核心模块有三：

* **Reason（推理）**：借鉴 CoT“Chain-of-Thought（思维链）”，让模型先自我拆解、自我提问的方式地分析问题。此外，这个模块的核心能力是上下文理解、策略规划，更重要的是，它还能根据反馈的信息对错误内容进行纠正，对行为做策略进行调整。
* **Act（行动）**：根据推理结果，调用工具执行操作（如 API、数据库、计算器、搜索）。
* **Observation（观察）**：查看行动结果，进入下一轮推理或行动。核心能力在于结果的收集与分析，以及信息的过滤提取。

现如今，ReAct 已成为智能体（如 ChatGPT 插件、机器人控制、智能助手）的核心底层架构之一，广泛应用于需要动态交互的真实场景。

# 02 LangGraph 解读

### 2.1 LangGraph 解决了什么问题？

传统的 AI 模型不能维持这种**多步骤的连续推理过程**，每次对话都是独立的，不能记住之前的处理结果，也不能基于中间结果动态调整后续步骤。

LangGraph 正是为了解决这个问题而生。它能将复杂任务分解成多个步骤，记住每个步骤的结果，并根据情况灵活调整执行路径，最终完成整个任务链条。

### 2.2 LangGraph 的核心机制

LangGraph 把 AI 的思考过程变成了一个**流程图**：

每个步骤是一个节点，箭头是路径。AI 可以：

* **记住**每个步骤的结果
* **决定**下一步该做什么
* **调用**各种外部工具
* **循环**处理直到完成任务

这就是为什么叫 LangGraph：Lang（语言）+ Graph（图）= 用图结构组织语言 AI 的思考流程。

# 03 langgraph-up-react 解读

LangGraph 虽好，但熟练掌握状态管理、节点设计、边控制、错误处理等细节需大量精力；模型接入、工具集成、环境配置等基础工作需从零开始，还可能遇兼容性问题。

尤其多步骤流程出错时，问题定位难度大（可能出现在任一节点或状态转换）；且业务需求变化会导致代码结构复杂化，功能修改与扩展成本剧增。

**因此，我们需要一套成熟的 LangGraph 模板：**开发者只需下载代码、配置 API 密钥，即可快速启动完整 AI Agent 系统。

这类模板集成经大量项目验证的架构设计与代码规范，规避常见设计陷阱；预置工具、测试用例及部署脚本，可避免重复造轮子，让开发者专注业务逻辑实现。

而 langgraph-up-react 是专为国内开发者打造的 LangGraph 模板，核心特性如下：

* 🇨🇳 国内模型优先：深度集成通义千问、DeepSeek、智谱 AI 等
* 🔧 完整 MCP 工具生态：内置丰富 MCP 工具与适配器
* ⚡ 开箱即用：简化配置流程，5 分钟快速启动
* 🧪 完善测试：提供全面单元测试与集成测试

# 04 启动模板

### 1.环境依赖安装

```
```
curl -LsSf https://astral.sh/uv/install.sh | sh
```
```

### 2.下载项目到本地

```
```
git clone https://github.com/webup/langgraph-up-react.gitcd langgraph-up-react
```
```

### 3.安装项目依赖

```
```
uv sync --dev
```
```

### 4.配置项目环境

#### 4.1 创建 `.``env` 文件

```
```
cp .env.example .env
```
```

#### 4.2 填写 API-KEY

```
```
# Web 搜索功能（必需）TAVILY_API_KEY=your-tavily-api-key# 模型提供商（至少选择一个）DASHSCOPE_API_KEY=your-dashscope-api-key  # 千问模型（默认推荐）OPENAI_API_KEY=your-openai-api-key  # OpenAI 或兼容平台# OPENAI_API_BASE=https://your-api-endpoint  # OpenAI 兼容平台需设置# 可选：模型供应商的区域支持REGION=prc  # 中国大陆地区# 可选：启用文档工具ENABLE_DEEPWIKI=true
```
```

#### 4.3 启动项目

```
```
# 启动开发服务器（不带界面）make dev# 启动带 LangGraph Studio UI 的开发服务器make dev_ui
```
```

#### 4.4 访问环境

# 05 模板使用场景示例

### 5.1 企业知识库智能问答（Agentic RAG）

构建企业内部文档的智能问答系统。系统将公司的技术文档、产品手册、FAQ 等内容向量化存储到 Milvus 数据库中，当员工提问时，Agent 会自动检索相关文档片段，并结合上下文生成准确答案。

*提示：在实际应用中，需要根据数据规模选择合适的 Milvus 版本：小规模数据使用 Milvus Lite，中等规模使用 Standalone 版本，大规模数据则选择分布式部署。同时要优化 Milvus 的索引配置，合理设置 HNSW 参数以平衡检索精度和性能。建立完整的监控体系，实时跟踪向量检索的响应时间和准确率，确保系统稳定运行。*

### 5.2 多智能体协作系统

构建多个专业 Agent 协同工作的系统，比如软件开发助手。产品经理 Agent 负责需求分析，架构师 Agent 设计技术方案，开发 Agent 编写代码，测试 Agent 进行质量检查。

具体教程，敬请期待本系列第三篇文章。

作者介绍

Zilliz 黄金写手：尹珉

推荐阅读

[全面测评LangChain vs LangGraph：谁是agent落地最优解](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510179&idx=1&sn=dcdddcffd80931259f7087377438f232&scene=21#wechat_redirect)

[Embedding无敌？是做文档处理RAG最大的幻觉](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510140&idx=1&sn=b4314a435c9e4e010856dbe63bac0ab3&scene=21#wechat_redirect)

[Hugging Face轻量化框架Candle实战｜基于Rust，比PyTorch配置简单](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510123&idx=1&sn=cdc0436e5a2585df0ae788d5ae887bc4&scene=21#wechat_redirect)

[Word2Vec、 BERT、BGE-M3、LLM2Vec，embedding模型选型指南｜最全](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510057&idx=1&sn=58fdfef27956475e9eb974e3bd822859&scene=21#wechat_redirect)