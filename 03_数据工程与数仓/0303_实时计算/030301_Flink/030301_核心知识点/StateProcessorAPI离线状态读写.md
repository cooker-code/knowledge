# State Processor API 离线状态读写

> 验证版本：Flink 1.9+

## 来源
- [State Processor API：如何读写和修改 Flink 应用程序的状态](../文章/done-State%20Processor%20API：如何读写和修改%20Flink%20应用程序的状态.md)

## 核心问题
Flink 1.9 之前，Savepoint 内部是自定义二进制格式，无法从外部读取或修改，相当于黑匣子。State Processor API 将 Savepoint/Checkpoint 中的状态公开为可操作的数据集（DataSet），使得离线读取、修改、导入应用状态成为可能。

## 判断准则

### 核心能力（能做什么）

| 操作 | 说明 |
|---|---|
| 读取状态 | 将 Savepoint/Checkpoint 中的 Keyed State 或 Operator State 读取为 DataSet |
| 修改状态 | 修复状态数据错误、改变状态数据类型 |
| 写入状态 | 从外部系统（数据库、文件）导入数据作为初始状态 |
| 元数据变更 | 调整算子最大并行度、拆分/合并算子状态、重新分配算子 UID |

### 状态与数据集的映射关系

Savepoint 可类比为一个数据库：
- 每个算子（由 UID 标识）对应一个命名空间
- Operator State 映射为该命名空间下的表（一列，每行是一个并行实例的状态）
- Keyed State 映射为表（Key 一列 + 每个状态字段各一列）

```
MyApp Savepoint
├── Src (UID: "src-uid")
│   └── os1: Operator State → 表 [value列，5行（5个并行度）]
├── Proc (UID: "proc-uid")
│   ├── os2: Operator State → 表 [value列]
│   └── ks1, ks2: Keyed State → 表 [key列, ks1列, ks2列]
└── Snk (UID: "snk-uid") → 无状态，命名空间为空
```

### 适用场景

- **调试验证**：获取生产 Savepoint，用批处理验证状态数据是否正确
- **状态初始化**：从关系数据库读取历史数据，写入 Savepoint，新作业从有状态起点启动
- **迁移**：从 HashMapStateBackend 迁移到 RocksDB；修改状态类型
- **修复**：修复 Savepoint 中不一致的状态条目，再恢复作业

### 技术实现基础（1.9 版本）
- 基于 DataSet API（批处理）实现 Input/Output Formats 来读写 Savepoint
- DataSet API 与 Table API 可互转，因此也可以用 SQL 分析状态数据
- 注意：DataSet API 在后续版本中被废弃，State Processor API 的底层 API 可能在新版本中迁移

### 1.9 之前的替代方案的局限性

| 方案 | 局限 |
|---|---|
| Queryable State | 只支持点查（按 key），不保证一致性，不能写/修改 |
| 直接读 Savepoint 文件 | 自定义二进制格式，无公开 API |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| State Processor API 用于流处理中访问状态 | 它是离线批处理工具，读写的是静态的 Savepoint/Checkpoint 快照 |
| 可以在运行中作业上用 State Processor API 修改状态 | 只能操作静态 Savepoint，不能热修改运行中作业的状态 |
| State Processor API 基于 DataStream API | 1.9 版本基于 DataSet API（后续版本可能迁移） |
| Queryable State 能替代 State Processor API | Queryable State 只读不写，且一致性保证弱 |

## 待验证缺口
- State Processor API 在 Flink 1.14+ 后是否已迁移到 DataStream API（DataSet API 废弃后的接替方案）
- 通过 State Processor API 修改后的 Savepoint，是否能直接被新版本作业恢复（Savepoint 格式兼容性）
- Keyed State 和 Operator State 的实际读写 API 调用方式（文章层面未给出代码示例，需查官方文档）
