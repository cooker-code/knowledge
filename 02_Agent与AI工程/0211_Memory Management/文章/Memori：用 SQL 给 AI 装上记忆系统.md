---
title: Memori：用 SQL 给 AI 装上记忆系统
author: 异或Lambda
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYyNDI2MjgzMA==&mid=2247483836&idx=1&sn=c8726bc99ef16208b2c3aa86c71507df&chksm=f1d79621a5acec0acb53be25e392793ef0302fdf6875e501c5e88dc29923cc3fcfb433cfd338&mpshare=1&scene=24&srcid=1119H4nkMkJWBQqsmwfqsQ5Q&sharer_shareinfo=099ff52bcf58cc20ee20ee9129c08ab0&sharer_shareinfo_first=099ff52bcf58cc20ee20ee9129c08ab0#rd
---

**你有没有遇到过这种情况？** 上午跟 AI 助手说"我在做 FastAPI 项目"，下午再问"怎么加认证"，它却完全不记得你用的什么技术栈。每次都得把背景重新说一遍，就像在跟一个失忆症患者对话。

现在有个开源项目解决了这个问题，而且只需要一行代码——用的还是你最熟悉的 SQLite。

## AI 为什么总"失忆"

传统的 LLM 每次对话都是独立的，不会记住之前说过什么。要让 AI 有记忆，业内常用的方案是向量数据库，比如 Pinecone、Weaviate 这些。

但问题来了：

* 每个月几十美元的订阅费
* 数据存在别人的服务器上
* 换个工具，之前的记忆全丢了

**Memori 给出了一个更直接的答案：用 SQL 数据库存储 AI 的记忆。**

## 一行代码搞定

集成 Memori 简单到让人意外：

```
from memori import Memorifrom openai import OpenAI  
memori = Memori(conscious_ingest=True)memori.enable()  # 就这一行  
client = OpenAI()# 之后所有对话自动拥有记忆能力
```

装上 Memori 后，AI 会自动记住你说过的话，下次对话时主动调用相关记忆。不需要改业务代码，不需要学新概念。

## 它是怎么工作的

Memori 采用了**拦截器模式**，在你调用 LLM 的前后自动处理记忆：

**调用前**：从数据库里检索相关的历史记忆，注入到当前对话  
**调用后**：提取对话中的实体、关系、事件，结构化存储到数据库

整个流程是这样的：

```
你的代码 → Memori 拦截 → 注入记忆 → OpenAI → 提取知识 → 存入 SQL
```

### 双模记忆机制

Memori 提供了两种记忆模式，可以单独用也可以组合：

**Conscious Mode（意识模式）**  
一次性注入工作记忆，适合短期任务追踪

**Auto Mode（自动模式）**  
每次查询动态搜索相关记忆，适合长期知识积累

组合使用就像人的短期记忆和长期记忆一样，该记住的记住，该遗忘的遗忘。

## 核心优势

### 用你已有的数据库

Memori 支持主流 SQL 数据库：

* SQLite：单文件，零配置，开发测试首选
* PostgreSQL / MySQL：生产环境部署
* Neon / Supabase：云原生托管方案

```
memori = Memori(    database_connect="postgresql://user:pass@localhost/memori")
```

### 兼容所有主流 LLM

通过 LiteLLM 的回调系统，Memori 可以无缝接入：

* OpenAI、Anthropic 原生支持
* LangChain 通过 LiteLLM 集成
* 100+ 模型即插即用

### 记忆是透明的

不同于向量数据库的黑盒，SQL 记忆完全可查询：

```
SELECT * FROM memories WHERE entity = 'FastAPI'   AND namespace = 'my_project';
```

你可以用任何 SQL 工具查看、分析、导出 AI 的记忆内容。数据在你自己手里，想怎么用就怎么用。

## 实际应用场景

### 场景一：代码助手

```
# 第一次对话"我在用 FastAPI + PostgreSQL 做 API 项目"  
# 一周后直接问"帮我优化数据库查询性能"# Memori 自动注入：用户项目用的 FastAPI + PostgreSQL
```

### 场景二：多智能体协作

多个 AI Agent 共享同一个 SQL 记忆库：

* Agent A 负责需求分析，把用户需求记录下来
* Agent B 负责写代码，读取需求记忆
* Agent C 负责测试，了解完整的项目上下文

云栈社区的开发者可以用这个架构搭建团队级的 AI 协作系统。

### 场景三：企业级部署

* 数据存储在自己的 PostgreSQL 实例
* 符合企业数据合规要求
* 可审计、可备份、可随时迁移

## 成本对比

| 方案 | 月成本 | 数据主权 | 可迁移性 |
| --- | --- | --- | --- |
| Pinecone | $70+ | ❌ | ❌ |
| Weaviate Cloud | $50+ | ❌ | ⚠️ |
| **Memori + SQLite** | **$0** | **✅** | **✅** |
| **Memori + 自建 PG** | **已有成本** | **✅** | **✅** |

成本能降低 80-90%，而且完全没有供应商锁定。

## 快速上手

安装：

```
pip install memorisdk
```

最简单的用法：

```
from memori import Memorifrom openai import OpenAI  
memori = Memori(conscious_ingest=True)memori.enable()  
client = OpenAI()response = client.chat.completions.create(    model="gpt-4o-mini",    messages=[{"role": "user", "content": "你的问题"}])
```

Memori 会在当前目录自动创建 `memori.db`（SQLite 文件），开箱即用。

## 生产环境配置

推荐用环境变量管理配置：

```
export MEMORI_DATABASE__CONNECTION_STRING="postgresql://..."export MEMORI_AGENTS__OPENAI_API_KEY="sk-..."export MEMORI_MEMORY__NAMESPACE="production"
```

代码里直接自动加载：

```
from memori import Memori, ConfigManager  
config = ConfigManager()config.auto_load()  # 自动读取环境变量  
memori = Memori()memori.enable()
```

## 项目现状

* **GitHub Star**：4.9k（快速增长中）
* **开源协议**：MIT
* **生产就绪**：已有企业在实际使用
* **社区活跃**：Discord 有日常技术答疑

## 为什么值得关注

Memori 的价值不在于技术有多炫，而在于它足够务实：

✅ 用成熟的 SQL 技术解决 AI 记忆问题  
✅ 一行代码集成，学习成本几乎为零  
✅ 数据完全自主可控，成本接近零  
✅ 可查询、可审计、可随时迁移

AI 的记忆不应该是黑盒，也不应该被供应商绑架。这个项目给出了一个清晰、实用的解决方案。

---

**关注《异或Lambda》，发现更多改造世界的开源项目**

🔗 **项目地址**：`GibsonAI/Memori`

📖 **官方网站**：`memorilabs.ai`

💬 **AI人工智能课程**：`https://yunpan.plus/f/29-1`

标签：#Memori #Github #AI记忆引擎 #开源项目 #LLM #SQL数据库 #AI工程化

原文：https://yunpan.plus/t/622-1-1