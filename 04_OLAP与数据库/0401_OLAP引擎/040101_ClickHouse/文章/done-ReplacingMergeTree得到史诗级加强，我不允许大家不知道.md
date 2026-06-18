> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040101_ClickHouse/040101_核心知识点/ClickHouseReplacingMergeTree与Upsert边界|ClickHouseReplacingMergeTree与Upsert边界]]
---
title: ReplacingMergeTree得到史诗级加强，我不允许大家不知道
author: ClickHouse的秘密基地
date:
url: http://mp.weixin.qq.com/s?__biz=MzA4MDIwNTY4MQ==&mid=2247485444&idx=1&sn=cbf8089986420d5b68683cd3080036f2&chksm=9fa68a1aa8d1030cd145277b1cc1c07b6fef893e42ff6967eea7f6ab24b8c974ea8027c3a357&mpshare=1&scene=24&srcid=0703YPkGWoziAzvxLLpj3wfv&sharer_sharetime=1688343348005&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

MergeTree家族枝繁叶茂，人数众多。

但要选个最能打的，ReplacingMergeTree肯定算一个。

Long Long Ago，天地浑浊，ReplacingMergeTree出现了。

起初，它的出现是为了解决重复数据的问题。再往后，利用新版本覆盖旧版本的特性，人们就变相实现了 UPDATE 的功能。

大家一看，INSERT、UPDATE 都有了，那索性再来个 DELETE 吧。于是利用 ReplacingMergeTree 的删除方案就有了，在表上加一个\_delete字段，0表示有效，1表示无效。查询的时候把\_delete=1的过滤掉。

皆大欢喜，增删改查都齐活了。

But，这种删除方案还是有缺点：

* 查询的时候过滤要自己实现，有点麻烦
* \_delete=1的数据并没有被物理删除

时光荏苒，斗转星移，一晃就到2023年了。在 ClickHouse 的新版本中，ReplacingMergeTree 又得到了史诗级加强，居然内置了删除能力。这么一来，你让 CollapsingMergeTree 怎么活呀。

新版本中，ReplacingMergeTree(ver, is\_deleted) 多了一个选填参数，

```
`is_deleted`：Column data type — `Int8`.1：删除0：正常必须和ver一起使用
```

接下来举例说明：

创建一张表：

```
CREATE TABLE replace_test_a(  user_id UInt64,  score String,  is_deleted UInt8 DEFAULT 0,  create_time DateTime DEFAULT toDateTime(0))ENGINE= ReplacingMergeTree(create_time,is_deleted)ORDER BY user_id
```

写入1000w数据：

```
INSERT INTO TABLE replace_test_a(user_id,score)WITH(  SELECT ['A','B','C','D','E','F','G'])AS dictSELECT number AS user_id, dict[number%7+1] FROM numbers(10000000)
SELECT *FROM replace_test_aLIMIT 5
Query id: b023c6e8-6a3f-4419-887a-d024b808c20f
┌─user_id─┬─score─┬─is_deleted─┬─────────create_time─┐│       0 │ A     │          0 │ 1970-01-01 08:00:00 ││       1 │ B     │          0 │ 1970-01-01 08:00:00 ││       2 │ C     │          0 │ 1970-01-01 08:00:00 ││       3 │ D     │          0 │ 1970-01-01 08:00:00 ││       4 │ E     │          0 │ 1970-01-01 08:00:00 │└─────────┴───────┴────────────┴─────────────────────┘
```

修改1行数据, 将 user\_id=0的 改成 AAAA：

```
INSERT INTO TABLE replace_test_a(user_id,score,create_time) VALUES(0,'AAAA',now())
SELECT *FROM replace_test_aFINALLIMIT 5
Query id: e01df531-6c81-4ef0-a079-05f78293e4de
┌─user_id─┬─score─┬─is_deleted─┬─────────create_time─┐│       0 │ AAAA  │          0 │ 2023-06-29 19:39:27 ││       1 │ B     │          0 │ 1970-01-01 08:00:00 ││       2 │ C     │          0 │ 1970-01-01 08:00:00 ││       3 │ D     │          0 │ 1970-01-01 08:00:00 ││       4 │ E     │          0 │ 1970-01-01 08:00:00 │└─────────┴───────┴────────────┴─────────────────────┘
```

可以看到，修改成功。

接下来是重头戏，我们删除user\_id=0的数据：

```
INSERT INTO TABLE replace_test_a(user_id,score,is_deleted,create_time) VALUES(0,'AAAA',1,now())
SELECT *FROM replace_test_aFINALLIMIT 5
Query id: 2b1b10f7-884d-4dfd-a9a2-8982cb899292
┌─user_id─┬─score─┬─is_deleted─┬─────────create_time─┐│       1 │ B     │          0 │ 1970-01-01 08:00:00 ││       2 │ C     │          0 │ 1970-01-01 08:00:00 ││       3 │ D     │          0 │ 1970-01-01 08:00:00 ││       4 │ E     │          0 │ 1970-01-01 08:00:00 ││       5 │ F     │          0 │ 1970-01-01 08:00:00 │└─────────┴───────┴────────────┴─────────────────────┘
```

删除成功，接下来删除前50w行试试:

```
INSERT INTO TABLE replace_test_a(user_id,score,is_deleted,create_time)WITH(  SELECT ['AA','BB','CC','DD','EE','FF','GG'])AS dictSELECT number AS user_id, dict[number%7+1],1, now() AS create_time FROM numbers(500000)

SELECT COUNT()FROM replace_test_aFINAL
Query id: 3e738515-b23c-48de-80ca-c90eb35ed6df
┌─count()─┐│ 9500000 │
```

数据少了50w行，删除成功。

ReplacingMergeTree的实现原理是：

* 查询时过滤 is\_deleted = 1 的数据
* Merge时物理删除is\_deleted = 1 的数据

赶快去试试这项新功能吧。

btw，我在例子里的查询都带了FINAL。FINAL的性能在新版本中也得到了加强，以后我会专门写一篇解析的文章。

欢迎大家扫码关注我的公众号:

**ClickHouse的秘密基地**

**往期精彩推荐:**

[【专辑】ClickHouse实战系列课程](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA4MDIwNTY4MQ==&action=getalbum&album_id=1931467025848074243#wechat_redirect)

[【专辑】ClickHouse的资讯手札](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1336889409916043265&__biz=MzA4MDIwNTY4MQ==#wechat_redirect)

[【专辑】ClickHouse的原理巩固](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1339543307420106752&__biz=MzA4MDIwNTY4MQ==#wechat_redirect)

[【专辑】ClickHouse的经验分享](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1342290453848326144&__biz=MzA4MDIwNTY4MQ==#wechat_redirect)