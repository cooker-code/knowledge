---
title: 大厂数据优化：Null值的高阶处理
author: 熊大数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486357&idx=1&sn=0fda23423ce8097ebe367614e1d4869d&chksm=e998b6484baa76540b8ec214e3393fd8fdb84d704d4ea49d5716c82a17614fc90a67bb0d7e4b&mpshare=1&scene=24&srcid=052100f9AZuvxoyOZ8bkCFBl&sharer_shareinfo=cff128218dc7efb6ddfc5e5ab32f853a&sharer_shareinfo_first=cff128218dc7efb6ddfc5e5ab32f853a#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL行列转换与空值处理|SQL行列转换与空值处理]]


我是大熊！某大厂数据部门负责人

将熊大数据设为“星标⭐”，第一时间推送

**Null值处理只会：COALESCE ？**

**所以今天内容比较多，目录如下：**

一、常规处理流程

二、预处理

三、高阶优化策略（复杂场景）

四、DQC校验规则（全量&抽样）

五、成本治理加速

**-1-**

**常规处理流程**

**业务语义层考量**

**首先我们的第一步是确认Null值的具体含义：**

* **NULL是否代表"未知"、"未发生"或"数据缺失"？**
* **是否需要将NULL转化为特定默认值（如"空字符串"、"0"、"N/A"或"-1"）？**

一般来说：存在即合理，我列举一些业务允许null值的场景：

* 用户主动未填写

  用户资料的个人简介字段
* 未触发的业务行为

  直播间的“结束时间”（未下播）
* 关联关系未简历

  用户等级勋章表ID（未佩戴就是null）
* 技术埋点限制

  Android端的版本兼容性缺失导致上报为null

当然也有不合理的null值场景

* 关键主键缺失

  直播间的room\_id为null
* 业务强依赖字段为空

  礼物打赏记录的支付金额为null
* 时间轴断裂

  用户观看行为表，进入直播间的时间为null
* 维度属性异常缺失

  A主播表的签约MCN机构ID为null

我梳理了一个常规的处理流程，以便于大家遇到这种情况有一个更全面的思考

**-2-**

**预处理**

****数据应用场景****

将登录前同一设备产生的行为数据关联到最终用户ID：

**→ 设备匿名阶段（无user\_id）**

**→ 登录事件（带user\_id）**

**→ 登录后行为（包含user\_id）**

### Step1：ODS预处理

```
-- 创建设备登录标记表（记录首次登录时间）CREATE TABLE tmp_device_first_login ASSELECT     device_id,    MIN(event_time) AS first_login_time,    MAX(user_id) AS user_id       -- 假设同一设备首次登录的user_id唯一FROM     ods_raw_user_behaviorWHERE     action_type = 'login'    AND user_id IS NOT NULLGROUP BY     device_id;
```

**Step2：DWD层补全逻辑**

```
-- 创建用户行为表，基于设备首次登录时间回填user_idCREATE TABLE dwd_user_behavior ASSELECT    b.event_time,    b.device_id,    -- 关键补全逻辑：基于设备首次登录时间回填user_id    CASE        WHEN a.first_login_time IS NOT NULL          AND b.event_time <= a.first_login_time        THEN a.user_id   -- 登录前行为打标        ELSE b.user_id   -- 维持原值    END AS user_id_clean,    b.page_url,    b.action_typeFROM    ods_raw_user_behavior bLEFT JOIN    tmp_device_first_login aON    b.device_id = a.device_id;
```

```
-3-

高阶优化策略（复杂场景）
```

场景1：

设备被多个账户复用（如手机），我们采用最近一次非空登入策略

```
-- 使用LAST_VALUE函数获取每个设备最新的非空user_idSELECT    event_time,    device_id,    LAST_VALUE(user_id, true) OVER (        -- 忽略空值        PARTITION BY device_id        ORDER BY event_time        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW    ) AS user_id_clean_latestFROM    ods_raw_user_behavior;
```

场景2：

**跨设备登录合并（手机→PC），构建用户-设备映射表**

```
-- 创建用户设备映射表，收集每个用户的关联设备CREATE TABLE dim_user_device_mapping ASSELECT    user_id,    COLLECT_SET(device_id) AS related_devices      -- 合并多端设备FROM    (SELECT            user_id,device_id        FROM            ods_raw_user_behavior        WHERE user_id IS NOT NULL        GROUP BY            user_id,device_id    )GROUP BY user_id;
```

行为数据扩展关联

```
-- 查询用户行为表，通过设备映射表关联跨设备用户IDSELECT    b.*,    m.user_id AS cross_device_user_id   -- 跨设备用户IDFROM    dwd_user_behavior bJOIN    dim_user_device_mapping mON ARRAY_CONTAINS(m.related_devices, b.device_id);
```

#### **``` -4- DQC校验规则（全量&抽样） ``` 全量检测： 检查登录后行为是否100%补全ID ``` -- 查询用户表，统计总函数和user_id_clean为空的数量SELECT    COUNT(*) AS total_actions,   -- 总行为数    SUM(        CASE            WHEN user_id_clean IS NULL THEN 1 ELSE 0        END    ) AS null_count   -- user_id_clean为空的记录数FROM    dwd_user_behaviorWHERE    device_id IN (        SELECT device_id        FROM tmp_device_first_login    ); ``` **抽样检测** 如果数据量大，查询用户行为表，获取设备ID、清理后的用户ID以及登录用户ID，并进行抽样 ``` SELECT    u.device_id,    u.user_id_clean,    a.user_id AS login_user_idFROM dwd_user_behavior uJOIN    tmp_device_first_login aON u.device_id = a.device_idTABLESAMPLE (0.1PERCENT); ```** **-5-** **成本治理加速** ****场景1：降低存储成本+加速扫描**** **问题情况：**某直播用户标签表 user\_tags 中，last\_live\_time 字段 NULL 占比超 60%，全表扫描延迟高 **优化方案**：

```
-- 建表时定义默认值（Hive Metastore）CREATE TABLE optimized_user_tags (  user_id BIGINT,  last_live_time TIMESTAMP COMMENT '采用1970-01-01表示未开播') USING PARQUETTBLPROPERTIES (  'parquet.compression'='ZSTD',  -- 启用高效压缩  'parquet.enable.dictionary'='true'  -- 强制字典编码);-- 数据加载时转换（若原始数据存在NULL）INSERT INTO optimized_user_tagsSELECT   user_id,  COALESCE(last_live_time, TIMESTAMP'1970-01-01 00:00:00') AS last_live_time FROM raw_user_tags;-- 查询时反向转换NULL语义SELECT   user_id,  CASE WHEN last_live_time = '1970-01-01' THEN NULL ELSE last_live_time END AS real_last_live_timeFROM optimized_user_tags;
```

效果验证

```
# 原始表（含NULL）hdfs dfs -du -h /user/hive/warehouse/raw_user_tags# 输出: 4.7G# 优化后表hdfs dfs -du -h /user/hive/warehouse/optimized_user_tags  # 输出: 1.8G (压缩率提升61%)# 执行计划对比（原始 vs 优化）Spark.sql("EXPLAIN EXTENDED SELECT COUNT(*) FROM user_tags WHERE last_live_time > '2023-01-01'").show(truncate=false)# 优化后观察到 Parquet 过滤器下推（无需读全量数据）
```

****场景2：规避Shuffle倾斜****

**问题情况：**分析直播间打赏TOP用户时，`gift_records` 中 `sender_id` 30%为 NULL，导致 GROUP BY 卡在最后两个 Task

**优化方案**：

```
# 策略：给NULL sender_id分配随机虚拟值分散计算virtual_gift_df = (spark.table("gift_records")  .withColumn("skew_sender_id",      F.when(F.col("sender_id").isNull(),             F.concat(F.lit("NULL_"), F.floor(F.rand() * 100)))      .otherwise(F.col("sender_id"))))# 两阶段聚合（预聚合 + 最终聚合）result = (virtual_gift_df  .groupBy("skew_sender_id")  .agg(F.sum("gift_value").alias("partial_sum"))  .groupBy(F.when(F.col("skew_sender_id").startswith("NULL_"), None)           .otherwise(F.col("skew_sender_id")).alias("sender_id"))  .agg(F.sum("partial_sum").alias("total_gift_value"))  .orderBy(F.desc("total_gift_value")))result.explain()# 观察 Exchange 阶段分区数是否均匀
```

**执行计划验证**：

```
== Physical Plan ==*(5) Sort [total_gift_value#10L DESC NULLS LAST], true, 0+- Exchange rangepartitioning(total_gift_value#10L DESC NULLS LAST, 200)   +- *(4) HashAggregate(keys=[CASE WHEN startsWith(skew_sender_id#8, NULL_) THEN NULL ELSE skew_sender_id#8 END AS sender_id#9], functions=[sum(partial_sum#7L)])      +- Exchange hashpartitioning(CASE WHEN startsWith(skew_sender_id#8, NULL_) THEN NULL ELSE skew_sender_id#8 END, 200)         +- *(3) HashAggregate(keys=[skew_sender_id#8], functions=[sum(gift_value#2L)])            +- Exchange hashpartitioning(skew_sender_id#8, 200)               +- *(2) Project [gift_value#2L, ...skew_sender_id#8]                  +- *(1) Filter ...
```

**优化收益**：

原有单个 Task 处理 3 千万行 NULL 数据，优化后均匀散布至 100 个虚拟桶，Shuffle 时间从 12min 下降至 43s

**场景3：内存优化**

**问题情况：实时计算直播间在线人数时，`user_status` 中 `last_active_time为`NULL 过多导致堆内存吃紧**

**优化方案**：

启用列式压缩后，TIMESTAMP 类型用字典编码 + RLE 压缩，NULL 被压缩为标记位，内存占用下降60%

```
// 启用列式内存编码（默认OFF）spark.conf.set("spark.sql.inMemoryColumnarStorage.enabled", true)spark.conf.set("spark.sql.inMemoryColumnarStorage.compressed", true)  val activeUsers = spark.read.parquet("user_status")  .selectExpr("user_id", "COALESCE(last_active_time, current_timestamp()) AS safe_active_time")activeUsers.persist(StorageLevel.MEMORY_ONLY)  // 触发列式缓存
```

**优化收益**：

```
// 查看存储大小 原始: 12GB → 优化后: 4.8GBprintln(activeUsers.storageLevel.useMemory)  
```

**-6-**

**小结**

1. **早介入**

   在数仓设计阶段与业务、上游开发同步NULL处理逻辑。
2. **保一致**

   通过DWD层统一封装，避免各层逻辑分散。
3. **可追溯**

   通过日志或标记字段记录原始NULL状态，方便回溯分析。
4. **动态调整**

   定期复盘NULL规则，响应业务变化（如新增场景需区分历史NULL值）。

个人微信

**往期精彩文章合集**

【数据建模】

 [深度调研：外卖数仓的建设](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486319&idx=1&sn=899d4638bcff5c6bddc207f24e0b9100&scene=21#wechat_redirect)

 [数仓专家如何进行数据调研？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485454&idx=1&sn=0cb476d43665eea03750ab2562ba9ad9&scene=21#wechat_redirect)

 [从业务到数仓-网约车平台Gra建模设计](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485035&idx=1&sn=9ec02657a9d224e8ecbbb94ae5750918&scene=21#wechat_redirect)

【数据性能优化】

 [宇宙厂Flink优化：实时榜单最佳实践](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486294&idx=1&sn=4ab511381ff73b31107ada027281e2bd&scene=21#wechat_redirect)

 [SQL千亿数据膨胀OOM优化经验](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247484805&idx=1&sn=7082c25af5d666a54756007770303ee7&scene=21#wechat_redirect)

【面试经验】
 [明知我只写SQL，为何面试考动态规划](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485491&idx=1&sn=0e13cf91fc90fd2a58651b973367b689&scene=21#wechat_redirect)
 [宇宙厂数据岗 3-1 初面，等消息...](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486036&idx=1&sn=afc7122b4da23ca0d1ff1ebf97e066eb&scene=21#wechat_redirect)

【工作十年】

[做数开，别把路走窄了](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485682&idx=1&sn=4c77735ce24319509c116cd45417702e&scene=21#wechat_redirect)

[入职阿里，跟不上节奏](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486072&idx=1&sn=227bd9fbc857eebc51f43df75c80e6d4&scene=21#wechat_redirect)