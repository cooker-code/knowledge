---
title: 重磅！Github Copilot开源，深度剖析核心Agent原理
author: 小智技术分享
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzNDQ5MDAzNw==&mid=2247483953&idx=1&sn=1effe59972a366426201754c7d4568e2&chksm=e9bc43cb17c5958c0ccbe363dd6f6f31d6dcdb763cda9ecac3b968a64e34ce48f02f4e316c01&mpshare=1&scene=24&srcid=11144Y2Sja0y8xziFmSQLuLw&sharer_shareinfo=b81d7cae3342468006f1ca5f5c811a9f&sharer_shareinfo_first=b81d7cae3342468006f1ca5f5c811a9f#rd
---

Github Copilot vscode chat源码重磅开源，我们对其中最核心的Agent的原理进行以下解读和分析。

# 深入剖析 editCodeIntent.ts

editCodeIntent可以说是整个代码库中最核心的文件之一。它不仅实现了所有“编辑代码”相关的功能，更重要的是，它为更高级的 AgentIntent（即 Agent 模式）提供了底层的实现基础和框架。理解了它，就等于理解了  
Copilot 是如何将语言模型的输出转化为对代码的实际修改的。

## 核心概念解析

在深入流程之前，我们必须先理解几个关键的类和概念，它们是构成整个流程的基石：

1. `IIntent` (意图):

* 是什么: 代表用户的一个高级目标或想法。例如，“我想编辑代码” (EditCodeIntent)，“我想运行一个 Agent” (AgentIntent)。
* 作用: 作为一个路由器和工厂。当用户输入指令后，系统会根据内容和上下文选择一个最合适的 IIntent 实现来处理这个请求。它的 invoke 方法会创建一个 IIntentInvocation 实例来执行具体工作。

2. `IIntentInvocation` (意图调用):

* 是什么: 一个 IIntent 的单次执行实例。它是有状态的，包含了处理单次用户请求所需的所有信息和逻辑。
* 作用: 负责执行任务的核心生命周期：buildPrompt (构建发送给 LLM 的提示) 和 processResponse (处理 LLM 返回的响应)。我们重点分析的 EditCodeIntentInvocation 就是它的一个具体实现。

3. `EditCodeStep` (编辑步骤):

* 是什么: 一个状态管理对象，专门用于跟踪一次多轮“编辑会话”的状态。
* 作用: 它维护着一个非常重要的概念——工作集 (Working Set)。工作集包含了本次编辑任务需要引用的所有文件快照。当您和 Copilot 就一个修改进行多轮对话时，EditCodeStep  
  会在后台悄悄地更新工作集和对话历史，确保 LLM 在每一步都有完整的上下文。

4. `ICodeMapperService` (代码映射服务):

* 是什么: 这是将“魔法”变为现实的关键服务。
* 作用: LLM 返回的代码通常不是完整的文件，而可能是一个带有 ... existing code ... 标记的片段，或者是一个类似 diff 的格式。CodeMapperService  
  的职责就是精确地解析这段LLM输出，并将其智能地应用（映射）到工作集中的原始文件上，计算出最终的、可执行的文本编辑操作。

5. `PromptRenderer` (提示渲染器):

* 是什么: 一个基于 TSX (TypeScript + JSX) 的模板引擎。
* 作用: 它将结构化的数据（如工作集、聊天历史、指令）和一个 TSX 模板（如 EditCodePrompt.tsx）结合起来，最终“渲染”出一段格式化好的、将要发送给 LLM 的纯文本字符串。

## 整体控制流程

EditCodeIntent 的整个生命周期可以被清晰地划分为两个主要阶段：构建提示 (Request) 和 处理响应 (Response)，处理流程如图：

### 阶段一: 构建提示 (Build Prompt)

这个阶段的目标是为 LLM 准备一份内容详尽、上下文完整的“任务说明书”。

1. 入口 (`handleRequest`):

* EditCodeIntent 的 handleRequest 方法是流程的起点。
* 它首先会检查是否有一些预处理任务，比如 \_handleCodesearch，这体现了框架的可扩展性。
* 然后，它并不自己处理复杂的请求-响应循环，而是将任务委托给一个专门的处理器：EditIntentRequestHandler。

2. 请求处理 (`EditIntentRequestHandler`):

* 这个类进一步将通用的 LLM 请求逻辑委托给 DefaultIntentRequestHandler，它封装了与 LLM 通信的标准流程。
* DefaultIntentRequestHandler 会调用 EditCodeIntentInvocation 实例的 buildPrompt 方法，正式开始构建提示。

3. 构建提示核心 (`EditCodeIntentInvocation.buildPrompt`):

* 创建 `EditCodeStep`: 这是第一件也是最重要的一件事。它会分析聊天历史和用户提供的引用（比如 @file），创建一个 EditCodeStep 实例。这个实例会建立起包含所有相关文件的工作集 (WorkingSet)。
* 准备上下文: 它将用户当前的查询、从 EditCodeStep 获得的工作集和指令、以及其他上下文信息打包。
* 渲染提示: 它实例化一个 PromptRenderer，并明确指定使用 EditCodePrompt 这个 TSX 模板。然后调用 render() 方法，将所有结构化的上下文数据“渲染”成最终的纯文本提示，发送给 LLM。

### 阶段二: 处理响应 (Process Response)

当 LLM 开始返回数据流时，这个阶段开始。目标是将 LLM 的输出转化为实际的文件修改。

1. 响应入口 (`processResponse`):

* DefaultIntentRequestHandler 接收到来自 LLM 的响应流，并将其传递给 EditCodeIntentInvocation 的 processResponse 方法。

2. 解析代码块:

* processResponse 方法不会简单地将所有返回的 Markdown 显示出来。它使用一个 CodeBlockProcessor (通过 getCodeBlocksFromResponse 函数) 来实时地解析响应流。
* CodeBlockProcessor 会识别 Markdown 中的代码块（...），并将其解析成结构化的 CodeBlock 对象。如果代码块上方有 ### path/to/file.ts  
  这样的标记，它会一并解析出来，形成一个带有关联资源的代码块。

3. 映射并应用编辑:

* 将代码块和当前会话的工作集打包成一个请求。
* 调用 ICodeMapperService.mapCode() 方法。
* CodeMapperService 在内部进行复杂的 diff 和算法分析，计算出如何将这个代码块应用到原始文件中。
* 最终，它通过 VS Code 的 API (outputStream.textEdit 或 outputStream.notebookEdit) 将一系列精确的文本编辑操作应用到工作区的文件上。

* 对于每一个带有关联资源的代码块，processResponse 会执行以下操作：

4. 完成与记录:

* 所有代码块处理完毕后，processResponse 会记录本次编辑的元数据（用于遥测和历史记录），然后结束。用户在界面上会看到代码被实时地修改。

## 与 AgentIntent 的关系

现在，最重要的部分来了：AgentIntent 继承自 `EditCodeIntent`。

这意味着 AgentIntent 复用了 `EditCodeIntent` 整个阶段二（处理响应）的能力。当 Agent 模式下的 LLM 经过多轮工具调用后，最终决定要修改文件时，它生成的也是带文件路径的代码块。AgentIntent  
会直接使用继承来的 processResponse -> CodeBlockProcessor -> CodeMapperService 这一整套成熟的流程来将代码应用到文件上。

AgentIntent 的主要区别和扩展在于阶段一（构建提示）：

* 它重写了 buildPrompt 方法。
* 它使用自己的 AgentPrompt.tsx 模板。
* 它的提示中包含了可用工具的详细描述。
* 它引入了工具调用循环 (Tool-Calling Loop)，在 buildPrompt 和 processResponse 之间增加了一个循环，直到 LLM 认为任务完成，才会生成最终的代码或文本。

## 小结

今天带大家对github copilot的agent模式核心原理进行了分析和理解，接下来我会继续解读其他的代码原理，对我们构建Agent有很好的启发作用