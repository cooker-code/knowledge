---
title: Hive 优化：如何用倍数小表优化数据倾斜？
author: 会飞的一十六
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484834&idx=1&sn=48bea34c29e926e00ed440239c4fd9d2&chksm=e906a1e8735414a0ec4f37bcb3b3bd1eb44b5fadad96b26fb1358d47f06f2b4d0556d1b7f3a5&mpshare=1&scene=24&srcid=0831yDANXGjvSN9RsFs02dwY&sharer_shareinfo=29470e89c9037119b4cadc6ff7ea3237&sharer_shareinfo_first=29470e89c9037119b4cadc6ff7ea3237#rd
---

01

—

场景描述

[有一张日志表](http://mp.weixin.qq.com/s?__biz=MjM5NDAwMTA2MA==&mid=2695729815&idx=1&sn=7c7feb801bfb8fcf5dfe4e782a4ba3c1&chksm=83d74b5cb4a0c24a21e621ed3428fd808e80c2c5e806a8162cceee62a5930b0430d5506e0f8c&scene=21#wechat_redirect)log表（memberid,pvtime），和会员表memberid(memberid),统计每一个会员总浏览时长。

计算该问题时一般先用日志表与会员表进行join过滤出会员的日志信息，但是在过滤日志时进行join时候，由于每个会员活跃程度不一样，出现部分会员非常活跃，导致关联时key分布不均出现数据倾斜会员表相对于日志表来说比较小，不是很大，但是走mapjoin的话资源又不足，应该做出怎样的SQL优化？

02

—

问题分析

针对上述问题，此时我们给出一种比较通用的解法倍数小表法（将每个memberid数据扩容，相当于表扩容）。表扩容（key扩容）我们采用lateral view explode() 的方法，如下将小表扩容10倍

```
lateral view explode(array(1,2,3,4,5,6,7,8,9,10)) mytable as rd --生成随机数用于均匀发往reduce中
```

第一步生成倍数表

```
select  memberid, rdfrom members dlateral view explode(array(1,2,3,4,5,6,7,8,9,10)) mytable as rd
```

第二步：日志表与会员表关联。

**日志表与会员表关联时候需要按照随机数均匀进行分散，发往各个redcue中，所以关联条件中需要加入随机数条件。**

```
select t1.memberid memberid      ,t2.pvtime pvtime      ,t1.rd rdfrom (select memberid,pvtime,cast(rand()*10 as int) + 1 as rdfrom log) t2join(select  memberid, rdfrom members dlateral view explode(array(1,2,3,4,5,6,7,8,9,10)) mytable as rd ) t1on t1.memberid = t2.memberid and t1.rd = t2.rd
```

或者

```
select t1.memberid memberid      ,t2.pvtime pvtime      ,t1.rd rdfrom   log t2join(select  memberid, rdfrom members dlateral view explode(array(1,2,3,4,5,6,7,8,9,10)) mytable as rd ) t1on t1.memberid = t2.memberid and t1.rd = pmod(t2.pvtime,10)+1 --思考pmod()函数用法
```

第三步：按照随机数部分分组汇总（优化group by）

```
select rd,memberid,sum(pvtime) pvtimefrom(    select t1.memberid memberid          ,t2.pvtime pvtime          ,t1.rd rd    from     (select memberid,pvtime,cast(rand()*10 as int) + 1 as rd    from log    ) t2    join    (select  memberid, rd    from members d    lateral view explode(array(1,2,3,4,5,6,7,8,9,10)) mytable as rd     ) t1    on t1.memberid = t2.memberid and t1.rd = t2.rd) tgroup by rd,memberid --一定注意rd随机数在前,memberid在后，思考为什么？
```

第四步：按会员id整体汇总

```
select memberid,sum(pvtime) pvtimefrom(    select rd,memberid,sum(pvtime) pvtime        from(            select t1.memberid memberid                  ,t2.pvtime pvtime                  ,t1.rd rd            from             (select memberid,pvtime,cast(rand()*10 as int) + 1 as rd            from log            ) t2            join            (select  memberid, rd            from members d            lateral view explode(array(1,2,3,4,5,6,7,8,9,10)) mytable as rd             ) t1            on t1.memberid = t2.memberid and t1.rd = t2.rd        ) t        group by rd,memberid --一定注意rd随机数在前,memberid在后，思考为什么？) tgroup by memberid
```

03

—

小结

本文给出了一种利用倍数小表优化数据倾斜的一种通用方法，该方法适用场景为数据倾斜时不能用mapjoin的时候，也就是集群资源不足时候，通过该方法能够有效缓解数据倾斜，但不能根除数据倾斜。处理数据倾斜最好的方法还是采用**分治思想**，利用**mapjoin一分为二的处理**，倾斜key单独走mapjoin，非倾斜key走reduce join，最终将数据union all起来，采用mapjoin的方法切断了shuffle过程，也就没有数据倾斜这一说，缺点耗资源，前提是集群资源足够条件下，且满足mapjoin的条件。

**往期精彩**

**[Hive 利用Distribute by 解决动态分区小文件过多问题 | 小文件优化](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484802&idx=1&sn=fece76652d41448b4fbc9d9f0ce41e17&chksm=e8e211a2df9598b47da590a1fbad98c7c00dd0963958bc1a66156f2fa6de514cd6ca7568efa3&scene=21#wechat_redirect)**

**[万字详解Spark并行度  |  从spark.default.parallelism参数来看Spark并行度、并行计算任务概念](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484770&idx=1&sn=b22c7a46e8c60e98f48fc0c4e9ff484e&chksm=e8e21142df959854bb9294fe8965951be21d93c0e20b24a69a291aaad0fca6e0173eb44a2c40&scene=21#wechat_redirect)**

**[数仓建模：Hive加载数据的几种方式分析。](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484745&idx=1&sn=faf0db3ca4f4829028bfc9b8955ea960&chksm=e8e21169df95987fa92323b0a024cdf7f4c0d2a9d457a084ab4f1b39529c8fbbb14ad0a4f4bc&scene=21#wechat_redirect)**

**[SQL进阶技巧：如何计算先进先出库龄问题？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484330&idx=1&sn=ee9b76bde4035d1a8cca8d9a803112a0&chksm=e8e2178adf959e9cd0803de7d2c6a152d37daeebd8be3d27b1447828c3cba3a5060e02c4fb3d&scene=21#wechat_redirect)**