---
title: 我愿称为最强终端工具 Warp
author: ITPostman
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIyOTYxNjU0OA==&mid=2247484190&idx=1&sn=eb34888e6c98be57395c0ab67873abed&chksm=e9cc4a855044b66304f0b0ee25e0d84d94224e1126f194468e29d8bbf0834d6562955253053e&mpshare=1&scene=24&srcid=1219LKKwrsCmHREmUGYSb612&sharer_shareinfo=2d08200370b7d8f524de47a9edcb7fac&sharer_shareinfo_first=2d08200370b7d8f524de47a9edcb7fac#rd
---

你能想象一款终端工具，它不仅速度飞快，还拥有智能补全、内置 Agent 编程工具和现代化的用户界面吗？这就是 Warp，一款重新定义终端体验的工具，支持多标签页和分屏操作，让你可以轻松管理多个终端会话，Warp 还内置了强大的搜索功能，可以快速查找历史命令和输出内容。它支持语法高亮和代码片段，让你在编写脚本时更加高效。此外，Warp 还集成了 Git 功能，让你可以直接在终端中进行版本控制操作，无需切换到其他工具。

打开 Warp，你会发现它给人感觉像是代码编辑器，是不是有点像 VSCode？Warp 的界面设计现代且直观，支持自定义主题和布局，让你可以根据个人喜好调整工作环境。它还支持插件扩展，允许用户根据自己的需求添加功能模块，进一步提升工作效率。

Warp 终端界面

它还支持文件浏览和管理，你可以直接在终端中浏览文件系统，进行文件操作，而无需切换到图形界面。Warp 还支持多平台使用，无论你是在 macOS、Linux 还是 Windows 上，都能享受到一致的终端体验。

Warp 多标签页和分屏

它的功能太多了，今天我主要介绍它的 Agent 功能和如何在 Warp 中使用 Claude Code 和 Codex。

## Warp Agent 功能

Warp Agent 内置了多款大模型，包括 Claude、OpenAI、Gemini 还有国产的 GLM 4.6。和 Cursor 类似，它还有一个 auto agent，可以根据你的需求自动选择合适的模型来完成任务。

Warp Agent 界面

目前包含有限的免费额度，超出后需要付费。

Warp Agent 价格

## 在 Warp 中使用 Claude Code 和 Codex

目前使用 Claude Code 和 Codex 一般有两种方式：

1. 1. 在其他终端工具中使用，比如 iTerm2、Windows Terminal 等。
2. 2. 在 Cursor、VSCode 等代码编辑器中安装插件使用。

第一种方式需要对于大部分开发者有抵触，觉得麻烦且不方便。第二种方式虽然方便，个人感觉太重了，不够轻量级。

而 Warp 它很好的结合了这两种方式的优点，既有终端的轻量级，又有代码编辑器的便利性。

目前情况下，你打开 Warp 后，它是在 Agent 模式下运行的，你需要切换到 Terminal 模式，才能使用 Claude Code 和 Codex。

Warp 切换到 Terminal 模式

点击左上角的 Agent 图标，选择 Terminal，就可以切换到终端模式。

然后你就可以像在其他终端工具中一样，使用 Claude Code 和 Codex 了（前提是你安装了 Claude Code 和 Codex）。

Warp 使用 Claude Code 和 Codex

乍一看，这和在其他终端工具中使用 Claude Code 和 Codex 没有区别，Warp 的优势在于它的用户体验和功能集成。

点击左上角的 Tools panel，可以看到文件浏览器，这样你就可以直接在终端中浏览和管理文件了。

Warp 文件浏览器

你还可以拖拽文件到终端中，这样就可以快速获取文件路径，极大提升了工作效率。

Warp 拖拽文件

此外，Warp 还支持多标签页和分屏操作，你可以同时打开多个终端会话，方便进行多任务处理。这里我就不一一介绍了，大家可以自行探索。