> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: SkillShare入门：基于Git跨平台同步AI智能体SKILL能力
author: 王嘉祥
date: 王嘉祥王嘉祥
url: https://mp.weixin.qq.com/s?__biz=MzI4OTM2NDY1Mw==&mid=2247483896&idx=1&sn=c01de8c1f2cca3542ac0adc565af413b&chksm=ed516b5fa2000c0195fd6d2b2dd8a50a002e75128b14a4805c58e94e5cdc4c4465b47c3b474e&mpshare=1&scene=24&srcid=043096PSBOBAItoMIK3uzrS6&sharer_shareinfo=5890ae6f8bd9176b1f19d06d3423020e&sharer_shareinfo_first=5890ae6f8bd9176b1f19d06d3423020e#rd
---

> 字数 923，阅读大约需 5 分钟

# 背景

我从 OpenClaw 火了以后就开始折腾 SKILL，所谓 SKILL，就是把一些 SOP 或者工作流以文档的形式整理成提示词，AI 工具会读取这些文档，从而胜任更复杂的任务。

SKILL 的使用与维护有一个痛点，就是每次切换 AI 工具，都得同步一遍自己维护的 SKILL 库，特别是要在多台电脑直接还有 NAS 的远程环境中同步更改，维护的心智负担很重。

为了解决这个问题，我问了 Gemini，给我推荐 SkillShare 这个开源的跨平台 SKILL 同步工具，上手试了一下，果然很好用，所以写这篇文章分享出来。

# 介绍

SkillShare[1] 的官方介绍如下：

> Sync skills across all AI CLI tools with one command and simplify team sharing. Supporting Codex, Claude Code, OpenClaw & more

简单来说，就是用户都可以在本地维护一个专属于自己的 SKILL 库，底层是基于 Git 仓库。用户可以自由存储自己整理的 SKILL，并且通过 GIT 同步，在其他的电脑上也能给 AI 提供一样的能力。

SkillShare 的使用非常简单，只要懂 Git 就能快速上手。

# 安装

SkillShare 安装只需要一行命令，macOS 和 Linux 通用：

```
curl -fsSL https://raw.githubusercontent.com/runkids/skillshare/main/install.sh | sh
```

注意，这条命令要求你的系统已经安装了 Git，因为 SkillShare 的底层就是跑在 Git 之上的。

# 初始化

安装完成后，需要做一次初始化。它会帮你在本地创建一个专属的 Git 仓库来存放所有的 SKILL 包。

第一步，先在 GitHub 或者你用惯的 Git 平台上手动创建一个空仓库，把地址记下来。

第二步，运行初始化命令并指定远端地址：

```
skillshare init --remote https://github.com/你的用户名/你的仓库名
```

这条命令会在本地初始化 Git 目录，并把远端仓库地址写入配置。执行完之后，你就可以开始往里面添加 SKILL 了。

# 同步操作

SkillShare 的使用逻辑和 Git 几乎一致，这一点很棒。

把远端的更新同步到本地：

```
skillshare sync
```

skillshare 还可以自动识别本地安装的 AI 助手或者开发工具，并且自动同步到对应的 SKILL 目录。

把本地的改动推送到远端：

```
skillshare push
```

整理完一个新 SKILL，运行这条命令就推到远端去了。

# 填坑：私有仓库认证

如果你和我一样，把技能库放在私有仓库里，克隆和拉取的时候会遇到认证问题。官方文档给了清晰的解决路径：

1. 1. 安装 gh（GitHub CLI），参考：https://docs.github.com/zh/github-cli/github-cli/quickstart
2. 2. 运行 gh auth login，按提示完成 GitHub 账号授权

完成之后，就可以正常拉取私有仓库的内容了。

# 总结

用下来 SkillShare 确实很方便，把零散的 SKILL 管理得井井有条。可目前最大的挑战变成了 SKILL 太多却不知道如何选择，而不是同步了。

SKILL 相关的开源项目一般会打包几十个 SKILL，不同项目之间有 SKILL 功能有重合，可每个项目都说自己很牛逼，确实让人难以选择。

现阶段先使用 SkillShare 作为 AI 工作流的一个环节，希望将来能有一个权威的 SKILL 评测体系，为普通用户提供 AI 模型和 SKILL 的性能表现矩阵。

值得一提的是，截至本文发布时间，稳定可靠的 SkillShare 的 GitHub stars 还没有超过 2000 个，而那些让人感觉很牛逼的 SKILL 仓库动辄上万 stars，又何尝不是一种悲哀。

#### 引用链接

`[1]` SkillShare: *https://github.com/runkids/skillshare*

 

---

公众号内容可能过时，请点击阅读原文访问文章原文，文章原文保持更新。