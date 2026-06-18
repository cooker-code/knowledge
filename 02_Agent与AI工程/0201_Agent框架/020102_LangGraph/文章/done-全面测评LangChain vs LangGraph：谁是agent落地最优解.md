> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020102_LangGraph/020102_核心知识点/LangGraph DeepResearch监督者与状态管理|LangGraph DeepResearch监督者与状态管理]]
---
title: 全面测评LangChain vs LangGraph：谁是agent落地最优解
author: Zilliz
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510179&idx=1&sn=dcdddcffd80931259f7087377438f232&chksm=fb9dec47e79b0ea3d23630bdb3a521af5eee63f6aba9437c90ebe804c1bd2cba0627b7538359&mpshare=1&scene=24&srcid=0912LzkQpXJtGdmXIcKcP5MZ&sharer_shareinfo=f302d721a02100e434c35c5a54dd8483&sharer_shareinfo_first=f302d721a02100e434c35c5a54dd8483#rd
---

大模型落地，要么做workflow、要么做agent，而无论哪一种，全都离不开框架，已经成为共识。而框架的选型，也直接决定着项目落地的效果好坏。

在众多开发框架中，LangChain 和 LangGraph 作为同一生态系统的重要组成部分，讨论度常年居高不下。

那么，到底该如何选型？先说结论，LangChain 专注于提供组件和 LCEL 编排能力，适合简单的一次性任务；LangGraph 则是构建有状态 Agent 系统的理想选择。两者相辅相成，可根据场景灵活选用。

具体原理，本文将从技术架构、应用场景等维度深入分析这两个框架，帮你做出决策参考。

## 01

## 框架概述与核心理念

### 1.1 LangChain：组件库与 LCEL 编排系统

LangChain 是一个专为开发大型语言模型应用而设计的开源框架，其核心是提供丰富的组件库和 LCEL（LangChain Expression Language）编排能力。

**核心特点：**

* **丰富的组件库**：提供文档加载器、文本分割器、向量存储、模型接口等多种组件
* **LCEL 编排能力**：通过 LangChain Expression Language 实现组件的灵活组合
* **易于集成**：与外部 API、数据库等工具无缝对接
* **成熟生态**：拥有完善的文档和活跃的社区支持

### 1.2 LangGraph：构建有状态 Agent 系统的框架

LangGraph 是 LangChain 生态系统中的专门库，专注于构建有状态的多智能体系统。它采用图（Graph）结构来表示任务和流程，其中节点代表操作或步骤，边表示节点之间的依赖关系。

**核心特点：**

* **图结构架构**：支持循环、回溯和复杂的控制流
* **状态管理**：拥有强大的中央状态组件，确保上下文连续性
* **多智能体支持**：天然适合构建多个智能体协作的场景
* **调试支持**：LangSmith Studio 提供图结构的可视化调试能力

## 02

## 技术架构深度对比

### 2.1 架构模式差异

**LangChain 的 LCEL 架构：**

```
```
# LangChain的LCEL编排示例from langchain.prompts import ChatPromptTemplatefrom langchain.chat_models import ChatOpenAIprompt = ChatPromptTemplate.from_template("请回答以下问题：{question}")model = ChatOpenAI()# LCEL链式编排chain = prompt | model# 运行链result = chain.invoke({"question": "什么是人工智能？"})
```

LangChain 通过 LCEL 提供了声明式的组件编排方式，适合构建简单的线性处理流程。
```

**LangGraph 架构特点：**

```
```
# LangGraph的图结构定义from langgraph.graph import StateGraphfrom typing import TypedDictclass State(TypedDict):    messages: list    current_step: strdef node_a(state: State) -> State:    # 节点A的处理逻辑    return {"messages": state["messages"] + ["处理A"], "current_step": "A"}def node_b(state: State) -> State:    # 节点B的处理逻辑    return {"messages": state["messages"] + ["处理B"], "current_step": "B"}# 创建图结构graph = StateGraph(State)graph.add_node("node_a", node_a)graph.add_node("node_b", node_b)graph.add_edge("node_a", "node_b")
```

LangGraph 支持复杂的图结构，可以处理循环、条件分支等复杂逻辑，特别适合构建 Agent 系统。
```

### 2.2 状态管理机制

**LangChain 的状态管理：**

* 通过 Memory 组件实现简单的状态保持
* 主要支持线性的上下文传递
* 适合单轮或简单多轮对话场景

**LangGraph 的状态管理：**

* 提供强大的中央状态管理系统
* 支持复杂的状态变更和回溯
* 能够处理长期的、多步骤的交互流程

### 2.3 执行模式对比

## 03

## 应用场景深度分析

### 3.1 适合 LangChain 的场景

#### 简单的一次性任务处理

LangChain 非常适合处理那些不需要复杂状态管理的一次性任务：

```
```
# 使用 LCEL 实现简单的文本翻译from langchain.prompts import ChatPromptTemplatefrom langchain.chat_models import ChatOpenAIprompt = ChatPromptTemplate.from_template("将以下文本翻译成英文：{text}")model = ChatOpenAI()chain = prompt | modelresult = chain.invoke({"text": "你好，世界！"})
```

实际应用：浏览器插件中的文本翻译功能，用户选中文本后一键翻译，不需要保持复杂状态。
```

#### 基础组件提供

LangChain 提供了丰富的基础组件，可以作为构建更复杂系统的基石：

* **向量存储接口**：与各种向量数据库的标准化接口
* **文档处理工具**：加载、分割、处理各类文档
* **模型接口**：统一的模型调用接口

实际应用：作为 LangGraph 应用的组件提供者，为图结构中的各个节点提供基础功能。

### 3.2 适合 LangGraph 的场景

#### Agent 系统

LangGraph 是构建 Agent 系统的理想选择：

```
```
# Agent系统的简化示例def agent(state):    messages = state["messages"]    # Agent思考并决定下一步行动    action = decide_action(messages)    return {"action": action, "messages": messages}def tool_executor(state):    # 执行工具调用    action = state["action"]    result = execute_tool(action)    return {"result": result, "messages": state["messages"] + [result]}# 构建Agent图graph = StateGraph()graph.add_node("agent", agent)graph.add_node("tool_executor", tool_executor)graph.add_edge("agent", "tool_executor")graph.add_edge("tool_executor", "agent")
```

实际应用：GitHub Copilot X 的高级功能采用 Agent 架构，能够理解用户意图，执行多步操作完成复杂编程任务。
```

#### 多轮对话系统

LangGraph 的状态管理能力使其非常适合构建复杂的多轮对话系统：

* **客服系统**：能够跟踪对话历史，理解上下文，提供连贯的回应
* **教育辅导系统**：根据学生的回答历史调整教学策略
* **面试模拟系统**：根据应聘者的回答调整面试问题

实际应用：Duolingo 的 AI 辅导系统能够根据学习者的答题历史和学习进度，提供个性化的语言学习体验。

#### 多智能体协作系统

LangGraph 天然支持多智能体协作：

* **团队协作模拟**：多个角色协同完成复杂任务
* **辩论系统**：多个角色持不同观点进行辩论
* **创意协作平台**：不同专业领域的智能体共同创作

实际应用：一些高级研究项目中，多个专业领域的 AI 智能体协作解决复杂问题，如药物发现、材料设计等领域。

## 04

## 选择决策框架

### 技术选型矩阵

## 05

## 写在结尾

LangChain 和 LangGraph 作为同一生态系统的不同部分，各自承担着不同的角色。LangChain 提供了丰富的组件和 LCEL 编排能力，适合简单的一次性任务；而 LangGraph 则提供了强大的状态管理和图结构支持，特别适合构建 Agent 系统和复杂的多轮交互应用。

**选择建议：**

* **选择 LangChain**：如果你的项目是简单的一次性任务，如文本翻译、内容优化等无状态操作
* **选择 LangGraph**：如果你需要构建 Agent 系统、多轮对话应用或多智能体协作系统
* **组合使用**：在实际项目中，往往可以结合两者的优势，使用 LangChain 提供的组件，通过 LangGraph 构建复杂的应用逻辑

最终，框架的选择应该基于具体的项目需求。随着技术的发展，LangChain 和 LangGraph 也在不断演进，关注官方文档和社区动态，能够帮助你更好地利用这些强大的工具。

## 彩蛋

## 好书推荐

本文的两位作者尹珉与张海立，是Milvus社区的北辰使者，也是最近热门新书《*LangGraph* 实战：构建新一代 AI 智能体系统》的作者。这本书解读了*LangGraph的基础原理以及各种落地实战案例，风格深入浅出、循循善诱。*

欢迎大家在评论区留言，聊聊你与Milvus与LangGraph的故事，24小时内点赞前三的朋友，后台私信，即可获得作者签名赠书，或者Milvus纪念品套装一份哦~

作者介绍

Zilliz 黄金写手：尹珉

推荐阅读

[Word2Vec、 BERT、BGE-M3、LLM2Vec，embedding模型选型指南｜最全](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510057&idx=1&sn=58fdfef27956475e9eb974e3bd822859&scene=21#wechat_redirect)

[n8n部署RAG太麻烦？MCP+自然语言搞定n8n workflow 的时代来了！](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510045&idx=1&sn=2e6a7faf45640fd1afeef100bb680089&scene=21#wechat_redirect)

[Embedding无敌？是做文档处理RAG最大的幻觉（含LangExtract+Milvus教程）](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510140&idx=1&sn=b4314a435c9e4e010856dbe63bac0ab3&scene=21#wechat_redirect)

[LLM、RAG、workflow、Agent，大模型落地该选哪个？一个决策矩阵讲透](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510010&idx=1&sn=55f15daab9e2eb4de4f274a9fe0d8abe&scene=21#wechat_redirect)