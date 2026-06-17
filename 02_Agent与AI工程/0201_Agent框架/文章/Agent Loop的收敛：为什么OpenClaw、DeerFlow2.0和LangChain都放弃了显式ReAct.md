---
title: Agent Loop的收敛：为什么OpenClaw、DeerFlow2.0和LangChain都放弃了显式ReAct
author: 天降陨石坑
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4MjEwNjAzNA==&mid=2257484731&idx=1&sn=19c7ba87589b2619b68f381c07658851&chksm=9dceb86e8ce1ec835a282e1f7d1bc7cf9e7e70c54f25cd2cf63a26db0777b805d0a6db210185&mpshare=1&scene=24&srcid=0409TC1MRNJO0GSZz0toJzil&sharer_shareinfo=8445a8ef242ea1e42d935c136fdb228f&sharer_shareinfo_first=8445a8ef242ea1e42d935c136fdb228f#rd
---

# 1.缘起

最近拆解不同 Agent 系统实现时，注意力都会放在谁的规划更强、谁能完成更复杂的多步任务、谁的工具调用更稳定，好的Agent框架需要直面一个基础问题，**如何稳定地“运行”起来推动复杂任务执行。**

无论是 OpenClaw 对 session 与 lifecycle 的严格工程化管理，还是 LangChain/DeepAgents 通过TodoListMiddleware拆解多步任务，还是DeerFlow2.0依靠并行分解判断是Sub-agent模式还是PlanMode模式，大家都在尝试解决同一个难题：如何让 Agent 在长链路、多步骤、不确定环境中保持可控与一致。

当把这些设计层层剥离之后，会发现一个有趣的现象——虽然上层的 Harness Engineering 分化明显，但**底层的 Agent Loop 却正在悄然收敛**。

最近我在阅读和对比不同 Agent 框架的源码与设计文档，同时也尝试从零手搓一个极简 runtime 去验证这些思路。当真正把 loop 写出来、跑起来、再不断推翻重构时，领会到现代Agent正在从显式的ReAct，走向一种以**隐式状态**为核心的通用Agent Loop模式。

# 2.快速领略从ReAct到隐式状态

## 2.1 ReAct范式回顾

在早期 Agent 实践中，几乎所有实现都建立在 **ReAct（Reason + Act）** 范式之上。它的核心思想非常直接：让模型在推理（Thought）与行动（Action）之间交替，通过不断观察环境反馈（Observation）逐步逼近任务目标。

一次典型的 ReAct 运行流程大致如下：

1. 用户输入进入上下文；
2. 模型生成包含 *Thought* 与 *Action* 的文本；
3. 解析 \_Action\_，调用对应工具；
4. 将工具结果作为 Observation 写回上下文；
5. 模型再次推理，决定下一步行动；
6. 循环直至模型输出 Final Answer。

但随着实践深入，问题也逐渐显现：

> **ReAct强耦合Prompt**。

ReAct 的运行依赖于 **对模型输出文本的解析**：需要从自然语言中提取 Action、参数与状态，这使系统对 prompt 格式高度敏感；一旦模型输出轻微偏离约定格式，整个 loop 就可能失效。

举例ReAct的prompt：

```
    prompt_template = """  
    你是ReAct智能体，需通过"思维→行动→观察"循环完成任务，严格遵循以下规则：  
    1. 思维：分析任务目标与历史轨迹，说明下一步行动的逻辑依据；  
    2. 行动：仅使用提供的工具，格式为"工具名[参数]"，支持工具：{tool_descriptions}；  
    3. 观察：根据工具反馈调整后续策略，不可仅凭记忆回答。  
  
    当前任务：{task}  
    历史轨迹：{context}  
    请输出当前步骤的思维和行动（仅输出思维和行动，无其他内容）：  
    思维：  
    行动：  
    """
```

任务流程实际上被“编码”进 **prompt**，导致可维护性与稳定性在复杂场景下迅速下降。这也是Harness Engineering中尝试将任务编排往LLM外转移或小范围可控的探索原因之一。

处理ReAct我们要根据大模型输出做很强的判定，这就是不稳定的根源。

```
     thought = llm_output.split("思维：")[1].split("行动：")[0].strip()  
        action = llm_output.split("行动：")[1].strip()  
  
        执行行动并获取观察结果  
        if action.startswith("finish["):  
            # 任务完成，提取结果  
            result = action[len("finish["):-1].strip()  
            return result, context_manager.get_context_str()
```

## 2.2 隐式状态的Agent Loop

随着模型能力与工具调用协议的发展，一种更简化的 Agent Loop 开始出现：**不再显式解析 ReAct 文本，而是让“状态”隐含在消息历史之中**。

Runtime 不再理解 *Thought* 或 \_Action\_，而只判断模型是否请求工具调用。

一个极简的隐式状态 Agent Loop，实际上可以被压缩为如下伪代码：

```
messages = [UserMessage(input)]  
  
while (true) {  
  aiMessage = model.invoke(messages)  
  messages.push(aiMessage)  
  
// 是否需要行动？  
if (!aiMessage.tool_calls?.length) {  
    return aiMessage.content  
  }  
  
// 执行工具  
for (call of aiMessage.tool_calls) {  
    result = tools.execute(call)  
    messages.push(ToolMessage(call.name, result))  
  }  
}
```

整个运行过程只有一个核心判断：

> **模型是否产生 tool\_calls。**

如果产生工具调用，runtime 执行工具并将结果追加回消息历史；如果没有，则认为任务已经完成。所谓的 *Thought → Action → Observation* 并没有消失，而是被内化为消息序列的自然演化过程——模型通过上下文自行理解“已经做了什么、接下来该做什么”。

这种设计带来的一个关键变化是：

Agent Loop 不再依赖强耦合**prompt 约定**。Runtime 无需解析特定格式的 *Thought* 或 *Action* 字符串，也不需要维护复杂的文本协议；结构化的 tool calling 本身成为执行信号，而 message history 成为唯一可信状态。Agent 从**解析模型输出**转变为**推进对话状态**，Loop 因此变得更稳定。

# 3. OpenClaw的Agent Loop

OpenClaw主要精力放在处理更高层次抽象的任务，底层Agent任务执行依赖的是pi-mono这个库。

> In OpenClaw, a loop is a single, serialized run per session that emits lifecycle and stream events as the model thinks, calls tools, and streams output.

其loop在OpenClaw的流程是：

```
intake → context → inference → tool → stream → persistence
```

真正执行在 `runEmbeddedPiAgent`（pi-agent-core 运行时）中，执行完整的 agent loop 并返回结果。AgentSession.prompt是 `@mariozechner/pi-agent-core` 的核心执行引擎，负责管理对话轮次、工具调用和事件流。

```
```ts  
  
await session.prompt(effectivePrompt, { images: imageResult.images });  
  
```  
The SDK handles the full agent loop: sending to LLM, executing tool calls, streaming responses.
```

https://github.com/openclaw/openclaw/blob/main/docs/zh-CN/concepts/agent-loop.md[1]

我们追踪到`pi-agent-core`中的源码`runLoop`方法，省去一些分支逻辑，和我们之前看到的隐式状态的loop模式完全一致。

```
 while (true) {  
let hasMoreToolCalls = true;  
  
// Inner loop: process tool calls and steering messages  
while (hasMoreToolCalls || pendingMessages.length > 0) {  
   if (!firstTurn) {  
    await emit({ type: "turn_start" });  
   } else {  
    firstTurn = false;  
   }  
   // Stream assistant response  
   const message = await streamAssistantResponse(currentContext, config, signal, emit, streamFn);  
   newMessages.push(message);  
   if (message.stopReason === "error" || message.stopReason === "aborted") {  
    await emit({ type: "turn_end", message, toolResults: [] });  
    await emit({ type: "agent_end", messages: newMessages });  
    return;  
   }  
   // Check for tool calls  
   const toolCalls = message.content.filter((c) => c.type === "toolCall");  
   hasMoreToolCalls = toolCalls.length > 0;  
   const toolResults: ToolResultMessage[] = [];  
   if (hasMoreToolCalls) {  
    toolResults.push(...(await executeToolCalls(currentContext, message, config, signal, emit)));  
    for (const result of toolResults) {  
     currentContext.messages.push(result);  
     newMessages.push(result);  
    }  
   }  
   await emit({ type: "turn_end", message, toolResults });  
   pendingMessages = (await config.getSteeringMessages?.()) || [];  
  }  
// No more messages, exit  
break;  
 }
```

https://github.com/badlogic/pi-mono/blob/a3bf1eb3992587d44ae572b465bfe652f2bfa83d/packages/agent/src/agent-loop.ts[2]

# 4. DeerFlow2.0和LangChain的Agent Loop

DeerFlow2.0拆解到Agent执行任务，依赖的是**LangGraph**，所以DeerFlow2.0和LangChain在这一层面机制是相同的。 旧版的LangChain曾经提供过ReAct模式，但是废弃了。

```
@deprecated(    
    "0.1.0",    
    message=AGENT_DEPRECATION_WARNING,    
    removal="1.0",    
)    
class ReActDocstoreAgent(Agent):    
    """Agent for the ReAct chain."""
```

https://github.com/langchain-ai/langchain/blob/50febb79/libs/langchain/langchain\_classic/agents/react/base.py#L31-L36[3]

现在推荐使用 `create_agent` 替代传统方式。

```
[`create_agent`][langchain.agents.create_agent] from the `langchain`  
package, which provides a more flexible agent factory with middleware  
support, structured output, and integration with LangGraph.
```

`create_agent`底层使用LangGraph以图的形式组织任务，在 `_make_model_to_tools_edge` 函数中实现了关键的循环控制逻辑。

```
def _make_model_to_tools_edge(state: dict[str, Any]) -> str | list[Send] | None:    
    # 1. 检查是否有显式的 jump_to    
    if jump_to := state.get("jump_to"):    
        return _resolve_jump(jump_to, ...)    
        
    # 2. 如果没有 AIMessage，退出循环    
    if last_ai_message isNone:    
        return end_destination    
        
    # 3. 如果模型没有调用工具，退出循环  
    if len(last_ai_message.tool_calls) == 0:    
        return end_destination    
        
    # 4. 如果有待处理的工具调用，跳转到工具节点    
    if pending_tool_calls:    
        return [Send("tools", ...) for tool_call in pending_tool_calls]    
        
    # 5. 如果有结构化响应，退出循环    
    if"structured_response"in state:    
        return end_destination    
        
    # 6. 否则回到模型节点继续循环    
    return model_destination
```

https://github.com/langchain-ai/langchain/blob/50febb79/libs/langchain\_v1/langchain/agents/factory.py[4]

在新版 LangChain 中，显式 ReAct Agent 已逐步被统一的 `create_agent` 所取代。Runtime 不再解析 Thought/Action 文本，而是仅检查最新 `AIMessage` 是否包含 `tool_calls`：若存在则执行工具并追加 `ToolMessage`，随后再次调用模型。这使 ReAct 从一种显式执行框架，演化为基于消息历史的隐式状态循环，Agent Loop 因而更加通用与稳定。

# 5. 隐式状态 Agent Loop 的方向

## 为什么Agent Loop会收敛

隐式 Agent Loop 的出现，并不是框架设计上的偶然选择，而是模型能力演进后的自然结果。

* 一方面，LLM 的推理与上下文理解**能力显著提升**，已经能够在连续对话中自行维持简单步骤与中间目标，无需 runtime 显式维护 Thought 或计划结构；
* 另一方面，结构化 **tool calling** 成为模型原生能力，使“是否需要行动”可以通过协议而非文本解析来判断。

当模型既能维持短程规划，又能以结构化方式请求外部动作时，Agent Loop 便可以被极度简化，runtime 只需推进消息状态，而不再负责理解任务流程。

## 长程任务的演化方向

通用的隐式 Agent Loop 已经证明可以稳定支撑单一任务执行，因此当前创新的焦点正在从 loop 本身转移到 **Harness Engineering**。

正如开头所说，OpenClaw、DeerFlow2.0和LangChain选择在更高层解决复杂多步问题，任务拆分为多个可控执行单元，再由上层机制决定协作方式。

复杂性不再完全依赖模型即时推理，而是逐步外移到可工程化、可观测、可干预的执行控制层之中。

## 6.小结

回看 Agent 的演进路径，显式 ReAct 曾是连接模型与行动世界的桥梁，随着模型能力提升以及tool calling原生能力普及，如何让推理被稳定地运行与约束成为了工程重点。隐式状态的 Agent Loop 将这种机制内化，使 runtime 从流程控制者转变为状态推进者。

当底层 loop 开始趋同，Agent 架构的竞争焦点也随之上移，是谁能更好地管理状态、规划与长期执行的不确定性，这也是**Harness Engineering**的价值所在。

参考资料

[1] 

OpenClaw agent-llop concept: *https://github.com/openclaw/openclaw/blob/main/docs/zh-CN/concepts/agent-loop.md*

[2] 

pi-mono agent-loop: *https://github.com/badlogic/pi-mono/blob/a3bf1eb3992587d44ae572b465bfe652f2bfa83d/packages/agent/src/agent-loop.ts*

[3] 

Deprecated ReAct Agent: *https://github.com/langchain-ai/langchain/blob/50febb79/libs/langchain/langchain\_classic/agents/react/base.py#L31-L36*

[4] 

\_make\_model\_to\_tools\_edge: *https://github.com/langchain-ai/langchain/blob/50febb79/libs/langchain\_v1/langchain/agents/factory.py*