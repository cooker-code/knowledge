---
title: 分享一些 Function<T, R>、Consumer<T>、Supplier<T> 巧妙使用技巧！
author: Java程序员成神之路
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNDgxMDI5NQ==&mid=2247485830&idx=1&sn=4064acf2cbe7645fa24d2762df9b418f&chksm=c3bfd5103d3ae024a13bd7759d972fecabde8c5e5439fc4440edc815fde66c563c3652224435&mpshare=1&scene=24&srcid=0910UNMCltizHNerqdNG6p2t&sharer_shareinfo=8cd37e423e6e0be8507c4596d9fe595e&sharer_shareinfo_first=8cd37e423e6e0be8507c4596d9fe595e#rd
---

> 字数 1131，阅读大约需 3 分钟

可能大多数人对 `Function<T, R>`、`Consumer<T>`、`Supplier<T>` 印象还停留在“Java 8 提供的函数式接口，可以写点 lambda 少点匿名类”。如果只是停留在 `list.forEach(System.out::println)` 或者 `Function<String, Integer> f = Integer::parseInt` 这种例子，其实完全没体现出它们的威力。  
在真实项目里，越是小而通用的抽象，越能成为灵活的“齿轮”。这几个接口看似简单，但组合起来能解决很多老生常谈的业务痛点。

## 1. Function：比策略模式更轻的策略

很多人写策略模式，动辄一堆类，一个接口几十个实现。其实很多时候只需要一个 `Function<Input, Output>` 就能解决。

```
// 传统策略模式：一堆实现类  
public interface DiscountStrategy {  
    BigDecimal apply(BigDecimal origin);  
}  
  
// Function 化：一行搞定  
private static final Map<String, Function<BigDecimal, BigDecimal>> discountMap = Map.of(  
    "VIP", price -> price.multiply(new BigDecimal("0.8")),  
    "STAFF", price -> price.multiply(new BigDecimal("0.7")),  
    "DEFAULT", price -> price  
);  
  
public BigDecimal calc(String userType, BigDecimal origin) {  
    return discountMap.getOrDefault(userType, discountMap.get("DEFAULT"))  
                      .apply(origin);  
}
```

这种用法的好处是**灵活组合**：你可以动态拼接 `Function`，而不用设计一堆 class。

```
Function<BigDecimal, BigDecimal> vipDiscount = p -> p.multiply(new BigDecimal("0.8"));  
Function<BigDecimal, BigDecimal> withTax = p -> p.multiply(new BigDecimal("1.13"));  
  
BigDecimal finalPrice = vipDiscount.andThen(withTax).apply(new BigDecimal("100"));
```

传统策略模式里你得在类里写 `afterProcess` 扩展点；`Function` 的组合天然就是管道。

---

## 2. Consumer：流水线里的“副作用”

`Consumer<T>` 在我看来，最大的价值在于——**可以显式表达副作用逻辑**，而且能动态挂载。

```
Consumer<String> logger = msg -> System.out.println("[LOG] " + msg);  
Consumer<String> saver  = msg -> repository.save(msg);  
  
// 串联：打印 + 存库  
Consumer<String> logAndSave = logger.andThen(saver);  
logAndSave.accept("user created");
```

我曾在一个 **异步消息处理** 项目里，用 `Consumer` 来替代冗余的监听器接口：

```
Map<String, Consumer<Message>> handler = Map.of(  
    "CREATE", msg -> createOrder(msg),  
    "CANCEL", msg -> cancelOrder(msg)  
);  
  
public void handle(Message msg) {  
    handler.getOrDefault(msg.getType(), m -> log.warn("skip: {}", m))  
           .accept(msg);  
}
```

这里的爽点在于：你不用再写一堆 if-else，也不用造接口。**业务逻辑天然就是可组合的 Consumer**。

---

## 3. Supplier：延迟计算的救命稻草

`Supplier<T>` 我觉得是最容易被低估的。很多人只把它当作“生产一个对象”，但我发现它在**延迟计算**和**条件执行**方面特别好用。

例如：数据库操作经常有这样的逻辑——如果缓存有就直接返回，否则再查库。传统写法是 if-else；但 `Supplier` 能帮你把这类逻辑“声明化”。

```
public <T> T cacheOrFetch(String key, Supplier<T> fetcher) {  
    T cached = cache.get(key);  
    if (cached != null) return cached;  
    T value = fetcher.get();  
    cache.put(key, value);  
    return value;  
}  
  
// 用法  
String user = cacheOrFetch("user:1", () -> userRepository.findById(1L));
```

这比“写死查库逻辑”更优雅，因为 `fetcher` 可以换成任意来源：RPC、文件、远程 API。  
在复杂系统里，**把可变的逻辑塞进 Supplier，主流程变得干净**。

---

## 4. 三者的组合技：轻量 DSL

最让我兴奋的玩法是三者组合，可以拼成一种轻量 DSL。比如做一个“带日志的执行模板”：

```
public <T, R> R executeWithLog(  
        Supplier<T> supplier,            // 生产输入  
        Function<T, R> processor,        // 处理逻辑  
        Consumer<R> postHandler          // 后置处理  
) {  
    T input = supplier.get();  
    R result = processor.apply(input);  
    postHandler.accept(result);  
    return result;  
}  
  
// 用法  
String res = executeWithLog(  
    () -> "raw-data",  
    data -> data.toUpperCase(),  
    out -> System.out.println("RESULT = " + out)  
);
```

在实际项目里，我把这个模式用在“数据加工流水线”：从消息队列取数据（`Supplier`）、做 ETL 处理（`Function`）、写回下游（`Consumer`）。  
代码一眼就能看出结构，比继承一堆 Template 类轻巧得多。

---

## 5. 避坑与经验

这些接口虽好，但也有几个坑需要小心：

1. 1. **过度函数化**：不是所有场景都要搞成 lambda，尤其是复杂逻辑，拆成小方法反而更清晰。
2. 2. **调试难度**：一堆 `andThen` 拼在一起，出问题时 stacktrace 不直观，最好在关键节点 log 一下。
3. 3. **异常处理**：标准的函数式接口不允许抛 checked exception，我一般会写一个工具把 `Function` 包装成能处理异常的版本。

```
@FunctionalInterface  
public interface CheckedFunction<T, R> {  
    R apply(T t) throws Exception;  
  
    static <T, R> Function<T, R> unchecked(CheckedFunction<T, R> f) {  
        return t -> {  
            try {  
                return f.apply(t);  
            } catch (Exception e) {  
                throw new RuntimeException(e);  
            }  
        };  
    }  
}
```

---

## 结语

`Function`、`Consumer`、`Supplier` 看似只是 Java 8 附带的小玩具，但当你把它们当作**构建业务逻辑的基本积木**，会发现很多模板化的模式都能被它们取代：策略模式、责任链、模板方法、监听器……  
这是一种思维方式的转变：**业务逻辑的颗粒度可以用函数组合，不是简单的类堆叠**。

 

识别二维码关注我们

**点击分享此文**