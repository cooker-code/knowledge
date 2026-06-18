---
title: SQLBot：一款基于大语言模型和RAG的智能数据分析工具
author: SQL编程思想
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMDI3OTgyNw==&mid=2247489444&idx=1&sn=7ec61d304db389214314e1692ff213f2&chksm=c39f67a861918f9c24b2a0cb9cbd668faa03d6d67b8b44e6e9d35bf725ff47b370739acc4abc&mpshare=1&scene=24&srcid=0912062tsUdSyT9TZT8lasqy&sharer_shareinfo=aeac88a36be54a41c8c1b69ae8cc2b21&sharer_shareinfo_first=aeac88a36be54a41c8c1b69ae8cc2b21#rd
---

SQLBot 是一款基于大语言模型和 RAG 的智能问数系统（Text-to-SQL），可以实现数据的即问即答，快速提炼获取用户所需的数据信息及可视化图表，并且支持进一步开展智能分析。

SQLBot 项目由飞致云维护并开源，该企业开源的其他软件包括 1Panel 运维管理面板、JumpServer 堡垒机、Halo 建站工具、DataEase 可视化分析工具等。

## 系统架构

SQLBot 整体技术架构如下图所示：

用户提出问题之后，从数据库中获取表结构作为提示词发送给大语言模型，大模型将问题转换为 SQL 语句查询数据库中的数据，然后大模型基于结果和用户的需求整理生成表格和图表。

## 功能特性

* • **开箱即用**：只需简单配置大模型和数据源即可开启问数之旅。SQLBot 充分发挥大语言模型强大的自然语言理解和 SQL 生成能力，结合 RAG 技术提升生成精度，实现高质量的 Text-to-SQL 转换体验。

* • **提问分析**：用户聊天对话方式提问，大模型解析问题意图，结合所选数据源生成图表与分析。

* • **深度探索**：在获得基础图表结果后，进一步进行分析、解释、验证和预测，支持更强的业务决策支持。

* • **数据管理**：支持用户配置、管理多种类型的数据源（MySQL、SQL Server、Oracle、PostgreSQL、Excel、CSV）和数据表，支持按需配置和管理。

* • **看板搭建**：用户可通过拖拽、自定义布局、添加说明等方式，将零散的分析结果转化为结构化的数据看板，便于复用、协作和汇报。

* • **安全可控**：提供基于工作空间的资源隔离机制，支持细粒度的数据权限配置，确保用户在使用过程中拥有清晰的数据边界与权限控制能力，保障数据访问的安全与合规。

* • **易于集成**：支持多种集成方式，能够快速嵌入到 n8n、MaxKB、Dify、Coze 等 AI 应用开发平台，让各类应用快速拥有智能问数能力。

## 下载安装

支持多种安装方式，使用 Docker Compose 进行部署的命令如下：

```
# 创建目录  
mkdir sqlbot  
cd  sqlbot  
  
# 下载 docker-compose.yaml  
curl -o docker-compose.yaml https://raw.githubusercontent.com/dataease/SQLBot/main/docker-compose.yaml  
  
# 启动服务  
docker compose up -d
```

启动服务之后，在浏览器中打开以下地址：http://<主机IP>:8000/。

默认的用户名和密码分别是：admin 和 SQLBot@123456。

SQLBot 主界面导航栏包含四大核心模块：【智能问数】、【数据源】、【仪表板】和【设置】。

在使用智能问数和其他功能之前，还需要进行 AI 大模型和数据源的配置。官方文档提供了详细的介绍，配置过程也非常简单：

https://dataease.cn/sqlbot/v1/quick\_start/

## 总结

SQLBot 打破了数据与业务人员之间的技术壁垒，允许任何业务人员（如运营、市场、产品经理等）无需学习复杂的 SQL 语法和了解底层数据库结构，只需用自然语言提出问题，就能快速、准确地获取他们想要的数据和分析结果。