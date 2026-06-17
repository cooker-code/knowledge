---
title: AgentMemory 解析：给 Coding Agent 装上本地长期记忆
author: AI贺贺
date: 艾贺艾贺
url: https://mp.weixin.qq.com/s?__biz=MzAxNjIzOTkyOQ==&mid=2449867900&idx=1&sn=2f0be3d5ffc1d10d84c38c394e70a9a1&chksm=8da38ba7e44698be942fa3c3ceff3230b2b0ee918010aced5ebee8cd56c98896b9c18f107518&mpshare=1&scene=24&srcid=0603D11sikNqRUpZefPPvWlY&sharer_shareinfo=5b0b6d4a2e28eafdb70a33ef298dd5d2&sharer_shareinfo_first=5b0b6d4a2e28eafdb70a33ef298dd5d2#rd
---

`@agentmemory/agentmemory` 很容易被误解成“又一个向量数据库包装器”。看完源码后，我更愿意把它定义成一个本地 Agent 记忆运行时：它不是只提供 `save` 和 `search`，而是把 hook 捕获、隐私过滤、观察记录、压缩、索引、检索、上下文注入、MCP 工具、REST API、viewer、审计和多 Agent 协作都放进了一个可启动的本地服务里。

一句话讲清楚：

> AgentMemory 通过 Agent hooks 自动捕获会话和工具调用，把它们写成观察记录，再用 BM25、向量和图关系检索，在下一次会话里按预算找回最相关的上下文。

我分析的是 npm 当前最新包 `@agentmemory/agentmemory@0.9.20`，对应 GitHub 仓库 `rohitg00/agentmemory`。这篇文章不复述官网宣传，而是从代码路径出发，讲清楚它到底怎么实现、应该怎么用、以及哪些开关要谨慎打开。

## 一、先说结论：它解决的不是“存储”，而是“下次还记得”

写代码 Agent 的长期记忆通常有三种做法。

第一种是静态文件，比如 `CLAUDE.md`、`AGENTS.md`、Cursor rules。优点是透明，缺点是很快变成“塞满但不精准”的大文本。

第二种是向量库。它能检索，但需要你自己决定什么时候写入、写什么、怎么压缩、怎么删除、怎么注入回上下文。

第三种是 AgentMemory 这种运行时方案：它把“记忆”看成一个完整生命周期，而不是一个数据库表。

这张图里最关键的是两个“默认关闭”：

* `AGENTMEMORY_AUTO_COMPRESS` 默认关闭：避免每个工具调用都烧 LLM token。
* `AGENTMEMORY_INJECT_CONTEXT` 默认关闭：避免每次工具调用都把记忆塞进模型上下文。

也就是说，它默认是一个后台记录器，而不是一个会悄悄改变模型输入的东西。这一点很重要。

## 二、安装包与运行形态：它不是一个普通 npm 库

从 npm 包看，`@agentmemory/agentmemory@0.9.20` 是一个 Node ESM CLI 包：

```
{  
  "name": "@agentmemory/agentmemory",  
"version": "0.9.20",  
"bin": {  
    "agentmemory": "dist/cli.mjs"  
  },  
"main": "dist/index.mjs",  
"engines": {  
    "node": ">=20.0.0"  
  }  
}
```

它发布到 npm 的内容不是只有 `dist/`，还包括：

```
dist/                 # 编译后的运行时  
plugin/               # Claude / Codex 插件、hooks、skills、MCP 配置  
iii-config.yaml       # iii-engine 本地运行配置  
docker-compose.yml    # Docker fallback  
.env.example          # 配置模板  
README.md  
AGENTS.md  
LICENSE
```

这说明它的产品形态是“一条 CLI 启动一套本地记忆服务”，而不是让你在业务代码里 import 一个库。

它有几个主要入口：

```
npm install -g @agentmemory/agentmemory  
agentmemory  
agentmemory demo  
agentmemory connect codex  
agentmemory doctor  
agentmemory status  
agentmemory mcp  
agentmemory import-jsonl
```

如果不用全局安装，也可以：

```
npx -y @agentmemory/agentmemory@latest
```

源码里 `src/cli.ts` 负责 CLI 入口。它会处理启动 iii-engine、连接不同 Agent、诊断、导入 JSONL、停止服务、清理卸载等命令。

## 三、核心架构：Node Worker 跑在 iii-engine 上

AgentMemory 的底层不是 Express + SQLite 这种常见 Node 服务，而是构建在 `iii-engine` 上。`iii-engine` 提供几个基础能力：

* HTTP triggers
* WebSocket streams
* KV state
* worker supervision
* cron / queue / pubsub / observability

这里要把 `iii-engine` 单独说清楚。它不是 npm 里的一个普通 JS library，也不是 AgentMemory 自己写的小型 HTTP server。它是 `iii-hq/iii` 项目的核心运行时，本体主要是 Rust 写的本地二进制，安装后通常体现为一个 `iii` 命令。

AgentMemory 里的关系可以这样理解：

```
agentmemory = Node CLI + memory worker + plugin/hooks/skills  
iii         = Rust runtime / 本地服务引擎
```

你运行：

```
agentmemory
```

背后实际会启动或复用 `iii` 这个本地 engine。`iii` 负责读取配置、监听端口、管理 worker、路由 function 调用、提供 KV/stream/HTTP/cron/queue 等底层能力；AgentMemory 的 `node dist/index.mjs` 则作为一个 worker 连接进去，注册 `mem::observe`、`mem::search`、`mem::context` 这些记忆业务函数。

这也是为什么 AgentMemory 不是纯 JS 进程就结束了。它的运行时更像：

```
agentmemory CLI  
  -> 启动 iii Rust 二进制  
  -> iii 读取 iii-config.yaml  
  -> iii 打开 REST / stream / worker bus  
  -> iii-exec 运行 node dist/index.mjs  
  -> Node worker 注册 AgentMemory 的 functions 和 triggers
```

`iii` 项目本身是公开源码的 runtime/platform，不只是一个库。需要注意 license 边界：`engine/` 使用 Elastic License 2.0，SDK、CLI、Console、docs 等是 Apache License 2.0。也就是说，可以看源码、clone、本地跑、自托管；但如果要把 engine 包进商业托管服务或做再分发，需要认真看 ELv2 的限制。

`iii-config.yaml` 默认配置里能看到几个关键端口：

```
workers:  
  -name:iii-http  
    config:  
      port:3111  
      host:127.0.0.1  
-name:iii-stream  
    config:  
      port:3112  
      host:127.0.0.1  
-name:iii-exec  
    config:  
      exec:  
        -nodedist/index.mjs
```

所以实际运行时是：

`src/index.ts` 是真正的核心注册器。它启动后会做这些事情：

1. 读取 `~/.agentmemory/.env` 和进程环境变量。
2. 检测 LLM provider 和 embedding provider。
3. 创建 `StateKV`、BM25 index、vector index。
4. 注册大量 `mem::*` 函数，比如 `mem::observe`、`mem::compress`、`mem::search`、`mem::context`、`mem::remember`。
5. 注册 REST API triggers。
6. 注册 MCP endpoint。
7. 启动 viewer。
8. 加载或重建持久化索引。
9. 定时执行 auto-forget、lesson decay、consolidation。

关键点：AgentMemory 的每个能力都被包装成 iii function。REST API、MCP 工具、hook 脚本本质上都只是这些 function 的不同入口。

如果不用 `iii-engine`，AgentMemory 就要自己维护 HTTP server、KV 存储、WebSocket stream、定时任务、worker 生命周期、日志和 trace。现在这些底层能力都被收敛到 `iii` 的三个抽象里：

| 抽象 | 含义 | 在 AgentMemory 里的例子 |
| --- | --- | --- |
| Worker | 连接到 engine 并注册能力的进程 | `node dist/index.mjs` |
| Function | 有稳定 ID 的可调用处理器 | `mem::observe` 、`mem::search` |
| Trigger | 触发 function 的入口 | HTTP、cron、stream、direct trigger |

## 四、核心数据模型：Session、Observation、Memory

源码里的核心类型在 `src/types.ts`。

`Session` 记录一次 Agent 会话：

```
interface Session {  
  id: string;  
  project: string;  
  cwd: string;  
  startedAt: string;  
  endedAt?: string;  
  status: "active" | "completed" | "abandoned";  
  observationCount: number;  
  firstPrompt?: string;  
  summary?: string;  
}
```

`RawObservation` 是 hook 捕获到的原始事件：

```
interface RawObservation {  
  id: string;  
  sessionId: string;  
  timestamp: string;  
  hookType: HookType;  
  toolName?: string;  
  toolInput?: unknown;  
  toolOutput?: unknown;  
  userPrompt?: string;  
  raw: unknown;  
}
```

`CompressedObservation` 是真正用于检索的结构化记录：

```
interface CompressedObservation {  
  id: string;  
  sessionId: string;  
  timestamp: string;  
  type: ObservationType;  
  title: string;  
  facts: string[];  
  narrative: string;  
  concepts: string[];  
  files: string[];  
  importance: number;  
}
```

`Memory` 则是显式保存的长期记忆：

```
interface Memory {  
  id: string;  
  type: "pattern" | "preference" | "architecture" | "bug" | "workflow" | "fact";  
  title: string;  
  content: string;  
  concepts: string[];  
  files: string[];  
  strength: number;  
  version: number;  
  isLatest: boolean;  
  supersedes?: string[];  
}
```

KV scope 在 `src/state/schema.ts` 里定义，比如：

```
sessions: "mem:sessions"  
observations: (sessionId) => `mem:obs:${sessionId}`  
memories: "mem:memories"  
summaries: "mem:summaries"  
relations: "mem:relations"  
audit: "mem:audit"  
actions: "mem:actions"  
slots: "mem:slots"  
commits: "mem:commits"
```

这套 schema 说明它不是只存聊天记录，而是把“会话、观察、总结、长期记忆、关系、审计、任务、租约、信号、slot”都作为记忆系统的一部分。

## 五、核心实现链路：Hook 如何变成可检索记忆

以 Codex plugin 为例，npm 包里的 `plugin/.codex-plugin/plugin.json` 会注册：

```
{  
  "name": "agentmemory",  
  "mcpServers": "./.mcp.json",  
  "hooks": "./hooks/hooks.codex.json",  
  "skills": "./skills/"  
}
```

Codex 的 hook 只有 6 个：

```
SessionStart  
UserPromptSubmit  
PreToolUse  
PostToolUse  
PreCompact  
Stop
```

Claude Code plugin 注册更多，OpenCode plugin 甚至有 22 个 hook。但原理一样：hook 脚本从 stdin 读取宿主传来的 JSON，再 POST 到本地 REST API。

例如 `src/hooks/post-tool-use.ts` 做的事情很直接：

1. 读取 hook 输入。
2. 提取 `session_id`、`tool_name`、`tool_input`、`tool_output`。
3. 截断过长输出。
4. 如果输出里有 base64 图片，把图片数据抽出来。
5. POST 到：

```
POST http://localhost:3111/agentmemory/observe
```

body 大致是：

```
{  
  "hookType": "post_tool_use",  
"sessionId": "...",  
"project": "...",  
"cwd": "...",  
"timestamp": "...",  
"data": {  
    "tool_name": "Read",  
    "tool_input": {},  
    "tool_output": "..."  
  }  
}
```

REST 入口在 `src/triggers/api.ts`，它再触发：

```
mem::observe
```

真正写入逻辑在 `src/functions/observe.ts`：

这里有几个工程细节值得注意。

第一，写入前会做隐私过滤。`observe.ts` 会先 `JSON.stringify`，再调用 `stripPrivateData`，然后再解析回 JSON。它不会保证所有敏感信息都能识别，但至少把过滤放在了入库前。

第二，默认不调用 LLM。`AGENTMEMORY_AUTO_COMPRESS` 没开时，它走 `buildSyntheticCompression(raw)`，直接生成可检索的结构化观察，并加入 BM25 / vector index。

第三，hook 失败不会阻塞 Agent。多数 hook 脚本都用了短 timeout 和 silent catch。这是对的：记忆系统不应该让 `Read`、`Edit`、`Write` 变慢或失败。

### 5.1 压缩链路：LLM 是增强项，不是启动前提

如果打开：

```
AGENTMEMORY_AUTO_COMPRESS=true
```

每条 observation 会触发 `mem::compress`。源码在 `src/functions/compress.ts`。

LLM 压缩流程大致是：

1. 如果 observation 包含图片，先尝试用 vision provider 生成图片描述。
2. 用 `buildCompressionPrompt` 构造压缩 prompt。
3. 调用 provider。
4. 期望模型返回 XML。
5. 解析 `<type>`、`<title>`、`<facts>`、`<concepts>`、`<files>`、`<importance>` 等字段。
6. 用 Zod schema 校验输出。
7. 计算 quality score。
8. 写回 KV。
9. 加入 BM25 和 vector index。
10. 推送 viewer stream。

它支持多种 provider：

```
OpenAI-compatible  
MiniMax  
Anthropic  
Gemini  
OpenRouter  
agent-sdk fallback  
noop
```

但默认是 `noop`。`src/config.ts` 里明确写了：没有 provider key 时，LLM 压缩和总结关闭，只保留 synthetic compression 与搜索。

这对普通开发者是好事。否则一个活跃的 Codex/Claude Code 会话里，几十上百次工具调用会把 LLM 压缩成本放大到不可控。

### 5.2 检索链路：BM25 + Vector + Graph，再用 RRF 融合

AgentMemory 的检索不是单纯向量召回。核心在 `src/state/hybrid-search.ts`。

它有三路信号：

```
BM25     关键词、路径、概念、文件名、错误信息  
Vector   embedding 相似度  
Graph    实体和关系扩展
```

BM25 由 `src/state/search-index.ts` 实现，特点是：

* 自己维护 inverted index。
* 使用 BM25 公式。
* 支持 stem。
* 支持 synonyms。
* 支持 prefix match。
* CJK 文本可用 `@node-rs/jieba` / `tiny-segmenter` 做分词。

Vector index 在 `src/state/vector-index.ts`，实现很朴素：

* 内存里维护 `Map<obsId, Float32Array>`。
* 查询时逐个算 cosine similarity。
* 支持序列化到 base64。
* 启动时会校验 embedding 维度，避免不同 provider 写出的向量混在一起导致检索静默失效。

Hybrid search 的融合方式是 RRF：

```
combinedScore =  
  bm25Weight * (1 / (RRF_K + bm25Rank)) +  
  vectorWeight * (1 / (RRF_K + vectorRank)) +  
  graphWeight * (1 / (RRF_K + graphRank))
```

然后再做两件事：

* session diversification：默认每个 session 最多拿 3 条，避免一个会话刷屏。
* optional rerank：`RERANK_ENABLED=true` 时对前 20 条做重排。

这套方案比纯向量搜索更适合代码场景。因为代码记忆里很多查询是精确关键词：文件路径、函数名、错误码、命令、commit SHA。纯向量可能会把这些细节稀释掉，BM25 正好补上。

### 5.3 上下文注入：默认很克制

`mem::context` 在 `src/functions/context.ts`。它会从几个来源组装上下文：

* pinned memory slots
* project profile
* lessons
* 最近 sessions 的 summary
* 没有 summary 时的重要 observations

然后按 token budget 选择能塞进去的块，最后输出 XML 包裹的上下文：

```
<agentmemory-context project="...">  
...  
</agentmemory-context>
```

但这段上下文默认不会注入到模型输入。

为什么？看 `src/hooks/pre-tool-use.ts` 的注释就很清楚：早期版本会在每次 file-touching tool call 前注入上下文，这会让 Claude Code / Codex 的输入 token 随工具调用次数线性膨胀。于是现在 `PreToolUse` 默认直接 return，只有显式设置：

```
AGENTMEMORY_INJECT_CONTEXT=true
```

才会真正调用 `/agentmemory/enrich` 并把结果写到 stdout。

`SessionStart` 也是类似逻辑：

* 默认：只注册 session，不输出 context。
* 开启 `AGENTMEMORY_INJECT_CONTEXT=true`：才把 `/agentmemory/session/start` 返回的 context 写到 stdout。

`PreCompact` 不同，它会在压缩前尝试输出一段上下文，帮助模型在上下文压缩时保留记忆。

实际使用建议是：先不开注入，让它记录几天；确认 recall 质量后，再在特定场景开启注入。

## 六、对外接口：MCP、REST 和 Codex 插件

AgentMemory 同时提供 REST API 和 MCP。

MCP 有两层：

```
@agentmemory/mcp  
  -> 如果能连上 http://localhost:3111，代理到完整 AgentMemory server  
  -> 如果连不上，退化成本地 InMemoryKV fallback
```

standalone shim 在 `src/mcp/standalone.ts`，探测逻辑在 `src/mcp/rest-proxy.ts`。它会访问：

```
GET /agentmemory/livez
```

如果 server 可用，就代理到完整 REST/MCP surface；如果不可用，就用本地 fallback。

这解释了一个常见问题：

> 为什么 MCP 里只有几个 memory 工具？

因为你可能只启动了 standalone MCP，没有启动完整 server。源码里 fallback 只实现了 7 个工具：

```
memory_save  
memory_recall  
memory_smart_search  
memory_sessions  
memory_export  
memory_audit  
memory_governance_delete
```

而 `src/mcp/tools-registry.ts` 在当前版本定义了 53 个 `memory_*` 工具。注意，README 和部分文案里有时写 51 或 53，这是文档口径没有完全同步；以 `0.9.20` 源码为准，registry 里是 53 个。

默认服务端可见工具不是全部。`getVisibleTools()` 读取：

```
AGENTMEMORY_TOOLS=core
```

默认 `core` 只暴露一组 essential tools。想在 MCP client 里看到完整工具，需要：

```
AGENTMEMORY_TOOLS=all
```

常用工具可以按场景理解：

| 场景 | 工具 |
| --- | --- |
| 查过去做过什么 | `memory_recall` 、`memory_smart_search` |
| 显式保存长期知识 | `memory_save` |
| 看最近会话 | `memory_sessions` |
| 查文件历史 | `memory_file_history` |
| 查项目画像 | `memory_profile` |
| 查关系图 | `memory_graph_query` 、`memory_relations` |
| 删除隐私数据 | `memory_governance_delete` 、`memory_audit` |
| 多 Agent 协作 | `memory_action_create` 、`memory_lease`、`memory_signal_send` |
| 导出到 Obsidian | `memory_obsidian_export` |

### 6.1 REST API：MCP 之外的稳定接口

如果你的 Agent 不支持 MCP，也可以直接调 REST API。默认端口是 `3111`。

健康检查：

```
curl http://localhost:3111/agentmemory/health
```

写入一条观察：

```
curl -X POST http://localhost:3111/agentmemory/observe \  
  -H 'Content-Type: application/json' \  
  -d '{  
    "hookType": "post_tool_use",  
    "sessionId": "demo-session",  
    "project": "/path/to/project",  
    "cwd": "/path/to/project",  
    "timestamp": "2026-05-19T00:00:00.000Z",  
    "data": {  
      "tool_name": "Read",  
      "tool_input": {"file_path": "src/auth.ts"},  
      "tool_output": "read auth middleware"  
    }  
  }'
```

搜索：

```
curl -X POST http://localhost:3111/agentmemory/smart-search \  
  -H 'Content-Type: application/json' \  
  -d '{"query": "auth middleware jwt", "limit": 10}'
```

显式保存记忆：

```
curl -X POST http://localhost:3111/agentmemory/remember \  
  -H 'Content-Type: application/json' \  
  -d '{  
    "content": "This repo uses jose middleware for JWT auth because it is Edge-compatible.",  
    "type": "architecture",  
    "concepts": ["jwt", "jose", "edge-runtime"],  
    "files": ["src/middleware/auth.ts"]  
  }'
```

如果设置了：

```
AGENTMEMORY_SECRET=your-secret
```

就要加：

```
-H "Authorization: Bearer your-secret"
```

### 6.2 Codex 里怎么用

有两种用法：MCP-only 和 full plugin。

MCP-only 更轻，只让 Codex 能调用 memory tools：

```
codex mcp add agentmemory -- npx -y @agentmemory/mcp
```

或手动写进 `~/.codex/config.toml`：

```
[mcp_servers.agentmemory]  
command = "npx"  
args = ["-y", "@agentmemory/mcp"]  
  
[mcp_servers.agentmemory.env]  
AGENTMEMORY_URL = "http://localhost:3111"
```

full plugin 会注册 MCP + hooks + skills：

```
codex plugin marketplace add rohitg00/agentmemory  
codex plugin install agentmemory
```

插件里的 Codex hook 会捕获：

```
SessionStart  
UserPromptSubmit  
PreToolUse  
PostToolUse  
PreCompact  
Stop
```

插件还带了几个可直接触发的技能：

| Skill | 什么时候用 |
| --- | --- |
| `/recall` | 搜索过去的观察、会话和经验 |
| `/remember` | 显式保存一个决策、偏好或实现经验 |
| `/session-history` | 查看当前项目最近会话 |
| `/forget` | 删除指定记忆或敏感内容 |
| `/recap` | 按时间窗口汇总最近会话 |
| `/handoff` | 接续最近一次工作 |
| `/commit-context` | 从代码行/文件追到对应 agent session |
| `/commit-history` | 查看 agent 产出的 commit 记录 |

如果只配置了 MCP，没有安装 full plugin，也能手动调用 MCP 工具，但不会自动捕获 Codex 生命周期事件。

## 七、推荐配置：先保守，再增强

第一次使用，不要一上来打开所有功能。建议从这个最小配置开始：

```
npm install -g @agentmemory/agentmemory  
agentmemory init  
agentmemory  
agentmemory connect codex  
agentmemory doctor
```

`~/.agentmemory/.env` 先只放必要项：

```
AGENTMEMORY_SECRET=change-me-if-exposing-beyond-localhost  
AGENTMEMORY_URL=http://localhost:3111  
AGENTMEMORY_TOOLS=core
```

如果你想提高检索质量，再加 embedding：

```
EMBEDDING_PROVIDER=local
```

如果你确认能接受更多工具暴露，再打开：

```
AGENTMEMORY_TOOLS=all
```

如果你想让模型真的在会话开始时拿到历史上下文，再打开：

```
AGENTMEMORY_INJECT_CONTEXT=true
```

如果你想让每条 observation 都有 LLM 生成的高质量摘要，再打开：

```
AGENTMEMORY_AUTO_COMPRESS=true  
ANTHROPIC_API_KEY=...
```

但这两个开关都要谨慎。一个会增加模型输入 token，一个会增加 LLM 调用成本。

更完整的功能可以按需打开：

```
CONSOLIDATION_ENABLED=true  
GRAPH_EXTRACTION_ENABLED=true  
AGENTMEMORY_REFLECT=true  
OBSIDIAN_AUTO_EXPORT=true
```

这些功能通常需要 LLM provider，适合你已经确认基础捕获和检索稳定之后再开。

## 八、Viewer：它不只是 Dashboard

AgentMemory 默认会启动 viewer，端口是 REST port + 2，也就是：

```
http://localhost:3113
```

viewer 可以看：

* live observation stream
* session explorer
* memory browser
* knowledge graph
* replay timeline
* health dashboard

这对排查很有用。比如你问“为什么 Codex 没有记住刚才的操作”，不要先猜 prompt，先看 viewer 里有没有 observation：

```
flowchart TD  
  A[记忆没生效] --> B{viewer 有 session 吗}  
  B -->|没有| C[hook/plugin 没生效]  
  B -->|有| D{observationCount 增长吗}  
  D -->|没有| E[observe API 或 hook payload 问题]  
  D -->|有| F{smart-search 能搜到吗}  
  F -->|不能| G[index / embedding / query 问题]  
  F -->|能| H{上下文注入了吗}  
  H -->|没有| I[AGENTMEMORY_INJECT_CONTEXT 默认关闭]  
  H -->|有| J[检查 token budget 和排序]
```

这比在 Agent 对话里问“你记得吗”可靠得多。

## 九、同类很多时，AgentMemory 的优势到底是什么

如果只看“基于 hook 捕获会话”，AgentMemory 没有不可替代性。现在很多项目都能做这件事：ClawMem 能通过 Claude Code hooks、MCP、OpenClaw、Hermes 写入同一个本地 vault；Engram 用 Go binary、SQLite/FTS5、HTTP API、MCP 和 Claude plugin 做 session tracking；claude-memory-compiler 走另一条路线，用 Claude Code hooks 捕获 transcript，再编译成 markdown 知识库。

所以问题不是“谁能捕获 hook”。真正要比较的是：捕获之后，系统能不能把这些事件变成长期可用、可检索、可观察、可删除、可跨 Agent 复用的记忆。

AgentMemory 的优势主要在这几个点。

第一，它不是只存日志，而是完整 pipeline。hook 进来后，会走去重、隐私过滤、raw observation、synthetic/LLM compression、BM25/vector/graph index、MCP/REST 查询、viewer 展示、审计删除。很多项目做到的是“捕获 -> 写 markdown/SQLite -> 搜索”，AgentMemory 做得更像一个本地 memory runtime。

第二，它默认成本更克制。`AGENTMEMORY_AUTO_COMPRESS` 默认关闭，`AGENTMEMORY_INJECT_CONTEXT` 也默认关闭。默认状态下，它更像后台记录器，而不是一个会悄悄把每次工具调用都送去 LLM 压缩、或把大量历史内容塞回模型上下文的系统。这个取舍对长期使用很关键。

第三，它的检索不是纯向量搜索。AgentMemory 使用 BM25 + vector + graph，再用 RRF 融合，并做 session diversification。代码场景里，文件名、函数名、错误码、命令、commit SHA 这类精确词非常重要，纯向量检索容易把这些细节稀释掉。BM25 在这里不是落后方案，而是必要信号。

第四，它的接口面更完整。AgentMemory 同时有 Codex plugin、Claude plugin、MCP shim、REST API、viewer 和 plugin skills。对个人来说这可能显得重；但对混用 Codex、Claude Code、Cursor、Gemini CLI、OpenCode 的团队来说，“同一套本地 memory server 被不同 Agent 访问”是实用价值。

第五，它更重视可观察性和治理。viewer、health、diagnostics、audit、governance delete、export、snapshot、team、lease、signals、actions 这些能力，对简单个人记忆不一定需要，但对团队协作、多 Agent 运行、故障排查很有用。尤其是“它到底有没有记住”这个问题，能在 viewer 和 API 里看到证据，比靠模型自述可靠。

可以这样对比：

| 项目 | 更像什么 | AgentMemory 的相对优势 |
| --- | --- | --- |
| claude-memory-compiler | Markdown 知识库编译器 | AgentMemory 更实时，有 MCP/REST/viewer，不依赖事后编译成文章 |
| ClawMem | 本地 SQLite/RAG 记忆层 | AgentMemory 的 REST、MCP、viewer、audit、协作工具面更大 |
| Engram | Go binary + SQLite/FTS5 + MCP/TUI | AgentMemory 的 hybrid search、graph、consolidation、插件面更完整 |
| Mem0 / Letta | 通用 agent memory 平台 | AgentMemory 更贴近 coding agent hooks、本地 CLI 和跨工具开发工作流 |

但优势不是免费的。AgentMemory 更复杂，要跑 `iii-engine`，要理解 MCP proxy/full server、plugin、hooks、端口和配置；`iii-engine` 的 engine 部分也不是 Apache/MIT，而是 ELv2。轻量个人使用时，markdown 或 SQLite 型方案可能更省心。

我的判断是：如果目标只是“把 Claude Code 的会话存起来”，AgentMemory 未必是最简单的选择；如果目标是“给多个 coding agent 配一个可检索、可观察、可治理的本地长期记忆运行时”，AgentMemory 的优势会明显很多。

对我们这种 Codex/Claude/Cursor 混用、还希望沉淀团队配置文档和可复用插件的场景，我最看重它三点：

```
1. Codex plugin + MCP + REST 都有  
2. 默认不烧 token，注入和 LLM 压缩都要显式打开  
3. viewer / audit / diagnostics 方便排查“到底有没有记住”
```

## 十、什么时候适合用 AgentMemory

适合：

* 你每天在同几个代码仓库里反复工作。
* 你经常需要解释项目结构、测试命令、部署流程、历史决策。
* 你想让 Codex、Claude Code、Cursor、Gemini CLI 共享同一套本地记忆。
* 你需要能审计、删除、导出、回放过去会话。
* 你愿意让一个本地后台服务长期运行。

不适合：

* 你只偶尔用一次 Agent，不需要跨会话记忆。
* 你的工作内容高度敏感，但没有时间做本地隔离和 secret 配置。
* 你希望“装上就自动把所有上下文注入模型”，但又不想承担 token 成本。
* 你不想引入 `iii-engine` 这个额外运行时。
* 你只想要一个简单 markdown memory 文件。

## 十一、使用时最容易踩的坑

### 11.1 只启动 MCP，没有启动 server

现象：MCP 里只有少数工具，或者搜索结果很少。

原因：`@agentmemory/mcp` 没连上 `http://localhost:3111`，退化到本地 fallback。

处理：

```
agentmemory  
curl http://localhost:3111/agentmemory/livez
```

如果 MCP client 在 sandbox 里访问不到宿主机 localhost，需要设置：

```
AGENTMEMORY_FORCE_PROXY=1  
AGENTMEMORY_URL=http://<reachable-host>:3111
```

### 11.2 以为默认会自动注入上下文

默认不会。默认只是后台捕获和索引。

要注入：

```
AGENTMEMORY_INJECT_CONTEXT=true
```

但要知道这会增加模型输入 token。

### 11.3 以为没有 API key 就完全不能用

也不是。没有 LLM key 时，provider 是 `noop`，LLM 总结、反思、consolidation 会关闭，但 observation 捕获、synthetic compression、BM25 搜索仍然能工作。

### 11.4 打开自动压缩后 token 消耗突然上升

`AGENTMEMORY_AUTO_COMPRESS=true` 会让每条 observation 走 LLM 压缩。活跃编码会话里 tool call 很多，这个成本不是线性的“小开销”，而是会快速累积。

### 11.5 README 里工具数量和源码不完全一致

当前 `0.9.20` 源码里 `tools-registry.ts` 定义了 53 个 `memory_*` 工具。部分 README 文案写 51 或 53，属于文档同步问题。排查时以运行时 `tools/list` 和源码为准。

## 十二、我对它的判断

AgentMemory 的价值不在“它能存向量”，而在它把 Agent 长期记忆里最容易漏掉的工程问题都纳入了范围：

* 什么时候捕获？
* 捕获失败会不会影响主流程？
* 怎么避免重复写？
* 怎么过滤秘密？
* 没有 LLM key 时能不能降级？
* 怎么把 recall 暴露给不同 Agent？
* 怎么看见系统到底记了什么？
* 怎么删除和审计？

这让它更像一个 Agent 运行时组件，而不是一个工具函数库。

但它的代价也很明确：你需要接受一个本地 daemon、一个 iii-engine 运行时、一组 hook 脚本和一套较复杂的配置面。对重度 Agent 用户来说，这个代价可能值得；对轻量用户来说，可能太重。

我的建议是：

先把它当作“会话黑盒记录器”使用，不要一开始就开启注入和自动 LLM 压缩。等你通过 viewer 和 `memory_smart_search` 确认它真的记录到了有用信息，再逐步打开 `AGENTMEMORY_TOOLS=all`、embedding、context injection、consolidation 和 graph。

长期看，Agent 记忆系统最重要的不是“记得更多”，而是“在正确的时候想起正确的东西，并且可以被人审计和删除”。AgentMemory 的源码实现，基本就是围绕这个目标在做工程化。

## 十三、资料来源

* npm package：@agentmemory/agentmemory
* GitHub repository：rohitg00/agentmemory
* iii repository：iii-hq/iii
* iii Architecture docs：iii.dev/docs/architecture
* iii Quickstart：iii.dev/docs/quickstart
* ClawMem：yoloshii/ClawMem
* claude-memory-compiler：coleam00/claude-memory-compiler
* Engram：syntax-syndicate/engram-agent-memory
* 源码重点文件：

+ `src/cli.ts`
+ `src/index.ts`
+ `src/functions/observe.ts`
+ `src/functions/compress.ts`
+ `src/functions/context.ts`
+ `src/functions/search.ts`
+ `src/state/hybrid-search.ts`
+ `src/state/search-index.ts`
+ `src/state/vector-index.ts`
+ `src/mcp/standalone.ts`
+ `src/mcp/rest-proxy.ts`
+ `src/mcp/tools-registry.ts`
+ `src/hooks/*.ts`
+ `plugin/.codex-plugin/plugin.json`
+ `plugin/hooks/hooks.codex.json`