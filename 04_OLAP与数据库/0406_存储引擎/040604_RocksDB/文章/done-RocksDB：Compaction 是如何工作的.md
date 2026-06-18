> 已吸收至：[[04_OLAP与数据库/0406_存储引擎/040604_RocksDB/040604_核心知识点/RocksDBCompaction读写空间放大边界|RocksDBCompaction读写空间放大边界]]
---
title: RocksDB：Compaction 是如何工作的
author: 编程宇航员
date: 干货干货
url: https://mp.weixin.qq.com/s?__biz=MzAwNjExNTU1MQ==&mid=2247489561&idx=1&sn=19517ca09708760e124a5d43ca9c96a0&chksm=9aa96e1d0976fa0a287da6dc6b1e0d4700fa19760522c8a44caa0226771176b49aaba3e05b1f&mpshare=1&scene=24&srcid=0519ory3U86tpGfRzyQfMFph&sharer_shareinfo=5b340d161d5f21e3ab6b0b61ca4d84fc&sharer_shareinfo_first=5b340d161d5f21e3ab6b0b61ca4d84fc#rd
---

线上用 RocksDB 的系统，最常见的抱怨通常不是“写不进去”，而是写着写着开始抖。磁盘带宽看起来没打满，CPU 也没有长时间 100%，但 P99 写入延迟突然拉长；过一会儿又恢复，随后再来一波。与此同时，目录里的 `.sst` 文件越来越多，`rocksdb.stats` 里 compaction pending、write stall、L0 文件数这些指标也开始冒头。

这类问题很容易被误判成“机器不行”或者“磁盘慢”。但在 RocksDB 里，很多写入抖动其实来自同一个后台动作：Compaction。它一边帮你把新写入的数据从内存推到磁盘，一边把磁盘上的 SSTable 分层合并；它能降低读放大和空间放大，也会制造写放大。调 RocksDB，本质上经常是在调这三种放大之间的边界。

先把一条写入放进真实链路里看。



## 写入先落在 MemTable，不是直接改 SSTable

RocksDB 是 LSM-tree 形态的存储引擎。一次 `Put`、`Delete` 或 `WriteBatch` 进来后，数据通常先写 WAL，再进入 MemTable。默认 MemTable 可以理解成内存里的有序结构，常见实现是 SkipList。写入会带上 sequence number，用来在后续查询、合并和快照里判断哪个版本更新。

这个设计的好处是前台写入不需要在磁盘上原地修改某个 page。SSTable 是不可变文件，一旦生成就不再被局部更新。写入路径变成追加 WAL 和更新内存结构，前台延迟更稳定，随机写也被转换成顺序写和后台批处理。

问题也从这里开始。MemTable 不能无限长大。达到 `write_buffer_size` 之类的阈值后，当前 MemTable 会变成 immutable memtable，新的写入切到另一个 MemTable，后台 flush 线程把这个 immutable memtable 排序后写成一个 SST 文件。这个新 SST 一般进入 L0。

L0 很特殊。它不是一个按 key range 严格切好的层，而是一批按 flush 时间产生的文件。多个 L0 文件的 key range 可以大量重叠，因为每个 MemTable 都可能包含任意 key。比如下面三次写入分别 flush：

```
flush-1.sst: user:001 ... user:900flush-2.sst: user:100 ... user:800flush-3.sst: user:050 ... user:950
```

这三个文件在 L0 里不能靠 key range 排除彼此。一次点查 `user:500` 时，RocksDB 可能要按新到旧检查多个 L0 文件，结合 Bloom filter、索引块和 sequence number 才能判断真正结果。L0 文件数越多，读放大越明显，写入也越容易被后面的 compaction 追上。

## Flush 只是在还债，不是在消灭债务

Flush 的动作看起来像“把内存落盘”，但它没有解决 LSM 的根本问题：同一个 key 的多个版本、删除标记和重叠文件仍然存在，只是从内存换到了 L0。

假设一个订单状态被反复更新：

```
seq=101  order:7 -> createdseq=205  order:7 -> paidseq=318  order:7 -> shippedseq=420  delete order:7
```

这些版本可能分散在 MemTable、L0、L1 甚至更底层。读的时候要找到对当前 snapshot 可见的最新版本；空间上，旧版本和 tombstone 也不能立即删除，因为还要考虑快照、迭代器、事务和更底层是否仍有旧数据。

Compaction 真正做的事，是把一批有序输入流合并成新的有序输出文件。在合并过程中，RocksDB 可以丢掉被新版本覆盖、对当前可见性已经没用的旧版本；在安全条件满足时，也可以让删除标记和被删除的数据一起消失。换句话说，flush 只是把写入从内存交给 LSM，compaction 才是整理历史、回收空间、降低读取路径长度的动作。

这里有个容易忽略的边界：Compaction 不会凭空免费完成。它要读旧 SST，解压 block，合并 key/value，重新生成新 SST，再写回磁盘。前台写入越快，后台整理越追不上，L0 和上层文件就越容易堆起来；后台整理越积极，磁盘写入又会被 compaction 放大。

## Level Compaction 怎样把 L0 变成分层 SST

RocksDB 默认常见的是 level style compaction。准确说，官方文档也把 RocksDB 的 level compaction 描述成一种 tiered + leveled 的混合：MemTable 和 L0 更像 tiered，多份新 run 先堆在一起；从 L1 开始，层内文件通常按 key range 分片，并尽量保持互不重叠。



一次典型的 L0 -> L1 compaction，可以按这几个动作理解：

1. 后台线程发现 L0 文件数、大小或 compaction score 超过阈值。
2. 从 L0 选择一批文件作为输入，因为 L0 文件互相重叠，经常不能只挑一个很小的范围。
3. 找出这些 L0 文件在 L1 中重叠的 SST 文件，一起作为输入。
4. 多路归并这些有序流，按 sequence number 和可见性保留结果。
5. 按 `target_file_size_base` 之类的配置切成新的 SST，安装到新的 Version 里。
6. 等没有快照、迭代器或旧 Version 引用后，旧 SST 才能被删除。

从 L1 往下合并时，选择范围通常更窄。因为 L1、L2、L3 这些层内 key range 基本不重叠，RocksDB 可以选择上一层某个文件，再找到下一层与它 key range 重叠的文件。官方文档把这种方式称为 some-to-some：不是把某一层全量和下一层全量合并，而是只合并发生重叠的一段。

这也是 level compaction 能控制读放大和空间放大的原因。L1 之后每层同一个 key range 通常只有一个候选文件，点查时不会在同一层扫很多文件；旧版本也会随着层层下推被淘汰。代价是明显的：同一份用户数据可能在 L0、L1、L2、L3 之间被反复读取和重写。

## 为什么 L0 文件多会直接拖垮读写

很多 RocksDB 抖动，都能在 L0 找到影子。L0 太多会先伤读，因为 L0 文件 key range 互相重叠，点查需要检查更多候选文件。即使 Bloom filter 帮你跳过很多不可能命中的文件，每个 filter、index、metadata 的判断也不是完全免费；如果 filter 不在 cache，问题会更明显。

L0 太多也会伤写。后台 compaction 如果追不上 flush，新 MemTable 还在不断变成 L0 文件。到 `level0_slowdown_writes_trigger` 附近，RocksDB 会主动放慢前台写入；到 `level0_stop_writes_trigger`，可能直接停写等待后台追赶。这个时候应用层看到的就是写入延迟尖刺。

所以只看平均写吞吐很容易被骗。系统可能在大部分时间写得很快，但每隔一段时间被 L0 和 compaction debt 拉住。更有效的观察方式，是同时看：

* `rocksdb.num-files-at-level0`
* `rocksdb.compaction-pending`
* `rocksdb.mem-table-flush-pending`
* `rocksdb.actual-delayed-write-rate`
* `rocksdb.stall-micros`
* `rocksdb.stats` 里的 compaction read/write bytes

这些指标放在一起，能看出问题是前台写入真的太大，还是后台 flush/compaction 线程、磁盘带宽、压缩 CPU、cache 抖动跟不上。

## 写放大：为什么写 10MB 可能让磁盘写 50MB

写放大指的是底层存储实际写入字节和用户写入字节的比值。官方调优文档给过一个很直接的判断方式：如果数据库写入是 10MB/s，而磁盘写入观测到 30MB/s，写放大就是 3。

在 RocksDB 里，写放大不只来自 WAL 和 flush，更主要来自 compaction。用户写入一条 key/value 后，它先进入 WAL，之后 flush 成 L0 SST；随后可能被 L0 -> L1 重写，再被 L1 -> L2 重写，最后继续往更底层合并。level style 的目标是把数据整理成读起来更友好的形状，但整理过程中会反复写同一份逻辑数据。

写放大高的时候，系统的真实瓶颈可能不是用户写入量，而是 compaction 消耗的磁盘带宽。比如磁盘顺序写能力 500MB/s，用户只写 50MB/s，看起来还很宽松；但如果综合写放大接近 10，磁盘已经被后台写满，前台写入自然会被拖住。SSD 场景还要额外考虑擦写寿命。

减少写放大的常见方向，是降低数据被反复改写的次数。可以增大 MemTable，让每次 flush 形成更大的 SST；可以调大 level size ratio，让数据经过更少层；可以对写多读少的场景考虑 universal compaction；也可以通过 bulk load 或 ingest external SST 把有序数据直接放到更合适的层。每个方向都有代价：内存占用、读放大、临时空间、恢复时间或者实现复杂度都会变化。

## 读放大：为什么 Compaction 又是在帮读请求还账

读放大指的是一次逻辑读需要访问多少内部结构。点查通常会先看 mutable MemTable，再看 immutable memtables，然后检查 L0 到更底层的 SST。官方博客里提到，L0 文件按 flush 时间组织，key range 往往互相重叠，所以可能要检查多个 L0 文件；从 L1 往下，层内 key range 互斥，可以通过文件元数据和索引缩小候选范围。

Compaction 帮读请求省掉的成本就在这里。它把重叠文件合并，把较新的版本压到更底层的有序结构里，把旧版本和无效删除清掉。L0 文件少了，点查少看几个候选；层内不重叠，范围查询也能更顺地按 key order 走。

但读放大不是只由文件数决定。Bloom filter 可以减少不命中点查的磁盘读取；block cache 可以让索引块、filter block、data block 命中内存；prefix extractor 能让前缀查询更准；压缩策略会影响 CPU 和 IO 的平衡。Compaction 负责整理形状，cache 和 filter 负责减少真正读盘，两者经常要一起看。

这也是为什么“把 compaction 调弱一点”不能只看写吞吐。短期内写放大可能下降了，但 L0 文件和上层重叠 run 增多，读请求会在更多候选里兜圈。如果业务是 Kafka Streams 状态存储、在线特征查询、去重索引这类读写混合场景，读放大的恶化会很快反映到尾延迟上。

## 空间放大：旧版本和临时输出都要占地方

空间放大指磁盘上数据库文件大小和逻辑有效数据大小的比值。它有两层来源。

第一层是稳定存在的冗余。旧版本、tombstone、过期但尚未合并到底层的数据，都会让磁盘上保存的字节多于当前有效数据。只要 compaction 还没把覆盖关系处理完，这些字节就会留着。

第二层是 compaction 过程中的临时空间。合并一批输入 SST 时，新输出文件要先写出来，旧文件不能马上删。只有新的 Version 安装成功，并且没有读请求还引用旧 Version，旧 SST 才能成为 stale file 被清理。对大 compaction 来说，这个临时空间可能很可观。

Universal compaction 在这里尤其要小心。它通常用更低写放大换取更高读放大和空间放大；官方调优文档也提醒，universal compaction 过程中可能临时带来接近一倍的额外空间需求。因此大库或单个 column family 很大的场景，不能只因为“写放大小”就直接切过去。

## 三种放大不是三个独立旋钮

把写放大、读放大、空间放大分开讲，是为了看清成本；真正调参时，它们很少能单独下降。



Level compaction 更愿意花后台写入，把 SST 整理成层内少重叠的形状。它通常读放大和空间放大更可控，但写放大可能高。Universal compaction 更像按大小相近的 run 做归并，少改写下一层已有数据，写放大更低；代价是同一层或同一范围可能保留更多 run，读放大和临时空间更难压。FIFO compaction 则更适合缓存、日志型、按时间淘汰的数据，它不追求保留所有历史版本的最优查询形状，而是到期或超限就丢旧文件。

所以选 compaction style 之前，先要问业务问题：

* 写入是不是持续高峰，磁盘写带宽是不是主要瓶颈。
* 点查和范围查询占比有多高，能不能接受更多候选 SST。
* 数据是否有大量覆盖写、删除、TTL 或过期清理。
* 磁盘空间有没有足够余量承受临时输出。
* 是否允许通过分片把单个 RocksDB 实例变小。
* 重启恢复、备份、快照和迭代器会不会长期持有旧 Version。

这些条件比“某个参数推荐值”更重要。RocksDB 的调优经常不是把某个值调到更大，而是确认瓶颈到底在前台写、后台 IO、压缩 CPU、cache 命中、L0 文件数，还是空间回收滞后。

## 一个写入抖动案例应该怎么拆

假设线上看到 RocksDB 写入 P99 周期性升高，应用日志里没有明显慢 RPC，机器层面的 `iostat` 显示磁盘写入有脉冲，但平均利用率不算夸张。这个时候可以先别急着换盘，先看 compaction debt。

第一步看 L0 文件数是不是周期性上升。如果 `num-files-at-level0` 往上冲，同时 `compaction-pending=1`，说明后台已经欠账。再看 `stall-micros` 和 delayed write rate，如果它们也跟着上来，前台写入已经被 RocksDB 主动限速。

第二步看 flush 和 compaction 哪个慢。如果 immutable memtable 堆积，可能是 flush 线程、WAL/fsync、目标目录写入慢；如果 L0 不断增加但 flush 正常，更多是 L0 -> L1 或后续 level compaction 追不上。`rocksdb.stats` 里每层的 compaction bytes、micros、pending bytes 能帮你分清是哪一层在堆。

第三步看 compaction 慢在哪里。压缩算法太重时，CPU 会成为瓶颈；value 很大、block cache 被挤、读输入 SST 不顺时，读 IO 会变成瓶颈；输出文件太大、后台线程太少、多个 column family 抢同一个 Env 线程池，也会让 compaction 排队。

第四步才是调参数。写入峰值短而猛，可以适当增大 write buffer 或增加 `max_write_buffer_number`，让前台有更大缓冲，但这会增加内存和恢复成本。后台追不上，可以调整 `max_background_jobs`，但前提是磁盘和 CPU 还有余量。L0 太容易触发，可以看 `level0_file_num_compaction_trigger`、target file size 和 level size；但如果只是把触发线放宽，读放大可能先恶化。

## 删除为什么不一定马上释放空间

很多人第一次观察 RocksDB 删除数据，会困惑：明明已经 delete 了，目录大小为什么没立刻下降？原因是 `Delete` 写入的通常是一个删除标记。它要等 compaction 证明更底层没有需要被它遮住的旧版本，且没有快照需要看到旧视图时，才能连同旧值一起被丢掉。

这件事在有长迭代器、长 snapshot、事务或备份时尤其明显。只要旧 Version 还被引用，SST 文件就不能马上删除。即使新的 compaction 已经产生了更干净的输出，旧文件也可能因为读者还在使用而短暂保留。

TTL 和 periodic compaction 也是同一个思路。它们不是魔法删除文件，而是让数据有机会经过 compaction filter 或底层合并流程。真正释放空间，还是要等合并、可见性判断和旧文件清理都走完。

## Compaction 不是后台小事，而是 LSM 的价格标签

RocksDB 把前台随机写变成了 WAL + MemTable + SSTable 的顺序化路径，这是它适合高写入场景的基础。但这条路径不是免费午餐。Flush 把内存变成 L0 文件，compaction 再把 L0 的重叠 run 推向更底层；每推进一步，都在写放大、读放大和空间放大之间做选择。

写多读少、允许更多临时空间的场景，可以牺牲一部分读放大换低写放大。读多写少、点查尾延迟敏感的场景，通常更愿意用后台写入把 SST 形状整理好。空间紧张的场景，要格外小心 universal compaction、大范围删除、长快照和 compaction backlog。

真正稳定的 RocksDB 配置，不是让 compaction “尽量少跑”，也不是让它“越积极越好”。它应该让前台写入速度、后台整理能力和磁盘空间余量保持同一个节奏。只要这三者有一个长期落后，欠账就会以 L0 文件数、write stall、读放大或磁盘膨胀的形式回来。

## 参考入口

* RocksDB Wiki: Compaction
* RocksDB Wiki: Leveled Compaction
* RocksDB Wiki: RocksDB Tuning Guide
* RocksDB Blog: Indexing SST Files for Better Lookup Performance
* RocksDB Blog: Bulkloading by ingesting external SST files

- END -