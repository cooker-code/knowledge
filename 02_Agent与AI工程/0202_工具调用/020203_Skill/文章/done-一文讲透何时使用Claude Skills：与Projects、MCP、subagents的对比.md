> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 一文讲透何时使用Claude Skills：与Projects、MCP、subagents的对比
author: AI咖啡馆
date:
url: https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486656&idx=1&sn=e176ee5e2dc38a0e1238b3f617cfdf23&chksm=9a58a5913d573b78e80be9f374a3f5d259eb0d9ac2713241aaafc1d427333816c890c8a4cbe8&mpshare=1&scene=24&srcid=1119R2lJnLY9mqvGOEWauqei&sharer_shareinfo=11c5566250cd0b1a14bc3d358968357a&sharer_shareinfo_first=11c5566250cd0b1a14bc3d358968357a#rd
---

# 导读

**先说关于 Skills 的几个判断：（围绕 Claude / MCP / Context Engineering）**

1. 在 Claude 生态 里，Skills 比 MCP 更值得个人和小团队优先关注，它是当下「Context Engineering 最佳实践」。

2. Skills 把「某一类任务的 SOP / 工作流」显式写出来，强化了 Claude Code 作为通用 Agent 的落地能力，兼具「通用 + 专业」。

3. Skills 更像 AI Coding 时代的代码高级封装：封装的不是函数，而是「任务 + 上下文 + 行为规范」。

---

**这是Claude Skills系列的第3篇，前面两分别是：**

* **[Claude推出Agent Skills: 上下文工程的优雅实践](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486292&idx=1&sn=71eb78883abf1e2f6da679bc85520094&scene=21#wechat_redirect)**
* **[如何使用Agent Skills优化前端设计](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486628&idx=1&sn=73d4d3b8721d09a932c8989c725ef421&scene=21#wechat_redirect)**

**后续文章会探讨：**

* 优质 Skills 工具推荐
* 开源版本的 Agent Skills 介绍
* 如何在Codex、Cursor中使用 Skills？

---

“Skills” 作为一种创建自定义 AI 工作流和 Agent 的工具，其功能日益强大。但它在 Claude 的技术栈中究竟扮演何种角色？本文将解析各种工具的适用时机，以及它们之间协同工作的机制。

自 Skills 功能推出以来，开发者对于 Claude 的 Agentic 生态系统中各组件如何协同工作非常感兴趣。

无论是利用 Claude Code 构建复杂的工作流，通过 API 创建企业级解决方案，还是在 Claude.ai 上最大化个人生产力，深刻理解“何时”与“何种”工具的选择，都将彻底改变用户与 Claude 协同工作的方式。

本文将分解阐述每个构建模块，解释各自的适用时机，并展示如何将它们组合以实现强大的 Agentic 工作流。

## 理解你的 Agentic 构建模块

### 什么是 Skills？

**Skills 是包含指令、脚本和资源的文件夹**，Claude 会在任务相关时动态地发现并加载它们。可将其视为专业的培训手册，为 Claude 赋予特定领域的专业能力：从处理 Excel 电子表格到遵循组织机构的开发指南。

**Skills 的工作原理：** 当 Claude 遇到任务时，它会扫描可用的 Skills 以寻找相关匹配项。Skills 采用渐进式披露 (progressive disclosure) 机制：首先加载元数据 (约 100 token)，提供足量信息以便 Claude 判断 Skill 是否相关。仅在需要时才会加载完整指令 (小于 5k token)，而捆绑的文件或脚本也仅在被调用时才会加载。

**何时使用 Skills：** 当需要 Claude **一致且高效地执行专业任务**时，应选择 Skills。它们是以下场景的理想选择：

* • **组织工作流**：开发指南、合规流程、文档模板
* • **领域专业知识**：Excel 公式、PDF 操作、数据分析
* • **个人偏好**：笔记系统、编码模式、研究方法

**示例：** 创建一个 品牌指南 Skill，其中包含公司的色板、排版规则和布局规范。当 Claude 创建演示文稿或文档时，它会自动应用这些标准，无需用户每次都重复解释。

可进一步了解 Skills，并查看官方持续更新的 Skills 库。

### 关于提示词 (prompts)

提示词 是指用户在对话中以自然语言向 LLM 提供的指令。它们是即时、对话式且反应性的：用户在当下提供上下文和方向。

**何时使用提示词：** 在以下情况使用提示词：

* • 一次性请求：“总结这篇文章”
* • 对话式优化：“让语气更专业一些”
* • 即时上下文：“分析这些数据并找出趋势”
* • 临时指令：“将此格式化为项目符号列表”

**示例：**
请对这段代码进行一次全面的安全审查。我需要关注：
**1. 常见漏洞，包括：**

* • 注入缺陷（SQL、命令、XSS 等）
* • 身份验证和授权问题
* • 敏感数据泄露
* • 安全配置错误
* • 访问控制损坏
* • 加密失败
* • 输入验证问题
* • 错误处理和日志记录问题

**2. 对于发现的每个问题，请提供：**

* • 严重级别（严重/高/中/低）
* • 代码中的位置（行号或函数名）
* • 解释为何存在安全风险及其被利用的方式
* • 在可能的情况下提供包含代码示例的具体修复建议
* • 防止类似问题的最佳实践指南

**3. 代码上下文：**

描述代码功能、语言/框架及其运行环境。例如：“这是一个 Node.js REST API，用于处理用户身份验证和支付数据”

**4. 额外考量：**

* • 是否存在任何 OWASP Top 10 漏洞？
* • 代码是否遵循了 [特定框架/语言] 的安全最佳实践？
* • 是否存在已知漏洞的依赖项？
  请按严重性和潜在影响对发现的问题进行排序。

**专业提示：** 提示词是与 LLM 交互的主要方式，但它们不会跨对话持久存在。对于重复的工作流或专业知识，应考虑将提示词捕获为 Skills 或项目指令。

**何时应改用 Skill：** 如果发现自己在多个对话中重复输入相同的提示词，那么就该创建一个 Skill 了。将诸如“使用 OWASP 标准审查此代码的安全漏洞”或“使用执行摘要、关键发现和建议来格式化此分析”之类的重复性指令转化为 Skills。这可以省去每次重新解释流程的麻烦，并确保执行的一致性。

查看Claude官方的提示词库、提示词最佳实践或智能提示词生成器以开始使用。

### 什么是 Projects？

**Projects 是独立的、自成一体的工作空间**，拥有各自的聊天历史和知识库。每个项目包含一个 200K 的上下文窗口，用户可以上传文档、提供背景信息，并设置适用于该项目内所有对话的自定义指令。

**Projects 的工作原理：** 用户上传到项目知识库的所有内容，都可在该项目的所有聊天中被访问。Claude 自动使用这些上下文来提供信息更充分、更相关的回复。当项目知识接近上下文限制时，Claude 会无缝启用检索增强生成 (RAG) 模式，将容量扩展高达 10 倍。

**何时使用 Projects：** 在需要以下功能时，应选择 Projects：

* • **持久上下文**：应为每次对话提供信息的背景知识
* • **工作空间组织**：为不同计划分离独立的上下文
* • **团队协作**：共享的知识和对话历史（适用于团队和企业版）
* • **自定义指令**：项目特定的语气、视角或方法

**示例：** 创建一个“第四季度产品发布”项目，其中包含市场研究、竞品分析和产品规格。此项目中的每一次聊天都可以访问这些知识，无需用户重新上传或解释上下文。

**何时应改用 Skill**： Projects 和 Skills 解决的是不同需求。当你需要一个持久的知识库和上下文工作空间来服务于特定计划时（例如，为“第四季度产品发布”上传报告和规格），请使用 Projects。**Project 提供的是静态的“背景知识” (what)**。当你需要**可移植的、流程化的专业知识 (how)**，且这种知识需要能被**动态应用于任何项目或对话**时（例如，一个“如何分析数据”的 Skill 或一个“品牌指南” Skill），请使用 Skills。Skills 是可复用的能力，而 Projects 是集中化的上下文。

注：Claude官网关于何时使用 Skills 取代 Projects 的描述存在明显的编辑错误（粘贴成了 subagents 的内容），上面是结合 Projects 文档整理的。

参考文档：什么是Projects。

### 什么是 subagents？

subagents 是专业的 AI 助手，拥有各自的上下文窗口、自定义系统提示和特定的工具权限。subagents 在 Claude Code 和 Claude Agent SDK 中可用，它们可独立处理离散任务，并将结果返回给主 Agent。

**subagents 的工作原理：** 每个 subagent 都按其自身的配置运行：用户定义了它的功能、处理问题的方式以及可以访问的工具。Claude 会根据 subagents 的描述自动将任务委派给它们，用户也可以明确请求一个特定的 subagent。

**何时使用 subagents：** 在以下情况使用 subagents：

* • **任务专业化**：代码审查、测试生成、安全审计
* • **上下文管理**：在将专业工作分流出去的同时，保持主对话的专注
* • **并行处理**：多个 subagents 可以同时处理不同方面
* • **工具限制**：将特定 subagents 限制在安全操作范围内（例如，只读访问）

**示例：**
创建一个代码审查 subagent，使其有权访问 Read、Grep 和 Glob 工具，但无权访问 Write 或 Edit。当用户修改代码时，Claude 会自动委派给此 subagent 进行质量和安全审查，而不会有意外更改代码的风险。

**何时应改用 Skill：** 如果多个 Agent 或对话需要相同的专业知识，例如安全审查流程或数据分析方法，应创建一个 Skill，而不是将该知识构建到各个 subagents 中。Skills 是可移植和可复用的，而 subagents 则是为特定工作流而专门构建的。应使用 Skills 来传授任何 Agent 都能应用的专业知识；而在需要独立的、具有特定工具权限和上下文隔离的任务执行时，则使用 subagents。

参考文档：什么是subagents。

### 什么是 MCP？

MCP 在 AI 应用程序与你现有的工具和数据源之间创建了一个通用的连接层。

模型上下文协议 (MCP) 是一种开放标准，用于将 AI 助手连接到数据所在的外部系统，如内容存储库、业务工具、数据库和开发环境。

**MCP 的工作原理：** MCP 提供了一种将 Claude 连接到用户的工具和数据源的标准化方式。用户无需为每个数据源构建自定义集成，而是针对单一协议进行构建。MCP 服务器负责暴露数据和功能；MCP 客户端 (如 Claude) 则连接到这些服务器。

**何时使用 MCP：** 在需要 Claude 执行以下操作时，应选择 MCP：

* • 访问外部数据：Google Drive、Slack、GitHub、数据库
* • 使用业务工具：CRM 系统、项目管理平台
* • 连接到开发环境：本地文件、IDE、版本控制
* • 与自定义系统集成：用户专有的工具和数据源

**示例：** 通过 MCP 将 Claude 连接到公司的 Google Drive。现在 Claude 可以搜索文档、读取文件并引用内部知识，无需手动上传，这种连接是持久的并会自动更新。

**何时应改用 Skill：** MCP 负责将 Claude 连接到数据；Skills 则指导 Claude 如何处理这些数据。如果用户是在解释 **如何** 使用工具或遵循流程，例如“查询我们的数据库时，必须始终先按日期范围过滤”或“使用这些特定公式格式化 Excel 报告”，这便是一个 Skill。如果需要 Claude **访问** 数据库或 Excel 文件，这便是 MCP。两者应协同使用：MCP 用于连接，Skills 用于流程知识。

参考文档：MCP ，以及关于如何构建 MCP 服务器。

## 这些工具如何协同工作？

当把这些构建模块组合起来时，真正的力量才会显现。每一个模块都有其独特的目的，它们共同创造出复杂的 Agentic 工作流。

### 比较：选择正确的工具

注：

* Skills 与 Projects 都是跨对话的，即都可以在多个对话中复用。
* subagents 不支持跨对话，可以并行执行且上下文隔离，但其主 agent 可以获取所有 subagents 的返回。

### 示例 Agentic 工作流：研究 Agent

让我们构建一个结合了多种构建模块的综合研究 Agent。此示例展示了如何组装和激活一个用于竞品分析的 Agent。

**第 1 步：设置你的 Project**
创建一个“竞争情报” Project，并上传：

* • 行业报告和市场分析
* • 竞争对手产品文档
* • 来自 CRM 的客户反馈
* • 过往的研究摘要

\*添加项目指令：
从我方产品战略的视角分析竞争对手。专注于差异化机会和新兴市场趋势。用具体的证据和可操作的建议来呈现研究结果。

**第 2 步：通过 MCP 连接数据源**
为以下服务启用 MCP 服务器：

* • Google Drive (访问共享的研究文档)
* • GitHub (审查竞争对手的开源仓库)
* • Web 搜索 (获取实时的市场信息)

**第 3 步：创建专业 Skills**
创建一个 "competitive-analysis" (竞品分析) Skill：

```
# 我司 GDrive 导航 Skill ## 概述
针对 Meridian Tech 的 Google Drive 结构优化的搜索和检索策略。
使用此 Skill 可高效定位内部文档、研究和战略材料。
## Drive 组织结构
**顶层结构：**
- `/Strategy & Planning/` - OKR、季度计划、董事会演示
- `/Product/` - PRD、路线图、技术规格
- `/Research/` - 市场研究、竞品情报、用户研究
- `/Sales & Marketing/` - 案例研究、宣讲材料、活动资料
- `/Customer Success/` - 实施指南、成功指标
- `/Company Ops/` - 政策、组织结构图、团队名录
**命名约定：**
- 格式：`YYYY-MM-DD_DocumentName_vX`
- 最终版本标有 `_FINAL`
- 草稿包含 `_DRAFT` 或 `_WIP`
## 搜索最佳实践
1. **先宽泛，后过滤** - 使用文件夹上下文 + 关键词
2. **定位文档所有者** - 销售材料应来自 Sales/ 目录，而非根目录
3. **检查时效性** - 优先考虑过去 6 个月的文档以获取当前策略
4. **寻找“单一事实来源”** - 查找带有 `_FINAL`、`_APPROVED` 标记或位于 `/Archives/Official/` 中的文件
## 研究 Agent 工作流
1. 确定主题类别（产品、市场、客户）
2. 使用目标关键词搜索相关文件夹
3. 检索 3-5 份最新/最相关的文档
4. 与 `/Strategy & Planning/` 交叉引用以获取背景
5. 引用来源时注明文件名和日期
```

**第 4 步：配置 subagents (仅限 Claude Code/SDK)**
创建专业的 subagents：

`market-researcher` (市场研究员) subagent:

```
name: market-researcher
description: 研究市场趋势、行业报告和竞品格局数据。主动用于竞品分析。
tools: Read, Grep, Web-search
---
你是一名专注于竞争情报的市场研究分析师。在研究时：
1. 识别权威来源 (Gartner, Forrester, 行业报告)
2. 收集定量数据 (市场份额, 增长率, 融资情况)
3. 分析定性见解 (分析师观点, 客户评论)
4. 综合趋势和模式
用引文和置信度来呈现研究结果。
```

`technical-analyst` (技术分析师) subagent:

```
name: technical-analyst
description: 分析技术架构、实现方法和工程决策。用于技术性竞品分析。
tools: Read, Bash, Grep
---
你是一名技术架构师，负责分析竞争对手的技术选择。在分析时：
1. 审查公开仓库和技术文档
2. 评估架构模式和技术栈
3. 评估可扩展性和性能方法
4. 识别技术优势和局限性
专注于能为我们的产品决策提供信息的可操作的技术见解。
```

**第 5 步：激活研究 Agent**
现在，当用户向 Claude 提问：“分析我们的前三大竞争对手如何定位他们的新 AI 功能，并找出我们可以利用的差距”

将会发生以下情况：

1. 1. **Project 上下文加载**：Claude 访问用户上传的研究文档并遵循项目指令。
2. 2. **MCP 连接激活**：Claude 搜索用户的 Google Drive 以获取最近的竞品简报，并拉取 GitHub 数据。
3. 3. **Skills 介入**："competitive-analysis" Skill 提供了分析框架。
4. 4. **Subagents 执行** (在 Claude Code 中)：`market-researcher` 收集行业数据，而 `technical-analyst` 则审查技术实现。
5. 5. **提示词优化**：用户提供对话式指导：“请特别关注医疗保健领域的企业客户”。

**结果：** 一份全面的竞品分析报告，它借鉴了多个数据源，遵循了用户的分析框架，利用了专业知识，并在整个研究项目中保持了上下文的连贯性。

# 常见问题与总结

### Skills 是如何工作的？

Skills 使用 渐进式披露 (progressive disclosure) 来保持 Claude 的高效。在处理任务时，Claude 首先扫描 Skill 元数据 (描述和摘要) 以识别相关匹配项。如果 Skill 匹配，Claude 会加载完整的指令。最后，如果 Skill 包含可执行代码或参考文件，这些内容也仅在需要时才加载。

这种架构意味着用户可以拥有许多可用的 Skills，而不会耗尽 Claude 的上下文窗口。Claude 总是在需要时才精确地访问它所需要的内容。

### Skills 与 subagents：何时使用？

**何时使用 Skills：** 当希望任何 Claude 实例都能加载和使用某些功能时。Skills 就像培训材料，它们使 Claude 在所有对话中都更擅长执行特定任务。

**何时使用 subagents：** 当需要为特定目的设计的、能独立处理工作流的、完全自洽的 Agents 时。Subagents 就像拥有自己上下文和工具权限的专业雇员。

**何时协同使用：** 当需要 subagents 拥有专业知识时。例如，一个代码审查 subagent 可以使用 Skills 来获取特定语言的最佳实践，从而将 subagent 的独立性与 Skills 的可移植专业知识结合起来。

### Skills 与提示词：何时使用？

**何时使用提示词：** 当提供一次性指令、即时上下文或进行对话式往来时。提示词是反应性的、即时的。

**何时使用 Skills：** 当拥有需要重复使用的流程或专业知识时。Skills 是主动性的 (Claude 知道何时应用它们)，并且跨对话持久有效。

**何时协同使用：** 提示词和 Skills 天然互补。使用 Skills 提供基础专业知识，然后使用提示词为每个任务提供特定的上下文和精细调整。

### Skills 与 Projects：何时使用？

**何时使用 Projects：** 当需要背景知识和上下文来为关于特定计划的所有对话提供信息时。Projects 提供始终加载的静态参考材料。

**何时使用 Skills：** 当需要仅在相关时才激活的流程知识和可执行代码时。Skills 提供按需加载的动态专业知识，从而节省上下文窗口。

**何时协同使用：** 当既需要持久的上下文，又需要专业的能力时。例如，一个包含产品规格和用户研究的“产品开发” Project，再结合用于创建技术文档和分析用户反馈数据的 Skills。

**关键区别：** Projects 提供的是“你需要知晓的背景知识”。Skills 提供的则是“如何执行任务的方法”。Projects 提供一个用户在其中工作的知识库。Skills 则提供随处可用的能力，适用于任何对话、任何项目。

### Subagents 可以使用 Skills 吗？

可以。在 Claude Code 和 Agent SDK 中，subagents 可以像主 Agent 一样访问和使用 Skills。这创造了强大的组合，让专业的 subagents 能够利用可移植的专业知识。

例如，用户的 `python-developer` subagent 可以使用 `pandas-analysis` Skill 来遵循团队规范执行数据转换，而 `documentation-writer` subagent 则可以使用 `technical-writing` Skill 来一致地格式化 API 文档。

# 开始使用

准备好使用 Skills 进行构建了吗？可以这样开始：

**Claude.ai 用户：**

* • 在“设置” → “功能/Features”中启用 Skills
* • 在 claude.ai/projects 创建你的第一个项目

+ • "Projects" 是独立的“工作空间”。启用 Skills 后，你可以创建一个 Project（例如“我的数据分析项目”）。在 Project 中，可以上传文件（如 CSV、PDF），这些文件将作为 Skills 可以处理的“知识库”。

* • 尝试将项目知识与 Skills 结合完成分析任务：

+ • 使用预置 Skills： 启用 Skills 后，你可以直接使用 Anthropic 提供的预置 Skills。例如，上传一个 .xlsx 文件到 Project 中，然后提问：“使用 Excel Skill 帮我分析这份数据并制作一个图表。”
+ • 创建自定义 Skill： 也可以使用一个名为 skill-creator 的特殊 Skill 来创建你自己的 Skill。示例提示： “@claude 使用 skill-creator 帮我创建一个 Skill，这个 Skill 知道我常用的 Python 编码风格指南。”

**API 开发者：**

在文档中探索 Skills 端点。

* • 探索 Skills 端点 (Endpoint)：

+ • "Skills" 功能是通过特定的 API 端点（如 /v1/skills）来管理的。你需要使用此端点来上传、管理和版本化你的自定义 Skills。
+ • 关键前提： 使用 API 版 Skills 功能，在调用 API 时必须在请求头 (headers) 中包含特定的 **beta 标识**，例如：

- • code-execution-2025-08-25
- • skills-2025-10-02
- • files-api-2025-04-14 (如果你的 Skill 需要处理文件)

* • 调用 Skills：

+ • 在 API 请求（例如 ``/v1/messages`）中，通过在 tools 中添加一个 container 参数来指定要使用的 Skills。
+ • 预置 Skills： 可以直接引用 Anthropic 托管的 Skills，例如：`"skill_id": "anthropic/pptx"` 或 `"anthropic/xlsx"`。
+ • 自定义 Skills： 需要先通过 /v1/skills 端点上传你的 Skill，获取一个 skill\_id，然后才能在请求中引用它。

**Claude Code 用户：**

* • 通过插件市场安装 Skills

+ • Claude Code 的功能主要通过插件和 Skills 来扩展。可以使用命令行来添加新的 Skills 或插件市场。
+ • 示例命令： `/plugin marketplace add anthropics/skills` (添加官方的 Skills 市场)

* • 管理你的 Skills

+ • 本地 Skills： 可以将 Skills 直接放在项目的 `.claude/skills/` 目录中，它们将自动被该项目加载。
+ • 全局 Skills： 也可以将通用的 Skills 放在本地计算机的 `~/.claude/skills/` 目录中，这样它们在你所有的 Claude Code 项目中都可用。

**查看官方 Skills 指南**

这是 Anthropic 官方的 GitHub 仓库，包含了大量代码示例和 Skills “配方”。

https://github.com/anthropics/claude-cookbooks/tree/main/skills

---

 

更多优质信息，请关注！

🔥推荐阅读

[近距离观察：打入Cursor内部的60天](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486564&idx=1&sn=5600505bc773cb8f4688538f280b7330&scene=21#wechat_redirect)

[化解MCP上下文瓶颈：工具调用 -&gt; 即时代码执行](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486544&idx=1&sn=bcdb0e8142b27cc0741bec31324e5b91&scene=21#wechat_redirect)

[30多位Founder眼中的Agentic AI落地现状](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486533&idx=1&sn=42b7753501f79d4fbabfa73a42b4e413&scene=21#wechat_redirect)

[AI能力6个月翻倍，关于AI与经济学的思考](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486521&idx=1&sn=a00d3cfc8e59d3c3355752629e0640ab&scene=21#wechat_redirect)

[创企Every的六位工程师的 AI 工作流程](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486481&idx=1&sn=85ca67791563343c01e9b2bd9b270015&scene=21#wechat_redirect)

[Github推出Agent HQ，适配任何Agent，任何工作方式](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486458&idx=1&sn=dec4d41471f5eedf1c235e3cdabf36e0&scene=21#wechat_redirect)

[规范驱动开发(SDD)的哲学：代码服务于规范](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486449&idx=1&sn=01c63d079c75771a0552ee6888e7ed99&scene=21#wechat_redirect)

[AI编码悖论：既令人震撼，也同样愚蠢](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486427&idx=1&sn=b60f547fb8de8998b67ce7da0bf86e1e&scene=21#wechat_redirect)

[RL之父萨顿：苦涩的教训](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486406&idx=1&sn=4e1996d28f5d1caaa8cea390ee7ff6ff&scene=21#wechat_redirect)

[a16z深度好文：AI正开启万亿美元级的新技术栈](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486400&idx=1&sn=0d3b356e9071d2d1ff9c7da39f3e4136&scene=21#wechat_redirect)

[数据模型：产品的基因与格局](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486340&idx=1&sn=aff6602919cef44d29e9ca2e87431025&scene=21#wechat_redirect)

[Claude推出Agent Skills: 上下文工程的优雅实践](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486292&idx=1&sn=71eb78883abf1e2f6da679bc85520094&scene=21#wechat_redirect)

[对AI的技术乐观与恰当恐惧](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486249&idx=1&sn=b5caaff802e90227f902daec633a050f&scene=21#wechat_redirect)

[只有5%的AI Agent生产可用？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486244&idx=1&sn=7823d2c3ed1f0eb8816766b26a36b3cc&scene=21#wechat_redirect)

[英伟达的AI帝国：投资100+家顶级初创企业](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486237&idx=1&sn=b46f3bb357c214081a72e1c0aec60fea&scene=21#wechat_redirect)

[Agents 2.0：从浅循环到深度 Agent](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486232&idx=1&sn=c045306d0d17735bf5f3c1bf0a6b4ee1&scene=21#wechat_redirect)

[有了AI，你还思考吗？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486224&idx=1&sn=e0ee4b2dad526c6ba7828e85a701df75&scene=21#wechat_redirect)

[Workflow vs Agent 构建器，LangChain创始人的思考](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486213&idx=1&sn=e3be1a4eb2e251f9bcfc93f4b302a556&scene=21#wechat_redirect)

[AI：我们时代的信息塑料](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486177&idx=1&sn=daeada73bfdf3be7f41539c29c3a9692&scene=21#wechat_redirect)

[验证的不对称性与验证器法则](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486125&idx=1&sn=268e856820a1e2292a20c493171e39b1&scene=21#wechat_redirect)

[「可验证性」是AI编程的极限所在](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486119&idx=1&sn=83e6941d88dfe9fb2088e9bfbac7162d&scene=21#wechat_redirect)

[OpenAI Codex 团队关于AI编程的洞见分享](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486093&idx=1&sn=b4c981019388bf787f034bba2e433ab7&scene=21#wechat_redirect)

[没有银弹：软件工程的本质与偶然](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486030&idx=1&sn=4a0be1382cd23b7e61e244363c3bc41e&scene=21#wechat_redirect)

[选择大模型的三个最关键因素](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486020&idx=1&sn=9438de82589c6ef09b1b33aaa3d8dede&scene=21#wechat_redirect)

[以「可读性」框架理解大型软件公司的怪现象](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485953&idx=1&sn=8162d34b772da077d25904e9b8e18c7d&scene=21#wechat_redirect)

[使用AI的术与道](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485892&idx=1&sn=8d6fefe21c82c3ed4a3d7cbba56f9986&scene=21#wechat_redirect)

[AI入侵文化，人类的想象力将归于何处？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485793&idx=1&sn=5ce171a02353775ee24d76681305e782&scene=21#wechat_redirect)

[Cline是如何思考上下文工程的？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485777&idx=1&sn=89b303e2f527c01c3e3a3c879c903a1a&scene=21#wechat_redirect)

[“心智模型”是人类工程师无法被替代的壁垒](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485738&idx=1&sn=286517469b0b2272f7b3bf69747916f6&scene=21#wechat_redirect)

[核心员工对OpenAI的思考](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485733&idx=1&sn=7bfe1cd2497c63b7c33866abf3564672&scene=21#wechat_redirect)

[我所了解的优秀系统设计（译）](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485640&idx=1&sn=566b0fbb36676f0811956c1fd5e1aeae&scene=21#wechat_redirect)

[YC创始人：做那些无法规模化的事](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485632&idx=1&sn=d42e44f00603841d543a06deceb5f8fc&scene=21#wechat_redirect)

[什么编程语言更适合Vibe Coding？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485517&idx=1&sn=e1ae86693d56834ab41b505c5ccb4fc7&scene=21#wechat_redirect)

[Manus的上下文工程：从构建AI Agent中学到的教训](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485469&idx=1&sn=7086d6be440f9274a5dd804d78f637af&scene=21#wechat_redirect)

[AK最新演讲：我们处于软件3.0时代](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485221&idx=1&sn=c3943017566bc03984351d8578a8bd7f&scene=21#wechat_redirect)

[“AI-Ready”是AI提效和转型的前提](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485178&idx=1&sn=53e69c655d4c587211e69a774ac2c4fd&scene=21#wechat_redirect)

[Anthropic：多Agent系统比单Agent得分高90%](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485136&idx=1&sn=edb797ad4ef690562b881cd8bbe11757&scene=21#wechat_redirect)

[AI原生时代，GUI转向“文本优先”的技术必然性](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485057&idx=1&sn=0ab49a9bd4f3ffbd7843acd4953a8636&scene=21#wechat_redirect)

[OpenAI关于人机关系的思考与实践](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485050&idx=1&sn=39268c00e593081927fe3ddbc7f39be8&scene=21#wechat_redirect)

[不要刻舟求剑，关于Vibe Coding的几点感想](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485006&idx=1&sn=19ff3c6bd200d6f94a82f695ae93864b&scene=21#wechat_redirect)

[写作即思考，YC创始人对“好文笔”的思考](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247484970&idx=1&sn=aa7e6f1238632ee4b1b6d74ce0091232&scene=21#wechat_redirect)

[AI的下半场：从解决问题到定义问题](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247484717&idx=1&sn=cadcd05afb486c704963dc4ee95001c6&scene=21#wechat_redirect)

[谷歌：欢迎来到AI的经验时代](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247484692&idx=1&sn=3a05eec5ea6c9af31c3f4b2db0ea7a4a&scene=21#wechat_redirect)

[没有prompt，一切都是上下文？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247484523&idx=1&sn=7497623fb31fc384ba76ec36f3c1b314&scene=21#wechat_redirect)

[如何用好AI大模型：苏格拉底式提问法](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247484418&idx=1&sn=c9a63801c07bc8c8cac23ce84fa608d3&scene=21#wechat_redirect)

#vibecoding  #skill #claude #project #Agent #智能体