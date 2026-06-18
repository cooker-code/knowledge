---
title: Dory：一个AI原生的SQL数据工作台
author: SQL编程思想
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMDI3OTgyNw==&mid=2247492282&idx=1&sn=6bb930fbf6b13192e56417ffd01cb92d&chksm=c3549f47bb16cd1e9b2f57a8cbc41dc8660124b8f86afce6e11227f08e17fc42c76bd79102d7&mpshare=1&scene=24&srcid=0424mYkWKnknvI0Rd9VoNpc5&sharer_shareinfo=624f2f84ace5a8eca76139f1a898e67d&sharer_shareinfo_first=624f2f84ace5a8eca76139f1a898e67d#rd
---

Dory 是一个面向现代数据库的 AI 原生数据工作台，集成了 SQL 编辑、AI Copilot、数据库对话、数据浏览与运维能力，旨在帮助工程师和数据分析师更高效地理解和使用数据库。

Dory 项目采用 TypeScript 语言开发，遵循 Apache 2.0 开源协议，代码托管在 GitHub：

https://github.com/dorylab/dory 

## 功能特性

* • **跨平台**：Dory 支持 Windows、macOS、Docker 部署。
* • **多种数据库**：包括 ClickHouse、PostgreSQL、Neon、MySQL、MariaDB、SQLite，支持 SSL/TSL 以及 SSH Tunnel 安全连接。

* • **SQL Copilot**：AI 驱动的智能助手，基于实时数据库结构和查询上下文提供自然语言生成 SQL 语句，修复或者重写查询，解释查询逻辑。

* • **智能编辑器**：可以提供基于真实数据库结构的自动补全和代码提示，支持复杂的多表连接和子查询；提供多标签、多结果集查询体验；支持保存查询、上下文与结果。

* • **AI 模型**：桌面版本内置了 OpenAI 模型，开源版本通过可配置环境变量支持 OpenAI、Anthropic、Google Gemini 等大语言模型。
* • **资源浏览器**：提供关于数据库、表、视图的结构信息、数据概览、语义注释与使用提示。

* • **ClickHouse 监控**：内置集成 ClickHouse 监控接口，提供基础运行态、查询与负载、慢查询于错误查询等指标，支持按照用户、数据库、查询类型、时间范围等维度过滤。

* • **ClickHouse 权限管理**：提供用户和角色管理功能，包括创建、编辑、删除用户，创建角色和授权操作；支持集群级别的权限操作。
* • **本地优先**：数据查询、个人设置与工作流都保存在本地，确保隐私与安全。

## 在线体验

Dory 提供了一个免费在线的体验环境，网址如下：

https://app.getdory.dev/ 

直接点击“Sign in as demo”开始：

## 下载安装

Dory 官方下载网址如下：

https://github.com/dorylab/dory/releases 

使用 Docker 进行部署的命令如下：

```
docker run -d --name dory \  
  -p 3000:3000 \  
  -e DS_SECRET_KEY="$(openssl rand -base64 32 | tr -d '\n')" \  
  -e BETTER_AUTH_SECRET="$(openssl rand -hex 32)" \  
  -e BETTER_AUTH_URL="http://localhost:3000" \  
  -e DORY_AI_PROVIDER=openai \  
  -e DORY_AI_MODEL=gpt-4o-mini \  
  -e DORY_AI_API_KEY=your_api_key_here \  
  -e DORY_AI_URL=https://api.openai.com/v1 \  
  -e NEXT_PUBLIC_REQUIRE_EMAIL_VERIFICATION=false \  
  dorylab/dory:latest
```

## 总结

Dory 是一个懂数据库结构的 AI 数据工作室，也是传统数据库客户端的现代化 AI 替代方案。