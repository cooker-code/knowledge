---
title: 阿里面试：Paimon Changelog 和 合并引擎有哪些组合，分别适用于哪些场景？
author: 大数据技能圈
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491598&idx=1&sn=48f2d1bc3696d67d6262859dcad984d1&chksm=c12f56c4e36e14737120a119ebde1baf7976773c5cbb8dd24d47e5d7f7a857ad63bfaec82223&mpshare=1&scene=24&srcid=072295nmYYAlTlsIeyDQ6VuE&sharer_shareinfo=ce8290dfb94ed9d41e82979ce4137487&sharer_shareinfo_first=ce8290dfb94ed9d41e82979ce4137487#rd
---

01

**加入知识星球《随川陪你学大数据》将获得以下权益**

扫描二维码加入星球，随加入人数增加，逐步涨价，现在就是最优惠的

01

**Paimon changelog和合并引擎如何选择？**

Paimon 把「怎么产生 changelog」和「怎么合并多条变更」拆成了两个正交维度，只要理解下面两张“坐标轴”，就能拼出所有计算场景：

1. changelog-producer 轴（行级语义）  
   • none：不写 changelog，只能离线查询。  
   • input：原样透传上游的 +I/-U/+U/-D 消息，适合链路里已经有 Flink 状态、只想把 Paimon 当落地表的“无状态”场景。  
   • lookup：每次 Flush 时 Lookup 当前行值，生成完整 -U / +U 对；适合需要回撤语义的流读。  
   • full-compaction：只在 Compaction 后生成 changelog，吞吐高、代价小，但延迟取决于 compaction-interval。
2. merge-engine 轴（列级语义）  
   • deduplicate（默认）：主键去重，保留最新整行。  
   • partial-update：同一主键多次更新不同列，空值不覆盖。  
   • aggregation：按主键对列做预聚合（sum/max/last\_non\_null…）。  
   • first-row：保留主键第一行，不接受回撤。

下面用 4 组最常用“坐标点”给出配置范例，每条都能直接复制运行。

---

场景 1：离线宽表 T+1 快照  
changelog-producer = none + merge-engine = deduplicate  
不需要流读，只要每天产出一份最新镜像。

sql

复制

```
CREATETABLE ods_order (  
  order_id BIGINT,  
  user_id STRING,  
  amount DECIMAL(18,2),  
PRIMARYKEY(order_id)NOT ENFORCED  
)WITH(  
'changelog-producer'='none',  
'merge-engine'='deduplicate'  
);
```

---

场景 2：流式 ETL 去重落地  
changelog-producer = input + merge-engine = deduplicate  
上游 Kafka 已带 -U/+U，无须再次 Lookup，链路无状态，纯 Append 写。

sql

复制

```
CREATETABLE dwd_order_unique (  
  order_id BIGINT,  
  user_id STRING,  
  amount DECIMAL(18,2),  
PRIMARYKEY(order_id)NOT ENFORCED  
)WITH(  
'changelog-producer'='input',  
'merge-engine'='deduplicate'  
);
```

---

场景 3：多流 Join 的 Partial Update（实时宽表）  
changelog-producer = full-compaction + merge-engine = partial-update  
不同作业分别更新不同字段，compaction 后产出完整 -U/+U 给下游。

sql

复制

```
CREATETABLE dw_order_detail (  
  order_id STRING,  
  product_type STRING,  
  plat_name STRING,  
  logistics_id BIGINT,  
  dispatch_time TIMESTAMP(3),  
  finish_time TIMESTAMP(3),  
  order_status INT,  
  update_time TIMESTAMP(3),  
PRIMARYKEY(order_id)NOT ENFORCED  
)WITH(  
'bucket'='20',  
'sequence.field'='update_time',  
'changelog-producer'='full-compaction',  
'changelog-producer.compaction-interval'='2 min',  
'merge-engine'='partial-update',  
'partial-update.ignore-delete'='true'  
);
```

---

场景 4：实时聚合指标表  
changelog-producer = lookup + merge-engine = aggregation  
需要秒级回撤，故用 lookup；price 列取 max，sales 列做 sum。

sql

复制

```
CREATETABLE ads_product_amt (  
  product_id BIGINT,  
  price      DOUBLE,  
  sales      BIGINT,  
PRIMARYKEY(product_id)NOT ENFORCED  
)WITH(  
'changelog-producer'='lookup',  
'merge-engine'='aggregation',  
'fields.price.aggregate-function'='max',  
'fields.sales.aggregate-function'='sum',  
'fields.price.ignore-retract'='true',  
'fields.sales.ignore-retract'='false'-- 允许回撤  
);
```

---

组合速查表（文字版）

表格

复制

| 场景描述 | changelog-producer | merge-engine | 下游能否流读 | 备注 |
| --- | --- | --- | --- | --- |
| 离线快照 | none | deduplicate | 否 | 无 changelog，仅批读 |
| 流式去重落地 | input | deduplicate | 是 | 上游已有完整 changelog |
| 实时宽表（多流拼字段） | full-compaction | partial-update | 是 | compaction 后产出 -U/+U，延迟分钟级 |
| 实时聚合回撤 | lookup | aggregation | 是 | 逐行 lookup，延迟秒级，支持回撤 |
| 只保留首行（埋点日志） | lookup | first-row | 是 | 仅 insert 流，无回撤 |

记住口诀：  
“只落盘选 none，上游有变更用 input；  
要回撤用 lookup，能等 compaction 就用 full-compaction；  
字段补全用 partial-update，指标汇总用 aggregation，只留最早一条用 first-row。”

更多文档已放入知识星球《随川陪你学大数据》，可扫码获取

获取更多信息，关注大数据技能圈

**欢迎添加作者交流**

## 推荐阅读系列文章

* [腾讯面试：Spark内存如何优化？包含哪几个方面？](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491167&idx=1&sn=d62e305f6803f484aa40469393b1fdb8&scene=21#wechat_redirect)
* [Clickhouse经典面试题200道](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491591&idx=1&sn=455c326ed0b34b578d044bd82efa1632&scene=21#wechat_redirect)