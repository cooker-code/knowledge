# 后端架构
## 知识点入口

- 本模块先看宏观流程，再看文章：[流程化知识点总览](knowledge/07_工程与架构/0701_后端架构/核心知识点/流程化知识点总览.md)。
- 新文章必须先归入流程节点，再判断是补充、冲突、不同层次还是降权。
- `文章/` 只保留原文锚点，长期知识必须沉淀到 `核心知识点/`。


## 目录目的

本目录不维护独立来源汇总文件，而是按后端实现语言建立“应用流程入口”。

每个语言目录的 `AGENTS.md` 都应该回答：

1. 该语言后端应用从开发到运行有哪些流程节点。
2. 每个核心知识点对应哪个流程节点。
3. 当前文章对哪个节点做了补充、冲突、降权或更好的方式。
4. 新文章来了，应该先路由到哪个节点，再和已有沉淀对比。
5. 文章文件放在对应语言目录的 `文章/` 下，只作为来源锚点，不单独形成来源汇总文件。

## 三条语言路线

| 路线 | 流程入口 | 当前状态 |
|---|---|---|
| Python | [Python/AGENTS.md](Python/AGENTS.md) | 当前主线是 FastAPI：应用入口、路由、Pydantic 校验、依赖注入、SQLAlchemy、实时通信、配置、镜像部署 |
| Java | [Java/AGENTS.md](Java/AGENTS.md) | 当前主线是 Spring Boot：Maven、分层、入口适配、参数校验、幂等、应用服务、DDD、端口适配器、批处理、插件化 |
| Node | [Node/AGENTS.md](Node/AGENTS.md) | 当前证据不足：已有文章只能校准 Next/Nuxt 渲染边界、TypeScript 运行时校验缺口、序列化和前端状态持久化风险 |

## 统一处理规则

| 规则 | 说明 |
|---|---|
| 先有流程节点 | 不从文章反推目录；先定义后端服务的主流程 |
| 核心知识点挂节点 | 一个知识点必须说明服务流程中解决哪个环节 |
| 文章按节点吸收 | 新文章先判断优化哪个节点，再和节点已有沉淀对比 |
| 对比优先于摘要 | 重点写补充、冲突、不同层次、更好方式，而不是复述文章 |
| 文章本地化 | 被引用文章必须放在语言目录的 `文章/` 下 |
| 缺节点先补节点 | 如果文章确实有价值但没有对应节点，先调整 `AGENTS.md` 流程 |

## 跨语言对比入口

| 流程问题 | Python | Java | Node |
|---|---|---|---|
| 项目边界 | FastAPI 应用结构、配置、依赖 | Maven 模块、Spring Boot 项目边界 | Node 运行时、BFF、SSR/API 边界 |
| 入口适配 | Router、Depends、中间件 | Controller、Scheduler、Consumer、JobLauncher | Router、Controller、Middleware |
| 请求校验 | Pydantic / SQLModel | Bean Validation / DTO | TypeScript 不够，需 Zod/class-validator |
| 业务编排 | Service 层 | Application Service / UseCase | Service / UseCase |
| 数据访问 | SQLAlchemy 2.0 / SQLModel | Repository / JPA / MyBatis | Prisma / TypeORM / Drizzle，当前缺来源 |
| 实时通信 | WebSocket / Socket.IO / SSE | WebSocket / MQ / 服务调用治理 | WebSocket / Socket.IO，当前缺来源 |
| 可靠性 | 连接池、事务、中间件、镜像 | 幂等、超时、fallback、批处理、插件隔离 | 当前缺限流、任务队列、观测和部署来源 |

## 新文章路由方式

新文章进入后端架构目录时，先判断它属于哪种语言，再进入对应语言 `AGENTS.md` 的流程节点：

```text
文章主问题
  -> 语言路线: Python / Java / Node
  -> 流程节点: Pxx / Jxx / Nxx
  -> 读取节点已有沉淀
  -> 读取对应核心知识点
  -> 判断补充 / 冲突 / 不同层次 / 更好的方式 / 低价值
  -> 更新核心知识点和 AGENTS 节点
```

## 当前补证优先级

| 优先级 | 方向 | 原因 |
|---|---|---|
| P0 | Node 真实服务端框架 | 当前 Node 证据不足，不能指导真实 API 后端 |
| P0 | Java Maven、DDD、安全、测试、观测 | Java 当前流程完整，但若要指导真实开发还缺生产关键节点 |
| P1 | Python 后台任务、安全、可观测性、部署 | FastAPI 路线已有基础，但缺生产治理 |
| P1 | 跨语言通用后端问题 | 幂等、事务、权限、观测、部署可以横向对比三种语言实现 |
