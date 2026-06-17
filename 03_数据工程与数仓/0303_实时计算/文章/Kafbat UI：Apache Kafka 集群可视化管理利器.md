---
title: Kafbat UI：Apache Kafka 集群可视化管理利器
author: 早起的码农
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI2ODYzMjkwMg==&mid=2247485544&idx=1&sn=a5881074d682b93e3ec4aae779b3d82f&chksm=ebc1061cacb4157ad5037bc89bcffb03c26a5c650fef4d31a0890408b0af2d7d68d37d34b897&mpshare=1&scene=24&srcid=0405kn9gGYr6jNghTYrxpKUm&sharer_shareinfo=496c526031a718542e78b05fa73e7008&sharer_shareinfo_first=496c526031a718542e78b05fa73e7008#rd
---

在分布式消息系统的世界里，Apache Kafka 毫无疑问是中流砥柱。但对于开发者和运维同学来说，原生命令行工具的排查效率往往让人头大。如果你在寻找一款界面清爽、功能全面且支持云原生部署的 Kafka 可视化工具，那么 Kafbat UI（原名 Kafka UI）绝对是不二之选。

## **01****Kafbat UI 是什么**

Kafbat UI 是一款免费、开源的 Apache Kafka Web 管理界面，前身是社区广泛使用的 provectus/kafka-ui（12k+ Star）。2024 年起，由项目早期核心贡献者组成的 Kafbat 团队独立运营，在 kafbat/kafka-ui 仓库持续迭代，目前已积累 2.1k+ Star。

它的定位很清晰：让 Kafka 的日常运维、排障、协作从命令行走向可视化，降低团队使用 Kafka 的门槛。

## **02 核心能力**

2.1 多集群统一管理

一个 Kafbat UI 实例可以同时接入多个 Kafka 集群（开发/测试/生产），在同一界面切换查看，避免多环境管理混乱。

2.2 Broker 概览

查看所有 Broker 节点状态

分区分配情况

Controller 角色归属

2.3 Topic 管理

一键创建 Topic，支持自定义分区数、副本因子、清理策略等参数

列表视图展示分区数、副本状态、自定义配置

支持动态修改 Topic 配置

2.4 Consumer Group 监控

查看每个 Consumer Group 的消费进度

分区级别的 Lag（堆积量）监控

成员列表与分配状态

2.5 消息浏览与生产

按 Topic/Partition/Offset 浏览消息

支持 JSON、Plain Text、Avro、Protobuf 编码

支持 Live View（实时消息流）

支持 CEL（Common Expression Language）自定义过滤

可直接在 UI 中向 Topic 发送消息

2.6 Schema Registry 集成

支持 Avro、JSON Schema、Protobuf 三种 Schema 类型

可视化创建、查看、管理 Schema

生产 Avro/Protobuf 消息前自动关联 Schema

2.7 Kafka Connect 可视化

查看 Connector 状态与任务信息

支持从 Connector 视图跳转到对应 Topic，反向亦可

2.8 安全与权限

认证方式：支持 OAuth 2.0（GitHub/GitLab/Google）、LDAP、Basic Auth

云 IAM 集成：支持 AWS IAM、GCP IAM、Azure IAM

RBAC：基于角色的细粒度访问控制

数据脱敏：对消息中的敏感字段做遮蔽处理

2.9 托管 Kafka 服务支持

AWS MSK（含 Serverless）

Azure EventHub

Google Cloud Managed Service for Apache Kafka

2.10 自定义 SerDe 插件

内置 AWS Glue、Smile 等序列化/反序列化支持，也可以自行开发插件扩展。

2.11 API 与 MCP

内置 Swagger UI，可通过 SWAGGER\_UI\_ENABLED=true 开启完整 API 文档

支持 MCP（Model Context Protocol）Server

## **03 快速部署**

3.1 Docker 一键体验

```
docker run -it -p 8080:8080 \  -e DYNAMIC_CONFIG_ENABLED=true \  -e SWAGGER_UI_ENABLED=true \  ghcr.io/kafbat/kafka-ui
```

启动后访问 http://localhost:8080 即可。

3.2 Docker Compose 持久化部署

```
services:  kafbat-ui:    container_name: kafbat-ui    image: ghcr.io/kafbat/kafka-ui:latest    ports:      - 8080:8080    environment:      DYNAMIC_CONFIG_ENABLED: 'true'      SWAGGER_UI_ENABLED: 'true'    volumes:      - ~/kui/config.yml:/etc/kafkaui/dynamic_config.yaml
```

3.3 Kubernetes Helm 部署

官方提供 Helm Chart，适合在 K8s 环境中标准化部署。

3.4 健康检查

存活/就绪探针：/actuator/health

构建信息：/actuator/info

## **04. 核心亮点**

在众多的可视化工具（如 CMAK, Redpanda Console）中，Kafbat UI 凭借以下几点脱颖而出：

轻量级与容器化：官方提供的 Docker 镜像极小，部署只需一行命令，非常适合 Kubernetes 环境。

全能的消息浏览：支持 Protobuf, Avro, JSON, Schema Registry。对于从事大数据开发、需要频繁查看 Schema 结构的同学来说，这是刚需。

多集群管理：一个实例即可统一监控和管理生产、测试等多套 Kafka 集群。

动态配置：支持在 UI 界面上直接修改 Topic 配置（如 retention.ms），无需重启 Broker 或敲复杂的命令行。

ACL 与 RBAC：内置了细粒度的权限控制，适合企业内部团队协作。

## **05 典型使用场景**

场景一：联调阶段快速验证

开发人员在联调时，打开 Kafbat UI 直接查看目标 Topic 是否有新消息写入、消息格式是否正确，无需编写消费者代码。

场景二：线上消费堆积排查

运维收到 Lag 告警后，打开 Consumer Group 页面，定位到具体哪个 Partition 堆积、哪个消费实例掉线，快速定位根因。

场景三：数据治理与合规

通过数据脱敏功能，在 UI 中浏览消息时自动遮蔽敏感字段（如手机号、身份证），满足安全审计要求。

场景四：多环境统一管控

一个 UI 实例同时接入 dev/staging/prod 集群，运维在一个页面切换观察，避免误连错误环境。

Kafbat UI 是目前 Kafka 生态中功能最全面的开源 Web 管理工具之一。它不改变 Kafka 的核心机制，但能显著提升团队在集群观察、消息排查、Consumer 监控、多环境管理上的效率。