# Gravitino AI元数据与上下文工程边界

> 验证版本：Gravitino 1.0+，具体能力以官方 Release 为准

## 来源
- [Apache Gravitino 1.0.0：从元数据管理到智能上下文工程](<../文章/done-Apache Gravitino 1.0.0：从元数据管理到智能上下文工程.md>)
- [基于 Gravitino 的 AI 元数据管理系统设计与实践](<../文章/done-基于 Gravitino 的 AI 元数据管理系统设计与实践.md>)
- [Apache Gravitino 在B站的最佳实践](<../文章/done-Apache Gravitino 在B站的最佳实践.md>)

## 核心问题

Gravitino 文章的认知增量是：统一元数据平台正在从“Catalog/目录服务”扩展到策略、统计、任务、MCP 工具和上下文工程。但这些能力仍要按版本和治理动作验证。

## 判断准则

| 判断项 | 准则 |
|---|---|
| Metadata Lake | 关注多源元数据统一、Metalake/Schema/Table/FileSet/Model 等对象抽象 |
| AI 上下文 | 只有能稳定提供 schema、owner、权限、质量、血缘和业务语义时，才适合进入 Agent 上下文 |
| MCP/工具化 | MCP 只是调用入口，底层 API 的权限、审计和幂等仍要独立治理 |
| 生产实践 | B 站等实践只吸收统一 Catalog、权限、血缘和平台接入边界，不复制场景口号 |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| AI 元数据管理就是给模型喂表说明 | 可用上下文要包含权限、血缘、质量、owner、版本和动作入口 |
| MCP 化后治理就自动完成 | MCP 只是接口暴露方式，治理仍依赖元模型、策略、审计和执行系统 |

## 待验证缺口

- 验证 Gravitino MCP Server 的工具边界、权限校验和对大规模 schema 查询的延迟。
