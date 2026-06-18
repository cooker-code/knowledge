# 缓存与 KV

> 分类规则、一二级类目定位、跨域归类边界以根 [目录划分.md](../../目录划分.md) 为准。
> 文章理解的四步法、整理流程以根 [AGENTS.md](../../AGENTS.md) 为准。
> 本文件只维护本领域的认知重点、排重指纹、已覆盖三级节点和待补缺口。

## 三级节点入口

| 节点 | 用途 |
|---|---|
| [040501_Kvrocks](040501_Kvrocks/AGENTS.md) | Redis 兼容、基于 RocksDB 的持久化 KV |
| [040502_Redis](040502_Redis/AGENTS.md) | 内存 KV、缓存、数据结构、持久化和高可用 |
| [040503_Redisson](040503_Redisson/AGENTS.md) | Java Redis 客户端与分布式对象/锁抽象 |

## 用户认知重点

| 项 | 内容 |
|---|---|
| 已知基础 | 用户知道 Redis 常用作缓存与简单数据结构 |
| 待补边界 | 数据结构、持久化、Pipeline、Lua、双活、可观测性、Redis 兼容 KV 与分布式锁边界 |
| 易偏差点 | 把后端三层缓存、HTTP 缓存、Caffeine/Guava 等业务代码当 Redis 本体 |
| 优先抽取 | RESP、事件循环、RDB/AOF、Pipeline、Lua、Cluster、复制、Sentinel |

## 排重指纹

```text
缓存或 KV 技术本体 + 数据结构/协议/持久化/高可用/客户端抽象模块 + 核心机制 + 解决问题 + 适用边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都讲 Redis 快 | 按网络往返、事件循环、数据结构、内存模型、持久化、Pipeline、Lua、缓存命中拆分 |
| 都讲缓存策略 | 主问题是后端缓存层次时移到后端架构；Redis 机制留本类 |
| 都讲分布式锁 | Redis 命令级实现归 Redis；Redisson 看门狗、公平锁、客户端封装归 Redisson |
| Redis 与 RocksDB 对比 | 理解 Redis 边界放 Redis；主问题是 LSM/Compaction 放存储引擎 |

## 已覆盖问题（按三级节点）

| 节点 | 已覆盖 | 还缺 |
|---|---|---|
| Redis | 内存 KV/缓存/数据结构/持久化/高可用初始承接 | RESP、事件循环、RDB/AOF、Pipeline、Lua、Stream、Bitmap、HLL、Geo、Cluster、复制、Sentinel 完整沉淀 |
| Kvrocks | Redis 兼容持久化 KV 候选承接 | RocksDB 底层、冷热数据、复制、Redis 协议兼容限制 |
| Redisson | 分布式对象/锁抽象候选承接 | RLock、Watchdog、Pub/Sub 唤醒、公平锁队列、Lua 脚本一致性边界 |

## 待补缺口

| 优先级 | 项 | 为什么补 |
|---|---|---|
| 高 | Redis 持久化与高可用 | RDB/AOF/复制/Sentinel/Cluster 是生产部署的核心 |
| 高 | Redisson 分布式锁机制 | 看门狗、公平锁、Lua 一致性边界，业务高频踩坑点 |
| 中 | Kvrocks 与 Redis 协议兼容边界 | 决定能否替换 Redis 做持久化 KV |
