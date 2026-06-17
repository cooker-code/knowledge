---
title: Doris 实践 | Apache Doris Spark-Connecter实战
author: HBase技术社区
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU5OTQ1MDEzMA==&mid=2247490921&idx=1&sn=fd629010004e6af0382d404d0f56bfe0&chksm=feb5ea14c9c26302ffec81665741a8016b7a1c8e62b7f835525dcca4ec1eaf029ea3bde1ca0c&mpshare=1&scene=24&srcid=06242LGfdx3yhqlTVJhRwFYk&sharer_sharetime=1656031991962&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

| 从https://github.com/apache/incubator-doris-spark-connector 下载并编译，编译的时候建议使用Doris官方提供的编译镜像编译。

```
$ docker pull apache/incubator-doris:build-env-ldb-toolchain-latest
```

编译结果如下：

```
[root@xxx spark-doris-connector]# pwd  
/data/incubator-doris-spark-connector/spark-doris-connector  
[root@xxx spark-doris-connector]# ls target/ -trhl  
总用量 7.7M  
drwxr-xr-x 3 root root 4.0K 5月  11 22:32 maven-shared-archive-resources  
-rw-r--r-- 1 root root    1 5月  11 22:32 classes.520860527.timestamp  
drwxr-xr-x 5 root root 4.0K 5月  11 22:32 classes  
drwxr-xr-x 5 root root 4.0K 5月  11 22:32 generated-sources  
drwxr-xr-x 3 root root 4.0K 5月  11 22:32 maven-status  
drwxr-xr-x 3 root root 4.0K 5月  11 22:32 thrift-dependencies  
drwxr-xr-x 2 root root 4.0K 5月  11 22:32 maven-archiver  
-rw-r--r-- 1 root root 174K 5月  11 22:32 spark-doris-connector-3.1_2.12-1.0.0-SNAPSHOT-sources.jar  
-rw-r--r-- 1 root root    1 5月  11 22:32 test-classes.104362910.timestamp  
drwxr-xr-x 4 root root 4.0K 5月  11 22:32 test-classes  
drwxr-xr-x 3 root root 4.0K 5月  11 22:32 generated-test-sources  
drwxr-xr-x 2 root root 4.0K 5月  11 22:32 surefire-reports  
-rw-r--r-- 1 root root 551K 5月  11 22:32 original-spark-doris-connector-3.1_2.12-1.0.0-SNAPSHOT.jar  
-rw-r--r-- 1 root root 7.0M 5月  11 22:32 spark-doris-connector-3.1_2.12-1.0.0-SNAPSHOT.jar  
[root@xx spark-doris-connector]#
```

从官网下载Spark，如果官网比较慢，这里有个腾讯的镜像地址，十分方便。https://mirrors.cloud.tencent.com/apache/spark/spark-3.1.2/

执行命令下载编译好的spark包，并解压。

```
#下载  
wget https://mirrors.cloud.tencent.com/apache/spark/spark-3.1.2/spark-3.1.2-bin-hadoop3.2.tgz  
#解压  
 tar -xzvf spark-3.1.2-bin-hadoop3.2.tgz
```

配置Spark环境。

```
vim /etc/profile  
export SPARK_HOME=/your_parh/spark-3.1.2-bin-hadoop3.2  
export PATH=$PATH:$SPARK_HOME/bin  
source /etc/profile
```

将编译好的spark-doris-connector-3.1\_2.12-1.0.0-SNAPSHOT.jar复制到spark 的jars目录。

```
cp /your_path/spark-doris-connector/target/spark-doris-connector-3.1_2.12-1.0.0-SNAPSHOT.jar  $SPARK_HOME/jars
```

## Scala 使用方式

执行 spark-shell，进入Spark交互环境。确定Spark的版本。

8. 执行如下命令，通过Spark-Doris-Conneter 查询Doirs数据。

   ```
   import org.apache.doris.spark._  
   val dorisSparkRDD = sc.dorisRDD(  
     tableIdentifier = Some("mongo_doris.data_sync_test"),  
     cfg = Some(Map(  
    "doris.fenodes" -> "127.0.0.1:8030",  
    "doris.request.auth.user" -> "root",  
    "doris.request.auth.password" -> ""  
     ))  
   )  
   dorisSparkRDD.collect()
   ```

5. doris.request.auth.password为密码。
6. doris.request.auth.user为用户名，
7. doris.fenodes为FE节点的IP和http\_port，
8. data\_sync\_test为表名称，
9. mongo\_doris为数据库名称，

9. 执行完成后会将数据输出在控制台，如果看到数据输出则代表对接完成了。完整的情况如下：

```
scala> import org.apache.doris.spark._  
import org.apache.doris.spark._  
  
scala> val dorisSparkRDD = sc.dorisRDD(  
     |   tableIdentifier = Some("mongo_doris.data_sync_test"),  
     |   cfg = Some(Map(  
     |     "doris.fenodes" -> "127.0.0.1:8030",  
     |     "doris.request.auth.user" -> "root",  
     |     "doris.request.auth.password" -> ""  
     |   ))  
     | )  
dorisSparkRDD: org.apache.spark.rdd.RDD[AnyRef] = ScalaDorisRDD[0] at RDD at AbstractDorisRDD.scala:32  
  
scala> dorisSparkRDD.collect()  
res0: Array[AnyRef] = Array([4, 1, alex, Document{{key1=1.0}}, 20.0, 3.14, 123456.0, 2022-05-10, false], [2, 1, alex, [1.0, 2.0, 3.0], 20.0, 3.14, 123456.0, 2022-05-09, false], [3, 1, alex, [Document{{key1=1.0}}], 20.0, 3.14, 123456.0, 2022-05-10, false])
```

集群版Spark一般会将依赖Jar包上传到HDFS，然后通过spark.yarn.jars添加HDFS路径，Spark会从HDFS上读取Jar包。

```
spark.yarn.jars=local:/usr/lib/spark/jars/*,hdfs:///spark-jars/doris-spark-connector-3.1.2-2.12-1.0.0.jar  
具体参照：https://github.com/apache/incubator-doris/discussions/9486
```

## pyspark使用方式

输入如下命令进入pyspark

```
[root@xxx ~]# pyspark  
Python 3.6.9 (default, Dec  8 2021, 21:08:43)   
[GCC 8.4.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
22/05/12 10:29:25 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable  
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties  
Setting default log level to "WARN".  
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).  
22/05/12 10:29:26 WARN Utils: Service 'SparkUI' could not bind on port 4040. Attempting port 4041.  
Welcome to  
      ____              __  
     / __/__  ___ _____/ /__  
    _\ \/ _ \/ _ `/ __/  '_/  
   /__ / .__/\_,_/_/ /_/\_\   version 3.1.2  
      /_/  
  
Using Python version 3.6.9 (default, Dec  8 2021 21:08:43)  
Spark context Web UI available at http://jiafeng:4041  
Spark context available as 'sc' (master = local[*], app id = local-1652322566766).  
SparkSession available as 'spark'.
```

通过pysprk从Doris读取数据.

```
dorisSparkDF = spark.read.format("doris").option("doris.table.identifier", "mongo_doris.data_sync_test").option("doris.fenodes", "127.0.0.1:8030").option("user", "root").option("password", "").load()  
  
# 显示5行数据  
  
dorisSparkDF.show(5)
```

4 . 完成运行结果如下:

```
>>> dorisSparkDF = spark.read.format("doris").option("doris.table.identifier", "mongo_doris.data_sync_test").option("doris.fenodes", "127.0.0.1:8030").option("user", "root").option("password", "").load()  
>>> dorisSparkDF.show(5)  
+---+---+---------+--------------------+----+------+------------+-----------+----------+  
|_id| id|user_name|         member_list| age|height|lucky_number|create_time|is_married|  
+---+---+---------+--------------------+----+------+------------+-----------+----------+  
|  3|  1|     alex|[Document{{key1=1...|20.0|  3.14|    123456.0| 2022-05-10|     false|  
|  4|  1|     alex|Document{{key1=1.0}}|20.0|  3.14|    123456.0| 2022-05-10|     false|  
|  2|  1|     alex|     [1.0, 2.0, 3.0]|20.0|  3.14|    123456.0| 2022-05-09|     false|  
+---+---+---------+--------------------+----+------+------------+-----------+----------+
```