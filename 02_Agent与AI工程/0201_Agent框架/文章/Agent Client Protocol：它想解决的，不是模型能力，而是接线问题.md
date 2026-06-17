---
title: Agent Client Protocol：它想解决的，不是模型能力，而是接线问题
author: 码上源泉
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNzg1NjUzNg==&mid=2247484227&idx=1&sn=f8dd6269f329abfac618d47ad760e86e&chksm=c30a9b3c96fa68c7e1b96a85bff3fbf19a1d4afe4a095d0d0222553b323f989d8217eda45dec&mpshare=1&scene=24&srcid=04098lKwWBWiCXMNgmb1ruT2&sharer_shareinfo=5e6fa5e0e8284a1a8e2a8f1790f96bdc&sharer_shareinfo_first=5e6fa5e0e8284a1a8e2a8f1790f96bdc#rd
---

## ACP 是什么

ACP 全称是 **Agent Client Protocol**。

它是一套开放标准，用来约定**编辑器 / IDE / 客户端**，怎么和 **AI Agent** 通信。

你可以把它理解成 AI 编程世界里的“通用插座”。

前面是 JetBrains、Zed、VS Code 插件、终端工具这类客户端。  
后面是 Copilot、Gemini CLI、Claude Code 这类 Agent。  
ACP 就是夹在中间的那层协议。

它不负责替你写代码。

它负责的是：

* • 怎么建立连接
* • 怎么创建会话
* • 怎么发 prompt
* • 怎么持续回传进度
* • 怎么处理权限请求
* • 怎么结束或恢复一次任务

如果你熟悉 LSP，那理解 ACP 会更快。

LSP 解决的是“一个语言服务器，怎么被多个编辑器接起来”。  
ACP 想解决的是“一个 Agent，怎么被多个客户端接起来”。

可以看到在**WebStorm**里，可以安装很多Agent。

然后在WebStorm的对话框使用 **OpenCode** 而不需要开启终端

## ACP 解决什么问题？

核心就一个：**M×N 集成问题**。

什么意思？

假设现在有：

* • 10 个编辑器或客户端
* • 10 个 AI Agent

如果没有统一协议，理论上就要做 100 次集成。

每一对组合，都得自己处理：

* • 会话怎么建
* • 消息怎么传
* • 权限怎么管
* • 流式输出怎么显示
* • 任务怎么取消

这件事不是技术上做不到。

是做起来特别碎。

对编辑器厂商来说，接一个 Agent，就是一份新工作。  
对 Agent 团队来说，进一个新客户端，又是一份新工作。

最后的结果就是三件事：

1. 1. 集成成本高
2. 2. 用户切换成本高
3. 3. 容易形成厂商绑定

这也是为什么 ACP 现在值得看。

它不是在发明第 N 个 AI 编程产品。  
它是在补“互操作”这一层。

说白了，它想把原来那种“一对一焊死”的接法，改成更像 USB-C 的接法。

不是所有东西瞬间自动兼容。

但至少，大家开始朝同一个插口标准靠。

## ACP 是怎么工作的？

ACP 底层走的是 **JSON-RPC 2.0**。

大多数本地场景里，客户端会拉起一个 Agent 子进程，然后通过 **stdin / stdout** 和它通信。  
官方架构文档里写得很直接：当用户连接某个 Agent 时，编辑器会按需启动这个子进程，通信走标准输入输出。

一轮典型流程，大概是这样：

1. 1. 客户端先初始化连接
2. 2. 看看这个 Agent 支持哪些能力
3. 3. 新建一个 session，或者恢复旧 session
4. 4. 把 prompt 发过去
5. 5. Agent 一边干活，一边回传进度更新
6. 6. 如果需要跑命令、访问资源、申请权限，再继续交互
7. 7. 任务结束，客户端展示结果

这套流程听起来不酷。

但价值恰恰就在这。

因为只要双方都按这套规则来，客户端就不用为每个 Agent 重新发明一套通信方式。

## 现在谁在推 ACP？

截至 **2026 年 3 月 28 日**，ACP 已经不是一个“只有概念”的东西了。

几个信号比较明确。

### 1. JetBrains 和 Zed 在一起推

这件事本身就值得看。

因为这不是一家厂商自己搞自留地，而是两家编辑器生态一起推开放协议。

JetBrains 在 2026 年 2 月介绍 ACP Agent Registry 时提到，他们和 Zed 一起做了一个 Agent 目录，目的是让用户能在 IDE 里更方便地发现、安装、连接 ACP Agent。

这说明 ACP 已经开始往“能直接拿来接”的方向走，而不是只停在协议文档。

### 2. GitHub Copilot CLI 已经公开预览支持 ACP

GitHub 在 **2026 年 1 月 28 日** 的公告里明确写了，Copilot CLI 已经实现 ACP，并进入 public preview。

官方给出的启动方式也很直接：

```
copilot --acp
```

或者开一个端口：

```
copilot --acp --port 8080
```

这个信号很关键。

因为这说明 ACP 不是作者自己在自嗨，已经有主流 Agent 产品正式给出 ACP 入口了。

### 3. 官网列出来的客户端已经不少了

按照 ACP 官网 2026 年 3 月的 clients 页面，已经能看到不少客户端或接入形态，比如：

* • JetBrains
* • Zed
* • Visual Studio Code 的 ACP Client 扩展
* • neovim 相关插件
* • Emacs
* • Obsidian
* • Unity 相关客户端

## 参考资料

* • ACP 官网：`https://agentclientprotocol.com/`
* • ACP Protocol Overview：`https://agentclientprotocol.com/protocol/overview`
* • ACP Architecture：`https://agentclientprotocol.com/get-started/architecture`
* • ACP Clients 页面：`https://agentclientprotocol.com/get-started/clients`
* • Agent Client Protocol GitHub 仓库：`https://github.com/zed-industries/agent-client-protocol`
* • JetBrains：ACP Agent Registry 介绍：`https://blog.jetbrains.com/ai/2026/02/acp-agent-registry/`
* • GitHub Changelog：Copilot CLI 支持 ACP（2026 年 1 月 28 日）：`https://github.blog/changelog/2026-01-28-acp-support-in-copilot-cli-is-now-in-public-preview/`

---

如果你觉得这篇文章对你有帮助，记得点赞、分享，关注，万分感谢！