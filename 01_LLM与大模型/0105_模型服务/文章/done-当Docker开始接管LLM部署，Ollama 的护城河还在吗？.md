---
title: 当Docker开始接管LLM部署，Ollama 的护城河还在吗？
author: 王人书
date:
url: https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487618&idx=1&sn=a05eec88024532e59cc2a3476019871b&chksm=9e1495ae8fd4245fe76d4e980af820a2819ac973b912dd22ec19ebbf74a4c11985bbf628dc31&mpshare=1&scene=24&srcid=1128jMB8OIk7RvthcuFB6vF9&sharer_shareinfo=73ef81bbcad1e7e50499d16d380c78bb&sharer_shareinfo_first=73ef81bbcad1e7e50499d16d380c78bb#rd
---
> 已吸收至：[[01_LLM与大模型/0105_模型服务/0105_核心知识点/模型服务网关与部署边界准则|模型服务网关与部署边界准则]]

## 点击👇🏻可关注，文章来自 大家好，我是Alden，LLM开发工程师。随着 vLLM 后端的原生集成，Docker Model Runner 正式完成从实验特性到生产组件的蜕变。本文将初步介绍其架构演进，对比 Ollama 的工程差异，探讨 DMR 如何以标准化、确定性的理念，打造企业级 LLM基础设施的运维体系。下期，我们将详细介绍如何使用 DMR 进行模型本地部署。 字数 2130，预计阅读 7 分钟

##

## 背景：从实验特性到生产组件

当 Docker 在今年 4 月首次推出 **Docker Model Runner (DMR)** 时，外界普遍将其视为 Ollama 的“容器化模仿者”——仅支持 GGUF 格式，局限于本地开发。

但 **11月19日原生集成 vLLM 后端及 .safetensors 支持** 的更新，让其定位发生了本质跃迁。DMR 从此摆脱了“玩具”标签，直接具备了生产级的高吞吐能力。对于习惯了 k8s 的工程师来说，这意味着终于可以用管理 Redis 的方式来管理 LLM——**告别割裂的工具链，大幅降低异构负载的运维认知负担。**

## 1. DMR 的架构演进：双轨路由机制

DMR 目前的核心逻辑在于根据模型权重的格式，自动路由到底层不同的推理引擎。这种设计试图兼顾本地开发的低资源需求与生产环境的高性能需求。

### 1.1 开发侧：GGUF 与 llama.cpp

当 DMR 识别到 OCI Artifact 为 `.gguf` 格式时，它会调用 **llama.cpp** 后端。

* **场景**：本地调试、Apple Silicon 设备、显存受限环境。
* **特性**：低冷启动时间，对硬件兼容性极好。

### 1.2 生产侧：Safetensors 与 vLLM

这是本次更新的核心。当拉取模型权重是 `.safetensors` 的模型时，DMR 会调用 **vLLM** 引擎。

* **技术红利**：原生获得了 PagedAttention、Continuous Batching（连续批处理）以及 Tensor Parallelism（张量并行）的能力。
* **运维视角**：这一层抽象使得基础设施工程师无需深入钻研 vLLM 复杂的 Python 依赖环境配置，直接获得了一个经过厂商验证的、支持高并发的标准推理单元，减少了因环境依赖冲突导致的“生产环境无法启动”事故。

## 2. 工程视角的差异：显式定义 vs. 隐式便利

在实际落地中，Ollama 和 DMR 代表了两种截然不同的工程哲学。Ollama 倾向于“Convention over Configuration”（约定优于配置），而 DMR 则严格遵循“Explicit is better than Implicit”（显式优于隐式）。

### 2.1 量化与资源管理的确定性

* **Ollama 的动态策略**： Ollama 的优势在于极致的易用性。例如，拉取一个模型时，它通常会根据硬件情况默认选择 4-bit 量化版本；运行时会根据 VRAM 负载动态调整 KV Cache 的量化级别。这种“黑盒”优化对个人开发者极其友好，但在生产环境中，**“动态”往往意味着“不可复现”**。性能波动或精度漂移可能仅仅是因为重启后显存碎片率不同导致的。
* **DMR 的静态契约**： DMR 沿用了 Docker 镜像的不可变基础设施理念。

+ 模型精度被固化在 Tag 中（如 `qwen3:4B-F16` 或 `gemma3:4B-Q4_K_M`）。
+ 一旦选定 Tag，无论是在开发笔记本还是生产服务器上，加载的权重二进制流是完全一致的。
+ 参数透传（Runtime Flags）必须在 Compose 文件中显式声明（如 `--max-model-len`），没有隐含的自动调整。
+ **这点对于 SRE 至关重要：** 它消除了因环境差异导致的“幻觉”概率波动，让故障排查回归到确定的配置版本上。

### 2.2 基础设施即代码 (IaC) 的集成度

DMR 最显著的变革在于它修改了 Docker Compose 的语法规范。它将 **`models` 提升为与 `services`、`networks`、`volumes` 平级的顶层元素**。

这意味着在 Docker 的定义中，AI 模型不再是某个服务内部的附属依赖，而是成为了基础设施中的\*\*“一等公民” (First-class Citizen)\*\*。

**DMR Compose 示例：**

```
# docker-compose.yml
services:
rag-api:
    image:my-org/api:v1
    # 通过服务名引用，自动注入 LLM_ENDPOINT 环境变量
    models:
      -llm-service

# models 现在是顶层配置项
models:
llm-service:
    model:ai/llama-3.2:3b-safetensors# 明确指定生产级格式
    driver:vllm                       # 明确指定后端
    gpus:all
    runtime_flags:                     # 显式控制推理参数
      -"--gpu-memory-utilization 0.9"
```

这种架构上的升维，使得模型服务可以像 Redis 、 PostgreSQL 一样被版本控制、审查和回滚。对于已经建立起 GitOps 流程（如使用 ArgoCD）的团队，DMR 几乎是零成本接入，无需编写额外的 Operator 或复杂的初始化脚本。

## 3. 技术规格对比

下表从工程落地的维度对 Ollama 与 Docker Model Runner 二者进行对比(2025.11)：

| 维度 | **Ollama** | **DMR** |
| --- | --- | --- |
| **核心定位** | 极致的本地/边缘推理工具 (Tool) | 标准化的容器化推理组件 (Component) |
| **推理后端** | 定制版 llama.cpp (主力) + 实验性扩展 | **vLLM** (生产) + llama.cpp (兼容) |
| **硬件兼容** | **高适配性** 。自动兼容 AMD ROCm、Intel NPU 及旧版 GPU。 | **有门槛** 。Windows/Linux 严格依赖特定 NVIDIA 驱动版本。 |
| **精度控制** | **隐式/动态** 。默认优选 Q4，运行时自动优化 KV Cache。 | **显式/静态** 。通过 Image Tag 严格锁定模型版本与精度。 |
| **权重管理** | 自有 Registry + Modelfile。 | **OCI 标准** 。复用 Docker Hub/Harbor，支持镜像签名与扫描。 |
| **网络拓扑** | 默认监听 localhost，需配置 host 绑定。 | 天然集成 Docker Network，服务间隔离通信。 |

## 4. 生态影响与选型思考

本次 vLLM 的加入，补齐了 Docker 在 LLM 基础设施版图中最关键的一块拼图。

### 供应链的统一

对于企业 IT 部门而言，DMR 的吸引力在于**供应链管理的复用**。

* **无需新增白名单**：无需为 LLM 模型单独维护一套鉴权、存储和传输机制，直接复用现有的 Harbor、Artifactory 以及 RBAC 策略。
* **安全合规更轻松**：模型作为 OCI Artifacts，天然支持 Docker Content Trust 签名和漏洞扫描流程。对于那些对数据出境和二进制文件来源极其敏感的 InfoSec 团队来说，DMR 提供了一个更容易通过安全审计的方案。

Ollama 虽然支持私有库，但在企业级安全合规的集成深度上，Docker 生态的既有惯性构成了强大的护城河。下期，我们详细介绍如何使用 DMR 进行本地模型部署。

---

**Reference Source:**

1. **Docker Model Runner Integrates vLLM for High-Throughput Inferencing** (Blog)

* *https://blog.vllm.ai/2025/11/19/docker-model-runner-vllm.html/*

2. **Docker Model Runner** (GitHub Repo)

* *https://github.com/docker/model-runner/*

3. **Define AI Models in Docker Compose applications** (Docs)

* *https://docs.docker.com/ai/compose/models-and-compose/*

4. **Get started with DMR** (Docs)

* *https://docs.docker.com/ai/model-runner/get-started/*

5. **DMR REST API Reference**

* *https://docs.docker.com/ai/model-runner/api-reference/*

关联阅读：

[AI for Coding：从 Vibe Coding 到规范驱动开发](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487527&idx=1&sn=6d63dcc5bfc87e643a7cf3723b671918&scene=21#wechat_redirect)

[LLM重塑系统边界：二阶序与认知工具的知识平权](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487467&idx=1&sn=e96a030fdf68cf0eb1bf100302a16738&scene=21#wechat_redirect)

[黑客、语言囚徒与技术时代的共构](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487438&idx=1&sn=38c98f1935e71961957412879b60afde&scene=21#wechat_redirect)

[Vibe coding：LLM 时代的“没有银弹”回响](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487420&idx=1&sn=9473debc94d7635caec40304ec72504e&scene=21#wechat_redirect)

[跨域的结构同构性](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487400&idx=1&sn=7ed8064bd816774c8a97080409acef50&scene=21#wechat_redirect)

[澄明之镜：Manus AI 与人的存在主义觉醒](https://mp.weixin.qq.com/s?__biz=MzA3NDE2ODIwMw==&mid=2247487242&idx=1&sn=313d3572793322e862118b11cd297938&scene=21#wechat_redirect)
