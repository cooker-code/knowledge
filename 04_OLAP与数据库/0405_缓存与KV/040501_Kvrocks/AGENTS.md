# Kvrocks

## 技术定位

| 项 | 内容 |
|---|---|
| 技术名 | Kvrocks |
| 一级类目 | OLAP 与数据库 |
| 二级类目 | 缓存与 KV |
| 技术本体 | Redis 协议兼容的持久化 KV 数据库，底层基于 RocksDB |
| 全局架构位置 | Redis-compatible 持久化 KV 层，适合需要 Redis 生态接口但数据规模或持久化要求更高的场景 |
| 主要使用者 | 后端工程师、平台工程师、缓存平台维护者 |
| 主要产出 | Redis 协议兼容 KV 服务、持久化数据文件、复制和命令兼容能力 |

## 已沉淀核心知识点

| 主题 | 文件 | 问题指纹 | 解决什么问题 | 认知增量 |
|---|---|---|---|---|
| Redis 兼容持久化 KV 边界 | [KvrocksRedis兼容持久化KV边界](040501_核心知识点/KvrocksRedis兼容持久化KV边界.md) | Kvrocks + Redis 兼容持久化 KV 边界 + 机制/边界/验证 | Kvrocks 的选型点不是“比 Redis 新”，而是 Redis 协议兼容与 RocksDB 持久化容量之间的取舍 | 形成可复用判断，不保留文章池 |

## 当前文章

| 文章 | 阅读投入建议 | 处理建议 |
|---|---|---|
| [Apache Kvrocks 2.14.0 版本发布](<文章/done-Apache Kvrocks 2.14.0 版本发布.md>) | 略读 | 作为版本动态锚点，精修时需补官方 release 和 RocksDB 架构边界 |

## 横向对标

| 对标对象 | 对标点 | 使用判断 |
|---|---|---|
| Redis | 协议和数据结构生态 | 低延迟纯内存优先 Redis；持久化容量和成本压力大时看 Kvrocks |
| RocksDB | 底层存储 | Kvrocks 是服务化 KV；RocksDB 是嵌入式存储引擎 |
| TiKV | 分布式 KV | 强分布式事务和水平扩展看 TiKV；Redis 协议兼容看 Kvrocks |

## 后续追查

- RocksDB 配置、压缩、Compaction、复制机制、命令兼容矩阵。
