> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020301_GraphRAG/020301_核心知识点/GraphRAG构建检索与评估边界|GraphRAG构建检索与评估边界]]
---
title: 手把手教你用LangChain和Neo4j快速创建RAG应用
author: AI科技论谈
date:
url: http://mp.weixin.qq.com/s?__biz=MzU0NzM2NzY4NA==&mid=2247485069&idx=1&sn=0fd552f00c9d4e4687a0202907ceaabf&chksm=fb4e38e6cc39b1f04dd86fdeba234f8d625f29792ab12b9fc781b90f9fd47b24eff8aedebd50&mpshare=1&scene=24&srcid=05116N8gVtclIUdUK86Zex7E&sharer_shareinfo=a2c1df6877a8e8225cb0924db82e3131&sharer_shareinfo_first=a2c1df6877a8e8225cb0924db82e3131#rd
---

> 介绍利用Neo4j Aura和Neo4j Desktop存储向量索引，并在LangChain框架辅助下构建高效的检索增强生成（RAG）应用。

长按关注《AI科技论谈》

Neo4j 通过集成原生的向量搜索功能，增强了其对检索增强生成（RAG）应用的支持，这标志着一个重要的里程碑。这项新功能通过向量索引搜索处理非结构化文本，增强了 Neo4j 在存储和分析结构化数据方面的现有优势，进一步巩固了其在存储和分析结构化数据方面的领先地位。

本文详细介绍如何利用 Neo4j Desktop（本地版）和 Neo4j Aura（云服务版）来存储向量索引，并构建一个基于纯文本数据的 RAG 应用。

## 1 云服务部署

要使用基于云的 Neo4j Aura，需要按照以下步骤操作：

首先，点击链接创建一个实例（https://neo4j.com）。在设置过程中，系统会提示输入默认的用户名（neo4j）和实例的密码。请务必记下这个密码，因为设置后将无法再次查看。

创建账户后，会看到这样的界面：

实例启动并运行后，接下来的任务是生成嵌入向量并将其存储。这里采用OpenAI的嵌入技术，这需要一个OPENAI\_API\_KEY。

为了将这些嵌入向量上传到Neo4j Aura实例，需要准备好以下环境变量：NEO4J\_URI（Neo4j实例的URI）、NEO4J\_USERNAME（用户名）和NEO4J\_PASSWORD（密码）。

使用LangChain的WikipediaLoader功能，直接从Wikipedia网页中导入文章内容。

然后，将文章拆分成多个段落，并去除所有元数据，因为我们不需要存储这些信息。

```
import os
from langchain.vectorstores import Neo4jVector
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.document_loaders import WikipediaLoader
from langchain.text_splitter import CharacterTextSplitter
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough

# OPENAI API 密钥
os.environ["OPENAI_API_KEY"] = "sk-G7F8rdGxxXOWegj5nxxx3BlbkFJj7AuFUP5yyyAKKxSVTGQw"
# neo4j 凭证
NEO4J_URI="neo4j+s://9cb33544.databases.neo4j.io"
NEO4J_USERNAME="neo4j"
NEO4J_PASSWORD="rexxxJJOzDt4kjaaKgM_VyWUdT9GE4hNBXXGMNubg"

# 加载数据和分块
# 读取 Wikipedia 文章
raw_documents = WikipediaLoader(query="Leonhard Euler").load()
# 定义分块策略
text_splitter = CharacterTextSplitter.from_tiktoken_encoder(
    chunk_size=1000, chunk_overlap=20
)
# 分块文档
documents = text_splitter.split_documents(raw_documents)

# 从元数据中移除摘要
for d in documents:
    del d.metadata['summary']
```

以下代码片段可将嵌入向量导入 Neo4j 实例：

```
# 实例化 Neo4j 向量
neo4j_vector = Neo4jVector.from_documents(
    documents,
    OpenAIEmbeddings(),
    url=NEO4J_URI,
    username=NEO4J_USERNAME,
    password=NEO4J_PASSWORD
)
```

要在 Neo4j Aura 中访问和检查嵌入向量，需点击界面上的打开图标，会在浏览器中新开一个标签页。在这个新标签页中，可以查看到块和向量索引的详细信息。我们共有56个块，在系统中被识别为节点。此外，还可以在这个标签页中查看每个块对应的嵌入向量及其具体细节。

**向量检索**

这段代码片段通过使用 Neo4jVector 对象并进行相似性搜索，帮助检索与查询“Euler 在哪里长大？”相关的前 4 个相关块。这段代码默认采用余弦相似性方法来识别和排序向量之间的相似度。

```
query = "Where did Euler grow up?"
results = neo4j_vector.similarity_search(query=query, k=4)
print(results)

# 检索到的四个文档
# [Document(page_content='== Early life ==\nLeonhard Euler was born on 15 April 1707, in Basel to Paul III Euler, a pastor of the Reformed Church, and Marguerite (née Brucker), whose ancestors include a number of well-known scholars in the classics. He was the oldest of four children, having two younger sisters, Anna Maria', metadata={'title': 'Leonhard Euler', 'source': 'https://en.wikipedia.org/wiki/Leonhard_Euler'}), ...]
```

**创建链**

我们构建了一个名为`final_chain`的处理链，旨在高效地处理问题并生成答案。这个链的工作原理是：首先，它接收并传递上下文信息给`Neo4jVector retriever`，以便从Neo4j数据库中检索相关的向量。随后，链会利用一个OpenAI模型（版本为`gpt-4-1106-preview`）处理接收到的提示。最终，通过一个解析器对模型的输出进行处理，以提炼出精确的答案。`final_chain`的设计实现了在特定上下文中对问题的智能处理和答案生成，提高了整个操作的自动化和效率。

```
prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the context provided.
    
    Context: {context}
    
    Question: {question}"""
)

# 创建一个 lambda 函数将上下文传递给 Neo4jVector retriever
context_to_retriever = lambda x: x["question"]

# 创建链，将上下文赋值给 Neo4jVector retriever
final_chain = (
    RunnablePassthrough.assign(context=context_to_retriever, target=lambda x: neo4j_vector)
    | prompt
    | ChatOpenAI(model="gpt-4-1106-preview")
    | StrOutputParser()
)

result = final_chain.invoke({'question': query})

# 最终结果
print(result)
# Euler 在瑞士巴塞尔长大。
```

## 2 本地部署

如果想在本地的Neo4j Desktop中存储嵌入向量，可以直接在本地环境中运行该应用。操作起来非常简单，只需对凭证信息进行更新，其余的步骤则无需更改。

具体来说，需要分别为数据库和数据库管理系统设置用户名和密码。完成这些设置后，就可以在本地的Neo4j Desktop上顺利地执行应用程序了。

```
NEO4J_URI="bolt://localhost:7687"
NEO4J_USERNAME="neo4j"
NEO4J_PASSWORD="newpassword"
```

其余部分与上述相同。

## 3 总结

总结来说，Neo4j 通过整合其内置的向量搜索功能，显著提升了对检索增强生成（RAG）应用的支持能力。这不仅加强了其在传统结构化数据分析方面的优势，还使其能够更有效地处理非结构化文本数据。本文详细介绍了如何利用Neo4j Aura和Neo4j Desktop来存储向量索引，并在LangChain框架的辅助下，构建出高效的RAG应用。

## 推荐书单

### 《数据库系统概念》（原书第7版）

《数据库系统概念》是数据库系统方面的经典教材之一-其内容由浅入深-既包含数据库系统基本概念-又反映数据库技术新进展。本书基于该书第7版进行改编-保留其中的基本内容-压缩或删除了一些高级内容-更加适合作为国内高校计算机及相关专业本科生数据库课程教材。

**购买链接：https://item.jd.com/13502132.html**

> > ## 精彩回顾
> >
> > [机器学习新动向，用PyTorch实现液态神经网络（Liquid Neural Network）](https://mp.weixin.qq.com/s?__biz=MzU0NzM2NzY4NA==&mid=2247484998&idx=1&sn=f219f1b20977cc37940e10d7a93fe921&scene=21#wechat_redirect)
> >
> > [入门PyTorch，看这一篇就够了](https://mp.weixin.qq.com/s?__biz=MzU0NzM2NzY4NA==&mid=2247484915&idx=1&sn=59054302abd6b8493054e639ae4115f6&scene=21#wechat_redirect)
> >
> > [5款能在本地流畅运行大模型的免费工具](https://mp.weixin.qq.com/s?__biz=MzU0NzM2NzY4NA==&mid=2247484900&idx=1&sn=806c3f6cd25db7d6f19fceb6a6e3a0e4&scene=21#wechat_redirect)
> >
> > [LangChain结合DSPy，高效实现提示工程自动优化](https://mp.weixin.qq.com/s?__biz=MzU0NzM2NzY4NA==&mid=2247484773&idx=1&sn=4b2688031772f6aff9c8142bdfd2b344&scene=21#wechat_redirect)
> >
> > [LlamaIndex对比LangChain，大模型框架孰优孰劣](https://mp.weixin.qq.com/s?__biz=MzU0NzM2NzY4NA==&mid=2247484722&idx=1&sn=470ba60d59427cf83fdad3ca06fe31ec&scene=21#wechat_redirect)
> >
> > [Llama3来袭，解析最新最强开源大模型](https://mp.weixin.qq.com/s?__biz=MzU0NzM2NzY4NA==&mid=2247484763&idx=1&sn=a98b4e5b8e2fc18041b22c9fd067539f&scene=21#wechat_redirect)
> >
> > [使用LangChain和Llama-Index实现多重检索RAG](https://mp.weixin.qq.com/s?__biz=MzU0NzM2NzY4NA==&mid=2247484734&idx=1&sn=3ecee8c33ed3145843225c9b98ca3d7e&scene=21#wechat_redirect)

> > 长按关注《AI科技论谈》
> >
> > 长按访问【IT今日热榜】，发现每日技术热点