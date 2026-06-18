> 已吸收至：[[04_OLAP与数据库/0402_关系数据库/040203_TiDB/040203_核心知识点/TiDBDM迁移复杂度边界|TiDBDM迁移复杂度边界]]
---
title: TiDB 工具分享｜TiDB DM 简化数据迁移流程复杂度
author: TiDB Club
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg3NTU3NTMxNg==&mid=2247485692&idx=1&sn=c1d666a576640d71a3da0f583f025dbc&chksm=cf3e2987f849a09130cf9e6645d7aea63f73d305d10a470116a1f6b566e914b045b1af334b8f&mpshare=1&scene=24&srcid=0805oN4XEvFzFKuvVZ6GUvbL&sharer_sharetime=1659672858543&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**点击图片，立即试用 TiDB 企业版**

TiDB 的一键水平伸缩特性，帮助用户告别了分库分表查询和运维带来的复杂度，但是在从分库分表方案切换到 TiDB 的过程中，这个复杂度转移到了数据迁移流程里。

TiDB 提供了丰富的工具，可以帮助你进行部署运维、数据管理（例如，数据迁移、备份恢复、数据校验）、在 TiKV 上运行 Spark SQL 等。

TiDB Data Migration (DM) 是一款便捷的数据迁移工具，支持从与 MySQL 协议兼容的数据库（MySQL、MariaDB、Aurora MySQL）到 TiDB 的全量数据迁移和增量数据同步。使用 DM 工具有利于简化数据迁移过程，降低数据迁移运维成本。

本文将从 DML处理逻辑、DDL协调模式、以及 relay log 性能优化几个方面展开，详细介绍 DM 的实现原理。

**DML处理逻辑**

**处理流程**

从上图可以大致了解到 Binlog replication 的逻辑处理流程

1

从 MySQL/MariaDB 或者 relay log 读取 binlog events

2

对 binlog events 进行处理转换

a.Binlog Filter：根据 binlog 表达式过滤 binlog，通过 filters 配置    

b.Routing：根据“库/表”路由规则对“库/表”名进行转换，通过 routes 配置    

c.Expression Filter: 根据 SQL 表达式过滤 binlog，通过 expression-filter 配置

3

对 DML 执行进行优化

a.Compactor：将对同一条记录（主键相同）的多个操作合并成一个操作，通过 syncer.compact 开启    

b.Causality：将不同记录（主键不同）进行冲突检测，分发到不同的 group 并发处理   

c.Merger：将多条 binlog 合并成一条 DML，通过 syncer.multiple-rows 开启

4

将 DML 执行到下游

4

定期保存 binlog position/gtid 到 checkpoint

**优化逻辑**

1

**Compactor**

DM 根据上游 binlog 记录，捕获记录的变更并同步到下游，当上游对同一条记录短时间内做了多次变更时（insert/update/delete），DM 可以通过 Compactor 将这些变更压缩成一次变更，减少下游压力，提升吞吐，如

2

**Causality**

MySQL binlog 顺序同步模型要求按照 binlog 顺序一个一个来同步 binlog event，这样的顺序同步势必不能满足高 QPS 低同步延迟的同步需求，并且不是所有的 binlog 涉及到的操作都存在冲突。

DM 采用冲突检测机制，鉴别出来需要顺序执行的 binlog，在确保这些 binlog 的顺序执行的基础上，最大程度地保持其他 binlog 的并发执行来满足性能方面的要求。

Causality 采用一种类似并查集的算法，对每一个 DML 进行分类，将相互关联的 DML 分为一组。具体算法可参考[https://pingcap.com/zh/blog/tidb-binlog-source-code-reading-8#](https://pingcap.com/zh/blog/tidb-binlog-source-code-reading-8)并行执行DML

3

**Merger**

MySQL binlog 协议，每条 binlog 对应一行数据的变更操作，DM 可以通过 Merger 将多条 binlog 合并成一条 DML 执行到下游，减少网络的交互，如：

[**点击此处查看**

**执行逻辑及精准处理的更多内容**](https://app.ma.scrmtech.com/material/read/index?new_id=330203&wx_id=1864 "https://app.ma.scrmtech.com/material/read/index?new_id=330203&wx_id=1864")

**DM 分库分表 DDL “悲观协调” 模式介绍**

TiDB 作为分库分表方案的一个 “终结者”，获得了许多用户的青睐。在切换到 TiDB 之后，用户告别了分库分表查询和运维带来的复杂度。但是在从分库分表方案切换到 TiDB 的过程中，这个复杂度转移到了数据迁移流程里。TiDB DM 工具为用户提供了分库分表合并迁移功能，在数据迁移的过程中，支持将分表 DML 事件合并迁移，并一定程度支持上游分表进行 DDL 变更。

**分库分表 DDL 的问题**

假设在两个上游有两个分表 t1、t2，下游表为 t。

t1 接下来的同步事件是 INSERT (3,3);

t2 接下来的同步事件是 DROP COLUMN c2,

如果 DROP COLUMN c2 先被同步到下游，在同步到 INSERT (3,3) 时就会因为缺少 c2 列而报错。

因此我们要对 DDL 同步事件进行特殊处理。

[**点击此处查看特殊处理、结局方法、悲观协调例子的更多内容**](https://app.ma.scrmtech.com/material/read/index?new_id=330200&wx_id=1864 "https://app.ma.scrmtech.com/material/read/index?new_id=330200&wx_id=1864")

**DM 分库分表 DDL “乐观协调” 模式介绍**

**对于悲观协调模式限制**

可以看到悲观协调模式解决方法有如下的限制：

1

出现 DDL 同步事件时分表会暂停，会导致同步延迟增加。这可能会导致恢复同步时，上游 binlog 已经被清理。

2

不支持只变更部分分表以进行灰度测试时的场景。灰度期间其余分表的同步会暂停。此外如果灰度测试结果是回滚时，无法恢复同步。

3

要求所有分表以相同的顺序出现 DDL 同步事件。

4

如果分表由于误操作而进入 DDL 不一致的状态，修复操作较为复杂。

5

对于 DM 的使用者而言，可能无法控制上游 DDL 的发起从而无法满足条件。

为此，DM 提供新的乐观协调模式，在一个分表上执行的 DDL，自动修改成兼容其他分表的 DDL 语句后立即应用到下游，不会阻挡任何分表执行的 DML 的迁移。乐观协调模式适用于上游灰度更新、发布的场景，或者是对上游数据库表结构变更过程中同步延迟比较敏感的场景。

悲观协调和乐观协调的对比

**原理**

DM worker 的所有 DML 会直接同步到下游（出错时例外）。

DM worker 内嵌了一个小型 TiDB（通称 schema tracker），用来记录各个上游分表的表结构，当接收到来自上游的 DDL 后，会根据 schema tracker 里 DDL 的执行结果，把更新后的表结构转送给 DM master。DM master 将收到的不同分片的表结构合并成可兼容所有分片的 DML 的合成结构，即不同分片表结构的并集（此过程类似于 SQL 语句中的 JOIN 语句），然后根据合成的表结构和 DM worker 发来的表结构的不同处得到对应的 DDL 语句（即合成的表结构与原表结构的差集），同步到下游。

[**点击此处查看**

**乐观协调的更多内容**](https://app.ma.scrmtech.com/material/read/index?new_id=330201&wx_id=1864 "https://app.ma.scrmtech.com/material/read/index?new_id=330201&wx_id=1864")

**DM 中 relay log 性能优化实践**

Relay log 类似 binary log，是指一组包含数据库变更事件的文件，加上相关的 index 和 mata 文件，具体细节参考官方文档。在 DM 中针对某个上游开启 relay log 后，相比不开启，有如下优势：

1

不开启 relay log 时，每个 subtask 都会连接上游数据库拉取 binlog 数据，会对上游数据库造成较大压力，而开启后，只需创建一个连接拉取 binlog 数据到本地，各个 subtask 可读取本地的 relay log 数据。

2

上游数据库对 binlog 一般会有一个失效时间，或者会主动 purge binlog，以清理空间。在不开启 relay log 时，如果 DM 同步进度较为落后，一旦 binlog 被清理，会导致同步失败，只能重新进行全量迁移；开启 relay log 后，binlog 数据会被实时拉取并写入到本地，与当前同步进度无关，可有效避免前面的问题。

但在 DM 版本 <=  v2.0.7 中，开启 relay log 后有如下问题：

1

数据同步延迟相比不开启 relay log 有明显上升，下面的表格是一个单 task 的 benchmark 的测试结果，可看出平均 latency 有明显的增长。下表中以“.”开头的是延迟的百分位数据。

2

开启 relay 后 CPU 消耗增加。（由于 latency 的增长，在一些简单场景下（比如只有 1 个 task）相比不开启 relay log，资源使用率反而是下降的。但当 task 增多时，开启 relay 的 CPU 消耗就增加了）。

[**点击此处查看**

**当前 relay 实现、Latency优化、****CPU 优化、遗留问题及未来工作的更多内容**](https://app.ma.scrmtech.com/material/read/index?new_id=330202&wx_id=1864 "https://app.ma.scrmtech.com/material/read/index?new_id=330202&wx_id=1864")

💡 点击【阅读原文】，立即试用 TiDB 企业版 ！