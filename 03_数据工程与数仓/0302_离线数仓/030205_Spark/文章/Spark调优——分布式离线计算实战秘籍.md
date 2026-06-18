---
title: Spark调优——分布式离线计算实战秘籍
author: 柯基BigData
date: 柯基kj柯基kj
url: https://mp.weixin.qq.com/s?__biz=MzY4MjI3ODA4Ng==&mid=2247483823&idx=1&sn=da096fd9db4f55b87a0da96636dcf910&chksm=f2ef83a7a55fb1c4073ecd4612f2a79b4847cb31128f236dcd223ed259c64a1f59c54dc2e6fd&mpshare=1&scene=24&srcid=0528jmrwKgDGdhHsOgxtTUDj&sharer_shareinfo=d1e4c3df2caca74408a290e4cb0f37f0&sharer_shareinfo_first=d1e4c3df2caca74408a290e4cb0f37f0#rd
---

---

## 开头：为什么你的Spark跑得像蜗牛？

你有没有这种感觉：写Spark代码的时候，脑子里想的是“分分钟处理TB级数据”，实际跑起来却是“分分钟等出结果”？

五分钟？十分钟？一小时？对不起，您的任务已超时。

很多人遇到这种情况，第一反应是“集群不够大”。然后呢？申请更多资源。老板问为什么这么慢？答曰：“资源不够。”

**但真相是什么？**

**90%的Spark性能问题，都是代码写得像便秘，跟集群有什么关系？**

就像你开着一辆法拉利，非要在菜市场里龟速前行。不是车不行，是路和开法的问题。

Spark调优这件事，本质上就是让你的代码从“菜市场龟速”变成“高速公路狂飙”。

今天这篇文章，我不跟你讲什么JVM内存模型、GC原理那些教科书上的废话。我就用大白话告诉你： **Spark调优的核心是什么，怎么调，哪些坑别踩。**

看完这篇文章，你会明白为什么同样跑一个任务，别人5分钟搞定，你要50分钟。不是你代码逻辑不对，是你的“开法”有问题。

**准备好了吗？发车。**

---

## 第一章：内存管理——租房住还是住大别墅？

### 你的内存，到底在干什么？

在讲Spark内存管理之前，先问一个问题： **你知道Spark跑任务的时候，内存都花在哪儿了吗？**

很多人会说：“花在数据处理上啊！”

对，但不全对。

Spark的内存，就像你家的水费账单——你以为全花在喝水上了，实际上冲马桶、洗澡、洗菜各占一部分。你不搞清楚每项占比，永远不知道哪里在漏水。

Spark内存分两大块：

**执行内存（Execution Memory）**：处理数据时用的，比如Shuffle时的排序、聚合操作。就像你做饭时的灶台，得有地方放切好的菜。

**存储内存（Storage Memory）**：用来缓存数据的，比如.cache()、persist()操作。就像你家冰箱，用来存放还没做的食材。

还有一小部分是用户内存，用来存你自己的数据结构；和预留内存，给系统自己用。

### 打个比方你就懂了

想象你要办一场家宴。

**没调优的Spark**：就像你租了个十平米的小单间。客厅是客厅，厨房是厨房，卧室也是厨房。所有东西挤在一起，施展不开。客人来了没地方坐，菜做好了没地方放，整个一狼狈。

**调优后的Spark**：就像你直接搬进了三百平米的大别墅。客厅可以接待客人，厨房有足够的操作台，冰箱可以存放大批食材。想做什么菜做什么菜，从容不迫。

内存调优的核心，就是 **给不同用途分配合适的空间**。

### 实战怎么调？

核心参数就两个：

```
spark.memory.fraction = 0.6  
spark.memory.storageFraction = 0.5
```

**spark.memory.fraction**控制留给执行和存储的内存占总堆内存的比例，默认0.6。建议保持默认，除非你明确知道自己在干什么。

**spark.memory.storageFraction**控制存储内存占总内存的比例，默认0.5。这意味着执行和存储各占一半的可用内存。

但这里有个坑： **执行内存可以抢占存储内存，存储内存不能抢占执行内存**。

什么意思？

就像你家客厅——平时用来会客（存储），但来很多客人的话，你也可以把餐厅、甚至卧室临时改成会客区（执行）。但反过来不行——你不能让客人睡厨房。

所以如果你的任务经常需要Shuffle、排序、聚合，那就把存储比例调低一点，给执行多留点空间。如果你的任务大量使用.cache()，那就多留点给存储。

还有最重要的两个参数：spark.driver.memory 和 spark.executor.memory 。

这两个控制的是你实际拿到的“别墅大小”。

如果你的executor只有2G内存，那再怎么调参都是小打小闹。就像你租了个5平米的小隔间，讨论客厅占比60%还是70%有什么意义？

**所以调优第一步：先确认你的内存够不够用。不够的话，先加内存。**

---

## 第二章：并行度——多车道高速公路 vs 单车道乡村路

### 并行度是个什么鬼？

先说个扎心的真相： **你代码跑得慢，很可能是因为你只用了一只手在干活。**

Spark是一个分布式计算框架，核心思想是“分而治之”——把一个大任务拆成很多小任务，分配到不同的Executor上并行执行。

并行度（Parallelism），就是你同时干活的手的数量。

**parallelism = partition数量 = 任务并行度**

每个partition对应一个task，每个task跑在一个Executor的core上。

### 打个比方你就懂了

想象你要搬100箱行李。

**低并行度**：就你一个人，一箱一箱搬。搬完这箱再搬下一箱，累死累活干一天。

**高并行度**：你找了100个帮手，一人搬一箱。十分钟搞定，剩下的时间喝茶。

这就是并行度的威力。

但问题来了—— **是不是并行度越高越好？**

当然不是。

就像你找帮手，也要考虑成本。找100个人来搬10箱行李，光等人齐、分工、交接的时间就够你一个人搬完了。

**并行度太低**：资源浪费，任务跑得慢。

**并行度太高**：任务太碎，管理开销变大，反而变慢。

这就是所谓的“过犹不及”。

### 实战怎么调？

核心原则是： **让每个task处理的数据量在128MB到256MB之间。**

这叫“最佳数据块大小”，是Spark官方推荐值。

计算公式很简单：

```
partition数 = max(集群总core数, 数据总大小 / 128MB)
```

具体参数：

**spark.sql.shuffle.partitions**：控制Shuffle时的分区数，默认200。这个数字往往太高了，很多人跑个小任务，Shuffle出200个分区，task一堆，小文件一堆，性能垃圾。

建议改成：集群core数的2到4倍。

比如你集群有100个core，那就设成200到400。

**spark.default.parallelism**：RDD的默认分区数。如果没有特殊需求，可以设成集群总core数的2到3倍。

还有一个点： **数据源分区数也很重要**。

如果你的数据源分区数太少，比如Parquet文件只有10个分区，那就算你Spark参数设得再漂亮，也是巧妇难为无米之炊。

所以读取数据时，尽量让数据按照合理的粒度分区存储。

**一句话总结：并行度不是越高越好，让每个task干适量的活，才是王道。**

---

## 第三章：Shuffle——数据的搬家之旅

### 为什么Shuffle是性能杀手？

如果说Spark性能问题有一个万恶之源，那一定是 **Shuffle**。

什么是Shuffle？

简单说，就是 **数据的重新分区**。

当你的算子需要跨Executor访问数据时——比如groupBy、join、reduceByKey——数据必须重新分配到不同的节点上。这个重新分配的过程，就是Shuffle。

### 打个比方你就懂了

想象你要组织一场同学会。

**没有Shuffle**：所有人按城市住，你负责北京的同学，我负责上海的同学。各自整理各自名单就行，不用搬来搬去。

**有Shuffle**：现在要按“行业”分组。不管你住哪儿，只要你是金融行业的，就得挪到“金融组”这个房间来。

**这个挪动的过程，就是Shuffle。**

你需要把所有金融行业的人从各个城市找出来，然后统一送到指定地点。这中间涉及：打包、清点、搬运、签收、重新整理——每一个环节都耗时耗力。

**Shuffle的本质，就是数据的“搬家”。**

### Shuffle为什么慢？

三个原因：

**一、数据要落盘再拉取**

Shuffle时，数据先写入本地磁盘（相当于打包行李），然后网络传输到目标节点（相当于搬家公司运货），最后从磁盘读取到内存（相当于在新家拆包）。

这一来一回，速度能快吗？

**二、会产生大量小文件**

每个Executor写出一批文件，下一个stage的Executor去读取这些文件。小文件一多，磁盘IO和网络连接数都爆炸。

**三、容易引发数据倾斜**

如果你的数据分布不均匀，比如按用户ID分组，但90%的数据都是同一个用户——那这个用户的数据就会跑死，其他task却空着。

### 实战怎么调？

**参数一：spark.shuffle.file.buffer**

默认值32KB，控制shuffle write时写入磁盘的缓冲区大小。建议调到1MB。

```
spark.shuffle.file.buffer = 1mb
```

就像搬家时用大箱子还是小盒子——大箱子效率高。

**参数二：spark.shuffle.io.preferDirectBufs**

开启后使用堆外内存进行shuffle，减少GC压力。建议开启。

```
spark.shuffle.io.preferDirectBufs = true
```

**参数三：spark.shuffle.sort.bypassMergeThreshold**

当reducer数量小于这个阈值时，使用sort-based shuffle的bypass模式，跳过排序直接合并文件。默认值200，建议保持。

**最重要的优化：减少Shuffle次数**

很多新手写代码，动不动来个groupBy再join再groupBy，一个任务Shuffle四五次。

就像搬家的时候，箱子先从A楼搬到B楼，再从B楼搬到C楼

能用map、filter搞定的，别用reduceByKey。

能用广播变量broadcast解决的join，别用普通的join。

**Shuffle能少则少，这是性能优化的第一原则。**

---

## 第四章：数据倾斜——有人太闲，有人累死

### 什么是数据倾斜？

数据倾斜就是 **数据分布不均匀，导致任务执行时间差异巨大**。

大部分task都跑完了，个别task还在那儿磨蹭。整个job就卡在这几个慢task上。

### 打个比方你就懂了

想象你是一家奶茶店的老板，雇了十个员工负责收银。

**正常情况**：客人均匀分布到十个收银台，每个人干得差不多。

**数据倾斜**：九个收银台前各排2个人，一个收银台前排了500个人。那九个人干完活在那聊天打牌，就那一个收银台还在忙。结果呢？后面排队的人都等着，整个店效率被一个人拖死。

**数据倾斜的本质：少数节点承担了太多数据量，其他节点却闲着没事干。**

### 数据倾斜怎么破？

四个思路：

**思路一：增加随机前缀，打散数据**

适用场景：key的重复度高，但业务上可以加前缀。

比如join时数据倾斜，可以在key前面加个随机数，把一个key分成N份，这样每个key的数据就分散了。join完成后再去掉前缀。

```
# 原代码  
df1.join(df2,"key")  
  
# 优化代码：给key加随机前缀  
frompyspark.sql.functionsimportconcat,lit,rand  
  
df1_with_salt=df1.withColumn("salt",(rand(10)*n).cast("int"))  
df2_with_salt=df2.withColumn("salt",(rand(10)*n).cast("int"))  
  
df1_with_salt=df1_with_salt.withColumn("key_salted",concat("key",lit("_"),"salt"))  
df2_with_salt=df2_with_salt.withColumn("key_salted",concat("key",lit("_"),"salt"))  
  
result=df1_with_salt.join(df2_with_salt,"key_salted")
```

**思路二：扩大join的资源**

适用场景：内存瓶颈。

倾斜的那个key数据量太大，处理不动，那就给它多分配点内存。

```
spark.conf.set("spark.sql.shuffle.partitions",1000)# 增加分区  
spark.conf.set("spark.executor.memory","8g")# 增加内存
```

**思路三：过滤掉倾斜的key**

适用场景：倾斜的key没有业务价值。

比如日志分析时，某些系统内部ID占了99%的数据，但实际分析的是用户行为。那直接过滤掉这些垃圾数据。

```
df.filter(~df.key.isin(["SYS_INTERNAL","HEARTBEAT"]))
```

**思路四：使用salting + 两阶段聚合**

适用场景：groupBy时的数据倾斜。

```
# 第一阶段：局部聚合，加随机前缀  
df_with_salt=df.withColumn("salt",(rand(10)*10).cast("int"))  
grouped=df_with_salt.groupBy("key","salt").agg(count("*").alias("cnt"))  
local_agg=grouped.groupBy("key").agg(sum("cnt").alias("total_cnt"))  
  
# 第二阶段：全局聚合，去掉前缀  
result=local_agg.groupBy("key").agg(sum("total_cnt").alias("final_cnt"))
```

**一句话总结：数据倾斜的本质是分布不均，解决思路是分散、过滤、加资源。找到那个“排队500人的收银台”，对症下药。**

---

## 第五章：广播变量——共享资料柜的艺术

### 为什么要用广播变量？

先问一个问题： **你知道join的时候，数据是怎么传输的吗？**

普通join：小表的数据跟着大表一起Shuffle，每个Executor都收到完整的小表数据，然后在本地join。

广播join：将小表广播到每个Executor的内存里，所有task共享一份数据，不需要Shuffle。

### 打个比方你就懂了

**普通join**：就像每个部门都要用一份公司制度手册，结果公司派专员一家家送。送一次不够，每次开会都要送。效率低到令人发指。

**广播join**：就像把公司制度手册扫描一份，放到公司共享资料柜里。哪个部门要用，自己去取就行。一份资料，全公司共享。

**广播变量的本质：一处存放，多处共享。**

### 什么时候用广播变量？

**广播join的适用场景：**

* 小表join大表，小表可以完全放入内存
* 表的大小在几十MB到几百MB之间
* 小表是事实表，大表是维度表（星型模型）

**不建议使用广播的场景：**

* 表太大，放不进Executor内存
* 两张大表join
* 表频繁变更（广播后不会自动更新）

### 实战怎么用？

```
frompyspark.sql.functionsimportbroadcast  
  
# 广播小表  
result=large_df.join(broadcast(small_df),"key")
```

就这么简单，加个broadcast()就行。

**参数调优：**

```
spark.sql.autoBroadcastJoinThreshold=10485760# 默认10MB，可以调大  
spark.broadcast.blockSize=40960# 广播块大小，默认4MB
```

**一个血的教训：**

很多新手觉得广播变量好，就使劲用。结果把几个G的数据广播出去，Executor内存直接爆掉，任务直接挂。

**广播变量是用来分享小数据的，不是用来分享大数据的。记住，Executor的内存是有限的。**

---

## 第六章：算子优化——厨房做菜的艺术

### 算子选择决定性能

很多人写Spark代码，只管功能实现，不管算子选择。

结果呢？同一个需求，有人写得快如闪电，有人写得慢如蜗牛。

区别在哪？ **算子选择。**

### 打个比方你就懂了

想象你要做一桌家宴。

**写法一**：所有菜都从洗菜开始，炒一道吃一道，再洗菜再炒下一道。客人等得花都谢了。

**写法二**：先把所有菜洗好切好码在盘子里，需要什么直接下锅。行云流水，效率拉满。

**Spark算子优化，就是让你学会“备菜”。**

把能提前做的操作先做掉，把能合并的步骤合并掉，让真正需要Shuffle的环节来得更少、更高效。

### 具体怎么优化？

**技巧一：map端预聚合**

reduceByKey比groupByKey好，因为它会在map端先做一次本地聚合，减少Shuffle数据量。

```
# ❌ 不推荐  
rdd.groupByKey().mapValues(_.sum)  
  
# ✅ 推荐  
rdd.reduceByKey(_+_)
```

**技巧二：用filter替代where**

在数据源层面过滤，比读取全部数据再过滤要快得多。

```
# ❌ 不推荐：读取全部再过滤  
df=spark.read.parquet("path")  
df.filter(df.status=="active").show()  
  
# ✅ 推荐：读取时就过滤（支持谓词下推的数据源）  
df=spark.read.parquet("path",filter="status = 'active'")
```

**技巧三：避免使用自定义函数**

能用内置函数的，别自己写UDF。UDF无法享受Spark的优化器，效率差很多。

```
# ❌ 不推荐  
frompyspark.sql.functionsimportudf  
  
@udf  
defmy_func(x):  
returnx.upper()  
  
df.select(my_func("name"))  
  
# ✅ 推荐  
frompyspark.sql.functionsimportupper  
  
df.select(upper("name"))
```

**技巧四：复用同一个DataFrame**

多次使用同一个DataFrame？先.cache()一下。

```
df=spark.read.parquet("path")  
  
# 第一次使用  
count=df.count()  
  
# 第二次使用  
result=df.groupBy("category").count()  
  
# 如果不使用cache，Spark会重新读取两次数据
```

**技巧五：避免Spill到磁盘**

Spark在做聚合、排序时，如果内存不够，会Spill到磁盘。磁盘IO是性能杀手。

```
spark.sql.shuffle.partitions=200# 不要太少，分担内存压力  
spark.memory.fraction=0.6# 保证足够的执行内存
```

**一句话总结：算子优化的核心是“能合并就合并，能预计算就预计算，能用内置就不用自定义”。**

---

## 结尾：调优的本质是什么？

写到这里，我想问你一个问题： **你觉得Spark调优是什么？**

是改参数？改代码？还是加机器？

**都对，但都不本质。**

Spark调优的本质，是 **对计算资源的高效利用，和对数据流动的精细管控**。

你调配内存，是在决定“别墅里每个房间怎么用”。

你调整并行度，是在决定“多少人同时干活”。

你优化Shuffle，是在减少“不必要的搬家”。

你解决数据倾斜，是在平衡“工作量分配”。

你使用广播变量，是在追求“资源共享”。

你优化算子，是在提升“烹饪效率”。

**所有的调优，都是为了让有限的资源，发挥出最大的价值。**

就像过日子，不是房子越大越好，而是每个房间都用到刀刃上。

就像开车，不是马力越大越快，而是知道什么时候踩油门、什么时候减速、什么时候换挡。

**好的工程师，不是拥有最豪华的车队，而是能把一辆普通车开出超跑的感觉。**

Spark调优，就是让你成为这样的工程师。

---

## 附录：核心参数速查表

| 参数 | 默认值 | 推荐值 | 说明 |
| --- | --- | --- | --- |
| spark.driver.memory | 1g | 看数据量 | Driver内存 |
| spark.executor.memory | 1g | 4g-8g | Executor内存 |
| spark.executor.cores | 1 | 2-4 | 每个Executor的CPU核数 |
| spark.sql.shuffle.partitions | 200 | core数×2-4 | Shuffle分区数 |
| spark.default.parallelism | —— | core数×2-3 | RDD默认分区数 |
| spark.memory.fraction | 0.6 | 0.6-0.7 | 执行+存储内存占比 |
| spark.memory.storageFraction | 0.5 | 0.3-0.5 | 存储内存占比 |
| spark.shuffle.file.buffer | 32kb | 1mb | Shuffle缓冲区大小 |
| spark.sql.autoBroadcastJoinThreshold | 10mb | 100mb | 广播join阈值 |

**记住：参数只是工具，思维才是核心。**

祝你调优愉快，任务飞起来。

---

**你用过MySQL吗？遇到过什么坑？****欢迎评论区留言，我们聊聊。**

**如果觉得有收获，"点赞"和"在看"是对我最大的支持。**

---

*本文作者：一个不说废话只讲干货的技术人。*

*关注我，带你用最通俗的方式，搞懂最硬核的技术。*