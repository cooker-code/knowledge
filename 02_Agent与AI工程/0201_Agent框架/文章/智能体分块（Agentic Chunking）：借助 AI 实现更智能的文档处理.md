---
title: 智能体分块（Agentic Chunking）：借助 AI 实现更智能的文档处理
author: 番石榴AI
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk2ODA3MTgyOQ==&mid=2247484252&idx=1&sn=46193c85a9066b973c6e75efff0c9c6d&chksm=c5271bbc7092fb74eef2cc3bcbe34b869e460803556724cef5c76a519d7f13b99fa7b4d77754&mpshare=1&scene=24&srcid=0912yPpsJQJlEZgI0wPA2k0q&sharer_shareinfo=8df2d0debbdcdb851558479eb2533c44&sharer_shareinfo_first=8df2d0debbdcdb851558479eb2533c44#rd
---

人工智能应用面临的挑战之一是处理海量文本数据。传统分块技术常常会遗漏上下文关联，这降低了基于检索的 AI 模型的效用。而**智能主体分块（Agentic Chunking）** 作为一种更智能的文档分割方法，能够提升检索、总结和处理效率，可有效解决这一问题。

本文主要有以下内容：

* 智能主体分块的定义及其重要性
* 大型语言模型（LLMs）如何助力生成有意义的文本片段
* 向量存储与嵌入（Embeddings）在智能检索中的作用
* 一个实用的 Python 实现方案，该方案运用了 Hugging Face、Pinecone 和 LangChain 工具

通过阅读本文，你将了解智能主体分块如何通过将非结构化文本转化为可检索的知识，从而释放 AI 应用的潜力。

## 一、什么是智能主体分块？

传统分块技术会使用任意分隔符（如换行符）、固定令牌数量限制或句子数量来自动分割文本。这往往会导致上下文丢失，关键概念要么被遗漏，要么分散在多个片段中。

智能主体分块则采用了更智能的方法：

* 识别逻辑断点，例如章节、主题或论证流程的自然分隔处
* 借助 AI 为每个分块生成标题和摘要，以便于检索
* 通过分块间的内容重叠，保持上下文的连贯性

这种方法确保每个分块都包含一个完整、独立的概念，从而提升了 AI 驱动的检索和总结效果。

## 二、智能主体分块的工作原理

### 1. 文本提取与清洗

* PDF 和各类文档中通常包含冗余信息（如页眉、页脚、页码）
* 我们使用 PDFMiner 工具提取原始文本，并清理其中不必要的元素

### 2. 智能文本分割

智能主体分块不采用僵化的令牌数量限制，而是动态分割文本：

* 一级分割：按逻辑结构（页面、章节、段落）分割
* 二级分割：采用滑动窗口法，确保分块间存在内容重叠

### 3. 借助 LLM 生成摘要与标题

利用大型语言模型（如 deepseek）对每个分块进行优化：

* 生成关键信息的简洁摘要
* 创建描述性标题，方便快速查阅

### 4. 嵌入处理与 Pinecone 存储

* 将每个分块转换为向量嵌入（使用 Hugging Face 模型）
* 存储到 Pinecone 中，实现语义搜索和高效检索

## 三、智能主体分块的核心优势

1. **更精准的 AI 检索**

   ：AI 不再检索随机的文本片段，而是获取连贯、带摘要的完整章节
2. **更优质的总结效果**

   ：AI 生成的摘要能保留关键信息，降低 “幻觉”（生成虚假信息）的风险
3. **更快的搜索与检索速度**

   ：Pinecone 向量存储支持基于语义含义的即时查询，而非仅依赖关键词
4. **更贴近人类的 AI 响应**

   ：应用于聊天机器人和检索增强生成（RAG）时，响应会更具上下文关联性和针对性

## 四、代码实现：智能主体分块的实际应用

下面将逐步介绍 Python 实现方案。该系统可实现以下功能：

* 从 PDF 中提取文本
* 采用上下文感知方法对文本进行分块
* 借助 deepseek 为每个分块生成摘要和标题
* 使用 Hugging Face 对分块进行嵌入处理
* 存储到 Pinecone 中，以备检索使用

### 1. 从 PDF 中提取文本

```
from pdfminer.high_level import extract_text  
def extract_text_from_pdf(pdf_path: str) -> str:    """从pdf中抽取文本"""    try:        return extract_text(pdf_path)    except Exception as e:        print(f"提取错误 {pdf_path}: {e}")        return ""
```

### 2. 文本清洗与预处理

```
import re  
def clean_text(text: str) -> str:    """移除不需要的字符"""    text = re.sub(r'[^A-Za-z0-9\s.,;:()\-]', '', text)    return re.sub(r'\s+', ' ', text).strip()
```

### 3. 带重叠的智能分块

```
class AgenticChunker:    def __init__(self, llm):        self.llm = llm  
    def chunk_document(self, text: str, target_chars=2000, overlap_chars=200):        """将文本分割成重叠的块，并为每个块提供摘要。"""        segments = text.split("\n\n")         full_text = " ".join(segments)        chunks, start = [], 0  
        while start < len(full_text):            end = start + target_chars            chunk = full_text[start:end]            chunks.append(chunk)            start = max(0, start + target_chars - overlap_chars)  
        return [{"text": chunk, "summary": self._get_summary(chunk)} for chunk in chunks]  
    def _get_summary(self, text: str):        """利用llm生成摘要"""        return self.llm.generate_summary(text)
```

```

```

### 4. 借助 deepseek生成摘要与标题

```
from langchain_openai import ChatOpenAI  
llm = ChatOpenAI(model_name='deepseek', temperature=0.2)  
def generate_summary(text):    """摘要"""    prompt = f"请用一句话概括这段文本:\n\n{text}"    return llm.predict(prompt)
```

```

```

### 5. 将分块转换为向量嵌入（使用 Hugging Face）

```
from transformers import AutoTokenizer, AutoModelimport torch  
tokenizer = AutoTokenizer.from_pretrained("dwzhu/e5-base-4k")model = AutoModel.from_pretrained("dwzhu/e5-base-4k")  
def get_embedding(text: str):    inputs = tokenizer(text, return_tensors='pt', truncation=True, max_length=512)    with torch.no_grad():        outputs = model(**inputs)    return outputs.last_hidden_state.mean(dim=1).squeeze().numpy().tolist()
```

```

```

### 6. 存储到 Pinecone 中以备检索

```
from pinecone import Pinecone  
pinecone_api_key = "您的PINECONE API密钥"pc = Pinecone(api_key=pinecone_api_key)index = pc.Index("agentic-chunks")  
def upsert_to_pinecone(chunk_id, text, summary, embedding):    vector = {        "id": chunk_id,        "values": embedding,        "metadata": {"text": text, "summary": summary}    }    index.upsert(vectors=[vector])
```

## 五、智能主体分块的潜在应用场景

目前，这项技术已被应用于多种高级 AI 应用中，例如：

* **检索增强生成（RAG）**

  ：AI 模型获取上下文相关的文本片段，以提供精准响应
* **企业文档搜索**

  ：大型企业利用智能主体分块检索内部文档
* **AI 聊天机器人**

  ：更智能的聊天机器人能够检索包含完整上下文的数据库响应
* **法律与医疗 AI**

  ：在检索大型数据库时，不丢失已提取的文本信息

## 六、总结

智能主体分块将非结构化文本转化为可利用的知识。与简单的基于令牌的分割方法相比，这种方法具有以下优势：

* 保留上下文关联性
* 生成具有洞察力的摘要
* 提升 AI 检索效果