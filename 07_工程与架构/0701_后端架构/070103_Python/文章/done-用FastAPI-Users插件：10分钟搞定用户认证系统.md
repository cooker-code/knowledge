> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: 用FastAPI-Users插件：10分钟搞定用户认证系统
author: 测开工程师的烦恼
date: 洛水之枫洛水之枫
url: https://mp.weixin.qq.com/s?__biz=MzA5NzM0NjU1Nw==&mid=2647871953&idx=1&sn=003bcfdea38ccab4ec392cc604137d33&chksm=894cb7e14bd8cf48ba6c3592dcf1faa13aede15a5638c9143e106d0373b7c797fca82303a9e2&mpshare=1&scene=24&srcid=0501Aii4MGZAAtamr7ldvcej&sharer_shareinfo=181cd9f33177677d121a4fd32a207cbf&sharer_shareinfo_first=181cd9f33177677d121a4fd32a207cbf#rd
---

## 导读

上一篇讲了网关: [用FastAPI开发一个网关服务：从0到1构建API统一入口](https://mp.weixin.qq.com/s?__biz=MzA5NzM0NjU1Nw==&mid=2647871946&idx=1&sn=03d752feca8ff3694c380218da6a63ed&scene=21#wechat_redirect)

网关后面会路由很多个业务服务，大多数业务服务也需要用户信息，比如查看公司哪位同学最近创建了压测计划，哪位同学执行了接口自动化任务等，这个时候需要一个统一认证的服务来提供用户相关业务，接入企微认证，飞连等第三方认证。

本文将手把手教你用 FastAPI-Users 插件，**10分钟搭出一套完整的用户认证系统**，从此告别手写登录注册的痛苦。

本文搭建用户框架，企微认证登录，第三方登录在下篇文章中分享。

## 一、FastAPI-Users 是什么？

说白了，FastAPI-Users 是一个**开箱即用的用户认证插件**，帮你把注册、登录、密码重置、邮箱验证这些繁琐功能都封装好了。

它的核心能力：

* ✅ 用户注册（自动密码哈希）
* ✅ JWT 登录（生成 token）
* ✅ 路由保护（登录才能访问）
* ✅ 权限控制（区分普通用户/管理员）
* ✅ 密码重置（邮件找回）

如果你自己从零写这些功能，少说也得一两天。用 FastAPI-Users，几行代码就搞定。

## 二、项目搭建

### 项目结构

```
fastapi_users_demo/
├── main.py              # 入口文件
├── models.py            # 用户模型
├── schemas.py           # Pydantic 模型
├── auth.py              # 认证配置
├── db.py                # 数据库连接
└── requirements.txt     # 依赖
```

### 安装依赖

```
# requirements.txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
fastapi-users[sqlalchemy]==12.1.2
sqlalchemy[asyncio]==2.0.23
aiosqlite==0.19.0
python-dotenv==1.0.0
```

```
pip install -r requirements.txt
```

> **注意**：`fastapi-users[sqlalchemy]` 包含了 SQLAlchemy 支持，如果不用 ORM 可以用其他后端。

## 三、核心代码实现

### 1. 用户模型（models.py）

FastAPI-Users 要求用户模型继承它的基类，这样它才能识别哪些字段是做什么的。

```
from fastapi_users.db import SQLAlchemyBaseUserTableUUID
from sqlalchemy.ext.asyncio import AsyncAttrs
from sqlalchemy.orm import DeclarativeBase

class Base(DeclarativeBase, AsyncAttrs):
    """SQLAlchemy 基类，支持异步"""
    pass

class User(SQLAlchemyBaseUserTableUUID, Base):
    """
    用户模型
    
    继承 SQLAlchemyBaseUserTableUUID，自动获得：
    - id: UUID 主键
    - email: 邮箱（唯一）
    - hashed_password: 哈希后的密码
    - is_active: 是否激活
    - is_superuser: 是否超级管理员
    - is_verified: 是否已验证邮箱
    """
    pass
```

**为什么用这个基类？**

因为它已经帮你定义好了标准字段：

* `email`：登录账号
* `hashed_password`：密码（自动 bcrypt 哈希）
* `is_active`：账号是否可用
* `is_superuser`：是否管理员
* `is_verified`：邮箱是否验证过

### 2. 数据库配置（db.py）

```
from sqlalchemy.ext.asyncio import async_sessionmaker, create_async_engine
from sqlalchemy.pool import NullPool
from models import Base

# 使用 SQLite 演示（生产环境换 PostgreSQL）
DATABASE_URL = "sqlite+aiosqlite:///./test.db"

engine = create_async_engine(
    DATABASE_URL,
    poolclass=NullPool,  # SQLite 用 NullPool 避免并发问题
)

# 创建会话工厂
async_session_maker = async_sessionmaker(engine, expire_on_commit=False)

async def create_db_and_tables():
    """启动时创建表"""
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

async def get_async_session():
    """获取数据库会话（依赖注入用）"""
    async with async_session_maker() as session:
        yield session
```

### 3. Pydantic 模型（schemas.py）

```
from fastapi_users import schemas
from pydantic import ConfigDict
from uuid import UUID

class UserRead(schemas.BaseUser[UUID]):
    """
    用户响应模型（返回给前端）
    
    继承 BaseUser，自动包含：
    - id, email, is_active, is_superuser, is_verified
    """
    model_config = ConfigDict(from_attributes=True)

class UserCreate(schemas.BaseUserCreate):
    """
    用户创建模型（注册时用）
    
    继承 BaseUserCreate，要求必须提供：
    - email: 邮箱
    - password: 密码
    """
    pass

class UserUpdate(schemas.BaseUserUpdate):
    """
    用户更新模型（修改信息用）
    
    所有字段都是可选的，可以只更新部分信息
    """
    pass
```

### 4. 认证配置（auth.py）

这是核心文件，把 FastAPI-Users 的各个组件组装起来。

```
from fastapi_users import FastAPIUsers
from fastapi_users.authentication import (
    AuthenticationBackend,
    BearerTransport,
    JWTStrategy,
)
from fastapi_users.db import SQLAlchemyUserDatabase
from sqlalchemy.ext.asyncio import AsyncSession
from models import User
from db import get_async_session

# 1. 配置 JWT
SECRET = "YOUR-SECRET-KEY-CHANGE-IN-PRODUCTION"  # 生产环境放环境变量！

def get_jwt_strategy() -> JWTStrategy:
    """JWT 策略：token 30 天过期"""
    return JWTStrategy(secret=SECRET, lifetime_seconds=60 * 60 * 24 * 30)

# 2. 配置 Bearer 传输（HTTP Header 中传 token）
bearer_transport = BearerTransport(tokenUrl="auth/jwt/login")

# 3. 创建认证后端
auth_backend = AuthenticationBackend(
    name="jwt",
    transport=bearer_transport,
    get_strategy=get_jwt_strategy,
)

# 4. 创建用户数据库接口
async def get_user_db(session: AsyncSession = Depends(get_async_session)):
    """获取用户数据库操作对象"""
    yield SQLAlchemyUserDatabase(session, User)

# 5. 组装 FastAPI-Users
fastapi_users = FastAPIUsers[User, UUID](
    get_user_db,
    [auth_backend],
)

# 6. 导出常用依赖
# current_active_user：获取当前登录用户（必须已登录且账号有效）
current_active_user = fastapi_users.current_user(active=True)

# current_superuser：获取当前管理员（必须已登录且是超级管理员）
current_superuser = fastapi_users.current_user(active=True, superuser=True)
```

### 5. 入口文件（main.py）

```
from fastapi import FastAPI, Depends, HTTPException, status
from contextlib import asynccontextmanager
from db import create_db_and_tables
from models import User
from schemas import UserRead, UserCreate, UserUpdate
from auth import fastapi_users, auth_backend, current_active_user, current_superuser

@asynccontextmanager
async def lifespan(app: FastAPI):
    """应用生命周期：启动时创建表"""
    await create_db_and_tables()
    yield

app = FastAPI(
    title="FastAPI-Users 演示",
    description="10分钟搞定用户认证系统",
    lifespan=lifespan,
)

# ========== 1. 认证路由（注册/登录/登出） ==========
# FastAPI-Users 自动生成以下接口：
# POST /auth/register    注册
# POST /auth/jwt/login   登录（返回 token）
# POST /auth/jwt/logout  登出
app.include_router(
    fastapi_users.get_auth_router(auth_backend),
    prefix="/auth/jwt",
    tags=["auth"],
)

app.include_router(
    fastapi_users.get_register_router(UserRead, UserCreate),
    prefix="/auth",
    tags=["auth"],
)

# ========== 2. 用户管理路由（CRUD） ==========
# 自动生成：
# GET    /users/me          获取当前用户
# PATCH  /users/me          更新当前用户
# GET    /users/{id}        获取指定用户（管理员）
# PATCH  /users/{id}        更新指定用户（管理员）
# DELETE /users/{id}        删除指定用户（管理员）
app.include_router(
    fastapi_users.get_users_router(UserRead, UserUpdate),
    prefix="/users",
    tags=["users"],
)

# ========== 3. 受保护的业务接口 ==========
@app.get("/protected", summary="普通用户可访问")
async def protected_route(user: User = Depends(current_active_user)):
    """
    登录后才能访问的接口
    
    - 自动验证 token
    - 自动检查用户是否激活
    - 返回当前用户信息
    """
    return {
        "message": f"你好，{user.email}！",
        "user_id": str(user.id),
        "is_superuser": user.is_superuser,
    }

@app.get("/admin-only", summary="仅管理员可访问")
async def admin_route(user: User = Depends(current_superuser)):
    """
    只有超级管理员才能访问
    
    - 自动验证 token
    - 自动检查是否是超级管理员
    - 不是管理员会返回 403
    """
    return {
        "message": "管理员专属区域",
        "admin_email": user.email,
    }

@app.get("/public", summary="公开接口")
async def public_route():
    """无需登录，谁都能访问"""
    return {"message": "这是公开信息，无需登录"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=True)
```

## 四、测试验证

### 1. 启动服务

```
python main.py
```

访问 http://127.0.0.1:8000/docs[1] 查看自动生成的 Swagger 文档。

### 2. 注册用户

```
curl -X POST "http://127.0.0.1:8000/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "strongpassword123"
  }'
```

返回：

```
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "test@example.com",
  "is_active": true,
  "is_superuser": false,
  "is_verified": false
}
```

### 3. 登录获取 Token

```
curl -X POST "http://127.0.0.1:8000/auth/jwt/login" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=test@example.com&password=strongpassword123"
```

返回：

```
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "token_type": "bearer"
}
```

### 4. 访问受保护接口

```
curl "http://127.0.0.1:8000/protected" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
```

返回：

```
{
  "message": "你好，test@example.com！",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "is_superuser": false
}
```

### 5. 测试管理员接口（会被拒绝）

```
curl "http://127.0.0.1:8000/admin-only" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1Qi..."
```

返回：

```
{
  "detail": "Forbidden"
}
```

> 因为我们注册的是普通用户，不是管理员。

## 五、进阶：自定义用户字段

默认的用户模型只有基础字段，实际项目中你需要扩展，比如加昵称、头像、手机号等。

### 修改用户模型

```
from fastapi_users.db import SQLAlchemyBaseUserTableUUID
from sqlalchemy import String, Boolean
from sqlalchemy.orm import Mapped, mapped_column

class User(SQLAlchemyBaseUserTableUUID, Base):
    """扩展用户模型"""
    
    # 自定义字段
    nickname: Mapped[str] = mapped_column(String(50), nullable=True)
    avatar_url: Mapped[str] = mapped_column(String(255), nullable=True)
    phone: Mapped[str] = mapped_column(String(20), nullable=True, unique=True)
    
    # 你可以加任何业务字段
    department: Mapped[str] = mapped_column(String(50), nullable=True)
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), 
        server_default=func.now()
    )
```

### 修改 Pydantic 模型

```
from typing import Optional
from pydantic import Field

class UserRead(schemas.BaseUser[UUID]):
    """用户响应模型（增加自定义字段）"""
    nickname: Optional[str] = None
    avatar_url: Optional[str] = None
    phone: Optional[str] = None
    department: Optional[str] = None
    
    model_config = ConfigDict(from_attributes=True)

class UserCreate(schemas.BaseUserCreate):
    """用户创建模型"""
    nickname: Optional[str] = Field(None, max_length=50)
    phone: Optional[str] = Field(None, max_length=20)
```

这样注册时就可以传额外字段，返回时也会带上这些信息。

## 六、进阶：多角色权限控制

除了超级管理员，你可能还需要普通用户、VIP 用户、编辑等角色。

### 方法一：基于角色的权限（推荐）

```
from fastapi import Depends, HTTPException, status
from enum import Enum

class UserRole(str, Enum):
    """用户角色枚举"""
    USER = "user"
    VIP = "vip"
    EDITOR = "editor"
    ADMIN = "admin"

# 在 User 模型中添加角色字段
class User(SQLAlchemyBaseUserTableUUID, Base):
    role: Mapped[str] = mapped_column(
        String(20), 
        default=UserRole.USER.value,
        nullable=False
    )

def require_role(*allowed_roles: str):
    """角色权限检查装饰器"""
    async def role_checker(user: User = Depends(current_active_user)):
        if user.role not in allowed_roles and not user.is_superuser:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"需要角色: {', '.join(allowed_roles)}"
            )
        return user
    return role_checker

# 使用方式
@app.get("/vip-content", summary="VIP 专属内容")
async def vip_content(
    user: User = Depends(require_role(UserRole.VIP.value, UserRole.ADMIN.value))
):
    return {"message": f"欢迎 VIP 用户 {user.email}！"}
```

### 方法二：基于 OAuth2 Scopes（更细粒度）

```
from fastapi.security import OAuth2PasswordBearer, SecurityScopes

# 定义权限范围
oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="auth/jwt/login",
    scopes={
        "users:read": "读取用户信息",
        "users:write": "修改用户信息",
        "items:read": "读取物品信息",
        "items:write": "管理物品",
        "admin": "所有权限",
    }
)

def get_current_user_with_scopes(
    security_scopes: SecurityScopes,
    token: str = Depends(oauth2_scheme)
):
    """验证用户是否有指定权限范围"""
    # 解析 token，检查 scopes
    ...

# 使用方式
@app.get("/items", summary="读取物品列表")
async def get_items(
    user: User = Security(get_current_user_with_scopes, scopes=["items:read"])
):
    return {"items": []}
```

## 总结

今天我们一起学习了 FastAPI-Users，这套**开箱即用的用户认证解决方案**：

* **FastAPI-Users 的本质**：把注册、登录、密码管理这些通用功能封装成插件，几行代码就能用
* **核心组件**：User 模型 + AuthenticationBackend + FastAPIUsers 主类
* **使用步骤**：定义模型 → 配置认证 → 注册路由 → 用 Depends 保护接口
* **进阶方向**：自定义字段扩展、多角色权限、OAuth2 Scopes 细粒度控制
* **生产注意**：密钥放环境变量、密码强度校验、邮箱验证、限流防爆破

掌握了 FastAPI-Users，你再也不用花时间写重复的用户认证代码，专注业务逻辑就好。

---

**每日踩一坑**

注册接口默认是开放的，生产环境一定要加限流（比如 5 分钟最多注册 3 次），否则会被恶意批量注册刷爆数据库。

本期分享就到这里啦，祝君在测开之路越走越远，越走越顺。