---
title: SparkSql不同写法的一些坑(性能优化)
author: 数据仓库践行者
date:
url: http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485561&idx=1&sn=f4937d7f1fb7bcafab80eac6a868dd0f&chksm=fe6c5966c91bd070487cd549f2a69cf49e107b8327dc58eee65f49c41a34d0e239c407edc546&mpshare=1&scene=24&srcid=0820V0eDNnj6EQPwtQl4xxmn&sharer_sharetime=1660986844790&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_核心知识点/SparkSQL业务逻辑优化方法|SparkSQL业务逻辑优化方法]]


说三种情况，看大家有没有遇到类似的场景。

**第一种情况：**

这种情况也是我经常会遇到的一个场景，之前也有同学拿着sql来问，说这样写会不会影响运行效率：

```
select    tmp.A from (select A,B from testdata2) tmp
```

**结论是**

不用担心，这样写完全可以被优化    

```
== Analyzed Logical Plan ==Project [A#3]+- SubqueryAlias tmp   +- Project [A#3, B#4]      +- SubqueryAlias testdata2         +- View (`testData2`, [a#3,b#4])            +- SerializeFromObject [knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).a AS a#3, knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).b AS b#4]               +- ExternalRDD [obj#2]                           == Optimized Logical Plan ==Project [A#3]+- SerializeFromObject [knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).a AS a#3]   +- ExternalRDD [obj#2]
```

从执行计划上清晰的看到，最终被优化成

```
select A from testdata2
```

这样的效果，主要是 ColumnPruning（列裁剪） 和 CollapseProject（合并Project）这两种优化器起到作用。

**第二种情况：**

这种情况之前一直没在意，发现我写过的一些代码里默默都这么用了

```
 -- 其中myudf是一个自定义UDF函数，返回一个数组 select    myudf(A,B)[0] as a1,   myudf(A,B)[1] as a2,   myudf(A,B)[2] as a3  from testdata2
```

这里myudf(A,B)执行几遍？

**结论是**

会执行三遍。

如果myudf是一个很复杂的函数，要合并两个非常复杂的字符串A和B，这个也是我工作中的一个场景。这样的话，执行三遍，非常不合理。

怎么办？

改写：

```
 -- 其中myudf是一个自定义UDF函数，返回一个数组 select    atmp[0] as a1,   atmp[1] as a2,   atmp[2] as a3  from  (select    myudf(A,B) as atmp from testdata2 ) tmp
```

这样改写就万事大吉了嘛？

在sparksql branch3.3 这样改写完全没问题，但毕竟3.3是新版本，大部分人都还没用上，换到3.3之前的版本，分分钟再给变（优化）成第一种写法（执行三遍的）。

**branch3.3（是ok的，内层先计算出myudf的值，外层用计算过的值取数）：**

```
== Analyzed Logical Plan ==Project [atmp#10[0] AS a1#11, atmp#10[1] AS a2#12, atmp#10[2] AS a3#13]+- SubqueryAlias tmp   +- Project [myudf(A#3,B#4) AS atmp#10]      +- SubqueryAlias testdata2         +- View (`testData2`, [a#3,b#4])            +- SerializeFromObject [knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).a AS a#3, knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).b AS b#4]               +- ExternalRDD [obj#2]


== Optimized Logical Plan ==Project [atmp#10[0] AS a1#11, atmp#10[1] AS a2#12, atmp#10[2] AS a3#13]+- Project [myudf(A#3,B#4) AS atmp#10]   +- SerializeFromObject [knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).a AS a#3, knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).b AS b#4]      +- ExternalRDD [obj#2]
```

**branch3.3之前的版本（不管我们愿不愿意，都给合并喽）：**

```
== Analyzed Logical Plan ==Project [atmp#10[0] AS a1#11, atmp#10[1] AS a2#12, atmp#10[2] AS a3#13]+- SubqueryAlias tmp   +- Project [concat(array(A#3), array(B#4)) AS atmp#10]      +- SubqueryAlias testdata2         +- SerializeFromObject [knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).a AS a#3, knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).b AS b#4]            +- ExternalRDD [obj#2]            == Optimized Logical Plan ==            Project [myudf(A#3,B#4)[0] AS a1#11, myudf(A#3,B#4)[1] AS a2#12, myudf(A#3,B#4)[2] AS a3#13]+- SerializeFromObject [knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).a AS a#3, knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).b AS b#4]   +- ExternalRDD [obj#2]
```

一直不信，怎么会不这么不智能，具体原因是啥？该怎么避免？这个在上周六的直播分享里讲过了。

**第三种情况：**

这种也会经常遇到，并且也会经常被其他朋友问到能不能被优化

```
 // 其中用collect_set来代表聚合函数select   collect_set(a)[0] as c1,    collect_set(a)[1] as c2,   collect_set(a)[3] as c3from testdata2 group by b
```

这里的collect\_set(a)会执行几遍？

**结论是**

执行一遍。

这样类似的还有：count(xxx),count(distinct xxx) 等等，聚合函数在重复用时，不用担心，sparksql会给优化。所以，我们在写代码时就不用考虑再在外面写一层，从而避免多写一层，造成数据多流转一次的浪费。

看看吧，不同的情况，会有不同的优化结果，如果知道原理，就能避开一些坑。

以上！

相关阅读：[SparkSql数组操作的N种骚气用法](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485486&idx=1&sn=4b0a86c6fbec7cdb5ab6c05c3d0f0c92&chksm=fe6c5931c91bd0271612baefed99d0b992fd7d84ffdd9595d94ac921dccb508f9b01268fea48&scene=21#wechat_redirect)

精读源码，是一种有效的培养专长的方式~~

如果你想培养自己的优势

通过优势来提高自己在职场的影响力

但不知道如何开始

或者对自己没有信心

欢迎加入我创办的硬核源码学习社群

每周六直播，历史录屏，随到随学，长期陪跑，如果你有兴趣，欢迎加微信了解（一个小而美的学习社群）：

Hey!

我是小萝卜算子

欢迎关注公众号

每天学习一点点

知识增加一点点

思考深入一点点

在成为最厉害最厉害最厉害的道路上

很高兴认识你