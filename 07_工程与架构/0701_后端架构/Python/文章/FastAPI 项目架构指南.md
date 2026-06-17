---
title: FastAPI 项目架构指南
author: DeepNoMind
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU2MTgxODgwNA==&mid=2247491718&idx=1&sn=71084ca329fc7fed9ef8fe6f8603ea85&chksm=fd8efc6beb4223f10917fc59ff4a747568e987b5e009d7fcadf966742331335e90034667a4d7&mpshare=1&scene=24&srcid=1223NbYmUsLj1ZXyyCFkD3Ec&sharer_shareinfo=d76886cffd2ccb049b9e2a45e17ff89c&sharer_shareinfo_first=d76886cffd2ccb049b9e2a45e17ff89c#rd
---

> *本文介绍了在 Python 项目中使用 FastAPI 构建产品的代码架构设计模式，通过良好的代码架构，可以清晰的组织代码功能，有助于开发功能良好的产品。原文：FastAPI Architecture Guide: Build Scalable and Secure Systems with This Production-Ready Template[1]*

在生产环境中运行这个架构之后，可以自信的说，该架构可以轻松扩容、很好的进行维护，并且可以保持生产力。如果想在项目中直接使用这个 FastAPI 架构，请查看 GitHub 的完整项目设置：FastAPI Project Structure[2]

# 简介

在过去一年里，我几乎每天都用 FastAPI。我在这段时间里开发了一个标准的项目架构，帮助我轻松应对变更和添加新功能，同时又能很好的扩展。本文分解该 FastAPI 架构的每个组件。无论你希望更快迭代想法，组织不断增长的代码库，还是加强应用程序安全性，此架构都将帮助你自信的构建并像专家一样发展项目。

# 项目架构概述

中间件体系架构模式，具有双向连接多个应用程序和数据库的集成平台

一个组织良好的 FastAPI 项目的魅力在于其可预测性。当每个组件都有其合理的归属位置时，就能节省寻找文件的时间，而将更多时间用于构建功能。这种架构清晰划分了各个职责：路由逻辑留在路由器中，业务逻辑存在于服务中，数据验证在模式中进行，配置则集中于专门的模块中。

# App 目录：你的应用引擎

App 目录是应用程序的引擎和核心，包含支持大多数后端基础设施用例所需的所有模块和代码。下面解释该目录中的每个子目录和模块、使用它们的原因、要避免的常见陷阱，以及为提高代码质量和开发人员体验所做的改进。

#### `__init__.py`：包声明

在每个 Python 包中放置 `__init__.py` 是标准做法，它将该文件夹声明为常规 Python 包，从而在整个应用程序中实现适当的导入。

#### `api_router.py`：中心化路由管理

这个模块作为管理应用程序中所有路由的中心化组件，有助于版本控制（如 `/v1/users`、`/v2/users`），并保持代码库清晰易读。

```
# apps/api_router.py  
  
from fastapi import APIRouter  
api_v1 = APIRouter(prefix="/v1")  
api_v2 = APIRouter(prefix="/v2")  
# include routes to a root route  
# from app.routers import routers as user_routers  
# api_v1.include_router(users_routers)
```

这种中心化方法意味着，在需要引入 API 版本控制或弃用旧端点时，有一个单一的事实来源。无需在分散的路由文件中寻找，搞不清楚哪个版本服务于哪个端点。

#### `logger.py`：生产级日志记录

日志记录是每个软件应用程序中不可或缺的一部分，能提供运行时可见性，并在出错时帮助定位和解决问题（而且这种情况肯定会发生）。与其陷入复杂配置之中，这个模板在几乎所有项目中都表现得极为出色。可以通过集成外部处理程序将日志发送到可视化仪表板或分析工具中来对其进行扩展。

该配置使用了 `TimedRotatingFileHandler` 保存每日日志，从而使得调试工作更加高效：

```
# apps/logger.py  
  
import logging  
import json  
from logging.handlers import TimedRotatingFileHandler  
logger = logging.getLogger()  
class JsonFormatter(logging.Formatter):  
    def format(self, record):  
        log_record = {  
            "timestamp": self.formatTime(record, self.datefmt),  
            "level": record.levelname,  
            "module": record.module,  
            "funcName": record.funcName,  
            "lineno": record.lineno,  
            "message": record.getMessage(),  
        }  
        return json.dumps(log_record)  
file_handler = TimedRotatingFileHandler(  
    "logs/app.log", when="midnight", interval=1 / 86400, backupCount=7  
)  
file_handler.setFormatter(JsonFormatter())  
logger.handlers = [file_handler]  
logger.setLevel(logging.INFO)
```

JSON 格式使日志可由机器读取，非常适合与日志聚合系统（如 ELK Stack 或 CloudWatch）集成。定时轮换可以在维护一周的历史数据时防止占用过多磁盘空间。

#### `main.py`：应用程序入口和安全中心

API 限速图示，当超过请求限制时，客户端请求被限速器阻止并显示 429 Too Many Requests 错误

主模块是应用程序入口，其中包含了大部分安全策略和规则。在这里，中间件会依次叠加起来，以保护 API 免受常见威胁：

```
# apps/main.py  
  
from contextlib import asynccontextmanager  
from datetime import datetime, UTC  
from fastapi import FastAPI  
from starlette.middleware.trustedhost import TrustedHostMiddleware  
from starlette.middleware.base import BaseHTTPMiddleware  
from fastapi.middleware.gzip import GZipMiddleware  
from fastapi.middleware.cors import CORSMiddleware  
from fastapi.responses import RedirectResponse, JSONResponse  
from fastapi.requests import Request  
from fastapi import HTTPException  
from slowapi import Limiter  
from slowapi.util import get_remote_address  
from app.logger import logger  
from app.api_router import api  
from app.settings import Settings  
from app.middlewares import log_request_middleware  
settings = Settings()  
@asynccontextmanager  
asyncdef lifespan(app: FastAPI):  
    yield  
def initiate_app():  
    app = FastAPI(  
        title="FastAPI Sample Project",  
        summary="API for FastAPI Sample Project",  
        lifespan=lifespan,  
    )  
    origins = [  
        # Add allowed origins here  
    ]  
    app.add_middleware(  
        CORSMiddleware,  
        allow_origins=origins,  
        allow_credentials=True,  
        allow_methods=["*"],  
        allow_headers=["*"],  
    )  
    app.add_middleware(GZipMiddleware, minimum_size=100)  
    app.add_middleware(  
        TrustedHostMiddleware,  
        allowed_hosts=[  
            # Add allowed hosts here  
        ],  
    )  
    app.add_middleware(BaseHTTPMiddleware, dispatch=log_request_middleware)  
      
    limiter = Limiter(key_func=get_remote_address)  
    app.state.limiter = limiter  
    app.include_router(api)  
    return app  
app = initiate_app()
```

CORS 中间件负责处理跨源资源共享问题，能够控制哪些域名可以访问 API。这样可以防止未经授权的网站向端点发起请求。

GZip 中间件会压缩超出指定最小尺寸的响应数据，从而降低带宽使用量，并为连接速度较慢的客户端提高响应速度。

TrustedHost 中间件能够限制哪些主机可以运行应用程序，从而防止主机头注入攻击。

速率限制可保护 API 避免遭受滥用和拒绝服务攻击，其工作原理是限制单个客户端在一定时间段内所能发出的请求数量。

异常处理程序能够在整个应用程序中提供一致的错误响应：

```
@app.exception_handler(HTTPException)  
asyncdef http_exception_handler(request: Request, exc: HTTPException):  
    return JSONResponse(  
        status_code=exc.status_code,  
        content={  
            "detail": exc.detail,  
            "path": request.url.path,  
            "timestamp": datetime.now(UTC).isoformat(),  
        },  
    )  
  
@app.exception_handler(Exception)  
asyncdef global_exception_handler(request: Request, exc: Exception):  
    logger.error(f"Unexpected error: {str(exc)}")  
    return JSONResponse(  
        status_code=500,  
        content={  
            "detail": "An unexpected error occurred",  
            "path": request.url.path,  
        },  
    )  
@app.get("/", tags=["Root"])  
asyncdef root():  
    return RedirectResponse("/docs")
```

将根目录重定向到 `/docs` 这一做法虽小却颇具价值 —— 任何访问 API 基本 URL 的用户都能立即看到交互式文档，从而使 API 更易于被发现且更便于开发者使用。

#### `settings.py`：环境配置管理

使用 Pydantic 的 `BaseSettings` 来处理环境变量非常简单且清晰。可以为环境变量定义类型，这些类型在应用程序启动时会进行验证，并为可选变量提供默认值：

```
# apps/settings.py  
  
from pydantic_settings import BaseSettings  
from pydantic import ConfigDict, AnyUrl  
class Settings(BaseSettings):  
    model_config = ConfigDict(env_file=".env")  
    example_secret: str = "example secret value"  
    JWT_SECRET: str  # required environment variable   
    JWT_ALGORITHM: str = "HS256"  # optional with default
```

这种方法能在配置错误到达生产环境之前将其捕获。如果缺少必需的环境变量，应用程序在启动时会立即失败，而不是在运行时莫名其妙地出现故障。

#### `dependencies.py`：集中式依赖管理

该模块集中管理自定义路由依赖项。像 `get_db`（用于获取数据库会话）这样的常见依赖项就在此处声明。一个单独的依赖项模块有助于保持代码组织更清晰、更模块化 —— 不同路由通常依赖于这些共享依赖项 —— 并且使得测试更容易，因为逻辑可以被隔离并进行验证。

#### `middlewares.py`：自定义中间件中心

该模块集中管理应用程序的自定义中间件。正如依赖模块所描述的那样，其模块化的优势在此同样适用。用于请求日志记录、身份验证检查或性能监控的中间件都集中在一个可预测的位置。

# 目录组织

#### 路由器目录：路由逻辑分离

我倾向于为每个逻辑路由组创建独立模块，这样便于开发且更易于操作。例如，`auth.py` 模块包含与用户认证和个人资料管理相关的所有路由，`product.py` 模块包含产品管理相关的路由，`admin.py` 模块包含所有管理员 API 路由。

在可能的情况下，我会将路由函数的代码量控制在两行以内：路由函数仅用于声明路由、定义依赖项以及指定请求参数。而每个路由的业务逻辑则存在于对应的服务函数中：

```
# apps/routers/auth.py  
  
from typing import Annotated  
from pydantic import EmailStr  
from fastapi.routing import APIRouter  
from sqlalchemy.ext.asyncio import AsyncSession  
from fastapi import Depends, Body, BackgroundTasks  
from app.dependencies import get_db  
from app.schemas import auth as auth_schemas  
from app.services import auth as auth_services  
router = APIRouter(prefix="/auth", tags=["Authentication"])  
EmailBody = Annotated[EmailStr, Body(embed=True)]  
DBDep = Annotated[AsyncSession, Depends(get_db)]  
@router.post("/signup", response_model=auth_schemas.UserModel)  
asyncdef signup(  
    db: DBDep,  
    bg_task: BackgroundTasks,  
    request_data: auth_schemas.UserSignUpData,  
):  
    returnawait auth_services.signup_user(request_data, db, bg_task)
```

这种干净的路由功能将业务逻辑委托给服务函数。路由本身仅关注依赖关系、请求体或查询参数。在调试或添加功能时，能立即知道该从何处着手 —— 路由定义了 API 接口，而服务则实现了相应功能。

#### 模式目录：数据验证层

我将所有 Pydantic 模型都放在模式模块中。与路由目录类似，模式目录包含了所有的模式模块。通常，每个路由模块对应一个模式模块和一个服务模块。这样，每个应用程序组件都能以一种逻辑且可预测的方式进行分类。

例如，`auth.py` 的模式模型：

```
# apps/schemas/auth.py  
  
from uuid import UUID  
from datetime import datetime  
from typing import Annotated  
from pydantic import BaseModel, EmailStr, Field  
class UserSignUpData(BaseModel):  
    password: Annotated[str, Field(min_length=8)]  
    email: Annotated[EmailStr, Field(max_length=254)]  
class UserModel(BaseModel):  
    id: UUID  
    email: EmailStr  
    date_created: datetime  
    date_updated: datetime
```

Pydantic 模式具备自动验证、序列化和文档生成的功能，在 API 与客户端之间充当契约，确保在应用程序边界处的数据完整性。

#### 服务目录：业务逻辑实现

服务目录存储了实际的业务逻辑模块 —— 对第三方 API 的调用、数据库查询以及复杂操作。通过这种方式进行分类，实现了路由的声明性 API 要求与具体实现之间的关注点分离。

与“路由”和“模式”目录一样，在组件层面也保持了一对一的对应关系。下面以“认证”组件为例进行说明：

```
# apps/services/auth.py  
  
from datetime import timedelta  
from fastapi import HTTPException  
from passlib.context import CryptContext  
from fastapi.security import OAuth2PasswordBearer  
from sqlalchemy import select  
from sqlalchemy.ext.asyncio import AsyncSession  
from app.settings import Settings  
from app.models import User as UserDB  
from app.schemas import auth as auth_schema  
settings = Settings()  
JWT_SECRET = settings.JWT_SECRET  
JWT_ALGORITHM = settings.JWT_ALGORITHM  
ACCESS_TOKEN_LIFESPAN = timedelta(days=2)  
REFRESH_TOKEN_LIFESPAN = timedelta(days=5)  
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")  
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="v1/auth/token")  
def verify_password(plain_password: str, hashed_password: str):  
    return pwd_context.verify(plain_password, hashed_password)  
def get_password_hash(password: str):  
    return pwd_context.hash(password)  
asyncdef get_user(email: str, session: AsyncSession) -> UserDB | None:  
    stmt = select(UserDB).where(UserDB.email == email)  
    result = await session.execute(stmt)  
    return result.scalar_one_or_none()  
asyncdef create_user(  
    user_data: auth_schema.UserSignUpData,  
    session: AsyncSession,  
):  
    result = await session.execute(  
        select(UserDB).where(UserDB.email == user_data.email)  
    )  
    if result.scalar_one_or_none():  
        raise HTTPException(status_code=400, detail="Email already registered")  
    hashed_password = get_password_hash(user_data.password)  
    new_user = UserDB(  
        email=user_data.email,  
        password=hashed_password,  
        username=user_data.email,  
    )  
    session.add(new_user)  
    await session.commit()  
    await session.refresh(new_user)  
    return new_user  
asyncdef signup_user(  
    data: auth_schema.UserSignUpData,  
    session: AsyncSession,  
):  
    returnawait create_user(data, session)
```

这种分层架构使得测试变得极为简便，可以独立于业务逻辑来测试路由验证。在测试路由时，可以模拟服务函数，并使用测试数据库来测试服务函数，而无需触及 HTTP 层。

# 开发人员生产力：Makefile 的优势

除了核心 Python 后端组件之外，强烈建议使用诸如 Makefile 这样的工具来简化常见命令行操作 —— 启动 FastAPI 服务器、运行 pytest 以及生成代码覆盖率报告：

```
# Makefile  
  
run-local:  
    fastapi dev app/main.py  
test-local:  
    pytest -s --cov  
coverage-report:  
    coverage report  
coverage-html:  
    coverage report && coverage html
```

要用 make，首先需要安装。一旦项目根目录下有上述 Makefile，启动 FastAPI 服务器就变成了：

```
make run-local
```

无需每次都输入完整命令。对于经常使用的命令，这样做能极大提高效率。此外，还能使团队成员使用的命令保持一致 —— 无论个人的环境配置如何，所有人都用相同的命令。

# 可扩展性与生产准备性

该架构已在生产环境中得到验证，能够同时处理 500 多个并发用户。关注点分离意味着可以独立扩展不同组件。需要优化数据库查询吗？专注于服务模块。想添加缓存吗？在依赖级别进行注入。需要更换身份验证提供者吗？无需修改路由，直接修改身份验证服务即可。

中间件提供了多层次防护措施：速率限制可防止滥用行为，CORS 可防止未经授权的访问，可信主机中间件可抵御注入式攻击，而全面的日志记录则在出现问题时提供监控视图。

# 总结

在该模板中，并未包含数据库连接的设置部分 —— 这会因所使用数据库系统（如 PostgreSQL、MongoDB 或其他系统）的不同而有很大差异。保持项目结构和代码库的模块化能够带来超出预期的益处：对功能和工具进行有条理、逻辑的组织，使得代码库的“外观”超越了其实际功能本身。

尽管这一模式在过去一年里极大提高了我的工作效率，但我仍在不断学习并采用更优的后端架构方法和实践。此模板的完整代码可在 GitHub 上获取：https://github.com/brianobot/fastAPI\_project\_structure[3]

该架构为项目发展提供了空间。先从小规模开始，设置几条路由和模式，然后随着应用需求的变化逐步扩展。无论是构建周末项目还是企业系统，这种架构都能提供支持。最重要的是，能让开发过程充满乐趣 —— 当我们享受在代码库中工作时，就能开发出更优质的软件。

---

> 你好，我是俞凡，在Motorola做过研发，现在在Mavenir做技术工作，对通信、网络、后端架构、云原生、DevOps、CICD、区块链、AI等技术始终保持着浓厚的兴趣，平时喜欢阅读、思考，相信持续学习、终身成长，欢迎一起交流学习。为了方便大家以后能第一时间看到文章，请朋友们关注公众号"DeepNoMind"，并设个星标吧，如果能一键三连(转发、点赞、在看)，则能给我带来更多的支持和动力，激励我持续写下去，和大家共同成长进步！

参考资料

[1] 

FastAPI Architecture Guide: Build Scalable and Secure Systems with This Production-Ready Template: *https://jinlow.medium.com/fastapi-architecture-guide-build-scalable-and-secure-systems-with-this-production-ready-template-141cece89e45*

[2] 

FastAPI Project Structure: *https://github.com/brianobot/fastAPI\_project\_structure*

[3] 

https://github.com/brianobot/fastAPI\_project\_structure: *https://github.com/brianobot/fastAPI\_project\_structure*