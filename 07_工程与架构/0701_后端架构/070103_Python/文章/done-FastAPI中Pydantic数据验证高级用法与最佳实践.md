> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/FastAPI生产边界与降权准则|FastAPI生产边界与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python架构实现路线|Python架构实现路线]]
---
title: FastAPI中Pydantic数据验证高级用法与最佳实践
author: 测开工程师的烦恼
date:
url: https://mp.weixin.qq.com/s?__biz=MzA5NzM0NjU1Nw==&mid=2647871811&idx=1&sn=b508e5332a248267becb333d2a33a49a&chksm=8925b1553424f9dfedaf9ac46cf280723a4022401c42608d6641f3e55720188efc3059fbe4f9&mpshare=1&scene=24&srcid=0415y5jeR9en9siiAN7uYMAt&sharer_shareinfo=47f31187c83e36a56d4e88c7567c76bb&sharer_shareinfo_first=47f31187c83e36a56d4e88c7567c76bb#rd
---

## 导读

你去餐厅点餐，服务员会问你：

* "您几位？"（必填）
* "有忌口吗？"（可选）
* "手机号码留一下？"（格式要正确）

如果信息不对，服务员会当场指出，而不是等菜做好了才发现问题。

**Pydantic 就是这个"服务员"**——在数据进入你的接口之前，帮你检查格式、类型、范围，不合格的当场打回去。

这篇文章，咱们把 Pydantic 的数据验证玩明白，从此告别脏数据！

---

## 一、基础验证：类型检查只是起点

### 1.1 最简单的模型

```
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str

# 正确数据
user = User(name="张三", age=25, email="zhangsan@example.com")
print(user)  # name='张三' age=25 email='zhangsan@example.com'

# 错误数据
user = User(name="张三", age="不是数字", email="zhangsan@example.com")
# ValidationError: 1 validation error for User
# age
#   value is not a valid integer
```

**简单说**：类型不对，直接报错，不会流到业务代码里。

### 1.2 可选字段与默认值

```
from typing import Optional
from pydantic import BaseModel

class Product(BaseModel):
    name: str                    # 必填
    price: float                 # 必填
    description: Optional[str] = None  # 可选，默认 None
    stock: int = 0              # 可选，默认 0

# 只传必填字段
product = Product(name="苹果手机", price=5999.0)
print(product.stock)  # 0
```

---

## 二、进阶验证：Field 的强大功能

### 2.1 数值范围限制

```
from pydantic import BaseModel, Field

class Score(BaseModel):
    # ge = greater than or equal, le = less than or equal
    math: int = Field(..., ge=0, le=100, description="数学成绩")
    english: int = Field(..., ge=0, le=100, description="英语成绩")

# 正确
score = Score(math=85, english=90)

# 错误：超出范围
score = Score(math=150, english=90)
# ValidationError: math 必须在 0-100 之间
```

### 2.2 字符串长度与正则

```
from pydantic import BaseModel, Field

class UserRegister(BaseModel):
    # min_length/max_length 限制长度
    username: str = Field(..., min_length=3, max_length=20, description="用户名")
    
    # regex 正则匹配
    phone: str = Field(..., pattern=r"^1[3-9]\d{9}$", description="手机号")
    
    # 密码复杂度要求
    password: str = Field(
        ..., 
        min_length=8, 
        max_length=32,
        description="密码"
    )

# 测试
try:
    user = UserRegister(username="ab", phone="13800138000", password="12345678")
except Exception as e:
    print(e)
    # username 长度必须在 3-20 之间
```

### 2.3 列表长度限制

```
from typing import List
from pydantic import BaseModel, Field

class OrderCreate(BaseModel):
    items: List[str] = Field(..., min_length=1, max_length=100, description="商品列表")
    tags: List[str] = Field(default=[], max_length=10, description="标签")

# 正确
order = OrderCreate(items=["手机", "充电器"])

# 错误：items 不能为空
order = OrderCreate(items=[])
# ValidationError: items 至少需要一个元素
```

---

## 三、自定义验证器：Validator

### 3.1 字段级验证

```
from pydantic import BaseModel, field_validator

class User(BaseModel):
    name: str
    age: int
    confirm_password: str
    password: str

    @field_validator("name")
    @classmethod
    def name_must_not_be_numeric(cls, v: str) -> str:
        """用户名不能全是数字"""
        if v.isdigit():
            raise ValueError("用户名不能全是数字")
        return v

    @field_validator("confirm_password")
    @classmethod
    def passwords_match(cls, v: str, info) -> str:
        """确认密码要与密码一致"""
        if "password" in info.data and v != info.data["password"]:
            raise ValueError("两次密码不一致")
        return v

# 测试
try:
    user = User(name="12345", age=20, password="secret", confirm_password="secret")
except Exception as e:
    print(e)  # 用户名不能全是数字
```

### 3.2 模型级验证

```
from pydantic import BaseModel, model_validator
from typing import Literal

class DateRange(BaseModel):
    start_date: str
    end_date: str
    range_type: Literal["day", "week", "month"]

    @model_validator(mode="after")
    def check_date_range(self):
        """验证结束日期必须晚于开始日期"""
        if self.end_date < self.start_date:
            raise ValueError("结束日期必须晚于开始日期")
        return self

# 测试
try:
    date_range = DateRange(start_date="2024-03-01", end_date="2024-02-01", range_type="day")
except Exception as e:
    print(e)  # 结束日期必须晚于开始日期
```

---

## 四、在 FastAPI 中使用验证

### 4.1 请求体验证

```
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
from typing import Literal

app = FastAPI(title="Pydantic 验证示例")

class UserCreate(BaseModel):
    """用户创建请求模型"""
    username: str = Field(..., min_length=3, max_length=20, pattern=r"^[a-zA-Z0-9_]+$")
    email: str = Field(..., pattern=r"^[\w\.-]+@[\w\.-]+\.\w+$")
    age: int = Field(..., ge=18, le=120, description="年龄必须在 18-120 之间")
    role: Literal["user", "admin", "vip"] = Field(default="user")

@app.post("/users", summary="创建用户")
async def create_user(user: UserCreate):
    """
    创建新用户
    
    - 用户名：3-20 位，只允许字母数字下划线
    - 邮箱：标准邮箱格式
    - 年龄：18-120 岁
    - 角色：user/admin/vip
    """
    return {
        "code": 0,
        "msg": "创建成功",
        "data": {
            "id": 1,
            "username": user.username,
            "email": user.email,
            "age": user.age,
            "role": user.role
        }
    }
```

### 4.2 查询参数验证

```
from fastapi import FastAPI, Query
from typing import Annotated

app = FastAPI()

@app.get("/products", summary="查询商品列表")
async def list_products(
    page: Annotated[int, Query(ge=1, description="页码，从1开始")] = 1,
    size: Annotated[int, Query(ge=1, le=100, description="每页数量")] = 20,
    min_price: Annotated[float, Query(ge=0, description="最低价格")] = 0,
    max_price: Annotated[float, Query(ge=0, description="最高价格")] = 999999
):
    """
    查询商品列表，支持分页和价格筛选
    """
    if max_price < min_price:
        raise HTTPException(400, "最高价格不能低于最低价格")
    
    return {
        "code": 0,
        "data": {
            "page": page,
            "size": size,
            "filters": {
                "min_price": min_price,
                "max_price": max_price
            }
        }
    }
```

### 4.3 路径参数验证

```
from fastapi import FastAPI, Path
from typing import Annotated

app = FastAPI()

@app.get("/users/{user_id}", summary="获取用户信息")
async def get_user(
    user_id: Annotated[int, Path(ge=1, description="用户ID，必须大于0")]
):
    """
    根据ID获取用户信息
    """
    return {
        "code": 0,
        "data": {"id": user_id, "name": "张三"}
    }
```

---

## 五、嵌套模型与复杂数据结构

### 5.1 嵌套模型验证

```
from typing import List, Optional
from pydantic import BaseModel, Field
from decimal import Decimal

class Address(BaseModel):
    """地址模型"""
    province: str = Field(..., min_length=2, max_length=20)
    city: str = Field(..., min_length=2, max_length=20)
    detail: str = Field(..., min_length=5, max_length=200)
    zip_code: Optional[str] = Field(None, pattern=r"^\d{6}$")

class OrderItem(BaseModel):
    """订单项"""
    product_id: int = Field(..., ge=1)
    product_name: str = Field(..., min_length=1, max_length=100)
    quantity: int = Field(..., ge=1, le=999)
    unit_price: Decimal = Field(..., gt=0, decimal_places=2)
    
    @property
    def total_price(self) -> Decimal:
        return self.unit_price * self.quantity

class OrderCreate(BaseModel):
    """创建订单请求"""
    user_id: int = Field(..., ge=1)
    items: List[OrderItem] = Field(..., min_length=1, max_length=50)
    address: Address
    remark: Optional[str] = Field(None, max_length=500)
    
    @property
    def total_amount(self) -> Decimal:
        return sum(item.total_price for item in self.items)

# 使用示例
order_data = {
    "user_id": 10001,
    "items": [
        {
            "product_id": 1,
            "product_name": "iPhone 15",
            "quantity": 1,
            "unit_price": "5999.00"
        },
        {
            "product_id": 2,
            "product_name": "AirPods Pro",
            "quantity": 1,
            "unit_price": "1999.00"
        }
    ],
    "address": {
        "province": "广东",
        "city": "深圳",
        "detail": "南山区科技园xxx号",
        "zip_code": "518000"
    },
    "remark": "请尽快发货"
}

order = OrderCreate(**order_data)
print(f"订单总金额: ¥{order.total_amount}")  # 订单总金额: ¥7998.00
```

### 5.2 FastAPI 中使用嵌套模型

```
from fastapi import FastAPI
from typing import List

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

class Order(BaseModel):
    items: List[Item]
    total: float

@app.post("/orders")
async def create_order(order: Order):
    # FastAPI 会自动验证整个嵌套结构
    calculated_total = sum(item.price for item in order.items)
    
    if abs(calculated_total - order.total) > 0.01:
        raise HTTPException(400, "订单金额计算不正确")
    
    return {"status": "success", "order_id": "ORD-123456"}
```

---

## 六、完整项目示例

### 项目结构

```
myproject/
├── main.py           # FastAPI 入口
├── models.py         # Pydantic 模型
├── requirements.txt  # 依赖
└── README.md
```

### requirements.txt

```
fastapi==0.104.1
uvicorn[standard]==0.24.0
pydantic==2.5.0
pydantic[email]       # 邮箱验证支持
```

### models.py

```
from pydantic import BaseModel, Field, EmailStr, field_validator
from typing import Optional, Literal
from datetime import date

class UserCreate(BaseModel):
    """用户注册请求模型"""
    username: str = Field(
        ...,
        min_length=3,
        max_length=20,
        pattern=r"^[a-zA-Z][a-zA-Z0-9_]*$",
        description="用户名：字母开头，只允许字母数字下划线"
    )
    email: EmailStr = Field(..., description="邮箱地址")
    password: str = Field(
        ...,
        min_length=8,
        max_length=32,
        description="密码：8-32位"
    )
    confirm_password: str = Field(..., description="确认密码")
    birthday: Optional[date] = Field(None, description="出生日期")
    gender: Literal["male", "female", "other"] = Field(default="other")
    
    @field_validator("confirm_password")
    @classmethod
    def passwords_match(cls, v: str, info) -> str:
        if "password" in info.data and v != info.data["password"]:
            raise ValueError("两次输入的密码不一致")
        return v
    
    @field_validator("birthday")
    @classmethod
    def birthday_must_be_past(cls, v: Optional[date]) -> Optional[date]:
        if v and v > date.today():
            raise ValueError("出生日期不能是将来")
        return v

class UserResponse(BaseModel):
    """用户响应模型"""
    id: int
    username: str
    email: str
    age: Optional[int] = None
    gender: str
    
    class Config:
        from_attributes = True
```

### main.py

```
from fastapi import FastAPI, HTTPException
from models import UserCreate, UserResponse

app = FastAPI(title="用户系统", version="1.0.0")

# 模拟数据库
users_db = []

@app.post("/register", response_model=UserResponse, summary="用户注册")
async def register(user: UserCreate):
    """
    用户注册接口
    
    - **username**: 3-20位，字母开头
    - **email**: 有效邮箱格式
    - **password**: 8-32位
    - **birthday**: 可选，不能是将来日期
    """
    # 检查用户名是否已存在
    if any(u["username"] == user.username for u in users_db):
        raise HTTPException(400, "用户名已存在")
    
    # 创建用户
    new_user = {
        "id": len(users_db) + 1,
        "username": user.username,
        "email": user.email,
        "gender": user.gender,
        "age": (date.today() - user.birthday).days // 365 if user.birthday else None
    }
    users_db.append(new_user)
    
    return new_user

@app.get("/users/{user_id}", response_model=UserResponse, summary="获取用户信息")
async def get_user(user_id: int):
    """根据ID获取用户信息"""
    user = next((u for u in users_db if u["id"] == user_id), None)
    if not user:
        raise HTTPException(404, "用户不存在")
    return user

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 测试

```
# 1. 安装依赖
pip install -r requirements.txt

# 2. 启动服务
python main.py

# 3. 测试注册（成功）
curl -X POST "http://localhost:8000/register" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "email": "john@example.com",
    "password": "SecurePass123",
    "confirm_password": "SecurePass123",
    "birthday": "1990-01-01",
    "gender": "male"
  }'

# 4. 测试注册（密码不一致）
curl -X POST "http://localhost:8000/register" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "test_user",
    "email": "test@example.com",
    "password": "SecurePass123",
    "confirm_password": "WrongPass",
    "gender": "female"
  }'
# 返回：{"detail":[{"msg":"两次输入的密码不一致"}]}
```

---

## 总结

今天我们一起学习了 Pydantic 数据验证的方方面面：

* **基础验证**：类型检查、可选字段、默认值
* **Field 验证**：数值范围、字符串长度、正则匹配、列表长度
* **自定义验证器**：`@field_validator` 字段级验证、`@model_validator` 模型级验证
* **FastAPI 集成**：请求体、查询参数、路径参数的验证
* **嵌套模型**：复杂数据结构的验证
* **最佳实践**：分层验证、自定义错误消息、组合验证器

**记住这句话**：数据验证前置，能让你的接口更健壮，业务代码更干净。

---

**每日踩一坑，生活更轻松。**

> Pydantic 的 `default=[]` 如果用于可变对象（如列表），会导致多个实例共享同一个列表！正确做法是使用 `Field(default_factory=list)`。

本期分享就到这里啦，祝君在测开之路越走越远，越走越顺。