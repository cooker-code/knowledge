> 已吸收至：[[04_OLAP与数据库/0402_关系数据库/040202_PostgreSQL/040202_核心知识点/PostgreSQLpgvector与多模态检索边界|PostgreSQLpgvector与多模态检索边界]]
---
title: pgvector 非权威指南
author: 老胡闲话
date:
url: https://mp.weixin.qq.com/s?__biz=MzIyOTY1NDYyMw==&mid=2247485590&idx=1&sn=2407e50ebc9946739ca64d8b5df0a7af&chksm=e9825a7bf91d617395fbc694c9616ab8a2220aca266a726769440b6a7d09c3b52bfd2ff04607&mpshare=1&scene=24&srcid=1226StYsr0AZesJJEDciM29d&sharer_shareinfo=e1835fc30de5a342b4a358db8e330bb3&sharer_shareinfo_first=e1835fc30de5a342b4a358db8e330bb3#rd
---

在构建 RAG（检索增强生成）应用时，向量数据库是不可或缺的组件。PostgreSQL 的 pgvector 扩展为我们提供了一个强大而熟悉的选择。本文将全面介绍 pgvector 的使用，从基础概念到生产环境的最佳实践。

## 基础概念：字符串、向量与嵌入

在 RAG 的上下文中，**向量**是字符串语义含义的数值化表示。例如：

```
ounter(line"cat" → [0.2, 0.8, 0.1, ...]
```

\*\*嵌入（Embedding）\*\*是将字符串转换为向量的过程。通过嵌入模型，我们可以将文本转换为机器可以理解和比较的数值形式。

### 为什么向量很重要？

传统的文本比较依赖于字符串匹配，而向量比较则考虑了语义含义。这使得向量搜索能够：

* **语义搜索**：找到意义相似的文档，而非仅仅是关键词匹配
* **RAG 应用**：为大语言模型检索相关上下文
* **推荐系统**：发现相似的产品或内容
* **分类聚类**：按语义相似度对内容进行分组
* **去重**：识别近似重复的内容

## 向量维度

向量维度指的是向量数组的长度。不同的嵌入模型提供商有各自的默认维度：

```
ounter(lineounter(lineOpenAI text-embedding-3-small  → 1536 维OpenAI text-embedding-3-large  → 3072 维
```

### 维度越高越好吗？

更高的维度有其优缺点：

**优点：**

* ✅ 更细腻的语义关系表达
* ✅ 复杂任务的准确度更高

**缺点：**

* ❌ 存储空间更大（3072 个浮点数 vs 1536 个）
* ❌ 查询速度更慢
* ❌ API 调用成本更高

**最佳实践**：使用满足精度需求的最小维度。

### 维度截断

当 LLM 返回的维度超过数据库支持时，可以使用 API 参数进行截断：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(line// OpenAI text-embedding-3-large 返回 3072 维const fullEmbedding = await openai.embeddings.create({  model: "text-embedding-3-large",  input: "your text",  dimensions: 1536  // ✅ 请求截断版本});
```

OpenAI 支持通过 API 参数动态截断维度。

## 向量距离

距离度量用于衡量两个向量的相似程度。在 pgvector 中，距离越小表示向量越相似。

```
-- 通过 L2 距离获取最近邻SELECT * FROM items ORDER BY embedding <-> '[3,1,2]' LIMIT 5;
```

### pgvector 中的距离类型

pgvector 提供了 6 种距离运算符：

| 运算符 | 类型 | 公式 | 常见用例 | 数据类型 |
| --- | --- | --- | --- | --- |
| `<->` | L2（欧几里得） |  | 图像识别；关注绝对幅度差异 | 连续（浮点） |
| `<#>` | （负）内积 |  | 推荐系统；同时关注方向和幅度 | 连续（浮点） |
| `<=>` | 余弦距离 |  | 文本相似度（NLP）和推荐系统；忽略幅度 | 连续（浮点） |
| `<+>` | L1（曼哈顿） |  | 所有维度权重相等的特征向量；对异常值不太敏感 | 连续（浮点） |
| `<~>` | 汉明距离 | 向量不同位数的计数 | 二进制特征比较、哈希匹配和错误检测 | 二进制向量 |
| `<%>` | Jaccard 距离 |  | 集合相似度；比较客户购买历史或分类数据 | 二进制向量 |

## pgvector 数据类型

pgvector 支持四种向量类型：

```
CREATE TABLE embeddings (  id serial PRIMARY KEY,  vec1 vector(1536),        -- 精确表示  vec2 halfvec(1536),       -- 半精度（存储空间减半）  vec3 bit(1536),           -- 二进制（存储空间减少 32 倍）  vec4 sparsevec(5)         -- 稀疏向量（仅存储非零值）);
```

**注意**：可以省略维度参数（例如 `vector` 而非 `vector(n)`）来在同一列中存储不同维度的向量。但是，索引需要固定维度。

### 向量类型权衡

| 类型 | 存储空间 | 精度 | 使用场景 |
| --- | --- | --- | --- |
| `vector` | 1536 × 4 字节 | 精确 | 生产环境默认 |
| `halfvec` | 1536 × 2 字节 | 约 0.1% 损失 | 大数据集 |
| `bit` | 1536 ÷ 8 字节 | 二进制 | 极大规模 |
| `sparsevec` | (NNZ × 8) + 16 字节 | 精确 | 大部分为零的值 |

NNZ = 非零元素数量

## 索引类型

pgvector 提供两种索引类型：

```
-- IVFFlat：适用于 < 100 万向量CREATE INDEX ON embeddingsUSING ivfflat (embedding vector_l2_ops);
-- HNSW：更适合 > 100 万向量CREATE INDEX ON embeddingsUSING hnsw (embedding vector_l2_ops);
```

### 索引类型比较

| 索引 | 构建时间 | 查询速度 | 内存使用 | 召回率 | 最适合 |
| --- | --- | --- | --- | --- | --- |
| 无索引 | - | 慢 | - | 100% | < 1 万行 |
| IVFFlat | 快 | 良好 | 低 | ~95% | 1 万 - 100 万行 |
| HNSW | 慢 | 快 | 高 | ~99% | > 100 万行 |

## 使用 Drizzle ORM

Drizzle 为 pgvector 提供了优秀的 TypeScript 支持。

### 定义列和索引

```
// CREATE INDEX ON items USING hnsw (embedding vector_l2_ops);
const table = pgTable('items', {    embedding: vector({ dimensions: 3 })}, (table) => [  index('l2_index').using('hnsw', table.embedding.op('vector_l2_ops'))])
```

### 插入嵌入向量

```
import { db } from './db';
await db.insert(documents).values({  content: 'PostgreSQL is a powerful database',  embedding, // [...]});
```

### 使用距离函数查询

Drizzle 提供了预定义的距离辅助函数：

`l2Distance`、`l1Distance`、`innerProduct`、`cosineDistance`、`hammingDistance`、`jaccardDistance`

```
db.select().from(items).orderBy(l2Distance(items.embedding, [3,1,2]))
```

## 常见陷阱与解决方案

### 陷阱 1：索引时的维度限制

* 索引只能在相同维度的向量上创建：`vector(n)` 而非 `vector`
* HNSW 和 IVFFlat 对每种向量类型都有最大维度限制：

| 类型 | HNSW | IVFFlat |
| --- | --- | --- |
| vector | 2000 | 2000 |
| halfvec | 4000 | 4000 |
| bit | 64000 | 64000 |
| sparsevec | 1000 NNZ | - |

**解决方案**：对于超过 2000 维的向量，使用 `halfvec` 或选择维度更小的嵌入模型。

### 陷阱 2：向量归一化

在存储向量到数据库之前，确保向量已归一化。务必查看提供商的文档。例如，Gemini 的嵌入 API 说明：

> 3072 维嵌入已归一化。归一化嵌入通过比较向量方向而非幅度，产生更准确的语义相似度。对于其他维度，包括 768 和 1536，你需要按如下方式归一化嵌入：

```
// TypeScriptfunction normalize(vector: number[]): number[] {  const magnitude = Math.sqrt(    vector.reduce((sum, val) => sum + val * val, 0)  );  return vector.map(v => v / magnitude);}
```

```
# Pythonimport numpy as np
def normalize(vector: list[float]) -> list[float]:    magnitude = np.linalg.norm(vector)    return (vector / magnitude).tolist()
```

大多数 OpenAI 嵌入是预归一化的。

### 陷阱 3：索引存储

HNSW 索引的大小可能是原始数据的 2-4 倍。

```
-- 监控索引大小SELECT pg_size_pretty(pg_relation_size('embeddings_embedding_idx'));
```

为大数据集规划相应的存储空间。

### 陷阱 4：距离运算符匹配

查询运算符必须与索引类型匹配：

```
-- 使用 vector_l2_ops 创建的索引CREATE INDEX ON docs USING hnsw (embedding vector_l2_ops);
-- ✅ 使用 L2 查询SELECT * FROM docs ORDER BY embedding <-> query LIMIT 10;
-- ❌ 使用余弦查询（不会使用索引）SELECT * FROM docs ORDER BY embedding <=> query LIMIT 10;
```

### 陷阱 5：相似度 ≠ 相关性

这是最容易被忽视但又最重要的概念：

**相似度**：嵌入空间中的数学距离

* 向量搜索测量的是什么
* "这些文本讨论相似的主题"

**相关性**：对回答查询的有用程度

* 用户真正关心的是什么
* "这段文本回答了我的问题"

#### 示例：为什么相似度 ≠ 相关性

查询："如何修复 Python 内存泄漏？"

```
// 高相似度，低相关性"Java 也有内存管理挑战..."           // 0.85 相似度"C++ 使用 RAII 模式保证内存安全..."    // 0.82 相似度
// 较低相似度，高相关性"使用 tracemalloc 调试：import tracemalloc..." // 0.78 相似度"调用 gc.collect() 强制垃圾回收..."           // 0.75 相似度
```

向量搜索按相似度排序，而非相关性！

#### 三层方法提升相关性

* **第 1 层**：用向量相似度广撒网（召回率）
* **第 2 层**（可选）：用关键词（BM25）过滤以提高精确度
* **第 3 层**：用交叉编码器重排序以获得真正的相关性

⚠️ pgvector 不原生支持 BM25 或重排序。

#### BM25 替代方案：PostgreSQL 的 tsvector

```
-- 添加全文搜索列ALTER TABLE docs ADD COLUMN fts tsvector  GENERATED ALWAYS AS (to_tsvector('english', content)) STORED;
CREATE INDEX ON docs USING GIN(fts);
-- 带排序的关键词搜索SELECT *, ts_rank(fts, query) AS keyword_scoreFROM docs, plainto_tsquery('english', 'postgresql') queryWHERE fts @@ query;
```

#### 混合搜索：向量 + 关键词

```
WITH vector_results AS (  SELECT id, (1 / (1 + (embedding <-> $1::vector))) AS vec_score  FROM docs ORDER BY embedding <-> $1::vector LIMIT 100),keyword_results AS (  SELECT id, ts_rank(fts, query) AS kw_score  FROM docs, plainto_tsquery('english', $2) query  WHERE fts @@ query)SELECT d.*,  COALESCE(v.vec_score, 0) * 0.7 + COALESCE(k.kw_score, 0) * 0.3 AS scoreFROM docs dLEFT JOIN vector_results v ON d.id = v.idLEFT JOIN keyword_results k ON d.id = k.idORDER BY score DESC LIMIT 10;
```

#### 在应用代码中重排序

pgvector 不支持重排序，需要在应用代码中实现：

```
// 1. 从 pgvector 获取初始结果const candidates = await db  .select()  .from(docs)  .orderBy(sql`embedding <-> ${queryEmbedding}::vector`)  .limit(100);
// 2. 使用交叉编码器重排序（例如 Cohere、Jina）const reranked = await cohere.rerank({  query: userQuery,  documents: candidates.map(c => c.content),  top_n: 10});
```

#### 流行的交叉编码器

* **Cohere Rerank API**：基于云的高质量服务
* **Jina Reranker**：开源，可自托管
* **Cross-Encoder（Sentence-Transformers）**：Python 库
* **Voyage Rerank API**：性价比高的替代方案

💡 重排序 50-100 个候选项，而非数千个（太慢）。

### 陷阱 6：分数 vs 距离

* **距离**：运算符（`<->`、`<#>`、`<=>`）的原始指标
* **分数**：归一化的 0-1 值，用于相关性排序

```
-- 距离（越小越好）SELECT embedding <-> query AS distance FROM docs;
-- 转换为分数（越大越好）SELECT 1 / (1 + (embedding <-> query)) AS score FROM docs;
```

## 多模态嵌入

pgvector 可以存储任何类型的嵌入——关键是使用兼容的模型：

* **图像**：CLIP、ImageBind、SigLIP
* **音频**：Wav2Vec 2.0、Whisper 编码器
* **视频**：VideoMAE、TimeSformer（帧级或时序）
* **代码**：CodeBERT、GraphCodeBERT

同一向量空间 → 可实现跨模态搜索。

## 最佳实践总结

1. **维度**：使用满足精度需求的最小维度；如需要可通过 API 截断
2. **距离运算符**：了解全部 6 种类型；明智选择
3. **归一化**：查看提供商文档，必要时进行归一化
4. **索引**：> 100 万行用 HNSW，1 万 - 100 万行用 IVFFlat，< 1 万行不用索引
5. **相关性**：混合搜索（BM25）和重排序（交叉编码器）
6. **多模态**：使用对齐的模型（CLIP）进行跨模态搜索
7. **存储**：监控 HNSW 索引大小（2-4 倍数据量）

## 结语

pgvector 为在 PostgreSQL 中构建生产级向量搜索和 RAG 应用提供了强大而灵活的解决方案。通过理解其核心概念、正确选择数据类型和索引，以及应用本文介绍的最佳实践，你可以构建高性能、可扩展的语义搜索系统。

记住，向量搜索只是 RAG 流程的一部分。结合混合搜索和重排序技术，可以显著提升检索的相关性，从而为大语言模型提供更高质量的上下文。

## 参考资源

* pgvector 官方文档
* PgVector 距离函数详解
* PostgreSQL 向量相似度搜索深入探讨
* Drizzle ORM pgvector 支持
* 使用 Drizzle ORM 在 Postgres 中存储向量