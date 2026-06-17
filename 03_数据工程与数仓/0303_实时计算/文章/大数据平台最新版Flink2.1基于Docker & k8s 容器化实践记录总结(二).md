---
title: 大数据平台最新版Flink2.1基于Docker & k8s 容器化实践记录总结(二)
author: 大数据从业者
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU1NjYyMDE5OA==&mid=2247486921&idx=1&sn=045c2b3dc07113353e57ac146a5bdb51&chksm=fa24f8057c796e439a7b2046646bb586a80be87dbe3106450643d0d059be864b3bd5eb17fdf8&mpshare=1&scene=24&srcid=1111NZ5tBlPO9WYYlIUB7vnP&sharer_shareinfo=532456c74afd2e2eafde8752e121e31c&sharer_shareinfo_first=532456c74afd2e2eafde8752e121e31c#rd
---

前言

本系列文章从0到1记录Flink Docker & k8s容器化实践总结，本文属于第二篇。强烈推荐使用K8S。文章发布于微信公众号:大数据从业者，其它均为转载，原创不易，欢迎您点赞关注推荐转发，谢谢！

基于Docker Compose构建Flink

Docker Compose is a way to run a group of Docker containers locally.

Application模式

docker-compose-application.yaml

```
version: "2.2"services:  jobmanager:    image: flink:2.1.0-scala_2.12-java21    ports:      - "8081:8081"    command: standalone-job --job-classname org.apache.flink.streaming.examples.wordcount.WordCount --jars file:///opt/flink/examples/streaming/WordCount.jar    environment:      - |        FLINK_PROPERTIES=        jobmanager.rpc.address: jobmanager        parallelism.default: 2          
  taskmanager:    image: flink:2.1.0-scala_2.12-java21    depends_on:      - jobmanager    command: taskmanager    scale: 1    environment:      - |        FLINK_PROPERTIES=        jobmanager.rpc.address: jobmanager        taskmanager.numberOfTaskSlots: 2        parallelism.default: 2  docker compose -f docker-compose-application.yaml up
```

Session模式

docker-compose-session.yaml

```
version: "2.2"services:  jobmanager:    image: flink:2.1.0-scala_2.12-java21    ports:      - "8081:8081"    command: jobmanager    environment:      - |        FLINK_PROPERTIES=        jobmanager.rpc.address: jobmanager          
  taskmanager:    image: flink:2.1.0-scala_2.12-java21    depends_on:      - jobmanager    command: taskmanager    scale: 1    environment:      - |        FLINK_PROPERTIES=        jobmanager.rpc.address: jobmanager        taskmanager.numberOfTaskSlots: 2docker compose -f docker-compose-session.yaml up
```

```
http://felixzh1:8081/#/overview
```

跨节点提交任务效果：

```
./flink run -m http://felixzh1:8081 ../examples/streaming/WordCount.jar
```

Session模式sql client

docker-compose-session-sqlclient.yaml

```
version: "2.2"services:  jobmanager:    image: flink:2.1.0-scala_2.12-java21    ports:      - "8081:8081"    command: jobmanager    environment:      - |        FLINK_PROPERTIES=        jobmanager.rpc.address: jobmanager          
  taskmanager:    image: flink:2.1.0-scala_2.12-java21    depends_on:      - jobmanager    command: taskmanager    scale: 1    environment:      - |        FLINK_PROPERTIES=        jobmanager.rpc.address: jobmanager        taskmanager.numberOfTaskSlots: 2          sql-client:    image: flink:2.1.0-scala_2.12-java21    command: bin/sql-client.sh    depends_on:      - jobmanager      - taskmanager    environment:      - |        FLINK_PROPERTIES=        jobmanager.rpc.address: jobmanager        rest.address: jobmanagerdocker compose -f docker-compose-session-sqlclient.yaml run sql-client
```

```
CREATE TABLE input (order_number BIGINT) WITH ('connector' = 'datagen');    CREATE TABLE output (order_number BIGINT) WITH ('connector' = 'print');   insert into output select * from input;
```

总结

本文主要记录实践基于docker compose构建本地模式的Flink集群(application、session)，下一篇将记录基于K8S构建Flink最新版本2.1.0的集群的两种使用方式：Flink Native Kubernetes和flink-kubernetes-operator。文章发布于微信公众号:大数据从业者，其它均为转载，原创不易，欢迎您点赞关注推荐转发，谢谢！