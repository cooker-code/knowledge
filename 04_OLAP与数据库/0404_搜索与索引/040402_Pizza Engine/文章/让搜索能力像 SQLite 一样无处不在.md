---
title: 让搜索能力像 SQLite 一样无处不在
author: 弹性搜索
date: MedclMedcl
url: https://mp.weixin.qq.com/s?__biz=MzI2NjM1MzkwMg==&mid=2247484097&idx=1&sn=1eaca875c5a0ac8b355e02a98d8e34b7&chksm=ebfb23b2c6b264e5ccf8794f943f9e3831e545662ec1dcfbbeff8c926e5b52f2c825fcbb8ece&mpshare=1&scene=24&srcid=0522lWwQUjRcF2eXXPsMw4Q9&sharer_shareinfo=0e772be61e207cd7464b48b3f06806b1&sharer_shareinfo_first=0e772be61e207cd7464b48b3f06806b1#rd
---

Pizza 已经陆陆续续做了几年了，借助 AI 的力量，迭代速度直线上升，核心引擎进一步成熟，今天介绍一下 Pizza Engine，同时回答几个网友的问题。

---

# - Pizza Engine 是什么？

Pizza Engine 是 Pizza 项目 (项目网址：https://pizza.rs) 的核心引擎，内部代号 FIRE（Fast Indexing and Retrieval Engine）。

它是一个基于 Rust 构建的轻量级实时搜索引擎，定位为：

面向 AI Native、Browser Native、Edge Native 场景的新一代 Search Runtime。

Pizza Engine 的核心目标：

* 极致轻量、高性能
* 可嵌入、支持 WASM
* 丰富的搜索能力
* 面向边缘与本地搜索场景
* 支持实时更新、原地更新
* 扩展性好
* Browser Native
* AI Native

和传统搜索引擎最大的不同在于：

Pizza Engine 不只是一个“搜索服务”，更像是一个“搜索运行时（Search Runtime）”。

它既可以运行在：

* Linux/macOS/Windows
* Server
* Edge Node
* Tauri/Electron
* Browser
* WebAssembly
* 甚至 #![no\_std] 环境

也可以作为一个 Rust Library 被直接嵌入到应用里面。

---

# Pizza Engine 的几个核心特性

## 1. Browser Native

Pizza 可以完整运行在浏览器里面。

不仅仅是简单的 JS 搜索，而是真正的：

* 倒排索引、丰富的查询
* BM25 & Ranking
* Vector Search
* Aggregation
* Geo Search
* Graph Search
* Full Featured Query Engine

全部运行在 WASM 中。

支持：

* OPFS 持久化
* HTTP Range 请求
* Segment Cache
* 离线索引、本地搜索

---

## 2. AI Native

Pizza Engine 从一开始就是按照 AI Retrieval 场景设计的。

支持：

* BM25
* Dense Vector
* Sparse Vector
* Hybrid Retrieval
* Streaming Query
* Real-time Indexing

非常适合：

* RAG
* AI Agent
* Long-term Memory
* Local AI
* Browser AI

---

## 3. Streaming Query Engine

Pizza 使用纯 Cursor + Streaming 的执行模型：

posting -> cursor -> streaming merge -> scoring

而不是传统的大量：

HashSet / Vec materialization

优势：

* 内存占用低
* 更适合 WASM
* 更适合边缘计算
* 更适合 AI Retrieval

---

## 4. Single-threaded, Share-nothing

Pizza 采用：One Engine Per Core 设计。

避免：锁竞争 跨线程同步 Shared State

特别适合：

* Browser
* WASM
* Edge Runtime
* 实时系统

---

# - Pizza Engine 浏览器版本适合什么场景？

Pizza Engine 最大的价值是：

* “把完整搜索引擎直接带到浏览器端。”
* 不依赖后端 ES 服务。
* 不依赖云端。

真正做到：Search Anywhere

---

# 典型场景

## 1. AI 本地知识库（最重要）

这是 Pizza Engine 最核心的方向。

例如：

* 本地 RAG
* 浏览器 AI 助手
* Local AI
* NotebookLM Local
* ChatGPT + Local Files

用户拖入：

* PDF
* Markdown
* 代码仓库
* 网页
* 文档

浏览器即可：

* 建索引
* Chunk
* Embedding
* Retrieval
* Search
* 无需服务器。

---

## 2. 企业知识库 / Wiki

例如：

* 内部 Wiki
* 离线文档
* 团队知识库
* Browser CRM
* Browser ERP

特点：

* 数据不出本地
* 支持离线
* 秒级搜索
* 部署成本低

---

## 3. 文档搜索

非常适合：

* Hugo
* Docusaurus
* VitePress
* API Docs
* SDK 文档

替代：

* Algolia
* Lunr.js
* Pagefind
* 但能力更强。

---

## 4. Browser AI Agent

未来 AI Agent：

* 长期记忆
* 本地上下文
* 浏览器内知识库

这些都需要 Local Retrieval Engine，而 Pizza Engine 很适合作为 AI Agent 的本地检索内核。

---

## 5. Tauri / Electron 应用

例如：

* AI 桌面助手
* IM 搜索
* 本地文件搜索
* IDE 搜索
* 邮件搜索
* Pizza Engine 可以直接嵌入应用内部。

> Coco AI 已集成 Pizza Engine 来进行本地轻量级查询。

---

## 6. Edge Search

Pizza Engine 很适合：

* Edge Computing
* CDN Search
* Browser Extension
* PWA
* Offline-first App

---

# - Pizza Engine 和 Elasticsearch / Lucene 的区别？

很多人容易误解：

Pizza Engine 是不是 “另一个 Elasticsearch”。

它们的设计目标完全不同。

---

Elasticsearch 定位： 分布式企业级搜索与分析平台。

适合：

* 大规模集群
* 日志分析
* 海量数据
* 企业后台
* 实时分析

特点：

* JVM
* 分布式
* 重服务架构
* 运维复杂
* 适合数据中心

---

Lucene 定位：JVM 世界的底层全文检索库。

Lucene 更像：

* Search Kernel
* Index Library

Elasticsearch/Solr 本质上都是基于 Lucene。

---

Pizza Engine 定位：面向 Browser / Edge / AI 的 Embedded Search Runtime。

核心特点：

| **特性** | **Pizza Engine** | **Elasticsearch** |
| --- | --- | --- |
| 运行环境 | Browser / WASM /   Edge / Embedded | Server / Cluster |
| 架构 | Embeddable Runtime | Distributed Service |
| 语言 | Rust | Java |
| 资源占用 | 极低 | 较高 |
| 实时性 | 极强 | 中等 |
| Browser 支持 | 原生支持 | 基本不支持 |
| WASM | 原生支持 | 不支持 |
| AI Retrieval | 核心方向 | 后期增强 |
| 部署方式 | Library | Cluster |
| 本地优先 | 是 | 否 |
| 离线运行 | 是 | 不适合 |
| `#![no_std]` | 支持 | 不支持 |

---

# Pizza Engine 更像什么？

如果一定要类比：

| **项目** | **定位** |
| --- | --- |
| SQLite | Embedded Database |
| DuckDB-WASM | Browser Analytics Engine |
| Lucene | JVM Search Kernel |
| Pizza Engine | Browser & AI Search Runtime |

---

# - Pizza 会开源么？

会。

Pizza 从一开始就是按照：

开放、可扩展、社区驱动的方向设计的。

> 持续完善中，各位客官请稍等，不过本文要是访问量过十万咱立马开源 :)

---

# 为什么选择开源？

因为搜索引擎本身就是基础设施。

我们希望：

* 更多开发者参与更多 AI 场景落地
* 更多 Browser Native 应用出现
* 推动 WASM Search 生态发展

---

# Pizza 想做什么？

我们并不只是想做：

“一个轻量版 Elasticsearch”

而是希望构建：下一代搜索基础设施

包括：

* Browser Search
* AI Retrieval
* Local-first Search
* Edge Search
* Embedded Search
* Agent Memory Runtime

---

# Pizza 的目标

一句话：

让搜索能力像 SQLite 一样无处不在。