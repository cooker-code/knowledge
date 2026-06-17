---
title: Python之FastAPI的入门到精通系列：fastapi在工程里的代码配置和启用
author: 猫咪不吃愚
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNDY1NzE0NQ==&mid=2247486924&idx=1&sn=6e1bb5012f542152d7ce7ee186fb18ee&chksm=c3f5fdc9d25f84971bc43ab5ad0bb01408b23195a99eb5ab318753469f48a104a24473ff1ced&mpshare=1&scene=24&srcid=1224RFRe6kVT9rZ8NcolPFng&sharer_shareinfo=02e616a257f1285f5ca87d3c6b8b4dfd&sharer_shareinfo_first=02e616a257f1285f5ca87d3c6b8b4dfd#rd
---

> 字数 1136，阅读大约需 6 分钟

# Python之FastAPI的入门到精通系列：fastapi架构和运行

  
代码地址：`git@github.com:FunkyGod/fastapi-demo.git`

## 运行方式

**在本地开发，容器化部署**，本文主要快速介绍FastAPI框架在工程应用目录和配置，**以后会根据具体开发任务和知识点，展开介绍**。

### 前提准备

在本地安装`docker、docker-compose、postgres`

### 安装依赖

推荐使用UV包管理：`python -m uv sync`

### 镜像构建和运行

构建镜像，服务在容器里执行：`docker-compose build`  
运行容器：`docker-compose up -d`

## 项目架构

本项目是一个基于 FastAPI 构建的现代化 Python Web 服务，具有清晰、分层的项目结构，旨在实现高内聚、低耦合的设计目标。

### 技术栈

* • **Web 框架**: FastAPI[1] - 一个现代、高性能的 Python Web 框架，用于构建 API。
* • **数据库 ORM**: SQLAlchemy[2] - 强大的 SQL 工具包和对象关系映射器。
* • **数据库迁移**: Alembic[3] - 一个轻量级的数据库迁移工具，与 SQLAlchemy 配合使用。
* • **数据校验**: Pydantic[4] - 基于 Python 类型提示进行数据验证和设置管理。
* • **容器化**: Docker[5] & Docker Compose[6] - 用于构建、部署和运行应用。
* • **分布式追踪**: OpenTelemetry[7] - 用于生成和收集遥测数据（Traces），实现可观测性。
* • **依赖管理**: UV[8] - 一个极速的 Python 包安装器和解析器。

### 架构图

```
数据库

核心应用 (app/)

跨域/认证/追踪

路由转发

增删改查

数据校验

ORM操作

SQLAlchemy

客户端请求

中间件

API 层 - api/

业务逻辑层

数据访问层 - crud/

数据模型层 - schema/

数据库ORM模型 - model/

数据库 - db/
```

### 目录结构解析

* • `alembic/`: 存放 Alembic 数据库迁移脚本。
* • `app/`: 应用核心代码目录。

+ • `api/`: API 路由定义，按版本（如 `v1`）组织。

- • `deps.py`: 定义 FastAPI 的依赖注入项。
- • `endpoint/`: 存放各个模块的 API 端点。

+ • `core/`: 存放应用的核心配置，如从环境变量加载的设置。
+ • `crud/`: 包含与数据库交互的增删改查（CRUD）函数。
+ • `db/`: 数据库连接、会话管理和基础模型定义。
+ • `model/`: SQLAlchemy 的数据表模型。
+ • `schema/`: Pydantic 的数据校验模型，用于 API 的请求和响应。
+ • `utility/`: 存放各类工具函数，如日志、链路追踪等。

* • `middleware/`: 自定义中间件，如认证、追踪等。
* • `script/`: 存放各类辅助脚本，如数据库初始化、服务启动脚本等。
* • `docker-compose.yml`: Docker Compose 配置文件，用于编排服务。
* • `pyproject.toml`: 项目配置文件，包含依赖列表和工具配置。

## 启动FastAPI项目

```
$ docker-compose up   
time="2025-10-30T23:07:55+08:00" level=warning msg="D:\\Code\\fastapi-demo\\docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion"  
[+] Running 1/1  
 ✔ Container fastapi-demo-fastapi-demo-1  Recreated                                                                                                                                                                                                                                                       0.2s   
Attaching to fastapi-demo-1  
fastapi-demo-1  | INFO:__main__:Initializing service  
fastapi-demo-1  | INFO:__main__:Starting call to '__main__.init', this is the 1st time calling it.  
fastapi-demo-1  | INFO:__main__:Service finished initializing  
fastapi-demo-1  | INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.  
fastapi-demo-1  | INFO  [alembic.runtime.migration] Will assume transactional DDL.  
fastapi-demo-1  | INFO:__main__:Creating initial data  
fastapi-demo-1  | INFO:__main__:Initial data created  
fastapi-demo-1  | INFO:     Will watch for changes in these directories: ['/service']  
fastapi-demo-1  | INFO:     Uvicorn running on http://0.0.0.0:8989 (Press CTRL+C to quit)  
fastapi-demo-1  | INFO:     Started reloader process [1] using WatchFiles  
fastapi-demo-1  | 2025-10-30 23:07:59.143 | INFO     | uvicorn.server:_serve:84 - Started server process [11]  
fastapi-demo-1  | 2025-10-30 23:07:59.143 | INFO     | uvicorn.lifespan.on:startup:48 - Waiting for application startup.  
fastapi-demo-1  | 2025-10-30 23:07:59.143 | INFO     | uvicorn.lifespan.on:startup:62 - Application startup complete.
```

## swagger

通过`http://localhost:9100/docs#`，打开地址成功，用于调试和管理API，不过推荐使用apifox, 以后会介绍。  

## 检查DB表

可以看到fastapi连接到postgres的数据库，且fastapi\_demo\_db表初始化成功  

---

## 号外号外

刚刷到的朋友注意啦！点击【关注】锁定宝藏库，从此升职加薪不迷路 ✨  
若觉得内容有用，长按点赞3秒引爆爱心特效！你的每次互动，都是我深夜码字的星光 🌟

**【三步操作，终身受益】**  
✅ 点击「**关注**」→ 持续收获成长能量  
✅ 点亮「**点赞**」→ 为干货内容打call  
✅ 设为「**星标**」⭐️→ 算法优先推送，更新不错过

---

**📢 云资源限时福利**  
有云服务器、CDN、对象存储、网络防护等需求的朋友，欢迎联系下方腾讯云官方销售 👇  
✔️ 内部专属折扣，价格更优  
✔️ 量大可谈，支持定制方案  
✔️ 技术咨询与售后无忧

#### 引用链接

`[1]` FastAPI: *https://fastapi.tiangolo.com/*  
`[2]` SQLAlchemy: *https://www.sqlalchemy.org/*  
`[3]` Alembic: *https://alembic.sqlalchemy.org/en/latest/*  
`[4]` Pydantic: *https://docs.pydantic.dev/latest/*  
`[5]` Docker: *https://www.docker.com/*  
`[6]` Docker Compose: *https://docs.docker.com/compose/*  
`[7]` OpenTelemetry: *https://opentelemetry.io/*  
`[8]` UV: *https://github.com/astral-sh/uv*