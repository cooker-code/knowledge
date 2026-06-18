> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_核心知识点/SparkAQE与执行计划诊断|SparkAQE与执行计划诊断]]
---
title: 【极速学习spark调优】Spark SQL 执行计划怎么看？一眼定位慢 SQL
author: 大数据之夏
date: 大数据之夏大数据之夏
url: https://mp.weixin.qq.com/s?__biz=Mzg5NDYyMzE5OA==&mid=2247484406&idx=1&sn=2afbdd89ee33390ae94bf62470f487e6&chksm=c13bad2d78cd9cfd889c1f7548ce87ab39ba76eb781a1b44098027fdf1bbc78b715184d8aa95&mpshare=1&scene=24&srcid=05233HBXNDyNfcCPQgArSSAU&sharer_shareinfo=1e1f1f79b8247b1e83d7791e822b1bdf&sharer_shareinfo_first=1e1f1f79b8247b1e83d7791e822b1bdf#rd
---

> 结论：**会看执行计划的人，调 Spark SQL 永远比别人快一倍**

---

## 一、为什么一定要看执行计划？

你可能遇到过这些问题：

* SQL 很简单，但跑得特别慢
* 明明是小表 Join，却全量 Shuffle
* 改了参数，性能一点没变

👉 **不看执行计划，等于闭着眼睛调 SQL**

---

## 二、explain 能看到什么？

```
explain select * from a join b on a.id= b.id;
```

通常会看到 **三层结构**：

1. Logical Plan（逻辑计划）
2. Optimized Logical Plan（优化后逻辑计划）
3. Physical Plan（物理执行计划）

👉 **真正决定性能的是 Physical Plan**

---

## 三、看执行计划，只盯这 5 个点（重点）

---

## 1、有没有 Exchange（非常重要!!!）

### Exchange 是什么？

> **Exchange = Shuffle**

示例：

```
Exchange hashpartitioning(id#12, 200)
```

### 解读

* 有 Exchange → 一定发生 Shuffle
* Exchange 多 → Shuffle 多 → 慢

👉 **慢 SQL，第一眼数 Exchange 个数**

---

## 2、Join 用的是什么策略？

常见关键字：

### Broadcast Join（最优）

```
BroadcastHashJoin
```

说明：

* 小表被广播
* 几乎没有 Shuffle

---

### Sort Merge Join（最常见，也偏慢）

```
SortMergeJoin
```

说明：

* 两边都 Shuffle
* 还要排序

👉 **看到它，心里就要有性能预期**

---

## 3、有没有被提前过滤（Filter 下推）

### 好的计划

```
Filter (dt = '2026-01-01')
  Scan ...
```

### 坏的计划

```
Scan ...
Join ...
Filter ...
```

👉 **Filter 在 Join 后，基本等于白写**

---

## 4、Shuffle Partition 数量合不合理？

```
hashpartitioning(id#12, 200)
```

* 200 就是 Shuffle Partition 数
* 小数据：太多
* 大数据：可能不够

👉 **这是判断并行度是否合理的关键信息**

---

## 5、是否出现超大的中间算子

常见危险信号：

* 多个 Join 连在一起
* Join 后紧跟 Aggregate
* 无谓的 Project（列太多）

👉 **中间结果越大，越容易慢**

---

## 四、一个慢 SQL 的真实排查过程

### 原始 SQL

```
select *
from orders o
join users u on o.user_id = u.id
where o.dt='2026-01-01'
```

---

### 执行计划关键信息

```
SortMergeJoin
  Exchange
    Scan orders
  Exchange
    Scan users
```

### 问题一眼看出

1. users 是小表，却没 Broadcast
2. where 条件没下推到 Scan 前

---

### 优化后

```
select /* BROADCAST(u) */  *
from orders o
join users u on o.user_id= u.id
where o.dt='2026-01-01'
```

执行计划变为：

```
BroadcastHashJoin
  Scan orders
  BroadcastExchange
    Scan users
```

👉 **Shuffle 直接消失**

---

## 五、explain 的几个常用姿势

### 1、explain（基础）

```
explain select ...
```

---

### 2、explain extended（推荐）

```
explain extended select ...
```

看到更完整的优化信息。

---

### 3、 DataFrame API (经常使用的方式!!!)

```
df.explain(true)
```

---

## 六、90% 慢 SQL 的通用排查路径

1. 看 Physical Plan
2. 数 Exchange
3. 看 Join 策略
4. 看 Filter 是否提前
5. 再调参数 / 改 SQL (有时候就是简单调整SQL, 整个计划会变得很快, 所以对复杂任务经常就是调整思路SQL, 整合SQL的方式方法的调整)

---

## 七、常见误区

### 1、执行计划太复杂，看不懂

错。你只需要盯 **Exchange、Join、Filter**。

---

### 2、有 Broadcast 就一定快

错。广播过大，Executor 会 OOM。

---

### 3、只看 SQL，不看计划

错。SQL 是“你写的”，计划是“Spark 真干的”。

---

## 八、总结

> **执行计划就是 Spark SQL 的“体检报告”**
>
> 看懂它，慢 SQL 基本无处藏身。

---

> 每天花费5分钟学习spark，让你技术之路走得更稳、更快。
>
> 喜欢的点个关注。