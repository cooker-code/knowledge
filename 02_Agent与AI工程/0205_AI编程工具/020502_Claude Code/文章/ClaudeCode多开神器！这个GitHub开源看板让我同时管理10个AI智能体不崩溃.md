---
title: ClaudeCode多开神器！这个GitHub开源看板让我同时管理10个AI智能体不崩溃
author: GHub开源精选库
date: 小果小果
url: https://mp.weixin.qq.com/s?__biz=MzkwOTY1MTI3Mw==&mid=2247488485&idx=1&sn=49af124ebf58c70728ad7e7c1e3f8b9c&chksm=c078df433b3873b2c6ae72cbc2513504a0b2ce9cb673c5ffddd0f90484d595972bef831d8c90&mpshare=1&scene=24&srcid=04290ASoQzSccSihBKC9DC7J&sharer_shareinfo=0563c90909bd588f46d8d3f38bf1a27e&sharer_shareinfo_first=0563c90909bd588f46d8d3f38bf1a27e#rd
---

---

还在为Claude Code多任务混乱头疼吗？我自己就经历过那种绝望——五个终端窗口来回切换，分不清哪个在跑前端，哪个在调试后端，还有一个卡在了权限提示上半天没动静。

最近逛GitHub时发现了个宝藏项目，简直是给Claude Code量身定做的"IDE for 2026"。

kanban-code GitHub仓库

这个项目叫**Kanban Code**，是LangWatch团队开源的看板工具。它不是那种简单的任务列表，而是直接把Claude Code会话、git worktree、tmux终端和GitHub PR全部串联起来的"指挥中心"。

想象一下，每个AI任务都是一张卡片，从待办到完成自动流转，还能在手机收到推送通知——这才是真正的AI代理编排该有的样子。

**先看它能干啥**

最爽的是那个**智能看板**，有六个自动流转的列：Backlog（积压任务）、In Progress（Claude正在干活）、Waiting（卡住了等你救场）、In Review（PR已开）、Done（已合并）、还有All Sessions（历史存档）。Claude一开始工作，卡片自动跳到进行中；遇到权限提示或者停下来思考，卡片就滑到Waiting列，同时给你的手机发Pushover通知。不用一直盯着终端，去泡杯咖啡回来就知道哪些Agent需要你。

每个任务都跑在独立的**tmux会话**里，这意味着你随时能接管控制权。更绝的是内置了SwiftTerm终端模拟器，直接在卡片里看Claude干活，真彩色、Unicode、鼠标事件全支持。 ten个Agent同时在五个项目里跑，每个的终端输出都一目了然，再也不会搞混哪个dev server属于哪个分支。

还有**Git Worktree集成**——开新任务时自动创建独立分支，完成后自动清理。配合GitHub CLI，PR状态、CI检查结果、代码审查意见都实时显示在卡片上。甚至可以配置远程执行，把重活扔给服务器，本地只负责看板管理，笔记本风扇再也不狂转了。

**安装其实挺简单**

macOS用户直接去Releases页面下载`.app`文件，拖到Applications就行。因为是开源项目没做公证，第一次打开可能会提示拦截，右键选"打开"或者去系统设置里点"仍要打开"就好。需要macOS 26 (Tahoe)和Claude Code CLI已经装好。

想从源码编译也很方便：

```
git clone https://github.com/langwatch/kanban-code.git  
cd kanban-code  
make run-app
```

Windows用户也不落下，用Tauri框架实现的版本需要Node.js(v18+)、Rust和Claude Code CLI：

```
git clone https://github.com/langwatch/kanban-code.git  
cd kanban-code/windows  
npm install  
npm run tauri dev  # 开发模式  
# 或者打包成exe  
npm run tauri build
```

**上手步骤一步步来**

第一次启动时，Kanban Code会自动扫描`~/.claude/projects/`目录，把你之前的所有会话都找出来，显示在All Sessions列里。

接着去Settings添加项目路径，比如`~/Projects/my-app`。如果是Git仓库且配好了`gh` CLI，它会自动拉取GitHub Issues填充Backlog列。然后装上Claude Code hooks，这样Kanban Code才能实时感知Claude的状态变化——什么时候开始干活、什么时候停下来、什么时候需要用户输入，全靠这些hooks上报。

创建新任务时，点卡片上的Start，Kanban Code会自动：创建git worktree（可选）、启动tmux会话、运行Claude Code、打开嵌入式终端。当然你也可以继续在终端里用`claude`命令启动，Kanban Code会自动发现并创建对应的卡片。

**几个贴心的小细节**

* **BM25全文搜索**：找历史会话特别快，支持按提示词、对话内容或项目名搜索
* **Fork和Checkpoint**：比Claude Code自带的更快，直接操作会话文件，能分叉对话或回滚到任意点
* **防睡眠模式**：和Amphetamine集成，Agent干活时Mac不会自动休眠
* **键盘优先**：⌘K搜索、⌘N新建任务，支持拖拽和右键菜单

用了几天下来，最大的感受是**上下文切换成本**大幅降低。以前同时跑多个Claude任务简直是灾难，现在每个任务都有独立的worktree、终端和PR跟踪，像指挥交响乐团一样管理AI智能体。而且它是AGPLv3开源协议，意味着你可以自由修改和分发。

如果你也是Claude Code的重度用户，或者正在尝试多Agent并行开发，强烈建议试试这个。比起在十个终端窗口里迷失，这种可视化的管理方式真的能让生产力上一个台阶。

项目地址在这里：

https://github.com/langwatch/kanban-code

赶紧去点个Star，让作者知道我们做用户的有多需要这种工具！

专注分享 GitHub知识，分享AI 资讯和AI搞米经验，分享OpenClaw使用经验。

想**领取完整版OpenClaw资料**，围观朋友圈，一起交流AI的，可加我VX，备注“gith**ub**"。