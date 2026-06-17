---
title: Apache Griffin+Flink+Kafka实现流式数据质量监控实战
author: 大数据技术与架构
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247512073&idx=1&sn=b537dbcea355b59a2d302dbcbc94c3aa&chksm=fd3ef69cca497f8a9c7e855c53906f28f6d108b1ece352fb826286fde2c41e78c5721d7379e2&mpshare=1&scene=24&srcid=0316UrVhYmhOedgGSf09RR2T&sharer_sharetime=1647432696933&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

点击上方**蓝色字体**，选择“设为星标”

回复"**面试"**获取更多惊喜

[**八股文教给我，你们专心刷题和面试**](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247512011&idx=1&sn=68aaa9c5c42e2087d56d170c442b30ba&chksm=fd3ee95eca496048b24a717a30fe820b9188d02e014ba7b554afe86f8da1efab189799fe7087&scene=21#wechat_redirect)

> Hi，我是王知无，一个大数据领域的原创作者。
>
> 放心关注我，获取更多行业的一手消息。

## 一. 组件及版本

本文用的组件包括以下几个，是参考了官方案例,版本可以参考github以及里面的pom文件。本文假定以下环境均已安装好。

* JDK (1.8)
* MySQL(version 5.6)
* Hadoop (2.7.2)
* Hive (version 2.4)
* Spark (version 2.4.1)
* Kafka (version 0.11)
* Griffin (version 0.6.0)
* Zookeeper (version 3.4.1)

这里有详细的配置过程和可能遇到的bug。

## 二. kafka数据生成脚本

由于是测试案例，我们就写一个生成数据的脚本，并且把数据写到kafka source中，真实的场景应该是源源不断写数据到kafka中的（比如flume或者其他工具），具体数据脚本和模版可以参考官方demo数据

gen-data.sh

```
#!/bin/bash  
  
#current time  
cur_time=`date +%Y-%m-%d_%H:%M:%S`  
sed s/TIME/$cur_time/ /opt/module/data/source.temp > /opt/module/data/source.tp  
  
#create data  
for row in 1 2 3 4 5 6 7 8 9 10  
do  
  sed -n "${row}p" < /opt/module/data/source.tp > sline  
  cnt=`shuf -i1-2 -n1`  
  clr="red"  
  if [ $cnt == 2 ]; then clr="yellow"; fi  
  sed s/COLOR/$clr/ sline >> /opt/module/data/source.data  
done  
rm sline  
  
rm source.tp  
  
#import data  
kafka-console-producer.sh --broker-list hadoop101:9092 --topic source < /opt/module/data/source.data  
  
rm source.data  
  
echo "insert data at ${cur_time}"
```

streaming-data.sh

```
#!/bin/bash  
  
#create topics  
kafka-topics.sh --create --zookeeper hadoop101:2181 --replication-factor 1 --partitions 1 --topic source  
kafka-topics.sh --create --zookeeper hadoop101:2181 --replication-factor 1 --partitions 1 --topic target  
  
#every minute  
set +e  
while true  
do  
  /opt/module/data/gen-data.sh  
  sleep 90  
done  
set -e
```

source.temp

```
{"id": 1, "name": "Apple", "color": "COLOR", "time": "TIME"}  
{"id": 2, "name": "Banana", "color": "COLOR", "time": "TIME"}  
{"id": 3, "name": "Cherry", "color": "COLOR", "time": "TIME"}  
{"id": 4, "name": "Durian", "color": "COLOR", "time": "TIME"}  
{"id": 5, "name": "Lichee", "color": "COLOR", "time": "TIME"}  
{"id": 6, "name": "Peach", "color": "COLOR", "time": "TIME"}  
{"id": 7, "name": "Papaya", "color": "COLOR", "time": "TIME"}  
{"id": 8, "name": "Lemon", "color": "COLOR", "time": "TIME"}  
{"id": 9, "name": "Mango", "color": "COLOR", "time": "TIME"}  
{"id": 10, "name": "Pitaya", "color": "COLOR", "time": "TIME"}
```

## 三. Flink流式处理

flink流式数据分成三个部分，读取kafka，业务处理，写入kafka

1. 首先交代我的pom.xml引入的依赖

```
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
  
    <groupId>com.xxxx</groupId>  
    <artifactId>kafka_Flink_kafka_Test</artifactId>  
    <version>1.0-SNAPSHOT</version>  
  
    <build>  
        <plugins>  
            <plugin>  
                <artifactId>maven-compiler-plugin</artifactId>  
                <version>3.7.0</version>  
                <configuration>  
                    <source>1.8</source>  
                    <target>1.8</target>  
                </configuration>  
            </plugin>  
  
  
  
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-shade-plugin</artifactId>  
                <version>3.1.0</version>  
                <executions>  
                    <execution>  
                        <phase>package</phase>  
                        <goals>  
                            <goal>shade</goal>  
                        </goals>  
                        <configuration>  
                            <transformers>  
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">  
                                    <mainClass>com.ink.FlinkLambdaTest.FlinkToLambda</mainClass>  
                                </transformer>  
                                <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">  
                                    <resource>reference.conf</resource>  
                                </transformer>  
                            </transformers>  
                            <relocations>  
                                <relocation>  
                                    <pattern>org.codehaus.plexus.util</pattern>  
                                    <shadedPattern>org.shaded.plexus.util</shadedPattern>  
                                    <excludes>  
                                        <exclude>org.codehaus.plexus.util.xml.Xpp3Dom</exclude>  
                                        <exclude>org.codehaus.plexus.util.xml.pull.*</exclude>  
                                    </excludes>  
                                </relocation>  
                            </relocations>  
                        </configuration>  
                    </execution>  
                </executions>  
            </plugin>  
        </plugins>  
    </build>  
  
    <dependencies>  
        <!--<dependency>-->  
        <!--<groupId>org.apache.flink</groupId>-->  
        <!--<artifactId>flink-table_2.10</artifactId>-->  
        <!--<version>1.3.2</version>-->  
        <!--</dependency>-->  
        <dependency>  
            <groupId>org.json</groupId>  
            <artifactId>json</artifactId>  
            <version>20090211</version>  
        </dependency>  
        <dependency>  
            <groupId>com.google.code.gson</groupId>  
            <artifactId>gson</artifactId>  
            <version>2.6.2</version>  
        </dependency>  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-java</artifactId>  
            <version>1.10.1</version>  
        </dependency>  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-streaming-java_2.11</artifactId>  
            <version>1.10.1</version>  
        </dependency>  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-clients_2.11</artifactId>  
            <version>1.10.1</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-scala_2.11</artifactId>  
            <version>1.10.1</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-streaming-scala_2.11</artifactId>  
            <version>1.10.1</version>  
        </dependency>  
  
  
  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-connector-kafka-0.10_2.11</artifactId>  
            <version>1.10.1</version>  
        </dependency>  
    </dependencies>  
  
</project>
```

2. 先写个bean类模版，用来接收json数据

```
import java.util.Date;  
  
public class Student{  
    private int id;  
    private String name;  
    private String color;  
    private Date time;  
  
    public Student(){}  
  
    public Student(int id, String name, String color, Date time) {  
        this.id = id;  
        this.name = name;  
        this.color = color;  
        this.time = time;  
    }  
  
    public int getId() {  
        return id;  
    }  
  
    public void setId(int id) {  
        this.id = id;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
  
    public String getColor() {  
        return color;  
    }  
  
    public void setColor(String color) {  
        this.color = color;  
    }  
  
    public Date getTime() {  
        return time;  
    }  
  
    public void setTime(Date time) {  
        this.time = time;  
    }  
  
    @Override  
    public String toString() {  
        return "Student{" +  
                "id=" + id +  
                ", name='" + name + '\'' +  
                ", color='" + color + '\'' +  
                ", time='" + time + '\'' +  
                '}';  
    }  
}
```

3. 读取kafka，有关读取和写入kafka的配置信息，是可以写到kafkaUtil工具类中的，我这里为了方便，就直接嵌入到代码中了，就做个测试

```
// 创建Flink执行环境  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        // 一定要设置启动检查点！！  
        //env.enableCheckpointing(5000);  
        //env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);  
        env.setParallelism(1);  
  
        // Kafka参数  
        Properties properties = new Properties();  
        properties.setProperty("bootstrap.servers", "hadoop101:9092");  
        properties.setProperty("group.id", "consumer-group");  
        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");  
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");  
        properties.setProperty("auto.offset.reset", "latest");  
        String inputTopic = "source";  
        String outputTopic = "target";  
  
        // Source  
        FlinkKafkaConsumer010<String> consumer =  
                new FlinkKafkaConsumer010<String>(inputTopic, new SimpleStringSchema(), properties);  
        DataStream<String> stream = env.addSource(consumer);
```

4. flink业务处理，这一块由于所处的业务不同，我只是简单demo以下，以20%的概率修改数据使之成为异常数据用于检测，这是为了模拟业务中可能对数据处理有误而发生数据质量问题。这里要特别提一下，本案例是假定flink业务处理时延忽略不计，真实场景中可能由于flink处理延迟导致target端误认为数据丢失，这一部分我还在研究他的源码，日后更新，有了解的大神，还请指点迷津。

```
//使用Flink算子简单处理数据  
        // Transformations  
        // 使用Flink算子对输入流的文本进行操作  
        // 按空格切词、计数、分区、设置时间窗口、聚合  
        //{"id": 1, "name": "Apple", "color": "COLOR", "time": "TIME"}  
        DataStream<String> outMap = stream.map(new MapFunction<String, String>() {  
            @Override  
            public String map(String value) throws Exception {  
                return handleData(value);  
            }  
        });
```

```
public static String handleData(String line){  
        try {  
                if (line!=null&& !line.equals("")){  
                    Gson gson = new GsonBuilder().setLenient().setDateFormat("yyyy-MM-dd_HH:mm:ss").create();  
                    JsonReader reader = new JsonReader(new StringReader(line));  
                    Student student = gson.fromJson(reader, Student.class);  
                    int rand = ra.nextInt(10) + 1;  
                    if (rand > 8) student.setName(student.getName() + "_" + ra.nextInt(10));  
                    return gson.toJson(student);  
                }  
                else return "";  
        }catch (Exception e){  
            return "";  
        }  
    }
```

因为遇到了几个bug，所以这样创建gson

5. 写入kafka，其中FlinkKafkaProducer010我们选择的构造器是（brokerList，topicId，serializationSchema）

```
//Sink  
        outMap.addSink(new FlinkKafkaProducer010<String>(  
                "hadoop101:9092",  
                "target",  
                new SimpleStringSchema()  
        ));  
        outMap.print();  
        env.execute();
```

## 四. Apache Griffin配置与启动

有关griffin的streaming模式配置，就是配置dq.json和env.json

dq.json

```
{  
  "name": "streaming_accu",  
  "process.type": "streaming",  
  "data.sources": [  
    {  
      "name": "src",  
      "baseline": true,  
      "connector":   
        {  
          "type": "kafka",  
          "version": "0.10",  
          "config": {  
            "kafka.config": {  
              "bootstrap.servers": "hadoop101:9092",  
              "group.id": "griffin",  
              "auto.offset.reset": "largest",  
              "auto.commit.enable": "false"  
            },  
            "topics": "source_1",  
            "key.type": "java.lang.String",  
            "value.type": "java.lang.String"  
          },  
          "pre.proc": [  
            {  
              "dsl.type": "df-opr",  
              "rule": "from_json"  
            }  
          ]  
        }  
      ,  
      "checkpoint": {  
        "type": "json",  
        "file.path": "hdfs://hadoop101:9000/griffin/streaming/dump/source",  
        "info.path": "source_1",  
        "ready.time.interval": "10s",  
        "ready.time.delay": "0",  
        "time.range": ["-5m", "0"],  
        "updatable": true  
      }  
    }, {  
      "name": "tgt",  
      "connector":   
        {  
          "type": "kafka",  
          "version": "0.10",  
          "config": {  
            "kafka.config": {  
              "bootstrap.servers": "hadoop101:9092",  
              "group.id": "griffin",  
              "auto.offset.reset": "largest",  
              "auto.commit.enable": "false"  
            },  
            "topics": "target_1",  
            "key.type": "java.lang.String",  
            "value.type": "java.lang.String"  
          },  
          "pre.proc": [  
            {  
              "dsl.type": "df-opr",  
              "rule": "from_json"  
            }  
          ]  
        }  
      ,  
      "checkpoint": {  
        "type": "json",  
        "file.path": "hdfs://hadoop101:9000/griffin/streaming/dump/target",  
        "info.path": "target_1",  
        "ready.time.interval": "10s",  
        "ready.time.delay": "0",  
        "time.range": ["-1m", "0"]  
      }  
    }  
  ],  
  "evaluate.rule": {  
    "rules": [  
      {  
        "dsl.type": "griffin-dsl",  
        "dq.type": "accuracy",  
        "out.dataframe.name": "accu",  
        "rule": "src.login_id = tgt.login_id AND src.bussiness_id = tgt.bussiness_id AND src.event_id = tgt.event_id",  
        "details": {  
          "source": "src",  
          "target": "tgt",  
          "miss": "miss_count",  
          "total": "total_count",  
          "matched": "matched_count"  
        },  
        "out":[  
          {  
            "type":"metric",  
            "name": "accu"  
          },  
          {  
            "type":"record",  
            "name": "missRecords"  
          }  
        ]  
      }  
    ]  
  },  
  "sinks": ["HdfsSink"]  
}
```

env.json

```
{  
  "spark": {  
    "log.level": "WARN",  
    "checkpoint.dir": "hdfs://hadoop101:9000/griffin/checkpoint",  
    "batch.interval": "20s",  
    "process.interval": "1m",  
    "init.clear": true,  
    "config": {  
      "spark.default.parallelism": 4,  
      "spark.task.maxFailures": 5,  
      "spark.streaming.kafkaMaxRatePerPartition": 1000,  
      "spark.streaming.concurrentJobs": 4,  
      "spark.yarn.maxAppAttempts": 5,  
      "spark.yarn.am.attemptFailuresValidityInterval": "1h",  
      "spark.yarn.max.executor.failures": 120,  
      "spark.yarn.executor.failuresValidityInterval": "1h",  
      "spark.hadoop.fs.hdfs.impl.disable.cache": true  
    }  
  },  
  "sinks": [  
    {  
      "name":"ConsoleSink",  
      "type": "console"  
    },  
    {  
      "name":"HdfsSink",  
      "type": "hdfs",  
      "config": {  
        "path": "hdfs://hadoop101:9000/griffin/persist"  
      }  
    },  
    {  
      "name":"ElasticsearchSink",  
      "type": "elasticsearch",  
      "config": {  
        "method": "post",  
        "api": "http://hadoop101:9200/griffin/accuracy"  
      }  
    }  
  ],  
  "griffin.checkpoint": [  
    {  
      "type": "zk",  
      "config": {  
        "hosts": "hadoop101:2181",  
        "namespace": "griffin/infocache",  
        "lock.path": "lock",  
        "mode": "persist",  
        "init.clear": true,  
        "close.clear": false  
      }  
    }  
  ]  
}
```

最后把项目提交到spark上运行，检测数据

```
spark-submit --class org.apache.griffin.measure.Application --master yarn --deploy-mode client --queue default \  
--driver-memory 1g --executor-memory 1g --num-executors 3 \  
<path>/griffin-measure.jar \  
<path>/env.json <path>/dq.json
```

## 五. 全局代码

在本地创建个maven项目，由于这是个简单的测试项目，自己构建就好，我只写了两个类做测试

Student.class

```
import java.util.Date;  
  
public class Student{  
    private int id;  
    private String name;  
    private String color;  
    private Date time;  
  
    public Student(){}  
  
    public Student(int id, String name, String color, Date time) {  
        this.id = id;  
        this.name = name;  
        this.color = color;  
        this.time = time;  
    }  
  
    public int getId() {  
        return id;  
    }  
  
    public void setId(int id) {  
        this.id = id;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
  
    public String getColor() {  
        return color;  
    }  
  
    public void setColor(String color) {  
        this.color = color;  
    }  
  
    public Date getTime() {  
        return time;  
    }  
  
    public void setTime(Date time) {  
        this.time = time;  
    }  
  
    @Override  
    public String toString() {  
        return "Student{" +  
                "id=" + id +  
                ", name='" + name + '\'' +  
                ", color='" + color + '\'' +  
                ", time='" + time + '\'' +  
                '}';  
    }  
}
```

flinkProcess.class

```
import com.google.gson.Gson;  
import com.google.gson.GsonBuilder;  
import com.google.gson.stream.JsonReader;  
import org.apache.flink.api.common.functions.MapFunction;  
import org.apache.flink.streaming.api.datastream.DataStream;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer010;  
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer010;  
import org.apache.flink.streaming.util.serialization.SimpleStringSchema;  
  
import java.io.StringReader;  
import java.util.Properties;  
import java.util.Random;  
  
public class flinkProcess {  
    public static Random ra = new Random();  
    public static void main(String[] args) throws Exception {  
        // 创建Flink执行环境  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        // 一定要设置启动检查点！！  
        //env.enableCheckpointing(5000);  
        //env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);  
        env.setParallelism(1);  
  
        // Kafka参数  
        Properties properties = new Properties();  
        properties.setProperty("bootstrap.servers", "hadoop101:9092");  
        properties.setProperty("group.id", "consumer-group");  
        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");  
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");  
        properties.setProperty("auto.offset.reset", "latest");  
        String inputTopic = "source";  
        String outputTopic = "target";  
  
        // Source  
        FlinkKafkaConsumer010<String> consumer =  
                new FlinkKafkaConsumer010<String>(inputTopic, new SimpleStringSchema(), properties);  
        DataStream<String> stream = env.addSource(consumer);  
  
        //使用Flink算子简单处理数据  
        // Transformations  
        // 使用Flink算子对输入流的文本进行操作  
        // 按空格切词、计数、分区、设置时间窗口、聚合  
        //{"id": 1, "name": "Apple", "color": "COLOR", "time": "TIME"}  
        DataStream<String> outMap = stream.map(new MapFunction<String, String>() {  
            @Override  
            public String map(String value) throws Exception {  
                return handleData(value);  
            }  
        });  
  
        //Sink  
        outMap.addSink(new FlinkKafkaProducer010<String>(  
                "hadoop101:9092",  
                "target",  
                new SimpleStringSchema()  
        ));  
        outMap.print();  
        env.execute();  
    }  
  
    public static String handleData(String line){  
        try {  
                if (line!=null&& !line.equals("")){  
                    Gson gson = new GsonBuilder().setLenient().setDateFormat("yyyy-MM-dd_HH:mm:ss").create();  
                    JsonReader reader = new JsonReader(new StringReader(line));  
                    Student student = gson.fromJson(reader, Student.class);  
                    int rand = ra.nextInt(10) + 1;  
                    if (rand > 8) student.setName(student.getName() + "_" + ra.nextInt(10));  
                    return gson.toJson(student);  
                }  
                else return "";  
        }catch (Exception e){  
            return "";  
        }  
    }  
}
```

提示：在kafka中如果生成了一些不合格式的数据，程序会一直报错，可以参考这篇文章删除掉相应的kafka dataDir和zookeeper的znode数据，重新生成数据，运行代码。

如果这个文章对你有帮助，不要忘记 **「在看」** **「点赞」** **「收藏」** 三连啊喂！

[2022年全网首发|大数据专家级技能模型与学习指南(胜天半子篇)](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247510040&idx=1&sn=aa335f25965975731173916f012d56f4&chksm=fd3eee8dca49679b82f632048fb64d21fac01497d1a0fe33917edb01e194caf0f9a1930a55ce&scene=21#wechat_redirect)

[互联网最坏的时代可能真的来了](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247508317&idx=1&sn=0bcb7fb6997b42306994b890eaa0d47f&chksm=fd3ee7c8ca496ede347a2971002c6ea68dcfd70abeeb90b43c8b02a2a60ebc7cec3a87ef7d52&scene=21#wechat_redirect)

[我在B站读大学，大数据专业](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247507860&idx=1&sn=807ac5003762f29c127bc4071dcebe33&chksm=fd3e9901ca4910178cfc816043ea86c29f881f70cd943d252de9ae2e21cb7e8ba80bff162e1b&scene=21#wechat_redirect)

[我们在学习Flink的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247499604&idx=1&sn=d938dfb30d221774704982d2938b30c1&chksm=fd3eb9c1ca4930d76a391241333de461ca22d2aa27472eab3cffab1564872ae37f1b48fe2d3c&scene=21#wechat_redirect)

[193篇文章暴揍Flink，这个合集你需要关注一下](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504856&idx=2&sn=6f62e2a0c756ce56773eed253d2f41ee&chksm=fd3e954dca491c5b732b7e2aa46db32efbc3f690e528522098659bb3c6e78e1582ab4529b5dc&scene=21#wechat_redirect)

[Flink生产环境TOP难题与优化，阿里巴巴藏经阁YYDS](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504742&idx=1&sn=8765c198d8ad66219a7bcabb221e4a23&chksm=fd3e95f3ca491ce52d0724b9e4154a47af0f5ea349e1e184c8fbc65aa17c0000242aa9b58429&scene=21#wechat_redirect)

[Flink CDC我吃定了耶稣也留不住他！| Flink CDC线上问题小盘点](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504813&idx=1&sn=f5cd6ae2aa2b1e30f87a5ae55971c514&chksm=fd3e9538ca491c2eb191677d070f2c7e4f1098eece00e256d6205b8a5d21b21c1e74f875f16f&scene=21#wechat_redirect)

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