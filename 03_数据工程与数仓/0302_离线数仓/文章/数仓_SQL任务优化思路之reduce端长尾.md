---
title: 数仓|SQL任务优化思路之reduce端长尾
author: 大数据技术与数仓
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU2ODQ3NjYyMA==&mid=2247488825&idx=1&sn=a92729b696a59b33456be3913ca8de7e&chksm=fc8c039acbfb8a8c62f9529b7c453bd9bdadc5466e3e5f82465dcb4fda323ee5a09f7d75d36b&mpshare=1&scene=24&srcid=0428WTHcioNhwptV53pVvER8&sharer_shareinfo=098a6def547673753446ced0e10e945f&sharer_shareinfo_first=098a6def547673753446ced0e10e945f#rd
---

## 写在前面

在[SQL任务优化思路之map端长尾](https://mp.weixin.qq.com/s?__biz=MzU2ODQ3NjYyMA==&mid=2247488801&idx=1&sn=0af1052b4e0a6c73b3c5b6b05b70cc09&scene=21#wechat_redirect)以及[SQL任务优化思路之JOIN端长尾](https://mp.weixin.qq.com/s?__biz=MzU2ODQ3NjYyMA==&mid=2247488817&idx=1&sn=b156182e2e97762c89ef2640e336ba8c&scene=21#wechat_redirect)这两篇分享中，我们介绍Map端长尾和JOIN端长尾的优化思路。本文将延续前两篇分享，继续介绍Reduce端长尾相关问题，通过本文你可以了解到：

* Reduce端Shuffle过程
* Reduce端长尾的原因
* Reduce端长尾优化思路

感谢关注，希望本文对你有所帮助。

## Reduce端Shuffle过程

* Reduce任务通过RPC (Remote Procedure Call) 向JobTracker询问，以确定Map任务是否已经完成。一旦Map任务完成，Reduce任务就会从相应的Map任务那里领取处理好的中间结果数据。
* Reduce任务从不同的Map节点领取的数据会先放入本地的缓存。这些领取的数据来自于不同的Map机器，因此Reduce任务需要先对它们进行归并，再将归并好的数据合并，并写入磁盘以形成溢写文件。
* 如果有多个溢写文件，Reduce任务需要将这些文件再归并成一个或多个更大的文件。在这个过程中，保持键值对是有序的，以便后续的处理。
* 最后，如果Map端输出的数据量很少，那么可能不需要将数据溢写到磁盘。这时，可以直接在内存缓存中进行归并，然后输出给Reduce函数进行最终的处理。

img

## Reduce端长尾的原因

Reduce端负责的是对Map端梳理后的有序Key-Value键值对进行聚合，即进行Count、Sum、Avg等聚合操作。Reduce端产生长尾的主要原因就是因为Key的数据分布的不均匀导致。某些Reduce任务Instance处理的数据记录多，有些处理的少，造成了Reduce端长尾。造成Reduce端长尾的场景主要有：

* 在Join阶段中会存在一些表的Null值很多，造成很多Null值被分发到同一个Reduce任务Instance上，造成Reduce端长尾；
* 对同一个表按照不同维度组合对不同的列进行Count Distinct操作，造成Map端数据膨胀从而Reduce出现长尾；
* 动态分区数过多时可能造成的小文件数过多；
* Map端对分发维度的值进行随机化(**Distribute By**)，造成Reduce端计算资源紧张；
* 多个Distinct同时出现在一段SQL代码中时，数据会被分发多次，不仅会造成数据膨胀多倍，还会把长尾现象放大多倍；

数据倾斜可分为以下两种类型:

* **键值倾斜：键值单一**，即少量的键对应的大量的值或记录。相同key的记录会被分配到同一个reducer上，则某个reducer要处理该key所有记录，造成该任务处理的时间较长。(某些键非常常见（例如，在一个网站的日志分析中，“首页”的访问量可能远远高于其他任何页面），而其余的键则相对较少出现。)
* \*\*分区倾斜：键值分散,\*\*但分区后过于聚集.即不同键值的记录数虽然不多，但经过默认Hash分区之后，过于聚集在某个reducer中。MapReduce任务中出现数据倾斜问题时，一般表现为整个任务进度长时间维持在99％(或100％)。而当查看任务监控页面时，发现只有1个或几个Reduce任务显示未完成。这种情况通常是由于某一或某几个reducer需处理的记录数远高于平均记录数，使得这些reducer的运行时长远大于平均时长。(即使所有键的出现频率相对平均，但由于默认的哈希分区函数或者键的分布，可能导致某些分区接收到过多的数据)

常见的数据倾斜，经常集中在以下操作

* group by
* count distinct
* TopN

## Reduce端长尾优化思路

### group by数据倾斜

* 正常做法

```
SELECT  
      key  
      ,count(*  
            )  
FROM tablename  
GROUP BY key
```

* 将倾斜的key打散，进行二次GROUP BY

```
-- 假设长尾的Key已经找到是K1  
SELECT  a.Key  
        ,SUM(a.Cnt) AS Cnt  
FROM    (  
            SELECT  Key  
                    ,COUNT(*) AS Cnt  
            FROM    TableName  
            GROUP BY Key  
                     ,CASE    WHEN KEY = 'K1' THEN Hash(Random()) % 50   
                              ELSE 0   
                      END  
        ) a  
GROUP BY a.Key  
;
```

### count distinct数据倾斜

#### 原理

在一个分布式计算环境中，比如使用Hadoop的Hive，执行带有 COUNT(DISTINCT ...) 的 GROUP BY 查询时，如果某个或某些键(key)对应的唯一值的数量非常多，这些键将成为热点，会导致数据倾斜问题。

在MapReduce作业中，这种情况通常表现为大多数Reduce任务很快完成它们的工作，而少数几个Reduce任务需要处理大量的数据，因此耗时会明显更长。这些处理大量数据的Reduce任务通常是由于它们需要处理的键(key)有大量的唯一值(distinct values)。

当只有一个Distinct字段时，将Group by字段和Distinct字段组合为map输出key，利用MapReduce的排序，同时将Group by字段作为分区key，即partition key，在Reduce阶段根据Distinct字段完成去重。

* Map：将groupby和distinct 字段组合为map的输出key，value设置为1，利用mapreduce的排序，同时将groupby字段作为分区key，可以确保相同groupby字段的记录被分发到同一个reducer。
* Shuffle：相同key按value排序，并按照分区key分发到Reduce
* Reduce：按顺序取出组合键中的distinct字段（这时distinct字段也是排好序的），依次遍历distinct字段，每找到一个不同值，计数器就自增1，即可得到count distinct结果

```
select dealid,count(distinct uid) as num from order group by dealid;
```

img

#### 优化方案

```
SELECT  c1  
        ,count(DISTINCT c2)  
FROM    明细表  
GROUP BY c1
```

* 两阶段聚合

先对要分组去重的表按照相应的粒度去重，然后再做聚合

```
SELECT  c1  
        ,COUNT(*) AS cnt  
FROM    (  
            SELECT  c1  
                    ,c2  
            FROM    明细表  
            GROUP BY c1  
                     ,c2  
        )   
GROUP BY c1  
;
```

* 两阶段聚合+拼接随机数

如果上述表c2存在数据倾斜，则通过两阶段聚合的方式优化效果不会很明显，这个时候可以考虑通过拼接随机数的形式将数据打散

```
SELECT  SPLIT_PART(rand_c1, '_',2)  
        ,COUNT(*) AS cnt  
FROM    (  
            SELECT  CONCAT(ROUND(RAND(),1)*10,'_', c1) AS rand_c1  
                    ,c2  
            FROM    明细表  
            GROUP BY CONCAT(ROUND(RAND(),1)*10,'_', c1)  
                     ,c2  
        )   
GROUP BY SPLIT_PART(rand_c1, '_',2)  
;
```

### TopN数据倾斜

在数据开发过程中，经常会遇到取某个维度下的Top数据。比如要取类目下的TopN的属性值，通用的方法是使用Row\_Number排序，然后再取Top。

* 正常写法

```
select   cate_id  
        ,property_id  
        ,value_id  
from  
        (select   cate_id  
                 ,property_id  
                 ,value_id  
                 ,row_number() over(partition by cate_id order by property_id asc,value_id asc) as rn  
         from     demo_tbl  
         where    ds = '${bizdate}'  
         ) p  
where    rn <= N;
```

Row\_Number按照`cate_id`分组，在每个分组内进行排序。如果一个`cate_id`下有大量的属性值就会发生倾斜。那么如何才能既防止倾斜，又能实现排序呢？

* 优化写法

```
select   cate_id  
        ,property_id  
        ,value_id  
from  
        (select   cate_id  
                 ,property_id  
                 ,value_id  
                 ,row_number() over(partition by cate_id order by property_id asc,value_id asc) as rn  
         from  
                 (select   cate_id  
                          ,property_id  
                          ,value_id  
                  from  
                          (select   cate_id  
                                   ,property_id  
                                   ,value_id  
                                   ,row_number() over(partition by cate_id,sec_part order by property_id asc,value_id asc)        as rn  
                           from  
                                   (select   cate_id  
                                            ,property_id  
                                            ,value_id  
                                            ,ceil(M*rand())%P as sec_part  
                                    from     demo_tbl  
                                    where    ds = '${bizdate}'  
                                    ) s  
                           ) p  
                  where    rn <= N  
                 ) part  
        ) a  
where    rn <= N
```

在提供的SQL中，通过引入一个额外的随机数分区字段（sec\_part）和两级Top-N查询来避免数据倾斜：

* **随机数分区**：首先，通过ceil(M\*rand())%P as sec\_part计算出一个随机分区标识，目的是将同一类别（cate\_id）的数据随机均匀地分散到不同的分区（sec\_part）中。
* **局部Top-N查询**：在内层查询中，使用row\_number() over(partition by cate\_id,sec\_part order by property\_id asc,value\_id asc)为每个类别的每个分区内的数据排名（局部Top-N），接着通过where rn <= N过滤出每个分区的Top-N记录。
* **全局Top-N查询**：在最外层的查询中，再次使用row\_number()，这次是在前面筛选过的数据上进行全局排名（全局Top-N），再次通过where rn <= N过滤得到每个类别的全局Top-N记录。

**关于M的选择：** M的选择是为了生成足够大范围的随机数，确保随机数分布的均匀性。如果M值太小，生成的随机数可能不够分散；如果M值太大，可能会生成不必要的大范围值，但这其实不会对结果有太大影响，因为最终的分区是通过取模%P来确定的。因此，M通常选取为P的一个较大的倍数（如10倍），这样做可以确保随机数分布在一个较宽的范围内，使得取模后的结果分布更加均匀。

**关于P的选择：** P的选择是为了确定分区的个数，它决定了数据将被分散到多少个不同的分区中。选择P为素数是为了减少哈希冲突的概率，从而使得数据分布更加均匀。如果P被选为一个不是素数的数字，那么某些分区可能会接收到不成比例的数据量，而其他分区则相对较少，从而没有达到预期的负载均衡。

在实际应用中，P的选择需要根据数据的实际大小和分布情况来确定。如果原有分组中最大的Instance处理的数据量是A个，那么理论上分成P组后，每组的数据量约为A/P个。显然，P的值不能过小，否则每组的数据量仍然可能太大，导致处理时仍然存在倾斜；同样，P的值也不能过大，否则会产生过多的小分区，从而导致任务管理和调度上的开销。

在确定了P值之后，可以通过实际的测试运行来评估查询性能和数据负载情况，如果必要，可以进一步调整P的值以达到最优的查询效率和负载平衡。

## 面试题：COUNT(DISTINCT)为什么会比较慢

COUNT(DISTINCT)操作的执行涉及两个核心步骤：Map阶段和Reduce阶段。这些步骤是按照MapReduce框架来执行的。

### Map阶段：

* 每个Mapper读取其分配的数据块，然后对查询涉及的列进行处理。
* Mapper将要去重的列和分组列的值作为输出key，通常会有一个常数值（如1）作为输出值。

### Shuffle阶段（MapReduce中的中间步骤）：

* Mapper的输出在传输到Reducer之前会进行排序和分组。Hive会将具有相同键（在这里是要去重的列的值）的值发送到同一个Reducer。

### Reduce阶段：

* Reducer接收到所有Mapper输出的键值对，并且聚合相同键的值。
* Reducer对其接收到的键列表去重，以确保每个键只计入一次。
* Reducer计算去重后的键的数量，这就是COUNT(DISTINCT)的结果。

### 耗时环节：

在这个过程中，主要耗时的环节是Reduce阶段，特别是Reducer的去重操作。以下是详细说明：

1. **数据倾斜**：如果某些唯一值非常常见，则会导致一部分Reducer任务要处理的数据量远远超过其他Reducer，造成资源分配不均匀和数据处理瓶颈。
2. **去重开销**：在Reducer中，为了计算唯一值的数量，必须在内存中维护一个全局的唯一值集合（如HashSet），这需要对每个值进行查找、插入和更新操作，是计算密集型的。
3. **内存和I/O限制**：对于大量的不同值，Reducer需要足够的内存来存储这些唯一值。如果内存不足以存放所有唯一值，可能会导致磁盘I/O操作，进一步影响性能。
4. **网络传输**：在Shuffle阶段，所有的数据必须通过网络从Mapper传输到Reducer，如果数据量大，这个过程会很慢，并且容易成为整个查询的瓶颈。

## 总结

本文主要介绍了Reduce端长尾常见的问题以及解决思路，当然，法无定法，在实际的应用中要结合具体的案例去分析，根据数据的特点选择不同的处理方案。