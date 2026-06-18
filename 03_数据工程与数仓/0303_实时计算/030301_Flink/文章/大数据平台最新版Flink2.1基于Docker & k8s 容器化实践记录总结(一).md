---
title: 大数据平台最新版Flink2.1基于Docker & k8s 容器化实践记录总结(一)
author: 大数据从业者
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU1NjYyMDE5OA==&mid=2247486907&idx=1&sn=ca3f30246f09b4d1e90aaa9114decdd2&chksm=fa4a5ed8ee4d759e1f714213572e999229f98c91eb233593442622b7965550043574c7ba2bb5&mpshare=1&scene=24&srcid=1104HkhnBReUYQEi42CZpXtA&sharer_shareinfo=f5255a06bb07393e2c97d79db4a2a67e&sharer_shareinfo_first=f5255a06bb07393e2c97d79db4a2a67e#rd
---

前言

实时计算平台的Flink在既有的基于Hadoop Yarn资源调度的弊端日益突出，迁移到k8s容器化环境迫在眉睫。本系列文章从0到1记录Flink Docker & k8s容器化实践总结，本文属于第一篇。文章发布于微信公众号:大数据从业者，其它均为转载，原创不易，欢迎您点赞关注推荐转发，谢谢！

Docker生态

主要涉及Docker、Docker Compose、Docker Swarm，简要对比图如下：

Docker管理单个容器生命周期：创建、启动、停止、删除，需手动管理容器间的依赖关系（如网络、存储）。

Docker Compose通过声明式YAML文件管理同一主机的多容器应用，自动化服务依赖、网络和存储。通过depends\_on控制服务启动顺序。通过自定义网络实现容器间基于服务名可以直接通信。

Docker Swarm将多台主机抽象为统一集群，实现容器跨节点调度和负载均衡。核心组件，Manager节点：集群控制面（推荐≥3个，基于Raft协议高可用）。 Worker节点：运行容器实例。

接下来介绍下安装，根据官网文档安装:

```
https://docs.docker.com/engine/install/centos/
```

官方yum源不稳定，建议使用阿里源：

```
https://developer.aliyun.com/mirror/
```

```
yum install -y yum-utilsyum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repoyum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-pluginsystemctl start dockersystemctl enable dockerdocker version
```

Docker Hub

官方docker hub仓库（实际项目都会使用自己公司内部flink版本制作镜像，使用公司内部私有hub仓库）：

```
https://hub.docker.com/_/flink/tags
```

官方仓库不稳定，可以配置Docker加速镜像仓库地址，全局配置：/etc/docker/daemon.json

阿里的Docker仓库也行：

```
"registry-mirrors": ["https://registry.cn-hangzhou.aliyuncs.com"]：
```

拉取镜像

```
docker pull flink:2.1.0-scala_2.12-java21
```

也可以命令指定镜像仓库地址：

```
docker pull docker.m.daocloud.io/flink:2.1.0-scala_2.12-java21
```

注意：这种方式拉取得的镜像会带仓库地址，不推荐。

当然，也可以之后再使用docker tag ac1e0f4b0e48 flink:2.1.0-scala\_2.12改下镜像名(类似于linux系统的软连接的效果)：

基于Docker构建Flink

```
docker network create flink-network
```

application模式

```
docker run \    --memory=8g \    --rm \    --env FLINK_PROPERTIES="jobmanager.rpc.address: jobmanager  	taskmanager.numberOfTaskSlots: 2" \    --name=application \    --network flink-network \	--publish 8081:8081 \    flink:2.1.0-scala_2.12-java21  standalone-job \    --job-classname org.apache.flink.streaming.examples.wordcount.WordCount \--jars file:///opt/flink/examples/streaming/WordCount.jar
```

注意：file路径为容器内绝对路径！也支持s3，如下：

```
docker run \    --rm \    --env FLINK_PROPERTIES="jobmanager.rpc.address: jobmanager	s3.endpoint: http://felixzh:8082" \	--env ENABLE_BUILT_IN_PLUGINS=flink-s3-fs-hadoop-2.1.0.jar \	--env AWS_ACCESS_KEY_ID=minioadmin \    --env AWS_SECRET_ACCESS_KEY=minioadmin \    --name=application \    --network flink-network \	--publish 8081:8081 \    flink:2.1.0-scala_2.12-java21  standalone-job \    --job-classname org.apache.flink.streaming.examples.wordcount.WordCount \--jars s3://flink/WordCount.jar
```

注意：以MinIO为例，如上需开启s3插件、增加s3用户/密码、s3.endpoint配置！

session模式

启动一个JobManager：

```
docker run -d \    --rm \    --name=jobmanager \    --network flink-network \    --publish 8081:8081 \    --env FLINK_PROPERTIES="jobmanager.rpc.address: jobmanager" \flink:2.1.0-scala_2.12-java21  jobmanager
```

启动n个taskmanager：

```
docker run -d \    --rm \    --name=taskmanager \    --network flink-network \    --env FLINK_PROPERTIES="jobmanager.rpc.address: jobmanager" \    flink:2.1.0-scala_2.12-java21  taskmanager
```

```
http://felixzh1:8081/
```

跨节点提交任务效果：

```
./flink run -m http://felixzh1:8081 ../examples/streaming/WordCount.jar
```

总结

本文主要记录Docker环境构建，基于Docker实践Flink两种集群模式：application、session。后续系列文章更新基于Docker compose、基于k8s 构建Flink集群的实践记录。文章发布于微信公众号:大数据从业者，其它均为转载，原创不易，欢迎您点赞关注推荐转发，谢谢！