---
title: StarRocks 3.4 发布--AI 场景新支点，Lakehouse 能力再升级
author: StarRocks
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247497002&idx=1&sn=66f0ec481a22d1e39e7317df01497d31&chksm=e82e1d2db9c0d22698bd2fe8a575b02cf32c7a9e477eefd553a2ed27d1d408322ad85150af8f&mpshare=1&scene=24&srcid=0121NoTqH4SNtzORHzd2zomJ&sharer_shareinfo=535c22c54d83dd5660a38b65d3f6ca62&sharer_shareinfo_first=535c22c54d83dd5660a38b65d3f6ca62#rd
---

自 StarRocks 3.0 起，社区明确了以 Lakehouse 为核心的发展方向。Lakehouse 的价值在于融合数据湖与数据仓库的优势，能有效应对大数据量增长带来的存储成本压力，做到 single source of truth 的同时继续拥有极速的查询性能，同时也为 AI 时代的多样化数据需求提供可扩展的高效访问能力。

构建 Lakehouse 后，你将拥有开放统一的数据存储与基于一份数据，支持多样化的 workload，服务企业 AI、BI 的数据应用，进而实现**“One Data, All Analytics”** 的业务价值。

在之前的版本中，StarRocks 已对 Lakehouse 的性能与易用性进行了一年多的打磨；**在 3.4 版本中，我们聚焦** **AI** **场景的扩展，推出了 Vector Index、Python** **UDF** **支持，并实现了 Arrow Flight 协议，**为数据分析和 AI 检索场景提供更强大的功能和性能支持，帮助用户更高效地实现业务目标。

Lakehouse 的优化也在此次版本中得到了进一步的提升：从数据文件管理、数据读取、统计辅助、查询优化到数据缓存等多个环节进行了深度优化，显著提升了系统性能。同时，湖生态对接能力也得到了持续完善，建表、数据导入和系统稳定性等方面的改进进一步增强了 StarRocks 的易用性和可靠性。

本文将深入解析 StarRocks 3.4 的新特性，带你全方位了解这款强大的湖仓分析引擎如何助力用户挖掘数据潜力，释放更多潜在价值！

**更强的 AI 场景支持**

## **Vector Index**

## 

向量数据库能够支持快速的高维向量相似性搜索，通过近似最近邻（ANN）算法，高效地找到与给定向量最相似的其他向量，这在推荐系统、自然语言处理、图像和文本检索等领域至关重要。

在 StarRocks 3.4 版本中，**我们引入了 Vector Index，支持两种重要且常用的索引类型：****IVFPQ（Inverted File Index with** **Product Quantization****，倒排****乘积量化****）****和****HNSW（Hierarchical Navigable Small World，****分层小世界图）****。**从而能够在大规模、高维向量数据中进行高效的近似最近邻搜索（ANNS），这一技术在机器学习中尤为重要。

使用方式也很简单，和普通的建表、导入、数据检索没有太大的区别。

```
-- Create an vector index based on an ARRAY field by using IVFQP (similar to HNSW)CREATE TABLE test_ivfpq (    id BIGINT(20),    vector ARRAY<FLOAT>,  
    INDEX ivfpq_vector (vector) USING VECTOR (        "index_type" = "ivfpq", ...  -- some specific parameters    ));  
-- Insert some data.INSERT INTO test_ivfpq VALUES (1, [1,2,3,4,5]), (2, [4,5,6,7,8]);  
-- Do vector index search with normal SQL filter: to find some nearest vectorsSELECT *FROM (SELECT id, approx_l2_distance([1,1,1,1,1], vector) score      FROM test_ivfpq) a WHERE id in (1, 2, 3) AND score < 40ORDER BY scoreLIMIT 3;
```

## **Python UDF[Experimental]**

##

## 

StarRocks 目前提供了数百个常用函数，同时支持 Java UDF，帮助用户实现更多业务需求中的特殊函数。然而，考虑到 AI 领域的用户更为熟悉 Python，StarRocks 3.4 版本新增了对 Python UDF 的支持。这一功能让用户可以灵活高效地定义自己的函数，便于对训练数据进行预处理、嵌入式模型推理或使用机器学习相关的 Python 库等。

使用起来也非常方便，只需通过 Python 代码就能轻松扩展功能，满足复杂的业务需求。

```
-- Create a python function with inline mode to clean a textCREATE FUNCTION python_clear_text(STRING) RETURNS STRING    type = 'Python'    symbol = 'clear_text'    file = 'inline'    input = 'scalar'  -- you can use arrow to improve the computation performanceAS$$import string  
def clear_text(input_text):    translator = str.maketrans('', '', string.punctuation)    cleaned_text = input_text.translate(translator)    return cleaned_text.lower()$$;  
-- Do cleaning text like a normal functionSELECT id, python_clear_text(str_field)FROM tbl;  -- columns of tbl: id, str_field
```

> 目前只支持用户自定义标量函数（Scalar UDF）。
>
> 为了提升 Python UDF 的处理性能，也可以使用 vectorized input，使用上基本一样。
>
> 创建复杂的处理函数时，也可以使用打成 zip 包的 python 文件。`file` 指定 zip 包地址。

## 

## **Arrow Flight[Experimental]**

##

## 

如果经常需要将数据库中的大量数据导出后做机器学习相关工作，V3.4 中提供的 Arrow Flight 就能比较好的满足这个需要。相对于 MySQL 协议，Arrow Flight 协议能够更快速、更高效地获取大批量的查询结果，更好地支持涉及多种数据源和系统的 AI 应用中跨系统间的数据协作。同时，也能减少对 StarRocks FE 的查询压力。

**全面进化的 Lakehouse 能力**

自 v3.0 起，StarRocks 正式开启了 Lakehouse 新范式。在此过程中，我们通过对存算一体功能和性能的持续对齐，提供了更高性价比的存算分离架构，并不断加强对 Iceberg、Hudi、Delta Lake 等生态的支持，推动高性能湖数据分析的实现。同时，物化视图的持续优化和建表、导入能力的简化，进一步推动了 Lakehouse 新范式的深度落地和快速发展。到 v3.4，StarRocks 在功能、性能、易用性和稳定性等方面进一步提升，全面强化了 Lakehouse 能力。

## **Data Lake Analytics**

**1**

**Iceberg v2 equality delete 读取优化**

V3.3 中，StarRocks 已经通过优化 page index 处理、使用 SIMD 优化计算、优化 ORC 小文件读取等，优化了读取 data file 时查询性能。同时，V3.3 中增加了对 Iceberg V2 表的 equality delete 的支持，从而使得用户能够高效分析使用 Flink 写入的 Iceberg upsert 数据。然而，当 upsert 操作频繁时，查询大量 Iceberg 数据时可能会重复读取相同的 delete file（一个delete file 可能对应多个data file），导致性能下降并占用大量内存。

为了优化这一问题，3.4 版本通过单独生成一个 Iceberg eq-delete scan node 的方式改写 Query plan，采用 shuffle 或 broadcast delete file 数据，从而避免重复读取。**在 delete file 较多的场景中，****查询性能可****提升数倍。**

### **2** **异步投递查询片段(Async scan framework)**

在涉及大量文件的数据湖查询中，FE 获取所有查询文件的列表往往需要消耗较长时间。StarRocks 3.4 版本实现了异步投递查询片段的功能，使得 FE 获取文件列表与 BE 执行查询可以并行进行，从而能够减少 FE 获取文件列表对整个查询延迟的影响。对于尚未缓存文件列表的冷查询，整个查询的时延得到显著缩短，同时在保证查询性能的前提下，用户也可以适当减少文件列表缓存的大小，从而缓解内存消耗。

### **3** **Data cache 优化和统一**

除了不断进行对于数据文件的读取优化，Data cache 也是 StarRocks 能提供优异查询性能的重要一环。此前版本已支持缓存预热、异步填充、优先级控制、动态参数调节等等功能，在 StarRocks 3.4 版本中，针对偶发大查询污染场景和易用性做了进一步改进，以提升缓存性能并应对偶发大查询对整体查询的影响。

通过引入全新的 Segmented LRU (SLRU) 缓存淘汰策略，将缓存空间分为淘汰段和保护段，分别采用 LRU 策略进行管理，从而有效减少偶发大查询引起的缓存污染问题，提升缓存命中率并减少查询性能波动。**在模拟测试中，查询性能提升幅度可达 70% 至数倍。**

此外，我们还优化了 Data cache 的自适应  I/O 策略，使得系统能够根据缓存盘的负载和性能，自适应地将部分查询请求路由到远端存储。**这一优化在高负载场景下显著提升了系统的访问吞吐能力，查询性能可提升 1 倍至数倍。**

为了提升易用性，我们统一了存算分离和数据湖查询中 Data Cache 的内部实例、配置参数和观测指标。这不仅简化了配置过程，还优化了资源的使用率，因为不再需要为每个实例单独预留资源。

### **4** **深入裁剪和谓词下推**

在持续优化湖数据分析相关查询性能的同时，StarRocks 也没有忽视查询引擎本身的性能提升。本版本进一步深入分析了常见的查询模式（query patterns），实现利用主外键进行表裁剪、主外键裁剪聚合列、聚合下推位置改进、多列 OR 谓词下推等优化。在 TPC-DS 1TB Iceberg 数据 benchmark 测试中，StarRocks 实现了 20% 的查询性能提升。

### **5** **利用数据统计和历史查询优化查询计划**

除了优化硬查询能力，StarRocks 也在持续探索如何更好地利用统计数据和历史查询计划来进一步提升查询性能。在本版本中，我们支持了查询自动触发 ANALYZE 任务以采集外表统计信息的功能。相比传统的 metadata file，这种方式还能收集更精确的 NDV 信息，从而优化 Query Plan，提升查询性能。而且，这一功能无需用户手动干预或额外学习，真正实现了开箱即用。

同时，V3.4 还提供了 Query Feedback 功能，这是利用历史查询计划优化未来查询性能的一项重要进展。通过该功能，系统会持续收集慢查询，根据执行详情自动分析一条 SQL 的Query Plan 是否存在潜在的优化空间，并可能生成专属的调优建议。当后续遇到相同的查询时，系统能够对此 Query Plan 进行局部调整，以生成更高效的 Query Plan，从而进一步提升查询性能。

### 

### **6** **Data Lake 生态支持**

### 

### 

在持续提升查询性能的同时，StarRocks 也在不断完善 Data Lake 生态的支持。本版本中，我们支持了 Iceberg Time Travel 功能，允许用户创建或删除 BRANCH 和 TAG，并通过指定 TIMESTAMP 或 VERSION 来查询特定分支或标签的数据。这使得用户能够轻松访问不同版本的数据，对于恢复意外删除的数据尤为重要。同时，也支持在不同分支上进行数据写入，以便进行并行开发测试、变更隔离等。

```
-- Create a BRANCHALTER TABLE iceberg.sales.order CREATE BRANCH `test-branch`AS OF VERSION 12345  -- Create a BRANCH on the specified snapshot idRETAIN 7 DAYSWITH SNAPSHOT RETENTION 2 SNAPSHOTS;  
-- Write data into the BRANCHINSERT INTO iceberg.sales.order [FOR] VERSION AS OF `test-branch`SELECT k1, v1 FROM tbl;  
-- Query data on the BRANCHSELECT * FROM iceberg.sales.order VERSION AS OF `test-branch`;  
-- You can fast forward your main branch to the test-branch after you've finished a testALTER TABLE iceberg.sales.orderEXECUTE fast_forward('main', 'test-branch');
```

此外，本版本还新增了对 Delta Column Mapping 功能的支持，用户现在可以查询经过Delta schema evolution 处理的 Delta Lake 数据。

## **更易用的建表和导入**

在专注极速查询性能的同时，StarRocks 也持续关注易用性的提升。在本版本中，我们继续优化了建表和数据导入功能，增加了多个新功能，显著提升了易用性。

### **1** **建表**

在建表方面，StarRocks 提供了 4 种表类型，其中聚合表也是用于提升查询性能的常见选择，特别是在能够通过聚合原始数据来加速查询的场景。过去，聚合表仅支持 SUM、MIN、MAX、BITMAP\_UNION 等少数几个基础聚合函数，无法满足更复杂的聚合需求。在此次版本中，StarRocks 引入了通用的聚合函数状态存储框架，使得聚合表能够支持几乎所有 StarRocks 支持的聚合函数，极大地丰富了用户的聚合选择。此功能不仅适用于聚合表，还能在同步和异步物化视图中使用，并支持查询透明改写，进一步满足了用户在各种场景下进行预聚合加速查询的需求。

* 用法上，基本和原有方式一样定义聚合表，只是聚合列指定具体的通用聚合函数即可。然后在使用时，根据不同场合，需要使用到几个不同的函数来处理和实现聚合。

  ```
  -- Create a AGGR table with some aggregate columnsCREATE TABLE test_create_agg_table (  dt VARCHAR(10),  v BIGINT sum,  -- normal aggregate columns  avg_agg avg(bigint),  -- new general aggregate functions  array_agg_agg array_agg(int),  min_by_agg min_by(varchar, bigint))AGGREGATE KEY(dt)PARTITION BY (dt)DISTRIBUTED BY HASH(dt) BUCKETS 4;
  ```

  在创建表后，根据不同使用场景需要，选择聚合函数对应的不同的 combinator 函数：

+ 对于导入数据，需要使用 `xxx_state(original_value)` 函数来转化数据为中间态数据并存储到 StaRocks 中。以聚合函数 `avg` 为例，中间态数据是对输入数据 value 分别求 sum 和 count 后的两个值（实际为一个 binary 类型字段），并存储，而不是一个原值 value。

  ```
  INSERT INTO test_create_agg_tableSELECT dt, v,  -- directly use v when the aggregate func is SUM    avg_state(id),  -- use xxx_state() to get the intermediate value    array_agg_state(id),    min_by_state(province, id)FROM t1;
  ```
+ 对于中间态数据做聚合并继续存储为中间态数据时，需使用 `xxx_union(mid_value)`来**部分聚合**中间态数据。对于 `avg` 来说，相当于把中间态的 sum/count 做局部聚合，并继续存储为 sum/count 两个值。

  ```
  SELECT dt, sum(v),    avg_union(avg_agg),    array_agg_union(array_agg_agg),    min_by_union(min_by_agg)FROM test_create_agg_tableGROUP BY dt;
  ```
+ 根据中间态数据计算最终聚合结果时，需要使用 `xxx_merge(mid_value)`。对于 `avg`，相当于计算 `sum/count` 来最终得到 `average(id)`。

  ```
  SELECT dt, sum(v),    avg_merge(avg_agg),    array_agg_merge(array_agg_agg),    min_by_merge(min_by_agg)FROM test_create_agg_tableGROUP BY dt;
  ```

* 对于物化视图，则可以直接使用 `xxx_union(xxx_state(origin_value))` 来定义聚合列：

  ```
  CREATE MATERIALIZED VIEW test_mv1 ASSELECT dt, min(id) as min_id, sum(id) as sum_id,  -- old aggr funcs    avg_union(avg_state(id)) as avg_id,    array_agg_union(array_agg_state(id)) as array_agg_id,    min_by_union(min_by_state(province, id)) as min_by_province_idFROM t1GROUP BY dt;
  ```

使用时则很简单，直接在 base 表上调用聚合函数即可，系统会自动进行透明改写。

```
SELECT dt, min(id), sum(id)    avg(id),    array_agg(id),    min_by(province, id)FROM t1GROUP BY dt;
```

在分区方面，StarRocks 支持 Range 分区、表达式分区和 List 分区三种方式，虽然提供了灵活的选择，但也增加了学习和理解的难度。尤其是，表达式分区过去仅支持简单的表达式，且函数支持也有限，如仅有 date\_trunc 等少数几个函数。**此次版本中，StarRocks 对分区方式进行了统一，采用了表达式分区，并支持多级分区的灵活配置，**每一级都可以使用任意表达式。语法更加简洁、强大。

```
CREATE TABLE multi_column_part_tbl (    dt DATETIME,    tenant_id INT,    region_id INT,    event STRING)PARTITION BY tenant_id, date_trunc('month', dt), FLOOR(FLOOR(region_id/10000) * 10000)DISTRIBUTED BY RANDOM;
```

* 这里由 tenant ID、month、以及 region 区间组合成分区列，其中 region 区间是一个函数表达式。从而将数据按照 tenant ID 划分的同时，也按照 month 粒度进一步来划分分区，以适用于一般只查较新数据的场景，并且还按照 region 的大区间进行划分，以适合具有区域查询特点的场景。

### 

### 

## **2** **导入**

### 

### 

**2.1 INSERT & FILES**

从 V3.1 开始，StarRocks 就引入了 FILES 表函数，用户可以通过 INSERT from FILES 简单实现数据导入。此次版本，我们对该功能进行了全面优化，使得 INSERT from FILES 基本可以替代 Broker Load，成为批量数据导入的首选方式，并且使用起来更加简便、直观。具体优化包括：

* 支持 LIST 远程存储中的目录和文件，用户可以在导入前轻松查看和核对数据文件及目录结构；
* 引入了 BY NAME 映射方式，能将数据文件中的列映射到目标表中列，特别适用于列数较多且列名相同的情况；
* 支持下推 target table schema 及合并不同 schema 的文件，更好地自动推断和处理导入数据的 schema，让导入变得更简单智能；
* 新增了 PROPERTIES 子句，支持 strict\_mode、max\_filter\_ratio 和 timeout 等参数，从而 INSERT 中也可以根据导入数据的质量控制导入行为。

从而，一个复杂的导入都可以按如下方式轻松地实现：

```
-- 1. List the data files, check it whether they are indeed what you want to loadSELECT * FROM FILES(    'path' = 's3://bucket/path/to/file',    'list_files_only' = 'true',  -- list file, not the data content    'list_recursively' = 'true',);  
-- 2. View the data schema by DESC or direct SELECTDESC FILES(...);SELECT * FROM FILES(...) LIMIT 5;  
-- 3. Use a simple CTAS to create a table and load some dataCREATE TABLE ctas_tbl ASSELECT * FROM FILES (...) LIMIT 10;  
-- 4. Now, it's time to create the final target table, and load the data.-- 4.a Create a table, refer to the schema of `DESC` command or SELECT statement.CREATE TABLE fine_tbl(    id INT,    v2 VARCHAR(16),    v3 DECIMAL(18, 10));  
-- 4.b Load the data, no matter the data have done some schema change/evolution or notINSERT INTO fine_tbl BY NAMEPROPERTIES (    'strict_mode' = 'true',    'max_filter_ratio' = '0.001',  -- only 0.1% error rows at most are allowed    'timeout' = '3600')SELECT * FROM FILES(    'path' = 's3://bucket/path/to/dir/*/*',  -- columns from files are: v2, id, v3    'format' = 'parquet',    'fill_mismatch_column_with' = 'NULL'  -- fill NULL if some columns are missed in some files);
```

* `/path/to/dir/` 目录下一些文件有 3 列，一些文件可能只有 2 列甚至 1 列，FILES 支持合并不同 schema 的文件，并提供 `fill_mismatch_column_with` 参数以控制有缺失列时的行为：补`NULL`或报错。
* 数据文件中的 `id` 列可以直接按照 `INT` 读取数据，并可能过滤溢出的数据。（通过 FE 配置项 `files_enable_insert_push_down_schema` 设置）
* 最终数据文件中的列可以直接按照名字映射导入到目标表中，而不需要一列列对应设置，对于导入几百上千列时的大宽表特别方便。

结合 FILES 表函数，还可以在 SELECT 语句中实现更复杂的数据转换、JOIN 等操作，满足更多的应用场景。因此，当需要从云存储导入数据时，INSERT from FILES 将是最优的选择。

**2.2 INSERT OVERWRITE**

除了用于数据导入，INSERT 还广泛应用于 ETL 过程。用户常通过 INSERT INTO SELECT 实现数据的批量处理，将数据导入到另一个表中，甚至形成一个 pipeline 处理。同时，遇到需要修复数据场景时，可以使用 INSERT OVERWRITE 来修复错误数据。

在本版本中，INSERT OVERWRITE 新增支持 Dynamic Overwrite 语义。系统能够自动根据导入的数据创建分区，并只覆盖包含新数据的分区，未涉及的老分区将不被删除。这对于那些在修复数据时仅需覆盖部分分区、但无法明确指定分区或不易指定分区的场景尤为方便。由于新导入的数据可能涉及不存在的分区，而修复数据可能涉及多个分区，且由于使用表达式分区或多列 List 分区，用户可能无法清楚了解所有分区的名称，因此让系统自动确定并替换分区的方式显得尤为实用。

使用方法很简单，设置 session 变量 `dynamic_overwrite = true`即可。

```
SET dynamic_overwrite = true;  
-- Use INSERT OVERWRITE just as normalINSERT OVERWRITE part_tblSELECT * FROM nonpart_tbl;
```

当然，也可以设置 `dynamic_overwrite = false` 以让系统替换所有分区。

**2.3 M****erge commit**

除了上述的易用性改进，本版本的 StarRocks 还新增了 merge commit 功能，以优化高并发实时数据导入的场景。merge commit 功能能够将同一时间区间内针对同一表的多个并发 Stream Load 合并为一个导入事务进行提交，从而提升实时数据导入的吞吐能力，**尤其适用于高并发、小批量（从** **KB** **到几十** **MB****）的实时导入场景**。

此外，merge commit 还能够有效减少因版本过多引发的版本数量限制问题，降低后续需要进行数据版本 Compaction 合并时的资源消耗，减少小文件过多所导致的数据查询时 IOPS 增加和 I/O 延迟，从而提升查询性能。

* FE 会按照作业配置的允许合并时间划分为一个个时间区间，将一个时间区间内的很多小批量导入作业合并提交为一个新的导入作业，再类似于 Broker Load 一样进行数据导入。

使用方式和原来的 Stream Load 方式基本一样，加入几个 merge commit 控制参数即可：

```
curl --location-trusted -u <username>:<password> \    -H "Expect:100-continue" \    -H "column_separator:," \    -H "columns: id, name, score" \    -H "enable_merge_commit:true" \    -H "merge_commit_interval_ms:1000" \    -H "merge_commit_parallel:2" \    -T example1.csv \    -XPUT http://<fe_host>:<fe_http_port>/api/mydb/tbl/_stream_load
```

## **Stability & Security**

##

## 系统的高可用性是作为基于数据多副本的 StarRocks 系统天生注重的一个能力。V3.4 中，StarRocks 提供了 Graceful Exit 功能，确保在 BE/CN 节点退出时查询能够在短时间内继续执行完毕，从而尽量减少节点升降级过程对业务的影响，进一步提升系统的高可用性。

此外，通过实现 Follower FE 支持 Checkpoint，减少了对 Leader FE 的内存压力，进一步增强了系统的稳定性。在细节方面，诸如日志打印优化等小改进，都有效提升了系统的稳定性和整体表现。

在数据保护与容灾方面，StarRocks 在 V3.2 中引入的 Cluster Sync 功能提供了高可用的容灾能力，但备份/恢复依然是最基础且高性价比的容灾手段。因此，StarRocks 3.4 版本进一步完善了备份功能，支持备份 Logical View、External Catalog 等更多对象，同时也支持备份采用表达式分区和 List 分区的表；并且，也优化了备份性能，能达到了每节点 1GB/s 的备份速度，基本能够满足大部分场景的基础需求。

****👇更详细的 feature 介绍参考：****

> Release note：https://docs.mirrorship.cn/zh/releasenotes/release-3.4/
>
> 下载：https://www.mirrorship.cn/zh-CN/download/starrocks

**关于 StarRocks**

StarRocks 是隶属于 Linux Foundation 的开源 Lakehouse 引擎 ，采用 Apache License v2.0 许可证。StarRocks 全球社区蓬勃发展，聚集数万活跃用户，GitHub 星标数已突破 9500，贡献者超过 450 人，并吸引数十家行业领先企业共建开源生态。

StarRocks Lakehouse 架构让企业能基于一份数据，满足 BI 报表、Ad-hoc 查询、Customer-facing 分析等不同场景的数据分析需求，实现 "One Data，All Analytics" 的业务价值。StarRocks 已被全球超过 500 家市值 70 亿元人民币以上的顶尖企业选择，包括中国民生银行、沃尔玛、携程、腾讯、美的、理想汽车、Pinterest、Shopee 等，覆盖金融、零售、在线旅游、游戏、制造等领域。

**行业优秀实践案例**

**泛金融：**[中国民生银行**｜**](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)[平安银行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493044&idx=1&sn=580ea0a521f9c9e4099356022536a585&chksm=e9f29a90de851386c04da4c3810b3c1ce0fce8e7f128fcbee0fb1c1a480a17aa5582325f20ee&scene=21#wechat_redirect)｜[中信银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[四川银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[南京银行](https://mp.weixin.qq.com/s?__biz=MzkxOTQyNzU1NQ==&mid=2247484420&idx=1&sn=54b813186be038ad1db9db7957eb5a19&scene=21#wechat_redirect)｜[宁波银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中原银行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487331&idx=1&sn=e6cdcdf2c46762135b6d95bf58e47664&chksm=e9f17047de86f9519e4d58f47745fe64788f2a5b3117133fc4b39db1a50c50d163a467f20950&scene=21#wechat_redirect)｜[中信建投｜](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488978&idx=1&sn=68da6c95be3337048c5900ce70d56c5e&scene=21#wechat_redirect)[苏商银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247487346&idx=2&sn=263c5029751e0ec8ed81df1760deb1d6&scene=21#wechat_redirect)｜[微众银行](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[杭银消费金融](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[马上消费金融](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中信建投](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[申万宏源](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492791&idx=1&sn=e4eba3e3d3de2fc4e59148a2e52f7201&chksm=e9f29b93de851285bf00f6e94a0f096b24d9a494b0013d43fb42d006eabea27a615e605b5a1b&scene=21#wechat_redirect)[｜](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247485524&idx=1&sn=4a5d0f877d39ed674cbc0b8ba03847c6&scene=21#wechat_redirect)[西南证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中泰证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[国泰君安证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[广发证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[国投证券](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中欧财富](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247485524&idx=1&sn=4a5d0f877d39ed674cbc0b8ba03847c6&scene=21#wechat_redirect)｜[创金合信基金](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[泰康资产](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[人保财险](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)

**互联网：**[微信｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484928&idx=1&sn=87d2572bba55afef5abd5f7b8571a443&chksm=e9f17924de86f032c3dca0fdc69b9b8ab8b873bc9a5bc4fdb284c06c8539da0e24e6647ded93&scene=21#wechat_redirect)[小红书｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484997&idx=1&sn=8644dacbcf467f6946add574b738e5d9&chksm=e9f17961de86f0776f5cbb201601010374c7e6c81b0a3250d93a32a997edbf694f4d471185eb&scene=21#wechat_redirect)[滴滴](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490839&idx=1&sn=a007c5207f15129cd8c91da15cc30093&chksm=e9f16233de86eb25f585ef9b911e54ae6dae9547bd240a9554dd52ba449a74289a13d3df99fa&scene=21#wechat_redirect)｜[B站](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492334&idx=1&sn=dd0de6197bc6f3f67ffb0704b72ae02f&scene=21#wechat_redirect)｜[携程](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491637&idx=1&sn=daec09546a550346b65414567dc8db7b&chksm=e9f29f11de851607eb14a9e006803d8f52951d367f19a2477d5abee92f309c8c8145655a959c&scene=21#wechat_redirect)｜[同程旅行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490253&idx=1&sn=93f2a0369f9a9bff2215031efd197530&chksm=e9f165e9de86ecffe737cfebf729668df7efe09e6f9c4c976dc17668dc8042ef7895aef74ea3&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488632&idx=1&sn=0f76b5ce855b3070a138cb58e5fb8112&chksm=e9f16b5cde86e24a853e743df776e25d2358e710442cb419d32c3dd504d8e328e06b5cb473fd&scene=21#wechat_redirect)[芒果TV](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490596&idx=1&sn=9de0a6ce3608875f17287a197b7847b1&chksm=e9f16300de86ea16b83b7f5d8a29b20c137b1bc641d4d190c4d73d717a56d8c104a1f585f4d7&scene=21#wechat_redirect)｜[得物](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487306&idx=1&sn=46b784eb29c1a4d645946a017deb7f8a&chksm=e9f1706ede86f97861ce9ed2952335312704fe2cc9adc35b3cfa5709f2abdeea9a2779ff18d6&scene=21#wechat_redirect)｜[贝壳](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490822&idx=1&sn=a8bc0b31aa6568f57e919632e3f15877&chksm=e9f16222de86eb34d2a4eabd28c36ff2b29f3d9ad72855e8a9064d3b84b24a1e13970a80d3e3&scene=21#wechat_redirect)｜[汽车之家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484870&idx=1&sn=74ab3350c6b7e86376c009b3ee6b5b98&chksm=e9f17ae2de86f3f45cc32c4ae153b61c57cdf6e66dd7b10ae9fb8ec689994598130257c43936&scene=21#wechat_redirect)｜[腾讯大数据](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492960&idx=1&sn=f69f94be1400c0d3260e5c1b0aaa48cb&chksm=e9f29a44de8513520a3d4e6db9abb3936c806ad82ce7194421147c96dac10638cbae5ee817dd&scene=21#wechat_redirect)｜[腾讯音乐](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493768&idx=1&sn=5076f547377aeb56ba351b03fae0730f&chksm=e9f297acde851eba21783c944fe7da28ccafe8eb86063e2100672f7982afd449441f47e73a3e&scene=21#wechat_redirect)｜[饿了么](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247494190&idx=1&sn=89f15ad79ef3aa83db4a6f9cb9ee8bce&chksm=e9f2950ade851c1ce73cb480fcab5f714564ea21f69ef75d565f02750d7760ee3128bc2319da&scene=21#wechat_redirect)｜[七猫](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247494193&idx=1&sn=61b083ca4befbb0fe951131a5efd4687&chksm=e9f29515de851c0383357500c59f07cda799d6a51ea04721326a9af54db0642068dbb6c68645&scene=21#wechat_redirect)｜[金山办公](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493868&idx=1&sn=ad2326665f7aa0b42358af6ceab0fbe9&chksm=e9f297c8de851ede838a7db7a6bbd78d735711a420e26092878f7c334da50fcc0540c648775b&scene=21#wechat_redirect)｜[Pinterest](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493896&idx=1&sn=673579fbfcaf50fe5577b2ec91c5639a&chksm=e9f2962cde851f3a6d60b30eac4c3cabd033307c0456e3cac048a934891a46c83e76dbc75eb9&scene=21#wechat_redirect)｜[欢聚集团](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486170&idx=1&sn=bd5de60aa15f03a6972467c45c58bb9b&chksm=e9f175fede86fce8c2495df1d7f6f3096b70ec8ac69ee94e12f96f3e15c256a85a94c3e3c5f7&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490839&idx=1&sn=a007c5207f15129cd8c91da15cc30093&chksm=e9f16233de86eb25f585ef9b911e54ae6dae9547bd240a9554dd52ba449a74289a13d3df99fa&scene=21#wechat_redirect)[美团餐饮](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488130&idx=1&sn=53c32ae5448505db505872e0d8e8d7e1&chksm=e9f16da6de86e4b03a1295902ac27ff3fe9a554dd76383b6d24c155fb11778bc4d4afa7d9494&scene=21#wechat_redirect)[｜](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492334&idx=1&sn=dd0de6197bc6f3f67ffb0704b72ae02f&scene=21#wechat_redirect)[58同城](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487038&idx=1&sn=583cf978f57802edd653034d83ac30af&chksm=e9f1711ade86f80cc37de844768228ba7e6a6abc06dd8b4867303334d31334e7b23273d12c39&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490253&idx=1&sn=93f2a0369f9a9bff2215031efd197530&chksm=e9f165e9de86ecffe737cfebf729668df7efe09e6f9c4c976dc17668dc8042ef7895aef74ea3&scene=21#wechat_redirect)[网易邮箱](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488632&idx=1&sn=0f76b5ce855b3070a138cb58e5fb8112&chksm=e9f16b5cde86e24a853e743df776e25d2358e710442cb419d32c3dd504d8e328e06b5cb473fd&scene=21#wechat_redirect)｜[360](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485899&idx=1&sn=c8b13a6a6f9d0d59de1721a36f52c4cd&chksm=e9f176efde86fff974e81dfe2fb72a834528141a9b0f4a99f647b11c1a2e9001822b82ddbd1e&scene=21#wechat_redirect)｜[腾讯游戏｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486696&idx=1&sn=d4bfaf2c2fb84a890922a0dfd4879522&chksm=e9f173ccde86fadaa1313f3e1c82d71f44b3e244113c712a272d3c85543765752431f8896ae6&scene=21#wechat_redirect)[波克城市](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486150&idx=1&sn=bc272d1174edb74233c584c1b7f970da&chksm=e9f175e2de86fcf4b78f09ce88ead3dc591e11edd11a3efafe74441591928d37c213b9871e7d&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486170&idx=1&sn=bd5de60aa15f03a6972467c45c58bb9b&chksm=e9f175fede86fce8c2495df1d7f6f3096b70ec8ac69ee94e12f96f3e15c256a85a94c3e3c5f7&scene=21#wechat_redirect)[37手游](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486749&idx=1&sn=610a64862a91df789faaa2469f2c8116&chksm=e9f17239de86fb2f78abd9f391d3c780f371213c93150d8617bb1c9a1845b837293f203f7d41&scene=21#wechat_redirect)｜[游族网络｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487470&idx=1&sn=e8844ab1037069989e3dc95f5e9092e8&chksm=e9f170cade86f9dcd62d810c6f7132002c4cf7329dbf20f232db82ca6cca955ffd934be04e0c&scene=21#wechat_redirect)[喜马拉雅｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247494895&idx=1&sn=f09d7e5262feb121d12b486d8bd88d29&chksm=e9f293cbde851add02bc6d438118e530d7b569b5d3b788d1abe9237edcb76542f9a62c749b58&scene=21#wechat_redirect)[Shopee](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247495088&idx=1&sn=a42354398d35329e62241705d3e79eec&chksm=e9f29294de851b82d2b55a7c2cf8d5f75a37f1025c6a1f1b07e9fdbddddc865ea55560dffafa&scene=21#wechat_redirect)

**新经济：**[蔚来汽车｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491009&idx=1&sn=d95f01aea67e68b617cb891be823dd9a&chksm=e9f162e5de86ebf310ec56cbe8c950a69ef5a311b8213bb3adaf843069f9f315a5bede006cdd&scene=21#wechat_redirect)[理想汽车｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485820&idx=1&sn=7aa47ec35feec64e49ea462fe7506864&chksm=e9f17658de86ff4e1bd61f12d70ee8b478cc0ecd8fa7860df72ce2b5dc4b8ac05c77235809d6&scene=21#wechat_redirect)[吉利汽车](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247485625&idx=1&sn=20dabf0908da3aa394f6b2553c363141&scene=21#wechat_redirect)｜[顺丰｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484829&idx=1&sn=344fb330aac7d722a57a9591a82ca019&chksm=e9f17ab9de86f3af2e6d995b9ff8493c2a4d8e992d30e205904e3bd0ba88075061ea0158cb82&scene=21#wechat_redirect)[京东物流｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486458&idx=1&sn=88058c480d5f163155bb885e3d1fbc2a&chksm=e9f174dede86fdc8defec71ca637bc2a5a9c87dd245c6bf9c51e11f2811fcdbcd507776ccd71&scene=21#wechat_redirect)[跨越速运](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=2&sn=7912601d564276718811d53c60116a69&chksm=e9f16e7dde86e76ba4a28a99faf69c33a0553e4d383d4e083c3e0387e1e6d088fd13094c055b&scene=21#wechat_redirect)｜[沃尔玛](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[屈臣氏](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[麦当劳](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[大润发｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488098&idx=1&sn=cae2b42fd8fdb77ed4e67fc83173291f&chksm=e9f16d46de86e450859f842b471de1ee4e57118901f6f1f7f2c545fefdbdfbe355c9dc3e7965&scene=21#wechat_redirect)[华润集团｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487047&idx=1&sn=74201f01931823c9cdc6abc71d3dba68&chksm=e9f17163de86f875439549071e7c79d3c2762cf245b69367be7024ed9d04380816da4348b38d&scene=21#wechat_redirect)[TCL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=1&sn=cb78e47e420191b2345aff1366709384&chksm=e9f16e7dde86e76b77ecfd0d72d164313e18ef2fad887c3e41d30693bfd5e79629018be53a6a&scene=21#wechat_redirect) ｜[万物新生](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490969&idx=1&sn=22a377677f403c37579b2309686d559b&chksm=e9f162bdde86ebab62ce4bd3905d63870947f5b42076fc4d3d1c60a0303e2565a01cb5e2b644&scene=21#wechat_redirect)｜[百草味](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487394&idx=2&sn=242d444b7c02721b503de9383329edcd&chksm=e9f17086de86f990d2e18132c2b9beb3fa3984f5accb03cb34e0d479e8575666426c5508913c&scene=21#wechat_redirect)｜[多点 DMALL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484971&idx=1&sn=8f7e7ec424335b81fb34f1da85efbf8f&chksm=e9f1790fde86f019016e67a8a820f866bcbffe484414fa93b403605f957874bb7450672bfc3d&scene=21#wechat_redirect)｜[酷开科技](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486836&idx=1&sn=399086d79a513d311083c30ec7746589&scene=21#wechat_redirect)[｜vivo](https://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492884&idx=1&sn=48b9e9c521654f22576c0a28bb5dbf5b&scene=21#wechat_redirect)｜[聚水潭](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492598&idx=1&sn=0d25b7c62770e39a0961acd9501e4ad5&chksm=e9f29cd2de8515c400f7c6076ef91d6f13121d6f0dfd0f2743fb2afc16d07fc8e61d440a330a&scene=21#wechat_redirect)｜[泸州老窖](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[中免集团](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[蓝月亮](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[立白](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[美的](https://mp.weixin.qq.com/s?__biz=MjM5NzIzODg0MQ==&mid=2656464503&idx=2&sn=765854a37cc14f3032699266110c51b0&scene=21#wechat_redirect)｜[伊利](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)｜[公牛](https://mp.weixin.qq.com/s?__biz=Mzg5MTg0NDM2OQ==&mid=2247489219&idx=1&sn=20700c3ec7daf32b6f036b8559615761&scene=21#wechat_redirect)