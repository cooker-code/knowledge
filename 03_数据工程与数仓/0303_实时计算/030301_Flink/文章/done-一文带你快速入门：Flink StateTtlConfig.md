> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkStateTTL状态生命周期治理|FlinkStateTTL状态生命周期治理]]
---
title: 一文带你快速入门：Flink StateTtlConfig
author: 语兴数据
date:
url: https://mp.weixin.qq.com/s?__biz=MzkwODYwNjExMA==&mid=2247487388&idx=1&sn=87bc34e87051d7a1a44462d4610e90a2&chksm=c14840d2cd2d9cceb5619492b8f6049891ce3070b18e8c25b1e5a80b16bf255ea98ea044cd21&mpshare=1&scene=24&srcid=0807h0gmuwo5J1UxTajzK9Pa&sharer_shareinfo=f6b71200cc618ce91900eedd49c00c98&sharer_shareinfo_first=f6b71200cc618ce91900eedd49c00c98#rd
---

### 作者：左美美（Apache Flink贡献者）

想要更直观地学习 Flink 相关技术？

欢迎关注B站账号：左美美（https://space.bilibili.com/1710526702）

在这里你可以找到：

* • Flink 核心概念视频讲解
* • 实战案例演示
* • 性能优化实践

关注公众号：**漫话架构之美**，获取更多架构设计与技术实践文章

### Flink StateTtlConfig 详解

`StateTtlConfig` 是 Apache Flink 中用于配置状态生存时间 (Time-To-Live, TTL) 的重要类。

它允许你为 Flink 应用程序中的托管状态（Managed State，如 `ValueState`, `MapState`, `ListState` 等）设置过期策略，从而自动清理不再需要的数据，有效控制状态大小，优化内存和磁盘使用，并提高应用程序的稳定性和性能。

### 为什么需要 State TTL？

在 Flink 流处理应用中，状态会随着时间的推移不断积累。如果不加以管理，状态可能会无限增长，导致以下问题：

* • **内存/磁盘压力：** 占用过多的内存或磁盘空间，导致 OOM (Out Of Memory) 或磁盘写满，影响作业的正常运行。
* • **Checkpoint/Savepoint 效率下降：** 检查点和保存点的大小会随着状态增大而增大，导致其创建和恢复时间变长，影响故障恢复速度。
* • **性能下降：** 大状态会增加状态访问的开销，降低处理吞吐量。
* • **业务需求：** 某些业务场景下，只需要在一定时间内保留数据，过期数据就没有价值了。

`StateTtlConfig` 正是为了解决这些问题而设计的。

### `StateTtlConfig` 的主要配置项及使用场景

`StateTtlConfig` 通过其 `Builder` 类来构建，提供了一系列灵活的配置选项。下面我们来详细解析各个配置项及其使用场景：

#### 1. `setTtl(Time ttl)`

* • **作用：** 设置状态的生存时间。这是 TTL 配置的核心，定义了状态项从最后一次更新或创建开始，多长时间后过期。
* • **参数：** `Time ttl`，一个 `org.apache.flink.api.common.time.Time` 对象，表示时间长度（例如 `Time.minutes(5)`, `Time.hours(3)`）。
* • 使用场景：

+ • **会话管理：** 记录用户会话信息，当用户长时间不活跃时，会话状态过期。例如，一个电商网站的用户购物车状态，如果用户 30 分钟没有操作，购物车内容就过期。
+ • **实时指标计算：** 统计过去一段时间内的某个指标，例如过去 5 分钟的点击量。超过 5 分钟的数据就无需保留。
+ • **反欺诈：** 记录用户在短时间内的行为，例如某个 IP 在 10 秒内尝试登录失败的次数。超过 10 秒的尝试记录就无效。
+ • **实时数据缓存：** 将外部数据缓存到状态中，设置 TTL 以确保缓存数据不过期，例如，缓存最近 1 小时热门商品的库存信息。

#### 2. `setTtlTimeCharacteristic(StateTtlConfig.TtlTimeCharacteristic ttlTimeCharacteristic)`

* • **作用：** 配置 TTL 的时间特性，即基于什么时间来计算状态的过期时间。
* • 参数：

  ```
  StateTtlConfig.TtlTimeCharacteristic
  ```

  枚举类型，只有1个值：

+ • `ProcessingTime` (默认值): 基于 Flink 任务的处理时间（系统时间）来计算 TTL。

* • 使用场景：

+ • `ProcessingTime` (处理时间):

- • **最常用和简单的场景：** 当你对时间戳的准确性要求不高，或者事件时间与处理时间差异不大时，使用处理时间更简单高效。
- • **实时报警：** 对当前正在处理的数据进行实时监控，并设置过期时间以避免长时间保留旧的告警状态。
- • **资源敏感型应用：** 处理时间 TTL 通常开销更小，因为它不需要维护事件时间水位线。

#### 3. `setUpdateType(StateTtlConfig.UpdateType updateType)`

* • **作用：** 配置状态 TTL 的更新策略，即什么时候重置状态的 TTL 计时器。
* • 参数：

  ```
  StateTtlConfig.UpdateType
  ```

  枚举类型，有三个值：

+ • `OnCreateAndWrite` (默认值): 在状态创建时和每次写入（更新）时重置 TTL 计时器。
+ • `OnReadAndWrite`: 在状态创建、写入和读取时重置 TTL 计时器。
+ • `OnCreate`: 只有在状态创建时重置 TTL 计时器，后续写入或读取不会重置。

* • 使用场景：

+ • `OnCreateAndWrite` (默认):

- • **大多数场景适用：** 这是一个平衡的策略，既能保证状态在不活动时过期，也能在状态频繁更新时保持活跃。例如，一个会话状态，用户每次操作都会更新会话状态，从而延长其生命周期。
- • **计数器：** 统计某个键在一段时间内的事件数量，每次事件到来更新计数器，并延长计数器的生命周期。

+ • `OnReadAndWrite`:

- • **活跃度监控：** 需要精确知道状态是否被“使用”的场景，例如，如果一个缓存项被频繁读取，那么它应该更长时间地保留。如果一个用户一直活跃地使用系统，即使没有写入数据，其会话状态也应该保持。
- • **读写都很重要的交互型应用：** 例如，一个需要频繁查询和更新的用户个人资料信息。

+ • `OnCreate`:

- • **固定生命周期的数据：** 某些数据一旦创建就有一个固定的生命周期，无论后续是否被读取或写入。例如，一个订单在创建后，希望在 24 小时后过期，无论它是否被查询或修改。
- • **一次性事件：** 记录一次性事件的状态，该状态在事件发生后一段时间内有效，之后就过期。

#### 4. `setStateVisibility(StateTtlConfig.StateVisibility stateVisibility)`

* • **作用：** 配置过期状态的可见性，即过期但尚未被清理的状态是否仍然可以被读取。
* • 参数：

  ```
  StateTtlConfig.StateVisibility
  ```

  枚举类型，有两个值：

+ • `NeverReturnExpired` (默认值): 永远不返回已过期但尚未被物理清理的状态。当状态过期后，即使它仍然存在于存储中，也会被 Flink 逻辑上视为不存在。
+ • `ReturnExpiredIfNotCleanedUp`: 如果过期状态尚未被物理清理，则可以返回。这意味着你可以读取到逻辑上已经过期但物理上还在的状态。

* • 使用场景：

+ • `NeverReturnExpired` (默认):

- • **严格的数据一致性要求：** 大多数场景下推荐使用此选项，确保应用程序只会处理有效（未过期）的数据。
- • **防止逻辑错误：** 避免业务逻辑因为读取到“脏”数据而产生错误。

+ • `ReturnExpiredIfNotCleanedUp`:

- • **调试和审计：** 在某些特殊的调试或审计场景下，你可能需要查看已经过期但尚未被清理的数据。
- • **非严格一致性要求：** 如果你的应用程序能够容忍偶尔读取到过期数据，并且希望在数据被物理清理之前仍然可以访问，可以考虑使用此选项。但通常不推荐在生产环境中使用，除非有非常明确的理由。
- • **注意：** 使用此选项意味着你可能会读取到逻辑上过期的数据，需要业务逻辑自己处理这种情况。

#### 5. 清理策略 (Cleanup Strategies)

除了上述核心配置外，`StateTtlConfig` 还提供了多种过期状态的清理策略。FLink 默认会通过后台线程进行异步清理。你可以通过 `disableCleanupInBackground()` 禁用默认后台清理。

* • **`cleanupIncrementally(int cleanupSize, boolean runCleanupForEveryRecord)`**

+ • **作用：** 配置增量式清理。当访问状态时，会检查并清理一部分过期状态。
+ • 参数：

- • `cleanupSize`: 每次清理时检查并清理的记录数量。
- • `runCleanupForEveryRecord`: 是否对每条记录都执行清理操作。如果为 `true`，则每次访问状态时都会触发清理；如果为 `false`，则会根据 `cleanupSize` 和内部逻辑进行批次清理。

+ • 使用场景：

- • **状态访问频繁的场景：** 适用于那些状态会被频繁读写，并且希望在访问时顺便清理过期数据的场景。
- • **避免一次性大量清理导致的暂停：** 增量清理可以将清理开销分摊到正常的处理过程中，避免在检查点时一次性清理大量数据导致的长时间暂停。
- • **资源受限但需要及时清理的场景：** 在资源受限的情况下，增量清理可以更平滑地释放资源。

* • **`cleanupFullSnapshot()`**

+ • **作用：** 配置在生成完整检查点（Full Checkpoint）时清理过期状态。
+ • 使用场景：

- • **对检查点大小敏感的场景：** 每次完整检查点都会清理过期数据，从而减小检查点的大小，加速恢复。
- • **状态后端支持快照清理：** 某些状态后端（如 RocksDB）可能更好地支持在快照生成时进行清理。
- • **注意：** 这种清理方式只在完整检查点时发生，增量检查点不会触发清理。因此，在两个完整检查点之间，过期状态可能仍然存在。

* • **`cleanupInRocksdbCompactFilter(long queryTimeAfterNumEntries)` (仅适用于 RocksDBStateBackend)**

+ • **作用：** 配置在 RocksDB compaction（压缩）过程中通过 RocksDB 的 Compaction Filter 来清理过期状态。
+ • **参数：** `queryTimeAfterNumEntries`: 用于优化查询的参数，具体作用与 RocksDB 内部机制相关。
+ • 使用场景：

- • **使用 RocksDBStateBackend 的场景：** 这是 RocksDB 特有的优化，利用其内部的压缩机制来清理过期数据，效率较高。
- • **处理大状态的场景：** RocksDB 的 Compaction 是后台操作，可以异步清理大量过期状态，对 Flink 主处理线程的影响较小。
- • **希望更彻底清理磁盘空间的场景：** RocksDB 的 Compaction 会真正地从磁盘上删除过期数据。

+ • **注意：** 这种清理方式是异步的，并且依赖于 RocksDB 的 Compaction 策略，可能无法立即释放空间。

* • **`disableCleanupInBackground()`**

+ • **作用：** 禁用 Flink 默认的后台清理机制。
+ • 使用场景：

- • **你完全依赖其他清理策略时：** 如果你明确配置了增量清理或 RocksDB Compaction 清理，并且认为默认的后台清理是多余的或干扰的，可以禁用它。
- • **自定义清理逻辑时：** 在一些非常特殊的场景下，你可能希望完全控制状态的清理，此时可以禁用 Flink 的内置清理机制，并实现自己的清理逻辑（例如，通过自定义定时器）。

### 示例代码

```
import org.apache.flink.api.common.state.StateTtlConfig;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.util.Collector;

publicclassFlinkStateTtlExample {

    publicstaticvoidmain(String[] args)throws Exception {
        StreamExecutionEnvironmentenv= StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 示例数据：(userId, eventType)
        DataStream<Tuple2<String, String>> inputStream = env.fromElements(
                Tuple2.of("user1", "login"),
                Tuple2.of("user2", "view"),
                Tuple2.of("user1", "click"),
                Tuple2.of("user3", "add_to_cart"),
                Tuple2.of("user1", "logout")
        );

        // 构建 StateTtlConfig
        StateTtlConfigttlConfig= StateTtlConfig
                .newBuilder(Time.minutes(5)) // 设置状态 TTL 为 5 分钟
                .setTtlTimeCharacteristic(StateTtlConfig.TtlTimeCharacteristic.ProcessingTime) // 基于处理时间
                .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite) // 在创建和写入时更新 TTL
                .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired) // 永不返回过期状态
                // .cleanupIncrementally(10, true) // 增量清理，每次清理 10 条记录，每条记录访问时都触发
                // .cleanupFullSnapshot() // 在完整快照时清理
                // .cleanupInRocksdbCompactFilter(1000L) // RocksDB 特有清理 (如果使用 RocksDBStateBackend)
                .build();

        inputStream
                .keyBy(t -> t.f0) // 按 userId 分组
                .process(newKeyedProcessFunction<String, Tuple2<String, String>, String>() {

                    // 声明一个 ValueState 用于存储用户最后一次操作的时间
                    private ValueState<Long> lastOperationTime;

                    @Override
                    publicvoidopen(Configuration parameters)throws Exception {
                        // 创建 ValueStateDescriptor 并应用 TTL 配置
                        ValueStateDescriptor<Long> descriptor = newValueStateDescriptor<>(
                                "lastOperationTime",
                                Long.class
                        );
                        descriptor.enableTimeToLive(ttlConfig); // 启用 TTL

                        lastOperationTime = getRuntimeContext().getState(descriptor);
                    }

                    @Override
                    publicvoidprocessElement(Tuple2<String, String> value, Context ctx, Collector<String> out)throws Exception {
                        LongcurrentTime= System.currentTimeMillis();

                        if (lastOperationTime.value() == null) {
                            out.collect("User " + value.f0 + " first operation at " + currentTime);
                        } else {
                            out.collect("User " + value.f0 + " operation at " + currentTime + ", previous operation at " + lastOperationTime.value());
                        }

                        // 更新状态，这将重置 TTL 计时器
                        lastOperationTime.update(currentTime);

                        // 模拟一些延迟，以便观察 TTL 效果
                        Thread.sleep(1000); // 模拟处理时间流逝
                    }
                })
                .print();

        env.execute("Flink State TTL Example");
    }
}
```

**代码解释：**

1. 1. **`StateTtlConfig.newBuilder(Time.minutes(5))`**: 初始化 `StateTtlConfig`，设置状态的生存时间为 5 分钟。
2. 2. **`setTtlTimeCharacteristic(StateTtlConfig.TtlTimeCharacteristic.ProcessingTime)`**: 指定基于处理时间计算 TTL。
3. 3. **`setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)`**: 每次创建或更新状态时，都会刷新 TTL 计时器。
4. 4. **`setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)`**: 一旦状态过期，即使尚未被物理清理，也不会被读取到。
5. 5. **`descriptor.enableTimeToLive(ttlConfig)`**: 在 `ValueStateDescriptor` 上启用 TTL 配置。这是将 TTL 应用到特定状态的关键步骤。

在这个例子中，`lastOperationTime` 状态会为每个用户存储其最后一次操作的时间。如果一个用户在 5 分钟内没有任何操作，那么其 `lastOperationTime` 状态就会过期，下次再访问时会得到 `null` 值，模拟会话过期。

### 总结

`StateTtlConfig` 是 Flink 状态管理中一个非常强大和实用的功能。通过合理配置 TTL，可以：

* • **有效控制状态大小：** 避免状态无限增长，提高资源利用率。
* • **优化 Checkpoint/Savepoint：** 减小快照大小，缩短创建和恢复时间。
* • **提高作业性能：** 减少状态访问开销。
* • **满足业务需求：** 自动清理过期数据，简化业务逻辑。

### 关注语兴数据官方账号，学习更多数据技术知识