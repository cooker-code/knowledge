---
title: DuckDB:从使用到内核——我们学到了什么？—— DuckDB 的设计哲学与启示
author: MaxAiDB
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUwOTU4OTU2NQ==&mid=2247483726&idx=1&sn=cdc2a6abff28fb3abc599e695e30f0d4&chksm=f8f4968f236d47832b298136bcfb87e02a39d49624f6ed3bdc46894e50907b59f8450ded0eab&mpshare=1&scene=24&srcid=0416JGShcMqVEpGgZF5JMXhk&sharer_shareinfo=eb7f7e2124ce4eb71e26987f1d66b26a&sharer_shareinfo_first=eb7f7e2124ce4eb71e26987f1d66b26a#rd
---

—— DuckDB 的设计哲学与启示

> 本文是《DuckDB：从上手到内核》系列的第 16 篇，也是最后一篇。15 篇文章走下来，我们从一行 `pip install duckdb` 开始，一路看到了向量化执行引擎、列式存储、35 个优化器 Pass、16 种压缩算法、Morsel-Driven 并行调度……这篇不再拆代码，而是退一步，聊聊 DuckDB 给我们的启示。

---

## "嵌入式分析"这股风，是怎么吹起来的？

过去十年，数据分析的主流叙事是：**把数据送上云端，用分布式系统处理。** Hadoop、Spark、Snowflake、BigQuery——越大越好，越分布越强。

然后一些尴尬的事实浮出水面：

•你 80% 的分析任务，数据不超过 100GB•你的 Spark 集群大部分时间 CPU 利用率不到 20%•从写 SQL 到看到结果，中间隔了"提交作业→等调度→启动容器→shuffle→写中间文件→……"•一个简单的 `GROUP BY` 查询，本地 Pandas 跑 3 秒，上了 Spark 反而要 30 秒（启动开销）

**DuckDB 的出现，是对"万物上云、万物分布式"的一次反思。** 它的核心主张很简单：

> 如果你的数据能放进一台机器的内存（或者不太超出），那你不需要一个分布式系统。你需要的是一个足够快的单机引擎。

这不是倒退，这是回归理性。

---

## DuckDB 的三个成功要素

### 1. 简单

DuckDB 的安装：

```
pip install duckdb
```

没有服务器、没有配置文件、没有端口、没有认证、没有 Docker、没有 YAML。一行命令，下一行就能写 SQL。

对比一下其他"嵌入式"方案：

•SQLite：也很简单，但不适合分析查询•ClickHouse Local：功能强但安装包大（200MB+），API 不如 DuckDB 友好•Apache DataFusion：Rust 库，需要编程才能用

**DuckDB 的简单不是功能缺失，而是接口设计上的克制。** 它把复杂度藏在了引擎内部——你不需要知道什么是向量化执行，也不需要理解 Zone Map，只要写 SQL 就够了。

### 2. 快

我们在这个系列中拆解了"快"的每一层：

| 层次 | 技术 | 系列文章 |
| --- | --- | --- |
| 存储 | 列式存储 + 16 种自动压缩 | 第 5、8 篇 |
| 扫描 | Zone Map 跳过 + 列裁剪 | 第 5、14 篇 |
| 执行 | 向量化（2048 行/批）+ Push-Based Pipeline | 第 6 篇 |
| 优化 | 35 个优化器 Pass + DPccp Join Order | 第 7 篇 |
| 聚合 | Radix 分区 Hash 表 + Perfect Hash | 第 9 篇 |
| 并行 | Morsel-Driven + 无锁任务队列 | 第 10 篇 |
| 数据交换 | Arrow 零拷贝 | 第 13 篇 |

这些技术单独看都不是 DuckDB 独创的（向量化来自 MonetDB/X100，Morsel-Driven 来自 TUM 的论文），但 DuckDB 把它们**整合在一个嵌入式引擎里**——这是工程能力的体现。

### 3. 专注

DuckDB 很清楚自己**不做什么**：

•❌ 不做分布式（MotherDuck 做这件事）•❌ 不做高并发 OLTP（那是 PostgreSQL/MySQL 的领域）•❌ 不做流处理（那是 Flink/Kafka 的领域）•❌ 不做全文搜索（虽然有 fts 扩展，但不是核心竞争力）•❌ 不做机器学习（虽然能跑 SQL-based ML，但不是重点）

这种"不做什么"的判断力，比"做什么"更难。很多项目死在了功能膨胀上——什么都想做，什么都做不好。

---

## 与 SQLite 的设计哲学对比

DuckDB 经常被称为"SQLite for Analytics"。这个类比既准确又有局限。

| 维度 | SQLite | DuckDB |
| --- | --- | --- |
| 定位 | 嵌入式 OLTP | 嵌入式 OLAP |
| 存储 | 行式（B-Tree） | 列式（RowGroup + Segment） |
| 执行 | 逐行处理 | 向量化（2048 行/批） |
| 并发 | 单写多读（WAL 模式） | 单写多读（MVCC + WAL） |
| 安装 | 零依赖 | 零依赖 |
| SQL 兼容 | PostgreSQL 子集 | PostgreSQL 超集 |
| 扩展 | 有限 | 丰富（INSTALL/LOAD） |
| 代码量 | ~15 万行 C | ~30 万行 C++ |
| 代码分发 | 单文件（sqlite3.c） | CMake 多文件 |

**两者共同的设计哲学**：

1.**嵌入式**：库，不是服务2.**零依赖**：不需要安装数据库服务器3.**单文件数据库**：数据存在一个文件里，拷贝即备份4.**零配置**：打开就能用

**关键区别**：面向的工作负载完全不同。SQLite 擅长的是"一个用户的 App 数据"（通讯录、消息记录），DuckDB 擅长的是"一堆数据的分析查询"（聚合、JOIN、窗口函数）。

**DuckDB 从 SQLite 学到的最重要的东西不是技术，而是理念：** 一个数据库可以不需要服务器，可以像库一样嵌入到应用中，可以让用户在 5 秒内从安装到运行第一个查询。

---

## 对数据工程的三个启示

### 启示 1：Local-First（本地优先）

"本地优先"不是说不用云，而是说：**默认在本地处理，只在必要时上云。**

DuckDB 的典型工作流：

```
# 本地开发 & 探索import duckdbduckdb.sql("SELECT * FROM read_parquet('s3://bucket/data.parquet') LIMIT 100")  
# 本地跑完整查询duckdb.sql("""    COPY (        SELECT region, SUM(amount) FROM read_parquet('s3://bucket/*.parquet')        GROUP BY region    ) TO 'result.parquet'""")  
# 只有最终结果才上传到云端
```

这个模式的好处：

•快速迭代（不等调度器）•低成本（不需要集群）•可离线工作•数据安全（敏感数据不离开本机）

### 启示 2：SQL-First（SQL 为先）

DuckDB 证明了：**SQL 作为数据分析的接口，比 DataFrame API 更通用。**

•Pandas 的 API：`df.groupby('region').agg({'amount': 'sum'}).sort_values('amount')`•Polars 的 API：`df.group_by('region').agg(pl.col('amount').sum()).sort('amount')`•DuckDB 的 SQL：`SELECT region, SUM(amount) FROM df GROUP BY region ORDER BY 2`

三种写法做的是同一件事。但 SQL 有几个独特优势：

1.**通用性**：几乎所有数据工具都支持 SQL2.**声明式**：你描述"要什么"，不描述"怎么做"——优化器来决定执行策略3.**可组合**：子查询、CTE、窗口函数——复杂逻辑用 SQL 表达比链式 API 更清晰4.**可优化**：DuckDB 的 35 个优化器能对 SQL 做深度优化，DataFrame API 的优化空间有限

这不是说 DataFrame API 没用——Pandas 做数据清洗很方便，Polars 处理时间序列很高效。但**当你的分析逻辑变复杂时（多表 JOIN、窗口函数、嵌套聚合），SQL 通常更清晰、更快**。

DuckDB 的聪明之处在于：你不需要二选一。`duckdb.sql("SELECT * FROM my_dataframe")` 让你在 SQL 和 DataFrame 之间无缝切换。

### 启示 3：Library, Not Service（库，而非服务）

传统数据库架构：

```
你的应用 → 网络 → 数据库服务器 → 磁盘
```

DuckDB 架构：

```
你的应用 → DuckDB（进程内）→ 磁盘
```

少了一层网络，少了一个独立进程，少了连接池、认证、序列化/反序列化。

这个设计选择的深远影响：

•**部署简单**：pip install 就完事了•**资源控制**：DuckDB 使用你进程的内存，你完全掌控•**无冷启动**：不需要等服务器启动•**可嵌入任何地方**：Python 脚本、Jupyter Notebook、Web 浏览器（WASM）、移动 App

---

## 什么时候该用 DuckDB？什么时候不该？

这大概是整个系列最实用的一张图：

```
你的场景是什么？  │  ├─ 分析查询（聚合、JOIN、窗口函数）？  │     ├─ 数据 < 100GB → ✅ DuckDB（最佳选择）  │     ├─ 数据 100GB-1TB → ✅ DuckDB（需要 SSD + 足够内存）  │     └─ 数据 > 1TB → ⚠️ 考虑 ClickHouse / Snowflake  │  ├─ ETL / 数据转换？  │     ├─ 本地文件处理 → ✅ DuckDB（CSV → Parquet 等）  │     ├─ 定时批处理 → ✅ DuckDB + cron / dbt  │     └─ 实时流处理 → ❌ 用 Flink / Kafka  │  ├─ 数据科学 / 探索？  │     ├─ Jupyter 交互分析 → ✅ DuckDB + Python  │     ├─ 机器学习特征工程 → ✅ DuckDB（SQL 做特征）  │     └─ 模型训练 → ❌ DuckDB 不做 ML  │  ├─ 嵌入式 BI / 报表？  │     ├─ 单用户报表 → ✅ DuckDB  │     ├─ 多用户并发仪表板 → ⚠️ 小规模可以，大规模用 ClickHouse  │     └─ 需要实时更新 → ⚠️ DuckDB 写入不是强项  │  ├─ OLTP（事务处理）？  │     ├─ 高并发读写 → ❌ 用 PostgreSQL / MySQL  │     ├─ 单用户读写 → ⚠️ 简单场景可以  │     └─ 金融交易级一致性 → ❌ DuckDB 不是为此设计的  │  └─ 分布式 / 多节点？        └─ ❌ DuckDB 是单机引擎（MotherDuck 提供云端方案）
```

### 一句话总结适用性

**DuckDB 最适合：一个人（或少数人），在一台机器上，对中等规模数据，做分析查询。**

如果你的场景符合这个描述——恭喜，DuckDB 可能是你目前能找到的最好的工具。

---

## DuckDB vs MotherDuck：本地与云端如何协同？

MotherDuck 是 DuckDB Labs 联合创始人 Jordan Tigani（前 Google BigQuery 工程总监）创立的公司，做的事情是：**把 DuckDB 搬到云端，让它能处理更大的数据。**

```
本地 DuckDB                     MotherDuck┌────────────────┐              ┌────────────────────┐│ 你的笔记本电脑  │    同步       │  云端 DuckDB 实例    ││                │ ◄──────────► │                    ││ duckdb://local │              │ md:my_database     ││                │              │                    ││ 数据: 本地文件  │              │ 数据: 云端存储       │└────────────────┘              └────────────────────┘
```

核心思路是 **Hybrid Execution**（混合执行）：

•小数据、敏感数据：本地执行•大数据、共享数据：云端执行•DuckDB 的 SQL 方言完全相同，切换无感

这其实呼应了前面的"Local-First"理念——**本地是默认，云端是补充。**

---

## 未来展望：DuckDB 能否成为"数据分析的默认引擎"？

让我们做一个大胆的预测：**在 3-5 年内，DuckDB 会成为大多数数据分析师的默认工具。**

支撑这个判断的趋势：

1.

**硬件越来越强**：M4 MacBook Pro 配 128GB 内存已经不稀奇了。单机能处理的数据量在快速增长。10 年前 100GB 需要集群，现在笔记本就能搞定。

2.

**Arrow 生态成熟**：Pandas、Polars、Spark、R——所有主流数据框架都在向 Arrow 靠拢。DuckDB 通过 Arrow 零拷贝和所有人对接（第 13 篇详细讲了）。

3.

**"Shift Left" 趋势**：数据处理从中心化的大数据平台，转向开发者本地。DuckDB 完美契合这个趋势。

4.

**WASM 支持**：DuckDB 已经可以在浏览器里运行（通过 WebAssembly）。想象一下：打开一个网页就能 SQL 分析数据，不需要安装任何东西。

5.

**AI 工作流**：LLM 生成 SQL、Agent 自动分析数据——这些新范式都需要一个轻量、快速、可嵌入的 SQL 引擎。DuckDB 天然适合。

当然，也有局限：

•**单机上限**：无论硬件多强，TB 级数据还是需要分布式方案•**写入瓶颈**：单写者模型限制了写入密集型场景•**生态成熟度**：和 PostgreSQL 几十年的生态比，DuckDB 还很年轻

但对于 80% 的分析场景——**DuckDB 已经足够好了。**

---

## DuckDB 在数据工程技术栈中的定位

```
┌─────────────────────────────────────────────────────────────────────┐│                      数据工程技术栈全景                               ││                                                                     ││  数据来源          处理引擎           存储/输出         消费端        ││  ────────          ────────           ────────         ────────     ││                                                                     ││  ┌─────────┐    ┌──────────────┐    ┌─────────┐    ┌─────────┐    ││  │ API     │    │              │    │ Parquet │    │ BI 工具  │    ││  │ CSV     │───►│   DuckDB     │───►│ DuckDB  │───►│ Jupyter │    ││  │ JSON    │    │  (本地分析)   │    │ CSV     │    │ Dashboard│    ││  │ Parquet │    │              │    │         │    │ API      │    ││  │ S3      │    └──────────────┘    └─────────┘    └─────────┘    ││  └─────────┘           │                                           ││                        │ 数据量/复杂度增长                          ││                        ▼                                           ││                 ┌──────────────┐    ┌─────────┐    ┌─────────┐    ││                 │ MotherDuck   │    │ Iceberg │    │ Superset│    ││                 │ (云端 DuckDB)│───►│ Delta   │───►│ Metabase│    ││                 └──────────────┘    │ Hive    │    │ Grafana │    ││                        │            └─────────┘    └─────────┘    ││                        │ 继续增长                                   ││                        ▼                                           ││                 ┌──────────────┐                                    ││                 │ Spark        │                                    ││                 │ ClickHouse   │    ← 当真的需要分布式时             ││                 │ Snowflake    │                                    ││                 └──────────────┘                                    │└─────────────────────────────────────────────────────────────────────┘
```

**DuckDB 占据的是中间层**：比 Pandas 强大，比 Spark 简单。大多数人的大多数分析任务，都落在这个区间里。

---

## 这个系列，我们到底学到了什么？

回顾一下 16 篇文章的知识图谱：

```
第 1-3 篇（快速上手）  └─ DuckDB 是什么 → SQL 用法 → Python 集成  
第 4-6 篇（核心架构）  └─ 6 层架构 → 列式存储 + Zone Map → 向量化执行  
第 7-9 篇（深入算法）  └─ 35 个优化器 → 16 种压缩 → 聚合 + JOIN 实现  
第 10-12 篇（系统设计）  └─ Morsel-Driven 并行 → MVCC 事务 → 扩展机制  
第 13-16 篇（生态与展望）  └─ Arrow 零拷贝 → 性能调优 → 版本演进 → 设计哲学
```

如果只能记住三件事：

1.

**列式存储 + 向量化执行 = 分析查询的最优解。** DuckDB 每次处理 2048 行（不是 1 行），数据按列存储（不是按行），这两个设计决策贡献了大部分性能优势。

2.

**"嵌入式"不等于"玩具"。** DuckDB 30 万行 C++ 代码，35 个优化器，16 种压缩算法——它的内核复杂度不亚于任何"大型"数据库。只是它选择了不同的部署方式。

3.

**好的工程设计是做好取舍。** DuckDB 不做分布式、不做高并发写入、不做流处理——正是这些"不做"，让它在自己的领域做到了极致。

---

## 写在最后

写这个系列的过程中，我反复翻看 DuckDB 的源代码，有几个感受想分享：

**第一，DuckDB 的代码质量非常高。** 命名清晰、注释到位、架构分层合理。如果你想学习数据库内核开发，DuckDB 是目前最好的入门对象之一——代码量适中（30 万行，比 PostgreSQL 的 130 万行好啃得多），且是现代 C++ 风格。

**第二，理论到工程的距离很远。** 向量化执行的论文只有几页，DuckDB 的实现有几万行。论文里的"Arrow 零拷贝"听起来简单，实际上要处理类型映射、内存生命周期、格式版本兼容——细节决定成败。

**第三，"简单"是最难的设计目标。** `pip install duckdb` 背后是六年的工程积累。让用户觉得"这东西真简单"，需要开发者把所有复杂度都消化在内部。

如果这个系列能帮你在下次数据分析时多一个工具选择，或者在读数据库论文时多一些具象的理解，那就足够了。

**DuckDB 的旅程还在继续。我们的旅程，也是。**

---

## 参考资料

•DuckDB 官方网站[1]•DuckDB 官方博客[2]•DuckDB GitHub[3]•MotherDuck[4]•Apache Arrow[5]•Mark Raasveldt, Hannes Mühleisen. **DuckDB: an Embeddable Analytical Database**. SIGMOD 2019.•Thomas Neumann. **Efficiently Compiling Efficient Query Plans for Modern Hardware**. VLDB 2011.•Peter Boncz, Marcin Zukowski, Niels Nes. **MonetDB/X100: Hyper-Pipelining Query Execution**. CIDR 2005.•Viktor Leis et al. **Morsel-Driven Parallelism: A NUMA-Aware Query Evaluation Framework for the Many-Core Age**. SIGMOD 2014.

---

### References

`[1]` DuckDB 官方网站: *https://duckdb.org/*  
`[2]` DuckDB 官方博客: *https://duckdb.org/news/*  
`[3]` DuckDB GitHub: *https://github.com/duckdb/duckdb*  
`[4]` MotherDuck: *https://motherduck.com/*  
`[5]` Apache Arrow: *https://arrow.apache.org/*