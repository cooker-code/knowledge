---
title: Hive MetaStore的实现和优化
author: 数新智能科技号
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526532&idx=1&sn=67534b91947b6d2e8f3b03ab6986ac08&chksm=cfb616fdd4740048322797bd44167a8532f039a69f7dbf800ae0dd06896332755fd2a50547e2&mpshare=1&scene=24&srcid=0520HKVSrImh6Ihi62Q5059i&sharer_shareinfo=d4ff5d8c8d404136cffe5e600d9554b4&sharer_shareinfo_first=d4ff5d8c8d404136cffe5e600d9554b4#rd
---

在大数据领域，数据管理与存储至关重要，Hive MetaStore（HMS）作为 Hive 数据仓库的核心组件，承担着元数据管理的关键职责。随着数据规模不断膨胀，其性能与稳定性面临挑战。本文将深入剖析 HMS 的实现机制，结合 Hive Client、MetaCat 等相关内容展开探讨，分析各项操作流程，同时针对 HMS 及 MetaCat 提出优化方案，并阐述与 Spark 的关联等内容，助力读者全面掌握 HMS 技术要点。

**HMS**

相关类图如下：

上图颜色分类

* 绿色部分是 hive 的
* 橙色部分是 thrift 框架自动生成的代码
* 白色部分是 JDK 的

右边是 hive meta-store client，兼容了这个客户端协议的框架，如 spark，会通过 hive meta-store 协议连接过来   
左边是服务端的实现，主要继承自`ThriftHiveMetastore.Iface`，这个类包含了很多操作，CURD库、表，函数的等等

`HMSHandler`实现了这个接口，然后调用`RawStre`去一个具体数据源来获取信息，或者创建信息   
`RowStore`的实现类`ObjectStore`则调用了`javax.jdo`去连接一个真实的数据库，完成此操作

jdo 是 ORM 的实现，比 JDBC 更高层一些，这里会有一些表的连接操作，然后会转为更底层的 SQL   
hive 使用的 ORM 框架为 DataNucleus  
几个包

* org.apache.thrift.protocol 这个是底层 thrift RPC 相关类，会做序列化操作
* org.apache.hadoop.hive.metastore.api，这是Iface里面库、表会引用到这些RPC类，这些类的一些数据结构定义又引用了更下沉的thrift类
* org.apache.hadoop.hive.metastore.model，将数据中的数据读取封装成对象，再将这个对象转为thrift RPC中的对象

另外 HMS 也有直接使用SQL 链接数据库  
`ObjectStore`引用了`MetaStoreDirectSql`，这个类中就包含了很多 sql 语句，不通过 ORM 框架，直接访问  
可能是 hive 层做的一些优化

**Hive Client**

自定义hiveserver设计  
相关类图

上图颜色分类

* 蓝色部分是 自动生成的代码
* 深灰色是 client实现
* 浅灰色是 服务端逻辑
* 绿色部分是 service 层逻辑

服务端和客户端 都实现了 Iface 逻辑，也就是 thrift RPC 协议  
服务端有不同的传输实现类，binary 和 http  
业务逻辑调用绿色部分，再委托给 SessionManager获取一个session  
然后执行具体的 sql 任务

**MetaCat-相关操作**

架构如如下：

大致分类

* metacat-controller，CURD 元数据的操作
* partition-controller，分区操作，mysql 不支持这样的操作
* tag-controller，可以给表打标签
* 其他，如创建 meta-cat试图等，mysql不支持

查询 catalog

结果

查询 数据库

结果

查询表

结果

mysql 插件的配置信息：

给 tomcat 增加几个 -D 参数

* -Dmetacat.usermetadata.config.location=/usermetadata.properties的具体路径
* -Dmetacat.plugin.config.location=catalog的具体路径

**HMS 优化**

相关类如下：

颜色分类

* 灰色部分，HMS相关，其中深灰色是自动生成的代码
* 红色部分，web controller 相关的代码
* 绿色部分，service 相关逻辑
* 蓝色部分，操作 HMS 相关的类
* 黄色部分，外部依赖

Iface 是最核心类，metacat 继承了这个类，相当于是实现了 HMS 的server端RPC协议  
其中一般操作是直接调用了 controller 的类，所以 RPC 和 http 的逻辑实际是统一的  
有一些特殊的 controller，如 Tag、metadata、Search，这些会调用外部类  
如搜索会直接调用 ES，tag 和 metadata 会调用 MySQL，所以需要一个外部的 ES，MySQL 来支持这些功能

蓝色部分是操作 HMS 相关的类，分为表、库、partition 三个类  
这里有两种情况

* 浅色部分的实现类，直接调用了 Hive metastore client 来实现的
* 深蓝色部分，绕过了Hice Client，直接链接了底层的数据库，通过SQL 来交互的

所以 metacat 对HMS 的优化，可以理解为

* 将ORM操作，转为了直接的 JDBC操作
* 将ORM生成的多表关联，改为了很多单表查询，再加载到内存中做管理，减轻了 数据库端的计算压力
* 本质上相当于对 SQL 做优化

**Spark和HMS**

相关类图如下

灰色的是 spark v1 体系的类  
黄色部分是 外部catalog 相关类  
v1 中包含了两个 catalog

* InMemoryCatalog
* HiveExternalCatalog

HiveExternalCatalog 调用了 IMetaStoreClient 实现类  
也就是通过 hive MS client 向服务端发起了 RPC 请求实现 catalog 查找的

这里使用了多版本机制

**MetaCat 一些优化-对HMS做的优化SQL**

这里不通过 HMS，而是通过 JDBC，直接连底层的数据源  
DirectSqlDatabase 的主要 SQL 如下：

DirectSqlGetPartition 的主要 SQL 如下：

DirectSqlSavePartition 的主要 SQL 如下：

DirectSqlTable 的主要 SQL 如下：

**ThriftHiveMetastore 相关函数**

ThriftHiveMetastore.Iface 的所有函数

org.apache.hadoop.hive.metastore.api 包下的所有类

org.apache.hadoop.hive.metastore.api 包下的类

**参考**

* Metacat: Making Big Data Discoverable and Meaningful at Netflix
* Netflix Metacat: Origin, Architecture, Features & More
* Data Catalog and crawlers in AWS Glue

**相关文章**

* Hive论文

**往期推荐**

* [Data Ingestion: Architectural Patterns](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526497&idx=1&sn=2bbc73783fc71305d0eb36634bca52cc&scene=21#wechat_redirect)
* [Data engineering at Meta](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526484&idx=1&sn=46844f912d250474a8d4203139420312&scene=21#wechat_redirect)
* [The Life of a Read/Write Query for Apache Iceberg Tables](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526472&idx=1&sn=e8143cda02b33dd8565114a7c86fe26b&scene=21#wechat_redirect)
* [Compaction in Apache Iceberg](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526448&idx=1&sn=370b8fabffd9c5354470cdd23b8c4702&scene=21#wechat_redirect)
* [Spark原理-解析过程和Catalog](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526421&idx=1&sn=188fed136a79f9d4b77d349ec4ce67bc&scene=21#wechat_redirect)
* [Janino简单使用](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526368&idx=1&sn=8ce4f95e6b6f2a4b35f01dceeca34e3f&scene=21#wechat_redirect)
* [Oracle的CDC工具OpenLogReplicator编译](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526335&idx=1&sn=c7a3c207030624b20a1587059fa45e14&scene=21#wechat_redirect)
* [OpenLogReplicator的一些改动](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526279&idx=1&sn=983ebb9cd0e3ef34eb5bbef712543245&scene=21#wechat_redirect)
* [Spark-Streaming 原理](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526262&idx=1&sn=fad63d75a9d3dfaa8928ed656c3d9d62&scene=21#wechat_redirect)
* [P](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526198&idx=1&sn=71b952292c44f65324281b23de9d5f27&scene=21#wechat_redirect)[ar](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526198&idx=1&sn=71b952292c44f65324281b23de9d5f27&scene=21#wechat_redirect)[quet](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526198&idx=1&sn=71b952292c44f65324281b23de9d5f27&scene=21#wechat_redirect)[for](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526198&idx=1&sn=71b952292c44f65324281b23de9d5f27&scene=21#wechat_redirect)[Spar](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526198&idx=1&sn=71b952292c44f65324281b23de9d5f27&scene=21#wechat_redirect)[k](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxNDM2OA==&mid=2247526198&idx=1&sn=71b952292c44f65324281b23de9d5f27&scene=21#wechat_redirect)

如果您对相关服务感兴趣，

欢迎点击下方“阅读原文”深入了解～

↓↓↓