---
title: 面试通关秘籍！Hive高频难题精选 →
author: 大数据技能圈
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488645&idx=1&sn=991faaf2f231512e4a5e3157d7ea3840&chksm=c0296f1af75ee60c7c490d18fb9ae26509663a1f279b88eb95e838071084842263b0e6761cdf&mpshare=1&scene=24&srcid=0510So8r1sPKym42tuKOFZmm&sharer_shareinfo=19e51b9cdf21bc40ee51f39972309012&sharer_shareinfo_first=19e51b9cdf21bc40ee51f39972309012#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030202_Hive/030202_知识地图|知识地图]]


**1 Hive 的架构**

**2 HQL 转换为 MR 流**

（1）解析器（SQLParser）：将 SQL 字符串转换成抽象语法树（AST）

（2）语义分析器（Semantic Analyzer）：将 AST 进一步抽象为 QueryBlock（可以理解

为一个子查询划分成一个 QueryBlock）

（2）逻辑计划生成器（Logical Plan Gen）：由 QueryBlock 生成逻辑计划

（3）逻辑优化器（Logical Optimizer）：对逻辑计划进行优化

（4）物理计划生成器（Physical Plan Gen）：根据优化后的逻辑计划生成物理计划

（5）物理优化器（Physical Optimizer）：对物理计划进行优化

（6）执行器（Execution）：执行该计划，得到查询结果并返回给客户端

**3 Hive 和数据库比**

Hive 和数据库除了拥有类似的查询语言，再无类似之处。

1）数据存储位置

Hive 存储在 HDFS 。数据库将数据保存在块设备或者本地文件系统中。

2）数据更新

Hive 中不建议对数据的改写。而数据库中的数据通常是需要经常进行修改的。

3）执行延迟

Hive 执行延迟较高。数据库的执行延迟较低。当然，这个是有条件的，即数据规模较小，当数据规模大到超过数据库的处理能力的时候，Hive 的并行计算显然能体现出优势。

4）数据规模

Hive 支持很大规模的数据计算；数据库可以支持的数据规模较小。

**4 内部表和外部表**

元数据、原始数据

1）删除数据时

内部表：元数据、原始数据，全删除

外部表：元数据 只删除

2）在公司生产环境下，什么时候创建内部表，什么时候创建外部表？

在公司中绝大多数场景都是外部表。

自己使用的临时表，才会创建内部表

**5 系统函数**

1）数值函数

（1）round：四舍五入；（2）ceil：向上取整；（3）floor：向下取整

2）字符串函数

（1）substring：截取字符串；（2）replace：替换；（3）regexp\_replace：正则替换

（4）regexp：正则匹配；（5）repeat：重复字符串；（6）split：字符串切割

（7）nvl：替换 null 值；（8）concat：拼接字符串；

（9）concat\_ws：以指定分隔符拼接字符串或者字符串数组；

（10）get\_json\_object：解析 JSON 字符串

3）日期函数

（1）unix\_timestamp：返回当前或指定时间的时间戳

（2）from\_unixtime：转化 UNIX 时间戳（从 1970-01-01 00:00:00 UTC 到指定时间的秒数）到当前时区的时间格式

（3）current\_date：当前日期

（4）current\_timestamp：当前的日期加时间，并且精确的毫秒

（5）month：获取日期中的月；（6）day：获取日期中的日

（7）datediff：两个日期相差的天数（结束日期减去开始日期的天数）

（8）date\_add：日期加天数；（9）date\_sub：日期减天数

（10）date\_format：将标准日期解析成指定格式字符串

4）流程控制函数

（1）case when：条件判断函数

（2）if：条件判断，类似于 Java 中三元运算符

5）集合函数

（1）array：声明 array 集合

（2）map：创建 map 集合

（3）named\_struct：声明 struct 的属性和值

（4）size：集合中元素的个数

（5）map\_keys：返回 map 中的 key

（6）map\_values：返回 map 中的 value

（7）array\_contains：判断 array 中是否包含某个元素

（8）sort\_array：将 array 中的元素排序

6）聚合函数

（1）collect\_list：收集并形成 list 集合，结果不去重

（2）collect\_set：收集并形成 set 集合，结果去重

**6 自定义 UDF、UDTF 函数**

1）在项目中是否自定义过 UDF、UDTF 函数，以及用他们处理了什么问题，及自定义步骤？

（1）目前项目中逻辑不是特别复杂就没有用自定义 UDF 和 UDTF

（2）自定义 UDF：继承 G..UDF，重写核心方法 evaluate

（3）自定义 UDTF：继承自 GenericUDTF，重写 3 个方法：initialize（自定义输出的列名和类型），process（将结果返回 forward（result）），close

2）企业中一般什么场景下使用 UDF/UDTF？

（1）因为自定义函数，可以将自定函数内部任意计算过程打印输出，方便调试。

（2）引入第三方 jar 包时，也需要。

**7 窗口函数**

一般在场景题中出现手写：分组 TopN、行转列、列转行。

按照功能，常用窗口可划分为如下几类：聚合函数、跨行取值函数、排名函数。

1）聚合函数

max：最大值。

min：最小值。

sum：求和。

avg：平均值。

count：计数。

2）跨行取值函数

（1）lead 和 lag

注：lag 和 lead 函数不支持自定义窗口

（2）first\_value 和 last\_value

**8 Hive 优化**

**8.1 分组聚合**

一个分组聚合的查询语句，默认是通过一个 MapReduce Job 完成的。Map 端负责读取数据，并按照分组字段分区，通过 Shuffle，将数据发往 Reduce 端，各组数据在 Reduce 端完成最终的聚合运算

分组聚合的优化主要围绕着减少 Shuffle 数据量进行，具体做法是 map-side 聚合。所谓map-side 聚合，就是在 map 端维护一个 Hash Table，利用其完成部分的聚合，然后将部分聚 合的结果，按照分组字段分区，发送至 Reduce 端，完成最终的聚合

-启用 map-side 聚合，默认是 true

```
set hive.map.aggr=true;
```

--用于检测源表数据是否适合进行 map-side 聚合。检测的方法是：先对若干条数据进行

map-side 聚合，若聚合后的条数和聚合前的条数比值小于该值，则认为该表适合进行

map-side 聚合；否则，认为该表数据不适合进行 map-side 聚合，后续数据便不再进行

map-side 聚合。

```
set hive.map.aggr.hash.min.reduction=0.5;
```

--用于检测源表是否适合 map-side 聚合的条数。

```
set hive.groupby.mapaggr.checkinterval=100000;
```

--map-side 聚合所用的 hash table，占用 map task 堆内存的最大比例，若超出该值，

则会对 hash table 进行一次 flush。

```
set hive.map.aggr.hash.force.flush.memory.threshold=0.9;
```

**8.2 Map Join**

Hive 中默认最稳定的 Join 算法是 Common Join。其通过一个 MapReduce Job 完成一个 Join 操作。Map 端负责读取 Join 操作所需表的数据，并按照关联字段进行分区，通过 Shuffle， 将其发送到 Reduce 端，相同 key 的数据在 Reduce 端完成最终的 Join 操作。

优化 Join 的最为常用的手段就是 Map Join，其可通过两个只有 Map 阶段的 Job 完成一 个 join 操作。第一个 Job 会读取小表数据，将其制作为 Hash Table，并上传至 Hadoop 分布 式缓存（本质上是上传至 HDFS）。第二个 Job 会先从分布式缓存中读取小表数据，并缓存 在 Map Task 的内存中，然后扫描大表数据，这样在 map 端即可完成关联操作

注：由于 Map Join 需要缓存整个小标的数据，故只适用于大表 Join 小表的场景。

--启动 Map Join 自动转换

```
set hive.auto.convert.join=true;
```

--开启无条件转 Map Join

```
set hive.auto.convert.join.noconditionaltask=true;
```

--无条件转 Map Join 小表阈值，默认值 10M，推荐设置为 Map Task 总内存的三分之一到二分之一

```
set hive.auto.convert.join.noconditionaltask.size=10000000;
```

**8.3 SMB Map Join**

上节提到，Map Join 只适用于大表 Join 小表的场景。若想提高大表 Join 大表的计算效

率，可使用 Sort Merge

需要注意的是 SMB Map Join 有如下要求：

（1）参与 Join 的表均为分桶表，且分桶字段为 Join 的关联字段。

（2）两表分桶数呈倍数关系。

（3）数据在分桶内是按关联字段有序的。

SMB Join 的核心原理如下：只要保证了上述三点要求的前两点，就能保证参与 Join 的两张表的分桶之间具有明确的关联关系，因此就可以在两表的分桶间进行 Join 操作了。

若能保证第三点，也就是参与 Join 的数据是有序的，这样就能使用数据库中常用的 Join 算法之一——Sort Merge Join 了，Merge Join 原理如下：

在满足了上述三点要求之后，就能使用 SMB Map

由于 SMB Map Join 无需构建 Hash Table 也无需缓存小表数据，故其对内存要求很低。适用于大表 Join 大表的场景。

**8.4 Reduce 并行度**

Reduce 端的并行度，也就是 Reduce 个数，可由用户自己指定，也可由 Hive 自行根据该 MR Job 输入的文件大小进行估算。

Reduce 端的并行度的相关参数如下

--指定 Reduce 端并行度，默认值为-1，表示用户未指定

```
set mapreduce.job.reduces;
```

--Reduce 端并行度最大值

```
set hive.exec.reducers.max;
```

--单个 Reduce Task 计算的数据量，用于估算 Reduce 并行度

```
set hive.exec.reducers.bytes.per.reducer;
```

Reduce 端并行度的确定逻辑如下：

若指定参数 mapreduce.job.reduces 的值为一个非负整数，则 Reduce 并行度为指定值。否则，Hive 自行估算 Reduce 并行度，估算逻辑如下：假设 Job 输入的文件大小为 totalInputBytes参数 hive.exec.reducers.bytes.per.reducer 的值为 bytesPerReducer。参数 hive.exec.reducers.max 的值为 maxReducers。则 Reducee 端的并行度为

根据上述描述，可以看出，Hive 自行估算 Reduce 并行度时，是以整个 MR Job 输入的文件大小作为依据的。因此，在某些情况下其估计的并行度很可能并不准确，此时就需要用 户根据实际情况来指定 Reduce 并行度了。需要说明的是：若使用 Tez 或者是 Spark 引擎，Hive 可根据计算统计信息（Statistics） 估算 Reduce 并行度，其估算的结果相对更加准确。

**8.5 小文件合并**

若 Hive 的 Reduce 并行度设置不合理，或者估算不合理，就可能导致计算结果出现大量的小文件。该问题可由小文件合并任务解决。其原理是根据计算任务输出文件的平均大小进行判断，若符合条件，则单独启动一个额外的任务进行合并。

相关参数为：

--开启合并 map only 任务输出的小文件

```
set hive.merge.mapfiles=true;
```

--开启合并 map reduce 任务输出的小文件

```
set hive.merge.mapredfiles=true;
```

--合并后的文件大小

```
set hive.merge.size.per.task=256000000;
```

--触发小文件合并任务的阈值，若某计算任务输出的文件平均大小低于该值，则触发合并

```
set hive.merge.smallfiles.avgsize=16000000;
```

**8.6 谓词下推**

谓词下推（predicate pushdown）是指，尽量将过滤操作前移，以减少后续计算步骤的数 据量。开启谓词下推优化后，无需调整 SQL 语句，Hive 就会自动将过滤操作尽可能的前移动。

相关参数为：

--是否启动谓词下推（predicate pushdown）优化

```
set hive.optimize.ppd = true
```

Hive 会将一个 SQL 语句转化成一个或者多个 Stage，每个 Stage 对应一个 MR Job。默认情况下，Hive 同时只会执行一个 Stage。但是某 SQL 语句可能会包含多个 Stage，但这多个 Stage 可能并非完全互相依赖，也就是说有些 Stage 是可以并行执行的。此处提到的并行

执行就是指这些 Stage 的并行执行。相关参数如下：

--启用并行执行优化，默认是关闭的

```
set hive.exec.parallel=true;
```

--同一个 sql 允许最大并行度，默认为 8

```
set hive.exec.parallel.thread.number=8;
```

**8.8 CBO 优化**

CBO 是指 Cost based Optimizer，即基于计算成本的优化。

在 Hive 中，计算成本模型考虑到了：数据的行数、CPU、本地 IO、HDFS IO、网络 IO等方面。Hive 会计算同一 SQL 语句的不同执行计划的计算成本，并选出成本最低的执行计划。目前 CBO 在 Hive 的 MR 引擎下主要用于 Join 的优化，例如多表 Join 的 Join的顺序

相关参数为：

--是否启用 cbo 优化

set hive.cbo.enable=true;

**8.9 列式存储**

采用 ORC 列式存储加快查询速度。

```
id name age1 zs 182 lishi 19行：1 zs 18 2 lishi 19列：1 2 zs lishi 18 19select name from use
```

**8.10 压缩**

压缩减少磁盘 IO：因为 Hive 底层计算引擎默认是 MR，可以在 Map 输出端采用Snappy 压缩。

Map（Snappy ） Reduc

**8.11 分区和分桶**

（1）创建分区表 防止后续全表扫描

（2）创建分桶表 对未知的复杂的数据进行提前采样

**8.12 更换引擎**

1）MR/Tez/Spark 区别：

MR 引擎：多 Job 串联，基于磁盘，落盘的地方比较多。虽然慢，但一定能跑出结果。一般处理，周、月、年指标。

Spark 引擎：虽然在 Shuffle 过程中也落盘，但是并不是所有算子都需要 Shuffle，尤其是多算子过程，中间过程不落盘 DAG 有向无环图。兼顾了可靠性和效率。一般处理天指标。

2）Tez 引擎擎的优点

（1）使用 DAG 描述任务，可以减少 MR 中不必要的中间节点，从而减少磁盘 IO 和网络 IO。

（2）可更好的利用集群资源，例如 Container 重用、根据集群资源计算初始任务的并行度等。

（3）可在任务运行时，根据具体数据量，动态的调整后续任务的并行度。

**8.13 几十张表 join 如何优化**

（1）减少 join 的表数量：不影响业务前提，可以考虑将一些表进行预处理和合并，从 而减少 join操作

（2）使用 Map Join：将小表加载到内存中，从而避免了 Reduce 操作，提高了性能。通 过设置 hive.auto.convert.join 为 true 来启用自动 Map Join。

（3）使用 Bucketed Map Join：通过设置hive.optimize.bucketmapjoin为 true来启用 Bucketed Map Join

（4）使用 Sort Merge Join：这种方式在 Map 阶段完成排序，从而减少了 Reduce 阶段的 计算量。通过设置 hive.auto.convert.sortmerge.join

（5）控制 Reduce 任务数量：通过合理设置 hive.exec.reducers.bytes.per.reducer 和mapreduce.job.reduces 参数来控制 Reduce 任务的数量

（6）过滤不需要的数据：join 操作之前，尽量过滤掉不需要的数据，从而提高性能。

（7）选择合适的 join 顺序：将小表放在前面可以减少中间结果的数据量，提高性能。

（8）使用分区：可以考虑使用分区技术。只需要读取与查询条件匹配的分区数据，从 而减少数据量和计算量。

（9）使用压缩：通过对数据进行压缩，可以减少磁盘和网络 IO，提高性能。注意选择 合适的压缩格式和压缩级别。

（10）调整 Hive 配置参数：根据集群的硬件资源和实际需求，合理调整 Hive 的配置参 数，如内存、CPU、IO 等，以提高性能。

**9 Hive 解决数据倾斜方法**

数据倾斜问题，通常是指参与计算的数据分布不均，即某个 key 或者某些 key 的数据量远超其他 key，导致在 shuffle 阶段，大量相同 key 的数据被发往同一个 Reduce，进而导致该Reduce 所需的时间远超其他 Reduce，成为整个任务的瓶颈。以下为生产环境中数据倾斜的现象：

Hive 中的数据倾斜常出现在分组聚合和 join 操作的场景中，下面分别介绍在上述两种场景下的优化思路。

1）分组聚合导致的数据倾斜

前文提到过，Hive 中的分组聚合是由一个 MapReduce Job 完成的。Map 端负责读取数据，并按照分组字段分区，通过 Shuffle，将数据发往 Reduce 端，各组数据在 Reduce 端完成最终的聚合运算。若group by分组字段的值分布不均，就可能导致大量相同的key进入同一Reduce，从而导致数据倾斜。

由分组聚合导致的数据倾斜问题，有如下解决思路：

（1）判断倾斜的值是否为 null

若倾斜的值为 null，可考虑最终结果是否需要这部分数据，若不需要，只要提前将 null过滤掉，就能解决问题。若需要保留这部分数据，考虑以下思路。

（2）Map-Side 聚合

开启 Map-Side 聚合后，数据会现在 Map 端完成部分聚合工作。这样一来即便原始数据是倾斜的，经过 Map 端的初步聚合后，发往 Reduce 的数据也就不再倾斜了。最佳状态下，Map 端聚合能完全屏蔽数据倾斜问题。

相关参数如下：

```
set hive.map.aggr=true;set hive.map.aggr.hash.min.reduction=0.5;set hive.groupby.mapaggr.checkinterval=100000;set hive.map.aggr.hash.force.flush.memory.threshold=0.9
```

（3）Skew-GroupBy 优化

Skew-GroupBy 是 Hive 提供的一个专门用来解决分组聚合导致的数据倾斜问题的方案。其原理是启动两个 MR 任务，第一个 MR 按照随机数分区，将数据分散发送到 Reduce，并完成部分聚合，第二个 MR 按照分组字段分区，完成最终聚合。

相关参数如下：

--启用分组聚合数据倾斜优化

```
set hive.groupby.skewindata=true;
```

2）Join 导致的数据倾斜

若 Join 操作使用的是 Common Join 算法，就会通过一个 MapReduce Job 完成计算。Map端负责读取 Join 操作所需表的数据，并按照关联字段进行分区，通过 Shuffle，将其发送到Reduce 端，相同 key 的数据在 Reduce 端完成最终的 Join 操作。如果关联字段的值分布不均，就可能导致大量相同的 key 进入同一 Reduce，从而导致数据倾斜问题。

由 Join 导致的数据倾斜问题，有如下解决思路：

（1）Map Join

使用 Map Join 算法，Join 操作仅在 Map 端就能完成，没有 Shuffle 操作，没有 Reduce阶段，自然不会产生 Reduce 端的数据倾斜。该方案适用于大表 Join 小表时发生数据倾斜的场景。

相关参数如下：

```
set hive.auto.convert.join=true;set hive.auto.convert.join.noconditionaltask=true;set hive.auto.convert.join.noconditionaltask.size=10000000;
```

（2）Skew Join

若参与 Join 的两表均为大表，Map Join 就难以应对了。此时可考虑 Skew Join，其核心原理是 Skew Join 的原理是，为倾斜的大 key 单独启动一个 Map Join 任务进行计算，其余 key进行正常的 Common Join。原理图如下：

相关参数如下：

--启用 skew join 优化

```
set hive.optimize.skewjoin=true;
```

--触发 skew join 的阈值，若某个 key 的行数超过该参数值，则触发

```
set hive.skewjoin.key=100000;
```

3）调整 SQL 语句

若参与 Join 的两表均为大表，其中一张表的数据是倾斜的，此时也可通过以下方式对SQL 语句进行相应的调整。

假设原始 SQL 语句如下：A，B 两表均为大表，且其中一张表的数据是倾斜的。

```
hive (default)>select*from Ajoin Bon A.id=B.id
```

其 Join过程如下:

图中 1001 为倾斜的大 key，可以看到，其被发往了同一个 Reduce 进行处理。

调整之后的 SQL 语句执行计划如下图所示

```
调整 SQL 语句如下：hive (default)>select*from(select --打散操作concat(id,'_',cast(rand()*2 as int)) id,valuefrom A)tajoin(select --扩容操concat(id,'_',1) id,valuefrom Bunion allselectconcat(id,'_',2) id,valuefrom B)tbon ta.id=tb.id
```

**10 Hive 的数据中含有字段的分隔符怎么处理？**

Hive 默认的字段分隔符为 Ascii 码的控制符\001（^A），建表的时候用 fields terminated by'\001'。注意：如果采用\t 或者\001 等为分隔符，需要要求前端埋点和 JavaEE 后台传递过来的数据必须不能出现该分隔符，通过代码规范约束。一旦传输过来的数据含有分隔符，需要在前一级数据中转义或者替换（ETL）。通常采用 Sqoop 和 DataX 在同步数据时预处理。

```
id name age1 zs 182 li 分隔符 si 19
```

****11 MySQL 元数据备份****

元数据备份（重点，如数据损坏，可能整个集群无法运行，至少要保证每日零点之后备份到其它服务器两个复本）

（1）MySQL 备份数据脚本（建议每天定时执行一次备份元数据）

```
#/bin/bash#常量设置MYSQL_HOST='hadoop102'MYSQL_USER='root'MYSQL_PASSWORD='000000'# 备份目录，需提前创建BACKUP_DIR='/root/mysql-backup'# 备份天数，超过这个值，最旧的备份会被删除FILE_ROLL_COUNT='7'# 备份 MySQL 数据库[ -d "${BACKUP_DIR}" ] || exit 1mysqldump \--all-databases \--opt \--single-transaction \--source-data=2 \--default-character-set=utf8-h"${MYSQL_HOST}" \-u"${MYSQL_USER}" \-p"${MYSQL_PASSWORD}" | gzip > "${BACKUP_DIR}/$(date +%F).gz"if [ "$(ls "${BACKUP_DIR}" | wc -l )" -gt "${FILE_ROLL_COUNT}" ]thenls "${BACKUP_DIR}" | sort |sed -n 1p | xargs -I {} -n1 rm -rf"${BACKUP_DIR}"/{}fi
```

（2）MySQL 恢复数据脚本

```
#/bin/bash#常量设置MYSQL_HOST='hadoop102'MYSQL_USER='root'MYSQL_PASSWORD='000000'BACKUP_DIR='/root/mysql-backup'# 恢复指定日期，不指定就恢复最新数据RESTORE_DATE=''[ "${RESTORE_DATE}" ] && BACKUP_FILE="${RESTORE_DATE}.gz" ||BACKUP_FILE="$(ls ${BACKUP_DIR} | sort -r | sed -n 1p)"gunzip "${BACKUP_DIR}/${BACKUP_FILE}" --stdout | mysql \-h"${MYSQL_HOST}" \-u"${MYSQL_USER}" \-p"${MYSQL_PASSWORD}"
```

**12 如何创建二级分区表？**

```
create table dept_partition2(deptno int, -- 部门编号dname string, -- 部门名称)partitioned by (day string, hour string)row format delimited fields terminated by '\t'
```

**进群添加作者：**

**获取离线文档：**

**精彩推荐**

[实时离线数仓实战No.2 | 数仓业务详解 →](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488612&idx=1&sn=9ea5038cd8b6fbb19be2fde2ce8f50bf&chksm=c0296ffbf75ee6ed28a8908fefd612debf81d52a229db06911022bdb084f7f3aef1870756acb&scene=21#wechat_redirect)

[实时离线数仓实战V2 | 发布预告 →](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488611&idx=1&sn=c5474d6765bb325ac096a5b808f528d5&chksm=c0296ffcf75ee6ea73c755520b63d04f4824813f6bea3e012d23a2d6ade03a19d1b8dfb59ec0&scene=21#wechat_redirect)

[SeaTunnel配置大全 | 50页详解Transform →](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488493&idx=1&sn=e791942b3ffe0c5616f15b95185f9990&chksm=c0296872f75ee164a96df5528f4746a4a63edd9b947afb4107b985484f65eea48a44d4d3f51c&scene=21#wechat_redirect)

[SeaTunnel配置秘籍 | 400页文档详尽指南 →](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488486&idx=1&sn=a163d9713691b8943f9a737fe4d981d1&chksm=c0296879f75ee16f26cc4e96c45eaa912236cb4f8e8a41844fea7eab52ec71f00fa404aa71a1&scene=21#wechat_redirect)

[数据可视化新篇章：Superset之后，Datart如何重塑行业格局？](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488465&idx=1&sn=c615f072c0c6c78fd44a2e227c23f7ab&chksm=c029684ef75ee1588005ffbd2202cfe264dbcb29a0aba717955c59c0c62d27dc1350c149f8cc&scene=21#wechat_redirect)

请各位读者动动手指点赞、收藏、在看，您的支持是我持续创作的动力，感谢。