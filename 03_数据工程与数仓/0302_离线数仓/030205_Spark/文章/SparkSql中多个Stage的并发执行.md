---
title: SparkSql中多个Stage的并发执行
author: 数据仓库践行者
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485386&idx=1&sn=8e377e5d37fb32077b534392b4c41c86&chksm=fe6c56d5c91bdfc327a8f410339516a387ba9e959957dcf980428ce67623aaa8678520163868&mpshare=1&scene=24&srcid=0709EkiUZi8OtbLGydIAkXtL&sharer_sharetime=1657334554554&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

写一篇水水的技术文，总结一下sparksql中不同stage的并行执行相关，也是来自于一位群友的提问：

我们群里有很多技术很棒并且很热心的大佬，哈哈~

**Hive中Job并发执行**

hive中，同一sql里，如果涉及到多个job，默认情况下，每个job是顺序执行的。

但如果某些job没有前后依赖关系的话，是阔以并行执行的，这样可能使得整个job的执行时间缩短。

可以通过设置参数 set hive.exec.parallel=true，实现job并发执行，该参数默认可以并发执行的job数为8，相关参数如下：

```
set hive.exec.parallel=true;              //打开任务并行执行set hive.exec.parallel.thread.number=16;  //同一个sql允许最大并行度，默认为8。
```

**Spark中多个Stage的并发执行**

**先给结论：**

* 没有相互依赖关系的Stage是可以并行执行的，比如union all 两侧的sql
* 存在依赖的Stage必须在依赖的Stage执行完成后才能执行下一个Stage
* Stage的并行度取决于资源数（配制的参数以及队列资源的核数、内存等），相关参数如下：

```
set spark.dynamicAllocation.enabled=true; --动态资源开关set spark.executor.cores=4; --每个Container分配的core数量set spark.dynamicAllocation.maxExecutors=1200;  --executor最大申请个数set spark.dynamicAllocation.initialExecutors=5;  -- 初始申请executor个数,默认等于minExecutorsset spark.dynamicAllocation.minExecutors =5; --executor最小申请个数  
大概就是这几个参数，代表能并行4*1200 ，也就是4800个task。把maxExecutors调大点，就能并行的更多
```

```

```

**源码角度的解释**

如果一个Stage有多个依赖，会依次递归（按stage id从小到大排列，也就是stage是从后往前提交的）提交父stages，直到到了根节点，如果有多个根节点，都会通过submitMissingTasks 提交上去运行。 

**提交stage:**

```
/** Submits stage, but first recursively submits any missing parents. */  private def submitStage(stage: Stage): Unit = {   //获取stage所属的active的JobId    val jobId = activeJobForStage(stage)    if (jobId.isDefined) {      logDebug(s"submitStage($stage (name=${stage.name};" +        s"jobs=${stage.jobIds.toSeq.sorted.mkString(",")}))")      //非waiting、running、failed的stage      if (!waitingStages(stage) && !runningStages(stage) && !failedStages(stage)) {        //获取该stage未提交的父stages，并按stage id从小到大排序，也就是stage是从后往前提交的        val missing = getMissingParentStages(stage).sortBy(_.id)        logDebug("missing: " + missing)        if (missing.isEmpty) {          logInfo("Submitting " + stage + " (" + stage.rdd + "), which has no missing parents")          //若无未提交的父stage, 则提交该stage对应的tasks          submitMissingTasks(stage, jobId.get)        } else {          //若存在未提交的父stage, 依次提交所有父stage (若父stage也存在未提交的父stage, 则提交, 依次类推)          for (parent <- missing) {            submitStage(parent)          }          //并把该stage添加到等待stage队列中          waitingStages += stage        }      }    } else {      abortStage(stage, "No active job for stage " + stage.id, None)    }  }
```

**获取未提交的父stages:**

```
//以参数stage为起点，向前遍历所有stage，判断stage是否为未提交，若使则加入missing中private def getMissingParentStages(stage: Stage): List[Stage] = {  val missing = new HashSet[Stage] //未提交的stage  val visited = new HashSet[RDD[_]]  //存储已经被访问到得RDD  // We are manually maintaining a stack here to prevent StackOverflowError  // caused by recursively visiting  val waitingForVisit = new ListBuffer[RDD[_]]  waitingForVisit += stage.rdd  def visit(rdd: RDD[_]): Unit = {    if (!visited(rdd)) {      visited += rdd      val rddHasUncachedPartitions = getCacheLocs(rdd).contains(Nil)      if (rddHasUncachedPartitions) {        for (dep <- rdd.dependencies) {          dep match {             //若为宽依赖，生成新的stage            case shufDep: ShuffleDependency[_, _, _] =>            //根据shufDep.shuffleId获取对应的ShuffleMapStage              val mapStage = getOrCreateShuffleMapStage(shufDep, stage.firstJobId)              if (!mapStage.isAvailable || !mapStage.shuffleDep.shuffleMergeFinalized) {                  //若stage得状态为available或者 基于push based shuffle合并没有完成，则为未提交stage                missing += mapStage              } else {                // Forward the nextAttemptId if skipped and get visited for the first time.                // Otherwise, once it gets retried,                // 1) the stuffs in stage info become distorting, e.g. task num, input byte, e.t.c                // 2) the first attempt starts from 0-idx, it will not be marked as a retry                mapStage.increaseAttemptIdOnFirstSkip()              }            case narrowDep: NarrowDependency[_] =>             //若为窄依赖，那就属于同一个stage。并将依赖的RDD放入waitingForVisit中，以能够在下面的while中继续向上visit，直至遍历了整个DAG图              waitingForVisit.prepend(narrowDep.rdd)          }        }      }    }  }  while (waitingForVisit.nonEmpty) {    visit(waitingForVisit.remove(0))  }  missing.toList}
```

以上！

---

********精读源码，是一种有效的修炼技术内功的方式~~********

********快来加入我创办的、最硬核的源码学习社群吧，精进的具体内容********：

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

**推荐阅读：**

[sparksql源码系列 | 一文搞懂Partitioning源码体系(spark3.2)](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485292&idx=1&sn=907f316b9cae21411ad9fd76d073439b&chksm=fe6c5673c91bdf65c211d91cf5311317514cd9523b0b7195e32faaea1c9797f90a11ef977335&scene=21#wechat_redirect)

[sparksql源码系列 | 一文搞懂Distribution源码体系(spark3.2)](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485282&idx=1&sn=548aaf891ad4b9c6569e0ffd39662636&chksm=fe6c567dc91bdf6bc2a97d50592c1c4b863a707cfa8a3068e0116c2500018d9fa710d7d5b816&scene=21#wechat_redirect)

[sparksql源码系列 | 一文搞懂with one count distinct 执行原理](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485214&idx=1&sn=98c7166eab5d129f36b30c7db4404b65&chksm=fe6c5601c91bdf1741a44a3a27ad03562d3bc3c58cbfa6fd571f4655b1c5ebc1e6958dde2ca7&scene=21#wechat_redirect)

[Sparksql源码系列 | 读源码必须掌握的scala基础语法](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485200&idx=1&sn=d50c0b2899aa0ceea9f53ffeabfe89ee&chksm=fe6c560fc91bdf19f2c56b6ca2d5326eadc2a8b8a7bff8b08623967c2f22a933ecbd08ae66ce&scene=21#wechat_redirect)

[澄清 | snappy压缩到底支持不支持split? 为啥？](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484994&idx=1&sn=2b7f7ac375517d34759412699f322133&chksm=fe6c575dc91bde4bf6ccbfcb1763a191e8b804b4e37ebd809a08f1072b864530b06dc607b8b0&scene=21#wechat_redirect)