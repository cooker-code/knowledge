# Flink TaskManager 内存模型

## 来源
- [精准调优！Flink内存模型详解与RocksDB调优指南](../文章/done-精准调优！Flink内存模型详解与RocksDB调优指南.md)
- [Flink RocksDB 状态后端深度调优](../文章/done-Flink%20RocksDB%20状态后端深度调优.md)
- [Flink 超大状态作业优化实战](../文章/done-Flink%20超大状态作业优化实战.md)
- [腾讯面试：Flink100G大状态如何优化？有哪些参数可以调整？](../文章/done-腾讯面试：Flink100G大状态如何优化？有哪些参数可以调整？.md)

## 核心问题
Flink TaskManager 的内存由哪些区域构成？RocksDB 使用的是哪块内存？各区域如何配置才能避免 OOM 且充分利用资源？

## 内存层次结构

```
进程总内存 (Total Process Memory) = taskmanager.memory.process.size
│
├── Flink 使用内存 (Total Flink Memory) = taskmanager.memory.flink.size
│   │
│   ├── 框架堆内存 (Framework Heap Memory)    ← Flink 框架自用
│   ├── 任务堆内存 (Task Heap Memory)         ← 用户代码运行
│   ├── 框架堆外内存 (Framework Off-Heap Memory)
│   ├── 任务堆外内存 (Task Off-Heap Memory)
│   ├── 网络内存 (Network Memory)             ← 数据传输缓冲
│   └── 托管内存 (Managed Memory)             ← RocksDB 使用这里！
│
└── JVM 特有内存 (JVM-Specific Memory)
    ├── JVM 元空间 (Metaspace)
    └── JVM 执行开销 (Overhead)
```

**关键结论：RocksDB 的 Block Cache、MemTable、Index/Filter 都从托管内存（Managed Memory）分配，不占用 JVM Heap。**

## 各内存区域配置规则

| 区域 | 配置参数 | 典型值 | 调优方向 |
|---|---|---|---|
| 进程总内存 | `taskmanager.memory.process.size` | 8g / 16g | 根据节点规格设定 |
| 任务堆内存 | `taskmanager.memory.task.heap.size` | 2g | 用户代码对象，避免 GC 过频 |
| 托管内存 | `taskmanager.memory.managed.size` | 4g-8g | **大状态作业最关键**，分给 RocksDB |
| 托管内存比例 | `taskmanager.memory.managed.fraction` | 0.3-0.5 | 与 `managed.size` 二选一 |
| 网络内存 | `taskmanager.memory.network.fraction` | 0.08 | 并发度高时可适当增大 |
| JVM 元空间 | `taskmanager.memory.jvm-metaspace.size` | 256m | 大量类加载时增大 |
| JVM 执行开销 | `taskmanager.memory.jvm-overhead.min/max` | 1g-3g | 通常保持默认 |

## 与 RocksDB 的联动关系

开启托管内存统一管理后，RocksDB 在托管内存内部按比例分配：

```yaml
state.backend.rocksdb.memory.managed: true      # 开启托管内存管理（强烈推荐）
state.backend.rocksdb.memory.write-buffer-ratio: 0.4  # 40% 给 MemTable，60% 给 Block Cache
```

**内存分配经验比例：**
- Block Cache：托管内存的 60%（读多写少）
- MemTable × N：托管内存的 30-40%（写多场景调大）
- 索引/过滤器等：约 10%

**每个 Slot 的托管内存 = `managed.size` / `numberOfTaskSlots`**

## 配置示例

### 8GB 节点（小状态作业）

```yaml
taskmanager.memory.process.size: 8g
taskmanager.memory.task.heap.size: 2g
taskmanager.memory.managed.size: 2g     # 状态小，管理内存适中
state.backend.rocksdb.memory.managed: true
```

### 16GB 节点（大状态作业，推荐配置）

```yaml
taskmanager.memory.process.size: 16g
taskmanager.memory.task.heap.size: 2g
taskmanager.memory.managed.size: 8g     # 最关键：分配足够托管内存给 RocksDB
taskmanager.memory.network.fraction: 0.08
taskmanager.memory.jvm-overhead.min: 1g
taskmanager.memory.jvm-overhead.max: 3g
taskmanager.numberOfTaskSlots: 2        # 每 Slot 分到 4g Managed Memory
state.backend.rocksdb.memory.managed: true
state.backend.rocksdb.memory.write-buffer-ratio: 0.5
```

### JVM GC 推荐配置

```yaml
env.java.opts.taskmanager: >
  -XX:+UseG1GC
  -XX:MaxGCPauseMillis=200
  -XX:G1HeapRegionSize=32m
  -XX:InitiatingHeapOccupancyPercent=45
  -XX:ParallelGCThreads=4
  -XX:ConcGCThreads=2
  -XX:+ParallelRefProcEnabled
  -Xloggc:/data/logs/flink-gc.log
```

## 关键监控指标

```
# 托管内存使用
Managed_Memory_Used
Managed_Memory_Available

# JVM 内存
JVM_Heap_Used
JVM_NonHeap_Used
JVM_Metaspace_Used

# GC
Garbage_Collection_Time   # >1秒/分钟 需优化
Garbage_Collection_Count
```

**监控告警阈值：**
- 内存使用率 > 85% → 触发警告
- GC 时间 > 1秒/分钟 → 需要优化
- Full GC 间隔 < 1小时 或单次 > 1秒 → 堆内存配置不合理

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| RocksDB 内存不受 JVM Heap 控制，不会 OOM | RocksDB 使用托管内存（堆外）；托管内存配置过小或未开启 `memory.managed=true` 时，RocksDB 会无限制增长导致进程级 OOM |
| 增大 JVM Heap 可以解决 RocksDB OOM | RocksDB 不用 JVM Heap，增大 Heap 无效；应增大 `managed.size` |
| 托管内存越大越好 | 托管内存占用的是进程内存，过大会挤压网络缓冲、元空间等，也可能导致 OS 层面内存压力 |
| Network Memory 不影响状态性能 | 高并发度作业中，Network Buffer 不足会加剧反压，间接影响 Checkpoint 超时和状态积压 |

## 待验证缺口
- `taskmanager.memory.managed.fraction` 和 `managed.size` 同时设置时的优先级规则？
- Flink 1.14+ 中 Changelog State Backend 对托管内存的额外消耗如何估算？
- 多 Slot 场景下，各 Slot 是否共享 Block Cache 实例还是独立分配？
