---
title: ClickHouse：向量化执行如何压榨 CPU
author: 编程宇航员
date: 干货干货
url: https://mp.weixin.qq.com/s?__biz=MzAwNjExNTU1MQ==&mid=2247489615&idx=1&sn=c791353ad283c1a42027723bdfc577d6&chksm=9ae9cf7c5b607554b82befdc4462f4c955c25545fdd4733e0043d26961b9b6479360c90b4304&mpshare=1&scene=24&srcid=05296oigI973nffwye9uXgWI&sharer_shareinfo=6304c2c6f6d521242097d9987e7ca379&sharer_shareinfo_first=6304c2c6f6d521242097d9987e7ca379#rd
---

ClickHouse 跑分析查询时，经常会出现一个很有意思的现象：CPU 打得很满，但查询并不一定慢。相反，如果 CPU 利用率不高、线程在等磁盘、等网络、等小块调度，查询才更可能抖。列式存储只是前半段，真正把 CPU 吃起来的是后半段执行模型：数据按 `Block` 批次流动，列以 `Column` 形式连续存放，函数一次处理一整列或一段列，Pipeline 把读取、过滤、表达式计算、聚合和排序拆成可并行的处理器。

这和传统“一行一行执行表达式”的模型很不一样。行执行里，每来一行就解释一次表达式、做一次类型分派、调用一次函数、处理一堆分支。向量化执行把这些动作攒成批：一次拿几千到几万行，按列连续访问，在紧凑循环里做比较、加法、哈希、过滤和拷贝。CPU 更喜欢这种活：连续内存、少分支、少虚函数、更多机会让编译器自动向量化或走手写 SIMD 路径。

先从一个最普通的查询看数据怎么流。

  

## Block 是 ClickHouse 执行里的批次单位

ClickHouse 内部不是把每一行作为执行单元往下传，而是把一批行装成一个 `Block`。一个 `Block` 可以理解成一组等长列，每列有名字、类型和 `Column` 数据对象。比如这条查询：

```
SELECT    tenant_id,    count() AS c,    sum(value) AS sFROM eventsWHERE event_date = '2026-05-24'  AND event_type = 'click'GROUP BY tenant_id;
```

从 MergeTree 读出来时，ClickHouse 不需要把整行所有字段都取出来。它只读过滤和计算需要的列：`event_date`、`event_type`、`tenant_id`、`value`。这些列被组成一个个 `Block` 往下游传。过滤算子在一个 `Block` 上生成 filter mask，表达式算子在 `Column` 上批量计算，聚合算子把一批 key/value 喂进哈希表。

这个批次大小不是越大越好。太小，函数调用和调度开销摊不薄；太大，缓存压力、内存峰值和延迟都会上来。ClickHouse 有 `max_block_size` 这类设置控制处理块大小，很多源码和文档也会围绕 `Block`、`Chunk`、`Processor` 这些对象描述执行链路。

批处理的核心收益在这里：调度一次，处理很多行；类型检查一次，循环很多元素；读一段连续内存，让 CPU cache 和预取器有机会工作。

## Column 为什么比 Row 更容易喂饱 CPU

行式布局里，一行记录经常长这样：

```
tenant_id | event_date | event_type | value | extra_json | ...
```

如果查询只需要 `value`，CPU 仍然要跨过很多不相关字段。对分析查询来说，这会浪费内存带宽，也破坏缓存局部性。ClickHouse 的列式布局把同一列的数据放在一起：

```
tenant_id:  [42, 42, 7, 9, 42, ...]value:      [1.2, 0.8, 3.4, 5.1, ...]event_type: ["click", "view", "click", ...]
```

对 `sum(value)` 这种计算，执行器可以沿着 `value` 列连续扫描。对 `tenant_id = 42` 这种过滤，可以连续比较一整段 `tenant_id`。如果列是定长数值类型，底层数组尤其适合紧凑循环和 SIMD；如果是字符串，ClickHouse 也会把字符数据和 offsets 分开，让许多操作先在 offsets 或字节段上批量处理。

  

这也是为什么 ClickHouse 的函数接口通常不是“输入一行，返回一个值”，而是“输入若干列，输出一列”。一个函数看到的是 `ColumnPtr`、类型、行数和上下文。它要么批量生成结果列，要么对常量列、Nullable 列、LowCardinality 列做专门路径。这样函数内部可以把循环写成对数组的批处理，而不是每行都重新走一遍通用逻辑。

## Function 批量执行省掉了什么

拿一个简单表达式看：

```
WHERE value * 100 > 50
```

行执行可以想象成这样：

```
for each row:  read value  call multiply  call greater  branch by result
```

向量化执行更像这样：

```
input column: value[0..N)tmp column:   value * 100filter:       tmp > 50apply filter to all referenced columns
```

表面上多了中间列，实际省掉的是大量逐行解释和逐行分派。函数实现可以一次拿到整列类型，选择一条具体执行路径，然后在 tight loop 里处理 N 个元素。对 `UInt64 + UInt64`、`Float64 * Float64`、`Date` 比较这类简单操作，循环体非常规整，CPU 能做流水线、预取、向量化和分支预测。

ClickHouse 里很多函数还会专门处理常量列。比如 `value * 100` 中的 `100` 不需要展开成 N 个元素；函数可以识别常量参数，用一个标量和整列相乘。Nullable 也不是每行都做复杂对象判断，而是有 null map，先批量处理值列，再合并 null 信息。

真正省下来的成本包括：

* 每行函数调用和虚分派。
* 每行表达式解释。
* 对不需要列的内存访问。
* 随机访问和结构体跨字段读取。
* 分支不稳定带来的流水线停顿。

这些成本单看每行都很小，但一秒处理几亿行时，会变成 CPU 时间的大头。

## Filter 是一段 mask，不是每行马上返回

过滤在向量化执行里也很关键。`WHERE` 条件通常先生成一个 `UInt8` filter column，也就是每行 0/1 的 mask。随后 ClickHouse 用这个 mask 去过滤其他列：保留命中的行，丢掉不命中的行。

这比逐行“判断后立刻把整行丢给下游”更适合列式数据。因为 filter mask 本身是连续数组，下游可以对每列独立执行批量过滤。对数值列，是按 mask 选择元素；对字符串列，是按 offsets 和 chars 复制命中的段；对 LowCardinality 列，则可能先过滤字典 key。

一个简化流程是：

```
event_type column  -> equals(event_type, 'click')  -> filter mask [1,0,1,0,0,1...]  -> apply mask to tenant_id/value/ts columns
```

这里也有边界。过滤选择率太低时，生成和应用 mask 的成本可能主要花在跳跃复制；选择率很高时，过滤本身省不掉多少后续计算。ClickHouse 的执行器会有一些优化路径，但大方向不变：过滤先变成列式 mask，再作用到整个 `Block`。

## Pipeline 把批处理串成可并行的流

向量化解决的是一个算子内部怎么高效处理一批数据；Pipeline 解决的是多个算子怎么并行流动。ClickHouse 的查询执行会被拆成 processors，比如从 MergeTree 读取、表达式计算、过滤、聚合、排序、合并结果。`EXPLAIN PIPELINE` 可以看到这些处理器如何连接、哪里有并行分支、哪里需要汇总。

一条查询大概会变成这样：

```
ReadFromMergeTree x N  -> Expression  -> Filter  -> AggregatingTransform  -> Resize / StrictResize  -> MergingAggregated  -> Output
```

读取端可以按 part、mark range、线程并行拆开。每个读取线程不断产出 `Block`，下游表达式和过滤继续处理。聚合阶段通常先做局部聚合，每个线程维护自己的哈希表或局部状态；最后再合并这些局部结果。这样做的好处是减少全局锁争用，也让每个线程尽量在本地数据上连续工作。

  

Pipeline 不是越宽越好。线程太少，CPU 吃不满；线程太多，内存、上下文切换、哈希表合并和磁盘读取都会抖。`max_threads`、读取任务数量、part/mark 数量、远端 shard、聚合基数都会影响最终并行度。很多 ClickHouse 查询慢，不是因为某个函数不会 SIMD，而是 Pipeline 某个阶段变成窄口：读取不够并行、聚合合并太重、排序必须集中、网络汇总拖住。

## SIMD 友好不等于所有地方都用 SIMD

SIMD，中文可以叫单指令多数据。它的直觉很简单：一个 CPU 指令同时处理多个数据元素。比如一次比较多个 `UInt32`，一次累加多个 `Float64`。列式连续数组给 SIMD 提供了很好的输入形态。

但不能把 ClickHouse 的快简单归因成“用了 SIMD”。真正重要的是 SIMD 友好的路径：

```
连续内存  -> 类型稳定  -> 分支少  -> 循环体简单  -> 批次足够大  -> 编译器或手写代码能向量化
```

如果函数里有复杂字符串解析、正则、JSON 提取、用户自定义逻辑、字典查找或大量分支，SIMD 能帮的部分就有限。列式批处理仍然能减少调度和解释开销，但瓶颈可能转向内存分配、哈希表、字符串比较、解压、网络或 I/O。

所以说 ClickHouse “压榨 CPU”，不是每一步都跑满 SIMD 寄存器，而是尽量把数据组织成 CPU 喜欢的样子：按列连续、按块批量、按阶段流水、能并行就并行，不能并行的地方尽量缩小集中处理范围。

## 聚合为什么先局部再合并

分析查询里，聚合经常是 CPU 大头。`GROUP BY tenant_id` 看起来简单，实际要把每个 block 里的 key 计算 hash，找或插入聚合状态，再更新 `count`、`sum`、`uniq` 等状态。高基数下，哈希表访问会变成大量随机内存访问；低基数下，状态更新又会非常密集。

ClickHouse 通常会先在线程局部做聚合。每个线程处理自己的 block 流，维护局部 hash table。等局部结果生成后，再进入合并阶段，把多个局部状态合成最终结果。

这个设计有几个收益：

* 前半段避免多个线程同时抢一个全局 hash table。
* 每个线程能在自己的批次上连续处理，减少锁和同步。
* 低基数聚合可以在局部先缩小数据量。
* 分布式查询里，每个 shard 也可以先做局部聚合，再回 coordinator 合并。

代价是内存。局部 hash table 多了，峰值内存会上升；合并阶段也可能成为瓶颈。如果 group by key 基数很高、状态很大，或者局部聚合根本压不下数据，查询会从“CPU 紧凑循环”变成“内存随机访问 + hash table 扩容 + 状态合并”。这类场景里，向量化仍然有帮助，但瓶颈已经不只是 SIMD。

## Block 太小和太大都会出问题

批处理依赖合适的 Block 大小。Block 太小，函数每次处理的行数少，虚函数调用、类型分派、processor 调度、内存分配这些固定成本摊不薄。CPU 还没进入稳定循环，数据就处理完了。

Block 太大，也会出问题。中间列、filter mask、聚合状态、排序缓冲都会变大。CPU cache 放不下，内存带宽压力升高，单个 processor 持有数据时间变长，下游等待也会增加。对交互式查询来说，过大的批次还可能拉高首批结果延迟。

ClickHouse 用 `max_block_size` 之类的设置控制许多处理阶段的目标块大小，但真实 block 行数还受读取粒度、part/mark、过滤选择率、聚合和远端传输影响。调这个参数时不能只看吞吐，也要看内存峰值、延迟、CPU cache miss 和 pipeline 是否出现背压。

一个实用判断是：如果 ProfileEvents 里函数执行、表达式计算、聚合都很多，但 CPU 利用率不高，可能不是 Block 太大，而是上游读不动或下游卡住；如果 CPU 很高但每行成本异常，才更值得看表达式、函数、字符串处理、Block 大小和数据类型。

## 字符串和 Nullable 会破坏一部分理想路径

数值列最适合向量化。`UInt64`、`Float64`、`DateTime` 这类定长列是连续数组，比较、加减、聚合都容易写成紧凑循环。字符串就复杂一些。ClickHouse 的字符串列通常是 chars 数据加 offsets 数组，访问第 i 个字符串要看前后 offset。批处理仍然存在，但每个元素长度不一样，CPU 分支和内存访问更难预测。

Nullable 也会增加成本。它通常有一个 null map。函数不仅要处理值列，还要传播 null 信息。对简单函数，ClickHouse 可以批量合并 null map；对复杂函数，null 判断可能进入内层循环。LowCardinality 能减少重复字符串存储和比较，但也引入字典和 key 的解码路径。

所以 schema 设计会影响向量化效果。能用数值编码就不要把高频过滤列做成复杂字符串；能避免无意义 Nullable 就不要把每列都设成 Nullable；LowCardinality 适合低基数字符串，但不是所有字符串列都应该套。执行器很强，但输入数据形态越友好，它越能把 CPU 用在真正计算上。

## 查询慢时怎么判断是不是执行路径问题

ClickHouse 给了几类很实用的观察入口。

`EXPLAIN PIPELINE` 看执行图。它能告诉你读取阶段有多少并行流，表达式、过滤、聚合、排序和合并在哪里发生，哪个阶段变成单路。看到最后有很重的 merge/sort，或者中间某个 `Resize` 之后并行度突然收窄，就要警惕 Pipeline 瓶颈。

`system.query_log` 看行数、字节、ProfileEvents 和内存。关注 `read_rows`、`read_bytes`、`result_rows`、`memory_usage`、`ProfileEvents` 里的 selected marks、read bytes、function execute、aggregation、rows before limit 等信号。读很多但过滤后很少，可能是索引/mark 裁剪问题；读不多但 CPU 很高，可能是函数或聚合重。

`system.trace_log` 或采样 profiler 看热点函数。字符串函数、正则、JSON、哈希表、解压、排序、聚合状态合并，各自的热点形态不一样。不要看到 CPU 高就默认“向量化没生效”。CPU 高可能正是执行器在努力工作，只是表达式太重或数据形态不友好。

`EXPLAIN indexes = 1` 和 `EXPLAIN PIPELINE` 要一起看。前者回答“少读了吗”，后者回答“读出来以后怎么处理”。如果前者已经读了太多 mark，执行器再向量化也只是高效地处理了过量数据；如果前者裁剪很好但后者聚合/排序很重，问题就更靠近执行 pipeline。

  

## 向量化不是替代好 SQL 的魔法

向量化执行能把 CPU 用得更充分，但它不能替代数据裁剪、排序键设计和合理表达式。一个没有命中 Primary Key 的查询，读了几十亿行，再高效的批处理也要付出解压、过滤、聚合成本。一个到处 `JSONExtract`、正则匹配、复杂字符串转换的查询，连续列布局也救不了所有 CPU。

它真正擅长的是这种路径：

```
尽量少读列  -> 尽量少读 mark  -> 以 Block 为单位批量传递  -> 以 Column 为单位连续计算  -> 过滤和表达式生成新列  -> 局部聚合压缩中间结果  -> Pipeline 并行合并
```

这里每一步都在减少固定开销、减少不必要数据、扩大批处理粒度、提高 CPU cache 和 SIMD 机会。ClickHouse 的快不是某个单点技巧，而是从磁盘布局到执行器接口都在配合“按列、按块、按批次”。

## 真正压榨 CPU 的是整条路径

ClickHouse 的向量化执行，不只是把 for 循环从一行改成一列。它是一整条路径：MergeTree 按列读出需要的数据，`Block` 把一批列组织成执行批次，`Column` 用连续内存承载同类型数据，`Function` 批量处理输入列并输出新列，`Pipeline` 让多个处理器并行消费 block，聚合和排序再在必要位置做局部和全局合并。

这条路径成立时，CPU 做的是它擅长的事：连续读、批量算、少分支、少调度、并行流动。路径被破坏时，瓶颈也很具体：读太多 mark、Block 太碎、字符串函数太重、Nullable 太多、聚合状态太大、Pipeline 某个阶段变窄、远端汇总拖住。

所以评估 ClickHouse 查询性能，不能只问“有没有向量化”。更好的问题是：数据有没有被裁剪到足够少，列布局是否适合批量计算，函数是否走紧凑路径，Pipeline 是否有足够并行度，最后的合并是否成为瓶颈。把这些问题串起来，才能看懂 ClickHouse 为什么能把 CPU 压到很满，也能看懂它什么时候会把 CPU 花在不该花的地方。

## 参考入口

* ClickHouse Docs: Query execution resources and limits
* ClickHouse Docs: EXPLAIN Statement
* ClickHouse Docs: Analyzer
* ClickHouse Docs: system.query\_log
* ClickHouse Docs: system.trace\_log
* ClickHouse Blog: ClickHouse vs PostgreSQL: A complete comparison

- END -