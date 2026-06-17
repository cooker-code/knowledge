---
title: DuckDB Extensions：让本地分析更强大的秘密武器
author: 新语数据故事汇
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg2NzgzMjU5MA==&mid=2247492941&idx=1&sn=e7a1fa444c79e3d99353b9182cd9a974&chksm=cf364336f2f115bdea5c2b897913288e1025b4ce57292210a678678a8e06ee8c7f6b0306e6be&mpshare=1&scene=24&srcid=1117CExKEGIdtk0TEnkb9Z9W&sharer_shareinfo=9c009cc745f15573a3ee51d98690d0f9&sharer_shareinfo_first=9c009cc745f15573a3ee51d98690d0f9#rd
---

DuckDB 这几年越来越火。轻量、嵌入式、零运维，却能跑出媲美大仓的分析速度。但大多数人只用了它的一半实力 —— 因为忽略了 **Extensions（扩展）**。

这些“小插件”能让 DuckDB 直接访问云端文件、解析 JSON、做地理分析、全文检索，甚至跨库查询 SQLite 或 Postgres。无需驱动、无需服务进程，**一条 SQL 全搞定。**

## DuckDB 扩展是什么？

DuckDB 的核心是一个极简的 OLAP 引擎。Extensions 是可选加载的模块，用来增加数据源、数据类型或功能。

在探索阶段可以开启 `SET autoinstall_known_extensions = true;`，让 DuckDB 自动安装常用扩展；但在生产脚本中，应显式使用 `INSTALL` / `LOAD`，确保执行过程可控且可重复。

## 1️⃣ HTTPFS — 云端文件本地查

直接读取 S3、GCS 或 HTTPS 上的 Parquet / CSV。

> 💡 场景：不下载、不同步，本地 SQL 直连云对象。

## 2️⃣ JSON — 解析嵌套结构不再痛苦

一行 SQL 搞定 API 日志和事件流。无需 Python 预处理，也不用“flatten” 半天。

## 3️⃣ Spatial — 空间计算像查表一样

地理位置、服务区、配送距离……都能直接算。

## 4️⃣ FTS — 全文检索内置版

快速搜索产品标题、问题描述或备注字段。不替代搜索引擎，但在分析中非常实用。

## 5️⃣ SQLite Scanner — 直接查本地 App 数据

很多桌面工具或脚本都把数据存在 SQLite。DuckDB 可以直接“读”这些数据库，零导出。

## 6️⃣ Postgres Scanner — 联邦查询神器

无需 ETL，把 Postgres 表直接拉进 DuckDB 做 join。非常适合小规模审计、回溯或快速对比分析。

## 7️⃣ Excel — 不再手动“导出 CSV”

直接读 Excel 文件，保留数据类型更准确。适合做快速分析或格式整理。

## 💡 小结：让你的笔记本电脑变成“小仓库”

DuckDB 的魅力在于**减少摩擦**。扩展模块则是让这份体验更完整的秘密武器。开启几个合适的扩展，你的笔记本电脑瞬间变成一个：

> “能查云、能查库、能查表，还能查地理”的小型数仓。