---
title: Flink Kafka Sink 根据用户配置的字段自动分发到对应Topic改造的思路
author: ConradJam
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5MTAzNjY0Mg==&mid=2247483682&idx=1&sn=6272132200b1edbbd677c28ff91c6608&chksm=97145fe4958a804688c453ed5ef2a7f646363ce787857cf684906068fc3b8f1055b2eeb101ab&mpshare=1&scene=24&srcid=0901rh1C7gLSwfxP9J9dQn9o&sharer_shareinfo=85997d6df0682eb580f7501d1f7178d7&sharer_shareinfo_first=85997d6df0682eb580f7501d1f7178d7#rd
---

最近在项目上有一些改造Flink Kafka Sink的需求，用户需要根据计算后的某个字段的值，分发到对应的Topic。同时如果这个Topic不存在的时候，我们就分发到默认设置的topic。这个需求需要用FlinkSQL完成表达，不能使用JAR包，避免难以维护

# 项目背景

为什么突然有这样的需求

* 埋点的总表是一个Topic承接所有流量，我们现在需要做分发动作
* 分发需要按照埋点某些字段进行分发，例如按照kafka\_topic这样的一个字段去做一些分发
* 业务方需要使用FlinkSQL，不想使用JAR代码去完成这样的功能
* 如果用户没设置分发的字段，还是按照原有的逻辑进行分发，当流Join无法匹配到的Topic时候，我们需要发送至默认的topic，而不是创建topic进行推送

# 实现调研

看看有没人实现类似的功能，或者我们看看AI如何解答这个问题？

## 豆包AI建议

豆包通过AI编程，代码导入了Flink Kafka Connector的连接器，开始填写以下promt

你现在是一个Flink和Kafka专家，我现在有个需求，用户可以通过FlinkSQL定义，把需要把计算后的某个字段，通过Kafka动态Sink到多个Topic，我如何改造该connector实现性能最高，最大吞吐量的按照某个字段发送到多个Topic，需要考虑当某个topic不存在的时候，自动发送到默认配置的topic

AI给出答案：

1. **自定义 KafkaRecordSerializationSchema**：实现一个能够从输入数据中提取 Topic 名称的选择器
2. **自定义 Topic 选择逻辑**：实现 `TopicSelector` 接口，根据计算后的某个字段选择目标 Topic。
3. **集成到 Flink SQL**：创建自定义 TableSink，使其支持 SQL 语法
4. **优化性能**：处理大量 Topic 时的性能优化策略

我仔细看完这个方案，它存在一些问题：

* kafkaSink在内部一般只维护一个，在静态情况下，只需要维护一个。如果动态情况下，意味着要动态维护多个kafkaSink
* 自定义kafka topic逻辑需要考虑到默认的topic
* 集成到flink-sql中，没有单独可以填写的参数

## KeyedSerializationSchema

看到一些微信公众号实现类似如下（Scala代码，自己转换一下Java吧）

```
class OverridingTopicSchema extends KeyedSerializationSchema[Map[String, Any]] {  
  
  override def serializeKey(element: Map[String, Any]): Array[Byte] = null  
  
  override def serializeValue(element: Map[String, Any]): Array[Byte] = JsonTool.encode(element) //这里用JsonTool指代json序列化的工具类  
  
  override def getTargetTopic(element: Map[String, Any]): String = {  
    if (element != null && element.contains(“@topic”)) {  
      element(“@topic”).toString  
    } else null  
  }  
}
```

之后在new `FlinkKafkaProducer`对象的时候 把上面我们实现的这个`OverridingTopicSchema`传进去就行了。这种方法的确也可以，但是只限于DataStream流

# 如何实现

实验环境

|  |  |
| --- | --- |
| **类型** | **值** |
| Flink版本 | 1.20.1 |
| Flink Kafka Sink Version | 3.4 |

## 实现过程

看看有没人实现类似的功能

**KafkaConnectorOptions**

```
public static final ConfigOption<String> DYNAMIC_TOPIC_SOURCE_FIELD = ConfigOptions            .key("dynamic-topic-source-field")            .stringType()            .defaultValue(null)            .withDescription("target topic form source field,if can't send record or not have "                    + "topic,will be send record to topic by default");
```

**ResolvedSchemaUtils.getDynamicTopicFiledIndex(context,dynamicTopicFiled)**

这段逻辑主要就是从我们的index中取出来，然后根据用户配置的字段获取flink rowdata对应定义的字段下标，定义完成后，我们通过解析看看这个字段用户有没传入，没有直接返回null，按照原来的逻辑去处理，如果发现就解析，如果解析失败，没有对应的字段，我们就抛出异常。

```
package org.apache.flink.connector.kafka.util;  
import org.apache.flink.table.catalog.Column;import org.apache.flink.table.catalog.ResolvedSchema;import org.apache.flink.table.factories.DynamicTableFactory;  
import org.slf4j.Logger;import org.slf4j.LoggerFactory;  
import java.util.List;import java.util.Optional;  
/**   * ResolvedSchema Get Sink/Source filed index  * @Author ConradJam  * @Link https://github.com/czy006 */public class ResolvedSchemaUtils {  
    private static final Logger LOG = LoggerFactory.getLogger(ResolvedSchemaUtils.class);  
    public static Integer getDynamicTopicFiledIndex(            DynamicTableFactory.Context context, String dynamicTopicFiled) {        if (dynamicTopicFiled != null) {            ResolvedSchema schema = context.getCatalogTable().getResolvedSchema();            Optional<Column> schemaColumn = schema.getColumn(dynamicTopicFiled);            if (schemaColumn.isPresent()) {                String dynamicTopicFiledName = schemaColumn.get().getName();                Integer dynamicTopicFiledIndex =                        ResolvedSchemaUtils.getFieldIndex(schema, dynamicTopicFiledName);                LOG.info(                        "Using Kafka DYNAMIC Topic filed in Kafka name {}. index",                        dynamicTopicFiledIndex);                return dynamicTopicFiledIndex;            }            throw new IllegalArgumentException(                    "Can't find DYNAMIC_TOPIC_SOURCE_FIELD in " + "Column:  " + dynamicTopicFiled);        } else {            return null;        }    }  
    public static Integer getFieldIndex(ResolvedSchema schema, String fieldName) {        List<Column> columns = schema.getColumns();        for (int i = 0; i < columns.size(); i++) {            Column column = columns.get(i);            if (column.getName().equals(fieldName)) {                return i;            }        }        throw new RuntimeException("Can't find field with Index name " + fieldName);    }}
```

**KafkaDynamicSink**（214行）

把dynamicTopicFiledIndex获得对应字段的下标传入**DynamicKafkaRecordSerializationSchema**类中，后续用于RowData进行解析

```
final KafkaSink<RowData> kafkaSink =sinkBuilder	.setDeliveryGuarantee(deliveryGuarantee)	.setBootstrapServers(	properties.get(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG).toString())	.setKafkaProducerConfig(properties)	.setRecordSerializer(new DynamicKafkaRecordSerializationSchema(	topics,	topicPattern,	dynamicTopicFiledIndex,	partitioner,	keySerialization,	valueSerialization,	properties,	getFieldGetters(physicalChildren, keyProjection),	getFieldGetters(physicalChildren, valueProjection),	hasMetadata(),	getMetadataPositions(physicalChildren),	upsertMode)).build();
```

**DynamicKafkaRecordSerializationSchema**

在getTarget方法中，我们可以加入以下判断，当我们发现这个index不为空的时候，我把RowData的数据取出来，然后返回对应发送的topic即可

```
        if (dynamicTopicFiledIndex != null) {  
            String targetTopic = element.getString(dynamicTopicFiledIndex).toString();  
            if (targetTopic != null) {  
                return targetTopic;  
            }  
        }
```

到此为止我们的代码基本核心逻辑开发完成，大家可以根据这个思路进行改造

## 功能验证

下面写出使用案例去给大家参考，这里使用了Faker随机产生数据，按照每秒5条进行产生。其中topic\_event\_1，topic\_event\_2在Kafka集群已经创建（关闭kafka自动创建topic功能），**topic\_event\_3没有创建此topic**。

我们期望效果是：根据 **dynamic-topic-source-field**（kafka\_topic）字段去分发到数据到对应的topic上，当此topic不存在时候，发送到 **topic**中

```
CREATE TABLE faker_source (  
  `kafka_topic` STRING,  
  `lv1_subject` STRING  
) WITH (  
  'connector' = 'faker',  
  'fields.kafka_topic.expression' = '#{Options.option ''topic_event_1'',''topic_event_2'',''topic_event_3''}',  
  'fields.lv1_subject.expression' = '#{superhero.name}',  
  'rows-per-second' = '5'  
);  
  
  
CREATE TABLE kafka_event_topic_dymic_sink (  
  `kafka_topic` STRING,  
  `lv1_subject` STRING  
) WITH (  
  'connector' = 'kafka',  
  'topic' = 'topic_event_1',  
  'dynamic-topic-source-field' = 'kafka_topic',  
  'properties.bootstrap.servers' = 'xxxxxxxxx',  
  'properties.group.id' = 'dymanic-kafka-test',  
  'format' = 'json'  
);  
  
  
INSERT INTO kafka_event_topic_dymic_sink  
select  
    kafka_topic,  
    lv1_subject  
FROM default_catalog.default_database.faker_source;
```

# 感悟

实现过程中的一些感悟：

代码在实现过程，现在我会先习惯把仓库先扔给AI分析一下这个功能，看看AI给出的建议和思考，结合AI和以及个人在实时计算领域这个专业程度，去实现代码。目前AI智能程度无法完全替代人类，但是能给予最基本的思路帮助。例如我在写的时候，就知道修改的大概位置，可以根据这些位置我们再做详细的代码工程思考和改造。特别对于不熟悉的一些代码时候有较大的帮助