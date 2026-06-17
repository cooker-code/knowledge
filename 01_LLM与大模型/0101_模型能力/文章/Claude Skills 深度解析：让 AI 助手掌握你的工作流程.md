---
title: Claude Skills 深度解析：让 AI 助手掌握你的工作流程
author: Agent时代
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUyNDk2MTcyOQ==&mid=2247494648&idx=1&sn=e1b86db5045cded8e60cc78e3a0cd1fc&chksm=fb16828db36a8536fb0a24cb82f1a2a852ef9af278f897965f491ba550e1be03847361405ff1&mpshare=1&scene=24&srcid=1118rJgkGJ3SvyWvk0qqPfza&sharer_shareinfo=320fa241eefa15fb09c1f2d5a15c2445&sharer_shareinfo_first=320fa241eefa15fb09c1f2d5a15c2445#rd
---

> “
>
> "Skills 是 AI Agent 领域今年最被低估的创新。" 这是 Simon Willison 在 2025 年 10 月对 Claude Skills 的评价。2025 年 10 月 16 日，Anthropic 正式发布 Agent Skills 功能，通过简单的文件夹和 Markdown 文件，让 Claude 能够按需加载专业知识和工作流程。与复杂的 MCP 协议相比，Skills 以极简的设计理念和极高的实用性，为 AI Agent 的定制化提供了全新路径。

Claude Skills 将"AI 定制化"从复杂的协议实现简化为文件夹管理，Anthropic 官方数据显示，Skills 的元数据仅占用约 100 个 token，相比 GitHub 的 MCP 实现动辄数万 token 的开销，效率提升超过 100 倍。更重要的是，Skills 采用渐进式披露（Progressive Disclosure）架构，只在需要时加载详细内容，这种设计让 Claude 可以同时装备数十个专业技能而不会造成上下文窗口拥堵。

## 1 什么是 Claude Skills

Claude Skills 本质上是包含指令、脚本和资源的文件夹，Claude 会根据任务需要自动加载相关技能。Anthropic 将其定义为"可组合、可移植、高效且强大"的模块化能力系统。

*图1：Skills 在 Agent 虚拟机中的文件系统结构，展示了 Skills 目录如何与 Agent 配置和 MCP 服务器协同工作*

**核心特性**：

**可组合性（Composable）**：多个 Skills 可以自动协同工作。你可以同时装备品牌设计规范、技术文档标准、法律条款库等多个 Skills，Claude 会根据任务自动调用相关组合。

**可移植性（Portable）**：相同的 Skill 格式可在 Claude.ai、Claude Code、Claude API 等所有平台使用，无需针对不同环境修改。

**高效性（Efficient）**：采用三层加载机制，元数据（约 100 token）始终加载，主体指令（通常小于 5k token）仅在触发时加载，资源文件按需访问。

**强大性（Powerful）**：可包含可执行代码，实现复杂任务的可靠执行，从数据处理到文档生成一应俱全。

## 2 Skills 的工作原理

### 2.1 渐进式披露架构

Skills 采用渐进式披露（Progressive Disclosure）设计，这是其高效性的核心。

*图2：Skills 的三层加载机制，从元数据到详细指令再到资源文件，逐层按需加载*

**三层加载机制**：

**Level 1 - 元数据层（约 100 tokens）**：始终加载，包含 Skill 的 name 和 description，Claude 通过这些信息判断是否需要触发该 Skill。

**Level 2 - 指令层（<5k tokens）**：当 Skill 被触发时加载 SKILL.md 的 Markdown 内容，包含主要工作流程和指导方针。

**Level 3 - 资源层（无限制）**：Claude 根据需要通过 Read 工具动态加载引用文件、模板、脚本等资源，这些内容不会一次性占用上下文窗口。

这种设计让你可以在一个 Skill 中打包几乎无限量的专业知识，而只在真正需要时才消耗上下文资源。

### 2.2 Skill 触发机制

Claude 使用"纯 LLM 推理"进行 Skill 选择，没有使用算法匹配、向量嵌入或分类器，而是完全依靠语言理解能力。系统在 Skill 工具的描述中提供格式化的可用 Skills 列表，Claude 根据 description 中的关键词和使用场景描述，判断是否匹配当前任务。

这意味着 **description 字段的质量直接决定 Skill 的可发现性**。一个好的 description 应该：

* 清晰说明 Skill 的功能
* 明确指出何时应该使用
* 包含相关的关键术语
* 使用第三人称描述

### 2.3 上下文注入机制

当 Skill 被触发时，系统通过两条独立的用户消息进行上下文注入：

**元数据消息（isMeta: false）**：对用户可见，显示哪个 Skill 被激活，保持透明度。

**Skill 提示消息（isMeta: true）**：对用户隐藏但发送给 Claude API，包含详细的工作流程指令，这种设计避免了 UI 被大量 AI 指令淹没的问题。

**为什么不放在系统提示中？** Skills 需要临时、限定范围的行为改变，用户消息配合 `isMeta: true` 标志可以让 Skill 效果局限于当前交互，而不是永久改变 Claude 在整个会话中的行为。

## 3 如何创建 Skills

### 3.1 基础结构

每个 Skill 的核心是一个 SKILL.md 文件，包含 YAML frontmatter 和 Markdown 内容。

*图3：SKILL.md 文件剖析，展示 YAML frontmatter 和 Markdown 内容的组织方式*

**最简 Skill 示例**：

```
---  
name: data-analyzer  
description: 专业数据分析助手，擅长处理 CSV 文件，生成统计报告和可视化图表。适用于需要数据清洗、统计分析、趋势识别的任务。  
---  
  
# 数据分析工作流程  
  
## 任务理解  
首先明确分析目标：  
- 数据来源和格式  
- 关键指标和维度  
- 预期输出格式  
  
## 数据处理步骤  
1. 使用 Read 工具加载数据文件  
2. 检查数据质量：缺失值、异常值、数据类型  
3. 根据需求进行清洗和转换  
4. 执行统计分析  
5. 生成可视化（如需要）  
6. 输出结构化报告  
  
## 输出规范  
- 统计结果保留 2 位小数  
- 图表使用 matplotlib 生成  
- 报告包含：数据概览、关键发现、建议行动
```

### 3.2 Frontmatter 字段详解

**必需字段**：

**name**（必需）：Skill 标识符，最大 64 字符，只能包含小写字母、数字和连字符。命名建议使用动名词形式（verb + -ing），如 `processing-pdfs`、`analyzing-spreadsheets`。

**description**（必需）：最大 1024 字符，这是 Claude 选择 Skill 的主要依据。应包含功能说明、适用场景、关键术语。

**可选字段**：

**allowed-tools**：预授权工具列表，可以避免用户频繁确认。例如 `"Read,Write,Bash"` 或使用范围限定如 `"Bash(git:*)"` 只允许 git 相关命令。

**model**：覆盖当前会话模型，可以为特定 Skill 指定使用 Opus、Sonnet 或 Haiku。

**disable-model-invocation**：禁止自动调用，适用于需要明确授权的危险操作。

### 3.3 文件组织最佳实践

**目录结构示例**：

```
my-skill/  
├── SKILL.md              # 核心提示模板  
├── scripts/              # 可执行 Python/Bash 脚本  
│   ├── validator.py  
│   └── processor.sh  
├── references/           # 加载到上下文的文档  
│   ├── terminology.md  
│   └── examples.md  
└── assets/               # 模板和静态资源  
    ├── template.docx  
    └── config.json
```

**关键原则**：

**简洁性优先**：SKILL.md 主体应保持在 500 行以内，只包含 Claude 没有的知识，挑战每一段内容："Claude 真的需要这个解释吗？"

**一层深度**：引用文件应该距离 SKILL.md 只有一层，使用描述性文件名（不要用 file1.md、file2.md），超过 100 行的文件应包含目录。

**使用变量**：在 SKILL.md 中使用 `{baseDir}` 引用资源路径而非硬编码绝对路径，确保跨环境可移植性。

**脚本与引用的区别**：

* 引用文件（references/）：通过 Read 工具加载到上下文中
* 资源文件（assets/）：仅通过路径引用，不加载内容
* 脚本文件（scripts/）：Claude 执行但不一定加载到上下文

## 4 实际应用案例

### 4.1 品牌合规检查 Skill

Anthropic 官方示例库中的 brand-guidelines Skill 展示了如何确保输出符合品牌标准：

```
---  
name: brand-guidelines  
description: Anthropic 品牌规范应用助手，确保所有输出内容符合品牌视觉标识、语言风格和设计标准。适用于创建营销材料、文档、演示文稿等需要品牌一致性的任务。  
allowed-tools: "Read,Write"  
---  
  
# Anthropic 品牌规范  
  
## 视觉识别系统  
品牌资源存储在 `{baseDir}/assets/` 目录：  
- 徽标文件：logo.svg、logo.png  
- 色彩代码：colors.json  
- 批准字体：fonts/  
  
## 设计原则  
创建品牌内容时遵循：  
1. 使用官方色彩值（#FF6B35 主色、#004E89 辅助色）  
2. 应用批准的字体层次（标题 Inter Bold、正文 Inter Regular）  
3. 保持 1.5 倍行高以确保可读性  
4. Logo 周围保持最小留白区域  
  
详细规范请查看 `{baseDir}/references/design-specs.md`
```

这个 Skill 打包了徽标文件、颜色代码、字体和布局模板，Claude 在创建任何品牌相关内容时会自动应用这些标准。

### 4.2 Slack GIF 创建 Skill

官方仓库中的 slack-gif-creator 展示了如何处理特定平台限制：

**核心价值**：自动检查生成的 GIF 是否符合 Slack 的 2MB 大小限制，如果超出则自动调整压缩参数重新生成。

**实现方式**：Skill 包含验证函数脚本，Claude 执行后如果发现文件过大，会自主迭代优化直到满足要求，无需人工干预。

这展示了 Skills 的"强大性"特征：通过可执行代码实现可靠的自动化工作流。

### 4.3 文档处理 Skills

Anthropic 提供了四个生产级文档 Skill：

**pdf Skill**：提取 PDF 文本和表格、合并/拆分文档、填写表单字段
**pptx Skill**：创建和编辑 PowerPoint 演示文稿，支持布局、图表、动画
**xlsx Skill**：生成包含公式、格式化、多工作表的专业 Excel 文件
**docx Skill**：创建符合标准的 Word 文档，支持样式、表格、图片

这些 Skills 展示了"高级模式"：处理复杂文件格式和二进制数据，包含完整的 Python 库调用和错误处理逻辑。

## 5 最佳实践指南

### 5.1 描述优化

**❌ 糟糕的 description**：

```
description: Helper for PDFs
```

问题：太模糊，没有说明功能范围和使用场景

**✅ 优秀的 description**：

```
description: 综合 PDF 工具包，用于提取文本和表格、合并/拆分文档、填写表单。适用于需要从 PDF 读取数据、重组文档、或生成填写后的表单文件的任务。
```

优点：明确功能、列举场景、包含关键术语（提取、合并、表单）

### 5.2 自由度控制

根据任务的"脆弱性"选择适当的指导级别：

**高自由度（适用于灵活任务）**：提供指导原则而非具体步骤，适合文本创作、分析报告等创造性任务。

**低自由度（适用于脆弱任务）**：提供具体脚本和详细步骤，适合数据处理、格式转换等容错率低的操作。

**示例对比**：

```
# 高自由度方法  
根据数据特征选择合适的可视化类型：  
- 趋势数据使用折线图  
- 分类比较使用柱状图  
- 占比关系使用饼图  
  
# 低自由度方法  
执行以下 Python 脚本生成可视化：  
```python  
import matplotlib.pyplot as plt  
data = load_data()  
plt.plot(data['x'], data['y'])  
plt.savefig('output.png')
```

```
### 5.3 验证与反馈循环  
  
对于质量关键的任务，建立"运行验证器 → 修复错误 → 重复"循环：  
  
```markdown  
## 数据处理工作流  
  
1. 执行 `scripts/process_data.py` 处理原始数据  
2. 运行 `scripts/validator.py` 检查输出质量  
3. 如果验证失败，根据错误消息调整参数  
4. 重复步骤 1-2 直到通过验证  
5. 生成最终报告
```

脚本应该主动解决问题而非静默失败，提供有用的错误消息和替代方案。

### 5.4 多模型测试

不同的 Claude 模型（Haiku、Sonnet、Opus）能力不同，Skill 的有效性依赖于底层模型。在发布前应该测试：

* Haiku：速度最快，适合简单任务
* Sonnet：平衡性能，通用场景
* Opus：最强能力，复杂推理

确保你的 Skill 在目标模型上能正常工作。

### 5.5 评估驱动开发

**先构建测试用例，再编写详细文档**。创建一组代表性任务场景，在真实使用中验证 Skill 是否解决实际问题，避免过度设计。

**迭代优化**：用一个 Claude 实例创建 Skill，用另一个测试。观察 Claude 如何导航你的 Skill 结构，根据实际使用模式调整组织方式。

## 6 Skills vs MCP：技术对比

### 6.1 Token 效率

**Skills**：元数据约 100 tokens，主体小于 5k tokens，资源按需加载
**MCP**：GitHub 官方实现的单个 server 可能消耗数万 tokens

效率差距超过 100 倍，这意味着你可以同时装备数十个 Skills，而 MCP 组合 3-4 个 server 就会遇到上下文限制。

### 6.2 实现复杂度

**Skills**：Markdown + 可选脚本，任何人可以在几分钟内创建
**MCP**：需要实现完整协议规范，涉及多层架构

Simon Willison 评价："Skills 简单得令人震惊，同时又足够强大。"

### 6.3 语言与模型无关性

**Skills**：纯 Markdown 和脚本，可跨不同 LLM 使用（理论上）
**MCP**：绑定特定协议实现

Skills 的可移植性更强，更容易分享和迭代。

### 6.4 组合能力

**Skills**：可以自由组合，Claude 自动判断调用哪些
**MCP**：token 开销限制了实际组合数量

Skills 的渐进式披露设计使其在组合场景中具有明显优势。

## 7 平台支持与部署

### 7.1 Claude.ai（网页端）

**可用计划**：Pro、Max、Team、Enterprise 用户

**内置 Skills**：PowerPoint、Excel、Word、PDF 文档处理

**自定义 Skills**：通过 `skill-creator` 交互式工具创建，或手动上传文件夹

**触发方式**：自动触发，Claude 根据任务自动选择相关 Skills

### 7.2 Claude API（开发者平台）

**新增端点**：`/v1/skills` 提供编程控制

**要求**：需要启用 Code Execution Tool beta

**管理方式**：通过 Claude Console 创建、版本管理和部署 Skills

**共享范围**：工作区级别共享，团队成员可复用

### 7.3 Claude Code（IDE 集成）

**安装方式**：

* 通过 anthropics/skills 市场插件安装
* 手动放入 `~/.claude/skills` 目录

**加载机制**：自动加载，当工作流程相关时触发

**特殊权限**：完整网络访问权限和本地包安装能力

### 7.4 Claude Agent SDK

**集成方式**：通过配置文件指定 Skills 路径

**运行环境**：自定义 Agent 虚拟机环境

**安全限制**：API 环境无网络访问、无运行时包安装，仅支持预配置依赖

## 8 安全考虑

Anthropic 官方文档明确警告："**仅使用来自可信来源的 Skills，并彻底审计所有打包文件。**"

**潜在风险**：

**恶意工具调用**：恶意 Skills 可以指示 Claude 以有害方式调用工具，例如删除文件、修改系统配置

**数据泄露**：通过允许的工具（如网络请求）将敏感数据发送到外部服务器

**权限滥用**：`allowed-tools` 字段预授权工具，恶意 Skill 可能滥用这些权限执行未经用户确认的操作

**防护建议**：

1. **审查来源**：只使用官方或可信开发者的 Skills
2. **检查代码**：仔细检查 SKILL.md 和所有脚本文件
3. **限制权限**：使用 `allowed-tools` 的范围限定功能，如 `Bash(git:*)` 只允许 git 命令
4. **隔离测试**：新 Skills 先在测试环境验证行为
5. **版本控制**：通过 API 的版本管理追踪 Skills 变更历史

## 9 未来展望

Claude Skills 代表了 AI Agent 定制化的新方向：**通过简单的文档而非复杂的协议实现专业化能力**。

**可能的发展方向**：

**Skill 市场生态**：社区驱动的 Skills 分享平台，就像 npm 之于 JavaScript，开发者可以发布、评级、组合 Skills

**跨模型兼容**：Skills 的语言无关特性使其有潜力成为通用 AI Agent 技能格式，不限于 Claude

**企业级编排**：通过 API 实现 Skills 的程序化组合、A/B 测试、性能监控

**领域专家 Agents**：通过组合多个深度 Skills 创建真正的领域专家，例如法律顾问 Agent = 法律条款库 + 案例分析 + 文档生成

**自动化 Skill 生成**：通过观察用户工作流程自动提取和封装为 Skills，降低创建门槛

正如 Simon Willison 所说："想象一下可组合的 Agent 能力——数据访问、数据库加载、发布、可视化、领域专业知识——通过简单文档实现自主工作流。" Skills 让这一愿景变得触手可及。

## 写在最后

Claude Skills 用极简的设计解决了 AI 定制化的核心问题：如何让 AI 既具备通用能力，又能深度专业化？通过 Markdown 文件和渐进式披露，它找到了效率与能力的最佳平衡点。

对于开发者而言，Skills 的学习曲线几乎为零——如果你会写 Markdown，你就会创建 Skills。对于企业用户，Skills 提供了将组织知识和流程封装为可复用模块的简单路径。对于整个 AI 行业，Skills 展示了"少即是多"的设计哲学：有时最强大的创新来自最简单的抽象。

现在就开始创建你的第一个 Skill 吧！从一个简单的工作流程文档开始，逐步打包你的专业知识，让 Claude 成为真正懂你的 AI 专家。

---

**问题与交流**：你会用 Skills 封装什么样的专业知识？是数据分析流程、代码审查规范，还是内容创作模板？在评论区分享你的想法，让我们一起探索 AI 定制化的更多可能性。

如果这篇深度解析帮你理解了 Claude Skills 的技术原理和实践方法，点个赞👍和推荐❤️，让更多开发者知道这个强大工具。

转发给做 AI 开发的朋友，Skills 的简洁设计值得每个人了解。

---

*⚠️ 本文由 AI 辅助生成，内容可能存在事实性错误或理解偏差，请读者注意甄别核实。*

---

*数据来源：*

* *Anthropic 官方文档：https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview[1]*
* *Anthropic 工程博客：https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills[2]*
* *Anthropic Skills 公告：https://claude.com/blog/skills[3]*
* *GitHub Skills 仓库：https://github.com/anthropics/skills[4]*
* *Simon Willison 技术分析：https://simonwillison.net/2025/Oct/16/claude-skills/[5]*
* *Lee Han Chung 深度解析：https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/[6]*

*题图来源：Anthropic 官方技术文档*

### 引用链接

[1]*https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview*

[2]*https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills*

[3]*https://claude.com/blog/skills*

[4]*https://github.com/anthropics/skills*

[5]*https://simonwillison.net/2025/Oct/16/claude-skills/*

[6]*https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/*