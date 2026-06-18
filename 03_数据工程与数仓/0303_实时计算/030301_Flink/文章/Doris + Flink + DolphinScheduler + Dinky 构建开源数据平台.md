---
title: Doris + Flink + DolphinScheduler + Dinky 构建开源数据平台
author: Dinky开源
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg3ODYxOTQxMA==&mid=2247486025&idx=1&sn=b3d5e1f5edb1ad8b9114964367561dca&chksm=cf11b434f8663d2298ce68a93ded103d4a571b0fd1f8306174cbea1eccd632f5aa919f245939&mpshare=1&scene=24&srcid=09014cFYsNgwpu46ewUXrFGO&sharer_sharetime=1661999470524&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

本文整理自 Dinky 实时计算平台 Maintainer 亓文凯老师在 Apache Doris & Apache SeaTunnel 联合 meetup 的实践分享，通过 Doris + Flink + DolphinScheduler + Dinky 构建开源数据平台。

**作者：**开源项目 Dinky 的发起人，亓文凯

GitHub 地址

https://github.com/DataLinkDC/dlink

https://gitee.com/DataLinkDC/Dinky

欢迎大家关注 Dinky 的发展~

**一、背景**

当前行业不断有许多新概念与新技术涌现，同时伴随着大量开源项目的诞生和发展，也有越来越多的企业转向开源软件。面对海量的业务需求和数据，应该如何高效地进行数据处理与分析，如何搭建一个数据平台？如何选择合适的开源项目来搭建呢？这是目前大家比较困扰的一个问题。

本次分享将介绍如何运用  Doris + Flink + DolphinScheduler + Dinky 四个开源项目来构建一个基本的数据平台，并支持离线、实时、OLAP 三种技术需求。

**二、开源数据平台思路**

本章节主要讲述数据平台搭建所用的开源项目介绍以及设计思路。

### **技术介绍**

#### Apache Doris

**首先要运用到的是 Apache Doris。Apache Doris 是一个基于 MPP 架构的高性能、实时的分析型数据库，以极速易用的特点被人们所熟知，仅需亚秒级响应时间即可返回海量数据下的查询结果，不仅可以支持高并发的点查询场景，也能支持高吞吐的复杂分析场景。**

**如图所示，一般在平台架构中，Doris 常作为数据仓库使用，并向用户提供各种实时高效的查询能力。其数据输入可以使用常见的数据集成框架或工具，如 Flink、Spark 等。此外 Doris 还可以以外表的形式连接 Hive、Iceberg 、数据湖及 MySQL、Oracle 数据库，这也为数仓转型和数据库分析带来更多易用便捷的能力。**

* 核心优势

1. 丰富的数据导入：提供丰富的数据同步方式，支持快速加载来自本地、Hadoop、Flink、Spark、Kafka、SeaTunnel 等业务系统及数据处理组件中的数据。
2. 强大的数据读取：Apache Doris 可以直接访问 MySQL、PostgreSQL、Oracle、S3、Hive、Iceberg、Elasticsearch 等系统中的数据，而无需数据复制。存储在 Doris 中的数据也可以被 Spark、Flink 读取，并且可以输出给上游数据应用进行展示分析。
3. 友好的数据应用：Apache Doris 支持通过 JDBC 标准协议将数据输出给下游应用，也支持各类 BI/客户端工具通过  MySQL 协议连接 Doris。基于此，Apache Doris 在多维报表、用户画像、即席查询、实时大屏等诸多业务领域都能得到很好应用。
4. 极致性能：高效的列存储引擎和现代 MPP 架构，结合智能物化视图、向量化执行和各种索引加速，实现极致的查询性能。
5. 流批一体：支持离线批量数据和实时流式数据高效导入，秒级实时性保证。多版本机制结合导入事务支持，解决读写冲突并实现 Exactly-Once。
6. 极简运维：高度一体，无任何外部组件依赖，集群规模在线弹性伸缩。系统高可用，节点故障自动副本切换，数据分片自动负载均衡。

#### Apache  Flink

**Flink 是一个计算框架和分布式处理引擎，主要用于无边界与有边界数据流上进行有状态的计算，Flink 能在所有常见集群环境中运行，并且能以内存速度和任意规模进行计算。在企业应用中，Flink 常用于高效连接消息流，如 Kafka，各种数据库、文件系统等，可以实时加工处理、也支持批处理，最终将数据高效写入消息流、数据库、软件系统等。**

* 核心优势

1. 高吞吐量、低延迟、高性能；
2. 支持 Event Time 和乱序事件；
3. 支持 Exactly-Once 语义；
4. 支持多种流式窗口；
5. 自身的内存模型与管理；
6. Batch 与 Stream 在 SQL 层实现一体；
7. 丰富的 Connector。

#### Flink CDC

    Flink CDC 是 Flink 的子项目，是 Flink 的一组原连接器，用于 CDC 从不同数据库接收/更改数据，Flink CDC 将 Debezium 集成为引擎，异步或数据更改，因此 Flink CDC 可以充分使用和发挥 Debezium 的能力，并且可以无缝对接 Flink 使用其 SQL API 和 DataStream API 的能力，最终写入各种数据源。

* 核心优势

1. 简化实时数据集成：无须额外部署 Debezium、Canal、Kafka 等组件，运维成本大幅降低，链路稳定性提升。
2. 支持丰富的数据源：目前支持 MongoDB、Mysql、OceanBase、Oracle、Postgres、SQLServer、TiDB 数据源的 CDC。
3. 支持全量、增量订阅及自动切换：能进行全量与增量自动切换，支持 Exactly-once 语义，支持无锁并发读取，支持从检查点、保存点恢复， 断点续传，保证数据的准确性。
4. 无缝对接 Flink：无缝对接 Flink 生态，利用 Flink 众多 Source 及 Sink 能力，可发挥 Flink 双流、流维关联等能力。
5. 支持 FlinkSQL：支持 FlinkSQL 定义 Flink CDC 任务，进一步降低使用门槛与运维成本。

#### Apache DolphinScheduler

**DolphinScheduler 是一个分布式去中心化，易扩展的可视化 DAG 工作流任务调度平台。致力于解决数据处理流程中错综复杂的依赖关系，使调度系统在数据处理流程中开箱即用。DolphinScheduler 采用了多 Master 和多 Worker 的实现形式，实现分布式、去中心化，支持 Flink、SQL等丰富的任务类型，且易于扩展。**

* 核心优势

1. 高可靠性：去中心化的多 Master 和多 Worker 服务对等架构, 避免单 Master 压力过大，另外采用任务缓冲队列来避 免过载。
2. 简单易用：DAG 监控界面，所有流程定义都是可视化，通过拖拽任务完成定制 DAG，通过 API 方式与第三方系统集成， 一键部署。
3. 丰富的使用场景：支持多租户，支持暂停恢复操作。紧密贴合大数据生态，提供 Spark，Hive，M/R，Python，Sub\_process， Shell 等近 20 种任务类型。
4. 高扩展性：支持自定义任务类型，调度器使用分布式调度，调度能力随集群线性增长，Master 和 Worker 支持动态上下线。

#### **Dinky 实时计算平台**

    Dinky 是基于 Apache Flink 二次开发的一款实时计算平台，主要为了更好地进行数据仓库和数据湖的建设与运维。Dinky 主要分为两大块，Data Studio 以及运维平台，数据开发方面主要支持 Flink SQL、Flink Jar 以及普通的 SQL 任务。

    Dinky 平台是通过 Flink API、Flink Client、Yarn、K8s 等提交和管理 Flink 任务，全过程只需要在 Dinky 中开发 Flink SQL ，不需要进行编译打包，Flink 任务就可自动提交到各种环境的集群。它支持多种提交方式，包括 Local、Standalone、Yarn、K8s 等方式，此外还提供了平台的管理能力，比如数据源、集群、监控报警的能力。对外可以通过 Flink 连接器以及数据源连接扩展来对外部数据源进行数据处理及管理。

* 核心功能

1. 沉浸式 FlinkSQL 和 SQL 的数据开发平台：自动提示补全、语法高亮、语句美化、语法校验、调试执行、执行计划、MetaStore、血缘分析、版本对比等
2. 支持多版本的 FlinkSQL 作业各种提交方式：Local、Standalone、Yarn/Kubernetes Session、Yarn Per-Job、Yarn/Kubernetes Application
3. 支持 Apache Flink 所有原生及扩展的 Connector、UDF、CDC 等
4. 支持 FlinkSQL 语法增强：兼容 Apache Flink SQL、表值聚合函数、全局变量、执行环境、语句合并、整库同步、共享会话等
5. 支持易扩展的 SQL 作业：ClickHouse、Doris、Hive、Mysql、Oracle、Phoenix、PostgreSql、SqlServer 等
6. 支持 FlinkCDC（Source 合并）整库实时入仓入湖
7. 支持实时调试预览 Table 和 ChangeLog 数据及 Charts 图形展示
8. 支持 Flink 元数据、数据源元数据查询及管理
9. 支持实时任务运维：上线下线、作业信息、集群信息、作业快照、异常信息、数据地图、数据探查、历史版本、报警记录等
10. 支持作为多版本 FlinkSQL Server 以及 OpenApi 的能力
11. 支持易扩展的实时作业报警及报警组：钉钉、微信企业号、飞书、邮箱等
12. 支持完全托管的 SavePoint/CheckPoint 启动及触发机制：最近一次、最早一次、指定一次等
13. 支持多种资源管理：集群实例、集群配置、Jar、数据源、报警组、报警实例、文档、用户、系统配置等

* 核心优势

1. 多兼容：基于 Apache Flink 源码二次开发，兼容官方 1.11~1.15 版本源码，也兼容用户自己的分支改进版。支持官方及其他扩展的 SQL Connector，如 ChunJun。支持 FlinkCDC 官方的 CDC SQL Connector。
2. 无侵入：Spring Boot 轻应用快速部署，不需要在任何 Flink 集群修改源码或添加额外插件，无感知连接和监控 Flink 集群。如果要使用 Flink MetaStore、整库同步等功能，则需要在 Flink lib 中添加对应的依赖包。
3. 无依赖：只需要 Mysql 数据库与 JDK1.8 环境，不依赖任何其他中间件，如 zookeeper、hadoop 等。
4. 易用性：Flink 多种执行模式无感知切换，支持 Flink 多版本切换，自动托管实时任务、恢复点、报警等， 自定义各种配置，持久化管理的 Flink Catalog (即 Flink MetaStore)。
5. 增强式：兼容且增强官方 FlinkSQL 语法，如 SQL 表值聚合函数、全局变量、CDC 整库同步、执行环境、 语句合并、共享会话等。
6. 易扩展：源码采用 SPI 插件化及各种设计模式支持用户快速扩展新功能，如连接器、数据源、报警方式、 Flink Catalog、CDC 整库同步、自定义 FlinkSQL 语法等。
7. 沉浸式：提供专业的 DataStudio 功能，支持全屏开发、自动提示与补全、语法高亮、语句美化、语法校验、 调试预览结果、全局变量、MetaStore、字段级血缘分析、元数据查询、FlinkSQL 生成等功能。
8. 一站式：提供从 FlinkSQL 开发调试到上线下线的运维监控及 SQL 的查询执行能力，使数仓建设及数据治理 一体化。
9. 易二开：源码后端基于 Spring Boot 框架开发，前端基于 React (Ant Design Pro) 开发，及其易扩展的设计， 易于企业进行定制化功能开发或集成到已有的开源或自建数据平台

### **设计思路**

    开源数据平台的设计思路是通过 Flink SQL Batch 以及 Doris SQL 的能力实现一个离线任务的开发；使用 DolphinScheduler 进行离线工作流编排和调度；通过 Flink CDC 和 Flink SQL 实现流处理能力，进行实时任务的开发；选择 Doris 作为实时数据仓库来写入数据并进行 OLAP 查询；通过 Dinky 来提供一个完整的任务开发运维的平台能力，满足常见的企业数据平台需求。

    在本文中，Doris SQL 是一个探索阶段的思路，目前主要以 Flink SQL 来实现。在我们的生产实践中，Doris SQL 多用于对 Flink 清洗加工后的明细数据进行快速关联统计，然后采用视图或将计算结果 insert 到下游 Doris 主题表里，最终通过 Doris 对下游 BI 或系统提供高效的 OLAP 查询支撑，减少实时和离线数仓的建设成本。

**三、离线数据分析平台**

接下来将分享如何构建一个离线的数据平台。

### **思路介绍**

**在离线数据分析平台中，Doris 作为核心数据仓库，Flink SQL Batch 和 Doris 的外部表提供一个数据的装载能力，也可以通过 Flink SQL Batch 和 Flink SQL 的提供一个 ETL 的能力，在 Dinky上进行 Flink SQL 和 Doris SQL 的开发、调试以及运维工作，而离线任务调度则使用 DolphinScheduler 来提供工作流的调度，最终通过 Doris 提供一个 OLAP 的能力，供下游的业务系统或 BI 系统来直接消费。**

### **FlinkSQL 写入 Doris**

    首先会运用到 Flink SQL 来写入Doris（本文介绍的是 Doris 版本为 0.15，1.1 版本改动较大请参考 Doris 官网文档），需要在 Flink Lib 以及 Dinky Plugins 下添加对应编译好的 Flink、Doris 的连接器，需要注意 Flink 、Scala、Doris对应版本，其次是如果要实现一个写入的更新，则需要开启一些配置。比如说：

1) `‘sink.enable-delete’ = ‘true’`

2) 只支持 Unique 模型

3) FlinkDDL 指定主键信息

    在写入的过程中，可能会由于换行符导致分割错误，我们通常会加上两行配置，重新定义下数据格式，解决分割错误：

```
'sink.properties.format' = 'json','sink.properties.strip_outer_array' = 'true'
```

### **FlinkSQL 读取 Doris**

**在 FlinkSQL 读取 Doris 过程中通常会遇到一个问题，在默认的 Doris 连接器实现中存在一个隐藏列，因此需要在 Flink DDL 中声明 Doris 的隐藏列，如下图所示。**

### **FlinkSQL 调试查询**

    之后可以在 Dinky 中来实现一个 FlinkSQL 的调试查询。下图是 Dinky 的开发页面，中间是SQL 开发编辑器，右侧是作业的配置；下方则是 FlinkSQL 实时调试的查询结果，类似于 SQL-Client。

### **Flink Catalog 管理**

    为方便进行 Flink Catalog 的管理。

1. Dinky 实现了 MysqlCatalog 管理，为区分其他社区及个人实现的 MysqlCatalog，其 Type 为 Dlink\_MySQL。
2. Dinky 的 MysqlCatalog 管理需要进行数据库表的初始化，即执行根目录下 SQL 文件夹里的 dlinkmysqlcatalog.sql 脚本。如果要使用 FlinkSQL 环境中默认的 default\_catalog ，则需要在 dlink 的数据库中执行该脚本。也可以在其他 MySQL 数据库中执行该脚本，则需要自定义 FlinkSQLEnv 任务来提供 Catalog 的环境。
3. FlinkSQLEnv 中可以定义多个 Catalog 共同使用，如 Flink 官方的 HiveCatalog。在 FlinkSQL 任务中使用 catalog.database.table 来操作表，或者使用 use catalog 来切换不同的 Catalog。

### **FlinkSQL 表值聚合函数 AGGTABLE**

Dinky 提供了很多特殊的语法。比如表示聚合的语法：AGGTABLE

1. Dinky 自定义了 CREATE AGGTABLE ... AS ... 语法， 该语法实现了 TableApi 层的表值聚合处理，使得 SQL 也 支持该操作（Flink 官方未实现）。
2. AGGTABLE 中 GROUP BY 为分组字段，AGG BY 为 聚合函数及其输出的字段信息。如 AGG BY TOP2(score) as (score,rank) 则为对 score 字段进行分组聚合操作，取每组内最大值与次大值，然后返回多行结果。
3. 对于表值聚合函数的自定义，则需要按照 Flink 官网中 的 Table Aggregate Functions 文档进行扩展。扩展完成后打包成 jar 文件，将其添加至 Dinky 的 plugins 和 Flink 的 lib 下，重启 Dinky 与 Flink 则生效。由于内存 Catalog 及 Dinky 的 default\_catalog 中不包含该函数的定义，所以使用时需要先通过 CREATE FUNCTION ... AS ... 进行自定义函数的注册。

### **FlinkSQL 全局变量**

    全局变量在企业数据开发中是非常关键和灵活的。

1. 通过开启右侧作业配置中的全局变量可以启用 Dinky 内部实现的 FlinkSQL 全局变量（SQL 片段）功能，可以将需要复用的 SQL 片段或变量值进行定义，避免重复维护的工作。
2. 语法：变量名:=变量值; 注意事项：全英文符号，以英文分号标志结束。
3. 全局变量会自动加载数据源链接配置变量，可以规避敏感信息， 如数据库连接地址及用户名和密码。如图 Doris （数据源实例名） 变量则为 Doris OLAP 的链接配置，可以在 Flink DDL 中直接使用。

### **FlinkSQL 血缘分析**

    Dinky 提供了字段血缘分析的能力，可以从 Flink SQL 作业中的多个 insert 语句中分析出该任务的字段血缘分析。

    此前 Dinky 是基于 StreamGraph 来分析 Flink 血缘，在 Local 的环境下对 StreamGraph 进行字段血缘的推导。在推导的过程中，通常会运用到一些比较关键的信息，下图中所展示的是 SQL 任务提交过程中构建的 StreamGraph，血缘实现的原理是基于 Pact、Contents、Predecessors 等参数实现 Source 和 Sink 字段关系的推导，但是有一些自定义的 UDF 以及连接器是不包含类似元数据的信息，比如说 Hudi 的连接器是无法进行 Hudi 的血缘。

    目前 Dinky 的血缘算法进行了优化，直接通过 Optimized Logical Plan 来进行推导，降低了算法成本，并且解决了上文提到的 UDF 与 Hudi 的问题。

### **Doris 血缘分析**

    Dinky 也提供了 Doris 的血缘分析能力。该血缘分析主要是基于 Alibaba Druid 来实现的。

### **构建 DolphinScheduler 工作流任务**

    基于上文血缘分析的能力，可以人工编排在数据仓库中的 DAG 工作流。主要是通过 DolphinScheduler 的工作流进行处理，在 DolphinScheduler 中扩展了 Dinky 的作业类型，目前需要等到 3.1 的版本才可以使用。

### **任务监控**

    通过 DolphinScheduler 调度的任务，在 Dinky 计算平台中也可以实时看到作业的运行情况。在运维中心中可以看到每个作业的细节，在 DolphinScheduler 所实现的任务类型既支持实时也支持离线，实时需要等到任务触发手动停止才会跑到下一个节点，离线则会等到离线作业 Finished，再进行到下一个节点。Dinky 提供了简单的一个任务监控，也提供了 Flink Web UI 的快捷跳转的按钮跳转到 Web UI 来查看更多详细信息。

### **作业发布版本对比**

    在管理大量作业的时候，通常会发生数据口径 FlinkSQL 的变化。Dinky 提供了作业版本管理的能力，在发布的时候会自动创建一个新的版本，后续可以对比每个版本中 SQL 的差异性，以及进行作业版本回滚操作。

### **Doris OLAP 及 Charts 渲染**

    Dinky 支持 Doris SQL 语句查询和执行能力，此外还提供了简单的 BI 的功能，可以将 Doris 语句的查询结果进行 BI 渲染，如柱状图、折线图、饼图等。

### **总结**

1.核心存储层选择 Doris 来提供统一数仓的能力，可以同时支持实时数据服务、交互数据分析和离线数据处理场景。

2.计算层选择 Flink 来提供各种数据源数据离线采集及清洗转换的 ETL 能力。可以发挥其高吞吐、高性能的优势。

3.平台层选择 Dinky 来提供数据开发与运维的能力， 通过 DolphinScheduler 提供工作流调度的能力。

**四、实时数据分析平台**

    本文分享的的实时数据分析平台解决方案链路比较短。主要是：

* Flink CDC 作为 CDC 技术；
* Flink SQL 流处理能力；
* Dinky 整库同步能力；
* Doris 提供 OLAP 查询能力，上游通过实时写入，Doris 数据查询也具备了一定的实时性；
* Dinky 在实时数据的开发上提供了监控和报警的能力，当任务触发异常或者是完成的时候，可以通过钉钉或邮箱等进行一个报警通知。

### **FlinkCDC+FlinkSQL 入库 Doris**

    在使用 Flink CDC 和 FlinkSQL 的过程中：

    第一种方式是使用官方 Flink CDC + Flink SQL 进行 ETL 操作。对于实时性要求较高且比较独立重要的需求，比如：不是在 Doris 中进行一个数仓的分层处理的，如 DWD、DWS 等，可以从源头 CDC 进行流处理后将结果写入 Doris 中，再通过 Doris 供上游 BI 系统直接使用 MySQL 协议来进行查询消费。

    Flink CDC 目前支持了非常多的数据源，我们主要用到关系型的数据库，比如 MySQL、Oracle、Postgres 等 。Flink CDC 每一个 DDL 都会创建一个连接数，需要在使用的过程中保证足够的连接数，否则可能会影响业务系统的正常运转。

### **Dinky 整库同步 CDCSOURCE 语法**

    第二种方式是 Dinky 提供了整库同步 CDCSOURCE 语法来进行 ELT 操作 。CDCSOURCE 语法会创建一个完整的连接数只有 1 的 FlinkCDC 整库同步任务。主要是使用了分流原理，此外可以通过 Sink 来指定下游数据库的各种的配置。它在创建任务时，会自动获取数据源元数据信息，自动映射出对应的字段名和类型，自动构建每个表的 Sink，且支持 Flink SQL 的所有 Sink 类型。当前如果源库 DDL 发生变动时，通常只能通过从恢复点重启 CDCSOURCE 任务来自动映射变动后的 DDL。

### **Dinky 整库同步原理**

    整库同步的原理是一个分流的操作：从 CDC 中读取出数据换成 Map，通过 Map 对 Schema 和 Table 进行过滤 ，过滤完之后组装成 Datastream<Row> ，最终再将 Datastream 转换成 TemporaryView，转换成视图之后，就可以使用 Flink SQL 进行操作。目前的操作是直接将它输出到任意的一个 Sink，当然此处可以进行改造，比如添加其他的定制处理，此外在上图的第四步，在组装  Datastream<Row>  后，也可以直接使用 Datastream 的 API 进行输出。当前 Dinky 对第三步 Filter 进行了优化，采用测输出流的方式来提升吞吐性能，提升约 20 倍。   

### **总结**

1.采集层使用 Flink CDC 替代传统的 CDC 方案，功能更全面，且无缝对接 FlinkSQL。

2.对于 ETL 需求可以直接使用 FlinkSQL+ Flink CDC 计算完成后入库 Doris。

3.对于 ELT 需要整库同步至 Doris 作为 ODS 进行实时 OLAP 时，可以采用 Dinky 整库同步来快捷构建实时任务。

**五、未来规划**

### **Dinky Roadmap**

1. 多租户及角色权限的实现：需要一个多租户的能力来分离不同数据团队或项目间的业务数据，需要角色权限来授权作业、 资源等使用，满足企业的基本管理需求。
2. 全局血缘与影响分析：需要将已发布的作业的字段级血缘进行存储，以构建全局的血缘和影响分析，方便用户更容易地追溯数据问题，同时也可为自动构建 DolphinScheduler 提供 DAG 关系基础。
3. 统一元数据管理：需要统一的元数据中心来管理外部数据源元数据，使其可以自动同步数据库物理模型与平台逻辑模型之间的结构，增强平台一站式的开发能力。
4. 多版本 Flink-Client Server：需要实现客户端与服务端分离，单独实现多版本的 Server。

****Dinky on Doris Roadmap****

1. 支持 Doris 外表创建等特殊语句的校验和执行：Dinky 目前不能执行 Doris 的特殊语法，如 SHOW、扩展表等语句。需要支持全面的 Doris 语句以提供一站式的开发运维能力。
2. 支持 Doris 语句执行记录：Dinky 目前只记录了 FlinkSQL 语句的执行情况，需要对 Doris 等 SQL 支持执行记录以追溯历史操作。
3. 支持 Doris 元数据的可视化管理：Dinky 目前只能查询 Doris 等的元数据，通过 DDL 语句进行元数据管理。后续需要支持对 Doris 等数据源的可视化管理，提升沉浸式体验。
4. 选择以 Doris 为存储层：Dinky 未来将以 Doris 为最佳存储层，分享更多的 Doris 数仓方面实践。

******Dinky on Flink Roadmap******

1. 支持 UDF 的自动化托管：Dinky 目前不能托管 UDF，需要人工打包并上传 Jar，然后重启实例才可生效，而且无法做到 UDF 的隔离。在多版本的 Flink-Client Server 实现后，将 Flink-Client 环境进行隔离，以支持 UDF 的自动加载及隔离管理。
2. 支持可视化构建 Flink 集群及其依赖：Dinky 目前可以根据 flink-conf 及其相关依赖来自动构建 Per-Job 和 Application 集群，该过程需要人工解决依赖间的问题以及版本的选择。后续将支持通过页面可视化配置用户预期的 Flink 环境，Dinky 自动将 Flink 环境部署或准备就绪，向 Flink 全托管前进。
3. 跟从 FlinkCDC 社区探索 Schema Evolution 和整库同步：Dinky 目前虽然支持整库同步的自动构建，但无法动态同步 DDL 变动，以及在大量表构建时存在性能问题。后续将跟从上游社区探索和实践相关内容。
4. 支持更多 Flink 生态：Dinky 未来将支持更多的 Flink 生态，如 Flink CEP、Flink ML、Flink Table Store、Flink Kubernetes Operator 等。

********Dinky on DolphinScheduler Roadmap********

1. 优化非 FlinkSQL 的任务执行：DolphinScheduler 的 Dinky Task 目前主要支持 FlinkSQL 批流任务的执行、监控、停止，对其他任务类型支持待优化，可能出现意外的问题。
2. 支持自动在 DolphinScheduler 上构建任务实例：Dinky 后续支持在 Dinky 数据开发页面上可以一键通过 API 来自动构建 DolphinScheduler 的任务实例， 避免用户需要频繁切换平台来配置调度任务。
3. 支持自动在 DolphinScheduler 上构建完整的工作流实例：Dinky 后续支持通过自身的全局血缘分析来推导 DolphinScheduler 工作流实例所需要 DAG 关系，并自动构建完整的工作流实例，并且可以在 Dinky 中对其进行基本的管理。
4. 选择以 DolphinScheduler 为调度平台：Dinky 未来将以 DolphinScheduler 为最佳调度平台进行深度集成，提供工作流编排能力。

**六、结束语**

    以上就是本次分享的所有内容，主要受益于 Doris 所提供的实时数仓 OLAP 能力，如有错误还请指出，谢谢。

    最后要感谢 Apache Doris 、Apache Flink、Apache DolphinScheduler、Flink CDC、Apache SeaTunnel、Apache Hudi 等社区的大力支持；感谢家峰、立冬等老师的开源指导；感谢我们39位的贡献者，以及数百位小伙伴的认可与同行。

**交流**

欢迎您加入社区交流分享与批评，也欢迎您为社区贡献自己的力量。

QQ社区群：**543709668**，申请备注 “ Dinky+企业名+职位”，不写不批。

微信官方群（推荐）：添加 wenmo\_ai ，申请备注“ Dinky+企业名+职位”，不写不批谢谢。

钉钉社区群（推荐）：

公众号：Dinky开源

扫描二维码获取

更多精彩

Dinky开源