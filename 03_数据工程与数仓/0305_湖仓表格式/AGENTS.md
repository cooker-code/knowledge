# 湖仓表格式
## 知识点入口

- 本模块先看宏观流程，再看文章：[流程化知识点总览](knowledge/03_数据工程与数仓/0305_湖仓表格式/核心知识点/流程化知识点总览.md)。
- 新文章必须先归入流程节点，再判断是补充、冲突、不同层次还是降权。
- `文章/` 只保留原文锚点，长期知识必须沉淀到 `核心知识点/`。


## 类目定位

| 项 | 内容 |
|---|---|
| 一级类目 | 数据工程与数仓 |
| 二级类目 | 湖仓表格式 |
| 核心问题 | 如何在数据湖文件之上提供表语义、快照、事务、Schema 演进、增量和多引擎互操作 |
| 不解决什么 | 不直接替代计算引擎、调度系统、OLAP 查询服务或消息队列 |
| 用户当前认知假设 | L1 到 L2：知道湖仓概念和 Hive 边界，但需要进一步区分 Iceberg、Hudi、Paimon 的机制和场景 |

## 用户认知重点

| 认知项 | 当前假设 | 后续整理策略 |
|---|---|---|
| 已知基础 | 数据湖、数仓、Hive 表的大体区别 | 不重复讲概念史 |
| 待补边界 | 表格式和计算引擎、OLAP 引擎、消息队列之间的边界 | 每篇文章必须横向对标 |
| 易偏差点 | “湖仓地基”“替代数仓”“实时数仓全部能力”这类表述容易夸大 | 优先做降权和边界校准 |

## 排重准则

问题指纹：

```text
表格式 + 表类型/模块 + 快照/事务/增量/更新机制 + 解决问题 + 引擎/延迟/成本边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是湖仓综述 | 如果没有具体机制和对标，通常略读 |
| 都是 Paimon | 按主键表、Append 表、Changelog、Compaction、Flink 集成拆分 |
| 都是 Iceberg/Hudi/Paimon 对比 | 有明确选型准则才沉淀 |
| 只讲口号 | 跳过或只保留认知校准 |
| 提供实践链路 | 优先沉淀，进入后续实验 |

## 已覆盖技术

| 技术 | index | 已覆盖问题 | 还缺什么 |
|---|---|---|---|
| Paimon | [Paimon](Paimon/AGENTS.md) | 实时湖仓表格式定位、CDC 接入、Merge Engine、Changelog、Append 表边界、外部状态、LSM、Bloom、生命周期治理 | 官方版本限制、查询引擎兼容矩阵、最小实验 |
| Iceberg | [Iceberg](Iceberg/AGENTS.md) | Snapshot、Branch/Tag、v3 删除向量、行级血缘、查询加速平台边界 | 官方 v3 支持矩阵、Flink CDC/增量读写、Catalog 选型 |
| Hudi | [Hudi](Hudi/AGENTS.md) | Timeline、Index、COW/MOR、Payload 多流拼接、并发写边界 | 当前版本补证、Flink 写入、Compaction/Clean 实验 |
| Delta Lake | [Delta Lake](Delta%20Lake/AGENTS.md) | 本轮仅创建待补入口，未做正式文章沉淀 | 需要本地原文、官网/GitHub、与 Iceberg/Hudi/Paimon 对比 |

## 待补技术和问题

| 技术/问题 | 为什么要补 | 优先级 |
|---|---|---|
| Delta Lake | 文章来源本轮没有本地原文，但它是 Iceberg/Hudi/Paimon 的核心横向对标对象 | 高 |
| Iceberg/Hudi/Paimon/Delta Lake 对比 | 需要形成表格式选型准则，避免被“谁取代谁”标题误导 | 高 |
| Flink CDC -> 表格式链路对比 | 用户关注 CDC、Schema 演进、增量读写和下游一致性 | 高 |
| OLAP 引擎读取湖表边界 | Doris/StarRocks/Trino/ClickHouse 与湖表的职责容易混 | 中 |

<!-- MANUAL:PARALLEL_READING_2026_06_13_START -->
## 本轮并行精读沉淀

> 2026-06-13，本轮只使用本地 `本地文章目录` 与已有 `knowledge`，未联网补官网、GitHub 或外部资料。新建技术目录的官网/GitHub 字段均标为“后续补证”。

| 指标 | 数量 |
|---|---:|
| 处理精读候选 | 16 |
| 新增技术目录 | 3 |
| 新增核心知识点 | 7 |
| 正式/合并沉淀原文 | 14 |
| 合并或跳过原文 | 2 |
| Delta Lake 本地原文 | 0 |

### 本轮处理结果

| 技术对象 | 原文 | 处理 | 进入文件 | 原因 |
|---|---|---|---|---|
| Hudi | Hudi 核心知识点详解（万字长文） | 正式沉淀 | [Hudi Timeline、索引与 COW/MOR 表类型](Hudi/核心知识点/Hudi%20Timeline、索引与%20COW-MOR%20表类型.md) | 补 Hudi 本体、Timeline、索引、COW/MOR、查询类型 |
| Hudi | 万字长文：基于Apache Hudi + Flink多流拼接(大宽表)最佳实践 | 正式沉淀 | [Hudi Payload 多流拼接与并发写边界](Hudi/核心知识点/Hudi%20Payload%20多流拼接与并发写边界.md) | 补 Payload、多流拼接、OCC、Marker、失败场景 |
| Iceberg | Hive 实践 \| Apache Hive 4.x与Iceberg分支和标签 | 正式沉淀 | [Iceberg 快照分支标签与 Hive 接入边界](Iceberg/核心知识点/Iceberg%20快照分支标签与%20Hive%20接入边界.md) | 补 Branch/Tag 快照生命周期，按 Iceberg 重路由 |
| Iceberg | 为什么 Iceberg v3 是数据湖仓的"iPhone 时刻"？ | 合并沉淀 | [Iceberg v3 删除向量与行级血缘边界](Iceberg/核心知识点/Iceberg%20v3%20删除向量与行级血缘边界.md) | 补 v3 删除向量、行级血缘、VARIANT，同时降权宣传话术 |
| Iceberg | Mooncake: 基于 Apache Iceberg 构建的实时存储引擎 | 合并沉淀 | [Iceberg v3 删除向量与行级血缘边界](Iceberg/核心知识点/Iceberg%20v3%20删除向量与行级血缘边界.md) | 作为 Iceberg 生态实时扩展边界，不单独建 Mooncake 目录 |
| Iceberg | 秒级响应！B站基于 Iceberg 的湖仓一体平台构建实践 | 正式沉淀 | [Iceberg 查询加速与平台化边界](Iceberg/核心知识点/Iceberg%20查询加速与平台化边界.md) | 补 Iceberg 与 Trino、缓存、索引、Cube、OLAP 出口边界 |
| Paimon | 代替Kafka? Paimon追加表真的可以 | 合并沉淀 | [Paimon追加表与外部状态边界](Paimon/核心知识点/Paimon追加表与外部状态边界.md) | 标题需降权，保留 Append 表队列式流读边界 |
| Paimon | 揭秘Fluss、Kafka、Paimon 的联系和区别 | 合并沉淀 | [Paimon追加表与外部状态边界](Paimon/核心知识点/Paimon追加表与外部状态边界.md) | 补 Kafka/Fluss/Paimon 横向边界，Fluss 待补证 |
| Paimon | Flink+Paimon实时数据湖仓实践分享 | 合并沉淀 | [Paimon追加表与外部状态边界](Paimon/核心知识点/Paimon追加表与外部状态边界.md) | 补 Paimon 外部状态、LookupJoin、bucket key 失败场景 |
| Paimon | Paimon Bloom索引深度解析：从38s到1.5s的性能飞跃与生产最佳实践 | 合并沉淀 | [Paimon LSM、Bloom 与生命周期治理](Paimon/核心知识点/Paimon%20LSM、Bloom%20与生命周期治理.md) | 保留 Bloom 文件裁剪和误判边界，降权性能数字 |
| Paimon | 美团面试：Paimon LSM-Tree 分层机制是怎么样的？ | 合并沉淀 | [Paimon LSM、Bloom 与生命周期治理](Paimon/核心知识点/Paimon%20LSM、Bloom%20与生命周期治理.md) | 补 LSM 分层和 Compaction 读写放大 |
| Paimon | 腾讯面试：Paimon自动分区清理与快照清理机制是怎么样的？哪个先清理？ | 合并沉淀 | [Paimon LSM、Bloom 与生命周期治理](Paimon/核心知识点/Paimon%20LSM、Bloom%20与生命周期治理.md) | 补快照清理、分区逻辑/物理删除依赖 |
| Paimon | Apache Paimon 1.3 核心亮点总结 | 合并沉淀 | [Paimon LSM、Bloom 与生命周期治理](Paimon/核心知识点/Paimon%20LSM、Bloom%20与生命周期治理.md) | 只保留 Row Tracking、Data Evolution、Incremental Clustering 的待补证方向 |
| Paimon | Apache Paimon核心配置参数详解（二） | 合并沉淀 | [Paimon LSM、Bloom 与生命周期治理](Paimon/核心知识点/Paimon%20LSM、Bloom%20与生命周期治理.md) | 合并 Bucket、Changelog、保留策略参数，不单独写参数清单 |
| Paimon | 从数据湖到数据湖仓：开放表格格式驱动的架构革新 | 合并/跳过 | 本 index | 综述价值高但机制浅，只补开放表格式横向视角 |
| Paimon | 数据仓库的黄昏？Paimon与Hive的正面对决与终极融合 | 合并/跳过 | [Paimon](Paimon/AGENTS.md) | 标题宣传强，保留 Hive/Paimon 边界，不新建笔记 |

### 主要冲突点

| 冲突点 | 本轮处理 |
|---|---|
| 原目录冲突 | Hudi、Iceberg、Paimon 多篇来自 `09_其他`、`08_职业与管理`、raws，均按技术本体重路由到湖仓表格式 |
| 标题/观点降权 | “代替 Kafka”“iPhone 时刻”“数据仓库的黄昏”“性能飞跃”等不直接采信，只保留机制和边界 |
| 图片缺失 | Hudi 多流、Iceberg 平台架构、Paimon LSM/清理等均标记原图缺失，并在核心笔记中用 Mermaid 重建 |
| 证据不足 | 版本、默认值、性能数字、厂商支持未联网补证，均保留为待验证，不写成当前事实 |
| 排重冲突 | Paimon 参数/LSM/Bloom/清理类文章合并为一个生产边界主题，避免碎片化扩写 |
<!-- MANUAL:PARALLEL_READING_2026_06_13_END -->

<!-- AUTO:SECONDARY_INIT_START -->
## 全量文章来源初始化

> 自动生成。初始化阶段只使用本地 `本地文章目录`、已有 `knowledge` 和本地 `wiki`，不联网补官网或外部证据。

- 全量文章来源：[文章](文章/)
- 全局明细：`scripts/output/knowledge-secondary-pools.json`

| 指标 | 数量 |
|---|---:|
| 文章数 | 46 |
| 正式沉淀原文数 | 16 |
| 已引用锚点 | 38 |
| 核心知识点数 | 11 |
| 精读候选 | 6 |
| 略读 | 23 |
| 跳过 | 1 |
| 低置信 | 0 |
| 原图缺失 | 17 |

### 主题簇

| 技术/主题 | 文章数 | 正式沉淀 | 精读候选 | 原图缺失 | 处理决策 | 认知校准点 |
|---|---:|---:|---:|---:|---|---|
| Paimon | 37 | 13 | 3 | 13 | 以已有核心知识点为排重基线，只补边界、失败场景或实践证据 | 原目录存在误导，按技术本体重路由；有技术图缺失，精修时需回原文或重建 |
| Iceberg | 6 | 1 | 3 | 1 | 以已有核心知识点为排重基线，只补边界、失败场景或实践证据 | 原目录存在误导，按技术本体重路由；有技术图缺失，精修时需回原文或重建 |
| Hudi | 3 | 2 | 0 | 3 | 已形成初始锚点，后续补缺口 | 原目录存在误导，按技术本体重路由；有技术图缺失，精修时需回原文或重建 |

### 精读候选

| 技术对象 | 原文 | 冲突点 | 处理建议 |
|---|---|---|---|
| Iceberg | [Hive 实践 \| Apache Hive 4.x与Iceberg分支和标签](文章/Hive 实践 _ Apache Hive 4.x与Iceberg分支和标签.md) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Iceberg | [Mooncake: 基于 Apache Iceberg 构建的实时存储引擎](文章/Mooncake_ 基于 Apache Iceberg 构建的实时存储引擎.md) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Iceberg | [为什么 Iceberg v3 是数据湖仓的"iPhone 时刻"？](文章/为什么 Iceberg v3 是数据湖仓的_iPhone 时刻_？.md) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Paimon | [从数据湖到数据湖仓：开放表格格式驱动的架构革新](文章/从数据湖到数据湖仓：开放表格格式驱动的架构革新.md) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Paimon | [代替Kafka? Paimon追加表真的可以](文章/代替Kafka_ Paimon追加表真的可以.md) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Paimon | [数据仓库的黄昏？Paimon与Hive的正面对决与终极融合](文章/数据仓库的黄昏？Paimon与Hive的正面对决与终极融合.md) | 标题/观点需要降权；正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |

### 冲突与缺口

- 冲突分布：原目录与最终归类不一致(23)、正文提到技术图但 Markdown 无图(17)、标题/观点需要降权(3)
- 存在原图缺失，精修时只补架构图、流程图、说明图和对比图。
- 精读候选不直接写笔记，先和已有问题指纹对比。
- 初始化阶段不补外部官网/GitHub；需要官方证据时在后续精修阶段补证。

### 下一步

| 优先级 | 动作 |
|---|---|
| P0 | 先处理精读候选，按主题簇合并，不逐篇扩写 |
| P1 | 对已有核心知识点补充排重依据和认知校准点 |
| P2 | 精修阶段再补官网、GitHub、版本状态和官方架构图 |
<!-- AUTO:SECONDARY_INIT_END -->

