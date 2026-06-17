---
title: 腾讯Youtu-agent 不会代码，也能搞定数据分析+深度研究+HTML报告
author: CourseAI
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNjgwNzMwNQ==&mid=2247487003&idx=1&sn=5978ecc8fb66910cd86bd60c8d4ad17c&chksm=c32442799c7e60e10770f03e473aaeec1cbc1dadfd9dec1b07aefb390073909b516e266a5e82&mpshare=1&scene=24&srcid=0910BPrsOHAHkGbJ91FGg4hU&sharer_shareinfo=33764ed732033df1c744efa4e9d0615c&sharer_shareinfo_first=33764ed732033df1c744efa4e9d0615c#rd
---

本公众号主要关注NLP、CV、LLM、RAG、Agent等AI前沿技术，免费分享业界实战案例与课程，助力您全面拥抱AIGC。

# 

腾讯开源的Agent框架，`Youtu-agent` 是一个灵活、高性能的框架，用于构建、运行和评估自主智能体。

* 可以用于数据分析、文件处理和深度研究，支持CSV 分析、文献综述、个人文件整理以及播客和视频生成等任务。
* 基于openai-agents构建的，继承了 streaming、tracing 和 agent-loop 能力，确保了与 `responses` 和 `chat.completions` API 的兼容性，能够适配各种模型 API，工具包（Toolkit）等。
* 最突出的优势：能自动化生成智能体及其配置的能力

* 在其他框架中，定义特定任务的智能体通常需要编写代码或是精心设计提示词
* `Youtu-agent`采用基于 YAML 的简洁配置方案，实现了高效自动化
* 内置的“元智能体”与用户对话并捕获需求，然后自动生成智能体配置。
* 完全异步：实现了高性能和高效执行，尤其有利于高效的评估。
* 目前实战的场景包括以下几个方面：

> https://github.com/Tencent/Youtu-agent