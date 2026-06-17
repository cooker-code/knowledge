---
title: Python asyncio 教程：从第一性原理深入解析（第二部分）
author: 送君千里平川
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg2NDYyMTc3Nw==&mid=2247484529&idx=1&sn=14e1227ccd16c7b75f96b60268a31d1f&chksm=cf94c438ccbc261fcedd095e0386e7c2c903c6ed5283408fb413f729257d6bd1951fcbfea7af&mpshare=1&scene=24&srcid=1001AIQOlEZ1nGU6jRTmpQyb&sharer_shareinfo=be730bd929992402f294ab55dacfd820&sharer_shareinfo_first=be730bd929992402f294ab55dacfd820#rd
---

是时候补充点`asyncio`的知识了……下面内容由`gemini-2.5-pro`对ytb`Corey Schafer`对应视频的提取，仅作为备份备查，推荐学习原视频。

这是第二部分，第一部分见[Python asyncio 教程：从第一性原理深入解析（第一部分）](https://mp.weixin.qq.com/s?__biz=Mzg2NDYyMTc3Nw==&mid=2247484525&idx=1&sn=8db0f80c2bed69192b3c76b409ce9615&scene=21#wechat_redirect)。

### 第六章：一次性处理多个任务

到目前为止，我们都是逐个创建和等待任务。对于少数几个任务来说这没问题，但如果你有几十、几百甚至几千个任务呢？手动管理它们将是一场噩梦。`asyncio` 提供了强大的工具来并发运行许多任务：`asyncio.gather` 和更现代的 `asyncio.TaskGroup`。

#### `asyncio.gather` - 收集可等待对象

`asyncio.gather` 是一个简单直接的方法，用于并发运行一个可等待对象（协程或任务）的列表。

让我们看看 `example_7.py`。我们可以用列表推导式来创建任务，而不是在不同的行上创建 `task1` 和 `task2`。

**收集协程 (Gathering Coroutines):**

```
# 收集协程  
coroutines = [fetch_data(i) for i in range(1, 3)]  
results = await asyncio.gather(*coroutines)  
print(f"协程结果: {results}")
```

在这里，我们创建了一个协程对象的列表。`coroutines` 前面的 `*` 至关重要——它会解包列表，将每个协程作为独立的参数传递给 `gather`。然后 `gather` 会自动将这些协程包装成任务并运行它们。

**收集任务 (Gathering Tasks):**你也可以先显式创建任务，然后再收集它们。

```
# 收集任务  
tasks = [asyncio.create_task(fetch_data(i)) for i in range(1, 3)]  
results = await asyncio.gather(*tasks)  
print(f"任务结果: {results}")
```

细微的区别在于任务*何时*被调度。使用 `asyncio.create_task`，任务会立即被调度到事件循环上。而当收集原始协程时，它们在 `gather` 被调用时才被调度。

#### `asyncio.TaskGroup` - 结构化并发

`TaskGroup` 在 Python 3.11 中引入，提供了所谓的“结构化并发”。它是一种更清晰、更安全的方式，使用 `async with` 块来管理一组任务。

```
# 任务组 (Task Group)  
async with asyncio.TaskGroup() as tg:  
    tasks = [tg.create_task(fetch_data(i)) for i in range(1, 3)]  
# 当上下文管理器退出时，所有任务都已被等待。  
results = [task.result() for task in tasks]  
print(f"任务组结果: {results}")
```

`TaskGroup` 的神奇之处在于 `async with` 块在其中创建的所有任务（`tg.create_task`）完成之前不会退出。这就不再需要之后手动 `await` 结果，并确保你不会意外地让任务在后台运行。

#### `gather` vs. `TaskGroup`：错误处理上的关键区别

推荐使用 `TaskGroup` 而不是 `gather` 的主要原因是其更优越的错误处理机制。

* **`asyncio.gather` (默认行为):** 如果组中的任何任务引发异常，`gather` 会立即取消 `await` 并传播它遇到的第一个异常。然而，组中的其他任务*不会*被取消，并会继续在后台运行。这些被称为“孤儿任务”，可能导致不可预测的行为和资源泄漏。
* **`asyncio.gather(return_exceptions=True)`:** 你可以告诉 `gather` 等待所有任务完成，并将任何异常作为结果列表的一部分返回。这可以防止孤儿任务，但需要你手动检查结果中的每一项，看它是一个成功的结果还是一个异常对象。
* **`asyncio.TaskGroup`:** 这是最安全的选择。如果组内的任何任务失败，`TaskGroup` 会自动**取消组中所有其他任务**。然后它会等待它们全部结束，并引发一个包含所有发生异常的 `ExceptionGroup`。这种“快速失败”和清理行为更加健壮和可预测。

对于新代码，**`asyncio.TaskGroup` 几乎总是更好的选择。**

### 第七章：真实世界示例 - 异步图片下载器

现在，让我们将所学的一切应用到一个真实世界的问题上。我们有一个同步的 Python 脚本，它做两件事：

1. **下载图片：** 它接受一个 URL 列表，并使用阻塞的 `requests` 库逐一下载它们。这是一个 **I/O 密集型 (I/O-bound)** 操作。
2. **处理图片：** 它接收下载的图片，并使用 `Pillow` 库为每张图片应用边缘检测滤镜。这涉及大量的像素操作，是一个 **CPU 密集型 (CPU-bound)** 操作。

同步运行这个脚本大约需要 **23.5 秒**。让我们看看如何能加快它。

#### 步骤 1：性能分析以寻找瓶颈

在优化之前，我们需要知道时间花在了哪里。使用像 `scalene` 这样的性能分析工具（`uv run -m scalene real_world_example_sync_v1.py`），我们可以分析代码的性能。

分析报告证实了我们的假设：

* `download_single_image` 函数大部分时间花在“系统（System）”上，这意味着它在等待外部资源（网络）。**这是一个完美的并发候选者。**
* `process_single_image` 函数大部分时间花在“Python”上，意味着 CPU 正在积极地进行数值计算。**这是一个 CPU 密集型任务。**

#### 步骤 2：为任务选择合适的工具

根据我们的性能分析，我们可以制定一个策略：

1. **下载 (I/O 密集型):** 我们可以并发地运行这些任务。由于 `requests` 库是阻塞的，我们需要在单独的**线程**中运行每次下载。一个更好的长期解决方案是使用像 `httpx` 这样的原生异步 HTTP 库。
2. **处理 (CPU 密集型):** 为了获得真正的并行性并绕过 Python 的全局解释器锁 (GIL)，我们需要在单独的**进程**中运行这项工作。

#### 步骤 3：重构代码

让我们将同步脚本转换为一个高性能的异步脚本。

**1. 将主函数转换为 `async`**首先，我们将 `download_images`、`process_images` 和 `main` 函数通过添加 `async` 关键字变成协程。我们将使用 `asyncio.run(main())` 来运行顶层的 `main` 函数。

**2. 使用 `httpx` 和 `aiofiles` 进行异步下载**我们将不再在线程中运行阻塞的 `requests` 库，而是改用 `httpx` 和 `aiofiles` 来获得原生的异步性能。

```
# pip install httpx aiofiles  
import httpx  
import aiofiles  
  
asyncdef download_single_image(client, url, ...):  
    response = await client.get(url, timeout=10, follow_redirects=True)  
    response.raise_for_status()  
  
    # ... 文件名逻辑 ...  
  
    asyncwith aiofiles.open(download_path, "wb") as f:  
        asyncfor chunk in response.aiter_bytes():  
            await f.write(chunk)
```

注意我们现在 `await` 了网络请求 (`client.get`) 和每次文件写入 (`f.write`)。我们甚至使用 `async for` 循环来迭代流入的响应数据。

**3. 使用 `Semaphore` 控制并发度**我们不希望一次性发送数千个请求。我们可以使用 `asyncio.Semaphore` 来限制并发度。

```
DOWNLOAD_LIMIT = 5  
dl_semaphore = asyncio.Semaphore(DOWNLOAD_LIMIT)  
  
async def download_single_image(client, url, ..., semaphore):  
    async with semaphore:  # 如果已有 5 个下载在进行，这里会等待  
        # ... 上面的下载逻辑 ...
```

现在，最多只会有 5 个下载任务同时运行。

**4. 使用 `ProcessPoolExecutor` 进行 CPU 密集型处理**对于我们的 `process_images` 函数，我们将使用进程池来运行 CPU 密集型的工作。

```
from concurrent.futures import ProcessPoolExecutor  
  
async def process_images(orig_paths):  
    loop = asyncio.get_running_loop()  
    with ProcessPoolExecutor() as executor:  
        tasks = [  
            loop.run_in_executor(executor, process_single_image, path)  
            for path in orig_paths  
        ]  
        processed_paths = await asyncio.gather(*tasks)  
    return processed_paths
```

#### 最终结果

通过对脚本的每个部分应用正确的并发策略，总执行时间从 **23.5 秒** 降到了 **5 秒** 以下。

* **图片下载 (I/O 密集型):** 由 `asyncio` 和 `httpx` 处理，在单个线程上并发运行。
* **图片处理 (CPU 密集型):** 由 `ProcessPoolExecutor` 处理，在多个 CPU 核心上并行运行。

### 第八章：总结与最佳实践

我们已经涵盖了很多内容，从核心概念到完整的真实世界重构。这里有一个快速的思考清单，当你在自己的项目中处理并发时可以参考：

1. **这项工作是 I/O 密集型还是 CPU 密集型？** 如果不确定，就使用性能分析工具。
2. **对于 I/O 密集型工作：**

* **最佳选择：** 使用原生异步库（`httpx`, `aiofiles`, `asyncpg` 等）和 `async`/`await`。
* **不错选择：** 如果必须使用阻塞库，请使用 `asyncio.to_thread` 或 `loop.run_in_executor` 在线程池中运行它。

3. **对于 CPU 密集型工作：**

* 使用 `loop.run_in_executor` 和进程池来实现真正的并行。

4. **管理任务组：** 使用 `asyncio.TaskGroup` 进行结构化、安全和健壮的任务管理。
5. **避免常见错误：**

* 记住要 `await` 你的协程和任务。
* 永远不要在协程内部直接调用阻塞代码。
* 在开发过程中使用像 Ruff 这样的代码检查工具，并用 `asyncio.run(debug=True)` 运行你的代码来捕获常见错误。

Python 中的异步编程是构建高性能应用程序的一个极其强大的工具。虽然初始概念可能有些棘手，但理解事件循环、I/O 与 CPU 密集型任务的基础知识，以及 `asyncio` 提供的工具，将使你能够编写更快、更高效的代码。