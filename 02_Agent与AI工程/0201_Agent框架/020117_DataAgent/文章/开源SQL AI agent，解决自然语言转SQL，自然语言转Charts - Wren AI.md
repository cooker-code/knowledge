---
title: 开源SQL AI agent，解决自然语言转SQL，自然语言转Charts - Wren AI
author: 智海观潮
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247499149&idx=1&sn=fe5eff29689a33d9d186e2caf45e7e1a&chksm=e81045d9d6ee0084c3846d83caf2231aa8d5e3bdaeb325dca80fb97db4cf4ad21a3ccda1c9bd&mpshare=1&scene=24&srcid=0320qJCA8TsReEutQu4dyH1w&sharer_shareinfo=bd7d9d5150022412e0e0babcc453bba1&sharer_shareinfo_first=bd7d9d5150022412e0e0babcc453bba1#rd
---

作为一名曾经的数据分析人员，每天穿梭于产品、开发、业务团队之间，写各种SQL来满足老板们、各个团队对数据的需求。996家常便饭，即使想通过知识分享的方式，“培养”这些对数据有需求的同事，自己写SQL完成诸如报表生成等的数据需求场景。然而现实往往是残酷的，毕竟对于非技术人员，从海量数据中获取有价值的信息并非易事，SQL语言的复杂性成为了他们与数据之间的一个不可逾越的屏障。

现在人工智能大模型非常火爆，想着如果能有一款工具可以直接将自然语言转换为SQL或者生成报表，那就太好了。

但是在使用RAG与LLMs查询数据库通常会有很多挑战，比如**上下文收集**时，不同来源数据的互操作性及数据与元数据的复杂链接是难题；**检索阶段**，向量存储优化和语义搜索准确性至关重要；**SQL生成**需确保查询的准确性和可执行性，同时适应不同数据库方言。

直到最近在github冲浪时，发现了一款开源SQL AI agent - Wren AI，黎明的曙光终于到来了..

Wren AI 是一个开源的 SQL 人工智能代理，它通过聊天、精心设计且直观的用户界面和用户体验，为数据、产品和业务团队提供见解，并且能够无缝集成Excel和Google表格等工具。可以通过用自然语言查询数据库，生成相应的SQL（Text2SQL/NL2SQL）、图表（Text2Chart/NL2Chart）来满足业务需求场景，极大简化了数据交互方式，让数据和智能深度融合。

1. 架构

2. 功能特性

* 支持多种语言

中文、英语、德语、西班牙语、法语、日语、韩语、葡萄牙语、等

* 支持多种数据源

MySQL、Oracle、PostgreSQL、Microsoft SQL Server、ClickHouse、Redshift、BigQuery、DuckDB、Trino、Athena (Trino)、Snowflake。

* 支持的常见的Large Language Models

OpenAI Models、Azure OpenAI Models、DeepSeek Models、Google AI Studio – Gemini Models、Vertex AI Models (Gemini + Anthropic)、Bedrock Models、Anthropic API Models、Groq Models、Ollama Models、Databricks Models。

3. 应用场景

* 语义索引搭配精心设计的用户界面/用户体验

Wren AI 实现了一种语义引擎架构，以提供与您业务相关的大型语言模型（LLM）上下文。您可以轻松地在数据架构上建立逻辑表示层，帮助LLM更好地了解您的业务上下文。

* **生成SQL查询 with context**

借助Wren AI，可以运用“建模定义语言”来处理元数据、架构、术语、数据关系以及计算和聚合背后的逻辑，从而减少重复编码并简化数据连接。

* 无需编码即可获取洞察

在Wren AI中开始新对话时，你的问题将被用来找到最相关的数据表。从这些数据表中，大型语言模型（LLM）会生成三个相关问题供用户选择。你还可以提出后续问题以获取更深入的见解。

* **生成式商业智能（GenBI）**

**为用户提供人工智能生成的总结，这些总结与SQL查询一起提供关键见解，简化复杂数据。用户可以立即将查询结果转换为人工智能生成的报告和图表，将原始数据转化为清晰、可操作的可视化信息。**

**4. 项目地址**

https://github.com/Canner/WrenAI