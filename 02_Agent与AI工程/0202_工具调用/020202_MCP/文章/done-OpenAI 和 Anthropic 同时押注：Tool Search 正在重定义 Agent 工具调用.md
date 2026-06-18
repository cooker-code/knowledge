> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP与CLI代码执行边界|MCP与CLI代码执行边界]]、[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: OpenAI 和 Anthropic 同时押注：Tool Search 正在重定义 Agent 工具调用
author: 歪脖抠腚
date:
url: https://mp.weixin.qq.com/s?__biz=MzY0MDAzNzk3MA==&mid=2247484038&idx=1&sn=03425bf7f5fc29f63d08bfa85295244e&chksm=f1b3f717999ce5253f50f9bba112464d46c632c1323bb5f15bec7ec183718150c613c8d4dde6&mpshare=1&scene=24&srcid=04153ORpVQN4pxdPWXT6Iqxs&sharer_shareinfo=4c3c726f46c100b132f92d0e3ab60efa&sharer_shareinfo_first=4c3c726f46c100b132f92d0e3ab60efa#rd
---

> Tool Search（工具搜索）是 2025-2026 年 AI Agent 基础设施领域最重要的架构创新之一。它从根本上改变了大模型与外部工具的交互方式——从"预加载所有工具定义"转向"按需发现、动态加载"。本文基于 OpenAI GPT-5.4、Anthropic Claude、Spring AI 等平台的官方文档和技术博客进行深度调研。

## 核心发现速览

| 维度 | 关键信息 |
| --- | --- |
| **本质** | 工具的"懒加载"机制——模型只加载当前任务需要的工具，而非全量预载 |
| **解决的问题** | 上下文膨胀（Context Bloat）、工具选择准确率下降、Token 成本爆炸 |
| **Token 节省** | 85%+（Anthropic）/ 34-64%（Spring AI 跨平台基准） |
| **准确率提升** | Claude Opus 4: 49% → 74%；Claude Opus 4.5: 79.5% → 88.1% |
| **适用场景** | 10+ 工具、多 MCP 服务器、工具定义超过 10K tokens |
| **主要实现** | OpenAI `tool_search`（GPT-5.4）、Anthropic `tool_search_tool`（Claude Sonnet 4+）、Spring AI `ToolSearchToolCallAdvisor` |
| **核心思想** | Just-in-Time Retrieval（即时检索）——上下文工程的重要原则 |

---

## 一、什么是 Tool Search

### 1.1 Tool Search  定义

**Tool Search 是一种让 AI 模型按需发现和加载工具的机制**。模型初始只持有一个轻量的"搜索工具"，当需要特定能力时，通过搜索查询找到相关工具，再将其完整定义动态注入上下文。

这类似于编程中的懒加载（Lazy Loading）或操作系统中的按需分页（Demand Paging）——资源不在启动时全部加载，而是在首次访问时才载入内存。

### 1.2 传统方式 vs Tool Search

**传统 Tool Calling 流程：**

```
┌─────────────────────────────────────────────────────┐│  系统提示词 + 所有工具定义（一次性全量加载）           ││  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   ││  │ GitHub  │ │  Slack  │ │  Jira   │ │ Sentry  │...││  │ 35 工具 │ │ 11 工具 │ │ 20 工具 │ │  5 工具 │   ││  │ ~26K tok│ │ ~21K tok│ │ ~17K tok│ │  ~3K tok│   ││  └─────────┘ └─────────┘ └─────────┘ └─────────┘   ││                                                      ││  总计：~72K tokens 在对话开始前已被消耗               ││  剩余上下文空间严重受限                               │└─────────────────────────────────────────────────────┘
```

**Tool Search 流程：**

```
┌─────────────────────────────────────────────────────┐│  系统提示词 + Tool Search 工具（~500 tokens）         ││  ┌───────────────┐                                   ││  │ tool_search   │  ← 唯一预加载的工具               ││  │   ~500 tok    │                                   ││  └───────────────┘                                   ││                                                      ││  用户："帮我在 GitHub 创建一个 PR"                    ││  模型 → tool_search("github pull request")           ││        → 发现 github.createPullRequest               ││        → 动态加载该工具定义（~800 tok）               ││        → 调用 github.createPullRequest(...)           ││                                                      ││  总计：~1.3K tokens（vs 传统方式 ~72K）               ││  节省：~98% 的工具定义 Token 开销                     │└─────────────────────────────────────────────────────┘
```

### 1.3 核心设计理念

Tool Search 体现了 **Just-in-Time Retrieval（即时检索）** 原则——这是 Anthropic 在其「Effective Context Engineering」中提出的上下文工程核心方法论。核心思想是：

> 不要把所有可能有用的信息一次性塞入上下文，而是在模型需要时再检索注入。

这与 RAG（检索增强生成）的理念一脉相承，只不过 RAG 检索的是知识文档，Tool Search 检索的是工具定义。

---

## 二、完整实例对比：传统方式 vs Tool Search

以一个真实场景为例：你有 3 个工具——`get_weather`（查天气）、`search_restaurants`（搜餐厅）、`book_reservation`（预订餐厅），用户问"旧金山今天天气怎么样？"。实际只需要 `get_weather`，但传统方式下所有工具定义都会被发送给模型。

### 2.1 传统方式：全量发送

无论使用 Claude 还是 OpenAI，传统方式都是把所有工具定义一次性放入请求：

**Anthropic Claude 传统请求：**

```
{  "model": "claude-sonnet-4-5-20250929",  "max_tokens": 1024,  "messages": [    {"role": "user", "content": "旧金山今天天气怎么样？"}  ],  "tools": [    {      "name": "get_weather",      "description": "获取指定地点的当前天气信息",      "input_schema": {        "type": "object",        "properties": {          "location": {"type": "string", "description": "城市名称"},          "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}        },        "required": ["location"]      }    },    {      "name": "search_restaurants",      "description": "根据位置、菜系、价格范围搜索附近餐厅",      "input_schema": {        "type": "object",        "properties": {          "location": {"type": "string", "description": "城市名称"},          "cuisine": {"type": "string", "description": "菜系类型，如中餐、意大利菜"},          "price_range": {"type": "string", "enum": ["$", "$$", "$$$", "$$$$"]},          "open_now": {"type": "boolean", "description": "是否只显示当前营业的餐厅"}        },        "required": ["location"]      }    },    {      "name": "book_reservation",      "description": "在指定餐厅预订座位",      "input_schema": {        "type": "object",        "properties": {          "restaurant_id": {"type": "string"},          "date": {"type": "string", "description": "日期，格式 YYYY-MM-DD"},          "time": {"type": "string", "description": "时间，格式 HH:MM"},          "party_size": {"type": "integer", "description": "用餐人数"},          "special_requests": {"type": "string", "description": "特殊要求"}        },        "required": ["restaurant_id", "date", "time", "party_size"]      }    }  ]}
```

**模型实际看到的上下文：**

```
┌─────────────────────────────────────────────────────┐│ [系统提示词]                                         ││ [get_weather 完整定义]         ← ~150 tokens        ││ [search_restaurants 完整定义]  ← ~200 tokens  未使用 ││ [book_reservation 完整定义]    ← ~250 tokens  未使用 ││ [用户消息: "旧金山今天天气怎么样？"]                  ││                                                      ││ 工具定义总计: ~600 tokens（3 个工具还好，             ││ 但 50+ 工具时就是 ~55,000 tokens）                   │└─────────────────────────────────────────────────────┘
```

**模型响应 → 直接调用 `get_weather`：**

```
{  "role": "assistant",  "content": [    {"type": "text", "text": "让我查一下旧金山的天气。"},    {      "type": "tool_use",      "id": "toolu_01ABC",      "name": "get_weather",      "input": {"location": "San Francisco", "unit": "celsius"}    }  ]}
```

**问题：** 3 个工具还可以接受。但当工具增至 50-200 个时，每次请求都要发送全部定义，即使只用到 1 个。

### 2.2 关键概念：HTTP 请求体 ≠ 模型上下文

在看 Tool Search 的例子之前，必须先理解一个关键区别：

```
┌──────────────────────────────────────────────────────────────────┐│                        你发给 API 的 HTTP 请求                    ││                                                                  ││  tools: [                                                        ││    tool_search_tool,           ──┐                               ││    get_weather (defer:true),     │  API 服务器收到所有定义        ││    search_restaurants (defer:true),│  但它不会把 defer=true 的    ││    book_reservation (defer:true) │  工具注入模型上下文            ││  ]                             ──┘                               ││                                                                  ││         ┌──── API 服务器在这里做了一次"分拣" ────┐               ││         │                                        │               ││         ▼                                        ▼               ││  ┌─────────────────┐              ┌─────────────────────────┐   ││  │ 模型上下文窗口   │              │ 工具注册表（模型看不到）  │   ││  │                 │              │                         │   ││  │ tool_search_tool│              │ get_weather 完整定义     │   ││  │ （~500 tok）    │              │ search_restaurants 完整  │   ││  │                 │              │ book_reservation 完整    │   ││  │ 用户消息        │              │                         │   ││  └─────────────────┘              └─────────────────────────┘   ││   模型只看到这部分                  搜索时才从这里提取            ││   按这部分计费                                                   │└──────────────────────────────────────────────────────────────────┘
```

**核心原理：`defer_loading: true` 是给 API 服务器的指令，不是给模型的。**

•你在 HTTP 请求的 `tools` 数组里传了所有工具的完整定义——这是给 API 服务器的"工具目录"•API 服务器收到后，把 `defer_loading: true` 的工具**拦截**下来，放入一个内部索引•模型的上下文窗口里**只注入**非 defer 的工具（即 tool\_search\_tool 本身）•当模型调用 tool\_search 搜索并返回 `tool_reference` 时，API 服务器才从索引中取出对应工具的完整定义，注入到模型上下文的**末尾**•**你按 input tokens 付费，计费基准是模型上下文中的 tokens，不是 HTTP 请求体的大小**

这就是为什么 3 个工具时差异不大（反正一共才 ~~600 tokens），但 200 个工具时差异巨大（~~55,000 tokens 的完整定义 vs ~500 tokens 的搜索工具）。

> OpenAI 的实现有一个额外细节：对于单个 defer 的函数（非 Namespace），模型仍然能看到函数名和描述，只是参数 schema 被延迟了。所以 OpenAI 推荐用 Namespace 分组——模型只看到 Namespace 的名称和描述，内部函数完全不可见，Token 节省更显著。

### 2.3 Anthropic Claude Tool Search 方式

Claude 提供了两种内置搜索变体，以及支持自定义搜索实现。以同样的 3 个工具为例：

### 变体 A：BM25 自然语言搜索（推荐通用场景）

**你发送的 HTTP 请求（包含所有工具的完整定义，作为"目录"交给 API 服务器）：**

```
{  "model": "claude-opus-4-6",  "max_tokens": 1024,  "messages": [    {"role": "user", "content": "旧金山今天天气怎么样？"}  ],  "tools": [    {      "type": "tool_search_tool_bm25_20251119",      "name": "tool_search_tool_bm25"    },    {      "name": "get_weather",      "description": "获取指定地点的当前天气信息",      "input_schema": {        "type": "object",        "properties": {          "location": {"type": "string", "description": "城市名称"},          "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}        },        "required": ["location"]      },      "defer_loading": true    },    {      "name": "search_restaurants",      "description": "根据位置、菜系、价格范围搜索附近餐厅",      "input_schema": {        "type": "object",        "properties": {          "location": {"type": "string"},          "cuisine": {"type": "string"},          "price_range": {"type": "string", "enum": ["$", "$$", "$$$", "$$$$"]},          "open_now": {"type": "boolean"}        },        "required": ["location"]      },      "defer_loading": true    },    {      "name": "book_reservation",      "description": "在指定餐厅预订座位",      "input_schema": {        "type": "object",        "properties": {          "restaurant_id": {"type": "string"},          "date": {"type": "string"},          "time": {"type": "string"},          "party_size": {"type": "integer"},          "special_requests": {"type": "string"}        },        "required": ["restaurant_id", "date", "time", "party_size"]      },      "defer_loading": true    }  ]}
```

**但模型实际看到的上下文完全不同——API 服务器做了"分拣"：**

```
┌─────────────────────────────────────────────────────┐│  模型上下文窗口（模型实际看到的，也是计费依据）       ││                                                      ││  [系统提示词]                                        ││  [tool_search_tool_bm25 定义]   ← ~500 tokens      ││                                                      ││  [用户消息: "旧金山今天天气怎么样？"]                 ││                                                      ││  ❌ get_weather       — defer=true，不在这里         ││  ❌ search_restaurants — defer=true，不在这里        ││  ❌ book_reservation  — defer=true，不在这里         ││                                                      ││  工具定义总计: ~500 tokens                           ││  （传统方式: ~600 tokens，差距不大因为只有 3 个工具） ││  （200 个工具时: ~500 vs ~55,000，差距 100 倍）      │└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐│  API 服务器内部的工具索引（模型看不到，不计费）       ││                                                      ││  get_weather:        完整 input_schema...            ││  search_restaurants: 完整 input_schema...            ││  book_reservation:   完整 input_schema...            ││                                                      ││  等待被 tool_search 查询时才注入模型上下文            │└─────────────────────────────────────────────────────┘
```

**模型响应 — 先搜索、再调用：**

```
{  "role": "assistant",  "content": [    {      "type": "text",      "text": "让我搜索天气相关的工具。"    },    {      "type": "server_tool_use",      "id": "srvtoolu_01XYZ",      "name": "tool_search_tool_bm25",      "input": {        "query": "weather forecast current conditions"      }    },    {      "type": "tool_search_tool_result",      "tool_use_id": "srvtoolu_01XYZ",      "content": {        "type": "tool_search_tool_search_result",        "tool_references": [          {"type": "tool_reference", "tool_name": "get_weather"}        ]      }    },    {      "type": "text",      "text": "找到了天气工具，让我查询旧金山的天气。"    },    {      "type": "tool_use",      "id": "toolu_01ABC",      "name": "get_weather",      "input": {"location": "San Francisco", "unit": "celsius"}    }  ],  "stop_reason": "tool_use"}
```

**搜索命中后，模型上下文发生了变化——`get_weather` 的完整定义被注入末尾：**

```
┌─────────────────────────────────────────────────────┐│  模型上下文窗口（搜索后）                             ││                                                      ││  [系统提示词]                         ← 缓存命中     ││  [tool_search_tool_bm25 定义]         ← 缓存命中     ││  [用户消息]                           ← 缓存命中     ││  [模型: "让我搜索天气相关的工具"]      ← 新内容       ││  [tool_search → tool_reference: get_weather]         ││  [模型: "找到了天气工具..."]                          ││                                                      ││  ┌───────────────────────────────────────────────┐   ││  │ get_weather 完整定义（此时才注入到末尾！）      │   ││  │ name: "get_weather"                            │   ││  │ description: "获取指定地点的当前天气信息"       │   ││  │ input_schema: {location, unit, ...}            │   ││  └───────────────────────────────────────────────┘   ││                                                      ││  ❌ search_restaurants — 仍然不在上下文中             ││  ❌ book_reservation  — 仍然不在上下文中              ││                                                      ││  模型现在可以调用 get_weather（因为看到了完整定义）    ││  但仍然看不到其他两个工具                             │└─────────────────────────────────────────────────────┘
```

`tool_reference` 的展开是 API 服务器自动完成的——它从内部索引中取出 `get_weather` 的完整 `input_schema`，注入到模型上下文的末尾。开发者不需要手动处理展开逻辑。

### 变体 B：Regex 正则搜索（适合工具命名规范明确的场景）

唯一区别是声明搜索工具的类型和模型使用的查询方式：

```
{  "type": "tool_search_tool_regex_20251119",  "name": "tool_search_tool_regex"}
```

模型会用 Python 正则表达式搜索，而非自然语言：

```
{  "type": "server_tool_use",  "name": "tool_search_tool_regex",  "input": {    "query": "(?i)weather"  }}
```

常见正则模式：

•`"(?i)weather"` — 大小写不敏感搜索•`"get_.*_data"` — 匹配 `get_user_data`、`get_weather_data` 等•`"restaurant|booking|reservation"` — OR 多关键词

### 变体 C：自定义客户端搜索（适合需要语义嵌入等高级搜索的场景）

你可以实现自己的搜索逻辑（如向量嵌入搜索），只需返回 `tool_reference` 块：

```
{  "tools": [    {      "name": "my_custom_tool_search",      "description": "搜索可用工具",      "input_schema": {        "type": "object",        "properties": {          "query": {"type": "string"}        },        "required": ["query"]      }    },    {      "name": "get_weather",      "description": "获取天气",      "input_schema": {"type": "object", "properties": {...}},      "defer_loading": true    }  ]}
```

当模型调用你的自定义搜索工具时，你的应用执行搜索后返回：

```
{  "type": "tool_result",  "tool_use_id": "toolu_custom_search_id",  "content": [    {"type": "tool_reference", "tool_name": "get_weather"}  ]}
```

API 会自动将 `tool_reference` 中引用的工具展开为完整定义。

### 2.3 OpenAI GPT-5.4 Tool Search 方式

OpenAI 同样提供了两种模式，但设计理念有显著差异——强调 **Namespace（命名空间）** 分组。

### 模式 A：Hosted Tool Search（托管搜索，最简单）

```
{  "model": "gpt-5.4",  "input": [    {"role": "user", "content": "旧金山今天天气怎么样？"}  ],  "tools": [    {"type": "tool_search"},    {      "type": "function",      "name": "get_weather",      "description": "获取指定地点的当前天气信息",      "parameters": {        "type": "object",        "properties": {          "location": {"type": "string"},          "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}        },        "required": ["location"]      },      "defer_loading": true    },    {      "type": "function",      "name": "search_restaurants",      "description": "根据位置、菜系搜索附近餐厅",      "parameters": {        "type": "object",        "properties": {          "location": {"type": "string"},          "cuisine": {"type": "string"},          "price_range": {"type": "string", "enum": ["$", "$$", "$$$", "$$$$"]}        },        "required": ["location"]      },      "defer_loading": true    },    {      "type": "function",      "name": "book_reservation",      "description": "在指定餐厅预订座位",      "parameters": {        "type": "object",        "properties": {          "restaurant_id": {"type": "string"},          "date": {"type": "string"},          "time": {"type": "string"},          "party_size": {"type": "integer"}        },        "required": ["restaurant_id", "date", "time", "party_size"]      },      "defer_loading": true    }  ]}
```

**响应（搜索 + 加载 + 调用在一次响应中完成）：**

```
{  "output": [    {      "type": "tool_search_call",      "id": "ts_01ABC",      "execution": "server",      "call_id": null    },    {      "type": "tool_search_output",      "id": "tso_01ABC",      "tools": ["get_weather"]    },    {      "type": "function_call",      "id": "fc_01XYZ",      "call_id": "call_01XYZ",      "name": "get_weather",      "arguments": "{\"location\": \"San Francisco\", \"unit\": \"celsius\"}"    }  ]}
```

注意 Hosted 模式下 `execution: "server"` 且 `call_id: null`——搜索在 OpenAI 服务端完成，开发者只需处理最终的 `function_call`。

### 模式 B：Client-executed Tool Search（客户端搜索）

当工具发现依赖于项目状态、租户配置等外部系统时使用：

```
{  "model": "gpt-5.4",  "input": [    {"role": "user", "content": "旧金山今天天气怎么样？"}  ],  "tools": [    {      "type": "tool_search",      "execution": "client",      "input_schema": {        "type": "object",        "properties": {          "query": {"type": "string", "description": "搜索关键词"}        },        "required": ["query"]      }    },    {      "type": "function",      "name": "get_weather",      "description": "获取天气",      "parameters": {...},      "defer_loading": true    }  ]}
```

**Turn 1 — 模型发出搜索请求并停止：**

```
{  "output": [    {      "type": "tool_search_call",      "id": "ts_01ABC",      "execution": "client",      "call_id": "call_search_01",      "arguments": "{\"query\": \"weather\"}"    }  ],  "status": "incomplete"}
```

**你的应用执行搜索后，返回结果继续对话：**

```
{  "input": [    {      "type": "tool_search_output",      "call_id": "call_search_01",      "tools": [        {          "type": "function",          "name": "get_weather",          "description": "获取指定地点的当前天气信息",          "parameters": {            "type": "object",            "properties": {              "location": {"type": "string"},              "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}            },            "required": ["location"]          }        }      ]    }  ]}
```

**Turn 2 — 模型调用已加载的工具：**

```
{  "output": [    {      "type": "function_call",      "name": "get_weather",      "arguments": "{\"location\": \"San Francisco\"}"    }  ]}
```

### OpenAI Namespace 方式（推荐用于大量工具）

OpenAI 推荐用 Namespace 将工具分组，模型先看到分组描述，按需加载整个分组：

```
{  "model": "gpt-5.4",  "tools": [    {"type": "tool_search"},    {      "type": "namespace",      "name": "dining",      "description": "餐饮相关工具：搜索餐厅、查看菜单、预订座位。",      "tools": [        {          "type": "function",          "name": "search_restaurants",          "description": "搜索附近餐厅",          "defer_loading": true,          "parameters": {...}        },        {          "type": "function",          "name": "book_reservation",          "description": "预订餐厅座位",          "defer_loading": true,          "parameters": {...}        }      ]    },    {      "type": "namespace",      "name": "weather",      "description": "天气和环境相关工具：查询天气、空气质量、紫外线指数。",      "tools": [        {          "type": "function",          "name": "get_weather",          "description": "获取指定地点的天气",          "defer_loading": true,          "parameters": {...}        }      ]    }  ]}
```

**模型初始看到的内容：**

```
┌─────────────────────────────────────────────────────┐│ [tool_search 工具]                                   ││ [namespace: dining — "餐饮相关工具：搜索餐厅..."]     ││ [namespace: weather — "天气和环境相关工具..."]         ││                                                      ││ ⚠️ 模型只看到 Namespace 名称和描述                   ││    看不到里面各函数的参数 schema                      ││    直到 tool_search 加载某个 Namespace                │└─────────────────────────────────────────────────────┘
```

当模型判断需要天气能力时，tool search 加载 `weather` namespace，其中 `get_weather` 的完整参数定义才进入上下文。

### 2.4 各平台 Tool Search 配置方式对比

| 配置维度 | Anthropic Claude | OpenAI GPT-5.4 |
| --- | --- | --- |
| **搜索工具声明** | `{"type": "tool_search_tool_bm25_20251119"}` 或 `{"type": "tool_search_tool_regex_20251119"}` | `{"type": "tool_search"}` |
| **延迟加载标记** | 工具定义中加 `"defer_loading": true` | 工具定义中加 `"defer_loading": true` |
| **搜索执行方** | 服务端（BM25/Regex），也支持客户端自定义 | 服务端（Hosted）或客户端（`"execution": "client"`） |
| **搜索查询方式** | BM25: 自然语言 / Regex: Python 正则 | Hosted: 模型自动决定 / Client: 开发者自定义 schema |
| **工具分组** | MCP Toolset (`"type": "mcp_toolset"`) | Namespace (`"type": "namespace"`) |
| **分组级延迟** | `"default_config": {"defer_loading": true}` 整个 MCP 服务器延迟 | Namespace 内各工具独立设置 `defer_loading` |
| **分组内例外** | `"configs": {"常用工具名": {"defer_loading": false}}` 单个工具例外 | 同一 Namespace 内可混合 defer/非 defer |
| **搜索范围** | 工具名、描述、参数名、参数描述 | 工具名、描述（Namespace 时主要看 Namespace 描述） |
| **结果格式** | `tool_search_tool_result` → `tool_reference` 自动展开 | `tool_search_call` + `tool_search_output` → `function_call` |
| **自定义搜索** | 返回 `tool_reference` 块的任意工具 | `"execution": "client"` + 自定义 `input_schema` |
| **搜索结果数** | 3-5 个最相关工具 | 未公开限制，可加载多个 Namespace |
| **最大工具数** | 10,000 | 未公开 |
| **模型要求** | Sonnet 4.0+ / Opus 4.0+（不支持 Haiku） | GPT-5.4 |

### 2.5 关键设计差异总结

**Anthropic 的设计哲学：搜索是一个"特殊工具"**

•搜索工具有明确的类型标识（`tool_search_tool_bm25_20251119`），是一个服务端执行的特殊工具（`server_tool_use`）•搜索结果通过 `tool_reference` 引用机制自动展开，开发者无需手动注入工具定义•提供 BM25 和 Regex 两种搜索算法让开发者选择•也可以用任何普通工具作为自定义搜索，只要返回 `tool_reference` 即可

**OpenAI 的设计哲学：搜索是一个"系统能力"**

•搜索工具声明极简（`{"type": "tool_search"}`），作为系统内置能力•更强调 Namespace 分组——模型训练时主要学习了在 Namespace 级别搜索•对于单个 defer 函数，模型仍能看到函数名和描述，实际只延迟了参数 schema•区分 Hosted（OpenAI 服务端搜索）和 Client-executed（开发者应用搜索）两种模式•Client-executed 模式允许完全自定义搜索参数和返回的工具集

**核心理念一致，实现路径不同：**

```
Anthropic: 工具级粒度 → BM25/Regex 搜索 → tool_reference 自动展开OpenAI:    Namespace 级粒度 → Hosted/Client 搜索 → tool_search_output 手动/自动注入
```

---

## 三、Tool Search 解决什么问题

### 3.1 问题一：上下文膨胀（Context Bloat）

随着 MCP（Model Context Protocol）生态的爆发，一个 AI Agent 连接的工具数量快速增长。真实场景中的 Token 消耗：

| MCP 服务器 | 工具数量 | Token 消耗 |
| --- | --- | --- |
| GitHub | 35 | ~26,000 |
| Slack | 11 | ~21,000 |
| Jira | 20 | ~17,000 |
| Sentry | 5 | ~3,000 |
| Grafana | 5 | ~3,000 |
| Splunk | 2 | ~2,000 |
| **合计** | **78** | **~72,000** |

下图是 Claude Code 中执行 `/context` 命令的真实截图，可以直观看到 **System tools 占据了 16.6k tokens（8.3%）**——而这还只是一个工具数量适中的配置：

Anthropic 内部测试中，工具定义最多曾消耗 **134,000 tokens**。一个 Docker MCP 服务器的 135 个工具定义就消耗了约 125,000 tokens。

这意味着在 200K 上下文窗口中，**超过三分之一的空间在对话开始前就被工具定义占据**，严重压缩了对话历史、系统提示词和实际推理的空间。

### 3.2 问题二：工具选择准确率下降

实验数据表明，当可用工具超过 30-50 个时，模型的工具选择准确率会**显著下降**。尤其是当工具名称相似时（如 `notification-send-user` vs `notification-send-channel`），模型容易混淆和误选。

这是一个信息过载问题——就像让一个人从 100 本书中挑选最相关的一本，和从 5 本中挑选，后者的准确率显然更高。

Anthropic 的内部评测数据：

| 模型 | 无 Tool Search | 有 Tool Search | 提升 |
| --- | --- | --- | --- |
| Claude Opus 4 | 49% | 74% | +25pp |
| Claude Opus 4.5 | 79.5% | 88.1% | +8.6pp |

### 3.3 问题三：Token 成本爆炸

每次 API 请求都需要携带完整的工具定义。即使用户只是问一个简单问题，所有工具定义的 Token 费用仍然照付。

以一个 5 服务器、78 个工具的配置为例：

•每次请求的工具定义开销：~72,000 tokens•假设每天 10,000 次 API 调用•仅工具定义就消耗 **7.2 亿 tokens/天**•按 Claude Sonnet 4 的输入价格（$3/M tokens），每天仅工具定义成本约 **$2,160**

使用 Tool Search 后，每次请求的工具定义开销降至 ~3,000-5,000 tokens（搜索工具 + 3-5 个按需加载的工具），**成本降低 85-95%**。

### 3.4 问题四：Prompt Caching 失效

传统方式下，如果不同请求需要不同的工具子集，工具定义的变化会破坏 Prompt Cache，导致每次请求都需要重新处理完整的工具列表。

Tool Search 的设计巧妙地解决了这个问题：延迟加载的工具被完全排除在初始 prompt 之外，只有 Tool Search 工具本身和少量核心工具被包含。这使得系统提示词和核心工具定义可以稳定地被缓存。在 OpenAI 的实现中，新发现的工具被注入到上下文窗口的**末尾**，进一步保护了前序内容的缓存。

---

## 四、各平台实现对比

### 4.1 Anthropic Claude：Tool Search Tool

**发布时间：** 2025 年 11 月 24 日（Beta），持续迭代至今

**支持模型：** Claude Sonnet 4.0+、Claude Opus 4.0+（不支持 Haiku）

**两种搜索变体：**

| 变体 | 类型标识 | 查询方式 | 适用场景 |
| --- | --- | --- | --- |
| **Regex** | `tool_search_tool_regex_20251119` | Python 正则表达式 | 精确模式匹配，已知工具命名规范 |
| **BM25** | `tool_search_tool_bm25_20251119` | 自然语言查询 | 模糊搜索，语义匹配 |

**核心 API 设计：**

```
{  "tools": [    // 1. 声明搜索工具（始终加载）    {"type": "tool_search_tool_bm25_20251119", "name": "tool_search_tool_bm25"},
    // 2. 标记延迟加载的工具    {      "name": "github.createPullRequest",      "description": "Create a pull request in a GitHub repository",      "input_schema": {...},      "defer_loading": true   // 关键标记：不在初始上下文中加载    },
    // 3. 保留高频工具立即加载    {      "name": "search_files",      "description": "Search files in workspace",      "input_schema": {...}      // 无 defer_loading → 立即可用    }  ]}
```

**MCP 服务器级别的延迟加载：**

```
{  "type": "mcp_toolset",  "mcp_server_name": "github",  "default_config": {"defer_loading": true},  // 整个服务器默认延迟  "configs": {    "search_repos": {"defer_loading": false}   // 高频工具例外  }}
```

**响应格式（搜索 → 发现 → 调用）：**

```
{  "content": [    // Step 1: 模型决定搜索    {"type": "server_tool_use", "name": "tool_search_tool_bm25",     "input": {"query": "github pull request"}},    // Step 2: 返回搜索结果（tool_reference 自动展开为完整定义）    {"type": "tool_search_tool_result",     "content": {"tool_references": [       {"type": "tool_reference", "tool_name": "github.createPullRequest"}     ]}},    // Step 3: 模型调用发现的工具    {"type": "tool_use", "name": "github.createPullRequest",     "input": {"repo": "myorg/myrepo", "title": "Fix bug #123", ...}}  ]}
```

**关键数据：**

•上下文消耗从 ~77K tokens 降至 ~8.7K tokens（降低 85%+）•每次搜索返回 3-5 个最相关工具•支持最多 10,000 个工具的目录•正则查询最大长度：200 字符•也支持自定义客户端搜索实现（返回 `tool_reference` 块即可）

### 4.2 OpenAI GPT-5.4：Tool Search

**发布时间：** 2026 年 3 月（随 GPT-5.4 发布）

**两种执行模式：**

| 模式 | 执行方 | 适用场景 |
| --- | --- | --- |
| **Hosted（托管）** | OpenAI 服务端 | 工具在请求时已知，最简单 |
| **Client-executed（客户端）** | 开发者应用 | 工具发现依赖项目/租户状态等外部系统 |

**核心设计差异：**

OpenAI 的实现强调 **Namespace（命名空间）** 概念。模型主要训练在对 Namespace 和 MCP 服务器级别进行搜索，而非单个函数。对于单个函数的延迟加载，模型仍然能看到函数名和描述，实际延迟的主要是参数 schema。

```
建议：将延迟加载的函数分组到 Namespace 中（每个 Namespace < 10 个函数），用清晰的高层描述让模型知道何时需要加载哪个分组。
```

**Hosted 模式流程：**

```
开发者声明所有工具（含 defer_loading: true）    ↓模型接收请求，只看到工具名/描述 + tool_search 工具    ↓模型判断需要某个延迟加载的工具    ↓API 自动执行搜索，返回 tool_search_call + tool_search_output    ↓加载的工具注入上下文末尾（保护缓存）    ↓模型调用已加载的工具
```

**Client-executed 模式流程：**

```
模型发出 tool_search_call（附带搜索参数）    ↓开发者应用执行自定义搜索逻辑    ↓返回 tool_search_output（包含要加载的工具定义）    ↓模型在后续 turn 中调用已加载的工具
```

**缓存优化：** 新发现的工具被注入到上下文窗口**末尾**，确保前序内容的缓存不被打破。

### 4.3 Spring AI：跨平台 Tool Search Tool

**发布时间：** 2025 年 12 月

**定位：** 将 Tool Search 模式从特定平台抽象为可移植的跨 LLM 框架能力

Spring AI 通过 `ToolSearchToolCallAdvisor`（继承自 `ToolCallAdvisor`）实现了动态工具发现，可在 OpenAI、Anthropic、Gemini、Ollama、Azure OpenAI 等所有 Spring AI 支持的 LLM 上运行。

**三种可插拔搜索策略：**

| 策略 | 实现类 | 适用场景 |
| --- | --- | --- |
| **语义搜索** | `VectorToolSearcher` | 自然语言查询、模糊匹配 |
| **关键词搜索** | `LuceneToolSearcher` | 精确术语匹配、已知工具名 |
| **正则匹配** | `RegexToolSearcher` | 工具名模式（`get_*_data`） |

**跨平台基准测试结果（28 个工具，Lucene 搜索）：**

| 模型 | 传统方式 (tokens) | Tool Search (tokens) | 节省比例 |
| --- | --- | --- | --- |
| Gemini 3 Pro | 5,375 | 2,165 | **60%** |
| GPT-5 Mini | 7,175 | 4,706 | **34%** |
| Claude Sonnet 4.5 | 17,342 | 6,273 | **64%** |

关键发现：**Token 节省主要来自 Prompt Tokens 的减少**——Tool Search 模式下只有被发现的工具定义被包含在 prompt 中。

### 4.4 三平台对比总结

| 维度 | Anthropic Claude | OpenAI GPT-5.4 | Spring AI |
| --- | --- | --- | --- |
| **搜索类型** | Regex + BM25 | Hosted + Client-executed | Vector + Lucene + Regex |
| **延迟加载粒度** | 单工具 / MCP 服务器 | 单函数 / Namespace / MCP | 任意注册工具 |
| **搜索执行方** | 服务端（支持自定义客户端） | 服务端 或 客户端 | 客户端（框架内） |
| **缓存策略** | 延迟工具排除在初始 prompt 外 | 新工具注入上下文末尾 | 依赖底层 LLM 实现 |
| **最大工具数** | 10,000 | 未公开限制 | 无硬限制 |
| **跨模型支持** | 仅 Claude | 仅 GPT-5.4 | 全部 LLM |

---

## 五、技术深潜：Tool Search 的工作原理

### 5.1 系统架构

```
                    ┌────────────────────────────┐                    │     Tool Registry           │                    │  ┌──────┐ ┌──────┐ ┌──────┐│                    │  │Tool A│ │Tool B│ │Tool C││                    │  │defer │ │defer │ │defer ││                    │  └──────┘ └──────┘ └──────┘│                    │  ┌──────┐ ┌──────┐          │                    │  │Tool D│ │Tool E│  ...     │                    │  │defer │ │defer │          │                    │  └──────┘ └──────┘          │                    └──────────┬─────────────────┘                               │                    ┌──────────▼─────────────────┐                    │     Search Index            │                    │  (BM25 / Regex / Semantic)  │                    │  索引：工具名、描述、参数名   │                    └──────────┬─────────────────┘                               │         ┌─────────────────────▼──────────────────────┐         │              LLM Context Window             │         │                                             │         │  ┌─────────────────────────────────────┐   │         │  │ System Prompt                        │   │         │  └─────────────────────────────────────┘   │         │  ┌─────────────────────────────────────┐   │         │  │ Always-loaded Tools (3-5 高频工具)   │   │         │  └─────────────────────────────────────┘   │         │  ┌─────────────────────────────────────┐   │         │  │ Tool Search Tool（~500 tokens）      │   │         │  └─────────────────────────────────────┘   │         │  ┌─────────────────────────────────────┐   │         │  │ Conversation History                 │   │         │  └─────────────────────────────────────┘   │         │  ┌─────────────────────────────────────┐   │         │  │ Dynamically Loaded Tools (末尾注入)  │ ← 按需加载         │  └─────────────────────────────────────┘   │         └─────────────────────────────────────────────┘
```

### 5.2 搜索 → 发现 → 加载 → 调用的完整生命周期

以一个用户请求"帮我查看阿姆斯特丹今天的天气，推荐穿什么衣服，再找几家营业中的服装店"为例：

**Turn 1：模型只看到 `toolSearchTool`**

```
用户 → "帮我在阿姆斯特丹规划今天的穿搭和购物"模型 → calls toolSearchTool(query="current time date")       → 搜索结果：["currentTime"]
```

**Turn 2：模型看到 `toolSearchTool` + `currentTime`**

```
模型 → calls toolSearchTool(query="weather location")  → ["weather"]       calls currentTime("Amsterdam")                   → "2025-12-08T11:30"
```

**Turn 3：模型看到 `toolSearchTool` + `currentTime` + `weather`**

```
模型 → calls toolSearchTool(query="clothing shops")     → ["clothing"]       calls weather("Amsterdam", "2025-12-08T11:30")    → "Sunny, 15°C"
```

**Turn 4：模型看到全部已发现工具**

```
模型 → calls clothing("Amsterdam", "2025-12-08T11:30")  → ["H&M", "Zara", "Uniqlo"]模型 → 生成最终回答，综合天气、穿搭建议和商店推荐
```

全程 28 个注册工具中只有 3 个被实际加载，其余 25 个从未进入上下文。

### 5.3 与 RAG 的类比

Tool Search 本质上是将 RAG（Retrieval-Augmented Generation）的理念应用到工具管理领域：

| 维度 | RAG（知识检索） | Tool Search（工具检索） |
| --- | --- | --- |
| **检索对象** | 文档/知识片段 | 工具定义（名称、描述、参数 schema） |
| **索引方式** | 向量嵌入、BM25 | BM25、Regex、语义嵌入 |
| **触发条件** | 用户提问 | 模型判断需要特定能力 |
| **注入位置** | 上下文中 | 上下文末尾 |
| **目标** | 减少幻觉，提供准确知识 | 减少上下文膨胀，提高工具选择准确率 |

---

## 六、配套能力：与 Tool Search 协同的高级特性

Anthropic 在发布 Tool Search Tool 时，同时推出了两个配套能力，三者协同构成了完整的"高级工具使用"体系：

### 6.1 Programmatic Tool Calling（代码化工具调用）

**问题：** 传统工具调用中，每次调用都需要完整的模型推理，中间结果全部回流到上下文。

**方案：** 模型编写 Python 代码来编排多个工具调用，中间结果在代码执行环境中处理，只有最终结果进入上下文。

```
# 模型生成的编排代码（在沙箱中执行）team = await get_team_members("engineering")expenses = await asyncio.gather(*[    get_expenses(m["id"], "Q3") for m in team])exceeded = [    {"name": m["name"], "spent": sum(e["amount"] for e in exp)}    for m, exp in zip(team, expenses)    if sum(e["amount"] for e in exp) > budgets[m["level"]]["travel_limit"]]print(json.dumps(exceeded))  # 只有这个结果进入模型上下文
```

**效果：** Token 消耗从 43,588 降至 27,297（减少 37%），同时准确率提升。

### 6.2 Tool Use Examples（工具使用示例）

**问题：** JSON Schema 定义了工具的结构，但无法表达使用模式——什么时候该传哪些可选参数、ID 格式约定等。

**方案：** 在工具定义中直接提供 `input_examples`，让模型通过示例学习正确的调用模式。

**效果：** 复杂参数处理的准确率从 72% 提升至 90%。

### 6.3 三者的协同关系

```
┌──────────────────────────────────────────────────────┐│                 Agent 工具使用生命周期                 ││                                                      ││  1. Tool Search        → 发现正确的工具               ││     "找到我需要的工具"                                ││                                                      ││  2. Tool Use Examples  → 正确地调用工具               ││     "学会怎么用这个工具"                              ││                                                      ││  3. Programmatic TC    → 高效地执行工具               ││     "用代码编排多步调用，只返回关键结果"              │└──────────────────────────────────────────────────────┘
```

最佳实践是根据瓶颈逐步引入：

•**参数错误多** → 先加 Tool Use Examples•**中间结果膨胀** → 先加 Programmatic Tool Calling•**工具定义太多** → 先加 Tool Search

---

## 七、实践指南

### 7.1 何时使用 Tool Search

**推荐使用：**

•系统中有 10+ 个可用工具•构建多 MCP 服务器的系统（GitHub + Slack + Jira + ...）•工具定义消耗超过 10K tokens•遇到工具选择准确率问题•工具库会随时间增长

**不推荐使用：**

•工具少于 10 个•所有工具在每次请求中都会被使用•工具定义非常精简（总计 <100 tokens）

### 7.2 最佳实践

**1. 保留 3-5 个高频工具始终加载**

```
{  "name": "search_files",  "description": "Search files in workspace",  "input_schema": {...}  // 无 defer_loading → 始终可用，无需搜索}
```

**2. 工具命名和描述要清晰、可搜索**

```
// 好的命名{"name": "search_customer_orders", "description": "Search for customer orders by date range, status, or total amount. Returns order details including items, shipping, and payment info."}
// 差的命名{"name": "query_db_orders", "description": "Execute order query"}
```

**3. 使用一致的命名空间前缀**

```
github_create_pr, github_list_issues, github_merge_prslack_send_message, slack_list_channelsjira_create_ticket, jira_update_status
```

**4. 在系统提示词中概述可用能力**

```
你可以搜索以下类别的工具：- Slack 消息和频道管理- GitHub 仓库和 PR 操作- Jira 工单追踪- Sentry 错误监控使用 tool search 来查找具体功能。
```

**5. 利用 Namespace 分组（OpenAI）**

每个 Namespace 控制在 10 个函数以内，用清晰的描述概括 Namespace 的能力，让模型能有效决定何时需要加载哪个分组。

### 7.3 权衡与取舍

| 收益 | 代价 |
| --- | --- |
| 大幅减少 Token 消耗 | 额外的搜索步骤增加 1-2 次 API 往返 |
| 提高工具选择准确率 | 搜索可能遗漏相关工具（取决于描述质量） |
| 保护 Prompt Cache | 实现复杂度略增 |
| 支持上千工具的扩展 | 需要维护清晰的工具描述和命名规范 |

---

## 八、Tool Search 与 Prompt Cache 的深度交互

Prompt Cache（提示词缓存）是当前 LLM API 降低成本和延迟的核心机制。Tool Search 的引入对缓存行为有深远影响——既解决了旧问题，也引入了新的设计约束。这一话题在社区中引发了大量讨论。

### 8.1 Prompt Cache 基础：前缀匹配原则

Prompt Cache 的核心原理是**前缀匹配**——API 会缓存从请求开头到某个断点的 KV 张量（Key-Value tensors）。后续请求如果共享相同的前缀，就可以复用这些已计算的张量，跳过重复计算。

```
请求 1:  [系统提示词] [工具定义] [对话历史 Turn 1]                              ↑ 缓存到这里
请求 2:  [系统提示词] [工具定义] [对话历史 Turn 1] [对话历史 Turn 2]         ├── 缓存命中 (复用) ──┤ ├── 新计算 ──────┤
```

**关键约束：前缀中任何位置的变化都会使其后所有内容的缓存失效。**

各平台的缓存实现差异：

| 平台 | 缓存方式 | 最小 Token 阈值 | 缓存保留时间 | 成本折扣 |
| --- | --- | --- | --- | --- |
| **OpenAI** | 自动（前缀匹配） | 1,024 tokens | 5-10 分钟（标准）/ 24 小时（扩展） | 输入价格减 50-90% |
| **Anthropic** | 开发者控制（显式断点） | 1,024 tokens | 5 分钟 / 1 小时 | 缓存读取 90% 折扣 |
| **Google** | 自动 + 显式 | 4,096 tokens | 可配置 | 75% 折扣 |

### 8.2 传统工具定义为何破坏缓存

在没有 Tool Search 的传统模式下，工具定义是缓存前缀的一部分。这导致几种常见的缓存失效场景：

**场景 1：工具增删**

Claude Code 团队在其工程博客中明确指出：

> **"Never Add or Remove Tools Mid-Session."** Changing the tool set in the middle of a conversation is one of the most common ways people break prompt caching.. But because tools are part of the cached prefix, adding or removing a tool invalidates the cache for the entire conversation.

**场景 2：工具顺序变化**

```
请求 A: [系统提示词] [Tool_1] [Tool_2] [Tool_3] [对话...]  ← 缓存建立请求 B: [系统提示词] [Tool_2] [Tool_1] [Tool_3] [对话...]  ← 缓存全部失效！                              ↑ 从这里开始不匹配
```

工具定义的非确定性排序（如使用 HashMap 遍历）是一个隐蔽但致命的缓存杀手。

**场景 3：工具参数更新**

修改工具参数（如更新描述、添加可选字段）也会破坏缓存前缀。Claude Code 团队提到他们曾因"更新 AgentTool 可调用的 agent 列表"而意外触发全局缓存失效。

### 8.3 Tool Search 如何保护缓存

Tool Search 通过两个关键设计巧妙地解决了缓存问题：

**设计 1：延迟工具完全排除在初始前缀之外**

Anthropic 官方文档明确说明：

> "Tool Search Tool doesn't break prompt caching because deferred tools are excluded from the initial prompt entirely. They're only added to context after Claude searches for them, so your system prompt and core tool definitions remain cacheable."

```
传统模式（缓存脆弱）：[系统提示词] [Tool 1~50 的完整定义: ~55K tokens] [对话...]              ↑ 任何工具变化都破坏这里的缓存
Tool Search 模式（缓存稳定）：[系统提示词] [tool_search_tool: ~500 tokens] [3-5 个核心工具] [对话...]              ↑ 始终不变，缓存高度稳定
```

初始前缀从 ~55K tokens 缩减至 ~3-5K tokens，且内容高度稳定。

**设计 2：新发现的工具注入上下文末尾**

OpenAI 在其 Tool Search 文档中强调：

> "Tool search is designed to preserve the model's cache. When new tools are discovered by the model, they are injected at the end of the context window."

```
Turn 1: [系统提示词] [search_tool] [核心工具] [对话 T1]                                                         ↑ 末尾加载 github.createPR
Turn 2: [系统提示词] [search_tool] [核心工具] [对话 T1] [对话 T2] [github.createPR]         ├──────── 缓存命中（前缀不变）────────┤         ├── 新内容 ──────────────┤
```

由于前缀始终一致，缓存不会被动态加载的工具打破。

### 8.4 Claude Code 的缓存工程实践

Claude Code 团队将 Prompt Cache 视为产品基础设施的核心支柱，围绕缓存设计了整个系统架构。以下是他们公开分享的关键经验：

**原则：像监控 uptime 一样监控缓存命中率**

> "We alert on cache breaks and treat them as incidents. A few percentage points of cache miss rate can dramatically affect cost and latency."

**Plan Mode 的缓存友好设计**

直觉做法：进入 Plan Mode 时替换为只读工具集 → **会破坏缓存**

实际做法：始终保持所有工具，用 `EnterPlanMode` / `ExitPlanMode` 作为工具本身，通过系统消息告知模型当前处于计划模式。工具定义永远不变。

```
// 不好：切换工具集Plan Mode 开启 → 移除 Write, Edit 工具 → 缓存失效！
// Claude Code 的做法：用工具表示状态Plan Mode 开启 → 发送系统消息 "你现在在 Plan Mode" → 缓存完好                  EnterPlanMode/ExitPlanMode 始终在工具列表中
```

**Tool Search 在缓存上下文中的角色**

> "Our solution: `defer_loading`. Instead of removing tools, we send lightweight stubs — just the tool name, with `defer_loading: true` — that the model can 'discover' via a ToolSearch tool when needed. The full tool schemas are only loaded when the model selects them. "

核心洞察：`defer_loading` 不是"不发送工具"，而是"发送轻量存根（stub）"。存根始终存在、顺序不变，因此前缀稳定可缓存。

**Compaction（上下文压缩）的缓存安全设计**

当上下文窗口耗尽需要压缩时，Claude Code 使用与父对话**完全相同**的系统提示词、用户上下文和工具定义来执行压缩。这样压缩请求可以复用父对话的缓存前缀，而非从头计算。

### 8.5 学术研究：缓存策略的量化评估

2026 年 1 月，PwC 团队发表论文 **"Don't Break the Cache: An Evaluation of Prompt Caching for Long-Horizon Agentic Tasks"**（arXiv:2601.06007），首次系统量化了不同缓存策略在 Agent 工作负载下的效果。

**实验设置：**

•4 个模型：GPT-5.2、GPT-4o、Claude Sonnet 4.5、Gemini 2.5 Pro•500+ agent 会话，10,000-token 系统提示词•3 种缓存策略 vs 无缓存基线

**三种缓存策略：**

| 策略 | 描述 | 缓存范围 |
| --- | --- | --- |
| **Full Context** | 全量缓存，不加限制 | 系统提示词 + 工具 + 对话历史 + 工具结果 |
| **System Prompt Only** | 只缓存系统提示词 | 系统提示词（在其末尾断开缓存） |
| **Exclude Tool Results** | 排除工具返回结果 | 系统提示词 + 工具 + 对话（排除动态工具结果） |

**核心发现：**

| 指标 | 成本降低 | TTFT（首 Token 延迟）降低 |
| --- | --- | --- |
| 所有模型平均 | **41-80%** | **13-31%** |

**关键结论：**

1.

**全量缓存可能适得其反。** "Naively enabling full-context caching can paradoxically increase latency, as dynamic tool calls and results may trigger cache writes for content that will not be reused across sessions." —— 把动态工具结果也纳入缓存，会产生大量无法复用的缓存写入，反而增加延迟。

2.

**"System Prompt Only" 策略在成本和延迟两个维度都最稳定。** 这恰好与 Tool Search 的设计理念一致——保持前缀（系统提示词 + 搜索工具）稳定可缓存，动态内容放在末尾。

3.

**最大化成本节省的策略不一定最大化延迟节省。** 需要根据优化目标（成本 vs 速度）选择不同策略。

### 8.6 社区讨论与争议

**GitHub Issue #19436：多层级缓存优化**

社区用户 `@guillaume-paradise` 提出了一个细致的优化方案——按内容变化频率使用不同的缓存 TTL：

```
Tier 1 (1 小时缓存): 系统提示词 + 工具定义    ← 极少变化Tier 2 (1 小时缓存): CLAUDE.md 用户配置       ← 偶尔变化Tier 3 (5 分钟缓存): Skills、Hooks、环境变量   ← 每会话变化
```

他计算了潜在收益：假设每小时 10 个会话，当前方案（全部 5 分钟 TTL）需要 10 次缓存写入，而多层级方案只需 1 次写入 + 9 次读取，**缓存操作成本降低 56%**。

该 issue 被标记为 #2603 的重复，说明 Anthropic 团队已在关注这个方向。

**GitHub Issue #525（Agent SDK）& #124（TypeScript SDK）：SDK 层面的 defer\_loading 支持**

多个社区用户报告，虽然 Anthropic API 已支持 `defer_loading`，但官方 SDK 尚未暴露这一能力。一个典型用户反馈：

> "12 SDK tools: ~6,000 tokens; 8 MCP tools: ~5,285 tokens. 在对话开始前就消耗了 15,000-20,000 tokens。"

另一个企业级案例：

> "150+ MCP tools，上下文窗口在开始工作前就已经显著消耗。"

这些 issue 反映出 Tool Search 的 API 可用性与 SDK/产品集成之间仍存在 gap。

**ToolCaching 论文（arXiv:2601.15335）：专用工具缓存框架**

2026 年 1 月的另一篇论文提出了 **VAAC（Value-Aware Adaptive Caching）** 算法，专门针对 LLM 工具调用场景优化缓存。它综合考虑请求频率、时间衰减和缓存价值，实现了：

•缓存命中率提升 **11%**•延迟降低 **34%**

这代表了缓存优化从"通用前缀匹配"向"工具感知的智能缓存"演进的方向。

### 8.7 缓存视角下的 Tool Search 设计总结

```
┌────────────────────────────────────────────────────────────┐│              Prompt Cache 与 Tool Search 的协同              ││                                                            ││  ┌──────────────────────────────────────────────────┐      ││  │  缓存友好前缀（稳定不变）                         │      ││  │  ┌─────────────────────────────────────────────┐ │      ││  │  │ 系统提示词                                   │ │      ││  │  │ + Tool Search Tool (~500 tok)               │ │      ││  │  │ + 3-5 核心工具 (defer_loading: false)        │ │ ← 被缓存│  │  │ + 延迟工具存根 (name only, defer_loading:true)│ │      ││  │  └─────────────────────────────────────────────┘ │      ││  └──────────────────────────────────────────────────┘      ││                                                            ││  ┌──────────────────────────────────────────────────┐      ││  │  动态内容（不缓存 / 会话内缓存）                   │      ││  │  ┌─────────────────────────────────────────────┐ │      ││  │  │ 对话历史                                     │ │      ││  │  │ + 工具调用结果                               │ │ ← 每轮变化│  │  │ + 动态加载的完整工具定义（注入末尾）           │ │      ││  │  └─────────────────────────────────────────────┘ │      ││  └──────────────────────────────────────────────────┘      │└────────────────────────────────────────────────────────────┘
```

**核心设计原则：**

1.**静态在前，动态在后** —— 系统提示词和搜索工具放在最前面，确保跨会话缓存命中2.**只加不减** —— 永远不移除工具定义，只通过 `defer_loading` 延迟加载3.**存根占位** —— 延迟工具以轻量存根形式始终存在，保持前缀顺序稳定4.**末尾注入** —— 新发现的工具加在上下文末尾，不扰动前缀5.**状态用消息而非工具变更表达** —— Plan Mode 等状态切换通过系统消息实现，不改变工具集

---

## 九、行业影响与未来展望

### 9.1 从"函数调用"到"工具生态"的范式转移

Tool Search 的出现标志着 AI Agent 工具使用进入了新阶段：

| 阶段 | 时间 | 特征 |
| --- | --- | --- |
| **函数调用** | 2023 | 模型学会按 JSON Schema 调用函数 |
| **多工具编排** | 2024 | 并行调用、链式调用、工具间依赖管理 |
| **工具生态** | 2025-2026 | 动态发现、按需加载、上千工具的规模化管理 |

这与软件工程中从"静态链接"到"动态链接"再到"微服务发现"的演进路径高度相似。

### 9.2 MCP 与 Tool Search 的共生关系

MCP 协议为 AI 模型提供了连接外部工具的标准接口，但也加剧了"工具爆炸"问题。Tool Search 是 MCP 规模化的关键基础设施：

```
MCP 让 Agent 能连接无限工具    ↓工具数量爆炸导致上下文不可承受    ↓Tool Search 让 Agent 能管理无限工具    ↓真正可扩展的 Agent 工具生态
```

### 9.3 未来方向

1.**更智能的搜索策略：** 从 BM25/Regex 向深度语义理解演进，结合用户意图、历史使用模式进行个性化工具推荐2.**工具元数据标准化：** MCP 协议可能原生支持 `defer_loading` 和搜索元数据3.**跨 Agent 工具共享：** 多个 Agent 协作时共享工具发现结果，避免重复搜索4.**工具质量评估：** 结合 LLM-as-Judge 模式，在工具发现后验证选择是否正确5.**预测性加载：** 基于对话历史和任务类型，预判可能需要的工具并提前准备

---

## 十、结论

Tool Search 不是一个孤立的功能特性，而是 AI Agent 从"玩具"走向"生产"的关键基础设施。它解决的核心矛盾是：

> **Agent 需要的能力越来越多（需要更多工具），但模型的注意力和上下文空间是有限的。**

通过将"全量预加载"变为"按需发现"，Tool Search 让 AI Agent 在保持高精度的同时，能够接入成百上千个工具——这是构建真正实用的 AI Agent 系统的前提条件。

OpenAI 和 Anthropic 几乎同时推出这一能力，Spring AI 等开源框架迅速跟进实现跨平台抽象，说明 Tool Search 已经成为行业共识。随着 MCP 生态的持续扩展，Tool Search 将成为每个 AI Agent 开发者工具箱中的标配。

---

## 参考资料

### References

`[1]` Anthropic - Introducing Advanced Tool Use on the Claude Developer Platform:*https://www.anthropic.com/engineering/advanced-tool-use*
`[2]`Anthropic - Tool Search Tool Documentation:*https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool*
`[3]`OpenAI - Using GPT-5.4: Tool Search:*https://developers.openai.com/api/docs/guides/latest-model*
`[4]`OpenAI - Tool Search Guide:*https://developers.openai.com/api/docs/guides/tools-tool-search*
`[5]`Spring AI - Smart Tool Selection: 34-64% Token Savings with Dynamic Tool Discovery:*https://spring.io/blog/2025/12/11/spring-ai-tool-search-tools-tzolov*
`[6]`TechBuddies - How Claude Code's New MCP Tool Search Slashes Context Bloat:*https://www.techbuddies.io/2026/01/18/how-claude-codes-new-mcp-tool-search-slashes-context-bloat-and-supercharges-ai-agents/*
`[7]`MoneyControl - OpenAI launches GPT-5.4 with major accuracy gains and new tool search system:*https://www.moneycontrol.com/technology/openai-launches-gpt-5-4-with-major-accuracy-gains-and-new-tool-search-system-article-13852424.html*
`[8]`Dev Genius - AI Agent Tool Overload? Cut Token Usage by 99% While Scaling to 1,000+ Tools:*https://blog.devgenius.io/ai-agent-tool-overload-cut-token-usage-by-99-while-scaling-to-1-000-tools-fc91f8e2b6ab*
`[9]`Anthropic - Effective Context Engineering for AI Agents:*https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents*
`[10]`Lessons from Building Claude Code: Prompt Caching Is Everything:*https://www.techtwitter.com/articles/lessons-from-building-claude-code-prompt-caching-is-everything*
`[11]`Don't Break the Cache: An Evaluation of Prompt Caching for Long-Horizon Agentic Tasks:*https://arxiv.org/html/2601.06007v2*
`[12]`ToolCaching: Towards Efficient Caching for LLM Tool-calling:*https://arxiv.org/abs/2601.15335*
`[13]`GitHub Issue #19436 - Feature Request: Multi-Tiered Prompt Caching for Cost Optimization:*https://github.com/anthropics/claude-code/issues/19436*
`[14]`GitHub Issue #525 - Support for Tool Search Tool and Deferred Loading in Claude Agent SDK:*https://github.com/anthropics/claude-agent-sdk-python/issues/525*
`[15]`OpenAI - Prompt Caching Guide:*https://platform.openai.com/docs/guides/prompt-caching*
`[16]`Cursor - Dynamic Context Discovery: *https://cursor.com/blog/dynamic-context-discovery*