> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: fastapi中加载环境变量
author: 测开工程师的烦恼
date:
url: https://mp.weixin.qq.com/s?__biz=MzA5NzM0NjU1Nw==&mid=2647871462&idx=1&sn=64c71196282d9efd0cbbcb10a9defaba&chksm=8927d8dc9ca9314daea6674fd37487e22cfd1ddcb7ea399778bbd4d26ef8821d1e280983dbc5&mpshare=1&scene=24&srcid=12114RtWKLDBE78nb7mib5QG&sharer_shareinfo=837e12fd87f3738e2ad7bf71abe6c6fc&sharer_shareinfo_first=837e12fd87f3738e2ad7bf71abe6c6fc#rd
---

## 导读

在实际开发过程中，我们经常需要根据不同的环境（开发、测试、生产）使用不同的配置，比如数据库连接、Redis地址、API密钥等。

将这些配置硬编码在代码中显然不是一个好的做法，不仅不利于维护，还会带来安全风险。

在 `FastAPI`项目中，我们可以通过环境变量来管理这些配置，实现配置与代码的分离，提高项目的可维护性和安全性。

## 思路

在 `FastAPI`中加载环境变量主要有以下几种方式：

1. **使用 `pydantic-settings`**（推荐）：这是 `FastAPI`官方推荐的方式，基于 `Pydantic`的 `BaseSettings`，支持类型验证、自动加载 `.env`文件等特性。
2. **使用 `python-dotenv`**：通过 `python-dotenv`库手动加载 `.env`文件，然后使用 `os.getenv()`获取环境变量。
3. **直接使用 `os.getenv()`**：直接从系统环境变量中读取，适合简单的场景。

本文主要介绍使用 `pydantic-settings`的方式，这是最推荐的做法。

## 方案

### 1. 安装依赖

首先确保安装了必要的 Python 包：

```
pip install pydantic-settings
```

### 2. 创建环境变量文件

在项目根目录下，创建不同环境的 `.env`文件：

**`.env.dev`**（开发环境）：

```
APP_ENV=development
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
REDIS_HOST=192.168.201.200
REDIS_PORT=6379
REDIS_PASSWORD=your_password
REDIS_DB=9
API_KEY=dev_api_key
DEBUG=True
```

**`.env.production`**（生产环境）：

```
APP_ENV=production
DATABASE_URL=postgresql://user:password@prod-db:5432/mydb
REDIS_HOST=prod-redis.example.com
REDIS_PORT=6379
REDIS_PASSWORD=prod_password
REDIS_DB=0
API_KEY=prod_api_key
DEBUG=False
```

**`.env.test`**（测试环境）：

```
APP_ENV=test
DATABASE_URL=postgresql://user:password@test-db:5432/mydb
REDIS_HOST=test-redis.example.com
REDIS_PORT=6379
REDIS_PASSWORD=test_password
REDIS_DB=1
API_KEY=test_api_key
DEBUG=True
```

### 3. 创建配置类

在 `app/core/config.py`文件中，创建配置类：

```
import os
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    # 根据环境变量 APP_ENV 选择不同的 .env 文件
    if os.getenv("APP_ENV") == "development"ornot os.getenv("APP_ENV"):
        model_config = SettingsConfigDict(
            env_file=".env.dev", 
            env_ignore_empty=True, 
            extra="ignore"
        )
    elif os.getenv("APP_ENV") == "production":
        model_config = SettingsConfigDict(
            env_file=".env.production", 
            env_ignore_empty=True, 
            extra="ignore"
        )
    elif os.getenv("APP_ENV") == "test":
        model_config = SettingsConfigDict(
            env_file=".env.test", 
            env_ignore_empty=True, 
            extra="ignore"
        )
    else:
        # 默认使用 .env.dev
        model_config = SettingsConfigDict(
            env_file=".env.dev", 
            env_ignore_empty=True, 
            extra="ignore"
        )

    # 项目配置
    PROJECT_NAME: str = "My FastAPI Project"
    APP_ENV: str = "development"
    DEBUG: bool = False
    
    # 数据库配置
    DATABASE_URL: str
    
    # Redis 配置
    REDIS_HOST: str
    REDIS_PORT: int
    REDIS_PASSWORD: str
    REDIS_DB: int
    
    # API 配置
    API_KEY: str
    
    # 其他配置
    SECRET_KEY: str = "your-secret-key-here"
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30


# 创建全局配置实例
settings = Settings()
```

### 4. 在项目中使用配置

在 `app/main.py`或其他地方使用配置：

```
from fastapi import FastAPI
from app.core.config import settings

app = FastAPI(
    title=settings.PROJECT_NAME,
    debug=settings.DEBUG,
)

@app.get("/")
asyncdef root():
    return {
        "message": "Hello World",
        "environment": settings.APP_ENV,
        "debug": settings.DEBUG
    }
```

### 5. 使用配置连接数据库或 Redis

```
from app.core.config import settings
import redis

# 使用 Redis 配置
redis_client = redis.Redis(
    host=settings.REDIS_HOST,
    port=settings.REDIS_PORT,
    password=settings.REDIS_PASSWORD,
    db=settings.REDIS_DB,
    decode_responses=True
)

# 使用数据库配置
from sqlalchemy import create_engine
engine = create_engine(settings.DATABASE_URL)
```

### 6. 配置说明

**`SettingsConfigDict`参数说明：**

* `env_file`: 指定要加载的环境变量文件路径
* `env_ignore_empty`: 忽略空值，如果环境变量值为空字符串，则使用默认值
* `extra`: 控制如何处理未定义的字段，`"ignore"`表示忽略未定义的字段

**环境变量优先级：**

1. 系统环境变量（最高优先级）
2. `.env`文件中定义的值
3. 类中定义的默认值（最低优先级）

### 7. 注意事项

1. **不要将 `.env`文件提交到版本控制系统**：在 `.gitignore`中添加：

   ```
   .env*
   !.env.example
   ```
2. **使用 `.env.example`作为模板**：创建一个 `.env.example`文件，包含所有需要的环境变量（不包含敏感信息），供其他开发者参考。
3. **生产环境配置**：在生产环境中，建议通过系统环境变量或容器环境变量来设置配置，而不是使用 `.env`文件。
4. **类型验证**：`Pydantic`会自动进行类型验证，如果环境变量的类型不匹配，会抛出验证错误。

## 总结

* 使用 `pydantic-settings`可以方便地管理环境变量，支持类型验证和自动加载。
* 通过不同环境的 `.env`文件，可以轻松切换配置。
* 配置与代码分离，提高了项目的可维护性和安全性。
* 环境变量优先级：系统环境变量 > `.env`文件 > 默认值。

---

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。