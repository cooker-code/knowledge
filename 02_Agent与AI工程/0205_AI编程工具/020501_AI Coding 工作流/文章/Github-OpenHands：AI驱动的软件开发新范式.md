---
title: Github-OpenHands：AI驱动的软件开发新范式
author: Zzz小生
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk2NDQ3ODc4Mw==&mid=2247484757&idx=1&sn=f92e4201449401fd72359b22e118b890&chksm=c500a141e369aa23cded59eb20cef7eaa5fa0695b91460ee117efc3632264ca87fbef5f0464e&mpshare=1&scene=24&srcid=0106jvskNTDWLWcSqFFKFpYb&sharer_shareinfo=49b9624343b4d67b8aa7fc5682e08f0e&sharer_shareinfo_first=49b9624343b4d67b8aa7fc5682e08f0e#rd
---

Github-OpenHands：AI驱动的软件开发新范式

```
https://github.com/OpenHands/OpenHands
```

## 项目概述

OpenHands 是一个专注于 **AI驱动开发** 的开源社区和平台。其核心目标是利用人工智能技术，特别是大型语言模型（LLM），来辅助和自动化软件开发过程。项目提供了从本地到云端、从命令行到图形界面的多种工具，旨在成为开发者的“AI助手”，提升编码效率和软件质量。

## 主要功能与目的

OpenHands 旨在构建一个完整的AI驱动开发生态系统，主要功能包括：

1. **智能代码生成与补全**：理解开发者的意图，自动生成、修改或重构代码。
2. **多模态交互**：支持通过CLI、本地GUI和云端Web应用等多种方式与AI代理交互。
3. **可组合的代理框架**：提供SDK，允许开发者自定义和编排具有不同技能的AI代理。
4. **企业级集成与协作**：支持与Slack、Jira、Linear等工具的集成，并提供多用户、权限管理和协作功能。
5. **性能评估**：内置评估基础设施，在SWE-bench基准测试中取得了72.8的高分，证明了其解决真实世界软件工程问题的能力。

## 核心技术栈

* **后端/核心**：Python (使用Poetry进行依赖管理)
* **前端**：React (用于本地GUI和Web界面)
* **容器化**：Docker, Docker Compose
* **部署**：Kubernetes (用于企业版)
* **构建工具**：Makefile
* **AI模型支持**：兼容Claude、GPT等多种主流LLM

## 项目结构概览

项目采用模块化设计，主要目录结构如下：

```
├── .devcontainer/          # 开发容器配置  
├── .github/               # GitHub工作流和模板  
├── .openhands/            # 项目特定配置  
├── containers/            # Docker容器定义  
├── enterprise/            # **企业版源代码（需商业许可）**  
├── evaluation/            # 评估和基准测试相关代码  
├── frontend/              # 前端React应用  
├── kind/                  # Kubernetes in Docker配置  
├── openhands/             # **核心Python SDK和库**  
├── openhands-cli/         # 命令行界面  
├── openhands-ui/          # 用户界面相关组件  
├── scripts/               # 构建和实用脚本  
├── skills/                # AI代理的技能定义  
├── tests/                 # 测试套件  
├── third_party/           # 第三方依赖或代码  
├── config.template.toml   # 配置文件模板  
├── docker-compose.yml     # 本地开发环境编排  
├── pyproject.toml         # Python项目配置和依赖  
└── poetry.lock            # 精确的Python依赖锁文件
```

## 核心组件与使用指南

OpenHands 提供了多种使用方式，满足不同用户的需求：

1. **OpenHands Software Agent SDK (核心引擎)**

* **定位**：可组合的Python库，包含所有智能体技术。
* **使用**：开发者可以在代码中定义AI代理，在本地运行或扩展到云端数千个代理。
* **指南**：查看SDK文档或源代码。

2. **OpenHands CLI (最易上手)**

* **定位**：命令行工具，体验类似于Claude Code或Codex。
* **使用**：在终端中直接与AI代理交互，支持多种LLM后端。
* **指南**：查看CLI文档或源代码。

3. **OpenHands Local GUI (本地图形界面)**

* **定位**：在笔记本电脑上运行代理的本地应用，提供REST API和单页React应用。
* **使用**：提供类似Devin或Jules的交互体验。
* **指南**：查看本地设置文档，源代码位于本仓库的`frontend/`和`openhands-ui/`目录。

4. **OpenHands Cloud (云端服务)**

* **定位**：托管在基础设施上的OpenHands GUI部署。
* **使用**：注册即可获得10美元免费额度，体验企业级功能。
* **访问**：app.all-hands.dev

5. **OpenHands Enterprise (企业自托管)**

* **定位**：供大型企业在自有VPC中通过Kubernetes自托管OpenHands Cloud。
* **许可**：`enterprise/`目录下的代码为**源码可用**，但生产使用超过一个月需要购买商业许可。
* **了解更多**：openhands.dev/enterprise

**快速开始**：对于大多数用户，推荐从CLI或Local GUI开始。使用Docker Compose可以快速搭建本地开发环境。

## 潜在应用场景

* **个人开发者**：自动化日常编码任务，如代码生成、调试、编写测试和文档。
* **初创团队**：快速原型开发，代码审查辅助，减少初级开发者的学习曲线。
* **大型企业**：集成到现有DevOps流程中，实现自动化问题排查、代码库维护和跨团队协作。
* **教育领域**：作为编程教学助手，为学生提供实时、个性化的编码指导。
* **开源项目维护**：自动化处理重复性issue、生成发布说明和进行依赖更新。

## 核心特点与创新点

1. **高性能基准**：在权威的SWE-bench基准测试中达到\*\*72.8%\*\*的通过率，表明其解决复杂、真实世界软件工程问题的强大能力。
2. **混合许可模式**：核心部分（`openhands`， `agent-server` Docker镜像）采用**宽松的MIT许可证**，鼓励社区贡献和广泛采用；企业功能采用**源码可用**的商业许可，确保项目可持续发展。
3. **完整的开发生态**：并非单一工具，而是提供了从底层SDK、CLI、GUI到云端服务的全栈解决方案。
4. **强大的可扩展性**：SDK允许深度定制AI代理和技能，支持从单机扩展到云端大规模部署。
5. **活跃的社区与协作**：拥有详细的贡献指南、行为准则和社区文档，鼓励用户加入Slack进行交流。
6. **研究与实践结合**：项目附有技术报告，并开源了评估框架和心智理论模块，体现了深厚的研究背景。

## 总结

OpenHands 是一个雄心勃勃且执行度很高的项目，它试图将AI驱动开发从概念和简单的代码补全，推进到一个系统化、可扩展、可用于生产环境的平台。其出色的基准测试成绩、清晰的架构设计以及兼顾开源与商业化的策略，使其在众多AI编程助手项目中脱颖而出，有望成为未来软件开发工作流中的重要组成部分。无论是想体验AI编程的开发者，还是寻求将AI集成到企业流程中的团队，OpenHands都提供了一个值得深入探索的起点。