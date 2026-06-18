> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code Sub-Agent 原理与实践：极简的深度调研 Agent
author: 棒棒糖兔
date:
url: https://mp.weixin.qq.com/s?__biz=MzA3ODM0NzkzNw==&mid=2648025402&idx=1&sn=ba6c80cd1a80ce75d7577bcd2b9a16d0&chksm=864d2ebf8546b07a932f70d25262d914b8ab7fd3b69a0f45aab3eb4b0d0e360cd8ed30d5af83&mpshare=1&scene=24&srcid=1106U0BVy87QueV7OqAC3DSg&sharer_shareinfo=5794ac6e41338ba035922926476e1417&sharer_shareinfo_first=5794ac6e41338ba035922926476e1417#rd
---

最近我利用 Claude Code 的 Sub-Agent 能力完成了一项关于"代码 Agent 产品"的调研，从输入一个复杂的研究任务，到输出 600+ 行的详细研究报告，整个过程完全自动化完成。

那么，Sub-Agent 到底是什么？它为什么如此强大？如何构建这样的系统？本文将简单解析这些问题。

目录：

1. 什么是 Sub-Agent？
2. 为什么需要 Sub-Agent？
3. Sub-Agent 的三大核心优势
4. Sub-Agent 的架构设计
5. Sub-Agent 的定义与使用
6. 案例：深度调研（Simple Deep Research示例）
7. Simple Deep Research 运行成果
8. 总结：Sub-Agent 的未来价值

01. 什么是 Sub-Agent？

Sub-Agent（子代理）是 Claude Code 独立于主对话运行的子系统。简单来说，它是一个预先配置好的、具有特定专业领域的 AI 人格，可以独立处理特定类型的任务。

02. 为什么需要 Sub-Agent？

传统的单一 AI 助手模式存在几个核心问题：

1. 上下文污染：复杂任务会产生大量中间信息，污染主对话的上下文
2. 能力泛化：一个通用 AI 很难在所有专业领域都表现出色
3. 安全风险：给 AI 过多权限存在安全隐患
4. 效率问题：处理复杂任务时容易迷失在细节中

Sub-Agent 通过"分而治之"的思路完美解决了这些问题。

03. Sub-Agent 的三大核心优势

1. 独立的上下文窗口

每个 Sub-Agent 都在自己独立的"房间"里工作，不会干扰主对话或其他 Sub-Agent 的思考过程。这极大地保护了主对话的上下文，使其始终聚焦于最高级别的目标。

2. 精确的工具权限控制

我们可以精确地为每个 Sub-Agent 分配它完成任务所必需的最小工具集。例如：

* 一个"代码审查" Agent 可能只需要 `git diff` 和 `Read` 权限
* 一个"文档生成" Agent 只需要 `Write` 权限
* 一个"数据分析" Agent 可能需要 `Bash` 和 `Read` 权限

这种精确的权限控制大大增强了系统的安全性。

3. 定制化的系统提示词

每个 Sub-Agent 都有自己的 System Prompt，这使得我们可以为特定领域的任务进行精细的指令微调，从而在特定任务上取得更高的成功率。

04. Sub-Agent 的架构设计

Sub-Agent 系统采用了"管理者-专家"的团队协作模式，主管理 Agent （启动 Claude Code 后与用户对话）负责任务的分解与最终结果的应用（传递给其他 Agent，或在主 Agent 内作为上下文）。专业 Sub-Agents 各自专注于特定领域的执行工作。

05. Sub-Agent 的定义与使用

每个 Agent 的核心是它的定义文件，这个文件就是它的"大脑"。创建 Agent 就是在 `.claude/agents/` 目录下创建一个 Markdown 文件。

在 Claude Code 中可以通过 /agents 命令创建 Agent。我们看一下数据 Agent 的数据结构。

---

推荐你看看腾讯公益项目

春蕾计划她们想上学

每人 20 元，帮困境女童凑出下学期的学费

---

文件结构包含两部分：

1. 元数据 (YAML Frontmatter)

举例：

```
name: deep-research-task-managerdescription: 当你需要执行一个复杂的、需要系统性分解和管理多个子任务的深度研究时，使用此 Agent。tools: Read, Write, Bash, Grep, Glob  # 声明该 Agent 可以使用的工具model: sonnet  # 指定使用的模型
```

2. 核心逻辑 (Markdown Body)

用自然语言详细描述 Agent 的工作逻辑和行为准则，定义其工作流程。

举例：

```
你是一位深度研究任务经理，一位专家级的智能体，旨在通过结构化分解和管理子任务，系统地执行复杂的研究任务。你的核心职责是将广泛的研究目标转化为可操作、可追踪的组成部分，并确保全面执行和整合。
```

使用时，在 Claude Code 内，使用 @ 选择 Agent，附加具体任务描述即可。在不使用 @ 时，根据对话上下文仍有概率匹配到合适的 Agent。

06. 案例：深度调研（Demo示例）

解决 Context 长度限制的挑战

LLM 的一个核心挑战是 Context 的长度限制。我通过一套目录约定，为每一次任务执行创建了一个临时的"工作空间"。

1. tasks/active/{task-id}/task.md: 存储最原始的任务需求，确保 Agent 在任何时候都能"回忆"起它的最终目标。
2. tmp/{task-id}/: 这是 Agent 的主要工作目录。所有的临时文件、子任务、中间结果都存放在这里。
3. todo-{subtask-id}.md: Agent 会把一个大任务分解成多个小任务，每个子任务都是一个 todo 文件。
4. done-{subtask-id}.md: 当一个子任务完成后，对应的文件会从 todo 变为 done，并把执行结果记录在文件里。
5. updates/{date-folder}/{task-id}.md: 当所有的子任务都完成后，Agent 会进入"结果合成"阶段，读取所有 done 文件，形成一份最终的、完整的报告，存放在这里。

```
```plaintextdaily-updates/├── .claude/agents/        # Agent定义文件├── tasks/active/          # 活跃任务├── tmp/                   # 临时工作空间│   └── {task-id}/│       ├── todo-001.md    # 待执行子任务│       ├── done-001.md    # 已完成子任务│       └── progress.log   # 进度跟踪└── updates/               # 最终结果    └── {date-folder}/        └── {task-id}.md   # 综合报告```
```

定义工作流程（原文有开源链接）

1. 任务初始化：创建工作目录 `tmp/{task-id}/`
2. 任务分解：生成多个 `todo-\*.md` 文件
3. 子任务执行：循环处理所有 todo 文件，转换为 done 文件
4. 结果综合：合并所有 done 文件内容，生成最终报告

关键参数配置

1. `--permission-mode bypassPermissions`：赋予 Agent "无监督运行"的能力
2. `--model glm-4.6`：指定强大的模型作为 Agent 的"大脑"
3. `BASH\_DEFAULT\_TIMEOUT\_MS`：延长 Shell 命令的默认超时时间

自动化扩展

1. 使用场景

* 夜间研究：每天晚上自动创建研究任务
* 白天阅读：白天自动生成阅读报告
* 结果推送：通过邮件或消息推送结果

2. 定时任务执行

```
```bash# 每天晚上10点执行研究任务0 22 * * * /path/to/agent-runner.sh```
```

3. 批量任务处理

```
```bash# 批量执行多个研究任务for task in task1 task2 task3; do    claude-agent --task $task --model glm-4.6done```
```

07. Simple Deep Research 运行成果

1. 任务摘要

* 输入：复杂的研究任务，包含多个约束条件
* 输出：600+ 行的详细研究报告
* 执行时间：单次执行完成
* 结果质量：超过预期目标（原定20个产品，实际发现30+个）
* 数据完整性：100% 满足所有约束条件

2. 产品分类发现

系统成功识别并分类了8个主要类别的产品：

1. 核心AI助手 (3个)：Gemini CLI, Claude Code, Codex CLI

2. 会话管理器 (3个)：ccmanager, hydra, agent-sessions

3. 配置工具 (4个)：rulebook-ai, rulesify, vibe-mcp, contexthub

4. GUI包装器 (4个)：Gemini-CLI-UI, codexia, prompt-line, aider-desk

5. MCP服务器 (5个)：gemini-bridge, codex-bridge, mcpdog, vibe-mcp, codex-mcp-server

6. 专业工具 (6个)：deepseek-cli, Apple-AI-CLI, Xandai-CLI, oclai, shai, Rust-Coder-CLI

7. 工作流工具 (5个)：zcf, cc-sdd, gptree, yarepl.nvim, jrdev

8. 教育资源 (2个)：awesome-ai-coding-techniques, awesome-ai-coding

08. 总结：Sub-Agent 的未来价值

Claude Code 的 Sub-Agent 系统展示了 AI 协作的巨大潜力。通过合理的架构设计、精确的权限控制和基于文件系统的状态管理，我们可以构建出处理复杂任务的智能协作系统。

这种模式不仅解决了单一 AI 的局限性，更为我们实现更高级别的自动化研究、数据分析和内容生成任务提供了清晰的路径。随着 AI 技术的不断发展，Sub-Agent 协作模式必将成为构建复杂 AI 系统的重要范式。