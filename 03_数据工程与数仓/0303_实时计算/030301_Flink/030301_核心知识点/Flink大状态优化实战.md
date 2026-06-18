# 大状态优化实战（100GB+）

## 来源
- [Flink 超大状态作业优化实战](../文章/done-Flink%20超大状态作业优化实战.md)
- [Flink 实践 _ 字节跳动使用 Flink State 的经验分享](../文章/done-Flink%20实践%20_%20字节跳动使用%20Flink%20State%20的经验分享.md)
- [腾讯面试：Flink100G大状态如何优化？有哪些参数可以调整？](../文章/done-腾讯面试：Flink100G大状态如何优化？有哪些参数可以调整？.md)

## 核心问题
状态规模达到 100GB+ 时，默认配置为何失效？应按什么顺序排查和调优？有哪些来自生产的实测数据？

## 超大状态的症状清单

出现以下 2 个以上症状，说明状态规模已超出默认配置承载范围：

- Checkpoint 耗时 > 5 分钟（或持续超时）
- 作业恢复时间 > 10 分钟
- TaskManager 频繁 GC 或 OOM
- RocksDB Compaction 积压，吞吐骤降
- HDFS/S3 Checkpoint 目录占用 > 500GB
- 水印推进缓慢，窗口延迟触发
- Managed 内存用满，触发 Spill

## 状态规模快速诊断

```bash
# 通过 Flink REST API 获取各子任务状态分布
# GET http://flink-jobmanager:8081/jobs/{job-id}/checkpoints/details/{checkpoint-id}
# 关注：state_size（总大小） 和 subtask_stats 中的分布差异
# 某分区 state_size 远大于其他 → 数据倾斜

# HDFS 命令行查看 Checkpoint 大小
hdfs dfs -du -h /flink/checkpoints/{job-id}/chk-{checkpoint-id}/
```

## 四维优化框架

### 1. 存储层：RocksDB 配置

```yaml
# 必须开启增量 Checkpoint（状态 > 10GB）
state.backend.incremental: true
state.backend.rocksdb.memory.managed: true
taskmanager.memory.managed.size: 8g

# Block Cache（托管内存的 60%）
state.backend.rocksdb.block.cache-size: 2048m  # 或由 managed 自动分配

# MemTable
state.backend.rocksdb.write-buffer-size: 256m    # 默认 64MB
state.backend.rocksdb.max-write-buffer-number: 3  # 默认 2
state.backend.rocksdb.memory.write-buffer-ratio: 0.5

# 多磁盘分散 IO
state.backend.rocksdb.localdir: /data/disk1/flink,/data/disk2/flink

# Compaction 线程（CPU核心数的50%）
# 通过 RocksDBOptionsFactory 设置 setMaxBackgroundJobs(8)
```

### 2. 一致性层：Checkpoint 策略

```java
CheckpointConfig cp = env.getCheckpointConfig();
cp.setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
cp.setCheckpointInterval(120_000);         // 2分钟（大状态不要太频繁）
cp.setCheckpointTimeout(600_000);          // 超时 10 分钟
cp.setMinPauseBetweenCheckpoints(60_000);  // 两次间隔 >= 1 分钟
cp.setMaxConcurrentCheckpoints(1);         // 禁止并发 Checkpoint
cp.enableUnalignedCheckpoints();           // 反压场景必开
```

```yaml
# 开启本地恢复（恢复时间从 30 分钟 → 5 分钟）
execution.checkpointing.local-backup.enabled: true
execution.checkpointing.local-backup.dirs: /data/flink-local-recovery
```

**Savepoint 策略：每天定期触发全量 Savepoint 作为稳定基线。**
增量 Checkpoint 依赖历史 SST 文件链，基线越旧，恢复链越长。

### 3. 计算层：并行度与数据倾斜

**并行度设计原则**：每并行实例处理状态 3-4GB（100GB 状态 → 并行度约 32）

```yaml
parallelism.default: 32
state.backend.max-parallelism: 1024    # KeyGroup 数量 = 并行度的32倍
```

**热点 Key 处理（两阶段聚合）：**

```java
// 第一阶段：局部聚合，Key 加随机桶散开
DataStream<Result> partial = stream
    .keyBy(event -> event.getKey() + "_" + (event.hashCode() % 16))
    .process(new LocalAggregateFunction());

// 第二阶段：去掉随机桶后缀，全局聚合
DataStream<Result> result = partial
    .keyBy(r -> r.getKey())   // 去掉随机桶后缀
    .process(new GlobalAggregateFunction());
```

**时间桶 Key 设计（防止状态无限积累）：**

```java
// 原始 Key（状态持续累积）
.keyBy(event -> event.getUserId())

// 优化 Key（每小时一个时间桶，结合 TTL 自动清理）
.keyBy(event -> {
    long hourBucket = event.getTimestamp() / 3_600_000;
    return event.getUserId() + "_" + hourBucket;
})
```

### 4. 资源层：内存与 JVM

参见 [Flink 内存模型](Flink内存模型.md) 知识点。

## 增量 Checkpoint 与全量快照（Savepoint）恢复速度对比

| 场景 | 增量 Checkpoint | 全量 Savepoint |
|---|---|---|
| 非扩缩容恢复 | **更快**：多线程拉 SST 文件，直接初始化 RocksDB | 较慢：单线程遍历所有 KV 写入本地 RocksDB |
| 扩缩容恢复 | **慢**：多组 RocksDB 合并，大量 Compaction，写放大严重 | **更快**：和非扩缩容相同逻辑 |

**结论：扩缩容时优先从 Savepoint 恢复，避免增量 CP 合并导致的写放大。**

## 其他优化技巧（字节跳动生产经验）

### 减少 Checkpoint DFS 压力

```yaml
# 小文件合并：小于此阈值的状态以 byte[] 形式放在 _metadata，不单独落 DFS
state.backend.fs.memory-threshold: 20480  # 20KB（根据作业状态大小调整）

# 注意：阈值设置过高（MB级别）会导致 _metadata 过大，增加 JobMaster GC 压力
```

```yaml
# Checkpoint 上传线程数（过大会打满 HDFS NameNode 的连接）
state.backend.rocksdb.checkpoint.transfer.thread.num: 4  # 不要超过10
```

### Union State 注意事项

**避免在高并行度作业中使用 Union State！**
Union State 恢复时，每个并行度都需要读取所有并行度的状态文件，复杂度为 O(N²)。1000 并行度作业中每个 Task 都需要读 1000 个文件。

### 应用层缓存

在 StateBackend 和 Operator 之间构建应用层缓存（Java 对象），对热点数据收益明显。
注意：
- 控制缓存阈值，防止 GC 压力过大
- 注意 TTL 逻辑，防止脏读

## 生产案例：用户画像实时更新（400GB 状态）

| 优化项 | 优化前 | 优化后 |
|---|---|---|
| Checkpoint 方式 | 全量 Checkpoint | 增量 Checkpoint |
| Checkpoint 间隔 | 30 秒 | 3 分钟 |
| 状态 TTL | 无 | 90 天 |
| 本地恢复 | 关闭 | 开启 |
| TM 内存 | 8GB | 24GB（含 12GB Managed） |
| RocksDB Block Cache | 256MB | 6GB |
| 并行度 | 16 | 32 |
| **Checkpoint 耗时** | 8.5 分钟 | **42 秒** |
| **作业恢复时间** | 45 分钟 | **6 分钟** |
| **OOM 频率** | 每周 1-2 次 | **0 次** |
| **吞吐** | 3.2 万/秒 | **7.5 万/秒** |

## 超大状态作业调优 Checklist

```
基础配置
  □ 开启增量 Checkpoint（state.backend.incremental: true）
  □ 开启非对齐 Checkpoint（高负载必须）
  □ 开启本地状态恢复（local-backup.enabled: true）
  □ Checkpoint 超时设置充足（>= 状态大小 / 100MB/s）

内存
  □ 托管内存分配 >= 状态总量 / 并行度 × 1.5
  □ state.backend.rocksdb.memory.managed = true
  □ Block Cache 占托管内存 60-70%
  □ 使用 G1GC，MaxGCPauseMillis <= 200ms

状态设计
  □ 所有状态配置合理的 TTL
  □ 定期通过监控检查状态分布（防倾斜）
  □ 每天定期触发全量 Savepoint 作为基线
  □ 扩缩容时从 Savepoint（而非增量 CP）恢复

运维
  □ Checkpoint 大小告警（> 预期 3 倍触发告警）
  □ 恢复演练（每月 1 次模拟故障恢复，验证 SLA）
  □ 定期清理旧 Checkpoint（只保留最近 5 个）
```

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| 大状态加资源（并行度/内存）就能解决 | 状态倾斜、增量 CP 文件链过长、TTL 缺失等根因，加资源只是临时缓解 |
| 增量 Checkpoint 恢复总比全量快 | 扩缩容场景下增量 CP 恢复涉及多组 RocksDB 合并，写放大严重，反而比 Savepoint 慢 |
| Checkpoint 间隔越短越安全 | 大状态作业 Checkpoint 间隔太短（< 2分钟）会导致上一次 CP 还没完成就触发下一次，加剧 DFS 和 I/O 压力 |
| 状态倾斜只影响计算性能 | 倾斜节点的状态也大，Checkpoint 时该分片上传耗时拖慢整体 CP 完成时间 |

## 待验证缺口
- 开启本地恢复后，TM 节点重启（而非同机恢复）的场景下，本地缓存是否仍然有效？
- `state.backend.fs.memory-threshold` 过高导致 `_metadata` 过大的精确阈值？
- 字节跳动的 Union State 优化（JobMaster 端聚合成 1 个文件）是否已进入社区版本？
