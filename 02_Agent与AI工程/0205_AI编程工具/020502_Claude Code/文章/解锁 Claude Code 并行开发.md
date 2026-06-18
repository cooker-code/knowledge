---
title: 解锁 Claude Code 并行开发
author: 六六柒
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI4Nzg5MzAyOA==&mid=2247484113&idx=1&sn=16d4aea36c6f9c7934fb0b84945d543e&chksm=ea78291e589a62500e3408bbc4ed733aee026e970455ae276821792e0a862176812fee20e418&mpshare=1&scene=24&srcid=0411iVIA22X4AFGqdtiiVZT7&sharer_shareinfo=444765f894d5b0d818eabb877f30bed4&sharer_shareinfo_first=444765f894d5b0d818eabb877f30bed4#rd
---

你有没有遇到过这种情况：

功能开发做到一半，线上突然来了紧急 bug。代码改得乱七八糟，既不能提交，又不想 stash。于是你花了十分钟收拾现场，切到 hotfix 分支修完，再切回来——思路全断了。

或者更糟：Claude Code 正在跑一个耗时的重构任务，你干坐着等，什么都做不了。

这两个问题的根源一样：**一个仓库同一时间只能处于一个状态**。

Git Worktree 就是用来解这道题的。它允许你把同一仓库的不同分支，同时检出到不同目录，每个目录是完全独立的工作空间。配合 Claude Code 的 `-w` 参数，并行开发可以做到真正的零干扰。

但真正用起来，问题才刚开始。

## 一、紧急需求来了，怎么操作

**不需要切换当前分支，不需要 stash。**

在原有终端保持不动的情况下，新开一个终端，cd 进入同一个项目目录，执行：

```
claude -w hotfix
```

Claude Code 会自动基于 master 创建一个独立 worktree，专门用于这个紧急任务。两个终端、两个独立环境，互不干扰。

**`-w` 参数帮你做了什么？**

手动创建 worktree 需要这样：

```
mkdir -p ../worktrees  
git worktree add ../worktrees/hotfix -b hotfix/login  
cd ../worktrees/hotfix  
claude
```

`claude -w hotfix` 把这四步压成一步，并自动将 worktree 存放在项目内的 `.claude/worktrees/hotfix`，不需要手动管理目录。

## 二、新的 worktree 基于哪个分支

默认基于 master/main，与你当前的开发分支完全隔离，**两者之间永远不会产生冲突**。

如果需要基于当前开发分支创建紧急工作区，手动指定：

```
claude -w hotfix-from-feature feature/login
```

冲突只会在合并回 master 时发生，和并行开发本身无关。

## 三、代码存在哪里，安全吗

`claude -w` 创建的 worktree 存放在：

```
你的项目/.claude/worktrees/xxx
```

`.claude` 文件夹默认被 Git 忽略，不会污染主项目结构。但这只是目录本身被忽略——**worktree 内的代码可以正常提交、推送**，走的是独立分支，和主项目代码没有任何关联。

在新终端里，直接用常规 Git 命令提交即可：

```
git add .  
git commit -m "fix: login timeout"  
git push origin hotfix/login
```

手动用 `git worktree add ../worktrees/xxx` 创建的 worktree 会放在平行目录，和 `claude -w` 仅存放位置不同，功能完全一致。

## 四、在编辑器里查看改动，打开哪个目录

这里有个容易踩的坑：**必须打开 `.claude/worktrees/xxx` 这个目录，不要打开主项目根目录。**

打开对应工作区目录，编辑器才能正确识别当前分支，只显示该 worktree 的代码改动。打开主项目根目录会导致 Git 状态混乱，容易误操作。

更安全的方式是直接在终端用命令确认：

```
git status  
git diff
```

## 五、`.mcp.json` 等配置文件，需要每个 worktree 重复配置吗

**不需要。所有 worktree 自动共享主项目的配置文件。**

Git Worktree 隔离的本质是：**只隔离被 Git 追踪的代码文件**（已执行 `git add`/`git commit` 的文件），与 `.gitignore` 无关。

`.mcp.json`、`.env` 等未被 Git 追踪的本地配置，只存在于主项目根目录，所有 worktree 启动时都会自动读取，不需要复制，也不会泄露密钥。

## 六、不同 worktree 里的项目记忆，Claude 会混吗

**不会混，会自动归集到同一个项目。**

Claude Code 识别项目的依据是主项目根目录，而不是 worktree。无论开多少个 `claude -w` 工作区，只要在同一个主项目下启动，所有的上下文记忆、代码规范、文档都统一归集、共用。

你不需要在每个 worktree 里重复教 Claude 规则，也不需要重复上传文档。

## 七、项目专属配置如何同步给所有 worktree

在某个 worktree 里总结的项目专属 Skill 或配置，想让整个项目的所有 worktree 都能用，方法是提交到 master：

```
# 强制将 .claude 文件夹加入 Git 追踪（默认被忽略）  
git add -f .claude/  
  
git commit -m "build: 添加项目专属 Claude 配置"  
git push
```

后续更新配置，直接在 master 分支修改 `.claude` 下的内容，提交推送后，所有新开的 worktree 自动继承。

## 八、主分支该怎么用

**master 只用来同步，不直接开发。**

```
# master 只做这一件事  
git checkout main && git pull
```

所有开发任务——新特性、紧急修复——都通过 `claude -w 任务名` 创建独立工作区来做。这样主分支永远干净，每个任务完全隔离，切换零成本。

## 一张图记住核心原则

```
主分支（master）  
    │  
    ├── 只做 git pull，永不直接开发  
    │  
    ├── claude -w feature-auth   → .claude/worktrees/feature-auth  
    ├── claude -w hotfix-login    → .claude/worktrees/hotfix-login  
    └── claude -w review-pr123   → .claude/worktrees/review-pr123  
  
三个 worktree，三个终端，三个独立 Claude 会话
```

回到最开头的场景：紧急 bug 来了，你不需要收拾现场。新开一个终端，`claude -w hotfix`，30秒内进入修复状态。原来的任务一行代码没动，Claude Code 的上下文完好无损，配置共享，记忆归集，代码完全隔离。

并行开发应有的样子，大概就是这样。

（本文完）