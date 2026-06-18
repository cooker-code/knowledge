> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040101_ClickHouse/040101_核心知识点/ClickHouseAggregateFunction与物化视图预聚合边界|ClickHouseAggregateFunction与物化视图预聚合边界]]
---
title: 「ClickHouse系列」实时分析优化AggregateFunction及物化视图
author: 大数据技术与架构
date:
url: http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247512491&idx=1&sn=84bd50bb724d2217a78bce1236ddf4d5&chksm=fd3ef73eca497e288922d5bf09b821b68e7ac8053b21de6832d7f73c80a85d62ee6bc6e26ce6&scene=126&&sessionid=1648625983#rd
---

点击上方**蓝色字体**，选择“设为星标”

回复"**面试"**获取更多惊喜

[**八股文教给我，你们专心刷题和面试**](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247512011&idx=1&sn=68aaa9c5c42e2087d56d170c442b30ba&chksm=fd3ee95eca496048b24a717a30fe820b9188d02e014ba7b554afe86f8da1efab189799fe7087&scene=21#wechat_redirect)

> Hi，我是王知无，一个大数据领域的原创作者。
> 放心关注我，获取更多行业的一手消息。

## AggregateFunction

AggregatingMergeTree有些许数据立方体的意思，它能够在合并分区的时候，按照预先定义的条件，聚合数据。

同时，根据预先定义的聚合函数，计算数据并通过二进制的格式存入表内。

将同一分组下的多行数据，聚合成一行，既减少了数据行，又降低了后续聚合查询的开销。

```
-- 建表语句
CREATE TABLE agg_table( 
 id String,
 city String,
 code AggregateFunction(uniq,String), 
 value AggregateFunction(sum,UInt32), 
 create_time DateTime
)
ENGINE = AggregatingMergeTree() 
PARTITION BY toYYYYMM(create_time) 
ORDER BY (id,city)
PRIMARY KEY id
```

其中的uniq、sum是指定的聚合函数。大家可以在官网aggregate-functions 下查看更多的相关函数。

AggregateFunction是ClickHouse提供的一种特殊的数据类型，它能够以二进制的形式存储中间状态结果。

其使用方法也十分特殊，对于AggregateFunction类型的列字段，数据的写入和查询都与寻常不同。

在写入数据时，需要调用State函数。而在查询数据时，则需要调用相应的Merge函数。

例如：在写入数据时需要调用与uniq、sum对应的uniqState和sumState函数，并使用INSERT SELECT 语法:

```
-- 写入测试数据id = A000, code相同;
INSERT INTO TABLE agg_table SELECT 'A000','test', uniqState('code1'), sumState(toUInt32(100)), '2019-08-10 17:00:00';
 INSERT INTO TABLE agg_table SELECT 'A000','test', uniqState('code1'), sumState(toUInt32(100)), '2019-08-10 17:00:00';
-- 写入测试数据id = A001, code不同;
 
 INSERT INTO TABLE agg_table SELECT 'A001','test', uniqState('code1'), sumState(toUInt32(100)), '2019-08-10 17:00:00';
  INSERT INTO TABLE agg_table SELECT 'A001','test', uniqState('code2'), sumState(toUInt32(50)), '2019-08-10 17:00:00';
```

而在查询数据时，如果直接使用列名访问code和value，将会是无法显示的二进制。

此时，则需要调用与uniq、sum对应的uniqMerge、sumMerge函数:

```
 SELECT id,city,uniqMerge(code),sumMerge(value) FROM agg_table GROUP BY id,city;
 
┌─id───┬─city─┬─uniqMerge(code)─┬─sumMerge(value)─┐
│ A001 │ test │               2 │             150 │
│ A000 │ test │               1 │             200 │
└──────┴──────┴─────────────────┴─────────────────┘

2 rows in set. Elapsed: 0.002 sec. 
```

可以看到id = A000 的uniqMerge(code) 为1、可见uniq函数是生效的。

但是你是否会认为AggregatingMergeTree使用起来过于繁琐呢?

连正常进行数据写入都行不通，还需要借助INSERT…SELECT的句式并调用特殊函数。如果直接像刚才示例中那样使用AggregatingMergeTree，确实会非常的麻烦。

不过各位读者并不需要忧虑，因为目前介绍的这种使用方式，并不是它的主流用法。

## 物化视图

AggregatingMergeTree更为常⻅的应用方式，是结合物化视图使用，将它作为物化视图的表引擎。

而这里的物化视图，则是作为其他数据表上层的一种查询视图。

现在用一组示例说明它的用法，首先，建立明细数据表，也就是俗称的底表。

```
CREATE TABLE agg_table_basic( id String,
    city String,
    code String,
    value UInt32
)ENGINE = MergeTree() PARTITION BY city ORDER BY (id,city)
```

通常会使用MergeTree作为底表，用于存储全部的明细数据，并以此对外提供实时查询。

接着，新建一张物化视图:

```
CREATE MATERIALIZED VIEW agg_view 
ENGINE = AggregatingMergeTree() 
PARTITION BY city
ORDER BY (id,city)
AS SELECT id,
    city,
    uniqState(code) AS code,
    sumState(value) AS value
FROM agg_table_basic GROUP BY id, city;
```

物化视图使用AggregatingMergeTree表引擎，用于特定场景的数据查询，相比MergeTree它拥有更高的性能。

在新增数据时，面向的对象是底表MergeTree:

```
 INSERT INTO TABLE agg_table_basic VALUES
  ('A000','wuhan','code1',100),
  ('A000','wuhan','code2',200),
  ('A000','zhuhai','code1',200);
```

数据会自动同步到物化视图，并按照AggregatingMergeTree引擎的规则处理。

在查询数据时，面向的对象则是物化视图AggregatingMergeTree:

```
SELECT id, sumMerge(value), uniqMerge(code) FROM agg_view GROUP BY id, city;

┌─id───┬─sumMerge(value)─┬─uniqMerge(code)─┐
│A000│ 200 │ 1 │
│A000│ 300 │ 2 │ 
└──────┴───────────────┴───────────────┘
```

接下来，简单梳理一下AggregatingMergeTree的处理逻辑:

1. 使用ORBER BY排序键，作为聚合数据的条件Key
2. 使用AggregateFunction字段类型，定义聚合函数的类型以及聚合的字段
3. 只有在合并分区的时候，才会触发聚合计算的逻辑
4. 以数据分区为单位，聚合数据。当分区合并时，同一数据分区内，聚合Key相同的数据，会合并计算；而不同分区之间，那些跨越分区的数据，则不会被计算
5. 在进行数据计算时，因为分区内的数据已经基于ORBER BY排序，所以能够找到那些相邻的，拥有 相同聚合Key的数据
6. 在聚合数据时，同一分区内，相同聚合Key的多行数据，会合并成一行。对于那些非主键、非AggregateFunction类型字段，则会使用第一行数据的取值
7. AggregateFunction类型的字段使用二进制存储，在写入数据时，需要调用State函数；而在查询数据时，则需要调用相应的Merge函数。其中，\*表示定义时使用的聚合函数
8. AggregatingMergeTree通常作为物化视图的表引擎，与普通MergeTree搭配使用

物化视图完整语法：

```
CREATE [MATERIALIZED] VIEW [IF NOT EXISTS] [db.]table_name [TO[db.]name] 
 [ENGINE = engine] [POPULATE] AS SELECT ...
```

当物化视图创建之后，如果源表被写入了新数据，那么物化视图也会同步更新。

POPULATE修饰符决定了物化视图的初始化策略:

* 如果使用了POPULATE修饰符，那么在创建视图的过程中，会连带将源表中 已存在的数据一并导入，如同执行了SELECT INTO一般;
* 反之，如果不使用POPULATE修饰符，那么物化视图在创建之后是没有数据的，它只会同步在此之后被写入源表的数据。

物化视图目前并不支持同步删除，如果在源表中删除了数据，物化视图的数据仍会保留。

物化视图本质上是一张特殊的数据表，例如使用SHOW TABLE查看数据表的列表:

```
SHOW TABLES; 

┌─name────────┐
│ .inner.view_test2 │ │ .inner.view_test3 │ 
└───────────┘
```

## 原理

上述的AggregateFunction底层原理是使用的预计算，也就是每写入一批数据，trigger就会触发一次计算结果更新视图。所以在海量数据的场景下这种查询效率也是非常高的。

如果这个文章对你有帮助，不要忘记 **「在看」** **「点赞」** **「收藏」** 三连啊喂！

[2022年全网首发|大数据专家级技能模型与学习指南(胜天半子篇)](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247510040&idx=1&sn=aa335f25965975731173916f012d56f4&chksm=fd3eee8dca49679b82f632048fb64d21fac01497d1a0fe33917edb01e194caf0f9a1930a55ce&scene=21#wechat_redirect)

[互联网最坏的时代可能真的来了](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247508317&idx=1&sn=0bcb7fb6997b42306994b890eaa0d47f&chksm=fd3ee7c8ca496ede347a2971002c6ea68dcfd70abeeb90b43c8b02a2a60ebc7cec3a87ef7d52&scene=21#wechat_redirect)

[我在B站读大学，大数据专业](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247507860&idx=1&sn=807ac5003762f29c127bc4071dcebe33&chksm=fd3e9901ca4910178cfc816043ea86c29f881f70cd943d252de9ae2e21cb7e8ba80bff162e1b&scene=21#wechat_redirect)

[我们在学习Flink的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247499604&idx=1&sn=d938dfb30d221774704982d2938b30c1&chksm=fd3eb9c1ca4930d76a391241333de461ca22d2aa27472eab3cffab1564872ae37f1b48fe2d3c&scene=21#wechat_redirect)

[193篇文章暴揍Flink，这个合集你需要关注一下](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504856&idx=2&sn=6f62e2a0c756ce56773eed253d2f41ee&chksm=fd3e954dca491c5b732b7e2aa46db32efbc3f690e528522098659bb3c6e78e1582ab4529b5dc&scene=21#wechat_redirect)

[Flink生产环境TOP难题与优化，阿里巴巴藏经阁YYDS](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504742&idx=1&sn=8765c198d8ad66219a7bcabb221e4a23&chksm=fd3e95f3ca491ce52d0724b9e4154a47af0f5ea349e1e184c8fbc65aa17c0000242aa9b58429&scene=21#wechat_redirect)

[Flink CDC我吃定了耶稣也留不住他！| Flink CDC线上问题小盘点](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504813&idx=1&sn=f5cd6ae2aa2b1e30f87a5ae55971c514&chksm=fd3e9538ca491c2eb191677d070f2c7e4f1098eece00e256d6205b8a5d21b21c1e74f875f16f&scene=21#wechat_redirect)

[我们在学习Spark的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=1&sn=1f693e549f76622725b936041ff8896e&chksm=e9bff2fbdec87bedecbdeed4547d7d612c72fc8d4918614a26a4e31666cf0128fd57b537f347&scene=21#wechat_redirect)

[在所有Spark模块中，我愿称SparkSQL为最强！](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504834&idx=1&sn=46ca1a3924b8fdb89ac1c2acb0319d6c&chksm=fd3e9557ca491c41d837b917639ea62007ea16d3c2cc6c0c02d1f8a8c6e44318f5183b787c46&scene=21#wechat_redirect)

[硬刚Hive | 4万字基础调优面试小总结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247502750&idx=1&sn=bd9a9173d060dc4e4ebd49c8efc6acfe&chksm=fd3e8d0bca49041dea84da93910e5efdc4935e520525c09887c986691377aeb48e5cf7fb5667&scene=21#wechat_redirect)

[数据治理方法论和实践小百科全书](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504382&idx=2&sn=550b78802acfe727e0e77cd9195f8784&chksm=fd3e976bca491e7db2b8b2446d231736df01bbf13653804d680ac4390597ec17fa466ad4ae83&scene=21#wechat_redirect)

[标签体系下的用户画像建设小指南](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503741&idx=1&sn=e5039be93123f2e337013756a818bfc3&chksm=fd3e89e8ca4900fe603b63c5722a6fb8a32bd63d6ba23e0028851948a71b877eb1f742d95087&scene=21#wechat_redirect)

[4万字长文 | ClickHouse基础&实践&调优全视角解析](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503675&idx=1&sn=3ee6af64d0126c78b48cad219308f81e&chksm=fd3e89aeca4900b8b8954e9569ee3c0877881fac8c792bfafc22e7e9d3e8524da8eb860d33d8&scene=21#wechat_redirect)

[【面试&个人成长】2021年过半，社招和校招的经验之谈](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=2&sn=57ecc77718f1b6f2f262d62e8318dcc9&chksm=e9bff2fbdec87bedb1986765bfae7dbabb9aece45b0b2af147bbad1ac5d79ebc934c64df40ea&scene=21#wechat_redirect)

[大数据方向另一个十年开启 |《硬刚系列》第一版完结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504478&idx=1&sn=14efef3868ba42bd044f9618745a7fdc&chksm=fd3e94cbca491dddb961b7b8b93105b2869c5bcf4c03f9f0dc83ad62be0dce4c124b52ed0ba3&scene=21#wechat_redirect)

[我写过的关于成长/面试/职场进阶的文章](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504410&idx=1&sn=7e81ab5a324395eb0f12397c40247ca1&chksm=fd3e948fca491d9946456acbd93b2d651ae4d7a7d0127a211981e08f50f377e60d1b7c0d7b3a&scene=21#wechat_redirect)

[当我们在学习Hive的时候在学习什么？「硬刚Hive续集」](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504783&idx=1&sn=72aed147a459368ed934a007a2df3bc9&chksm=fd3e951aca491c0cade212390011eca7d8a68951fee6aa2844b8f4d14bcbd5b3ed26305d69dc&scene=21#wechat_redirect)