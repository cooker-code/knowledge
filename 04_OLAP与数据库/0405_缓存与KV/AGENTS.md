# 缓存与 KV

## 知识点入口

- 本模块先看宏观流程，再看文章：[流程化知识点总览](knowledge/04_OLAP与数据库/0405_缓存与KV/核心知识点/流程化知识点总览.md)。
- 新文章必须先判断主问题是缓存/KV 技术本体，还是后端应用层缓存策略。
- `文章/` 只作为临时入口；正式归档必须落到 `Redis/`、`Kvrocks/`、`Redisson/` 或后续新增的具体技术目录。

## 类目定位

| 项 | 内容 |
|---|---|
| 一级类目 | OLAP 与数据库 |
| 二级类目 | 缓存与 KV |
| 核心问题 | 数据如何以键值、内存缓存、持久化 KV 或分布式对象形式提供低延迟访问能力 |
| 不解决什么 | 普通后端缓存分层、HTTP 缓存、Guava/Caffeine 应用代码封装，除非文章主问题是 Redis/KV 本体 |
| 用户当前认知假设 | 用户知道 Redis 常用作缓存，需要补数据结构、持久化、Pipeline、Lua、双活、可观测性、Redis-compatible KV 和分布式锁边界 |

## 划分准则

| 保留条件 | 归入位置 | 示例判断 |
|---|---|---|
| 讲 Redis 服务端机制、协议、数据结构、持久化、Pipeline、Lua、双活、延迟排查 | `Redis/` | Redis Pipeline、RDB/AOF、Lua、双活、慢请求 |
| 讲 Redis 兼容持久化 KV 或 Redis 协议替代实现 | `Kvrocks/` | Kvrocks 版本、新特性、持久化 KV 边界 |
| 讲 Redisson 分布式对象、分布式锁、看门狗、公平锁等 Redis 客户端抽象 | `Redisson/` | Redisson 分布式锁源码和 Lua 脚本 |
| 讲后端三层缓存、HTTP 缓存、Caffeine/Guava 与业务代码组合 | 移到 `07_工程与架构/0701_后端架构/` | 主问题是后端架构和应用缓存策略 |

## 技术子目录

| 技术 | 目录 | 文章数 | 定位 |
|---|---|---:|---|
| Redis | [Redis/](Redis/) | 9 | 内存 KV、缓存、数据结构、持久化和高可用 |
| Kvrocks | [Kvrocks/](Kvrocks/) | 1 | Redis 兼容、基于 RocksDB 的持久化 KV |
| Redisson | [Redisson/](Redisson/) | 1 | Java Redis 客户端与分布式对象/锁抽象 |

## 排重准则

问题指纹：

```text
缓存或 KV 技术本体 + 数据结构/协议/持久化/高可用/客户端抽象模块 + 核心机制 + 解决问题 + 适用边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都讲 Redis 快 | 按网络往返、事件循环、数据结构、内存模型、持久化、Pipeline、Lua、缓存命中拆分 |
| 都讲缓存策略 | 如果主问题是后端缓存层次，移到后端架构；如果主问题是 Redis 机制，留在 Redis |
| 都讲分布式锁 | Redis 命令级实现归 Redis；Redisson 看门狗、公平锁、客户端封装归 Redisson |
| Redis 与 RocksDB 对比 | 如果目的是理解 Redis 边界，放 Redis；如果主问题是 LSM/Compaction，放存储引擎 |

## 后续追查

- Redis：RESP、事件循环、RDB/AOF、Pipeline、Lua 原子性、Stream、Bitmap、HLL、Geo、Cluster、复制、Sentinel。
- Kvrocks：RocksDB 底层、冷热数据、复制、Redis 协议兼容限制。
- Redisson：RLock、Watchdog、Pub/Sub 唤醒、公平锁队列、Lua 脚本一致性边界。
