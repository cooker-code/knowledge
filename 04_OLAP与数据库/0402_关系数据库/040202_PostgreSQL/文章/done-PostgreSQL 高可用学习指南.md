> 已吸收至：[[04_OLAP与数据库/0402_关系数据库/040202_PostgreSQL/040202_核心知识点/PostgreSQL架构WAL与高可用边界|PostgreSQL架构WAL与高可用边界]]
---
title: PostgreSQL 高可用学习指南
author: DBA百宝箱
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg2MDUyODE5OA==&mid=2247484060&idx=1&sn=0e4888041f765a5696545818deffa7d8&chksm=cfd04feb27e6faa5f8df194e0ed5728900f06042cb0bf90db62fbdaee01e5c07c038393883be&mpshare=1&scene=24&srcid=11071fAjz0WotTFdGWZc2vSN&sharer_shareinfo=fd0290ff576af3846fc3cda16704bfdd&sharer_shareinfo_first=fd0290ff576af3846fc3cda16704bfdd#rd
---

PostgreSQL（简称 PG）的高可用（High Availability, HA）设计强调零数据丢失、快速故障转移和负载均衡，主要通过流复制（Streaming Replication）、逻辑复制（Logical Replication）、WAL（预写日志）同步、集群管理工具（如 Patroni、repmgr）和监控实现。PG 18（2025 年 9 月发布）新增了增强的逻辑复制订阅和 failover 优化，进一步提升 HA 鲁棒性。学习 PG HA 有助于 DBA 构建生产级集群、实现 RTO（恢复时间目标）<1 分钟和 RPO（恢复点目标）=0。本指南基于 2025 年 10 月最新资源，整合官方文档、书籍、课程和社区资料，适合有 PG 基础的学习者。建议结合 PG 18 源码和 Kubernetes 环境（如 CloudNativePG）实践。

     指南分为书籍、课程、在线资源和学习路径四个部分。每个部分附实践提示，帮助从理论到部署落地。

#### 1. 推荐书籍

书籍聚焦 HA 原理、工具部署和实战案例，按难度排序。2025 年新增 PG 18 兼容更新，强调 Patroni 和多主复制。

| 书籍名称 | 作者/译者 | 关键内容与适用性 | 推荐理由 |
| --- | --- | --- | --- |
| **《PostgreSQL 高可用性手册》（PostgreSQL High Availability Cookbook, Second Edition）** | Shaun M. Thomas（有中译本） | 流复制配置、同步/异步模式、故障转移脚本、pgpool-II 负载均衡、备份恢复。 | 经典 cookbook 式指南，覆盖 PG 18 的 WAL 优化和多租户 HA。适合初学者，包含脚本示例。 |
| **《PostgreSQL 高可用实战》** | 胡辉 等（电子工业出版社） | repmgr、Patroni、BDR 多主复制、真实环境集群部署、故障模拟。 | 实战导向，揭示主流 HA 技术的原理与 pitfalls，如脑裂（split-brain）避免。2025 年建议结合 PG 18 逻辑复制阅读。 |
| **《PostgreSQL DBA (v17, v16, v15, v14, v13) - 2025 第二版》** | 不详（Amazon） | HA 集群规划、监控（pg\_stat\_replication）、安全强化、云部署。 | 全面 DBA 手册，包含秘密技巧如自动 failover 和性能调优。适用于 on-prem 和云环境。 |
| **《PostgreSQL 12 High Availability Cookbook》** | Packt Publishing | 硬件规划、复制工具保护数据、集群管理。 | 虽基于 PG 12，但核心 HA 机制通用；2025 年更新版覆盖 PG 18 扩展。强调高效运行的硬件选型。 |
| **《深入浅出 PostgreSQL》** | 屠要峰 | 入门到进阶 HA，包括复制和恢复。 | 基础+HA 结合，适合新手；PDF 版 206MB，包含安装和查询实战。 |

**实践提示**：阅读时，用 Docker Compose 搭建简单复制集群：docker run -d --name primary postgres:18，配置 postgresql.conf 中的 wal\_level = replica 和 max\_wal\_senders = 10，然后创建 standby：pg\_basebackup -h primary -D /var/lib/postgresql/data -U replicator -P -v -R。用 pg\_isready 测试 failover。

#### 2. 推荐课程

课程以视频+实验为主，2025 年新增 Patroni 和云 HA 专题。免费/付费混搭，强调 Kubernetes 集成。

* **PostgreSQL High Availability Architecture with Patroni**

  （Cybertec 课程）：全面介绍 Patroni 实现 HA，包括自动 failover、ETCD 协调和监控。时长 8+ 小时，包含 Kubernetes 部署 demo。适合企业级实践。
* **How to configure a High Availability System in PostgreSQL**

  （Udemy）：步步指导安装、配置和监控 replica 系统，附带 24x7 可用脚本。2025 年更新 PG 18 版本，免费试看。
* **EDB Postgres AI and PostgreSQL Courses - High Availability with EDB Postgres Distributed**

  （EnterpriseDB On-Demand）：设计、部署和管理分布式 HA 集群，覆盖读写分离和全球复制。免费目录，技能导向 DBA。
* **PolarDB for PostgreSQL 高可用系列**

  （阿里云/Bilibili）：10+ 讲，聚焦云 HA、流复制和故障转移。示例：Patroni + PolarDB X 的多主架构。播放量高，适合中文用户。
* **Mastering PostgreSQL HA with Patroni**

  （HexaCluster 网络研讨会，2025 年 9 月）：1 小时 webinar，讲解 leader election、best practices 和 Patroni vs. 其他工具。注册链接：https://hexacluster.com/webinar（免费）。

**实践提示**：跟随课程用 Patroni 搭建 3 节点集群：安装 pip install patroni[etcd]，配置 YAML 中的 postgresql 和 bootstrap 部分，然后 patroni /etc/patroni.yml 启动。监控用 patronictl list 检查角色切换。

#### 3. 在线资源与教程

免费资源以博客、PDF 和社区为主，2025 年聚焦 PG 18 的 HA 新特性（如增强订阅 failover）。分类精选。

* **官方核心文档**

  ：

+ **PostgreSQL 18 Replication**

  （Chapter 26-28，https://www.postgresql.org/docs/18/replication.html）：详解流/逻辑复制、同步承诺（synchronous\_standby\_names）、热备和 failover。关键：PG 18 新增的 pg\_stat\_subscription 视图监控延迟。
+ **High Availability Guide**

  （官方 Wiki）：概述工具集成，如 pg\_rewind 修复分歧。

* **英文博客与系列**

  ：

+ **Achieving PostgreSQL High Availability: Strategies and Setup Guide**

  （Percona，2025）：复制、failover、负载均衡和集群工具（如 repmgr）。包含 best practices 和部署脚本。
+ **PostgreSQL High Availability Options: A Guide**

  （Yugabyte）：消除单点故障，比较异步/同步复制和分布式 HA。适合云迁移。
+ **Complete Guide to PostgreSQL: Features, Use Cases, and Tutorial**

  （Instaclustr）：HA 用例，如读副本和自动备份。包含 JSON 查询下的 HA 扩展。

* **中文资源**

  ：

+ **从零掌握 PostgreSQL：新手万字全指南（2025 版）**

  （CSDN）：覆盖 HA 运维、性能优化和集群实战。万字详解，适合入门。
+ **PostgreSQL 面试题宝典：从基础到高级（2025 持续更新版）**

  （CSDN）：HA 专题，包括 Patroni 配置和故障模拟题。系统知识体系。
+ **墨天轮 PostgreSQL 精品学习资源合集**

  ：汇总 HA 手册、repmgr/Patroni 案例。免费下载部分资源。
+ **PostgreSQL – HA cluster s Patroni**

  （Root.cz，2025 年 9 月）：webinar 回放，聚焦 Patroni 集群。

* **社区与 X（Twitter）讨论**

  ：

+ Reddit r/PostgreSQL：线程“How to become a Postgres expert?” 推荐 HA 工具学习路径；“Book recommendations on deploying PostgreSQL 17 clusters” 讨论 on-prem HA 书籍。
+ X 帖子：@NaveenS16 分享 CloudNativePG HA 视频（YouTube）；@hexaclustercorp  webinar on Patroni（2025 年 9 月）；@Percona 推广企业 PG HA 无锁-in。

**实践提示**：用 pg\_stat\_replication 监控复制延迟：SELECT \* FROM pg\_stat\_replication;，模拟 failover：停止 primary，观察 standby 提升（pg\_ctl promote）。安装 pgpool-II 测试负载均衡。

#### 4. 学习路径建议

分阶段推进，预计 2-4 个月。结合 PG 18 新特性（如订阅增强）。

* **阶段 1: 入门（1-2 周）**

  ：读官方 Replication 章节 + 《PostgreSQL 高可用性手册》复制部分。搭建简单异步复制环境，测试 WAL 发送。实践：配置 hot\_standby = on 启用读查询。
* **阶段 2: 进阶工具（2-4 周）**

  ：跟随 Cybertec/Udemy 课程 + 《PostgreSQL 高可用实战》。重点：Patroni 自动 failover（TTL=30s）和 repmgr 见证节点。实践：Kubernetes 上部署 CloudNativePG，模拟节点故障。
* **阶段 3: 深化云与监控（1 月）**

  ：读 Percona 指南 + EDB 课程。重点：逻辑复制（CREATE PUBLICATION）、多 AZ HA 和监控（Prometheus + pg\_exporter）。实践：Azure/Google Cloud PG HA 配置，测试 RPO=0 的同步模式。
* **阶段 4: 项目与社区（持续）**

  ：参与 PGConf 2025 或 HexaCluster webinar；贡献 HA patch。用 pgBadger 分析日志，优化 max\_standby\_streaming\_delay。

**常见 pitfalls**：忽略 WAL 归档（archive\_mode = on）导致数据丢失；脑裂风险（用 quorum 机制）。