# AI 编程工具

> 分类规则、一二级类目定位、跨域归类边界以根 [目录划分.md](../../目录划分.md) 为准。
> 文章理解的四步法、整理流程以根 [AGENTS.md](../../AGENTS.md) 为准。
> 本文件只维护本领域的认知重点、排重指纹、已覆盖三级节点和待补缺口。

## 三级节点入口

| 节点 | 用途 |
|---|---|
| [020502_Claude Code](<020502_Claude Code/AGENTS.md>) | 动态搜索、上下文压缩、Hooks、权限、Skill 治理 |
| [020503_Cursor](020503_Cursor/AGENTS.md) | 动态上下文发现、Rules、Plan 工作流 |
| [020504_Hermes](020504_Hermes/AGENTS.md) | CLI/Web/Desktop、Profile、Kanban、多 Agent 协作 |
| 020505_OpenCode（候选，未建目录） | 终端 Agent、oh-my-opencode 编排、Skill、LSP |
| [020506_workBuddy](020506_workBuddy/AGENTS.md) | 自动化工作流、浏览器/微信控制、长记忆 |
| [020507_Codex](020507_Codex/AGENTS.md) | OpenAI Codex CLI 与沙箱机制 |

## 用户认知重点

| 项 | 内容 |
|---|---|
| 已知基础 | 用户知道 AI 编程工具能读写代码、执行命令、生成项目规则 |
| 待补边界 | Skill、MCP、Hook、Subagent、上下文压缩、动态搜索的边界 |
| 易偏差点 | 工具文章易把效率体验、编码方式和产品资讯混在一起 |
| 优先抽取 | 工具本体能力、权限风险、上下文机制、可回滚性；跨工具 Workflow/VibeCode/SpecCode 转入 `0206_AI编码方式` |

## 排重指纹

```text
工具本体 + 工程环节 + 上下文机制/工具机制 + 解决问题 + 权限或验证边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是 Claude Code 技巧 | 只是命令清单，不新建，只保留到路由表 |
| 都是上下文管理 | 区分动态搜索、压缩、规则文件、会话记忆、RAG，不同机制可分开 |
| 都是 Skill/MCP | 区分能力封装、外部工具接入、工作流约束、评估治理 |
| 只有使用体验 | 没有机制和边界时降为略读或跳过 |
| 有权限、安全、可回滚讨论 | 优先沉淀为认知校准点 |
| 主问题是编码方式 | Workflow、VibeCode、SpecCode、需求到交付闭环转入 `../0206_AI编码方式/AGENTS.md` |

## 已覆盖问题（按三级节点）

| 节点 | 已覆盖 | 还缺 |
|---|---|---|
| Claude Code | 动态代码搜索、上下文压缩、Hooks、权限、Skill 治理 | 官方机制和版本状态需后续补证 |
| Cursor | 动态上下文发现、Rules、Plan 工作流、IDE Agent 定位 | 与 Claude Code、OpenCode、OpenSpec 的实测边界 |
| OpenCode | 终端 Agent、oh-my-opencode 编排、Skill、LSP/AST 工具 | 官方文档、权限模型、版本兼容性 |
| Hermes | CLI/Web UI/Desktop、Profile/SOUL、Kanban、多 Agent、Skill、记忆、Webhook、Workspace | 官方版本、权限模型、本地最小实验 |
| workBuddy | 基础设置、自动化工作流、浏览器/微信控制、长记忆、编程协作 | 官方配置、权限、失败恢复、同任务对照证据 |

## 待补缺口

| 优先级 | 项 | 为什么补 |
|---|---|---|
| 高 | AI 编程评估 | 防止只看主观效率 |
| 高 | Context Engineering 边界 | 文章常把上下文能力和具体工具混写 |
| 高 | 工具权限与 Hook 治理 | 防止把自动化能力误当成无风险提效 |
| 高 | AI 编码方式分流 | Workflow、VibeCode、SpecCode 不再放在工具目录，避免把方法论和工具本体混写 |
| 高 | Hermes / workBuddy 官方补证 | 当前文章多为体验，需校验配置、权限、记忆边界 |
| 高 | 官方补证与本地实验 | 官网/GitHub、版本状态、实践结论需补证 |
