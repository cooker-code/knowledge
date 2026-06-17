---
title: 被 Claude 吞噬的应用层：从 MCP、Skills 到 PTC 的跨维度打击
author: 王人书
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487632&idx=1&sn=3e8558250b98c6dda4d38a8213d83ee4&chksm=9e420e9c744b75cc1ddc3c4221c5c92c223d4c8886090f96de4098977c024008342eb4bd1c12&mpshare=1&scene=24&srcid=1126jQcbNmwYhNdmbU3cRdbh&sharer_shareinfo=a59fceb1c1ed6b3de93563db88a93f03&sharer_shareinfo_first=a59fceb1c1ed6b3de93563db88a93f03#rd
---

# 点击👇🏻可关注，文章来自

## 大家好，我是Alden，LLM开发工程师。从 MCP 打通数据孤岛，到 PTC 实现代码即推理，Anthropic 用一年时间重塑 LLM 的交互协议，完成了从“提示工程”向“环境工程”的范式跃迁。这不仅是功能的叠加，更是对应用层的“跨维度打击”——基础模型正在逐步吞噬中间件，进化为新的操作系统（LLMOps）。 字数 3710，预计阅读 12 分钟

> 今天看到 *@五道口纳什* 同学在群里分享了 Anthropic 最新发布的3个实验性feature，其中 Programmatic Tool Calling (PTC) 使我大为感慨。在我看来，Anthropic 这家公司可以称之为 **“在有技术的公司中是野心最大的、在商业公司当中是技术最强的”**。（原文见文末）

回顾自 2024 年 11 月 25 日至 2025 年 11 月 24 日这整整一年，Anthropic 在 LLM 基础设施层面的布局呈现出极强的工程理性主义特征。除了当下最强模型 Claude Opus 4.5，Anthropic 通过 **Model Context Protocol (MCP)**、**Agent Skills** 与 **Programmatic Tool Calling (PTC)** 逐步完成了一场静悄悄的架构革命。这 不仅仅是功能的叠加，而是对“模型-环境”交互协议的标准化重塑。

一言以蔽之，这一年的核心演进趋势是：从基于文本的隐式“提示工程” (Prompting Engineering)，转向基于协议的显式“环境工程” (Environment Engineering)。将模型从单纯的对话者，重构为具备标准化 I/O、按需加载能力与逻辑执行沙箱的智能内核。

柒哥在7月份前瞻性地提出 Environmental Engineering 的概念

## 对比总览

下表总结了三种技术在系统架构层面的核心差异与定位：

| 维度 | MCP | Agent Skills | PTC |
| --- | --- | --- | --- |
| **核心解决的问题** | **数据孤岛与连接成本** ：解决  个模型连接  个数据源的  集成复杂度。 | **上下文污染与认知过载** ：解决海量工具/SOP 全部塞入 Prompt 导致的注意力分散与 Token 浪费。 | **交互延迟与逻辑脆弱性** ：解决多步工具调用产生的网络往返开销及复杂逻辑在 JSON Schema 中的表达瓶颈。 |
| **交互原语** | **资源 (Resources)** / **工具 (Tools)** / **提示 (Prompts)** | **技能定义 (Definitions)** / **按需加载 (Lazy Loading)** | **代码块 (Code Blocks)** / **沙箱执行 (Sandbox Execution)** |
| **LLM 自主性** | **中** ：模型决定何时调用工具或读取资源，但依赖 Client 提供的静态列表。 | **高** ：模型根据任务目标，自主决定检索并“学习”哪些具体技能手册。 | **极高** ：模型编写完整逻辑脚本，自主编排控制流 (Loop/If)，而非依赖外部驱动。 |
| **工程师控制权** | **数据接入层** ：控制数据暴露的边界与权限。 | **知识定义层** ：控制 SOP 的颗粒度与层次结构。 | **执行环境层** ：控制代码沙箱的依赖库、算力限制与安全围栏。 |
| **系统熵值** | **降低** ：通过标准化协议减少了定制化 API 胶水代码的混乱。 | **降低** ：通过渐进式披露减少了上下文窗口中的噪声干扰。 | **最低** ：通过确定性代码执行替代了概率性多轮对话，极大降低了执行路径的不确定性。 |

## 核心技术演进详解

### 1. MCP (Model Context Protocol): 标准化数据总线

**发布时间**：2024年11月25日

大家最熟悉的模型上下文协议，其设计哲学是 **Standardization & Decoupling (标准化与解耦)**，核心在于将“模型”与“数据源”彻底解耦，建立通用的 Client-Host-Server 架构。这在 OS 层面类似于 **Device Drivers / File System**——如同操作系统通过统一驱动挂载硬件，MCP 让 LLM 通过统一协议“挂载”数据资源。

MCP 架构图

#### 问题与背景

在 MCP 之前，连接 GitHub、Slack 或数据库需要为每个 AI 应用编写特定的 Connector，导致严重的碎片化和重复造轮子。核心痛点在于缺乏统一的“上下文协议”。

#### 实现机制

MCP 基于 JSON-RPC 协议定义了三种原语：

1. **Resources**：类似文件读取，用于被动获取数据（如日志、代码）。
2. **Tools**：类似函数调用，执行具体操作（如提交 Commit）。
3. **Prompts**：预定义的交互模板。

工程师只需构建一个 MCP Server，任何支持该协议的客户端（如 Claude Desktop, IDEs）即可即插即用。

#### 场景与效果

典型场景如本地开发环境连接，工程师可直接通过 MCP 连接本地 Postgresql 和 Git。这极大地降低了集成的边际成本，将“上下文获取”变成了标准化的基础设施服务。例如，一个 `git-mcp-server` 暴露读取仓库和创建分支的能力，用户授权后，Claude 即可直接读取状态并修复代码，无需手动复制粘贴。

### 2. Agent Skills: 动态能力的渐进式披露

**发布时间**：2025年10月16日

Agent Skills 的设计哲学是 **Progressive Disclosure & Composability (渐进式披露与组合性)**，核心在于从“单体大模型”向“模块化认知工程”转型。在 OS 层面，这完美对应了 **Header Files / Dynamic Linking (.h / .so)**——Agent 启动时仅加载轻量级的元数据（如同编译器读取头文件），仅在运行时根据任务调用栈的需求，动态加载具体的技能包（如同按需加载动态链接库）。

Agent Skills 架构图

#### 问题与背景

随着 Agent 接管复杂任务，传统的 Context Management 面临“带宽瓶颈”。将几百页的 PDF 规范或庞大的代码库文档一次性 Dump 进 Prompt，不仅成本高昂，更会稀释模型注意力，导致“认知过载”和幻觉。同时，Prompt 难以像代码一样进行模块化复用和版本控制。

#### 实现机制

Agent Skills 采用“文件系统即知识库”的架构，每个 Skill 是一个独立的文件夹。

1. **元数据索引 (Indexing)**：每个 Skill 包含一个 `SKILL.md`，其头部的 YAML Frontmatter 定义了技能名称与简介。模型启动时仅读取这部分极小的上下文。
2. **懒加载 (Lazy Loading)**：Progressive Disclosure，当模型判断当前任务需要特定能力时，会通过 Bash 工具（如 `cat`）主动读取具体的 Markdown 指南或关联文档。
3. **组合式构建 (Composability)**：技能像 npm 包或积木一样，支持独立开发、测试并挂载到不同的 Agent 上。

```
## Overview  
  
This guide covers essential PDF processing operations using Python libraries  
  
## Quick Start  
  
``python  
from pypdf import PdfReader, PdfWriter  
  
# Read a PDF  
reader = PdfReader("document.pdf")  
print(f"Pages: {len(reader.pages)}")  
  
# Extract text  
text = ""  
for page in reader.pages:  
    text += page.extract_text()  
``
```

#### 场景与效果

典型场景如复杂的企业表单填写或遗留代码重构。它实现了 **Effectively Unbounded Context (有效无限上下文)**——理论上可以挂载 TB 级的知识库，因为模型只在运行时“取一瓢饮”。例如，一个“税务申报 Agent”平时占用极低内存，只有在处理特定表格时，才会动态加载对应的 `forms.md` 规范和 `extract.py` 脚本，瞬间从通用模型切换为领域专家，且大幅降低了推理成本。

### 3. PTC (Programmatic Tool Calling): 代码即推理

**发布时间**：2025年11月24日

Programmatic Tool Calling， 可翻译为“程序化工具调用”，其设计哲学是 **Compute over Context & Coding as Reasoning**，实现了“概率”与“逻辑”的正交分离。在 OS 层面，这类似于 **Shell Scripting / Kernel Execution**——模型不再是呆板地发送单个 API 请求，而是编写完整脚本提交给内核批量执行。搭配同时发布的 **Tool Search Tool, Tool Use Examples** 一起使用，Anthropic 称可将工具调用的 token 消耗减少70%-90%。

PTC 架构图

#### 问题与背景

传统的 Tool Use 是“多轮对话乒乓”，存在两大缺陷：

1. **延迟与成本**：简单循环需要多次网络往返。
2. **逻辑脆弱**：用 JSON 表达复杂控制流（Loop/If）非常笨拙且易错。

#### 实现机制

PTC 允许模型直接编写一段 Python 代码，在安全隔离的沙箱中运行。代码可包含变量、循环、运算及外部函数调用。模型只需输出一次，沙箱执行完毕后返回最终结果。

```
team = await get_team_members("engineering")  
  
# Fetch budgets for each unique level  
levels = list(set(m["level"] for m in team))  
budget_results = await asyncio.gather(*[  
    get_budget_by_level(level) for level in levels  
])  
  
# Create a lookup dictionary: {"junior": budget1, "senior": budget2, ...}  
budgets = {level: budget for level, budget in zip(levels, budget_results)}  
  
# Fetch all expenses in parallel  
expenses = await asyncio.gather(*[  
    get_expenses(m["id"], "Q3") for m in team  
])  
  
# Find employees who exceeded their travel budget  
...
```

#### 场景与效果

特别适用于数据分析与批量操作。两到三名超出预算的人员。两千多行明细条目、中间汇总数据和预算查询都不会影响 Claude 的上下文，从而将原始支出数据从 200KB 压缩至仅 1KB 的结果数据量。同时，循环和判断由解释器保证确定性，而非依赖 LLM 的概率预测，极大地提升了逻辑鲁棒性。

> 例如，“找出素数并求和”的任务，传统方式需请求 100 次工具，而 PTC 方式下模型只需编写一行 `sum([n for n in numbers if is_prime(n)])` 即可直接获得结果。

## 对 LLM 工程师与系统工程的影响

从 MCP 到 PTC 的演进，清晰地勾勒出 LLM 工程师职能的迁移路径：从 **Prompt Engineer** 进化为 **Context Engineer**，最终抵达 **Environment Engineer**。

### 1. Prompt 变薄，Environment 变厚

工程师不再需要在 Prompt 中绞尽脑汁地描写复杂的“思维链”或通过 Few-shot 强行规定输出格式。现在的重点转移到了构建**环境**：

* **构建 MCP Servers**：将企业内部数据和服务封装成标准接口。
* **编写 Skills Manuals**：将领域知识（Know-how）结构化为模型可按需调用的技能手册。
* **配置 Runtime Environment**：为 PTC 提供安全的执行沙箱和预置的依赖库。

### 2. 新关注维度

工程师现在需要像设计微服务架构一样设计 Agent 系统。

* **可调用的工具库 (Tooling Strategy)**：决定哪些逻辑应该硬编码在工具里，哪些逻辑留给模型编写代码实现。
* **系统感知能力 (System Observability)**：确保模型能感知到环境的状态变化（如文件系统的变动），而非仅仅依赖对话历史。

这标志着 AI 开发从“炼丹术”（调整 Prompt 词句）逐步进入了“土木工程”阶段（搭建稳固的基础设施与交互协议）。

## 基座模型厂商的平台化趋势

Anthropic 的这一系列动作揭示了一个明显的趋势：**Model Provider 正在通过协议标准化，逐步吞噬应用层的“薄封装”(Thin Wrappers)。**

过去，像 LangChain 或 LlamaIndex 这样的中间件框架，很大一部分价值在于处理数据连接、工具编排和内存管理。

* **MCP** 直接在协议层解决了数据连接问题，削弱了第三方 Connector 的价值。
* **Agent Skills** 在模型侧原生实现了复杂的路由和显存管理，替代了应用层的 RAG 检索路由逻辑。
* **PTC** 将代码解释器和逻辑编排内化为模型的一种原生交互模式，降低对外部 Chain/Loop 编排工具的依赖。

这种下沉意味着，构建在 LLM 之上的简单套壳应用将大幅缩小生存空间。模型厂商正在成为新的操作系统（LLMOps），提供算力（LLM）、I/O（MCP）、驱动管理（Skills）和执行环境（PTC）。

## LLM 工程师的新大陆在哪

在这一技术演进下，LLM 工程师的核心价值不再是写出一段漂亮的 Prompt，而是**架构设计**与**边界管控**。

新大陆可能的探索方向：

1. **确定性与概率性的边界设计**：利用 PTC，精确划分哪些任务交给 Python 解释器（确定性），哪些交给 LLM 推理（概率性）。这是一种新型的混合编程范式。
2. **语义层面的接口契约**：设计高质量的 Agent Skills 和 MCP 接口描述。这相当于为 AI 编写“头文件”，其命名的准确性和文档的清晰度直接决定了系统的智能上限。
3. **安全架构**：当模型能够编写代码并执行（PTC），能够连接任意数据源（MCP）时，如何设计沙箱隔离、权限控制和审计机制，成为系统能否落地的决定性因素。

未来的 LLM 工程师，本质上是在为概率性的智能体构建一个确定性的、可观测的、安全的**数字空间**。

## 参考资料 [1]  Introducing the Model Context Protocol: *https://www.anthropic.com/news/model-context-protocol* [2]  Equipping agents for the real world with Agent Skills: *https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills* [3]  Introducing advanced tool use on the Claude Developer Platform: *https://www.anthropic.com/engineering/advanced-tool-use* [4]  Documentation and cookbook for Programmatic Tool Calling: *https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling*

关联阅读：

[LLM 推理引擎选型指南：Transformers、llama.cpp 与 vLLM 该怎么选？](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487624&idx=1&sn=494a35447b642fe8a865016e135a0796&scene=21#wechat_redirect)

[当Docker开始接管LLM部署，Ollama 的护城河还在吗？](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487618&idx=1&sn=a05eec88024532e59cc2a3476019871b&scene=21#wechat_redirect)

[AI for Coding：从 Vibe Coding 到规范驱动开发](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487527&idx=1&sn=6d63dcc5bfc87e643a7cf3723b671918&scene=21#wechat_redirect)

[LLM重塑系统边界：二阶序与认知工具的知识平权](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487467&idx=1&sn=e96a030fdf68cf0eb1bf100302a16738&scene=21#wechat_redirect)

[黑客、语言囚徒与技术时代的共构](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487438&idx=1&sn=38c98f1935e71961957412879b60afde&scene=21#wechat_redirect)

[Vibe coding：LLM 时代的“没有银弹”回响](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487420&idx=1&sn=9473debc94d7635caec40304ec72504e&scene=21#wechat_redirect)

[跨域的结构同构性](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487400&idx=1&sn=7ed8064bd816774c8a97080409acef50&scene=21#wechat_redirect)

[澄明之镜：Manus AI 与人的存在主义觉醒](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487242&idx=1&sn=313d3572793322e862118b11cd297938&scene=21#wechat_redirect)