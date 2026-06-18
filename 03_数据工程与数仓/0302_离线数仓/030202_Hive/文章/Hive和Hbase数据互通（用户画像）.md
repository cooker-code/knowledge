---
title: Hive和Hbase数据互通（用户画像）
author: 浪尖聊大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzUyMDA4OTY3MQ==&mid=2247510948&idx=1&sn=43abe642eb0ede67db24707e04c47644&chksm=f9ed528cce9adb9a1ea4091bdf681fdae9e878ee93158a3e61e01eff7e640ff0abbef624f819&mpshare=1&scene=24&srcid=0315utRYitFI3xco0fmNLur8&sharer_shareinfo=19f9886768e6983fe283d5db63260507&sharer_shareinfo_first=19f9886768e6983fe283d5db63260507#rd
---

## 背景

依旧是用户画像的项目，现在标签化的数据存放在hive中，而查询是要在hbase上进行查询，所以需要将hive的数据导入hbase中。

方案：

1、hive和hbase的表建立映射关系，读取的是同一份HDFS文件，只是在上层建立hbase到hive表的映射。

优点：一份数据存储，两种查询模式，数据存储最低；

缺点：底层还是格式化的HDFS文件，查询需要进行映射转换，效率较低；

2、将hive的数据通过生成hfile，通过bulkload导入到hbase，这样底层数据的格式会转变成Hfile存储在hbase中，将hbase完全作为一个数据库去查询

优点：查询效率高；

缺点：同一份数据，两份存储格式，空间换取时间；

## 介绍

1、环境问题

之前因为各种操作，导致hive的对应的数据存储路径被删了，所以先对hive的环境进行重新配置，主要配置和mysql的互通；

```
1、删除mysql对应的hive库；  
2、执行schematool -dbType mysql -initSchema  
3、重启hive  
4、查看hive-site的配置  
  <property>  
    <name>hive.metastore.warehouse.dir</name>  
    <value>/user/hive/warehouse</value>  
    <description>location of default database for the warehouse</description>  
  </property>
```

2、spark运行环境的配置

在测试的时候，spark的运行环境出现了很多问题，主要是jar包冲突和找不到类的问题。

所以基于hbase的类主要是：

```
        <dependency>  
            <groupId>org.apache.hbase</groupId>  
            <artifactId>hbase-client</artifactId>  
            <version>1.1.2</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.apache.hbase</groupId>  
            <artifactId>hbase-protocol</artifactId>  
            <version>1.1.2</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.apache.hbase</groupId>  
            <artifactId>hbase-common</artifactId>  
            <version>1.1.2</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.apache.hbase</groupId>  
            <artifactId>hbase-server</artifactId>  
            <version>1.1.2</version>  
        </dependency>
```

同时spark的代码框架中要加入resouces包，并将hive-site.xml、core-site.xml、hdfs-site.xml、hbase.xml配置文件扔进去，方便spark运行是能够找到依赖的环境。

3、hive映射hbase的表。

在[Spark读写Hbase（用户画像）](http://mp.weixin.qq.com/s?__biz=MzUyMDA4OTY3MQ==&mid=2247498588&idx=2&sn=cb0bd22364a1a6378763be4493484a39&chksm=f9ed0274ce9a8b6228491a6e126fc08281caed385843e6d87e42762815e58597f9d77825d5e9&scene=21#wechat_redirect)将如何像hbase写数据方式介绍了，而且在hbase中建立了一张表:TEST.USER\_INFO

现在将这张吧映射到hive中：

建立hive映射表：

```
CREATE EXTERNAL TABLE IF NOT EXISTS test_user_info  
(  
key string,  
C1 string,  
C2 string,  
C3 string  
)  
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'  
with serdeproperties ("hbase.columns.mapping" = "  
:key,  
INFO:C1,  
INFO:C2,  
INFO:C3  
")  
tblproperties("hbase.table.name" = "TEST.USER_INFO");
```

* stored by指定数据的存储方式。
* SERDEPROPERTIES：表示字段映射，对应hive中的表字段的顺序，需要注意的是 :key指的是Hbase中的rowdy，hive表中要有一个key字段与之对应，否则会报错的。
* TBLPROPERTIES:表示表名映射，指定需要映射的Hbase表名。

具体的映射规则：

* hbase中的空cell在hive中会补null。
* hive和hbase中不匹配的字段会补null。
* hive内部表的数据，由hive自己管理，因此删除hive表，则对应的Hbase表也会被删除。
* hbase对应的hive没有时间戳概念，默认返回最新版本的值。
* 由于HBase中没有数据类型信息，所以在存储数据的时候都转化为String类型。
* 建表如果没有指定:key，则第一列默认为行健。

建表语句：

查询结果：

在hbase中新增只有两个列的rowKey。

查询结果：

可以看到在不匹配的列中会自动补NULL。

4、整个hbase的map映射到hive

规则和上面基本一样，只不过建立hive表的时候指定的列的类型修改一下。

```
CREATE EXTERNAL TABLE test_user_info_2  
(key string,  
value map<string,string>)  
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'  
WITH SERDEPROPERTIES (  
"hbase.columns.mapping" = ":key,INFO:"  
)  
tblproperties("hbase.table.name" = "TEST.USER_INFO");
```

查询结果：

5、spark生成hive表数据

```
val RDD = spark.sparkContext.textFile("hdfs://localhost:9000/data/user/*")  
    import spark.implicits._  
    val DF = RDD.map(f => (f.split(",")(0),  
       f.split(",")(1),  
       f.split(",")(2),  
       f.split(",")(3),  
       f.split(",")(4),  
       f.split(",")(5),  
       f.split(",")(6),  
       f.split(",")(7))).  
       toDF("uid","date_create","create_type","level","follow_num","first_follow_time","last_follow_time","follow_dur")  
    RDD.foreach(println)  
    DF.write.mode("overwrite").insertInto("default.user_message_1")  
    //todo 查询hive表数据  
   spark.sql("select * from default.user_message_1").show
```

在hive上建立hbase的映射表：

```
CREATE TABLE user_message  
(  
uid string,  
date_create string,  
create_type int,  
level string,   
follow_num int,   
first_follow_time string,   
last_follow_time string,    
follow_dur bigint  
)  
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'  
with serdeproperties ("hbase.columns.mapping" = "  
:key,  
user_info:date_create,  
user_info:create_type,  
user_info:level,  
follow_info:follow_num,  
follow_info:first_follow_time,  
follow_info:last_follow_time,  
follow_info:follow_dur  
")  
tblproperties("hbase.table.name" = "default:TEST.user_message","hbase.mapred.output.outputtable" = "default:TEST.user_message");
```

查看hbase

将hive中user\_message\_1中的数据导入user\_message中

```
insert overwrite table user_message select * from user_message_1;
```

hive中查询结果：

Hbase中查询结果：

这样两边的数据映射成功。

6、查询hive数据写入Hbase

```
package sparkTest  
import org.apache.hadoop.fs.{FileSystem, Path}  
import org.apache.hadoop.hbase.client.ConnectionFactory  
import org.apache.hadoop.hbase.{HBaseConfiguration, TableName}  
import org.apache.hadoop.hbase.io.ImmutableBytesWritable  
import org.apache.hadoop.hbase.mapreduce.TableInputFormat  
import org.apache.log4j.{Level, Logger}  
import org.apache.spark.sql.SparkSession  
import org.apache.hadoop.conf.Configuration  
import org.apache.hadoop.hbase._  
import org.apache.hadoop.hbase.client._  
import org.apache.hadoop.hbase.mapred.TableOutputFormat  
import org.apache.hadoop.hbase.util.Bytes  
import org.apache.hadoop.mapred.JobConf  
/** *  
  *  
  * @autor gaowei  
  * @Date 2020-08-06 09:53   
  */  
object HfiletoHbase {  
  def main(args: Array[String]): Unit = {  
    Logger.getLogger("org.apache.spark").setLevel(Level.ERROR)  
    val spark = SparkSession  
       .builder()  
       .appName("HfiletoHbase")  
       .enableHiveSupport()  
       .config("spark.master", "local")  
       .getOrCreate()  
    val sc = spark.sparkContext  
    val hiveContext = new org.apache.spark.sql.hive.HiveContext(sc)  
    hiveContext.sql("SET hive.exec.dynamic.partition = true")  
    hiveContext.sql("SET hive.exec.dynamic.partition.mode = nonstrict ")  
    hiveContext.sql("SET mapreduce.input.fileinputformat.input.dir.recursive = true")  
    hiveContext.sql("SET hive.input.dir.recursive = true")  
    hiveContext.sql("SET hive.mapred.supports.subdirectories = true")  
    hiveContext.sql("SET hive.supports.subdirectories = true")  
  
    val tablename = "TEST.user_message_test_c"  
    val conf = HBaseConfiguration.create()  
    //设置zooKeeper集群地址，也可以通过将hbase-site.xml导入classpath，但是建议在程序里这样设置  
    conf.set("hbase.zookeeper.quorum","localhost")  
    //设置zookeeper连接端口，默认2181  
    conf.set("hbase.zookeeper.property.clientPort", "2181")  
    creteHTable(tablename, conf)  
    conf.set(TableInputFormat.INPUT_TABLE, tablename)  
  
    val DF = spark.sql(  
      s"""  
        |select uid,  
        |date_create,  
        |create_type,  
        |level,  
        |ifnull(follow_num,0) as follow_num,  
        |first_follow_time,  
        |last_follow_time,  
        |ifnull(follow_dur,0) as  follow_dur  
        |from user_message_1  
      """.stripMargin)  
    val RDD = DF.rdd.map(f => (f.getAs[String]("uid"),  
       f.getAs[String]("date_create"),  
       f.getAs[Int]("create_type").toString,  
       f.getAs[String]("level"),  
       f.getAs[Int]("follow_num").toString,  
       f.getAs[String]("first_follow_time"),  
       f.getAs[String]("last_follow_time"),  
       f.getAs[Long]("follow_dur").toString))  
    for(arr <- RDD.collect()){println(arr)}  
  
    val jobConf = new JobConf()  
    jobConf.setOutputFormat(classOf[TableOutputFormat])  
    jobConf.set(TableOutputFormat.OUTPUT_TABLE,tablename)  
    RDD.map{f => {  
    val put = new Put(Bytes.toBytes(f._1))  
    put.add(Bytes.toBytes("user_info"),Bytes.toBytes("date_create"),Bytes.toBytes(f._2))  
    put.add(Bytes.toBytes("user_info"),Bytes.toBytes("create_type"),Bytes.toBytes(f._3))  
    put.add(Bytes.toBytes("user_info"),Bytes.toBytes("level"),Bytes.toBytes(f._4))  
    put.add(Bytes.toBytes("follow_info"),Bytes.toBytes("follow_num"),Bytes.toBytes(f._5))  
    put.add(Bytes.toBytes("follow_info"),Bytes.toBytes("first_follow_time"),Bytes.toBytes(f._6))  
    put.add(Bytes.toBytes("follow_info"),Bytes.toBytes("last_follow_time"),Bytes.toBytes(f._7))  
    put.add(Bytes.toBytes("follow_info"),Bytes.toBytes("follow_dur"),Bytes.toBytes(f._8))  
    (new ImmutableBytesWritable,put)  
  }}.saveAsHadoopDataset(jobConf)  
  
  sc.stop()  
}  
  
  def creteHTable(tableName: String, hBaseConf : Configuration) = {  
    val connection = ConnectionFactory.createConnection(hBaseConf)  
    val hBaseTableName = TableName.valueOf(tableName)  
    val admin = connection.getAdmin  
    if (!admin.tableExists(hBaseTableName)) {  
      val tableDesc = new HTableDescriptor(hBaseTableName)  
      tableDesc.addFamily(new HColumnDescriptor("user_info".getBytes))  
      tableDesc.addFamily(new HColumnDescriptor("follow_info".getBytes))  
      admin.createTable(tableDesc)  
    }  
    connection.close()  
  }  
}
```

结果：