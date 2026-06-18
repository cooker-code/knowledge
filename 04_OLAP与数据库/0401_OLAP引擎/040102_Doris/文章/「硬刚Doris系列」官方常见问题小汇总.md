---
title: 「硬刚Doris系列」官方常见问题小汇总
author: 大数据技术与架构
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247513605&idx=1&sn=5c617e579d32ebab4fa54c95e2a77872&chksm=fd3ef090ca49798613743a978f0a0852882af1cbb094fd692817d130d1ab241776df77f9e428&mpshare=1&scene=24&srcid=06023glhMWqYuXogMZoBNU4Z&sharer_sharetime=1654146887554&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

点击上方**蓝色字体**，选择“设为星标”

回复"**面试"**获取更多惊喜

> [轻戳有惊喜：全网最全大数据面试提升手册！](https://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247512011&idx=1&sn=68aaa9c5c42e2087d56d170c442b30ba&scene=21#wechat_redirect)

阅读读本文前必读：

* [「](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247513463&idx=1&sn=e00b910f4db4f5c6648526533f1bc26e&chksm=fd3ef3e2ca497af404c4c45a60e9d82a59cc1f73030f3b6c8868cf6feaf57698bb972704359a&scene=21#wechat_redirect)[硬刚Doris系列」Apache Doris基本使用和数据模型](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247513421&idx=1&sn=59392c6c261f4d968cb2e46ba7c881cc&chksm=fd3ef3d8ca497aceffe1658ff4f2ec846760942ff230198ca5f265afb57ac81319e062221a37&scene=21#wechat_redirect)
* [「硬刚Doris系列」Apache Doris架构原理及核心特性解读](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247513508&idx=1&sn=6197f809026c8dcab7c8a921bd68cd53&chksm=fd3ef331ca497a27b069cc1d673f9de536d8e77ee347bc3c269d3a599324624d46b09005ca35&scene=21#wechat_redirect)
* [「硬刚Doris系列」Doris高级用法](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247513463&idx=1&sn=e00b910f4db4f5c6648526533f1bc26e&chksm=fd3ef3e2ca497af404c4c45a60e9d82a59cc1f73030f3b6c8868cf6feaf57698bb972704359a&scene=21#wechat_redirect)
* [「硬刚Doris系列」Apache Doris的向量化和Roaring BitMap](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247513526&idx=1&sn=429a35a1e2df404ca8e563eb60125295&chksm=fd3ef323ca497a3574ebb8779407d107c2c07df9c609bc67955e2cadfb38b11db56353d3ac9e&scene=21#wechat_redirect)

## 第一部分：运维常见问题：

##### Q1. 通过 DECOMMISSION 下线BE节点时，为什么总会有部分tablet残留？

在下线过程中，通过 show backends 查看下线节点的 tabletNum ，会观察到 tabletNum 数量在减少，说明数据分片正在从这个节点迁移走。当数量减到0时，系统会自动删除这个节点。但某些情况下，tabletNum 下降到一定数值后就不变化。这通常可能有以下两种原因：

1. 这些 tablet 属于刚被删除的表、分区或物化视图。而刚被删除的对象会保留在回收站中。而下线逻辑不会处理这些分片。可以通过修改 FE 的配置参数 catalog\_trash\_expire\_second 来修改对象在回收站中驻留的时间。当对象从回收站中被删除后，这些 tablet就会被处理了。
2. 这些 tablet 的迁移任务出现了问题。此时需要通过 show proc "/cluster\_balance" 来查看具体任务的错误了。

对于以上情况，可以先通过 show proc "/statistic" 查看集群是否还有 unhealthy 的分片，如果为0，则可以直接通过 drop backend 语句删除这个 BE 。否则，还需要具体查看不健康分片的副本情况。

##### Q2. priorty\_network 应该如何设置？

priorty\_network 是 FE、BE 都有的配置参数。这个参数主要用于帮助系统选择正确的网卡 IP 作为自己的 IP 。建议任何情况下，都显式的设置这个参数，以防止后续机器增加新网卡导致IP选择不正确的问题。

priorty\_network 的值是 CIDR 格式表示的。分为两部分，第一部分是点分十进制的 IP 地址，第二部分是一个前缀长度。比如 10.168.1.0/8 会匹配所有 10.xx.xx.xx 的IP地址，而 10.168.1.0/16 会匹配所有 10.168.xx.xx 的 IP 地址。

之所以使用 CIDR 格式而不是直接指定一个具体 IP，是为了保证所有节点都可以使用统一的配置值。比如有两个节点：10.168.10.1 和 10.168.10.2，则我们可以使用 10.168.10.0/24 来作为 priorty\_network 的值。

##### Q3. FE的Master、Follower、Observer都是什么？

首先明确一点，FE 只有两种角色：Follower 和 Observer。而 Master 只是一组 Follower 节点中选择出来的一个 FE。Master 可以看成是一种特殊的 Follower。所以当我们被问及一个集群有多少 FE，都是什么角色时，正确的回答当时应该是所有 FE 节点的个数，以及 Follower 角色的个数和 Observer 角色的个数。

所有 Follower 角色的 FE 节点会组成一个可选择组，类似 Poxas 一致性协议里的组概念。组内会选举出一个 Follower 作为 Master。当 Master 挂了，会自动选择新的 Follower 作为 Master。而 Observer 不会参与选举，因此 Observer 也不会称为 Master 。

一条元数据日志需要在多数 Follower 节点写入成功，才算成功。比如3个 FE ，2个写入成功才可以。这也是为什么 Follower 角色的个数需要是奇数的原因。

Observer 角色和这个单词的含义一样，仅仅作为观察者来同步已经成功写入的元数据日志，并且提供元数据读服务。他不会参与多数写的逻辑。

通常情况下，可以部署 1 Follower + 2 Observer 或者 3 Follower + N Observer。前者运维简单，几乎不会出现 Follower 之间的一致性协议导致这种复杂错误情况（百度内部集群大多使用这种方式）。后者可以保证元数据写的高可用，如果是高并发查询场景，可以适当增加 Observer。

##### Q4. 节点新增加了新的磁盘，为什么数据没有均衡到新的磁盘上？

当前Doris的均衡策略是以节点为单位的。也就是说，是按照节点整体的负载指标（分片数量和总磁盘利用率）来判断集群负载。并且将数据分片从高负载节点迁移到低负载节点。如果每个节点都增加了一块磁盘，则从节点整体角度看，负载并没有改变，所以无法触发均衡逻辑。

此外，Doris目前并不支持单个节点内部，各个磁盘间的均衡操作。所以新增磁盘后，不会将数据均衡到新的磁盘。

但是，数据在节点之间迁移时，Doris会考虑磁盘的因素。比如一个分片从A节点迁移到B节点，会优先选择B节点中，磁盘空间利用率较低的磁盘。

这里我们提供3种方式解决这个问题：

1. 重建新表

   通过create table like 语句建立新表，然后使用 insert into select的方式将数据从老表同步到新表。因为创建新表时，新表的数据分片会分布在新的磁盘中，从而数据也会写入新的磁盘。这种方式适用于数据量较小的情况（几十GB以内）。
2. 通过Decommission命令

   decommission命令用于安全下线一个BE节点。该命令会先将该节点上的数据分片迁移到其他节点，然后在删除该节点。前面说过，在数据迁移时，会优先考虑磁盘利用率低的磁盘，因此该方式可以“强制”让数据迁移到其他节点的磁盘上。当数据迁移完成后，我们在cancel掉这个decommission操作，这样，数据又会重新均衡回这个节点。当我们对所有BE节点都执行一遍上述步骤后，数据将会均匀的分布在所有节点的所有磁盘上。

   注意，在执行decommission命令前，先执行以下命令，以避免节点下线完成后被删除。

   `admin set frontend config("drop_backend_after_decommission" = "false");`
3. 使用API手动迁移数据

   Doris提供了HTTP API，可以手动指定一个磁盘上的数据分片迁移到另一个磁盘上。

##### Q5. 如何正确阅读 FE/BE 日志?

很多情况下我们需要通过日志来排查问题。这里说明一下FE/BE日志的格式和查看方式。

1. FE

   FE日志主要有：

* fe.log：主日志。包括除fe.out外的所有内容。
* fe.warn.log：主日志的子集，仅记录 WARN 和 ERROR 级别的日志。
* fe.out：标准/错误输出的日志（stdout和stderr）。
* fe.audit.log：审计日志，记录这个FE接收的所有SQL请求。

  一条典型的FE日志如下：

  `2021-09-16 23:13:22,502 INFO (tablet scheduler|43) [BeLoadRebalancer.selectAlternativeTabletsForCluster():85] cluster is balance: default_cluster with medium: HDD. skip`
* 2021-09-16 23:13:22,502：日志时间。
* INFO：日志级别，默认是INFO。
* (tablet scheduler|43)：线程名称和线程id。通过线程id，就可以查看这个线程上下文信息，方面排查这个线程发生的事情。
* BeLoadRebalancer.selectAlternativeTabletsForCluster():85：类名、方法名和代码行号。
* cluster is balance xxx：日志内容。

通常情况下我们主要查看fe.log日志。特殊情况下，有些日志可能输出到了fe.out中。

2. BE

   BE日志主要有：

* be.INFO：主日志。这其实是个软连，连接到最新的一个 be.INFO.xxxx上。
* be.WARNING：主日志的子集，仅记录 WARN 和 FATAL 级别的日志。这其实是个软连，连接到最新的一个 be.WARN.xxxx上。
* be.out：标准/错误输出的日志（stdout和stderr）。

  一条典型的BE日志如下：

  `I0916 23:21:22.038795 28087 task_worker_pool.cpp:1594] finish report TASK. master host: 10.10.10.10, port: 9222`
* I0916 23:21:22.038795：日志等级和日期时间。大写字母I表示INFO，W表示WARN，F表示FATAL。
* 28087：线程id。通过线程id，就可以查看这个线程上下文信息，方面排查这个线程发生的事情。
* task\_worker\_pool.cpp:1594：代码文件和行号。
* finish report TASK xxx：日志内容。

通常情况下我们主要查看be.INFO日志。特殊情况下，如BE宕机，则需要查看be.out。

##### Q6. FE/BE 节点挂了应该如何排查原因?

1. BE

   BE进程是 C/C++ 进程，可能会因为一些程序Bug（内存越界，非法地址访问等）或 Out Of Memory（OOM）导致进程挂掉。此时我们可以通过以下几个步骤查看错误原因：

   1.查看be.out

   BE进程实现了在程序因异常情况退出时，会打印当前的错误堆栈到be.out里（注意是be.out，不是be.INFO或be.WARNING）。通过错误堆栈，通常能够大致获悉程序出错的位置。

   注意，如果be.out中出现错误堆栈，通常情况下是因为程序bug，普通用户可能无法自行解决，欢迎前往微信群、github discussion 或dev邮件组寻求帮助，并贴出对应的错误堆栈，以便快速排查问题。

   2.dmesg

   如果be.out没有堆栈信息，则大概率是因为OOM被系统强制kill掉了。此时可以通过dmesg -T 这个命令查看linux系统日志，如果最后出现 Memory cgroup out of memory: Kill process 7187 (palo\_be) score 1007 or sacrifice child 类似的日志，则说明是OOM导致的。

   内存问题可能有多方面原因，如大查询、导入、compaction等。Doris也在不断优化内存使用。欢迎前往微信群、github discussion 或dev邮件组寻求帮助。

   3.查看be.INFO中是否有F开头的日志。

   F开头的日志是 Fatal 日志。如 F0916 ，表示9月16号的Fatal日志。Fatal日志通常表示程序断言错误，断言错误会直接导致进程退出（说明程序出现了Bug）。欢迎前往微信群、github discussion 或dev邮件组寻求帮助。

   4.Minidump(removed)

   Mindump 是 Doris 0.15 版本之后加入的功能，具体可参阅文档 (opens new window)。
2. FE

   FE 是 java 进程，健壮程度要由于 C/C++ 程序。通常FE 挂掉的原因可能是 OOM（Out-of-Memory）或者是元数据写入失败。这些错误通常在 fe.log 或者 fe.out 中有错误堆栈。需要根据错误堆栈信息进一步排查。

##### Q7. 关于数据目录SSD和HDD的配置, 建表有时候会遇到报错Failed to find enough host with storage medium and tag

Doris支持一个BE节点配置多个存储路径。通常情况下，每块盘配置一个存储路径即可。同时，Doris支持指定路径的存储介质属性，如SSD或HDD。SSD代表高速存储设备，HDD代表低速存储设备。

如果集群只有一种介质比如都是HDD或者都是SSD，最佳实践是不用在be.conf中显式指定介质属性。如果遇到上述报错`Failed to find enough host with storage medium and tag`，一般是因为be.conf中只配置了SSD的介质，而fe中参数default\_storage\_medium默认为HDD，因此建表时会发现没有HDD介质的存储而报错。解决方案可以修改此FE配置并重启FE生效；或者将be.conf中SSD的显式配置去掉；或者建表时增加properties参数 `properties {"storage_medium" = "ssd"}`均可

通过指定路径的存储介质属性，我们可以利用Doris的冷热数据分区存储功能，在分区级别将热数据存储在SSD中，而冷数据会自动转移到HDD中。

需要注意的是，Doris并不会自动感知存储路径所在磁盘的实际存储介质类型。这个类型需要用户在路径配置中显式的表示。比如路径 "/path/to/data1.SSD" 即表示这个路径是SSD存储介质。而 "data1.SSD" 就是实际的目录名称。Doris是根据目录名称后面的 ".SSD" 后缀来确定存储介质类型的，而不是实际的存储介质类型。也就是说，用户可以指定任意路径为SSD存储介质，而Doris仅识别目录后缀，不会去判断存储介质是否匹配。如果不写后缀，则默认为HDD。

换句话说，".HDD" 和 ".SSD" 只是用于标识存储目录“相对”的“低速”和“高速”之分，而并不是标识实际的存储介质类型。所以如果BE节点上的存储路径没有介质区别，则无需填写后缀。

##### Q8. 多个FE，在使用Nginx实现web UI负载均衡时，无法登录

Doris 可以部署多个FE，在访问Web UI的时候，如果使用Nginx进行负载均衡，因为Session问题会出现不停的提示要重新登录，这个问题其实是Session共享的问题，Nginx提供了集中Session共享的解决方案，这里我们使用的是nginx中的ip\_hash技术，ip\_hash能够将某个ip的请求定向到同一台后端，这样一来这个ip下的某个客户端和某个后端就能建立起稳固的session，ip\_hash是在upstream配置中定义的：

```
upstream  doris.com {  
   server    172.22.197.238:8030 weight=3;  
   server    172.22.197.239:8030 weight=4;  
   server    172.22.197.240:8030 weight=4;  
   ip_hash;  
}
```

完整的Nginx示例配置如下:

```
user nginx;  
worker_processes auto;  
error_log /var/log/nginx/error.log;  
pid /run/nginx.pid;  
  
# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.  
include /usr/share/nginx/modules/*.conf;  
  
events {  
    worker_connections 1024;  
}  
  
http {  
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '  
                      '$status $body_bytes_sent "$http_referer" '  
                      '"$http_user_agent" "$http_x_forwarded_for"';  
  
    access_log  /var/log/nginx/access.log  main;  
  
    sendfile            on;  
    tcp_nopush          on;  
    tcp_nodelay         on;  
    keepalive_timeout   65;  
    types_hash_max_size 2048;  
  
    include             /etc/nginx/mime.types;  
    default_type        application/octet-stream;  
  
    # Load modular configuration files from the /etc/nginx/conf.d directory.  
    # See http://nginx.org/en/docs/ngx_core_module.html#include  
    # for more information.  
    include /etc/nginx/conf.d/*.conf;  
    #include /etc/nginx/custom/*.conf;  
    upstream  doris.com {  
      server    172.22.197.238:8030 weight=3;  
      server    172.22.197.239:8030 weight=4;  
      server    172.22.197.240:8030 weight=4;  
      ip_hash;  
    }  
  
    server {  
        listen       80;  
        server_name  gaia-pro-bigdata-fe02;  
        if ($request_uri ~ _load) {  
           return 307 http://$host$request_uri ;  
        }  
  
        location / {  
            proxy_pass http://doris.com;  
            proxy_redirect default;  
        }  
        error_page   500 502 503 504  /50x.html;  
        location = /50x.html {  
            root   html;  
        }  
    }  
 }
```

##### Q9. FE启动失败，fe.log中一直滚动 "wait catalog to be ready. FE type UNKNOWN"

这种问题通常有两个原因：

1. 本次FE启动时获取到的本机IP和上次启动不一致，通常是因为没有正确设置 priority\_network 而导致 FE 启动时匹配到了错误的 IP 地址。需修改 priority\_network 后重启 FE。
2. 集群内多数 Follower FE 节点未启动。比如有 3 个 Follower，只启动了一个。此时需要将另外至少一个 FE 也启动，FE 可选举组方能选举出 Master 已提供服务。

如果以上情况都不能解决，可以按照 Doris 官网文档中的元数据运维文档进行恢复。

##### Q10. Lost connection to MySQL server at 'reading initial communication packet', system error: 0

如果使用 MySQL 客户端连接 Doris 时出现如下问题，这通常是因为编译 FE 时使用的 jdk 版本和运行 FE 时使用的 jdk 版本不同导致的。注意使用 docker 编译镜像编译时，默认的 JDK 版本是 openjdk 11，可以通过命令切换到 openjdk 8（详见编译文档）。

##### Q11. recoveryTracker should overlap or follow on disk last VLSN of 4,422,880 recoveryFirst= 4,422,882 UNEXPECTED\_STATE\_FATAL

有时重启 FE，会出现如上错误（通常只会出现在多 Follower 的情况下）。并且错误中的两个数值相差2。导致 FE 启动失败。

这是 bdbje 的一个 bug，尚未解决。遇到这种情况，只能通过元数据运维文档 中的 故障恢复 进行操作来恢复元数据了。

##### Q12. Doris编译安装JDK版本不兼容问题

在自己使用 Docker 编译 Doris 的时候，编译完成安装以后启动FE，出现 `java.lang.Suchmethoderror: java.nio. ByteBuffer. limit (I)Ljava/nio/ByteBuffer;` 异常信息，这是因为Docker里默认是JDK 11，如果你的安装环境是使用JDK8 ，需要在 Docker 里 JDK 环境切换成 JDK8，具体切换方法参照编译文档

##### Q13. 本地启动 FE 或者启动单元测试报错 Cannot find external parser table action\_table.dat

执行如下命令

```
cd fe && mvn clean install -DskipTests
```

如果还报同样的错误，手动执行如下命令

```
cp fe-core/target/generated-sources/cup/org/apache/doris/analysis/action_table.dat fe-core/target/classes/org/apache/doris/analysis
```

##### Q14. Doris 升级到1.0 以后版本通过ODBC访问MySQL外表报错 Failed to set ciphers to use (2026)

这个问题出现在doris 升级到1.0 版本以后，且使用 Connector/ODBC 8.0.x 以上版本，Connector/ODBC 8.0.x 有多种获取方式，比如通过yum安装的方式获取的 `/usr/lib64/libmyodbc8w.so` 依赖的是 `libssl.so.10` 和 `libcrypto.so.10` 而doris 1.0 以后版本中openssl 已经升级到1.1 且内置在doris 二进制包中，因此会导致 openssl 的冲突进而出现 类似 如下的错误

```
ERROR 1105 (HY000): errCode = 2, detailMessage = driver connect Error: HY000 [MySQL][ODBC 8.0(w) Driver]SSL connection error: Failed to set ciphers to use (2026)
```

解决方式是使用`Connector/ODBC 8.0.28` 版本的 ODBC Connector， 并且选择 在操作系统处选择 `Linux - Generic`, 这个版本的ODBC Driver 使用 openssl 1.1 版本。具体使用方式见 ODBC外表使用文档 可以通过如下方式验证 MySQL ODBC Driver 使用的openssl 版本

```
ldd /path/to/libmyodbc8w.so |grep libssl.so
```

如果输出包含 `libssl.so.10` 则使用过程中可能出现问题， 如果包含`libssl.so.1.1` 则与doris 1.0 兼容

## 第二部分：数据操作问题：

##### Q1. 使用 Stream Load 访问 FE 的公网地址导入数据，被重定向到内网 IP？

当 stream load 的连接目标为FE的http端口时，FE仅会随机选择一台BE节点做http 307 redirect 操作，因此用户的请求实际是发送给FE指派的某一个BE的。而redirect返回的是BE的ip，也即内网IP。所以如果你是通过FE的公网IP发送的请求，很有可能因为redirect到内网地址而无法连接。

通常的做法，一种是确保自己能够访问内网IP地址，或者是给所有BE上层架设一个负载均衡，然后直接将 stream load 请求发送到负载均衡器上，由负载均衡将请求透传到BE节点。

##### Q2. Doris 是否支持修改列名？

不支持修改列名。

Doris支持修改数据库名、表名、分区名、物化视图（Rollup）名称，以及列的类型、注释、默认值等等。但遗憾的是，目前不支持修改列名。

因为一些历史原因，目前列名称是直接写入到数据文件中的。Doris在查询时，也是通过类名查找到对应的列的。所以修改列名不仅是简单的元数据修改，还会涉及到数据的重写，是一个非常重的操作。

我们不排除后续通过一些兼容手段来支持轻量化的列名修改操作。

##### Q3. Unique Key模型的表是否支持创建物化视图？

不支持。

Unique Key模型的表是一个对业务比较友好的表，因为其特有的按照主键去重的功能，能够很方便的同步数据频繁变更的业务数据库。因此，很多用户在将数据接入到Doris时，会首先考虑使用Unique Key模型。

但遗憾的是，Unique Key模型的表是无法建立物化视图的。原因在于，物化视图的本质，是通过预计算来将数据“预先算好”，这样在查询时直接返回已经计算好的数据，来加速查询。在物化视图中，“预计算”的数据通常是一些聚合指标，比如求和、求count。这时，如果数据发生变更，如udpate或delete，因为预计算的数据已经丢失了明细信息，因此无法同步的进行更新。比如一个求和值5，可能是 1+4，也可能是2+3。因为明细信息的丢失，我们无法区分这个求和值是如何计算出来的，因此也就无法满足更新的需求。

##### Q4. tablet writer write failed, tablet\_id=27306172, txn\_id=28573520, err=-235 or -215 or -238

这个错误通常发生在数据导入操作中。新版错误码为 -235，老版本错误码可能是 -215。这个错误的含义是，对应tablet的数据版本超过了最大限制（默认500，由 BE 参数 `max_tablet_version_num` 控制），后续写入将被拒绝。比如问题中这个错误，即表示 27306172 这个tablet的数据版本超过了限制。

这个错误通常是因为导入的频率过高，大于后台数据的compaction速度，导致版本堆积并最终超过了限制。此时，我们可以先通过show tablet 27306172 语句，然后执行结果中的 show proc 语句，查看tablet各个副本的情况。结果中的 versionCount即表示版本数量。如果发现某个副本的版本数量过多，则需要降低导入频率或停止导入，并观察版本数是否有下降。如果停止导入后，版本数依然没有下降，则需要去对应的BE节点查看be.INFO日志，搜索tablet id以及 compaction关键词，检查compaction是否正常运行。关于compaction调优相关，可以参阅 ApacheDoris 公众号文章：Doris 最佳实践-Compaction调优(3)

-238 错误通常出现在同一批导入数据量过大的情况，从而导致某一个 tablet 的 Segment 文件过多（默认是 200，由 BE 参数 `max_segment_num_per_rowset` 控制）。此时建议减少一批次导入的数据量，或者适当提高 BE 配置参数值来解决。

##### Q5. tablet 110309738 has few replicas: 1, alive backends: [10003]

这个错误可能发生在查询或者导入操作中。通常意味着对应tablet的副本出现了异常。

此时，可以先通过 show backends 命令检查BE节点是否有宕机，如 isAlive 字段为false，或者 LastStartTime 是最近的某个时间（表示最近重启过）。如果BE有宕机，则需要去BE对应的节点，查看be.out日志。如果BE是因为异常原因宕机，通常be.out中会打印异常堆栈，帮助排查问题。如果be.out中没有错误堆栈。则可以通过linux命令dmesg -T 检查是否是因为OOM导致进程被系统kill掉。

如果没有BE节点宕机，则需要通过show tablet 110309738 语句，然后执行结果中的 show proc 语句，查看tablet各个副本的情况，进一步排查。

##### Q6. disk xxxxx on backend xxx exceed limit usage

通常出现在导入、Alter等操作中。这个错误意味着对应BE的对应磁盘的使用量超过了阈值（默认95%）此时可以先通过 show backends 命令，其中MaxDiskUsedPct展示的是对应BE上，使用率最高的那块磁盘的使用率，如果超过95%，则会报这个错误。

此时需要前往对应BE节点，查看数据目录下的使用量情况。其中trash目录和snapshot目录可以手动清理以释放空间。如果是data目录占用较大，则需要考虑删除部分数据以释放空间了。具体可以参阅磁盘空间管理。

##### Q7. 通过 Java 程序调用 stream load 导入数据，在一批次数据量较大时，可能会报错 Broken Pipe

除了 Broken Pipe 外，还可能出现一些其他的奇怪的错误。

这个情况通常出现在开启httpv2后。因为httpv2是使用spring boot实现的http 服务，并且使用tomcat作为默认内置容器。但是tomcat对307转发的处理似乎有些问题，所以后面将内置容器修改为了jetty。此外，在java程序中的 apache http client的版本需要使用4.5.13以后的版本。之前的版本，对转发的处理也存在一些问题。

所以这个问题可以有两种解决方式：

1. 关闭httpv2

   在fe.conf中添加 enable\_http\_server\_v2=false后重启FE。但是这样无法再使用新版UI界面，并且之后的一些基于httpv2的新接口也无法使用。（正常的导入查询不受影响）。
2. 升级

   可以升级到 Doris 0.15 及之后的版本，已修复这个问题。

##### Q8. 执行导入、查询时报错-214

在执行导入、查询等操作时，可能会遇到如下错误：

```
failed to initialize storage reader. tablet=63416.1050661139.aa4d304e7a7aff9c-f0fa7579928c85a0, res=-214, backend=192.168.100.10
```

-214 错误意味着对应 tablet 的数据版本缺失。比如如上错误，表示 tablet 63416 在 192.168.100.10 这个 BE 上的副本的数据版本有缺失。（可能还有其他类似错误码，都可以用如下方式进行排查和修复）。

通常情况下，如果你的数据是多副本的，那么系统会自动修复这些有问题的副本。可以通过以下步骤进行排查：

首先通过 `show tablet 63416` 语句并执行结果中的 `show proc xxx` 语句来查看对应 tablet 的各个副本情况。通常我们需要关心 `Version` 这一列的数据。

正常情况下，一个 tablet 的多个副本的 Version 应该是相同的。并且和对应分区的 VisibleVersion 版本相同。

你可以通过 `show partitions from tblx` 来查看对应的分区版本（tablet 对应的分区可以在 `show tablet` 语句中获取。）

同时，你也可以访问 `show proc` 语句中的 CompactionStatus 列中的 URL（在浏览器打开即可）来查看更具体的版本信息，来检查具体丢失的是哪些版本。

如果长时间没有自动修复，则需要通过 `show proc "/cluster_balance"` 语句，查看当前系统正在执行的 tablet 修复和调度任务。可能是因为有大量的 tablet 在等待被调度，导致修复时间较长。可以关注 `pending_tablets` 和 `running_tablets` 中的记录。

更进一步的，可以通过 `admin repair` 语句来指定优先修复某个表或分区，具体可以参阅 `help admin repair`;

如果依然无法修复，那么在多副本的情况下，我们使用 `admin set replica status` 命令强制将有问题的副本下线。具体可参阅 `help admin set replica status` 中将副本状态置为 bad 的示例。（置为 bad 后，副本将不会再被访问。并且会后续自动修复。但在操作前，应先确保其他副本是正常的）

##### Q9. Not connected to 192.168.100.1:8060 yet, server\_id=384

在导入或者查询时，我们可能遇到这个错误。如果你去对应的 BE 日志中查看，也可能会找到类似错误。

这是一个 RPC 错误，通常有两种可能：1. 对应的 BE 节点宕机。2. rpc 拥塞或其他错误。

如果是 BE 节点宕机，则需要查看具体的宕机原因。这里只讨论 rpc 拥塞的问题。

一种情况是 OVERCROWDED，即表示 rpc 源端有大量未发送的数据超过了阈值。BE 有两个参数与之相关：

1. `brpc_socket_max_unwritten_bytes`：默认 1GB，如果未发送数据超过这个值，则会报错。可以适当修改这个值以避免 OVERCROWDED 错误。（但这个治标不治本，本质上还是有拥塞发生）。
2. `tablet_writer_ignore_eovercrowded`：默认为 false。如果设为true，则 Doris 会忽略导入过程中出现的 OVERCROWDED 错误。这个参数主要为了避免导入失败，以提高导入的稳定性。

第二种是 rpc 的包大小超过 max\_body\_size。如果查询中带有超大 String 类型，或者 bitmap 类型时，可能出现这个问题。可以通过修改以下 BE 参数规避：

```
brpc_max_body_size：默认 3GB.
```

## 第三部分：SQL问题：

##### Q1. 查询报错：Failed to get scan range, no queryable replica found in tablet: xxxx

这种情况是因为对应的 tablet 没有找到可以查询的副本，通常原因可能是 BE 宕机、副本缺失等。可以先通过 `show tablet tablet_id` 语句，然后执行后面的 `show proc` 语句，查看这个 tablet 对应的副本信息，检查副本是否完整。同时还可以通过 `show proc "/cluster_balance"` 信息来查询集群内副本调度和修复的进度。

关于数据副本管理相关的命令，可以参阅 数据副本管理。

# Q2. show backends/frontends 查看到的信息不完整

在执行如`show backends/frontends` 等某些语句后，结果中可能会发现有部分列内容不全。比如show backends结果中看不到磁盘容量信息等。

通常这个问题会出现在集群有多个FE的情况下，如果用户连接到非Master FE节点执行这些语句，就会看到不完整的信息。这是因为，部分信息仅存在于Master FE节点。比如BE的磁盘使用量信息等。所以只有在直连Master FE后，才能获得完整信息。

当然，用户也可以在执行这些语句前，先执行 `set forward_to_master=true;` 这个会话变量设置为true后，后续执行的一些信息查看类语句会自动转发到Master FE获取结果。这样，不论用户连接的是哪个FE，都可以获取到完整结果了。

##### Q3. invalid cluster id: xxxx

这个错误可能会在show backends 或 show frontends 命令的结果中出现。通常出现在某个FE或BE节点的错误信息列中。这个错误的含义是，Master FE向这个节点发送心跳信息后，该节点发现心跳信息中携带的 cluster id和本地存储的 cluster id不同，所以拒绝回应心跳。

Doris的 Master FE 节点会主动发送心跳给各个FE或BE节点，并且在心跳信息中会携带一个cluster\_id。cluster\_id是在一个集群初始化时，由Master FE生成的唯一集群标识。当FE或BE第一次收到心跳信息后，则会将cluster\_id以文件的形式保存在本地。FE的该文件在元数据目录的image/目录下，BE则在所有数据目录下都有一个cluster\_id文件。之后，每次节点收到心跳后，都会用本地cluster\_id的内容和心跳中的内容作比对，如果不一致，则拒绝响应心跳。

该机制是一个节点认证机制，以防止接收到集群外的节点发送来的错误的心跳信息。

如果需要恢复这个错误。首先要先确认所有节点是否都是正确的集群中的节点。之后，对于FE节点，可以尝试修改元数据目录下的 image/VERSION 文件中的 cluster\_id 值后重启FE。对于BE节点，则可以删除所有数据目录下的 cluster\_id 文件后重启 BE。

##### Q4. Unique Key 模型查询结果不一致

某些情况下，当用户使用相同的 SQL 查询一个 Unique Key 模型的表时，可能会出现多次查询结果不一致的现象。并且查询结果总在 2-3 种之间变化。

这可能是因为，在同一批导入数据中，出现了 key 相同但 value 不同的数据，这会导致，不同副本间，因数据覆盖的先后顺序不确定而产生的结果不一致的问题。

比如表定义为 k1, v1。一批次导入数据如下：

```
1, "abc"  
1, "def"
```

那么可能副本1 的结果是 `1, "abc"`，而副本2 的结果是 `1, "def"`。从而导致查询结果不一致。

为了确保不同副本之间的数据先后顺序唯一，可以参考 Sequence Column 功能。

如果这个文章对你有帮助，不要忘记 **「在看」** **「点赞」** **「收藏」** 三连啊喂！

[2022年全网首发|大数据专家级技能模型与学习指南(胜天半子篇)](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247510040&idx=1&sn=aa335f25965975731173916f012d56f4&chksm=fd3eee8dca49679b82f632048fb64d21fac01497d1a0fe33917edb01e194caf0f9a1930a55ce&scene=21#wechat_redirect)

[互联网最坏的时代可能真的来了](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247508317&idx=1&sn=0bcb7fb6997b42306994b890eaa0d47f&chksm=fd3ee7c8ca496ede347a2971002c6ea68dcfd70abeeb90b43c8b02a2a60ebc7cec3a87ef7d52&scene=21#wechat_redirect)

[我在B站读大学，大数据专业](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247507860&idx=1&sn=807ac5003762f29c127bc4071dcebe33&chksm=fd3e9901ca4910178cfc816043ea86c29f881f70cd943d252de9ae2e21cb7e8ba80bff162e1b&scene=21#wechat_redirect)

[我们在学习Flink的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247499604&idx=1&sn=d938dfb30d221774704982d2938b30c1&chksm=fd3eb9c1ca4930d76a391241333de461ca22d2aa27472eab3cffab1564872ae37f1b48fe2d3c&scene=21#wechat_redirect)

[193篇文章暴揍Flink，这个合集你需要关注一下](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504856&idx=2&sn=6f62e2a0c756ce56773eed253d2f41ee&chksm=fd3e954dca491c5b732b7e2aa46db32efbc3f690e528522098659bb3c6e78e1582ab4529b5dc&scene=21#wechat_redirect)

[Flink生产环境TOP难题与优化，阿里巴巴藏经阁YYDS](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504742&idx=1&sn=8765c198d8ad66219a7bcabb221e4a23&chksm=fd3e95f3ca491ce52d0724b9e4154a47af0f5ea349e1e184c8fbc65aa17c0000242aa9b58429&scene=21#wechat_redirect)

[Flink CDC我吃定了耶稣也留不住他！| Flink CDC线上问题小盘点](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504813&idx=1&sn=f5cd6ae2aa2b1e30f87a5ae55971c514&chksm=fd3e9538ca491c2eb191677d070f2c7e4f1098eece00e256d6205b8a5d21b21c1e74f875f16f&scene=21#wechat_redirect)

[我们在学习Spark的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=1&sn=1f693e549f76622725b936041ff8896e&chksm=e9bff2fbdec87bedecbdeed4547d7d612c72fc8d4918614a26a4e31666cf0128fd57b537f347&scene=21#wechat_redirect)

[在所有Spark模块中，我愿称SparkSQL为最强！](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504834&idx=1&sn=46ca1a3924b8fdb89ac1c2acb0319d6c&chksm=fd3e9557ca491c41d837b917639ea62007ea16d3c2cc6c0c02d1f8a8c6e44318f5183b787c46&scene=21#wechat_redirect)

[硬刚Hive | 4万字基础调优面试小总结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247502750&idx=1&sn=bd9a9173d060dc4e4ebd49c8efc6acfe&chksm=fd3e8d0bca49041dea84da93910e5efdc4935e520525c09887c986691377aeb48e5cf7fb5667&scene=21#wechat_redirect)

[数据治理方法论和实践小百科全书](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504382&idx=2&sn=550b78802acfe727e0e77cd9195f8784&chksm=fd3e976bca491e7db2b8b2446d231736df01bbf13653804d680ac4390597ec17fa466ad4ae83&scene=21#wechat_redirect)

[标签体系下的用户画像建设小指南](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503741&idx=1&sn=e5039be93123f2e337013756a818bfc3&chksm=fd3e89e8ca4900fe603b63c5722a6fb8a32bd63d6ba23e0028851948a71b877eb1f742d95087&scene=21#wechat_redirect)

[4万字长文 | ClickHouse基础&实践&调优全视角解](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503675&idx=1&sn=3ee6af64d0126c78b48cad219308f81e&chksm=fd3e89aeca4900b8b8954e9569ee3c0877881fac8c792bfafc22e7e9d3e8524da8eb860d33d8&scene=21#wechat_redirect)

[【面试&个人成长】2021年过半，社招和校招的经验之谈](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=2&sn=57ecc77718f1b6f2f262d62e8318dcc9&chksm=e9bff2fbdec87bedb1986765bfae7dbabb9aece45b0b2af147bbad1ac5d79ebc934c64df40ea&scene=21#wechat_redirect)

[大数据方向另一个十年开启 |《硬刚系列》第一版完结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504478&idx=1&sn=14efef3868ba42bd044f9618745a7fdc&chksm=fd3e94cbca491dddb961b7b8b93105b2869c5bcf4c03f9f0dc83ad62be0dce4c124b52ed0ba3&scene=21#wechat_redirect)

[我写过的关于成长/面试/职场进阶的文章](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504410&idx=1&sn=7e81ab5a324395eb0f12397c40247ca1&chksm=fd3e948fca491d9946456acbd93b2d651ae4d7a7d0127a211981e08f50f377e60d1b7c0d7b3a&scene=21#wechat_redirect)

[当我们在学习Hive的时候在学习什么？「硬刚Hive续集」](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504783&idx=1&sn=72aed147a459368ed934a007a2df3bc9&chksm=fd3e951aca491c0cade212390011eca7d8a68951fee6aa2844b8f4d14bcbd5b3ed26305d69dc&scene=21#wechat_redirect)