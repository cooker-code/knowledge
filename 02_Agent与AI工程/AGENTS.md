# Agent 与 AI 工程
## 知识点入口

- 本模块先看宏观流程，再看文章：[知识地图](0200_核心知识点/知识地图.md)。
- 新文章必须先归入流程节点，再判断是补充、冲突、不同层次还是降权。
- 各二级/三级节点的 `文章/` 只保留原文锚点，长期知识必须沉淀到 `0200_核心知识点/`。


> 本表按 2026-06-17 重路由后的二级目录维护；初始化阶段不联网补证。

| 二级目录 | 当前文章数 | 分类边界 | 后续处理 |
|---|---:|---|---|
| [Agent 框架](0201_Agent框架/AGENTS.md) | 147 | LangGraph、DeerFlow、Vercel AI / Agent 工程栈、Pi runtime、运行图、Agent 模式、具体 Agent 系统 | 已把 Harness、Hermes、Memory、Prompt 等横切主题迁出；Vercel 普通部署和 Pi 普通 CLI 使用体验不归这里 |
| [工具调用](0202_工具调用/AGENTS.md) | 206 | MCP、Skill、Tool Calling、Computer Use、工具协议与权限 | MCP 文章不因 `Context Protocol` 名称进入 Context Engineering |
| [RAG 与知识库](0203_RAG与知识库/AGENTS.md) | 91 | 检索、索引、重排、知识库构建 | 记忆生命周期类文章迁入 Memory Management |
| [评估与观测](0204_评估与观测/AGENTS.md) | 19 | Trace、评测、坏例、观测指标 | Harness 方法论文章迁出 |
| [AI 编程工具](0205_AI编程工具/AGENTS.md) | 456 | Claude Code、Cursor、OpenCode、Hermes 等具体 AI Coding 工具 | 方法论文章按 Context/Harness/Prompt/Memory 分流 |
| [工作流编排](0206_工作流编排/AGENTS.md) | 27 | n8n、Dify、Coze、低代码工作流 | 不承接 Agent Harness 方法论 |
| [Prompt Engineering](<0207_Prompt Engineering/AGENTS.md>) | 87 | Agent 指令接口、System/User Prompt、示例反例、输出契约、Prompt 治理 | 纯模型/图像提示词仍留在 LLM 与大模型；推理范式已提升到 COT&TOT&REACT，Prompt 侧只保留互链 |
| [Context Engineering](<0208_Context Engineering/AGENTS.md>) | 42 | 上下文发现、选择、切分、压缩、缓存、注入、验证 | 不承接纯 MCP 协议文章 |
| [Harness Engineering](<0209_Harness Engineering/AGENTS.md>) | 42 | Agent 运行时、状态机、环境、门禁、评估、回滚 | 与 Workflow、Prompt、具体工具保持边界 |
| [安全与权限](0210_安全与权限/AGENTS.md) | 27 | 沙箱、权限、审计、Agent 安全模型 | Prompt 注入可与 Prompt Engineering 互链 |
| [Memory Management](<0211_Memory Management/AGENTS.md>) | 32 | Agent 记忆类型、写入、存储、检索、压缩、污染、遗忘、评估 | 普通 RAG 建库不直接归入 |
| [COT&TOT&REACT](<0212_COT&TOT&REACT/AGENTS.md>) | 12 | CoT、ToT、ReAct、Reflection、LATS、推理路径控制、外部反馈闭环 | 与 Prompt Engineering、工具调用、Harness Engineering 保持边界；显式 ReAct 文本格式不等于生产架构 |
