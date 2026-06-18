> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020301_GraphRAG/020301_核心知识点/GraphRAG图谱构建检索与多模态边界|GraphRAG图谱构建检索与多模态边界]]、[[02_Agent与AI工程/0203_RAG与知识库/020301_GraphRAG/020301_核心知识点/GraphRAG构建检索与评估边界|GraphRAG构建检索与评估边界]]
---
title: 04.构建图RAG管道：用LangChain+Neo4j构建与评估图RAG管道的全流程
author: AI技术研学社
date:
url: https://mp.weixin.qq.com/s?__biz=MzkxNDQ5NjU2OQ==&mid=2247483765&idx=1&sn=8b0a3d9377499d97cd9a0aac29356955&chksm=c0aa3784623a5612b46ae6d1d41f84d03f0abce07b4c95325ce1da71aa5feba3250eff442f1c&mpshare=1&scene=24&srcid=0926PiIlqv74vdbY4OvLibtb&sharer_shareinfo=3d2ad1b074655ba7b739201b1b6fe7bb&sharer_shareinfo_first=3d2ad1b074655ba7b739201b1b6fe7bb#rd
---

大家好，上次我和大家分享了怎么用LangChain配合图数据库Neo4j构建知识图谱应用，想必大家对这块内容已经有了基本的了解。这次我就接着带大家使用LangChain和Neo4j实现一个真实的图RAG管道，然后我还会采用最简单的方式，也就是LangChain中一个简单的用来评估模型答案准确性的类`QAEvalChain`对图RAG结果的准确性进行简单的评估，简单的模拟真实开发场景。

## 1.构建知识图谱

在之前的分享中，我随便在网上摘抄了一段非常简单的文本来呈现知识图谱的构建流程，这样做的目的主要是为了研究学习，让大家看清楚整个流程。在实际的工作中，引入的数据源可不会这么简单，为了更贴近真实场景，这次我会使用一份关于滑雪的PDF文件来构建知识图谱，整个构建过程依次包含PDF文本提取、文本分块、图文档生成与保存四个阶段。

### 1.1 引入包

```
import os
from dotenv import load_dotenv

from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import PyPDFLoader
from langchain_experimental.graph_transformers import LLMGraphTransformer
from langchain_neo4j import Neo4jGraph
from langchain_openai import ChatOpenAI
```

### 1.2 获取Neo4j实例连接凭证

```
load_dotenv()

URI = os.getenv("NEO4J_URI")
USER = os.getenv("NEO4J_USER")
PASS = os.getenv("NEO4J_PASS")
```

### 1.3 提取PDF文本

对于PDF文件，这里使用的是LangChain自带的用来导入PDF文件并从中提取文本的工具`PyPDFLoader` 。 通过调用`PyPDFLoader`的`load()`方法，会返回一个Document列表，列表中每个Document对应PDF中的一页，列表的大小对应PDF的页数。

> “
>
> Tip：LangChain中的Document是用来**表示文档内容片段的类**， 作用主要是**传递文档的内容和元数据**。

```
pdf_path = "data/skiing.pdf"
loader = PyPDFLoader(pdf_path)
# pages是一个Document列表
pages = loader.load()
```

### 1.4 文本分块

为了**规避大语言模型上下文窗口的限制**，需将原文本切分成更小的块，分块这里我们采用最为常用的递归切分。

> “
>
> Tip：递归切分演练场

```
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500, 
    chunk_overlap=50
)

chunks = text_splitter.split_documents(pages)
```

### 1.5 生成图文档

```
llm = ChatOpenAI(model="gpt-5-nano", temperature=0)
llm_transformer = LLMGraphTransformer(llm=llm)

# graph_documents是包含节点与关系的图文档
graph_documents = llm_transformer.convert_to_graph_documents(chunks)
print(f"Nodes:{graph_documents[0].nodes}")
print(f"Relationships:{graph_documents[0].relationships}")
```

### 1.6 保存图文档

```
graph = Neo4jGraph(url=URI, username=USER, password=PASS)
graph.add_graph_documents(
    graph_documents,
    # 图谱中要包含引用的源
    include_source = True
)
```

### 1.7 完整代码

```
import os
from dotenv import load_dotenv

from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import PyPDFLoader
from langchain_experimental.graph_transformers import LLMGraphTransformer
from langchain_neo4j import Neo4jGraph
from langchain_openai import ChatOpenAI  

load_dotenv()

URI = os.getenv("NEO4J_URI")
USER = os.getenv("NEO4J_USER")
PASS = os.getenv("NEO4J_PASS")

pdf_path = "data/skiing.pdf"
loader = PyPDFLoader(pdf_path)
pages = loader.load()

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500, 
    chunk_overlap=50
)
chunks = text_splitter.split_documents(pages)

llm = ChatOpenAI(model="gpt-5-nano", temperature=0)
llm_transformer = LLMGraphTransformer(llm=llm)
graph_documents = llm_transformer.convert_to_graph_documents(chunks)
print(f"Nodes:{graph_documents[0].nodes}")
print(f"Relationships:{graph_documents[0].relationships}")

graph = Neo4jGraph(url=URI, username=USER, password=PASS)
graph.add_graph_documents(
    graph_documents,
    include_source = True
)
```

执行结果如下图所示：

## 2.设置检索

之前我们简单分享了使用`GraphCypherQAChain`基于Neo4j数据库进行问答的简单流程，这是图RAG应用最基础的功能。在真实的使用场景中，大家可能会发现大语言模型与图配合得并不理想——无法生成正确的Cypher查询语句。

什么原因？这是因为大语言模型理解语言，但未必理解数据库模式。在人工智能领域有句名言\*\*“垃圾进，垃圾出”**，这既可以指**输入数据的质量\*\*，也可以指**提示的有效性**，我们这里主要讨论后者——提示的有效性，目的是创建出最佳提示，让模型**理解图的模式(Schema)**。

模式指的是数据的组织方式。在SQL中，模式中包含了表、表中的列以及表与表之间的关系。对于图而言，模式**描述的是节点之间的关联关系**。要获取图的模式，有两种方式：

1. 在Neo4j浏览器中执行`CALL db.schema.visualization`

2. 在使用LangChain进行图RAG编码时，让LangChain告知模式。

```
graph = Neo4jGraph(url=URI, username=USER, password=PASS)
# 查看图的模式
print(graph.schema)
```

```
# 启用模式增强，让框架从Neo4j中提取更详细的模式
enhanced_graph = Neo4jGraph(url=URI, username=USER, password=PASS, enhanced_schema=True)
# 查看图的模式
print(enhanced_graph.schema)
```

关于这个`schema`，`GraphCypherQAChain`内部会在查询时自动提取和传递，无需手动提取和传递，这是LangChain框架封装的便捷功能，目的是确保大语言模型始终能获取到最新的`schema`，生成更准确的Cypher查询。

如果我们还想更进一步，我们可以把正确示例包含在提示中，也就是`few-shot prompt`，这会让生成的Cypher更加准确。接下来我们就以这个为例，看看具体的代码。

### 2.1 引入包

```
import os
from dotenv import load_dotenv

from langchain_neo4j import Neo4jGraph,GraphCypherQAChain
from langchain_openai import ChatOpenAI
from langchain.prompts import PromptTemplate
```

### 2.2 获取Neo4j实例连接凭证

```
load_dotenv()

URI = os.getenv("NEO4J_URI")
USER = os.getenv("NEO4J_USER")
PASS = os.getenv("NEO4J_PASS")
```

### 2.3 创建数据库问答链

```
llm = ChatOpenAI(model="gpt-5-nano", temperature=0)
graph = Neo4jGraph(url=URI, username=USER, password=PASS, enhanced_schema=True)

CYPHER_GENERATION_TEMPLATE = """
    You are an expert Neo4j Developer translating user questions into Cypher to
    answer questions about skiing.
    Convert the user's question based on the schema.
    When you are presented with query properties such as id's like "alpine touring",
    be sure to convert the first letter to capital case, such as "Alpine Touring"
    before you run the Cypher query.

    For example, if I were  to ask "Tell me about telemark skiing," you should create
    a Cypher query that finds all nodes with the id "Telemark Skiing" and then find
    all nodes connected to those nodes and use those to forumulate your answer, like this:

    MATCH (a {id: "Telemark Skiing"})-[r]-(b)
    RETURN a, r, b

    Schema:{schema}
    Question: {question}
"""

cypher_generation_prompt = PromptTemplate(
    template = CYPHER_GENERATION_TEMPLATE,
    input_variables = ["schema", "question"]
)

cypher_chain = GraphCypherQAChain.from_llm(
    llm = llm,
    graph = enhanced_graph,
    prompt = cypher_generation_prompt,
    verbose = True,
    allow_dangerous_requests = True
)
```

### 2.4 查询图数据库

```
cypher_chain.invoke({"query": "Tell me about freestyle?"})
```

### 2.5 完整代码

```
import os
from dotenv import load_dotenv

from langchain_neo4j import Neo4jGraph,GraphCypherQAChain
from langchain_openai import ChatOpenAI
from langchain.prompts import PromptTemplate

load_dotenv()

URI = os.getenv("NEO4J_URI")
USER = os.getenv("NEO4J_USER")
PASS = os.getenv("NEO4J_PASS")  

llm = ChatOpenAI(model="gpt-5-nano", temperature=0)
graph = Neo4jGraph(url=URI, username=USER, password=PASS, enhanced_schema=True)

CYPHER_GENERATION_TEMPLATE = """
    You are an expert Neo4j Developer translating user questions into Cypher to
    answer questions about skiing.
    Convert the user's question based on the schema.
    When you are presented with query properties such as id's like "alpine touring",
    be sure to convert the first letter to capital case, such as "Alpine Touring"
    before you run the Cypher query.

    For example, if I were  to ask "Tell me about telemark skiing," you should create
    a Cypher query that finds all nodes with the id "Telemark Skiing" and then find
    all nodes connected to those nodes and use those to forumulate your answer, like this:

    MATCH (a {id: "Telemark Skiing"})-[r]-(b)
    RETURN a, r, b

    Schema:{schema}
    Question: {question}
"""

cypher_generation_prompt = PromptTemplate(
    template = CYPHER_GENERATION_TEMPLATE,
    input_variables = ["schema", "question"]
)

cypher_chain = GraphCypherQAChain.from_llm(
    llm = llm,
    graph = enhanced_graph,
    prompt = cypher_generation_prompt,
    verbose = True,
    allow_dangerous_requests = True
)

cypher_chain.invoke({"query": "Tell me about freestyle?"})
```

## 3.评估图RAG管道

对于采用大语言模型构建的任何应用，都必须在构建完成后制定测试和评估计划，评估其是否满足上生产环境的条件，这一步至关重要。即使对其进行了细微的调整，最好的做法也是对其再次进行评估，否则如何确认修改的有效性。

截止目前，我们已经构建了知识图谱，设置了检索功能，接下来我们要做的就是判断这个系统是否真的有效。我们的目标很简单——**确保答案既相关又准确**。

**准确性**指的是模型是否正确回答了用户问题，准确性最简单的评估方式是通过给模型提供一系列已标注答案的问题让模型生成自己的答案，然后将模型的输出与预期答案进行对比评分。

理想情况下我们应当准备大量评估问题，不过这样做既有挑战性，也非常耗时，所以我们就让模型来代劳。补充一下，目前市面上有许多评估人工智能应用性能的工具包，其中包含LangChain中的LangSmith，但我们这里主要使用的是LangChain中一个简单的类`QAEvalChain`，它可以用来评估模型生成的答案是否匹配参考答案，我们具体来看看怎么做。

### 3.1 引入包

```
import os
from dotenv import load_dotenv

from langchain.evaluation.qa.eval_chain import QAEvalChain
from langchain_neo4j import Neo4jGraph,GraphCypherQAChain
from langchain_openai import ChatOpenAI
```

### 3.2 获取Neo4j实例凭证

```
load_dotenv()

URI = os.getenv("NEO4J_URI")
USER = os.getenv("NEO4J_USER")
PASS = os.getenv("NEO4J_PASS")
```

### 3.3 构造测试集

需要注意这里为了演示我只构造了少量测试集，对于测试集来说最佳的做法是尽可能构造大规模、包含多样化查询问题的测试集。

```
examples = [
    {"query": "What sports is the International Ski And Snowboard Federation responsible for?",
     "answer": "Alpine Skiing, Freestyle Skiing, Snowboarding, Nordic Combined, Ski Jumping, Cross-Country Skiing"},
    {"query": "What activity are ski poles not used in?", "answer": "Ski jumping"},
    {"query": "Who do athletes get help from?", "answer": "Coaches, Peer Mentors, and Sports Psychologists"},
]
```

### 3.4 创建数据库问答链

```
graph = Neo4jGraph(url=URI, username=USER, password=PASS, enhanced_schema=True)  
llm = ChatOpenAI(model="gpt-5-nano", temperature=0)
chain = GraphCypherQAChain.from_llm(
    graph=graph,
    llm=llm,
    verbose=True,
    allow_dangerous_requests=True,
)
```

### 3.5  执行并保存模型答案

```
predictions = []
for ex in examples:
    response = chain.invoke({"query": ex["query"]})
    predictions.append({"result": response["result"].strip()})
```

### 3.6  使用模型进行评估

```
eval_chain = QAEvalChain.from_llm(llm)
results = eval_chain.evaluate(examples, predictions)
```

### 3.7 构造打印评估结果

```
correct = 0
for i, res in enumerate(results):
    print(f"Query: {examples[i]['query']}")
    print(f"Prediction from graph: {predictions[i]['result']}")
    print(f"Gold answer: {examples[i]['answer']}")
    print(f"Grade: {res['results']}")
    print("---")
    if res["results"] == "CORRECT":
        correct += 1

accuracy = correct / len(examples)
print(f"Graph QA Accuracy: {accuracy:.2f}")
```

### 3.8 完整代码

```
import os
from dotenv import load_dotenv

from langchain.evaluation.qa.eval_chain import QAEvalChain
from langchain_neo4j import Neo4jGraph,GraphCypherQAChain
from langchain_openai import ChatOpenAI  

load_dotenv()

URI = os.getenv("NEO4J_URI")
USER = os.getenv("NEO4J_USER")
PASS = os.getenv("NEO4J_PASS")

examples = [
    {"query": "What sports is the International Ski And Snowboard Federation responsible for?",
     "answer": "Alpine Skiing, Freestyle Skiing, Snowboarding, Nordic Combined, Ski Jumping, Cross-Country Skiing"},
    {"query": "What activity are ski poles not used in?", "answer": "Ski jumping"},
    {"query": "Who do athletes get help from?", "answer": "Coaches, Peer Mentors, and Sports Psychologists"},
]

graph = Neo4jGraph(url=URI, username=USER, password=PASS, enhanced_schema=True)  
llm = ChatOpenAI(model="gpt-5-nano", temperature=0)
chain = GraphCypherQAChain.from_llm(
    graph=graph,
    llm=llm,
    verbose=True,
    allow_dangerous_requests=True,
)

predictions = []
for ex in examples:
    response = chain.invoke({"query": ex["query"]})
    predictions.append({"result": response["result"].strip()})  
    
eval_chain = QAEvalChain.from_llm(llm)
results = eval_chain.evaluate(examples, predictions)  

correct = 0
for i, res in enumerate(results):
    print(f"Query: {examples[i]['query']}")
    print(f"Prediction from graph: {predictions[i]['result']}")
    print(f"Gold answer: {examples[i]['answer']}")
    print(f"Grade: {res['results']}")
    print("---")
    if res["results"] == "CORRECT":
        correct += 1

accuracy = correct / len(examples)
print(f"Graph QA Accuracy: {accuracy:.2f}")
```

由于模型生成答案的随机性，可能每次结果都不一样，所以应当运行多次，这其实也是需要大量测试问题的原因，目的是为了更好的理解评分分布。我们采用的这种方法仅评估基础准确率是否符合预期，要知道还有很多其它指标用来对图RAG进行评估，比如实用性、相关性，甚至图的可追溯性。

## 4.总结

这次分享我们完整走了一遍图RAG管道的构建流程：从用PDF构建滑雪领域知识图谱，到优化提示词提升检索准确性，再到用QAEvalChain评估系统性能。希望通过这个案例，能让大家对图RAG真实构建过程有一定的了解，对大家有点帮助。

关于图RAG的这个系列，目前就暂告一段落了，至于市面上火热的GraphRAG、LightRAG等开源框架，后期根据时间情况再看，可能会不定期更新，好了，后面再会了：）