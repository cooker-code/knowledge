---
title: Spark和Spring整合处理离线数据
author: 智海观潮
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484178&idx=1&sn=2db8f723b841eb4ece49604cebea1c96&chksm=e976ff28de01763e03d296633739ff3b3f9155507ca27f476d413e7856138e53c28c9bd0ea64&mpshare=1&scene=24&srcid=0613FA9CiJoadsYvANkAUKjy&sharer_shareinfo=01e36b4cff0fd660f6cf087deb4b55ad&sharer_shareinfo_first=01e36b4cff0fd660f6cf087deb4b55ad#rd
---

如果你比较熟悉JavaWeb应用开发，那么对Spring框架一定不陌生，并且JavaWeb通常是基于SSM搭起的架构，主要用Java语言开发。但是开发Spark程序，Scala语言往往必不可少。

众所周知，Scala如同Java一样，都是运行在JVM上的，所以它具有很多Java语言的特性，同时作为函数式编程语言，又具有自己独特的特性，实际应用中除了要结合业务场景，还要对Scala语言的特性有深入了解。

如果想像使用Java语言一样，使用Scala来利用Spring框架特性、并结合Spark来处理离线数据，应该怎么做呢？

本篇文章，通过详细的示例代码，介绍上述场景的具体实现，大家如果有类似需求，可以根据实际情况做调整。

**1.定义一个程序启动入口**

---

```
object Bootstrap {  private val log = LoggerFactory.getLogger(Bootstrap.getClass)  
  //指定配置文件如log4j的路径  val ConfFileName = "conf"  val ConfigurePath = new File("").getAbsolutePath.substring(0, if (new File("").getAbsolutePath.lastIndexOf("lib") == -1) 0  else new File("").getAbsolutePath.lastIndexOf("lib")) + this.ConfFileName + File.separator  
  //存放实现了StatsTask的离线程序处理的类  private val TASK_MAP = Map("WordCount" -> classOf[WordCount])  
  def main(args: Array[String]): Unit = {    //传入一些参数，比如要运行的离线处理程序类名、处理哪些时间的数据    if (args.length < 1) {      log.warn("args 参数异常！！！" + args.toBuffer)      System.exit(1)    }    init(args)  }  
  def init(args: Array[String]) {    try {      SpringUtils.init(Array[String]("applicationContext.xml"))      initLog4j()  
      val className = args(0)      // 实例化离线处理类      val task = SpringUtils.getBean(TASK_MAP(className))  
      args.length match {        case 3 =>          // 处理一段时间的每天离线数据          val dtStart = DateTimeFormat.forPattern("yyyy-MM-dd").parseDateTime(args(1))          val dtEnd = DateTimeFormat.forPattern("yyyy-MM-dd").parseDateTime(args(2))          val days = Days.daysBetween(dtStart, dtEnd).getDays + 1          for (i <- 0 until days) {            val etime = dtStart.plusDays(i).toString("yyyy-MM-dd")            task.runTask(etime)  
            log.info(s"JOB --> $className 已成功处理: $etime 的数据")          }  
        case 2 =>          // 处理指定的某天离线数据          val etime = DateTimeFormat.forPattern("yyyy-MM-dd").parseDateTime(args(1)).toString("yyyy-MM-dd")          task.runTask(etime)          log.info(s"JOB --> $className 已成功处理: $etime 的数据")  
        case 1 =>          // 处理前一天离线数据          val etime = DateTime.now().minusDays(1).toString("yyyy-MM-dd")          task.runTask(etime)          log.info(s"JOB --> $className 已成功处理: $etime 的数据")  
        case _ => println("执行失败 args参数:" + args.toBuffer)      }    } catch {      case e: Exception =>        println("执行失败 args参数:" + args.toBuffer)        e.printStackTrace()    }  
    // 初始化log4j    def initLog4j() {      val fileName = ConfigurePath + "log4j.properties"      if (new File(fileName).exists) {        PropertyConfigurator.configure(fileName)        log.info("日志log4j已经启动")      }    }  }}
```

**2.加载Spring配置文件工具类**

---

```
object SpringUtils {  private var context: ClassPathXmlApplicationContext = _  
  def getBean(name: String): Any = context.getBean(name)  
  def getBean[T](name: String, classObj: Class[T]): T = context.getBean(name, classObj)  
  def getBean[T](_class: Class[T]): T = context.getBean(_class)  
  def init(springXml: Array[String]): Unit = {    if (springXml == null || springXml.isEmpty) {      try        throw new Exception("springXml 不可为空")      catch {        case e: Exception => e.printStackTrace()      }    }    context = new ClassPathXmlApplicationContext(springXml(0))    context.start()  }  
}
```

**3.Spring配置文件applicationContext.xml**

---

```
<?xml version="1.0" encoding="UTF-8"?><beans xmlns="http://www.springframework.org/schema/beans"       xmlns:context="http://www.springframework.org/schema/context"       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd          http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context-4.0.xsd">  
    <!-- 配置包扫描 -->    <context:component-scan base-package="com.bigdata.stats"/>  
</beans>
```

**4.定义一个trait，作为离线程序的公共"父类"**

---

```
trait StatsTask extends Serializable {  //"子类"继承StatsTask重写该方法实现自己的业务处理逻辑   def runTask(etime: String)}
```

**5.继承StatsTask的离线处理类**

---

```
//不要忘记添加 @Component ，否则无法利用Spring对WordCount进行实例化@Componentclass WordCount extends StatsTask {  
  override def runTask(etime: String): Unit = {    val sparkSession = SparkSession      .builder()      .appName("test")      .master("local[*]")      .getOrCreate()  
    import sparkSession.implicits._  
    val words = sparkSession.read.textFile("/Users/BigData/Documents/data/wordcount.txt").flatMap(_.split(" "))      .toDF("word")  
    words.createOrReplaceTempView("wordcount")  
    val df = sparkSession.sql("select word, count(*) count from wordcount group by word")  
    df.show()  }}
```

推荐文章：

[Spark流式状态管理](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484166&idx=1&sn=11674d789da9c574ef2517261ff30644&chksm=e976ff3cde01762ae518afbbeb5418e3331ecdbf93b2b928abc5275018a477199ac0a8d61a53&scene=21#wechat_redirect)

[Spark实现推荐系统中的相似度算法](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484120&idx=1&sn=86f53bf31f4779afcda9a5dbf829a168&chksm=e976fee2de0177f409516ba6902192ef2671cb84e861827047a71ae3c4de80ab9ac2b774ee89&scene=21#wechat_redirect)

[Scala中的IO操作及ArrayBuffer线程安全问题](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484118&idx=1&sn=05e944ac401404860d55d865cb17ddaf&chksm=e976feecde0177fa53b8af1d6d661adafd4014d4e878ee2fdccf518c4e94e1d6a278e8f5a3bb&scene=21#wechat_redirect)

[学好Spark必须要掌握的Scala技术点](http://mp.weixin.qq.com/s?__biz=MzI0Mjc0MDU2NQ==&mid=2247484067&idx=1&sn=8b60a7c9e5a0f4e24a9945b450ad3e8e&chksm=e976fe99de01778f492f8ea3fb79c495f966422c0684caa2831e10046490569bb3d8159b0e3d&scene=21#wechat_redirect)