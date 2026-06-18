> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: Python异步编程完全指南
author: 赛博零号
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0MjUxOTkyOA==&mid=2247484271&idx=1&sn=09ff178658c5e83b55ab143636806d54&chksm=c3ebeed9428bf071ab4861373a80697ddbf7a4fbfbe678711d74820f0af86b2adea22b04825e&mpshare=1&scene=24&srcid=1030bpOkG9fQrUz5wbaqOfks&sharer_shareinfo=e23ce3b5607a2500ca178c75753d8f52&sharer_shareinfo_first=e23ce3b5607a2500ca178c75753d8f52#rd
---

# 前言

异步编程在现代Python开发中已经成为处理高并发场景的标配方案。从Web服务到爬虫，从数据库操作到网络通信，异步编程能够在不增加线程开销的情况下，显著提升程序的并发处理能力。

Python 3.5引入了`async/await`语法后，异步编程变得更加直观。但要真正用好异步，需要理解协程的运行机制、事件循环的工作原理，以及各种异步工具的使用场景。本文将系统性地介绍Python异步编程的核心知识点，从基础用法到进阶技巧，帮助你构建高性能的异步应用。

## 环境准备

Python 3.7+已经内置了完整的asyncio支持，无需额外安装。但为了演示一些实用场景，我们需要安装几个常用的异步库：

```
pip install aiohttp aiofiles aiomysql redis
```

## 异步基础使用

### 协程的定义与执行

异步函数使用`async def`定义，调用异步函数会返回一个协程对象，需要通过事件循环来执行：

```
import asyncio
import time

async def fetch_data(id):
    print(f"开始获取数据 {id}")
    await asyncio.sleep(2)  # 模拟IO操作
    print(f"数据 {id} 获取完成")
    return f"data_{id}"

async def main():
    start = time.time()
    
    # 串行执行
    result1 = await fetch_data(1)
    result2 = await fetch_data(2)
    
    print(f"串行耗时: {time.time() - start:.2f}秒")
    
    # 并发执行
    start = time.time()
    results = await asyncio.gather(
        fetch_data(3),
        fetch_data(4),
        fetch_data(5)
    )
    print(f"并发耗时: {time.time() - start:.2f}秒")
    print(f"结果: {results}")

# Python 3.7+
asyncio.run(main())
```

**运行结果：**

```
开始获取数据 1
数据 1 获取完成
开始获取数据 2
数据 2 获取完成
串行耗时: 4.00秒
开始获取数据 3
开始获取数据 4
开始获取数据 5
数据 3 获取完成
数据 4 获取完成
数据 5 获取完成
并发耗时: 2.00秒
结果: ['data_3', 'data_4', 'data_5']
```

### 创建任务

使用`create_task`可以立即调度协程执行，而不用等待await：

```
async def process_data(name, delay):
    print(f"{name} 开始处理")
    await asyncio.sleep(delay)
    print(f"{name} 处理完成")
    return f"{name}_result"

async def main():
    # 创建任务会立即开始执行
    task1 = asyncio.create_task(process_data("任务1", 2))
    task2 = asyncio.create_task(process_data("任务2", 1))
    
    print("任务已创建，做点其他事情...")
    await asyncio.sleep(0.5)
    
    # 等待任务完成
    result1 = await task1
    result2 = await task2
    print(f"结果: {result1}, {result2}")

asyncio.run(main())
```

**运行结果：**

```
任务已创建，做点其他事情...
任务1 开始处理
任务2 开始处理
任务2 处理完成
任务1 处理完成
结果: 任务1_result, 任务2_result
```

### 超时控制

使用`wait_for`设置超时：

```
async def long_running_task():
    await asyncio.sleep(5)
    return "完成"

async def main():
    try:
        result = await asyncio.wait_for(long_running_task(), timeout=2.0)
        print(result)
    except asyncio.TimeoutError:
        print("任务超时")

asyncio.run(main())
```

## 异步上下文管理器

异步上下文管理器使用`async with`语法，适用于需要异步初始化和清理的资源：

```
import asyncio

class AsyncDBConnection:
    def __init__(self, db_name):
        self.db_name = db_name
        self.connection = None
    
    async def __aenter__(self):
        print(f"正在连接数据库 {self.db_name}...")
        await asyncio.sleep(1)  # 模拟连接延迟
        self.connection = f"Connection to {self.db_name}"
        print("数据库连接成功")
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("正在关闭数据库连接...")
        await asyncio.sleep(0.5)
        self.connection = None
        print("数据库连接已关闭")
        return False  # 不抑制异常
    
    async def query(self, sql):
        print(f"执行查询: {sql}")
        await asyncio.sleep(0.3)
        return [{"id": 1, "name": "test"}]

async def main():
    async with AsyncDBConnection("mydb") as db:
        results = await db.query("SELECT * FROM users")
        print(f"查询结果: {results}")
    
    print("上下文已退出")

asyncio.run(main())
```

**运行结果：**

```
正在连接数据库 mydb...
数据库连接成功
执行查询: SELECT * FROM users
查询结果: [{'id': 1, 'name': 'test'}]
正在关闭数据库连接...
数据库连接已关闭
上下文已退出
```

## 异步装饰器

装饰器在异步编程中同样重要，可以用来实现日志、重试、缓存等功能：

```
import asyncio
import functools
import time

def async_timer(func):
    @functools.wraps(func)
    async def wrapper(*args, **kwargs):
        start = time.time()
        result = await func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__} 耗时: {elapsed:.2f}秒")
        return result
    return wrapper

def async_retry(max_attempts=3, delay=1):
    def decorator(func):
        @functools.wraps(func)
        async def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"尝试 {attempt + 1} 失败: {e}，{delay}秒后重试...")
                    await asyncio.sleep(delay)
        return wrapper
    return decorator

@async_timer
@async_retry(max_attempts=3, delay=0.5)
async def unstable_api_call(success_rate=0.3):
    import random
    await asyncio.sleep(0.5)
    if random.random() > success_rate:
        raise Exception("API调用失败")
    return "成功"

async def main():
    try:
        result = await unstable_api_call(success_rate=0.9)
        print(f"结果: {result}")
    except Exception as e:
        print(f"最终失败: {e}")

asyncio.run(main())
```

## 回调函数

虽然async/await是主流，但回调模式在某些场景下仍然有用：

```
import asyncio

async def task_with_callback(value, callback):
    await asyncio.sleep(1)
    result = value * 2
    # 调用回调
    await callback(result)
    return result

async def on_complete(result):
    print(f"任务完成，结果: {result}")

async def on_error(error):
    print(f"任务失败，错误: {error}")

async def main():
    await task_with_callback(10, on_complete)
    
    # 使用add_done_callback
    task = asyncio.create_task(task_with_callback(20, on_complete))
    
    def done_callback(future):
        try:
            result = future.result()
            print(f"通过done_callback获取结果: {result}")
        except Exception as e:
            print(f"任务异常: {e}")
    
    task.add_done_callback(done_callback)
    await task

asyncio.run(main())
```

**运行结果：**

```
任务完成，结果: 20
任务完成，结果: 40
通过done_callback获取结果: 40
```

## 在异步中执行同步函数

有时需要在异步代码中调用阻塞的同步函数，可以使用`run_in_executor`：

```
import asyncio
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def blocking_io_task(n):
    print(f"同步任务 {n} 开始")
    time.sleep(2)  # 阻塞操作
    print(f"同步任务 {n} 完成")
    return f"result_{n}"

def cpu_bound_task(n):
    """CPU密集型任务"""
    print(f"CPU任务 {n} 开始")
    total = sum(i * i for i in range(10**7))
    print(f"CPU任务 {n} 完成")
    return total

async def main():
    loop = asyncio.get_event_loop()
    
    # 在线程池中执行IO密集型任务
    start = time.time()
    with ThreadPoolExecutor(max_workers=3) as executor:
        tasks = [
            loop.run_in_executor(executor, blocking_io_task, i)
            for i in range(3)
        ]
        results = await asyncio.gather(*tasks)
    print(f"IO任务耗时: {time.time() - start:.2f}秒")
    print(f"结果: {results}")
    
    # 在进程池中执行CPU密集型任务
    start = time.time()
    with ProcessPoolExecutor(max_workers=2) as executor:
        tasks = [
            loop.run_in_executor(executor, cpu_bound_task, i)
            for i in range(2)
        ]
        results = await asyncio.gather(*tasks)
    print(f"CPU任务耗时: {time.time() - start:.2f}秒")

asyncio.run(main())
```

## 异步同步原语

### 异步锁（Lock）

用于保护共享资源，防止竞态条件：

```
import asyncio

class Counter:
    def __init__(self):
        self.value = 0
        self.lock = asyncio.Lock()
    
    async def increment(self, name):
        async with self.lock:
            print(f"{name} 获得锁")
            temp = self.value
            await asyncio.sleep(0.1)  # 模拟处理时间
            self.value = temp + 1
            print(f"{name} 释放锁，当前值: {self.value}")

async def main():
    counter = Counter()
    
    # 没有锁的情况（注释掉锁相关代码观察差异）
    tasks = [counter.increment(f"任务{i}") for i in range(5)]
    await asyncio.gather(*tasks)
    
    print(f"最终值: {counter.value}")

asyncio.run(main())
```

**运行结果：**

```
任务0 获得锁
任务0 释放锁，当前值: 1
任务1 获得锁
任务1 释放锁，当前值: 2
任务2 获得锁
任务2 释放锁，当前值: 3
任务3 获得锁
任务3 释放锁，当前值: 4
任务4 获得锁
任务4 释放锁，当前值: 5
最终值: 5
```

### 异步信号量（Semaphore）

限制并发数量：

```
import asyncio
import time

async def access_resource(sem, task_id):
    async with sem:
        print(f"任务 {task_id} 获得资源")
        await asyncio.sleep(1)
        print(f"任务 {task_id} 释放资源")

async def main():
    # 最多允许3个并发
    semaphore = asyncio.Semaphore(3)
    
    start = time.time()
    tasks = [access_resource(semaphore, i) for i in range(10)]
    await asyncio.gather(*tasks)
    
    print(f"总耗时: {time.time() - start:.2f}秒")

asyncio.run(main())
```

### 异步条件变量（Condition）

用于生产者-消费者模式：

```
import asyncio

class AsyncQueue:
    def __init__(self):
        self.queue = []
        self.condition = asyncio.Condition()
    
    async def produce(self, item):
        async with self.condition:
            self.queue.append(item)
            print(f"生产: {item}，队列长度: {len(self.queue)}")
            self.condition.notify()  # 通知消费者
    
    async def consume(self):
        async with self.condition:
            while not self.queue:
                print("队列为空，等待...")
                await self.condition.wait()  # 等待通知
            
            item = self.queue.pop(0)
            print(f"消费: {item}，队列长度: {len(self.queue)}")
            return item

async def producer(queue, n):
    for i in range(n):
        await asyncio.sleep(0.5)
        await queue.produce(f"item_{i}")

async def consumer(queue, n):
    for i in range(n):
        await queue.consume()
        await asyncio.sleep(1)

async def main():
    queue = AsyncQueue()
    
    await asyncio.gather(
        producer(queue, 5),
        consumer(queue, 5)
    )

asyncio.run(main())
```

**运行结果：**

```
队列为空，等待...
生产: item_0，队列长度: 1
消费: item_0，队列长度: 0
生产: item_1，队列长度: 1
生产: item_1，队列长度: 0
消费: item_2，队列长度: 1
生产: item_3，队列长度: 2
消费: item_2，队列长度: 1
生产: item_4，队列长度: 2
消费: item_3，队列长度: 1
消费: item_4，队列长度: 0
```

### 异步事件（Event）

用于协程间的信号传递：

```
import asyncio

async def waiter(event, name):
    print(f"{name} 等待事件...")
    await event.wait()
    print(f"{name} 收到事件，开始执行")

async def setter(event):
    await asyncio.sleep(2)
    print("触发事件")
    event.set()

async def main():
    event = asyncio.Event()
    
    await asyncio.gather(
        waiter(event, "任务1"),
        waiter(event, "任务2"),
        waiter(event, "任务3"),
        setter(event)
    )

asyncio.run(main())
```

**运行结果：**

```
任务1 等待事件...
任务2 等待事件...
任务3 等待事件...
触发事件
任务1 收到事件，开始执行
任务2 收到事件，开始执行
任务3 收到事件，开始执行
```

## 异步队列（Queue）

asyncio的Queue是线程安全的，适合协程间通信：

```
import asyncio
import random

async def producer(queue, producer_id):
    for i in range(5):
        item = f"P{producer_id}-Item{i}"
        await queue.put(item)
        print(f"生产者 {producer_id} 生产: {item}")
        await asyncio.sleep(random.uniform(0.1, 0.5))
    
    await queue.put(None)  # 结束标志

async def consumer(queue, consumer_id):
    while True:
        item = await queue.get()
        if item is None:
            await queue.put(None)  # 传递结束信号
            break
        
        print(f"消费者 {consumer_id} 消费: {item}")
        await asyncio.sleep(random.uniform(0.2, 0.6))
        queue.task_done()

async def main():
    queue = asyncio.Queue(maxsize=10)
    
    producers = [asyncio.create_task(producer(queue, i)) for i in range(2)]
    consumers = [asyncio.create_task(consumer(queue, i)) for i in range(3)]
    
    await asyncio.gather(*producers)
    await queue.join()  # 等待所有任务完成
    
    # 取消消费者任务
    for c in consumers:
        c.cancel()

asyncio.run(main())
```

### 优先级队列

```
import asyncio
from dataclasses import dataclass, field
from typing import Any

@dataclass(order=True)
class PriorityItem:
    priority: int
    data: Any = field(compare=False)

async def main():
    queue = asyncio.PriorityQueue()
    
    # 添加不同优先级的任务
    await queue.put(PriorityItem(3, "低优先级"))
    await queue.put(PriorityItem(1, "高优先级"))
    await queue.put(PriorityItem(2, "中优先级"))
    
    while not queue.empty():
        item = await queue.get()
        print(f"处理: {item.data} (优先级: {item.priority})")

asyncio.run(main())
```

**运行结果：**

```
处理: 高优先级 (优先级: 1)
处理: 中优先级 (优先级: 2)
处理: 低优先级 (优先级: 3)
```

## 异步迭代器和生成器

### 异步迭代器

```
import asyncio

class AsyncRange:
    def __init__(self, start, end):
        self.start = start
        self.end = end
        self.current = start
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        if self.current >= self.end:
            raise StopAsyncIteration
        
        await asyncio.sleep(0.1)  # 模拟异步操作
        value = self.current
        self.current += 1
        return value

async def main():
    async for i in AsyncRange(0, 5):
        print(f"值: {i}")

asyncio.run(main())
```

### 异步生成器

```
import asyncio

async def async_generator(n):
    for i in range(n):
        await asyncio.sleep(0.5)
        yield f"item_{i}"

async def main():
    async for item in async_generator(5):
        print(f"收到: {item}")

asyncio.run(main())
```

## 异常处理和取消

### 异常处理

```
import asyncio

async def task_may_fail(task_id, should_fail=False):
    await asyncio.sleep(1)
    if should_fail:
        raise ValueError(f"任务 {task_id} 失败")
    return f"任务 {task_id} 成功"

async def main():
    tasks = [
        asyncio.create_task(task_may_fail(1, False)),
        asyncio.create_task(task_may_fail(2, True)),
        asyncio.create_task(task_may_fail(3, False)),
    ]
    
    # gather会在遇到第一个异常时停止，除非设置return_exceptions=True
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f"任务 {i} 异常: {result}")
        else:
            print(f"任务 {i} 结果: {result}")

asyncio.run(main())
```

**运行结果：**

```
任务 0 结果: 任务 1 成功
任务 1 异常: 任务 2 失败
任务 2 结果: 任务 3 成功
```

### 任务取消

```
import asyncio

async def long_task(name):
    try:
        print(f"{name} 开始")
        await asyncio.sleep(10)
        print(f"{name} 完成")
    except asyncio.CancelledError:
        print(f"{name} 被取消")
        raise  # 重新抛出，让调用者知道任务被取消

async def main():
    task = asyncio.create_task(long_task("长任务"))
    
    await asyncio.sleep(2)
    print("取消任务")
    task.cancel()
    
    try:
        await task
    except asyncio.CancelledError:
        print("任务已确认取消")

asyncio.run(main())
```

## 常用异步库

### aiohttp - 异步HTTP客户端和服务器

```
import aiohttp
import asyncio

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    urls = [
        'http://httpbin.org/delay/1',
        'http://httpbin.org/delay/2',
    ]
    
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        print(f"获取了 {len(results)} 个响应")

# asyncio.run(main())  # 实际使用时取消注释
```

### aiofiles - 异步文件操作

```
import aiofiles
import asyncio

async def process_files():
    # 并发读写多个文件
    async def write_file(filename, content):
        async with aiofiles.open(filename, 'w') as f:
            await f.write(content)
    
    async def read_file(filename):
        async with aiofiles.open(filename, 'r') as f:
            return await f.read()
    
    # 并发写入
    await asyncio.gather(
        write_file('file1.txt', '内容1'),
        write_file('file2.txt', '内容2'),
        write_file('file3.txt', '内容3')
    )
    
    # 并发读取
    contents = await asyncio.gather(
        read_file('file1.txt'),
        read_file('file2.txt'),
        read_file('file3.txt')
    )
    
    print(contents)

# asyncio.run(process_files())
```

### aiomysql - 异步MySQL客户端

```
import aiomysql
import asyncio

async def mysql_example():
    conn = await aiomysql.connect(
        host='localhost',
        user='root',
        password='password',
        db='testdb'
    )
    
    async with conn.cursor() as cursor:
        await cursor.execute("SELECT * FROM users")
        result = await cursor.fetchall()
        print(result)
    
    conn.close()

# asyncio.run(mysql_example())
```

### 其他常用异步库

* • **aioredis / redis[async]**: 异步Redis客户端
* • **motor**: 异步MongoDB驱动
* • **asyncpg**: 高性能异步PostgreSQL驱动
* • **httpx**: 支持async/await的HTTP客户端
* • **websockets**: 异步WebSocket库
* • **aio-pika**: 异步RabbitMQ客户端
* • **aiokafka**: 异步Kafka客户端
* • **aiosmtplib**: 异步SMTP客户端
* • **aiodns**: 异步DNS解析
* • **trio**: 替代asyncio的异步框架，更现代化的API

## 高级话题

### 自定义事件循环策略

在某些场景下，需要自定义事件循环的行为：

```
import asyncio
import uvloop  # 高性能事件循环

# 使用uvloop替代默认事件循环（需要安装：pip install uvloop）
# asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())

async def main():
    print(f"当前事件循环: {type(asyncio.get_event_loop())}")

asyncio.run(main())
```

### 协程嵌套与任务组（Task Groups）

Python 3.11引入了TaskGroup，提供更好的结构化并发：

```
import asyncio

async def task_with_exception(name, should_fail=False):
    await asyncio.sleep(1)
    if should_fail:
        raise ValueError(f"{name} 失败")
    print(f"{name} 完成")
    return name

async def main():
    # Python 3.11+
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(task_with_exception("任务1", False))
            tg.create_task(task_with_exception("任务2", False))
            tg.create_task(task_with_exception("任务3", False))
        
        print("所有任务成功完成")
    except* ValueError as eg:
        print(f"捕获到 {len(eg.exceptions)} 个异常")
        for exc in eg.exceptions:
            print(f"  - {exc}")

# Python 3.11+
# asyncio.run(main())
```

### 异步上下文变量（ContextVar）

在异步环境中传递上下文信息，类似线程局部变量但对协程友好：

```
import asyncio
from contextvars import ContextVar

# 定义上下文变量
request_id = ContextVar('request_id', default='unknown')
user_info = ContextVar('user_info', default=None)

async def log(message):
    rid = request_id.get()
    user = user_info.get()
    print(f"[{rid}] [{user}] {message}")

async def process_request(req_id, user):
    # 设置上下文
    request_id.set(req_id)
    user_info.set(user)
    
    await log("开始处理请求")
    await asyncio.sleep(0.5)
    await log("处理完成")

async def main():
    # 每个请求都有独立的上下文
    await asyncio.gather(
        process_request("req-001", "Alice"),
        process_request("req-002", "Bob"),
        process_request("req-003", "Charlie")
    )

asyncio.run(main())
```

**运行结果：**

```
[req-001] [Alice] 开始处理请求
[req-002] [Bob] 开始处理请求
[req-003] [Charlie] 开始处理请求
[req-001] [Alice] 处理完成
[req-002] [Bob] 处理完成
[req-003] [Charlie] 处理完成
```

### 异步信号处理

在长期运行的异步应用中优雅关闭：

```
import asyncio
import signal

async def worker(name):
    try:
        while True:
            print(f"{name} 工作中...")
            await asyncio.sleep(2)
    except asyncio.CancelledError:
        print(f"{name} 收到取消信号，清理资源...")
        await asyncio.sleep(1)  # 模拟清理
        print(f"{name} 已清理完成")
        raise

async def main():
    # 创建工作任务
    tasks = [
        asyncio.create_task(worker("Worker1")),
        asyncio.create_task(worker("Worker2"))
    ]
    
    # 注册信号处理
    loop = asyncio.get_event_loop()
    stop_event = asyncio.Event()
    
    def signal_handler():
        print("\n收到停止信号，开始优雅关闭...")
        stop_event.set()
    
    for sig in (signal.SIGINT, signal.SIGTERM):
        loop.add_signal_handler(sig, signal_handler)
    
    # 等待停止信号
    await stop_event.wait()
    
    # 取消所有任务
    for task in tasks:
        task.cancel()
    
    # 等待所有任务清理完成
    await asyncio.gather(*tasks, return_exceptions=True)
    print("应用已关闭")

# asyncio.run(main())  # 运行后按Ctrl+C测试
```

## 性能优化技巧

### 1. 避免在循环中创建任务

```
import asyncio

# ❌ 不好的做法
async def bad_practice():
    for i in range(1000):
        await asyncio.create_task(some_async_func(i))

# ✅ 好的做法
async def good_practice():
    tasks = [asyncio.create_task(some_async_func(i)) for i in range(1000)]
    await asyncio.gather(*tasks)
```

### 2. 使用信号量控制并发

```
import asyncio

async def controlled_concurrency():
    semaphore = asyncio.Semaphore(10)  # 最多10个并发
    
    async def limited_task(i):
        async with semaphore:
            await some_async_func(i)
    
    tasks = [limited_task(i) for i in range(1000)]
    await asyncio.gather(*tasks)
```

### 3. 批量处理减少上下文切换

```
import asyncio

async def batch_processing(items, batch_size=100):
    for i in range(0, len(items), batch_size):
        batch = items[i:i + batch_size]
        tasks = [process_item(item) for item in batch]
        await asyncio.gather(*tasks)
        # 批次间可以做检查点或限流
        await asyncio.sleep(0.1)
```

## 常见陷阱与解决方案

### 1. 阻塞事件循环

```
import asyncio
import time

# ❌ 错误：使用time.sleep阻塞整个事件循环
async def bad_sleep():
    time.sleep(1)  # 阻塞！其他协程无法执行

# ✅ 正确：使用asyncio.sleep
async def good_sleep():
    await asyncio.sleep(1)  # 释放控制权
```

### 2. 忘记await

```
# ❌ 错误：忘记await，协程不会执行
async def bad_call():
    result = some_async_func()  # 只是获得协程对象

# ✅ 正确
async def good_call():
    result = await some_async_func()
```

### 3. 在同步函数中调用异步函数

```
# ❌ 错误
def sync_function():
    result = await async_function()  # SyntaxError

# ✅ 正确方式1：使用asyncio.run
def sync_function():
    result = asyncio.run(async_function())

# ✅ 正确方式2：获取现有循环
def sync_function():
    loop = asyncio.get_event_loop()
    result = loop.run_until_complete(async_function())
```

## 调试和监控

### 启用asyncio调试模式

```
import asyncio
import warnings

# 启用调试模式
asyncio.run(main(), debug=True)

# 或者
# import os
# os.environ['PYTHONASYNCIODEBUG'] = '1'

# 显示未await的协程警告
warnings.simplefilter('always', ResourceWarning)
```

### 监控事件循环

```
import asyncio

async def monitor_loop():
    loop = asyncio.get_event_loop()
    
    while True:
        # 获取当前所有任务
        tasks = asyncio.all_tasks(loop)
        print(f"当前任务数: {len(tasks)}")
        
        # 检查是否有长时间运行的任务
        for task in tasks:
            if not task.done():
                print(f"  未完成任务: {task.get_name()}")
        
        await asyncio.sleep(5)

async def main():
    monitor = asyncio.create_task(monitor_loop())
    
    # 你的其他任务
    await asyncio.sleep(20)
    
    monitor.cancel()

asyncio.run(main())
```

## 总结

异步编程是Python处理高并发场景的利器，但也需要理解其工作原理和最佳实践。本文涵盖了从基础的async/await语法到高级的并发控制，从同步原语到与多线程、多进程的协作，希望能帮助你构建高性能的异步应用。

### 关键要点回顾

1. 1. **基础概念**：协程是可以暂停和恢复的函数，通过事件循环调度执行
2. 2. **并发控制**：使用Lock、Semaphore、Event等同步原语保证线程安全
3. 3. **资源管理**：异步上下文管理器确保资源正确释放
4. 4. **性能优化**：控制并发数量、使用批处理、避免阻塞操作
5. 5. **错误处理**：使用try-except捕获异常，正确处理取消信号
6. 6. **混合编程**：在异步中执行同步代码，结合线程池和进程池

### 选择异步的场景

* • ✅ IO密集型任务（网络请求、文件读写、数据库查询）
* • ✅ 需要处理大量并发连接（WebSocket、长轮询）
* • ✅ 微服务间的异步通信
* • ❌ CPU密集型计算（考虑多进程）
* • ❌ 简单的脚本任务（增加复杂度不值得）

### 延伸阅读

* • Python官方asyncio文档
* • aiohttp文档
* • Real Python的异步教程
* • asyncio源码

掌握异步编程需要实践，建议从简单的异步HTTP请求开始，逐步过渡到复杂的生产环境应用。记住，异步不是银弹，合适的场景用合适的工具才是王道。