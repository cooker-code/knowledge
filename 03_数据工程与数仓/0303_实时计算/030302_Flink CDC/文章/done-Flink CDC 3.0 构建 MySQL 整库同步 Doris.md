> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/030302_核心知识点/FlinkCDC整库同步Doris链路边界|FlinkCDC整库同步Doris链路边界]]
---
title: Flink CDC 3.0 构建 MySQL 整库同步 Doris
author: 数智AI科技盒
date:
url: http://mp.weixin.qq.com/s?__biz=MzA5MDQyNjMxMg==&mid=2649593820&idx=1&sn=edd429f5914a812ae1f2bde029c01d77&chksm=8812e97bbf65606debc0839e5e503874b7bab6b500cc181a64e9953b16dc41bf84122bb6a8cf&mpshare=1&scene=24&srcid=0102QGk6NxqOLotyXuml9U3g&sharer_shareinfo=483a64f1a78bd0f85cb474ed809b3f45&sharer_shareinfo_first=483a64f1a78bd0f85cb474ed809b3f45#rd
---

**一、前置准备**

1、因为后面要搭建集群，所以需要准备3台linux服务器，集群服务器节点提前安装JDK 8以上版本。

2、集群的服务器之间配置好ssh免密登录，避免后续搭建出现麻烦，这一步一定要做。简单步骤如下：

1. 在master机器执行`ssh-keygen -t rsa`
2. 在master机器执行命令，将密钥拷贝到其余服务器`ssh-copy-id -i /root/.ssh/id_rsa.pub 目标服务器IP`

### 二、Flink Standalone搭建

### **1、下载flink**

https://dlcdn.apache.org/flink/flink-1.18.0/flink-1.18.0-bin-scala\_2.12.tgz

在JobManager服务器解压安装包到/opt/flink-1.18.0 目录

**2、配置flink环境变量**

vim /etc/profile

export FLINK\_HOME=/opt/flink-1.18.0

export PATH=$FLINK\_HOME/bin:$PATH

source /etc/profile

**3、修改配置文件conf/flink-conf.yaml**

新增或修改如下

taskmanager.numberOfTaskSlots: 8

jobmanager.rpc.address: 10.10.0.10

rest.bind-address: 0.0.0.0

#自定义IP可设置

rest.port: 31992

state.savepoints.dir: file:///opt/flink/savepoints

### 4、修改workers文件

workers文件必须包含所有需要启动的TaskManager节点的主机名，且每个主机名占一行。在JobManager服务器，执行以下操作

vim conf/workers

修改为其余两台TaskManager的ip地址：

```
10.10.0.11

10.10.0.12
```

### 5、复制Flink安装文件到其他服务器

在JobManager服务器执行命令，将安装文件复制到其余TaskManager服务器，命令如下：

scp -r /opt/flink-1.18.0/ 10.10.0.11:/opt/flink-1.18.0

scp -r /opt/flink-1.18.0/ 10.10.0.12:/opt/flink-1.18.0

**6、**启动集群

在JobManager节点上进入Flink安装目录，执行以下命令启动Flink集群：

bin/start-cluster.sh

启动完毕后，在集群各服务器上通过jsp命令查看Java进程。若各节点存在以下进程，则说明集群启动成功：

JobManager节点：StandaloneSessionClusterEntrypoint

TaskManager1节点：TaskManagerRunner

TaskManager2节点：TaskManagerRunner

**启动成功的话，可以在****http://10.10.0.10:8081/****访问到 Flink Web UI，如下所示：**

**三、通过 FlinkCDC cli 提交任务**

## **1、下载connector 包，并且移动到 lib 目录下**

https://repo1.maven.org/maven2/com/ververica/flink-cdc-pipeline-connector-doris/3.0.0/flink-cdc-pipeline-connector-doris-3.0.0.jar

https://repo1.maven.org/maven2/com/ververica/flink-cdc-pipeline-connector-mysql/3.0.0/flink-cdc-pipeline-connector-mysql-3.0.0.jar

## **2、编写任务配置 yaml 文件**

下面给出了一个整库同步的示例文件 mysql-to-doris.yaml：

```
source:
  type: mysql
  hostname: localhost
  port: 3306
  username: root
  password: 123456
  tables: mysql.\.*
  server-id: 5400-5404
  server-time-zone: UTC

sink:
  type: doris
  fenodes: 127.0.0.1:8030
  username: root
  password: ""
  table.create.properties.light_schema_change: true
  table.create.properties.replication_num: 1

pipeline:
  name: Sync MySQL Database to Doris
  parallelism: 8
```

```
3、通过命令行提交任务到 Flink Standalone cluster

  bash bin/flink-cdc.sh mysql-to-doris.yaml

提交成功后，返回信息如：

Pipeline has been submitted to cluster.
  Job ID: 30f4580f1918bebf16752d4963dc54

Job Description: Sync MySQL Database to Doris
```

打开 Flink Web UI，查看任务