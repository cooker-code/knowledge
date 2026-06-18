---
title: Harness 工程 Skill：使用 Entrix 技能开始你的代码熵治理
author: phodal
date: 
url: https://mp.weixin.qq.com/s?__biz=MjM5Mjg4NDMwMA==&mid=2652980357&idx=1&sn=96ab04cc1264079cb5751e27c288d8d2&chksm=bc9a576fb1b0d98b33ffa00cc609efdbe9bf5bbb355375f2f6f6d276c86cc05705ac57dfdfd5&mpshare=1&scene=24&srcid=0424RLzxYKFwr6HFI2qzHRO3&sharer_shareinfo=85c323ae9c5b250b8329fc4bd759de33&sharer_shareinfo_first=85c323ae9c5b250b8329fc4bd759de33#rd
---

过去我们关心的是代码写得够不够快、自动化够不够多，而现在，越来越多团队首先要回答的是另一个问题：当生成速度不断提高之后，系统靠什么抵抗持续上升的代码熵。 在 Routa 的持续实践里，我们逐步把这类经验沉淀成了 Entrix。

项目地址：https://github.com/phodal/entrix

Entrix 不是一个单纯跑检查脚本的工具，也不是把已有 lint 和 test 再包装一层，而是试图把仓库中原本分散的质量规则、完成条件和升级路径，收敛为一套可以执行、可以理解、也可以被可视化的 Harness Engineering 结构。

欢迎入群讨论：

## 安装和使用 Entrix

Entrix 的接入方式很轻，这一点很重要。因为治理能力只有足够轻量，才能先进入日常工作流，再逐渐长成工程能力，而不是停留在一篇理念文章里。当前既可以把它作为 Claude Code Plugin 安装，也可以直接作为本地 CLI 使用：

```
1. /plugin marketplace add phodal/entrix
2. /plugin install entrix@entrix
```

或者

```
2. uv tool install entrix
3. # 或
4. pip install entrix

6. entrix run --tier fast
```

项目地址：https://github.com/phodal/entrix

安装之后，Entrix 真正提供的，并不只是“多跑几条命令”。它更像是在把仓库自己的完成条件重新拉回到交付过程里。

Skill 目录如下：

像 `docs/fitness/*.md`、 `manifest.yaml`、 `review-triggers.yaml` 这样的结构，表达的都不是单点检查，而是一个仓库如何定义质量维度、如何组织证据，以及在什么情况下自动化应该停止并升级为人工判断。对于 Agent 来说，这意味着它面对的不再只是代码和测试，而是仓库本身的治理结构。也正因为如此，代码熵治理不再只是事后补救，而开始变成开发过程中的默认约束。

## 在 Routa 中可视化

而在 Routa 中，这套能力第一次变得直观。从你提供的截图里可以看到，Entrix 的规则已经不再只是藏在 Markdown、YAML 和命令行里，而是被组织成一个团队能够直接理解的治理视图。上方的 `Dimension Radar` 并不是一个普通的雷达图，它把 `docs/fitness` 下分散的规则重新组织成一张质量轮廓图。

下载新版本（v0.2.9）体验：https://github.com/phodal/routa/releases/tag/v0.2.9 

从 `testability`、 `security`、 `code_quality` 到 `api_contract`、 `design_system`、 `observability` 和 `performance`，系统展示的不只是某一条规则是否通过，而是这个仓库当前的治理重心和治理薄弱点分别在哪里。换句话说，它把原本只有熟悉项目的人才能从一组规则文件里读出来的结构，转成了一眼就能共享的整体视图。

同样重要的是，截图左侧的 `Fitness files` 和右侧的 `Source View` 把这种整体视图重新落回到了具体规则本身。左边展示的是系统识别出来的一组治理资产，例如 `README.md`、 `manifest.yaml`、 `code-quality.md`、 `engineering-governance.md`、 `security.md` 和 `api-contract.md`；右边则进一步展开某一个规则文件中的 frontmatter、metric、dispatch、weight、pass、warn 等信息。

这样一来，Entrix 在 Routa 中就不再只是一个 CLI，也不再只是仓库中的若干规则文件，而是同时成为可执行的规则源、Agent 可消费的上下文，以及团队可以共同浏览和讨论的治理界面。规则第一次不只是“存在”，而是既能被执行，也能被解释，还能被团队直观看见。

## 总结

这也是我理解的 Entrix Skill 的意义。它不是让 Agent 更自由地生成代码，而是让仓库终于有办法把自己的完成条件施加到 Agent 身上。过去很多团队并不是没有规则，而是规则没有真正进入执行路径，于是系统只能不断依赖人去补救熵的累积。Entrix 和 Routa 把这件事往前推了一步：先把规则写进仓库，再把规则可视化，最后让这些规则真正进入 Agent 的工作上下文。