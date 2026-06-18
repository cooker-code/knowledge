> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 谈谈 `Claude Skills`
author: 周末程序猿
date:
url: https://mp.weixin.qq.com/s?__biz=MzA3Njk4MjkyNw==&mid=2247487709&idx=1&sn=8374c48dc31c0d4d403b507ac643bff7&chksm=9e073432fe4eb184ddcdd678f48deace6a16f61ccec63d7a39f667b302dff1d49d9d54b40079&mpshare=1&scene=24&srcid=1029oHG0ORTWC53F4mONKYh7&sharer_shareinfo=e859a76cd1cee6a1b31e8899281f83d2&sharer_shareinfo_first=e859a76cd1cee6a1b31e8899281f83d2#rd
---

最近 Claude 在 `github` 上有一个比较火的项目：`https://github.com/anthropics/skills`，项目是围绕新概念 `Skills`，我觉得实际上是：通过工程化结构来实现智能体，用一种工程范式来完成子功能，就像C++代码中提出一种更好的设计模式来解决通用问题。

### 1. 什么是 `Skills`

`Skills` 是一组由说明文档、脚本与资源组成的 “技能包”，Claude 会在需要时动态加载，用于提升在特定任务上的一致性与表现，借助 `Skills`，Claude 能以可复用的方式完成某类任务，例如：按公司品牌规范创作文档、按组织特定流程分析数据，或自动化个人工作流。

### `Skills` 如何工作

`Skills` 通过 “逐步迭代” 的方式工作：当你发起任务时，Claude 会审视可用 `Skills`，自动挑选相关的技能并只加载完成任务所需的指令与资源，这样既能提升速度与效果，也能避免将无关内容塞满上下文窗口，从而保持对话与计算的高效与稳定。

**核心机制：**

1. 你提出任务请求
2. Claude 在可用 `Skills` 中进行匹配与选择
3. 自动加载相关技能的指令/脚本/资源
4. 按技能流程执行任务，提高一致性与完成质量

**优势：**

* 提升专项任务表现：为文档创作、数据分析与领域任务提供专门能力，补强通用模型的知识与流程执行力
* 组织知识沉淀：将组织流程、最佳实践与制度化知识打包，使团队成员与 Claude 保持一致的执行标准
* 易于定制：使用 Markdown 编写说明即可创建基础 `Skills`，如需更高级功能，可为自定义技能附加可执行脚本
* 易于组合：对于复杂的任务，可以通过多个 `Skills` 组合迭代处理

**SKILL.md 模版文件：**

```
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---

# My Skill Name

[Add your instructions here that Claude will follow when this skill is active]

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
```

### 2. 与 Claude 其他功能对比

**Skills vs. Projects：**

* Projects：为特定项目对话提供 “持续加载” 的静态背景知识
* Skills：提供 “按需激活” 的专门流程与操作指南，并可在任意对话中动态生效

**Skills vs. MCP（Model Context Protocol）：**

* MCP：将 Claude 连接到外部服务与数据源
* `Skills`：提供 “如何完成任务” 的流程性知识与操作步骤
* 二者可协同：MCP 负责接入工具与数据，`Skills` 负责教会 Claude 如何有效使用这些工具

**Skills vs. Prompt**

* Prompt：对所有对话生效，强调通用偏好与风格
* `Skills`：面向特定任务，仅在相关时加载，更适合专业工作流与场景化流程

### 3. `Skills` 目录结构

**`Skills` 目录结构如下：**

```
my-skill/
├── SKILL.md (required)
├── reference.md (optional documentation)
├── examples.md (optional examples)
├── scripts/
│   └── helper.py (optional utility)
└── templates/
    └── template.txt (optional template)
```

#### 3.1 基础结构

* `Skills` 目录与 `SKILL.md` 文件
* 每个 `Skills` 至少包含一个目录和其中的 `SKILL.md` 文件
* `SKILL.md` 顶部必须以 `YAML frontmatter` 开始，至少包含 `name` 与 `description` 两个必填元数据字段
* 其余内容采用 “迭代执行” 原则：先读元数据，必要时再加载正文、参考文件与脚本，避免上下文过载

#### 3.2 元数据

* name【必填】：人类可读的 Skill 名称（≤ 64 字符），示例：Brand Guidelines
* description【必填】：Skill 的用途与触发条件（≤ 200 字符；Claude 依此判断何时调用），示例：Apply Acme Corp brand guidelines to presentations and documents, including official colors, fonts, and logo usage.
* version【可选】：版本号（便于演进与回滚），示例：1.0.0
* dependencies【可选】：所需软件包，示例：python>=3.8, pandas>=1.5.0

#### 3.3 `SKILL.md` 的正文

* 当仅凭元数据不足以执行任务时，Claude 会加载正文
* 正文可包含：执行步骤、规范细则、示例、何时/何不适用、引用的资源文件等

**`SKILL.md`样例：**

```
---
name: Brand Guidelines
description: Apply Acme Corp brand guidelines to all presentations and documents, including official colors, fonts, and logo usage.
version: 1.0.0
dependencies: []
---

# Overview
This Skill provides Acme Corp's official brand guidelines for creating consistent, professional materials. When creating presentations, documents, or marketing materials, apply these standards to ensure all outputs match Acme's visual identity. Claude should reference these guidelines whenever creating external-facing materials or documents that represent Acme Corp.

## Brand Colors
- Primary: #FF6B35 (Coral)
- Secondary: #004E89 (Navy Blue)
- Accent: #F7B801 (Gold)
- Neutral: #2E2E2E (Charcoal)

## Typography
- Headers: Montserrat Bold
- Body text: Open Sans Regular
- Size guidelines:
  - H1: 32pt
  - H2: 24pt
  - Body: 11pt

## Logo Usage
- Use the full-color logo on light backgrounds; use the white logo on dark backgrounds.
- Maintain minimum spacing of 0.5 inches around the logo.

## When to Apply
Apply these guidelines when creating:
- PowerPoint presentations
- Word documents for external sharing
- Marketing materials
- Client-facing reports

## Examples
- Input: “Create a 10-slide pitch deck for Acme Corp’s new product.”
- Output: A PPT with the above colors, fonts, logo rules, and spacing applied consistently.

## Resources
See the resources folder for logo files and font downloads.
```

#### 3.4 `Skill` 涉及的资源

**`SKILL.md` 需要引入的资源文件**

* 当信息较多、且部分内容仅在特定情境需要时，可将补充材料拆分为独立文件（如 REFERENCE.md、CHECKLIST.md、TEMPLATES/）
* 在 `SKILL.md` 中引用这些文件，Claude 将按需加载，减少无关内容占用

**可执行脚本**

* 进阶 `Skill` 可附带可执行代码文件，供 Claude 调用以完成复杂任务（例如文档批量处理、数据清洗、可视化等）
* 常用语言与生态：

+ Python（pandas、numpy、matplotlib 等）
+ JavaScript / Node.js等

* 包管理约束：

+ Claude 与 Claude Code 在加载 `Skill` 时可从标准仓库安装依赖（Python PyPI、JavaScript npm）
+ 使用 API Skills 时，运行时无法临时安装新包，所有依赖必须预装在容器中

### 4. 在 Claude 上如何使用

**上传前：**

1. 通读 `SKILL.md`，确保指令清晰、步骤完备
2. 确认 description 能准确描述 “何时使用”
3. 校验被引用文件路径与命名无误
4. 以若干示例提示词本地自测，观察 Claude 是否会触发该 Skill

**上传到 Claude 后：**

1. 在 Settings > Capabilities 中启用该 Skill
2. 使用多种应触发它的提示词进行验证
3. 确认 Claude 在响应中已加载 Skill（例如输出中体现技能名称或遵循技能流程）
4. 若未按预期触发，迭代完善 description 与正文的触发条件与范围

### 5. 最佳实践：什么样的 `Skills` 才是好 “Skill”

* 指令清晰、可被 Claude 稳定执行
* 面向一个明确、可重复的任务
* 聚焦单一工作流：面向不同流程分别创建技能，小而专注的 Skills 更易组合复用
* 描述要具体：明确“适用对象、时机、边界与例外”，便于 Claude 正确判定调用
* 从简入手：先用 Markdown 指令跑通骨架，再逐步引入脚本与复杂逻辑
* 加示例：在 Skill.md 放入典型输入/输出，帮助对齐 “何为成功”
* 版本化：持续维护 version 字段，便于回溯与排错
* 渐进测试：每次重要变更后快速验证，避免一次性 “大改大上”
* 组合性：尽管 Skills 不能显式互相引用，但 Claude 可自动同时使用多个技能，发挥 “组装效应”

### 6. `Claude Skills` 官方示例

Claude 官方提供 `https://github.com/anthropics/skills` 20+个样例，本文就拆解其中一个比较常用的，自动执行 `Web 应用程序测试` ，目录结构如下：

```
webapp-testing/
├── SKILL.md
├── scripts/
│   └── with_server.py
└── examples/
    └── console_logging.py
    └── element_discovery.py
    └── static_html_automation.py
```

#### 6.1 功能描述

根据 description 介绍当前功能，用于使用 Playwright 交互和测试本地 Web 应用程序的工具包，支持验证前端功能、调试 UI 行为、捕获浏览器截图以及查看浏览器日志。

```
---
name: webapp-testing
description: Toolkit for interacting with and testing local web applications using Playwright. Supports verifying frontend functionality, debugging UI behavior, capturing browser screenshots, and viewing browser logs.
license: Complete terms in LICENSE.txt
---
```

#### 6.2 正文-工作流程

这部分是正文重点的部分，介绍当前决策树的工作流，不过我觉得用时序图，EARS等也可以，只要能介绍清楚每个步骤需要做什么。

```
## Decision Tree: Choosing Your Approach

User task → Is it static HTML?
    ├─ Yes → Read HTML file directly to identify selectors
    │         ├─ Success → Write Playwright script using selectors
    │         └─ Fails/Incomplete → Treat as dynamic (below)
    │
    └─ No (dynamic webapp) → Is the server already running?
        ├─ No → Run: python scripts/with_server.py --help
        │        Then use the helper + write simplified Playwright script
        │
        └─ Yes → Reconnaissance-then-action:
            1. Navigate and waitfor networkidle
            2. Take screenshot or inspect DOM
            3. Identify selectors from rendered state
            4. Execute actions with discovered selectors
```

#### 6.3 正文-使用脚本示例

这部分主要是介绍脚本怎么使用，脚本执行前需要做什么，脚本执行后需要做什么等，如下：

```
## Example: Using with_server.py

To start a server, run `--help` first, then use the helper:
...
```

#### 6.4 正文-其他

其他部分就是包括一些流程需要的细节，注意事项，涉及的一些文件是做什么的等介绍，如下：

```
## Reconnaissance-Then-Action Pattern
...

## Common Pitfall
...

## Best Practices
...

## Reference Files
...
```

### 7. 总结

Claude `Skills` 是提供了一种工程范式，以前大家通过各种 `MCP`，`Agent` 等将功能组合起来，中间层通过 `Prompt` 粘合，一方面不容易维护和继承，另一方面没有规范会导致不稳定，但是 `Skills` 通过工程范式约束很大程度上解决 `AI` 项目的工程化问题；
同时每一个 `Skills` 文件夹就是小的 `Agent`，这些 `Skills` 又可以组合为一个复杂的 `Agent`，就像我们写代码一样，先有基础库，然后通过基础库再组合复杂工程逻辑，这个大概是就是从混沌到规范化的历程。

### 参考

（1）https://github.com/anthropics/skills
（2）https://docs.claude.com/zh-CN/docs/claude-code/skills