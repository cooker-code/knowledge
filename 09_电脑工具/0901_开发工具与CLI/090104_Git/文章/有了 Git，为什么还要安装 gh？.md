---
title: 有了 Git，为什么还要安装 gh？
author: 方圆Talk
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU4Mjk2MjA3MQ==&mid=2247484384&idx=1&sn=97f64f04dc39363e5a6e6e1b9d9093a5&chksm=fc1d37b7416f899890e433889c5e41b34239cd3b3fe83880c05c6758b13776b1121898d89490&mpshare=1&scene=24&srcid=0412LR1oFiv2DB1gveqbTyJD&sharer_shareinfo=0d869b36f3f7c9bbdce9e1bd9c4bdf3d&sharer_shareinfo_first=0d869b36f3f7c9bbdce9e1bd9c4bdf3d#rd
---

---

你大概率很熟悉这套节奏：

`git add`、`git commit`、`git push`。

代码推上去之后，你会下意识切到浏览器，去做另外一串动作：开 Pull Request、看 CI、回评论、查 Issue、Fork 仓库、翻 Release。

问题也正出在这里。

**我们明明已经在终端里写代码了，为什么一到 GitHub 协作环节，就又被拽回网页？**

这就是 `gh` 存在的意义。

**一句话先说结论： 负责版本控制， 负责 GitHub 协作。**

## `gh` 是什么？

`gh` 是 GitHub 官方推出的命令行工具，也就是 GitHub CLI。

它不是另一个 `git`，而是把 GitHub 上那些原本要在网页里做的事情，直接搬到了终端里。比如：

* • 查看、创建、合并 Pull Request
* • 查看、创建 Issue
* • 创建、克隆、Fork 仓库
* • 查看 GitHub Actions 运行状态
* • 管理 Release、Gist
* • 调用 GitHub API

所以，理解 `gh` 最简单的方法不是把它看成“Git 增强版”，而是把它看成：

> **一个懂 GitHub 平台概念的命令行工具。**

它理解的不是“提交对象”和“索引区”，而是 `PR`、`Issue`、`Actions`、`Repo`、`Release` 这些 GitHub 工作流里的实体。

## 有了 `git`，为什么还要安装 `gh`？

因为 `git` 很强，但它解决的是“代码怎么被管理”，而不是“代码怎么在 GitHub 上协作”。

`git` 的核心能力是：

* • 初始化仓库
* • 提交变更
* • 分支管理
* • 合并、变基、回滚
* • 与远程仓库同步

这套能力已经足够支撑版本控制了，但它并不天然理解 Pull Request、Issue、Review、CI、Fork 这些 GitHub 平台层的概念。

而现代团队开发里，真正频繁发生的，往往不是“再执行一次 `git status`”，而是下面这些动作：

* • 我这个分支的 PR 发出去了吗？
* • CI 过了没有？
* • 这个 Issue 现在归谁？
* • 我能不能直接在终端里切到同事的 PR？
* • 我想 Fork 一个开源项目，能不能别再复制仓库地址了？

这时候，`gh` 的价值就出来了。

### `git` 和 `gh` 的根本区别

你可以把它们理解成两层：

* • **是底层**：管理代码历史、分支、提交和远程同步
* • **是上层**：操作 GitHub 平台上的协作对象和流程

它们不是替代关系，而是协作关系。

最理想的工作方式往往是这样的：

1. 1. 用 `git` 管理本地代码变更
2. 2. 用 `git push` 把分支推上远程
3. 3. 用 `gh` 发 PR、看检查结果、处理协作流程

也就是说：

**管代码流， 管协作流。**

## `git` 和 `gh` 最常用命令对照表

| 场景 | `git` 常用命令 | `gh` 常用命令 | 说明 |
| --- | --- | --- | --- |
| 初始化仓库 | `git init` | `gh repo create` | `git` 建本地仓库，`gh` 可直接创建 GitHub 仓库 |
| 克隆仓库 | `git clone <url>` | `gh repo clone owner/repo` | `gh` 直接使用 `owner/repo` |
| 查看本地状态 | `git status` | `gh status` | `git` 看工作区，`gh` 看 GitHub 上与你相关的工作状态 |
| 新建分支 | `git checkout -b feat/x` | 无直接对应 | 分支管理仍然属于 Git 的职责 |
| 推送代码 | `git push origin feat/x` | 无直接对应 | 代码同步靠 `git` |
| 创建 PR | 无直接对应 | `gh pr create --fill` | PR 是 GitHub 平台概念 |
| 查看当前 PR 状态 | 无直接对应 | `gh pr status` | 适合快速查看你创建的、分配给你的、等待处理的 PR |
| 查看某个 PR | 无直接对应 | `gh pr view 123` | 直接在终端查看 PR 信息 |
| 切到某个 PR | `git fetch` + `git checkout` | `gh pr checkout 123` | `gh` 自动完成拉取与切换 |
| 查看 Issue | 无直接对应 | `gh issue list` | Issue 属于 GitHub 平台层 |
| 创建 Issue | 无直接对应 | `gh issue create` | 终端里直接建 Issue |
| 查看 CI / Actions | 无直接对应 | `gh run list` / `gh run watch` | `git` 不管理 CI，`gh` 可以 |
| Fork 仓库 | 无直接对应 | `gh repo fork owner/repo --clone` | 适合开源协作 |
| 调 GitHub API | `curl` | `gh api` | `gh api` 自带 GitHub 认证上下文 |

这张表背后的本质很简单：

**凡是代码版本控制相关的事情，交给 ；凡是 GitHub 协作平台相关的事情，交给 。**

## `gh` 使用教程：一套最实用的上手路径

如果你已经会 `git`，那学习 `gh` 最好的方式不是通读文档，而是先掌握一套最常用的工作流。

### 1. 安装 `gh`

```
# macOS  
brew install gh  
  
# Windows  
winget install --id GitHub.cli
```

Linux 发行版较多，最稳妥的方式是看 GitHub CLI 官方安装说明。

### 2. 登录 GitHub

```
gh auth login  
gh auth status
```

登录过程中通常会让你选择：

* • 登录 GitHub.com 还是企业版实例
* • 使用 `HTTPS` 还是 `SSH`
* • 是否让 `gh` 帮你处理 Git 凭据

如果你平时就是 HTTPS 工作流，这一步通常能顺手把后续 `git push` 的认证体验也梳理好。

### 3. 先记住这几个高频命令

```
gh repo view  
gh pr status  
gh issue list --assignee "@me"  
gh run watch  
gh browse
```

它们分别对应五个高频动作：

* • 看一眼当前仓库
* • 看一眼当前 PR 状态
* • 看一眼分配给自己的 Issue
* • 盯住 CI 跑完没有
* • 需要回网页时，从终端直接打开对应页面

### 4. 日常开发里怎么把 `git` 和 `gh` 串起来？

下面这套命令，就是很多团队里最顺手的日常流：

```
git checkout -b feat/login  
git add .  
git commit -m "feat: add login"  
git push -u origin feat/login  
  
gh pr create --fill  
gh pr checks --watch
```

这套流程的分工非常清晰：

* • 用 `git` 管代码和分支
* • 用 `gh` 发 PR、看检查、继续协作

一旦你习惯这套方式，就会明显减少“写着代码突然切去浏览器点半天”的中断感。

### 5. 几个特别值回票价的命令

查看某个 PR：

```
gh pr view 123
```

切到某个 PR 本地调试：

```
gh pr checkout 123
```

创建仓库并顺手克隆下来：

```
gh repo create my-project --public --clone
```

Fork 一个开源项目并克隆到本地：

```
gh repo fork owner/repo --clone
```

给常用命令起别名：

```
gh alias set pv 'pr view'  
gh pv 123
```

直接调用 GitHub API：

```
gh api repos/{owner}/{repo}/releases
```

## 谁最应该尽快装上 `gh`？

如果你只是偶尔把代码推到远程备份，`git` 可能已经够用。

但下面这三类人，装上 `gh` 基本都会很快感受到效率差异：

* • **经常提 PR 的团队开发者**
* • **经常做开源贡献的开发者**
* • **希望把终端工作流串得更顺的人**

因为他们每天真正消耗时间的，不只是提交代码，而是那些散落在浏览器里的协作动作。

## 写在最后

回到标题里的问题：

**有了 ，为什么还要安装 ？**

因为 `git` 解决的是版本控制，`gh` 解决的是 GitHub 协作。

前者让你能管理代码历史，后者让你能在终端里完成 PR、Issue、Actions、Fork、Repo 这些原本必须切去网页才能完成的事。

它们不是二选一。

真正高效的工作流，恰恰是：

**写代码用 ，做协作用 。**

当你把这两者拼在一起，终端才会从“写代码的地方”，真正变成“完成整条开发链路的地方”。

## 参考资料

* • GitHub CLI 官方文档：https://docs.github.com/github-cli
* • GitHub CLI Manual：https://cli.github.com/manual/
* • GitHub CLI 仓库：https://github.com/cli/cli
* • Git 官方站点：https://git-scm.com/