---
title: Hermes Agent 架构设计与技术文档
author: 笔辰的AI之旅
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYyMzM4MDY4MA==&mid=2247483698&idx=1&sn=5dac7348bb6f981f14a0accd1838fade&chksm=fee87ca769228683488e54eb48912f53ea9e13c2fc52e6ba4b5f6921ffcf0ad1c2a24498dac5&mpshare=1&scene=24&srcid=0410B0p457Y9q1OpY9frs026&sharer_shareinfo=4a38df034be6c080153314b42b93aa9c&sharer_shareinfo_first=4a38df034be6c080153314b42b93aa9c#rd
---

> 基于 v2026.4.8 源码分析

---

## 1. 项目概览

**Hermes Agent** 是由 Nous Research 开发的一个自改进式通用 AI Agent，主打 CLI 和多平台消息接入能力。

### 核心特性

* • 通用工具调用（Tool Calling）
* • 技能创建与复用（Skills System）
* • 持久化记忆（Persistent Memory）
* • 多平台消息网关（Telegram、Discord、Slack、WhatsApp 等）
* • 自改进式迭代循环
* • 上下文压缩与提示缓存

### 主要入口

| 入口 | 文件 | 职责 |
| --- | --- | --- |
| `AIAgent` | `run_agent.py` | 核心 Agent 循环 |
| `HermesCLI` | `cli.py` | 交互式终端 TUI |
| `GatewayRunner` | `gateway/run.py` | 消息平台网关 |
| `hermes` 命令 | `hermes_cli/main.py` | CLI 命令分发 |

---

## 2. 技术栈

### 语言与运行时

* • **Python 3.11+** — 核心实现
* • **JavaScript/Node.js** — 浏览器工具

### 核心依赖

| 类别 | 库 |
| --- | --- |
| LLM 接入 | `openai` , `anthropic` |
| 网络 | `httpx` |
| 数据验证 | `pydantic` |
| 配置 | `pyyaml` , `jinja2` |
| CLI 界面 | `rich` , `prompt_toolkit`, `fire` |
| 工具类 | `exa-py` , `firecrawl-py`, `parallel-web`, `fal-client` |
| 消息平台 | `python-telegram-bot` , `discord.py`, `slack-bolt` |
| 语音 | `edge-tts` , `faster-whisper` |
| 可选扩展 | `modal` , `daytona`, `honcho-ai`, `mcp` |

### 基础设施

* • Docker / Nix Flakes / Homebrew

---

## 3. 架构设计

### 3.1 核心 Agent 循环

位于 `run_agent.py` 的 `AIAgent` 类实现了一个同步工具调用循环：

```
while api_call_count < self.max_iterations and self.iteration_budget.remaining > 0:  
    response = client.chat.completions.create(  
        model=model, messages=messages, tools=tool_schemas  
    )  
    if response.tool_calls:  
        for tool_call in response.tool_calls:  
            result = handle_function_call(tool_call.name, tool_call.args, task_id)  
            messages.append(tool_result_message(result))
```

消息格式遵循 OpenAI 格式：`{"role": "system/user/assistant/tool", ...}`，其中 `assistant_msg["reasoning"]` 用于存储推理过程。

### 3.2 组件关系图

```
tools/registry.py (无依赖)  
    ↑ 导入  
tools/*.py (各模块在导入时调用 registry.register())  
    ↑  
model_tools.py (导入 registry，触发工具发现)  
    ↑  
run_agent.py, cli.py, batch_runner.py, environments/
```

### 3.3 关键组件

| 组件 | 路径 | 职责 |
| --- | --- | --- |
| `AIAgent` | `run_agent.py` | 对话循环、工具执行 |
| `HermesCLI` | `cli.py` | 交互式 TUI，正斜杠命令，自动补全 |
| `ToolRegistry` | `tools/registry.py` | 集中式工具 schema/handler 注册 |
| `MemoryManager` | `agent/memory_manager.py` | 多记忆提供者编排 |
| `ContextCompressor` | `agent/context_compressor.py` | 长对话自动摘要压缩 |
| `GatewayRunner` | `gateway/run.py` | 多平台消息路由 |
| `SessionStore` | `hermes_state.py` | SQLite FTS5 会话存储 |
| `CronScheduler` | `cron/scheduler.py` | 定时任务调度执行 |

---

## 4. 功能模块

### 4.1 工具系统（Tools）

约 40 个工具，组织为多个工具集（toolsets）：

| 工具 | 路径 | 功能 |
| --- | --- | --- |
| `terminal_tool.py` | `tools/` | 命令执行（本地/Docker/SSH/Daytona/Singularity/Modal） |
| `browser_tool.py` | `tools/` | 浏览器自动化 |
| `file_tools.py` | `tools/` | 文件读写与打补丁 |
| `web_tools.py` | `tools/` | 网页搜索与内容提取 |
| `delegate_tool.py` | `tools/` | 子 Agent 委托 |
| `mcp_tool.py` | `tools/` | MCP 客户端集成 |
| `memory_tool.py` | `tools/` | 内置 MEMORY.md/USER.md |
| `todo_tool.py` | `tools/` | 内存任务规划 |
| `skills_tool.py` | `tools/` | 技能管理 |

### 4.2 技能系统（Skills）

技能文件位于 `~/.hermes/skills/`，格式为 Markdown + YAML frontmatter，作为程序性记忆（procedural memory）。

技能在加载时作为**用户消息**注入（而非系统提示），以保留提示缓存优势。

### 4.3 消息网关（Gateway）

支持平台：

* • 即时通讯：Telegram、Discord、Slack、WhatsApp、Signal
* • 智能家居：Home Assistant、Matrix
* • 企业协作：DingTalk、FeiShu、WeCom
* • 其他：SMS、Email、Webhook

### 4.4 定时任务（Cron）

基于文件的定时任务，支持自然语言 cron 语法解析，多平台投递。

---

## 5. Agent 系统核心剖析

### 5.1 记忆架构（Memory System）

#### 记忆提供者抽象（MemoryProvider ABC）

所有记忆提供者继承 `agent/memory_provider.py` 中的 `MemoryProvider` 抽象类，其生命周期方法：

```
initialize() → system_prompt_block() → prefetch() → sync_turn()  
→ get_tool_schemas() → handle_tool_call() → shutdown()
```

可选钩子：

* • `on_turn_start()` — 回合开始时
* • `on_session_end()` — 会话结束时
* • `on_pre_compress()` — 压缩前
* • `on_memory_write()` — 记忆写入时
* • `on_delegation()` — 委托时

#### MemoryManager 编排逻辑

`agent/memory_manager.py` 中的 `MemoryManager` 规则：

1. 1. **内置提供者最先注册**，不可移除
2. 2. **只允许一个外部插件提供者**，防止 schema 膨胀
3. 3. **各提供者独立失败**，一个失败不影响其他

#### 内置记忆（BuiltinMemoryProvider）

使用 `tools/memory_tool.py` 实现，管理两个核心文件：

| 文件 | 用途 |
| --- | --- |
| `MEMORY.md` | Agent 个人笔记，记录环境事实、项目约定 |
| `USER.md` | 用户画像，记录偏好、沟通风格 |

**Frozen Snapshot 模式**：会话开始时以快照形式注入系统提示（保留提示缓存）；会话中期写入文件不会改变已注入的系统提示。

#### 外部记忆插件

`plugins/memory/` 下支持：

| 插件 | 来源 |
| --- | --- |
| `honcho/` | Honcho AI dialectic 用户建模 |
| `mem0/` | Mem0 Platform 语义搜索 |
| `holographic/` | 全息记忆存储 |
| `hindsight/` | 回顾性记忆 |
| `retaindb/` | RetainDB |
| `openviking/` | OpenViking |
| `byterover/` | ByteRover |
| `supermemory/` | SuperMemory |

### 5.2 运行时系统（Runtime）

#### 迭代预算（Iteration Budget）

* • 线程安全计数器
* • 父 Agent 默认 90 次迭代
* • 子 Agent 独立继承预算
* • `execute_code` 迭代会被退款，不消耗预算

#### 上下文压缩（Context Compressor）

`agent/context_compressor.py` 实现：

* • 在上下文使用达到 50% 时触发（可配置）
* • 算法：

1. 1. 裁剪旧工具结果
2. 2. 保护头部 + 尾部消息
3. 3. LLM 摘要中间部分

* • 迭代式摘要保证压缩后的历史信息不丢失

#### 提示缓存（Prompt Caching）

`agent/prompt_caching.py`：

* • Claude 模型通过 OpenRouter 或原生 Anthropic 自动启用
* • 使用 `system_and_3` 策略（4 个断点），5 分钟 TTL

#### 提供者路由（Provider Routing）

`agent/auxiliary_client.py` 支持多 Provider 自动路由：

| Provider | 模式 |
| --- | --- |
| OpenRouter | `chat_completions` |
| Nous Portal | `chat_completions` |
| OpenAI | `chat_completions` |
| Anthropic | `anthropic_messages` |
| Google | 待查 |
| Kimi | 待查 |
| MiniMax | 待查 |

自动检测 API 模式（`chat_completions` vs `anthropic_messages` vs `codex_responses`）。

### 5.3 底层框架（Underlying Framework）

#### 工具发现机制

`model_tools.py` 中的 `_discover_tools()` 在启动时导入所有工具模块，各模块在导入时调用 `registry.register()` 完成自注册，无需中心化列表。

#### 工具并行执行

通过 `ThreadPoolExecutor`（最大 8 个 worker）并行执行工具。

#### 工具级拦截

`run_agent.py` 中 `todo` 和 `memory` 工具在正常调度前被拦截；`delegate` 工具以 `skip_memory=True` 启动子 Agent。

#### 路径作用域工具冲突检查

`read_file`、`write_file`、`patch_file` 等路径相关工具执行前检查冲突。

---

## 6. 部署方案

### 6.1 Docker 部署

* • `Dockerfile` — Debian 基础镜像，安装 Python + Node 依赖
* • `docker/entrypoint.sh` — 启动脚本
* • 数据卷：`HERMES_HOME=/opt/data`

### 6.2 一键安装（Linux/macOS/WSL2）

```
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

### 6.3 源码安装

```
git clone https://github.com/NousResearch/hermes-agent.git  
cd hermes-agent  
uv venv venv --python 3.11  
source venv/bin/activate  
uv pip install -e ".[all,dev]"
```

### 6.4 配置

* • `~/.hermes/config.yaml` — 主配置（模型、Provider、工具、记忆等）
* • `~/.hermes/.env` — API Key 和密钥
* • `HERMES_HOME` 环境变量 — 多实例隔离，支持独立配置文件

### 6.5 CLI 命令

```
hermes              # 交互式 CLI  
hermes model        # 选择 LLM Provider  
hermes tools        # 配置工具集  
hermes gateway      # 启动消息网关  
hermes setup        # 完整设置向导  
hermes memory setup # 配置记忆提供者
```

---

## 7. 关键设计模式

### 7.1 Frozen Snapshot 模式

记忆在会话开始时以快照形式注入系统提示，保留提示缓存；会话中期的写入更新文件但不改变已注入的系统提示。

### 7.2 提供者插件架构

记忆系统通过 ABC 定义生命周期，各提供者独立注册，支持热插拔。

### 7.3 Profile 隔离

`HERMES_HOME` 环境变量将所有状态（配置、记忆、会话、技能）作用域限定到各实例目录。

### 7.4 自注册工具注册表

各工具模块在导入时通过 `registry.register()` 自注册，无需中心化清单。

### 7.5 Safe STDIO

`_SafeWriter` 封装器捕获 `OSError`/`ValueError`（Broken Pipe），兼容 systemd/Docker/headless 场景。

---

## 8. 文件结构一览

```
hermes-agent/  
├── run_agent.py              # AIAgent 核心循环  
├── cli.py                    # HermesCLI 交互界面  
├── model_tools.py            # 工具发现与执行引擎  
├── tools/  
│   ├── registry.py           # ToolRegistry 注册中心  
│   ├── terminal_tool.py     # 终端工具集  
│   ├── browser_tool.py       # 浏览器工具  
│   ├── file_tools.py        # 文件工具集  
│   ├── web_tools.py         # Web 工具集  
│   ├── memory_tool.py        # 内置记忆工具  
│   ├── todo_tool.py         # 任务规划工具  
│   ├── delegate_tool.py     # 委托工具  
│   ├── mcp_tool.py          # MCP 客户端  
│   └── skills_tool.py       # 技能工具  
├── agent/  
│   ├── memory_manager.py    # 记忆管理者  
│   ├── memory_provider.py   # MemoryProvider ABC  
│   ├── context_compressor.py # 上下文压缩器  
│   ├── prompt_caching.py    # 提示缓存  
│   ├── auxiliary_client.py   # 多 Provider 路由  
│   └── skill_commands.py     # 技能命令  
├── gateway/  
│   └── run.py               # 消息网关  
├── cron/  
│   └── scheduler.py        # Cron 调度器  
├── hermes_state.py          # SQLite 会话存储  
├── hermes_cli/  
│   └── main.py              # CLI 入口  
├── skills/                  # 技能定义  
├── plugins/memory/          # 外部记忆插件  
└── docker/  
    └── entrypoint.sh        # Docker 入口脚本
```

---

## 9. Hermes vs OpenClaw 对比分析

> OpenClaw 版本：2026.4.8

### 9.1 概览对比

| 维度 | Hermes Agent | OpenClaw |
| --- | --- | --- |
| **组织** | Nous Research | openclaw（独立开源项目） |
| **语言** | Python 3.11+ | TypeScript / Node.js（v22.16+） |
| **运行时** | 自研同步工具调用循环 | Pi Agent（`@mariozechner/pi-agent-core`） |
| **记忆架构** | MemoryProvider ABC + Frozen Snapshot | Dreaming System（三阶段记忆巩固） |
| **上下文管理** | ContextCompressor（LLM 摘要中间段） | ContextEngine（可插拔，assemble/compact/maintain） |
| **工具数量** | ~40 个 | 111 个扩展插件（工具集） |
| **消息平台** | 约 12 个 | 30+ 个 |
| **技能数量** | 从文件系统加载 | 55 个捆绑技能 |
| **部署形态** | CLI + Docker + Homebrew | Daemon 长期运行 + CLI + Docker |
| **数据存储** | SQLite FTS5 + 文件系统 | 文件 transcript + 插件化存储（LanceDB 等） |

### 9.2 架构层面对比

#### Agent 运行时

**Hermes** 采用自研同步循环，逻辑透明、直接可控：

```
# Hermes: run_agent.py  
while api_call_count < self.max_iterations and self.iteration_budget.remaining > 0:  
    response = client.chat.completions.create(model=model, messages=messages, tools=tool_schemas)
```

**OpenClaw** 依赖第三方 Pi Agent，提供了更抽象的运行时封装，但黑盒程度更高：

```
src/agents/agent-command.ts    # 34KB 主处理器  
src/agents/agent-scope.ts     # 工作区作用域管理  
src/agents/acp-spawn.ts       # 子 Agent 孵化（39KB）
```

OpenClaw 额外支持模型 failover（自动切换 Provider）和模型覆盖（per-session 调整模型）。

#### 记忆系统

两者走了截然不同的路线：

| 维度 | Hermes | OpenClaw |
| --- | --- | --- |
| **内置方案** | `MEMORY.md` + `USER.md` 文件（Frozen Snapshot） | `memory-core` 插件（Dreaming 三阶段） |
| **外部插件** | 8 个插件（mem0、holographic、honcho 等） | `memory-lancedb` 、`memory-wiki` 等 |
| **记忆 consolidation** | 无自动 consolidation，靠用户手动写入 | Dreaming System：Light → Deep → REM 三阶段 |
| **记忆注入方式** | 会话开始时快照注入，保留 prompt cache | 按需 assemble，通过 ContextEngine 管理 |

**Dreaming System** 是 OpenClaw 最独特的创新——受睡眠研究启发，在凌晨定时（默认 `0 3 * * *`）对记忆进行轻/深/REM 三个阶段的处理，实现自动化记忆整合。

Hermes 则走实用路线：记忆就是 Markdown 文件，结构简单、人类可读、版本控制友好。

#### 上下文压缩

* • **Hermes**：50% 阈值触发，裁剪旧工具结果 + 保护头尾 + LLM 摘要中间段
* • **OpenClaw**：ContextEngine 支持 `compact()` 方法，策略可插拔，assemble 时在 token 预算内动态构建

OpenClaw 的上下文管理更灵活但也更复杂；Hermes 的压缩算法更直观但干预时机较机械。

#### 工具系统

| 维度 | Hermes | OpenClaw |
| --- | --- | --- |
| **注册方式** | 模块导入时 `registry.register()` 自注册 | Plugin SDK + Extension 体系 |
| **并行执行** | `ThreadPoolExecutor` （max 8 workers） | 通过 Extension SDK 分发 |
| **路径工具冲突检查** | 有（read\_file/write\_file/patch\_file） | 未明确 |
| **MCP 集成** | `mcp_tool.py` 客户端 | `mcporter` 外部桥接 |
| **技能系统** | Markdown + YAML frontmatter，作为 user message 注入 | 55 个捆绑技能（GitHub、Notion、Obsidian 等） |

OpenClaw 的 Extension 生态（111 个插件）远超 Hermes 的工具集规模，但 Hermes 的工具更侧重命令行/开发场景（terminal、browser、file），OpenClaw 的工具更侧重平台集成（Slack、Telegram、GitHub API）。

### 9.3 消息网关对比

| 维度 | Hermes | OpenClaw |
| --- | --- | --- |
| **实现方式** | Python `python-telegram-bot`、`discord.py` 等 | TypeScript `ws` WebSocket 架构 |
| **平台数量** | ~12 个 | 30+ 个 |
| **特色功能** | 企业协作（DingTalk、FeiShu、WeCom） | Signal、Live Canvas、A2UI 可视化工作区 |
| **控制面** | `gateway/run.py` | Gateway WS control plane + Web UI |

OpenClaw 提供了 Web 控制台（Dashboard）和 Live Canvas；Hermes 是纯 CLI/TUI 体验。

### 9.4 安全模型对比

| 维度 | Hermes | OpenClaw |
| --- | --- | --- |
| \*\* pairing\*\* | 无强制 pairing | DM 默认需要配对码（Pairing Code） |
| **敏感操作审批** | 无内置 | 多级 Approval System |
| **网络安全** | 未明确 | SSRF 防护、输入净化、发件人白名单 |

OpenClaw 在安全模型上更成熟，适合作为多租户/公开消息平台的接入层；Hermes 更适合个人/受控环境。

### 9.5 部署与生态对比

| 维度 | Hermes | OpenClaw |
| --- | --- | --- |
| **Daemon 化** | 无原生 daemon，通过 Docker/systemd | 原生 daemon（launchd/systemd） |
| **客户端** | 纯 CLI（TUI） | macOS 菜单栏应用 + iOS/Android companion |
| **语音能力** | `edge-tts` + `faster-whisper` | Voice Wake/Talk（macOS/iOS）+ Android 连续语音 |
| **UI** | Terminal TUI | Web Dashboard + Live Canvas（A2UI） |
| **包管理** | Homebrew + Nix + 源码 | npm/pnpm 全局安装 |

### 9.6 设计哲学总结

| 维度 | Hermes | OpenClaw |
| --- | --- | --- |
| **哲学** | 简单、透明、可破解 | 生态丰富、功能完备、企业级 |
| **复杂度** | 低（单仓库 ~15 个核心文件） | 高（百级 Extension + Plugin SDK + ACP 协议） |
| **可定制性** | 直接改源码，Python 生态丰富 | 写 Extension 插件，或 fork 后改 ACP |
| **适用场景** | 个人开发者CLI工具、本地开发辅助 | 多平台消息聚合、商业/团队协作 |
| **学习曲线** | 陡（需理解 Python 异步/迭代预算/压缩逻辑） | 平缓（npm 安装即用，但深层定制需理解 Pi Agent） |

**Hermes** 是"给我一个 Agent，自己改"的设计——代码量小、逻辑清晰、Python 原生，适合深度定制或嵌入自有系统。

**OpenClaw** 是"给我一个 Agent，什么都能接"的设计——开箱即用、插件生态丰富、30+ 平台接入，适合作为个人/团队的多合一 AI 消息中枢。

### 9.7 关键差异速查表

```
Hermes                              OpenClaw  
══════════                          ══════════  
Python 3.11+                     →  TypeScript / Node.js v22+  
自研同步循环                       →  Pi Agent runtime  
MEMORY.md 文件                     →  Dreaming 三阶段 consolidation  
手动写入记忆                       →  自动凌晨记忆整合  
迭代预算计数器                     →  模型 failover + per-session override  
8 worker 并行工具                  →  111 个 Extension 插件  
~12 消息平台                       →  30+ 消息平台  
纯 CLI/TUI                        →  Web Dashboard + Live Canvas  
个人/开发者场景                   →  多租户/团队协作场景  
零安全模型                        →  Pairing + Approval + SSRF 防护
```