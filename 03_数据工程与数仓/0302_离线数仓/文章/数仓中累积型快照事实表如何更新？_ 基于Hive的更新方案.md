---
title: 数仓中累积型快照事实表如何更新？| 基于Hive的更新方案
author: 会飞的一十六
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247501009&idx=1&sn=5d9bd5f5bd0031a583e8ce7385e3af1f&chksm=e9a3fc7fe544d8534fc88218ca87984b33df5da159a34b91d7663b50fa12fcab6587038224ad&mpshare=1&scene=24&srcid=0307CGFnDV3mwSEcjZ7GiMOY&sharer_shareinfo=cfea6aca58f5ddd07db96bad0a16209c&sharer_shareinfo_first=cfea6aca58f5ddd07db96bad0a16209c#rd
---

### 一、事实表类型对比

| 类型 | 特点 | 典型场景 | 更新模式 |
| --- | --- | --- | --- |
| 事务型事实表 | 仅追加，不修改历史记录 | 订单支付、退款 | Insert Only |
| 周期型快照事实表 | 定期覆盖历史状态快照 | 每日商品库存 | Overwrite |
| **累积型快照事实表** | 跟踪业务全生命周期状态 | 订单状态流转 | 增量更新 |

---

### 二、订单状态表设计原理

  

#### 数据流转路径

---

### 三、核心实现步骤

#### 3.1 数据准备

```
-- 创建数据库：e-commerce    
DROP DATABASE IF EXISTS ec CASCADE;    
CREATE DATABASE ec LOCATION '/ec';    
USE ec;    
  
-- 原始层订单状态表    
DROP TABLE IF EXISTS ec.ods_order_status;    
CREATE TABLE ec.ods_order_status (    
    order_id STRING,    
    order_status STRING,    
    operation_time STRING    
) PARTITIONED BY (ymd STRING)    
LOCATION '/ec/ods_order_status';    
  
-- 明细层累积型快照事实表    
DROP TABLE IF EXISTS ec.dwd_order;    
CREATE TABLE ec.dwd_order (    
    order_id STRING,    
    start_time STRING,    
    end_time STRING    
) PARTITIONED BY (ymd STRING)    
LOCATION '/ec/dwd_order';    
  
-- 生成测试数据并写入原始层    
INSERT INTO TABLE ec.ods_order_status PARTITION(ymd='2020-01-01') VALUES    
("P1","start","2020-01-01 08:00:00"),    
("P1","end","2020-01-01 08:01:00"),    
("P2","start","2020-01-01 22:45:00"),    
("P3","start","2020-01-01 23:30:00");    
  
INSERT INTO TABLE ec.ods_order_status PARTITION(ymd='2020-01-02') VALUES    
("P3","end","2020-01-02 00:15:00"),    
("P4","start","2020-01-02 06:30:00");  
```

---

#### **解决方案**：

通过 **全量合并（Full Merge）** + **动态分区** 实现，核心流程：

#### 3.2 动态分区配置

```
-- 启用动态分区功能    
SET hive.exec.dynamic.partition=true;    
-- 设置非严格模式（允许全动态分区）    
SET hive.exec.dynamic.partition.mode=nonstrict;  
```

---

#### 3.3 首日数据写入

##### 数据转换逻辑

```
WITH t1 AS (    
    SELECT    
        order_id,    
        STR_TO_MAP(CONCAT_WS(',', COLLECT_SET(CONCAT(order_status, '=', operation_time))), ',', '=') AS m    
    FROM ec.ods_order_status    
    WHERE ymd='2020-01-01'    
    GROUP BY order_id    
)    
SELECT    
    order_id,    
    m['start'] AS start_time,    
    m['end'] AS end_time,    
    CASE    
        WHEN m['end'] IS NOT NULL THEN '2020-01-01'    
        ELSE '9999-12-31'    
    END AS ymd    
FROM t1;  
```

  

##### 数据写入操作

```
WITH t1 AS (    
    -- 同上数据转换逻辑    
)    
INSERT OVERWRITE TABLE ec.dwd_order PARTITION(ymd)    
SELECT    
    order_id,    
    m['start'],    
    m['end'],    
    CASE    
        WHEN m['end'] IS NOT NULL THEN '2020-01-01'    
        ELSE '9999-12-31'    
    END AS ymd    
FROM t1;  
```

  

---

#### 3.4 次日增量更新

```
WITH    
    t1 AS (    
        -- 次日新增数据处理逻辑（同首日）    
    ),    
    new AS (    
        SELECT order_id, start_time, end_time,    
            CASE WHEN m['end'] IS NOT NULL THEN '2020-01-02' ELSE '9999-12-31' END AS ymd    
        FROM t1    
    ),    
    old AS (SELECT * FROM ec.dwd_order WHERE ymd='9999-12-31')    
INSERT OVERWRITE TABLE ec.dwd_order PARTITION(ymd)    
SELECT    
    COALESCE(new.order_id, old.order_id),    
    COALESCE(new.start_time, old.start_time),    
    COALESCE(new.end_time, old.end_time),    
    COALESCE(new.ymd, old.ymd)    
FROM new    
FULL OUTER JOIN old ON new.order_id = old.order_id;  
```

  

---

### 四、拓展：完整业务方案

完整订单状态流转逻辑与扩展字段示例：  

```
-- 扩展字段查询示例    
WITH t1 AS (    
    -- 同上逻辑，扩展状态字段    
)    
SELECT    
    order_id,    
    m['已支付'] AS pay_time,    
    m['已取消'] AS cancel_time,    
    m['确认收货'] AS confirm_time,    
    m['退款中'] AS refund_time,    
    m['退款完成'] AS refund_finish_time,    
    m['支付过期'] AS overdue_time,    
    m['结束'] AS end_time,    
    CASE    
        WHEN m['结束'] IS NOT NULL THEN '昨日'    
        ELSE '9999-12-31'    
    END AS ymd    
FROM t1;  
```

---

### 五、性能优化方案

#### 5.1 分区生命周期管理

#### 5.2 存储压缩配置

```
-- 启用输出压缩    
SET hive.exec.compress.output=true;    
-- 指定Snappy压缩算法    
SET mapred.output.compression.codec=org.apache.hadoop.io.compress.SnappyCodec;  
```

---

### 六、异常处理机制

#### 6.1 数据逻辑校验

```
-- 时间顺序合理性检查    
SELECT    
    order_id,    
    CASE    
        WHEN pay_time < create_time THEN '支付时间早于创建时间'    
        WHEN deliver_time < pay_time THEN '发货时间早于支付时间'    
        ELSE '数据正常'    
    END AS time_check    
FROM dwd_order    
WHERE ymd = '9999-12-31';  
```

#### 6.2 数据回溯流程

---

（全文完）

往期精彩

[3分钟学会SQL中的流式状态分析技术，轻松搞定离散事件流处理问题](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247500988&idx=1&sn=83eb4d03ae9197df606af82aeb839b40&scene=21#wechat_redirect)

[如何通过数仓模型高效计算用户流失与回流指标 ？| 周期快照模型实战](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247500916&idx=1&sn=18e4a5bbf6cbcecda6191ad9b8c2a08c&scene=21#wechat_redirect)

[Hive 动态分区小文件过多问题优化](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247500909&idx=1&sn=6f283be8676b0e0f45e2337a4619d5dd&scene=21#wechat_redirect)

[从零构建企业级财务分析数仓 | Hive建模实战](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247500843&idx=1&sn=6556304e654a145805e4eff561421791&scene=21#wechat_redirect)

[数仓建模：基于OTD流程的订单履约分析](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247490597&idx=1&sn=5e109436a85b3fdb728d3381711c9a94&scene=21#wechat_redirect)

[Hive动态时间窗口如何实现？](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247490511&idx=1&sn=d6cd0c3a99344de9037882d030e12867&scene=21#wechat_redirect)