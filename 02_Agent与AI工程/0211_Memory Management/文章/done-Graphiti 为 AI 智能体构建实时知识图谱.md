> 已吸收至：[[02_Agent与AI工程/0211_Memory Management/0211_核心知识点/记忆管理分层与注入边界|记忆管理分层与注入边界]]
---
title: Graphiti 为 AI 智能体构建实时知识图谱
author: 山行AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzU2NzkxNDY0Ng==&mid=2247489229&idx=1&sn=30c5f4c83d8455139810d3be0e7a0077&chksm=fd621c102f0c0df2b494d5e78e40b86ebf2d20e8cc46c9806e703508a8cd7f75953134e56856&mpshare=1&scene=24&srcid=1226ZcTPJrOysrM6CzHU716R&sharer_shareinfo=de0e2a60c6902d6c68a15abbb4b953d2&sharer_shareinfo_first=de0e2a60c6902d6c68a15abbb4b953d2#rd
---

### 💡查看Graphiti 的新 MCP 服务器[1]Claude、Cursor 和其他 MCP 客户端提供强大的基于知识图谱的记忆功能。

---

### 什么是 Graphiti？

Graphiti 是一个框架，用于构建和查询具有时间感知的知识图谱，特别适用于在动态环境中操作的 AI 智能体。与传统的检索增强生成（RAG）方法不同，Graphiti 能够持续地将用户互动、结构化与非结构化的企业数据以及外部信息集成到一个连贯的、可查询的图谱中。该框架支持增量数据更新、高效检索，并且可以精确地进行历史查询，而无需重新计算整个图谱，因此非常适合开发交互式、具有上下文感知的 AI 应用程序。

使用 Graphiti 可以：

•集成并维护动态的用户互动和业务数据。•促进基于状态的推理和任务自动化。•使用语义、关键词和基于图谱的搜索方法查询复杂的、不断发展的数据。

知识图谱是由一系列相互连接的事实组成的网络，例如 “Kendra 喜欢 Adidas 鞋子”。每个事实是一个由两个实体（或节点）和它们之间的关系（或边）组成的“三元组”（例如，“Kendra”，“Adidas 鞋子”和“喜欢”）。知识图谱在信息检索中被广泛应用。Graphiti 的独特之处在于，它能够自主构建知识图谱，同时处理关系的变化并维护历史上下文。

### Graphiti 与 Zep 的上下文工程平台

Graphiti 为 Zep 的核心提供支持，Zep 是一个为 AI 智能体提供开箱即用的上下文工程平台。Zep 提供智能体记忆、图谱 RAG（用于动态数据）以及上下文检索和组装功能。

我们很高兴开源 Graphiti，认为它的潜力远远超出了 AI 记忆应用的范畴。

## Zep vs Graphiti

| **方面** | **Zep** | **Graphiti** |
| --- | --- | --- |
| **它们是什么** | 完全托管的平台，用于上下文工程和 AI 记忆 | 开源图谱框架 |
| **用户与会话管理** | 内置用户、线程和消息存储 | 自定义实现 |
| **检索与性能** | 预配置的、生产就绪的检索，性能可达到200ms以下，适合大规模使用 | 需要自定义实现；性能取决于您的设置 |
| **开发者工具** | 提供图谱可视化的仪表盘、调试日志、API日志；提供 Python、TypeScript 和 Go 的 SDK | 需要自定义工具 |
| **企业功能** | 服务级别协议（SLA）、支持、安全保障 | 自行管理 |
| **部署方式** | 完全托管或在您的云中部署 | 仅支持自托管 |

### 选择哪个？

•**选择 Zep**：如果您需要一个开箱即用的企业级平台，具备安全性、性能和支持。•**选择 Graphiti**：如果您需要一个灵活的开源核心，并且愿意构建/运营周围的系统。

---

### 为什么选择 Graphiti？

传统的 RAG 方法通常依赖于批量处理和静态数据汇总，这使得它们在处理频繁变化的数据时效率较低。Graphiti 通过以下方式解决了这些问题：

•**实时增量更新**：即时集成新的数据事件，而无需批量重新计算。•**双时间数据模型**：明确跟踪事件发生和数据摄取的时间，允许精确的时间点查询。•**高效的混合检索**：结合语义嵌入、关键词（BM25）和图遍历，实现低延迟查询，无需依赖大语言模型（LLM）摘要。•**自定义实体定义**：通过简单的 Pydantic 模型，支持灵活的本体创建和开发者定义实体。•**可扩展性**：高效地管理大型数据集，支持并行处理，适合企业级环境。

## Graphiti vs. GraphRAG

| **方面** | **GraphRAG** | **Graphiti** |
| --- | --- | --- |
| **主要用途** | 静态文档总结 | 动态数据管理 |
| **数据处理** | 批量处理 | 持续增量更新 |
| **知识结构** | 实体簇和社区总结 | 事件数据、语义实体、社区 |
| **检索方法** | 顺序的大语言模型（LLM）摘要 | 混合语义、关键词和基于图谱的搜索方法 |
| **适应性** | 低 | 高 |
| **时间处理** | 基本的时间戳追踪 | 明确的双时间追踪 |
| **矛盾处理** | 基于 LLM 的摘要判断 | 时间边缘失效 |
| **查询延迟** | 几秒到几十秒 | 通常为亚秒级延迟 |
| **自定义实体类型** | 否 | 是，可自定义 |
| **可扩展性** | 中等 | 高，针对大数据集进行了优化 |

### Graphiti 与 GraphRAG 的区别

Graphiti 专门设计用来解决动态和频繁更新的数据集问题，特别适用于需要实时交互和精确历史查询的应用。

---

## 安装

### 要求：

•Python 3.10 或更高版本•Neo4j 5.26 / FalkorDB 1.1.2 / Kuzu 0.11.2 / Amazon Neptune 数据库集群或 Neptune Analytics 图 + Amazon OpenSearch Serverless 集合（作为全文搜索后台）•OpenAI API 密钥（Graphiti 默认使用 OpenAI 进行 LLM 推理和嵌入）

### 重要说明：

Graphiti 最适合与支持结构化输出的 LLM 服务一起使用（例如 OpenAI 和 Gemini）。如果使用其他服务，可能会导致输出模式不正确或数据摄取失败。尤其是在使用较小的模型时，这个问题会更明显。

### 可选项：

•Google Gemini、Anthropic 或 Groq API 密钥（用于替代的 LLM 提供商）

### 提示：

安装 Neo4j 最简单的方式是通过 **Neo4j Desktop**。它提供了一个用户友好的界面来管理 Neo4j 实例和数据库。或者，您可以通过 Docker 本地使用 FalkorDB，并使用快速入门示例进行快速启动：

```
docker run -p 6379:6379-p 3000:3000-it --rm falkordb/falkordb:latestpip install graphiti-core
```

或使用以下命令：

```
uv add graphiti-core
```

### 安装 FalkorDB 支持

如果您计划使用 **FalkorDB** 作为图谱数据库后台，请使用 FalkorDB 扩展进行安装：

```
pip install graphiti-core[falkordb]或使用 uv 安装：
uv add graphiti-core[falkordb]
```

### 安装 Kuzu 支持

如果您计划使用 Kuzu 作为图谱数据库后台，请使用 Kuzu 扩展进行安装：

```
pip install graphiti-core[kuzu]或使用 uv 安装：
uv add graphiti-core[kuzu]
```

### 安装 Amazon Neptune 支持

如果您计划使用 Amazon Neptune 作为图谱数据库后台，请使用 Amazon Neptune 扩展进行安装：

```
pip install graphiti-core[neptune]或使用 uv 安装：
uv add graphiti-core[neptune]
```

### 安装可选的 LLM 提供商

您还可以选择安装可选的 LLM 提供商作为扩展：

```
安装Anthropic支持：pip install graphiti-core[anthropic]
安装Groq支持：pip install graphiti-core[groq]
安装GoogleGemini支持：pip install graphiti-core[google-genai]
安装多个 LLM 提供商：pip install graphiti-core[anthropic,groq,google-genai]
安装FalkorDB和 LLM 提供商：pip install graphiti-core[falkordb,anthropic,google-genai]
安装AmazonNeptune支持：pip install graphiti-core[neptune]
```

## 默认低并发设置；LLM 提供商 429 限速错误

Graphiti 的数据摄取管道是为高并发设计的。默认情况下，并发量设置为较低，以避免 LLM 提供商出现 429 限速错误。如果您发现 Graphiti 性能较慢，可以按照下面的说明增加并发量。

并发量由 `SEMAPHORE_LIMIT` 环境变量控制。默认情况下，`SEMAPHORE_LIMIT` 设置为 10 并发操作，以帮助防止 LLM 提供商返回 429 限速错误。如果您遇到此类错误，可以尝试降低该值。

如果您的 LLM 提供商允许更高的吞吐量，您可以增加 `SEMAPHORE_LIMIT` 来提升数据摄取性能。

---

## 快速开始

### 重要说明

Graphiti 默认使用 OpenAI 进行 LLM 推理和嵌入。确保在您的环境中设置了 `OPENAI_API_KEY`。也可以支持 Anthropic 和 Groq 的 LLM 推理。其他 LLM 提供商可能通过 OpenAI 兼容的 API 进行支持。

### 完整工作示例

有关完整的工作示例，请参阅 `examples` 目录中的快速入门示例。该示例演示了以下内容：

•连接到 Neo4j、Amazon Neptune、FalkorDB 或 Kuzu 数据库•初始化 Graphiti 索引和约束•向图谱中添加数据事件（包括文本和结构化 JSON）•使用混合搜索方法搜索关系（边）•使用图距离对搜索结果进行重新排序•使用预定义的搜索配方搜索节点

该示例有完整的文档，清晰地解释了每个功能，并包含了详细的 README 文件，提供了设置说明和后续步骤。

---

### 使用 Docker Compose 运行

您可以使用 Docker Compose 快速启动所需的服务：

•Neo4j Docker：

```
docker compose up
```

此命令将启动 Neo4j Docker 服务及相关组件。

•FalkorDB Docker：

```
docker compose --profile falkordb up
```

此命令将启动 FalkorDB Docker 服务及相关组件。

## MCP 服务器

`mcp_server` 目录包含了 Graphiti 的 **模型上下文协议**（MCP）服务器实现。该服务器允许 AI 助手通过 MCP 协议与 Graphiti 的知识图谱功能进行交互。

### MCP 服务器的主要功能：

•数据事件管理（添加、检索、删除）•实体管理和关系处理•语义和混合搜索功能•数据分组管理，用于组织相关数据•图谱维护操作

MCP 服务器可以使用 Docker 部署并与 Neo4j 集成，使 Graphiti 容易集成到您的 AI 助手工作流中。

有关详细的设置说明和使用示例，请参阅MCP 服务器的 README 文件[2]

## REST 服务

`server` 目录包含了与 Graphiti API 交互的 API 服务，使用 **FastAPI** 构建。

更多信息请参考server[3]中的 README 文件。

---

## 可选环境变量

除了 Neo4j 和 OpenAi 兼容的凭证外，Graphiti 还提供了几个可选的环境变量。如果您使用的是我们的支持模型之一（例如 Anthropic 或 Voyage 模型），则必须设置相关的环境变量。

### 数据库配置

数据库名称在驱动程序构造器中直接配置：

•**Neo4j**：数据库名称默认为 `neo4j`（在 Neo4jDriver 中硬编码）•**FalkorDB**：数据库名称默认为 `default_db`（在 FalkorDriver 中硬编码）

从 v0.17.0 版本开始，如果您需要自定义数据库配置，可以实例化一个数据库驱动程序，并通过 `graph_driver` 参数将其传递给 Graphiti 构造函数。

####

#### Neo4j 自定义数据库名称

```
from graphiti_core importGraphitifrom graphiti_core.driver.neo4j_driver importNeo4jDriver
# 创建一个带有自定义数据库名称的 Neo4j 驱动driver =Neo4jDriver(    uri="bolt://localhost:7687",    user="neo4j",    password="password",    database="my_custom_database"# 自定义数据库名称)
# 将驱动传递给 Graphitigraphiti =Graphiti(graph_driver=driver)
```

#### FalkorDB 自定义数据库名称

```
from graphiti_core importGraphitifrom graphiti_core.driver.falkordb_driver importFalkorDriver
# 创建一个带有自定义数据库名称的 FalkorDB 驱动driver =FalkorDriver(    host="localhost",    port=6379,    username="falkor_user",# 可选    password="falkor_password",# 可选    database="my_custom_graph"# 自定义数据库名称)
# 将驱动传递给 Graphitigraphiti =Graphiti(graph_driver=driver)
```

#### Kuzu 驱动

```
from graphiti_core importGraphitifrom graphiti_core.driver.kuzu_driver importKuzuDriver
# 创建一个 Kuzu 驱动driver =KuzuDriver(db="/tmp/graphiti.kuzu")
# 将驱动传递给 Graphitigraphiti =Graphiti(graph_driver=driver)
```

#### Amazon Neptune 驱动

```
from graphiti_core importGraphitifrom graphiti_core.driver.neptune_driver importNeptuneDriver
# 创建一个 Amazon Neptune 驱动driver =NeptuneDriver(    host="<NEPTUNE_ENDPOINT>",    aoss_host="<Amazon_OpenSearch_Serverless_Host>",    port="<PORT>",# 可选，默认为 8182    aoss_port="<PORT>"# 可选，默认为 443)
# 将驱动传递给 Graphitigraphiti =Graphiti(graph_driver=driver)
```

## 使用 Graphiti 与 Azure OpenAI

Graphiti 支持 Azure OpenAI，用于 LLM 推理和嵌入，兼容 Azure 的 OpenAI v1 API。

### 快速开始

```
from openai importAsyncOpenAIfrom graphiti_core importGraphitifrom graphiti_core.llm_client.azure_openai_client importAzureOpenAILLMClientfrom graphiti_core.llm_client.config importLLMConfigfrom graphiti_core.embedder.azure_openai importAzureOpenAIEmbedderClient
# 使用标准的 OpenAI 客户端初始化 Azure OpenAI 客户端# 并使用 Azure 的 v1 API 端点azure_client =AsyncOpenAI(    base_url="https://your-resource-name.openai.azure.com/openai/v1/",    api_key="your-api-key",)
# 创建 LLM 和 Embedder 客户端llm_client =AzureOpenAILLMClient(    azure_client=azure_client,    config=LLMConfig(model="gpt-5-mini", small_model="gpt-5-mini")# 你的 Azure 部署名称)embedder_client =AzureOpenAIEmbedderClient(    azure_client=azure_client,    model="text-embedding-3-small"# 你的 Azure 嵌入部署名称)
# 使用 Azure OpenAI 客户端初始化 Graphitigraphiti =Graphiti("bolt://localhost:7687","neo4j","password",    llm_client=llm_client,    embedder=embedder_client,)
# 现在您可以使用 Graphiti 与 Azure OpenAI 进行交互
```

> 重点说明：
>
> •用标准的 `AsyncOpenAI` 客户端，并使用 Azure 的 v1 API 端点格式：`https://your-resource-name.openai.azure.com/openai/v1/`•部署名称（例如：`gpt-5-mini`，`text-embedding-3-small`）应与您的 Azure OpenAI 部署名称一致。•请参阅 `examples/azure-openai/` 获取完整的工作示例。•将占位符值替换为实际的 Azure OpenAI 凭证和部署名称。

---

## 使用 Graphiti 与 Google Gemini

Graphiti 支持 Google 的 Gemini 模型进行 LLM 推理、嵌入和跨编码/重新排序。要使用 Gemini，您需要配置 LLM 客户端、嵌入器和跨编码器，并提供您的 Google API 密钥。

安装 Graphiti：

```
uv add "graphiti-core[google-genai]"或
pip install "graphiti-core[google-genai]"
```

```
from graphiti_core importGraphitifrom graphiti_core.llm_client.gemini_client importGeminiClient,LLMConfigfrom graphiti_core.embedder.gemini importGeminiEmbedder,GeminiEmbedderConfigfrom graphiti_core.cross_encoder.gemini_reranker_client importGeminiRerankerClient
# Google API 密钥配置api_key ="<your-google-api-key>"
# 使用 Gemini 客户端初始化 Graphitigraphiti =Graphiti("bolt://localhost:7687","neo4j","password",    llm_client=GeminiClient(        config=LLMConfig(            api_key=api_key,            model="gemini-2.0-flash")),    embedder=GeminiEmbedder(        config=GeminiEmbedderConfig(            api_key=api_key,            embedding_model="embedding-001")),    cross_encoder=GeminiRerankerClient(        config=LLMConfig(            api_key=api_key,            model="gemini-2.5-flash-lite")))
# 现在您可以使用 Graphiti 与 Google Gemini 的所有组件
```

Gemini 重新排序器默认使用`gemini-2.5-flash-lite`模型，该模型经过优化，适用于成本效益高且延迟低的分类任务。它采用与 OpenAI 重新排序器相同的布尔分类方法，利用 Gemini 的对数概率功能对段落相关性进行排名。

## 使用 Graphiti 与 Ollama（本地 LLM）

Graphiti 支持 Ollama，通过 Ollama 的 OpenAI 兼容 API 运行本地 LLM 和嵌入模型。这对于注重隐私的应用程序或希望避免 API 成本的场景非常适合。

**注意**：对于 Ollama 和其他 OpenAI 兼容的提供商（如 LM Studio），请使用 `OpenAIGenericClient`（而不是 `OpenAIClient`）。`OpenAIGenericClient` 针对本地模型进行了优化，具有更高的默认最大 token 限制（16K vs 8K），并完全支持结构化输出。

安装模型：

```
ollama pull deepseek-r1:7b# LLMollama pull nomic-embed-text  # 嵌入模型
```

```
from graphiti_core importGraphitifrom graphiti_core.llm_client.config importLLMConfigfrom graphiti_core.llm_client.openai_generic_client importOpenAIGenericClientfrom graphiti_core.embedder.openai importOpenAIEmbedder,OpenAIEmbedderConfigfrom graphiti_core.cross_encoder.openai_reranker_client importOpenAIRerankerClient
# 配置 Ollama LLM 客户端llm_config =LLMConfig(    api_key="ollama",# Ollama 不需要真正的 API 密钥，但需要一个占位符    model="deepseek-r1:7b",    small_model="deepseek-r1:7b",    base_url="http://localhost:11434/v1",# Ollama 的 OpenAI 兼容端点)
llm_client =OpenAIGenericClient(config=llm_config)
# 使用 Ollama 客户端初始化 Graphitigraphiti =Graphiti("bolt://localhost:7687","neo4j","password",    llm_client=llm_client,    embedder=OpenAIEmbedder(        config=OpenAIEmbedderConfig(            api_key="ollama",# 占位符 API 密钥            embedding_model="nomic-embed-text",            embedding_dim=768,            base_url="http://localhost:11434/v1",)),    cross_encoder=OpenAIRerankerClient(client=llm_client, config=llm_config),)
# 现在您可以使用本地的 Ollama 模型与 Graphiti 进行交互
```

运行以下命令启动 Ollama 服务，并确保已经拉取了您想要使用的模型：`ollama serve`

```
https://github.com/getzep/graphiti
```

### References

`[1]` Graphiti 的新 MCP 服务器: *https://github.com/getzep/graphiti/blob/main/mcp\_server/README.md*
`[2]` MCP 服务器的 README 文件: *https://github.com/getzep/graphiti/blob/main/mcp\_server/README.md*
`[3]` server: *https://github.com/getzep/graphiti/blob/main/server/README.md*