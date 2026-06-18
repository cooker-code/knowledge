---
title: hive面试必备题
author: 浪尖聊大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzUyMDA4OTY3MQ==&mid=2247511106&idx=1&sn=7a8415a1cc45e48ad7a2e5519080b019&chksm=f9ed556ace9adc7c211ef0ce599423d5bb68a2dfce0782890c22d7b4a3fd5d5f5bba5bad9b3c&mpshare=1&scene=24&srcid=0407OhfVOeC3x1rjELvk0uAr&sharer_shareinfo=cab67c46b6537601937ba53e97ea64a1&sharer_shareinfo_first=cab67c46b6537601937ba53e97ea64a1#rd
---

1. Hadoop中两个大表实现JOIN的操作

在Hadoop和Hive中处理两个大表的JOIN操作通常涉及以下策略：

* 利用Hive分区：通过在创建表时定义分区策略，可以在执行JOIN时只处理相关的分区数据，减少需要处理的数据量。
* 优化HQL语句：选择性查询所需字段而非全表、全字段查询，减少数据加载和处理时间。使用适当的JOIN策略，比如利用`/*+ MAPJOIN(小表名) */`提示对小表使用MapJoin，以及设置`hive.auto.convert.join=true`让Hive自动选择最佳JOIN策略。

### 2. Hive中存放是什么？

Hive存储的是逻辑上的数据仓库信息，包括表的定义、数据的存储位置（HDFS路径）、分区和表的元数据等。实际的数据文件存储在HDFS上，Hive通过HQL（Hive Query Language）实现对这些数据的SQL-like查询，本质上是将SQL查询转换为MapReduce任务在Hadoop上执行。

### 3. Hive与关系型数据库的关系

Hive是基于Hadoop的数据仓库工具，与传统的关系型数据库在本质上有所不同。Hive主要用于数据分析和处理大规模数据集，支持一次写入多次读取的操作模式，而不适合实时的CRUD操作。相较于关系型数据库，Hive的设计重点是高效地执行大规模数据集的批量处理和分析，而不是低延迟的数据交互。

### 4. Hive中的排序关键字

Hive提供了多种排序关键字，适用于不同的排序和数据分发需求：

* ORDER BY：实现全局排序，但因为只使用一个Reducer，处理大数据量时效率较低。
* SORT BY：在每个Reducer内进行排序，但不保证全局排序。适用于数据量大且对全局排序要求不高的场景。
* DISTRIBUTE BY：按照指定字段对数据进行分发，使得相同键值的数据被分配到同一个Reducer。常与SORT BY联合使用，以在分发的基础上实现排序。
* CLUSTER BY：等同于同时使用DISTRIBUTE BY和SORT BY，当两者的字段相同时使用。它既按指定字段分发数据，又在每个Reducer内排序。

### 5. 大表和小表JOIN

在处理大表与小表的JOIN操作时，可以使用Map Side Join（MapJoin）策略：

* 将小表加载到内存中，使每个Map任务都保有一份小表的副本（例如存储在HashMap中）。这样，Map任务在处理大表的数据时，可以直接在内存中查找小表的匹配项，大大减少数据shuffle和排序的开销，提高JOIN操作的效率。

### 注意事项

* 在应用上述策略时，需要考虑数据的实际分布和大小，以及集群的资源和配置，以确保查询执行的效率和稳定性。
* 使用MapJoin时，确保小表的大小不会超过配置参数`hive.mapjoin.smalltable.filesize`的限制，以免导致内存溢出。

### 6. 如何使用Spark进行数据清洗

数据清洗目的是提高数据质量，包括完整性、唯一性、一致性、合法性和权威性。使用Spark进行数据清洗，可以有效处理大规模数据集：

* 完整性：使用`.filter()`去除缺失关键信息的记录，或`.na.fill()`填充缺失值。
* 唯一性：使用`.dropDuplicates()`去重，保留唯一记录。
* 一致性：对数据格式不一致的情况使用`.withColumn()`结合自定义函数UDF转换为统一格式。
* 合法性：使用`.filter()`结合正则表达式等校验数据合法性。
* 权威性：根据业务规则，通过`.join()`关联权威数据源，修正或验证数据。

示例代码：

```
import org.apache.spark.sql.SparkSessionimport org.apache.spark.sql.functions._  
val spark = SparkSession.builder.appName("DataCleaning").getOrCreate()  
val df = spark.read.option("header", "true").csv("path/to/your/data.csv")  
// 完整性 - 去除某列缺失值val nonNullDf = df.filter(col("someColumn").isNotNull)  
// 唯一性 - 去重val uniqueDf = nonNullDf.dropDuplicates("idColumn")  
// 一致性 - 格式转换val standardizedDf = uniqueDf.withColumn("dateColumn", to_date(col("dateColumn"), "yyyy-MM-dd"))  
// 合法性 - 过滤非法记录val validDf = standardizedDf.filter(col("someColumn").rlike("yourRegex"))  
// 权威性 - 数据修正// 假设authorityDf是一个权威数据源DataFrameval correctedDf = validDf.join(authorityDf, validDf("keyColumn") === authorityDf("keyColumn"), "left_outer")
```

### 7. Hadoop中的二次排序

Hadoop中实现二次排序主要依赖于自定义排序策略：

* 定义一个复合键（CompositeKey），该复合键包括需要排序的主键和次键。
* 实现自定义的Partitioner，确保相同的主键分配到相同的Reducer。
* 实现自定义的SortComparator，根据复合键中的主键和次键进行全局排序。
* 实现自定义的GroupingComparator，确保具有相同主键的记录分到同一个Reducer中的同一组。

### 8. Hadoop常见的join操作

* Reduce Side Join：最常见的Join类型，适用于大表与大表之间的Join，但由于涉及大量的数据shuffle，性能较低。
* Map Side Join：适用于大表与小表的Join，小表先加载到内存中，大表在Map阶段直接与之Join，减少了shuffle。
* Semi Join：通过在Map阶段过滤不需要参与Join的数据来减少数据量，减轻了网络IO和计算压力。

### 9. Hive优化策略

* 数据存储及压缩：选择合适的存储格式（如ORC、Parquet）和压缩方式（如Snappy、GZIP）可以显著减少存储空间并提高IO效率。
* 调参优化：合理配置并行度、内存和执行计划等参数，以提升执行效率。
* 数据集规模优化：通过对大表进行分区和分桶，减小单次查询处理的数据量。
* SQL优化：优化查询语句，如合理使用JOIN策略，避免全表扫描，仅查询需要的字段等，以提高查询性能。

**10.窗口函数及对应代码案例**

Hive窗口函数允许对数据集进行复杂的聚合计算，而不需要对数据进行分组。窗口函数可以在SELECT语句的OVER子句中指定，并可以对数据集中的每行进行计算，同时还可以访问行之间的关系。窗口函数主要分为以下几类：

### a. 排名函数

* `ROW_NUMBER()`: 对每个分区的结果集行进行唯一编号。
* `RANK()`: 在结果集分区内对行进行排名，相同值会得到相同的排名，但之后的排名会留空。
* `DENSE_RANK()`: 类似于`RANK()`，但之后的排名不会留空。

### b. 分析函数

* `LEAD()`: 返回当前行之后的指定行的值。
* `LAG()`: 返回当前行之前的指定行的值。
* `FIRST_VALUE()`: 返回窗口中的第一个值。
* `LAST_VALUE()`: 返回窗口中的最后一个值。

### c. 聚合函数

聚合函数（如`SUM()`, `AVG()`, `MIN()`, `MAX()`等）也可以在窗口函数中使用，为每个窗口计算聚合值。

### d.代码案例

假设有一个销售数据表`sales`，包含字段`department_id`（部门ID）、`employee_id`（员工ID）和`sales_amount`（销售金额）。

1. #### 使用`ROW_NUMBER()`为每个部门的销售记录进行编号

```
SELECT department_id, employee_id, sales_amount,ROW_NUMBER() OVER(PARTITION BY department_id ORDER BY sales_amount DESC) as rankFROM sales;
```

#### 2. 使用`LAG()`查找每个员工相对于前一个员工的销售增长额

```
SELECT department_id, employee_id, sales_amount,sales_amount - LAG(sales_amount, 1, 0) OVER(PARTITION BY department_id ORDER BY employee_id) as growthFROM sales;
```

#### 3. 使用聚合窗口函数计算每个部门的累计销售额

```
SELECT department_id, employee_id, sales_amount,SUM(sales_amount) OVER(PARTITION BY department_id ORDER BY employee_id) as cumulative_salesFROM sales;
```

### e.注意事项

* 使用窗口函数时，OVER子句是必需的，它定义了窗口的分区和排序规则。
* 窗口函数不能直接用在WHERE子句中，因为WHERE子句在结果集生成之前进行过滤，而窗口函数是在结果集生成之后应用的。
* `ORDER BY`在窗口函数中定义排序，`PARTITION BY`用于将数据分成不同的部分，以独立计算每个部分的窗口函数值。
* 考虑到性能，避免在大数据集上使用过于复杂的窗口函数操作，特别是在没有分区的情况下。

**11.分析下hive数据倾斜问题，有什么解决⽅案？**

Hive数据倾斜问题主要源于数据分布不均匀，导致部分Reducer负载过重，影响整体作业的执行效率。以下是分析和解决方案的进一步丰富和细化：

### 原因分析

* Key分布不均匀：部分Key对应的数据量远大于其他Key，导致处理这些Key的Reducer工作量巨大。
* 业务数据特性：某些特定业务逻辑导致数据集中在特定的Key上。
* SQL语句造成数据倾斜：错误的Join或分组条件可能导致大量数据集中到少数Reducer上。

### 解决方案

#### 1. 参数调节

* `hive.map.aggr=true`：在Map阶段进行局部聚合，减少向Reducer传输的数据量。
* `hive.groupby.skewindata=true`：开启这个参数后，Hive会对倾斜的Key进行特殊处理，尝试平衡Reducer的负载。

#### 2. SQL语句调节

* 选择均匀分布的Key进行Join：在进行Join操作时，选择分布较为均匀的字段作为Join Key，减少数据倾斜。
* 列裁剪和过滤：只查询需要的字段，并在可能的情况下通过WHERE子句过滤掉不需要的记录，减少数据量。
* Map Join：对于大表和小表的Join，使用Map Join可以将小表加载到每个Mapper的内存中，减少数据通过网络传输。
* 处理空值和特殊值：对于倾斜严重的特殊值（如空值），可以单独处理或过滤，避免造成Reducer的过载。

#### 3. 数据预处理

* 重分布数据：对倾斜的数据进行预处理，如添加随机前缀或后缀，使得数据更加均匀地分布到Reducer中。
* 分桶（Bucketing）：通过分桶将数据预先均匀分配，可在Join时利用Bucket Map Join优化，减少数据倾斜。

#### 4. 使用高级特性

* Skew Join优化：对于大表Join大表的场景，可以探索使用Hive的Skew Join优化特性，通过设置`hive.optimize.skewjoin`开启。

### 注意事项

* 监控和诊断：使用EXPLAIN命令查看执行计划，识别可能导致数据倾斜的操作。
* 渐进式优化：数据倾斜问题没有一劳永逸的解决方案，需要根据具体情况调整优化策略。
* 资源管理：合理配置Hive作业的资源，如内存和CPU，确保作业在资源充足的情况下运行。

通过综合运用上述策略，可以有效缓解或解决Hive中的数据倾斜问题，提升查询和作业的执行效率。

**12.描述数据中的null,在hive底层如何存储？**

Hive处理空值（null）的方式确实是通过使用特定的字符序列来表示，其中默认的表示null值的字符序列是"\N"（反斜杠加大写的N）。这种表示方式允许Hive在处理文本文件（如CSV或TSV文件）时，能够区分数据中的空值和其他字符串值。在Hive的文本文件存储格式中，任何字段值如果为null，在文件中就会被替换成"\N"。

### 存储和处理null值

* 在文本文件中，null值被存储为字符串"\N"。
* 在二进制格式中（如ORC或Parquet），null值的处理会更为高效。这些格式有专门的机制来表示和存储null值，而不是使用特定的字符串序列。

### Sqoop导出数据时处理null

当使用Sqoop从Hive（或HDFS）导出数据到关系型数据库（如MySQL）时，如果不对null值进行特殊处理，可能会遇到数据类型不匹配的问题。因为"\N"字符串在数据库中不会被自动解释为null值。为了处理这种情况，Sqoop提供了`--null-string`和`--null-non-string`这两个参数，允许用户指定在导出过程中应该如何处理null值：

* `--null-string '\N'`：定义非字符串字段的null值表示方法。
* `--null-non-string '\N'`：定义字符串字段的null值表示方法。

例如，如果希望在导出到MySQL时，将null字符串值转换为MySQL中的NULL，可以在Sqoop命令中这样设置：

```
sqoop export --connect jdbc:mysql://<MySQL-HOST>/<DB> --username <USER> --password <PASSWORD> --table <TABLE> --null-string '\\N' --null-non-string '\\N' ...
```

请注意，对于命令行参数中的转义字符，可能需要根据具体的Shell环境使用适当的转义方法。

### 注意事项

* 理解Hive中null值的表示和存储方式对于数据处理和数据迁移是非常重要的。
* 在设计Hive表和进行数据迁移时（如使用Sqoop导出数据），需要注意如何处理null值，以确保数据的准确性和一致性。
* 不同的文件格式（文本文件、ORC、Parquet等）在存储和处理null值时的效率和方法可能不同，选择合适的存储格式可以优化存储效率和查询性能。

**13.Hive内外部表的区别**

Hive支持两种类型的表：内部表（Managed Table）和外部表（External Table）。这两种表在数据管理和使用场景上有显著的区别：

### a. 数据的所有权

* 内部表：当你创建一个内部表时，Hive对该表中的数据拥有完全的所有权。数据实际存储在Hive的warehouse目录下的一个路径中，这个路径是由Hive控制的。
* 外部表：外部表仅保存数据的元数据，而数据本身存放在HDFS上的任意位置。Hive不拥有这些数据，仅记录数据的存储位置。

### b. 删除表的影响

* 内部表：删除内部表时，Hive会删除表的元数据以及表中存储的数据。这意味着一旦内部表被删除，其对应的数据也会从HDFS上被永久删除。
* 外部表：删除外部表时，Hive仅删除表的元数据，而表中的数据仍然保留在HDFS上的原位置。这是因为Hive认为外部表的数据可能被其他应用或查询所使用。

### c. 使用场景

* 内部表适用于：只在Hive中使用的数据。当数据仅仅是作为临时的分析或处理的一部分时，可以使用内部表。一旦分析完成，数据和表可以一并删除，不会影响其他系统。
* 外部表适用于：需要在多个服务或应用间共享的数据。当数据由外部程序产生并管理，且在Hive之外还要被其他应用访问时，应该使用外部表。

### d. 创建语句的区别

创建内部表的语句：

```
CREATE TABLE internal_table (column1 INT, column2 STRING) STORED AS TEXTFILE;
```

创建外部表的语句：

```
CREATE EXTERNAL TABLE external_table (column1 INT, column2 STRING)LOCATION 'hdfs://path/to/data/' STORED AS TEXTFILE;
```

### 注意事项

* 考虑数据的所有权和使用场景来选择表类型。
* 对于需要长期和跨应用共享的数据，推荐使用外部表。
* 内部表适合临时分析任务，数据处理完成后，表和数据一起删除，便于管理。
* 删除外部表前，需要明确这一操作仅移除元数据，而数据仍然保留在HDFS上。

**14.Hive的权限管理**

Hive的权限管理主要通过几个层面来实现，涉及到数据的访问控制、安全认证和授权。以下是Hive进行权限管理的几种方式：

### a. Hive内置的权限模型

Hive支持基本的权限管理功能，包括对数据库、表、视图等对象的`SELECT`、`INSERT`、`UPDATE`和`DELETE`权限控制。通过`GRANT`和`REVOKE`语句，管理员可以控制用户对特定数据的访问权限。这些操作基于Hive的元数据存储，并在执行查询时进行检查。

### b. 存储级别的权限控制

由于Hive数据实际存储在HDFS上，因此可以利用HDFS的权限系统来进行更底层的访问控制。这包括对数据文件和目录的读写权限设置，可以通过Hadoop的`hadoop fs -chmod`和`hadoop fs -chown`命令来配置。

### c. 集成Kerberos认证

为了提供更强的安全性，Hive支持与Kerberos集成，实现安全的认证机制。Kerberos是一种网络认证协议，通过票据（Ticket）机制来允许节点之间安全地传递身份信息。在启用Kerberos认证的Hadoop集群中，用户和服务都必须通过Kerberos认证后才能访问Hive。这提供了一种强大的防止未授权访问的方法。

### d. 使用Apache Sentry或Apache Ranger进行细粒度的权限控制

对于需要更细粒度权限控制的场景，可以使用Apache Sentry或Apache Ranger这样的第三方安全框架。这些框架提供了基于角色的访问控制（RBAC）、列级别的安全控制、数据掩码和审计等高级安全特性。

* Apache Sentry提供了细粒度的数据访问控制，适用于多租户环境。
* Apache Ranger除了提供细粒度的访问控制外，还提供了安全策略管理、安全审计等功能，并支持多个Hadoop生态系统组件。

### e.注意事项

* 在设计数据安全策略时，需要综合考虑数据存储、传输和访问各个环节的安全需求。
* 定期审计和监控数据访问行为，确保权限设置正确无误，防止数据泄露和未授权访问。
* 在启用Kerberos或集成第三方安全框架时，需要详细规划认证和授权流程，确保与现有的IT安全策略一致。

通过这些方法，Hive能够在大数据环境下提供可靠的权限管理和数据安全保障。