# LSM-Tree

## 技术定位

| 项 | 内容 |
|---|---|
| 技术名 | LSM-Tree |
| 一级类目 | OLAP 与数据库 |
| 二级类目 | 存储引擎 |
| 技术本体 | 写优化的存储结构，通过内存写入、顺序落盘和后台 Compaction 平衡写吞吐、读放大和空间放大 |
| 全局架构位置 | RocksDB、LevelDB、HBase、Cassandra、部分 HTAP/KV 系统的底层存储思想 |
| 主要使用者 | 数据库内核工程师、存储系统工程师、平台工程师 |
| 主要产出 | WAL、MemTable、SSTable、分层文件、Compaction 结果 |

## 已沉淀核心知识点

| 主题 | 文件 | 问题指纹 | 解决什么问题 | 认知增量 |
|---|---|---|---|---|
| WAL、MemTable、SSTable 与 Compaction 边界 | [LSMTreeWALMemTableSSTable与Compaction边界](040602_核心知识点/LSMTreeWALMemTableSSTable与Compaction边界.md) | LSM-Tree + WAL、MemTable、SSTable 与 Compaction 边界 + 机制/边界/验证 | LSM-Tree 用写入先进入 WAL/MemTable、顺序落盘 SSTable、后台 Compaction 的方式换取高写吞吐 | 形成可复用判断，不保留文章池 |

## 当前文章

| 文章 | 阅读投入建议 | 处理建议 |
|---|---|---|
| [LSM数据结构在大数据领域的应用](文章/done-LSM数据结构在大数据领域的应用.md) | 精读候选 | 用于建立 LSM 在 HBase/Cassandra/RocksDB 中的横向位置 |
| [数据库筑基课-高能长文说说LSM-Tree](文章/done-数据库筑基课-高能长文说说LSM-Tree.md) | 精读候选 | 只保留架构、Compaction、适用边界，降权泛泛扩展 |
| [跟着论文学习数据库4：LSM-tree 数据结构](<文章/done-跟着论文学习数据库4：LSM-tree 数据结构.md>) | 精读候选 | 需要补原图或重建流程图 |

## 核心判断

| 判断项 | 准则 |
|---|---|
| 为什么写快 | 写入先进入内存和 WAL，落盘以顺序写为主 |
| 为什么读可能慢 | 读请求可能跨 MemTable、多层 SSTable 和 Bloom Filter |
| Compaction 的代价 | 用后台合并偿还写入债务，带来写放大、读放大和空间放大权衡 |
| 适合场景 | 写多读少、批量写、可接受后台整理的 KV/宽表/日志类系统 |

## 后续追查

- size-tiered、leveled、universal compaction 的差异。
- Bloom Filter、Block Index、Manifest 与崩溃恢复。
