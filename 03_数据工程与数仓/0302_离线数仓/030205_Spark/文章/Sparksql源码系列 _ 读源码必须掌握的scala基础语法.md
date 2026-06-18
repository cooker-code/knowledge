---
title: Sparksql源码系列 | 读源码必须掌握的scala基础语法
author: 数据仓库践行者
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485200&idx=1&sn=d50c0b2899aa0ceea9f53ffeabfe89ee&chksm=fe6c560fc91bdf19f2c56b6ca2d5326eadc2a8b8a7bff8b08623967c2f22a933ecbd08ae66ce&mpshare=1&scene=24&srcid=0512te8mZLqb1lvPjTF2zidI&sharer_sharetime=1652328654553&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

这篇文章总结一下我在学习spark sql源码时，曾经纠结过的一些scala语法。

在精读sparksql源码之前，我们需要有一定的scala语法知识，来保证能够看懂sparksql代码，并上手调试。 

有同学不会scala，就会有一种恐惧心理，其实不用怕，因为我一开始也不会scala代码。我是边看sparksql源码，边学习语法，看到不懂的地方，就从网上搜索相关的语法，把相关语法弄懂了之后，再写个scala的测试类，实现一个案例执行一下，加深理解，然后，再继续读源码。 

到现在，我已经能熟练的用scala写一段了，虽然写不是很规范，但是看懂、调试、修改代码完全没有问题。  并且边用边学这种方式效率很高，这么说，并不是鼓励大家都用我这种方式，如果有条件，还是从网上找一些scala的基础视频看看，提前学一学，肯定会更好~

**1、偏函数**

当在调用一个函数时，把这个函数应用到参数中。如果传递所有预期的参数，则表示您已完全应用它。如果只传递几个参数并不是全部参数，那么将返回部分应用的函数。这样就可以方便地绑定一些参数，其余的参数可稍后填写补上。

```
trait PartialFunction[-A, +B] extends (A => B) { self =>  import PartialFunction._  def isDefinedAt(x: A): Boolean    def orElse[A1 <: A, B1 >: B](that: PartialFunction[A1, B1]): PartialFunction[A1, B1] =    new OrElse[A1, B1] (this, that)   override def andThen[C](k: B => C): PartialFunction[A, C] =    new AndThen[A, B, C] (this, k)  
  def lift: A => Option[B] = new Lifted(this)  
  def applyOrElse[A1 <: A, B1 >: B](x: A1, default: A1 => B1): B1 =    if (isDefinedAt(x)) apply(x) else default(x)  
  def runWith[U](action: B => U): A => Boolean = { x =>    val z = applyOrElse(x, checkFallback[B])    if (!fallbackOccurred(z)) { action(z); true } else false  }}
```

可以说，sparksql的源码中，到处都是偏函数。

比如：生成解析后逻辑执行计划中的解析器、优化逻辑执行计划的优化器等。

逻辑执行计划解析器ResolveRelations（解析表和视图）：

逻辑执行计划优化器ColumnPruning（列剪裁）：

**2、嵌套函数**

Scala允许定义函数内部的函数，而在其他函数中定义的函数称为局部函数。

sparksql源码中有太多这样的应用。

比如QueryPlan类中mapExpressions方法：

比如TreeNode类中legacyWithNewChildren方法：

**3、柯里化**函数****

柯里化(Currying)函数是一个带有多个参数，并引入到一个函数链中的函数，每个函数都使用一个参数。

比如ParseDriver中的parse方法：

parse方法是个scala语法中的柯里化函数，它有两个输入参数，一个是查询语句，另外一个参数是方法参数。

toResult方法的实现是通过柯里化函数的参数传入的。

****4、可变参数函数****

Scala允许指定函数的最后一个参数可重复，这允许客户端将可变长度参数列表传递给函数。

**5、case模式匹配**

用的最多，解析规则、优化器中会经常用到

**6、case类**

case类在模式匹配中经常使用到，当一个类被定义成为case类后：

* Scala会自动创建一个伴生对象并实现了apply方法，我们使用时不需要用new关键字就能创建该类对象。
* 实现了unapply方法，可以通过模式匹配来获取类属性。
* 实现了类构造参数的getter方法（构造参数默认被声明为val）
* 实现了toString,equals，copy和hashCode等方法

为了方便模式匹配，LogicalPlan、SparkPlan都是case类

****7、case类的copy()方法****

copy()方法返回当前对象的复制，可以通过传递属性名 = 值的方式来自定义赋值出的对象的值

ColumnPruning（列裁剪）优化器，通过copy方法把子节点中不需要的列裁剪掉：

**8、product类**

TreeNode继承product类，通过Product类中的方法(productArity、productElement、productIterator)来操纵TreeNode实现类的参数

mapProductIterator：

**9、scala隐式类**

# Scala中有个隐式转换系统，包括隐式参数 、隐式类、隐式对象等。

Scala中的隐式类是对类功能增强的一种形式。

比如AstBuilder导入ParserUtils，用到optionalMap方法：

而optionalMap方法是在ParserUtils中的EnhancedLogicalPlan隐式类里定义的：

**10、foldLeft**

在sparksql源码中第一次看到foldLeft语法时，理解了好长时间，才弄明白。

比如规则执行器RuleExecutor：

以上列了10种比较特殊的语法，还有一些，比如：

* 列表（List）、集合（Set）、映射（Map）、选项（Option）、元组（Tuple）这些集合的基础用法
* 类、对象、特质、继承等这些概念的理解

大家在学习scala时，重点关注一下就ok!

---

****我办了一个源码共读的实训活动（收费的），主要是精读sparksql源码，每周带大家共读调试1个半小时的源码，通过这个来提高我们的学习能力和独立深挖问题的能力。****

****详情戳：********[Sparksql源码实训第二期](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485159&idx=1&sn=25c994e7ba09088ef1a30c109812cd63&chksm=fe6c57f8c91bdeee5ebe6ba6f8ccca6ba254af1809d2747a6ce43903070c2c3d0eea04c68b6a&scene=21#wechat_redirect)****

****目前已经有120+人参加了，如********果你有兴趣，欢迎加微信了解：****

Hey!

我是小萝卜算子

欢迎关注公众号

每天学习一点点

知识增加一点点

思考深入一点点

在成为最厉害最厉害最厉害的道路上

很高兴认识你

**推荐阅读：**

[源码视频系列 | Show create table执行原理](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485168&idx=1&sn=15d42ad11920f0aa374c6bb71f438924&chksm=fe6c57efc91bdef9adbb84cae6f7a1441c12c834ff1035afbc7b926fc3cbe9d6ef231f7f973d&scene=21#wechat_redirect)

[sparksql源码系列 | 一文搞懂Show create table 执行原理](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485144&idx=1&sn=4aba68f864d284c41db75bee878f3e85&chksm=fe6c57c7c91bded11c3af9bb37147f31f0fc7586bd577b43089f3316581596d01fc20dffc5de&scene=21#wechat_redirect)

[sparksql源码系列 |  最全的logical plan优化规则整理（spark3.2）](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485121&idx=1&sn=b96103b2a3f495421ed1114a24271a02&chksm=fe6c57dec91bdec855d937bd249d028136789449a2948a7c679f15db339a8c7a52aae26cfc16&scene=21#wechat_redirect)

[澄清 | snappy压缩到底支持不支持split? 为啥？](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484994&idx=1&sn=2b7f7ac375517d34759412699f322133&chksm=fe6c575dc91bde4bf6ccbfcb1763a191e8b804b4e37ebd809a08f1072b864530b06dc607b8b0&scene=21#wechat_redirect)

[以后的事谁也说不准](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485016&idx=1&sn=2fe05cee0c69a8b4ae9b026132f8827c&chksm=fe6c5747c91bde5113c93c9f16e839f7b5d8738b858e9171a5f8e86e50bf28538ae0a85a7495&scene=21#wechat_redirect)

[转型【数仓开发】该怎么学](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484695&idx=1&sn=d0b793b6c2a09e048ff9a21f4b4ee1d0&chksm=fe6c5408c91bdd1ebddde0e9e1777d2afd755c250d15877c79b94eae1e2009d31080df18b16f&scene=21#wechat_redirect)

[大数据开发轻量级入门方案](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484678&idx=1&sn=a13955f253d989e728685e8c79513cb8&chksm=fe6c5419c91bdd0f4af36413a08a29faf054b20a2c2011c2720b0df37d0ae768b7af8030713c&scene=21#wechat_redirect)

[OLAP | 基础知识梳理](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484657&idx=1&sn=151480a2dc70e93eaac28404a4c7bd61&chksm=fe6c55eec91bdcf8916040a64cefd4be3b089e1093c07c23d3efe6b783f9c23ce523e798b8f2&scene=21#wechat_redirect)

[Flink系列 - 实时数仓之数据入ElasticSearch实战](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484637&idx=1&sn=dc579db216d0b700b7378b5d63af4a0a&chksm=fe6c55c2c91bdcd4b1c9bcd73273620cc22c7a88a9e734e91b9ec7a356d1e7d53fd03fd65e47&scene=21#wechat_redirect)

[Flink系列 - 实时数仓之FlinkCDC实现动态分流实战](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484637&idx=2&sn=9a5ca67783e341e318f0c1c92875b7b5&chksm=fe6c55c2c91bdcd4f30e5d665ee58c8e620bdc95d0879a84a898b77afbba3c64f1a0100c552b&scene=21#wechat_redirect)