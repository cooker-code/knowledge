---
title: AutoDocs：一键生成代码库文档，智能解析依赖关系
author: Halo咯咯
date:
url: https://mp.weixin.qq.com/s?__biz=MzI0NTg0Njk1OQ==&mid=2247493596&idx=2&sn=64e68bfc507a74204dc4fce4c95256b7&chksm=e8224fd092b3a4d610aafb3adc2a40298fdbdaeeebc482fb4757ed3c362490f58333eb78b452&mpshare=1&scene=24&srcid=09120vqcHvt9vBhz8YyB5jQS&sharer_shareinfo=117d48769f2d854f6648891a69a12d4d&sharer_shareinfo_first=117d48769f2d854f6648891a69a12d4d#rd
---
> 已吸收至：[[09_电脑工具/0902_文档与知识工具/090209_代码知识文档/090209_核心知识点/代码知识文档生成边界|代码知识文档生成边界]]


**AutoDocs** 是一个由 Sita 提供的自动化文档生成工具，旨在为代码库生成和维护文档，同时提供依赖感知的搜索功能，帮助工具更好地理解代码库及其规范。

* **核心功能**：

+ 使用 **tree-sitter**（AST 解析）和 **SCIP**（符号解析）解析代码库。
+ 构建代码依赖图（文件、定义、调用、导入）并按依赖顺序排序。
+ 遍历依赖图以生成依赖感知的仓库级文档和摘要。
+ 提供 **FastAPI** 后端用于数据摄取和搜索，以及 **Next.js** Web UI 用于聊天和探索。
+ 内置 **MCP** 服务器，使编码代理可以通过 HTTP 深度搜索代码。

* **安装与配置**：

+ 需要安装 **pnpm**（10+）、**uv**（Python 包管理器）和 **Docker + Docker Compose**。
+ 可选配置 GitHub 个人访问令牌（PAT），用于访问私有仓库或提高 API 调用限额。
+ 提供了详细的环境配置和快速启动指南，包括数据库迁移和本地运行步骤。

* **使用与更新**：

+ 提供 Web UI 和 API 服务，分别位于 `http://localhost:3000` 和 `http://localhost:8000`。
+ 支持通过 MCP 服务器进行代码库范围内的问答，查询 AutoDocs 生成的分析数据库。
+ 提供了开发工作流指南，方便开发者贡献代码。

* **项目布局**：

+ 包括 `ingestion/`（Python FastAPI 服务、AST 解析器、图构建器、嵌入和搜索）。
+ `webview/`（Next.js 应用程序和共享 TS 包）。
+ `docker-compose.yml`（本地 Postgres、API 和 Web 服务）。
+ `tools/`（辅助脚本）。

 

**参考：**

1. https://github.com/TrySita/AutoDocs

---

点个**分****享、****点赞**与**在看**，你最好看~
