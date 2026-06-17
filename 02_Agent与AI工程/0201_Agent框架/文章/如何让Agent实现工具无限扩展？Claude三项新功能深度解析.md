---
title: 如何让Agent实现工具无限扩展？Claude三项新功能深度解析
author: 小窗幽记机器学习
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwNjE2ODMxNQ==&mid=2247490584&idx=1&sn=9b7a6c53e6afd9f349752d49d84fd986&chksm=c1cc1124c638c6353a7fa65b97270fcddc1177964517155d11c24dbc53814050bbf877ce1b11&mpshare=1&scene=24&srcid=1129kD7ANrcpKwjRn20e396l&sharer_shareinfo=d5f7020ce07efe9d9b390d662128000a&sharer_shareinfo_first=d5f7020ce07efe9d9b390d662128000a#rd
---

## 0. 引言

Anthropic近日发布了三项Beta功能，让Claude能够动态发现、学习并执行工具。今天这篇小作文将详细拆解了这些功能的工作原理与最佳实践。 

参考原文：https://www.anthropic.com/engineering/advanced-tool-use

更多AI相关欢迎关注微信公众号"小窗幽记机器学习": 

## 1. 背景：为什么需要这些功能

真正高效的AI Agent需要在数百甚至数千种工具间无缝协作——无论是整合git、文件管理、测试框架的IDE助手，还是同时连接Slack、GitHub、Jira等十几个服务的运维协调器。

然而，将所有工具定义塞进上下文会带来严重问题：工具结果和定义有时在Agent开始工作前就消耗超过50,000个token。此外，每次自然语言工具调用都需要完整推理过程，中间结果不断累积，而JSON schema只能定义结构有效性，无法表达使用模式。

此次发布的三项功能直接解决这些痛点：

* **工具搜索工具（Tool Search Tool）**：按需访问数千种工具，不占用上下文窗口
* **程序化工具调用（Programmatic Tool Calling）**：通过代码执行环境调用工具，减少上下文消耗
* **工具使用示例（Tool Use Examples）**：通过示例演示正确的工具使用方式

内部测试中，Claude for Excel正是借助程序化工具调用，实现了对数千行电子表格的读写，同时不导致上下文过载。

## 2. 工具搜索工具

### 问题

MCP工具定义的token消耗会随服务器增多快速累积。一个典型的五服务器配置：

* GitHub：35个工具（约26K token）
* Slack：11个工具（约21K token）
* Sentry + Grafana + Splunk：约8K token

**58个工具在对话开始前就消耗约55K token**。加入Jira等更多服务，很快超过100K。Anthropic内部曾观察到优化前工具定义消耗达134K token的情况。

更关键的是，最常见的失败原因是**工具选择错误和参数填写不正确**，尤其当工具名称相似时（如`notification-send-user`与`notification-send-channel`）。

### 解决方案

工具搜索工具不再预先加载所有定义，而是**按需发现**。Claude只看到当前任务实际需要的工具。

*工具搜索工具相比传统方式可节省191,300 token的上下文空间*

**对比效果：**

| 方式 | 上下文消耗 |
| --- | --- |
| 传统方式 | 约77K token（50+ MCP工具约72K + 其他） |
| 工具搜索工具 | 约8.7K token（搜索工具500 + 3-5个相关工具3K） |

**Token减少85%，同时保留完整工具库访问能力。**

内部测试显示准确率显著提升：Opus 4从49%提升至74%，Opus 4.5从79.5%提升至88.1%。

### 工作原理

将工具标记为`defer_loading: true`实现按需发现。当Claude需要特定功能时搜索相关工具，匹配结果在上下文中展开为完整定义。

```
{  
  "tools": [  
    {"type": "tool_search_tool_regex_20251119", "name": "tool_search_tool_regex"},  
    {  
      "name": "github.createPullRequest",  
      "description": "Create a pull request",  
      "input_schema": {...},  
      "defer_loading": true  
    }  
  ]  
}
```

对于MCP服务器，可延迟加载整个服务器，同时保持高频工具处于加载状态：

```
{  
  "type": "mcp_toolset",  
  "mcp_server_name": "google-drive",  
  "default_config": {"defer_loading": true},  
  "configs": {  
    "search_files": {"defer_loading": false}  
  }  
}
```

**提示缓存说明**：延迟加载的工具不包含在初始提示中，因此系统提示词和核心工具定义仍可被缓存。

### 适用场景

**适合使用：**

* 工具定义超过10K token
* 遇到工具选择准确率问题
* 构建多服务器MCP系统
* 可用工具超过10个

**收益较小：**

* 工具库规模小（少于10个）
* 每次会话都使用所有工具
* 工具定义本身精简

## 3. 程序化工具调用

### 问题

传统工具调用存在两个根本性问题：

1. **上下文污染**：分析10MB日志文件时，整个文件进入上下文，尽管只需要错误摘要。中间结果消耗token预算，可能将重要信息挤出上下文。
2. **推理开销**：每次调用需要完整模型推理。五次工具调用意味着五次推理过程，加上解析、比较、综合——全通过自然语言处理。

### 解决方案

程序化工具调用让Claude通过代码而非逐个API往返来编排工具。中间结果由脚本处理而非进入上下文。

程序化工具调用支持并行工具执行

#### 示例：预算合规检查

任务："哪些团队成员超出了Q3差旅预算？"

**传统方式**：获取20人 → 每人获取费用（20次调用，2000+条明细进入上下文）→ 手动汇总比较

**程序化调用**：Claude编写Python脚本编排整个流程，只输出超出预算的人员：

```
team = await get_team_members("engineering")  
  
levels = list(set(m["level"] for m in team))  
budget_results = await asyncio.gather(*[  
    get_budget_by_level(level) for level in levels  
])  
budgets = {level: budget for level, budget in zip(levels, budget_results)}  
  
expenses = await asyncio.gather(*[  
    get_expenses(m["id"], "Q3") for m in team  
])  
  
exceeded = []  
for member, exp in zip(team, expenses):  
    budget = budgets[member["level"]]  
    total = sum(e["amount"] for e in exp)  
    if total > budget["travel_limit"]:  
        exceeded.append({  
            "name": member["name"],  
            "spent": total,  
            "limit": budget["travel_limit"]  
        })  
  
print(json.dumps(exceeded))
```

**效果对比：**

* Token消耗从200KB原始数据减至1KB结果
* 平均token使用从43,588降至27,297（减少37%）
* 知识检索准确率从25.6%提升至28.5%
* GIA基准测试从46.5%提升至51.2%

### 工作原理

**1. 标记可从代码调用的工具**

```
{  
  "tools": [  
    {"type": "code_execution_20250825", "name": "code_execution"},  
    {  
      "name": "get_team_members",  
      "description": "Get all members of a department...",  
      "input_schema": {...},  
      "allowed_callers": ["code_execution_20250825"]  
    }  
  ]  
}
```

**2. Claude生成编排代码**

```
{  
  "type": "server_tool_use",  
  "id": "srvtoolu_abc",  
  "name": "code_execution",  
  "input": {"code": "team = get_team_members('engineering')\n..."}  
}
```

**3. 工具执行不影响上下文**

工具请求带有`caller`字段标识来源：

```
{  
  "type": "tool_use",  
  "id": "toolu_xyz",  
  "name": "get_expenses",  
  "input": {"user_id": "emp_123", "quarter": "Q3"},  
  "caller": {  
    "type": "code_execution_20250825",  
    "tool_id": "srvtoolu_abc"  
  }  
}
```

结果在代码环境处理，不进入上下文。

**4. 只有最终输出返回**

```
{  
  "type": "code_execution_tool_result",  
  "tool_use_id": "srvtoolu_abc",  
  "content": {  
    "stdout": "[{\"name\": \"Alice\", \"spent\": 12500, \"limit\": 10000}...]"  
  }  
}
```

### 适用场景

**适合使用：**

* 处理大型数据集只需聚合/摘要
* 三个或更多依赖工具调用的多步骤工作流
* 需在Claude看到结果前进行过滤/转换
* 中间数据不应影响推理的任务
* 并行操作（如检查50个端点）

**收益较小：**

* 简单单工具调用
* Claude需要查看所有中间结果的任务
* 响应量小的快速查询

## 4. 工具使用示例

### 问题

JSON Schema定义结构有效性，但无法表达使用模式。考虑一个工单API：

```
{  
  "name": "create_ticket",  
"input_schema": {  
    "properties": {  
      "title": {"type": "string"},  
      "priority": {"enum": ["low", "medium", "high", "critical"]},  
      "labels": {"type": "array", "items": {"type": "string"}},  
      "reporter": {"type": "object", "properties": {...}},  
      "due_date": {"type": "string"},  
      "escalation": {"type": "object", "properties": {...}}  
    },  
    "required": ["title"]  
  }  
}
```

关键问题未解答：

* `due_date`用什么格式？
* `reporter.id`是UUID还是"USR-12345"？
* 什么时候填充嵌套的`contact`对象？
* `escalation`参数与`priority`如何关联？

### 解决方案

工具使用示例直接在定义中提供示例调用：

```
{  
    "name": "create_ticket",  
    "input_schema": {...},  
    "input_examples": [  
      {  
        "title": "Login page returns 500 error",  
        "priority": "critical",  
        "labels": ["bug", "authentication", "production"],  
        "reporter": {  
          "id": "USR-12345",  
          "name": "Jane Smith",  
          "contact": {"email": "jane@acme.com", "phone": "+1-555-0123"}  
        },  
        "due_date": "2024-11-06",  
        "escalation": {"level": 2, "notify_manager": true, "sla_hours": 4}  
      },  
      {  
        "title": "Add dark mode support",  
        "labels": ["feature-request", "ui"],  
        "reporter": {"id": "USR-67890", "name": "Alex Chen"}  
      },  
      {  
        "title": "Update API documentation"  
      }  
    ]  
  }
```

Claude从示例学到：

* **格式约定**：日期用YYYY-MM-DD，ID用USR-XXXXX，labels用kebab-case
* **嵌套结构模式**：如何构建reporter及其contact对象
* **可选参数关联**：紧急bug需完整信息+升级配置；功能请求有reporter但无联系信息；内部任务只有标题

**内部测试中，复杂参数处理准确率从72%提升至90%。**

### 适用场景

**适合使用：**

* 复杂嵌套结构（有效JSON ≠ 正确用法）
* 多可选参数且填写模式重要
* 有领域特定约定的API
* 相似工具需示例区分（如`create_ticket` vs `create_incident`）

**收益较小：**

* 用法显而易见的简单工具
* 标准格式如URL或邮箱
* 更适合JSON Schema约束的验证问题

## 5. 最佳实践

### 分层策略

从最大瓶颈开始：

* 上下文膨胀 → 工具搜索工具
* 中间结果污染 → 程序化工具调用
* 参数/格式错误 → 工具使用示例

三者互补：工具搜索确保找到正确工具，程序化调用确保高效执行，示例确保正确调用。

### 工具搜索工具设置

使用清晰、描述性的定义提高发现准确率：

```
// 好的做法  
{  
    "name": "search_customer_orders",  
    "description": "Search for customer orders by date range, status, or total amount..."  
}  
  
// 不好的做法  
{  
    "name": "query_db_orders",  
    "description": "Execute order query"  
}
```

保持3-5个最常用工具始终加载，其余延迟加载。

### 程序化工具调用设置

清晰记录返回格式，帮助Claude编写正确的解析逻辑：

```
{  
    "name": "get_orders",  
    "description": "Retrieve orders for a customer.  
Returns:  
    List of order objects, each containing:  
    - id (str): Order identifier  
    - total (float): Order total in USD  
    - status (str): One of 'pending', 'shipped', 'delivered'..."  
}
```

### 工具使用示例设置

* 使用真实数据（真实城市名、合理价格）
* 展示多样性：最小、部分和完整填写模式
* 每个工具1-5个示例
* 只在schema无法明确正确用法时添加

## 6. 开始使用

这些功能目前处于Beta阶段，需添加beta header：

```
client.beta.messages.create(  
    betas=["advanced-tool-use-2025-11-20"],  
    model="claude-sonnet-4-5-20250929",  
    max_tokens=4096,  
    tools=[  
        {"type": "tool_search_tool_regex_20251119", "name": "tool_search_tool_regex"},  
        {"type": "code_execution_20250825", "name": "code_execution"},  
        # 带有 defer_loading、allowed_callers 和 input_examples 的工具  
    ]  
)
```