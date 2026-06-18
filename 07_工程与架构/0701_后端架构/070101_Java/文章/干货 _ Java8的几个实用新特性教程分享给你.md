---
title: 干货 | Java8的几个实用新特性教程分享给你
author: 儒猿技术架构
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU2Njg3OTU1Mg==&mid=2247486678&idx=1&sn=1b1785c0af086e53c4cb662cb4542257&chksm=fca4f8f9cbd371ef7402c34ff39721bd0e1bed49b7dca5eb83bd83f9fa00e18497ca71399ead&mpshare=1&scene=24&srcid=0818MOKnMavTdIjcVq87zicH&sharer_sharetime=1660781968965&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

文章来源：https://juejin.cn/post/7130883974846480391

**目录**

## 1.延迟队列

## 2.时间格式的天数

## 3. 印戳锁

## 4. 并行累加器

## 5. 十六进制格式

## 6. 数组的二分法检索

## 7. 位图

## 8. 移相器

在本文中，你可以了解一些可能没有听说过的有用的 Java 特性。这是我最近使用的功能的私人列表，或者是我在阅读有关 Java 的文章时偶然发现的。我不会关注语言方面，而是关注 API。

## 

> > **1.延迟队列**
> >
> > **【Delay Queue】**

## 

如您所知，Java 中有许多类型的集合可用。但你听说了`DelayQueue`吗？它是一种特定类型的 Java 集合，它允许我们根据元素的延迟时间对元素进行排序。老实说，这是一门非常有趣的课。尽管 DelayQueue该类是 Java 集合的成员，但它属于 `java.util.concurrent` 包。它实现了`BlockingQueue`接口。只有当元素的时间到期时，才能从队列中取出元素。

为了使用它，首先在类里面需要实现接口中的`getDelay`方法`Delayed`。它不必是一个类——你也可以使用 Java Record。

```
public record DelayedEvent(long startTime, String msg) implements Delayed {  
  
    public long getDelay(TimeUnit unit) {  
        long diff = startTime - System.currentTimeMillis();  
        return unit.convert(diff, TimeUnit.MILLISECONDS);  
    }  
  
    public int compareTo(Delayed o) {  
        return (int) (this.startTime - ((DelayedEvent) o).startTime);  
    }  
  
}
```

假设我们想推迟元素10秒。我们只需要设置当前时间增加了10秒DelayedEvent类。

```
final DelayQueue<DelayedEvent> delayQueue = new DelayQueue<>();  
final long timeFirst = System.currentTimeMillis() + 10000;  
delayQueue.offer(new DelayedEvent(timeFirst, "1"));  
log.info("Done");  
log.info(delayQueue.take().msg());
```

上面可见代码的输出是什么？让我们来看看。

> > **2.时间格式的天数**
> >
> > **【Period of Days in Time Format】**

Java 8 改进了很多时间处理 API。从这个版本的 Java 开始，在大多数情况下，可能不必使用任何额外的库，如 Joda Time。你能想象从 Java 16 开始，你甚至可以使用标准格式化程序来表示一天中的时段，例如“早上”或“下午”？有一种称为 B 的新格式模式。

```
String s = DateTimeFormatter  
  .ofPattern("B")  
  .format(LocalDateTime.now());  
System.out.println(s);
```

这是我的结果。当然结果取决于在一天中的时间。

## 

> > **3. 印戳锁**
> >
> > **【StampedLock】**

Java Concurrent 是最有趣的 Java 包之一。同时，它也是开发人员鲜为人知的一种，尤其是当他们主要使用 Web 框架时。有多少人曾经在 Java 中使用过锁？锁定是一种比“同步”块更灵活的线程同步机制。从 Java 8 开始，您可以使用一种称为“StampedLock”的新型锁。StampedLock 是使用 `ReadWriteLock` 的替代方法。它允许对读取操作进行乐观锁定。此外，它比 ReentrantReadWriteLock 具有更好的性能。

假设我们有两个线程。其中第一个更新余额，而第二个读取余额的当前值。为了更新余额，我们当然需要先读取它的当前值。我们在这里需要某种同步，假设第一个线程同时运行多次。第二个线程只是说明了如何使用乐观锁进行读取操作。

```
StampedLock lock = new StampedLock();  
Balance b = new Balance(10000);  
Runnable w = () -> {  
   long stamp = lock.writeLock();  
   b.setAmount(b.getAmount() + 1000);  
   System.out.println("Write: " + b.getAmount());  
   lock.unlockWrite(stamp);  
};  
Runnable r = () -> {  
   long stamp = lock.tryOptimisticRead();  
   if (!lock.validate(stamp)) {  
      stamp = lock.readLock();  
      try {  
         System.out.println("Read: " + b.getAmount());  
      } finally {  
         lock.unlockRead(stamp);  
      }  
   } else {  
      System.out.println("Optimistic read fails");  
   }  
};
```

现在，让我们通过同时运行两个线程 50 次来测试它。它应该按预期工作 - 余额的最终值为`60000`。

```
ExecutorService executor = Executors.newFixedThreadPool(10);  
for (int i = 0; i < 50; i++) {  
   executor.submit(w);  
   executor.submit(r);  
}
```

> > **4. 并行累加器**
> >
> > **【Concurrent accumulators】**

## 

锁并不是 Java Concurrent 包中唯一有趣的特性。另一种称为并发累加器。还有并发加法器，但它是一个非常相似的功能。`LongAccumulator`（也有`DoubleAccumulator`）使用提供的函数更新值。它允许我们在许多场景中实现无锁算法。当多个线程更新一个公共值时，通常最好使用 `AtomicLong`。

让我们看看它是如何工作的。为了创建它，您需要在构造函数中设置两个参数。第一个是用于计算累加结果的函数。通常，您会使用`sum` 方法。第二个参数表示我们的累加器的初始值。

现在，让我们使用初始值`10000`创建`LongAccumulator`，然后从多个线程调用`accumulate()`方法。最终结果是什么？

```
LongAccumulator balance = new LongAccumulator(Long::sum, 10000L);  
Runnable w = () -> balance.accumulate(1000L);  
  
ExecutorService executor = Executors.newFixedThreadPool(50);  
for (int i = 0; i < 50; i++) {  
   executor.submit(w);  
}  
  
executor.shutdown();  
if (executor.awaitTermination(1000L, TimeUnit.MILLISECONDS))  
   System.out.println("Balance: " + balance.get());  
assert balance.get() == 60000L;
```

> > **5. 十六进制格式**
> >
> > **【Hex Format】**

## 

有时我们需要在十六进制格式的字符串、字节或字符之间进行转换。从 Java 17 开始，您可以使用 `HexFormat` 类。只需创建一个 HexFormat 实例，然后您就可以格式化例如输入 `byte` 到十六进制字符串的表。你也可以例如将输入的十六进制字符串解析为字节表，如下所示。

```
HexFormat format = HexFormat.of();  
  
byte[] input = new byte[] {127, 0, -50, 105};  
String hex = format.formatHex(input);  
System.out.println(hex);  
  
byte[] output = format.parseHex(hex);  
assert Arrays.compare(input, output) == 0;
```

> > **6. 数组的二分法检索**
> >
> > **【Binary Search for Arrays】**

## 

假设我们想在排序表中插入一个新元素。Arrays.binarySearch() 返回搜索键的索引（如果包含）。否则，它返回一个**i**插入点，我们可以使用它来计算新键的索引：`-(insertion point)-1`。此外，binarySearch 方法是在 Java 中查找已排序数组中元素的最简单、最有效的方法。

示例如下，我们有一个输入表，其中四个元素按升序排列。我们想在此表中插入数字“3”。这是我们如何计算插入点的索引的方法。

```
int[] t = new int[] {1, 2, 4, 5};  
int x = Arrays.binarySearch(t, 3);  
  
assert ~x == 2;
```

> > **7. 位图**
> >
> > **【Bit Set】**

## 

如果我们需要对位数组执行一些操作怎么办？你会为此使用``boolean[]`？？有一种更有效和内存效率更高的方法来实现它。它是 BitSet 类。BitSet 类允许我们存储和操作位数组。与` boolean[] `相比，它消耗的内存少了 8 倍。我们可以对数组执行逻辑操作，例如`and`,` or`,` xor`。

假设我们有两个输入位数组。我们想对它们执行xor 操作。事实上，只返回那些元素的操作，这些元素只插入一个数组，而不是另一个数组。为此，我们需要创建两个“BitSet”实例并在其中插入示例元素，如下所示。最后应该在 `BitSet` 实例之一上调用 xor 方法。它将第二个 BitSet实例作为参数。

```
BitSet bs1 = new BitSet();  
bs1.set(0);  
bs1.set(2);  
bs1.set(4);  
System.out.println("bs1 : " + bs1);  
  
BitSet bs2 = new BitSet();  
bs2.set(1);  
bs2.set(2);  
bs2.set(3);  
System.out.println("bs2 : " + bs2);  
  
bs2.xor(bs1);  
System.out.println("xor: " + bs2);
```

这是运行上面可见的代码后的结果。

## 

> > **8. 移相器**
> >
> > **【Phaser】**

## 

和这里的其他一些例子一样，它也是 Java Concurrent 包的元素。它被称为“移相器”。它与更知名的 `CountDownLatch` 非常相似。但是，它提供了一些附加功能。它允许我们设置在继续执行之前需要等待的动态线程数。使用“Phaser”，定义的线程数需要在屏障上等待，然后才能进入下一步执行。多亏了这一点，我们可以协调执行的多个阶段。

在下面的示例中，我们设置了 50 个线程的屏障，以便在进入下一个执行阶段之前到达。然后我们创建一个线程，在Phaser 实例上调用方法arriveAndAwaitAdvance()。它阻塞线程，直到所有 50 个线程都不会到达屏障。然后它进入 `phase-1` 并调用方法 `arriveAndAwaitAdvance()`。

```
Phaser phaser = new Phaser(50);  
Runnable r = () -> {  
   System.out.println("phase-0");  
   phaser.arriveAndAwaitAdvance();  
   System.out.println("phase-1");  
   phaser.arriveAndAwaitAdvance();  
   System.out.println("phase-2");  
   phaser.arriveAndDeregister();  
};  
  
ExecutorService executor = Executors.newFixedThreadPool(50);  
for (int i = 0; i < 50; i++) {  
   executor.submit(r);  
}
```

这是执行上面可见的代码后的结果。

欢迎扫码加入儒猿技术交流群，每天晚上20:00都有Java面试、Redis、MySQL、RocketMQ、SpringCloudAlibaba、Java架构等技术答疑分享，更能跟小伙伴们一起交流技术

**另外推荐儒猿课堂的1元系列课程给您，欢迎加入一起学习~**

**互联网Java工程师面试突击课**

**（1元专享）**

**SpringCloudAlibaba零基础入门到项目实战**

**（1元专享）**

**亿级流量下的电商详情页系统实战项目**

**（1元专享）**

**Kafka消息中间件内核源码精讲**

**（1元专享）**

**12个实战案例带你玩转Java并发编程**

**（1元专享）**

**Elasticsearch零基础入门到精通**

**（1元专享）**

**基于Java手写分布式中间件系统实战**

**（1元专享）**

**基于ShardingSphere的分库分表实战课**

**（1元专享）**