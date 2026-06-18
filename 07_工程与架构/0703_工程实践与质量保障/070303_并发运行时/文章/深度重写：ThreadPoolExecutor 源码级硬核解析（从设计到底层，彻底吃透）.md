---
title: 深度重写：ThreadPoolExecutor 源码级硬核解析（从设计到底层，彻底吃透）
author: 狂热大猩猩的技术宅
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNDY5MjY1Ng==&mid=2247483931&idx=1&sn=ae7870dc38dedbee5e4e63a7e4afa944&chksm=c3347a369b161b50a030a605792a14b819327c101468696a329b4222f62d0b9528ca3be7fd20&mpshare=1&scene=24&srcid=04155AKiMFhEHfrU4XdYQT4k&sharer_shareinfo=811fe05677671da1ba2d594e4febfee2&sharer_shareinfo_first=811fe05677671da1ba2d594e4febfee2#rd
---

## 前言

市面上 90% 的线程池文章只讲**执行流程**，但 ThreadPoolExecutor 的核心魅力在于：**用极致精巧的设计，在高并发下保证线程安全、线程复用、资源管控**。本文从**变量设计 → 核心方法 → 并发安全 → 底层原理 → 生产坑点**全链路解析，不背八股，只讲源码。

一、灵魂变量：ctl 原子整数（最精妙的设计）

```
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

**这一个变量，同时存了两个核心信息：**

* **高 3 位**

  线程池运行状态（RUNNING / SHUTDOWN / STOP / TIDYING / TERMINATED）
* **低 29 位**

  工作线程数量（最大支持 2^29-1 个线程，足够生产使用）

### 为什么这么设计？

* **原子性**

  用一个 CAS 操作同时修改状态和线程数，避免加锁，提升并发性能；
* **节省内存**

  无需两个独立变量，减少内存占用与并发竞争；
* **一致性**

  状态和线程数强绑定，保证二者修改的原子性，不会出现中间状态。

### 5 种线程池状态（必须理解流转）

* **RUNNING**

  接收新任务 + 处理队列任务
* **SHUTDOWN**

  不接收新任务，但处理队列剩余任务
* **STOP**

  不接收新任务，不处理队列任务，中断正在执行的任务
* **TIDYING**

  所有任务终止，线程数清零，准备执行 terminated ()
* **TERMINATED**

  terminated () 执行完成，线程池彻底死亡

**状态流转**：RUNNING → SHUTDOWN / STOP → TIDYING → TERMINATED

二、核心方法：execute () 源码逐行解析（并发安全全靠它）

这是线程池的入口，**无锁 + 双重检查 + CAS** 保证高并发安全：

```
public void execute(Runnable command) {    if (command == null) throw new NullPointerException();    // 1. 获取ctl（状态+线程数）    int c = ctl.get();    // 2. 工作线程数 < 核心线程数：直接创建核心线程执行    if (workerCountOf(c) < corePoolSize) {        if (addWorker(command, true)) return;        // 创建失败（并发/状态变更），重新获取ctl        c = ctl.get();    }    // 3. 线程池运行中 && 队列未满：将任务加入队列    if (isRunning(c) && workQueue.offer(command)) {        // 双重检查：防止加入队列后线程池被关闭        int recheck = ctl.get();        // 线程池已关闭 → 移除任务并执行拒绝策略        if (!isRunning(recheck) && remove(command)) reject(command);        // 无工作线程：创建一个非核心线程（兜底，保证队列任务能被执行）        else if (workerCountOf(recheck) == 0) addWorker(null, false);    }    // 4. 队列已满：创建非核心线程执行    else if (!addWorker(command, false)) {        // 5. 线程数达到最大：拒绝策略        reject(command);    }}
```

### 核心考点 / 原理（深度）

* **为什么要双重检查线程池状态？**

  高并发下，任务加入队列后，线程池可能被立即关闭，双重检查避免**已关闭的线程池还执行新任务**。
* **为什么队列满了才创建非核心线程？**

  设计初衷：**核心线程优先 + 队列缓冲**，减少线程创建销毁的开销，符合线程池设计思想。
* **无锁设计**

  全程用 CAS + 状态判断，未加重量级锁，并发性能拉满。

三、线程复用的核心：Worker + runWorker ()

这是线程池**不销毁线程、反复执行任务**的底层原理，90% 的文章只说 “死循环”，完全没讲透。

## 1. Worker 类：自带 AQS 的工作线程

```
private final class Worker extends AbstractQueuedSynchronizer implements Runnable
```

### Worker 的 3 个核心作用：

* 封装工作线程，持有一个 Thread 对象；
* **继承 AQS**

  实现**不可重入锁**，标记线程是否正在执行任务；
* 绑定第一个任务，执行后循环获取队列任务。

### 为什么 Worker 要用 AQS？

* **防止任务执行中被中断**

  只有线程空闲时（未加锁），才能被中断；
* **轻量锁**

  不用 ReentrantLock，AQS 更轻量，无额外开销；
* **不可重入**

  保证中断操作不会在任务执行时触发。

## 2. runWorker ()：线程复用的终极逻辑

```
final void runWorker(Worker w) {    Thread wt = Thread.currentThread();    Runnable task = w.firstTask;    w.firstTask = null;    // 允许中断    w.unlock();    // 死循环：不断从队列取任务执行 → 这就是线程复用！    while (task != null || (task = getTask()) != null) {        // 加锁：标记线程正在执行任务，禁止中断        w.lock();        try {            // 执行任务前钩子            beforeExecute(wt, task);            // 执行用户任务            task.run();            // 执行后钩子            afterExecute(task, null);        } finally {            // 清空任务，解锁，允许下一次中断            task = null;            w.unlock();        }    }    // 跳出循环：线程空闲超时/线程池关闭 → 销毁线程    processWorkerExit(w, completedAbruptly);}
```

### 线程复用的本质：

**工作线程启动后，不会退出，而是在 while 循环里阻塞等待队列中的任务**，拿到就执行，没拿到就阻塞，直到超时或线程池关闭。

四、线程存活核心：getTask () 超时机制

```
private Runnable getTask() {    // 标记是否超时    boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;    // 循环获取队列任务    for (;;) {        // 超时：返回null，线程退出        if (timed && timedOut) return null;        // 阻塞/超时获取任务        Runnable r = timed ?            workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :            workQueue.take();        if (r != null) return r;        timedOut = true;    }}
```

### 核心原理：

* **核心线程默认不会退出**

  `timed=false`，调用 `take()` 无限阻塞，一直等任务；
* **非核心线程 / 开启核心线程超时**

  调用 `poll()`，等待 `keepAliveTime` 后无任务则**线程退出**，释放资源。

五、生产级重点：线程池的 4 个 “致命坑”

## 1. FixedThreadPool OOM 风险

* 队列：`new LinkedBlockingQueue<>(Integer.MAX_VALUE)`
* 风险：任务无限堆积，撑爆堆内存 → 生产必须指定队列容量。

## 2. 线程池中断陷阱

* `shutdown()`

  温柔关闭，等任务执行完；
* `shutdownNow()`

  暴力关闭，中断线程，清空队列；
* **禁止在业务代码里手动中断线程池线程，会导致 Worker 锁异常。**

## 3. 线程数公式误区

网上公式：CPU 密集 = core+1，IO 密集 = core×2

**正确做法**：

* CPU 密集：core = CPU 核心数（减少上下文切换）；
* IO 密集：core = 核心数 × (1 + 平均 IO 等待时间 / 平均 CPU 执行时间)；
* 最终以**压测结果**为准，公式仅参考。

## 4. 拒绝策略选择

* `AbortPolicy`

  默认，直接抛异常（生产常用，便于告警）；
* `CallerRunsPolicy`

  调用者执行（适合不能丢任务的场景）；
* `DiscardPolicy`

  直接丢弃（禁止生产使用）。

# 六、总结：ThreadPoolExecutor 核心设计思想

* **一个变量管全局**

  ctl 原子存储状态 + 线程数，无锁高效；
* **无锁并发**

  execute 全程 CAS + 双重检查，无重量级锁；
* **线程复用**

  Worker 死循环阻塞取任务，避免线程频繁创建销毁；
* **AQS 保障安全**

  防止任务执行中被中断，保证业务稳定性；
* **弹性管控**

  核心线程常驻，非核心线程超时销毁，平衡性能与资源。

### 总结

### 线程池的执行流程：任务进来 → 先开核心线程 → 塞满队列 → 再开最大线程 → 都满了就拒绝。

当调用 **threadPool.execute(task)** 时，线程池会**严格按下面 5 步执行**：

1. ### 第 1 步：判断核心线程数

   如果 **当前运行的线程 < 核心线程数（corePoolSize）**→ **直接创建一个新的核心线程**执行任务。（即使其他核心线程空闲，也会优先新建）
2. 第 2 步：核心线程满了 → 放入阻塞队列

   如果 **当前线程数 == 核心线程数**→ **把任务放入阻塞队列（workQueue）**。

   线程池此时**不会创建新线程**，而是等核心线程空闲后来队列取任务。
3. ### 第 3 步：队列也满了 → 创建最大线程

   如果 **阻塞队列也满了**→ **创建新的非核心线程执行任务**，直到线程数达到 **最大线程数（maximumPoolSize）**。
4. ### 第 4 步：最大线程也满了 → 执行拒绝策略

   如果 **线程数达到最大线程数 + 队列也满了**→ **触发拒绝策略（RejectedExecutionHandler）**。

### 执行过程中的重要细节

* **核心线程会一直存活（默认不会超时）**
* **非核心线程空闲超过一定时间会被回收（keepAliveTime）**
* **队列是先塞满，才会创建新线程（这是很多人搞错的点）**
* **只要线程数 ≤ 核心数，永远优先新建线程，而不是复用**
* **线程执行完任务不会销毁，会回到循环里阻塞等待新任务→ 这就是线程复用**

这篇文章是**真正的源码深度解析**，没有废话、全是底层原理，既适合面试，也能直接用于生产调优。

学习是一场修行，让我们砥砺前行，于喧嚣中守心，于沉淀中精进，以微光汇星河，以坚持赴远方，不负时光，不负成长。