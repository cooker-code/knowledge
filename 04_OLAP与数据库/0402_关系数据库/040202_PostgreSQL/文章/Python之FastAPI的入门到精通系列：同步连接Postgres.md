---
title: Python之FastAPI的入门到精通系列：同步连接Postgres
author: 猫咪不吃愚
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNDY1NzE0NQ==&mid=2247486958&idx=1&sn=f44a5260f5aa64adbdcd7c34655dd1b4&chksm=c3a4410c5ed29eb478c0c4d26b0b47f2052f43df266b878b0475ce33546a31edeb8d7f913606&mpshare=1&scene=24&srcid=1108nY77GhJOk0ceoIzUyJ5E&sharer_shareinfo=79e85f7a34ebb76868a6d4d797f3c5c2&sharer_shareinfo_first=79e85f7a34ebb76868a6d4d797f3c5c2#rd
---

> 字数 1208，阅读大约需 7 分钟

# Python之FastAPI的入门到精通系列：同步连接数据库Postgres

  
代码地址：`git@github.com:FunkyGod/fastapi-demo.git`

## SQLalchemy同步连接数据库

本文**演示如何使用SQLalchemy管理数据库，演示在fastapi维护DB管理**, 主要使用engine和sessionmaker，今天演示的是`同步连接DB`, **在async里协程最好是使用异步库连接DB**，避免阻碍事件循环：

### 1. Engine (数据库引擎)

使用engine，SQLAlchemy 的核心组件，可以**把 engine 理解为应用程序与数据库之间的桥梁**：

1. 1. 数据库连接管理器: 负责管理与数据库的所有连接
2. 2. 连接池管理: 维护一个连接池，复用数据库连接以提高性能
3. 3. SQL 执行入口: 所有 SQL 语句最终都通过 engine 发送到数据库
4. 4. 负责处理不同数据库（如 PostgreSQL、MySQL）的连接

### 2. SessionMaker (会话工厂)

sessionmaker 扮演会话生成器的角色：

1. 1. Session 工厂: 用于创建 Session 对象的工厂类
2. 2. 配置模板: 预先配置好 Session 的行为（如绑定到哪个 engine）
3. 3. 统一入口: 确保应用程序中所有 Session 都使用相同的配置

---

## 代码示例

1. 1. 创建一个数据库会话的上下文管理器。
2. 2. 创建一个 Session 工厂，用于后续创建独立的数据库会话实例

```
import json  
from contextlib import contextmanager  
import pydantic.json  
from sqlalchemy import create_engine  
from sqlalchemy.orm import sessionmaker  
  
from app.core.config import settings  
  
from .base_class import Base  # noqa  
def _custom_json_serializer(*args, **kwargs) -> str:  
    """  
    Encodes json in the same way that pydantic does.  
    """  
    return json.dumps(*args, default=pydantic.json.pydantic_encoder, **kwargs)  
# 创建数据库引擎  
# 引擎是 SQLAlchemy 应用的起点，它管理着与数据库的连接池。  
engine = create_engine(  
    # 数据库连接字符串，从配置中读取  
    settings.SQLALCHEMY_DATABASE_URI,  
    # 启用连接池预检测，每次从连接池获取连接时，会发送一个简单的查询以检查连接是否有效  
    pool_pre_ping=True,  
    # 自定义的 JSON 序列化函数  
    json_serializer=_custom_json_serializer,  
    # 连接池大小  
    pool_size=settings.SQLALCHEMY_POOL_SIZE,  
    # 连接回收时间（秒），-1 表示不回收  
    pool_recycle=settings.SQLALCHEMY_POOL_RECYCLE,  
    # 获取连接的超时时间（秒）  
    pool_timeout=settings.SQLALCHEMY_POOL_TIMEOUT,  
    # 连接池中允许超过 pool_size 的最大连接数  
    max_overflow=settings.SQLALCHEMY_POOL_OVERFLOW,  
    # 传递给数据库驱动的额外参数  
    connect_args=dict(  
        options=(  
            # 设置 PostgreSQL 的事务中空闲会话的超时时间  
            "-c idle_in_transaction_session_timeout="  
            f"{settings.POSTGRES_IDLE_IN_TRANSACTION_SESSION_TIMEOUT}"  
        )  
    ),  
)  
  
# 创建一个 Session 工厂，用于后续创建独立的数据库会话实例  
# autocommit=False 和 autoflush=False 是 SQLAlchemy 推荐的默认设置，  
# 这样可以手动控制事务的提交和刷新。  
_SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)  
  
  
@contextmanager  
def get_session():  
    """  
    创建一个数据库会话的上下文管理器。  
  
    用法:  
        with get_session() as db:  
            db.query(...).all()  
  
    该方法利用 @contextmanager 装饰器和 try...finally 结构，  
    确保数据库会话在使用完毕后，无论是否发生异常，  
    其 close() 方法都能被调用，从而安全地关闭和回收连接资源。  
    这是一种推荐的数据库会话管理模式，可以有效避免资源泄露。  
    """  
    db = _SessionLocal()  
    try:  
        yield db  
    finally:  
        # 此处的 close() 方法并不会直接关闭数据库的物理连接，  
        # 而是将本次会话（Session）所使用的连接释放，归还到连接池（Connection Pool）中。  
        # 这样做是为了复用连接，避免了频繁创建和销毁数据库连接带来的性能开销。  
        db.close()
```

## 在程序里复用连接池里的session

```
from typing import Generator  
from app.db import _SessionLocal  
  
def get_db() -> Generator:  
    try:  
        db = _SessionLocal()  
        yield db  
    finally:  
        db.close()
```

## 测试验证DB连接

```
import logging  
  
from app.db.init_db import init_db  
from app.db import get_session  
  
logging.basicConfig(level=logging.INFO)  
logger = logging.getLogger(__name__)  
  
  
def init() -> None:  
    with get_session() as db:  
        init_db(db)  
  
from sqlalchemy.orm import Session  
  
  
def init_db(db: Session) -> None:  
    pass
```

终端输出，执行DB连接和初始化成功，数据库表迁移成功  

  
在postgres数据库里查看表写入成功！

```
fastapi_demo_db=# \d  
             List of relations  
 Schema |      Name       | Type  |  Owner    
--------+-----------------+-------+---------  
 public | alembic_version | table | service  
(1 row)  
  
fastapi_demo_db=# \c fastapi_demo_db   
You are now connected to database "fastapi_demo_db" as user "service".  
fastapi_demo_db=# \d  
             List of relations  
 Schema |      Name       | Type  |  Owner    
--------+-----------------+-------+---------  
 public | alembic_version | table | service  
(1 row)  
  
fastapi_demo_db=# 
```

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