---
title: 使用 Neo4j 和 Langchain 进行知识图谱创建：详细指南
author: AI Agent入门到放弃
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIwODM1OTYzOQ==&mid=2247486343&idx=1&sn=fdaf0bde1800d0d8c96d3f33634466ab&chksm=9705123fa0729b2903729a9266401c929b81ed7de46b32a920e34f2bf7d8629d319044db69a9&mpshare=1&scene=24&srcid=0420TELlRMVNifEoTHyhN7b1&sharer_shareinfo=9870e7cdfa9e7c4c25fa9749f4daed58&sharer_shareinfo_first=9870e7cdfa9e7c4c25fa9749f4daed58#rd
---

# 使用 Neo4j 和 Langchain 进行知识图谱创建：详细指南

作者和 DALL.E-3 的图片

### 知识图谱在人工智能和数据管理中的力量和潜力

知识图谱是组织和整合信息的强大工具。它们提供了一种以实体为节点、以关系为边的结构化表示知识的方式。这种结构化表示允许进行高效的查询、分析和推理，使知识图谱在各种应用中非常有价值，从搜索引擎和推荐系统到自然语言处理和人工智能。

在人工智能领域，例如，知识图谱可以通过提供额外的上下文信息来增强机器学习模型的性能。它们可以帮助改善对自然语言的理解，通过映射不同单词或短语之间的关系。此外，它们可以通过提供丰富的结构化信息源来帮助创建更具交互性和智能的人工智能系统。

以电影数据库为例。在知识图谱中，每部电影都将是一个节点。电影的属性，如标题、上映日期和导演，将是节点的属性。其他实体，如演员，也将是节点，并且它们与电影的连接将表示为边。例如，一条边可以将一个演员节点与一个电影节点连接起来，并标有演员在电影中扮演的角色。这种结构允许高效地回答复杂的查询，例如“找出所有在[年份]发布的电影，其中包含演员X”，还允许深入的分析，例如识别演员职业生涯中的模式或了解电影类型随时间的变化趋势。因此，知识图谱不仅组织数据，还揭示了关系和见解，这些见解很难通过传统的数据分析方法发现。

然而，构建和维护知识图谱并不是一项琐碎的任务。它涉及从各种来源提取信息，确保这些信息的准确性，并在新信息可用时更新图谱。尽管存在这些挑战，但知识图谱所提供的潜在好处使它们成为活跃研究和开发的领域。

### 使用 Neo4j 创建和实施知识图谱

创建知识图谱遵循一个结构化的过程，从建立最小可行图（MVG）开始，然后逐步扩展。让我们以从 Form 10-K 报告中提取信息的示例来看看这个过程是如何展开的：

来源：DeepLearning AI 课程 'Knowledge Graphs for RAG'

1. 1. **提取**：最初，从 Form 10-K 报告中提取相关信息，这是上市公司向证券交易委员会（SEC）提交的一份全面年度报告。这些数据被解析并结构化为可管理的块，这些块作为知识图谱中的节点。

来源：DeepLearning AI 课程 'Knowledge Graphs for RAG'

1. 1. **增强**：在提取后，数据经过增强以丰富其价值。为每个块添加嵌入，为信息提供额外的上下文和深度。这一步对于使图形更加健壮并能够产生更丰富的见解至关重要。

来源：DeepLearning AI 课程 'Knowledge Graphs for RAG'

1. 1. **扩展**：一旦数据得到增强，图就准备好扩展了。这涉及将节点连接到彼此，以扩展图内的上下文和关系。通过在 Form 10-K 报告中建立信息片段之间的关系，图变得更加复杂，并更好地表示数据的相互关系。

来源：DeepLearning AI 课程 'Knowledge Graphs for RAG'

1. 1. **迭代细化**：提取、增强和扩展的过程可以根据需要重复进行，纳入额外的 Form 10-K 报告、外部数据源和用户反馈，持续细化和改进图的相关性和准确性。这种迭代方法确保了知识图谱随着时间的推移而发展，以纳入新信息并满足不断变化的分析需求。

来源：DeepLearning AI 课程 'Knowledge Graphs for RAG'

来源：DeepLearning AI 课程 'Knowledge Graphs for RAG'

1. 1. **视觉分析**：在最后阶段，可以向图添加地址节点，实现对 Form 10-K 报告的空间关系进行视觉分析和探索。这允许回答更多问题，例如识别彼此相邻的公司或分析投资公司相对于它们投资的公司的地理分布。由此产生的知

识图谱为公司披露和财务报告的各个方面提供了宝贵的见解。

通过遵循这种结构化方法，并从 Form 10-K 报告中纳入相关数据，创建一个全面且动态的知识图谱是可行的。这样的图谱不仅有助于深入了解公司披露，还能在各个领域支持知情决策。

### 使用 Neo4j 和 Langchain 从《吠陀·歌》评论 PDF 中创建基本知识图谱

让我们详细说明如何使用《吠陀·歌》数字副本，这是一部著名的精神文本，创建一个基本的知识图谱。这部文本是由斯里·斯瓦米·西瓦南达编写的，充满了丰富的信息，我们可以利用知识图谱进行组织。我们将使用 Neo4j，它帮助我们管理和结构化我们的图形，以及 Langchain，它帮助我们处理文本。

**首先，使用 Neo4j 创建一个免费帐户。对于本例，我们将使用免费层，允许创建一个实例。**

**凭证文件将包含以下细节，您将需要在随后的代码中使用这些细节：**

```
# 在使用这些详细信息进行连接之前等待 60 秒，或登录 https://console.neo4j.io 验证 Aura 实例是否可用  
NEO4J_URI=值  
NEO4J_USERNAME=neo4j  
NEO4J_PASSWORD=值  
AURA_INSTANCEID=值  
AURA_INSTANCENAME=Instance01
```

**现在让我们创建知识图谱。**

1. 1. 安装库

```
from dotenv import load_dotenv  
import os  
  
# 常见的数据处理  
import textwrap  
  
# Langchain  
from langchain_community.graphs import Neo4jGraph  
from langchain_community.vectorstores import Neo4jVector  
from langchain.text_splitter import RecursiveCharacterTextSplitter  
from langchain.chains import RetrievalQAWithSourcesChain  
from langchain.llms import OpenAI  
from langchain.embeddings import OpenAIEmbeddings  
from langchain.document_loaders import PyPDFLoader
```

1. 1. \_从 PDF 中提取文本\_：第一步是加载 PDF 文件并将其页面拆分为可管理的文本块。我们利用 langchain 库中的 PyPDFLoader 模块来完成这项任务。

```
# 加载 PDF 文件  
loader = PyPDFLoader("您的/pdf/文件路径.pdf")  
pages = loader.load_and_split()
```

1. 1. \_将文本拆分为块\_：接下来，我们将提取的文本拆分为更小的块，以便进一步处理。我们使用 langchain 中的 RecursiveCharacterTextSplitter 类来实现此目的。

```
# 将页面拆分为块  
text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=0)  
chunks = text_splitter.split_documents(pages)
```

1. 1. \_创建向量存储，生成嵌入并存储在 Neo4j 中\_：我们创建一个 Neo4jVector 对象，将文本块的嵌入存储在 Neo4j 图数据库中。这使我们能够以后有效地检索和操作嵌入。

```
# 警告控制  
import warnings  
warnings.filterwarnings("ignore")  
  
# 从凭证文件中加载环境变量  
load_dotenv('.env', override=True)  
NEO4J_URI = os.getenv('NEO4J_URI')   
  
NEO4J_USERNAME = os.getenv('NEO4J_USERNAME')  
  
NEO4J_PASSWORD = os.getenv('NEO4J_PASSWORD')  
  
NEO4J_DATABASE = os.getenv('NEO4J_DATABASE') or 'neo4j'  
NEO4J_DATABASE = 'neo4j'  
# 全局常量  
VECTOR_INDEX_NAME = 'pdf_chunks'  
VECTOR_NODE_LABEL = 'Chunk'  
VECTOR_SOURCE_PROPERTY = 'text'  
VECTOR_EMBEDDING_PROPERTY = 'textEmbedding'  
  
kg = Neo4jGraph(  
    url=NEO4J_URI, username=NEO4J_USERNAME, password=NEO4J_PASSWORD, database=NEO4J_DATABASE  
)
```

```
# 创建 Neo4j 向量存储  
neo4j_vector_store = Neo4jVector.from_documents(  
    embedding=OpenAIEmbeddings(),  
    documents=chunks,  
    url=NEO4J_URI,  
    username=NEO4J_USERNAME,  
    password=NEO4J_PASSWORD,  
    index_name=VECTOR_INDEX_NAME,  
    text_node_property=VECTOR_SOURCE_PROPERTY,  
    embedding_node_property=VECTOR_EMBEDDING_PROPERTY,  
)
```

1. 1. \_构建关系\_：我们在图中建立文本块之间的关系，指示它们的顺序和与父 PDF 文档的关联。

```
# 创建一个 PDF 节点  
cypher = """  
MERGE (p:PDF {name: $pdfName})  
RETURN p  
"""  
kg.query(cypher, params={'pdfName': "您的/pdf/文件路径.pdf"})  
  
# 使用 PART_OF 关系将块连接到其父 PDF  
cypher = """  
MATCH (c:Chunk), (p:PDF)  
WHERE p.name = $pdfName  
MERGE (c)-[newRelationship:PART_OF]->(p)  
RETURN count(newRelationship)  
"""  
kg.query(cypher, params={'pdfName': "您的/pdf/文件路径.pdf"})  
  
# 在后续块之间创建 NEXT 关系  
cypher = """  
MATCH (c1:Chunk), (c2:Chunk)  
WHERE c1.chunkSeqId = c2.chunkSeqId - 1  
MERGE (c1)-[r:NEXT]->(c2)  
RETURN count(r)  
"""  
kg.query(cypher)
```

1. 1. \_问答\_：最后，我们可以利用构建的知识图谱执行问答任务。我们从向量存储创建一个检索器，并从 PDF 文档的内容基于问题回答链创建一个聊天问答链。

```
# 从向量存储创建一个检  
  
索器  
retriever = neo4j_vector_store.as_retriever()  
  
# 从检索器创建一个聊天问答链  
chain = RetrievalQAWithSourcesChain.from_chain_type(  
    OpenAI(temperature=0),   
    chain_type="stuff",  
    retriever=retriever  
)  
  
# 提出一个问题  
question = "这份 PDF 文档的主题是什么？"  
answer = chain(  
    {"question": question},  
    return_only_outputs=True,  
)  
print(textwrap.fill(answer["answer"]))
```

**以下是检查 Neo4j 中数据的一些查询**

节点计数

```
# 返回节点计数  
kg.query("""  
         MATCH (n)  
         RETURN count(n) as nodeCount  
         """)
```

打印模式

```
kg.refresh_schema()  
print(kg.schema)
```

显示索引

```
kg.query("SHOW INDEXES")
```

**样本输出**

Neo4j 仪表板

作者在 neo4j 上的仪表板

Q&A 输出

作者的笔记本

作者的笔记本

使用知识图谱以及像 Neo4j 和 Langchain 这样的工具，我们可以将复杂的、非结构化的文本转换为易于分析的结构化、相互连接的数据。这个过程可以应用于各种类型的信息，从财务报告到精神文本。这个示例是创建知识图谱的基本说明。随着我们继续探索和发展这项技术，我们可以发现理解和解释数据的新方法。

> **LM Studio 的 Python 演示：创造性地编写代码**

---

参考文献。

1. 1. Knowledge Graphs for RAG - DeepLearning.AI
2. 2. Neo4j 图数据库与分析 | 图数据库管理系统
3. 3. Laksh-star · GitHub