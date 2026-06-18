---
title: DataLens：现代化开源数据分析与可视化平台
author: 小林聊编程
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU1MjAwODAzOA==&mid=2247485211&idx=1&sn=06a5cd159152c9a36db6f85e9f3c5403&chksm=faad765539ef8ea1a2e198e73183b6c21b08e4735aa58b9d3b14d68efbe693170124ebeb6bd3&mpshare=1&scene=24&srcid=0916xocwDPTyjo4BuAYWdNdc&sharer_shareinfo=6905c8241dc38377cbd5e6d625d44089&sharer_shareinfo_first=6905c8241dc38377cbd5e6d625d44089#rd
---

# 你点赞了吗？你关注了吗？每天分享干货好文。

# **工作 3 年还在写 CRUD？**

# **工作3-5年如何突破技术瓶颈？**

# **想转技术管理但不会带团队？**

# **想跳槽没有面试的机会？**

# **不懂如何面试，迟迟拿不到 offer？** **面试屡屡碰壁，失败原因无人指导？**

***在竞争激烈的大环境下，只有不断提升核心竞争力才能立于不败之地。***

***公众号留言【我要晋级】，一对一指导，带你晋级。***

前言

DataLens 是一项商业分析服务，允许您连接到各种数据源、可视化数据、创建仪表板并共享结果。  
借助 DataLens，您可以直接从数据源跟踪产品和业务指标，从而做出数据驱动的决策。

开源地址：

https://github.com/datalens-tech/datalens 

## 一、DataLens 项目概述

DataLens 是一个**功能强大的开源数据可视化与分析平台**，它提供了从数据连接到仪表盘展示的全套解决方案。该项目旨在帮助用户轻松地从复杂数据中提取有价值信息，并通过直观的图表和仪表盘展示出来。

作为 Yandex Cloud 团队对开源软件世界的又一重要贡献，DataLens 最初是云基础架构中众多 Yandex 服务和数千家外部公司的骨干系统。2023年，Yandex 宣布将其开源，现在任何人都可以自由使用并根据自己的独特需求进行定制。

## 二、核心功能与特性

DataLens 提供了一系列强大功能，使其成为企业级数据分析和可视化的理想选择：

### 2.1 多数据源支持

DataLens 支持连接多种类型的数据源，包括：

* **传统数据库**：ClickHouse、PostgreSQL、MySQL、Greenplum、SQL Server 等
* **云服务**：Yandex Metrica、AppMetrica 服务等
* **文件数据**：CSV 文件、Excel 表格等

### 2.2 直观的可视化界面

* **拖拽式操作**：业务人员和分析师能通过直观的拖放界面自主进行数据探索和可视化，减少对开发团队的依赖
* **丰富图表类型**：提供折线图、柱状图、饼图、散点图、面积图、表格、透视表等多种可视化选项
* **仪表板定制**：可以灵活组合图表、选择器、文本等组件创建个性化仪表板

### 2.3 强大的协作能力

* **权限管理**：提供用户管理功能，可以为不同用户设置不同访问权限（管理员、编辑和查看）
* **分享与嵌入**：支持一键分享分析结果，并提供 API 接口允许将仪表板嵌入到现有企业系统中

### 2.4 实时数据处理

* 支持**实时数据刷新**功能，使用户能够第一时间获取最新的数据分析结果
* 定时自动刷新仪表板数据，确保决策基于最新信息

## 三、技术架构与实现

DataLens 采用现代化的技术栈，确保了项目的可扩展性和易用性。

### 3.1 技术组成

* **后端服务**：基于 Node.js，利用其高效的异步处理能力确保数据处理的高性能
* **数据存储**：使用 PostgreSQL 作为数据存储的核心，提供强大的数据管理和查询能力
* **前端界面**：采用 React 框架构建，提供了丰富的交互体验和响应式设计
* **容器化部署**：通过 Docker 容器化技术，能够轻松地在不同环境中部署，确保开发和生产环境的一致性

### 3.2 架构演进

DataLens 在 v2.0.0 版本中实现了显著的架构革新：

* **数据库整合**：将原本四个独立的 PostgreSQL 容器（pg-us-db、pg-auth-db、pg-compeng-db、pg-demo-db）整合到单一容器中，通过 schema 级别隔离替代容器级别隔离，提升了资源利用率和性能
* **认证体系重构**：用原生认证服务（datalens-auth）取代了原有的 Zitadel 身份认证系统，实现了更紧密的集成和更好的性能

以下是 DataLens 关键技术特性的总结：

## 四、安装与部署

DataLens 提供了灵活的部署方案，适合不同场景的需求。

### 4.1 环境要求

* **Docker 和 Docker Compose**：推荐使用 Docker 部署 DataLens
* **系统资源**：建议至少 4GB RAM 和 2CPU

### 4.2 部署方式

**Docker Compose 部署（推荐）**

```
git clone https://github.com/datalens-tech/datalenscd datalensHC=1 docker compose up
```

服务启动后，访问 `http://localhost:3030` 即可使用默认用户名和密码（admin/admin）登录。

**Kubernetes 部署**  
v2.0.0 版本新增了 Helm chart，方便在 K8s 集群中部署和管理 DataLens。

**云平台部署**  
提供了 Terraform 示例，支持在主流云平台上自动化部署整套解决方案。

### 4.3 升级注意事项

升级到新版本（如 v2.0.0）时需要注意：

* 这是一个**破坏性更新**（BREAKING CHANGE），所有现有用户账户需要重新创建
* 如果使用新的 `CONTROL_API_CRYPTO_KEY` 值，需要重新配置所有数据连接中的密码信息
* 如果使用外部 PostgreSQL 集群，需要创建额外的数据库

## 五、使用指南

### 5.1 基本工作流程

DataLens 的数据分析流程通常包括以下步骤：

1. **创建连接**（Connection）：连接到数据源
2. **定义数据集**（Dataset）：基于连接定义数据集
3. **创建图表**：利用数据集创建可视化图表
4. **构建仪表板**：将图表、选择器、文本等组件组合成仪表板

### 5.2 图表创建

DataLens 的图表向导（Wizard）是一个基于数据集的图表构建器，用户可以基于单个数据集创建无限数量的图表。图表向导界面包括：

* **数据集选择区**：位于界面底部，用于选择构建图表的数据集
* **字段列表**：显示所选数据集中可用的字段，包括维度（dimensions）和度量（measures）
* **图表类型选择**：提供多种图表类型选择按钮
* **分区设置**：根据所选图表类型提供不同的分区设置（如 X 轴、Y 轴、过滤器等）
* **预览区域**：显示可视化效果

### 5.3 高级功能

**工作簿导入导出**  
v2.2.0 版本引入了工作簿导入导出功能，方便用户在不同 DataLens 实例间迁移工作簿。该功能基于：

* **Meta-Manager 服务**：处理长时间运行任务（如工作簿导入导出）
* **Temporal 工作流引擎**：确保操作即使系统故障也能成功完成

**连接器增强**

* OracleDB 连接器新增 TCPS 支持
* YDB 连接器增加 TLS 连接支持

## 六、应用场景

DataLens 适用于多种数据分析和可视化场景：

### 6.1 业务分析

帮助企业分析销售数据、用户行为等，为决策提供数据支持。销售和市场部门可以通过 DataLens 快速搭建数据看板，监控关键业务指标（如销售额、用户增长）。

### 6.2 运营监控

IT 运维部门可以实时追踪系统性能，识别并预警潜在问题。

### 6.3 数据报告与研究

研究人员或咨询师可以使用 DataLens 创建专业的报告，清晰呈现研究发现。

### 6.4 教育与培训

教学环境中，教师可以利用 DataLens 展示数据科学概念，帮助学生理解和实践数据分析。

## 七、优势与价值主张

DataLens 作为一个开源数据分析平台，具有以下核心优势：

1. **开源免费**：完全开源，用户可以自由使用、修改和分发
2. **易于部署**：通过 Docker 和 Docker Compose，用户可以快速在本地或云端部署
3. **强大的数据处理能力**：基于 PostgreSQL 和 Node.js，能够处理大规模数据集
4. **直观的用户界面**：使用 React 构建的前端界面，使用户能轻松创建和定制图表
5. **灵活的扩展性**：允许用户根据需求进行定制和扩展

## 八、总结与展望

DataLens 作为一个**现代化、可扩展的开源商业智能系统**，通过提供从数据连接到仪表盘展示的全套解决方案，极大地降低了数据分析与可视化的门槛。

随着 v2.0.0 和 v2.2.0 版本的发布，DataLens 在**架构设计、安全性和部署方式**上实现了重大突破，为开发者带来了更稳定、更安全的开源数据分析体验。其持续的版本更新和功能增强表明这是一个活跃且值得关注的开源项目。

对于寻求开源商业智能解决方案的企业和组织，DataLens 提供了一个**功能丰富、易于使用且可高度定制**的选择。无论是数据分析师、开发者还是业务决策者，都能通过 DataLens 更好地理解和利用数据，实现数据驱动的决策和创新。

##### 架构设计之道在于在不同的场景采用合适的架构设计，架构设计没有完美，只有合适。

### 在代码的路上，我们一起砥砺前行。用代码改变世界！

* **[>>> 精选 SpringBoot 应用系列 >>>](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU1MjAwODAzOA==&action=getalbum&album_id=3962295143138246661#wechat_redirect)**
* **[>>> 精选 JVM 系列 >>>](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU1MjAwODAzOA==&action=getalbum&album_id=3921799178770104328#wechat_redirect)**
* **[>>> 精选排序算法系列 >>>](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU1MjAwODAzOA==&action=getalbum&album_id=3940434490433912837#wechat_redirect)**
* **[>>> 镜像常见数据库优化/SQL优化系列 >>>](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU1MjAwODAzOA==&action=getalbum&album_id=3928562202449199110#wechat_redirect)**
* **[>>> 常用中间件部署与使用 >>>](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU1MjAwODAzOA==&action=getalbum&album_id=3858009968628383745#wechat_redirect)**
* **[>>> 编程技巧与原理解析 >>>](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU1MjAwODAzOA==&action=getalbum&album_id=3863770327008067586#wechat_redirect)**
* **[>>> 开源项目推荐 >>>](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU1MjAwODAzOA==&action=getalbum&album_id=3985540521048326148#wechat_redirect)**
* **[>>> 设计模式系列 >>>](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU1MjAwODAzOA==&action=getalbum&album_id=3990993257143386124#wechat_redirect)**
* **[>>> 达梦数据库合集 >>](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU1MjAwODAzOA==&action=getalbum&album_id=4028920795467268101#wechat_redirect)**
* **[>>> 高频面试合集 >>>](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU1MjAwODAzOA==&action=getalbum&album_id=4047876008559427596#wechat_redirect)**
* **[普通学历程序员逆袭之旅 | 如今应届生该不该选择程序员？](https://mp.weixin.qq.com/s?__biz=MzU1MjAwODAzOA==&mid=2247484450&idx=1&sn=8921bff01c23e4bdf39ef5b6ab17af22&scene=21#wechat_redirect)**
* **[程序员年薪 25 万 到 35 万到底是什么水平？](https://mp.weixin.qq.com/s?__biz=MzU1MjAwODAzOA==&mid=2247484427&idx=1&sn=a0cdba15dc765b65601d85b78f4799f4&scene=21#wechat_redirect)**
* **[DeepSeek杀入编程圈：程序员集体失业？还是效率革命？](https://mp.weixin.qq.com/s?__biz=MzU1MjAwODAzOA==&mid=2247484129&idx=1&sn=61c90e68610601abd47a80c9e53cf7fa&scene=21#wechat_redirect)**
* **[深入剖析 Java 精度丢失：从二进制存储到浮点运算的底层逻辑](https://mp.weixin.qq.com/s?__biz=MzU1MjAwODAzOA==&mid=2247484606&idx=1&sn=773b9cbcbf9a004e4d6d3eaaf55209a6&scene=21#wechat_redirect)**
* **[别再用 JWT 当系统 Token 了，后果很严重！](https://mp.weixin.qq.com/s?__biz=MzU1MjAwODAzOA==&mid=2247484575&idx=1&sn=5aef18b7f65cd7971f66cb3455950560&scene=21#wechat_redirect)**
* **[SpringBoot 轻量级一站式实现日志可视化与JVM监控](https://mp.weixin.qq.com/s?__biz=MzU1MjAwODAzOA==&mid=2247485200&idx=1&sn=50a2bbe4f9bfa8252911ea4a43ea7513&scene=21#wechat_redirect)**
* **[蚂蚁又开源了一个顶级 Java 项目！](https://mp.weixin.qq.com/s?__biz=MzU1MjAwODAzOA==&mid=2247485185&idx=1&sn=d7b311e32aec54a5febe6125455f597c&scene=21#wechat_redirect)**

如果有其它问题，欢迎评论区沟通。

感谢观看，如果觉得对您有用，还请动动您那发财的手指头，点赞、转发、在看、收藏

关注公众号🔽🔽🔽🔽🔽🔽🔽🔽

# **工作 3 年还在写 CRUD？**

# **工作3-5年如何突破技术瓶颈？**

# **想转技术管理但不会带团队？**

# **想跳槽没有面试的机会？**

# **不懂如何面试，迟迟拿不到 offer？** **面试屡屡碰壁，失败原因无人指导？**

***在竞争激烈的大环境下，只有不断提升核心竞争力才能立于不败之地。***

***扫码添加备注【*我要晋级*】加入开发技术面试交流群。一对一指导，带你晋级。***

—— 斩获心仪Offer，破解面试密码 ——