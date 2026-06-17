---
title: Flink CDC 导入表数据过亿，咋个整？
author: 安瑞哥是码农
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI0OTEwNzQyNA==&mid=2247489713&idx=1&sn=7b017f6d51a7d3091ad814bb5718b9dd&chksm=e8857d20449c08f30e032700145e70e480b5212c7aa528afb60996bbef5ecda65351953cbd93&mpshare=1&scene=24&srcid=06101SJwuUDt9NvAp4iMlWWl&sharer_shareinfo=b62f78242d7d9485ca4917507a7165ff&sharer_shareinfo_first=b62f78242d7d9485ca4917507a7165ff#rd
---

前段时间刚搞定「Doris 整库同步」以及相关的，各种形形色色乱八七糟的问题，这一次，又遇到新的的了。

具体什么问题呢？

Flink CDC 导数据时，如果上游 MySQL 或者 PG(postgresql) 库里面，出现了部分数据量非常大的表，比如动不动几亿、十几亿的数据量吓死你。

这个时候，如果还沿用老思维，想通过单个 Flink CDC 任务来同步上游整库实现「一步到位」，不是不可以，只不过那耗费的内存，怕是有点多。

最近就有同学遇到了这么个问题，在做整库同步时，上游库里面就有若干张数据量过亿(历史数据)的表，关键这个表字段里面，还存在个别内容特别大的 json，CDC 程序跑起来那叫一个酸爽。

想一次搞定，有同学已经这么玩过了，放张图，让你感受一下：

硬上的话，500G 的内存，一个 CDC 任务就能给你干掉 490，就问你服不服。

最关键的是，一次导这么大的数据量，即便占了这么大的内存，过程能不能顺利，还得打个大大的问号，很可能出现导到一部分时，程序就因为各种莫名其妙的原因嗝屁了。

那针对这种比较「变态」的场景，我们可以怎么优雅的来解决呢？

根据我的经验，可以这么来干。

0. 可行的解决方案

像这种库里出现「个别超级大表」的情况，比较合理的玩法，就是根据数据量大小，把这些表分为两大类。

一类：归为一般的中小表，比如千万规模及以下，这种可以利用之前说过的「Flink CDC 整库同步」方案；

另一类：归为超级大表，比如数据量过亿，这部分把它单独拎出来，作为特殊的表，用特殊的方式进行处理。

好，战略层面咱就这么来定，接下来看具体的实现。

1. 怎么筛选所有中小表

既然要归为两类，那么就要先确定，在沿用「Flink CDC 整库同步」方案时，怎么把那些个「超级大表」给排除掉。

在读取上游数据库的 API 里，添加 「tableList」函数

这个方法的优点在于，可以支持「正则表达式」，

所以，我们完全可以通过它，来把个别「超级大表」给排除掉。

比如像下面这样(具体玩法可以参考之前的文章)：

```
.hostname("192.168.xxx.xxx")  
.port(5432)  
.database("db_name")//库名     
.tableList("^(?!.*schema_name\\.big_table_name).*")
```

这样一来，就把大表「schema\_name.big\_table\_name」给排掉了(可以写多个)。

此时你 API 读到的，就是这个库里面所有的「中小表」了。

2. 怎么解决超级大表

根据以往的经验， 想要以一种平稳、优雅、且不耗费巨大内存的情况下导出这种大表，最优的方案就是「分而治之」。

也就把历史数据拆成多个小份，每次导出 1 份，分多次把它给「蚕食」掉，而对于后续进来的实时数据，再启用原来的 CDC 方案给接上。

具体玩法是利用 Flink 的 JDBC API，是的，虽然它看起来比较笨拙，而且也不「高级」，但关键时刻，它却能解决棘手的问题。

根据官网给的案例，在读上游表的时候，可以这么来干(以读取 PG 为例)：

```
package cn.pcl.csrc.cdc.postgresql  
  
import org.apache.flink.api.common.eventtime.WatermarkStrategy  
import org.apache.flink.api.common.typeinfo.{TypeInformation}  
import org.apache.flink.connector.jdbc.source.reader.extractor.ResultExtractor  
import org.apache.flink.connector.jdbc.source.{JdbcSource}  
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment  
import java.sql.ResultSet  
  
object FlinkReadPGWithJDBC {  
  
    private case class User(id: Int, name: String)  
  
    def main(args: Array[String]): Unit = {  
        val env = StreamExecutionEnvironment.getExecutionEnvironment  
  
        import org.apache.flink.streaming.api.scala._  //引入隐式转换函数  
  
        val jdbcSource = JdbcSource.builder[User]()  
                                .setDBUrl("jdbc:postgresql://192.168.***.***:5432/test_db")  
                                .setUsername("postgres")  
                                .setPassword("***")  
                                .setSql("select * from public.big_table where id >=0 and id < 10000000") //根据条件把数据切成多份  
                                .setResultExtractor(newResultExtractor[User]() {  
                                    overridedef extract(resultSet: ResultSet): User = {  
                                        val id = resultSet.getInt("id")  
                                        val name = resultSet.getString("name")  
                                        User(id, name)  
                                    }  
                                })  
                                .setDriverName("org.postgresql.Driver")  
                                .setTypeInformation(TypeInformation.of(classOf[User]))  
                                .build()  
  
        val rawDS = env.fromSource(jdbcSource, WatermarkStrategy.noWatermarks(), "PGSource")  
  
        rawDS.addSink(...)  
  
        env.execute("Read_PG_With_JDBC")  
    }  
}
```

这个是根据 Flink 官方文档调试出来的(官方文档一如既往的有坑，这事咱就不提了)。

先说这里面必要的设置，有下面 3 个：

第 1 个：必须要设置查询的 SQL，这个的目的，就是用来切分数据的，最好选一个索引字段，这样切数据的效率就会高很多，至于切成多少份、怎么来切、需要根据你这个表的数据分布特点，以及单个任务能承担的硬件资源来定；

第 2 个：必须要定义数据的「提取方式」，也就是「setResultExtractor」，因为是 JDBC 的玩法，就需要对 ResultSet 的结果进行必要字段的提取；

第 3 个：必须要定义一个 pojo 类，来承载上一步提取后的数据，使其成为一个对象；

当然，还有其他关于数据源端的设置，比如 url、用户名、密码之类的，这里就不废话了。

（PS：官网还提到了可以用基于 JDBC 的流方式读取，但我试了，感觉完全是在扯淡）

最后

这个方案可能对于一些同学来说，太麻烦，没有那种「一键式」的高大上，但你要知道，咱们讨论的，可是实打实的，面对生产上那种复杂场景的解决方案。

以我的经验，就几乎没见过哪个真实大数据量场景下的数据导出，可以那么的简单和理想化，那些宣称 so easy 的方案，无一例外全是 demo。

真刀真枪的解决实际问题，向来都不太容易，你觉得呢？

---

你可以添加我的私人微信入讨论群，跟一群热爱技术的小伙伴一起成长...