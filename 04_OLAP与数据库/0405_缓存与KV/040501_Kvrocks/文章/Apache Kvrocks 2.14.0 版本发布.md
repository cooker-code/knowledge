---
title: Apache Kvrocks 2.14.0 版本发布
author: Apache Kvrocks
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUxNTg5NzM1Nw==&mid=2247484266&idx=1&sn=14b629fdfb9f440caaa09eb03f7cd138&chksm=f88856d831083ccea017aeaf8e0f79e42add766cb8a0c9cc181ab963ddbcae3f325f6b00e132&mpshare=1&scene=24&srcid=1125UEQwYiD8FRDAdvOOcD1S&sharer_shareinfo=2a22b0281beb882bb260c3a36833cb8c&sharer_shareinfo_first=2a22b0281beb882bb260c3a36833cb8c#rd
---

# Apache Kvrocks 2.14.0 版本发布

Apache Kvrocks 社区近日发布了 2.14.0 版本。作为一款基于 RocksDB 构建的分布式键值（Key-Value）NoSQL 数据库，Kvrocks 兼容 Redis 协议，旨在通过将数据存储在磁盘（SSD）上来降低内存成本并提升数据容量，同时维持与 Redis 类似的访问接口。

本次 2.14.0 版本主要在数据一致性选项、统计类数据结构支持以及脚本执行并发度方面进行了功能扩展与优化。以下是该版本的详细变更介绍。

## 核心特性更新

### 1. 支持 WAIT 命令：同步复制能力的补齐

在 2.14.0 之前，Kvrocks 主要采用异步复制模式。为了满足部分业务对数据一致性的更高要求，新版本引入了 `WAIT` 命令。该命令会阻塞当前客户端连接，直到之前的写操作被成功同步到指定数量的副本节点中，或者达到设定的超时时间。

这一功能允许开发者在需要强一致性的关键写入场景（如金融交易记录）中，显式地确认数据已复制到从节点，从而在系统故障时提供更好的数据安全性保障。

**使用示例：**

```
> SET mykey "value"  
OK  
  
# 等待写操作同步到至少 1 个副本，超时时间为 1000 毫秒  
> WAIT 1 1000  
(integer) 1
```

### 2. 新增 T-Digest 数据结构

针对延时监控、流量分析等场景中常见的百分位数计算（如 P99, P95）需求，Kvrocks 2.14.0 原生支持了 T-Digest 概率数据结构。

T-Digest 是一种空间效率极高的算法，能够在极小的内存占用下，对流式数据的秩（Rank）和百分位数（Quantile）进行高精度的估算。直接在数据库层面支持该结构，意味着用户无需将大量原始数据拉取到应用层计算，也无需维护额外的时序数据库组件即可获取分布统计信息。

**支持命令包括 `TDIGEST.CREATE`, `TDIGEST.ADD`, `TDIGEST.QUANTILE` 等。使用示例：**

```
# 创建一个名为 'latency' 的 T-Digest 结构  
> TDIGEST.CREATE latency  
  
# 添加一组延时样本数据  
> TDIGEST.ADD latency 10.5 20.1 15.3 99.8  
  
# 计算 P99 (99th percentile)  
> TDIGEST.QUANTILE latency 0.99  
"99.8"  
  
# 查看最小值和最大值  
> TDIGEST.MIN latency  
"10.5"  
> TDIGEST.MAX latency  
"99.8"
```

### 3. Lua 脚本严格模式与并发执行

为了提升脚本执行效率，新版本引入了 Lua 脚本的“严格键访问模式”（Strict Key-Access Mode）。

在传统模式下，为了保证原子性，脚本执行往往会阻塞服务。而在严格模式下，Kvrocks 强制要求用户在执行 `EVAL` 或 `FCALL` 时，必须显式在命令行中声明脚本所需访问的所有 Key。基于这些声明，Kvrocks 能够识别脚本的数据依赖范围。对于操作不同 Key、互不冲突的脚本，系统现在可以并行执行。这一改进能够有效利用多核 CPU 资源，提升高并发场景下的整体吞吐量。

**使用示例：**

```
-- 脚本中只能访问 KEYS 数组中声明的键  
-- 如果尝试访问未声明的 'key2'，在严格模式下会报错  
> EVAL "return redis.call('get', KEYS[1])" 1 key1  
"value1"
```

## 其他改进与修复

除了上述主要特性外，2.14.0 版本还包含以下底层更新：

* • **RocksDB 升级**：底层存储引擎 RocksDB 版本升级至 **10.6.2**，引入了上游的性能优化与缺陷修复。
* • **RISC-V 架构支持**：新增了对 RISC-V 架构的编译与运行支持，扩展了 Kvrocks 的硬件适配范围。
* • **Search 功能优化**：增强了 Search 模块，新增 Compaction Filter，支持在查询过程中自动跳过并清理已过期的 Key，减少无效数据对查询性能的影响。
* • **复制稳定性**：针对主从复制流程进行了多项改进，提升了在网络环境不稳定时的同步效率和系统稳定性。

## 参考资料

* • **Release Notes**: https://github.com/apache/kvrocks/releases/tag/v2.14.0
* • **官方网站**: https://kvrocks.apache.org/
* • **GitHub 仓库**: https://github.com/apache/kvrocks