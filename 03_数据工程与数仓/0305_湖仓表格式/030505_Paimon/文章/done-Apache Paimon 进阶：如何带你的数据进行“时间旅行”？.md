> 已吸收至：[[03_数据工程与数仓/0305_湖仓表格式/030505_Paimon/030505_核心知识点/Paimon时间旅行快照与系统表|Paimon时间旅行快照与系统表]]
---
title: Apache Paimon 进阶：如何带你的数据进行“时间旅行”？
author: 数仓生态圈
date: 数仓生态圈数仓生态圈
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODY0NjY1Ng==&mid=2247486655&idx=1&sn=140f9ecda35f46f7875178e1d10fb215&chksm=c4d206468889b053aba5b19d6c85b0b1855325498b6b8aff7431ca6958b14162ce3db77ce515&mpshare=1&scene=24&srcid=0519Cl0yIXJZ2oyrUXeMkEO9&sharer_shareinfo=4f9d852b64dbbb015164de934a79b3c1&sharer_shareinfo_first=4f9d852b64dbbb015164de934a79b3c1#rd
---

一、什么是快照

快照（Snapshot），简单来说就是表在某一次提交后的状态记录。和我们平时说的数据库备份不一样，数据库备份是对当前最新数据的备份，而 Paimon 的快照记录的是每次提交时刻表的完整状态——包括这次提交新增了哪些文件、修改了什么、删除了什么。

Paimon 的快照存储结构是这样的：

```
snapshot → manifest list → manifest file → data file
```

一次快照会指向一个 manifest list，manifest list 里面记录了多个 manifest file，而 manifest file 记录了具体的数据文件信息。有了这个层级关系，我们就可以通过快照回溯到任意一次提交时表的状态，这就是我们说的"时间旅行"。

二、快照生成

我们先建一张表，插入几条数据，生成快照

```
create table snaptable (  id BIGINT,  name STRING,  PRIMARY KEY (id) NOT ENFORCED);insert into snaptable values(1,'aa');insert into snaptable values(2,'bb');insert into snaptable values(3,'cc');
```

所以每个 commit 都会生成一个快照。第一次 insert 生成了 snapshot-1，第二次生成了 snapshot-2，第三次生成了 snapshot-3。

三、scan.mode 参数

这个参数决定了读取数据时的起始策略，不管是流还是批都会用到。

|  |  |
| --- | --- |
| **scan.mode 值** | **说明** |
| `default` | 流模式下等同`latest`，批模式下等同`compacted-full` |
| `latest` | 从最新的快照开始读取（流模式默认） |
| `latest-full` | 先读最新快照的全量数据，再继续消费增量 |
| `from-snapshot` | 从指定快照 ID 开始读取 |
| from-snapshot-full | 从指定快照 ID 的全量数据开始，再继续消费增量 |
| `compacted-full` | 读取最近一次压缩（compaction）产生的快照全量数据（批模式默认） |
| `incremental` | 增量读取 |

Paimon 会自动推断模式，所以不设置Paimon也会自动帮我们设置好对应的模式。

四、流处理查询

1、可以根据快照的id进行查询，也就是名字-后面的数字

```
Flink SQL> SELECT * FROM snaptable /*+ OPTIONS('scan.snapshot-id' = '1') */;+----+----------------------+--------------------------------+| op |                   id |                           name |+----+----------------------+--------------------------------+| +I |                    1 |                             aa || +I |                    2 |                             bb || +I |                    3 |                             cc |
Flink SQL> SELECT * FROM snaptable /*+ OPTIONS('scan.snapshot-id' = '3') */;+----+----------------------+--------------------------------+| op |                   id |                           name |+----+----------------------+--------------------------------+| +I |                    3 |                             cc |
```

可以看到流模式下查询最开始的快照时，返回的不是快照1的数据，而是从快照1开始到最新的数据增量都显示出来。

2、根据提交时间查询

可以通过查询paimon系统表来查看快照生成的具体时间

```
Flink SQL> SELECT snapshot_id,commit_time,commit_kind FROM `snaptable$snapshots`;+----+----------------------+-------------------------+--------------------------------+| op |          snapshot_id |             commit_time |                    commit_kind |+----+----------------------+-------------------------+--------------------------------+| +I |                    1 | 2026-05-17 18:56:15.128 |                         APPEND || +I |                    2 | 2026-05-17 18:56:21.945 |                         APPEND || +I |                    3 | 2026-05-17 18:56:29.872 |                         APPEND |+----+----------------------+-------------------------+--------------------------------+
```

根据这个commit\_time来进行查询，查询的是根据提交时间以后的数据

```
Flink SQL> SELECT * FROM snaptable FOR SYSTEM_TIME AS OF TIMESTAMP '2026-05-17 18:56:15.128';+----+----------------------+--------------------------------+| op |                   id |                           name |+----+----------------------+--------------------------------+| +I |                    1 |                             aa || +I |                    2 |                             bb || +I |                    3 |                             cc |Flink SQL> SELECT * FROM snaptable FOR SYSTEM_TIME AS OF TIMESTAMP '2026-05-17 18:56:16.128';+----+----------------------+--------------------------------+| op |                   id |                           name |+----+----------------------+--------------------------------+| +I |                    2 |                             bb || +I |                    3 |                             cc |
```

如果想查询2个时间点增量的数据，可以使用下面方式

```
Flink SQL> SELECT * FROM snaptable /*+ OPTIONS('incremental-between-timestamp' = '2026-05-17 18:56:14.128,2026-05-17 18:56:22.945') */;
```

在flink web ui上查看有异常，增量查询只适合在批模式下查看

```
Caused by: java.lang.IllegalArgumentException: Cannot read incremental in streaming mode.  at org.apache.paimon.utils.Preconditions.checkArgument(Preconditions.java:127)  at org.apache.paimon.table.source.AbstractDataTableScan.createStartingScanner(AbstractDataTableScan.java:281)  at org.apache.paimon.table.source.DataTableStreamScan.initScanner(DataTableStreamScan.java:142)  at org.apache.paimon.table.source.DataTableStreamScan.plan(DataTableStreamScan.java:124)
```

五、批模式下查询

流模式与批模式基本用法都差不多，但是返回的数据含义却大不同

```
Flink SQL> SELECT * FROM snaptable /*+ OPTIONS('scan.snapshot-id' = '1') */;+----+------+| id | name |+----+------+|  1 |   aa |+----+------+Flink SQL> SELECT * FROM snaptable /*+ OPTIONS('scan.snapshot-id' = '3') */;+----+------+| id | name |+----+------+|  1 |   aa ||  2 |   bb ||  3 |   cc |+----+------+
```

查询的就是快照对应时刻的全量数据而不是增量。

查询2个时间点增量的数据，批处理方式可以查询。

```
Flink SQL> SELECT * FROM snaptable /*+ OPTIONS('incremental-between-timestamp' = '2026-05-17 18:56:14.128,2026-05-17 18:56:22.945') */;+----+------+| id | name |+----+------+|  1 |   aa ||  2 |   bb |+----+------+
```

除了可以使用时间戳查询增量，也可以使用id查询增量

```
SELECT * FROM snaptable /*+ OPTIONS('incremental-between' = '1,3') */;
```

六、快照过期与清理

快照虽然好，但是也不能一直保留着，时间一长，HDFS 上的 snapshot 文件会越来越多，占的存储也会越来越大。所以 Paimon 提供了快照过期机制。

1、自动过期

Paimon 默认会保留最近的快照，超过数量的旧快照会被自动清理。

|  |  |  |
| --- | --- | --- |
| **参数** | **默认值** | **说明** |
| `snapshot.time-retained` | 1 h | 快照保留时长，超过此时长的快照会被标记过期 |
| `snapshot.num-retained.min` | 10 | 最少保留的快照数量 |
| `snapshot.num-retained.max` | Integer.MAX\_VALUE | 最多保留的快照数量 |

建表的时候可以进行设置

```
CREATE TABLE snaptable (  id BIGINT,  name STRING,  PRIMARY KEY (id) NOT ENFORED) WITH (  'snapshot.time-retained' = '24h',  'snapshot.num-retained.min' = '20',  'snapshot.num-retained.max' = '50');
```

2、手动清理

```
CALL sys.expire_snapshots('default.snaptable', '2026-05-17 18:56:20');
```

这条命令会把 2026-05-17 18:56:20 之前的快照都标记为过期并清理掉。

七、总结

Paimon 的快照机制给了我们"时间旅行"的能力

1、`scan.mode`是控制读取策略的核心参数，不同的值对应不同的读取方式。

2、流模式查增量，批模式查全量-同样的参数，两种模式返回的数据含义完全不同。

3、增量查询只支持批模式，流模式下会报错。

4、快照要及时清理，通过`snapshot.time-retained`和`snapshot.num-retained`控制过期策略。