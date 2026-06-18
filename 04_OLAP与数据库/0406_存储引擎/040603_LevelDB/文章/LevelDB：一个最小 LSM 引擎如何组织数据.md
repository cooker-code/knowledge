---
title: LevelDB：一个最小 LSM 引擎如何组织数据
author: 编程宇航员
date: 干货干货
url: https://mp.weixin.qq.com/s?__biz=MzAwNjExNTU1MQ==&mid=2247489580&idx=1&sn=f8bf1739fb07aeb0146622be761559e2&chksm=9ad8ce95c549628f54d50ec8aee1e06430cb5f6e6a35e4e92426010f39369e3fbfa06737dada&mpshare=1&scene=24&srcid=0522CTMK3dbZ3kUjqWMKUyfl&sharer_shareinfo=5ef05a783d315122453e412d103a76ea&sharer_shareinfo_first=5ef05a783d315122453e412d103a76ea#rd
---

很多人第一次看 LevelDB，会觉得它不像一个“完整数据库”。没有 SQL，没有事务引擎，没有复杂的后台调度，也没有 RocksDB 后来扩展出来的一堆 Column Family、RateLimiter、Compaction Style。它更像一个被压到很小的 LSM 引擎样本：写入先进 WAL 和 MemTable，内存满了变成 Immutable MemTable，后台刷成 SSTable，Manifest 记录哪些 SSTable 构成当前版本。

这个“小”反而适合把 LSM 的主链路看清楚。线上很多存储系统的基本动作，其实都能在 LevelDB 里找到原型：为什么写入要先追加日志，为什么 SSTable 不原地修改，为什么 L0 文件会重叠，为什么重启后能找回最新状态，为什么删除只是一个标记，为什么 compaction 会影响读写延迟。

先从一条 `Put("user:7", "active")` 进入系统开始。

  

## 写入为什么要同时进 WAL 和 MemTable

LevelDB 的前台写入会先进入日志文件，也就是 WAL。磁盘目录里它通常是一个编号文件，形如 `000123.log`。日志是顺序追加的，保存最近更新；同一批更新还会被插入当前 MemTable。MemTable 是内存里的有序结构，LevelDB 里用 SkipList 保存 internal key。

这两个动作解决的是两件事。WAL 负责崩溃恢复：进程挂了，内存里的 MemTable 没了，重启时可以重放 log。MemTable 负责最新读可见性：写入刚完成，还没生成 SSTable，读请求也应该能读到它。

所以 LevelDB 官方实现文档里会说，当前 log 的一份拷贝保存在内存结构里，每次读都会查询它。这句话很朴素，但它正是 LSM 写入模型的核心：磁盘先保底，内存先服务读，后台再慢慢把内存整理成有序文件。

一个写入批次大概可以这样看：

```
WriteBatch:  seq=101 Put("user:7", "active")  seq=102 Put("user:8", "disabled")  
append to 000123.loginsert into MemTable by internal keyreturn to caller
```

这里的 sequence number 很重要。LevelDB 不是只按 user key 排序，而是按 internal key 排序。Internal key 可以理解成：

```
user_key + sequence_number + value_type
```

同一个 user key 多次写入时，新版本靠 sequence number 区分。删除也不是把旧值擦掉，而是写入 value type 为 deletion 的 internal key。后续读取和 compaction 都要理解这套版本顺序。

## WAL 文件为什么是按块和记录组织的

WAL 看起来只是“追加日志”，但 LevelDB 给它定义了一个很具体的格式。官方 log format 文档里写得很清楚：log 文件由 32KB block 组成，每个 block 里是一组 record。每条 record 有 checksum、length、type 和 data。type 可以是 `FULL`，也可以是 `FIRST/MIDDLE/LAST`，用来表示一个用户记录跨 block 被切成多个片段。

这个格式服务于两个工程目标。第一，崩溃或损坏后容易重新同步。读 log 时可以跳到下一个 32KB block 边界继续扫，不需要靠猜测寻找下一条记录。第二，大的 WriteBatch 不需要一次塞进某个固定大小记录里，它可以按 block 边界拆成多个 fragment。

对理解 LevelDB 来说，不需要背每个字节的编码，但要知道 WAL 不是“随便写文本”。它是一种可顺序扫描、可校验、可恢复的二进制日志。MemTable 丢了以后，LevelDB 就靠这些 log record 重新构建最近还没落成 SSTable 的更新。

## MemTable 满了以后为什么要先变 immutable

MemTable 不能无限增长。默认配置里，log 文件增长到大约 4MB 时，LevelDB 会切换一组新的写入目标：创建新的 log 文件和新的 MemTable，未来写入进新对象；旧 MemTable 不再接收写入，变成 Immutable MemTable，简称 imm。

这个状态切换很关键。前台写入不能等旧 MemTable 慢慢刷盘后再继续，否则一次 flush 就会把所有写请求堵住。LevelDB 的做法是把“还能写”和“等待刷盘”分开：

```
当前写入:  log = 000124.log  mem = mutable MemTable  
等待落盘:  old log = 000123.log  imm = Immutable MemTable
```

读路径也要同时看这两份内存结构。先查 mutable MemTable，再查 immutable MemTable，然后才查 SSTable。因为 imm 虽然不再接收新写入，但里面的数据仍然比磁盘上的旧 SSTable 新。

这里的代价也很直接。后台如果来不及把 imm 写成 SSTable，LevelDB 就不能无限制造新的 imm。写入可能被放慢甚至等待。最小 LSM 引擎看起来很简单，但“前台继续写、后台慢慢刷”的节奏一旦失衡，写延迟照样会抖。

## Flush 生成的 SSTable 长什么样

后台线程把 Immutable MemTable 按 key 顺序遍历，生成一个新的 SSTable。LevelDB 文档里这个文件扩展名写作 `*.ldb`，也常被称为 table file。它是不可变的，一旦生成就不在原地修改。

  

一个 SSTable 的主体是有序 key/value，被切成多个 Data Block。Data Block 之后是 Meta Block，比如 Filter Block；然后是 Metaindex Block、Index Block，最后是固定大小的 Footer。Footer 里保存 metaindex 和 index block 的 BlockHandle，BlockHandle 又包含 offset 和 size。

一次点查进入某个 SSTable 时，不会从文件开头一路扫到结尾。LevelDB 会借助 index block 找到可能包含目标 key 的 data block；如果启用了 filter policy，还能通过 filter block 先判断某个 key 是否可能在对应 block 附近。最终真正保存 key/value 的，仍然是 Data Block。

可以把 SSTable 理解成这样一组职责：

```
Data Block    : 保存排序后的 key/valueFilter Block  : 用少量位图减少无效读Index Block   : 从 key 边界定位 Data Block 的 offset/sizeFooter        : 告诉读者 index 和 metaindex 在哪里
```

这个格式解释了 LSM 为什么适合后台批量写。MemTable 本来就是有序结构，flush 时顺序写出一批 Data Block，再补上索引和 footer。前台不需要在磁盘上随机更新旧页，随机写被推迟成批量顺序写。

## L0 为什么是特殊的一层

Immutable MemTable flush 出来的 SSTable 会进入 Level 0。L0 特殊在于：它的文件来自不同时间点的 MemTable，每个文件都可能覆盖任意 key range，所以 L0 文件之间可以重叠。

比如三次 flush 之后可能得到：

```
L0/000201.ldb: [user:001 ... user:900]L0/000205.ldb: [user:100 ... user:800]L0/000209.ldb: [user:050 ... user:950]
```

点查 `user:500` 时，LevelDB 不能只靠 key range 排除其中两个。它要按文件新旧顺序检查多个 L0 文件，因为同一个 key 的新版本可能在更晚 flush 的文件里。

从 L1 开始，LevelDB 维持一个更规整的约束：同一层内文件的 key range 不重叠。这样读某个 key 时，每层最多找一个候选文件。代价是后台 compaction 要持续把 L0 的重叠文件合并到 L1，再从 L1 往更深层推进。

LevelDB 官方实现文档给了几个很有代表性的默认尺度：L0 文件数量超过阈值时会触发向 L1 的合并；L1 大约 10MB，L2 大约 100MB，后续层级按约 10 倍增长；输出 SSTable 目标大小大约 2MB。这些数字本身不是重点，重点是它们体现了一个最小 leveled LSM 的形状：上层小而新，下层大而旧，新更新通过 compaction 逐步迁移到底层。

## Manifest 记录的是“当前有哪些 SSTable”

写入和 flush 只解释了数据怎么变成文件，还没解释另一个问题：重启之后，LevelDB 怎么知道哪些 `.ldb` 文件还属于当前数据库版本，哪些是 compaction 产生的临时文件，哪些已经过时？

这个答案在 Manifest。

Manifest 文件不是保存用户 key/value 的地方。它保存的是元数据变更日志，也就是 VersionEdit：新增了哪个 table file、删除了哪个 table file、每个文件属于哪一层、key range 是什么、当前最大 sequence number 到哪里。LevelDB 当前服务的视图，可以理解成从 Manifest 重放出来的一组 Version。

磁盘目录里还有一个 `CURRENT` 文件，它很小，只保存当前最新 Manifest 的文件名。重启时，LevelDB 先读 `CURRENT`，找到 `MANIFEST-xxxxxx`，再读取 Manifest，把各层 SSTable 的元数据恢复出来。

这套设计有个重要好处：Manifest 是追加式的。Compaction 完成后，不需要重写一大份全量元数据，只要把“删除旧文件、增加新文件”的 VersionEdit 追加进去。等 Manifest 变大或数据库重新打开时，再创建新的 Manifest 快照当前状态。

## 一次 Flush 如何提交到 Manifest

Immutable MemTable 写成 SSTable 后，还不能只因为文件存在就认为它属于数据库。LevelDB 必须完成一个状态提交：

```
imm -> build 000201.ldb   -> 生成 file metadata: level=0, number=201, smallest/largest key   -> append VersionEdit to MANIFEST   -> install new Version   -> old log / old imm 可以被清理
```

这一步解决的是崩溃一致性。假设 SSTable 文件已经写出来，但 Manifest 还没记录，重启后 LevelDB 不应该把它当成当前版本的一部分；它只是一个可能残留的未提交文件，后续可以清理。反过来，如果 Manifest 已经记录某个 table file，重启恢复时就必须能找到它。

这也是为什么 Manifest 不只是“目录清单”。它是 LevelDB 元数据提交协议的一部分。WAL 保证最近写入能重放，SSTable 保存批量有序数据，Manifest 说明哪些 SSTable 被当前版本引用。

## 读请求怎样穿过内存和多层 SSTable

读路径能把这些组件串起来。一次 `Get("user:7")` 大致按这个顺序走：

```
mutable MemTable  -> immutable MemTable  -> L0 files, newer files first  -> L1 file whose range covers key  -> L2 file whose range covers key  -> ...
```

MemTable 和 Immutable MemTable 保存最新写入，所以先查。L0 文件可能重叠，所以要检查多个候选，并且更晚生成的文件优先。L1 之后同层文件不重叠，查找范围更窄。

这个顺序也解释了 deletion marker 的语义。删除在 LevelDB 里也是一条 internal key。如果读路径在高层先看到某个 key 的删除标记，并且 sequence number 对当前读可见，就应该返回 NotFound，即使更底层 SSTable 里还有旧值。旧值什么时候真正消失，要等 compaction 证明下面没有需要被它遮住的数据。

最小 LSM 引擎的读路径并不神秘，但它很容易被文件数量拖慢。L0 文件多了，读请求要合并更多候选；层级变深了，miss 查找要一路试探；filter 没命中 cache，点查会多读元数据。写入顺序化带来的代价，最终会在读路径和 compaction 上还回来。

## Compaction 如何把版本往下推

Compaction 是 LevelDB 里最核心的后台动作。它会从某一层选择一个或多个输入文件，再找下一层 key range 重叠的文件，一起归并成新的更深层 SSTable。旧文件从 Version 里删除，新文件加入 Version，变化追加到 Manifest。

L0 到 L1 的 compaction 比较特殊，因为 L0 文件互相重叠。一次 L0 compaction 可能会选择多个 L0 文件，并把它们与所有重叠的 L1 文件合并。非 L0 的 compaction 更规整：从 level L 选一个文件，再选 level L+1 中所有重叠文件，输出到 level L+1。

Compaction 做的不只是搬文件。它会合并同一个 user key 的多个版本，丢掉被覆盖的旧值；在安全条件满足时，也会丢掉 deletion marker。官方实现文档里提到，如果没有更高编号层的文件范围还覆盖当前 key，删除标记就可以被丢掉。这句话背后是同一个原则：不能让底层旧值在未来读路径里重新出现。

  

## 重启恢复时这几类文件怎么配合

真正能检验一个存储引擎组织是否清楚的，是崩溃恢复。LevelDB 重启时大致按这条链路恢复：

1. 读取 `CURRENT`，找到最新 committed Manifest。
2. 读取 Manifest，恢复各层 SSTable 的元数据和全局 sequence 信息。
3. 清理不再被当前 Version 引用的过时文件。
4. 找到 Manifest 之后仍需要重放的 log 文件。
5. 读取 log record，重建 MemTable；必要时把恢复出来的 MemTable flush 成 L0 SSTable。
6. 创建新的 log 和 MemTable，开始接收新写入。

  

这个顺序把每类文件的责任划得很清楚：

```
CURRENT  : 指向当前 ManifestMANIFEST : 恢复当前有哪些 SSTable，以及各层 key range*.ldb    : 保存已经提交的有序数据*.log    : 重放尚未完全落成 SSTable 的最近写入LOCK     : 避免多个进程同时打开同一个 DB 目录LOG      : 人看的信息日志，不参与数据恢复语义
```

如果只看单个文件，很容易误会 LevelDB 的状态。例如目录里出现一个 `.ldb` 文件，不代表它一定属于当前版本；只有 Manifest 引用了它，它才是当前数据库状态的一部分。目录里有旧 log，也不代表都要重放；恢复逻辑会根据 Manifest 记录和文件编号判断哪些 log 仍然相关。

## 为什么说 LevelDB 是最小 LSM 样本

LevelDB 的价值不在功能复杂，而在主链路足够清楚。WAL 解决崩溃恢复，MemTable 解决最新写入可见，Immutable MemTable 把前台写入和后台 flush 分开，SSTable 把内存有序数据变成不可变磁盘文件，Manifest 把“哪些文件构成当前版本”变成可恢复的元数据日志。

这条链路已经包含 LSM 的主要取舍。写入不原地改磁盘，所以前台写入更像顺序追加；读请求要同时查内存和多层文件，所以读放大会出现；删除只是标记，所以空间回收要等 compaction；Manifest 让版本提交可恢复，但也要求每次文件增删都被严肃记录。

RocksDB、Pebble、TiKV 底层的 RocksDB、Kafka Streams 状态存储里的 RocksDB，都在这条主链路上加了更多工程能力。理解 LevelDB 的好处，是先把最小闭环看明白：一条写入从 WAL 到 MemTable，从 Immutable MemTable 到 SSTable，从 Manifest 到重启恢复，最后再被 compaction 推向更深层。这个闭环稳定了，后面的优化才有落点。

## 参考入口

* LevelDB 官方文档: Implementation notes
* LevelDB 官方文档: Log format
* LevelDB 官方文档: Table format
* LevelDB GitHub: google/leveldb

- END -