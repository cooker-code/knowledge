---
title: Doris到底有多强？
author: 锋哥聊湖仓
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI4ODMyNTcwMw==&mid=2247486157&idx=1&sn=2765de5ba9517631123b5f7832a262d5&chksm=ebc163e5dcb6eaf3ceb9202dc95b22f694eb7b91686fdd6a73e82f29a8740745526f302a2721&mpshare=1&scene=24&srcid=1220GlRZL7VdQ8AWVPicxzpU&sharer_shareinfo=35cc1b2cf50b033c4628beb8772eb6af&sharer_shareinfo_first=35cc1b2cf50b033c4628beb8772eb6af#rd
---

## 缘起

首先声明，本人无意叛变，依然是ClickHouse的忠实信徒。

对于Doris，一直听圈内的人在说，吹得神乎其神，但到底有多强，从来没有真正的去尝试一把。

直到这次，被人狠狠上了一课。

在一次全文检索的模糊查询的场景PK中，ClickHouse一败涂地，让本人很是没面子，咳咳，大哥被人欺负了，这能忍？

一直知道Doris在多表join方面很强，没想到在全文检索也能弯道超车，这实在是猝不及防啊。

啥也别说了，盘他！

咱就是说，知己知彼，方能百战不殆。就算要死，也得死得明白不是？何况咱的目的是接近它，了解它，成为它，并打败它。

## Doris到底是个啥

首先不得不说，Doris有完备的中文版文档，这对于我这种英语渣来说，简直太有诱惑力了。不得不说，在鲨人诛心方面，还是Doris玩得溜哇。

Doris的整体架构分为两大部分，FE和BE。

FE使用Java语言实现，自带前端页面(前端代码为vue3)。主要用来做数据接入，查询解析，元数据管理，节点管理等。

BE使用C++实现，主要用来存储和查询。

简单来说，它把职责划分得很清楚，所有的调度，交互都交给了FE，BE成了完完全全的工具人，FE让我存我就存，让我查我就查，查出来的数据怎么用，怎么返回，咱不关心。

打个比方，FE就是产品经理，负责出设计出需求，BE就是苦逼的打工码农，干活，背锅，一样不落。BE唯一的价值就是它的计算资源和存储资源。

除了FE和BE两大模块之外，Doris还提供了一个叫做Brokers的模块。这个组件对于Doris来说不是必须的。但是提供一定的扩展能力，如实现从HDFS、BOS、AFS导入数据等。它是一种异步的导入方式，因为这个组件是用Java写的，资源占用相对来说比较高，在离线数据大批量导入场景，甚至比Spark Load要消耗更多的资源。所以对于这个组件，我的建议是，有"SPA"上"SPA"，没"SPA"咱还是"Bro"。

## 编译

为了简化流程，我们使用官方提供的docker编译镜像。

下载镜像：

```
docker pull apache/doris:build-env-ldb-toolchain-latest
```

进入容器：

```
docker run --rm -w /var/src -v `pwd`/build/.m2:/root/.m2 -v `pwd`:/var/src -it apache/doris:build-env-ldb-toolchain-latest bash
```

上述命令中，我们将.m2目录从容器里挂载出来，并将本地的源码挂载到容器内部。这样做的好处是可以让编译的产物（包括临时产物）都留在本地，下次编译时无需重复下载依赖。（Java的maven依赖下载非常耗时）

直接执行 build.sh。

```
./build.sh
```

整个编译过程耗时接近一个半小时，相对来说还算可以接受的。

编译完成后，产物在output目录中：

友情提醒：对于没有二次开发需求的用户，建议直接下载官方提供的release版本使用即可。

## 集群部署

集群部署架构初步规划如下：

| 角色 | 节点 | ip |
| --- | --- | --- |
| FE | leader | 192.168.101.94 |
| BE | node1 | 192.168.101.93 |
| BE | node2 | 192.168.101.94 |
| BE | node3 | 192.168.101.96 |

Doris的官方安装包分为avx2和非avx2版本，根据自己的CPU支持情况下载即可。整个安装包2.8个G，请在网络环境比较好的时候下载。

解压后有三个文件夹：

fe文件夹用于fe部署，be文件夹用于be部署，extensions文件夹是一些扩展，如hdfs的brokers部署，可以先不用管。

## FE部署

进入fe文件夹，修改配置文件：

修改conf/fe.conf：

```
priority_networks = 192.168.101.0/24 # CIDR方式寻址，fe集群之间，fe 与be之间通讯的关键， 可以写多组  
  
# 数据目录  
meta_dir = ${DORIS_HOME}/doris-meta  
  
# 端口  
http_port = 58030    # 前端页面端口  
rpc_port = 59020  # thrift server 端口  
query_port = 59030   # 查询的端口， mysql client可以使用该端口连接上Doris  
edit_log_port = 59010  # FE集群之间通信的端口
```

端口需要看一下本地有没有被占用，尽量找没有占用的端口。由于我部署时默认的8030已被本地的hadoop应用占用，所以使用了58030， 如果默认端口没有被占用，使用默认端口即可。

启动应用：

```
bin/start_fe.sh --daemon
```

验证fe是否成功启动：

```
[root@master94 fe]# curl http://192.168.101.94:58030/api/bootstrap  
{"msg":"success","code":0,"data":{"replayedJournalId":0,"queryPort":0,"rpcPort":0,"version":""},"count":0}
```

或直接登录前端页面：

```
http://192.168.101.94:58030/login
```

显示如下页面，说明部署fe成功：

默认登录用户为 root， 密码为空。

登录进去后页面如图所示：

## BE部署

1. 修改配置文件：

修改 conf/be.conf:

```
priority_networks = 192.168.101.0/24 #需要和fe配置一致  
  
be_port = 59060 # 和FE通讯的端口，用来接收FE的请求  
webserver_port = 58040 # http端口  
heartbeat_service_port = 59050 # 心跳端口，用来接收FE的心跳  
brpc_port = 58060 # BE之间通讯  
  
# storage_root_path = ${DORIS_HOME}/storage  
storage_root_path = /data01/app/apache-doris-2.0.3-bin-x64/data/ssd,medium:SSD;/data01/app/apache-doris-2.0.3-bin-x64/data/hdd,medium:HDD  
# HDD 和 SSD 并不是真的固态盘和机械盘，只需要有相应的目录即可，SSD代表热数据目录，HDD代表冷数据目录，用于BE数据的冷热机制
```

1. 设置JAVA\_HOME：

修改bin/start\_be.sh，在脚本第一行增加:

```
export JAVA_HOME=/usr/java/jdk1.8.0_201
```

以上操作在所有需要部署BE的节点上都要操作一遍。

1. 将所有的BE节点加入到FE中：

从FE的页面操作：

进入Playground页面，执行如下SQL：

```
ALTER SYSTEM ADD BACKEND "192.168.101.93:59050";
```

其中， 192.168.101.93为BE的节点IP，有几个节点就需要增加几次；需要注意的是该IP需要与IP匹配。

59050为heartbeat\_service\_port， 根据配置文件填写即可。

如下图所示：

通过上面的方式，将192.168.101.93和192.168.101.94都添加进去。

从命令行操作：

我们可以使用mysql的client登陆到Doris的FE：

```
mysql -h 192.168.101.94 -P 59030 -uroot
```

192.168.101.94指的是FE的地址。

59030是FE配置的query\_port。

登录进去后，执行相同的SQL语句。

我们使用这种方式将192.168.101.96加进去：

```
[root@master94 be]# mysql -h 192.168.101.94 -P 59030 -uroot  
Welcome to the MySQL monitor.  Commands end with ; or \g.  
Your MySQL connection id is 2  
Server version: 5.7.99 Doris version doris-2.0.3-rc06-37d31a5  
  
Copyright (c) 2000, 2023, Oracle and/or its affiliates.  
  
Oracle is a registered trademark of Oracle Corporation and/or its  
affiliates. Other names may be trademarks of their respective  
owners.  
  
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
  
mysql> ALTER SYSTEM ADD BACKEND "192.168.101.96:59050";  
Query OK, 0 rows affected (0.01 sec)
```

1. 启动BE：

```
sysctl -w vm.max_map_count=2000000  
bin/start_be.sh --daemon
```

1. 验证BE是否正常启动

在FE的web页面执行SQL：SHOW PROC '/backends';

当看到Alive均为true， 集群即部署成功。

## 简单操作验证

1. 创建数据库：

```
create database demo;
```

1. 创建表：

```
use demo;  
  
CREATE TABLE IF NOT EXISTS demo.example_tbl  
(  
    `user_id` LARGEINT NOT NULL COMMENT "用户id",  
    `date` DATE NOT NULL COMMENT "数据灌入日期时间",  
    `city` VARCHAR(20) COMMENT "用户所在城市",  
    `age` SMALLINT COMMENT "用户年龄",  
    `sex` TINYINT COMMENT "用户性别",  
    `last_visit_date` DATETIME REPLACE DEFAULT "1970-01-01 00:00:00" COMMENT "用户最后一次访问时间",  
    `cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费",  
    `max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间",  
    `min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时间"  
)  
AGGREGATE KEY(`user_id`, `date`, `city`, `age`, `sex`)  
DISTRIBUTED BY HASH(`user_id`) BUCKETS 1  
PROPERTIES (  
    "replication_allocation" = "tag.location.default: 1"  
);
```

1. 准备数据：

```
10000,2017-10-01,北京,20,0,2017-10-01 06:00:00,20,10,10  
10000,2017-10-01,北京,20,0,2017-10-01 07:00:00,15,2,2  
10001,2017-10-01,北京,30,1,2017-10-01 17:05:45,2,22,22  
10002,2017-10-02,上海,20,1,2017-10-02 12:59:12,200,5,5  
10003,2017-10-02,广州,32,0,2017-10-02 11:20:00,30,11,11  
10004,2017-10-01,深圳,35,0,2017-10-01 10:00:15,100,3,3  
10004,2017-10-03,深圳,35,0,2017-10-03 10:20:22,11,6,6
```

将上面的数据保存在test.csv中。

1. 导入数据：

```
curl  --location-trusted -u root: -T test.csv -H "column_separator:," http://127.0.0.1:58030/api/demo/example_tbl/_stream_load
```

执行完结果如下：

```
[root@master94 apache-doris-2.0.3-bin-x64]# curl  --location-trusted -u root: -T test.csv -H "column_separator:," http://127.0.0.1:58030/api/demo/example_tbl/_stream_load  
{  
    "TxnId": 2,  
    "Label": "af5b1ad2-a86b-4dac-a1b8-ba91e950fc02",  
    "Comment": "",  
    "TwoPhaseCommit": "false",  
    "Status": "Success",  
    "Message": "OK",  
    "NumberTotalRows": 7,  
    "NumberLoadedRows": 7,  
    "NumberFilteredRows": 0,  
    "NumberUnselectedRows": 0,  
    "LoadBytes": 399,  
    "LoadTimeMs": 300,  
    "BeginTxnTimeMs": 25,  
    "StreamLoadPutTimeMs": 211,  
    "ReadDataTimeMs": 0,  
    "WriteDataTimeMs": 8,  
    "CommitAndPublishTimeMs": 53  
}
```

1. 查询数据

## 关于Doris集群相关说明

Doris集群里并没有集群名的概念。其中，FE集群和BE集群是互相独立，互不干扰的。

你可以这么理解：FE集群相当于CK中的zookeeper集群，BE集群才是真正的数据库集群。

与zookeeper类似，FE采用了类似Raft的选举机制，分为三类角色，分别为Leader， Follower， Oberserver。

一般可使用3 \* Follower + n \* Oberserver 的架构来保证FE集群的读写高可用。

Oberserver 是只读的扩展节点，可以通过水平扩展的方式来增加读的能力。

Doris集群没有分片和副本的概念。它有一点类似于kafka集群的机制，分片和副本是针对表维度的。也就是说，每张表，你可以设置不同的分片和副本数。

Doris的集群扩缩容非常简单，几乎不需要考虑配置的同步问题，以及元数据的同步的问题，仅需要一个命令就可以实现。

FE增加节点：

```
#扩容FE节点，可以将新节点添加为follower  
ALTER SYSTEM ADD FOLLOWER "fe_host:edit_log_port";  
  
#或新节点添加为observer  
ALTER SYSTEM ADD OBSERVER "fe_host:edit_log_port";
```

BE增加节点:

```
ALTER SYSTEM ADD BACKEND "be_host:heartbeat_service_port";
```

增加节点后，集群内部会自动进行数据的再均衡。

删除节点也很简单，只需要将上述SQL的ADD换成DECOMMISSION即可（FE使用DROP下线节点）。

由上面的操作可知，BE之间其实是互相不可见的，BE的集群统一通过FE来调度，因此不存在BE之间互相同步数据带来的资源消耗。

## Doris 常见概念介绍

## 数据模型

Doris的表也有Engine的概念，但是Doris的Engine和ClickHouse的表Engine不太一样。

Doris的Engine主要用来区分数据源的，比如OLAP、MySQL、ES、BROKER等。只有OLAP这个engine是由Doris自己负责数据的存储和管理。它也是默认的engine。

而真正和ClickHouse中表engine可以类比的概念则是数据模型。它决定了Doris在存储数据时，在内存的分布形式。

Doris提供了三类数据模型。

| 数据模型 |  |  |
| --- | --- | --- |
| Aggregate | 数据按key做聚合 | 聚合类型包括SUM、REPLACE、MAX、MIN、REPLACE\_IF\_NOT\_NULL、HLL\_UNION、BITMAP\_UNION等，支持agg\_state操作。会丢失明细数据。 |
| Unique | 数据按主键唯一 | 分merge\_on\_read和merge\_on\_write两种。merge\_on\_read可以看成是特殊的Aggregate模型，相当于按照key做replace，查询性能比较慢；merge\_on\_write是在插入时即完成了去重，查询性能高。 |
| Duplicate | 数据可重复 | 分为排序和不排序两类。数据可重复，不会丢失明细数据。类似clickhouse中的原生Mergetree。 |

## bucket、tablet、partition

Doris的表分为两层结构，分别为分区和分桶。

每个分桶文件就是一个数据分片（tablet）。tablet是数据划分的最小逻辑单元。各个tablet之间的数据在物理上独立，多个tablet组成一个partition，所以各partition之间的数据在物理上也是独立的。多个partition组成了一张表。

不同于ClickHouse，Doris没有本地表和分布式表的概念。它的查询是由FE调度，在各个BE节点上查询后进行汇总的，因此每次查询都是分布式查询。数据的分布由hash key保证。

```
DISTRIBUTED BY HASH(`user_id`) BUCKETS 16
```

partition可以按range分区和list分区。

range分区即是指按时间列进行分区，语法如下所示：

```
PARTITION BY RANGE(`date`)  
(  
    PARTITION `p201701` VALUES LESS THAN ("2017-02-01"),  
    PARTITION `p201702` VALUES LESS THAN ("2017-03-01"),  
    PARTITION `p201703` VALUES LESS THAN ("2017-04-01")  
)
```

如果导入的数据不在表的分区范围内，则是无法导入的。

上述这种静态分区是非常不灵活的，只适用于存量数据的性能测试场景。在不断有实时数据生成的场景，是不适合的，此时我们可以采用动态分区的方式：

```
PROPERTIES (  
  "dynamic_partition.enable" = "true",  
  "dynamic_partition.time_unit" = "DAY",  
  "dynamic_partition.start" = "-30",  
  "dynamic_partition.end" = "3",  
  "dynamic_partition.prefix" = "p",  
  "dynamic_partition.create_history_partition"="true",  
  "replication_num" = "1"  
);
```

如上参数说明如下：

* dynamic\_partition.enable：开启动态分区开关
* dynamic\_partition.time\_unit：时间单位，可以是hour、day、week、month、year
* dynamic\_partition.start：以当前时间为基准，往前保留多少个分区，多余的分区会被删除掉
* dynamic\_partition.end：以当前时间为基准，往后新创建多少个分区
* dynamic\_partition.prefix：动态分区的前缀
* dynamic\_partition.create\_history\_partition：是否创建历史分区

list分区，就是通过key进行分类了，相同的key的数据自动落在同一个分区内，语法如下：

```
PARTITION BY LIST(`city`)  
(  
    PARTITION `p_cn` VALUES IN ("Beijing", "Shanghai", "Hong Kong"),  
    PARTITION `p_usa` VALUES IN ("New York", "San Francisco"),  
    PARTITION `p_jp` VALUES IN ("Tokyo")  
)
```

Doris表的副本数指定是在建表时指定的，因此每张表可以有不同的副本数。

```
PROPERTIES  
    (  
        "replication_num" = "3"  
    );
```

如果不指定，默认副本数为3。由Doris自己保证数据在各个BE的均匀分布，如下图所示。

如果BE节点发生了扩容，数据也会进行重新均衡，当然均衡的基本单位是整个tablet移动。

因此，单个tablet不建议太大，否则会影响数据查询和副本迁移的性能。当然也不建议太小，否则聚合效果不佳，建议单个tablet在1-10G之间，理论上是可以没有上限的。

tablet的数量 = partition数量 \* bucket数量 \* 副本数。

partition内bucket的数量一旦指定就不能更改了，因此在指定bucket数量时，应充分考虑到集群后续扩容的情况。

## 索引介绍

Doris的索引分类两类。

一类是内建的索引，包括前缀索引和ZoneMap索引。内建索引不需要显式创建。

另一类是二级索引，包括倒排、布隆过滤器、bitmap索引等，从概念上来说，和ClickHouse的跳数索引类似。

### 前缀索引

ZoneMap索引是在列存格式上，自动维护每一列的索引信息，包括MinMax、Null值等，没有太多可以扩展讲解的东西。

重点来看前缀索引。

前缀索引是在排序的基础上，根据给定前缀列，快速查询数据的索引方式。

Doris默认会取表结构的前36个字节作为前缀索引列，特例是遇到VARCHAR自动截止。

这就要求我们在建表时，合理编排字段的顺序，有助于加速查询效率。

如下面的示例：

```
CREATE TABLE IF NOT EXISTS demo.example_tbl  
(  
    `user_id` LARGEINT NOT NULL COMMENT "用户id",  
    `date` DATE NOT NULL COMMENT "数据灌入日期时间",  
    `city` VARCHAR(20) COMMENT "用户所在城市",  
    `age` SMALLINT COMMENT "用户年龄",  
    `sex` TINYINT COMMENT "用户性别",  
    `last_visit_date` DATETIME REPLACE DEFAULT "1970-01-01 00:00:00" COMMENT "用户最后一次访问时间",  
    `cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费",  
    `max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间",  
    `min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时间"  
)  
AGGREGATE KEY(`user_id`, `date`, `city`, `age`, `sex`)  
DISTRIBUTED BY HASH(`user_id`) BUCKETS 1  
PROPERTIES (  
    "replication_allocation" = "tag.location.default: 1"  
);
```

上例中，前缀索引就是 user\_id(8字节) + date(3字节) + city(20)字节，虽然说前三个字段没有达到36字节，但是由于遇到了varchar，所以自动截止，不再往下了。

如果我们查询`where age = 20`，那么就不会走到前缀索引。其查询效率远远比不上`where user_id = 1001 and age = 20`（走到了前缀索引）。

有人肯定有疑问了，where条件千变万化，我怎么能保证一定能命中前缀索引呢？只有36字节的限制实在是太小了。不可能囊括所有查询场景。

的确如此，所以Doris推出了rollup的概念，可以通过rollup来调整前缀索引，使得常用的查询都能命中前缀索引，从而加速查询。这点我们在后面介绍rollup的时候再说。

### 倒排索引

是的，你没有看错。这个才是原汁原味的倒排索引。

相比于ClickHouse里的inverted，Doris的倒排，才是真正的大杀器。这也是为什么本文迫不及待想要了解Doris的主要原因。

与Doris的inverted比起来，ClickHouse的倒排索引粗陋、简单，更像一个还未进化成熟的小玩具。

语法如下：

```
CREATE TABLE table_name  
(  
  columns_difinition,  
  INDEX idx_name1(column_name1) USING INVERTED [PROPERTIES("parser" = "english|unicode|chinese")] [COMMENT 'your comment']  
  INDEX idx_name2(column_name2) USING INVERTED [PROPERTIES("parser" = "english|unicode|chinese")] [COMMENT 'your comment']  
  INDEX idx_name3(column_name3) USING INVERTED [PROPERTIES("parser" = "chinese", "parser_mode" = "fine_grained|coarse_grained")] [COMMENT 'your comment']  
  INDEX idx_name4(column_name4) USING INVERTED [PROPERTIES("parser" = "english|unicode|chinese", "support_phrase" = "true|false")] [COMMENT 'your comment']  
  INDEX idx_name5(column_name4) USING INVERTED [PROPERTIES("char_filter_type" = "char_replace", "char_filter_pattern" = "._"), "char_filter_replacement" = " "] [COMMENT 'your comment']  
  INDEX idx_name5(column_name4) USING INVERTED [PROPERTIES("char_filter_type" = "char_replace", "char_filter_pattern" = "._")] [COMMENT 'your comment']  
)  
table_properties;
```

其中：

* parser：分词器，不指定代表不分词。

+ english: 英文分词，用空格和标点进行分词
+ chinese：中文分词，性能比english低
+ unicode：中英文混合场景

* parser\_mode：分词模式

+ fine\_grained：细粒度模式
+ coarse\_grained：粗粒度模式（默认模式）

* support\_phrease：用于指定是否支持match\_phrease短语查询加速

+ true为支持，但是需要更多存储空间
+ false为不支持，更省存储空间，可以使用match\_all来查询多个关键字（默认配置）

* char\_filter：在分词前对字符串提前处理

+ char\_filter\_type：处理类型（当前仅支持char\_replace）

- char\_replace：将pattern中的字符做替换

* char\_fileter\_pattern：需要被替换的字符数组
* char\_filter\_replacement：替换后的字符数组，可以不用替换，默认是空格

对已有表增加倒排索引，需要显式物化，才能对存量数据生效。物化的语法如下：

```
BUILD INDEX index_name ON table_name [PARTITIONS(partition_name1, partition_name2)];
```

我们翻阅Doris源码，来探究一下Doris的分词原理。可以看到源码里，对分词器一共分成了三种类型，分别为standard(unicode)、simple(english)和chinese。具体的实现还是直接使用了lucence的分词逻辑， 因此可以达到和ES相差无几的效果。

```
//be/src/olap/rowset/segment_v2/inverted_index_reader.cpp  
std::unique_ptr<lucene::analysis::Analyzer> InvertedIndexReader::create_analyzer(  
        InvertedIndexCtx* inverted_index_ctx) {  
    std::unique_ptr<lucene::analysis::Analyzer> analyzer;  
    auto analyser_type = inverted_index_ctx->parser_type;  
    if (analyser_type == InvertedIndexParserType::PARSER_STANDARD ||  
        analyser_type == InvertedIndexParserType::PARSER_UNICODE) {  
        analyzer = std::make_unique<lucene::analysis::standard95::StandardAnalyzer>();  
    } else if (analyser_type == InvertedIndexParserType::PARSER_ENGLISH) {  
        analyzer = std::make_unique<lucene::analysis::SimpleAnalyzer<char>>();  
    } else if (analyser_type == InvertedIndexParserType::PARSER_CHINESE) {  
        auto chinese_analyzer =  
                std::make_unique<lucene::analysis::LanguageBasedAnalyzer>(L"chinese", false);  
        chinese_analyzer->initDict(config::inverted_index_dict_path);  
        auto mode = inverted_index_ctx->parser_mode;  
        if (mode == INVERTED_INDEX_PARSER_COARSE_GRANULARITY) {  
            chinese_analyzer->setMode(lucene::analysis::AnalyzerMode::Default);  
        } else {  
            chinese_analyzer->setMode(lucene::analysis::AnalyzerMode::All);  
        }  
        analyzer = std::move(chinese_analyzer);  
    } else {  
        // default  
        analyzer = std::make_unique<lucene::analysis::SimpleAnalyzer<char>>();  
    }  
    return analyzer;  
}
```

上述代码中，英文分词采用的是simple的分词器，其分词逻辑非常简单：

```
template<typename T>  
bool SimpleTokenizer<T>::isTokenChar(const T c) const {  
    //return _istalnum(c)!=0;  
    return (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z') || (c >= '0' && c <= '9');  
}
```

中文分词则是从本地的dict中读取的，本地的路径通过inverted\_index\_dict\_path来配置，默认是在dict目录下，我们打开be的dict目录，可以看到如下内容：

可以看到，Doris使用的中文分词器是大名鼎鼎的jieba分词。并在初始化阶段，将所有的字典都加载进去。

```
static cppjieba::Jieba& getInstance(const std::string& dictPath = "") {  
        static cppjieba::Jieba instance(dictPath + "/" + "jieba.dict.utf8",  
                                        dictPath + "/" + "hmm_model.utf8",  
                                        dictPath + "/" + "user.dict.utf8",  
                                        dictPath + "/" + "idf.utf8",  
                                        dictPath + "/" + "stop_words.utf8");  
        return instance;  
    }
```

其中：

* jieba.dict.utf8：最大概率法分词
* hmm\_model.utf8：隐式马尔科夫分词
* idf.utf8：在KeywordExtractor中，使用的是经典的TF-IDF算法，所以需要这么一个词典提供IDF信息
* stop\_words.utf8：停用词词典

Unicode采用standard分词，其逻辑主要是利用flex词法解析器去解析stop\_word。

lucence standard95下包含的stop\_word包含以下内容：

```
static std::unordered_set<std::string_view> stop_words = {  
    "a",    "an",   "and",  "are",  "as",   "at",    "be",   "but",   "by",  
    "for",  "if",   "in",   "into", "is",   "it",    "no",   "not",   "of",  
    "on",   "or",   "such", "that", "the",  "their", "then", "there", "these",  
    "they", "this", "to",   "was",  "will", "with"};
```

由于是按照unicode进行分词的，所以这种分词几乎支持所有的语言类型，包括中文、英文、韩文、日文、emoji表情等，所有类型列举如下：

```
 public:  
  static constexpr int32_t WORD_TYPE = 0; //单词  
  static constexpr int32_t NUMERIC_TYPE = 1; // 数字  
  static constexpr int32_t SOUTH_EAST_ASIAN_TYPE = 2; //东南亚  
  static constexpr int32_t IDEOGRAPHIC_TYPE = 3;    //表意文字  
  static constexpr int32_t HIRAGANA_TYPE = 4;   //日语  
  static constexpr int32_t KATAKANA_TYPE = 5;   // 片假名  
  static constexpr int32_t HANGUL_TYPE = 6;     // 韩文  
  static constexpr int32_t EMOJI_TYPE = 7;      //emoji表情
```

但是因为其是要实时通过词法解析的，所以从效率上来讲，肯定没有纯english的快。

### BloomFilter

BloomFilter索引的概念在clickhouse中也有，所以此处就不深入介绍了。

其语法如下：

```
PROPERTIES (  
"bloom_filter_columns"="saler_id,category_id",  
)
```

我们只需要指定字段即可，而无需指定bloom filter的长度和hash函数的个数（和clickhouse的bloom filter的二级索引相同）。

它可以指定多个列，但不可以创建多个索引。其索引粒度是block。这里提一嘴，前缀索引是以Block为粒度创建的稀疏索引，一个Block包含1024行数据，每个Block，以该Block的第一行数据的前缀列的值作为索引。

它的主要使用场景有：

* 非前缀过滤
* 适用于高基数列
* 查询条件是in和=（不支持like我也是没想到）

### NGram BloomFilter

前面不是说BloomFilter不支持like么，这不就安排了？

NGram BloomFilter主要就是为了增强like查询性能的二级索引。

其语法如下：

```
CREATE TABLE xxx (  
    ...  
    INDEX idx_ngrambf (`username`) USING NGRAM_BF PROPERTIES("gram_size"="3", "bf_size"="256") COMMENT 'username ngram_bf index'  
) ...
```

gram的个数跟实际查询场景相关，通常设置为大部分查询字符串的长度，bloom filter字节数，可以通过测试得出，通常越大过滤效果越好，可以从256开始进行验证测试看看效果。当然字节数越大也会带来索引存储、内存cost上升。

如果数据基数比较高，字节数可以不用设置过大，如果基数不是很高，可以通过增加字节数来提升过滤效果。

需要注意的是， NGram BloomFilter索引和BloomFilter索引为互斥关系，即同一个列只能设置两者中的一个。

### Bitmap

位图结构，主要用来加速查询。这个索引使用的并不多，未来可能会被倒排所替代。

语法如下：

```
CREATE INDEX [IF NOT EXISTS] index_name ON table1 (siteid) USING BITMAP COMMENT 'balabala';
```

## rollup和物化视图

### rollup

rollup称之为"上卷"，它的概念有点类似clickhouse中的projection。但与projection又有所不同（功能会弱很多）。

rollup的数据是会物化存储到磁盘上的，其生命周期和base表一样，同生同灭。

rollup语法如下：

```
ALTER TABLE table1 ADD ROLLUP rollup_city(citycode, pv);
```

修改后，无需显式物化，它会自动在后台进行存量数据的物化。

rollup的作用主要有两个：

* 减少查询的范围
* 调整前缀索引，加速查询

### 物化视图

物化视图其实是rollup的一种能力补足。因为rollup是不支持基于明细模型做预聚合的，而物化视图是在rollup的基础上增加了预聚合的能力。

物化视图的数据依赖于底表，但是生命周期需要单独管理，和底表完全独立。你可以将物化视图理解为一种特殊的表。

物化视图语法示例：

```
CREATE MATERIALIZED VIEW < MV name > as   
SELECT select_expr[, select_expr ...]  
FROM [Base view name]  
GROUP BY column_name[, column_name ...]  
ORDER BY column_name[, column_name ...]  
[PROPERTIES ("key" = "value")]
```

Doris物化视图比较牛逼的地方在于，它可以在查询时自动匹配，也就是说，在查询时，我们依然可以查底表，Doris会根据查询语句自动选择一个最优的物化视图进行查询，而不需要显示地指定查询物化视图。

## join查询

Doris提供了多种join方式。FE在规划分布式查询计划时，优先选择的顺序为：

```
Colocate Join -> Bucket Shuffle Join -> Broadcast Join -> Shuffle Join
```

其中：

* colocate join:

+ 提出CG的概念（colocation group）， 即将需要进行查询 的多张表编入一组group中，这些表具有相同的hash字段，相同的分桶类型，分桶树以及副本数
+ CG的作用是使所有的join操作都是本地join，而不需要分布式查询。
+ CG的创建语法：

CREATE TABLE tbl (k1 int, v1 int sum)  
DISTRIBUTED BY HASH(k1)  
BUCKETS 8  
PROPERTIES(  
"colocate\_with" = "group1"  
);

+ 由于colocate表的数据要保证相同hash key的数据处于相同的node上，所以做均衡的时候，数据迁移是要同步的， 这难免会带来迁移的资源开销，且存在一定的数据倾斜风险。
+ 当副本数据在进行修复或者迁移时，colocate表处于不可用状态，此时colocate join会退化为普通的join，会极大降低查询性能

* bucket shuffle join:

+ 只生效于join条件为等值的场景。依赖hash来计算确定的数据分布
+ 右表加载到内存，将右（小）表先查出来，然后根据hash计算出来的数据分布，将小表的数据发送到各个节点进行本地join
+ 只能保证左表为单分区时生效（where条件筛选出来的数据处于同一个分区）
+ 类似clickhouse的global join

* broadcast join：

+ 将右表全量数据发送到各个节点，与左表在各个节点上做本地join， 内存和网络开销都是N\*B

* shuffle join:

+ 将左表和右表的数据经过hash计算分散到各个节点中，网络开销为 A+ B,内存开销为B。

除此之外，Doris为了加速join查询，还提供了runtime filter机制。

所谓的runtime filter，因为一般左表join右表，右表需要加载到内存，通常会比较小，所以当扫描左表和加载右表同时进行时，右表一般会率先完成，此时根据 join on cause动态生成一些过滤条件，并广播给正在各个节点扫描的左表，使得左表扫描的数据量减少，从而加速整个查询，避免不必要的网络开销。（有点类似谓词下推，但不完全是）

因此，runtime filter主要对左表很大，右表很小的情况下有明显的优化效果。如果左表和右表的规模相差不大，则加速效果不大。

| shuffle方式 | 网络开销 | 内存开销 | 物理算子 | 适用场景 |
| --- | --- | --- | --- | --- |
| broadcast join | N\* T(R) | N\*T(R) | Hash Join/Nest Loop Join | 通用 |
| shuffle join | T(S)+ T(R) | T(R) | Hash Join | 通用 |
| bucket shuffle join | T(R) | T(R) | Hash Join | 左表为单分区 |
| colocate join | 0 | T(R) | Hash Join | 左右表属于同一个CG |

## 数据写入

Doris支持很多中数据源的数据导入，它提供了丰富的内置数据导入方案。如insert 语法， 利用broker load导入，routine load导入， spark load导入等，同时，我们也可以通过诸如SetTuneal之类的第三方工具进行数据导入。

Doris内置的数据导入方式支持CSV、ORC、JSON等多种格式。接下来我们就简单介绍一下比较常见的数据导入方式。

## broker load

在文章一开头介绍Doris组件时，就介绍过Doris除了FE和BE之外，还有一类可选组件叫broker。broker组件就是专门用来写入数据的。

broker组件需要额外安装。

它的原理是由FE创建broker计划，然后BE根据计划从具体的broker去拉取数据。

broker支持的数据源包括HDFS、BOS、AFS等文件系统。

使用broker load导入数据的语法如下：

```
LOAD LABEL broker_load_2022_03_23  
(  
    DATA INFILE("hdfs://192.168.20.123:8020/user/hive/warehouse/ods.db/ods_demo_detail/*/*")  
    INTO TABLE doris_ods_test_detail  
    COLUMNS TERMINATED BY ","  
  (id,store_id,company_id,tower_id,commodity_id,commodity_name,commodity_price,member_price,cost_price,unit,quantity,actual_price)   
    COLUMNS FROM PATH AS (`day`)  
   SET   
   (rq = str_to_date(`day`,'%Y-%m-%d'),id=id,store_id=store_id,company_id=company_id,tower_id=tower_id,commodity_id=commodity_id,commodity_name=commodity_name,commodity_price=commodity_price,member_price=member_price,cost_price=cost_price,unit=unit,quantity=quantity,actual_price=actual_price)  
    )  
WITH BROKER "broker_name_1"   
    (   
      "username" = "hdfs",   
      "password" = ""   
    )  
PROPERTIES  
(  
    "timeout"="1200",  
    "max_filter_ratio"="0.1"  
);
```

broker load 方式支持ORC、CSV、parquet、gzip等格式的数据。

## stream load

stream load主要用于导入本地文件或者内存中的数据，它通过HTTP协议将数据写入到Doris。支持CSV和JSON格式。

stream load方式写入数据，会先选定一个BE节点作为Coordinator（协调者）节点，数据会先发往Coordinator节点，然后由Coordinator节点分发到各个BE节点，因此会出现写放大现象。

常用语法如下所示：

```
curl --location-trusted -u user:passwd [-H ""...] -T data.file -XPUT http://fe_host:http_port/api/{db}/{table}/_stream_load
```

stream load的任务无法手动取消，只能等待其成功或出错退出。

## routine load

例行导入，目前仅支持kafka数据源。支持CSV和JSON格式。

FE会将一个导入作业拆分成若干task，每个task负责导入一部分的数据，不同的task被分配到不同的BE上去执行。

在BE上，每个task会被当成普通的stream load任务去执行，导入完成后，向FE汇报。

FE根据汇报结果，继续生成新的task，或重试失败的task。

支持无认证的kafka集群，SSL认证的kafka集群，以及kerberos认证的kafka集群。

在作业运行期间，如果修改表的schema，或者删除partition，可能会导致任务失败或者阻塞。

由于有task失败重试机制，所以在作业期间，即使kafka出现短暂失联，依然不影响数据的写入。

其语法如下所示：

```
CREATE ROUTINE LOAD example_db.test_json_label_1 ON table1  
COLUMNS(category,price,author)  
PROPERTIES  
(  
    "desired_concurrent_number"="3",  
    "max_batch_interval" = "20",  
    "max_batch_rows" = "300000",  
    "max_batch_size" = "209715200",  
    "strict_mode" = "false",  
    "format" = "json"  
)  
FROM KAFKA  
(  
    "kafka_broker_list" = "broker1:9092,broker2:9092,broker3:9092",  
    "kafka_topic" = "my_topic",  
    "kafka_partitions" = "0,1,2",  
    "kafka_offsets" = "0,0,0"  
 );
```

## 其他

除了上面三种常见的写入方式外，Doris还提供了一些其他的写入方式。如：

* spark load：

+ 通过spark任务实现对数据导入的预处理，如排序，分区，聚合，构建索引等
+ 由于预处理被spark任务提前完成了，因此可以大大节省Doris的资源消耗（如果spark资源和Doris部署在同一台机器上，当老夫没说）
+ 支持所有spark资源可以访问的数据源，如HDFS、HIVE

* mysql load：

+ 说白了还是stream load，都是导入本地数据到Doris，不过是以mysql的SQL语法方式（load data infile xxxx into table）
+ 仅支持CSV格式。

* s3 load:

+ 顾名思义，就是将S3的数据导入到Doris
+ 语法和broker类似，举例如下：

```
LOAD LABEL example_db.exmpale_label_1  
 (  
 DATA INFILE("s3://your_bucket_name/your_file.txt")  
 INTO TABLE load_test  
 COLUMNS TERMINATED BY ","  
 )  
 WITH S3  
 (  
 "AWS_ENDPOINT" = "AWS_ENDPOINT",  
 "AWS_ACCESS_KEY" = "AWS_ACCESS_KEY",  
 "AWS_SECRET_KEY"="AWS_SECRET_KEY",  
 "AWS_REGION" = "AWS_REGION"  
 )  
    PROPERTIES  
 (  
 "timeout" = "3600"  
 );
```

* insert into:

+ 这个就不用解释了，使用SQL的方式插入数据。

## 与ClickHouse对比

## 集群部署运维难度

由于Doris自带的FE具有元数据管理能力，不需要像ClickHouse还要依赖zookeeper或者clickhouse-keeper这类第三方元数据管理工具，所以Doris的集群的集群运维管理非常方便。

以增加节点为例。

Doris增加节点，只需要一条SQL就能搞定。

它自己内部会自动维护配置文件，并同步表的结构。

如果是ClickHouse，需要维护者手动修改集群配置，手动同步元数据信息，如果涉及到分片的扩容，还需要考虑数据的再均衡。整套流程操作下来繁琐易出错，没有二十年的功力，根本挡不住。

但话又说回来，老夫若祭出ckman神器，阁下又该如何应对？

说句大不要脸的话，这一局勉强算个“平手”，不过分吧（阴险笑）？

## 生态完备性

如果把ClickHouse比作手动挡超级跑车，那么Doris更像是具备了智能车机平台的自动挡新能源势力。

ClickHouse什么都需要自己去做 。集群管理需要自己做，数据写入需要自己做，更别说让人眼花缭乱的海量调优的参数了。甚至SQL都搞了许多方言实现，需要很多的额外改写工作。如果不是对ClickHouse有专业级理解，很难玩得转ClickHouse（不是说用不起来，而是说释放不出ClickHouse的全部性能）。

而Doris就鸡贼很多，完备的数据写入能力，查询能力，完全兼容MySQL协议的语法，你甚至用MySQL的client都能连接到Doris上，这对于刷惯了MySQL八股文的国人开发者来说，这在开发和使用上，几乎没有什么学习成本，岂不是有手就行？

## 数据写入

Doris内置了很多数据导入的工具，并且由于Doris本身事务的支持，使得Doris写入数据容错能力，各种数据源的接入能力都比较优秀。更离谱的是，stream load、routine load、broker load等导入工具还支持简单的过滤和解析逻辑，简直是一条龙服务。

而ClickHouse并没有提供专门的导数方案。虽然也有Kafka、HDFS等外部数据引擎可以直接查询外部存储的数据，但由于不再是Mergetree 引擎，无法利用Mergetree的特性对查询进行加速。我们必须自己实现或者依赖第三方的数据写入工具来进行数据的导入，如clickhouse\_sinker、Seatuneal等。

但从写入性能方面来说，Doris单个任务只能做到50M/s左右，而clickhouse则可以达到200M/s， 速度可谓是碾压。

当然Doris可以通过增加并发度来提高写入速度，但代价就是要消耗更多的资源。

## 数据查询

首先全文检索方面，Doris支持倒排索引，clickhouse仅支持通过布隆过滤器来进行加速，不消说完败。（听到了听到了，别再鞭尸了）

其次是分布式join查询能力。ClickHouse仅提供broadcast join，而Doris的bucket shuffle join以及colocate join都是非常牛逼的存在。虽然clickhouse可以通过手动指定hash key来达到colocate join的效果，但毕竟不是原生就支持的能力。

double kill。

clickhouse和Doris都是列存，同样都支持向量化搜索。所谓向量化，说通俗点就是数据一批一批的去执行，多批数据之间可以并发执行，这种查询可以大大加速查询性能。在单表查询以及聚合查询场景下，clickhouse的能力还是要比Doris要强不少的。

而且Doris的列存不同于其他的OLAP数据库，特别是在Aggregate模型下，由于其内部加入了聚合算子，如果要计算count，性能会特别拉胯。而Clickhouse由于在批量插入时，会同时记录count到磁盘，基本可以秒出结果。

## 用户权限管理

Doris的权限管理比较粗糙，ClickHouse不仅支持完善的行级别查询限制，以及用户级别的内存限制，查询内存限制，还支持查询行数的限制，而这些都是Doris所不具备的。

clickhouse利用这些权限管理，可以提供更稳定的体验。比如限制查询用户的内存和线程数，保证导入数据的性能不受影响，以及一些大查询的查询次数限制，返回行数限制等。从而减小查询带来的大开销。

## 总结

到底是我太天真了。以为靠一篇文章可以将Doris一把梭。

其实仔细想想怎么可能。clickhouse那样的体量，研究并使用了这么多年尚且没搞明白，Doris毕竟是对标clickhouse的存在，自然有其该有的深度与咖位。

所以本文权且当做一个引子，仅作为Doris入门读物。后面我将分专题分享一些Doris的性能测试，以及与clickhouse的对比文章。让我们看看Doris到底能快到什么地步，而clickhouse这部手动超级跑车，能否通过老司机的神（性）级（能）操（调）作（优），让其发挥出不亚于Doris的性能。

让我们拭目以待。

原文地址：https://zhuanlan.zhihu.com/p/672707775