---
title: FlinkCdc中Event最详解，你不知道了吧？
author: 阿龙大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA3OTExMjA0MA==&mid=2247485253&idx=1&sn=9745e1473d66641c745584622fb25b4e&chksm=9e2b93335fb0a0cd7c8f6f15700f940ae372b70665300af7853adb5d9f8fbd9ee42bc9b814f0&mpshare=1&scene=24&srcid=0110JgqiUt8iwONnW4EYQZPF&sharer_shareinfo=802d60ee3944cd87cf8f1042b84eb92e&sharer_shareinfo_first=802d60ee3944cd87cf8f1042b84eb92e#rd
---

一、介绍

    Flink CDC 环境下的事件是 Flink 数据流中的一种特殊记录，它描述在源端外部系统中捕获到的变化，经过 Flink CDC 构建的内部算子进行处理和转换，最终传递到数据接收器端写入或应用到接收器端外部系统。每个更改事件都包含其所属的表 ID 以及事件携带的有效负载。

二、事件分类

1、数据变更事件（DataChangeEvent）：

DataChangeEvent 描述源中的数据变化。它由 5 个字段组成。

Table ID：

    所属表的ID

Before：

    数据的原像

After：

    数据的后像

Operation type：

    变更操作的类型

Meta：

    变更的元数据

对于操作类型字段，我们预定义了4种操作类型：

1.1、插入：

    新数据条目，带有before = null和after = new data

1.2、删除：

    删除数据，同时保留before = removed数据和after = null

1.3、更新：

    更新现有数据，before = data before change 并after = data after change

1.4、代替：~

2、Schema变更事件（SchemaChangeEvent ）：

    SchemaChangeEvent描述的是源端的表结构变化。相比于 DataChangeEvent，SchemaChangeEvent 的 payload 描述的是外部系统中表结构的变化，包括：

2.1、AddColumnEvent：

    表格中的新列

2.2、AlterColumnTypeEvent：

    改变列的类型

2.3、CreateTableEvent：

    创建新表。还用于描述预先发出的DataChangeEvent 的架构。

2.4、DropColumnEvent：

    删除一列

2.5、RenameColumnEvent：

    更改列的名称

3、Event解析：

3.1、数据案例

```
{  "before": {    "id": "PF1784570096901248",    "out_no": "J1784570080435328",    "add_time": 1686758315000,    "remark": "充值办卡"  },  "after": {    "id": "PF1784570096901248",    "out_no": "J1784570080435328",    "add_time": 1686758315000,    "remark": "充值办卡1"  },  "source": {    "version": "1.6.4.Final",    "connector": "mysql",    "name": "mysql_binlog_source",    "ts_ms": 1686734882000,    "snapshot": "false",    "db": "cloud_test",    "sequence": null,    "table": "acct_profit",    "server_id": 1,    "gtid": null,    "file": "mysql-bin.000514",    "pos": 650576218,    "row": 0,    "thread": null,    "query": null  },  "op": "u",  "ts_ms": 1686734882689,  "transaction": null}
```

3.2、数据解析

    这是一个Flink CDC（Change Data Capture）的数据格式示例，用于表示数据库中一行数据的变更信息。以下是对这个示例数据的解析：

before：

    表示变更前的数据，包含了旧的字段值。

after：

    表示变更后的数据，包含了新的字段值。

source：

    包含了事件的元数据信息，例如事件发生的时间戳、数据库名称、表名、操作类型等。

op：

    表示操作类型，这里是u，代表update操作。

ts\_ms：

    表示事件的时间戳。

transaction：

    表示事务信息，这里是null，表示该事件不属于任何事务。

说明：

    在这个示例中，表示的是一条对acct\_profit表的update操作，具体变更内容如下：将remark字段的值从“充值办卡”更新为“充值办卡1”。其他字段的数值均保持不变。

3.3、Event中source数据含义

```
"source": {    "version": "1.6.4.Final",    "connector": "mysql",    "name": "mysql_binlog_source",    "ts_ms": 1686734882000,    "snapshot": "false",    "db": "cloud_test",    "sequence": null,    "table": "acct_profit",    "server_id": 1,    "gtid": null,    "file": "mysql-bin.000514",    "pos": 650576218,    "row": 0,    "thread": null,    "query": null  }
```

    在Flink CDC数据中，source字段包含了事件的元数据信息，用于描述事件的来源和相关信息。以下是对source字段中各个子字段的解析：

version：

    Debezium或其他CDC组件的版本号。

connector：

    指示所使用的连接器类型，这里是mysql，表示是MySQL数据库的数据变更事件。

name：

    连接器的名称，这里是mysql\_binlog\_source。

ts\_ms：

    事件的时间戳，表示事件发生的时间。

snapshot：

    指示是否为快照事件，这里是false，表示不是快照事件。

db：

    数据库名称，这里是cloud\_test。

sequence：

    事件的序列号，一般用于多台数据变更源的复制同步。

table：

    表名，这里是acct\_profit。

server\_id：

    服务器ID，用于标识不同的MySQL实例。

gtid：

    全局事务ID，用于标识全局事务的唯一标识。

file：

    binlog文件名，记录了MySQL二进制日志的文件名。

pos：

    binlog文件中的位置，表示事件的位置。

row：

    在binlog文件中的行号。

thread：

    事件所在的线程。

query：   

    可能包含了触发事件的SQL查询语句。

3.4、对比：source中的ts\_ms 和外层ts\_ms对比

记录每一份热爱,让美好永远陪伴。