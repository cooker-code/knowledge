---
title: 所有做 AI Agent 团队建议都接一下 LangSmith
author: AI设计湿
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxMjg3NzI5OQ==&mid=2247485052&idx=1&sn=c35a50401045362159f66738302fef67&chksm=9a3f96d7b32e7b89129ba0a90a58e3d03d907f542fdbf13309234eb6fb1c1ed7ea2c25a8c52f&mpshare=1&scene=24&srcid=1209JU2oN7l1RorWqW7byxm7&sharer_shareinfo=85af53c2708280f7287cd26198de8576&sharer_shareinfo_first=85af53c2708280f7287cd26198de8576#rd
---

> hi 我是 Link，Agent 全栈工程师

我之前参与的项目使用 Java 技术栈来构建 LLM 应用，有一个非常大的痛点无法做到 agent tracing，只能通过 log 来调试和 debug 功能，日志（log） 的最大问题是非常不可视化，看起来太累，以至于我们经常不怎么管我们开发的功能是否 OK，全部交由产品测试，非常黑盒！

但后来切换到 python 后，选择了使用 langgraph 技术栈来实现 agent 或 workflow 功能开发，则可以通过集成 langsmith 来实现 tracing。如下图所示：

在监控选项卡中，可以查看一段时间内的汇总统计数据。这些统计数据包括特定于 LLM 的指标，如延迟、首次令牌时间、成本、令牌、反馈等。

### LangSmith 特点

LangSmith 是一个完整的平台，用于测试、调试和评估大型语言模型应用。也许，它最重要的功能是 LLM 输出评估和性能监控。

#### 1. 快速集成

程序员可以在几分钟内开始用 LangSmith 进行实验，而不是几个小时。这意味着一个小团队或独立开发者可以在一个周末内原型开发一个 AI 应用，并有可能在周一前发布付费服务。

只需要几个步骤就能集成到你的应用：

#### 2. 质量保证

通过严谨的评估工具，公司可以避免错误 AI 发布带来的尴尬和成本。LangSmith 允许你根据内置标准（如帮助性、连贯性）或自然语言的定制评估来核对，比如"输出内容是否符合预期"。

#### 3. 实时监控与可视化

LangSmith 使用跟踪记录几乎所有 LLM 运行的各个方面。这些指标包括延迟、token 数量、运行成本以及各种元数据。网页界面允许你快速根据错误百分比、延迟、日期，甚至使用自然语言的文本内容筛选运行。这意味着，比如说，如果 AI 对真实客户的回复出现故障，你可以在几个小时内推送修复。  

#### 4. Datasets 数据

LangSmith 的另一个重要功能是数据集。它们可用于在部署前针对一组标准化示例改进 LangChain、代理或模型。例如，我们可能有一个包含两列的 CSV 文件——特定格式的问题和答案。

通过将该文件转换为参考数据集，我们可以指示大型语言模型使用上述质量保证指标来评估自身输出。

### 集成 Langsmith

**1. 创建 Project**  
登录 https://smith.langchain.com/， 点击右上角 New Project  

**2. 获取 API Key**  
点击 New Project 后弹窗点击 Generate API Key 就会获得一个 API Key  

**3. 配置环境变量**

```
# LangSmith Configuration  
# 在temporal workers里启动调用的agent，将这些环境变量在启动workers的terminal里export:  
LANGCHAIN_API_KEY=lsv2_ptxxx  
LANGCHAIN_PROJECT=xxxxx  
LANGCHAIN_TRACING_V2=true
```

**4. 集成到代码**

```
config: dict[str, Any] = {  
    "configurable": {"thread_id": thread_id},  
    "run_name": f"CampaignAgent_{str(campaign_id)[:8]}",  
    "tags": [  
        "campaign_agent",  
        f"workspace:{workspace_id}",  
        f"campaign:{campaign_id}",  
    ],  
    "metadata": {  
        "workspace_id": workspace_id,  
        "campaign_id": campaign_id,  
        "thread_id": thread_id,  
        "initiated_by": str(initiated_by) if initiated_by else None,  
    },  
}  
current_state = await graph.aget_state(config)
```

**5. 测试**  
在你的 LLM 应用中发起对 LLM 的调用 case，然后到后台就可以看到 tracing 记录。

### 为什么需要评测你的应用

在机器学习应用开发中，你需要**收集数据、训练、微调、测试和部署模型**——步骤定义得很清楚。而使用大语言模型应用时，通常你会从 LLM 供应商那里的现成模型开始。微调？这可能会很贵。所以，你会非常专注于设计合适的提示词——关键是向你的 LLM 应用提出正确的问题。可以把它想象成需要大量提示词来测试，就像你需要大量数据来做一个好的机器学习模型一样。

不过，评估提示词时你面对的是对话消息，而不是数字。因此，通常测量误差或准确性的方法，比如 MSE 或交叉熵，在这里行不通。此外，想象一下要阅读每一个输入和输出进行评估——如果你有成千上万个提示词要评估，这需要几天时间。

所以，你需要一个工作流，专注于高效创建和测试这些提示词，看看你的 LLM 应用表现如何。

### 如何评测

**1. 数据集**  
可以使用 SDK、文件导入、Runs 创建在线 dataset  

**2. 评估函数**  
定义一个评估函数，判断输出值是否与期望值相等，相等则评分为1，不相等则评分为0。

```
from langsmith.evaluation import LangChainStringEvaluator  
  
# 使用 LangSmith 内置的评测器  
results = evaluate(  
    run_agent,  
    data=dataset_name,  
    evaluators=[  
        # 自定义评测器  
        tool_selection_accuracy,  
          
        # LangSmith 内置评测器  
        LangChainStringEvaluator("cot_qa"),  # Chain of Thought QA  
        LangChainStringEvaluator("qa"),       # 基础 QA  
        LangChainStringEvaluator("context_qa"),  # 带上下文的 QA  
    ],  
    experiment_prefix="agent-with-builtin-evaluators"  
)
```

**3. 查看评测结果**

```
# 在代码中查看  
print(results)  
  
# 或者在 LangSmith UI 中查看  
# https://smith.langchain.com/  
# 进入你的项目 -> Datasets -> 选择数据集 -> 查看评测结果
```

### 最后

LangSmith 是 Agent 开发的一个重要的 infra，在基于 LLM 的应用开发过程中无论是调试、监控还是评测，都能够带来非常大的帮助，本文只是浅尝辄止的介绍，希望后面有机会能够分享更多 LangSmith 用法。

有兴趣的可以关注我，带你学习 AI 智能体开发！