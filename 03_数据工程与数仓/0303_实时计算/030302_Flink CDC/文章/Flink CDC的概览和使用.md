---
title: Flink CDC的概览和使用
author: 语兴数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkwODYwNjExMA==&mid=2247483976&idx=1&sn=e0b27fcb81526918faa5a84a6136c5e0&chksm=c0c62db9f7b1a4af30aea32b77eaa8bedbfa628124a5befe131567e25cc7b1a6beb14548d8ed&mpshare=1&scene=24&srcid=1222Fpc39ui5ofN34R6Bhocx&sharer_shareinfo=9fc41d9ec280d422e3b2ce0c01bace20&sharer_shareinfo_first=9fc41d9ec280d422e3b2ce0c01bace20#rd
---

## 什么是CDC

CDC（Change Data Capture）是一种用于跟踪数据库中数据更改的技术。它用于监视数据库中的变化，并捕获这些变化，以便实时或定期将变化的数据同步到其他系统、数据仓库或分析平台。CDC 技术通常用于数据复制、数据仓库更新、实时报告和数据同步等场景。

CDC 可以捕获数据库中的以下类型的数据变化：

1. 插入（Insert）：当新数据被插入到数据库表中时。
2. 更新（Update）：当数据库表中的现有数据被修改时。
3. 删除（Delete）：当数据从数据库表中被删除时

## 什么是Flink CDC

Flink CDC（Change Data Capture，即数据变更抓取）是一个开源的数据库变更日志捕获和处理框架，它可以实时地从各种数据库（如MySQL、PostgreSQL、Oracle、MongoDB等）中捕获数据变更并将其转换为流式数据。Flink CDC 可以帮助实时应用程序实时地处理和分析这些流数据，从而实现数据同步、数据管道、实时分析和实时应用等功能。

本质上是一系列的Flink Source Connector集合，用于来获取数据库的实时变更，底层基于Debezium实现。

https://github.com/ververica/flink-cdc-connectors

## Flink CDC 前生今世

### Flink CDC 1.x

Flink CDC 1.x开启了Flink在CDC上的实践之路，Flink CDC 1.x第一次引入了Debezium框架，利用Debezium已有的能力将数据库实时变更接入到Flink流计算框架中，利用Flink丰富的生态对数据进行加工处理，满足不同的业务需求，在功能层面上而言，Flink CDC 1.x只能说是可以用，但不能生产上用，为什么：

1. 1.x版本全增量切换时会对表加锁，在同步过程中有段时间业务会处于暂停状态
2. 各方面功能还不够完善，比如自动加表、DDL事件传递等

总体而言Flink CDC 1.x只能说是一个比较有趣的小玩具，还不具备大规模商业盈利的价值。

### Flink CDC 2.x

在2.x版本中，Flink CDC引入了Netfix DBLog中的无锁算法，彻底解决了全增量切换上业务停滞的问题，同时得益于FLIP-27对Flink Source API的重构，Flink CDC也基于FLIP-27升级到了新的框架设计，至此，Flink CDC被大规模公司使用并投入到生产中。

Flink CDC 2.0 正式发布，详解核心改进-阿里云开发者社区

### Flink CDC 3.x

近期，Flink CDC发布了全新的3.0版本，并宣布捐赠回Flink主项目，在新的3.0版本中，Flink CDC对于接口和架构上做了很大的升级和调整，对于整体项目的定位也从之前的Flink Source Connector转变为了Data Integration Engine，未来将与SeaTunnel、DataX、Chunjun等一系列老牌数据集成项目同台竞技，让我们拭目以待。

流计算迎来代际变革：流式湖仓 Flink + Paimon 加速落地、Flink CDC 重磅升级\_大数据\_Apache Flink\_InfoQ写作社区

## Flink CDC使用

1. 在本地启动一个MySQL的Docker环境

```
docker run -it --rm --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=debezium -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpw -e TZ=Asia/Shanghai quay.io/debezium/example-mysql:2.4
```

1. 创建表

```
create database cdc_test;  
use cdc_test;  
  
create table cdc_table (  
    id int primary key auto_increment,  
    name varchar(1000),  
    age int  
);
```

1. 在idea中新建一个Java项目
2. 导入依赖：

```
<flink-cdc.version>2.4.2</flink-cdc.version>  
<flink.version>1.16.3</flink.version>  
<logback.version>1.2.7</logback.version>  
  
<dependency>  
    <groupId>com.ververica</groupId>  
    <artifactId>flink-connector-mysql-cdc</artifactId>  
    <version>${flink-cdc.version}</version>  
</dependency>  
  
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-connector-base</artifactId>  
    <version>${flink.version}</version>  
</dependency>  
  
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-clients</artifactId>  
    <version>${flink.version}</version>  
</dependency>  
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-table-runtime</artifactId>  
    <version>${flink.version}</version>  
</dependency>  
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-runtime-web</artifactId>  
    <version>${flink.version}</version>  
</dependency>  
  
<dependency>  
    <groupId>ch.qos.logback</groupId>  
    <artifactId>logback-classic</artifactId>  
    <version>${logback.version}</version>  
</dependency>
```

1. 编写代码

```
public class FlinkCDCApplication {  
    public static void main(String[] args) throws Exception {  
  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
  
        env.setParallelism(1);  
        env.enableCheckpointing(60000L);  
  
        MySqlSource<String> mySqlSource = MySqlSource.<String>builder()  
            .hostname("localhost")  
            .port(3306)  
            .databaseList("cdc_test") // set captured database, If you need to synchronize the whole database, Please set tableList to ".*".  
            .tableList("cdc_test.cdc_table") // set captured table  
            .username("root")  
            .password("debezium")  
            .includeSchemaChanges(true)  
            .startupOptions(StartupOptions.latest())  
            .deserializer(new JsonDebeziumDeserializationSchema()) // converts SourceRecord to JSON String  
            .build();  
  
        env.fromSource(mySqlSource, WatermarkStrategy.noWatermarks(), "MySQL-CDC")  
            .print();  
        env.execute();  
    }  
}
```

1. 添加日志配置

```
<!--  
  ~ Licensed to the Apache Software Foundation (ASF) under one or more  
  ~ contributor license agreements.  See the NOTICE file distributed with  
  ~ this work for additional information regarding copyright ownership.  
  ~ The ASF licenses this file to You under the Apache License, Version 2.0  
  ~ (the "License"); you may not use this file except in compliance with  
  ~ the License.  You may obtain a copy of the License at  
  ~  
  ~    http://www.apache.org/licenses/LICENSE-2.0  
  ~  
  ~ Unless required by applicable law or agreed to in writing, software  
  ~ distributed under the License is distributed on an "AS IS" BASIS,  
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
  ~ See the License for the specific language governing permissions and  
  ~ limitations under the License.  
  -->  
  
<configuration>  
       <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">  
          <encoder>  
             <pattern>%d{yyyy-MM-dd HH:mm:ss} %p %c - %msg %n</pattern>  
          </encoder>  
       </appender>  
  
       <root level="INFO">  
          <appender-ref ref="STDOUT" />  
       </root>  
</configuration>
```

## Debezium标准CDC Event格式详解

```
{  
    "before": null,  
    "after": {  
        "id": 1,  
        "name": "xing.yu",  
        "age": 26,  
        "new_column": "dewu"  
    },  
    "source": {  
        "version": "1.9.7.Final",  
        "connector": "mysql",  
        "name": "mysql_binlog_source",  
        "ts_ms": 1702723640000,  
        "snapshot": "false",  
        "db": "cdc_test",  
        "sequence": null,  
        "table": "cdc_table",  
        "server_id": 223344,  
        "gtid": null,  
        "file": "mysql-bin.000003",  
        "pos": 2394,  
        "row": 0,  
        "thread": 39,  
        "query": null  
    },  
    "op": "c",  
    "ts_ms": 1702723640483,  
    "transaction": null  
}  
{  
    // 表数据更新前的值，update/delete  
    "before": {},  
    // 表数据更新后的值，create/update  
    "after": {},  
    // 元数据信息  
    "source": {},  
    // 操作类型 c/d/u  
    "op": "",  
    // 记录解析时间  
    "ts_ms": "",  
    "transaction": ""  
}
```

## Flink CDC生产case

https://ververica.github.io/flink-cdc-connectors/master/content/%E5%BF%AB%E9%80%9F%E4%B8%8A%E6%89%8B/mysql-postgres-tutorial-zh.html