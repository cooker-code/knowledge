# RocksDB

## 技术定位

| 项 | 内容 |
|---|---|
| 技术名 | RocksDB |
| 一级类目 | OLAP 与数据库 |
| 二级类目 | 存储引擎 |
| 技术本体 | 基于 LSM-Tree 的嵌入式持久化 KV 存储引擎 |
| 全局架构位置 | 上层数据库、KV 系统、流处理状态后端或缓存系统的本地存储层 |
| 主要使用者 | 存储系统工程师、数据库内核工程师、平台工程师 |
| 主要产出 | SST 文件、WAL、MemTable、Compaction、Block Cache |

## 当前文章

| 文章 | 阅读投入建议 | 处理建议 |
|---|---|---|
| [RocksDB：Compaction 是如何工作的](文章/RocksDB：Compaction%20是如何工作的.md) | 精读候选 | 可沉淀 Compaction、写放大、读放大、空间放大和写入抖动 |

## 排重准则

| 判断项 | 处理 |
|---|---|
| 只讲 LSM 抽象 | 放 LSM-Tree |
| 讲 RocksDB 参数、Block Cache、Column Family、Write Stall | 放 RocksDB |
| 只把 RocksDB 作为对比对象 | 按文章主问题归 Redis/Kvrocks/FlowDB，不强行放 RocksDB |

## 后续追查

- Write Stall、L0 文件堆积、Compaction 线程、Block Cache、Column Family 参数。
