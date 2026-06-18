---
title: 腾讯面试：Paimon自动分区清理与快照清理机制是怎么样的？哪个先清理？
author: 大数据技能圈
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491577&idx=1&sn=9e2482d8e421adab5ceaad7c788cd231&chksm=c1306dc1d98efcf670b05900d107e66d51366751d8f523798592fd3587735ff502b6915b2cb4&mpshare=1&scene=24&srcid=0715aQ46cLw1HP0P1w8I2FXy&sharer_shareinfo=450ea537fed6bbf6476ae40a5d4faebd&sharer_shareinfo_first=5a51eb5738852e90c797864f02b5c682#rd
---

01

**加入知识星球《随川陪你学大数据》将获得以下权益**

扫描二维码加入星球，随加入人数增加，逐步涨价，现在就是最优惠的

01

**Paimon 自动分区清理与快照清理机制详解**

## 引言：数据湖存储的生命周期管理挑战

在实时数据湖架构中，Apache Paimon以其高效的流批一体能力成为核心存储层。随着数据持续写入，快照文件和历史分区不断累积，不仅占用大量存储空间，还会降低查询性能。**自动分区清理**与**自动快照清理**是Paimon实现存储优化的两大核心机制，二者协同工作，共同保障数据湖的高效运行。本文将深入解析这两种机制的原理、关联关系、配置方法及最佳实践，为数据湖运维提供全面指导。

## 核心概念：快照与分区的定义与作用

### 快照（Snapshot）：数据版本的时间切片

**快照**是Paimon表在某个时间点的状态快照，记录了该时刻表的完整数据视图。每个快照包含以下关键信息：

* 对应的Schema文件
* 清单列表（manifest list），记录数据文件的增删变更
* 生成时间戳及元数据

快照的核心作用是支持**时间旅行（Time Travel）**，用户可通过指定快照ID查询历史数据。例如，通过`SELECT * FROM table TIMESTAMP AS OF '2024-07-01 00:00:00'`访问特定时间点的表状态。

### 分区（Partition）：数据的逻辑划分单元

**分区**是Paimon借鉴Apache Hive的逻辑划分机制，将表数据按分区键（如时间、地域）拆分为多个独立目录。分区的主要价值在于：

* 减少查询扫描范围，提升读取效率
* 实现数据生命周期的精细化管理
* 支持按分区并行写入和处理

例如，按`dt`（日期）分区的表，数据会存储在`dt=20240701`、`dt=20240702`等子目录中。

## 自动快照清理：版本管理的核心机制

### 快照清理的触发逻辑

Paimon的快照清理由写入作业在提交新数据时自动执行，触发条件基于以下三个表属性的组合判断：

| 参数名称 | 数据类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `snapshot.num-retained.min` | Integer | 10 | 至少保留的快照数量，即使已超过`time-retained`也不会删除 |
| `snapshot.num-retained.max` | Integer | 2147483647 | 最多保留的快照数量，超过此值时触发清理 |
| `snapshot.time-retained` | Duration | 1h | 快照的最长保留时间，超过此时间且数量超过`min`时触发清理 |

**清理规则**：当快照总数超过`num-retained.max`，或最早快照的生成时间超过`time-retained`时，系统会删除最旧的快照，直至满足以下条件：

* 剩余快照数量 ≤ `num-retained.max`
* 所有剩余快照的生成时间均未超过`time-retained`
* 且至少保留`num-retained.min`个快照

### 快照清理的工作流程

1. **标记过期快照**

   ：写入作业提交时，检查当前快照列表，筛选出符合清理条件的快照。
2. **删除快照元数据**

   ：删除过期快照的JSON文件（位于`snapshot`目录下）。
3. **级联删除数据文件**

   ：删除仅被过期快照引用的数据文件（`.orc`/`.parquet`）和清单文件（`manifest`）。

**注意**：若数据文件同时被未过期的快照引用，则不会被删除。这确保了时间旅行查询的正确性。

### 配置样例：基础快照清理策略

#### 1. 创建表时配置快照清理

```
```
CREATETABLE user_behavior (  
    user_id BIGINT,  
    item_id BIGINT,  
    behavior STRING,  
    dt STRING,  
PRIMARYKEY(user_id, dt)NOT ENFORCED  
)WITH(  
'snapshot.num-retained.min'='5',-- 至少保留5个快照  
'snapshot.num-retained.max'='20',-- 最多保留20个快照  
'snapshot.time-retained'='24h',-- 快照保留24小时  
'partition.expiration-time'='7d'-- 分区过期时间（后续详解）  
);
```
```

#### 2. 动态修改快照清理策略

```
```
ALTERTABLE user_behavior SET(  
'snapshot.time-retained'='48h',-- 调整为保留48小时  
'snapshot.num-retained.max'='30'-- 最多保留30个快照  
);
```
```

## 自动分区清理：数据生命周期的精细化控制

### 分区清理的核心参数

分区清理通过以下参数定义过期规则，仅对分区表生效：

| 参数名称 | 数据类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `partition.expiration-time` | Duration | 无（需显式配置） | 分区的存活时间阈值，超过此时间则标记为过期 |
| `partition.timestamp-pattern` | String | 无（默认使用第一个分区字段） | 从分区值提取时间字符串的格式串，使用`$分区列名`引用分区字段 |
| `partition.timestamp-formatter` | String | `yyyy-MM-dd HH:mm:ss` /`yyyy-MM-dd` | 时间字符串转时间戳的格式，兼容Java `DateTimeFormatter` |
| `partition.expiration-strategy` | String | `values-time` | 过期策略：`values-time`（基于分区值时间）/`update-time`（基于最后更新时间） |

### 分区清理的执行逻辑

#### 1. 分区过期判断

**values-time策略**（默认）：

* 通过`timestamp-pattern`将分区值转换为时间字符串。例如，分区`dt=20240701`，配置`'partition.timestamp-pattern' = '$dt'`，`'partition.timestamp-formatter' = 'yyyyMMdd'`，则提取时间为`2024-07-01 00:00:00`。
* 分区存活时长 = 当前系统时间 - 提取的时间戳，若超过`expiration-time`则标记过期。

**update-time策略**：

* 基于分区的最后更新时间（由Paimon自动记录）判断是否过期，无需配置`timestamp-pattern`和`formatter`。适用于分区值非日期格式的场景（如`region=华北`）。

#### 2. 分区的逻辑删除与物理删除

* **逻辑删除**

  ：分区过期后，最新快照将不再包含该分区的数据，查询时无法访问，但物理文件仍存在。
* **物理删除**

  ：仅当包含该分区的所有快照均过期后，分区数据文件才会被物理删除。这是Paimon保障数据一致性的关键机制。

### 配置样例：多场景分区清理策略

#### 场景1：单分区字段（dt），按日期过期

```
```
CREATETABLE sales (  
    order_id BIGINT,  
    amount DECIMAL(10,2),  
    dt STRING,-- 分区列，格式如'20240701'  
PRIMARYKEY(order_id, dt)NOT ENFORCED  
) PARTITIONED BY(dt)  
WITH(  
'partition.expiration-time'='30d',-- 分区保留30天  
'partition.timestamp-pattern'='$dt',-- 从dt字段提取时间  
'partition.timestamp-formatter'='yyyyMMdd',-- dt格式为年月日  
'partition.expiration-strategy'='values-time'-- 基于分区值时间  
);
```
```

#### 场景2：多分区字段（year, month, day）

```
```
CREATETABLE logs (  
    log_id BIGINT,  
    content STRING,  
yearINT,  
monthINT,  
dayINT,  
PRIMARYKEY(log_id,year,month,day)NOT ENFORCED  
) PARTITIONED BY(year,month,day)  
WITH(  
'partition.expiration-time'='90d',-- 分区保留90天  
'partition.timestamp-pattern'='$year-$month-$day',-- 组合为'2024-07-01'  
'partition.timestamp-formatter'='yyyy-MM-dd'-- 匹配组合后的格式  
);
```
```

#### 场景3：非日期分区，基于更新时间过期

```
```
CREATETABLE device_status (  
    device_id STRING,  
status STRING,  
    region STRING,-- 分区列，如'华北'、'华东'  
PRIMARYKEY(device_id, region)NOT ENFORCED  
) PARTITIONED BY(region)  
WITH(  
'partition.expiration-time'='7d',-- 分区保留7天  
'partition.expiration-strategy'='update-time'-- 基于最后更新时间  
);
```
```

## 自动分区清理与快照清理的协同关系

### 依赖关系：分区清理依赖快照清理

分区数据的物理删除是**快照清理的副产品**。即使分区已逻辑过期（超过`expiration-time`），只要仍有未过期的快照引用该分区，其数据文件就不会被删除。只有当所有包含该分区的快照均过期并被清理后，分区文件才会被物理删除。

**示例**：  
某分区`dt=20240601`配置`expiration-time=30d`，于7月1日逻辑过期。若此时仍有6月15日生成的快照（`time-retained=30d`，将于7月15日过期）引用该分区，则该分区的物理删除需等待至7月15日快照清理后执行。

### 冲突与协调：参数配置的联动性

1. **快照保留时间过短**：若`snapshot.time-retained`小于`partition.expiration-time`，可能导致分区未逻辑过期但快照已清理，此时分区数据文件会被提前删除，引发查询异常。

   **解决方案**：确保`snapshot.time-retained` ≥ `partition.expiration-time`，例如分区保留7天，则快照至少保留7天。
2. **快照数量过多**：若`snapshot.num-retained.max`设置过大，可能导致大量旧快照引用历史分区，阻碍分区物理删除。

   **解决方案**：结合业务查询需求，合理设置`num-retained.max`，避免快照无限制累积。

## 手动清理与运维工具

### 清理废弃文件（Orphan Files）

由于作业失败、中断等异常情况，Paimon表目录中可能遗留未被任何快照引用的临时文件（如`.tmp`、未提交的日志文件），需通过`remove_orphan_files`存储过程手动清理：

```
```
-- 清理创建时间超过1天的废弃文件（默认行为）  
CALL`paimon_catalog`.sys.remove_orphan_files('mydb.user_behavior');  
  
-- 清理指定时间前的废弃文件（如清理2024-07-01前创建的文件）  
CALL`paimon_catalog`.sys.remove_orphan_files('mydb.user_behavior','2024-07-01 00:00:00');
```
```

### 手动触发快照/分区清理

若需立即执行清理（如存储空间紧急），可通过`expire_snapshots`和`expire_partitions`存储过程手动触发：

```
```
-- 手动触发快照清理，保留最多10个快照，且仅保留24小时内的快照  
CALL sys.expire_snapshots(  
table=>'mydb.sales',  
    retain_max =>10,  
    older_than =>TIMESTAMP'2024-07-14 00:00:00'  
);  
  
-- 手动触发分区清理，指定过期时间为30天  
CALL sys.expire_partitions(  
table=>'mydb.logs',  
    expiration_time =>'30d',  
    timestamp_pattern =>'$year-$month-$day',  
    timestamp_formatter =>'yyyy-MM-dd'  
);
```
```

## 最佳实践与注意事项

### 1. 参数配置的业务适配

* **实时场景**

  （如监控数据）：快照保留短（1-2小时），分区保留短（7-15天），配置`'snapshot.time-retained' = '2h'`，`'partition.expiration-time' = '7d'`。
* **批处理场景**

  （如报表数据）：快照保留长（7-30天），分区保留长（90-180天），配置`'snapshot.time-retained' = '30d'`，`'partition.expiration-time' = '180d'`。

### 2. 版本兼容性

* 自动分区清理和快照清理功能需**Paimon 0.4+** 及 **Flink VVR 8.0.5+** 支持，低版本需升级引擎。
* Paimon 0.8.0+ 新增`changelog.time-retained`参数，支持变更日志与快照生命周期解耦，进一步优化存储效率。

### 3. 监控与告警

* 定期监控快照数量（`snapshot`目录文件数）和分区大小，避免存储膨胀。
* 配置告警阈值：当快照数量超过`num-retained.max`的80%或分区存储占比超过阈值时触发告警。

### 4. 结合Tag功能保存关键历史状态

对于需长期保留的重要数据版本（如月末结算数据），可通过**Tag**功能固化快照，避免被自动清理：

```
```
-- 创建Tag固化快照（基于最新快照）  
CREATETABLE user_behavior WITH(  
'tag.automatic-creation'='process-time',-- 自动创建Tag  
'tag.creation-period'='daily',-- 每天创建一个Tag  
'tag.num-retained-max'='90'-- Tag保留90天  
);
```
```

Paimon的自动分区清理与快照清理是实现数据湖存储优化的核心机制。**快照清理通过控制版本数量和保留时间管理数据文件生命周期，分区清理则基于时间策略实现逻辑数据的精细化淘汰**，二者通过“快照引用”机制紧密联动，共同保障数据湖的高效、低成本运行。

以上。

本详细文档已放入知识星球《随川陪你学大数据》，可扫码获取

获取更多信息，关注大数据技能圈

**欢迎添加作者交流**

## 推荐阅读系列文章

* [腾讯面试：Spark内存如何优化？包含哪几个方面？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491167&idx=1&sn=d62e305f6803f484aa40469393b1fdb8&scene=21#wechat_redirect)
* [腾讯面试：Flink Checkpoint与两阶段提交对下游算子拉取数据时机的影响](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491572&idx=1&sn=cf86c4bc3430d9d496528ef86a55532a&scene=21#wechat_redirect)
* [超强整理：Paimon最新学习文档（十一章）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491570&idx=1&sn=cd99032a86449361ab503844e37af5d3&scene=21#wechat_redirect)
* [腾讯面试：请详细描述Paimon如何基于LSM树实现高吞吐写入和高效查询？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491536&idx=1&sn=01cb739322d9fd0e5ef4530e6970187c&scene=21#wechat_redirect)
* [腾讯面试：Flink100G大状态如何优化？有哪些参数可以调整？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491532&idx=1&sn=18c1952668c000328b2fa3081fc80906&scene=21#wechat_redirect)
* [阿里面试：Paimon QPS太低怎么优化？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491526&idx=1&sn=d6364e30ac6442c02c7a147337c4d44a&scene=21#wechat_redirect)
* [腾讯面试：Flink出现反压如何排查？有哪些参数可以调整？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491520&idx=1&sn=b8ab27aa17c78653d6598faff0bc85e9&scene=21#wechat_redirect)
* [超强总结：Iceberg可以优化的方方面面](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491505&idx=1&sn=e385a35b8d736b2cc639225b8487563c&scene=21#wechat_redirect)
* [阿里面试：Hudi，Iceberg，Paimon之间的差异有哪些？该如何选择？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491499&idx=1&sn=dc6844323e421ff97157ffe6de5b9bbb&scene=21#wechat_redirect)
* [阿里面试：如果让你负责大数据平台的架构，需要考虑哪些点？如何设计？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491478&idx=1&sn=baf72d7547630119ff4da6c0ba6b1239&scene=21#wechat_redirect)
* [阿里面试：请详细解释一下Flink内存管理，具体有哪些参数可以调整？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491468&idx=1&sn=524bcce4dc664fe78d4442daa6ab21f6&scene=21#wechat_redirect)
* [腾讯面试：介绍一下Doris问题排查思路，有没有总结过相关文档？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491462&idx=1&sn=24b0982030c010dd904b65de15bd17df&scene=21#wechat_redirect)
* [这篇文章把Paimon和Fluss的关系给彻底说清楚了](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491452&idx=1&sn=f6e98294672db035964f347a290f16f4&scene=21#wechat_redirect)
* [腾讯面试：Doris 物化视图的使用场景是怎么样的，有哪几种数据更新方式？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491386&idx=1&sn=d29b14d6912d0efdb941ae5e1dbd1dfb&scene=21#wechat_redirect)
* [腾讯面试：详细介绍Spark的Shuffle阶段数据从输入到输出经历了哪些步骤？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491384&idx=1&sn=5ab414f60fda52a3b9b88e1a7f2cdecd&scene=21#wechat_redirect)
* [腾讯面试：数仓分层架构是怎么样的？为什么要这样设计？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491372&idx=1&sn=97dfb7d58f8b889a8cbaeb4229091e98&scene=21#wechat_redirect)
* [蚂蚁面试：Flink并行度、算子、算子链、Slot、Slot共享组之间的关系是什么？如何设置能够使资源利用最大化？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491363&idx=1&sn=17d02a075e33592aabc517f974e87a02&scene=21#wechat_redirect)
* [网易面试：Hudi、Iceberg、Paimon有什么异同点？如何选型？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491351&idx=1&sn=df26d44b5c9ac16eb3be5a2f3027d5f6&scene=21#wechat_redirect)
* [Flink 反压问题深度剖析与解决方案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491343&idx=1&sn=a898b2172b5a694dfa03e356758ff08a&scene=21#wechat_redirect)
* [小米面试：Paimon Join用法有哪些？大规模数据场景下如何优化 Join 性能？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491218&idx=1&sn=b5aa336a2fb6a37d1093ec6318e8e39d&scene=21#wechat_redirect)
* [蚂蚁面试：Kafka如何做压测？如何保证系统稳定？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491146&idx=1&sn=e2c4a1e298aec2b6d4d42cffb2f331ef&scene=21#wechat_redirect)
* [字节面试：Flink如何做压测？如何保证系统稳定？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491131&idx=1&sn=6bac85b9e17620b49b37ebd3224b3a83&scene=21#wechat_redirect)
* [Flink内存调优指南（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=2&sn=cccaa1289c4581a2dd407d21a8c5ae09&scene=21#wechat_redirect)附500页16万字答案[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=2&sn=cccaa1289c4581a2dd407d21a8c5ae09&scene=21#wechat_redirect)
* [Zookeeper 经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[附1400页21万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)
* [Hbase 经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491195&idx=1&sn=0c93336f246fde472c336ce764623a75&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491209&idx=1&sn=adb85c2a4da09364764146cb95d6f4c4&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491195&idx=1&sn=0c93336f246fde472c336ce764623a75&scene=21#wechat_redirect)[附550页12万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491195&idx=1&sn=0c93336f246fde472c336ce764623a75&scene=21#wechat_redirect)
* [Hive经典面试题200道（附8万字420页答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491138&idx=1&sn=f1acb3164d5f0f7e8be7f2037f4a838b&scene=21#wechat_redirect)
* [Kafka 经典面试题200道（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)[附8万字420页答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491138&idx=1&sn=f1acb3164d5f0f7e8be7f2037f4a838b&scene=21&token=1460012704&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)
* [Spark经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491174&idx=1&sn=93767125faa648bcd31b9ecf9442b855&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)[附500页8万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491106&idx=1&sn=d412a81e6d5742c3e4acf6a680761139&scene=21&token=1460012704&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491159&idx=1&sn=609c798a7017905162e6e823c2d3734b&scene=21#wechat_redirect)
* [ElasticSearch经典面试题200道（附400页12万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491122&idx=1&sn=cb7f3563a8082d248b1a0b768bd4d567&scene=21#wechat_redirect)
* [FlinkCDC经典面试题200道（附500页8万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491106&idx=1&sn=d412a81e6d5742c3e4acf6a680761139&scene=21#wechat_redirect)
* [StarRocks](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect) [经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect)[附550页12万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491091&idx=1&sn=18d4c7f514cb4fa85275e0d6abdfce58&scene=21#wechat_redirect)
* [Flink源码分析 经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[（](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)[附1200页32万字答案](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21&token=1757226191&lang=zh_CN#wechat_redirect)[）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491057&idx=1&sn=08a2914ec0a44196c248d093488cdaf3&scene=21#wechat_redirect)
* [FlinkSQL 经典面试题200道（附1200页32万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491051&idx=1&sn=808fcc2c97f11cb71024b7208d4b7d98&scene=21#wechat_redirect)
* [Paimon经典面试题200道题（附500页16万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491044&idx=1&sn=f11db4583012987748f532d11ebaf1f0&scene=21#wechat_redirect)
* [Hudi经典面试题](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491340&idx=1&sn=fbfde6a8f265acaa2eb3cf320fc635fc&scene=21#wechat_redirect)[200道（附1050页39万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491037&idx=1&sn=24bcca607e01daae65783775dcc3716e&scene=21#wechat_redirect)
* [Doris经典面试题200道（附1050页39万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491037&idx=1&sn=24bcca607e01daae65783775dcc3716e&scene=21#wechat_redirect)
* [Flink经典面试题200道（附1060页26万字答案）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491335&idx=1&sn=97f9e76a359119d51ff26fd177140437&scene=21#wechat_redirect)
* [建议收藏 | Kafka 实战文章总结](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490811&idx=1&sn=8e756190799e98e0e017b5994b6d0c3b&scene=21#wechat_redirect)
* [建议收藏 | Dinky系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489859&idx=1&sn=6b59becc16f653b609d01090e40f9e20&chksm=c02962dcf75eebcabfa43a1e3e4a1b210df6ac0725685a9087984261626223cc339c0c363a24&scene=21#wechat_redirect)
* [建议收藏 | Flink系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489608&idx=1&sn=c055844c7ac20eabbe11e331f0d629c2&chksm=c02963d7f75eeac15892c32660c1e90e000ad72e66ab97ed4051273bb5e3f6299998d6c81097&scene=21#wechat_redirect)
* [建议收藏 | Flink CDC 系列总结篇](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489988&idx=1&sn=c57b867645834257e8567a6e671273ff&chksm=c029625bf75eeb4d29d3f43d1b845aec3a7d176cfbceddd16ae6d195d4d96fa5cf6bfa9c362d&scene=21#wechat_redirect)
* 建议收藏 | [Doris实战文章合集](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490251&idx=1&sn=ae993d282a0a3cd968c3d6a19f79dce8&chksm=c0296154f75ee8428ee77a8e0c86ca08648967e7a68987aaf79d20bdbd481f0ab6d1d2729b53&scene=21#wechat_redirect)
* [建议收藏 | Paimon 实战文章总结](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490466&idx=1&sn=c3bdd1c25d72ba89186d4ed63634f7b3&scene=21#wechat_redirect)
* [建议收藏 | Fluss 实战文章总结](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490616&idx=1&sn=20c60ca77763b23d7c5b3a8785f58007&scene=21#wechat_redirect)
* 建议收藏 | [Seatunnel 实战文章系列合集](http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490357&idx=1&sn=61ae7c852b2a41e192c0797cc28fd182&chksm=c02960aaf75ee9bcd422d82ceb80362a0959df74ee7d336e0015c51b3eb6eb1a6d6774e99483&scene=21#wechat_redirect)
* 建议收藏 | [实时离线输数仓（数据湖）总结篇](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488973&idx=1&sn=42d2f8235c822030f187c0d49deb54f5&scene=21#wechat_redirect)
* [建议收藏 | 实时离线数仓实战第一阶段总](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247488426&idx=1&sn=6fe3666f5157c651c08a0c8862c1efad&scene=21#wechat_redirect)结
* [超700star！电商项目数据湖建设实战代码 ，拿来即用！](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490491&idx=1&sn=9162eb30018ad3a0c5663afe1d1b0c85&scene=21#wechat_redirect)
* [从0到1建设电商项目数据湖实战教程](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490348&idx=1&sn=e7c6b0d224fea9560218213e7b2e388c&scene=21#wechat_redirect)
* [推荐一套开源电商项目数据湖建设实战代码](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247489555&idx=1&sn=e923d58dbabca58d00d76cead2a9580b&scene=21#wechat_redirect)