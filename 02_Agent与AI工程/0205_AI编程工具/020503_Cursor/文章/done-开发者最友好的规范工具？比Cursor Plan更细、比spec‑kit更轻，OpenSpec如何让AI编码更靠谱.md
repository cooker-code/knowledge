> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor Rules与Plan工作流边界|Cursor Rules与Plan工作流边界]]、[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor工程使用与上下文边界|Cursor工程使用与上下文边界]]
---
title: 开发者最友好的规范工具？比Cursor Plan更细、比spec‑kit更轻，OpenSpec如何让AI编码更靠谱
author: kate人不错
date:
url: https://mp.weixin.qq.com/s?__biz=MzI5MjQ3ODY3Mw==&mid=2247493448&idx=1&sn=30d5b64bbd3a243e4a69f642c957aaf5&chksm=ed1f1d57a5c53494dc7561cb61adfca9236ec294ece9cc51c11172d24dbfe1211cc33503433f&mpshare=1&scene=24&srcid=1014SxL25x95Maqg3oAHfByx&sharer_shareinfo=74db5f149f91ca8f0bd66fbb61d08806&sharer_shareinfo_first=74db5f149f91ca8f0bd66fbb61d08806#rd
---

大家好，我是 Kate。今天想跟你系统介绍并演示一下 OpenSpec。这是我三天前在星球里分享过的一个仓库，有星友反馈它和 Codex（内置 GPT‑5）配合非常好。

https://github.com/Fission-AI/OpenSpec

### OpenSpec 是什么

* OpenSpec 旨在让人类与 AI 编码助手通过“规范驱动的开发”达成一致。
* 相比我之前分享过的 spec‑kit、BMAD-METHOD，OpenSpec 更轻量，也比 Cursor 当前的 Plan 模式更完善一些，但整体流程依然简单。

### 工作流总览：两大部分、五个步骤

* 两大部分：

+ specs：当前的规范状态
+ changes：变更提案与演进记录

* 基本步骤：

1. 提案（propose）
2. 审查与对齐规范（review & align）
3. 由 AI 反复分析并给出计划（plan with feedback loop）
4. 实施任务（implement）
5. 归档并更新规范（archive & update specs）

在整个过程中，你可以让 AI 不停分析、生成计划，并在反馈循环中迭代直至满意，再实施与归档。

### 工具支持与交互方式

* OpenSpec 支持多种常用工具，并且：

+ 提供便捷的命令行操作
+ 支持自然语言对话驱动流程
+ 与 AGENTS.md 兼容，便于在常见开发工具中安装和使用

### 安装与初始化

###

* 安装完成后进入你的项目，运行初始化：`openspec init`
* 可通过 `openspec list` 验证设置并查看更改
* init 之后即可创建你的变更（change）

### 与 spec‑kit / Kiro 的对比

* spec‑kit 在“从 0 到 1”搭建上很强；
* Kiro 与 OpenSpec 的一个差异是：OpenSpec 会把“功能的每次变更”分组到一个文件夹中，便于跟踪关联的规范、任务和设计。

---

## 实战：用 OpenSpec 改进一个 OpenJourney 项目

### 背景与目标

* 右侧展示的 UI（兔子、茶杯等）是我不久前生成的，下方的小猫图是 79 天前生成的。
* OpenJourney 是谷歌员工开源、模仿 Midjourney UI 的项目。

https://github.com/ammaarreshi/openjourney

* 我之前做过一次修改：

+ 将 Gemini API 调用改为 Replicate API；
+ 调整了图片和视频生成模型；
+ 支持将数据存储在本地。

* 本次我通过 Codex 与 OpenSpec 的协作，将“生成图片模型”替换为 Qwen Image：

+ 原因：在 Replicate 上成本更低、生成质量更高；
+ 同时在设置中增强了可调参数：推理步数、图片尺寸、输出格式、输出质量等。

### 用 OpenSpec 发起提案与校验

* 进入项目后运行 `openspec init`，系统会确认所用的 AI 编程工具（我选择了 Codex，顺带感谢一位 B 站网友的赞助 Codex 的使用）。随后它提示我三项工作：

1. 让它阅读 OpenSpec 里的 `project.md`，并帮助我填写有关我的项目、技术堆栈和惯例的详细信息；
2. 创建第一个变更提案（我要添加的功能是什么）；
3. 让 AI 解释 OpenSpec 的工作流。

* 初始化后会生成 `openspec/changes` 与 `openspec/specs` 两个主要文件夹，以及 `AGENTS.md` 和 `project.md` 文件。

（下图是存档后的目录）

* 第一步是让 AI 了解当前程序并把理解写入 `project.md`。由于文档默认是英文，我先让它翻译成中文再阅读补全。
* 项目里最初使用了两个模型，其中一个是“黑森林的 FLUX”。我提出把该图片模型改为 Qwen。

* 左侧随即出现了提案文档，包含 Why / What / Impact 的结构（OpenSpec 内置提示词）。

* 它提示我运行命令来校验提案格式。运行后系统将规范性用语修正为 MUST，以满足 OpenSpec 的校验规则；再运行提示，严格校验通过。

对应的文档内容如图获取

* 我把在 Replicate 上调用 Qwen Image 的详细文档发给它，OpenSpec/Codex 据此更新了提案说明和实施任务，明确需要支持的一些关键参数。再次校验依旧通过。

### 实施与调试要点

* 我让它开始实施。项目在修改前界面可正常输入文字、选择图片或视频并生成内容，但左侧有一个小报错。
* 遇到的“尴尬问题”：几个月前配置的 API Key 已被我删除。尽管规范和任务已很明确，但运行失败的根因是 Key 失效。更换有效的 API Key 后即可恢复正常生成。
* 选择 Expand 进入的模式很像 Midjourney，右侧可见历史生成图。

* 我还注意到最初设置里没有足够详细的“图片参数”。通过 OpenSpec 给 Codex 的提示，它帮我把这些可调项（步数、尺寸、格式、质量等）都补齐了。

* 最终 Codex 也修复了页面上的报错，视频、图片等生成流程全部恢复正常。

### 归档与查询

* 我提示 Codex 将更改存档。`archive` 文件夹中新增了“图片生成”的 `spec.md`，顶部可见 ADDED Requirements。

* 一些常用命令回顾：

+ `openspec list`：查看当前变更（若已归档为空）
+ `openspec view`：查看现有 spec（如图片生成 spec）
+ `openspec show` + 选择 `change`：若提示“no change found”，多半是已归档

* 完成后我检查了 `tasks.md`：最初未按预期打 ✓，于是让 AI 自检并更新状态。现在可见已完成项被正确勾选；也有少数如“在本地用真实提示词验证并保存到图库”的步骤，需要我手动完成。

---

## 何时优先选用 OpenSpec

* 当你在“修改现有功能”或“触及多个规范”时，OpenSpec 的变更分组与规范驱动流程尤其有用。
* spec‑kit 更适合从 0 到 1；需要系统化跟踪“规范—任务—设计”的演进时，OpenSpec 的结构更利于协作与追溯。

## 操作与提示词速查

* 常用命令：`openspec init` → `openspec list` → `openspec view` → `openspec show`
* 提示词（由系统引导，不必强记）：

1. “创建变更提案（用于×××）”
2. “验证提案/规范”（系统会在每一步自动提示下一条命令）
3. “完善规范（specs）→ 实施 → 归档”

## 小结与回顾

* 流程回看：

1. 在已有项目中 `openspec init`，生成 `project.md`
2. 让 AI 审阅项目并补充 `project.md`
3. 发起变更提案，AI 产出计划与任务
4. 人机共审、修订、严格校验（MUST 等用语规范）
5. 满意后实施，问题定位与修复（如 API Key 失效）
6. 归档变更并检查 `tasks.md` 的完成状态

以上就是我今天关于 OpenSpec 的分享。希望能帮你把 AI 编程流程跑通、跑稳。

# 广告

过去我已创作了 340+ 篇AI主题原创内容，我对继续写作充满信心，因为这是我的爱好，我非常热爱这件事。

如果喜欢我的文章和视频，欢迎加入我的知识星球，我会分享最新的 AI 资讯、源代码，回答你的问题。我们下次再见啦！



**最近文章，请看这里：**

**[实测Claude Sonnet 4.5：一条提示词做表格/报告/PPT，文档流水线搭建指南](https://mp.weixin.qq.com/s?__biz=MzI5MjQ3ODY3Mw==&mid=2247493418&idx=1&sn=a01abc1800445812a3a88e4a20e304b6&scene=21#wechat_redirect)**

**[比我想象更强！Claude Sonnet 4.5 写作实测：细节、节奏、冲突全拉满 | Claude Agent SDK 教程](https://mp.weixin.qq.com/s?__biz=MzI5MjQ3ODY3Mw==&mid=2247493329&idx=1&sn=4fb82e910ea0ee166d63deff201cdfc7&scene=21#wechat_redirect)**

**[不是广告，实测智谱 GLM‑4.6，编码真的强！](https://mp.weixin.qq.com/s?__biz=MzI5MjQ3ODY3Mw==&mid=2247493288&idx=1&sn=7a99c81ae046024c992d7932013b4840&scene=21#wechat_redirect)**

**[讲透 Claude Sonnet 4.5：实测，细节到位](https://mp.weixin.qq.com/s?__biz=MzI5MjQ3ODY3Mw==&mid=2247493236&idx=1&sn=1494a166ddee944804c55d2f5c3fa0fd&scene=21#wechat_redirect)**

**[实测 DeepSeek V3.2‑Exp，实战 PK Qwen 3 Max](https://mp.weixin.qq.com/s?__biz=MzI5MjQ3ODY3Mw==&mid=2247493190&idx=1&sn=676f4c7c0affcdd0cd3e71d6487b9f80&scene=21#wechat_redirect)**

**[50天复盘：看懂 GPT‑5 thinking 的真正价值——不是答案，更是工作流](https://mp.weixin.qq.com/s?__biz=MzI5MjQ3ODY3Mw==&mid=2247493156&idx=1&sn=5eb0641cde138b3cd58fa702d397e5c1&scene=21#wechat_redirect)**

**[Auggie 实战指南，Augment Code CLI，详解 + 实测](https://mp.weixin.qq.com/s?__biz=MzI5MjQ3ODY3Mw==&mid=2247493133&idx=1&sn=ee13f74d66812aaa8db3d5778dfb038e&scene=21#wechat_redirect)**