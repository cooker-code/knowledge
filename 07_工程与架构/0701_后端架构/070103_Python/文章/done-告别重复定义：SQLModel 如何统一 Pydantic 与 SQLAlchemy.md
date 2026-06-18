> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/FastAPI生产边界与降权准则|FastAPI生产边界与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python架构实现路线|Python架构实现路线]]
---
title: 告别重复定义：SQLModel 如何统一 Pydantic 与 SQLAlchemy
author: 全栈数据
date:
url: https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356995&idx=1&sn=3154df0a5e3e541c5893d7d163f66ddb&chksm=81cf42f146b406baec8510f64768da12cbf55b7d9ac491cb6975a4e2dd56e6cf9cee85cbea51&mpshare=1&scene=24&srcid=04215acV9FGqBFGplvfYwez2&sharer_shareinfo=9cad246ba3b28bf5556ffd1fbd71721f&sharer_shareinfo_first=9cad246ba3b28bf5556ffd1fbd71721f#rd
---

在 Python 的 Web 开发生态中，尤其是 FastAPI 框架流行之后，开发者长期面临着一个“双重定义”的痛点：我们需要使用 Pydantic 来定义数据验证模型（用于 API 请求和响应），同时又需要使用 SQLAlchemy 来定义数据库表模型（用于 ORM 操作）。这不仅导致了大量的样板代码，还增加了维护成本。

**SQLModel**正是为了解决这一问题而生。作为 FastAPI 作者 Sebastián Ramírez (tiangolo) 开发的又一力作，SQLModel 巧妙地融合了 Pydantic 的数据验证能力与 SQLAlchemy 的数据库交互能力。它允许开发者仅用一套代码，同时实现 API 数据校验与数据库持久化，真正做到了“一个模型走天下”。

**系统特点**

SQLModel 的设计哲学可以概括为简洁性、兼容性与健壮性。

* DRY 原则的极致体现：这是 SQLModel 最核心的优势。它继承了 Pydantic 的 BaseModel 和 SQLAlchemy 的 ORM 特性，使得同一个类既可以是 Pydantic 模型（支持字段校验、类型提示、OpenAPI 文档生成），也可以是 SQLAlchemy 模型（支持数据库表映射、增删改查）。
* 类型安全与 IDE 友好：SQLModel 基于 Python 类型注解设计，提供完整的 MyPy 支持。这意味着开发者在编写代码时就能享受到 IDE 的智能补全和实时错误提示，极大减少了运行时错误。
* 无缝集成 FastAPI：由于设计初衷就是为了配合 FastAPI，SQLModel 在 Web 开发场景中表现卓越。它支持自动的请求/响应验证，能够显著简化 API 开发流程。
* 强大的兼容性：它不是对 SQLAlchemy 的简单封装，而是深度兼容。你可以直接使用 SQLAlchemy 的核心功能，甚至在同一项目中混合使用 SQLModel 和原生 SQLAlchemy 代码。

**系统架构**

SQLModel 的架构设计非常清晰，它构建在 Python 强大的类型系统之上，主要依赖以下核心技术栈：

* 核心引擎：基于 SQLAlchemy 2.0+，利用其成熟的 ORM 引擎处理数据库连接、会话管理和查询构建。
* 数据验证层：集成 Pydantic 2.0+，负责数据的解析、验证和序列化。
* 类型系统：利用 Python 内置的 typing 模块，通过类型注解来驱动字段定义和关系映射。

在架构层面，SQLModel 通过 Field 函数巧妙地连接了这两层。Field 既接受 Pydantic 的参数（如 min\_length、regex 用于校验），也接受 SQLAlchemy 的参数（如 primary\_key、unique、index 用于数据库约束）。这种设计使得开发者在定义模型时，能够同时控制 API 层的行为和数据库层的结构。

**功能列表**

* 统一模型定义：单一类定义同时满足 API 序列化与数据库 ORM 需求。
* 自动数据验证：利用 Pydantic 机制，自动处理请求体解析与字段校验（如 min\_length、regex）。
* 类型安全支持：提供完整的类型提示，支持 MyPy 静态类型检查。
* 异步数据库支持：支持 async/await 语法，适应高并发场景。
* 关系映射：支持一对一、一对多及多对多（通过中间表）的复杂关系定义。
* 自动文档生成：与 FastAPI 配合，自动生成包含数据模型信息的 Swagger/OpenAPI 文档。

**项目地址**

GitCode 镜像：https://gitcode.com/gh\_mirrors/sq/sqlmodel

官方文档：SQLModel Documentation

PyPI 仓库：sqlmodel

中文文档地址：https://sqlmodel.fastapi.org.cn/#editor-support-everywhere

**快速体验**

为了让你快速感受 SQLModel 的魅力，以下是一个标准的 FastAPI + SQLModel 实战示例。我们将定义一个 User 模型，并实现简单的创建与查询接口。

**1. 安装依赖**

```
pip install sqlmodel fastapi uvicorn
```

**2. 定义模型与数据库 (main.py)**

```
from typing import Optionalfrom fastapi import FastAPI, Depends, HTTPExceptionfrom sqlmodel import SQLModel, Field, Session, create_engine, select
# 1. 数据库配置engine = create_engine("sqlite:///./test.db", echo=True)
# 2. 定义统一模型# 这个类既是数据库表，又是 Pydantic 模型class User(SQLModel, table=True):    id: Optional[int] = Field(default=None, primary_key=True)    name: str = Field(min_length=1, max_length=50, index=True)  # Pydantic校验生效    email: str = Field(unique=True, regex=r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$")    age: Optional[int] = Field(default=None, ge=0, le=150)
# 创建数据库表def create_db_and_tables():    SQLModel.metadata.create_all(engine)
# 3. 会话依赖def get_session():    with Session(engine) as session:        yield session
# 4. 定义 API 路由app = FastAPI()
@app.on_event("startup")def on_startup():    create_db_and_tables()
@app.post("/users/", response_model=User)def create_user(user: User, session: Session = Depends(get_session)):    # 检查邮箱是否已存在    statement = select(User).where(User.email == user.email)    existing_user = session.exec(statement).first()    if existing_user:        raise HTTPException(status_code=400, detail="Email already registered")        session.add(user)    session.commit()    session.refresh(user)    return user
@app.get("/users/", response_model=list[User])def read_users(session: Session = Depends(get_session)):    return session.exec(select(User)).all()
```

**3. 运行项目**

```
uvicorn main:app --reload
```

启动后，访问 http://127.0.0.1:8000/docs，你将看到自动生成的交互式 API 文档。尝试发送一个 POST 请求，你会发现 name 的长度限制和 email 的格式校验已经自动生效，而数据也被成功保存到了 SQLite 数据库中。这就是 SQLModel 带来的极简开发体验。

**更多历史热门文章**

**[用 Vue3 + Tailwind CSS 告别重复造轮子：Art Design Pro 如何让后台开发效率翻倍](https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356277&idx=2&sn=b07bfb52c82dab992ae8046625e68708&scene=21#wechat_redirect)**

**[一套API通吃所有数据库？dbVisitor的双层适配架构揭秘](https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356296&idx=2&sn=42e308740c4398e81706a9e4bf9c100c&scene=21#wechat_redirect)**

**[告别繁琐配置！用 LinkUp + SeaTunnel 打造你的轻量级数据中台](https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356275&idx=2&sn=698218488010c3955e4e6e66dba5075b&scene=21#wechat_redirect)**

**[让 Excel/ET、Word/WPS、PPT/WPP“更听话”：一套统一 API 的 Office 操作方案](https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356261&idx=1&sn=ef0fab378863ca018467b6c4df48030f&scene=21#wechat_redirect)**

**[蚂蚁AntV L7：用WebGL点燃你的地理空间数据可视化](https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356221&idx=1&sn=b96661107942d6d96dec79d0315ac745&scene=21#wechat_redirect)**

**[告别 Pandas 瓶颈：用 Polars 重构你的数据处理流水线](https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356138&idx=1&sn=16764a5ecc92e95e34b7a8aafb2df354&scene=21#wechat_redirect)**

**[Postman 已死？这个国产开源神器正悄悄取代它](https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356062&idx=1&sn=8bd65ade5360ea947590d81f5858b209&scene=21#wechat_redirect)**

**[EasyETL：零代码也能玩转大数据调度？开源分布式中台实战解析](https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356929&idx=1&sn=f1e3cef6c992681f55d4628ec4bc56ba&scene=21#wechat_redirect)**

**[All-in-One 王炸！这款桌面客户端把数据库、服务器和 AI 全“吃”进去了](https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356874&idx=1&sn=2cb46895b7615f8eb779a6038bf13bb6&scene=21#wechat_redirect)**

**[Flink + Dinky = 实时 ETL 的终极组合？一文带你实战落地](https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356864&idx=2&sn=0194a933104279e71e945d6aeb5b3f0e&scene=21#wechat_redirect)**

更多数据科学与技术，请关注：**全栈数据**