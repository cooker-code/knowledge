---
title: LangChain v1.0 让AI工作流更加立体
author: 乔治的美好生活
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI2NDM2ODkwMw==&mid=2247484098&idx=1&sn=698aae5f0811620efff2c92657f4c8a6&chksm=ebe639e48e34fc4025da5cb30a29888fe76d51183dd1fc42fb782a850d45fb92394df4904d00&mpshare=1&scene=24&srcid=1220BvZXOMZ53cBqpKj0Tb2k&sharer_shareinfo=c6516612769727e88fbbb1c5bd8577fe&sharer_shareinfo_first=c6516612769727e88fbbb1c5bd8577fe#rd
---

LangChain 是AI辅助工作的主流框架。最近它进行了大版本更新，语法变得更加优雅。准备用他做一个数据分析流程正好学习了一下，在这里把核心的几个变化进行总结。我们从create\_agent类进行出发，介绍其几个核心功能：tool和Middleware。

# create\_agent

create\_agent是 v1.0 的核心入口，一个函数搞定 Agent 创建：

```
from langchain.agents import create_agent  
agent = create_agent( model=llm, 模型 tools=[...], # 工具：让 AI 能做事 middleware=[...], # 中间件：事前/事后处理 system_prompt="...", # 系统提示词)  
result = agent.invoke({ "messages": [{"role": "user", "content": "你的问题"}]})
```

核心组成：

```
┌─────────────────────────────────────────────────┐│ create_agent │├─────────────────────────────────────────────────┤│ model → 大语言模型 ││ tools → 工具列表（让 AI 能做事） ││ middleware → 中间件（事前检查/事后处理） │└─────────────────────────────────────────────────┘
```

下面分别介绍Tool和Middleware的用法。

Tool = Agent 的"手脚"，让 AI 能执行实际操作（查天气、调 API、操作数据库...），像不像热门词汇MCP

## 1.1 定义

```
from langchain.tools import tool  
@tooldef get_weather(city: str) -> str: """获取城市天气""" ← docstring 决定 AI 何时调用 return f"{city}：晴天 25°C"
```

三要素：

| 要素 | 说明 |
| --- | --- |
| 函数名 | 简洁明了，如 `get_weather` |
| **docstring** | **最重要！AI 据此判断何时调用** |
| 参数类型 | 帮助 AI 正确传参 |

## 1.2 使用

```
agent = create_agent( model=llm, tools=[get_weather, calculate], 传入工具列表)
```

## 1.3 完整示例

```
from langchain.agents import create_agentfrom langchain.tools import toolfrom langchain_community.chat_models import ChatTongyi  
@tooldef get_weather(city: str) -> str: """获取城市天气""" weathers = {"北京": "晴天 25°C", "上海": "多云 28°C"} return weathers.get(city, f"{city}：暂无数据")  
@tooldef calculate(expression: str) -> str: """计算数学表达式，如 1+2*3""" return f"结果是 {eval(expression)}"  
llm = ChatTongyi(model_name="qwen-plus")  
agent = create_agent( model=llm, tools=[get_weather, calculate],)  
  
AI 会自动选择合适的工具result = agent.invoke({ "messages": [{"role": "user", "content": "北京天气怎么样？"}]})print(result["messages"][-1].content)
```

# 二、Middleware 中间件

Middleware = 在模型调用前后插入自定义逻辑，这个可以大大优化逻辑，让业务逻辑和功能逻辑解耦，非常好用。langchain也不在使用链式语言，归功于中间件的建构。

## 2.1 定义

```
from langchain.agents.middleware import AgentMiddleware  
class MyMiddleware(AgentMiddleware):  
 def before_model(self, state, runtime): """模型调用前""" return None 返回 None = 放行  
 def after_model(self, state, runtime): """模型返回后""" return None # 返回 None = 不修改结果
```

关键点：

| 返回值 | 效果 |
| --- | --- |
| `None` | 放行 / 不修改 |
| `{"jump_to": "end"}` | **拦截，跳过模型调用** |
| `{"messages": [...]}` | 修改消息内容 |

## 2.2 使用

```
agent = create_agent( model=llm, tools=[...], middleware=[MyMiddleware()], ← 直接加到列表)
```

## 2.3 示例：脏话过滤

```
from langchain.agents import create_agentfrom langchain.agents.middleware import AgentMiddlewarefrom langchain.messages import AIMessagefrom langchain_community.chat_models import ChatTongyi  
class BadWordFilter(AgentMiddleware): """脏话过滤中间件"""  
 BAD_WORDS = ["傻瓜", "笨蛋", "垃圾"]  
 def before_model(self, state, runtime): text = state["messages"][-1].content  
 for word in self.BAD_WORDS: if word in text: 检测到脏话，拦截并返回提示 return { "messages": state["messages"] + [ AIMessage(content="请文明用语哦") ], "jump_to": "end" # 跳过模型，直接结束 }  
 return None # 放行  
llm = ChatTongyi(model_name="qwen-plus")  
agent = create_agent( model=llm, tools=[], middleware=[BadWordFilter()],)  
# 测试result = agent.invoke({ "messages": [{"role": "user", "content": "你好"}]})print(result["messages"][-1].content) # 正常回复  
result = agent.invoke({ "messages": [{"role": "user", "content": "你这个傻瓜"}]})print(result["messages"][-1].content) # "请文明用语哦"
```

# 三、调用方式

| 场景 | 写法 |
| --- | --- |
| 简单问答 | `model.invoke("问题").content` |
| 需要工具 | `create_agent(tools=[...])` |
| 需要拦截/日志 | `create_agent(middleware=[...])` |

```
简单问答：直接调用from langchain_community.chat_models import ChatTongyi  
llm = ChatTongyi(model_name="qwen-plus")result = llm.invoke("什么是 Python？")print(result.content)
```

这个句式真是舒服了，之前需要定义llm，定义调用函数，定义输出结构。：）

# 四、create\_agent 参数

```
agent = create_agent( model=llm, 模型实例 tools=[...], # 工具列表 system_prompt="...", # 系统提示词 middleware=[...], # 中间件列表 memory=True, # 启用记忆 max_iterations=10, # 最大迭代次数（防止死循环）)
```

```
| 参数 | 说明 || ---------------- | ----------------------------------- || `model` | 模型实例 || `tools` | 工具列表，可为空 `[]` || `system_prompt` | 设定 AI 角色/风格 || `middleware` | 中间件列表，事前/事后处理 || `memory` | 启用后配合 `thread_id` 实现多轮对话 || `max_iterations` | 防止 AI 无限调用工具 |
```

# 五、完整示例

```
"""综合示例：Tool + Middleware + Agent"""from langchain.agents import create_agentfrom langchain.agents.middleware import AgentMiddlewarefrom langchain.tools import toolfrom langchain.messages import AIMessagefrom langchain_community.chat_models import ChatTongyi  
  
========== 1. 定义工具 ==========  
@tooldef get_weather(city: str) -> str: """获取城市天气""" weathers = {"北京": "晴天 25°C", "上海": "多云 28°C"} return weathers.get(city, f"{city}：暂无数据")  
@tooldef calculate(expression: str) -> str: """计算数学表达式""" return f"结果是 {eval(expression)}"  
  
# ========== 2. 定义中间件 ==========  
class BadWordFilter(AgentMiddleware): """脏话过滤"""  
 def before_model(self, state, runtime): text = state["messages"][-1].content if any(w in text for w in ["傻瓜", "笨蛋"]): return { "messages": state["messages"] + [ AIMessage(content="请文明用语") ], "jump_to": "end" } return None  
  
# ========== 3. 创建 Agent ==========  
llm = ChatTongyi(model_name="qwen-plus")  
agent = create_agent( model=llm, tools=[get_weather, calculate], system_prompt="你是一个helpful助手，简洁回答", middleware=[BadWordFilter()], max_iterations=10,)  
  
# ========== 4. 使用 ==========  
# 查天气result = agent.invoke({ "messages": [{"role": "user", "content": "北京天气如何？"}]})print(result["messages"][-1].content)  
# 计算result = agent.invoke({ "messages": [{"role": "user", "content": "123 * 456 等于多少？"}]})print(result["messages"][-1].content)  
# 脏话测试（会被拦截）result = agent.invoke({ "messages": [{"role": "user", "content": "你这个傻瓜"}]})print(result["messages"][-1].content) # "请文明用语"
```

最后做一个总结：TOOL可以引入外部操作函数，让AI去进行一些功能实现，Middleware则是大大简化了AI的业务流程。

我理解的Agent=LLM+MCP，大脑加上肢体，协调的工作。

```
┌─────────────────────────────────────────────────┐│ Tool = 让 AI 能做事 ││ Middleware = 在调用前后加逻辑 ││ Agent = Tool + Middleware 的容器 │└─────────────────────────────────────────────────┘
```