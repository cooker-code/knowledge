> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020303_RAG/020303_核心知识点/RAG生命周期与评估边界|RAG生命周期与评估边界]]
---
title: RAG 2.0 时代：从检索增强到主动推理的演进之路
author: 乾与千浔
date:
url: https://mp.weixin.qq.com/s?__biz=MzI5MDUzNTUzNA==&mid=2247485006&idx=1&sn=f395b9f638a4e411c3540532f680a8f8&chksm=ede0b4f918b18942c07dbed0387da4274079aaa82da074f6157917293f118e7e796a40ed8771&mpshare=1&scene=24&srcid=0417bsS7c8O3PETuZTheXfCJ&sharer_shareinfo=3dc85c555245685f39454ff565453570&sharer_shareinfo_first=3dc85c555245685f39454ff565453570#rd
---

# RAG 2.0 时代：从检索增强到主动推理的演进之路

当你的 AI 助手在回答"如何优化支付系统的分布式事务"时,不仅需要检索相关文档,还要判断检索到的内容是否真正相关、是否需要多次检索、甚至要反思自己的回答是否有事实依据——这就是 RAG 正在发生的革命。

2025 年初,一篇来自华盛顿大学和 IBM 的最新综述论文揭示了这个趋势:传统 RAG 正在被 Agentic RAG 取代,后者通过嵌入自主 AI Agent 实现动态检索策略管理、迭代式上下文理解优化,以及多步推理能力。本文将深入技术实现层面,拆解 RAG 2.0 的核心架构、关键技术,以及生产环境落地方案。

---

## 传统 RAG 的三大瓶颈

在深入 RAG 2.0 之前,我们先明确传统 RAG 面临的核心问题。

### 瓶颈一:盲目检索,无法自适应

传统 RAG 采用固定的检索策略——无论查询复杂度如何,都按照预设的步骤执行检索。比如「只在开始时检索一次」或「每 k 个 token 检索一次」。这种机械式的检索会导致两个极端:

**过度检索**:对于「什么是 Python for 循环」这类简单问题,LLM 本身就能回答好,额外的检索反而引入噪音,降低响应速度。

**检索不足**:对于「对比 Saga、TCC、2PC 三种分布式事务方案在高并发场景下的性能表现」这类复杂查询,单次检索很难覆盖所有必要信息,需要多轮检索才能获得完整答案。

### 瓶颈二:缺乏质量评估机制

传统 RAG 检索到文档后,会直接将其拼接到 prompt 中,不会评估:

* 检索到的文档是否真正相关?
* 生成的答案是否得到文档支撑?
* 引用的来源是否准确可靠?

这导致一个常见问题:LLM 可能基于不相关的检索结果「一本正经地胡说八道」,产生所谓的「幻觉」。

### 瓶颈三:单一知识源的局限性

标准 RAG 通常只查询一个外部知识源(如向量数据库),无法整合来自不同来源的信息——SQL 数据库、API 接口、Web 搜索、内部文档系统。当查询需要跨数据源综合分析时,传统 RAG 就力不从心了。

**案例**:「根据我们 Q4 财报和行业研究报告,分析我司在支付领域的竞争力」。这需要同时访问内部财报数据库、外部行业报告,以及可能的实时市场数据。

---

## Agentic RAG 的核心设计哲学

Agentic RAG 的本质是将 AI Agent 的自主决策能力注入到 RAG 流程中,实现从「被动检索」到「主动推理」的跃迁。

### 设计原则一:需求驱动的检索

Agent 不再按固定步骤检索,而是基于当前上下文动态判断:

* **是否需要检索**:LLM 自身知识足够时,跳过检索
* **检索什么内容**:根据查询意图选择检索策略
* **检索多少次**:根据信息完整度决定是否继续检索

这通过 **Retrieval Tokens**(检索标记)实现。LLM 在生成过程中会预测特殊 token(如 `[Retrieve]`),触发检索动作。

### 设计原则二:多维度的自我评估

Agentic RAG 引入 **Reflection Tokens**(反思标记),让 LLM 对自己的输出进行多维度评估:

* **`IsRel`**(相关性):检索到的文档是否相关?
* **`IsSup`**(支撑性):生成的内容是否有文档支撑?
* **`IsUse`**(实用性):答案整体质量如何?

这些评估结果会反馈到生成过程,形成一个「检索-生成-评估-优化」的闭环。

### 设计原则三:多源协同的知识整合

Agentic RAG 通过多 Agent 协作,实现跨数据源的知识整合:

* **Document Agents**:每个文档有专属 Agent,负责回答该文档相关的问题
* **Meta-Agent**:协调多个 Document Agents,综合各方信息
* **Tool Agents**:调用外部 API、数据库查询、Web 搜索等工具

---

## Agentic RAG 的技术架构深度剖析

### 架构演进:从 Naive RAG 到 Agentic RAG

让我们用对比的方式理解架构演进:

**Naive RAG 架构**:

```
用户查询 → Embedding → 向量检索 → 拼接 Prompt → LLM 生成 → 返回结果
```

这是一个线性流程,中间没有决策节点,也没有反馈机制。

**Agentic RAG 架构**:

```
用户查询 → Agent 决策中枢 
           ├→ 判断是否需要检索
           ├→ 选择检索策略(向量/SQL/API/Web)
           ├→ 多轮检索与质量评估
           ├→ 动态调整检索路径
           └→ 综合生成与自我验证 → 返回结果
```

这是一个图(Graph)结构,Agent 作为决策中枢,可以在不同节点间灵活跳转。

### 单 Agent 架构:路由与决策

最简单的 Agentic RAG 是一个 **Router Agent**(路由 Agent),它的核心职责是「选择最合适的知识源」。

**实现细节**:

1. **多知识源注册**:

```
knowledge_sources = {
    "vector_db": VectorStoreRetriever(collection="tech_docs"),
    "sql_db": SQLRetriever(database="company_data"),
    "web_search": TavilySearchRetriever(),
    "api_tool": SlackRetriever()  # 查询 Slack 历史消息
}
```

2. **Agent 决策逻辑**:

```
def route_retrieval(query: str, agent_llm: LLM) -> str:
    """Agent 根据查询意图,选择最合适的知识源"""
    routing_prompt = f"""
    Given the query: "{query}"
    Available knowledge sources: {list(knowledge_sources.keys())}
    
    Which source should we retrieve from? Think step by step:
    1. What type of information does this query need?
    2. Which source is most likely to contain this information?
    
    Return the source name only.
    """
    return agent_llm.invoke(routing_prompt).strip()
```

3. **检索执行**:

```
selected_source = route_retrieval(query, agent_llm)
retrieved_docs = knowledge_sources[selected_source].retrieve(query)
```

**适用场景**:知识源有限(2-5 个),查询意图明确的应用,如企业内部知识库、FAQ 系统。

### 多 Agent 架构:专业化分工

当需要处理复杂多源查询时,单 Agent 会遇到能力瓶颈。多 Agent 架构通过「专业化分工」解决这个问题。

**典型设计**:

```
from langgraph.graph import StateGraph, END

# 定义 Agent 状态
class MultiAgentState(TypedDict):
    query: str
    document_results: Dict[str, str]  # 各文档 Agent 的结果
    final_answer: str
    retrieval_history: List[str]

# 文档专属 Agent
class DocumentAgent:
    def __init__(self, doc_id: str, doc_content: str):
        self.doc_id = doc_id
        self.doc_content = doc_content
    
    def answer(self, query: str) -> str:
        """基于特定文档回答问题"""
        prompt = f"""
        Document: {self.doc_content}
        Question: {query}
        
        Answer based ONLY on this document. If the document doesn't 
        contain relevant information, say "Not found in this document."
        """
        return llm.invoke(prompt)

# Meta-Agent:协调与综合
def meta_agent_node(state: MultiAgentState):
    """Meta-Agent 协调各文档 Agent,综合答案"""
    query = state["query"]
    
    # 激活所有 Document Agents
    document_agents = [
        DocumentAgent("doc1", load_document("payments_architecture")),
        DocumentAgent("doc2", load_document("distributed_transactions")),
        DocumentAgent("doc3", load_document("saga_pattern_guide"))
    ]
    
    # 并行查询各 Agent
    results = {}
    for agent in document_agents:
        results[agent.doc_id] = agent.answer(query)
    
    # 综合生成最终答案
    synthesis_prompt = f"""
    Query: {query}
    
    Information from different documents:
    {json.dumps(results, indent=2)}
    
    Synthesize a comprehensive answer by combining relevant 
    information from all documents. Cite sources for each claim.
    """
    
    final_answer = llm.invoke(synthesis_prompt)
    
    return {
        "document_results": results,
        "final_answer": final_answer
    }

# 构建 Multi-Agent Graph
workflow = StateGraph(MultiAgentState)
workflow.add_node("meta_agent", meta_agent_node)
workflow.set_entry_point("meta_agent")
workflow.add_edge("meta_agent", END)

graph = workflow.compile()
```

**关键优势**:

* **可扩展性**:添加新文档只需注册新 Agent,不影响现有系统
* **并行处理**:多个 Agent 可同时工作,提升效率
* **专业性**:每个 Agent 专注于特定领域,答案质量更高

**适用场景**:大型文档库、跨领域知识整合、需要引用溯源的场景。

### ReAct 模式:推理与行动的统一

ReAct(Reasoning + Acting)是 Agentic RAG 的核心设计模式,它将 Agent 的「思考」和「行动」交替进行。

**ReAct 循环**:

```
1. Thought(思考):Agent 分析当前情况,决定下一步行动
2. Action(行动):执行工具调用(检索、查询 API 等)
3. Observation(观察):获取行动结果
4. 重复 1-3,直到得出最终答案
```

**LangGraph 实现**:

```
from langgraph.prebuilt import create_react_agent
from langchain.tools import tool

# 定义工具
@tool
def retrieve_vector_db(query: str) -> str:
    """从向量数据库检索相关文档"""
    docs = vector_store.similarity_search(query, k=3)
    return"\n\n".join([doc.page_content for doc in docs])

@tool
def query_sql_database(sql: str) -> str:
    """执行 SQL 查询"""
    result = db_executor.execute(sql)
    return str(result)

# 创建 ReAct Agent
tools = [retrieve_vector_db, query_sql_database]
agent = create_react_agent(
    model="claude-sonnet-4-20250514",
    tools=tools,
    prompt="你是一个技术助手,擅长回答支付系统相关问题。"
)

# 运行 Agent
result = agent.invoke({
    "messages": [{
        "role": "user", 
        "content": "分析我们 Q4 支付交易量增长趋势"
    }]
})
```

**内部执行流程**(Agent 的「心理活动」):

```
Thought: 用户询问 Q4 支付交易量,这需要查询数据库
Action: query_sql_database("SELECT month, sum(amount) FROM transactions 
                           WHERE quarter=4 GROUP BY month")
Observation: [('Oct', 1500000), ('Nov', 1800000), ('Dec', 2100000)]

Thought: 数据显示交易量逐月增长,但需要上下文来分析增长原因
Action: retrieve_vector_db("Q4 支付业务分析报告")
Observation: "Q4 双十一、黑五促销活动带动交易量增长..."

Thought: 现在有足够信息生成答案
Answer: Q4 支付交易量呈现强劲增长趋势,从 10 月的 150 万增长到 
       12 月的 210 万,增幅达 40%。主要驱动因素包括...
```

**技术要点**:

* **工具调用**:LLM 通过 Function Calling 机制调用工具
* **状态管理**:LangGraph 的 `StateGraph` 维护对话历史和中间结果
* **决策控制**:通过 `tools_condition` 判断是否继续调用工具

---

## Self-RAG:让 LLM 学会自我反思

Self-RAG 是 Agentic RAG 的一个重要变体,它通过训练 LLM 生成 **Reflection Tokens**,实现自主的检索决策和质量评估。

### 核心机制:Reflection Tokens

Self-RAG 定义了一组特殊 token,用于不同维度的自我评估:

| Token 类型 | 说明 | 可能值 |
| --- | --- | --- |
| `[Retrieve]` | 是否需要检索 | `Yes` / `No` |
| `[IsRel]` | 检索内容是否相关 | `Relevant` / `Irrelevant` |
| `[IsSup]` | 生成内容是否有支撑 | `Fully Supported` / `Partially Supported` / `No Support` |
| `[IsUse]` | 答案整体是否有用 | `5` / `4` / `3` / `2` / `1` |

### 训练流程

**第一步:Critic Model 训练**

先训练一个独立的 Critic Model,用于标注训练数据:

```
# 伪代码示例
def train_critic_model(training_data):
    """训练 Critic Model,用于生成 Reflection Tokens"""
    for sample in training_data:
        query = sample.query
        retrieved_doc = sample.document
        generated_answer = sample.answer
        
        # 人工标注或启发式标注
        is_relevant = evaluate_relevance(query, retrieved_doc)
        is_supported = evaluate_support(generated_answer, retrieved_doc)
        is_useful = evaluate_usefulness(generated_answer, query)
        
        # 将标注添加到训练样本
        sample.add_labels({
            "IsRel": is_relevant,
            "IsSup": is_supported,
            "IsUse": is_useful
        })
    
    # 训练 Critic Model
    critic_model = train_model(training_data)
    return critic_model
```

**第二步:生成器训练**

使用 Critic Model 标注的数据,训练最终的 Self-RAG 模型:

```
def prepare_training_data(raw_data, critic_model):
    """使用 Critic Model 标注训练数据"""
    enhanced_data = []
    for sample in raw_data:
        # 使用 Critic Model 插入 Reflection Tokens
        annotated_text = f"""
        Query: {sample.query}
        [Retrieve] Yes  # Critic Model 判断需要检索
        Retrieved: {sample.document}
        [IsRel] Relevant  # Critic Model 评估相关性
        Generated: {sample.answer}
        [IsSup] Fully Supported  # Critic Model 评估支撑性
        [IsUse] 5  # Critic Model 评估实用性
        """
        enhanced_data.append(annotated_text)
    
    return enhanced_data

# 训练 Self-RAG 模型
training_data = prepare_training_data(raw_data, critic_model)
self_rag_model = train_language_model(training_data)
```

### 推理流程:Adaptive Retrieval

Self-RAG 在推理时,通过预测 Reflection Tokens 实现自适应检索:

```
def self_rag_inference(query: str, model: SelfRAGModel):
    """Self-RAG 推理流程"""
    output = query
    
    whilenot is_complete(output):
        # 模型预测下一个 token
        next_token = model.predict_next(output)
        
        if next_token == "[Retrieve]":
            # 模型决定是否检索
            retrieve_decision = model.predict_next(output + next_token)
            
            if retrieve_decision == "Yes":
                # 执行检索
                docs = retriever.retrieve(query)
                output += f"\n[Retrieve] Yes\nRetrieved: {docs}"
                
                # 评估相关性
                relevance = model.predict_next(output + "\n[IsRel]")
                output += f"\n[IsRel] {relevance}"
                
                if relevance == "Irrelevant":
                    # 重新检索
                    docs = retriever.retrieve(query, rerank=True)
                    output += f"\nRetrieved (reranked): {docs}"
            else:
                output += "\n[Retrieve] No"
        
        elif next_token.startswith("[Is"):
            # 评估 token,记录但不影响生成
            output += next_token
        
        else:
            # 正常生成
            output += next_token
    
    return extract_final_answer(output)
```

### 性能提升数据

根据 Self-RAG 论文的实验结果:

* **事实核查准确率**:从 71%(传统 RAG)提升到 81%
* **长文本生成的引用准确率**:Self-RAG 达到 80%,ChatGPT 仅 71%
* **开放域问答**:在 TriviaQA 等基准上,7B 参数的 Self-RAG 超越 13B Llama2-chat + RAG

---

## 向量数据库与 Embedding 模型选型

Agentic RAG 的性能很大程度上取决于底层的向量检索效率和 Embedding 质量。

### 2025 年主流 Embedding 模型对比

| 模型 | 维度 | 特点 | 适用场景 |
| --- | --- | --- | --- |
| **Voyage-3-large** | 1024 | 2025 最新,检索准确率领先 | 高精度要求的企业应用 |
| **Cohere Embed v3** | 1024 | 支持长文档,多语言 | 跨语言、长文档场景 |
| **OpenAI text-embedding-3-large** | 3072 | 成熟稳定,生态完善 | 通用场景,快速集成 |
| **BGE-M3** | 1024 | 开源,中英日三语优化 | 亚洲语言场景 |
| **E5-Mistral** | 4096 | 开源,融合 Mistral 编码器 | 预算敏感,需要自托管 |

**选型建议**:

1. **快速 MVP**:OpenAI text-embedding-3-small(成本 $0.02/百万 token)
2. **生产环境**:Voyage-3-large 或 Cohere Embed v3
3. **成本优化**:BGE-M3 或 E5-Mistral(开源自托管)
4. **多语言**:Cohere Embed v3(140+ 语言支持)

### 向量数据库选型

**核心考量维度**:

1. **性能**:QPS(每秒查询数)、P99 延迟
2. **规模**:支持的向量数量(百万 vs 十亿级)
3. **特性**:混合检索(向量 + 关键词)、元数据过滤、多租户
4. **成本**:托管服务 vs 自部署

**主流方案对比**:

| 数据库 | 类型 | 特点 | 适用场景 |
| --- | --- | --- | --- |
| **Pinecone** | 托管 | 零运维,自动扩展 | 初创团队,快速上线 |
| **Weaviate** | 开源/托管 | 模块化,支持多模态 | 需要灵活定制 |
| **Milvus** | 开源 | 高性能,云原生 | 大规模部署,技术团队强 |
| **Qdrant** | 开源/托管 | Rust 实现,高效过滤 | 性能敏感场景 |
| **Chroma** | 开源 | 轻量级,易集成 | 本地开发,小规模应用 |

**索引算法选择**:

```
# HNSW:生产环境默认选择,平衡召回率和速度
index_params = {
    "index_type": "HNSW",
    "M": 16,  # 每个节点的连接数
    "efConstruction": 200,  # 构建时的搜索深度
}

# IVF+PQ:十亿级向量,牺牲少量准确率换取速度
index_params = {
    "index_type": "IVF_PQ",
    "nlist": 1024,  # 聚类中心数
    "m": 8,  # PQ 子向量数
}

# 查询参数调优
search_params = {
    "ef": 100,  # HNSW 搜索深度,越大越准确但越慢
    "nprobe": 10,  # IVF 探测的聚类数
}
```

**性能优化技巧**:

1. **批量 Embedding**:将多个文档合并到一个 API 调用,减少开销 10 倍
2. **向量量化**:INT8 量化可减少 4 倍内存,召回率损失 <1%
3. **两阶段检索**:

* 第一阶段:低成本模型召回 Top-100
* 第二阶段:高精度 Reranker 精排 Top-10

```
# 两阶段检索实现
def two_stage_retrieval(query: str, k: int = 10):
    # 第一阶段:快速召回
    cheap_embeddings = cheap_model.embed(query)
    candidates = vector_db.search(cheap_embeddings, top_k=100)
    
    # 第二阶段:精排
    reranked = reranker_model.rerank(
        query=query,
        documents=[c.text for c in candidates]
    )
    
    return reranked[:k]
```

---

## 生产环境部署:从原型到规模化

### 架构设计:模块化与可观测

**推荐架构**:

```
┌─────────────────┐
│  API Gateway    │  ← 统一入口,限流、鉴权
└────────┬────────┘
         │
    ┌────┴────┐
    │ Router  │  ← 路由层,选择处理策略
    └────┬────┘
         │
    ┌────┴──────────────────┐
    │                       │
┌───┴────┐          ┌──────┴─────┐
│Retrieval│         │  Agent     │
│ Service│          │ Orchestrator│
└───┬────┘          └──────┬─────┘
    │                      │
┌───┴────┐          ┌──────┴─────┐
│Vector  │          │   LLM      │
│  DB    │          │  Service   │
└────────┘          └────────────┘
         │                  │
    ┌────┴──────────────────┴────┐
    │  Observability Platform    │
    │  (日志、指标、追踪)         │
    └────────────────────────────┘
```

**关键设计原则**:

1. **服务拆分**:检索、生成、评估各自独立,便于扩展和调试
2. **异步处理**:长时间任务(如多轮检索)使用消息队列
3. **缓存策略**:

* Embedding 缓存:相同文本不重复计算
* 检索结果缓存:高频查询直接返回
* 语义缓存:语义相似的查询共享结果

### 延迟优化:从秒级到毫秒级

**关键指标**:

* **P50 延迟**:中位数响应时间,目标 < 500ms
* **P99 延迟**:99% 用户的体验,目标 < 2000ms
* **长尾优化**:处理慢查询,避免超时

**优化策略**:

```
# 1. 并行检索
asyncdef parallel_retrieval(query: str):
    tasks = [
        vector_db.search_async(query),
        sql_db.query_async(query),
        web_search.search_async(query)
    ]
    results = await asyncio.gather(*tasks)
    return merge_results(results)

# 2. 语义缓存
from functools import lru_cache

@lru_cache(maxsize=1000)
def semantic_cache(query_embedding: tuple) -> str:
    """基于 Embedding 的缓存"""
    # 查找最相似的已缓存查询
    similar_query = find_most_similar(query_embedding, cache_keys)
    if similarity(query_embedding, similar_query) > 0.95:
        return cache[similar_query]
    returnNone

# 3. 预热常见查询
def warm_cache():
    """系统启动时预加载高频查询结果"""
    common_queries = [
        "什么是 Saga 模式",
        "如何优化数据库查询",
        # ...
    ]
    for query in common_queries:
        _ = rag_system.invoke(query)  # 触发缓存

# 4. 超时与降级
asyncdef rag_with_timeout(query: str, timeout: float = 2.0):
    try:
        result = await asyncio.wait_for(
            rag_system.invoke_async(query),
            timeout=timeout
        )
        return result
    except asyncio.TimeoutError:
        # 降级策略:返回无检索的 LLM 回答
        return llm.invoke(query)
```

**Perplexity AI 的优化实践**:

* 语义缓存命中率 > 30%,节省大量计算
* 批量 Embedding 处理,吞吐量提升 10 倍
* GPU 加速推理,延迟降低到 < 200ms

### 成本优化:控制 Token 与计算开销

**成本构成**:

1. **Embedding 成本**:0.13 每百万 token
2. **LLM 推理成本**:15 每百万 token
3. **向量数据库**:存储 + 计算费用

**优化方法**:

```
# 1. 智能 Chunking:避免过小的块
def smart_chunking(document: str, target_size: int = 512):
    """基于段落和语义边界切分"""
    chunks = []
    current_chunk = ""
    
    for paragraph in document.split("\n\n"):
        if len(current_chunk) + len(paragraph) < target_size:
            current_chunk += paragraph + "\n\n"
        else:
            chunks.append(current_chunk.strip())
            current_chunk = paragraph + "\n\n"
    
    if current_chunk:
        chunks.append(current_chunk.strip())
    
    return chunks

# 2. 增量 Embedding:只计算新增/修改的文档
class IncrementalEmbedder:
    def __init__(self):
        self.embeddings_cache = {}  # {doc_id: (embedding, version)}
    
    def embed_documents(self, documents: List[Dict]):
        """只 embed 未缓存或已修改的文档"""
        new_embeddings = []
        
        for doc in documents:
            doc_id = doc["id"]
            doc_version = doc["version"]
            
            cached = self.embeddings_cache.get(doc_id)
            if cached and cached[1] == doc_version:
                # 使用缓存
                new_embeddings.append(cached[0])
            else:
                # 计算新 Embedding
                emb = embedding_model.embed(doc["content"])
                self.embeddings_cache[doc_id] = (emb, doc_version)
                new_embeddings.append(emb)
        
        return new_embeddings

# 3. 动态 Top-K:根据查询复杂度调整检索数量
def adaptive_top_k(query: str) -> int:
    """简单查询检索少,复杂查询检索多"""
    query_length = len(query.split())
    
    if query_length < 5:
        return3# 简单问题
    elif query_length < 15:
        return5# 中等复杂度
    else:
        return10# 复杂问题

# 4. 模型降级:高峰时使用更便宜的模型
def select_model_by_load(current_qps: int):
    """根据负载选择模型"""
    if current_qps < 100:
        return"claude-sonnet-4-20250514"# 高质量
    elif current_qps < 500:
        return"claude-haiku-4-5-20251001"# 平衡性价比
    else:
        return"gpt-3.5-turbo"# 成本优先
```

**成本监控**:

```
# 实时成本追踪
class CostTracker:
    def __init__(self):
        self.costs = {
            "embedding": 0.0,
            "llm": 0.0,
            "vector_db": 0.0
        }
    
    def track_embedding(self, num_tokens: int, model: str):
        cost_per_million = {
            "voyage-3-large": 0.13,
            "text-embedding-3-large": 0.13,
            "bge-m3": 0.0# 自托管
        }
        self.costs["embedding"] += (num_tokens / 1_000_000) * cost_per_million[model]
    
    def track_llm(self, input_tokens: int, output_tokens: int, model: str):
        pricing = {
            "claude-sonnet-4-20250514": (3.0, 15.0),  # (输入, 输出)
            "claude-haiku-4-5-20251001": (0.8, 4.0),
            "gpt-3.5-turbo": (0.5, 1.5)
        }
        input_cost, output_cost = pricing[model]
        self.costs["llm"] += (input_tokens / 1_000_000) * input_cost
        self.costs["llm"] += (output_tokens / 1_000_000) * output_cost
    
    def get_total_cost(self) -> float:
        return sum(self.costs.values())
```

### 可观测性:三大支柱

**日志记录**:

```
import structlog

logger = structlog.get_logger()

def rag_pipeline_with_logging(query: str):
    request_id = generate_request_id()
    
    logger.info("rag_request_start", 
                request_id=request_id, 
                query=query)
    
    # 检索阶段
    start_time = time.time()
    docs = retriever.retrieve(query)
    retrieval_time = time.time() - start_time
    
    logger.info("retrieval_complete",
                request_id=request_id,
                num_docs=len(docs),
                retrieval_time_ms=retrieval_time * 1000)
    
    # 生成阶段
    start_time = time.time()
    answer = llm.generate(query, context=docs)
    generation_time = time.time() - start_time
    
    logger.info("generation_complete",
                request_id=request_id,
                generation_time_ms=generation_time * 1000,
                answer_length=len(answer))
    
    return answer
```

**指标监控**:

```
from prometheus_client import Counter, Histogram

# 定义指标
rag_requests = Counter('rag_requests_total', 'Total RAG requests')
rag_latency = Histogram('rag_latency_seconds', 'RAG request latency')
retrieval_docs = Histogram('retrieval_docs_count', 'Number of retrieved docs')
rag_errors = Counter('rag_errors_total', 'RAG errors', ['error_type'])

# 在代码中使用
@rag_latency.time()
def process_rag_request(query: str):
    rag_requests.inc()
    try:
        docs = retriever.retrieve(query)
        retrieval_docs.observe(len(docs))
        return generate_answer(query, docs)
    except RetrievalError as e:
        rag_errors.labels(error_type='retrieval').inc()
        raise
    except GenerationError as e:
        rag_errors.labels(error_type='generation').inc()
        raise
```

**分布式追踪**:

```
from opentelemetry import trace
from opentelemetry.trace import Status, StatusCode

tracer = trace.get_tracer(__name__)

def rag_with_tracing(query: str):
    with tracer.start_as_current_span("rag_pipeline") as span:
        span.set_attribute("query", query)
        
        # 检索追踪
        with tracer.start_as_current_span("retrieval"):
            docs = retriever.retrieve(query)
            span.set_attribute("num_docs", len(docs))
        
        # 生成追踪
        with tracer.start_as_current_span("generation"):
            answer = llm.generate(query, context=docs)
            span.set_attribute("answer_length", len(answer))
        
        span.set_status(Status(StatusCode.OK))
        return answer
```

**关键追踪维度**:

* 端到端延迟分解(检索、生成、后处理各占比)
* 检索质量(召回率、准确率、相关性评分)
* 生成质量(幻觉检测、引用准确率)
* 资源使用(GPU 利用率、内存占用)

---

## 主流框架对比与选型

### LangChain:生态最完善

**优势**:

* 200+ 集成(LLM、向量数据库、工具)
* 社区活跃,文档齐全
* LCEL(LangChain Expression Language)表达力强

**劣势**:

* 抽象层次多,性能开销大
* 复杂场景下调试困难
* 版本迭代快,API 不稳定

**适用场景**:快速原型开发,需要大量第三方集成

**示例**:

```
from langchain.chains import RetrievalQA
from langchain.vectorstores import Pinecone
from langchain.embeddings import VoyageEmbeddings

# 快速构建 RAG
vectorstore = Pinecone.from_existing_index(
    index_name="tech-docs",
    embedding=VoyageEmbeddings()
)

qa_chain = RetrievalQA.from_chain_type(
    llm=ChatAnthropic(model="claude-sonnet-4-20250514"),
    retriever=vectorstore.as_retriever(search_kwargs={"k": 5}),
    return_source_documents=True
)

result = qa_chain.invoke("什么是 Saga 模式?")
```

### LangGraph:复杂流程编排

**优势**:

* 状态机设计,清晰建模复杂流程
* 内置循环、条件分支、并行执行
* 人机交互(Human-in-the-Loop)支持

**劣势**:

* 学习曲线陡峭
* 文档相对不足
* 生态尚不成熟

**适用场景**:多步推理、Agent 协作、需要复杂控制流的场景

**示例**(Multi-Agent RAG):

```
from langgraph.graph import StateGraph, END

class AgenticRAGState(TypedDict):
    query: str
    documents: List[str]
    answer: str
    reflections: List[str]

def retrieve_node(state: AgenticRAGState):
    docs = retriever.retrieve(state["query"])
    return {"documents": [d.page_content for d in docs]}

def generate_node(state: AgenticRAGState):
    context = "\n\n".join(state["documents"])
    answer = llm.invoke(f"Context: {context}\n\nQuery: {state['query']}")
    return {"answer": answer}

def reflect_node(state: AgenticRAGState):
    """Self-RAG 反思节点"""
    relevance = evaluate_relevance(state["query"], state["documents"])
    support = evaluate_support(state["answer"], state["documents"])
    
    if relevance < 0.7or support < 0.8:
        return {"reflections": ["需要重新检索更相关的文档"]}
    return {"reflections": []}

def should_continue(state: AgenticRAGState):
    return"retrieve"if state["reflections"] else END

# 构建图
workflow = StateGraph(AgenticRAGState)
workflow.add_node("retrieve", retrieve_node)
workflow.add_node("generate", generate_node)
workflow.add_node("reflect", reflect_node)

workflow.set_entry_point("retrieve")
workflow.add_edge("retrieve", "generate")
workflow.add_edge("generate", "reflect")
workflow.add_conditional_edges("reflect", should_continue)

graph = workflow.compile()
```

### Haystack:企业级 NLP 管道

**优势**:

* 模块化设计,易于定制
* 企业功能完善(评估、部署、监控)
* 支持多种检索方式(BM25、向量、混合)

**劣势**:

* Agent 能力相对弱
* 社区规模小于 LangChain
* 学习资源较少

**适用场景**:企业级搜索、文档问答、需要稳定性和可维护性的场景

**示例**:

```
from haystack import Pipeline
from haystack.components.retrievers import InMemoryBM25Retriever
from haystack.components.generators import OpenAIGenerator

# 混合检索管道
pipeline = Pipeline()
pipeline.add_component("bm25_retriever", InMemoryBM25Retriever(document_store))
pipeline.add_component("vector_retriever", InMemoryEmbeddingRetriever(document_store))
pipeline.add_component("generator", OpenAIGenerator(model="gpt-4"))

pipeline.connect("bm25_retriever", "generator.documents")
pipeline.connect("vector_retriever", "generator.documents")

result = pipeline.run({
    "bm25_retriever": {"query": "Saga pattern"},
    "vector_retriever": {"query": "Saga pattern"},
    "generator": {"prompt": "Answer based on: {documents}"}
})
```

### LlamaIndex:数据为中心

**优势**:

* 丰富的数据连接器(150+ 数据源)
* 专注于索引优化和检索质量
* 高级 RAG 技术内置(Multi-Document、Recursive Retrieval)

**劣势**:

* Agent 能力不如 LangGraph
* 文档结构复杂
* 版本迭代激进

**适用场景**:大规模文档索引、复杂数据结构、需要高级 RAG 技术

**示例**:

```
from llama_index import VectorStoreIndex, SimpleDirectoryReader
from llama_index.retrievers import RecursiveRetriever
from llama_index.query_engine import RetrieverQueryEngine

# 递归检索(先检索摘要,再检索细节)
documents = SimpleDirectoryReader("./docs").load_data()
index = VectorStoreIndex.from_documents(documents)

retriever = RecursiveRetriever(
    "vector",
    retriever_dict={"vector": index.as_retriever()},
    node_dict={"vector": index.docstore}
)

query_engine = RetrieverQueryEngine.from_args(retriever)
response = query_engine.query("Explain distributed transactions")
```

### 选型决策树

```
是否需要复杂控制流(循环、分支、多Agent)?
├─ 是 → LangGraph
└─ 否
    ├─ 是否需要大量第三方集成?
    │   ├─ 是 → LangChain
    │   └─ 否
    │       ├─ 数据源复杂,需要高级索引?
    │       │   ├─ 是 → LlamaIndex
    │       │   └─ 否 → Haystack(企业级) 或 自研(极致性能)
```

---

## 真实案例:某金融科技公司的 RAG 演进之路

### 背景:传统 RAG 的痛点

某支付公司技术团队构建了一个内部技术知识库问答系统,使用传统 RAG 架构:

**痛点 1:检索召回率低**

* 用户查询「如何处理支付超时」,系统检索到「订单超时处理」文档,实际不相关
* 召回率仅 60%,大量用户抱怨答非所问

**痛点 2:答案缺乏上下文**

* 问「Saga 模式的优缺点」,只返回优点,没有提到缺点(因为分散在不同文档)
* 需要多轮对话才能获得完整信息

**痛点 3:无法处理多步推理**

* 问「对比 Saga、TCC、2PC 在高并发场景下的性能」,系统只检索单一模式的文档
* 无法综合对比多个方案

### 改进方案:引入 Agentic RAG

**第一阶段:Router Agent**(1 个月)

引入路由 Agent,根据查询类型选择不同知识源:

```
knowledge_sources = {
    "architecture": VectorRetriever(collection="architecture_docs"),
    "code": CodeSearchRetriever(repo="payment-system"),
    "incidents": SQLRetriever(table="incidents"),
    "external": WebSearchRetriever()
}

def route_query(query: str) -> str:
    """基于查询意图路由"""
    routing_llm = ChatAnthropic(model="claude-haiku-4-5-20251001")
    
    prompt = f"""
    Query: {query}
    Categories: {list(knowledge_sources.keys())}
    
    Which category best matches this query? Return category name only.
    """
    
    return routing_llm.invoke(prompt).strip()
```

**效果**:

* 召回率从 60% 提升到 78%
* P99 延迟从 3.5s 降低到 2.1s(减少不必要的检索)

**第二阶段:Multi-Agent 协作**(2 个月)

针对复杂查询,引入多 Agent 架构:

```
# 为每个主题域创建专属 Agent
agents = {
    "saga": ExpertAgent(docs=load_docs("saga_pattern")),
    "tcc": ExpertAgent(docs=load_docs("tcc_pattern")),
    "2pc": ExpertAgent(docs=load_docs("2pc_pattern"))
}

# Meta-Agent 协调
def multi_agent_answer(query: str):
    # 识别需要的 Agents
    required_agents = identify_agents(query)  # ["saga", "tcc", "2pc"]
    
    # 并行查询
    results = {}
    for agent_name in required_agents:
        results[agent_name] = agents[agent_name].answer(query)
    
    # 综合答案
    synthesis_prompt = f"""
    Query: {query}
    
    Expert opinions:
    {json.dumps(results, indent=2)}
    
    Provide a comprehensive comparison synthesizing all expert views.
    """
    
    return llm.invoke(synthesis_prompt)
```

**效果**:

* 多方案对比类查询的满意度从 65% 提升到 89%
* 答案完整度显著提升(用户追问率下降 40%)

**第三阶段:Self-RAG 反思**(3 个月)

引入 Self-RAG 机制,让系统自我评估和优化:

```
def self_rag_pipeline(query: str):
    # 初次检索
    docs = retriever.retrieve(query)
    
    # 评估相关性
    relevance_scores = [
        evaluate_relevance(query, doc) for doc in docs
    ]
    
    if max(relevance_scores) < 0.7:
        # 重新检索,调整查询
        refined_query = refine_query(query, docs)
        docs = retriever.retrieve(refined_query)
    
    # 生成答案
    answer = llm.generate(query, context=docs)
    
    # 评估支撑性
    support_score = evaluate_support(answer, docs)
    
    if support_score < 0.8:
        # 答案不够可靠,添加警告或重新生成
        answer = f"⚠️ 答案置信度较低\n\n{answer}"
    
    return answer, docs
```

**效果**:

* 事实准确率从 82% 提升到 93%
* 用户信任度显著提升(引用来源更准确)

### 关键收获

1. **分阶段演进**:不要一步到位,先解决最痛的问题
2. **数据驱动**:基于真实用户查询日志优化,而非主观猜测
3. **成本平衡**:Haiku 用于路由,Sonnet 用于最终生成,节省 60% 成本
4. **可观测性**:每个阶段都加强监控,快速发现问题

---

## 未来趋势:RAG 的下一个十年

### 趋势一:多模态 RAG

未来 RAG 不仅检索文本,还会检索图片、视频、音频:

```
# 多模态检索(概念性示例)
query = "展示支付系统的架构图"

# 同时检索文本和图片
text_docs = text_retriever.retrieve(query)
image_docs = image_retriever.retrieve(query)  # 基于 CLIP 的图片检索

# 多模态 LLM 综合生成
answer = multimodal_llm.generate(
    query=query,
    text_context=text_docs,
    image_context=image_docs
)
```

### 趋势二:长上下文 + RAG 的融合

随着 LLM 上下文窗口扩大(Gemini 2.0 Flash 已达 100 万 token),RAG 的角色将转变:

* **短上下文场景**:继续使用 RAG 检索
* **长上下文场景**:将整个知识库放入上下文,RAG 用于「智能导航」而非检索

```
# 长上下文 + RAG
def long_context_rag(query: str, knowledge_base: str):
    """知识库放入上下文,RAG 用于定位关键段落"""
    
    # 粗筛:RAG 找到相关文档
    relevant_docs = retriever.retrieve(query, top_k=50)
    
    # 精读:全部文档放入 LLM 长上下文
    full_context = "\n\n".join([doc.content for doc in relevant_docs])
    
    prompt = f"""
    Knowledge Base (50 documents):
    {full_context}
    
    Question: {query}
    
    Answer based on the knowledge base, citing specific document sections.
    """
    
    return long_context_llm.invoke(prompt)
```

### 趋势三:端到端的神经检索

传统 RAG 依赖独立的 Embedding 模型,未来会出现端到端训练的检索-生成模型:

* **Retrieval-Augmented Dual Instruction Tuning**(RA-DIT):同时优化检索器和生成器
* **Differentiable Search Index**(DSI):将整个知识库编码到模型参数中

### 趋势四:个性化与隐私保护

企业 RAG 将更关注:

* **用户个性化**:基于用户历史和偏好优化检索
* **隐私保护**:联邦学习、差分隐私、本地化部署
* **细粒度权限控制**:不同用户看到不同检索结果

---

## 总结:从检索增强到主动推理

RAG 技术正经历从「被动检索」到「主动推理」的范式转变:

**传统 RAG**:像图书馆管理员,你问什么,他找什么,不会主动思考。

**Agentic RAG**:像资深顾问,不仅帮你找资料,还会:

* 判断是否真的需要查资料(也许我已经知道答案)
* 评估找到的资料质量(这份资料真的相关吗)
* 综合多方信息给出建议(让我对比一下这几个方案)
* 反思自己的答案(我的结论有事实支撑吗)

**核心要点回顾**:

1. **技术架构**:从线性流程到 Agent 决策中枢,从单一知识源到多源协同
2. **关键技术**:Router Agent、Multi-Agent、ReAct、Self-RAG
3. **生产部署**:模块化设计、延迟优化、成本控制、全方位可观测
4. **框架选型**:LangGraph(复杂流程)、LangChain(快速原型)、LlamaIndex(数据为中心)、Haystack(企业级)

**实践建议**:

* **开发者**:从简单 Router Agent 入手,掌握 LangGraph 构建复杂流程
* **架构师**:关注多 Agent 协作设计,平衡性能与成本
* **技术管理者**:分阶段演进,数据驱动优化,建立完善的评估体系

RAG 2.0 不是终点,而是起点。随着 LLM 能力持续提升,我们将看到更智能、更自主的 AI 系统,而 Agentic RAG 正是通往这个未来的桥梁。

---

**你在实际项目中遇到过哪些 RAG 的痛点?或者对 Agentic RAG 的某个技术细节特别感兴趣?欢迎留言交流,我们一起探讨!** 🚀