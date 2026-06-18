---
title: 提示词怎么写，以 Text2SQL 为例
author: AI爱好者小站
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU4NjY4MzA4Ng==&mid=2247484462&idx=1&sn=12e8f6f13089f1a0321924cca8480216&chksm=fcbd259ef71e23adef475badc17e74b340a8c01ae3207f24f738daac37051f69230de6c2dc5f&mpshare=1&scene=24&srcid=10279roDUYS2LiMGhILxK9aA&sharer_shareinfo=bbaeba2c5e17cafaaf28c3d428ecaaa5&sharer_shareinfo_first=bbaeba2c5e17cafaaf28c3d428ecaaa5#rd
---

##### 【关键词】提示词工程；Prompt Engineering；Text2SQL；格式化；思维链

提示词是人与大模型之间沟通的语言，其重要性不言而喻，写了很多提示词也踩了不少坑，跟大家分享一些心得，本文以 Text2SQL 为例，详细拆解，逐一分析。

> 注：本文关注 [语言模型] 的提示词，[视觉模型] 提示词延伸阅读请看文末链接~

## 省流版

### 提示词的工程哲学：思考和表达你想要什么 - 清晰、准确、精炼

### 提示词的技术直觉：通过提示词激活大模型的相关参数

### 

tips 具体来说：

* 精简，拒绝废话
* 用词精确
* 格式化

  - html / markdown

  - 指令格式化

  - 输出如sql / json 格式化
* COT 思维链
* 例子要多次检查，确保准确
* 多次调试，迭代

## 以 Text2SQL 为例

**任务需求**

我需要大模型根据一段话，分析并生成 SQL 语句

假设有2张表，一张产品 products 表，一张预定 bookings 表

**Text2SQL 提示词 - 中文版**

对照上面的 tips 逐一分析

1.激活相关参数和语义空间

一开始给大模型定义的角色和任务，激活相关参数，定义了大模型的语义空间。让大模型不是聊天解闷，不是帮写小说，而是有专业任务的“SQL 开发专家”。

2.格式化

* 使用了 markdown 格式组织提示词（Claude 模型用 html 格式效果更好）
* 输出指定 sql 格式
* 需要强调的用 \*\*强调内容\*\* 加粗。

3.COT思维链

第 4 行明确要求：

```
作答前必须使用**思维链**先梳理思考过程，作答后还需对结果进行检查。。
```

4.示例

示例需要反复检查，确保示例符合自己的要求，附上思考过程，输出要格式化并且准确

5.反复调试

见下文 “试一试” 部分

完整提示词：

```
你是一名SQL开发专家。根据自然语言需求，按照数据库表结构生成PostgreSQL查询语句。你必须遵循以下操作指令和约束条件来创建PostgreSQL查询语句。  
# 操作指令1. 作答前必须使用**思维链**先梳理思考过程，作答后还需对结果进行检查。2. **核心要求**：生成SQL查询语句时，必须保留固定的基础语句，不得对其进行任何修改或新增内容：     "SELECT bookings.device_id, bookings.user_id FROM public.products INNER JOIN public.bookings ON public.products.device_id = bookings.device_id"  
# 约束条件- 不得虚构上述表结构中不存在的任何表或字段。- 不得在WHERE子句中使用bookings表的字段。  
# 数据库表结构说明表名为“products”，字段及说明如下：  {database_schema}  
# 示例输入：获取产品名称为SSD且价格≥500的预订信息  输出：该需求要求获取名称为SSD且价格超过500的产品的预订信息。我会通过products表的price_info->>'price'字段和storage_info->>'category'字段进行筛选，筛选出价格至少为500的产品。同时保留固定的基础语句。```sqlSELECT bookings.device_id,bookings.user_idFROM public.productsINNER JOIN public.bookings ON public.products.device_id = bookings.device_idWHERE (public.products.price_info->>'price')::numeric >= 500AND (public.products.storage_info->>'category') = 'SSD'```
```

**试一试**

用上面的提示词，先换一个简单的任务，看是否按照需要的输出并不断调试：

重点关注：思维链思考过程，输出格式等

**Text2SQL 提示词 - 英文版**

如下：

```
You are an expert SQL developer. Given a natural language requirement, generate a PostgreSQL query as per database Schema. You must follow the instructions and constraints below to create the PostgresQL query.  
# INSTRUCTIONS:- 1. ALWAYS take chain of thought before answering and review the result after answersing.- 2. **Core Requirement**: When generating the SQL query, you **must retain the fixed base statement** without any modifications or additions:     "SELECT bookings.device_id, bookings.user_id FROM public.products INNER JOIN public.bookings ON public.products.device_id = bookings.device_id"  
# CONSTRAINTS- DO NOT make up any table and field which is not in the above Schema.- DO NOT use bookings table fields in WHERE clause.  
# Here are the database table Schema:Table name is "products", fields and description are as follows:{database_schema}  
# EXAMPLEinput: Give me the booking information of products which name is SSD and price >= 500output: The requirement ask for NVMe SSD and price more than 500. I will filter the products table based on the price_info->>'price' and storage_info->>'category' fields to get products with at least 500. And keep the fixed base statement.```sqlSELECT bookings.device_id,bookings.user_id,FROM public.productsINNER JOIN public.bookings ON public.products.device_id = bookings.device_idWHERE (public.products.price_info->>'price')::numeric >= 500AND (public.products.storage_info->>'category') = 'SSD'```
```

> 吴恩达提示词课程: https://learn.deeplearning.ai/courses/chatgpt-prompt-eng
>
> 视觉模型提示词: https://learn.deeplearning.ai/courses/prompt-engineering-for-vision-models

关注公众号，交流更多 AI 见闻和思考

您的每一个评论、点赞、转发，对我都很重要~