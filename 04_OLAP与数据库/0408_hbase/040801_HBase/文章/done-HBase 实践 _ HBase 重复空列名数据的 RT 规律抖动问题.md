> 已吸收至：[[04_OLAP与数据库/0408_hbase/040801_HBase/040801_核心知识点/HBaseCompaction与低延时调优边界|HBaseCompaction与低延时调优边界]]
---
title: HBase 实践 | HBase 重复空列名数据的 RT 规律抖动问题
author: HBase技术社区
date:
url: http://mp.weixin.qq.com/s?__biz=MzU5OTQ1MDEzMA==&mid=2247492593&idx=1&sn=08994141e0ea2af04c3ab71780475ea2&chksm=feb6148cc9c19d9a51fd6e24ca79fbcb6daac6d51822b6650492b41e9ac64da9add5b6eb4686&mpshare=1&scene=24&srcid=0517ALlApmVWl2K7FuD94g2K&sharer_shareinfo=a8e29d21e4d0fb7b7765baad7d78fc59&sharer_shareinfo_first=a8e29d21e4d0fb7b7765baad7d78fc59#rd
---

## 现象

在我们公司HBase作为Apache hugegraph的存储引擎，共同⽀撑着公司⻛控相关的图查询场景， 并已经在线上平稳运⾏⼀年以上，但有⼀⽇发现实时图谱业务RSGroup的节点⼀直存在规律性的rt与 cpu抖动，周期（约1h）。起初因为抖动的峰值并不⾼，所以并未过于重视。⽽在某⽇cpu的峰值异常 波动，严重影响了实时图谱查询的rt。

|  |  |
| --- | --- |
| ⽇常cpu与rt的抖动规律 | 14⽇cpu抖动峰值异常⾼    ‍                **图谱 P99** |

## 现象分析

### 2.1 监控分析

初步从监控分析，发现⼀个奇观的现象：

🚫 异常情况⼀：同⼀张表（g\_oe），Get请求rt受影响，Scan请求并未受到 影响。

从表级别的监控，发现异常情况：

🚫 异常情况⼆：Get RT的周期与flush操作，时间线上⾼度相关 图谱在有两条链路进⾏双跑，两边的读写流量均⼀样。从两个HBase集群的监控发现异常情况：

🚫 异常情况三：两个集群异常情况同步，⾮主机硬件问题。

使⽤arthas在集群cpu⾼峰期抓取了⽕焰图，同时结合集群的io+cpu监控，发现异常情况：

🚫 异常情况四：耗CPU的操作为Get请求。

结合以上异常现象，初步得出结论：

✅ 初步结论：⽆硬件问题，纯CPU运算。Get请求的某些特殊设置引发了组件bug：跳表性能问题 or 其他？

❓ 存疑点：

1. 1. Get的延时与Flush有何关联？
2. 2. 跳表性能太差还是调⽤太频繁？

### 2.2 数据侧分析

抽样⽤⼾数据，发现：

🚫 异常情况⼀：列名为空 

到这⾥，怀疑：**列名为空是否为引发跳表的性能问题？**

紧接着，从数据层⾯排查出同⼀个⽤⼾短时间内存在⼤量的重复数据。

|  |  |
| --- | --- |
| **只有txmodify变化** | **短时间内频繁更新** |

🚫 异常情况⼆：同⼀个rowkey的数据，短时间内存在⼤量重复数据 

    接着让业务在备⽤链路集群停⽌了实时写⼊的任务，观察⼀⼩时后，集群cpu的抖动不再出现。

✅ 初步结论：对重复数据的Get请求会触发跳表性能问题 or 其他？

## 代码分析

### 3.1 重复数据对请求的影响

#### 3.1.1 源码分析

    结合⽕焰图中的堆栈，最终定位到 `org.apache.hadoop.hbase.regionserver.StoreScanner#next` 这个⽅法中的 matcher.match(cell) ⽅法总是返回 SEEK\_NEXT\_COL 这个状态码，从⽽触发对跳表的数据搜 索。最终在 `org.apache.hadoop.hbase.regionserver.querymatcher.ScanQueryMatcher#getK eyForNextColumn` 这个⽅法尝试跳到下⼀个column时发现，代码对于列名⻓度为0时有⼀些特殊的 处理。

查阅代码注释中的HBASE-18471，发现问题点。

背景知识点：

1. 1. HBase的KeyVaule在是有序的，按照（rowkey增序，列族增序，列名增序，timestamp降序， type降序）的⽅式排列
2. 2. 在Scan的状态转换中，如果 SEEK\_NEXT\_COL ，会修改当前cell的timestamp为 Long.MIN\_VALUE，从⽽在seek时直接跳过当前column历史版本数据。
4. 假设现在MemStore中存在如下数据：

|  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- |
| 数据写⼊顺序（ID） | rowkey | 列族 | 列名（空⽩代 表列名为空） | timestamp | type |
| 1 | 1 | cf | a | t1 | Put |
| 2 | 1 | cf |  | t2 | DeleteFamily |
| 3 | 1 | cf |  | t3 | Put |
| 4 | 1 | cf |  | t4 | Put |

代码处理流程如下：

|  |  |  |
| --- | --- | --- |
| 代码版本 | 代码处理步骤 | 后果 |
| HBASE-18471之前 | 1. ⾸先查到ID=4的数据 2. 对⽐ID=4和ID=3的两条数据，在matcher.match(cell) 中应该返回SEEK\_NEXT\_COL 3. 紧接着 ScanQueryMatcher#getKeyForNextColumn中，通过修改当前cell的time=Long.MIN\_VALUE，从而通过索引直接跳过ID=2的cell | 此时ID=1的数据本来应该是  被删除了，不应该被读出来 |
| HBASE-18471之后 | 1. ⾸先查到ID=4的数据 2. 对⽐ID=4和ID=3的两条数据，在matcher.match(cell) 中应该返回SEEK\_NEXT\_COL 3. 紧接着 ScanQueryMatcher#getKeyForNextColumn对列名为空做了特殊处理，通过修改当前cell的type=Minimum（⽐Put⼩），从⽽会顺序的扫描ID=3和ID=2的Cell | 避免因为跳过了  DeleteFamily引发的脏读问  题。 |

2. ✅ 初步结论：数据重复+空列名，会导致Seek Next Column时需要⽐较所有 重复的数据，从⽽占⽤CPU。

#### 3.1.2 数据验证

    通过在`org.apache.hadoop.hbase.client.TestFromClientSide3#testScanAfterDeletingSpecifiedRow`方法做一些微调验证上述结论。

    将下述两个验证的代码，分别覆盖带`testScanAfterDeletingSpecifiedRow`中的数据构建那一块，在`org.apache.hadoop.hbase.regionserver.StoreScanner#next`的`ScanQueryMatcher.MatchCode qcode = matcher.match(cell);`进行DEBUG，可以发现符合上述的结论。

```
Put put = new Put(row);
put.addColumn(FAMILY, QUALIFIER, VALUE);
t.put(put);
Delete d = new Delete(row);
t.delete(d);
put = new Put(row);
put.addColumn(FAMILY, null, value0);
t.put(put);
put = new Put(row);
put.addColumn(FAMILY, null, value1);
t.put(put);
put = new Put(row);
put.addColumn(FAMILY, null, Bytes.toBytes("value_2"));
t.put(put);

结论：
该场景下，每一个Cell都会进行比较，比较的MatchCode如下：
INCLUDE -> SEEK_NEXT_COL -> SEEK_NEXT_COL -> SKIP -> SEEK_NEXT_COL
Cell(value2) -> Cell(value1) -> Cell(value0) -> Cell(delete) -> Cell(Value)

可以发现，rowkey、列族相同且列名为空时，所有的老版本数据都会进行一次比较，虽然每一次的结果都是 SEEK_NEXT_COL ...
```

```
验证2 ：列名不为空时，是否能跳过value0这个Cell

Put put = new Put(row);
put.addColumn(FAMILY, Bytes.toBytes("thstQualifier"), VALUE); // the timestamp of cell is less than the timestamp of delete family 
t.put(put);
Delete d = new Delete(row);
t.delete(d);
put = new Put(row);
put.addColumn(FAMILY, QUALIFIER, value0);
t.put(put);
put = new Put(row);
put.addColumn(FAMILY, QUALIFIER, value1);
t.put(put);
put = new Put(row);
put.addColumn(FAMILY, QUALIFIER, Bytes.toBytes("value_2"));
t.put(put);

结论：
该场景下，部分Cell会进行比较，比较的MatchCode如下：
SKIP -> INCLUDE -> SEEK_NEXT_COL -> SEEK_NEXT_COL
Cell(delete) -> Cell(value2) -> Cell(value1) -> Cell(VALUE) 

可以发现，rowkey、列族、列名相同时会直接过滤掉旧版本数据(Cell(value0))，不会进行多余的比较。
```

### 3.2 Flush对Get延时的影响

    从2.1章节已经知道，Get请求的延时周期规律与Flush操作在时间线上存在⾼度相关性。结合⽇志 和监控，发现存在规律：Flush均由MemStoreChore线程触发，时间间隔为1h，⽽Get RT的抖动周期 也差不多为1h。结合2.3.1章节中数据重复的问题，有⼀个初步结论：

📌 初步猜想：Flush将MemStore的数据落盘时会去重，Get的延时就降低了

#### 3.2.1 测试验证

|  |  |  |
| --- | --- | --- |
| 步骤描述 | 命令 | 现象&分析 |
| 创建测试表 | create 'zx','f' | 创建只有⼀个Region的表，同时version=1 |
| 写⼊1条数据 | 1. put 'zx','1','f:name','xiao' 2. scan 'zx' |  |
| 覆盖写⼊数  据 | 1. put 'zx','1','f:name','zhang' 2. scan 'zx' 3. scan 'zx',   {RAW=>true,VERSIONS=>2} | 此时数据还在MemStore中，同⼀个rowkey+列簇+列  名，有多个版本的数据 |
| 执⾏flush，  检查HFile是  否去重 | 1. flush 'zx' 2. 打印hfile内容 | 执⾏flush落盘后，⽂件中只保留了最新版本的数据。 |
| 再查询⼀次  表 | 1. scan 'zx',   {RAW=>true,VERSIONS=>2} |  |

    通过在测试环境验证，确定了上述猜想的准确性，去重后没有了3.1中的问题，对Get的rt确实是有帮助的。

#### 3.2.2 代码分析

    通过对Flush过程的源码分析，Flush memstore其实就是通过构建scanner读取memstore数据， 过程与get的实现⼀样，在match时也会不断获得SEEK\_NEXT\_COL ，也会由于列名为空导致⽐较所 有重复的数据，⽆法直接跳过column历史版本数据。但是Scanner返回的结果中，依旧会过滤掉不需 要的数据（历史的版本），这样在flush后，storeFile中就不会存在重复数据，因此get时⽆需再⽐较重 复数据，RT下降。

## 解决⽅案

    从上⾯的分析中可知，正常情况下，只要列名不为空，就不会出现当前的问题。但对于当前业务 来说，把数据的列名补⻬，改造周期和难度都⽐较⼤。

    通过对 `org.apache.hadoop.hbase.regionserver.StoreScanner#next` ⽅法的代码 分析，发现只要 `matcher.match(cell)` ⽅法返回状态 `INCLUDE` ，便不会触发上述的逻辑。最后 在 `org.apache.hadoop.hbase.regionserver.querymatcher.UserScanQueryMatcher# matchColumn 的 MatchCode matchCode = columns.checkColumn(cell, typeByte);` 发现只要查询时指定列名，并且命中便会返回状态 `INCLUDE`。 

    在hugegraph查询HBase时，增加了如下的条件： 

```
byte[] EMPTY_BYTES = new byte[0]; scan.addColumn(family, qualifier);
```

    最后上线后成功结果问题，效果如下：