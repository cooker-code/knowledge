---
title: FastAPI 实战 —— OAuth2 密码流（Resource Owner Password）入门（第六篇）
author: D先生的自白
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYzNTE0NDIyMg==&mid=2247483684&idx=1&sn=9800d3eeb966c71e598d0ada936f9581&chksm=f14d2416c780e1d1263aa511375d45df3a394c8d7b74bbbb86b4275ee170ced95b5c7194b555&mpshare=1&scene=24&srcid=1115M7nkie47WdPpdYuDFyKY&sharer_shareinfo=2101bd92380de9b2eee667610cb0e444&sharer_shareinfo_first=2101bd92380de9b2eee667610cb0e444#rd
---

> 本文将介绍如何在 FastAPI 中使用 OAuth2 密码流（password grant）进行认证与授权。

---

## 🎯 阅读收获

通过本教程，你将掌握：

✅ 理解 OAuth2 密码流的基本概念与适用场景； ✅ 在 FastAPI 中使用 `OAuth2PasswordBearer` 与 `OAuth2PasswordRequestForm` 实现登录与受保护接口； ✅ 如何在本地通过 curl 测试认证流程。

---

## 🧩 OAuth2 密码流是什么？

密码流（Resource Owner Password Credentials Grant）允许用户直接向授权服务器提交用户名和密码，换取访问令牌（access token）。流程简要：

1. 客户端向 `/token`（token 端点）发送用户名和密码；
2. 服务器校验凭据，返回访问令牌；
3. 客户端使用该令牌访问受保护的资源（在 HTTP 头中携带 `Authorization: Bearer <token>`）。

注意：密码流通常只适用于第一方客户端（trusted clients），或在受控环境用于学习/调试。生产环境建议使用 OAuth2 的授权码（authorization code）流配合 PKCE 或使用 JWT + 刷新令牌。

---

## 与传统的HTTP 基础认证有什么区别

| 特性 | HTTP Basic | OAuth2 |
| --- | --- | --- |
| 传输方式 | 用户名密码每次请求都发送 | 使用token，不暴露密码 |
| 加密 | Base64编码（可逆） | Token（不可逆） |
| 过期机制 | 无内置过期 | 支持token过期和刷新 |
| 作用域控制 | 无 | 精细的权限控制 |

## ⚙️ 在 FastAPI 中的核心类

* `OAuth2PasswordBearer(tokenUrl="token")`：声明依赖，告诉 FastAPI 哪个路径用于获取 token。
* `OAuth2PasswordRequestForm`：用于从表单中读取 `username` 与 `password`。
* `Depends`：依赖注入，用于把当前用户或 token 注入路由函数中。

---

## 💻 代码示例（完整、可运行）

由于涉及到表单信息，所以需要额外安装`python-multipart`模块，通过命令`pip install python-multipart`即可安装。

下面的示例是简化版本，便于快速理解流程：

```
from fastapi import FastAPI, Depends, HTTPException, status  
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm  
from pydantic import BaseModel  
from typing import Optional  
  
app = FastAPI()  
  
# 声明 token 获取地址（用于 OpenAPI 文档）  
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/token")  
  
class User(BaseModel):  
    username: str  
    email: Optional[str] = None  
    disabled: Optional[bool] = False  
  
# 模拟数据库  
fake_users_db = {  
    "alice": {"username": "alice", "email": "alice@example.com", "hashed_password": "fakehashedsecret", "disabled": False},  
}  
  
def fake_hash_password(password: str) -> str:  
    return"fakehashed" + password  
  
def get_user(db, username: str):  
    if username in db:  
        user_dict = db[username]  
        return User(**user_dict)  
  
def authenticate_user(db, username: str, password: str):  
    user = get_user(db, username)  
    ifnot user:  
        returnFalse  
    # 简化验证：把明文加上 fake 前缀与存储的对比  
    if fake_hash_password(password) != db[username]["hashed_password"]:  
        returnFalse  
    return user  
  
@app.post("/token")  
def login_for_access_token(form_data: OAuth2PasswordRequestForm = Depends()):  
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)  
    ifnot user:  
        raise HTTPException(  
            status_code=status.HTTP_401_UNAUTHORIZED,  
            detail="Incorrect username or password",  
            headers={"WWW-Authenticate": "Bearer"},  
        )  
    # 注意：真实场景应返回 JWT 或随机令牌并保存服务端状态/签名  
    access_token = user.username  # 简化：使用用户名作为 token  
    return {"access_token": access_token, "token_type": "bearer"}  
  
@app.get("/users/me")  
def read_users_me(token: str = Depends(oauth2_scheme)):  
    # oauth2_scheme 会从 Authorization: Bearer <token> 中解析出 token 字符串  
    user = get_user(fake_users_db, token)  
    ifnot user:  
        raise HTTPException(status_code=401, detail="Invalid authentication credentials")  
    if user.disabled:  
        raise HTTPException(status_code=400, detail="Inactive user")  
    return user
```

保存为 `main.py` 并启动：

```
uvicorn main:app --reload
```

---

## 🔍 如何测试

1. 使用 Swagger UI（推荐）

* 打开 `http://127.0.0.1:8000/docs`。
* 找到 `POST /token`，点开右侧的 "Try it out"，会自动生成表单（username/password）。用户名alice，密码secret，提交后会得到 `access_token`。
* 在页面右上角点 Authorize，输入 `Bearer <access_token>`（Swagger 有自动填充），然后访问受保护的 `GET /users/me`。

2. 使用 curl（命令行）

* 请求 token：

```
curl -X POST "http://127.0.0.1:8000/token" -H "Content-Type: application/x-www-form-urlencoded" -d "username=alice&password=secret"
```

* 使用 token 访问受保护接口（假设返回的 token 为 alice）：

```
curl -H "Authorization: Bearer alice" http://127.0.0.1:8000/users/me
```

（示例中 token 就是用户名，仅作演示）

---

## ⚠️ 常见问题与注意事项

* 密码流并不安全用于第三方客户端或公开客户端 —— 只适合可信任的第一方客户端。
* 切勿在生产中把用户名当作 token；应使用签名的 JWT 或服务端存储的随机令牌并设置过期时间。
* 所有带凭据的请求必须使用 HTTPS，避免凭据被中间人窃取。
* 在比较密码或敏感字符串时使用 `secrets.compare_digest` 来防止时序攻击。
* 如果使用 JWT，务必：设置合适的签名算法、过期时间（exp）、以及刷新令牌策略；并验证令牌的签名与声明（claims）。

---

## ✅ 常见扩展建议（下一步学习）

1. 把示例中的简化 token 替换为 JWT（使用 `pyjwt` 或 `python-jose`），示例应包含签发（iss）、过期（exp）与用户 id（sub）。
2. 实现刷新令牌（refresh token）机制以延长会话并可在服务器端撤销访问。
3. 研究 OAuth2 授权码（Authorization Code）流 + PKCE，用于第三方/浏览器端应用。
4. 引入数据库（例如 SQLite / PostgreSQL）存储用户与令牌元数据，并加入速率限制与审计日志。

---

## 小结

本文介绍了 FastAPI 中使用 OAuth2 密码流的基础步骤与关键点，给出可运行的最小示例并演示如何通过 Swagger UI 与 curl 测试。密码流便于理解 OAuth2 的工作原理，但生产环境应使用更安全的令牌方案（如 JWT + 刷新令牌或授权码流）。