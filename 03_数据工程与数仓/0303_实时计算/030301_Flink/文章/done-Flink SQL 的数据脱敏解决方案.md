> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkSQL字段血缘|FlinkSQL字段血缘]]
---
title: Flink SQL 的数据脱敏解决方案
author: Apache Flink
date:
url: http://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247505840&idx=1&sn=fcd3010e92f8653c87aa1d8a4e36bd05&chksm=fd3859f2ca4fd0e4055660cc32da2751d784f44f5d09b125c0644c82ed6723c2d5d3bc9165bf&mpshare=1&scene=24&srcid=0511YtCZpgwUlF5fbJEYXaW0&sharer_sharetime=1683802896966&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

Flink SQL 的数据脱敏解决方案，支持面向用户级别的数据脱敏访问控制，即特定用户只能访问到脱敏后的数据。此方案是实时领域Flink的解决思路，类似于离线数仓 Hive 中 Ranger Column Masking 方案。

**01**

**基础知识**

##

### **1.1 数据脱敏**

数据脱敏(Data Masking)是一种数据安全技术，用于保护敏感数据，以防止未经授权的访问。该技术通过将敏感数据替换为虚假数据或不可识别的数据来实现。例如可以使用数据脱敏技术将信用卡号码、社会安全号码等敏感信息替换为随机生成的数字或字母，以保护这些信息的隐私和安全。

### **1.2 业务流程**

下面用订单表 orders 的两行数据来举例，示例数据如下:

#### **■ 1.2.1 设置脱敏策略**

管理员配置用户、表、字段、脱敏条件，例如下面的配置。

#### **■****1.2.2 用户访问数据**

当用户在Flink上查询 orders 表的数据时，会在底层结合该用户的脱敏条件重新生成 SQL，即让数据脱敏生效。当用户 A 和用户 B 在执行下面相同的 SQL 时，会看到不同的结果数据。

```
```
SELECT * FROM orders
```
```

用户 A 查看到的结果数据如下，customer\_name 字段的数据被全部掩盖掉。

用户 B 查看到的结果数据如下，customer\_name 字段的数据只会显示前 4 位，剩下的用 x 代替。

**02**

**Hive 数据脱敏解决方案**

##

在离线数仓工具 Hive 领域，由于发展多年已有 Ranger 来支持字段数据的脱敏控制，详见参考文献 [1]。下图是在 Ranger 里配置 Hive 表数据脱敏条件的页面，供参考。

但由于 Flink 实时数仓领域发展相对较短，Ranger 还不支持 Flink SQL，以及依赖 Ranger 的话会导致系统部署和运维过重，因此开始**自研实时数仓的数据脱敏解决工具**。当然本文中的核心思想也适用于 Ranger 中，可以基于此较快开发出 ranger-flink 插件。

**03**

**Flink SQL 数据脱敏解决方案**

##

### **3.1 解决方案**

#### **■ 3.1.1 Flink SQL 执行流程**

可以参考作者文章 Flink SQL 字段血缘解决方案及源码 [2]，本文根据 Flink 1.16 修正和简化后的执行流程如下图所示。

在 CalciteParser.parse() 处理后会得到一个 SqlNode 类型的抽象语法树，本文会针对此抽象语法树来组装脱敏条件后来生成新的 AST，以实现数据脱敏控制。

#### **■ 3.1.2 Calcite 对象继承关系**

下面章节要用到 Calcite 中的 SqlNode、SqlCall、SqlIdentifier、SqlJoin、SqlBasicCall 和 SqlSelect 等类，此处进行简单介绍以及展示它们间继承关系，以便读者阅读本文源码。

| 序号 | 类 | 介绍 |
| --- | --- | --- |
| 1 | SqlNode | A SqlNode is a SQL parse tree. |
| 2 | SqlCall | A SqlCall is a call to an SqlOperator operator. |
| 3 | SqlIdentifier | A SqlIdentifier is an identifier, possibly compound. |
| 4 | SqlJoin | Parse tree node representing a JOIN clause. |
| 5 | SqlBasicCall | Implementation of SqlCall that keeps its operands in an array. |
| 6 | SqlSelect | A SqlSelect is a node of a parse tree which represents a select statement, the parent class is SqlCall |

#### **■ 3.1.3 解决思路**

针对输入的 Flink SQL，在 CalciteParser.parse() 进行语法解析后生成抽象语法树(Abstract Syntax Tree，简称 AST)后，采用自定义 Calcite SqlBasicVisitor 的方法遍历 AST 中的所有 SqlSelect，获取到里面的每个输入表。如果输入表中字段有配置脱敏条件，则针对输入表生成子查询语句，并把脱敏字段改写成 CAST(脱敏函数(字段名) AS 字段类型) AS 字段名,再通过 CalciteParser.parseExpression() 把子查询转换成 SqlSelect，并用此 SqlSelect 替换原 AST 中的输入表来生成新的 AST，最后得到新的 SQL 来继续执行。

### **3.2 详细方案**

#### **■****3.2.1 解析输入表**

通过对 Flink SQL 语法的分析和研究，最终出现输入表的只包含以下两种情况:

1. SELECT 语句的 FROM 子句，如果是子查询，则递归继续遍历。

2. SELECT ... JOIN 语句的 Left 和 Right 子句，如果是多表 JOIN，则递归查询遍历。

因此，下面的主要步骤会根据 FROM 子句的类型来寻找输入表。

#### **■****3.2.2 主要步骤**

####

主要通过 Calcite 提供的访问者模式自定义 DataMaskVisitor 来实现，遍历 AST 中所有的 SqlSelect 对象用子查询替换里面的输入表。

下面详细描述替换输入表的步骤，整体流程如下图所示。

1. 遍历 AST 中 SELECT 语句。

2. 判断是否自定义的 SELECT 语句(由下面步骤 10 生成)，是则跳转到步骤 11，否则继续步骤 3。

3. 判断 SELECT 语句中的 FROM 类型，按照不同类型对应执行下面的步骤 4、5、6 和 11。

4. 如果 FROM 是 SqlJoin 类型，则分别遍历其左 Left 和 Right 右节点，即执行当前步骤 4 和步骤 7。由于可能是三张表及以上的 Join，因此进行递归处理，即针对其左 Left 节点跳回到步骤 3。

5. 如果 FROM 是 SqlIdentifier 类型，则表示是表。但是输入 SQL 中没有定义表的别名，则用表名作为别名。跳转到步骤 8。

6. 如果 FROM 是 SqlBasicCall 类型，则表示带别名。但需要判断是否来自子查询，是则跳转到步骤 11 继续遍历AST，后续步骤 1 会对子查询中的 SELECT 语句进行处理。否则跳转到步骤 8。

7. 递归处理 Join 的右节点，即跳回到步骤3。

8. 遍历表中的每个字段，如果某个字段有定义脱敏条件，则把改字段改写成格式 CAST(脱敏函数(字段名) AS 字段类型) AS 字段名，否则用原字段名。

9. 针对步骤 8 处理后的字段，构建子查询语句，形如 (SELECT 字段名1, 字段名2, CAST(脱敏函数(字段名3) AS 字段类型) AS 字段名3、字段名4 FROM 表名) AS 表别名。

10. 对步骤 9 的子查询调用 CalciteParser.parseExpression()进行解析，生成自定义 SELECT 语句，并替换掉原 FROM。

11. 继续遍历 AST，找到里面的 SELECT 语句进行处理，跳回到步骤 1。

#### **■ 3.2.3 Hive 及 Ranger 兼容性**

在 Ranger 中，默认的脱敏策略的如下所示。通过调研发现 Ranger 的大部分脱敏策略是通过调用 Hive 自带或自定义的系统函数实现的。

| 序号 | 策略名 | 策略说明 | Hive系统函数 |
| --- | --- | --- | --- |
| 1 | Redact | 用x屏蔽字母字符，用n屏蔽数字字符 | mask |
| 2 | Partial mask: show last 4 | 仅显示最后四个字符,其他用x代替 | mask\_show\_last\_n |
| 3 | Partial mask: show first 4 | 仅显示前四个字符,其他用x代替 | mask\_show\_first\_n |
| 4 | Hash | 用值的哈希值替换原值 | mask\_hash |
| 5 | Nullify | 用NULL值替换原值 | Ranger自身实现 |
| 6 | Unmasked | 原样显示 | Ranger自身实现 |
| 7 | Date: show only year | 仅显示日期字符串的年份 | mask |
| 8 | Custom | Hive UDF来自定义策略 |  |

由于 Flink 支持 Hive Catalog，在 Flink 能调用 Hive 系统函数。因此，本方案也支持在 Flink SQL 配置 Ranger 的脱敏策略。

**04**

**用例测试**

##

用例测试数据来自于 CDC Connectors for Apache Flink [4]官网，本文给 orders 表增加一个 region 字段，同时增加'connector'='print'类型的 print\_sink 表，其字段和 orders 表的相同。

可通过 Maven 运行单元测试。

```
$ cd flink-sql-security$ mvn test
```

详细测试用例可查看源码中的单测 RewriteDataMaskTest 和 ExecuteDataMaskTest，下面只描述两个案例。

### **4.1 测试 SELECT**

#### **■****4.1.1 输入 SQL**

用户 A 执行下述 SQL:

```
```
SELECT order_id, customer_name, product_id, region FROM orders
```
```

#### **■****4.1.2 根据脱敏条件重新生成SQL**

1. 输入 SQL 是一个简单 SELECT 语句，其 FROM 类型是 SqlIdentifier，由于没有定义别名，用表名 orders 作为别名。

2. 由于用户A针对字段 customer\_name 定义脱敏条件 MASK(对应函数是脱敏函数是 mask)，该字段在流程图中的步骤 8 中被改写为 CAST(mask(customer\_name) AS STRING) AS customer\_name，其余字段未定义脱敏条件则保持不变。

3. 然后在步骤 9 的操作中，表名 orders 被改写成如下子查询，子查询两侧用括号()进行包裹，并且用 AS 别名来增加表别名。

```
```
(SELECT     order_id,     order_date,     CAST(mask(customer_name) AS STRING) AS customer_name,     product_id,     price,     order_status,     regionFROM     orders) AS orders
```
```

#### **■****4.1.3 输出 SQL 和运行结果**

最终执行的改写后SQL如下所示，这样用户A查询到的顾客姓名 customer\_name 字段都是掩盖后的数据。

```
```
SELECT    order_id,    customer_name,    product_id,    regionFROM (    SELECT          order_id,         order_date,         CAST(mask(customer_name) AS STRING) AS customer_name,         product_id,         price,         order_status,         region    FROM          orders     ) AS orders
```
```

### **4.2 测试 INSERT-SELECT**

#### **■****4.2.1 输入 SQL**

用户 A 执行下述 SQL:

```
```
INSERT INTO print_sink SELECT * FROM orders
```
```

#### **■****4.2.2 根据脱敏条件重新生成 SQL**

通过自定义 Calcite DataMaskVisitor 访问生成的 AST，能找到对应的 SELECT 语句是 SELECT order\_id, customer\_name, product\_id, region FROM orders。

针对此 SELECT 语句的改写逻辑同上，不再阐述。

#### **■****4.2.3 输出 SQL 和运行结果**

最终执行的改写后 SQL 如下所示，注意插入到 print\_sink 表的 customer\_name 字段是掩盖后的数据。

```
```
INSERT INTO print_sink (    SELECT         *     FROM (        SELECT             order_id,             order_date,             CAST(mask(customer_name) AS STRING) AS customer_name,             product_id,             price,             order_status,             region         FROM             orders    ) AS orders)
```
```

##

## **参考文献**

[1] Apache Ranger Column Masking in Hive:

https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.0/authorization-ranger/content/dynamic\_resource\_based\_column\_masking\_in\_hive\_with\_ranger\_policies.html

[2] Flink SQL 字段血缘解决方案及源码:

https://github.com/HamaWhiteGG/flink-sql-lineage/blob/main/README\_CN.md

[3] 从 SQL 语句中解析出源表和结果表:

https://blog.jrwang.me/2018/parse-table-in-sql

[4] 基于 Flink CDC 构建 MySQL 和 Postgres 的 Streaming ETL:

https://ververica.github.io/flink-cdc-connectors/master/content/%E5%BF%AB%E9%80%9F%E4%B8%8A%E6%89%8B/mysql-postgres-tutorial-zh.html

[5] HiveQL—数据脱敏函数:

https://blog.csdn.net/CPP\_MAYIBO/article/details/104065839

往期精选

---

▼ 精彩直播回顾 ▼

▼ 关注「**Apache Flink**」，获取更多技术干货 ▼

   **点击「阅****读原文****」，查看更多技术内容**