> 已吸收至：[[02_Agent与AI工程/0208_Context Engineering/0208_核心知识点/上下文工程治理与边界|上下文工程治理与边界]]
---
title: LangChain 1.0 版本入门，agent 创建和 context engineering 都简单多了
author: 说人话的算法工程师
date:
url: https://mp.weixin.qq.com/s?__biz=MzI5OTAwNzQ2MQ==&mid=2455846572&idx=1&sn=336b5a9e909db82336135d7ce5e9a10f&chksm=fa05ff4c38d320515dd308400c5619913f7a8096c899a47dd6176125df30c8b86f552d11b52d&mpshare=1&scene=24&srcid=1128GdQd12RY4ydXNIRfS04f&sharer_shareinfo=5dbffb2ce788e49e965dfd33c6834a5e&sharer_shareinfo_first=5dbffb2ce788e49e965dfd33c6834a5e#rd
---

LangChain 最近更新了1.0版本，其中最重要的两个内容是 **create\_agent** 和 **Middleware**。

# create\_agent

create\_agent 接口让我们可以用几行代码就创建一个**带工具调用的 ReAct agent**。

```
from langchain.agents import create_agent

agent = create_agent(
    model="claude-sonnet-4-5-20250929",
    tools=[search_web, analyze_data, send_email],
    system_prompt="You are a helpful research assistant."
)

result = agent.invoke({
    "messages": [
        {"role": "user", "content": "Research AI safety trends"}
    ]
})
```

它的实现原理是，每次循环都去判断结果中**是否带有 tool\_call** 的要求，如果有就循环，直到没有了就结束。

agent

从源码可以看到 model 和 tool 节点之间定义了两条边。

model to tools

tools to model

# Middleware基本思路

Middleware 可以让我们在 ReAct 框架下做 Context Engineering 变得非常方便。在 ReAct 框架的“思考→行动→观察”循环模式下，我们经常需要在**相邻两个步骤之间**做一些处理，这正是你可能听说过的 context engineering。在 LangChain 1.0版本之前，我们只能写很多硬编码的逻辑去实现，修改和维护起来比较乱。

image

Middleware 使用了**切面编程**的思想，设定了6个hook：before\_model, after\_model, wrap\_model\_call, wrap\_tool\_call, after\_model, after\_agent。我们可以根据需要，把需要处理的逻辑放到合适的时机。

# 内置 Middleware

LangChain 已经有一些内置的 middleware，

* HumanInTheLoopMiddleware 可以实现一些高危的操作人工确认，比如读写文件等
* ModelCallLimitMiddleware，ToolCallLimitMiddleware 可以实现请求重试次数限制，防止重试次数过多
* ContextEditingMiddleware 可以做上下文的清理和编辑

如果定义了多个 Middleware

```
agent = create_agent(
    model="gpt-4o",
    middleware=[middleware1, middleware2, middleware3],
    tools=[...],
)
```

他们之间的先后顺序如下：

启动agent前

* 1.middleware1.before\_agent()
* 2.middleware2.before\_agent()
* 3.middleware3.before\_agent()

Agent开始

* 4.middleware1.before\_model()
* 5.middleware2.before\_model()
* 6.middleware3.before\_model()

调用模型

* 7.middleware1.wrap\_model\_call() → middleware2.wrap\_model\_call() → middleware3.wrap\_model\_call() → model

模型调用结束

* 8.middleware3.after\_model()
* 9.middleware2.after\_model()
* 10.middleware1.after\_model()

Agent 结束

* 11.middleware3.after\_agent()
* 12.middleware2.after\_agent()
* 13.middleware1.after\_agent()

#

# 应用举例

下面两个例子我们可以体会 ReAct **多轮循环**的运行过程。

## 应用1：智能问答

定义两个工具，一个查询知识库，一个网页搜索。第一轮对话先查询知识库，如果查不到第二轮再去做网页搜索。

## 应用2：智能客服

引入 human-in-the-loop 环节，如果用户提问（查询最近的医院）缺少关键信息（用户目前的定位在哪），系统会要求用户补充，下一轮再执行搜索。

上面两张图的代码在文末参考链接中的 towardsai 文章中， 没有用到1.0版本新的API，大家可以看到处理节点之间流转的代码有很多。如果用 create\_agent 可以把图相关的代码简化掉，另外 tool call 的调用也比**判断输出中是否包含特定的字符串**优雅。

## 参考

https://docs.langchain.com/oss/python/releases/changelog

https://pub.towardsai.net/agentic-ai-build-react-agent-using-langgraph-facac8ae6031

https://docs.langchain.com/oss/python/langchain/middleware#built-in-middleware