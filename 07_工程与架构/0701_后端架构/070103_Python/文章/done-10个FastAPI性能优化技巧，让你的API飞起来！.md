> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/FastAPI生产边界与降权准则|FastAPI生产边界与降权准则]]
---
title: 10个FastAPI性能优化技巧，让你的API飞起来！
author: 点石成金x
date:
url: https://mp.weixin.qq.com/s?__biz=MzYyMTkyNjc0Ng==&mid=2247483677&idx=1&sn=e3acd467e87f44c3cbdb7d70b297bb52&chksm=fe3431e13d0d7be8851ea3e516b73e2076a778bc321561da33c02a17ca9d2f3f40542a8076d2&mpshare=1&scene=24&srcid=1122YRdfmJmDzWVdlUJGsS0S&sharer_shareinfo=409a8eaa2e6fbe3038e0c6c0ec99e604&sharer_shareinfo_first=409a8eaa2e6fbe3038e0c6c0ec99e604#rd
---

FSI未来超级个体

执行力总动员
思多乱其智，行者皆披靡

 

# 🚀10个FastAPI性能优化技巧，让你的API飞起来！

> 资深开发者总结，新手也能快速上手

今天为大家带来10个实用的FastAPI性能优化技巧。这些技巧都是从实际项目中总结出来的，能够显著提升你的API性能！

## 🎯 技巧1：安装性能增强包

默认情况下，Uvicorn没有包含`uvloop`和`httptools`，这两个包能显著提升性能：

```
pip install uvloop httptools
```

安装后，Uvicorn会自动使用它们。

⚠️ **注意**：Windows用户无法安装`uvloop`。如果你在本地使用Windows，在生产环境使用Linux，可以通过环境标记来避免在Windows上安装。

## ⚡ 技巧2：谨慎使用非异步函数

在FastAPI中使用非异步函数会有性能损失，因为FastAPI会使用线程池来运行这些函数。

```
# 不推荐 - 会在线程池中运行
def sync_function():
    return {"message": "Hello"}

# 推荐 - 直接在事件循环中运行
async def async_function():
    return {"message": "Hello"}
```

线程池默认只有40个线程，如果全部占用，应用会被阻塞。你可以通过以下代码调整线程数：

```
import anyio
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    limiter = anyio.to_thread.current_default_thread_limiter()
    limiter.total_tokens = 100  # 增加到100个线程
    yield

app = FastAPI(lifespan=lifespan)
```

## 🔄 技巧3：WebSocket使用async for循环

很多网络示例使用`while True`来处理WebSocket消息，但实际上使用`async for`更优雅：

```
# 传统写法
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message: {data}")

# 推荐写法
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    async for data in websocket.iter_text():
        await websocket.send_text(f"Message: {data}")
```

`async for`会自动处理WebSocket断开连接异常，代码更简洁。

## 🧪 技巧4：使用HTTPX的AsyncClient进行测试

既然你的应用使用异步函数，测试时也应该使用异步客户端：

```
# 同步测试客户端（不推荐）
from starlette.testclient import TestClient
client = TestClient(app)
response = client.get("/")

# 异步测试客户端（推荐）
import anyio
from httpx import AsyncClient, ASGITransport

async def main():
    async with AsyncClient(transport=ASGITransport(app=app)) as client:
        response = await client.get("/")
        assert response.status_code == 200

anyio.run(main)
```

## 🏗️ 技巧5：使用Lifespan State替代app.state

新的lifespan state提供了更标准的方式来管理需要在启动时创建的对象：

```
from typing import TypedDict
from httpx import AsyncClient

class State(TypedDict):
    client: AsyncClient

@asynccontextmanager
async def lifespan(app: FastAPI):
    async with AsyncClient() as client:
        yield {"client": client}

app = FastAPI(lifespan=lifespan)

@app.get("/")
async def read_root(request: Request):
    client = request.state.client
    response = await client.get("/")
    return response.json()
```

## 🐛 技巧6：启用AsyncIO调试模式

要找出阻塞事件循环的端点，可以启用AsyncIO调试模式：

```
PYTHONASYNCIODEBUG=1 python main.py
```

当任务执行超过100ms时，Python会打印警告信息，帮助你定位性能瓶颈。

## 🚀 技巧7：实现纯ASGI中间件

虽然`BaseHTTPMiddleware`使用简单，但存在性能损失。对于高性能需求，建议实现纯ASGI中间件：

```
# 高性能的纯ASGI中间件实现
class PureASGIMiddleware:
    def __init__(self, app):
        self.app = app
  
    async def __call__(self, scope, receive, send):
        # 在调用应用之前处理请求
        await self.app(scope, receive, send)
        # 在应用响应之后处理响应
```

## 🧵 技巧8：注意依赖项的运行线程

如果依赖项是非异步函数，它会在线程中运行：

```
# 在线程中运行
def http_client(request: Request) -> AsyncClient:
    return request.state.client

# 在事件循环中运行
async def http_client(request: Request) -> AsyncClient:
    return request.state.client
```

可以通过监控线程使用情况来验证：

```
async def monitor_thread_limiter():
    limiter = current_default_thread_limiter()
    while True:
        print(f"当前使用线程数: {limiter.borrowed_tokens}")
        await anyio.sleep(1)
```

## 🧪 技巧9：使用pytest.mark.anyio进行测试

既然已经安装了anyio，就应该使用`pytest.mark.anyio`而不是`pytest.mark.asyncio`：

```
import pytest

@pytest.mark.anyio
async def test_async_function():
    # 测试代码
    pass

# 指定后端
@pytest.fixture
def anyio_backend():
    return "asyncio"  # 或者 "trio"
```

## 💡结语

这些技巧都是经过实践检验的，能够显著提升你的FastAPI应用性能。建议在实际项目中逐步应用，并根据具体情况进行调整。

如果你有更好的技巧，欢迎在评论区分享！🚀

 

---

🌈 期待在这个技术快速变革的时代，与各位一起探索技术人的成长之道。
👀 欢迎关注我的公众号，让我们在交流中共同进步。

📱 **关注公众号，获取每天技术、认知干货**