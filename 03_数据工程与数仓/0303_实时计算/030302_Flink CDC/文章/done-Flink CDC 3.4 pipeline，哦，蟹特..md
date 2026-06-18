> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/030302_核心知识点/FlinkCDC版本演进与Pipeline连接器边界|FlinkCDC版本演进与Pipeline连接器边界]]
---
title: Flink CDC 3.4 pipeline，哦，蟹特.
author: 安瑞哥是码农
date:
url: https://mp.weixin.qq.com/s?__biz=MzI0OTEwNzQyNA==&mid=2247490142&idx=1&sn=7e2d2b3817f2e7551aa53c2b3269515c&chksm=e8127c1eeca991251521b7922865a374295e129699b504d70f3cbdab8e320319fe772a7c99d2&mpshare=1&scene=24&srcid=0822HviMcTOiwtGmGDivsXhv&sharer_shareinfo=48b675f61b4d38bf43dbfa4e4ea3002c&sharer_shareinfo_first=48b675f61b4d38bf43dbfa4e4ea3002c#rd
---

今年 5 月份的时候，Flink CDC 发布了最新 release 版 3.4.0，当时就有同学邀请我测它一测。

好吧，虽然有点晚了，但是，虽迟但到，今天就给安排上。

开测前，我特意看了一遍官方公众号文章发布的，关于最新 Flink CDC 3.4.0 新增或者优化的一些新特性，虽然列举了一大堆，但我更关心它的落地表现。

比如，它基于 pipeline 方式下的数据同步(单表或者整库)，是不是还只能停留在 demo 阶段？

以及，pipeline 任务的提交过程跟方式，还是不是跟以前一样，要越过正常人的思维逻辑？

0. 外表变化

从官网目前可下载的版本来看，能跟最新 release 版兼容的，最低 Flink 版本已经提高到 1.19 了(上个版本是1.18)，幸好我当前环境有这个版本，要不然还玩不转了。

0.1 包大小的变化

来，先瞅一眼最新 3.4.0 版的 jar 包，相比之前的版本，明显变大了一些：

解压后发现，这个变大的部分，就是这个核心 jar 包：

可能是集成一些了不起的功能了吧。

0.2 cdc 命令行功能的变化(flink-cdc.sh)

这里只说多出来的部分。

第 1 个：

多了一个可以通过 -D 参数，用来指定 CDC 程序运行时的一些配置。

第 2 个：

多了可以在命令中，指定任务的运行方式的参数。

相比上一个版本，这多出来的两个参数选项，在我看来已经有比较大的进步了，至于进步在哪，下面会考。

1. 跑起来

这里以测试最简单的， MySQL 同步到 Kafka 为例。

1.1 跑前准备

老规矩，因为是用的 pipeline 方式(纯命令行)，所以一开始，就需要把「数据源方」跟「数据目的方」对应的 connector ，给下载下来。

像这样：

需要注意的是——对于 MySQL 来说，还需要额外下载一个 MySQL 的 connector。

1.2 提交命令

因为是用命令行方式，所以需要准备一个同步任务的 yaml 文件。

类似这样，先测个单表的试试水：

```
source:
  type:mysql
hostname:192.168.xxx.xxx
port:3306
username:user
password:****
tables:test.test01
server-id:5400-5404
server-time-zone:Asia/Shanghai
scan.incremental.snapshot.chunk.size:10
scan.snapshot.fetch.size:10
debezium.min.row. count.to.stream.result:10

sink:
type:kafka
name:KafkaSink
properties.bootstrap.servers:PLAINTEXT://192.168.xxx.xxx:6667
topic:test

pipeline:
name:SyncMySQLDatabasetoKafka
parallelism:3
```

用什么方式提交呢？

跟上次测试的一样，yarn-session(老版本唯一支持的方式)，主要是想看看上次 3.1.1 版本遇到的那几个傻缺提交问题，这次解决了没有。

先说一个值得夸赞的地方——提交方式，终于变聪明了，或者说，终于被设计成正常人类好理解的了。

这一次，可以这样提交任务了(这里省掉了启动 yarn session 的步骤)：

```
bin/flink-cdc.sh -t yarn-session -Dyarn.application.id=application_1750922509742_0002 conf/mysql2kafka01.yml
```

你可以直接在命令行指定具体的 yarn session id。

而之前的玩法，只能是你在 Flink 的主配置文件里(flink-conf.yaml)，去额外添加下面的设置：

才可以(可能是被一个脑子清醒的人给优化了)。

但是，官方文档对于这部分的描述，却没有改过来。

而上面这种方式，是我自己试出来的。

而对于另外一个，提交任务时的 jar 包依赖问题——很遗憾，问题依然存在。

那就是，我明明在 Flink CDC 的 lib 目录下，放了这个 MySQL connector。

但是，提交集群的时候，虽然从命令行看，它能提交成功。

但是，一提交到集群就不行了：

```
Caused by: java.lang.ClassNotFoundException: com.mysql.cj.jdbc.Driver
```

根上一个版本的毛病一毛一样，没有丝毫改进。

想要解决这个问题，还得在命令行里，额外带上这个 jar 包，也就是说，你还得这样：

```
bin/flink-cdc.sh -t yarn-session -Dyarn.application.id=application_1750922509742_0002 conf/mysql2kafka01.yml --jar lib/mysql-connector-java-8.0.27.jar
```

2. 验证结果

请先容许我深呼吸一下，要不然怕忍不住说脏话......

我要说，这个最新 3.4.0 版本的 pipeline 表现，还不如上一个 3.1.1 版本，你敢信吗？

这里省掉 1306 个字......

具体表现是——只能同步历史数据(单表或者整库)，后续的新增、修改、删除都不行。

而且我还知道，这个问题的原因，是运行的 CDC 程序，没有checkpoint 导致的。

当前 Flink CDC 3.4.0 基于 Flink 1.19 环境提交的 CDC 任务

Flink CDC 3.1.1 基于 Flink 1.18 环境提交的 CDC 任务

（PS: 有兴趣的同学，可以参考 [Flink CDC 3.1.1，我又来啦.](https://mp.weixin.qq.com/s?__biz=MzI0OTEwNzQyNA==&mid=2247488385&idx=1&sn=3c0cd2e9cfc8b7cf8094a861ae43e16a&scene=21#wechat_redirect)）

但问题就在于，怎么来设置这个 checkpoint 目录，而且，让它生效。

如果在任务的命令行里指定(按理说可以)，但是，我试了下面两种方式.

第 1 种，新版本的参数方式：

```
-Dstate.backend.checkpoints.dir=hdfs://192.168.xxx.xxx:8020/tmp/flink_default_checkpoint_dir
```

不行。

第 2 种，老的参数方式：

```
-Dstate.checkpoints.dir=hdfs://192.168.xxx.xxx:8020/tmp/flink_default_checkpoint_dir
```

依然不生效。

好吧，那我用之前验证过可行的老方式，在配置文件里面指定，总可以了吧。

抱歉，还是不行(当前的 Flink 默认环境变量，指向的就是当前1.19版本)。

到这里，我已经有砸电脑的冲动了。

最后

只能说，又是一个没有经过严谨测试就敢 release 的版本，像极了一个刚完成作业还没来得及检查正确率，就急着要表扬的小学生。

本来还想测试一下，之前遇到的其他方面的 bug 的，算了，没有意义了。

当然，以上遇到的问题，希望只是我当前集群环境表现出的个例，你们验证的结果，都表现的跟营销文描述的一样完美。

---

你可以添加我的私人微信入讨论群，跟一群热爱技术的小伙伴一起成长...