> 已吸收至：[[03_数据工程与数仓/0305_湖仓表格式/030505_Paimon/030505_核心知识点/Paimon LSM、Bloom 与生命周期治理|Paimon LSM、Bloom 与生命周期治理]]、[[03_数据工程与数仓/0305_湖仓表格式/030505_Paimon/030505_核心知识点/Paimon版本演进与特性边界|Paimon版本演进与特性边界]]
---
title: Apache Paimon 1.3 核心亮点总结
author: 大数据技能圈
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247492033&idx=1&sn=a26b7c90c6e6f99195e30f6f94324489&chksm=c197c58c2517efdac764599bb9180a7a73a495cfa7c4c91717bf43761ee518a1ccc568687fc3&mpshare=1&scene=24&srcid=1127NYmpTLHHHpPxo1Cybwi4&sharer_shareinfo=6d4ebdd550dec0388370fcf67bad0686&sharer_shareinfo_first=6d4ebdd550dec0388370fcf67bad0686#rd
---

### Apache Paimon 1.3 核心亮点总结

Apache Paimon 1.3 版本经过三个多月的开发，汇集了 500 多项代码提交，带来了一系列面向现代数据湖和 AI 应用场景的关键能力提升，主要体现在以下五个方面：

#### 1. **全新 PyPaimon：纯 Python SDK，摆脱 JVM 依赖**

* • 重构了原有的 Python SDK，不再依赖 Py4j 和 JVM，实现了完全原生的 Python 实现。
* • 借助 Arrow 的高效读写能力，在部分场景下性能甚至超越 Java SDK。
* • 当前支持 Append 表的完整读写，主键表仅支持基础去重操作，后续将扩展更多功能，并计划对接 Ray、Daft 等 AI/数据处理引擎。

#### 2. **Row Tracking + Data Evolution：轻量级列更新机制**

* • 启用 Row Tracking 后，每行数据自动获得全局唯一的 `_ROW_ID` 和版本号 `_SEQUENCE_NUMBER`，为后续高级功能奠定基础。
* • Data Evolution 允许在 Append 表中**仅更新特定列**，无需重写整行数据。例如 MERGE INTO 操作可只写入变更字段，大幅降低 I/O 与存储开销。

+ • 实测显示：MERGE 耗时从 27 分钟降至 17 分钟，存储占用从 170 GB 骤减至 1 GB。

#### 3. **Incremental Clustering：Append 表的智能数据布局优化**

* • 引入增量聚类机制，在合并小文件的同时对数据按指定键排序，显著提升查询效率。
* • 支持动态调整聚类键，无需全量重排；采用分层 LSM 结构控制写放大。
* • 性能表现：

+ • 单键过滤查询提速超 **17 倍**；
+ • 双键过滤查询提速高达 **150 倍**；
+ • 聚类执行速度比全量排序快 **20 倍以上**。

#### 4. **Virtual File System（PVFS）：统一目录与权限管理**

* • 通过 `pvfs://catalog/db/table/` 路径，用户可直接访问 REST Catalog 管理下的底层文件（如 CSV、Parquet 等）。
* • 所有访问均复用 Paimon 的权限体系，避免额外维护文件系统权限，提升安全性和易用性。
* • 已支持 Spark SQL 等引擎无缝集成。

#### 5. **其他关键优化**

* • 查询性能增强：支持 Spark 的 **TopN 下推** 和 **Limit 下推**，引入高性能 **Range Bitmap**。
* • Manifest 缓存按分区和 Bucket 组织，加速 OLAP 查询。
* • 修复了 `MERGE INTO` 与 `COMPACT` 并发执行时可能导致的数据一致性问题，尤其在 Deletion Vectors 模式下。

---

### 面向未来的方向

Paimon 正积极拓展**多模态数据湖**能力，包括：

* • 支持文本、图像、音视频等非结构化数据及其标签、向量的统一存储；
* • 开发 Blob 存储与全局索引（标量/向量/B树/Bitmap）；
* • 深度集成 AI 生态，强化 Python SDK 与分布式训练/推理框架的协同。

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

本学习文档已放入星球，扫描二维码加入星球获取