---
title: Flink CDC 3.3.0 发布公告
author: Flink CDC
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzk0OTY0MjM4OQ==&mid=2247483797&idx=1&sn=4a0c33e3ee4ea5a2a66e2a0cf7d4b104&chksm=c2cb4efe161f81d61ac7078a648573beacffc30ece7fb484a985505be403ab6d5b08dd6ca8c4&mpshare=1&scene=24&srcid=0122I2674yRTYL2WeSqoaQFa&sharer_shareinfo=37136912826bfa0aba9547b38b90dbbf&sharer_shareinfo_first=37136912826bfa0aba9547b38b90dbbf#rd
---

**Apache Flink 社区非常高兴地宣布 Flink CDC 的下一个主要版本 3.3.0 已经发布。**

现在，您可以访问 Flink CDC Release 页面[1] 下载 CDC 3.3.0 的二进制包，也可以在文档网站[2] 上访问最新版本的文档。如果您在使用时遇到任何问题，欢迎在 Flink 用户邮件列表[3]、用户钉群、GitHub Discussions[4] 或 Flink JIRA 看板[5] 上提出问题或发起讨论。

此次更新为数据集成 Pipeline 引入了 AI Model 支持，新增了 Oceanbase 与 MaxCompute Sink 连接器，并为 Transform 模块带来了若干改进。我们推荐您升级到 Flink CDC 3.3.0 版本。

> 此版本不再保证与 Flink 1.18 及更早版本的兼容性。

## 新功能速览

### Transform

* 支持在 Transform 表达式中调用 AI Model。目前内置 OpenAI Chat 模型及 Embedding 向量化模型。

* 新增了操作时间戳的 `TIMESTAMPADD`、`TIMESTAMPDIFF`、`UNIX_TIMESTAMP` 内置函数。
* 支持“逻辑删除”转换功能，将来自上游的 DELETE 事件转换为带特殊标记的 INSERT 事件。

### Connectors

* 适用于 OceanBase 和 MaxCompute 的 Pipeline 连接器现已提供。
* 为接入增量快照框架的 CDC 连接器实现了异步分片功能，并完善了 Metrics 支持。
* 优化了 CDC 连接器 ROW 类型字段的反序列化效率。

#### OceanBase Pipeline Connector

OceanBase 是一款原生分布式关系型数据库，具备高性能和高可用性，能够支持海量数据和高并发。现在可以作为 YAML Pipeline Sink 使用。

#### MaxCompute Pipeline Connector

MaxCompute 是阿里云提供的分布式大数据处理平台，广泛应用于数据分析、报表生成、机器学习等场景。现在可以作为 YAML Pipeline Sink 使用。

#### Paimon Pipeline Connector

* 支持同步列默认值。
* 支持应用 TRUNCATE TABLE 和 DROP TABLE 事件。
* 更新 Paimon 依赖版本到 0.9.0。

#### MySQL Connector

* 支持解析 gh-ost 和 pt-osc 等无锁 Schema 变更工具产生的 DDL 变更事件。
* 新增是否将 TINYINT(1) 映射到 BOOLEAN 类型的配置。
* 支持同步表注释及行注释。
* MySQL CDC 下发的增量数据记录中现在携带 `op_ts` 元数据列，可以在 Transform 表达式中进行操作。

#### PostgreSQL CDC Connector

* 减少不必要的 Schema 查询，优化初次启动时间。
* 支持 Heartbeat 心跳包。
* 增加 `op_type` 元数据列。

### Common

* 新增了用于快速搭建数据集成验证环境的 cdc-up 脚本。

## 缺陷修复

* 修复了 MySQL CDC 处理新增表时可能的死锁问题。
* 修复了 MySQL CDC 处理 JSON 类型、带精度 FLOAT 类型的处理行为。
* 修复了 Paimon Sink 重复 commit 导致作业失败问题。
* 修复了 Transform 底层实现参数传递顺序问题。
* 修复了并发执行 Schema Evolution 时作业挂起的问题。
* 修复了作业失败重启后，Data Sink 内部状态不正确的问题。

## 致谢

感谢以下 37 名开发者对 Flink CDC 3.3 版本做出的贡献：

Chaoming Zhang, ConradJam, Hang Ruan, hiliuxg, Hongshun Wang, Jason Zhang, Junbo wang, Jzjsnow, jzjsnow, Kunni, Leonard Xu, liuxiaodong, MOBIN, MOBIN-F, molin.lxd, moses, North Lin, Olivier, ouyangwulin, Petrichor, Robin Moffatt, Runkang He, Sergei Morozov, Seung-Min Lee, Shawn Huang, stayrascal, Thorne, Timi, Umesh Dangat, wenmo, Wink, wudi, wuzhiping, Xin Gong, yuanoOo, yuxiqian, Zexian Wu

---

[1] https://github.com/apache/flink-cdc/releases/tag/release-3.3.0

[2] https://nightlies.apache.org/flink/flink-cdc-docs-stable

[3] https://flink.apache.org/what-is-flink/community

[4] https://github.com/apache/flink-cdc/discussions

[5] https://issues.apache.org/jira/projects/FLINK/summary