> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030203_Kyuubi/030203_核心知识点/Kyuubi企业级SQL网关1.8边界|Kyuubi企业级SQL网关1.8边界]]、[[03_数据工程与数仓/0302_离线数仓/030203_Kyuubi/030203_核心知识点/Kyuubi多引擎SQL网关接入边界|Kyuubi多引擎SQL网关接入边界]]
---
title: 大数据统一SQL网关：最新版Kyuubi整合Flink、Spark方案的实践案例总结
author: 大数据从业者
date:
url: http://mp.weixin.qq.com/s?__biz=MzU1NjYyMDE5OA==&mid=2247485848&idx=1&sn=fcbfde1aba78756f03e35b3e87fac203&chksm=fbc30951ccb48047dd64525aaf8c6ad72d5c78d515ebeb6c6eb797461c499b292812021c5b5c&mpshare=1&scene=24&srcid=070939Qf4v2YII8JUkeGHpYo&sharer_shareinfo=3a692c31e5c2d48df800fcf05a8d9344&sharer_shareinfo_first=3a692c31e5c2d48df800fcf05a8d9344#rd
---

## **前言**

Kyuubi最新版本已经发布，本文主要介绍基于Kyuubi SQL网关整合多计算引擎Flink和Spark实践案例总结。另外，翻看Release Notes发现Kyuubi Web UI功能增强，新增SQL编辑器，本文亦一并尝鲜实践记录。

详细的组件版本信息如下：

|  |  |
| --- | --- |
| Kyuubi | 1.9.0 |
| Spark | 3.3.2 |
| Flink | 1.17.2 |
| Hive | 3.1.0.3.1.5.0-152 |
| Hadoop | 3.1.1.3.1.5.0-152 |

欢迎关注微信公众号大数据从业者！

## **Flink整合Hadoop Hive**

Flink与周边兼容性良好，这里直接使用官方安装包即可，不源码编译！

下载官方安装包

```
[root@felixzh myHadoopCluster]# wget https://archive.apache.org/dist/flink/flink-1.17.2/flink-1.17.2-bin-scala_2.12.tgz[root@felixzh myHadoopCluster]# tar -xvf flink-1.17.2-bin-scala_2.12.tgz
```

准备相关Jar

将flink-sql-connector-hive-3.1.3\_2.12-1.17.2.jar放置到Flink/lib目录

```
https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-hive-3.1.3_2.12/1.17.2/flink-sql-connector-hive-3.1.3_2.12-1.17.2.jar     
```

设置环境变量

```
[root@felixzh flink-1.17.2]# vim bin/config.shexport HADOOP_CLASSPATH=/usr/hdp/3.1.5.0-152/hadoop-hdfs/*:/usr/hdp/3.1.5.0-152/hadoop-hdfs/lib/*:/usr/hdp/3.1.5.0-152/hadoop-yarn/*:/usr/hdp/3.1.5.0-152/hadoop-yarn/lib/*:/usr/hdp/3.1.5.0-152/hadoop/*:/usr/hdp/3.1.5.0-152/hadoop-mapreduce/*export HADOOP_CONF_DIR=/etc/hadoop/conf/          
```

设置配置信息(可选)

```
[root@felixzh flink-1.17.2]# vim conf/flink-conf.yaml
high-availability.type: zookeeperhigh-availability.zookeeper.quorum: felixzh:2181high-availability.storageDir: hdfs:///flink/ha/classloader.check-leaked-classloader: false
```

实践验证

通过提交简单word count测试例，验证Flink与Hadoop整合效果，如下：

通过FlinkSqlClient验证Hive Catalog整合效果，如下：             

## **Spark整合Hadoop Hive**

本文不再重复赘述，请参见历史文章《一文掌握最新数据湖方案Spark+Hadoop+Hudi+Hive整合案例实践总结》相关章节

通过SparkSql验证整合效果如下：

## **Kyuubi源码编译**

下载源码    

```
[root@felixzh mySourceCode]# git clone -b v1.9.0 https://github.com/apache/kyuubi.git
```

编译打包

```
[root@felixzh mySourceCode]# build/dist --tgz --spark-provided --flink-provided --hive-provided --web-ui -Pspark3.3 -Pflink-1.17
```

如上图所示，Kyuubi源码根目录可以找到安装包apache-kyuubi-1.9.0-bin.tgz          

说明：

```
1.编译环境maven版本要与Kyuubi源码默认maven版本一致。2.注意周边配套组件版本。3.Spark、Flink、Hive已单独提供，编译不再下载。4.--web-ui表示启用Kyuubi Web UI源码编译。
```

## **Kyuubi整合Flink Spark**

基于上述安装包解压

```
[root@felixzh myHadoopCluster]# tar -xvf apache-kyuubi-1.9.0-bin.tgz  
```

设置环境变量

```
[root@felixzh apache-kyuubi-1.9.0-bin]# vim conf/kyuubi-env.shexport SPARK_HOME=/home/myHadoopCluster/spark-3.3.2-bin-hadoop3/    export SPARK_CONF_DIR=/home/myHadoopCluster/spark-3.3.2-bin-hadoop3/confexport HADOOP_CONF_DIR=/etc/hadoop/confexport FLINK_HOME=/home/myHadoopCluster/flink-1.17.2export FLINK_HADOOP_CLASSPATH=/usr/hdp/3.1.5.0-152/hadoop-hdfs/*:/usr/hdp/3.1.5.0-152/hadoop-hdfs/lib/*:/usr/hdp/3.1.5.0-152/hadoop-yarn/*:/usr/hdp/3.1.5.0-152/hadoop-yarn/lib/*:/usr/hdp/3.1.5.0-152/hadoop/*:/usr/hdp/3.1.5.0-152/hadoop-mapreduce/* 
```

设置配置信息

```
[root@felixzh apache-kyuubi-1.9.0-bin]# vim conf/kyuubi-defaults.confkyuubi.ha.zookeeper.quorum  felixzh:2181kyuubi.ha.zookeeper.namespace  kyuubi1.9.0kyuubi.engine.type SPARK_SQLkyuubi.frontend.rest.bind.port 10099kyuubi.engine.share.level USERkyuubi.metadata.store.jdbc.database.type MYSQLkyuubi.metadata.store.jdbc.url jdbc:mysql://felixzh:3306/kyuubikyuubi.metadata.store.jdbc.user rootkyuubi.metadata.store.jdbc.password 123456    
```

说明：metadata store使用MYSQL，需要提前自行创建数据库(如kyuubi)、需要自行准备JDBC驱动mysql-connector-java-8.0.29.jar到Kyuubi/jars目录。       

启动Kyuubi Server

```
[root@felixzh apache-kyuubi-1.9.0-bin]# ./bin/kyuubi start
```

浏览器访问：http://felixzh:10099/，即可看到Kyuubi Web UI，如下：

      

### **Kyuubi on Spark**

kyuubi.engine.type默认为SPARK\_SQL，这里直接使用kyuubi beeline连接基于zooKeeper开启HA的Kyuubi Server即可，如下：

```
[root@felixzh bin]# ./beeline -u 'jdbc:hive2://felixzh:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=kyuubi1.9.0'
```

上图可以看到，Kyuubi已经基于spark-submit启动SparkSQLEngine。

通过Kyuubi Web UI可以看到对应的Session信息，如下：

   

通过Kyuubi Web UI可以看到对应的Operation信息，如下：

通过Kyuubi Web UI可以看到对应的Engine信息，如下：

并且，通过上图中Engine UI可以直接跳转到对应的Application Matser UI，这点还是很方便的，如下：

通过Kyuubi Web UI可以看到对应的Server信息，如下：

至于SQL Editor，目前仅支持SPARK\_SQL引擎类型。

测试基本功能，如下：   

```
create table kyuubitest (username string);insert into kyuubitest values ("felixzh");select * from kyuubitest;
```

### **Kyuubi on Flink**

kyuubi.engine.type默认为SPARK\_SQL，jdbc url支持动态设置引擎类型和运行模式。

#### Application Mode

```
[root@felixzh bin]# ./beeline -u 'jdbc:hive2://felixzh:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=kyuubi1.9.0;#kyuubi.engine.type=FLINK_SQL;flink.execution.target=yarn-application'
```

执行简单测试用例如下：

```
SELECT  name,  COUNT(*) AS cntFROM  (VALUES ('Bob'), ('Alice'), ('Greg'), ('Bob')) AS NameTable(name)GROUP BY name;
```

#### Session Mode

```
[root@felixzh bin]# ./yarn-session.sh -d
```

```
[root@felixzh bin]# ./beeline -u 'jdbc:hive2://felixzh:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=kyuubi1.9.0;#kyuubi.engine.type=FLINK_SQL;flink.execution.target=yarn-session;flink.yarn.application.id=application_1698577744226_0068'
```

执行上述测试用例，可以看到SQL已经提交到yarn-session集群运行。