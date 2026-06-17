---
title: DuckDB：事务与 MVCC—— 嵌入式数据库也能有完整的 ACID 保证
author: MaxAiDB
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUwOTU4OTU2NQ==&mid=2247483715&idx=1&sn=69e8180cdf2cec0f33d659e79ef1143a&chksm=f8b0f36bdf37f7b71d1642fbbd9ecc1dc1c95a15941ba1b20500dd9216ccf4def11851a09b26&mpshare=1&scene=24&srcid=0411q32KcIr9svKtQEwFOKAE&sharer_shareinfo=0a5304ac841612ea1999482a2fb0be7e&sharer_shareinfo_first=0a5304ac841612ea1999482a2fb0be7e#rd
---

—— 嵌入式数据库也能有完整的 ACID 保证

> 本文是《DuckDB：从上手到内核》系列的第 11 篇。前面的文章都在讲"怎么快"——列式存储、向量化执行、并行调度。但数据库不只要快，还要**对**。今天我们来聊一个"正确性"话题：事务。

---

## 你在分析数据时，另一个线程正在写入——数据会乱吗？

假设你正在用 DuckDB 跑一个报表查询，耗时 5 秒。与此同时，另一个线程正在往同一张表里插入新数据。你的报表会不会读到"插了一半"的数据？查出来一个不一致的结果？

答案是：**不会。** DuckDB 支持完整的 ACID 事务，每个查询看到的都是数据的一个一致"快照"。

这对一个嵌入式数据库来说，其实相当不容易——SQLite 长期被诟病只支持表级锁，并发性能差。DuckDB 从一开始就设计了 MVCC（多版本并发控制），实现了比 SQLite 更好的并发支持。

---

## ACID 四大属性：DuckDB 怎么实现的？

先快速回顾一下 ACID——这是数据库事务的四个基本保证：

| 属性 | 含义 | DuckDB 实现方式 |
| --- | --- | --- |
| **A** tomicity（原子性） | 事务要么全成功，要么全失败 | UndoBuffer + WAL |
| **C** onsistency（一致性） | 数据库始终处于合法状态 | Binder 语义检查 + 约束验证 |
| **I** solation（隔离性） | 并发事务互不干扰 | MVCC 快照隔离 |
| **D** urability（持久性） | 提交后的数据不会丢失 | WAL 先行写入 + Checkpoint |

下面我们一个一个拆开看。

---

## MVCC：每个事务看自己的"快照"

### 通俗理解

MVCC（Multi-Version Concurrency Control，多版本并发控制）的核心思想是：**数据不止有一个版本。** 当你修改一行数据时，不是直接覆盖旧值，而是创建一个新版本，旧版本还保留着。

打个比方：你在用 Google Docs 编辑文档。你的同事打开了同一篇文档，但看到的是你编辑之前的版本。你俩各改各的，互不影响——直到其中一个人"提交"。

```
  事务 A (start_time = 100)        事务 B (start_time = 105)  ─────────────────────            ─────────────────────  开始查询                            读到 price = 100       ←                                             开始事务                                    UPDATE price = 200                                    （创建新版本）  仍然读到 price = 100   ←         price 的版本链:  （看的是 start=100 的快照）         [100, commit=99] ← [200, txn_id=B]  
                                    COMMIT (commit_id = 110)  仍然读到 price = 100                （事务 A 的快照不变）              版本链更新:                                    [100, commit=99] ← [200, commit=110]  
  图 1：MVCC 快照隔离示意图
```

### 版本链：UpdateInfo

DuckDB 用 **UpdateInfo** 数据结构来管理版本链（`src/include/duckdb/transaction/update_info.hpp`）。每次 UPDATE 操作都会创建一个 `UpdateInfo` 节点，通过 `prev` / `next` 指针形成双向链表。

关键的可见性判断逻辑只有一行：

```
bool AppliesToTransaction(transaction_t start_time, transaction_t transaction_id) {    return version_number > start_time && version_number != transaction_id;}
```

翻译成白话：**如果这个版本是在我的事务开始之后才写入的（不管是否已提交），而且不是我自己写的——那我就需要使用这个版本保存的旧值。**

### 时间戳设计

DuckDB 巧妙地用了两种 ID：

•**transaction\_id**：从一个很大的数字开始（`TRANSACTION_ID_START`），标识未提交的事务•**commit\_id**：从 2 开始递增的小数字，标识已提交事务的时间点

因为 `transaction_id >> commit_id`，一个版本号就能区分"这个数据是已提交的还是正在修改中的"——不需要额外的标志位。

---

## UndoBuffer：回滚的后悔药

每个事务都有一个 **UndoBuffer**——它记录了事务期间所有修改操作的"旧值"。如果事务需要回滚，就从 UndoBuffer 里把旧值恢复回去。

UndoBuffer 支持六种条目类型：

| 条目类型 | 说明 |
| --- | --- |
| INSERT\_TUPLE | 记录插入操作（回滚时删除） |
| DELETE\_TUPLE | 记录删除操作（回滚时恢复） |
| UPDATE\_TUPLE | 记录更新操作（回滚时恢复旧值） |
| CATALOG\_ENTRY | 记录 DDL 操作（CREATE/DROP 表等） |
| SEQUENCE\_VALUE | 记录序列值变更 |
| ATTACHED\_DATABASE | 记录数据库附加操作 |

**回滚时最关键的一点：必须反向遍历 UndoBuffer。** 因为后面的操作可能依赖前面操作的结果。想象你先插入一行，再更新它——回滚时要先撤销更新，再撤销插入，顺序不能反。

---

## WAL：先记日志，再改数据

WAL（Write-Ahead Log，预写日志）是持久性的保障。它的核心原则很简单：**在修改数据之前，先把这次修改记到日志里。**

```
  正常写入流程:  
  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐  │ 1. 写入 WAL  │ ──→ │ 2. 修改内存  │ ──→ │ 3. 返回成功  │  │ (落盘)       │     │ 中的数据     │     │             │  └─────────────┘     └─────────────┘     └─────────────┘  
  崩溃恢复流程:  
  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐  │ 1. 加载最近   │ ──→ │ 2. 发现有    │ ──→ │ 3. 重放 WAL   │  │ checkpoint   │     │ WAL 文件     │     │ 恢复数据     │  └──────────────┘     └──────────────┘     └──────────────┘  
  图 2：WAL + Checkpoint 协作流程图
```

DuckDB 的 WAL 支持记录各种操作——DDL（建表、删表、改表）、DML（插入、删除、更新）、序列值变更等。WAL 有两个版本号：标准版（v2）和加密版（v3）。

### 大事务的优化

如果一个事务插入了几百万行数据，写 WAL 会产生很大的文件。DuckDB 有个聪明的优化：**当事务数据量足够大时，跳过 WAL 写入，直接触发 Checkpoint。** 这避免了"先写 WAL，Checkpoint 时再写一遍数据文件"的双重写入开销。

---

## Checkpoint：把内存里的数据刷到磁盘

WAL 是一个不断增长的日志文件。如果永远不清理，它会越来越大，启动恢复的时间也越来越长。Checkpoint 就是来解决这个问题的——它把内存中所有已提交的数据写入主数据文件，然后截断 WAL。

DuckDB 的 Checkpoint 触发条件（在事务提交时自动检查）：

•WAL 文件大小超过阈值•不是只读事务•能获取到 Checkpoint 独占锁

DuckDB 还支持**并发 Checkpoint**——如果有其他只读事务正在运行，可以同时进行 Checkpoint，不需要等它们结束。但如果当前事务包含 UPDATE 或 DROP 操作，就必须等其他事务结束后才能 Checkpoint。

---

## 事务的完整生命周期

```
  ┌─────────────┐  │  BEGIN       │  分配 start_time 和 transaction_id  │  (开始事务)   │  创建空的 UndoBuffer + LocalStorage  └──────┬──────┘         │  ┌──────▼──────┐  │  执行 SQL    │  INSERT → LocalStorage 缓存  │              │  DELETE → RowVersionManager 标记  │              │  UPDATE → 创建 UpdateInfo 版本链  │              │  每个操作都在 UndoBuffer 中记录  └──────┬──────┘         │    ┌────┴────┐    │         │┌───▼───┐ ┌──▼────┐│COMMIT │ │ROLLBACK│└───┬───┘ └───┬───┘    │         │    ▼         ▼  写入WAL   反向遍历  分配      UndoBuffer  commit_id 恢复旧值  更新版本号 丢弃  WAL 落盘  LocalStorage  清理旧版本  (可能触发 Checkpoint)  
  图 3：事务提交/回滚的状态机
```

提交流程中有个精巧的设计：**WAL 写入期间会释放事务锁**，让只读事务可以并发启动和提交。这提高了整体吞吐量。

---

## 单写多读：DuckDB 的并发模型

DuckDB 采用 **"单写多读"** 模型。这意味着：

•**多个读事务可以并发执行**——每个读事务都看到一个一致的快照•**同一时间只能有一个写事务在提交**——通过 WAL 锁保证

这和 SQLite 的 WAL 模式类似，但 DuckDB 的隔离级别更高（快照隔离 vs SQLite 的读提交）。

| 操作组合 | 是否允许 | 说明 |
| --- | --- | --- |
| 多个读 + 0 个写 | ✅ | 完全并发 |
| 多个读 + 1 个写 | ✅ | 读事务看快照，不受写影响 |
| 0 个读 + 多个写 | ⚠️ | 写事务可以并发执行，但提交时串行化 |
| Checkpoint + 读 | ✅ | 并发 Checkpoint 支持 |
| Checkpoint + 写 | ❌ | Checkpoint 需要独占锁 |

对于大多数 OLAP 场景，这个模型是完全够用的——分析查询通常是读密集的，偶尔有 ETL 任务写入数据。

---

## 动手实验

### 实验 1：验证快照隔离

用 Python 多线程模拟并发读写：

```
import duckdbimport threadingimport time  
# 创建持久化数据库con = duckdb.connect('test_mvcc.duckdb')con.execute("CREATE TABLE IF NOT EXISTS counter (id INTEGER, value INTEGER)")con.execute("INSERT INTO counter VALUES (1, 100)")con.close()  
def writer():    """写线程：2 秒后把 value 改成 999"""    time.sleep(2)    wcon = duckdb.connect('test_mvcc.duckdb')    wcon.execute("UPDATE counter SET value = 999 WHERE id = 1")    print(f"[Writer] Updated value to 999 at {time.time():.1f}")    wcon.close()  
def reader():    """读线程：每秒读一次，持续 5 秒"""    rcon = duckdb.connect('test_mvcc.duckdb')    for i in range(5):        result = rcon.execute("SELECT value FROM counter WHERE id = 1").fetchone()        print(f"[Reader] t={i}s, value = {result[0]}")        time.sleep(1)    rcon.close()  
t1 = threading.Thread(target=reader)t2 = threading.Thread(target=writer)t1.start()t2.start()t1.join()t2.join()
```

预期输出：Reader 在 Writer 更新之前看到 100，更新之后看到 999——但如果 Reader 在一个事务内，它应该始终看到同一个值。

### 实验 2：WAL 和 Checkpoint

```
-- 创建数据库并写入数据-- duckdb wal_test.duckdb  
CREATE TABLE data AS SELECT range AS id, random() * 100 AS value FROM range(100000);  
-- 查看 WAL 文件大小-- ls -la wal_test.duckdb.wal  
-- 强制 CheckpointCHECKPOINT;  
-- WAL 文件应该被截断（大小变为 0 或消失）-- ls -la wal_test.duckdb.wal  
-- 再插入一些数据INSERT INTO data SELECT range + 100000 AS id, random() * 100 AS value FROM range(100000);  
-- WAL 文件又有内容了-- ls -la wal_test.duckdb.wal  
-- 验证行数SELECT COUNT(*) FROM data;  -- 应该是 200000
```

---

## 和 SQLite 的对比

既然 DuckDB 经常被拿来和 SQLite 比较，这里做个事务层面的对比：

| 维度 | DuckDB | SQLite (WAL 模式) |
| --- | --- | --- |
| 隔离级别 | 快照隔离 | 读提交 |
| 并发读 | 多读并发，无锁 | 多读并发，无锁 |
| 并发写 | 单写，提交时串行 | 单写，写入时串行 |
| 版本管理 | MVCC（UndoBuffer + UpdateInfo 版本链） | Copy-on-Write（WAL 中保存旧页） |
| 锁粒度 | 行级版本控制 | 数据库级锁 |
| Checkpoint | 自动触发 + 手动 `CHECKPOINT` | 自动触发 + 手动 `PRAGMA wal_checkpoint` |

DuckDB 的主要优势在于**行级版本控制**——SQLite 在写入时会锁住整个数据库文件，而 DuckDB 只在提交时串行化，执行期间读写互不阻塞。

---

## 小结

1.**DuckDB 支持完整的 ACID 事务**，不是一个"凑合用"的嵌入式数据库。2.**MVCC（快照隔离）** 让并发读写互不干扰——每个事务看到一致的数据快照。3.**UndoBuffer** 记录所有修改操作的旧值，回滚时反向遍历恢复。4.**WAL（预写日志）** 保证持久性——先记日志再改数据，崩溃后重放日志恢复。5.**Checkpoint** 定期把数据刷入主文件并截断 WAL，防止 WAL 无限增长。6.**单写多读**是 DuckDB 的并发模型——对 OLAP 读密集场景完全够用。

---

**Q1：什么是"快照隔离"？和"读提交"有什么区别？**

"快照隔离"就像给数据库在你的事务开始时拍了一张照片。不管其他人后来怎么改数据，你看到的永远是那张照片。

"读提交"每次读数据都看最新已提交的版本。如果你的查询跑 5 秒，第 1 秒读到 A=100，第 3 秒别人把 A 改成 200 并提交了，你第 4 秒再读 A 就变成 200 了——同一个查询内看到了不一致的数据。

快照隔离更安全，但需要更多的版本管理开销。DuckDB 选择了快照隔离。

**Q2：DuckDB 的事务有什么限制吗？**

主要限制是**写事务不能完全并行提交**——提交时需要串行化。对于 OLAP 场景（主要是大批量写入 + 大量读查询），这不是问题。但如果你需要每秒几千个小写事务（典型 OLTP），DuckDB 不适合——用 PostgreSQL 或 MySQL。

**Q3：内存数据库（`:memory:`）也有事务保证吗？**

有！MVCC 和快照隔离在内存模式下完全一样。区别是没有 WAL 和 Checkpoint——因为没有磁盘需要持久化。一旦进程退出，数据就没了。

---

## 延伸阅读

**源码路径：**

•事务核心：`src/transaction/duck_transaction.cpp`•事务管理器：`src/transaction/duck_transaction_manager.cpp`•UndoBuffer：`src/transaction/undo_buffer.cpp`•UpdateInfo 版本链：`src/include/duckdb/transaction/update_info.hpp`•WAL 写入：`src/transaction/wal_write_state.cpp`•WAL 重放：`src/storage/wal_replay.cpp`•Checkpoint：`src/include/duckdb/storage/checkpoint_manager.hpp`•LocalStorage：`src/include/duckdb/transaction/local_storage.hpp`

**参考资料：**

•DuckDB 官方文档 - 事务：https://duckdb.org/docs/sql/statements/transactions[1]•MVCC 经典论文：「An Empirical Evaluation of In-Memory Multi-Version Concurrency Control」, VLDB 2015

---

### References

`[1]`: *https://duckdb.org/docs/sql/statements/transactions*