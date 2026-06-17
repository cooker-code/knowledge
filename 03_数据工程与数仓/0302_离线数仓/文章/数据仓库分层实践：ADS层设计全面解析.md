---
title: 数据仓库分层实践：ADS层设计全面解析
author: 马伟说
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU3NDEzODU4MA==&mid=2247483981&idx=1&sn=56f3b4fd826eba15beb25126d2312f58&chksm=fc12f1c511663691bed4c5a434f0935ab10f162890831c7880b05728cf95820f6c40d0643ca4&mpshare=1&scene=24&srcid=0315OrtxAArSEPnbA8xMFagR&sharer_shareinfo=b3f786aa70188410ff0fceffb0515268&sharer_shareinfo_first=b3f786aa70188410ff0fceffb0515268#rd
---

**万物皆有声，关注即共鸣...**

在数据仓库的架构设计中，ADS(Application Data Service)作为面向业务应用的最终数据输出层，承接了从数据加工到业务输出的关键任务，起到了承上启下的关键作用。它位于DWS之上，直接服务于业务需求，以满足报表展示、即席查询、数据分析等应用场景，为此提供结构化的主题数据集市（Data Mart）。

在设计ADS层时，您必须注意以下几点：

* 必须以业务需求为核心，将DWS层整合为业务所需要的数据形式。虽然在设计时需要考虑到未来可能的业务扩展，但还是要尽量避免过度设计，减少冗余字段或者预留通用字段等。
* 可以通过宽表化、预聚合、分区等手段，减少复杂计算和多表关联。尽量屏蔽底层复杂的数据处理逻辑，为上层应用提供简洁、易用的数据接口。
* 设计时要保证指标口径、命名规则与字段含义的一致性与可读、直观性。确保数据访问权限控制，防止敏感数据泄露。

# 01 设计原则与步骤

ADS层的主要数据来源是DWS层，通过进一步加工（如维度关联、指标计算）生成面向应用的主题数据集市。因此，尽量避免或者说减少ADS直接访问DWD或ODS层，以保持分层逻辑的清晰性。

### 1.1 设计原则

在设计ADS层时，一般需要遵循以下原则：

1. 业务导向

ADS层的首要设计原则就是业务导向。因此，每个数据集市都应该对应明确的业务主题，如销售分析、用户行为分析等。

同时，需要深入地理解业务需求，包括关键问题、分析维度、指标定义、时效性要求等等。

2. 性能优先

对于ADS层来说，性能至关重要，直接影响用户的使用效率与系统体验。可以参考如下方法来提升性能：

1. 采用星型模型或宽表设计，减少多表关联。
2. 预计算常用的聚合指标，优化查询性能。
3. 合理的采用分区和索引方法来提升查询效率。

3. 口径一致

数据口径的一致性在整个设计中都非常重要，您需要：

1. 统一维度定义，如时间粒度、客户分类。
2. 统一指标口径，如GMV、DAU的计算规则。
3. 提供数据字典，明确字段含义和计算逻辑。

4. 可扩展性

在可扩展性方面，您可以采用如下建议：

1. 使用通用的维度和事实表设计，便于横向扩展。
2. 预留冗余字段，支持未来需求变更，但一定要注意一个度，切勿过度设计。
3. 模块化设计，便于垂直扩展新的数据集市。

5. 安全可控

作为最直接面向应用的数据层，其数据安全性不言而喻，您可以参考如下建议：

1. 实现细粒度的访问控制，保护敏感数据。
2. 对敏感信息（如用户隐私数据）进行脱敏处理。
3. 记录操作审计，确保数据访问可追溯。

6. 简洁易用

最后，尽量屏蔽底层复杂的数据处理逻辑，为上层应用提供简洁、易用的数据接口。表与字段命名规范且直观，含义清晰，可读可维护性高。

### 1.2 一般设计步骤

参考以上的设计原则，ADS层的一般设计步骤如下：

1. 确定数据集市

根据业务需求，确定需要构建的数据集市，每个数据集市对应一个明确的业务主题和应用场景。例如，在电商应用场景中，常见数据集市包括如下：

```
销售分析集市用户行为分析集市商品分析集市营销效果分析集市供应链优化集市财务分析集市
```

2. 设计星型模型

对于每个数据集市，您可以采用星型模型进行设计，描述清楚一个事实表和多个维度表之间的关系。

3. 设计预计算

为提升查询性能，预计算一些常用的聚合指标，例如按天、按月汇总的销售数据等。

4. 优化查询性能

通过分区、索引、物化视图等方式优化查询性能。

5. 数据安全与权限管理

实现细粒度的访问控制、数据脱敏和操作审计，确保数据安全。

### 1.3 一个简单的设计示例

以“每日销售分析”为例，核心的分析需求包括：

* 按日期、地区、商品类别、渠道查看销售数据。
* 核心指标：销售额、订单量、下单用户数、退货率。
* 数据需按天更新，支持快速查询。

数据来源如下：

* DWS层表：

```
dws_sales_order_daily：每日销售汇总表，按订单日期、地区、商品类别、渠道汇总，包含销售额、订单量、用户数、退货计数等指标。
```

* 维度表：

```
dim_region：地区维度表。dim_category：商品类别维度表。dim_channel：渠道维度表。dim_date：日期维度表。
```

其中：

* 分区设计：按日期dim\_date进行分区，如分区值为“2023-10-01”。

* 唯一约束：使用“dim\_date + dim\_region\_id + dim\_category\_id + dim\_channel\_id”作为组合主键。

根据上面的描述，设计ADS表ads\_sales\_summary\_daily如下：

```
-- 设置并行度和资源分配SET mapreduce.job.reduces=50;SET hive.exec.dynamic.partition.mode=nonstrict;  
-- 创建ADS层表CREATE TABLE ads_sales_summary_daily (    dim_region_id STRING COMMENT '地区ID',    dim_region_name STRING COMMENT '地区名称（英文）',    dim_region_name_cn STRING COMMENT '地区名称（中文）',    dim_category_id STRING COMMENT '商品类别ID',    dim_category_name STRING COMMENT '商品类别名称',    dim_channel_id STRING COMMENT '渠道ID',    dim_channel_name STRING COMMENT '渠道名称',    met_sales_amt DECIMAL(18,2) COMMENT '销售额（单位：元）',    met_order_cnt BIGINT COMMENT '订单量',    met_user_cnt BIGINT COMMENT '下单用户数',    met_return_rate DECIMAL(5,2) COMMENT '退货率（%）',    meta_create_time TIMESTAMP COMMENT '数据创建时间',    meta_update_time TIMESTAMP COMMENT '数据更新时间',    CONSTRAINT pk_sales_summary PRIMARY KEY (dim_region_id, dim_category_id, dim_channel_id) DISABLE NOVALIDATE)PARTITIONED BY (dim_date STRING)STORED AS PARQUET;  
-- 插入数据（每日全量更新）INSERT OVERWRITE TABLE ads_sales_summary_daily PARTITION (dim_date='${bizdate}')SELECT     t1.region_id AS dim_region_id,    r.region_name AS dim_region_name,    r.region_name_cn AS dim_region_name_cn,    t1.category_id AS dim_category_id,    c.category_name AS dim_category_name,    t1.channel_id AS dim_channel_id,    ch.channel_name AS dim_channel_name,    COALESCE(SUM(t1.sales_amt), 0) AS met_sales_amt,    COALESCE(SUM(t1.order_cnt), 0) AS met_order_cnt,    COALESCE(SUM(t1.user_cnt), 0) AS met_user_cnt,    COALESCE(100.0 * SUM(t1.return_cnt) / NULLIF(SUM(t1.order_cnt), 0), 0) AS met_return_rate,    CURRENT_TIMESTAMP AS meta_create_time,    CURRENT_TIMESTAMP AS meta_update_timeFROM     dws_sales_order_daily t1LEFT JOIN     dim_region r ON t1.region_id = r.region_idLEFT JOIN     dim_category c ON t1.category_id = c.category_idLEFT JOIN     dim_channel ch ON t1.channel_id = ch.channel_idLEFT JOIN     dim_date d ON t1.order_date = d.date_idWHERE     t1.order_date = '${bizdate}'GROUP BY     t1.region_id,    r.region_name,    r.region_name_cn,    t1.category_id,    c.category_name,    t1.channel_id,    ch.channel_nameHAVING SUM(t1.order_cnt) > 0; -- 确保有订单数据才写入分区
```

如果采用增量更新机制，可以参考如下：

```
-- 使用MERGE语句进行增量更新MERGE INTO ads_sales_summary_daily tUSING (    SELECT * FROM dws_sales_order_daily     WHERE order_date = '${bizdate}') sON (t.dim_date = s.order_date AND t.dim_region_id = s.region_id     AND t.dim_category_id = s.category_id AND t.dim_channel_id = s.channel_id)WHEN MATCHED THEN     UPDATE SET         t.met_sales_amt = s.sales_amt,        t.met_order_cnt = s.order_cnt,        t.met_user_cnt = s.user_cnt,        t.met_return_rate = 100.0 * s.return_cnt / NULLIF(s.order_cnt, 0),        t.meta_update_time = CURRENT_TIMESTAMPWHEN NOT MATCHED THEN    INSERT (dim_date, dim_region_id, dim_region_name, dim_region_name_cn,             dim_category_id, dim_category_name, dim_channel_id, dim_channel_name,            met_sales_amt, met_order_cnt, met_user_cnt, met_return_rate,             meta_create_time, meta_update_time)    VALUES (s.order_date, s.region_id, r.region_name, r.region_name_cn,            s.category_id, c.category_name, s.channel_id, ch.channel_name,            s.sales_amt, s.order_cnt, s.user_cnt,             100.0 * s.return_cnt / NULLIF(s.order_cnt, 0),            CURRENT_TIMESTAMP, CURRENT_TIMESTAMP);
```

# 02 命名规范与字段设计

命名规范也是ADS层设计的重要环节，统一的命名规范可以提升表和字段的可读性与维护性。以下是一些推荐性的命名规范与字段设计规范：

### 2.1 命名规范

1. 表名规范

建议采用“ads\_业务域\_主题\_时间范围”的格式，例如：

```
ads_sales_order_daily：销售订单日统计表ads_user_behavior_monthly：用户行为月统计表
```

2. 字段命名

字段命名使用小写字母，单词间用下划线分隔。为了确保字段含义的清晰易读性，尽量避免采用缩写形式，除非是一些业界惯例的通用缩写。通过数据字典或元数据管理工具（如Apache Atlas）统一命名规则。

为了命令更加规范与明确，你可以使用给字段统一加前缀的方式进行命名，如下所示：

* 维度字段：建议以“dim\_”开头，例如dim\_date（日期）、dim\_region（地区）。
* 指标字段：建议以“met\_”开头，例如met\_sales\_amt（销售额）、met\_order\_cnt（订单量）、met\_conversion\_rate（转化率）。
* 元数据字段：建议以“meta\_”开头，例如meta\_create\_time（创建时间）、meta\_update\_time（更新时间）。

### 2.2 注意事项

字段设计一般需要注意如下事项：

* 避免冗余：只保留业务需求所需的字段，避免无意义的字段堆砌。
* 类型准确：根据字段含义选择合适的数据类型，例如金额用decimal，计数用bigint。
* 默认值与空值：指标字段若为空，建议填充0；维度字段若为空，需明确业务含义。
* 注释完善：为每个字段添加详细注释，说明其含义、来源和计算逻辑。
* 分区设计：对于大表，建议按时间分区（如按天或按月），提升查询效率。
* 唯一约束：设计组合主键（如dim\_date+dim\_region），避免重复数据。

# 03 最后小结

ADS层作为数据仓库面向业务应用的“最后一公里”，它直接影响用户体验，因此必须重视其设计的合理性与执行性能。除此之外，ADS层也是需求变更最为频繁的一层，您可以通过“影子表、新增表分区、或者使用Iceberg来管理表结构"。

---

更多相关的内容，请参考往期文章：

[数据仓库分层实践：DWS层设计全面解析](https://mp.weixin.qq.com/s?__biz=MzU3NDEzODU4MA==&mid=2247483975&idx=1&sn=49421ea5c617f0ba2b85c26f2f7864d7&scene=21#wechat_redirect)

[数据仓库分层实践：DWD层设计全面解析](https://mp.weixin.qq.com/s?__biz=MzU3NDEzODU4MA==&mid=2247483968&idx=1&sn=889406779dc5469c4d56732fe3490d91&scene=21#wechat_redirect)

[数据仓库分层实践：ODS层设计全面解析](https://mp.weixin.qq.com/s?__biz=MzU3NDEzODU4MA==&mid=2247483961&idx=1&sn=ab81d98eea4f6b84b6415b31a0aad195&scene=21#wechat_redirect)

[数仓分层：ODS、DWD、DWS、DWT、ADS...这些层级你了解多少？](https://mp.weixin.qq.com/s?__biz=MzU3NDEzODU4MA==&mid=2247483933&idx=1&sn=3acb3f816c6a4b85bcc5c8d83af3927f&scene=21#wechat_redirect)

[数据仓库维度建模：了解维度模型与建模步骤](https://mp.weixin.qq.com/s?__biz=MzU3NDEzODU4MA==&mid=2247483913&idx=1&sn=b83a86f82b9594c144a247a279a86699&scene=21#wechat_redirect)

[数据仓库维度建模：深入了解维度表](https://mp.weixin.qq.com/s?__biz=MzU3NDEzODU4MA==&mid=2247483904&idx=1&sn=2066ba7b7388f210b5b67cd5342d5a63&scene=21#wechat_redirect)

[数据仓库维度建模：深入了解事实表](https://mp.weixin.qq.com/s?__biz=MzU3NDEzODU4MA==&mid=2247483884&idx=1&sn=791d78ba4d960d51a08f2977c31070a0&scene=21#wechat_redirect)