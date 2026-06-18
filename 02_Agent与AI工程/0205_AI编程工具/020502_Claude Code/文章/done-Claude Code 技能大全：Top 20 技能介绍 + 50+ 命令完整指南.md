> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 技能大全：Top 20 技能介绍 + 50+ 命令完整指南
author: 运维也AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzY0MDI2OTczNA==&mid=2247484470&idx=1&sn=3986c71ea43ad891c9cc09cb5d0b47bc&chksm=f1966fc7b5ded53f58a527333e631fde8b5e8bb5f027d05a6a09f4535c791578471c1bf1260d&mpshare=1&scene=24&srcid=0414X6lwNj5CBpWb20bgykrD&sharer_shareinfo=976e7fa21f9f991d64a462cf2343b8e6&sharer_shareinfo_first=976e7fa21f9f991d64a462cf2343b8e6#rd
---

Claude Code 技能大全：Top 20 技能介绍 + 50+ 命令完整指南

**摘要**：本文整合 skills.sh 热门技能排行榜和 Claude Code 完整命令体系，涵盖技能安装、使用方法、50+ 命令详解、生产工作流和隐藏功能。看完这篇，Claude Code 对你来说将不再有秘密。

目录

1Agent Skills 生态系统简介

2Top 20 热门技能介绍

3在 Claude Code 中安装和使用技能

4Claude Code 命令体系完整解析

5生产工作流实战

6隐藏功能和进阶技巧

1. Agent Skills 生态系统简介

什么是 Agent Skills？

**Agent Skills** 是 AI 智能体的可复用能力模块。就像给你的手机安装 App 一样，你可以为 AI 智能体安装各种技能，让它获得新的能力。

text

```
███████╗██╗ ██╗██╗██╗ ██╗ ███████╗
██╔════╝██║ ██╔╝██║██║ ██║ ██╔════╝
███████╗█████╔╝ ██║██║ ██║ ███████╗
╚════██║██╔═██╗ ██║██║ ██║ ╚════██║
███████║██║ ██╗██║███████╗███████║
╚══════╝╚═╝ ╚═╝╚═╝╚══════╝╚══════╝

The Open Agent Skills Ecosystem
```

**核心优势：**

✅📦 **即装即用** - 一条命令安装新能力

✅🔄 **可复用** - 一次编写，多次使用

✅🌐 **开放生态** - 任何人都可以创建和分享技能

✅🔌 **跨平台** - 兼容多种 AI 智能体框架

快速开始

bash

```
# 安装技能
npx skills add <owner/repo>

# 示例
npx skills add microsoft/github-copilot-for-azure
```

2. Top 20 热门技能介绍

根据 skills.sh 官方排行榜，以下是目前最受欢迎的 Agent Skills：

🔥 第一梯队（百万级使用量）

| 排名 | 技能名称 | 使用量 | 提供者 | 功能简介 |
| --- | --- | --- | --- | --- |
| 1 | **GitHub Copilot for Azure** | 2.4M | Microsoft | Azure 云服务的 AI 辅助开发，包含 16+ 子技能 |
| 2 | **Azure Skills** | 1.9M | Microsoft | Azure 资源管理、部署、监控全套技能 |
| 3 | **Core Skills** | 759.8K | inferen-sh | 基础通用技能集合，包含 6+ 核心功能 |
| 4 | **Advanced Skills** | 431.1K | inferen-sh | 高级开发技能，包含 3+ 专业功能 |

📈 第二梯队（十万级使用量）

| 排名 | 技能名称 | 使用量 | 提供者 | 功能简介 |
| --- | --- | --- | --- | --- |
| 5 | **Impeccable Code** | 228.7K | pbakaus | 代码质量检查和优化，包含 7+ 审查规则 |
| 6 | **Marketing Skills** | 123.2K | coreyhaines31 | 营销文案、SEO、内容创作技能 |
| 7 | **Code Review** | 112.0K | pbakaus | 自动化代码审查，包含 3+ 审查模式 |
| 8 | **Marketing Copy** | 91.7K | coreyhaines31 | 广告文案、落地页优化技能 |

🛠️ 第三梯队（常用技能）

| 排名 | 技能名称 | 类型 | 功能简介 |
| --- | --- | --- | --- |
| 9 | **Git Workflow** | 开发 | Git 分支管理、合并、冲突解决 |
| 10 | **Database Design** | 开发 | 数据库建模、SQL 优化、迁移脚本 |
| 11 | **API Design** | 开发 | RESTful API 设计、OpenAPI 规范 |
| 12 | **Test Generator** | 测试 | 单元测试、集成测试自动生成 |
| 13 | **Documentation** | 文档 | README、API 文档、注释生成 |
| 14 | **Security Audit** | 安全 | 代码安全扫描、漏洞检测 |
| 15 | **Performance** | 优化 | 性能分析、瓶颈定位、优化建议 |
| 16 | **Docker** | DevOps | Dockerfile 编写、容器编排 |
| 17 | **Kubernetes** | DevOps | K8s 配置、部署、故障排查 |
| 18 | **CI/CD** | DevOps | GitHub Actions、Jenkins 流水线 |
| 19 | **CloudFormation** | 云原生 | AWS 基础设施即代码 |
| 20 | **Terraform** | 云原生 | 多云基础设施编排 |

3. 在 Claude Code 中安装和使用技能

3.1 安装技能

**方法一：使用 npx 命令**

bash

```
# 安装单个技能
npx skills add microsoft/github-copilot-for-azure

# 安装多个技能
npx skills add microsoft/github-copilot-for-azure microsoft/azure-skills

# 从本地目录安装
npx skills add ./my-custom-skill
```

**方法二：在 Claude Code 会话中安装**

bash

```
# 启动 Claude Code
claude

# 在会话中执行安装命令
/skills install microsoft/github-copilot-for-azure
```

3.2 技能目录结构

安装后的技能位于：

text

```
~/.claude/skills/
├── microsoft/
│   ├── github-copilot-for-azure/
│   │   ├── SKILL.md          # 技能说明
│   │   ├── scripts/          # 执行脚本
│   │   └── references/       # 参考文档
│   └── azure-skills/
├── inferen-sh/
│   └── skills/
└── pbakaus/
    └── impeccable/
```

3.3 使用技能

**在 Claude Code 中调用技能：**

bash

```
# 查看已安装的技能列表
/skills list

# 使用特定技能
@github-copilot-for-azure "帮我创建一个 Azure Function"

# 查看技能详情
/skills info microsoft/github-copilot-for-azure
```

**技能调用示例：**

bash

```
# 使用 Azure 技能创建资源
@azure-skills "创建一个 Azure Web App，使用 Node.js 运行时"

# 使用代码审查技能
@impeccable "审查这个文件的安全漏洞"

# 使用营销技能
@marketing-skills "为这个产品写一个落地页文案"
```

3.4 技能配置

**创建配置文件 `~/.claude/skills-config.json`：**

json

```
{
  "defaultSkills": [
    "microsoft/github-copilot-for-azure",
    "pbakaus/impeccable"
  ],
  "autoInvoke": {
    "test-generator": ["*.test.ts", "*.spec.ts"],
    "documentation": ["README.md", "*.md"]
  },
  "aliases": {
    "azure": "microsoft/azure-skills",
    "review": "pbakaus/impeccable"
  }
}
```

4. Claude Code 命令体系完整解析

Claude Code 内置了**超过 50 个命令**，但大多数开发者只用了其中 3-5 个。下面按 7 大模块详细解析。

4.1 三种命令类型

| 类型 | 触发方式 | 示例 |
| --- | --- | --- |
| **CLI 命令** | 终端启动时执行 | `claude -c` |
| **斜杠命令** | 会话内输入 `/` 触发 | `/init` |
| **键盘快捷键** | 直接按键生效 | `Ctrl+C` |

4.2 日常核心命令（核心 10 个）

1️⃣ `/init` — 项目初始化

bash

```
/init
```

**作用：** 在项目根目录创建 `CLAUDE.md` —— Claude 每次会话都会读取的持久记忆文件。

**生成的 CLAUDE.md 示例：**

markdown

```
# Project: E-Commerce API

## Tech Stack
- Node.js + Express
- PostgreSQL via Prisma
- JWT authentication

## Rules
- Use async/await, never callbacks
- Write tests for all endpoints
- Return structured errors: { error, code }

## Patterns
- All database queries in /services
- Route handlers in /routes
- Middleware in /middleware
```

**💡 最佳实践：**`/init` 后立即追加具体规则，消除 80% 的重复上下文设置。

2️⃣ `/compact` — 上下文压缩

bash

```
# 基础压缩
/compact

# 定向压缩（保留特定上下文）
/compact retain the error handling patterns and auth module changes
```

**使用时机：**

✅会话超过 30 分钟

✅出现"上下文过大"警告

✅Claude 开始遗忘先前决策

**💡 建议：** 上下文用到 70-80% 时主动压缩，不要等到 95%。

3️⃣ `/clear` — 硬重置

bash

```
/clear
```

**作用：** 完全清除对话历史，从零开始。

**`/compact` vs `/clear`：**

| 命令 | 效果 | 使用场景 |
| --- | --- | --- |
| `/compact` | 摘要并保留上下文 | 继续同一任务，上下文较重 |
| `/clear` | 硬重置，全新开始 | 切换到不同任务 |

4️⃣ `/model` — 切换模型

bash

```
/model              # 交互式选择器
/model sonnet       # 切换到 Sonnet 4.6
/model opus         # 切换到 Opus 4.6
/model haiku        # 切换到 Haiku 4.5
```

**模型选择策略：**

| 模型 | 适用场景 | 成本 |
| --- | --- | --- |
| **Sonnet 4.6** | 日常编码、重构、Bug 修复 | 中等 |
| **Opus 4.6** | 复杂规划、架构决策、关键代码 | 高 |
| **Haiku 4.5** | 简单编辑、样板代码、快速提问 | 低 |

**💡 日常策略：** Sonnet 起步，遇到瓶颈切 Opus，琐碎任务交给 Haiku。

5️⃣ `/cost` — Token 用量

bash

```
/cost
```

**输出示例：**

text

```
Session cost: $2.47
Input tokens: 48,392
Output tokens: 12,847
Model: claude-sonnet-4-20250514
```

**💡 建议：** 每次大交互后跑一次 `/cost`，活跃开发中一个会话费用在 $5-$50 之间。

6️⃣ `/context` — 上下文窗口用量

bash

```
/context
```

**输出示例：**

text

```
Context usage: 67% (134,400 / 200,000 tokens)
```

7️⃣ `/diff` — 查看最近更改

bash

```
/diff                 # 显示所有更改
/diff src/auth.ts     # 显示特定文件更改
```

**使用场景：**

✅提交代码前审查

✅审查 Claude 到底改了什么

✅排查意外修改

8️⃣ `/help` — 命令列表

bash

```
/help
```

显示所有可用的斜杠命令，是当前版本的权威来源。

9️⃣ `/memory` — 编辑 CLAUDE.md

bash

```
/memory
```

**快速记忆语法：**

bash

```
# 无需打开编辑器即可添加到记忆
# Use async/await for all database queries
```

前缀 `#` 的内容会直接追加到 `CLAUDE.md`。

🔟 `/resume` — 继续过去的会话

bash

```
# 恢复最近的会话
claude --resume

# 按名称恢复特定会话
claude --resume auth-refactor

# 从会话列表中选择
/resume
```

4.3 进阶命令

11️⃣ `/btw` — 不打断上下文的提问

bash

```
# Claude 正在重构一个大模块
# 你突然需要查看某些内容

/btw What is the difference between useEffect and useLayoutEffect?
# Claude 回答后继续重构
```

**💡 价值：** 改变操作习惯的命令，不打断上下文就能提问。

12️⃣ `/fast` — 极速模式

bash

```
/fast    # 切换开启/关闭
```

**说明：** Fast Mode 运行的是同一个 Opus 4.6，但调整了 API 配置以优化速度。

**适用场景：** 交互式快速迭代、实时调试、快速实验

**⚠️ 注意：** 开启后，之前积累的全部上下文会按 Fast Mode 费率重新计费。

13️⃣ `/plan` — 计划模式（只读）

bash

```
# 切换计划模式
Shift+Tab    # 循环切换模式

# 或显式切换
/plan
```

**三种模式对比：**

| 模式 | 行为 | 使用场景 |
| --- | --- | --- |
| **Normal** | 每次工具执行前要求确认 | 默认安全模式 |
| **Auto-Accept** | 无需确认直接执行 | 写测试、生成样板代码 |
| **Plan** | 只展示方案等待审批 | 配置、数据库迁移、package.json |

14️⃣ `/fork` — 实验性分支

bash

```
/fork

# 尝试实验性重构
# 效果不好？
# 关闭分支，返回主对话
```

**适用场景：** 测试高风险重构、探索多种方案、快速实验

15️⃣ `/rewind` — 撤销对话或代码

bash

```
Esc Esc    # 打开回退菜单

# 选项:
# - Rewind conversation only (keep code)
# - Rewind code only (keep conversation)
# - Rewind both
```

**典型用法：**

bash

```
# 尝试实验性重构
# → 效果不好
# → Esc Esc
# → "Rewind code only"
# → 代码恢复，对话历史保留
```

16️⃣ `/todos` — 持久化任务列表

bash

```
# 切换任务列表显示
Ctrl+T

# 使用自然语言创建任务
"Add authentication feature. Break it down into tasks by dependency"
```

**💡 特性：** 关闭会话后任务不会消失，上下文压缩也不会影响它。

17️⃣ `/simplify` — 代码审查（替代 `/review`）

bash

```
/simplify
```

**功能：** 用三个并行 Agent 执行代码审查，覆盖代码质量、安全漏洞、最佳实践违规、性能问题和测试覆盖率。

**工作流：** 编写功能 → `/simplify` → 审查反馈 → 修复问题 → 提交

18️⃣ `/output-style` — 调整输出风格

bash

```
/output-style

# Options:
# - Concise
# - Educational
# - Code Reviewer
# - Rapid Prototyping
```

**隐藏入口：**

bash

```
@agent-output-mode-setup
```

执行后会在 `~/.claude/output-modes/` 下生成四种自定义模式。

19️⃣ `/permissions` — 管理自动审批

bash

```
/permissions

# Example config:
# Auto-approve: npm install, git status, file reads
# Require approval: git push, file deletions, npm publish
```

20️⃣ `/agents` — 子 Agent 管理

bash

```
/agents

# Create sub-agent
@agent-create test-writer "Writes comprehensive Jest tests"
```

4.4 CLI 标志和启动选项

| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `claude --print` | 一次性查询 | `claude --print "解释 async/await"` |
| `claude -c` | 恢复最近会话 | `claude -c` |
| `--append-system-prompt` | 追加系统指令 | `claude --append-system-prompt "始终使用 TypeScript"` |
| `--system-prompt` | 替换系统指令 | `claude --system-prompt "你是 Python 专家"` |
| `--dangerously-skip-permissions` | 跳过权限确认 | ⚠️ 仅限可信容器 |
| `--agents` | 启动时定义子 Agent | `claude --agents '{...}'` |
| `--output-format json` | 结构化输出 | `claude --print "..." --output-format json` |

4.5 键盘快捷键

核心快捷键

| 快捷键 | 功能 |
| --- | --- |
| `Ctrl+C` | 取消当前生成 |
| `Ctrl+R` | 搜索命令历史 |
| `Tab` | 切换思考显示 |
| `Shift+Tab` | 循环切换模式 |
| `Esc Esc` | 回退菜单 |

导航与编辑

| 快捷键 | 功能 |
| --- | --- |
| `Ctrl+T` | 切换任务列表 |
| `Alt+M` | 切换模式 |
| `Alt+P/N` | 上一条/下一条消息 |
| `Alt+B/F` | 对话后退/前进 |

文件与命令

| 快捷键 | 功能 |
| --- | --- |
| `@ + path` | 文件自动补全 |
| `! + command` | 直接执行 bash |
| `# + text` | 快速记忆添加 |

**示例：**

bash

```
@src/auth.ts          # 文件自动补全
! git status          # 直接执行 bash
# Use JWT for auth    # 快速记忆
```

4.6 隐藏和未公开功能

| 功能 | 命令 | 说明 |
| --- | --- | --- |
| **Vim 键位** | `/vim` | 为提示输入启用 vim 键位绑定 |
| **手机控制** | `/remote-control` | 通过 claude.ai 网页界面控制本地会话 |
| **导出会话** | `/export` | 将对话导出为可搜索文档 |
| **对话克隆** | 组合实现 | 从同一起点探索多种方案 |
| **使用报告** | `/usage-report` | 月度分析报告（HTML 格式） |
| **PR 状态** | 底部栏自动显示 | PR 审查状态实时刷新 |

4.7 配置文件和自定义

**文件位置：**

text

```
~/.claude/                    # 主配置目录
~/.claude/projects/           # 会话历史
~/.claude/commands/           # 自定义斜杠命令（旧版）
~/.claude/skills/             # Agent Skills（2026 标准）
~/.claude/output-modes/       # 自定义输出模式
~/.claude/keybindings.json    # 键盘快捷键
```

**键位绑定自定义 `~/.claude/keybindings.json`：**

json

```
{
  "toggle_thinking": "Tab",
  "cancel": "Ctrl+C",
  "search_history": "Ctrl+R",
  "toggle_task_list": "Ctrl+T"
}
```

5. 生产工作流实战

5.1 功能开发工作流

bash

```
# 1. 启动会话
claude

# 2. 设置上下文
/init                    # 如果是项目首次
/memory                  # 更新当前功能需求

# 3. 添加快速记忆
# Use JWT for auth
# Write tests for all endpoints
# Follow RESTful conventions

# 4. 实现
"Create authentication middleware for Express that validates JWT tokens"

# 5. 审查
/diff

# 6. 运行测试
! npm test

# 7. 检查费用
/cost

# 8. 提交
! git add .
! git commit -m "feat: add JWT auth middleware"
```

5.2 调试工作流

bash

```
# 1. 继续现有会话
claude -c

# 2. 展示错误
"Here's the error I'm getting:"
[paste error]

# 3. Claude 进行调查
# 使用 /btw 进行附带提问而不打断主线

# 4. 应用修复
/diff    # 审查更改

# 5. 测试
! npm test

# 6. 如果有效，压缩后继续
/compact
```

5.3 大规模重构工作流

bash

```
# 1. 以计划模式启动
claude
Shift+Tab    # 启用计划模式

# 2. 描述重构
"Refactor auth module to use bcrypt instead of plain passwords"

# 3. 在执行前审查计划
# 批准或调整

# 4. 监控上下文
/context

# 5. 在 70% 时主动压缩
/compact retain auth patterns and migration strategy

# 6. 切换到自动接受模式处理常规更改
Shift+Tab    # 自动接受模式

# 7. 最终审查
/diff
/simplify    # 代码审查

# 8. 导出为文档
/export
```

5.4 多 Agent 委派工作流

bash

```
# 1. 主对话：架构设计
"Design the database schema for user authentication"

# 2. 委派给子 Agent
/agents
@agent-create test-writer "Write comprehensive tests"

# 3. 主对话继续
"Now implement the auth middleware"

# 4. 子 Agent 并行工作
@test-writer "Generate tests for auth middleware"

# 5. 合并结果
# 主对话保持清洁和专注
```

6. 隐藏功能和进阶技巧

6.1 Vim 模式深度使用

bash

```
/vim    # 启用 vim 键位
```

**支持的 vim 功能：**

✅模式切换：Normal / Insert

✅导航：`h/j/k/l`、`w/b/e`、`0/$`

✅字符跳转：`f/F/t/T`

✅编辑操作：`d`、`c`、`y`、`p`

✅文本对象：`iw`、`aw`、`i"`、`a()`

6.2 手机远程控制

bash

```
/remote-control
```

**使用场景：** 通勤途中想起有个 bug 需要修，打开手机上的 claude.ai，连接本地会话，直接让 Claude 执行修复。

6.3 会话导出为文档

bash

```
/export
```

**最佳实践：** 解决棘手问题之后执行 `/export`，导出的内容可以作为可搜索的技术文档、学习资料或团队知识库。

6.4 底部栏 PR 状态

在有已打开 PR 的分支上工作时，Claude Code 底部栏会自动显示 PR 状态：

| 颜色 | 状态 |
| --- | --- |
| 🟢 绿色 | 已批准 |
| 🟡 黄色 | 请求更改 |
| 🔴 红色 | 已阻塞 |
| ⚪ 灰色 | 等待审查 |

**刷新频率：** 每 60 秒自动刷新

7. 关键要点总结

🎯 先掌握核心 10 个

`/init`、`/compact`、`/clear`、`/model`、`/cost`、`/context`、`/diff`、`/help`、`/memory`、`/resume`

**光是这些就能带来 3-4 倍的效率增幅。**

💡 改变习惯的命令

**`/btw`** — 不打断上下文就能提问，放心多用。

⏰ 主动压缩上下文

上下文用到 **70-80%** 就该执行 `/compact`，不是等到 95%。

📝 CLAUDE.md 一次配好

每次会话省 10-15 分钟。

🔀 模式切换防止误操作

| 模式 | 使用场景 |
| --- | --- |
| **Auto-accept** | 样板代码和测试 |
| **Plan mode** | 生产关键文件 |
| **Normal** | 其他场景 |

📤 导出问题解决过程

`/export` 把问题解决过程变成可复用的文档。

💰 关注成本

Opus 会话费用可能到 $50，每次大交互后跑一次 `/cost`。

⌨️ 记住四个快捷键

| 快捷键 | 功能 |
| --- | --- |
| `Shift+Tab` | 模式切换 |
| `Ctrl+T` | 任务列表 |
| `Esc Esc` | 回退菜单 |
| `Ctrl+R` | 历史搜索 |

🤖 子 Agent 承担专项工作

主对话保持干净和专注。

🔍 定期探索隐藏功能

`/vim`、`/remote-control`、`/usage-report` 这些隐藏功能确实可用，定期翻翻 `/help` 会有收获。

结语

Claude Code 有 50 多个命令，到这里已经全部覆盖。多数开发者还是会停留在 3-5 个的用法上，但全部的工具箱现在已经摊开了。

**把 Claude Code 当"终端版 ChatGPT"用和把它当可编程编码伙伴用，差别就在对命令体系的熟悉程度。**

**建议学习路径：**

1第一周：掌握核心 10 个命令

2第二周：每天尝试 1 个进阶命令

3第三周：实践生产工作流

4第四周：探索隐藏功能

从核心 10 个开始，每周加一个新命令，把关键会话导出保留。三个月后，你会发现自己已经离不开这套强大的工具链了。

*作者：小胡 🐾**创建时间：2026-03-26**参考来源：skills.sh、阿里云开发者社区*