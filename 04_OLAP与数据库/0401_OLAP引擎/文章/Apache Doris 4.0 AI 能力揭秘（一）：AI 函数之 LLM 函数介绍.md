---
title: Apache Doris 4.0 AI 能力揭秘（一）：AI 函数之 LLM 函数介绍
author: SelectDB
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537405&idx=1&sn=67932fff0e5a57189162236eb67560ad&chksm=ce38a22ecb5f994cd7ef75f2f62fb13d149a9d42159f78817bfb35c5728142241782414fd2a1&mpshare=1&scene=24&srcid=0909quXseZgsk09UhafuLMlc&sharer_shareinfo=ef99729c1543d103eb0977b190e32823&sharer_shareinfo_first=ef99729c1543d103eb0977b190e32823#rd
---

**导读**

---

在数据日益密集的当下，我们总在寻求更高效、更智能的数据分析工具。随着大语言模型（LLM）的兴起，如何将这些前沿的 AI 能力与日常的数据分析工作相结合，已然成为一个极具探索价值的方向。

基于此，我们在 Apache Doris 4.0 版本中实现了一系列 LLM 函数。这使得数据分析能够凭借简洁的 SQL 语句，直接调用大语言模型开展文本处理工作。无论是从文本中精准提取特定重要信息，还是对评论进行细致的情感分类，亦或是生成精炼的文本摘要，皆可在数据库内部无缝完成。

> 扫码添加 Doris 小助手微信并备注“4.0”，抢先参与 4.0 版本内测：

## 应用场景

---

在即将发布的 4.0 版本中，Apache Doris LLM 函数可应用的场景包括但不限于：

* 智能反馈：自动识别用户意图、情感。
* 内容审核：批量检测并处理敏感信息，保障合规。
* 用户洞察：自动分类、摘要用户反馈。
* 数据治理：智能纠错、提取关键信息，提升数据质量。

所有大语言模型必须在 Doris 外部提供，并且支持文本分析。此外，所有 LLM 函数调用结果和成本取决于外部 LLM 供应商及其所使用的模型。

## 函数支持

---

* **LLM\_CLASSIFY**：在给定的标签中提取与文本内容匹配度最高的单个标签字符串。

* **LLM\_EXTRACT**：根据文本内容，为每个给定标签提取相关信息。
* **LLM\_FILTER**: 判断文本内容是否正确，返回值为 `bool` 类型。
* **LLM\_FIXGRAMMAR**：修复文本中的语法、拼写错误。
* **LLM\_GENERATE**：基于参数内容生成内容。
* **LLM\_MASK**: 根据标签，将原文中的敏感信息用`[MASKED]`进行替换处理。
* **LLM\_SENTIMENT**：分析文本情感倾向，返回值为 `positive`、`negative`、`neutral`、`mixed`其中之一。
* **LLM\_SIMILARITY**：判断两文本的语义相似度，返回值为 0 - 10 之间的浮点数，值越大代表语义越相似。
* **LLM\_SUMMARIZE**：对文本进行高度总结概括。
* **LLM\_TRANSLATE**：将文本翻译为指定语言。

## LLM 配置相关参数

---

Doris 通过资源机制对 LLM API 的访问进行集中管理，旨在确保密钥安全与权限可控。目前可选择的参数如下：

* `type`： 必填，且必须为 `llm` ，作为 llm 的类型标识。
* `llm.provider_type`： 必填，外部 LLM 厂商类型。
* `llm.endpoint`： 必填，LLM API 接口地址。
* `llm.model_name`： 必填，模型名称。
* `llm_api_key`：除 `llm.provider_type = local`的情况外必填，API 密钥。
* `llm.temperature`：可选，控制生成内容的随机性，取值范围为 0 到 1 的浮点数。默认值为 -1，表示不设置该参数。
* `llm.max_tokens`：可选，限制生成内容的最大 token 数。默认值为 -1，表示不设置该参数。Anthropic 默认值为 2048。
* `llm.max_retries`：可选，单次请求的最大重试次数。默认值为 3。
* `llm.retry_delay_second`：可选，重试的延迟时间（秒）。默认值为 0。

## 厂商支持

---

目前直接支持的厂商有：OpenAI、Anthropic、Gemini、DeepSeek、Local（Ollama 等）、MoonShot、MiniMax、Zhipu、Qwen、Baichuan。

若有不在上列的厂商，但其 API 格式与 OpenAI/Anthropic/Gemini 相同的，在填入参数 `llm.provider_type` 时可直接选择三者中格式相同的厂商。原因是厂商选择只影响 Doris 内部所构建的 API 格式。

## 快速上手

---

为了帮助用户尽快上手，我们准备了一些示例 Demo，以下示例均为最小实现：

> 具体步骤参考文档：https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/llm-functions/llm-function（复制前往浏览器打开）

### 01 配置 LLM 资源

示例一：

```
```
CREATE RESOURCE 'openai_example'  
PROPERTIES (      
    'type' = 'llm',      
    'llm.provider_type' = 'openai',      
    'llm.endpoint' = 'https://api.openai.com/v1/responses',      
    'llm.model_name' = 'gpt-4.1',      
    'llm.api_key' = 'xxxxx'  
);
```
```

示例二：

```
```
CREATE RESOURCE 'deepseek_example'  
PROPERTIES (      
    'type'='llm',      
    'llm.provider_type'='deepseek',      
    'llm.endpoint'='https://api.deepseek.com/chat/completions',      
    'llm.model_name' = 'deepseek-chat',      
    'llm.api_key' = 'xxxxx'  
);
```
```

### 02 设置默认资源（可选）

```
```
SET default_llm_resource='llm_resource_name';
```
```

### 03 执行 SQL 查询

**case1:**

假设存在如下数据表，表中存储了与数据库相关的文档内容：

```
```
CREATE TABLE doc_pool (      
    id  BIGINT,      
    c   TEXT  
) DUPLICATE KEY(id)  
DISTRIBUTED BY HASH(id) BUCKETS 10  
PROPERTIES (      
    "replication_num" = "1"  
);
```

若需筛选与 Doris 相关性最高的 10 条记录，可采用如下查询：
```

```
```
SELECT      
    c,      
    CAST(LLM_GENERATE(CONCAT('Please score the relevance of the following document content to Apache Doris, with a floating-point number from 0 to 10, output only the score. Document:', c)) AS DOUBLE) AS score  
FROM doc_pool   
ORDER BY score DESC   
LIMIT 10;
```
```

该查询将利用 LLM 生成每条文档内容与 Apache Doris 的相关性评分，并按得分降序筛选前 10 条结果。

```
```
+---------------------------------------------------------------------------------------------------------------+-------+  
| c                                                                                                             | score |  
+---------------------------------------------------------------------------------------------------------------+-------+  
| Apache Doris is a lightning-fast MPP analytical database that supports sub-second multidimensional analytics. |   9.5 |  
| In Doris, materialized views can automatically route queries, saving significant compute resources.           |   9.2 |  
| Doris's vectorized execution engine boosts aggregation query performance by 5–10×.                            |   9.2 |  
| Apache Doris Stream Load supports second-levelreal-timedata ingestion.                                      |   9.2 |  
| Doris cost-based optimizer (CBO) generates better distributed execution plans.                                |   8.5 |  
| Enabling the Doris Pipeline execution engine noticeably improves CPU utilization.                             |   8.5 |  
| Doris supports Hive externaltablesfor federated queries without moving data.                                |   8.5 |  
| Doris Light SchemaChange lets you addordropcolumns instantly.                                             |   8.5 |  
| Doris AUTOBUCKET automatically scales bucketcountwithdata volume.                                         |   8.5 |  
| Using Doris inverted indexes enables second-levellog searching.                                              |   8.5 |  
+---------------------------------------------------------------------------------------------------------------+-------+
```
```

**case2:**

该表模拟招聘场景的候选人简历和职业要求：

```
```
CREATE TABLE candidate_profiles (  
    candidate_id INT,  
    name         VARCHAR(50),  
    self_intro   VARCHAR(500)  
)  
DUPLICATE KEY(candidate_id)  
DISTRIBUTED BY HASH(candidate_id) BUCKETS 1  
PROPERTIES (  
    "replication_num" = "1"  
);   
  
CREATE TABLE job_requirements (  
    job_id   INT,  
    title    VARCHAR(100),  
    jd_text  VARCHAR(500)  
)  
DUPLICATE KEY(job_id)  
DISTRIBUTED BY HASH(job_id) BUCKETS 1  
PROPERTIES (  
    "replication_num" = "1"  
);   
  
INSERT INTO candidate_profiles VALUES  
(1, 'Alice', 'I am a senior backend engineer with 7 years of experience in Java, Spring Cloud and high-concurrency systems.'),   
(2, 'Bob',   'Frontend developer focusing on React, TypeScript and performance optimization for e-commerce sites.'),   
(3, 'Cathy', 'Data scientist specializing in NLP, large language models and recommendation systems.');   
  
INSERT INTO job_requirements VALUES  
(101, 'Backend Engineer', 'Looking for a senior backend engineer with deep Java expertise and experience designing distributed systems.'),   
(102, 'ML Engineer',      'Seeking a data scientist or ML engineer familiar with NLP and large language models.');
```
```

可以通过 `LLM_FILTER` 将职业要求和候选人简介进行语义匹配，快速筛选出合适的候选人。

```
```
SELECT      
    c.candidate_id, c.name,      
    j.job_id, j.title  
FROM candidate_profiles AS c  
JOIN job_requirements AS j  
WHERE LLM_FILTER(CONCAT('Does the following candidate self-introduction match the job description?',                   
    'Job: ', j.jd_text, ' Candidate: ', c.self_intro));
```
```

输出结果参考：

```
```
+--------------+-------+--------+------------------+  
| candidate_id | name  | job_id | title            |  
+--------------+-------+--------+------------------+  
|            3 | Cathy |    102 | ML Engineer      |  
|            1 | Alice |    101 | Backend Engineer |  
+--------------+-------+--------+------------------+
```
```

**case3:**

该表模拟保险公司的理赔申请数据：

```
```
CREATE TABLE claims (  
    claim_id INT COMMENT '索赔编号',  
    policy_id INT COMMENT '保单编号',  
    claim_date DATE COMMENT '索赔日期',  
    incident_description VARCHAR(1000) COMMENT '事故描述'  
) DUPLICATE KEY(claim_id)  
DISTRIBUTED BY HASH(claim_id) BUCKETS 5  
PROPERTIES (  
    "replication_num" = "1"  
);  
  
CREATE TABLE policies (  
    policy_id INT COMMENT '保单编号',  
    policy_type VARCHAR(50) COMMENT '保单类型',  
    insured_item VARCHAR(255) COMMENT '承保物品/对象'  
) DUPLICATE KEY(policy_id)  
DISTRIBUTED BY HASH(policy_id) BUCKETS 5  
PROPERTIES (  
    "replication_num" = "1"  
);  
  
INSERT INTO claims VALUES  
(1, 101, '2025-08-18', '昨天下午三点左右，我在东三环辅路开车时，与前车发生了追尾。'),  
(2, 102, '2025-08-18', '上周五在公司楼下，我不小心扭伤了脚踝，需要理赔医疗费用。'),  
(3, 103, '2025-08-18', '8月17日夜里家中管道破裂，导致部分家具被水浸泡。'),  
(4, 104, '2025-08-18', '晚上喝酒后开车回家与其他车辆发生了碰撞。'),  
(5, 105, '2025-08-18', '早上8点，在去上班的路上，发现车辆停放时被刮擦。');  
  
  
INSERT INTO policies VALUES  
(101, '车险', '宝马 X5'),  
(102, '健康险', '个人意外险'),  
(103, '家财险', '住宅房屋'),  
(104, '车险', '丰田 凯美瑞'),  
(105, '车险', '奥迪 A8');
```
```

可以利用 `LLM_CLASSIFY` 函数对事件性质进行智能分类，并通过 `LLM_FILTER` 函数对事件进行有效性校验，以筛选出符合理赔标准的有效事件。

```
```
SELECT  
    c.claim_id,  
    c.incident_description,  
    llm_classify(c.incident_description, ['交通事故', '人身意外', '财产损失', '其他']) AS incident_category  
FROM claims AS c  
JOIN policies AS p ON c.policy_id = p.policy_id  
WHERE  
    p.policy_type = '车险' AND LLM_FILTER(CONCAT('下列情形是否支持保险赔偿：', c.incident_description));
```

输出结果参考：

```
+----------+-----------------------------------------------------------------------------------------+-------------------+  
| claim_id | incident_description                                                                    | incident_category |  
+----------+-----------------------------------------------------------------------------------------+-------------------+  
|        1 | 昨天下午三点左右，我在东三环辅路开车时，与前车发生了追尾。                                        | 交通事故           |  
|        5 | 早上8点，在去上班的路上，发现车辆停放时被刮擦。                                                 | 财产损失           |  
+----------+-----------------------------------------------------------------------------------------+-------------------+
```
```

## 设计原理

---

### 01 函数执行流程

### 02 资源化管理

Doris 将 LLM 能力抽象为资源（Resource），统一管理各种大模型服务（如 OpenAI、DeepSeek、Moonshot、本地模型等）。每个资源都包含了厂商、模型类型、API Key、Endpoint 等关键信息，简化了多模型、多环境的接入和切换，同时也保证了密钥安全和权限可控。

### 03 兼容主流大模型

由于厂商之间的 API 格式存在差异，Doris 为每种服务都实现了请求构造、鉴权、响应解析等核心方法，让 Doris 能够根据资源配置，动态选择合适的实现，无需关心底层 API 的差异。用户只需声明提供厂商，Doris 就能自动完成不同大模型服务的对接和调用。

## **总结**

---

Apache Doris 4.0 的 LLM 函数为数据分析与智能应用场景注入了强大的能力，覆盖智能反馈、内容审核、用户洞察和数据治理等多领域需求。通过灵活的资源化管理和对主流大模型（如 OpenAI、Anthropic、DeepSeek 等）的广泛兼容，Doris 提供了一站式的智能分析解决方案，极大简化了复杂模型的接入与使用流程。无论是高效的语义匹配、情感分析，还是自动化内容生成与数据优化，Doris LLM 函数都能以高性能、低成本的方式助力企业释放数据潜能。

### 现在就行动！

🚀 参与 Apache Doris 4.0 内测，体验 “SQL + AI” 的颠覆性组合，让数据分析从“被动查询”迈向“主动洞察”。更多 LLM 函数示例与最佳实践，请前往 Apache Doris 官网文档或 Doris x 4.0 内测交流群获取。加入社区，与全球开发者共同打造新一代实时智能数仓！

添加 Doris 小助手，留言“4.0”加入内测交流群，还可以免费领取 Doris x AI 和 100+ 企业实践案例集，获取技术帮助、了解最新动态，并与更多开发者和用户互动。

## **- END -**

**更多标杆企业信赖**

智慧金融与政企：[东北证券](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247534484&idx=1&sn=be3c0e26da40a1da03dd3bc487961880&scene=21#wechat_redirect)｜[国金证券](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535362&idx=1&sn=745fd6aa178aae78e807f59932c7e5eb&scene=21#wechat_redirect)|[国信证券](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537026&idx=1&sn=19a88be7610378cb0283f43e0563cb61&scene=21#wechat_redirect)｜[杭银消金](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247517678&idx=1&sn=2fa963e0cf8194ad8a027f2c108d5459&chksm=cf2f8be9f85802ffb84237297a0c2ec6efdf1266f9213ee028f46e7e6aad43e7f10042015095&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[河北幸福消费金融](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522589&idx=1&sn=2c8e14756aa6727ef51da608dfb074f7&chksm=cf2f971af8581e0c2fdd887636a9844eef3b16c3c9832d9e62e1457cff641935db69832f885e&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[汇添富基金](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532266&idx=1&sn=12b08c27cc55dd9e6074f565ddeb9378&chksm=cf2f70edf858f9fb7d19675a068a77b92bba293fba079e267cefcb46ae5ec452b646d7b69d93&scene=21#wechat_redirect)｜[金融壹账通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522120&idx=1&sn=32bfd0bec1a56c7ecea05c088e566cd1&chksm=cf2f994ff858105979a27bd768133ff2c663e3f7ed5f894e373d834a1e35e67b15ca15c2de6d&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[陆金所控股](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247534403&idx=1&sn=a07c42af38ce146f75fbf3fc1a96f4ba&scene=21#wechat_redirect)｜[霖梓控股](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535962&idx=1&sn=1d348942d631ebc15a1a4cb5b73b2177&scene=21#wechat_redirect)｜[拉卡拉](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536412&idx=1&sn=099a0e58cfb6a49a444a6afa4268438b&scene=21#wechat_redirect)｜[平安人寿](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526035&idx=1&sn=ce723ff107a98a6d8887d45455c9e4c0&chksm=cf2f6894f858e182ab243f4e96ce9d675831802272f5e40915bf9c7edd8d36b0b022d9ec3ca1&scene=21&cur_album_id=2524165801138995201#wechat_redirect)[｜](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526035&idx=1&sn=ce723ff107a98a6d8887d45455c9e4c0&chksm=cf2f6894f858e182ab243f4e96ce9d675831802272f5e40915bf9c7edd8d36b0b022d9ec3ca1&scene=21&cur_album_id=2524165801138995201#wechat_redirect)[奇富科技](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526469&idx=1&sn=4b2d7748da1a3b601499d7d2d80748b2&chksm=cf2f6642f858ef54b7dc5b307ec4eaba7d867ee5ab5862b9e60274cc56940787686da30873fa&scene=21#wechat_redirect)｜[同程数科](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247521341&idx=1&sn=3e2b5ee81ebe6ba8b2a238e91ea7f7b7&chksm=cf2f9a3af858132c380332c123813f1df24169e30040363428db78cc0fef541f6ee802f2262d&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[通联支付](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532843&idx=1&sn=ec7c674e29a66dcb4a78ad24e9507bc2&chksm=cf2f4f2cf858c63a7953dbbe8893c2335bee15b0abe3d97d45725cfe80d9b35aa465f43d6264&scene=21#wechat_redirect)｜[无锡锡商银行](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529958&idx=1&sn=e720bfefc8cc63148c3aa6ceeee69be6&chksm=cf2f7be1f858f2f75dfab49839eafeffeddb93c49851fc8ebdcbcfd788230f6d06b1be6ad9fb&scene=21#wechat_redirect)｜[星云零售信贷](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522161&idx=1&sn=40862998ddead3c398c7765e1e8b57d8&chksm=cf2f9976f8581060f9cbf6e402f3a39d35e86a559f7d59c80b680d41977c6165e0f42cce8422&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[星火保](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532661&idx=1&sn=adba43453acddfdcf2f294f2e46727fc&chksm=cf2f4e72f858c76437d3dc998b712deb7841fb2d65c45e017aeb4f9e61e74e4ad710a5cd0df1&scene=21#wechat_redirect)｜[银联商务](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526954&idx=1&sn=4dae0891ccbaae99146b5a6ba783f392&chksm=cf2f642df858ed3b939be0b04d264ddb0529556fe15ad70c3501af44d0026972cd1bef49a7b7&scene=21#wechat_redirect)｜[易生支付](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532494&idx=1&sn=aec3c408890ce0b6fd802a7f7d7e2bd1&chksm=cf2f71c9f858f8dfff7ae63f7bd3b4b91bc2717f83824d3245afb8ebbc44b1be96cbe9403354&scene=21#wechat_redirect)｜[招商信诺人寿](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524176&idx=1&sn=19138ac93dd44f395d68745cf94d9a6b&chksm=cf2f9157f858184176169417464dce040cd89fab6b35099d8b76086342f9682c49fd8cc87582&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[招联金融](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533296&idx=1&sn=b94ffa477e3398afe94e38b66479bf57&chksm=cf2f4cf7f858c5e10f996f0634f967de6478eaa573a45f102502602e16e142397935d58205d5&scene=21#wechat_redirect)｜[中信银行信用卡中心](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535858&idx=1&sn=8edeb10d97936a88b52543c2ac108288&scene=21#wechat_redirect)｜[360 数科](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247509531&idx=1&sn=60f3b5160acf1f2e6df2f30242dc4306&chksm=cf2fa81cf858210abd2b7158c19b39dc1431cc9114b1f463d25586dafe8c60673992deda574a&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[360 企业安全浏览器](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247527814&idx=1&sn=f4b56af0fac8b465b40819cd080f7563&chksm=cf2f6381f858ea97d8121642b03ae99fe48cd8c5a514c18e571be235405a90397d25f243b624&scene=21#wechat_redirect)

**互联网与文娱**：[菜鸟](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537342&idx=1&sn=118685b9bfe9c0442e0bb1558718431f&scene=21#wechat_redirect)｜[抖音集团](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247530668&idx=1&sn=3ecfbb295fc4e3fd7617044ac3575543&chksm=cf2f76abf858ffbd0718c2102777dfbb914f07deb05bf6e07a663107f1d0f1897257ba02a131&scene=21#wechat_redirect)｜[斗鱼](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247517967&idx=1&sn=0896767aa1b7d1f0b314c22023e4de3c&chksm=cf2f8908f858001e935f7f0c1c8082847c31851dc27377652b57a8c6b3c4883890e0448eff27&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[叮咚买菜](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247518311&idx=1&sn=d18e9d2be16c26833d4d3e03f5c2836b&chksm=cf2f8660f8580f7632c6211ede6d56a4a79531c95981c72dccc76948dd9f7afdb2c419db10db&scene=21&cur_album_id=2524165801138995201#wechat_redirect)|[浩瀚深度](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537072&idx=1&sn=6b5f4fae6224b2535badfd29ba3c8ecb&scene=21#wechat_redirect)｜[京东](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536067&idx=1&sn=6325bd6379241c56925cdfd0b748212e&scene=21#wechat_redirect)｜[工商信息查询平台](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526281&idx=1&sn=a586ef0e7b8d1cc631cc08da63331dbc&chksm=cf2f698ef858e098407b107df2511260590c09f2629071fcc40f24632c92582304230cb7bf7f&scene=21#wechat_redirect)｜[货拉拉](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247496431&idx=1&sn=65b8ff94a0c2de9e60b42ebfe454a4dd&scene=21#wechat_redirect)｜[快手](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533822&idx=1&sn=36a3db48f30db59f05ff4313d205d146&chksm=cf2f4af9f858c3ef54b109d4fc465d6883a541b8b18adc23b0dcb15ca5e876201af2498cc811&scene=21#wechat_redirect)｜[荔枝微课](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247519304&idx=1&sn=6c59fae3838e1f02a29eebca1b5799e9&chksm=cf2f824ff8580b593720ebed5234490c9b4a00fa616218b2ee3f8e89beb95eab11c720022e91&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[票务平台](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247521422&idx=1&sn=e42a6e93170f14bd17a48812f1e8ece4&chksm=cf2f9a89f858139f1ffa47678d3c4d068f44788ac18f288b1e31243bad0bfff1745b5763794c&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[墨迹天气](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247530525&idx=1&sn=33ac4ce5192c12a8b19b6b18a344f718&chksm=cf2f761af858ff0c0c69f833e501ca52e0a5818a17e0fcc8c1d7f86d6b8d364b83b5ad6d2244&scene=21#wechat_redirect)｜[MiniMax](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536310&idx=1&sn=8c2d8980a4c7b3f88421f1ed5f1d0eba&scene=21#wechat_redirect)｜[奇安信](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524942&idx=1&sn=331f494035518da1875fca0fc9c5437a&chksm=cf2f6c49f858e55fd2ad0aa4a2f958b9b1b0f790441a519471debe217c38b6a5bde06ae46e87&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[趣丸科技](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247530649&idx=1&sn=da6e4444e1e50b54ba640cc40385b33c&chksm=cf2f769ef858ff88bc36e0ef5e894b3898348123db2cc3d9d4d55b7b56e17e65402f09bc32a0&scene=21#wechat_redirect)｜[顺丰科技](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536639&idx=1&sn=2f0cb56207b97a63aa5857dffac9ea83&scene=21#wechat_redirect)｜[腾讯音乐](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533968&idx=1&sn=bd1f32f608c3ce648c12f5cf3365db8c&chksm=cf2f4b97f858c2811330a4247d40bfd404ea031cc0c4f2c7d5ae79309b204c13afe8d81ff1a4&scene=21#wechat_redirect)｜[天眼查](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247519778&idx=1&sn=e32c89396fc87a09965cc89b5e9e1b4b&chksm=cf2f8025f858093392bffd5619ba53be539a865128bcedd6e6094cf6e49082282cd7c3bc5298&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[网易](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529891&idx=1&sn=fbdc2154807b766bd4534522dc48295d&chksm=cf2f7ba4f858f2b26996ef3fc3564da96a632a81d4ae01500a77e58ef925cbdb14d6e22efdad&scene=21#wechat_redirect)｜[网易游戏](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247531491&idx=1&sn=bbff8938a6d08b6ead0a08e203709f01&chksm=cf2f75e4f858fcf2a6e4023e93aec4fd165af096388a6fa61906fa640b82ff4e60cd3d8a2641&scene=21#wechat_redirect)｜[网易严选](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247503369&idx=1&sn=29fab18c22f5778b0bb19409cb68f567&chksm=cf2fc00ef8584918fc809d661066f139d002f0202f2f6477a02286527409296d6212a373e58a&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[网易云信](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536880&idx=1&sn=710229bd13e19f06216eb7211220a0ce&scene=21#wechat_redirect)｜[网易云音乐](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535994&idx=1&sn=88d69524871d7b34191d2f2d778e78e6&scene=21#wechat_redirect)｜[小米](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247537383&idx=1&sn=47a4bc984bf6097a74b343e77ec4a86b&scene=21#wechat_redirect)｜[小鹅通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524553&idx=1&sn=63af19e32467049aa3b2fff79898c5ce&chksm=cf2f6ecef858e7d8dc3939f71ebef3bf17962e50889707b8393080a7e2ffca7520c814232bd7&scene=21&cur_album_id=2524165801138995201&poc_token=HCXDWmWjWHZ5_WrVJMWvScoFjGBJjOj_ZV5DcLPW#wechat_redirect)｜[迅雷](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247531772&idx=1&sn=3db2e0c4f8dec01085bd6e24d2c857b3&chksm=cf2f72fbf858fbed95f9aa74e103b76c828dd5433a0338e97354db4781974e9444a957e9adde&scene=21#wechat_redirect)｜[约苗](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247521201&idx=1&sn=dc8f3d55343cd054974e8c5f4bfe50a2&chksm=cf2f9db6f85814a055a245c88c5c5c138fb7b9a9bbacedfa39dae563726ae43ad237993a5872&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[字节跳动](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247516005&idx=1&sn=84d069e950cbb178a45cd569b244d6a4&chksm=cf2fb162f8583874b83a2a591a92e75cc689f56f4ff2324ccbb583494daf76d9a96e3d6feb42&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[知乎](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247520797&idx=1&sn=1fd8394aafaddbd0ab0fbfc322e5fb25&chksm=cf2f9c1af858150c4ef3e2d9b5762477584d3e0908d71ab8871b81061253343e689c6aa18ac0&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[360 商业化](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247518769&idx=1&sn=60b5ac3a78930baddc845f1e6c66a29c&chksm=cf2f8436f8580d201bacec5b3434ee62b2fea52718c4b82355c8e8c698fc2d4228a2b0cc0bef&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)

**企业服务与新经济**：[宝尊科技](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536183&idx=1&sn=d4ddeaf5a8ebbb284b5207c62ea50c40&scene=21#wechat_redirect)｜[Cisco](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535303&idx=1&sn=3a57e0de6153d6715e5d19f1f3ff0717&scene=21#wechat_redirect)｜[橙联](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247507345&idx=1&sn=4dc83cae32333c2aa2355c9447ecfb81&chksm=cf2fd396f8585a8018f753f50d89f6bbbb607c86350bf49957bbc07894477b88a61f3eba9342&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[度言](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247512880&idx=1&sn=9b3adbddb44b7287835adf560a56e65b&chksm=cf2fbd37f8583421316ef89513c84654956b8d9e2231868613ce4c9ad439127bb5a8a4077b73&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[观测云](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526244&idx=1&sn=af3b3bc7a5d5419d003ada6b9ea9c376&chksm=cf2f6963f858e0758e623e16612bceae95457c6d7aeed918be6d0d5bd53e8962d14549b64de0&scene=21#wechat_redirect)｜[慧策](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247515310&idx=1&sn=abfa22039b546171dd6cb1ead692de0f&chksm=cf2fb2a9f8583bbff034fc5824f89bf6a1d960a1f7997f3c3fc42af27b271c76d28aecf4b522&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[快成物流](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247531733&idx=1&sn=53e4df3ca765685dcc2b429db833a64d&chksm=cf2f72d2f858fbc4e5ccc99ea75262b6903467c9c3ea3e66919a587545fe1ea01cd7b2fd3b62&scene=21#wechat_redirect)｜[领健](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247512380&idx=1&sn=e7bb6b39a71b2a350acd7ac8d2f3b202&chksm=cf2fbf3bf858362d4e4326712e9d360493c2660e5d619e0c7ec23383b560bea2f0644b10c84f&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[领创](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247495090&idx=1&sn=314d0a12cab7985f822876ba99e29e39&chksm=cf2fe3b5f8586aa36938cd8493f98caba5fb821ca897ecf06412e6f90cb1c3deed02c4e7ed89&token=2145458351&lang=zh_CN&scene=21#wechat_redirect)｜[灵犀科技](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535808&idx=1&sn=142386b7a74856d424676f79b281d07a&scene=21#wechat_redirect)｜[名创优品](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529981&idx=1&sn=85ba7efaa7c947573d59d87ae4897cc4&chksm=cf2f7bfaf858f2ec757fe383ec0bd4053bdfb09c09197fa1587c36d8c3947394d6621116341a&scene=21#wechat_redirect)｜[Moka BI](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247518538&idx=1&sn=d9a39d040c9f430a841a2045a2bafb90&chksm=cf2f874df8580e5b0d77c6235aadd8610c80b4afd1c2781172ba6e1fb734dbb6e463d6f8794e&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[美联物业](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247517792&idx=1&sn=455c085a102c7af1fcac44e62bc06ca1&chksm=cf2f8867f8580171c05af38785d7c7a32766308df46f482f7731cdc0d3dc76d9ed0a69b864f2&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[钱大妈](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529314&idx=1&sn=e1564a7bc1184ba73c353c465eae503d&chksm=cf2f7d65f858f473f71cad992eb80cdf095cb873fe2394c6a2deab07ecbd12aae6eb5de72a60&scene=21#wechat_redirect)｜[拈花云科](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522076&idx=1&sn=c2b2b0f72e9bc25b9020238048a3c6ae&chksm=cf2f991bf858100dfdacf18748ef75a4e76e6cbf34e8b1e52ffc55232e6f6db0ae6c4d9b1b69&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[森马](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536942&idx=1&sn=aeb684eaa379dbb4d910d9fefc678ccd&scene=21#wechat_redirect) |[思必驰](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247506825&idx=1&sn=b6d002ddef220acff8963b0af04e3676&chksm=cf2fd58ef8585c9829f474fe4f75a8b6fa9e2a923b474043d0f92582318197fd737933d4cc59&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[顺丰科技](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536639&idx=1&sn=2f0cb56207b97a63aa5857dffac9ea83&scene=21#wechat_redirect)｜[上海家化](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529885&idx=1&sn=b7c4ae401c29dfa64d10097a2ae3c678&chksm=cf2f7b9af858f28c309136b6a68ede29631849dceba2b84d180b07cd04d02937c940c74975ae&scene=21#wechat_redirect) | [物易云通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247490382&idx=1&sn=531f8d388526ee57aff6e86c8befe66c&scene=21#wechat_redirect)｜[云积互动](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247508739&idx=1&sn=52d028e91f9fa9cf4e93036970711eed&chksm=cf2fad04f8582412ec4cfd3c7a4a672326652c0062a481eda98a704ec245a878f5246ad236bc&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[有赞](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524089&idx=1&sn=95b479a8648809456100e121677fb29d&chksm=cf2f90fef85819e8e73021e841d396ad1cc44e002bff0002c1b431741b3790a775415b829bf8&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[雨润集团](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532231&idx=1&sn=ff3913472ee157907f40e174ddf370bd&chksm=cf2f70c0f858f9d69c4f195e01df934a28149797aacaffb8382d0883c7fb07dcf5a861aa239c&scene=21#wechat_redirect)｜[纵腾集团](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247513813&idx=1&sn=348def05d3f82fc3e7b119a78b3c2b84&chksm=cf2fb8d2f85831c45349ea5a3e4b6c27503aa41f783903a856b915a6556b3d4c81691fb24a7c&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[中通快递](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533447&idx=1&sn=68dcc5af518f418d77536c711c3a4639&scene=21#wechat_redirect)

**先进智造与电信**：[爱玛](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529522&idx=1&sn=95d61e4fea493b667612a946ed126b2d&chksm=cf2f7a35f858f323918bfa574f60d3c8af817c60c6e1680f0b639533528fce3183e60ac90e95&scene=21#wechat_redirect)｜[长安汽车](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526078&idx=1&sn=55ac5982a5a81eb98d380cdca2fe4a1f&chksm=cf2f68b9f858e1af4f53a4eeb489516d2508f513435f9c82b50896acce8cbe522f165034ce9f&scene=21&cur_album_id=2524165801138995201#wechat_redirect)[｜](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526078&idx=1&sn=55ac5982a5a81eb98d380cdca2fe4a1f&chksm=cf2f68b9f858e1af4f53a4eeb489516d2508f513435f9c82b50896acce8cbe522f165034ce9f&scene=21&cur_album_id=2524165801138995201#wechat_redirect)[极越汽车](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247530634&idx=1&sn=b02e765afeefbe7eb709260ec5127031&chksm=cf2f768df858ff9b78f0e61907bf86ef319c7d03e6eddc116c0da70713bb853e3301da160148&scene=21#wechat_redirect)｜[金风科技](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533704&idx=1&sn=f1c619e15f318e58ad70befd0e70fc91&scene=21#wechat_redirect)｜[科大讯飞](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247534327&idx=1&sn=60c35720f3f412db6a78079815acc592&chksm=cf2f48f0f858c1e6f7ca4b881ed20e60c0aba8554972514674e291e26face6586fc96413e2e9&scene=21#wechat_redirect)｜[Lifewit](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247487975&idx=1&sn=0cfd5f9d748cb982e1ff5abc35fddb9a&chksm=cf2c1fe0f85b96f652975a5f88a9ca85d6ef75e55427dda02126d34f06d2e2a3ac15bda56413&token=2145458351&lang=zh_CN&scene=21#wechat_redirect)｜[哪吒科技](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247531497&idx=1&sn=473ea00ef53c0bd902968446e8b0e6e4&chksm=cf2f75eef858fcf850ab57e1c28d5fbfc81aa369c865c1153105b9d8e4a61d910a739634686d&scene=21#wechat_redirect)｜[四川航空](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536110&idx=1&sn=86a18fe553ddc620be30173c663101e4&scene=21#wechat_redirect)｜[上海通用五菱](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247530723&idx=1&sn=47c5c9d104566b638cb09e9d0ab3f448&chksm=cf2f76e4f858fff26b1aac2bb8d6f2c718ac8025a33aeed62bb319a43863f8a50eb2b79a21d7&scene=21#wechat_redirect)｜[三星电子](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247533303&idx=1&sn=5b8a674e5ab8fbc2a9d2d462e1527a32&chksm=cf2f4cf0f858c5e689ecebf5c2f2a60681926946038f3be3ffddc11bfb29b256c158870b1ee5&scene=21#wechat_redirect)｜[蜀海供应链](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247496660&idx=1&sn=1c0738a95564fc4cebb8a56cb539f703&scene=21#wechat_redirect)｜[特步](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247535841&idx=1&sn=7a0a2851555bdd5f340bb3aa640fc61c&scene=21#wechat_redirect)｜[天翼云](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247536234&idx=1&sn=957e6d589138555c5c7c94e62a793126&scene=21#wechat_redirect)｜[雅迪](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247532027&idx=1&sn=d4de3a3326c1450b063d017aaaa78506&chksm=cf2f73fcf858faea620ba48c0b78b3746485a74fd5f1d29482b0dbed3cbd6d374ecc17cd2453&scene=21#wechat_redirect)｜[中国联通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247523168&idx=1&sn=f6a8195d485b56438a4413ca05c9a5e7&chksm=cf2f9567f8581c71f476c52f89de7e4e6769d17e232c8930e257d2db71a45b9ab83a50382655&scene=21&cur_album_id=2524165801138995201#wechat_redirect)

作为基于 Apache Doris 的商业化公司，飞轮科技秉承着 “开源技术创新”和“实时数仓服务”双轮驱动的战略，在投入资源大力参与 Apache Doris 社区研发和推广的同时，基于 Apache Doris 内核打造了聚焦于企业大数据实时分析需求的企业级产品 SelectDB ，面向新一代需求打造世界领先的实时分析能力。自 2022 年成立以来，获得 IDG 资本、红杉中国、襄禾资本等顶级 VC 的近 10 亿元融资，创下了近年来开源基础软件领域的新纪录。

#### 

#### ▼ 点击阅读原文，下载 Doris 最新版本！