# Redisson

## 技术定位

| 项 | 内容 |
|---|---|
| 技术名 | Redisson |
| 一级类目 | OLAP 与数据库 |
| 二级类目 | 缓存与 KV |
| 技术本体 | Java Redis 客户端与分布式对象框架，封装锁、队列、集合、信号量等分布式能力 |
| 全局架构位置 | Java 应用访问 Redis 的客户端抽象层 |
| 主要使用者 | Java 后端工程师 |
| 主要产出 | RLock、分布式对象、Lua 脚本执行、Pub/Sub 通知、Watchdog 自动续期 |

## 当前文章

| 文章 | 阅读投入建议 | 处理建议 |
|---|---|---|
| [分布式锁工具Redisson，太香了！！](文章/分布式锁工具Redisson，太香了！！.md) | 精读候选 | 标题降权，只沉淀 RLock、可重入、Watchdog、公平锁和 Lua 脚本边界 |

## 排重准则

| 判断项 | 处理 |
|---|---|
| 只讲 Redis 分布式锁命令 | 可归 Redis，不一定进 Redisson |
| 讲 RLock、Watchdog、Pub/Sub 唤醒、公平锁队列 | 归 Redisson |
| 只给使用示例不讲失败场景 | 略读或只保留锚点 |

## 后续追查

- Watchdog 续期失败模式。
- 客户端宕机、网络分区、Redis 主从切换下的锁语义。
- Redlock 与单 Redis 锁的边界。
