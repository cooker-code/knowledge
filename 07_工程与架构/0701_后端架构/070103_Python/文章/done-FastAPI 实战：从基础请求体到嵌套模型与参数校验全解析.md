> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: FastAPI 实战：从基础请求体到嵌套模型与参数校验全解析
author: 无法直视众平台
date:
url: https://mp.weixin.qq.com/s?__biz=MzI2MjAwMjY0Nw==&mid=2247484081&idx=2&sn=fe1790610ddf56b26d72d4c5650f4622&chksm=eb7e1c35dc44c299ad53a9a15efbcd493b49ebcb921f9008563f613c448bf2a25305253d5737&mpshare=1&scene=24&srcid=1103zMaLc3mTydSfLJQ4BOzy&sharer_shareinfo=1a4c9435f56dbd55d100febba3f28c6c&sharer_shareinfo_first=1a4c9435f56dbd55d100febba3f28c6c#rd
---

# 🚀 FastAPI 实战：从基础请求体到嵌套模型与参数校验全解析

> 用 Python 写 API，FastAPI 是目前最优雅、高效的选择之一。本文通过一个完整示例，带你深入理解 FastAPI 中 **请求体（Body）处理、参数校验、嵌套模型** 等核心功能。

---

## 🔧 背景介绍

FastAPI 是一个现代、快速（高性能）的 Web 框架，基于 Python 类型提示（Type Hints）自动生成 API 文档（Swagger / ReDoc），并内置数据校验和序列化功能。本文将通过一段实际代码，逐步解析 FastAPI 中处理请求体的多种方式。

---

## 📦 基础请求体：Pydantic 模型

首先，我们定义一个基础的 Pydantic 模型 `Item`：

```
class Item(BaseModel):
    user_id: str
    token: str
    timestamp: str
    article_id: Optional[str] = None
```

然后通过 POST 接口接收整个 JSON 对象：

```
@app.post("/action/")
def callback(item: Item):
    return item.dict()
```

✅ **特点**：

* • 客户端发送的 JSON 会自动映射为 `Item` 实例；
* • `article_id` 是可选字段，默认为 `None`；
* • FastAPI 自动校验字段类型，类型错误会返回 422 错误。

---

## 🧪 高级用法：直接使用 Body 参数

有时我们不想定义模型，而是直接从请求体中提取字段：

```
@app.post("/action/body")
def callbackbody(
    token: str = Body(...),
    user_id: int = Body(..., gt=10),  # 必须大于10
    timestamp: str = Body(...),
    article_id: str = Body(default=None),
):
    return { ... }
```

✅ **关键点**：

* • `Body(...)` 表示该字段**必须存在**；
* • `gt=10` 是 Pydantic 提供的**数值校验器**（greater than）；
* • 所有字段都从 JSON 根层级提取（非嵌套）。

> 💡 注意：这种方式适用于字段较少、逻辑简单的接口。字段多时建议使用模型。

---

## 🔄 可选 Body：全字段可为空

```
@app.post("/action/body2")
def callbackbody(
    token: str = Body(default=None),
    user_id: int = Body(default=None, gt=10),
    ...
):
```

此时所有字段都可省略。但注意：**如果 `user_id` 传了，就必须 >10**；如果不传，则为 `None`。

---

## 📦 嵌入模型 vs 非嵌入模型

FastAPI 默认将 Pydantic 模型作为 JSON 的**顶层对象**接收。但有时你希望字段直接平铺在 JSON 根层级，这时可用 `embed=True`：

```
class Itement(BaseModel):
    user_id: int = Body(..., gt=10, embed=True)  # ⚠️ 注意：embed 在字段上无效！
    token: str
    ...

@app.post("/action/body3")
def callbackbody(item: Itement = Body(embed=False)):
    return {"body": item}
```

> ❗ 重要提示：`embed` 参数**只能用于 `Body()` 装饰整个模型**，不能用于字段内部。
> 正确用法是：
>
> ```
> item: Itement = Body(..., embed=True)
> ```
>
> 这样客户端需发送：
>
> ```
> { "item": { "user_id": 15, "token": "xxx" } }
> ```
>
> 而 `embed=False`（默认）则直接：
>
> ```
> { "user_id": 15, "token": "xxx" }
> ```

---

## 👥 多模型联合：PUT 接口实战

FastAPI 支持在同一个接口中接收**多个 Pydantic 模型**：

```
@app.put("/items/")
async def update_item1111(item: ItemUser, user: User):
    return {"item": item, "user": user}
```

客户端需发送两个独立的 JSON 对象（FastAPI 会自动拆分）：

```
{
  "item": { "name": "Book", "price": 29.9 },
  "user": { "username": "alice" }
}
```

✅ 这是 FastAPI 的强大之处：**自动按模型名拆分请求体**！

---

## 🔗 模型嵌套 + 额外 Body 字段

更复杂的场景：既有嵌套模型，又有独立参数：

```
@app.put("/items/more")
async def update_item(
    item: Item, 
    user: User, 
    importance: int = Body(..., gt=0)
):
    return {"item": item, "user": user, "importance": importance}
```

请求体结构：

```
{
  "item": { ... },
  "user": { ... },
  "importance": 5
}
```

---

## 🧱 深度嵌套：集合类型与子模型

FastAPI 完美支持复杂嵌套结构：

```
class ItemUser3(BaseModel):
    name: str
    user: User
    tags: Set[str] = []          # 去重字符串集合
    users: List[User] = None     # 用户列表
```

接口使用：

```
@app.put("/items/body5")
async def update_item(item: ItemUser3, importance: int = Body(...)):
    return {"item": item, "importance": importance}
```

示例请求：

```
{
  "item": {
    "name": "Advanced API",
    "user": { "username": "admin" },
    "tags": ["fastapi", "python", "api"],
    "users": [
      { "username": "alice" },
      { "username": "bob" }
    ]
  },
  "importance": 10
}
```

✅ FastAPI 会自动：

* • 校验 `tags` 是否为字符串集合（自动去重）；
* • 校验 `users` 中每个元素是否符合 `User` 模型；
* • 对 `importance` 进行 `gt=0` 校验。

---

## 🧪 自动文档：Swagger UI

运行代码后，访问 `http://127.0.0.1:8000/docs`，你会看到：

* • 所有接口自动生成交互式文档；
* • 请求体结构清晰展示；
* • 字段校验规则（如 `gt=10`）也会显示；
* • 可直接在浏览器中测试接口！

---

## ✅ 总结

| 场景 | 推荐方式 |
| --- | --- |
| 简单 JSON 对象 | 使用 Pydantic 模型 |
| 少量字段 + 校验 | 直接 `Body()` 参数 |
| 多对象联合 | 多个模型参数 |
| 复杂嵌套结构 | 模型嵌套 + 集合类型 |
| 需要严格校验 | 利用 Pydantic 的 `gt`, `min_length` 等 |

FastAPI 的设计哲学是：**用类型提示表达一切**。你写的类型，就是文档、就是校验、就是接口契约。

---

## 📌 小贴士

* • 开发时开启 `reload=True`，代码修改自动重启；
* • 使用 `uvicorn` 启动服务，性能优异；
* • 结合 Docker + FastAPI，可轻松部署到生产环境（尤其适合你已在用 Docker Compose 的场景）。

---

**喜欢这篇文章？欢迎点赞、转发，关注我们获取更多 Python & FastAPI 实战技巧！**

> 🐍 技术栈：Python 3.12 + FastAPI + Pydantic + Uvicorn
> 📦 项目结构清晰，适合集成到你的微服务架构中