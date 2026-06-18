---
title: Flink2.1 AI+LLM大模型调用初体验
author: 大数据技术与架构
date:
url: https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247524531&idx=1&sn=7e2a2f99f0f2f05172682405d1237df2&chksm=fcc3a18e3687f2f32ab83575168bedbfbec04a32971defdc9f942d23d52edaa54952b5706c39&mpshare=1&scene=24&srcid=0825xhX73zzOXfdPxF2RZrO2&sharer_shareinfo=2bb192afe024411b548eb94a300653c9&sharer_shareinfo_first=2bb192afe024411b548eb94a300653c9#rd
---

> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink2.0关键变化与选型影响|Flink2.0关键变化与选型影响]]


Apache Flink 2.1版本在8月中旬正式发布，标志着实时数据处理引擎向统一Data + AI平台的里程碑式演进。

其中很重要的一个能力是关于Flink在实时AI能力上的突破：

* 新增AI模型DDL，支持通过Flink SQL与Table API创建和修改AI模型，实现AI模型的灵活管理。
* 扩展ML\_PREDICT表值函数，支持通过Flink SQL实时调用AI模型，为构建端到端实时AI工作流奠定基础。

我们今天来看一下这个功能，如何使用呢？

首先，Flink允许我们使用`CREATE MODEL`语法创建一个模型：

```
CREATE MODEL `compliance_model`
INPUT (text STRING)
OUTPUT (response STRING)
WITH(
'provider'='openai',
'endpoint'='https://api.openai.com/v1/llm/v1/chat',
'api-key'='abcdefg',
'system_prompt' = '你是电商合规审核员，请判断商品标题是否含有酒精、烟草等敏感内容，仅返回JSON：{"risk":0.0~1.0,"reason":"原因"}',
'model'='gpt-4o'
);
```

然后，通过`ML_PREDICT`函数进行实时推理：

```
SELECT * FROM ML_PREDICT(
  INPUT => TABLE input_table,
  MODEL => MODEL my_model,
  ARGS => DESCRIPTOR(text),
  CONFIG => MAP['async', 'true']
);
```

其中`INPUT`代表我们的输入数据，一般是类似下面这样的source table：

```
CREATE TABLE product_source (
  id   STRING,
  title STRING,
  ts   TIMESTAMP_LTZ(3) METADATA FROM 'timestamp',
  WATERMARK FOR ts AS ts - INTERVAL '5' SECOND
) WITH (
  'connector' = 'kafka',
  'topic'     = 'product_source',
  'properties.bootstrap.servers' = 'localhost:9092',
  'format'    = 'json'
);
```

然后我们定义结果表：

```
CREATE TABLE risk_sink (
  id      STRING,
  title   STRING,
  risk    DOUBLE,
  reason  STRING
) WITH (
  'connector' = 'kafka',
  'topic'     = 'risk_sink',
  'properties.bootstrap.servers' = 'localhost:9092',
  'format'    = 'json'
);
```

然后我们就可以通过一条SQL启动作业：

```
INSERT INTO risk_sink
SELECT
  id,
  title,
  CAST(JSON_VALUE(response,'$.risk') AS DOUBLE)  AS risk,
  JSON_VALUE(response,'$.reason')                AS reason
FROM (
  SELECT
    id,
    title,
    ML_PREDICT(compliance_model, title) AS response
  FROM product_source
) t;
```

假如我们的输入数据为：

```
kafka-console-producer.sh --broker-list localhost:9092 --topic product_source
>{"id":"1","title":"葡萄汁无酒精"}
>{"id":"2","title":"天然香草提取物"}   # LLM 会识别其含酒精
>{"id":"3","title":"茅台飞天53度"}     # 高风险
```

输出结果数据：

```
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic risk_sink --from-beginning

>{"id":"2","title":"天然香草提取物","risk":0.92,"reason":"香草提取物通常含酒精"}
>{"id":"3","title":"茅台飞天53度","risk":0.99,"reason":"明确含酒精饮品"}
```

根据官方的文档，Flink对大模型的调用支持异步访问，并且默认打开。

在资源规划上，可以参考Little定律进行资源规划:

* L：队列槽位（对应max-concurrent-operations）
* λ：请求速率（对应预期的QPS）
* W：平均延迟（对应模型的响应时间）

例如：对于目标100QPS和1.2秒的99百分位延迟，我们需要120个最大并发请求（max-concurrent-operations）。此外，考虑到队列长度和平均行大小，我们需要更多关注 TaskManager 中的内存设置。适当的调优可能显著提升运行AI函数的吞吐量和稳定性。

此外，Flink 2.1的ML框架已经原生支持「Embedding→向量存储→向量检索→LLM」的RAG链路，我们后面再单独分享。

好啦，本次分享就到这里。

最后，欢迎加入我们的知识星球小圈子：
 [《300万字！全网最全大数据学习面试社区等你来》](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247518343&idx=1&sn=eaf44bee8c4d90aedd508201475b57a9&chksm=fd3ece12ca494704cdf02300bbfe39c1d4245ac30f41aba3065efe66eb9aa453a23e0b6dae6e&scene=21&token=1745806505&lang=zh_CN#wechat_redirect)。

如果这个文章对你有帮助，不要忘记 **「在看」** **「点赞」** **「收藏」** 三连啊喂！