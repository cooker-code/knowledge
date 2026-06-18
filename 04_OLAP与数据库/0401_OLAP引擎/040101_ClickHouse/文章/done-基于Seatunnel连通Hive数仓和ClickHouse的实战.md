> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040101_ClickHouse/040101_核心知识点/ClickHouse导入与外部引擎接入边界|ClickHouse导入与外部引擎接入边界]]
---
title: 基于Seatunnel连通Hive数仓和ClickHouse的实战
author: 大数据技术与架构
date:
url: http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247512101&idx=1&sn=147ac5578d6967004e28379d072f37f8&chksm=fd3ef6b0ca497fa6e5d616a40c7cf968f0481dd74d3df515e02e8f0d23e095098b80be4bc4e3&mpshare=1&scene=24&srcid=0323S5G8MXzxDXYboODQoQjR&sharer_sharetime=1647992828553&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

点击上方**蓝色字体**，选择“设为星标”

回复"**面试"**获取更多惊喜

[**八股文教给我，你们专心刷题和面试**](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247512011&idx=1&sn=68aaa9c5c42e2087d56d170c442b30ba&chksm=fd3ee95eca496048b24a717a30fe820b9188d02e014ba7b554afe86f8da1efab189799fe7087&scene=21#wechat_redirect)

> Hi，我是王知无，一个大数据领域的原创作者。
>
> 放心关注我，获取更多行业的一手消息。

## 背景

目前公司的分析数据基本存储在 Hive 数仓中，使用 Presto 完成 OLAP 分析，但是随着业务实时性增强，对查询性能的要求不断升高，同时许多数据应用产生，比如对接 BI 进行分析等，Presto不能满足需求，在这个阶段我们引入了ClickHouse，用来建设性能更强悍，响应时间更短的数据分析平台，以满足实时性要求，但如何连通 Hive 数仓和ClickHouse呢？没错，当然是 Seatunnel 啦！

## 01 环境准备

> 官方推荐的
> seatunnel1.5.7+spark2.4.8+scala2.11

全部解压安装到`/u/module`下即可

`[hadoop@hadoop101 module]$ unzip /u/software/19.Seatunnel/seatunnel-1.5.7.zip -d /u/module/`

`[hadoop@hadoop101 module]$ tar -zxvf /u/software/19.Seatunnel/spark-2.4.8-bin-hadoop2.7.tgz -C /u/module`

`[hadoop@hadoop101 module]$ tar -zxvf /u/software/19.Seatunnel/scala-2.11.8.tgz -C /u/module`

将 hive-site.xml 复制到 spark2/conf 目录下，这里取的是从 hive 复制到 Hadoop 配置目录下的

`[hadoop@hadoop101 module]$ cp $HADOOP_CONF/hive-site.xml /u/module/spark-2.4.8-bin-hadoop2.7/conf`

注意：如果你跟我一样，原来 Hive 默认使用Spark3，那么需要设置一个 Spark2 的环境变量

`[hadoop@hadoop101 module]$ sudo vim /etc/profile`

```
# SPARK_HOME
export SPARK_HOME=/u/module/spark
export PATH=$PATH:$SPARK_HOME/bin
# SPARK_END

# 多版本共存Spark，for waterdrop and Hive
export SPARK2_HOME=/u/module/spark-2.4.8-bin-hadoop2.7

#Scala Env
export SCALA_HOME=/u/module/scala-2.11.8/
export PATH=$PATH:$SCALA_HOME/bin
```

`[hadoop@hadoop101 module]$ source /etc/profile`

创建jobs目录存放执行conf文件

```
[hadoop@hadoop101 module]$ mkdir /u/module/seatunnel-1.5.7/jobs
```

## 02 数据准备

##### Hive：

```
drop table if exists prod_info;
create table prod_info
(
    prod_sn    string comment 'sn',
    create_time string comment '创建时间'
)COMMENT '产品信息表'
PARTITIONED BY (`dt` string)
STORED AS PARQUET
TBLPROPERTIES ("parquet.compression" = "lzo");
```

##### 插入数据：

```
insert into prod_info values ('F0001','2022-01-18 00:00:00.0','2022-01-18');
insert into prod_info values ('F00012','2022-01-19 00:00:00.0','2022-01-19');
```

##### ClickHouse:

```
drop table if exists prod_info;
create table prod_info
(
    prod_sn    String,
    create_time DateTime
)engine =MergeTree
    partition by toYYYYMMDD(create_time)
 primary key (prod_sn)
    ORDER BY (prod_sn)
```

## 03 多表全量or增量数据导入CK

使用cat <<!EOF把变量传进去，把脚本生成在jobs文件夹中，然后再使用 seatunnel 的命令执行

关键点：

1. 将输入参数封装成一个方法，方便一个脚本操作多个数仓表;
2. 加入CK远程执行命令，插入前清除分区，以免导入双倍数据;
3. 加入批量执行条件;

`[hadoop@hadoop101 module]$ touch ~/bin/mytest.sh && chmod u+x ~/bin/mytest.sh && vim ~/bin/mytest.sh`

注意：

1. 这边 hive 中表压缩格式是 parquet+lzo ，读取出来没问题，插入时报错，我直接将之前搭建 Hadoop集群时$HADOOP\_HOME/share/hadoop/common/hadoop-lzo-0.4.20.jar放到/u/module/spark-2.4.8-bin-hadoop2.7/jars（spark 目录下的 jars ）下，即可解决
2. 若 hive 表中有做分区，则需指定 spark.sql.hive.manageFilesourcePartitions=false

```
#!/bin/bash

# 环境变量
unset SPARK_HOME
export SPARK_HOME=$SPARK2_HOME
SEATUNNEL_HOME=/u/module/seatunnel-1.5.7
CLICKHOUSE_CLIENT=/usr/bin/clickhouse-client
# 接收两个参数，第一个为要抽取的表，第二个为抽取时间
# 若输入的第一个值为first，不输入第二参数则直接退出脚本
if [[ $1 = first ]]; then
  if [ -n "$2" ] ;then
   do_date=$2
  else 
   echo "请传入日期参数"
   exit
  fi 
# 若输入的第一个值为all，不输入第二参数则取前一天
elif [[ $1 = all ]]; then
    # 判断非空，如果不传时间默认取前一天数据，传时间就取设定，主要是用于手动传参
  if [ -n "$2" ] ;then
    do_date=$2
  else
    do_date=`date -d '-1 day' +%F`
  fi
else
  if [ -n "$2" ] ;then
   do_date=$2
  else 
   echo "请传入日期参数"
   exit
  fi 
fi

echo "日期：$do_date"

import_conf(){
  # 打印数据传输脚本并赋值
cat>$SEATUNNEL_HOME/jobs/hive2ck_test.conf<<!EOF
spark {
  spark.sql.catalogImplementation = "hive"
  spark.app.name = "hive2clickhouse"
  spark.executor.instances = 4
  spark.executor.cores = 4
  spark.executor.memory = "4g"
  # 此参数为调用Hive分区必带！
  spark.sql.hive.manageFilesourcePartitions=false
}

input {
    hive {
                pre_sql = "$1"
                table_name = "$2"
    }
}

filter {}

output {
    clickhouse {
                host = "$3"
                database = "$4"
                table = "$5"
                fields = $6
                username = "default"
                password = ""
    }
}

!EOF
$SEATUNNEL_HOME/bin/start-seatunnel.sh  --config $SEATUNNEL_HOME/jobs/hive2ck_test.conf -e client -m 'local[4]'
}
# 全量数据导入
import_prod_info_first(){
  $CLICKHOUSE_CLIENT --host hadoop101 --database test --query="truncate table test.prod_info"
  import_conf "select prod_sn,substring(create_time,1,19) as create_time from default.prod_info" "prod_info" "hadoop101:8123" "test" "prod_info" "[\"prod_sn\",\"create_time\"]"
}

# 增量数据导入
import_prod_info(){
  do_date_2=`echo $do_date |sed 's/-//g'`
  # 为避免重复导入，导入前先清除分区，这是在建立表分区的前提下
  $CLICKHOUSE_CLIENT --host hadoop101 --database test --query="alter table test.prod_info drop partition '${do_date_2}'"
  import_conf "select prod_sn,substring(create_time,1,19) as create_time from default.prod_info where dt='${do_date}'" "prod_info" "hadoop101:8123" "test" "prod_info" "[\"prod_sn\",\"create_time\"]"
}


case $1 in
"prod_info_first"){
    import_prod_info_first
};;
"prod_info"){
    import_prod_info
};;
"first"){
 import_prod_info_first
};;
"all"){
 import_prod_info
};;
 "tmp"){
  import_prod_info
};;
esac
```

##### 03.1首日全量导入

执行首日全量导入，后面的 2022-01-19 是为了配合数仓流程加入的

```
[hadoop@hadoop101 bin]$ mytest.sh first 2022-01-19
```

ClickHouse中查看是否导入：

查看CK的当前分区:

```
select * from system.parts p where table = 'prod_info' order by partition desc ;
```

可见数据导入无误～

##### 03.2每日增量导入

hive中新增记录测试增量更新：

```
hive> insert into prod_info values ('F000123','2022-01-20 00:00:00.0','2022-01-20');
```

```
[hadoop@hadoop101 bin]$ mytest.sh all 2022-01-20
```

可见增量更新脚本也无误！

调试时可以修改 tmp 条件里的内容，然后使用调度工具如Dolphin Scheduler、Azkaban上监控多个脚本的分步执行情况，以便定位问题。

## 04 总结

本文主要分享了一个基于 Seatunnel 的生产力脚本，介绍了如何连通 Hive 数仓与 ClickHouse ，将 ClickHouse 无缝加入离线数仓流程，并进行流程测试。实际生产使用时，数据传输速度飞快。

如果这个文章对你有帮助，不要忘记 **「在看」** **「点赞」** **「收藏」** 三连啊喂！

[2022年全网首发|大数据专家级技能模型与学习指南(胜天半子篇)](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247510040&idx=1&sn=aa335f25965975731173916f012d56f4&chksm=fd3eee8dca49679b82f632048fb64d21fac01497d1a0fe33917edb01e194caf0f9a1930a55ce&scene=21#wechat_redirect)

[互联网最坏的时代可能真的来了](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247508317&idx=1&sn=0bcb7fb6997b42306994b890eaa0d47f&chksm=fd3ee7c8ca496ede347a2971002c6ea68dcfd70abeeb90b43c8b02a2a60ebc7cec3a87ef7d52&scene=21#wechat_redirect)

[我在B站读大学，大数据专业](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247507860&idx=1&sn=807ac5003762f29c127bc4071dcebe33&chksm=fd3e9901ca4910178cfc816043ea86c29f881f70cd943d252de9ae2e21cb7e8ba80bff162e1b&scene=21#wechat_redirect)

[我们在学习Flink的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247499604&idx=1&sn=d938dfb30d221774704982d2938b30c1&chksm=fd3eb9c1ca4930d76a391241333de461ca22d2aa27472eab3cffab1564872ae37f1b48fe2d3c&scene=21#wechat_redirect)

[193篇文章暴揍Flink，这个合集你需要关注一下](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504856&idx=2&sn=6f62e2a0c756ce56773eed253d2f41ee&chksm=fd3e954dca491c5b732b7e2aa46db32efbc3f690e528522098659bb3c6e78e1582ab4529b5dc&scene=21#wechat_redirect)

[Flink生产环境TOP难题与优化，阿里巴巴藏经阁YYDS](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504742&idx=1&sn=8765c198d8ad66219a7bcabb221e4a23&chksm=fd3e95f3ca491ce52d0724b9e4154a47af0f5ea349e1e184c8fbc65aa17c0000242aa9b58429&scene=21#wechat_redirect)

[Flink CDC我吃定了耶稣也留不住他！](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504813&idx=1&sn=f5cd6ae2aa2b1e30f87a5ae55971c514&chksm=fd3e9538ca491c2eb191677d070f2c7e4f1098eece00e256d6205b8a5d21b21c1e74f875f16f&scene=21#wechat_redirect)[| Flink CDC线上问题小盘点](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504813&idx=1&sn=f5cd6ae2aa2b1e30f87a5ae55971c514&chksm=fd3e9538ca491c2eb191677d070f2c7e4f1098eece00e256d6205b8a5d21b21c1e74f875f16f&scene=21#wechat_redirect)

[我们在学习Spark的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=1&sn=1f693e549f76622725b936041ff8896e&chksm=e9bff2fbdec87bedecbdeed4547d7d612c72fc8d4918614a26a4e31666cf0128fd57b537f347&scene=21#wechat_redirect)

[在所有Spark模块中，我愿称SparkSQL为最强！](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504834&idx=1&sn=46ca1a3924b8fdb89ac1c2acb0319d6c&chksm=fd3e9557ca491c41d837b917639ea62007ea16d3c2cc6c0c02d1f8a8c6e44318f5183b787c46&scene=21#wechat_redirect)

[硬刚Hive | 4万字基础调优面试小总结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247502750&idx=1&sn=bd9a9173d060dc4e4ebd49c8efc6acfe&chksm=fd3e8d0bca49041dea84da93910e5efdc4935e520525c09887c986691377aeb48e5cf7fb5667&scene=21#wechat_redirect)

[数据治理方法论和实践小百科全书](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504382&idx=2&sn=550b78802acfe727e0e77cd9195f8784&chksm=fd3e976bca491e7db2b8b2446d231736df01bbf13653804d680ac4390597ec17fa466ad4ae83&scene=21#wechat_redirect)

[标签体系下的用户画像建设小指南](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503741&idx=1&sn=e5039be93123f2e337013756a818bfc3&chksm=fd3e89e8ca4900fe603b63c5722a6fb8a32bd63d6ba23e0028851948a71b877eb1f742d95087&scene=21#wechat_redirect)

[4万字长文 | ClickHouse基础&实践&调优全视角解析](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503675&idx=1&sn=3ee6af64d0126c78b48cad219308f81e&chksm=fd3e89aeca4900b8b8954e9569ee3c0877881fac8c792bfafc22e7e9d3e8524da8eb860d33d8&scene=21#wechat_redirect)

[【面试&个人成长】2021年过半，社招和校招的经验之谈](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=2&sn=57ecc77718f1b6f2f262d62e8318dcc9&chksm=e9bff2fbdec87bedb1986765bfae7dbabb9aece45b0b2af147bbad1ac5d79ebc934c64df40ea&scene=21#wechat_redirect)

[大数据方向另一个十年开启 |《硬刚系列》第一版完结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504478&idx=1&sn=14efef3868ba42bd044f9618745a7fdc&chksm=fd3e94cbca491dddb961b7b8b93105b2869c5bcf4c03f9f0dc83ad62be0dce4c124b52ed0ba3&scene=21#wechat_redirect)

[我写过的关于成长/面试/职场进阶的文章](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504410&idx=1&sn=7e81ab5a324395eb0f12397c40247ca1&chksm=fd3e948fca491d9946456acbd93b2d651ae4d7a7d0127a211981e08f50f377e60d1b7c0d7b3a&scene=21#wechat_redirect)

[当我们在学习Hive的时候在学习什么？「硬刚Hive续集」](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504783&idx=1&sn=72aed147a459368ed934a007a2df3bc9&chksm=fd3e951aca491c0cade212390011eca7d8a68951fee6aa2844b8f4d14bcbd5b3ed26305d69dc&scene=21#wechat_redirect)