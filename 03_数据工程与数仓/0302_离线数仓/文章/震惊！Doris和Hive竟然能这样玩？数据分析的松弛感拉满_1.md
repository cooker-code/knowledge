---
title: 震惊！Doris和Hive竟然能这样玩？数据分析的松弛感拉满
author: 一臻数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650486089&idx=1&sn=07c51b59bff10411e3e0e7dcd2b10ba8&chksm=f2a299ed56dd60525628e916f5234d242d7406f3d2ff7da1fa067699de1b0be6193a1b13ee0f&mpshare=1&scene=24&srcid=1213yeAFo1pBdxUpJWwYUsvW&sharer_shareinfo=bf4dfc59e51bdf03c5169146d71bdab9&sharer_shareinfo_first=bf4dfc59e51bdf03c5169146d71bdab9#rd
---

更多趣文请关注一臻数据

> ❝
>
> 凌晨三点，办公室里只剩下屏幕的幽光。
>
> 数据工程师小明正在和两个"大家伙"较劲 —— Doris和Hive。
>
> "导出、清洗、导入..."他机械地在不同组件来回重复着这些步骤，眼睛都开始冒金星了。这样的场景在数据团队中太常见了，
>
> 让人不禁想问：难道真的要这样手动倒腾数据一辈子吗？就在这时，Doris向Hive递出了"橄榄枝" —— Hive Catalog闪亮登场！
>
> 它就像是给这对"数据CP"安排了一场完美联姻，从此Doris可以直接读写Hive的数据，让两个系统能够"双宿双飞"。不管是HDFS还是对象存储，不管是简单查询还是复杂分析，一个Catalog就能搞定！
>
> 这个神奇的功能让小明眼前一亮，终于可以跟那些繁琐的数据同步说再见了。让我们一起来揭秘这个数据工程师的"续命神器"吧！

## Doris与Hive的完美邂逅

深夜，小明正对着屏幕发愁。作为一名数据工程师，他面临着一个棘手的问题：公司的数据分散在Doris和Hive两个系统中，每次跨系统分析数据都要手动导出导入，繁琐且低效。

"要是能让Doris直接读写Hive的数据就好了..."他喃喃自语。

这个烦恼不止小明一个人有。随着数据量的爆炸式增长，企业的数据架构越发复杂，数据存储分散在各个系统中。如何打通这些数据孤岛，实现统一的数据访问和分析，成为了一个普遍的技术痛点。

好消息是，Apache Doris通过Hive Catalog功能早已完美解决了这个问题。它好比是在Doris和Hive之间架起了一座桥梁，让两个系统能够无缝协作。

**从Doris 2.1.3版本开始**，通过Hive Catalog，Doris不仅能够查询和导入Hive中的数据，还能**执行创建表、回写数据**等操作，真正实现了湖仓一体的架构设计。

Hive Catalog的核心价值在于它提供了一个统一的数据访问层。对数据开发人员来说，不需要再关心数据具体存储在哪里，只需要通过Doris就能完成所有数据操作。比如，可以直接在Doris中创建Hive表：

```
CREATE CATALOG hive PROPERTIES (  
    'type'='hms',  
    'hive.metastore.uris' = 'thrift://172.21.16.47:7004'  
);
```

设置完成后，就能像操作普通Doris表一样操作Hive表了。不仅支持查询，还能执行INSERT、CREATE TABLE AS SELECT等写入操作。系统会自动处理分区管理、文件格式转换等复杂细节。

更令人振奋的是，Doris还提供了完善的安全机制。通过集成Kerberos认证和Ranger权限管理，企业不用担心数据安全问题。可以精确控制到列级别的访问权限，保证数据访问的合规性。

现在，小明终于露出了笑容。有了Hive Catalog，他每天的工作效率提升了不少。跨系统的数据分析变得如此简单，就像在操作同一个系统一样流畅。

这仅仅是开始，下文我们将探讨Hive Catalog更多强大的特性。一起来瞅瞅Doris+Hive数据湖仓一体化的新篇章！

## Doris-Hive Catalog核心特性

小明最近又遇到了新的挑战。公司的数据分析场景越来越复杂，既有传统的HDFS存储，又引入了对象存储。

如何让Doris优雅地处理这些不同的存储媒介？让我们深入浅出Doris Hive Catalog的强大功能。

### 多样化的存储支持

每个存储系统都有其特色。HDFS+Hive适合历史全量大规模离线数据处理，对象存储则具有高可扩展性和低成本优势...

But，Hive Catalog提供了统一的访问接口，屏蔽了底层存储的差异：

```
-- 连接S3  
CREATE CATALOG hive_s3 PROPERTIES (  
    "type"="hms",  
    "hive.metastore.uris" = "thrift://172.0.0.1:9083",  
    "s3.endpoint" = "s3.us-east-1.amazonaws.com",  
    "s3.region" = "us-east-1",  
    "s3.access_key" = "ak",  
    "s3.secret_key" = "sk",  
    "use_path_style" = "true"  
);  
-- 可选属性：  
-- s3.connection.maximum：S3 最大连接数，默认 50  
-- s3.connection.request.timeout：S3 请求超时时间，默认 3000ms  
-- s3.connection.timeout：S3 连接超时时间，默认 1000ms  
  
-- 连接OSS  
CREATE CATALOG hive_oss PROPERTIES (  
    "type"="hms",  
    "hive.metastore.uris" = "thrift://172.0.0.1:9083",  
    "oss.endpoint" = "oss.oss-cn-beijing.aliyuncs.com",  
    "oss.access_key" = "ak",  
    "oss.secret_key" = "sk"  
);  
  
...
```

### 智能的元数据管理

Doris采用智能的元数据缓存机制，在保证数据一致性的同时提供高性能查询：

**本地缓存策略**

Doris会在本地缓存表的元数据信息，减少对HMS的访问频率。当缓存数量超过阈值后，会使用 LRU（Least-Recent-Used）策略移除部分缓存。

**智能刷新**

通过订阅HMS的Notification Event，Doris能及时感知元数据变更。例如可以在创建 Catalog 时，设置该 Catalog 的定时刷新：

```
CREATE CATALOG hive PROPERTIES (  
    'type'='hms',  
    'hive.metastore.uris' = 'thrift://172.0.0.1:9083',  
    'metadata_refresh_interval_sec' = '3600'  
);
```

也可以按需手动刷新：

```
-- 刷新指定 Catalog。  
REFRESH CATALOG ctl1 PROPERTIES("invalid_cache" = "true");  
  
-- 刷新指定 Database。  
REFRESH DATABASE [ctl.]db1 PROPERTIES("invalid_cache" = "true");  
  
-- 刷新指定 Table。  
REFRESH TABLE [ctl.][db.]tbl1;
```

### 企业级安全特性

安全永远是企业数据管理的重中之重。Hive Catalog同时也提供了完整的安全解决方案：

**Ranger权限控制**

Apache Ranger 是一个用来在 Hadoop 平台上进行监控，启用服务，以及全方位数据安全访问管理的安全框架。

Doris 支持为指定的 External Hive Catalog 使用 Apache Ranger 进行鉴权。

目前支持 Ranger 的库、表、列的鉴权，暂不支持加密、行权限、Data Mask 等功能。

只需要配置下FE所有环境，再创建 Catalog 时增加即可：

```
-- access_controller.properties.ranger.service.name 指的是 service 的类型  
-- 例如 hive，hdfs 等。并不是配置文件中 ranger.plugin.hive.service.name 的值。  
"access_controller.properties.ranger.service.name" = "hive",  
"access_controller.class" = "org.apache.doris.catalog.authorizer.ranger.hive.RangerHiveAccessControllerFactory",
```

**Kerberos认证**

除了可以集成Ranger，Doris Hive Catalog也支持与企业现有的Kerberos认证体系无缝集成。例如：

```
CREATE CATALOG hive_krb PROPERTIES (  
    'type'='hms',  
    'hive.metastore.uris' = 'thrift://172.0.0.1:9083',  
    'hive.metastore.sasl.enabled' = 'true',  
    'hive.metastore.kerberos.principal' = 'your-hms-principal',  
    'hadoop.security.authentication' = 'kerberos',  
    'hadoop.kerberos.keytab' = '/your-keytab-filepath/your.keytab',     
    'hadoop.kerberos.principal' = 'your-principal@YOUR.COM',  
    'yarn.resourcemanager.principal' = 'your-rm-principal'  
);
```

...

小明现在可以根据不同的业务需求，灵活选择存储方式和安全模式，真正实现了Doris+Hive数据的统一管理和高效分析。

**数据湖和数据仓库的边界正在模糊，Doris通过Hive Catalog架起了连接两个世界的桥梁**。随着技术的不断演进，我们期待看到更多创新的应用场景。

下期，我们将一起探讨其它更有趣有用有价值的内容，敬请期待！

---

一臻数据致力于大数据AI时代的前沿内容分享，会持续分享更多有趣有用有态度的知识。同时也欢迎大家**投稿，共建共进**，帮助圈友们冲破认知壁垒，实现自我提升！

另外，整理了份《**一臻数据知识库**》，其中包含 **Apache Doris** 和**Data+AI****的学习资料、学习课程、白皮书、研究报告、行业标准**和**实践指南** 等内容，会持续更新，欢迎**关注公众号，免费领取**。

**资料获取** 🔗 欢迎扫描下方二维码图片 备注【**Doris**】免费领取❗️

---

往期推荐

[*走进*开源，拥抱开源](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483656&idx=1&sn=300e90f5017ebb3d97d3e98d26d52ff7&chksm=f374e9aec40360b85c87a26d9d1af93b2807ad1c54340676ce3173fb0508b3be7ca9595182f0&scene=21#wechat_redirect)

[*大数据*平台开发规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482778&idx=1&sn=6a4a6b3bf16ab818aa38d222ce46fed6&chksm=f374ea3cc403632a0f6ef1a9728393b459c3d19926f8e9672467f278e9f56abb010b198d2b34&scene=21#wechat_redirect)

[*大数据*仓库开发规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483237&idx=1&sn=824d2125280a009dddeec3f0aa60c4f6&chksm=f374e843c40361551cbbf48c7ad58fb054246a1abc5a60166a29aa7c7a547903848873149902&scene=21#wechat_redirect)

[*大数据*质量管制规范示例](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483244&idx=1&sn=1f87c073c411c338ff73f095d250f2e5&chksm=f374e84ac403615c9006d4bcfa49374d5f652b742a7851f9848cb9ea8c0909d53c70dee36f40&scene=21#wechat_redirect)

[*Flink* CDC 1.0至3.0回忆录](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483092&idx=1&sn=396c5873c5b2bf6d66532daeae4b445f&chksm=f374ebf2c40362e4ebac29580dcb7add14c9980290769de9366924d98ad27b3e5dc3f1095613&scene=21#wechat_redirect)

[【Apache *Doris*】Manager 极致丝滑地运维管理](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482971&idx=1&sn=b0953da7f28e016edd032a5be9bd34e5&chksm=f374eb7dc403626b97e57c4e2ec4922e3c17fa71fd2714623cd84328716a53b839976e59f0c3&scene=21#wechat_redirect)

[【Apache *Doris*】如何一键实现MySQL万表整库同步？](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482994&idx=1&sn=1e10161f6f7e9777a8fd572e7bd91482&chksm=f374eb54c4036242d7b83a58bd0533e6187444adae3c6defb0e0dc6ecfe0f314c3965c2d4872&scene=21#wechat_redirect)

[【Apache *Doris*】如何实现高并发点查？（原理+实践全析）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483049&idx=1&sn=e60a95fe683a663f249af230e6606bce&chksm=f374eb0fc403621998ca50c01bad8e093f788a982ca33463d28863840e3bfd3c94791f38efb9&scene=21#wechat_redirect)

[为什么Apache *Doris*适合做大数据的复杂计算，MySQL不适合？](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650483064&idx=1&sn=330709a47b85d247af7510b1557fe868&chksm=f374eb1ec40362088903ac541bd4ea3d1d4ea7962b95214405132d4a0a96fae6e23c671787d6&scene=21#wechat_redirect)

[如何正确地使用ChatGPT（角色扮演+提示工程）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482824&idx=1&sn=429e15f252a79bf18f72161c9ccc04af&chksm=f374eaeec40363f8aff13083a2ee0ef60b07ab56ac6c7aaff5ae282a5092aae82d00455c1633&scene=21#wechat_redirect)

[超强满血不收费的AI绘图教程来了（在线Stable Diffusion一键即用）](http://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650482860&idx=1&sn=ad75b434b7eae3511017c67d14dad143&chksm=f374eacac40363dc192372fbdd36ff76ad8cbdd866df5a3287c3e48476f4c9c88f2ef3e5f3db&scene=21#wechat_redirect)

点击下方蓝字关注一臻数据