> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkJoin技术全景|FlinkJoin技术全景]]
---
title: 从原理到实战：Flink Regular Join 如何玩转海量数据流？
author: 数栈智算
date:
url: http://mp.weixin.qq.com/s?__biz=Mzk5MDA1NTAyNQ==&mid=2247483705&idx=1&sn=a469cfc66ce1009b2c18f29cb135ef3a&chksm=c482deb4aa848298492dc1bf12d9177841922c875f98b5643c4618422d00c0da2e6bbadc4e4f&mpshare=1&scene=24&srcid=0506zC6SqfhKZugjOuqxPXYk&sharer_shareinfo=576057a7d7aa21412d44731f360a1914&sharer_shareinfo_first=576057a7d7aa21412d44731f360a1914#rd
---

引言：

很多同学在数据处理过程中，对离线Join（Batch Join）觉得So Easy，因为可以依赖成熟的批处理框架（如Hive、Spark SQL），数据全量可见，可全局优化，数据是静态的，可以一次性加载所有数据，轻松完成关联操作。然而，一旦切换到实时Join（Streaming Join），就会遇到各种棘手问题，甚至感觉无从下手，实时Join的难点在于数据是动态的、无序的、无限的，并且需要低延迟处理。

---

1️⃣ 什么是Flink Regular Join

Flink Regular Join（常规连接）是Apache Flink流处理引擎中提供的一种基本连接操作，用于将两个数据流根据指定的条件进行关联，它与Hive SQL一样包含：inner join、left join、right join、full outer join。

2️⃣ Flinksql双流join的底层原理

如上图所示：在flinksql中不管是inner join还是outer join，都需要对左右两边的数据流进行保存，当左流的数据到达，数据会存进L-state，然后会去跟R-state进行关联，然后将关联后的结果输出下游；我们都知道join操作会产生shuffle，那它是怎么保障左右两边流中的数据进行join的时候在同一节点呢？它是通过把on的条件进行分区，然后把相同条件的数据分发到同一个分区里，这就保障了数据在同一节点上处理。

3️⃣ Regular Join 类型详解

在流处理中，Regular Join 支持多种连接方式（以 L 表示左流数据，R 表示右流数据）：  

* Inner Join（内连接）

行为：

仅当左流（L）和右流（R）的数据成功匹配时才会输出结果。

输出：

匹配成功输出+[L, R]

* Left Join（左外连接）

行为：

左流数据到达时立即输出

输出：

匹配成功 → +[L, R] 

匹配失败 → +[L, null]

如果后续右流数据到达，并发现左流之前有未匹配的记录：  

先回撤之前未匹配的记录 → -[L, null] 

再输出正确匹配的结果 → +[L, R]  

* Right Join（右外连接）

行为：

与 Left Join 逻辑相同，但左流和右流的角色互换。

* Full Join（全外连接）

行为：  

任意一条流数据到达时，无论是否匹配，都会立即输出：  

左流数据到达时：  

匹配成功 → +[L, R]  

匹配失败 → +[L, null]  

右流数据到达时：  

匹配成功 → +[L, R]  

匹配失败 → +[null, R]  

当一条流数据到达后，发现另一条流之前有未匹配的记录：  

左流数据到达时：  

回撤右流未匹配的记录 → -[null, R] 

输出正确匹配结果 → +[L, R]  

右流数据到达时：  

回撤左流未匹配的记录 → -[L, null]  

输出正确匹配结果 → +[L, R] 

* 符号说明

+[L, R]：表示新增一条匹配成功的记录  

-[L, null]：表示回撤（删除）之前未匹配的记录  

+[L, null] / +[null, R]：表示暂时输出未匹配的记录（后续可能被修正）

4️⃣ 案例实战：

开发充值订单、提现订单宽表

```
--********************************************************************---- Author:         Jack-- Created Time:   2025-05-05 19:55:32-- Description:    Write your description here-- Hints:          You can use SET statements to modify the configuration--********************************************************************--
SET 'execution.checkpointing.interval' = '10s';
set 'table.exec.sink.upsert-materialize' = 'NONE';
SET 'pipeline.operator-chaining' = 'false';
CREATE TEMPORARY TABLE IF NOT EXISTS `paimon-catalog`.`dwm`.`dwm_recharge_withdraw_info`(    `user_id`                   BIGINT              NOT NULL COMMENT '用户ID'    ,`business_code`       VARCHAR(20)         NOT NULL COMMENT '站点'    ,`k1`                  DATE                NOT NULL COMMENT '分区'    ,`rechargr_amt`      DECIMAL(30, 10)     NULL COMMENT '实际充值到账币种金额'    ,`withdraw_amt`      DECIMAL(30, 10)     NULL COMMENT '实际充值到账币种金额'    ,CONSTRAINT `PK_id` PRIMARY KEY (`user_id`, `k1`, `business_code`) NOT ENFORCED)COMMENT '充值、提现订单事实表'with (    'connector' = 'selectdb'    ,'jdbc-url' = <jdbc-url:9030>    ,'load-url' = <jdbc-url:8030>    ,'cluster-name' = <你集群名称>    ,'table.identifier' = 'dwm.dwm_recharge_withdraw_info'    ,'username' = <账号>    ,'password' = <密码>    ,-- 同步delete事件    'sink.enable-delete' = 'true');

insert into `paimon-catalog`.`dwm`.`dwm_recharge_withdraw_info`with recharge as (select user_id,                         business_code,                         to_date(from_unixtime(deposit_time)) as recharge_date,                         sum(deposit_amount)               as recharge_amt                  from `paimon-catalog`.ods_test.ods_ar_finance_test_recharge_order                  where status = 2                  group by user_id, business_code, to_date(from_unixtime(deposit_time))),     withdraw as (select user_id,                         business_code,                         to_date(from_unixtime(deposit_time)) as withdraw_date,                         sum(`real`)                       as withdraw_amt                  from `paimon-catalog`.ods_test.ods_ar_finance_test_withdraw_order                  where status = 5                  group by user_id, business_code, to_date(from_unixtime(deposit_time)))select coalesce(recharge.user_id, withdraw.user_id)             as user_id,       coalesce(recharge.business_code, withdraw.business_code) as business_code,       coalesce(recharge.recharge_date, withdraw.withdraw_date) as pay_date,       recharge.recharge_amt,       withdraw.withdraw_amtfrom recharge         full outer join withdraw                         on recharge.user_id = withdraw.user_id and recharge.business_code = withdraw.business_code and                            recharge.recharge_date = withdraw.withdraw_date;
```

作业部署详情：

作业状态总览：

###

5️⃣ 注意事项

* **状态管理**：Regular Join 状态持续增长，需通过 `table.exec.state.ttl` 设置合理的状态保留时间，避免内存溢出。

* **数据乱序**：如果数据延迟到达超过 TTL，会导致关联失败（需权衡 TTL 和业务需求）。

**📢 行动号召：**

* 请您详细讲解一下Regular Join的应用场景