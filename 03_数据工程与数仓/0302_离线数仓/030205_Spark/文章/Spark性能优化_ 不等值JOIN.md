---
title: Spark性能优化: 不等值JOIN
author: 架构587
date: 
url: http://mp.weixin.qq.com/s?__biz=MzUxNTUyMTE4MA==&mid=2247484111&idx=1&sn=6a34384bbf553f8e2999024eda5de72b&chksm=f9b42d3ccec3a42a5b60871738f24e60169d739d32524df6813493b836b124bffba59ab5efd9&mpshare=1&scene=24&srcid=1122J9F1C3lKZnvncSXwCQYJ&sharer_shareinfo=f565de86bc2c84f9c2be5479c0f869df&sharer_shareinfo_first=f565de86bc2c84f9c2be5479c0f869df#rd
---

性能问题往往有各种不同的表象, 也有各种不同的原因. 但是细细分析, 它们往往是有不同的模式. 本文将从一个具体的例子出发, 分析一个性能问题的原因, 并给出一种较通用的解决方案.

# 问题描述

本周接到一个客户反馈, 有个ETL的CPU占用非常高, 但是数据量并不大, 为什么会这样呢?

首先, 了解了一下客户的实际场景, ETL本身有不少复杂逻辑, 但是出问题的节点很快识别为一个"不等值Join". 具体场景是, 对于每个访客, 根据其IP地址, 通过IP地址库查出其所在的城市.

简化一下, 我们可以认为系统中有两个表, 一个是"访客表", 其中有访客的ID和IP地址, 另一个是"IP地址库", 其中有IP地址和城市的对应关系, 由于IP地址是非常多的, 所以, 其使用了一个IP地址段的概念, 也就是说, 一个IP地址库中的一条记录, 其IP地址是一个范围, 而不是一个具体的IP地址.

比如:

| ip\_start | ip\_end | city |
| --- | --- | --- |
| 40.73.225.0 | 40.73.226.255 | Shanghai |

那这样, 我们查找访客IP所对应的城市, 就需要做一个类似的JOIN

```
SELECT visitor.id, visitor.ip, ip_lookup.city  
FROM visitor  
 LEFT OUTER JOIN ip_lookup  
ON visitor.ip >= ip_lookup.ip_start   
  AND visitor.ip <= ip_lookup.ip_end
```

注意: 这里由于Spark本身不支持"IP地址"这种数据类型, 而只能当String来处理, 所以这里面其实不能简单的 <= 或者 >=, 而是需要转为整数, 然后再比较.

比如: 我们可以定义一个类似的SQL Macros (Spark原生不支持Macros, 但是这里只是示意)

```
{% macro ip_text_to_number(ip_column) %}  
    (CAST(split(ip_column, '\\.')[0] AS BIGINT) * 256 * 256 * 256 +  
       CAST(split(ip_column, '\\.')[1] AS BIGINT) * 256 * 256 +  
       CAST(split(ip_column, '\\.')[2] AS BIGINT) * 256 +  
       CAST(split(ip_column, '\\.')[3] AS BIGINT))      
{% endmacro %}
```

后面的SQL中, 如果结尾是 \_num, 比如: ip\_start\_num, 那么就是调用这个Macros, 将IP地址转为整数后的列了.

当前的问题是: 这个SQL非常慢. 而且似乎这个不等值的JOIN是难以避免的.

# 问题重现

由于这个ETL非常慢, 我们最好不要在客户的环境上进行调试, 所以, 我们需要首先在Spark上重现这个性能问题.

首先, 网上搜索一些"IP地址库"数据集, 比如, 在 Kaggle 上找到了一个 "Global IP Dataset by Location 2023": https://www.kaggle.com/datasets/joebeachcapital/global-ip-dataset-by-location-2023?select=geolite2-city-ipv4.csv 这个IP地址库对于国内的IP地址范围并不准确, 但是对于我们的内部复现来说是没什么影响的.

这个数据集有大概282万行数据. 首先, 把这个CSV数据集加工为Parquet格式的文件并赋予列名, 方便后续的Spark进行处理. 这里我使用了Duckdb来处理(用 Python 的 pandas 库处理应该也是类似的):

```
COPY (  
  SELECT *  
  FROM read_csv_auto('geolite2-city-ipv4.csv',  
     names=['ip_start', 'ip_end', 'country_code', 'state', 'sub_state',  
         'city', 'post_code', 'latitude', 'longitude', 'timezone'])  
) TO 'ip_lookup.parquet' (FORMAT PARQUET)
```

接下来我们准备一些访客数据, 这里我使用了一个随机生成的访客数据(只有id 和 ip 两列) , 只生成了 1000 行数据, 因为上面的不等值JOIN实在是太慢了, 1000行数据就已经可以重现问题了.

```
COPY (  
  SELECT UNNEST(RANGE(0, 1 * 1000)) as id,  
       HOST('255.255.255.255'::INET - cast(random() * 256*256*256*256 as long)) as ip  
) TO 'visitor.parquet' (FORMAT PARQUET)
```

生成的数据类似于:

| id | ip |
| --- | --- |
| 0 | 196.116.173.6 |
| 1 | 95.118.87.33 |
| 2 | 113.157.61.202 |

然后在我的本地macbook上, 使用Spark(使用了4核CPU)运行如下SQL, 运行时间需要大概1分钟, 考虑到vistor只有1000行, 这个速度非常慢, 问题得以复现.

```
WITH visitor_num AS (  
    SELECT *, iptext_to_number(ip) AS ip_num  
    FROM visitor  
), ip_lookup_num AS (  
   SELECT *, iptext_to_number(ip_start) AS ip_start_num,  
       iptext_to_number(ip_end) AS ip_end_num,  
    FROM ip_lookup  
)  
SELECT visitor_num.*, ip_lookup_num.city  
FROM visitor_num LEFT OUTER JOIN ip_lookup_num  
ON visitor_num.ip_num >= ip_lookup_num.ip_start_num  
  AND visitor_num.ip_num <= ip_lookup_num.ip_end_num
```

为什么会这么慢? 如果是技术的角度, 可以解释为, 这里的只有 >= 和 <= 作为关联条件的JOIN, Spark只能使用"Broadcast Nested Loop Join"来完成Join, 而这种Join的性能是非常差的. 但是这里为了让更多的人能明白, 就先忽略掉纯技术的解释. 由于没有"等值条件", 对于"vistor"访客表中的每一个IP, 我们都需要遍历"ip\_lookup"表的每一行记录, 并进行 >= 和 <= 的比较, 这样的话, 1000行数据每一行都需要遍历282万行数据, 这样的性能是非常差的.

# 尝试1, 是否能额外增加一个等值条件?

考虑到每个不同的IP, 最多只会关联到 ip\_lookup 表的一条记录(一个ip范围: [ip\_start, ip\_end]), 所以, 我们是否可以增加一个等值条件, 让Spark能够使用"Broadcast Hash Join"来完成Join呢, 从而每个IP只需要查找ip\_lookup表的一条记录, 而不是遍历所有的记录呢?

观察到ip\_lookup表的数据:

| ip\_start | ip\_end | city |
| --- | --- | --- |
| 40.73.225.0 | 40.73.226.255 | Shanghai |

我们似乎可以去掉ip地址的最后一位, 而是使用ip地址的前三位来进行等值JOIN. 这样我们创建一个新列 ip\_base\_num, 来代表ip地址的前三位. 但是需要额外注意的是, 这个 ip\_start, 和 ip\_end 可能对应于多个区间. 所以我们还需要使用Spark的 explode 来把一行变为多行. 使用SQL如下:

```
SELECT *, EXPLODE(  
  SEQUENCE( floor(ip_start_num/256), floor(ip_end_num/256) )  
) AS ip_base_num  
FROM ip_lookup
```

这样我们就可以得到如下的结果:

| ip\_start | ip\_end | city | ip\_base\_num |
| --- | --- | --- | --- |
| 40.73.225.0 | 40.73.226.255 | Shanghai | 2640353 |
| 40.73.225.0 | 40.73.226.255 | Shanghai | 2640354 |

然后我们可以用这个 ip\_base\_num 来进行等值JOIN了, 使用SQL如下:

```
WITH visitor_num AS (  
    SELECT *, iptext_to_number(ip) AS ip_num  
    FROM visitor  
), ip_lookup_num AS (  
   SELECT *, iptext_to_number(ip_start) AS ip_start_num,  
       iptext_to_number(ip_end) AS ip_end_num,  
   FROM ip_lookup  
), ip_lookup_num_with_base AS(  
   SELECT *, EXPLODE(  
       SEQUENCE( floor(ip_start_num/256), floor(ip_end_num/256) )  
   ) AS ip_base_num  
   FROM ip_lookup_num  
)  
SELECT visitor_num.*, ip_lookup_num_with_base.city  
FROM visitor_num LEFT OUTER JOIN ip_lookup_num_with_base  
ON floor(visitor_num.ip_num / 256) = ip_lookup_num_with_base.ip_base_num
```

执行时间由1分钟降低到了7秒钟. 感觉是个不小的提升. 但是, 遗憾的是, 当我们选择结果的行数的时候, 发现这个SQL的结果是错误的, 我们得到了 1020 行结果, 而不是预期的 1000 行结果.

# 尝试2, 既有等值条件, 又保留不等值条件

分析一下为什么会造成结果比预期的多. 会发现ip v4的地址段并不是按照前3位大片大片的分配的, 毕竟 IPv4 的地址数量有限. 导致一些类似于 ["17.85.6.64", "17.85.6.71"] 这种半段的区间.

这会导致 同一个 ip\_base\_num 可能对应到多个 ip\_start 和 ip\_end. 那是否尝试1中的等级条件就没用了, 不, 它们仍能起到筛选作用, 只不过, 我们仍然需要最早的那个 >= 和 <= 做一下二次过滤, 最终我们得到正确且高效的SQL(区别在最后两行)如下:

```
WITH visitor_num AS (  
    SELECT *, iptext_to_number(ip) AS ip_num  
    FROM visitor  
), ip_lookup_num AS (  
   SELECT *, iptext_to_number(ip_start) AS ip_start_num,  
       iptext_to_number(ip_end) AS ip_end_num,  
   FROM ip_lookup  
), ip_lookup_num_with_base AS(  
   SELECT *, EXPLODE(  
       SEQUENCE( floor(ip_start_num/256), floor(ip_end_num/256) )  
   ) AS ip_base_num  
   FROM ip_lookup_num  
)  
SELECT visitor_num.*, ip_lookup_num_with_base.city  
FROM visitor_num LEFT OUTER JOIN ip_lookup_num_with_base  
ON floor(visitor_num.ip_num / 256) = ip_lookup_num_with_base.ip_base_num  
  AND visitor_num.ip_num >= ip_lookup_num_with_base.ip_start_num  
  AND visitor_num.ip_num <= ip_lookup_num_with_base.ip_end_num
```

# 总结

当Spark中遇到没有等值条件的JOIN时, 我们就需要考虑一下是否会造成"Nested Loop Join"这种低效的JOIN方式, 从而使得SQL大大变慢. 如果是, 我们就需要考虑一下是否能增加额外的等值条件, 有了这个额外的等值条件, 你的SQL也许就能变快非常多了. 从而为环保事业做出贡献.