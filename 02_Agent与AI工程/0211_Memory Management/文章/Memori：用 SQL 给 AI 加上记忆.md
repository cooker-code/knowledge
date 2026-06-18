---
title: Memori：用 SQL 给 AI 加上记忆
author: AI工程化
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5MTIxNTY4MQ==&mid=2461154504&idx=1&sn=ed4f364714d0a5f29f8c1bb4d84cb007&chksm=869b2ffaa3b7d8c1a7e77ab0216430b285782b2e9b30014c08cb72cd07a759eecd32536c3071&mpshare=1&scene=24&srcid=0922oq783Ci7IwFV5ZyQqAvT&sharer_shareinfo=c43888f6e74f2a3ad87485ca381d9372&sharer_shareinfo_first=c43888f6e74f2a3ad87485ca381d9372#rd
---

关于AI记忆的项目很多，之前我们介绍过Mem0之类的产品([mem0推出王炸mcp工具OpenMemory，打造用户私有、跨应用的共享记忆层](https://mp.weixin.qq.com/s?__biz=MzA5MTIxNTY4MQ==&mid=2461152184&idx=1&sn=c7cac29807a7328de6f44f19c7d52f9a&scene=21#wechat_redirect))。今天我们来介绍一个新的项目 Memori，它在技术选型上做了个有趣的选择：当大家都在追捧向量数据库和图数据库时，它选择了回归 SQL。

Memori 是 Gibson 团队开发的开源记忆引擎，专门给大语言模型和 AI Agent 加上持久记忆。简单说，就是让 AI 能记住你们之前的对话，不用每次都从头开始。

目前主流的 AI 记忆方案各有各的问题：

**提示词填充** 最直接，把历史对话不断塞进上下文。短对话还行，一旦聊久了，token 数量和成本都会失控。

**向量数据库** 是当前主流，用 Pinecone、Weaviate 存储语义嵌入。但语义检索本身就是模糊的，经常召回一堆相关但不精确的内容，而且结构化信息容易丢失。

**图数据库** 在表达实体关系上确实强大，做推理很合适。但构建和维护成本高，扩展性也是个问题。

而 Memori 的方案是用关系型数据库存储记忆，理由很简单：SQL 数据库运行了几十年，成熟可靠，用 JOIN 和索引做精确检索正是它的强项。对于需要结构化存储、精确召回的记忆场景，SQL 可能确实更合适。

这个项目最有意思的地方在于它的双模式设计：

**Conscious Mode（意识模式）** 模拟人类的短期记忆。启动时，AI 会分析历史对话，把最重要的 5-10 段内容提取出来作为工作记忆。比如你的名字、正在做的项目、常用技术栈这些信息，会被优先记住并在对话开始时一次性注入。

**Auto Mode（自动模式）** 则是动态搜索整个记忆库。每次你提问时，AI 都会智能分析需要什么背景信息，然后从数据库里检索相关记忆。这个模式更适合需要大量历史信息的复杂对话。

使用起来很简单：

```
from memori import Memori  
from openai import OpenAI  
  
# 初始化  
memori = Memori(conscious_ingest=True)  
memori.enable()  
  
# 正常使用 OpenAI  
client = OpenAI()  
response = client.chat.completions.create(  
    model="gpt-4o-mini",  
    messages=[{"role": "user", "content": "帮我优化代码"}]  
)  
# AI 会自动记住你之前提到的项目信息
```

从功能上看，Memori 支持 SQLite、PostgreSQL 和 MySQL，用 Pydantic 做结构化数据验证，能自动提取对话中的实体（人名、技术、项目等）并分类存储。它可以跟任何 LLM 库配合使用，已经集成了 LangChain、CrewAI、AgentOps 等主流框架。

比较实用的是多用户隔离功能。通过 namespace 参数，每个用户的记忆可以完全独立，适合做 SaaS 应用。项目还提供了 FastAPI 的多用户示例代码。

当然，这不意味着向量数据库没用。语义搜索在某些场景下不可替代。但 Memori 的实践提醒我们：解决问题不一定要用最新的技术，有时候最合适的答案就在那些被验证过的"老技术"里。

如果你在开发 AI 应用，特别是需要长期记忆的场景（客服机器人、个人助理、多智能体系统），这个工具值得一试。毕竟，一个能记住上下文的 AI，比每次都要重新介绍自己的 AI 实用多了。

项目地址：https://github.com/GibsonAI/memori

关注公众号回复“进群”入群讨论。