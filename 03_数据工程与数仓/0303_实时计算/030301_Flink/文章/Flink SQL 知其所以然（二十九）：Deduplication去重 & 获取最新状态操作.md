---
title: Flink SQL 知其所以然（二十九）：Deduplication去重 & 获取最新状态操作
author: Flink
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI0NTIxNzE1Ng==&mid=2651225662&idx=1&sn=dfd4078b778d466f224f4b71d43ab29c&chksm=f2a33cd5c5d4b5c396b4b26eb48bd462d9a5a147e8df64d4d3e25a68c1a6afcf867f9d29ea45&mpshare=1&scene=24&srcid=07188kOIhqsRIWmI2POKY0nu&sharer_sharetime=1658103451177&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

## DML：Deduplication

大家好，今天我们来学习 Flink SQL 中的 Deduplication 去重以及如何通过 Deduplication 操作获取最新的状态。

1. ⭐ Deduplication 定义（支持 Batch\Streaming）：Deduplication 其实就是去重，也即上文介绍到的 TopN 中 row\_number = 1 的场景，但是这里有一点不一样在于其排序字段一定是时间属性列，不能是其他非时间属性的普通列。在 row\_number = 1 时，如果排序字段是普通列 planner 会翻译成 TopN 算子，如果是时间属性列 planner 会翻译成 Deduplication，这两者最终的执行算子是不一样的，Deduplication 相比 TopN 算子专门做了对应的优化，性能会有很大提升。
2. ⭐ 应用场景：比如上游数据发重了，或者计算 DAU 明细数据等场景，都可以使用 Deduplication 语法去做去重。
3. ⭐ SQL 语法标准：

```
SELECT [column_list]  
FROM (  
   SELECT [column_list],  
     ROW_NUMBER() OVER ([PARTITION BY col1[, col2...]]  
       ORDER BY time_attr [asc|desc]) AS rownum  
   FROM table_name)  
WHERE rownum = 1
```

其中：

* ⭐ `ROW_NUMBER()`：标识当前数据的排序值
* ⭐ `PARTITION BY col1[, col2...]`：标识分区字段，代表按照这个 col 字段作为分区粒度对数据进行排序
* ⭐ `ORDER BY time_attr [asc|desc]`：标识排序规则，必须为时间戳列，当前 Flink SQL 支持处理时间、事件时间，ASC 代表保留第一行，DESC 代表保留最后一行
* ⭐ `WHERE rownum = 1`：这个子句是一定需要的，而且必须为 rownum = 1

4. ⭐ 实际案例：

博主这里举两个案例：

* ⭐ 案例 1（事件时间）：是腾讯 QQ 用户等级的场景，每一个 QQ 用户都有一个 QQ 用户等级，需要求出当前用户等级在 `星星`，`月亮`，`太阳` 的用户数分别有多少。

```
-- 数据源：当每一个用户的等级初始化及后续变化的时候的数据，即用户等级变化明细数据。  
CREATE TABLE source_table (  
    user_id BIGINT COMMENT '用户 id',  
    level STRING COMMENT '用户等级',  
    row_time AS cast(CURRENT_TIMESTAMP as timestamp(3)) COMMENT '事件时间戳',  
    WATERMARK FOR row_time AS row_time  
) WITH (  
  'connector' = 'datagen',  
  'rows-per-second' = '1',  
  'fields.level.length' = '1',  
  'fields.user_id.min' = '1',  
  'fields.user_id.max' = '1000000'  
);  
  
-- 数据汇：输出即每一个等级的用户数  
CREATE TABLE sink_table (  
    level STRING COMMENT '等级',  
    uv BIGINT COMMENT '当前等级用户数',  
    row_time timestamp(3) COMMENT '时间戳'  
) WITH (  
  'connector' = 'print'  
);  
  
-- 处理逻辑：  
INSERT INTO sink_table  
select   
    level  
    , count(1) as uv  
    , max(row_time) as row_time  
from (  
      SELECT  
          user_id,  
          level,  
          row_time,  
          row_number() over(partition by user_id order by row_time) as rn  
      FROM source_table  
)  
where rn = 1  
group by   
    level
```

输出结果：

```
+I[等级 1, 6928, 2021-1-28T22:34]  
-I[等级 1, 6928, 2021-1-28T22:34]  
+I[等级 1, 8670, 2021-1-28T22:34]  
-I[等级 1, 8670, 2021-1-28T22:34]  
+I[等级 1, 77287, 2021-1-28T22:34]  
...
```

可以看到其有回撤数据。

其对应的 SQL 语义如下：

* ⭐ `数据源`：消费到 Kafka 中数据后，将数据按照 partition by 的 key 通过 hash 分发策略发送到下游去重算子
* ⭐ `Deduplication 去重算子`：接受到上游数据之后，根据 order by 中的条件判断当前的这条数据和之前数据时间戳大小，以上面案例来说，如果当前数据时间戳大于之前数据时间戳，则撤回之前向下游发的中间结果，然后将最新的结果发向下游（发送策略也为 hash，具体的 hash 策略为按照 group by 中 key 进行发送），如果当前数据时间戳小于之前数据时间戳，则不做操作。次算子产出的结果就是每一个用户的对应的最新等级信息。
* ⭐ `Group by 聚合算子`：接受到上游数据之后，根据 Group by 聚合粒度对数据进行聚合计算结果（每一个等级的用户数），发往下游数据汇算子
* ⭐ `数据汇`：接收到上游的数据之后，然后输出到外部存储引擎中
* ⭐ 案例 2（处理时间）：最原始的日志是明细数据，需要我们根据用户 id 筛选出这个用户当天的第一条数据，发往下游，下游可以据此计算分各种维度的 DAU

```
-- 数据源：原始日志明细数据  
CREATE TABLE source_table (  
    user_id BIGINT COMMENT '用户 id',  
    name STRING COMMENT '用户姓名',  
    server_timestamp BIGINT COMMENT '用户访问时间戳',  
    proctime AS PROCTIME()  
) WITH (  
  'connector' = 'datagen',  
  'rows-per-second' = '1',  
  'fields.name.length' = '1',  
  'fields.user_id.min' = '1',  
  'fields.user_id.max' = '10',  
  'fields.server_timestamp.min' = '1',  
  'fields.server_timestamp.max' = '100000'  
);  
  
-- 数据汇：根据 user_id 去重的第一条数据  
CREATE TABLE sink_table (  
    user_id BIGINT,  
    name STRING,  
    server_timestamp BIGINT  
) WITH (  
  'connector' = 'print'  
);  
  
-- 处理逻辑：  
INSERT INTO sink_table  
select user_id,  
       name,  
       server_timestamp  
from (  
      SELECT  
          user_id,  
          name,  
          server_timestamp,  
          row_number() over(partition by user_id order by proctime) as rn  
      FROM source_table  
)  
where rn = 1
```

输出结果：

```
+I[1, 用户 1, 2021-1-28T22:34]  
+I[2, 用户 2, 2021-1-28T22:34]  
+I[3, 用户 3, 2021-1-28T22:34]  
...
```

可以看到这个处理逻辑是没有回撤数据的。其对应的 SQL 语义如下：

* ⭐ `数据源`：消费到 Kafka 中数据后，将数据按照 partition by 的 key 通过 hash 分发策略发送到下游去重算子
* ⭐ `Deduplication 去重算子`：处理时间语义下，如果是当前 key 的第一条数据，则直接发往下游，如果判断（根据 state 中是否存储过改 key）不是第一条，则直接丢弃
* ⭐ `数据汇`：接收到上游的数据之后，然后输出到外部存储引擎中

> 注意：
>
> 在 Deduplication 关于是否会出现回撤流，博主总结如下：
>
> 1. ⭐ Order by 事件时间 DESC：会出现回撤流，因为当前 key 下 `可能会有` 比当前事件时间还大的数据
> 2. ⭐ Order by 事件时间 ASC：会出现回撤流，因为当前 key 下 `可能会有` 比当前事件时间还小的数据
> 3. ⭐ Order by 处理时间 DESC：会出现回撤流，因为当前 key 下 `可能会有` 比当前处理时间还大的数据
> 4. ⭐ Order by 处理时间 ASC：不会出现回撤流，因为当前 key 下 `不可能会有` 比当前处理时间还小的数