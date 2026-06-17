---
title: Claude Code 这次推出的 Agent View，auto-coder 的 /async 一年前就在跑了
author: 祝威廉
date: 祝威廉祝威廉
url: https://mp.weixin.qq.com/s?__biz=MzIyNzQyNzgxNQ==&mid=2247486147&idx=1&sn=948f6c53599db3285d5daad74c9d8a77&chksm=e9e3030c76941d0fccf83e43674504f48b7551f52275a81088f06e1d5c0065334e70ba466c6e&mpshare=1&scene=24&srcid=0512lqsu1Le28VCNSLp9VXNp&sharer_shareinfo=25f3e1c7d14544e35c78088913fb522f&sharer_shareinfo_first=25f3e1c7d14544e35c78088913fb522f#rd
---

2026-05-11 ， Anthropic 给 Claude Code 推出 **Agent View**（ research preview ，要求 v2.1.139+）：把所有后台 agent 列在一屏里，每个 agent 是独立的 `claude --bg` 进程， per-user supervisor 统一托管，文件改动写到独立的 git worktree 里。

国内 Auto-Coder-02 群第二天上午就在转。第一反应是芒果干那两句：

💬 Auto-Coder-02 群第一反应

芒果干:

祝大领先。

芒果干:

对面抄的有点快。

但翻了翻仓库，才发现这次"领先"不止是 6 天那么简单。

---

## 一、 auto-coder 其实有两个后台机制

很多人提到 auto-coder 的后台能力，第一反应是 **`/bg`** —— 5 月 6 日刚合主线（ commit `073ef0b8`）， 16 个会话槽位活在同一个 TUI 进程里，`/bg N` 是双向 tab 切换。详细介绍我之前写过一篇：给 auto-coder.chat 加上 /bg[1]。

但这只是其中一条路**。另一条更老的路叫 `/async`， 2025 年 7 月 26 日就上线了**（ commit `9b0c8878`，"实现异步代理运行器功能"）。一年前的事了。

`/async` 的设计思路非常具体：

```
# 在后台异步起一个 agent ，专门干一件事，跑完就完  
/async /name fix_login "排查并修复登录页 race condition"  
  
# 限时 5 分钟做某个调研  
/async /name probe /time 5m "扫一下 packages 里所有用 axios 的地方"  
  
# 让它循环跑 3 次  
/async /name iter /loop 3 "持续优化 README 文案，每轮都比上轮更紧"  
  
# 跑一个预定义工作流  
/async /name deploy /workflow staging-deploy
```

子命令完整一套：`/async /list` 看所有任务，`/async /task <id>` 看某个任务的输出，`/async /kill <id>` 杀任务，`/async /drop <id>` 连日志一起删除。

底层做的事，今天回头看跟 Claude Code 5/11 那套**形神俱似**：

**进程隔离**：每个任务起一个独立的 `auto-coder.run` 子进程，跟 TUI 解耦，关掉 TUI 任务还在跑

**文件隔离**：自动创建 git worktree ，挂在 `~/.auto-coder/async_agent/tasks/<task_name>/`，每个任务在自己的 checkout 里改文件，互不踩

**元数据持久化**：任务状态、查询、模型、日志路径全部落到 `~/.auto-coder/async_agent/meta/<task_id>.json`，重启 TUI 也能 `/async /list` 把历史任务列出来

**fire-and-forget**：起完就不阻塞前台，用 `/async /task <id>` 异步看结果

⚡ 一句话

这套架构是 2025 年 7 月就跑起来的——比 Claude Code 5/11 推出 Agent View 早了将近 10 个月。

---

## 二、三向对照表：架构究竟谁像谁

把三条路平铺开来对一遍。

*图：把"终端里管多个 AI 任务"这件事拆成 9 个维度对照——维度过多，图片版更清晰*

读完这张表，几个事实就摆出来了：

**1. Claude Code 的 Agent View 等于 auto-coder 的 `/async` ＋ `/bg` 合在一个面板里**。

`claude --bg "task"` —— fire-and-forget 后台跑，对应 auto-coder 的 `/async`。

`/bg`（在 interactive session 里把当前会话沉后台，注意这里 Claude Code 的 `/bg` 是 `/background` 的 alias ，不是新的）—— 对应 auto-coder 的 `/bg`。

`claude agents` —— 一个统一面板把两类后台 session 列在一起，可以在任意一行 attach 、 reply 、 peek 。

**2. supervisor + worktree 这套架构， auto-coder `/async` 一年前就跑通了**。

per-user 父进程托管多个 agent 子进程、每个 agent 写自己的 worktree 、状态元数据落盘 —— 这套设计在 auto-coder 里叫 `WorktreeManager` + `AsyncTaskService` + `TaskMetadataManager`， 2025-07-26 就在 PyPI 上发了。

**3. Claude Code 这次的整合做得漂亮**。

`/async` 跟 `/bg` 在 auto-coder 里是两条相互独立的命令家族，要写两份 `/list` 才能看全。 Claude Code 的 Agent View 用一个面板把"派出去的 worker"和"沉到后台的 interactive session"放在一起，键盘导航、 attach 、 reply 都在一个 view 里 —— 这个 UX 思路值得抄。

---

## 三、两条路的味道为什么不一样

`/async` 跟 `/bg` 在 auto-coder 里**故意**没合并 —— 因为它们对应的工作姿势真的不一样。

**`/async` 是"派工"姿势**：

我有一个明确的小任务——修一个 bug 、扫一遍代码、跑一个工作流——需求一句话写完，剩下的让 agent 自己干。我不打算跟它来回对话，干完看结果就行。 Worktree 隔离让它放手改文件不污染我手头的 checkout ，`/loop` 让它持续迭代，`/time` 给它一个时间盒。

群里 nightsailer 当时点出的味道：

> 那不就相当于 claude 的 /bg / 每个 agent 等于 claude --bg 的 cli 实例 / 有专门的 supervisord 来运行

我当时回的那句话现在看更准：

> 那这个其实更像 /async / 我们调用一个 /async 会启动一个新进程

Claude Code 的 `claude --bg "..."` 跟 auto-coder 的 `/async /name xxx "..."`，从用户视角到底层进程模型，几乎是同一件事。

**`/bg` 是"翻 tab"姿势**：

我正在跟 agent 来回对话——它给我出方案，我追问，它修改——这个对话上下文很重，不能丢。但前台跑了某个 5 分钟的长任务，我想去做点别的，又不想退出会话。`/bg` 就是这个场景：把当前会话原地沉到一个槽位，给我一个干净的新前台，回头 `/bg N` 切回去整个 TUI 重绘，对话上下文一字不差地回来。

> 我们的 bg 则是完全意义上的切换任务用的。切换后台任务，整个前台界面会重绘

Claude Code 的 `/bg` 也是这个场景的一种实现，区别在于它把 sessions 沉到 supervisor 而不是同进程槽位。结果是 Claude Code 的 `/bg` 抗终端意外退出， auto-coder 的 `/bg` 切换更瞬时**。没有谁更优，是不一样的取舍。**

---

## 四、群里早就把这件事点透了

5/12 上午 nightsailer 在群里转发 AI 寒武纪那篇介绍 Agent View 的文章后，几条对话其实已经把架构对照说清楚了：

💬 Auto-Coder-02 群对照分析

nightsailer:

每个 agent 等于 claude --bg 的 cli 实例 / 有专门的 supervisord 来运行

张义军:

opencode 早就是这种模式： tmux 或者 desktop 都是通过它 SSE 接入的

祝威廉:

那这个其实更像 /async / 我们调用一个 /async 会启动一个新进程

nightsailer:

就是统一管理你本机的所有后台运行 session ，可以是不同项目 / 我大概明白用法了 tmux 基本上可以不用了

群里几个人来来回回 5 分钟，自己就把"对面新出的东西约等于我们 `/async` ＋ `/bg`"这件事推导出来了。

---

## 五、为什么 auto-coder 一年前就把这条路走完了

老实回答：因为这条路对小团队来说**简单**。

把 agent 当成"派出去的 worker"，每个 worker 一个 git worktree 、一个子进程、一份元数据 —— 这是 2025 年中段任何做 code agent 的人都会画到白板上的设计。难的不是想到，而是**坚持把它当作主流形态做下去**。

auto-coder 那时候做这套，是被一个真实场景逼出来的：用户想让 agent 跑长任务，又不想守着前台。`/async` 上线之后还顺手补了 `/loop`（循环跑）、`/time`（限时跑）、`/workflow`（按预定义工作流跑），半年时间不停打磨。这些细节是任何"想到这条路但只做了 demo"的产品都补不上的。

到 2026-05-06 那天，需求换成了"在交互会话里也能 tab 切换"，于是出来了 `/bg` —— 复用了 `SessionRegistry` + `SessionSlot` 这套抽象，但因为是交互场景，不需要 worktree ，直接同进程 16 个槽位最快。

到 2026-05-11 ， Claude Code 推出 Agent View ，把 `--bg`（ fire-and-forget ）和 `/bg`（沉交互会话）整合到同一面板里 —— 这个**整合姿势**是 auto-coder 还没做的，值得学习。但底层那两条路的架构本身， auto-coder 早跑通了。

---

## 六、立刻上手这两条命令

```
pip install -U auto-coder  
cd your-project  
auto-coder.chat.lite
```

进入终端后，看完整帮助：

```
/async /help  
/bg /help
```

派 fire-and-forget 后台任务（更接近 Claude Code 的 `claude --bg`）：

```
/async /name fix_login "排查并修复登录页的 race condition"  
/async /list  
/async /task fix_login
```

挂多个交互会话（更接近"翻 tab"）：

```
/bg              # 把当前会话沉到后台  
/bg /list        # 看所有后台槽位  
/bg 2            # 把 2 号槽位拉回前台（双向交换）
```

💡 搭配建议

长跑型派给 `/async`，多线对话用 `/bg`，前台给自己留一手——三件事并行，没人守进度条。

---

## 结尾

终端里管多个 AI 任务这件事，过去一年其实有两类做法：

一类是 **fire-and-forget + worktree 隔离**， auto-coder 叫 `/async`（ 2025-07-26 ）， Claude Code 叫 `claude --bg`（ 2026-03-12 起逐步成熟）

一类是 **interactive 会话沉到后台再切回**， auto-coder 叫 `/bg`（ 2026-05-06 ）， Claude Code 叫 `/bg`（也是 `/background` 的 alias ）

Claude Code 在 2026-05-11 用 Agent View 把这两类整合在一个面板里，这是它的贡献。 auto-coder 在产品层面要补的功课，是把 `/async` 跟 `/bg` 也合并成一个统一视图 —— 已经在做了。

底层架构层面，`/async` 这条路 auto-coder 走完一年了。这件事值得说出来。

---

**延伸阅读**

`/bg` 详细设计：给 auto-coder.chat 加上 /bg[2]

Claude Code 官方 Agent View 文档：code.claude.com/docs/en/agent-view[3]

AI 寒武纪原版报道：[再也不用盯着几十个终端窗口](https://mp.weixin.qq.com/s?__biz=Mzg3MTkxMjYzOA==&mid=2247514719&idx=1&sn=5ef2a87b5d061edf8349a76a78cc4509&scene=21#wechat_redirect)

参考链接

[1] 给 auto-coder.chat 加上 /bg: https://zhuhailin.com/zh/blog/bg-multi-session

[2] 给 auto-coder.chat 加上 /bg: https://zhuhailin.com/zh/blog/bg-multi-session

[3] code.claude.com/docs/en/agent-view: https://code.claude.com/docs/en/agent-view