---
title: Flink 中的 EventTimeTrigger 和 ProcessingTimeTrigger 详解
author: zhisheng
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIxMTE0ODU5NQ==&mid=2650248742&idx=1&sn=6c09219c8a18e4989812d568a91200b7&chksm=8f5afc7ab82d756ca5a694783f7aa7df231af2a5f1800727b06cbfacc0e780026ab08f1cb8c1&mpshare=1&scene=24&srcid=1103TdUv3RwyG8PoqZc2cuwr&sharer_sharetime=1667437961489&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

## EventTimeTrigger

EventTimeTrigger 的触发完全依赖 watermark，换言之，**如果 stream 中没有 watermark，就不会触发 EventTimeTrigger。**

watermark 之于事件时间就是如此重要，来看一下 watermark 的定义先~

Watermarks 是某个 event time 窗口中所有数据都到齐的标志。  
Watermarks 作为数据流的一部分流动并携带时间戳 t，Watermark(t) 断言数据流中不会再有小于时间戳 t 的事件出现。  
换言之，**当 Watermark(t) 到达时，标志着所有小于时间戳 t 的事件都已到齐**，可放心地对时间戳 t 之前的所有事件执行聚合、窗口关闭等动作。

来看一下 《Streaming Systems》书中关于 Watermark 原汁原味的定义：

> Watermarks are temporal notions of input completeness in the event-time domain. Worded differently, they are the way the system measures progress and completeness relative to the event times of the records being processed in a stream of events.

> That point in event time, E, is the point up to which the system believes all inputs with event times less than E have been observed. In other words, **it’s an assertion that no more data with event times less than E will ever be seen again.**

如下图所示，watermark 和 event 一样在 pipeline 中流动，并且都携带时间戳，W(4) 表示在此之后不会再收到事件时间小于 4 的事件，W(9) 表示在此之后不会再接收到事件时间小于 9 的事件。

watermark 示意图

假设我们准备对这个数据流做窗口聚合操作，时间窗口大小为 4 个时间单位，窗口内元素做求和聚合操作。示例代码如下：

```
input.window(TumblingEventTimeWindows.of(Time.minutes(4L)))  
     .trigger(EventTimeTrigger.create())  
     .reduce(new SumReduceFunction());
```

我们跟踪 Flink 源码看看 EventTimeTrigger 的实现逻辑：

```
/**  
 * A Trigger that fires once the watermark passes the end of the window to which a pane belongs.  
 */  
public class EventTimeTrigger extends Trigger<Object, TimeWindow> {  
    private static final long serialVersionUID = 1L;  
  
    private EventTimeTrigger() {}  
  
    @Override  
    public TriggerResult onElement(  
            Object element, long timestamp, TimeWindow window, TriggerContext ctx)  
            throws Exception {  
        if (window.maxTimestamp() <= ctx.getCurrentWatermark()) {  
            // if the watermark is already past the window fire immediately  
            return TriggerResult.FIRE;  
        } else {  
            ctx.registerEventTimeTimer(window.maxTimestamp());  
            return TriggerResult.CONTINUE;  
        }  
    }  
  
    @Override  
    public TriggerResult onEventTime(long time, TimeWindow window, TriggerContext ctx) {  
        return time == window.maxTimestamp() ? TriggerResult.FIRE : TriggerResult.CONTINUE;  
    }  
  
    public static EventTimeTrigger create() {  
        return new EventTimeTrigger();  
    }  
}
```

**onElement** 方法在每次数据进入该 window 时都会触发：首先判断当前的 watermark 是否已经超过了 window 的最大时间（窗口右界时间），如果已经超过，返回触发结果 **FIRE**；如果尚未超过，则根据窗口右界时间注册一个事件时间定时器，标记触发结果为 **CONTINUE**。

继续跟踪 registerEventTimeTimer 的动作，发现原来是将定时器放到了一个优先队列 eventTimeTimersQueue 中。优先队列的权重为定时器的时间戳，时间越小越希望先被触发，自然排在队列的前面。

**onEventTime** 方法是在什么时候被调用呢，一路跟踪来到了 InternalTimerServiceImpl 类，发现只有在 advanceWatermark 的时候会触发 onEventTime，具体逻辑是：当 watermark 到来时，根据 watermark 携带的时间戳 t，从事件时间定时器队列中出队所有时间戳小于 t 的定时器，然后触发 onEventTime。onEventTime 方法的具体实现中，首先比较触发时间是否是自己当前窗口的结束时间，是则 **FIRE**，否则继续 **CONTINUE**。

```
public class InternalTimerServiceImpl<K, N> implements InternalTimerService<N> {  
  
    /** Event time timers that are currently in-flight. */  
    private final KeyGroupedInternalPriorityQueue<TimerHeapInternalTimer<K, N>>  
            eventTimeTimersQueue;  
  
    @Override  
    public void registerEventTimeTimer(N namespace, long time) {  
        eventTimeTimersQueue.add(  
                new TimerHeapInternalTimer<>(time, (K) keyContext.getCurrentKey(), namespace));  
    }  
  
    public void advanceWatermark(long time) throws Exception {  
        currentWatermark = time;  
        InternalTimer<K, N> timer;  
        while ((timer = eventTimeTimersQueue.peek()) != null && timer.getTimestamp() <= time) {  
            eventTimeTimersQueue.poll();  
            keyContext.setCurrentKey(timer.getKey());  
            triggerTarget.onEventTime(timer);  
        }  
    }  
}
```

我们以上述示例数据为例，看一下具体的执行过程：

* 当事件到达时，根据事件时间将事件分配到相应的窗口中，事件 '2' 被分配到时间窗口 T1-T4 中，如下图所示：

* 后续时间陆续到达，事件 '2'，'3'，'1'，'3' 都陆续被分配到时间窗口 T1-T4 中，由于 currentWatermark 未超过窗口结束时间 4 ，因此注册事件时间定时器 Timer(4)；事件 '7' 被分配到窗口 T5-T8 中，也因为 currentWatermark 未超过窗口结束时间 8，因此注册事件时间定时器 Timer(8)。目前为止事件时间定时器队列中有 2 个定时器。

* 重点来了，接下来到达的是 watermark(4)，那么就去事件时间定时器队列中找到所有定时时间小于等于 4 的定时器，Timer(4) 出队，触发 onEventTime 方法，窗口 T1-T4 的结束时间和 Timer(4) 的时间戳相等，FIRE 窗口 T1-T4（图中标记为绿色表示 FIRE）。此时定时器队列中只剩下 1 个定时器。

* 重复同样过程，事件 '5'，'9'，'6' 接踵而至，'5', '6' 被分配到窗口 T5-T8 中，'9' 则被分配到新窗口 T9-T12 中。'5'，'6' 对应的注册定时器为 Timer(8)，'9' 注册了一个新定时器 Timer(12)。此时定时器队列中剩下 2 个定时器。

watermark(9) 的到达会促使定时器 Timer(8) 出队，进而 FIRE 窗口 T5-T8。以此类推。

## ProcessingTimeTrigger

与 EventTimeTrigger 相比，ProcessingTimeTrigger 就相对简单了，它的触发只依赖系统时间。我们跟踪 Flink 源码看看 ProcessingTimeTrigger 的实现逻辑：

```
/**  
 * A Trigger that fires once the current system time passes the end of the window to which a pane belongs.  
 */  
public class ProcessingTimeTrigger extends Trigger<Object, TimeWindow> {  
    private static final long serialVersionUID = 1L;  
  
    private ProcessingTimeTrigger() {}  
  
    @Override  
    public TriggerResult onElement(  
            Object element, long timestamp, TimeWindow window, TriggerContext ctx) {  
        ctx.registerProcessingTimeTimer(window.maxTimestamp());  
        return TriggerResult.CONTINUE;  
    }  
  
    @Override  
    public TriggerResult onProcessingTime(long time, TimeWindow window, TriggerContext ctx) {  
        return TriggerResult.FIRE;  
    }  
  
    /** Creates a new trigger that fires once system time passes the end of the window. */  
    public static ProcessingTimeTrigger create() {  
        return new ProcessingTimeTrigger();  
    }  
}
```

**onElement** 方法在每次数据进入该 window 时都会触发：直接根据窗口结束时间注册一个处理时间定时器。同样是放到一个处理时间定时器优先队列中。

系统根据系统时间定时地从处理时间定时器队列中取出小于当前系统时间的定时器，然后调用 **onProcessingTime** 方法，FIRE 窗口。

```
public class InternalTimerServiceImpl<K, N> implements InternalTimerService<N> {  
  
    /** Processing time timers that are currently in-flight. */  
    private final KeyGroupedInternalPriorityQueue<TimerHeapInternalTimer<K, N>>  
  
    @Override  
    public void registerProcessingTimeTimer(N namespace, long time) {  
        InternalTimer<K, N> oldHead = processingTimeTimersQueue.peek();  
        if (processingTimeTimersQueue.add(  
                new TimerHeapInternalTimer<>(time, (K) keyContext.getCurrentKey(), namespace))) {  
            long nextTriggerTime = oldHead != null ? oldHead.getTimestamp() : Long.MAX_VALUE;  
            // check if we need to re-schedule our timer to earlier  
            if (time < nextTriggerTime) {  
                if (nextTimer != null) {  
                    nextTimer.cancel(false);  
                }  
                nextTimer = processingTimeService.registerTimer(time, this::onProcessingTime);  
            }  
        }  
    }  
  
    private void onProcessingTime(long time) throws Exception {  
        // null out the timer in case the Triggerable calls registerProcessingTimeTimer()  
        // inside the callback.  
        nextTimer = null;  
        InternalTimer<K, N> timer;  
        while ((timer = processingTimeTimersQueue.peek()) != null && timer.getTimestamp() <= time) {  
            processingTimeTimersQueue.poll();  
            keyContext.setCurrentKey(timer.getKey());  
            triggerTarget.onProcessingTime(timer);  
        }  
  
        if (timer != null && nextTimer == null) {  
            nextTimer =  
                    processingTimeService.registerTimer(  
                            timer.getTimestamp(), this::onProcessingTime);  
        }  
    }  
  
}
```