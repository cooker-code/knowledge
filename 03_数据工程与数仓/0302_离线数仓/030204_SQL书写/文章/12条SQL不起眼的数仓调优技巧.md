---
title: 12条SQL不起眼的数仓调优技巧
author: 浪尖聊大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzUyMDA4OTY3MQ==&mid=2247510967&idx=2&sn=05d652e360037c0d1b1965bebfb172f5&chksm=f9ed529fce9adb89dedc42c7605baf5aeef66c9388d19160f31839e3c10f63e8873ec74939f8&mpshare=1&scene=24&srcid=0322odMfhX3v5yFvDURNlCW6&sharer_shareinfo=5fe3666912dc04ff4a29afce0d1d0191&sharer_shareinfo_first=5fe3666912dc04ff4a29afce0d1d0191#rd
---

作者：KubeData

Tips：在数据处理中，不怕数据量大，就怕数据倾斜（简单讲就是数据热点）

**01 请慎重使用COUNT（DISTINCT col）**

**问题原因：**

distinct会将b列所有的数据保存到内存中，形成一个类似hash的结构，速度是十分的块；但是在大数据背景下，因为b列所有的值都会形成以key值，极有可能发生OOM

**解决方案：**

所以，可以考虑使用Group By 或者 ROW\_NUMBER() OVER(PARTITION BY col)方式代替COUNT(DISTINCT col)

**02 小文件会造成资源的过度占用以及影响查询效率**

**问题原因：**

* 众所周知，小文件在HDFS中存储本身就会占用过多的内存空间，那么对于MR查询过程中过多的小文件又会造成启动过多的Mapper Task, 每个Mapper都是一个后台线程，会占用JVM的空间
* 在Hive中，动态分区会造成在插入数据过程中，生成过多零碎的小文件（请回忆昨天讲的动态分区的逻辑）
* 不合理的Reducer Task数量的设置也会造成小文件的生成，因为最终Reducer是将数据落地到HDFS中的
* Hive中分桶表的设置

**解决方案：**

在数据源头HDFS中控制小文件产生的个数，比如

* 采用Sequencefile作为表存储格式，不要用textfile，在一定程度上可以减少小文件（常见于在流计算的时候采用Sequencefile格式进行存储）
* 减少reduce的数量(可以使用参数进行控制)
* 慎重使用动态分区，最好在分区中指定分区字段的val值

最好数据的校验工作，比如通过脚本方式检测hive表的文件数量，并进行文件合并合并多个文件数据到一个文件中，重新构建表

**03 请慎重使用SELECT(\*)**

**问题原因：**在大数据量多字段的数据表中，如果使用 SELECT \* 方式去查询数据，会造成很多无效数据的处理，会占用程序资源，造成资源的浪费

**解决方案：**在查询数据表时，指定所需的待查字段名，而非使用 \* 号

**04 不要在表关联后面加WHERE条件**

**问题原因：**

比如以下语句：

SELECT \* FROM stu as t LEFT JOIN course as t1ON t.id=t2.stu\_idWHERE t.age=18;

请思考上面语句是否具有优化的空间？如何优化？

**解决方案：**

采用谓词下推的技术，提早进行过滤有可能减少必须在数据库分区之间传递的数据量

谓词下推的解释：

所谓谓词下推就是通过嵌套的方式，将底层查询语句尽量推到数据底层去过滤，这样在上层应用中就可以使用更少的数据量来查询，这种SQL技巧被称为谓词下推(Predicate pushdown)

那么上面语句就可以采用这种方式来处理：

SELECT \* FROM (SELECT \* FROM stu WHERE age=18) as t LEFT JOIN course AS t1 on t.id=t1.stu\_id

**05 处理掉字段中带有空值的数据**

**问题原因：**

一个表内有许多空值时会导致MapReduce过程中,空成为一个key值,对应的会有大量的value值, 而一个key的value会一起到达reduce造成内存不足

**解决方式：**

1、在查询的时候，过滤掉所有为NULL的数据，比如：

create table res\_tbl as  select n.\* from (select \* from res where id is not null ) n left join org\_tbl o on n.id = o.id;

2、查询出空值并给其赋上随机数,避免了key值为空（数据倾斜中常用的一种技巧）

create table res\_tbl asselect n.\* from res n full join org\_tbl o on case when n.id is null then concat('hive', rand()) else n.id end = o.id

**06 处理掉字段中带有空值的数据**

通过设置参数 hive.exec.parallel 值为 true，就可以开启并发执行。不过，在共享集群中，需要注意下，如果 job 中并行阶段增多，那么集群利用率就会增加。

//打开任务并行执行

* set hive.exec.parallel=true;

//同一个 sql 允许最大并行度，默认为 8

* set hive.exec.parallel.thread.number=16;

**07 设置合理的Reducer数量**

**问题原因：**

* 过多的启动和初始化 reduce 也会消耗时间和资源
* 有多少个Reduer就会有多少个文件产生，如果生成了很多个小文件，那么如果这些小文件作为下一个任务的输入，则也会出现小文件过多的问题

**解决方案：**

Reducer设置的原则：

每个Reduce处理的数据默认是256MB

* hive.exec.reducers.bytes.per.reducer=256000000

每个任务最大的reduce数，默认为1009

* hive.exec.reducers.max=1009

计算reduce数的公式

N=min(每个任务最大的reduce数，总输入数据量/reduce处理数据量大小)

设置Reducer的数量

* set mapreduce.job.reduces=n

**08 JVM重用**

JVM重用是Hadoop中调优参数的内容，该方式对Hive的性能也有很大的帮助，特别对于很难避免小文件的场景或者Task特别多的场景,这类场景大数据书执行时间都很短

Hadood的默认配置通常是使用派生JVM来执行map和reduce任务的，会造成JVM的启动过程比较大的开销，尤其是在执行Job包含有成百上千个task任务的情况。

JVM重用可以使得JVM实例在同一个job中重新使用N次，N的值可以在hadoop的mapred-site.xml文件中进行设置

* mapred.job.reuse.jvm.num.tasks10

**09 为什么任务执行的时候只有一个reduce？**

**问题原因：**

使用了Order by （Order By是会进行全局排序）

直接COUNT(1),没有加GROUP BY，比如：

有笛卡尔积操作

SELECT COUNT(1) FROM tbl WHERE pt=’202109’

**解决方案：**

避免使用全局排序，可以使用sort by进行局部排序

使用GROUP BY进行统计，不会进行全局排序，比如：

* SELECT pt,COUNT(1) FROM tbl WHERE pt=’202109’group by pt;

**10 选择使用Tez引擎**

Tez: 是基于Hadoop Yarn之上的DAG（有向无环图，Directed Acyclic Graph）计算框架。它把Ｍap/Reduce过程拆分成若干个子过程，同时可以把多个Ｍap/Reduce任务组合成一个较大的DAG任务，减少了Ｍap/Reduce之间的文件存储。同时合理组合其子过程，也可以减少任务的运行时间。

虽然现在最新版本的Hive默认其实支持的Tez引擎， 但是很多人或者大部分人往往还是希望用MR引擎，特别是在Tez报错，然后MR运行正常的时候

设置

* hive.execution.engine = tez;

通过上述设置，执行的每个HIVE查询都将利用Tez

当然，也可以选择使用spark作为计算引擎

**11 选择使用本地模式**

有时候Hive处理的数据量非常小，那么在这种情况下，为查询出发执行任务的时间消耗可能会比实际job的执行时间要长，对于大多数这种情况，hive可以通过本地模式在单节点上处理所有任务，对于小数据量任务可以大大的缩短时间

可以通过

* hive.exec.mode.local.auto=true

**12 选择使用严格模式**

Hive提供了一种严格模式，可以防止用户执行那些可能产生意想不到的不好的影响查询

比如：

* 对于分区表，除非WHERE语句中含有分区字段过滤条件来限制数据范围，否则不允许执行，也就是说不允许扫描所有分区
* 使用ORDER BY 语句进行查询是，必须使用LIMIT语句，因为ORDER BY 为了执行排序过程会将所有结果数据分发到同一个reduce中进行处理，强制要求用户添加LIMIT可以防止reducer额外的执行很长时间

严格模式的配置：

* hive.mapred.mode=strict

好了，以上这十二条虽然不多，并且看起来简单，你可以作为一种复习来看，那么对于刚开始做不久的同学，可以将这些技巧严格的执行在日常工作中，并且希望你具备一定的调优的意识。

- END -