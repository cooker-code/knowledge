# 数据集成

> 分类规则、一二级类目定位、跨域归类边界以根 [目录划分.md](../../目录划分.md) 为准。
> 文章理解的四步法、整理流程以根 [AGENTS.md](../../AGENTS.md) 为准。
> 本文件只维护本领域的认知重点、排重指纹、已覆盖三级节点和待补缺口。

## 三级节点入口

| 节点 | 用途 |
|---|---|
| [030701_Debezium](030701_Debezium/AGENTS.md) | CDC 事件采集与 Kafka Connect 集成 |
| [030702_SeaTunnel](030702_SeaTunnel/AGENTS.md) | 批式同步与多源 CDC 集成框架 |

## 用户认知重点

| 项 | 内容 |
|---|---|
| 已知基础 | 用户知道数据同步是数仓入口，无需重复讲"什么是同步" |
| 待补边界 | 数据集成与实时计算、调度、湖仓表格式、OLAP 导入的边界；Flink CDC 已迁入实时计算目录 |
| 易偏差点 | Doris/StarRocks 同步文章易误归 OLAP；CDC 文章易跨域归实时计算 |
| 优先抽取 | 链路语义、失败模式、Schema 演进、下游一致性 |

## 排重指纹

```text
数据集成工具 + 全量/增量/Schema/一致性/下游写入模块 + 核心机制 + 解决问题 + 适用边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是 CDC | 按全量快照、增量日志、无锁切换、Schema 演进、下游一致性拆分 |
| 都是 MySQL -> 目标端 | 主问题是采集就归数据集成，主问题是目标端查询/索引才归 OLAP |
| 都是教程 | 没有失败模式、版本边界和验收指标时只精读或略读 |
| 有生产故障链路 | 优先沉淀为实践候选 |
| 只讲工具安装 | 不新建核心知识点 |

## 已覆盖问题（按三级节点）

| 节点 | 已覆盖 | 还缺 |
|---|---|---|
| Debezium | Debezium -> Kafka/Schema Registry -> Hudi Deltastreamer 低延迟入湖链路、初始快照与 bootstrap、source ordering field、删除语义 | 当前版本补证、Kafka Connect 运行边界、Schema Registry 兼容、入湖恢复实验 |
| SeaTunnel | 批式同步与 CDC 常驻任务边界、Source/Transform/Sink、MySQL-CDC 多表路由、SeaTunnel + Hive 在超大 Upsert 场景的边界 | 官方版本补证、SeaTunnel Engine 架构、Checkpoint 和 Sink 一致性实验 |

## 待补缺口

| 优先级 | 项 | 为什么补 |
|---|---|---|
| 高 | SeaTunnel 一致性与恢复实验 | 已建技术入口，但缺端到端恢复和 Sink 幂等验证 |
| 高 | CDC 工具横向边界 | Debezium、SeaTunnel CDC、Flink CDC、Canal 的职责和运行模型容易混淆 |
| 中 | DataX / Chunjun | 离线同步和批式集成常见工具 |
| 中 | Debezium 当前版本和运行边界 | 已补入湖链路，但官方版本、Kafka Connect 和 Schema Registry 边界未校准 |
