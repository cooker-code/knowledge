---
title: 一份FastAPI的必备类库清单，技术选型必备
author: 飞哥专栏
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxNjk5NjI1Mg==&mid=2247484209&idx=1&sn=1cb3b7da7a27c1386bb8329f92d3e58f&chksm=9a9b1c063b2d45117a7027b9d92530013ef8469a6e5dd0df76a782b98b7c89983ee6614b2403&mpshare=1&scene=24&srcid=1031j3gfhVaZhulDAlxmTG43&sharer_shareinfo=af6a7d7068736bafb7bebfc64a229e5b&sharer_shareinfo_first=af6a7d7068736bafb7bebfc64a229e5b#rd
---

作为Python的异步开发框架，FastAPI和老大哥Django各分秋色，在 AI 的流行下，FastAPI在Github已超过91.2K，飞哥用FastAPI实现各种 API 服务上面，性能高，开发速度快。

PDF智转（www.pdfai.cn） 就是用的FastAPI搭建了后端API服务。虽然快，但Python类库多，但在技术选型上面要花不少时间，整理如下最佳实践。

### 技术选型

* 轻量，选择合适的搭配，选择最近的稳定版本
* 很多 AI 编程工具自动生成的FastAPI版本太老，强力建议用 fastapi==0.116.2
* 部署必须用uvicorn，配合Supervisor更好
* 任务调度 `apscheduler`，轻量简洁。
* 数据库 ORM sqlmodel
* 队列 `rq`，简单好用。这里飞哥没有推荐`celery`和`flower`，如果你必须用得上
* 一些必备的类库：`pydash`、`funcy`
* 文件管理：阿里云OSS
* API请求：`httpx`、`aiohttp`
* 命令行：typer

以下是`requirements.txt`参考，目前各版本的时间截至2025年10月27日

```
# FastAPI项目依赖，python 3.12

fastapi==0.116.2

uvicorn[standard]==0.32.0

pydantic-settings==2.6.1

httpx==0.28.1

sqlmodel==0.0.22

pymysql==1.1.1

cryptography==43.0.3

redis==5.2.0

python-multipart==0.0.12

python-jose[cryptography]==3.3.0

passlib[bcrypt]==1.7.4

slowapi==0.1.9

aiofiles==23.1.0

typer==0.12.5

qrcode[pil]==7.4.2

oss2==2.18.3

apscheduler==3.11.0

pydash==8.0.5

funcy==2.0

aiohttp==3.12.15

rq==2.6.0

numpy==1.26.4
```

另外给一份项目目录参考

```
fastapi-app/  
├── app/  
│   ├── __init__.py  
│   ├── main.py                 # FastAPI应用入口  
│   ├── config.py              # 配置管理  
│   ├── database.py            # 数据库连接  
│   ├── redis_client.py        # Redis客户端  
│   ├── docs.py                # 文档配置，docs、redoc、swagger  
│   ├── middleware/            # 中间件  
│   │   ├── __init__.py  
│   │   ├── rate_limit.py      # 限流中间件  
│   │   ├── cors.py            # CORS中间件  
│   │   └── logging.py         # 日志中间件  
│   │   └── allow_ips.py       # 允许IP中间件  
│   ├── models/                # 数据模型  
│   │   ├── __init__.py  
│   │   ├── api_log.py         # API请求日志模型  
│   │   ├── app_code.py        # 应用代码字典（状态码/业务码），用在中间件 HeaderJWTMiddleware 进行header校验  
│   │   ├── app_config.py      # 应用配置项模型  
│   │   ├── task.py            # 异步任务模型  
│   │   └── user.py            # 用户模型  
│   ├── schemas/               # Pydantic模式  
│   │   ├── __init__.py  
│   │   └── response.py  
│   ├── api/                   # API路由  
│   │   ├── __init__.py  
│   │   └── v1/                # API版本1  
│   │       ├── __init__.py  
│   │       └── api.py         # API路由  
│   │       └── qrcode.py         # 二维码接口  
│   ├── schemas/              # Pydantic模式  
│   │   ├── __init__.py  
│   │   ├── response.py        # 响应格式化  
│   ├── services/              # 业务逻辑  
│   │   ├── __init__.py  
│   │   ├── config_service.py # 应用配置服务  
│   │   ├── task_scheduler.py # 任务调度服务  
│   ├── tasks/                 # Celery任务  
│   │   ├── __init__.py  
│   │   └── task_registry.py # 任务注册  
│   │   └── app_data_push_task.py # 应用数据推送任务  
│   └── utils/                 # 工具函数  
│       ├── __init__.py  
│       └── security.py       # 安全相关  
│       └── oss_util.py       # OSS工具  
│       └── qrcode_util.py    # 二维码工具  
├── scripts/                   # 脚本文件  
│   ├── dev.py                # 开发启动脚本  
│   ├── start.py               # 启动脚本  
│   └── init_db.py            # 数据库初始化  
├── tests/                     # 测试文件  
│   ├── __init__.py  
├── .env.example              # 环境变量示例  
├── .env                      # 环境变量（本地）  
├── .gitignore               # Git忽略文件  
├── requirements.txt         # Python依赖  
├── requirements-dev.txt     # 开发依赖  
└── README.md               # 项目说明
```

### 总结

FastAPI虽然没有Spring Boot和Laravel的生态好，但是手动搭建轮子的乐趣也是不错的，搭建好了，后续也是超级方便。有了这份类库参考，那效率必须的。