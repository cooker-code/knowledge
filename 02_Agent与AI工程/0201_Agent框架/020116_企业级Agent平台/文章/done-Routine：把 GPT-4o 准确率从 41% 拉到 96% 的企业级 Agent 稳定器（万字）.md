> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020116_企业级Agent平台/020116_核心知识点/意图识别与Routine约束稳定器|意图识别与Routine约束稳定器]]
---
title: Routine：把 GPT-4o 准确率从 41% 拉到 96% 的企业级 Agent 稳定器（万字）
author: 觉察流
date:
url: https://mp.weixin.qq.com/s/yIjiafE0aYPLfyl0lz6I5A
---

*点击👇🏻可关注，文章来自*

🙋‍♂️ 想加入社群的朋友，可看文末方法，进群交流。

---

**“** Routine 框架用 400 行 JSON 把 GPT-4o 在企业场景的端到端准确率从 41% 拉到 96%，14B 小模型也能逼近 SOTA——本文章拆解它到底做对了什么。**”**

大家好，我是肆〇柒。在企业落地 AI 的进程中，自主智能体凭借其强大的自主决策与任务执行能力，可以成为企业提升效率、优化流程的关键力量。然而，当我们将视角聚焦于企业级应用场景时，不难发现，部署智能体系统并非易事。在企业级 LLM 智能体（Agent）系统的落地过程中，开发者往往被两个问题反复折磨：

1. 1. 工具调用链路的正确率不可控；
2. 2. 一旦工具规模超过 20 个，准确率便雪崩式下滑。

比如，以企业财务报表智能分析为例，智能体需精准调度数据采集、预处理、指标计算等工具。然而，实际运行中，规划模型常因缺乏对财务系统深度耦合的认知，生成的执行方案漏洞百出：关键的财务指标计算工具未被调用，数据预处理步骤顺序颠倒，最终导致整个分析流程陷入混乱。这样的案例揭示出企业在利用智能体解决复杂任务时遭遇的瓶颈——智能体系统难以在企业特定场景中保持执行的稳定性和准确性。

这引发了一个需要解答的关键问题：如何为智能体系统注入强大的“导航仪”，使其在企业复杂的数字化场景中稳定、精准地执行任务？

本文聚焦的 Routine 框架（Digital China AI Research）正是为了解决上述痛点而生。它用“结构化规划”取代“自由生成”，用“可验证步骤”取代“模糊指令”，从而在真实企业场景中把 GPT-4o 的端到端正确率从 41% 提升到 96%，让 14B 参数的本地模型逼近 GPT-4o 水平。下面将围绕 Routine 的机制设计、系统架构、数据工程与训练策略展开，所有结论均基于企业级实验验证。

# Routine 机制设计：从痛点到解决方案

为了更好地理解Routine框架如何解决痛点，下图展示了其引导LLM智能体完成多步骤工具调用的机制。通过这种结构化的方式，Routine为智能体的执行过程提供了清晰的指导。

Routine框架引导LLM智能体完成多步骤工具调用的机制示意图

### 显式工具字段

早期痛点：工具选择错误率高，模型在无明确指引时难以从海量工具中精准挑选适配工具。当工具池规模达到 25 个及以上时，GPT-4o 的工具选择错误率高达 58.9%。在企业级场景中，智能体需要从海量工具中精准挑选适配工具，但现有模型在无明确指引时难以胜任这一任务。

Routine 约束：在每一步骤中显式标注工具名称（Step Tool）。

实现方式：定义了 “Step Tool” 字段，确保智能体在执行时直接读取指定工具，无需推理工具选择。这为执行模块提供了精准的工具调用蓝图，减少了因规划模糊导致的工具误调用等问题。

### 输入输出参数描述

早期痛点：参数幻觉问题频发，模型常因缺乏参数上下文而生成错误调用指令。

Routine 约束：为每一步骤提供详细的输入描述（Input Description）和输出描述（Output Description）。

实现方式：通过自然语言明确参数来源和目标，帮助智能体准确填充工具参数。这为模型提供了丰富的上下文信息，尤其对中小规模模型助力显著，使其参数准确率大幅提升。

### 变量内存优化

早期痛点：企业任务流程复杂且环环相扣，对准确性和稳定性要求极高。以金融领域的风险评估为例，智能体需从多个异构数据库中提取数据，运用复杂模型运算，最终生成评估报告。任何一个微小偏差都可能引发巨大风险。现有智能体架构在处理此类任务时，常因上下文管理效率低下而陷入混乱。所以，上下文爆炸问题导致模型执行效率低下，尤其在多步骤工具调用中表现明显。

Routine 约束：引入 Variable Memory 机制，采用 key-value 映射优化参数传递。

实现方式：智能体只需传递变量键而非完整值，显著降低上下文压力。当工具返回 10KB 的 JSON 时，执行模块仅存储键 memory\_xxx，后续步骤通过键引用实际值，使上下文长度降低 90%。

### AI 驱动的 Routine 优化

AI驱动的Routine优化是提升智能体执行效率的关键环节。下图详细展示了AI如何优化和管理Routine的流程，通过这种方式， Routine能够更好地适应企业不断变化的需求。

AI优化和管理Routine的流程图

早期痛点：现有智能体系统对领域专家的依赖程度较高。当前系统对专家草稿的耦合度≈100%。当企业引入新型工具（如区块链溯源工具）时，规划模型无法自主识别其在流程中的精准定位，需依赖专家重新起草规划草稿。领域专家稀缺，智能体系统难以快速复用专家知识。

Routine 约束：通过 AI 优化专家草稿，生成结构化 Routine。

实现方式：AI 将专家草稿转化为包含工具映射的完整 Routine，减少专家重复劳动。例如，专家提供 “查询库存 → 比对订单 → 生成补货单” 的草稿，AI 通过附录 A.1 的 Prompt 模板将其转化为包含工具名的 5 步 Routine，其中 “比对订单” 被拆分为两个子步骤以适配工具接口限制。

### 分支逻辑处理

早期痛点：复杂流程中的分支逻辑导致执行稳定性下降，模型在面对条件判断时容易出错。

Routine 约束：支持分支结构（Branch），通过条件判断引导不同执行路径。

实现方式：定义了分支步骤格式（Branch X-n Step i），为智能体提供清晰的条件判断逻辑。若 Routine 第 3 步为分支判断（如 “库存 < 阈值？”），执行模块需调用库存查询工具，解析返回结果中的数值，根据条件跳转到 “生成补货单” 或 “通知采购” 分支。

# Routine 框架与系统架构：从规划到执行的闭环设计

下图呈现了Routine框架在智能体系统中的整体架构，展示了从规划到执行的各个模块如何协同工作。这一架构设计确保了智能体能够在企业复杂场景中高效、稳定地执行任务。

Routine框架与系统架构图

### 一条 Routine 的生命周期：从文本到 JSON

#### 组成要素

Routine 的组成部分。注：标有 \* 的组件为可选元素

Routine 的设计比较严谨，其组成部分环环相扣，共同构建起智能体执行的坚实基础。步骤编号为智能体指明执行顺序；步骤名称以简洁语言概括步骤核心功能；步骤描述则详细阐述执行条件、目标及操作细节，为智能体提供详尽的操作指南。

输入描述清晰界定步骤所需参数，确保智能体在执行前就备齐“弹药”。输出描述提前规划步骤完成后产生的结果，为后续步骤衔接提供保障。步骤工具精准锚定每一步骤对应的工具，实现智能体能力与任务需求的完美匹配。

以企业财务报表智能分析场景为例，一个典型的 Routine 可能包含以下步骤：

* • 步骤 1 为“数据采集”，利用“数据库查询工具”从财务系统中提取特定时间段的收入、支出等数据；
* • 步骤 2 为“数据预处理”，调用“数据清洗工具”对采集到的数据进行缺失值填充、异常值处理等操作；
* • 步骤 3 为“财务指标计算”，运用“财务分析模型工具”计算毛利率、净利率等关键指标。

对于包含分支结构的 Routine，其格式如下：

```
Step X. <Step Name>: This step performs a branch condition check:
• Branch X-1 Step 1. <Step Name>: If <Condition>, perform <Step Description>, using the <Tool Name> tool;
• Branch X-1 Step 2. <Step Name>: <Step Description>, using the <Tool Name> tool;
• Branch X-2 Step 1. <Step Name>: If <Condition>, perform <Step Description>, using the <Tool Name> tool; Step Y. <Step Name>: <Step Description>, using the <Tool Name> tool; Step Z. <Step Name>: <Step Description>, using the <Tool Name> tool, and terminate the workflow;
```

#### 双格式表示

Routine 支持自然语言格式与结构化 JSON 格式并存，二者相得益彰，满足不同场景需求。自然语言 Routine 以流畅语言描述步骤流程，便于人类专家快速理解与审核，相当于为智能体执行流程撰写了一份通俗易懂的“说明书”。

结构化 JSON 格式则以严谨的数据结构呈现 Routine，为智能体系统解析与执行提供便利。它将每一步骤的编号、名称、描述、工具等信息封装为标准字段，确保智能体能够精准无误地读取与解析。

JSON 格式示例如下：

```
[
  {
    "step":"1",
    "name":"Get announcements",
    "description":"Download the latest employee handbook file from the company’s internal system",
    "tool":"fetch_latest_announcements",
    "type":"node"
},
{
    "step":"2",
    "name":"Download handbook",
    "description":"Obtain the company’s most recent official announcement for consistency check with the employee handbook",
    "tool":"download_file",
    "type":"node"
},
{
    "step":"3",
    "name":"Read PDF content",
    "description":"Use text parsing tools to extract all text content from the employee handbook PDF file",
    "tool":"read_pdf",
    "type":"node"
},
{
    "step":"4",
    "name":"Compare text differences",
    "description":"Compare the relevant content in the employee handbook with the company’s latest announcement word by word",
    "tool":"compare_texts",
    "type":"finish"
}
]
```

特别地，`type: finish` 字段用于标记流程终止步骤，确保智能体在执行到该步骤时能够正确结束工作流程。

#### AI 优化生成

Routine 的生成从专家编写的规划提示开始。专家根据对企业任务流程的深刻理解，以自然语言草稿形式勾勒出任务执行的大致轮廓，包括关键步骤、目标等核心要素。

AI 驱动的优化过程首先把规划拆解为更小的可操作任务。首先对规划进行深度分解，将模糊的步骤细化为具体可操作的子步骤。以“处理客户投诉”任务为例，专家草稿仅提及“分析投诉原因”，AI 则将其细化为“提取投诉关键词”“识别投诉类别”“定位问题根源”等子步骤。

接下来，模型将这些子步骤精准映射到可用工具上。在映射过程中，模型充分考量工具的功能说明、参数要求等细节，确保工具选用契合度。最终，AI 输出结构化的 Routine，其语言流畅、逻辑严谨，为执行模块提供清晰指引。

为助力 Routine 生成，研究者设计了如下 Prompt 模板：

```
prompt = f"""You are a Routine workflow writer for a company. You can write the operation step flow based on the process information provided by the user and the available tools.
The steps are written in structured json and lists. Write the flow in the following way:
[{{"step": "1", "name": "xxxxx", "description": "xxxxxxxxxxxx", "tool": "tool_X", "type": "node"}},
{{"step": "2", "name": "xxxxx", "description": "xxxxxxxxxxxx", "tool": "tool_Y", "type": "node"}}]

The format is a json list. Each step contains the step number, step name, step action description, step input, step output, step tool, and node type.
The input and output of the step do not have to be very specific. Use natural language to write the possible input and output according to the tool. Only one tool is used for each step.
When you may encounter branch condition judgment in a certain step, express it in the following way and indicate under what conditions to enter a branch, what tool to use;

{{"step": "x", "name": "xxxxx", "type": "branch"}},
{{"step": "x-1_1", "name": "xx", "description": "xxxx", "tool": "tool_X1", "type": "branchnode"}},
{{"step": "x-2_1", "name": "xx", "description": "xxxx", "tool": "tool_X2", "type": "branchnode"}},
{{"step": "y", "name": "xxxxx", "description": "xxxxxx", "tool": "tool_Y", "type": "node"}}
If the next branch step involves multiple steps, you can open a new branch workflow, for example:
{{"step": "x-n_1", "name": "xx", "description": "xxxx", "tool": "tool_X", "type": "branchnode"}},
{{"step": "x-n_2", "name": "xx", "description": "xxxx", "tool": "tool_Y", "type": "branchnode"}}

Regarding the writing of step numbers, x-n_i represents the i-th step in the n-th branch of the main line step x;
Please pay attention to the description in the tool and the parameters that need to be filled in, which need to be fed back in the input of each step;
Pay attention to the branch judgment in the process information, and do not write multiple possibilities of branch conditions in the steps of the same line;
When a step is completed and the workflow needs to be ended, please change the node type of the step to "finish", set "type": "finish"; For example:
{{"step": "x", "name": "xxxxx", "description": "xxx", "tool": "tool_X", "type": "finish"}}
Note: Each workflow step must use a tool provided in the tool list, or perform branch condition judgment. There will be no "no tool needed", "no tool used ", or use of non-existent tools. Each step only uses one tool.
The following is the process information provided by this user: {routine_draft};
In the tool list, these tools are available: {tool_list}; Now please convert it into a structured Routine workflow. Do not output other prefixes, suffixes, or meaningless information, and please output in Chinese."""
```

# 数据工程：让 Routine 可训练、可蒸馏

#### 通用 Routine 数据合成

研究团队利用 BUTTON 开源数据集合成通用 Routine 数据，以增强模型在不同场景下的鲁棒性。BUTTON 数据集包含 8,000 个单轮多步工具调用实例，涵盖多种常见任务类型。每个实例交替包含 tool\_call 和 observation，形成清晰的执行轨迹。

通过 GPT-4o 和专用 Prompt 模板，研究团队生成精准结构化的 Routine，每个 Routine 包括步骤索引、名称、功能目标和对应工具名称。基于标准化执行 Prompt 模板构建新系统 Prompt，提升模型对多种工作流机制的适应性。

数据过滤规则：

1. 1. Routine 文本验证：过滤掉空响应或异常输出；
2. 2. 移除自然语言摘要：保留用户查询、工具调用和对应观察；
3. 3. 长度和结构过滤：剔除超过 8 个工具调用步骤的实例，移除含复杂数据结构的实例。

最终保留 4,209 条高质量轻量级训练数据。

以下是通用 Routine 跟随数据集过滤流程图，展示了如何通过一系列步骤筛选出高质量的训练数据：

常规 Routine-following 数据集过滤流程

#### 场景特定数据的 GPT-4o 蒸馏

在特定场景中，Routine 还可用于知识蒸馏。以 HR 智能体场景为例，研究团队设计 50-60 个用户查询模板，基于 10 个非分支子场景生成约 50-60 个用户查询。通过数据清洗和 LLM 同义改写，增强数据多样性。

使用 GPT-4o 和 Routine 蒸馏生成 537 条单轮多步用户查询，包含 3,108 个标注工具调用指令。这些数据用于训练场景特定的执行模型，显著提升模型在特定场景下的准确率。

# 系统模块：执行、工具与记忆

#### 执行模块与小型 LLM 设计

执行模块负责接收规划模块生成的 Routine，并按其路径输出工具调用指令。考虑到执行过程中的大量上下文，执行模块通常由小型专用指令跟随模型驱动。

选择小型模型的优势：

1. 1. 节省资源：小型模型在执行每一步骤时消耗较少算力和时间，易于在企业环境中部署；
2. 2. 高效指令跟随：只需关注指令跟随和多步工具调用，无需复杂逻辑推理。

Qwen3 系列小型 Instruct 模型在指令跟随和中文理解方面表现出色，且参数量小，是执行模块的理想选择。

#### 工具模块与 MCP 协议

工具模块负责接收工具调用指令并返回执行结果。MCP 服务器作为工具模块的核心，定义和管理智能体可用的工具集合。

MCP 协议的优势：

1. 1. 工具标准化：统一描述工具名称、参数类型和调用约束，使执行模块无需管理工具实现细节；
2. 2. 高可扩展性：便于开发者添加新功能或连接新系统，构建多样化工具生态。

#### 记忆模块

记忆模块负责存储和检索任务执行过程中的关键信息。它包含两种内存形式：长期 Procedure Memory 和短期 Variable Memory。

Procedure Memory 存储专家与规划模块协作生成的 Routine 集合。在部署前，专家将必要的 Routine 集合存入内存基座。系统接收查询时，基于 Routine 描述与用户任务的相似性计算，检索对应 Routine，协助执行模块。

Variable Memory 优化多步工具调用间的参数传递。当工具调用返回过长参数时，系统自动存储至变量内存基座。模型只需提供对应键，而非完整值。接收工具调用请求时，记忆模块将键回溯为实际参数值。Variable Memory 通过键值映射减少 90% 上下文长度，显著降低上下文压力，减少 token 消耗和语法错误，提升执行模块性能。

以下是智能体变量内存机制示意图，展示了 Variable Memory 如何优化多步工具调用中的参数传递：

智能体可变记忆机制示意图

# 训练策略：场景化 LoRA 微调

#### 基模选择与训练目标

在 HR 智能体场景中，执行模型需要具备指令跟随、初步工具调用能力和较强中文理解能力。选择 Qwen3 系列小型 Instruct 模型作为基模。训练目标是提升模型对 Routine 的遵循能力，而非利用推理生成工具调用指令。

#### LoRA 超参数与硬件配置

采用 LoRA 轻量级微调策略，避免过拟合，提升计算性价比。实验使用 LLaMA-Factory 框架，结合 DeepSpeed ZeRO-3 和 Flash Attention-2，最大化计算效率，显著降低多 GPU 训练时的 GPU 内存使用。

训练参数：

* • LoRA 秩：8
* • 批大小：1×4 GPU
* • 有效批大小：16
* • 学习率：1e-4
* • 训练周期：3 epoch

以下是智能体执行模型训练过程图，展示了整个训练流程，包括通用 Routine 跟随能力训练和场景特定工具调用能力训练：

智能体执行 routine（模型训练流程）

# 实验验证与结果分析

### 工具调用准确率提升

在未引入 Routine 时，模型表现不佳。实验数据显示，工具选择错误占比高达 85%。在企业智能法务审核场景中，模型需从 30 余种工具中精准挑选，无 Routine 指导时，工具选择错误率高达 78%。

引入 Routine 后，模型执行流程变得有序。在 HR 智能体测试集（1,148 个样本）中，GPT-4o 的端到端准确率从 41.1% 提升至 96.3%。Qwen3-14B 在有分支 Routine 下的准确率为 81.3%，无分支 Routine 下的准确率为 83.6%。GPT-4o 基于 Routine 蒸馏生成的 537 条场景特定数据，使 Qwen3-14B 的准确率达到 95.5%。

具体到各项指标：

* • 结构正确率从 68% 提升至 97%；
* • 工具选择准确率从 35% 提升至 88%；
* • 参数准确率从 52% 提升至 81%。

以下是企业场景中 AST 评估工作流程图，展示了如何基于 BFCL 框架对模型的工具调用进行评估：

企业场景下的 AST evaluation workflow（沿用 routine 框架）

以下是不同 Routine 配置下 HR 智能体系统场景中各模型的整体准确率表：

在 HR 智能体系统场景下，不同 Routine 配置下模型的整体准确率

### 训练策略的影响

#### 常规 Routine 跟随微调

基于通用 Routine 跟随数据集的微调可显著提升模型在有 Routine 指导场景下的执行准确率。以 Qwen3-14B 为例，微调后整体准确率提升 14.6 个百分点。但此类模型在无 Routine 指导场景下的表现有所下降，表明其更擅长执行明确规划的任务，而非自主规划。

#### 场景特定数据蒸馏微调

基于场景特定数据蒸馏的微调可显著提升模型在无 Routine 指导场景下的表现。以 Qwen3-14B 为例，微调后整体准确率提升 22.9 个百分点，甚至超越原始模型在有 Routine 指导下的表现。这表明场景特定数据蒸馏可将 procedural knowledge 内化于模型权重中，减少模型对显式规划的依赖。

### 消融研究

#### Routine 组件的影响

以下是不同 Routine 组件对模型整体准确率的影响表：

整体模型精度在不同 routine 组件上的表现

实验表明，工具规格说明是提升准确率的核心要素。当 Routine 中包含工具名称时，模型准确率较不含工具名称时平均提升 18.3 个百分点。输入输出参数描述为模型提供了丰富的上下文信息，尤其对中小规模模型助力显著，使其参数准确率提升 12.7 个百分点。

#### AI 优化机制的价值

以下是不同 Routine 生成方法下模型整体准确率表：

Routine 在不同生成方法下的整体模型准确率

AI 优化可显著提升 Routine 质量。以 Qwen3-14B 为例，AI 优化后的 Routine 使模型准确率达到 76.7%，接近手动标注 Routine 的效果（83.3%）。这表明 AI 优化是企业场景下高效生成高质量 Routine 的可行路径。

#### Routine 数量的影响

以下是智能体规划的不同方式图，展示了从用户草稿到 AI 优化再到手动标注的 Routine 生成过程：

智能体规划的多种实现路径

实验显示，提供单个正确 Routine 时模型表现最佳。当引入干扰 Routine 时，高性能模型的准确率显著下降，但随着 Routine 数量增加，准确率逐渐恢复。这表明模型可能从“组合步骤”转向“选择机制”，识别并执行最相关的 Routine。

以下是不同数量 Routine 下模型整体准确率表：

整体模型在多个 Routine 上的综合准确率

# Routine 的局限性

当前规划模型在生成 Routine 流程时，对企业领域专家草稿的依赖程度较高。这在一定程度上限制了智能体系统面对新工具引入或工作流变更时的泛化能力。当企业引入新型区块链溯源工具，试图将其融入现有供应链智能管理 Routine 时，规划模型往往无法自主识别该工具在流程中的精准定位，需依赖专家重新起草规划草稿。

执行模型主要通过指令微调和知识蒸馏进行适应，这种方式在面对企业流程的细微变更时，如订单处理流程中新增的环保合规审查步骤，模型难以迅速调整自身以匹配新的 Routine 要求。这表明当前智能体系统在动态适应性方面仍有较大提升空间。

针对上述挑战，研究者提出将基于强化学习（RL）的智能体框架引入工作流的改进方向。强化学习通过构建奖励模型，让智能体在与环境交互中自主学习优化 Routine 生成策略。例如，在智能体尝试生成包含新引入区块链工具的 Routine 时，奖励模型可根据流程执行效率、结果准确性等指标给予即时反馈。若智能体尝试将区块链工具错误放置在“订单创建”步骤，导致执行效率低下，奖励模型会给予负向反馈，引导智能体调整工具位置至“产品发货”步骤。

多智能体框架的探索同样前景广阔。在该框架下，高级智能体犹如调度员，依据结构化的 Routine 流和集中交互协议，协调多个专业智能体。在企业智能工厂场景中，高级智能体根据生产调度 Routine，指挥生产执行智能体、质量检测智能体、物流配送智能体协同作业。这种层次化交互方案有效降低了单个智能体承担的 Routine 计划复杂度，让每个智能体专注于自身擅长的领域任务，大幅提升企业工作流执行的稳定性与智能性。

# 总结：从规划到落地的关键跃迁

Routine 作为一款结构精巧、内容全面的规划框架，为企业智能体系统的多步骤工具执行提供了精准导航。

### Routine 的四重价值主张

1. 1. 结构化导航：它通过卓越的流程编排手段，将复杂的企业任务分解为清晰有序的步骤序列，确保智能体在执行过程中每一步都踩在关键节点上。
2. 2. 数据蒸馏引擎：在训练数据合成领域，Routine 展现出强大的蒸馏能力。通过将专家规划草稿转化为结构化的 Routine 数据集，为模型训练提供了高质量的营养源泉。
3. 3. 小模型杠杆：用 14B 参数逼近 GPT-4o 的 96% 准确率。在企业智能运维场景中，基于 Routine 合成的数据集让模型精准学习到服务器巡检、故障排查等复杂任务的执行逻辑。更值得一提的是，Routine 在特定领域多步骤工具调用数据集生成方面的作用无可替代。通过对企业业务流程的深度建模，Routine 生成的数据集完美契合企业实际需求。
4. 4. 场景即插即用：在智能体系统开发中，开发者依据 Routine 数据集训练出的模型，能够深度适配企业的财务审计、客户关系管理等核心业务流程。

### 实验结果的总结

在 HR 智能体测试集（1,148 个样本）中，GPT-4o 的端到端准确率从 41.1%（无 Routine）提升至 96.3%（Routine 指导）。

该测试包含 3 个含分支的 Routine，分支逻辑使 Qwen3-14B 准确率从 83.6%（无分支）降至 81.3%（有分支）。

GPT-4o 基于 Routine 蒸馏生成的 537 条场景特定数据，包含 3,108 个标注工具调用指令，用于训练 Qwen3-14B，使其准确率达 95.5%。LoRA 微调参数为：秩 8，批大小 1×4 GPU，有效批大小 16，学习率 1e-4，训练 3 epoch。

### 下一步：从 “流程执行” 到 “流程进化”

**动态适应**：RLHF 驱动的 Routine 自进化机制
**多智能体编排**：用「主-专」架构降低单 Routine 复杂度
**零样本迁移**：新工具接入时的自主工作流重构能力

### AI for Process

Routine 机制，让智能体系统深度融入企业流程，成为企业智能自动化与决策支持的中流砥柱。在企业智能决策场景中，智能体依据 Routine 调用数据分析工具、构建预测模型，为企业管理层提供精准决策依据。从生产排程到市场营销策略制定，Routine 驱动的智能体系统全方位提升企业运营效率与质量。

各位，看过此文有什么感想？如有其他想法可以在评论区留言，我们聊聊。或者加入“觉察流”社区群，与群里的小伙伴一起学习、交流。加入方法，私信回复“入群”“加群”即可。

如果你关注 AI Agent 技术，可以点击订阅主题👉“[AI Agent](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk2NDA0MzcxNw==&action=getalbum&album_id=3820320836491411457&from_itemidx=1&from_msgid=2247487838&scene=173&subscene=&sessionid=1745800257&enterid=1745948015&count=3&nolastread=1)”。

参考资料

* • Routine: A Structural Planning Framework for LLM Agent System in Enterprise
  https://arxiv.org/pdf/2507.14447
* https://github.com/davidkimai/Context-Engineering

#觉察流 #AI全栈 #AI论文 #AI社区 #开源 #开源项目 #Routine #企业级LLM智能体 #多步骤工具调用 #执行稳定性 #模型性能优化  #结构化规划 #数据蒸馏 #LoRA微调 #MCP工具协议 #变量内存机制 #Agent #智能体 #LLM #大型语言模型 #AIAgent

往期回顾

◆[分享一个开源深度研究框架：DeepResearch Eco递归式工作流的设计与应用](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247490010&idx=1&sn=795c6d71fe2f7655a83f3aada9a383fe&scene=21#wechat_redirect)

◆[Agentic Enterprise：把 AI 从神坛拉回用户办公桌](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247490004&idx=1&sn=308212a49a58454d042c24be930abd57&scene=21#wechat_redirect)

◆[需求侧智能体及其商业模式设计](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489977&idx=1&sn=94d63478df750cc7b8849327b1c44743&scene=21#wechat_redirect)

◆[把科研写成 Python：X-Master 用代码拆碎“人类最后考试”](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489971&idx=1&sn=a0a0110a826ed989533ec4648d635152&scene=21#wechat_redirect)

◆🔥[从聊天记录到数字资产：MIRIX 让记忆可买卖](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489945&idx=1&sn=f136cd819fd7abcc09348a1c1d3a8814&scene=21#wechat_redirect)

◆[AGENTGROUPCHAT-V2：大型语言模型多智能体协作的创新思考（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489920&idx=1&sn=628444f81ea8a1150604a07c61022b89&scene=21#wechat_redirect)

◆[OpenAgentSafety 框架：AI 智能体安全评估的创新实践](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489899&idx=1&sn=6dec0b183969d53d4746e091b5d9a2a6&scene=21#wechat_redirect)

◆🔥[WebSailor 突破边界：助力开源智能体跨越复杂推理 “天花板”](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489833&idx=1&sn=580d8790da87a921b353c1c59fd52d6a&scene=21#wechat_redirect)

◆[AUTOMIND：自动化数据科学的创新框架（AI4Science）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489654&idx=1&sn=179249e8686125fb5a0f8a18e2c64216&scene=21#wechat_redirect)

◆[Mind2Web 2：智能体搜索系统的进化与评估之道](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489638&idx=1&sn=a7cae2a222a391130fcb7bcf88277e39&scene=21#wechat_redirect)

◆🔥[深度解析 MEM1：开启智能体长时序高效交互之门（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489598&idx=1&sn=dc25298f495576ea83344caecaaf4729&scene=21#wechat_redirect)

◆[MCP 安全之殇：智能体系统的隐忧与破局](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489572&idx=1&sn=e594f947bacc52a2e46bdf51794f569c&scene=21#wechat_redirect)

◆[STORYWRITER：长篇故事生成的多智能体框架](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489551&idx=1&sn=2e18514bb6cea32de494f2e69d91b9ff&scene=21#wechat_redirect)

◆🔥[你的 Cursor 用对了吗：SWE agent 智能协作之道（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489511&idx=1&sn=72a43f4b71a38e7ee8ef93785ec91bc1&scene=21#wechat_redirect)

◆🔥[MemOS：打破 LLM “记忆”孤岛，实现 Agent 协同智能（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489435&idx=1&sn=eee1ba59908ac037d03fec5bb519fd6b&scene=21#wechat_redirect)

◆🔥[掌控 AI 智能体自主性：五级框架下的人机协作之道（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489430&idx=1&sn=e730b22ea84c66e130f51ecb842014db&scene=21#wechat_redirect)

◆[AgentRM 奖励建模：智能体泛化能力的“导航仪”与“加速器”（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489359&idx=1&sn=1c12b4ab38f600ff72d4210148e1cbce&scene=21#wechat_redirect)

◆🔥[智能体协作的力量：Anthropic 的「Research」多智能体实践](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489333&idx=1&sn=f75e1927b0197805e19765297596eaa2&scene=21#wechat_redirect)

◆[Agentic Neural Networks（ANN）：自我演化的多智能体系统](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489288&idx=1&sn=3fe0c95136c4633ecce3ad0b1ec1f594&scene=21#wechat_redirect)

◆[AgentCPM-GUI：强化微调（RFT）赋能的移动设备 GUI 智能体（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489260&idx=2&sn=ff11ddc354ad68d9bec06a8ac3500f44&scene=21#wechat_redirect)

◆[ComfyUI-Copilot：AI 艺术创作的智能加速器，让创意触手可及](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489178&idx=2&sn=06cc0ca255ce85ee017ffc6eee27a842&scene=21#wechat_redirect)

◆🔥[Test-Time Scaling：挖掘大型语言模型推理潜能（3万字综述）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489178&idx=1&sn=95339e919429ac35cdccd1a695d375f7&scene=21#wechat_redirect)

◆[RL 驱动 LLM 智能体：ML-Agent 创新自主机器学习工程（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247489010&idx=1&sn=19df748b9545f388c45b353f4500cb69&scene=21#wechat_redirect)

◆[论智能体互联网的崛起：智能经济性驱动的价值转移与生态重构（二万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488996&idx=1&sn=9e195cd27c959fc1fa4445143583a6cc&scene=21#wechat_redirect)

◆[极简设计铸就卓越性能：Alita 通用智能体的进化思考](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488892&idx=1&sn=1a4a6f2b33ec3543415c2c7c1f0a132b&scene=21#wechat_redirect)

◆🔥[AutoRefine：RL加持RAG，边想边搜并精炼，革新LLM推理（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488880&idx=1&sn=22118f024c33eae24f678f82a1e80a6d&scene=21#wechat_redirect)

◆[SCIENCEBOARD：构建智能体驱动的科学探索新「环境」（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488774&idx=1&sn=3a42c7c7a52dc942d4c70f665ba521bc&scene=21#wechat_redirect)

◆🔥[Microsoft 推出 Magentic-UI：网页多智能体，革新式人机协作（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488593&idx=1&sn=3110682f00ec8baf6bd071c2394eb037&scene=21#wechat_redirect)

◆[AWS 开源 Strands Agents SDK：用几行代码唤醒 AI 智能体（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488422&idx=1&sn=cb1ada5fea6ff5cb3b2c2a5c4072f6c5&scene=21#wechat_redirect)

◆[Windsurf 发 SWE-1：以数据+智能飞轮驱动软件工程 AI 进化](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488399&idx=1&sn=16b127e218c3998cbedaf66c61daed2d&scene=21#wechat_redirect)

◆🔥[进化智能体 AlphaEvolve：科学发现与算法优化的新引擎（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488363&idx=1&sn=81153714eec46e6c9b2a354c7f7a3517&scene=21#wechat_redirect)

◆🔥[2025 生成式 AI 大棋局：全球数据报告里的趋势解读（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488338&idx=1&sn=88c226916596af6dbed1c7cb63b44069&scene=21#wechat_redirect)

◆[加入W3C GROUP，共筑 AI Agent 通信标准未来](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488182&idx=1&sn=032be10bc06c05c4bbbf4bd20bced632&scene=21#wechat_redirect)

◆[交互式生成视频（IGV）：重塑游戏、智能与驾驶的交互革命（二万字长文）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488071&idx=1&sn=a4eaa40c5e4169fdb0b2ae4a2765fb2d&scene=21#wechat_redirect)

◆[智能体协议：Coral Protocol 全面解析（万字长文）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488058&idx=1&sn=8af23d2285aca104f7156cf53d35f099&scene=21#wechat_redirect)

◆🔥[GitHub 十大开源 AI 项目盘点：从 MCP 到多智能体协作（万字）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488033&idx=1&sn=ee96164002a4a4af94eb0cfdeb3a946b&scene=21#wechat_redirect)

◆[WebThinker：赋能大型推理模型的深度研究“Deep ReSearch 神器”（1.5万字长文）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488026&idx=1&sn=d9367c80781775c6636bdbe0516f40fa&scene=21#wechat_redirect)

◆[LLM 推理新境界：多语言思考的力量](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247488002&idx=1&sn=60900dba14b8a595c4f3e6de7c8eb0ee&scene=21#wechat_redirect)

◆[AI 社会中的共识：语言理解能力如何塑造 AI 的群体决策？](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487944&idx=1&sn=588eefed55006ae3d932c1617a17c2b8&scene=21#wechat_redirect)

◆[Anthropic Claude 发布 Advanced Research：进入你的真实世界 使用私域数据进行智能协作](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487925&idx=1&sn=fde21e144ec69c0e289daf6c44fdbd06&scene=21#wechat_redirect)

◆[Computer Use 智能体框架：Agent S2 的创新与应用（万字长文）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487879&idx=1&sn=8eaee61ae2302627dae49dac28c00840&scene=21#wechat_redirect)

◆🔥[Windsurf 发布：“ What is an Agent？” （1.5万字长文）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487853&idx=1&sn=c7a0dbd18f9287a8759b81a6c0748c60&scene=21#wechat_redirect)

◆🔥[智能体「Agent」技术全景（6万字综述 合集）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk2NDA0MzcxNw==&action=getalbum&album_id=3961600786462900240&scene=21)

◆[智能体「Agent」技术全景：挑战、机遇与未来（6万字综述 下篇）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487838&idx=2&sn=21f98024d5bcca6bef2ae8e83bb30185&scene=21#wechat_redirect)

◆🔥[智能体「Agent」技术全景：挑战、机遇与未来（6万字综述 上篇）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487838&idx=1&sn=19914ee290d51f46ff07c760f0371bff&scene=21#wechat_redirect)

◆[FlowReasoner：自动化查询级 Multi-Agent 系统（万字长文）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487771&idx=1&sn=02eaf62b43017e25b1b6d4c4081cef96&scene=21#wechat_redirect)

◆[清华团队 LLM×MapReduce-V2：让学术综述一键 “飞” 起](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487692&idx=2&sn=9778c951a0ca344e72efeb5894f51911&scene=21#wechat_redirect)

◆🔥[AI Agent 协议：未来AI智能生态的基础设施（万字综述）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487639&idx=1&sn=aa144b44fc867e6c0c37513d617b18a2&scene=21#wechat_redirect)

◆[Agent A/B：用 AI Agent 颠覆传统网页 A/B 测试](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487533&idx=2&sn=c84e1763b9ccbce4032144b1e80db4ad&scene=21#wechat_redirect)

◆🔥[OpenAI发布：企业AI落地指南——应用场景识别与规模化应用策略](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487453&idx=1&sn=bc89b07d8d8efaa74c75a432e7a6fb1d&scene=21#wechat_redirect)

◆[OpenAI 发布：构建 AI Agent 实用指南](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487453&idx=2&sn=a5949727f355651f6ec849e23cae436b&scene=21#wechat_redirect)

◆🔥[OpenAI 发布企业 AI 集成技术手册：从评估到自动化](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487420&idx=1&sn=0c0d91562520ec392d38d47b2616196a&scene=21#wechat_redirect)

◆[FoA 框架：AI Agent “舰队” 征服复杂问题的利器](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487420&idx=2&sn=ecffac56537d401f68dd8f284723d2ff&scene=21#wechat_redirect)

◆🔥[守护 AI Agent：Progent 的细粒度权限控制策略](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487364&idx=1&sn=a9dafda501881ecdfef73b87d63bf39f&scene=21#wechat_redirect)

◆🔥[MCP 安全：守护 AI 系统的 “神经中枢”](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487350&idx=1&sn=c99d66914c1eb273078a3461f0b378cb&scene=21#wechat_redirect)

◆[AI 的下半场：从解决问题到定义问题](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487340&idx=1&sn=8e217f47094ce8dbca2c21e312d59932&scene=21#wechat_redirect)

◆🔥[降本增效！Gitee Code MCP 让 AI 代码评审从此无忧！](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487258&idx=2&sn=fdbe6f57c694ef8959a0598e9d55ff2d&scene=21#wechat_redirect)

◆🔥[Google Cloud Next 2025：AI 如何重塑云计算的未来（万字长文）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487178&idx=1&sn=19a6ab4731cd2d83bf28ac6a416b0102&scene=21#wechat_redirect)

◆[测试工程师的未来：AI Agent 能否成为好帮手？](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247487086&idx=2&sn=7cf779d48297ae500dd8b622d037001d&scene=21#wechat_redirect)

◆🔥[重磅！谷歌 A2A vs ANP：智能体通信的桥梁还是全新网络规则？](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486994&idx=1&sn=9ecf7d5bcec7d3b1f3b562cdff8328bd&scene=21#wechat_redirect)

◆🔥[MCP：AI 与工具交互的“瑞士军刀”](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486897&idx=1&sn=920d0e534885bef5499c6b68d763e892&scene=21#wechat_redirect)

◆[幻觉、攻击与伦理：GUI 智能体的可信性挑战（综述）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486897&idx=2&sn=a6f1c059bdb7081ac2ebc209553f79a7&scene=21#wechat_redirect)

◆[让AI学会自我认知：KnowSelf如何赋予智能体情境自知力](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486818&idx=2&sn=23bdc8c60bc768a89ba094313a797b95&scene=21#wechat_redirect)

◆[ReSearch 框架：让 AI 像人类一样边思考边搜索](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486765&idx=1&sn=ec4803dfe5951b0196d9f861ed852667&scene=21#wechat_redirect)

◆[LLM智能体：重新定义人机关系的 AI 科技（综述万字长文）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486717&idx=2&sn=eee250ce75f3e1ef775917ea1a8c2db9&scene=21#wechat_redirect)

◆🚀[智能体经济战略前瞻：颠覆与新生（二万字长文）](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486697&idx=1&sn=abd7a293e862cf00decf18d650e1e8cb&scene=21#wechat_redirect)

◆[SalesRLAgent：销售AI，从工具变战略伙伴](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486682&idx=1&sn=80e819efda7e9f4e710df0c1b1e405de&scene=21#wechat_redirect)

◆🔥[MCP协议的安全隐患：AI智能体的“隐形炸弹”](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486664&idx=1&sn=8adfb149851c736698c69b86abf81912&scene=21#wechat_redirect)

◆🔥[MemInsight：让AI的记忆像人类一样高效](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486664&idx=2&sn=d3338886f13bbe1f09a6d81ade46a87f&scene=21#wechat_redirect)

◆[MAPS：基于苏格拉底引导的Multi-Agent系统，解决多模态科学问题新思路](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486632&idx=1&sn=a368e966ae8a9fcf28f879477274f4ad&scene=21#wechat_redirect)

◆[SICOG：让多模态模型学会 “观察” 和 “思考”](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486632&idx=2&sn=bed931fa29520fc888e7285e42f57caa&scene=21#wechat_redirect)

◆[Loong：通过 Verifiers 实现大规模合成数据，解锁多领域推理能力](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486632&idx=3&sn=554f440beb7d3612a5e801575143e4e9&scene=21#wechat_redirect)

◆[构建下一代智能体：从 Prompt 到 End-to-End RL](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486589&idx=2&sn=261620db3d854744db428442537f71e2&scene=21#wechat_redirect)

◆[AGDebugger：Multi-Agent 系统的开发调试与引导利器](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486475&idx=2&sn=319274a866278286c6a5908ab197416f&scene=21#wechat_redirect)

◆[复杂任务不再难，ARMAP 助力 AI Agent 大显身手](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486431&idx=2&sn=7c50fe5e8f6f6d12bdaebb140c57618f&scene=21#wechat_redirect)

◆🔥[模型吞噬代码，Agent重构世界：当AI Agent与模型协同进化 (万字长文)](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486415&idx=1&sn=cfc4470a8dbba084d496e2064c8e13fb&scene=21#wechat_redirect)

◆[COWPILOT：人机协作网页导航的新思路](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486415&idx=2&sn=4cae1db291c9bc0641963502c6df2c0a&scene=21#wechat_redirect)

◆[ScoreFlow：让 AI Agent 协作更智能、更高效](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486398&idx=1&sn=2864f6d1ba9df899ec7f92a780d74cf3&scene=21#wechat_redirect)

◆🔥[PLAN-AND-ACT：提升AI Agent长期任务规划能力的新思路](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486398&idx=2&sn=964ad7e7047f9f7c8cfd9d5192a85f9d&scene=21#wechat_redirect)

◆🔥[为何Multi-Agent系统成功率不够高？14种失败模式大揭秘](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486388&idx=1&sn=c9c14d7916bd62531dd8a68dbb9cf6a5&scene=21#wechat_redirect)

◆[STEVE：让 AI 更智能地操控图形界面](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486248&idx=2&sn=3b0f5ee27f5a5d12c86881e998f2a61d&scene=21#wechat_redirect)

◆[AutoAgent：让AI智能体开发变得触手可及](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486232&idx=2&sn=152207453b6d69da4de1ac13a055f5c8&scene=21#wechat_redirect)

◆[AI 智能化的选择：API Agents 和 GUI Agents 的碰撞与融合](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486137&idx=1&sn=e0c594bd89fa949f49e5490ebd6e334e&scene=21#wechat_redirect)

◆[探索 MovieAgent：Multi-Agent CoT 规划的电影生成](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486137&idx=2&sn=018250db1ecf6c009b3a48d3e4deed8a&scene=21#wechat_redirect)

◆🔥[Agentic Workflows：让工作流更智能、更灵活](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486106&idx=1&sn=fcbea1f00ff05d72ed2e324fbec576f7&scene=21#wechat_redirect)

◆🔥[开源Agent通信协议对比分析：MCP、ANP、Agora、agents.json、LMOS、AITP (万字长文)](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486062&idx=1&sn=de427a336b0ee26411901f4d2dcfac7e&scene=21#wechat_redirect)

◆🔥[实用MCP Server分享，让Agent解锁 Claude AI 的无限可能](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247486041&idx=1&sn=deee296cc36e2a9b9a2fa192257506af&scene=21#wechat_redirect)

◆[2025 年金融行业 AI 工具大盘点：十大变革力量来袭](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485945&idx=1&sn=2f3b3d26808cd628aac31228fc5c638b&scene=21#wechat_redirect)

◆[OpenAI 发布新工具：让构建AI Agent智能体更简单](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485941&idx=1&sn=61219b79e26970154156432e89717217&scene=21#wechat_redirect)

◆🔥[TwinMarket：用 AI Agent 模拟市场行为，揭开金融市场的神秘面纱](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485941&idx=2&sn=632e7c759cd8754b208d61cdcce8457e&scene=21#wechat_redirect)

◆🔥[AI智能体的未来：硅谷投资风向、Manus的启示与OWL等开源探索](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485872&idx=1&sn=6d91a23bde48a567f321f4e2ea222096&scene=21#wechat_redirect)

◆🔥[从Manus到OpenManus：AI产品如何赢得未来？](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485827&idx=1&sn=4ed66df6f9880e7fe0d5abacea6f9391&scene=21#wechat_redirect)

◆🔥[干不过 AI 就加入它，MGX Agent 前端开发最佳实践-案例](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485808&idx=1&sn=622ef164dc4a172a9946777bab4b202a&scene=21#wechat_redirect)

◆🔥[MGX，开启 AI 软件开发新纪元，万字长文深度解析](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485210&idx=3&sn=fad66a91df14d7050ffe04e451554f4a&scene=21#wechat_redirect) 

◆[A-MEM：让 AI Agent 拥有动态记忆组织](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485758&idx=2&sn=0e8a944f6b42d327d857bd5cd9090453&scene=21#wechat_redirect)

◆🔥[PlanGEN：让 AI 规划更智能的多智能体框架](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485647&idx=1&sn=75fae50f64d52fb45723ce9eb8cc0d19&scene=21#wechat_redirect)

◆[MCTD：解锁 AI 规划的超级引擎](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485647&idx=2&sn=9a76972e501077891021f4a23555460b&scene=21#wechat_redirect)

◆[单智能体规划：多智能体系统中的最优决策框架](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485329&idx=1&sn=23e4773c6193125de93dee3c8288b8b7&scene=21#wechat_redirect) 

◆🚀 [四个平替 OpenAI Deep Research 的强大](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485210&idx=1&sn=086f0d8be535d550f5574b8f58f54499&scene=21#wechat_redirect)[开源工具](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485210&idx=1&sn=086f0d8be535d550f5574b8f58f54499&scene=21#wechat_redirect)

◆[CODESIM：多智能体代码生成与问题解决的新思路](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485295&idx=1&sn=b234bf53899fe81f4e59768b7ea408a4&scene=21#wechat_redirect)

◆[打破传统：多智能体架构探索的新范式 ——MaAS 框架解读](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485244&idx=1&sn=2394bc4c3991ca219004814fe624c496&scene=21#wechat_redirect)

◆[AI Agent基础设施：解锁潜力与管理风险的关键](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485133&idx=2&sn=5f31fcf1a55036ce31cef3d796dfb6dd&scene=21#wechat_redirect)

◆🔥[解锁 AI Agent 构建密码：六大开源框架解析](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484917&idx=1&sn=31637e03ccfe430402980393043c9855&scene=21#wechat_redirect)

◆🔥[AFLOW：用AI优化AI，开启高效工作流的新篇章](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484342&idx=1&sn=d48c452aa793a414bba78d29fb8291a7&scene=21#wechat_redirect)

◆🔥[2025 年 13 门免费 AI Agent 课程资源](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247483932&idx=2&sn=85aa238175e9e4721ade247f40c1b3c1&scene=21#wechat_redirect)

◆[使用 PydanticAI 框架快速构建 Multi-Agent 系统](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484006&idx=1&sn=cd4ca19a12d6661a101bc5e808adb4f0&scene=21#wechat_redirect)

◆🔥[Eko：用自然语言驱动前端开发，AI Agent 工作流新体验！](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484320&idx=1&sn=d5cbc7cc386254899affd9b6d53e4d95&scene=21#wechat_redirect)

◆[下一代AI Agent的"工具手"：MCP如何让AI自主操作数据库/浏览器/API](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484423&idx=1&sn=381cbeefd1b46115be547dabfe31b59f&scene=21#wechat_redirect)

◆[IntellAgent：对话式 AI 的评估框架](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484618&idx=2&sn=c935f054c8f02f8c3ac1220d0aa3c39f&scene=21#wechat_redirect)

◆[AI的自我进化之路：Multi-Agent系统的自主迭代优化](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484727&idx=1&sn=9411a74f525e7afe2a8512a4a5ddf7f9&scene=21#wechat_redirect)

◆🔥[从理论到现实：OpenAI 的 Operator 展示 CCA 的巨大潜力](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484783&idx=1&sn=584dd83b325d3bb431674677dd3cafa5&scene=21#wechat_redirect)

◆[AI Agent 实战：用 LangGraph 实现持久化与流式传输](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484786&idx=2&sn=fb53e44dd654a319012288dc32c6f2c9&scene=21#wechat_redirect)

◆🔥[Search-o1：动态检索 + 文档精炼，让 AI 推理解锁知识盲区](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484806&idx=1&sn=10e6f062305ba98fa2c6fba2f65c59ee&scene=21#wechat_redirect)

◆🔧 [CHRONOS：AI 迭代自我问答，精准构建新闻时间线](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484818&idx=2&sn=2d01ed5d6a7328e7d4758d8d8da91e35&scene=21#wechat_redirect)

◆🔥[AI学会自我反思？Agent-R 使用蒙特卡洛树搜索(MCTS)自我训练自动纠错，让AI更聪明](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484867&idx=1&sn=54dfc08be9c192f2f6d80811680f46ae&scene=21#wechat_redirect)

◆[为AI Agent设定边界：自然语言权限与结构化权限的结合](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484940&idx=2&sn=58698969deb4738f4e1a3d0278450b60&scene=21#wechat_redirect) 

◆🔥[AI 落地的抉择：函数、多工具Agent还是Multi-Agent？](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247484940&idx=1&sn=b814d22603fdabc0968ee7f8bd47bf2b&scene=21#wechat_redirect)

◆[Cline 3.3 新版本：编程界的 “安全卫士” 与 “效率先锋”](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485000&idx=1&sn=b8b79a6cce7f86dd2d647dbb93ea2b37&scene=21#wechat_redirect)

◆[Self-MoA：大道至简，聚焦单一模型打破传统MoA，简化LLM集成](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485073&idx=1&sn=4d699eee479c0375d1c3764c9097c98b&scene=21#wechat_redirect)

◆🔥[多智能体系统优化新突破：Mass 框架引领智能协作新思路](https://mp.weixin.qq.com/s?__biz=Mzk2NDA0MzcxNw==&mid=2247485093&idx=2&sn=2dfdcec6920e972ed37ebc2d99cd3fb0&scene=21#wechat_redirect)

注：本文素材由AI辅助翻译，内容由人工整理/审核发出

*欢迎*点****、**加****、**关注**。公号加⭐️精彩不错过**

---

我是肆〇柒🐝，一名热爱AI的互联网人。在这里，我分享自己的观察与思考，希望我的探索能激发同样热爱科技与生活的你，为你带来灵感与思考。

期待我们的不期而遇。点击👇🏻关注

---

*🙋‍♂️*入群交流

1. 公众号菜单点击“社群”，扫码入群。

2. 回复“入群”“加群”等，添加作者微信进群。