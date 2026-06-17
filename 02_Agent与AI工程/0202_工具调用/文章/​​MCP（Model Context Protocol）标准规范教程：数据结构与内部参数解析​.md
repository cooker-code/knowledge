---
title: ​​MCP（Model Context Protocol）标准规范教程：数据结构与内部参数解析​
author: 现实映射理想
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk2NDg3Nzg0OQ==&mid=2247483723&idx=1&sn=684166acc9173c2640fdd116c81bf6fd&chksm=c5408c95b77d4643cd29b3b0c7a5d08e516135b6ec2f94d867e5d5249f25825a09c71db59ec5&mpshare=1&scene=24&srcid=1127MxKfzTNL4GO8Ib8JFHX0&sharer_shareinfo=f2c2ba62413b54623da67066e25b6bd5&sharer_shareinfo_first=f2c2ba62413b54623da67066e25b6bd5#rd
---

大家好！欢迎来到这份**MCP 标准规范教程**，专注于**数据结构**与**内部参数解析**。作为前一份“完全指南”的深度扩展，本教程针对开发者痛点，直击 MCP 的核心协议细节：从 JSON-RPC 基础，到三大原语（Tools、Resources、Prompts）的 Schema 定义，再到方法参数的逐字段拆解。基于 2025 年 6 月 18 日最新规范（Spec 2025-06-18），我们将用真实 JSON 示例、表格和解析图解，帮助你从“看懂”到“实现”。

**为什么这份教程必备？**MCP 的强大在于其标准化数据结构，但官方 Spec 往往晦涩（如 schema.ts 文件）。这里，我们用 Python/Pydantic 示例解析内部参数，避免“黑箱”开发。预计阅读 20-30 分钟，建议结合 GitHub 仓库实践（https://github.com/modelcontextprotocol/modelcontextprotocol）。**赶紧收藏，调试 MCP 时神器！**

**数据来源**：Anthropic 官方 Spec、GitHub 仓库及社区解析（如 Medium 教程）。

---

## 第一章：MCP 协议基础——JSON-RPC 2.0 数据结构（5 分钟入门）

### 1.1 为什么用 JSON-RPC？（内部原理简析）

MCP 不是从零发明轮子，而是基于 **JSON-RPC 2.0**（RFC 862）构建，确保消息结构统一、易序列化。 这是一种轻量级 RPC 协议，支持**请求（Request）**、**响应（Response）**和**通知（Notification）**三种消息类型，适用于 Stdio（本地进程间）和 HTTP+SSE（远程流式）传输。

**核心设计原则**：

* **确定性**

  ：每个参数有严格 Schema（JSON Schema 风格），减少歧义。
* **状态化会话**

  ：Client-Server 连接保持状态，支持能力协商（capabilities negotiation）。
* **错误标准化**

  ：用负整数码（如 -32600 Invalid Request）+ 消息描述。

**MCP 消息通用结构**（JSON 对象）：

```
{  
  "jsonrpc": "2.0",          // 固定版本字符串  
  "id": 1,                   // 请求 ID（整数/字符串/空），用于匹配响应  
  "method": "tools/call",    // 方法名（字符串），如 initialize、tools/list  
  "params": {                // 参数对象（任意 JSON），Schema 依方法而定  
    "name": "fetch_pr",  
    "arguments": {"repo": "anthropic/mcp"}  
  }  
}
```

* **Request**

  ：有 `id`和 `method`。
* **Response**

  ：有 `id`、`result`（成功）或 `error`（失败）。
* **Notification**

  ：无 `id`，用于单向推送（如订阅事件）。

**内部参数解析**：`params`是“心脏”，每个方法有专用 Schema。Spec 用 TypeScript 定义（如 schema.ts），Python SDK 用 Pydantic 验证。

### 1.2 传输层数据结构（Stdio vs. HTTP）

| 传输类型 | 数据结构特点 | 内部参数示例 | 适用场景 |
| --- | --- | --- | --- |
| **Stdio** | 纯文本流，消息以换行分隔 | `Content-Length: 123\n\n{JSON}` | 本地 Server（如文件系统） |
| **HTTP+SSE** | POST 请求 + Server-Sent Events | `event: message\ndata: {JSON}` | 远程（如云 GitHub），支持流式结果 |
| **WebSocket** (扩展) | 双向帧 | `{"type": "message", "data": {JSON}}` | 实时订阅（如日志监控） |

**解析提示**：传输不改变 JSON 核心，但 SSE 的 `data`字段需累积多行解析成完整消息。

---

## 第二章：三大原语的数据结构（核心 Schema 详解，10 分钟）

MCP 的“灵魂”是**Primitives**：Tools（模型控制）、Resources（应用控制）、Prompts（用户控制）。每个原语有独立 Schema，定义在 Spec 的 `/primitives/`部分。 我们用 JSON Schema + 示例解析。

### 2.1 Tools（工具）数据结构

**作用**：AI 主动调用函数，如“fetch\_pr”。Schema 强调输入/输出验证，防注入。

**完整 Schema**（简化 JSON Schema）：

```
{  
  "type": "object",  
  "properties": {  
    "name": {"type": "string", "description": "唯一工具名"},  
    "description": {"type": "string", "description": "自然语言描述"},  
    "inputSchema": {  
      "type": "object",  // JSON Schema for arguments  
      "properties": {  
        "repo": {"type": "string"}  
      },  
      "required": ["repo"]  
    },  
    "outputSchema": {  
      "type": "object",  
      "properties": {  
        "content": {"type": "string"},  
        "mimeType": {"type": "string", "enum": ["text/json", "text/markdown"]}  
      }  
    }  
  },  
  "required": ["name", "description", "inputSchema"]  
}
```

**内部参数解析**：

* **name**

  ：字符串，必填。内部用作方法路由（如 `tools/call`的子命令）。
* **inputSchema**

  ：嵌套 JSON Schema，定义 arguments（如 Pydantic BaseModel）。支持类型（string/number/array）、约束（minLength/regex）。
* **outputSchema**

  ：可选，指导 Client 解析响应。内部用 `mimeType`决定渲染（e.g., JSON vs. Markdown）。
* **扩展参数**

  （2025 更新）：`isUnsafe`(bool) 标记高风险工具；`roots`(array) 限制文件访问根目录。

**示例：GitHub Fetch Tool**

```
{  
  "name": "fetch_pr",  
  "description": "Fetch GitHub PR details",  
  "inputSchema": {  
    "type": "object",  
    "properties": {"repo": {"type": "string"}, "pr_number": {"type": "integer"}},  
    "required": ["repo"]  
  }  
}
```

调用时：`params: {"name": "fetch_pr", "arguments": {"repo": "anthropic/mcp", "pr_number": 123}}`。

### 2.2 Resources（资源）数据结构

**作用**：只读注入上下文，如文件内容。强调安全（app-controlled）。

**Schema**：

```
{  
  "type": "object",  
  "properties": {  
    "uri": {"type": "string", "format": "uri"},  // 资源标识，如 file:///docs/report.md  
    "mimeType": {"type": "string"},  
    "content": {"type": "string" | "object"},    // 实际数据，支持 base64 for binary  
    "metadata": {  
      "type": "object",  
      "properties": {"size": {"type": "number"}, "lastModified": {"type": "string"}}  
    }  
  },  
  "required": ["uri", "mimeType"]  
}
```

**内部参数解析**：

* **uri**

  ：唯一键，格式如 `scheme://path`。内部解析为访问路径，Server 验证权限。
* **content**

  ：字符串或对象。内部用 `mimeType`分支处理（e.g., text/plain 直接注入 Prompt）。
* **metadata**

  ：可选，包含大小/时间戳。2025 Spec 加 `hash`(SHA-256) 防篡改。
* **分页支持**

  ：大资源用 `offset`/`limit`参数，避免 token 爆炸。

**示例：文件资源**

```
{  
  "uri": "file:///project/report.md",  
  "mimeType": "text/markdown",  
  "content": "# Report\nContent here...",  
  "metadata": {"size": 1024, "lastModified": "2025-11-26T00:00:00Z"}  
}
```

### 2.3 Prompts（提示）数据结构

**作用**：用户模板，提升输出一致性。

**Schema**：

```
{  
  "type": "object",  
  "properties": {  
    "name": {"type": "string"},  
    "description": {"type": "string"},  
    "template": {"type": "string"},  // Jinja2-like 模板  
    "parameters": {  
      "type": "object",  
      "properties": {"user": {"type": "string"}}  
    }  
  },  
  "required": ["name", "template"]  
}
```

**内部参数解析**：

* **template**

  ：字符串，支持占位符如 `{{param}}`。内部用 Client 渲染（e.g., Jinja2）。
* **parameters**

  ：Schema for 动态输入。内部验证后注入 LLM Prompt。
* **变体**

  ：`variants`(array) 支持多版本，如 desktop/mobile。

**示例：总结提示**

```
{  
  "name": "summarize",  
  "template": "Summarize the following: {{content}}\nFocus: {{focus}}",  
  "parameters": {"properties": {"focus": {"type": "string", "enum": ["key points", "action items"]}}}  
}
```

渲染：`get_prompt("summarize", {"focus": "key points"})`→ “Summarize the following: {{content}}\nFocus: key points”。

---

## 第三章：核心方法与参数详解（实践解析，10 分钟）

MCP 定义 10+ 方法，分为生命周期（initialize/shutdown）和原语操作。每个用 `method`+ `params`。 下面表格 + 示例。

### 3.1 方法参数表格

| 方法 | 描述 | params Schema（关键内部参数） | 返回 result Schema | 示例调用 |
| --- | --- | --- | --- | --- |
| **initialize** | 会话启动，能力协商 | `{"protocolVersion": "2024-11-05", "capabilities": {"tools": {}, "resources": {}}}`  – protocolVersion: 字符串，必填 – capabilities: 对象，协商支持（如 sampling: bool） | `{"capabilities": {...}, "serverInfo": {"name": "GitServer"}}` | `{"method": "initialize", "params": {"protocolVersion": "2025-06-18"}}` |
| **tools/list** | 列出工具 | 无 params | 数组 of Tool Schema | – |
| **tools/call** | 调用工具 | `{"name": str, "arguments": object (inputSchema)}`  – name: 工具名 – arguments: 验证后执行 | Tool outputSchema | 如 2.1 示例 |
| **resources/read** | 读资源 | `{"uri": str, "offset": int? (分页)}` | Resource Schema | `{"method": "resources/read", "params": {"uri": "file:///report.md"}}` |
| **prompts/list** | 列提示 | 无 | 数组 of Prompt Schema | – |
| **prompts/get** | 获取提示 | `{"name": str, "arguments": object}` | 渲染后字符串 | 如 2.3 示例 |
| **shutdown** | 关闭会话 | 无 | 无 | `{"method": "shutdown"}` (通知) |

**错误结构**（通用）：

```
{  
  "jsonrpc": "2.0",  
  "id": 1,  
  "error": {  
    "code": -32602,  // Invalid params  
    "message": "Missing required arg: repo",  
    "data": {"field": "arguments.repo"}  
  }  
}
```

内部码：-32000 ~ -32099 为 MCP 扩展（如 -32001 Permission Denied）。

### 3.2 高级内部参数：能力协商（Capabilities）

在 `initialize`的 `capabilities`中，定义协议扩展：

```
{  
  "tools": {"call": true},  // 支持工具调用  
  "resources": {"subscribe": true},  // 支持订阅更新  
  "sampling": {  // 2025 新增：Server 请求 LLM 采样  
    "models": ["claude-3.5-sonnet"],  
    "temperature": 0.7  
  }  
}
```

**解析**：Client 比较后返回支持集。内部用作位掩码，优化传输（e.g., 无 sampling 则禁用）。

**安全参数**：`authorization`(OAuth-like) 在 params 中，包含 `token`和 `scopes`（e.g., [“read:repo”]）。

---

## 第四章：实现与调试——Pydantic 示例（5 分钟动手）

用 Python SDK 验证结构：

```
from mcp.types import Tool, JsonSchema  # 从 SDK 导入  
from pydantic import BaseModel  
  
class FetchPRInput(BaseModel):  
    repo: str  
    pr_number: int | None = None  
  
tool = Tool(  
    name="fetch_pr",  
    description="Fetch PR",  
    inputSchema=JsonSchema(type="object", properties={"repo": {"type": "string"}})  # 内部验证  
)  
  
# 调用解析  
arguments = {"repo": "test/repo"}  
validated = FetchPRInput(**arguments)  # 抛 Invalid arg 若错
```

**调试技巧**：

* 用 `mcp debug`命令打印 JSON。
* 验证 Schema：工具如 JSON Schema Validator。
* 常见坑：arguments 未匹配 inputSchema → -32602 错误。

---

## 结语：掌握结构，征服 MCP！

恭喜！你已解锁 MCP 的数据“黑箱”——从 JSON-RPC 骨架，到原语 Schema 的血肉，再到参数的脉络。这不仅是规范，更是构建可靠 AI 代理的蓝图。**行动：fork GitHub Spec，添加自定义 Tool 测试！**未来，2026 Spec 或加多模态 Schema（图像/视频）。

疑问？查官方 Spec（https://yingjuxia.com/archives/6284）或社区（如 Medium）。基于 2025-11-26 数据，保持更新！