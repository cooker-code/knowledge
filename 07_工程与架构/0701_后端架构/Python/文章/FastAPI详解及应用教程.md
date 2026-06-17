---
title: FastAPI详解及应用教程
author: Linux开源之家
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI0NTA4Mjg3OA==&mid=2650561241&idx=1&sn=be8d348e80c98fcb5a82211194cc573c&chksm=f0fa5d0e222a73466d2e14e8716e98e75b8f00982d1fef3da05ac75983245dff87742e0917d0&mpshare=1&scene=24&srcid=010288M0oXg4AM4QPiVWErn1&sharer_shareinfo=66eb104d6d9e20994c082aa6987f5251&sharer_shareinfo_first=66eb104d6d9e20994c082aa6987f5251#rd
---

由于FastAPI是一个现代、快速（高性能）的Web框架，用于构建API，它基于Python 3.6+并使用了标准类型提示。本教程将详细介绍FastAPI的核心概念、特性以及如何构建一个完整的API。

目录：

1. 安装FastAPI
2. 创建一个简单的FastAPI应用
3. 路径参数
4. 查询参数
5. 请求体
6. 响应模型
7. 错误处理
8. 依赖注入
9. 安全性
10. 中间件
11. 数据库集成（以SQLAlchemy为例）
12. 部署

让我们开始吧。

1. 安装FastAPI  
   首先，需要安装FastAPI和uvicorn（一个ASGI服务器）：

```
pip install fastapi uvicorn
```

2. 创建一个简单的FastAPI应用  
   创建一个main.py文件，并写入以下内容：

```
from fastapi import FastAPI  
app = FastAPI()  
@app.get("/")async def root():    return {"message": "Hello World"}
```

运行应用：

```
uvicorn main:app --reload
```

访问 http://127.0.0.1:8000 ，你将看到JSON响应：{"message": "Hello World"}

3. 路径参数  
   你可以使用与Python格式化字符串相同的语法来声明路径参数。

```
@app.get("/items/{item_id}")async def read_item(item_id: int):    return {"item_id": item_id}
```

路径参数item\_id将作为参数传递给函数，并自动转换为整数类型。

4. 查询参数  
   当你声明不属于路径参数的其他函数参数时，它们将被解释为查询参数。

```
from typing import Optional  
@app.get("/items/")async def read_items(skip: int = 0, limit: int = 10, q: Optional[str] = None):    return {"skip": skip, "limit": limit, "q": q}
```

访问 http://127.0.0.1:8000/items/?skip=0&limit=20 将返回相应的JSON。

5. 请求体  
   当你需要从客户端发送数据到API时，你可以使用请求体。在FastAPI中，我们使用Pydantic模型来定义请求体的结构。

```
from pydantic import BaseModel  
class Item(BaseModel):    name: str    description: Optional[str] = None    price: float    tax: Optional[float] = None  
@app.post("/items/")async def create_item(item: Item):    return item
```

在这个例子中，我们定义了一个Item模型，它有一个name（字符串）、description（可选字符串）、price（浮点数）和tax（可选浮点数）。然后，我们在路径操作函数中声明一个参数item，其类型为Item。FastAPI将自动读取请求体，解析为JSON，转换为相应的类型，并进行验证。

6. 响应模型  
   你可以使用response\_model参数来定义响应模型，确保返回的数据符合模型定义。

```
@app.post("/items/", response_model=Item)async def create_item(item: Item):    return item
```

这样，即使你的函数返回了额外的数据，FastAPI也会根据response\_model进行过滤和验证。

7. 错误处理  
   FastAPI提供了HTTPException异常，你可以抛出它来返回错误响应。

```
from fastapi import HTTPException  
@app.get("/items/{item_id}")async def read_item(item_id: int):    if item_id not in items:        raise HTTPException(status_code=404, detail="Item not found")    return {"item_id": item_id}
```

你也可以自定义异常处理器。

8. 依赖注入  
   FastAPI有一个强大的依赖注入系统。你可以声明依赖项，FastAPI会自动处理它们。

```
from fastapi import Depends  
async def common_parameters(q: Optional[str] = None, skip: int = 0, limit: int = 100):    return {"q": q, "skip": skip, "limit": limit}  
@app.get("/items/")async def read_items(commons: dict = Depends(common_parameters)):    return commons
```

在这个例子中，common\_parameters函数是一个依赖项，它返回一个字典。在路径操作函数中，我们声明一个参数commons，它依赖于common\_parameters。FastAPI将调用common\_parameters函数，并将结果注入到commons中。

9. 安全性  
   FastAPI提供了多种安全工具，包括OAuth2、JWT等。

例如，使用OAuth2密码流程：

```
from fastapi.security import OAuth2PasswordBearer  
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")  
@app.get("/items/")async def read_items(token: str = Depends(oauth2_scheme)):    return {"token": token}
```

这样，你的API将需要一个token参数（在请求头中）来访问。

10. 中间件  
    你可以添加中间件来处理请求和响应。

```
from fastapi.middleware.cors import CORSMiddleware  
app.add_middleware(    CORSMiddleware,    allow_origins=["*"],    allow_credentials=True,    allow_methods=["*"],    allow_headers=["*"],)
```

   这个例子添加了CORS中间件，允许跨域请求。

11.数据库集成（以SQLAlchemy为例）  
首先，安装SQLAlchemy和数据库驱动（如sqlite）：

```
pip install sqlalchemy
```

然后，你可以定义模型、数据库会话等。由于篇幅限制，这里只给出一个简单示例。

建议使用FastAPI的官方项目模板来组织大型项目，它已经包含了数据库配置。

12.部署  
你可以使用uvicorn来部署FastAPI应用，也可以使用Gunicorn配合uvicorn worker。

生产部署建议：

```
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

或者使用Gunicorn（如果使用UvicornWorker）：

```
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker -b 0.0.0.0:8000
```

这只是一个快速的FastAPI教程概述。FastAPI还有更多高级特性，如WebSocket、GraphQL、后台任务等。你可以查阅官方文档以获取更多信息：https://fastapi.tiangolo.com/

# FastAPI 详细教程

## 目录

1. FastAPI 简介
2. 安装与配置
3. 基础应用结构
4. 路径操作
5. 请求与响应
6. 数据验证与模型
7. 依赖注入
8. 数据库集成
9. 认证与授权
10. 中间件
11. 部署

---

## 1. FastAPI 简介

### 什么是 FastAPI？

FastAPI 是一个现代、快速（高性能）的 Web 框架，用于基于标准 Python 类型提示构建 API。主要特性包括：

* **快速**：非常高的性能，与 NodeJS 和 Go 相当
* **快速编码**：开发功能速度提高约 200% 到 300%
* **更少的错误**：减少约 40% 的人为错误
* **直观**：强大的编辑器支持
* **简单**：易于学习和使用
* **简短**：减少代码重复
* **健壮**：生产就绪的代码
* **标准化**：基于并完全兼容 API 开放标准（OpenAPI 和 JSON Schema）

### 主要特性

* 自动生成交互式 API 文档
* 基于 Python 类型提示
* 数据验证
* 异步支持
* 依赖注入系统
* OAuth2 和 JWT 支持
* CORS、GZip、静态文件等中间件

---

## 2. 安装与配置

### 基本安装

```
# 安装 FastAPI 和 ASGI 服务器pip install fastapipip install uvicorn[standard]  
# 可选：用于数据库操作pip install sqlalchemypip install databasespip install alembic  
# 可选：用于认证pip install python-jose[cryptography]pip install passlib[bcrypt]pip install python-multipart
```

### 最小示例

```
# main.pyfrom fastapi import FastAPI  
app = FastAPI()  
@app.get("/")async def root():    return {"message": "Hello World"}
```

运行应用：

```
uvicorn main:app --reload
```

访问：

* API: http://127.0.0.1:8000
* 文档: http://127.0.0.1:8000/docs
* 备用文档: http://127.0.0.1:8000/redoc

## 3. 基础应用结构

### 项目结构

```
myproject/├── app/│   ├── __init__.py│   ├── main.py│   ├── api/│   │   ├── __init__.py│   │   ├── v1/│   │   │   ├── __init__.py│   │   │   ├── endpoints/│   │   │   └── router.py│   ├── core/│   │   ├── config.py│   │   └── security.py│   ├── models/│   ├── schemas/│   └── crud/├── tests/└── requirements.txt
```

### 应用工厂模式

```
# app/main.pyfrom fastapi import FastAPIfrom app.api.v1.api import api_routerfrom app.core.config import settings  
def create_application() -> FastAPI:    application = FastAPI(        title=settings.PROJECT_NAME,        version=settings.VERSION,        openapi_url=f"{settings.API_V1_STR}/openapi.json"    )  
    # 注册路由    application.include_router(api_router, prefix=settings.API_V1_STR)  
    return application  
app = create_application()
```

## 4. 路径操作

### 基本路径操作

```
from fastapi import FastAPI  
app = FastAPI()  
# GET 请求@app.get("/items/{item_id}")async def read_item(item_id: int):    return {"item_id": item_id}  
# POST 请求@app.post("/items/")async def create_item(item: dict):    return item  
# PUT 请求@app.put("/items/{item_id}")async def update_item(item_id: int, item: dict):    return {"item_id": item_id, "item": item}  
# DELETE 请求@app.delete("/items/{item_id}")async def delete_item(item_id: int):    return {"deleted": item_id}  
# PATCH 请求@app.patch("/items/{item_id}")async def patch_item(item_id: int, item: dict):    return {"item_id": item_id, "updated": item}
```

### 路径参数

```
from enum import Enum  
class ModelName(str, Enum):    alexnet = "alexnet"    resnet = "resnet"    lenet = "lenet"  
@app.get("/models/{model_name}")async def get_model(model_name: ModelName):    if model_name == ModelName.alexnet:        return {"model_name": model_name, "message": "Deep Learning FTW!"}    if model_name.value == "lenet":        return {"model_name": model_name, "message": "LeCNN all the images"}    return {"model_name": model_name, "message": "Have some residuals"}  
# 包含路径的路径参数@app.get("/files/{file_path:path}")async def read_file(file_path: str):    return {"file_path": file_path}
```

### 查询参数

```
from typing import Optional, List  
# 必需参数和可选参数@app.get("/items/")async def read_items(    skip: int = 0,          # 必需参数    limit: int = 10,        # 必需参数    q: Optional[str] = None # 可选参数):    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}    if q:        results.update({"q": q})    return results  
# 多个路径和查询参数@app.get("/users/{user_id}/items/{item_id}")async def read_user_item(    user_id: int,    item_id: str,    q: Optional[str] = None,    short: bool = False):    item = {"item_id": item_id, "owner_id": user_id}    if q:        item.update({"q": q})    if not short:        item.update({"description": "This is an amazing item"})    return item
```

## 5. 请求与响应

### 请求体

```
from pydantic import BaseModelfrom typing import Optional  
class Item(BaseModel):    name: str    description: Optional[str] = None    price: float    tax: Optional[float] = None  
@app.post("/items/")async def create_item(item: Item):    return item  
# 多个请求体参数class User(BaseModel):    username: str    full_name: Optional[str] = None  
@app.put("/items/{item_id}")async def update_item(item_id: int, item: Item, user: User):    return {        "item_id": item_id,        "item": item,        "user": user    }
```

### 响应模型

```
from typing import List  
# 基础响应模型class ItemResponse(BaseModel):    name: str    description: Optional[str] = None    price: float    tax: Optional[float] = None  
@app.post("/items/", response_model=ItemResponse)async def create_item(item: Item):    return item  
# 排除响应字段@app.get("/items/", response_model=List[ItemResponse])async def read_items():    return [        Item(name="Foo", price=50.2),        Item(name="Bar", price=62.5)    ]  
# 响应状态码from fastapi import status  
@app.post("/items/", status_code=status.HTTP_201_CREATED)async def create_item(item: Item):    return item
```

### 响应参数

```
from fastapi import Response, Cookie, Headerfrom typing import Optional, List  
# 设置响应头@app.get("/headers/")async def read_items(response: Response):    response.headers["X-Custom-Header"] = "CustomValue"    return {"message": "Hello World"}  
# Cookie 操作@app.get("/cookies/")async def read_cookies(cookie_id: Optional[str] = Cookie(None)):    return {"cookie_id": cookie_id}  
@app.post("/cookies/")async def create_cookie(response: Response):    response.set_cookie(key="fakesession", value="fake-cookie-session-value")    return {"message": "Cookie set"}  
# Header 参数@app.get("/headers/")async def read_headers(    user_agent: Optional[str] = Header(None),    x_token: Optional[List[str]] = Header(None)):    return {        "User-Agent": user_agent,        "X-Token": x_token    }
```

## 6. 数据验证与模型

### Pydantic 模型

```
from pydantic import BaseModel, Field, EmailStr, validatorfrom typing import Optional, Listfrom datetime import datetime  
class UserBase(BaseModel):    email: EmailStr    username: str  
class UserCreate(UserBase):    password: str = Field(..., min_length=8)  
    @validator('password')    def validate_password(cls, v):        if not any(c.isupper() for c in v):            raise ValueError('Password must contain at least one uppercase letter')        if not any(c.isdigit() for c in v):            raise ValueError('Password must contain at least one digit')        return v  
class User(UserBase):    id: int    is_active: bool = True    created_at: datetime  
    class Config:        orm_mode = True  # 允许从ORM对象创建模型  
# 嵌套模型class Item(BaseModel):    name: str    price: float    tags: List[str] = []  
class Order(BaseModel):    id: int    user: User    items: List[Item]    total: float
```

### 字段验证

```
from pydantic import BaseModel, Field, constr, conint, confloat  
class Item(BaseModel):    # 字符串验证    name: constr(min_length=1, max_length=50)  
    # 数值验证    price: confloat(gt=0)    quantity: conint(ge=0)  
    # 使用 Field 验证    description: str = Field(        default=None,        title="商品描述",        description="商品的详细描述",        max_length=300    )  
    # 正则表达式验证    code: str = Field(        ...,        regex=r'^[A-Z]{3}-\d{3}$',        example="ABC-123"    )
```

### 表单数据

```
from fastapi import Form  
@app.post("/login/")async def login(    username: str = Form(...),    password: str = Form(...)):    return {"username": username}
```

### 文件上传

```
from fastapi import UploadFile, Filefrom typing import List  
# 单个文件@app.post("/upload/")async def upload_file(file: UploadFile = File(...)):    contents = await file.read()    return {        "filename": file.filename,        "content_type": file.content_type,        "size": len(contents)    }  
# 多个文件@app.post("/upload-multiple/")async def upload_multiple_files(files: List[UploadFile] = File(...)):    return [{"filename": file.filename} for file in files]
```

## 7. 依赖注入

### 基础依赖

```
from fastapi import Depends, FastAPI, HTTPExceptionfrom typing import Optional  
app = FastAPI()  
# 简单的依赖函数async def common_parameters(    q: Optional[str] = None,    skip: int = 0,    limit: int = 100):    return {"q": q, "skip": skip, "limit": limit}  
@app.get("/items/")async def read_items(commons: dict = Depends(common_parameters)):    return commons  
@app.get("/users/")async def read_users(commons: dict = Depends(common_parameters)):    return commons
```

### 类作为依赖

```
class CommonQueryParams:    def __init__(        self,        q: Optional[str] = None,        skip: int = 0,        limit: int = 100    ):        self.q = q        self.skip = skip        self.limit = limit  
@app.get("/items/")async def read_items(commons: CommonQueryParams = Depends()):    return commons
```

### 依赖嵌套

```
def query_extractor(q: Optional[str] = None):    return q  
def query_or_body_extractor(    q: str = Depends(query_extractor),    last_query: Optional[str] = None):    if not q:        return last_query    return q  
@app.get("/items/")async def read_query(query_or_default: str = Depends(query_or_body_extractor)):    return {"q_or_default": query_or_default}
```

### 数据库依赖

```
from sqlalchemy.orm import Sessionfrom app.db.session import SessionLocal  
# 数据库会话依赖def get_db():    db = SessionLocal()    try:        yield db    finally:        db.close()  
@app.get("/users/{user_id}")async def read_user(user_id: int, db: Session = Depends(get_db)):    user = db.query(User).filter(User.id == user_id).first()    if user is None:        raise HTTPException(status_code=404, detail="User not found")    return user
```

### 带参数的依赖

```
from typing import Optional  
# 带参数的依赖工厂class CheckPermissions:    def __init__(self, required_permissions: list):        self.required_permissions = required_permissions  
    def __call__(self, user: User = Depends(get_current_user)):        for permission in self.required_permissions:            if permission not in user.permissions:                raise HTTPException(                    status_code=403,                    detail=f"Missing permission: {permission}"                )        return user  
@app.get("/admin/")async def admin_panel(user: User = Depends(CheckPermissions(["admin"]))):    return {"message": "Welcome admin"}
```

## 8. 数据库集成

### SQLAlchemy 配置

```
# app/db/session.pyfrom sqlalchemy import create_enginefrom sqlalchemy.ext.declarative import declarative_basefrom sqlalchemy.orm import sessionmakerfrom app.core.config import settings  
engine = create_engine(    settings.DATABASE_URL,    pool_pre_ping=True,    pool_recycle=3600)  
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)Base = declarative_base()  
# app/db/base.pyfrom app.db.session import Basefrom sqlalchemy import Column, Integer, String, DateTime, Boolean, ForeignKeyfrom sqlalchemy.orm import relationshipfrom datetime import datetime  
class User(Base):    __tablename__ = "users"  
    id = Column(Integer, primary_key=True, index=True)    email = Column(String, unique=True, index=True, nullable=False)    username = Column(String, unique=True, index=True, nullable=False)    hashed_password = Column(String, nullable=False)    is_active = Column(Boolean, default=True)    created_at = Column(DateTime, default=datetime.utcnow)  
    items = relationship("Item", back_populates="owner")  
class Item(Base):    __tablename__ = "items"  
    id = Column(Integer, primary_key=True, index=True)    title = Column(String, index=True)    description = Column(String, index=True)    owner_id = Column(Integer, ForeignKey("users.id"))  
    owner = relationship("User", back_populates="items")
```

### CRUD 操作

```
# app/crud/base.pyfrom typing import Any, Dict, Generic, List, Optional, Type, TypeVar, Unionfrom fastapi.encoders import jsonable_encoderfrom pydantic import BaseModelfrom sqlalchemy.orm import Session  
ModelType = TypeVar("ModelType", bound=Any)CreateSchemaType = TypeVar("CreateSchemaType", bound=BaseModel)UpdateSchemaType = TypeVar("UpdateSchemaType", bound=BaseModel)  
class CRUDBase(Generic[ModelType, CreateSchemaType, UpdateSchemaType]):    def __init__(self, model: Type[ModelType]):        self.model = model  
    def get(self, db: Session, id: Any) -> Optional[ModelType]:        return db.query(self.model).filter(self.model.id == id).first()  
    def get_multi(        self, db: Session, *, skip: int = 0, limit: int = 100    ) -> List[ModelType]:        return db.query(self.model).offset(skip).limit(limit).all()  
    def create(self, db: Session, *, obj_in: CreateSchemaType) -> ModelType:        obj_in_data = jsonable_encoder(obj_in)        db_obj = self.model(**obj_in_data)        db.add(db_obj)        db.commit()        db.refresh(db_obj)        return db_obj  
    def update(        self,        db: Session,        *,        db_obj: ModelType,        obj_in: Union[UpdateSchemaType, Dict[str, Any]]    ) -> ModelType:        obj_data = jsonable_encoder(db_obj)        if isinstance(obj_in, dict):            update_data = obj_in        else:            update_data = obj_in.dict(exclude_unset=True)        for field in obj_data:            if field in update_data:                setattr(db_obj, field, update_data[field])        db.add(db_obj)        db.commit()        db.refresh(db_obj)        return db_obj  
    def remove(self, db: Session, *, id: int) -> ModelType:        obj = db.query(self.model).get(id)        db.delete(obj)        db.commit()        return obj  
# app/crud/user.pyfrom app.models.user import Userfrom app.schemas.user import UserCreate, UserUpdatefrom app.core.security import get_password_hash  
class CRUDUser(CRUDBase[User, UserCreate, UserUpdate]):    def get_by_email(self, db: Session, *, email: str) -> Optional[User]:        return db.query(User).filter(User.email == email).first()  
    def create(self, db: Session, *, obj_in: UserCreate) -> User:        db_obj = User(            email=obj_in.email,            hashed_password=get_password_hash(obj_in.password),            username=obj_in.username,        )        db.add(db_obj)        db.commit()        db.refresh(db_obj)        return db_obj  
user = CRUDUser(User)
```

### 数据库迁移（Alembic）

```
# alembic.ini[alembic]script_location = alembicprepend_sys_path = .version_path_separator = os  
# alembic/env.pyfrom logging.config import fileConfigfrom sqlalchemy import engine_from_configfrom sqlalchemy import poolfrom alembic import contextfrom app.db.base import Basefrom app.core.config import settings  
config = context.configconfig.set_main_option("sqlalchemy.url", settings.DATABASE_URL)  
target_metadata = Base.metadata  
def run_migrations_offline():    context.configure(        url=settings.DATABASE_URL,        target_metadata=target_metadata,        literal_binds=True,        dialect_opts={"paramstyle": "named"},    )    with context.begin_transaction():        context.run_migrations()  
def run_migrations_online():    connectable = engine_from_config(        config.get_section(config.config_ini_section),        prefix="sqlalchemy.",        poolclass=pool.NullPool,    )    with connectable.connect() as connection:        context.configure(            connection=connection,            target_metadata=target_metadata        )        with context.begin_transaction():            context.run_migrations()  
if context.is_offline_mode():    run_migrations_offline()else:    run_migrations_online()
```

## 9. 认证与授权

### JWT 认证

```
# app/core/security.pyfrom datetime import datetime, timedeltafrom typing import Optionalfrom jose import JWTError, jwtfrom passlib.context import CryptContextfrom app.core.config import settings  
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")  
def verify_password(plain_password: str, hashed_password: str) -> bool:    return pwd_context.verify(plain_password, hashed_password)  
def get_password_hash(password: str) -> str:    return pwd_context.hash(password)  
def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):    to_encode = data.copy()    if expires_delta:        expire = datetime.utcnow() + expires_delta    else:        expire = datetime.utcnow() + timedelta(minutes=15)    to_encode.update({"exp": expire})    encoded_jwt = jwt.encode(        to_encode,        settings.SECRET_KEY,        algorithm=settings.ALGORITHM    )    return encoded_jwt  
# app/api/deps.pyfrom fastapi import Depends, HTTPException, statusfrom fastapi.security import OAuth2PasswordBearerfrom jose import JWTError, jwtfrom app.core.config import settings  
oauth2_scheme = OAuth2PasswordBearer(tokenUrl=f"{settings.API_V1_STR}/login")  
async def get_current_user(token: str = Depends(oauth2_scheme)):    credentials_exception = HTTPException(        status_code=status.HTTP_401_UNAUTHORIZED,        detail="Could not validate credentials",        headers={"WWW-Authenticate": "Bearer"},    )    try:        payload = jwt.decode(            token,            settings.SECRET_KEY,            algorithms=[settings.ALGORITHM]        )        username: str = payload.get("sub")        if username is None:            raise credentials_exception    except JWTError:        raise credentials_exception    user = get_user(username=username)    if user is None:        raise credentials_exception    return user  
# 登录路由@app.post("/login")async def login(    form_data: OAuth2PasswordRequestForm = Depends(),    db: Session = Depends(get_db)):    user = authenticate_user(db, form_data.username, form_data.password)    if not user:        raise HTTPException(            status_code=400,            detail="Incorrect username or password"        )    access_token_expires = timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)    access_token = create_access_token(        data={"sub": user.username},        expires_delta=access_token_expires    )    return {        "access_token": access_token,        "token_type": "bearer"    }
```

### OAuth2 密码流

```
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm  
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")  
@app.post("/token")async def login(form_data: OAuth2PasswordRequestForm = Depends()):    user_dict = fake_users_db.get(form_data.username)    if not user_dict:        raise HTTPException(status_code=400, detail="Incorrect username or password")    user = UserInDB(**user_dict)    hashed_password = fake_hash_password(form_data.password)    if not hashed_password == user.hashed_password:        raise HTTPException(status_code=400, detail="Incorrect username or password")  
    return {"access_token": user.username, "token_type": "bearer"}  
@app.get("/users/me")async def read_users_me(current_user: User = Depends(get_current_user)):    return current_user
```

### 角色权限控制

```
from typing import Listfrom fastapi import Security  
class RoleChecker:    def __init__(self, allowed_roles: List):        self.allowed_roles = allowed_roles  
    def __call__(self, user: User = Depends(get_current_user)):        if user.role not in self.allowed_roles:            raise HTTPException(                status_code=403,                detail="Operation not permitted"            )        return user  
allow_admin = RoleChecker(["admin"])allow_admin_editor = RoleChecker(["admin", "editor"])  
@app.get("/admin-only", dependencies=[Depends(allow_admin)])async def admin_only():    return {"message": "Admin only endpoint"}  
@app.get("/admin-editor", dependencies=[Depends(allow_admin_editor)])async def admin_editor():    return {"message": "Admin and editor endpoint"}
```

## 10. 中间件

### CORS 中间件

```
from fastapi.middleware.cors import CORSMiddleware  
app.add_middleware(    CORSMiddleware,    allow_origins=["http://localhost:3000"],  # 前端地址    allow_credentials=True,    allow_methods=["*"],    allow_headers=["*"],)
```

### 自定义中间件

```
import timefrom fastapi import Request  
@app.middleware("http")async def add_process_time_header(request: Request, call_next):    start_time = time.time()    response = await call_next(request)    process_time = time.time() - start_time    response.headers["X-Process-Time"] = str(process_time)    return response  
@app.middleware("http")async def log_requests(request: Request, call_next):    logger.info(f"Request: {request.method} {request.url}")    response = await call_next(request)    logger.info(f"Response status: {response.status_code}")    return response
```

### 认证中间件

```
from starlette.middleware.base import BaseHTTPMiddleware  
class AuthenticationMiddleware(BaseHTTPMiddleware):    async def dispatch(self, request: Request, call_next):        # 检查请求头中的认证信息        auth_header = request.headers.get("Authorization")        if auth_header:            # 解析 token            pass  
        response = await call_next(request)        return response  
app.add_middleware(AuthenticationMiddleware)
```

### 速率限制中间件

```
from slowapi import Limiter, _rate_limit_exceeded_handlerfrom slowapi.util import get_remote_addressfrom slowapi.errors import RateLimitExceeded  
limiter = Limiter(key_func=get_remote_address)app.state.limiter = limiterapp.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)  
@app.get("/home")@limiter.limit("5/minute")async def homepage(request: Request):    return {"message": "Hello World"}
```

## 11. 部署

### 生产配置

```
# app/core/config.pyfrom pydantic import BaseSettingsfrom typing import Optional  
class Settings(BaseSettings):    PROJECT_NAME: str = "My FastAPI Project"    VERSION: str = "1.0.0"    API_V1_STR: str = "/api/v1"  
    # 安全    SECRET_KEY: str    ALGORITHM: str = "HS256"    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30  
    # 数据库    DATABASE_URL: str  
    # CORS    BACKEND_CORS_ORIGINS: list = ["http://localhost:3000"]  
    class Config:        env_file = ".env"  
settings = Settings()
```

### Docker 部署

```
# DockerfileFROM python:3.9-slim  
WORKDIR /app  
COPY requirements.txt .RUN pip install --no-cache-dir -r requirements.txt  
COPY . .  
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]  
# docker-compose.ymlversion: '3.8'  
services:  web:    build: .    ports:      - "8000:8000"    environment:      - DATABASE_URL=postgresql://user:password@db:5432/mydb    depends_on:      - db  
  db:    image: postgres:13    environment:      - POSTGRES_USER=user      - POSTGRES_PASSWORD=password      - POSTGRES_DB=mydb    volumes:      - postgres_data:/var/lib/postgresql/data  
volumes:  postgres_data:
```

### Nginx 配置

```
# nginx.confupstream fastapi {    server web:8000;}  
server {    listen 80;  
    location / {        proxy_pass http://fastapi;        proxy_set_header Host $host;        proxy_set_header X-Real-IP $remote_addr;        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;        proxy_set_header X-Forwarded-Proto $scheme;    }  
    location /static {        alias /app/static;    }}
```

### Gunicorn 部署

```
# gunicorn_conf.pyimport multiprocessing  
workers = multiprocessing.cpu_count() * 2 + 1worker_class = "uvicorn.workers.UvicornWorker"bind = "0.0.0.0:8000"accesslog = "-"errorlog = "-"timeout = 120keepalive = 5  
# 启动命令gunicorn -c gunicorn_conf.py app.main:app
```

### 健康检查

```
@app.get("/health")async def health_check():    return {        "status": "healthy",        "timestamp": datetime.utcnow().isoformat()    }  
@app.get("/ready")async def readiness_check(db: Session = Depends(get_db)):    # 检查数据库连接    try:        db.execute("SELECT 1")        return {"database": "ready"}    except Exception as e:        raise HTTPException(status_code=503, detail="Database not ready")
```

### 监控与日志

```
import loggingfrom pythonjsonlogger import jsonlogger  
# 配置 JSON 日志logger = logging.getLogger()logHandler = logging.StreamHandler()formatter = jsonlogger.JsonFormatter(    "%(asctime)s %(levelname)s %(message)s",    datefmt="%Y-%m-%dT%H:%M:%S")logHandler.setFormatter(formatter)logger.addHandler(logHandler)logger.setLevel(logging.INFO)  
# 结构化日志@app.middleware("http")async def log_requests(request: Request, call_next):    logger.info({        "event": "request_start",        "method": request.method,        "url": str(request.url),        "client_ip": request.client.host    })  
    try:        response = await call_next(request)        logger.info({            "event": "request_end",            "method": request.method,            "url": str(request.url),            "status_code": response.status_code        })        return response    except Exception as e:        logger.error({            "event": "request_error",            "method": request.method,            "url": str(request.url),            "error": str(e)        })        raise
```

## 总结

FastAPI 是一个功能强大且高效的 Web 框架，特别适合构建现代 API。本教程涵盖了从基础到高级的各个方面：

1. **快速启动**：简单的安装和最小示例
2. **核心概念**：路径操作、请求/响应处理、数据验证
3. **高级特性**：依赖注入、数据库集成、认证授权
4. **生产就绪**：中间件、监控、部署配置

通过合理利用 FastAPI 的特性，可以构建出高性能、易于维护且文档完善的 API 服务。建议在实际项目中根据具体需求选择合适的模式和最佳实践。