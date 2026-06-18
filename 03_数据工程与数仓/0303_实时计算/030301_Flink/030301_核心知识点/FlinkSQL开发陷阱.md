# SQL 开发隐蔽陷阱与 View 避坑

## 来源
- [Flink技术实践-Flink SQL 开发中的隐蔽陷阱](../文章/done-Flink技术实践-Flink SQL 开发中的隐蔽陷阱.md)
- [Flink技术实践-FlinkSQL视图View避坑指南](../文章/done-Flink技术实践-FlinkSQL视图View避坑指南.md)
- [深入理解 Flink SQL 状态：原理、应用与优化](../文章/done-深入理解 Flink SQL 状态：原理、应用与优化.md)

## 核心问题
Flink SQL 看似正确但结果不对/作业崩溃的问题有哪些根本原因？View 为什么会导致多个下游结果不一致或性能急剧下降？如何系统地规避这些隐蔽陷阱？

## 判断准则

### 陷阱速查表

| 陷阱编号 | 现象 | 根因 | 解决方案 |
|---|---|---|---|
| 1 | 窗口持续不关闭，统计结果不输出 | 空闲 Kafka 分区阻塞全局 Watermark | 配置空闲超时 `scan.watermark.idle-timeout` 或全局 `table.exec.source.idle-timeout` |
| 2 | 数据乱序导致统计偏差，无法复现 | Processing Time 与 Event Time 混用 | 实时统计强制用事件时间；禁止混用 |
| 3 | `'100' + 20` 抛出 CastException | Flink SQL 强类型安全，不支持 Hive/MySQL 式隐式类型转换 | 所有转换用 CAST；容错场景用 `TRY_CAST` |
| 4 | 双流 Join 运行几天后状态暴涨 OOM | 普通双流 Join 永久保存两侧数据，无自动清理 | 搭配窗口（TUMBLE/HOP）限定范围；或设 State TTL |
| 5 | 维表 Join 后 DB QPS 突增、作业反压 | 默认同步查询，每条数据发一次 DB 请求 | DDL 配置 LRU 缓存 + `lookup.async = 'true'` |
| 6 | JOIN 后某维度数据静默"蒸发" | NULL 值三值逻辑：`NULL = NULL` 结果是 UNKNOWN 而非 TRUE，记录被排除 | DDL 声明 NOT NULL；Join 条件用 COALESCE 或 IS NOT NULL 过滤 |
| 7 | 改聚合函数后用 Savepoint 重启报错 | SQL 变更可能生成不同执行计划，状态结构不兼容 | 上线前核对官方兼容性参考表；容忍状态丢失可设 `execution.savepoint.ignore-unclaimed-state=true` |
| 8 | GROUP BY 只有一个 Subtask busy，持续反压 | 数据倾斜，热点 Key 集中 | MiniBatch + LocalGlobal 两阶段聚合；COUNT DISTINCT 开 Split Distinct |

### 空闲源导致窗口不触发（陷阱 1）详解

```sql
-- 表级配置（优先级高）
CREATE TABLE kafka_table (...) WITH (
  'connector' = 'kafka',
  ...
  'scan.watermark.idle-timeout' = '1min'  -- Flink 1.18+
);

-- 全局配置
SET 'table.exec.source.idle-timeout' = '60s';
-- 表级参数优先于全局参数
```

### NULL 三值逻辑（陷阱 6）详解

```sql
-- 错误：NULL 关联键导致数据丢失
SELECT a.*, b.name FROM a JOIN b ON a.user_id = b.user_id;
-- NULL = NULL → UNKNOWN → 记录被排除

-- 正确：显式处理 NULL
SELECT a.*, b.name FROM a
JOIN b ON COALESCE(a.user_id, '') = COALESCE(b.user_id, '');

-- 或在 DDL 中声明 NOT NULL
CREATE TABLE source (..., user_id BIGINT NOT NULL, ...);
```

### 数据倾斜优化配置（陷阱 8）

```sql
-- 1. 开启 MiniBatch
SET 'table.exec.mini-batch.enabled' = 'true';
SET 'table.exec.mini-batch.allow-latency' = '2s';
SET 'table.exec.mini-batch.size' = '5000';

-- 2. LocalGlobal 两阶段聚合（需先开 MiniBatch）
SET 'table.optimizer.agg-phase-strategy' = 'TWO_PHASE';

-- 3. Split Distinct（针对 COUNT DISTINCT）
SET 'table.optimizer.distinct-agg.split.enabled' = 'true';
```

---

### View（视图）陷阱速查表

| 陷阱 | 现象 | 原因 | 解决方案 |
|---|---|---|---|
| 临时视图 vs 永久视图混淆 | 生产环境部署时"视图不存在" | 临时视图（TEMPORARY VIEW）会话结束后自动删除 | 生产环境用永久视图，纳入版本控制 |
| 多下游引用同一视图结果不一致 | 多个 Sink 引用同一视图时数据有偏差 | 视图每次引用都重新展开执行，涉及外部状态时结果不同 | 用物理中间表（如 Paimon/Kafka）物化共享逻辑 |
| 视图嵌套过深性能急剧下降 | 嵌套超 3 层时延迟持续升高 | 优化器无法生成最优计划，"查询膨胀" | 视图扁平化；物化中间结果；用 CTE 替代 |
| 字段类型不匹配 | 关联查询类型转换错误 | 视图字段依赖自动推断，基础表变更后视图未同步 | 显式声明字段类型；基础表变更时校验视图兼容性 |
| 窗口函数时间属性传递错误 | 窗口无结果输出；状态持续增长 | 视图中时间属性未正确传递或被修改 | 视图内不修改时间字段值；确保 WATERMARK 正确传递 |
| 误以为普通视图会物化数据 | 期望查询性能提升，实际无改善 | 普通视图无物理存储，每次查询动态计算 | Flink 1.20+ 才支持物化视图，需显式指定刷新策略 |

### View 的核心特性

| 特性 | View | 物理表 |
|---|---|---|
| 存储方式 | 无物理存储，仅保存查询逻辑 | 对应外部存储（Kafka/MySQL/Paimon） |
| 计算时机 | 查询时动态计算 | 数据写入时持久化 |
| 是否可作为 Sink | 不可（只读） | 可以 |
| 生命周期 | 临时视图=会话级；永久视图=元数据级 | 独立，不受会话影响 |

### 多 Sink 引用同一视图的正确做法

```
错误方式：
  ViewA → Sink1
  ViewA → Sink2   (ViewA 被展开两次，结果可能不同)

正确方式：
  ViewA → 物理中间表（Kafka/Paimon）
  物理中间表 → Sink1
  物理中间表 → Sink2

或使用 StatementSet（INSERT OVERWRITE），在同一执行计划内共享 ViewA
```

### Flink SQL 状态管理核心原则

```sql
-- 窗口计算：状态随窗口自动清理
SELECT window_start, window_end, COUNT(*)
FROM TABLE(TUMBLE(TABLE orders, DESCRIPTOR(order_time), INTERVAL '5' MINUTE))
GROUP BY window_start, window_end;

-- 非窗口 Join/聚合：必须设 TTL 防止状态无限增长
SET 'table.exec.state.ttl' = '3600000';  -- 1 小时（按业务生命周期设定）

-- RocksDB TTL 压缩（减少物理存储）
-- 通过 Java API 配置 enableTtlCompactionFilter
```

**TTL 设置原则**：
- 不是越短越好：TTL < 业务关联窗口时，状态过早清理导致结果错误
- 不是越长越好：TTL 过大导致状态无限增长，OOM
- 应按**业务生命周期**设定：例如订单关联场景按最长等待时间 + 余量

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| Flink SQL 与 MySQL 行为一致 | Flink SQL 是强类型、三值逻辑、流式执行，与 MySQL/Hive 有多处差异 |
| View 被多处引用时会复用执行结果 | View 每次引用都重新展开，可能产生不同结果 |
| 临时视图可以跨会话使用 | 临时视图是会话级，生产环境必须用永久视图 |
| SQL 语法改了一点，Savepoint 肯定还能用 | 任何 SQL 变更都可能改变底层算子拓扑，状态可能不兼容 |
| 空闲 Kafka 分区不影响其他分区处理 | 空闲分区会阻塞全局 Watermark，导致所有窗口卡住不触发 |
| 双流 Join 无需特别处理状态 | 普通双流 Join 状态永久增长，必须限定时间范围或设 TTL |
| NULL 值在 JOIN 条件中等于空字符串 | NULL = 任何值的结果都是 UNKNOWN，不是 TRUE，记录会被排除 |

## 待验证缺口
- 空闲分区阻塞 Watermark 的边界：部分分区空闲 vs 全部分区空闲的行为差异
- State TTL 设置后，RocksDB 的实际清理时机（是 TTL 到期立即清理还是延迟）
- StatementSet 对多 Sink 引用同一视图的优化效果
- Flink 1.20 物化视图的刷新策略配置和适用场景
