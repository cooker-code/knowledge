---
title: 数据仓库分层设计：ODS/DWD/DWS/ADS到底该怎么划边界？
author: 数据与模型之美
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247484641&idx=1&sn=549ef56038554163b0d12343b2c11446&chksm=9706f5d0db982fb9eda567bcfaeaeb5f368f518dad85a4f20c60abdc9c5eaed308c926ff9f9c&mpshare=1&scene=24&srcid=08148tNuEpf7lDtoJFVZPwe4&sharer_shareinfo=582be0a4d77d00c293028fda0056afb1&sharer_shareinfo_first=582be0a4d77d00c293028fda0056afb1#rd
---

上一篇：[Doris vs StarRocks vs ClickHouse：新一代MPP引擎的终极对决](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247484607&idx=1&sn=6d27e96ad8a7498833372cc51a79a557&scene=21#wechat_redirect)

添加微信，备注资料获取更多福利

## 一、数据分层概述

在大数据领域，数据分层是数据仓库设计的核心环节，合理的分层设计能够显著提升数据管理的效率和质量。目前业界普遍采用的分层模型通常包含以下四个主要层次：

1. **ODS（Operational Data Store）**：操作数据层
2. **DWD（Data Warehouse Detail）**：数据仓库明细层
3. **DWS（Data Warehouse Summary）**：数据仓库汇总层
4. **ADS（Application Data Store）**：应用数据层

这种分层架构源于数据仓库理论中的Inmon和Kimball方法论，并在互联网行业实践中逐渐演化形成。合理的边界划分需要考虑数据流转、业务需求和技术实现三个维度的平衡。

## 二、各层详细定义与边界划分

### 1. ODS层：原始数据保留区

**核心职责**：

* 保持与源系统数据完全一致（结构+内容）
* 提供历史数据存储能力
* 作为数据故障恢复的基准点

**边界划分原则**：

* **数据内容**：保留源系统原始数据，不做业务逻辑处理
* **数据时效**：通常按天分区存储，保留一定时间周期（如3-6个月）
* **处理方式**：

+ 只进行必要的技术处理（编码转换、字段类型调整）
+ 不进行业务逻辑加工
+ 可增加数据抽取时间、来源系统等元数据

* **命名规范**：建议采用`ods_[源系统]_[表名]`格式

**典型数据**：

```
-- 用户订单ODS表示例  
CREATETABLE ods_mysql_orders (  
    order_id STRING COMMENT'订单ID',  
    user_id STRING COMMENT'用户ID',  
    order_amount DECIMAL(18,2) COMMENT'订单金额',  
    order_status INT COMMENT'订单状态',  
    create_time TIMESTAMP COMMENT'创建时间',  
    update_time TIMESTAMP COMMENT'更新时间',  
    etl_date DATE COMMENT'ETL处理日期'  
) PARTITIONED BY (dt STRING);
```

### 2. DWD层：企业级明细数据

**核心职责**：

* 提供业务过程最细粒度的数据
* 实现数据标准化和一致性
* 支持跨业务主题的关联分析

**边界划分原则**：

* **数据内容**：

+ 保持原子粒度（不聚合）
+ 完成数据清洗（去脏、补全、标准化）
+ 实现维度退化（常用维度字段冗余存储）

* **处理方式**：

+ 实施一致的编码规范（如性别统一为'M/F'）
+ 处理缓慢变化维（SCD）问题
+ 建立一致性维度

* **数据模型**：

+ 以事实表为核心
+ 采用星型或雪花模型

* **命名规范**：建议`dwd_[业务域]_[事实/维度]_[表名]`

**典型转换**：

```
-- ODS到DWD的转换示例  
INSERT OVERWRITE TABLE dwd_trade_orders_fact PARTITION(dt='${bizdate}')  
SELECT  
    order_id,  
    user_id,  
    CAST(order_amount AS DECIMAL(18,2)) AS order_amount,  
    CASE order_status   
        WHEN 1 THEN'paid'  
        WHEN 2 THEN'shipped'  
        ELSE'unknown'  
    ENDAS order_status,  
    create_time,  
    update_time,  
    -- 添加统一时间维度  
    TO_DATE(create_time) AS order_date  
FROM ods_mysql_orders  
WHERE dt = '${bizdate}'  
AND order_id IS NOT NULL;
```

### 3. DWS层：主题域汇总数据

**核心职责**：

* 面向分析主题的轻度汇总
* 提升常用查询性能
* 支持跨业务线的数据整合

**边界划分原则**：

* **数据内容**：

+ 按主题域组织数据（用户、商品、交易等）
+ 保留适当的时间粒度（日、周、月）

* **处理方式**：

+ 基于业务指标进行汇总
+ 处理指标口径一致性
+ 考虑汇总粒度的平衡（太细失去意义，太粗失去灵活性）

* **数据模型**：

+ 宽表模型为主
+ 包含常用衍生指标

* **命名规范**：建议`dws_[主题域]_[时间粒度]_[表名]`

**典型示例**：

```
-- 用户日汇总宽表  
CREATE TABLE dws_user_d (  
    user_id STRING COMMENT'用户ID',  
    register_date DATE COMMENT'注册日期',  
    -- 订单指标  
    order_count BIGINT COMMENT'订单数',  
    order_amount DECIMAL(18,2) COMMENT'订单金额',  
    -- 支付指标  
    payment_count BIGINT COMMENT'支付次数',  
    -- 登录行为  
    login_count BIGINT COMMENT'登录次数',  
    -- 维度属性  
    user_level STRING COMMENT'用户等级'  
) PARTITIONED BY (dt STRING);  
  
-- 汇总计算逻辑  
INSERT OVERWRITE TABLE dws_user_d PARTITION(dt='${bizdate}')  
SELECT  
    u.user_id,  
    u.register_date,  
    -- 订单指标  
    COUNT(DISTINCT o.order_id) AS order_count,  
    SUM(o.order_amount) AS order_amount,  
    -- 支付指标  
    COUNT(DISTINCT p.payment_id) AS payment_count,  
    -- 登录行为  
    l.login_count,  
    -- 用户属性  
    u.user_level  
FROM dim_user u  
LEFT JOIN (  
    SELECT user_id, order_id, order_amount   
    FROM dwd_trade_orders_fact   
    WHERE dt = '${bizdate}'AND order_status = 'paid'  
) o ON u.user_id = o.user_id  
LEFT JOIN (  
    SELECT user_id, COUNT(*) AS login_count  
    FROM dwd_event_login_fact  
    WHERE dt = '${bizdate}'  
    GROUPBY user_id  
) l ON u.user_id = l.user_id  
-- 其他关联...  
GROUPBY u.user_id, u.register_date, u.user_level, l.login_count;
```

### 4. ADS层：应用数据服务

**核心职责**：

* 面向具体应用场景的数据输出
* 满足最终业务需求
* 支持高性能数据访问

**边界划分原则**：

* **数据内容**：

+ 高度聚合的指标数据
+ 面向报表、API等具体应用

* **处理方式**：

+ 实现最终业务逻辑
+ 处理指标间的计算关系
+ 考虑查询性能优化

* **数据模型**：

+ 完全面向应用场景
+ 可能采用特殊存储格式（如MySQL关系表、Redis缓存等）

* **命名规范**：建议`ads_[应用名称]_[表名]`

**典型示例**：

```
-- 电商管理后台报表  
CREATE TABLE ads_ec_admin_daily (  
    report_date DATE COMMENT'报告日期',  
    -- 用户指标  
    new_user_count BIGINT COMMENT'新增用户数',  
    active_user_count BIGINT COMMENT'活跃用户数',  
    -- 交易指标  
    order_count BIGINT COMMENT'订单数',  
    success_order_count BIGINT COMMENT'成功订单数',  
    gmv DECIMAL(18,2) COMMENT'成交金额',  
    -- 转化率  
    payment_conversion_rate DECIMAL(5,2) COMMENT'支付转化率'  
) COMMENT'电商管理后台日报';  
  
-- 数据加工逻辑  
INSERT OVERWRITE TABLE ads_ec_admin_daily  
SELECT  
    '${bizdate}'AS report_date,  
    -- 新用户数  
    COUNT(CASEWHEN register_date = '${bizdate}'THEN user_id END) AS new_user_count,  
    -- 活跃用户数  
    COUNT(DISTINCT user_id) AS active_user_count,  
    -- 订单指标  
    SUM(order_count) AS order_count,  
    SUM(CASEWHEN order_status = 'success'THEN order_count ELSE0END) AS success_order_count,  
    SUM(order_amount) AS gmv,  
    -- 转化率计算  
    ROUND(SUM(success_order_count)/SUM(order_count)*100, 2) AS payment_conversion_rate  
FROM (  
    -- 从DWS层获取数据  
    SELECT  
        user_id, register_date,  
        order_count, order_amount,  
        order_status  
    FROM dws_user_d   
    WHERE dt = '${bizdate}'  
) t;
```

## 三、关键边界问题处理

### 1. ODS与DWD的边界争议

**常见问题**：

* 数据清洗应该在ODS还是DWD层进行？
* 简单的字段转换属于哪一层的职责？

**处理原则**：

* **ODS层**：仅处理技术层面的格式转换（如字符集转换、时间格式标准化）
* **DWD层**：处理业务语义的清洗（如非法值过滤、枚举值映射）

**错误示例**：

```
-- 错误的ODS层处理（包含业务逻辑）  
SELECT   
    order_id,  
    CASE WHEN user_id < 0 THEN NULL ELSE user_id END, -- 业务逻辑判断  
    order_amount  
FROM source_orders;
```

### 2. DWD与DWS的粒度把控

**常见问题**：

* 哪些维度属性应该退化到DWD层？
* 多细粒度的数据应该放在DWS层？

**最佳实践**：

* **DWD层维度退化**：

+ 高频访问维度（如商品名称、类目名称）
+ 变化缓慢的维度（如用户注册渠道）
+ 数据量小的维度（如省份信息）

* **DWS层汇总粒度**：

+ 按业务分析的最小单位确定（如用户+天）
+ 保留向下钻取的必要维度（如渠道、地区）

### 3. DWS与ADS的应用区分

**常见问题**：

* 计算指标应该放在DWS还是ADS层？
* 跨主题的宽表应该属于哪一层？

**决策标准**：

* **DWS层指标**：

+ 通用性强的基础指标（如订单数、支付金额）
+ 被多个应用共享的指标

* **ADS层指标**：

+ 特定业务场景的复合指标（如转化率、环比增长）
+ 包含复杂业务规则的指标

## 四、分层实施的最佳实践

### 1. 数据流管控原则

* **单向流动**：严格遵循ODS→DWD→DWS→ADS的数据流向，禁止逆向或跨层引用
* **血缘追踪**：每个ADS指标都应能追溯到最细粒度的DWD数据
* **分层检查**：通过数据质量工具验证各层数据的完备性和一致性

### 2. 元数据管理策略

* **技术元数据**：

+ 记录各层表的Schema、分区信息
+ 跟踪数据血缘关系

* **业务元数据**：

+ 明确指标的业务定义
+ 记录口径变更历史

* **示例元数据表**：

  ```
  CREATE TABLE meta_indicator (  
      indicator_id STRING COMMENT'指标ID',  
      indicator_name STRING COMMENT'指标名称',  
      biz_definition STRING COMMENT'业务定义',  
      calculation_formula STRING COMMENT'计算公式',  
      layer_source STRING COMMENT'来源层级',  
      source_table STRING COMMENT'来源表',  
      owner STRING COMMENT'负责人',  
      update_time TIMESTAMP COMMENT'更新时间'  
  );
  ```

### 3. 性能优化建议

* **ODS层**：

+ 采用列式存储格式（如ORC、Parquet）
+ 合理设置分区策略（通常按天分区）

* **DWD层**：

+ 对常用关联键建立索引
+ 考虑热点数据的预排序

* **DWS层**：

+ 使用物化视图加速查询
+ 针对核心宽表进行列裁剪

* **ADS层**：

+ 根据访问模式选择存储引擎（OLAP、KV存储等）
+ 实现结果集缓存

## 五、典型分层误区与规避

### 1. 过度分层问题

**错误表现**：

* 创建不必要的中间层（如DWT层）
* 相同粒度的数据在多层重复出现

**解决方案**：

* 评估每层数据的不可替代性
* 合并功能重复的层级

### 2. 层间耦合问题

**错误表现**：

* ADS层直接引用ODS层数据
* DWS层包含应用特有的业务逻辑

**解决方案**：

* 通过代码扫描工具检测非法引用
* 建立分层规范的评审机制

### 3. 指标口径不一致

**错误表现**：

* 同一指标在不同层计算结果不同
* 业务变更未同步更新所有层级

**解决方案**：

* 建立统一的指标管理平台
* 实现指标定义的集中化管理

## 六、现代架构下的演进趋势

随着数据架构的发展，传统分层模式也在演进：

1. **Data Mesh的影响**：

* 领域导向的数据产品概念
* 分层可能按领域而非技术层级划分

2. **实时数据仓库**：

* ODS层支持流式接入
* DWD层实现实时ETL
* DWS层采用流批一体计算

3. **湖仓一体化**：

* ODS层与数据湖原始区合并
* 分层数据统一存储在对象存储上
* 通过计算引擎实现虚拟分层

## 结语

数据分层的边界划分没有放之四海皆准的标准答案，需要结合以下因素综合考虑：

* 组织的数据成熟度
* 业务需求的复杂度
* 技术栈的特点
* 团队的技术能力

建议的实践路径是：

1. 先建立符合标准理论的基础分层
2. 在运行中持续观察各层的数据流动
3. 针对痛点问题进行渐进式优化
4. 定期评估分层架构的适用性

良好的分层设计应该像优秀的城市规划——各功能区划分清晰，交通动线流畅，同时保留适应未来发展的弹性空间。

---

据统计，99%的大咖都关注了这个公众号👇

大家都在看：

[Hive优化十大法则：让慢查询从2小时降到5分钟的秘籍](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247484554&idx=1&sn=c9810c6d480b8afaa82415a9bf9f480d&scene=21#wechat_redirect)

[数据仓库中的“一致性维度”是什么？为什么它能统一指标口径？(文末送福利)](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247484359&idx=1&sn=36999fea430ac28318a7be077c0e4c1e&scene=21#wechat_redirect)

[数据仓库面试必看：这5个技术问题让无数候选人当场崩溃！](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247484347&idx=1&sn=4a6fc894d87250de61fc9ce8a14e2f00&scene=21#wechat_redirect)

[数据仓库经典面试题附参考答案(建议收藏)](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247484083&idx=1&sn=91ff3ae085b36cf24b67572518034f04&scene=21#wechat_redirect)

[数据仓库监控体系搭建：任务告警/资源调度的自动化方案](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247484272&idx=1&sn=08860e27eeef2ff094379214a4b8f3be&scene=21#wechat_redirect)

[数据仓库架构设计：如何避免常见的陷阱？](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247484085&idx=1&sn=69cccfd861d63c0cac7f52edcb15d537&scene=21#wechat_redirect)

[OLTP vs OLAP：数据仓库中两种核心处理模式的对比分析](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247483866&idx=1&sn=8162da77e55aeb12fdb09a4c5175cda5&scene=21#wechat_redirect)

[实时数仓 vs  离线数仓：2025年企业如何选择？](https://mp.weixin.qq.com/s?__biz=MzE5MTU5OTE0NA==&mid=2247483783&idx=1&sn=90a28f8333a0e012120d575ddfce0047&scene=21#wechat_redirect)

**扫码加入星球🪐 所有资料都可以直接下载**

**⏬**