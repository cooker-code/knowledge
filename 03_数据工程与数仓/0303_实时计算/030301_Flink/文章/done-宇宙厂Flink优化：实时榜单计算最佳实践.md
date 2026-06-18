> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink调优方法论|Flink调优方法论]]
---
title: 宇宙厂Flink优化：实时榜单计算最佳实践
author: 熊大数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486294&idx=1&sn=4ab511381ff73b31107ada027281e2bd&chksm=e91e1d139c6eff83cd6fe1d20bb38a7c0457d93bf20034bbfb1031cbc33755c2bbf6888383f0&mpshare=1&scene=24&srcid=0506D6Y7rMK2AnoSBO5YegKT&sharer_shareinfo=96cd28334ca816cd7048fc3da9af2138&sharer_shareinfo_first=96cd28334ca816cd7048fc3da9af2138#rd
---

我是大熊！某大厂数据部门负责人

我们有一些实时需求：电商的销量排行榜、最受欢迎的商品、点击率最高的广告等

这些需求的本质是一个Top N的计算，尤其是在实时流处理中，如何高效地计算并维护前N个元素是关键。

今天内容涉及到源码分析，目录如下：

一、排序Top 10的FlinkSQL实现

二、优化方案 - 面试加分项

三、源码进阶 - 搞定底层原理

四、拓展思考

一、排序Top 10的FlinkSQL实现

业务场景

提取每个日期支付金额排名前10的卖家信息。

涉及`shopsales`表进行聚合计算，按`seller_id`和`stat_date`分组，计算每个卖家在每个日期的支付订单金额总和（`pay_ord_amt`）

具体实现

FlinkSQL没有提供Top N的具体语法，而是使用OVER窗口子句和过滤条件来表达。我们会想到用row\_number来处理，例如：

```
-- 直接上大招【最终版】SELECT  seller_id,  stat_date,  pay_ord_amtFROM (    SELECT      seller_id,      stat_date,      pay_ord_amt,      ROW_NUMBER() OVER (        PARTITION BY stat_date        ORDER BY pay_ord_amt DESC      ) AS rownum    FROM (        SELECT          seller_id,          stat_date,          SUM(total_fee) FILTER           (WHERE total_fee >= 0)            AS pay_ord_amt        FROM          shopsales        GROUP BY          seller_id,          stat_date      ) AS subquery) AS ranked_queryWHERE  rownum <= 10;
```

二、优化方案-面试加分项

1. 避免输出 rownum的值

而是采用前端展示的时候，按（`pay_ord_amt`）进行排序，进而显示行号。这样的可以显著的减少数据量。示例：销售额更新后的数据量变化

无排名输出时

只输出记录的实际变化，当记录在TopN范围内移动时，只需要输出一次更新。不需要维护和输出排名信息，输出数据量 = 实际变化的记录数

有排名输出时

需要输出每条记录的排名变化，会影响到其他记录的排名，需要输出所有受影响的记录的排名更新。输出数据量 = 实际变化的记录数 + 排名变化的记录数

2. 增加 Top N的 Cache大小

```
table.exec.rank.topn-cache-size: 200000
```

TopN为了提升性能，有一个State Cache层，Cache层能够提升对State的访问效率。Cache命中率的计算公式：

```
cache_hit = cache_size*parallelism/top_n/partition_key_num
```

当PartitionBy的Key维度较大（如10万级别）时，Top100缓存10000条、并发50的情况下，Cache命中率仅为5%（10000×50/100÷100000），大量请求会直接访问磁盘，导致性能下降，State Seek Metric也可能出现毛刺。

因此，建议在PartitionKey维度较大时，适当增加TopN缓存大小（如从10000条调整到200000条），并相应增加TopN节点的堆内存。

3. 数据倾斜

分组后的TopN计算，如果某个大卖家搞活动，某seller id的数据量极大，可能导致某些任务节点负载过高。可以通过 开启局部/全局聚合，将聚合分为两个阶段：首先在上游（shuffle 之前）进行局部聚合，然后在下游（shuffle 之后）进行全局聚合。局部聚合与其上游算子链接在一起，这样可以避免 shuffle 造成的数据倾斜。

严重的话，可能需要通过预处理来分散数据，比如加盐或随机前缀，然后再合并处理，这样可以缓解单个key的压力。代码重写：

```
SELECT    seller_id,    stat_date,    SUM(pay_ord_amt) AS total_pay_ord_amtFROM (    SELECT        seller_id,        stat_date,        total_fee AS pay_ord_amt,        MOD(HASH_CODE(seller_id), 1024) AS hash_bucket    FROM        shopsales    WHERE        total_fee >= 0) AS subqueryGROUP BY    seller_id,    stat_date,    hash_bucket;
```

最终改写后的效果

不过加盐的方法会增加计算的复杂度，需要在子分组内进行再聚合，需要具体分析是否值得这样做。   

三、源码进阶，直击底层原理

这里Row\_number排序涉及3 种Rank算法，性能依次降低：

* AppendRank：仅支持插入状态的静态数据
* UpdateFastRank：仅满足下面条件才支持

+ 输入数据产生更新变化
+ 输入UPSERT键必须包含OVER查询的分区键
+ OVER查询的排序字段必须是单调，单调的方向与排序方向相反

* RetractRank：支持所有查询，原因是它存储了所有输入数据，所以状态特别大。

Row\_number的主要源码实现在（flink-table-runtime/rank/AbstractTopNFunction）这里，我们主要关注UpdateFastRank，该函数支持多种排名方式：

```
public enum RankType {    // 正常排序    ROW_NUMBER,      // 相同值排名，跳过后续排名    RANK,            // 相同值排名，跳过后续排名    DENSE_RANK }
```

核心的数据结构：

```
// 存储从排序键到行键列表的缓冲区，用于维护TopN排序private transient TopNBuffer buffer;
// 存储从行键到记录的映射，包含行数据和内部排名private transient Map<RowData, RankRow> rowKeyMap;
// 存储从分区键到其rowKeyMap的映射，用于分区级别的缓存private transient Cache<RowData, Tuple2<TopNBuffer, Map<RowData, RankRow>>> kvRowKeyMap;
```

核心排序流程

```
private void processElementWithRowNumber(RowData inputRow, Collector<RowData> out) throws Exception {    // 1. 获取排序键和行键    RowData sortKey = sortKeySelector.getKey(inputRow);    RowData rowKey = rowKeySelector.getKey(inputRow);
    if (rowKeyMap.containsKey(rowKey)) {        // 2. 处理更新记录        RankRow oldRow = rowKeyMap.get(rowKey);        RowData oldSortKey = sortKeySelector.getKey(oldRow.row);
        if (sortKeyComparator.compare(sortKey, oldSortKey) == 0) {            // 2.1 排序键未改变，只更新行内容            Tuple2<Integer, Integer> rankAndInnerRank = rowNumber(sortKey, rowKey, buffer);            rowKeyMap.put(rowKey, new RankRow(inputRowSer.copy(inputRow), rankAndInnerRank.f1, true));        } else {            // 2.2 排序键改变，需要重新排序            buffer.remove(oldSortKey, rowKey);            int size = buffer.put(sortKey, rowKey);            rowKeyMap.put(rowKey, new RankRow(inputRowSer.copy(inputRow), size, true));            updateInnerRank(oldSortKey);        }    } else if (checkSortKeyInBufferRange(sortKey, buffer)) {        // 3. 处理新记录        int size = buffer.put(sortKey, rowKey);        rowKeyMap.put(rowKey, new RankRow(inputRowSer.copy(inputRow), size, true));    }}
```

排名计算

```
private Tuple2<Integer, Integer> rowNumber(RowData sortKey, RowData rowKey, TopNBuffer buffer) {    Iterator<Map.Entry<RowData, Collection<RowData>>> iterator = buffer.entrySet().iterator();    int curRank = 1;    while (iterator.hasNext()) {        Map.Entry<RowData, Collection<RowData>> entry = iterator.next();        RowData curKey = entry.getKey();        Collection<RowData> rowKeys = entry.getValue();
        if (curKey.equals(sortKey)) {            // 在相同排序键内计算内部排名            Iterator<RowData> rowKeysIter = rowKeys.iterator();            int innerRank = 1;            while (rowKeysIter.hasNext()) {                if (rowKey.equals(rowKeysIter.next())) {                    return Tuple2.of(curRank, innerRank);                }                innerRank += 1;                curRank += 1;            }        } else {            curRank += rowKeys.size();        }    }}
```

流程说明

1.数据输入阶段：

* 接收输入数据

* 提取排序键和行键

* 判断是更新还是新行

2.更新处理阶段：

* 如果是更新，检查排序键是否改变

* 如果排序键未变，只更新行数据

* 如果排序键改变，重新计算排名

3.新行处理阶段：

* 计算当前排名

* 查找合适的插入位置

* 更新缓冲区和状态

4.排名计算阶段：

* 遍历缓冲区计算排名

* 处理相同排序键的情况

* 维护内部和外部排名

5.状态更新阶段：

* 更新 rowKeyMap

* 更新 buffer

* 处理排名变化

6.输出阶段：

* 发送更新前事件

* 发送更新后事件

* 输出结果

四、拓展思考

1. 状态管理

由于TopN通常需要维护一个有序集合，状态可能会增长得很快。Flink的状态TTL可以用来设置状态的过期时间，特别是当数据具有时效性时，及时清理过期状态能减少内存占用。

2. 使用 Window Top

这在某些版本中已经支持，可能更高效。另外，考虑到Flink的流处理模型，可能更倾向于使用增量计算而非全量计算，这可以减少每次处理的数据量。

3. 代码改造

在代码层面，如果使用DataStream API，自定义的ProcessFunction可能需要高效的管理状态结构。优先使用优先级队列或者TreeMap等结构来维护TopN元素，避免每次全量排序。

4. 数据延迟

在允许一定延迟的情况下，设置适当的allowedLateness和侧输出，可以确保TopN结果的正确性，但这也可能增加状态存储的压力，需要衡量延迟容忍和资源消耗。

参考：

1.https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sql/queries/topn/

2.https://www.alibabacloud.com/help/tc/flink/user-guide/optimize-flink-sql?spm=a2c63.p38356.0.i14#section-b0v-rga-gqc

以上就是所有内容，大厂最新的优化实践。

个人微信

新建了一个免费知识星球，只分享干货

加VX进入

**往期精彩文章合集**

【数据建模】

 [数仓专家如何进行数据调研？](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485454&idx=1&sn=0cb476d43665eea03750ab2562ba9ad9&scene=21#wechat_redirect)

 [从业务到数仓-网约车平台Gra建模设计](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485035&idx=1&sn=9ec02657a9d224e8ecbbb94ae5750918&scene=21#wechat_redirect)

【数据性能优化】

 [直播大促如何确保实时大屏Flink稳定性](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485259&idx=1&sn=4ccb54ea0487534e551960924a513d13&scene=21#wechat_redirect)？

 [SQL千亿数据膨胀OOM优化经验](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247484805&idx=1&sn=7082c25af5d666a54756007770303ee7&scene=21#wechat_redirect)

【面试经验】
 [明知我只写SQL，为何面试狂考动态规划](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485491&idx=1&sn=0e13cf91fc90fd2a58651b973367b689&scene=21#wechat_redirect)？
 [宇宙厂数据岗 3-1 初面，等消息...](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486036&idx=1&sn=afc7122b4da23ca0d1ff1ebf97e066eb&scene=21#wechat_redirect)

【工作十年】

[做数开，别把路走窄了](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247485682&idx=1&sn=4c77735ce24319509c116cd45417702e&scene=21#wechat_redirect)

[入职阿里，跟不上节奏](https://mp.weixin.qq.com/s?__biz=MzIyNzc2OTQ5NQ==&mid=2247486072&idx=1&sn=227bd9fbc857eebc51f43df75c80e6d4&scene=21#wechat_redirect)