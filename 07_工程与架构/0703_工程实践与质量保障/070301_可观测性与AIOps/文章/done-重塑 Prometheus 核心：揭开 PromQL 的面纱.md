> 已吸收至：[[07_工程与架构/0703_工程实践与质量保障/070301_可观测性与AIOps/070301_核心知识点/结构化日志与LLM_RCA证据边界|结构化日志与LLM_RCA证据边界]]
---
title: 重塑 Prometheus 核心：揭开 PromQL 的面纱
author: 云原生运维圈
date:
url: http://mp.weixin.qq.com/s?__biz=MzUxNTg5NTQ0NA==&mid=2247489479&idx=1&sn=6b9877207d988d1d046b899a6cde7f70&chksm=f828595faad26bbbbdbd6065fbece582d113ea3b8b061c7ff33835763945e0ebac1edf0e8688&mpshare=1&scene=24&srcid=0108PUxcIAXb5x8Kchq1kemT&sharer_shareinfo=8ff2e8fdefe383faa55a5423fa160cd6&sharer_shareinfo_first=f2f9dfeb7e64f2d41c20afeee14ae307#rd
---

Bale of Straw, South Germany

# 引言

在我们重塑 Prometheus 之前，我们还有一个重要的角色需要熟悉和理解，那就是我们的 `PromQL`。

提前声明下，本篇文章的最大敌人就是`枯燥`，比较`无聊`。

亲爱的朋友，你是否武装好了呢，反正我是没有，哈哈，开玩笑。**反正我自己看着文章都头疼，就如同健身中的练腿，特别是深蹲**。但是为了不让你们小看我，我咬咬牙，就过去了，反正学习之前，需要准备一下，不然受不了。

如果你已经知道了或者很熟悉了，那这篇文章你可以选择在学习下，那它是什么呢，我们来介绍下。

# 介绍

`PromQL（Prometheus Query Language）`是一种功能强大的查询语言，专为 Prometheus 设计。它允许用户以灵活的方式从时间序列数据库中选择、提取、聚合和分析数据，支持多种聚合和计算操作。PromQL的设计目标是提供一种简单且灵活的方式来查询、分析监控数据，使用户能够快速获取所需的信息。Prometheus提供两种查询：**瞬时查询（instant query，查询某个时间点的数据）、范围查询（ range query，在开始和结束时间之间以均匀间隔进行数据查询），可以将范围查询看做在不同时间点上多次进行瞬时查询。**

官方文档[1]

# 开始

## 四种指标类型

* • Counter（计数器）
* • Gauge （仪表类型）
* • Histogram（直方图类型）
* • Summary （摘要类型）

### Counter（计数器）

Counter **(只增不减的计数器)** 类型的指标其工作方式和计数器一样，只增不减。常见的监控指标，如 `http_requests_total`、 `node_cpu_seconds_total` 都是 Counter 类型的监控指标。

在 node-exporter 返回的样本数据中，其注释中也包含了该样本的类型。例如：

```
HELP node_cpu_seconds_total Seconds the cpus spent in each mode.TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="cpu0",mode="idle"} 362812.7890625
```

* • **#HELP：** 解释当前指标的含义，上面表示在每种模式下 node 节点的 cpu 花费的时间，以 `s` 为单位。
* • **#TYPE：** 说明当前指标的数据类型，上面是 counter 类型。

counter 是一个简单但又强大的工具，例如我们可以在应用程序中记录某些事件发生的次数，通过以时间序列的形式存储这些数据，我们可以轻松的了解该事件产生的速率变化。PromQL 内置的聚合操作和函数可以让用户对这些数据进行进一步的分析，例如，通过 rate() 函数获取 HTTP 请求量的增长率：

```
rate(http_requests_total[5m])
```

查询当前系统中，访问量前 10 的 HTTP 请求：

```
topk(10, http_requests_total)
```

### Gauge （仪表类型）

与 Counter 不同， Gauge **(可增可减的仪表盘)** 类型的指标侧重于反应系统的当前状态。因此这类指标的样本数据可增可减。常见指标如：`node_memory_MemFree_bytes`（主机当前空闲的内存大小）、 `node_memory_MemAvailable_bytes`（可用内存大小）都是 Gauge 类型的监控指标。通过 Gauge 指标，用户可以直接查看系统的当前状态：

```
node_memory_MemFree_bytes
```

对于 Gauge 类型的监控指标，通过 PromQL 内置函数 delta() 可以获取样本在一段时间范围内的变化情况。例如，计算 CPU 温度在两个小时内的差异：

```
delta(cpu_temp_celsius{host="zeus"}[2h])
```

还可以直接使用 `predict_linear()` 对数据的变化趋势进行预测。例如，预测系统磁盘空间在4个小时之后的剩余情况：

```
predict_linear(node_filesystem_free_bytes[1h], 4 * 3600)
```

### Histogram（直方图类型） 和 Summary（摘要类型）

除了 Counter 和 Gauge 类型的监控指标以外，Prometheus 还定义了 Histogram 和 Summary 的指标类型。Histogram 和 Summary 用于统计和分析样本的分布情况。

* • 在大多数情况下人们都倾向于使用某些量化指标的`平均值`，例如 CPU 的平均使用率、页面的平均响应时间，这种方式也有很明显的问题，以系统 API 调用的平均响应时间为例：如果大多数 API 请求都维持在 100ms 的响应时间范围内，而个别请求的响应时间需要 5s，那么就会导致某些 WEB 页面的响应时间落到中位数上，而这种现象被称为`长尾问题`。
* • 为了区分是平均的慢还是长尾的慢，最简单的方式就是按照请求延迟的范围进行`分组`。例如，统计延迟在 `010ms` 之间的请求数有多少，而 `1020ms` 之间的请求数又有多少。通过这种方式可以快速分析系统慢的原因。Histogram 和 Summary 都是为了能够解决这样的问题存在的，通过 Histogram 和 Summary 类型的监控指标，我们可以快速了解监控样本的分布情况。

例如，指标 `prometheus_tsdb_wal_fsync_duration_seconds` 的指标类型为 Summary。它记录了 Prometheus Server 中 `wal_fsync` 的处理时间，通过访问 Prometheus Server 的 `/metrics` 地址，可以获取到以下监控样本数据：

```
HELP prometheus_tsdb_wal_fsync_duration_seconds Duration of WAL fsync.TYPE prometheus_tsdb_wal_fsync_duration_seconds summary
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.5"} 0.012352463
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.9"} 0.014458005
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.99"} 0.017316173
prometheus_tsdb_wal_fsync_duration_seconds_sum 2.888716127000002
prometheus_tsdb_wal_fsync_duration_seconds_count 216
```

从上面的样本中可以得知当前 Prometheus Server 进行 wal\_fsync 操作的总次数为 216 次，耗时 2.888716127000002s。其中中位数（quantile=0.5） 的耗时为 0.012352463，9 分位数（quantile=0.9）的耗时为 0.014458005s。

在 Prometheus Server 自身返回的样本数据中，我们还能找到类型为 Histogram 的监控指标 `prometheus_tsdb_compaction_chunk_range_seconds_bucket`：

```
HELP prometheus_tsdb_compaction_chunk_range_seconds Final time range of chunks on their first compactionTYPE prometheus_tsdb_compaction_chunk_range_seconds histogram
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="100"} 71
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="400"} 71
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="1600"} 71
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="6400"} 71
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="25600"} 405
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="102400"} 25690
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="409600"} 71863
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="1.6384e+06"} 115928
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="6.5536e+06"} 2.5687892e+07
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="2.62144e+07"} 2.5687896e+07
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="+Inf"} 2.5687896e+07
prometheus_tsdb_compaction_chunk_range_seconds_sum 4.7728699529576e+13
prometheus_tsdb_compaction_chunk_range_seconds_count 2.5687896e+07
```

与 Summary 类型的指标相似之处在于 Histogram 类型的样本同样会反映当前指标的记录的总数(以 \_count 作为后缀)以及其值的`总量`（以 `_sum` 作为后缀）。不同在于 Histogram 指标直接反映了在不同区间内样本的`个数`，区间通过标签 `le` 进行定义。

## 偏移（offset）修饰符

偏移修饰符用于在`查询中更改瞬时和范围查询的时间偏移`。

例如，查询 `http_requests_total` 在当前时间戳前 5 分钟的值：

```
http_requests_total offset 5m
```

注意，偏移修饰符需要紧跟选择器之后，下面是正确的格式：

```
sum(http_requests_total{method="GET"} offset 5m) // 正确
```

下面这个是错误的格式：

```
sum(http_requests_total{method="GET"}) offset 5m // 无效
```

offset 同样适用于`范围查询`。`http_requests_total` 在一周前的 5 分钟速率：

```
rate(http_requests_total[5m] offset 1w)
```

## @ 修饰符

@ 修饰符用于在查询中`更改瞬时和范围查询的当前时间戳`。提供给 @ 修饰符的时间是一个 `Unix` 时间戳，并用`浮点数值`描述。

例如，http\_requests\_total 在 2021-01-04T07:40:00+00:00 的值：

```
http_requests_total @ 1609746000
```

注意，@ 修饰符需要紧跟选择器之后，下面是正确的格式：

```
sum(http_requests_total{method="GET"} @ 1609746000) // 正确
```

下面这个是错误的格式：

```
sum(http_requests_total{method="GET"}) @ 1609746000 // 无效
```

@ 修饰符同样适用范围查询。http\_requests\_total 在 2021-01-04T07:40:00+00:00 的 5 分钟速率：

```
rate(http_requests_total[5m] @ 1609746000)
```

> @ 修饰符可以与偏移修饰符一起工作，这时偏移就应用于 @ 修饰符指定时间。这两个修饰符之间的顺序不会对结果产生影响。

例如，如下两个查询将产生相同的结果：

**在 @ 之后偏移**

```
http_requests_total @ 1609746000 offset 5m
```

**在 @ 之前偏移**

```
http_requests_total offset 5m @ 1609746000
```

此外，start() 和 end() 函数可以作为 @ 修饰符的值来使用。

对于范围查询来说，它们分别解析为范围查询的开始时间和结束时间。

对于瞬时查询，start() 和 end() 都解析为当前查询时间。

```
http_requests_total @ start()rate(http_requests_total[5m] @ end())
```

## 表达式四种数据类型

PromQL 查询语句即表达式，实现的四种数据类型：

* • **瞬时向量（Instant vector）**：`一组时间序列`，每个时间序列包含单个样本，它们共享相同的时间戳。也就是说，表达式的返回值中只会包含该时间序列中最新的一个样本值。
* • **区间向量（Range vector）**：`一组时间序列`，每个时间序列包含一段时间范围内的样本数据。
* • **标量（Scalar）**：`一个浮点型的数据值`，没有时序。可以写成[-]（digits）[.（digits）]的形式。需要注意的是，使用表达式 count（http\_requests\_total）返回的数据类型依然是瞬时向量，用户可以通过内置函数 scalar() 将单个瞬时向量转换为标量。
* • **字符串（String）**：`一个简单的字符串值`。字符串可以用单引号（''）、双引号（""）或反引号（``）来指定。

### 瞬时向量（Instant vector）

`Instance vector（瞬时向量）`表示一个时间序列的集合，但是每个时序只有最近的一个点，而不是线。

### 区间向量（Range vector）

`Range vector（范围向量）`表示一段时间范围里的时序，每个时序可包含多个点 。

### 标量（Scalar）

`Scalar（标量）`通常为数值，可以将只有一个时序的 Instance vector 转换成 Scalar。

### 字符串（String）

一个简单的字符串值。字符串可以用单引号（''）、双引号（""）或反引号（``）来指定。

## 时间序列（向量）

按照时间顺序记录系统、设备状态变化的数据，每个数据成为一个`样本`。

* • 数据采集以特定的时间周期进行，因而，随着时间流逝，将这些样本数据记录下来，将生成一个离散的样本数据序列。
* • 该序列也称为`向量（Vector）`, 以时间轴为横坐标、序列为纵坐标，这些数据点连接起来就会形成一个矩阵。

### 时间序列的构成

每条时间序列（Time Series）是通过 `指标名称（Metrics name）和一组标签集（Label set）` 来命名的。

如果 `time` 相同，但是指标名称或者标签集不同，那么时间序列也不同。

### 样本构成

矩阵中每一个点都可称为一个`样本（Sample）`，样本主要由 3 构成。

* • **指标（Metrics）**：包括指标名称（Metrics name）和一组标签集（Label set）名称，如 `request_total{path="/status"，method="GET"`} 。
* • **时间戳（TimeStamp）**：这个值默认精确到`毫秒`。
* • **样本值（Value）**：这个值默认使用 `Float64` 浮点类型。

时间序列的指标（Metrics）存储格式为 key-value。

`http_request_total{status="200"，method="GET"}@1434417560938=>94355` 为例：

> 在 Key-Value 关系中，94355 作为 Value（也就是样本值Sample Value），前面的 `http_request_total{status="200"，method="GET"} @1434417560938` 一律作为 Key。

### key 的组成

* • **Metric Name**：指标名（例子中的 `http_request_total` ）
* • **Label**：标签（例子中的`{status="200"，method="GET"}`）
* • **Timestamp**：时间戳（例子中的 `@1434417560938`）

Prometheus Metrics两种表现形式：

## 标签过滤器4种运算符

* • = `与字符串匹配`
* • != `与字符串不匹`
* • =~ `与正则匹配`
* • !~ `与正则不匹配`

### 匹配器（Matcher）

匹配器是作用于标签上的，标签匹配器可以对时间序列进行过滤，Prometheus 支持完全匹配和正则匹配两种模式：`完全匹配和正则表达式匹配`。

#### 完全匹配

**相等匹配器（=）**

`相等匹配器（Equality Matcher）`，用于选择与提供的字符串完全相同的标签。下面介绍的例子中就会使用相等匹配器按照条件进行一系列过滤。

```
node_cpu_seconds_total{instance="xaw"}
```

**不相等匹配器（!=）**

`不相等匹配器（Negative Equality Matcher）`，用于选择与提供的字符串不相同的标签。它和相等匹配器是完全相反的。举个例子，如果想要查看job并不是 HelloWorld 的 HTTP 请求总数，可以使用如下不相等匹配器。

#### 正则表达式匹配

**正则表达式匹配器（=~）**

`正则表达式匹配器（Regular Expression Matcher）`，用于选择与提供的字符串进行正则运算后所得结果相匹配的标签。Prometheus 的正则运算是强指定的，比如正则表达式 a 只会匹配到字符串 a ，而并不会匹配到 ab 或者 ba 或者abc。如果你不想使用这样的强指定功能，可以在正则表达式的前面或者后面加上".\*"。比如下面的例子表示 job 是所有以 Hello 开头的 HTTP 请求总数。

```
node_cpu_seconds_total{instance=~"xaw-.*", mode="idle"}
```

`node_cpu_seconds_total` 直接等效于 **{**name**="node\_cpu\_seconds\_total"} ，后者也可以使用和前者一样的4种匹配器（=，!=，=，!**）。比如下面的案例可以表示所有以 Hello 开头的指标。

```
{__name__="node_cpu_seconds_total",instance=~"xaw-.*", mode="idle"}
```

**正则表达式相反匹配器（!~）**

`正则表达式相反匹配器（Negative Regular Expression Matcher）`，用于选择与提供的字符串进行正则运算后所得结果不匹配的标签。因为 PromQL 的正则表达式基于 `RE2` 的语法，但是 RE2 不支持向前不匹配表达式，所以 **!~** 的出现是作为一种替代方案，以实现基于正则表达式排除指定标签值的功能。在一个选择器当中，可以针对同一个标签来使用多个匹配器。比如下面的例子，可以实现查找 job 名是 node 且安装在 `/prometheus` 目录下，但是并不在 `/prometheus/user` 目录下的所有文件系统并确定其大小。

```
node_filesystem_size_bytes{job="node",mountpoint=~"/prometheus/.*", mountpoint!~ "/prometheus/user/.*"}
```

## 范围选择器

我们可以通过将 时间范围选择器[2] 附加到查询语句中，指定为每个返回的区间向量样本值中提取多长的时间范围。每个时间戳的值都是按时间倒序记录在时间序列中的，该值是从时间范围内的时间戳获取的对应的值。

时间范围通过数字来表示，单位可以使用以下其中之一的时间单位：

* • s - `秒`
* • m - `分钟`
* • h - `小时`
* • d - `天`
* • w - `周`
* • y - `年`

因为现在每一个时间序列中都有多个时间戳多个值，所以没办法渲染，必须是标量或者瞬时向量才可以绘制图形。

不过通常区间向量都会应用一个函数后变成可以绘制的瞬时向量，Prometheus 中对瞬时向量和区间向量有很多操作的 函数[3]，不过对于区间向量来说最常用的函数并不多，使用最频繁的有如下几个函数：

* • **rate()** : 计算整个时间范围内区间向量中时间序列的`每秒平均增长率`。
* • **irate()** : 仅使用时间范围中的`最后两个数据点`来计算区间向量中时间序列的每秒平均增长率， irate 只能用于绘制`快速变化的序列`，在长期趋势分析或者告警中更推荐使用 rate 函数。
* • **increase()** : 计算`所选时间范围内时间序列的增量`，它基本上是速率乘以`时间范围选择器中的秒数`。

## PromQL 运算符

### 数学运算符

数学运算符比较简单，就是简单的加减乘除等。

例如：我们通过 `prometheus_http_response_size_bytes_sum` 可以查询到 Prometheus 这个应用的 HTTP 响应字节总和。但是这个单位是`字节`，我们希望用 MB 显示。那么我们可以这么设置：`prometheus_http_response_size_bytes_sum/8/1024`。

PromQL支持的所有数学运算符如下所示：

* • + `(加法)`
* • - `(减法)`
* • \* `(乘法)`
* • / `(除法)`
* • % `(求余)`
* • ^ `(幂运算)`

### 布尔运算符

布尔运算符支持用户根据时间序列中样本的值，对时间序列进行过滤。

例如：我们可以通过 `prometheus_http_requests_total` 查询出每个接口的请求次数，但是如果我们想筛选出请求次数超过 20 次的接口呢？

此时我们可以用下面的 PromQL 表达式：

```
prometheus_http_requests_total > 20
```

> 从上面的图中我们可以看到，value 的值还是具体的数值。但如果我们希望对符合条件的数据，value 变为 1。不符合条件的数据，value 变为 0。那么我们可以使用 `bool` 修饰符。

我们使用下面的 PromQL 表达式：

```
prometheus_http_requests_total > bool 20
```

目前，Prometheus支持以下布尔运算符如下：

* • == `（相等）`
* • !=`（不相等）`
* • > `（大于）`
* • < `（小于）`
* • =`（大于或等于）`
* • <=`（小于或等于）`

### 集合运算符

通过集合运算，可以在两个瞬时向量与瞬时向量之间进行相应的集合操作。这个和我们理解的可不一样，仔细看下面的解释。目前，Prometheus支持以下集合运算符：

* • and `与操作`
* • or `或操作`
* • unless `排除操作`

**and 与操作**

结果：返回两个向量中共有的标签（匹配的时间序列）。

示例：

```
vector1 = {A, B, C}
vector2 = {B, C, D}
   - 结果：{B, C}
```

**or 或操作**

结果：返回两个向量的联合，包括所有标签。

示例：

```
vector1 = {A, B, C}
vector2 = {B, C, D}
   - 结果：{A, B, C, D}
```

**unless 排除操作**

结果：从第一个向量中排除与第二个向量中匹配的时间序列。

示例：

```
vector1 = {A, B, C}
vector2 = {B, C, D}
   - 结果：{A}
```

### 操作符优先级

在PromQL操作符中优先级由高到低依次为：

* • ^
* • \*, /, %
* • +, -
* • ==, !=, <=, <, >=, >
* • and, unless
* • or

## 内置函数

Prometheus 内置不少函数，通过灵活的应用这些函数，可以更方便的查询及数据格式化。本文将选取其中较常使用到的几个函数进行讲解。

### ceil 函数

ceil 函数会将返回结果的值`向上取整数`。

```
ceil(avg(promhttp_metric_handler_requests_total{code="200"}))
```

### floor 函数

floor 函数与 ceil 相反，将会进行`向下取整`的操作。

### rate 函数

rate函数是使用频率最高，也是最重要的函数之一。rate 用于取某个时间区间内`每秒的平均增量数`，它会以该时间区间内的所有数据点进行统计。rate 函数通常作用于 `Counter` 类型的指标，用于了解增量情况。

示例：获取 `http_request_total` 在1分钟内，平均每秒新增的请求数

```
rate(promhttp_metric_handler_requests_total{handler="/rules"}[1m])
```

### irate函数

相比 rate 函数，irate 提供了更高的`灵敏度`。irate 函数是通过时间区间中`最后两个样本数据来计算区间向量的增长速率`，从而`避免范围内的平均值拉低峰值的情况` 。

示例：该函数用法与rate相同

```
irate(promhttp_metric_handler_requests_total{handler="/rules"}[1m])
```

### 其它内置函数

除了上面提到的这些函数外，PromQL 还提供了大量的其他函数供使用，功能范围涵盖了日常所需的功能，如用于标签替换的 `label_replace` 函数、统计 Histogram 指标分位数的 `histogram_quantile` 函数

更多信息可参阅官方文档[4]

## PromQL查询示例

### 基本查询

查询所有实例的 `http_requests_total` 指标：

```
http_requests_total
```

### 聚合查询

计算所有实例的 `http_requests_total` 总和：

```
sum(http_requests_total)
```

### 时间函数查询

计算过去10分钟内每秒的请求速率：

```
rate(http_requests_total[10m])
```

### 复杂查询

计算每个实例CPU使用率，并按实例（instance）进行分组：

```
sum(rate(cpu_usage_seconds_total[5m])) by (instance)
```

## 常见应用场景

### 监控CPU、内存使用情况

使用PromQL可以轻松监控CPU、内存的使用情况。例如，查询每个实例过去5分钟的CPU使用率：

```
sum(rate(cpu_usage_seconds_total[5m])) by (instance)
```

### 监控网络流量

PromQL还可以用于监控网络流量。例如，查询每个实例过去5分钟的网络接收、发送字节数：

```
sum(rate(network_receive_bytes_total[5m])) by (instance)
sum(rate(network_transmit_bytes_total[5m])) by (instance)
```

### 监控应用程序性能

通过PromQL可以监控应用程序的性能指标，如请求延迟、错误率等。例如，查询每个实例过去5分钟的请求延迟的95分位值：

```
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (instance))
```

### 监控自定义指标

PromQL允许监控自定义指标。例如，监控特定业务逻辑的指标，如过去1小时用户注册数增量：

```
sum(increase(user_registration_total[1h]))
```

## PromQL 聚合操作

Prometheus 还提供了聚合操作符，这些操作符作用于`瞬时向量`。可以将瞬时表达式返回的样本数据进行`聚合`，形成一个新的`时间序列`。目前支持的聚合函数有：

* • sum `求和`
* • min `最小值`
* • max `最大值`
* • avg `平均值`
* • stddev `标准差`
* • stdvar `标准方差`
* • count `计数`
* • count\_values `对value进行计数`
* • bottomk `后n条时序`
* • topk `前n条时序`

### sum 求和

用于对记录的 value 值进行`求和`。

例如：sum(prometheus\_http\_requests\_total) 表示统计所有 HTTP 请求的次数。

```
sum(prometheus_http_requests_total)
```

### min 最小值

返回所有记录的`最小值`。

例如：min(prometheus\_http\_requests\_total) 表示获取数据集合中的最小值。

```
min(prometheus_http_requests_total)
```

### max 最大值

返回所有记录的`最大值`。

例如：maxmetheus\_http\_requests\_total) 表示获取数据集合中的最大值。

```
max(prometheus_http_requests_total)
```

### avg 平均值

avg 函数返回所有记录的`平均值`。

例如：avg(metheus\_http\_requests\_total) 表示获取数据集合中的平均值。

```
avg(prometheus_http_requests_total)
```

### stddev 标准差

标准差（Standard Deviation）常用来描述数据的`波动大小`。

例如： 统计出不同 HTTP 请求的数量波动情况。

```
stddev(prometheus_http_requests_total)
```

### count 计数

count 函数返回所有记录的`计数`。

例如：count(prometheus\_http\_requests\_total) 表示统计所有 HTTP 请求的次数。

```
count(prometheus_http_requests_total)
```

### bottomk 后几条

bottomk 用于对样本值进行`排序`，返回当前样本值后 N 位的`时间序列`。

例如：获取 HTTP 请求量后 5 位的请求，可以使用表达式：

```
bottomk(5, prometheus_http_requests_total)
```

### topk 前几条

topk 用于对样本值进行`排序`，返回当前样本值前 N 位的`时间序列`。

例如：获取 HTTP 请求量前 5 位的请求，可以使用表达式：

```
topk(5, prometheus_http_requests_total)
```

### PromQL 语法总结

由于所有的 PromQL 表达式必须至少包含一个`指标名称`，或者至少有一个不会匹配到空字符串的标签过滤器，因此结合 Prometheus 官方文档，可以梳理出如下非法示例。

```
{job=~".*"} 非法！ .*表示任意一个字符,这就包括空字符串，且还没有指标名称
{job=""}    非法！
{job!=""}   非法！

相反，如下表达式是合法的。
{job=~".+"}               合法！.+表示至少一个字符
{job=~".*",method="get"}  合法！.*表示任意一个字符
{job="",method="post"}    合法！存在一个非空匹配
{job=~".+",method="post"} 合法！存在一个非空匹配
```

## 性能优化

在使用PromQL时，性能是一个重要的考虑因素。下面是一些常用性能优化技巧：

* • **合适的时间查询范围：** 查询时选择合适的时间范围，以避免不必要的数据查询处理。
* • **避免过于复杂的查询：** 尽量简化查询，避免使用过多的聚合计算和运算符操作。
* • **指标数据缓存：** 对于频繁查询的指标，可以考虑使用缓存机制。

如果需要查询处理大量数据，页面绘图可能会超时或使服务器、浏览器过载。因此，在构建未知规模的数据查询时，先从 Prometheus 的表格视图开始构建，直到结果看起来合理（最多数百个时间序列，而不是数千个时间序列）。只有在充分过滤或聚合后，才能切换到图形视图。如果仍然需要太长时间才能绘制图形，**建议使用记录规则进行预先处理**。此外，聚合多个时间序列即使输出只有少量时间序列结果，也会对服务器产生严重负载，这类似于在关系数据库中对一列的所有值求和，即使输出值只有一个数字，也会很慢。

# 结语

终于结束了，如果你坚持看完了，那么，恭喜你，你已经超越了很多人了。但是，我看的已经快不行了，需要紧急治疗，哈哈，开玩笑。

经过上面的学习，我们已经把我们必须的已经熟悉了，这为我们后面的部分，打下了坚实的基础。

#### 引用链接

`[1]` 官方文档: *https://prometheus.io/docs/prometheus/latest/querying/basics*
`[2]` 时间范围选择器: *https://prometheus.io/docs/prometheus/latest/querying/basics/*
`[3]` 函数: *https://prometheus.io/docs/prometheus/latest/querying/functions/*
`[4]` 官方文档: *https://prometheus.io/docs/prometheus/latest/querying/functions/*