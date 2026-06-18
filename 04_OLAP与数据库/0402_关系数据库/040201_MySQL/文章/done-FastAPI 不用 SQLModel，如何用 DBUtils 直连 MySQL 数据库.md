> 已吸收至：[[04_OLAP与数据库/0402_关系数据库/040201_MySQL/040201_核心知识点/MySQL连接池与EventScheduler边界|MySQL连接池与EventScheduler边界]]
---
title: FastAPI 不用 SQLModel，如何用 DBUtils 直连 MySQL 数据库
author: 测开工程师的烦恼
date:
url: https://mp.weixin.qq.com/s?__biz=MzA5NzM0NjU1Nw==&mid=2647871508&idx=1&sn=5ce31f8bb136e0b1afccd0c5e061ba39&chksm=89d30e77e37fbd8f41c69992b48a06cd58b2d4bd340366b36b3700095aace5d6557c14cfe610&mpshare=1&scene=24&srcid=12190WFTJAcWU55gAWdYZzaw&sharer_shareinfo=74b31adf8df64f30a0a2c4be15dd5800&sharer_shareinfo_first=74b31adf8df64f30a0a2c4be15dd5800#rd
---

---

FastAPI 不用 SQLModel，如何用 DBUtils 直连 MySQL 数据库

* FastAPI
* 数据库
* 连接池
* MySQL
* DBUtils
* 后端开发

---

## 导读

很多同学在用 FastAPI 做项目时，第一反应就是：上 ORM，用 `SQLModel`、`SQLAlchemy`这一套。

但在不少业务场景下，我们其实并不需要完整的 ORM 能力，而是更希望：

* **沿用 DBA 已经写好的 SQL**，后端只负责调用；
* **自己手写 SQL**，精细控制索引、锁、执行计划；
* **项目体量不大**，上 ORM 反而增加心智负担和调试成本。

这时候一个很自然的问题就来了：

> 在 FastAPI 里，如果我不用 SQLModel，只想**直连 MySQL 数据库并执行 SQL**，还能不能优雅一点？

答案是：完全可以，而且还可以做得很专业。

这篇文章我们就用一个实战方案，带你搞懂：

* **如何在 FastAPI 中用 DBUtils 管理 MySQL 连接池**；
* 不用 SQLModel，只靠 **`pymysql`+ 手写 SQL**就能跑通生产项目；
* 如何写出既**性能稳定**又**易于维护**的数据库访问代码。

---

## 一、为什么一定要上连接池？

先统一一个共识：**不用 SQLModel ≠ 裸连数据库**。

如果只是简单地在接口里：

```
import pymysql

conn = pymysql.connect(...)
# 执行 SQL
conn.close()
```

在本地测试、Demo 场景当然没问题。但一旦到了线上高并发环境，很容易踩坑：

* **每个请求都新建连接**：频繁 TCP 握手、认证，开销大、延迟高；
* **容易忘记关闭连接**：代码分支一复杂，没处理好的地方就会连接泄漏；
* **连接数量不可控**：并发上来后，MySQL 连接数很容易被打满。

为了同时兼顾“直连 SQL 的灵活性”和“生产级的稳定性”，最合适的做法就是：

> **连接池 + 手写 SQL**

这里我们选择：

* **驱动**：`pymysql`（同步 MySQL 驱动，生态成熟）；
* **连接池**：`DBUtils`（专门做数据库连接池的库，简单好用）。

FastAPI 支持同步视图函数（`def`）和异步视图函数（`async def`），我们完全可以：

* 用 **同步代码 + 连接池**处理数据库；
* 用 FastAPI 的依赖注入管理连接的“借用/归还”。

---

## 二、方案整体设计

我们希望做到几件事：

1. **应用启动时**，创建一个 MySQL 连接池；
2. **每个请求到来时**，从池里借出一个连接；
3. 业务代码只需要：

* 获取连接 → 执行 SQL → 提交/回滚 → 归还连接；

4. **应用关闭时**，优雅关闭连接池，释放资源。

用更形象的话来讲：

> 把「连接池」当做停车场：
>
> * 启动时建好停车场（创建一定数量的连接）；
> * 每辆车（每个请求）来时，从停车场里“借一个车位”（拿一个连接）；
> * 用完了就把车位还回去（连接归还给池子）；
> * 停车场本身负责车位的总数控制和资源清理。

下面我们一步步实现这个方案。

---

## 三、安装依赖

我们需要的核心依赖有：

* `fastapi`：Web 框架；
* `uvicorn`：ASGI 服务器；
* `pymysql`：MySQL 驱动；
* `DBUtils`：连接池管理库。

```
pip install fastapi uvicorn pymysql DBUtils
```

### 踩坑日记：ModuleNotFoundError: No module named 'DBUtils'

这里顺便记录一下一个非常常见、但又挺“迷惑”的报错：

```
ModuleNotFoundError: No module named 'DBUtils'
```

明明你已经写了：

```
pip install DBUtils
```

代码里也老老实实：

```
from DBUtils.PooledDB import PooledDB
```

却还是报模块找不到，常见原因有几类：

* **安装到的环境不对**：

+ 例如你在系统 Python 或某个虚拟环境里装的包，
+ 但运行 FastAPI 服务用的是另外一个 Python 解释器；
+ 用 `which python`/ `which uvicorn`或 `pip show DBUtils`检查一下是否同一个环境。

* **pip 和 python 版本不匹配**：

+ 例如 `python3`在用，结果你执行的是 `pip`（对应 Python 2 或别的环境）；
+ 可以强制指定：`python3 -m pip install DBUtils`或 `python -m pip install DBUtils`，确保装到当前 Python 里。

* **容器 / 线上环境漏装依赖**：

+ 本地可以 import，是因为你本地装过；
+ Docker 镜像或服务器没跑 `pip install DBUtils`，需要在 `requirements.txt`或构建脚本里补上。

* **不同版本/发行版的包名和导入路径不一致**：

+ 有些环境你会看到写法是：`from dbutils.pooled_db import PooledDB`（全小写 `dbutils`）；
+ 有些环境则是：`from DBUtils.PooledDB import PooledDB`（首字母大写的 `DBUtils`）；
+ 核心原则是：**以 `pip show`和官方文档为准**，确认你当前安装的包名和推荐的导入路径。

我的经验是：

1. 先在启动 FastAPI 的同一个环境里执行：

   ```
   python -c "import DBUtils; print(DBUtils.__file__)"
   ```

   或者：

   ```
   python -c "from dbutils.pooled_db import PooledDB; print(PooledDB)"
   ```

   能跑通就说明模块和导入路径是对的；
2. 如果这里都报 `ModuleNotFoundError`，那就 99% 是装错环境或没装，重新用 `python -m pip install DBUtils`或对应的小写包名装一次通常就好了。

---

## 四、封装 DBUtils MySQL 连接池

先写一个单独的模块，比如 `db.py`，专门负责：

* 创建/关闭连接池；
* 从连接池获取连接；
* 和 FastAPI 生命周期集成。

```
# db.py
from contextlib import contextmanager
from typing import Generator

import pymysql
from DBUtils.PooledDB import PooledDB
from fastapi import FastAPI


class MySQLPool:
    """基于 DBUtils 的 MySQL 连接池封装。"""

    def __init__(
        self,
        host: str,
        port: int,
        user: str,
        password: str,
        database: str,
        min_cached: int = 1,
        max_cached: int = 5,
        max_connections: int = 10,
        charset: str = "utf8mb4",
    ) -> None:
        self._pool: PooledDB | None = None
        self._pool_config = {
            "creator": pymysql,  # 使用 pymysql 作为驱动
            "host": host,
            "port": port,
            "user": user,
            "password": password,
            "database": database,
            "charset": charset,
            "mincached": min_cached,  # 启动时创建的空闲连接数量
            "maxcached": max_cached,  # 连接池中最多缓存的连接数量
            "maxconnections": max_connections,  # 最大连接数
            "blocking": True,  # 连接数耗尽时是否阻塞等待
            "ping": 1,  # 检查连接是否可用
        }

    def init_pool(self) -> None:
        """在应用启动时创建连接池。"""
        if self._pool isNone:
            self._pool = PooledDB(**self._pool_config)

    def close_pool(self) -> None:
        """在应用关闭时清理连接池（如果需要）。"""
        # 对于 DBUtils，连接池自己会管理底层连接的释放，这里一般不用显式关闭。
        self._pool = None

    @contextmanager
    def connection(self) -> Generator[pymysql.connections.Connection, None, None]:
        """从连接池获取一个连接，使用 with 语法自动归还。"""
        if self._pool isNone:
            raise RuntimeError("Connection pool is not initialized")

        conn = self._pool.connection()
        try:
            yield conn
        finally:
            # DBUtils 的 connection() 返回的是一个封装对象，
            # 调用 close() 实际上是把连接归还给连接池。
            conn.close()


# 可以根据实际环境调整这些配置
mysql_pool = MySQLPool(
    host="127.0.0.1",
    port=3306,
    user="root",
    password="password",
    database="mydb",
)


def init_app(app: FastAPI) -> None:
    """把连接池初始化逻辑挂到 FastAPI 生命周期上。"""

    @app.on_event("startup")
    def _startup() -> None:# noqa: D401
        # 应用启动时创建连接池
        mysql_pool.init_pool()

    @app.on_event("shutdown")
    def _shutdown() -> None:# noqa: D401
        # 应用关闭时清理连接池引用
        mysql_pool.close_pool()
```

这里有几个关键点：

* **`MySQLPool`封装了 DBUtils 的 `PooledDB`**，避免在业务代码里直接操作底层细节；
* 通过 `@contextmanager`提供 `connection()`方法：

+ 可以 `with mysql_pool.connection() as conn:`的写法；
+ `with`代码块结束时，会自动把连接归还给连接池；

* 通过 `init_app(app)`把连接池和 FastAPI 的 `startup`/ `shutdown`生命周期绑定在一起。

---

## 五、在 FastAPI 中使用连接池执行 SQL

接下来我们在 `main.py`中：

* 初始化应用；
* 把连接池挂载到应用上；
* 通过依赖注入获取连接；
* 写几个简单的接口做演示。

```
# main.py
from typing import Generator, List

from fastapi import Depends, FastAPI

from db import init_app, mysql_pool


app = FastAPI(title="FastAPI MySQL Pool Demo")
init_app(app)


def get_db_conn() -> Generator:
    """FastAPI 依赖：从连接池获取一个连接。"""
    with mysql_pool.connection() as conn:
        yield conn


@app.get("/users")
def list_users(conn=Depends(get_db_conn)) -> List[dict]:
    """查询用户列表，演示 SELECT。"""
    with conn.cursor() as cursor:
        sql = "SELECT id, username, email FROM users ORDER BY id DESC LIMIT 20"
        cursor.execute(sql)
        rows = cursor.fetchall()

    # pymysql 默认返回 tuple，可以配置 cursorclass=DictCursor 直接返回 dict
    # 这里演示手动转换
    column_names = [desc[0] for desc in cursor.description]
    return [dict(zip(column_names, row)) for row in rows]


@app.post("/users")
def create_user(username: str, email: str, conn=Depends(get_db_conn)) -> dict:
    """新建用户，演示 INSERT。"""
    with conn.cursor() as cursor:
        sql = """
        INSERT INTO users (username, email)
        VALUES (%s, %s)
        """
        cursor.execute(sql, (username, email))
        conn.commit()

        user_id = cursor.lastrowid

    return {"id": user_id, "username": username, "email": email}
```

几点说明：

* 这里我们直接用 **同步视图函数**（`def`），用起来和传统 Flask、Django 中的写法类似；
* 通过 `Depends(get_db_conn)`，把「从连接池获取连接」这件事交给 FastAPI 管理；
* 所有 SQL 都是手写：

+ **完全不依赖 SQLModel / ORM**；
+ 需要什么 SQL 就写什么，DBA 给的脚本可以直接复用；

* 使用参数化 SQL（`%s`+ 参数元组）可以避免 SQL 注入风险。

如果你希望 `cursor.fetchall()`直接返回 dict，可以在创建连接池时配置：

```
from pymysql.cursors import DictCursor

# 在 MySQLPool.__init__ 中的 _pool_config 加一项：
"cursorclass": DictCursor,
```

这样在接口中就可以直接：

```
rows = cursor.fetchall()
return list(rows)
```

---

## 六、不用连接池时的常见写法对比

很多入门教程里会给出类似这样的示例：

```
import pymysql
from fastapi import FastAPI

app = FastAPI()


@app.get("/users")
def list_users():
    conn = pymysql.connect(
        host="127.0.0.1",
        port=3306,
        user="root",
        password="password",
        database="mydb",
        charset="utf8mb4",
    )
    try:
        with conn.cursor() as cursor:
            cursor.execute("SELECT id, username, email FROM users")
            rows = cursor.fetchall()
        conn.commit()
        return rows
    finally:
        conn.close()
```

这种写法：

* **优点**：入门门槛低，逻辑非常直观，适合 Demo 或脚本；
* **缺点**：

+ 每个请求都要 `connect`/ `close`，并发一上来，性能会明显下滑；
+ 一旦某个分支里忘记 `close`，连接泄漏就会变得非常难排查；
+ 连接数量完全依赖并发量，**不可控**。

而使用 DBUtils 连接池的写法：

* 连接在 **应用启动时一次性建立**一部分；
* 并发请求只是 **从池子里借/还连接**，不会无限制创建新连接；
* 通过 `maxconnections`等参数，可以和 DBA 商量一个合理的连接数上限；
* 代码中用 `with`管理连接和游标，资源释放变得更安全、可控。

---

## 七、实战中的一些优化建议

最后再给几个在真实项目里经常会用到的小建议：

* **和 DBA 对齐连接数配置**：

+ 例如 MySQL 最大连接数是 500，那你的应用不要单实例就跑出 400 个连接；
+ 通常会预留一部分给排查/备份/其它服务使用。

* **合理设置连接池参数**：

+ `mincached`：可以设置为 1～2，保证冷启动时就有可用连接；
+ `maxcached`/ `maxconnections`：结合 QPS 和 SQL 执行时间来调优；
+ `blocking=True`：当连接用完时阻塞等待，而不是直接抛错，避免突发流量时崩掉。

* **统一封装数据库访问层**：

+ 不要在每个接口里都手写 `cursor.execute`+ SQL 拼接；
+ 可以抽象出一层 `repository`或 `dao`，接口层只关心业务对象即可。

* **监控和日志**：

+ 记录每条重要 SQL 的耗时、影响行数；
+ 对慢 SQL 做专门日志，方便后期优化。

---

## 总结

* **不用 SQLModel，也完全可以在 FastAPI 中优雅地直连 MySQL**，关键是：

+ 使用 `DBUtils`管理连接池；
+ 用 FastAPI 的依赖注入把「借连接/还连接」这件事标准化。

* 方案的核心是：

+ **`pymysql`+ DBUtils 连接池 + 手写 SQL`**，兼顾灵活性和性能；
+ 应用启动时初始化连接池，请求阶段按需借用连接，应用关闭时释放资源。

* 真实项目里，建议再搭配一层轻量封装（DAO/Repository），让业务代码只和“对象/字典”打交道，把 SQL 细节收敛在数据访问层中。

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝你在测开与后端之路上越走越稳，越走越远。