---
title: sparksql源码系列 | ResolveReferences规则count(*)详解
author: 数据仓库践行者
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485109&idx=1&sn=d808812e988ea5669f39f2d52d0d9ae2&chksm=fe6c57aac91bdebc66f7830b0e0c7bef0244c9ede42847ab861d78739afbdcfa73f06cb41570&mpshare=1&scene=24&srcid=042535NgKLqdC82vKRW0nlLz&sharer_sharetime=1650899037718&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**大家想要获取《sparksql内核心剖析》电子书的话，一定记得后台发** **sparksql源码**

**这几个字，中间不要有空格，也不要加-。错误的示范：【spark-sql源码】、【spark sql源码】  这样是不会返回的，因为我设置的是精准匹配，看到有好多小伙伴发错了**

本文基于spark 3.2

这篇文章是做上次源码调试分享上留的一个作业题

|  |
| --- |
| 1、select \* from TESTDATA2，分析一下【\*】的情况，看看是怎么把【\*】转化为对应字段的。  匹配ResolveReferences中的这段代码：  case p: Project if containsStar(p.projectList) =>   p.copy(projectList = buildExpandedProjectList(p.projectList, p.child)) |

**sql:**

```
select * from testdata2
```

**对应astTree:**

# **unresolved logical plan 、resolved Logical Plan 以及这中间用到的规则：**

[生成resolved Logical Plan用的所有规则一览](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485068&idx=1&sn=198208226a184c380ebf2d8a3b8b21a2&chksm=fe6c5793c91bde854b5781996c10783e5a8a062fd4fa6f6dcd8b3522aea8959465e6561a82a4&scene=21#wechat_redirect)

# 

```
==  Parsed Logical Plan  =='Project [*]+- 'UnresolvedRelation [testdata2], [], false  
  
//*********************** 规则1************************org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations'Project [*]+- SubqueryAlias testdata2   +- View (`testData2`, [a#3,b#4])      +- SerializeFromObject [knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).a AS a#3, knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).b AS b#4]         +- ExternalRDD [obj#2]  
//*********************** 规则2************************org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveReferencesProject [a#3, b#4]+- SubqueryAlias testdata2   +- View (`testData2`, [a#3,b#4])      +- SerializeFromObject [knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).a AS a#3, knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).b AS b#4]         +- ExternalRDD [obj#2]  
== Analyzed Logical Plan ==Project [a#3, b#4]+- SubqueryAlias testdata2   +- View (`testData2`, [a#3,b#4])      +- SerializeFromObject [knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).a AS a#3, knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).b AS b#4]         +- ExternalRDD [obj#2]
```

**源码过程分析**

主要看Project [\*] 是怎么转化为 Project [a#3, b#4] 的，**ResolveReferences 规则的作用在源码共读分享上说过了：**

主要是把 UnresolvedAttribute 替换为AttributeReference

从代码，可以看到，把\*展开，用的是buildExpandedProjectList方法：

【\*】是UnresolvedStar，UnresolvedStar是Star的子类：

所以，会走第一个case，expand方法，而expand最终调用了UnresolvedStar 的 expand 方法：

我们来debug康康input.output 里面有啥：

这里的input是SubqueryAlias节点，output方法，实际上就是遵循逻辑执行计划的output方法，这个在上一次的源码共读分享中很详细的讲过了：

最后，总结一下：output每一步都是根据底部已经resloved的Attribute来给顶部的Attribute赋值，从而保证两个Attribute是指向同一个。

---

**我办了一个源码共读的实训活动（收费的），目前已经有100人参加了，主要是精读sparksql源码，每周六带大家共读调试1个半小时的源码，通过这个来提高我们的学习能力和独立深挖问题的能力，如果你有兴趣，欢迎加微信了解：**

Hey!

我是小萝卜算子

欢迎关注公众号

每天学习一点点

知识增加一点点

思考深入一点点

在成为最厉害最厉害最厉害的道路上

很高兴认识你

**推荐阅读：**

[sparksql源码系列 | 生成resolved logical plan的解析规则整理](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485068&idx=1&sn=198208226a184c380ebf2d8a3b8b21a2&chksm=fe6c5793c91bde854b5781996c10783e5a8a062fd4fa6f6dcd8b3522aea8959465e6561a82a4&scene=21#wechat_redirect)

[Spark DataSource API  v2 版本对比 v1有哪些改进？](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485057&idx=1&sn=0830eaa28a1b0553958c7bada56c359f&chksm=fe6c579ec91bde88f470e71f34b1ff245cfcbc81fb3e9953e708343b174478c80e7430bfc93b&scene=21#wechat_redirect)

[面试 | 你真的了解count(\*)和count(1)嘛？](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485023&idx=1&sn=1c2ca45b4c310d391eaf5e0e1b7533b1&chksm=fe6c5740c91bde563bafa2fb9193e3b881a095bae7431c25da2ebe9959c7820cc496ef979ab6&scene=21#wechat_redirect)

[sparksql源码共读 | 复习&答疑&大家遇到问题总结](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485049&idx=1&sn=83449b3544e1e02d8a085f8391af4722&chksm=fe6c5766c91bde70a1d4ffa3cd76ab5602733ee593dffa00f6d21caf90bd82276dbe2cfbdcd8&scene=21#wechat_redirect)

[澄清 | snappy压缩到底支持不支持split? 为啥？](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484994&idx=1&sn=2b7f7ac375517d34759412699f322133&chksm=fe6c575dc91bde4bf6ccbfcb1763a191e8b804b4e37ebd809a08f1072b864530b06dc607b8b0&scene=21#wechat_redirect)