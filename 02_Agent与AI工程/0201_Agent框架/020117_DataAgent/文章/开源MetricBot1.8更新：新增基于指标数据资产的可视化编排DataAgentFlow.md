---
title: 开源MetricBot1.8更新：新增基于指标数据资产的可视化编排DataAgentFlow
author: DClusterAI
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk0NDMyMzEwNQ==&mid=2247484886&idx=1&sn=6e99611170c4c569bee2b07eb4314216&chksm=c2644e678efa575a9f7d122f7dd94f31c375f11225691eee8ba75faec0a06f996bebff5e8383&mpshare=1&scene=24&srcid=0317eWGp9WRVFwEdA7QGZdHO&sharer_shareinfo=53c919fba0aa643c2ab4aa3e50193efc&sharer_shareinfo_first=53c919fba0aa643c2ab4aa3e50193efc#rd
---

MetricBot是基于指标数据资产+langgraph+langchain的可视化编排的数据分析DataAgentFlow开源项目，支持window客户端下载使用。经过几个月的努力，在MetricBot的1.8版本上实现了langgraph+langchain可视化编排能力。

为什么要做可视化编排DataAgentFlow

当前很多AI数据分析的方向还是在为了攻克AI问数的准确性和可用性上，但根据我自己的思考，我们其实可以把AI问数当成是为了完成一个agent skill，然后进行一个封装提供过大模型调用，作为一个核心的节点。

实际业务中的数据分析是需要大量的规范的口径统一的指标数据进行分析的。如果要跟AI结合起来的话，实际上是要给AI提供多种主题的数据，才能得到一份可以参考的数据分析报告。类似Dify，N8N这些优秀的开源项目，实际上是有可以构建的方案，它们支持多个节点通过sql查询所需要的数据，然后再喂给大模型进行深度分析。

有个需要解决的问题就是，并不是所有人对sql都很熟悉，并且每次的数据分析都需要找很多的数据表和写大量的sql。这就比较难做的指标资产的沉淀和AI数据分析的飞轮闭环。

但如果能持续构建出符合业务数据决策的指标体系，再结合可视化DataAgent的构建能力，帮助业务能快速构建出个性化的DataAgent。这样或许在能增效的同时，还能产生更大的数据决策价值。

基于此，在metricbot本来就具备指标数据资产构建的同时，引入可视化DataAgent的构建能力，就成了后续持续迭代的大方向。

如何做

以langgraph+langchain为核心，构建图的可视化工作流执行引擎以及采用langchain的ReAct执行策略。

langgraph

可以通过图结构来组织任务流程， 支持复杂的控制逻辑和状态管理，适用于需要多步骤协作、动态决策和持久化的 AI 应用场景。

在实现更多业务需求时，我们可以实现多种符合业务需要的节点功能。

ReAct

react的核心在于构建了一个动态的‌思考‌（Thought）循环：

‌‌ ‌ ‌ ‌ ‌   思考‌（Reasoning）：智能体先分析任务目标，拆解问题，规划下一步该做什么。例如面对“查询明天深圳到杭州最便宜航班并预订”的指令，它会先推理出需要确认日期、查询航班、比价、最后预订的步骤。

‌‌ ‌ ‌ ‌ ‌    行动‌（Acting）：基于思考结果，智能体调用具体工具（如搜索引擎、API 接口、计算器）执行操作，获取新信息或改变环境状态。

‌‌ ‌ ‌ ‌ ‌    观察‌（Observation）：行动产生的结果（如搜索到的航班列表、API 返回的订单号）被反馈给智能体，作为下一轮思考的输入，直到任务完成。

langchain给我们提供了ReAct策略的使用入口，我们需要做的就是实现多种skill供大模型调用

MetricBot当前进展

当前已经跑通了从页面构建DataAgentFlow到问答分析的流程，具体如下：

1.agent管理

提供了快速查询使用的agent展示页面，支持编辑和删除，点击编辑即可进入可视化流程编排和问答页面。

2.agent flow构建

新增和编辑页面采用flet python实现，可通过可视化动态增删和配置节点信息。

目前支持了大模型节点和指标快速查询节点、sql查询节点。基本上能实现不同业务的数据分析报告生成和自定义AI问答agent。

3.agent 问答

构建的agent flow 一般需要包含开始节点-数据查询节点-大模型节点-结束节点（其中开始节点和结束节点必须要有），

在开始节点配置中将启动问答打开，同时设置好其他节点后，点击执行按钮就进入了对话页面。

后续规划

大模型ReAct支持

agent skill 扩展

flow 节点扩展

AIBot 功能实现

本地大模型支持

体验优化

关于MetricBot

MetricBot是基于指标数据资产+langgraph+langchain的可视化编排的数据分析DataAgentFlow开源项目，通过Text2DSL2SQL等指标语义模型实现sql生成和chatBI。持续迭代的核心功能有DataAgentFlow、指标口径问答、chatBI、sql智能生成、指标资产&血缘可视化、指标预警等。

如果对您有帮助，帮帮忙点下star。

如果使用有问题，方便留言区沟通与讨论。

gitee :

https://gitee.com/zhenglv123456/metric-bot

https://gitee.com/zhenglv123456/dcluster、