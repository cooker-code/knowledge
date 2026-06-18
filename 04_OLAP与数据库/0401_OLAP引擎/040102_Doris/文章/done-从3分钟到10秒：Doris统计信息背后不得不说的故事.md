> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040102_Doris/040102_核心知识点/Doris索引统计信息与查询缓存边界|Doris索引统计信息与查询缓存边界]]
---
title: 从3分钟到10秒：Doris统计信息背后不得不说的故事
author: 一臻数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650485864&idx=1&sn=9009039ff8405c379e6fdf3780a2fdf3&chksm=f25564f772546d52736e41ecd7c5752a105d95644e467f7c6068c2f4ca45bc7311ca55c3b1a3&mpshare=1&scene=24&srcid=0106Py8L9ZcptIKPz1l9fOMh&sharer_shareinfo=2035ac9fbc98796c9acaf86c3389f13c&sharer_shareinfo_first=2035ac9fbc98796c9acaf86c3389f13c#rd
---

更多趣文请关注一臻数据

> ❝
>
> 瞧！又一位被查询性能困扰的数据工程师正对着电脑发呆。屏幕上的SQL执行时间像极了等外卖的倒计时 - 永远看不到头。
>
> 这位同学已经尝试过八百种优化方案，却忘了Doris数据库里有个默默付出的"隐形英雄" - 统计信息。
>
> 统计信息就像是CBO优化器的"千里眼"和"顺风耳"，没有它，优化器就成了"睁眼瞎"。很多工程师都知道CBO能带来性能提升，却不知道统计信息的收集状况直接决定了CBO的"火眼金睛"到底有多准。
>
> 在Apache Doris 2.0版本中，统计信息管理系统经过全面升级，不仅支持自动收集，还能进行健康度评估。这就像给数据库配了个"私人医生"，定期体检，及时预警。
>
> 让我们一起揭开统计信息的神秘面纱，看看这个默默无闻的"幕后英雄"如何让你的查询性能起飞！

## CBO优化的幕后英雄

深夜加班写SQL，查询速度慢得让你抓狂？CBO优化器明明是性能提升的法宝，为何对你的查询无动于衷？别着急，问题很可能出在统计信息这个"隐形助手"身上。

作为Apache Doris 2.0版本引入的重磅特性，CBO（基于代价的优化器）需要准确的统计信息才能发挥威力。好比厨师需要了解食材的特性才能烹饪出美味佳肴，CBO也需要掌握数据的分布特征才能制定最优的执行计划。

统计信息包含了表的行数、列的数据分布、空值数量等关键指标。这些看似简单的数字，却是CBO优化的核心依据。准确的统计信息能帮助优化器在多个可能的执行计划中选择代价最小的那个，显著提升查询性能。

让我们一起走进Doris统计信息的世界，解锁查询优化的关键密码。

### 统计信息全景图

统计信息看似简单，实则暗藏玄机。从收集到使用，从自动到手动，从基础统计到高级特性，构成了一个完整而精密的体系。

收集方式上，Doris提供了手动和自动两种选择。手动收集通过ANALYZE语句触发，可以精确控制收集的范围和时机。自动收集则由系统根据健康度评估自动触发，解放运维人员的双手。

统计内容方面，系统会收集行数、数据分布、空值数量等多维度信息。这些数据构成了优化器的"情报系统"，帮助其更准确地评估查询代价。

在管理层面，Doris提供了完整的作业管理、统计信息查看和删除功能，让用户可以随时掌握统计信息的健康状况。

### 自动收集机制

```
-- 自动收集开关
-- 自动收集功能自 2.0.3 版本起开始支持，且默认全天开启。用户可以通过设置 ENABLE_AUTO_ANALYZE 变量来控制该功能的启用或停用
SET GLOBAL ENABLE_AUTO_ANALYZE = TRUE; // 打开自动收集
SET GLOBAL ENABLE_AUTO_ANALYZE = FALSE; // 关闭自动收集
```

自动收集堪称统计信息管理的"智能管家"。它会定期扫描集群中的表，评估统计信息的健康度。当发现某张表的数据发生显著变化，或者某些列缺失统计信息时，就会自动触发收集任务。

系统默认每5分钟进行一次健康度检查。健康度低于60分的表会被重新收集统计信息。这个阈值可以通过`table_stats_health_threshold`参数调整，让自动收集更符合业务需求。

为了平衡资源消耗，自动收集采用采样方式，默认采样400万行数据。对于大表来说，这个采样量既能保证统计信息的准确性，又不会占用过多系统资源。

### 手动收集的艺术

```
-- 手动收集语法
ANALYZE < TABLE table_name > | < DATABASE db_name > 
    [ (column_name [, ...]) ]
    [ [ WITH SYNC ] [ WITH SAMPLE PERCENT | ROWS ] ];
```

有时我们需要对统计信息进行精细化管理。手动收集好比一把手术刀，让我们能精准控制统计信息的收集范围和时机。

ANALYZE语句是手动收集的指挥棒。你可以选择对整个表收集统计信息：

```
ANALYZE TABLE lineitem;
```

也可以只收集关键列的信息：

```
ANALYZE TABLE lineitem (l_orderkey, l_linenumber);
```

面对超大表时，全量收集可能会耗费较多资源。这时可以使用采样收集：

```
ANALYZE TABLE lineitem WITH SAMPLE PERCENT 10;
```

### 统计信息的健康诊断

```
-- 查看统计作业
SHOW [AUTO] ANALYZE < table_name | job_id >
    [ WHERE STATE = < "PENDING" | "RUNNING" | "FINISHED" | "FAILED" > ];
    
-- 查看统计任务
SHOW ANALYZE TASK STATUS [job_id]

-- 查看统计信息
SHOW COLUMN [cached] STATS table_name [ (column_name [, ...]) ];

-- 查看表信息概况
SHOW TABLE STATS table_name;
```

统计信息也需要定期体检。Doris提供了一系列方式来监测统计信息的健康状况。

查看统计作业状态：

```
SHOW ANALYZE;
```

这里你能看到每个收集任务的执行进度、状态和耗时，就像医院的检查报告一样详细。

查看具体列的统计信息：

```
SHOW COLUMN STATS table_name;
```

输出结果包含了NDV(不同值数量)、空值数量、数据大小等关键指标，帮你全面了解数据特征。

## 性能调优小结

说了这么多理论，来点实际的。我们经常遇到这样的场景：

1. 张工写了个复杂查询，发现执行特别慢。检查发现是JOIN的两张表缺少统计信息，导致优化器选择了错误的JOIN策略。立即对这两张表收集统计信息后，查询时间从原来的3分钟降到了10秒。
2. 李工负责的实时数仓每天有大量数据导入。通过设置合理的健康度阈值和采样比例，实现了统计信息的及时更新，查询性能始终保持稳定。
3. 王工发现自动收集占用资源过多。通过调整收集时间窗口到业务低峰期，既保证了统计信息的及时性，又避免了对业务的影响。

通过大量实践，我们总结出一些统计信息管理的黄金法则：

1. 对大表采用采样收集，采样量建议在400万行以上，既保证准确性又节省资源。
2. 定期检查统计信息的健康度，尤其是在大批量数据导入后。
3. 将自动收集时间窗口设置在业务低峰期，避免资源竞争。
4. 对于关键查询涉及的表，可以设置更严格的健康度阈值，确保统计信息的及时更新。
5. 复杂查询调优时，优先检查统计信息的完整性和准确性。

统计信息好比数据库的指南针，指引着查询优化的方向。掌握了统计信息管理的技巧，让CBO优化器成为你的得力助手，告别查询调优的烦恼。**小小的配置，大大的学问。**

下期，我们将一起探讨其它更有趣有用有价值的内容，敬请期待！

---

一臻数据致力于大数据AI时代的前沿内容分享，会持续分享更多有趣有用有态度的知识。同时也欢迎大家**投稿，共建共进**，帮助圈友们冲破认知壁垒，实现自我提升！

另外，整理了份《**一臻数据知识库**》，其中包含 **Apache Doris** 和**Data+AI****的学习资料、学习课程、白皮书、研究报告、行业标准**和**实践指南** 等内容，会持续更新，欢迎**关注公众号，免费领取**。

**资料获取** 🔗 欢迎扫描下方二维码图片 备注【**Doris**】免费领取❗️

---

往期推荐

[*走进*开源，拥抱开源](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483656&idx=1&sn=300e90f5017ebb3d97d3e98d26d52ff7&chksm=f374e9aec40360b85c87a26d9d1af93b2807ad1c54340676ce3173fb0508b3be7ca9595182f0&scene=21#wechat_redirect)

[*大数据*平台开发规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482778&idx=1&sn=6a4a6b3bf16ab818aa38d222ce46fed6&chksm=f374ea3cc403632a0f6ef1a9728393b459c3d19926f8e9672467f278e9f56abb010b198d2b34&scene=21#wechat_redirect)

[*大数据*仓库开发规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483237&idx=1&sn=824d2125280a009dddeec3f0aa60c4f6&chksm=f374e843c40361551cbbf48c7ad58fb054246a1abc5a60166a29aa7c7a547903848873149902&scene=21#wechat_redirect)

[*大数据*质量管制规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483244&idx=1&sn=1f87c073c411c338ff73f095d250f2e5&chksm=f374e84ac403615c9006d4bcfa49374d5f652b742a7851f9848cb9ea8c0909d53c70dee36f40&scene=21#wechat_redirect)

[*Flink* CDC 1.0至3.0回忆录](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483092&idx=1&sn=396c5873c5b2bf6d66532daeae4b445f&chksm=f374ebf2c40362e4ebac29580dcb7add14c9980290769de9366924d98ad27b3e5dc3f1095613&scene=21#wechat_redirect)

[【Apache *Doris*】Manager 极致丝滑地运维管理](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482971&idx=1&sn=b0953da7f28e016edd032a5be9bd34e5&chksm=f374eb7dc403626b97e57c4e2ec4922e3c17fa71fd2714623cd84328716a53b839976e59f0c3&scene=21#wechat_redirect)

[【Apache *Doris*】如何一键实现MySQL万表整库同步？](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482994&idx=1&sn=1e10161f6f7e9777a8fd572e7bd91482&chksm=f374eb54c4036242d7b83a58bd0533e6187444adae3c6defb0e0dc6ecfe0f314c3965c2d4872&scene=21#wechat_redirect)

[【Apache *Doris*】如何实现高并发点查？（原理+实践全析）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483049&idx=1&sn=e60a95fe683a663f249af230e6606bce&chksm=f374eb0fc403621998ca50c01bad8e093f788a982ca33463d28863840e3bfd3c94791f38efb9&scene=21#wechat_redirect)

[为什么Apache *Doris*适合做大数据的复杂计算，MySQL不适合？](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483064&idx=1&sn=330709a47b85d247af7510b1557fe868&chksm=f374eb1ec40362088903ac541bd4ea3d1d4ea7962b95214405132d4a0a96fae6e23c671787d6&scene=21#wechat_redirect)

[如何正确地使用ChatGPT（角色扮演+提示工程）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482824&idx=1&sn=429e15f252a79bf18f72161c9ccc04af&chksm=f374eaeec40363f8aff13083a2ee0ef60b07ab56ac6c7aaff5ae282a5092aae82d00455c1633&scene=21#wechat_redirect)

[超强满血不收费的AI绘图教程来了（在线Stable Diffusion一键即用）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482860&idx=1&sn=ad75b434b7eae3511017c67d14dad143&chksm=f374eacac40363dc192372fbdd36ff76ad8cbdd866df5a3287c3e48476f4c9c88f2ef3e5f3db&scene=21#wechat_redirect)

点击下方蓝字关注一臻数据