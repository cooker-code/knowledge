> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkWebUI指标解读|FlinkWebUI指标解读]]、[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink调优方法论|Flink调优方法论]]
---
title: Flink 性能调优实战
author: DATA炼狱
date:
url: https://mp.weixin.qq.com/s?__biz=MzUyMDkzNjAzMQ==&mid=2247484367&idx=1&sn=18876c9ddf4bdddd1c3af481624c2ffb&chksm=f85ea618c4ead4e5e94c8b40d60de8736ebb353015c806fa1dc8b0c26993e56159b8189bdd0d&mpshare=1&scene=24&srcid=0414nkf6NDvs93x6U1JPNKw3&sharer_shareinfo=f5c7af2e1c504202505531b08d8ea3e6&sharer_shareinfo_first=f5c7af2e1c504202505531b08d8ea3e6#rd
---

# Flink 性能调优实战

## 性能调优全局思路

Flink 性能问题排查需要从以下维度逐步定位：

```
调优优先级（高 → 低）：

① 反压排查      ← 最常见，先确认瓶颈在哪个算子
② 数据倾斜      ← keyBy 分配不均导致部分 Task 负载过高
③ Checkpoint 慢 ← 影响故障恢复与整体吞吐
④ 资源配置      ← 并行度、内存、CPU 配置是否合理
⑤ 代码层优化    ← 减少对象创建、序列化、不必要的状态
```

**必备工具：**

| 工具 | 用途 |
| --- | --- |
| Flink Web UI | 查看反压、Task 耗时、Checkpoint 状态 |
| Flame Graph | 分析 CPU 热点（Flink 1.13+ 内置） |
| JVM GC 日志 | 分析内存和 GC 压力 |
| Metrics 监控 | Prometheus + Grafana 实时观测指标 |

---

## 反压排查与解决

### 发现反压

进入 Flink Web UI → 作业 → 选择某个算子 → **Backpressure** 标签：

| 状态 | 含义 |
| --- | --- |
| `OK (0%)` | 无反压，运行正常 |
| `LOW (< 10%)` | 轻微反压，暂时可忽略 |
| `HIGH (> 50%)` | 严重反压，需立即排查 |

### 定位反压根源

```
反压传导方向（从下游向上游）：

  Source → op1 → op2 → op3 → Sink
                        ↑
                   这里是瓶颈

  op3 处理慢 → op2 发不出数据 → op1 积压 → Source 降速

  判断方法：从 Sink 往上找，第一个"正常"的算子的下游就是瓶颈
  - 瓶颈算子：输入 Busy 高、输出 Backpressure 高
  - 上游算子：输出 Backpressure 高（被下游堵住）
```

### 常见反压原因与解决方案

**原因1：下游 Sink 写入慢**

```
// 问题：JDBC Sink 每条记录单独写入
stream.addSink(JdbcSink.sink(...));

// 解决：批量写入
JdbcExecutionOptions options = JdbcExecutionOptions.builder()
    .withBatchSize(5000)           // 攒够5000条才写
    .withBatchIntervalMs(2000)     // 或者每2秒写一次
    .withMaxRetries(3)
    .build();

// 解决：并发写入（提高 Sink 并行度）
stream.addSink(sink).setParallelism(8);
```

**原因2：外部调用阻塞**

```
// 问题：同步调用 Redis/HTTP，每条记录都等待
stream.map(record -> {
    String result = httpClient.get("http://api/enrich/" + record.getId()); // 阻塞！
    return enrich(record, result);
});

// 解决：使用异步 IO
DataStream<EnrichedRecord> enriched = AsyncDataStream.unorderedWait(
    stream,
    new AsyncEnrichFunction(),
    5000, TimeUnit.MILLISECONDS,
    200  // 最大并发请求数
);
```

**原因3：GC 停顿**

```
# 查看 GC 日志
-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:/tmp/gc.log

# 解决：减小堆内存，使用 G1GC
-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:G1HeapRegionSize=32m

# 或者将大状态迁移到 RocksDB（堆外）
env.setStateBackend(new EmbeddedRocksDBStateBackend());
```

---

## 数据倾斜处理

### 识别数据倾斜

Web UI 中 Task 指标差异悬殊：某些 Task 处理了 95% 的数据，其他几乎空闲。

```
// 代码排查：打印 key 分布
stream.keyBy(r -> r.getUserId())
      .process(new KeyedProcessFunction<String, Record, String>() {
          private ValueState<Long> countState;
          @Override
          public void processElement(Record r, Context ctx, Collector<String> out) throws Exception {
              Long count = countState.value();
              count = (count == null ? 0 : count) + 1;
              countState.update(count);
              if (count % 10000 == 0) {
                  // 每万条打印一次，看哪个 key 特别多
                  System.out.println("key=" + r.getUserId() + " count=" + count);
              }
          }
      });
```

### 解决方案一：两阶段聚合（打散热点 key）

```
// 第一阶段：key 加随机后缀，先局部聚合（减少数据量）
DataStream<Tuple2<String, Long>> localAgg = stream
    .map(r -> Tuple2.of(r.getCategory() + "_" + (int)(Math.random() * 8), 1L))
    .returns(Types.TUPLE(Types.STRING, Types.LONG))
    .keyBy(t -> t.f0)
    .reduce((a, b) -> Tuple2.of(a.f0, a.f1 + b.f1));

// 第二阶段：去掉后缀，做全局聚合
DataStream<Tuple2<String, Long>> globalAgg = localAgg
    .map(t -> Tuple2.of(t.f0.substring(0, t.f0.lastIndexOf("_")), t.f1))
    .returns(Types.TUPLE(Types.STRING, Types.LONG))
    .keyBy(t -> t.f0)
    .reduce((a, b) -> Tuple2.of(a.f0, a.f1 + b.f1));
```

### 解决方案二：热点 key 单独处理

```
// 识别热点 key（如大 V 用户、爆款商品）
Set<String> hotKeys = new HashSet<>(Arrays.asList("hot_item_001", "hot_item_002"));

// 热点 key 走单独处理逻辑
DataStream<Record> hotStream = stream.filter(r -> hotKeys.contains(r.getItemId()));
DataStream<Record> normalStream = stream.filter(r -> !hotKeys.contains(r.getItemId()));

// 热点流：直接发到 Sink，不做 keyBy 聚合（避免倾斜）
hotStream.addSink(new HotItemSink());

// 普通流：正常聚合
normalStream.keyBy(Record::getItemId).window(...).aggregate(...);
```

### 解决方案三：Flink SQL 打散倾斜

```
-- 使用随机分桶减少倾斜（适合窗口聚合）
SELECT
    category,
    SUM(local_sum) AS total_amount
FROM (
    -- 第一层：随机分桶，局部聚合
    SELECT
        category,
        bucket,
        SUM(amount) AS local_sum
    FROM (
        SELECT *, FLOOR(RAND() * 8) AS bucket
        FROM orders
    )
    GROUP BY category, bucket,
        window_start, window_end
    -- （TVF窗口写法）
)
GROUP BY category;
```

---

## Checkpoint 调优

### Checkpoint 耗时分析

Web UI → 作业 → **Checkpoints** → 查看各阶段耗时：

| 阶段 | 耗时来源 | 优化方向 |
| --- | --- | --- |
| Alignment | 等待 Barrier 对齐 | 开启非对齐 Checkpoint |
| Sync Duration | 同步快照（stop-the-world） | 减小状态大小 |
| Async Duration | 异步上传到 HDFS/S3 | 提升存储 IO，用 S3 并发上传 |

### 核心优化配置

```
CheckpointConfig config = env.getCheckpointConfig();

// 1. 开启非对齐 Checkpoint（反压严重时 Checkpoint 超时问题）
config.enableUnalignedCheckpoints();

// 2. 使用 RocksDB 增量 Checkpoint（只上传变化的 SST 文件）
EmbeddedRocksDBStateBackend backend = new EmbeddedRocksDBStateBackend(true);
env.setStateBackend(backend);

// 3. 调整 Checkpoint 间隔和超时
env.enableCheckpointing(120_000);           // 2分钟一次
config.setCheckpointTimeout(180_000);       // 3分钟超时
config.setMinPauseBetweenCheckpoints(60_000); // 两次间隔至少1分钟

// 4. 允许 Checkpoint 失败（不影响作业继续运行）
config.setTolerableCheckpointFailureNumber(5);
```

### RocksDB 调优

```
# flink-conf.yaml 中 RocksDB 相关配置

# 增大写缓冲区（减少刷盘次数）
state.backend.rocksdb.writebuffer.size: 128mb
state.backend.rocksdb.writebuffer.count: 3

# 增大读缓存（提升读性能）
state.backend.rocksdb.block.cache-size: 256mb

# 并发压缩线程
state.backend.rocksdb.compaction.level.max-size-level-base: 256mb

# 使用 SSD 路径（减少 IO 延迟）
state.backend.rocksdb.localdir: /ssd/flink/rocksdb
```

---

## 资源与并行度调优

### 并行度设置原则

```
// 全局并行度 = TaskManager 数 × 每个 TM 的 Slot 数
// 建议：Source 并行度 = Kafka 分区数，其他算子视处理能力调整

// Source：与 Kafka 分区数相同（充分利用 Kafka 并行度）
kafkaSource.setParallelism(kafkaPartitions);

// 重计算算子（CPU 密集）：适当提高
computeHeavyOp.setParallelism(kafkaPartitions * 2);

// Sink：根据目标系统并发限制设置
sink.setParallelism(8);

// 窗口算子：不需要太高（数据已经按 key 路由）
windowOp.setParallelism(4);
```

### 内存配置

```
# flink-conf.yaml

# TaskManager 总内存
taskmanager.memory.process.size: 8g

# 各部分内存分配
taskmanager.memory.task.heap.size: 2g        # 用户代码 Heap
taskmanager.memory.managed.fraction: 0.4     # 托管内存（RocksDB/排序）40%
taskmanager.memory.network.fraction: 0.1     # 网络缓冲区 10%
taskmanager.memory.framework.heap.size: 256m # Flink 框架 Heap

# 每个 Slot 数（建议 1~2 个）
taskmanager.numberOfTaskSlots: 2
```

### 算子链调优

```
// 禁止某个算子加入链（当该算子是瓶颈，需要独立线程时）
stream.map(heavyMapper).disableChaining();

// 强制开始新的算子链
stream.map(mapper1).startNewChain().map(mapper2);

// 全局禁止算子链（调试用，生产不推荐）
env.disableOperatorChaining();
```

---

## 序列化调优

Flink 默认使用 Kryo 序列化，对 POJO 类型会触发类型推断，性能较差。

### 使用 TypeInformation 明确类型

```
// 不推荐：让 Flink 推断类型（可能用 Kryo）
stream.map(r -> new Order(r.getId(), r.getAmount()));

// 推荐：明确告知类型
stream.map(r -> new Order(r.getId(), r.getAmount()))
      .returns(TypeInformation.of(Order.class));

// 或在 Lambda 中使用 returns()
stream.flatMap((String s, Collector<Tuple2<String, Integer>> out) -> {
    out.collect(Tuple2.of(s, 1));
}).returns(Types.TUPLE(Types.STRING, Types.INT));
```

### POJO 类型优化

Flink 对 POJO 类型有专门的序列化器（比 Kryo 快），满足以下条件自动启用：

```
// POJO 条件：
// 1. public 类
// 2. 无参构造函数
// 3. 所有字段 public 或有 getter/setter
// 4. 字段类型本身也是 POJO 或基本类型

public class Order {
    public String orderId;    // public 字段
    public double amount;
    public long eventTime;

    public Order() {}         // 无参构造

    public Order(String orderId, double amount, long eventTime) {
        this.orderId = orderId;
        this.amount = amount;
        this.eventTime = eventTime;
    }
}
```

### 注册 Kryo 序列化器

```
// 对无法变成 POJO 的类（如第三方类），注册专用序列化器
env.getConfig().registerTypeWithKryoSerializer(MyClass.class, MyKryoSerializer.class);

// 完全禁用 Kryo（强制使用 Flink 原生序列化，发现不支持的类型会报错）
env.getConfig().disableGenericTypes();
```

---

## 网络与吞吐调优

### 缓冲区调优

```
# 网络缓冲区超时（数据在缓冲区等待多久后强制刷出）
# 降低延迟：缩短超时（0 = 立即发送，牺牲吞吐）
# 提高吞吐：延长超时（100ms 可显著提升批量传输效率）
taskmanager.network.netty.sendReceiveBufferSize: 4096
execution.buffer-timeout: 100ms       # 默认 100ms

# 每个 Gate 的缓冲区数量
taskmanager.network.memory.buffers-per-channel: 2
taskmanager.network.memory.floating-buffers-per-gate: 8
```

### 微批优化（提升吞吐）

```
// 关闭对象重用（默认关闭，开启后对象被复用，提高性能但不安全）
env.getConfig().enableObjectReuse();

// 注意：开启对象重用后，不能在异步操作中持有对象引用
```

---

## 性能调优 Checklist

| 检查项 | 命令/位置 | 目标值 |
| --- | --- | --- |
| 反压状态 | Web UI → Backpressure | 全部 OK |
| Task 负载均衡 | Web UI → Task Managers → Task | 各 Task 记录数相近 |
| Checkpoint 耗时 | Web UI → Checkpoints | < 作业间隔的 50% |
| GC 时间占比 | JVM Metrics → GC Time | < 5% |
| 网络缓冲使用率 | Metrics → inPoolUsage | < 80% |
| 状态大小 | Metrics → stateSize | 符合预期，无持续增长 |
| CPU 利用率 | TaskManager Metrics | 60%~80%（留有余量） |