> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/030302_核心知识点/FlinkCDCMySQLSourceEnumerator表发现与全增量切换|FlinkCDCMySQLSourceEnumerator表发现与全增量切换]]
---
title: flink-cdc 3.4.0笔记 MySqlSourceEnumerator是如何发现需要处理的数据库表的
author: 源码涓流
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0NjYzNzI5Mw==&mid=2247484943&idx=1&sn=7cf2e1647a7ce9070a2d659da6f10b7d&chksm=c2ff13f84d7f6aae3267995670b44b53d6617da9edbff1e6251e089a818a117710b2df5da6ae&mpshare=1&scene=24&srcid=0901jdv3ZE86XofIRFrCmRgd&sharer_shareinfo=968b0fdff1eb7fd863ed5f88ea5b5138&sharer_shareinfo_first=968b0fdff1eb7fd863ed5f88ea5b5138#rd
---

# MySqlSourceEnumerator的启动流程

当flink-cdc任务提交到flink集群后，集群在jobmanager节点上给这个任务创建一个JobMaster对象，JobMaster对象负责管理flink任务的生命周期和工作分配。

JobMaster对象从任务中取出数据源相关信息，创建一个SourceCoordinator对象。SourceCoordinator对象调用org.apache.flink.cdc.connectors.mysql.source.enumerator.MySqlSourceEnumerator#start方法启动MySqlSourceEnumerator。

主要干了如下几件事

初始化分片分配器，使其能够开始向 Source Reader 提供数据分片。

如果当前任务已经完成了快照阶段，或者它是从 Binlog 阶段的检查点恢复的，那么它可能就需要立即请求 Binlog 分片更新。它是一个条件性操作。通常，当一个 Flink CDC Job 启动时，它首先需要读取数据库的历史数据快照（snapshot）。只有当快照数据读取完成后，才开始读取增量数据（binlog）。

查询 Flink 运行时，获取当前所有已经成功注册到 Coordinator 的 Source Reader 的信息（例如它们的 ID、状态等）。

定期检查和同步 Source Reader 状态

# 初始化MySqlHybridSplitAssigner

## 发现需要捕获的数据库表

调用org.apache.flink.cdc.connectors.mysql.source.assigners.MySqlSnapshotSplitAssigner#discoveryCaptureTables方法发现需要捕获的数据库表。

实际工作委托org.apache.flink.cdc.connectors.mysql.source.utils.TableDiscoveryUtils#listTables方法完成。

执行如下SQL语句获取数据库列表及每个数据库下的表列表

```
SHOW DATABASES;SHOW FULL TABLES IN `employees` where Table_Type = 'BASE TABLE';SHOW FULL TABLES IN `information_schema` where Table_Type = 'BASE TABLE';SHOW FULL TABLES IN `mysql` where Table_Type = 'BASE TABLE';SHOW FULL TABLES IN `performance_schema` where Table_Type = 'BASE TABLE';SHOW FULL TABLES IN `sys` where Table_Type = 'BASE TABLE';
```

获取全部数据库输出结果

```
mysql> SHOW DATABASES;+--------------------+| Database           |+--------------------+| employees          || information_schema || mysql              || performance_schema || sys                |+--------------------+5 rows in set (0.01 sec)
```

获取指定数据库的表输出结果

```
mysql> SHOW FULL TABLES IN `employees` where Table_Type = 'BASE TABLE';+---------------------+------------+| Tables_in_employees | Table_type |+---------------------+------------+| departments         | BASE TABLE || dept_emp            | BASE TABLE || dept_manager        | BASE TABLE || employees           | BASE TABLE || salaries            | BASE TABLE || titles              | BASE TABLE |+---------------------+------------+6 rows in set (0.07 sec)
```

每个表经过如下函数过滤检查，满足条件的放到final List capturedTableIds = new ArrayList<>();中

```
public Predicate<TableId> getTableFilter() {        RelationalTableFilters tableFilters = dbzMySqlConfig.getTableFilters();        return tableId -> tableFilters.dataCollectionFilter().isIncluded(tableId);    }
```

发现的表存储到org.apache.flink.cdc.connectors.mysql.source.assigners.MySqlSnapshotSplitAssigner#remainingTables中

## 发现可能新增的表

立即调用org.apache.flink.cdc.connectors.mysql.source.assigners.MySqlSnapshotSplitAssigner#captureNewlyAddedTables一次，发现可能新增的表

这是有条件的执行，只有当启用了动态表扫描、不是纯快照模式，并且已经完成（或绕过了）快照阶段时，才会执行后续的表检测逻辑。条件表达式如下图所示

还是调用org.apache.flink.cdc.connectors.mysql.debezium.DebeziumUtils#discoverCapturedTables方法获取满足条件的全部表列表，存储到currentCapturedTables中。

获取 Flink CDC 内部当前已知的、正在捕获的表列表，存储到previousCapturedTables中。

计算被移除的表，求差集：previousCapturedTables - currentCapturedTables = 被移除的表，存储到tablesToRemove中。

计算被新增的表，求差集：差集：currentCapturedTables - previousCapturedTables = 新添加的表，存储到newlyAddedTables中。

### 处理被移除的表

遍历 Flink CDC 内部所有与这些被移除表相关的状态，包括：

清除assignedSplits：已分配给 Reader 但可能尚未完成处理的快照分片。

清除splitFinishedOffsets：这些分片的处理偏移量。

清除tableSchemas：这些表的 Schema 信息缓存。

清除remainingSplits：尚未分配出去的快照分片。

清除remainingTables：等待处理的表列表。

清除alreadyProcessedTables：已经处理过的表列表。

通过将这些相关状态清理掉，确保 Flink CDC 不会再尝试捕获这些已经不存在或不再符合规则的表。

### 处理新添加的表

remainingTables.addAll(newlyAddedTables);: 将这些新表添加到 remainingTables 列表中。这意味着这些新表现在被 Flink CDC 视为需要处理的对象。

if (AssignerStatus.isAssigningFinished(assignerStatus)): 再次检查分片分配器的状态。

新添加的表会进入 remainingTables 列表，等待被分配给 Reader 进行处理。

如果已经处于 Binlog 读取阶段 (isAssigningFinished 为 true)，那么会立即调用 this.startAssignNewlyAddedTables() 方法。这个方法会触发为这些新表生成快照分片（如果需要的话）以及开始捕获其 Binlog 的过程。

如果处于INITIAL\_ASSIGNING\_FINISHED状态，切换到NEW\_ADDED\_ASSIGNING状态

如果处于NEWLY\_ADDED\_ASSIGNING\_FINISHED状态，切换到NEWLY\_ADDED\_ASSIGNING状态

## 启动异步分割流程，将表的快照读取任务分割成多个表的块读取任务

org.apache.flink.cdc.connectors.mysql.source.assigners.MySqlSnapshotSplitAssigner#startAsynchronouslySplit

满足条件才会提交分裂任务org.apache.flink.cdc.connectors.mysql.source.assigners.MySqlSnapshotSplitAssigner#splitChunksForRemainingTables到线程池org.apache.flink.cdc.connectors.mysql.source.assigners.MySqlSnapshotSplitAssigner#executor运行，这个线程池只有一个线程，名称为snapshot-splitting。

条件表达式如下图所示，满足两个条件中的一个即可

条件一，currentSplittingTableId不为空

条件二，remainingTables不为空

## org.apache.flink.cdc.connectors.mysql.source.assigners.MySqlSnapshotSplitAssigner#assignerStatus状态

MySqlSnapshotSplitAssigner持有的状态流转如下图所示