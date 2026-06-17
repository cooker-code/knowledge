---
title: 深度好文！FlinkSQL字段血缘解决方案及源码
author: Apache Hudi
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIyMzQ0NjA0MQ==&mid=2247489623&idx=1&sn=6c44a37b719ac7f78fd93df1270726bd&chksm=e81f4d21df68c437a68d45b34bc0998aee7354235c40b8b22b34907b2f1483e811943c988d10&mpshare=1&scene=24&srcid=0819we3nwZJ2YsR3UciIqIm0&sharer_sharetime=1660900234587&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

## FlinkSQL字段血缘解决方案及源码

| 序号 | 作者 | 版本 | 时间 | 备注 |
| --- | --- | --- | --- | --- |
| 1 | HamaWhite | 1.0.0 | 2022-08-09 | 增加文档和源码 |

作者邮箱: song.bs@dtwave-inc.com

## 一、基础知识

### 1.1 Apache Calcite简介

Apache Calcite是一款开源的动态数据管理框架，它提供了标准的SQL语言、多种查询优化和连接各种数据源的能力，但不包括数据存储、处理数据的算法和存储元数据的存储库。Calcite采用的是业界大数据查询框架的一种通用思路，它的目标是“one size fits all”，希望能为不同计算平台和数据源提供统一的查询引擎。Calcite作为一个强大的SQL计算引擎，在Flink内部的SQL引擎模块也是基于Calcite。

Calcite工作流程如下图所示，一般分为Parser、Validator和Converter、Optimizer阶段。

1.1 Calcite工作流程图.png

详情请参考How to screw SQL to anything with Apache Calcite[1]

### 1.2 Calcite RelNode介绍

在CalciteSQL解析中，Parser解析后生成的SqlNode语法树，经过Validator校验后在Converter阶段会把SqlNode抽象语法树转为关系运算符树(RelNode Tree)，如下图所示。

1.2 Calacite SqlNode vs RelNode.png

### 1.3 组件版本信息

| 组件名称 | 版本 | 备注 |
| --- | --- | --- |
| JDK | 1.8 | scala 2.12 |
| Hadoop | 3.2.2 |  |
| Hive | 3.1.2 |  |
| Flink | 1.14.4 |  |
| Hudi | 0.12.0-SNAPSHOT | 本地源码编译，支持Flink-1.14 |

## 二、字段血缘解析核心思想

### 2.1 FlinkSQL 执行流程解析

根据源码整理出FlinkSQL的执行流程如下图所示，主要分为五个阶段:

1. 1. **Parse阶段**

语法解析，使用JavaCC把SQL转换成抽象语法树(AST)，在Calcite中用SqlNode来表示。

1. 1. **Validate阶段**

语法校验，根据元数据信息进行语法验证，例如查询的表、字段、函数是否存在，会分别对from、where、group by、having、select、orader by等子句进行validate，验证后还是SqlNode构成的语法树AST；

1. 1. **Convert阶段**

语义分析，根据SqlNode和元数据信息构建关系表达式RelNode树，也就是最初版本的逻辑计划。

1. 1. **Optimize阶段**

逻辑计划优化，优化器会基于规则进行等价变换，例如谓词下推、列裁剪等，最终得到最优的查询计划。

1. 1. **Execute阶段**

把逻辑查询计划翻译成物理执行计划，依次生成StreamGraph、JobGraph，最终提交运行。

2.1 FlinkSQL执行流程图.png

> 注1: 图中的Abstract Syntax Tree、Optimized Physical Plan、Optimized Execution Plan、Physical Execution Plan名称来源于StreamPlanner中的explain()方法。  
> 注2: 相比Calcite官方工作流程图，此处把Validate和Convert分为两个阶段。

### 2.2 字段血缘解析思路

2.2 FlinkSQL字段血缘解析思路图.png

FlinkSQL字段血缘解析分为三个阶段:

1. 1. 对输入SQL进行Parse、Validate、Convert，生成关系表达式RelNode树，对应FlinkSQL 执行流程图中的第1、2和3步骤。
2. 2. 在优化阶段，只生成到Optimized Logical Plan即可，而非原本的Optimized Physical Plan。要**修正**FlinkSQL 执行流程图中的第4步骤。

2.2 FlinkSQL字段血缘解析流程图.png

1. 3. 针对上步骤优化生成的逻辑RelNode，调用RelMetadataQuery的getColumnOrigins(RelNode rel, int column)查询原始字段信息。然后构造血缘关系，并返回结果。

   ### 2.3 核心源码阐述

   parseFieldLineage(String sql)方法是对外提供的字段血缘解析API，里面分别执行三大步骤。

```
public List<FieldLineage> parseFieldLineage(String sql) {  
    LOG.info("Input Sql: \n {}", sql);  
    // 1. Generate original relNode tree  
    Tuple2<String, RelNode> parsed = parseStatement(sql);  
    String sinkTable = parsed.getField(0);  
    RelNode oriRelNode = parsed.getField(1);  
    LOG.debug("Original RelNode: \n {}", oriRelNode.explain());  
  
    // 2. Optimize original relNode to generate Optimized Logical Plan  
    RelNode optRelNode = optimize(oriRelNode);  
    LOG.debug("Optimized RelNode: \n {}", optRelNode.explain());  
  
    // 3. Build lineage based from RelMetadataQuery  
    return buildFiledLineageResult(sinkTable, optRelNode);  
}
```

#### 2.3.1 根据SQL生成RelNode树

调用ParserImpl.List parse(String statement) 方法即可，然后返回第一个operation中的calciteTree。此代码限制只支持Insert的血缘关系。

```
private Tuple2<String, RelNode> parseStatement(String sql) {  
    List<Operation> operations = tableEnv.getParser().parse(sql);  
      
    if (operations.size() != 1) {  
        throw new TableException(  
            "Unsupported SQL query! only accepts a single SQL statement.");  
    }  
    Operation operation = operations.get(0);  
    if (operation instanceof CatalogSinkModifyOperation) {  
        CatalogSinkModifyOperation sinkOperation = (CatalogSinkModifyOperation) operation;  
          
        PlannerQueryOperation queryOperation = (PlannerQueryOperation) sinkOperation.getChild();  
        RelNode relNode = queryOperation.getCalciteTree();  
        return new Tuple2<>(  
            sinkOperation.getTableIdentifier().asSummaryString(),  
            relNode);  
    } else {  
        throw new TableException("Only insert is supported now.");  
    }  
}
```

#### 2.3.2 生成Optimized Logical Plan

在第4步骤的逻辑计划优化阶段，根据源码可知核心是调用FlinkStreamProgram的中的优化策略，共包含12个阶段(subquery\_rewrite、temporal\_join\_rewrite...logical\_rewrite、time\_indicator、physical、physical\_rewrite)，优化后生成的是Optimized Pysical Plan。根据SQL的字段血缘解析原理可知，只要解析到logical\_rewrite优化后即可，因此复制FlinkStreamProgram源码为FlinkStreamProgramWithoutPhysical类，并删除time\_indicator、physical、physical\_rewrite策略及最后面chainedProgram.addLast相关代码。然后调用optimize方法核心代码如下:

```
//  this.flinkChainedProgram = FlinkStreamProgramWithoutPhysical.buildProgram(configuration);  
  
/**  
 *  Calling each program's optimize method in sequence.  
 */  
private RelNode optimize(RelNode relNode) {  
    return flinkChainedProgram.optimize(relNode, new StreamOptimizeContext() {  
        @Override  
        public boolean isBatchMode() {  
            return false;  
        }  
  
        @Override  
        public TableConfig getTableConfig() {  
            return tableEnv.getConfig();  
        }  
  
        @Override  
        public FunctionCatalog getFunctionCatalog() {  
            return ((PlannerBase)tableEnv.getPlanner()).getFlinkContext().getFunctionCatalog();  
        }  
  
        @Override  
        public CatalogManager getCatalogManager() {  
            return tableEnv.getCatalogManager();  
        }  
  
        @Override  
        public SqlExprToRexConverterFactory getSqlExprToRexConverterFactory() {  
            return relNode.getCluster().getPlanner().getContext().unwrap(FlinkContext.class).getSqlExprToRexConverterFactory();  
        }  
  
        @Override  
        public <C> C unwrap(Class<C> clazz) {  
            return StreamOptimizeContext.super.unwrap(clazz);  
        }  
  
        @Override  
        public FlinkRelBuilder getFlinkRelBuilder() {  
            return ((PlannerBase)tableEnv.getPlanner()).getRelBuilder();  
        }  
  
        @Override  
        public boolean needFinalTimeIndicatorConversion() {  
            return true;  
        }  
  
        @Override  
        public boolean isUpdateBeforeRequired() {  
            return false;  
        }  
  
        @Override  
        public MiniBatchInterval getMiniBatchInterval() {  
            return MiniBatchInterval.NONE;  
        }  
    });  
}
```

> 注: 此代码可参考StreamCommonSubGraphBasedOptimizer中的optimizeTree方法来书写。

#### 2.3.3 查询原始字段并构造血缘

调用RelMetadataQuery的getColumnOrigins(RelNode rel, int column)查询原始字段信息，然后构造血缘关系，并返回结果。buildFiledLineageResult(String sinkTable, RelNode optRelNode)

```
private List<FieldLineage> buildFiledLineageResult(String sinkTable, RelNode optRelNode) {  
    // target columns  
    List<String> targetColumnList = tableEnv.from(sinkTable)  
            .getResolvedSchema()  
            .getColumnNames();  
  
    RelMetadataQuery metadataQuery = optRelNode.getCluster().getMetadataQuery();  
  
    List<FieldLineage> fieldLineageList = new ArrayList<>();  
  
    for (int index = 0; index < targetColumnList.size(); index++) {  
        String targetColumn = targetColumnList.get(index);  
  
        LOG.debug("**********************************************************");  
        LOG.debug("Target table: {}", sinkTable);  
        LOG.debug("Target column: {}", targetColumn);  
  
        Set<RelColumnOrigin> relColumnOriginSet = metadataQuery.getColumnOrigins(optRelNode, index);  
  
        if (CollectionUtils.isNotEmpty(relColumnOriginSet)) {  
            for (RelColumnOrigin relColumnOrigin : relColumnOriginSet) {  
                // table  
                RelOptTable table = relColumnOrigin.getOriginTable();  
                String sourceTable = String.join(".", table.getQualifiedName());  
  
                // filed  
                int ordinal = relColumnOrigin.getOriginColumnOrdinal();  
                List<String> fieldNames = table.getRowType().getFieldNames();  
                String sourceColumn = fieldNames.get(ordinal);  
                LOG.debug("----------------------------------------------------------");  
                LOG.debug("Source table: {}", sourceTable);  
                LOG.debug("Source column: {}", sourceColumn);  
  
                // add record  
                fieldLineageList.add(buildRecord(sourceTable, sourceColumn, sinkTable, targetColumn));  
            }  
        }  
    }  
    return fieldLineageList;  
}
```

## 三、测试结果

详细测试用例可查看代码中的单测，此处只描述两个测试点。

### 3.1 建表语句

下面新建三张表，分别是: ods\_mysql\_users、dim\_mysql\_company和dwd\_hudi\_users。

#### 3.1.1 新建mysql cdc table-ods\_mysql\_users

```
DROP TABLE IF EXISTS ods_mysql_users;  
  
CREATE TABLE ods_mysql_users(  
  id BIGINT,  
  name STRING,  
  birthday TIMESTAMP(3),  
  ts TIMESTAMP(3),  
  proc_time as proctime()  
) WITH (  
  'connector' = 'mysql-cdc',  
  'hostname' = '192.168.90.xxx',  
  'port' = '3306',  
  'username' = 'root',  
  'password' = 'xxx',  
  'server-time-zone' = 'Asia/Shanghai',  
  'database-name' = 'demo',  
  'table-name' = 'users'  
);
```

#### 3.1.2 新建mysql dim table-dim\_mysql\_company

```
DROP TABLE IF EXISTS dim_mysql_company;  
  
CREATE TABLE dim_mysql_company (  
    user_id BIGINT,   
    company_name STRING  
) WITH (  
    'connector' = 'jdbc',  
    'url' = 'jdbc:mysql://192.168.90.xxx:3306/demo?useSSL=false&characterEncoding=UTF-8',  
    'username' = 'root',  
    'password' = 'xxx',  
    'table-name' = 'company'  
);
```

#### 3.1.3 新建hudi sink table-dwd\_hudi\_users

```
DROP TABLE IF EXISTS dwd_hudi_users;  
  
CREATE TABLE dwd_hudi_users (  
    id BIGINT,  
    name STRING,  
    company_name STRING,  
    birthday TIMESTAMP(3),  
    ts TIMESTAMP(3),  
    `partition` VARCHAR(20)  
) PARTITIONED BY (`partition`) WITH (  
    'connector' = 'hudi',  
    'table.type' = 'COPY_ON_WRITE',  
    'path' = 'hdfs://192.168.90.xxx:9000/hudi/dwd_hudi_users',  
    'read.streaming.enabled' = 'true',  
    'read.streaming.check-interval' = '1'  
);
```

### 3.2 测试SQL及血缘结果

#### 3.2.1 测试insert-select

* • 测试SQL

```
INSERT INTO  
    dwd_hudi_users  
SELECT  
    id,  
    name,  
    name as company_name,  
    birthday,  
    ts,  
    DATE_FORMAT(birthday, 'yyyyMMdd')  
FROM  
    ods_mysql_users
```

* • 测试结果

| **sourceTable** | **sourceColumn** | **targetTable** | **targetColumn** |
| --- | --- | --- | --- |
| ods\_mysql\_users | id | dwd\_hudi\_users | id |
| ods\_mysql\_users | name | dwd\_hudi\_users | name |
| ods\_mysql\_users | name | dwd\_hudi\_users | company\_name |
| ods\_mysql\_users | birthday | dwd\_hudi\_users | birthday |
| ods\_mysql\_users | ts | dwd\_hudi\_users | ts |
| ods\_mysql\_users | birthday | dwd\_hudi\_users | partition |

#### 3.2.2 测试insert-select-join

* • 测试SQL

```
SELECT  
    a.id as id1,  
    CONCAT(a.name, b.company_name),  
    b.company_name,  
    a.birthday,  
    a.ts,  
    DATE_FORMAT(a.birthday, 'yyyyMMdd') as p  
FROM  
    ods_mysql_users as a  
JOIN   
    dim_mysql_company as b  
ON a.id = b.user_id
```

* • RelNode树展示Original RelNode

```
 LogicalProject(id1=[$0], EXPR$1=[CONCAT($1, $6)], company_name=[$6], birthday=[$2], ts=[$3], p=[DATE_FORMAT($2, _UTF-16LE'yyyyMMdd')])  
  LogicalJoin(condition=[=($0, $5)], joinType=[inner])  
    LogicalProject(id=[$0], name=[$1], birthday=[$2], ts=[$3], proc_time=[PROCTIME()])  
      LogicalTableScan(table=[[hive, flink_demo, ods_mysql_users]])  
    LogicalTableScan(table=[[hive, flink_demo, dim_mysql_company]])
```

经过optimize(RelNode relNode)优化后的Optimized RelNode结果如下:

```
 FlinkLogicalCalc(select=[id AS id1, CONCAT(name, company_name) AS EXPR$1, company_name, birthday, ts, DATE_FORMAT(birthday, _UTF-16LE'yyyyMMdd') AS p])  
  FlinkLogicalJoin(condition=[=($0, $4)], joinType=[inner])  
    FlinkLogicalTableSourceScan(table=[[hive, flink_demo, ods_mysql_users]], fields=[id, name, birthday, ts])  
    FlinkLogicalTableSourceScan(table=[[hive, flink_demo, dim_mysql_company]], fields=[user_id, company_name])
```

* • 测试结果

| **sourceTable** | **sourceColumn** | **targetTable** | **targetColumn** |
| --- | --- | --- | --- |
| ods\_mysql\_users | id | dwd\_hudi\_users | id |
| dim\_mysql\_company | company\_name | dwd\_hudi\_users | name |
| ods\_mysql\_users | name | dwd\_hudi\_users | name |
| dim\_mysql\_company | company\_name | dwd\_hudi\_users | company\_name |
| ods\_mysql\_users | birthday | dwd\_hudi\_users | birthday |
| ods\_mysql\_users | ts | dwd\_hudi\_users | ts |
| ods\_mysql\_users | birthday | dwd\_hudi\_users | partition |

#### 3.2.3 测试insert-select-lookup-join

上述步骤完成后还不支持Lookup Join的字段血缘解析，测试情况如下所述。

* • 测试SQL

```
SELECT  
    a.id as id1,  
    CONCAT(a.name, b.company_name),  
    b.company_name,  
    a.birthday,  
    a.ts,  
    DATE_FORMAT(a.birthday, 'yyyyMMdd') as p  
FROM  
    ods_mysql_users as a  
JOIN   
    dim_mysql_company FOR SYSTEM_TIME AS OF a.proc_time AS b  
ON a.id = b.user_id
```

* • 测试结果

| **sourceTable** | **sourceColumn** | **targetTable** | **targetColumn** |
| --- | --- | --- | --- |
| ods\_mysql\_users | id | dwd\_hudi\_users | id |
| ods\_mysql\_users | name | dwd\_hudi\_users | name |
| ods\_mysql\_users | birthday | dwd\_hudi\_users | birthday |
| ods\_mysql\_users | ts | dwd\_hudi\_users | ts |
| ods\_mysql\_users | birthday | dwd\_hudi\_users | partition |

可以看到，维表dim\_mysql\_company的字段血缘关系都被丢失掉，因此继续进行下面的步骤。

## 四、修改Calcite源码支持Lookup Join

### 4.1 实现思路

针对Lookup Join，Parser会把SQL语句“FOR SYSTEM\_TIME AS OF ”解析成 SqlSnapshot ( SqlNode)，validate() 将其转换成 LogicalSnapshot(RelNode)。

Lookup Join-Original RelNode

```
 LogicalProject(id1=[$0], EXPR$1=[CONCAT($1, $6)], company_name=[$6], birthday=[$2], ts=[$3], p=[DATE_FORMAT($2, _UTF-16LE'yyyyMMdd')])  
  LogicalCorrelate(correlation=[$cor0], joinType=[inner], requiredColumns=[{0, 4}])  
    LogicalProject(id=[$0], name=[$1], birthday=[$2], ts=[$3], proc_time=[PROCTIME()])  
      LogicalTableScan(table=[[hive, flink_demo, ods_mysql_users]])  
    LogicalFilter(condition=[=($cor0.id, $0)])  
      LogicalSnapshot(period=[$cor0.proc_time])  
        LogicalTableScan(table=[[hive, flink_demo, dim_mysql_company]])
```

但calcite-core中RelMdColumnOrigins这个Handler类里并没有处理Snapshot类型的RelNode，导致返回空，继而丢失Lookup Join字段的血缘关系。因此，需要在RelMdColumnOrigins增加一个处理Snapshot的getColumnOrigins(Snapshot rel,RelMetadataQuery mq, int iOutputColumn)方法。

由于flink-table-planner是采用maven-shade-plugin打包的，因此修改calcite-core后要重新打flink包。flink-table/flink-table-planner/pom.xml。

```
<plugin>  
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-shade-plugin</artifactId>  
  ...  
    <artifactSet>  
      <includes combine.children="append">  
        <include>org.apache.calcite:*</include>  
        <include>org.apache.calcite.avatica:*</include>  
  ...             
```

本文在下面的4.2-4.4小节给出基础性操作步骤，分别讲述如何修改calcite、flink源码，以及如何编译、打包。

同时在4.5小节也提供另外一种实现路径，即通过动态编辑Java字节码技术来增加getColumnOrigins方法，源码已默认采用此技术，读者也可直接跳到4.5小节进行阅读。

### 4.2 重新编译Calcite源码

#### 4.2.1 下载源码及创建分支

flink1.14.4依赖的calcite版本是1.26.0，因此基于tag calcite-1.26.0来修改源码。并且在原有3位版本号后面再增加一位版本号，以区别于官方发布的版本。

```
# 下载github上源码  
$ git clone git@github.com:apache/calcite.git  
  
# 切换到 calcite-1.26.0 tag  
$ git checkout calcite-1.26.0  
  
# 新建分支calcite-1.26.0.1  
$ git checkout -b calcite-1.26.0.1
```

#### 4.2.2 修改源码

1. 1. 在calcite-core模块，给RelMdColumnOrigins类增加方法 getColumnOrigins(Snapshot rel,RelMetadataQuery mq, int iOutputColumn)。org.apache.calcite.rel.metadata.RelMdColumnOrigins

```
public Set<RelColumnOrigin> getColumnOrigins(Snapshot rel,  
        RelMetadataQuery mq, int iOutputColumn) {  
    return mq.getColumnOrigins(rel.getInput(), iOutputColumn);  
}
```

1. 1. 修改版本号为 1.26.0.1，calcite/gradle.properties`# 修改前  
   calcite.version=1.26.0  
   # 修改后  
   calcite.version=1.26.0.1`
2. 2. 删除打包名称上的SNAPSHOT，由于未研究出Gradlew 打包参数，此处直接修改build.gradle.kts代码。calcite/build.gradle.kts

```
# 修改前  
val buildVersion = "calcite".v + releaseParams.snapsnapshotSuffixshotSuffix  
  
#修改后  
val buildVersion = "calcite".v
```

#### 4.2.3 编译源码和推送到本地仓库

```
# 编译源码  
$ ./gradlew build -x test   
  
# 推送到本地仓库  
$ ./gradlew publishToMavenLocal
```

运行成功后查看本地maven仓库，已经产生calcite-core-1.26.0.1.jar。

```
$ ll ~/.m2/repository/org/apache/calcite/calcite-core/1.26.0.1  
  
-rw-r--r--  1 baisong  staff  8893065  8  9 13:51 calcite-core-1.26.0.1-javadoc.jar  
-rw-r--r--  1 baisong  staff  3386193  8  9 13:51 calcite-core-1.26.0.1-sources.jar  
-rw-r--r--  1 baisong  staff  2824504  8  9 13:51 calcite-core-1.26.0.1-tests.jar  
-rw-r--r--  1 baisong  staff  5813238  8  9 13:51 calcite-core-1.26.0.1.jar  
-rw-r--r--  1 baisong  staff     5416  8  9 13:51 calcite-core-1.26.0.1.pom
```

### 4.3 重新编译Flink源码

#### 4.2.1 下载源码及创建分支

基于tag calcite-1.26.0来修改源码。并且在原有3位版本号后面再增加一位版本号，以区别于官方发布的版本。

```
# 下载github上flink源码  
$ git clone git@github.com:apache/flink.git  
  
# 切换到 release-1.14.4 tag  
$ git checkout release-1.14.4  
  
# 新建分支release-1.14.4.1  
$ git checkout -b release-1.14.4.1
```

#### 4.3.2 修改源码

1. 1. 在flink-table模块，修改calcite.version的版本为 1.26.0.1，flink-table-planner会引用此版本号。即让flink-table-planner引用calcite-core-1.26.0.1。flink/flink-table/pom.xml。

```
<properties>  
    <!-- When updating Janino, make sure that Calcite supports it as well. -->  
    <janino.version>3.0.11</janino.version>  
    <!--<calcite.version>1.26.0</calcite.version>-->  
    <calcite.version>1.26.0.1</calcite.version>  
    <guava.version>29.0-jre</guava.version>  
</properties>
```

1. 1. 修改flink-table-planner版本号为1.14.4.1，包含下面3点。flink/flink-table/flink-table-planner/pom.xml。

```
<artifactId>flink-table-planner_${scala.binary.version}</artifactId>  
<!--1.新增此行-->  
<version>1.14.4.1</version>  
<name>Flink : Table : Planner</name>  
  
<!--2. 全局替换${project.version}为${parent.version}-->  
  
<!--3.新增加此依赖，强制指定flink-test-utils-junit版本，否则编译会报错-->  
<dependency>  
    <artifactId>flink-test-utils-junit</artifactId>  
    <groupId>org.apache.flink</groupId>  
    <version>${parent.version}</version>  
    <scope>test</scope>  
</dependency>
```

#### 4.3.3 编译源码和推送到远程仓库

```
# 只编译 flink-table-planner  
$ mvn clean install -pl flink-table/flink-table-planner -am -Dscala-2.12 -DskipTests -Dfast -Drat.skip=true -Dcheckstyle.skip=true -Pskip-webui-build
```

如果要推送到Maven仓库，修改pom.xml 增加仓库地址(以数澜科技仓库为例)。

```
<distributionManagement>  
    <repository>  
        <id>releases</id>  
        <url>http://xxx.xxx-inc.com/repository/maven-releases</url>  
    </repository>  
    <snapshotRepository>  
        <id>snapshots</id>  
        <url>http://xxx.xxx-inc.com/repository/maven-snapshots</url>  
    </snapshotRepository>  
</distributionManagement>
```

```
# 进入flink-table-planner模块  
$ cd flink-table/flink-table-planner  
  
# 推送到到远程仓库  
$ mvn clean deploy -Dscala-2.12 -DskipTests -Dfast -Drat.skip=true -Dcheckstyle.skip=true -Pskip-webui-build -T 1C
```

### 4.4 修改Flink依赖版本并测试Lookup Join

修改pom.xml中依赖的flink-table-planner的版本为1.14.4.1。

```
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-table-planner_2.12</artifactId>  
    <version>1.14.4.1</version>  
</dependency>
```

执行第3.2.3章节的测试用例得到Lookup Join血缘结果如下，已经包含维表dim\_mysql\_company的字段血缘关系。

| **sourceTable** | **sourceColumn** | **targetTable** | **targetColumn** |
| --- | --- | --- | --- |
| ods\_mysql\_users | id | dwd\_hudi\_users | id |
| dim\_mysql\_company | company\_name | dwd\_hudi\_users | name |
| ods\_mysql\_users | name | dwd\_hudi\_users | name |
| dim\_mysql\_company | company\_name | dwd\_hudi\_users | company\_name |
| ods\_mysql\_users | birthday | dwd\_hudi\_users | birthday |
| ods\_mysql\_users | ts | dwd\_hudi\_users | ts |
| ods\_mysql\_users | birthday | dwd\_hudi\_users | partition |

### 4.5 动态编辑Java字节码增加getColumnOrigins方法

Javassist是可以动态编辑Java字节码的类库，它可以在Java程序运行时定义一个新的类并加载到JVM中，还可以在JVM加载时修改一个类文件。因此，本文通过Javassist技术来动态给RelMdColumnOrigins类增加getColumnOrigins(Snapshot rel,RelMetadataQuery mq, int iOutputColumn)方法。

核心代码如下:

```
/**  
 * Dynamic add getColumnOrigins method to class RelMdColumnOrigins by javassist:  
 *  
 * public Set<RelColumnOrigin> getColumnOrigins(Snapshot rel,RelMetadataQuery mq, int iOutputColumn) {  
 *      return mq.getColumnOrigins(rel.getInput(), iOutputColumn);  
 * }  
 */  
static {  
    try {  
        ClassPool classPool = ClassPool.getDefault();  
        CtClass ctClass = classPool.getCtClass("org.apache.calcite.rel.metadata.RelMdColumnOrigins");  
  
        CtClass[] parameters = new CtClass[]{classPool.get(Snapshot.class.getName())  
                , classPool.get(RelMetadataQuery.class.getName())  
                , CtClass.intType  
        };  
        // add method  
        CtMethod ctMethod = new CtMethod(classPool.get("java.util.Set"), "getColumnOrigins", parameters, ctClass);  
        ctMethod.setModifiers(Modifier.PUBLIC);  
        ctMethod.setBody("{return $2.getColumnOrigins($1.getInput(), $3);}");  
        ctClass.addMethod(ctMethod);  
        // load the class  
        ctClass.toClass();  
    } catch (Exception e) {  
        throw new TableException("Dynamic add getColumnOrigins() method exception.", e);  
    }  
}
```

> 注1: 也可把RelMdColumnOrigins类及package拷贝到项目中，然后手动增加getColumnOrigins方法。但是此方法兼容性不够友好，后续calcite源码进行迭代后血缘代码要跟随calcite一起修正。

上述代码增加后，执行Lookup Join的测试用例后就能看到维表dim\_mysql\_company的字段血缘关系，如4.4节的表格所示。

## 五、参考文献

1. 1. How to screw SQL to anything with Apache Calcite[2]
2. 2. 使用build.gradle.kts发布到mavenLocal[3]
3. 3. Flink SQL LookupJoin终极解决方案及Flink Rule入门[4]
4. 4. 基于Calcite解析Flink SQL列级数据血缘[5]
5. 5. 干货|详解FlinkSQL实现原理[6]

#### 引用链接

`[1]` How to screw SQL to anything with Apache Calcite: *https://zephyrnet.com/how-to-screw-sql-to-anything-with-apache-calcite/*  
`[2]` How to screw SQL to anything with Apache Calcite: *https://zephyrnet.com/how-to-screw-sql-to-anything-with-apache-calcite/*  
`[3]` 使用build.gradle.kts发布到mavenLocal: *https://www.javaroad.cn/questions/71299*  
`[4]` Flink SQL LookupJoin终极解决方案及Flink Rule入门: *https://zhuanlan.zhihu.com/p/546080679*  
`[5]` 基于Calcite解析Flink SQL列级数据血缘: *https://blog.csdn.net/nazeniwaresakini/article/details/121652104*  
`[6]` 干货|详解FlinkSQL实现原理: *https://www.modb.pro/db/133495*