---
title: Docker Model Runner 集成 vLLM
author: 峥嵘岁月AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5MTYwMjQ3NQ==&mid=2247484339&idx=1&sn=cd57a34b959ca00c44d5092070bdfbe7&chksm=97d7f0c0c37931cf9e1895df203fc8314ea19dcf9213b100cae2813287f4313340fce5db2d62&mpshare=1&scene=24&srcid=1204XlSxjngqXuFSb8Ns6D8u&sharer_shareinfo=a64e0bf6a73143a3e3cb8e61b556fb80&sharer_shareinfo_first=a64e0bf6a73143a3e3cb8e61b556fb80#rd
---
> 已吸收至：[[01_LLM与大模型/0105_模型服务/0105_核心知识点/模型服务网关与部署边界准则|模型服务网关与部署边界准则]]

Docker Model Runner 集成 vLLM了：实现大模型推理的高吞吐量与可移植性，不错不错！Docker Model Runner旨在简化开发者运行和实验大型语言模型（LLMs）的过程。最新的重大进展是集成了 vLLM 推理引擎，这使得开发者能够在熟悉的 Docker 工作流中，利用 vLLM 的高性能特性，实现 高吞吐量、低延迟 的 AI 推理服务。这一集成消除了在易用性和性能之间的权衡，为 LLM 的从原型设计到生产部署提供了统一且灵活的解决方案。

vLLM：高吞吐量推理的核心驱动力

      vLLM 是一个专为高效服务 LLM 而设计的开源推理引擎，因其在吞吐量、延迟和内存效率方面的卓越表现而被广泛应用于生产环境。vLLM 的核心优势在于其采用了创新的 PagedAttention 算法。

Docker Model Runner 的统一力量

      在集成 vLLM 之前，开发者往往需要在易用性（如 llama.cpp 的可移植性）和性能（如 vLLM 的高吞吐量）之间做出选择。Docker Model Runner 通过将多个推理引擎统一在一个接口下，解决了这一难题 1。

      Docker Model Runner 现在支持两种最主要的开源模型格式，并能根据格式自动选择最佳的推理引擎：

这种智能路由机制意味着开发者可以使用 相同的 Docker 命令、CI/CD 工作流和部署环境，在本地使用 llama.cpp 进行快速原型设计，然后无缝扩展到使用 vLLM 的生产环境。这使得 Docker Model Runner 成为业界首个允许在单个、可移植、容器化工作流中切换多种推理引擎的工具，真正实现了 AI 的 可移植性。

## 快速上手示例

通过 Docker Model Runner 运行 vLLM 模型非常简单，无需复杂的环境配置。

1.安装 vLLM 后端：

docker model install-runner  --backend vllm  --gpu cuda

2.运行模型（CLI 方式）：

docker model run ai/smollm2-vllm  "Are you ready?"

3.通过 API 访问： 

## Docker Model Runner 还会自动启动一个兼容 OpenAI 格式的 API 服务，方便集成到应用程序中。

##

##

## 主流大语言模型都支持：

##

##

## 结论

      Docker Model Runner 集成 vLLM，标志着 LLM 部署进入了一个新阶段。它将 vLLM 的高性能推理能力与 Docker 固有的可移植性和简易性相结合，为开发者提供了一个从个人电脑到大型集群都能保持一致体验的强大工具。

参考文献

https://www.docker.com/blog/docker-model-runner-integrates-vllm/
