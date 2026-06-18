---
title: 我把 worktree 做成了一个顺手的 CLI
author: 何余生
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzOTY3MTExMQ==&mid=2247483807&idx=1&sn=9787209e9cde4ce4d4f76631290de70a&chksm=e8f754a8cb6b8615c38fa97185a4677b8950ce17a2a6420fdbe24c632a7c946efdf584684532&mpshare=1&scene=24&srcid=0203AwKx38FKKSPcYCxWrT9o&sharer_shareinfo=4d7b5eb17ed1e41d7d2aed3e86a4f79d&sharer_shareinfo_first=4d7b5eb17ed1e41d7d2aed3e86a4f79d#rd
---

# gmc 你的下一个 Git 助手

把自己常用的几个 Git 操作封装成了一个 CLI 工具，叫 `gmc`（Git Message Commit）。

项目地址：https://github.com/samzong/gmc

核心能力三个：

* • `gmc`：读 staged diff，用 LLM 生成 commit 提交信息
* • `gmc wt`：提供基于 .bare + worktree 的多工作区方案
* • `gmc tag`：规则 + 大模型，给出合理的语义化版本号建议

## 自动生成 Commit Message

这是 `gmc` 最早做的功能。读 staged diff，调 LLM 生成 Conventional Commits 风格的提交信息，给个交互确认 `y/n/r/e`。

```
# 标准用法  
git add -A  
gmc  
  
# 懒人模式，自动 git add  
gmc -a  
  
# 只提交指定文件  
gmc -a path/to/file1 path/to/file2  
  
# 追加 issue 引用  
gmc --issue 123  
  
# 临时加约束  
gmc --prompt "Focus on user-visible behavior"  
  
# 只生成不提交  
gmc --dry-run
```

还有个 stdin 模式：

```
git diff | gmc -
```

可以把 `gmc` 当成 diff → commit message 的转换器。

其实，还有少有趣的功能，比如自动 sign-off, Emoji 等， 有兴趣可以研究下。

今天重点想分享的是 git worktree 部分的能力。

## worktree 管理：gmc wt

在用 Codex、Claude 写代码之后，真切的感受是工程师的并行能力被极大加速。  
之前没太在意的多工作区的价值就体现出来；同时多任务并行，同一个任务多个大模型处理。

但只用 `git checkout` 切分支很快就乱了：

* • A 分支跑到一半，B 分支也想试试，但 A 还没提交
* • 两边都要跑测试起服务，一直来回切
* • 想并行推进，却被一个工作目录绑死了

Git 有 `git worktree` 解决这个问题：同一个仓库可以同时有多个目录，每个目录对应一个分支，互不干扰。  
但原生命令有点烦，在手工使用 git worktree 2 个月之后，我给 gmc 加上了 `gmc wt`。

### 2.1 把仓库 clone 成 `.bare + worktree` 结构

经过一段时间的摸索，我主要使用 worktree 的形式是 `.bare` 模式：仓库本体是 bare repo，每个分支对应一个 worktree 目录。

所以 gmc 也完全遵循了裸仓库的最佳实践。

```
gmc wt clone https://github.com/user/repo.git --name my-project
```

会创建 `.bare/` 目录和默认分支的工作区（比如 `main/`），后续就可以随便开分支开目录。

### 2.2 开源贡献场景：clone + upstream 一步到位

给开源项目提 PR（fork + upstream）最烦的一步：clone 完 fork 还要手动加 upstream remote。

`gmc wt clone` 支持 `--upstream`：

```
gmc wt clone https://github.com/me/my-fork.git \  
  --upstream https://github.com/org/upstream-repo.git \  
  --name upstream-repo
```

后续在默认分支工作区里同步 upstream，再开新 feature worktree 就行。

### 2.3 一键多开工作区：gmc wt dup

git worktree 有一个限制，一个分支只能绑定一个工作区；  
在使用大模型开发时，我希望针对一个问题同时跑多个实现、挑一个最好的保留：

```
gmc wt dup 3 -b main
```

创建 3 个临时 worktree，每个有自己的临时分支，在不同目录里同时推进，互不影响。

### 2.4 把临时分支扶正：gmc wt promote

评估完之后，把最好的那套提升为正式分支：

```
gmc wt promote .dup-1 feature/best-solution
```

本质就是重命名临时分支，把一次并行实验变成可维护的分支。

### 2.5 其他常用命令

```
gmc wt ls  
gmc wt add feature-login -b main  
gmc wt rm feature-login  
gmc wt rm feature-login -D
```

## 版本号建议：gmc tag

做这个功能的原因：我不喜欢那种"只要有一个 feat 就直接升 minor"的策略。一个 feat 可能就是加个可选参数、补个小能力。

`gmc tag` 的思路：

1. 1. 先用规则引擎给一个保守且可解释的版本建议
2. 2. 如果配了 API Key，再问大模型哪个版本更合适
3. 3. LLM 结果会校验，有兜底，不会出现版本倒退

```
gmc tag  
gmc tag --yes
```

## 安装

### Homebrew

```
brew tap samzong/tap  
brew install gmc
```

### 初始化配置

提交信息生成需要配置 API Key：

```
gmc init
```

或者手动：

```
gmc config set apikey YOUR_OPENAI_API_KEY  
gmc config set apibase https://your-proxy-domain.com/v1  
gmc config set model gpt-5.2
```

几个细节：

* • macOS/Linux 上配置文件会被设为 `0600` 权限
* • 支持在仓库里放 `.gmc.yaml` 做项目级覆盖

## 写在最后

`gmc` 最初是为了解决烦人的生成 commit message； 后来增加了 worktree 的能力，基本算是一个比较稳定的状态。