---
title: 分享 Java Function 的 10 个高阶用法！
author: Java程序员成神之路
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNDgxMDI5NQ==&mid=2247485893&idx=1&sn=f933fec10e06b3c74e59db93ddd1806d&chksm=c3286df6ea15e32f26c751bda5545b4a9c1b68e873c27893098ce1094fa199c216a95acaae64&mpshare=1&scene=24&srcid=0910PeCZi7rxoW35wCqxJxse&sharer_shareinfo=ac224521b4fec5fe34724bc4b39d7db7&sharer_shareinfo_first=ac224521b4fec5fe34724bc4b39d7db7#rd
---

> 字数 1062，阅读大约需 3 分钟

 

### 引言

大多数人对 `Function<T,R>` 的理解停留在“函数式接口，支持 λ 表达式，能 map 一下集合”。但真正把它用到极致，你会发现它不仅是工具方法的简化，还能成为 DSL 的拼装器、策略的动态注入器、缓存的延迟计算器，甚至能替代部分反射逻辑。本文分享我在实际项目中总结的 10 个 Function 高阶用法，这些用法并不常见，却在关键场景下解决了不少棘手问题。

---

### 1. **惰性计算缓存器**

```
private static final Map<String, Object> CACHE = new ConcurrentHashMap<>();  
  
public static <T> T getOrCompute(String key, Function<String, T> computer) {  
    return (T) CACHE.computeIfAbsent(key, computer);  
}
```

在复杂业务里，缓存逻辑往往冗长，`Function` 让延迟计算优雅无比。  
**亮点**：规避了重复计算逻辑的散落，完全以函数拼装。

---

### 2. **枚举驱动的行为映射**

```
enum BizType {  
    ORDER_PROCESS(s -> processOrder(s)),  
    REFUND_PROCESS(s -> processRefund(s));  
  
    private final Function<String, String> action;  
    BizType(Function<String, String> action) { this.action = action; }  
    public String apply(String input) { return action.apply(input); }  
}
```

传统写法要么 `switch-case`，要么 if-else。借助 `Function`，行为直接挂在枚举上，扩展时零侵入。

---

### 3. **错误恢复策略链**

```
Function<String, String> retry = s -> {  
    try { return riskyOp(s); }  
    catch (Exception e) { return fallbackOp(s); }  
};
```

结合 `Function.andThen()` 可以拼装出**带降级策略的计算链**。比起 try-catch 零散分布，函数组合的表现力更高。

---

### 4. **动态 DSL 拼装**

```
Function<String, String> upper = String::toUpperCase;  
Function<String, String> trim = String::trim;  
  
Function<String, String> pipeline = upper.andThen(trim).andThen(s -> "[" + s + "]");  
System.out.println(pipeline.apply(" hello "));
```

通过 `Function` 链接形成 DSL，不同函数片段可由外部动态注入，拼装成完整规则。  
在规则引擎、数据清洗中尤其实用。

---

### 5. **规避反射的字段提取**

```
<T, R> R extract(T obj, Function<T, R> getter) {  
    return getter.apply(obj);  
}  
  
String name = extract(user, User::getName);
```

传统写法往往要用反射或硬编码字段名，Function 利用方法引用规避反射成本，同时保持编译期检查。

---

### 6. **延迟日志拼装**

```
void log(Function<Void, String> supplier) {  
    if (logger.isDebugEnabled()) {  
        logger.debug(supplier.apply(null));  
    }  
}
```

避免字符串拼接的无谓开销，`Function` 在这里比 `Supplier` 更灵活：可以根据输入上下文动态生成日志内容。

---

### 7. **条件函数的管道化**

```
Function<Integer, String> check = i -> i > 0 ? "POS" : "NEG";  
Function<Integer, String> decorated = check.andThen(s -> "Result: " + s);
```

很多 if/else 判断可以转为函数拼装，让逻辑像数据流一样流动。  
这种风格比硬编码分支更适合维护复杂条件的“可组合性”。

---

### 8. **线程池任务的包装器**

```
Function<Runnable, Runnable> timeCost = task -> () -> {  
    long start = System.nanoTime();  
    task.run();  
    System.out.println("Cost: " + (System.nanoTime() - start));  
};  
executor.submit(timeCost.apply(() -> realWork())));
```

比 AOP 更轻量，直接在任务提交处注入功能。Function 让“横切逻辑”自由组合。

---

### 9. **配置驱动的动态装配**

```
Map<String, Function<String, String>> strategies = Map.of(  
    "UPPER", String::toUpperCase,  
    "LOWER", String::toLowerCase  
);  
  
String mode = config.get("mode");  
String result = strategies.get(mode).apply("test");
```

很多场景本质是 **配置到行为的映射**，Function 使得映射更自然，避免冗余的类和反射。

---

### 10. **函数合成替代策略模式**

```
Function<Order, Order> discount = o -> { o.setPrice(o.getPrice() * 0.9); return o; };  
Function<Order, Order> coupon = o -> { o.setPrice(o.getPrice() - 20); return o; };  
  
Function<Order, Order> composed = discount.andThen(coupon);  
Order result = composed.apply(order);
```

这已经接近**无类化策略模式**。策略组合只需要函数合成，不必再定义大量类。

---

### 总结

`Function<T,R>` 的威力在于它是**最小单位的可组合逻辑**。  
一旦你习惯用它拼装，而不是写死逻辑，就会发现代码从命令式切换到声明式，灵活度和扩展性都大幅提升。

这 10 个场景并非教材上的“map/filter”，而是我在真实业务中踩坑总结的技巧。它们的共同点是：

1. 1. 用函数拼装代替 if/else 和反射。
2. 2. 把横切逻辑函数化，按需组合。
3. 3. 让行为成为一等公民，可以被配置、缓存、动态切换。

很多人说 Java 的函数式编程是“阉割版”的，但如果你真正理解 `Function` 的组合精神，就会发现它恰好是工程化最佳平衡点：够轻量，够强大，也足够可维护。

 

识别二维码关注我们

**点击分享此文**