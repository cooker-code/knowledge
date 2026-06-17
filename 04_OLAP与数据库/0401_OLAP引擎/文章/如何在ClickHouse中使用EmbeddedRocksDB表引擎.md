---
title: 如何在ClickHouse中使用EmbeddedRocksDB表引擎
author: ClickHouse的秘密基地
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA4MDIwNTY4MQ==&mid=2247485323&idx=1&sn=7297d00807dd7a023f381aa1d369e4c6&chksm=9fa68595a8d10c83a2523e545b75ec894d3c8b275284a6ea0459c798ea6baad37bb1eb1257dd&mpshare=1&scene=24&srcid=030750rtsQLmgVTBsk66xEAD&sharer_sharetime=1646608007572&sharer_shareid=4145ee29355a80b1b2b01921ded70e3a#rd
---

RocksDB 想必大家并不陌生，它是一款高性能的嵌入式KV数据库，是众多自研数据库背后的男人。

现在我们可以利用 EmbeddedRocksDB 表引擎，直接在 ClickHouse 中使用 RocksDB，非常适用 KV 查询的场景。

使用的方法非常简单，接下来就用一个简短示例说明，创建一张测试表，使用 EmbeddedRocksDB 引擎:

```
CREATE TABLE test_rocksDB(    `A` UInt64,    `B` String)ENGINE = EmbeddedRocksDBPRIMARY KEY A
```

必须指定 PRIMARY KEY ， 它表示 RocksDB Key

接着写入 1亿 测试数据：

```
INSERT INTO test_rocksDB SELECT    number,    toString(cityHash64(number))FROM numbers(100000000)
```

查看表文件，会发现直接是 RocksDB 的存储文件：

接着就可以查询了：

```
SELECT count()FROM(    SELECT *    FROM test_rocksDB    WHERE A IN (        SELECT toUInt64(rand64() % 100000000)        FROM numbers(10000)    ))  
Query id: 96f56029-9201-4985-bf9f-8d714a3bc1bd  
┌─count()─┐│   10000 │└─────────┘  
1 rows in set. Elapsed: 0.221 sec. Processed 10.00 thousand rows, 364.02 KB (45.23 thousand rows/s., 1.65 MB/s.)
```

‍

我用 MergeTree 做同样的查询，查询的实效差距不大（RocksDB适用默认参数），但是MergeTree扫描的数据是 RocksDB 表引擎的5000倍

```
# RocksDB1 rows in set. Elapsed: 0.221 sec. Processed 10.00 thousand rows, 364.02 KB (45.23 thousand rows/s., 1.65 MB/s.)#MergeTree1 rows in set. Elapsed: 0.213 sec. Processed 56.58 million rows, 452.67 MB (265.18 million rows/s., 2.12 GB/s.)
```

原创不易，如果这篇文章对你有帮助，欢迎 点赞、转发、在看 三连击

欢迎大家扫码关注我的公众号:

**ClickHouse的秘密基地**

**往期精彩推荐:**

[【专辑】ClickHouse实战系列课程](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA4MDIwNTY4MQ==&action=getalbum&album_id=1931467025848074243#wechat_redirect)

[【专辑】ClickHouse的资讯手札](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1336889409916043265&__biz=MzA4MDIwNTY4MQ==#wechat_redirect)

[【专辑】ClickHouse的原理巩固](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1339543307420106752&__biz=MzA4MDIwNTY4MQ==#wechat_redirect)

[【专辑】ClickHouse的经验分享](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1342290453848326144&__biz=MzA4MDIwNTY4MQ==#wechat_redirect)