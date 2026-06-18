> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 2.1.90 发布：`/powerup` 教学模式上线，权限安全与性能体验一起补强
author: 阿俊聊AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzYzODc1NzU3Ng==&mid=2247483864&idx=1&sn=1ceeeea6e18e24bb6db2a7952c300c67&chksm=f1ceaaff26b2a3d859dfbbeb5ce4bc5a83bdb353cd4d93f3aafd6f52d66960cb67aacceadce4&mpshare=1&scene=24&srcid=0417RbxjZPI8zqorliu6cysz&sharer_shareinfo=428177a4b284f86fa0250857e657691f&sharer_shareinfo_first=428177a4b284f86fa0250857e657691f#rd
---

Claude Code 2.1.90 已发布。
如果只看这一版最值得关注的变化，可以浓缩成一句话：

> **新手更容易上手，老用户用起来更稳，自动化场景更安全。**

这次更新不是“花哨功能堆叠”，而是非常典型的一次**实用型增强版本**：一边补上更友好的学习入口，一边继续打磨权限控制、恢复能力、性能和稳定性。

---

## 这版最重要的 5 个更新

### 1）`/powerup` 上线：Claude Code 开始“边教边带你用”

本次最亮眼的新功能，是新增了：

* `/powerup`

它提供的是**交互式课程**，并配有**动画演示**，用来帮助用户更快理解 Claude Code 的核心能力和使用方式。

这意味着什么？

* 对新用户：上手门槛明显降低
* 对团队推广：更适合内部培训和快速普及
* 对老用户：也能更系统地补齐一些没用过的能力点

如果说之前 Claude Code 更像“强大但需要摸索的终端搭档”，那现在它开始主动承担一部分“教学产品”的角色了。

**一句话评价：这是本次版本里最有感知、也最适合被广泛传播的更新。**

---

### 2）自动模式更听话了：明确边界终于会被尊重

这次修复了一个非常关键的问题：

* **自动模式现在会更好地尊重用户明确设下的边界**
* 例如：

+ “不要 push”
+ “等 X 完成后再做 Y”

即使某个动作在权限上原本是允许的，Claude Code 现在也会优先遵守你在上下文里说清楚的限制。

这件事为什么重要？

因为对真正把 Claude Code 用进开发流程的人来说，**“能做”不等于“该做”**。
一个好用的编码代理，不只是执行能力强，更重要的是：

* 能理解约束
* 能记住边界
* 不擅自越线

这一修复，实际上是在补 Claude Code 的“协作可信度”。

---

### 3）权限与安全进一步收紧：尤其是 PowerShell 场景

2.1.90 对权限安全做了较大幅度加强，重点包括：

* **强化了 PowerShell 工具权限检查**
* 修复了多种可能绕过或削弱限制的边界问题，包括：

+ trailing `&` 后台任务绕过
+ `-ErrorAction Break` 导致的调试器挂起
+ archive extraction 的 TOCTOU 风险
+ 解析失败时 deny 规则退化的问题

另外还有一项很值得注意的调整：

* **从 auto-allow 中移除了**

+ `Get-DnsClientCache`
+ `ipconfig /displaydns`

官方给出的理由很直接：**DNS 缓存隐私**。

这说明 Claude Code 正在持续收紧“默认放行”的边界，把“方便”让位给“更稳妥的安全模型”。

**对企业、团队、以及更重视本地隐私的用户来说，这是很重要的信号。**

---

### 4）`--resume` 与长会话体验继续优化

如果你平时会频繁恢复历史会话，这次更新也很有价值。

#### 与 `--resume` 相关的关键修复与优化包括：

* 修复了在某些场景下，`--resume` 首次请求会导致**完整 prompt cache miss** 的问题

+ 使用 deferred tools 的用户
+ 使用 MCP servers 的用户
+ 使用 custom agents 的用户

+ 受影响用户主要包括：

* 优化了 `/resume` 的 all-projects 视图

+ **项目会话改为并行加载**
+ 项目多的时候，加载速度会明显更快

* 调整了 `--resume` 会话选择器的显示逻辑

+ `claude -p`
+ SDK 调用
  创建的会话

+ 不再展示由：

这类更新看起来不“炸裂”，但对重度用户很关键。
因为 Claude Code 一旦进入高频工作流，**恢复会话、切回上下文、保持流畅**，就是决定体验是否顺手的核心环节。

---

### 5）性能与稳定性继续补课：长会话、大数据流场景更友好

2.1.90 还做了一批很扎实的底层优化：

#### 性能方面

* 消除了每轮对 MCP tool schema 做 `JSON.stringify` 的额外开销
* SSE transport 处理大型流式 frame 时，从**二次复杂度**优化为**线性时间**
* SDK 长对话在写 transcript 时，不再随着会话变长而出现明显的二次变慢

#### 稳定性方面

修复了不少影响真实使用体验的问题，例如：

* 触发 usage limit 后，速率限制弹窗会反复自动打开，最终导致会话崩溃
* `Edit` / `Write` 在 format-on-save hook 改写文件后，误报 “File content has changed”
* `PreToolUse` hook 输出 JSON 并以 code 2 退出时，未能正确阻止工具调用
* 权限对话框收到异常工具输入时可能引发 UI 崩溃
* `/model`、`/config` 等选择界面滚动时标题消失
* 浅色终端主题下，部分 hover 文本几乎不可见

这些问题单看都不算“大功能”，但它们共同决定了一件事：

> **Claude Code 是否足够稳定，能不能放心长期挂在真实开发流程里。**

---

## 这次更新，适合谁重点关注？

### 1. 新上手 Claude Code 的用户

重点看：

* `/powerup` 交互式教学
* 更清晰、更稳的使用体验

这是本次最明显的“入门友好型”提升。

---

### 2. 重度终端用户 / 长会话用户

重点看：

* `--resume` 修复
* `/resume` 并行加载
* 长会话性能优化
* 各类 UI 与 hook 相关稳定性修复

如果你平时就是拿 Claude Code 跑复杂项目、跨多轮会话工作，这版值得尽快升级。

---

### 3. 团队 / 企业 / 自动化场景用户

重点看：

* 自动模式更尊重用户边界
* PowerShell 权限检查加强
* DNS 缓存相关 auto-allow 收紧
* 离线环境下 marketplace cache 保留能力

其中新增环境变量：

* `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE`

当 `git pull` 失败时，可以保留现有 marketplace 缓存，**对离线或弱网环境更友好**。

这类增强，明显是在往更可控、更适合正式环境部署的方向走。

---

## 一句话总结这次 2.1.90

如果让我用一句适合公众号封面的判断来概括这次更新，那就是：

> **Claude Code 2.1.90，最重要的不只是新增了 `/powerup`，而是它正在同时补齐“会教、会守规矩、还能稳定跑久”的三件事。**

换句话说，这不是一个只顾“更强”的版本，
而是一个明显在朝着\*\*“更适合长期协作”\*\*进化的版本。

---

## 核心更新速览

* 新增 `/powerup`：交互式课程 + 动画演示
* 新增 `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE`：离线场景更友好
* `.husky` 被加入 protected directories（acceptEdits mode）
* 自动模式更尊重用户明确边界
* 强化 PowerShell 权限检查
* 移除 `Get-DnsClientCache` 与 `ipconfig /displaydns` 的 auto-allow
* 修复 `--resume` 首次请求缓存失效问题
* `/resume` 多项目会话并行加载，更快
* 优化 MCP、SSE、SDK 长会话性能
* 修复多项 UI、hook、编辑链路稳定性问题

---

## 结尾

从 2.1.90 来看，Claude Code 的演进方向已经越来越清晰：

* **不是只拼模型能力**
* **而是在补齐真正可用的工程化体验**

对于开发者来说，这往往比单一的新功能更重要。

如果你最近正好在用 Claude Code，这一版，值得更新。