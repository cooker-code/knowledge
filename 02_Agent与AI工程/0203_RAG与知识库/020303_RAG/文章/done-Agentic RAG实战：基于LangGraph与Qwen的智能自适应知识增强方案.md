> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020303_RAG/020303_核心知识点/RAG生命周期与评估边界|RAG生命周期与评估边界]]
---
title: Agentic RAG实战：基于LangGraph与Qwen的智能自适应知识增强方案
author: 凝思AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzNTY5MDYzOQ==&mid=2247483976&idx=1&sn=95df1d907bce640fbb65883cfd20d0ff&chksm=c3d7e13d46bd799c15673966f79f7181a75ba0b145759086e04e97c19529f37b3da609ed256e&mpshare=1&scene=24&srcid=0912PSeHDnAUy2ubHFqlyNJV&sharer_shareinfo=289b55d1272a7aeaab132684cac8e869&sharer_shareinfo_first=289b55d1272a7aeaab132684cac8e869#rd
---

检索增强生成（RAG）彻底改变了AI系统访问和利用外部知识进行推理的模式。然而，随着应用场景复杂性的不断提升，传统RAG方法的局限性也日益凸显。如今，RAG正从单一的线性流程，进化为能够根据查询复杂度和上下文，动态调整检索与生成策略的智能自适应系统。

本文将深入解析Agentic RAG系统，重点介绍如何基于**LangGraph**和Qwen构建**Agentic RAG**。在进入具体实现之前，让我们先明确什么是Agentic RAG。

### 一、理解Agentic RAG

Agentic RAG是一种更先进的RAG策略，它融合了两大核心能力：动态查询分析和自我纠错机制。它被认为是RAG发展中最成熟的形态，其核心理念是：**查询的复杂度各不相同**。研究表明，现实场景中的查询复杂度差异显著：

* **简单查询**：如“成都是哪个省的省会？”—— 语言模型即可直接回答。
* **多跳查询**：如“清朝时期，康熙皇帝什么时候派人修建圆明园，并由谁主持设计？”—— 需要经过多轮推理才能得出准确答案。

### Agentic RAG系统工作流程图

### Agentic RAG的核心流程分为以下几步：

### 1. 查询路由与分类

系统首先通过**训练好的复杂度分类器**对输入问题进行分析，这不是简单的关键词匹配，而是基于语义的**智能评估**，以确定：

* 是否需要检索，模型自身知识是否足以回答。
* 如果需要检索，问题需要多复杂的处理路径。
* 最终将问题路由到最优策略：**无需检索、单步检索处理或多跳推理。**

### 2. 动态知识获取策略

根据分类结果，系统会**动态选择最合适的知识获取方式**：

* **基于索引的检索**：适用于已有知识库即可覆盖的问题。
* **网络搜索**：适用于需要最新信息或本地知识不足的场景。
* **无需检索**：模型自身知识即可快速生成答案。

### 3. 多阶段质量保障

在生成答案的多个环节，系统通过多层评估确保结果的**准确性与可靠性**：

* **文档相关性评估**：通过置信度评分判断检索内容的匹配度。
* **幻觉检测**：识别并避免无依据的生成内容。
* **答案质量评估**：确保最终输出完整、准确且符合查询需求。

### 二、Agentic RAG实现指南

本文将落地一套自适应Agentic RAG系统：对用户查询做细粒度判断，在检索与路由之间做出智能决策，强调可复用、可扩展与工程落地。该实现展示了：

* **智能查询分析：**根据问题类型与难度自动选择最优执行路径；
* **系统化评估框架：**以离线与在线指标校验回答，确保可靠与一致；
* **自适应架构设计：**在向量库、内部API与互联网搜索之间自由路由与切换。

本方案在原始LangChain实现基础上完成重构，显著提升代码可读性与可维护性，并优化开发者体验。我们将使用以下技术栈和组件：

* **LangGraph**：编排复杂的有状态工作流；
* **Qwen**：作为主要语言模型；
* **向量数据库**：用于高效文档检索；
* **网络搜索集成**：获取实时信息；
* **全链路的评估框架**：从数据到答案的质量保障。

接下来，我们将**循序渐进地构建一个Agentic RAG系统**，并按照最有助于理解整体流程的逻辑顺序，逐一拆解各组件的功能与实现细节。

第1步：定义状态管理系统与核心常量

在**Agentic RAG**系统中，**状态管理**是整个流程的基础，它决定了信息在图结构中的流动方式，也是系统能实现动态自适应决策的核心环节。

```
import osfrom typing import List, TypedDict, Dict, Any, Literalfrom dotenv import find_dotenv, load_dotenvfrom pydantic import BaseModel, Field
from langchain_core.prompts import ChatPromptTemplatefrom langchain_core.runnables import RunnableSequencefrom langchain_core.output_parsers import StrOutputParserfrom langchain.text_splitter import RecursiveCharacterTextSplitterfrom langchain_chroma import Chromafrom langchain_community.chat_models.tongyi import ChatTongyifrom langchain_community.document_loaders import WebBaseLoaderfrom langchain_community.embeddings import HuggingFaceBgeEmbeddingsfrom langchain.schema import Documentfrom langchain import hubfrom langgraph.graph import END, StateGraph
from chinese_recursive_text_splitter import ChineseRecursiveTextSplitterfrom langchain_tavily import TavilySearch
class GraphState(TypedDict):    """    表示图中每个节点的状态。    属性：        question: 用户输入的问题        generation: LLM生成的回答        web_search: 是否需要进行网络搜索        documents: 存放本地或网络检索到的文档列表        fallback: 是否由LLM直接生成    """    question: str    generation: str    web_search: bool    documents: List[str]    fallback: bool
```

在开始编码前，我们引用了相关的依赖库。

然后，我们使用`TypedDict`来定义 **图中每个节点的状态**，既保证类型安全，又保留工作流动态处理的灵活性。

接下来，定义图中各节点的名称常量：

```
RETRIEVE = "retrieve"GRADE_DOCUMENTS = "grade_documents"GENERATE = "generate"WEBSEARCH = "websearch"LLM_FALLBACK = "llm_fallback"
```

这些常量有助于**保持代码一致性**，集中管理节点名称不仅方便重构，也能减少在工作流中引用节点时出现错误的可能性。

第2步：初始化模型文件

这段代码的作用是**初始化核心模型文件**，为后续构建Agentic RAG系统做准备，包含两个部分：初始化大语言模型和嵌入模型。

```
# 加载环境变量load_dotenv(find_dotenv())
# 加载大语言模型llm_model = ChatTongyi(model_name="qwen-max", streaming=True, temperature=0)
# 加载嵌入模型embed_model = HuggingFaceBgeEmbeddings(                model_name="../bge-large-zh-v1.5",                model_kwargs={'device': 'cuda'},                encode_kwargs={'normalize_embeddings': True}                )
```

**第3步：构建查询路由链**

**在**Agentic RAG**系统中，查询路由器是首个决策环节，其核心任务是**智能判断用户问题的类型，并选择最优知识来源**，确保系统能够高效、准确地响应各种查询。**

```
# 定义路由输出结构class RouteQuery(BaseModel):    """将用户的问题路由到最相关的数据源"""    datasource: Literal["vectorstore", "websearch", "llm_fallback"] = Field(        ...,        description="根据用户问题，将其路由到向量数据库、网络搜索或直接由LLM生成",    )
# 将 LLM 包装成结构化输出模式，输出 RouteQuery 类型structured_llm_router = llm_model.with_structured_output(RouteQuery)
# 系统提示词system = """你是一个专家级的路由器，负责判断用户问题应该选择哪个路径。1. 如果问题是闲聊或寒暄（如“你好”、“讲个笑话”、“你是谁？”），选择llm_fallback；2. 如果问题与AI技术文档相关，选择vectorstore；3. 如果需要最新实时信息，选择websearch；只输出vectorstore、websearch或llm_fallback即可。"""
# 构建提示模板route_prompt = ChatPromptTemplate.from_messages(    [        ("system", system),        ("human", "{question}"),    ])
# 组合提示模板与结构化路由模型question_router = route_prompt | structured_llm_router
def route_question(state: GraphState) -> str:    """根据路由器决定问题走向"""    print(state)    print("---路由问题---")    source: RouteQuery = question_router.invoke({"question": state["question"]})    if source.datasource == "websearch":        return WEBSEARCH    elif source.datasource == "vectorstore":        return RETRIEVE    else:        return LLM_FALLBACK
```

在这一环节，系统会对用户问题进行智能分类并选择最合适的处理路径：

* **vectorstore**：当问题涉及AI技术内容时，直接从向量数据库检索答案。
* **websearch**：当问题需要最新实时信息时，调用网络搜索获取数据。
* **llm\_fallback**：对于闲聊、寒暄等问题，直接由LLM生成答案。

通过这一智能化路由策略，**Agentic RAG**系统在知识覆盖、实时性与灵活性之间实现了平衡：本地知识库命中时快速返回高质量答案；知识库覆盖不足时，网络搜索补充最新信息；对于无需检索的场景，LLM\_FALLBACK确保系统仍能生成合理且连贯的回应。

第4步：构建文档向量检索节点

文档向量检索是**RAG系统的知识核心**，负责将用户问题与本地知识库中的高相关内容匹配，为生成答案提供可靠上下文。

```
def create_vectorstore():    """创建或加载向量存储，用于文档检索"""    chroma_path = "./chroma_langchain_db"        # 如果本地已经存在向量数据库，直接加载    if os.path.exists(chroma_path):        print("正在加载本地向量存储...")        vectorstore = Chroma(            persist_directory=chroma_path,            embedding_function=embed_model,            collection_name="rag-chroma",        )        return vectorstore.as_retriever()        # 如果没有，构建新的向量数据库    print("正在创建新的向量存储...")    urls = [        "https://aws.amazon.com/cn/what-is/large-language-model/",        "https://zhuanlan.zhihu.com/p/659386520",        "https://zhuanlan.zhihu.com/p/620342675",    ]        # 从网页加载内容    docs = [WebBaseLoader(url).load() for url in urls]    docs_list = [item for sublist in docs for item in sublist]        # 将长文本拆分为小片段    text_splitter = ChineseRecursiveTextSplitter(        chunk_size=250,  # 每段大小约 250 tokens        chunk_overlap=0  # 不重叠    )    doc_splits = text_splitter.split_documents(docs_list)        # 构建 Chroma 向量存储并持久化到本地    vectorstore = Chroma.from_documents(        documents=doc_splits,        collection_name="rag-chroma",        embedding=embed_model,        persist_directory=chroma_path,    )        print("向量存储创建完成！")    return vectorstore.as_retriever()
# 初始化检索器retriever = create_vectorstore()
def retrieve(state: Dict[str, Any]) -> Dict[str, Any]:    """从向量存储中检索相关文档"""    print("---RETRIEVE---")    question = state["question"]    # 使用预先配置好的检索器获取最语义相关的文档    documents = retriever.invoke(question)    return {"documents": documents, "question": question}
```

这一环节是整个知识库的核心，流程可以拆解为以下几个部分：

1. **加载环境变量**
   通过`load_dotenv`读取配置文件，方便灵活调用不同模型或参数，减少硬编码带来的维护成本。
2. **定义高质量知识源**
   我们选取了三篇技术文章作为示例，作为知识库的基础内容，后续也可以根据业务需要进行扩展。
3. **加载网页内容**
   利用`WebBaseLoader`抓取网页内容，并转换为标准的文档对象，便于后续处理。
4. **文本智能切分**
   通过Chinese`RecursiveTextSplitter`将长文本切分成**250 tokens的小片段**，且不设置重叠。这种切分方式可以兼顾语义完整性和检索效率，确保模型在检索时定位更加精准。
5. **构建本地向量存储**
   调用`Chroma`向量数据库，将切分后的文本片段向量化，并结合嵌入模型生成高质量语义向量，再持久化存储到本地。这意味着即使重启环境，也能快速加载，无需重复构建。
6. **实现检索节点**
   当用户输入问题时，检索器会从本地数据库中找到语义相关度最高的文档，为后续的生成环节提供精准上下文。

通过这一流程，RAG系统具备了**高效、稳定的语义检索能力**。后续再结合评分和决策机制，就能进一步过滤噪声信息，仅保留高质量、强相关的文档，从而显著提升生成内容的**准确性与可靠性**。

**第5步：构建网络搜索节点**

**为了**扩展知识覆盖范围**，Agentic RAG系统引入网络搜索节点，用于处理**实时信息**或**本地知识库未覆盖的领域问题。****

```
# 初始化 Tavily 搜索工具，最多返回 3 条结果web_search_tool = TavilySearch(max_results=3)
def web_search(state: GraphState) -> Dict[str, Any]:    print("---WEB SEARCH---")    question = state["question"]        # 获取已有文档，若无则创建空列表    documents = state.get("documents", [])        # 调用网络搜索    tavily_results = web_search_tool.invoke({"query": question})["results"]    joined_tavily_result = "\n".join(        [tavily_result["content"] for tavily_result in tavily_results]    )    web_results = Document(page_content=joined_tavily_result)        # 将搜索结果加入文档列表    if documents:        documents.append(web_results)    else:        documents = [web_results]    return {"documents": documents, "question": question}
```

**网络搜索节点**的引入是为了弥补本地向量数据库的局限，在处理**实时问题**或**未覆盖的领域知识**时显得尤为重要：

1. **接入Tavily搜索**

   Tavily是针对AI应用优化的搜索API，能够快速找到与问题最相关的网页内容。每次搜索最多返回3条结果，确保信息简洁且精准。
2. **合并搜索结果**

   搜索完成后，系统会将多条网页内容合并为一个`Document`对象，并追加到当前的文档集合中。
3. **实现混合知识检索**

   系统可同时利用本地向量库处理领域专业问题，也可通过网络搜索获取最新、全面的答案。

这种**“本地知识+网络搜索”**的混合策略，让RAG系统既能保持生成结果的**准确性**，又具备**广度和灵活性**，从而应对从技术深研到新闻热点等多样化的查询场景。

**第6步：创建文档检索评分节点**

文档检索评分器是整个流程的**质量把控环节**，用于校验从向量数据库检索出的文档是否真正契合用户问题。这一步至关重要，因为**向量相似度≠语义相关性**，有时检索结果看似匹配，但实际语境并不契合。

```
# 定义评分输出结构class GradeDocuments(BaseModel):    """用于检查检索文档是否与问题相关的二元评分"""    binary_score: str = Field(        description="判断文档是否与问题相关，仅输出 'yes' 或 'no'"    )
# 将 LLM 模型封装为结构化输出模式structured_llm_grader = llm_model.with_structured_output(GradeDocuments)
# 系统提示词system_prompt = """你是一个负责评估检索结果相关性的评分器。如果文档内容包含了与问题相关的关键词，或者语义上与问题匹配，请评为 'yes'。如果完全无关，则返回 'no'。请严格输出 'yes' 或 'no'，用于标记该文档是否与用户问题相关。"""
# 构建提示模板grade_prompt = ChatPromptTemplate.from_messages([    ("system", system_prompt),    ("human", "检索到的文档:\n\n {document} \n\n 用户问题: {question}"),])
# 组合评分链retrieval_grader = grade_prompt | structured_llm_grader
# 评分函数def grade_documents(state: Dict[str, Any]) -> Dict[str, Any]:    """    判断检索到的文档是否与问题相关。    如果有任何文档不相关，将触发 web 搜索标记。    Args:        state (dict): 当前图状态    Returns:        state (dict): 筛选出相关文档并更新 web_search 状态    """    print("---检查文档与问题的相关性---")    question = state["question"]    documents = state["documents"]    filtered_docs = []    web_search = False        for d in documents:        # 使用检索评分器评估文档相关性        score = retrieval_grader.invoke({            "question": question,            "document": d.page_content        })        grade = score.binary_score        if grade.lower() == "yes":            print("---评分结果：文档相关---")            filtered_docs.append(d)        else:            print("---评分结果：文档不相关---")            web_search = True            continue    return {"documents": filtered_docs, "question": question, "web_search": web_search}
```

其工作机制可以拆解为以下三点：

1. **结构化输出**

   通过`GradeDocuments`模型强制生成**yes**或**no**的结果，确保后续处理简单且稳定。
2. **精准提示词**

   同时检查**关键词匹配**与**语义匹配**，实现更全面的相关性评估。
3. **质量把控**

   若检索内容不相关，则直接丢弃，避免误导生成；当无合格文档时，系统会自动触发网络搜索，补充缺失信息。

这种评分机制让检索模块在**效率与准确性之间实现最佳平衡**：本地知识库命中率高时响应更快，而在知识覆盖不足时，又能自适应扩展搜索范围，从而保障用户始终获得**完整、准确的答案**。

**第7步：构建生成节点**

**生成节点是**RAG系统的大脑**，负责将用户问题与检索到的上下文信息结合，生成最终答案，实现从检索到生成的完整闭环。**

```
# 从 LangChain Hub 拉取优化好的 RAG Promptprompt = hub.pull("rlm/rag-prompt")
# 构建生成链：提示模板 → LLM → 输出解析器generation_chain = prompt | llm_model | StrOutputParser()
def generate(state: GraphState) -> Dict[str, Any]:    """使用文档和问题生成答案"""    print("---GENERATE---")    question = state["question"]    documents = state["documents"]    # 调用生成链生成最终答案    generation = generation_chain.invoke({"context": documents, "question": question})    return {"documents": documents, "question": question, "generation": generation}
def llm_fallback(state: GraphState) -> Dict[str, Any]:    """当问题无需检索或搜索时，直接由 LLM 生成答案"""    print("---LLM FALLBACK---")    question = state["question"]    generation = llm_model.invoke(question).content    return {        "question": question,        "generation": generation,        "documents": [],        "web_search": False,        "fallback": True,    }
```

生成节点的核心特点：

1. #### **优化提示模板**

   我们直接从**LangChain Hub**拉取专为**检索增强生成（RAG）**场景优化的Prompt模板。这个模板能更好地融合检索到的上下文信息，帮助模型生成更准确、更有逻辑的回答。
2. #### **输出结构化**

   通过**StrOutputParser**，输出会被规范化为干净的纯文本格式，方便后续进行评分、质检或二次处理。
3. #### **核心价值**

   生成链将**用户问题、检索文档、提示模板**有机结合，确保回答不仅完整、准确，还能直接用于**答案质量评估**和**幻觉检测**。

通过这个节点，RAG系统真正实现了**“从检索到生成”**的完整闭环，生成的内容能够无缝支持**答案质量评估**与**幻觉检测**等后续流程。

**第8步：构建生成内容幻觉检测系统**

**幻觉检测器是**RAG系统中至关重要的质量保障组件**，用于确保生成答案基于事实，避免出现“貌似合理但错误”的内容。**

```
# 定义幻觉评分输出结构class GradeHallucinations(BaseModel):    """用于检测生成答案中是否存在幻觉的二元评分"""    binary_score: bool = Field(        description="答案是否基于事实，仅输出 True 或 False"    )
# 将 LLM 封装为结构化输出模式structured_llm_grader = llm_model.with_structured_output(GradeHallucinations)
# 系统提示词system = """你是一个评分器，用于判断 LLM 生成的回答是否基于一组检索到的事实。请严格给出二元评分 'yes' 或 'no'。 'Yes' 表示答案基于/支持所提供的事实，'No' 表示答案未得到事实支持。"""
# 构建提示模板hallucination_prompt = ChatPromptTemplate.from_messages(    [        ("system", system),        ("human", "事实集合: \n\n {documents} \n\n LLM 生成内容: {generation}"),    ])
# 组合成可执行评分链hallucination_grader: RunnableSequence = hallucination_prompt | structured_llm_grader
```

幻觉检测器是RAG系统**最关键的组件之一**，用于确保生成内容**真实且有事实依据**：

1. **验证生成答案的真实性**

   将模型输出与检索到的文档逐一对比，判断信息是否有可靠支撑。
2. **二元评分**

   采用布尔值或**'yes'/'no'**的结果，避免模棱两可的判断。
3. **防止“貌似合理但错误”的回答**

   一旦检测到幻觉，系统会触发**重新生成**或**补充检索**，确保输出内容准确可靠。

这一环节显著提升了RAG系统的**可信度与安全性**，让最终生成结果既智能又可靠。

**第9步：创建答案质量评分器**

**答案质量评分器负责评估生成内容是否**真正回应用户问题**，确保输出不仅基于事实，还具有针对性和实用性。**

```
# 定义答案评分输出结构class GradeAnswer(BaseModel):    binary_score: bool = Field(        description="答案是否有效回应问题，仅输出 True 或 False"    )
# 将 LLM 封装为结构化输出模式structured_llm_grader = llm_model.with_structured_output(GradeAnswer)
# 系统提示词system = """你是一个评分器，用于判断生成的答案是否解决了用户的问题。请严格给出二元评分 'yes' 或 'no'。'Yes' 表示答案有效回应并解决了问题，'No' 表示未解决。"""
# 构建提示模板answer_prompt = ChatPromptTemplate.from_messages(    [        ("system", system),        ("human", "用户问题: \n\n {question} \n\n LLM 生成内容: {generation}"),    ])
# 组合成可执行评分链answer_grader: RunnableSequence = answer_prompt | structured_llm_grader
```

答案质量评分器用于评估生成内容是否**真正回应用户问题**：

1. **验证回答的针对性**

   **即便答案基于事实，也需判断其是否直接解决了用户的具体需求。**
2. **二元化评分**

   **采用****'yes'**或**'no'**标记，确保评估结果清晰、易于处理。
3. 触发补救措施

   若答案未能有效回应问题，将启动**额外检索**或**网络搜索**，以获取更匹配的信息。

借助答案质量评分器，系统输出能够做到**准确且切题**，进一步提升用户体验与整体可靠性。

第10步：构建完整图工作流

这一环节是**Agentic RAG系统的核心**，将前面构建的检索、生成、评分和搜索模块有机整合，形成一个**端到端智能闭环。**

```
# 决策是否进入生成环节def decide_to_generate(state):    """根据文档评分决定走 web 搜索还是直接生成答案"""    print("---评估文档---")    return WEBSEARCH if state["web_search"] else GENERATE
# 对生成内容进行幻觉与答案质量评分def grade_generation_grounded_in_documents_and_question(state):    """评分生成答案的真实性与有效性"""    print("---检查幻觉与答案质量---")    question = state["question"]    documents = state["documents"]    generation = state["generation"]    # 检查是否基于事实    score = hallucination_grader.invoke({"documents": documents, "generation": generation})    if score.binary_score:        # 检查答案是否有用        score = answer_grader.invoke({"question": question, "generation": generation})        return "useful" if score.binary_score else "not useful"    else:        return "not supported"
# 构建工作流workflow = StateGraph(GraphState)workflow.add_node(RETRIEVE, retrieve)workflow.add_node(GRADE_DOCUMENTS, grade_documents)workflow.add_node(GENERATE, generate)workflow.add_node(WEBSEARCH, web_search)workflow.add_node(LLM_FALLBACK, llm_fallback)workflow.set_conditional_entry_point(    route_question,    {WEBSEARCH: WEBSEARCH, RETRIEVE: RETRIEVE, LLM_FALLBACK: LLM_FALLBACK},)workflow.add_edge(RETRIEVE, GRADE_DOCUMENTS)workflow.add_conditional_edges(    GRADE_DOCUMENTS,    decide_to_generate,    {WEBSEARCH: WEBSEARCH, GENERATE: GENERATE},)workflow.add_conditional_edges(    GENERATE,    grade_generation_grounded_in_documents_and_question,    {"not supported": GENERATE, "useful": END, "not useful": WEBSEARCH},)workflow.add_edge(WEBSEARCH, GENERATE)workflow.add_edge(LLM_FALLBACK, END)app = workflow.compile()
# 导出图可视化app.get_graph().draw_mermaid_png(output_file_path="graph.png")
```

这个图工作流是**Agentic RAG系统的核心**，将前面构建的各模块有机整合，形成完整闭环：

1. **条件入口点**
   系统根据路由器的判断，将问题分配至**本地检索**或**网络搜索**。
2. **自适应决策逻辑**
   `decide_to_generate`：根据文档评分决定生成答案或进入网络搜索流程。
   `grade_generation_grounded_in_documents_and_question`：执行**自我纠错机制**，检测生成内容的真实性与质量，必要时触发重新生成或补充检索。
   `route_question`：负责初始问题的智能路由。
3. **动态工作流**
   编译后的工作流能根据每一步的信息质量动态调整执行路径，确保最终输出**可靠、准确**。
4. **可视化**
   最终导出`graph.png`，直观展示整个系统的执行流程。

通过这一环节，Agentic RAG系统完成了**智能、自适应的端到端问答闭环**，从用户提问到答案输出，每一步都可控、可验证，确保生成内容**高质量、高可靠性**。

LangGraph状态流程图

**第11步：创建主应用入口**

**主应用入口是**Agentic RAG系统**的用户交互界面，为系统提供了一个简洁、直观的**命令行测试环境。****

```
def format_response(result):    """提取并格式化工作流输出"""    if isinstance(result, dict):        return result.get("generation") or result.get("answer", "")    return str(result)
def main():    """自适应 RAG 系统命令行界面"""    print("=== Adaptive RAG System ===")    print("输入 'quit' 退出程序。\n")    while True:        question = input("Question: ").strip()        if question.lower() in ['quit', 'exit', 'q', '']:            break        print("Processing...")        try:            # 调用工作流生成答案            for output in app.stream({"question": question}):                result = next(iter(output.values()))            print(f"\nAnswer: {format_response(result)}\n")        except Exception as e:            print(f"Error: {e}")
if __name__ == "__main__":    main()
```

核心解析：

1. **加载环境变量**
   确保程序能正确读取API密钥和模型配置。
2. 调用工作流

   通过已编译的app，利用app.stream()实现流式输出。
3. 结果处理

   format\_response函数从输出中提取生成的答案，方便直观展示。
4. **交互体验**
   用户在终端输入问题即可获得完整的**Adaptive RAG问答体验**。

有了这个入口文件，整个系统即可完成**端到端运行和测试**，轻松展示其智能、灵活的自适应问答能力。

四、运行系统

完成环境配置与代码准备后，即可启动**Agentic RAG系统**：

```
正在加载本地向量存储...流程图导出为 graph.png=== Adaptive RAG System ===输入 'quit' 退出程序。
Question: 明天天津天气怎么样？有雨吗？
Processing...{'question': '明天天津天气怎么样？有雨吗？'}
---路由问题---{'question': '明天天津天气怎么样？有雨吗？'}
---WEB SEARCH---{'question': '明天天津天气怎么样？有雨吗？', 'documents': [Document(metadata={}, page_content='... 明天上午阴有阵雨,下午阴转多云,南风3-4级转微风,今天夜间最低气温23度,明天白天最高气温27度,相对湿度90%到67%.今夜我市有明显降雨过程,降雨时伴有雷电,短时强降\n今起三天，我国降雨将分为南北两条雨带，北方降雨集中在华北、东北等地，明天起新一轮降雨将来袭；南方雨带位于云南、贵州、四川盆地西部一带，部分地区有大到暴雨。\n星期五 08/22. 阴. 西北风 ; 星期六 08/23. 雷阵雨. 北风 ; 星期日 08/24. 雷阵雨. 东北风 ; 星期一 08/25. 晴. 北风 ; 星期二 08/26. 多云. 东风.')]}
---GENERATE---{'documents': [Document(metadata={}, page_content='... 明天上午阴有阵雨,下午阴转多云,南风3-4级转微风,今天夜间最低气温23度,明天白天最高气温27度,相对湿度90%到67%.今夜我市有明显降雨过程,降雨时伴有雷电,短时强降\n今起三天，我国降雨将分为南北两条雨带，北方降雨集中在华北、东北等地，明天起新一轮降雨将来袭；南方雨带位于云南、贵州、四川盆地西部一带，部分地区有大到暴雨。\n星期五 08/22. 阴. 西北风 ; 星期六 08/23. 雷阵雨. 北风 ; 星期日 08/24. 雷阵雨. 东北风 ; 星期一 08/25. 晴. 北风 ; 星期二 08/26. 多云. 东风.')], 'question': '明天天津天气怎么样？有雨吗？', 'generation': '明天天津上午有阵雨，下午转为多云。全天南风3-4级转微风，气温在23到27度之间。'}
---检查幻觉与答案质量---
Answer: 明天天津上午有阵雨，下午转为多云。全天南风3-4级转微风，气温在23到27度之间。
```

从运行示例可以看到，系统完整执行了自适应的推理流程。

首先，它会对问题进行智能分析，判断是直接检索本地知识库还是调用网络搜索。在这个示例中，问题涉及最新天气情况，因此系统选择了实时搜索，从互联网上获取了最新的气象信息。

接着，系统将检索到的内容与问题进行匹配，生成自然语言回答。生成结果会经过幻觉检测与质量评估，以确保信息准确、逻辑合理并且与上下文一致。

最终，系统输出了一条可靠的回答：

> 明天天津上午有阵雨，下午转为多云，南风 3-4 级转微风，气温在 23 到 27 度之间。

整个过程充分体现了**Agentic RAG系统的自适应能力**——它能灵活选择信息来源，快速整合数据，并通过多重验证机制保证回答的准确性和可信度。

四、总结与未来方向

本文深入探讨了**Agentic RAG系统**的核心理念与实现路径，详细解析了如何基于**LangGraph**构建有状态的工作流，并结合**Qwen模型**、本地向量检索与网络搜索，实现具备**动态路由、智能决策、自我纠错**的增强生成系统。该方案能够精准识别查询复杂度，灵活选择最优检索与生成策略，并通过多阶段质量控制大幅提升答案的**准确性、可靠性和时效性**。

展望未来，**Agentic RAG**将向**多模态融合**与**自主智能化**方向持续演进。一方面，引入**图像、视频、表格等多模态数据**进行统一检索与生成，扩展系统在复杂任务中的适用性与表达能力；另一方面，结合**强化学习（RLHF）**及**反馈驱动的优化机制**，系统将具备持续迭代、动态学习的能力，实现更加精准的任务决策与答案优化。此外，借助**大规模知识图谱**与**分布式检索架构**，Agentic RAG有望实现**更高效、更低延迟**的知识利用，推动智能应用迈向更智能、更自适应的新阶段。

---

写这篇文章花了我很多时间与心力，如果您觉得内容对您有所启发，欢迎点赞并关注「凝思AI」——您的支持，是我持续创作优质内容的不竭动力！❤️❤️❤️

---

---

专注于人工智能与语言、视觉、数据交汇处的前沿探索，涵盖大语言模型（LLMs）、多模态模型、智能体框架、机器学习方法，以及基于数据的潜在空间建模等。