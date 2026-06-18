> 已吸收至：[[02_Agent与AI工程/0210_sandbox/021001_AgentSandbox/021001_核心知识点/AgentSandbox隔离与审计边界|AgentSandbox隔离与审计边界]]、[[02_Agent与AI工程/0210_sandbox/021001_AgentSandbox/021001_核心知识点/Sandbox运行时实现与选型|Sandbox运行时实现与选型]]
---
title: 在 Kubernetes 上使用 Agent Sandbox 运行智能体
author: 云原生探索号
date:
url: https://mp.weixin.qq.com/s?__biz=MzkyMTI3OTM5Ng==&mid=2247485026&idx=1&sn=da7bffdcfc780910be0c4284e0116dbc&chksm=c0362d20de0bdd50d8b130f9b9c6e8194eba5c2c89a0ba0a20a5cbc78f1ebb47f7bb5dfdb7dc&mpshare=1&scene=24&srcid=0510vpzbDnk4dch2Pjm5e7z2&sharer_shareinfo=2381bd3bf661f35089f2a8f414a1c4bc&sharer_shareinfo_first=2381bd3bf661f35089f2a8f414a1c4bc#rd
---

借由 **Janet Kuo Justin Santa Barbara** | 译者 **Xin Li** |

2026.03.20

人工智能领域正经历着一场巨大的架构变革。 在生成式人工智能的早期，与模型交互通常被视为一个瞬态的、无状态的函数调用：一个启动、执行可能仅 50 毫秒便终止的请求。

如今，人工智能 2.0 正在取代人工智能 1.0。 人工智能生态系统正从短暂、孤立的任务转向部署多个持续运行的、协同工作的 AI 智能体。 这些自主智能体需要维护上下文信息、使用外部工具、编写和执行代码，并在较长时间内相互通信。

当平台工程团队寻找合适的架构来托管这些新型 AI 工作负载时， Kubernetes 脱颖而出，成为自然之选。 然而，将这些独特的智能体工作负载映射到传统的 Kubernetes 原语需要一种新的抽象。

这正是新的 Agent Sandbox 项目（目前由 SIG Apps 开发）发挥作用的地方。

## Kubernetes 的优势（以及抽象鸿沟）

Kubernetes 之所以成为云原生应用编排的事实标准， 正是因为它解决了可扩展性、稳健的网络和生态系统成熟度方面的挑战。 然而，随着人工智能从短暂的推理请求演变为长时间运行的自主智能体，我们正在见证一种新的运行模式的出现。

相比之下，AI 智能体通常是隔离的、有状态的、单例工作负载。 它们充当生命周期管理（LLM）的数字工作空间或执行环境。 智能体需要一个持久的身份和一个安全的暂存区来编写和执行（通常是不受信任的）代码。 至关重要的是，由于这些长时间运行的智能体除了短暂的活动爆发外，大部分时间都处于空闲状态， 因此它们需要一个支持诸如暂停和快速恢复等机制的生命周期。

虽然理论上可以通过为每个智能体串联一个大小为 1 的 StatefulSet、一个无头服务和一个 PersistentVolumeClaim 来近似实现这一点，但大规模管理这些组件将成为运维噩梦。

由于这些独特的特性，传统的 Kubernetes 原语无法完美契合。

## Kubernetes Agent Sandbox 简介

为了弥合这一差距，SIG Apps 正在开发 agent-sandbox。 该项目引入了一个声明式、标准化的 API，专门针对单例、有状态工作负载（例如 AI 智能体运行时）量身定制。

该项目的核心是引入了 Sandbox CRD。它是一个轻量级的单容器环境，完全基于 Kubernetes 原语构建，提供以下功能：

* **针对不受信任代码的强隔离**

  ：当 AI 智能体自主生成和执行代码时，安全性至关重要。 Sandbox 自定义资源原生支持不同的运行时环境，例如 gVisor 或 Kata Containers。 这为多租户、不受信任的执行提供了必要的内核和网络隔离。
* **生命周期管理**

  ：与针对稳定、无状态流量优化的传统 Web 服务器不同， AI 智能体作为有状态的工作空间运行，在两次任务之间可能被闲置数小时。 Agent Sandbox 支持将这些闲置环境缩减至零以节省资源，同时确保它们可以从上次中断的地方恢复。
* **稳定的身份**

  ：协同的多智能体系统需要稳定的网络。 每个 Sandbox 都拥有稳定的主机名和网络身份，使不同的智能体能够无缝地相互发现和通信。

## 利用扩展来缩放智能体规模

由于人工智能领域发展迅猛，我们构建了一个扩展 API 层，以支持更快的迭代和开发。

启动一个新的 Pod 会增加大约一秒钟的开销。 部署新版本的微服务时，这完全可以接受，但如果智能体在空闲后被调用，一秒钟的冷启动会中断交互的连续性。 它迫使用户或编排服务等待环境配置完成，模型才能开始思考或行动。SandboxWarmPool 通过维护一个预配置的 Sandbox Pod 池来解决这个问题，从而有效地消除了冷启动。 用户或编排服务只需针对 SandboxTemplate 发出 SandboxClaim， 控制器就会立即将一个预热的、完全隔离的环境交给智能体。

## 快速入门

准备好亲自体验了吗？你可以选择最新版本，将 Agent Sandbox 的核心组件和扩展程序直接安装到你的学习或 Sandbox 集群中。

由于项目进展迅速，我们建议你使用最新版本。

```
# 将 “vX.Y.Z” 替换为来自 https://github.com/kubernetes-sigs/agent-sandbox/releases
# 的特定版本标签（例如，“v0.1.0”）。
exportVERSION="vX.Y.Z"

# 安装核心组件：
kubectl apply -f https://github.com/kubernetes-sigs/agent-sandbox/releases/download/${VERSION}/manifest.yaml

# 安装扩展组件（可选）：
kubectl apply -f https://github.com/kubernetes-sigs/agent-sandbox/releases/download/${VERSION}/extensions.yaml

# 安装 Python SDK（可选）：
# 创建一个虚拟 Python 环境
python3 -m venv .venv
source .venv/bin/activate
# 从 PyPI 安装
pip install k8s-agent-sandbox
```

安装完成后，你可以试用 AI 智能体的 Python SDK， 或者部署一个现成的示例， 看看启动一个隔离的智能体环境是多么容易。

## 智能体的未来在于云原生

无论是 50 毫秒的无状态任务，还是持续数周、大部分时间处于空闲状态的协作流程， 通过扩展 Kubernetes，添加专为隔离的有状态单例设计的原语， 我们都能充分利用云原生生态系统的强大优势。

Agent Sandbox 项目是开源且由社区驱动的。 如果你正在构建 AI 平台、开发智能体框架，或者对 Kubernetes 的可扩展性感兴趣， 我们诚邀您参与其中：

* 在 GitHub 上查看项目：kubernetes-sigs/agent-sandbox
* 加入 Kubernetes Slack 上的 #sig-apps 和 #agent-sandbox 频道参与讨论。