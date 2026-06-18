---
title: Anthropic 新工具：Sonnet 遇到难题可以请教 Opus 了
author: 赛博禅心
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNDQxOTU2MQ==&mid=2247515301&idx=1&sn=b76cd0e46546f6ca10c725f7519de7dc&chksm=c37e20dae6a9b0a50b38d4a5e79a9cb3b9dd2f8634ea5a85b39fc030ab94bdceafc5e60de3fd&mpshare=1&scene=24&srcid=0412rXwny4nZRxOIukIpy1EU&sharer_shareinfo=c2f96658f67c07dc64c58b9a361c0eea&sharer_shareinfo_first=c2f96658f67c07dc64c58b9a361c0eea#rd
---

PRODUCT

Anthropic 发布了一个新的 API 工具，让 Sonnet 或 Haiku 在跑任务的过程中，遇到搞不定的决策时自动请教 Opus，拿到指导后继续干活。这个策略叫 **Advisor Strategy**，工具叫 **Advisor Tool**

效果是：**智能接近 Opus**，成本接近 Sonnet

Advisor 策略的工作方式：Sonnet 执行，遇到难题请教 Opus

## 反过来的 Sub-Agent 模式

行业里常见的多 Agent 模式是：大模型当指挥官，拆解任务分给小模型去执行。Advisor 策略把这个方向反过来了

**Sonnet（或 Haiku）作为 Executor** 全程执行任务，调用工具、读取结果、迭代推进。当它遇到一个自己判断力不够的决策点时，它会调用 Opus 作为 Advisor。Opus 拿到共享的上下文，返回一个计划、一个纠正、或者一个停止信号。然后 Sonnet 继续执行

Advisor 不调用工具，不产出面向用户的输出，只提供指导。前沿级推理只在 Executor 需要的时候介入，其余时间全部按 Executor 的价格计费

不是大模型指挥小模型干活，是小模型干活遇到难题请教大模型

这个设计的好处是：不需要任务拆解逻辑，不需要 worker pool，不需要编排框架。Executor 自己判断什么时候需要升级，整个过程在一次 API 调用里完成

## 评测数据

先看 Sonnet + Opus Advisor 的组合

### SWE-bench Multilingual

Sonnet + Advisor 比 Sonnet 单独跑提升了 **2.7 个百分点**，同时每个任务的成本降低了 11.9%。成本降低的原因是 Advisor 的介入让 Executor 少走弯路，减少了总 token 消耗

SWE-bench Multilingual：Sonnet + Advisor vs Sonnet Solo vs Opus Solo

### BrowseComp 和 Terminal-Bench 2.0

在 BrowseComp 和 Terminal-Bench 2.0 上，Sonnet + Advisor 同样超过了 Sonnet 单独跑，而且每个任务的成本更低

BrowseComp + Terminal-Bench：Sonnet + Advisor 的表现和成本

再看 Haiku + Opus Advisor 的组合，这个更有趣

在 BrowseComp 上，Haiku + Advisor 得分 **41.2%**，是 Haiku 单独跑（19.7%）的两倍多。跟 Sonnet 单独跑比，分数低了 29%，但成本低了 **85%**

BrowseComp：Haiku + Advisor vs Haiku Solo vs Sonnet Solo

对于高吞吐、需要平衡智能和成本的场景，这个组合很有吸引力。用 Haiku 的价格拿到接近 Sonnet 水平的结果

## 怎么用

API 层面非常简单。在 Messages API 请求的 tools 数组里加一个 `advisor_20260301` 类型的工具，指定 Advisor 模型是 Opus，设一个 `max_uses` 限制每次请求最多请教几次

整个模型交接在一次 `/v1/messages` 请求里完成，不需要额外的网络来回，不需要自己管理上下文传递。Executor 决定什么时候调用 Advisor，Anthropic 负责把精选的上下文路由给 Advisor 模型，拿到计划后 Executor 继续执行

计费方式：Advisor 的 token 按 Advisor 模型的价格算（Opus 的 $5/$25），Executor 的 token 按 Executor 模型的价格算（Sonnet 的 $3/$15 或 Haiku 的 $1/$5）。因为 Advisor 每次只生成一个短计划（通常 400-700 个 token），整体成本远低于全程跑 Opus

可以通过 `max_uses` 限制 Advisor 调用次数来控制成本。Advisor 的 token 消耗在 usage 中单独报告

## 早期用户怎么说

在复杂任务上做出了更好的架构决策，在简单任务上没有任何额外开销。计划和执行轨迹完全是两个级别

Eric Simmons，Bolt CEO

我们看到了 Agent 轮次、工具调用次数和整体分数的明确改善，比我们自己构建的 planning 工具效果更好

Kay Zhu，Genspark 联合创始人兼 CTO

在结构化文档提取任务上，Advisor 让 Haiku 4.5 按需请教 Opus 4.6，达到了前沿模型的质量，成本低 5 倍

Anuraj Pandey，Eve Legal 机器学习工程师

## 几个信号

第一，这是 Anthropic 第一次在 API 层面提供模型间协作的原生支持。之前想让 Sonnet 和 Opus 配合，你得自己写编排逻辑、管理上下文传递、处理两次 API 调用的状态。现在一个 tool 声明就搞定

第二，定价逻辑很巧妙。Advisor 每次只输出 400-700 个 token 的短计划，按 Opus 价格算也就几分钱。但这几分钱的指导可以让 Executor 少走弯路，减少总 token 消耗。所以出现了「加了 Advisor 反而总成本更低」的现象

花几分钱请教一次 Opus，省下来的是 Sonnet 走弯路烧掉的几毛钱

第三，Haiku + Opus Advisor 的组合值得关注。BrowseComp 41.2% 的成绩用 Haiku 的价格拿到，比 Sonnet 单独跑便宜 85%。对于大规模、成本敏感的 Agent 部署场景，这个组合可能比 Sonnet 更合适

第四，时间线继续加密。Mythos、Managed Agents、Advisor Tool，Anthropic 在一周内连续发布了最强模型、Agent 基础设施平台、模型间协作工具，产品线的密度在快速增加

参考材料

The Advisor Strategy 官方博客  
`https://claude.com/blog/the-advisor-strategy`

Advisor Tool API 文档  
`https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool`