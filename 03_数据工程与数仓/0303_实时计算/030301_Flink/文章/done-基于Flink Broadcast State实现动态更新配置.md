> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkBroadcastState动态配置更新|FlinkBroadcastState动态配置更新]]
---
title: 基于Flink Broadcast State实现动态更新配置
author: 大数据技术与架构
date:
url: http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247515329&idx=1&sn=b0ef02abf3ec13137cca132c2be27516&chksm=fd3efa54ca497342380bac423e31c6e66be7fa5bde2def4e5aea201ee576b445f52cb7e4d367&mpshare=1&scene=24&srcid=0824vWG7NAJvpLBT6HmbjBkX&sharer_sharetime=1661302530287&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

> **[**全网最全大数据面试提升手册！**](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247514568&idx=1&sn=659d7464a3b61ebf8397d2597069f4ab&chksm=fd3eff5dca49764beef9ab14400e96715b86b3394f05e161181c03424fa579cdbc242a401411&scene=21#wechat_redirect)**

Flink从1.5.0开始支持广播状态(Broadcast State)。广播状态可以用来解决如下问题:一条流需要根据规则或配置处理数据，而规则或配置又是随时变化的。此时，就可将规则或配置作为广播流广播出去，并以Broadcast State的形式存储在下游Task中。下游Task根据Broadcast State中的规则或配置来处理常规流中的数据。

场景举例:

1. 动态更新计算规则: 如事件流需要根据最新的规则进行计算，则可将规则作为广播状态广播到下游Task中。
2. 实时增加额外字段: 如事件流需要实时增加用户的基础信息，则可将用户的基础信息作为广播状态广播到下游Task中。

注意:

1. Broadcast State是Map类型，即K-V类型。
2. Broadcast State只有在广播的一侧,即在BroadcastProcessFunction或KeyedBroadcastProcessFunction的processBroadcastElement方法中可以修改。在非广播的一侧，即在BroadcastProcessFunction或KeyedBroadcastProcessFunction的processElement方法中只读。
3. Broadcast State中元素的顺序，在各Task中可能不同。基于顺序的处理，需要注意。
4. Broadcast State在Checkpoint时，每个Task都会Checkpoint广播状态。
5. Broadcast State在运行时保存在内存中，目前还不能保存在RocksDB State Backend中。

假设我们现在有这样的需求：基于Broadcast State 动态更新配置以实现实时过滤数据并增加字段。

### 需求背景

假设有这样的一个需求，需要实时过滤出配置中的用户，并在事件流中补全这批用户的基础信息。

1. 事件: 在Kafka中，自己造的数据，格式如下。

```
# 某个用户在某个时刻浏览或点击了某个商品
{"userID": "user_3", "eventTime": "2019-08-17 12:19:47", "eventType": "browse", "productID": 1}
{"userID": "user_2", "eventTime": "2019-08-17 12:19:48", "eventType": "click", "productID": 1}
```

2. 配置: 在Mysql中，自己造的数据，表结构如下。

```
# 用户的基础信息
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| userID   | varchar(20) | NO   | PRI | NULL    |       |
| userName | varchar(10) | YES  |     | NULL    |       |
| userAge  | int(11)     | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
```

部分依赖

```
<!--Kafka连接器-->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.10_2.11</artifactId>
    <version>1.8.0</version>
</dependency>

<!--Mysql JDBC Driver-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
</dependency>
```

代码实现

```
package com.bigdata.flink;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import lombok.extern.slf4j.Slf4j;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.common.state.BroadcastState;
import org.apache.flink.api.common.state.MapStateDescriptor;
import org.apache.flink.api.common.state.ReadOnlyBroadcastState;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.*;
import org.apache.flink.api.java.utils.ParameterTool;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.runtime.state.StateBackend;
import org.apache.flink.runtime.state.filesystem.FsStateBackend;
import org.apache.flink.shaded.guava18.com.google.common.collect.Maps;
import org.apache.flink.streaming.api.CheckpointingMode;
import org.apache.flink.streaming.api.datastream.*;
import org.apache.flink.streaming.api.environment.CheckpointConfig;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.streaming.api.functions.co.BroadcastProcessFunction;
import org.apache.flink.streaming.api.functions.source.RichSourceFunction;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer010;
import org.apache.flink.util.Collector;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

/**
 * Summary:
 *   基于Broadcast State 动态更新配置以实现实时过滤数据并增加字段
 */
@Slf4j
public class TestBroadcastState {
    public static void main(String[] args) throws Exception{

        //1、解析命令行参数
        ParameterTool fromArgs = ParameterTool.fromArgs(args);
        ParameterTool parameterTool = ParameterTool.fromPropertiesFile(fromArgs.getRequired("applicationProperties"));

        //checkpoint配置
        String checkpointDirectory = parameterTool.getRequired("checkpointDirectory");
        long checkpointSecondInterval = parameterTool.getLong("checkpointSecondInterval");

        //事件流配置
        String fromKafkaBootstrapServers = parameterTool.getRequired("fromKafka.bootstrap.servers");
        String fromKafkaGroupID = parameterTool.getRequired("fromKafka.group.id");
        String fromKafkaTopic = parameterTool.getRequired("fromKafka.topic");

        //配置流配置
        String fromMysqlHost = parameterTool.getRequired("fromMysql.host");
        int fromMysqlPort = parameterTool.getInt("fromMysql.port");
        String fromMysqlDB = parameterTool.getRequired("fromMysql.db");
        String fromMysqlUser = parameterTool.getRequired("fromMysql.user");
        String fromMysqlPasswd = parameterTool.getRequired("fromMysql.passwd");
        int fromMysqlSecondInterval = parameterTool.getInt("fromMysql.secondInterval");

        //2、配置运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //设置StateBackend
        env.setStateBackend((StateBackend) new FsStateBackend(checkpointDirectory, true));
        //设置Checkpoint
        CheckpointConfig checkpointConfig = env.getCheckpointConfig();
        checkpointConfig.setCheckpointInterval(checkpointSecondInterval * 1000);
        checkpointConfig.setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        checkpointConfig.enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);

        //3、Kafka事件流
        //从Kafka中获取事件数据
        //数据：某个用户在某个时刻浏览或点击了某个商品,如
        //{"userID": "user_3", "eventTime": "2019-08-17 12:19:47", "eventType": "browse", "productID": 1}
        Properties kafkaProperties = new Properties();
        kafkaProperties.put("bootstrap.servers",fromKafkaBootstrapServers);
        kafkaProperties.put("group.id",fromKafkaGroupID);

        FlinkKafkaConsumer010<String> kafkaConsumer = new FlinkKafkaConsumer010<>(fromKafkaTopic, new SimpleStringSchema(), kafkaProperties);
        kafkaConsumer.setStartFromLatest();
        DataStream<String> kafkaSource = env.addSource(kafkaConsumer).name("KafkaSource").uid("source-id-kafka-source");

        SingleOutputStreamOperator<Tuple4<String, String, String, Integer>> eventStream = kafkaSource.process(new ProcessFunction<String, Tuple4<String, String, String, Integer>>() {
            @Override
            public void processElement(String value, Context ctx, Collector<Tuple4<String, String, String, Integer>> out){
                try {
                    JSONObject obj = JSON.parseObject(value);
                    String userID = obj.getString("userID");
                    String eventTime = obj.getString("eventTime");
                    String eventType = obj.getString("eventType");
                    int productID = obj.getIntValue("productID");
                    out.collect(new Tuple4<>(userID, eventTime, eventType, productID));
                }catch (Exception ex){
                    log.warn("异常数据:{}",value,ex);
                }
            }
        });

        //4、Mysql配置流
        //自定义Mysql Source，周期性地从Mysql中获取配置，并广播出去
        //数据: 用户ID,用户姓名，用户年龄
        DataStreamSource<HashMap<String, Tuple2<String, Integer>>> configStream = env.addSource(new MysqlSource(fromMysqlHost, fromMysqlPort, fromMysqlDB, fromMysqlUser, fromMysqlPasswd, fromMysqlSecondInterval));

        /*
          (1) 先建立MapStateDescriptor
          MapStateDescriptor定义了状态的名称、Key和Value的类型。
          这里，MapStateDescriptor中，key是Void类型，value是Map<String, Tuple2<String,Int>>类型。
         */
        MapStateDescriptor<Void, Map<String, Tuple2<String,Integer>>> configDescriptor = new MapStateDescriptor<>("config", Types.VOID, Types.MAP(Types.STRING, Types.TUPLE(Types.STRING, Types.INT)));

        /*
          (2) 将配置流广播，形成BroadcastStream
         */
        BroadcastStream<HashMap<String, Tuple2<String, Integer>>> broadcastConfigStream = configStream.broadcast(configDescriptor);

        //5、事件流和广播的配置流连接，形成BroadcastConnectedStream
        BroadcastConnectedStream<Tuple4<String, String, String, Integer>, HashMap<String, Tuple2<String, Integer>>> connectedStream = eventStream.connect(broadcastConfigStream);

        //6、对BroadcastConnectedStream应用process方法，根据配置(规则)处理事件
        SingleOutputStreamOperator<Tuple6<String, String, String, Integer, String, Integer>> resultStream = connectedStream.process(new CustomBroadcastProcessFunction());

        //7、输出结果
        resultStream.print();

        //8、生成JobGraph，并开始执行
        env.execute();

    }

    /**
     * 自定义BroadcastProcessFunction
     * 当事件流中的用户ID在配置中出现时，才对该事件处理, 并在事件中补全用户的基础信息
     * Tuple4<String, String, String, Integer>: 第一个流(事件流)的数据类型
     * HashMap<String, Tuple2<String, Integer>>: 第二个流(配置流)的数据类型
     * Tuple6<String, String, String, Integer,String, Integer>: 返回的数据类型
     */
    static class CustomBroadcastProcessFunction extends BroadcastProcessFunction<Tuple4<String, String, String, Integer>, HashMap<String, Tuple2<String, Integer>>, Tuple6<String, String, String, Integer, String, Integer>>{

        /**定义MapStateDescriptor*/
        MapStateDescriptor<Void, Map<String, Tuple2<String,Integer>>> configDescriptor = new MapStateDescriptor<>("config", Types.VOID, Types.MAP(Types.STRING, Types.TUPLE(Types.STRING, Types.INT)));

        /**
         * 读取状态，并基于状态，处理事件流中的数据
         * 在这里，从上下文中获取状态，基于获取的状态，对事件流中的数据进行处理
         * @param value 事件流中的数据
         * @param ctx 上下文
         * @param out 输出零条或多条数据
         * @throws Exception
         */
        @Override
        public void processElement(Tuple4<String, String, String, Integer> value, ReadOnlyContext ctx, Collector<Tuple6<String, String, String, Integer, String, Integer>> out) throws Exception {

            //事件流中的用户ID
            String userID = value.f0;

            //获取状态
            ReadOnlyBroadcastState<Void, Map<String, Tuple2<String, Integer>>> broadcastState = ctx.getBroadcastState(configDescriptor);
            Map<String, Tuple2<String, Integer>> broadcastStateUserInfo = broadcastState.get(null);

            //配置中有此用户，则在该事件中添加用户的userName、userAge字段。
            //配置中没有此用户，则丢弃
            Tuple2<String, Integer> userInfo = broadcastStateUserInfo.get(userID);
            if(userInfo!=null){
                out.collect(new Tuple6<>(value.f0,value.f1,value.f2,value.f3,userInfo.f0,userInfo.f1));
            }

        }

        /**
         * 处理广播流中的每一条数据，并更新状态
         * @param value 广播流中的数据
         * @param ctx 上下文
         * @param out 输出零条或多条数据
         * @throws Exception
         */
        @Override
        public void processBroadcastElement(HashMap<String, Tuple2<String, Integer>> value, Context ctx, Collector<Tuple6<String, String, String, Integer, String, Integer>> out) throws Exception {

            //获取状态
            BroadcastState<Void, Map<String, Tuple2<String, Integer>>> broadcastState = ctx.getBroadcastState(configDescriptor);

            //清空状态
            broadcastState.clear();

            //更新状态
            broadcastState.put(null,value);

        }
    }



    /**
     * 自定义Mysql Source，每隔 secondInterval 秒从Mysql中获取一次配置
     */
    static class MysqlSource extends RichSourceFunction<HashMap<String, Tuple2<String, Integer>>> {

        private String host;
        private Integer port;
        private String db;
        private String user;
        private String passwd;
        private Integer secondInterval;

        private volatile boolean isRunning = true;

        private Connection connection;
        private PreparedStatement preparedStatement;

        MysqlSource(String host, Integer port, String db, String user, String passwd,Integer secondInterval) {
            this.host = host;
            this.port = port;
            this.db = db;
            this.user = user;
            this.passwd = passwd;
            this.secondInterval = secondInterval;
        }

        /**
         * 开始时, 在open()方法中建立连接
         * @param parameters
         * @throws Exception
         */
        @Override
        public void open(Configuration parameters) throws Exception {
            super.open(parameters);
            Class.forName("com.mysql.jdbc.Driver");
            connection= DriverManager.getConnection("jdbc:mysql://"+host+":"+port+"/"+db+"?useUnicode=true&characterEncoding=UTF-8", user, passwd);
            String sql="select userID,userName,userAge from user_info";
            preparedStatement=connection.prepareStatement(sql);
        }

        /**
         * 执行完，调用close()方法关系连接，释放资源
         * @throws Exception
         */
        @Override
        public void close() throws Exception {
            super.close();

            if(connection!=null){
                connection.close();
            }

            if(preparedStatement !=null){
                preparedStatement.close();
            }
        }

        /**
         * 调用run()方法获取数据
         * @param ctx
         */
        @Override
        public void run(SourceContext<HashMap<String, Tuple2<String, Integer>>> ctx) {
            try {
                while (isRunning){
                    HashMap<String, Tuple2<String, Integer>> output = new HashMap<>();
                    ResultSet resultSet = preparedStatement.executeQuery();
                    while (resultSet.next()){
                        String userID = resultSet.getString("userID");
                        String userName = resultSet.getString("userName");
                        int userAge = resultSet.getInt("userAge");
                        output.put(userID,new Tuple2<>(userName,userAge));
                    }
                    ctx.collect(output);
                    //每隔多少秒执行一次查询
                    Thread.sleep(1000*secondInterval);
                }
            }catch (Exception ex){
                log.error("从Mysql获取配置异常...",ex);
            }


        }

        /**
         * 取消时，会调用此方法
         */
        @Override
        public void cancel() {
            isRunning = false;
        }
    }
}
```

运行结果

```
//修改配置前
(user_5,2019-08-18 08:17:19,browse,3,user_name5,25)
(user_3,2019-08-18 08:17:19,click,1,user_name3,23)
(user_2,2019-08-18 08:17:20,click,3,user_name2,22)
(user_5,2019-08-18 08:17:20,browse,1,user_name5,25)

//Flink 应用不停，更新配置(用户基础信息)，这里更新user_2、user_5的年龄
//可以看到，动态修改的配置已生效
(user_5,2019-08-18 08:19:51,click,3,user_name5,15)
(user_3,2019-08-18 08:19:52,browse,1,user_name3,23)
(user_4,2019-08-18 08:19:52,browse,1,user_name4,24)
(user_2,2019-08-18 08:19:53,click,3,user_name2,12)
(user_3,2019-08-18 08:19:53,click,3,user_name3,23)

//Flink 应用不停，删除配置(删除部分用户)，这里只保留user_2
//可以看到，动态修改的配置已生效
(user_2,2019-08-18 08:23:14,click,1,user_name2,12)
(user_2,2019-08-18 08:23:16,click,1,user_name2,12)
(user_2,2019-08-18 08:23:20,browse,2,user_name2,12)
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