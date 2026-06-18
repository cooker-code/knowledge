> 已吸收至：[[03_数据工程与数仓/0305_湖仓表格式/030505_Paimon/030505_核心知识点/Paimon主键表合并引擎与ChangelogProducer|Paimon主键表合并引擎与ChangelogProducer]]
---
title: Paimon 合并引擎到底怎么选？一文讲透 Deduplicate、Partial Update、Aggregation
author: 三石大数据
date:
url: https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247487343&idx=1&sn=4832bc28762425c6d2b1311f4ff178d5&chksm=fb8465498b1522ad08a472ddc7d812fe1f2d791eadd250fb9ffe04cc5eca5d2514ca768e08a1&mpshare=1&scene=24&srcid=0327GAkCAwmViLi6VDAQtXSq&sharer_shareinfo=f26b5320fdccbf7975adc1b5ae74635b&sharer_shareinfo_first=f26b5320fdccbf7975adc1b5ae74635b#rd
---

# 推荐阅读文章列表

[2025最新大数据开发面试笔记V7.0——试读](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247486661&idx=1&sn=dcf91fcdb56ffef0835a5f5cae0780a6&scene=21#wechat_redirect)

[没有实习经历，还有机会进大厂吗](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247486517&idx=1&sn=d4be6b7204151101c502d071aa48bac5&scene=21#wechat_redirect)

[简历指导套餐4.0——对标大厂的PB级数仓项目](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247486265&idx=1&sn=7b41a46f09a79ae1796c821ea628ad5a&scene=21#wechat_redirect)

[企业级项目一：金融信贷离线数仓建设](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247487216&idx=1&sn=e89b5756f3d218f0d6bc5ed571b3a614&scene=21#wechat_redirect)

[企业级项目二：基于Doris+LangChain构建数据智能运营AI助手](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247487169&idx=1&sn=1b59381a556784ee30fbd734270161ab&scene=21#wechat_redirect)

# 前言

很多人刚接触 Paimon，最容易困惑的一个点就是：为什么同样是主键表，还要分不同的合并引擎？

有的表适合保留最新状态，有的表适合按字段补齐，还有的表适合直接做聚合沉淀；如果这个地方没理解透，后面建表、写实时链路、做宽表和指标层时，就很容易选错。

这篇文章，我就用最常见的三种引擎：

* Deduplicate
* Partial Update
* Aggregation

结合实际案例，帮你把这件事讲清楚。

# 什么是 Paimon 合并引擎？

先说结论：合并引擎，本质上就是同一个主键来了多条数据后，Paimon 应该怎么处理？

比如一条订单数据，可能会经历： 下单 -> 支付 ->发货 -> 完成

同一个 order\_id 会反复更新。

那问题来了：这些数据写进 Paimon 后，到底是保留最后一条，还是按字段补齐，还是按某种聚合方式累加？

这套规则就是 Merge Engine（合并引擎）。你可以把它理解成：Paimon 主键表的数据合并规则

它不是一个可有可无的小配置，而是决定这张表最终长什么样的核心能力。

所以在实际项目里，选合并引擎之前，先问自己一个问题：这张表到底是状态表、宽表，还是指标表？

因为不同语义，对应的合并方式完全不同。

# 三种常见合并引擎，到底适合什么场景？

## Deduplicate：保留最新一条

它适合的场景是：同一个主键来了多条记录，我只关心最终状态。

比如订单状态表：

* 1001，待支付
* 1001，已支付
* 1001，已发货
* 1001，已完成

最后查询时，你通常只想看到：

* 1001，已完成

这就是 Deduplicate 的典型用法。 它最适合：

* 订单当前状态表
* 用户最新信息表
* 商品最新信息表

一句话总结：只关心最新值，就选 Deduplicate。

## Partial Update：按字段补齐

这个是 Paimon 里非常实用的一种能力。

很多业务表不是一次性写完整的，而是不同字段从不同链路同步过来。

比如用户画像宽表：

* 一条链路写基础信息：姓名、城市、性别
* 一条链路写会员信息：会员等级、到期时间
* 一条链路写行为信息：点击数、下单数

这些更新不是同时到达的。

如果你使用Deduplicate引擎，后来的数据就会把之前所有字段覆盖掉。而选择Partial Update 的意思就是：本次来了哪些字段，就更新哪些字段；没来的字段保留原值。

它很适合：

* 用户画像宽表
* 商品画像宽表
* 风控标签宽表

一句话总结：同一个 key，不同字段分批补齐，就选 Partial Update。

## Aggregation：按聚合规则合并

它不是覆盖，也不是按列补，而是：同一个主键来了多条数据，直接按照聚合函数合并

比如商品销售汇总表：

* 销量做 sum
* 销售额做 sum
* 最高价格做 max

那么同一个商品一天内来了多笔订单后，Paimon 可以直接把结果聚合起来。

它特别适合：

* 商品销量汇总
* 用户行为计数
* 广告曝光点击统计

一句话总结：同一个 key 需要累加、取最大、取最小，就选 Aggregation。

# 结合实际案例，看三种引擎怎么用

## 案例一：订单状态表，用 Deduplicate

比如我们要做一张订单最新状态表：

```
CREATE TABLE dwd_order_status (
    order_id BIGINT,
    user_id BIGINT,
    order_status STRING,
    pay_amount DECIMAL(16,2),
    update_time TIMESTAMP(3),
    PRIMARY KEY (order_id) NOT ENFORCED
) WITH (
    'merge-engine' = 'deduplicate'
);
```

订单 1001 依次写入：

```
1001, 20001, CREATED,  99.00, 2026-03-20 10:00:00
1001, 20001, PAID,     99.00, 2026-03-20 10:05:00
1001, 20001, FINISHED, 99.00, 2026-03-20 10:30:00
```

最终查出来，通常只关心最后状态：

```
1001, 20001, FINISHED, 99.00, 2026-03-20 10:30:00
```

这就是最典型的「状态覆盖型」表。

## 案例二：用户画像宽表，用 Partial Update

比如做一张用户画像表：

```
CREATE TABLE dws_user_profile (
    user_id BIGINT,
    user_name STRING,
    gender STRING,
    city STRING,
    member_level STRING,
    member_expire_time TIMESTAMP(3),
    click_cnt_7d BIGINT,
    order_cnt_30d BIGINT,
    PRIMARY KEY (user_id) NOT ENFORCED
) WITH (
    'merge-engine' = 'partial-update'
);
```

先来基础信息：

```
10001, Alice, Female, Shanghai, null, null, null, null
```

再来会员信息：

```
10001, null, null, null, VIP, 2026-12-31 23:59:59, null, null
```

再来行为信息：

```
10001, null, null, null, null, null, 25, 3
```

最终结果会被补齐成：

```
10001, Alice, Female, Shanghai, VIP, 2026-12-31 23:59:59, 25, 3
```

这类场景在企业里非常常见，尤其适合多流宽表建设。

## 案例三：商品销量汇总表，用 Aggregation

如果我们要沉淀商品日销售指标表：

```
CREATE TABLE dws_product_sales_day (
    stat_date STRING,
    product_id BIGINT,
    sale_cnt BIGINT,
    sale_amount DECIMAL(16,2),
    max_price DECIMAL(16,2),
    PRIMARY KEY (stat_date, product_id) NOT ENFORCED
) WITH (
    'merge-engine' = 'aggregation',
    'fields.sale_cnt.aggregate-function' = 'sum',
    'fields.sale_amount.aggregate-function' = 'sum',
    'fields.max_price.aggregate-function' = 'max'
);
```

假设当天来了三笔数据：

```
20260324, 9001, 1, 199.00, 199.00
20260324, 9001, 2, 398.00, 199.00
20260324, 9001, 1, 299.00, 299.00
```

最终结果会变成：

```
20260324, 9001, 4, 896.00, 299.00
```

这类表本质上就是「指标沉淀表」，特别适合 DWS 层。

# 总结

Paimon 合并引擎，表面看是存储参数，实际上背后是建模思路

1、最新状态表，用 Deduplicate，适合状态型数据

2、多源宽表，用 Partial Update，适合字段分批补齐的表

3、指标结果表，用 Aggregation，适合指标累加沉淀

# 写在最后

### V7.0笔记获取方式

> 后台回复：大数据面试笔记