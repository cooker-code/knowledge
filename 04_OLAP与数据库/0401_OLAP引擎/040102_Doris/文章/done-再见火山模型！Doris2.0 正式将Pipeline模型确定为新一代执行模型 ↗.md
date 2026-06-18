> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040102_Doris/040102_核心知识点/Doris架构与Pipeline执行边界|Doris架构与Pipeline执行边界]]
---
title: 再见火山模型！Doris2.0 正式将Pipeline模型确定为新一代执行模型 ↗
author: 大数据技能圈
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247490228&idx=1&sn=bab1d3d5d29f409baae0dcd5ccf6dd8d&chksm=c11e21bcd05f8cf2e6e6dd3f7d0246f5fb5eee38b083580ce102c60b72147dac2e3a990e3cce&mpshare=1&scene=24&srcid=1106YmB6SzjSc8BF8tdQiMwb&sharer_shareinfo=19b943a702718c108a54cba913b3cbf2&sharer_shareinfo_first=19b943a702718c108a54cba913b3cbf2#rd
---

在现代数据库系统中，执行引擎在数据库体系结构中起着承上启下的作用，与查询优化器和存储引擎共同组成了数据库的三大模块。我们以 SQL 语句在数据库系统中的完整执行过程为例，来介绍执行引擎在其中发挥的作用：

* 在接收到一条 SQL 查询语句之后，查询优化器会对 SQL 进行语法/词法分析，基于代价模型和规则生成最优执行计划；
* 执行引擎会将生成的执行计划调度到计算节点，按照最优执行计划对底层存储引擎中的数据进行操作并返回查询结果；

在整个查询过程中，查询执行是至关重要的环节，往往需要通过数据读取、过滤、排序、聚合等操作，才能提交给执行引擎进行下一步查询，这几个步骤的设计是否合理直接影响到查询的性能及资源的利用率。而这些能力均由执行模型来提供，不同的执行模型在数据处理、查询优化和并发控制等方面存在较大差异，因此，一个合适的执行模型对于提高查询效率和系统性能至关重要。

目前业界常见的执行模型有迭代模型/火山模型（Iterator Model）、物化模型（Materialization Model）、向量化/批处理模型（Vectorized / Batch Model）。其中火山模型（Volcano Model）是数据库查询优化和执行中最为常用的执行模型。每一种操作抽象为一个 Operator，整个 SQL 查询被构建成一个 Operator 树。查询执行时，树自顶向下调用`next()`接口，数据则自底向上被拉取处理，因此这种处理方式也被称为拉取执行模型（Pull Based）。火山模型因其具有很高灵活性高、可扩展性好、易于实现和优化等特性，被广泛应用于数据库查询优化和执行中。

作为典型的 MPP 数据库，过去版本中 Apache Doris 亦采取的也是火山模型。当用户发起 SQL 查询时，Apache Doris 会将查询解析成分布式执行计划并分发到执行节点执行，分发到节点的单个执行任务被称为 Instance，在此我们一条简单的 SQL 查询来了解 Instance 在火山模型下的执行过程：

```
```
select age, sex from employees where age > 30
```
```

如上图可知，Instance 是一个算子（ExecNode）树，算子之间通过数据重分布（Exchange）算子连接起来，从而实现数据流的传递和处理，每个算子实现`next()`方法。当对算子的`next()`方法进行调用时，该算子将调用其孩子算子的`next()`方法来获取输入的数据，然后对数据进行逻辑加工并输出。而因为算子的 `next()` 方法是同步方法，在没有数据产生时，`next()`方法将会持续阻塞。这时候需要循环调用根节点算子的 `next()` 方法，直到全部数据处理完毕，即可得到整个 Instance 的计算结果。

从上述执行过程可以看出，火山模型是一种简单易用、灵活性高的执行模型，但在单机多核的场景下，存在一些问题需要进一步解决和优化，具体体现在以下几方面：

* 线程阻塞执行：在线程池大小固定的情况下，当一个 Instance 占用一个线程阻塞执行时，如果存在大量的 Instance 同时请求，执行线程池将被占满，从而导致查询引擎出现假死状态，无法响应后续请求。特别是在存在 Instance 之间相互依赖的情况下，还可能会出现逻辑死锁的情况，比如当前线程中正在执行的 Instance 依赖于其他的 Instance，而这些 Instance 正处于等待队列中，无法得到执行，从而加剧系统的负载和压力。当一个执行节点同时运行的 Instance 线程数远大于 CPU 核数时，Instance 间的调度将依赖于系统调度机制，这就可能产生 Context 切换开销，尤其是在系统混部的场景中，线程切换的开销会更加显著。
* CPU 资源抢占：Instance 线程之间出现争抢 CPU 资源的问题，可能导致不同大小的查询、不同租户之间互相影响。
* 无法充分利用多核计算能力：执行计划的并行度取决于数据分布，当一台执行节点上存在 N 个数据分桶时，该节点上运行的 Instance 数量不能超过 N，因此分桶的设置显得尤为重要。如果分桶设置过少，难以充分利用多核计算能力，反之，则会带来碎片化问题。多数场景下进行性能调优时需要手动设置并行度，而在生产环境中，预估数据分桶数是一项极具挑战性的任务，不合理的分桶使得 Doris 的性能优势无法得到充分发挥，无法充分利用多核计算能力。

# **Pipeline 执行模型的引入**

---

为了解决过去版本存在的问题，Apache Doris 自 2.0 版本起引入了 Pipeline 执行模型以替换过去的火山模型，并在 2.1 版本对 Pipeline 执行模型进行了进一步的升级。

以 Join 场景为例，下图展示了 Pipeline 执行模型下两个 Instance 组成查询计划的效果。

# 在这个计划中，Join 的 Probe 操作依赖于哈希表的构建操作（Build），因此 Build 操作必须在 Exchange 获取的数据全部处理完成并构建完哈希表之后才能启动，这种依赖关系导致每个 Instance 被拆分成两个 Pipeline Task。Pipeline 调度器将 Pipeline Task 放置于工作线程池的 Ready 队列，工作线程根据不同的策略获取 Pipeline Task，Pipeline Task 计算完成一个数据块后是否让出线程取决于其前置数据是否 Ready 以及运行时间是否超过上限。

# **Pipeline 执行模型的设计实现**

---

Pipeline 执行模型通过阻塞逻辑将执行计划拆解成 Pipeline Task，将 Pipeline Task 分时调度到线程池中，实现了阻塞操作的异步化，解决了 Instance 长期占用单一线程的问题。同时，我们可以采用不同的调度策略，实现 CPU 资源在大小查询间、不同租户间的分配，从而更加灵活地管理系统资源。Pipeline 执行模型还采用了数据池化技术，将单个数据分桶中的数据进行池化，从而解除分桶数对 Instance 数量的限制，提高 Apache Doris 对多核系统的利用能力，同时避免了线程频繁创建和销毁的问题，提高了系统的并发性能和稳定性。

## **01  去阻塞化改造**

从上文介绍可知，在之前版本的火山模型下，执行引擎存在阻塞操作，这会带来两个核心问题：一是阻塞线程过多会导致线程池打满，无法响应后续查询；二是执行线程调度完全依赖操作系统，无法根据查询优先级进行调度，性能有待提升。为了解决这两个问题，我们重新设计了去阻塞化的执行逻辑。

针对第一个问题，我们固定一个大小与 CPU 核数相同的执行线程池，并保证执行线程中不会存在阻塞操作。为了避免线程阻塞导致操作系统级别的线程调度，我们在所有发生阻塞的算子中拆分了 Pipeline Task，比如使用独立线程进行磁盘 I/O 和 RPC 等操作。

针对第二个问题，我们设计了一个纯用户态的轮询调度器，通过不停轮询所有可执行 Pipeline Task 的状态，将当前需要执行的 Task 交给执行线程执行。这种做法避免了操作系统频繁线程切换的开销，同时也可以加入更多优先级等定制化的调度策略，提高系统灵活性和可扩展性。

## **02  并行化改造**

在 2.0 之前版本中，Apache Doris 执行引擎的并发度需要由用户手动设置（即会话变量`parallel_fragment_exec_instance_num`），无法根据不同的 Workload 进行动态调整。而为了设置一个合理的并发度，往往需要进行细致的分析，这无疑是增加了用户的负担。同时，使用不合理的并发度可能会导致性能问题。因此，如何充分利用机器资源来实现每个查询任务的自动并发，成为亟需解决的问题。

当前常见的 Pipeline 并发方案分别以 Presto、DuckDB 为代表，Presto 并发方案是在执行过程中将数据 Shuffle 成合理的分区数量，这样做的好处是基本不需要特别的并发控制。DuckDB 并发方案执行过程中不会引入额外的 Shuffle 操作，但是需要引入额外的同步机制。我们对以上方案进行了综合对比，我们认为 DuckDB 并发方案在实现上很难规避使用锁，而锁的存在有悖于我们去阻塞化改造的思路，因此我们选择了以 Presto 为代表的实现方案。

为了实现 Pipeline 并发，Presto 引入了 Local Exchange 对数据进行了重分区，例如对于 Hash Aggregation，Presto 根据聚合 Key 进一步将数据分为 N 份，这样就可以充分利用机器的 N Cores，每个执行线程只需要构建更小的 Hash Table。而对于 Apache Doris，我们选择充分利用 MPP 自身的架构，在 Shuffle 时就直接将数据分区成合理的分区数，因此不再需要额外引入 Local Exchange。

基于这个特性，我们需要对两个方面进行改造：一是在 Shuffle 时增加并发，二是在 Scan 层读取数据后实现并发执行能力。对于前者，我们只需要在 FE 感知 BE 环境，然后设置合理的分区数即可。而对于后者，目前 Doris 在 Scan 层的执行线程与存储 Tablet 数量是强绑定的，因此需要重构 Scan 层并发逻辑，以满足我们的需求。

Scan 池化的基本思路是将 Scanner 线程读取的数据进行池化，多个 Pipeline Task 可以直接从池中取数据执行。这样的方式可以充分解耦 Scanner 和执行线程，提高系统的并发性能和稳定性。

# **Pipeline 执行模型的进一步完善**

---

Pipeline 执行模型的引入为 Apache Doris 在混合负载场景中的查询性能和稳定性都得到了明显提升，但在 Apache Doris 2.0 版本中仍为实验性功能，在社区用户使用的过程中，一些新的问题开始浮现：

* 执行并发受限：由于当前版本 Doris 执行并发仍收到 FE 设置的静态并发参数和存储层 Tablet 数量限制，使得执行引擎无法充分利用机器的多核资源，同时存储层可能会存在数据倾斜问题，导致查询执行出现长尾。
* 执行开销较大：表达式各 Instance 相互独立，而 Instance 的初始化参数存在大量公共部分，这导致每次执行都需要额外进行重复的初始化步骤，显著增加了执行开销。
* 调度开销较大：在查询执行过程中，当前调度器会把阻塞 Task 全部放入一个阻塞队列中，由一个线程负责轮询并从阻塞队列中取出可执行 Task 放入 Runnable 队列，所以在有查询执行的过程中，会固定有一个核的资源作为调度的开销。尤其是在一些小机型上，固定调度线程带来的开销非常明显。
* Profile 可读性差：Pipeline Profile 指标缺乏直观性和可读性，使得性能分析变得比较困难。

为了提供更高的查询性能和更稳定的查询体验，**Apache Doris 在最新发布的 2.1 版本中，对 Pipeline 执行模型进行大幅优化，将其改造为基于事件驱动的执行模型，并对已存在问题提供了改进方案**。为便于理解，后文将改进后的 Pipeline 执行模型称为 PipelineX。

## **01  执行并发改造**

前文提及，Pipeline 执行并发受两个因素制约：FE 设置的静态并发参数和存储层 Tablet 数量限制，这就导致执行引擎无法充分利用机器资源。另外如果数据本身存在倾斜，还可能导致查询执行时出现长尾问题。为此，我们以一个简单的聚合查询为例展开详细介绍。

假定有 Table A，Table A 中 tablet 总数为 1 ，共有数据 100M 行，执行聚合查询：

```
```
 SELECT COUNT(*) FROM A GROUP BY A.COL_1;
```
```

一般而言，在查询 SQL 的完整执行过程中，查询会被切分成为多个**查询分片（Fragment）**，每个查询分片表示查询执行过程中的逻辑概念，可能包含多个 SQL 算子。当 BE 收到 FE 下发的 Fragment 后，启动多个执行线程并行执行 Fragment，确保每个 Fragment 均能得到高效处理。如下图，Doris 将其切分成了 2 个 Fragment 分别执行：

为便于理解，仅介绍逻辑计划的第一部分（Fragment 0）。由于 Table A 只有一个 Tablet，因此 Fragment 0 的执行并发始终被限制为 1，即由单线程完成 100M 行数据的聚合。而在理想状态下，16 核可承载并发数为 8，假定执行时间为 x，每个执行线程可读取 100M/8 行数据，那么执行时间约为 x/8。然而在该例子中，大约会带来 8 倍的性能损失。

为解决这一问题，**Apache Doris  2.1 版本在执行引擎中引入了 Local Shuffle 节点，摆脱了存储层 Tablet 数量对执行并发的限制**。具体实现上：

* 执行线程执行各自的 Pipeline Task，而 Pipeline Task 仅持有一些运行时状态（即 Local State）。全局信息则由多个 Task 共享的同一个 Pipeline 对象持有（即 Global State）。
* 在单个 BE 上，数据分发由 Local Shuffle 节点完成，并由 Local Shuffle 保证多个 Pipeline Task 间的数据均衡。

上述问题阐述了 PipelineX 执行引擎如何摆脱 Tablet 数量的限制，除此之外，**Local Shuffle 还可以规避数据倾斜带来的长尾查询问题**。我们仍假定使用上面的聚合查询，将 Table A 的 Tablet 数量改为 2，其中 Tablet 1 有 10M 行数据、Tablet 2 有 90M 行数据：

* Pipeline 引擎：在改造之前（下图左），当执行 Fragment 1 时，Thread 2 的执行时间约为 Thread 1 的 9 倍。
* PipelineX 引擎：在改造之后（下图右），Local Shuffle 会把这 100M 行数据均匀地分发给 2 个执行线程，使其不再受存储层数据倾斜的影响，执行时间相同。

## **02  执行流程改造**

上文中提到，表达式各 Instance 相互独立，而 Instance 的初始化参数存在大量公共部分，这导致每次执行都需要额外进行重复的初始化步骤。为了降低不必要的执行开销，**PipelineX 对共享状态进行了复用**，将 Pipeline 执行流程中的第 3 步拆分为 Pipelinex 执行流程中的第 3 步和第 5 步。这样就可以只对较重的 Global State 进行一次初始化，而对更轻量级的 Local State 进行串行初始化。

## **03  调度模型改造**

Pipeline 调度过程中，就绪 Task 保存在就绪队列中等待调度、阻塞 Task 保存在阻塞队列中等待满足执行条件，因此额外需要一个 CPU Core 去轮询阻塞队列，如果 Task 满足执行条件则保存在就绪队列中。**而 PipelineX 将阻塞条件通过 Dependency 封装，Task 的阻塞/就绪状态完全依赖于事件通知**。当 RPC 数据到达时，将触发 ExchangeSourceOperator 满足执行条件，并进入就绪队列。

**PipelineX 对执行调度的核心改造就是引入了事件驱动**，一个查询被分割为多个 Pipeline，所有的 Pipeline 组成一个有向无环图（DAG）。以 Pipeline 为点、上下游 Pipeline 彼此的依赖作为边，我们将所有边抽象为 Dependency，每个 Pipeline 是否可以执行取决于其所有的 Dependency 是否满足执行条件。继续以简单聚合查询为例，查询被切分成如下 DAG：

简单起见，图上只标明了 Pipeline 上下游之间构成的 Dependency。事实上，Pipeline 所有的阻塞条件都被抽象成为了 Dependency，例如 Scan Node 依赖 Scanner 读取数据才可以执行，这一部分同样被抽象成为 Dependency 作为 Pipeline 0 是否可以执行的条件。

对于每个 Pipeline 来说，执行流程图如下：

在经由事件驱动的 PipelineX 改造后，每个 Pipeline Task 在执行前都会判断所有的执行条件是否满足。当所有依赖关系都满足执行条件时，Pipeline 被执行。当有条件不满足时，Task 会被添加到相应 Dependency 的阻塞队列中。当有外部事件到达时，所有阻塞 Task 重新判断执行条件，条件满足则进入执行队列中。

基于以上改造，**PipelineX 消除了轮询线程的额外开销，尤其是消除了当集群负载较高时轮询线程轮询所有 Pipeline Task 带来的性能损耗。**同时得益于 Dependency 的封装，Doris 的 PipelineX 引擎也拥有了更灵活的调度框架，使得后续实现 Spill 更容易。

## **04  Profile 改造**

对于 Operator Profile，PipelineX 引擎进行了重新整理，删除了不合理的指标并新增了必要的指标。除此以外，得益于对调度模型的改造、所有阻塞都被 Dependency 封装，我们将所有 Dependency 的就绪时间添加到 Profile 中，通过`WaitForDependency`可直观掌握每个环节的时间开销。以 Profile 中的 Scan operator 和 Exchange Source Operator 为例：

**Scan Operator:**`OLAP_SCAN_OPERATOR`的执行总时间是 457.750ms（包括 Scanner 读数据和执行时间），因 Scanner 扫描数据阻塞了 436.883ms。

```
OLAP_SCAN_OPERATOR  (id=4.  table  name  =  Z03_DI_MID):
    -  ExecTime:  457.750ms
    -  WaitForDependency[OLAP_SCAN_OPERATOR_DEPENDENCY]Time:  436.883ms
```

**Exchange Source Operator：**`EXCHANGE_OPERATOR`的执行时间为 86.691us，等待上游数据的时间为 409.256us。

```
EXCHANGE_OPERATOR  (id=3):
    -  ExecTime:  86.691us
    -  WaitForDependencyTime:  0ns
        -  WaitForData0:  409.256us
```

# **总结与展望**

---

在完成 Pipeline 执行模型的改造后，Apache Doris 在高负载情况下集群假死以及资源抢占的问题得以彻底解决、CPU 利用率得到大幅提升，而 PipelineX 执行引擎的迭代又进一步优化了执行引擎的并发执行模式与调度模式，使得 Apache Doris 执行引擎取得了显著的收益和进步，能够在真实生产环境中帮助用户进一步提升执行效率。

目前，我们正在将广泛应用于大数据场景的数据落盘技术与 PipelineX 引擎相结合，旨在进一步提升查询的性能及可靠性。未来，我们计划在 PipelineX 运行时实现更多的自动优化功能，如自适应并发和自适应计划调优，以进一步提高执行效率和性能。同时，我们也将深耕 NUMA（非一致性存储访问）本地性，以更充分利用硬件资源，提供更卓越的查询性能表现。

# **Reference**

---

> * Peter A. Boncz, Marcin Zukowski, Niels Nes.MonetDB/X100: Hyper-Pipelining Query Execution. CIDR 2005: 225-237.
> * Leis, Viktor and Boncz, Peter and Kemper, Alfons and Neumann, Thomas. Morsel-driven parallelism: A NUMA-aware query evaluation framework for the many-core age. SIGMOD 2014: 743-754.
> * DSIP-027 Pipeline Execution Engine：https://cwiki.apache.org/confluence/display/DORIS/DSIP-027%3A+Support+Pipeline+Exec+Engine
> * DSIP-035 PipelineX Execution Engine：https://cwiki.apache.org/confluence/display/DORIS/DSIP-035%3A+PipelineX+Execution+Engine
> * Pipeline 执行引擎文档：https://doris.apache.org/docs/query/pipeline/pipeline-execution-engine/
> * PipelineX 执行引擎文档：https://doris.apache.org/docs/query/pipeline/pipeline-x-execution-engine/

**- END-**

**更多标杆企业信赖**

**智慧金融与政企**：[杭银消金](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247517678&idx=1&sn=2fa963e0cf8194ad8a027f2c108d5459&chksm=cf2f8be9f85802ffb84237297a0c2ec6efdf1266f9213ee028f46e7e6aad43e7f10042015095&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[河北幸福消费金融](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522589&idx=1&sn=2c8e14756aa6727ef51da608dfb074f7&chksm=cf2f971af8581e0c2fdd887636a9844eef3b16c3c9832d9e62e1457cff641935db69832f885e&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[金融壹账通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522120&idx=1&sn=32bfd0bec1a56c7ecea05c088e566cd1&chksm=cf2f994ff858105979a27bd768133ff2c663e3f7ed5f894e373d834a1e35e67b15ca15c2de6d&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[平安人寿｜](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526035&idx=1&sn=ce723ff107a98a6d8887d45455c9e4c0&chksm=cf2f6894f858e182ab243f4e96ce9d675831802272f5e40915bf9c7edd8d36b0b022d9ec3ca1&scene=21&cur_album_id=2524165801138995201#wechat_redirect)[奇富科技](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526469&idx=1&sn=4b2d7748da1a3b601499d7d2d80748b2&chksm=cf2f6642f858ef54b7dc5b307ec4eaba7d867ee5ab5862b9e60274cc56940787686da30873fa&scene=21#wechat_redirect)｜[同程数科](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247521341&idx=1&sn=3e2b5ee81ebe6ba8b2a238e91ea7f7b7&chksm=cf2f9a3af858132c380332c123813f1df24169e30040363428db78cc0fef541f6ee802f2262d&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[无锡锡商银行](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529958&idx=1&sn=e720bfefc8cc63148c3aa6ceeee69be6&chksm=cf2f7be1f858f2f75dfab49839eafeffeddb93c49851fc8ebdcbcfd788230f6d06b1be6ad9fb&scene=21#wechat_redirect)｜[星云零售信贷](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522161&idx=1&sn=40862998ddead3c398c7765e1e8b57d8&chksm=cf2f9976f8581060f9cbf6e402f3a39d35e86a559f7d59c80b680d41977c6165e0f42cce8422&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[银联商务](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526954&idx=1&sn=4dae0891ccbaae99146b5a6ba783f392&chksm=cf2f642df858ed3b939be0b04d264ddb0529556fe15ad70c3501af44d0026972cd1bef49a7b7&scene=21#wechat_redirect)｜[招商信诺人寿](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524176&idx=1&sn=19138ac93dd44f395d68745cf94d9a6b&chksm=cf2f9157f858184176169417464dce040cd89fab6b35099d8b76086342f9682c49fd8cc87582&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[360数科](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247509531&idx=1&sn=60f3b5160acf1f2e6df2f30242dc4306&chksm=cf2fa81cf858210abd2b7158c19b39dc1431cc9114b1f463d25586dafe8c60673992deda574a&token=2001517976&lang=zh_CN&scene=21#wechat_redirect) ｜[360企业安全浏览器](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247527814&idx=1&sn=f4b56af0fac8b465b40819cd080f7563&chksm=cf2f6381f858ea97d8121642b03ae99fe48cd8c5a514c18e571be235405a90397d25f243b624&scene=21#wechat_redirect)

**互联网与文娱**：[斗鱼](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247517967&idx=1&sn=0896767aa1b7d1f0b314c22023e4de3c&chksm=cf2f8908f858001e935f7f0c1c8082847c31851dc27377652b57a8c6b3c4883890e0448eff27&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[叮咚买菜](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247518311&idx=1&sn=d18e9d2be16c26833d4d3e03f5c2836b&chksm=cf2f8660f8580f7632c6211ede6d56a4a79531c95981c72dccc76948dd9f7afdb2c419db10db&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[工商信息查询平台](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526281&idx=1&sn=a586ef0e7b8d1cc631cc08da63331dbc&chksm=cf2f698ef858e098407b107df2511260590c09f2629071fcc40f24632c92582304230cb7bf7f&scene=21#wechat_redirect)｜[货拉拉](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247496431&idx=1&sn=65b8ff94a0c2de9e60b42ebfe454a4dd&scene=21#wechat_redirect)｜[荔枝微课](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247519304&idx=1&sn=6c59fae3838e1f02a29eebca1b5799e9&chksm=cf2f824ff8580b593720ebed5234490c9b4a00fa616218b2ee3f8e89beb95eab11c720022e91&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[票务平台](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247521422&idx=1&sn=e42a6e93170f14bd17a48812f1e8ece4&chksm=cf2f9a89f858139f1ffa47678d3c4d068f44788ac18f288b1e31243bad0bfff1745b5763794c&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[奇安信](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524942&idx=1&sn=331f494035518da1875fca0fc9c5437a&chksm=cf2f6c49f858e55fd2ad0aa4a2f958b9b1b0f790441a519471debe217c38b6a5bde06ae46e87&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[腾讯音乐](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247523858&idx=1&sn=f24c266dd0e45abb74a7d2d27645aa89&chksm=cf2f9015f858190355c7375c846de6bf5e991654c2a88cc5befaef529821055d0ddaa19ee9e1&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[天眼查](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247519778&idx=1&sn=e32c89396fc87a09965cc89b5e9e1b4b&chksm=cf2f8025f858093392bffd5619ba53be539a865128bcedd6e6094cf6e49082282cd7c3bc5298&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[网易](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529891&idx=1&sn=fbdc2154807b766bd4534522dc48295d&chksm=cf2f7ba4f858f2b26996ef3fc3564da96a632a81d4ae01500a77e58ef925cbdb14d6e22efdad&scene=21#wechat_redirect)｜[网易互娱](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247491448&idx=1&sn=542e611ecd96f8d6c72765a0d8108fc1&scene=21#wechat_redirect)｜[网易严选](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247503369&idx=1&sn=29fab18c22f5778b0bb19409cb68f567&chksm=cf2fc00ef8584918fc809d661066f139d002f0202f2f6477a02286527409296d6212a373e58a&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[小米](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247513051&idx=1&sn=e22451fa38da598cf60691114fabdf69&chksm=cf2fbddcf85834ca38fee6c50087d8d588a12dae44895e0ac62d2ec5e5cd60374d24d8f86a6e&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[小鹅通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524553&idx=1&sn=63af19e32467049aa3b2fff79898c5ce&chksm=cf2f6ecef858e7d8dc3939f71ebef3bf17962e50889707b8393080a7e2ffca7520c814232bd7&scene=21&cur_album_id=2524165801138995201&poc_token=HCXDWmWjWHZ5_WrVJMWvScoFjGBJjOj_ZV5DcLPW#wechat_redirect)｜[约苗](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247521201&idx=1&sn=dc8f3d55343cd054974e8c5f4bfe50a2&chksm=cf2f9db6f85814a055a245c88c5c5c138fb7b9a9bbacedfa39dae563726ae43ad237993a5872&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[字节跳动](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247516005&idx=1&sn=84d069e950cbb178a45cd569b244d6a4&chksm=cf2fb162f8583874b83a2a591a92e75cc689f56f4ff2324ccbb583494daf76d9a96e3d6feb42&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[知乎](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247520797&idx=1&sn=1fd8394aafaddbd0ab0fbfc322e5fb25&chksm=cf2f9c1af858150c4ef3e2d9b5762477584d3e0908d71ab8871b81061253343e689c6aa18ac0&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[360商业化](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247518769&idx=1&sn=60b5ac3a78930baddc845f1e6c66a29c&chksm=cf2f8436f8580d201bacec5b3434ee62b2fea52718c4b82355c8e8c698fc2d4228a2b0cc0bef&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)

**企业服务与新经济**：[橙联](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247507345&idx=1&sn=4dc83cae32333c2aa2355c9447ecfb81&chksm=cf2fd396f8585a8018f753f50d89f6bbbb607c86350bf49957bbc07894477b88a61f3eba9342&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[度言](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247512880&idx=1&sn=9b3adbddb44b7287835adf560a56e65b&chksm=cf2fbd37f8583421316ef89513c84654956b8d9e2231868613ce4c9ad439127bb5a8a4077b73&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[观测云](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526244&idx=1&sn=af3b3bc7a5d5419d003ada6b9ea9c376&chksm=cf2f6963f858e0758e623e16612bceae95457c6d7aeed918be6d0d5bd53e8962d14549b64de0&scene=21#wechat_redirect)｜[慧策](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247515310&idx=1&sn=abfa22039b546171dd6cb1ead692de0f&chksm=cf2fb2a9f8583bbff034fc5824f89bf6a1d960a1f7997f3c3fc42af27b271c76d28aecf4b522&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[领健](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247512380&idx=1&sn=e7bb6b39a71b2a350acd7ac8d2f3b202&chksm=cf2fbf3bf858362d4e4326712e9d360493c2660e5d619e0c7ec23383b560bea2f0644b10c84f&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[领创](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247495090&idx=1&sn=314d0a12cab7985f822876ba99e29e39&chksm=cf2fe3b5f8586aa36938cd8493f98caba5fb821ca897ecf06412e6f90cb1c3deed02c4e7ed89&token=2145458351&lang=zh_CN&scene=21#wechat_redirect)｜[名创优品](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529981&idx=1&sn=85ba7efaa7c947573d59d87ae4897cc4&chksm=cf2f7bfaf858f2ec757fe383ec0bd4053bdfb09c09197fa1587c36d8c3947394d6621116341a&scene=21#wechat_redirect)｜[Moka BI](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247518538&idx=1&sn=d9a39d040c9f430a841a2045a2bafb90&chksm=cf2f874df8580e5b0d77c6235aadd8610c80b4afd1c2781172ba6e1fb734dbb6e463d6f8794e&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[美联物业](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247517792&idx=1&sn=455c085a102c7af1fcac44e62bc06ca1&chksm=cf2f8867f8580171c05af38785d7c7a32766308df46f482f7731cdc0d3dc76d9ed0a69b864f2&token=2001517976&lang=zh_CN&scene=21#wechat_redirect)｜[拈花云科](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247522076&idx=1&sn=c2b2b0f72e9bc25b9020238048a3c6ae&chksm=cf2f991bf858100dfdacf18748ef75a4e76e6cbf34e8b1e52ffc55232e6f6db0ae6c4d9b1b69&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[上海家化](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529885&idx=1&sn=b7c4ae401c29dfa64d10097a2ae3c678&chksm=cf2f7b9af858f28c309136b6a68ede29631849dceba2b84d180b07cd04d02937c940c74975ae&scene=21#wechat_redirect)｜[思必驰](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247506825&idx=1&sn=b6d002ddef220acff8963b0af04e3676&chksm=cf2fd58ef8585c9829f474fe4f75a8b6fa9e2a923b474043d0f92582318197fd737933d4cc59&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[物易云通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247490382&idx=1&sn=531f8d388526ee57aff6e86c8befe66c&scene=21#wechat_redirect)｜[云积互动](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247508739&idx=1&sn=52d028e91f9fa9cf4e93036970711eed&chksm=cf2fad04f8582412ec4cfd3c7a4a672326652c0062a481eda98a704ec245a878f5246ad236bc&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[有赞](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247524089&idx=1&sn=95b479a8648809456100e121677fb29d&chksm=cf2f90fef85819e8e73021e841d396ad1cc44e002bff0002c1b431741b3790a775415b829bf8&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[纵腾集团](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247513813&idx=1&sn=348def05d3f82fc3e7b119a78b3c2b84&chksm=cf2fb8d2f85831c45349ea5a3e4b6c27503aa41f783903a856b915a6556b3d4c81691fb24a7c&scene=21&cur_album_id=2524165801138995201#wechat_redirect)

**先进智造与电信**：[爱玛](http://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247529522&idx=1&sn=95d61e4fea493b667612a946ed126b2d&chksm=cf2f7a35f858f323918bfa574f60d3c8af817c60c6e1680f0b639533528fce3183e60ac90e95&scene=21#wechat_redirect)｜[长安汽车](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247526078&idx=1&sn=55ac5982a5a81eb98d380cdca2fe4a1f&chksm=cf2f68b9f858e1af4f53a4eeb489516d2508f513435f9c82b50896acce8cbe522f165034ce9f&scene=21&cur_album_id=2524165801138995201#wechat_redirect)｜[Lifewit](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247487975&idx=1&sn=0cfd5f9d748cb982e1ff5abc35fddb9a&chksm=cf2c1fe0f85b96f652975a5f88a9ca85d6ef75e55427dda02126d34f06d2e2a3ac15bda56413&token=2145458351&lang=zh_CN&scene=21#wechat_redirect)｜[蜀海供应链](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247496660&idx=1&sn=1c0738a95564fc4cebb8a56cb539f703&scene=21#wechat_redirect)｜[中国联通](https://mp.weixin.qq.com/s?__biz=Mzg3Njc2NDAwOA==&mid=2247523168&idx=1&sn=f6a8195d485b56438a4413ca05c9a5e7&chksm=cf2f9567f8581c71f476c52f89de7e4e6769d17e232c8930e257d2db71a45b9ab83a50382655&scene=21&cur_album_id=2524165801138995201#wechat_redirect)

作为基于 Apache Doris 的商业化公司，飞轮科技秉承着 “开源技术创新”和“实时数仓服务”双轮驱动的战略，在投入资源大力参与 Apache Doris 社区研发和推广的同时，基于 Apache Doris 内核打造了聚焦于企业大数据实时分析需求的企业级产品 SelectDB ，面向新一代需求打造世界领先的实时分析能力。自成立一年来，获得 IDG 资本、红杉中国、襄禾资本等顶级 VC 的近 10 亿元融资，创下了近年来开源基础软件领域的新纪录。

####