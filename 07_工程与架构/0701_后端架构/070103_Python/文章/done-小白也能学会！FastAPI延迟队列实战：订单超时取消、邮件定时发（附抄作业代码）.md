> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: 小白也能学会！FastAPI延迟队列实战：订单超时取消、邮件定时发（附抄作业代码）
author: python小甲鱼
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5ODkxMTA0NQ==&mid=2247484412&idx=1&sn=0dd28b5ae87312e272b31e16744aaad0&chksm=c1ebab8fadac24138e6459a04fa5b17a357170b251371839315704604d5766281fbbf1abe78b&mpshare=1&scene=24&srcid=1030HLxe022HCZ7DyehQQhvO&sharer_shareinfo=b5c8d8eb96e67bd71643e326d6cc7a08&sharer_shareinfo_first=b5c8d8eb96e67bd71643e326d6cc7a08#rd
---

#

你有没有遇到过这些场景？

* • 电商订单：用户下单后30分钟没付款，得自动取消；
* • 注册欢迎：用户注册后，想3秒后自动发欢迎邮件；
* • 定时提醒：会议开始前15分钟，给参会人推通知。

这些“到点才干活”的需求，其实用「延迟队列」就能轻松实现！今天就用FastAPI（超火的Python后端框架）+ Redis，手把手教你搭一个延迟队列，代码复制就能跑，新手也能上手～

## 先搞懂：什么是延迟队列？（比喻版）

延迟队列就像“外卖催单提醒”：

* • 你下单后（生产者发消息），外卖平台不会立刻催单；
* • 而是等30分钟（延迟时间），如果骑手还没取餐，才会自动发提醒（消费者执行任务）。

简单说：**把任务“存起来”，到指定时间再执行**，避免让用户等着（比如下单后不用等“取消逻辑”完成，直接返回成功）。

## 为什么选FastAPI+Redis？（新手友好版）

新手不用纠结复杂工具，先从“轻量级组合”入手：

* • **FastAPI**：Python后端框架，写接口快、自带文档，新手好上手；
* • **Redis**：内存数据库，用来存延迟任务，速度快，配置简单；
* • **ARQ**：专门帮FastAPI做异步任务的工具，不用自己写复杂调度逻辑。

这套组合不用装太多东西，命令行敲几下就能跑起来～

## 第一步：环境准备（3分钟搞定）

先把需要的工具装好，打开命令行，复制粘贴下面的命令：

```
# 安装依赖（版本都帮你配好了，避免冲突）
pip install fastapi==0.109.0 arq==0.26.0 pydantic==2.5.3 redis==5.0.0 uvicorn==0.27.1
```

* • `fastapi`：核心框架；
* • `arq`：处理异步任务；
* • `pydantic`：帮你校验数据格式（比如订单ID必须是字符串）；
* • `redis`：连接Redis数据库；
* • `uvicorn`：启动FastAPI服务的工具。

另外，你需要装一个Redis（本地用的话很简单）：

* • Windows：搜“Redis Windows版”，下载后双击`redis-server.exe`启动；
* • Mac：用`brew install redis`，然后`redis-server`启动；
* • 启动后，命令行输`redis-cli ping`，如果返回`PONG`，说明Redis没问题了！

## 第二步：项目结构（照着建文件夹）

先建一个项目文件夹（比如叫`fastapi_delay_queue`），里面放4个文件，结构如下：

```
fastapi_delay_queue/
├── main.py       # FastAPI入口（写接口的地方）
├── worker.py     # ARQ小助手（执行延迟任务的地方）
├── schemas.py    # 数据规矩（比如订单要包含哪些信息）
└── tasks.py      # 任务定义（比如“取消订单”“发邮件”怎么干）
```

不用记，照着这个结构建4个空文件就行～

## 第三步：逐文件写代码（抄作业版）

每个文件的代码都给你写好了，复制进去，关键地方有注释，新手也能看懂！

### 1. schemas.py：给数据定规矩（避免传错格式）

比如“提交订单”时，必须传`order_id`（订单号），可选传`expire_minutes`（延迟多久取消，默认30分钟）。这里用Pydantic帮我们校验数据，比如传个数字当订单号，会自动报错。

```
# schemas.py
from pydantic import BaseModel

# 订单延迟任务的数据规矩
class OrderPayload(BaseModel):
    order_id: str  # 必须传：订单号（比如"order_123456"）
    expire_minutes: int = 30  # 可选：延迟时间（默认30分钟）

# 邮件延迟任务的数据规矩
class EmailPayload(BaseModel):
    email: str  # 必须传：收件人邮箱（比如"user@example.com"）
    template: str = "welcome"  # 可选：邮件模板（默认"欢迎模板"）
```

### 2. tasks.py：定义“要干的活”（比如取消订单、发邮件）

这里写两个实际任务：`process_order`（处理订单超时）和`send_email`（延迟发邮件），还有ARQ的配置（比如启动时连Redis）。

```
# tasks.py
from arq import cron  # ARQ的定时任务工具
from .schemas import OrderPayload, EmailPayload  # 导入上面定的 data规矩

# 任务1：处理订单超时（比如30分钟后取消订单）
async def process_order(ctx, payload: dict):
    """
    ctx：上下文（可以拿到Redis连接、日志等）
    payload：任务数据（比如订单号、延迟时间）
    """
    # 这里可以加实际业务逻辑，比如查数据库、改订单状态
    print(f"✅ 订单 {payload['order_id']} 超时未支付，已自动取消！（延迟了{payload['expire_minutes']}分钟）")

# 任务2：延迟发送邮件（比如注册后3秒发欢迎邮件）
async def send_email(ctx, payload: dict):
    """延迟发送邮件"""
    # 这里可以加实际发邮件的代码（比如用smtplib）
    print(f"✅ 已给 {payload['email']} 发送 {payload['template']} 邮件！")

# ARQ启动时要做的事：比如连接Redis
async def startup(ctx):
    # 把Redis连接放进上下文，后面任务能用
    ctx["redis"] = await ctx["pool"]

# ARQ的配置类（告诉ARQ怎么干活）
class WorkerSettings:
    # 定时任务（可选，比如每天8点、12点、18点执行某个任务）
    cron_jobs = [
        cron(
            process_order,  # 要执行的任务
            hour={8, 12, 18},  # 每天8点、12点、18点执行
            run_at_startup=True  # 启动Worker时先执行一次（测试用）
        )
    ]
    on_startup = startup  # 启动时执行上面的startup函数
    # Redis连接配置（默认本地Redis，端口6379，不用改）
    redis_settings = {"host": "localhost", "port": 6379}
```

### 3. main.py：FastAPI入口（写接口，让前端调用）

这里写两个接口：

* • `/submit-order`：提交订单到延迟队列（30分钟后取消）；
* • `/welcome-email`：提交邮件任务（3秒后发邮件）。

```
# main.py
from fastapi import FastAPI
from arq.connections import create_pool  # ARQ连接Redis的工具
from .schemas import OrderPayload, EmailPayload  # 数据规矩
from .tasks import WorkerSettings  # ARQ配置

# 初始化FastAPI app
app = FastAPI(title="小白的延迟队列Demo", version="1.0")

# FastAPI启动时：连接Redis（只连一次，避免每次接口调用都连）
@app.on_event("startup")
async def init_redis():
    # 创建Redis连接池，存在app.state里（全局可用）
    app.state.redis = await create_pool(WorkerSettings.redis_settings)

# 接口1：提交订单到延迟队列（比如用户下单后调用）
@app.post("/submit-order", summary="提交订单（延迟取消）")
async def submit_order(order: OrderPayload):
    """
    接收订单信息，30分钟后自动取消（可改expire_minutes调整时间）
    - order_id：订单号（比如"order_123"）
    - expire_minutes：延迟时间（默认30分钟）
    """
    # 把订单任务放进延迟队列
    await app.state.redis.enqueue_job(
        "process_order",  # 要执行的任务名（必须和tasks.py里的函数名一致）
        order.dict(),     # 任务数据（转成字典，方便传输）
        _defer_by=order.expire_minutes * 60  # 延迟时间（转成秒，因为ARQ认秒）
    )
    return {"msg": "订单已提交！30分钟后未支付会自动取消", "order_id": order.order_id}

# 接口2：提交邮件任务（比如用户注册后调用）
@app.post("/welcome-email", summary="提交邮件（延迟发送）")
async def schedule_email(email: EmailPayload):
    """
    接收邮箱信息，3秒后自动发欢迎邮件
    - email：收件人邮箱（比如"user@example.com"）
    - template：邮件模板（默认"welcome"）
    """
    await app.state.redis.enqueue_job(
        "send_email",  # 任务名（和tasks.py里的函数名一致）
        email.dict(),  # 邮件数据
        _defer_by=3  # 延迟3秒（不用转，直接写秒数）
    )
    return {"msg": "邮件任务已安排！3秒后发送", "email": email.email}
```

### 4. worker.py：启动ARQ小助手（执行延迟任务）

这个文件超简单，就一行代码，告诉ARQ“去tasks.py里找配置和任务”：

```
# worker.py
# 一行代码：启动ARQ Worker，用tasks.py里的WorkerSettings配置
arq tasks.WorkerSettings
```

## 第四步：启动服务，测试效果！（关键步骤）

分两步启动：先启动“干活的小助手（Worker）”，再启动“FastAPI接口服务”，顺序别错了～

### 步骤1：启动ARQ Worker（执行任务的小助手）

打开一个命令行，进入项目文件夹（`fastapi_delay_queue`），输入：

```
# 启动Worker（用Python执行worker.py）
python -m worker
```

如果看到类似下面的信息，说明Worker启动成功，正在等任务：

```
2025-08-23 10:00:00.123 INFO arq.worker: Starting worker for 2 functions: process_order, send_email
2025-08-23 10:00:00.124 INFO arq.worker: Connected to redis://localhost:6379/0
```

### 步骤2：启动FastAPI服务（提供接口）

再打开一个新的命令行（别关刚才的Worker窗口），进入项目文件夹，输入：

```
# 启动FastAPI服务，用uvicorn
uvicorn main:app --reload
```

启动成功后，会看到类似这样的信息：

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [12345] using WatchFiles
```

### 步骤3：测试接口（两种方式，新手推荐第一种）

#### 方式1：用FastAPI自带的文档（不用写代码）

打开浏览器，输入`http://127.0.0.1:8000/docs`，会看到自动生成的接口文档：

1. 1. 找到`/submit-order`接口，点“Try it out”；
2. 2. 输入订单号，比如`{"order_id": "order_123456", "expire_minutes": 1}`（延迟1分钟，方便测试）；
3. 3. 点“Execute”，如果返回`{"msg": "订单已提交..."}`，说明任务已经进队列了；
4. 4. 等1分钟，看刚才启动Worker的命令行，会打印`✅ 订单 order_123456 超时未支付...`。

同理测试`/welcome-email`接口：

* • 输入`{"email": "test@example.com"}`，点“Execute”；
* • 等3秒，Worker窗口会打印`✅ 已给 test@example.com 发送 welcome 邮件！`。

#### 方式2：用Postman/ curl（可选）

如果会用Postman，也可以发POST请求：

* • 地址：`http://127.0.0.1:8000/submit-order`；
* • 请求体：`{"order_id": "order_654321", "expire_minutes": 1}`；
* • 发送后，等1分钟看Worker输出。

## 新手必看：常见坑和解决方案

刚上手容易踩坑，这里帮你提前避坑～

### 坑1：Worker启动报错“Redis connection error”

**原因**：Redis没启动，或者连接地址错了。
**解决**：

1. 1. 先启动Redis（双击`redis-server.exe`或`redis-server`命令）；
2. 2. 检查`tasks.py`里的`redis_settings`是不是`{"host": "localhost", "port": 6379}`（默认配置，没改就对）；
3. 3. 命令行输`redis-cli ping`，返回`PONG`再启动Worker。

### 坑2：接口调用返回422错误（Validation Error）

**错误示例**：

```
{
  "detail": [
    {"loc": ["body", "expire_minutes"], "msg": "value is not a valid integer", "type": "type_error.integer"}
  ]
}
```

**原因**：数据格式错了，比如`expire_minutes`传了字符串（比如`"30"`），而我们定义的是整数（`30`）。
**解决**：

* • 用`/docs`接口测试时，严格按格式传（比如`{"order_id": "order_123", "expire_minutes": 30}`，数字不用加引号）；
* • 前端调用时，要发JSON格式，别发文本格式（比如用`fetch`时加`body: JSON.stringify(data)`）。

### 坑3：Worker没执行任务，没打印信息

**原因**：

1. 1. 没启动Worker，或者Worker启动后又关了；
2. 2. 任务名写错了（比如`enqueue_job("processOrder")`，而实际函数名是`process_order`，大小写或下划线错了）。
   **解决**：
3. 3. 确保Worker窗口没关，且显示“Connected to redis”；
4. 4. 检查`main.py`里的`enqueue_job`第一个参数，和`tasks.py`里的函数名完全一致（大小写、下划线都要对）。

## 总结：学完能做什么？

今天的代码虽然简单，但能覆盖很多实际需求：

* • 电商：订单超时取消、优惠券到期提醒；
* • 社交：用户注册后延迟发引导邮件、点赞后延迟推送通知；
* • 办公：会议前定时提醒、每日凌晨生成统计报表。

你可以试着改改代码：比如把订单延迟时间改成5秒（方便测试），或者在`send_email`函数里加实际发邮件的代码（用`smtplib`库），成就感会很强～

如果在操作中遇到问题，评论区留言，我会帮你解答！觉得有用的话，点赞收藏，下次需要延迟队列直接抄作业～