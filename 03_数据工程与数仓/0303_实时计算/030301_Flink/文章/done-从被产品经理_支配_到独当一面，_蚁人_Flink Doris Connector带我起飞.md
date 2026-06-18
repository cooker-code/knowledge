> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkDorisConnector写入机制|FlinkDorisConnector写入机制]]
---
title: 从被产品经理"支配"到独当一面，"蚁人"Flink Doris Connector带我起飞
author: 一臻数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650486656&idx=1&sn=211e3c76c141ffb0b6bfb2f291a088c8&chksm=f25f55a6d81635088a91291486e766b8be9c9d60fb2af5d975c044f75e23928f34c8dda19344&mpshare=1&scene=24&srcid=0109xyRT2305M3mHkEWcvaHd&sharer_shareinfo=ee7e231c977048b433acc97403b5f80a&sharer_shareinfo_first=ee7e231c977048b433acc97403b5f80a#rd
---

更多趣文请关注一臻数据

> ❝
>
> 不知道你是否遇到过这样的场景： 
>
> 产品经理急匆匆跑来说"Doris数据怎么还没实时同步？" 
>
> 老板突然要求"把所有数据实时展示！" 
>
> 半夜被数据延迟告警吵醒... 
>
> 别担心，一起来跟随新晋数据工程师小张的起飞经历，领略Flink Doris Connector的神奇魅力！

## "蚁人"英雄 - Flink Doris Connector

小张是一名刚入职的数据工程师。

在一个繁忙的周一早晨，他收到了一个紧急任务：业务部门需要实时查看各个销售渠道的订单数据。这个场景让他想起了漫威电影中的蚁人，能在微观和宏观世界自如穿梭。

而在Doris生态中，恰巧有一位类似的英雄 - Flink Doris Connector，它好比数据世界的"蚁人"，能够在数据的源头和目的地之间自由穿行。

让我们通过一张简易的战斗装备图来了解这位数据英雄：

"数据，启动！"小张轻声念道。

在这个数字世界中，Flink Doris Connector好比一位全能型超级英雄，装备着各种超能力装备：

1. 形态进化器：支持表结构动态变更
2. 数据侦测器：兼容识别多种数据源的信号
3. 高速传输装置：配备实时流处理和批处理双引擎
4. 数据防护盾：搭载Exactly Once保护机制 ...

小张打开了版本选择器，发现这位"蚁人"有多个形态可供选择：

每个版本都像是这位"蚁人"的不同等级形态，能够适应不同阶级的战场需求。

在数据实时同步的战场上，Flink Doris Connector是一位身手敏捷的特工。

它能够悄无声息地潜入数据源，精准捕获每一个数据变更，然后以闪电般的速度将数据安全送达目的地Doris。无论是日常的数据同步任务，还是紧急的数据迁移需求，它都能完美完成任务。

"这简直太酷了！"小张兴奋地说道。有了这样一位数据英雄的帮助，他对接下来的任务充满了信心。

正如蚁人在量子领域穿梭自如，Flink Doris Connector在数据世界中也展现出同样的灵活性。

"蚁人"不仅能处理实时数据流，还能执行批量数据处理，甚至能够在两种模式之间自由切换，就像在不同维度间穿行的超级英雄。

## "蚁人"的实战手册

"小张，周一的销售数据大屏怎么样了？" 产品经理焦急地询问。

"放心吧，已经部署上线了！" 小张自信地说。

这一周，他通过Flink Doris Connector的几大"秘技"，成功解决了实时数据同步的难题。让我们一起来看看这位"蚁人"英雄的实战绝技。

首先来看整体作战流程图：

#### 技能一：闪电同步术

"产品说要实时看到订单数据，这可怎么办？"小张回忆起接到任务时的紧张。直到他发现了Flink CDC这个神器：

只需几行SQL，就能实现MySQL到Doris的实时同步：

```
-- enable checkpoint
SET'execution.checkpointing.interval' = '10s';

CREATETABLE cdc_mysql_source (
idint
  ,nameVARCHAR
  ,PRIMARY KEY (id) NOTENFORCED
) WITH (
'connector' = 'mysql-cdc',
'hostname' = '127.0.0.1',
'port' = '3306',
'username' = 'root',
'password' = 'password',
'database-name' = 'database',
'table-name' = 'table'
);

-- 支持同步 insert/update/delete 事件
CREATETABLE doris_sink (
idINT,
nameSTRING
) 
WITH (
'connector' = 'doris',
'fenodes' = '127.0.0.1:8030',
'table.identifier' = 'database.table',
'username' = 'root',
'password' = '',
'sink.properties.format' = 'json',
'sink.properties.read_json_by_line' = 'true',
'sink.enable-delete' = 'true',  -- 同步删除事件
'sink.label-prefix' = 'doris_label'
);

insertinto doris_sink selectid,namefrom cdc_mysql_source;
```

"酷！数据像闪电一样快速同步过来了！" 小张兴奋地说。

#### 技能二：多维表合体术

数据同步搞定了，但产品经理又提出新需求："能不能把用户的城市信息也显示出来？"

这时，Lookup Join技能派上了用场：

通过Lookup Join，可以将实时流的数据实时关联Doris城市维度信息：

```
CREATE TABLE fact_table (
`id`BIGINT,
`name`STRING,
`city`STRING,
`process_time`as proctime()
) WITH (
'connector' = 'kafka',
  ...
);

createtable dim_city(
`city`STRING,
`level`INT ,
`province`STRING,
`country`STRING
) WITH (
'connector' = 'doris',
'fenodes' = '127.0.0.1:8030',
'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
'table.identifier' = 'dim.dim_city',
'username' = 'root',
'password' = ''
);

SELECT a.id, a.name, a.city, c.province, c.country,c.level 
FROM fact_table a
LEFTJOIN dim_city FOR SYSTEM_TIME ASOF a.process_time AS c
ON a.city = c.city
```

#### 技能三：整库迁移术

"小张，老板说要把所有业务库的数据都全量+增量同步到Doris数仓，这个..."

"别担心，整库同步我也在行！" 小张笑着说。只见他快速敲下命令：

```
<FLINK_HOME>bin/flink run \
    -Dexecution.checkpointing.interval=10s \
    -Dparallelism.default=1 \
    -c org.apache.doris.flink.tools.cdc.CdcTools \
    lib/flink-doris-connector-1.16-24.0.1.jar \
    mysql-sync-database \
    --database test_db \
    --mysql-conf hostname=127.0.0.1 \
    --mysql-conf port=3306 \
    --mysql-conf username=root \
    --mysql-conf password=123456 \
    --mysql-conf database-name=mysql_db \
    --including-tables "tbl1|test.*" \
    --sink-conf fenodes=127.0.0.1:8030 \
    --sink-conf username=root \
    --sink-conf password=123456 \
    --sink-conf jdbc-url=jdbc:mysql://127.0.0.1:9030 \
    --sink-conf sink.label-prefix=label \
    --table-conf replication_num=1 
```

"看，整个数据库都在自动同步了，包括表结构、数据变更，都能自动处理！"

在经过一系列实战后，小张总结出了几点经验：

1. 流模式开启Checkpoint很重要，它能保证数据的一致性
2. 合理设置批量写入参数，比如几万一批次可以大幅提升整体性能
3. 适当的缓存能显著提升维表关联效率
4. Schema变更时要特别注意数据类型的匹配

"滴滴滴..." 监控大屏上的数据开始实时滚动，产品经理露出了满意的笑容。

## "蚁人"的进阶修炼

一个平静的下午，小张正在悠闲地喝着咖啡，突然...

"小张，救命！数据大屏卡住了！" 产品经理慌张地跑过来。

"别慌，这种情况我已经见过很多了。" 小张放下咖啡，打开了他的"蚁人装备优化手册"。

让我们看看小张是如何进行性能调优的：

#### 写入加速秘籍

"看，这是我的独门秘籍，" 小张打开配置文件：

```
CREATE TABLE doris_sink (
    id INT,
    name STRING,
    order_time TIMESTAMP
) WITH (
    'connector' = 'doris',
    'sink.enable.batch-mode' = 'true',
    'sink.buffer-flush.max-rows' = '5000000',
    'sink.buffer-flush.max-bytes' = '100MB',
    'sink.buffer-flush.interval' = '10s',
    'sink.properties.format' = 'json'
);
```

"不需要流模式ckp时，大规模数据通过攒批模式写入性能提升了N倍！" 小张自豪地说。

#### 查询提速法则

对于经常需要关联的维度表，小张使用了缓存神器：

```
CREATE TABLE dim_table (
    id INT,
    info STRING
) WITH (
    'connector' = 'doris',
    'lookup.cache.max-rows' = '100000',
    'lookup.cache.ttl' = '60s',
    'lookup.max-retries' = '3'
);
```

"缓存一开，查询速度嗖嗖的！但是要注意资源使用率，别负载了。"

#### 常见问题破解

突然，监控系统报警：

```
errCode = 2, detailMessage = Label [label_0_1] has already been used
```

"啊，这个我遇到过，" 小张迅速打开他的问题速查手册翻找：

**Label重复问题**

```
# 异常日志：
errCode = 2, detailMessage = 
Label [label_0_1] has already been used, relate to txn [19650]

-- 解决方案：更换label前缀
'sink.label-prefix' = 'timestamp_prefix'
```

**事务超时问题**

```
# 异常日志：
errCode = 2, detailMessage = 
transaction [19650] not found

# FE配置调整
streaming_label_keep_max_second = 43200  # 12小时
```

**并发写入限制**

```
# 异常日志：
errCode = 2, detailMessage = 
current running txns on db 10006 is 100, larger than limit 100

# 调整并发数
max_running_txn_num_per_db = 1000
```

...

小张的运维经验：

1. 定期检查Checkpoint

* 监控Checkpoint完成时间
* 观察数据积压情况

2. 资源规划

* 合理设置TaskManager内存
* 适当调整并行度

3. 监控指标关注

* totalFlushLoadBytes：已写入字节数
* flushTotalNumberRows：处理总行数
* writeDataTimeMs：写入耗时

...

"搞定！" 小张轻松地靠在椅子上，"有了这些优化技巧，再大的数据量也不怕了。"

这时，办公室的同事们都围了过来：

"小张，你这招太厉害了！"

"能不能分享更多实战经验？"

"什么时候开个分享会呗？"

小张笑着说："好好好...**Doris生态的奥秘无穷无尽，我们一起探索，一起进步！**"

看着数据大屏上流畅滚动的数据，小张知道，这只是Flink Doris Connector精彩故事的开始。接下来，还会有更多的挑战，更多的优化空间，等待着这位"蚁人"去探索和征服。

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

[【Apache Doris】*Manager* 极致丝滑地运维管理](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482971&idx=1&sn=b0953da7f28e016edd032a5be9bd34e5&chksm=f374eb7dc403626b97e57c4e2ec4922e3c17fa71fd2714623cd84328716a53b839976e59f0c3&scene=21#wechat_redirect)

[【Apache Doris】如何实现*高并发*点查？（原理+实践全析）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483049&idx=1&sn=e60a95fe683a663f249af230e6606bce&chksm=f374eb0fc403621998ca50c01bad8e093f788a982ca33463d28863840e3bfd3c94791f38efb9&scene=21#wechat_redirect)

[*为什么*Apache Doris适合做大数据的复杂计算，MySQL不适合？](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483064&idx=1&sn=330709a47b85d247af7510b1557fe868&chksm=f374eb1ec40362088903ac541bd4ea3d1d4ea7962b95214405132d4a0a96fae6e23c671787d6&scene=21#wechat_redirect)

[*深夜*无需加班，Apache Doris让数据自己会跑](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650485454&idx=1&sn=a73a4a6e78b610cdc47d5e31293d871a&chksm=f374f0a8c40379be78b40cab31721c31f485af5272299ef7f8dbfdd30cff7dc6fcc27b49eb4e&scene=21#wechat_redirect)

[别让你的CPU打盹儿：Apache Doris*并行*执行原理大揭秘！](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650486255&idx=1&sn=8e8c4f95293ed2451c2085efbb72fc72&chksm=f374f789c4037e9f5a981d03ce8bdb90a9cf5f26e3f268d177708bcbea62b567774a1536627b&scene=21#wechat_redirect)

[如何正确地使用Chat*GPT*（角色扮演+提示工程）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482824&idx=1&sn=429e15f252a79bf18f72161c9ccc04af&chksm=f374eaeec40363f8aff13083a2ee0ef60b07ab56ac6c7aaff5ae282a5092aae82d00455c1633&scene=21#wechat_redirect)

[*超强*满血不收费的AI绘图教程来了（在线Stable Diffusion一键即用）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482860&idx=1&sn=ad75b434b7eae3511017c67d14dad143&chksm=f374eacac40363dc192372fbdd36ff76ad8cbdd866df5a3287c3e48476f4c9c88f2ef3e5f3db&scene=21#wechat_redirect)

点击下方蓝字关注一臻数据