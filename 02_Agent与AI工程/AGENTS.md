# Agent 与 AI 工程

> 一级目录全量蒸馏入口。分类边界以根目录 `目录划分.md` 为准；文章处理以根目录 `AGENTS.md` 和 `knowledge-distiller` 流程为准。

## 二级目录收口状态

| 二级目录 | done 文章 | 裸文章 | 文章目录 | 状态 |
|---|---:|---:|---:|---|
| [0201_Agent框架](0201_Agent框架/AGENTS.md) | 91 | 0 | 13 | 已收口 |
| [0202_工具调用](0202_工具调用/AGENTS.md) | 197 | 0 | 3 | 已收口 |
| [0203_RAG与知识库](0203_RAG与知识库/AGENTS.md) | 96 | 0 | 4 | 已收口 |
| [0204_评估与观测](0204_评估与观测/AGENTS.md) | 20 | 0 | 2 | 已收口 |
| [0205_AI编程工具](0205_AI编程工具/AGENTS.md) | 351 | 0 | 5 | 已收口 |
| [0206_AI编码方式](0206_AI编码方式/AGENTS.md) | 121 | 0 | 3 | 已拆分收口 |
| [0207_Prompt Engineering](0207_Prompt Engineering/AGENTS.md) | 87 | 0 | 1 | 已收口 |
| [0208_Context Engineering](0208_Context Engineering/AGENTS.md) | 42 | 0 | 1 | 已收口 |
| [0209_Harness Engineering](0209_Harness Engineering/AGENTS.md) | 42 | 0 | 1 | 已收口 |
| [0210_sandbox](0210_sandbox/AGENTS.md) | 16 | 0 | 1 | 已重蒸馏 |
| [0211_Memory Management](0211_Memory Management/AGENTS.md) | 32 | 0 | 1 | 已收口 |
| [0212_COT&TOT&REACT](0212_COT&TOT&REACT/AGENTS.md) | 14 | 0 | 1 | 已收口 |

## 一级目录排重准则

- `0201_Agent框架`：框架、运行时、协作模式、Agent 平台和工程方法。
- `0202_工具调用`：MCP、Skill、Tool Calling 等能力接入方式。
- `0203_RAG与知识库`：检索增强、知识库、GraphRAG、LLM Wiki 和 RAGFlow。
- `0204_评估与观测`：AI 应用和 Agent 的质量、Trace、评估集和过程证据。
- `0205_AI编程工具`：Claude Code、Codex、Cursor、Hermes、workBuddy 等具体 AI 编程工具。
- `0206_AI编码方式`：Workflow、VibeCode、SpecCode 等跨工具 AI 编码方式和交付约束。
- `0207_Prompt Engineering`：任务契约、提示模板、角色边界、输出约束和提示词安全。
- `0208_Context Engineering`：上下文发现、裁剪、压缩、缓存、结构化输出和渐进式披露。
- `0209_Harness Engineering`：Agent 运行底盘、规格、环境、工具、状态、回滚和监控。
- `0210_sandbox`：Agent 执行环境隔离、Docker/K8s/微虚拟机/浏览器沙箱、凭证边界、网络策略和审计。
- `0211_Memory Management`：长期记忆、短期记忆、知识图谱、压缩和记忆注入。
- `0212_COT&TOT&REACT`：推理链、规划、反思、行动循环和自我纠错。

## 本轮处理说明

- 本轮将旧 `020501_AI Coding 工作流` 和历史目录名 `0206_工作流编排` 拆分到 `0206_AI编码方式`，并把 Git/worktree 与企业协作 CLI 文章转入 `09_电脑工具`。
- 文章只作为来源锚点；长期知识进入各节点根部的 `*_知识地图.md`、`*_核心知识点/` 主题页或已有主题页。
- 涉及官方版本、性能数字、安全能力的内容，本轮只标待验证，不凭文章直接固化为事实。
- `0210` 已从泛化“安全与权限”收敛为 `0210_sandbox`，只保留 Sandbox 本体文章；Prompt 注入、Skill 审查、供应链投毒和通用权限模型不再作为本目录长期来源。
