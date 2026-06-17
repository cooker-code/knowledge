# LevelDB

## 技术定位

| 项 | 内容 |
|---|---|
| 技术名 | LevelDB |
| 一级类目 | OLAP 与数据库 |
| 二级类目 | 存储引擎 |
| 技术本体 | Google 开源的嵌入式 KV 存储引擎，是理解最小 LSM 实现的典型样本 |
| 全局架构位置 | 本地嵌入式 KV 层，也常作为学习 LSM 文件组织和版本管理的入口 |
| 主要使用者 | 存储学习者、数据库内核工程师 |
| 主要产出 | WAL、MemTable、SSTable、Manifest、VersionEdit |

## 当前文章

| 文章 | 阅读投入建议 | 处理建议 |
|---|---|---|
| [LevelDB：一个最小 LSM 引擎如何组织数据](文章/LevelDB：一个最小%20LSM%20引擎如何组织数据.md) | 精读候选 | 适合沉淀 WAL、MemTable、SSTable、Manifest、Flush、恢复路径 |

## 后续追查

- Manifest 和 VersionSet。
- Restart recovery。
- 与 RocksDB 的差异。
