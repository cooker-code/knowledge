> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040102_Doris/040102_核心知识点/DorisBE内存问题HeapProfile排查|DorisBE内存问题HeapProfile排查]]
---
title: Apache Doris BE 内存问题深度排查：从监控异常到 Heap Profile 精准定位
author: 数栖云间
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzMTAwMTcwOA==&mid=2247484316&idx=1&sn=6025954400eec2d26694a1aa024e336a&chksm=c3cddfde9a691a8bd0ad4ab830898ea9d6d257ce325aaf7956a1cec2d9aa9deb667d90984fed&mpshare=1&scene=24&srcid=0411zGV0rrOTDZxHJq7QxRKV&sharer_shareinfo=aa0ae41b0d1e27aa2a73426a7b529873&sharer_shareinfo_first=aa0ae41b0d1e27aa2a73426a7b529873#rd
---

> SelectDB：Apache Doris 原厂服务
>
> 免运维 · 多云 · SLA保障
>
> 私有化/SaaS 任选，专业团队兜底
>
> 欢迎大佬们来交流，让Doris更稳定 👉

# 一、背景

在 Apache Doris 的架构中，BE（Backend）是真正承担计算与存储的核心节点。它同时处理查询执行、数据导入、Compaction、缓存管理等多类任务，这意味着 BE 的内存压力来自多个并发维度，而不是单一来源。每一个查询、每一个导入任务、每一次后台合并，都在争夺有限的内存资源。

从 Doris 1.2.2 版本开始，BE 默认切换到 **Jemalloc** 作为内存分配器，替代了原有的 TCMalloc。Jemalloc 在多线程场景下的内存碎片控制更优，能够更好地应对高并发环境。但这项优化也带来了一个新问题：**内存的实际占用与业务逻辑之间的映射关系变得更难直接观察**。Jemalloc 的 Arena 机制、线程缓存等设计，使得内存分配变得复杂而抽象。

这就是为什么当你在监控上看到 BE 内存缓慢爬升、最终打满时，单靠 `top` 或 `free` 命令根本无法定位根因——这时候需要一种能够还原内存分配调用栈的剖析工具 Heap Profile。只有深入到函数调用级别，才能看清究竟是哪个模块、哪段代码在不断吞噬内存。

> 在现代分布式系统中，内存问题的本质不是“不够用”，而是“被谁用、为什么没还回来”。Heap Profile 就是那把让你看清楚的放大镜。

---

## 二、判断泄漏，工具介入

### 2.1 如何判断是内存泄漏而非正常增长

BE 内存增长分两类，排查前需要先做区分：

**正常增长**：

* 查询并发上升，查询内存池扩张
* 导入任务增多，MemTable 占用增加
* Page Cache、BloomFilter 等缓存随数据量增长而扩大
* Compaction 任务积压，临时内存占用上升

**内存泄漏的典型特征**：

* 集群负载（查询 QPS、导入量）没有明显变化，甚至处于业务低谷
* BE 内存持续单调递增，不随任务结束而回落
* 重启 BE 后内存恢复正常，但运行一段时间后又开始爬升
* 最终触发 OOM 或 BE 进程被系统 Kill

**判断方法**：**对比监控曲线**。如果内存曲线与业务负载曲线完全解耦——业务低谷时内存仍在涨——则高度怀疑存在泄漏。这种“脱缰”的内存增长，往往意味着某个对象生命周期管理出了问题。

### 2.2 Jemalloc Heap Profile 的工作原理

Jemalloc 内置了采样式内存剖析能力，它通过在分配路径上埋点，定期记录调用栈信息。其核心参数如下：

| 参数 | 含义 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `prof:true` | 开启 Heap Profile 采集 | false | 必须显式开启 |
| `lg_prof_sample` | 采样粒度，2^N 字节采样一次 | 19（512KB） | 越小精度越高，开销越大 |
| `lg_prof_interval` | 累计分配多少字节后自动 dump | 32（4GB） | 控制 dump 文件生成频率 |
| `prof_gdump` | 内存达到历史峰值时自动 dump | false | 可用于追踪峰值场景 |
| `prof_leak` | 进程退出时 dump 泄漏对象 | false | 需结合 `prof_final` 使用 |

**采样原理**：Jemalloc 不记录每一次 `malloc`，而是每隔 `2^lg_prof_sample` 字节采样一次，记录当时的调用栈。这种采样设计使得 Profile 的性能开销极低（通常 < 5%），可以在生产环境短暂开启。采样不是计数，而是按概率记录，因此最终结果反映的是内存分配的统计分布，而非精确值。

**jeprof 分析原理**：`jeprof` 工具读取 `.heap` dump 文件，将采样数据与 BE 二进制的符号表（`doris_be`）结合，还原出每个函数调用路径的内存分配量。其核心是地址符号化——将内存地址转换为函数名和行号。最终输出为 DOT 格式的有向图，直观展示调用关系和分配量。

### 2.3 Realtime Dump vs Regular Dump 的架构差异

Doris 提供了两种 Heap Dump 触发方式，适用于不同场景，理解其差异有助于选择正确方案：

**Realtime Heap Dump**：

* 通过 HTTP 接口按需触发，不依赖定时写文件
* 对磁盘无持续压力，仅在需要时生成 dump
* 需要 BE 的 Web 端口（默认 8040）可达
* **优点**：灵活可控，不会意外打爆磁盘
* **缺点**：依赖网络连通性，无法在端口不通的环境使用

**Regular Heap Dump**：

* 通过 `be.conf` 配置 `JEMALLOC_PROF_PRFIX` 开启定时写文件
* 每累计分配 `2^lg_prof_interval` 字节自动写一个 `.heap` 文件
* **风险**：高导入/查询场景下可能频繁写文件，需要合理调大 `lg_prof_interval`，否则可能打爆磁盘
* **优点**：不依赖网络，自动持续记录
* **缺点**：磁盘空间管理成为新的运维负担

Realtime 模式遵循“按需采集”原则，Regular 模式则是“持续记录”思路。在云原生环境中，Realtime 模式更符合现代运维理念。

---

## 三、完整排查操作步骤

### Step 1：确认内存异常

在开始任何 profiling 之前，先确认现象是否真的异常：

```
# 查看 BE 进程内存占用（RSS）
ps aux | grep doris_be

# 查看 BE 内存使用情况，可以通过监控进行观测，或者通过 metrics 接口查看
curl http://be_host:8040/metrics | grep "doris_be_memory_allocated_bytes"
```

通过监控平台确认：BE 内存在无明显负载变化的情况下持续增长。这一步的关键是**排除干扰**——确认不是由于业务负载上升导致的正常内存波动。如下图所示：

### Step 2：开启 Jemalloc Heap Profile 采集

#### 方案 A：Realtime Heap Dump

```
# 开启 profile 采集
curl -u username:password http://127.0.0.1:8040/jeheap/active/true

# 验证端口是否可达
curl -u username:password http://be_host:be_webport/jeheap/dump

# 如果开启成功，执行 dump 会返回文件路径
```

修改 `be.conf` 中的 `JEMALLOC_CONF`，将 `prof:false` 改为 `prof:true`，然后重启 BE：

```
# be.conf 中修改
JEMALLOC_CONF="percpu_arena:percpu,background_thread:true,metadata_thp:auto,\
muzzy_decay_ms:15000,dirty_decay_ms:15000,oversize_threshold:0,\
lg_tcache_max:20,prof:true,lg_prof_interval:32,lg_prof_sample:19,\
prof_gdump:false,prof_accum:false,prof_leak:false,prof_final:false"
```

重启 BE 后，即可通过 HTTP 接口触发 dump。

#### 方案 B：Regular Heap Dump

修改 `be.conf`，同时设置 `prof:true` 和 `JEMALLOC_PROF_PRFIX`：

```
JEMALLOC_CONF="percpu_arena:percpu,background_thread:true,metadata_thp:auto,\
muzzy_decay_ms:15000,dirty_decay_ms:15000,oversize_threshold:0,\
lg_tcache_max:20,prof:true,lg_prof_interval:32,lg_prof_sample:19,\
prof_gdump:false,prof_accum:false,prof_leak:false,prof_final:false"

JEMALLOC_PROF_PRFIX="jemalloc"
```

重启 BE 后，dump 文件自动写入 `${DORIS_HOME}/log/` 目录，文件名前缀为 `jemalloc`，格式为 `jemalloc.<pid>.<seq>.heap`。

**关键参数调优**：

`lg_prof_interval` 默认值 32，即累计分配 2^32 = 4GB 触发一次 dump。如果集群内存很大且导入量高，需要适当调大，避免频繁写文件。以下是一些参考值：

| 集群内存规模 | 建议 lg\_prof\_interval | dump 触发阈值 | 适用场景 |
| --- | --- | --- | --- |
| < 64GB | 32（默认） | 4GB | 小型集群，写入量不大 |
| 64GB ~ 256GB | 34 | 16GB | 中型集群，中等写入负载 |
| > 256GB | 36 | 64GB | 大型集群，高写入负载 |
| 导入密集型 | 38 | 256GB | 极端写入场景，避免频繁 IO |

> **注意**：`lg_prof_interval` 是 2 的指数，每增加 1，触发阈值翻倍。调大此值可降低 dump 频率，但会降低时间分辨率。

### Step 3：使用 jeprof 分析 Dump 文件

将 `jeprof` 工具上传到 BE 所在机器（通常 Doris 发行版自带，位于 `bin/` 目录下），执行以下命令：

```
# 1. 分析单个 Heap Profile 
./jeprof --dot /opt/selectdb/be/lib/doris_be lowmem.heap > lowmem.dot

# 2. 通过在一段时间内多次执行 Heap Dump，可以生成多个 heap 文件。选取较早时间的 heap 文件作为 baseline，与较晚时间的 heap 文件进行对比分析 diff。

# 分析低内存时的 dump 
./jeprof --dot /opt/selectdb/be/lib/doris_be lowmem.heap > lowmem.dot

# 分析高内存时的 dump
./jeprof --dot /opt/selectdb/be/lib/doris_be highmem.heap > highmem.dot

# 最关键：diff 两个 dump，找出增量内存的来源
./jeprof --dot /opt/selectdb/be/lib/doris_be --base=lowmem.heap highmem.heap > diff.dot
```

.dot 内容举例

```
Dropping nodes with <= 196.8 MB; edges with <= 39.4 abs(MB)
digraph "/opt/selectdb/be/lib/doris_be; 39368.9 MB" {
node [width=0.375,height=0.25];
Legend [shape=box,fontsize=24,shape=plaintext,label="/opt/selectdb/be/lib/doris_be\lTotal MB: 39368.9\lFocusing on: 39368.9\lDropped nodes with <= 196.8 abs(MB)\lDropped edges with <= 39.4 MB\l"];
N1 [label="__clone@@GLIBC_2.2.5\n0.0 (0.0%)\rof 24855.2 (63.1%)\r",shape=box,fontsize=8.0];
N2 [label="pthread_condattr_setpshared@GLIBC_2.2.5\n0.0 (0.0%)\rof 24855.2 (63.1%)\r",shape=box,fontsize=8.0];
N3 [label="doris\nDeltaWriter\ninit@c4ef400\n0.0 (0.0%)\rof 19417.4 (49.3%)\r",shape=box,fontsize=8.0];
N4 [label="std\n_Function_handler\n_M_invoke@c4f6ff0\n0.0 (0.0%)\rof 17487.1 
...... 省略
N79 -> N78 [label=1686.6, weight=100000, style="setlinewidth(0.257044)"];
N70 -> N80 [label=1646.7, weight=100000, style="setlinewidth(0.250959)"];
N72 -> N28 [label=1072.3, weight=100000, style="setlinewidth(0.163427)"];
N72 -> N27 [label=51.0, weight=100000, style="setlinewidth(0.007778)"];
} 
```

**参数说明**：

* `--dot`：输出 Graphviz DOT 格式，便于可视化
* `--base`：指定基线 dump，diff 模式下只展示增量分配
* 第二个参数为 BE 二进制路径

尽可能在同一台机器上完成 Dump Heap 和 `jeprof` 解析的操作，即尽可能在运行 Doris BE 的机器上直接解析 Heap Profile。或者确认运行 Doris BE 的机器 Linux 内核版本，将 `be/bin/doris_be` 二进制文件和 Heap Profile 文件下载到相同内核版本的机器上执行 `jeprof`。

### Step 4：可视化分析调用图

将 `.dot` 文件内容粘贴到在线 Graphviz 工具（如 http://www.webgraphviz.com/）或本地工具生成调用图。

从图中可以直观的看到占用内存较大的部分，如上图所示就能判断出和inverted index 相关。

### Step 5：清理与收尾

**重要**：分析完成后必须关闭 Profile 采集，否则持续写 dump 文件会打爆磁盘。这是一个常见的“二次伤害”场景——排查内存泄漏，结果导致磁盘写满。

---

## 四、行业启示与排查思维框架

从这套排查流程中，可以提炼出一个通用的 BE 内存问题分析框架：

```
监控异常（内存持续增长）
    ↓
区分正常增长 vs 泄漏（对比负载曲线）
    ↓
文本 Profile 快速定位模块（/profile 接口）
    ↓
Heap Dump 精准定位调用栈（jeprof diff）
    ↓
识别根因（对象生命周期 / 缓存未释放 / 数据结构膨胀）
    ↓
提交社区同学分析 + 关闭 Profile 采集
```

之后遇到内存相关的问题，可以先按照这个思维框架进行排查，这样能快速定位到是哪里导致的问题，达到事半功倍的效果。

欢迎大佬们多多交流～

**完**

数栖云间

【数栖云间｜Apache Doris 技术生态圈】

**专注 Apache Doris 深度赋能**
聚焦 **运维实践、核心原理、前沿动态** ，为您提供：
✅ **运维实战指南**：生产环境调优、故障排查、集群管理等硬核经验
✅ **原理深度解析**：存储引擎、查询优化、分布式架构等技术揭秘
✅ **前沿动态追踪**：版本特性、社区动向、行业应用案例一手资讯
✅ **资源宝库**：白皮书/源码笔记/性能优化手册，助你从入门到精通

 扫码关注，解锁技术全景视角 ⤵️

**关于社区**

Apache Doris

Apache Doris 是一个基于 MPP 架构的高性能、实时的分析型数据库，以极易易用的特点被人们所熟知，仅需亚秒级响应时间即可返回海量数据下的查询结果，不仅可以支持高并发发点查询场景，也能支持高吞吐的复杂分析场景。

如果您对 Apache Doris 感兴趣，可以通过以下入口访问官方网站、社区论坛、GitHub 和 dev 邮件组：

* 官方文档: https://doris.apache.org
* 社区论坛: https://ask.selectdb.com
* GitHub: https://github.com/apache/doris
* dev 邮件组: dev@doris.apache.org

可以加 **作者微信 (Liyy\_222\_8993)** 直接进 Doris 官方社区群。