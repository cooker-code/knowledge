---
title: Function + 异常策略链：从“用函数”到“打造函数基础设施”
author: Java程序员成神之路
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNDgxMDI5NQ==&mid=2247485831&idx=1&sn=b2c8ed04ad524cfb2745a36edba1d102&chksm=c3ec4fd4cf41a3ca41869f1e9ac28012168912afb24ab3ff569827b97e5d7c1e47fcd397301f&mpshare=1&scene=24&srcid=0911zalN7MI4d8zS3dUOVess&sharer_shareinfo=0d1a43f49b8ae114a2b0e1df8703c6bd&sharer_shareinfo_first=0d1a43f49b8ae114a2b0e1df8703c6bd#rd
---

> 字数 1196，阅读大约需 6 分钟

今天，我们聚焦一个非常实战、但鲜有人深入系统化处理的场景：

> **如何用 `Function` 构建一个可复用、可组合、支持异常处理的“策略链”工具类**

教你怎么用 lambda，教你**怎么设计一套实用的基础设施**，让函数式逻辑在你的项目里真正可控、可维护、可调试。

### 背景：

在真实业务开发中，我们经常会遇到以下场景：

* • 某段逻辑包含多个转换/处理步骤。
* • 每个步骤都有可能失败或抛出异常。
* • 某些步骤希望失败不中断，继续执行；而另一些希望直接终止。
* • 异常日志和监控希望可控、可感知。

这时候你需要的不仅是 `Function`，而是一个支持 **组合链、异常处理策略、统一日志输出、默认回退机制** 的“函数链工具”。

---

## 一、问题示例：从“看似优雅”的链式调用说起

你可能见过这种写法：

```
Function<String, String> pipeline = trim  
    .andThen(upper)  
    .andThen(validate)  
    .andThen(saveToDb); // 假设这也包装成 Function
```

看起来优雅，但只要 **任何一步抛出异常**，后面的步骤就不会执行。更可怕的是：

* • 异常被吞没或输出不明确。
* • 想对不同函数设置不同的异常策略根本做不到。
* • 不具备组合重用性，只能从头再写一条链。

于是，链式调用变成了“玻璃架构” —— 漂亮但脆弱。

---

## 二、构建：一个真正实用的 Function 异常策略链工具

我们来构建一个类：`FunctionChain<T, R>`

功能目标：

1. 1. 每个步骤是一个 Function，可组合。
2. 2. 每个 Function 允许配置异常处理策略（跳过、终止、fallback）。
3. 3. 整个链条有清晰的日志输出。
4. 4. 支持默认值 / fallback 机制。

---

## 三、核心设计思路（先看结构，不急写代码）

```
FunctionChain<String, Result> chain = FunctionChain  
    .<String, String>start()  
    .then(trim, onErrorContinue())  
    .then(upper, onErrorBreak())  
    .then(validate, onErrorFallback("DEFAULT"))  
    .then(parseToResult, onErrorThrow());  
  
Result result = chain.execute("   hello  ");
```

这就是我们要实现的目标语义：

* • 每个 `.then()` 都是添加一个处理节点。
* • 每个节点可以定义自己的异常策略。
* • `execute()` 时自动按顺序调用，并应用策略。
* • 比起裸 `Function`，它具备组合性、调试性、策略性。

---

## 四、完整代码实现（基础版）

```
@Slf4j  
public class FunctionChain<T, R> {  
  
    private final List<ChainNode<?, ?>> chain = new ArrayList<>();  
  
    private FunctionChain() {}  
  
    public static <T, R> FunctionChain<T, R> start() {  
        return new FunctionChain<>();  
    }  
  
    public <V> FunctionChain<T, R> then(Function<V, ?> function, ErrorStrategy<V> strategy) {  
        chain.add(new ChainNode<>(function, strategy));  
        return (FunctionChain<T, R>) this;  
    }  
  
    public R execute(T input) {  
        Objectcurrent= input;  
        for (ChainNode<?, ?> node : chain) {  
            current = node.apply(current);  
        }  
        return (R) current;  
    }  
  
    private static class ChainNode<V, U> {  
        private final Function<V, U> function;  
        private final ErrorStrategy<V> strategy;  
  
        public ChainNode(Function<V, U> function, ErrorStrategy<V> strategy) {  
            this.function = function;  
            this.strategy = strategy;  
        }  
  
        public Object apply(Object input) {  
            try {  
                return function.apply((V) input);  
            } catch (Exception e) {  
                return strategy.handle((V) input, e);  
            }  
        }  
    }  
  
    public interface ErrorStrategy<V> {  
        Object handle(V input, Exception e);  
    }  
  
    // 常用策略封装  
    public static <V> ErrorStrategy<V> onErrorContinue() {  
        return (input, e) -> {  
            log.warn("Step failed, skipping. Input: {}", input, e);  
            return input;  
        };  
    }  
  
    public static <V> ErrorStrategy<V> onErrorBreak() {  
        return (input, e) -> {  
            log.error("Step failed, terminating. Input: {}", input, e);  
            throw new RuntimeException(e);  
        };  
    }  
  
    public static <V> ErrorStrategy<V> onErrorFallback(Object fallback) {  
        return (input, e) -> {  
            log.warn("Step failed, using fallback. Input: {}, Fallback: {}", input, fallback, e);  
            return fallback;  
        };  
    }  
  
    public static <V> ErrorStrategy<V> onErrorThrow() {  
        return (input, e) -> {  
            throw new RuntimeException("Unhandled exception in FunctionChain", e);  
        };  
    }  
}
```

---

## 五、使用示例：策略链的实战应用

```
Function<String, String> trim = String::trim;  
Function<String, String> upper = String::toUpperCase;  
Function<String, String> validate = s -> {  
    if (s.length() > 5) throw new IllegalArgumentException("Too long");  
    return s;  
};  
Function<String, Integer> parse = Integer::parseInt;  
  
FunctionChain<String, Integer> chain = FunctionChain  
    .<String, String>start()  
    .then(trim, onErrorContinue())  
    .then(upper, onErrorBreak())  
    .then(validate, onErrorFallback("OK"))  
    .then(parse, onErrorFallback(0));  
  
Integerresult= chain.execute("   abcdefg   ");  
System.out.println(result);  // 输出 0，表示走了 fallback
```

---

## 六、扩展优化点（如果你愿意继续进阶）

* • **链路日志统一 TraceId：** 每个 step 自动打上链路 trace。
* • **返回中间每步状态：** 让 `execute()` 支持返回每个阶段的输出记录。
* • **支持异步链：** 把 `Function<T, R>` 升级成 `Function<T, CompletableFuture<R>>`。
* • **策略链自动配置化：** 从 YAML / JSON 加载策略节点和处理方式。

---

## 七、总结：函数式的尽头是可控链条

如果你还在用裸 `Function` 做多步处理逻辑，那你已经遇到了“扩展性、异常兜底、维护成本”三个核心问题。

而本文展示的 `FunctionChain` 方案提供了一种：

* • **语义清晰**（每步是什么，出错怎么办）
* • **行为可预测**（统一策略）
* • **结构解耦**（每个 Function 都是独立可测）  
  的链式逻辑建模方式。

用 Java 写函数式，不应该追求“简短”，而是追求 **“结构清晰 + 错误可控”**。

你可以把 `FunctionChain` 作为你项目的一个基础工具类库，逐步用它替代那些又长又难测的处理逻辑，让你的系统逻辑更具可组合性和鲁棒性。

 

识别二维码关注我们

**点击分享此文**