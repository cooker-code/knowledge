> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020303_RAG/020303_核心知识点/RAG生命周期与评估边界|RAG生命周期与评估边界]]
---
title: 腾讯开源 WeKnora：基于大模型的智能知识库，轻松部署全攻略
author: 星哥玩云
date:
url: https://mp.weixin.qq.com/s?__biz=MzU0NDc0MTAyMg==&mid=2247488011&idx=1&sn=509bc033405434d08a5b6061febc3f2e&chksm=facdd83d3071d9cb5e188dc7c9084889d7a2db1585f2bf4ab758535d87b28b607abb77e44f8c&mpshare=1&scene=24&srcid=0909nH73rqqi3BssHneG2sUY&sharer_shareinfo=986983495add08eeb6480699ac46b41e&sharer_shareinfo_first=986983495add08eeb6480699ac46b41e#rd
---

# 腾讯开源 WeKnora：基于大模型的智能知识库，轻松部署全攻略

## 前言

在企业知识管理、科研文献分析、技术支持、法律合规审查等场景中，传统的全文检索和关键词匹配已经无法满足复杂、多模态的文档理解需求。

腾讯近期开源的 **WeKnora** 框架，将大语言模型（LLM）与语义检索、智能推理深度融合，为结构复杂、内容异构的文档提供高质量的问答与分析能力。

本文将带你快速了解 WeKnora 的核心特性，并手把手演示如何在 **飞牛 NAS** 上通过 Docker 部署，让你的私有知识库秒变“智能问答专家”。

img

## WeKnora 项目介绍

WeKnora（维娜拉） 是一款基于大语言模型（LLM）的文档理解与语义检索框架，专为结构复杂、内容异构的文档场景而打造。

框架采用模块化架构，融合多模态预处理、语义向量索引、智能召回与大模型生成推理，构建起高效、可控的文档问答流程。核心检索流程基于 RAG（Retrieval-Augmented Generation） 机制，将上下文相关片段与语言模型结合，实现更高质量的语义回答。

官网： https://weknora.weixin.qq.com

## 核心特性

* • **🔍 精准理解**：支持 PDF、Word、图片等文档的结构化内容提取，统一构建语义视图
* • **🧠 智能推理**：借助大语言模型理解文档上下文与用户意图，支持精准问答与多轮对话
* • **🔧 灵活扩展**：从解析、嵌入、召回到生成全流程解耦，便于灵活集成与定制扩展
* • **⚡ 高效检索**：混合多种检索策略：关键词、向量、知识图谱
* • **🎯 简单易用**：直观的Web界面与标准API，零技术门槛快速上手
* • **🔒 安全可控**：支持本地化与私有云部署，数据完全自主可控

## 使用场景

img

## 功能模块能力

img

## 在飞牛 NAS 上部署 WeKnora

> 理论上，任何支持 Docker 的设备都可部署 WeKnora，这里以飞牛 NAS 为例。

确保本地已安装以下工具：

Docker、Docker Compose、Git

### 1. 启用 SSH 登录

在飞牛 OS 中开启 **SSH 登录** 功能，并切换到 `root` 用户：

```
sudo -i
```

### 2. 创建部署目录

```
mkdir -p /docker/docker && cd /docker/docker
```

### 3. 克隆项目

```
# 克隆主仓库
git clone https://github.com/Tencent/WeKnora.git
cd WeKnora
```

### 4. 配置环境变量

```
cp .env.example .env
# 编辑 .env 填写对应配置信息
```

> `.env.example` 文件中有详细注释，可根据实际情况修改。

### 5.启动服务

如果需要安装ollama则执行这个，我这边不执行，因为我已经有安装ollama了。

```
# 启动全部服务（含 Ollama 与后端容器）
./scripts/start_all.sh
# 或
make start-all
```

### 5.启动服务备选

```
# 启动服务
docker compose up -d
```

img

接下来就看网络的情况了。

### 6.停止服务

```
./scripts/start_all.sh --stop
# 或
make stop-all
```

### 服务访问地址

启动成功后，可访问以下地址：

* • Web UI：`http://localhost`
* • 后端 API：`http://localhost:8080`
* • 链路追踪（Jaeger）：`http://localhost:16686`

首次进入需配置：

首次访问会自动跳转到初始化配置页面，配置完成后会自动跳转到知识库页面。请按照页面提示信息完成模型的配置。

* • 大模型（本地 ollama 或远程 API）
* • Embedding 模型
* • Rerank 模型
* • 多模态解析
* • 文档分割策略

---

## 使用体验

1. 1. **上传文档**
   支持批量上传，系统会自动解析并生成知识库索引。
2. 2. **智能问答**
   输入问题，WeKnora 会结合知识库内容进行精准回答，并给出引用来源。
3. 3. **多轮对话**
   支持上下文关联的连续提问，适合深度分析与探索。

WeKnora 提供了一系列 RESTful API，用于创建和管理知识库、检索知识，以及进行基于知识的问答。本文档详细描述了这些 API 的使用方式。

## 如何查看日志？

```
# 查看 主服务 日志
docker exec -it WeKnora-app tail -f /var/log/WeKnora.log

# 查看 文档解析模块 日志
docker exec -it WeKnora-docreader tail -f /var/log/docreader.log
```

## 如何启动和停止服务？

```
# 启动服务
./scripts/start_all.sh

# 停止服务
./scripts/start_all.sh --stop

# 清空数据库
./scripts/start_all.sh --stop && make clean-db
```

## 应用场景

* • 企业内部知识管理
* • 科研文献分析
* • 产品技术支持
* • 法律合规审查
* • 医疗知识辅助

## 总结

WeKnora 作为腾讯开源的 **智能知识库框架**，在文档解析、语义检索、智能推理等方面表现出色。 结合飞牛 NAS（其他的服务器也可以） 的本地化部署能力，你可以轻松构建一个 **安全可控、功能强大** 的私有知识库系统。

如果你正在寻找一款 **可本地部署、支持多模态、基于大模型的知识库解决方案**，WeKnora 值得一试。

写文不易，如果你都看到了这里，请点个赞和在看，分享给更多的朋友；也别忘了关注星哥玩云！这里有满满的干货分享，还有轻松有趣的技术交流～点个赞、分享给身边的小伙伴，一起成长，一起玩转技术世界吧！ 😊