---
title: Bun v1.3.14 深度解析：Image API、HTTP/3、全局虚拟存储与五十项变革
author: iDao技术魔方
date: iDao技术魔方iDao技术魔方
url: https://mp.weixin.qq.com/s?__biz=MzU3NTYzNzYzMQ==&mid=2247484250&idx=1&sn=ad5713bc7463cc0c2d673d7a8a3ec956&chksm=fc1e91c624894c88ff02dff270f090c4405e410a64a4c53657c8eac2a0c2031f315a4a1f1a74&mpshare=1&scene=24&srcid=0514Hpiuyw1xAQU5g5KHhfPT&sharer_shareinfo=a403ad01c28bce128ebdd6cc16052326&sharer_shareinfo_first=a403ad01c28bce128ebdd6cc16052326#rd
---

> Bun v1.3.14 发布于 2026 年 5 月 13 日。这是 Bun 历史上功能最密集的版本之一——50+ 项变更，涵盖内置图像处理引擎、HTTP/3 服务器与客户端、全新的全局包缓存架构。如果用一句话概括这个版本：**Bun 正在把自己从一个"更快的 Node.js"变成"全能的基础设施平台"**。

---

## 一、Bun.Image：不再需要 sharp

v1.3.14 最引人注目的新特性是 `Bun.Image`——一个完全内置的图像处理 API。在它之前，Node.js 生态做图像处理几乎绕不开 `sharp`（原生 C++ 模块，安装需要编译环境）。Bun 用 Zig 实现了纯编译时内联的图像编解码，不依赖任何原生模块。

### 支持格式

Bun.Image 在 v1.3.14 中支持以下格式的**解码**：

| 格式 | 支持 |
| --- | --- |
| JPEG | ✅ |
| PNG | ✅ |
| WebP | ✅ |
| GIF | ✅ (静态) |
| BMP | ✅ |
| HEIC / HEIF | ✅ |
| AVIF | ✅ |
| TIFF | ✅ |

编码输出支持 JPEG、PNG、WebP 和 AVIF。

### 处理管线

Bun.Image 的设计是"可链式组合"的——每个操作返回一个新的 Image 实例，不会就地修改：

```
import { Image } from "bun";  
  
const image = await Image.open("photo.heic");  
  
const processed = image  
  .resize(800, 600, { fit: "cover" })  
  .rotate(-90)  
  .flip("horizontal")  
  .modulate({ brightness: 0.8, contrast: 1.2 })  
  .crop(100, 100, 400, 300);  
  
// 编码输出  
await processed.save("output.avif", { quality: 85 });  
await processed.encode("jpeg", { quality: 90 });  // 返回 Buffer  
  
// 直接响应 HTTP  
const body = new Response(processed.stream("webp"));
```

### 性能对比 sharp

官方 blog 的数字：Bun.Image **通常比 sharp 快 1-2 倍**，在元数据读取场景尤为明显——`Image.metadata()` 比 sharp 的同类调用快 **70 倍**。这因为 sharp 的 metadata() 需要完整解码图像后才能解析元数据，而 Bun.Image 只在文件头部解析关键数据块就返回了结果。

resize 操作（将 4K JPEG 缩放到 1920×1080）在两个库的实现上接近持平，Bun 略快 10-20%。但 Bun.Image 的初始化和内存分配开销明显更小——因为不需要在运行时加载任何动态库。

### 滤镜与兼容性

Bun.Image 在 resize 方法上兼容 sharp 的命名参数：

```
// 和 sharp 相同的 resize 参数结构  
image.resize(800, 600, {  
  fit: "cover",     // cover / contain / fill / inside / outside  
  position: "center",  
  background: { r: 255, g: 255, b: 255, alpha: 1 },  
  withoutEnlargement: true,  
  kernel: "lanczos3",  // nearest / cubic / mitchell / lanczos2 / lanczos3  
});
```

目前缺少的高阶特性：文本叠加、SVG 渲染、复杂路径裁剪、多页 GIF 动画。但对于 90% 的图像处理场景——缩略图生成、格式转换、质量压缩、元数据提取——Bun.Image 已经是一个不需要任何安装步骤的全能方案。

---

## 二、全局虚拟存储：bun install 架构升级

Node.js 生态的包管理器在过去十年里走过了一条清晰的演化路径：从 `npm`（node\_modules 树）到 `pnpm`（link store）到 `Yarn PnP`（zip 虚拟文件系统）。Bun 在 v1.3.14 中提出了自己的方案：**全局虚拟存储**（Global Virtual Store）。

### 原理

在 `bun install` 新增的 `--linker=isolated` 模式下，如果启用 `globalStore=true`：

1. 1. 所有包的物理内容只下载**一次**到全局缓存（`~/.bun/install/global/`）
2. 2. 每个包的 node\_modules 中只存放一个**虚拟映射文件**（vfs 文件）
3. 3. Bun 的运行时在模块加载时通过 vfs 文件直接访问全局缓存中的包内容

这个方案的精妙之处在于：**对 node\_modules 的目录结构不做任何破坏性改变**。不像 pnpm 用 symlink 改变了 node\_modules 的物理布局，也不像 Yarn PnP 完全抛弃了 node\_modules。Bun 的 node\_modules 看起来和 npm 一样——只是里面的文件变成了轻量级索引。

### 性能数据

官方测试：安装 1,400 个包（Next.js 应用），**冷启动**首次安装：

| 方式 | 耗时 | 备注 |
| --- | --- | --- |
| bun install（默认） | ~6.2s | 标准方式，需要大量文件拷贝 |
| npm install | ~42s | 基准 |
| pnpm install | ~18s |  |

**预热后**（相同依赖再次安装，仅更新锁文件）：

| 方式 | 耗时 | clonefileat 调用 |
| --- | --- | --- |
| bun install（默认） | ~841ms | ~1400 |
| bun install（globalStore） | **~115ms** | **0** |

全局虚拟存储的预热安装快了约 **7x**。clonefileat 调用降为零意味着在持续集成（CI）等场景中，磁盘 I/O 开销几乎被消除——对于高频安装依赖的 CI 流水线来说，这是实打实的时间节省。

---

## 三、HTTP/3 服务器：509K req/s 的 QUIC

### 服务器端 QUIC

Bun.serve 在 v1.3.14 中加入了**高度实验性**的 HTTP/3（QUIC）支持，底层使用 `lsquic v4.6.2` 库。

```
const server = Bun.serve({  
  port: 443,  
  // 同时监听 HTTP/1.1、HTTP/2 和 HTTP/3  
  tls: { key: Bun.file("key.pem"), cert: Bun.file("cert.pem") },  
  h3: true,  // 🔥 开启 HTTP/3  
  
  fetch(req) {  
    return new Response("Hello from HTTP/3!");  
  },  
});
```

关键设计：**同一个 fetch() 处理器同时支持 HTTP/1.1、HTTP/2 和 HTTP/3**。不需要为不同协议写不同的路由逻辑。

性能对比（静态路由，TLS）：

| 协议 | 请求/秒 |
| --- | --- |
| HTTP/1.1 (HTTPS) | ~189K |
| HTTP/2 | ~510K |
| HTTP/3 (QUIC) | ~509K |

H2 和 H3 在这个基准上几乎持平。但 QUIC 的真正优势体现在丢包、高延迟网络环境（移动网络、跨洋连接）——传统 TCP 在这种场景下的队头阻塞问题会严重降低吞吐，而 QUIC 基于 UDP 的独立流设计天然不受影响。

说"高度实验性"不是谦虚。目前的 H3 实现还缺乏一些生产级特性——连接迁移、0-RTT 重连的优化、大规模并发下的内存管理——但这个方向已经明确了：Bun 的目标是在协议层面做到不需要反向代理。

### HTTP/2 客户端

fetch() 在 v1.3.14 中新增了实验性 HTTP/2 支持。和 H2 服务器的搭配意味着同一个 TCP+TLS 连接可以被多个并发的 fetch() 请求复用：

```
// 自动使用 H2（如果服务端支持）  
const response = await fetch("https://example.com/api", {  
  protocol: "h2",  // 可选: "h2" | "h3" | "http1.1"  
});
```

H2 客户端还内置了对 CONTINUATION flood 攻击的防护——这是一个 CVE 级别的安全问题，某些畸形的 H2 帧会导致服务端内存耗尽。

### HTTP/3 客户端

fetch() 也支持通过 HTTP/3 发起请求。支持 `Alt-Svc` 头部自动升级——如果一个服务返回了 `Alt-Svc: h3=":443"`，下次 fetch 自动换成 QUIC。

```
// 强制使用 HTTP/3  
const response = await fetch("https://h3-enabled.example.com", {  
  protocol: "h3",  
});
```

---

## 四、--no-orphans：进程孤儿院关门了

当 Bun 作为父进程启动子进程，而父进程崩溃或被强制杀死时，子进程会成为"孤儿"继续运行——这是长期困扰 Node.js 进程管理的问题。

v1.3.14 引入的 `--no-orphans` 标志解决了这个问题：

```
bun --no-orphans run server.ts
```

实现机制因平台而异：

| 平台 | 机制 |
| --- | --- |
| Linux | `prctl(PR_SET_PDEATHSIG, SIGTERM)` |
| macOS | kqueue `EVFILT_PROC` + `NOTE_EXIT` |
| Windows | Job Object +`SetProcessShutdownParameters` |

策略是"stop-verify-kill"：先发 SIGTERM 让子进程优雅退出，等待一段时间后检查是否仍在运行，如果还在就 SIGKILL。

这个功能对于 Docker 容器内运行 Bun 的场景尤其重要——容器终止时，作为 PID 1 的 Bun 需要确保所有子进程在退出前被正确清理，否则容器会进入僵尸进程蔓延的状态。

### process.execve()

配套的 API 是 `process.execve()`，与 Node.js v24 的接口对齐——用新进程替换当前进程：

```
process.execve("/usr/bin/node", ["--version"], {  
  env: { ...process.env, NODE_ENV: "production" },  
});
```

---

## 五、fs.watch 重写

v1.3.14 重写了文件系统监听的核心实现。这不是小修小补——之前版本的 fs.watch 有多个长期未修复的问题：

* • **Linux**：递归监听时，新建的目录不会被自动订阅——导致用户必须在回调中手动 `fs.watch` 每一级子目录
* • **macOS**：创建后立即删除再重名的文件，不会有 change 事件触发
* • **macOS**：双线程架构（一个读 FSEvents，一个读 kqueue），在低端设备上额外线程的开销不可忽略

新的实现统一使用 **inotify（Linux）/ FSEvents（macOS）/ kqueue（FreeBSD）**。在 macOS 上不再使用双线程，单线程处理 FSEvents 流，通过 `kqueue` 仅处理自建目录的拦截——线程数减半，唤醒次数减半。

对于需要在 Webpack/Vite 热更新之外做精细化文件监听的场景（如自动重载配置、开发工具、构建管道），这次重写的价值是实实在在的。

---

## 六、SSL\_CTX 共享与连接池

一个容易被忽略但影响面很大的修复：**Shared SSL\_CTX cache**。

问题背景：每次 TLS 连接都初始化一个独立的 SSL 上下文集。对于 MongoDB、Mongoose 这类会大量创建短连接的场景，每个连接额外消耗 ~50KB 的 TLS 状态。在高并发下（例如一个 Web API 服务每请求查询 MongoDB），内存增长很快。

Bun 在 v1.3.14 中将 SSL\_CTX 改为全局共享——同一目标地址的所有连接共享 SSL 会话上下文。对于 MongoDB/Mongoose 用户，内存泄漏问题基本消除；对于一般 fetch() 调用，TLS 握手开销也显著降低。

---

## 七、跨语言 LTO 与二进制瘦身

Zig 是一种编译时和 C ABI 交互非常方便的语言。Bun 的核心由 Zig 编写，上层绑定大量 C/C 库（libuv、lsquic、libarchive 等）。v1.3.14 引入了 \*\*Zig ↔ C 的跨语言链接时优化（LTO）\*\*。

效果：

| 指标 | 改善 |
| --- | --- |
| HTTP 处理 | +3.5% |
| 二进制大小（Windows） | -17MB |
| 二进制大小（Linux） | -8MB |
| 二进制大小（macOS） | -9MB |

LTO 的难处在于 Zig 的编译模型和传统的 C LTO 不同。Bun 团队花了不少功夫让两者在 LLVM IR 层面能互相优化——例如 Zig 编译的矩阵运算代码直接内联到 C 的 JSON 序列化循环中。

---

## 八、JavaScriptCore 升级与性能细节

JSC（JavaScriptCore）在 Bun 中是底层引擎（Bun 不采用 V8）。v1.3.14 同步了上游 **565 个提交**，带来一系列性能改进：

* • **更快的 async**：async 函数的调用和创建开销降低了约 15%
* • **`Array.prototype.shift`**：shift 一个空数组不再分配新对象（这是一个常见的"问题代码"——`arr.shift()` 被误用的情况很多，JSC 现在优化了这个模式）
* • **JSON.parse 短字符串**：对小于 64 字节的 JSON 字符串采用快速路径
* • **`Intl.NumberFormat`**：格式化性能改进
* • **Wasm relaxed SIMD**：WebAssembly SIMD 指令集支持更广泛

### ESM 加载提速

Bun 的模块加载器在这个版本中做了优化。根据官方 benchmark，加载 500 个 ES 模块从 140ms 降至 **123ms（~12% 加速）**。优化点主要是减少了模块解析时的字符串复制和哈希计算。

### GC 开销降低

内置类（如 `Response`、`Request`、`URL`）的垃圾回收开销通过 JSC API 变更降低——不再需要每个实例都生成额外的包装对象。

---

## 九、生态兼容性与修复

| 修复内容 | 影响 |
| --- | --- |
| Windows `tls.getCACertificates` | macOS 不再因加载 CA 证书而 stall；Windows 能正确加载 Intermediate CA 和 TrustedPeople 存储 |
| `WebSocket perMessageDeflate: false` | 之前被忽略，现在被正确尊重 |
| Windows `SIGHUP` / `SIGBREAK` | 现在可以注册信号处理器 |
| `Bun.Terminal` Windows | 通过 ConPTY 实现终端 API，输入处理不再阻塞 |
| `using` / `await using` | 当 target=Bun 时，不再被降级为兼容代码——原生 JSC 支持 |
| `bun publish` | 现在把 README 发送到注册表了（之前漏了） |
| **SQLite** | 升级到 3.53.0 |
| FreeBSD / Android 构建 | 支持这两个平台 |

---

## 十、Bun 正在变成什么

从 v1.3.14 这个版本来看，Bun 的发展方向逐渐清晰：

第一，**内置替代原生模块**。Bun.Image 是对 sharp 的替代——不是说 sharp 不好，而是"编译原生模块"这个流程本身就有问题。Bun 正在逐个消灭需要 `node-gyp` 或 `prebuild-install` 才能使用的场景。

第二，**协议层的内置支持**。H2 和 H3 客户端/服务器意味着用户越来越不需要在 Bun 前面再放一个 Nginx 或 Caddy。对大多数中小型服务来说，Bun.serve 本身已经是一个够用的反向代理。

第三，**包管理器的工业化**。全局虚拟存储的 115ms 安装时间把"安装依赖"这个操作从"去泡杯咖啡"变成了"打一个喷嚏就完了"。这个改进的受众不只是开发者——CI 流水线是更大的受益者。

第四，**跨语言优化的极限**。Zig↔C++ LTO 说明 Bun 团队在挖掘 JavaScript 运行时底层的每一点性能潜力。3.5% 的 HTTP 加速看起来不大，但这个数字来自于链接时优化——不修改任何代码就能让所有用户受益，这是纯工程投入的回报。

v1.3.14 不是 Bun 的一个"转折点"版本，但它密集地展示了这个项目的技术能力边界。从图像处理到 QUIC 协议栈到包缓存架构，Bun 正在覆盖越来越多传统上由不同独立工具处理的功能领域。这对于开发者意味着更少的工具链拼装和更一致的开发体验，对于运行时间意味着更少的跨进程通信开销和更大的优化空间。

说 Bun 会取代 Node.js 可能还为时过早——它的生态兼容性还不是 100%，Windows 的支持虽然在大幅改进但仍有边缘情况。但 v1.3.14 传递的信号很明确：Bun 不是在做"又一个 Node 替代品"，它在重新定义"一个现代 JavaScript 运行时应该内置什么"。

> 原创技术博客 · 开源项目分享 · AI全栈创作社区  idao.fun