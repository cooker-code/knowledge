> 已吸收至：[[04_OLAP与数据库/0403_嵌入式分析/040301_DuckDB/040301_核心知识点/DuckDB本地ETL与分析场景边界|DuckDB本地ETL与分析场景边界]]
---
title: 从 75 秒到 0.24 秒：Polars/DuckDB 的魅力
author: Python是纸老虎
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0ODYyODEyNw==&mid=2247483814&idx=1&sn=efdd6935e36a0c80b8d40f3ebb7f9ad0&chksm=c26446e37bffa0fb675abfa61a12f5e85a55ff9b278c35b2eeca5fb0d8f62cc06004ade15bb2&mpshare=1&scene=24&srcid=1110hJ5rWJsYpUDk0dhT4Uwh&sharer_shareinfo=2984134a3e536f051d25f8dcc26a47cc&sharer_shareinfo_first=2984134a3e536f051d25f8dcc26a47cc#rd
---

> “
>
> 面对一个 2GB、460 万行的嵌套数据筛选任务，传统的 Tidyverse 和 Pandas 耗时数十秒，而现代工具 Polars 和 DuckDB 仅需0.3秒。本文尝试对 Python 和 R 两门语言的六大工具进行了基准测试，直观展示了“全量加载”与“惰性计算”之间惊人的 300 倍性能鸿沟，并分享了为什么 Polars 和 DuckDB 是现代数据分析的利器。
>
> 正文字数：约 2200 字

## 背景

最近干活时碰到一个逻辑相当简单的小任务：有一个约 2GB 的 Parquet 数据文件，其中包含了大概 460 万行、多个常规列和嵌套列，我需要从一个名为 `concepts` 的字符串列表列中，筛选出包含特定概念（例如 `"人工智能"`）的行，然后按 `year` 列进行分组计数，从而得到此概念每年的出现频率。

我选择我最喜欢也最熟悉的 Python Polars 库来完成这个任务。关于什么是 Polars，可以参考我之前这一篇公众号文章。得益于 Polars 符合人类自然思维和语言直觉的表达式，我很快便写出了如下代码：

```
import polars as pl

(
    pl.scan_parquet(FILE_PATH)
    .filter(pl.col("concepts").list.contains(TARGET_CONCEPT))
    .select(pl.col("year").value_counts())
    .unnest("year")
    .sort("year")
    .collect()
)
```

Code 1: Polars in Python

非常流畅！`scan_parquet`惰性扫描数据，`filter+list.contains`表达式筛选行，`select+value_counts`进行频数统计，`sort`进行排序，一切都那么自然。

由于 Polars 的性能优势（包括但不限于 Rust 编写、惰性查询优化、高度并行等），这几行代码几乎瞬间便完成了这个查询和统计任务。这种高性能是我个人喜爱 Polars 的重要原因，但它到底有多快？

我此前只看过一些其他作者进行的 Benchmark 测试，却没有亲自比较过自己熟悉的几种工具的速度差异，特别是 Python和 R 之间的比较。于是，我便拿手头的数据和这个非常有代表性的数据处理任务为测试对象，亲手编写了一个 Benchmark 测试。

## 测试内容

我选择了这几种 Python 和 R 中极具代表性的数据处理工具作为测试主角：

* **Python 🐍:**

+ **Pandas: 最主流的 Python 数据科学生态。**
+ **Polars: Rust 编写的新时代高性能数据框库，以其并行计算和惰性优化扬名。**
+ **DuckDB: C++ 编写的高性能分析型数据库，可以在各种主流语言中调用。**

* **R 语言:**

+ **Tidyverse: R 语言数据处理全家桶，以其优雅和链式操作的语法深入人心。**
+ **Tidypolars: 一个非常有趣的项目，允许用 Tidyverse 的语法来驱动 Polars 的高性能引擎。**
+ **DuckDB: DuckDB 在 R 中的接口。**

其中，Pandas 和 Tidyverse 分别是 Python 和 R 进行数据处理的最经典、最主流工具；Polars（和 Tidypolars）则是新时代高性能数据框处理库的代表，融入了大量现代特性；DuckDB 则是基于 SQL 进行数据分析的代表产品。

我在我的笔记本电脑上进行测试（i9-12900H + 32GB内存，Windows 11 + Python 3.13.8 和 R 4.5.1，所有库均为最新版本）。每个工具重复运行同样的任务流（读取数据+数据筛选+聚合统计）各 3 次，取其平均时间。为确保可比性，每个工具的数据算法流程完全一致，且进行一次预运行以确保不会受到缓存干扰。

## 对比结果

结果如下表所示（按速度从快到慢排序，内存占用不一定十分准确）：

| 排名 | 工具 | 语言 | 平均用时 | 内存占用 |
| --- | --- | --- | --- | --- |
| 🏆1 | **Polars** | Python | **0.24 s** | 1228.29 MB |
| 🥈2 | **DuckDB** | R | **0.29 s** | **63.82 MB** |
| 🥉3 | **DuckDB** | Python | **0.35 s** | **67.41 MB** |
| 4 | Tidypolars | R | **1.87 s** | 2662.15 MB |
| 5 | Pandas | Python | 20.45 s | 15320.66 MB😰 |
| 6 | Tidyverse | R | 75.45 s 😰 | 4398.17 MB |

表1: 不同工具运行时间和内存占用情况

图1: 不同工具运行时间和内存占用情况（对数尺度）

**速度最快的是 Python Polars，居然比最慢的 Tidyverse 快了 300 多倍！**DuckDB 表现同样惊艳，它在 R 和 Python 中的表现非常接近（甚至在 R 中稳定地更快），也证明了其底层核心引擎的强大和一致性。配合 Parquet 文件的列式存储特性，无论是 Lazy Polars 还是 DuckDB 都无需将源数据全部加载到内存中，特别是 DuckDB 几乎完全不将数据读入内存计算，在速度极快的同时实现了极低的内存占用。

```
SELECT
    year,
    COUNT(*) AS count
FROM read_parquet('{FILE_PATH}')
WHERE list_contains(concepts_LLM, '{TARGET_CONCEPT}')
GROUP BY year
ORDER BY year;
```

Code 2: DuckDB Expr

Tidypolars 虽然比原生 Polars 慢了近 10 倍，但依然比 Pandas 和 Tidyverse 快一个数量级。它将 Tidyverse 的语法和 Polars 的性能结合起来，尽管存在一些额外的性能和内存开销，但能在几乎不改变传统代码风格情况下大幅提高性能，在必要时也能方便地转换回 `data.frame/tibble`，在我看来是值得尝试的。

```
library(tidypolars)
library(tidyverse)

scan_parquet_polars(file_path) |>
  select(all_of(columns_to_read)) |>
  filter(target_concept %in% concepts_LLM) |>
  summarise(count = n(), .by = "year") |>
  arrange(-year) |>
  collect()
```

Code 3: Tidypolars in R

Pandas 比 Tidyverse 快得多（我推测是因为 Pandas 默认启用了 arrow 作为数据后端），但和几个新工具相比仍然慢得难以忍受，且 Pandas 的代码是我认为最丑陋的！而且在启动 Pandas 任务后，**内存占用从 10.5 GB 暴涨到 25 GB 左右**，要知道这只是一个逻辑简单的小任务，太恐怖了！

```
import pandas as pd

df = pd.read_parquet(FILE_PATH)
(
    df[df["concepts_LLM"].apply(lambda values: TARGET_CONCEPT in values)]
    .year.value_counts()
    .sort_index(ascending=False)
)
```

```
Code 4: Tidypolars in R
```

```
library(tidyverse)

arrow::read_parquet(
  file_path,
  col_select = columns_to_read,
  as_data_frame =TRUE
)%>%
  filter(purrr::map_lgl(concepts_LLM,~ target_concept %in% .x))%>%
  summarise(count = n(), .by ="year")%>%
  arrange(-year)
```

```
Code 5: Tidyverse in R
```

## 总结

同样的任务、同样的算法（纸面上），最快的工具和最慢的工具之间，性能差距达到了惊人的 300 倍 (75.45 vs. 0.24)！

在我看来，Polars 是一个真正将惰性计算、自动优化、并行计算等现代计算特性，与 R Tidyverse 的**优雅管道式数据处理**流合二为一的库，在提供极具表现力的语法同时提供了极强的性能。可以说 Polars 对于让我逐渐转向使用 Python 来完成项目的几乎每个环节（除了最后的统计和可视化部分还是会用 R）起了至关重要的作用。

不过工具并无优劣，只有合适不合适之分。Polars 和 DuckDB 并非要取代 Pandas 或 Tidyverse，而是为性能敏感的大数据量场景提供了更优的解决方案，它们的设计理念之一就是为了榨干现代硬件的性能。但我们也不得不面对一个现实，即**数据场景正在面临不断庞大的数据量、**更加**抽象的数据结构、日益复杂的任务流**，此时新兴工具 Polars 和 DuckDB 的优势也必然会更加明显。

此外，知道为什么慢（比如 `.apply` 和 `map` 的循环机制）和为什么快（比如向量化执行、惰性计算和并行处理），能帮助我们写出更高效的代码。事实上，我在编写 Pandas 和 Tidyverse 代码时已经有意识地进行过优化了，但很遗憾，我发现 Pandas 无法绕开 `.apply`，Tidyverse 同样需要依赖自定义匿名函数来逐行处理嵌套列表，对于复杂的数据结构，这两个工具缺乏原生且高效的支持。

总而言之，人生苦短，我用 Polars 和 DuckDB。数据驱动和人工智能的时代，多学一些、多用一些现代的工具总是好的！

---

> 冬生夏阳 秋池结霜
>
> 轮回往复 依循时光
>
> 岁岁年年 叠于我身
>
> 皆循时光 逝而不返