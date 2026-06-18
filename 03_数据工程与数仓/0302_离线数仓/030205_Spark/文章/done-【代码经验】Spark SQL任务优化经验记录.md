> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_核心知识点/SparkSQL业务逻辑优化方法|SparkSQL业务逻辑优化方法]]
---
title: 【代码经验】Spark SQL任务优化经验记录
author: 傻文和靓燕
date:
url: https://mp.weixin.qq.com/s?__biz=MzAwOTkwNzI4Ng==&mid=2247483903&idx=1&sn=cf603112ed4ba4e7ddeae3c17e5993e7&chksm=9a777047048bbd61a7629e55a3712256e206b19344f4cf31492e51bcad374c7cefa3ecd35a17&mpshare=1&scene=24&srcid=0901xYTiGZaR1HESCFNSfEF1&sharer_shareinfo=7ecc191af35986b3aa1251d1aa20f8ef&sharer_shareinfo_first=7ecc191af35986b3aa1251d1aa20f8ef#rd
---

由于我负责组内的数据质量管理和夜间任务质量管理工作，接触的异常任务比较多，总结出一些代码优化的经验，在这里做简单记录和分享。

## 可能导致数据异常问题

### 1、时间周期过滤建议闭合

想要限制日期参数从某一个时间到现在，建议不要使用day>时间参数，可以使用day between 时间参数 and current\_date。

开放式的时间过滤写法在任务回溯时，可能会导致数据统计问题。如下代码中，因为回溯所需要的时间是2021-01-01到回溯日的时间周期，开放式的周期对应的是2021-01-01到现在。

```
-- 不建议写法，这种写法在任务回溯的时候，会导致数据问题。
select
    *
from table
where day>'2021-01-01'

-- 建议写法：使用between 或者day> and day<，时间过滤要写的具体完整。
select
    *
from table
where day between '2021-01-01' and '当前时间参数'
```

### 2、full join问题

full join的时候，要注意关联键是否为null，因为两侧表的某个关联键都为NULL的时候，是无法进行关联的，会导致主键重复结果。

```
t1的数据(null,a;1,b);t2的数据(null,a;2,b)
```

```
-- 原写法
select
    nvl(t1.id,t2.id) as id,
    nvl(t1.type,t2.type) as type
    ...
from t1 
full outer join t2 
on t1.id=t2.id
and t1.type=t2.type
--关联之后的结果为：
(null,a;null,a;1,b;2,b)

-- 建议写法
select
    nvl(t1.id,t2.id) as id,
    nvl(t1.type,t2.type) as type
    ...
from t1 
full outer join t2 
on nvl(t1.id,'')=nvl(t2.id,'')
and nvl(t1.type,'')=nvl(t2.type,'')
--关联之后的结果为：
(null,a;1,b;2,b)
```

### 3、grouping sets问题

grouping sets的时候，使用GROUPING\_\_ID判断维度是否生效的时候，不要直接使用是否为null判断，建议使用 GROUPING\_\_ID&1=0判断。

如下代码案例中，如果type有null值存在时，nvl()写法中null的数据会计算到all值维度中，导致最终结果有所偏差。

```
-- 不建议写法
select
    id,
    nvl(type,'all') as type
from table
group by (id,type)
grouping sets(
    (id,type),
    (id)
)

-- 建议写法
select
    id,
    -- 关于GROUPING__ID 值的问题，Spark sql和Hive sql可能有语法区别，可自行查询
    if(cast(GROUPING__ID as tinyint)&2=0,'all',type) as type 
from table
group by (id,type)
grouping sets(
    (id,type),
    (id)
)
```

### 4、聚合key是否唯一

例如想要对用户进行聚合时，如果对(id,name)两个维度进行聚合，相同id可能对应多个name，就会导致聚合结果出错。聚合时需要明确主键，确保主键是唯一的。

这个问题看似很简单，或者不值一提，可是实际工作中真的碰到过不止一次，还是有必要记录的，警醒自己！

## 任务优化

问题点：执行时间长，经常失败。
必不可少：**观察Spark UI！！！**

### 1、两表关联前，提前进行过滤

老生常谈，Shuffle操作前，对数据进行必要的过滤。尽管这个优化点最常提到，但在代码中还是会存在未提前过滤的情况出现。

### 2、with cube 转为多个union all；count(distinct id) 转为size(collect\_set(id))

优化效果非常棒，具体可查看该文章[从2小时到3分钟：Spark SQL多维分析性能优化实战](https://mp.weixin.qq.com/s?__biz=Mzg2Mjc0MDMxNQ==&mid=2247483777&idx=1&sn=2a83002890c8ce76d1dafe6b7882468f&scene=21#wechat_redirect)。

### 3、数据源选择，选择可替代的粗粒度的数据，或者创建中间层提前对数据进行聚合。

该优化点和数仓使用规范的一个点相似，避免跨层引用，da层尽量使用dws层，da表在进行指标计算时，尽量使用聚合后的数据，减少冗余计算。

### 4、对于过滤后数据量满足广播阈值的数据，明确使用broadcast 的hint。

```
select /*+ BROADCAST(t2) */
    t1.id
from t1 
left join (
    select
        id
    from t2 
    group by id
) as t2
on t1.id=t2.id
```

### 5、特定的导致数据倾斜的key提取出来，单独计算。

如果导致数据倾斜的Key是固定不变的，则可以使用该方案，将数据量较大的Key提取出来单独处理。

### 6、小文件问题

详细可查看小文件对应文章[【数仓】小文件的问题及一些解决方案](https://mp.weixin.qq.com/s?__biz=MzAwOTkwNzI4Ng==&mid=2247483883&idx=1&sn=6ed5c6ceb8b9d2c5d0f1746535f7142e&scene=21#wechat_redirect)

### 7、distribute by 主键+随机值，将数据提前按照主键打散

对于出现了数据倾斜情况的任务，可使用该方法进行调整，作用不明显，因为该方法会对所有的Key进行重分区。

```
select
    id
from (
    select
        id,、、、
    from table 
    distribute by concat(id,rand()*10) -- 将id随机打散为10份
) as t1
group by 
id
```

### 8、公共逻辑如果下游使用较多的情况，使用临时表，不要使用with as

```
-- 原写法
with tmp as (
    select ...
)

select 
    
from tmp...

select from tmp...

-- 建议写法
create table tmp as 
select ...

select

from tmp ...

select

from tmp ...
```

### 9、灵活使用repartition 的hint和spark.sql.shuffle.partitions

如果某Stage中的task很多或者很少，可以通过调整spark.sql.shuffle.partitions的值来调整并发。

如果数据源表存在大量小文件，导致计算中task数量很大，则可以使用repartition的hint进行处理。

## 关于数据倾斜

经常被提到的数据倾斜问题，本文提到的解决方法很少，因为在我的实际工作中，确实遇到过数据倾斜的问题，但是导致数据倾斜的Key通常都**不是固定**的，今天某个Key可能由于某个业务活动出现倾斜了，但是明天又不倾斜了，如果我今天对这个数据倾斜问题做一番优化，可能在明天的数据场景中就已经不适用了，这也是一直困扰我的问题。

数据倾斜问题就像是一支敌人的游击队，不确定某天就从某个地方钻出来打两枪然后迅速躲藏。

## 总结

以上是我的一些浅显的工作经验，这里做一下记录，刚开始准备材料的时候感觉东西很多，但是整理之后发现内容不够充实。对于任务优化相关的问题我会持续思考，持续整理。

如果有任何问题，可评论或者私信。