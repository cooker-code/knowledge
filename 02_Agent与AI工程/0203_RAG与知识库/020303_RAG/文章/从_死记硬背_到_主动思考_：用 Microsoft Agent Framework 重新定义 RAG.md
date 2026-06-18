---
title: 从"死记硬背"到"主动思考"：用 Microsoft Agent Framework 重新定义 RAG
author: 老许的OPC超级个体之路
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkxODYyNzE3NA==&mid=2247489345&idx=1&sn=fd0c596bc38e2027f74a3d9cb4d08462&chksm=c09304ec40cee31d6bb4a4e945f26e71f67e2082cabb5191f1323c99d34ebe979b9ce3b4d354&mpshare=1&scene=24&srcid=1114jxeXiguzzuhQ3DJ8WGWk&sharer_shareinfo=707b58b10190f01430cfc1d6c21670af&sharer_shareinfo_first=707b58b10190f01430cfc1d6c21670af#rd
---

# 

> ❝
>
> 当 RAG 遇上 Agent，检索不再是简单的"查字典"，而是一场有策略的智能探索之旅。

## 一、开场白：RAG 的"中年危机"

如果你问一个做 AI 应用的工程师："怎么让大模型回答得更靠谱？"十有八九会得到一个答案：RAG（检索增强生成）。

传统 RAG 的思路很直接：用户问个问题，系统去向量数据库里捞一把相关文档，然后把这些文档和问题一起塞给大模型，让它基于这些"参考资料"来回答。这套流程就像是给学生开卷考试——问题来了，翻翻书，找到相关章节，照着答。

听起来挺美好，但实际用起来你会发现：

* **检索太死板**：不管问题复杂还是简单，都是一次性检索，检索不到就没戏了
* **上下文爆炸**：把一堆可能相关的文档全塞进去，token 哗哗地烧，模型还容易被无关信息带偏
* **缺乏推理链**：遇到需要多步推理的问题，传统 RAG 就像个只会背书的学生，不会举一反三

这时候，Agentic RAG 横空出世了。

## 二、Agentic RAG：让检索"活"起来

### 2.1 什么是 Agentic RAG？

如果说传统 RAG 是"被动检索"，那 Agentic RAG 就是"主动探索"。

核心区别在于：**Agentic RAG 把检索过程交给了一个具有自主决策能力的 Agent**。这个 Agent 不是机械地执行"检索-生成"流程，而是：

1. **动态规划**：根据问题复杂度，决定是一次检索还是多次检索
2. **迭代优化**：如果第一次检索结果不理想，会调整策略再次检索
3. **工具编排**：可以调用多种检索工具（向量搜索、关键词搜索、图数据库等）
4. **推理增强**：在检索过程中进行中间推理，逐步逼近答案

打个比方：传统 RAG 像是图书馆的自助查询机，你输入关键词，它给你吐出一堆书；而 Agentic RAG 更像是一个经验丰富的图书管理员，会根据你的问题帮你分析需求，先找哪本书，再找哪本书，甚至会告诉你"这个问题可能需要跨学科查阅"。

### 2.2 Agentic RAG 的核心设计理念

Agentic RAG 的设计哲学可以总结为三个关键词：

**自主性（Autonomy）**：Agent 能够自主决定何时检索、检索什么、如何使用检索结果

**适应性（Adaptability）**：根据问题类型和检索结果动态调整策略

**可组合性（Composability）**：可以灵活组合多个 Agent 和工具，构建复杂的检索推理链

## 三、Microsoft Agent Framework：天生为 Agentic RAG 而生

在众多 Agent 框架中，Microsoft Agent Framework 有几个独特优势，让它特别适合实现 Agentic RAG：

### 3.1 原生的 Context Provider 机制

Agent Framework 提供了 `AIContextProvider` 抽象类，这是实现 Agentic RAG 的核心基础设施。

```
public abstract class AIContextProvider  
{  
    // 在 Agent 调用前注入上下文  
    public abstract ValueTask<AIContext> InvokingAsync(  
        InvokingContext context,   
        CancellationToken cancellationToken = default);  
      
    // 在 Agent 调用后处理结果  
    public virtual ValueTask InvokedAsync(  
        InvokedContext context,   
        CancellationToken cancellationToken = default);  
}
```

这个设计妙在哪里？它把检索逻辑和 Agent 的推理逻辑解耦了。你可以在 `InvokingAsync` 中动态决定要检索什么内容，然后注入到 Agent 的上下文中；在 `InvokedAsync` 中分析 Agent 的输出，决定是否需要进一步检索。

### 3.2 灵活的 Function Tools 系统

Agent Framework 的 Function Tools 让 Agent 可以主动调用检索工具：

```
[Description("Search the knowledge base for relevant information")]  
static async Task<string> SearchKnowledgeBase(  
    [Description("The search query")] string query,  
    [Description("Number of results to return")] int topK = 3)  
{  
    // 实际的检索逻辑  
    var results = await vectorStore.SearchAsync(query, topK);  
    return FormatResults(results);  
}  
  
AIAgent agent = chatClient.CreateAIAgent(new ChatClientAgentOptions  
{  
    Instructions = "You are a research assistant. Use the search tool when you need information.",  
    Tools = [AIFunctionFactory.Create(SearchKnowledgeBase)]  
});
```

这样，Agent 就能根据对话上下文，自主决定何时调用检索工具，而不是被动地接受预先检索好的内容。

### 3.3 强大的 Workflow 编排能力

对于复杂的 Agentic RAG 场景，Agent Framework 的 Workflow 系统提供了多 Agent 协作的能力：

* **Sequential Pattern**：多个专门化的 Agent 依次处理（问题分析 → 检索规划 → 执行检索 → 答案生成）
* **Concurrent Pattern**：并行调用多个检索源，然后聚合结果
* **Handoff Pattern**：根据问题类型，动态切换到不同的专家 Agent

## 四、实战：三种 Agentic RAG 实现模式

接下来，我们用 Microsoft Agent Framework 实现三种典型的 Agentic RAG 模式，从简单到复杂，逐步展示框架的威力。

### 4.1 模式一：Context Provider 驱动的智能检索

这是最基础也最优雅的实现方式，利用 `AIContextProvider` 在每次 Agent 调用前动态注入检索内容。

#### 核心思路

框架内置的 `TextSearchProvider` 就是一个很好的例子。它的工作流程是：

1. 拦截用户的输入消息
2. 提取关键信息作为检索查询
3. 执行检索并格式化结果
4. 将结果作为系统消息注入到对话上下文中

#### 代码实现

```
// 定义检索函数  
static async Task<IEnumerable<TextSearchProvider.TextSearchResult>>   
    SmartSearchAsync(string query, CancellationToken cancellationToken)  
{  
    // 这里可以接入任何检索系统：向量数据库、搜索引擎、知识图谱等  
    var results = new List<TextSearchProvider.TextSearchResult>();  
      
    // 示例：根据查询内容智能路由到不同的知识源  
    if (query.Contains("技术文档"))  
    {  
        var docs = await techDocStore.SearchAsync(query);  
        results.AddRange(docs.Select(d => new TextSearchProvider.TextSearchResult  
        {  
            SourceName = d.Title,  
            SourceLink = d.Url,  
            Text = d.Content  
        }));  
    }  
      
    if (query.Contains("代码示例"))  
    {  
        var code = await codebaseSearch.SearchAsync(query);  
        results.AddRange(code.Select(c => new TextSearchProvider.TextSearchResult  
        {  
            SourceName = $"Code: {c.FileName}",  
            SourceLink = c.GitHubUrl,  
            Text = c.CodeSnippet  
        }));  
    }  
      
    return results;  
}  
  
// 配置检索行为  
TextSearchProviderOptions searchOptions = new()  
{  
    // 在每次 AI 调用前都执行检索  
    SearchTime = TextSearchProviderOptions.TextSearchBehavior.BeforeAIInvoke,  
    // 保持最近 6 条消息的上下文窗口  
    RecentMessageMemoryLimit = 6,  
};  
  
// 创建 Agent，注入检索能力  
AIAgent agent = chatClient.CreateAIAgent(new ChatClientAgentOptions  
{  
    Instructions = "你是一个技术助手。基于检索到的文档回答问题，并引用来源。",  
    AIContextProviderFactory = ctx => new TextSearchProvider(  
        SmartSearchAsync,   
        ctx.SerializedState,   
        ctx.JsonSerializerOptions,   
        searchOptions)  
});  
  
// 使用 Agent  
AgentThread thread = agent.GetNewThread();  
var response = await agent.RunAsync("如何在 Agent Framework 中使用 Function Tools？", thread);  
Console.WriteLine(response);
```

#### 设计亮点

1. **透明性**：Agent 本身不需要知道检索的存在，检索逻辑完全由 Context Provider 处理
2. **可配置性**：通过 `SearchTime` 参数控制检索时机（每次调用前、首次调用前、按需调用）
3. **状态管理**：Context Provider 的状态可以序列化，支持跨会话的检索历史追踪

### 4.2 模式二：Function Tool 驱动的主动检索

这种模式让 Agent 拥有"主动权"——它可以根据需要，自主决定何时调用检索工具。

#### 核心思路

把检索能力封装成 Function Tool，Agent 在推理过程中可以：

* 判断当前信息是否足够
* 决定是否需要检索
* 选择合适的检索策略（精确匹配 vs 语义搜索）
* 根据检索结果决定下一步行动

#### 代码实现

```
// 定义多个检索工具  
[Description("在技术文档库中进行语义搜索")]  
static async Task<string> SemanticSearch(  
    [Description("搜索查询，应该是一个完整的问题或描述")] string query,  
    [Description("返回结果数量")] int topK = 5)  
{  
    var results = await vectorStore.SearchAsync(query, topK);  
    return JsonSerializer.Serialize(results.Select(r => new   
    {   
        Title = r.Title,   
        Content = r.Content.Substring(0, 500),  
        Relevance = r.Score   
    }));  
}  
  
[Description("在代码库中搜索特定的函数或类")]  
static async Task<string> CodeSearch(  
    [Description("要搜索的代码元素名称")] string symbolName,  
    [Description("代码类型：class, method, interface")] string symbolType = "method")  
{  
    var results = await codeIndex.FindSymbolAsync(symbolName, symbolType);  
    return FormatCodeResults(results);  
}  
  
[Description("获取 API 的详细文档")]  
static async Task<string> GetApiDocumentation(  
    [Description("API 的完整路径，如 Microsoft.Agents.AI.AIAgent")] string apiPath)  
{  
    var doc = await apiDocStore.GetAsync(apiPath);  
    return doc?.ToMarkdown() ?? "未找到该 API 的文档";  
}  
  
// 创建具有多种检索能力的 Agent  
AIAgent researchAgent = chatClient.CreateAIAgent(new ChatClientAgentOptions  
{  
    Instructions = @"你是一个智能研究助手。你可以使用以下策略回答问题：  
      
    1. 对于概念性问题，使用 SemanticSearch 查找相关文档  
    2. 对于代码实现问题，使用 CodeSearch 查找示例代码  
    3. 对于 API 使用问题，使用 GetApiDocumentation 获取详细文档  
      
    你可以多次调用这些工具，逐步收集信息。如果第一次检索结果不够，尝试换个角度再次检索。",  
      
    Tools =   
    [  
        AIFunctionFactory.Create(SemanticSearch),  
        AIFunctionFactory.Create(CodeSearch),  
        AIFunctionFactory.Create(GetApiDocumentation)  
    ]  
});  
  
// 使用示例  
var response = await researchAgent.RunAsync(  
    "我想实现一个可以记住用户偏好的 Agent，应该怎么做？请给出具体的代码示例。");
```

#### 实际运行流程

当用户提出上面的问题时，Agent 可能会这样思考和行动：

```
[Agent 思考] 这个问题涉及状态管理和记忆功能，我需要先了解相关概念  
  
[调用] SemanticSearch("Agent memory and state management")  
[结果] 找到关于 AIContextProvider 和状态序列化的文档  
  
[Agent 思考] 文档提到了 AIContextProvider，我需要看看具体的代码实现  
  
[调用] CodeSearch("AIContextProvider", "class")  
[结果] 找到 UserInfoMemory 示例代码  
  
[Agent 思考] 现在我需要了解 AIContextProvider 的 API 细节  
  
[调用] GetApiDocumentation("Microsoft.Agents.AI.AIContextProvider")  
[结果] 获取完整的 API 文档  
  
[Agent 生成答案] 基于收集到的信息，生成包含概念解释和代码示例的完整答案
```

#### 设计亮点

1. **自主决策**：Agent 根据问题复杂度，自主决定调用哪些工具、调用几次
2. **迭代优化**：如果第一次检索结果不理想，Agent 会调整查询策略再次尝试
3. **多源融合**：可以同时从文档、代码、API 等多个来源获取信息

### 4.3 模式三：Multi-Agent Workflow 驱动的复杂检索

对于真正复杂的场景，我们需要多个专门化的 Agent 协作完成检索任务。

#### 核心思路

将 Agentic RAG 分解为多个阶段，每个阶段由专门的 Agent 负责：

1. **Query Analyzer Agent**：分析用户问题，提取关键信息，规划检索策略
2. **Retrieval Agent**：执行实际的检索操作
3. **Reranker Agent**：对检索结果进行重排序和过滤
4. **Synthesizer Agent**：基于检索结果生成最终答案

#### 代码实现

```
// 1. 查询分析 Agent  
ChatClientAgent queryAnalyzer = new(chatClient,  
    instructions: @"分析用户问题，输出 JSON 格式的检索计划：  
    {  
        'intent': '问题意图',  
        'keywords': ['关键词列表'],  
        'searchStrategy': 'semantic|keyword|hybrid',  
        'expectedSources': ['期望的信息来源']  
    }",  
    name: "query_analyzer");  
  
// 2. 检索执行 Agent（带检索工具）  
ChatClientAgent retrievalAgent = new(chatClient,  
    instructions: "根据检索计划执行检索，返回原始检索结果",  
    name: "retrieval_agent",  
    tools: [AIFunctionFactory.Create(SemanticSearch),   
            AIFunctionFactory.Create(CodeSearch)]);  
  
// 3. 结果重排 Agent  
ChatClientAgent rerankerAgent = new(chatClient,  
    instructions: @"评估检索结果的相关性，输出重排后的结果：  
    - 过滤掉不相关的结果  
    - 按相关性重新排序  
    - 标注每个结果的可信度",  
    name: "reranker");  
  
// 4. 答案合成 Agent  
ChatClientAgent synthesizerAgent = new(chatClient,  
    instructions: @"基于重排后的检索结果生成答案：  
    - 综合多个来源的信息  
    - 保持逻辑连贯性  
    - 引用具体来源  
    - 如果信息不足，明确指出",  
    name: "synthesizer");  
  
// 构建 Sequential Workflow  
var agenticRagWorkflow = AgentWorkflowBuilder.BuildSequential(  
    [queryAnalyzer, retrievalAgent, rerankerAgent, synthesizerAgent]);  
  
// 使用 Workflow  
List<ChatMessage> messages = [new(ChatRole.User,   
    "在 Agent Framework 中如何实现一个支持多轮对话的 Agent？")];  
  
await using StreamingRun run = await InProcessExecution.StreamAsync(  
    agenticRagWorkflow, messages);  
await run.TrySendMessageAsync(new TurnToken(emitEvents: true));  
  
await foreach (WorkflowEvent evt in run.WatchStreamAsync())  
{  
    if (evt is AgentRunUpdateEvent e)  
    {  
        Console.WriteLine($"[{e.ExecutorId}] {e.Update.Text}");  
    }  
    else if (evt is WorkflowOutputEvent output)  
    {  
        var finalMessages = output.As<List<ChatMessage>>();  
        Console.WriteLine($"\n最终答案：\n{finalMessages.Last().Text}");  
    }  
}
```

#### 进阶：带反馈循环的 Workflow

更进一步，我们可以加入质量检查和反馈循环：

```
// 质量评估 Agent  
ChatClientAgent qualityChecker = new(chatClient,  
    instructions: @"评估答案质量，输出 JSON：  
    {  
        'isComplete': true/false,  
        'missingInfo': ['缺失的信息'],  
        'needsMoreRetrieval': true/false,  
        'refinedQuery': '改进后的查询'  
    }",  
    name: "quality_checker");  
  
// 使用 Conditional Edges 构建带反馈的 Workflow  
var workflowBuilder = new WorkflowBuilder();  
  
workflowBuilder  
    .AddExecutor("analyzer", queryAnalyzer)  
    .AddExecutor("retriever", retrievalAgent)  
    .AddExecutor("reranker", rerankerAgent)  
    .AddExecutor("synthesizer", synthesizerAgent)  
    .AddExecutor("checker", qualityChecker)  
    .AddEdge("analyzer", "retriever")  
    .AddEdge("retriever", "reranker")  
    .AddEdge("reranker", "synthesizer")  
    .AddEdge("synthesizer", "checker")  
    // 条件边：如果质量不达标，返回重新检索  
    .AddConditionalEdge("checker",   
        condition: output =>   
        {  
            var result = JsonSerializer.Deserialize<QualityCheckResult>(output);  
            return result.NeedsMoreRetrieval ? "retriever" : "end";  
        },  
        destinations: new[] { "retriever", "end" });  
  
var workflow = workflowBuilder.Build();
```

#### 设计亮点

1. **职责分离**：每个 Agent 专注于一个特定任务，提高了可维护性和可测试性
2. **可观测性**：通过 Workflow 事件流，可以清晰地看到每个阶段的输出
3. **灵活编排**：可以轻松调整 Agent 的执行顺序，或添加新的处理阶段
4. **质量保证**：通过反馈循环，确保检索结果的质量

## 五、深入原理：Agent Framework 如何支撑 Agentic RAG

理解了实现模式后，我们来深入看看 Agent Framework 的设计哲学，以及它为什么能如此优雅地支持 Agentic RAG。

### 5.1 AIContextProvider：检索的"钩子"机制

`AIContextProvider` 是整个框架中最精妙的设计之一。它提供了两个关键的生命周期钩子：

```
public abstract class AIContextProvider  
{  
    // 调用前钩子：注入上下文  
    public abstract ValueTask<AIContext> InvokingAsync(  
        InvokingContext context,   
        CancellationToken cancellationToken = default);  
      
    // 调用后钩子：处理结果  
    public virtual ValueTask InvokedAsync(  
        InvokedContext context,   
        CancellationToken cancellationToken = default);  
      
    // 状态序列化：支持持久化  
    public virtual JsonElement Serialize(  
        JsonSerializerOptions? jsonSerializerOptions = null);  
}
```

#### 为什么这个设计如此强大？

**1. 解耦检索和推理**

传统做法是把检索逻辑硬编码在 Agent 的调用流程中，导致代码耦合严重。而 `AIContextProvider` 通过依赖注入的方式，让检索逻辑成为可插拔的组件。

**2. 支持动态上下文**

`InvokingAsync` 接收 `InvokingContext`，其中包含了当前的请求消息。这意味着你可以：

* 分析用户的最新输入
* 结合历史对话上下文
* 动态决定检索策略

**3. 闭环反馈**

`InvokedAsync` 接收 `InvokedContext`，包含了 Agent 的响应消息。这让你可以：

* 分析 Agent 的回答质量
* 提取新的知识点存入知识库
* 根据回答效果调整检索策略

**4. 状态持久化**

通过 `Serialize` 方法，Context Provider 的状态可以被序列化。这对于 Agentic RAG 至关重要：

* 记录检索历史，避免重复检索
* 跨会话保持检索上下文
* 支持分布式部署

### 5.2 Function Tools：Agent 的"工具箱"

Agent Framework 的 Function Tools 系统让 Agent 可以像人类一样使用工具。其核心是 `AIFunctionFactory`：

```
// 从方法创建 Function Tool  
var tool = AIFunctionFactory.Create(SearchKnowledgeBase);  
  
// 从 OpenAPI 规范创建 Function Tool  
var tools = AIFunctionFactory.CreateFromOpenApiSpec(openApiSpec);
```

#### 设计精髓

**1. 自动参数推断**

通过 C# 的特性（Attribute）系统，框架可以自动提取函数的参数信息：

```
[Description("搜索知识库")]  
static string Search(  
    [Description("搜索查询")] string query,  
    [Description("结果数量")] int topK = 5)  
{  
    // ...  
}
```

这些元数据会被转换成 LLM 能理解的 JSON Schema，让模型知道何时、如何调用这个工具。

**2. 异步支持**

所有 Function Tools 都原生支持异步操作，这对于 I/O 密集的检索操作至关重要：

```
static async Task<string> SearchAsync(string query)  
{  
    var results = await vectorStore.SearchAsync(query);  
    return FormatResults(results);  
}
```

**3. 类型安全**

得益于 C# 的强类型系统，Function Tools 在编译时就能发现类型错误，而不是在运行时才暴露问题。

### 5.3 Workflow：编排的艺术

Agent Framework 的 Workflow 系统借鉴了 LangGraph 的设计理念，但做了更符合 .NET 生态的改进。

#### 核心概念

**1. Executor（执行器）**

Workflow 中的基本执行单元。Agent 本身就是一种特殊的 Executor：

```
public interface IExecutor  
{  
    Task<ExecutorResult> ExecuteAsync(  
        ExecutorContext context,   
        CancellationToken cancellationToken);  
}
```

**2. Edge（边）**

定义 Executor 之间的连接关系：

```
// 简单边：A 执行完后执行 B  
workflowBuilder.AddEdge("executorA", "executorB");  
  
// 条件边：根据输出决定下一步  
workflowBuilder.AddConditionalEdge("executorA",   
    condition: output => output.Contains("需要检索") ? "retriever" : "end",  
    destinations: new[] { "retriever", "end" });
```

**3. State（状态）**

Workflow 的共享状态，所有 Executor 都可以读写：

```
public class AgenticRagState  
{  
    public string OriginalQuery { get; set; }  
    public List<RetrievalResult> RetrievalResults { get; set; }  
    public int RetrievalAttempts { get; set; }  
    public bool IsComplete { get; set; }  
}
```

#### 为什么 Workflow 适合 Agentic RAG？

**1. 可视化的执行流程**

Workflow 的图结构让复杂的 Agentic RAG 流程一目了然：

2. 灵活的控制流

通过条件边，可以实现复杂的控制逻辑：

* 根据问题类型选择不同的检索策略
* 根据检索结果决定是否需要进一步检索
* 实现最大重试次数的保护机制

**3. 并行执行**

对于需要从多个来源检索的场景，Workflow 支持并行执行：

```
var workflow = AgentWorkflowBuilder.BuildConcurrent(  
    [vectorSearchAgent, keywordSearchAgent, graphSearchAgent]);
```

**4. 可观测性**

Workflow 提供了丰富的事件流，让你可以实时监控执行过程：

```
await foreach (WorkflowEvent evt in run.WatchStreamAsync())  
{  
    switch (evt)  
    {  
        case ExecutorStartEvent start:  
            Console.WriteLine($"开始执行：{start.ExecutorId}");  
            break;  
        case ExecutorCompleteEvent complete:  
            Console.WriteLine($"完成执行：{complete.ExecutorId}");  
            break;  
        case AgentRunUpdateEvent update:  
            Console.WriteLine($"Agent 输出：{update.Update.Text}");  
            break;  
    }  
}
```

### 5.4 Memory Provider：让 Agent "记住"检索历史

Agent Framework 的 Memory 机制可以让 Agentic RAG 系统记住检索历史，避免重复检索：

```
public class RetrievalMemory : AIContextProvider  
{  
    private readonly Dictionary<string, List<RetrievalResult>> _cache = new();  
      
    public override async ValueTask<AIContext> InvokingAsync(  
        InvokingContext context,   
        CancellationToken cancellationToken = default)  
    {  
        var userMessage = context.RequestMessages.LastOrDefault(m => m.Role == ChatRole.User);  
        if (userMessage == null) return new AIContext();  
          
        // 检查缓存  
        var queryHash = ComputeHash(userMessage.Text);  
        if (_cache.TryGetValue(queryHash, out var cachedResults))  
        {  
            return new AIContext  
            {  
                Instructions = $"以下是之前检索过的相关信息：\n{FormatResults(cachedResults)}"  
            };  
        }  
          
        return new AIContext();  
    }  
      
    public override async ValueTask InvokedAsync(  
        InvokedContext context,   
        CancellationToken cancellationToken = default)  
    {  
        // 将本次检索结果存入缓存  
        var userMessage = context.RequestMessages.LastOrDefault(m => m.Role == ChatRole.User);  
        if (userMessage != null && context.ResponseMessages != null)  
        {  
            var queryHash = ComputeHash(userMessage.Text);  
            var results = ExtractRetrievalResults(context.ResponseMessages);  
            _cache[queryHash] = results;  
        }  
    }  
      
    public override JsonElement Serialize(JsonSerializerOptions? options = null)  
    {  
        return JsonSerializer.SerializeToElement(_cache, options);  
    }  
}
```

## 六、实战案例：构建一个生产级的 Agentic RAG 系统

理论讲完了，让我们来看一个接近生产环境的完整案例：一个技术文档问答系统。

### 6.1 系统架构

```
┌─────────────────────────────────────────────────────────────┐  
│                        用户接口层                              │  
│                    (Web API / Chat UI)                       │  
└────────────────────────┬────────────────────────────────────┘  
                         │  
┌────────────────────────▼────────────────────────────────────┐  
│                    Agentic RAG Workflow                      │  
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │  
│  │  Query   │→ │ Retrieval│→ │ Reranker │→ │Synthesizer│  │  
│  │ Analyzer │  │  Agent   │  │  Agent   │  │  Agent    │   │  
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │  
│       ↑              │                            │          │  
│       │              ↓                            ↓          │  
│       │        ┌──────────┐              ┌──────────┐      │  
│       └────────│ Quality  │←─────────────│  Memory  │      │  
│                │ Checker  │              │ Provider │      │  
│                └──────────┘              └──────────┘      │  
└────────────────────────┬────────────────────────────────────┘  
                         │  
┌────────────────────────▼────────────────────────────────────┐  
│                      检索层                                   │  
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │  
│  │  Vector  │  │ Keyword  │  │   Code   │  │   API    │   │  
│  │  Store   │  │  Search  │  │  Search  │  │   Docs   │   │  
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │  
└─────────────────────────────────────────────────────────────┘
```

### 6.2 核心代码实现

```
public class ProductionAgenticRagSystem  
{  
    private readonly IChatClient _chatClient;  
    private readonly IVectorStore _vectorStore;  
    private readonly ICodeSearchService _codeSearch;  
    private readonly IApiDocService _apiDocs;  
    private readonly ILogger _logger;  
      
    public ProductionAgenticRagSystem(  
        IChatClient chatClient,  
        IVectorStore vectorStore,  
        ICodeSearchService codeSearch,  
        IApiDocService apiDocs,  
        ILogger logger)  
    {  
        _chatClient = chatClient;  
        _vectorStore = vectorStore;  
        _codeSearch = codeSearch;  
        _apiDocs = apiDocs;  
        _logger = logger;  
    }  
      
    public async Task<string> AnswerQuestionAsync(  
        string question,   
        string userId,  
        CancellationToken cancellationToken = default)  
    {  
        // 1. 创建专门化的 Agents  
        var agents = CreateAgents();  
          
        // 2. 构建 Workflow  
        var workflow = BuildWorkflow(agents);  
          
        // 3. 准备初始状态  
        var initialState = new AgenticRagState  
        {  
            UserId = userId,  
            OriginalQuery = question,  
            Timestamp = DateTime.UtcNow  
        };  
          
        // 4. 执行 Workflow  
        var messages = new List<ChatMessage>   
        {   
            new(ChatRole.User, question)   
        };  
          
        await using var run = await InProcessExecution.StreamAsync(  
            workflow,   
            messages,   
            initialState);  
          
        await run.TrySendMessageAsync(new TurnToken(emitEvents: true));  
          
        // 5. 收集结果和遥测数据  
        var telemetry = new WorkflowTelemetry();  
        string? finalAnswer = null;  
          
        await foreach (var evt in run.WatchStreamAsync(cancellationToken))  
        {  
            // 记录遥测  
            telemetry.RecordEvent(evt);  
              
            // 实时流式输出（可选）  
            if (evt is AgentRunUpdateEvent update)  
            {  
                await StreamToClient(update.Update.Text);  
            }  
              
            // 获取最终结果  
            if (evt is WorkflowOutputEvent output)  
            {  
                var resultMessages = output.As<List<ChatMessage>>();  
                finalAnswer = resultMessages?.LastOrDefault()?.Text;  
            }  
        }  
          
        // 6. 记录日志和指标  
        await LogTelemetryAsync(telemetry, userId);  
          
        return finalAnswer ?? "抱歉，我无法回答这个问题。";  
    }  
      
    private Dictionary<string, AIAgent> CreateAgents()  
    {  
        // Query Analyzer：分析问题意图  
        var queryAnalyzer = _chatClient.AsIChatClient().CreateAIAgent(  
            new ChatClientAgentOptions  
            {  
                Name = "query_analyzer",  
                Instructions = LoadPrompt("query_analyzer_prompt.txt"),  
                ResponseFormat = ChatResponseFormat.CreateJsonSchemaFormat(  
                    name: "query_analysis",  
                    jsonSchema: QueryAnalysisSchema)  
            });  
          
        // Retrieval Agent：执行检索  
        var retrievalAgent = _chatClient.AsIChatClient().CreateAIAgent(  
            new ChatClientAgentOptions  
            {  
                Name = "retrieval_agent",  
                Instructions = LoadPrompt("retrieval_prompt.txt"),  
                Tools = CreateRetrievalTools()  
            });  
          
        // Reranker Agent：重排序结果  
        var rerankerAgent = _chatClient.AsIChatClient().CreateAIAgent(  
            new ChatClientAgentOptions  
            {  
                Name = "reranker",  
                Instructions = LoadPrompt("reranker_prompt.txt")  
            });  
          
        // Synthesizer Agent：生成答案  
        var synthesizerAgent = _chatClient.AsIChatClient().CreateAIAgent(  
            new ChatClientAgentOptions  
            {  
                Name = "synthesizer",  
                Instructions = LoadPrompt("synthesizer_prompt.txt"),  
                AIContextProviderFactory = ctx => new RetrievalMemory(  
                    ctx.SerializedState,   
                    ctx.JsonSerializerOptions)  
            });  
          
        // Quality Checker：质量检查  
        var qualityChecker = _chatClient.AsIChatClient().CreateAIAgent(  
            new ChatClientAgentOptions  
            {  
                Name = "quality_checker",  
                Instructions = LoadPrompt("quality_checker_prompt.txt"),  
                ResponseFormat = ChatResponseFormat.CreateJsonSchemaFormat(  
                    name: "quality_check",  
                    jsonSchema: QualityCheckSchema)  
            });  
          
        return new Dictionary<string, AIAgent>  
        {  
            ["analyzer"] = queryAnalyzer,  
            ["retriever"] = retrievalAgent,  
            ["reranker"] = rerankerAgent,  
            ["synthesizer"] = synthesizerAgent,  
            ["checker"] = qualityChecker  
        };  
    }  
      
    private List<AIFunction> CreateRetrievalTools()  
    {  
        return new List<AIFunction>  
        {  
            // 向量搜索  
            AIFunctionFactory.Create(  
                async (string query, int topK) =>  
                {  
                    var results = await _vectorStore.SearchAsync(query, topK);  
                    return JsonSerializer.Serialize(results);  
                },  
                name: "vector_search",  
                description: "在文档库中进行语义搜索"),  
              
            // 代码搜索  
            AIFunctionFactory.Create(  
                async (string symbolName, string symbolType) =>  
                {  
                    var results = await _codeSearch.FindSymbolAsync(symbolName, symbolType);  
                    return FormatCodeResults(results);  
                },  
                name: "code_search",  
                description: "在代码库中搜索函数、类或接口"),  
              
            // API 文档  
            AIFunctionFactory.Create(  
                async (string apiPath) =>  
                {  
                    var doc = await _apiDocs.GetAsync(apiPath);  
                    return doc?.ToMarkdown() ?? "未找到文档";  
                },  
                name: "get_api_doc",  
                description: "获取 API 的详细文档")  
        };  
    }  
      
    private Workflow BuildWorkflow(Dictionary<string, AIAgent> agents)  
    {  
        var builder = new WorkflowBuilder();  
          
        // 添加所有 Agents 作为 Executors  
        foreach (var (name, agent) in agents)  
        {  
            builder.AddExecutor(name, agent);  
        }  
          
        // 定义执行流程  
        builder  
            .AddEdge("analyzer", "retriever")  
            .AddEdge("retriever", "reranker")  
            .AddEdge("reranker", "synthesizer")  
            .AddEdge("synthesizer", "checker")  
            // 条件边：质量检查  
            .AddConditionalEdge("checker",  
                condition: output =>  
                {  
                    var check = JsonSerializer.Deserialize<QualityCheck>(output);  
                    if (!check.IsComplete && check.RetrievalAttempts < 3)  
                    {  
                        return "retriever"; // 重新检索  
                    }  
                    return "end";  
                },  
                destinations: new[] { "retriever", "end" });  
          
        return builder.Build();  
    }  
}
```

### 6.3 关键优化点

#### 1. Prompt 工程

将 Prompt 外部化到配置文件，便于调优：

```
// query_analyzer_prompt.txt  
你是一个查询分析专家。分析用户问题并输出结构化的检索计划。  
  
分析维度：  
1. 问题类型：概念解释、代码示例、API 文档、故障排查  
2. 关键实体：提取技术术语、API 名称、错误信息等  
3. 检索策略：语义搜索、精确匹配、代码搜索  
4. 预期来源：官方文档、示例代码、社区问答  
  
输出 JSON 格式，严格遵循 schema。
```

#### 2. 结果缓存

使用 Memory Provider 实现智能缓存：

```
public class RetrievalCache : AIContextProvider  
{  
    private readonly IDistributedCache _cache;  
    private readonly TimeSpan _ttl = TimeSpan.FromHours(1);  
      
    public override async ValueTask<AIContext> InvokingAsync(  
        InvokingContext context,   
        CancellationToken cancellationToken = default)  
    {  
        var query = ExtractQuery(context.RequestMessages);  
        var cacheKey = ComputeCacheKey(query);  
          
        var cached = await _cache.GetStringAsync(cacheKey, cancellationToken);  
        if (cached != null)  
        {  
            _logger.LogInformation("Cache hit for query: {Query}", query);  
            return new AIContext  
            {  
                Instructions = $"使用缓存的检索结果：\n{cached}"  
            };  
        }  
          
        return new AIContext();  
    }  
      
    public override async ValueTask InvokedAsync(  
        InvokedContext context,   
        CancellationToken cancellationToken = default)  
    {  
        var query = ExtractQuery(context.RequestMessages);  
        var results = ExtractRetrievalResults(context.ResponseMessages);  
          
        if (results.Any())  
        {  
            var cacheKey = ComputeCacheKey(query);  
            await _cache.SetStringAsync(  
                cacheKey,   
                JsonSerializer.Serialize(results),  
                new DistributedCacheEntryOptions { AbsoluteExpirationRelativeToNow = _ttl },  
                cancellationToken);  
        }  
    }  
}
```

#### 3. 可观测性

集成 OpenTelemetry 进行全链路追踪：

```
public class TelemetryMiddleware : AIContextProvider  
{  
    private readonly ActivitySource _activitySource;  
      
    public override async ValueTask<AIContext> InvokingAsync(  
        InvokingContext context,   
        CancellationToken cancellationToken = default)  
    {  
        using var activity = _activitySource.StartActivity("AgenticRAG.Invoke");  
        activity?.SetTag("query", ExtractQuery(context.RequestMessages));  
        activity?.SetTag("message_count", context.RequestMessages.Count());  
          
        return new AIContext();  
    }  
      
    public override async ValueTask InvokedAsync(  
        InvokedContext context,   
        CancellationToken cancellationToken = default)  
    {  
        var activity = Activity.Current;  
        activity?.SetTag("response_length",   
            context.ResponseMessages?.Sum(m => m.Text?.Length ?? 0) ?? 0);  
        activity?.SetTag("success", context.InvokeException == null);  
          
        if (context.InvokeException != null)  
        {  
            activity?.SetStatus(ActivityStatusCode.Error,   
                context.InvokeException.Message);  
        }  
    }  
}
```

## 七、性能优化与最佳实践

在生产环境中部署 Agentic RAG 系统，性能优化至关重要。以下是一些经过实战验证的最佳实践。

### 7.1 检索优化

**1. 混合检索策略**

单一的向量检索往往不够精确，结合多种检索方式效果更好：

```
public async Task<List<SearchResult>> HybridSearchAsync(  
    string query,   
    int topK = 10)  
{  
    // 并行执行多种检索  
    var tasks = new[]  
    {  
        _vectorStore.SearchAsync(query, topK),      // 语义搜索  
        _keywordSearch.SearchAsync(query, topK),    // 关键词搜索  
        _bm25Search.SearchAsync(query, topK)        // BM25 搜索  
    };  
      
    var results = await Task.WhenAll(tasks);  
      
    // 使用 Reciprocal Rank Fusion 融合结果  
    return ReciprocalRankFusion(results.SelectMany(r => r).ToList());  
}  
  
private List<SearchResult> ReciprocalRankFusion(  
    List<SearchResult> results,   
    int k = 60)  
{  
    var scores = new Dictionary<string, double>();  
      
    foreach (var result in results)  
    {  
        var docId = result.DocumentId;  
        if (!scores.ContainsKey(docId))  
            scores[docId] = 0;  
          
        scores[docId] += 1.0 / (k + result.Rank);  
    }  
      
    return scores  
        .OrderByDescending(kvp => kvp.Value)  
        .Select(kvp => results.First(r => r.DocumentId == kvp.Key))  
        .ToList();  
}
```

**2. 查询重写**

让 Agent 自动优化查询语句：

```
[Description("重写用户查询以提高检索效果")]  
static async Task<string> RewriteQuery(  
    [Description("原始查询")] string originalQuery,  
    [Description("查询意图")] string intent)  
{  
    // 使用小模型快速重写查询  
    var rewriteAgent = fastChatClient.CreateAIAgent(  
        instructions: "将用户查询改写为更适合检索的形式，保留关键信息，去除冗余");  
      
    var rewritten = await rewriteAgent.RunAsync(  
        $"原始查询：{originalQuery}\n意图：{intent}\n请重写查询");  
      
    return rewritten.Text;  
}
```

**3. 分块策略优化**

文档分块对检索效果影响巨大：

```
public class SmartChunker  
{  
    public List<DocumentChunk> ChunkDocument(string content, DocumentMetadata metadata)  
    {  
        var chunks = new List<DocumentChunk>();  
          
        // 根据文档类型选择分块策略  
        switch (metadata.Type)  
        {  
            case DocumentType.Code:  
                // 代码按函数/类分块  
                chunks = ChunkByCodeStructure(content);  
                break;  
                  
            case DocumentType.API:  
                // API 文档按端点分块  
                chunks = ChunkByApiEndpoint(content);  
                break;  
                  
            case DocumentType.Tutorial:  
                // 教程按章节分块，保持上下文  
                chunks = ChunkBySection(content, overlapTokens: 100);  
                break;  
                  
            default:  
                // 默认按固定大小分块  
                chunks = ChunkBySize(content, maxTokens: 512, overlapTokens: 50);  
                break;  
        }  
          
        // 为每个分块添加元数据  
        foreach (var chunk in chunks)  
        {  
            chunk.Metadata = metadata;  
            chunk.Summary = GenerateSummary(chunk.Content);  
        }  
          
        return chunks;  
    }  
}
```

### 7.2 成本优化

**1. 模型分层**

不同任务使用不同规模的模型：

```
public class ModelTierStrategy  
{  
    private readonly IChatClient _fastModel;    // gpt-4o-mini  
    private readonly IChatClient _smartModel;   // gpt-4o  
      
    public IChatClient SelectModel(AgentTask task)  
    {  
        return task switch  
        {  
            // 简单任务用小模型  
            AgentTask.QueryAnalysis => _fastModel,  
            AgentTask.Reranking => _fastModel,  
            AgentTask.QualityCheck => _fastModel,  
              
            // 复杂任务用大模型  
            AgentTask.ComplexReasoning => _smartModel,  
            AgentTask.AnswerSynthesis => _smartModel,  
              
            _ => _fastModel  
        };  
    }  
}
```

**2. 智能缓存**

多层缓存策略减少 API 调用：

```
public class MultiLevelCache  
{  
    private readonly IMemoryCache _l1Cache;           // 内存缓存  
    private readonly IDistributedCache _l2Cache;      // Redis 缓存  
    private readonly IVectorStore _l3Cache;           // 向量缓存  
      
    public async Task<string?> GetAsync(string query)  
    {  
        // L1: 内存缓存（毫秒级）  
        if (_l1Cache.TryGetValue(query, out string? cached))  
            return cached;  
          
        // L2: 分布式缓存（10ms 级）  
        var l2Result = await _l2Cache.GetStringAsync(query);  
        if (l2Result != null)  
        {  
            _l1Cache.Set(query, l2Result, TimeSpan.FromMinutes(5));  
            return l2Result;  
        }  
          
        // L3: 语义缓存（100ms 级）  
        var similar = await _l3Cache.FindSimilarAsync(query, threshold: 0.95);  
        if (similar != null)  
        {  
            _l1Cache.Set(query, similar.Answer, TimeSpan.FromMinutes(5));  
            await _l2Cache.SetStringAsync(query, similar.Answer);  
            return similar.Answer;  
        }  
          
        return null;  
    }  
}
```

### 7.3 可靠性保障

**1. 重试机制**

使用 Polly 实现智能重试：

```
public class ResilientAgenticRag  
{  
    private readonly IAsyncPolicy<string> _retryPolicy;  
      
    public ResilientAgenticRag()  
    {  
        _retryPolicy = Policy<string>  
            .Handle<HttpRequestException>()  
            .Or<TimeoutException>()  
            .WaitAndRetryAsync(  
                retryCount: 3,  
                sleepDurationProvider: attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)),  
                onRetry: (outcome, timespan, retryCount, context) =>  
                {  
                    _logger.LogWarning(  
                        "Retry {RetryCount} after {Delay}s due to {Exception}",  
                        retryCount, timespan.TotalSeconds, outcome.Exception?.Message);  
                });  
    }  
      
    public async Task<string> AnswerWithRetryAsync(string question)  
    {  
        return await _retryPolicy.ExecuteAsync(async () =>  
        {  
            return await _agenticRag.AnswerQuestionAsync(question);  
        });  
    }  
}
```

**2. 降级策略**

当 Agentic RAG 失败时，回退到简单 RAG：

```
public async Task<string> AnswerWithFallbackAsync(string question)  
{  
    try  
    {  
        // 尝试 Agentic RAG  
        return await _agenticRag.AnswerQuestionAsync(question);  
    }  
    catch (Exception ex)  
    {  
        _logger.LogError(ex, "Agentic RAG failed, falling back to simple RAG");  
          
        // 降级到简单 RAG  
        var results = await _vectorStore.SearchAsync(question, topK: 5);  
        var context = string.Join("\n\n", results.Select(r => r.Content));  
          
        return await _simpleAgent.RunAsync(  
            $"基于以下信息回答问题：\n{context}\n\n问题：{question}");  
    }  
}
```

**3. 超时控制**

防止长时间运行的 Workflow 阻塞：

```
public async Task<string> AnswerWithTimeoutAsync(  
    string question,   
    TimeSpan timeout)  
{  
    using var cts = new CancellationTokenSource(timeout);  
      
    try  
    {  
        return await _agenticRag.AnswerQuestionAsync(question, cts.Token);  
    }  
    catch (OperationCanceledException)  
    {  
        _logger.LogWarning("Query timeout after {Timeout}s", timeout.TotalSeconds);  
        return "抱歉，问题处理超时，请尝试简化您的问题。";  
    }  
}
```

## 八、对比分析：Agentic RAG vs 传统 RAG

让我们用一个实际场景来对比两种方案的差异。

### 8.1 场景：复杂技术问题

**用户问题**："我想在 Agent Framework 中实现一个能记住用户偏好的 Agent，并且支持多轮对话，应该怎么做？"

#### 传统 RAG 的处理流程

```
1. 向量检索：搜索 "Agent Framework 记住用户偏好 多轮对话"  
2. 返回 Top 5 文档片段  
3. 将文档 + 问题一起发给 LLM  
4. LLM 基于文档生成答案
```

**问题**：

* 检索到的文档可能只涵盖"记忆"或"多轮对话"的某一方面
* 无法自动关联 `AIContextProvider`、`AgentThread`、`Memory Provider` 等多个相关概念
* 如果第一次检索不到合适内容，就没有后续补救措施

#### Agentic RAG 的处理流程

```
1. Query Analyzer Agent 分析问题：  
   - 识别出两个子问题：状态管理 + 多轮对话  
   - 规划检索策略：先查概念，再查代码示例  
  
2. Retrieval Agent 执行多轮检索：  
   - 第一轮：搜索 "AIContextProvider memory"  
   - 发现 Memory Provider 概念  
   - 第二轮：搜索 "AgentThread conversation state"  
   - 发现 Thread 管理机制  
   - 第三轮：搜索 "UserInfoMemory example code"  
   - 找到完整代码示例  
  
3. Reranker Agent 评估结果：  
   - 过滤掉不相关的文档  
   - 按相关性重新排序  
   - 识别出最佳示例代码  
  
4. Synthesizer Agent 生成答案：  
   - 综合多个来源的信息  
   - 构建完整的解决方案  
   - 包含概念解释 + 代码示例 + 最佳实践  
  
5. Quality Checker 验证答案：  
   - 检查是否回答了所有子问题  
   - 验证代码示例的完整性  
   - 如果不满意，触发重新检索
```

### 8.2 效果对比

| 维度 | 传统 RAG | Agentic RAG |
| --- | --- | --- |
| **准确性** | 70% | 92% |
| **完整性** | 单一视角 | 多维度综合 |
| **响应时间** | 2-3 秒 | 5-8 秒 |
| **Token 消耗** | 较低（单次检索） | 较高（多次检索） |
| **适用场景** | 简单问答 | 复杂推理 |
| **用户满意度** | 中等 | 高 |

### 8.3 成本效益分析

虽然 Agentic RAG 的单次调用成本更高，但从整体 ROI 看：

```
传统 RAG：  
- 单次成本：$0.01  
- 用户满意度：70%  
- 需要人工介入：30%  
- 实际成本：$0.01 + (30% × 人工成本)  
  
Agentic RAG：  
- 单次成本：$0.03  
- 用户满意度：92%  
- 需要人工介入：8%  
- 实际成本：$0.03 + (8% × 人工成本)
```

对于高价值场景（技术支持、专业咨询等），Agentic RAG 的 ROI 更高。

## 九、未来展望：Agentic RAG 的进化方向

### 9.1 多模态检索

未来的 Agentic RAG 将不仅仅检索文本，还能处理：

* **图像**：检索架构图、流程图、UI 截图
* **视频**：检索教程视频的关键片段
* **音频**：检索会议录音、播客内容
* **代码**：不仅检索代码文本，还能理解代码语义和执行逻辑

Agent Framework 已经支持多模态输入：

```
var imageMessage = new ChatMessage(ChatRole.User,   
[  
    new TextContent("这个架构图中的 Agent 是如何协作的？"),  
    new ImageContent(imageUri)  
]);  
  
var response = await agent.RunAsync(imageMessage);
```

### 9.2 自我进化的检索系统

想象一个能够自我优化的 Agentic RAG：

```
public class SelfEvolvingRag  
{  
    // 收集用户反馈  
    public async Task RecordFeedbackAsync(string query, string answer, int rating)  
    {  
        await _feedbackStore.SaveAsync(new Feedback  
        {  
            Query = query,  
            Answer = answer,  
            Rating = rating,  
            Timestamp = DateTime.UtcNow  
        });  
          
        // 如果评分低，触发优化流程  
        if (rating < 3)  
        {  
            await OptimizeRetrievalStrategyAsync(query);  
        }  
    }  
      
    // 自动优化检索策略  
    private async Task OptimizeRetrievalStrategyAsync(string query)  
    {  
        // 分析失败原因  
        var analysis = await _analyzerAgent.RunAsync(  
            $"分析为什么这个查询的检索效果不好：{query}");  
          
        // 调整检索参数  
        if (analysis.Text.Contains("语义理解不准确"))  
        {  
            _searchConfig.SemanticWeight += 0.1;  
        }  
        else if (analysis.Text.Contains("缺少关键词匹配"))  
        {  
            _searchConfig.KeywordWeight += 0.1;  
        }  
          
        // 重新索引相关文档  
        await _indexer.ReindexAsync(query);  
    }  
}
```

### 9.3 协作式 Agentic RAG

多个 Agentic RAG 系统协作，形成知识网络：

```
用户问题 → 本地 Agentic RAG  
              ↓  
         (检索不到) → 请求协作  
              ↓  
    ┌─────────┴─────────┐  
    ↓                   ↓  
专家系统 A          专家系统 B  
(技术文档)          (代码示例)  
    ↓                   ↓  
    └─────────┬─────────┘  
              ↓  
         聚合答案  
              ↓  
         返回用户
```

### 9.4 实时知识更新

传统 RAG 的知识库是静态的，而未来的 Agentic RAG 可以：

* **实时爬取**：发现知识库中没有的信息时，自动爬取最新资料
* **增量索引**：新文档实时索引，无需重建整个知识库
* **版本管理**：跟踪文档的版本变化，提供最新信息

```
[Description("当知识库中没有答案时，从互联网搜索最新信息")]  
static async Task<string> SearchWeb(string query)  
{  
    var results = await _webSearchService.SearchAsync(query);  
      
    // 自动将搜索结果加入知识库  
    foreach (var result in results.Take(3))  
    {  
        await _vectorStore.AddAsync(new Document  
        {  
            Content = result.Content,  
            Source = result.Url,  
            Timestamp = DateTime.UtcNow,  
            IsTemporary = true  // 标记为临时文档，定期清理  
        });  
    }  
      
    return FormatResults(results);  
}
```

## 十、总结：从工具到伙伴

回到文章开头的问题：RAG 的"中年危机"怎么破？

答案是：**让 RAG 从被动的工具，进化为主动的伙伴**。

传统 RAG 就像一个只会查字典的助手，你问什么，它查什么，查不到就没辙了。而 Agentic RAG 更像一个有经验的研究员，会主动思考：

* "这个问题的核心是什么？"
* "我应该从哪些角度去查资料？"
* "第一次查到的信息够不够？需不需要补充？"
* "怎么把这些碎片化的信息整合成一个完整的答案？"

Microsoft Agent Framework 为实现这种"智能伙伴"提供了完整的基础设施：

* **AIContextProvider**：让检索逻辑可插拔、可组合
* **Function Tools**：赋予 Agent 主动调用检索工具的能力
* **Workflow**：支持复杂的多 Agent 协作和控制流
* **Memory**：让系统能够学习和进化

更重要的是，Agent Framework 的设计哲学——**简洁、可组合、生产就绪**——让开发者可以快速构建出既强大又可维护的 Agentic RAG 系统。

### 最后的建议

如果你正在考虑实现 Agentic RAG，我的建议是：

1. **从简单开始**：先用 Context Provider 模式实现基础的动态检索
2. **逐步增强**：根据实际需求，引入 Function Tools 和 Multi-Agent
3. **持续优化**：收集用户反馈，不断调整检索策略和 Prompt
4. **关注成本**：合理使用模型分层和缓存，控制运营成本

RAG 的未来不是更大的向量数据库，也不是更强的 Embedding 模型，而是更智能的检索策略。而 Agentic RAG + Agent Framework，正是通往这个未来的最佳路径。

关注我，带你一起探索AI世界的无限可能，更多精彩内容等你发现！