---
title: Day 4｜如何给智能体设计“任务规划器”？
author: Wise玩转AI
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5MzY4NjM0MA==&mid=2247483909&idx=1&sn=3f332c12de29ec050183fef1d016e966&chksm=c1ca30d4854e2a2648da146ff1b9d7df449acae8aeec7f25b6eca6f5de944eb9a9ad5ab82ade&mpshare=1&scene=24&srcid=1122pjUhrraDxflYGoXz7VpX&sharer_shareinfo=67b507cd1c216a83197113cae48be72c&sharer_shareinfo_first=67b507cd1c216a83197113cae48be72c#rd
---

从 0 构建能真正执行任务的 Planner**，Planner 才是****智能体****系统的灵魂，它决定了「这事该怎么做」。没有 Planner，所谓智能体只是在“长篇大论”。**我拆解了一下好的智能体系统，得到一个非常核心的结论：**智能体真正的能力不是“回答问题”，而是“拆解任务 → 分步执行 → 完成目标”。而这三件事，全部依赖 Planner。**今天我们把智能体中最重要、但最容易被忽视的部分讲清楚：**任务规划器（Task Planner）**。

# 🚀 1. 为什么智能体必须要 Planner？

你让模型做任务，它不会自动规划，而是直接给结果。

例如：

> “帮我做一份明天的行业日报。”

模型通常会：

* 直接写日报（但它没有真实数据）
* 不会先爬数据
* 不会核实信息
* 不会处理异常
* 不会构建任务链路

这就是普通 LLM 与智能体最大的差异：

> LLM：给你答案  Agent：为你完成任务

`而“完成任务”需要三步：`

1. 明确目标
2. 拆解子任务
3. 给每个子任务分配工具与行为

→ 这正是 **Planner 的职责**。

# 🚀 2. 一个合格的任务规划器要做什么？

Planner = 任务拆解器 + 流程指挥官。

它必须完成以下职责：

### ✔ 1. 理解用户意图

把“模糊意图”变成“可执行目标”。

### ✔ 2. 将任务拆分成可执行步骤

`步骤 1：爬取 6 个数据源``步骤 2：提取结构化信息``步骤 3：生成内容摘要``步骤 4：组装日报模板``步骤 5：推送给用户`

### ✔ 3. 将步骤映射到工具

* 爬取 → WebCrawler
* 摘要 → LLM Summary Tool
* 生成内容 → LLM Writer
* 推送 → Webhook

### ✔ 4. 控制执行顺序

这是智能体最重要的能力之一：

* 顺序执行
* 分支
* 循环
* 条件判断

### ✔ 5. 异常处理

如爬虫失败 → 重试 → 换源。

没有异常处理的智能体都是玩具。

# 🚀 3. Planner 的标准架构

标准结构（非常稳定）：

```
```
┌─────────────────────────┐│        User Intent       │└─────────────────────────┘             │             ▼┌─────────────────────────┐│ Intent Interpreter (LLM) │   ← 理解需求└─────────────────────────┘             │             ▼┌─────────────────────────┐│    Task Planner (LLM)    │ ← 拆解步骤、构建流程└─────────────────────────┘             │             ▼┌─────────────────────────┐│   Tool Router / Selector │ ← 决定用哪个工具└─────────────────────────┘             │             ▼┌─────────────────────────┐│   Executor / Action Hub  │ ← 执行每个步骤└─────────────────────────┘
```
```

> Planner 是中枢。其他都是围绕它来工作的。

🚀 4. Planner 的 Prompt（工程级模板）

开发智能体最常用的 Prompt 模板。你可以直接用于：

* API 项目
* CrewAI / AutoGen
* 企业内部 AgentHub
* Notebook 开发

## ✅ **任务规划 Prompt 模板（可直接复制）**

```
```
你是一名资深 Task Planner，负责将用户的需求拆解为可执行的步骤。必须遵守以下规则：1. 每一步必须是可执行行为（Action），禁止输出抽象描述。2. 每一步必须关联目标，并说明需要的工具。3. 保持步骤数量在 3～8 之间，不可过多或过少。4. 避免任何不必要的解释。5. 输出内容必须使用 JSON 格式。请将用户需求转换为如下结构：{"goal": "最终任务目标","steps": [    {"id": 1,"action": "具体可执行动作","tool": "需要使用的工具名称（或 none）","input": "该步骤的输入来源"    }  ]}现在开始，根据用户需求生成任务规划。
```
```

# 🚀 5. 实战示例：自动生成行业日报

用户输入：

> “做一份 AI 行业日报，并在 9 点推送飞书。”

Planner 输出

```
```
{  "goal": "生成并推送 AI 行业每日简报",  "steps": [    {      "id": 1,      "action": "从指定的 6 个数据源爬取新闻数据",      "tool": "WebCrawler",      "input": "data_sources"    },    {      "id": 2,      "action": "将新闻内容转换为结构化数据",      "tool": "ContentExtractor",      "input": "raw_html"    },    {      "id": 3,      "action": "对结构化信息执行摘要提取",      "tool": "LLM_Summarizer",      "input": "structured_data"    },    {      "id": 4,      "action": "生成日报文案",      "tool": "LLM_Writer",      "input": "summaries"    },    {      "id": 5,      "action": "通过飞书 Webhook 推送日报",      "tool": "FeishuWebhook",      "input": "final_report"    }  ]}
```
```

这一点非常关键：**每一步都是可执行的动作，而不是“写一写、整理一下”这种无意义表达。**

# 🚀 6. Planner 的代码结构（Python 版 ）

下面是非常适合新手的 Planner 结构，可以直接扩展到你的智能体系统。

```
```
from openai import OpenAIimport jsonclient = OpenAI(api_key="YOUR_KEY")PLANNER_PROMPT = """（此处放前面的 Planner 模板）"""def plan_task(user_input):    response = client.chat.completions.create(        model="gpt-4.1",        messages=[            {"role": "system", "content": PLANNER_PROMPT},            {"role": "user", "content": user_input}        ]    )    plan = response.choices[0].message["content"]return json.loads(plan)    plan = plan_task("自动生成 AI 行业日报并推送飞书")    print(plan)
```
```

这就是一个「最小可执行 Planner」。

你可以继续迭代：

* 失败重试
* 工具映射
* 状态管理
* 任务缓存
* 任务检查器（Verifier）

很快就能发展成一个完整 Agent。

# 🚀 7. Planner最佳实践

### ✔ 最佳实践 1：规划与执行必须分离

Planner 做规划，不做执行。否则会非常混乱。

### ✔ 最佳实践 2：输出必须为结构化格式

JSON、YAML 都可以。这让你的系统可控、可测试、可复现。

### ✔ 最佳实践 3：限制步骤数量

3～8 步之间最合适。太多 → 模型迷路，太少 → 粗糙不可执行

### ✔ 最佳实践 4：提前定义工具能力说明（Toolspec）

模型不知道工具能做什么。必须在 Planner Prompt 中明确写：

```
ToolName: 能做什么 → 输入 → 输出
```

`让模型能正确选择工具。`

### ✔ 最佳实践 5：引入 Verifier（审查器）

验证 Planner 生成的步骤：

* 是否可执行
* 格式是否正确
* 步骤是否覆盖任务目标

> 这是大模型时代的“单元测试”。

# 

# 

Planner 决定智能体的上限，如果你准备做智能体产品或 AI 工具型内容创作：**先把 Planner 练好，你才能构建真正的智能体系统。否则永远停留在 Prompt + 工作流的层面。**