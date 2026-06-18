# ClickHouse MergeTree 批处理预排序与 LSM 边界

## 原文锚点

- 本地文件：[「ClickHouse系列」ClickHouse的优化之Block+LSM](../文章/「ClickHouse系列」ClickHouse的优化之Block+LSM.md)
- 原文链接：`http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247513879&idx=1&sn=f6d921cada4fb08b4b3e3abbd24e94a2`
- 关键段落：Block 批处理、预排序、有序范围查询、写入后形成多个有序文件、后台合并、写入延迟和更新删除代价。
- 关键图：正文提到查询读放大示意、缓存命中率关系图、未合并结果图，但本地 Markdown 没有图片。

## 图片处理

| 图片 | 类型 | 是否保留 | 理由 | 处理方式 |
|---|---|---|---|---|
| 有序/无序范围读取对比图 | 对比图 | 原图缺失 | 帮助理解排序键对范围扫描的影响 | Mermaid 重建 |
| 缓存命中率关系图 | 说明图 | 原图缺失 | 原文用它支撑读量估算，但缺实验上下文 | 不进入结论 |
| 未合并数据文件查询结果图 | 说明图 | 原图缺失 | 帮助理解后台 merge 前的多 part 状态 | Mermaid 重建 |

```mermaid
flowchart LR
  Insert["批量写入"] --> Sort["内存按排序键预排序"]
  Sort --> Part["写出不可变 Part"]
  Part --> Query["范围查询按排序键裁剪"]
  Part --> Merge["后台 Merge"]
  Merge --> BiggerPart["更大有序 Part"]
  BiggerPart --> Query
```

## 一句话结论

这篇文章值得精读：ClickHouse 的高吞吐查询不是单靠列存，而是由批处理、压缩、排序键和后台 merge 共同换来的；代价是小批频繁写入、单行删除和更新都不是它的舒适区。

## 用户相关性判断

| 项 | 内容 |
|---|---|
| 用户当前认知层级 | ClickHouse / OLAP 引擎：L2 draft |
| 认知成熟度 | draft |
| 阅读投入建议 | 精读 |
| 阅读投入理由 | 能补 ClickHouse 存储模型和写读取舍边界，但原文性能数字缺少完整版本、硬件和 Profile 证据 |
| 对用户的新信息 | LSM-like 思路在 ClickHouse 中主要服务于预排序和 part merge，不应被简单理解为写多读少数据库的通用 LSM |
| 问题指纹 | ClickHouse + MergeTree 存储 + Block/排序键/Part Merge/压缩 + 范围查询加速 + 小批写入和更新删除边界 |
| 排重判断 | 新建 |
| 置信度 | 高 |

## 认知校准点

| 校准点 | 文章观点/信息 | 与用户认知或价值观的关系 | 处理建议 |
|---|---|---|---|
| ClickHouse 的排序键是读路径前提 | 原文用范围查询说明有序存储能减少扫描和 IO | 补充 OLAP 存储模型边界 | 设计表时先看高频过滤和排序字段，不泛化为“列存即可快” |
| Block 的价值要和压缩一起理解 | 原文强调 Block 批处理的关键收益来自列存压缩和批量读取 | 纠偏“Block 只减少 IO 次数”的浅层理解 | 写入 index 的核心模块 |
| ClickHouse 使用 LSM 思路不等于写多读少 | 原文对比 LevelDB，指出 ClickHouse 面向读多写少分析 | 纠正 LSM 关键词误导 | 横向对标 RocksDB/HBase 时先看主问题 |
| Merge 前后查询状态不同 | 多批写入会产生多个有序文件，后台合并前查询可能需要处理多个 part | 补足失败/边界场景 | 和 Compaction、ReplacingMergeTree、Upsert 一起追查 |
| 性能倍数不能直接采信 | 原文给出 18 倍、24 倍等估算 | 符合用户反无基线性能结论偏好 | 只保留机制，不保留为选型结论 |

## 冲突点

| 冲突类型 | 具体表现 | 影响 | 处理 |
|---|---|---|---|
| 图片缺失 | 多处“如下图/表中”在 Markdown 中没有图片 | 影响证据链完整性 | 重建机制图，性能图不重建 |
| 证据不足 | 性能估算缺少版本、磁盘、压缩算法、排序键和查询 Profile | 不能直接指导选型 | 标为待验证 |
| 关键词误导 | LSM 容易让人联想到写多读少 KV 系统 | 会误读 ClickHouse 的设计目标 | 明确是 MergeTree/part merge 语境 |
| 已知可跳过 | 文末大量公众号推荐链接 | 干扰阅读 | 不进入知识点 |

## 待吸收点

| 分级 | 内容 | 为什么值得吸收 | 后续动作 |
|---|---|---|---|
| 理解 | ClickHouse 写入会先按排序键组织数据，再形成不可变 part | 解释范围查询和数据裁剪为什么依赖排序键 | 追查 MergeTree part、mark、granule |
| 理解 | Block 批处理与列存压缩共同降低读取成本 | 是存储模型的基础机制 | 和 Skip Index 笔记合并到阅读路线 |
| 记住 | 高频小批写入会制造更多 part，merge 未完成前查询成本上升 | 会影响实时写入边界 | 对照 Doris/StarRocks Compaction |
| 记住 | ClickHouse 不适合把单行修改、事务更新、随机删除当主路径 | 影响选型判断 | 用 Upsert 增强文章做横向边界 |
| 实践 | 用同一表不同排序键、批大小和查询条件比较扫描量 | 可验证排序键和 part merge 的效果 | 后续补本地实验 |

## 已知可跳过

| 内容 | 跳过理由 |
|---|---|
| 列存数据库适合分析查询 | 用户大概率已知 |
| LSM 基础历史和 LevelDB 泛讲 | 只作为对标背景 |
| 工程师精神和公众号引流 | 不影响技术判断 |

## 实践门槛

| 门槛 | 判断 | 证据 |
|---|---|---|
| 可运行 | 部分 | 原文有查询例子，但没有完整建表、数据和 Profile |
| 可验证 | 否 | 缺扫描量、压缩率、part 数、硬件和版本对照 |
| 可排障 | 部分 | 能解释小批写入和 merge 未完成导致的查询成本 |
| 可迁移 | 是 | 可迁移到 OLAP 表排序键和写入批次设计 |
| 结论 | 降为精读 | 机制可沉淀，实践需另做实验 |

## 归类判断

| 项 | 内容 |
|---|---|
| 技术本体 | ClickHouse MergeTree 存储模型 |
| 文章主问题 | ClickHouse 如何通过 Block、预排序和后台 merge 提升范围分析查询 |
| 使用场景 | 大批量写入、列式压缩、范围查询、日志和明细分析 |
| 关键词干扰 | LSM、LevelDB、HBase 是对标背景，不改变主类目 |
| 最终归类 | OLAP 与数据库 / OLAP 引擎 / ClickHouse |
| 归类理由 | 主问题是 ClickHouse 存储结构和读写取舍，不是通用存储引擎或 KV 数据库 |

## 技术定位

| 项 | 内容 |
|---|---|
| 技术类型 | OLAP 存储模型机制 |
| 所属领域 | OLAP 与数据库 |
| 二级类目 | OLAP 引擎 |
| 全局架构位置 | ClickHouse MergeTree 写入和读取路径的存储组织层 |
| 涉及模块 | Block、排序键、Part、Merge、压缩、范围扫描 |
| 解决问题 | 用顺序批量写入和有序列存提升范围查询吞吐 |
| 原文局限 | 图缺失，性能估算不可直接复用，缺版本说明 |
| 我的结论 | 以后关注，作为 ClickHouse 存储模型入口 |

## 跨域判断

| 问题 | 判断 |
|---|---|
| 它本体属于哪里 | OLAP 与数据库 / OLAP 引擎 |
| 这篇文章为什么可能跨域 | 提到 LSM、LevelDB、HBase 等存储系统 |
| 当前文章主问题是否改变分类 | 不改变，主问题是 ClickHouse MergeTree |
| 应避免的误归类 | 不归到通用存储引擎或 KV 缓存 |

## 纵向理解

| 维度 | 判断 |
|---|---|
| 全局架构 | 数据源 -> ClickHouse insert -> 内存排序 -> part 写出 -> 后台 merge -> SQL 读取 part/granule |
| 本文位置 | 只讲 MergeTree 存储与 part merge，不讲分布式查询、物化视图和 Upsert |
| 核心机制 | 批处理、排序键、列存压缩、不可变 part、后台 merge |
| 使用链路 | 选排序键 -> 控制写入批次 -> 写入形成 part -> 等待 merge -> 通过查询 Profile 验证扫描量 |
| 前置条件 | 数据量足够大，查询能利用排序键，写入不是极小批高频随机更新 |
| 边界 | 低选择性查询、排序键不匹配、频繁 update/delete、小 part 堆积都会削弱收益 |

## 横向对标

| 对标技术 | 实现方式 | 优势 | 劣势 | 适合场景 |
|---|---|---|---|---|
| ClickHouse MergeTree | 有序 part + 列存压缩 + 后台 merge | 范围分析和明细扫描强 | 更新删除和小批写入弱 | 日志、明细、宽表分析 |
| Doris Rowset/Compaction | Rowset 版本 + Compaction | 导入和服务化分析能力强 | Compaction 积压会影响读写 | 实时分析和报表服务 |
| StarRocks Primary Key | 主键索引 + Delete Vector + 新版本 | 实时 Upsert 查询性能好 | 主键索引和写入提交有成本 | CDC 后实时分析 |
| RocksDB/LevelDB | LSM memtable + SSTable | 写入吞吐和点查成熟 | 分析 SQL 能力不是主目标 | KV/嵌入式存储 |

## 后续追查

- 关键词：ClickHouse MergeTree、Part、Block、Mark、Granule、Merge、ORDER BY。
- 相关技术：Skip Index、AggregatingMergeTree、ReplacingMergeTree、Doris Compaction、StarRocks Primary Key。
- 需要补读的文章：ClickHouse MergeTree 官方文档、ReplacingMergeTree、火山引擎 ClickHouse Upsert 增强边界。
