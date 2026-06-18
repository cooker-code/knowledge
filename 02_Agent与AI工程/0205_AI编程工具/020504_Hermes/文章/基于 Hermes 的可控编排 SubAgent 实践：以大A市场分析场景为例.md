---
title: 基于 Hermes 的可控编排 SubAgent 实践：以大A市场分析场景为例
author: 小陆的大模型之旅
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4MzY5MDkyNg==&mid=2247483862&idx=1&sn=00bfedabe398f5f9534883da1fc27c63&chksm=9e830635f0edc1bca7d88a60aadd822d995972f6161da08d68d44dc6b5086d16432f66ac660d&mpshare=1&scene=24&srcid=0424poB1wSOpHULVA2B76RiY&sharer_shareinfo=e531d9dd565b9c23c88035406cbc9b57&sharer_shareinfo_first=e531d9dd565b9c23c88035406cbc9b57#rd
---

在agent实践中，复杂任务如果交给一个Agent处理，通常会很快失控：Prompt 越写越长，职责混在一起，工具调用越来越随机，结果也很难复盘。问题往往不在模型能力，而在执行链没有被约束。

Hermes 体系中 SubAgent 可以把复杂任务拆成多个可控节点。每个节点都是一个独立 Agent，只负责单一职责，主控 Agent 则只负责调度、汇总和汇报。

以A股市场分析场景为例，可以拆成 5 个角色：

* `stock-analysis（主）`：大A市场分析主控编排
* `news-impact-analyst（子）`：判断新闻是否值得继续跟
* `theme-mapper（子）`：把事件映射成产业链主题
* `equity-picker（子）`：从主题中挑候选股
* `risk-judge（子）`：风险复核

这样的拆分有一个明显好处：主控不再自己做所有事，下游也不会越权决定流程。每个节点只处理自己那一段输入，并输出固定结果，整条链路就会稳定很多。Agent 可以显式包装成工具，由主控调用。例如：

```
invoke_agent(agent="news-impact-analyst", input={...})invoke_agent(agent="theme-mapper", input={...})invoke_agent(agent="equity-picker", input={...})invoke_agent(agent="risk-judge", input={...})
```

这样一来，编排关系就是显式的，执行链也从“模型自由发挥”变成“主控智能体按步骤调度”。

这里有三个关键控制点。

第一，调用边界要收敛。`invoke_agent` 最好只允许调用预注册的自定义 Agent，而不是把所有内置能力、外部 Agent、其他系统节点都暴露出来。这样可以避免链路不断扩散。

第二，必须禁止自调用。比如 `stock-analysis` 不能再次调用 `stock-analysis`，否则很容易形成递归。

第三，输入输出要结构化。主控传给子 Agent 的内容要自包含，子 Agent 返回结果最好是严格 JSON。只有这样，下游节点才能稳定消费，上游也能做结果校验。

另外，可以针对智能体不同的定位，能力和skill可以按智能体的定位来，从而压缩上下文。对于主控 Agent，最好直接不给外部搜索能力；对于下游 Agent，只保留真正必要的工具，例如只给 `file:read`，不给 `web_search`。提示词中只能劝，权限控制才是真正的约束。

总结一下：在 Hermes 里，一个靠谱的多 Agent 系统，是一条被严格约束的执行链。主控负责编排，子 Agent 负责单点执行，系统层负责权限、结构和校验。把这些边界收紧之后，Agent 才会从“聪明但不稳定”，变成“可控且能上线”。