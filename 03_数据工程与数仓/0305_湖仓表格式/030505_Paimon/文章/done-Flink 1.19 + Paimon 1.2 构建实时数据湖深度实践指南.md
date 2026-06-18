---
title: Flink 1.19 + Paimon 1.2 构建实时数据湖深度实践指南
author: 大数据技能圈
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491650&idx=1&sn=642b6e8878454ce7e369ec9d074accbb&chksm=c172550454439fe1dceec1efe305bcc1ae30704545faffdb8902067875784426ba0093623c72&mpshare=1&scene=24&srcid=080445vTQhIS1VNu6PlZijv4&sharer_shareinfo=433c70a3c3dff1505824b4ef171390e1&sharer_shareinfo_first=433c70a3c3dff1505824b4ef171390e1#rd
---

> 已吸收至：[[03_数据工程与数仓/0305_湖仓表格式/030505_Paimon/030505_核心知识点/Flink SQL 与 Paimon 流式湖仓实践|Flink SQL 与 Paimon 流式湖仓实践]]


01

**加入知识星球《随川陪你学大数据》将获得以下权益**

扫描二维码加入星球，随加入人数增加，逐步涨价，现在就是最优惠的

01

**Flink+Paimon构建实时数据湖实战**

**摘要：** 本文深入探讨如何利用 Apache Flink 1.9 与 Apache Paimon 1.2 构建高性能、低延迟的实时数据湖。文章从核心概念、架构设计、环境搭建、数据写入、实时查询、性能优化到运维监控，提供详尽的中文说明和可运行的样例代码，帮助读者掌握这一前沿技术栈的落地实践。

---

### 目录

1. **引言：实时数据湖的演进与挑战**

* 1.1 传统数据仓库与数据湖的局限
* 1.2 实时数据湖的核心诉求
* 1.3 Flink + Paimon：实时数据湖的理想组合
* 1.4 版本选择说明 (Flink 1.9 & Paimon 1.2)

2. **核心概念与技术栈解析**

* 2.2.1 Paimon 核心设计理念
* 2.2.2 Paimon 1.2 关键特性
* 2.2.3 Paimon vs. Iceberg/Hudi/Delta Lake

* 2.1.1 Flink 1.9 核心特性回顾
* 2.1.2 Table API & SQL (DataStream API 的补充)

* 2.1 Apache Flink：流批一体的计算引擎
* 2.2 Apache Paimon：新一代流式数据湖存储
* 2.3 Flink 与 Paimon 的协同工作机制

3. **实时数据湖架构设计**

* 3.2.1 数据摄入层 (Kafka/Pulsar)
* 3.2.2 实时计算层 (Flink)
* 3.2.3 数据湖存储层 (Paimon on HDFS/OSS/S3)
* 3.2.4 查询分析层 (Presto/Trino/Spark/Flink)

* 3.1 整体架构图
* 3.2 数据流向详解
* 3.3 关键组件选型与配置考量

4. **环境搭建与配置**

* 4.3.1 下载 Paimon JAR 包
* 4.3.2 将 Paimon JAR 放入 Flink Lib 目录
* 4.3.3 配置 Catalog (使用 Flink SQL)

* 4.2.1 下载与解压
* 4.2.2 配置文件 (`flink-conf.yaml`, `workers`, `masters`)
* 4.2.3 启动 Flink 集群 (Standalone/YARN)

* 4.1.1 JDK (推荐 OpenJDK 8/11)
* 4.1.2 Hadoop (HDFS/YARN, 2.7+/3.x)
* 4.1.3 分布式文件系统 (HDFS, OSS, S3)

* 4.1 基础环境准备
* 4.2 Apache Flink 1.9 部署
* 4.3 Apache Paimon 1.2 集成
* 4.4 依赖管理 (Maven 示例)

5. **数据写入：构建实时数据湖核心**

* 5.4.1 并行度设置
* 5.4.2 检查点 (Checkpoint) 配置
* 5.4.3 内存管理 (TaskManager 内存)
* 5.4.4 Paimon 写入参数 (`write-buffer-size`, `write-buffer-spillable`等)

* 5.3.1 引入 Paimon Sink
* 5.3.2 定义数据流与转换逻辑
* 5.3.3 配置 Sink 参数 (并行度、检查点等)
* 5.3.4 样例代码：自定义处理逻辑写入 Paimon

* 5.2.1 创建 Kafka Source 表
* 5.2.2 数据转换与清洗 (ETL)
* 5.2.3 写入 Paimon 表 (`INSERT INTO`)
* 5.2.4 完整 Flink SQL 作业提交示例

* 5.1.1 基本建表语法
* 5.1.2 核心表属性详解 (分区、主键、合并引擎、文件格式等)
* 5.1.3 样例：创建用户行为事件表

* 5.1 创建 Paimon 表 (Flink SQL DDL)
* 5.2 从 Kafka 实时摄入数据 (Flink SQL)
* 5.3 使用 DataStream API 写入 Paimon (高级场景)
* 5.4 写入性能优化与调优

6. **数据查询：实时与批处理统一**

* 6.4.1 分区裁剪 (Partition Pruning)
* 6.4.2 文件过滤 (File Filtering)
* 6.4.3 索引利用 (Paimon 的 Bucket & Index)
* 6.4.4 列裁剪 (Column Pruning)

* 6.3.1 Spark 集成 Paimon
* 6.3.2 查询示例

* 6.2.1 Presto/Trino 集成 Paimon Connector
* 6.2.2 创建 Catalog 与查询示例
* 6.2.3 性能考量

* 6.1.1 实时流式查询 (Streaming Mode)
* 6.1.2 批处理查询 (Batch Mode)
* 6.1.3 时间旅行查询 (Time Travel)

* 6.1 使用 Flink SQL 查询 Paimon 表
* 6.2 使用 Presto/Trino 查询 Paimon 数据湖
* 6.3 使用 Spark SQL 查询 Paimon 表 (可选)
* 6.4 查询优化策略

7. **核心特性深度实践**

* 7.4.1 快照机制原理
* 7.4.2 查询历史快照 (`FOR SYSTEM_TIME AS OF`)
* 7.4.3 快照过期与清理 (`snapshot.expire.limit`)

* 7.3.1 添加列 (ADD COLUMN)
* 7.3.2 修改列类型 (ALTER COLUMN ... TYPE) (注意兼容性)
* 7.3.3 删除列 (DROP COLUMN) (注意兼容性)
* 7.3.4 Flink 1.9 下的限制与建议

* 7.2.1 时间分区 (`PARTITIONED BY (dt, hh)`)
* 7.2.2 动态分区 (Flink SQL 支持情况)
* 7.2.3 分区生命周期管理 (TTL)

* 7.1.1 主键表的工作原理
* 7.1.2 合并引擎 (Merge Engine) 详解 (`deduplicate`, `partial-update`, `aggregation`)
* 7.1.3 样例：实现用户画像的 Partial Update

* 7.1 主键表与 Upsert 语义
* 7.2 分区表管理
* 7.3 模式演化 (Schema Evolution)
* 7.4 快照 (Snapshot) 与时间旅行

8. **性能优化与调优**

* 8.3.1 块大小 (Block Size) 调整
* 8.3.2 本地缓存 (HDFS Short-Circuit Local Reads)
* 8.3.3 对象存储优化 (连接池、重试策略)

* 8.2.1 文件格式选择 (Parquet/ORC/Avro) 与压缩
* 8.2.2 Bucket 数量设置 (`'bucket' = '-1'` 自动 vs. 手动)
* 8.2.3 文件大小控制 (`file.target-size`, `file.compaction.threshold`)
* 8.2.4 合并 (Compaction) 策略调优 (`compaction.max-file-num`, `compaction.max-size-amplification-percent`)

* 8.1.1 状态后端 (State Backend) 选择 (FsStateBackend/RocksDBStateBackend)
* 8.1.2 检查点优化 (间隔、超时、对齐)
* 8.1.3 反压监控与处理
* 8.1.4 网络缓冲区调优

* 8.1 Flink 作业优化
* 8.2 Paimon 表优化
* 8.3 存储层优化 (HDFS/OSS/S3)

9. **运维与监控**

* 9.3.1 关键指标告警 (Checkpoint 失败、反压、延迟)
* 9.3.2 作业重启策略配置
* 9.3.3 资源弹性伸缩 (YARN/K8s)

* 9.2.1 快照信息查询 (`SHOW SNAPSHOTS`)
* 9.2.2 文件信息查询 (`SHOW FILES`, `SHOW PARTITIONS`)
* 9.2.3 合并任务监控
* 9.2.4 使用 Paimon Tools 进行诊断

* 9.1.1 Flink Web UI
* 9.1.2 指标系统 (Metrics Reporter - Prometheus, InfluxDB)
* 9.1.3 日志分析 (Log4j)

* 9.1 Flink 作业监控
* 9.2 Paimon 表监控
* 9.3 告警与自动化运维

10. **高级场景与扩展**

* 10.2.1 维表 Join (Lookup Join)
* 10.2.2 双流 Join (Regular Join, Interval Join)

* 10.1.1 使用 Flink CDC Source (Debezium)
* 10.1.2 处理 ChangeLog (INSERT, UPDATE, DELETE)
* 10.1.3 写入 Paimon 主键表

* 10.1 CDC 数据入湖 (MySQL -> Kafka -> Flink -> Paimon)
* 10.2 流式 Join (Flink SQL)
* 10.3 物化视图 (Materialized View) (Paimon 1.2 实验性特性)

11. **挑战、限制与未来展望**

* 11.2.1 部分高级特性可能不兼容
* 11.2.2 性能优化空间

* 11.1.1 Table API/SQL 功能成熟度
* 11.1.2 State TTL 支持情况
* 11.1.3 新特性缺失 (如 Hybrid Source, Savepoint 升级)

* 11.1 Flink 1.9 的局限性
* 11.2 Paimon 1.2 在 Flink 1.9 下的限制
* 11.3 升级建议 (Flink 1.15+ & Paimon 最新版)
* 11.4 实时数据湖未来趋势

12. **总结**

以上4万字详细文档已放入知识星球《随川陪你学大数据》，可扫码获取

获取更多信息，关注大数据技能圈

## 推荐阅读系列文章

* [超强整理：Iceberg最新学习文档（十四章）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491583&idx=1&sn=50016bf38b99c5a1f33a441b841c1b96&scene=21#wechat_redirect)
* [超强总结：Spark最新学习文档（十二章）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491603&idx=1&sn=4770b27307ff0c6ff2b1d1763e1e8482&scene=21#wechat_redirect)
* [超强总结：Flink最新学习文档（十二章）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491637&idx=1&sn=589ae0f30eb9dd02ab959c3cb18e5b0f&scene=21#wechat_redirect)