---
title: SQL进阶技巧：Hive中Left Join基于or形式匹配连接的一种优雅实现方式？
author: 会飞的一十六
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247485465&idx=1&sn=301daaadc12ee74521c9eac8654f8c26&chksm=e98bcf2fd7f263d20fc27189943b293e40c33c6d68950380b5d927a4dd1c3adf1b7750f7910e&mpshare=1&scene=24&srcid=1105doIibFgzzgP2Fel3JtcN&sharer_shareinfo=279012aed5a5bc1a45c41468b223975f&sharer_shareinfo_first=279012aed5a5bc1a45c41468b223975f#rd
---

**“**  Hive中对于模糊匹配关联是不支持的，如or 连接，基于like的模糊匹配连接，对于此类问题往往需要找新的方案，对于or连接网上给出了解决方案如union的实现形式，本文借助于locate()+concat\_ws()函数进行优雅的实现。**”**

01

—

场景描述

t表

```
id    dt1    2022-06-032    2022-05-043    2022-04-014    2022-05-22
```

t1表：

```
id    dt1                                                                            dt21    2022-06-03                                                               2022-05-032    2022-05-25                                                               2022-05-043    2022-03-01                                                               2022-05-04
```

找出t表中时间字段dt在t1表中dt1,dt2任意出现的id，及时间，保留t表中数据，如果能够匹配到取匹配的时间，未匹配到置为NULL

02

—

数据准备

```
create table t asselect 1 as id,'2022-06-03' as dtunion all select 2 as id,'2022-05-04' as dtunion all select 3 as id,'2022-04-01' as dtunion allselect 4 as id,'2022-05-22' as dt------------------create table t1 asselect 1 as id,'2022-06-03' as dt1,'2022-05-03' as dt2union allselect 2 as id,'2022-05-25' as dt, '2022-05-04' as dt2union allselect 3 as id,'2022-03-01' as dt1, ' 2022-05-04' as dt2
```

03

—

问题分析

针对本问题如果在oracle或mysql中则比较好实现，具体如下：

```
select t.id,case when t1.id is not null then t.dt else null end as dtfrom t join t1on t.id = t1.idand (t.dt=t1.dt1 or t.dt=t1.dt2)
```

但由于**Hive低版本**中不支持不等连接，使本问题增加了难度，传统的解法，采用union+去重的实现方式，具体SQL如下：

```
select id1 as id   ,max(case when id2 is not null then dt else null end) as dtfrom (select cast(t.id as string) as id1,t.dt as dt,cast(t1.id as string) as id2from t left join t1on t.id = t1.idand t.dt=t1.dt1 union select cast(t.id as string) as id1,t.dt as dt,cast(t1.id as string) as id2from tleft join t1on t.id = t1.idand t.dt=t1.dt2 ) tgroup by id1
```

```
结果如下：OKid      dt1       2022-06-032       2022-05-043       NULL4       NULLTime taken: 3.343 seconds, Fetched: 4 row(s)
```

但是上述的方式显示显得很啰嗦，如果后面需要匹配的or比较多，比如有5个的时候，那么同样的逻辑就要union 5次代码看起来相当繁琐，且性能较低。

针对以上问题，本文采用一种优雅的实现方式：我们知道采用or连接的时候，无非就是t表中的字段在t1表中匹配到了就成功，对于这种需要匹配就成功的连接方式，我们自然想到hive中高效的实现方式locate()函数，对于该函数的理解，可以具体参考我如下文章：

[SQL进阶技巧：如何从不固定位置提取字符串元素？【日志清洗】](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484648&idx=1&sn=df75f82071bbfbc435014cb1f37bd01c&chksm=e8e210c8df9599dee323858d58780dc08eb90b33e9c6a7de9d4ab383513ab812147bc47fd51b&scene=21#wechat_redirect)

由于t表中的某个字段需要在t1表中的多个字段中去匹配，为了满足locate()函数使用条件，我们利用concat\_ws()函数进行列转行拼成一个串，然后让该字段在串中去匹配如果能匹配成功则表明该条数据可以匹配到，然后在where语句或case when中进行过滤。注意locate()这种模糊匹配也是不能放在on条件中的，这种本质上也是一种不等连接，只能放在where或case when中过滤，至于为什么要用locate()而不用like函数，原因是locate()的性能要优于like。因此我们具有如下实现方式：

首先用left join做等值连接，然后select语句通过case when去判断匹配，具体SQL如下：

```
select t.id,case when locate(t.dt,concat_ws(',',t1.dt1,t1.dt2))>0 then t.dt else null end as dtfrom tleft join t1on t.id = t1.id
```

具体结果如下：

```
t.id    dt2       2022-05-043       NULL1       2022-06-034       NULLTime taken: 2.275 seconds, Fetched: 4 row(s)
```

从执行性能上来看此种的实现方式也是要优于union的实现方式。

04

—

小结

本文介绍了一种在Hive中处理不支持模糊匹配关联问题的高效解决方案，利用locate()和concat\_ws()函数实现or连接，避免了冗长的union操作，提升了性能。作者通过实际案例展示了如何使用这些函数进行时间字段的精确匹配或NULL填充。

**往期精彩**

[SQL进阶技巧：如何将字符串数组清洗为简单map结构? | translate + regexp\_replace方法](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247485428&idx=1&sn=e149a9c5bdad0201fe3b705d418cda0a&chksm=e8e213d4df959ac24ac3b760f4c57e376dccba675bc4c41187b386725c63f0be0c22e2913535&scene=21#wechat_redirect)

[SQL进阶技巧：如何利用if语句简化where或join中的条件](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247485394&idx=1&sn=cf3dff1478ccb5ca2d8fafb2613816e1&chksm=e8e213f2df959ae4facdc48ad6c88bc598ead7dc8287be3be39d374c5f3d5cd874e8b8823299&scene=21#wechat_redirect)

[SQL进阶技巧：如何查询最近一笔有效订单？ | 近距离有效匹配问题](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484853&idx=1&sn=41bfa4a88d2667550d51fd2a4889dd54&chksm=e8e21195df959883b28a732cca70816361eb80a2bfa10264e91f675eac22b8d65e380c57a52b&scene=21#wechat_redirect)

[SQL进阶技巧：每个HR负责的部门的人数之和 | 层次查询父子关系问题](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247485413&idx=1&sn=2e5d323d1719eb79a012d3e8186ac1b7&chksm=e8e213c5df959ad3a20ede115b589491b34e4161b85c04e5b649a5e3174dc98517f11747ccdf&scene=21#wechat_redirect)

[SQL进阶技巧：如何对数据进行两两组合分析？|  广告策略投放转化问题](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484801&idx=1&sn=1e692f45e061e016a07357084c8c9edc&chksm=e8e211a1df9598b714771aa79d92424b51d524965b3bc7bdae7832070e4da74c2a7a78ef02f0&scene=21#wechat_redirect)

[SQL进阶技巧：如何优雅实现根据组内排序规则生成新列问题？ | 分组TON问题](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484739&idx=1&sn=c8367893ad7a66593c3046c90d440a08&chksm=e8e21163df959875ee357534899daf2591d814fc4d6dcae2a362fbbd3a788d182df6bd26eaa0&scene=21#wechat_redirect)

[SQL进阶技巧：如何从不固定位置提取字符串元素？【日志清洗】](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484648&idx=1&sn=df75f82071bbfbc435014cb1f37bd01c&chksm=e8e210c8df9599dee323858d58780dc08eb90b33e9c6a7de9d4ab383513ab812147bc47fd51b&scene=21#wechat_redirect)

[SQL进阶技巧：时点值状态统计如何分析？【某时点旅店客人在住房间数量统计】](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484605&idx=1&sn=d4cadba7b4599fb546e896956fd66393&chksm=e8e2109ddf95998b76c500fd2fb60543e4184ed4bb13671a878911910bf34bb489831f63cc40&scene=21#wechat_redirect)

[SQL进阶技巧：如何利用Bitmap思想优化Array\_contains()函数？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484567&idx=1&sn=4d10c88acc86abd5a13d2a9b42284bb4&chksm=e8e210b7df9599a15423d1373ba3e8d2a9b66e4ed7e39b6e6269ab76810b9a0bea6d61863733&scene=21#wechat_redirect)

[SQL进阶技巧：数据清洗如何分析商品入库采购成本数据缺失问题？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484495&idx=1&sn=af4f5ce3f3f2ad580d4cc85b3777c0a7&chksm=e8e2106fdf959979d7c49120f757e0729af7e54b90c71980536087c6860d8a5dd250df3aa994&scene=21#wechat_redirect)

[SQL进阶技巧：如何计算先进先出库龄问题？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484330&idx=1&sn=ee9b76bde4035d1a8cca8d9a803112a0&chksm=e8e2178adf959e9cd0803de7d2c6a152d37daeebd8be3d27b1447828c3cba3a5060e02c4fb3d&scene=21#wechat_redirect)

[SQL进阶技巧：如何计算重叠区间合并问题？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484320&idx=1&sn=9141f312c268f73a16315347707d1a97&chksm=e8e21780df959e96e06e9a49b721e8fab8bbd086da6f87ea8815302782ebe096d8f45b533fe3&scene=21#wechat_redirect)

[SQL进阶技巧：如何进行用户行为路径分析、构建桑基图？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484200&idx=1&sn=4caf63d8307cfa0ae69096625b8ce5f2&chksm=e8e21708df959e1e25b72f6125f330d06fa778c8f2b8dd1a7b2a2067de02889404d59673d782&scene=21#wechat_redirect)