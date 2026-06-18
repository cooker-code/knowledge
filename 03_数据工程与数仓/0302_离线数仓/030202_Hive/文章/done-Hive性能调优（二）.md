> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030202_Hive/030202_核心知识点/Hive性能优化|Hive性能优化]]
---
title: Hive性能调优（二）
author: 每天学点填坑小技巧
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg4ODMwNTYwNg==&mid=2247483844&idx=1&sn=992b29f0a2a9f542f68883f4b862a400&chksm=cffc636df88bea7b3475d24e5eb215945d9140fa3dc90af3c9a39648e53d0ddb09278cdb59c3&mpshare=1&scene=24&srcid=0412oxAEe3JPhCulg3tN9GU6&sharer_shareinfo=7a6465c2a9a41274136840777f1b294f&sharer_shareinfo_first=7a6465c2a9a41274136840777f1b294f#rd
---

书接上一篇，计算引擎除了采用tez外，还可以采用Spark、Storm，这里就不一一列举了，我们继续看一些新的Hive优化方式。

三、数据去重

有一个简单的数据去重，从用户表中判断某城市是否有用户及其数量：

```
SELECT s_city, COUNT(*) as city_countFROM usrGROUP BY s_city;
```

这便是我们最开始就会想到的sql语句，但聪明的小白会这样认为，如果我先找出city再计数，根据Hive列式数据的特点，是不是能加快sql查询速度呢，那么这个sql就可以这么写了：

```
SELECT s_city, COUNT(1) as city_countFROM (SELECT s_cityFROM usr--GROUP BY s_city) tGROUP BY s_city;
```

这个句子中子查询的GROUP BY s\_city只能算语句分析的中间步骤，但写完全句后它实际上是多余的，所以删除了，因为s\_city已经是唯一的，但是如果你这么写，就永远只会得到计数1.

```
SELECT s_city, COUNT(1) as city_countFROM (SELECT s_cityFROM usrGROUP BY s_city) t;
```

其实，我们大致想想，全国也就几百个城市，全世界也就几千个城市，针对这个列的优化意义不大。我们再看看distinct和group by的特性，distinct的真实实现是在内存中构建一个hashtable，hashtable查找去重的时间复杂度是O(1)；group by则是一种排序去重方式，无论如何，排序还不能做到复杂度到O(1)级别，虽然不同数据库在不同版本对group by优化比较大，有的也会用构建hashtable的形式去重，但这还做不到直接采用hashtable的复杂度。从Hive 3.0开始，也可以通过配置hive.optimize.countdistinct，进行count(distinct )优化，自动改变SQL执行的逻辑，即使真的出现数据倾斜也可以自动优化。

因此就用最快能想到的，加快工作进度。

四、小文件处理

我们一直说，Hive的底层是MapReduce实现计算，从本质上说，移动计算比移动数据快，如果，我们面对的全是小文件。那么在进行查询时，每个小文件都会当成一个块，启动一个Map任务来完成，Map任务启动和初始化的时间远远大于逻辑处理的时间，就会造成很大的资源浪费，结果合并的时间也会大大增加，同时可执行的Map数量是受限的。这又加慢了执行速度。小文件处理实操如下：

1.合并小文件，使用INSERT INTO语句将多个小文件合并为一个大文件：

```
INSERT OVERWRITE TABLE usrSELECT * FROM usr_ori;
```

这段代码会将表中的所有数据重新插入到同一个表中，从而合并小文件为大文件。

在合并小文件为大文件后，我们可以使用Hive提供的压缩功能对大文件进行压缩，通过设置Hive中的一些参数，将输出结果进行压缩以减少磁盘空间的占用和提高查询性能：

```
SET hive.exec.compress.output=true;SET mapreduce.output.fileoutputformat.compress=true;SET mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
```

上面的SnappyCodec是Hadoop中一种高效的压缩算法，你也可以选择其他的压缩算法。

如果你的数据表是分区表，那么在进行小文件治理后，我们还需要重新分区，以提高查询性能：

ALTER TABLE table\_name RECOVER PARTITIONS;

这段代码会重新搜集和加载所有的分区信息，以确保分区表的元数据信息是最新的。

2.使用hive 自带concatenate命令，自动合并小文件

#对于非分区表

alter table usr concatenate;

#对于分区表

alter table usr partition(dt='2024-04-07',hr='12') concatenate;

concatenate 命令只支持 RCFILE 和 ORC 文件类型。

但是使用concatenate命令合并小文件时不能指定合并后的文件数量，但可以多次执行该命令。 合理设置文件大小也很重要，当多次使用concatenate后文件数量不在变化，这个跟参数 mapreduce.input.fileinputformat.split.minsize=256mb 的设置有关，可设定每个文件的最小size。

3.使用hadoopgetmerge来合并小文件

```
hadoop fs -getmerge /user/hive/warehouse/xxxx.db/xxxx/pdate=20230923/*  /home/hadoop/pdate/20230923hadoop fs -rm /user/hive/warehouse/xxxx.db/xxxx/pdate=20230923/*hadoop fs -mkdir -p /user/hive/warehouse/xxxx.db/xxxx/pdate=20230923hadoop fs -put /home/hadoop/pdate/20230923  /user/hive/warehouse/xxxx.db/xxxx/pdate=20230923/*
```

最后，在数据插入时候尽量使用批量插入数据的方式，减少插入操作次数，从而减少小文件的生成。如LOAD DATA LOCAL INPATH '/path/to/local/usr.data' INTO TABLE usr;

还有尽量使用ORC或Parquet格式：这些格式比文本格式更紧凑，并且支持更好的压缩，有助于减少小文件的数量。

五、并行优化

默认情况下，Hive的查询转化为MapReduce任务后一次只会执行一个stage。不过，但是有些特定的job可能包含众多的stage，而实际上这些stage可能并非完全互相依赖的，也就是说有些阶段是可以并行执行的，这样可能使得整个job的执行时间缩短。如果有更多的阶段可以并行执行，那么job可能就越快完成。在资源充足情况下，可以设置：

set hive.exec.parallel=true; //开启任务并行

set hive.exec.parallel.thread.number=16; //一个sql允许的最大并行度，默认为8。