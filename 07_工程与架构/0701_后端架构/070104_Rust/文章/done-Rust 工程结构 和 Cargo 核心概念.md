> 已吸收至：[[07_工程与架构/0701_后端架构/070104_Rust/070104_核心知识点/Rust工程边界与性能准则|Rust工程边界与性能准则]]、[[07_工程与架构/0701_后端架构/070104_Rust/070104_知识地图|070104_Rust知识地图]]

---
title: Rust 工程结构 和 Cargo 核心概念
author: 十万公里通票
date: LaurenceLaurence
url: https://mp.weixin.qq.com/s?__biz=MzIwMjgzNjA3NQ==&mid=2247486559&idx=1&sn=d42e9e2dbd47b345032f6573698b64dc&chksm=97b7a46f14eae2854cbab0aa3dddd59e46624a6b45e222e43d379cadcbb8d8ab496ee7c459a4&mpshare=1&scene=24&srcid=0529mQ4zRQlDgtpLUcoda8yT&sharer_shareinfo=0ca171da906ffd93bd1116a2c069294b&sharer_shareinfo_first=0ca171da906ffd93bd1116a2c069294b#rd
---

本文将围绕 Rust 项目的工程结构介绍一下相关的概念：crate、package、module、path、workspace。最终，我们会清晰地理解像下图所示的一个典型 Rust 项目结构中的各个组成部分：

# 1. Crate

Crate 是 Rust 中的最小编译单元（mod 不是独立的编译单元）。这一点和 C/C++、Java 等其他主流语言很不同，它们的最小编译单元可以是单个源文件。编译时，rust 会先把 crate 里的所有代码扫描一遍，建立符号表，然后再开始类型检查和编译，所以，相比 C/C++，这将带来以下显著收益：

* 没有 #include，不再需要头文件
* 规避了“分别编译”引发的大量复杂问题

Crate 有两种：会被编译为可执行程序的 `binary crate` 以及会被编译成库的 `library crate`。和 C/C++ 类似，如果是 binary crate，程序中必须要有一个 main 函数作为应用程序的“入口”。为此 cargo 规定：

* binary crate 必须提供一个 `src/main.rs` 作为根文件（编译的起始文件，必须含有一个 main 函数）
* library crate 必须提供一个 `src/lib.rs` 作为根文件（编译的起始文件，没有其他硬性要求）

注意：根文件以及它们的命名并不是 rust 语言要求的规范，而是 cargo 定的规范。实际上，对于 cargo 来说，它就是靠检测 src 下的是 `main.rs` 还是 `lib.rs` 来判定一个 crate 是 `binary create` 还是 `library crate` 的。当然，一个 crate 也可以同时含有这两个文件。

# 2. Package

一个 Packag 包含一份元数据（Cargo.toml）以及 `0..1` 个 library crate 和 `0..*` 个 binary crate，但是必须至少包含 `1` 个 crate（无论是库的还是二进制的）。

先说元数据 `Cargo.toml`，它定义了包的基本信息和依赖等若干信息，它像极了 maven 的 pom.xml，以下是一份示例：

```
[package]name = "example-crate"version = "0.1.0"edition = "2021"[dependencies]anyhow = "1.0.86"
```

从 `Cargo.toml` 提供的内容和 package 至少包含 `1` 个 crate 的规定，都可以看出来：**Rust 的一个 package 就是一个 project** （Rust 官方最佳事件也是这样推荐的）。

通常，我们并不需要显式地指出一个 package 都有哪些 crates，cargo 会基于默认的标准位置自动探测。因为 crate 有标志性的 `src/main.rs` 或 `src/lib.rs` 文件。这里， 使用了与 Maven 一样的设计哲学：约定大于配置！这里路径和文件名是可以通过配置修改的，但很少有人会这样做，使用约定的配置是最佳实践。

# 3. Module

一个 crate 里面对代码还可以根据功能和逻辑紧密程度再细分出成多个 module 并且控制它们的可见性。Rust 的 Module 是很独特的一种代码层级或者说是组织粒度，在其他语言里并没有与之完全对等的机制。它的 C++ 中的 namespace 比较接近，但是，module 和文件是强绑定的（这是好的选择）。我们来仔细看一下。在 rust 中创建一个 module 有三种方式:

## 3.1 内联模块

内联模块是使用 mod 关键字直接在文件中声明 module：

```
// main.rsmod math {    pub fn add(a: i32, b: i32) -> i32 { a + b }}fn main() {    math::add(1, 2);}
```

上述代码在 main.rs 文件中创建了一个名为 math 的 module，在这个 moduel 中定了一个函数 add，然后在 mian 函数中使用了它。

## 3.2 单文件模块

在 rust 工程中单独的源文件会被自动认定为一个 module，moudle 名与源文件同名，也就是说：**一个 .rs 文件天然就是一个 module**，以下是一个示例：

* 工程目录

```
src/  main.rs  math.rs
```

* math.rs

```
pub fn add(a: i32, b: i32) -> i32 { a + b }
```

* main.rs

```
mod math; // 告诉编译器：引入 math.rs 作为模块fn main() {    math::add(1, 2);}
```

关于 Rust 单个文件自动成为一个 module 的设计非常值得思考，这比大多数语言里的“module”粒度都要细，实际上，Rust 的 module 有其自身语言背景下的设计用意：由于 Rust 抛弃的了 Class，这使得一个逻辑上对应为一个类的代码会被拆散成很多部分，而把它们组织到一起的，其实是 module! 一个文件对应一个 module 就如同在其他语言里一个类对应一个文件，至少在粒度上是对齐的，而且 module 可以控制可见性，这又可以用来控制类成员方法的可见性，从这个应用角度看待 module 就会觉得“顺眼”得多了。以下是一个示例：

```
mod user {    pub struct User {        name: String,    }    impl User {        pub fn new(name: String) -> Self {            Self { name }        }    }}
```

在这个 module 里定义了一个完整的“类”，并且，还保证了：User 类型公开可见；name 字段私有，可见 module、struct、imple 三者组合在一起配合地非常好！

## 3.3 目录模块

内联模块和单文件模块都不太像是大型项目会使用的声明模块的方式，因为一般来说， 一个模块往往要包含多个文件，这时候，就应该使用最常用的目录模块了。顾名思义，就是：将一个目录下的所有源文件认定为一个模块，使用一个 mod.rs 文件作为 module 入口（类似 crate 的做法），module 名与目录同名。以下是示例：

* 工程目录

```
src/  main.rs  math/        🢠 文件夹    mod.rs     🢠 模块入口    utils.rs   🢠 子模块（一个天然的“单文件模块”！并不是 math 模块的直接源文件，而是其子模块！
```

* math/mod.rs

```
pub mod utils; // 引用子模块pub fn add(a: i32, b: i32) -> i32 { a + b }
```

* main.rs

```
mod math;fn main() {    math::add(1,2);    math::utils::something();}
```

注意：在这个示例中，我们也一并揭示了另一个“隐藏规则”：**在 Rust 中，永远不可能有两个文件属于同一个 module**！示例中的 utils 就是生动地印证了这个结论。当我们试图将 math 目录下的源代码组织成一个 module 时，前面介绍的 module 声明方式二：“单文件模块”规则在这里发发挥了作用，使得 utils.rs 变成了一个独立的 module，不过，这也不算是和“目录模块”规则相冲突，而是它会作为子 math 的子模块存在。

下图是以图解方式呈现的另一个例子：

## 3.4 Module Tree

介绍完 module 就很容易理解 **module tree** 了。由于 moudle 有内联模块/单文件模块/目录模块三种不同形式，这将导致 module 的结构必将和代码目录结构不再是对等的了，所以，一个 crate 的 module 自然会形成自己的一个独立的树形结构！这个 **module tree 结构是项目代码的更真实、准确的结构描述**，而 Rust 的编译器也会通过扫描代码来构建出这个 module tree 以推进代码编译工作。下图是以图解方式呈现的一个 module tree 的例子：

从上面的映射图中可以看出：module tree 和源文件的目录树并不是对等的结构，但是，从工程目录结构和文件名是可以推导到 module tree 的。

# 4. Path

当我们使用 use 关键字导入一个 module 时，就会使用 path，Rust 支持两种模块路径：绝对路径 和 相对路径。

* 绝对路径

> 项目内模块：以 `crate` 关键字开始，例如：`use crate::math::utils;`
>  第三方模块：以 crate 名称开始，例如：`use serde::Serialize;`

* 相对路径

> 相对路径只能是项目内模块，它以代码所在的模块为起点，使用 `self`、`super` 关键字在路径中导航（类似于目录中 `.` 和 `..` 字符）

# 5. Workspace

当一个项目足够复杂和庞大时，就需要使用 workspace 了。我们说：基本上，一个项目是一个 package，一个 package 对应一个 Cargo.toml，如果你熟悉 Maven，那么其实**一个 workspace 的粒度相当于一个 multi-module 的 maven 项目**。以一些常见大型项目的组成为例，一个 workspace 可能包含有这样一些典型的 crates：core，client，server等（仅仅是示意性举例，方便立即，实际的项目中可能千差万别）。

# 6. 典型项目结构

尽管在前面的诸多示例里我们已经见过一些项目的结构了，但没有对一个典型目录里的各种文件和目录作详细的介绍，本节我们用一个典型的项目结构为示例，详细介绍一下。首先，一个很重要的认识：**Cargo 采用了与 Maven 一样的设计理念：约定大于配置！它的很多文件和文件夹的命名是约定好的，有着具体而清晰的定位**。这是非常好的设计理念，它能省去大量直白且重复配置。下面是一个典型的 Cargo 项目结构：

```
my_workspace/          # 工作空间根目录（git 仓库根目录）├── Cargo.toml         # 作空间核心配置（只有 [workspace]）├── Cargo.lock         # 全局共享锁文件（所有 crate 共用）├── target/            # 全局编译输出目录（所有 crate 共享）│├── crates/            # 所有子 crate 统一放这里│   ├── core/          # 示例：核心库 (library crate)│   │   ├── Cargo.toml│   │   └── src/lib.rs│   ││   ├── api/           # 示例：接口库 (library crate)│   │   ├── Cargo.toml│   │   └── src/lib.rs│   ││   └── app/           # 示例：主程序 (binary crate)│       ├── Cargo.toml│       └── src/main.rs│├── bin/               # (可选) 小型命令行工具├── examples/          # (可选) 示例代码├── tests/             # (可选) 集成测试└── README.md
```

上述项目目录中，除了 core、api、app 这些 crate 的名称是由开发者定义之外，所有其他的**目录和文件名都是“约定”的！也就是说：99% 的情况下你不应该也不需要修改它们，就像 Maven 中的 pom.xml、src、target 一样，这是基于“约定大于配置”的理念设计的**。

# 7. 回顾与小结

最后，我们把所有元素组合起来，再回看一下文章开头给出的项目结构示例：

* 一个 Workspace: 对应一个大型项目，是多个子项目(Package)的集合，类似于 Maven 中的 Multi Module 项目
* 一个 Package: 通常对应一个独立项目，包含一个 Cargo.loml，类似于 Maven 中的 pom.xml
* 一个 Crate 代表一个编译的产出物（artifact），一个 Package 通常含有 `0..1` 个 library crate 和 `0..*` 个 binary crate，但是必须至少包含 `1` 个 crate，高度类似于 CMake 中的 target (同样分 executable 和 lib 两种）
* 一个 Module 代表一个单元模块，是一组逻辑关系紧密的源文件集合。一个 Crate 可以含有 `0..n` 个 modules。Rust 的 Moduel 比较特别，与其他语言中的“模块”有所不同，它的粒度“细至单个文件”，这在其他语言里很少见，但它又可以通过父模块放粗粒度，得到其他语言中粒度对等的模块粒度。不过，这不是对 Rust 的 Moudle 的正确看待方式。实际上，**Module 的粒度细至单个文件后形成的 Module Tree 才是（编译器眼里）更准确的工程结构**。

---

参考文章：

https://yuxuetr.com/blog/2024/05/31/rust-mod
https://iota-for-flutter.github.io/tutorial/fundamentals/rust/project-structure.html