> 已吸收至：[[03_数据工程与数仓/0308_元数据血缘与治理/030801_元数据平台/030801_核心知识点/元数据持久化与统计口径边界|元数据持久化与统计口径边界]]
---
title: 基于Hive进行数仓建设的资源元数据信息统计
author: 智海观潮
date:
url: http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484806&idx=1&sn=fec69b9028ba9f98e50685217b1ab394&chksm=e976f9bcde0170aa51f0a968cabb96ccc4678d305bcba8e18ec900a36389df03495205b806b7&mpshare=1&scene=24&srcid=0525bxkfGqUgy1HqpcNOk8Cg&sharer_sharetime=1684978441901&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

在数据仓库建设中，元数据管理是非常重要的环节之一。根据Kimball的数据仓库理论，可以将元数据分为这三类：

1. **技术元数据**，如表的存储结构结构、文件的路径
2. **业务元数据**，如血缘关系、业务的归属
3. **过程元数据**，如表每天的行数、占用HDFS空间、更新时间

而基于这3类元数据"搭建"起来的元数据系统，通常又会实现如下核心功能：

1. **血缘关系**

   如表级别/字段级别的血缘关系，这些主要体现在我们日常的SQL和ETL任务里。
2. **大数据集群计算资源管理**

   针对利用不同的计算引擎如Spark/Flink/Mapreduce，可以到Yarn（也可能是其他资源管理器）上采集相关任务的使用情况。如CPU、内存、磁盘IO使用情况。

   然后可以把这些资源使用情况绘制成图。通过可视化界面可以直观发现某些任务中的异常情况，以及发现某些严重消耗资源的表或业务，及时通知相关负责人有针对性的分析处理和优化。
3. **数据如何同步以及权限管理****等**
4. **Hive库表元数据信息统计**

   这里对Hive库表统计信息主要是指：行数、文件数、所占HDFS存储大小、最后一次操作时间等。

   通过持续不断的采集这些指标，形成可视化曲线图，数据仓库相关人员都可以从这个图中发现数据规律或数据质量问题。对于利用数仓进行业务开发的人员，可以通过这些曲线图来分析业务量变化趋势。在此基础之上，还可以做数据质量校验、数值分布探查等功能。

本文主要介绍如何利用Hive和Spark进行对Hive库、分区表/非分区表相关指标的统计。

而在我们实际生产中，我们不仅可以通过如下的方式及时更新和获取Hive元数据库中相关表记录的指标信息，我们也可以参考下述相关SQL在Hive/Spark底层的执行过程，实现我们自己的一整套业务逻辑。

**1. Hive元数据库中主要涉及的元数据表**

```
DBS：存储Hive中所有数据库的基本信息，如库ID、表ID、创建时间、用户、表名、表的类型等。TBS：存储Hive表、视图等的基本信息，如表ID、表名、创建时间、用户、表类型等。TABLE_PARAMS：存储表等的属性信息，表ID、PARAM_KEY（如EXTERNAL）、PARAM_VALUE（与PARAM_KEY对应的值）。PARTITIONS：存储Hive分区统计信息相关的元数据，如分区ID、表ID、创建时间、分区名（partCol=partVal）等信息。PARTITION_PARAMS：存储Hive分区统计信息相关的元数据，如分区ID、PARAM_KEY（如文件数）、PARAM_VALUE（与PARAM_KEY对应的值）。
```

**2. Hive和Spark支持的Hive库表元数据信息统计**

**2.1 Hive**

**2.1.1 语法支持**

默认情况下，在对Hive表进行数据insert时，会自动更新元数据库表中的统计信息，但主要是文件数、占用HDFS空间大小等，不包括行数。

1）分区表

Hive分区表元数据统计信息SQL语法需要指定到具体分区，如分区字段或者分区名=分区值

```
-- 1. 统计更新tab_partition的分区字段为dt的所有元数据信息analyze table tab_partition partition(dt) COMPUTE STATISTICS;
-- 2. 统计更新单个分区元数据统计信息analyze table tab_partition partition(dt='20200722000000') COMPUTE STATISTICS;
```

在Hive shell中执行analyze时，如果进行了元数据信息统计会打印类似如下信息：

```
Partition default.test_partition2{dt=20200718000000} stats: [numFiles=1, numRows=2, totalSize=418, rawDataSize=6]
```

2）非分区表

```
-- 非分区表粒度到表analyze table tab_no_partition COMPUTE STATISTICS;
```

**2.1.2 Hive元数据库中涉及的元数据统计信息字段**

1）Hive分区表

```
-- 表级别：TABLE_PARAMS-- Hive分区级别：PARTITION_PARAMS
numFiles：文件数numRows：行数totalSize：占用HDFS空间大小rawDataSize：原生数据大小transient_lastDdlTime：最近一次操作时间
```

2）Hive非分区表

对于Hive分区表，因为最小粒度是表级别。因此，元数据统计信息也是表级别的。

```
-- TABLE_PARAMSnumFiles、numRows、totalSize、rawDataSize、transient_lastDdlTime：含义同上
```

**2.2 Spark**

注意：Spark默认不统计文件数

**2.2.1 语法支持**

1）分区表

Spark对Hive分区表元数据统计，跟Hive原生对分区表的统计支持略有不同。

Spark既支持具体到分区的元数据信息统计，也支持整个表级别的元数据信息统计（但不会对具体分区做处理）

```
-- 统计tab_partition数据所占HDFS空间总大小和总行数。-- Hive目前不支持直接这样解析分区表-- 注意：执行该SQL不会处理表中具体分区统计信息analyze table tab_partition COMPUTE STATISTICS;
-- 同Hiveanalyze table tab_partition partition(partCol) COMPUTE STATISTICS;
-- 同Hiveanalyze table tab_partition partition(partCol='20200722000000') COMPUTE STATISTICS;
```

2）非分区表

```
analyze table tab_no_partition COMPUTE STATISTICS;
```

**2.2.2 Hive元数据库中涉及的元数据统计信息字段**

1）Hive分区表

```
-- 表级别：TABLE_PARAMS-- Hive分区级别：PARTITION_PARAMS
spark.sql.statistics.numRows：文件数（同Hive统计中的numRows，但不会更新Hive的统计信息）spark.sql.statistics.totalSize：行数（同Hive统计中的totalSize，但不会更新Hive的统计信息）transient_lastDdlTime：同Hive
```

2）Hive非分区表

```
-- 统计级别同Hive，TABLE_PARAMSspark.sql.statistics.numRows、spark.sql.statistics.totalSize、transient_lastDdlTime：含义同上
```

**3. Hive和Spark对Hive库表元数据信息统计的主要区别**

1. 对Hive表元数据信息统计的SQL语法支持不同

   如Spark支持对Hive分区表进行表级别的统计，但Hive需要指定到具体分区
2. 对Hive表元数据信息统计在Hive元数据库中的体现不同

   如同样是行数，Hive用numRows，而Spark用spark.sql.statistics.numRows
3. Spark默认不统计文件数，但Hive统计

Hive和Spark对Hive库表元数据信息统计的区别包括但不限于以上3种区别。具体的看之前的介绍，以及通过下面以Hive分区表为例，看看主要的具体细节：

**3.1 Hive**

默认情况下，在对Hive表进行数据insert时，Hive会自动更新元数据统计信息，但是不统计行数。如需获取numRow，可以再次执行analyze SQL

1）直接通过Hive进行表的创建

以分区表testdb.test\_analyze为例，表刚创建时Hive元数据库中表TABLE\_PARAMS的信息：

```
+------+---------------------+-----------+|TBL_ID|           PARAM_KEY |PARAM_VALUE|+------+---------------------+-----------+|  3016|            EXTERNAL |       TRUE||  3016|transient_lastDdlTime| 1595405772|+------+---------------------+-----------+
```

2）对表testdb.test\_analyze进行数据的保存和元数据信息统计：

```
insert overwrite table testdb.test_analyze partition(partCol=20200721000000) select id,name from testdb.test_partition1 where partCol=20190626000000;
analyze table testdb.test_analyze partition(partCol='20200721000000') COMPUTE STATISTICS;
```

3）连接Hive元数据库，查询testdb.test\_analyze的元数据统计信息

```
-- 1. 连接Hive元数据库connect jdbc whereurl="jdbc:mysql://localhost:3306/hive?useUnicode=true&amp;characterEncoding=UTF-8"and driver="com.mysql.jdbc.Driver"and user="root"and password="root"as db_1;
-- 2. 将TABLE_PARAMS、DBS、TBLS、PARTITIONS、PARTITION_PARAMS注册为临时表
-- load jdbc.`db_1.TABLE_PARAMS` as TABLE_PARAMS ;load jdbc.`db_1.DBS` as dbs;load jdbc.`db_1.TBLS` as tbls;load jdbc.`db_1.PARTITIONS` as partitions;load jdbc.`db_1.PARTITION_PARAMS` as partition_params;
-- 3. 获取testdb.test_analyze的元数据统计信息select d.NAME,t.TBL_NAME,t.TBL_ID,p.PART_ID,p.PART_NAME,a.*   from tbls t   left join dbs d  on t.DB_ID = d.DB_ID  left join partitions p  on t.TBL_ID = p.TBL_ID   left join partition_params a  on p.PART_ID=a.PART_IDwhere t.TBL_NAME='test_analyze' and d.NAME='testdb';
```

4）结果

```
-- 测试时，testdb.test_analyze只有partCol=20200721000000的分区。因此，统计信息也只有partCol=20200721000000的
+------+------------+------+-------+----------------------+-------+--------------------+--------------------+|  NAME|    TBL_NAME|TBL_ID|PART_ID|             PART_NAME|PART_ID|           PARAM_KEY|         PARAM_VALUE|+------+------------+------+-------+----------------------+-------+--------------------+--------------------+|testdb|test_analyze|  3016|  52976|partCol=20200721000000|  52976|COLUMN_STATS_ACCU...|{"BASIC_STATS":"t...||testdb|test_analyze|  3016|  52976|partCol=20200721000000|  52976|            numFiles|                   1||testdb|test_analyze|  3016|  52976|partCol=20200721000000|  52976|             numRows|                   1||testdb|test_analyze|  3016|  52976|partCol=20200721000000|  52976|         rawDataSize|                   3||testdb|test_analyze|  3016|  52976|partCol=20200721000000|  52976|           totalSize|                 383||testdb|test_analyze|  3016|  52976|partCol=20200721000000|  52976|transient_lastDdl...|          1595407507|+------+------------+------+-------+----------------------+-------+--------------------+--------------------+
```

**3.2 Spark**

1）通过Spark创建Hive表

以分区表testdb.test\_analyze\_spark为例，表刚创建时Hive元数据库中表TABLE\_PARAMS的信息：

```
+------+------------------------------------+--------------------+|TBL_ID|                           PARAM_KEY|         PARAM_VALUE|+------+------------------------------------+--------------------+|  3018|                            EXTERNAL|                TRUE||  3018|            spark.sql.create.version|               2.4.3||  3018|spark.sql.sources.schema.numPartCols|                   1||  3018|   spark.sql.sources.schema.numParts|                   1||  3018|     spark.sql.sources.schema.part.0|{"type":"struct",...||  3018|  spark.sql.sources.schema.partCol.0|                  dt||  3018|               transient_lastDdlTime|          1595409374|+------+------------------------------------+--------------------+
```

2）对表testdb.test\_analyze进行数据的保存和元数据信息统计

```
insert overwrite table testdb.test_analyze partition(partCol=20200721000000) select id,name from testdb.test_partition1 where partCol=20190626000000;
```

执行上述SQL后，Hive内部会启动一个任务进行Hive表操作的分区元数据信息统计，但是没有numRows。如下：

```
+------+------------------+------+-------+----------------------+-------+--------------------+-----------+|  NAME|          TBL_NAME|TBL_ID|PART_ID|             PART_NAME|PART_ID|           PARAM_KEY|PARAM_VALUE|+------+------------------+------+-------+----------------------+-------+--------------------+-----------+|testdb|test_analyze_spark|  3018|  52977|partCol=20200721000000|  52977|            numFiles|          1||testdb|test_analyze_spark|  3018|  52977|partCol=20200721000000|  52977|           totalSize|        389||testdb|test_analyze_spark|  3018|  52977|partCol=20200721000000|  52977|transient_lastDdl...| 1595409909|+------+------------------+------+-------+----------------------+-------+--------------------+-----------+
```

3）连接Hive元数据库，查询testdb.test\_analyze\_spark的元数据统计信息

```
connect jdbc whereurl="jdbc:mysql://localhost:3306/hive?useUnicode=true&amp;characterEncoding=UTF-8" and driver="com.mysql.jdbc.Driver" and user="root" and password="root" as db_1;
-- load jdbc.`db_1.TABLE_PARAMS` as TABLE_PARAMS ;load jdbc.`db_1.TBLS` as tbls;load jdbc.`db_1.DBS` as dbs;load jdbc.`db_1.PARTITIONS` as partitions;load jdbc.`db_1.PARTITION_PARAMS` as partition_params;
select d.NAME,t.TBL_NAME,t.TBL_ID,p.PART_ID,p.PART_NAME,a.*   from tbls t   left join dbs d  on t.DB_ID = d.DB_ID  left join partitions p  on t.TBL_ID = p.TBL_ID   left join partition_params a  on p.PART_ID=a.PART_IDwhere t.TBL_NAME='test_analyze_spark' and d.NAME='testdb' ;
```

4）结果

```
-- Spark在执行analyze table mlsql_test.test_analyze_spark partition(dt='20200721000000') COMPUTE STATISTICS; 时，会对分区行数进行统计：+------+------------------+------+-------+----------------------+-------+-------------------------------+-----------+|  NAME|          TBL_NAME|TBL_ID|PART_ID|             PART_NAME|PART_ID|                      PARAM_KEY|PARAM_VALUE|+------+------------------+------+-------+----------------------+-------+-------------------------------+-----------+|testdb|test_analyze_spark|  3018|  52977|partCol=20200721000000|  52977|                       numFiles|          1||testdb|test_analyze_spark|  3018|  52977|partCol=20200721000000|  52977|   spark.sql.statistics.numRows|          1||testdb|test_analyze_spark|  3018|  52977|partCol=20200721000000|  52977| spark.sql.statistics.totalSize|        389||testdb|test_analyze_spark|  3018|  52977|partCol=20200721000000|  52977|                      totalSize|        389||testdb|test_analyze_spark|  3018|  52977|partCol=20200721000000|  52977|          transient_lastDdlTime| 1595410238|+------+------------------+------+-------+----------------------+-------+-------------------------------+-----------+
```

5）通过Spark对整个Hive分区表元数据信息的统计

```
-- 1. 执行：analyze table testdb.test_analyze_spark COMPUTE STATISTICS;-- 2. Hive元数据库中表TABLE_PARAMS的包含的testdb.test_analyze_spark信息：
connect jdbc where url="jdbc:mysql://localhost:3306/hive?useUnicode=true&amp;characterEncoding=UTF-8" and driver="com.mysql.jdbc.Driver" and user="root" and password="root" as db_1;
-- 获取mlsql_test的DB_ID（49）load jdbc.`db_1.DBS` as dbs;select DB_ID from dbs where NAME='testdb' as db;
-- 获取test_analyze_spark的TBL_ID（3018）load jdbc.`db_1.TBLS` as tbls;select TBL_ID from tbls where DB_ID=49 and TBL_NAME='test_analyze_spark' as t2;
-- 获取testdb.test_analyze_spark表级别统计信息load jdbc.`db_1.TABLE_PARAMS` as TABLE_PARAMS ;select * from TABLE_PARAMS where TBL_ID=3018 ;
-- 结果+------+------------------------------------+--------------------+|TBL_ID|                           PARAM_KEY|         PARAM_VALUE|+------+------------------------------------+--------------------+|  3018|                            EXTERNAL|                TRUE||  3018|            spark.sql.create.version|               2.4.3||  3018|spark.sql.sources.schema.numPartCols|                   1||  3018|   spark.sql.sources.schema.numParts|                   1||  3018|     spark.sql.sources.schema.part.0|{"type":"struct",...||  3018|  spark.sql.sources.schema.partCol.0|                  partCol||  3018|        spark.sql.statistics.numRows|                   1||  3018|      spark.sql.statistics.totalSize|                 389||  3018|               transient_lastDdlTime|          1595410958|+------+------------------------------------+--------------------+
```

推荐文章：

[监听MySQL的binlog日志工具分析：Canal、Maxwell](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484788&idx=1&sn=d445a368e700743bd282791add1da01c&chksm=e976f94ede01705882f29d1f491a3162f38bccb335d935172662f4102cf7076fb60d0b80b21f&scene=21#wechat_redirect)

[Hive Query生命周期 —— 钩子（Hook）函数篇](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484739&idx=1&sn=24a1967ad44d898b45e659d154859c60&chksm=e976f979de01706f92b849e3bad839944d07c7bac7fe82f0147dd76fc04cab21b6117ead67b6&scene=21#wechat_redirect)

[Hive实现自增序列及元数据问题](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484649&idx=1&sn=4fd912db299cd8f989a05ba3f741bcf2&chksm=e976f8d3de0171c5c304dbb920358f0fa811101e1d676d2147231f1b8cb1a4666e29fae34445&scene=21#wechat_redirect)

[SparkSQL与Hive metastore Parquet转换](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484333&idx=1&sn=a47a4c52b22e8ee9224c5ae6d7231eef&chksm=e976ff97de017681810f13b919887e3db3cee3684f6408f26bd693fe80b65493d9a93d481296&scene=21#wechat_redirect)

[Hive数据导入HBase引起数据膨胀引发的思考](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484075&idx=1&sn=3dfcc8398f8a112a90bb02c183c55379&chksm=e976fe91de0177876fd094702d35480497e3e42dda932b79bf4378461c7f6d94281c8f66a0ff&scene=21#wechat_redirect)

[Hive Join优化](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247483888&idx=1&sn=e0b0feb48c570abde6ef1e7d63789172&chksm=e976fdcade0174dc6520c3b277d87b60e88190854a4496d53ea7b9e28304be45e912ebeb498c&scene=21#wechat_redirect)

[如何有效恢复误删的HDFS文件](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484750&idx=1&sn=fea4ab3adfaa0f20159e7d3c7d636399&chksm=e976f974de017062cc1e04d0b93f0141dddfec420cdf3141111adfc937bff1e85a1d36635e70&scene=21#wechat_redirect)

[Spark存储Parquet数据到Hive，对map、array、struct字段类型的处理](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247483978&idx=1&sn=e7bd84249ec451eed63bb29a47e339d3&chksm=e976fe70de017766e3f7877f57636c3453938825a4d5d42da16b4b1e81b024c4ecbe315c419e&scene=21#wechat_redirect)

[Hadoop支持的压缩格式对比和应用场景以及Hadoop native库](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484466&idx=1&sn=44e383f8aca0e0707132bae1a8a60b48&chksm=e976f808de01711ee939719bc836ebc8da1d3f8738d8c35b2c0c9efa1219af82a432846faec9&scene=21#wechat_redirect)

---

关注**大数据学习与分享**，获取更多技术干货