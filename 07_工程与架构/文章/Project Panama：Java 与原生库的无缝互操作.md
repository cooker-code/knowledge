---
title: Project Panama：Java 与原生库的无缝互操作
author: 架构师营地
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk0ODQyNzEyNQ==&mid=2247484027&idx=1&sn=bb4a576ea0f4e22241ef2a1c0bb8099b&chksm=c2cf51be7962f7b9ce9172ef6feaacc2476b502e7293adcbb212b3614af0963e62e6c0df22ea&mpshare=1&scene=24&srcid=09104PpP5O0cZsAEDOw2XxJX&sharer_shareinfo=d2913fe9c187dca63a977c1ecb8fc3ad&sharer_shareinfo_first=d2913fe9c187dca63a977c1ecb8fc3ad#rd
---

在现代软件开发中，Java 一直以跨平台和稳定性著称，但在与 C/C++ 等原生库交互时，往往需要依赖繁琐的 **JNI (Java Native Interface)**。JNI 语法复杂、性能开销大、可维护性差，也成为 Java 与底层世界之间的一道“鸿沟”。

为了解决这一痛点，OpenJDK 引入了 **Project Panama** —— 一个旨在简化 Java 与原生代码（native libraries）互操作的长期计划。它让 Java 程序可以更自然地调用 C API，直接访问原生内存，降低 JNI 带来的心智负担。

---

## 一、什么是 Project Panama？

Project Panama 是 OpenJDK 的一项孵化项目，目标是：

* **简化与 C 库的交互**：无需手写 JNI 头文件和桥接代码。
* **高性能外部函数调用**：通过 FFI（Foreign Function & Memory API）实现更快的跨语言调用。
* **安全内存访问**：提供受控的 native memory API，替代 `sun.misc.Unsafe`。

核心模块包括：

1. **Foreign Function & Memory API (FFM API)**提供对原生函数和内存的安全访问。
2. **jextract 工具**自动生成 Java 绑定代码，直接使用现有 C 头文件。

Panama 在 **JDK 14** 开始孵化，经过多次预览，在 **JDK 21** 正式稳定。

---

## 二、传统 JNI 的痛点

以调用 C 标准库的 `strlen` 为例，JNI 方式需要：

* 写 Java 方法声明 `native String strlen(String s);`
* 生成 `.h` 头文件
* 编写 C/C++ 实现函数
* 编译成动态库并加载

不仅步骤繁琐，而且调试困难。

---

## 三、使用 Project Panama 的示例

### 1. 访问 C 标准库函数

假设我们要调用 C 的 `strlen`：

```
import java.lang.foreign.*;  
import java.lang.invoke.*;  
  
publicclass PanamaDemo {  
    public static void main(String[] args) throws Throwable {  
        try (Arena arena = Arena.ofConfined()) {  
            // 加载标准 C 库  
            Linker linker = Linker.nativeLinker();  
            SymbolLookup stdlib = linker.defaultLookup();  
  
            // 找到 strlen 函数  
            MethodHandle strlen = linker.downcallHandle(  
                stdlib.find("strlen").get(),  
                FunctionDescriptor.of(ValueLayout.JAVA_LONG, ValueLayout.ADDRESS)  
            );  
  
            // 在原生内存中写入字符串  
            MemorySegment cStr = arena.allocateFrom("Hello Panama!");  
              
            long len = (long) strlen.invoke(cStr);  
            System.out.println("Length = " + len);  
        }  
    }  
}
```

**运行结果**：

```
Length = 13
```

无需 JNI，直接在 Java 代码中调用 C 函数！

---

### 2. 操作原生内存

```
try (Arena arena = Arena.ofConfined()) {  
    MemorySegment segment = arena.allocate(100); // 分配 100 字节原生内存  
    segment.set(ValueLayout.JAVA_INT, 0, 42);    // 在偏移 0 写入 int  
    int value = segment.get(ValueLayout.JAVA_INT, 0);  
    System.out.println(value); // 输出 42  
}
```

这比 `ByteBuffer` 更高效、更安全，避免了 `Unsafe` 的直接使用。

---

### 3. 使用 jextract 工具

如果要调用更复杂的 C 库（比如 OpenSSL、libpng），手工编写方法签名会很麻烦。`jextract` 工具可以自动解析头文件，生成对应的 Java API。

示例：

```
jextract -t com.example.ffi -d out /usr/include/stdio.h
```

生成的 `com.example.ffi.*` 包下的类可直接在 Java 中调用，无需 JNI 代码。

---

## 四、优势对比：Panama vs JNI

| 特性 | JNI | Project Panama |
| --- | --- | --- |
| API 复杂度 | 高，需要 C/C++ 代码 | 低，纯 Java |
| 性能 | 有一定开销 | 更快，更接近原生调用 |
| 内存安全 | 容易出错，需手工管理 | 提供 `Arena` 管理内存生命周期 |
| 可维护性 | 难维护，跨平台性差 | 简单，跨平台性强 |
| 自动化 | 需手工写桥接代码 | `jextract` 自动生成绑定 |

---

## 五、应用场景

1. **调用高性能原生库**

* 图像处理：OpenCV、libpng
* 加密算法：OpenSSL
* 压缩库：zlib

2. **系统级交互**

* 调用 Linux 系统 API
* 嵌入 C 驱动接口

3. **替代 JNI/Unsafe**

* 提供安全的 native memory API
* 更好的可读性和可维护性

---

## 六、现状与未来

* **JDK 14–20**：FFM API 作为孵化/预览特性
* **JDK 21**：正式引入稳定的 FFM API
* **未来**：可能扩展到 GPU 互操作、SIMD 向量化加速

Project Panama 为 Java 打开了与原生世界的“高速通道”，让 Java 能够更轻松地利用系统能力和高性能库。

---

## 七、总结

* **Project Panama = Java 与 C/C++ 的桥梁**，简化了跨语言调用。
* 提供 **FFM API**，安全高效地访问原生函数与内存。
* 借助 **jextract**，开发者无需再写 JNI。
* 在 **高性能计算、系统编程、原生库集成** 等场景下，Panama 将极大增强 Java 的能力。

一句话总结：

> “
>
> **有了 Project Panama，Java 不再孤立，而是可以无缝拥抱原生世界。**