---
title: 对StarRocks的tablet的理解和实验
author: PostgreSQL技术之家
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwMzY2NDMyMw==&mid=2247483750&idx=1&sn=d418e1d12dd021e5b266cb38622a0c53&chksm=c1ca686a883700e8663bfdd7633068886c3faa8edc22dd798e15f49557998327c1a949ebd6d4&mpshare=1&scene=24&srcid=0918qa2mZ5QedgkGPUpddl91&sharer_shareinfo=b71b638d588a76828a61242b88804daa&sharer_shareinfo_first=b71b638d588a76828a61242b88804daa#rd
---

## 0. starrocks的介绍

StarRocks是一款高性能、全场景的 MPP（大规模并行处理）列式分析型数据库，其通过列式存储和向量化引擎（对于x86为avx指令），可以对group by 等聚合操作达到极致的性能（单节点情况下亿级别数据秒级别出结果）。

1. 问与clickhouse有什么区别？

clickhouse不适合多表join，而StarRocks解决了这个瓶颈。

2. 与Greenplum有什么区别？

StarRocks是一个列式的引擎。在列式引擎上，StarRocks比Greenplum快很多。不过Greenplum长处在行式存储和对事务的完整支持，而StarRocks只支持原子操作，并不支持复杂的事务。

 

## 1 tablet 说明

### 1.1 tablet 是什么

* • Tablet 是表的逻辑分片。一张表可以有多个 Tablet，每个 Tablet 默认有 3 个副本（由 FE 参数default\_replication\_num 控制，此参数为动态参数，修改方式为：ADMIN SET FRONTEND CONFIG ("default\_replication\_num" = "2");即时生效）
* • StarRocks 采用多版本并发控制(MVCC)技术，通过复制这些多版本数据的物理副本，保证版本修复的高效进行。StarRocks是以 tablet 这个维度进行数据的管理

### 1.2 数据的写入流程

* • 客户端提交导入请求至 FE。
* • FE 节点选择一个 BE 节点作为该导入事务的 Coordinator BE 节点，并为该事务生成执行计划。
* • Coordinator 节点从客户端读取要导入的数据。
* • Coordinator 节点将数据分发到所有副本的 Tablet 中。
* • 数据导入并存储到所有 Tablet 后，FE 将导入的数据变为可见。
* • FE 向客户端返回导入成功

### 1.3 tablet 的管理

* • tablet 的元数据由 FE 节点进行管理，不同数量的 tablet 需要配置对应的 JVM 内存，以便管理
* • tablet过多会增加 FE/BE 的元数据管理和调度的资源消耗

| Tablet 数量 | FE 内存大小 |
| --- | --- |
| 100 万以下 | 16 GB |
| 100 万 ～ 200 万 | 32 GB（初始化建议， -Xms 和 -Xmx保持一致即可，所有FE节点均一致） |
| 200 万 ～ 500 万 | 64 GB |
| 500 万 ～ 1 千万 | 128 GB |

### 1.4 tablet 相关的问题处理

* • BE 节点崩溃或导入任务失败都可能导致副本异常。StarRocks 会自动修复这些异常的副本
* • 如果有表是单副本且该表的部分 tablet 分布在要删除的 BE 上，则不允许删除该 BE
* • 每隔 tablet\_sched\_checker\_interval\_seconds 中指定的周期，默认 20 秒，FE 中的 Tablet Checker 就会扫描 StarRocks 集群中所有表的 Tablet 副本
* • 在将一个表删除时，使用SHOW PROC '/statistic';查看会看到 tablet 立即减少了，但是 show backends;时，TabletNum 数量不会减少，是因为 SR 默认回收站保留一天的数据（由 动态 FE 参数catalog\_trash\_expire\_second 控制，默认 86400 秒）
* • 在添加 BE 或者下线 BE 节点时，系统会自动根据系统压力对 tablet 进行重平衡，此时 show backends;可以看到 tabletnum 数量再不断变化，如果是添加 BE 节点，则会将原有节点的 tablet 平衡一部分到新节点，如果是下线BE 节点则反之。

+ • 如果需要加快或者降低 tablet 平衡对现有集群的影响，可以调整如下几个FE参数
+ • tablet\_sched\_slot\_num\_per\_path（默认值：8）：一个 BE 存储目录能够同时执行 tablet 相关任务的数目（可用于降低扩缩容 tablet 迁移的影响）
+ • tablet\_sched\_max\_scheduling\_tablets（默认值10000）：可同时调度的 Tablet 的数量。如果正在调度的 Tablet 数量超过该值，跳过 Tablet 均衡和修复检查
+ • tablet\_sched\_max\_balancing\_tablets（默认值：500）：正在均衡的 Tablet 数量的最大值。如果正在均衡的 Tablet 数量超过该值，跳过 Tablet 重新均衡

### 1.5 tablet 相关查询

#### 1.5.1 查看数据库的统计信息

* • 查看当前集群各数据库的统计信息：SHOW PROC '/statistic';

| **返回** | **说明** |
| --- | --- |
| DbId | 数据库 ID。 |
| DbName | 数据库名称。 |
| TableNum | 数据库中表的数量。 |
| PartitionNum | 数据库中分区的数量。 |
| IndexNum | 数据库中索引的数量。 |
| TabletNum | 数据库中 Tablet 的数量。 |
| ReplicaNum | 数据库中副本的数量（tabletnum\*副本数量） |
| UnhealthyTabletNum | 数据重分布过程中还未完成的 Tablet 数量。 |
| InconsistentTabletNum | 数据库中不一致的 Tablet 数量。 |
| CloningTabletNum | 数据库中正在进行 Clone 操作的 Tablet 数量。 |
| ErrorStateTabletNum | 主键表中错误状态的 Tablet 数量。 |
| ErrorStateTablets | 主键表中错误状态的 Tablet 的 ID。 |

* • 进一步查看某个db下tablet的状态：show PROC '/statistic/dbid';

```
mysql> show proc '/statistic/356645';  
+------------------+---------------------+----------------+-------------------+  
| UnhealthyTablets | InconsistentTablets | CloningTablets | ErrorStateTablets |  
+------------------+---------------------+----------------+-------------------+  
| []               | []                  | []             | []                |  
+------------------+---------------------+----------------+-------------------+  
1 row in set (0.01 sec)
```

#### 1.5.2 查询表副本情况

##### 1.5.2.1 查询所有副本情况

```
#查看某个table下的tablet，可以根据字段条件进行过滤  
mysql> show tablet from db2.tbl2 where BackendId=10170\G  
*************************** 1. row ***************************  
               TabletId: 371281  
              ReplicaId: 371282  
              BackendId: 10170  
             SchemaHash: 0  
                Version: 4  
            VersionHash: 0  
      LstSuccessVersion: 4  
  LstSuccessVersionHash: 0  
       LstFailedVersion: -1  
   LstFailedVersionHash: 0  
          LstFailedTime: NULL  
               DataSize: 1.7GB  
               RowCount: 90000000  
                  State: NORMAL  
LstConsistencyCheckTime: NULL  
           CheckVersion: -1  
       CheckVersionHash: 0  
           VersionCount: 1  
               PathHash: -1  
                MetaUrl: http://10.197.165.202:8040/api/meta/header/371281  
       CompactionStatus: http://10.197.165.202:8040/api/compaction/show?tablet_id=371281  
           DiskRootPath: Unknown  
           IsConsistent: true  
               Checksum: -1  
1 row inset (0.00 sec)
```

##### 1.5.2.2 查询副本同步状态

```
#查看表副本同步状态  
ADMIN SHOW REPLICA STATUS FROM db2.tbl2 WHERE STATUS != "OK";
```

##### 1.5.2.3 查询副本分布情况

```
#查看表所有副本的分布情况，可以根据条件进行过滤  
ADMIN SHOW REPLICA DISTRIBUTION FROM test.t1;  
  
+-----------+------------+-------+---------+  
| BackendId | ReplicaNum | Graph | Percent |  
+-----------+------------+-------+---------+  
| 10002     | 3          | >     | 33.33 % |  
| 10170     | 3          | >     | 33.33 % |  
| 10171     | 3          | >     | 33.33 % |  
+-----------+------------+-------+---------+  
3 rows in set (0.01 sec)
```

##### 1.5.2.4 查询具体的tablet

```
#首先查询表有哪些tablet  
SHOW tablet from db2.tbl2\G  
#查询某个tablet  
SHOW tablet 371281\G  
       DbName: db2  
    TableName: tbl2  
PartitionName: tbl2  
    IndexName: tbl2  
         DbId: 356645  
      TableId: 371279  
  PartitionId: 371278  
      IndexId: 371280  
       IsSync: true  
    DetailCmd: SHOW PROC '/dbs/356645/371279/partitions/371278/371280/371281';  
1 row inset (0.00 sec)  
#查询tablet的具体情况  
mysql> SHOW PROC '/dbs/356645/371279/partitions/371278/371280/371281'\G  
*************************** 1. row ***************************  
            ReplicaId: 371282  
            BackendId: 10170  
              Version: 4  
          VersionHash: 0  
    LstSuccessVersion: 4  
LstSuccessVersionHash: 0  
     LstFailedVersion: -1  
 LstFailedVersionHash: 0  
        LstFailedTime: NULL  
           SchemaHash: -1  
             DataSize: 1898090776  
             RowCount: 90000000  
                State: NORMAL  
                IsBad: false  
        IsSetBadForce: false  
         VersionCount: 1  
             PathHash: -1  
              MetaUrl: http://10.197.165.202:8040/api/meta/header/371281  
     CompactionStatus: http://10.197.165.202:8040/api/compaction/show?tablet_id=371281&schema_hash=-1  
         IsErrorState: false  
1 row inset (0.00 sec)
```

##### 1.5.2.5 查询副本数量

```
SELECT  
    table_schema,  
    table_name,  
    SUBSTRING_INDEX(SUBSTRING_INDEX(PROPERTIES, '"replication_num":"', -1), '"', 1) AS replication_num  
FROM  
    information_schema.tables_config  
WHERE  
    table_schema IN('test','db1','db2')  -- 选择数据库  
    AND PROPERTIES LIKE '%"replication_num":"3"}%'; -- 选择副本数量  
  
+--------------+------------+-----------------+  
| table_schema | table_name | replication_num |  
+--------------+------------+-----------------+  
| db1          | t2         | 3               |  
| test         | t1         | 3               |  
+--------------+------------+-----------------+  
2 rows inset (0.05 sec)
```

#### 1.5.3 查询 tablet 迁移状态

* • cluster\_load\_stat: 集群当前的负载状态。
* • working\_slots: 当前可用的工作插槽数。
* • sched\_stat: 调度系统的当前状态。
* • priority\_repair: 当前需要优先处理的 Tablet 修复任务数。
* • pending\_tablets: 当前等待处理的 Tablet 数量。
* • running\_tablets: 当前正在修复的 Tablet 数量。
* • history\_tablets: 历史上修复过的 Tablet 数量。

##### 1.5.3.1 还未添加 BE 节点时

```
mysql> SHOW PROC '/cluster_balance';  
  
+-------------------+--------+  
| Item              | Number |  
+-------------------+--------+  
| cluster_load_stat | 1      |  
| working_slots     | 3      |  
| sched_stat        | 1      |  
| priority_repair   | 0      |  
| pending_tablets   | 0      |  
| running_tablets   | 0      |  
| history_tablets   | 0      |  
| all_tablets       | 0      |  
+-------------------+--------+  
8 rows in set (10.19 sec)
```

##### 1.5.3.2 添加 BE 节点后

```
#可以看到正在执行的只有 8 个（对应tablet_sched_slot_num_per_path参数的默认值）  
mysql> SHOW PROC '/cluster_balance';  
+-------------------+--------+  
| Item              | Number |  
+-------------------+--------+  
| cluster_load_stat | 1      |  
| working_slots     | 4      |  
| sched_stat        | 1      |  
| priority_repair   | 0      |  
| pending_tablets   | 471    |  
| running_tablets   | 8      |  
| history_tablets   | 1000   |  
| all_tablets       | 479    |  
+-------------------+--------+  
8 rows in set (0.00 sec)
```

## 2 tablet生成测试

### 2.1 新增、删除表测试

#### 2.1.1 新建表之前

```
#当前tablet数量  
mysql> show backends\G  
*************************** 1. row ***************************  
            BackendId: 10002  
                   IP: 10.197.165.201  
        HeartbeatPort: 9050  
               BePort: 9060  
             HttpPort: 8040  
             BrpcPort: 8060  
        LastStartTime: 2025-09-01 16:27:49  
        LastHeartbeat: 2025-09-01 17:08:36  
                Alive: true  
 SystemDecommissioned: false  
ClusterDecommissioned: false  
            TabletNum: 3755  
     DataUsedCapacity: 605.218 MB  
        AvailCapacity: 13.919 GB  
        TotalCapacity: 29.485 GB  
              UsedPct: 52.79 %  
       MaxDiskUsedPct: 52.79 %  
               ErrMsg:   
              Version: 3.3.17-3eac4a9  
               Status: {"lastSuccessReportTabletsTime":"2025-09-01 17:07:56"}  
    DataTotalCapacity: 14.510 GB  
          DataUsedPct: 4.07 %  
             CpuCores: 4  
             MemLimit: 3.654GB  
    NumRunningQueries: 0  
           MemUsedPct: 13.55 %  
           CpuUsedPct: 20.2 %  
     DataCacheMetrics: Status: Normal, DiskUsage: 0B/0B, MemUsage: 0B/0B  
             Location: 
```

#### 2.1.2 生成大量 tablet

```
# 生成 大量 tablet  
CREATE TABLE partitioned_table (  
    event_day DATE,  
    id BIGINT  
)   
PARTITION BY RANGE(event_day) (  
    START ("2024-01-01") END ("2025-01-01") EVERY (INTERVAL 1 DAY)  
)   
DISTRIBUTED BY HASH(id) BUCKETS 10; -- 会生成 366*10 个 tablet
```

#### 2.1.3 建表后

```
#建表后、当前tablet数量  
mysql> show backends\G  
*************************** 1. row ***************************  
            BackendId: 10002  
                   IP: 10.197.165.201  
        HeartbeatPort: 9050  
               BePort: 9060  
             HttpPort: 8040  
             BrpcPort: 8060  
        LastStartTime: 2025-09-01 16:27:49  
        LastHeartbeat: 2025-09-01 17:09:56  
                Alive: true  
 SystemDecommissioned: false  
ClusterDecommissioned: false  
            TabletNum: 7415  
     DataUsedCapacity: 605.218 MB  
        AvailCapacity: 13.919 GB  
        TotalCapacity: 29.485 GB  
              UsedPct: 52.79 %  
       MaxDiskUsedPct: 52.79 %  
               ErrMsg:   
              Version: 3.3.17-3eac4a9  
               Status: {"lastSuccessReportTabletsTime":"2025-09-01 17:09:56"}  
    DataTotalCapacity: 14.510 GB  
          DataUsedPct: 4.07 %  
             CpuCores: 4  
             MemLimit: 3.654GB  
    NumRunningQueries: 0  
           MemUsedPct: 14.07 %  
           CpuUsedPct: 0.7 %  
     DataCacheMetrics: Status: Normal, DiskUsage: 0B/0B, MemUsage: 0B/0B  
             Location: 
```

#### 2.1.4 删除表

```
# 这里可以看到tablet 还未删除，是由于SR 默认保留一天的数据（catalog_trash_expire_second：默认 86400 秒）  
mysql>  drop table db2.partitioned_table;  
Query OK, 0 rows affected (0.06 sec)  
  
mysql> show backends\G  
*************************** 1. row ***************************  
            BackendId: 10002  
                   IP: 10.197.165.201  
        HeartbeatPort: 9050  
               BePort: 9060  
             HttpPort: 8040  
             BrpcPort: 8060  
        LastStartTime: 2025-09-01 16:27:49  
        LastHeartbeat: 2025-09-01 17:10:16  
                Alive: true  
 SystemDecommissioned: false  
ClusterDecommissioned: false  
            TabletNum: 7415  
     DataUsedCapacity: 605.218 MB  
        AvailCapacity: 13.919 GB  
        TotalCapacity: 29.485 GB  
              UsedPct: 52.79 %  
       MaxDiskUsedPct: 52.79 %  
               ErrMsg:   
              Version: 3.3.17-3eac4a9  
               Status: {"lastSuccessReportTabletsTime":"2025-09-01 17:09:56"}  
    DataTotalCapacity: 14.510 GB  
          DataUsedPct: 4.07 %  
             CpuCores: 4  
             MemLimit: 3.654GB  
    NumRunningQueries: 0  
           MemUsedPct: 14.07 %  
           CpuUsedPct: 0.2 %  
     DataCacheMetrics: Status: Normal, DiskUsage: 0B/0B, MemUsage: 0B/0B  
             Location:   
               
# 此处临时修改一下catalog_trash_expire_second  
mysql> ADMIN SHOW FRONTEND CONFIG like 'catalog_trash_expire_second';  
+-----------------------------+------------+-------+------+-----------+---------+  
| Key                         | AliasNames | Value | Type | IsMutable | Comment |  
+-----------------------------+------------+-------+------+-----------+---------+  
| catalog_trash_expire_second | []         | 86400 | long | true      |         |  
+-----------------------------+------------+-------+------+-----------+---------+  
1 row inset (0.00 sec)  
  
mysql> ADMIN SET FRONTEND CONFIG ("catalog_trash_expire_second" = "0");  
Query OK, 0 rows affected (0.05 sec)  
  
mysql> ADMIN SHOW FRONTEND CONFIG like 'catalog_trash_expire_second';  
+-----------------------------+------------+-------+------+-----------+---------+  
| Key                         | AliasNames | Value | Type | IsMutable | Comment |  
+-----------------------------+------------+-------+------+-----------+---------+  
| catalog_trash_expire_second | []         | 0     | long | true      |         |  
+-----------------------------+------------+-------+------+-----------+---------+  
1 row inset (0.00 sec)  
  
#可以看到，tablet立马就删除了  
mysql> show backends\G  
*************************** 1. row ***************************  
            BackendId: 10002  
                   IP: 10.197.165.201  
        HeartbeatPort: 9050  
               BePort: 9060  
             HttpPort: 8040  
             BrpcPort: 8060  
        LastStartTime: 2025-09-01 16:27:49  
        LastHeartbeat: 2025-09-01 17:10:51  
                Alive: true  
 SystemDecommissioned: false  
ClusterDecommissioned: false  
            TabletNum: 3755  
     DataUsedCapacity: 605.218 MB  
        AvailCapacity: 13.913 GB  
        TotalCapacity: 29.485 GB  
              UsedPct: 52.81 %  
       MaxDiskUsedPct: 52.81 %  
               ErrMsg:   
              Version: 3.3.17-3eac4a9  
               Status: {"lastSuccessReportTabletsTime":"2025-09-01 17:09:56"}  
    DataTotalCapacity: 14.504 GB  
          DataUsedPct: 4.08 %  
             CpuCores: 4  
             MemLimit: 3.654GB  
    NumRunningQueries: 0  
           MemUsedPct: 14.06 %  
           CpuUsedPct: 0.0 %  
     DataCacheMetrics: Status: Normal, DiskUsage: 0B/0B, MemUsage: 0B/0B  
             Location: 
```

### 2.2 tablet 对应物理文件

#### 2.2.1 物理文件说明

* • 对应到BE节点上物理文件的存储，tablet对应的存储路径是在 be.conf 里的 storage\_root\_path 配置的目录下，相关的 tablet 数据就存在 ${storage\_root\_path}/data 下
* • tablet是逻辑分片概念，具体到物理文件存储则是segment文件，文件以及目录层级如下

```
[root@zyf-sr-02 1374885024]# du -sh *  
1022M   0200000000003645be4fec146c4c1883227773a343c970af_0.dat（rowset_id + "_" + segment_id + ".dat"）  
178M    0200000000003645be4fec146c4c1883227773a343c970af_1.dat  
[root@zyf-sr-02 1374885024]# pwd  
/data/other/starrocks/data/data/374（shard_id）/371281（tablet id）/1374885024（schema_hash）
```

#### 2.2.2 建表测试

##### 2.2.2.1 PG 数据库生成数据

```
#PG数据库生成源数据  
drop table if exists tbl1;    
create table tbl1 (    
id int primary key,    
  info text,    
  c1 int,    
  c2 float,    
  ts timestamp    
);     
insert into tbl1 selectid,md5(random()::text),random()*1000,random()*100,clock_timestamp() from generate_series(1,60000000) id;    
\dt+ tbl1  
                    List of relations  
 Schema | Name | Type  |  Owner   |  Size   | Description   
--------+------+-------+----------+---------+-------------  
 public | tbl1 | table | postgres | 2664 MB |   
(1 row)  
copy tbl1 to '/tmp/tbl1.csv' delimiter ',';
```

##### 2.2.2.2 SR 建表入数（智能分桶）

```
#这里我们只创建一个副本的数据，方便观察  
create table tbl1 (id int ,info text,c1 int,c2 float,ts datetime) PROPERTIES ("replication_num" = "1");  
#数据导入  
curl --location-trusted -u root:root -H "column_separator:," \  
    -H "Expect:100-continue" \  
    -H "columns:id,info,c1,c2,ts" \  
    -T /tmp/tbl1.csv -XPUT \  
    http://10.197.165.201:8030/api/db2/tbl1/_stream_load
```

##### 2.2.2.3 观察数据文件

```
#首先查询表有哪些tablet（可以看到系统智能分桶分了 3 个 tablet）  
SHOW tablet from db2.tbl1\G  
*************************** 1. row ***************************  
               TabletId: 556040  
              ReplicaId: 556041  
              BackendId: 10002  
             SchemaHash: 0  
                Version: 2  
            VersionHash: 0  
      LstSuccessVersion: 2  
  LstSuccessVersionHash: 0  
       LstFailedVersion: -1  
   LstFailedVersionHash: 0  
          LstFailedTime: NULL  
               DataSize: 403.5MB  
               RowCount: 10000018  
                  State: NORMAL  
LstConsistencyCheckTime: NULL  
           CheckVersion: -1  
       CheckVersionHash: 0  
           VersionCount: 1  
               PathHash: -1  
                MetaUrl: http://10.197.165.201:8040/api/meta/header/556040  
       CompactionStatus: http://10.197.165.201:8040/api/compaction/show?tablet_id=556040  
           DiskRootPath: Unknown  
           IsConsistent: true  
               Checksum: -1  
*************************** 2. row ***************************  
               TabletId: 556042  
              ReplicaId: 556043  
              BackendId: 10170  
             SchemaHash: 0  
                Version: 2  
            VersionHash: 0  
      LstSuccessVersion: 2  
  LstSuccessVersionHash: 0  
       LstFailedVersion: -1  
   LstFailedVersionHash: 0  
          LstFailedTime: NULL  
               DataSize: 403.4MB  
               RowCount: 9999967  
                  State: NORMAL  
LstConsistencyCheckTime: NULL  
           CheckVersion: -1  
       CheckVersionHash: 0  
           VersionCount: 1  
               PathHash: -1  
                MetaUrl: http://10.197.165.202:8040/api/meta/header/556042  
       CompactionStatus: http://10.197.165.202:8040/api/compaction/show?tablet_id=556042  
           DiskRootPath: Unknown  
           IsConsistent: true  
               Checksum: -1  
*************************** 3. row ***************************  
               TabletId: 556044  
              ReplicaId: 556045  
              BackendId: 10171  
             SchemaHash: 0  
                Version: 2  
            VersionHash: 0  
      LstSuccessVersion: 2  
  LstSuccessVersionHash: 0  
       LstFailedVersion: -1  
   LstFailedVersionHash: 0  
          LstFailedTime: NULL  
               DataSize: 403.4MB  
               RowCount: 10000015  
                  State: NORMAL  
LstConsistencyCheckTime: NULL  
           CheckVersion: -1  
       CheckVersionHash: 0  
           VersionCount: 1  
               PathHash: -1  
                MetaUrl: http://10.197.165.203:8040/api/meta/header/556044  
       CompactionStatus: http://10.197.165.203:8040/api/compaction/show?tablet_id=556044  
           DiskRootPath: Unknown  
           IsConsistent: true  
               Checksum: -1  
3 rows inset (0.00 sec)  
  
#查找一下对应的数据文件（需要在对应的BackendId10002查找对应的556040）  
[root@zyf-sr-01 ~]# find / -name 556040  
/data/other/starrocks/data/data/155/556040  
[root@zyf-sr-01 ~]# tree /data/other/starrocks/data/data/155/556040  
/data/other/starrocks/data/data/155/556040  
└── 1374885024  
    ├── 0200000000001cc8e540263d189634598cee6af6d99e56af_0.dat  
    ├── 0200000000001cc8e540263d189634598cee6af6d99e56af_1.dat  
    ├── 0200000000001cc8e540263d189634598cee6af6d99e56af_2.dat  
    ├── 0200000000001cc8e540263d189634598cee6af6d99e56af_3.dat  
    ├── 0200000000001cc8e540263d189634598cee6af6d99e56af_4.dat  
    ├── 0200000000001cc8e540263d189634598cee6af6d99e56af_5.dat  
    └── 0200000000001cc9e540263d189634598cee6af6d99e56af_0.dat  
  
1 directory, 7 files  
[root@zyf-sr-01 ~]# cd /data/other/starrocks/data/data/155/556040  
[root@zyf-sr-01 556040]# cd 1374885024/  
[root@zyf-sr-01 1374885024]#  pwd  
/data/other/starrocks/data/data/155/556040/1374885024  
[root@zyf-sr-01 1374885024]# du -sh *  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_0.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_1.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_2.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_3.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_4.dat  
57M     0200000000001cc8e540263d189634598cee6af6d99e56af_5.dat  
404M    0200000000001cc9e540263d189634598cee6af6d99e56af_0.dat
```

##### 2.2.2.4 版本合并

```
#进行第二次数据导入、可以看到数据文件增加了  
[root@zyf-sr-01 1374885024]# du -sh *  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_0.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_1.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_2.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_3.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_4.dat  
57M     0200000000001cc8e540263d189634598cee6af6d99e56af_5.dat  
404M    0200000000001cc9e540263d189634598cee6af6d99e56af_0.dat  
70M     0200000000001d3ae540263d189634598cee6af6d99e56af_0.dat  
70M     0200000000001d3ae540263d189634598cee6af6d99e56af_1.dat  
70M     0200000000001d3ae540263d189634598cee6af6d99e56af_2.dat  
70M     0200000000001d3ae540263d189634598cee6af6d99e56af_3.dat  
70M     0200000000001d3ae540263d189634598cee6af6d99e56af_4.dat  
57M     0200000000001d3ae540263d189634598cee6af6d99e56af_5.dat  
0       0200000000001d3be540263d189634598cee6af6d99e56af_0.dat  
#在多次stream导入命令执行完成后，show tablet from db2.tbl2;信息还未更新，生成的文件比较碎片，执行如下命令对版本进行合并，会将小文件合并成大文件，但是原来的文件并不会删除。  
ALTER TABLE db2.tbl1 COMPACT;  
#可以看到文件进行了合并，但是原来的文件还未进行删除  
[root@zyf-sr-01 1374885024]# du -sh *  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_0.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_1.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_2.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_3.dat  
70M     0200000000001cc8e540263d189634598cee6af6d99e56af_4.dat  
57M     0200000000001cc8e540263d189634598cee6af6d99e56af_5.dat  
404M    0200000000001cc9e540263d189634598cee6af6d99e56af_0.dat  
70M     0200000000001d3ae540263d189634598cee6af6d99e56af_0.dat  
70M     0200000000001d3ae540263d189634598cee6af6d99e56af_1.dat  
70M     0200000000001d3ae540263d189634598cee6af6d99e56af_2.dat  
70M     0200000000001d3ae540263d189634598cee6af6d99e56af_3.dat  
70M     0200000000001d3ae540263d189634598cee6af6d99e56af_4.dat  
57M     0200000000001d3ae540263d189634598cee6af6d99e56af_5.dat  
723M    0200000000001d3be540263d189634598cee6af6d99e56af_0.dat
```

##### 2.2.2.5 SR 建表（手动分桶）

```
CREATE TABLE tbl3 (id int ,info text,c1 int,c2 float,ts datetime) DISTRIBUTED BY RANDOM BUCKETS 1  PROPERTIES ("replication_num" = "1");  
#可以看到这边只会生成一个 tablet  
mysql> show tablet from db2.tbl3\G  
*************************** 1. row ***************************  
               TabletId: 556065  
              ReplicaId: 556066  
              BackendId: 10002  
             SchemaHash: 0  
                Version: 1  
            VersionHash: 0  
      LstSuccessVersion: 1  
  LstSuccessVersionHash: 0  
       LstFailedVersion: -1  
   LstFailedVersionHash: 0  
          LstFailedTime: NULL  
               DataSize: 0B  
               RowCount: 0  
                  State: NORMAL  
LstConsistencyCheckTime: NULL  
           CheckVersion: -1  
       CheckVersionHash: 0  
           VersionCount: -1  
               PathHash: -1  
                MetaUrl: http://10.197.165.201:8040/api/meta/header/556065  
       CompactionStatus: http://10.197.165.201:8040/api/compaction/show?tablet_id=556065  
           DiskRootPath: Unknown  
           IsConsistent: true  
               Checksum: -1  
1 row inset (0.00 sec)
```

## 3 数据分桶

### 3.1 分桶说明

* • 建表时，您可以通过设置合理的分区和分桶，实现数据均匀分布和查询性能提升。数据均匀分布是指数据按照一定规则划分为子集，并且均衡地分布在不同节点上。查询时能够有效裁剪数据扫描量，最大限度地利用集群的并发性能，从而提升查询性能。
* • 自 2.5.7 版本起，您在建表和新增分区时可以不设置分桶数量 (BUCKETS)。StarRocks 默认自动设置分桶数量，如果自动设置分桶数量后性能未能达到预期，并且您比较熟悉分桶机制，则您也可以[手动设置分桶数量
* • 分桶是实际物理文件组织的单元
* • Tablet 过多会增加 FE/BE 的元数据管理和调度的资源消耗
* • 分区和分桶应该尽量覆盖查询语句所带的条件，这样可以有效减少扫描数据，提高并发
* • 查看分桶数量：SHOW PARTITIONS
* • 手动设置分桶

+ • 如果表单个分区原始数据规模预计超过 100 GB，建议您手动设置分区中分桶数量
+ • 每10GB一个Tablet：DISTRIBUTED BY HASH(site\_id,city\_code) BUCKETS 30; -- 假设导入一个分区的原始数据量为 300 GB，则按照每 10 GB 原始数据一个 Tablet，则分区中分桶数量可以设置为 30。
+ • 如果需要开启并行扫描 Tablet，则您需要确保系统变量 enable\_tablet\_internal\_parallel 全局生效 SET GLOBAL enable\_tablet\_internal\_parallel = true;
+ •

  ```
  #建表时  
  CREATE TABLE site_access (  
      site_id INT DEFAULT '10',  
      city_code SMALLINT,  
      user_name VARCHAR(32) DEFAULT '',  
      event_day DATE,  
      pv BIGINT SUM DEFAULT '0')  
  AGGREGATE KEY(site_id, city_code, user_name,event_day)  
  PARTITION BY date_trunc('day', event_day)  
  DISTRIBUTED BY HASH(site_id,city_code); -- 无需手动设置分区中分桶数量  
  #建表后  
  -- 自动设置所有分区的分桶数量，并且不开启按需动态增加分桶数量，分桶数量固定。  
  ALTER TABLE details DISTRIBUTED BY RANDOM;  
  -- 自动设置所有分区的分桶数量，并且开启按需动态增加分桶数量。  
  ALTER TABLE details SET("bucket_size"="1073741824");  
  -- 自动设置指定分区的分桶数量  
  ALTER TABLE details PARTITIONS (p20230103, p20230104)  
  DISTRIBUTED BY RANDOM;  
  -- 自动设置新增分区中分桶数量  
  ALTER TABLE details ADD PARTITION  p20230106 VALUES [('2023-01-06'), ('2023-01-07'))  
  DISTRIBUTED BY RANDOM;
  ```

### 3.2 随机分桶

* • 自 3.1 版本起，您在建表和新增分区时可以不设置分桶键（即 DISTRIBUTED BY 子句）。StarRocks 默认使用随机分桶，将数据随机地分布在分区的所有分桶中
* • 不过值得注意的是，如果查询海量数据且查询时经常使用一些列会作为条件列，随机分桶提供的查询性能可能不够理想。建议使用哈希分桶，当查询时经常使用这些列作为条件列时，只需要扫描和计算查询命中的少量分桶，则可以显著提高查询性能。
* • 使用限制

+ • 仅支持明细表
+ • 不支持指定 Colocation Group
+ • 不支持 Spark Load

### 3.3 哈希分桶

* • 哈希分桶

+ • 根据数据的分桶键值，将数据划分至分桶。选择查询时经常使用的条件列组成分桶键，能有效提高查询效率。
+ • 列同时满足高基数（唯一值较多）、作为查询条件，则进行哈希分桶

* • 优点

+ • 提高查询性能：相同分桶键值的行会被分配到一个分桶中，在查询时能减少扫描数据量
+ • 均匀分布数据：选择高基数的列为分桶键，保证数据在各个分桶中尽量均衡

* • 如何选择分桶键

+ • 如果查询比较复杂，则建议选择高基数的列为分桶键，保证数据在各个分桶中尽量均衡，提高集群资源利用率
+ • 如果查询比较简单，则建议选择经常作为查询条件的列为分桶键，提高查询效率。
+ • 并且，如果数据倾斜情况严重，您还可以使用多个列作为数据的分桶键，但是建议不超过 3 个列

* • 注意事项

+ • 分桶键仅支持：整型、DECIMAL、DATE/DATETIME、CHAR/VARCHAR/STRING 数据类型
+ • 自 3.2 起，建表后支持通过 ALTER TABLE 修改分桶键
+ • 主键表修改分桶键如果不包含主键，SQL 执行正常，但是show alter table optimize\G 会提示：create temp partitions failed java.lang.RuntimeException: create partitions failed: Distribution column[id2] is not key column