> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Obsidian教程15：Claudian——把 Claude Code 装进 Obsidian 侧边栏
author: HelloRanceLee
date:
url: https://mp.weixin.qq.com/s?__biz=MzU2NzcwOTQyMQ==&mid=2247484789&idx=1&sn=6b4a92e44b7c59a74bd8a93ccd717250&chksm=fdce9055070c301093f164a584726fabce4a3ba0c5a8794f34951c5c1f733965f413c0492595&mpshare=1&scene=24&srcid=0414RlKxH0ZWUHPUx1qX7spv&sharer_shareinfo=26bf258a7679261414f9ff962d659a7e&sharer_shareinfo_first=26bf258a7679261414f9ff962d659a7e#rd
---

上一期讲了 AI 和 Obsidian 集成的三种方式：直接给文件路径、Obsidian CLI、还有 Skill。那三种方式里，CLI 和 Skill 的组合效果最好，但门槛也最高——你得开着终端，得了解 CLI 命令，Skill 要自己写或者改。

如果你觉得这些太麻烦，或者只是偶尔想让 AI 帮你处理一下笔记，不想折腾这一套，今天介绍的这个插件是个更轻量的选择：**Claudian**。

---

# 插件方式和直接用终端有什么区别

在说 Claudian 之前，先说清楚这两种方式的根本差别。

**直接用终端**的时候，你有两个窗口：Obsidian 和 Claude Code。你在 Obsidian 里写着笔记，想让 AI 帮你处理，切到终端，说"帮我看一下这个文件：`/Users/你/Obsidian/某某笔记.md`"，AI 处理完了，你再切回 Obsidian 看结果。两个窗口来回跳，每次都得手动说清楚你在处理哪个文件。

**插件方式**不同。Claudian 把 AI 嵌进了 Obsidian 的侧边栏。你打开哪篇笔记，AI 自动知道你在看什么，不需要你说路径，直接在右边的面板里对话就行。处理完了也不用切窗口，结果直接反映在笔记里。

打个比方：终端方式是在旁边开了一间办公室，你得跑过去说"帮我处理这份文件"，然后把文件带过去；插件方式是在你桌子旁边放了个助理，你说"帮我改一下这段"，它直接在你面前动手。

有一件事值得说清楚：**Claudian 本质上调用的还是 Claude Code CLI**，能力和在终端直接用 Claude Code 一样，不是用了什么别的 AI。区别只是入口换了，从终端变成了 Obsidian 侧边栏。

所以如果你已经在认真用 Skill + CLI 这套方案，Claudian 并不是更强的替代，只是让操作更顺手。如果你不怎么用终端，或者觉得那一套太重，Claudian 才是更适合你的切入点。

---

# Claudian 是什么

Claudian 是一个 Obsidian 插件，作者是 YishenTu，开源，MIT 协议，GitHub 地址：

https://github.com/YishenTu/claudian

2026 年初发布，短时间内就涨到 4500+ stars，在 Obsidian 和 Claude 用户圈子里传播很快。

目前它还不在 Obsidian 官方插件市场里，需要手动安装。

---

# 怎么安装

安装之前先确认两件事：

* 电脑上装了 Claude Code CLI（命令行版本，不是网页）
* Obsidian 版本是 1.8.9 或更新

这两个是前置条件，缺一不可。Claude Code 的安装可以看我之前 AI 教程系列里的入门篇，这里不重复。

模型这块支持的范围挺广，Claude 订阅、API Key、或者接入 OpenRouter、DeepSeek 这类支持 Anthropic API 格式的平台都行。

**安装方式一：从 GitHub Release 手动下载（推荐）**

1. 打开 https://github.com/YishenTu/claudian/releases，进最新版本的 Release 页面
2. 下载三个文件：`main.js`、`manifest.json`、`styles.css`
3. 在你的 Vault 里找到 `.obsidian/plugins/` 文件夹，在里面新建一个叫 `claudian` 的文件夹
4. 把三个文件放进去
5. 打开 Obsidian → 设置 → 第三方插件 → 找到 Claudian → 启用

**安装方式二：用 BRAT 插件自动安装**

BRAT 是一个专门用来安装还没上架官方市场的插件的工具，装完之后 Claudian 有更新也会自动提醒你。

1. 在 Obsidian 社区插件里搜 BRAT，安装并启用
2. 打开 BRAT 的设置，点 "Add Beta plugin"
3. 输入 `https://github.com/YishenTu/claudian`，点 Add Plugin
4. 装完后在第三方插件里启用 Claudian

两种方式都行，手动下载更直接，BRAT 更新方便。

---

# Claudian 能做什么

## 侧边栏对话，直接操作 Vault

点击左侧 ribbon 的机器人图标，或者用命令面板打开 Claudian 面板，右边会多出一个对话栏。

在这里跟 AI 说的话，它能直接在 Vault 里执行：读文件、写文件、搜索内容、运行命令——和你在终端里用 Claude Code 是一样的能力。唯一的区别是你全程没有离开 Obsidian 界面。

## 自动感知当前笔记

这个功能我觉得是最省事的地方。

你打开哪篇笔记，Claudian 就自动把它附加进上下文，AI 一开口就知道你在看什么，不用你每次说文件路径。

如果你想引用其他文件，在对话框里输入 `@`，会弹出一个下拉列表，你可以从 Vault 文件里选，也可以引用 MCP 服务或者自定义 Agent。

另外你可以给某些笔记打标签（比如 `private`），Claudian 会自动跳过这些文件，不会把它们的内容传给 AI。

## Inline Edit：选中文字直接改

这个功能适合你只想改一段，不想让 AI 动整篇笔记的场景。

选中你想改的那段文字，触发快捷键（可以自己设定），一个小面板弹出来，你说"改成更口语一点"、"把这段翻译成英文"之类的指令，AI 只处理你选中的部分。

## 图片支持

对话框里可以传图片：直接拖进去、粘贴截图、或者输入图片文件路径，三种方式都行。

AI 看到图片之后可以描述内容、提取文字、回答"这张图在说什么"之类的问题。对于经常截图做笔记的人来说挺实用的。

## Slash Commands

输入 `/`，会弹出你设定好的指令模板列表。

比如你常用"帮我总结当前笔记的要点"这个操作，可以把它存成 `/总结`。下次直接输入 `/总结`，指令就展开了，不用每次重新打一遍。

模板里可以带参数占位符，也可以引用 `@文件`，还可以内嵌 bash 命令。如果你用 Claude Code 的 Skill，Claudian 也认同样的格式，两边可以共用。

## Plan Mode

按 Shift+Tab 切换到 Plan 模式。

在这个模式下，AI 不会直接动你的文件，而是先探索、先规划，把计划列出来给你看，确认了再执行。

适合你给 AI 下一个范围比较大的任务，比如"帮我整理 `01笔记/` 文件夹的结构"、"把这几篇笔记合并成一篇综述"。这类操作如果直接跑，AI 可能会改很多你没预料到的东西。先看计划，比事后看着文件一团乱再撤销要稳很多。

## 权限模式

Claudian 有三种权限模式，控制 AI 执行操作时需不需要你确认：

* **YOLO**：全自动，AI 做任何操作都不弹窗，直接执行。效率最高，但要对 AI 的判断有信心
* **Plan**：就是上面说的 Plan Mode，先规划后执行

日常用 YOLO 就够了，对自己 Vault 结构比较在意的可以用 Safe，大操作前先开 Plan 看一眼。

## 模型选择

可以在 Haiku、Sonnet、Opus 之间切换，满足不同任务的精度需求。问个简单问题用 Haiku，省 token；整理复杂笔记用 Sonnet；需要深度推理的用 Opus。

---

# 总结

**今天学到了什么：**

1. 插件方式和终端方式的核心区别：不用切窗口，AI 自动感知当前笔记
2. Claudian 是把 Claude Code 嵌进 Obsidian 侧边栏的插件，底层调用的还是 Claude Code CLI
3. 安装需要 Claude Code CLI + Obsidian 1.8.9+，从 GitHub Release 手动下载或用 BRAT 都可以
4. 核心功能：侧边栏对话、自动附加当前笔记、Inline Edit、图片支持、Slash Commands、Plan Mode、权限控制

**核心要点：**

* 如果你不想折腾终端和 Skill，Claudian 是更低门槛的入口
* 如果你已经在用 CLI + Skill 方案，Claudian 不是替代，是让操作更顺手
* Inline Edit 适合精准改局部，Plan Mode 适合做范围大的操作
* 权限模式从 YOLO → Plan，越往后越谨慎，根据场景选

---

如果觉得有帮助，记得关注这个系列！