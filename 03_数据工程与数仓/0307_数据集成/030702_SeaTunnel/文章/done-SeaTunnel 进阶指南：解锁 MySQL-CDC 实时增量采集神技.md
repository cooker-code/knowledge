> 已吸收至：[[03_数据工程与数仓/0307_数据集成/030702_SeaTunnel/030702_核心知识点/SeaTunnelCDC与批式同步边界|SeaTunnelCDC与批式同步边界]]
---
title: SeaTunnel 进阶指南：解锁 MySQL-CDC 实时增量采集神技
author: 白鲸开源
date:
url: https://mp.weixin.qq.com/s?__biz=MzkyODg0MzA1MQ==&mid=2247491409&idx=1&sn=19065833b6c05f50c3b7a63c847b6d6c&chksm=c3395fcb0f557dfb5684b0f74796a183c2347288b45cfdd7fcddbb6391cd9f1c947d116f04fb&mpshare=1&scene=24&srcid=06045fpbfBVm3SsfsvDqTIu0&sharer_shareinfo=74d296d16ad044855262b1b792d6590f&sharer_shareinfo_first=74d296d16ad044855262b1b792d6590f#rd
---

https://github.com/apache/seatunnel

**点击蓝字**

**关注我们**

**实时增量采集：变更数据捕获（CDC）**

* mysql-cdc官方文档：https://seatunnel.apache.org/zh-CN/docs/2.3.3/connector-v2/source/MySQL-CDC/
* cdc可以一个seatunnel的cdc任务监控多个表，进行同步
* 必须用mysql8.0.33以上的jdbc驱动

+ 下载地址：https://downloads.mysql.com/archives/c-j/

### 1、SeaTunnel支持几个数据库的CDC？

* https://seatunnel.apache.org/zh-CN/docs/2.3.12/connector-v2/source

```
|  |
| --- |
| 官方支持的 CDC 源连接器 |
|  |
| 1、MongoDB CDC 源连接器 |
| 2、MySQL CDC 源连接器 |
| 3、Opengauss CDC 源连接器 |
| 4、Oracle CDC 源连接器 |
| 5、PostgreSQL CDC 源连接器 |
| 6、SQL Server CDC 源连接器 |
| 7、TiDB CDC 源连接器 |
```

### 2、CDC的SeaTunnel服务启动后，不会停止

一个**标准的CDC（Change Data Capture）任务，其设计目标就是“持续运行，永不停止”的流式服务**。

* **核心区别**：

+ **离线/批处理任务：**

  `job.mode = "BATCH"`。执行一次全量或增量的SQL查询，处理完现有数据后，**任务自然结束**。
+ **CDC/流处理任务：**

  `job.mode = "STREAMING"`。启动后，它会：

1. 首先进行可选的**初始全量快照**（如果配置了`scan.startup.mode = "initial"`）。
2. 然后**挂起并持续监听**MySQL的binlog文件流。
3. 一旦源数据库有任何新的数据变更（增、删、改），任务会**立即捕获、处理并写入**目标端。
4. 这个监听过程**理论上会无限期进行下去**，直到你**手动停止**任务，或任务因错误而失败。

* **如何停止任务**：在运行SeaTunnel的终端中，通常可以按 **`Ctrl + C`** 组合键来优雅地停止正在运行的CDC任务。在生产环境中，可能需要通过作业调度或管理平台来发送停止指令。

简单来说，你可以把CDC任务理解为一个 **“常驻的订阅服务”** ，它订阅了数据库的变更日志，并在有新事件时实时处理。这不同于执行一次就退出的“离线任务”。

* 💡 配置建议与常见问题

1. **关于 `startup.mode：`**

   对于首次启动任务，**强烈建议使用 `initial`**，这样可以确保先获取一份完整的当前数据快照，之后再无缝衔接增量变更。如果设为 `latest`，则会**丢失所有历史数据**，只监听启动后的新变更。
2. **关于 `server-id：`**

   生产环境中**务必手动设置一个唯一值**，避免使用随机值可能导致的冲突，从而引发任务不稳定。

### 3、CDC模式与jdbc模式

* 区别

| 特性维度 | **MySQL-CDC (流式处理)** | **JDBC批处理 (批处理)** |
| --- | --- | --- |
| **任务模式 (`job.mode`)** | `STREAMING` | `BATCH` |
| **数据来源** | 数据库的**二进制日志 (Binlog)** | 自定义的 **SQL 查询结果集** |
| **数据内容** | **变更事件流** (包含`op`操作类型、前后镜像数据、元数据) | **静态数据快照** (查询返回的原始行数据) |
| **配置核心** | **`table-names`** (指定要监控的表) | **`query`** 或 **`table` + `sql`** (指定要查询的SQL) |
| **同步类型** | **实时增量** (可先全量快照，再持续增量) | **一次性全量/批量增量** |
| **输出持续性** | **持续运行** ，直到手动停止 | **自动结束** ，数据读取完毕后任务停止 |
| **典型场景** | 实时数仓、实时分析、异地容灾 | T+1报表、数据迁移、历史数据补录 |
| **对源库压力** | 增量阶段持续低压力读取Binlog | 每次执行查询时产生一次性压力 |
| **数据完整性** | 可保证**不丢不重** (Exactly-once) | 依赖查询条件，可能重复或遗漏 |

* 区别图

```
|  |
| --- |
| +-------------------+       +------------------------------+ |
| |   开始数据同步     |       |        CDC流处理路径          | |
| +-------------------+       +------------------------------+ |
| |                            | |
| v                            v |
| +-------------------+       +------------------------------+ |
| |   选择同步模式     |-----> | 1. 指定表名                  | |
| |   (决策点)        |       |    table-names=["db.tbl"]   | |
| +-------------------+       +------------------------------+ |
| |                            | |
| |                            v |
| |                    +------------------------------+ |
| |                    | 2. 连接器读取表结构          | |
| |                    +------------------------------+ |
| |                            | |
| |                            v |
| |                    +------------------------------+ |
| |                    | 3. 全量阶段: 读取快照        | |
| |                    +------------------------------+ |
| |                            | |
| |                            v |
| |                    +------------------------------+ |
| |                    | 4. 增量阶段: 持续监听Binlog  | |
| |                    +------------------------------+ |
| |                            | |
| |                            v |
| |                    +------------------------------+ |
| |                    | 5. 输出结构化变更事件        | |
| |                    |   (+I/-U/+U/-D 等)           | |
| |                    +------------------------------+ |
| |                            | |
| |                            | |
| v                            v |
| +-------------------+       +------------------------------+ |
| |  批处理JDBC路径   |       |                              | |
| +-------------------+       |       数据汇聚               | |
| |                 |       (写入Sink)             | |
| v                 |                              | |
| +-------------------+       +------------------------------+ |
| | 1. 自定义SQL查询   |                    ^ |
| |   SELECT ...      |                    | |
| +-------------------+                    | |
| |                              | |
| v                              | |
| +-------------------+                    | |
| | 2. 执行查询       |--------------------+ |
| |   读取结果集      | |
| +-------------------+ |
```

* 图2

* cdc下，无法用source的query定制sql过滤
* 如果要做数据清洗转换、过滤，只能再transform中做在MySQL-CDC模式下，你通常无法像使用普通的JDBC Source那样，在 source 模块里通过一个自定义的 query 参数（例如 SELECT \* FROM ... WHERE ...）来指定任意SQL语句。

CDC不支持自定义`query`的主要原因：

1. **数据来源不同**

* **JDBC批处理：数据来源于你自定义的**SQL查询结果集**。连接器只是执行并返回这个结果。**
* **MySQL-CDC：数据来源于数据库的**二进制日志（Binlog）**。CDC连接器（底层基于Debezium）会将自己伪装成一个MySQL副本，持续接收并解析原始的行级别变更事件流。**

2. **数据形态不同**

* **JDBC批处理：你查询出来的是什么数据，同步的就是什么数据。**

* **MySQL-CDC：它输出的是**变更事件**，每条记录不仅包含变更后的数据（`after`状态），还包含变更类型（`op`字段，如 `+I` 表示插入，`-U` 表示更新前，`+U`表示更新后，`-D` 表示删除）以及元数据（如源库、表、时间戳等）。自定义的 `SELECT` 语句无法生成这种结构化的变更事件。**

### 4、 如何在CDC模式下实现“筛选”或“转换”

虽然无法在数据源头（`source`）进行筛选，但你有多种方式在后续环节处理数据：

| 方法 | 实施位置 | 说明 |
| --- | --- | --- |
| **1. 使用 Transform 转换** | SeaTunnel 配置文件的 `transform` 部分 | 这是**最常用和推荐**的方式。你可以在数据流出Source后、写入Sink前，通过SQL语句进行过滤、字段选择、重命名等操作。 |
| **2. 在 Sink 中处理** | Sink 连接器的配置中 | 部分Sink支持在生成写入语句时进行条件过滤或字段映射，但能力有限，不如Transform灵活。 |
| **3. 启用 Schema 演进** | 表级别同步 | 通过只同步部分列的方式间接实现“列筛选”，但这需要表结构预先变更，且通常用于长期同步场景。 |

5、CDC可以同时监控多张表，进行实时同步

当你需要用一个CDC任务监控多个表时，SeaTunnel的配置非常直观。关键在于**源端（Source）的“一对多”配置**，以及**Sink的“一对一”自动映射**。

#### 📝 如何配置“一对多”CDC监控

你只需要在 `source` 部分的 `table-names` 列表中指定所有需要监控的表即可。

```
|  |
| --- |
| source { |
| MySQL-CDC { |
| base-url = "jdbc:mysql://192.168.1.107:51382/cs1" |
| username = "root" |
| password = "zysoft" |
| database-names = ["cs1"] |
| # 核心：在列表中添加多个表，格式必须是：数据库名.表名 |
| table-names = [ |
| "cs1.t_8_100w",          # 表1 |
| "cs1.order_table",       # 表2 |
| "cs1.user_profile"       # 表3 |
| # ... 可以继续添加更多表 |
| ] |
| startup.mode = "initial" |
| server-id = 5400 |
| server-time-zone = "Asia/Shanghai" |
| } |
| } |
```

#### 🎯 Sink的配置：自动路由与并行写入

SeaTunnel的JDBC Sink有一个非常强大的特性：**自动表路由**。你几乎**不需要为多表同步做特殊配置**。

**1. 核心配置（与单表一致，关键在 `table` 参数）：**

```
|  |
| --- |
| sink { |
| jdbc { |
| url = "jdbc:mysql://192.168.1.107:51382/cs2" |
| driver = "com.mysql.cj.jdbc.Driver" |
| user = "root" |
| password = "zysoft" |
| generate_sink_sql = true |
| database = "cs2" # 目标数据库 |
| # 核心技巧：使用变量动态匹配来源表名 |
| table = "${table_name}" |
| # table = "prefix_${table_name}"  # 也可以加前缀 |
| # table = "${database_name}_${table_name}_suffix" # 或使用库名、后缀 |
|  |
| schema_save_mode = "CREATE_SCHEMA_WHEN_NOT_EXIST" |
| data_save_mode = "APPEND_DATA" |
| batch_size = 5000 |
| # ... 其他连接和调优参数保持不变 |
| } |
| } |
```

**关键说明：**

* **`${table_name}`**

  和 **`${database_name}`** 是SeaTunnel的**内置变量**。运行时，它们会自动被替换为上游CDC数据记录中携带的**源表名**和**源数据库名**。
* 这样，`cs1.t_8_100w` 的数据会自动写入 `cs2.t_8_100w`，`cs1.order_table` 的数据会自动写入 `cs2.order_table`，实现完美的 **1:1 映射**。

**2. 如果需要对不同表采用不同的写入策略怎么办？**你需要为每个表单独配置一个 `sink` 模块，并使用 `filter` 条件进行路由：

```
|  |
| --- |
| sink { |
| # Sink 1: 专门处理 t_8_100w 表 |
| jdbc { |
| # 通过目标表名固定，明确写入哪个表 |
| table = "t_8_100w_target" |
| # 使用 filter 只让来自特定源表的数据流入此sink |
| source_table_name = "t_8_100w" |
| # ... 其他配置 |
| } |
| } |
| sink { |
| # Sink 2: 专门处理 order_table 表，可以采用不同的 data_save_mode 等 |
| jdbc { |
| table = "order_table_target" |
| source_table_name = "order_table" |
| data_save_mode = "DROP_DATA" # 例如，对这个表采用清空重灌策略 |
| # ... 其他配置 |
| } |
| } |
```

#### 💡 重要提醒与最佳实践

1. **目标表结构：确保目标数据库 `cs2` 中已经存在与源表结构兼容的表（或启用 `schema_save_mode = “CREATE_SCHEMA_WHEN_NOT_EXIST”` 让SeaTunnel自动创建）。**
2. **性能与隔离：监控多表时，所有表的变更会共用同一个数据流。如果某个表变更异常频繁，可能会轻微影响其他表的同步延迟。对于重要性或流量差异极大的表，**建议拆分到独立的CDC任务中**，以获得更好的隔离性和可维护性。**
3. **初始快照：当 `startup.mode = “initial”` 时，任务启动时会**依次对所有监控表进行全量快照读取**。请确保数据库有足够资源应对同时进行的多个全表扫描。**

**总结：对于大多数“多表同步到对应结构目标表”的场景，你只需要在 `source` 中列出多个表，然后在 `sink` 中配置 `table = “${table_name}”` 即可，SeaTunnel会自动完成路由和写入。**

如果你需要对特定表进行特殊的数据转换，可以在 `transform` 部分使用SQL，并同样通过 `filter` 或条件判断来区分不同表的数据流。

#### 路由filter

4. **路由与过滤：如果你希望数据**有选择地**流入不同的 `sink`，而不是全部复制到每一个 `sink`，则需要在每个 `sink` 前配置 **`filter`** 或使用侧流输出等更高级的API（在配置文件中通常通过条件表达式实现）。**

### 💡 针对你“多表CDC”场景的配置建议

结合你之前的问题，一个典型的多表CDC同步到不同目标表的配置如下：

```
|  |
| --- |
| source { |
| MySQL-CDC { |
| table-names = ["cs1.t_order", "cs1.t_user", "cs1.t_log"] |
| # ... 其他配置 |
| } |
| } |
|  |
| sink { |
| # 订单表 -> 订单归档库 |
| jdbc { |
| table = "t_order_archive" |
| # 使用filter，只同步t_order表的数据 |
| filter { |
| source_table_name == "t_order" |
| } |
| # ... |
| } |
|  |
| # 用户表 -> 用户分析库 |
| jdbc { |
| table = "t_user_analysis" |
| filter { |
| source_table_name == "t_user" |
| } |
| data_save_mode = "OVERWRITE" # 对这个表采用覆盖策略 |
| # ... |
| } |
|  |
| # 日志表 -> 日志中心（这里示例写入HDFS） |
| hdfs { |
| path = "/data/lake/log/${source_table_name}/dt=${now(date='yyyy-MM-dd')}" |
| filter { |
| source_table_name == "t_log" |
| } |
| # ... |
| } |
| } |
```

**实现MySQL-CDC的步骤**

#### 1、查看MySQL Binlog是否开启、开启

```
|  |
| --- |
| -- 最直接的查看方式 |
| SHOW VARIABLES LIKE'log_bin'; |
| -- 更详细的查看，会显示binlog的文件名和路径 |
| SHOW MASTER STATUS; |
| -- 查看binlog的格式，确认是否为ROW |
| SHOW VARIABLES LIKE'binlog_format'; |
| -- 查看binlog镜像模式，确认是否为FULL |
| SHOW VARIABLES LIKE'binlog_row_image'; |
```

* 如果没开启binlog，需要修改mysql配置文件，然后重启

```
|  |
| --- |
| [mysqld] |
| server-id = 123# 设置一个唯一的服务器ID[citation:2] |
| log_bin = /var/lib/mysql/mysql-bin  # 开启binlog并指定路径 |
| binlog_format = ROW                  # 必须设置为ROW模式[citation:2][citation:3][citation:5] |
| binlog_row_image = FULL              # 必须设置为FULL[citation:2][citation:4] |
| expire_logs_days = 10# 日志保留天数，建议至少2天[citation:2] |
```

#### 2、Mysql-CDC常用参数

**演示MySQL-CDC（单表）**

* 建表

```
|  |
| --- |
| -- demo7-1-mysql-cdc2mysql-qxzh-st-107.conf |
| CREATETABLE cs2.`t_8_100w_imp_st_qxzh_cdc_demo7_1`  ( |
| `id` bigintNOTNULL COMMENT '主键', |
| `user_name` varchar(2000) NULL COMMENT '名字', |
| `sex` varchar(20) null COMMENT '性别：男；女', |
| `decimal_f` decimal(32, 6) NULL COMMENT '大数字', |
| `phone_number` varchar(20)  COMMENT '电话', |
| `age` intNULL COMMENT '字符串年龄转数字', |
| `create_time` timestamp COMMENT '新增时间', |
| `description` longtext NULL COMMENT '大文本', |
| `address` varchar(2000) NULL COMMENT '空地址转默认值：未知', |
| PRIMARY KEY (`id`) |
| ); |
```

* 执行语句

```
|  |
| --- |
| # demo7-1-mysql-cdc2mysql-qxzh-st-107.conf |
| sh/data/tools/seatunnel/seatunnel-2.3.12/bin/seatunnel.sh --config /data/tools/seatunnel/myconf/demo7-1-mysql-cdc2mysql-qxzh-st-107.conf -m local |
```

* conf

```
|  |
| --- |
| # demo7-1-mysql-cdc2mysql-qxzh-st-107.conf |
| env{ |
| # 并行度（线程数） |
| execution.parallelism = 5 |
| # 任务模式：BATCH:批处理模式；STREAMING:流处理模式（CDC的关键） |
| job.mode = "STREAMING" |
| } |
|  |
| source{ |
| MySQL-CDC{ |
| base-url = "jdbc:mysql://ip:port/cs1" |
| username = "root" |
| password = "zysoft" |
| # query在cdc中无效 |
| # 数据库 |
| database-names = ["cs1"] |
| # 监控的表，必须带数据库名（The table name needs to include the database name, for example: database_name.table_name） |
| table-names = ["cs1.t_8_100w"] |
| # 启动模式：'initial'表示先做全量快照，再持续读增量；'latest'表示只从最新位点读增量 |
| # Optional startup mode for MySQL CDC consumer, valid enumerations are "initial", "earliest", "latest" and "specific". |
| startup.mode = initial |
| # 启动时间，可以设置这个时间之后再启动。startup.mode=timestamp时，这个参数必须有 |
| # startup.timestamp |
| # 会生成随机数，非常重要！指定CDC客户端的唯一ID（如5400）或范围（如5400-6408），不能与MySQL集群中任何现有服务器ID冲突。 |
| server-id = 5400 |
| # 停止模式 |
| # stop.mode |
| # 停止时间。当stop.mode=timestamp时，这个参数必须有 |
| # stop.timestamp |
| # 数据库服务器的会话时区，建议设置为 Asia/Shanghai 以正确解析时间戳。 |
| server-time-zone = "Asia/Shanghai" |
| } |
| } |
|  |
| # 清洗转换（cdc的清洗转换，必须在transform中来做） |
| transform{ |
| # 1. 字段映射 |
| # 除了Sql插件，还可以用：FieldMapper插件，来映射字段。必须写出目标需要的字段。不写的字段的值不会采集。 |
| FieldMapper{ |
| field_mapper = { |
| id = id |
| name = user_name |
| sex = sex |
| decimal_f = decimal_f |
| phone_number = phone_number |
| age = age |
| create_time = create_time |
| description = description |
| } |
| } |
|  |
| # 2. 手机号脱敏：13812341234 -> 138****1234 |
|  |
| # 3. 年龄转换：字符串转整数（实际生产中，不用转换，也没有内置的转换插件，可以直接保存成功） |
|  |
| # 4. 性别转换：1->男，2->女 |
|  |
| # 5. 数据过滤：只保留 age > 25 的记录。 |
|  |
| # 6. 地址默认值：空地址设为'未知' |
| } |
|  |
| sink{ |
| jdbc{ |
| url = "jdbc:mysql://ip:port/cs2" |
| driver = "com.mysql.cj.jdbc.Driver" |
| user = "root" |
| password = "zysoft" |
| # 生成自动插入sql。如果目标库没有表，也会自动建表 |
| generate_sink_sql = true |
| # generate_sink_sql=true。所以：database必须要 |
| database = cs2 |
| table = "t_8_100w_imp_st_qxzh_cdc_demo7_1" |
|  |
| # 表不存在时报错（任务失败），一般用：CREATE_SCHEMA_WHEN_NOT_EXIST（表不存在时创建表；表存在时跳过操作（保留数据）） |
| schema_save_mode = "ERROR_WHEN_SCHEMA_NOT_EXIST" |
| # APPEND_DATA：保留表结构和数据，追加新数据（不删除现有数据）(一般用这个) |
| # DROP_DATA：保留表结构，删除表中所有数据（清空表）——实现清空重灌 |
| data_save_mode = "APPEND_DATA" |
|  |
| # 批量写入条数 |
| batch_size = 5000 |
| # 批次提交间隔 |
| # batch_interval_ms = 0 |
| # 重试次数 |
| max_retries = 3 |
|  |
| # 连接参数 |
| # 连接超时时间300ms |
| connection_check_timeout_sec = 300 |
| properties = { |
| useUnicode = true |
| characterEncoding = "utf8" |
| serverTimezone = "Asia/Shanghai" |
|  |
| # 关键：启用批量重写 |
| rewriteBatchedStatements = "true" |
| # 启用压缩 |
| useCompression = "true" |
| # 禁用服务端预处理 |
| useServerPrepStmts = "false" |
| } |
| } |
| } |
```

* 结果

```
|  |
| --- |
| 2025-12-1115:23:41,170 INFO  [o.a.s.e.s.CoordinatorService  ] [pool-7-thread-1] - [localhost]:5801 [seatunnel-186786] [5.1] |
| *********************************************** |
| CoordinatorServiceThread Pool Status |
| *********************************************** |
| activeCount :                   2 |
| corePoolSize :                  10 |
| maximumPoolSize :          2147483647 |
| poolSize :                  10 |
| completedTaskCount :                 218 |
| taskCount :                 220 |
| *********************************************** |
|  |
| 2025-12-1115:23:41,172 INFO  [o.a.s.e.s.CoordinatorService  ] [pool-7-thread-1] - [localhost]:5801 [seatunnel-186786] [5.1] |
| *********************************************** |
| Jobinfo detail |
| *********************************************** |
| createdJobCount :                   0 |
| pendingJobCount :                   0 |
| scheduledJobCount :                   0 |
| runningJobCount :                   1 |
| failingJobCount :                   0 |
| failedJobCount :                   0 |
| cancellingJobCount :                   0 |
| canceledJobCount :                   0 |
| finishedJobCount :                   0 |
| *********************************************** |
|  |
| 2025-12-1115:23:46,167 INFO  [o.a.s.e.c.j.JobMetricsRunner  ] [job-metrics-runner-1051397352734064641] - |
| *********************************************** |
| JobProgress Information |
| *********************************************** |
| JobId                    : 1051397352734064641 |
| ReadCount So Far         :             1000009 |
| WriteCount So Far        :             1000009 |
| AverageRead Count        :                 0/s |
| AverageWrite Count       :                 0/s |
| LastStatistic Time       : 2025-12-11 15:22:46 |
| CurrentStatistic Time    : 2025-12-11 15:23:46 |
| *********************************************** |
|  |
| 2025-12-1115:23:46,573 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - wait checkpoint completed: 36 |
| 2025-12-1115:23:46,597 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - pending checkpoint(36/1@1051397352734064641) notify finished! |
| 2025-12-1115:23:46,597 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - start notify checkpoint completed, job id: 1051397352734064641, pipeline id: 1, checkpoint id:36 |
| 2025-12-1115:23:56,573 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - wait checkpoint completed: 37 |
| 2025-12-1115:23:56,588 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - pending checkpoint(37/1@1051397352734064641) notify finished! |
| 2025-12-1115:23:56,588 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - start notify checkpoint completed, job id: 1051397352734064641, pipeline id: 1, checkpoint id:37 |
| 2025-12-1115:24:06,573 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - wait checkpoint completed: 38 |
| 2025-12-1115:24:06,589 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - pending checkpoint(38/1@1051397352734064641) notify finished! |
| 2025-12-1115:24:06,589 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - start notify checkpoint completed, job id: 1051397352734064641, pipeline id: 1, checkpoint id:38 |
| 2025-12-1115:24:16,574 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - wait checkpoint completed: 39 |
| 2025-12-1115:24:16,603 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - pending checkpoint(39/1@1051397352734064641) notify finished! |
| 2025-12-1115:24:16,603 INFO  [.s.e.s.c.CheckpointCoordinator] [seatunnel-coordinator-service-8] - start notify checkpoint completed, job id: 1051397352734064641, pipeline id: 1, checkpoint id:39 |
```

* 结果展示

**·END·**

**白鲸开源**

白鲸开源是一家开源原生的DataOps商业公司，是国家高新技术企业，由多个Apache Foundation Member成立，80%员工都是 Apache Committer，运营2个全球Apache开源项目(DolphinScheduler, SeaTunnel）。白鲸开源已根据全球最佳实践发布商业版产品WhaleStudio(含白鲸数据调度平台WhaleScheduler和白鲸数据集成平台WhaleTunnel）。我们致力于打造下一代开源原生的DataOps 平台，助力企业在大数据和云时代，智能化地完成多数据源、多云及信创环境的数据集成、调度开发和治理，以提高企业解决数据问题的效率，提升企业分析洞察能力和决策能力。

**了解更多**

公司网站:www.whaleops.com

联系邮箱: xiyan@whaleops.com

如果您希望深入了解文中提到的数据质量功能，或者讨论如何将 WhaleStudio 与你的业务流程相结合，我们非常愿意为你提供帮助。**欢迎扫码获取****WhaleStudio产品白皮书**。

---

**下滑探索**更多WhaleStudio的优势，让我们帮助你构建一个高效、安全的大数据解决方案。🚀

## 金融行业的应用实例

****↓↓↓**点击下面链接阅读↓↓↓**

[国内某头部理财服务提供商基于白鲸调度系统建立统一调度和监控运维](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485832&idx=1&sn=28d8b40b2a752a431359a8cede446542&chksm=c171c1daf60648cc1724b11fc42a52c16ac0b8e29d26d6208cf897ba9c613ae0a1be152638a3&scene=21#wechat_redirect)

[白鲸调度系统助力国内头部券商打造国产信创化 DataOps 平台](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485636&idx=1&sn=a32f18255fdbf8b3510181ecf5d24a0d&chksm=c171c096f60649807f7c65a1a9b8de96b990539177752d3d31b21d9e15720d09f7647d1f8b98&scene=21#wechat_redirect)

[白鲸开源 DataOps 平台助力证券行业实现信创数字化转型](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247484746&idx=1&sn=545477e0c76725117276d0aa0ab157b7&chksm=c171cd18f606440e0e683dfe8f346f98965f0dcd85def2ebb51371a871d202d40792567371a1&scene=21#wechat_redirect)

[最佳实践 | 从Airflow迁移到Apache DolphinScheduler](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485916&idx=1&sn=b9aa20d813ea8c729c601167a2e31183&chksm=c171c18ef6064898a0b996fd9f83c9ad0a6d507d0a5429ff0fd2b407112fd67ec3fd7382c79b&scene=21#wechat_redirect)

[Apache DolphinScheduler VS WhaleScheduler](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485887&idx=1&sn=59b4ca05a425734b02b5e530eeb082a6&chksm=c171c1edf60648fb49ed9ad7a9b6315a470daff4f22534e52a7a017fd10f3084f2fd006f5fcd&scene=21#wechat_redirect)

[代立冬：基于Apache Doris+WhaleTunnel 实现多源实时数据仓库解决方案探索实践](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247486052&idx=1&sn=b869e1243e2b1e65f3435c73efb6b1f4&chksm=c171c236f6064b2059ff35e5e104392d2b82801158d531eca1e5def35b474eacef78edb29ea6&scene=21#wechat_redirect)

[白鲸开源在中信建投 DataOps 应用实践](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247486014&idx=1&sn=b157c765dee67f7a53dcb73efffaf871&chksm=c171c26cf6064b7a472bf00207ba93a88b23a164b9c14a84b4a35212e8060ee8955b69a5ad21&scene=21#wechat_redirect)

## 商业版技术解析实例

**点击下面链接阅读↓↓↓**

[被热议的“DataOps”是炒作？](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247486151&idx=1&sn=81cd58a5d2d4248570ec1413b4a9111a&chksm=c171c295f6064b83d52e3f782ea99377355f06811ecb22ad32661c1d2e233d2add7cfcb84760&scene=21#wechat_redirect)

[WhaleScheduler：高并发下的稳定性与性能实践](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485953&idx=1&sn=7a9077f6e6b4067df585b16f6894cc09&chksm=c171c253f6064b454931eac815afc084110ab6c89098e2b6bb9255fa9e90b3834b873284f859&scene=21#wechat_redirect)

[驾驭数据的未来：WhaleStudio与DataOps的完美结合](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485494&idx=1&sn=2ea61e0878601f6518ac542b7af4e01b&chksm=c171c064f6064972f48ac5dd6bd24e6b1ec040d913a8503e17579f6c06426a724b5d3ce355f3&scene=21#wechat_redirect)

[WhaleStudio：创新性解决大数据挑战的工具](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485713&idx=1&sn=f970b361de1b5dece21d323787077271&chksm=c171c143f6064855ec5de00d6b54b4f5e6bb46e61a80100a59e0e31bb772855ddb7e03946c84&scene=21#wechat_redirect)

[支持全生态调度：构建企业数字化转型的桥梁](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485769&idx=1&sn=b9f39414db17bdd5b2efd308f78a55f6&chksm=c171c11bf606480d01406f37501b63fb28c181bcd15208acb6cb5a94bf9a4726a6ba59656269&scene=21#wechat_redirect)

**运营开源项目**

目前，北京白鲸开源科技有限公司运营着已经从 Apache 基金会毕业的大数据工作流调度平台 Apache DolphinScheduler，以及数据集成平台 Apache SeaTunnel，诚邀全球伙伴加入开源共建！

**Apache DolphinScheduler：**

仓库地址：https://github.com/apache/dolphinscheduler

官网：https://dolphinscheduler.apache.org/

**Apache SeaTunnel：**

仓库：https://github.com/apache/seatunnel

官网：https://seatunnel.apache.org/

点个在看你最好看