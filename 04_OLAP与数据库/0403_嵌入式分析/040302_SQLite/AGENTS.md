# SQLite
## 知识点入口

- 本模块先看宏观流程，再看文章：[知识地图](040302_知识地图.md)。
- 新文章必须先归入流程节点，再判断是补充、冲突、不同层次还是降权。
- `文章/` 只保留原文锚点，知识地图维护在 `040302_知识地图.md`；长期知识点写入 `040302_核心知识点/` 目录。

## 技术定位

| 项 | 内容 |
|---|---|
| 技术名 | SQLite |
| 一级类目 | OLAP 与数据库 |
| 二级类目 | 嵌入式分析 |
| 技术本体 | 面向本地、单文件、进程内嵌入的关系数据库，重点解决轻量应用状态、事务型本地存储和低运维数据库需求 |
| 全局架构位置 | 位于应用进程和本地文件系统之间，作为应用内数据库库文件被直接链接和调用 |
| 主要使用者 | 后端工程师、桌面应用工程师、移动端工程师、本地优先应用工程师、Agent 工具工程师 |
| 主要产出 | `.db` 文件、表、索引、事务、WAL 文件、本地应用状态 |

## 官方锚点

- 官网：[SQLite](https://www.sqlite.org/)
- 官方文档：[SQLite Documentation](https://www.sqlite.org/docs.html)
- 源码入口：[SQLite Source Repository](https://www.sqlite.org/src)
- WAL 文档：[Write-Ahead Logging](https://www.sqlite.org/wal.html)
- STRICT 表文档：[STRICT Tables](https://www.sqlite.org/stricttables.html)

## 架构图

```mermaid
flowchart LR
  App["应用进程\nCLI / 桌面 / 移动 / 后端"] --> API["SQLite C API / 语言绑定"]
  API --> SQL["SQL 解析 / 优化 / VDBE"]
  SQL --> Store["B-tree / Pager / Locking"]
  Store --> WAL["WAL / rollback journal"]
  Store --> File["单个 .db 文件"]
  File --> Backup["复制 / 备份 / 迁移"]
```

## 核心模块

| 模块 | 职责 | 重点问题 |
|---|---|---|
| SQL 层与 VDBE | 解析 SQL，生成并执行虚拟机指令 | SQL 方言、执行计划、函数和扩展 |
| B-tree / Pager | 管理页、表、索引和磁盘读写 | 页大小、B-tree 组织、文件增长 |
| 事务与锁 | 保证 ACID 和并发控制 | 单写者、读写锁、busy timeout、跨进程访问 |
| WAL / journal | 支持崩溃恢复和读写并发改善 | WAL checkpoint、网络文件系统边界、恢复路径 |
| 类型系统 | 通过类型亲和性和 STRICT 表管理约束 | 动态类型、时间类型、ANY、数据迁移 |
| 扩展能力 | FTS、JSON、RTree、load_extension 等 | 安全、可移植性、跨语言绑定 |

## 上下游

| 方向 | 对象 | 关系 |
|---|---|---|
| 上游 | 应用代码、ORM、CLI、脚本、移动端/桌面端 | 直接嵌入 SQLite 库并读写本地数据库文件 |
| 下游 | 本地文件、备份文件、导出 SQL、应用状态 | 产出可复制、可备份、可迁移的单文件数据库 |
| 依赖 | 本地文件系统、进程锁、语言绑定 | 文件系统和锁语义会直接影响可靠性和并发边界 |

## 横向对标

| 对标技术 | 对标点 | SQLite 优势 | SQLite 劣势 | 使用判断 |
|---|---|---|---|---|
| DuckDB | 嵌入式数据库 | SQLite 更适合事务型本地应用、点查更新和应用状态 | 不适合大文件列式扫描和分析型聚合 | 本地应用状态选 SQLite，本地分析和 Parquet/CSV 扫描选 DuckDB |
| PostgreSQL | 关系数据库 | SQLite 单文件、低运维、无网络延迟 | 多用户、高写并发、权限治理和 HA 能力弱 | 单应用服务器和本地优先可评估 SQLite；多服务共享写入选 PostgreSQL |
| MySQL | 关系数据库 | 部署和备份简单，适合小系统和嵌入式场景 | 服务化、复制、权限和生态治理能力弱 | 内部工具和 MVP 可评估 SQLite；标准服务端业务库选 MySQL/PostgreSQL |
| RocksDB / LevelDB | 嵌入式存储引擎 | SQLite 提供 SQL、事务和关系模型 | KV 读写路径和高吞吐写入不如专用 KV 引擎 | 需要 SQL 和关系约束选 SQLite；底层 KV 存储选 RocksDB/LevelDB |
| Redis / 消息队列 | 轻量异步能力 | SQLite 可通过表或扩展减少额外基础设施 | 不替代高吞吐队列、缓存和分布式协调 | 业务本来在 SQLite 且低吞吐可评估扩展；独立队列仍选专门中间件 |

## 当前文章

| 文章 | 阅读投入建议 | 处理建议 |
|---|---|---|
| [SQLite 又被重新看见：为什么 2026 年大家开始怀念一个单文件数据库？](<文章/done-SQLite 又被重新看见：为什么 2026 年大家开始怀念一个单文件数据库？.md>) | 精读候选 | 标题中的年份和“怀念”要降权，重点吸收单文件、低运维、读多写少、单写者和 PostgreSQL 边界 |
| [果不其然，SQLite的研究引来一堆人的关注，上一篇爆了](文章/done-果不其然，SQLite的研究引来一堆人的关注，上一篇爆了.md) | 精读候选 | 降权群宣和流量表达，重点吸收动态类型、STRICT 表、时间类型表达、库锁和传统关系库思维不适配 |

## 已沉淀核心知识点

| 主题 | 文件 | 问题指纹 | 解决什么问题 | 认知增量 |
|---|---|---|---|---|
| 单文件数据库边界 | [SQLite单文件数据库边界](040302_核心知识点/SQLite单文件数据库边界.md) | SQLite + 单文件数据库边界 + 机制/边界/验证 | SQLite 的重新被关注，本质是本地优先、低运维、单文件、读多写少和小团队部署成本的再平衡 | 形成可复用判断，不保留文章池 |
| 消息队列扩展边界 | [SQLite消息队列扩展边界](040302_核心知识点/SQLite消息队列扩展边界.md) | SQLite + 消息队列扩展边界 + 机制/边界/验证 | SQLite 可以通过表结构、触发器或扩展承载轻量队列，但这只适合低吞吐、本地可靠性要求可控、业务已经依赖 SQLite 的场景 | 形成可复用判断，不保留文章池 |
## 后续追查

- 关键词：WAL、checkpoint、锁、单写者、类型亲和性、STRICT 表、B-tree、Pager、VACUUM、backup API、FTS5、JSON1。
- 待读资料：SQLite 官方 WAL、锁、事务、STRICT 表、query planner、backup 文档。
- 待补实验：同一应用进程下读写并发、跨进程写入锁、WAL checkpoint、STRICT 表插入失败、DuckDB/PostgreSQL 对照。
