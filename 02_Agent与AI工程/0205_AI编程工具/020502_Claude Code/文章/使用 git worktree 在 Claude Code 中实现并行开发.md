---
title: 使用 git worktree 在 Claude Code 中实现并行开发
author: 闲不住的李先森
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzODYyNzM0OA==&mid=2247484340&idx=1&sn=fc8f652333c4cd289a2c22b8dbb19b86&chksm=e8ad833dbc54934c4dc050cb9471d6ade111f72516dad25c77fab8f83e522e3c4f68db1d00c2&mpshare=1&scene=24&srcid=02048OV9UoipRD3qgX4mWmqW&sharer_shareinfo=7253d3631d981f97611f6e2c90eec5e7&sharer_shareinfo_first=7253d3631d981f97611f6e2c90eec5e7#rd
---

Claude Code 是一个强大的 AI 编程助手，如果我们单线程的等待 Claude Code 完成任务， 那么我们可能需要等待很长时间。 但是， 如果我们能够并行的处理多个任务， 那么我们就可以大大提高工作效率。

我们可以打开多个 Claude Code 终端窗口， 每个终端窗口都对应一个工作区，这样可以实现简单的并行开发。

然而在多终端窗口并行开发时上下文管理和交叉引用管理是一个非常复杂的问题：

以下面的场景为例：

当我们在使用 Claude Code 给`电商项目`开发购物车功能时，突然来了一个商品模块的线上bug， 我们不得不在 `stash` 或者直接 `push` 购物车功能的代码，然后基于 main 分支创建一个 `hotfix` 分支，使用 Claude Code 给`电商项目`开发商品模块的修复功能。

在进行分支切换开发的过程中， 创建购物车的上下文可能会被 hotfix 分支的上下文`污染`， 甚至会因为 hotfix 的上下文过多，导致购物车的上下文被压缩失真而被覆盖。

使用两个Claude Code 终端窗口也无法规避上面的问题，因为多窗口所使用的上下文是相同的。

我们可以使用 `git worktree` 来实现 `多终端窗口对相同项目并行开发时， 保持不同的上下文`， 这样我们就可以在不打断开发购物车功能时， 同时新开一个终端窗口和新的上下文， 进行商品模块的修复工作。

## 什么是 git worktree

git worktree

它允许你在同一仓库中创建多个工作目录，每个目录绑定独立分支。相当于给你的代码库开了多个“平行空间”：

```
/home/user/demo   
├── hotfix/         # 热修复工作区    
├── feature-A/      # 新功能开发区  
├── ...             # 主工作区（如 feature-cart 正在开发的购物车分支） 
```

如上面的结构，hotfix 和 feature-A 目录是 git worktree 创建的工作区， 其中 hotfix 工作区是基于 main 分支创建的， 它用来修复商品模块的线上bug。 feature-A 工作区是基于 dev 分支创建的， 它用来开发商品模块的新功能。而主目录是我们正在开发的购物车功能工作区， 分支是 feature-cart。

hotfix 和 feature-A 工作区下各自维护一份项目代码， 这样就达成了各自拥有独立的上下文的能力， 另外各工作区又共享同一个 git 仓库（Git 历史）， 保证我们可以进行代码的git 操作。

### 如何使用 git worktree

**「1. 创建工作区目录」**

使用 git worktree 我们需要先创建工作区目录。创建 git worktree 的核心命令是 `git worktree add <工作区路径> <分支>`。

以上面的项目结构为例， 我们可以在主工作区下创建 hotfix 和 feature-A 工作区， 命令如下：

```
git worktree add -b hotfix ./hotfix main  # 从 main 创建 hotfix 分支， 分支放到 ../hotfix 目录下  
git worktree add -b feature-A ./feature-A origin/dev  # 基于远程 dev 分支， 分支放到 ../feature-A 目录下
```

`-b XX` 参数用于指定创建分支的名称， 如果不指定， 则默认使用工作区路径的最后一个组成部分。

**「2. 在工作区中进行开发和提交」**

在工作区中进行开发和提交代码， 和在主工作区中进行开发和提交代码一样， 只是需要切换到对应的工作区目录下。

```
cd ../hotfix  
## 进行开发  
git add .  
git commit -m "修复商品模块的线上bug"
```

**「3. 合并工作区代码到主工作区」**

对于已经完成开发的工作区， 我们可以将工作区代码合并到主工作区。执行合并工作区分支，分为两种情况：

情况一： 主目录和 worktree 基于同一分支

如果你的主目录也工作在 main（或 master）分支，而 worktree 正是从 main分支创建的用于修复 bug，那么你可以在主目录中直接使用 git pull。git pull这个命令相当于 git fetch和 git merge两步操作的结合

```
# 在主工作目录下，确保当前分支是 main  
git checkout main  
git pull origin main
```

情况二： 主目录和 worktree 在不同分支，你需要合并修改

更常见的场景是，主目录在 dev分支开发新功能，而 worktree 在 main分支上修复 bug。现在你需要将 main分支上的修复合并到 dev分支。

```
# 在主工作目录下操作  
git checkout dev # 确保切换到需要合并修改的分支，如 dev  
git merge origin/main # 将远程 main 分支的修改合并到当前所在的 dev 分支
```

**「4. 删除工作区（可选）」**

由于工作区中维护了完整的项目代码， 为了减少项目的磁盘占用， 在提交代码后我们可以删除工作区。

* 只有干净的工作区（没有未跟踪的文件和未修改的跟踪文件）可以被删除。脏工作区或有子模块的工作区可以用 --force 删除。主工作区不能被删除。
* 删除工作区只删除工作区目录， 不会删除工作区分支。

```
git worktree remove ./hotfix
```

了解了 git worktree 的基本使用， 我们来看一下如何使用 git worktree 在 Claude Code 中实现并行开发。

## 使用 git worktree 在 Claude Code 中实现并行开发

### 场景一： 开发购物车功能时，突然来了一个商品模块的线上bug

使用上面的开发场景（开发购物车功能时，突然来了一个商品模块的线上bug）为例：

首先我们打开了一个 Claude Code 终端窗口， 进行购物车功能的开发。（可以把终端窗口命名为 `购物车功能`）

这时我们突然来了一个商品模块的线上bug。我们不需要对购物车做任何 git 操作， 只需要新建一个终端窗口, 并创建一个新的 worktree 工作区， 并进入到工作区目录下， 然后进行商品模块的修复工作。 （ 记得把终端窗口命名为 `商品模块bug修复`， 在多窗口时， 方便区分）

```
# 在根目录下创建 hotfix 目录， 存放  hotfix/goods-price-fix 分支的代码  
git worktree add -b hotfix/goods-price-fix ./hotfix origin/main  
cd ./hotfix  
  
# 工作区是干净的项目代码， 如果需要运行项目，需要先安装 项目依赖  
npm install
```

在 `商品模块bug修复` 终端窗口中， 我们进行商品模块的修复工作。

```
# 使用 claude code 修复商品模块的线上bug  
  
# ... 省略 claude code 的输出和交互 ...  
  
git add .  
git commit -m "修复商品模块的线上bug"
```

修复完成后， 我们把工作区代码合并到主工作区（购物车功能工作区）继续开发购物车功能。

```
# 在 hotfix 工作区下， 切换到主工作区, 这时主工作区是 feature-cart 分支  
cd ..   
git merge hotfix/goods-price-fix
```

这样我们就完成了商品模块的线上bug修复， 并且没有影响到购物车功能在 claude code 中的上下文。

### 场景二： 并行开发功能

除了处理可能被突然打断的紧急任务， 我们还可以使用 Claude Code 进行并行开发功能。

例如：我们当前迭代中包含三个功能： 购物车功能、商品模块功能、订单模块功能。 我们可以使用 git worktree 在 Claude Code 中并行开发这三个功能。

这时我们可以把主目录分支设置为 dev 分支， 然后创建三个工作区， 分别用于开发购物车功能、商品模块功能、订单模块功能。

```
git checkout -b dev  
git worktree add -b feature-cart ./features/feature-cart origin/dev  
git worktree add -b feature-goods ./features/feature-goods origin/dev  
git worktree add -b feature-order ./features/feature-order origin/dev
```

打开三个终端窗口， 并分别启动 Claude Code, 用于开发购物车功能、商品模块功能、订单模块功能。

我们还可以分别为每个终端窗口设置不同的名称， 用来区分不同的功能。

当任务完成时， 我们可以选择当功能分支合并到 dev 分支。

```
git checkout dev  
git merge feature-cart  
git merge feature-goods  
git merge feature-order
```

这样我们就完成了三个功能的并行开发， 并且在 claude code 中， 上下文是隔离的， 对于有冲突的内容我们可以在合并时进行手动处理。

### 其他场景

* 相同功能使用不同模型对比实现效果， 选择最优的实现方案
* 针对每个功能， 使用不同的方案实现， 对比实现效果， 选择最优的实现方案
* 项目重构： 在孤立工作区中测试大规模重构， 避免影响主工作区
* 增强型代码审查：使用agent 对代码进行审查， 不设计主工作区
* 并行环境测试：在多个工作区中同时运行多个 Python 、 Node.js 、 Java 等环境， 测试不同版本的兼容性
* 安全测试主要依赖升级

## 总结

Claude 代码上下文依赖于： 文件系统、Git 历史、依赖理解、对话记忆 等， 借助于 git worktree 提供的并行工作区的能力， 可以让 Claude Code 在保持各自的上下文隔离的基础上并行做不同的事情，从而提高开发效率。

---

往期系列文章：

[Claude code 国内入门使用指南](https://mp.weixin.qq.com/s?__biz=MzIzODYyNzM0OA==&mid=2247484239&idx=1&sn=a7f7d56cc9cec63651cb354dcf3ebc2a&scene=21#wechat_redirect)

[Claude code 使用 API 中转原理解析及opusplan 智能模型配置](https://mp.weixin.qq.com/s?__biz=MzIzODYyNzM0OA==&mid=2247484249&idx=1&sn=039f7823207ac05e9e9e66aaf172033a&scene=21#wechat_redirect)

[cc-switch 与 claude-code-router（CCR） 深度横评，`省钱`和`智能`既要又要](https://mp.weixin.qq.com/s?__biz=MzIzODYyNzM0OA==&mid=2247484265&idx=1&sn=d97a6bec856ecf72e5b1223bf2caec9a&scene=21#wechat_redirect)

[如何为 AI Agents 写一份好规格说明（Spec）](https://mp.weixin.qq.com/s?__biz=MzIzODYyNzM0OA==&mid=2247484320&idx=1&sn=aeaa759a1a0d0967856441d16e45a7ac&scene=21#wechat_redirect)

---

公众号会持续输出，欢迎关注。 如果对你有帮助，欢迎点赞、收藏、关注。