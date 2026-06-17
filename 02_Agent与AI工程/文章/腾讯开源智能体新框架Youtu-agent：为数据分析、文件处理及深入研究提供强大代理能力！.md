---
title: 腾讯开源智能体新框架Youtu-agent：为数据分析、文件处理及深入研究提供强大代理能力！
author: AIGitHub
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5ODkyOTkwOQ==&mid=2247492631&idx=1&sn=2456f0c2541927616764187d9dc151c6&chksm=c134f2b8a01f1ddaa742ab7da259dd591234e0b761f160eab621eed2ab4baf958e4acff1a1aa&mpshare=1&scene=24&srcid=0908gC3JhnGihlqZcnQ3JXNm&sharer_shareinfo=a7cbaf67da4195f4192018d20deba4c4&sharer_shareinfo_first=a7cbaf67da4195f4192018d20deba4c4#rd
---

腾讯优图实验室重磅推出的**开源****智能体****框架: Youtu-agent**，致力于为自主智能体的构建、运行和评估提供专业解决方案。

依托开源模型 DeepSeek-V3 的技术优势，该框架不仅达成了领先的性能水准，还兼容多种模型 API 与工具集成，能够轻松应对数据分析、文件处理、深度研究等多元化需求，展现出卓越的智能体能力。

****功能特点****

**性能验证**：

WebWalkerQA：基于DeepSeek-V3.1达到**71.47%**准确率，刷新开源效果SOTA；

**GAIA（文本子集）：**基于DeepSeek-V3 Pass@1达到**72.8%**，不用充值Claude/GPT等闭源模型，验证了强大的研究和应用潜力。

**开源友好与成本意识**：优化低成本部署，不依赖闭源模型，适合广泛的应用场景。

**灵活架构**：基于openai-agents构建，支持多种模型API（如DeepSeek、gpt-oss）、工具集成和框架实现。

**自动化与简化**：基于YAML的配置、自动智能体生成和简化设置，减少手动操作。

****实测案例****

数据分析：分析 CSV 文件并生成 HTML 报告。

文件管理：为用户重命名本地文件并对其进行分类。

广泛的研究：收集大量信息以生成综合报告，复制 Manus 的功能。

论文分析：解析给定的论文，进行分析，并汇编相关文献以产生最终结果。

****技术原理****

**AgentConfig**：智能体的配置文件，用YAML格式定义智能体的行为、使用的工具、环境等，为智能体提供运行所需的参数和设置。

**Agent**：智能体的核心逻辑部分，根据AgentConfig中的配置运行，并在环境中执行任务。Agent是单个智能体（如SimpleAgent），也能是多个智能体协同工作（如OrchestraAgent）。

**Environment**：智能体运行的环境，提供智能体与外部世界交互的接口。例如，BrowserEnv支持智能体在浏览器中操作网页，ShellLocalEnv支持智能体访问本地文件系统。

**Toolkits**：智能体的能力集合，提供智能体能调用的各种工具。例如，search工具支持智能体进行网络搜索，file工具支持智能体操作文件。

**Evaluation Framework**：用于评估智能体性能的框架，提供标准化的评估流程，包括数据管理、处理和执行。

```
GitHub：https://github.com/TencentCloudADP/Youtu-agent
```

**一起交流AI前沿技术！**

**小编免费共享AI开源项目知识库，**

****实现大家的AI资讯自由！****

****直接扫码或点击链接即可查看！****

AI开源项目知识库：https://qyxznlkmwx.feishu.cn/wiki/BwWIwsCOuiMWGmkUzNHcKLvPnPh

点击下方名片「**关注我们**」第一时间收到推送