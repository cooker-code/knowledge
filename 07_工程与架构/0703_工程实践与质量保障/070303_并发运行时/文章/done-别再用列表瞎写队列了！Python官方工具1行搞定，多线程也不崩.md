> 已吸收至：[[07_工程与架构/0703_工程实践与质量保障/070303_并发运行时/070303_核心知识点/并发运行时资源边界准则|并发运行时资源边界准则]]、[[07_工程与架构/0703_工程实践与质量保障/070303_并发运行时/070303_知识地图|070303_并发运行时知识地图]]

---
title: 别再用列表瞎写队列了！Python官方工具1行搞定，多线程也不崩
author: python小甲鱼
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5ODkxMTA0NQ==&mid=2247484526&idx=1&sn=d710aecea76405de96358378d3a22a47&chksm=c17c64b9ef6662bfa434c6d96933f6b369dec35bf331c920c2cb5fc7033530fd58372cb9a3a6&mpshare=1&scene=24&srcid=1022fEMQVGO3q7NaUF8pV5Hs&sharer_shareinfo=4216fd44e24c3cac9b7b458a8018b91a&sharer_shareinfo_first=4216fd44e24c3cac9b7b458a8018b91a#rd
---

#

刚学Python时，我曾用列表做过“队列”——用`append()`加任务，`pop(0)`取任务，以为简单又省事。结果数据一多，程序直接卡成PPT，后来才知道：`pop(0)`要把列表里所有元素往前挪，数据越多越慢，简直是给自己挖坑！

其实Python早就给你备好“现成的队列工具”了——`queue`模块和`collections.deque`，不用自己写一行同步代码，线程安全还超快，小白复制代码就能用。今天用“食堂打饭”的例子，带你10分钟搞懂怎么选、怎么用！

## 先搞懂：队列到底是啥？用“食堂打饭”类比秒懂

队列的核心逻辑就一个：**先进先出（FIFO）**，像食堂打饭排队——先到的先打饭，后到的排后面。

| 队列操作 | 食堂场景 | Python里的对应操作 |
| --- | --- | --- |
| 入队（加任务） | 新同学排到队尾 | queue.put() / deque.append() |
| 出队（取任务） | 队头同学打完饭离开 | queue.get() / deque.popleft() |
| 判空（没任务） | 队伍没人了 | queue.empty() / len(deque)==0 |

简单说：队列就是“按顺序处理任务的容器”，关键是“不插队、不混乱”。

## 别踩坑！用列表做队列有多坑？

很多小白会用`list`做队列，比如`tasks = []; tasks.append("任务1"); tasks.pop(0)`，看似简单，实则藏着两个大问题：

### 1. 慢到离谱：数据越多越卡

`list.pop(0)`的本质是“删除第一个元素后，把后面所有元素往前挪一位”——比如列表有1万个元素，删第一个要挪9999个元素，相当于食堂排队时，前面人走了，所有人都要往前挪一步，多费时间？

而官方工具`deque.popleft()`和`queue.get()`，不用挪元素，直接“取队头”，像食堂取号机叫号，不管多少人，一秒就能找到下一个该打饭的人，效率天差地别！

### 2. 多线程必乱：数据丢一半

如果用列表在多线程里处理任务（比如3个线程同时取任务），会出现“两个线程拿到同一个任务”“任务没处理就丢了”的情况——就像食堂没管理员，大家乱插队，饭都打错了。

而`queue.Queue`天生“线程安全”，自带“管理员”（锁机制），多线程取任务也不会乱，这是列表永远做不到的！

## 官方推荐：两种队列工具，小白按需选

Python给小白准备了两个“队列神器”，不用记复杂原理，按场景选就行，比自己写省心10倍。

### 场景1：单线程用deque——轻量又超快

如果你的程序只有一个“任务处理线”（比如单线程处理日志、批量数据），用`collections.deque`就够了，轻量到像“揣在口袋里的小本子”，入队出队都是秒级。

#### 小白实战：单线程处理任务列表

```
# 1. 导入deque（Python自带，不用额外装）
from collections import deque
import time

# 2. 创建deque队列（相当于打开小本子准备记任务）
task_queue = deque()

# 3. 入队：加3个任务（相当于把任务记在小本子上）
task_queue.append("处理日志1")
task_queue.append("处理日志2")
task_queue.append("处理日志3")

# 4. 出队：按顺序处理任务（相当于按顺序做小本子上的任务）
print("开始处理任务：")
while task_queue:  # 只要队列不为空，就一直处理
    current_task = task_queue.popleft()  # 取队头任务（快！不挪元素）
    print(f"正在处理：{current_task}")
    time.sleep(1)  # 模拟处理耗时（实际场景替换成真实逻辑）

print("所有任务处理完！")
```

#### 运行效果：

```
开始处理任务：
正在处理：处理日志1
正在处理：处理日志2
正在处理：处理日志3
所有任务处理完！
```

#### 为啥好用？

* • 不用自己写判断空的逻辑，`while task_queue`直接判断；
* • `popleft()`比列表`pop(0)`快100倍，数据多了也不卡；
* • 自带`appendleft()`（队头加任务）、`clear()`（清空队列），功能够用。

### 场景2：多线程用queue.Queue——安全不崩

如果你的程序要“多个人同时干活”（比如3个线程同时处理订单，1个线程加订单），必须用`queue.Queue`——它自带“锁”，不会出现“两个线程抢一个任务”的情况，像食堂有管理员维持秩序，再忙也不乱。

#### 小白实战：多线程生产者消费者（订单处理系统）

比如“1个生产者加订单，3个消费者处理订单”，用`queue.Queue`几行代码就搞定，不用自己加锁：

```
# 1. 导入需要的库（queue是官方队列，Thread是多线程）
from queue import Queue
from threading import Thread
import time
import random

# 2. 生产者：往队列里加订单（相当于客服接订单）
defadd_orders(queue, total_orders):
    for i inrange(total_orders):
        order = f"订单{i+1}（买手机）"
        queue.put(order)  # 入队：加订单
        print(f"✅ 客服新增订单：{order}")
        time.sleep(random.uniform(0.5, 1.5))  # 模拟接订单的间隔

# 3. 消费者：从队列里取订单处理（相当于仓库发货）
defprocess_orders(queue, worker_name):
    whileTrue:
        # 出队：取订单，没订单时会阻塞（等订单，不用自己写循环等）
        order = queue.get()
        print(f"🔍 {worker_name} 开始处理：{order}")
        time.sleep(random.uniform(1, 2))  # 模拟处理订单耗时
        print(f"✅ {worker_name} 完成处理：{order}")
        queue.task_done()  # 告诉队列：这个任务我做完了

# 4. 主程序：启动生产者和消费者
if __name__ == "__main__":
    # 创建队列（最多存10个订单，满了会阻塞生产者，避免内存爆了）
    order_queue = Queue(maxsize=10)

    # 启动3个消费者线程（3个仓库工人）
    for i inrange(3):
        worker = Thread(
            target=process_orders,
            args=(order_queue, f"仓库工人{i+1}"),
            daemon=True# 设为后台线程：主程序结束，线程也结束
        )
        worker.start()
        print(f"👷 启动{worker.name}")

    # 启动生产者线程（1个客服）
    producer = Thread(
        target=add_orders,
        args=(order_queue, 5)  # 总共加5个订单
    )
    producer.start()
    producer.join()  # 等生产者加完所有订单

    # 等队列里所有订单都处理完，主程序再结束
    order_queue.join()
    print("\n🎉 所有订单处理完成！")
```

#### 运行效果（类似这样，顺序可能有差异）：

```
👷 启动Thread-1
👷 启动Thread-2
👷 启动Thread-3
✅ 客服新增订单：订单1（买手机）
🔍 仓库工人1 开始处理：订单1（买手机）
✅ 客服新增订单：订单2（买手机）
🔍 仓库工人2 开始处理：订单2（买手机）
✅ 客服新增订单：订单3（买手机）
🔍 仓库工人3 开始处理：订单3（买手机）
✅ 仓库工人1 完成处理：订单1（买手机）
✅ 客服新增订单：订单4（买手机）
🔍 仓库工人1 开始处理：订单4（买手机）
✅ 仓库工人2 完成处理：订单2（买手机）
✅ 客服新增订单：订单5（买手机）
🔍 仓库工人2 开始处理：订单5（买手机）
✅ 仓库工人3 完成处理：订单3（买手机）
✅ 仓库工人1 完成处理：订单4（买手机）
✅ 仓库工人2 完成处理：订单5（买手机）

🎉 所有订单处理完成！
```

#### 小白必懂：这3个关键点

1. 1. `daemon=True`：消费者线程设为“后台线程”，主程序结束时线程也跟着结束，不用手动关；
2. 2. `queue.join()`：等队列里所有任务都被`task_done()`标记后，主程序才结束，避免任务没处理完就退出；
3. 3. `maxsize=10`：队列满了会阻塞生产者（客服），避免订单加太多占满内存，超贴心！

## 小白总结：什么时候用哪个？一张表搞定

不用记复杂规则，对照这张表选，99%的场景都不会错：

| 场景 | 推荐工具 | 核心优势 | 代码示例（关键行） |
| --- | --- | --- | --- |
| 单线程（比如处理日志） | collections.deque | 轻量、快，不用多余配置 | `q = deque(); q.append(); q.popleft()` |
| 多线程（比如订单系统） | queue.Queue | 线程安全、支持阻塞、防内存爆 | `q = Queue(); q.put(); q.get(); q.task_done()` |
| 要按优先级处理任务 | queue.PriorityQueue | 按任务优先级取（比如紧急订单先处理） | `q.put((1, "紧急订单")); q.put((2, "普通订单"))` |

## 最后：别再自己造轮子了！

很多小白觉得“官方工具太重”，非要用列表写队列，结果要么卡成PPT，要么多线程数据乱。其实`deque`和`queue.Queue`都是Python自带的“轻量级神器”，不用装任何依赖，复制代码就能跑。

下次需要队列时，先问自己：是单线程还是多线程？按场景选对工具，比自己瞎写省时间，还不踩坑！