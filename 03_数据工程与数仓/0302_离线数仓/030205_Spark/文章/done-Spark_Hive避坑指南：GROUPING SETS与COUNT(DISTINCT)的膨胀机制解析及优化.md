> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL聚合去重与膨胀治理|SQL聚合去重与膨胀治理]]、[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_版本记录|030204_版本记录]]
---
title: Spark/Hive避坑指南：GROUPING SETS与COUNT(DISTINCT)的膨胀机制解析及优化
author: 数据打工人的自我修养
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247484010&idx=1&sn=297f3e573c1a50b7f44ca521f2632bb4&chksm=971d16a9d54094c06697fa76087e3c11e0183ce123086c7759a4a2ac2b1c1945b59d233b194f&mpshare=1&scene=24&srcid=0901TJIPpegmyUeYvbvotBzP&sharer_shareinfo=2841d283f62f07560f94d3b600e26038&sharer_shareinfo_first=2841d283f62f07560f94d3b600e26038#rd
---

聚合查询，尤其是带多个 `COUNT(DISTINCT)` 和多维聚合（诸如grouping sets）的场景，是大数据计算中最容易踩坑的性能瓶颈之一。本文将通过三个典型场景，结合 **Hive** 与 **SparkSQL** 引擎的底层执行机制，配合示例数据和表格，帮你看懂从 Map 端到 Reduce 端的每一步变化，并提供优化建议。

各大计算引擎在持续优化，如下基于hive2节spark3版本示例，仅供参考，具体请以使用版本为准。

以下内容以SparkSQL示例展开，HIVE在多维聚合诸如grouping sets以及多个countdistinct去重计数场景，数据膨胀逻辑整体与SparkSQL一致，这里不多做描述。

## 一、示例数据集

##

## 基于如下样例6行记录复制100w次，共600w行；表名：user\_log）

##

## 二、SparkSQL 引擎执行机制

###

### 场景 1：GROUP BY + 多个 COUNT(DISTINCT)

```
SELECT age  ,COUNT(DISTINCT user_id) AS uv  ,COUNT(DISTINCT product_id) AS pvFROM user_logGROUP BY age;
```

执行计划：不同于HIVE，SparkSQL的去重计数，物理执行计划会优化成两个MR执行

```
== Physical Plan ==AdaptiveSparkPlan isFinalPlan=false+- HashAggregate(keys=[age#199L], functions=[count(user_log.user_id#219L), count(user_log.product_id#220)])   +- Exchange hashpartitioning(age#199L, 200), ENSURE_REQUIREMENTS, [plan_id=718]      +- HashAggregate(keys=[age#199L], functions=[partial_count(user_log.user_id#219L) FILTER (WHERE (gid#218 = 1)), partial_count(user_log.product_id#220) FILTER (WHERE (gid#218 = 2))])         +- HashAggregate(keys=[age#199L, user_log.user_id#219L, user_log.product_id#220, gid#218], functions=[])            +- Exchange hashpartitioning(age#199L, user_log.user_id#219L, user_log.product_id#220, gid#218, 200), ENSURE_REQUIREMENTS, [plan_id=714]               +- HashAggregate(keys=[age#199L, user_log.user_id#219L, user_log.product_id#220, gid#218], functions=[])                  +- Expand [[age#199L, user_id#198L, null, 1], [age#199L, null, product_id#201, 2]], [age#199L, user_log.user_id#219L, user_log.product_id#220, gid#218]                     +- Project [user_id#198L, age#199L, product_id#201]                        +- Scan ExistingRDD[user_id#198L,age#199L,city#200,product_id#201]
```

执行逻辑类同如下SQL：

```
with tmp as (  -- 第一个MR  select distinct age,user_id,product_id,group_flag  from (  -- 数据膨胀两倍，这里使用explode供参考    select age       ,case when group_flag=2 then null else user_id end as user_id         ,case when group_flag=1 then null else product_id end as product_id       ,group_flag     from user_log     LATERAL VIEW EXPLODE(array(1,2)) t AS group_flag   -- 数据膨胀两倍  ) t) -- 第二个MR:全局聚合 select age   ,count(case when group_flag=1 then user_id end) as uv    ,count(case when group_flag=2 then product_id end) as pv  from tmp group by age 
```

Expand：两个countdistinct，数据膨胀两倍，数据由600w变成1200w

```
Expand [[age#199L, user_id#198L, null, 1], [age#199L, null, product_id#201, 2]]
```

**`HashAggregate：聚合，Exchange前是map端预聚合（第一次预聚合后58行），Exchange后是reduce聚合``。`**

**`如果不是最后结果输出的Stage和最开始的TableScan Stage，一个Stage里，SparkSQL处理逻辑 = HIVE的Reduce逻辑 + 下游Map逻辑，HIVE在处理Reduce后会写磁盘。``Exchange：shuffle环节，hashpartitioning字段是分区字段。`**

**执行过程与数据变化**

SparkSQL在处理countdistinct时，会自动将sql物理执行计划优化成两个MR，无须像HIVE那样改写两个MR去处理倾斜。

当执行如下sql-1时：

```
select age  ,count(distinct user_id) as user_cnt from user_log group by age
```

### SparkSQL等价于执行了如下sql-2：

```
select age,count(1) as cnt from (  select age,user_id  from user_log   group by age,user_id) t group by age 
```

### 而在hive中，sql-1只有一个MR

```
-- 查看hive执行计划explain select age  ,count(distinct user_id) as user_cnt from user_log group by age;
```

###

### 基于同个字段不同条件的countdistinct优化：

### 样例：按城市分组，不同年龄段用户数

```
select city  ,count(distinct case when age=18 then user_id end) as user_cnt_18  ,count(distinct case when age=19 then user_id end) as user_cnt_19  ,count(distinct case when age=20 then user_id end) as user_cnt_20  ,count(distinct case when age=21 then user_id end) as user_cnt_21  ,count(distinct case when age=22 then user_id end) as user_cnt_22  ,count(distinct case when age=23 then user_id end) as user_cnt_23  ,count(distinct case when age=24 then user_id end) as user_cnt_24from user_log group by city 
```

### 7个countdistinct字段，每个膨胀1倍，数据map端总膨胀7倍。

### 当涉及此类情况，countdistinct对同个字段去重计数字段过多，极其推荐手动改写为两个MR，改写后的示例如下：

```
select city  ,sum(case when is_18_user=1 then 1 else 0 end) as user_cnt_18  ,sum(case when is_19_user=1 then 1 else 0 end) as user_cnt_19  ,sum(case when is_20_user=1 then 1 else 0 end) as user_cnt_20  ,sum(case when is_21_user=1 then 1 else 0 end) as user_cnt_21  ,sum(case when is_22_user=1 then 1 else 0 end) as user_cnt_22  ,sum(case when is_23_user=1 then 1 else 0 end) as user_cnt_23  ,sum(case when is_24_user=1 then 1 else 0 end) as user_cnt_24from (  select city,user_id     ,max(case when age=18 then  1 else 0 end) as is_18_user    ,max(case when age=19 then  1 else 0 end) as is_19_user    ,max(case when age=20 then  1 else 0 end) as is_20_user    ,max(case when age=21 then  1 else 0 end) as is_21_user    ,max(case when age=22 then  1 else 0 end) as is_22_user    ,max(case when age=23 then  1 else 0 end) as is_23_user    ,max(case when age=24 then  1 else 0 end) as is_24_user  from user_log   group by city,user_id ) t  group by city
```

### 优化说明：多了一个mr，没有膨胀。针对海量用户id去重场景，该方式能显著减少单Task内存负载，显著减少运行时间。

### 场景 2：多维聚合（GROUPING SETS）

```
SELECT age, city, COUNT(*) AS cntFROM user_logGROUP BY age, cityGROUPING SETS ((age, city), (age), ());
```

执行计划：Expand数据膨胀了三倍，600w \* 3 = 1800w行

```
== Physical Plan ==AdaptiveSparkPlan isFinalPlan=false+- HashAggregate(keys=[age#317L, city#318, spark_grouping_id#316L], functions=[count(1)])   +- Exchange hashpartitioning(age#317L, city#318, spark_grouping_id#316L, 200), ENSURE_REQUIREMENTS, [plan_id=1100]      +- HashAggregate(keys=[age#317L, city#318, spark_grouping_id#316L], functions=[partial_count(1)])         +- Expand [[age#257L, city#258, 0], [age#257L, null, 1], [null, null, 3]], [age#317L, city#318, spark_grouping_id#316L]            +- Project [age#257L, city#258]               +- Scan ExistingRDD[user_id#256L,age#257L,city#258,product_id#259]
```

grouping sets有三个组合，数据膨胀3倍到1800w行（有几个组合膨胀几倍）。每一个分组会生成一个groupingid标识，如下的：0,1,3

```
Expand [  [age#257L, city#258, 0],   [age#257L, null, 1],   [null, null, 3]]
```

map预聚合后：81行记录

### map端预聚合的记录行数 = 各个map task预聚合后的记录数加总。

### 案例中预聚合后总81行记录，map task总24个

###

### 场景 3：多维聚合 + 多个 COUNT(DISTINCT)

```
SELECT age, city, COUNT(*) AS cnt  ,count(distinct case when product_id='P1' then user_id end) as user_cnt_1   ,count(distinct case when product_id='P2' then user_id end) as user_cnt_2   ,count(distinct case when product_id='P3' then user_id end) as user_cnt_3 FROM user_logGROUP BY age, cityGROUPING SETS ((age, city), (age), ());
```

膨胀后的数据 = 原始记录行：600w \* 多维组合数：3 \* countdistinct字段个数+1（如存在非去重聚合，比如这里的count\*）:4 = 7200w

**特点**

* 膨胀倍数 ≈ 原始行数 × 分组组合数 M × DISTINCT 数量 N + 1（如存在非去重聚合字段）
* Map 端预聚合可以缓解但不彻底

### 三、数据膨胀机制与核心风险

###

1. 膨胀原理

* 多字段**`COUNT DISTINCT`**：

SparkSQL会通过Expand操作将数据复制N份（N为去重字段数），每份标记`gid`区分字段，再按`gid`分组聚合。例如3个去重字段会使数据膨胀3倍，导致Shuffle和内存压力剧增。

* 多维聚合GROUPING SETS：

每个维度组合独立计算，若维度组合多（如22种），数据可能膨胀数十倍，且GROUPING SETS同样依赖Expand机制。

2. 主要风险

* 网络IO与内存OOM：膨胀后数据量远超输入，易导致Shuffle溢出和Executor内存不足。
* **长尾任务**：单个Task处理膨胀数据时延远高于其他Task，拖慢整体作业
* 资源浪费：预聚合失效时，高并发Task消耗大量CPU和内存。

### 四、多字段`COUNT DISTINCT`优化方案

###

#### 1. **分步计算法（拆解JOIN）：去重字段无强关联时，先拆分去重，再按groupby字段join。避免Expand，分治降低单Task压力。**

#### 2. **同字段多条件优化：同一字段的多个条件统计（如COUNT(DISTINCT CASE WHEN ...)，改写成两个MR，通过预聚合减少膨胀，转化为常规**

#### 3. **中间结果物化：将预处理结果（如去重后的中间表）写入临时表，后续查询直接读取，避免重复计算，尤其适用于多次复用场景。**

#### **4. 使用近似计算：APPROX\_COUNT\_DISTINCT替代精确去重，误差约0.1%~1%，但性能提升10倍+，**

#### **5. 精确计算使用bitmap去重：类似**APPROX\_COUNT\_DISTINCT，内存消耗相比高些。该数据结构能显著加速去重计数效率。****

####

### 五、多维聚合`GROUPING SETS`优化方案

###

#### 1. **按需裁剪维度组合：用GROUPING SETS替代CUBE：仅保留必要维度组合，减少计算量。组合数从指数级（2ⁿ）降至线性，减少膨胀 ``` -- 原始CUBE（8种组合）GROUP BY CUBE(a, b, c);-- 优化后（仅需4种组合）GROUP BY GROUPING SETS ((a,b), (a,c), (a), ()); ```**

#### 2. **拆分UNION ALL：维度组合过多且无法精简时，将GROUPING SETS拆为多个子查询，通过UNION ALL合并结果。避免单Job膨胀，但增加Job数（需权衡资源）。**

### 五、执行引擎调优策略

###

#### 1. **减小Map输入：设置spark.sql.files.maxPartitionBytes（**Spark，**如128MB → 64MB），降低单个Task处理基数，即使膨胀后仍在内存阈值内**

#### 2. **增加Reduce/Partition个数：通过spark.sql.shuffle.partitions（Spark）或mapred.reduce.tasks（Hive）提高分区数，分散膨胀数据。需注意，hive的reduce个数动态设置hive.exec.reducers.bytes.per.reducer，是根据TableScan表数据大小来评估的。如果预聚合能显著减少数据量，可视情况减少reduce个数。**

#### 3.增加内存资源配置：以免数据膨胀Task内存OOM。

---

SQL文章推荐：

[当我在谈SQL优化时，我谈些什么](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483791&idx=1&sn=57d50a169e342b6bdf84a5df76f415d4&scene=21#wechat_redirect)

[SQL跑不动？数据倾斜惹的祸！全场景优化方案大揭秘](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483826&idx=1&sn=de5d8134932605da5badc3e1fa8855c8&scene=21#wechat_redirect)

[零部署本地SQL神器：基于DuckDB的文件直查GUI，千万数据秒级分析](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483909&idx=1&sn=e68f1eab098f2cd96f59960fd7d9c5db&scene=21#wechat_redirect)

[挑战你的SQL基本功：NULL和JOIN，你真的会用吗？](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483865&idx=1&sn=616fe7373db1387a05ceac869e8d9228&scene=21#wechat_redirect)

[SparkSQL性能优化：你必须掌握的关键资源参数](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483847&idx=1&sn=91cbba52dfac89372833ab4510d1b64f&scene=21#wechat_redirect)

[效率飙升！数分师的AI新玩具：打造SQL知识库，解锁智能下钻自由](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483785&idx=1&sn=224276f5b32c123bfd6daf0624b3f2c7&scene=21#wechat_redirect)

[SQL语句优化：稍作修改，计算耗时节省80%以上](https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483810&idx=1&sn=c0e60d8d0000ce025622c2278a8f65d9&scene=21#wechat_redirect)

---

>>>>>>>>>> ✧ END ✧ <<<<<<<<<<

感谢阅读，点击下方关注我，谢谢支持！