> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020112_LangChain/020112_核心知识点/LangChain Middleware与Agent抽象边界|LangChain Middleware与Agent抽象边界]]
---
title: 浅层Agent过时了？ 深度解读Agent 2.0的四大核心支柱与LangChain最新实践
author: AI 博物院
date:
url: https://mp.weixin.qq.com/s?__biz=MzIzNTcxOTU3OA==&mid=2247487620&idx=1&sn=a908eb0892c20f13a3b2d96ea6909053&chksm=e90c83a509e19fa9deacee88e329e772f8a6b53ac93983397eceb7ff35fa60acf7ef6d03aad8&mpshare=1&scene=24&srcid=1124KedthEkzGpJPK1sh69qK&sharer_shareinfo=2271491c3d3f9ff12559cf4f7517628d&sharer_shareinfo_first=2271491c3d3f9ff12559cf4f7517628d#rd
---

本文整理 Agents 2.0 的核心思想，并结合 LangChain 的 Deep Agents 最新能力与生态动态，帮助理解从“浅层循环”到“深度智能”的范式迁移。

* 浅层 Agent 以“LLM+工具循环”为核心，状态短暂、上下文易溢出、复杂任务易偏航。
* 深度 Agent 以“规划、委派、持久记忆、人类在环（Human-in-the-loop, HITL）”为四大支柱，支持长周期复杂任务与可恢复执行。
* LangChain 的 Deep Agents 基于 LangGraph 提供有状态编排、检查点、HITL 中断、子代理（Subagents）与文件系统/Store 等能力；DeepAgents 开源 harness 提供规划工具与可插拔后端。
* 适用场景：研究、异步编码、跨会话协作与长程任务；需权衡子代理与记忆调用带来的成本。

## 1. 从 Shallow Loops 到 Deep Agents

浅层 Agent（Agent 1.0）通常是“LLM + 工具循环”：接收提示→调用工具→生成响应→循环直到结束。其状态完全依赖上下文窗口，因而在以下情境下表现脆弱：

* 上下文溢出：工具输出充斥对话历史，挤压指令与目标，导致遗忘与漂移。
* 目标丢失：中间步骤噪声过多，偏离原始任务意图。
* 无恢复机制：陷入错误路径后缺乏停—退—改的系统化能力。
* 任务规模受限：适合 5–15 步的事务性任务，不适合 500 步的长周期任务。

## 2. Deep Agents：核心理念与支柱

深度 Agent（Agent 2.0）将“规划”与“执行”解耦，管理上下文窗之外的持久记忆/状态，并通过专长子代理完成复杂任务的分工协作。

Agents 2.0 Overview

它的工程架构可以抽象为“四大支柱”：

### 支柱一：明确规划（Explicit Planning）

浅层 Agent 通过思维链隐式计划；深度 Agent 使用“规划工具”显式维护计划（如 To-Do 列表），并作为执行的锚点。

每一步之间，代理都会审查并更新计划（pending / in\_progress / completed / notes）；失败后不是盲目重试，而是调整计划以吸收失败并保持聚焦。

### 支柱二：层级委派（Hierarchical Delegation / Subagents）

深度 Agent 采用 Orchestrator → Sub-Agent 模式：编排器分派任务到具有清洁上下文的子代理（Researcher / Coder / Writer 等），每个子代理独立运行工具循环并返回综合结果。

该模式避免“一个提示做所有事”的上下文污染，提升稳定性与扩展性。

### 支柱三：持久记忆（Persistent Memory）

通过文件系统/向量库/Store 等外部状态作为“单一事实源”，沉淀中间产物并实现跨会话记忆与检索，范式从“记住一切”转向“知道信息在哪儿”。

### 支柱四：极致上下文工程（Extreme Context Engineering）

深度 Agent 依赖高细度的操作规程与上下文治理：何时停下来先规划、何时生成子代理、工具使用规范、文件命名与目录结构标准、人类在环的协作格式等。

（以上规范是让“LLM 工具循环”升级为“可工程化系统”的关键。）

### 典型流程（文字时序说明）

示例：复杂研究请求→规划→子代理工具循环→中间产物沉淀→综合与交付→可恢复与反思。 该流程对应四大支柱：显式规划引导编排，编排器将任务委派给子代理；中间产物沉淀于持久记忆；关键节点通过人类在环进行审批与纠偏。 下图为“研究量子计算并为文件写摘要”的序列示意。

Agents 2.0 Sequence

## 3. LangChain 的 Deep Agents

基于 LangGraph 的深度 Agent 提供“有状态编排 + 持久化 + HITL 中断 + 子代理 + 文件系统/Store”的组合能力（参考：LangChain 博客与 DeepAgents 项目）。核心要点：

* 有状态编排与持久化：检查点（checkpoint）可恢复执行，跨会话记忆与状态管理，流式可观测中间步骤。
* 文件系统/后端：通过 `ls/read_file/write_file/edit_file/glob/grep` 等工具暴露可插拔的文件系统表面，支持虚拟文件系统与路径路由。
* 子代理（Subagents）：以参数配置自定义子代理，隔离上下文、提供专用指令，提升稳定性与专长执行。
* 人类在环（HITL）：基于 LangGraph 的 interrupt 中断机制，对敏感工具操作进行审批（允许 approve/edit/reject）。
* 长期记忆（Long-term memory）：通过 CompositeBackend/Store 等实现混合存储，部分文件跨线程持久，部分短暂；配合检索与记忆抽取（如 LangMem）。
* 中间件架构（Middleware）：规划工具、文件系统、子代理等以模块化中间件组合；

何时采用 Deep Agents（场景与取舍）

* 长周期、多步骤、多角色任务：研究、异步编码、跨会话协作、自动化运营。
* 需要持久化与可恢复：断点续跑、失败重试、跨线程记忆与检索。
* 团队协作与治理：人类在环审批、质量控制、防偏航与可观测性。
* 成本与复杂性权衡：子代理与记忆调用提升稳定性，但会增加令牌与工程复杂度；建议先以小任务试点评估。

## 总结

深度 Agent 的价值在于把“LLM 工具循环”升级为“可工程化的长期系统”：以规划—委派—持久记忆—人类在环为核心支柱，借助 LangGraph 的有状态编排与 DeepAgents 的中间件能力，支撑小时到天级的复杂任务，并提供可恢复与可观测。