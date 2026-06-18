---
title: TiDB Chat2Query 深度解析：我们如何打造一款更高效、准确的智能 SQL 生成工具？
author: TiDB-平凯数据库
date:
url: http://mp.weixin.qq.com/s?__biz=MzI3NDIxNTQyOQ==&mid=2247525881&idx=1&sn=72c4c0f463340534fa315ddf3578ae5d&chksm=ea0ea03956f34a9492e4483270b544de76576ac6fd9d198766e582f3620be8f354d6a9c213b7&mpshare=1&scene=24&srcid=03101532uCixzhlmXLLEKiji&sharer_shareinfo=73c6ca7813f7099c16ea4a6039d4aaed&sharer_shareinfo_first=73c6ca7813f7099c16ea4a6039d4aaed#rd
---


> 已吸收至：[[05_数据分析与BI/0501_语义层与智能问数/050101_Text-to-SQL/050101_核心知识点/Text-to-SQL工程架构与SchemaLinking|Text-to-SQL工程架构与SchemaLinking]]

**导读**

2023 年 1 月，TiDB Cloud 发布了 Chat2Query 功能，在 TiDB Cloud 上通过自然语言提问，即可生成相应的 SQL，**对用户上传的任意数据集进行分析**。Chat2Query 正在彻底改变企业探索和理解数据的方式。

经过一年多的研发迭代，现在 Chat2Query 在 Spider 基准测试中得分 86.30，并曾在 BIRD 基准测试中跻身前四。本文将深入探讨 Chat2Query 的工作原理及其背后的 Text2SQL 技术，以及我们如何提升 Chat2Query 的能力。

* “上个季度的销售额是多少？”
* “哪个产品类别表现最佳？”
* “本月客户投诉的趋势如何？”

相比其他工具，Chat2Query 能够对用户上传的大规模数据集进行理解和分析，摒弃繁杂的专业术语和查询语句，Chat2Query 使得用户能够通过自然语言直接向数据库提问，并即时获得答案。

**丰富数据上下文**

##

首先，Chat2Query 需要熟悉您的数据。为此，它会使用关系型数据库和向量数据库来分析您的数据库。这种混合方法使 Chat2Query 能够理解数据库的结构以及表、列和实体之间的关系。其中，向量数据库尤其重要，它存储了更复杂的高维数据，帮助 Chat2Query 更好地理解数据点之间的关系。Chat2Query 对数据了解得越多，提供的回答就越准确和深入。

**提出问题**

##

在数据上下文得到充分丰富后，用户即可开始提问。Chat2Query 会将你的问题转换为 SQL 查询，提取相关数据，并生成答案——通常还会附带直观的图表或图形，让信息更加清晰易懂。

图 1. Chat2Query 的工作原理

**理解数据库（Understand DB）**

##

为了使 Chat2Query 高效运行，理解数据库的结构（schema）至关重要。这正是 “理解数据库”（Understand DB）功能的作用所在——就像给 Chat2Query 提供了一张数据的“地图”，帮助其掌握表、列和实体之间的相互关系。

* 这一步骤可以将 SQL 查询的准确率提高 2-3%（基于 Spider 等基准测试）。虽然增幅看似较小，但在处理大规模数据集时，这一提升具有重要意义。

图 2. 理解数据库

**优化提示词：提示工程（Prompt Engineering）**

##

我们不能期待随便向系统提一个复杂的问题，就获得精准的答案。因此，我们通过提示工程（Prompt Engineering）来确保 Chat2Query 精确响应问题。通过结合思维链（Chain of Thought, COT）和检索增强生成（Retrieval Augmented Generation, RAG）等先进技术，我们指导系统逐步推理问题，确保生成的 SQL 查询尽可能精准。

* COT + RAG 的结合使得 Chat2Query 在 Spider 和 BIRD 等基准测试中始终保持领先。这就是它取得卓越表现的关键。

图 3. 提示工程的主要步骤

**微调与后加工处理（Fine-Tuning with Post-Processing）**

##

即使使用 LLM（大语言模型） 这样的先进技术，幻觉现象（hallucinations） 仍可能发生。因此，我们在后加工处理阶段（post-processing）采用多智能体协作机制（multi-agent collaboration mechanism），让多个“专家”协同工作，审核 SQL 结果，识别潜在错误，并优化查询，提高其准确性。

这一机制确保即使模型遗漏或误解某些内容，系统的其他组件也能介入，捕捉问题并进行必要的调整。这种额外的优化层极大地增强了 Chat2Query 生成 SQL 查询的可靠性。

* 后处理机制 + 多智能体系统能将 SQL 查询的整体准确率提高 2-4%，有效减少错误，确保结果稳定、一致，并可直接用于业务决策。

图 4. 后加工处理机制的实际应用

现在，让我们看看 Chat2Query 如何帮助企业做出更明智的决策：

* **销售业绩分析：**无需等待报表或手动提取数据。只需询问 “本月的销售额相比上月增长了多少？”，即可即时获取数据，优化销售策略。
* **客户洞察：**快速了解客户反馈。例如，询问 “本月最常见的客户投诉类型是什么？”，即可发现需要改进的地方，从而提升服务质量。
* **供应链优化：**通过 “哪些产品的库存低于安全库存水平？” 或 “过去三个月中，哪些产品的库存周转率最高” 等问题，实时调整供应链策略，提高运营效率。
* **财务报告分析：**想提升企业财务透明度与决策效率？直接询问“本季度的总收入是多少？”或“上个月的运营成本是多少？”，即可快速获取关键财务数据，支持管理层评估收入表现、控制成本和制定战略，增强企业的整体财务健康。

随着 AI 技术的发展，企业与数据的交互方式也将更加简单。Chat2Query 仍在不断进化，把“数据驱动”的运营模式带给更多企业和个人。

**/** 相关推荐 **/**

[PingCAP 唐刘：一个咨询顾问对 TiDB Chat2Query Demo 提出的脑洞](https://mp.weixin.qq.com/s?__biz=MzI3NDIxNTQyOQ==&mid=2247509189&idx=1&sn=e2d0a145d38e0609f24ce2e841b2bd1e&scene=21#wechat_redirect)

[TiDB 8.5 LTS 发版——支持无限扩展，开启 AI 就绪新时代](https://mp.weixin.qq.com/s?__biz=MzI3NDIxNTQyOQ==&mid=2247523633&idx=1&sn=89b04901fcdcf145a5dc2b632db2f5ae&scene=21#wechat_redirect)

[黄东旭：2025 数据库技术展望](https://mp.weixin.qq.com/s?__biz=MzI3NDIxNTQyOQ==&mid=2247524462&idx=1&sn=6a19b8c2f5fbbdd49ab30d78b3f45b35&scene=21#wechat_redirect)

💡 点击文末**【阅读原文】**，立即下载试用 TiDB！