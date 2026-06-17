# Java 架构实现路线

## 这个文件的定位

这个文件不再承载所有文章分析。

它只做一件事：

> 把 Java 核心知识点准则串成开发前的决策路线。

具体准则分别沉淀在：

- [Java核心知识点准则.md](Java核心知识点准则.md)
- [标准项目架构与代码组织.md](标准项目架构与代码组织.md)
- [Maven项目管理与模块边界.md](Maven项目管理与模块边界.md)
- [DDD开发规范与落地边界.md](DDD开发规范与落地边界.md)
- [Java作为后端服务的完整流程.md](Java作为后端服务的完整流程.md)

## 开发前决策路线

| 顺序 | 决策 | 输出 |
|---|---|---|
| 1 | 判断项目类型 | CRUD / 核心业务 / 平台型系统 |
| 2 | 选择项目架构 | 轻量分层 / 六边形 / DDD + 六边形 |
| 3 | 选择 Maven 结构 | 单模块 / 多模块 / 多应用 |
| 4 | 定义包结构 | interfaces、application、domain、infrastructure 或简化结构 |
| 5 | 定义领域模型 | 实体、值对象、聚合、领域服务、领域事件 |
| 6 | 定义入口契约 | Controller、DTO、Command、Response、统一异常 |
| 7 | 定义事务边界 | Application Service 负责事务和用例编排 |
| 8 | 定义写入防线 | 参数校验、幂等、乐观锁、唯一键、重复提交控制 |
| 9 | 定义外部依赖边界 | Repository、ExternalClient、MessagePort、超时、fallback |
| 10 | 判断专项能力 | 批处理、插件化、安全、可观测性、部署发布 |

## Java 服务运行流程

详细流程见：[Java作为后端服务的完整流程.md](Java作为后端服务的完整流程.md)。

简化成一条链路是：

```text
外部入口
  -> Controller / Scheduler / Consumer / JobLauncher
  -> DTO 参数校验和统一异常
  -> 幂等与重复请求防线
  -> Application Service 事务和用例编排
  -> Domain 领域规则
  -> Port 输出端口
  -> Adapter 技术实现
  -> 外部依赖超时和 fallback
  -> 批处理或插件化等专项能力
```

## 最小可执行骨架

如果是普通 Spring Boot 业务系统，先从这个结构开始：

```text
src/main/java/com/example/app/
  interfaces/
    rest/
  application/
    command/
    service/
    port/
      outgoing/
  domain/
    model/
    service/
    repository/
  infrastructure/
    persistence/
    external/
    config/
```

如果只是简单后台管理，可以降级为：

```text
src/main/java/com/example/app/
  web/
  service/
  repository/
  integration/
  config/
```

## 和 AGENTS 流程节点的关系

本文件只保留开发前路线。

流程节点、核心知识点映射、新文章如何路由和对比，统一以 [../AGENTS.md](../AGENTS.md) 为准。新文章来了，不新增独立来源汇总文件，而是先判断它优化的是 AGENTS 中的哪个节点，再更新对应核心知识点。

## 现在不能假装已经完整的部分

| 部分 | 原因 |
|---|---|
| Maven 管理 | 当前缺文章和项目样例支撑 |
| DDD 规范 | 当前只有架构分层线索，缺战术模式和代码规范 |
| 安全权限 | 当前没有认证授权主线 |
| 测试质量 | 当前没有测试分层资料 |
| 可观测性和部署 | 当前没有运行期治理资料 |
