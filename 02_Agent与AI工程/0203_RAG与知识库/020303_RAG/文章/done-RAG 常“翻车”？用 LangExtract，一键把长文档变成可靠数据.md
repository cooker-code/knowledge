> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020303_RAG/020303_核心知识点/RAG生命周期与评估边界|RAG生命周期与评估边界]]
---
title: RAG 常“翻车”？用 LangExtract，一键把长文档变成可靠数据
author: 敬AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzYyMTMwMjQ0NQ==&mid=2247483680&idx=1&sn=87bf4f7a5181ada402674515f957e54d&chksm=feb55d56343dddda3a240d10e32b23f1cac8ddb7f33732ed738b8f3730d47c65e3e34c0335b4&mpshare=1&scene=24&srcid=0911a7SIBZn2B820e0SpPTPO&sharer_shareinfo=33359b3a4b5084e209292a250a5655f2&sharer_shareinfo_first=33359b3a4b5084e209292a250a5655f2#rd
---

检索增强生成（RAG）答非所问、引用不准？别再硬扛。LangExtract 直接从原文“挖”结构化信息，每一条都有确切来源可追溯，省心、省力、可核验。

为什么选 LangExtract？

* 精准溯源：每个字段都定位到原文位置，所见即所得，审核更放心。
* 结构稳定：少量示例定义 schema，受控生成，结果一致不跑偏。
* 长文不怕：智能分块＋并行处理＋多轮召回，解决“针在草堆”难题。
* 即看即用：一键生成交互式 HTML，高亮上千实体，快速复核与分享。
* 模型灵活：支持 Gemini、Ollama 等，本地/云端随心切换。
* 低门槛适配：用几条示例就能定制任务，无需微调，跨行业即插即用。
* 善用模型知识：通过清晰提示与示例，获得更可信的推断与补全。

适用场景：

* 医疗：从临床笔记提取诊断、用药、检查。
* 法务合规：合同要素提取与条款溯源。
* 金融与研究：研报、公告关键信息结构化。
* 客服与运维：日志、工单要点聚合。

第一步: 安裝 (非開發者)

conda create -n "LangExtract" python == "3.10"

conda activate LangExtract

pip install langextract openai

第二步: 取得OPENAI API KEY 國內 OPEN AI COMPATIBLE 端口

第三步: 跑簡單的例子

3.1 首先明確定義你想結構化的內容 (這裡是故事中的人物，心情，關係)

```
import langextract as lximport textwrap# 1. Define the prompt and extraction rulesprompt = textwrap.dedent("""\    Extract characters, emotions, and relationships in order of appearance.    Use exact text for extractions. Do not paraphrase or overlap entities.    Provide meaningful attributes for each entity to add context.""")
```

3.2 各提供一個簡單的抽取樣本

```
# 2. Provide a high-quality example to guide the modelexamples = [    lx.data.ExampleData(        text="ROMEO. But soft! What light through yonder window breaks? It is the east, and Juliet is the sun.",        extractions=[            lx.data.Extraction(                extraction_class="character",                extraction_text="ROMEO",                attributes={"emotional_state": "wonder"}            ),            lx.data.Extraction(                extraction_class="emotion",                extraction_text="But soft!",                attributes={"feeling": "gentle awe"}            ),            lx.data.Extraction(                extraction_class="relationship",                extraction_text="Juliet is the sun",                attributes={"type": "metaphor"}            ),        ]    )]
```

3.3 測試效果 (lx.extract 需要 API KEY)

```
# The input text to be processedinput_text = "Lady Juliet gazed longingly at the stars, her heart aching for Romeo"# Run the extractionresult = lx.extract(    text_or_documents=input_text,    prompt_description=prompt,    examples=examples,    model_id="gemini-2.5-flash",)
```

3.4 看答案 (利用JSONL生成可視化HTML檔案)

```
    # Save the results to a JSONL file    lx.io.save_annotated_documents([result], output_name="extraction_results.jsonl", output_dir=".")    # Generate the visualization from the file    html_content = lx.visualize("extraction_results.jsonl")    with open("visualization.html", "w", encoding="utf-8") as f:        if hasattr(html_content, 'data'):            f.write(html_content.data)  # For Jupyter/Colab        else:            f.write(html_content)
```

恭喜你，你已經成功建立NER (Name Entity Recognition)

4. 在 langextract/doc/examples 下面有更多應用場景:

例子

4.1 長文

4.2 醫學藥物

4.3 放射線(XRAY)診斷報告

5. 生成的NER可以導入知識圖譜Knowledge Graph (Node, Edge)大大降低幻覺!

結語

LangExtract 是谷歌开源的 Python 库，利用大语言模型从非结构化文本中精准提取结构化信息。它支持精确定位每条信息源头，结构稳定，能智能处理长文档。只需少量示例，无需微调，快速适配各种行业。适用于医疗、法务、金融等领域，极大提升数据提取的准确性和可追溯性。LangExtract 是谷歌开源的 Python 库，利用大语言模型从非结构化文本中精准提取结构化信息。它支持精确定位每条信息源头，结构稳定，能智能处理长文档。只需少量示例，无需微调，快速适配各种行业。适用于医疗、法务、金融等领域，极大提升数据提取的准确性和可追溯性。

引用

> https://github.com/google/langextract
>
> GOOGLE