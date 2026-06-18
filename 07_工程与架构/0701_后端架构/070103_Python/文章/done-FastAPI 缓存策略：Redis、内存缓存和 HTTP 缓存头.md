> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: FastAPI 缓存策略：Redis、内存缓存和 HTTP 缓存头
author: 代码麻辣烫
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0NjY0MTc0NA==&mid=2247497404&idx=1&sn=84ea9dea8c9b9b43fc72d5edca16b470&chksm=c277beec4c93edbe16d6ad1517d2c9b61a3ff74ee1b9ec094c7d124518f8235abac4c0f1fdbd&mpshare=1&scene=24&srcid=1108zTkY3dJfb71VuhoumBtu&sharer_shareinfo=7531457bed1ae112849945ab4e14e3e9&sharer_shareinfo_first=7531457bed1ae112849945ab4e14e3e9#rd
---

缓存看似简单，直到你意识到有三层缓存需要考虑。我见过一些团队浪费时间在可以用内存缓存的数据上实现 Redis 缓存，或者忘记 HTTP 缓存头，从而错失客户端优化的机会。

本指南涵盖实用的缓存策略：何时使用每一层缓存、如何实现它们以及真实的性能对比。

## 三层缓存

1. **HTTP 缓存头**（浏览器/CDN 缓存）——免费，无需服务器响应
2. **内存缓存**（应用实例）——快速，但不跨实例共享
3. **Redis 缓存**（分布式）——跨实例持久化，共享状态

大多数生产应用都需要这三层缓存。它们是互补的，而不是相互替代的。

## 第一层：HTTP 缓存头（被忽视的缓存）

这是最便宜的优化方式。让浏览器和 CDN 缓存你的响应，这样它们就永远不会击中你的服务器。

### 基本 Cache-Control 示例

```
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/products/{product_id}")
asyncdef get_product(product_id: int, response: Response):
    """很少变化的产品数据"""
    product = {
        "id": product_id,
        "name": "Widget",
        "price": 99.99
    }

    response.headers["Cache-Control"] = "public, max-age=3600"

    return product
```

### 常见的 Cache-Control 值

* `max-age=3600`：缓存 3600 秒（1 小时）
* `public`：任何缓存都可以存储（浏览器、CDN、代理）
* `private`：仅浏览器缓存，不包括 CDN
* `no-cache`：必须在使用前与服务器重新验证
* `no-store`：永不缓存此响应

### 使用 ETag 实现智能重新验证

ETag 让浏览器能够在不重新下载的情况下重用缓存数据。如果数据没有变化，服务器会返回 `304 Not Modified`，且没有正文。

```
from fastapi import FastAPI, Response, Request
from fastapi.responses import Response as FastAPIResponse
import hashlib
import json

app = FastAPI()

@app.get("/users/{user_id}")
asyncdef get_user(user_id: int, request: Request):
    """支持 ETag 的用户数据"""
    user_data = {
        "id": user_id,
        "name": "Alice",
        "email": "alice@example.com"
    }

    data_str = json.dumps(user_data, sort_keys=True)
    etag = f'"{hashlib.md5(data_str.encode()).hexdigest()}"'

    client_etag = request.headers.get("if-none-match")
    if client_etag == etag:
        return FastAPIResponse(
            status_code=304,
            headers={"ETag": etag}
        )

    return FastAPIResponse(
        content=json.dumps(user_data),
        media_type="application/json",
        headers={
            "ETag": etag,
            "Cache-Control": "public, max-age=300"
        }
    )
```

工作原理：

1. 服务器在响应中发送 ETag 哈希值
2. 浏览器缓存响应
3. 下一次请求时，浏览器发送 `If-None-Match: <etag>`
4. 服务器比较 ETags。如果相同，则返回 304
5. 浏览器使用缓存副本

### 使用 HTTP 缓存的场景

* 公开数据（产品目录、博客文章）
* 不常变化的数据
* 静态或半静态内容
* 任何在一段时间内返回相同数据的 GET 端点

## 第二层：内存缓存

适用于单实例应用或计算成本高昂但可以在重启时丢失的数据。

### 使用 functools.lru\_cache

```
from functools import lru_cache
from fastapi import FastAPI
import time

app = FastAPI()

@lru_cache(maxsize=128)
def get_user_permissions(user_id: int) -> list[str]:
    """自动缓存的权限计算"""
    time.sleep(0.5)  # 模拟耗时操作

    return ["read", "write", "delete"]

@app.get("/users/{user_id}/permissions")
asyncdef get_permissions(user_id: int):
    """使用缓存函数的端点"""
    permissions = get_user_permissions(user_id)
    return {"user_id": user_id, "permissions": permissions}

@app.post("/users/{user_id}/permissions/clear")
asyncdef clear_permissions_cache(user_id: int):
    """清除特定用户的缓存"""
    get_user_permissions.cache_clear()
    return {"message": "Cache cleared"}
```

**注意**：`lru_cache` 只适用于普通函数，而不是 `async def`。保持缓存函数同步。

### 使用 cachetools 实现 TTL

```
from fastapi import FastAPI
from cachetools import TTLCache
import time

app = FastAPI()

cache = TTLCache(maxsize=1000, ttl=300)

def expensive_computation(user_id: int) -> dict:
    """模拟耗时操作"""
    time.sleep(1)
    return {"user_id": user_id, "result": "expensive data"}

@app.get("/data/{user_id}")
asyncdef get_data(user_id: int):
    """带有内存 TTL 缓存的端点"""
    cache_key = f"user_data_{user_id}"

    if cache_key in cache:
        return {"cached": True, "data": cache[cache_key]}

    result = expensive_computation(user_id)
    cache[cache_key] = result

    return {"cached": False, "data": result}
```

**性能**：缓存命中约 0.5 毫秒，完整计算约 1000 毫秒。

### 使用内存缓存的场景

* 单实例部署
* 配置数据、权限、查找表
* 耗时计算的结果
* 可以在重启时丢失的数据
* 总缓存大小小于 100MB

### 局限性

* 不跨应用实例共享
* 重启后丢失
* 使用应用内存

## 第三层：Redis 缓存

当你需要跨实例共享缓存，或者需要在重启后保留缓存时，使用 Redis。

```
from fastapi import FastAPI
from contextlib import asynccontextmanager
import redis.asyncio as redis
import json

redis_client = None

@asynccontextmanager
asyncdef lifespan(app: FastAPI):
    """管理 Redis 连接生命周期"""
    global redis_client

    redis_client = redis.Redis(
        host="localhost",
        port=6379,
        decode_responses=True
    )

    try:
        await redis_client.ping()
        print("✓ Connected to Redis")
    except redis.ConnectionError:
        print("✗ Redis connection failed")
        redis_client = None

    yield

    if redis_client:
        await redis_client.aclose()

app = FastAPI(lifespan=lifespan)

@app.get("/posts/{post_id}")
asyncdef get_post(post_id: int):
    """存储在 Redis 中的帖子数据"""
    cache_key = f"post:{post_id}"

    if redis_client:
        try:
            cached = await redis_client.get(cache_key)
            if cached:
                return json.loads(cached)
        except redis.RedisError as e:
            print(f"Redis error: {e}")

    post = {
        "id": post_id,
        "title": "Post Title",
        "content": "Post content here..."
    }

    if redis_client:
        try:
            await redis_client.setex(
                cache_key,
                3600,
                json.dumps(post)
            )
        except redis.RedisError as e:
            print(f"Redis cache write failed: {e}")

    return post
```

### 可重用的缓存辅助函数

```
from typing import Callable, Any
import json

asyncdef cached(
    key: str,
    ttl: int,
    fetch_func: Callable[[], Any]
) -> Any:
    """通用缓存辅助函数，支持回退"""

    if redis_client:
        try:
            cached = await redis_client.get(key)
            if cached:
                return json.loads(cached)
        except redis.RedisError:
            pass

    data = await fetch_func() if callable(fetch_func) else fetch_func

    if redis_client:
        try:
            await redis_client.setex(key, ttl, json.dumps(data))
        except redis.RedisError:
            pass

    return data

@app.get("/users/{user_id}")
asyncdef get_user(user_id: int):
    """使用辅助函数的用户端点"""

    asyncdef fetch_user():
        return {"id": user_id, "name": "Alice"}

    user = await cached(
        key=f"user:{user_id}",
        ttl=1800,  # 30 分钟
        fetch_func=fetch_user
    )

    return user

### 缓存失效

缓存中最难的部分是保持数据的新鲜度。

```python
@app.put("/users/{user_id}")
asyncdef update_user(user_id: int, name: str):
    """更新用户并失效缓存"""

    updated_user = {"id": user_id, "name": name}

    if redis_client:
        try:
            await redis_client.delete(f"user:{user_id}")
        except redis.RedisError as e:
            print(f"Cache invalidation failed: {e}")

    return updated_user

@app.delete("/cache/pattern/{pattern}")
asyncdef clear_cache_pattern(pattern: str):
    """清除匹配模式的所有键
    示例：/cache/pattern/user:* 清除所有用户缓存
    """
    ifnot redis_client:
        return {"error": "Redis not available"}

    try:
        keys = []
        asyncfor key in redis_client.scan_iter(match=pattern):
            keys.append(key)

        if keys:
            await redis_client.delete(*keys)

        return {"cleared": len(keys), "pattern": pattern}
    except redis.RedisError as e:
        return {"error": str(e)}
```

**警告**：`redis_client.flushdb()` 会清空整个 Redis 数据库。仅在开发环境中使用。

### 使用 Redis 的场景

* 多实例部署
* 跨服务共享状态
* 重启后仍保留的缓存
* 会话数据
* 速率限制数据
* 大于每个实例可用内存的缓存

## 综合策略：生产环境的解决方案

将所有三层缓存结合起来使用：

```
from fastapi import FastAPI, Response, Request
from fastapi.responses import Response as FastAPIResponse
from contextlib import asynccontextmanager
import redis.asyncio as redis
import json
import hashlib
from functools import lru_cache

redis_client = None

@asynccontextmanager
asyncdef lifespan(app: FastAPI):
    global redis_client
    redis_client = redis.Redis(host="localhost", decode_responses=True)

    try:
        await redis_client.ping()
    except redis.ConnectionError:
        redis_client = None

    yield

    if redis_client:
        await redis_client.aclose()

app = FastAPI(lifespan=lifespan)

@lru_cache(maxsize=1)
def get_app_config() -> dict:
    """内存中缓存的应用配置"""
    return {
        "feature_flags": {"new_ui": True},
        "api_version": "1.0"
    }

asyncdef fetch_product_from_db(product_id: int) -> dict:
    """模拟从数据库获取数据"""
    return {
        "id": product_id,
        "name": f"Product {product_id}",
        "price": 99.99
    }

@app.get("/products/{product_id}")
asyncdef get_product(product_id: int, request: Request):
    """带有三层缓存的产品端点"""

    redis_key = f"product:{product_id}"
    data = None

    if redis_client:
        try:
            cached = await redis_client.get(redis_key)
            if cached:
                data = json.loads(cached)
        except redis.RedisError:
            pass

    ifnot data:
        data = await fetch_product_from_db(product_id)

    if redis_client:
        try:
            await redis_client.setex(redis_key, 3600, json.dumps(data))
        except redis.RedisError:
            pass

    data_str = json.dumps(data, sort_keys=True)
    etag = f'"{hashlib.md5(data_str.encode()).hexdigest()}"'

    if request.headers.get("if-none-match") == etag:
        return FastAPIResponse(
            status_code=304,
            headers={"ETag": etag}
        )

    return FastAPIResponse(
        content=data_str,
        media_type="application/json",
        headers={
            "ETag": etag,
            "Cache-Control": "public, max-age=1800"
        }
    )

@app.get("/config")
asyncdef get_config(response: Response):
    """仅在内存中缓存的配置"""
    config = get_app_config()
    response.headers["Cache-Control"] = "public, max-age=86400"
    return config
```

### 不要缓存的内容

* 实时数据（股票价格、传感器数据）
* 用户特定的敏感数据（除非有谨慎的过期策略）
* 可以通过其他方式优化的大响应

## 常见错误

### 错误 1：Redis 键没有设置 TTL

```
await redis_client.set("user_profile", user_data)

await redis_client.setex("user_profile", 3600, user_data)
```

### 错误 2：忘记失效缓存

```
@app.put("/users/{user_id}")
async def update_user(user_id: int, name: str):
    await db.update_user(user_id, name)
    return {"updated": True}

@app.put("/users/{user_id}")
async def update_user(user_id: int, name: str):
    await db.update_user(user_id, name)
    await redis_client.delete(f"user:{user_id}")
    return {"updated": True}
```

### 错误 3：在多实例设置中使用内存缓存

```
cache = {}

await redis_client.get(key)
```

### 错误 4：没有为 Redis 添加错误处理

```
cached = await redis_client.get(key)

try:
    cached = await redis_client.get(key)
except redis.RedisError:
    cached = None
```

### 错误 5：缓存踏脚（Cache Stampede）

当缓存过期时，多个请求同时击中数据库。

```
import asyncio

asyncdef get_with_lock(key: str, ttl: int, fetch_func):
    """使用锁防止缓存踏脚"""

    cached = await redis_client.get(key)
    if cached:
        return json.loads(cached)

    lock_key = f"{key}:lock"
    lock_acquired = await redis_client.set(
        lock_key,
        "1",
        ex=10,  # 锁的过期时间
        nx=True# 仅在键不存在时设置
    )

    if lock_acquired:
        try:
            data = await fetch_func()
            await redis_client.setex(key, ttl, json.dumps(data))
            return data
        finally:
            await redis_client.delete(lock_key)
    else:
        await asyncio.sleep(0.1)
        returnawait get_with_lock(key, ttl, fetch_func)
```

缓存功能强大，但会增加复杂性。从最简单的缓存层开始，解决你的问题，然后根据需要添加更多层。

如果你觉得这份指南很有用，可以关注我，获取更多关于 Python 和 FastAPI 开发的实用见解。