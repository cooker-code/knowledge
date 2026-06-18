> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkDorisConnector写入机制|FlinkDorisConnector写入机制]]
---
title: Flink 写Doris原理解析和性能调优
author: data之道
date: -->-->
url: https://mp.weixin.qq.com/s?__biz=MzI0MzYzNTk5NQ==&mid=2247484866&idx=1&sn=708b24df3d02e3f4bc25b5cb6f9bc31a&chksm=e8edde0c2697aae9b04d78ed2c7dada49d5e4ef65b225a6eec305e697aca0b075bc3a5f64c5f&mpshare=1&scene=24&srcid=0430Cp1RrBiCZHMBbwmaquI0&sharer_shareinfo=9c21ae86a3866a39e4c9f59bbfb5b1ea&sharer_shareinfo_first=9c21ae86a3866a39e4c9f59bbfb5b1ea#rd
---

## 背景

在流量数仓场景中，Flink作为流式计算框架，将流量数据实时写入到 Doris 数据仓库，用于后续的数仓开发和行为分析查询。基于 Doris 原生 Stream Load 协议的写入方式，flink每天处理26亿+条流量数据，在实际生成环境中中，遇到了性能问题，本文将详细解析 Flink 以 Stream Load 写入 Doris 的核心原理，包括数据缓存、Checkpoint 协同、Doris 事务提交等关键流程，并针对内存、并发、批次等核心维度，提供调优策略，解决生产中的写入瓶颈、数据一致性等问题。

## 一、Flink 基于 Stream Load 写入 Doris 的核心原理

Flink 与 Doris 的集成，本质是通过 Stream Load 协议（Doris 提供的高吞吐 HTTP 写入接口），结合 Flink 的 Checkpoint 机制与 Doris 的两阶段提交（2PC），实现数据的高效、可靠写入。整个流程围绕“数据缓存→批次发送→事务预提交→Checkpoint 确认→事务提交”展开，核心目标是兼顾高吞吐与 Exactly-Once 数据一致性。

## 1.1 核心技术底座：Stream Load 协议

Stream Load 是 Doris 提供的原生批量写入接口，基于 HTTP 协议实现，也是 Flink 写入 Doris 的底层通信通道，其核心优势在于分布式并行写入、事务化提交，无需依赖 JDBC 等中间层，大幅降低写入开销。

Stream Load 的基础流程如下：

1. Flink Sink 端通过 HTTP 协议（默认访问 Doris FE 的 8030 端口），向 Doris 提交写入请求；
2. Doris FE 接收请求后，根据表的分区、分桶规则，路由并指定一个 BE 节点作为 Coordinator（协调者）；
3. Coordinator BE 接收 Flink 发送的批量数据，按分桶规则分片后，分发到对应的数据 BE 节点；
4. 各 BE 节点将数据先写入内存 MemTable，待 MemTable 满后异步落盘，形成 Rowset 文件，最终完成数据持久化；
5. 整个过程通过事务管理，确保一批数据要么全部成功，要么全部失败，同时支持 Label 幂等性，避免重复写入。

注意：Doris 版本需在 0.14.0 及以上，且需开启配置 `enable_http_server_v2 = true`，才能正常支持 Flink 基于 Stream Load 的写入。

## 1.2 关键流程解析：数据缓存、Checkpoint 与事务提交

Flink 写入 Doris 的核心逻辑，是将 Flink 的 Checkpoint 机制与 Doris 的 2PC 事务深度绑定，既保证数据不丢不重，又实现高吞吐写入。具体可分为三个核心阶段：数据缓存阶段、Checkpoint 协同阶段、Doris 事务提交阶段。

### 1.2.1 数据缓存阶段：微批攒批提升吞吐

Flink Sink 端并不会将数据逐条写入 Doris（否则会导致网络开销激增、Doris 压力过大），而是通过“内存缓存+微批攒批”的方式，批量发送数据，这也是提升写入吞吐的关键。

核心逻辑如下：

1. Flink 计算后的数据流进入 Doris Sink 后，首先存入 Sink 端的内存缓冲区（由 TaskManager 内存分配支撑）；
2. 缓冲区的数据会持续攒集，直到满足预设的“触发条件”之一，才会批量发送到 Doris：

1. 条数触发：缓冲区数据量达到 `batch.size`（默认 10w 条，Flink 1.13 版本默认 100 条）；
2. 时间触发：缓冲区数据攒集时间达到 `batch.interval`（默认 1s）；
3. 大小触发：缓冲区数据字节数达到 `batch.byte.size`（默认 100MB）。

3. 未满足触发条件的数据，会一直缓存在 Flink TaskManager 内存中，直到触发批量发送或 Checkpoint 强制刷新。

此阶段的核心目的是“减少写入请求次数”，通过批量传输降低网络往返开销，同时减轻 Doris BE 节点的写入压力。

一句话总结：缓存区里的数据量越多、Flink TaskManager缓存压力就越大；同时在一个批次里写入到doris的数据量就越多、带宽压力、doris压力都会越大，但是写入的批次数就少、doris事务少。

### 1.2.2 Checkpoint 协同阶段：控制事务触发时机

Flink 的 Checkpoint 机制是实现 Exactly-Once 语义的核心，在 Flink 写入 Doris 的流程中，Checkpoint 不再只是“保存作业状态”，更承担着“事务触发、一致性保障”的关键作用，与 Doris 的 2PC 事务深度协同。

如前所述，Flink Sink 会按批次（batch.size/byte size/interval 触发）将数据发送到 Doris，但是此时执行 **PreCommit（预提交）**，也就是 Doris 仅写入数据但不 Commit，数据不可见，且每个批次对应一个预提交事务，而且无论本次 Checkpoint 期间 Flink 发送了多少个批次（可能是 1 个或多个），Doris 只会执行 1 次 Commit，实现 “一次 Checkpoint 对应一次 Doris 全局提交”，保证数据一致性。具体协同逻辑如下：

1. Checkpoint 触发时，Flink 会强制 flush 所有 Sink 端内存缓冲区的剩余数据（无论是否满足批次触发条件），确保缓冲区无残留数据；
2. Flink 将这批数据通过 Stream Load 发送到 Doris，同时触发 Doris 的“预提交（PreCommit）”操作，此时数据已写入 Doris BE 的 MemTable，但事务处于 PRECOMMIT 状态，数据对外不可见（查询不到）；
3. Flink 完成全作业的快照（包括 Kafka 消费 offset、Sink 端事务状态、Doris 事务 ID 等），确保所有算子的状态都已持久化；
4. 若 Checkpoint 完全成功，Flink 会向 Doris 发送“提交（Commit）”指令；若 Checkpoint 失败（如节点崩溃、网络异常），Flink 会向 Doris 发送“回滚（Abort）”指令。

一句话总结：Checkpoint 是 Doris 事务提交的“开关”——只有 Checkpoint 成功，Doris 的事务才会真正提交；Checkpoint 失败，所有预提交的数据都会被回滚，避免数据脏写。

### 1.2.3 Doris 事务提交阶段：保证数据一致性

Doris 接收 Flink 的写入请求后，通过 2PC 事务机制，配合 Stream Load 的 Label 幂等性，确保数据的一致性与可靠性，整个事务流程分为三个状态：

1. 预提交（PreCommit）：

   Flink 发送的批量数据到达 Doris 后，BE 节点写入 MemTable 并持久化到临时文件，事务状态标记为 PRECOMMIT，此时数据不可见，仅完成数据落地；
2. 提交（Commit）：

   收到 Flink 的 Commit 指令后（flink checkpoint触发commit），Doris 发布数据版本，将临时文件转为正式数据文件，事务状态标记为 VISIBLE，数据对外可见，完成最终写入；
3. 回滚（Abort）：

   收到 Flink 的 Abort 指令，或事务超时、节点故障时，Doris 清理临时文件与预提交数据，事务状态标记为 ABORTED，数据被丢弃。

其中，Label 幂等性是避免重复写入的关键：Flink 会为每个批次的写入生成唯一 Label（格式通常为`flink_ckp_{ckpId}_{subtaskId}`），Doris 会记录所有已成功提交的 Label，若收到重复 Label 的写入请求，会直接跳过，避免 Failover 重试或重复提交导致的数据重复。

###

###

### 1.2.4 完整数据流转总结

结合上述三个阶段，Flink 基于 Stream Load 写入 Doris 的完整数据流转如下：

数据源（Kafka/CDC/日志）→ Flink Source → Flink 转换算子 → Flink Doris Sink 内存缓冲区 → 满足批次条件/Checkpoint 触发 → Stream Load 发送到 Doris → Doris 预提交（不可见）→ Checkpoint 成功 → Doris 提交（可见）。

## 二、核心调优策略

生产环境中，Flink 写入 Doris 常遇到的问题的：写入超时、数据倾斜、Checkpoint 失败、OOM 重启、Doris 压力过大等。针对这些问题，结合原理中的关键流程，从“内存、并发、批次、Checkpoint、Doris 自身”五个核心维度，提供可直接落地的调优策略。

## 2.1 内存调优：避免 OOM，保障缓存与写入稳定性

内存是 Flink 写入 Doris 的核心支撑——缓冲区缓存数据、网络传输、Checkpoint 快照，都依赖 TaskManager 内存。内存配置不合理，会直接导致 OOM、Checkpoint 超时、写入延迟飙升。

### 2.1.1 Flink 端内存调优（核心配置）

重点调整 TaskManager 总内存、Flink 托管内存、网络内存，核心配置如下：

```
# TaskManager 总内存（根据集群资源调整，推荐 8G~16G）taskmanager.memory.process.size: 8g# Flink 托管内存（用于 Sink 缓冲区、RocksDB 状态、算子计算，占总内存 60%~70%）taskmanager.memory.flink.size: 6g# 网络内存比例（高吞吐场景需提高，避免网络缓冲区不足导致反压）taskmanager.memory.network.fraction: 0.2# 网络内存最小/最大值（防止网络内存不足）taskmanager.memory.network.min: 256mbtaskmanager.memory.network.max: 1gb# JVM 优化：启用 G1GC，避免频繁 Full GCtaskmanager.jvm.args: -XX:+UseG1GC -XX:MaxGCPauseMillis=500
```

补充说明：

* 若使用 RocksDB 作为状态后端（大状态场景），需额外为 RocksDB 分配内存，配置 `state.backend.rocksdb.memory.fixed-per-slot: 512mb`，避免 RocksDB 抢占缓冲区内存；
* Sink 缓冲区内存可通过 `sink.buffer-count`（缓存 buffer 个数）和 `sink.buffer-size`（单个 buffer 大小）调整，默认单个 buffer 为 1MB，个数为 3，高吞吐场景可适当增大。

###

### 2.1.2 避免 OOM 的关键注意事项

1. 控制批次大小：避免 `batch.size`或 `batch.byte.size`过大，导致缓冲区内存溢出；
2. 限制并发写入队列：通过 `sink.flush.queue-size`（默认 2）控制异步写入队列长度，避免队列堆积导致内存飙升；
3. 监控内存使用率：通过 Flink Web UI 监控 TaskManager 内存、GC 频率，若频繁 Full GC，需增大堆内存或优化批次配置。

##

## 2.2 并发调优：提升吞吐，避免数据倾斜

并发配置直接决定 Flink 写入 Doris 的吞吐上限，同时也影响 Doris BE 节点的负载均衡。核心是“匹配上下游并发，避免倾斜，充分利用集群资源”。

### 2.2.1 Flink 端并发配置

Flink 端的并发分为“全局并行度”和“Sink 算子并行度”，优先级为：算子级并行度 > 全局并行度 > 集群默认并行度，核心配置原则如下：

1. 全局并行度：建议与 Doris 表的分桶数、Kafka 数据源的分区数匹配（若为 CDC 数据源，需与 `server-id`范围数量匹配），避免并行度与分桶数不匹配导致的数据倾斜；
2. Sink 算子并行度：

1. 建议与全局并行度一致，或略低于全局并行度（避免 Sink 成为瓶颈）；
2. 单个 Doris BE 节点建议承载 3~5 个 Sink 并行度，多个 BE 节点可按比例增加并行度，避免单个 BE 节点负载过高。

3. 核心配置示例：

```
# 全局并行度（与 Kafka 分区数、Doris 分桶数匹配）parallelism.default: 6# TaskManager 插槽数（与并行度匹配，避免资源浪费）taskmanager.numberOfTaskSlots: 6
```

### 2.2.2 数据倾斜解决策略

数据倾斜是并发写入中的常见问题，表现为个别 Flink Subtask 延迟极高、个别 Doris BE 节点 CPU/磁盘打满，核心原因是热点 Key 集中到少数 BE 节点。解决策略如下：

1. Flink 层打散热点：

   对 KeyBy 后的热点 Key，通过“盐值打散”（如 `key + "_" + random.nextInt(并行度)`），将热点数据分散到多个 Subtask，再通过二次聚合去掉盐值，避免热点集中；
2. Doris 层优化分桶：

   合理设计分桶键，增加分桶数量（建议分桶数是 BE 节点数的 3~5 倍），确保数据均匀分布到各个 BE 节点；
3. 动态调整并行度：

   对倾斜的 Sink Subtask，单独调整其并行度，或启用 Flink 1.16+ 的自适应调度功能，自动平衡负载。

## 2.3 批次调优：平衡吞吐与延迟，减轻 Doris 压力

## 批次参数（`batch.size`、`batch.interval`、`batch.byte.size`）直接影响写入频率、Doris 事务数量，核心是“在吞吐与延迟之间找平衡”，避免过小批次导致 Doris 压力过大，或过大批次导致超时、OOM。

###

### 2.3.1 批次参数最优配置（生产实践）

结合 Checkpoint 间隔，批次参数需遵循“批次间隔 ≤ Checkpoint 间隔的 1/5 ~ 1/10”，确保一个 Checkpoint 内有多个批次，既避免小事务堆积，又避免大批次超时。具体配置如下：

```
# 批次条数（根据数据大小调整，建议 1w~5w 条）sink.batch.size: 20000# 批次间隔（建议 1s~3s，平衡延迟与吞吐）sink.batch.interval: 2s# 批次字节大小（建议 10MB~50MB，避免单批次过大）sink.batch.byte.size: 30MB# 启用批量写入模式（默认 true，关闭后不保证 Exactly-Once 语义）sink.enable.batch-mode: true# 批量写入队列大小（默认 2，高吞吐场景可调整为 3~5）sink.flush.queue-size: 3
```

### 2.3.2 不同场景的批次调整建议

* 高吞吐场景（如日志写入）：

  增大批次条数和字节大小，延长批次间隔（如 3s），减少写入请求次数，降低 Doris 压力；
* 低延迟场景（如实时监控）：

  减小批次间隔（如 1s），适当降低批次条数，确保数据快速到达 Doris，配合 Checkpoint 间隔（如 5s），实现端到端延迟 ≤ 10s；
* 大消息场景（单条消息 > 32KB）：

  降低批次条数，增大批次字节大小，避免单批次超时，同时调大 Doris Stream Load 的 `max_batch_size`。

##

## 2.4 Checkpoint 调优：保障事务一致性，避免超时

Checkpoint 是 Flink 写入 Doris 一致性的核心，Checkpoint 配置不合理，会导致事务堆积、数据不可见、写入超时等问题。核心调优方向是“保证 Checkpoint 稳定，避免频繁失败，控制事务数量”。

### 2.4.1 Checkpoint 核心配置（flink-conf.yaml）

```
# Checkpoint 间隔（建议 10s~30s，平衡延迟与稳定性）execution.checkpointing.interval: 10000# Checkpoint 超时时间（建议为间隔的 2~3 倍，避免误判失败）execution.checkpointing.timeout: 30000# 最大并发 Checkpoint 数量（建议 1，避免并行写导致 Doris 压力过大）execution.checkpointing.max-concurrent-checkpoints: 1# 启用非对齐 Checkpoint（Flink 1.15+，反压场景必备，减少 Barrier 对齐时间）execution.checkpointing.unaligned: true# 失败重试次数（默认 3 次，避免临时故障导致 Checkpoint 失败）sink.max-retries: 3
```

### 2.4.2 Checkpoint 优化关键注意事项

1. Checkpoint 间隔与批次间隔匹配：

   避免 Checkpoint 间隔过小（如 5s），导致批次频繁被强制 flush，产生大量小事务；也避免间隔过大（如 60s），导致数据延迟过高、事务堆积；
2. 非对齐 Checkpoint 启用：

   反压场景下（如 Doris 写入跟不上 Flink 计算），启用非对齐 Checkpoint，避免 Checkpoint 卡住；
3. 事务状态持久化：

   开启 `sink.use-cache = true`，异常时通过内存缓存恢复 Checkpoint 期间的数据，减少数据丢失风险；
4. 监控 Checkpoint 指标：

   通过 Flink Web UI 监控 Checkpoint 成功率、耗时，若耗时过长，需优化批次大小、内存配置或 Doris 性能。

## 2.5 Doris 端调优：提升写入承载能力

Flink 写入性能的上限，最终由 Doris 集群的承载能力决定。若 Doris 自身配置不足，即使 Flink 端优化到位，也会出现写入超时、压力过大等问题。核心调优方向是“提升 BE 写入能力、优化事务与合并机制”。

### 2.5.1 Doris BE 写入优化

1. 调整写入线程：

   增加 BE 节点的写入线程数（`stream_load_process_thread_num`），建议设置为 CPU 核数的 1~2 倍，提升并行写入能力；
2. 优化内存配置：

   增大 BE 节点的内存（`be_mem_limit`），分配足够的内存用于 MemTable 写入和数据缓存，避免频繁落盘导致的性能瓶颈；
3. 磁盘优化：

   使用 SSD 磁盘，提升数据落盘速度；避免单块磁盘负载过高，分散数据存储路径。

###

### 2.5.2 Doris 事务与合并优化

1. 控制事务数量：

   避免 Flink 小批次频繁写入，导致 Doris 事务堆积，可通过增大 Flink 批次间隔、调整 Checkpoint 间隔实现；
2. 优化 Compaction 机制：

   调整 Doris 的 Compaction 策略，增大 `base_compaction_num_threads`（基础合并线程数），加快小文件合并速度，避免合并跟不上写入速度导致的查询变慢；
3. 副本优化：

   若不需要高可用，可将表的副本数调整为 1（`table.create.properties.replication_num: 1`），减少副本同步开销，提升写入速度；高可用场景建议副本数为 3，确保数据安全；
4. Label 过期时间：

   设置 Doris Stream Load 的 Label 过期时间（`label_ttl_second`），默认 3600s，避免过期 Label 堆积，占用元数据资源。

###

### 2.5.3 Doris 表结构优化

1. 分桶与分区设计：

   按业务场景合理设计分桶键（避免热点 Key）和分区键（如按时间分区），确保数据均匀分布，提升并行写入和查询性能；
2. 模型选择：

   根据业务需求选择合适的 Doris 表模型——Unique Key 模型用于需要更新/删除的场景（如 CDC 同步），Aggregate 模型用于聚合统计场景，Duplicate 模型用于简单追加场景，不同模型的写入性能有所差异，需针对性优化；
3. 数据格式：

   Flink 写入时，优先使用 CSV 或 JSON 格式，指定合理的列分隔符（如 `sink.properties.column_separator = ","`），避免格式解析错误导致的写入失败；JSON 格式需配置 `sink.properties.format = "json"`和 `sink.properties.read_json_by_line = "true"`。

## 三、常见问题与解决方案

结合生产实践，常见问题、现象及解决方案：

| 常见问题 | 现象 | 解决方案 |
| --- | --- | --- |
| 写入超时 | Flink Sink 报 timeout、connect reset，Checkpoint 超时 | 1. 调大批次大小，减少写入请求；2. 调大 Flink 网络内存和写入超时时间；3. 优化 Doris BE 负载，增加 BE 节点；4. 启用非对齐 Checkpoint |
| 数据倾斜 | 个别 Subtask 延迟高，个别 Doris BE 负载打满 | 1. Flink 层盐值打散热点 Key；2. 优化 Doris 分桶设计；3. 调整 Sink 并行度 |
| 数据重复 | Doris 表中出现重复数据 | 1. 确保 Label 唯一性，启用幂等性；2. 检查 Checkpoint 配置，避免重复提交；3. 使用 Doris Unique Key 模型去重 |
| 数据不可见 | Flink 写入成功，但 Doris 查询不到数据 | 1. 检查 Checkpoint 是否成功，未成功则事务未提交；2. 排查 Doris 事务状态，清理悬停事务；3. 检查批次触发条件，确保数据已发送 |
| Flink TM OOM | TaskManager 重启，日志报 OutOfMemoryError | 1. 调小批次大小，减少缓冲区内存占用；2. 增大 TaskManager 总内存；3. 限制写入队列大小，避免堆积 |
| Doris Compaction 跟不上 | 写入变慢，查询延迟升高，BE 磁盘 IO 高 | 1. 增大 Compaction 线程数；2. 调整 Flink 批次配置，减少小文件数量；3. 清理过期数据，减轻合并压力 |

#

## 四、总结

# Flink  写 Doris 的核心，是“微批攒批提升吞吐 + Checkpoint 与 2PC 保障一致性”。调优的核心思路是“先保证稳定性，再追求性能”——先解决 OOM、Checkpoint 失败、数据不一致等问题，再通过内存、并发、批次等配置提升吞吐、降低延迟。