> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040102_Doris/040102_核心知识点/Doris版本演进与弹性冷存候选|Doris版本演进与弹性冷存候选]]
---
title: Apache Doris 2.0 冷热分离快速体验
author: 锋哥聊湖仓
date:
url: http://mp.weixin.qq.com/s?__biz=MzI4ODMyNTcwMw==&mid=2247485768&idx=1&sn=28156f62f109d361cb8353ab1855a23c&chksm=ebc16060dcb6e976e44bd3f076a30ac09898b370e34aa09479a9d6bfa74486b1845807f5e2ba&mpshare=1&scene=24&srcid=0511X7Cl8puKbO1Q0OnEivJm&sharer_sharetime=1685197837582&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

## 概述

对于任何一种数据库类软件来说，无论其基于传统数据库模型还是基于分布式结构，作为核心的永远是数据本身。而数据的生命周期，则体现在CRUD操作（创建、查询、更新、删除）上。任何一条数据从其生成的时刻开始，数据价值随着时间的推移而逐渐降低，直至成为无用数据，最终删除。

作为使用数据的主体用户，对于各种数据的需求程度是不同的，人们往往对重要的数据有更高效、稳定的访问需求；而对于不重要的数据则没有这么高的要求，而前者存储的代价往往是远高于后者的。用户在满足了自身对于数据使用要求的情况下，自然会开始考虑数据存储成本等方面的问题，对于那些很少访问甚至基本不访问的数据，使用成本更低的存储方式将是一种更好的选择。

针对这样的使用场景，我们将数据根据用户需求分为“热数据”与“冷数据”。顾名思义，“热数据”代表着用户对其有着更频繁的访问需求，“冷数据”则很少访问。一般数据在新创建的时候往往都是“热数据”，而随着时间的推移逐步变成“冷数据”。

对于热数据，其访问的频率很高，且往往是用户非常关心的数据，其实时性要求一般都很高，并且读写的频率也会更高，这正是DORIS本地存储重点解决的问题。

对于冷数据，其数据量往往远大于热数据，并且很少被访问，使用本地存储的代价就很高，这时使用存算分离模型，将其存储到代价更低的存储载体将大大降低成本。

未来一个很大的使用场景是类似于es日志存储，日志场景下数据会按照日期来切割数据，很多数据是冷数据，查询很少，需要降低这类数据的存储成本。从节约存储成本角度考虑

1. 各云厂商普通云盘的价格都比对象存储贵
2. 在doris集群实际线上使用中，普通云盘的利用率无法达到100%
3. 云盘不是按需付费，而对象存储可以做到按需付费
4. 基于普通云盘做高可用，需要实现多副本，某副本异常要做副本迁移。而将数据放到对象存储上则不存在此类问题，因为对象存储是共享的。

## 使用体验

下面我们 Minio 为例来演示怎么使用 Doris 基于对象存储的冷热分离功能。

我是在 MacOS 上来进行安装演示的

### MacOS Doris 的编译安装

编译具体可以参照官方文档：在macOS平台上编译 - Apache Doris

本地安装单节点：快速开始 - Apache Doris

如果你是 Linux 系统，可以下载官方编译好的2.0.0 alpha 版本进行快速体验：下载 - Apache Doris

```
curl https://doris.apache.org/download-scripts/2.0.0-alpha1/download_x64_tsinghua.sh | sh
```

### Minio 安装

本文是brew方式，Mac需安装brew支持，本文不再赘述, Linux 系统下的 Minio 网上很多教程，请自行百度

```
 brew install minio/stable/minio
```

然后可以看到安装成功的信息

```
Command-line Access: https://docs.min.io/docs/minio-client-quickstart-guide

Object API (Amazon S3 compatible):
   Go:         https://docs.min.io/docs/golang-client-quickstart-guide
   Java:       https://docs.min.io/docs/java-client-quickstart-guide
   Python:     https://docs.min.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.min.io/docs/javascript-client-quickstart-guide
   .NET:       https://docs.min.io/docs/dotnet-client-quickstart-guide

Talk to the community: https://slack.min.io
==> Get started:
NAME:
  minio server - start object storage server

USAGE:
  minio server [FLAGS] DIR1 [DIR2..]
  minio server [FLAGS] DIR{1...64}
  minio server [FLAGS] DIR{1...64} DIR{65...128}

DIR:
  DIR points to a directory on a filesystem. When you want to combine
  multiple drives into a single large system, pass one directory per
  filesystem separated by space. You may also use a '...' convention
  to abbreviate the directory arguments. Remote directories in a
  distributed setup are encoded as HTTP(s) URIs.

FLAGS:
  --address value              bind to a specific ADDRESS:PORT, ADDRESS can be an IP or hostname (default: ":9000") [$MINIO_ADDRESS]
  --console-address value      bind to a specific ADDRESS:PORT for embedded Console UI, ADDRESS can be an IP or hostname [$MINIO_CONSOLE_ADDRESS]
  --ftp value                  enable and configure an FTP(Secure) server
  --sftp value                 enable and configure an SFTP server
  --certs-dir value, -S value  path to certs directory (default: "/Users/zhangfeng/.minio/certs")
  --quiet                      disable startup and info messages
  --anonymous                  hide sensitive information from logging
  --json                       output logs in JSON format
  --help, -h                   show help

EXAMPLES:
  1. Start MinIO server on "/home/shared" directory.
     $ minio server /home/shared

  2. Start single node server with 64 local drives "/mnt/data1" to "/mnt/data64".
     $ minio server /mnt/data{1...64}

  3. Start distributed MinIO server on an 32 node setup with 32 drives each, run following command on all the nodes
     $ minio server http://node{1...32}.example.com/mnt/export{1...32}

  4. Start distributed MinIO server in an expanded setup, run the following command on all the nodes
     $ minio server http://node{1...16}.example.com/mnt/export{1...32} \
            http://node{17...64}.example.com/mnt/export{1...64}

  5. Start distributed MinIO server, with FTP and SFTP servers on all interfaces via port 8021, 8022 respectively
     $ minio server http://node{1...4}.example.com/mnt/export{1...4} \
           --ftp="address=:8021" --ftp="passive-port-range=30000-40000" \
           --sftp="address=:8022" --sftp="ssh-private-key=${HOME}/.ssh/id_rsa"
   /opt/homebrew/Cellar/minio/RELEASE.2023-05-04T21-44-30Z_1: 3 files, 100.9MB, built in 3 seconds
==> Running `brew cleanup minio`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
```

### 启动服务

设置 MINIO\_REGION 、MINIO\_ACCESS\_KEY 、MINIO\_SECRET\_KEY

```
export MINIO_REGION=xian
export MINIO_ACCESS_KEY=minioadmin
export MINIO_SECRET_KEY=minioadmin
```

将 minio 服务放到后台运行

```
nohup minio server  /Users/zhangfeng/minio_data > /Users/zhangfeng/minio_data/log/minio.log 2>&1 &
```

然后可以看见登录界面：

登录进去创建 bucket 下面我们就可以进行Doris的冷热分离操作了

## Doris 冷热分离体验

首先我们在 fe/fe.conf 里打开冷热分离这个功能，因为新的功能在第一个版本默认是关闭的，所以我们要手动打开，添加下面的内容

```
enable_storage_policy=true
```

然后重启 FE。

首先我们创建一个 Resource

创建S3 RESOURCE的时候，会进行S3远端的链接校验，以保证RESOURCE创建的正确

```
CREATE RESOURCE "remote_s3"
PROPERTIES
(
    "type" = "s3",
    "AWS_ENDPOINT" = "localhost:9000",
    "AWS_REGION" = "xian",
    "AWS_BUCKET" = "test",
    "AWS_ROOT_PATH" = "/test/test001",
    "AWS_ACCESS_KEY" = "minioadmin",
    "AWS_SECRET_KEY" = "minioadmin",
    "AWS_MAX_CONNECTIONS" = "50",
    "AWS_REQUEST_TIMEOUT_MS" = "3000",
    "AWS_CONNECTION_TIMEOUT_MS" = "1000"
);
```

然后我们创建数据迁移策略(STORAGE POLICY)，用于冷热数据转换

```
CREATE STORAGE POLICY test_policy
PROPERTIES(
    "storage_resource" = "remote_s3",
    "cooldown_ttl" = "1h"
);
```

1. cooldown\_datetime：热数据转为冷数据时间，不能与cooldown\_ttl同时存在。
2. cooldown\_ttl：热数据持续时间。从数据分片生成时开始计算，经过指定时间后转为冷数据。支持的格式：1d：1天 1h：1小时 50000: 50000秒

我们后面也可以根据自己的策略来修改这个 ttl 时间，修改命令示例：

```
ALTER STORAGE POLICY test_policy PROPERTIES("cooldown_ttl" = "5h");
```

我们创建一张表，并将这个数据迁移策略应用到这个表上

```
CREATE TABLE IF NOT EXISTS create_table_use_created_policy
(
    k1 BIGINT,
    k2 LARGEINT,
    v1 VARCHAR(2048)
)
UNIQUE KEY(k1)
DISTRIBUTED BY HASH (k1) BUCKETS 1
PROPERTIES(
    "storage_policy" = "test_policy",
    "replication_num" = "1"
);
```

我们插入几条数据：

```
 insert into create_table_use_created_policy values (10001,100001,'11');
 insert into create_table_use_created_policy values (10002,100001,'11');
 insert into create_table_use_created_policy values (10003,100001,'11');
```

这里我设置了1个小时后进行冷热迁移，一个小时后我们可以在对象存储上看到数据已经迁移过来

同时我们也可以通过 Doris 提供的命令来查看

```
show tablets from tbl
```

从这个图上我们也可以看到，已经将部分数据迁移到对象存储上了

还可以通过show proc '/backends'可以查看到每个be上传到对象的大小，RemoteUsedCapacity项

我们后面也会在 `show data`这个命令加上RemoteDataSize这个属性，这样更方便用户查看表的对象存储使用情况

是不是非常简单方便呢，快点动手体验提来吧