> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020303_RAG/020303_核心知识点/RAG生命周期与评估边界|RAG生命周期与评估边界]]
---
title: LanceDB: 多语言 SDK 与生态集成——TypeScript、Rust 和 AI 框架
author: MaxAiDB
date: maxaidbmaxaidb
url: https://mp.weixin.qq.com/s?__biz=MzUwOTU4OTU2NQ==&mid=2247483770&idx=1&sn=38ff9a9e1f229d6d2b0f9f594144fdb2&chksm=f8ab5311aa04564838403fefb276d690251108d2ff64e90274e4c177022a615354651681bc11&mpshare=1&scene=24&srcid=0601XmRbzGS7FJMf6ESzWSeN&sharer_shareinfo=13ce6f8309f5da2eb79977e37946ea06&sharer_shareinfo_first=13ce6f8309f5da2eb79977e37946ea06#rd
---

> **《LanceDB：从上手到内核》系列 · 第 3 篇**
>
> **分析基准**：LanceDB Python SDK `0.30.2` + Lance crate `6.0.0-beta.7` + `@lancedb/lancedb` (npm)

---

## 一句话速读

* • **三个成熟 SDK**：Rust（原生）、Python（PyO3）、TypeScript（napi-rs）—— 都是编译时绑定，非 FFI，非 REST
* • **Java SDK** 目前仅提供 Namespace 客户端，**没有 Table 操作 API**
* • **Embedding** 内置 18+ 实现（文本 / 多模态 / 本地 / 云端）
* • **Reranker** 内置 **10 种**（RRF / CrossEncoder / Cohere / Jina / VoyageAI / OpenAI / Colbert / AnswerDotAi / LinearCombination / MRR）
* • **AI 框架**：LangChain、LlamaIndex 官方 VectorStore 集成
* • **开放格式**：Lance 文件可被 DuckDB / Ray / Polars 直接读

---

## 一个数据库，三个团队

后端 Node.js、数据科学 Python、基础设施 Rust——一个向量数据库同时服务三个团队？LanceDB 做到了，而且不是用 REST 这种"最大公约数"方式，每个 SDK 都是**原生绑定**。

---

## SDK 成熟度一览

| SDK | 成熟度 | 绑定方式 | 覆盖范围 | 源码 |
| --- | --- | --- | --- | --- |
| **Rust** | 最完整 | 原生 | 全部功能 | `rust/lancedb/src/` |
| **Python** | 成熟 | PyO3 + maturin | 同步 + 异步全覆盖 | `python/python/lancedb/` |
| **TypeScript** | 成熟 | napi-rs | 全部功能 | `nodejs/lancedb/` |
| **Java** | 早期 | JNI | 仅 Namespace 连接，无 Table API | `java/lancedb-core/` |

绑定方式的意义：**零拷贝**传 Arrow Buffer 给 Rust 核心，不走序列化、不走 RPC、不走 localhost TCP。Python 和 TS 的搜索性能开销几乎只在语言边界的一次调用上。

> **Java 团队的过渡方案**：在 Table API 就绪前，可以用 Python/TS 做 sidecar 服务，或者等 Namespace 模式 + REST 服务端执行（`server_side_query`）配置齐全。

---

## TypeScript SDK

npm 包名是 **`@lancedb/lancedb`**（不是 `lancedb`）：

```
import * as lancedb from "@lancedb/lancedb";

const db = await lancedb.connect("/tmp/lancedb_ts_demo");

// 建表（数据结构和 Python 完全对等）
const data = Array.from({ length: 100 }, (_, i) => ({
  id: i,
  text: `document ${i}`,
  vector: Array.from({ length: 384 }, () => Math.random()),
}));
const table = await db.createTable("docs", data);

// 向量检索
const query = Array.from({ length: 384 }, () => Math.random());
const results = await table.search(query).limit(5).toArrow();

// 创建 IVF-PQ 索引
await table.createIndex("vector", {
  config: lancedb.Index.ivfPq({ numPartitions: 32, numSubVectors: 8 }),
});

// Tags / 版本 / 优化（与 Python 对等）
const version = await table.version();
const tagsManager = await table.tags();
await tagsManager.create("v1.0", version);
await table.optimize();
```

命名风格：`snake_case`（Python）→ `camelCase`（TS）。其他语义完全一致。

---

## Rust SDK

Rust 是核心实现，API 最完整：

```
use lancedb::connect;
use lancedb::index::Index;
use lancedb::index::vector::IvfPqIndexBuilder;

#[tokio::main]
async fn main() -> lancedb::Result<()> {
    let db = connect("/tmp/lancedb_rust_demo").execute().await?;
    let table = db.open_table("docs").execute().await?;

    // 建 IVF-PQ 索引
    table.create_index(
        &["vector"],
        Index::IvfPq(IvfPqIndexBuilder::default().num_partitions(32))
    ).execute().await?;

    Ok(())
}
```

所有表操作抽象在 `BaseTable` trait（`rust/lancedb/src/table.rs`），`NativeTable` 是本地/对象存储实现，`RemoteTable` 是 Cloud REST 实现。

---

## Embedding 生态（18+ 提供商）

### 文本 Embedding

| 实现 | 类名 | 特点 |
| --- | --- | --- |
| OpenAI | `OpenAIEmbeddings` | text-embedding-3-small/large |
| Cohere | `CohereEmbeddingFunction` | 多语言 |
| Sentence Transformers | `SentenceTransformerEmbeddings` | 本地运行，无需 API Key |
| Ollama | `OllamaEmbeddings` | 本地 Ollama 服务 |
| Alibaba GTE | `GteEmbeddings` | 中文优化 |
| HuggingFace | `TransformersEmbeddingFunction` | 任意 HF 模型 |
| Jina | `JinaEmbeddings` | 多语言、多模态 |
| VoyageAI | `VoyageAIEmbeddingFunction` | 高性能文本 |
| Google Gemini | `GeminiText` | Google 生态 |
| AWS Bedrock | `BedRockText` | AWS 生态 |
| Watsonx | `WatsonxEmbeddings` | IBM 生态 |
| Instructor | `InstructorEmbeddingFunction` | 指令可控 |

### 多模态 Embedding

| 实现 | 类名 | 特点 |
| --- | --- | --- |
| OpenCLIP | `OpenClipEmbeddings` | 图像 + 文本跨模态 |
| ImageBind | `ImageBindEmbeddings` | 图像 / 音频 / 文本统一空间 |
| ColPali | `ColPaliEmbeddings` | PDF / 文档页面多向量检索 |
| SigLip | `SigLipEmbeddings` | Sigmoid Loss CLIP 变体 |

（均位于 `python/python/lancedb/embeddings/`，查 `@register("<name>")` 找对应实现。）

### 自定义 Embedding（10 行注册一个）

```
import numpy as np
from lancedb.embeddings.base import TextEmbeddingFunction
from lancedb.embeddings.registry import register, get_registry

@register("my-model")
class MyEmbedding(TextEmbeddingFunction):
    def ndims(self): return 256
    def generate_embeddings(self, texts):
        # 换成真实模型推理即可
        return [np.random.rand(256).tolist() for _ in texts]

emb = get_registry().get("my-model").create()
```

之后和内置实现一样使用：`emb.SourceField()` / `emb.VectorField()`。

---

## 10 种内置 Reranker

| Reranker | 需外部 API | 推荐场景 | 类名 |
| --- | --- | --- | --- |
| **RRF** （默认） | ❌ | 零成本零延迟，Hybrid 首选 | `RRFReranker` |
| CrossEncoder | ❌ | 本地精排，质量高 | `CrossEncoderReranker` |
| Colbert | ❌ | 晚期交互，多向量 | `ColbertReranker` |
| AnswerDotAi | ❌ | answerdotai/rerankers 的本地实现 | `AnswerdotaiRerankers` |
| LinearCombination | ❌ | 向量分 × α + FTS 分 × (1-α) | `LinearCombinationReranker` |
| MRR | ❌ | Maximal Relevance Ranker | `MRRReranker` |
| Cohere | ✅ | Cohere 商业 API | `CohereReranker` |
| OpenAI | ✅ | OpenAI 精排 | `OpenaiReranker` |
| Jina | ✅ | Jina API | `JinaReranker` |
| VoyageAI | ✅ | VoyageAI API | `VoyageAIReranker` |

**推荐升级路径**：`RRF`（兜底）→ `CrossEncoder` / `Colbert`（本地精排）→ 云 API（最后一公里）。

> **注意**：Rust 内核目前只实现了 `RRF`（`rust/lancedb/src/rerankers/rrf.rs`）；其他 9 种 Reranker 都在 Python 层（`python/python/lancedb/rerankers/`）。TypeScript 目前也只导出了 RRF。

---

## AI 框架集成

### LangChain

```
from langchain_community.vectorstores import LanceDB
vectorstore = LanceDB(
    connection=db,               # 传入已有的 lancedb.connect() 返回对象
    embedding=embeddings,
    table_name="docs",
)
retriever = vectorstore.as_retriever(search_kwargs={"k": 5})
```

### LlamaIndex

```
from llama_index.vector_stores.lancedb import LanceDBVectorStore
vector_store = LanceDBVectorStore(
    uri="/tmp/lancedb_llamaindex",
    table_name="docs",
)
```

两个集成都在各自生态的**独立 PyPI 包**里发布（`langchain-community` / `llama-index-vector-stores-lancedb`），不随 `lancedb` 主包安装。

### DuckDB 互操作

Lance 格式是**开放**的——数据不被锁在 LanceDB 里。DuckDB 的 `lance` extension 可以直接读：

```
INSTALL lance;
LOAD lance;
SELECT COUNT(*) FROM '/path/to/table.lance';
```

这意味着已有的 BI / 数据分析管线（DuckDB / Polars / Ray Data）可以和 LanceDB 无缝协作——LanceDB 写入，其他工具直接读。

---

## 小结

1. 1. **三个成熟 SDK + 一个早期 SDK**：Rust、Python、TypeScript 完整；Java 仅 Namespace
2. 2. **18+ Embedding 实现** 覆盖主流文本 / 多模态 / 本地 / 云
3. 3. **10 种 Reranker**，Rust 核目前只实现 RRF，其他在 Python 层
4. 4. **LangChain / LlamaIndex** 在独立包中发布
5. 5. **开放格式**：DuckDB / Polars / Ray 可以直接读 Lance 文件

---

## 延伸阅读

* • **TS SDK 入口**：`nodejs/lancedb/connection.ts`、`nodejs/lancedb/table.ts`
* • **TS 索引构造**：`nodejs/lancedb/indices.ts`（`IvfPqOptions`、`Index.ivfPq(...)`）
* • **Rust SDK 核心 trait**：`rust/lancedb/src/table.rs`（`BaseTable` trait）
* • **Embedding 注册表**：`python/python/lancedb/embeddings/registry.py`
* • **Reranker 基类**：`python/python/lancedb/rerankers/base.py`
* • **可复现验证**：`verified-examples/03/test_sdk_ecosystem.py`
* • **下一篇**：第 4 篇——整体架构全景图