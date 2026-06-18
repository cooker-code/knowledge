---
title: Flink2.1.0,有史以来最强大的一个版本，开启新的纪元！！！
author: 3分钟秒懂大数据
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247515254&idx=1&sn=6ad2451091d3f3f03f8036f537909c69&chksm=c1f98594f4ddc4c4cf921f2cab8a0fcac868a105802b5b19e8f42e30826552e50fd341f40a73&mpshare=1&scene=24&srcid=0814odCRt4dgTZ42U4cd03zO&sharer_shareinfo=85906cb37c9c418e308fcbc3062458a3&sharer_shareinfo_first=85906cb37c9c418e308fcbc3062458a3#rd
---

> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_版本记录|版本记录]]


```
点击上方公众号进入 3分钟秒懂大数据 主页

然后点击右上角 “设为标星” 

比别人更快接收硬核文章
```

## 前言

大家好，我是土哥。

作为 Flink 生态圈忠实的探索者，看到 Apache Flink 2.1.0 版本正式发布，异常兴奋，本次版本发布，标志着实时数据引擎向 Data + AI 平台的里程碑演进。

## 不同版本迭代

### Flink 1.6 版本

依稀还记得 2018 年，我第一次接触 Flink 组件，当时还是 Flink 1.6 版本，了解到 Flink 组件的框架原理，在本地集群部署使用了一下，发现比 Spark 好用，瞬间就有了兴趣，但当时功能还不是很完善，大概记得有几点

Flink 1.6 版本引进了几点改进内容：

* 状态后端改进：支持基于 RocksDB 的增量 Checkpoint —— 降低状态快照存储空间。
* SQL支持增强：引入Table API & SQL的流处理支持，包括窗口TVF（Table-Valued Functions）
* 改进了 Metrics 系统：增强了监控和度量功能。

### Flink 1.11

Flink 1.11 版本印象深刻，是因为 2020年 Flink 1.11 版本推出，而我当时负责改造 Flink 流式写入 Hive 的相关 Connector 代码. 在该版本中，重点引入以下核心能力：

* 1、核心引擎部分引入了非对齐的 Checkpoint 机制。这一机制是对 Flink 容错机制的一个重要改进，它可以提高严重反压作业的 Checkpoint 速度。
* 2、实现了一套新的 Source 接口。通过统一流和批作业 Source 的运行机制，提供常用的内部实现如事件时间处理，watermark 生成和空闲并发检测，这套新的 Source 接口可以极大的降低实现新的 Source 时的开发复杂度。
* 3、Flink SQL 引入了对 CDC（Change Data Capture，变动数据捕获）的支持，它使 Flink 可以方便的通过像 Debezium 这类工具来翻译和消费数据库的变动日志。Table API 和 SQL 也扩展了文件系统连接器对更多用户场景和格式的支持，从而可以支持将流式数据从 Kafka 写入 Hive 等场景。
* 4、PyFlink 优化了多个部分的性能，包括对向量化的用户自定义函数（Python UDF）的支持。这些改动使 Flink Python 接口可以与常用的 Python 库（如 Pandas 和 NumPy）进行互操作，从而使 Flink 更适合数据处理与机器学习的场景。

### Flink 1.14

初次接触 Flink 1.14 版本，是因为当时负责我司的 Flink 版本重构升级，从 Flink 1.11 升级到 Flink 1.14, 在内部进行了许多的逻辑改造，其中一个重点改造内容就是 Flink 1.14之后，之前旧版的 SQL 引擎代码全部被移除掉，同时删除掉许多过时的接口，除此之外，还有很多改造功能，比如：

* 有界流 Checkpoint 机制
* 缓冲区去膨胀，细粒度资源管理
* 连接器指标的改进
* 删除旧版 SQL 引擎的相关接口及代码等

### Flink 1.18

在 Flink 1.18 版本中，功能改进点也非常大，比如 Flink 1.18 版本提供了 Flink SQL Gateway 的 JDBC Driver，用户可以使用支持 JDBC 的任何 SQL 客户端通过 Flink SQL 与您的表进行交互。同时该版本正在迈向 Streaming Lakehouse

在 Flink 1.18 版本中，这里重点提几个核心改进点

* 1、DDL 支持扩展

从 1.18 版本开始，Flink 支持以下功能：

* REPLACE TABLE AS SELECT
* CREATE OR REPLACE TABLE AS SELECT

这两个命令以及之前支持的 CREATE TABLE AS 现在都支持原子性，前提是底层连接器也支持。

此外，Apache Flink 现在支持在批处理模式下执行 TRUNCATE TABLE。与以前一样，底层连接器需要实现并提供此功能。

最后，还实现了通过以下方式支持添加、删除和列出分区：

* ALTER TABLE ADD PARTITION
* ALTER TABLE DROP PARTITION
* SHOW PARTITIONS
* 2、流处理提升

Table API & SQL 支持算子级别状态保留时间（TTL）

从 Flink 1.18 版本开始，Table API 和 SQL 用户可以为有状态的算子单独设置状态保留时间 (TTL)。在像流 regular join 这样的场景中，用户现在可以为左侧和右侧流设置不同的 TTL。在以前的版本中，状态保留时间只能在 pipeline 级别使用配置项 table.exec.state.ttl 进行控制。引入算子级别的状态保留后，用户现在可以根据其具体需求优化资源使用。

* SQL 的水印对齐（Watermark Alignment）和空闲检测（Idleness Detection）

用户可以使用 SQL Hint 配置水印对齐和数据源空闲超时。之前这些功能仅在 DataStream API 中可用。

* 通过 REST API 控制动态细粒度扩缩容

自 Flink 1.18 起，在作业运行时，可以通过 Flink Web UI 和 REST API 更改作业的任何 task 的并行度。

实现细节上，Apache Flink 在获得新并行度所需的资源后会立即执行扩缩容操作。扩缩容操作不基于 savepoint，而是基于普通的定期 checkpoint，这意味着它不会引入额外的 snapshot。对于状态规模较小的作业，重新调整操作几乎立即发生，且中断时间非常短。

与 Apache Flink Web UI 的反压监控相结合，现在更容易找到并维护使每个任务高效运行、无反压的并行度。

如果一个任务非常繁忙（红色），您可以增加并行度。 如果一个任务大部分时间处于空闲状态（蓝色），您可以减少并行度。

## Flink 2.1.0 新纪元

在 Flink 2.1.0 版本中，重点强化了实时 AI 与智能流处理的深度融合

* 实时 AI 能力突破：

新增 AI 模型 DDL，支持通过 Flink SQL 与 Table API 创建和修改 AI 模型，实现 AI 模型的灵活管理。

扩展 ML\_PREDICT 表值函数，支持通过 Flink SQL 实时调用 AI 模型，为构建端到端实时 AI 工作流奠定基础。

* 实时数据处理增强：

通过开放 Flink 核心能力——托管状态、事件时间流处理及表变更日志，Process Table Functions(PTFs) 使 Flink SQL 解锁了更强大的事件驱动型应用开发能力

引入 VARIANT 数据类型，高效处理 JSON 等半结构化数据，结合PARSE\_JSON函数与湖仓格式（如 Paimon），实现动态 Schema 数据分析。

重点优化流式 Join，创新性的引入 DeltaJoin 与 MultiJoin 策略，消除状态瓶颈，提升资源利用率与作业稳定性。

* Delta Join

引入一种新的 DeltaJoin 算子，相比传统的双流 Join 方案，配合 Apache Fluss 等流存储，可以实现 Join 算子无状态化，解决了大状态导致的资源瓶颈、检查点缓慢和恢复延迟等问题。该特性已默认启用，同时需要依赖 Apache Fluss 存储相应版本支持。

* 级联双流 Join 优化

使用多个级联流式 Join 的 Flink 作业经常会因 State 过大而导致运行不稳定和性能下降。在这个版本，引入了一种新的 MultiJoin 算子， 通过在单个算子内直接进行多流 Join，消除中间结果存储，每条流输入的记录最多存储一份，从而实现实现 "零中间状态" ，显著提升了资源利用率与作业稳定性。目前仅支持 INNER、LEFT JOIN ，并且需要通过参数 table.optimizer.multi-join.enabled=true 启用。

* 异步 Lookup Join 增强

异步 Lookup Join 在之前的版本中，即使用户将 table.exec.async-lookup.output-mode 设置为 ALLOW\_UNORDERED，引擎在处理更新流时仍会强制回退到 ORDERED 以保证正确性。从 2.1 版本开始，引擎允许并行处理无关的更新记录，同时保证正确性，从而在处理更新流时实现更高的吞吐。

* Sink 节点复用

当单个作业中多个 INSERT INTO 语句更新目标表相同列时（下版本将支持不同列），优化器将自动合并 Sink 节点实现复用，该特性可以极大提升 Apache Paimon 等湖仓格式的部分更新场景使用体验。

* Compiled Plan 支持 Smile 格式

新增 Smile 二进制格式（兼容 JSON 格式）用于执行计划序列化，较 JSON 更节省内存。默认仍使用 JSON 格式，需通过显示调用 CompiledPlan#asSmileBytes 和 PlanReference#fromSmileBytes 方法启用。

* 异步 Sink 可插拔的批量处理

支持自定义批量写入策略，用户可根据业务需求灵活扩展异步 Sink 的批量写入逻辑。

* 分片级水位线指标

在 Flink 2.1 版本，新增细粒度分片监控指标，涵盖水位线进度与状态统计：

currentWatermark：该分片最新接收到的水位线值

activeTimeMsPerSecond：该分片每秒处于数据处理状态的时间（毫秒）

pausedTimeMsPerSecond：因水位线对齐该分片每秒的暂停时间（毫秒）

idleTimeMsPerSecond：每秒空闲时长（毫秒）

accumulatedActiveTimeMs：累计活跃时长（毫秒）

accumulatedPausedTimeMs：累计暂停时长（毫秒）

accumulatedIdleTimeMs：累计空闲时长（毫秒）

* PyFlink：新增支持 Python 3.12，移除 Python 3.8 支持。

同时依赖升级：flink-shaded 升级至 20.0 以支持 Smile 格式，Parquet 升级至 1.15.3 修复安全漏洞（CVE-2025-30065）。

## 总结

总之，Flink2.1.0的发布，标志着 Flink 正式迈入新的纪元，如果你是一名 Flink 爱好者，欢迎进群交流，一起探讨技术。

## 面经分享

现在已经8月了，土哥辅导的小伙伴，好多都拿到了 offer ， 包括蔚来、拼多多、度小满、流利说、群核科技等等，还有一些在 offer 谈薪中，面试题已经整理好。

这里放一些之前整理的面试题，希望对你有帮助

[史上最全干货！Flink面试大全总结（全文6万字、110个知识点、160张图）](https://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247497240&idx=1&sn=954c0702a2d842f9facb4e36c8c44563&scene=21#wechat_redirect)

[Spark面试干货总结！（8千字长文、27个知识点、21张图）](https://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247500540&idx=1&sn=1ff423566e052b2b509c28811e7ea2f0&scene=21#wechat_redirect)

[干货总结！Kafka 面试大全（万字长文，37 张图，28 个知识点）](https://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247502522&idx=1&sn=c40e90bc0b3795eb31f9ec39f2f64874&scene=21#wechat_redirect)

[史上最全系列 | Redis 原理+知识点总结（1.5万字、8大知识点、17张图）](https://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247505344&idx=1&sn=6868afa74c816ccd0334055c899e4081&scene=21#wechat_redirect)

[Hive的25个高频面试问点](https://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247512423&idx=1&sn=9f6b7f38ca7eb8ee3fe5da81b0bdd317&scene=21#wechat_redirect)

[字节跳动大数据架构面经（超详细答案总结）](https://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247508412&idx=1&sn=b2f5f3b54b9bf6fe6c5375db6f05901a&scene=21#wechat_redirect)

[字节跳动大数据架构 2 面面经（吐血答案总结，超详细）](https://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247508685&idx=1&sn=5a882fc2dbd443116b2d45f32ea17484&scene=21#wechat_redirect)

[#秒懂大数据面试](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg5NDY3NzIwMA==&action=getalbum&album_id=2221600476708552706#wechat_redirect)

👆点击关注｜设为星标｜干货速递👆

**增值服务：简历修改|面试辅导|Flink资料|**模拟面试****

你好，我是土哥，计算机硕士毕业，现某大厂资深大数据开发工程师。出生在一个 18 线开外的小村庄，通过自己努力毕业一年在新一线城市买房，在社招、校招斩获 28 家中大厂 offer。

2025已经到3月了，很多公司岗位流动性非常大，已经到了求职的高峰期。如果你想跳槽，但苦于一个人孤军奋战、无人指导、复习无从下手，或者不擅长写简历，手上只有拿不出手的毫无难点亮点的项目经历...

那么我的建议是多和身边的大佬沟通，哪怕是付费咨询，只要你能从他身上学到经验，那就是值得的。如果身边没有这样的人，那么我就毛遂自荐一下吧，毕竟，茫茫网络你能看到这篇文章何尝不是一种命运安排。

[土哥社招参加 28 场面试，100% 通过率，拿到全部 offer！](http://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247511408&idx=1&sn=beb292ab97ada3ee486511bfe503117d&chksm=c01914cff76e9dd90fd81857805a57aadcf4fa0a3ce731e5939d8651ed9bac561dba6bb7e03a&scene=21#wechat_redirect)

[土哥这半年的悲惨人生，经历过被鸽 offer，最终触底反弹~](http://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247510455&idx=1&sn=9cccfbebca3d2ee9d72538d73dd6fe74&chksm=c0191008f76e991e1760857f7c8a75deb1e0231ce8ba189eaf280c84e8a5e65f7f0197988ff1&scene=21#wechat_redirect)

需要任一增值服务或进大厂交流求职群

都欢迎加土哥微信（备注来意）