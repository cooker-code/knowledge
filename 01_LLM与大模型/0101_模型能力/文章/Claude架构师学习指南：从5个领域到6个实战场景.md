---
title: Claude架构师学习指南：从5个领域到6个实战场景
author: 今天又学AI了
date: 在学AI了在学AI了
url: https://mp.weixin.qq.com/s?__biz=MzYyNDMwMjAxMA==&mid=2247484944&idx=1&sn=ff13b88470e899edeaf8cd26cce4e359&chksm=f1396e21d07fe15c8b9a33a39c19eab0b42b708a738c9d99f425eb4f46f27844885372f759ed&mpshare=1&scene=24&srcid=04270cS5PzNpbAQqoyKw75LM&sharer_shareinfo=b4705eb76b38d3a19d2ac08c556d4c04&sharer_shareinfo_first=b4705eb76b38d3a19d2ac08c556d4c04#rd
---

# 我花了1天时间，研究了一份我们考不了的考试。

X 上有条长推文，把整个 Claude 架构师认证考试的复习指南拆解出来了。

官方说法是 Claude Certified Architect (Foundations) 认证考试。现实是只有 Claude 合作伙伴才能考。

但谁在乎那张证书？你需要的是知识，是能构建生产级应用的能力。

我把它拆开揉碎，给你省流版。

---

## 省流版：5 个核心领域

1. 1. **Agentic Architecture & Orchestration (27%)** - 怎么设计 Agent，怎么让多个 Agent 协作
2. 2. **Tool Design & MCP Integration (18%)** - 怎么设计工具，怎么接 MCP
3. 3. **Claude Code Configuration & Workflows (20%)** - 怎么配置 Claude Code，怎么设计工作流
4. 4. **Prompt Engineering & Structured Output (20%)** - 怎么写提示词，怎么拿结构化输出
5. 5. **Context Management & Reliability (15%)** - 怎么管理上下文，怎么保证可靠性

**这5个领域，对应6个真实场景**：

* • Customer Support Resolution Agent（客服 Agent）
* • Code Generation with Claude Code（代码生成）
* • Multi-Agent Research System（多 Agent 协作）
* • Developer Productivity Tools（开发工具）
* • Claude Code for CI/CD（持续集成）
* • Structured Data Extraction（数据提取）

---

## Domain 1: Agentic Architecture & Orchestration (27%)

这是最重的一块，27% 的分值。

### 考试陷阱：3 个反模式

**考试会给你4个选项，让你选哪个是错的。**

记住这3个：

1. 1. **用自然语言解析来判断循环终止** ❌

* • 错误：让 AI "当你觉得任务完成时就停止"
* • 正确：用明确的 stop\_reason 字段（`tool_use` 或 `end_turn`）

2. 2. **用固定迭代次数作为主要停止机制** ❌

* • 错误：`for i in range(10): agent.step()`
* • 正确：基于 stop\_reason 和工具执行结果判断

3. 3. **检查 assistant 文本作为完成指示** ❌

* • 错误：`if "任务完成" in assistant_message: break`
* • 正确：检查 `stop_reason == "end_turn"`

**最大的坑**：很多人以为 subagent 和 coordinator 共享内存。

> **事实是**：Subagent 完全隔离，每条信息必须显式传递。

### 真实应用：Customer Support Agent

**场景**：用户投诉订单问题，Agent 需要查订单、退款、发邮件。

**架构设计**：

```
Coordinator Agent  
├── Tool: 查询订单系统  
├── Tool: 执行退款  
├── Subagent: 邮件发送（独立上下文）  
└── Escalation: 转人工客服
```

**关键代码逻辑**：

```
# 错误示范  
while True:  
    response = claude.messages.create(...)  
    if "完成" in response.content:  # ❌ 依赖文本判断  
        break  
  
# 正确示范  
while True:  
    response = claude.messages.create(...)  
    if response.stop_reason == "end_turn":  # ✅ 基于 stop_reason  
        break  
    elif response.stop_reason == "tool_use":  
        tool_results = execute_tools(response.tool_calls)  
        messages.append({"role": "user", "content": tool_results})
```

**安全加固**（考试重点）：

> 当涉及金钱或安全时，光靠提示词不够。  
> 必须用 hooks 和 prerequisite gates 强制工具调用顺序。

**学习资源**：

* • Agent SDK Overview（Agentic loop 机制）
* • Building Agents with Claude Agent SDK（Anthropic 官方最佳实践）
* • Agent SDK Python repo（hooks、custom tools、fork\_session 示例）

---

## Domain 2: Tool Design & MCP Integration (18%)

**最被低估的领域**：Tool descriptions。

### 考试陷阱：工具描述太随便

很多人写工具描述是这样的：

```
# ❌ 错误示范  
tools = [  
    {  
        "name": "search_database",  
        "description": "搜索数据库"  # 太模糊  
    }  
]
```

**考试会考**：给4个工具描述，让你选哪个最清晰。

**正确示范**：

```
# ✅ 正确示范  
tools = [  
    {  
        "name": "search_customer_orders",  
        "description": "Search customer orders by email or order ID. Returns order status, items, and tracking info.",  
        "input_schema": {  
            "type": "object",  
            "properties": {  
                "query": {  
                    "type": "string",  
                    "description": "Customer email or order ID"  
                }  
            },  
            "required": ["query"]  
        }  
    }  
]
```

### 真实应用：Developer Productivity Tools

**场景**：给开发团队构建工具集，集成 MCP servers。

**MCP 是什么**：

> **官方说法**：Model Context Protocol，一种标准化的上下文协议。  
> **人话翻译**：就像 USB 接口，让不同的工具能插到 Claude 上。

**MCP 集成示例**：

```
// 连接 MCP server  
const mcpClient = new MCPClient({  
  servers: [  
    {  
      name: "filesystem",  
      command: "mcp-server-filesystem",  
      args: ["/path/to/project"]  
    },  
    {  
      name: "postgres",  
      command: "mcp-server-postgres",  
      args: ["postgresql://localhost/mydb"]  
    }  
  ]  
});  
  
// Claude 自动获得这些工具  
const response = await claude.messages.create({  
  model: "claude-sonnet-4-6",  
  messages: [{role: "user", content: "查询用户表"}],  
  // 不需要手动定义 tools，MCP 自动提供  
});
```

**考试陷阱**：MCP server 的工具和手动定义的 tools 冲突时，谁优先？

> **答案**：手动定义优先，MCP 作为补充。

---

## Domain 3: Claude Code Configuration & Workflows (20%)

**区分度**：这域区分"用过 Claude Code"和"给团队配置过 Claude Code"的人。

### 考试陷阱：只当工具用，不配置

很多人只知道 `claude code`，不知道还能配置。

**核心配置文件**：

1. 1. **CLAUDE.md** - 项目级配置
2. 2. **Plan mode** - 非交互式规划
3. 3. **Slash commands** - 自定义命令

**CLAUDE.md 示例**：

```
# Project: E-commerce API  
  
## Architecture  
- Backend: Node.js + Express  
- Database: PostgreSQL  
- Auth: JWT  
  
## Code Style  
- Use TypeScript strict mode  
- Prefer functional components  
- Test coverage > 80%  
  
## Key Files  
- `src/routes/` - API endpoints  
- `src/models/` - Database models  
- `tests/` - Jest tests
```

### 真实应用：Code Generation

**场景**：让 Claude Code 自动生成符合项目规范的代码。

**Plan mode 工作流**：

```
# 1. 进入 plan mode  
claude code --plan  
  
# 2. Claude 分析项目结构  
> 基于你的 CLAUDE.md，我建议：  
> 1. 创建 src/routes/orders.ts  
> 2. 添加订单验证逻辑  
> 3. 生成 Jest 测试  
  
# 3. 确认后执行  
> Execute plan? (y/n) y  
  
# 4. 自动生成代码 + 测试
```

**Slash commands 示例**：

```
# 自定义命令：生成 API 端点  
claude code /api-endpoint --method POST --resource orders  
  
# 自动生成：  
# - Route handler  
# - Validation middleware  
# - Swagger docs  
# - Jest tests
```

---

## Domain 4: Prompt Engineering & Structured Output (20%)

**核心原则**：两个字——明确。

### 考试陷阱：不够明确

**错误示范**：

```
# ❌ 太模糊  
prompt = "提取用户信息"
```

**正确示范**：

```
# ✅ 明确的 schema  
prompt = """  
Extract user information from the text.  
  
Return JSON with this exact structure:  
{  
  "name": "string (full name)",  
  "email": "string (valid email format)",  
  "phone": "string (format: +1-XXX-XXX-XXXX)",  
  "address": {  
    "street": "string",  
    "city": "string",  
    "zip": "string (5 digits)"  
  }  
}  
  
If any field is missing, use null.  
"""
```

### 真实应用：Structured Data Extraction

**场景**：从非结构化文本提取结构化数据。

**Validation loop 示例**：

```
import json  
from jsonschema import validate  
  
schema = {  
  "type": "object",  
  "properties": {  
    "name": {"type": "string"},  
    "email": {"type": "string", "format": "email"}  
  },  
  "required": ["name", "email"]  
}  
  
def extract_with_validation(text, max_retries=3):  
    for attempt in range(max_retries):  
        response = claude.messages.create(  
            model="claude-sonnet-4-6",  
            messages=[{  
                "role": "user",  
                "content": f"Extract: {text}\n\nReturn JSON."  
            }]  
        )  
  
        try:  
            data = json.loads(response.content[0].text)  
            validate(instance=data, schema=schema)  # ✅ Schema 验证  
            return data  
        except (json.JSONDecodeError, ValidationError) as e:  
            if attempt < max_retries - 1:  
                # 反馈错误，让 Claude 修正  
                continue  
            raise
```

**考试重点**：当输出不符合 schema 时，怎么处理？

> **答案**：不要直接失败，而是把错误信息反馈给 Claude，让它修正。

---

## Domain 5: Context Management & Reliability (15%)

**最轻但最致命**：Context 超限 = 应用崩溃。

### 考试陷阱：Context 超限

**问题**：200K context window 看起来很大，但很快就会用完。

**解决方案**：

1. 1. **Context compression** - 压缩历史对话
2. 2. **Sliding window** - 只保留最近 N 轮对话
3. 3. **Summarization** - 定期总结旧对话

**代码示例**：

```
def manage_context(messages, max_tokens=150000):  
    # 1. 计算 token 数  
    total_tokens = count_tokens(messages)  
  
    if total_tokens > max_tokens:  
        # 2. 压缩策略：保留 system + 最近5轮  
        compressed = [messages[0]]  # system message  
        compressed += messages[-10:]  # 最近5轮（每轮2条消息）  
  
        # 3. 添加总结  
        summary = summarize_messages(messages[1:-10])  
        compressed.insert(1, {  
            "role": "user",  
            "content": f"[Previous context summary]: {summary}"  
        })  
  
        return compressed  
  
    return messages
```

### 真实应用：CI/CD Pipelines

**场景**：在 CI/CD 中使用 Claude Code，需要非交互式 + 结构化输出。

**GitHub Actions 示例**：

```
name: Code Review  
on: [pull_request]  
  
jobs:  
  review:  
    runs-on: ubuntu-latest  
    steps:  
      - uses: actions/checkout@v3  
  
      - name: Claude Code Review  
        env:  
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}  
        run: |  
          # 非交互式执行  
          claude code --non-interactive \  
            --output-format json \  
            --command "Review this PR and suggest improvements" \  
            > review_result.json  
  
      - name: Post Review Comment  
        run: |  
          comment=$(jq -r '.suggestions' review_result.json)  
          # Post to GitHub PR
```

**考试重点**：CI/CD 中如何处理错误？

> **答案**：
>
> 1. 1. 使用 `--non-interactive` 模式
> 2. 2. 设置超时时间
> 3. 3. 输出结构化 JSON
> 4. 4. 错误时自动回滚

---

## 实践指南：如何开始学习

### Step 1: 搭建开发环境

```
# 1. 安装 Claude Code  
npm install -g @anthropic/claude-code  
  
# 2. 配置 API key  
export ANTHROPIC_API_KEY="your-key"  
  
# 3. 初始化项目  
mkdir claude-architect-lab && cd claude-architect-lab  
claude code init
```

### Step 2: 动手实践 6 个场景

**Week 1**：Customer Support Agent

* • 实现 Agentic loop
* • 添加工具调用
* • 实现错误处理

**Week 2**：Code Generation

* • 配置 CLAUDE.md
* • 使用 Plan mode
* • 创建 Slash commands

**Week 3**：Multi-Agent System

* • 实现 Coordinator-Subagent 架构
* • 测试内存隔离
* • 添加 Hooks

**Week 4**：MCP Integration

* • 连接 MCP server
* • 测试工具冲突
* • 实现 fallback

**Week 5**：Structured Output

* • 设计 JSON schema
* • 实现 Validation loop
* • 测试错误修正

**Week 6**：CI/CD Pipeline

* • 配置非交互式模式
* • 设置 GitHub Actions
* • 测试自动化流程

---

## 最后的话

这份考不了的考试，教了你最值钱的东西。

5 个领域，6 个场景，每个都能直接变现：

* • Agentic Architecture - 能设计复杂的 AI 系统
* • Tool Design - 能构建实用的工具链
* • Claude Code - 能提升团队开发效率
* • Prompt Engineering - 能拿到可靠的结构化输出
* • Context Management - 能构建稳定的生产系统

不需要证书证明你能做这些。

做出东西来，就是最好的证明。