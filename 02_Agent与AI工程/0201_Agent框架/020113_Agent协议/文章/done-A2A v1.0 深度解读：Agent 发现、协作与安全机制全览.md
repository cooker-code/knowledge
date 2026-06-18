> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020113_Agent协议/020113_核心知识点/A2A发现协作与安全边界|A2A发现协作与安全边界]]
---
title: A2A v1.0 深度解读：Agent 发现、协作与安全机制全览
author: 诶谨特
date: 技术趋势技术趋势
url: https://mp.weixin.qq.com/s?__biz=MzcwOTI1Mzc3Mw==&mid=2247483736&idx=1&sn=a26f9af9aaf1d7f306184dc9d1b96eb0&chksm=f40cd81410359c6d44dd963bff90483e3147afc5178a8c4ee3f53415742ad81a79a95cc9e504&mpshare=1&scene=24&srcid=0518KLjk2hNnGWmo1SijXCfI&sharer_shareinfo=62875ae69355cad75ceb38fec727515f&sharer_shareinfo_first=62875ae69355cad75ceb38fec727515f#rd
---

2025 年 4 月，Google 发布 A2A（Agent-to-Agent）协议；随后捐赠给 Linux Foundation。一年后，支持组织从 50+ 增长到 150+，GitHub Star 超过 23K，v1.0 稳定规范发布，Azure、AWS、GCP 也都公开了集成进展。A2A 至少已经从一个“新提法”，走到了值得严肃研究的协议层。

这篇文章不打算复述新闻，而是回到协议本身：A2A 为什么出现、v1.0 到底改了什么、生产环境里应该怎么理解和使用它。

---

## 1. 为什么需要 A2A：Agent 互操作的三层困境

在 A2A 出现之前，构建跨框架、跨组织的多 Agent 系统面临三层困境：

**框架锁定层**：LangGraph、CrewAI、AutoGen、Google Agent Builder——每个框架都有自己的 Agent 通信方式。一个 LangGraph 的 Agent 无法直接调用 CrewAI 的 Agent，除非写大量胶水代码。

**组织边界层**：企业 A 的供应链 Agent 和企业 B 的物流 Agent 需要协作，但分属不同信任域、不同技术栈、不同安全策略。没有标准协议，只能点对点定制集成。

**经济协调层**：Agent 之间不仅需要通信，还需要交易。当 Agent A 委托 Agent B 执行付费任务时，如何保证支付的安全性和可审计性？

A2A 的核心洞察是：**Agent 互操作问题本质上是协议问题，不是 API 问题**。正如 TCP/IP 统一了异构网络的通信，A2A 要统一异构 Agent 的协作。

---

## 2. 协议设计深度解析

### 2.1 Agent Card：Agent 的数字名片

Agent Card 是 A2A 发现机制的核心。它是一个 JSON 文档，用来描述 Agent 的身份、接口、能力和认证要求。

**发现路径**：`/.well-known/agent-card.json`（遵循 RFC 8615 Well-Known URI 惯例）

**Agent Card 核心字段**：

```
{
"name":"research-agent",
"description":"检索最新文献和技术趋势的专业 Agent",
"version":"1.0.0",
"supportedInterfaces":[
{
"url":"https://agents.example.com/a2a/v1",
"protocolBinding":"JSONRPC",
"protocolVersion":"1.0"
}
],
"capabilities":{
"streaming":true,
"pushNotifications":true
},
"skills":[
{
"id":"literature-search",
"name":"文献检索",
"description":"根据关键词检索学术论文和技术文档",
"tags":["research","papers","search"],
"inputModes":["text/plain"],
"outputModes":["text/plain","application/json"]
}
],
"defaultInputModes":["text/plain"],
"defaultOutputModes":["text/plain"],
"securitySchemes":{
"bearerAuth":{
"httpAuthSecurityScheme":{
"scheme":"Bearer"
}
}
},
"security":[
{
"bearerAuth":[]
}
]
}
```

**v1.0 关键变化——Signed Agent Cards**：v1.0 引入了基于 JWS（JSON Web Signature，RFC 7515）和 JSON Canonicalization Scheme（RFC 8785）的签名验证。客户端可以在正式交互前校验 Agent Card 的真实性和完整性，这对跨组织协作尤其关键。

**踩坑提醒**：`defaultInputModes` / `defaultOutputModes` 必须使用完整 MIME 类型 `text/plain`，不能简写为 `text`。这是 v1.0 规范的严格要求，很多早期实现都踩过这个坑。

另一个容易混淆的点是：v1.0 的 Agent Card 结构和早期版本差异很大。像顶层 `url`、`protocolVersion`、`preferredTransport` 这类旧字段，在 v1.0 里已经收敛到 `supportedInterfaces`。

### 2.2 通信协议：三种协议绑定

A2A v1.0 支持三种协议绑定（Protocol Bindings），语义等价：

| 绑定方式 | 适用场景 | 说明 |
| --- | --- | --- |
| JSON-RPC over HTTP | 最通用 | 入门最简单，适合快速接入 |
| gRPC | 高性能场景 | 强类型、流式通信、更适合复杂服务间调用 |
| HTTP+JSON/REST | Web 友好 | 标准 HTTP 语义，便于网关和现有基础设施接入 |

**核心操作映射（v0.3 → v1.0 重命名）**：

| v0.3 操作 | v1.0 操作 | 说明 |
| --- | --- | --- |
| `message/send` | `SendMessage` | 发送消息，返回 Task 或 Message |
| `message/stream` | `SendStreamingMessage` | 流式发送，实时返回进度 |
| `tasks/get` | `GetTask` | 查询任务状态 |
| `tasks/list` | `ListTasks` | 列出任务（v1.0 新增，cursor 分页） |
| `tasks/cancel` | `CancelTask` | 取消任务 |
| `tasks/resubscribe` | `SubscribeToTask` | 重新订阅任务流 |
| `agent/getAuthenticatedExtendedCard` | `GetExtendedAgentCard` | 获取需认证的扩展 Agent Card |

**v1.0 Breaking Changes 速览**：

* 枚举值遵循 ProtoJSON 规范，采用 `SCREAMING_SNAKE_CASE`，例如 `TASK_STATE_COMPLETED`
* 移除 `kind` 鉴别字段，改用 JSON 成员名做类型判别（如 v0.3 的 `{"kind": "text", "text": "Hello"}` 变为 `{"text": "Hello"}`）
* 移除 `TaskStatusUpdateEvent` 的 `final` 布尔字段，改用协议绑定的流关闭机制
* ID 格式简化：移除复合 ID（如 `tasks/{id}`），改用简单 UUID

### 2.3 安全模型：Web 对齐而非重新发明

A2A 的安全设计哲学是：**不发明新的安全机制，复用 Web 生态已验证的方案**。

**认证层次**：

```
┌─────────────────────────────────────────────┐
│  Transport Security: HTTPS + TLS 1.3+       │
│  ├── Server Identity: TLS 证书验证           │
│  └── Client Auth: HTTP 层凭证传输            │
│       ├── API Key (X-API-Key header)         │
│       ├── OAuth 2.0 Bearer Token             │
│       │    ├── Authorization Code + PKCE     │
│       │    ├── Device Code Flow (RFC 8628)   │
│       │    └── OIDC Discovery                │
│       └── Mutual TLS                         │
├─────────────────────────────────────────────┤
│  Agent Card Security:                        │
│  └── Signed Agent Cards (JWS + JCS)         │
├─────────────────────────────────────────────┤
│  In-Task Auth:                               │
│  └── 任务执行中的二次凭证验证                  │
└─────────────────────────────────────────────┘
```

**认证流程**：

1. 客户端通过 Agent Card 的 `securitySchemes` 和 `securityRequirements` 字段发现服务端要求的认证方案
2. 通过 Out-of-Band 方式获取凭证（API Key、OAuth Token 等）
3. 在每次 HTTP 请求的 Header 中携带凭证（如 `Authorization: Bearer <token>`）
4. 服务端验证后，根据客户端身份和请求的 skill 进行授权

**OAuth 2.0 vs API Key**：

| 维度 | API Key | OAuth 2.0 |
| --- | --- | --- |
| Token 生命周期 | 通常较长 | 短期访问令牌 + 刷新机制 |
| 授权能力 | 粗粒度 | 可做细粒度 scope 控制 |
| 安全姿态 | 较弱 | 更适合标准化生产接入 |
| 生产推荐 | ❌ | ✅ |
| 适用场景 | 开发/测试 | 生产环境 |

v1.0 新增了 PKCE 支持和 Device Code Flow（RFC 8628），安全姿态显著提升。

### 2.4 流式协作：Agent 的实时对话

A2A 原生支持三种结果消费模式：

1. **轮询**：客户端定期调用 `GetTask` 检查状态。简单通用，但延迟高
2. **流式**：通过 `SendStreamingMessage` 实时接收 `TaskStatusUpdateEvent` 和 `TaskArtifactUpdateEvent`。具体流式机制取决于协议绑定（JSON-RPC 用 SSE，gRPC 用原生 streaming）
3. **Webhook 推送**：客户端注册回调 URL，服务端在任务状态变更时主动推送。适合超长任务

流式协作的关键特性：

* **可中断**：客户端随时可以发送 `CancelTask`
* **可迭代**：Agent 在执行过程中可以请求更多输入（`TASK_STATE_INPUT_REQUIRED` 状态）
* **多流并发**：v1.0 明确允许多个并发流，所有流接收相同顺序的事件

---

## 3. AP2 支付协议：Agent 经济的基础设施

与 A2A 同期推进的，还有 **AP2（Agent Payments Protocol）**。它关注的不是 Agent 之间“怎么说话”，而是 Agent 发起交易时“如何证明授权、如何保留证据、如何进入既有支付体系”。

**AP2 的核心思路——把用户授权变成可验证证据**：

AP2 官方文档更常用的表述是各种可验证授权凭证（Verifiable Credentials）；Linux Foundation 一周年新闻稿则提到 AP2 Mandates Extension。两者都指向同一件事：把“用户确实授权了这笔交易”变成可校验、可追溯的证据链。常见场景有两类：

* **人在场（Real-time）**：用户发出购物请求，Agent 生成意图授权，用户确认购物车后再签署最终授权，再进入支付执行
* **人不在场（Delegated）**：用户预先设定价格上限、时间条件等约束，Agent 在条件满足时按授权范围发起交易

**AP2 生态**：

* 60+ 支付和金融机构支持，包括 Adyen、American Express、Mastercard、PayPal、UnionPay International 等
* UCP（Universal Confirming Party）通过 AP2 Mandates Extension 兼容，能捕获用户购买意愿的强密码学证据
* 适用于高信任、强监管的金融环境

AP2 的意义在于：**Agent 经济不只是技术问题，更是信任问题**。没有标准化的支付协议，Agent 之间的经济协作只能停留在 PoC 阶段。

---

## 4. 云平台集成现状：Azure/AWS/GCP 已公开集成

| 云平台 | 集成方式 | 特色 |
| --- | --- | --- |
| **Microsoft Azure** | Azure AI Foundry + Copilot Studio | 官方已公开集成进展 |
| **AWS** | Amazon Bedrock AgentCore Runtime | 面向企业运行时场景 |
| **Google Cloud** | Vertex AI Agent Builder | Google 体系内重点推进 |

更稳妥的说法是：三大云厂商都已经公开站队并给出产品层集成信号。它还谈不上尘埃落定的“行业标准”，但至少已经不是实验室讨论。

---

## 5. A2A vs MCP vs OpenAPI：三层协议栈的分工与协作

这是社区最常困惑的问题。简单说：

```
┌─────────────────────────────────────────────────┐
│  A2A  — Agent ↔ Agent（互联网层）                │
│  跨组织边界通信协调，Agent 发现、协商、协作        │
├─────────────────────────────────────────────────┤
│  MCP  — Agent ↔ Tool/Data（工具层）              │
│  连接 Agent 到内部工具和数据源                     │
├─────────────────────────────────────────────────┤
│  OpenAPI — Service ↔ Service（服务层）            │
│  REST API 接口定义，HTTP 方法与资源模型            │
└─────────────────────────────────────────────────┘
```

**为什么是互补而非替代**：

MCP 解决的是一个 Agent 内部的问题：如何连接数据库、API、文件系统等工具。A2A 解决的是多个 Agent 之间的问题：如何发现彼此、协商能力、协作完成任务。A2A 官方一直把 MCP 定位为互补关系，而不是替代关系。

**实际架构中两者共存**：

```
Agent A (内部用 MCP 连接工具) ──A2A──→ Agent B (内部用 MCP 连接工具)
   │                                      │
   ├── MCP → PostgreSQL                    ├── MCP → REST API
   ├── MCP → Slack API                     ├── MCP → S3 Storage
   └── MCP → Internal Service              └── MCP → Payment Gateway
```

正如 Google Distinguished Engineer Todd Segal 所说：*"A2A provides the secure foundation for personal, team, and domain-specific agents to work together seamlessly across any platform."*

---

## 6. 生产部署指南

### 6.1 认证配置要点

生产环境推荐：

* 传输层：HTTPS + TLS 1.3+，强密码套件
* 客户端认证：OAuth 2.0（Authorization Code + PKCE），避免使用 API Key
* Agent Card：启用 Signed Agent Cards（JWS + JCS RFC 8785）
* v1.0 新增 Device Code Flow（RFC 8628），适用于无浏览器的 IoT/CLI 场景

### 6.2 多租户部署

v1.0 原生支持多租户：单端点可以安全托管多个 Agent，请求里也可以携带租户作用域。规范还明确要求服务端只能返回对调用方可见的任务，这一点对企业场景很重要。

### 6.3 监控与可观测性

A2A 的 Web 对齐架构意味着你可以直接复用现有的基础设施监控：

```
┌─ Load Balancer ─→ ┌─ API Gateway ─→ ┌─ A2A Server ─→ Agent
   (健康检查)          (限流/认证)       (业务逻辑)
       │                  │                 │
       └──── Observability Stack ───────────┘
            (Logs / Metrics / Traces)
```

**关键指标**：

* Agent 发现延迟（Agent Card 获取时间）
* Task 生命周期各阶段耗时
* 认证失败率
* 流式通信的中断率

### 6.4 分页与大规模任务管理

v1.0 新增 `ListTasks` 操作，采用基于游标的分页（比页码分页更适合大规模任务列表）：

```
// 以 JSON-RPC 调用为例
ListTasks({ limit: 50 })
→ 返回 tasks + nextCursor

ListTasks({ cursor: "<nextCursor>", limit: 50 })
→ 下一页
```

---

## 7. 生态全景：从 SDK 到 Inspector 到 TCK

**SDK 生态**（目前覆盖 5 种生产可用语言）：

* **Python**（`pip install a2a-sdk`）和 **JavaScript/TypeScript**（`npm install @a2a-js/sdk`）是目前最成熟的两套
* **Java**、**Go**、**.NET** 也已有可用实现，A2A 官方站点已将其列入 SDK 下载入口

**开发者工具**：

* **A2A Inspector**：调试 Agent Card 和交互流程
* **TCK（Technology Compatibility Kit）**：验证实现的协议兼容性
* **DeepLearning.AI 短课程**：系统学习 A2A 协议

**标准组织**：A2A 技术指导委员会（TSC）由 8 家公司代表组成——AWS、Cisco、Google、IBM Research、Microsoft、Salesforce、SAP、ServiceNow。这个阵容确保了协议不会偏向任何单一厂商。

**跨框架兼容**：官方与社区材料都在反复强调，LangGraph、CrewAI、Google Agent Builder 等不同栈上的 Agent 已经可以围绕 A2A 互通。更准确地说，A2A 正在成为这些框架之间的“共同语言”，而不是要求大家迁移到同一个框架。

---

## 8. 看法与展望：A2A 的挑战与未来

### 值得肯定的

1. **Web 对齐的设计哲学**：不重新发明轮子，复用 HTTP、JSON-RPC、OAuth、TLS 等成熟方案。这让 A2A 的上手门槛比很多“全新协议”低得多，也天然继承了 Web 基础设施的安全、扩缩容和可观测性能力。
2. **渐进式迁移策略**：Agent Card 后向兼容，允许同时声明 v0.3 和 v1.0 支持。这保护了早期投入，避免了强制迁移的痛苦。
3. **从通信到经济的完整视野**：A2A 之外，生态又把支付授权这块单独抽出来做 AP2，说明大家已经开始认真面对 Agent 经济中的责任、审计和争议处理问题。
4. **互补而非替代**：明确与 MCP 的分工，至少避免了“一个协议包打天下”的叙事陷阱。

### 仍需观察的

1. **语义互操作性**：A2A 解决了语法层的互操作（消息格式、传输协议），但语义层（Agent 如何理解彼此的技能描述、如何处理意图歧义）仍有大量工作。当前 Agent Card 的 `skills` 描述是自由文本，缺乏标准化本体。
2. **Registry 的集中化风险**：Agent 发现目前依赖 Well-Known URI 和私有注册中心。公共 Registry 如何避免成为单点故障和治理瓶颈？路线图中提到了 Registry 规范的整合，但细节待定。
3. **长尾 Agent 的可发现性**：150+ 组织听起来很多，但与互联网上的 Agent 总量相比仍是九牛一毛。如何让小团队、个人开发者构建的 Agent 被发现和使用？
4. **安全验证的实际复杂性**：Signed Agent Cards 和 OAuth 2.0 在纸面上都很完整，但一旦进入跨组织协作，PKI 信任链、Token 交换、密钥轮换、运维责任边界都会迅速变复杂。

### 展望

A2A 路线图中的关键方向：

* **互操作规范**：定义不同实现之间的一致性要求
* **Registry 规范整合**：统一 Agent 发现的注册中心标准
* **扩展的测试和工具链**：降低合规实现的门槛
* **安全和部署最佳实践**：从规范到实战的桥梁

Cisco Distinguished Engineer Luca Muscariello 说得好：*"A2A has emerged as the syntactic layer that makes agent-to-agent communication reliable and interoperable. What's most exciting is that this is just the beginning — there is enormous opportunity ahead to build richer semantics and interaction models that can turn an Internet of Agents into a planet-scale reality."*

一年从 0 到 v1.0，A2A 证明了开放协议在 Agent 互操作领域的可行性。下一个挑战是从"能通信"到"能理解"——那才是 Agent 互联网真正成型的时候。