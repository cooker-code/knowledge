---
title: 我用Doris SQL Cache拯救了每日早会，太绝了！
author: 一臻数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650487459&idx=1&sn=1d5c67651653b922077c63e7c38a2806&chksm=f2c2533d10e07181ccfc01807c91aa2068029ff9c8c98ccfdfaa6c6acd556e44dbd72db4ff07&mpshare=1&scene=24&srcid=0212EMiQRIfu3WUtEkDMGZP2&sharer_shareinfo=e41defa560a8672545bda4761f2199a4&sharer_shareinfo_first=e41defa560a8672545bda4761f2199a4#rd
---

点击上方 蓝字 关注一臻数据👆 

免费领取 一臻数据知识库 🔗 一起共建共进

> ❝
>
> "小张，这个SQL怎么跑了5分钟还没出结果？" 
>
> "我明明昨天查过这个数据，怎么今天又要等这么久？" 
>
> "早会马上开始了，报表还在加载中..." 
>
> 这些吐槽是否让你感同身受？作为一名DBA或数据工程师，你一定经历过被用户"围攻"的尴尬时刻。面对重复的查询需求，系统却像一位"健忘症患者"，每次都要重新计算，这着实让人抓狂。 
>
> 不过，今天我要介绍一位特别的"记忆大师" —— Doris SQL Cache，能够智能记住查询结果，让重复查询变得行云流水。从此，告别反复计算的烦恼，让你的系统性能起飞！

## Doris的高效缓存引擎

每天早上9点，运营同事都会查询昨天的销售数据，每个人都在跑相同的SQL，系统压力山大。看着日益增长的并发请求，DBA表示很头疼...

在数据分析领域，**重复查询**是一个普遍现象。小王每天查看销售报表，小李每天统计用户增长，这些查询逻辑往往大同小异。如果每次查询都要重新计算，岂不是太浪费资源了？

Doris的SQL Cache好比一台懂事的咖啡机。第一杯咖啡需要现磨现煮，之后的咖啡就能直接享用了。它通过智能缓存查询结果，大幅提升查询性能。

当查询请求到达时，Doris会进行一系列精确的匹配：**SQL文本是否相同？表的版本是否变化？权限是否一致？** 就像咖啡师在确认你的口味和要求。只有所有条件都匹配，才能享受到缓存带来的快速响应。

我们来看一个销售数据分析的场景：

```
-- 设置当前会话开启SQL Cache  
set enable_sql_cache=true;  
  
-- 分析昨日销售数据  
SELECT   
    province,  
    SUM(sales_amount) as total_sales,  
    COUNT(DISTINCT user_id) as buyer_count  
FROM sales_detail   
WHERE dt = '2025-02-09'  
GROUP BY province;
```

第一次执行这个查询时，Doris会从BE节点获取数据并计算。之后相同的查询就能直接从缓存获取结果，响应时间**从秒级降至毫秒级**。这对于每天早上的销售分析报表简直就是一剂强心针。

## Doris让缓存可管可控

光有缓存机制还不够，Doris还提供了丰富的SQL Cache监控指标，方便用户进行有效监控。

通过FE的HTTP接口，我们能看到缓存的命中次数：

```
# FE 的 HTTP 接口 http://${FE_IP}:${FE_HTTP_PORT}/metrics  
  
# 代表已经把 1 个 SQL 写入到缓存中  
doris_fe_cache_added{type="sql"} 1  
  
# 代表命中了两次 SQL Cache  
doris_fe_cache_hit{type="sql"} 2
```

通过BE的指标，我们能掌握缓存占用的内存大小：

```
#  BE 的 HTTP 接口 http://${BE_IP}:${BE_HTTP_PORT}/metrics  
  
# 代表当前 BE 的内存中存在 1205 个 Cache  
doris_be_query_cache_sql_total_count 1205  
  
# 当前所有 Cache 占用 BE 内存 44k  
doris_be_query_cache_memory_total_byte 44101
```

这些数据好比是给缓存装上了个"**体检仪**"，让我们随时掌握它的健康状况。

为了避免缓存过度占用内存，Doris还提供了灵活的内存控制机制：

**1. FE 内存控制**

```
-- 最多存放 100 个 Cache 元数据，超过时自动释放最近最久未使用的元数据。默认值为 100。    
ADMIN SET FRONTEND CONFIG ('sql_cache_manage_num'='100');    
    
-- 当 300 秒未访问该 Cache 元数据后，自动进行释放。默认值为 300。    
ADMIN SET FRONTEND CONFIG ('expire_sql_cache_in_fe_second'='300');  
  
-- 默认超过 3000 行结果时，不创建 SQL Cache。    
ADMIN SET FRONTEND CONFIG ('cache_result_max_row_count'='3000');    
    
-- 默认超过 30M 时，不创建 SQL Cache。    
ADMIN SET FRONTEND CONFIG ('cache_result_max_data_size'='31457280');
```

**2. BE 内存控制**

```
-- 当 Cache 的内存空间超过 query_cache_max_size_mb + query_cache_elasticity_size_mb 时，    
-- 释放最近最久未使用的 Cache，直至占用内存低于 query_cache_max_size_mb。    
query_cache_max_size_mb = 256    
query_cache_elasticity_size_mb = 128
```

这不就是给咖啡机设置了储水量上限，**既保证了性能，又避免了资源浪费**。

## 实践案例，化腐朽为神奇

回到开头。小张是某电商平台的DBA，遇到了一个棘手的问题：每天早上9点，系统CPU使用率飙升到90%，响应时间从毫秒级飙升到秒级。经过排查发现，这个时间点有大量运营人员在查询昨日销售数据。

让我们看看小张如何借助Doris SQL Cache发挥最大效力：

**时间窗口优化**

销售数据分析中，我们经常会用到now()函数获取当前时间。每一秒钟，这个函数的返回值都在变化，导致缓存频繁失效。聪明的做法是把**时间粒度调整得更粗一些**：

```
-- 优化前：缓存效果差  
SELECT * FROM sales WHERE create_time > now();  
  
-- 优化后：一天内都能复用缓存  
SELECT * FROM sales WHERE dt = DATE(now());
```

**查询模式优化**

在报表分析场景中，我们常常需要统计各种维度的数据。与其**每个维度单独查询，不如把相关维度的统计合并到一个查询中**：

```
-- 优化前：多次查询，缓存效果差  
SELECT COUNT(*) FROM orders WHERE dt='2025-02-08';  
SELECT SUM(amount) FROM orders WHERE dt='2025-02-08';  
  
-- 优化后：一次查询，充分利用缓存  
SELECT   
    COUNT(*) as order_count,  
    SUM(amount) as total_amount   
FROM orders   
WHERE dt='2025-02-08';
```

...

通过开启SQL Cache并优化查询模式，问题得到了完美解决，化腐朽为神奇：

* CPU使用率降到了50%以下
* 查询响应时间从2秒降到了50毫秒
* 运营团队再也不用担心系统卡顿了

小张的经验告诉我们：**SQL Cache不仅能提升查询性能，还能大幅降低系统资源消耗**。

下期，我们将一起探讨其它更有趣有用有价值的内容，敬请期待！

---

一臻数据致力于大数据AI时代的前沿内容分享，会持续分享更多有趣有用有态度的知识。同时也欢迎大家**投稿，共建共进**，帮助圈友们冲破认知壁垒，实现自我提升！

另外，整理了份《**一臻数据知识库**》，其中包含 **Apache Doris**和**Data+AI****的学习资料、学习课程、白皮书、研究报告、行业标准**和**实践指南** 等内容，会持续更新，欢迎**关注公众号，免费领取**。

**资料获取** 🔗 欢迎扫描下方二维码图片 备注【**Doris**】免费领取❗️

---

往期推荐

[*走进*开源，拥抱开源](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483656&idx=1&sn=300e90f5017ebb3d97d3e98d26d52ff7&chksm=f374e9aec40360b85c87a26d9d1af93b2807ad1c54340676ce3173fb0508b3be7ca9595182f0&scene=21#wechat_redirect)

[*大数据*平台开发规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482778&idx=1&sn=6a4a6b3bf16ab818aa38d222ce46fed6&chksm=f374ea3cc403632a0f6ef1a9728393b459c3d19926f8e9672467f278e9f56abb010b198d2b34&scene=21#wechat_redirect)

[*大数据*仓库开发规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483237&idx=1&sn=824d2125280a009dddeec3f0aa60c4f6&chksm=f374e843c40361551cbbf48c7ad58fb054246a1abc5a60166a29aa7c7a547903848873149902&scene=21#wechat_redirect)

[*大数据*质量管制规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483244&idx=1&sn=1f87c073c411c338ff73f095d250f2e5&chksm=f374e84ac403615c9006d4bcfa49374d5f652b742a7851f9848cb9ea8c0909d53c70dee36f40&scene=21#wechat_redirect)

[*Flink* CDC 1.0至3.0回忆录](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483092&idx=1&sn=396c5873c5b2bf6d66532daeae4b445f&chksm=f374ebf2c40362e4ebac29580dcb7add14c9980290769de9366924d98ad27b3e5dc3f1095613&scene=21#wechat_redirect)

[【Apache Doris】*Manager* 极致丝滑地运维管理](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482971&idx=1&sn=b0953da7f28e016edd032a5be9bd34e5&chksm=f374eb7dc403626b97e57c4e2ec4922e3c17fa71fd2714623cd84328716a53b839976e59f0c3&scene=21#wechat_redirect)

[【Apache Doris】如何实现*高并发*点查？（原理+实践全析）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483049&idx=1&sn=e60a95fe683a663f249af230e6606bce&chksm=f374eb0fc403621998ca50c01bad8e093f788a982ca33463d28863840e3bfd3c94791f38efb9&scene=21#wechat_redirect)

[*为什么*Apache Doris适合做大数据的复杂计算，MySQL不适合？](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483064&idx=1&sn=330709a47b85d247af7510b1557fe868&chksm=f374eb1ec40362088903ac541bd4ea3d1d4ea7962b95214405132d4a0a96fae6e23c671787d6&scene=21#wechat_redirect)

[*深夜*无需加班，Apache Doris让数据自己会跑](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650485454&idx=1&sn=a73a4a6e78b610cdc47d5e31293d871a&chksm=f374f0a8c40379be78b40cab31721c31f485af5272299ef7f8dbfdd30cff7dc6fcc27b49eb4e&scene=21#wechat_redirect)

[别让你的CPU打盹儿：Apache Doris*并行*执行原理大揭秘！](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650486255&idx=1&sn=8e8c4f95293ed2451c2085efbb72fc72&chksm=f374f789c4037e9f5a981d03ce8bdb90a9cf5f26e3f268d177708bcbea62b567774a1536627b&scene=21#wechat_redirect)

[如何正确地使用Chat*GPT*（角色扮演+提示工程）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482824&idx=1&sn=429e15f252a79bf18f72161c9ccc04af&chksm=f374eaeec40363f8aff13083a2ee0ef60b07ab56ac6c7aaff5ae282a5092aae82d00455c1633&scene=21#wechat_redirect)

[*超强*满血不收费的AI绘图教程来了（在线Stable Diffusion一键即用）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482860&idx=1&sn=ad75b434b7eae3511017c67d14dad143&chksm=f374eacac40363dc192372fbdd36ff76ad8cbdd866df5a3287c3e48476f4c9c88f2ef3e5f3db&scene=21#wechat_redirect)

点击下方蓝字关注一臻数据