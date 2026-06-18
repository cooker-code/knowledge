> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030202_Hive/030202_核心知识点/Hive拉链表与累积型快照事实表|Hive拉链表与累积型快照事实表]]
---
title: 数仓面试高频-如何在Hive中实现拉链表
author: 涤生大数据
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489455&idx=1&sn=3265f322898810f682fdfc3f513bee6b&chksm=cf6222fdf815abebb193cbd7dedce40708557aa451cbdb876b09b6791a6f188850cf45ad4be7&mpshare=1&scene=24&srcid=0704jLbqgPyl3AHp7SlL6PF6&sharer_shareinfo=d9a023e41c42cf6d8606bc8d6ba0cc54&sharer_shareinfo_first=d9a023e41c42cf6d8606bc8d6ba0cc54#rd
---

**1.什么是拉链表**

拉链，就是记录一个事物从开始，一直到当前状态的所有变化的信息。拉链表是一个逻辑上的概念，是针对数据仓库设计中表存储的方式定义而来，拉链表有数据的开始日期和结束日期，记录着数据的生命周期。总而言之，拉链表通过增量表进行不断的更新。

我们先看一个示例，这就是一张拉链表，存储的是用户购买商品的基本信息，另外也记录了每条订单的生命周期。下图展示订单号为1 的记录生命周期，我们可以很方便的使用这张表拿到最新的数据以及这条订单历史上的数据。

**2.拉链表产生的原因**

我们都知道hive的主要运用场景就是来做离线数仓，为了更好地构建离线数仓，我们是需要定时的从各种数据源总同步采集数据到hive中。数据同步分为增量同步和全量同步。全量同步，就是每天都将业务数据库中的全部数据同步一份到数据仓库，这是保证两侧数据同步的最简单的方式。增量同步，就是每天只将业务数据中的新增及变化数据同步到数据仓库。采用每日增量同步的表，通常需要在首日先进行一次全量同步。

例如，每天需要从MySQL中同步最新的订单信息、用户信息、店铺信息等到数据仓库中。假如我们MySQL中有一张用户表：t\_users，每个用户注册完成以后，就会在用户表中新增该用户的信息，记录该用户的id、手机号码、用户名、性别、地址等信息。

在以后的时间里表中的部分字段会被update更新操作，如用户联系方式，产品的描述信息，订单的状态等等。

2021-01-01：MySQL数据库中有10条用户信息

日期为：2021-01-02：MySQL数据库中新增2条用户注册数据，并且有008用户数据发生更新：

那么对于这种表我该如何设计呢？下面有几种方案可选：

方案一：每天只留最新的一份，每天抽取最新的一份全量数据到Hive中。优点：这个实现最简单，使用起来最方便，缺点：没有历史状态，008的地址是1月2号在sh，但是1月2号之前是在gz的，如果要查询008的1月2号之前的addr就无法查询，也不能使用sh代替。

方案二：使用分区表，每天保留一份全量的切片数据。记录了所有数据在不同时间的状态，冗余存储了很多没有发生变化的数据，导致存储的数据量过大。这个其实在企业中用的比较多，因为很多表并不是大表，我们可以用自己的空间换时间，来达到自己数据存储的目的。

方案三：使用拉链表。通过时间标记发生变化的数据的每种状态的时间周期。其实它能满足方案二所能满足的需求，既能获取最新的数据，也能添加筛选条件也获取历史的数据。就行下面的这幅图。

这里需要注意的是starttime表示该条记录的生命周期开始时间，endtime表示该条记录的生命周期结束时间。endtime = '9999-12-31'表示该条记录目前处于最新的状态。一般我们查有效状态的时候都加这个条件。如果查询当前所有有效的记录，则select \* from t\_user where endtime = '9999-12-31'。

如果查询2021-01-02的历史快照，则select \* from user where endtime <= '2021-01-02' and t\_end\_date >= '2021-01-02'。特出场景需要我们查询历史快照表。

**3.在Hive中实现拉链表**

七四在mysql忠或者其他数据库中我们可以更新表数据来达到拉链表的完成，但是在大数据场景下，大部分的公司都会选择以Hdfs分布式文件系统来存储数据，也会使用hive或者spark来完成计算。但是hdfs文件系统或者说hive他并不适合使用更新数据的操作，这个时候我们就需要变着法子来实现拉链表。

我们上面提到的用户表为例，我们要实现用户的拉链表。在实现它之前，我们需要先确定一下我们有哪些数据源可以用。

1.我们需要一张贴源层的的用户全量表。因为我们需要他进行拉链表的初始化动作。

2.需要做一张用户增量表，这个每天增量抽数即可。

整体实现过程一般分为4步首先初始化拉链表，第2步，先使用增量抽数的方式采集所有新增数据（就是新增数据或者变化数据）放入一张增量表。第3步创建一张临时表，用于将老的拉链表与增量表进行合并。第三步，最后使用overwrite将做好的临时表的数据覆盖写入拉链表中。

step1：我们需要用户全量表，初始化拉链表；

Step2：增量抽取变化的数据，放入增量表中

Step3：构建临时表，将Hive中的拉链表与临时表的数据进行合并

Step4：将临时表的数据覆盖写入拉链表中 

```
核心代码：insert overwrite table t_user_zipperselect  userid,  phone,  nick,  gender,  addr,  starttime,  endtimefrom ods_user_update bon a.userid = b.userid ;union all--查找原来拉链表里的所有数据，并将这次需要更新的增量数据的endTime更改为更新值的startTimeselect  a.userid,  a.phone,  a.nick,  a.gender,  a.addr,  a.starttime,  --但是这条数据没有更新或者这条数据不是要更改的数据，就保留原来的值，否则就改为新数据的开始时间-1，也就是昨天。  if(b.userid is null or a.endtime < '9999-12-31', a.endtime , date_sub(b.starttime,1)) as endtimefrom t_user_zipper  a  left join ods_user_update bon a.userid = b.userid ;
```

**4.总结**

拉链表是一种有效的数据仓库技术，它可以实现对数据的长期追踪和存档，同时节省存储空间。然而，拉链表也会面临一些挑战。首先，需要定期对数据进行归档和清理，这增加了数据处理的工作量。其次，查询时需要考虑时间戳字段，这增加了查询的复杂性。最后，对于需要频繁更新的数据，可能需要频繁地更新时间戳字段，这也增加了数据处理的工作量。

涤生大数据往期精彩推荐

1.[企业数仓DQC数据质量管理实践篇](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486027&idx=1&sn=bb92fa17fb2a12fb70ba8f4e2862699d&chksm=cf623f19f815b60fd49d9b18da21f73795a92566f95b9ff41046e7af0e72840b41b42a21f991&scene=21#wechat_redirect)

2.[企业数据治理实战总结--数仓面试必备](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486189&idx=1&sn=1360271d53b9c3a6e6bd16041ad8dd99&chksm=cf623fbff815b6a9b1ec61726e4b65b1495e27673b4d08b280d57924082b79755bc17462d24e&scene=21#wechat_redirect)

3.[OneData理论案例实战—企业级数仓业务过程](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486715&idx=1&sn=0a5d160583b8ddba256f5edf5c34767e&chksm=cf6239a9f815b0bf81fecfe3ecacd118a363b0d6912c5bc9bce094926a7f5f619abd31a8a044&scene=21#wechat_redirect)

4.[中大厂数仓模型规范与度量指标有哪些？](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486864&idx=1&sn=29f5ca1ee9ff947b4248a4e653706a4d&chksm=cf6238c2f815b1d42135e35800f77713bd1f2e7913fc473be0ee7ea05da45a145829bd7ea8bb&scene=21#wechat_redirect)

5.[手把手教你搭建用户画像系统（入门篇上）](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487012&idx=1&sn=6d12ebda43874d05c05e8cd2a24ec5c6&chksm=cf623b76f815b2602eb9b5da67d07a0e51575166ac1e31d13f42895239872b4be8a46ac1cf87&scene=21#wechat_redirect)

6.[手把手教你搭建用户画像系统（入门篇下）](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487053&idx=1&sn=ef949d4b7474526519998ff332abc4f0&chksm=cf623b1ff815b2091940f96db6623dc1c803be662c5d10a6d56ee6744cbeb36be00e111a8e9d&scene=21#wechat_redirect)

7.[SQL优化之诊断篇：快速定位生产性能问题实践](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487054&idx=1&sn=bb4a0df0fb986f9f3706f39b997771c7&chksm=cf623b1cf815b20a32ebd5546f3c92ad91617e6c03e4e28fb9e69a40425a88ce93cc805e7861&scene=21#wechat_redirect)

8.[SQL之优化篇：一文搞懂如何优化线上任务性能，增效降本！](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487073&idx=1&sn=230f8260142d9f8c26c871f4e580a322&chksm=cf623b33f815b22584a1fbd4ae7e9947221defa7aca0c6f5d4beb446f8101ffd469b1c882fb5&scene=21#wechat_redirect)

9.[新能源趋势下一个简单的数仓项目，助力理解数仓模型](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487251&idx=1&sn=19c8a32f674a1de5a9ac081673d200ac&chksm=cf623a41f815b357d81498488c623180b40ce7522abebb3a17c92b01c112f29f6d8ea97e5c57&scene=21#wechat_redirect)

10.[基于FlinkSQL +Hbase在O2O场景营销域实时数仓的实践](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487723&idx=1&sn=30f99be1730dfbfa303741f46b81c3ed&chksm=cf6225b9f815acafcc627bf071999209090beef02bf56e8a5eb4994bcd52d36aa977361265ef&scene=21#wechat_redirect)

11.[开发实战角度：distinct实现原理及具体优化总结](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487792&idx=1&sn=5e097df0c88e0a0aa3f2a3d0b5103f62&chksm=cf622462f815ad74e0728dd7fae8c04ae2a219cacf959f16cc79eb7a25da8036ae744236bf3d&scene=21#wechat_redirect)

12.[涤生大数据实战：基于Flink+ODPS历史累计计算项目分析与优化（一）](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487866&idx=1&sn=041685cb5b2135e4d75b7aab49e36347&chksm=cf622428f815ad3ef65fbbe90769541346106625dde774e20ca2aec0dd58aa96d1c7359af978&scene=21#wechat_redirect)

13.[涤生大数据实战：基于Flink+ODPS历史累计计算项目分析与优化（二）](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487874&idx=1&sn=15610fda5195189c32994b77eff822e5&chksm=cf6224d0f815adc6b5ee69cf530dc12be7141a3499641826a292ca896ce173ad3ee56fabda46&scene=21#wechat_redirect)

14.[5分钟了解实时车联网，车联网（IoV)OLAP 解决方案是怎样的？](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487981&idx=1&sn=f638499eb34a3b03fbd51c95b2f4be39&chksm=cf6224bff815ada9ab9017f3425de1751e38e664090ffdb37069f19579fdf13870e6d801ba73&scene=21#wechat_redirect)

15.[企业级Apache Kafka集群策略：Kakfa最佳实践总结](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247488177&idx=1&sn=5469c0af700aff357384e62b3b9cecfd&chksm=cf6227e3f815aef5c006659bc0835f42fb94ecf951c7f73a6c9dfa577e717e1d2ae0f670cde1&scene=21#wechat_redirect)

16.[玩转Spark小文件合并与文件读写提交机制](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247488932&idx=1&sn=acda9e39d8fc98b145e422179a7b5da1&chksm=cf6220f6f815a9e06b88e0e21c347371236867a248dae17cc456092652d34411b5e7fa803b02&scene=21#wechat_redirect)

17.[一文详解Spark内存模型原理，面试轻松搞定](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489016&idx=1&sn=9cce9cad9636a008429d7447c4354583&chksm=cf6220aaf815a9bce52e85681de705c8eaa48bd5668216b83fb58c465d360ffb1251f0fc146c&scene=21#wechat_redirect)

18.[大厂8年老司机漫谈数仓架构](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489193&idx=1&sn=0c3d73dcff09e74b14eb999a0006b7ca&chksm=cf6223fbf815aaed3c4ee5f541bdc94fc437d5720be66bff18c673ba7e4b4b11376a3ff8bcad&scene=21#wechat_redirect)

19.[一文带你深入吃透Spark的窗口函数](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489211&idx=1&sn=9f7731af8cd1361444cf097cfeb509fd&chksm=cf6223e9f815aaff227a3049ac5236bb59d961f604e63130b5c514eeb0bea2778fd6128f9b20&scene=21#wechat_redirect)

20.[大数据实战：基于Flink+ODPS进行最近N天实时标签构建](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489298&idx=1&sn=e14a0fdb270d0b751dc8fb4f00608142&chksm=cf622240f815ab563f0d2c9cf0ce26dd94000fd5ef749a930ec65b90fd13ae90fcb0080039d4&scene=21#wechat_redirect)

21.[数仓面试还不懂什么是基线管理？](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489345&idx=1&sn=19c3954a7fbfda3377859bc5ac0b6b7d&chksm=cf622213f815ab052e1dcf8c6d83ea936920519af23d76ab658c2ba1055a0a59e4ab36467c3d&scene=21#wechat_redirect)