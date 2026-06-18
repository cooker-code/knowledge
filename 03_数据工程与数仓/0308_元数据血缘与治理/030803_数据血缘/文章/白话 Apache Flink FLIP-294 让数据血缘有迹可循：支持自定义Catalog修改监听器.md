---
title: 白话 Apache Flink FLIP-294 让数据血缘有迹可循：支持自定义Catalog修改监听器
author: 循技漫聊
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg2NTU4NzU4Mw==&mid=2247484815&idx=2&sn=8992f96055e444575adb5d570ab61e23&chksm=cfbeaa642bfd9ac4ff1c4f90f652cf25b020940ffe948ebf6cde35ef5fa91f83c0e0fde369f9&mpshare=1&scene=24&srcid=101335UGz3qlXx6oWmqWRY5G&sharer_shareinfo=27e94c74363454f0d33e6887d497b76f&sharer_shareinfo_first=27e94c74363454f0d33e6887d497b76f#rd
---

## 开篇

想象一下，你是一家大型电商公司的数据工程师。每天有成百上千的数据表被创建、修改、删除，数据在各个系统之间流转，就像一张复杂的血缘关系网。突然有一天，老板问你："我们的订单数据是从哪里来的？经过了哪些处理？最终流向了哪些下游系统？"

如果没有一个完善的数据血缘追踪机制，这个问题可能会让你抓狂。今天我们要聊的 FLIP-294，就是为了解决这个痛点 —— 它让 Flink 能够监听目录的修改操作，帮助你自动收集和上报数据血缘信息。

## 现状与解决

### 数据血缘追踪的困境

在现有的 Flink 中，当你执行 DDL 操作（比如创建表、修改表结构、删除表）时，这些操作就像"黑盒"一样，外部系统无法感知到这些变化。这带来了几个问题：

1. 1. **血缘关系缺失**：你无法知道数据是从哪里来的，要到哪里去
2. 2. **手动维护成本高**：需要人工去各个系统收集血缘信息
3. 3. **实时性差**：当表结构发生变化时，下游系统可能不知道
4. 4. **合规风险**：在数据治理要求越来越严格的今天，缺乏完整的数据血缘会带来合规风险

### 核心思路

FLIP-294 引入了 **目录修改监听器（CatalogModificationListener）** 的概念。简单来说，就是在 Flink 执行 DDL 操作时，会触发相应的事件，外部系统可以通过监听这些事件来获取表的元数据信息。

### 主要组件

#### 1. CatalogModificationListener 接口

这是核心的监听器接口：

```
@PublicEvolving  
public interface CatalogModificationListener {  
    /** 当数据库/表被修改时触发的事件 */  
    void onEvent(CatalogModificationEvent event);  
}
```

#### 2. 事件体系

FLIP-294 定义了一系列事件类型：

**基础事件**：

* • `CatalogModificationEvent`：所有事件的基础接口

**数据库相关事件**：

* • `CreateDatabaseEvent`：创建数据库事件
* • `AlterDatabaseEvent`：修改数据库事件
* • `DropDatabaseEvent`：删除数据库事件

**表相关事件**：

* • `CreateTableEvent`：创建表事件
* • `AlterTableEvent`：修改表事件
* • `DropTableEvent`：删除表事件

#### 3. 上下文信息

每个事件都包含丰富的上下文信息：

```
@PublicEvolving  
publicinterfaceCatalogContext {  
    /** 目录名称 */  
    String getCatalogName();  
      
    /** 目录类别 */  
    Class<? extendsCatalog> getClass();  
      
    /** 目录工厂标识符，如 jdbc/iceberg/paimon */  
    Optional<String> getFactoryIdentifier();  
      
    /** 目录配置 */  
    Configuration getConfiguration();  
}
```

## 实际应用场景

### 场景 1：数据血缘自动上报

假设你想将数据血缘信息自动上报到 DataHub，可以这样实现：

```
public classDataHubCatalogListenerimplementsCatalogModificationListener {  
      
    @Override  
    publicvoidonEvent(CatalogModificationEvent event) {  
        if (event instanceof CreateTableEvent) {  
            CreateTableEventtableEvent= (CreateTableEvent) event;  
              
            // 获取表信息  
            ObjectIdentifiertableId= tableEvent.identifier();  
            CatalogBaseTabletable= tableEvent.table();  
              
            // 构建数据集信息  
            Datasetdataset= buildDataset(tableId, table, event.context());  
              
            // 异步上报到 DataHub  
            dataHubClient.ingestDatasetAsync(dataset);  
        }  
    }  
      
    private Dataset buildDataset(ObjectIdentifier tableId,   
                               CatalogBaseTable table,   
                               CatalogContext context) {  
        // 根据连接器类型和配置构建物理存储标识  
        Stringconnector= getConnectorType(table, context);  
        StringphysicalIdentifier= buildPhysicalIdentifier(connector, table);  
          
        return Dataset.builder()  
            .urn(physicalIdentifier)  
            .name(tableId.getObjectName())  
            .schema(convertSchema(table.getSchema()))  
            .build();  
    }  
}
```

### 场景 2：表变更通知

当表结构发生变化时，自动通知下游系统：

```
public classSchemaChangeNotifierimplementsCatalogModificationListener {  
      
    @Override  
    publicvoidonEvent(CatalogModificationEvent event) {  
        if (event instanceof AlterTableEvent) {  
            AlterTableEventalterEvent= (AlterTableEvent) event;  
              
            // 分析表结构变化  
            List<TableChange> changes = alterEvent.tableChanges();  
              
            for (TableChange change : changes) {  
                if (change instanceof TableChange.AddColumn) {  
                    notifyColumnAdded(alterEvent.identifier(),   
                                    (TableChange.AddColumn) change);  
                } elseif (change instanceof TableChange.DropColumn) {  
                    notifyColumnDropped(alterEvent.identifier(),   
                                      (TableChange.DropColumn) change);  
                }  
            }  
        }  
    }  
}
```

### 场景 3：物理存储标识处理

FLIP-294 考虑了一个重要问题：**同一个物理存储可能对应多个逻辑表**。

比如，下面两个表实际上指向同一个 Kafka Topic：

```
-- 表1  
CREATE TABLE orders_view1 (  
    order_id STRING,  
    amount DECIMAL  
) WITH (  
    'connector'='kafka',  
    'topic'='orders_topic',  
    'properties.bootstrap.servers'='localhost:9092',  
    'properties.group.id'='group1'  
);  
  
-- 表2    
CREATE TABLE orders_view2 (  
    order_id STRING,  
    amount DECIMAL  
) WITH (  
    'connector'='kafka',  
    'topic'='orders_topic',  -- 同一个 topic  
    'properties.bootstrap.servers'='localhost:9092',  
    'properties.group.id'='group2'  
);
```

在监听器中，你可以通过连接器配置来识别物理存储：

```
private String buildKafkaIdentifier(CatalogBaseTable table) {  
    Map<String, String> options = table.getOptions();  
      
    String servers = options.get("properties.bootstrap.servers");  
    String topic = options.get("topic");  
      
    // 构建物理标识：kafka://servers/topic  
    return String.format("kafka://%s/%s", servers, topic);  
}
```

## 配置和使用

### 1. 实现监听器工厂

```
public classMyListenerFactoryimplementsCatalogModificationListenerFactory {  
      
    @Override  
    public CatalogModificationListener createListener(Context context) {  
        Configurationconfig= context.getConfiguration();  
        ClassLoaderclassLoader= context.getUserClassLoader();  
        Executorexecutor= context.getIOExecutor();  
          
        returnnewMyCatalogListener(config, executor);  
    }  
}
```

### 2. 配置监听器

在 Flink 配置中添加：

```
table.catalog-modification.listeners: com.mycompany.MyListenerFactory,com.mycompany.MyListenerFactory
```

### 3. 部署监听器

将实现的监听器 JAR 包放到 Flink 的 classpath 中，确保客户端和集群都能访问到。

## 设计要点

### 1. 性能考虑

* • **同步执行**：监听器是同步执行的，建议不要在监听器中执行阻塞操作
* • **异步处理**：如果需要执行 I/O 操作，应该在监听器内部使用异步处理
* • **线程池**：可以使用 Context 提供的 IOExecutor 来执行异步任务

### 2. 错误处理

* • **独立性**：多个监听器是相互独立的，一个监听器出错不会影响其他监听器
* • **异常隔离**：Flink 会捕获监听器的异常，避免影响主流程

### 3. 安全考虑

* • **敏感信息**：对于数据库密码等敏感信息，用户可以在监听器中进行脱敏处理
* • **权限控制**：可以在监听器中根据用户权限过滤上报的信息

## 与 FLIP-314 的关系

FLIP-294 主要关注 **目录修改监听**，而即将推出的 FLIP-314 将支持 **作业血缘监听**。两者结合起来，可以提供完整的数据血缘追踪能力：

* • **FLIP-294**：追踪表的创建、修改、删除
* • **FLIP-314**：追踪作业运行时的数据流向关系

## 总结

FLIP-294 虽然看起来是一个"幕后英雄"的功能，但它为 Flink 的数据治理能力提供了重要的基础设施。通过这个功能，你可以：

1. 1. **自动化数据血缘追踪**：不再需要手动维护表之间的关系
2. 2. **实时感知表变更**：第一时间获知表结构的变化
3. 3. **集成外部系统**：方便地将元数据信息同步到外部系统
4. 4. **提升数据治理水平**：为合规和审计提供完整的数据链路信息

这个改进让 Flink 不仅仅是一个流处理引擎，更成为了现代数据架构中数据治理的重要一环。随着数据合规要求越来越严格，这样的能力将变得越来越重要。

在下一个版本中，配合 FLIP-314 的作业血缘监听功能，Flink 将能够提供端到端的完整数据血缘追踪能力，让你的数据治理工作变得更加轻松高效。