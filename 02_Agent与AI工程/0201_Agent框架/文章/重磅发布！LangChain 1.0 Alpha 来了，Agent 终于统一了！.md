---
title: 重磅发布！LangChain 1.0 Alpha 来了，Agent 终于统一了！
author: AI 博物院
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzNTcxOTU3OA==&mid=2247486350&idx=1&sn=986855dc33bf294c8a66b1c18a43ed33&chksm=e9b470456dc58f697826efe9c0200648d596e9caf778748493373282406b1e6b4378d21ed005&mpshare=1&scene=24&srcid=0918oLyFxyDJX2oGzTkYxu9I&sharer_shareinfo=717aa2757204b83390884116fb587a07&sharer_shareinfo_first=717aa2757204b83390884116fb587a07#rd
---

LangChain 自推出以来迅速成为开发者构建 LLM 驱动应用的重要框架之一，支持多种链（chains）、代理（agents）、工具调用、向量检索、模型接口等功能。其生态不断壮大，但也因模式多样、抽象纷繁导致开发者面对不同 “agent 模式” 时感到困惑。为提升一致性、可维护性与企业级稳定性，LangChain 在 2025 年 9 月推出了 v1.0.0 Alpha，与 LangGraph 一起迈入 1.x 版本演进。

# 为什么要推出 v1.0？

* **企业信心**：1.0 版本象征着更加稳定、信赖，降低企业采用风险。
* **抽象收敛**：过去存在多种 agent 模式（ReAct、Plan-and-Solve、React-Retry 等），v1.0 把主流用例统一迁移到一个成型的 agent 抽象上，复杂或非常规场景则推荐使用更底层、灵活的 LangGraph。
* **标准输出格式**：不同 LLM 提供商返回结构迥异，新增 `content_blocks` 属性，用以统一 reasoning、工具调用、引用、媒体类型等新 API 特性

# 核心新特性概览

### 统一代理抽象：`create_agent`

过去要实现“调用工具 → 整理结果 → 输出结构化 JSON”，要手写 chain，现在一行搞定：

```
from langchain.agents import create_agent  
from langchain_core.messages import HumanMessage  
from pydantic import BaseModel  
  
class WeatherInfo(BaseModel):  
    city: str  
    temperature_c: float  
    condition: str  
  
def fake_weather(city: str) -> str:  
    returnf"25,Sunny"if city == "Singapore"else"18,Rainy"  
  
agent = create_agent(  
    model="openai:gpt-4o-mini",  
    tools=[fake_weather],  
    response_format=WeatherInfo  
)  
  
res = agent.invoke({"messages": [  
    HumanMessage("What's the weather like in Singapore?")  
]})  
  
print(res["structured_response"])  
# WeatherInfo(city='Singapore', temperature_c=25.0, condition='Sunny')
```

#### 设计解析

* 所有旧版 agent（ReAct, Plan-and-Execute 等）均重构为 `create_agent` 的预设配置。
* 底层统一使用 `LangGraph` 作为执行引擎，暴露标准接口：`invoke(input: dict) -> dict`。
* 输入输出结构标准化：

+ 输入：`{"messages": List[BaseMessage]}`
+ 输出：`{"messages": List[BaseMessage], "structured_response": Optional[BaseModel]}`

#### 工程价值

* 降低学习成本：开发者只需掌握一个接口。
* 提升可测试性：输入输出结构固定，便于 mock 和断言。
* 支持热插拔：更换模型或工具集，不影响调用层。

### 标准化内容模型：`content_blocks`

所有 `AIMessage` 新增 `.content_blocks: List[ContentBlock]` 属性。

#### 类型定义（简化）

```
class TextBlock(ContentBlock):  
    text: str  
  
class ToolCallBlock(ContentBlock):  
    name: str  
    args: dict  
  
class CitationBlock(ContentBlock):  
    sources: List[Source]  
  
class ImageBlock(ContentBlock):  
    mime_type: str  
    data: bytes
```

#### 设计解析

* 所有 LLM 输出（文本、工具调用、引用、图像等）均封装为类型化块。
* 各提供商差异被抽象层屏蔽：

+ OpenAI `function_call` → `ToolCallBlock`
+ Anthropic `<thinking>` → `TextBlock` + `reasoning=True`
+ Claude XML 工具 → `ToolCallBlock`

* 支持懒加载：如 `ImageBlock.data` 仅在访问时解码。

#### 工程价值

* 模型切换成本趋近于零。
* 输出可被程序精确消费，无需正则/字符串匹配。
* 为多模态、流式输出、结构化引用提供扩展基础。

### Runtime：LangGraph 成为默认引擎

`create_agent` 返回的 agent 本质是一个编译后的 `LangGraph` 实例。

#### 核心能力

* **状态持久化**：对话历史、工具调用结果、中间变量存储于 `GraphState`。
* **流程控制**：

+ 条件边（conditional edges）：根据状态决定下一节点。
+ 循环控制：自动重试、最大步数限制。
+ 异常恢复：工具失败自动进入 error 节点。

* **可观测性**：内置 tracing，支持 LangSmith 集成。

#### 示例

自定义重试逻辑:

```
from langgraph.graph import StateGraph  
  
def tool_node(state):  
    try:  
        result = tool.invoke(state["input"])  
        return {"result": result, "error": None}  
    except Exception as e:  
        return {"result": None, "error": str(e)}  
  
def should_retry(state):  
    if state["error"] and state["retry_count"] < 3:  
        return"tool_node"  
    else:  
        return"end_node"  
  
graph = StateGraph(AgentState)  
graph.add_node("tool_node", tool_node)  
graph.add_conditional_edges("tool_node", should_retry)
```

#### 工程价值

* 企业级韧性：自动重试、熔断、降级。
* 复杂流程支持：审批流、多阶段任务、人工干预点。
* 调试友好：状态可序列化、可回放、可单元测试。

# 升级路径与兼容策略

### 包结构重构

| 旧模块 | 新位置 | 说明 |
| --- | --- | --- |
| `langchain.*` | `langchain.*` | 仅保留核心抽象 |
| `langchain.index` | `langchain-legacy` | 索引、社区工具等迁出 |
| `AgentExecutor` | `langchain-legacy` | 旧版 agent 执行器 |

### 渐进式迁移建议

1. **新项目**：直接使用 `create_agent` + `content_blocks`。
2. **存量项目**：

* 安装 `langchain-legacy` 保持兼容。
* 逐步替换 `initialize_agent(..., AgentType.REACT...)` → `create_react_agent(...)`（兼容层）。
* 最终迁移至 `create_agent(...)`。

3. **注意 Breaking Changes**：

* Python ≥ 3.10
* `BaseMessage.text()` → `.text`（属性）
* 默认返回 `AIMessage`，非 `str`