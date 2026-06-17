---
title: StarRocks FE 内存异常排查实战：从监控到根因定位（附真实案例）
author: StarRocks
date: StarRocksStarRocks
url: https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247498877&idx=1&sn=05738cf5a6a9a683ce3a2b4729deb526&chksm=e843c92c9af9a6280a084d4335fdbdaf2b4e91062af8d139940c14b51fe3e1895bc740e71bcc&mpshare=1&scene=24&srcid=0429o6osRCzuKpHkIF8UxvAN&sharer_shareinfo=c441bfd1edcf9b52f816835ae9880ef3&sharer_shareinfo_first=c441bfd1edcf9b52f816835ae9880ef3#rd
---

在生产环境中，FE 内存问题较为常见，尤其是 JVM Heap 相关问题，往往会影响集群稳定性与问题定位效率。本文基于实际案例，梳理 FE 内存问题的分析思路、排查方法与经验总结。

本文重点聚焦 FE JVM Heap 内存问题，覆盖监控配置、日志分析、工具使用与典型案例。文中案例均已脱敏处理，讨论版本以 **3.5 及之前版本**为主。堆外内存（Direct Memory / Native Memory）、工具崩溃、操作系统层面问题不作为重点展开，但会补充 2 个堆外内存相关实战案例。

全文分为三个部分：FE 内存组成与观测方式、生产环境典型案例解析、总结与规范说明。

# 一、FE 内存与观测

1

### ****FE 进程内存组成与配置指南****

由于 FE 是 Java 进程，其内存问题本质上主要体现为 JVM 内存问题。JVM 内存通常可分为 **Heap****内存**与 **Non-Heap 内存**两类。

* Heap 内存主要用于存储 Java 对象，承载 FE 核心业务逻辑中的各类对象，通常包括新生代与老年代两个区域。其中，新生代用于存放新创建、生命周期较短的对象；老年代用于存放存活时间较长的对象。
* Non-Heap 内存 则主要支撑 JVM 底层运行，包括 Metaspace、Direct Memory、线程栈以及本地方法接口相关内存。

**Heap****内存****是****FE****内存中最核心的部分，配置时需要重点关注****`-Xmx`****。**

* `-Xmx` 表示 Maximum Heap Size，用于控制 Heap 内存上限，通常在 `fe.conf` 的 `JAVA_OPTS` 参数中配置，决定 FE 最多可以使用多少堆内存。
* `-Xms` 表示 Initial Heap Size，用于控制 JVM 启动时初始申请的堆内存。一般情况下不建议手动设置，FE 会根据机器内存自动计算。若用户显式设置了 `-Xms`，需要在运行过程中重点关注 GC 情况：当 `-Xms` 设置过大时，JVM 启动后会直接申请较大的堆空间，G1 GC 可能不会过早、过于积极地回收对象；随着对象持续累积，后续可能出现连续 GC，甚至 Full GC。

堆外内存可通过 `MaxDirectMemorySize`、`MaxMetaspaceSize`、`-Xss` 等参数进行配置。通常情况下，这类参数仅在特殊业务场景下按需添加。

配置 FE 内存时，还需要为操作系统和其他进程预留足够空间；如果 FE 与其他组件混合部署，则应适当提高内存预留比例。对于独立部署的 FE 节点，可根据机器规格与业务负载，合理设置 Heap 上限，避免因配置过大影响系统稳定性。

独立部署 FE 节点时，可按机器内存比例设置 `-Xmx` 上限。由于 FE 进程运行时存在额外开销，实际物理内存占用（RSS）通常会高于 `-Xmx` 配置值，一般控制在 `-Xmx` 的 120% 以内属于正常范围。

Tablet 数量会直接影响 FE 内存配置。可通过以下命令查看集群整体 Tablet 总量，并结合集群规模匹配对应的内存配置：

```
SHOW PROC '/statistics'；
```

Tablet 数量越多，对 FE Heap 内存的要求越高，可参考以下区间进行配置。

**1.1 FE****Heap****内部占用与节点差异**

FE Heap 主要承载业务运行所需的各类对象。按照生命周期，可分为**运行时****内存**与**常驻内存**两类。

* 运行时内存是指 FE 在工作过程中临时产生、动态变化的对象，例如导入任务管理、事务状态管理、Query 执行上下文、统计信息及字典缓存等。这部分内存会随导入、查询、事务活跃度变化而波动。
* 常驻内存是指 FE 长期维护的元数据对象，例如 `TabletInvertedIndex`、`LocalMetastore` 等，主要用于管理库、表、分区、Tablet、Replica 等核心元数据。

运行时内存主要覆盖导入、事务、查询、统计信息与字典缓存等动态业务场景。

常驻内存主要用于维护 FE 长期持有的核心对象，保障集群元数据稳定管理。

2. **Leader****与****Follower****的****内存****差异**

FE 集群包含多种角色，其中 Leader 与 Follower 的内存压力存在明显差异。

整体来看，Leader 不仅提供查询服务，还需要承担元数据写入、事务协调、任务调度及 Checkpoint 等工作，因此内存压力通常显著高于 Follower。

* Follower 主要承担元数据同步和查询服务，整体内存压力相对较小。Leader 与 Follower 的内存差异，核心原因之一在于 Checkpoint 任务。
* Checkpoint 可以理解为 FE 定期生成元数据快照。日常元数据变更会先写入操作日志（Journal）；执行 Checkpoint 时，会先加载已有 Image 文件，再回放后续 Journal 到指定版本，并将当前完整的元数据状态写入新的 Image 文件。

通过这种机制，FE 重启或 Follower 同步时，无需从头回放全部日志。

由于 Checkpoint 过程中需要额外构造一份元数据对象用于日志回放和快照落盘，因此会产生额外的 Heap 内存开销。

在 3.4 版本之前，Checkpoint 任务由 Leader 节点独立承担，因此 Leader 内存占用通常明显高于 Follower。自 3.4 版本起，Checkpoint 调度机制完成优化，可将任务分配至 Follower 执行，Leader 与 Follower 的内存占用差距较 3.3 及更早版本已明显缩小。

2

### ****核心监控指标配置****

FE 内存监控主要包括 **Heap****监控**与 **GC****监控**两类。其中，Heap 监控可用于观察 FE 运行状态，也是事后定位内存问题的重要线索。

**2.1 Heap****监控指标**

通过 `heap_used`、`heap_max`、`heap_committed` 三项指标，可以清晰观察 FE Heap 的实际使用状态。

**2.2 GC****监控指标**

GC 监控可直观反映内存回收的触发频率与执行耗时，辅助运维人员判断 FE 内存回收是否正常。

官方提供了标准化监控模板。导入模板后，可在 **FE****JVM** 分类下查看全部内置指标，快速完成监控配置与数据观测。详情可参考：https://docs.starrocks.io/zh/docs/3.5/administration/management/monitoring/Monitor\_and\_Alert/

下图展示的是官方监控模板中的 JVM 监控面板，其中已包含前文提到的 Heap 与 GC 相关核心指标。

3

### ****Mem Profile 内存分析工具****

Mem Profile 基于开源工具 async-profiler，用于定时采集 FE 内存火焰图。

自 3.3.6+ 版本起，StarRocks 已支持自动输出 FE 内存火焰图，采集生成的压缩文件默认存储在 FE 日志目录：

`fe/log/proc_profile/mem-profile-$date.html.tar.gz`

该能力默认可用，无需额外配置。

在 3.3.6 之前版本，需通过脚本方式自行采样，并手动部署 `mem_profiler.sh`。

mem\_profile 配置参数详解

默认情况下，Mem Profile 执行单次 5 分钟采样，采集间隔为 5 分钟，采集文件默认保留 2 天，文件总容量控制在 2GB 左右。

在 **v3.4.5+ / v3.5+** 版本中，Mem Profile 采集机制完成优化：

* 移除 `collect_time_s` 参数；
* `proc_profile_collect_interval_s` 默认值由 300 调整为 120，即单次采样持续 2 分钟；
* 默认每 2 分钟采集一次，文件保留 1 天，总大小不超过 2GB。

需要注意的是，`mem_profile` 文件名中的时间表示采集开始时间，而不是压缩完成时间；文件生成时间则对应采样与压缩完成后的时间。排障时，应按照“采集窗口”对齐故障时间，即：

`[文件名时间，文件名时间 + collect_time_s]`，再定位对应的目标数据文件。

3.3.6 之前版本，可通过专用脚本完成 Mem Profile 工具的部署与配置。

```
注： 该脚本对fe 进程无影响，不影响服务正常使用，会对过期48小时的文件自动清理目的：在每台fe 的安装目录下新建脚本文件mem_profiler.sh将如下内容拷贝进文件#!/bin/bashmkdir -p mem_alloc_logcleanup_old_files() {    find mem_alloc_log -name "alloc-profile-*.html" -mmin +2880 -execrm -f {} \;}whiletruedo    cleanup_old_files    current_time=$(date +'%Y-%m-%d-%H-%M-%S')    file_name="mem_alloc_log/alloc-profile-${current_time}.html"    ./bin/profiler.sh -e alloc --alloc 2m -d 300 -f "$file_name" `cat bin/fe.pid`done#后台启动脚本chmod +x mem_profiler.shnohup ./mem_profiler.sh > mem_profiler.out 2>&1 &## 检查进程是否存在ps aux | grep mem_profiler.sh## 停止进程pkill -f mem_profiler.sh
```

4

### ****Jmap 排查流程与 Memory Tracker****

FE 内存问题排查通常可借助 **Jmap** 与 **Memory Tracker** 两类工具完成。

**4.1 Jmap**

Jmap 是 JDK 自带的内存分析工具，可用于查看 Java 进程中的对象分布、实例数量及内存占用情况。

Jmap 排查内存问题的常见流程如下：

第一步：确认 FE 进程的 PID

```
ps -ef | grep StarRocksFE | grep -v grep 或jps -l
```

第二步：通过 jmap 命令完成数据采样

```
jmap -histo > histo_1.log sleep60 jmap -histo > histo_2.log
```

通常可通过 `jmap -histo` 生成采样日志，间隔 60 秒连续采集两次。随后对比两份日志中的实例数量与字节数变化，判断对象占用是否持续增长。

第三步**：**必要时执行 Heap Dump，获取完整证据链

当 `jmap -histo` 无法准确定位问题时，可进一步执行 Heap Dump，并结合专业分析工具进行深度排查。

```
jmap -dump:format=b,file=/path/fe_.hprof
```

注意：谨慎使用 `live` 参数。

```
jmap -histo:livejmap -dump:live,format=b,file=...
```

带 `live` 参数的命令会触发 Full GC，并可能导致 FE 进程短暂停顿。生产环境中应谨慎使用，建议在业务低峰期执行。

**4.2 FE****Memory  Tracker  观测**

`MemoryUsageTracker` 是 FE 日志中的内存采样功能，v3.3.7 及之后版本支持。该功能会定期扫描已注册模块，估算各模块的内存占用与对象数量。

默认情况下，`MemoryUsageTracker` 每分钟采集一次，并将统计结果输出至 FE 日志。排查时，可通过 `MemoryUsageTracker` 关键字检索 FE 日志，查看相关采样信息与模块占用情况。

```
2025-06-06 16:37:50.667 INFO (MemoryUsageTracker|74) [...] Module TabletInvertedIndex - TabletInvertedIndex estimated 37.7MB of memory. Contains TabletMeta with 353459 object(s)... 2025-06-06 16:37:50.741 INFO (MemoryUsageTracker|74) [...] total tracked memory: 46.8MB, jvm: Process used: 18.6GB, heap used: 4.4GB, non heap used: 289.1MB, direct buffer used: 395.5MB
```

# 二、典型案例分析

1

### 案例一： Heap 配置不足导致 FE 集群不可用

**1.1 环境背景**

* 集群架构：3FE（3.3.14）
* Xmx 配置：8GB
* 机器内存：16GB
* 数据规模：Tablet 数量约 10W

**1.2 问题现象**

集群触发 FE 状态异常告警（Critical）。

该告警每 10 秒巡检一次 FE 状态，主要检查两项内容：

* FE 查询端口是否可正常执行 `SHOW FRONTENDS;`
* FE HTTP 端口 `/variable` 是否可访问，且返回内容非空

任一检查失败，该 FE 的失败计数加 1；当失败计数大于 3 时，触发 FE 状态异常告

* Leader 节点 `heap_used` 长期超过 90%；

* FE 日志中出现 `Java heap space` OOM 报错。

进一步检索 FE 日志中的 `MemoryUsageTracker` 信息后，确认日志中存在 OOM 记录。

**1.3 现场还原与问题分析**

* **T+0**: Heap 长期处于高运行状态, Leader 节点的 Heap 内存余量持续偏低。
* **T+1**:  SQL 查询执行过程中，内存占用继续上升，FE 内存压力进一步增大，并开始出现 Journal 提交失败。
* **T+2**:  FE 核心线程抛出 `Java heap space` OOM 异常。由于 FE 元数据依赖 BDBJE 维护，节点之间存在复制组与心跳机制。当 FE 发生 Full GC 或长时间进程停顿时，节点会暂时停止对外响应；若响应超时超过 BDBJE 心跳容忍时间，节点会被判定为不可用并标记为 `UNKNOWN`，原 Leader 随后退出集群。
* **T+3：**新 Leader 接管，集群恢复

**1.4 处理方式**

对 FE 节点进行内存扩容。建议 FE 节点机器内存至少配置为 16GB；本案例中，机器内存扩容至 36GB，并将 Heap 内存上限调整为 21GB。

**1.5 排查思路：**围绕高峰期 Heap 内存余量展开

* 确认 FE 状态异常的具体表现：进程处于运行状态但服务无法正常提供。生产环境建议配置 supervisor 组件管理心跳服务，保障服务异常后可自动拉起。
* 排查节点日志，优先检查 Leader 节点的日志文件，通过检索关键字判断 FE 是否发生重启或切主行为。

```
grep -E "shutdown hook|FE ShutDown|will exit|StarRocks FE starting|Got role:|notify new FE type transfer" fe.log
```

* 根据关键字判断切主失败原因，快速锁定问题源头

```
grep -E "transferToLeader|failed to init journal after transfer to leader|InsufficientReplicasException" fe.log
```

* 查看  Heap  用量历史

```
grep -E "OutOfMemoryError|Java heap space|MemoryUsageTracker.trackMemory\(\):108" fe.log
```

2

### ****案例二：复杂 SQL 并发引发 Follower FE OOM****

**2.1 环境背景**

* 集群架构：3 FE
* 机器内存：128GB
* `-Xmx`  配置：90GB
* 集群版本：2.5.22

**2.2 问题现象**

* Follower FE Hape 持续升高
* FE 处理能力变慢
* 同步作业大面积延迟
* 重启后 Heap 再次升高

通过脚本获取 `mem_profile` 信息后，可以看到内存主要集中分配在表达式树、类型树与 Planner Thrift 结构中。这些对象通常在 FE 端执行 SQL 解析、分析与规划过程中生成，是本次内存占用升高的主要来源。

**审计日志分析**：由于当前版本尚不支持 `queryFeMemory` 字段，无法精确定位高内存 SQL。

结合审计日志与请求分布来看，问题节点承接了大量业务请求，其中高频执行语句主要集中在两类：复杂 `INSERT ... SELECT` 导入语句，以及复杂 `LIMIT 0` 探测查询语句。

编写审计表查询语句，以分钟为单位聚合故障时段数据，确认两类语句在该窗口内集中执行。高请求量与大量复杂查询叠加，会持续抬高 FE 侧解析和规划阶段的内存占用。

**2.3 解决方案：扩容****内存**

扩容后的集群恢复稳定运行，Heap 内存可在高峰时段实现正常回落。

**2.4 排查思路**

* 查看 Heap 监控，锁定异常节点与故障时间窗口。
* 分析 `mem_profile` 文件，明确内存具体分配方向。
* 结合审计日志，定位引发高内存消耗的 SQL 语句。高版本集群支持 `queryFeMemory` 字段，可直接通过该字段筛选目标语句，提升定位效率与准确性。

3

### ****案例三 ：查询 Iceberg Catalog 外表导致 FE Full GC****

**3.1 环境背景**

* 集群架构：3 FE
* 集群版本：3.3.13
* 机器内存：32GB
* `-Xmx`配置：21GB

**3.2 问题现象**

* Follower  CPU 被打满
* 内存被打满
* FE 进程 Hang 住；
* GC 线程占满 CPU

在 CPU 高负载时段采集 `jstack` 信息后，确认资源占用主要来自 GC 线程，说明节点面临较大的内存回收压力。同时，`jstack` 中出现大量 `sr-iceberg-worker-pool` 热点线程，该类线程主要负责 Catalog 相关查询操作。

```
   at com.starrocks.connector.iceberg.IcebergMetadata.collectTableStatisticsAndCacheIcebergSplit    at com.starrocks.connector.iceberg.IcebergMetadata.triggerIcebergPlanFilesIfNeeded    at com.starrocks.connector.iceberg.IcebergMetadata.getPrunedPartitions    at com.starrocks.sql.optimizer.rule.transformation.ExternalScanPartitionPruneRule.transform    at com.starrocks.sql.StatementPlanner.plan    at com.starrocks.qe.StmtExecutor.execute
```

排查发现，这几个线程栈均为访问 Iceberg Catalog 的线程，说明问题可能与 Iceberg Catalog 相关。

默认情况下，Iceberg Catalog Plan 线程数与 FE CPU 核心数一致。此前已建议用户将 `iceberg_job_planning_thread_num` 降低至 4，但问题仍然存在，需要进一步分析线程栈。

Catalog  创建语句：

```
    catalog 配置如下：    CREATEEXTERNAL CATALOG prod_iceberg_catalog    PROPERTIES(    "aws.s3.access_key"  =  "xxx",    "aws.s3.secret_key"  =  "xxx",    "enable_iceberg_metadata_cache"  =  "false",    "aws.glue.use_instance_profile"  =  "false",    "iceberg.catalog.type"  =  "glue",    "type"  =  "iceberg",    "aws.glue.access_key"  =  "xx",    "iceberg_job_planning_thread_num"  =  "4" (默认fe cpu 核心数)    )
```

查看 fe.log 日志，锁定故障发生的时间窗口，发现对应时段内 memory tracker 记录的内存数据呈现持续升高趋势。

在审计日志中定位到 Iceberg 外表查询语句

```
selectcount(*) from prod_iceberg_catalog.dwd.dwd_test_iceberg ... 
```

通过 Profile 发现，本次查询命中了 **173,793 个 delete files**。

`delete files` 是 Iceberg 在更新、删除数据时生成的一类增量删除文件。这些删除信息不会立即合并回原有 `data file`，而是先独立存储；通常需要在后续 Compaction 等整理过程中，才会与 `data file` 完成合并。

在查询阶段，FE 需要对这些 `delete files` 进行匹配与索引构建，从而显著放大 Planning 阶段的内存开销。

**3.3 临时处理方案**

* 重启异常 Follower 节点；
* 同时禁用问题 SQL

**3.4 长期治理方案**

* 定期执行 Compaction，降低 Planning 阶段负担；
* 可将 `plan_mode` 调整为 `distributed`，降低 FE 单节点 Planning 压力；
* 如果版本较新（如 3.5.6+），可通过相关参数限制 Iceberg Cache 的内存使用。需要注意的是，这类缓存参数主要控制元数据缓存占用，并非 Planning 阶段的临时内存。

```
iceberg_data_file_cache_memory_usage_ratio=0.1 iceberg_delete_file_cache_memory_usage_ratio=0.1
```

**3.5 排查思路**

* 首先查看 Heap 监控与系统资源使用情况，确认是否存在明显 GC 压力。
* 结合 CPU 打满、服务 Hang 住等现象，及时抓取 `jstack` 信息。
* 通过 `jstack` 分析热点线程，结合日志定位具体 SQL，再通过 Profile 进一步确认问题来源。

4

### ****案例四 ：频繁建表、删表导致 FE 内存******升高**

**4.1 环境背景**

* 集群架构：3 FE
* 集群版本：3.1.13

**4.2 问题现象**

* Leader CPU 持续增高；
* Heap 持续增高；
* 重启后新 Leader  内存仍继续上涨。

使用 Arthas 工具观测堆内存后，确认当前内存占用处于较高水平。

进一步检索 FE 日志发现，在内存上涨阶段前，存在大量建表与删表操作，且删表操作均采用非 `FORCE` 模式执行。

大量 `DROP TABLE` 操作的 `force` 均为 `false`。这意味着表被删除后，元数据会先进入回收站，不会立即彻底清理。随着元数据持续进入回收站并不断积压，FE Heap 占用被逐步推高。

**4.3 恢复方法**

* 停止相关 DDL 作业；
* 重启 FE 节点，恢复集群服务；
* 长期来看，应降低 DDL 操作并发量，减少临时表元数据堆积；
* 如果存在高并发 DDL 验证场景，建议使用 `DROP TABLE FORCE` 或 `DROP PARTITION FORCE` 及时清理元数据。

5

### 案例五：MV 频繁刷新导致 Leader GC 退出切主

**5.1 环境背景**

* 集群架构：3 FE
* 集群版本：3.3.14

**5.2 问题现象**

* Leader 重启；
* Heap 持续走高。

通过查看 `MemoryUsageTracker` 日志，可以清晰观察到 Heap 内存连续上涨的趋势。

分析 GC 日志可见，节点触发了大量 Full GC。Full GC 执行过程中会显著影响进程响应效率，进而引发 BDB JE 复制心跳异常，最终导致 Leader 节点退出。

根据 `mem_profile` 数据发现，`java.util.concurrent.ThreadPoolExecutor.runWorker` 相关内存持续上涨，说明后台任务长时间运行并持续占用内存。这类后台任务通常包括用户提交任务以及 MV 刷新任务。

沿着这一方向继续排查 FE 日志，发现部分慢锁的 Owner 落在 `starrocks-taskrun-pool` 线程池。同时检查到 `task_runs_concurrency` 已被调整至 **16**，而默认值为 **4**，说明后台任务并发被明显放大。

结合任务类型判断，高并发后台任务大概率来自 MV 刷新。进一步统计 MV 刷新频率后发现，日志中存在大量 **5 秒级高频刷新**的 MV，且数量较多。

```
    grep -c 'interval step to 5, timeunit to SECOND' fe.log    grep -c 'interval step to 1, timeunit to MINUTE' fe.log    grep -c 'interval step to 2, timeunit to MINUTE' fe.log    grep -c 'interval step to 1, timeunit to HOUR' fe.log    5 SECOND: 1873    1 MINUTE: 1033    2 MINUTE: 407    1 HOUR: 2437
```

也就是说，MV 刷新任务本身触发频率就很高，导致 `task pool` 长时间处于较大压力之下。

同时，从 MV 刷新任务日志可以看到，`refresh` 实际会走 `INSERT OVERWRITE` 链路；而这些 MV 又涉及 Catalog 外部查询，因此在刷新过程中，还会叠加查询规划阶段访问外部元数据的开销。

高频 MV Refresh 与外表元数据开销叠加，最终导致 FE 内存占用持续升高。

**5.3 处理方案**

* 降低 MV 刷新频率。
* 其他优化措施

+ 下调 `task_runs_concurrency` 参数，降低后台任务并发度；
+ 将秒级刷新调整为分钟级刷新，缓解任务线程池压力；
+ 采用分区刷新模式；
+ 降低单次刷新的资源消耗；
+ 涉及 JDBC 外表等 Catalog 场景时，开启 Catalog cache 功能，减少元数据查询带来的内存开销。

**5.4 排查思路**

* 查看监控数据，确认 Leader 节点 Heap 内存持续处于高位；
* 查看 GC 日志，确认频繁 Full GC 且回收效率偏低的情况；
* 检索日志，重点关注 `TaskRun`、`MV Refresh`、慢锁等关键词；
* 检查 MV 的刷新频率，定位后台任务的异常状态。

6

### 案例六：InsertLoadJob 内存泄漏

**6.1 触发场景：**客户集群主要以导入任务为主。

**6.2 问题现象：**FE 内存异常升高，进程重启，且未正常完成 Leader 切换。

对 Leader 节点进程导出 `jmap` 进行排查：

```
jmap -histo <FEpid> > jmap.txt  # 该操作不会触发 Full GC
```

在导出文件中发现：

```
14: 521358154321968 com.starrocks.load.loadv2.InsertLoadJob
```

从导出结果可以看到，`InsertLoadJob` 实例数量较多。进一步分析发现，导入任务事务在 callback 引用环节未完成清理，导致 `InsertLoadJob` 对象出现内存泄漏。

可通过以下命令查看对象实例数量：

```
jmap -histo <FEpid> > jmap.txt
```

一般情况下，`InsertLoadJob` 实例数量超过万级即可判定存在泄漏风险；若超过 50 万，可能引发严重内存问题。

**6.3 问题版本：**

* 3.1.0 ~ 3.1.16
* 3.2.0 ~ 3.2.12
* 3.3.0 ~ 3.3.7

**6.4 修复版本:**

* 3.1.17+
* 3.2.13+
* 3.3.8+

**6.5 处理方式：**

升级至对应修复版本后，可彻底解决该对象泄漏导致的内存异常问题。

相关链接：

Issue: github.com/StarRocks/starrocks/issues/53810

PR: github.com/StarRocks/starrocks/pull/53809

7

### 案例七：Replica 内存泄漏

**7.1 版本范围：**

该问题主要影响 3.3.13 ～ 3.3.18 版本。如果发现 FE Heap 内存持续缓慢增长，可能与该问题有关。

**7.2 触发场景 ：**

* 大量 `DROP` 操作；
* 大量 `SWAP` 操作；
* 大量 `INSERT OVERWRITE` 操作。

**7.3 问题现象：**

通过导出 `jmap` 分析发现，`com.starrocks.catalog.Replica` 对象占用内存过多。

进一步排查确认，大量 `DROP`、`SWAP`、`INSERT OVERWRITE` 操作均可能触发该问题。

根因是 3.3.13 版本对 Tablet 删除路径进行优化后，引入了内存泄漏问题，并已在 3.3.18 版本修复。

* 3.3.13 相关优化 PR： [BugFix] Fix recycle bin missing to delete lake mv's expired partitions after mv refreshed (backport
* 3.3.18 修复 PR：https://github.com/StarRocks/starrocks/pull/61582

**7.4 处理方式：**

临时规避：重启 FE 节点

**7.5 根本解决：**

* 升级至修复版本（3.3.18、3.4.7、3.5.4）。

8

### ****Heap 问题排查思路****

8.1 排查路径

* 先看监控，明确异常节点与时间窗口：例如哪台 FE 出现异常、问题从什么时候开始。
* 判断内存上涨类型

+ 如果是突然上涨，优先查看 QPS、连接数、SQL 请求等监控指标，判断是否存在流量增长，并结合审计日志定位相关 SQL。 如果是缓慢上涨，优先排查内存泄漏或缓存累积，可借助 `mem_profile`、`jmap`、`jstack` 等工具，并结合日志分析对应操作。

* 根据问题类型做针对性处理

常用排查命令包括：通过 `jmap -histo` 查看对象分布，通过 `jstack -l` 抓取线程栈，通过 `mem_profile` 分析内存分配火焰图，并结合 `grep` 检索 OOM、Heap 等相关日志。

```
查看jmapjmap -histo <pid> | head -n 30获取线程栈jstack -l > jstack.log获取 mem_profilefe/log/proc_profile/x检索 OOM 日志等grep -E "OutOfMemoryError|Java heap space" fe.log
```

需要注意的是，工具本身不是目的。它们的价值在于帮助排查过程逐步收敛，最终定位到具体节点、时间窗口、对象类型、线程行为或业务操作。

9

### 4.0 新特性： FE 内存实时估算 API

在 4.0 版本中，StarRocks 新增 FE 内存统计接口，可对当前内存使用情况进行采样估算，统计范围覆盖 FE 内部具体模块。发生内存异常时，可通过该接口获取实时内存占用信息，辅助定位问题。

**9.1 核心能力**

* API 端点：`http://fe_ip:8030/api/memory_usage`，仅允许 `127.0.0.1` 访问；
* 基于 JOL（Java Object Layout）的对象图遍历与采样估算，对当前内存使用情况进行分析；
* 以 JSON 格式输出各模块的内存占用数据。

接口设置了安全访问限制，仅支持本机与本地 FE 调用。由于接口执行过程中会遍历 FE 内部对象，存在一定性能损耗，适合在内存异常时单次调用，不建议频繁使用。

# 特别提示与总结

1

### **async-profiler 可能引发****JVM****Crash**

触发问题前文提到的 `mem_profile` 基于开源工具 **async-profiler** 实现，用于采集内存火焰图。需要注意的是，async-profiler 同时也支持 CPU 采集；在少数场景下，CPU 采集可能引发 JVM Crash，导致 FE 进程直接退出。

1.1 典型现象

* FE 进程崩溃或自动重启；
* 常规 FE 日志中没有明显异常信息；
* FE 日志目录下生成 `hs_err_pid*.log` 文件。

如果 FE 进程直接退出，而常规日志中没有明显报错，同时在日志目录下生成 `hs_err_pid*` 文件，通常可判定为 **JVM****Crash**。该文件会记录完整的崩溃堆栈信息。若堆栈中出现指向 async-profiler 相关模块的异常调用链，则大概率说明本次 Crash 与采集工具运行异常有关。

2.2 处理方式

* 临时关闭 CPU 采集功能：`ADMIN SET FRONTEND CONFIG ("proc_profile_cpu_enable" = "false");`
* 在 `fe.conf` 中关闭配置（永久生效）： `proc_profile_cpu_enable=false`

自 **3.3.17** 版本起，CPU 采集功能默认关闭；更早版本默认开启。

2

### FE 堆外内存异常

2.1 问题现象

* FE 所在机器整体内存持续增长，使用率超过 95%；
* FE Heap 未明显上涨；
* 正常情况下，FE 进程总内存占用通常在 `-Xmx` 的约 120% 范围内趋于稳定，但当前场景持续超出该范围。

2.2 典型表现：

机器整体内存持续攀升，而 FE Heap 指标变化并不明显，说明额外内存占用并非来自 JVM Heap，而是来自堆外内存或底层内存分配机制。

JVM 在运行过程中，会通过底层 **glibc**（GNU C Library）申请部分内存。自 glibc 2.10 起，引入了**Arena**多线程内存分配机制：系统会为不同线程维护独立内存区域。线程释放资源后，这些内存区域未必立即归还操作系统，进而形成内存持续占用的现象。

👉参考文档 https://www.easyice.cn/archives/341

2.3 处理方式

* 配置 MALLOC\_ARENA\_MAX `MALLOC_ARENA_MAX = 1` ，限制 glibc Arena 创建数量
* 配置步骤

+ 在 fe.conf 中以大写格式添加对应配置项
+ 对 FE 节点执行滚动重启

**2.4 注意事项**

该调整可能对部分业务性能产生影响，建议重点观察 **P99 延迟** 指标。若出现 P99 升高，可结合 Leader 切换后继续观察整体性能变化。

3

### 事前 Chekclist

集群运维需要在日常阶段完成前置部署与配置。

* 完整配置监控体系与日志服务，确保 Heap 监控与 GC 日志持续采集，为事后定位与现场还原提供依据。
* 提前搭建日志与证据链体系。建议前置部署审计日志插件，便于后续问题追溯；出现 OOM 异常时，可通过专用配置在进程宕机时自动生成 Dump 文件，留存关键分析数据。
* 确保 `jmap`、`jstack`、`mem_profile` 等分析工具可用，满足突发问题排查需求。
* 结合集群规模开展容量规划，持续关注 Tablet 数量与元数据增长趋势，并根据业务规模及时推进 FE 节点扩容。

3

### 应急响应流程

故障发生时，首要任务是保护现场，为后续排查留存关键证据。

如果 FE 进程出现 Hang 住、无响应等情况，应第一时间采集 `jstack` 信息，用于捕捉异常堆栈、分析问题根源。一旦直接重启 FE 节点，堆栈与线程相关信息会丢失，影响故障现场还原。

如果怀疑 Heap 过高或 OOM，可通过 `jmap -histo` 获取内存对象快照，留存对象分布与占用数据。需要注意的是，添加 `live` 参数会触发 Full GC，可能加重节点压力，生产环境需谨慎使用，建议在业务低峰期执行。

完成 `jstack`、`jmap` 等证据采集后，再进行节点重启、扩容等处置操作，并基于留存证据判断问题类型，开展针对性处理。

若判定为对象泄漏类问题，需通过版本升级完成修复；若发现 QPS、连接数异常，则需排查业务请求合理性，必要时执行限流或调整业务并发，缓解节点资源压力。

如想进一步学习与交流，欢迎添加小助手，加入 StarRocks 社区群，与更多开发者共同探索交流。

**关于 StarRocks**

StarRocks 是隶属于 Linux Foundation 的开源 Lakehouse 引擎 ，采用 Apache License v2.0 许可证。StarRocks 全球社区蓬勃发展，聚集数万活跃用户，GitHub 星标数已突破 11,000，贡献者超过 500 人，并吸引数十家行业领先企业共建开源生态。

StarRocks Lakehouse 架构让企业能基于一份数据，满足 BI 报表、Ad-hoc 查询、Customer-facing 分析等不同场景的数据分析需求，实现 "One Data，All Analytics" 的业务价值。StarRocks 已被全球超过 600 家市值 70 亿元人民币以上的顶尖企业选择，包括中国民生银行、沃尔玛、携程、腾讯、美的、理想汽车、Pinterest、Shopee 等，覆盖金融、零售、在线旅游、游戏、制造等领域。

**行业优秀实践案例**

**泛金融：**[中国民生银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)**[｜](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)**[平安银行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493044&idx=1&sn=580ea0a521f9c9e4099356022536a585&chksm=e9f29a90de851386c04da4c3810b3c1ce0fce8e7f128fcbee0fb1c1a480a17aa5582325f20ee&scene=21#wechat_redirect)｜[中信银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[四川银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[南京银行](https://mp.weixin.qq.com/s?__biz=MzkxOTQyNzU1NQ==&mid=2247484420&idx=1&sn=54b813186be038ad1db9db7957eb5a19&scene=21#wechat_redirect)｜[宁波银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中原银行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487331&idx=1&sn=e6cdcdf2c46762135b6d95bf58e47664&chksm=e9f17047de86f9519e4d58f47745fe64788f2a5b3117133fc4b39db1a50c50d163a467f20950&scene=21#wechat_redirect)｜[中信建投｜](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488978&idx=1&sn=68da6c95be3337048c5900ce70d56c5e&scene=21#wechat_redirect)[苏商银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247487346&idx=2&sn=263c5029751e0ec8ed81df1760deb1d6&scene=21#wechat_redirect)｜[微众银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[杭银消费金融](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[马上消费金融](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中信建投](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[申万宏源](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492791&idx=1&sn=e4eba3e3d3de2fc4e59148a2e52f7201&chksm=e9f29b93de851285bf00f6e94a0f096b24d9a494b0013d43fb42d006eabea27a615e605b5a1b&scene=21#wechat_redirect)[｜](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247485524&idx=1&sn=4a5d0f877d39ed674cbc0b8ba03847c6&scene=21#wechat_redirect)[西南证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中泰证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[国泰君安证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[广发证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[国投证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中欧财富](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247485524&idx=1&sn=4a5d0f877d39ed674cbc0b8ba03847c6&scene=21#wechat_redirect)｜[创金合信基金](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[泰康资产](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[人保财险](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[随行付](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247498100&idx=1&sn=966cd218640936c6830de7f148a4d1e4&scene=21#wechat_redirect)

**互联网：**[微信｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484928&idx=1&sn=87d2572bba55afef5abd5f7b8571a443&chksm=e9f17924de86f032c3dca0fdc69b9b8ab8b873bc9a5bc4fdb284c06c8539da0e24e6647ded93&scene=21#wechat_redirect)[小红书｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484997&idx=1&sn=8644dacbcf467f6946add574b738e5d9&chksm=e9f17961de86f0776f5cbb201601010374c7e6c81b0a3250d93a32a997edbf694f4d471185eb&scene=21#wechat_redirect)[淘宝闪购](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247498373&idx=1&sn=b3f4cd88fd5539514afb0510760cf35a&scene=21#wechat_redirect)｜[滴滴](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490839&idx=1&sn=a007c5207f15129cd8c91da15cc30093&chksm=e9f16233de86eb25f585ef9b911e54ae6dae9547bd240a9554dd52ba449a74289a13d3df99fa&scene=21#wechat_redirect)｜[B站](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492334&idx=1&sn=dd0de6197bc6f3f67ffb0704b72ae02f&scene=21#wechat_redirect)｜[携程](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491637&idx=1&sn=daec09546a550346b65414567dc8db7b&chksm=e9f29f11de851607eb14a9e006803d8f52951d367f19a2477d5abee92f309c8c8145655a959c&scene=21#wechat_redirect)｜[同程旅行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490253&idx=1&sn=93f2a0369f9a9bff2215031efd197530&chksm=e9f165e9de86ecffe737cfebf729668df7efe09e6f9c4c976dc17668dc8042ef7895aef74ea3&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488632&idx=1&sn=0f76b5ce855b3070a138cb58e5fb8112&chksm=e9f16b5cde86e24a853e743df776e25d2358e710442cb419d32c3dd504d8e328e06b5cb473fd&scene=21#wechat_redirect)[芒果TV](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490596&idx=1&sn=9de0a6ce3608875f17287a197b7847b1&chksm=e9f16300de86ea16b83b7f5d8a29b20c137b1bc641d4d190c4d73d717a56d8c104a1f585f4d7&scene=21#wechat_redirect)｜[得物](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487306&idx=1&sn=46b784eb29c1a4d645946a017deb7f8a&chksm=e9f1706ede86f97861ce9ed2952335312704fe2cc9adc35b3cfa5709f2abdeea9a2779ff18d6&scene=21#wechat_redirect)｜[贝壳](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490822&idx=1&sn=a8bc0b31aa6568f57e919632e3f15877&chksm=e9f16222de86eb34d2a4eabd28c36ff2b29f3d9ad72855e8a9064d3b84b24a1e13970a80d3e3&scene=21#wechat_redirect)｜[汽车之家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484870&idx=1&sn=74ab3350c6b7e86376c009b3ee6b5b98&chksm=e9f17ae2de86f3f45cc32c4ae153b61c57cdf6e66dd7b10ae9fb8ec689994598130257c43936&scene=21#wechat_redirect)｜[哈啰](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247498659&idx=1&sn=fe5443fafd85ba0abddc188065582562&scene=21#wechat_redirect)｜[腾讯音乐](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493768&idx=1&sn=5076f547377aeb56ba351b03fae0730f&chksm=e9f297acde851eba21783c944fe7da28ccafe8eb86063e2100672f7982afd449441f47e73a3e&scene=21#wechat_redirect)｜[饿了么](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247494190&idx=1&sn=89f15ad79ef3aa83db4a6f9cb9ee8bce&chksm=e9f2950ade851c1ce73cb480fcab5f714564ea21f69ef75d565f02750d7760ee3128bc2319da&scene=21#wechat_redirect)｜[七猫](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247494193&idx=1&sn=61b083ca4befbb0fe951131a5efd4687&chksm=e9f29515de851c0383357500c59f07cda799d6a51ea04721326a9af54db0642068dbb6c68645&scene=21#wechat_redirect)｜[金山办公](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493868&idx=1&sn=ad2326665f7aa0b42358af6ceab0fbe9&chksm=e9f297c8de851ede838a7db7a6bbd78d735711a420e26092878f7c334da50fcc0540c648775b&scene=21#wechat_redirect)｜[Pinterest](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493896&idx=1&sn=673579fbfcaf50fe5577b2ec91c5639a&chksm=e9f2962cde851f3a6d60b30eac4c3cabd033307c0456e3cac048a934891a46c83e76dbc75eb9&scene=21#wechat_redirect)｜[欢聚集团](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486170&idx=1&sn=bd5de60aa15f03a6972467c45c58bb9b&chksm=e9f175fede86fce8c2495df1d7f6f3096b70ec8ac69ee94e12f96f3e15c256a85a94c3e3c5f7&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490839&idx=1&sn=a007c5207f15129cd8c91da15cc30093&chksm=e9f16233de86eb25f585ef9b911e54ae6dae9547bd240a9554dd52ba449a74289a13d3df99fa&scene=21#wechat_redirect)[美团餐饮](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488130&idx=1&sn=53c32ae5448505db505872e0d8e8d7e1&chksm=e9f16da6de86e4b03a1295902ac27ff3fe9a554dd76383b6d24c155fb11778bc4d4afa7d9494&scene=21#wechat_redirect)[｜](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492334&idx=1&sn=dd0de6197bc6f3f67ffb0704b72ae02f&scene=21#wechat_redirect)[58同城](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487038&idx=1&sn=583cf978f57802edd653034d83ac30af&chksm=e9f1711ade86f80cc37de844768228ba7e6a6abc06dd8b4867303334d31334e7b23273d12c39&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490253&idx=1&sn=93f2a0369f9a9bff2215031efd197530&chksm=e9f165e9de86ecffe737cfebf729668df7efe09e6f9c4c976dc17668dc8042ef7895aef74ea3&scene=21#wechat_redirect)[网易邮箱](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488632&idx=1&sn=0f76b5ce855b3070a138cb58e5fb8112&chksm=e9f16b5cde86e24a853e743df776e25d2358e710442cb419d32c3dd504d8e328e06b5cb473fd&scene=21#wechat_redirect)｜[360](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485899&idx=1&sn=c8b13a6a6f9d0d59de1721a36f52c4cd&chksm=e9f176efde86fff974e81dfe2fb72a834528141a9b0f4a99f647b11c1a2e9001822b82ddbd1e&scene=21#wechat_redirect)｜[腾讯游戏](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486696&idx=1&sn=d4bfaf2c2fb84a890922a0dfd4879522&chksm=e9f173ccde86fadaa1313f3e1c82d71f44b3e244113c712a272d3c85543765752431f8896ae6&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486696&idx=1&sn=d4bfaf2c2fb84a890922a0dfd4879522&chksm=e9f173ccde86fadaa1313f3e1c82d71f44b3e244113c712a272d3c85543765752431f8896ae6&scene=21#wechat_redirect)[波克城市](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486150&idx=1&sn=bc272d1174edb74233c584c1b7f970da&chksm=e9f175e2de86fcf4b78f09ce88ead3dc591e11edd11a3efafe74441591928d37c213b9871e7d&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486170&idx=1&sn=bd5de60aa15f03a6972467c45c58bb9b&chksm=e9f175fede86fce8c2495df1d7f6f3096b70ec8ac69ee94e12f96f3e15c256a85a94c3e3c5f7&scene=21#wechat_redirect)[37手游](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486749&idx=1&sn=610a64862a91df789faaa2469f2c8116&chksm=e9f17239de86fb2f78abd9f391d3c780f371213c93150d8617bb1c9a1845b837293f203f7d41&scene=21#wechat_redirect)｜[游族网络｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487470&idx=1&sn=e8844ab1037069989e3dc95f5e9092e8&chksm=e9f170cade86f9dcd62d810c6f7132002c4cf7329dbf20f232db82ca6cca955ffd934be04e0c&scene=21#wechat_redirect)[喜马拉雅｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247494895&idx=1&sn=f09d7e5262feb121d12b486d8bd88d29&chksm=e9f293cbde851add02bc6d438118e530d7b569b5d3b788d1abe9237edcb76542f9a62c749b58&scene=21#wechat_redirect)[Shopee](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247495088&idx=1&sn=a42354398d35329e62241705d3e79eec&chksm=e9f29294de851b82d2b55a7c2cf8d5f75a37f1025c6a1f1b07e9fdbddddc865ea55560dffafa&scene=21#wechat_redirect)｜[Demandbase](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497162&idx=1&sn=714a27698ac91c13cd942f06aa6b25d8&scene=21#wechat_redirect)｜[爱奇艺](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497211&idx=1&sn=ea35a147566d6853b9383022340c4022&scene=21#wechat_redirect)｜[阿里集团](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497320&idx=1&sn=dda2e5922343822003105e28044e12b1&scene=21#wechat_redirect)｜[Naver](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497228&idx=1&sn=d380139bc957d8420f2807ef583fc598&scene=21#wechat_redirect)｜[首汽约车](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497432&idx=1&sn=f0ff73acc5820681e20ecf028bab5694&scene=21#wechat_redirect) ｜[Cisco](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247498587&idx=1&sn=bb3ca2af1954e97c9b384d2558f35bad&scene=21#wechat_redirect)

**新经济：**[蔚来汽车｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491009&idx=1&sn=d95f01aea67e68b617cb891be823dd9a&chksm=e9f162e5de86ebf310ec56cbe8c950a69ef5a311b8213bb3adaf843069f9f315a5bede006cdd&scene=21#wechat_redirect)[理想汽车｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485820&idx=1&sn=7aa47ec35feec64e49ea462fe7506864&chksm=e9f17658de86ff4e1bd61f12d70ee8b478cc0ecd8fa7860df72ce2b5dc4b8ac05c77235809d6&scene=21#wechat_redirect)[吉利汽车](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247485625&idx=1&sn=20dabf0908da3aa394f6b2553c363141&scene=21#wechat_redirect)｜[顺丰｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484829&idx=1&sn=344fb330aac7d722a57a9591a82ca019&chksm=e9f17ab9de86f3af2e6d995b9ff8493c2a4d8e992d30e205904e3bd0ba88075061ea0158cb82&scene=21#wechat_redirect)[京东物流｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486458&idx=1&sn=88058c480d5f163155bb885e3d1fbc2a&chksm=e9f174dede86fdc8defec71ca637bc2a5a9c87dd245c6bf9c51e11f2811fcdbcd507776ccd71&scene=21#wechat_redirect)[跨越速运](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=2&sn=7912601d564276718811d53c60116a69&chksm=e9f16e7dde86e76ba4a28a99faf69c33a0553e4d383d4e083c3e0387e1e6d088fd13094c055b&scene=21#wechat_redirect)｜[沃尔玛](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[屈臣氏](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[麦当劳](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[大润发｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488098&idx=1&sn=cae2b42fd8fdb77ed4e67fc83173291f&chksm=e9f16d46de86e450859f842b471de1ee4e57118901f6f1f7f2c545fefdbdfbe355c9dc3e7965&scene=21#wechat_redirect)[华润集团｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487047&idx=1&sn=74201f01931823c9cdc6abc71d3dba68&chksm=e9f17163de86f875439549071e7c79d3c2762cf245b69367be7024ed9d04380816da4348b38d&scene=21#wechat_redirect)[TCL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=1&sn=cb78e47e420191b2345aff1366709384&chksm=e9f16e7dde86e76b77ecfd0d72d164313e18ef2fad887c3e41d30693bfd5e79629018be53a6a&scene=21#wechat_redirect) ｜[万物新生](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490969&idx=1&sn=22a377677f403c37579b2309686d559b&chksm=e9f162bdde86ebab62ce4bd3905d63870947f5b42076fc4d3d1c60a0303e2565a01cb5e2b644&scene=21#wechat_redirect)｜[百草味](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487394&idx=2&sn=242d444b7c02721b503de9383329edcd&chksm=e9f17086de86f990d2e18132c2b9beb3fa3984f5accb03cb34e0d479e8575666426c5508913c&scene=21#wechat_redirect)｜[多点 DMALL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484971&idx=1&sn=8f7e7ec424335b81fb34f1da85efbf8f&chksm=e9f1790fde86f019016e67a8a820f866bcbffe484414fa93b403605f957874bb7450672bfc3d&scene=21#wechat_redirect)｜[酷开科技](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486836&idx=1&sn=399086d79a513d311083c30ec7746589&scene=21#wechat_redirect)[｜vivo](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497338&idx=1&sn=d34ba1738661b7200efbe6fe9950036f&scene=21#wechat_redirect)｜[聚水潭](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492598&idx=1&sn=0d25b7c62770e39a0961acd9501e4ad5&chksm=e9f29cd2de8515c400f7c6076ef91d6f13121d6f0dfd0f2743fb2afc16d07fc8e61d440a330a&scene=21#wechat_redirect)｜[泸州老窖](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中免集团](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[蓝月亮](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[立白](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[美的](https://mp.weixin.qq.com/s?__biz=MjM5NzIzODg0MQ==&mid=2656464503&idx=2&sn=765854a37cc14f3032699266110c51b0&scene=21#wechat_redirect)｜[伊利](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[公牛](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[碧桂园](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497443&idx=1&sn=32057c373f4bc6800f76fe77d5d25be0&scene=21#wechat_redirect)