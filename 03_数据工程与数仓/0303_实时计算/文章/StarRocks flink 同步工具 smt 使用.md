---
title: StarRocks flink 同步工具 smt 使用
author: Flink菜鸟
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI3MjAxNDYwOA==&mid=2247484945&idx=1&sn=6fe64c1f6602f1614e18898b0eeadff3&chksm=eb384a6edc4fc378f423373cc1ef3ac5cc1085987f987c295a8a290413481389b12d124c255f&mpshare=1&scene=24&srcid=0922IZVQmFHSq23i3tKSCHg7&sharer_sharetime=1663823641697&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

* 注：本文主要内容来自于官网 MySQL 实时同步至StarRocks

## 功能简介

StarRocks 提供 Flink CDC connector、flink-connector-starrocks 和 StarRocks-migrate-tools（简称smt），实现 MySQL 数据实时同步至 StarRocks，满足业务实时场景的数据分析。

* smt 实际上是个读 mysql 生成 flink cdc 脚本、starrocks 表、starrocks mysql 外表的工具

## 基本原理

通过 Flink CDC connector、flink-connector-starrocks 和 smt 可以实现 MySQL 数据的秒级同步至StarRocks。

76.png

如图所示，Smt 可以根据 MySQL 和 StarRocks 的集群信息和表结构自动生成 source table 和 sink table 的建表语句。  
通过 Flink-cdc-connector 消费 MySQL 的 binlog，然后通过 Flink-connector-starrocks 写入 StarRocks。

## 操作步骤

* 忽略 mysql binlog 和 flink 相关配置

1. 下载并解压 smt.tar.gz https://www.starrocks.com/zh-CN/download/community

```
```
venn@venn smt % lsREADME.md conf result starrocks-migrate-tool
```
```

修改 conf/config\_prod.conf

```
venn@venn smt % cat conf/config_prod.conf [db]host = ip   # mysql 配置port = 3306user = userpassword = pass# currently available types: `mysql`, `pgsql`, `oracle`, `hive`, `clickhouse`, `sqlserver`, `tidb`type = mysql# # only takes effect on `type == hive`. # # Available values: kerberos, none, nosasl, kerberos_http, none_http, zk, ldap# authentication = kerberos  
[other]# number of backends in StarRocksbe_num = 3# `decimal_v3` is supported since StarRocks-1.8.1use_decimal_v3 = true# directory to save the converted DDL SQLoutput_dir = ./result   # 数据文件路径  
  
# !!!`database` `table` `schema` are case sensitive in `oracle`!!![table-rule.1]# pattern to match databases for setting properties# !!! database should be a `whole instance(or pdb) name` but not a regex when it comes with an `oracle db` !!!database = db_name  # 数据库名字，完整名字# pattern to match tables for setting propertiestable = ^table$     # 表名正则# `schema` only takes effect on `postgresql` and `oracle` and `sqlserver`schema = ^public$  
############################################### starrocks table configurations############################################# # set a column as the partition_keypartition_key = id   # 指定 starrocks 表的主键# # override the auto-generated partitions# partitions = START ("2021-01-02") END ("2021-01-04") EVERY (INTERVAL 1 day)# # only take effect on tables without primary keys or unique indexesduplicate_keys=id   # 指定 starrocks 表的重复键# # override the auto-generated distributed keysdistributed_by=id  # 指定 starrocks 表的分桶键# # override the auto-generated distributed bucketsbucket_num=8    # 桶数量# # properties.xxxxx: properties used to create tablesproperties.in_memory = false     
############################################### flink sink configurations### DO NOT set `connector`, `table-name`, `database-name`, they are auto-generated############################################   flink-connector-starrocks sink 表配置flink.starrocks.jdbc-url=jdbc:mysql://ip:19030flink.starrocks.load-url=ip:18030flink.starrocks.username=rootflink.starrocks.password=123456flink.starrocks.sink.max-retries=10flink.starrocks.sink.buffer-flush.interval-ms=1500flink.starrocks.sink.properties.format=jsonflink.starrocks.sink.properties.strip_outer_array=false# # used to set the server-id for mysql-cdc jobs instead of using a random server-id# flink.cdc.server-id = 5000  
############################################### flink-cdc configuration for `tidb`############################################# # Only takes effect on TiDB before v4.0.0. # # TiKV cluster's PD address.# flink.cdc.pd-addresses = 127.0.0.1:2379  
############################################### flink-cdc plugin configuration for `postgresql`############################################# # for `9.*` decoderbufs, wal2json, wal2json_rds, wal2json_streaming, wal2json_rds_streaming# # refer to https://ververica.github.io/flink-cdc-connectors/master/content/connectors/postgres-cdc.html # # and https://debezium.io/documentation/reference/postgres-plugins.html# flink.cdc.decoding.plugin.name = decoderbufs%
```

执行

```
[starrocks@dcmp12 smt]$ ./starrocks-migrate-tool   
2022/08/15 19:36:46 source/mysql.go:108 SLOW SQL >= 200ms[3785.337ms] [rows:2294] SELECT * FROM `information_schema`.`tables` WHERE TABLE_TYPE='BASE TABLE' ORDER BY TABLE_SCHEMA asc, TABLE_NAME asc  
2022/08/15 19:36:48 source/mysql.go:116 SLOW SQL >= 200ms[1853.744ms] [rows:35344] SELECT * FROM `information_schema`.`columns` ORDER BY ORDINAL_POSITION asc  
2022/08/15 19:36:48 source/mysql.go:124 SLOW SQL >= 200ms[216.231ms] [rows:2763] SELECT * FROM `information_schema`.`key_column_usage`Successfully got tables from the source database. Converting them to StarRocks DDL...Writing starrocks ddl reults...Done writing to: /opt/smt/resultWriting starrocks ddl reults...Done writing to: /opt/smt/resultWriting starrocks ddl reults...Done writing to: /opt/smt/result[starrocks@dcmp12 smt]$ ls conf/                   README.md               result/                 starrocks-migrate-tool  [starrocks@dcmp12 smt]$ ls result/flink-create.1.sql  flink-create.all.sql  starrocks-create.1.sql  starrocks-create.all.sql  starrocks-external-create.1.sql  starrocks-external-create.all.sql
```

## 生成 sql

### flink建表语句和insert

```
CREATE TABLE IF NOT EXISTS `default_catalog`.`test_db`.`test_table_src` (  `id` STRING NULL,  `memcardno` STRING NULL,  .....  
) with (  'connector' = 'mysql-cdc',  'hostname' = 'ip',  'port' = '3306',  'username' = 'root',  'password' = '123456',  'database-name' = 'db_name',  'table-name' = 'table_name');  
CREATE TABLE IF NOT EXISTS `default_catalog`.`test_db`.`test_table_sink` (  `id` STRING NULL,  `memcardno` STRING NULL,  ....  
) with (  'sink.buffer-flush.interval-ms' = '1500',  'username' = 'root',  'load-url' = 'xx.xx.xx.228:18030;xx.xx.xx.229:18030;xx.xx.xx.230:18030',  'table-name' = 'test_table',  'connector' = 'starrocks',  'database-name' = 'test_db',  'sink.properties.strip_outer_array' = 'false',  'sink.max-retries' = '10',  'jdbc-url' = 'jdbc:mysql://xx.xx.xx.228:19030,xx.xx.xx.229:19030,xx.xx.xx.230:19030',  'password' = '123456',  'sink.properties.format' = 'json');  
INSERT INTO `default_catalog`.`test_db`.`test_table_sink` SELECT * FROM `default_catalog`.`test_db`.`test_table_src`;
```

## starrocks 表建表语句

```
CREATE TABLE IF NOT EXISTS `test_db`.`test_table` (  `id` STRING NULL  COMMENT "自增主键",  `memcardno` STRING NULL  COMMENT "会员ID",    .....  
) ENGINE=olapDUPLICATE KEY(id)COMMENT "test_table"DISTRIBUTED BY HASH(id) BUCKETS 8PROPERTIES (  "in_memory" = "false",  "replication_num" = "3");
```

## starrocks mysql 外表建表语句

```
CREATE EXTERNAL TABLE `mysql_external_test_db`.`test_table` (  `id` STRING NULL  COMMENT "自增主键",  `memcardno` STRING NULL  COMMENT "会员ID",    .....) ENGINE=mysqlCOMMENT "test_table.xlsx"PROPERTIES (  "host" = "xx.xx.xx.xx",  "port" = "3306",  "user" = "root",  "password" = "123456",  "database" = "test_db",  "table" = "test_table");
```

## 已知局限

1. smt 只能生成明细表
2. 不能识别表的主键或者唯一键，只能指定固定字段名，对懒人来说，勉强凑合用