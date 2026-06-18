> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020116_企业级Agent平台/020116_核心知识点/意图识别与Routine约束稳定器|意图识别与Routine约束稳定器]]
---
title: Parlant：一个让AI Agent 真正听话的开源框架
author: AI工程化
date:
url: https://mp.weixin.qq.com/s?__biz=MzA5MTIxNTY4MQ==&mid=2461154289&idx=1&sn=1d6c71d7b4d5b60f8a8047845cf0e1f7&chksm=8637fd788e60047bd87c26cfe0e9d911d4ea2a240510bf956f099f92002180a6287d8e6231c1&mpshare=1&scene=24&srcid=0912X5YzadzLHhMWfpORcZdT&sharer_shareinfo=b3053a3c554b2f88c4c7e30f91f631d5&sharer_shareinfo_first=b3053a3c554b2f88c4c7e30f91f631d5#rd
---

AI Agent最大的痛点是什么？不听话。

Stop building AI agents that ignore your instructions" ， 这句话精准击中了每个AI开发者的痛处。

你花时间写了详细的系统提示词，测试环境运行完美，一到生产环境就开始自由发挥。用户问A，它答B，关键时刻还会胡编乱造。这是所有AI开发者都遇到过的问题。

Parlant这个开源框架专门解决这个问题。它提出与其指望LLM自觉遵守规则，不如从框架层面确保合规执行。

## 核心差异

传统方法是你通过写Prompt规则然后希望按预期执行，而Parlant是定义清晰的指导原则并通过上下文匹配保证执行。

对比代码：

```
# 传统方式：希望LLM听话
system_prompt = "你是助手，请遵循这些规则..."

# Parlant方式：确保执行
await agent.create_guideline(
    condition="用户询问退款",
    action="先检查订单状态判断是否符合退款条件",
    tools=[check_order_status],
)
```

区别在于，前者是"希望"，后者是"保证"。

## 主要功能模块

**Guidelines（行为准则）**定义的规则会被上下文匹配和强制执行。不是装饰性的提示词，是真正的行为控制机制。

**Journeys（对话流程）**当用户偏离预设对话路径时，系统能自动调整和引导，确保对话朝目标方向进行。

**Playground（调试环境）**提供完整的测试和调试界面，可以实时观察Agent的决策过程。

**Widget（生产界面）**即插即用的聊天UI组件，可以直接集成到任何Web应用中。

**Tool Integration（工具集成）**将外部API、数据库或后端服务绑定到特定的交互事件上，确保工具调用的准确性。

**Explainability（可解释性）**每个决策都有详细的执行日志，可以追溯为什么触发了某个准则。

## 快速上手

安装和基础使用：

```
pip install parlant

import parlant.sdk as p

@p.tool
asyncdef get_weather(context: p.ToolContext, city: str) -> p.ToolResult:
    # Your weather API logic here
    return p.ToolResult(f"Sunny, 72°F in {city}")

@p.tool
asyncdef get_datetime(context: p.ToolContext) -> p.ToolResult:
    from datetime import datetime
    return p.ToolResult(datetime.now())

asyncdef main():
    asyncwith p.Server() as server:
        agent = await server.create_agent(
            name="WeatherBot",
            description="Helpful weather assistant"
        )

        # Have the agent's context be updated on every response (though
        # update interval is customizable) using a context variable.
        await agent.create_variable(name="current-datetime", tool=get_datetime)

        # Control and guide agent behavior with natural language
        await agent.create_guideline(
            condition="User asks about weather",
            action="Get current weather and provide a friendly response with suggestions",
            tools=[get_weather]
        )

        # Add other (reliably enforced) behavioral modeling elements
        # ...

        # 🎉 Test playground ready at http://localhost:8800
        # Integrate the official React widget into your app,
        # or follow the tutorial to build your own frontend!

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

## 适用场景

这个框架特别适合对AI可靠性要求极高的场景：

* **金融服务**：合规优先设计，内置风险管控
* **医疗健康**：支持HIPAA标准，保护患者数据
* **电商平台**：大规模客服自动化，订单处理流程
* **法律科技**：精确的法律指导，文档审查辅助

## 小结

Parlant走上了与其它框架不一样的路，与其期待大模型的指令遵从能力提高，不如从框架规则角度出发解决现实困境。在一些金融场景中已经落地，摩根大通的AI团队负责人评价："目前见过最优雅的对话AI框架，用Parlant开发是种享受。” 它的定位不再是一个快速生成演示的AI Wrapper，而是能够解决生产环境AI可控性问题的基础设施。

但同时它这种模式或许也将会是模型下一次迭代的牺牲品，不论如何，想要在现阶段落地严肃场景的开发者们尝试尝试。

项目地址：https://github.com/emcie-co/parlant

官方文档：https://www.parlant.io/docs/quickstart/installation