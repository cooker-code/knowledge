> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020112_LangChain/020112_核心知识点/LangChain生产部署与持久化边界|LangChain生产部署与持久化边界]]
---
title: 【LangChain】生产级Agent部署与监控
author: Dev派
date:
url: https://mp.weixin.qq.com/s?__biz=MzU2NzAwMjkxMg==&mid=2247484031&idx=1&sn=43213ac4baf093fc9a08149b47d29089&chksm=fd48b29e25d80a8fb248a6a19253a316affa906727e0625d837c4b753e99d372e222646574e4&mpshare=1&scene=24&srcid=0414Agg1qE1bf7XiJ1UuoX6J&sharer_shareinfo=5abea8439667155b90e0c7d034544f2a&sharer_shareinfo_first=5abea8439667155b90e0c7d034544f2a#rd
---

> **"从玩具到产品，差的不是代码，是敬畏。"**

如果你也曾被凌晨3点的报警短信吓醒，看着账单上暴涨的Token费用一脸懵逼，或者在用户投诉"Agent又卡死了"时手足无措——这一章就是为你准备的。

把Agent从本地笔记本搬到生产环境，就像把实验室里的原型机搬进工厂车间。**你需要的不只是能跑，而是能扛、能省、能自愈。**

---

## 一、Checkpointer持久化：别让对话变成"金鱼记忆"

### 1.1 为什么MemorySaver不能上生产？

LangGraph默认的`MemorySaver`就像金鱼——7秒记忆，服务器一重启，所有对话历史灰飞烟灭。

```
from langgraph.checkpoint.memory import MemorySaver

# ❌ 开发环境可以，生产环境千万别用！
memory = MemorySaver()  # 数据存在RAM里，重启即丢失
```

**生产环境需要的是"大象记忆"**——持久化、可恢复、能审计。

### 1.2 SQLite方案：单服务器部署的救星

对于中小规模应用，`SqliteSaver`是最简单的持久化方案：

```
from langgraph.checkpoint.sqlite import SqliteSaver
from langgraph.graph import StateGraph, MessagesState
import os

# 创建持久化数据库文件
db_path = os.path.join(os.getcwd(), "checkpoints.db")

# ✅ 数据存在磁盘，重启后依然记得用户是谁
checkpointer = SqliteSaver.from_conn_string(db_path)

workflow = StateGraph(state_schema=MessagesState)
# ... 添加节点和边 ...

app = workflow.compile(checkpointer=checkpointer)

# 每个用户/对话都有唯一的thread_id
config = {"configurable": {"thread_id": "user_123:conv_456"}}

# 即使服务器重启，对话也能无缝继续
response = app.invoke(
    {"messages": [{"role": "user", "content": "我昨天说的项目截止日期是哪天？"}]},
    config=config
)
```

**关键点**：`thread_id`是持久化的钥匙。建议格式：`{user_id}:{conversation_id}`，方便后续分用户管理和数据清理。

### 1.3 PostgreSQL方案：高并发的终极答案

当单台服务器扛不住，或者需要多机共享状态时，PostgreSQL是你的归宿：

```
from langgraph.checkpoint.postgres import PostgresSaver
from psycopg import Connection

# 使用连接池管理数据库连接
conn_string = "postgresql://user:pass@localhost:5432/agent_db"

with Connection.connect(conn_string, autocommit=True) as conn:
    checkpointer = PostgresSaver(conn)
    checkpointer.setup()  # 创建必要的表结构
    
    app = workflow.compile(checkpointer=checkpointer)
```

**为什么选PostgreSQL？**

* ✅ 支持高并发读写
* ✅ 多机部署共享状态
* ✅ 成熟的备份恢复机制
* ✅ 可以结合Redis做二级缓存

---

## 二、错误重试与降级：让Agent学会"自救"

生产环境的API调用，就像在城市高峰期打车——**不是每次都能成功，但用户不能因此 stranded**。

### 2.1 错误分类处理策略

不是所有错误都一样对待。LangGraph官方建议的分层策略：

| 错误类型 | 处理者 | 策略 | 适用场景 |
| --- | --- | --- | --- |
| 瞬态错误（网络抖动、限流） | 系统自动 | 指数退避重试 | 临时故障，重试通常能解决 |
| LLM可恢复错误（工具失败、解析错误） | LLM自己 | 把错误喂给模型，让它重试 | 模型能看到错误并调整策略 |
| 需人工介入错误（信息缺失、指令不清） | 人类 | 中断等待`interrupt()` | 需要用户提供额外信息 |
| 未知错误 | 开发者 | 抛出异常，记录日志 | 需要排查的新问题 |

### 2.2 实战：三层防御体系

```
from deepagents import create_deep_agent
from langchain.agents.middleware import (
    ModelFallbackMiddleware,
    ModelRetryMiddleware,
    ToolRetryMiddleware,
)

agent = create_deep_agent(
    model="claude-sonnet-4",  # 主力模型
    middleware=[
        # 🛡️ 第一层：瞬态错误自动重试
        ModelRetryMiddleware(
            max_retries=3,
            backoff_factor=2.0,  # 指数退避：1s, 2s, 4s
            initial_delay=1.0,
            retry_on=(RateLimitError, TimeoutError, ConnectionError)
        ),
        
        # 🛡️ 第二层：主力模型挂了，自动降级
        ModelFallbackMiddleware(
            fallback_model="gpt-4.1-mini",  # 便宜且稳定的备胎
            fallback_on=(ServiceUnavailableError, AuthenticationError)
        ),
        
        # 🛡️ 第三层：特定工具的错误隔离
        ToolRetryMiddleware(
            max_retries=2,
            tools=["search", "fetch_url"],  # 只重试外部API工具
            retry_on=(TimeoutError, ConnectionError),
            # 本地文件操作不重试，因为重试也没用
        ),
    ],
)
```

**关键原则**：

* **只对瞬态错误重试**：文件读取失败重试100次也没用
* **降级要果断**：主模型挂了立刻切备胎，别让用户干等
* **Scope要精确**：ToolRetryMiddleware只绑定到特定工具，别一刀切

### 2.3 让LLM自己"反省"错误

有些错误，喂给模型比直接重试更有效：

```
from langgraph.types import Command

def execute_tool(state, config):
    try:
        result = risky_operation(state["tool_call"])
        return Command(update={"results": result}, goto="agent")
    except Exception as e:
        # 把错误信息丢给LLM，让它自己想办法
        return Command(
            update={"results": f"执行出错：{str(e)}。请尝试其他方法或询问用户。"},
            goto="agent"# 回到Agent节点，让LLM决定下一步
        )
```

---

## 三、性能监控与Token成本：每一分钱都要算清楚

### 3.1 LangSmith：你的Agent"黑匣子"

LangSmith不只是调试工具，它是生产环境的**成本监控仪表盘**。

**它能自动追踪：**

* 📊 每次调用的Token消耗和成本
* ⏱️ 响应延迟和瓶颈定位
* 🔍 完整的调用链路（Trace）
* 💰 按模型、按用户、按功能的成本分解

```
import os

# 只需配置环境变量，零代码侵入
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "production-agent"
os.environ["LANGCHAIN_API_KEY"] = "your-api-key"

# 然后正常调用你的Agent，数据自动上报
response = agent.invoke({"messages": user_input})
```

### 3.2 成本优化的四大实战技巧

根据LangSmith的数据反馈，你可以做这些优化：

**技巧1：设置Token天花板**

```
# 防止递归或循环导致Token爆炸
config = {
    "configurable": {
        "thread_id": "user_123",
        "max_tokens": 4000,  # 单次调用上限
        "max_iterations": 10# ReAct循环上限
    }
}
```

**技巧2：模型路由——该省省该花花**

```
def smart_model_router(query: str):
    """简单问题用便宜模型，复杂问题用好模型"""
    if is_simple_question(query):  # 基于关键词或分类判断
        return "gpt-4.1-mini"# $0.15/M tokens
    else:
        return "claude-sonnet-4"# $3/M tokens，但质量高

# 成本可能降低80%，但用户体验不打折
```

**技巧3：缓存高频查询**

```
from functools import lru_cache

@lru_cache(maxsize=1000)
def cached_embedding(text: str):
    """嵌入向量计算很贵，缓存能省一大笔"""
    return embedding_model.embed(text)
```

**技巧4：失败也消耗Token——要算清楚** 很多开发者不知道：**即使API调用失败，可能已经消耗了Prompt Tokens**。LangSmith能帮你识别这种"隐形浪费"。

---

## 四、并发控制与限流：别让Agent变成"洪水猛兽"

### 4.1 理解LangGraph的并发模型

LangGraph基于\*\*Super-step（超级步）\*\*执行模型：

* 同一step中没有依赖关系的节点并行执行
* 所有节点完成后才进入下一步
* 默认没有并发上限，但你可以控制

### 4.2 实战：限流配置

```
from langgraph.types import Send

# 控制并行度，防止把API打挂
config = {
    "max_concurrency": 5,  # 同时最多5个节点在执行
    "recursion_limit": 25# 防止无限循环，最多25个super-step
}

# Map-Reduce模式的动态分发
def orchestrator(state):
    tasks = state["tasks"]
    # 动态创建worker节点，但受max_concurrency限制
    return [Send("worker", {"task": t}) for t in tasks]

workflow.add_node("orchestrator", orchestrator)
workflow.add_node("worker", process_task)
```

### 4.3 生产级限流策略

```
import asyncio
from asyncio import Semaphore

# 全局信号量，控制整个应用的LLM调用频率
llm_semaphore = Semaphore(10)  # 最多10个并发LLM请求

async def rate_limited_llm_call(prompt):
    async with llm_semaphore:
        # 这里还可以加更精细的速率控制
        await asyncio.sleep(0.1)  # 100ms间隔，防止突发流量
        return await llm.ainvoke(prompt)

# 在Node中使用
async def my_node(state):
    results = await rate_limited_llm_call(state["messages"])
    return {"results": results}
```

**为什么要限流？**

* 防止触发LLM提供商的Rate Limit
* 保护下游服务（数据库、搜索API）不被打挂
* 控制成本，避免突发流量导致账单爆炸

---

## 五、生产部署Checklist

在把Agent推上线之前，对照这份清单：

| 检查项 | 状态 | 说明 |
| --- | --- | --- |
| ✅ 持久化配置 |  | 使用PostgresSaver/SqliteSaver，而非MemorySaver |
| ✅ 错误重试 |  | 配置了ModelRetryMiddleware和ToolRetryMiddleware |
| ✅ 降级策略 |  | 主模型挂了有备胎（ModelFallbackMiddleware） |
| ✅ 成本监控 |  | LangSmith已接入，设置了成本告警阈值 |
| ✅ 并发控制 |  | 配置了max\_concurrency和recursion\_limit |
| ✅ 超时设置 |  | 设置了节点级和全局超时，防止无限卡死 |
| ✅ 日志结构化 |  | 使用JSON格式日志，方便后续分析 |
| ✅ 敏感信息隔离 |  | API Keys、用户隐私数据不在日志中明文打印 |

---

## 六、本章小结

生产级Agent的核心就四个字：**稳、省、快、察**。

* **稳**：Checkpointer保证状态不丢，错误重试保证服务不崩
* **省**：Token成本实时监控，模型路由该省省该花花
* **快**：并发控制榨干性能，但不打挂下游
* **察**：LangSmith全程可观测，问题定位分钟级而非小时级

**记住**：用户不会原谅一个"有时候能用"的Agent。要么稳定可靠，要么别上生产。

---

关注公众号【dev派】，发送 "agent" 获取全部源码和模板