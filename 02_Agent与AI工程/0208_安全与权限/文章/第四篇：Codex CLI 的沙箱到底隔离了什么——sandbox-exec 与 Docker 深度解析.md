---
title: 第四篇：Codex CLI 的沙箱到底隔离了什么——sandbox-exec 与 Docker 深度解析
author: FutureCraft AI
date: AI追梦少年AI追梦少年
url: https://mp.weixin.qq.com/s?__biz=MzYzMzc2NTI3NA==&mid=2247484204&idx=1&sn=f3c83697cd397107e8cec4d12dfa4c70&chksm=f1f57db158e6692c6924bd9c8d0f73499baeb898796ee79949432c11003b83ca857659b46fe6&mpshare=1&scene=24&srcid=0521MFmkR3nId1yQcsQLGT2h&sharer_shareinfo=056b3186cb724f25792b4eb34ac46cc3&sharer_shareinfo_first=056b3186cb724f25792b4eb34ac46cc3#rd
---

# 第四篇：Codex CLI 的沙箱到底隔离了什么——sandbox-exec 与 Docker 深度解析

上一篇说到，Codex CLI 和 Claude Code 的核心安全哲学不同：Codex 选择了"自动隔离"，Claude Code 选择了"全程感知"。

这篇深入前者。

Codex CLI 的沙箱机制是它被反复提及的卖点之一——开箱就有，不用配置，跑 `full-auto` 放心交给它。但"有沙箱"和"沙箱隔离了什么"是两件完全不同的事。

我翻了 Codex CLI 的开源代码，实测了几个边界场景，把这个问题说清楚。

---

## 沙箱是怎么工作的

先说架构。Codex CLI 根据运行平台选择不同的沙箱实现：

* • **macOS**：使用 `sandbox-exec`，Apple 的 Seatbelt 沙箱技术
* • **Linux**：使用 Docker 容器隔离
* • **Windows**：目前没有内置沙箱（WSL 环境可以走 Linux 路径）

两种实现的目标一致：限制 AI 生成的代码能做什么，防止它在你不知情的情况下访问不该访问的资源。

但"限制"的具体内容差异很大。

---

## macOS：sandbox-exec 和 Seatbelt

macOS 的 `sandbox-exec` 是 Apple 的 Seatbelt 技术，最早用于沙箱化 Safari 的渲染进程和 App Store 应用。它通过一套声明式的 profile 文件定义"允许什么"，没被允许的操作默认拒绝。

Codex CLI 在 macOS 上运行命令时，会用 `sandbox-exec -f <profile>` 包裹执行。profile 文件内容大致如下：

```
  (version 1)  
(deny default)  
  
; 允许读取项目目录  
(allow file-read* (subpath "/path/to/project"))  
; 允许写入项目目录  
(allow file-write* (subpath "/path/to/project"))  
  
; 允许读取系统库（命令执行需要）  
(allow file-read* (subpath "/usr/lib"))  
(allow file-read* (subpath "/usr/local/lib"))  
  
; 允许进程创建（执行子命令需要）  
(allow process-exec)  
(allow process-fork)  
  
; 网络：默认拒绝
```

**这个 profile 实际隔离了什么：**

1. **文件系统**：只能读写你的项目目录，访问 `~/Documents`、`~/.ssh`、`~/.aws` 等敏感路径会被拒绝
2. **网络**：默认情况下，网络访问被拒绝——沙箱内的代码无法发起出站请求
3. **系统调用**：不允许加载内核扩展，不允许修改系统配置

**这个 profile 没有隔离什么：**

问题在这里。

Codex CLI 的 sandbox profile 是它自己生成的，不是操作系统强制的。而且 profile 是相对宽松的——它必须允许执行工具链（node、npm、python、git），这意味着很多系统路径是可读的。

更关键的是：**AGENTS.md 里声明的"允许自动执行命令"是不走沙箱的**。

```
  ## 允许自动执行的命令  
- npm test  
- npm run build  
- git add  
- git commit
```

这些命令被标记为白名单后，Codex CLI 会直接执行，不通过 sandbox-exec 包裹。逻辑上讲得通——你明确授权的命令应该有完整的能力——但这意味着沙箱边界由你写的 AGENTS.md 决定，而不是系统决定。

---

## Linux：Docker 隔离

Linux 上的隔离更彻底，也更容易理解。

Codex CLI 在 Linux 上会启动一个 Docker 容器，把你的项目目录挂载进去，AI 生成的命令在容器内执行：

```
  docker run --rm \  
  -v /path/to/project:/workspace \  
  -w /workspace \  
  --network none \          # 默认无网络  
  --read-only \             # 根文件系统只读  
  codex-sandbox:latest \  
  <command>
```

Linux 的方案干脆得多：

* • 容器内看不到宿主机的文件系统，只看到挂载进来的项目目录
* • `--network none` 意味着完全无法联网
* • 容器以非 root 用户运行

**但有一个前提**：你的机器上需要有 Docker，且 Docker daemon 在运行。如果没有 Docker，Codex CLI 会回退到"无沙箱"模式，直接在宿主机执行——这一点在文档里写得比较轻描淡写。

```
  # 检查 Codex CLI 是否真的在用沙箱  
codex --sandbox-status  
# 或者查看日志输出里有没有 [sandbox] 前缀
```

---

---

## full-auto 模式的真实风险边界

`full-auto` 是 Codex CLI 最吸引人的模式——你说一句需求，它自己跑完整个任务，中间不打扰你。

问题在于：沙箱的强度决定了这个"不打扰"有多安全。

**跑测试和格式化**：放心。这类任务完全在项目目录内，沙箱够用，`full-auto` 没问题。

**npm install / pip install**：要注意平台差异。包管理器会联网，在 macOS sandbox-exec 下，如果 profile 没有 deny network，这个请求会通过；Linux Docker 的 `--network none` 会直接让 install 失败。同一个命令，macOS 和 Linux 的沙箱行为不一样——这个不对称让我第一次发现时挺意外的。

**涉及 git 操作**：这是最容易被忽视的。`git push` 需要网络，如果你的 AGENTS.md 白名单了 git 操作，这些命令会绕过沙箱直接执行。full-auto 跑着跑着，AI 已经帮你 commit 甚至 push 了，你不一定察觉。

**读取项目外的配置（macOS）**：sandbox-exec profile 允许读系统库目录，而 AWS CLI、kubectl 这类工具会自动尝试读 `~/.aws/` 或 `~/.kube/config`。如果这些路径没被明确 deny，沙箱不拦。这不是 Codex CLI 的 bug，是 Seatbelt 的设计——你写什么 profile，就有什么边界，不多也不少。

---

---

## 如何自定义 sandbox profile

如果你需要更严格的控制，可以提供自己的 sandbox profile：

```
  # macOS：指定自定义 profile  
codex --sandbox-profile /path/to/my.sb "重构认证模块"
```

自定义 profile 示例——拒绝所有网络访问，只允许项目目录：

```
  (version 1)  
(deny default)  
  
; 只允许项目目录读写  
(allow file-read* (subpath "/Users/you/myproject"))  
(allow file-write* (subpath "/Users/you/myproject"))  
  
; 必须允许的系统路径（工具链依赖）  
(allow file-read*  
  (subpath "/usr")  
  (subpath "/bin")  
  (subpath "/private/tmp")  
  (literal "/dev/null"))  
  
; 明确拒绝敏感路径  
(deny file-read* (subpath "/Users/you/.ssh"))  
(deny file-read* (subpath "/Users/you/.aws"))  
(deny file-read* (subpath "/Users/you/.kube"))  
  
; 完全拒绝网络  
(deny network*)  
  
; 允许进程执行  
(allow process-exec)  
(allow process-fork)
```

这个 profile 比默认的更严格，代价是某些需要网络的工具会失败。根据你的任务类型选择。

---

## 与 Claude Code DevContainer 的对比

Claude Code 没有内置沙箱，但它支持通过 VS Code DevContainer 实现类似效果。

|  | Codex CLI（默认） | Codex CLI（自定义） | Claude Code + DevContainer |
| --- | --- | --- | --- |
| 开箱即用 | ✓ | 需配置 | 需配置 |
| 文件系统隔离 | 项目目录 | 可定制 | 容器内挂载目录 |
| 网络隔离 | macOS 不完全 / Linux 完全 | 可定制 | 可配置 |
| 白名单命令走沙箱 | ✗ | ✗ | N/A（权限确认） |
| 隔离强度 | 中 | 高（需成本） | 高（需配置） |
| 执行可见性 | 低（full-auto） | 低 | 高（逐步确认） |

两种方案的核心差异不是"谁更安全"，而是**隔离由谁负责**：Codex 把隔离做进工具里，Claude Code 把隔离外包给容器，把安全感还给用户的确认行为。

---

## 实际建议

用了一段时间，我自己的做法是这样的：

读写代码、执行测试、格式化、静态分析——full-auto 没问题，这些任务完全在项目目录内。

涉及网络的操作（install、push、API 调用），或者会读取 `~` 目录配置的工具——切 suggest 模式，或者自定义一个更严的 profile，别依赖默认行为。

如果你在 Linux 上：装好 Docker，让 Codex CLI 走容器路径，网络隔离是默认的，比 macOS 的 sandbox-exec 可靠得多。

如果你在 macOS 上：检查一遍 AGENTS.md 白名单，想清楚哪些命令真的需要白名单。白名单越短，沙箱才越有意义。

---

真正让沙箱有意义的，不是它存在，而是你知道它的边界在哪。

---

下一篇：**Codex CLI 多文件任务实战**——跨文件重构、大型代码库分析、真实项目里的上下文管理策略。

---

*本文是「Codex CLI 技术连载」第 4 篇。*  
*第 1 篇：Codex CLI 是什么？OpenAI 开源的终端 AI 编程 Agent | 第 2 篇：10 分钟上手 Codex CLI | 第 3 篇：Codex CLI vs Claude Code 深度对比*