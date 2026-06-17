# Flink CDC 整库同步 Doris 链路边界

## 原文锚点

- 本地文件：[Flink CDC 3.0 构建 MySQL 整库同步 Doris](../文章/Flink CDC 3.0 构建 MySQL 整库同步 Doris.md)
- 原文链接：http://mp.weixin.qq.com/s?__biz=MzA5MDQyNjMxMg==&mid=2649593820&idx=1&sn=edd429f5914a812ae1f2bde029c01d77
- 官方锚点：[Apache Flink CDC Documentation](https://nightlies.apache.org/flink/flink-cdc-docs-stable/docs/get-started/introduction/)、[Apache Doris](https://doris.apache.org/)
- 关键段落：Flink Standalone、Flink CDC Pipeline connector、`mysql-to-doris.yaml`、`server-id`、Doris FE 节点和自动建表属性。
- 关键图：原文截图缺失，不影响主链路。

## 图片处理

| 图片 | 类型 | 是否保留 | 理由 | 处理方式 |
|---|---|---|---|---|
| Flink Web UI 截图 | 截图 | 删除 | 只证明任务提交，不解释机制 | 不进入知识点 |
| MySQL 到 Doris 同步链路 | 架构图 | 重建 | 帮助判断归类和上下游 | Mermaid 重建 |

```mermaid
flowchart LR
  MySQL["MySQL\n全量快照 + Binlog"] --> CDC["Flink CDC Pipeline\nmysql source"]
  CDC --> Flink["Flink Runtime"]
  Flink --> Doris["Doris Sink\nFE/BE"]
  Doris --> Query["报表 / 即席查询 / 服务化分析"]
```

## 一句话结论

这篇文章只适合精读其中的链路骨架：MySQL 到 Doris 的主问题是数据集成链路，不是 Doris 查询优化；安装步骤和版本参数都不能直接沉淀为准则。

## 用户相关性判断

| 项 | 内容 |
|---|---|
| 用户当前认知层级 | Flink CDC / 数据集成 L2-L3 draft；Doris / OLAP L2 draft |
| 认知成熟度 | draft |
| 阅读投入建议 | 略读到精读之间 |
| 阅读投入理由 | 能补 MySQL -> Doris Pipeline 的架构位置，但教程化内容多，缺 Schema 演进、Exactly Once、失败恢复和 Doris 导入验收 |
| 对用户的新信息 | `Flink CDC -> Doris` 应优先按同步链路归数据集成，Doris 只是下游 Sink 和查询出口 |
| 问题指纹 | Flink CDC + MySQL-to-Doris Pipeline + YAML Source/Sink/server-id/auto create table + 整库同步链路边界 + 不等同 Doris 查询优化 |
| 排重判断 | 新建 |
| 置信度 | 中 |

## 认知校准点

| 校准点 | 文章观点/信息 | 与用户认知或价值观的关系 | 处理建议 |
|---|---|---|---|
| 下游是 Doris 不代表归 OLAP | 文章实际讲 Flink CDC Pipeline 同步 MySQL 到 Doris | 纠偏脚本粗路由 | 归数据集成，Doris index 只做关联 |
| 教程不等于实践 SOP | 有安装和 YAML，但无故障、验收、监控 | 避免误判实践 | 只沉淀链路骨架 |
| `server-id` 又出现 | MySQL Source 仍需要管理复制身份范围 | 与已有 server_id 排障互证 | 追加到 Flink CDC 生产准则 |
| Doris Sink 需要看导入语义 | 自动建表和 light schema change 不等于端到端一致性 | 补下游一致性缺口 | 后续查 Doris Sink 文档 |

## 冲突点

| 冲突类型 | 具体表现 | 影响 | 处理 |
|---|---|---|---|
| 原目录冲突 | 粗路由可能因 Doris 归到 OLAP | 错把同步链路当查询引擎文章 | 重路由到 Flink CDC |
| 实践门槛不足 | 缺输入数据、结果校验、失败恢复和监控指标 | 不能判实践 | 降为精读 |
| 版本时效 | Flink 1.18、Flink CDC 3.0 connector 路径可能过时 | 复制命令风险高 | 官方文档复核 |
| 证据不足 | 没有对 Doris 表模型、主键、幂等、DDL 冲突说明 | 下游正确性未证明 | 写入后续追查 |

## 待吸收点

| 分级 | 内容 | 为什么值得吸收 | 后续动作 |
|---|---|---|---|
| 理解 | MySQL -> Flink CDC Pipeline -> Doris 是数据同步链路 | 确认归类边界 | 更新数据集成 index |
| 理解 | YAML 里 Source、Sink、Pipeline 三段对应上游、下游和运行参数 | 与 Flink CDC 3.x 架构笔记呼应 | 合并到 CDC Pipeline 认知 |
| 记住 | 下游 Doris 的表模型、主键和 Schema 变化决定同步正确性 | 影响落地质量 | 后续补 Doris Sink 限制 |
| 实践 | 跑 MySQL -> Doris 最小链路，验证新增表、新增列、删除事件和重启恢复 | 能验证生产可用性 | 待实验 |

## 已知可跳过

| 内容 | 跳过理由 |
|---|---|
| SSH 免密、Flink Standalone 安装步骤 | 通用部署知识，且版本时效强 |
| Web UI 截图 | 不提供机制和验证 |
| 固定 IP、端口和密码示例 | 不可复用 |

## 实践门槛

| 门槛 | 判断 | 证据 |
|---|---|---|
| 可运行 | 部分 | 有 YAML 和提交命令 |
| 可验证 | 否 | 没有同步前后数据、Doris 表结构和校验 SQL |
| 可排障 | 否 | 没有 checkpoint、DDL、导入失败、重复写入处理 |
| 可迁移 | 部分 | 链路方向可迁移，参数需重查 |
| 结论 | 降为精读 | 只作为链路边界，不作为部署 SOP |

## 归类判断

| 项 | 内容 |
|---|---|
| 技术本体 | Flink CDC Pipeline |
| 文章主问题 | MySQL 整库同步到 Doris 的任务配置 |
| 使用场景 | 实时数仓/湖仓到 OLAP 查询出口 |
| 关键词干扰 | Doris、Flink Standalone、实战 |
| 最终归类 | 数据工程与数仓 / 数据集成 / Flink CDC |
| 归类理由 | 主职责是数据进入 Doris 的同步链路，不是 Doris 查询、索引或执行引擎 |

## 技术定位

| 项 | 内容 |
|---|---|
| 技术类型 | 教程型链路案例 |
| 所属领域 | 数据工程与数仓 |
| 二级类目 | 数据集成 |
| 全局架构位置 | 业务库到 OLAP 引擎的数据同步入口 |
| 涉及模块 | MySQL Source、Doris Sink、Flink Runtime、Pipeline YAML |
| 解决问题 | 快速搭建 MySQL 到 Doris 整库同步 |
| 原文局限 | 只讲跑起来，不讲正确性、恢复和生产边界 |
| 我的结论 | 仅作为链路锚点，后续需官方和实验验证 |

## 纵向理解

| 维度 | 判断 |
|---|---|
| 全局架构 | MySQL 日志 -> Flink CDC Pipeline -> Doris 导入 -> 查询服务 |
| 本文位置 | 只讲 Pipeline 配置和提交 |
| 核心机制 | Source/Sink 声明式连接和 Flink 作业运行 |
| 使用链路 | 准备 connector -> 写 YAML -> 提交任务 -> Doris 建表/写入 |
| 前置条件 | MySQL 权限、唯一 `server-id`、Doris FE/BE 可用、表模型设计 |
| 边界 | 不覆盖数据校验、DDL 冲突、导入幂等、Exactly Once 和补数 |

## 横向对标

| 对标技术 | 实现方式 | 优势 | 劣势 | 适合场景 |
|---|---|---|---|---|
| Flink CDC Pipeline -> Doris | YAML 声明同步链路 | 贴近 Flink CDC 3.x，整库同步友好 | 版本和 Sink 限制需核实 | 新建 CDC 到 Doris 链路 |
| Stream Load/Routine Load | Doris 原生导入 | Doris 生态成熟 | 源端 CDC 和 Schema 管理需额外处理 | Kafka/文件到 Doris |
| SeaTunnel -> Doris | 通用数据集成 | 多源多端更通用 | CDC 语义需看连接器 | 企业同步平台 |
| DataX -> Doris | 离线批导入 | 简单稳定 | 不适合实时增量 | T+1 批同步 |

## 后续追查

- 关键词：Flink CDC Doris Pipeline Connector、Doris Sink、light_schema_change、MySQL full database synchronization。
- 相关技术：Doris Stream Load、Routine Load、Unique Key/Primary Key、Flink Checkpoint。
- 需要补读的文章：Flink CDC Doris Sink 官方文档、Doris 导入事务语义、MySQL -> Doris DDL 演进处理。

