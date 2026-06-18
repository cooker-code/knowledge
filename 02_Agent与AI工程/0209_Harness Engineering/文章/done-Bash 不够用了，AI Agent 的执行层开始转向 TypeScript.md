> 已吸收至：[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/上下文与工具加载边界|上下文与工具加载边界]]
---
title: Bash 不够用了，AI Agent 的执行层开始转向 TypeScript
author: 木兆纸
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg3MjA1NzQ4Mw==&mid=2247484131&idx=1&sn=4334ff467e0b28177172f2b635d78157&chksm=cff2d705f5fbb1ebb7803014fdd165cf4474cf300f4cfce4fdc10d01522fe34110d9c2f9ef1a&mpshare=1&scene=24&srcid=0326UHcdUf44VCaFae8r4H9D&sharer_shareinfo=72cd0b6b4e006ffa667b85a8558a5dd4&sharer_shareinfo_first=72cd0b6b4e006ffa667b85a8558a5dd4#rd
---

> Bash一直是AI Agent很顺手的执行层：会写命令、能串工具、还能边跑边发现环境。但当Agent开始接入越来越多系统，Bash就慢慢不够用了——权限太粗、审计太难、上下文效率也不高。Cloudflare和独立开发者们现在开始转向另一个答案：**让Agent写TypeScript代码**，用一个"写代码"的工具，替代数百个API调用。

---

*注：本文综合了Cloudflare Code Mode论文、独立开发者Reese的Executor项目，以及技术博主ThePrimeagen关于AI Agent执行层的深度分析。*

---

## 一、MCP的困境：工具太多，Agent变笨

MCP（Model Context Protocol）是Anthropic在2024年底开源的标准协议，用于将AI应用连接到外部系统。它的价值很明显：标准化接口，降低集成成本。

但MCP有一个致命问题——**工具爆炸**。

### 案例1：Google Cloud SQL MCP的11个工具

以Google Cloud的Cloud SQL MCP服务器为例，它提供了11个工具：

* `clone_instance`

  ：克隆数据库实例
* `create_instance`

  ：创建新实例
* `create_user`

  ：创建数据库用户
* `execute`

  ：执行SQL查询
* `get_instance`

  ：获取实例信息
* `get_operation`

  ：获取操作状态
* ……还有5个其他工具

看起来不多？但当你把多个MCP服务器接入Agent时，工具数量会指数级增长。

### 案例2：真实场景下的工具膨胀

一个典型的开发者场景：你接入两个MCP服务器——

* **GPile**（文件系统工具）：5-6个工具
* **BTCA local**（本地开发工具）：2个工具
* **Agent自带工具**：8-12个工具

突然之间，Agent要面对**近20个工具定义**。这还只是基础配置。

### 案例3：PostHog MCP的"工具地狱"

PostHog是一个产品分析平台，它的MCP服务器提供了大量工具——用于创建仪表盘、查询事件数据、管理项目等。实际使用中，开发者发现这个MCP的工具数量多到"不想让它99%的时间都开着"，只在需要专门操作PostHog时才启用。

### 案例4：Uber内部的MCP网关

Uber构建了一个内部的MCP网关，同时暴露**数百个工具**——第一方MCP、第三方MCP、代理的HTTP端点（包括Protobuf和传统HTTP接口）。这个网关不在用户本地机器上，而是运行在远程服务器上，使得：

* **权限控制**：可以在网关层面统一处理认证和授权，不同团队（工程团队、销售团队）获得不同的工具集
* **集中审计**：所有操作日志、遥测数据在一个地方统一管理
* **服务发现**：作为一个注册中心，新MCP服务器的添加和发现变得极其简单

但即便如此，Uber仍然需要频繁切换工具开关，防止模型被工具定义淹没。

**工具越多，Agent越笨**，这不是比喻，是实测结论。模型的上下文窗口被工具定义占据后，留给实际任务的空间就少了，判断力也随之下降。

---

## 二、Bash的局限：自由过头的执行层

既然直接给工具定义会撑爆上下文，有人想到一个聪明的办法：**只给Agent一个Bash工具**。

Agent确实擅长写Bash命令。它们可以发现工具（用`which`、`grep`）、链式执行（管道）、处理文件系统……Bash本身就是一个完整的执行层。

但Bash的问题在于**太自由了**。

当你给Agent一个Bash终端，它实际上拥有了 root 权限（至少在沙箱外是这样）。这意味着：

* **权限控制困难**：你想让Agent读文件，但不想让它删文件？Bash做不到精确控制。
* **认证管理混乱**：多个Agent共享登录状态？不同团队需要不同工具集？Bash没有内建的认证层。
* **操作审计缺失**：Agent到底执行了什么？Bash日志不是为AI场景设计的。
* **审批流程笨重**：想让读操作自动执行，写操作需要人工确认？在Bash层面实现这个很痛苦。

对于个人开发者，Bash可能够用。但对于企业场景，比如销售团队需要Salesforce工具、工程师需要AWS工具，Bash的权限模型就显得力不从心了。

---

## 三、TypeScript方案：用"写代码"替代"调工具"

Cloudflare在2025年提出了一个新思路：**Code Mode**。

核心想法很简单：与其给Agent几百个工具定义，不如只给它一个工具——**写TypeScript代码**。

工作流程是这样的：

1. MCP服务器提供TypeScript类型定义，描述可用的工具
2. Agent写一段TypeScript代码，调用这些工具
3. 代码在沙箱中执行，通过RPC调用MCP服务器
4. 执行结果返回给Agent

**一个工具，替代几百个工具定义。**

### Cloudflare的实测数据

Cloudflare用Code Mode覆盖了整个Cloudflare API——**超过2500个端点**。传统方式需要把每个端点都暴露为一个工具，Agent的上下文窗口会被**117万个Token**的工具定义塞满。

用Code Mode呢？**约1000个Token**。

这是**99.9%的压缩率**。

### 为什么TypeScript有效？

**模型友好**：大语言模型在训练数据中见过大量TypeScript代码，它们天生擅长写TypeScript——这和Bash的情况类似。区别在于TypeScript有更好的类型安全和结构化能力，让模型能更好地理解和推理。

**上下文效率**：Agent只需要理解"写TypeScript代码"这一个工具，而不是几百个API端点的签名。想象一个普通TypeScript项目——即使是一个小项目，也有数百个函数可调用，但模型处理起来毫不费力，因为它已经习惯了这门语言。

**链式执行**：传统方式下，Agent每调用一个工具就要等待返回，一个5步任务需要5轮对话。用TypeScript，Agent可以一次写完5步代码，一轮完成。这不仅省Token，还减少模型在长对话中"迷失"的风险——每轮对话都可能累积幻觉和误差。

---

## 四、Executor：一个实际的TypeScript执行层

独立开发者RhysSullivan正在做一个叫**Executor**的项目，把Code Mode的想法落地。

Executor给Agent提供：

* 一个TypeScript运行时
* 一个可发现的工具目录
* 一个统一的入口，连接外部系统（OpenAPI、GraphQL、MCP服务器）

### 实战案例：用Executor查询Vercel生产环境错误

实际测试中，开发者让Agent用Executor查询Vercel API，获取生产环境的错误日志。任务描述是："找到davis7.sh网站的所有最新失败部署，打印出生产环境中的当前错误"。

**传统方式的问题**：Vercel API有几百个端点。如果每个端点都是一个工具，Agent会被工具定义淹没，根本无法有效工作。

**Executor的方式**：Agent写了一段TypeScript代码，直接调用Vercel SDK：

```
// Agent自己写的代码（简化版）
const project = await vercel.projects.getProject({ projectId });
const deployments = await vercel.deployments.getDeployments({ projectId });
const latestDeployment = deployments[0];
const logs = await vercel.deployments.getDeploymentEvents({
deploymentId: latestDeployment.id,
direction: 'backward',
limit: 100
});
// 过滤出errors和warnings
const errors = logs.filter(log => log.type === 'error' || log.type === 'warning');
```

代码里有try-catch、有类型过滤、有错误处理——这些都是Agent自己写的，因为它擅长写代码。

最终结果：Agent成功获取了部署失败信息、构建警告（未使用的输入、循环依赖、大chunk警告）、运行时错误。一轮对话完成，而不是传统方式需要的多轮工具调用。

### 这个方案的代价

当然，TypeScript执行层也有代价：

* **沙箱复杂度**：需要安全的代码执行环境，这在云端部署时比简单的HTTP请求复杂得多。Daytona、E2B等平台提供了现成的沙箱解决方案，它们甚至支持Webhook来同步沙箱状态
* **权限认证**：需要在代码执行层处理，而不是简单的API密钥管理
* **流式响应挑战**：某些场景（如Server-Sent Events流式响应）用代码模拟不如原生API方便

这些代价对于简单场景可能是过度工程，但对于需要精细控制的企业级应用，完全值得。

---

## 五、执行层的演进：从工具到代码

回顾AI Agent执行层的发展：

**第一代：直接给工具**
 Agent拿到一堆工具定义，自己判断该调用哪个。简单直接，但工具一多就撑爆上下文——模型要在几百个选项中做决定，判断力直线下降。

**第二代：Bash终端**
 Agent拿到一个Bash工具，自己写命令执行。Bash本身就是一个完整的发现和执行系统——用`which`找工具、用管道链式执行、处理文件系统。Agent自由度高了很多，但权限管理失控：读/写/删除没有精确控制，认证和审计都是难题。

**第三代：TypeScript代码**
 Agent拿到一个"写TypeScript"工具，自己写代码调用工具。上下文只有一套TypeScript类型定义，工具发现靠代码补全，权限控制在执行层处理。Cloudflare用Code Mode覆盖2500+ API端点只用约1000 Token；Uber的内部MCP网关也在往这个方向探索。

这不是说Bash会被完全替代。对于简单的本地任务，Bash仍然好用。但对于需要精细权限控制、多工具协作、企业级认证的场景，TypeScript方案显然更合适——它不是更"聪明"，而是更**可编排**。

---

## 六、现在该怎么做？

如果你正在构建AI Agent应用，可以考虑：

**评估你的工具数量**：如果Agent需要调用的工具超过20个，传统方式可能已经开始影响模型表现。

**尝试Code Mode思路**：即使不用完整的Executor方案，也可以考虑把多个工具封装成一个TypeScript SDK，让Agent通过代码调用，而非暴露几百个独立工具。

**关注沙箱方案**：TypeScript执行需要安全的沙箱环境。Daytona、E2B等平台提供了现成的解决方案，它们支持Webhook来同步沙箱状态、管理生命周期。

**保持观望**：这个领域还在快速演进。TypeScript可能不是最终答案，但它代表的方向——**让Agent写代码而不是调工具**——很可能是对的。

---

Bash给了Agent自由，但自由过头就会变成混乱。TypeScript给了Agent结构，同时保留了足够的表达能力。在AI Agent的执行层竞赛中，代码可能才是最终的赢家——不是因为它更"智能"，而是因为它更**可控、可审计、可扩展**。