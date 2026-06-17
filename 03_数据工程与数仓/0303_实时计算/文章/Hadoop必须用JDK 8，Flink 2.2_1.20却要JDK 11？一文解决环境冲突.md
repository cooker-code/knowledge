---
title: Hadoop必须用JDK 8，Flink 2.2/1.20却要JDK 11？一文解决环境冲突
author: BitRand
date: ggzoneggzone
url: https://mp.weixin.qq.com/s?__biz=MzkyNjQzNDE5Mw==&mid=2247483730&idx=1&sn=1c571e3424c8c53f5879f1d7e9c64c5d&chksm=c3de8eb6c95397ef2b6dbfca68e8f14f7f91799e47a4bb2d7b97586052940d946061ab9f3377&mpshare=1&scene=24&srcid=0525MUBSG7XdGOL9Nx6PhCmK&sharer_shareinfo=c150fae2ad55f742390ac1ccc99f31de&sharer_shareinfo_first=c150fae2ad55f742390ac1ccc99f31de#rd
---

> **组件版本**：Hadoop 3.3.6 | Flink 2.2.0 (兼容 1.20.2) | JDK 8 & 11

## 一、 背景与痛点

随着 **Flink 2.2.0** 的正式发布，许多企业开始规划升级路线。然而，一个尴尬的现实摆在面前：

**底层 Hadoop 3.3.6 集群由于历史包袱，必须运行在 JDK 8 上；而 Flink 2.x 及部分新生态（如新版 Iceberg、CDC）则强依赖 JDK 11。**

这就产生了核心需求：**在不统一替换集群节点 JDK 11 的前提下，如何让 Flink on YARN 的任务跑在 JDK 11 环境中？**

> **注**：本文方案同样适用于 **Flink 1.20.2** 版本，配置参数通用。

## 二、 核心思路：Ship Archives（分发归档）

既然不能动集群环境，我们就把 JDK “随身带”着跑。

Flink on YARN 提供了一个非常实用的参数：`yarn.ship-archives`。  
它的作用是：**在 Application 启动时，自动将指定的 JDK 压缩包分发到每个 Container 的工作目录中**。

这样，我们只需要告诉 Flink：“别用系统自带的 `JAVA_HOME`，用我带过来的这个。”

## 三、 环境准备

### 1. 准备 JDK 包

确保你的 JDK 11 已经上传至 HDFS（本地路径也行，但 HDFS 分发速度更快）。

```
hdfs:///user/user01/jdk/jdk-11.0.15.1_linux-x64_bin.tar.gz
```

### 2. 环境变量配置

下载 Flink 安装包解压后，配置环境变量。  
注：由于 Hadoop 3.3.6 本身支持 JDK 11，为了方便，这里直接在 ~/.bashrc 中切换了 JDK 11，你也可以配置仅在启动 Flink 时临时加载。

```
export HADOOP_CLASSPATH=$(hadoop classpath)export HIVE_CONF_DIR=/usr/bigtop/3.3.0/usr/lib/hive/confexport JAVA_HOME=/home/user01/jdk-11.0.15.1export PATH=$JAVA_HOME/bin:$PATH
```

## 四、 关键配置（避坑指南）

### 1. config.yaml 配置

这是最关键的一步。我们需要指定 JDK 归档包的位置，并告诉 Container 内部的 JAVA\_HOME 指向哪里。

```
# 本地测试（慢）# yarn.ship-archives: /home/user01/jdk-11.0.15.1_linux-x64_bin.tar.gz# HDFS分发（快，推荐）yarn.ship-archives: hdfs:///user/user01/jdk/jdk-11.0.15.1_linux-x64_bin.tar.gz  
# 指定容器内的Java路径# 注意：解压后会有一层目录 jdk-11.0.15.1_linux-x64_bin.tar.gz，里面才是真正的 jdk-11.0.15.1containerized.master.env.JAVA_HOME: '$PWD/jdk-11.0.15.1_linux-x64_bin.tar.gz/jdk-11.0.15.1'containerized.taskmanager.env.JAVA_HOME: '$PWD/jdk-11.0.15.1_linux-x64_bin.tar.gz/jdk-11.0.15.1'
```

**⚠️ 避坑点：**

1. **路径层级**：`ship-archives` 会自动解压，解压后的根目录名通常就是压缩包名，JDK 实际目录在里面。
2. **拒绝 ship-files**：尝试过 `yarn.ship-files` 方式，虽然文件能同步，但启动时会报动态链接库（`.so`）错误。
3. **命令行无效**：尝试过在命令行中指定这些配置，但未生效，写在 `config.yaml` 中是最稳妥的。

### 2. yarn-session模式验证

测试yarn session和sql-client：

```
bin/yarn-session.sh \  -d \  -qu "dev" \  -Djobmanager.memory.process.size=4096m \  -Dtaskmanager.memory.process.size=4096m \  -Dtaskmanager.numberOfTaskSlots=4 \  -Dparallelism.default=4# 由于Flink 2.2.0尚未发布对应的flink-sql-connector-hive,只测试了启动，但是Flink1.20.2访问hive正常。 bin/sql-client.sh \  -Dexecution.target=remote \  -Drest.address="xxxx" \  -Drest.port="37457"
```

### 3. yarn-application模式验证

YARN Application Mode 测试 ：

```
./bin/flink run -t yarn-application \-Dyarn.application.queue=dev \./examples/streaming/StateMachineExample.jar
```

---

## 五、 进阶探索：Docker 容器化方案

如果你的集群运维能力强，且愿意接受更高的复杂度，Hadoop 其实也支持**纯 Docker化**运行。Hadoop 可以将任务打包成 Docker 镜像，并以容器化的方式执行。但这需要将 NodeManager 的容器执行器切换为 **`LinuxContainerExecutor`**。

---

## 六、 总结

1. **Hadoop 3.3.6 集群**可以继续安稳地使用 **JDK 8**。
2. **Flink 2.2.0**（及 1.20.2）可以通过 `yarn.ship-archives` 携带 **JDK 11** 运行，无需动集群环境。

希望这篇笔记能帮你在升级 Flink 2.x 的路上少走弯路！ 🚀