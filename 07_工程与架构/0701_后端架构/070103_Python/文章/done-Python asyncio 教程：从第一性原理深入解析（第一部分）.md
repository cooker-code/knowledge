> 已吸收至：[[07_工程与架构/0701_后端架构/070103_Python/070103_核心知识点/Python来源校准与降权准则|Python来源校准与降权准则]]、[[07_工程与架构/0701_后端架构/070103_Python/070103_知识地图|070103_Python知识地图]]

---
title: Python asyncio 教程：从第一性原理深入解析（第一部分）
author: 送君千里平川
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg2NDYyMTc3Nw==&mid=2247484525&idx=1&sn=8db0f80c2bed69192b3c76b409ce9615&chksm=cf705b6fecf2d8308247b1e409ca85e61c15ff232c7873a322a9cfe7c78057697d43a261deb1&mpshare=1&scene=24&srcid=1001TiV2XmMQH6wDkqx2978v&sharer_shareinfo=c6d06c140d9dea139b031a75123df5e2&sharer_shareinfo_first=c6d06c140d9dea139b031a75123df5e2#rd
---

是时候补充点`asyncio`的知识了……下面内容由`gemini-2.5-pro`对ytb`Corey Schafer`对应视频的提取，仅作为备份备查，推荐学习原视频。

异步编程的学习曲线可能相当陡峭，尤其是在 Python 中使用 `asyncio` 库时。面对协程（coroutines）、事件循环（event loops）、任务（tasks）和期物（futures）等术语，我们很容易感到迷茫。但如果能从头开始，直观地看到每个部分是如何工作的，情况会怎样呢？

本教程旨在揭开 `asyncio` 的神秘面纱。我们将从基本概念入手，逐步构建实际示例，并使用可视化动画来精确解释底层发生的一切。在这两部分的系列文章结束时，您将对 `asyncio` 的工作原理以及如何有效使用它有扎实的掌握。

在第一部分中，我们将涵盖：

* 同步执行与并发执行的核心概念。
* 关键术语：协程、任务、期物和事件循环。
* 如何正确地将同步代码转换为异步代码。
* 阻塞事件循环的致命错误以及如何避免它。
* 如何使用线程和进程将阻塞的同步代码集成到 `asyncio` 应用程序中。

## 第一章：核心思想 - 同步代码 vs. 并发代码

在深入研究代码之前，让我们用一个简单的类比来理解同步执行和并发执行的根本区别。

* **同步（赛百味三明治排队）：** 当你在赛百味点三明治时，店员会接你的单，然后从头到尾制作你的整个三明治，之后才会与队伍中的下一个人交流。一个任务必须完全完成后，下一个任务才能开始。大多数标准的 Python 代码就是这样运行的。
* **并发（麦当劳汽车穿梭餐厅）：** 在快餐店的汽车穿梭餐厅，一个人负责接你的订单和收款，然后立即去接下一个人的订单。与此同时，后台的另一个团队已经在准备你的食物了。他们在同一时间处理多个订单，在“等待”期间（比如你的食物正在烹饪时）处理其他事情。这就是并发的本质。

需要理解的一个关键点是，**异步并不自动意味着更快**。它意味着更高效。我们的程序不是在等待某件事（如网络请求或数据库查询返回）时无所事事，而是可以切换去做其他有用的工作。这对于 **I/O密集型（I/O-bound）** 任务尤其强大——即那些你的程序大部分时间都在等待外部资源的任务。

## 第二章：Asyncio 的构建模块

让我们从一个简单的 Python 文件开始，定义我们的关键术语。我们将有两个函数：一个同步运行，一个异步运行。

```
# terms.py

import asyncio
import time

# 一个常规的同步函数
def sync_function(test_param: str) -> str:
    print("这是一个同步函数。")
    time.sleep(0.1) # 这是一个阻塞式休眠
    return f"同步结果: {test_param}"

# 一个异步函数，也被称为协程函数
async def async_function(test_param: str) -> str:
    print("这是一个异步协程函数。")
    await asyncio.sleep(0.1) # 这是一个非阻塞式休眠
    return f"异步结果: {test_param}"

async def main():
    # 我们将在这里运行我们的代码
    sync_result = sync_function("Test")
    print(sync_result)

if __name__ == "__main__":
    asyncio.run(main())
```

这段代码引入了几个关键的概念和术语。

### 协程（Coroutines）

协程是使用 `async def` 定义的特殊函数。当你调用一个协程函数，比如 `async_function("Test")` 时，它不会立即运行里面的代码。相反，它会返回一个**协程对象（coroutine object）**。这个对象是一个可等待对象（awaitable），意味着你可以对它使用 `await` 关键字来实际执行它。

```
# 这会创建一个协程对象，但不会运行函数内的代码
coroutine_obj = async_function("Test") 

# 这将运行协程并获取结果
coroutine_result = await coroutine_obj 
```

### 事件循环（The Event Loop）

事件循环是 `asyncio` 的核心。可以把它想象成一个调度器，负责管理和运行你所有的异步任务。它跟踪哪些任务正在运行，哪些在等待 I/O，哪些已准备好下次运行。

现在你通常不再需要直接与事件循环交互。你通过调用 `asyncio.run()` 并传入你的主入口点协程来启动它。

```
# 这会获取事件循环，运行 main 协程直到它完成，
# 然后关闭循环。
asyncio.run(main())
```

### 可等待对象（awaitables）

可等待对象是任何可以与 `await` 关键字一起使用的对象。`await` 关键字告诉事件循环：“在此处暂停当前协程的执行，去做其他工作，直到这个可等待对象完成。”

主要有三种类型的可等待对象：**协程（Coroutines）**、**任务（Tasks）** 和 **期物（Futures）**。

#### 期物（Futures）

Future 是一个低层级对象，代表异步操作的最终结果。它类似于 JavaScript 中的 `Promise`。你通常不会直接创建 Future，而是使用更高级别的 `Task` 和 `coroutine` 对象。

```
# 一个使用 Future 的低层级示例
loop = asyncio.get_running_loop()
future = loop.create_future()
print(f"空的 Future: {future}") # 打印一个 pending 状态的 future

# ... 在稍后的某个时刻，你可以设置结果
future.set_result("Future 结果: Test")

# 然后等待结果
future_result = await future
print(future_result)
```

#### 任务（Tasks）

任务是我们用来调度协程在事件循环上*并发*运行的方式。任务是协程的包装器。当你使用 `asyncio.create_task()` 将协程包装成任务时，你就是把它交给了事件循环，让它尽快运行，而不会阻塞当前函数。这是实现并发的关键。

```
# 创建一个任务会将协程调度到事件循环上
task = asyncio.create_task(async_function("Test"))

# 现在 main 函数可以在任务运行时做其他事情

# 为了获取结果，我们稍后 await 这个任务
task_result = await task
```

## 第三章：从同步到并发 - 一个实践演练

让我们看一个获取数据的实际例子，来观察这些概念的实际应用。

### 示例 1：同步基线

这是一个标准的同步程序。每次调用 `fetch_data` 都会阻塞指定的秒数。

```
# example_1.py
import time

def fetch_data(param):
    print(f"处理 {param}...")
    time.sleep(param)
    print(f"处理 {param} 完成")
    return f"{param} 的结果"

def main():
    result1 = fetch_data(1)
    print("获取 1 完全完成")
    result2 = fetch_data(2)
    print("获取 2 完全完成")
    return [result1, result2]

# ... 运行 main() 并计时的代码 ...
```

执行是严格顺序的：

1. `fetch_data(1)` 运行 1 秒。
2. `fetch_data(2)` 运行 2 秒。

总运行时间约为 **3 秒**。

### 示例 2：常见的错误 - 顺序 `await`

现在让我们把它转换成 `async`。一个常见的初次尝试是这样的：

```
# example_2.py (带有错误)
import asyncio

async def fetch_data(param):
    print(f"处理 {param}...")
    await asyncio.sleep(param)
    print(f"处理 {param} 完成")
    return f"{param} 的结果"

async def main():
    # 这仍然是顺序执行的！
    result1 = await fetch_data(1)
    print("任务 1 完全完成")
    result2 = await fetch_data(2)
    print("任务 2 完全完成")
    return [result1, result2]

# ... asyncio.run(main()) ...
```

虽然这段代码现在是异步的，但它不是并发的。第一行的 `await` 会暂停 `main` 函数，直到 `fetch_data(1)`*完全完成*。只有在那之后，它才会继续开始执行 `fetch_data(2)`。

总运行时间仍然是 **3 秒**。我们没有获得任何性能提升。

### 示例 3：使用 `asyncio.create_task()` 实现真正的并发

为了解决这个问题并并发地运行任务，我们必须在等待它们的结果*之前*，将它们调度到事件循环上。我们用 `asyncio.create_task()` 来实现这一点。

```
# example_3.py (正确的并发)
import asyncio

# fetch_data 与上面相同

async def main():
    # 立即在事件循环上调度这两个任务
    task1 = asyncio.create_task(fetch_data(1))
    task2 = asyncio.create_task(fetch_data(2))

    # 现在，等待结果。事件循环可以并发地运行它们。
    result1 = await task1
    print("任务 1 完全完成")
    result2 = await task2
    print("任务 2 完全完成")
    return [result1, result2]

# ... asyncio.run(main()) ...
```

发生了什么：

1. `task1` 被创建并调度。`main` 函数没有阻塞。
2. `task2` 被创建并调度。`main` 函数没有阻塞。
3. `main` 函数遇到 `await task1` 并暂停。
4. 事件循环现在空闲了。它看到两个准备好的任务（`task1` 和 `task2`）并开始运行它们。当 `task1` 遇到 `await asyncio.sleep(1)` 时，它交出控制权。事件循环接着运行 `task2`。当 `task2` 遇到 `await asyncio.sleep(2)` 时，它也交出控制权。
5. 事件循环等待。1 秒后，`task1` 完成。2 秒后，`task2` 完成。

总运行时间现在大约是 **2 秒**，即最长任务的持续时间！

## 第四章：原罪 - 阻塞事件循环

如果我们在一个 `async` 函数内部使用一个同步的阻塞调用，比如 `time.sleep()`，会发生什么？

```
# example_5.py (阻塞事件循环 - 不要这样做)
import asyncio
import time

async def fetch_data(param):
    print(f"处理 {param}...")
    # 这是一个阻塞调用！它会冻结整个事件循环。
    time.sleep(param) 
    print(f"处理 {param} 完成")
    return f"{param} 的结果"

async def main():
    task1 = asyncio.create_task(fetch_data(1))
    task2 = asyncio.create_task(fetch_data(2))
    
    result1 = await task1
    result2 = await task2
    return [result1, result2]
```

当事件循环开始运行 `fetch_data(1)` 并遇到 `time.sleep(1)` 时，整个 Python 线程都会被冻结。事件循环被阻塞了，无法切换到 `task2` 或做任何其他事情。它必须等待那 1 秒钟过去。之后，它运行 `fetch_data(2)` 并再次冻结 2 秒钟。

结果呢？总运行时间回到了 **3 秒**。在协程内部使用阻塞调用完全违背了 `asyncio` 的初衷。

## 第五章：处理阻塞代码 - 线程和进程

那么，如果你需要使用一个只提供阻塞、同步函数的库（比如流行的 `requests` 库），该怎么办？你不能直接在协程中调用它，因为那会阻塞事件循环。

解决方案是在一个单独的线程或进程中运行阻塞代码，而 `asyncio` 可以为你管理这些。

### 在线程中运行

你可以使用 `asyncio.to_thread` 在一个单独的线程中运行任何阻塞函数。这让你的主事件循环可以自由地运行其他任务。

```
# example_6.py (部分代码)

# 注意：fetch_data 现在又变回一个常规的同步函数了
def fetch_data(param):
    print(f"处理 {param}...")
    time.sleep(param)
    print(f"处理 {param} 完成")
    return f"{param} 的结果"

async def main():
    # # 在线程中运行
    task1 = asyncio.create_task(asyncio.to_thread(fetch_data, 1))
    task2 = asyncio.create_task(asyncio.to_thread(fetch_data, 2))

    result1 = await task1
    print("线程 1 完全完成")
    result2 = await task2
    print("线程 2 完全完成")
    # ...
```

在这里，`asyncio` 管理一个线程池，并在其中一个线程中运行我们的阻塞函数 `fetch_data`。事件循环可以自由地继续它的工作，从而实现了大约 **2 秒** 的并发执行时间。

### 在进程中运行

对于 **CPU密集型（CPU-bound）** 任务（大量计算、数据处理），由于 Python 的全局解释器锁（GIL），线程是不够的。为了实现真正的并行，你需要使用独立的进程。`asyncio` 也可以管理一个进程池。

```
# example_6.py (部分代码)
from concurrent.futures import ProcessPoolExecutor

# ... fetch_data 是同一个常规函数 ...

async def main():
    # # 在进程池中运行
    loop = asyncio.get_running_loop()
    with ProcessPoolExecutor() as executor:
        task1 = loop.run_in_executor(executor, fetch_data, 1)
        task2 = loop.run_in_executor(executor, fetch_data, 2)

        result1 = await task1
        print("进程 1 完全完成")
        result2 = await task2
        print("进程 2 完全完成")
        return [result1, result2]
```

这种方法更复杂一些。我们获取当前的事件循环，创建一个 `ProcessPoolExecutor`，然后使用 `loop.run_in_executor` 在一个单独的进程中运行我们的函数。结果同样是并发的，由于创建新进程的开销，耗时略多于 **2 秒**。

## 未完待续...

我们已经涵盖了 `asyncio` 的基础概念，从基本术语到实现并发的正确方法，甚至包括如何处理阻塞代码。

在**第二部分**中，我们将在此基础上探索更高级和实用的模式，包括：

* 如何一次性从多个任务中收集结果。
* 处理超时和取消。
* 一个从多个网页 URL 获取数据的真实世界示例。
* 分析我们的代码，看看 `asyncio` 在哪些方面能提供最大的好处。