---
title: 把 Claude Code 塞进 Obsidian：不是调 API，是真 fork 了一个子进程
author: 匠心格物
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkxNDczMDg0Nw==&mid=2247485185&idx=1&sn=083599407f734267d237ef791b170f6a&chksm=c0ae40fcb88a5aafc9948a741d46920d8e59fe9b1be6ce1463e45777c764e506cae90811dda8&mpshare=1&scene=24&srcid=0419Aja44IctjS4CVS0Atmfv&sharer_shareinfo=193c8052893e7c3dc9c539eb32332bef&sharer_shareinfo_first=193c8052893e7c3dc9c539eb32332bef#rd
---

关键字：Electron、CLI子进程、customSpawn、AsyncIterable 消息队列、崩溃恢复、跨域 instanceof

## 摘要

博文核心是 Claudian 插件如何在 Obsidian（Electron）里嵌入 Claude Code CLI 子进程。围绕三个工程难题展开源码拆解：PATH 缺失时暴力扫描 20+包管理器路径、AbortSignal 跨 V8 realm 的 instanceof 失败用 duck-typing 绕过、崩溃恢复状态机区分可重试与不可重试故障。

## 在笔记软件里跑一个 CLI 智能体

Claudian 是一个 Obsidian 插件，它把 Claude Code（Anthropic 官方的命令行 AI 编程助手）塞进了 Obsidian 的侧边栏。不是调 API，不是套壳 WebView，是真的在 Electron 里 fork 了一个 CLI 子进程，让它读写文件、跑 bash、调度子智能体。

听起来不难？试试看：

* Obsidian 是 Electron 应用，`$PATH` 只有可怜的几个系统目录，`node` 命令根本找不到
* Electron 和 Node.js 是不同的 JavaScript 运行时域，`AbortSignal instanceof EventTarget` 会返回 `false`
* CLI 子进程随时可能崩溃，而你的用户正盯着一个"思考中…"的动画等回复

Claudian 用 3.7 万行 TypeScript 把这些问题逐个解决了。今天我们不讲它能做什么，来看它怎么做到的。

## 不只是一个聊天插件

把 CLI 工具嵌入 GUI 应用，是一类很常见但又很容易踩坑的工程问题。Claudian 的解法值得细看。

这不是一个"调 API + 渲染 Markdown"的聊天插件。Claude Code 本身是一个完整的 CLI 智能体，通过 `@anthropic-ai/claude-agent-sdk` 派生 Node.js 子进程，用 JSON-RPC 协议通信，支持工具调用、子智能体协调、会话持久化。

要在 Obsidian 里嵌入这样一个东西，Claudian 得解决三个问题：

1. 进程派生：GUI 环境下怎么找到并启动 CLI 进程
2. 消息调度：在"一次一个轮次"的约束下管理用户消息
3. 容错恢复：子进程崩溃时自动检测和恢复

接下来逐一拆解这三个问题的源码实现。

## 数据怎么流的

Claudian 的架构可以用"三层四段"来概括：

* 三层：UI 层（Tab/Renderer）、服务层（ClaudianService）、SDK 层（claude-agent-sdk）
* 四段：消息发送 → 进程派生 → 流式消费 → UI 渲染

这里值得注意的是：SDK 把 MessageChannel 当作一个普通的 AsyncIterable 来消费，完全不知道背后有消息合并、背压控制、轮次管理。消费端只管 `for await...of`，生产端随便你怎么折腾。

## 三段值得细看的源码

### customSpawn：在 Electron 里找到 Node.js

`src/core/agent/customSpawn.ts` + `src/utils/env.ts`

在终端里敲 `node`，shell 会沿着 `$PATH` 找到它。但 Obsidian 是 GUI 应用，它继承的 `$PATH` 可能只有 `/usr/bin:/bin:/usr/sbin:/sbin`，连 `/usr/local/bin` 都没有。

Claudian 的解法分两步。

第一步：暴力扫描所有可能的 Node.js 安装路径（`src/utils/env.ts`）：

```
// 为 GUI 应用增强 PATH——覆盖你能想到的所有包管理器  
function getExtraBinaryPaths(): string[] {  
const home = getHomeDir();  
  
if (isWindows) {  
    // nvm-windows  
    const nvmSymlink = process.env.NVM_SYMLINK;  
    if (nvmSymlink) paths.push(nvmSymlink);  
  
    // volta  
    const voltaHome = process.env.VOLTA_HOME;  
    paths.push(path.join(voltaHome ?? home + '/.volta', 'bin'));  
  
    // fnm (Fast Node Manager)  
    const fnmMultishell = process.env.FNM_MULTISHELL_PATH;  
    if (fnmMultishell) paths.push(fnmMultishell);  
  
    // Chocolatey, scoop, 官方安装器...  
    // ...总计 20+ 个路径  
  }  
  
if (isDarwin) {  
    // Homebrew (Intel + Apple Silicon 两个路径)  
    paths.push('/usr/local/bin', '/opt/homebrew/bin');  
  
    // nvm: 解析 ~/.nvm/alias/default 找到实际版本  
    const nvmBin = resolveNvmDefaultBin();  
    if (nvmBin) paths.push(nvmBin);  
  
    // volta, fnm, asdf, mise...  
  }  
  
return paths;  
}
```

这段代码够直接的：把 nvm、volta、fnm、asdf、mise、homebrew、chocolatey、scoop 等几乎所有主流 Node 版本管理器的安装路径全列出来，macOS Intel/Apple Silicon、Windows、Linux 三个平台分别处理。有点笨，但管用。

第二步：自定义 spawn 绕过 AbortSignal 跨域问题（`src/core/agent/customSpawn.ts`）：

**实现拆解**：

注意第 33 行的注释。正常 Node.js 环境中，你可以直接把 `AbortSignal` 传给 `spawn()`，Node 会自动在 signal abort 时终止子进程。但在 Electron 中，主进程和渲染进程可能运行在不同的 V8 隔离区（realm），导致 `signal instanceof EventTarget` 返回 `false`，触发 Node.js 内部的类型检查报错。

解法：不传 signal，手动用 `addEventListener` 监听 abort 事件然后 `kill()`。`addEventListener` 是 duck-typing 式调用，不依赖 `instanceof` 检查，直接绕过了跨域问题。

如果你也在做 Electron/Tauri 插件开发，碰到 Node.js 内部的 `instanceof` 检查失败，别想着 polyfill 原型链，换一种不依赖 `instanceof` 的调用方式通常更干净。

---

### MessageChannel：用 AsyncIterable 管理消息队列

`src/core/agent/MessageChannel.ts`

Claude Code SDK 期望接收一个 `AsyncIterable<SDKUserMessage>` 作为消息源，它会用 `for await...of` 不断取出用户消息来处理。Claudian 需要在这个接口后面实现：

1. 一次一个轮次：上一个回复没完成之前，新消息必须排队
2. 文本合并：用户连续快速发送的多条文本消息合并成一条
3. 附件去重：带图片的消息只保留最新一条
4. 溢出保护：队列满了直接丢弃最新消息

```
export class MessageChannel implements AsyncIterable<SDKUserMessage> {  
private queue: PendingMessage[] = [];  
private turnActive = false;  
private resolveNext: ((value: IteratorResult<SDKUserMessage>) => void) | null  
    = null;  
  
enqueue(message: SDKUserMessage): void {  
    if (!this.turnActive) {  
      if (this.resolveNext) {  
        // 消费者正在等待——直接递送，标记轮次激活  
        this.turnActive = true;  
        constresolve = this.resolveNext;  
        this.resolveNext = null;  
        resolve({ value: message, done: false });  
      } else {  
        // 消费者还没来——入队等待  
        this.queue.push(/* ... */);  
      }  
      return;  
    }  
  
    // 轮次激活中——文本消息合并，附件消息替换  
    if (!hasAttachments) {  
      constexistingText = this.queue.find(m => m.type === 'text');  
      if (existingText) {  
        existingText.content += '\n\n' + newContent; // 合并！  
      }  
    } else {  
      // 带附件：后来者替换先到者  
      constexistingIdx = this.queue.findIndex(m => m.type === 'attachment');  
      if (existingIdx >= 0) this.queue[existingIdx] = { type: 'attachment', message };  
    }  
  }  
  
  [Symbol.asyncIterator](): AsyncIterator<SDKUserMessage> {  
    return {  
      next: () => {  
        if (this.queue.length > 0 && !this.turnActive) {  
          this.turnActive = true;  
          returnPromise.resolve({ value: this.dequeue(), done: false });  
        }  
        // 没有消息——挂起，等 enqueue 时 resolve  
        returnnewPromise(resolve => { this.resolveNext = resolve; });  
      },  
    };  
  }  
  
  onTurnComplete(): void {  
    this.turnActive = false;  
    // 如果队列里有待发消息，立即递送  
    if (this.queue.length > 0 && this.resolveNext) {  
      this.turnActive = true;  
      this.resolveNext({ value: this.dequeue(), done: false });  
    }  
  }  
}
```

**实现拆解**：

关键在 `resolveNext` 这个变量。它存的是 `next()` 方法中创建的 Promise 的 resolve 函数。消费者调用 `next()` 但没消息时，Promise 挂起；生产者调用 `enqueue()` 时，直接 resolve 这个 Promise，消费者立即收到消息。

本质上这是一个手写的无锁生产者-消费者队列，用 Promise 替代了传统的条件变量（Condition Variable）。不到 200 行，覆盖了四种边界情况：

比起回调地狱或者 EventEmitter，这种 `AsyncIterable` + Promise resolve 的方式写出来更清楚，类型也更安全。

---

### 崩溃恢复：别让用户看到白屏

`src/core/agent/ClaudianService.ts` 第 550-632 行

CLI 子进程可能在任何时刻崩溃：Node.js 版本不兼容、内存溢出、网络中断。用户看到的不应该是一个卡死的"思考中…"。

Claudian 的崩溃恢复靠两个状态变量：

```
class ClaudianService {  
  // 最后发送的消息——崩溃时用来重放  
  private lastSentMessage: SDKUserMessage | null = null;  
  // 是否已经尝试过崩溃恢复（防止无限重启）  
  private crashRecoveryAttempted = false;  
  
  private startResponseConsumer(): void {  
    const queryForThisConsumer = this.persistentQuery; // 快照当前 query  
  
    this.responseConsumerPromise = (async () => {  
      try {  
        forawait (const message of this.persistentQuery!) {  
          if (this.shuttingDown) break;  
          awaitthis.routeMessage(message);  
        }  
      } catch (error) {  
        // 防止被替换的旧消费者干扰新消费者  
        if (this.persistentQuery !== queryForThisConsumer  
            && this.persistentQuery !== null) {  
          return; // 已被新 query 替代，静默退出  
        }  
  
        const handler = this.responseHandlers.at(-1);  
        const messageToReplay = this.lastSentMessage;  
  
        // 崩溃恢复条件：未尝试过 + 有消息可重放 + 还没收到任何 chunk  
        if (!this.crashRecoveryAttempted  
            && messageToReplay  
            && handler  
            && !handler.sawAnyChunk) {  
          this.crashRecoveryAttempted = true;  
  
          // 重启持久查询，保留现有 handler  
          awaitthis.ensureReady({ force: true, preserveHandlers: true });  
          // 重放最后一条消息  
          this.messageChannel!.enqueue(messageToReplay);  
          return; // 新消费者接管，本轮退出  
        }  
  
        // 无法恢复——通知 handler 报错  
        handler?.onError(error);  
  
        // 即使当前消息失败，也重启 query 为下一条消息做准备  
        if (!this.crashRecoveryAttempted) {  
          this.crashRecoveryAttempted = true;  
          awaitthis.ensureReady({ force: true });  
        }  
      }  
    })();  
  }  
}
```

**实现拆解**：

崩溃恢复的判断逻辑有几层：

* `handler.sawAnyChunk === false`：Claude 还没开始回复就崩了，大概率是进程启动失败（比如 Node 路径变了），值得重试
* `handler.sawAnyChunk === true`：已经回复了一部分然后崩了，重试可能导致消息重复或上下文错乱，直接报错
* `crashRecoveryAttempted`：一次性开关，防止"崩溃 → 重启 → 再崩溃 → 再重启"的死循环
* `queryForThisConsumer` 快照：`ensureReady({ force: true })` 会创建新的 persistentQuery 和新的消费者，旧消费者的 catch 块通过比较快照发现自己已被替代，静默退出。这解决了新旧实例并存的竞态问题

注意 `preserveHandlers: true` 这个参数，它确保重启 query 时不清空 response handlers，重放消息的响应还是会路由到原来的 handler，用户完全无感知。

管理长生命周期子进程时，"崩溃恢复"远不是 try-catch 这么简单。你得区分哪些崩溃可以重试、哪些不行，得防止恢复本身陷入循环，还得处理新旧实例并存时的竞态。

## 哪些地方还不够好

这套方案也有代价：

1. PATH 扫描是脆弱的。`getExtraBinaryPaths()` 硬编码了 20+ 种包管理器的路径模式。新的版本管理器出现（或已有的换了安装路径），就得手动更新代码。本质上是在和"世界上所有可能的 Node.js 安装方式"赛跑。
2. 崩溃恢复只有一次机会。`crashRecoveryAttempted` 是一次性开关。如果恢复后还是崩（比如 SDK 版本不兼容），用户只能看到报错。这是在用户体验和防止无限重启之间的折中。
3. 持久查询的内存开销。每个标签页维持一个持久查询（即一个 Node.js 子进程），多标签页场景下内存占用不小。项目通过 `maxTabs` 配置（默认 5 个）来限制。
4. 消息合并可能丢失语义。`MessageChannel` 把多条文本消息用 `\n\n` 拼接。如果用户在等待回复时发了"取消上面的"和"改成这样..."，合并后 Claude 看到的是一条混合消息，语义边界就模糊了。

## 3.7 万行代码在处理什么

回头看这三个问题，有个共同点：它们都不是"怎么调 API"层面的挑战，而是"怎么让两个不同世界的软件稳定共存"。

GUI 有它的 PATH 规则、运行时域和生命周期；CLI 有它的进程模型、标准输入输出和崩溃语义。把它们粘在一起，得对两边都足够熟悉，还得在一堆边界条件里找到能 work 的平衡点。

3.7 万行代码，大部分不是在实现功能，是在处理"功能和环境之间的摩擦"。

敢紧行动起来(Obsidian + Claudian)，用了就会爱上它!!!

---

你在做 Electron/Tauri 应用时，碰到过什么诡异的兼容问题？评论区聊聊。