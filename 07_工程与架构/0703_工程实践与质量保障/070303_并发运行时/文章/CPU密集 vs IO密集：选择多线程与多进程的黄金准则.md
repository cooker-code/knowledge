---
title: CPU密集 vs IO密集：选择多线程与多进程的黄金准则
author: PYant
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYyNTM2MzkyNA==&mid=2247485194&idx=1&sn=1985c45d0577811d1c169b04ef955ad7&chksm=f1a97948bd4775289420d05fb51f6edca02891204da6c1620730c68a43c37e2739290de20a1b&mpshare=1&scene=24&srcid=04146JG0XUoS7gCMX4B3hpW9&sharer_shareinfo=68c7afee5759e31e4c68fa4a42742cf7&sharer_shareinfo_first=68c7afee5759e31e4c68fa4a42742cf7#rd
---

前面分享了Python多线程和多进程， 涉及到程序的任务类型：CPU密集型 和 IO密集型。可能不知道什么时候选择多进程？什么时候选择多线程？

今天我们就来聊聊程序任务的分类方法。搞清楚了这一点，选择多进程还是多线程，就跟“头痛医头，脚痛医脚”一样简单。

## 核心概念：两种截然不同的“体质”

程序的任务，大体可以归为两类：CPU密集型 和 IO密集型。

### 1.1 基础定义：脑力vs体力

* • **CPU密集型**：任务的核心消耗是**计算**。程序大部分时间都在“烧脑”，进行各种数学运算、逻辑处理、数据压缩/解压、图形渲染等。它就像一个正在解复杂方程式的数学家，核心资源是**大脑（CPU）**。
* • **IO密集型**：任务的核心消耗是**输入/输出等待**。这里的“IO”是个广义概念，包括：读写磁盘文件、从网络下载/上传数据、查询数据库、等待用户输入等。程序大部分时间在“等待”外部设备返回结果。它就像一个不停收发快递的仓库管理员，核心状态是**等待（IO阻塞）**。

### 1.2 核心特点：认清它们的“本性”

**CPU密集型任务的特点：**

1. 1. **吃光CPU**：一个任务就能让一个CPU核心满载。
2. 2. **讨厌被打扰**：频繁的线程切换（上下文切换）会造成巨大的性能损耗。
3. 3. **并行是王道**：多个计算任务放在多个CPU核心上**同时**跑，才能线性提升速度。

**IO密集型任务的特点：**

1. 1. **CPU很闲**：大部分时间CPU都在“空转”，等待IO操作完成。
2. 2. **并发优势大**：当一个任务在等待IO时，CPU可以立刻去处理其他任务，不让CPU闲着。
3. 3. **切换开销低**：相比于CPU计算，线程间切换的开销在IO等待面前几乎可以忽略。

## 对症下药：多进程治“脑力”，多线程治“等待”

Python有个著名的**GIL（全局解释器锁）**，它让一个Python进程内的多个线程，在同一时刻只能有一个线程执行Python字节码。这直接决定了我们的选择策略：

* • **对于CPU密集型任务**：GIL是致命的。用多线程，CPU也无法被充分利用。此时，**多进程 (`multiprocessing`)** 是解药。每个进程有独立的Python解释器和内存空间，能绕开GIL，真正利用多核CPU。
* • **对于IO密集型任务**：GIL影响不大。因为线程大部分时间在IO阻塞，不占用CPU。使用**多线程 (`threading`)** 或**异步IO (`asyncio`)** ，可以在一个进程内用极低的成本实现高并发，在等待IO时切换到其他任务，完美利用时间。

### 2.1 场景与代码示例

**场景一：CPU密集型 - 计算大量质数**

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

59

60

61

62

63

64

65

66

67

  

import time  
import threading  
import multiprocessing  
   
def is_prime(n):  
    """判断一个数是否为质数（计算密集型函数）"""  
    if n < 2:  
        return False  
    for i in range(2, int(n**0.5) + 1):  
        if n % i == 0:  
            return False  
    return True  
   
def count_primes(start, end):  
    """计算从start到end之间的质数个数"""  
    count = 0  
    for num in range(start, end + 1):  
        if is_prime(num):  
            count += 1  
    print(f"区间 [{start}, {end}] 完成，找到 {count} 个质数")  
    return count  
   
def main():  
    range_start, range_end = 1, 200000  
    # 1. 单线程 baseline  
    print("【单线程】开始计算...")  
    start = time.time()  
    count_primes(range_start, range_end)  
    print(f"耗时: {time.time() - start:.2f} 秒\n")  
   
    # 2. 多线程尝试（错误示范）  
    print("【多线程】开始计算...")  
    start = time.time()  
    threads = []  
    # 分成4份  
    chunk = (range_end - range_start + 1) // 4  
    for i in range(4):  
        s = range_start + i * chunk  
        e = s + chunk - 1 if i < 3 else range_end  
        t = threading.Thread(target=count_primes, args=(s, e))  
        threads.append(t)  
        t.start()  
    for t in threads:  
        t.join()  
    print(f"耗时: {time.time() - start:.2f} 秒 (可能比单线程还慢！)\n")  
   
    # 3. 多进程（正确示范）  
    print("【多进程】开始计算...")  
    start = time.time()  
    processes = []  
    results = []  
    with multiprocessing.Pool(processes=4) as pool: # 使用进程池  
        # 划分任务  
        chunks = []  
        chunk = (range_end - range_start + 1) // 4  
        for i in range(4):  
            s = range_start + i * chunk  
            e = s + chunk - 1 if i < 3 else range_end  
            chunks.append((s, e))  
        # map方法提交任务  
        results = pool.starmap(count_primes, chunks)  
    total_primes = sum(results)  
    print(f"总质数个数: {total_primes}")  
    print(f"耗时: {time.time() - start:.2f} 秒 (速度显著提升！)")  
   
if __name__ == '__main__':  
    main()
```

**场景二：IO密集型 - 模拟多个网络请求**

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

  

import time  
import threading  
import concurrent.futures  
import requests  
   
def download_url(url):  
    """模拟一个耗时的网络请求（IO密集型）"""  
    print(f"开始下载: {url}")  
    time.sleep(2)  # 模拟2秒网络延迟  
    print(f"下载完成: {url}")  
    return f"{url} 的内容"  
   
def main():  
    urls = [  
        "https://httpbin.org/delay/1",  # 这个API会真的延迟1秒  
        "https://httpbin.org/delay/1",  
        "https://httpbin.org/delay/1",  
        "https://httpbin.org/delay/1",  
    ]  
   
    # 1. 单线程  
    print("【顺序下载】")  
    start = time.time()  
    for url in urls:  
        # 实际请求，这里用time.sleep模拟  
        time.sleep(1)  
        print(f"完成: {url}")  
    print(f"总耗时: {time.time() - start:.2f} 秒\n")  
   
    # 2. 多线程（正确选择）  
    print("【并发下载 - 多线程】")  
    start = time.time()  
   
    def worker(url):  
        time.sleep(1)  
        print(f"完成: {url}")  
        return url  
   
    with concurrent.futures.ThreadPoolExecutor(max_workers=4) as executor:  
        futures = {executor.submit(worker, url): url for url in urls}  
        for future in concurrent.futures.as_completed(futures):  
            data = future.result()  
    print(f"总耗时: {time.time() - start:.2f} 秒 (接近最慢的那个任务！)\n")  
   
    # 3. 思考：如果用多进程会怎样？  
    # 答：创建进程的开销远大于线程，对于这种纯IO任务，多进程的速度可能和多线程差不多，但消耗的资源（内存）更多，得不偿失。  
   
if __name__ == '__main__':  
    main()
```

### 2.2 最佳实践小结

* • **CPU密集用`multiprocessing`**：计算、科学模拟、图像处理。
* • **IO密集用`threading`或`asyncio`**：网络爬虫、文件批量处理、Web服务响应。

### 实战进阶：混合型任务怎么办？

现实中的任务往往是混合的。比如，一个Web服务器既要处理网络请求（IO），又要对请求的数据进行JSON解析和业务逻辑计算（CPU）。

**策略：分工合作**

核心思想是“IO归IO，CPU归CPU”。通常采用**生产者-消费者模型**：

1. 1. **IO线程/进程** 作为生产者，负责接收请求，读取数据，然后将需要计算的“任务”放入一个队列。
2. 2. **计算进程池** 作为消费者，专门从队列中取出任务进行计算，然后将结果放入另一个队列。
3. 3. **IO线程/进程** 再取出结果，返回给客户端。

这样可以避免计算任务阻塞IO循环，也能让CPU计算得到充分的并行化。`concurrent.futures`模块的`ProcessPoolExecutor`和`ThreadPoolExecutor`结合队列，是实现这种模式的利器。

## 五、避坑指南

1. 1. **不要盲目用多进程**：进程创建、销毁和通信（IPC）成本很高。如果任务很轻量，创建进程的时间可能都超过任务本身。
2. 2. **注意线程安全**：多线程访问共享数据（如全局变量、列表、字典）时，必须使用锁（`threading.Lock`）来防止数据竞争。
3. 3. **进程间通信要小心**：`multiprocessing`提供了`Queue`、`Pipe`、`Manager`等方式，但传递大量数据时效率是关键考量。
4. 4. **异步IO (`asyncio`) 是更现代的方案**：对于纯IO密集型、特别是需要高并发连接的服务（如爬虫、聊天服务器），`asyncio`比多线程更轻量、更高效。它用单线程实现并发，通过“事件循环”在IO等待时切换任务。

## 六、实战速查手册：你的任务属于哪一类？

理论说千遍，不如例子看一遍。下面这个清单，几乎涵盖了日常开发中大部分并发场景，帮你快速判断该用多进程还是多线程/异步。

| 场景类别 | 典型任务 | 为什么？ | 推荐方案 |
| --- | --- | --- | --- |
| **CPU密集型** | **数据科学与计算** ：大规模数值计算（NumPy/Pandas）、机器学习模型训练/推理、密码哈希/加密解密。 | 核心是数学运算，极度消耗CPU算力。 | **多进程 (multiprocessing)** |
|  | **媒体处理** ：图片/视频的编码解码、压缩/解压缩（如PIL处理图片，OpenCV处理视频）。 | 涉及大量像素/帧数据的算法处理。 | **多进程 (multiprocessing)** |
|  | **模拟与渲染** ：金融模型蒙特卡洛模拟、3D图形渲染、物理引擎计算。 | 需要进行海量、独立的重复计算。 | **多进程 (multiprocessing)** |
|  | **复杂业务逻辑** ：对大型数据集进行聚合、转换、筛选等CPU开销大的操作。 | 纯内存计算，CPU是瓶颈。 | **多进程 (multiprocessing)** |
| **IO密集型** | **网络请求** ：爬虫抓取网页、调用外部API、微服务间通信。 | 绝大部分时间在等待网络响应。 | **多线程 (threading) 或 异步 (asyncio)** |
|  | **文件操作** ：批量读写磁盘文件（特别是机械硬盘）、日志处理。 | 绝大部分时间在等待磁盘I/O。 | **多线程 (threading)** |
|  | **数据库操作** ：执行SQL查询，尤其是网络数据库（MySQL, PostgreSQL等）。 | 时间花在数据库执行和网络传输上。 | **多线程 (threading) 或 异步 (asyncio)** |
|  | **Web服务** ：Flask/Django等后端服务处理HTTP请求。 | 大部分请求时间在等待DB查询、外部API调用。 | **多线程/多进程 + 异步** (如`gunicorn`多worker) |
|  | **用户界面** ：桌面GUI应用，保持界面响应。 | 不能让一个耗时操作（如下载）卡住整个界面。 | **多线程 (threading)** |

**一个实用的“三步判断法”：**

1. 1. **看等待**：你的代码在执行时，CPU是在疯狂运转，还是大部分时间在“等”（等网络回包、等磁盘读写、等用户输入）？
2. 2. **问自己**：如果我把这台电脑的CPU换成快一倍的，这个任务时间能减半吗？如果能，大概率是**CPU密集**。如果几乎没变化，大概率是**IO密集**。
3. 3. **看工具**：任务中主要用的库是`requests`（网络）、`sqlalchemy`（数据库）、`open()`（文件）？还是`numpy`、`pandas`、`tensorflow`（计算）？

**混合型任务怎么办？**

这是高级主题，但策略很清晰：**分层处理，IO和CPU解耦**。

* • **案例**：一个数据分析服务，需要先从数据库（IO）拉取数据，然后用Pandas（CPU）做复杂分析，最后把结果写入文件（IO）。
* • **策略**：

1. 1. 用**多线程**高效地拉取数据。
2. 2. 将拉取到的原始数据放入一个**队列**。
3. 3. 用一个独立的**多进程池**从队列中取数据，进行CPU密集的分析计算。
4. 4. 将计算结果放入另一个队列，再由**IO线程**写入文件。

* • **核心思想**：不要让CPU等IO，也不要让IO被CPU计算阻塞。用队列（`multiprocessing.Queue`）作为“缓冲区”和“连接器”。

## 七、总结与思考

程序优化，识“体”为先。CPU密集和IO密集，是决定并发策略的基石。记住这个简单的口诀：**CPU不够，进程来凑；IO太慢，线程/异步来办**。

下次当你觉得程序慢时，别急着`import threading`。先停下来分析一下：你的程序，到底是在疯狂计算，还是在焦灼等待？找准“病根”，才能一剂见效。

---

**动手试一试：**

写一个脚本，遍历一个目录下的所有文本文件（IO操作），并统计每个文件中某个关键词出现的次数（CPU计算）。尝试分别用纯顺序、多线程、多进程来实现，并比较它们的效率。你会如何设计架构来最大化利用资源？欢迎在评论区分享你的代码和思路！

 

**关注我**，免费分享Python学习资料！探索的乐趣，在于分享和发现。别忘了在评论区留下你的想法和问题，我们下期见！

 

 

---