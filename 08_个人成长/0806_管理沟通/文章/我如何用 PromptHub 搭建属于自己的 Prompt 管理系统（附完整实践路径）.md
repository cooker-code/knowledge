---
title: 我如何用 PromptHub 搭建属于自己的 Prompt 管理系统（附完整实践路径）
author: 产品经理应用AI详细记录
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA3MDI5NzU1NA==&mid=2247483814&idx=1&sn=41c41ed1d1bac5031868beb5b3763307&chksm=9efee3cc9f6633889c6d72e08ec5de52be957c4d199ba7dbb9a8e8fca6c31e8ecad90f0c48f0&mpshare=1&scene=24&srcid=120737Gbwj9oUbQzWV9mNSth&sharer_shareinfo=3a4424ecc8f944c5451176a0cb7217b6&sharer_shareinfo_first=3a4424ecc8f944c5451176a0cb7217b6#rd
---

写在前面：分享一下整个过程和这篇文章的产生方式：

1. 和gpt聊我的prompt 脚踩西瓜皮的问题寻求解决方案。

2. gpt开始引导我使用工具，推荐的n个工具里，我选择了这个工具。

3. gpt开始手把手教我怎么用这个工具。过程中虽然gpt 有一些拿旧版（或者是幻觉）的引导。但最终我还是成功的搭建了这个group。

4. 搭建完成之后，我告诉gpt把这个过程写成一篇文章，供我发布。他完成的还挺好的，还帮我把我截图的丑丑的图都换成了官方的示意图。

整个过程中从开始要找prompt管理工具到这片文章写完，总共花了1.5h。我最核心的是需要保持足够的耐心， 并且发现问题使劲的问AI。

最近这几周，我在做一个自己的内容 AI工厂：  
一个可以自动 **写短篇小说、续写、建角色卡、调整文风、润色、生成世界观大纲** 的 AI 系统 —— 我把它叫做 **造梦厂 DreamFactory**。

在项目推进过程中，我踩了一个非常典型的大坑：

> **Prompt 越写越多、越改越乱，没有结构，没有版本，也无法复用。  
> 最后基本是“脚踩西瓜皮滑到哪里算哪里”。**

直到我开始使用 PromptHub，把 Prompt 当做 **工程资产** 来管理，整个系统才真正变得可控、可复用、可扩展。

这篇文章，我会把整个流程拆清楚，包括截图，让你看懂 PromptHub 如何从“杂乱无章”变成“内容工程的操作系统”。

# # **01｜为什么需要 Prompt 管理？**

如果你把 Prompt 放在：

* 备忘录
* 微信聊天
* 随便一个文档
* ChatGPT 历史记录
* ide里面直接聊...

那么：

### ❌ v1、v2、v3 完全没法区分

### ❌ 多个任务（续写/润色/风格化）混在一起

### ❌ 团队或未来自己完全无法复用

### ❌ 也无法通过 API 调用变成真正的产品

PromptHub 解决的就是：

### ✔ Prompt 有结构

### ✔ Prompt 有版本

### ✔ Prompt 可测试

### ✔ Prompt 可复用

### ✔ Prompt 可被你的服务器调用

### ✔ Prompt 最终变成“工程产物”

这是你能把“写 Prompt”升级为“搭建 AI 产品”的关键一步。

---

# # **02｜PromptHub 的基本结构：Group → Project → Version**

PromptHub 的结构非常像 Git + 小型工作区：

**Group**：你的产品线（例如：造梦厂）  
**Project**：某个能力模块（例如：短篇、续写、人物）  
**Version**：Prompt 的不同迭代版本

我给自己的产品结构拆成：

```
DreamFactory  
│  
├── DF-ShortStory     # 短篇小说生成  
├── DF-Continue       # 续写  
├── DF-Characters     # 角色卡  
├── DF-Style          # 文风控制  
├── DF-Refine         # 润色修饰  
└── DF-World          # 世界观 & 长篇结构
```

每个 Project 对应一类任务，便于后续维护。

---

# # **03｜在 PromptHub 里创建 Project（示意截图）**

在 PromptHub 的「Library → Create Project」界面，我统一使用：

* Name：DF-ShortStory
* Name：DF-Continue
* ...

这种命名方式，让整体结构非常清晰。

---

# # **04｜编辑 Prompt：System + Prompt 文本 + Variables**

点击进入某个 Project，会看到这样的 Playground：

区域说明：

* **System Message**：定义 AI 角色
* **Prompt**：真正的指令内容
* **Variables**：可替换参数
* **Output**：运行结果预览
* **底部紫色按钮：Commit** → 最关键

---

# # **05｜Commit = Prompt 的版本管理（非常关键）**

当你写完后，点击右下角紫色按钮：

**Commit immediately / Commit initial version**

会出现类似 Git 的版本窗口：

填写：

* Title: `v1 short story prompt`
* Description: `三幕结构，画面化写作`

提交后 Prompt 正式进入 version history。

---

# # **06｜查看版本：每次修改都是一条历史记录**

在右上角，你可以随时点：

👉 **View version history**

**版本历史就是 Prompt 的成长轨迹。**  
你可以随时：

* 查看差异
* 回滚版本
* 测试历史版本

这让 Prompt 的迭代像“工程开发”一样严谨。

# 7**｜写在最后：Prompt 是未来内容工程的“代码”**

任何做 AI 工程、AI 创作、AI 内容产品的人，迟早会意识到：

> **Prompt 不应该散落在聊天记录里。  
> Prompt 必须工程化。**

PromptHub 提供了一个很好的开始：  
它让你可以用最小成本，把所有的 Prompt：

* 分类
* 版本化
* 测试
* 调用
* 上生产

就像软件工程一样。

我现在回头看，会觉得：

**能管理好 Prompt 的人，未来会比能写 Prompt 的人稀有十倍。**