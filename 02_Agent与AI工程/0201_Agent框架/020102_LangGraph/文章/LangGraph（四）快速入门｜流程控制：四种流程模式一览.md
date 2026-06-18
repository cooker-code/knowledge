---
title: LangGraph（四）快速入门｜流程控制：四种流程模式一览
author: 大数据知识库
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMTYzMTEzMg==&mid=2247485244&idx=1&sn=9b92febc61a3ae8f6da5bcbcf355df6b&chksm=c38ae2c0c8caaca089e7c7eced6ffc771e82be38f5ab98215a4aecb4551cda7198a5cf1ba84f&mpshare=1&scene=24&srcid=0415hYis41q2SBdNxXbByu0q&sharer_shareinfo=360e055443d612f2a629c82523602cc5&sharer_shareinfo_first=360e055443d612f2a629c82523602cc5#rd
---

# 流程控制

> LangGraph 通过**条件边实现分支决策**、**回边形成循环**、**扇出（Fan-out）触发并行**、**Send API 支持动态并行**、**子图实现模块化**，这五种机制组合可以构建任意复杂度的 AI 工作流。

## 概念速览

| 概念/术语 | 一句话解释 | 补充说明 |
| --- | --- | --- |
| 条件边 | 根据状态动态选择下一个节点 | `add_conditional_edges` |
| 路由函数 | 返回下一节点名称的函数 | 接收状态，返回节点名或列表 |
| 超级步 | 图执行的一次迭代单位 | 并行节点在同一超级步执行 |
| 递归限制 | 防止无限循环的保护机制 | 默认 1000 步（v1.0.6+） |
| Send API | 动态创建并行分支的工具 | 用于 Map-Reduce 模式 |
| 子图 | 作为节点嵌入的独立图 | 模块化封装 |

## 核心理解

> 把 LangGraph 的流程控制想象成**导航系统**——普通边是"直行"，条件边是"根据路况选择左转或右转"，循环是"绕圈找车位"，并行是"兵分三路同时探索"，子图是"进入商场内部的小地图"。这些机制组合起来，就能规划任意复杂的路线。

## 核心要点

•**核心结论**：条件边 + 循环 + 并行 是构建复杂 Agent 工作流的三大支柱

•**关键机制**：路由函数决定走向，Reducer 处理并行结果合并，递归限制防止死循环

•**适用边界**：简单流程用普通边，决策用条件边，迭代用循环，独立任务用并行

### 1. 条件分支：动态路由

条件边根据当前状态决定下一步执行哪个节点：

```
from langgraph.graph import StateGraph, START, END

def route_question(state) -> str:
    """路由函数：返回下一节点名称"""
    if "技术" in state["question"]:
        return "tech"
    return "sales"

workflow = StateGraph(State)
workflow.add_node("classify", classify_node)
workflow.add_node("tech", tech_node)
workflow.add_node("sales", sales_node)

workflow.add_edge(START, "classify")

# 核心 API
workflow.add_conditional_edges(
    "classify",           # 起始节点
    route_question,       # 路由函数
    {                     # 路由映射
        "tech": "tech",
        "sales": "sales"
    }
)

workflow.add_edge("tech", END)
workflow.add_edge("sales", END)
```

### 2. 循环迭代：反复优化

条件边指向上游节点形成循环，需要明确终止条件：

```
def should_continue(state) -> str:
    """判断是否继续循环"""
    if state["quality_score"] >= 0.8:
        return "done"      # 结束循环
    if state["iteration"] >= state["max_iterations"]:
        return "done"      # 强制终止
    return "improve"       # 继续循环

workflow.add_edge(START, "generate")
workflow.add_edge("generate", "check")

# 形成循环的关键
workflow.add_conditional_edges(
    "check",
    should_continue,
    {
        "improve": "generate",  # 回边 → 形成循环
        "done": END
    }
)
```

> **关键理解**：递归限制是安全保障（默认 1000 步），但应在路由函数中设计明确的退出条件，而非依赖强制终止。

### 3. 并行执行：多分支并发

**触发方式一：多条出边**

```
workflow.add_edge("source", "worker_a")
workflow.add_edge("source", "worker_b")
workflow.add_edge("source", "worker_c")
# → 三个 worker 在同一超级步并行执行
```

**触发方式二：路由返回列表**

```
def route_to_multiple(state) -> list[str]:
    return ["analyzer", "summarizer", "validator"]

workflow.add_conditional_edges("source", route_to_multiple)
```

**结果合并：使用 Reducer**

```
from typing import Annotated
import operator

class State(TypedDict):
    # 并行结果通过 Reducer 合并
    results: Annotated[list[str], operator.add]
```

### 4. Send API：动态并行（Map-Reduce）

当并行数量运行时才确定时，使用 Send API：

```
from langgraph.types import Send

def dispatch_tasks(state):
    """为每个任务创建独立的执行分支"""
    return [
        Send("worker", {"task_id": t["id"], "data": t["data"]})
        for t in state["tasks"]  # 任务数量运行时确定
    ]

workflow.add_conditional_edges("dispatcher", dispatch_tasks)
```

### 5. 子图嵌套：模块化设计

将复杂逻辑封装为子图，作为节点添加到父图：

```
# 定义子图
sub_workflow = StateGraph(SubState)
sub_workflow.add_node("process", process_node)
sub_workflow.add_edge(START, "process")
sub_workflow.add_edge("process", END)
subgraph = sub_workflow.compile()

# 子图作为节点
main_workflow = StateGraph(MainState)
main_workflow.add_node("sub_process", subgraph)
```

## 核心流程

## 关键差异

| 维度 | 普通边 | 条件边 | Send API |
| --- | --- | --- | --- |
| **语法** | `add_edge(A, B)` | `add_conditional_edges(A, func, map)` | `return [Send(...)]` |
| **路由** | 固定 | 动态（路由函数） | 动态（运行时） |
| **并行** | 多出边时并行 | 返回列表时并行 | 每个 Send 一个分支 |
| **状态** | 共享 | 共享 | 可独立 |
| **场景** | 顺序流程 | 决策分支 | Map-Reduce |

## 实战示例：智能客服路由

以下是一个完整的智能客服路由示例，综合运用条件分支实现动态分流：

```
from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict

class State(TypedDict):
    question: str
    category: str
    answer: str

def classify(state):
    q = state["question"].lower()
    if "退款" in q: return {"category": "refund"}
    if "物流" in q: return {"category": "shipping"}
    return {"category": "general"}

def handle_refund(state):
    return {"answer": "退款专员为您服务"}

def handle_shipping(state):
    return {"answer": "物流客服为您服务"}

def handle_general(state):
    return {"answer": "通用客服为您服务"}

def route(state) -> str:
    return state["category"]

# 构建图
workflow = StateGraph(State)
workflow.add_node("classify", classify)
workflow.add_node("refund", handle_refund)
workflow.add_node("shipping", handle_shipping)
workflow.add_node("general", handle_general)

workflow.add_edge(START, "classify")
workflow.add_conditional_edges("classify", route, {
    "refund": "refund",
    "shipping": "shipping",
    "general": "general"
})
workflow.add_edge("refund", END)
workflow.add_edge("shipping", END)
workflow.add_edge("general", END)

app = workflow.compile()
result = app.invoke({"question": "我要退款", "category": "", "answer": ""})
print(result["answer"])  # 退款专员为您服务
```

## 注意事项

•循环必须有可达的退出路径，否则会触发递归限制

•并行节点更新同一字段时，必须定义 Reducer 处理合并

•递归限制可通过 `config={"recursion_limit": 50}` 调整

•路由函数返回值必须在 `path_map` 中有对应项（或直接是节点名）

•Send API 创建的分支有独立状态，适合处理互不干扰的任务

---

## LLM 知识库

大模型应用开发系列 完整的笔记已更新到知识库里，关注本公众号 **【大数据知识库】** 获取：

> 这篇整理了核心要点，方便快速过一遍。想深入的话，参考 【LangGraph（四）知识详解｜流程控制：条件分支、循环迭代、并行执行与子图嵌套】。