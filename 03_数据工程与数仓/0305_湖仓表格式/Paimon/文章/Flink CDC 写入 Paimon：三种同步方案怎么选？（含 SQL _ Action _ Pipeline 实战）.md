---
title: Flink CDC 写入 Paimon：三种同步方案怎么选？（含 SQL / Action / Pipeline 实战）
author: 三石大数据
date: 三石三石
url: https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247487625&idx=1&sn=c1e2afaec1aa3aadfb0708a4caf8e26b&chksm=fbc864a1ea29637446bc8f14fdaa4e10178ee28a1c1cb64a3356cfa2bb14fc1d3c7b5e79d442&mpshare=1&scene=24&srcid=0526ns2B1qRw8tjuRihjsJMV&sharer_shareinfo=a3690e444f5f432730b04eae49ba2d30&sharer_shareinfo_first=a3690e444f5f432730b04eae49ba2d30#rd
---

# 推荐阅读文章列表

[2025最新大数据开发面试笔记V7.0——试读](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247486661&idx=1&sn=dcf91fcdb56ffef0835a5f5cae0780a6&scene=21#wechat_redirect)

[没有实习经历，还有机会进大厂吗](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247486517&idx=1&sn=d4be6b7204151101c502d071aa48bac5&scene=21#wechat_redirect)

[简历指导套餐4.0——对标大厂的PB级数仓项目](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247486265&idx=1&sn=7b41a46f09a79ae1796c821ea628ad5a&scene=21#wechat_redirect)

[企业级项目一：金融信贷离线数仓建设](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247487216&idx=1&sn=e89b5756f3d218f0d6bc5ed571b3a614&scene=21#wechat_redirect)

[企业级项目二：基于Doris+LangChain构建数据智能运营AI助手](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247487169&idx=1&sn=1b59381a556784ee30fbd734270161ab&scene=21#wechat_redirect)

[企业级项目三：基于Paimon湖仓的AI数据分析平台](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247487613&idx=1&sn=6c0ab9a6265e92e1d3895615369333dd&scene=21#wechat_redirect)

# 前言

在做实时数仓的时候，Flink CDC → Paimon 基本是绕不开的一环。

但真正落地的时候，很多人都会踩同样的坑：

* 我用 Flink SQL 写了一套 CDC，结果上游一加字段就崩
* 整库同步看起来很爽，但上线后不知道怎么维护
* 用 Pipeline 方案，又遇到 schema 不一致、表结构对不齐的问题

本质问题只有一个：

> 你没搞清楚：Paimon CDC 到底有哪几种接入方式，以及各自适用场景。

# 1. 先搞清楚：有几种方式？

Paimon 接 CDC 数据，主流有3种方式，能力差异很大：

| 方式 | Schema Evolution | 整库同步 | 灵活度 | 推荐场景 |
| --- | --- | --- | --- | --- |
| 方式1：Flink SQL | ❌ 静态 schema | ❌ 需逐表写 | 高 | 单表、需要加工逻辑 |
| 方式2：Paimon Action（CDC Ingestion） | ✅ 支持 | ✅ 支持 | 中 | 整库同步、自动建表 |
| 方式3：Flink CDC Pipeline（3.0+） | ✅ 支持 | ✅ 支持 | 中 | 新版推荐 |

# 2. Flink SQL 单表同步

## 2.1 创建 Paimon Catalog

```
CREATE CATALOG paimon_catalog WITH (  
    'type' = 'paimon',  
    'warehouse' = 'hdfs:///data/paimon/warehouse'  
);  
  
USE CATALOG paimon_catalog;  
CREATE DATABASE IF NOT EXISTS dw_mall;  
USE dw_mall;
```

## 2.2 创建 CDC Source（Flink SQL）

```
CREATE TEMPORARYTABLE mysql_orders (  
    id         BIGINT,  
    user_id    BIGINT,  
    amount     DECIMAL(10,2),  
    status     STRING,  
    created_at TIMESTAMP(3),  
    updated_at TIMESTAMP(3),  
    PRIMARY KEY (id) NOTENFORCED  
) WITH (  
    'connector'     = 'mysql-cdc',  
    'hostname'      = 'localhost',  
    'port'          = '3306',  
    'username'      = 'cdc_user',  
    'password'      = 'cdc_pass',  
    'database-name' = 'trade_db',  
    'table-name'    = 'orders',  
    -- 快照阶段并行度，加速全量  
    'scan.incremental.snapshot.chunk.size' = '8096',  
    'scan.snapshot.fetch.size'             = '1024',  
    -- server id 范围（多并发需要）  
    'server-id' = '5400-5410'  
);
```

## 2.3 创建 Paimon Sink 表

```
CREATE TABLEIFNOTEXISTS dw_mall.ods_orders (  
    id         BIGINT,  
    user_id    BIGINT,  
    amount     DECIMAL(10,2),  
    status     STRING,  
    created_at TIMESTAMP(3),  
    updated_at TIMESTAMP(3),  
    dt         STRING,  
    PRIMARY KEY (id) NOTENFORCED  
)  
PARTITIONED BY (dt)  
WITH (  
    -- 合并引擎：按主键去重（保留最新一条）  
    'merge-engine' = 'deduplicate',  
  
    -- changelog 生产模式（透传 CDC 变更）  
    'changelog-producer' = 'input',  
  
    -- 写入性能优化  
    'write-buffer-size' = '256mb',  
    'page-size' = '64kb',  
  
    -- Bucket（建议一定要加，避免小文件 + 提升并发）  
    'bucket' = '8',  
  
    -- Compaction（控制小文件合并）  
    'num-sorted-run.compaction-trigger' = '5',  
    'compaction.min.file-num' = '3',  
    'compaction.max.file-num' = '10',  
  
    -- Snapshot（控制版本数量）  
    'snapshot.num-retained.min' = '5',  
    'snapshot.num-retained.max' = '20',  
    'snapshot.time-retained' = '1 h'  
);
```

## 2.4 写入Sink表

```
INSERT INTO dw_mall.ods_orders  
SELECT  
    id,  
    user_id,  
    amount,  
    status,  
    created_at,  
    updated_at,  
    DATE_FORMAT(created_at, 'yyyy-MM-dd') AS dt  
FROM mysql_orders;
```

## 2.5 总结

* Schema 是静态的：上游加列，这个作业不会自动跟进
* 需要手动改 source/sink DDL → 重新提交
* 适合：列稳定、需要加工逻辑（过滤/聚合/join） 的场景

# 3. Paimon Action CDC Ingestion（推荐整库同步）

> 这是 Paimon 官方提供的CDC 整库同步方案

## 3.1 单表同步

```
${FLINK_HOME}/bin/flink run \  
    -c org.apache.paimon.flink.action.FlinkActions \  
    paimon-flink-action-0.9.jar \  
    mysql-sync-table \  
    --warehouse hdfs:///data/paimon/warehouse \  
    --database dw_mall \  
    --table ods_orders \  
    --primary-keys id \  
    --partition-keys dt \  
    --computed-column 'dt=date_format(created_at, yyyy-MM-dd)' \  
    --mysql-conf hostname=localhost \  
    --mysql-conf port=3306 \  
    --mysql-conf username=cdc_user \  
    --mysql-conf password=cdc_pass \  
    --mysql-conf database-name=trade_db \  
    --mysql-conf table-name=orders \  
    --catalog-conf type=paimon \  
    --catalog-conf warehouse=hdfs:///data/paimon/warehouse \  
    --table-conf changelog-producer=input \  
    --table-conf merge-engine=deduplicate \  
    --table-conf write-buffer-size=256mb \  
    --table-conf snapshot.num-retained.min=5 \  
    --table-conf snapshot.num-retained.max=20
```

## 关键参数

| 参数 | 说明 |
| --- | --- |
| --primary-keys | Paimon 表主键 |
| --partition-keys | 分区键 |
| --computed-column | 计算列（如从 timestamp 派生 dt） |
| --table-conf | Paimon 表属性 |
| --mysql-conf | MySQL CDC 连接配置 |

## 自动建表

如果 Paimon 目标表不存在，会自动根据 MySQL 表结构建表。

## Schema Evolution

上游 MySQL `ALTER TABLE ADD COLUMN`，Paimon 表会自动加列。

## 3.2 整库同步（重点）

```
${FLINK_HOME}/bin/flink run \  
    -c org.apache.paimon.flink.action.FlinkActions \  
    paimon-flink-action-0.9.jar \  
    mysql-sync-database \  
    --warehouse hdfs:///data/paimon/warehouse \  
    --database dw_mall \  
    --mysql-conf hostname=localhost \  
    --mysql-conf port=3306 \  
    --mysql-conf username=cdc_user \  
    --mysql-conf password=cdc_pass \  
    --mysql-conf database-name=trade_db \  
    --catalog-conf type=paimon \  
    --catalog-conf warehouse=hdfs:///data/paimon/warehouse \  
    --table-conf changelog-producer=input \  
    --table-conf merge-engine=deduplicate \  
    --table-conf write-buffer-size=256mb \  
    --table-conf snapshot.num-retained.min=5 \  
    --table-conf snapshot.num-retained.max=20 \  
    --including-tables 'orders|users|payments' \  
    --excluding-tables 'tmp_.*|test_.*' \  
    --table-prefix 'ods_' \  
    --mode divided
```

## 关键参数

| 参数 | 说明 |
| --- | --- |
| --including-tables | 正则，要同步哪些表 |
| --excluding-tables | 正则，排除哪些表 |
| --table-prefix | 目标表名前缀，如 ods\_ |
| --table-suffix | 目标表名后缀 |
| --mode | divided（每表一个 sink）/ combined |

## 整库同步会做什么

1. 自动读取 MySQL 数据库的所有匹配表
2. 自动在 Paimon 中建表（按 MySQL 表结构）
3. 全量快照 + 增量 binlog 持续同步
4. 上游新增表：自动建表并同步
5. 上游新增列：自动`schema Evolution`

## Schema Evolution 支持的变更

| 变更类型 | 是否支持 |
| --- | --- |
| ADD COLUMN | ✅ |
| 类型扩展（如 INT → BIGINT） | ✅ 部分 |
| 改列名（RENAME COLUMN） | ❌ 通常不支持 |
| 删列（DROP COLUMN） | ❌ 通常忽略 |
| 改主键 | ❌ |
| 改分区 | ❌ |

# 4. Flink CDC 3.x Pipeline 模式

> Flink CDC 3.0 之后推出了 Pipeline 模式，也支持直接写 Paimon

## 4.1 定义 pipeline YAML

```
source:  
  type:mysql  
hostname:localhost  
port:3306  
username:cdc_user  
password:cdc_pass  
tables:trade_db.\.*  
server-id:5400-5410  
  
sink:  
type:paimon  
warehouse:hdfs:///data/paimon/warehouse  
database:dw_mall  
catalog.type:paimon  
table.prefix:ods_  
changelog-producer:input  
merge-engine:deduplicate  
  
pipeline:  
name:MySQLtoPaimonCDCPipeline  
parallelism:4  
schema.change.behavior:evolve
```

## 4.2 YAML任务提交

```
${FLINK_HOME}/bin/flink run \  
    -c org.apache.flink.cdc.cli.CliFrontend \  
    flink-cdc-pipeline-cli-3.1.0.jar \  
    mysql-to-paimon.yaml
```

# 5. 常见问题与解决方案

## 问题1：全量阶段太慢

```
-- 调大 chunk size（加速全量阶段）  
'scan.incremental.snapshot.chunk.size' = '16384';  
  
-- 提高并行度  
SET 'parallelism.default' = '8';  
  
-- 注意：  
-- 如果表没有主键/唯一索引，CDC 无法分片  
-- 需要确保 MySQL 源表有主键
```

## 问题2：Checkpoint 超时

```
-- Checkpoint 优化（降低频率 + 提高容忍度）  
SET 'execution.checkpointing.interval' = '120s';  
SET 'execution.checkpointing.timeout'  = '900s';  
  
-- 写缓冲（减少 checkpoint 状态体积）  
'write-buffer-size' = '128mb';
```

## 问题3：小文件过多

```
-- 调大目标文件  
'target-file-size' = '256mb'  
  
-- 严格 compaction  
'num-sorted-run.compaction-trigger' = '3'  
'compaction.max.file-num' = '5'  
  
-- 独立 compaction 作业定期跑
```

##问题4：OOM

```
-- 给 TaskManager 足够内存  
-Dtaskmanager.memory.process.size=8g  
-Dtaskmanager.memory.managed.fraction=0.4  
  
-- 用 RocksDB 后端  
SET 'state.backend.type' = 'rocksdb';  
  
-- 降低写缓冲  
'write-buffer-size' = '128mb'
```

## 问题5：Schema Evolution 没生效

## 问题6：数据延迟高

```
数据可见延迟 ≈ Checkpoint 间隔 + Compaction 延迟  
  
优化：  
1. 缩短 checkpoint 间隔（但不要太短，影响吞吐）  
2. changelog-producer 用 'input'（延迟最低）  
3. 如果下游是流读，确保 consumer-id 持续消费
```

# 6. 总结：该用哪种方式？

# 写在最后

### Paimon x AI项目获取方式

> 后台回复：AI项目