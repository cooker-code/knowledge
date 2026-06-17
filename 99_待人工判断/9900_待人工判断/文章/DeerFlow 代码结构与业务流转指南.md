---
title: DeerFlow 代码结构与业务流转指南
author: 万智创界
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4MTc1MjMxOA==&mid=2457540368&idx=1&sn=236abcdc626665f0014cdf600ed194f3&chksm=89777879aece25e0b4614670c916284f1cd4cf8554fceef7cccd73e62498eb358d409f887967&mpshare=1&scene=24&srcid=0421F6ITmrhkyw8D786loFn6&sharer_shareinfo=0f2124254fc7a6abc8fea439e2fd4d3d&sharer_shareinfo_first=0f2124254fc7a6abc8fea439e2fd4d3d#rd
---

# DeerFlow 代码结构与业务流转指南

## 一、全局架构总览

```
deer-flow/  
├── frontend/          ← Next.js 16 前端 (React 19 + TypeScript)  
├── backend/           ← Python 后端 (FastAPI + LangGraph)  
├── skills/            ← Skill 定义文件 (SKILL.md 格式)  
├── docker/            ← Docker Compose 配置  
├── scripts/           ← 开发/部署脚本  
├── config.yaml        ← 主配置文件 (模型/工具/沙箱/记忆等)  
├── Makefile           ← 开发命令入口  
└── nginx (2026)       ← 统一反向代理 (make dev 自动启动)
```

**运行时进程拓扑** (make dev 启动 4 个进程):

```
浏览器 → http://localhost:2026  
            │  
            ▼  
     ┌─────────────── Nginx (Port 2026) ──────────────┐  
     │                                                  │  
     │  /api/langgraph/*  →  LangGraph Server :2024     │  
     │  /api/*            →  Gateway API     :8001      │  
     │  /                  →  Frontend        :3000     │  
     │                                                  │  
     └──────────────────────────────────────────────────┘
```

* **LangGraph Server (2024)**: 运行 Agent 图，处理对话推理，通过 SSE 流式返回结果
* **Gateway API (8001)**: FastAPI 服务，管理非推理类操作（模型列表、Skills、Memory、文件上传等）
* **Frontend (3000)**: Next.js 开发服务器
* **Nginx (2026)**: 统一入口，按路径分发到不同后端

---

## 二、后端代码结构（重点）

```
backend/  
├── app/                          ← HTTP 服务入口  
│   ├── gateway/                  ← FastAPI Gateway API  
│   │   ├── app.py                ← FastAPI 应用创建 + 路由注册  
│   │   ├── deps.py               ← 依赖注入 (LangGraph runtime 初始化)  
│   │   ├── services.py           ← Gateway 业务服务  
│   │   └── routers/              ← 15 个路由模块  
│   │       ├── models.py         ← GET /api/models (模型列表)  
│   │       ├── skills.py         ← GET/PUT /api/skills (Skills 管理)  
│   │       ├── mcp.py            ← GET/PUT /api/mcp/config (MCP 配置)  
│   │       ├── memory.py         ← GET /api/memory (记忆数据)  
│   │       ├── uploads.py        ← POST /api/threads/{id}/uploads (文件上传)  
│   │       ├── artifacts.py      ← GET /api/threads/{id}/artifacts/{path} (产出文件)  
│   │       ├── threads.py        ← DELETE /api/threads/{id} (清理线程)  
│   │       ├── agents.py         ← POST/GET/DELETE /api/agents (自定义 Agent)  
│   │       ├── suggestions.py    ← POST /api/threads/{id}/suggestions (后续建议)  
│   │       ├── runs.py           ← POST /api/runs/stream (无状态运行)  
│   │       ├── thread_runs.py    ← /api/threads/{id}/runs/* (线程内运行)  
│   │       └── channels.py       ← /api/channels (IM 通道状态)  
│   │  
│   └── channels/                 ← IM 通道集成  
│       ├── service.py            ← 通道服务启动/停止  
│       ├── manager.py            ← 通道管理器  
│       ├── message_bus.py        ← 消息总线  
│       ├── feishu.py             ← 飞书通道  
│       ├── slack.py              ← Slack 通道  
│       ├── telegram.py           ← Telegram 通道  
│       ├── wechat.py             ← 微信通道  
│       └── wecom.py              ← 企业微信通道  
│  
├── packages/harness/deerflow/    ← 核心业务逻辑（SDK 包）  
│   │  
│   ├── agents/                   ← 🧠 Agent 系统 (核心)  
│   │   ├── lead_agent/           ← 主 Agent  
│   │   │   ├── agent.py          ← make_lead_agent() — Agent 工厂入口  
│   │   │   └── prompt.py         ← System Prompt 模板  
│   │   ├── factory.py            ← create_deerflow_agent() — SDK 级工厂  
│   │   ├── features.py           ← RuntimeFeatures — 功能特性开关  
│   │   ├── thread_state.py       ← ThreadState — Agent 状态定义  
│   │   ├── middlewares/          ← 14 个中间件 (详见下文)  
│   │   ├── memory/               ← 记忆系统  
│   │   │   ├── storage.py        ← MemoryStore (memory.json 读写)  
│   │   │   ├── updater.py        ← MemoryUpdater (LLM 提取事实)  
│   │   │   ├── queue.py          ← MemoryQueue (debounce 批量更新)  
│   │   │   ├── message_processing.py ← 消息预处理  
│   │   │   └── prompt.py         ← Memory 提取用的 Prompt  
│   │   └── checkpointer/         ← 检查点 (会话持久化)  
│   │  
│   ├── sandbox/                  ← 📦 沙箱系统  
│   │   ├── sandbox.py            ← Sandbox 抽象基类 (execute_command/read_file/write_file/list_dir)  
│   │   ├── sandbox_provider.py   ← SandboxProvider (创建/回收沙箱实例)  
│   │   ├── middleware.py         ← SandboxMiddleware (生命周期管理)  
│   │   ├── tools.py              ← 沙箱工具 (bash_tool/ls_tool/read_file_tool/write_file_tool/str_replace_tool)  
│   │   ├── security.py           ← 安全策略 (沙箱类型判断)  
│   │   ├── file_operation_lock.py ← 文件操作锁 (并发安全)  
│   │   ├── search.py             ← GrepMatch + 搜索工具  
│   │   ├── local/                ← 本地沙箱实现  
│   │   └── exceptions.py         ← 沙箱异常定义  
│   │  
│   ├── subagents/                ← 🤖 子 Agent 系统  
│   │   ├── executor.py           ← SubagentExecutor (后台线程池执行)  
│   │   ├── registry.py           ← AgentRegistry (注册/查找 Agent)  
│   │   ├── config.py             ← 子 Agent 配置  
│   │   └── builtins/             ← 内置子 Agent  
│   │       ├── general-purpose/  ← 通用子 Agent (完整工具集)  
│   │       └── bash/             ← Bash 专家子 Agent  
│   │  
│   ├── tools/                    ← 🔧 工具系统  
│   │   ├── tools.py              ← get_available_tools() (工具收集主函数)  
│   │   ├── builtins/             ← 内置工具  
│   │   │   ├── task_tool.py      ← task() 工具 (委派子 Agent)  
│   │   │   ├── view_image_tool.py ← view_image() 工具  
│   │   │   ├── present_file_tool.py ← present_files() 工具  
│   │   │   ├── clarification_tool.py ← ask_clarification() 工具  
│   │   │   ├── setup_agent_tool.py ← setup_agent() 工具 (自定义 Agent 创建)  
│   │   │   ├── tool_search.py    ← 工具搜索 (延迟加载工具)  
│   │   │   └── invoke_acp_agent_tool.py ← ACP Agent 调用  
│   │   └── skill_manage_tool.py  ← Skill 管理工具  
│   │  
│   ├── skills/                   ← 📚 Skills 系统  
│   │   ├── loader.py             ← Skill 加载器 (递归发现 SKILL.md)  
│   │   ├── parser.py             ← SKILL.md 解析 (YAML frontmatter + body)  
│   │   ├── manager.py            ← SkillManager (启用/禁用/列表)  
│   │   ├── installer.py          ← .skill 归档安装  
│   │   ├── validation.py         ← Skill 验证  
│   │   ├── security_scanner.py   ← Skill 安全扫描  
│   │   └── types.py              ← Skill 数据类型  
│   │  
│   ├── mcp/                      ← 🔌 MCP (Model Context Protocol)  
│   │   ├── client.py             ← MCP 客户端 (stdio/SSE/HTTP 连接)  
│   │   ├── tools.py              ← MCP 工具注册  
│   │   ├── oauth.py              ← MCP OAuth 认证  
│   │   └── cache.py              ← MCP 工具缓存  
│   │  
│   ├── models/                   ← 🤖 模型系统  
│   │   ├── factory.py            ← create_chat_model() (模型工厂)  
│   │   ├── credential_loader.py  ← 凭证加载 (环境变量/文件)  
│   │   ├── patched_openai.py     ← OpenAI 补丁 (Responses API 等)  
│   │   ├── patched_deepseek.py   ← DeepSeek 补丁  
│   │   ├── patched_minimax.py    ← MiniMax 补丁  
│   │   ├── vllm_provider.py      ← vLLM 本地模型支持  
│   │   ├── claude_provider.py    ← Claude Code OAuth 支持  
│   │   └── openai_codex_provider.py ← Codex CLI 支持  
│   │  
│   ├── config/                   ← ⚙️ 配置系统 (23 个配置模块)  
│   │   ├── app_config.py         ← 主配置 (加载 config.yaml)  
│   │   ├── model_config.py       ← 模型配置  
│   │   ├── sandbox_config.py     ← 沙箱配置  
│   │   ├── memory_config.py      ← 记忆配置  
│   │   ├── subagents_config.py   ← 子 Agent 配置  
│   │   ├── skills_config.py      ← Skills 配置  
│   │   ├── tool_config.py        ← 工具配置  
│   │   ├── summarization_config.py ← 摘要配置  
│   │   ├── tracing_config.py     ← 追踪配置 (LangSmith/Langfuse)  
│   │   └── ...                   ← 其他 20+ 配置模块  
│   │  
│   ├── runtime/                  ← 🏃 运行时基础设施  
│   │   ├── runs/                 ← Run 管理 (Gateway Mode)  
│   │   │   ├── manager.py        ← RunManager (创建/取消/查询)  
│   │   │   ├── worker.py         ← RunWorker (异步执行 Agent)  
│   │   │   └── schemas.py        ← Run 数据模型  
│   │   ├── store/                ← 持久化存储  
│   │   │   ├── provider.py       ← Store Provider  
│   │   │   └── async_provider.py ← 异步 Store Provider  
│   │   ├── stream_bridge/        ← 流式桥接  
│   │   │   ├── base.py           ← StreamBridge 基类  
│   │   │   ├── async_provider.py ← 异步 StreamBridge  
│   │   │   └── memory.py         ← 内存 StreamBridge  
│   │   └── serialization.py      ← 序列化工具  
│   │  
│   ├── uploads/                  ← 📤 文件上传  
│   │   └── manager.py            ← UploadManager (PDF/PPT/Word→Markdown 转换)  
│   │  
│   ├── guardrails/               ← 🛡️ 安全护栏  
│   ├── tracing/                  ← 📊 链路追踪  
│   │   └── factory.py            ← 追踪回调工厂 (LangSmith/Langfuse)  
│   ├── reflection/               ← 🪞 动态模块加载  
│   ├── community/                ← 🌍 社区工具 (Tavily/Jina/Firecrawl 等)  
│   ├── client.py                 ← 📡 嵌入式 Python 客户端 (DeerFlowClient)  
│   └── utils/                    ← 🔧 通用工具  
│  
├── langgraph.json                ← LangGraph Server 配置  
│                                  指定入口: deerflow.agents:make_lead_agent  
├── pyproject.toml                ← Python 依赖  
├── tests/                        ← 测试  
└── docs/                         ← 文档
```

---

## 三、前端代码结构

```
frontend/src/  
├── app/                          ← Next.js App Router 页面  
│   ├── [lang]/                   ← 国际化路由 (i18n)  
│   │   └── workspace/  
│   │       └── chats/  
│   │           ├── page.tsx      ← 对话列表页 /chats  
│   │           ├── new/  
│   │           │   └── page.tsx  ← 新建对话 /chats/new  
│   │           └── [thread_id]/  
│   │               └── page.tsx ← 对话详情 /chats/{thread_id} ← 主要工作页  
│   ├── workspace/  
│   │   ├── layout.tsx            ← 工作区布局  
│   │   └── agents/               ← Agent 管理页  
│   ├── api/                      ← API Routes  
│   ├── layout.tsx                ← 根布局  
│   └── page.tsx                  ← 首页 (Landing)  
│  
├── components/                   ← React 组件  
│   ├── workspace/                ← 工作区组件 (对话列表、消息、输入框等)  
│   ├── ai-elements/              ← AI 相关 UI 元素  
│   ├── ui/                       ← Shadcn UI 基础组件  
│   └── landing/                  ← 首页组件  
│  
├── core/                         ← ⭐ 核心业务逻辑（最重要的目录）  
│   ├── api/                      ← API 客户端  
│   │   ├── api-client.ts         ← LangGraphClient 封装 (连接 LangGraph Server)  
│   │   └── stream-mode.ts        ← SSE 流模式处理 (values/messages-tuple/end)  
│   │  
│   ├── threads/                  ← 🧵 线程管理 (核心！)  
│   │   ├── hooks.ts              ← useThreadStream() — 最关键的 Hook  
│   │   │                         管理: 创建线程→发送消息→接收SSE→更新状态  
│   │   ├── types.ts              ← AgentThreadState 类型定义  
│   │   ├── utils.ts              ← 线程工具函数  
│   │   ├── export.ts             ← 对话导出  
│   │   └── index.ts              ← 公共导出  
│   │  
│   ├── messages/                 ← 消息处理  
│   ├── models/                   ← 模型管理 (列表/切换)  
│   ├── skills/                   ← Skills 前端 (列表/启用/禁用)  
│   ├── memory/                   ← 记忆前端 (查看/编辑)  
│   ├── mcp/                      ← MCP 前端配置  
│   ├── todos/                    ← Todo 列表展示  
│   ├── tools/                    ← 工具调用展示  
│   ├── artifacts/                ← 产出文件展示  
│   ├── uploads/                  ← 文件上传管理  
│   ├── tasks/                    ← 后台任务 (子 Agent) 展示  
│   ├── settings/                 ← 设置页逻辑  
│   ├── config/                   ← 前端配置  
│   ├── agents/                   ← Agent 管理前端  
│   ├── streamdown/               ← Markdown 渲染  
│   ├── i18n/                     ← 国际化  
│   ├── notification/             ← 通知系统  
│   ├── rehype/                   ← Rehype 插件  
│   └── utils/                    ← 工具函数  
│  
├── hooks/                        ← 自定义 Hooks  
├── lib/                          ← 共享库  
├── server/                       ← 服务端代码 (Better Auth)  
├── styles/                       ← 全局样式  
└── env.js                        ← 环境变量验证
```

---

## 四、核心业务流转（6 条主线）

### 流转 1: 用户发送消息 → Agent 回复（最核心）

```
[用户在前端输入消息]  
       │  
       ▼  
1. useThreadStream() hook (frontend/src/core/threads/hooks.ts)  
   │  创建或复用 LangGraph thread  
   │  调用 client.runs.stream(thread_id, "lead_agent", { input, stream_mode })  
   │  
   ▼  
2. @langchain/langgraph-sdk (frontend/src/core/api/api-client.ts)  
   │  发送 HTTP POST 到 /api/langgraph/threads/{id}/runs/stream  
   │  接收 SSE 事件流  
   │  
   ▼  
3. Nginx 路由 (/api/langgraph/* → LangGraph Server :2024)  
   │  
   ▼  
4. LangGraph Server 接收请求  
   │  读取 langgraph.json → 调用 make_lead_agent(config)  
   │  
   ▼  
5. make_lead_agent() (agents/lead_agent/agent.py)  
   │  解析 config.configurable:  
   │    - model_name → 选择模型  
   │    - thinking_enabled → 思考模式  
   │    - is_plan_mode → Todo 跟踪  
   │    - subagent_enabled → 子 Agent  
   │  组装:  
   │    - model = create_chat_model(name, thinking_enabled)  
   │    - tools = get_available_tools(model_name, groups, subagent_enabled)  
   │    - middleware = _build_middlewares(config, model_name)  
   │    - system_prompt = apply_prompt_template(subagent_enabled, ...)  
   │  
   ▼  
6. langchain.agents.create_agent(model, tools, middleware, system_prompt, state_schema=ThreadState)  
   │  返回 CompiledStateGraph  
   │  LangGraph Server 用 checkpointer 持久化状态  
   │  
   ▼  
7. Agent 执行循环 (LangGraph ReAct Loop)  
   │  a. 调用 LLM → 获取回复 (可能包含 tool_calls)  
   │  b. 如果有 tool_calls → 执行工具 → 获取结果 → 回到 a  
   │  c. 如果没有 tool_calls → 返回最终回复  
   │  
   ▼  
8. SSE 流式返回  
   │  stream_mode=["values", "messages-tuple", "end"]  
   │  每个步骤通过 SSE event 推送到前端  
   │  
   ▼  
9. 前端 useThreadStream() 接收 SSE  
   │  解析事件 → 更新 React state → UI 渲染  
   │  
   ▼  
[用户看到 Agent 回复]
```

### 流转 2: 中间件处理链（每轮 Agent 调用都经过）

```
make_lead_agent 组装的中间件链 (按执行顺序):  
  
 1. ThreadDataMiddleware        创建线程隔离目录 (workspace/uploads/outputs)  
 2. UploadsMiddleware           注入已上传文件到消息上下文  
 3. SandboxMiddleware           获取沙箱环境实例  
 4. DanglingToolCallMiddleware  修补悬空 tool_call (确保 tool_call_id 匹配)  
 5. ToolErrorHandlingMiddleware 捕获工具异常 → 转为 ToolMessage 反馈给 LLM  
 6. SummarizationMiddleware     token 超阈值时压缩对话历史  
 7. TodoMiddleware              (plan_mode) 管理 todo 列表  
 8. TokenUsageMiddleware        token 用量追踪  
 9. TitleMiddleware             首次对话后自动生成标题  
10. MemoryMiddleware            对话结束后排队更新记忆  
11. ViewImageMiddleware         (vision模型) 注入图片 base64  
12. DeferredToolFilterMiddleware 过滤延迟加载的工具  
13. SubagentLimitMiddleware     限制并发子 Agent 数量  
14. LoopDetectionMiddleware     检测工具调用死循环  
15. ClarificationMiddleware     拦截澄清请求并中断 (永远在最后)
```

### 流转 3: 工具调用流程

```
[Agent 决定调用工具]  
       │  
       ▼  
1. LLM 返回 tool_calls: [{name: "bash", args: {command: "ls"}}]  
       │  
       ▼  
2. LangGraph 路由到对应工具函数  
   │  
   ├── bash_tool (sandbox/tools.py)  
   │   └── sandbox.execute_command(cmd) → CommandResult  
   │  
   ├── read_file_tool (sandbox/tools.py)  
   │   └── sandbox.read_file(path) → str  
   │  
   ├── write_file_tool (sandbox/tools.py)  
   │   └── sandbox.write_file(path, content) → None  
   │  
   ├── str_replace_tool (sandbox/tools.py)  
   │   └── 读→查找→替换→写回 (加文件锁)  
   │  
   ├── task_tool (tools/builtins/task_tool.py)  
   │   └── SubagentExecutor.execute(agent_name, prompt)  
   │       ├── 创建子 Agent (从 AgentRegistry 获取配置)  
   │       ├── 后台线程池运行  
   │       ├── 轮询等待结果  
   │       └── 返回结果给主 Agent  
   │  
   ├── MCP 工具 (mcp/tools.py)  
   │   └── 通过 stdio/SSE/HTTP 调用外部 MCP 服务器  
   │  
   └── 其他工具 (community/)  
       └── Tavily/Jina/Firecrawl/DuckDuckGo 等  
       │  
       ▼  
3. 工具结果作为 ToolMessage 返回给 LLM  
       │  
       ▼  
4. LLM 继续推理 (可能再次调用工具或生成最终回复)
```

### 流转 4: Skills 加载与注入

```
[Agent 启动时]  
       │  
       ▼  
1. apply_prompt_template() (agents/lead_agent/prompt.py)  
   │  调用 Skills 系统  
   │  
   ▼  
2. SkillLoader (skills/loader.py)  
   │  递归扫描 skills/public/ 和 skills/custom/  
   │  发现所有 SKILL.md 文件  
   │  
   ▼  
3. SkillParser (skills/parser.py)  
   │  解析 SKILL.md:  
   │    - YAML frontmatter: name, description, allowed-tools, license  
   │    - Body: Skill 正文 (Prompt + 工作流)  
   │  
   ▼  
4. 注入到 System Prompt  
   │  渐进式加载 (Progressive Disclosure):  
   │    - 初始: 只注入 name + description (元数据)  
   │    - 按需: Agent 调用 read_skill 时加载完整内容  
   │  
   ▼  
5. SkillManager (skills/manager.py)  
   │  管理 enable/disable 状态  
   │  存储在 extensions_config.json
```

### 流转 5: 记忆系统流转

```
[用户对话中]  
       │  
       ▼  
1. MemoryMiddleware (middlewares/memory_middleware.py)  
   │  每次 Agent run 结束后触发  
   │  将对话消息入队  
   │  
   ▼  
2. MemoryQueue (memory/queue.py)  
   │  Debounce: 等待一段时间再批量处理  
   │  避免频繁调用 LLM  
   │  
   ▼  
3. MemoryUpdater (memory/updater.py)  
   │  调用 LLM 分析对话内容  
   │  提取: 用户画像/偏好/事实/知识  
   │  使用结构化 Prompt (memory/prompt.py)  
   │  
   ▼  
4. MemoryStore (memory/storage.py)  
   │  更新 memory.json:  
   │    {  
   │      "user_context": { "work": "...", "personal": "..." },  
   │      "facts": [{ "content": "...", "confidence": 0.9 }],  
   │      "history": [...]  
   │    }  
   │  
   ▼  
5. 下次对话时  
   │  MemoryMiddleware 读取 memory.json  
   │  注入 top facts + user_context 到 System Prompt  
   │  Agent 基于记忆提供个性化回复
```

### 流转 6: 文件上传流转

```
[用户拖拽上传文件]  
       │  
       ▼  
1. 前端 uploads 组件  
   │  POST /api/threads/{thread_id}/uploads  
   │  multipart/form-data  
   │  
   ▼  
2. Gateway uploads.py router  
   │  接收文件  
   │  
   ▼  
3. UploadManager (uploads/manager.py)  
   │  转换文件格式:  
   │    PDF  → Markdown (markitdown)  
   │    PPT  → Markdown  
   │    Word → Markdown  
   │    Excel → Markdown  
   │  保存到线程目录: .deer-flow/threads/{thread_id}/uploads/  
   │  
   ▼  
4. UploadsMiddleware (下一轮对话时)  
   │  检测到新上传文件  
   │  注入到消息上下文: "用户上传了以下文件: ..."  
   │  
   ▼  
5. Agent 可以读取/处理这些文件
```

---

## 五、关键文件速查表

### 想了解 X，就看 Y 文件

| 我想了解... | 看这些文件 |
| --- | --- |
| **Agent 怎么创建的** | `agents/lead_agent/agent.py` → `factory.py` |
| **中间件有哪些、什么顺序** | `agents/lead_agent/agent.py::_build_middlewares()` |
| **Agent 状态有哪些字段** | `agents/thread_state.py` (ThreadState) |
| **对话怎么流式传输** | `frontend/src/core/threads/hooks.ts` (useThreadStream) |
| **前端怎么连接后端** | `frontend/src/core/api/api-client.ts` (LangGraphClient) |
| **System Prompt 怎么构建** | `agents/lead_agent/prompt.py` |
| **模型怎么配置/创建** | `models/factory.py` + `config/model_config.py` |
| **沙箱怎么工作** | `sandbox/sandbox.py` + `sandbox/sandbox_provider.py` + `sandbox/middleware.py` |
| **工具怎么注册** | `tools/tools.py::get_available_tools()` |
| **子 Agent 怎么执行** | `subagents/executor.py` + `tools/builtins/task_tool.py` |
| **Skills 怎么加载** | `skills/loader.py` + `skills/parser.py` |
| **记忆怎么工作** | `agents/memory/updater.py` + `agents/memory/storage.py` + `agents/middlewares/memory_middleware.py` |
| **配置有哪些选项** | `config/app_config.py` + 根目录 `config.yaml` |
| **Gateway 有哪些 API** | `app/gateway/app.py` (路由注册) + `app/gateway/routers/` |
| **IM 通道怎么接入** | `app/channels/service.py` + `app/channels/manager.py` |
| **MCP 怎么集成** | `mcp/client.py` + `mcp/tools.py` |
| **文件上传怎么处理** | `uploads/manager.py` + `app/gateway/routers/uploads.py` |
| **LangGraph 配置** | `langgraph.json` (入口: `deerflow.agents:make_lead_agent`) |

---

## 六、配置系统流转

```
config.yaml (项目根目录)  
       │  
       ▼  
get_app_config() (config/app_config.py)  
   │  解析 YAML → Pydantic 模型  
   │  支持 $ENV_VAR 环境变量替换  
   │  缓存单例  
   │  
   ├── models → config/model_config.py → models/factory.py  
   ├── tools → config/tool_config.py → tools/tools.py  
   ├── sandbox → config/sandbox_config.py → sandbox/sandbox_provider.py  
   ├── skills → config/skills_config.py → skills/loader.py  
   ├── memory → config/memory_config.py → agents/memory/storage.py  
   ├── subagents → config/subagents_config.py → subagents/config.py  
   ├── summarization → config/summarization_config.py → agents/middlewares/summarization_middleware.py  
   ├── tracing → config/tracing_config.py → tracing/factory.py  
   └── channels → app/channels/service.py  
  
extensions_config.json (项目根目录)  
   │  
   ├── mcpServers → mcp/client.py (MCP 服务器连接)  
   └── skills → skills/manager.py (Skills 启用/禁用状态)
```

---

## 七、两种运行模式

### Standard Mode (默认，4 进程)

```
Frontend (3000) ←→ Nginx (2026) ←→ Gateway API (8001)  
                                  ←→ LangGraph Server (2024)
```

* LangGraph Server 独立运行 Agent
* Gateway 通过 HTTP 与 LangGraph Server 通信
* 前端通过 Nginx 代理访问两个后端

### Gateway Mode (实验性，3 进程)

```
Frontend (3000) ←→ Nginx (2026) ←→ Gateway API (8001)  
                                      │ 内嵌 Agent 运行时  
                                      ├── RunManager  
                                      ├── RunWorker  
                                      ├── StreamBridge  
                                      └── Checkpointer + Store
```

* Gateway 内嵌 Agent 执行能力
* 不需要独立的 LangGraph Server
* 通过 `runtime/` 模块实现
* `make dev-pro` 或 `make dev --gateway` 启动

---

## 八、技术栈速查

| 层 | 技术 | 版本 |
| --- | --- | --- |
| **前端框架** | Next.js (App Router) | 16 |
| **UI** | React + Tailwind CSS 4 + Shadcn UI | 19 |
| **状态管理** | TanStack Query | v5 |
| **AI SDK** | @langchain/langgraph-sdk | 1.5.3 |
| **后端框架** | LangGraph + LangChain | 1.0.6+ |
| **API 服务** | FastAPI | 0.115.0+ |
| **Python** |  | 3.12+ |
| **包管理** | uv (后端) + pnpm (前端) |  |
| **沙箱** | Local 或 Docker (agent-sandbox) |  |
| **模型** | OpenAI 兼容 API (任何模型) |  |

---

💡 **万智创界 - AI技术实战派布道者**

关注我，你将获得：

* ✅ AI前沿动态与趋势
* ✅ 真实项目案例 + 代码
* ✅ 工程化实践与避坑

让 AI 真正为业务创造价值，从理论到落地，我们一起前行！

> 本文原文已同步到 GitHub，仓库地址如下：https://github.com/wanrengang/wanzhi-ai-lab