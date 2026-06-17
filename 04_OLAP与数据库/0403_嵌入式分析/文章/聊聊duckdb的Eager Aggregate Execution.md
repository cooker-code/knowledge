---
title: 聊聊duckdb的Eager Aggregate Execution
author: Hello大数据
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5MDE0MDE1OQ==&mid=2247484285&idx=1&sn=89b815f411c11b4f450396592f19a6b9&chksm=91d202657ccf28ae2f33a1b54acbb580b191dc2278bb366685e864c67e2262f9da00035ce36d&mpshare=1&scene=24&srcid=0412TYaJReltX4Avfy6EZbq1&sharer_shareinfo=08e5428f488d380d730e1aa50e3b6b2f&sharer_shareinfo_first=08e5428f488d380d730e1aa50e3b6b2f#rd
---

## 一、Eager Aggregate Execution 是什么？

**Eager Aggregate Execution** 是 DuckDB 在**逻辑优化阶段**做的一种改写：  
当满足一定条件时，**不再保留「扫表 + 聚合」的计划**，而是**在优化时就用表的分区/行组统计（Partition Statistics）直接算出**`COUNT(*)`、`MIN(col)`、`MAX(col)` 的结果，把整棵聚合子树替换成一个**只输出这些常数的逻辑算子**（`LogicalExpressionGet`）。  
执行时就不再扫表、也不再跑聚合算子，等价于「在优化器里提前把聚合执行掉」。

所以：

* • **Eager** = 在优化阶段就执行（eager），而不是等到执行阶段。
* • **Aggregate Execution** = 用统计信息“执行”聚合，得到与真正扫表聚合相同的结果。

---

## 二、在优化流水线中的位置

触发点在**统计传播（Statistics Propagation）** 里，处理完 `LogicalAggregate` 后：

```
    aggr.distinct_validity = distinct_validity;  
  
    // after we propagate statistics - try to directly execute aggregates using statistics  
    TryExecuteAggregates(aggr, node_ptr);
```

也就是说：

* • 先对聚合做常规的统计传播（group\_stats、表达式统计等）；
* • 然后调用 `TryExecuteAggregates(aggr, node_ptr)`；
* • 若成功，`node_ptr` 会被替换成新的逻辑算子（见下文），原来的 `LogicalAggregate` 及其子计划不再保留。

---

## 三、触发条件（TryExecuteAggregates 的前置检查）

只有**全部**满足时才会做 Eager Aggregate，否则保持原计划。源码里是“有一项不满足就 return”的写法。

### 1. 聚合形态

* • **无 GROUP BY**：`aggr.groups.empty()`，即全局聚合（整表或过滤后整表只出一行）。
* • **聚合表达式只能是**：

+ • `COUNT(*)`（count\_star）
+ • `MIN(col)` / `MAX(col)`，且参数是**单列**（`BOUND_COLUMN_REF`），不能是复杂表达式。

* • 聚合**不能带 FILTER**（如 `COUNT(*) FILTER (WHERE ...)` 会直接 return）。
* • MIN/MAX 的列类型必须支持统计里的 min/max（数值、时间、VARCHAR 等），见 `GetComparator(fun_name, col_ref.return_type)`。

### 2. 数据源

* • 聚合的**直接子节点**在跳过若干层 `LOGICAL_PROJECTION` 后，必须是 **`LogicalGet`**（表扫描）。
* • 该 Get 对应的**表函数必须实现 `get_partition_stats`**，能返回 `vector<PartitionStatistics>`。
* • **不能带 sampling**：`get.extra_info.sample_options` 为空（只对“整表”用统计）。
* • MIN/MAX 的列要能映射到存储列（`get.TryGetStorageIndex(...)` 成功），否则无法用分区统计。

### 3. 分区统计可用且可精确使用

* • `get_partition_stats(context, input)` 返回非空。
* • 对 **COUNT(\*)**：每个分区的 `count_type` 必须是 **COUNT\_EXACT**；若有任一分区是 COUNT\_APPROXIMATE，则放弃 Eager。
* • 对 **MIN/MAX**：每个分区的 `partition_row_group->GetColumnStatistics(storage_index)` 要能给出列统计，且 **MinMaxIsExact** 为真（即该分区内 min/max 与真实数据一致，没有未加载的删除或版本信息等导致不精确）。

### 4. 与表过滤（Table Filters）的配合

若 `LogicalGet` 上已经下推了过滤条件（`get.table_filters.filters` 非空），Eager 会**按分区**用统计判断「该分区是否整个被过滤掉或整个保留」：

* • 对每个分区、每个过滤列，用 `filter->CheckStatistics(*column_stats)` 得到：

+ • **FILTER\_ALWAYS\_FALSE**：该分区内所有行都不满足 → 分区直接丢弃，不参与 COUNT/MIN/MAX。
+ • **FILTER\_ALWAYS\_TRUE**：该分区内所有行都满足 → 分区完整保留，继续用该分区的统计做 Eager。
+ • **其他（如 NO\_PRUNING\_POSSIBLE / SOMETIMES）**：无法仅凭统计断定整分区全删或全留 → **整条 Eager 优化放弃**（return），退回到普通扫表 + 聚合。

注释里说的就是这一点：

```
    // we can keep execute eager aggregate if all partitions could be either filtered entirely or remained entirely  
    // ...  
                // all filters passed - this partition should keep execute eager aggregate
```

即：**只有“每个分区要么整块被过滤掉，要么整块保留”时，才继续做 Eager**；一旦出现“部分满足”的分区，就不能再只靠统计得到正确结果。

---

## 四、具体怎么做（用统计“执行”聚合）

在通过所有检查后，逻辑如下。

### 1. 先处理过滤后的分区列表

* • 若有 table filters，按上面规则筛掉「整分区为 FILTER\_ALWAYS\_FALSE」的分区，得到 `remaining_partition_stats`（或等价地更新 `partition_stats`）。
* • 若无 filter，就使用全部 `partition_stats`。

### 2. COUNT(\*)

* • 对所有保留分区的 `count` 求和。
* • 若之前已发现任一分区是 COUNT\_APPROXIMATE，则不会走到这里（已 return）。
* • 结果是一个常量 `Value::BIGINT(count)`，对应一个 `BoundConstantExpression`。

### 3. MIN(col) / MAX(col)

* • 对每个分区：`TryGetValueFromStats(partition_stats[i], storage_index, comparator, result)`

+ • 通过 `partition_row_group->GetColumnStatistics(storage_index)` 取列统计；
+ • 要求 `MinMaxIsExact` 为真，并取该分区的 min 或 max（由 comparator 决定）。

* • 对所有分区得到的 min（或 max）再求**全局 min 或 max**，得到最终常量。
* • 每个 MIN/MAX 对应一个 `BoundConstantExpression`。

### 4. 替换计划

* • 把这些常量按原聚合表达式的顺序和别名组成一行结果，构造成 `LogicalExpressionGet(aggregate_index, types, expressions)`，其中 `expressions` 是 `vector<vector<unique_ptr<Expression>>>`，这里只有一行，即多个 `BoundConstantExpression`。
* • 子节点放一个 **`LogicalDummyScan`**（不扫任何表），表示“没有真实数据源，结果就是这些常量”：

```
    vector<vector<unique_ptr<Expression>>> expressions;  
    expressions.push_back(std::move(agg_results));  
    auto expression_get =  
        make_uniq<LogicalExpressionGet>(aggr.aggregate_index, std::move(types), std::move(expressions));  
    expression_get->children.push_back(make_uniq<LogicalDummyScan>(aggr.group_index));  
    node_ptr = std::move(expression_get);
```

也就是说：**原来的「Filter → Get → Aggregate」整棵子树被替换成「DummyScan → ExpressionGet(常量行)」**。

---

## 五、分区统计从哪里来？

* • **DuckDB 原生表**：`TableScanGetPartitionStats` → `storage.GetPartitionStats(context)` → `RowGroupCollection::GetPartitionStats()`，对每个 row group 调用 `RowGroup::GetPartitionStats(entry)`，得到：

+ • `count`（行数）
+ • `count_type`（COUNT\_EXACT 或 COUNT\_APPROXIMATE，取决于是否有未加载的 delete/版本信息）
+ • `partition_row_group`：一个 `DuckDBPartitionRowGroup`，提供 `GetColumnStatistics(storage_index)` 和 `MinMaxIsExact(...)`。

* • **Parquet**：Parquet 扫描同样通过 `get_partition_stats` 提供按 row group 的统计，因此 Eager 也可用于 Parquet（见 `eager_aggregate_execution_parquet.test`）。

`MinMaxIsExact` 为 true 的条件（DuckDB 行组）：  
无未加载的 delete、无版本信息，且数值列有 min/max 统计；字符串列还要满足长度与 min/max 一致等条件，这样才敢说分区内的 min/max 就是真实 min/max。

---

## 六、执行层：从 LogicalExpressionGet 到“零扫表、零聚合”

* • `LogicalExpressionGet` 在物理计划里会生成 **PhysicalExpressionScan**，其表达式就是上面那行常量。
* • 若这些表达式是**可折叠的**（无子查询、无参数等），`PhysicalPlanGenerator::CreatePlan(LogicalExpressionGet)` 会**在生成计划时就把表达式求值掉**，把结果放进一个小的 `ColumnDataCollection`，最终计划变成 **PhysicalColumnDataScan** 扫这个只有一行的结果集。
* • 因此执行时：**没有表扫描、没有聚合算子**，只是读一条预先算好的行。测试里用 `EXPLAIN` 检查计划中不出现 `UNGROUPED_AGGREGATE`、或出现 `AGGREGATE` 等，就是在验证是否走了 Eager 或普通聚合。

---

## 七、举例

以duckdb的iceberg插件为例，看下具体的实现

```
void IcebergMultiFileList::GetStatistics(vector<PartitionStatistics> &result) const {  
    if (GetMetadata().iceberg_version == 1) {  
        //! We collect no statistics information from manifests for V1 tables.  
        return;  
    }  
  
    if (!transaction_delete_manifests.empty() || !delete_manifests.empty()) {  
        //! if exist delete_manifests , return;  
        return;  
    }  
  
    idx_t count = 0;  
    for (idx_t i = 0; i < data_manifests.size(); i++) {  
        count += data_manifests[i].existing_rows_count;  
        count += data_manifests[i].added_rows_count;  
    }  
  
    for (idx_t i = 0; i < transaction_data_manifests.size(); i++) {  
        auto files = transaction_data_manifests[i].get().entries;  
        for (idx_t j = 0; j < files.size(); j++) {  
            count += files[j].data_file.record_count;  
        }  
    }  
  
    PartitionStatistics partition_stats;  
    partition_stats.count = count;  
    partition_stats.count_type = CountType::COUNT_EXACT;  
    result.push_back(partition_stats);  
}
```

核心操作就是从iceberg的元数中拿到相关的统计信息进行加和操作。

这个也是当时我们在实际查询过程中，发现当数据量大了之后，查询速度明显非常慢，于是就提了这个pr： https://github.com/duckdb/duckdb-iceberg/pull/567

## 八、小结

| 维度 | 内容 |
| --- | --- |
| **含义** | 在优化阶段用分区/行组统计直接算出 COUNT(\*)/MIN/MAX，用常量结果替换整棵聚合子树，执行时不再扫表、不再聚合。 |
| **触发位置** | 统计传播中，处理完 `LogicalAggregate` 后调用 `TryExecuteAggregates`。 |
| **聚合形式** | 仅无 GROUP BY 的全局聚合，且仅允许 COUNT(\*)、MIN(col)、MAX(col)，无 FILTER。 |
| **数据源** | 子节点（剥掉 Projection）为 LogicalGet，且支持 `get_partition_stats`，无 sampling。 |
| **统计要求** | 分区 count 为精确；MIN/MAX 列在分区上 MinMaxIsExact。 |
| **与过滤** | 若有表过滤，每个分区必须能判定为「整分区删除」或「整分区保留」，否则不做 Eager。 |
| **计划形态** | 原子树被替换为 `LogicalDummyScan` + `LogicalExpressionGet(常量行)`，物理层再折叠为对一行结果集的扫描。 |

所以 **Eager Aggregate Execution** 的本质是：**在优化器里利用存储层的分区/行组统计，把“扫表+聚合”收缩成“只读一行常量”**，从而在满足条件时实现零 I/O、零聚合执行。