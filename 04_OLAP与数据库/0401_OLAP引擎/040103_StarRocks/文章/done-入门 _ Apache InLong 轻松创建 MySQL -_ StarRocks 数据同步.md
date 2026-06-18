> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040103_StarRocks/040103_核心知识点/StarRocks实时数仓与湖仓接入边界|StarRocks实时数仓与湖仓接入边界]]
---
title: 入门 | Apache InLong 轻松创建 MySQL -> StarRocks 数据同步
author: Apache InLong
date:
url: http://mp.weixin.qq.com/s?__biz=Mzk0NDMwMTY1OQ==&mid=2247485763&idx=3&sn=36ea00ec710c3ff8ad515134c794673c&chksm=c327fa77f45073619baabe752f210fde4c265f2cedf0a87edfc23633071c71f4d1f2c94d953c&mpshare=1&scene=24&srcid=1226ixz32aCzTdXNWcZbRavb&sharer_shareinfo=10f1accddb722bf111c52e17d841343a&sharer_shareinfo_first=10f1accddb722bf111c52e17d841343a#rd
---

在下面的内容中，我们将通过一个完整的示例介绍如何使用 Apache InLong 创建 MySQL -> StarRocks 数据同步。

### 环境部署

##### 安装 InLong

InLong 有多重部署方式，这里我们使用 Docker 部署[1]。

* 下载最新发布的 Apache InLong 1.10.0 安装包，下载地址：https://downloads.apache.org/inlong/1.10.0/apache-inlong-1.10.0-bin.tar.gz

解压 apache-inlong-1.10.0-bin.tar.gz

```
# 进入安装目录
cd docker/docker-compose
# 启动
docker-compose up -d
```

##### 添加 Connectors

下载 Flink 1.13 对应版本的 connectors[2]，解压后将 sort-connector-starrocks-1.10.0-SNAPSHOT.jar 放在 /inlong-sort/connectors/ 目录下。

```
# docker cp sort-connector-starrocks-1.10.0-SNAPSHOT.jar manager:/opt/inlong-sort/connectors/
```

**注：inlong-sort 与 inlong-manager 同在 manager 容器中。**

##### 安装 StarRocks

```
docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 -itd starrocks.docker.scarf.sh/starrocks/allin1-ubuntu
```

部署之后可以使用 MySQL 客户端链接该 StarRocks

```
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

**注：详细安装文档请参考 Apache StarRocks 官网教程**：使用 Docker 部署 StarRocks[3]

### 集群初始化

容器启动成功后，访问 InLong Dashboard 地址 http://localhost，并使用以下默认账号登录：

```
User: admin
Password: inlong
```

##### 创建集群标签

页面点击 **【集群管理】->【标签管理】->【新建】**，指定集群标签名称和负责人：

**注：default\_cluster 是各个组件默认上报集群标签，如果使用其它名称，确认对应标签配置已修改。**

##### 注册 Pulsar 集群

页面点击 **【集群管理】 -> 【集群管理】 -> 【新建集群】**，注册 Pulsar 集群：

* 集群标签选择刚创建的 default\_cluster
* 配置 Docker 部署的 Pulsar 集群：Service URL 为 pulsar://pulsar:6650, Admin URL 为 http://pulsar:8080

##### 注册 StarRocks 数据节点

页面点击 **【数据节点】 -> 【创建】** ，新增 StarRocks 数据节点.

* LOAD URL 请勿携带 http:// 填写 IP + 端口即可。

### 任务创建

##### 新建数据流组

页面点击**【数据同步】 → 【创建】**，输入 Group ID、Steam ID 和 是否整库迁移：

##### 创建数据源

数据源中点击 **【新建】 → 【MySQL】** 配置数据源名称、地址、库表信息等。

* 读取模式选择 全量+增量 时，表中的存量数据也会被采集，仅增量 模式则不会。
* 表名白名单格式为.，支持正则表达。

##### 创建数据目标

数据目标中点击 **【新建】 → 【StarRocks】**，设置数据目标名称并选择创建好的 StarRocks 数据节点, 并填写库表名称。

##### 审批数据流

点击 **【审批管理】 -> 【我的审批】 -> 【审批】 -> 【通过】**.

返回【数据集成】，等待任务配置成功：

### 测试数据

##### 发送数据

进入 MySQL 容器

```
docker exec -it mysql /bin/bash
```

```
#!/bin/bash
# MySQL info
DB_HOST="mysql"
DB_USER="root"
DB_PASS="inlong"
DB_NAME="test"
DB_TABLE="source_table"

# Insert data in a loop
for ((i=1; i<=1000; i++))
do
    # Generate data
    id=$i
    name="name_$i"
    
    # Build an insert SQL
    query="INSERT INTO $DB_TABLE (id, name) VALUES ($id, '$name');"
    
    # Execute insert SQL
    mysql -h $DB_HOST -u $DB_USER -p$DB_PASS $DB_NAME -e "$query"
done
```

根据实际环境修改脚本中的变量，执行脚本向 source\_table 表中累计添加 1000 条数据:

##### 验证数据

进入 StarRocks，查看 sink\_table 表数据

也可以在 Dashboard 页面查看审计对账数据：

### 参考资料

[1]

Docker 部署: https://inlong.apache.org/zh-CN/docs/next/deployment/docker/

[2]

connectors: https://inlong.apache.org/zh-CN/downloads/

[3]

使用 Docker 部署 StarRocks: https://docs.starrocks.io/zh/docs/quick\_start/deploy\_with\_docker/