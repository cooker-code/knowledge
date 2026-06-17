---
title: 每周精品开源项目：Open WebUI
author: 架构师炼丹炉
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIwMjcwMTQzMA==&mid=2247485731&idx=1&sn=6defac959d8b1dfc1b15179a3a5530e6&chksm=971940532746b39713bfe182b5acbe113da80e8724bcea72e5de0eaf07653fade6a1a2a5f8b9&mpshare=1&scene=24&srcid=0116iDy4QCUnQCiZxmfMNmgE&sharer_shareinfo=92da0b438b5c1439806c25dbea6dfa90&sharer_shareinfo_first=92da0b438b5c1439806c25dbea6dfa90#rd
---

Open WebUI 是一个可扩展、功能丰富且用户友好的自托管 AI 平台，专为完全离线运行而设计。它支持各种 LLM 运行器，如 Ollama 和 OpenAI 兼容的 API，并内置用于 RAG 的推理引擎，使其成为一个强大的 AI 部署解决方案。

## Open WebUI 的主要功能

* 轻松设置：使用 Docker 或 Kubernetes（kubectl、kustomize 或 helm）无缝安装，提供便捷体验，支持 `:ollama` 和 `:cuda` 标记的镜像。
* Ollama/OpenAI API 集成：轻松集成 OpenAI 兼容 API，与 Ollama 模型一起进行多样化对话。自定义 OpenAI API URL 以连接 LMStudio、GroqCloud、Mistral、OpenRouter 等。
* 粒度化的权限和用户组：通过允许管理员创建详细的用户角色和权限，我们确保了安全的用户环境。这种粒度不仅增强了安全性，还允许定制用户体验，培养用户的归属感和责任感。
* 响应式设计：在桌面 PC、笔记本电脑和移动设备上享受无缝体验。
* 移动端渐进式网页应用（PWA）：通过我们的 PWA，在您的移动设备上享受类似原生应用的使用体验，支持本地离线访问和无缝用户界面。
* 支持完整的 Markdown 和 LaTeX：通过全面的 Markdown 和 LaTeX 功能，提升您的 LLM 体验，实现丰富的交互。
* 无线电语音/视频通话：使用多个语音转文本提供商（本地 Whisper、OpenAI、Deepgram、Azure）和文本转语音引擎（Azure、ElevenLabs、OpenAI、Transformers、WebAPI）体验无缝通信，支持动态和交互式聊天环境。
* 模型构建器：通过 Web UI 轻松创建 Ollama 模型。创建并添加自定义角色/代理，自定义聊天元素，并通过 Open WebUI 社区集成轻松导入模型。
* 本地 Python 函数调用工具：通过工具工作区中的内置代码编辑器增强您的 LLMs。通过简单添加您的纯 Python 函数实现自带函数（BYOF），实现与 LLMs 的无缝集成。
* 持久化工件存储：内置键值存储 API 用于工件，支持日志、追踪器、排行榜和跨会话的协作工具，具有个人和共享数据范围。
* 本地 RAG 集成：通过使用您选择的 9 种向量数据库和多种内容提取引擎（Tika、Docling、文档智能、Mistral OCR、外部加载器），体验突破性的检索增强生成（RAG）支持，开启聊天交互的未来。直接将文档加载到聊天中，或将文件添加到您的文档库，在查询前轻松使用 `#` 命令访问它们。
* RAG 的网页搜索：使用 15+ 个提供商（包括 `SearXNG` 、 `Google PSE` 、 `Brave Search` 、 `Kagi` 、 `Mojeek` 、 `Tavily` 、 `Perplexity` 、 `serpstack` 、 `serper` 、 `Serply` 、 `DuckDuckGo` 、 `SearchApi` 、 `SerpApi` 、 `Bing` 、 `Jina` 、 `Exa` 、 `Sougou` 、 `Azure AI Search` 和 `Ollama Cloud` ）进行网页搜索，将结果直接注入您的聊天体验中。
* 网页浏览功能：使用 `#` 命令后跟 URL，将网站无缝集成到您的聊天体验中。此功能允许您将网页内容直接整合到对话中，增强互动的丰富性和深度。
* 图像生成与编辑集成：使用多个引擎创建和编辑图像，包括 OpenAI 的 DALL-E、Gemini、本地 ComfyUI 和本地 AUTOMATIC1111，支持生成和基于提示的编辑工作流程。
* 多模型对话：轻松同时与多种模型互动，利用它们的独特优势以获得最佳响应。通过并行利用多种模型来增强您的体验。
* 基于角色的访问控制（RBAC）：确保安全访问，具有受限权限；只有授权人员才能访问您的 Ollama，管理员保留创建/拉取模型的专有权限。
* 支持灵活的数据库和存储选项：可选择 SQLite（可选加密）、PostgreSQL，或配置云存储后端（S3、Google Cloud Storage、Azure Blob Storage）以实现可扩展的部署。
* 支持高级向量数据库：可选 9 种向量数据库选项，包括 ChromaDB、PGVector、Qdrant、Milvus、Elasticsearch、OpenSearch、Pinecone、S3Vector 和 Oracle 23ai，以实现最佳的 RAG 性能。
* 企业认证：完全支持 LDAP/活动目录集成、SCIM 2.0 自动配置，以及通过受信任头进行 SSO，同时支持 OAuth 提供商。通过 SCIM 2.0 协议进行企业级用户和组配置，可与企业身份提供商（如 Okta、Azure AD 和 Google Workspace）无缝集成，实现自动化的用户生命周期管理。
* 云原生集成：原生支持 Google Drive 和 OneDrive/SharePoint 文件选择，可从企业云存储无缝导入文档。
* 生产可观测性：内置 OpenTelemetry 支持，用于跟踪、指标和日志，可使用现有可观测性堆栈进行全面监控。
* 水平可扩展性：基于 Redis 的会话管理和 WebSocket 支持适用于负载均衡器后面的多工作器和多节点部署。
* 多语言支持：使用我们的国际化（i18n）支持，以您喜欢的语言体验 Open WebUI。加入我们，一起扩展我们支持的语言！我们正在积极寻求贡献者！
* 管道、Open WebUI 插件支持：使用管道插件框架将自定义逻辑和 Python 库无缝集成到 Open WebUI 中。启动您的管道实例，将 OpenAI URL 设置为管道 URL，并探索无限可能。示例包括函数调用、用户速率限制以控制访问、使用 Langfuse 等工具进行使用监控、使用 LibreTranslate 进行实时翻译以支持多语言、恶意消息过滤等更多功能。
* 持续更新：我们致力于通过定期更新、修复和新功能来改进 Open WebUI。

## 如何安装

### 通过 Python pip 🐍 安装

Open WebUI 可以使用 pip（Python 软件包安装程序）进行安装。在进行之前，请确保您使用的是 Python 3.11 以避免兼容性问题。

1. 安装 Open WebUI：打开您的终端并运行以下命令来安装 Open WebUI：

   pip install open-webui
2. 运行 Open WebUI：安装后，您可以通过执行以下命令启动 Open WebUI：

   open-webui serve
3. 这将启动 Open WebUI 服务器，您可以在 http://localhost:8080 访问它

### 快速使用 Docker 🐳

> 请注意，对于某些 Docker 环境，可能需要额外的配置。如果您遇到任何连接问题，我们的 Open WebUI 文档中的详细指南将为您提供帮助。

> 使用 Docker 安装 Open WebUI 时，请确保在 Docker 命令中包含 -v open-webui:/app/backend/data 。这一步至关重要，因为它确保您的数据库被正确挂载，并防止任何数据丢失。

> 如果您希望使用包含 Ollama 或 CUDA 加速的 Open WebUI，我们建议使用带有 :cuda 或 :ollama 标签的官方镜像。要启用 CUDA，您必须在您的 Linux/WSL 系统上安装 Nvidia CUDA 容器工具包。

### 使用默认配置安装

* 如果 Ollama 在您的计算机上，请使用此命令：

  docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
* 如果 Ollama 在不同的服务器上，请使用此命令：要连接到另一台服务器上的 Ollama，请将 `OLLAMA_BASE_URL` 更改为服务器的 URL：

  docker run -d -p 3000:8080 -e OLLAMA\_BASE\_URL=https://example.com -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
* 要使用 Nvidia 显卡运行 Open WebUI，请使用此命令：

  docker run -d -p 3000:8080 --gpus all --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:cuda

### 仅用于 OpenAI API 使用的安装

* 如果您只使用 OpenAI API，请使用此命令：

  docker run -d -p 3000:8080 -e OPENAI\_API\_KEY=your\_secret\_key -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main

### 带有捆绑 Ollama 支持的 Open WebUI 安装

这种安装方法使用单个容器镜像，将 Open WebUI 与 Ollama 打包在一起，允许通过单个命令进行简化设置。根据您的硬件配置选择合适的命令：

1. 支持 GPU：通过运行以下命令来使用 GPU 资源：

   docker run -d -p 3000:8080 --gpus=all -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama
2. 仅限 CPU：如果你不使用 GPU，请使用此命令：

   docker run -d -p 3000:8080 -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama