---
title: SparkSql序列化时列的ID是在哪里生成的呢？
author: 数据仓库践行者
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485524&idx=1&sn=6f2003179fa9db5a5c98e037658a33f2&chksm=fe6c594bc91bd05dad35ea0cbab275fd9fc4377aa905de648b20f3f62f1d4b1b02d01d25ba76&mpshare=1&scene=24&srcid=0719tGu4HHmwyAZcGL5grdxe&sharer_sharetime=1658192784697&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

面向群友写文，哈哈

有点抽象，但群友们一定知道我在写什么

（原谅我，喜欢晒截图）

分享课上没有trace详细代码，这篇顺一下这块的代码。

sparksql生成解析后的逻辑执行计划时，会通过catalog把各个字段和元数据库绑定，也就说在ResolveLogical的阶段的字段是带了id的：

```
SELECT A,B FROM TESTDATA2  
==  Parsed Logical Plan  =='Project ['A, 'B]+- 'UnresolvedRelation [TESTDATA2], [], false  
== Analyzed Logical Plan ==Project [A#3, B#4]+- SubqueryAlias testdata2   +- View (`testData2`, [a#3,b#4])      +- SerializeFromObject [knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).a AS a#3, knownnotnull(assertnotnull(input[0, org.apache.spark.sql.test.SQLTestData$TestData2, true])).b AS b#4]         +- ExternalRDD [obj#2]
```

可以看到从未解析到解析，字段由**'Project ['A, 'B] --> Project [A#3, B#4]**

# **那这个id是什么时候生成的呢？**

id是在建表时或者创建临时视图时生成的。

我们以createOrReplaceTempView为例来看一下：

准备TESTDATA2测试数据时的逻辑——

# **1、SQLTestData 类中，生成testData2**

# 

# **2、SQLImplicits隐式转换把rdd转成DataSet**

# 

# **3、SQLImplicits 类的执行流程**

# 

# **SQLImplicits --> LowPrioritySQLImplicits -->newProductEncoder -->Encoders.product[T] -->** **ExpressionEncoder**

下面图按顺序：

从上图可知会用到ExpressionEncoder类

# **4、ExpressionEncoder类的运行流程**

**sparksql源码中有很多操作是初始化类的时候做的**

**ExpressionEncoder.apply 这里计算：**

```
val serializer = ScalaReflection.serializerForType(tpe)val deserializer = ScalaReflection.deserializerForType(tpe)
```

**-->** **new ExpressionEncoder[T](serializer,   deserializer, ClassTag[T](cls))**

**-->** **ExpressionEncoder.serializer (序列化操作)**

****-->CreateNamedStruct.flatten(匹配到If的分支调用******CreateNamedStruct.flatten)**********

****-->Alias(v, n.toString)(起别名)****

********-->exprId = NamedExpression.newExprId********(********ExprId****************************就是序列化的id********************)****************

****************--> ExprId(curId.getAndIncrement(), jvmId)**(******序列化的id最终生成********************)********************************

代码流程如下截图：

下篇写写createOrReplaceTempView的运行原理~

---

****************精读源码，是一种有效的培养自己专长的方式~~****************

********快来加入我创办的、最硬核的源码学习社群********（收费的）********吧，精进的具体内容********：

****************[SparkSql源码成神之路](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485325&idx=1&sn=dc2ea61206c63ca7d3089859d67ec227&chksm=fe6c5692c91bdf8475720aa329df306a04113aa9cd8b5c4016c1adff5de540d5a5045f9e7540&scene=21#wechat_redirect)****************

****************每周六直播，历史录屏，随到随学******，**************************长期陪跑，************************如****果你有兴趣，欢迎加微信了解（一个小而美的学习社群）：****

Hey!

我是小萝卜算子

欢迎关注公众号

每天学习一点点

知识增加一点点

思考深入一点点

在成为最厉害最厉害最厉害的道路上

很高兴认识你

**推荐阅读：**[SparkSql中多个Stage的并发执行](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485386&idx=1&sn=8e377e5d37fb32077b534392b4c41c86&chksm=fe6c56d5c91bdfc327a8f410339516a387ba9e959957dcf980428ce67623aaa8678520163868&scene=21#wechat_redirect)