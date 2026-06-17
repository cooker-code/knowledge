---
title: Lance Format 2.2：驯服复杂数据
author: Lance & LanceDB
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUzMjA3MTYzMg==&mid=2247484242&idx=1&sn=713af5b49b3ad4e6749841eb65f14370&chksm=fb84e6d9dfab452319b40eb8c6970eb770474425ead193ca332bc4cbfb6d640ba58f5b2b664e&mpshare=1&scene=24&srcid=03289JHzXXAvMtgldR5hNGAM&sharer_shareinfo=b9812d0cb9a542d353225de1cc761b80&sharer_shareinfo_first=b9812d0cb9a542d353225de1cc761b80#rd
---

> 本文翻译自 LanceDB Blog
>
> 作者：Xuanwo

在 LanceDB 内部，我们常说一句话：“100B is the new 100M”（千亿级已成为新的亿级）

过去，1 亿条记录就算得上是大规模多模态数据集了。

而今天，这一边界正向 1000 亿甚至更高量级推进。与此同时，AI/ML 的发展速度正在重新定义数据基础设施必须处理的内容。训练数据不再仅仅是文本和嵌入；多模态模型现在依赖于图像、音频、视频和其他大型工件。

RAG和智能体系统需要将向量与丰富且快速变化的元数据配对。特征工程的 Schema 变得越来越深，并随着每一次产品迭代而演进。随着数据集规模的持续扩大，存储成本已成为一项硬性限制，而非可有可无的备注。

这些趋势都指向同一个方向：一种专为 ML/AI 设计的数据格式必须同时具备处理**复杂类型**、**大对象**、**Schema 演进**以及**高效压缩**的能力。

**Lance 文件格式 2.2** (`data_storage_version="2.2"`) 正是围绕这些需求而构建的。

2.1 版本通过结构化编码（Structural encoding）奠定了基础，将随机访问 I/O 降低至 1-2 次往返。

2.2 版本在此基础上更进一步，带来了重新设计的 **Blob****V2**、灵活的**嵌套字段演进**、**原生 Map 类型**以及**压缩性能的全面提升**。其结果是，该格式能够更好地适应多模态、动态、大规模的 AI 数据负载。

**💡2.2 指的是文件格式版本**

本文中提到的版本号（2.0、2.1 和 2.2）是指 **Lance 文件格式版本**（即 `data_storage_version` 参数），它与您在 GitHub 上看到的 Lance 库发行版本是相互独立的。

**1.文件格式版本**决定了数据在磁盘上的编码和存储方式。

**2.库版本**决定了 SDK 可以读写哪些格式版本。

## 

以下章节重点介绍了 **Format 2.2** 中的新特性：

# Blob V2：将大文件视为“一等公民”

# 

此前，Lance 通过为 `LargeBinary` 字段标记 `lance-encoding:blob` 元数据来支持大对象存储，我们称之为 **Blob V1**。虽然这能处理基础场景，但它仍将每个 Blob 内联（Inline）存储在文件中。这导致处理极大对象时非常棘手，不仅要求将完整的 Blob 数据写入 Lance，还无法利用对象存储中已存在的外部文件。

同样重要的是，难点不仅在于文件布局，还在于**表级管理**：包括压缩、生命周期管理，以及在不强制重写整个数据集的情况下集成外部对象。Format 2.2 引入的 **Blob****V2** 是基于这两个层面的彻底重构，也是 Lance 广泛端到端格式策略的典型案例。对于新项目，Blob V2 是推荐路径；旧有的 Blob 数据在 Format 2.2 下依然可读。

其核心成果之一是**自适应存储布局**。在写入时，Blob V2 会根据实际数据自动选择最佳存储策略：

**💡小****负载**：内联存储在数据页内；

**💡中等****负载**：打包进共享数据块；

**💡大****负载**：存入专用区域。 这种自动分层机制既避免了小文件膨胀，又能在各种 Blob 尺寸下保持稳定的读写性能。

另一项核心成果是**外部****Blob****管理**。Blob V2 使用 `lance.blob.v2` 扩展类型标识符，写入通过专用的 `blob_field` 和 `blob_array` 构造函数进行，读取则返回延迟加载的 `BlobFile` 句柄，按需流式传输字节。更重要的是，你可以将外部存储（如 S3、GCS 路径等）中的媒体文件直接注册到 Lance 表中，**无需将数据复制到数据集中**。

对于拥有海量媒体资产的团队来说，这改变了游戏规则：一个 URI，甚至是 URI 中的一段字节范围，都能成为 Lance 统一管理的一部分，而无需进行大批量重写。

以下是一个典型的写入示例：

```
import lanceimport pyarrow as pafrom lance import blob_array, blob_field  
schema = pa.schema([    pa.field("id", pa.int64()),    blob_field("data"),])  
table = pa.table(    {        "id": [1, 2],        "data": blob_array([b"inline-bytes", "s3://bucket/path/video.mp4"]),    },    schema=schema,)  
# External URIs outside dataset base paths require this write optionds = lance.write_dataset(    table,    "./blobs.lance",    data_storage_version="2.2",    allow_external_blob_outside_bases=True,)  
# Returns a BlobFile handle for streaming accessblob = ds.take_blobs("data", indices=[0])[0]with blob as f:    print(f.read())
```

`take_blobs` 返回的是**延迟加载****句柄****（Lazy handles）**。在此阶段不会抓取任何数据。只有当你调用 `f.read()` 时，系统才会按需提取字节，这使其非常适合大文件的流式传输。

迁移路径非常明确：新项目应直接使用 V2 API；现有项目可以参考文档中的批量迁移示例逐步切换。类型定义、流式读取、外部引用：只需一套 API 即可覆盖所有场景。

# 嵌套 Schema 演进

# 

基于我们在维护特征工程项目中的经验，我们对这一领域有着非常深刻的理解。假设你有一个如下所示的 Schema：

```
{  "user_profile": {    "basic": {"age": 25, "gender": "M"},    "behavior": {"last_7d_click": 120, "embeddings": [0.1, 0.2, 0.3]}  }}
```

随后，产品团队需要在 `basic` 字段中增加一个 `country` 字段。在 Format 2.1 中，系统仅支持从Structs中删除子列。

而在 **Format 2.2** 中，Lance 已经支持**在嵌套结构体中添加新字段**（包括“结构体列表”布局），且无需重写已有的父级数据。对于添加全空值的情况，这可以仅作为一个**元数据****操作**完成；对于物化添加，Lance 会在保持现有数据文件完整的情况下，追加新字段的数据。在读取时，系统会自动根据需要将缺失的嵌套子字段合成为 Null 值：

```
import lanceimport pyarrow as pa  
table = pa.table(    {        "user_profile": [            {"basic": {"age": 25, "gender": "M"}},            {"basic": {"age": 30, "gender": "F"}},        ]    })dataset = lance.write_dataset(table, "nested_schema_demo.lance", data_storage_version="2.2")  
# Add a new nested field as an all-null column (metadata-only in format >= 2.2)dataset.add_columns(    pa.schema([        pa.field(            "user_profile",            pa.struct([                pa.field(                    "basic",                    pa.struct([                        pa.field("country", pa.string()),                    ]),                ),            ]),        ),    ]))
```

在底层实现上，Lance 为嵌套字段分配了**稳定的字段****ID**，并分别跟踪物理列的元数据。这使得引入新的嵌套子字段时，无需重写原始父列的有效负载。在许多 Schema 演进的工作流中，这一特性消除了对全量数据进行压缩的需求。

# 原生 Map 类型

# 

这一特性的诞生源于一个处理用户画像的团队，他们曾问道：“我们每天需要处理数百万用户的动态标签。键是字符串，值是字符串或整数。Lance 有好的解决方案吗？”

当时，每种选择都各有利弊：

**1.拆分为独立列**：仅适用于键值固定的场景。

**2.JSON**：灵活但查询速度慢。

**3.结构体**：类型安全但结构过于僵化。

最终，他们不得不通过“列表 + 结构体（List+Struct）”来模拟 Map 结构。虽然可行，但代码中充斥着繁琐的手动序列化和反序列化操作。

**Format 2.2 在规范中正式将 Map 列为“一等公民”：**

```
import pyarrow as pa  
schema = pa.schema([    ("user_id", pa.int64()),    ("tags", pa.map_(pa.string(), pa.string()))])
```

在物理存储上，**Map** 依然以 `List<Struct<key, value>>` 的形式存储，但在逻辑层和编码层，系统将其视为一种**独立类型**：

**1.清晰的语义**：Schema 直接声明为 Map 类型。读取端和写入端不再需要就“列表+结构体”的这种惯例达成共识。

**2.高效的编码**：偏移量和条目分别进行编码，复用了 Format 2.1 中的结构化编码框架。随机访问依然仅需 1–2 次 I/O 操作。

**3.可演进性**：Format 2.2 的嵌套演进能力同样适用于 Map。你可以独立地在键或值的子列上添加或删除字段，而无需重写整列。

对于用户而言，最直接的回报就是**代码的简化**。原本那些手动编写的 JSON 解析逻辑和拆分列的复杂逻辑，现在都可以直接删除了。

# 编码与压缩

# 

**Format 2.2** 是一次全面的压缩性能升级。2.1 版本确立了将**结构化编码**与**压缩**分离的双层架构；2.2 版本则将压缩覆盖范围扩展到了更多数据类型和编码路径。在许多场景下，与 2.0 版本相比，它在保持 Lance 卓越的扫描和随机访问性能的同时，实现了数倍的存储空间缩减。

**核心变化：**

**1.通用块压缩**：这是 2.2 版本中最大的变化。大于 **32KB** 的数据块会自动使用 **LZ4** 进行压缩，无需任何配置。如果需要更高的压缩率，可以通过元数据指定 **Zstd**。

**2.RLE 块编码**：**行程长度编码** 已从Mini-blocks提升到全块级编码。拥有大量重复值（如状态字段、分类标签等）的列将获得显著收益。

**3.字典值压缩**：字典编码后的值现在默认使用 **LZ4** 压缩。你可以通过 `lance-encoding:dict-values-compression` 自定义算法和级别。

**4.泛化常量布局**：对于每一行值都完全相同的页面（如全为 Null、全为相同默认值等），仅通过单个**内联标量**表示。存储开销实际上降为**零**。

**5.微块增强 (Mini-block Enhancement)**：最大分块大小从 32KB (`u16`) 提升至 **128KB+** (`u32`)。这不会改变默认行为，但在有利于提升压缩率时允许使用更大的分块。

**6.可变长打包结构体 (Variable Packed Struct)**：打包存储（Packed storage）从固定宽度字段扩展到了**可变宽度字段**。每个子字段独立压缩，然后转置为行主序（Row-major）布局。这非常契合需要一次性读取一组特征的 **ML 训练负载**。

**综合效果：** 

Format 2.2 实现了开箱即用，自动对大多数数据类型应用压缩。**文本、JSON、稀疏特征以及高度重复的标签列**改进最为显著。而对于嵌入向量和预压缩图像等高熵数据，收益则相对较小。我们建议针对您自己的数据进行基准测试：

```
import lanceimport pyarrow as pa  
data = pa.table({"id": [1, 2, 3], "text": ["a", "b", "c"]})ds = lance.write_dataset(data, "test.lance", data_storage_version="2.2")print(f"Rows: {ds.count_rows()}")# Compare the data directory size across different data_storage_version values
```

# 升级策略

# 

与之前的版本一样，Lance Format 2.2 采取的是选择性加入（Opt-in）机制。要使用该版本，您需要显式地进行指定：

```
import lancedb  
# 在创建或打开表时指定存储版本table = db.create_table(    "my_table",    data=data,    storage_version="2.2"  # 显式指定 2.2 版本)
```

默认的文件格式版本为 **stable**（即您安装的 Lance 版本所支持的最新的稳定格式），这让您可以完全掌控升级节奏，并有足够的空间在不同环境中进行测试。

以下是我们建议的逐步推广策略：

**1.优先升级所有环境**：确保每一个读取端的 Lance 库版本都支持 Format 2.2（请查阅发行说明以获取具体版本信息）最稳妥的方法是在所有环境中统一升级库版本。

**2.在非核心数据集上试点**：使用 Format 2.2 写入一批数据，运行所有下游任务，并对比性能与存储指标。

**3.逐步扩大范围**：从边缘业务负载开始，逐步迁移至核心生产数据。

# 何时使用

# 

以下场景将从新版本中获益最多：

**1.快速演进的嵌套结构**：机器学习特征工程、用户画像、复杂事件处理，以及任何 Schema 变更频繁的任务。

**2.动态键值属性**：埋点数据、实验参数和稀疏特征。**Map 类型**能消除大量冗余的胶水代码。

**3.多模态数据管理**：通过 **Blob V2** 统一管理图像、视频、音频和其他大对象。一套 API 即可覆盖所有场景。

**4.存储效率需求**：文本、JSON、稀疏特征及类似数据，将通过 Format 2.2 的编码改进获得显著的空间节省。

如果您的数据模型已经非常稳定，可以按照自己的节奏进行升级。Format 2.2 在读取端完全向下兼容 2.1 和 2.0 版本的数据，您可以自行决定升级窗口。

# Roadmap

# 

随着 **Format 2.2** 正式发布，我们的重点正转向以下领域：

**1.变体类型**：越来越多的用户将半结构化数据存储为 JSON。虽然我们引入了 JSONB，但其查询性能仍未达到预期。我们计划引入一种 **Variant** 类型，在保持半结构化数据灵活性的同时，提供列式查询性能和压缩效率。

**2.原生媒体类型支持**：在多模态工作流中，图像、音频和视频需要原始存储之外的“格式感知”元数据。例如图像的宽高、编码格式，或音频的采样率和声道数。目前这些元数据散落在应用层代码中。我们计划在类型系统中原生支持媒体类型，让 Lance 在存储层就能理解数据的物理特性，为下游的解码、转码和索引提供更好的基础。

**3.离散向量**：目前的向量索引主要为连续的浮点嵌入（Floating-point embeddings）设计。随着量化技术和 Token 级表示的兴起，对二进制向量、整数向量等离散向量类型的需求稳步增长。我们计划在编码和索引层原生支持离散向量，以实现更高效的存储和检索。

**4.更丰富的编码算法**：Format 2.2 中的通用块压缩和 RLE 只是开始。我们正在探索针对特定数据分布的编码算法（如 Frame-of-reference、Patched encoding、ALP 等），以进一步提升压缩率和解码速度。

**5.智能编码调优**：目前的编码决策主要基于规则。我们计划引入由数据采样驱动的**自适应编码选择**。在写入时，系统会分析数据分布并为每列自动匹配最优编码组合；在压缩（Compaction）过程中，系统会根据实际数据特征重新评估并调整编码策略，让存储效率随数据积累而不断进化。

我们的目标非常明确：**使 Lance 成为 AI/ML 工作负载的标准数据格式**。从多模态存储、动态 Schema 到高效压缩和智能编码，每一次迭代都在缩小数据基础设施与现代模型需求之间的差距。Format 2.2 是朝着该方向迈出的关键一步。

感谢所有提交 Issue、测试 Beta 版本以及为该版本贡献 PR 的开发者。Lance 的每一项改进都植根于真实世界的用例和反馈。如果您正在寻找专为 AI 工作负载设计的数据格式，欢迎加入我们。让我们一起定义下一代 ML 数据基础设施！

## 点击阅读原文，跳转LanceDB GitHub