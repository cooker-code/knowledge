---
title: sparksql源码系列 | 一文搞懂Partitioning源码体系(spark3.2)
author: 数据仓库践行者
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485292&idx=1&sn=907f316b9cae21411ad9fd76d073439b&chksm=fe6c5673c91bdf65c211d91cf5311317514cd9523b0b7195e32faaea1c9797f90a11ef977335&mpshare=1&scene=24&srcid=0526LLsZQVkG5meuieRqFjf7&sharer_sharetime=1653565183098&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

这篇文章主要介绍sparksql中Partitioning的源码体系，和上篇 [sparksql源码系列 | 一文搞懂Distribution源码体系(spark3.2)](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485282&idx=1&sn=548aaf891ad4b9c6569e0ffd39662636&chksm=fe6c567dc91bdf6bc2a97d50592c1c4b863a707cfa8a3068e0116c2500018d9fa710d7d5b816&scene=21#wechat_redirect)一样， Partitioning也是我们理解Physical Plan、executed Plan、shuffle、SparkSQL的AQE机制等的一个比较基础的知识点。

如果你正好也想了解这块，点赞、收藏吧，文末可加**微信**哦~

Partitioning定义了一个物理算子输出数据的分区方式，具体包括子Partitioning之间、目标Partitioning和Distribution之间的关系。

它用在什么地方呢？

每个physical operatior实现了outputPartitioning接口，以获得一个Partitioning的实例，**用于表示 operator输出数据满足的分布情况**。

## **类的依赖关系图**

* UnknownPartitioning：不进行分区
* SinglePartition：单分区
* RoundRobinPartitioning：在1-numPartitions范围内轮询式分区
* BroadcastPartitioning：广播分区
* HashPartitioning：基于哈希的分区方式
* RangePartitioning：基于范围的分区方式
* PartitioningCollection：分区方式的集合，描述物理算子的输出
* DataSourcePartitioning：V2 DataSource的分区方式

Partitioning接口定义如下：

```
trait Partitioning { //该sparkPlan输出RDD的分区数目  val numPartitions: Int //当前的partitioning操作能否得到所需的数据分布，当不满足时返回false，对数据进行重新组织 /** 需满足两个条件：  * 1、分区数numPartitions要相等   * 2、satisfies0方法返回true,satisfies0方法中写了和Distribution的关系  **/  final def satisfies(required: Distribution): Boolean = {    required.requiredNumPartitions.forall(_ == numPartitions) && satisfies0(required)  }  
  /**   * 1、如果requiredChildDistribution为UnspecifiedDistribution，则说明对子节点的分布没有要求，返回true   * 2、如果requiredChildDistribution为AllTuples，则只要numPartitions == 1，返回true   * 3、其他情况，返回false   **/  protected def satisfies0(required: Distribution): Boolean = required match {    case UnspecifiedDistribution => true    case AllTuples => numPartitions == 1    case _ => false  }}
```

**numPartitions：**指定该sparkplan输出的rdd分区数目。

**satisfies&satisfies0：**当前的partitioning操作能否得到所需的数据分布（required)。当不满足时，一般需要进行repartition操作，对数据进行重组织。做法就是添加exchange节点

Partitioning与Distribution关系理解

1、sparkplan定义了requiredChildDistribution接口，以获得一个Distribution的实例，**用于表示 operator对其input数据（**child节点的输出数据**）分布情况的要求**。

2、sparkplan定义了outputPartitioning接口，以获得一个Partitioning的实例，**用于表示 operator输出数据满足的分布情况**。

3、Distribution定义了createPartitioning接口，用来定义该distribution对应哪种Partitioning。

```
sealed trait Distribution { //分区数  def requiredNumPartitions: Option[Int]  
 //为Distribution创建默认分区，该分区可以满足此分布，同时匹配给定数量的分区。  def createPartitioning(numPartitions: Int): Partitioning}
```

```

```

4、Partitioning定义了satisfies接口，用来判断当前的partitioning操作能否得到所需的数据分布，当不满足时返回false。

总结一下

SparkPlan对输入数据的分布（Distribution）情况有着一定的要求，比如HashAggregateExec类型，要求输入数据key值按照hash方式分区，如果输入数据的分布无法满足（child.**outputPartitioning**.satisfies(**requiredChildDistributions**) ）当前节点的处理逻辑时，就需要添加一些shuffle操作来达到要求，体现在物理算子树上就是加Exchange节点。

决定要不要添加Exchange节点，主要是靠子节点的outputPartitioning(Partitioning)是否satisfies当前节点requiredChildDistributions(Distribution)来决定。

‍

以上

---

********精读源码，是一种有效的修炼技术内功的方式~~********

****我办了一个源码共读的实训活动，主要是精读sparksql源码，每周六带大家共读调试1个半小时的源码，通过这个来提高我们的学习能力和独立深挖问题的能力。****

********如****果你有兴趣，欢迎加微信了解：****

Hey!

我是小萝卜算子

欢迎关注公众号

每天学习一点点

知识增加一点点

思考深入一点点

在成为最厉害最厉害最厉害的道路上

很高兴认识你

**推荐阅读：**

[sparksql源码系列 | 最全的logical plan优化规则整理（spark2.3）](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485121&idx=1&sn=b96103b2a3f495421ed1114a24271a02&chksm=fe6c57dec91bdec855d937bd249d028136789449a2948a7c679f15db339a8c7a52aae26cfc16&scene=21#wechat_redirect)

[sparksql源码系列 | 一文搞懂with one count distinct 执行原理](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485214&idx=1&sn=98c7166eab5d129f36b30c7db4404b65&chksm=fe6c5601c91bdf1741a44a3a27ad03562d3bc3c58cbfa6fd571f4655b1c5ebc1e6958dde2ca7&scene=21#wechat_redirect)

[Sparksql源码系列 | 读源码必须掌握的scala基础语法](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485200&idx=1&sn=d50c0b2899aa0ceea9f53ffeabfe89ee&chksm=fe6c560fc91bdf19f2c56b6ca2d5326eadc2a8b8a7bff8b08623967c2f22a933ecbd08ae66ce&scene=21#wechat_redirect)

[澄清 | snappy压缩到底支持不支持split? 为啥？](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484994&idx=1&sn=2b7f7ac375517d34759412699f322133&chksm=fe6c575dc91bde4bf6ccbfcb1763a191e8b804b4e37ebd809a08f1072b864530b06dc607b8b0&scene=21#wechat_redirect)