---
title: SQL在查询过程中，遇到除数为0该怎么办？
author: 李岳AI
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA3MTg4NjY4Mw==&mid=2457328343&idx=2&sn=105143c9a3a18880942065fa4f0123b3&chksm=88a5c8e3bfd241f5a566d635cbc4e50c719612843872c139a171599441903e4682d6b7cf208c&mpshare=1&scene=24&srcid=0706NsR1SqavBSYu5LfhTD9n&sharer_sharetime=1657070115482&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**点击关注上方“****SQL数据库开发****”，**

**设为“置顶或星标****”，第一时间送达干货**

**SQL专栏**

[SQL基础知识第二版](http://mp.weixin.qq.com/s?__biz=MzA3MTg4NjY4Mw==&mid=2457300844&idx=1&sn=134dd477109fcff6593012b85e5d7c80&chksm=88a57d58bfd2f44ed56dacc801d7bc27fe941470bbaf5734f749289829c781d8f26ee011bdd8&scene=21#wechat_redirect)  
[SQL高级知识第二版](http://mp.weixin.qq.com/s?__biz=MzA3MTg4NjY4Mw==&mid=2457310325&idx=1&sn=f133bed8878a13be338c3fcaaf76c42b&chksm=88a58641bfd20f57c4ec00b1280926caea1f0b3851a95a35a03a7a7a9225b633d6e3d4d164d0&scene=21#wechat_redirect)

**问题**

我们在进行数据统计的时候，经常会遇到求百分比，环比，同比等这些需要除以某个数的情况，而如果除数为0，数据库是会报错的。

那么遇到这样的情况我们怎么处理呢？下面我们用示例给大家讲解一下处理方法。

**解决办法**

**情况一**

例如

```
SELECT  A/B FROM TAB
```

遇到这样的情况，一般的处理方法是用CASE WHEN来判断B的值

```
SELECT   
CASE WHEN B=0 THEN 0 ELSE A/B END    
FROM TAB
```

这样当B如果是0，我们直接赋一个值，避免A/B参与计算报错。

**情况二**

上面是一种常见的情况，但是如果遇到下面这样的聚合函数呢？

例如

```
SELECT  SUM(A)/COUNT(B) FROM TAB
```

遇到这样的情况CASE WHEN 不好判断COUNT(B)的值的，这个时候我们可以这样处理

```
SELECT    
ISNULL(SUM(A)/NULLIF(COUNT(B),0),0)   
FROM  TAB
```

其中这里使用了两个函数，NULLIF()和ISNULL()
NULLIF函数有两个参数,定义如下：

*NULLIF( expression1 , expression2 )*

其作用就是：如果两个指定的表达式相等，就返回NULL值。

ISNULL函数也有两个参数，定义如下：

*ISNULL( expression1 , expression2 )*

其作用是：如果第一个参数的结果为NULL，就返回第二个参数的值。

当COUNT(B)的结果为0时，恰好与第二个给定的参数0相等，这个时候NULLIF函数就会返回NULL，而SUM(A)在除以NULL时结果为NULL，外层使用ISNULL函数再对NULL值进行判断，这样最终结果就是0了。

这两种方法就是我们日常处理除数为0的情况了，一定要记得哦~

```
最后给大家分享我写的SQL两件套：《SQL基础知识第二版》和《SQL高级知识第二版》的PDF电子版。里面有各个语法的解释、大量的实例讲解和批注等等，非常通俗易懂，方便大家跟着一起来实操。

有需要的读者可以下载学习，在下面的公众号「数据前线」(非本号)后台回复关键字：SQL，就行

数据前线

#### ``` 后台回复关键字：1024，获取一份精心整理的技术干货 后台回复关键字：进群，带你进入高手如云的交流群 ``` 推荐阅读 * 经典SQL语句大全 * SQL中常用的四个排序函数 * SQL 常用函数 * 不懂就问：SQL 语句中 where 条件后 写上1=1 是什么意思 * 推荐一个 SQL 学习刷题网站！ ``` ```
```