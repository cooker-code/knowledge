> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: FastAPI 入门教程10：响应模型详解
author: 点石成金x
date:
url: https://mp.weixin.qq.com/s?__biz=MzYyMTkyNjc0Ng==&mid=2247483740&idx=3&sn=9241389e93b46889439c5c6c4b0f9cad&chksm=fedbb1639e06f37772995c35ebd03323de3c24a973ee16a1d63f3afc713b77d4fd0822196d85&mpshare=1&scene=24&srcid=1128YDevkpo7U0WfzLVUrELx&sharer_shareinfo=533a2c5c22d36d3d96827a20d025b93e&sharer_shareinfo_first=533a2c5c22d36d3d96827a20d025b93e#rd
---

#

FSI未来超级个体

执行力总动员
思多乱其智，行者皆披靡

 

 

 

 

# 🚀 FastAPI 入门教程10：响应模型详解

## 📚 目录

1. 1. 🔍 什么是响应模型？ FastAPI的“数据魔方”
2. 2. ⚡ 快速上手基础用法 秒会核心操作
3. 3. 🛡️ 安全实战：过滤敏感数据 守护你的API生命线
4. 4. 🎯 高级技巧：响应模型精细控制 像切蛋糕一样精准
5. 5. 🏗️ 企业级实践：构建通用响应模型 告别混乱，走向规范
6. 6. 💡 总结与最佳实践

---

## 🔍 什么是响应模型？FastAPI的“数据魔方” 🎲

想象一下，你的API后端是一个数据宝库，但直接把所有宝藏都展示出来，既不安全也显得杂乱。FastAPI的**响应模型**就是那个神奇的数据魔方，它能帮你：

* • **自动转换**：把数据库对象、字典等任何数据，自动整理成规范的JSON格式。🔄
* • **智能校验**：在数据出发前，做最后一次“安检”，确保万无一失。✅
* • **生成文档**：自动为你的API文档生成漂亮的数据模型图，别人一看就懂。📖
* • **安全过滤**：像带梳子的梳子，自动过滤掉你不想暴露的敏感信息。🕵️‍♂️

最重要的是，它能**将输出数据严格限制在模型定义内**，这有多重要？我们马上就见分晓！

看看它的基本写法，`response_model`是路径操作装饰器（如`@app.get`、`@app.post`）的一个参数，而不是函数返回值的类型注解哦！

```
from pydantic import BaseModel


# 声明一个Item模型
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list[str] = []

# 使用response_model声明单个Item响应
@app.post("/items/", response_model=Item)
async def create_item(item: Item) -> Any:
    # 函数内部可能返回dict、数据库对象等
    # FastAPI会自动用response_model处理它
    return item

# 也可以返回一个模型列表！
@app.get("/items/", response_model=list[Item])
async def read_items() -> Any:
    return [
        {"name": "Portal Gun", "price": 42.0},
        {"name": "Plumbus", "price": 32.0},
    ]
```

---

## 🛡️ 安全实战：过滤敏感数据，守护你的API生命线 🔒

这是响应模型最核心、最不能错过的功能！想象一个用户注册场景，我们用一个模型接收包含明文密码的输入：

⚠️ **危险示范：千万不要在生产环境这样做！** ⚠️

```
class UserIn(BaseModel):
    username: str
    password: str  # 哦吼，明文密码字段
    email: EmailStr
    full_name: Union[str, None] = None

# 错误示范：用输入模型同时作为响应模型
@app.post("/user/")
async def create_user(user: UserIn) -> UserIn:
    return user  # 💣 密码被直接返回了！
```

`请求体：`

`响应体：`

当你用这个API创建用户时，API会把包含`password`的完整`UserIn`对象直接返回给客户端。这可能泄露用户的私密信息！**永远不要在响应中返回密码！**

✅ **正确姿势：巧用输出模型，实现数据“金蝉脱壳”** ✅

解决方案是创建两个模型：一个用于接收数据（带密码），一个用于输出数据（不带密码）。

```
# 输入模型：包含密码
class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: str | None = None

# 输出模型：不包含密码
class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: str | None = None

# 在装饰器中指定输出模型
@app.post("/user/", response_model=UserOut)
async def create_user(user: UserIn) -> Any:
    # 函数返回的是完整的user对象（包含密码）
    return user
```

见证奇迹的时刻到了！即便路径函数`return`的`user`对象里包含`password`，但FastAPI看到`response_model=UserOut`后，会立刻启动“过滤器”，只把`UserOut`模型中声明的字段(`username`, `email`, `full_name`)返回给客户端，`password`字段被无情地过滤掉了！🔥

返回给客户端的清爽JSON将是这样的：
`{"username": "zhangsan", "email": "user@example.com", "full_name": "张三"}`

---

## 🎯 高级技巧：响应模型精细控制，像切蛋糕一样精准 🍰

FastAPI的响应模型还有更多神奇的参数，让你对输出有像素级的控制力！

#### **1. `response_model_exclude_unset=True`：排除未set的默认值**

当你的模型字段有默认值时，除非你明确给它赋了值，否则它就不出现在最终的JSON里，完美优化响应体积。

```
class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: float = 10.5  # 默认税率10.5
    tags: List[str] = []

items_database = {
    "foo": {"name": "Foo", "price": 50.2}, # 只有price，没有显式设置tax
    "bar": {"name": "Bar", "description": "The Bar fighters", "price": 62, "tax": 20.2},
}

@app.get("/items/{item_id}/name", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    return items_database[item_id]
```

**效果对比：**

* • 请求`/items/foo/`，响应中**不会**包含`tax`和`tags`，因为它们没有显式设置值：

  ```
  {"name": "Foo", "price": 50.2}
  ```
* • 请求`/items/bar/`，响应中**会**包含`tax`，因为它被显式设置成了`20.2`：

  ```
  {"name": "Bar", "description": "The Bar fighters", "price": 62.0, "tax": 20.2}
  ```

#### **2. `response_model_include` & `response_model_exclude`：按需索取/排除**

如果你只有一个庞大的模型，但只想临时在某个接口中返回部分字段，这两个参数是快捷方式。

```
# 只返回 name 和 description 字段
@app.get("/items/{item_id}/name", response_model=Item, response_model_include={"name", "description"})
async def read_item_name(item_id: str):
    return items_database[item_id]

# 返回除 tax 外的所有字段
@app.get("/items/{item_id}/public", response_model=Item, response_model_exclude={"tax"})
async def read_item_public_data(item_id: str):
    return items_database[item_id]
```

**注意：** 虽然方便，但从长远维护和API文档清晰度考虑，**创建多个专门的输出模型（如`UserPublic`, `UserPrivate`）是更好的实践**。

---

## 🏗️ 企业级实践：构建通用响应模型，告别混乱 🏢

在实际项目中，我们通常不会裸奔式地返回数据模型。一个优雅的、统一的响应结构至关重要。

```
from typing import Generic, TypeVar
from pydantic import BaseModel

T = TypeVar('T')

class APIResponse(BaseModel, Generic[T]):
    """
    通用API响应格式
    """
    code: int = 200
    message: str = "success"
    data: T | None = None

# 使用这个通用模型
@app.get("/items/{item_id}", response_model=APIResponse[Item])
async def read_item_with_wrapper(item_id: str):
    item_data = {"name": "Portal Gun", "price": 42.0}
    return APIResponse(data=Item(**item_data))
```

现在，你API的响应格式永远都是：

```
{
  "code": 200,
  "message": "success",
  "data": {
    "name": "Portal Gun",
    "price": 42.0
  }
}
```

这种结构清晰、易于前端处理，并且让所有接口风格统一，是大型项目的必备之选！当然还可以添加一些其他的字段比如时间戳，异常的traceid等

---

## 💡 总结与最佳实践 🏆

1. 1. **永远分离敏感数据**：这是响应模型最重要的价值！为输入和输出分别创建模型。🔐
2. 2. **善用`exclude_unset`**：让你的API响应更轻量，不返回无意义的默认值。🧹
3. 3. **拥抱通用响应结构**：使用泛型构建`APIResponse`，让所有接口整齐划一。📦
4. 4. **首选多个模型而非`include/exclude`**：虽然`include/exclude`很方便，但维护成本和文档清晰度上，专门的模型更胜一筹。
5. 5. **查阅自动生成的文档**：你的`response_model`设置得是否正确，去`/docs`页面一看便知！它是最好的“体检报告”。🔍

响应模型是FastAPI赋予开发者的一把利剑，善用它，你的API将变得既强大又可靠，既优雅又安全。现在就去你的项目中，打造属于你的完美响应模型吧！🚀

---

🌈 期待在这个技术快速变革的时代，与各位一起探索技术人的成长之道。
👀 欢迎关注我的公众号，让我们在交流中共同进步。

📱 **关注公众号，获取每天技术、认知干货**