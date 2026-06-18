---
title: Hive SQL优化思路
author: 浪尖聊大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzUyMDA4OTY3MQ==&mid=2247513646&idx=1&sn=cba29b3677676e7f1f04df9dfdefaf5f&chksm=f86e08ccb2cb7ec5f6bf2b2aaaa04f8e3dd54f7d1371dec4b96bb38ca29592fe6e83c60f8ed6&mpshare=1&scene=24&srcid=05298vAGgdmcX8r9areKJqBm&sharer_shareinfo=ac8ab4927858067999417d781fc66f7c&sharer_shareinfo_first=ac8ab4927858067999417d781fc66f7c#rd
---

---

Hive的优化主要分为：配置优化、SQL语句优化、任务优化等方案。其中在开发过程中主要涉及到的可能是SQL优化这块。

优化的核心思想是：

* 减少数据量（例如分区、列剪裁）
* 避免数据倾斜（例如加参数、Key打散）
* 避免全表扫描（例如on添加加上分区等）
* 减少job数（例如相同的on条件的join放在一起作为一个任务）

## HQL语句优化

### 1. 使用分区剪裁、列剪裁

在分区剪裁中，当使用外关联时，如果将副表的过滤条件写在Where后面，那么就会先全表关联，之后再过滤。

```
select a.*    
from a    
left join b on  a.uid = b.uid    
where a.ds='2020-08-10'    
and b.ds='2020-08-10'
```

**上面这个SQL主要犯了两个错误**：

1. 副表(上方b表)的where条件写在join后面，会导致先全表关联在过滤分区。

注：虽然a表的where条件也写在join后面，但是a表会进行谓词下推，也就是先执行where条件，再执行join，但是b表不会进行谓词下推！

1. on的条件没有过滤null值的情况，如果两个数据表存在大批量null值的情况，会造成数据倾斜。

**正确写法**：

```
select a.*    
from a    
left join b on (d.uid is not null and a.uid = b.uid and b.ds='2020-08-10')   
where a.ds='2020-08-10'
```

如果null值也是需要的，那么需要在条件上转换，或者单独拿出来

```
select a.*    
from a    
left join b on (a.uid is not null and a.uid = b.uid and b.ds='2020-08-10')    
where a.ds='2020-08-10'    
union all    
select a.* from a where a.uid is null 
```

或者：

```
select a.*    
from a    
left join b on     
case when a.uid is null then concat("test",RAND()) else a.uid end = b.uid and b.ds='2020-08-10'    
where a.ds='2020-08-10'
```

或者（子查询）：

```
select a.*    
from a    
left join     
(select uid from where ds = '2020-08-10' and uid is not null) b on a.uid = b.uid   
where a.uid is not null    
and a.ds='2020-08-10'
```

### 2. 尽量不要用COUNT DISTINCT

因为COUNT DISTINCT操作需要用一个Reduce Task来完成，这一个Reduce需要处理的数据量太大，就会导致整个Job很难完成，一般COUNT DISTINCT使用先GROUP BY再COUNT的方式替换，虽然会多用一个Job来完成，但在数据量大的情况下，这个绝对是值得的。

```
select count(distinct uid)    
from test    
where ds='2020-08-10' and uid is not null  
```

转换为：

```
select count(a.uid)    
from     
(select uid   
fromtest  
where uid isnotnulland ds = '2020-08-10'  
groupby uid  
) a
```

### 3. 使用with as

拖慢Hive查询效率除了join产生的shuffle以外，还有一个就是子查询，在SQL语句里面尽量减少子查询。with as是将语句中用到的子查询事先提取出来（类似临时表），使整个查询当中的所有模块都可以调用该查询结果。使用with as可以避免Hive对不同部分的相同子查询进行重复计算。

```
select a.*    
from  a    
left join b on  a.uid = b.uid    
where a.ds='2020-08-10'    
and b.ds='2020-08-10'  
```

可以转化为：

```
with test1 as  
(  
select uid    
from b    
where ds = '2020-08-10'and uid isnotnull  
)    
select a.*    
from a    
leftjoin test1 on a.uid = test1.uid    
where a.ds='2020-08-10'and a.uid isnotnull
```

### 4. 大小表的join

写有Join操作的查询语句时有一条原则：**应该将条目少的表/子查询放在Join操作符的左边**。原因是在Join操作的Reduce阶段，位于Join操作符左边的表的内容会被加载进内存，将条目少的表放在左边，可以有效减少发生OOM错误的几率。但新版的hive已经对小表JOIN大表和大表JOIN小表进行了优化。小表放在左边和右边已经没有明显区别。不过在做join的过程中通过小表在前可以适当的减少数据量，提高效率。

### 5. 数据倾斜

数据倾斜的原理都知道，就是某一个或几个key占据了整个数据的90%，这样整个任务的效率都会被这个key的处理拖慢，同时也可能会因为相同的key会聚合到一起造成内存溢出。

数据倾斜只会发生在shuffle过程中。这里给大家罗列一些常用的并且可能会触发shuffle操作的算子：distinct、 groupByKey、reduceByKey、aggregateByKey、join、cogroup、repartition等。出现数据倾斜时，可能就是你的代码中使用了这些算子中的某一个所导致的。

**hive的数据倾斜一般的处理方案**：

常见的做法，通过参数调优：

```
set hive.map.aggr=true;    
set hive.groupby.skewindata = ture;
```

当选项设定为true时，生成的查询计划有两个MapReduce任务。

在第一个MapReduce中，map的输出结果集合会随机分布到reduce中，每个reduce做部分聚合操作，并输出结果。

这样处理的结果是，相同的Group By Key有可能分发到不同的reduce中，从而达到负载均衡的目的；

第二个MapReduce任务再根据预处理的数据结果按照Group By Key分布到reduce中（这个过程可以保证相同的Group By Key分布到同一个reduce中），最后完成最终的聚合操作。

但是这个处理方案对于我们来说是个黑盒，无法把控。

一般处理方案是将对应的key值打散即可。

例如：

```
select a.*    
from a    
left join b on  a.uid = b.uid    
where a.ds='2020-08-10'    
and b.ds='2020-08-10'  
```

如果有90%的key都是null，这样不可避免的出现数据倾斜。

```
select a.uid    
from test1 as a    
join(    
   select case when uid is null then cast(rand(1000000) as int)    
   else uid    
   from test2 where ds='2020-08-10') b     
on a.uid = b.uid    
where a.ds='2020-08-10'  
```

当然这种只是理论上的处理方案。

正常的方案是null进行过滤，但是日常情况下不是这种特殊的key。

那么在日常需求的情况下如何处理这种数据倾斜的情况呢：

1. sample采样，获取哪些集中的key；
2. 将集中的key按照一定规则添加随机数；
3. 进行join，由于打散了，所以数据倾斜避免了；
4. 在处理结果中对之前的添加的随机数进行切分，变成原始的数据。

当然这些优化都是针对SQL本身的优化，还有一些是通过参数设置去调整的，这里面就不再详细描述了。

但是优化的核心思想都差不多：

1. 减少数据量；
2. 避免数据倾斜；
3. 减少JOB数；
4. 核心点：根据业务逻辑对业务实现的整体进行优化；
5. 解决方案：采用presto、impala等专门的查询引擎，采用spark计算引擎替换MR/TEZ。