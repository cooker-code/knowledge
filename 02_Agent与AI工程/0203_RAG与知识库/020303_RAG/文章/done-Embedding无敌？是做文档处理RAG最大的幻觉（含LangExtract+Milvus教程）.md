> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020303_RAG/020303_核心知识点/RAG生命周期与评估边界|RAG生命周期与评估边界]]
---
title: Embedding无敌？是做文档处理RAG最大的幻觉（含LangExtract+Milvus教程）
author: Zilliz
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510140&idx=1&sn=b4314a435c9e4e010856dbe63bac0ab3&chksm=fb79724962de678bc9a4aba297d06b511a8a5973a1c55c640982359337107d257118b5a60514&mpshare=1&scene=24&srcid=0912GPXaEddNcUgLPZWxINer&sharer_shareinfo=37bdf74610b11504f29180664ba3afea&sharer_shareinfo_first=37bdf74610b11504f29180664ba3afea#rd
---

前段时间，我们开源了代码上下文检索工具Claude Context。

在这之后，我们听到了很多支持的声音：Claude Code 与 Gemini 弃用代码索引、仅用 grep 方案，存在召回率低、检索无关内容多、token 消耗虚高的问题。

同时，也有开发者提出质疑：相似度不等于逻辑推理；grep、静态语法分析、RAG 无法解决复杂代码逻辑问题；语义相似难以区分接口、抽象等各类代码元素……

[Claude Code与Gemini放弃代码索引，是一步烂棋](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510107&idx=1&sn=3299448f5b8d4349fc1c6e873ecbfe78&scene=21#wechat_redirect)

那么，到底谁对谁错？

**事实是，全都没错。**Claude Context 能提升代码检索效率，但无法解决所有问题，也不会完全取代 grep。

**更重要的是，在实际落地中，embedding、全文检索、标签任何一个技术方案都不是万能的**，代码检索需 grep 与 embedding 结合，高质量 RAG 需全文与语义相似度检索结合，自动驾驶视频数据检索需标签与语义结合。

那么在实际落地中，如何将不同技术方案做好融合？

本文将结合谷歌开源结构化信息提取工具 LangExtract 与 Milvus，手把手教大家打造智能文档处理和检索系统。

这个方案的优势在于，**LangExtract擅长从非结构化文档中提取结构化信息，Milvus擅长做语义相似度检索，**双剑合璧，更适合法律、医疗、教育、法律等对内容准确度有更高要求的AI落地场景。

# **01**

# **LangExtract 解读**

谷歌LangExtract是一个 Python 库，可以通过LLM （支持 Gemini 系列、Ollama 本地模型、Gemma 开源模型等）根据用户定义的指令从**非结构化文本文档（如临床记录、合同等）中提取结构化信息**。

才刚发布几周，GitHub上已经有13.3K stars了。

其能力与优势包括：

* 自定义的结构化信息（姓名、年龄、籍贯、刑期、罪名等）提取能力。（用户需要提供提示词，以及一组高质量Few-Shot示例来定义 schema。）
* 精确源文本溯源能力，最大限度避免模型幻觉。
* 长上下文处理能力，通过分区块+多线程并行+多轮抽取的策略，可以处理百万字符内容。
* 数据标准化与可视化（生成 HTML 展示结果）的能力

效果展示

**具体场景中，LangExtract是可解决医疗、法律等领域手动提取信息费时、传统****NLP****工具训练复杂的问题，也适用于电子取证分析开示场景。**

比如，法律条文有更新的背景下，传统RAG只能召回所有的相关文本，但是LangExtract提取的日期等信息，就能帮助律师判断案件适用的条款与版本。

再比如，在医疗场景中“帮我找到2019年-2023年，年龄50岁以上，呼吸科所有CT结果中，出现某某特性肺结节的结果”就是一个典型的混合了标签+语义的混合检索场景。那么就可以用LangExtract+普通大模型提取结构化信息+语义信息，然后存在Milvus向量数据库做统一的检索与召回。

具体用法参考：[Milvus 2.5:全文检索上线,标量过滤提速,易用性再突破！](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247506604&idx=1&sn=7f602050e981e8861784634b6449e79b&scene=21#wechat_redirect)

# 02

# LangExtract + Milvus

# 构建高质量智能文档处理检索系统

本指南演示如何使用 LangExtract 与 Milvus 构建智能文档处理和检索系统。

LangExtract 是一个 Python 库，使用大型语言模型（LLM）从非结构化文本文档中提取结构化信息，并提供精确的来源定位。该系统将 LangExtract 的提取能力与 Milvus 的向量存储相结合，实现语义相似性搜索和精确的元数据过滤。

这种集成对于内容管理、语义搜索、知识发现以及基于提取的文档属性构建推荐系统特别有价值。

## 前提条件

在运行此笔记本之前，请确保已安装以下依赖项：

```
```
! pip install --upgrade pymilvus langextract google-genai requests tqdm pandas
```

在此示例中，我们将使用 Gemini 作为 LLM。您需要准备API 密钥GEMINI_API_KEY作为环境变量。
```

```
```
import osos.environ["GEMINI_API_KEY"] = "AIza*****************"
```

定义 LangExtract + Milvus 流程
```

我们将定义一个使用 LangExtract 进行结构化信息提取和 Milvus 作为向量存储的流程。

```
```
import langextract as lximport textwrapfrom google import genaifrom google.genai.types import EmbedContentConfigfrom pymilvus import MilvusClient, DataTypeimport uuid
```

配置和设置
```

让我们为集成配置全局参数。我们将使用 Gemini 的嵌入模型为文档生成向量表示。

```
```
genai_client = genai.Client()COLLECTION_NAME = "document_extractions"EMBEDDING_MODEL = "gemini-embedding-001"EMBEDDING_DIM = 3072  # Default dimension for gemini-embedding-001
```

初始化 Milvus 客户端
```

现在让我们初始化 Milvus 客户端。为了简单起见，我们将使用本地数据库文件，但这可以轻松扩展到完整的 Milvus 服务器部署。

```
```
client = MilvusClient(uri="./milvus_demo.db")
```
```

> 关于 `MilvusClient` 的参数：
>
> * 将 `uri` 设置为本地文件，例如 `./milvus.db`，是最方便的方法，因为它会自动使用 Milvus Lite 将所有数据存储在此文件中。
> * 如果您有大规模数据，可以在 docker 或 kubernetes 上设置性能更高的 Milvus 服务器。在此设置中，请使用服务器 uri，例如 `http://localhost:19530`，作为您的 `uri`。
> * 如果您想使用 Zilliz Cloud（Milvus 的完全托管云服务），请调整 `uri` 和 `token`，它们对应于 Zilliz Cloud 中的公共端点和 API 密钥。

## 样本数据准备

在此演示中，我们将使用电影描述作为样本文档。这展示了 LangExtract 从非结构化文本中提取结构化信息（如类型、角色和主题）的能力。

```
```
sample_documents = [    "John McClane fights terrorists in a Los Angeles skyscraper during Christmas Eve. The action-packed thriller features intense gunfights and explosive scenes.",    "A young wizard named Harry Potter discovers his magical abilities at Hogwarts School. The fantasy adventure includes magical creatures and epic battles.",    "Tony Stark builds an advanced suit of armor to become Iron Man. The superhero movie showcases cutting-edge technology and spectacular action sequences.",    "A group of friends get lost in a haunted forest where supernatural creatures lurk. The horror film creates a terrifying atmosphere with jump scares.",    "Two detectives investigate a series of mysterious murders in New York City. The crime thriller features suspenseful plot twists and dramatic confrontations.",    "A brilliant scientist creates artificial intelligence that becomes self-aware. The sci-fi thriller explores the dangers of advanced technology and human survival.",    "A romantic comedy about two friends who fall in love during a cross-country road trip. The drama explores personal growth and relationship dynamics.",    "An evil sorcerer threatens to destroy the magical kingdom. A brave hero must gather allies and master ancient magic to save the fantasy world.",    "Space marines battle alien invaders on a distant planet. The action sci-fi movie features futuristic weapons and intense combat in space.",    "A detective investigates supernatural crimes in Victorian London. The horror thriller combines period drama with paranormal investigation themes.",]print("=== LangExtract + Milvus Integration Demo ===")print(f"Preparing to process {len(sample_documents)} documents")
```

设置 Milvus 集合
```

在存储提取的数据之前，我们需要创建一个具有适当模式的 Milvus 集合。此集合将存储原始文档文本、向量嵌入和提取的元数据字段。

```
```
print("\n1. Setting up Milvus collection...")# Drop existing collection if it existsif client.has_collection(collection_name=COLLECTION_NAME):    client.drop_collection(collection_name=COLLECTION_NAME)    print(f"Dropped existing collection: {COLLECTION_NAME}")# Create collection schemaschema = client.create_schema(    auto_id=False,    enable_dynamic_field=True,    description="Document extraction results and vector storage",)# Add fields - simplified to 3 main metadata fieldsschema.add_field(    field_name="id", datatype=DataType.VARCHAR, max_length=100, is_primary=True)schema.add_field(    field_name="document_text", datatype=DataType.VARCHAR, max_length=10000)schema.add_field(    field_name="embedding", datatype=DataType.FLOAT_VECTOR, dim=EMBEDDING_DIM)# Create collectionclient.create_collection(collection_name=COLLECTION_NAME, schema=schema)print(f"Collection '{COLLECTION_NAME}' created successfully")# Create vector indexindex_params = client.prepare_index_params()index_params.add_index(    field_name="embedding",    index_type="AUTOINDEX",    metric_type="COSINE",)client.create_index(collection_name=COLLECTION_NAME, index_params=index_params)print("Vector index created successfully")
```

定义提取模式
```

LangExtract 使用提示和示例来指导 LLM 提取结构化信息。让我们为电影描述定义提取模式，指定要提取的信息以及如何对其进行分类。

```
```
print("\n2. Extracting tags from documents...")# Define extraction prompt - for movie descriptions, specify attribute value rangesprompt = textwrap.dedent(    """\    Extract movie genre, main characters, and key themes from movie descriptions.    Use exact text for extractions. Do not paraphrase or overlap entities.    For each extraction, provide attributes with values from these predefined sets:    Genre attributes:    - primary_genre: ["action", "comedy", "drama", "horror", "sci-fi", "fantasy", "thriller", "crime", "superhero"]    - secondary_genre: ["action", "comedy", "drama", "horror", "sci-fi", "fantasy", "thriller", "crime", "superhero"]    Character attributes:    - role: ["protagonist", "antagonist", "supporting"]    - type: ["hero", "villain", "detective", "military", "wizard", "scientist", "friends", "investigator"]    Theme attributes:    - theme_type: ["conflict", "investigation", "personal_growth", "technology", "magic", "survival", "romance"]    - setting: ["urban", "space", "fantasy_world", "school", "forest", "victorian", "america", "future"]    Focus on identifying key elements that would be useful for movie search and filtering.""")
```
```

## 提供示例以改善提取效果

为了提高提取的质量和一致性，我们将为 LangExtract 提供一些示例。这些示例演示了预期的格式，并帮助模型理解我们的提取要求。

```
```
# Provide examples to guide the model - n-shot examples for movie descriptions# Unify attribute keys to ensure consistency in extraction resultsexamples = [    lx.data.ExampleData(        text="A space marine battles alien creatures on a distant planet. The sci-fi action movie features futuristic weapons and intense combat scenes.",        extractions=[            lx.data.Extraction(                extraction_class="genre",                extraction_text="sci-fi action",                attributes={"primary_genre": "sci-fi", "secondary_genre": "action"},            ),            lx.data.Extraction(                extraction_class="character",                extraction_text="space marine",                attributes={"role": "protagonist", "type": "military"},            ),            lx.data.Extraction(                extraction_class="theme",                extraction_text="battles alien creatures",                attributes={"theme_type": "conflict", "setting": "space"},            ),        ],    ),    lx.data.ExampleData(        text="A detective investigates supernatural murders in Victorian London. The horror thriller film combines period drama with paranormal elements.",        extractions=[            lx.data.Extraction(                extraction_class="genre",                extraction_text="horror thriller",                attributes={"primary_genre": "horror", "secondary_genre": "thriller"},            ),            lx.data.Extraction(                extraction_class="character",                extraction_text="detective",                attributes={"role": "protagonist", "type": "detective"},            ),            lx.data.Extraction(                extraction_class="theme",                extraction_text="supernatural murders",                attributes={"theme_type": "investigation", "setting": "victorian"},            ),        ],    ),    lx.data.ExampleData(        text="Two friends embark on a road trip adventure across America. The comedy drama explores friendship and self-discovery through humorous situations.",        extractions=[            lx.data.Extraction(                extraction_class="genre",                extraction_text="comedy drama",                attributes={"primary_genre": "comedy", "secondary_genre": "drama"},            ),            lx.data.Extraction(                extraction_class="character",                extraction_text="two friends",                attributes={"role": "protagonist", "type": "friends"},            ),            lx.data.Extraction(                extraction_class="theme",                extraction_text="friendship and self-discovery",                attributes={"theme_type": "personal_growth", "setting": "america"},            ),        ],    ),]# Extract from each documentextraction_results = []for doc in sample_documents:    result = lx.extract(        text_or_documents=doc,        prompt_description=prompt,        examples=examples,        model_id="gemini-2.0-flash",    )    extraction_results.append(result)    print(f"Successfully extracted from document: {doc[:50]}...")print(f"Completed tag extraction, processed {len(extraction_results)} documents")
```
```

```
```
Successfully extracted from document: John McClane fights terrorists in a Los Angeles......Completed tag extraction, processed 10 documents
```
```

## 处理和向量化结果

现在我们需要处理提取结果并为每个文档生成向量嵌入。我们还将把提取的属性展平为单独的字段，以便在 Milvus 中搜索。

```
```
print("\n3. Processing extraction results and generating vectors...")processed_data = []for result in extraction_results:    # Generate vectors for documents    embedding_response = genai_client.models.embed_content(        model=EMBEDDING_MODEL,        contents=[result.text],        config=EmbedContentConfig(            task_type="RETRIEVAL_DOCUMENT",            output_dimensionality=EMBEDDING_DIM,        ),    )    embedding = embedding_response.embeddings[0].values    print(f"Successfully generated vector: {result.text[:30]}...")    # Initialize data structure, flatten attributes into separate fields    data_entry = {        "id": result.document_id or str(uuid.uuid4()),        "document_text": result.text,        "embedding": embedding,        # Initialize all possible fields with default values        "genre": "unknown",        "primary_genre": "unknown",        "secondary_genre": "unknown",        "character_role": "unknown",        "character_type": "unknown",        "theme_type": "unknown",        "theme_setting": "unknown",    }    # Process extraction results, flatten attributes    for extraction in result.extractions:        if extraction.extraction_class == "genre":            # Flatten genre attributes            data_entry["genre"] = extraction.extraction_text            attrs = extraction.attributes or {}            data_entry["primary_genre"] = attrs.get("primary_genre", "unknown")            data_entry["secondary_genre"] = attrs.get("secondary_genre", "unknown")        elif extraction.extraction_class == "character":            # Flatten character attributes (take first main character's attributes)            attrs = extraction.attributes or {}            if (                data_entry["character_role"] == "unknown"            ):  # Only take first character's attributes                data_entry["character_role"] = attrs.get("role", "unknown")                data_entry["character_type"] = attrs.get("type", "unknown")        elif extraction.extraction_class == "theme":            # Flatten theme attributes (take first main theme's attributes)            attrs = extraction.attributes or {}            if (                data_entry["theme_type"] == "unknown"            ):  # Only take first theme's attributes                data_entry["theme_type"] = attrs.get("theme_type", "unknown")                data_entry["theme_setting"] = attrs.get("setting", "unknown")    processed_data.append(data_entry)print(f"Completed data processing, ready to insert {len(processed_data)} records")
```
```

```
```
3. Processing extraction results and generating vectors...Successfully generated vector: John McClane fights terrorists......Completed data processing, ready to insert 10 records
```

将数据插入 Milvus
```

准备好处理后的数据后，让我们将其插入 Milvus 集合。这将使我们能够执行语义搜索和精确的元数据过滤。

```
```
print("\n4. Inserting data into Milvus...")if processed_data:    res = client.insert(collection_name=COLLECTION_NAME, data=processed_data)    print(f"Successfully inserted {len(processed_data)} documents into Milvus")    print(f"Insert result: {res}")else:    print("No data to insert")
```

```
4. Inserting data into Milvus...Successfully inserted 10 documents into MilvusInsert result: {'insert_count': 10, 'ids': ['doc_f8797155', 'doc_78c7e586', 'doc_fa3a3ab5', 'doc_64981815', 'doc_3ab18cb2', 'doc_1ea42b18', 'doc_f0779243', 'doc_386590b7', 'doc_3b3ae1ab', 'doc_851089d6']}
```

演示元数据过滤
```

将 LangExtract 与 Milvus 结合的主要优势之一是能够基于提取的元数据执行精确过滤。让我们通过一些过滤表达式搜索来演示这一点。

```
```
print("\n=== Filter Expression Search Examples ===")# Load collection into memory for queryingprint("Loading collection into memory...")client.load_collection(collection_name=COLLECTION_NAME)print("Collection loaded successfully")# Search for thriller moviesprint("\n1. Searching for thriller movies:")results = client.query(    collection_name=COLLECTION_NAME,    filter='secondary_genre == "thriller"',    output_fields=["document_text", "genre", "primary_genre", "secondary_genre"],    limit=5,)for result in results:    print(f"- {result['document_text'][:100]}...")    print(        f"  Genre: {result['genre']} ({result.get('primary_genre')}-{result.get('secondary_genre')})"    )# Search for movies with military charactersprint("\n2. Searching for movies with military characters:")results = client.query(    collection_name=COLLECTION_NAME,    filter='character_type == "military"',    output_fields=["document_text", "genre", "character_role", "character_type"],    limit=5,)for result in results:    print(f"- {result['document_text'][:100]}...")    print(f"  Genre: {result['genre']}")    print(        f"  Character: {result.get('character_role')} ({result.get('character_type')})"    )=== Filter Expression Search Examples ===Loading collection into memory...Collection loaded successfully1. Searching for thriller movies:- A brilliant scientist creates artificial intelligence that becomes self-aware. The sci-fi thriller e...  Genre: sci-fi thriller (sci-fi-thriller)- Two detectives investigate a series of mysterious murders in New York City. The crime thriller featu...  Genre: crime thriller (crime-thriller)- A detective investigates supernatural crimes in Victorian London. The horror thriller combines perio...  Genre: horror thriller (horror-thriller)- John McClane fights terrorists in a Los Angeles skyscraper during Christmas Eve. The action-packed t...  Genre: action-packed thriller (action-thriller)2. Searching for movies with military characters:- Space marines battle alien invaders on a distant planet. The action sci-fi movie features futuristic...  Genre: action sci-fi  Character: protagonist (military)
```

搜索结果准确匹配了 "惊悚片" 和 "军事角色" 的筛选条件。
```

## 结合语义搜索与元数据过滤

这种集成的真正威力来自于将语义向量搜索与精确的元数据过滤相结合。这使我们能够在应用基于提取属性的特定约束的同时找到语义相似的内容。

```
```
print("\n=== Semantic Search Examples ===")# 1. Search for action-related content + only thriller genreprint("\n1. Searching for action-related content + only thriller genre:")query_text = "action fight combat battle explosion"query_embedding_response = genai_client.models.embed_content(    model=EMBEDDING_MODEL,    contents=[query_text],    config=EmbedContentConfig(        task_type="RETRIEVAL_QUERY",        output_dimensionality=EMBEDDING_DIM,    ),)query_embedding = query_embedding_response.embeddings[0].valuesresults = client.search(    collection_name=COLLECTION_NAME,    data=[query_embedding],    anns_field="embedding",    limit=3,    filter='secondary_genre == "thriller"',    output_fields=["document_text", "genre", "primary_genre", "secondary_genre"],    search_params={"metric_type": "COSINE"},)if results:    for result in results[0]:        print(f"- Similarity: {result['distance']:.4f}")        print(f"  Text: {result['document_text'][:100]}...")        print(            f"  Genre: {result.get('genre')} ({result.get('primary_genre')}-{result.get('secondary_genre')})"        )# 2. Search for magic-related content + fantasy genre + conflict themeprint("\n2. Searching for magic-related content + fantasy genre + conflict theme:")query_text = "magic wizard spell fantasy magical"query_embedding_response = genai_client.models.embed_content(    model=EMBEDDING_MODEL,    contents=[query_text],    config=EmbedContentConfig(        task_type="RETRIEVAL_QUERY",        output_dimensionality=EMBEDDING_DIM,    ),)query_embedding = query_embedding_response.embeddings[0].valuesresults = client.search(    collection_name=COLLECTION_NAME,    data=[query_embedding],    anns_field="embedding",    limit=3,    filter='primary_genre == "fantasy" and theme_type == "conflict"',    output_fields=[        "document_text",        "genre",        "primary_genre",        "theme_type",        "theme_setting",    ],    search_params={"metric_type": "COSINE"},)if results:    for result in results[0]:        print(f"- Similarity: {result['distance']:.4f}")        print(f"  Text: {result['document_text'][:100]}...")        print(f"  Genre: {result.get('genre')} ({result.get('primary_genre')})")        print(f"  Theme: {result.get('theme_type')} ({result.get('theme_setting')})")print("\n=== Demo Complete ===")
```

```
=== Semantic Search Examples ===1. Searching for action-related content + only thriller genre:- Similarity: 0.6947  Text: John McClane fights terrorists in a Los Angeles skyscraper during Christmas Eve. The action-packed t...  Genre: action-packed thriller (action-thriller)- Similarity: 0.6128  Text: Two detectives investigate a series of mysterious murders in New York City. The crime thriller featu...  Genre: crime thriller (crime-thriller)- Similarity: 0.5889  Text: A brilliant scientist creates artificial intelligence that becomes self-aware. The sci-fi thriller e...  Genre: sci-fi thriller (sci-fi-thriller)2. Searching for magic-related content + fantasy genre + conflict theme:- Similarity: 0.6986  Text: An evil sorcerer threatens to destroy the magical kingdom. A brave hero must gather allies and maste...  Genre: fantasy (fantasy)  Theme: conflict (fantasy_world)=== Demo Complete ===
```

可以看到，使用Milvus的语义搜索结果既符合类型筛选条件，也与查询文本内容高度相关。
```

# 尾声

LangExtract 与 Milvus 集成，核心优势是充分释放非结构化数据价值、提升方案落地可靠性，尤其适合金融、医疗等场景。

按此思路，做好图片结构化信息挖掘并结合语义检索，能在电商推荐中为用户提供更精准的商品服务；

进一步做好视频结构化信息挖掘与语义检索，可在自动驾驶数据挖掘中发挥更大作用。

在此基础上，高效管理、低成本存储自动驾驶数据，还需依赖 Milvus 即将推出的向量数据湖方案 —— 该方案能为冷数据提供更低成本的存储，更好支持宽表等需求，且能更优结合多模态数据的 ETL 处理。

围绕这一思路，还有哪些实用新想法？大家对 Milvus 向量数据湖又有哪些期待？欢迎在评论区交流！

**作者介绍**

张晨

Zilliz Algorithm Engineer

推荐阅读

[Manus、LangChain是如何做](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247509923&idx=1&sn=4701e4c77e20b82489ba4628cc6b9e1f&scene=21#wechat_redirect)[context  engineering以及Multi Agent的？](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247509923&idx=1&sn=4701e4c77e20b82489ba4628cc6b9e1f&scene=21#wechat_redirect)

[Word2Vec、 BERT、BGE-M3、LLM2Vec，embedding模型选型指南｜最全](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510057&idx=1&sn=58fdfef27956475e9eb974e3bd822859&scene=21#wechat_redirect)

[LLM、RAG、workflow、Agent，大模型落地该选哪个？一个决策矩阵讲透](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510010&idx=1&sn=55f15daab9e2eb4de4f274a9fe0d8abe&scene=21#wechat_redirect)

[n8n部署RAG太麻烦？MCP+自然语言搞定n8n workflow 的时代来了！](https://mp.weixin.qq.com/s?__biz=MzUzMDI5OTA5NQ==&mid=2247510045&idx=1&sn=2e6a7faf45640fd1afeef100bb680089&scene=21#wechat_redirect)