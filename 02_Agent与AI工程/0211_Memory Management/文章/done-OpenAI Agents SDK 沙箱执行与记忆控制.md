> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020103_OpenAI Agents SDK/020103_核心知识点/OpenAI Agents SDK沙箱执行与记忆控制|OpenAI Agents SDK沙箱执行与记忆控制]]、[[02_Agent与AI工程/0211_Memory Management/0211_核心知识点/记忆管理分层与注入边界|记忆管理分层与注入边界]]
---
title: OpenAI Agents SDK 沙箱执行与记忆控制
author: SoftWorking
date: 阿柄阿柄
url: https://mp.weixin.qq.com/s?__biz=Mzg4Njc4Mjg2OQ==&mid=2247484075&idx=1&sn=d1f1453f795d98766a75de8828a42841&chksm=ce289a6a5aa9e9ba5db91c681abce24d1b88f530cf405bef227797558f278f6f7e5215bb1b1a&mpshare=1&scene=24&srcid=0603iG3tu2dpAVIgIQeFdFQ7&sharer_shareinfo=c5233a5c090f23ed5b9bb82464aeaf9b&sharer_shareinfo_first=c5233a5c090f23ed5b9bb82464aeaf9b#rd
---

长时程 Agent 任务面临一个根本问题：执行需要隔离（安全），但隔离意味着状态丢失（上下文断裂）。

过去的解决方案要么放弃隔离（代码直接在主机执行，安全隐患巨大），要么放弃状态连续性（每次任务重置，无法积累项目经验）。OpenAI 的答案是：**双层记忆架构 + 沙箱快照恢复**，让安全隔离和上下文连续性不再是非此即彼的选择。

本文将深入解析这次升级的技术核心：Harness/Compute 分离架构、沙箱执行机制、可配置记忆系统，以及它们如何共同支撑起生产级 Agent 的运行需求。

---

## 一、Harness 与 Compute 分离：安全架构的本质

### 1.1 为什么分离是必然选择

在 Agents SDK 旧版本中，所有组件——执行循环、模型调用、工具路由、凭证管理、运行状态——都堆在一个执行环境里。这带来的问题是：**凭证和执行环境共存，模型生成的代码可以直接访问敏感凭据**。

Prompt injection 攻击暂且不论，即使模型本身可信，生成的代码在执行过程中也可能被恶意输入误导。换言之，传统架构默认「执行环境是可信的」——这是一个危险的前提。

OpenAI 的解决方案是将架构一分为二：

**Harness（控制层）**：包含模型以外的所有组件——执行循环、模型调用、工具路由、任务交接、审批机制、追踪记录、恢复流程及运行状态管理。

**Compute（运算层）**：仅负责供应商特定的沙箱执行，即代码实际运行的容器环境。

### 1.2 分离架构的实战效果

分离的直接价值是：**凭证永远不会进入模型可触达的执行环境**。沙箱容器只持有运行任务所需的最小权限，即使 Agent 生成的代码遭遇恶意输入，攻击面也被严格限制在隔离环境内。

```
from agents.run import RunConfig from agents.sandbox import SandboxRunConfig from agents.sandbox.sandboxes.docker import DockerSandboxClient  run_config = RunConfig(     sandbox=SandboxRunConfig(         client=DockerSandboxClient.from_env(),         options={"image": "python:3.14-slim"}     ) )
```

这段配置将 Harness（控制在本地进程）与 Compute（执行在 Docker 沙箱中）完全分离。凭证、API 密钥、模型调用逻辑均在 Harness 侧，Compute 沙箱只通过工具调用协议与 Harness 通信。

### 1.3 架构对比

毫无疑问，Harness/Compute 分离是 Agent 架构史上的一次关键切割：安全不再是事后补救的加固层，而是架构设计的第一性约束。

---

## 二、沙箱执行：七家提供商与选择逻辑

### 2.1 两类本地客户端

SDK 内置两个本地沙箱客户端，开发者无需注册任何外部服务即可快速迭代：

**UnixLocalSandboxClient**：无额外依赖，macOS/Linux 原生适配，适合本地开发调试。默认选项。

**DockerSandboxClient**：需要 `openai-agents[docker]` 安装包，提供完整的容器隔离能力，适合需要环境一致性的生产场景。

```
# 安装 Docker 沙箱支持 pip install "openai-agents[docker]"
```

```
# 本地快速验证 from agents.sandbox.sandboxes.local import UnixLocalSandboxClient  client = UnixLocalSandboxClient()  # 无需任何配置
```

### 2.2 七家托管沙箱提供商

OpenAI 与七家基础设施供应商达成深度集成，企业可根据现有技术栈和合规需求选择：

**选择逻辑**：**快速原型** → UnixLocal ｜ **生产部署** → 托管沙箱

### 2.3 云存储挂载能力矩阵

沙箱执行的一个核心价值在于直接处理远程数据，而不需要将数据下载到主机。Manifest 的挂载条目（Mount Entries）定义了存储在沙箱中的暴露方式。

各提供商支持的远程存储后端：

---

## 三、Manifest 抽象：一次编写，到处一致运行

### 3.1 声明式工作空间描述

Manifest 是描述 Agent 工作空间的声明式接口，它解决的核心问题是：**从本地开发到生产部署，环境不一致导致的调试地狱**。

过去，Agent 在本地运行正常，在生产环境失败——通常是因为依赖版本、路径配置、文件权限的细微差异。Manifest 将工作空间描述为一份声明式配置，无论在哪家沙箱提供商上运行，Agent 拿到的环境语义完全一致。

### 3.2 Manifest 挂载条目配置

挂载条目定义存储在沙箱中的暴露方式，核心字段：

•**mount\_path**：沙箱内的显示路径

•**read\_only**：默认 `True`，仅在需要写回时设为 `False`

•**mount\_strategy**：必填，支持 rclone、mountpoint、FUSE 等多种后端

```
from agents.run import RunConfig from agents.sandbox import SandboxRunConfig, MountEntry from agents.sandbox.sandboxes.docker import DockerSandboxClient  run_config = RunConfig(     sandbox=SandboxRunConfig(         client=DockerSandboxClient.from_env(),         mounts=[             MountEntry(                 mount_path="/data",                 source="s3://my-bucket/project-data",                 mount_strategy="rclone",                 read_only=True             )         ]     ) )
```

这段配置将 S3 上的数据以只读方式挂载到沙箱的 `/data` 目录。Agent 在沙箱中可直接读取 S3 数据，无需手动下载。

### 3.3 多后端存储挂载的实战价值

跨云存储挂载能力对企业有直接意义：**数据无需离开云服务商即可被 Agent 处理**。这对于有严格数据主权要求或合规限制的行业（如金融、医疗、法律）是关键能力。

Agent 在沙箱中完成数据清洗、转换、分析，最终写回指定存储位置——整个过程凭证隔离，模型不可访问存储凭据。

---

## 四、可配置记忆：Session 与 Sandbox 的双层架构

### 4.1 为什么 Agent 需要记忆

传统聊天机器人只有一个记忆维度——对话历史（Session 记忆）。当 Agent 需要处理跨越数天的复杂任务时，这种单层记忆的根本缺陷暴露出来：

•**对话历史无法提炼**：Agent 记住了每一次对话，但无法将重复经验提炼为可操作的指导原则

•**跨会话上下文丢失**：昨天完成的重构经验，今天无法继承，每次都要重新解释项目规范

•**上下文窗口爆炸**：长期项目的对话历史可能超出模型上下文限制

### 4.2 双层记忆架构

Agents SDK 的记忆系统包含两个独立层次，它们解决不同层面的问题：

**Session 记忆（对话式）**：保留消息历史，用于多轮对话上下文。这是所有 Agent 框架都有的能力。

**Sandbox 记忆（沙箱记忆）**：将工作空间运行过程中积累的经验提炼为文件，供 Agent 后续任务参考。与 Session 记忆不同，沙箱记忆保存的是**可重用的工作指引**而非对话记录。

```
# Session 记忆：保留对话上下文 agent = Agent(     name="code-reviewer",     instructions="你是一个代码审查专家",     session_memory=[...],  # 对话历史     sandbox_memory=[...]   # 项目经验文件路径 )
```

具体来说，Agent 可在不重播每轮对话的情况下持续承载：

• 用户偏好（如代码风格、PR 规范）

• 修正建议（之前踩过的坑）

• 项目特定经验（哪个模块需要特别处理）

• 任务摘要（长期项目的当前进度）

这种双层架构令 Agent 具备接近人类员工的「工作记忆」特质，对需要长期项目管理的场景尤为重要。

### 4.3 记忆与安全的协同

Sandbox 记忆还有一个被忽视的价值：**它是在隔离环境中积累的，不会在沙箱销毁时一并丢失**。快照与恢复机制让 Agent 在重新启动后依然能读取之前的经验文件。

这意味着 Agent 的上下文连续性不再依赖主机的持久化存储，而是由沙箱快照机制保证。这对无状态服务器环境下的 Agent 部署尤为重要。

---

## 五、快照与恢复：长时程任务的可靠性保障

### 5.1 问题：沙箱容器不是永生的

沙箱容器会因为超时、节点故障或提供商调度而被销毁。如果 Agent 在运行中途失去了容器，所有状态——包括对话上下文、中间计算结果、文件修改——都会丢失。

对短时任务来说，这不是问题。但对于需要数小时乃至数天完成的复杂任务（跨系统数据整合、财务报告汇总、代码库大规模重构），状态丢失的代价是不可接受的。

### 5.2 快照机制

SDK 内置快照与重水化（rehydration）机制：当沙箱容器发生故障或过期时，Agent 状态从上次检查点恢复，在全新容器中继续运行。

**丢失沙箱容器不等于丢失运行**。这大幅提升了长时间运行任务的可靠性，将 Agent 的 SLA 从「每次任务必须一口气完成」提升为「支持断点续传」。

### 5.3 与双层记忆的协同

快照恢复解决的是**运行时状态**的问题，双层记忆解决的是**知识积累**的问题。两者结合，Agent 在三个方面都不丢失：

---

## 六、应用场景与客户案例

### 6.1 临床记录自动化处理

**案例：Oscar Health（美国医疗保险公司）**

*核心挑战：*正确理解复杂长记录中每次就诊的边界，并提取正确元数据。

*技术支撑：*沙箱隔离确保医疗数据不外泄；长时程任务支持处理跨多天的记录；Manifest 抽象确保开发和生产环境一致。

*结果：*从「无法可靠处理」升级为「生产级自动化」。

### 6.2 法律文档起草与工作流构建

**案例：律商联讯（LexisNexis）**

*核心挑战：*复杂法律文件起草需要在隔离环境中处理大量敏感文档，同时 Agent 需要保持跨会话的上下文理解。

*技术支撑：*沙箱隔离 + Sandbox 记忆（记住法律文档格式规范） + 快照恢复（长时间起草任务不中断）。

*评价来源：*律商联讯首席 AI 官 Min Chen 表示，团队得以专注于开发高价值、长期运行的法律 Agent，而非从零搭建基础设施。

### 6.3 企业代码库重构与自动化测试

典型流程：

1. Agent 在沙箱中读取代码文件

2. 执行测试命令

3. 写回修改结果

4. 全程在隔离环境中完成，无需担心代码对主机造成影响

Sandbox 记忆让 Agent 记住过往重构中的注意事项和项目特定规范，实现跨会话的持续优化。

---

## 总结

OpenAI Agents SDK 的这次升级，本质上是在解决一个问题：**如何让 Agent 从「能跑 demo」进化到「能跑生产」**。

三条技术路线汇合于此：

**Harness/Compute 分离**：安全不再是事后加固层，而是架构的第一性约束。凭证不进入执行环境，Prompt injection 的攻击面被严格限制在沙箱容器内。

**双层记忆架构**：Session 记忆保证对话上下文，Sandbox 记忆保证项目经验积累。长时程任务不再因上下文窗口耗尽而崩溃。

**快照与恢复**：将 Agent 的可靠性从「一次性任务」提升为「支持断点续传」，这对于需要数小时乃至数天的复杂任务是不可替代的能力。

从更宏观的视角看，这次更新揭示了一个行业趋势：**Agent 框架的竞争焦点正在从「模型能力」转向「工程基础设施」**。当模型能力本身不再是瓶颈，谁能解决安全隔离、环境一致性、长时程任务可靠性，谁就能在生产级部署中胜出。

这条路是对的。但对的方向不等于唯一的方向。最终胜出者不一定是功能最全的，而是开发者体验最好的。TypeScript 版本的延迟、新增的复杂度、以及与 Codex 的边界模糊，都是 OpenAI 接下来需要回答的问题。