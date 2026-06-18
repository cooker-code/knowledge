---
title: Flink CDC实现数据增量备份到ClickHouse实战
author: 大数据技术与架构
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247512051&idx=1&sn=801727f8dd655375581cbc90377556ce&chksm=fd3ee966ca496070903be739fad565796bef031baf1c1e2f1afd586295a4af8335d0588eed0f&mpshare=1&scene=24&srcid=0315N4AUgEkKShpxZHulswnJ&sharer_sharetime=1647351313890&sharer_shareid=4145ee29355a80b1b2b01921ded70e3a#rd
---

点击上方**蓝色字体**，选择“设为星标”

回复"**面试"**获取更多惊喜

[**八股文教给我，你们专心刷题和面试**](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247512011&idx=1&sn=68aaa9c5c42e2087d56d170c442b30ba&chksm=fd3ee95eca496048b24a717a30fe820b9188d02e014ba7b554afe86f8da1efab189799fe7087&scene=21#wechat_redirect)

> Hi，我是王知无，一个大数据领域的原创作者。
>
> 放心关注我，获取更多行业的一手消息。

本文我们首先来介绍什么是CDC，以及CDC工具选型，接下来我们来介绍如何通过Flink CDC抓取mysql中的数据，并把他汇入Clickhouse里，最后我们还将介绍Flink SQL CDC的方式。

# CDC

首先什么是CDC ？它是Change Data Capture的缩写,即变更数据捕捉的简称，使用CDC我们可以从数据库中获取已提交的更改并将这些更改发送到下游，供下游使用。这些变更可以包括INSERT,DELETE,UPDATE等操作。

其主要的应用场景：

* 异构数据库之间的数据同步或备份 / 建立数据分析计算平台
* 微服务之间共享数据状态
* 更新缓存 / CQRS 的 Query 视图更新

CDC 它是一个比较广义的概念，只要能捕获变更的数据，我们都可以称为 CDC 。业界主要有基于查询的 CDC 和基于日志的 CDC ，可以从下面表格对比他们功能和差异点。

|  | 基于查询的 CDC | 基于日志的 CDC |
| --- | --- | --- |
| 概念 | 每次捕获变更发起 Select 查询进行全表扫描，过滤出查询之间变更的数据 | 读取数据存储系统的 log ，例如 MySQL 里面的 binlog持续监控 |
| 开源产品 | Sqoop, Kafka JDBC Source | Canal, Maxwell, Debezium |
| 执行模式 | Batch | Streaming |
| 捕获所有数据的变化 | ❌ | ✅ |
| 低延迟，不增加数据库负载 | ❌ | ✅ |
| 不侵入业务（LastUpdated字段） | ❌ | ✅ |
| 捕获删除事件和旧记录的状态 | ❌ | ✅ |
| 捕获旧记录的状态 | ❌ | ✅ |

# Debezium

Debezium是一个开源项目，为捕获数据更改(change data capture,CDC)提供了一个低延迟的流式处理平台。你可以安装并且配置Debezium去监控你的数据库，然后你的应用就可以消费对数据库的每一个行级别(row-level)的更改。只有已提交的更改才是可见的，所以你的应用不用担心事务(transaction)或者更改被回滚(roll back)。Debezium为所有的数据库更改事件提供了一个统一的模型，所以你的应用不用担心每一种数据库管理系统的错综复杂性。另外，由于Debezium用持久化的、有副本备份的日志来记录数据库数据变化的历史，因此，你的应用可以随时停止再重启，而不会错过它停止运行时发生的事件，保证了所有的事件都能被正确地、完全地处理掉。

> Debezium is an open source distributed platform for change data capture. Start it up, point it at your databases, and your apps can start responding to all of the inserts, updates, and deletes that other apps commit to your databases. Debezium is durable and fast, so your apps can respond quickly and never miss an event, even when things go wrong

# ClickHouse

实时数据分析数据库，俄罗斯的谷歌开发的，推荐OLAP场景使用

**Clickhouse的优点.**

1. 真正的面向列的 DBMS

   ClickHouse 是一个 DBMS，而不是一个单一的数据库。它允许在运行时创建表和数据库、加载数据和运行

   查询，而无需重新配置和重新启动服务器。
2. 数据压缩

   一些面向列的 DBMS（InfiniDB CE 和 MonetDB）不使用数据压缩。但是，数据压缩确实提高了性能。
3. 磁盘存储的数据
4. 在多个服务器上分布式处理
5. SQL支持
6. 数据不仅按列存储，而且由矢量 - 列的部分进行处理，这使开发者能够实现高 CPU 性能

**Clickhouse的缺点**

1. 没有完整的事务支持，
2. 缺少完整的Update/Delete操作，缺少高频率、低延迟的修改或删除已存在数据的能力，仅能用于批量删

   除或修改数据
3. 聚合结果必须小于一台机器的内存大小：
4. 不适合key-value存储，

**什么时候不可以用Clickhouse?**

1. 事物性工作（OLTP）
2. 高并发的键值访问
3. Blob或者文档存储
4. 超标准化的数据

# Flink CDC

Flink cdc connector 消费 Debezium 里的数据，经过处理再sink出来，这个流程还是相对比较简单的

首先创建 Source 和 Sink（对应的依赖引用，在文末）

```
       SourceFunction<String> sourceFunction = MySQLSource.<String>builder()  
                .hostname("localhost")  
                .port(3306)  
                .databaseList("test")  
                .username("flinkcdc")  
                .password("dafei1288")  
                .deserializer(new JsonDebeziumDeserializationSchema())  
                .build();  
  
        // 添加 source  
        env.addSource(sourceFunction)  
        // 添加 sink  
        .addSink(new ClickhouseSink());
```

这里用到的JsonDebeziumDeserializationSchema，是我们自定义的一个序列化类，用于将Debezium输出的数据，序列化

```
// 将cdc数据反序列化  
    public static class JsonDebeziumDeserializationSchema implements DebeziumDeserializationSchema {  
        @Override  
        public void deserialize(SourceRecord sourceRecord, Collector collector) throws Exception {  
  
            Gson jsstr = new Gson();  
            HashMap<String, Object> hs = new HashMap<>();  
  
            String topic = sourceRecord.topic();  
            String[] split = topic.split("[.]");  
            String database = split[1];  
            String table = split[2];  
            hs.put("database",database);  
            hs.put("table",table);  
            //获取操作类型  
            Envelope.Operation operation = Envelope.operationFor(sourceRecord);  
            //获取数据本身  
            Struct struct = (Struct)sourceRecord.value();  
            Struct after = struct.getStruct("after");  
  
            if (after != null) {  
                Schema schema = after.schema();  
                HashMap<String, Object> afhs = new HashMap<>();  
                for (Field field : schema.fields()) {  
                    afhs.put(field.name(), after.get(field.name()));  
                }  
                hs.put("data",afhs);  
            }  
  
            String type = operation.toString().toLowerCase();  
            if ("create".equals(type)) {  
                type = "insert";  
            }  
            hs.put("type",type);  
  
            collector.collect(jsstr.toJson(hs));  
        }  
  
        @Override  
        public TypeInformation<String> getProducedType() {  
            return BasicTypeInfo.STRING_TYPE_INFO;  
        }  
    }
```

这里是将数据序列化成如下Json格式

```
{"database":"test","data":{"name":"jacky","description":"fffff","id":8},"type":"insert","table":"test_cdc"}
```

接下来就是要创建Sink，将数据变化存入Clickhouse中，这里我们仅以insert为例

```
public static class ClickhouseSink extends RichSinkFunction<String>{  
        Connection connection;  
        PreparedStatement pstmt;  
        private Connection getConnection() {  
            Connection conn = null;  
            try {  
                Class.forName("ru.yandex.clickhouse.ClickHouseDriver");  
                String url = "jdbc:clickhouse://localhost:8123/default";  
                conn = DriverManager.getConnection(url,"default","dafei1288");  
  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
            return conn;  
        }  
  
        @Override  
        public void open(Configuration parameters) throws Exception {  
            super.open(parameters);  
            connection = getConnection();  
            String sql = "insert into sink_ch_test(id,name,description) values (?,?,?)";  
            pstmt = connection.prepareStatement(sql);  
        }  
  
        // 每条记录插入时调用一次  
        public void invoke(String value, Context context) throws Exception {  
            //{"database":"test","data":{"name":"jacky","description":"fffff","id":8},"type":"insert","table":"test_cdc"}  
            Gson t = new Gson();  
            HashMap<String,Object> hs = t.fromJson(value,HashMap.class);  
            String database = (String)hs.get("database");  
            String table = (String)hs.get("table");  
            String type = (String)hs.get("type");  
  
            if("test".equals(database) && "test_cdc".equals(table)){  
                if("insert".equals(type)){  
                    System.out.println("insert => "+value);  
                    LinkedTreeMap<String,Object> data = (LinkedTreeMap<String,Object>)hs.get("data");  
                    String name = (String)data.get("name");  
                    String description = (String)data.get("description");  
                    Double id = (Double)data.get("id");  
                    // 未前面的占位符赋值  
                    pstmt.setInt(1, id.intValue());  
                    pstmt.setString(2, name);  
                    pstmt.setString(3, description);  
  
                    pstmt.executeUpdate();  
                }  
            }  
        }  
  
        @Override  
        public void close() throws Exception {  
            super.close();  
  
            if(pstmt != null) {  
                pstmt.close();  
            }  
  
            if(connection != null) {  
                connection.close();  
            }  
        }  
    }
```

完整代码案例：

```
package name.lijiaqi.cdc;  
  
import com.alibaba.ververica.cdc.debezium.DebeziumDeserializationSchema;  
import com.google.gson.Gson;  
import com.google.gson.internal.LinkedTreeMap;  
import io.debezium.data.Envelope;  
import org.apache.flink.api.common.typeinfo.BasicTypeInfo;  
import org.apache.flink.api.common.typeinfo.TypeInformation;  
import org.apache.flink.configuration.Configuration;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.functions.sink.RichSinkFunction;  
import org.apache.flink.streaming.api.functions.source.SourceFunction;  
import com.alibaba.ververica.cdc.connectors.mysql.MySQLSource;  
import org.apache.flink.util.Collector;  
import org.apache.kafka.connect.source.SourceRecord;  
  
import org.apache.kafka.connect.data.Field;  
import org.apache.kafka.connect.data.Schema;  
import org.apache.kafka.connect.data.Struct;  
  
import java.sql.Connection;  
import java.sql.DriverManager;  
import java.sql.PreparedStatement;  
import java.util.HashMap;  
  
public class MySqlBinlogSourceExample {  
    public static void main(String[] args) throws Exception {  
        SourceFunction<String> sourceFunction = MySQLSource.<String>builder()  
                .hostname("localhost")  
                .port(3306)  
                .databaseList("test")  
                .username("flinkcdc")  
                .password("dafei1288")  
                .deserializer(new JsonDebeziumDeserializationSchema())  
                .build();  
  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
  
        // 添加 source  
        env.addSource(sourceFunction)  
        // 添加 sink  
        .addSink(new ClickhouseSink());  
  
        env.execute("mysql2clickhouse");  
    }  
  
    // 将cdc数据反序列化  
    public static class JsonDebeziumDeserializationSchema implements DebeziumDeserializationSchema {  
        @Override  
        public void deserialize(SourceRecord sourceRecord, Collector collector) throws Exception {  
  
            Gson jsstr = new Gson();  
            HashMap<String, Object> hs = new HashMap<>();  
  
            String topic = sourceRecord.topic();  
            String[] split = topic.split("[.]");  
            String database = split[1];  
            String table = split[2];  
            hs.put("database",database);  
            hs.put("table",table);  
            //获取操作类型  
            Envelope.Operation operation = Envelope.operationFor(sourceRecord);  
            //获取数据本身  
            Struct struct = (Struct)sourceRecord.value();  
            Struct after = struct.getStruct("after");  
  
            if (after != null) {  
                Schema schema = after.schema();  
                HashMap<String, Object> afhs = new HashMap<>();  
                for (Field field : schema.fields()) {  
                    afhs.put(field.name(), after.get(field.name()));  
                }  
                hs.put("data",afhs);  
            }  
  
            String type = operation.toString().toLowerCase();  
            if ("create".equals(type)) {  
                type = "insert";  
            }  
            hs.put("type",type);  
  
            collector.collect(jsstr.toJson(hs));  
        }  
  
        @Override  
        public TypeInformation<String> getProducedType() {  
            return BasicTypeInfo.STRING_TYPE_INFO;  
        }  
    }  
  
  
    public static class ClickhouseSink extends RichSinkFunction<String>{  
        Connection connection;  
        PreparedStatement pstmt;  
        private Connection getConnection() {  
            Connection conn = null;  
            try {  
                Class.forName("ru.yandex.clickhouse.ClickHouseDriver");  
                String url = "jdbc:clickhouse://localhost:8123/default";  
                conn = DriverManager.getConnection(url,"default","dafei1288");  
  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
            return conn;  
        }  
  
        @Override  
        public void open(Configuration parameters) throws Exception {  
            super.open(parameters);  
            connection = getConnection();  
            String sql = "insert into sink_ch_test(id,name,description) values (?,?,?)";  
            pstmt = connection.prepareStatement(sql);  
        }  
  
        // 每条记录插入时调用一次  
        public void invoke(String value, Context context) throws Exception {  
            //{"database":"test","data":{"name":"jacky","description":"fffff","id":8},"type":"insert","table":"test_cdc"}  
            Gson t = new Gson();  
            HashMap<String,Object> hs = t.fromJson(value,HashMap.class);  
            String database = (String)hs.get("database");  
            String table = (String)hs.get("table");  
            String type = (String)hs.get("type");  
  
            if("test".equals(database) && "test_cdc".equals(table)){  
                if("insert".equals(type)){  
                    System.out.println("insert => "+value);  
                    LinkedTreeMap<String,Object> data = (LinkedTreeMap<String,Object>)hs.get("data");  
                    String name = (String)data.get("name");  
                    String description = (String)data.get("description");  
                    Double id = (Double)data.get("id");  
                    // 未前面的占位符赋值  
                    pstmt.setInt(1, id.intValue());  
                    pstmt.setString(2, name);  
                    pstmt.setString(3, description);  
  
                    pstmt.executeUpdate();  
                }  
            }  
        }  
  
        @Override  
        public void close() throws Exception {  
            super.close();  
  
            if(pstmt != null) {  
                pstmt.close();  
            }  
  
            if(connection != null) {  
                connection.close();  
            }  
        }  
    }  
}
```

执行查看结果

数据成功汇入

# Flink SQL CDC

接下来，我们看一下如何通过Flink SQL实现CDC ，只需3条SQL语句即可。

创建数据源表

```
        // 数据源表  
        String sourceDDL =  
                "CREATE TABLE mysql_binlog (\n" +  
                        " id INT NOT NULL,\n" +  
                        " name STRING,\n" +  
                        " description STRING\n" +  
                        ") WITH (\n" +  
                        " 'connector' = 'mysql-cdc',\n" +  
                        " 'hostname' = 'localhost',\n" +  
                        " 'port' = '3306',\n" +  
                        " 'username' = 'flinkcdc',\n" +  
                        " 'password' = 'dafei1288',\n" +  
                        " 'database-name' = 'test',\n" +  
                        " 'table-name' = 'test_cdc'\n" +  
                        ")";
```

创建输出表

```
 // 输出目标表  
        String sinkDDL =  
                "CREATE TABLE test_cdc_sink (\n" +  
                        " id INT NOT NULL,\n" +  
                        " name STRING,\n" +  
                        " description STRING,\n" +  
                        " PRIMARY KEY (id) NOT ENFORCED \n " +  
                        ") WITH (\n" +  
                        " 'connector' = 'jdbc',\n" +  
                        " 'driver' = 'com.mysql.jdbc.Driver',\n" +  
                        " 'url' = '" + url + "',\n" +  
                        " 'username' = '" + userName + "',\n" +  
                        " 'password' = '" + password + "',\n" +  
                        " 'table-name' = '" + mysqlSinkTable + "'\n" +  
                        ")";
```

这里我们直接将数据汇入

```
// 简单的聚合处理  
        String transformSQL =  
                "insert into test_cdc_sink select * from mysql_binlog";
```

完整参考代码

```
package name.lijiaqi.cdc;  
  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.table.api.EnvironmentSettings;  
import org.apache.flink.table.api.SqlDialect;  
import org.apache.flink.table.api.TableResult;  
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;  
  
public class MysqlToMysqlMain {  
    public static void main(String[] args) throws Exception {  
        EnvironmentSettings fsSettings = EnvironmentSettings.newInstance()  
                .useBlinkPlanner()  
                .inStreamingMode()  
                .build();  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env, fsSettings);  
  
  
  
        tableEnv.getConfig().setSqlDialect(SqlDialect.DEFAULT);  
  
  
        // 数据源表  
        String sourceDDL =  
                "CREATE TABLE mysql_binlog (\n" +  
                        " id INT NOT NULL,\n" +  
                        " name STRING,\n" +  
                        " description STRING\n" +  
                        ") WITH (\n" +  
                        " 'connector' = 'mysql-cdc',\n" +  
                        " 'hostname' = 'localhost',\n" +  
                        " 'port' = '3306',\n" +  
                        " 'username' = 'flinkcdc',\n" +  
                        " 'password' = 'dafei1288',\n" +  
                        " 'database-name' = 'test',\n" +  
                        " 'table-name' = 'test_cdc'\n" +  
                        ")";  
  
  
        String url = "jdbc:mysql://127.0.0.1:3306/test";  
        String userName = "root";  
        String password = "dafei1288";  
        String mysqlSinkTable = "test_cdc_sink";  
        // 输出目标表  
        String sinkDDL =  
                "CREATE TABLE test_cdc_sink (\n" +  
                        " id INT NOT NULL,\n" +  
                        " name STRING,\n" +  
                        " description STRING,\n" +  
                        " PRIMARY KEY (id) NOT ENFORCED \n " +  
                        ") WITH (\n" +  
                        " 'connector' = 'jdbc',\n" +  
                        " 'driver' = 'com.mysql.jdbc.Driver',\n" +  
                        " 'url' = '" + url + "',\n" +  
                        " 'username' = '" + userName + "',\n" +  
                        " 'password' = '" + password + "',\n" +  
                        " 'table-name' = '" + mysqlSinkTable + "'\n" +  
                        ")";  
        // 简单的聚合处理  
        String transformSQL =  
                "insert into test_cdc_sink select * from mysql_binlog";  
  
        tableEnv.executeSql(sourceDDL);  
        tableEnv.executeSql(sinkDDL);  
        TableResult result = tableEnv.executeSql(transformSQL);  
  
        // 等待flink-cdc完成快照  
        result.print();  
        env.execute("sync-flink-cdc");  
    }  
  
}
```

查看执行结果

添加依赖

```
    <dependencies>  
        <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-core -->  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-core</artifactId>  
            <version>1.13.0</version>  
        </dependency>  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-streaming-java_2.12</artifactId>  
            <version>1.13.0</version>  
        </dependency>  
  
<!--        <dependency>-->  
<!--            <groupId>org.apache.flink</groupId>-->  
<!--            <artifactId>flink-jdbc_2.12</artifactId>-->  
<!--            <version>1.10.3</version>-->  
<!--        </dependency>-->  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-connector-jdbc_2.12</artifactId>  
            <version>1.13.0</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-java</artifactId>  
            <version>1.13.0</version>  
        </dependency>  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-clients_2.12</artifactId>  
            <version>1.13.0</version>  
        </dependency>  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-table-api-java-bridge_2.12</artifactId>  
            <version>1.13.0</version>  
        </dependency>  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-table-common</artifactId>  
            <version>1.13.0</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-table-planner_2.12</artifactId>  
            <version>1.13.0</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-table-planner-blink_2.12</artifactId>  
            <version>1.13.0</version>  
        </dependency>  
        <dependency>  
            <groupId>org.apache.flink</groupId>  
            <artifactId>flink-table-planner-blink_2.12</artifactId>  
            <version>1.13.0</version>  
            <type>test-jar</type>  
        </dependency>  
  
        <dependency>  
            <groupId>com.alibaba.ververica</groupId>  
            <artifactId>flink-connector-mysql-cdc</artifactId>  
            <version>1.4.0</version>  
        </dependency>  
  
  
        <dependency>  
            <groupId>com.aliyun</groupId>  
            <artifactId>flink-connector-clickhouse</artifactId>  
            <version>1.12.0</version>  
        </dependency>  
        <dependency>  
            <groupId>ru.yandex.clickhouse</groupId>  
            <artifactId>clickhouse-jdbc</artifactId>  
            <version>0.2.6</version>  
        </dependency>  
        <dependency>  
            <groupId>com.google.code.gson</groupId>  
            <artifactId>gson</artifactId>  
            <version>2.8.6</version>  
        </dependency>  
    </dependencies>
```

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