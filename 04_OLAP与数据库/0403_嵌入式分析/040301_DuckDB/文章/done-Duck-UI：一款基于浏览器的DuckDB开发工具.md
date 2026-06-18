> 已吸收至：[[04_OLAP与数据库/0403_嵌入式分析/040301_DuckDB/040301_核心知识点/DuckDB扩展机制与生态边界|DuckDB扩展机制与生态边界]]
---
title: Duck-UI：一款基于浏览器的DuckDB开发工具
author: SQL编程思想
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzMDI3OTgyNw==&mid=2247491321&idx=1&sn=14ab2335d3cc36aaf1cce1fa0b181846&chksm=c35e29c250bbdae4fa7b7880c5616c28f9e74bd1b49f4f466f4d88e9ecae049bf47344bb8484&mpshare=1&scene=24&srcid=1211U9vvuFCWlZNn3n2o3XOr&sharer_shareinfo=c0cadd534ade8ffb4d24349c06cae801&sharer_shareinfo_first=c0cadd534ade8ffb4d24349c06cae801#rd
---

Duck-UI 是一款基于浏览器的现代化 DuckDB 开发工具，提供强大的 SQL 编辑和数据分析功能。

Duck-UI 使用 TypeScript 语法开发，遵循 Apache 2.0 开源协议，代码托管在 GitHub：

https://github.com/ibero-data/duck-ui 

## 功能特性

* • **无服务器**：使用 DuckDB 提供的 WebAssembly 支持功能，完全基于浏览器，不需要配置数据库服务器。
* • **SQL 编辑器**：提供编写和运行 SQL 查询功能，支持语法高亮和自动补全；支持查询历史。
* • **数据导入**：可以导入 CSV、JSON、Parquet、Arrow 格式文件，支持本地存储和 URL 远程数据导入。

* • **数据探索**：支持浏览和管理数据库和数据表。
* • **绘制图表**：查询结果可以使用图表形式（例如柱状图、散点图、饼图、直方图等）进行展示，并且可以导出图像文件。

* • **远程连接**：可以连接到远程 DuckDB 服务器。
* • **持久存储**： 可以使用 OPFS 技术实现数据持久化存储。
* • **隐私优先**：所有的数据处理过程都在浏览器本地执行。

## 在线体验

提供了一个在线的体验环境，网址如下：

https://demo.duckui.com/ 

## 下载安装

Duck-UI 推荐使用 Docker 进行快速部署体验，命令如下：

```
docker run --name duck-ui -p 5522:5522 \
  ghcr.io/ibero-data/duck-ui:latest
```

启动服务之后，在浏览器中输入以下地址进行访问：

http://localhost:5522 

参考文档：https://duckui.com/getting-started.html