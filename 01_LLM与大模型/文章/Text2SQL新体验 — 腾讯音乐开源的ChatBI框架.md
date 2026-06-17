---
title: Text2SQL新体验 — 腾讯音乐开源的ChatBI框架
author: AI工程师笔记
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkwMDY3MDA1MA==&mid=2247484722&idx=1&sn=070c0b7e9a01a9e6b6659c444dab2bf2&chksm=c041323ff736bb291aecd361f280f1cf6056d12ac1fe79e143f4fd7e2b035784ba38e8bd74be&mpshare=1&scene=24&srcid=0703ptuC2KQGdYqeItQEjtU9&sharer_shareinfo=dbb9bdf8442238381fb861c43fc48771&sharer_shareinfo_first=dbb9bdf8442238381fb861c43fc48771#rd
---

# 腾讯音乐开源ChatBI框架

### 简介

SuperSonic 是一个开源的 BI 平台，它通过融合 Chat BI（基于大型语言模型 LLM）和 Headless BI（基于语义层），旨在提供更加高效和智能的数据分析体验。该平台的核心功能包括将自然语言查询转换为 SQL 语句，并通过合适的可视化图表展示查询结果。用户无需修改或复制数据，只需构建逻辑语义模型即可使用。SuperSonic 采用 Java SPI 机制，设计为可插拔的框架，便于扩展定制功能。

项目的动机在于大型语言模型的出现正在改变信息检索方式，引领了 Chat BI 的新范式。然而，这些新方法在实际应用中的可靠性尚有欠缺。与此同时，Headless BI 的新兴范式专注于构建统一的语义数据模型，并通过 API 提供数据语义。SuperSonic 项目通过结合这两种范式，增强了 Text2SQL 的能力，减少了幻觉和复杂度。

SuperSonic 具备开箱即用的特性，包括内置的 Chat BI 和 Headless BI 界面、基于规则的语义解析器、高级特征如文本输入联想、多轮对话和查询后问题推荐等，以及三级权限控制。此外，SuperSonic 的组件包括模型知识库、模式映射器、语义解析器、语义修正器、语义翻译器和问答插件，都是易于扩展的。

### 线上环境体验

访问**http://117.72.46.148:9080** 注册新用户体验. 请勿修改系统配置。官方每周末定期重启重置配置。

### 本地构建

SuperSonic自带样例的语义模型和问答对话，只需以下三步即可快速体验：

* • 从release page下载预先构建好的发行包
* • 运行 "assembly/bin/supersonic-daemon.sh start"启动standalone模式的Java服务
* • 在浏览器访问http://localhost:9080 开启探索

详细操作

参考WIKI

```
https://github.com/tencentmusic/supersonic/wiki
```