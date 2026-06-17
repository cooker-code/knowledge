---
title: 面试被问Spark中的血缘关系，您是否清楚从哪些方面阐述呢？
author: 假装不靠谱儿
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI4MDkzNTg4NQ==&mid=2247503866&idx=1&sn=7925ae02488842d209a48c28389adb64&chksm=ebb26358dcc5ea4e2ae0a9d6ffb87ae4455bc0739b8770ba0ca0ab736ca3a7e283f0d85898cd&mpshare=1&scene=24&srcid=1214xzUS75KNFWXjZkuvSie0&sharer_shareinfo=3bbdf958d26e412c07156ea6c470431f&sharer_shareinfo_first=3bbdf958d26e412c07156ea6c470431f#rd
---

点击蓝字 · 关注我们

**前言**

我们知道Spark中的RDD可以从本地集合、外部文件系统创建，从其它RDD转化得到。从其它RDD通过转换算子得到新的RDD，这两个RDD之间具有依赖关系，即血缘。RDD和它依赖的父RDD之间有两种不同的依赖类型，即宽依赖和窄依赖。下面我们具体介绍一下Spark中的血缘关系。

**一**

**血缘关系**

RDD只支持粗粒度转换，即在大量记录上执行的单个操作。将创建RDD的一系列Lineage（血统）记录下来，以便恢复丢失的分区。RDD的Lineage会记录RDD的元数据信息和转换行为，当该RDD的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。

**代码实现：**

```
object Lineage01 {  def main(args: Array[String]): Unit = {  
    //1.创建SparkConf并设置App名称    val conf: SparkConf = new SparkConf().setAppName("SparkCoreTest").setMaster("local[*]")  
    //2.创建SparkContext，该对象是提交Spark App的入口    val sc: SparkContext = new SparkContext(conf)  
    //3.加载数据进行并进行数据转换    val rdd: RDD[String] = sc.makeRDD(List("hello world", "hello spark"))    val wordRDD: RDD[String] = rdd.flatMap(_.split(" "))    val mapRDD: RDD[(String, Int)] = wordRDD.map((_,1))    val resultRDD: RDD[(String, Int)] = mapRDD.reduceByKey(_+_)        //4.打印RDD之间的血缘关系    println(resultRDD.toDebugString)  
    //5.触发计算    resultRDD.collect()  
    //6.关闭连接    sc.stop()  }}
```

**查看运行结果：**

结果说明：圆括号中的数字12表示RDD的并行度，也就是有几个分区，[0]是第一步操作，创建集合RDD，[1]是第二步操作，执行flatMap算子，[2]是第三步操作，执行map算子，[3]是第四步操作，执行reduceByKey算子，因reduceByKey算子存在Shuffle过程，所以结果前面缩进显示，代表新的stage划分。血缘关系将RDD的一系列操作记录下来，当该RDD的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。

**二**

**依赖关系**

RDD之间的关系可以从以下角度来理解，RDD是从哪些RDD转换而来，RDD依赖于父RDD的哪些Partition，这种关系就是RDD之间的依赖。

**代码实现：**

```
object Lineage02 {  def main(args: Array[String]): Unit = {  
    //1.创建SparkConf并设置App名称    val conf: SparkConf = new SparkConf().setAppName("SparkCoreTest").setMaster("local[*]")  
    //2.创建SparkContext，该对象是提交Spark App的入口    val sc: SparkContext = new SparkContext(conf)  
    //3.打印RDD之间的依赖关系    val rdd: RDD[String] = sc.makeRDD(List("hello world","hello spark"))    println(rdd.dependencies)    println("----------------------")  
    val wordRDD: RDD[String] = rdd.flatMap(_.split(" "))    println(wordRDD.dependencies)    println("----------------------")  
    val mapRDD: RDD[(String, Int)] = wordRDD.map((_,1))    println(mapRDD.dependencies)    println("----------------------")  
    val resultRDD: RDD[(String, Int)] = mapRDD.reduceByKey(_+_)    println(resultRDD.dependencies)  
    //4.触发计算    resultRDD.collect()  
    //5.查看localhost:4040页面，观察DAG图    Thread.sleep(1000000)  
    //6.关闭连接    sc.stop()  }}
```

**查看运行结果：**

结果说明：第一步从集合中创建RDD，并是从其它RDD转换而来，没有RDD之间的依赖关系，所以执行sc.makeRDD打印依赖关系结果为List()，而后的flatMap算子和map算子对RDD进行数据转换操作，每一个父RDD的分区最多被子RDD的一个分区使用，所以依赖结果是OneToOneDependency，而reduceByKey，同一个父RDD的分区被多个子RDD的分区依赖，是一对多的关系，会引起Shuffle，所以依赖结果是ShuffleDependency。

**查看DAG图：**

注意：RDD和它依赖的父RDD的依赖关系有两种不同的类型，即窄依赖（NarrowDependency）和宽依赖（ShuffleDependency）。

**窄依赖：**窄依赖表示每一个父RDD的Partition最多被子RDD的一个Partition使用，是一对一或者多对一的关系。如下图所示：

**宽依赖：**宽依赖表示同一个父RDD的Partition被多个子RDD的Partition依赖，是一对多的关系，会引起Shuffle。如下图所示：

**三**

**总结**

通过这篇文章，大家对Spark RDD的血缘关系是不是有进一步的了解了，RDD的血缘关系是Spark的容错机制之一，当某个RDD的部分分区数据丢失时，它可以根据血缘关系来重新运算，以恢复丢失的数据分区。

**END**

NO.1

**往期回顾**

[Spark双Value算子是否产生Shuffle](https://mp.weixin.qq.com/s?__biz=MzI4MDkzNTg4NQ==&mid=2247501389&idx=2&sn=d0aa9d4a503e341767457317b343eadc&chksm=ebb26aefdcc5e3f9ec975493ca3b42540f91fb6ca93baf3ea07bdc8969ca3ebd5c050e78fdaf&scene=21#wechat_redirect)

[Spark Streaming窗口函数](https://mp.weixin.qq.com/s?__biz=MzI4MDkzNTg4NQ==&mid=2247500617&idx=2&sn=7ab85e0272fa321f32e3a5ad886ca5ac&chksm=ebb26febdcc5e6fd33cd8d4b39a70bbac84243c9015aad2400e487cb290538ee075f60c23365&scene=21#wechat_redirect)

[SPARK | 简述reduceByKey与groupByKey的区别](https://mp.weixin.qq.com/s?__biz=MzI4MDkzNTg4NQ==&mid=2247500573&idx=3&sn=1bd937aa984ae18147c19fc51bb47ddd&chksm=ebb26fbfdcc5e6a9b800c21f047c33a9ad250f5ae1a1a84732ba0206ed86b83a4424aa5efec6&scene=21#wechat_redirect)

[你怎么理解Spark RDD Cache缓存？](https://mp.weixin.qq.com/s?__biz=MzI4MDkzNTg4NQ==&mid=2247500418&idx=3&sn=88975953cd21a5838888f7bbc4677e14&chksm=ebb26e20dcc5e7369d9e89da567b3df30f38b0428d1590fab9f6e546ad164e9a4f3a201ee09d&scene=21#wechat_redirect)

分享，点赞，在看，

都在这儿,点我不香吗？