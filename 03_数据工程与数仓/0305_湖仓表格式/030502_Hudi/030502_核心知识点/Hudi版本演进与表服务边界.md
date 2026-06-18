# Hudi版本演进与表服务边界

> 验证版本：Hudi 1.0-1.2，具体能力以官方 Release 为准

## 来源
- [Hudi 1.0 新功能预览](<../文章/done-Hudi 1.0 新功能预览.md>)
- [Hudi 核心知识点详解（万字长文）](<../文章/done-Hudi 核心知识点详解（万字长文）.md>)

## 核心问题

Hudi 的版本演进主要围绕 Timeline、索引、并发控制、表服务、Flink/Spark 集成和湖仓生态扩展。版本文章只能用于提示能力变化，生产判断必须回到表服务和兼容性。

## 判断准则

| 判断项 | 准则 |
|---|---|
| 版本功能 | 只把影响写入、查询、索引、并发、表服务和兼容性的变化写入长期知识 |
| 表服务 | Compaction、Clustering、Clean、Index、Meta Sync 是生产治理核心，不是附属功能 |
| 引擎兼容 | Spark/Flink/Trino/Hive 的版本差异会影响同一 Hudi 表的读写语义 |
| 升级 | 升级前要验证 Timeline、表版本、写入客户端、元数据表和回滚路径 |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| Hudi 只是支持 upsert 的数据湖 | Hudi 的生产价值在 Timeline、索引、表服务和增量消费的整体闭环 |
| 新版本功能越多越该升级 | 湖表升级要看引擎兼容、历史表读写和表服务行为变化 |

## 待验证缺口

- Hudi 1.x 与当前 Spark/Flink/Trino 的兼容矩阵。
- COW/MOR 在同一 CDC 链路下的 compaction、clean 和增量查询成本。
