---
title: 用PostgreSQL搞定多模态数据：文本+图片Embedding实战
author: 终身求知者
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI5ODY2NzQyNA==&mid=2247484215&idx=1&sn=f88b9546cf80e2a74af055087e10c058&chksm=ed95cab7cf0b89265f0a1dd4ed7ea0e5bf9cdaa0117c8741d08408a7c80e9fb9e352132289e4&mpshare=1&scene=24&srcid=0408vL4IRqhzeaizMsnukHS3&sharer_shareinfo=3898edca0ab5a4b5b1da92d7d6701d39&sharer_shareinfo_first=3898edca0ab5a4b5b1da92d7d6701d39#rd
---

当 ChatGPT 能看懂你发的表情包，当 Claude 能分析你上传的截图，多模态 AI 已经不再是实验室里的概念，而是实实在在的业务需求。但随之而来的一个现实问题是：这些海量的文本向量、图片向量，到底该存哪儿？

Milvus、Pinecone、Weaviate 这些专用向量数据库确实很香，但引入一个新组件意味着额外的运维成本、学习曲线和数据同步链路。如果你已经在使用 PostgreSQL，其实完全可以原地进化，借助 `pgvector` 扩展，把关系型数据库变成能存能算的多模态向量数据库。

本文将手把手教你如何用 PostgreSQL 设计一套生产级的多模态数据管理方案，涵盖文本与图片 Embedding 的存储设计、跨模态检索策略，以及可直接落地的 SQL 实战。

---

## 一、为什么要在 PostgreSQL 里做多模态？

在谈技术细节前，先解决一个灵魂拷问：明明有专用的向量数据库，为什么非要在 PostgreSQL 里"硬刚"？

答案很简单：大多数业务数据本就是关系型的。你的用户表、订单表、内容表都在 PostgreSQL 里，向量只是这些实体的"语义索引"。如果为了存向量再搞一套数据库，你就需要：

1. 维护两套数据之间的同步链路
2. 处理分布式事务的一致性
3. 学习新的查询 DSL
4. 额外的基础设施成本

而 PostgreSQL + pgvector 的组合，让你可以用一条 SQL 同时过滤业务属性和语义相似度：

```
-- 查找"最近7天内上传的、与某张图片相似的夏季女装"  
SELECT * FROM products   
WHERE category = '女装'   
  AND created_at > NOW() - INTERVAL '7 days'  
ORDER BY image_embedding <-> query_vector  
LIMIT 10;
```

这种关系查询与向量检索的混血能力，正是多模态业务最核心的诉求。

---

## 二、Embedding 存储的 Schema 设计哲学

多模态数据管理最大的 Schema 设计挑战在于：文本和图片的 Embedding 维度通常不一样。

* 文本模型（如 text-embedding-3-large）：1536 维或 3072 维
* 图片模型（如 CLIP ViT-L/14）：768 维或 1024 维
* 你自己的微调模型：可能是 512 维或 2048 维

### 方案一：独立列存储（推荐）

最直接的方式是为不同模态创建独立的向量列。这种设计清晰、查询效率高，且便于为不同模态创建最优的索引。

```
CREATE EXTENSION IF NOT EXISTS vector;  
  
CREATE TABLE multimodal_content (  
    id BIGSERIAL PRIMARY KEY,  
    content_type VARCHAR(20) CHECK (content_type IN ('text', 'image', 'mixed')),  
  
    -- 原始内容  
    text_content TEXT,  
    image_url TEXT,  
  
    -- 向量存储（允许 NULL，根据内容类型填充）  
    text_embedding VECTOR(1536),  -- OpenAI text-embedding-3-large  
    image_embedding VECTOR(1024), -- CLIP 等视觉模型  
  
    -- 元数据  
    metadata JSONB DEFAULT '{}',  
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,  
  
    -- 约束：至少有一种 embedding  
    CONSTRAINT chk_has_embedding CHECK (  
        (text_embedding IS NOT NULL) OR (image_embedding IS NOT NULL)  
    )  
);  
  
-- 创建索引（IVFFlat 适合高维向量，HNSW 适合高精度检索）  
CREATE INDEX idx_text_embedding ON multimodal_content   
USING ivfflat (text_embedding vector_cosine_ops)   
WITH (lists = 100);  
  
CREATE INDEX idx_image_embedding ON multimodal_content   
USING hnsw (image_embedding vector_cosine_ops);
```

这种设计的优势是类型安全和性能最优。pgvector 的索引是针对特定维度优化的，分开存储可以避免维度不匹配的错误，也能针对不同模态的数据分布调整索引参数。

### 方案二：统一投影空间（进阶）

如果你使用的是 CLIP 这样的对齐模型（文本和图片映射到同一语义空间），或者你有自己的投影层将多模态向量映射到统一维度，可以只用一个向量列：

```
CREATE TABLE unified_content (  
    id BIGSERIAL PRIMARY KEY,  
    raw_text TEXT,  
    image_url TEXT,  
    unified_embedding VECTOR(512), -- 统一降维后的向量  
    source_modality VARCHAR(10),   -- 'text' 或 'image'，用于追踪来源  
    metadata JSONB  
);  
  
CREATE INDEX idx_unified_embedding ON unified_content   
USING hnsw (unified_embedding vector_cosine_ops);
```

这种方案适合跨模态检索（用文本搜相似图片，用图片搜相似文本），但会损失一些模态特有的细粒度信息。建议仅在向量维度差异巨大（如文本 3072 维 vs 图片 512 维）且需要频繁跨模态查询时考虑。

### 方案三：JSONB 灵活存储（应急）

如果你的向量维度经常变化，或者需要存储多个不同模型的 Embedding，可以用 JSONB 数组存储，但会牺牲索引和查询性能：

```
CREATE TABLE flexible_content (  
    id BIGSERIAL PRIMARY KEY,  
    embeddings JSONB DEFAULT '{}'::jsonb,  
    -- 示例：{"openai-text": [0.1, ...], "clip-image": [0.2, ...]}  
  
    -- 为了性能，建议同时存储主要查询向量的独立列  
    primary_embedding VECTOR(768)  
);
```

生产建议：优先使用方案一。清晰的数据模型比"灵活"更重要，且 pgvector 的索引无法在 JSONB 内的数组上生效。

---

## 三、多模态数据入库实战

设计好表结构后，我们来看如何插入数据。假设你已经有文本和图片的 Embedding 生成服务（可以是 Python 脚本、Lambda 函数或 MQ 消费者）。

### 单条插入

```
-- 插入文本内容  
INSERT INTO multimodal_content (content_type, text_content, text_embedding, metadata)  
VALUES (  
    'text',  
    'PostgreSQL 是一款强大的开源关系型数据库...',  
    '[0.023, -0.015, ..., 0.008]'::vector, -- 1536 维向量  
    '{"source": "tech_blog", "lang": "zh"}'::jsonb  
);  
  
-- 插入图片内容  
INSERT INTO multimodal_content (content_type, image_url, image_embedding, metadata)  
VALUES (  
    'image',  
    'https://cdn.example.com/images/cat.jpg',  
    '[0.115, -0.032, ..., 0.441]'::vector, -- 1024 维向量  
    '{"format": "jpg", "size": "1024x768"}'::jsonb  
);  
  
-- 插入混合内容（如图文笔记）  
INSERT INTO multimodal_content (  
    content_type,   
    text_content,   
    image_url,   
    text_embedding,   
    image_embedding,  
    metadata  
)  
VALUES (  
    'mixed',  
    '这只猫在键盘上睡觉...',  
    'https://cdn.example.com/cat-on-keyboard.jpg',  
    '[0.023, -0.015, ..., 0.008]'::vector,  
    '[0.115, -0.032, ..., 0.441]'::vector,  
    '{"tags": ["cat", "keyboard", "sleeping"]}'::jsonb  
);
```

### 批量插入优化

多模态数据往往量大（图片 Embedding 生成成本也高），批量插入能显著提升吞吐量：

```
-- 使用 COPY 或批量 INSERT  
INSERT INTO multimodal_content (content_type, text_content, text_embedding)   
VALUES   
    ('text', '内容1', '[...]'::vector),  
    ('text', '内容2', '[...]'::vector),  
    ('text', '内容3', '[...]'::vector)  
ON CONFLICT (id) DO UPDATE SET  
    text_embedding = EXCLUDED.text_embedding,  
    updated_at = CURRENT_TIMESTAMP;
```

注意：向量数据较大（1536 维 × 4 字节 ≈ 6KB/条），批量插入时建议每批控制在 1000-5000 条，避免单次事务过大。

---

## 四、多向量检索策略：不只是相似度计算

多模态检索的精髓在于如何利用两种（或多种）向量共同决策。这不是简单的 `WHERE text_embedding <-> query < 0.5`，而是需要融合策略。

### 策略一：模态路由（Modality Routing）

最简单的策略：根据查询类型选择对应向量列。用户输入文本查文本，上传图片查图片。

```
-- 文本搜文本  
SELECT id, text_content,   
       1 - (text_embedding <=> query_text_vector) AS similarity  
FROM multimodal_content  
WHERE content_type IN ('text', 'mixed')  
  AND text_embedding IS NOT NULL  
ORDER BY text_embedding <=> query_text_vector  
LIMIT 20;  
  
-- 图片搜图片  
SELECT id, image_url,  
       1 - (image_embedding <=> query_image_vector) AS similarity  
FROM multimodal_content  
WHERE content_type IN ('image', 'mixed')  
  AND image_embedding IS NOT NULL  
ORDER BY image_embedding <=> query_image_vector  
LIMIT 20;
```

这种策略实现简单，但无法实现"以文搜图"或"以图搜文"的跨模态能力。

### 策略二：晚期融合（Late Fusion）

分别检索，再合并结果。这是生产环境最常用的策略，灵活且效果好。

```
WITH text_results AS (  
    SELECT   
        id,  
        1 - (text_embedding <=> query_vector) AS score,  
        'text_match' AS match_type  
    FROM multimodal_content  
    WHERE text_embedding IS NOT NULL  
    ORDER BY text_embedding <=> query_vector  
    LIMIT 50  
),  
image_results AS (  
    SELECT   
        id,  
        1 - (image_embedding <=> query_vector) AS score,  
        'image_match' AS match_type  
    FROM multimodal_content  
    WHERE image_embedding IS NOT NULL  
    ORDER BY image_embedding <=> query_vector  
    LIMIT 50  
),  
combined AS (  
    -- 合并结果，对同一 ID 的分数进行加权  
    SELECT   
        id,  
        MAX(CASE WHEN match_type = 'text_match' THEN score END) AS text_score,  
        MAX(CASE WHEN match_type = 'image_match' THEN score END) AS image_score  
    FROM (  
        SELECT id, score, match_type FROM text_results  
        UNION ALL  
        SELECT id, score, match_type FROM image_results  
    ) t  
    GROUP BY id  
)  
SELECT   
    c.id,  
    m.text_content,  
    m.image_url,  
    c.text_score,  
    c.image_score,  
    -- 加权融合公式（权重可根据业务调整）  
    COALESCE(c.text_score, 0) * 0.6 + COALESCE(c.image_score, 0) * 0.4 AS final_score  
FROM combined c  
JOIN multimodal_content m ON c.id = m.id  
WHERE c.text_score IS NOT NULL OR c.image_score IS NOT NULL  
ORDER BY final_score DESC  
LIMIT 10;
```

关键点：

* 使用 CTE（Common Table Expressions）分别获取 Top-K，减少计算量
* 使用 `COALESCE` 处理只命中一种模态的情况
* 权重 0.6/0.4 只是示例，实际应通过 A/B 测试确定

### 策略三：RRF（Reciprocal Rank Fusion）

当不同模态的相似度分数不可比时（文本相似度 0.9 和图片相似度 0.9 不一定等价），使用 RRF 公式融合排名比融合分数更稳健：

```
WITH text_results AS (  
    SELECT id, row_number() OVER (ORDER BY text_embedding <=> query_vector) AS rank  
    FROM multimodal_content  
    WHERE text_embedding IS NOT NULL  
    ORDER BY text_embedding <=> query_vector  
    LIMIT 100  
),  
image_results AS (  
    SELECT id, row_number() OVER (ORDER BY image_embedding <=> query_vector) AS rank  
    FROM multimodal_content  
    WHERE image_embedding IS NOT NULL  
    ORDER BY image_embedding <=> query_vector  
    LIMIT 100  
),  
rrf_scores AS (  
    SELECT   
        COALESCE(t.id, i.id) AS id,  
        -- RRF 公式：score = Σ 1/(k + rank)，k 通常取 60  
        COALESCE(1.0 / (60 + t.rank), 0) +   
        COALESCE(1.0 / (60 + i.rank), 0) AS rrf_score  
    FROM text_results t  
    FULL OUTER JOIN image_results i ON t.id = i.id  
)  
SELECT   
    r.id,  
    m.text_content,  
    m.image_url,  
    r.rrf_score  
FROM rrf_scores r  
JOIN multimodal_content m ON r.id = m.id  
ORDER BY r.rrf_score DESC  
LIMIT 10;
```

RRF 的优势在于无需调参，对各种模态的评分尺度不敏感，特别适合快速原型验证。

### 策略四：条件过滤 + 向量检索

实际业务中，很少单纯做语义检索，通常需要结合业务属性过滤：

```
-- 查找"电子产品类别下，与某张图片相似且描述包含'便携'的商品"  
SELECT   
    id,   
    product_name,  
    image_url,  
    1 - (image_embedding <=> query_vector) AS visual_similarity  
FROM multimodal_content  
WHERE   
    -- 业务过滤（利用 B-Tree 索引）  
    metadata->>'category' = 'electronics'  
    AND created_at > '2024-01-01'  
  
    -- 文本过滤（利用 GIN 索引）  
    AND text_content ILIKE '%便携%'  
  
    -- 向量过滤（利用 IVFFlat/HNSW 索引）  
    AND image_embedding IS NOT NULL  
ORDER BY image_embedding <=> query_vector  
LIMIT 20;
```

性能提示：pgvector 的索引支持带过滤条件的向量检索，但过滤条件太严格（返回结果少于 1%）时，索引效率会下降。此时可以考虑使用 `materialized view` 按类别预分区。

---

## 五、跨模态对齐：当文本和图片在同一个向量空间

如果你使用的是 CLIP 或国产的 Chinese-CLIP 这类对齐模型，文本和图片已经被映射到同一语义空间。这时可以真正实现"以文搜图"：

```
-- 用文本向量直接检索图片  
WITH query_embedding AS (  
    -- 假设这是 "一只橘猫在睡觉" 的文本 Embedding  
    SELECT '[0.1, -0.2, ...]'::vector(512) AS vec  
)  
SELECT   
    id,  
    image_url,  
    metadata->>'description' AS auto_caption,  
    1 - (image_embedding <=> q.vec) AS similarity  
FROM multimodal_content, query_embedding q  
WHERE content_type IN ('image', 'mixed')  
ORDER BY image_embedding <=> q.vec  
LIMIT 10;
```

### 零样本分类

利用对齐的向量空间，甚至可以实现零样本（Zero-shot）分类：

```
-- 假设有预定义的类别文本向量  
WITH category_embeddings AS (  
    SELECT '风景' AS label, '[0.5, ...]'::vector AS vec  
    UNION ALL SELECT '人物', '[0.3, ...]'::vector  
    UNION ALL SELECT '美食', '[0.8, ...]'::vector  
)  
SELECT   
    m.id,  
    m.image_url,  
    c.label AS predicted_category,  
    1 - (m.image_embedding <=> c.vec) AS confidence  
FROM multimodal_content m  
CROSS JOIN category_embeddings c  
WHERE m.id = 12345 -- 某张待分类图片  
ORDER BY m.image_embedding <=> c.vec  
LIMIT 1;
```

---

## 六、性能优化与生产 checklist

把多模态数据放到 PostgreSQL 只是开始，要保证生产环境的查询性能，还需要以下优化：

### 1. 索引选择：IVFFlat vs HNSW

* IVFFlat

  内存占用小，适合静态数据（lists 参数建议设置为数据量的平方根）
* HNSW

  查询速度快，适合动态插入（ef\_construction 建议 64-128，m 建议 16-32）

对于 100 万条 1536 维向量：

```
-- IVFFlat（内存约 500MB）  
CREATE INDEX idx_ivf ON multimodal_content   
USING ivfflat (text_embedding vector_cosine_ops)   
WITH (lists = 1000);  
  
-- HNSW（内存约 2GB，但查询快 5-10 倍）  
CREATE INDEX idx_hnsw ON multimodal_content   
USING hnsw (text_embedding vector_cosine_ops)   
WITH (m = 16, ef_construction = 100);
```

### 2. 分区策略

如果数据量超过千万级，建议按时间或类别分区：

```
-- 按月分区  
CREATE TABLE multimodal_content_2024_01 PARTITION OF multimodal_content  
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');  
  
-- 每个分区单独建索引  
CREATE INDEX ON multimodal_content_2024_01   
USING hnsw (text_embedding vector_cosine_ops);
```

### 3. 向量量化（Product Quantization）

对于超大规模数据（亿级），pgvector 支持将向量转换为 `halfvec`（FP16）减少存储：

```
-- 查询时自动转换，节省 50% 存储  
SELECT id   
FROM multimodal_content  
ORDER BY text_embedding::halfvec <=> query_vector::halfvec  
LIMIT 10;
```

### 4. 连接池与并行查询

多模态检索往往涉及复杂的多表 JOIN 和 CTE，确保：

* 使用 PgBouncer 管理连接池
* 开启并行查询：`SET max_parallel_workers_per_gather = 4;`
* 为 metadata JSONB 列创建 GIN 索引：`CREATE INDEX idx_meta ON multimodal_content USING GIN (metadata);`

---

## 七、完整实战：构建图文混合推荐系统

最后，给出一个可直接运行的完整示例：为一个内容平台构建"相似内容推荐"功能。

```
-- 步骤 1：建表  
CREATE EXTENSION IF NOT EXISTS vector;  
  
CREATE TABLE content_repository (  
    id BIGSERIAL PRIMARY KEY,  
    title VARCHAR(255),  
    content_type VARCHAR(20) CHECK (content_type IN ('article', 'image', 'video')),  
  
    -- 多模态向量（假设都映射到 768 维统一空间）  
    text_vec VECTOR(768),  
    visual_vec VECTOR(768),  
  
    -- 业务字段  
    author_id INTEGER,  
    tags TEXT[],  
    publish_date DATE,  
    status VARCHAR(20) DEFAULT 'active',  
  
    -- 混合查询辅助列：是否有视觉内容  
    has_visual BOOLEAN GENERATED ALWAYS AS (visual_vec IS NOT NULL) STORED  
);  
  
-- 步骤 2：建索引  
CREATE INDEX idx_content_hnsw_text ON content_repository   
USING hnsw (text_vec vector_cosine_ops)   
WITH (m = 16, ef_construction = 64);  
  
CREATE INDEX idx_content_hnsw_visual ON content_repository   
USING hnsw (visual_vec vector_cosine_ops)   
WHERE visual_vec IS NOT NULL; -- 部分索引，只索引有图片的内容  
  
CREATE INDEX idx_content_tags ON content_repository USING GIN(tags);  
  
-- 步骤 3：推荐函数（简化版）  
CREATE OR REPLACE FUNCTION get_hybrid_recommendations(  
    query_text_vec VECTOR(768),  
    query_visual_vec VECTOR(768),  
    user_id INTEGER,  
    rec_limit INTEGER DEFAULT 10  
) RETURNS TABLE (  
    content_id BIGINT,  
    title VARCHAR,  
    similarity_score FLOAT,  
    match_source TEXT  
) AS   
  

BEGINRETURNQUERYWITHcandidatesAS(−−文本相似度检索SELECTc.id,c.title,1−(c.textvec<=>querytextvec)ASsim,′text′::TEXTASsourceFROMcontentrepositorycWHEREc.status=′active′ANDc.authorid!=userid−−不推荐自己的内容ANDc.textvecISNOTNULLORDERBYc.textvec<=>querytextvecLIMITreclimit∗2UNIONALL−−视觉相似度检索SELECTc.id,c.title,1−(c.visualvec<=>queryvisualvec)ASsim,′visual′::TEXTASsourceFROMcontentrepositorycWHEREc.status=′active′ANDc.authorid!=useridANDc.visualvecISNOTNULLORDERBYc.visualvec<=>queryvisualvecLIMITreclimit∗2),rankedAS(SELECTid,title,MAX(CASEWHENsource=′text′THENsimEND)AStextsim,MAX(CASEWHENsource=′visual′THENsimEND)ASvisualsim,−−混合评分：优先双匹配，其次文本，最后视觉CASEWHENCOUNT(DISTINCTsource)=2THEN0.6∗MAX(CASEWHENsource=′text′THENsimEND)+0.4∗MAX(CASEWHENsource=′visual′THENsimEND)WHENMAX(source)=′text′THEN0.8∗MAX(sim)ELSE0.7∗MAX(sim)ENDASfinalscoreFROMcandidatesGROUPBYid,title)SELECTr.id,r.title,r.finalscore::FLOAT,CASEWHENr.textsimISNOTNULLANDr.visualsimISNOTNULLTHEN′hybrid′WHENr.textsimISNOTNULLTHEN′text′ELSE′visual′ENDFROMrankedrORDERBYr.finalscoreDESCLIMITreclimit;END;BEGIN  
    RETURN QUERY  
    WITH candidates AS (  
        -- 文本相似度检索  
        SELECT   
            c.id,  
            c.title,  
            1 - (c.text_vec <=> query_text_vec) AS sim,  
            &#x27;text&#x27;::TEXT AS source  
        FROM content_repository c  
        WHERE c.status = &#x27;active&#x27;  
          AND c.author_id != user_id  -- 不推荐自己的内容  
          AND c.text_vec IS NOT NULL  
        ORDER BY c.text_vec <=> query_text_vec  
        LIMIT rec_limit * 2  
  
        UNION ALL  
  
        -- 视觉相似度检索  
        SELECT   
            c.id,  
            c.title,  
            1 - (c.visual_vec <=> query_visual_vec) AS sim,  
            &#x27;visual&#x27;::TEXT AS source  
        FROM content_repository c  
        WHERE c.status = &#x27;active&#x27;  
          AND c.author_id != user_id  
          AND c.visual_vec IS NOT NULL  
        ORDER BY c.visual_vec <=> query_visual_vec  
        LIMIT rec_limit * 2  
    ),  
    ranked AS (  
        SELECT   
            id,  
            title,  
            MAX(CASE WHEN source = &#x27;text&#x27; THEN sim END) AS text_sim,  
            MAX(CASE WHEN source = &#x27;visual&#x27; THEN sim END) AS visual_sim,  
            -- 混合评分：优先双匹配，其次文本，最后视觉  
            CASE   
                WHEN COUNT(DISTINCT source) = 2 THEN 0.6 * MAX(CASE WHEN source = &#x27;text&#x27; THEN sim END)   
                                                      + 0.4 * MAX(CASE WHEN source = &#x27;visual&#x27; THEN sim END)  
                WHEN MAX(source) = &#x27;text&#x27; THEN 0.8 * MAX(sim)  
                ELSE 0.7 * MAX(sim)  
            END AS final_score  
        FROM candidates  
        GROUP BY id, title  
    )  
    SELECT   
        r.id,  
        r.title,  
        r.final_score::FLOAT,  
        CASE   
            WHEN r.text_sim IS NOT NULL AND r.visual_sim IS NOT NULL THEN &#x27;hybrid&#x27;  
            WHEN r.text_sim IS NOT NULL THEN &#x27;text&#x27;  
            ELSE &#x27;visual&#x27;  
        END  
    FROM ranked r  
    ORDER BY r.final_score DESC  
    LIMIT rec_limit;  
END;

  
  
 LANGUAGE plpgsql;  
  
-- 步骤 4：使用  
SELECT * FROM get_hybrid_recommendations(  
    '[0.1, 0.2, ...]'::vector,  -- 当前文章文本向量  
    '[0.3, 0.4, ...]'::vector,  -- 当前文章配图向量（如果有）  
    12345,                      -- 当前用户 ID  
    10  
);
```

这个方案的核心优势在于：

1. 存储层统一

   所有数据在一个事务内保持一致性
2. 查询灵活性

   可以随意组合文本过滤、标签过滤和向量相似度
3. 可解释性

   通过 `match_source` 字段知道推荐来自文本理解还是视觉相似

---

## 八、边界与权衡：什么时候不该用 PostgreSQL？

虽然 PostgreSQL + pgvector 很强大，但也要清楚它的边界：

1. 超大规模场景：如果数据量超过 10 亿条向量，或者 QPS 超过 1 万，专用向量数据库（如 Milvus、Qdrant）的分布式架构更有优势。
2. 实时性要求极高：pgvector 的索引更新是实时的，但相比内存型数据库（如 Redis Vector），延迟还是稍高。
3. 复杂的向量运算：如果需要在数据库端执行向量聚类、降维等复杂 ML 运算，Python 生态（Faiss、Annoy）更灵活。

决策树：

* 数据量 < 1000 万，已有 PostgreSQL 基础设施 → 直接用 pgvector
* 需要强一致性（金融、电商库存）→ 必须用 pgvector（事务支持）
* 纯向量检索，数据量 > 1 亿，QPS > 10k → 考虑专用向量数据库
* 混合场景 → PostgreSQL 存主数据 + 专用向量库存索引（通过外部表或逻辑复制同步）

---

## 结语

多模态 AI 的落地，不仅是算法模型的比拼，更是数据工程的较量。PostgreSQL 凭借 `pgvector` 扩展，成功打破了关系型数据库与 AI 向量存储之间的壁垒，让我们可以用最熟悉的 SQL 玩转文本与图片的 Embedding。

从 Schema 设计的维度考量，到多向量检索的策略选择，再到生产环境的性能调优，希望这篇文章能给你一份实用的技术地图。下次当产品经理说"我们要做一个以图搜文的功能"时，你可以淡定地回复："给我半小时，在现有数据库上就能跑通 Demo。"

毕竟，在技术选型日益复杂的今天，能用熟悉的工具解决新问题，本身就是一种稀缺的能力。

---

参考文献与延伸阅读：

* pgvector 官方文档：https://github.com/pgvector/pgvector
* Multi-modal Retrieval 论文：Late Interaction 与 ColBERT 架构
* PostgreSQL 14+ 并行查询优化指南