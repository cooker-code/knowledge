---
title: Git Worktree，让你的 AI 并行 Coding
author: AgenticCoding 实验室
date:
url: https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484559&idx=1&sn=bf83b5af590f24dcf3131779623addef&chksm=9a196d86d722611e8a3c9906944d8d3a3cd6377b9427fb6a390cc716d35e3c0a0e5cdb5b887e&mpshare=1&scene=24&srcid=1120jrlRtqpuPqaKnRqoCrGL&sharer_shareinfo=a48f9d804557742f97c25f984c024f52&sharer_shareinfo_first=a48f9d804557742f97c25f984c024f52#rd
---
> 已吸收至：[[09_电脑工具/0901_开发工具与CLI/090104_Git/090104_核心知识点/GitWorktree与AI并行开发边界|Git Worktree 与 AI 并行开发边界]]


# Git Worktree，让你的 AI 并行 Coding

最近这段时间 **Git Worktree** 被提及的频率有点高。简单的说 Git Worktree 允许你在同一个 Git 仓库中同时切出多个工作目录，适合在多个分支之间切换、并行开发和测试的场景。

但之前很少有人提及这个东西，毕竟人的精力就那么多，并发的场景并不多。但 AI 时代，每个人都想着尽可能压榨 AI（毕竟冲了钱），那 Git worktree 就成了适合 AI 并行开发的解决方案。

## Git Worktree 简介

通常我们的仓库中只有一个工作目录，对应一个 `.git` 目录，但总会遇到一些问题:

1. 1. 同时开发多个分支
2. 2. 一个分支跑测试，一个分支开发
3. 3. 构建和测试多个版本

这个时候 Git Worktree 就很有用，操作上也很简单：

添加一个新的 worktree：

```
git worktree add <path> <branch>
```

```
git worktree add ../new-feature feature/new-ui
```

这会在 `new-feature` 创建一个目录，你可以认为是将你 `feature/new-ui` 分支的内容在 `new-feature` 创建了一份。这样就和主 worktree 互不影响。你可以在两个目录中愉快的玩耍了。

## AI 并行开发

既然本质上是**两个目录**，那就可以在两个目录中同时开启 AI Code 任务，完成多功能的测试和开发。

开发完成后，和普通的分支操作一样，将功能 merge 到主 worktree 就行了。

worktree 使用完毕后，`git worktree remove ../my-project-hotfix` 移除 worktree。

也可以手动删除 worktree 目录后， `git worktree prune` 清理残留。

---

如果你是包月无限流量，那快去压榨你的 AI 员工吧😂

## 一些 Git worktree 资源

Claude Code 官方 worktree 指南：https://code.claude.com/docs/zh-CN/common-workflows#%E4%BD%BF%E7%94%A8-git-worktrees-%E8%BF%90%E8%A1%8C%E5%B9%B6%E8%A1%8C-claude-code-%E4%BC%9A%E8%AF%9D

git-worktree-runner，简化 Git Worktree 使用：https://github.com/coderabbitai/git-worktree-runner
