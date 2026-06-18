---
title: Prompt & Context Engineering 实用指南
author: AI大模型观察站
date: 
url: https://mp.weixin.qq.com/s/W2jjSyoaH39o8Rgo-oFDBQ
---

注：为保持专业术语一致性，以下保留了诸如 LLM、Prompt、Context、RAG、Chain-of-Thought、Tree-of-Thought、Sculpting、System prompt、Few-shot、Agent、Context Window 等英文术语，并在必要处辅以中文说明。

如果你一直在关注《Master LLMs》系列，你已经看过此前的旅程：从在“ What Even Is an LLM?”中建立直觉，到在“How Do LLMs Actually Work?”中理解其工作机制，再到在“Learn How To Steer Your AI Outputs”中学习关键原则。

现在，这篇文章将转向上手实践，聚焦在真正“要动手构建”时，如何与 AI 高效沟通的实用技艺（非会员链接）。

---

有件事多数人在使用 LLM 时并不了解：

> 模型非常聪明，但也非常“按字面理解”。

我花了无数小时在调试和微调那些“看起来本该奏效”的 prompt，才明白与 LLM 交谈并不像与人交流。

它比你想的既简单，又复杂。

---

## 为什么这件事此刻如此重要

在本系列的上一篇文章中，我概述了 Prompt 与 Context engineering 的关键概念，解释了背后的主要技术，并强调了掌握这些技能的必要性。简要回顾：

> Prompt engineering 不是为了得到“还行”的答案，而是为了每一次都能稳定地得到“尽可能准确且格式正确”的答案。这对构建有效的 AI 系统与 Agent 至关重要。

做到这一点，比听起来难：

* 只改错一个词，你的 JSON 解析器就会崩。
* 只挪错一行，模型就会忘记你一半的规则。

➡ 微小的改动，巨大的连锁反应。

---

## 如何有效地组织你的 Prompt

在把东西“搞坏”了足够多次、并读了大量关于 prompt engineering 的博客、论文与指南之后，我沉淀出一套在大多数场景下“就很管用”的结构。

请始终按如下顺序组织你的 prompt：

1. System prompt “The Constitution”（Role，Goal，Guardrails，Structure）
2. Few-shot examples（如需要）
3. Conversation history（已有对话历史）
4. Retrieved documents（你的 RAG context）
5. User query（具体任务）

为什么这个顺序如此重要？因为 LLM 对“近因”非常敏感。最后出现的内容最容易被重视。若你把真正的指令埋在中间，就别惊讶模型会忽略它。

### 1. The System Prompt

你的 Role/Persona、Goal、Guardrails 和 Answer Structure 必须定义在这里，而不是放在用户的 query 里。就在 prompt 的首位。

这相当于模型的“宪法”：不论来什么具体问题，都始终适用的持久规则。

根据你的需求，System prompt 还可以包含其他分段。例如，当你用 GPT 5 构建一个 Agent 时，可以在 System prompt 中加入如下部分：

* <context\_gathering>：阐明 context 收集循环的步骤。
* ：用于鼓励模型的自主性。
* <tool\_preambles>：规定工具调用前后需遵循的步骤。
* <code\_editing\_rules>：编写代码时需要遵守的规则。

一个简单的实践示例如下：

```
## Role  
You are FinBot, an expert financial analyst assistant.  
Professional, precise, never gives financial advice.  
  
## Goal    
Analyze data, identify trends, summarize reports, answer factual   
questions about markets.  
  
## Guardrails  
1. DO NOT EVER give financial advice.  
2. If asked for advice, offer factual alternatives instead.  
3. Only use information from provided context.  
  
## Answer Structure [Markdown Format]  
- Use markdown formatting  
- Provide a title  
- Provide a 3-line executive summary    
- Create a list of the top 3 keypoints  
  
## Answer Structure [Json Format] Better performance  
Always respond in valid JSON matching this format:  
{  
  "title": "string",  
  "summary": "string",  
  "keypoints": ["string", "string", "string"]  
}
```

➡ 注意它有多“明确”。没有歧义，每一个词都很重要。

> 注：不同的 LLM 对 System prompt 的结构与分段方式有偏好差异。比如，Claude Sonnet 4.5 的 prompting 指南推荐使用 XML-style 标签（如 `<role>...</role>`），而 GPT-5 更偏好 Markdown-style 的格式化，就像上面的示例。为获得更佳效果，值得查看你所选模型的推荐语法，因为不同模型对标签/分段支持略有差异。

将 System prompt 针对你的具体需求进行定制确实具有挑战，没有“一套通吃”的模板，但你仍能在各处找到一些有意思的示例。例如，这里整理了很多知名 Coding Agents 使用的 System prompt：

> GitHub - x1xhlol/system-prompts-and-models-of-ai-tools: FULL Augment Code, Claude Code, Cluely...

### 2. Few-Shot Examples

当你希望模型遵循特定的推理过程或任务逻辑时，请在实际 query 之前提供少量示例。

```
Example 1:  
Input: 解释为什么句子 “The sky cried all night” 属于修辞用法。  
Output: 它把“天空”拟人化。用“哭泣”描述下雨，为自然赋予了人的情感。  
  
Example 2:  
Input: 解释为什么短语 “Time is a thief” 属于修辞用法。  
Output: 这是一个隐喻。时间不可能“偷窃”，但它会“带走”我们生命中的片刻，就像小偷一样。
```

两个示例就能教会模式和结构；多于三个只会让 prompt 不必要地变长。

### 3. Retrieval-Augmented Generation（RAG）

RAG 听起来像很复杂的技术流行词，但本质上就是：在正确的时间，把正确的文档喂给模型，以便它准确回答用户请求。

我不会在这里展开如何为 LLM 或 Agent 搭建完整的 RAG，这属于更进阶的话题，将在未来文章讨论。现在只展示如何把文档“正确地”喂给模型。下面是 RAG 模式的概要：

Basic RAG architecture

1. 将文档切块并进行 embedding
2. 针对查询检索最相关的片段
3. 结合明确的“grounding 指令”一并提供给模型，例如：

```
<retrieved_documents>  
  <document_1>  
  1) "在 2024-08-01，该公司发货 1,000 台（单位）……"  
</document_1>  
<document_2>  
  2) "Q2 收入为 $3.4M……"  
</document_2>  
<document_3>  
  3) "Q3 收入为 $2.8M……"  
</document_3>  
</retrieved_documents>  
  
<instruction>  
只使用以上文档回答：公司的 Q2 收入是多少？  
</instruction>
```

这部分放在第 4 位：对话历史之后、最终用户 query 之前。模型先看到你的持久规则（System prompt），再看到所需的具体上下文（Retrieved documents），然后才是眼前的问题。

### 其他相关 Prompting 技术

Chain-of-Thought：用于需要推理的任务（如数学、逻辑或多步分析），我们可以要求模型在输出中展示它的思考过程。

在像 GPT-o1、DeepSeek-R1 这样的“推理向”模型出现前，这一技术被广泛使用。对于这些专门的模型，通常不再需要显式的 Chain-of-Thought 提示，但对“常规” LLM 而言，它仍然非常有用。

简单示例：

```
# Task  
求解：某商店苹果每个 $2，橙子每个 $3。  
如果我买 3 个苹果和 2 个橙子，总价是多少？  
请展示分步计算过程，并给出最终答案。
```

Response:

```
Step 1: 计算苹果的花费。  
3 个苹果 × $2/个 = $6  
  
Step 2: 计算橙子的花费。  
2 个橙子 × $3/个 = $6  
  
Step 3: 相加得到总价。  
$6（苹果） + $6（橙子） = $12  
  
Final Answer: $12
```

Tree-of-Thought（One Prompt）：

与其让模型沿一条思路一路走到黑，不如明确要求它生成多条解题分支，探索不同方法，再选出最佳方案。就像让人先画出三种解法草图，再决定采用哪一种。

它比 CoT 更能避免模型被“第一个看起来像样的答案”困住。它鼓励更广阔的思考，往往能找到比直线式方法更好的解。

示例摘自 Hubert 2023：

```
Imagine three different experts are answering this question.  
All experts will write down 1 step of their thinking,  
then share it with the group.  
Then all experts will go on to the next step, etc.  
If any expert realises they're wrong at any point then they leave.  
  
The question is:  
Bob is in the living room.  
He walks to the kitchen, carrying a cup.  
He puts a ball in the cup and carries the cup to the bedroom.  
He turns the cup upside down, then walks to the garden.  
He puts the cup down in the garden, then walks to the garage.  
Where is the ball?
```

Response:

```
Expert 1: 球在客厅。  
Expert 2: 球在厨房。  
Expert 3: 球在卧室。  
  
Expert 1: Bob 把杯子带到了卧室，所以球一定在杯子里。  
Expert 2: 啊，我看到了我的错误。是的，球在杯子里。  
Expert 3: 同意，球在卧室里的杯子里。  
  
Expert 1: 接着，Bob 在卧室把杯子倒扣。  
Expert 2: 然后 Bob 走到花园，把杯子放下。  
Expert 3: 因此，当 Bob 在卧室把杯子倒扣时，球应该已经从杯子里掉出来了。所以球在卧室，不再在杯子里。  
  
Expert 1: 你说得对。我之前错了。球在卧室，不在杯子里。  
Expert 2: 同意，球在卧室。  
Expert 3: 随后 Bob 走向车库，所以球仍然留在卧室。不在花园或车库。  
  
Expert 1: 完全正确，球还在卧室。  
Expert 2: 是的，球没有离开卧室。  
Expert 3: 因此，球在卧室，而不在花园或车库。  
  
三位专家一致认为：球在卧室。
```

Sculpting：

这是一种较新的（发表于 Oct 2025）高约束 CoT 变体，旨在通过禁止模型依赖有缺陷的“常识”来最大化推理可靠性。

它将逐步推理与严格、明确的规则结合起来：为模型设定一个正式 persona（例如“You are a pure mathematical reasoning engine”），禁止使用任何外部知识，并要求只使用题目中给定的数字与关系。

和 CoT 一样，它必须展示每一个中间步骤，并以清晰格式给出 Final Answer。

尽管该新方法据称优于现有方法，但其优势尚未被更广泛的 AI 社区充分验证。

示例：

```
You are a pure mathematical reasoning engine. You must solve the following  
problem.  
  
**Rules:**  
1. You must use ONLY the numbers and relationships given in the problem.  
2. You must NOT use any outside common sense or real-world knowledge that  
isn't explicitly provided.  
3. You must break down your calculation step-by-step. Show all  
intermediate arithmetic.  
4. After your reasoning, state your final answer clearly prefixed   
with "Final Answer:".  
  
**Problem:**  
[Question Text]
```

---

## 关于 Context Windows 需要牢记的事

关键在于“精确”与“简洁”。prompt 的每一部分都应该只说必须说的，用尽可能少的 token。你的指令越清晰、越紧凑，你就能为示例、检索到的上下文、输出结构以及用户的真实问题保留越多空间。

> 实践中，Prompt 与 Context engineering 的核心，就是用“最大清晰度+最小冗余”表达一切。

我通常按这个顺序处理：

1. Trim：删掉问候或冗余客套。
2. Summarize：把示例浓缩成精炼要点。
3. Retrieve：只取价值最高的片段。
4. Chain prompts：对于长任务进行分步串联，而不是硬塞到一个 prompt 里。

对于长对话，我通常保留最近几条消息的原文，把更早的内容做摘要。

---

## 最后想法

这些技术并不难实现。它们是消除歧义、塑造模型思考方式的简单工具： 用 System prompt 定义 Role 与 Guardrails；用 Few-shot examples 教会格式；用 RAG 让输出有据可依；用 Chain-of-Thought、Tree-of-Thought、Sculpting 等推理策略，鼓励深思而非条件反射。

掌握这些基础，可让 LLM 更少“不可预测”。

而且，随着模型能力增强、Agent 更加自主，它们完成任务所需的 prompt 会越来越短。这就是为什么养成“写短且清晰的 prompt”的习惯至关重要。 ➡ 清晰的思考带来清晰的指令，清晰的指令带来可靠的系统。

---

在后续文章中，我们将探索 agent loops、进阶 RAG 架构，以及可扩展的 context 策略。本文属于《Master LLMs: A Practical Guide from Fundamentals to Mastery》系列的一部分，我们会把复杂的 AI 概念拆解成清晰、可实操的课程。如果你感兴趣，可以收藏列表或关注我，以便及时获取我发布的每一篇新文章。

原文地址：https://medium.com/towards-artificial-intelligence/a-practical-guide-to-prompt-context-engineering-049f4b32fde8