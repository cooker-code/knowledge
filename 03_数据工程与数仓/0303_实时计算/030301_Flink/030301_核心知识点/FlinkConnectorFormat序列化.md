# FlinkConnectorFormat序列化

## 来源
- [Flink SQL格式集成：JSON、Avro、Protobuf序列化详解](../文章/done-Flink SQL格式集成：JSON、Avro、Protobuf序列化详解.md)

## 核心问题
如何为 Flink SQL Connector 选择合适的序列化格式？JSON/Avro/Protobuf 在配置、性能、Schema 演进上的核心差异是什么？

## 判断准则

### 格式选择决策矩阵
| 维度 | JSON | Avro | Protobuf |
|---|---|---|---|
| 编码 | 文本（人类可读） | 二进制 | 二进制 |
| 性能 | 较低（解析开销大） | 高 | 最高 |
| Schema 约束 | 弱（可选 Schema 验证） | 强（必须预定义） | 强（必须预定义 .proto）|
| Schema 演进 | 手动路由多版本 | 原生支持（Schema Registry）| 向后兼容（字段编号稳定）|
| 跨语言 | 通用 | 需 Avro 库 | 官方多语言支持 |
| 调试友好度 | 高 | 低 | 低 |
| 适合场景 | 快速开发、数据量中等 | 企业数据管道、需要演进 | 高性能微服务、高吞吐场景 |

### JSON 核心配置参数
```sql
'format' = 'json'
'json.fail-on-missing-field' = 'false'    -- 缺失字段不报错（容错）
'json.ignore-parse-errors' = 'true'       -- 解析错误继续处理（容错）
'json.timestamp-format.standard' = 'ISO-8601'  -- 时间戳格式
'json.map-null-key.mode' = 'DROP'         -- Map 空 Key 处理：DROP/LITERAL/FAIL
```
嵌套字段访问：用反引号包裹点号路径，如 `` `user.address.city` ``

### Avro 核心配置参数
```sql
'format' = 'avro'
'avro.codec' = 'snappy'                          -- 压缩（高吞吐场景）
'avro.schema.registry.url' = 'http://sr:8081'   -- Schema Registry 地址
```
- Flink 字段名和类型必须与 Avro Schema 严格匹配
- 联合类型（union）在 Flink 中映射为 STRING 字段，需业务层解析

### Protobuf 核心配置参数
```sql
'format' = 'protobuf'
'protobuf.message-class-name' = 'com.example.UserEvent'  -- 必须指定消息类名
```
- repeated 字段 → `ARRAY<T>`
- map 字段 → `MAP<K, V>`
- enum 字段 → `STRING`（存枚举名）
- int64 时间戳字段需手动转换为 `TIMESTAMP(3)`

### Schema 多版本处理策略
当同一 Topic 存在多个 Schema 版本时，推荐用路由方式：
```sql
-- 将 raw_data (STRING) + schema_version (INT) 存入一张宽表
-- 按 schema_version 过滤，分别解析不同版本字段
-- 用 COALESCE 做字段对齐合并
```

### 格式转换注意事项
- JSON → Avro：类型需精确匹配（Avro 无 BIGINT，用 long）
- Avro → Protobuf：注意 Avro int 对应 Protobuf int32，需显式 CAST

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| JSON 格式解析失败会导致作业崩溃 | 配置 `json.ignore-parse-errors=true` 可跳过解析错误继续处理 |
| Avro 比 JSON 快很多，所有场景都应用 Avro | Avro 的 Schema 管理和部署成本更高；数据量不大时 JSON 的开发效率优势更显著 |
| Protobuf 字段可以随意改名 | 字段名可改，但字段编号（field number）不能变，否则反序列化失败 |
| 嵌套 JSON 字段无法用 Flink SQL 直接解析 | 支持用点号路径语法（需反引号包裹）或 JSON_VALUE/JSON_QUERY 函数动态提取 |

## 待验证缺口
- Avro Schema Registry 与 Flink Catalog 的集成是否支持自动 Schema 演进（字段增减无需重启作业）
- Protobuf 在 Flink SQL 中的 oneof 字段如何映射
- 高吞吐场景下三种格式的实际 TPS 对比（文章中未提供实测数据，仅为理论分析）
