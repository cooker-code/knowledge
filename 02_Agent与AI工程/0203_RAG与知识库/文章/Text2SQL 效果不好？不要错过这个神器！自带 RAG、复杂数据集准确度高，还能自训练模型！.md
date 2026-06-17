---
title: Text2SQL 效果不好？不要错过这个神器！自带 RAG、复杂数据集准确度高，还能自训练模型！
author: AI真好玩
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247484420&idx=1&sn=7d93c845bb3e68e79efdfa67ac1afffc&chksm=9051f8e1a72671f7500dfa77d22cfd0618fb25c933531fc883f15cfb7c1f21f86ba2d182facf&mpshare=1&scene=24&srcid=0603zIyGCi28lm5pVvTeO5kg&sharer_shareinfo=805eebb5fe52884552ebd39ffa1dde0f&sharer_shareinfo_first=805eebb5fe52884552ebd39ffa1dde0f#rd
---

Vanna[1] 是一个开源 Python RAG（检索增强生成）框架，用于 **SQL** 生成和相关功能。

## 近期热文

* [超强 OCR 新秀：支持 90 多种语言，性能超越云服务！](https://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247484310&idx=1&sn=9d1dd6a5d71fe482e4c46b3e7f79df27&scene=21#wechat_redirect)
* [当 AI 遇上爬虫：让数据提取变得前所未有的简单！](https://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247484004&idx=1&sn=a726b24672ecd7196a26f75e669571a4&scene=21#wechat_redirect)
* [2024 年最完整的 AI Agents 清单来了，涉及 13 个领域，上百个 Agents！](https://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247484029&idx=1&sn=0f4372f3221a913cbdd22db53cfe9530&scene=21#wechat_redirect)
* [7.8K Star RAG 引擎：基于深度文档理解，最大程度降低幻觉、无限上下文快速完成 “大海捞针” 测试！](https://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247484164&idx=1&sn=76649f7c04e7d611107f0a0680556381&scene=21#wechat_redirect)

## Vanna 主要特点

* **开源**：Vanna Python 软件包和各种前端集成都是开源的。您可以在自己的基础设施上运行 Vanna。
* **安全**：您的数据库内容永远不会发送到 LLM，除非您启用了该功能。元数据存储层只能看到模式、文档和查询。
* **易集成**：支持 Slackbot、Web App、Streamlit App 等多种集成方式。
* **自学习**：随着您更多地使用 Vanna，您的模型会随着我们增加您的训练数据而不断改进。
* **复杂数据集的高准确性**：Vanna 的能力与您提供的训练数据相关。更多的训练数据意味着大型复杂数据集的准确性更高。
* **支持多种数据库**：开箱即用，支持 `Snowflake`、`BigQuer` 和 `Postgres` 等数据库。还支持你自定义连接器来连接任何数据库。

## Vanna 工作流程

Vanna 只需两个简单的步骤：在数据上训练 **RAG 模型**，然后提出问题，这些问题将返回 SQL 查询，这些查询可以设置为在数据库中自动运行。

1. 基于你的数据训练 RAG 模型
2. 提问题

如果您不知道 RAG 是什么，也不要担心。您不需要知道它在幕后是如何工作的就可以使用它。您只需要知道您 “训练” 了一个模型，该模型存储一些元数据，然后用它来 “提出” 问题。

## RAG vs Fine-Tuning

### RAG

* 可跨 LLM 移植
* 如果训练数据过时，可轻松删除
* 运行成本比微调（Fine-Tuning）低得多
* 面向未来，如果出现了更好的 LLM，你只需更换即可

### Fine-Tuning

* 起步慢
* 训练和运行成本高
* 如果您需要在提示符中尽量减少 token 消耗，它是不错的选择

## Vanna 快速上手

### 前置条件

首先，需要根据 Vanna 文档[2] 配置数据库和使用的大语言模型。

### 安装 vanna

```
pip install vanna
```

### 使用 vanna

```
# The import statement will vary depending on your LLM and vector database. This is an example for OpenAI + ChromaDB  
  
from vanna.openai.openai_chat import OpenAI_Chat  
from vanna.chromadb.chromadb_vector import ChromaDB_VectorStore  
  
class MyVanna(ChromaDB_VectorStore, OpenAI_Chat):  
    def __init__(self, config=None):  
        ChromaDB_VectorStore.__init__(self, config=config)  
        OpenAI_Chat.__init__(self, config=config)  
  
vn = MyVanna(config={'api_key': 'sk-...', 'model': 'gpt-4-...'})  
  
# See the documentation for other options
```

## Vanna 模型训练

### 使用 DDL 语句训练

DDL 语句包含有关数据库中的表名、列、数据类型和关系的信息。

```
vn.train(ddl="""  
    CREATE TABLE IF NOT EXISTS my-table (  
        id INT PRIMARY KEY,  
        name VARCHAR(100),  
        age INT  
    )  
""")
```

### 使用文档训练

有些时候，您可能需要添加有关业务术语或定义的文档。

```
vn.train(documentation="Our business defines XYZ as ...")
```

### 使用 SQL 语句训练

您还可以将 SQL 查询添加到训练数据中。如果您已经有一些查询，这会很有用。您只需从编辑器中复制并粘贴这些内容即可开始生成新的 SQL。

```
vn.train(sql="SELECT name, age FROM my-table WHERE name = 'John Doe'")
```

## Vanna 生成 SQL

你可以通过调用 `ask` 方法传入 Prompt 文本：

```
vn.ask("What are the top 10 customers by sales?")
```

之后，您将获得对应的 SQL 语句：

```
SELECT c.c_name as customer_name,  
        sum(l.l_extendedprice * (1 - l.l_discount)) as total_sales  
FROM   snowflake_sample_data.tpch_sf1.lineitem l join snowflake_sample_data.tpch_sf1.orders o  
        ON l.l_orderkey = o.o_orderkey join snowflake_sample_data.tpch_sf1.customer c  
        ON o.o_custkey = c.c_custkey  
GROUP BY customer_name  
ORDER BY total_sales desc limit 10;
```

如果您已连接到 Vanna 提供的测试数据库，您将获得以下表格：

Vanna 还会自动生成绘图：

> https://github.com/vanna-ai/vanna

## 往期文章

* [用上 AI 几分钟轻松搞定深度行业报告！](https://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247484393&idx=1&sn=8d7d80d5df2c024217519f89f30440c7&scene=21#wechat_redirect)
* [超强 OCR 新秀：支持 90 多种语言，性能超越云服务！](https://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247484310&idx=1&sn=9d1dd6a5d71fe482e4c46b3e7f79df27&scene=21#wechat_redirect)
* [3 款强大的开源低代码 LLM 编排工具，可视化定制专属 AI Agent 和 AI 工作流！](https://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247484132&idx=1&sn=9d7197f5a1fd0cbf6e75b1d06bc086dd&scene=21#wechat_redirect)
* [25.4K Star 低代码LLM编排工具：终于支持 Multi Agent，内置 5 大 Multi Agent 开箱即用！](https://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247484201&idx=1&sn=da33fdcdd0e440bda438cf2ac15f68d8&scene=21#wechat_redirect)
* [Kimi+麦肯锡，5 秒摸透一个行业！](https://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247483932&idx=1&sn=f031c79afb9e8b2905b176650d10add1&scene=21#wechat_redirect)
* [Kimi 10 秒生成流程图，别再手动绘图了！](https://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247483922&idx=1&sn=4c2f114087b45bcad854fb51eeeee28b&scene=21#wechat_redirect)
* [万字长文秒变精华！Kimi 的超强提示词秘籍](https://mp.weixin.qq.com/s?__biz=MzA5NDMwMTU3OA==&mid=2247483953&idx=1&sn=61d881b1a7c5823a3e948806904786f5&scene=21#wechat_redirect)

欢迎您与我交流 AI 技术/工具

关注 AI 真好玩，带你玩转各类 AI 工具，掌控数字未来！

如果这篇文章对您有所帮助，请点赞、关注，并分享给您的朋友。感谢您的支持！

参考资料

[1] 

Vanna: *https://github.com/vanna-ai/vanna*

[2] 

Vanna 文档: *https://vanna.ai/docs/*