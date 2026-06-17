---
title: Dinky 扩展 iceberg 的实践分享
author: Dinky开源
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg3ODYxOTQxMA==&mid=2247485727&idx=1&sn=90c55422254afe2cb657ffd178579c5b&chksm=cf11b762f8663e744dd4901603d53fb2e8d91074f372e6eeb728ac6e05fc0abda18313a9f8a5&mpshare=1&scene=24&srcid=0523vNeCG1wXExJHMRlzYMXg&sharer_sharetime=1653268730125&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**摘要：**本文介绍了 Dinky 实时计算平台扩展 iceberg 的实践分享。内容包括：

1. 背景
2. 部署
3. 案例
4. 总结

**Tips：**历史传送门～

《[Dinky 扩展 kudu 实践分享](http://mp.weixin.qq.com/s?__biz=Mzg3ODYxOTQxMA==&mid=2247485714&idx=1&sn=ceb598387c6c68a728776a25bc342569&chksm=cf11b76ff8663e794174589d015d391523eaae067fcb5d3f3eb62abe89a8b033ffb197d16b58&scene=21#wechat_redirect)》

《[Dinky 构建 Flink CDC 整库入仓入湖](http://mp.weixin.qq.com/s?__biz=Mzg3ODYxOTQxMA==&mid=2247485697&idx=1&sn=bee980c1f35ddfb04be6c4831afa29bc&chksm=cf11b77cf8663e6a4a51188c4475a725a16696b9b902957cab094affa16d5bc9efb29d892031&scene=21#wechat_redirect)》

《[Dinky 0.6.1 已发布，优化 Flink 应用体验](http://mp.weixin.qq.com/s?__biz=Mzg3ODYxOTQxMA==&mid=2247485661&idx=1&sn=e400ae3d8a718d40785eab9fa7875993&chksm=cf11b6a0f8663fb65b49b1beeef4337c030aeb5517df264e24ce7cb71815f466af456f3071a1&scene=21#wechat_redirect)》

《[Dlink 在 FinkCDC 流式入湖 Hudi 的实践分享](http://mp.weixin.qq.com/s?__biz=Mzg3ODYxOTQxMA==&mid=2247485540&idx=1&sn=5ac50f8350ef8abd6e50dc433d88d83b&chksm=cf11b619f8663f0fe22f329416dcf9b9ab11a3512062f7028bde0c23d201462fc838ab8ed40c&scene=21#wechat_redirect)》

GitHub 地址

https://github.com/DataLinkDC/dlink

https://gitee.com/DataLinkDC/Dinky

欢迎大家关注 Dinky 的发展~

**一、背景**

Iceberg 是一个面向海量数据分析场景的开放表格式 (Table Format)。定义中所说的表格式 (Table Format)，可以理解为元数据以及数据文件的一种组织方式， 处于计算框架 (Flink, Spark...) 之下，数据文件之上。

Iceberg 数据湖是一个集中式存储库，可存储任意规模结构化和非结构化数据，支持大数据和 AI 计算。数据湖构建服务（Data Lake Formation, DLF）作为云原生数据湖架构核心组成部分，帮助用户简单快速地构建云原生数据湖解决方案。数据湖构建提供湖上元数据统一管理、企业级权限控制，并无缝对接多种计算引擎，打破数据孤岛， 洞察业务价值，面向海量数据处理，历史数据状态更新，查询响应快。

本文将带来基于 Dinky 来实现 Flink 流式入湖 iceberg 的实践分享。

**二、部署**

下载 Iceberg 相关依赖 Jar 包放入 Flink/lib 文件里 (注: dlink/plugins 目录里面也要有，HDFS 上 flink/lib 目录下面也要有，因为 Dinky 依赖于这些 jar 包构建数据湖)

* iceberg-flink-runtime-1.12-0.13.1
* iceberg-hive-runtime-0.13.1  用于集成hive数仓（iceberg (hive\_catalog) 与hive元数据互通）
* iceberg-mr-0.13.1 用于集成hive数仓 (iceberg (hadoop\_catalog) 与hive元数据互通)

**三、案例**

本案例为 FlinkCDC -> Kafka -> iceberg。

### **1.依赖准备**

需要相关依赖jar包，数据打通 Mysql，Kafka，Hive

https://repo.maven.apache.org/maven2/ 根据自己相应组件版本下载

* flink-sql-connector-mysql-cdc-2.2.1.jar 连接 Mysql，创建 flinkcdcmysql 表不报错
* flink-format-changelog-json-2.1.1.jar 用于 binlog 数据状态更新(增删改)
* flink-shaded-hadoop-3-uber-3.1.1.7.2.9.0-173-9.0.jar 用于 kafka 写入数据到 iceberg，如果没有这 jar 包数据写入不了，因为iceberg 元数据/数据是存储在 hdfs 上
* flink-sql-connector-hive-3.1.2\_2.12-1.13.6.jar 用于 flink 打通 hive
* flink-sql-connector-kafka\_2.12-1.13.5.jar ,kafka-clients-3.1.0.jar,kafka\_2.12-3.1.0.jar  用于 flink 打通 kafka

**2.创建 FlinkCDC\_Kafka\_Env FlinkSQLEnv**

在 Dinky 上创建 FlinkCDC\_Kafka\_Env 环境文件（FlinkSqlEnv文件）

```
USE CATALOG default_catalog;  
  
use default_database;  
  
CREATE TABLE test_table(  
BILLID varchar,  
CARDID varchar,  
PRIMARY KEY (BILLID) NOT ENFORCED  
) WITH (  
    'connector' = 'mysql-cdc',/*连接类型*/  
    'hostname' = '192.168.50.20',/*地址*/  
    'port' = '3307',/*端口*/  
    'username' = 'root',/*用户*/  
    'password' = 123456,/*密码*/  
    'database-name' = 'test',/*库名*/  
    'table-name' = 'test_table',/*表名*/  
    'connect.timeout' = '60s', /*连接超时时间*/  
    'debezium.skipped.operations'='d', /*跳过binglog删除操作*/  
    'server-id'='5401-5416',/*服务器 id*/  
    'server-time-zone' = 'Asia/Shanghai', /*时区调整*/  
    'scan.startup.mode'='initial',/*initial全量，latest-offset最新binglog读取及更新*/  
    'scan.incremental.snapshot.enabled'='true'/*增量快照*/  
);  
  
CREATE TABLE test_kfk(  
BILLID varchar,  
CARDID varchar,  
PRIMARY KEY (BILLID) NOT ENFORCED  
) WITH (  
      'connector' = 'kafka' /*kafka连接类型*/  
    , 'topic' = 'test_table/*topic名称*/  
    , 'scan.startup.mode' = 'earliest-offset'/*消费策略*/  
    , 'properties.bootstrap.servers' = '192.168.50.20:9092,192.168.50.21:9092,192.168.50.22:9092'/*连接地址*/  
    , 'properties.group.id' = 'source'/*消费者组*/  
    , 'value.format' = 'changelog-json'/*数据json格式解析*/  
    , 'sink.parallelism'='1'/*并行度设置*/  
);
```

**3.指定 Mysql 表和 Kafka 表**

**4.创建 FlinkCDC\_Kafka\_Sql 作业**

在 Dinky 上创建 FlinkCDC\_Kafka\_Sql 文件（ FlinkSQL 类型文件）

```
set  jobmanager.memory.process.size= 1024m;  
set  taskmanager.memory.process.size= 2048m;  
  
set execution.checkpointing.interval = 60s;  
set execution.checkpointing.timeout= 15000000;  
set execution.checkpointing.max-concurrent-checkpoints= 500;  
set execution.checkpointing.min-pause= 500;  
  
-- 开启状态后端类型为rocksdb，开启增量快照，开启checkpoints，记录数据状态，如果不开启checkpoints接下来查询kafka数据是查不到的  
set state.backend = rocksdb;  
set state.backend.incremental=true;  
set state.backend.rocksdb.metrics.block-cache-usage=true;  
set state.backend.rocksdb.block.cache-size= 128mb;  
set state.backend.rocksdb.block.blocksize= 64kb;  
set taskmanager.numberOfTaskSlots= 3;  
set table.exec.resource.default-parallelism = 3;  
  
set table.exec.iceberg.infer-source-parallelism=true;  
set table.exec.iceberg.infer-source-parallelism.max=3;  
  
-- Mysql数据插入到kafka  
insert into default_catalog.default_database.test_kafka select * from default_catalog.default_database.test_table
```

然后运行，去 Flink 页面看任务，看 jobmanager 日志，Mysql 先是切割数据成块，之前为什么要选定状态后端类型为 rocksdb，如果mysql 是一个亿数据，数据量很大，数据在切块的时候会报错在 rocksdb (磁盘里)，而不是内存里(jm)，查看 kafka 数据，数据是否进入，然后去创建 FlinkSQL 文件去查 kafka 数据，验证数据状态是否更新(增，删( mysqlcdc 参数里面有跳过删除 binlog 日志)，改)

**5.创建 Kafka\_Iceberg\_Env FlinkSQLEnv**

基于 Hadoop\_catalog 模式

```
USE CATALOG default_catalog;  
  
use default_database;  
  
-- 创建kafka表  
CREATE TABLE test_kafka(  
BILLID varchar,  
CARDID varchar  
PRIMARY KEY (BILLID) NOT ENFORCED  
) WITH (  
      'connector' = 'kafka' /*kafka连接类型*/  
    , 'topic' = 'test_table/*topic名称*/  
    , 'scan.startup.mode' = 'earliest-offset'/*消费策略*/  
    , 'properties.bootstrap.servers' = '192.168.50.20:9092,192.168.50.21:9092,192.168.50.22:9092'/*连接地址*/  
    , 'properties.group.id' = '2wbsink'/*消费者组*/  
    , 'value.format' = 'changelog-json'/*数据json格式解析*/  
);  
  
/*创建目录在Hadoop上构建数据湖*/  
CREATE CATALOG hadoop_catalog WITH (  
  'type'='iceberg',/*类型是iceberg*/  
  'catalog-type'='hadoop',/*目录存储类型在hadoop*/  
  'property-version'='1',/*版本*/  
  'warehouse'='hdfs://ns/warehouse/hadoop_catalog'/*目录创建地址*/  
);  
--查看hadoop上是否有hadoop_Catalog目录  
use catalog hadoop_catalog;  
--在hadoop_Catalog下创建数据库  
Create database ods_iceberg_yarn;  --创建完看hadoop上是否有目录，然后注释掉语句，要不然报数据库已存在  
use ods_iceberg_yarn;  
  
--创建iceberg表在hadoop_catalog.ods_iceberg_yarn，创建完注释掉表  
-- CREATE TABLE iceberg_table(  
-- BILLID varchar,  
-- CARDID varchar,  
-- PRIMARY KEY (BILLID) NOT ENFORCED  
-- ) WITH   
-- (   
-- 'write.format.default'='orc',  --数据压缩格式orc  
-- 'write.upsert.enabled' = 'true', --支持数据更新  
-- 'engine.hive.enabled'='true', --与hive元数据互通  
-- 'format-version'='2'/*可以更新删除数据*/  
-- );  
  
  
--分区表结构，分区字段必须包含在主键里面,分区字段字段长度，类型字符串类型，数据量上亿数据，数据入湖会出现资源溢出问题  
-- CREATE TABLE iceberg_table(  
-- BILLID varchar,  
-- CARDID varchar,  
-- PRIMARY KEY (BILLID,USERID) NOT ENFORCED  
-- ) PARTITIONED BY (  
--  USERID  
--)WITH   
-- (   
-- 'write.format.default'='orc',  --数据压缩格式orc  
-- 'write.upsert.enabled' = 'true', --支持数据更新  
-- 'engine.hive.enabled'='true', --与hive元数据互通  
-- 'format-version'='2'/*可以更新删除数据*/  
-- );
```

基于 Hive\_catalog 模式

```
USE CATALOG default_catalog;  
  
use default_database;  
  
CREATE TABLE test_kafka(  
BILLID varchar,  
CARDID varchar  
PRIMARY KEY (BILLID) NOT ENFORCED  
) WITH (  
      'connector' = 'kafka' /*kafka连接类型*/  
    , 'topic' = 'test_table/*topic名称*/  
    , 'scan.startup.mode' = 'earliest-offset'/*消费策略*/  
    , 'properties.bootstrap.servers' = '192.168.50.20:9092,192.168.50.21:9092,192.168.50.22:9092'/*连接地址*/  
    , 'properties.group.id' = '2wbsink'/*消费者组*/  
    , 'value.format' = 'changelog-json'/*数据json格式解析*/  
);  
  
CREATE CATALOG hive_catalog WITH (  
  'type'='iceberg',  
  'catalog-type'='hive',  
  'uri'='thrift://192.168.50.20:9083',  
  'clients'='5',  
  'property-version'='2',  
  'warehouse'='hdfs://ns/user/hive/warehouse'  
);  
  
use catalog hive_catalog;  
  
-- create database ods;  
  
use  ods;  
  
-- CREATE TABLE iceberg(  
-- BILLID varchar ,  
-- CARDID varchar  
-- PRIMARY KEY (BILLID) NOT ENFORCED  
-- ) WITH   
-- (   
-- 'write.format.default'='orc',  --数据压缩格式orc  
-- 'write.upsert.enabled' = 'true', --支持数据更新  
-- 'engine.hive.enabled'='true', --与hive元数据互通  
-- 'format-version'='2'/*可以更新删除数据*/  
-- );
```

**5.创建 Kafka\_Iceberg\_SQL 作业**

在Dlink上创建Kafka\_Iceberg\_SQL文件

```
set  jobmanager.memory.process.size= 2048m;  
set  taskmanager.memory.process.size= 2048m;  
  
set execution.checkpointing.interval = 60s;  
set execution.checkpointing.timeout= 15000000;  
set execution.checkpointing.max-concurrent-checkpoints= 500;  
set execution.checkpointing.min-pause= 500;  
-- 开启状态后端类型为rocksdb，开启增量快照，开启checkpoints，记录数据状态，如果不开启checkpoints接下来查询kafka数据是查不到的  
set state.backend = rocksdb;  
set state.backend.incremental=true;  
set state.backend.rocksdb.metrics.block-cache-usage=true;  
set state.backend.rocksdb.block.cache-size= 128mb;  
set state.backend.rocksdb.block.blocksize= 64kb;  
set taskmanager.numberOfTaskSlots= 3;  
set table.exec.resource.default-parallelism = 3;  
  
set table.exec.iceberg.infer-source-parallelism=true;  
set table.exec.iceberg.infer-source-parallelism.max=3;  
-- 开启flink动态参数  
SET table.dynamic-table-options.enabled=true;  
  
/*+OPTIONS('equality-field-columns'='BILLID')*/  这里是根据主键id更新数据  
insert into hive_catalog.ods.ods_iceberg_salebilltable2bw5 /*+OPTIONS('equality-field-columns'='BILLID')*/ select * from default_catalog.default_database.salebilltable1q_kfk_sink
```

Hadoop\_catalog 和 Hive\_catalog 注意 iceberg 库，目录指定hadoop\_catalog.database.iceberg\_table,hive\_catalog.database.iceberg\_table

运行任务，查看hadoop目录下hadoop\_catalog下iceberg表，data目录生成就代表数据入湖了，只用dbeaver查看hive表数据是否落仓。

**6.基于iceberg Hadoop\_catalog 模式映射 Hive**

https://wenku.baidu.com/view/53e456eba2c7aa00b52acfc789eb172ded6399ee.html

(1) .查询iceberg数据

```
set  jobmanager.memory.process.size= 1024m;  
set  taskmanager.memory.process.size= 1024m;  
  
set execution.checkpointing.interval = 3s;  
set execution.checkpointing.timeout= 15000000;  
set execution.checkpointing.max-concurrent-checkpoints= 500;  
set execution.checkpointing.min-pause= 500;  
  
set state.backend = rocksdb;  
set state.backend.incremental=true;  
set state.backend.rocksdb.metrics.block-cache-usage=true;  
set state.backend.rocksdb.block.cache-size= 128mb;  
set state.backend.rocksdb.block.blocksize= 64kb;  
set taskmanager.numberOfTaskSlots= 1;  
set table.exec.resource.default-parallelism = 2;  
  
/*创建目录在Hadoop上构建数据湖*/  
CREATE CATALOG hadoop_catalog WITH (  
  'type'='iceberg',/*类型是iceberg*/  
  'catalog-type'='hadoop',/*目录存储类型在hadoop*/  
  'property-version'='1',/*版本*/  
  'iceberg.engine.hive.enabled'='true',  
  'warehouse'='hdfs://ns/warehouse/hadoop_catalog'/*目录创建地址*/  
);  
  
use catalog hadoop_catalog;  
  
-- CREATE CATALOG hive_catalog WITH (  
--   'type'='iceberg',  
--   'catalog-type'='hive',  
--   'uri'='thrift://192.168.88.50:9083',  
--   'clients'='5',  
--   'property-version'='2',  
--   'warehouse'='hdfs://ns/user/hive/warehouse/hive_catalog'  
-- );  
  
-- use catalog hive_catalog;  
  
use ods_iceberg_yarn;  
  
select BILLID,CASH,C_BANK from ods_iceberg_yarn.ods_iceberg_salebilltable2bw4 where BILLID='A110801000032'  
  
-- select billid,c_bank,cash from ods_iceberg_yarn.ods_iceberg_salebilltable2bw4 where billid='A110801000027'
```

**四、总结**

本文章是 Dinky 集成 iceberg 打通案例，但我觉得还是多去 iceberg 官网看看原理，创表参数，读写参数，分区兼容 Flink 问题，之后 iceberg 新版本隐藏分区会支持 flink。

对于使用 Dinky 的感受，我觉得最好的还是省了写 flinkapi 代码了，方便管理任务，不同模式的提交，也不用去命令行写命令提交，还有checkpoint任务恢复，功能挺强大的。

**交流**

欢迎您加入社区交流分享与批评，也欢迎您为社区贡献自己的力量。

QQ社区群：**543709668**，申请备注 “ Dinky+企业名+职位 ”，不写不批

微信官方群（推荐）：添加 wenmo\_ai ，申请备注“ Dinky+企业名+职位”，缺一不批谢谢。

公众号：DataLink数据中台

扫描二维码获取

更多精彩

DataLink

数据中台