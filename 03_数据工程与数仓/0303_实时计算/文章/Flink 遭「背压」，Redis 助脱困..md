---
title: Flink 遭「背压」，Redis 助脱困.
author: 安瑞哥是码农
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI0OTEwNzQyNA==&mid=2247489252&idx=1&sn=c6b04310da0664c7bc1c6e8ea5f92d6e&chksm=e8b50d633b265066c04633a3dea2cb2f4229d683cc292bd7851a3fce4c6858b85d0bb4e3bea2&mpshare=1&scene=24&srcid=01147YLXz2v7A6nOYGF9xlp1&sharer_shareinfo=00ec43b7b373fd4c83c698040a687072&sharer_shareinfo_first=00ec43b7b373fd4c83c698040a687072#rd
---

上上篇文章写了解决 Flink 背压后的第一个办法——关闭 checkpoint，结果被评论区一个同学给 diss 了，说这算哪门子解决方案。

我是这么看待这件事的，对于一个问题的出现，尤其是比较复杂的问题，我们的应对策略，一般都可以分为两类：一类是用于临时应急；另一类，则是用来永久性解决问题。

而上次采用的关闭 Flink 的 checkpoint 功能，很明显属于前者，快速且有效；而对于后者的永久性解决方案，经过我的实际验证，确实可以比较完美的解决这个背压问题，但，这个方案执行起来，怎么着也得个 1-2 天时间。

技术架构以及编码的重新调整只是一方面，另一个比较麻烦的，就是需要跟团队其他小伙伴们沟通，晓之以理，动之以情地说服他们也要做代码上的改动，否则，容易挨揍。

同样，这第 2 个解决方案，经过了我超过 80 个小时的实践验证之后，不仅能彻底解决问题，而且，效率要比第 1 个应急方案更让人满意。

0. 优化方案确定

之前虽然分析过，这个 Flink 发生背压时，程序需要的内存 CPU 资源充足、以及系统负载、数据库负载、各种 IO 指标都没有出现瓶颈。

但它为什么还是跑得越来越慢？

核心原因在于，每一条流过来的 Kafka 数据，都需要去查一次 clickhouse 表，且因为每次的查询条件都不同，没有办法利用到数据库缓存，并且因为部分查询还是模糊匹配，所以导致查询速度上不去，逐渐就拖慢了程序下游的效率。

根据我以往解决这个问题的经验，像这种高速碰撞数据，只要被碰撞的数据量不是特别大(比如数亿级以上规模)，用一个单节点的 Redis，是能轻松搞定的。

而针对我当前的被碰撞数据量来说，也就 30w 左右，虽然后续会一直小幅度增加，但这个量级也完全能 hold 住。

把碰撞数据库优化为 Redis 之后，这个任务的数据处理逻辑，就从之前的这样：

变成了这样：

1. 方案执行 

方案一旦确定，接下来就是执行。

第 1 步：就是把之前存在 clickhouse 表的被碰撞数据，给迁移到 Redis，具体做法，上一篇文章已经扒开揉碎讲得很清楚了，[Spark 写 Redis，没辣么简单.](https://mp.weixin.qq.com/s?__biz=MzI0OTEwNzQyNA==&mid=2247489233&idx=1&sn=834b4df991d145d6fd31086d1897b65e&scene=21#wechat_redirect) 

那么第 2 步，就是改代码的核心逻辑：

主要变化在于，在 Flink 的 sink 富函数内部，将之前查询 clickhouse 的逻辑，给改为查询 Redis，其他部分不变。

核心变化代码如下：

```
addSink(new RichSinkFunction[(String, String)] {  
                /**在Redis进行碰撞查询*/  
                privateval redisHost = "${redis_ip}"  
                privateval port = 6379  
                privateval redisPasswd = "****"  
                privatevar jedis:Jedis = null  
  
                /**碰撞结果入库表*/  
                privateval sinkTable = "sink_db.sink_table"  
                privateval url = "jdbc:clickhouse://ck_ip:ck_port"  
                privateval ckPasswd = "****"  
                privatevar dataSource: ClickHouseDataSource = null  
                privatevar conn: Connection = null  
  
                overridedef open(parameters: Configuration): Unit = {  
                    /**Redis配置*/  
                    jedis = newJedis(redisHost, port)  
                    jedis.auth(redisPasswd)  
                    jedis.select(6)  
                    /**CK配置*/  
                    Class.forName("com.clickhouse.jdbc.ClickHouseDriver")  
                    dataSource = newClickHouseDataSource(url)  
                    conn = dataSource.getConnection("default", ckPasswd)  
                }  
  
                overridedef invoke(value: (String, String), context: SinkFunction.Context): Unit = {  
                  val domain = getDomain(value)  
                  val eventTime = getTime(value)  
  
                  /** 先查Redis */  
                  val domainArray = domain.split("\\.")  
                  if (domainArray.length >= 2){  
                      val subDomain = domainArray(domainArray.length - 2) + "." + domainArray(domainArray.length - 1)  
                      if(jedis.exists(subDomain)){/**说明碰撞成功*/  
                          val validTime = jedis.get(subDomain)  
  
                          /** 再写 CK*/  
                          if (eventTime >= validTime) {                          
                          /** 根据client IP 获取地理位置信息 */  
                            val region = getRegion(clientIP)  
                            val country = if (region.getCountry == null) ""else region.getCountry  
                            val province = if (region.getProvince == null) ""else region.getProvince  
                            val city = if (region.getCity == null) ""else region.getCity  
                           
                            val insertSQL = s"insert into $sinkTable (client_ip,country,province,city,domain,query_time) values (?,?,?,?,?,?)"  
  
                            try {  
                                val ps = conn.prepareStatement(insertSQL)  
                                ps.setString(1, clientIP)  
                                ps.setString(2, country)  
                                ps.setString(3, province)  
                                ps.setString(4, city)  
                                ps.setString(5, domain)  
                                ps.setString(6, eventTime)  
  
                                ps.addBatch()  
                                ps.executeBatch()  
                            } catch {  
                                case e:Exception => println("Fuck，出事了...",e)  
                            }  
                         }  
                      }  
                  }  
                }  
  
                overridedef close(): Unit = {}  
            }).name("Sink to CK")
```

代码实现非常的朴素简单，不需要连接池、不需要连接池、不需要连接池！

2. 执行效果

这个 UI 界面，是优化之前，跑 10 个小时后的样子，虽然这个时候碰撞结果入库延迟还不明显，但任务执行，已经开始累得有点喘气了：

而这个 Flink UI 界面，是用 Redis 优化后，也是执行到相同的时间点，一毛一样的资源、并行度、以及数据流量情况下的运行情况：

就问你对比明显不？

当然，当前任务，肯定是开启了 checkpoint 的。

而后台日志，再也没有出现过 checkpoint 的告警。

最关键的是，目前已经运行了超过 80 小时，这个优化后的 Flink 任务负载，还一如当初的矫健身姿，应对自如：

再来瞅一眼 Redis 的情况。

先看一眼它的负载：

前后几乎没有变化，即便当前程序对 Redis 的查询频率，大概为 500次/秒，但也就挠痒痒的程度。

再看一眼它的连接情况。

碰撞前为 5 个：

碰撞后为 13 个：

增加的 8 个，正好是这个 Flink 任务的并行度，对于 Redis来说，也不过毛毛雨而已。

最后

关于这个 Flink 碰撞任务的背压问题，到目前为止，算是彻底解决了(如果你担心 Redis 的稳定性，大可不必)。

事实证明，同样的查询条件，一般数据库的 IO 效率，的确干不过内存的 IO 效率，毕竟它俩就不是一个数量级的。

对于当前这个场景来说，优化之后，8 个并行度确实有点浪费了，还可以进一步压缩计算资源，你猜几个就够了呢？

下一次，咱再改用 Spark structured streaming，看相同的条件下，会不会比 Flink 表现得更优越。

---

你可以添加我的私人微信入讨论群，跟一群热爱技术的小伙伴一起成长...