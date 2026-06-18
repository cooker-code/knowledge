---
title: 一键实现万表 MySQL 整库同步至 Apache Doris
author: 锋哥聊湖仓
date:
url: http://mp.weixin.qq.com/s?__biz=MzI4ODMyNTcwMw==&mid=2247485917&idx=1&sn=7428f5e0c179d8835a89d7c23fb04b02&chksm=ebc160f5dcb6e9e34f4c2456b2bbd5e9d394190d1e6260417cd1eb9fd31799059bd89ef28d2a&mpshare=1&scene=24&srcid=0619RbcWLZkMddwVNAwA1aGn&sharer_sharetime=1687136123794&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/030302_核心知识点/FlinkCDC整库同步Doris链路边界|FlinkCDC整库同步Doris链路边界]]


作为连接 Apache Doris 与 Apache Flink 的桥梁，Doris-Flink-Connector 实现了数据在 Flink 和 Doris 两端的实时快速传输，同时结合两阶段提交实现了数据写入过程中的一致性和准确性，进一步提升了数据同步和处理的效率与质量。

经过近半年的打磨，我们于近期**正式发布了 Doris-Flink-Connector 1.4.0 版本**、引入了一系列重要更新，**其中最为重要的即集成了 Flink CDC、实现了从 MySQL 等关系型数据库到 Apache Doris 的一键整库同步**。在实际测试中，单个同步任务可以承载数千张表的实时并行写入，从此彻底告别过去繁琐复杂的同步流程，通过简单命令即可实现上游业务数据库的表结构及数据同步。与此同时，新版本还实现了异步的 Lookup Join、实现了 Stream Load 连接按需链接、引入了轮询机制等。通过以上更新与优化，进一步简化了 Doris-Flink-Connector 使用成本并提升了数据导入效率与维表查询效率。此外，新版本已实现了对 Apache Flink 1.17 最新版本的支持。欢迎大家即时体验！

**Jar 包下载地址：**

https://github.com/apache/doris-flink-connector/releases/tag/1.4.0

**Maven 地址：**

```
<dependency>
  <groupId>org.apache.doris</groupId>
  <artifactId>flink-doris-connector-1.15</artifactId>
  <!--artifactId>flink-doris-connector-1.16</artifactId-->
  <!--artifactId>flink-doris-connector-1.17</artifactId-->
  <version>1.4.0</version>
</dependency>
```

重要优化

# 在之前版本中，如果想快速将上游整个 MySQL 业务库接入到 Doris 中，通常需要手动编写 DataStream 程序，并在 Apache Doris 中创建映射表。对于包含成千上万张表的业务库，整库同步就显得非常复杂，同步难度相对较高。

在 Apache Doris Flink Connector 1.4.0 版本中，**我们集成了 FlinkCDC 并支持了 MySQL 的整库同步，无需提前在 Doris 中建表、用户可以直接使用 Connector 快速将上游业务库的表结构及数据接入到 Doris 中**。具体实现为，当 Flink 任务启动后，Doris Flink Connector 将自动识别对应的 Doris 表是否存在。如果表不存在，Doris Flink Connector 会自动创建表，并根据 Table 名称使用侧输出流进行分流，从而实现下游多个表的 Sink 接入；如果表存在，则直接启动同步任务。

## **使用方法**

##

对上游 MySQL 业务库`mysql_db`中`tbl`表和`test`开头的表进行同步。只需要执行以下命令，即可将上游多表整库同步到 Doris 中，同时在 Doris 中无需提前创建表。

```
<FLINK_HOME>/bin/flink run \
    -Dexecution.checkpointing.interval=10s \
    -Dparallelism.default=1 \
    -c org.apache.doris.flink.tools.cdc.CdcTools \
    lib/flink-doris-connector-1.16-1.4.0.jar \
    mysql-sync-database \
    --database test_db \
    --mysql-conf hostname=127.0.0.1 \
    --mysql-conf username=root \
    --mysql-conf password=123456 \
    --mysql-conf database-name=mysql_db \
    --including-tables "tbl|test.*" \
    --sink-conf fenodes=127.0.0.1:8030 \
    --sink-conf username=root \
    --sink-conf password=123456 \
    --sink-conf jdbc-url=jdbc:mysql://127.0.0.1:9030 \
    --sink-conf sink.label-prefix=label1 \
    --table-conf replication_num=1
```

详细使用可参考：*https://doris.apache.org/zh-CN/docs/dev/ecosystem/flink-doris-connector*

## **同步性能**

##

经过调研了解，我们发现在常用的场景中，用户对于整库级别（百、千级别的表，且分为活跃与非活跃表）的数据同步，通常要求达到秒级别延迟。为验证同步性能是否满足要求，我们在同等条件下进行了测试。

我们对 1000 张 MySQL 表，单表 100 字段，均为活跃表（即持续向 MySQL 表发送数据，每张表单次写入上百条）的上游业务库进行同步。我们将 Flink 作业的 Checkpoint 时间配置为 10 秒，经过压测，在 Doris 2.0 的表现非常稳定，具体核心指标如下：

此外，部分社区用户基于该版本也在生产环境中进行了**万表**级别**的整库同步**，而查询性能和系统稳定性依然表现出色。这充分证明了 Doris 与 Flink CDC 结合后，在多表同步场景的高效性和可靠性，已经完全具备了大规模数据快速处理的能力。

## **实际使用反馈**

##

对用户特别友好，不用关注建表语句怎么写，大大提高了工作效率 

—— *盛**玉，蜀海供应链 资深大数据工程师*

通过 Flink-CDC 即可平滑地实现上游数据库到 Doris 的 DDL 同步，节省了大量的表结构维护时间，提升了实时数仓数据处理的时效性和便捷性。以前使用 Flink-CDC 时，每张表都需要建立 Job，并在源端建立日志解析链路，而新版本中利用该 jar 通过库级别或多表的同步方式，缩短了与源端的日志解析链路，大大减少了源端数据库的资源消耗。

*——**李俊**，恒信集团 资深大数据工程师*

解决了全量 + 增量数据同步的统一性问题，提高了数据同步的效率，同时支持了源数据结构实时同步，极大提升使用的体验。

—— *蔡工，**招商新智 架构师*

新版本可以将开发人员从重复枯燥而且容易出错的建表工作中解放出来。特别是面成百上千的表同步时，节省了以天为单位的时间成本，真正为用户提供了一个开箱即用的使用体验。

——*菜堆，**武汉手盟 资深大数据工程师*

最新特性

1. ## **维表 Join**

用户通常会将维表放在 Doris 中，并使用 Flink 的实时流进行关联查询，比如维度补全。在之前的版本中，一般会使用同步的方式进行交互，该方式在关联查询时往往会导致实时流阻塞，同时可能出现反压问题。而在 Doris-Flink-Connector 1.4.0 版本中，我们基于 Flink 的 Async IO 实现了异步的 Lookup Join，在使用 Flink Lookup Join 时，无需阻塞等待，大大提高了流处理的效率。同时 1.4.0 版本还引入了攒批查询，它可以将多个查询拼接为一个大查询，一次性发送给 Doris 进行处理，从而提高了维表查询的效率的数据的吞吐量。

2. ## **引入 Thrift SDK**

在之前版本中，当使用 Connector 读取 Doris 的数据时，需要依赖 Thrift 插件，因此在编译时必须配置 Thrift 环境，这无疑是增加了编译的难度和使用成本。基于此，我们在 1.4.0 版本引入了 Thrift-service SDK，屏蔽了 Thrift 插件，大大简化了编译流程。

3. ## **实现 Stream Load 按需连接**

在之前的版本中，每次 Checkpoint 都会创建一个 Stream Load 连接，因此同步任务中如果存在大量非活跃表时，也将创建大量的无用连接，而这些无用链接将极大增加 Doris 集群的压力。在 1.4.0 版本中，我们对此进行了优化，我们设定当没有数据导入的时候，将不会发起 Stream Load 请求，避免无效链接占用集群资源。

4. ## **轮询 BE 节点导入**

在之前的版本中，数据导入会先调用 FE 来获取 BE 的节点列表，并随机选择一台 BE 发起导入请求。被选 BE 节点将长时间充当 Stream Load 的 Coordinator 直到 Flink 任务重启才有可能变更，这样将导致被选 BE 节点长时间处于较大压力状态。为解决该问题，我们在 1.4.0 版本引入了轮询机制，轮询机制开启后，会在每个 Checkpoint 期间更换 BE 节点，从而避免单个节点长时间充当 Coordinator，达到压力分摊的效果。

5. ## **支持 Doris Array/JSON 等类型**

Doris-Flink-Connector 1.4.0 版本新增了对 Doris  DecimalV3/DateV2/DateTimev2/Array/JSON 等数据类型的支持。（支持 Doris 2.0 及以上版本)。

问题修复

* 修复 Stream Load 连接 Check 时的并发问题
* 修复连接 Apache Doris 部分超时参数不生效的问题
* 修复数据写入时隐藏分隔符不生效的问题
* 修复读取 Doris 查询计划过长，序列化失败的问题

使用示例

**读取 Doris：**目前支持使用 DataStream 和 FlinkSQL 的方式读取 Doris（有界流），同时支持条件下推。

```
```
CREATE TABLE flink_doris_source (
    name STRING,
    age INT,
    score DECIMAL(5,2)
    ) 
    WITH (
      'connector' = 'doris',
      'fenodes' = '127.0.0.1:8030',
      'table.identifier' = 'database.table',
      'username' = 'root',
      'password' = 'password',
      'doris.filter.query' = 'age=18'
);

SELECT * FROM flink_doris_source;
```
```

**维表 Join**

```
```
CREATE TABLE fact_table (
  `id` BIGINT,
  `name` STRING,
  `city` STRING,
  `process_time` as proctime()
) WITH (
  'connector' = 'kafka',
  ...
);

create table dim_city(
  `city` STRING,
  `level` INT ,
  `province` STRING,
  `country` STRING
) WITH (
  'connector' = 'doris',
  'fenodes' = '127.0.0.1:8030',
  'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
  'lookup.jdbc.async' = 'true',
  'table.identifier' = 'dim.dim_city',
  'username' = 'root',
  'password' = ''
);

SELECT a.id, a.name, a.city, c.province, c.country,c.level 
FROM fact_table a
LEFT JOIN dim_city FOR SYSTEM_TIME AS OF a.process_time AS c
ON a.city = c.city
```
```

**写入 Doris**

```
CREATE TABLE doris_sink (
    name STRING,
    age INT,
    score DECIMAL(5,2)
    ) 
    WITH (
      'connector' = 'doris',
      'fenodes' = '127.0.0.1:8030',
      'table.identifier' = 'database.table',
      'username' = 'root',
      'password' = '',
      'sink.label-prefix' = 'doris_label',
      //json格式写入
      'sink.properties.format' = 'json',
      'sink.properties.read_json_by_line' = 'true'
);
```

致谢

在此向所有参与版本设计、开发、测试、讨论的社区贡献者们表示感谢，感谢他们为本次版本开发和优化提供了强有力支持。

**贡献者名单**

@caoliang-web

@DongLiang-0

@gnehil

@GoGoWen

@JNSimba

@legendtkl

@lsy3993

@Myasuka

@wolfboys

**- END-**

欢迎更多的开源技术爱好者加入 Apache Doris 社区交流群，携手成长，共建社区生态。**Apache Doris 社区当前已容纳了上万名开发者和使用者，承载了 30+ 交流社群**，如果你也是 Apache Doris 的爱好者，非常欢迎您的加入！