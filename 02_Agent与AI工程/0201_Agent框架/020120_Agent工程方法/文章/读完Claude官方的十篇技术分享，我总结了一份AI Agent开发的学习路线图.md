---
title: 读完Claude官方的十篇技术分享，我总结了一份AI Agent开发的学习路线图
author: 岛上的狮子
date: 
url: https://mp.weixin.qq.com/s?__biz=MjM5Nzg5MTc5NA==&mid=2247484157&idx=1&sn=1edd25da51e0e63190208fe54c956b4b&chksm=a79ca0178350fcedcc719679ce58784cc9f7f2d5acfa209ee63690b0ac92b47c95c674a45817&mpshare=1&scene=24&srcid=1030FZXcUAAg89NnIx1oYv0v&sharer_shareinfo=fd7dadc6f2bdf1b9d0846e0f8b2a122f&sharer_shareinfo_first=fd7dadc6f2bdf1b9d0846e0f8b2a122f#rd
---

# **Anthropic 官方分享过哪些 AI Agent 开发指南？**

## ✔引言

最近在Agent构建过程中遇到挺多问题，遂决定重新梳理一下思路。

学习Agent，Claude官方的资料不可不读，他们曾分享了相当多的Agent领域的技术博客和视频，其中包括知名的 **《Building Effective AI Agents》** 等十余篇；

我把这些多篇技术博客与视频进行整合，梳理出了这篇不一定完整的Agent开发的学习路线图，希望能帮助你了解如何学习构建**强大、可靠且高效的AI Agent系统**。

> "*本文档内容由Rosa基于Anthropic截止到25年9月的资料进行手工整理，在AI的辅助下完成编辑。*

## ✔**第一部分：核心理念与准备工作**

在开始构建之前，理解 Anthropic 提倡的核心理念至关重要。这能帮助您在开发过程中做出正确的技术选型和架构决策。

1. **理解智能体系统（Agentic Systems）的两种形态**

* **工作流（Workflows）**: 系统中的 LLM 和工具由预定义的代码路径进行编排。它就像一条固定的流水线，每一步都由您的代码精确控制。

  + 适用场景：任务路径明确、可预测。例如：接收用户邮件 -> 提取关键信息 -> 分类意图 -> 调用相应工具 -> 生成回复草稿。（例如：提取文本 -> 翻译 -> 格式化）。

  **智能体（Agents）**: LLM 在一个循环中动态地指导自己的流程和工具使用，直到找到解决方案。它更具自主性，是真正的“思考者”和“执行者”。 - 适用场景：开放式、无法预知所有步骤的复杂任务。例如：根据一份设计文档编写一个新功能、对一个全新的数据集进行探索性分析、在游戏中从零开始学习通关。

2. **开发核心原则：从简开始，按需迭代**

* **寻找最简单的解决方案**：并非所有问题都需要复杂的 Agent。很多时候，一个精心设计的单次 LLM 调用，结合检索增强（RAG）和上下文示例，就已经足够。过度设计是 Agent 开发中最常见的陷阱。

  **仅在必要时增加复杂性**：只有当简单的方案无法满足需求，且您能通过评估证明增加复杂性（如多步工作流或自主 Agent）能带来显著性能提升时，才应采纳。Agent 系统通常用更高的延迟和成本换取更好的任务性能，务必权衡。

  **构建可受益于模型进步的护城河**：您的产品价值不应仅仅是模型本身的能力。优秀的 Agent 应用应该是：“当模型变得更聪明时，我的产品会变得更好”，而不是“当模型变得更聪明时，我的护城河就消失了”。您的核心价值应在于围绕模型的独特编排、数据、工具和工作流。

3. **何时应该使用 Agent？（实用的检查清单）**

* **任务是否复杂？** 如果一个人类可以清晰地列出完成任务的每一步，那么您可能只需要一个工作流。Agent 适用于那些您知道最终目标，但具体路径不清晰的场景。

  **任务价值是否足够高？** Agent 会消耗更多资源（Token、时间和计算）。请确保您将其用在能产生高价值的任务上，例如显著提升高技能员工的效率，或直接产生收入。

  **任务的各个部分是否可行？** 您能否为 Agent 提供完成任务所需的工具和信息？如果 Agent 需要访问一个不存在的 API，或者信息被完全隔离，那么它将无能为力。

  **错误的成本是否可控？** 如果一次错误会导致灾难性后果（例如，在生产环境中错误地删除了数据库），那么独立的 Agent 可能不适合，需要引入“人在回路”（Human-in-the-loop）进行关键步骤的审批。如果错误可以轻松恢复（例如，一次失败的网页搜索可以重试），则更适合 Agent。

4. **建立正确的开发者心态：“像 Agent 一样思考”**

   这是Anthropic 工程师强调的构建高效 Agent 最重要的原则之一。

* 核心思想：您必须设身处地地从 Agent 的视角体验它的“世界”——即它能使用的工具和从工具返回的信息。

  实操方法：在调试时，进行一次思想实验。想象自己被关在一个没有窗户的房间里，唯一的输入是屏幕上显示的文本（如同 Agent 看到的上下文），唯一的输出是调用一组定义好的工具。您能完成任务吗？

  例如，在“Claude 玩宝可梦”项目中，工程师们会闭上眼睛一分钟，然后睁开眼看一秒钟屏幕截图，再闭上眼思考下一步操作。这种共情能让您立刻发现工具描述的模糊之处、信息返回的不足以及 Agent 面临的真正困境。

> "深入学习建议：
>
> 要深入理解 Agent 与工作流的区别、何时使用 Agent 的清单，以及“像 Agent 一样思考”的心态，可以观看视频 **《Building Effective Agents》** 和 **《Prompting for Agents》**。

---

## ✔**第二部分：构建智能体的四大基石**

一个强大的智能体离不开四大核心能力：知识、行动、思考和记忆。

#### **基石一：赋予知识 —— 高效的检索增强生成（RAG）**

1. **传统 RAG 的局限性**：切分文档导致上下文丢失。
2. **Anthropic 的解决方案：上下文检索（Contextual Retrieval）**

* **核心思想**：在编码前，使用 LLM 为每个文本块生成一段简明的“上下文描述”，显著提升检索准确率。

  **效果**：检索失败率降低 **49%**，结合重排序（Reranking）可降低 **67%**。

> "深入阅读建议：
>
> 详细实现方法请阅读 **《Contextual Retrieval in AI Systems》** 全文及其中的cookbook示例。

#### **基石二：赋予行动能力 —— 设计与优化工具（Tools）**

工具是 Agent 与世界交互的桥梁。Anthropic 提供了多种强大的内置工具，并强调了工具设计的重要性。

1. **Anthropic 提供的强大工具示例**：

* **代码执行（Code Execution）**：为 Claude 提供一个安全的、沙箱化的计算环境。当面对需要精确计算、数据分析或可重复执行的任务时，Claude 会选择编写并执行 Python 代码，而不是直接回答。这使得结果更可靠、可审计。

  **网页搜索（Web Search）**：这不是简单的“搜索框”，而是一个智能化的研究过程。Claude 会根据任务自主决定搜索什么、搜索多少次、如何深入挖掘，并为所有发现提供引用来源，确保信息的可追溯性。

  **MCP 连接器（MCP Connector）**：允许你的 Agent 连接到庞大且不断增长的第三方工具生态系统（如 Asana, Zapier, Cloudflare），极大地扩展了 Agent 的能力边界。

2. **高效工具设计的关键原则**：

* **选择正确的工具**：创建少数几个有思想的、面向工作流的工具，而不是为每个 API 端点都创建一个工具。应创建少数几个有思想的、面向工作流的工具，整合多个底层操作。

  + 反例：`list_users`, `list_events`, `create_event` 三个独立工具。

    正例：一个 `schedule_event` 工具，它在内部完成查找参会者有空时间、预定会议室、发送邀请等一系列操作。

  **命名空间（Namespacing）**：为工具添加清晰的前缀（如 `asana_projects_search`, `jira_issues_get`），明确功能边界，避免 Agent 在功能重叠的工具间混淆。

  **返回有意义的上下文**：工具的返回值应优先考虑对 Agent 有用的自然语言信息，而不是低级的技术标识符（如 `user_uuid: "u-1a2b3c"`）。Agent 更容易理解和处理 `user_name: "张三"`。

  **优化 Token 效率**：为可能返回大量数据的工具实现分页、截断等功能。当截断时，返回有帮助的提示，如：“结果已截断，共找到 150 条记录，当前显示前 20 条。请使用更精确的查询或翻页参数。”

  **精心编写工具描述（Prompt-Engineering）**：这是最常被忽视但效果最显著的一点。要像给团队新成员写函数文档一样，清晰、明确、无歧义地描述工具的功能、每个参数的含义和预期的格式。

> "深入学习建议：
>
> 观看 **《Building Blocks for Tomorrow’s AI Agents》** 可以直观了解代码执行、网页搜索等工具的强大功能。而 **《Writing effective tools for AI agents》** 则深入讲解了如何从零开始设计、评估和优化你自己的工具。

#### **基石三：赋予思考能力 —— “思考”工具（"think" tool）**

为了处理复杂任务，Agent 需要一个“思考空间”来规划、分析和反思。

* 它是什么：一个特殊的工具，不执行任何外部操作，只是让 Agent 有机会在行动前或行动后，将自己的思考过程、分析、计划记录下来。

  与“扩展思考”的区别：“扩展思考”是在生成任何响应之前进行的深度规划；而 `think` tool 则是在多步工具调用的过程中，用于处理新信息和进行决策。

  **何时使用**：在需要分析工具输出、遵循复杂策略、或进行错误代价高昂的顺序决策时。

  **如何实现**：在工具列表中添加 `think` 工具定义，并在系统提示中通过示例指导 Agent 何时以及如何使用它。

> "深入阅读建议：
>
> 获取 `think` tool 的标准定义和最佳实践，请阅读 **《The "think" tool: Enabling Claude to stop and think》**。

#### **基石四：赋予记忆 —— 克服上下文窗口限制**

这是从“Claude 玩宝可梦”项目中提炼出的关键实战技巧。

* **问题：** Agent 在长时间运行时会超出模型的上下文窗口（如 200k tokens），导致“失忆”，忘记任务的初始目标和之前的操作。

  **解决方案：** 长短期记忆结合

  长期记忆（Long-term Memory）：为 Agent 提供一个外部知识库（如一个简单的文本文件或数据库）。Agent 可以将关键信息、总体目标、已完成的里程碑、学到的经验教训等写入这个文件。这就像电影《记忆碎片》中的便签条，持久化存储核心记忆。

  上下文窗口压缩（Context Window Compaction）：当上下文接近满时（例如达到 190k tokens），自动调用一个工具让 Claude 将当前的对话历史和操作记录总结成一段简短精炼的摘要。然后，开启一个全新的会话，将这段摘要和从长期记忆文件中读取的核心信息一起作为初始上下文，从而实现无限运行。

> "深入学习建议：
>
> 这个极其重要的实战技巧详细源于 **《Lessons on AI agents from Claude Plays Pokemon》** 视频，是解决长任务 Agent 问题的关键。

---

## ✔**第三部分：智能体架构模式选择**

根据任务的复杂性，您可以选择或组合以下几种经过验证的架构模式：

1. **提示链（Prompt Chaining）**：线性执行，适用于可清晰分解的固定子任务。
2. **路由（Routing）**：根据输入分类，导向不同处理路径。适用于需要对不同类型请求进行专门处理的场景（如客服）。
3. **并行化（Parallelization）**：将任务分解为多个独立子任务并行处理，或对同一任务运行多次投票。
4. **编排者-工作者（Orchestrator-Workers）**：一个中心的“编排者”LLM 动态地分解任务，并将其分配给多个“工作者”LLM 并行处理。适用于无法预知子任务的复杂场景（如编码、研究）。
5. **评估者-优化者（Evaluator-Optimizer）**：一个 LLM 生成响应，另一个 LLM 在循环中对其进行评估和提供反馈，进行迭代优化。
6. **自主智能体（Autonomous Agents）**：最高级的模式，LLM 在一个循环中自主地使用工具、接收环境反馈并决定下一步行动。

> "深入阅读建议：
>
> 每种模式的详细图解和示例，请参考 **《Building Effective AI Agents》**。

---

## ✔**第四部分：高级主题与未来展望**

#### **1. 构建多智能体系统**

* **为何需要**：通过并行处理和分离关注点，解决单个 Agent 无法完成的超复杂任务。

  **架构模式**：通常采用**编排者-工作者**模式。

  **未来探索**：多智能体间的交互会产生有趣的涌现行为，例如让多个 Claude 实例扮演不同角色玩“狼人杀”游戏，可以帮助我们更好地理解模型的行为。

#### **2. Agent 应用的未来趋势**

* **被低估的领域**：为企业自动化重复性任务，即使每次只节省一分钟，但当你可以将这个任务的执行频率提高100倍时，其价值是巨大的。

  **被高估的领域（目前）**：面向消费者的 Agent（如全自动预订假期）。因为完全指定个人偏好和验证结果的成本，几乎和自己动手一样高。

> "深入学习建议：
>
> 关于多智能体系统的架构和提示工程，请阅读 **《How we built our multi-agent research system》**。关于未来趋势的讨论，可观看 **《Tips for building AI agents》**。

---

## ✔**第五部分：Agent 的提示工程**

为 Agent 编写提示与传统的提示工程有很大不同。

1. **给予合理的启发式原则（Heuristics），而非僵化的规则：**

* 不要试图预测所有情况。相反，应教会 Agent 一些通用原则。例如，为 Claude Code 灌输“不可逆性”的概念，让它避免执行 `rm -rf` 等可能对用户环境造成永久损害的操作。

  为 Agent 设定“预算”，例如告诉它简单的查询应使用少于 5 次工具调用，复杂的则可以用到 15 次，避免其无休止地搜索。

2. **明确指导工具选择和思考过程：**

* 不要假设 Agent 知道在您的特定业务场景下应该优先使用哪个工具。在提示中明确指出：“如果需要查询公司内部信息，优先搜索 Slack”。

  引导 Agent 如何使用“思考”过程。例如，在研究任务开始前，提示它在第一个思考块中规划好“查询有多复杂？需要多少次工具调用？如何判断成功？”等问题。

3. **警惕意外的副作用：**

* 您给出的指令可能会被 Agent “过度执行”。例如，当您告诉 Agent “直到找到最高质量的来源再停止”，它可能会因为找不到“完美”来源而陷入无限搜索。因此，需要设置停止条件。

4. 关于“少样本示例（Few-shot Examples）”：

* 对于先进的模型和 Agent 场景，提供大量具体的、循规蹈矩的示例反而会限制模型的智能和创造力。更有效的方法是给出高层次的原则和策略，让模型自主发挥。

> "深入学习建议：
>
> 这部分内容是 **《Prompting for Agents》** 视频的核心，对于编写高质量的 Agent 系统提示至关重要。

---

## ✔**第六部分：评估：贯穿始终的关键环节**

**衡量你的结果**是构建 Agent 过程中最重要、也最容易被忽视的一环。

* **从小样本开始，尽早评估**：不要等到搭建了包含数百个案例的庞大评估框架才开始。在开发初期，即使是10-20个手动测试用例，也能揭示出巨大的问题，因为初期的改进效果通常非常显著。

  **使用真实世界的任务**：评估用例应反映 Agent 在实际应用中会遇到的问题，而不是简单的“沙盒”或编程竞赛题。

  **善用 LLM-as-judge**：对于难以程序化评估的输出，可以使用 LLM 作为裁判，根据预设的评分标准（Rubric）来打分。

  **关注最终状态评估（Final-state Evaluation）**：对于会改变环境状态的 Agent（如客服 Agent 修改数据库），评估重点应是任务完成后的“最终状态”是否正确，而不是过程中的每一步。

> "深入学习建议：
>
> 评估的重要性在 **《Advice For Building AI Agents》** 和 **《Prompting for Agents》** 视频中被反复强调。

---

## ✔**第七部分：来自一线的终极课程：Claude 玩宝可梦**

这个项目不仅有趣，更是一个适合用于了解 Agent 本质的绝佳案例；详细案例可在《Lessons on AI agents from Claude Plays Pokemon》中了解。

1. \*\*模型的进步在于“策略”，而非“感官”：**Claude 玩宝可梦的能力从 3.5 到 3.7 的巨大飞跃，主要不是因为它的视觉能力（看清游戏画面）变强了，而是因为它**制定策略、从失败中学习、质疑旧策略并尝试新方法（坚韧性）\*\*的能力得到了质的提升。这正是高级 Agent 的核心。
2. 失败揭示了**模型的根本局限与工程解决方案**：

* **缺乏时间感：** Claude 会连续 8 小时按一个按钮试图关闭一个它误认为是对话框的“门垫”。

  + 解决方案：在信息中加入“步数计数器”，赋予它一个时间的代理指标，并提示它“如果长时间做某件事没有进展，应该重新考虑策略”。

  **缺乏自我认知：** 它不知道自己“看不清屏幕”，因此会一直撞墙。

  + 解决方案：通过提示和提供额外信息，帮助它建立对自身局限的“元认知”。

  **灾难性遗忘：** 在花费三天走出迷宫，离出口仅 15 步时，它迷路了，然后使用“逃生绳”传送回了洞穴入口，前功尽弃。

  + 解决方案：上文提到的“长短期记忆”机制。

3. 从兴趣出发，建立直觉：

* **构建 Agent 最好的方式是从一个您热爱的、有趣的项目开始实践。** 通过长时间、高频次的互动，您会对模型的优点、缺点和“脾气”建立起深刻的直觉。这种通过实践获得的经验，比任何一篇技术文档都更有价值。

*通过这份融合了理论与实践的完整路线图，希望你可以系统地构建、优化和扩展您的 AI Agent，真正释放 Claude 的潜力。*

当然，也需要注意以上均为来自Anthropic一家的观点建议，实际还有多家的经验也非常值得参考学习，强烈建议结合多方信息和自身的实践进行消化理解；后面有机会我也会继续整理和分享。

---

## ✔原文参考：

#### **博客文章 (Blog Posts)**

* \*\*Contextual Retrieval in AI Systems\*\*[1]

  \*\*Building Effective AI Agents\*\*[2]

  \*\*How we built our multi-agent research system\*\*[3]

  \*\*Writing effective tools for AI agents—using AI agents\*\*[4]

  \*\*The think tool: Enabling Claude to stop and think\*\*[5]

#### **视频内容 (Videos)**

* \*\*Lessons on AI agents from Claude Plays Pokemon\*\*[6]

  \*\*Advice For Building AI Agents\*\*[7]

  \*\*Building Blocks for Tomorrow's AI Agents\*\*[8]

  \*\*Prompting for Agents | Code w/ Claude\*\*[9]

  \*\*Tips for building AI agents\*\*[10]

### 关于我

> "Hi，我是Roxane，一个AI重度用户，前RPA产品经理，企业数智能化顾问，目前正在AItoB领域进行探索。*这里是我的知识仓库，我会不定期更新我的学习笔记、AI产品/案例/公司研究、实用工具分享等内容。*如果想与我聊聊，欢迎后台留言，也可以通过以下方式联系我：

* wx：`Roxaneisland`

  mail：`Alondite04@gmail.com`

  即刻：`@Roxane`

### 参考资料

[1] 

**Contextual Retrieval in AI Systems**: *https://www.anthropic.com/engineering/contextual-retrieval*

[2] 

**Building Effective AI Agents**: *https://www.anthropic.com/engineering/building-effective-agents*

[3] 

**How we built our multi-agent research system**: *https://www.anthropic.com/engineering/multi-agent-research-system*

[4] 

**Writing effective tools for AI agents—using AI agents**: *https://www.anthropic.com/engineering/writing-tools-for-agents*

[5] 

**The think tool: Enabling Claude to stop and think**: *https://www.anthropic.com/engineering/claude-think-tool*

[6] 

**Lessons on AI agents from Claude Plays Pokemon**: *https://www.youtube.com/watch?v=CXhYDOvgpuU*

[7] 

**Advice For Building AI Agents**: *https://www.youtube.com/watch?v=Ta5kUdng7Zc*

[8] 

**Building Blocks for Tomorrow's AI Agents**: *https://www.youtube.com/watch?v=oDks2gVHu4k*

[9] 

**Prompting for Agents | Code w/ Claude**: *https://www.youtube.com/watch?v=XSZP9GhhuAc*

[10] 

**Tips for building AI agents**: *https://www.youtube.com/watch?v=LP5OCa20Zpg*