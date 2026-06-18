> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkSQL流批统一与混合编程|FlinkSQL流批统一与混合编程]]
---
title: 克服Flink SQL限制的混合API方法
author: Apache Flink
date:
url: https://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247515035&idx=1&sn=868aea849ed1a78a7b2292b9bc183d7f&chksm=fc32599f8e0c32ad6b1905d8e363bfd5061d15adb3e5b428221b1ea0a71c395ffa9778f715f2&mpshare=1&scene=24&srcid=1230SSVr3OtmEMyE4iz1EabP&sharer_shareinfo=4830429d4f9a44d94ba2072a6f383bbd&sharer_shareinfo_first=4830429d4f9a44d94ba2072a6f383bbd#rd
---

作者：Gal Krispel

翻译：黄鹏程 阿里云实时计算 Flink 版产品负责人 

阅读时间：11分钟 · 2025年10月19日

## 译者注：

本博客文章探讨了 Apache Flink 中的混合 API 方法如何帮助克服 Flink SQL 的一些固有限制，特别是在与 Apache Kafka 集成时。文章深入探讨了两个常见挑战：

1) 对格式错误记录的有效错误处理；

2) Avro 的 Enum 和 TimestampMicros 类型在数据类型映射方面的限制。

DataStream API 和 ProcessFunction API 凭借其更底层的控制能力，可用于强大的模式验证和死信队列(DLQ)实现。这种预处理步骤通过优雅地处理损坏的记录而不导致应用程序重启，保护了输入 Flink SQL 的数据完整性。

此外，Flink SQL 的数据类型映射问题可以通过将 Flink Table 转换回DataStream<GenericRecord> 并应用自定义 RichMapFunction 来缓解。这允许对序列化进行精确控制，从而在将数据写回 Kafka 时正确处理 Avro Enum 和 TimestampMicros 类型。

虽然 Flink SQL 提供了高度声明性和用户友好的接口，但将其与 DataStream API 的细粒度控制相结合，为复杂的现实世界流式挑战提供了强大而灵活的解决方案。鼓励 Flink 用户考虑如何在应用程序中战略性地切换这些 API，以解锁更大的健壮性、灵活性和控制力，从而构建更具弹性和功能丰富的数据管道。

Apache Flink 是一个强大的数据处理框架，它提供了一个高吞吐量、低延迟的运行时环境，能够以统一的方式处理无界流数据和有界批数据。Flink 提供了多种 API，从类似 SQL 的声明式接口到基于操作符的底层接口，使团队能够构建从实时分析管道到事件驱动应用程序的各种应用，并精确控制所需的功能级别。

Apache Flink 提供了多种实现作业的接口。最受欢迎的接口是 DataStream API，这是一种用于流作业的命令式、底层接口，可对操作符、状态、序列化和错误处理提供强有力的控制。它还提供了插入自定义连接器的能力，以进一步提高灵活性，从而扩展了社区支持的开源连接器的种类。

第二个流行的接口是 Flink SQL（由 Table API 驱动），这是一种更高级别的声明式语法，允许用户使用 ANSI 标准 SQL语法定义数据处理管道。众多连接器支持 Table API，大大简化了编写数据处理作业复杂代码的开销。这一特性被技术背景较弱的用户或不需要掌握 Apache Flink 专业知识的自助服务场景广泛采用。假设每个业务利益相关者都对 Apache Flink 有超出基本水平的理解，那么 Flink SQL 可能就不会像今天这样被广泛采用。然而，现实中并非如此，其声明式语法和易用性使其对没有软件工程背景的数据分析专业人员特别有吸引力。如前所述，它也可用于自助服务场景。

**01**

**使用Flink SQL的隐藏陷阱**

对于流式用例，一个流行的选择是将 Table API 与 Apache Kafka 结合使用，并采用 avro-confluent 格式以确保严格的模式保证。Confluent Schema Registry 确保了减少的有效载荷大小、模式一致性和兼容性，以及用于与注册表交互的简单 REST 接口。

Flink SQL 为原生流式 Kafka Consumer & Producer 流程或 Kafka Streams 应用程序提供了一种简单的替代方案。这些应用程序从主题读取流数据，通过连接、过滤、聚合和窗口等数据操作对其进行处理，然后将输出生成到另一个流中进行进一步处理。这些应用程序带有技术和操作开销——它们需要对 Apache Kafka 有深入的理解，并处理应用程序状态优化。

与编写完整的 Kafka Streams 应用程序相比，Flink SQL 接口允许您以声明方式定义管道，就像使用 SQL 一样。虽然这听起来很理想，但生产用例往往会达不到这种期望。

将 Flink SQL 与 Apache Kafka 结合使用的两个常见挑战是：

1. 缺乏有效的错误处理模式，例如死信队列(DLQ)。在 Apache Kafka 主题中处理模式不兼容记录的情况并不少见，因为模式兼容性没有被强制执行，或者仅仅是因为生产者应用程序的 bug 导致为主题生成了错误的模式。当 Table API 消费者遇到由于故障记录导致的反序列化错误时，它会立即使用快速失败策略并重启应用程序。不幸的是，这种策略在处理坏记录时是无效的，因为它在设置"avro"或"confluent-avro"格式时无法跳过故障记录。应用程序将陷入重启模式，手动干预是唯一能从这种情况中恢复的方法。由于 Kafka 连接器中的 Avro 反序列化不支持 skip-on-error 标志，Flink SQL 缺乏任何真正的错误处理选项。

2. 数据类型映射限制，因为 Flink SQL 类型可能无法精确表示原生 Avro 类型。Flink SQL 的一个显著限制是 Enum 类型处理——所有 Enum 字段都被解释为字符串。从处理端来看，Enum 字段与 String 字段无法区分是有道理的，但在尝试将该字段作为 Enum 类型重新生成到 Apache Kafka 时会产生序列化错误，因为 Flink SQL 中没有官方支持 Enum 数据类型。Avro LogicalType 的 TimestampMicros 也存在类似问题，因为 Flink SQL 不支持按原样读写 timestamp 字段，而只能作为 BIGINT。

**02**

**混合接口方法**

如前所述，由于 DataStream API 和 ProcessFunction API 的底层特性，它们比 Flink SQL 接口具有更丰富的功能集。同时，Flink SQL 接口对于没有深入了解 Apache Flink 的用户来说更加友好和易于使用。在 Apache Flink 运行时中，可以在两种 API 之间切换，这在通用应用程序中是有意义的。

想象一个提供自助服务的应用程序，该应用程序的用户输入将是基于 Table API 的 SQL 进行数据处理，而接收器连接器将使用 DataStream API 或 ProcessFunction API 进行优化写入目标。甚至可能存在来回切换API的用例。

在下面的部分中，将提供一个如何将 Table API 与底层 API 的更深层次功能相结合的演练，例如使用 RichMapFunction 实时转换记录以克服 Enum 序列化问题，以及使用 ProcessFunction 处理坏记录并将其转移到侧输出。

**03**

**在Flink SQL之前验证模式：DLQ的底层API方法**

Flink SQL 允许用户以声明方式处理数据。虽然运行时数据处理错误不是典型场景，但在 SQL 处理步骤之前的模式反序列化阶段经常会遇到格式错误的记录。初步的 Kafka 源流将确保正确编码数据的已验证流，其中坏记录不会被跳过，而是被转移到侧 Kafka 主题，这是流应用程序中众所周知的错误处理模式。正确编码记录的已验证流将被转换为 Table API 进行 SQL 处理。同时，DLQ Kafka 主题将用于存放不兼容或故障记录，而不会对主处理流造成任何运行时异常。

第一步是为验证流设置 Kafka 源连接器。它应该以最原始的形式接收记录，即字节数组，无需事先反序列化。

```
package com.example.flink;
import org.apache.flink.connector.kafka.source.KafkaSource;import org.apache.kafka.clients.consumer.ConsumerRecord;
public class KafkaSourceFactory {    public static KafkaSource<ConsumerRecord<byte[], byte[]>> createRawBytesSource(            String bootstrapServers, String topic, String groupId) {        return KafkaSource.<ConsumerRecord<byte[], byte[]>>builder()                .setBootstrapServers(bootstrapServers)                .setTopics(topic)                .setGroupId(groupId)                .setDeserializer(new RawBytesDeserializer())                .build();    }}
```

RawBytesDeserializer 的实现非常简单，如下所示：

```
public class RawBytesDeserializer implements KafkaRecordDeserializationSchema<ConsumerRecord<byte[], byte[]>> {    @Override    public void deserialize(org.apache.kafka.clients.consumer.ConsumerRecord<byte[], byte[]> record, Collector<ConsumerRecord<byte[], byte[]>> out) {        out.collect(record);    }
    @Override    public TypeInformation<ConsumerRecord<byte[], byte[]>> getProducedType() {        return TypeInformation.of(new TypeHint<ConsumerRecord<byte[], byte[]>>(){});    }}
```

此外，必须为坏记录的 DLQ 输出设置 Kafka 接收器连接器。

```
package com.example.flink;
import org.apache.flink.connector.base.DeliveryGuarantee;import org.apache.flink.connector.kafka.sink.KafkaSink;import org.apache.kafka.clients.consumer.ConsumerRecord;
public class DLQKafkaSinkFactory {    public static KafkaSink<ConsumerRecord<byte[], byte[]>> createDLQSink(            String bootstrapServers, String dlqTopic) {        return KafkaSink.<ConsumerRecord<byte[], byte[]>>builder()                .setBootstrapServers(bootstrapServers)                .setDeliveryGuarantee(DeliveryGuarantee.AT_LEAST_ONCE)                .setRecordSerializer(new DlqRecordSerializer(dlqTopic))                .build();    }}
```

DlqRecordSerializer 是一个简单的 Kafka 记录序列化器，它将坏记录作为原始字节数组生成到 DLQ 主题。

```
public class DlqRecordSerializer implements KafkaRecordSerializationSchema<ConsumerRecord<byte[], byte[]>> {    private final String topic;
    public DlqRecordSerializer(String topic) {        this.topic = topic;    }
    @Override    public ProducerRecord<byte[], byte[]> serialize(ConsumerRecord<byte[], byte[]> element, KafkaSinkContext context, Long timestamp) {        return new ProducerRecord<>(topic, null, null, element.key(), element.value(), element.headers());    }}
```

ProcessFunction 将确定记录是否正确编码。如果验证失败，故障记录将被标记并转移到 DLQ 输出接收器。

```
package com.example.flink;
import org.apache.avro.generic.GenericRecord;import org.apache.flink.api.java.tuple.Tuple2;import org.apache.flink.formats.avro.registry.confluent.ConfluentRegistryAvroDeserializationSchema;import org.apache.flink.streaming.api.functions.ProcessFunction;import org.apache.flink.util.Collector;import org.apache.flink.util.OutputTag;import org.apache.kafka.clients.consumer.ConsumerRecord;
public class SchemaValidationFunction extends ProcessFunction<ConsumerRecord<byte[], byte[]>, Tuple2<String, GenericRecord>> {    private static final OutputTag<ConsumerRecord<byte[], byte[]>> DLQ_TAG = new OutputTag<>("dlq-output") {};
    private final boolean dlqEnabled;    private final ConfluentRegistryAvroDeserializationSchema<GenericRecord> deserializer;
    public SchemaValidationFunction(            boolean dlqEnabled,            ConfluentRegistryAvroDeserializationSchema<GenericRecord> deserializer) {        this.dlqEnabled = dlqEnabled;        this.deserializer = deserializer;    }
    @Override    public void processElement(            ConsumerRecord<byte[], byte[]> record,            Context ctx,            Collector<Tuple2<String, GenericRecord>> out) throws Exception {        try {            GenericRecord value = deserializer.deserialize(record.value());            if (value == null) throw new RuntimeException("null after deserialization");
            String key = record.key() == null ? null : new String(record.key());            out.collect(Tuple2.of(key, value));        } catch (Exception e) {            if (dlqEnabled) {                ctx.output(DLQ_TAG, record);            } else {                throw e;            }        }    }
    public static OutputTag<ConsumerRecord<byte[], byte[]>> getDlqTag() {        return DLQ_TAG;    }}
```

上述资源将在主流作业中初始化。下面是一个示例应用程序，它使用 DataStream 读取 Kafka 主题，验证 Avro 编码，然后将已验证的记录发送到下游进行进一步的 SQL 处理。故障记录将生成到 DLQ 主题。

```
// 获取输入Avro模式并设置反序列化器和转换器org.apache.avro.Schema avroSchema = schemaManager.fetchSchema(subject, version);ConfluentRegistryAvroDeserializationSchema<GenericRecord> deserializer =     ConfluentRegistryAvroDeserializationSchema.forGeneric(avroSchema, registryUrl);RowType rowType = (RowType) AvroSchemaConverter.convertToDataType(avroSchema.toString()).getLogicalType();AvroToRowDataConverters.AvroToRowDataConverter converter = AvroToRowDataConverters.createRowConverter(rowType);
// 使用DataStream API读取原始消息KafkaSource<ConsumerRecord<byte[], byte[]>> kafkaSource = KafkaSourceFactory.createRawBytesSource(props);DataStream<ConsumerRecord<byte[], byte[]>> rawStream = env.fromSource(kafkaSource, WatermarkStrategy.noWatermarks(), "Kafka Source");
TypeInformation<Tuple2<String, GenericRecord>> sourceTupleTypeInfo =     new TupleTypeInfo<>(TypeInformation.of(String.class), new GenericRecordAvroTypeInfo(avroSchema));
// 为已验证记录设置已验证流SingleOutputStreamOperator<Tuple2<String, GenericRecord>> validatedRecordStream = rawStream    .process(new SchemaValidationFunction(dlqEnabled, deserializer))    .returns(sourceTupleTypeInfo);
// 如果启用，则创建DLQ接收器if (dlqEnabled && !dlqTopic.isEmpty()) {    DataStream<ConsumerRecord<byte[], byte[]>> dlqStream = validatedRecordStream.getSideOutput(SchemaValidationFunction.getDlqTag());    KafkaSink<ConsumerRecord<byte[], byte[]>> dlqSink = DLQKafkaSinkFactory.createDLQSink(props, dlqTopic);    dlqStream.sinkTo(dlqSink);}
// 准备表环境基础表结构RowType originalRowType = (RowType) AvroSchemaConverter.convertToDataType(avroSchema.toString()).getLogicalType();
// 操作已验证流以包含元数据，保留键以供上下文使用（当前未使用）DataStream<RowData> rowStreamWithMetadata = validatedRecordStream    .map(tuple -> (RowData) converter.convert(tuple.f1))    .returns(InternalTypeInfo.of(originalRowType));
// 定义表的模式Schema.Builder schemaBuilder = Schema.newBuilder();schemaBuilder.fromRowDataType(AvroSchemaConverter.convertToDataType(inputAvroSchema.toString()));
// 将DataStream注册为具有avro模式的表Table t = tableEnv.fromDataStream(rowStreamWithMetadata, schemaBuilder.build());tableEnv.createTemporaryView("InputTable", t);
// 处理SQL查询tableEnv.sqlQuery(sqlQuery);
```

提供的代码片段展示了如何有效地结合底层 API 进行强大的模式验证和 DLQ 错误处理，以及 Flink SQL 的声明式优势进行后续数据处理。

**04**

**未来工作**

## 使用 DataStream API 克服 Flink SQL 数据类型映射限制

Table API 可以实现用户的 Flink SQL 查询并将其接收到底层各种接收器连接器中，包括 Apache Kafka。然而，如上所述，它在数据类型映射方面有一些限制：

* Enum 类型被转换为 String
* Flink SQL 不支持精度高于3（毫秒）的 TIMESTAMP

这可能会非常令人沮丧。通常，用户的意图是让 Apache Flink 使用现有模式，而不是让它自己注册模式。如果有技术要求阻止 Apache Flink 注册模式，那么所有带有 TimestampMicros 或 Enum 的模式都无法写入 Apache Kafka 接收器主题，这使得它们在 Flink SQL 中实际上不受支持。

虽然 Enum 数据类型和 TimestampMicros Avro 类型在 Flink SQL 中不受支持，但它们在 DataStream API 中完全受支持，当将 GenericRecord 或 SpecificRecord 写回 Kafka 时。

之前展示了一个如何将 DataStream 转换为 Flink Table 以确保有效数据在 Flink SQL 中处理的示例。要克服这个问题，必须进行相反的转换——将 Flink Table 转换回 DataStream<GenericRecord> 并将其接收至 Kafka。

从 TableAPI 切换到 DataStream 很简单，可以通过 toDataStream 完成。然而，这种转换将产生 DataStream<Row>，这是一种与将 Avro 编码记录写入 Kafka 不兼容的格式。需要应用用户定义函数来执行从 DataStream<Row> 到 DataStream<GenericRecord> 的转换。在该函数中，将应用自定义代码将数据写回 Kafka，包括 Flink SQL 不支持的字段，如 Enum 或 TimestampMicros。

下面是如何为原始类型、Enum 和 Timestamp 编写从 Row 到 GenericRecord 的映射器的示例（扩展映射器以支持您的模式选择；下面的示例处理特定情况和一般基本原语）。

```
class AvroRowMapper extends RichMapFunction<Row, GenericRecord> {    private final String schemaString;
    AvroRowMapper(String schemaString) {        this.schemaString = schemaString;    }
    @Override    public GenericRecord map(Row row) {        org.apache.avro.Schema schema = new org.apache.avro.Schema.Parser().parse(schemaString);        GenericRecord rec = new GenericData.Record(schema);
        for (org.apache.avro.Schema.Field f : schema.getFields()) {            String name = f.name();            Object v = row.getField(name);            org.apache.avro.Schema fs = unwrapNullable(f.schema());
            if (v == null) {                rec.put(name, null);                continue;            }
            String logical = fs.getProp("logicalType");            if ("timestamp-micros".equals(logical)) {                rec.put(name, toEpochMicros(v));                continue;            } else if ("timestamp-millis".equals(logical)) {                rec.put(name, toEpochMillis(v));                continue;            }
            switch (fs.getType()) {                case STRING:                    rec.put(name, v.toString());                    break;                case BOOLEAN:                    rec.put(name, (Boolean) v);                    break;                case INT:                    rec.put(name, ((Number) v).intValue());                    break;                case LONG:                    rec.put(name, ((Number) v).longValue());                    break;                case FLOAT:                    rec.put(name, ((Number) v).floatValue());                    break;                case DOUBLE:                    rec.put(name, ((Number) v).doubleValue());                    break;                case BYTES:                    if (v instanceof byte[])                        rec.put(name, java.nio.ByteBuffer.wrap((byte[]) v));                    else                        rec.put(name, v);                    break;                case ENUM:                    rec.put(name, new GenericData.EnumSymbol(fs, v.toString()));                    break;                default:                    rec.put(name, v);            }        }        return rec;    }
    private static long toEpochMillis(Object v) {        if (v instanceof java.time.LocalDateTime) {            return ((java.time.LocalDateTime) v).toInstant(java.time.ZoneOffset.UTC).toEpochMilli();        } else if (v instanceof java.time.Instant) {            return ((java.time.Instant) v).toEpochMilli();        } else if (v instanceof Number) {            return ((Number) v).longValue();        }        throw new IllegalArgumentException("Unsupported timestamp-millis value: " + v);    }
    private static long toEpochMicros(Object v) {        if (v instanceof java.time.LocalDateTime) {            long millis = ((java.time.LocalDateTime) v).toInstant(java.time.ZoneOffset.UTC).toEpochMilli();            return millis * 1_000L;        } else if (v instanceof java.time.Instant) {            long millis = ((java.time.Instant) v).toEpochMilli();            return millis * 1_000L;        } else if (v instanceof Number) {            return ((Number) v).longValue();        }        throw new IllegalArgumentException("Unsupported timestamp-micros value: " + v);    }
    private static org.apache.avro.Schema unwrapNullable(org.apache.avro.Schema s) {        if (s.getType() == org.apache.avro.Schema.Type.UNION) {            for (org.apache.avro.Schema t : s.getTypes()) {                if (t.getType() != org.apache.avro.Schema.Type.NULL)                    return t;            }        }        return s;    }}
```

创建一个 Kafka 接收器，将记录作为 Avro 编码消息返回到 Kafka。

```
package com.example.flink.sink;
import org.apache.avro.generic.GenericRecord;import org.apache.flink.connector.base.DeliveryGuarantee;import org.apache.flink.connector.kafka.sink.KafkaSink;import org.apache.flink.formats.avro.registry.confluent.ConfluentRegistryAvroSerializationSchema;import org.apache.flink.connector.kafka.sink.KafkaRecordSerializationSchema;
public class KafkaSinkFactory {    public static KafkaSink<GenericRecord> createKafkaSink(            String bootstrapServers,            String topic,            String subject,            org.apache.avro.Schema schema,            String schemaRegistryUrl) {        return KafkaSink.<GenericRecord>builder()                .setBootstrapServers(bootstrapServers)                .setDeliveryGuarantee(DeliveryGuarantee.AT_LEAST_ONCE)                .setRecordSerializer(                    KafkaRecordSerializationSchema.builder()                        .setTopic(topic)                        .setValueSerializationSchema(                            ConfluentRegistryAvroSerializationSchema.forGeneric(                                subject, schema, schemaRegistryUrl                            )                        )                        .build()                )                .build();    }}
```

预定义的映射函数可在主应用程序中如下使用：

```
// 将Table转换为Row流DataStream<Row> rows = tableEnv.toDataStream(result);
// 使用上面的最小映射器将行映射到GenericRecordString avroSchemaString = /* your Avro schema JSON */;DataStream<GenericRecord> records = rows.map(new AvroRowMapper(avroSchemaString))    .returns(new GenericRecordAvroTypeInfo(outputSchema));
// 构建一个将GenericRecord与Confluent Avro一起写入的Kafka接收器KafkaSink<GenericRecord> sink = KafkaSinkFactory(bootstrapServers,topic,subject,outputSchema,registryUrl)
// 发送它records.sinkTo(sink);
```

▼ 「实时计算 Flink 版」 ▼

复制下方链接或者扫描左边二维码

即可免费试用阿里云 **Serverless Flink**，体验新一代实时计算平台的强大能力！

了解试用详情：https://free.aliyun.com/?productCode=sc

---

▼ 关注「**Apache Flink**」 ▼

回复 FFA 2025 获取大会资料

**点击「阅****读****原文****」****跳转**阿里云实时计算 Flink～****