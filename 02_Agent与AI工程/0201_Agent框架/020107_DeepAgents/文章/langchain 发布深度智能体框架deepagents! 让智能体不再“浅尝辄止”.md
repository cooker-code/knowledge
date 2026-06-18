---
title: langchain 发布深度智能体框架deepagents! 让智能体不再“浅尝辄止”
author: AI 博物院
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzNTcxOTU3OA==&mid=2247486619&idx=1&sn=69772f997c122fa13d20bb5e55a54c94&chksm=e9ad7722d9908cb1510939139e446bfaaa187fe3acdc5f45fef70713e4eb67c91b56aa5f7d3f&mpshare=1&scene=24&srcid=0922TW9leTQ2jG2MInjG815n&sharer_shareinfo=6c34e3fda8c98f7a89d3dfacd539f2b0&sharer_shareinfo_first=6c34e3fda8c98f7a89d3dfacd539f2b0#rd
---

很多人做智能体时，会用“大模型 + 循环调用工具”的最简单架构。它好用，但常常很“浅”，一遇到复杂、长链路任务就容易跑偏、忘事、或停在半路。

像 Deep Research、Manus、Claude Code 这类“深度”智能体，是怎么补上的？核心其实就四件事：

* **规划工具**：先想清楚要做什么，再一步步做。
* **子智能体**：把复杂任务拆给更专精的小助手。
* **文件系统**：能读写文件，保留中间成果和上下文。
* **详细提示词**：把工作方法讲清楚，少走弯路。

# deepagents 是什么

`deepagents` 是一个 Python 包，把上面这四件事做成了通用能力，帮你更容易地搭出“深”智能体。它受 Claude Code 启发很深，目标是更通用、更好用。

deep agent

* **安装**：

```
pip install deepagents
```

* 如果要跑下面的入门示例，还需要：

```
pip install tavily-python
```

# 一个简洁的入门示例

```
import os  
from typing import Literal  
from tavily import TavilyClient  
from deepagents import create_deep_agent  
  
tavily_client = TavilyClient(api_key=os.environ["TAVILY_API_KEY"])  
  
def internet_search(  
    query: str,  
    max_results: int = 5,  
    topic: Literal["general", "news", "finance"] = "general",  
    include_raw_content: bool = False,  
):  
    """Run a web search"""  
    return tavily_client.search(  
        query,  
        max_results=max_results,  
        include_raw_content=include_raw_content,  
        topic=topic,  
    )  
  
research_instructions = """You are an expert researcher. Your job is to conduct thorough research, and then write a polished report.  
  
You have access to a few tools.  
  
## `internet_search`  
  
Use this to run an internet search for a given query. You can specify the number of results, the topic, and whether raw content should be included.  
"""  
  
agent = create_deep_agent(  
    [internet_search],  
    research_instructions,  
)  
  
result = agent.invoke({"messages": [{"role": "user", "content": "what is langgraph?"}]})
```

这个 `agent` 本质上就是一个 LangGraph 图，所以你可以用 LangGraph 的常用能力（流式、HITL、人类介入、记忆、Studio 等）。

# 自定义一个“深”智能体

* **tools（必填）**：一组函数或 LangChain 的 `@tool`。主智能体和子智能体都能用。
* **instructions（必填）**：这会成为提示词的一部分（系统提示词已内置，会和它一起起作用）。
* **subagents（选填）**：自定义子智能体，做专门的子任务。

子智能体有两种写法：

1. 简单版 `SubAgent`

* 必填字段：`name`（名字）、`description`（说明）、`prompt`（提示词）
* 可选字段：`tools`（可用工具，默认继承全部）、`model_settings`（该子智能体独立的模型设置）

```
research_subagent = {  
    "name": "research-agent",  
    "description": "Used to research more in depth questions",  
    "prompt": sub_research_prompt,  
}  
  
agent = create_deep_agent(  
    tools,  
    prompt,  
    subagents=[research_subagent]  
)
```

2. 进阶版 `CustomSubAgent`

* 直接把一个预先构建好的 LangGraph 图当作子智能体用：

```
from langgraph.prebuilt import create_react_agent  
  
custom_graph = create_react_agent(  
    model=your_model,  
    tools=specialized_tools,  
    prompt="You are a specialized agent for data analysis..."  
)  
  
custom_subagent = {  
    "name": "data-analyzer",  
    "description": "Specialized agent for complex data analysis tasks",  
    "graph": custom_graph  
}  
  
agent = create_deep_agent(  
    tools,  
    prompt,  
    subagents=[custom_subagent]  
)
```

# 模型怎么配

* **默认模型**：`"claude-sonnet-4-20250514"`
* 你可以传任意 LangChain 模型对象作为默认模型；也可以为某个子智能体单独指定模型与参数。

示例：用 Ollama 的自定义模型

```
from deepagents import create_deep_agent  
from langchain.chat_models import init_chat_model  
  
model = init_chat_model(model="ollama:gpt-oss:20b")  
  
agent = create_deep_agent(  
    tools=tools,  
    instructions=instructions,  
    model=model,  
)
```

示例：给“评审子智能体”单独上一个更快、更稳的模型

```
critique_sub_agent = {  
    "name": "critique-agent",  
    "description": "Critique the final report",  
    "prompt": "You are a tough editor.",  
    "model_settings": {  
        "model": "anthropic:claude-3-5-haiku-20241022",  
        "temperature": 0,  
        "max_tokens": 8192  
    }  
}  
  
agent = create_deep_agent(  
    tools=[internet_search],  
    instructions="You are an expert researcher...",  
    model="claude-sonnet-4-20250514",  
    subagents=[critique_sub_agent],  
)
```

# 内置工具

默认自带 5 个工具（可通过 `builtin_tools` 精简）：

* **write\_todos**：写待办（帮助“先计划，再执行”）
* **write\_file**：写文件（虚拟文件系统）
* **read\_file**：读文件
* **ls**：列文件
* **edit\_file**：编辑文件

精简示例（只保留待办工具）：

```
builtin_tools = ["write_todos"]  
agent = create_deep_agent(..., builtin_tools=builtin_tools, ...)
```

# 关键部件

* **系统提示词（System Prompt）**  
  已内置，参考了 Claude Code 的风格，又更通用。它把“怎么规划、怎么用文件、怎么调用子智能体”等规则说清楚。好的提示词，是深度的关键。
* **规划工具（Planning Tool）**  
  类似 Claude Code 的 TodoWrite。它不直接“做事”，而是先把计划写下来，放在上下文里，帮助后续执行。
* **虚拟文件系统（File System Tools）**  
  提供 `ls/read_file/write_file/edit_file`，用 LangGraph 的 State 模拟，不会动到真实磁盘，方便在一台机上开多个智能体也不相互影响。  
  目前支持一层目录；可以通过 State 中的 `files` 注入和读取。
* **子智能体（Sub Agents）**  
  内置一个通用子智能体（和主智能体同指令、同工具），也支持你自定义多个专门子智能体。好处是“隔离上下文”、“专人做专事”。
* **人机协同（Human-in-the-Loop）**  
  你可以给某些工具加“人工审批”拦截（`interrupt_config`）。支持：

+ **allow\_accept**：直接执行
+ **allow\_edit**：改工具或改参数再执行
+ **allow\_respond**：不执行，追加一条“工具消息”作为反馈  
  需要配一个检查点（如 `InMemorySaver`）。当前一次只能拦截一个并行工具调用。

* **MCP 工具**  
  通过 LangChain MCP Adapter 可以让 deepagents 使用 MCP 工具（注意使用 async 版本）。
* **配置化智能体（Configurable Agent）**  
  用 `create_configurable_agent`，把 `instructions/subagents` 等做成可配的构建器，方便在 `langgraph.json` 里部署和更新。也有 async 版本。

# 适合做什么

* **深度研究**：查资料、比对观点、整合成文。
* **代码助手**：规划变更、读改文件、分派子任务、总结提交。
* **数据/文档处理**：拆分任务、多步加工、阶段性落盘。
* **流程自动化**：有计划、有记忆、有分工、更可靠。

# 总结

建议先从小规模开始，逐步引入工具、子智能体和文件系统，并编写清晰的提示词指导智能体“先想后做、不确定时提问、记录关键步骤”。关键操作加入人工审核，并针对不同任务选用不同模型以平衡效果与成本。通过这些改进，原有的工具循环型智能体将变得更耐心、稳定，能更好地完成复杂任务。