---
title: 数据可视化工具Datart
author: 早起的码农
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI2ODYzMjkwMg==&mid=2247484538&idx=2&sn=b7c8570ec2c4f807649af387860031e2&chksm=eaedd253dd9a5b4584bc6b79b9ec6b7c989aec68a5609876e6241c35e270c35e771187b7e8eb&mpshare=1&scene=24&srcid=1222X9Rdc4pqIYE6JkNawzDg&sharer_shareinfo=da3ff787f3e7e7c5ce1da95acad979ae&sharer_shareinfo_first=da3ff787f3e7e7c5ce1da95acad979ae#rd
---

**Superset ,Granfan, Davinci**这几款开源的数据可视化BI报表工具想必老程序员并不陌生，今天我们来了解一款比较新的好用的可视化工具**Datart**。

### Datart 是新一代数据可视化开放平台，支持各类企业数据可视化场景需求，如创建和使用报表、仪表板和大屏，进行可视化数据分析，构建可视化数据应用等。由原 Davinci 主创团队出品，Datart 更加开放、可塑和智能，并在数据与艺术之间寻求最佳平衡。

### **设计理念 Design Philosophy**

* **开放 Openness**
* **流程标准化**：基于 Source > View > Chart > Visualization 建立 受管控的数据可视化应用 （Managed VizApp）开发、发布和使用的标准化流程
* **交互标准化**：Visualization 支持权限可控的标准化交互能力，如筛选、钻取、联动、跳转、弹窗、分享、下载、发送等
* **插件标准化**：在 Source、Chart、Visualization 层提供标准化可插拔扩展接口或SDK规范，支持开放扩展或按需定制
* **可塑 Integrability**

### **#****01** **本地****部署**

**1.1 软件要求**

* JDK 1.8+
* MySql5.7+
* datart 安装包（datart-server-\*-install.zip)
* Chrome 和 WebDriver （可选）
* Redis （可选）

**1.2 安装**

github 上下载了最新releas版本1.0.0-rc.3 Release

*mkdir datart*

*unzip datart-server-1.0.0-rc.3-install.zip  -d  ./datart*

*chmod -R 755 datart*

/etc/profile配置DATART\_HOME环境变量，并把${DATART\_HOME}/bin追加到PATH

*. /etc/profile*

*datart-server.sh start       # 启动*

*datart-server.sh stop        # 停止*

*datart-server.sh status      # 查看状态*

*datart-server.sh restart     # 重启*

**1.3 应用数据库配置**

      直接启动时使用的是内置的 H2 数据库作为应用数据库，升级应用时无法迁移数据，不建议在生产环境使用.

      启动之后通过 http://127.0.0.1:8080 地址访问应用主页，内置初始账户，用户名 demo 密码 123456

datart 目前支持配置 MySQL 作为应用数据库；需要 MySQL 5.7 及以上版本。

*mysql> CREATE DATABASE `datart` CHARACTER SET 'utf8' COLLATE 'utf8\_general\_ci';*

编辑 config/datart.conf 文件完成配置

*datasource.ip=localhost           # 数据库IP或域名*

*datasource.port=3306              # 数据库端口*

*datasource.database=datart        # 数据库名称*

*datasource.username=datart    # 用户名*

*datasource.password=12345678     # 密码*

首次启动连接会执行数据库初始化脚本,完成表结构创建，访问http://127.0.0.1:8080 会出现如下界面：

**1.4 配置文件含义**

* datart.conf 为快捷配置文件；如果你只想快速体验 datart 的功能，配置它就足够了。datart.conf 本质上是 application-config.yml 中常用配置的快捷方式
* jdbc-driver-ext.yml 为 JDBC 数据源扩展文件；详细内容请参考数据源和扩展 JDBC 数据源
* logback.xml 为日志配置文件
* application-config.yml 为应用配置文件，包含所有的应用配置。建议将所有的应用配置文件拷贝都放置到 profiles 目录下

### **1.5 缓存（Redis）配置**

配置了 Redis 之后，可以在数据视图的高级配置中中开启缓存，减轻数据库压力

*spring:*

*redis:*

*port: 6379*

*host: { HOST }*

**#****02** **立即试****用**

**2.1 创建数据源**

**2.2 新建表视图**

可以选择开启缓存

**2.3 新建仪表板**

编辑进入仪表板编辑，选择新建数据图表

编辑数据图表

添加查询下拉框等控制器

点击保存发布即可

更多信息请猛击：https://running-elephant.github.io/datart-docs/docs/。