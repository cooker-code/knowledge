---
title: 深度：Hermes Engineering如何用agent记忆升级skill？为什么说它只是半成品
author: Zilliz
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247512359&idx=1&sn=167dee05f7f90827d27f580450823ab5&chksm=fb71ea5af01150ca3b8a7c1b424e7848a9a8ab9b28b5e5c52676733e2650e25dbddad40ac3ea&mpshare=1&scene=24&srcid=0416iiAL5xWVsCFotrqIxKeM&sharer_shareinfo=896a99a03801abaaf1d4bd4fa2bebcea&sharer_shareinfo_first=896a99a03801abaaf1d4bd4fa2bebcea#rd
---

最近Hermes agent被讨论得沸沸扬扬的，今天，我们来深度拆解下它是怎么做Skills 闭环系统的。

相比市面上大多数 Agent 框架，它最大的特点在于能从历史交互中，提取经验、存储知识、做智能检索，然后不断更新skills，形成一个学习循环（Learning Loop）。

毋庸置疑，这个思路对复杂任务来说，带来的突破是革命性的。它让agent变得可以自主进化、不断学习，把历史记忆真正变成了工具能力，从而在类似场景中复用。

但说实话，现在的这套系统，最核心的检索系统做的实在是太粗糙了，还停留在关键词检索阶段，连找到相关历史经验都费劲，更别说打通从memory到skills的鸿沟，做到真正的进化了。

接下来，本文将用 Hermes Agent + Milvus 2.6 + 飞书的方式，搭一套能理解语义、跨会话记住操作方式、自主迭代skill的个人知识库 Agent，来解决这个问题。

# 一、Hermes Agent：架构与核心机制

要理解Hermes 的记忆设计到底厉害在哪里，我们需要现对其架构进行一个拆解。总共分四层：

L1 上下文记忆：当前会话窗口内的实时信息，会话结束后清空

L2：跨会话持久化的事实性知识，如项目技术栈、团队约定、已知结论

L3 原生检索：基于 SQLite FTS5 的关键词全文检索，用于在本地文件中定位信息

L4 Skills：以 Markdown 文档形式存储的操作流程，记录的是完成某类任务的具体步骤

L4 是四层里和其他agent最本质的区别，但原因不是其他框架没有类似概念。LangChain、AutoGPT 都有 Tool 和 Skill 的设计。区别在于来源：它们的skills需要开发者在部署前手动定义，用什么工具、按什么顺序、传什么参数，全部写在代码里，用户无法在使用过程中增加新skills。

Hermes 的 Skills 是从实际执行中自动提炼的。你用它完成一次检索任务，跨会话触发相同类型的工作流后，Learning Loop 自动评估这个流程是否值得固化，若值得就写入 Skill 文档，返回 💾 Memory updated · Skill 'xxx' created. 从此这个流程成为可复用的操作记忆，无需任何代码改动。其他框架的技能是开发者预先设计的，Hermes 的技能是 Agent 在使用中自己积累的。

但深度调研之后，我发现一件事情，要想让Hermes 学会高质量的“干中学”，其实仅靠 Hermes 的原生检索不够用。接下来，我们来拆解其原生检索系统的问题，以及我们如何使用 Milvus 对其改造的方式。

# 二、为什么加 Milvus：Hermes 原生检索有边界

Hermes 跨会话触发相同类型的历史工作流，需要使用检索系统先精准找到历史上的相似操作。

但是在检索系统的搭建上，它自带的 L3 检索是基于 SQLite FTS5，关键词倒排全文搜索只擅长精确匹配字面内容，在语义检索场景有明显短板。

典型案例：知识库里记录的是 asyncio 事件循环、异步任务调度、非阻塞 I/O，检索 Python 并发处理时，FTS5 可能一条都找不到，因为字面没有交集。

也是因此，当知识库规模超过数百份文档后，就需要我们引入向量语义检索来补齐 Hermes 的检索短板。这也是引入 Milvus 的直接原因。

## 2.1 选 Milvus 2.6 的两个实际理由

理由一：Hybrid Search（混合检索）

Milvus 2.6 支持在同一次查询中同时执行向量检索（语义）和 BM25 全文检索（关键词），用 RRF（Reciprocal Rank Fusion）算法合并两路结果。

比如，我们问 asyncio 的原理是什么，向量检索能命中语义相关内容；问 find\_similar\_task 这个函数在哪，BM25也能精确匹配代码中的函数名。

问涉及某个函数的某种类型任务时，Hybrid Search也能一次检索调用，就能给出正确结果，无需手写路由逻辑。

理由二：分层存储，实际部署内存可控

Milvus 2.6 引入三层存储（Hot 内存 / Warm SSD / Cold 对象存储），按访问频率自动冷热分级。500 份文档向量化后的知识库，无需全量常驻内存，按需加载可将内存控制在 2GB 以内，$5/月的 VPS 完全可以运行。

选型原因明确后，下面介绍系统整体架构与落地步骤。

# 三、系统架构

在这个新的体系里，Hermes思考需要做什么（Skill），Milvus回答检索什么（知识内容）。

# 四、安装部署

## 4.1 安装并初始化 Hermes

```
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

安装完成后运行 hermes，进入交互式初始化向导，完成：

LLM Provider 配置：支持 OpenAI、Anthropic、OpenRouter（含免费模型）

渠道配置：本文使用飞书机器人

## 4.2 部署 Milvus 2.6 Standalone

个人部署使用 Standalone 模式（单节点）足够：

```
curl -sfL https://raw.githubusercontent.com/milvus-io/milvus/master/scripts/standalone_embed.sh \-o standalone_embed.shbash standalone_embed.sh start验证服务状态docker ps | grep milvus
```

## 4.3 创建知识库 Collection

Schema 设计决定检索能力上限。以下结构同时支持稠密向量检索和 BM25 全文检索：

```
from pymilvus import MilvusClient, DataType, Function, FunctionTypeclient = MilvusClient(    uri="http://192.168.x.x:19530",)schema = client.create_schema(auto_id=True, enable_dynamic_field=True)schema.add_field("id", DataType.INT64, is_primary=True)# 原始文本（BM25 全文检索用）schema.add_field(    "text",    DataType.VARCHAR,    max_length=8192,    enable_analyzer=True,    enable_match=True)# 稠密向量（语义检索）schema.add_field("dense_vector", DataType.FLOAT_VECTOR, dim=1536)# 稀疏向量（BM25 自动生成，Milvus 2.6 特性）schema.add_field("sparse_vector", DataType.SPARSE_FLOAT_VECTOR)schema.add_field("source", DataType.VARCHAR, max_length=512)schema.add_field("chunk_index", DataType.INT32)# 告诉 Milvus 用BM25 把text 自动转换为 sparse_vectorbm25_function = Function(    name="text_bm25",    function_type=FunctionType.BM25,    input_field_names=["text"],    output_field_names=["sparse_vector"],)schema.add_function(bm25_function)index_params = client.prepare_index_params()# HNSW 图索引（稠密向量）index_params.add_index(    field_name="dense_vector",    index_type="HNSW",    metric_type="COSINE",    params={"M": 16, "efConstruction": 256})# BM25 倒排索引（稀疏向量）index_params.add_index(    field_name="sparse_vector",    index_type="SPARSE_INVERTED_INDEX",    metric_type="BM25")client.create_collection(    collection_name="hermes_milvus",    schema=schema,    index_params=index_params)
```

Hybrid\_search脚本

```
import sys, json                                                                                                                                                        from openai import OpenAI                                                                                                                                               from pymilvus import MilvusClient, AnnSearchRequest, RRFRanker                                                                                                            
client = MilvusClient("http://192.168.x.x:19530")                                                                                                                     oai    = OpenAI()                                                                                                                                                       COLLECTION = "hermes_milvus"                                                                                                                                              
def hybrid_search(query: str, top_k: int = 5) -> list[dict]:                                                                                                                # 1. 向量化查询                                                                                                                                                         dense_vec = oai.embeddings.create(                                                                                                                                          model="text-embedding-3-small",                                                                                                                                         input=query                                                                                                                                                         ).data[0].embedding                                                                                                                                                   
    # 2. 稠密向量检索（语义相关性）                                                                                                                                         dense_req = AnnSearchRequest(                                                                                                                                               data=[dense_vec],                                                                                                                                                       anns_field="dense_vector",                                                                                                                                              param={"metric_type": "COSINE", "params": {"ef": 128}},                                                                                                                 limit=top_k * 2       # 候选集适当放大，交给 RRF 做最终排序                                                                                                         )                                                                                                                                                                     
    # 3. BM25 稀疏向量检索（精确词项匹配）                                                                                                                                  bm25_req = AnnSearchRequest(                                                                                                                                                data=[query],                                                                                                                                                           anns_field="sparse_vector",                                                                                                                                             param={"metric_type": "BM25"},                                                                                                                                          limit=top_k * 2                                                                                                                                                     )                                                                                                                                                                     
    # 4. RRF 融合排序                                                                                                                                                       results = client.hybrid_search(                                                                                                                                             collection_name=COLLECTION,                                                                                                                                             reqs=[dense_req, bm25_req],                                                                                                                                             ranker=RRFRanker(k=60),                                                                                                                                                 limit=top_k,                                                                                                                                                            output_fields=["text", "source", "doc_type"]                                                                                                                        )                                                                                                                                                                     
    return [                                                                                                                                                                    {                                                                                                                                                                           "text":     r.entity.get("text"),                                                                                                                                       "source":   r.entity.get("source"),                                                                                                                                     "doc_type": r.entity.get("doc_type"),                                                                                                                                   "score":    round(r.distance, 4)                                                                                                                                    }                                                                                                                                                                       for r in results[0]                                                                                                                                                 ]                                                                                                                                                                     
if __name__ == "__main__":                                                                                                                                                  query= sys.argv[1] if len(sys.argv) > 1 else ""                                                                                                                         top_k  = int(sys.argv[2]) if len(sys.argv) > 2 else 5                                                                                                                   output = hybrid_search(query, top_k)                                                                                                                                    print(json.dumps(output, ensure_ascii=False, indent=2))   
```

环境与知识库就绪后，进入关键验证环节：观察 Hermes 的跨会话学习能力是否真实生效。

# 五、实战：Skills自动生成过程

Step 1　会话 1：首次指定脚本执行检索

打开 Hermes，在飞书发送指令，明确告知脚本路径和检索目标：

Hermes 通过 Terminal Tool 调用脚本，返回带本地文档来源的回答。此时 Skill 尚未创建，这只是一次普通的工具调用。

Step 2　重新提问

在飞书中清空对话记录，开启全新会话。不提脚本名称，不提路径，直接提问：

Step 3　从 Memory 更新到 Skill 正式创建

Learning Loop 触发，Memory 先行写入，Hermes 返回检索结果和回答，并明确告知：

这是 Learning Loop 的第一步动作。Memory 在此刻记录的是本次工作流的执行经验，包括：调用了哪个脚本、传入了什么参数、返回了什么结构的结果。这一步是积累，还不是固化。

Skill 正式创建

在跨会话触发确认复用价值后，Learning Loop 自动完成第二步，三个 Skill 被正式创建：

|  |  |
| --- | --- |
| Skill 名称 | 作用 |
| hybrid-search-doc-qa | 对 Memory 中的文档执行混合语义检索并生成回答 |
| milvus-docs-ingest-verification | 验证文档是否已正确摄取进知识库 |
| terminal | 执行终端命令，如运行脚本、设置环境变量等 |

需要注意，Memory 和 Skill 是两个不同层级的概念：Memory 是经验记录，Skill 是可被反复执行的封装能力。Memory 存的是发生过什么，Skill 存的是下次遇到同类问题，按什么步骤执行。Hermes 在这一刻完成的，是从积累经验到生成可复用技能的跃迁。

之后当用户提问时，Hermes 无需任何提示，自动识别意图、路由到对应 Skill、从 Memory 中召回相关文档片段，最终生成回答。用户不需要指定用哪个 Skill，这一切在后台自动完成。

另外，有一点值得单独说清楚。绝大多数 RAG 系统解决的是知识的存和取，但如何取这条链路是写死在代码里的，换个问法或者换个场景就可能失效。这套方案的不同之处在于：检索策略本身被存成了 Skill，变成了可持久化、可修改、有历史记录的对象。

在这之后💾 Memory updated · Skill 'hybrid-search-doc-qa' created. 出现，其实不是系统配置完成的标志，而是 Agent 第一次把某种行为模式判定为值得保留。

# 结语

最近各种五花八门的agent框架层出不穷，很多人可能会关心一个问题Hermes vs OpenClaw，到底有什么不同？我们如何对其进行分工。

在我看来，两个框架最大的差异在于设计取向不同。OpenClaw 的核心是编排，适合将复杂任务拆解给多个专业 Agent 分工执行，强调任务完成的自动化程度。Hermes 的核心是积累，单个 Agent 跨会话持续记忆，技能随实际使用增长，适合需要长期上下文和领域经验的场景。选哪个取决于你的问题是任务编排还是知识积累。

两者并不对立，并且，Hermes 官方提供 OpenClaw 迁移工具，可一键将 ~/.openclaw 下的、记忆文件、skills迁移过来。

作者介绍

Zilliz黄金写手：尹珉

```
```
```
```
阅读推荐

```
```
```
官宣：Zilliz Cloud&Milvus发布CLI工具与官方Skill，让AI Agent成为专业VDB运维与开发助手

用RAG的思路做agent知识管理，为什么跑不通
```
```
```
```
```
```
```

[黄仁勋GTC演讲上，Milvus为什么能够站稳非结构化数据处理C位](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247512061&idx=1&sn=5dccc84dc607489dabef2fe442f5d1bc&scene=21#wechat_redirect)

[2026年，Embedding要怎么选？（实测Gemini 、jina、Qwen、BGE、OpenAI十大模型）](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247512144&idx=1&sn=2783fc78e14c7a792a748f2053ca3284&scene=21#wechat_redirect)

[Claude Code 的记忆系统，比想象中初级](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247512194&idx=1&sn=189c8b17a0400f70f3292cb5b7750d53&scene=21#wechat_redirect)