> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Directory：给 Claude Code 找到最佳配置
author: 超级AI技术
date: 赚翻啦赚翻啦
url: https://mp.weixin.qq.com/s?__biz=MzUyNzA1NDY0MQ==&mid=2247485648&idx=1&sn=bbbb2579169a428871c87d04e912d453&chksm=fbe9a8c81c4c6e7169f17d595d6fe2a6bbb1e1207a097ceb731207721619ec389047212829cb&mpshare=1&scene=24&srcid=05099PEUjurE3hbAKB1Mq6Dc&sharer_shareinfo=7950af00d772aef3f4b3a1715460b162&sharer_shareinfo_first=7950af00d772aef3f4b3a1715460b162#rd
---

> 刚上手 Claude Code，面对空白的 CLAUDE.md 不知从何写起？

社区里已经有 100+ 配置模板，覆盖 TypeScript、Python、Go 等各种技术栈。Claude Directory (https://www.claudedirectory.org/) 就是收集这些配置的地方。

| 类别 | 数量 | 说明 |
| --- | --- | --- |
| Prompts | 24+ | CLAUDE.md 模板，按技术栈分类 |
| Skills | 37+ | 自定义斜杠命令，一键工作流 |
| Hooks | 22+ | 自动化脚本，工具调用前后触发 |
| MCP Servers | 若干 | 外部服务集成 |
| Plugins | 若干 | 功能扩展包 |
| Agents | 若干 | 专用子 Agent |

不用从零开始，找到适合的模板复制到项目即可。

---

## 一、为什么需要配置目录

Claude Code 的强大在于可配置。但配置什么、怎么配置，很多人不知道。

典型困境：

• **不知道 CLAUDE.md 写什么** — 空白文档无从下手

• **不知道 Skills 怎么写** — 斜杠命令听起来有用，但不知如何设计

• **不知道 Hooks 能做什么** — 自动化听起来好，但不知能自动化什么

• **配置被忽略** — 写了规则，Claude 还是我行我素

Claude Directory 解决了前三个问题。第四个问题，需要理解配置的边界。

---

## 二、Prompts：CLAUDE.md 模板库

### 什么是 Prompts

Prompts 就是 CLAUDE.md 模板。按技术栈分类，每个都是即用配置。

### 精选示例

**Bun + Hono API**

CLAUDE.md for high-performance TypeScript APIs

built with Bun and Hono, including testing,

middleware, and deployment conventions

**Next.js Development**

Comprehensive CLAUDE.md for Next.js App Router

projects with TypeScript and best practices

**Python Development**

CLAUDE.md for Python projects with modern

tooling and best practices

**TypeScript Best Practices**

CLAUDE.md focused on TypeScript patterns

and strict type safety

### 按技术栈浏览

支持的语言和框架：

TypeScript、Python、React、Go、Rust、Next.js

Django、FastAPI、Rails、Vue、Svelte、Flutter

Android、iOS、Kotlin、Swift、Elixir、Phoenix

### 使用方式

1. 浏览 https://www.claudedirectory.org/prompts

2. 找到匹配你技术栈的模板

3. 复制内容到项目根目录的 `CLAUDE.md`

4. 根据项目需求微调

---

## 三、Skills：一键工作流

### 什么是 Skills

Skills 是自定义斜杠命令，把多步操作变成一个命令。

### 精选示例

**SQL Query Optimizer**

Analyze slow SQL queries, explain the execution

plan, and suggest indexes or rewrites

**Skyvern Browser Automation**

AI-powered browser automation — navigate sites,

fill forms, extract structured data, log in with

stored credentials, and build reusable workflows

### 更多 Skills

| Skill | 功能 |
| --- | --- |
| `/code-review` | 代码审查工作流 |
| `/commit` | 规范化提交 |
| `/refactor` | 重构建议 |
| `/test` | 测试生成 |
| `/docs` | 文档生成 |

### 使用方式

1. 浏览 https://www.claudedirectory.org/skills

2. 复制 skill 文件到 `.claude/commands/`

3. 在 Claude Code 中输入 `/skill-name`

### 社区声音

HN 讨论中有人分享：

> 任何我作为一次性操作但知道以后还会做的事情，结束时让 Claude 写一个 skill。比如在 Azure 上查日志频率，现在只需运行 skill 就能获取数据。

---

## 四、Hooks：自动化守卫

### 什么是 Hooks

Hooks 是在 Claude 使用工具前后自动运行的脚本。

### 精选示例

**Dependency Vulnerability Check**

PostToolUse - 当 package.json 变化时

运行 npm audit / pip-audit / cargo audit

阻止引入已知漏洞

### 触发时机

| 事件 | 说明 |
| --- | --- |
| PreToolUse | 工具调用前 |
| PostToolUse | 工具调用后 |
| Notification | 通知事件 |
| Stop | 任务完成时 |

### 更多 Hooks 示例

**自动格式化**：

PostToolUse - 文件写入后

运行 prettier --write

保持代码风格一致

**危险命令拦截**：

PreToolUse - Bash 命令前

检测 rm -rf、DROP 等

阻止危险操作

### 使用方式

1. 浏览 https://www.claudedirectory.org/hooks

2. 复制 hook 配置到 `.claude/settings.json`

3. 根据需要调整匹配规则

---

## 五、Agents：专用子 Agent

### 什么是 Agents

Agents 是专门化的子 Agent，处理特定领域的任务。

### 精选示例

**Observability Engineer**

Instrumentation specialist who adds meaningful

metrics, logs, and traces — and designs dashboards

and alerts that surface the right signal at the

right time

### 使用场景

• 代码审查（安全、性能、风格并行）

• 多模块开发（不同 Agent 负责不同模块）

• 复杂研究（需要多轮讨论和观点碰撞）

---

## 六、按场景浏览

Claude Directory 支持按使用场景浏览：

| 场景 | 配置数量 |
| --- | --- |
| Code Review | 22 |
| Testing | 17 |
| Security | 16 |
| Git Workflows | 20 |
| Documentation | 16 |
| Debugging | 9 |
| Performance | 13 |
| Deployment & CI/CD | 17 |
| Databases | 26 |
| API Development | 18 |

---

## 七、社区的真实反馈

### 正面声音

> ".claude 已成为新的 dotfiles。人们复制别人的 dotfiles，这里也一样。"
>
> "我们目前对使用 Claude 持开放态度，但未知仍是未知。`.claude` 文件夹提供的护栏让我们在熟悉工具时更有信心。"
>
> "我组织内部已部署这些配置。我用一套小型 prompt 测试来衡量成功率，随时间监控，教育团队。"

### 批评声音

**配置可能被忽略**：

> "CLAUDE.md 只是 prompt 文本。压缩（compaction）会重写 prompt 文本。如果重要，用其他方式强制执行。"
>
> "Claude 经常忽略 CLAUDE.md 文件。这些文件的权重似乎不够高。"

**过度配置的风险**：

> "你应该从空的 `.claude`、空的 `AGENTS.md`、零 skills 开始，先学会操作这个工具。"
>
> "不要安装任何你自己没有创建的 skill。每个人的工作流不同，没人知道哪个是对的。把工具箱变成随机 skill 的垃圾抽屉，只会增加另一层不确定性。"

**Vercel 团队的发现**：

> "在 56% 的评估案例中，skill 从未被调用。Agent 可以访问文档但没有使用。"

### 中立观察

> "这些自定义东西往往是短命的补丁。随着默认工具和模型改进，可能不再需要。"
>
> "我 2 个月前用 Claude 构建了一个高级 Python CLI 脚本来搜索日志。但今天 Claude Code 已经内置了这个功能。"

---

## 八、实用建议

### 从小开始

第一周：只用 CLAUDE.md

第二周：尝试一个 skill

第三周：添加一个 hook

第四周：评估效果，决定是否继续

### 配置原则

1. **写真正重要的规则** — 不把 linter 能做的事写进 CLAUDE.md

2. **验证配置生效** — 写测试用例确认 Claude 遵循规则

3. **定期审查** — 删除不再需要的配置

4. **团队共享** — 把生效的配置提交到 Git

### 安全提醒

HN 讨论中有人警告：

> "有大量恶意 skills。非技术用户风险最大。不懂得识别恶意行为不代表不应该担心。"

建议：
- 只使用来源可信的配置
- 审查 skill 内容再安装
- 使用沙箱隔离（`/sandbox` 命令）

---

## 九、Claude Directory vs 官方目录

Claude Code 有官方目录 https://claude.ai/directory，Claude Directory 是社区替代品。

| 维度 | 官方目录 | Claude Directory |
| --- | --- | --- |
| 维护者 | Anthropic | 社区 |
| 配置数量 | 较少 | 100+ |
| 更新频率 | 官方节奏 | 社区贡献 |
| 审核机制 | 官方审核 | 社区审核 |

两个目录可以互补使用。

---

## 十、贡献你的配置

如果你有好的配置，可以贡献到 Claude Directory：

1. 访问 https://www.claudedirectory.org/

2. 点击 "Contribute"

3. 提交你的 CLAUDE.md、skill 或 hook

---

## 参考

• Claude Directory — https://www.claudedirectory.org/

• HN 讨论：Anatomy of the .claude/ folder — https://news.ycombinator.com/item?id=47543139

• Claude Code 官方文档 — https://code.claude.com/docs