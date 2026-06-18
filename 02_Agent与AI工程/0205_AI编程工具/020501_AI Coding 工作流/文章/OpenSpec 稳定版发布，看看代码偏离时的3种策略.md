---
title: OpenSpec 稳定版发布，看看代码偏离时的3种策略
author: 时间维度
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI4NTAwNjEzMQ==&mid=2651781103&idx=1&sn=7e32e7d634eeceda5e3eada17cca09b4&chksm=f142b297c25534189720bb2d1a83e33a00b5ea867a7844424530f9e1d9f999334093e0988ed7&mpshare=1&scene=24&srcid=0228yKGMKwxPu9WiwNvYpiFG&sharer_shareinfo=4c1a38721fa3982ac0252e033e6ef5e6&sharer_shareinfo_first=4c1a38721fa3982ac0252e033e6ef5e6#rd
---

在《[在下达任务与AI执行之间，协商机制可以有](https://mp.weixin.qq.com/s?__biz=MzI4NTAwNjEzMQ==&mid=2651780988&idx=1&sn=2264dfb6683b4fcae1be061ef982829d&scene=21#wechat_redirect)》中，介绍到 OpenSpec，通过规范来驱动 AI 辅助开发，让开发过程高效且可控。现在稳定版来了，OpenSpecv1.0.0 于 2026 年 1 月 26 日正式发布，有较大的变化，更好理解、更灵活易用了。当下，最新更新版本为OpenSpec v1.2.0，于 2026 年 2 月 24 日正式发布。

相关信息

https://openspec.dev/

https://github.com/Fission-AI/OpenSpec/

```
npm install -g @fission-ai/openspec@latestcd your-projectopenspec init
```

# 1、对话框命令前缀简化为opsx

为了提升在对话框中的输入效率，所有的指令前缀从 /openspec: 全面简化为了 /opsx:（OpenSpec eXecutable）

# 2、对话框命令组合自定义

默认并不开启所有命令，可根据需要开启命令组合。

需通过以下命令配置扩展命令：

```
openspec config profile
```

进入 `Workflows only` 菜单后，可以勾选开启关闭各个命令。

配置完成后，可通过以下命令应用生效。

```
openspec update
```

# 3、一些命令场景

## - /opsx:continue

场景：代码写到一半断了、Token 达到上限、或者 AI 在每步任务间停下等你确认。

理解：它是给 AI 的“加速器”，意味着：“AI，你接着做你没做完的活儿。”

- /opsx:ff (Fast-Forward)

场景：你发现 AI 写得太慢，自己动手改了几行代码，或者手动完成了任务清单里的某一步。

理解：它是给 AI 的“同步器”，意味着：“AI，这几步我已经写好了，你扫描一遍，然后把任务勾选上，跳到下一步。”

## - /opsx:new

场景：用于开启一个全新的功能域。

区别：/opsx:proposal 是在现有规格下改动；而 /opsx:new 是为了建立全新的 spec.md。比如你刚开始做“支付系统”这个模块，适合先用 new 定义支付的全局规格。

# 4、实现有偏离时的干预

在 OpenSpec (OpsX) 的工作流中，当执行 /opsx:apply 后的代码偏离了预期或规格时，不要直接在对话框里盲目地要求 AI “再改改”。这样做会导致代码与规格（Specs）脱节，项目会陷入混乱。

通常应该根据偏离的程度，采取以下三种递进的处理方式：

## （1） 微调模式

直接修改 `proposal.md` 或 `tasks.md`

如果你发现 AI 理解错了某个具体的逻辑，或者漏掉了一个小功能：

操作：直接在编辑器里打开相应的 proposal.md 或 tasks.md。

修正：修改对应的描述或增加缺失的步骤。

重跑：回到对话框，再次输入 /opsx:apply。

原理：当你再次运行 apply 时，AI 会重新读取这些文件，发现差异并修正代码。这确保了“文档”与“代码”永远同步。

## （2）协作模式

手动修改 + /opsx:ff (Fast Forward)

如果你觉得 AI 写得太烂，或者某个复杂的算法你自己写更快：

操作：直接在代码编辑器里动手修改代码。

同步：修改完成后，在对话框输入 /opsx:ff。

原理：/opsx:ff 会让 AI 扫描你刚刚亲手改动的代码，并自动勾选 tasks.md 中对应的任务。

适用场景：当你不想让 AI 继续在某个错误的方向上浪费 Token 时，这是夺回控制权的最快方法。

## （3）重启模式：撤销变更 (Undo)

如果 AI 的偏离极其严重，代码改得一团糟：

操作：使用 Git 放弃本次变更（git checkout .），重新审视 /opsx:proposal。

修正：通常严重的偏离是因为 proposal 写得太模糊，或者 specs/ 下的基础规则有冲突。

# 🛠 进阶工具

使用 /opsx:verify 发现偏离

通常大的检查也不需要肉眼去盯着每一行代码。在准备执行 archive 之前，先运行： /opsx:verify，它会像一个审计员一样，对照 proposal.md 和 specs/ 检查当前代码，列出哪些地方“违章”了，哪些地方“偷工减料”了。可以根据 verify 的结果决定是进行“微调”还是“手动干预”。