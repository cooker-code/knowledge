---
title: 强烈推荐skill-CLI-Anything：让所有软件都能被 AI 代理驱动 🤖
author: Gdplay玩AI草图
date: 
url: https://mp.weixin.qq.com/s?__biz=MzcwNzIwNjMxOA==&mid=2247484877&idx=1&sn=9ad34a967ae2b06dcc6d23e38081d175&chksm=f5222cebb5a95a05d8c933f63f2f12b498a5c9ead539519451fef8271653fec94029bd6ca4ef&mpshare=1&scene=24&srcid=0411Psjto3CQinnAkbSPLr2v&sharer_shareinfo=4db2142ad0e271ed15e3ef41799e902a&sharer_shareinfo_first=4db2142ad0e271ed15e3ef41799e902a#rd
---

> 🎯 **核心命题**：今日软件为人而生，明天的用户是 AI Agent

---

## 🚨 背景：一个正在被忽视的瓶颈

AI Agent 的推理能力已经很强了 💪

你可以让它做计划、写代码、分析数据——但当你真正想让它操控一个具体的专业软件时，问题就来了 😅

> 💬 *"能帮我把这段视频剪辑成竖版吗？"*  
> 💬 *"能生成一个 Blender 3D 渲染图吗？"*  
> 💬 *"能帮我在 Draw.io 里画一张架构图吗？"*

**答案是：要么做不到，要么做得很烂 ❌**

原因很简单：**几乎所有专业软件都是为人类设计的，从未考虑过被 AI 调用 🙈**

### 现有解法各有硬伤 💔

| 方案 | 问题 |
| --- | --- |
| 🖱️ **GUI 自动化**（截图+点击像素） | 脆弱得像纸糊的，一换分辨率就崩 |
| 🔌 **API 封装** | 覆盖面有限，通常阉割了 90% 的功能 |
| 🛠️ **重新实现** | 投入巨大，做出来还是个玩具 |

**CLI-Anything 的思路完全不同** 💡：与其改造 AI，不如改造软件接口。

**一条命令，把任意软件变成 AI 的原生工具 🚀**

---

## 💡 为什么是 CLI？

你可能会问：为什么偏偏是命令行？🤔

**因为 CLI 是人类和 AI Agent 之间天然的"共同语言"** 🌉

* • ✅ **结构化**：命令+参数，完美匹配 AI 的函数调用格式
* • ✅ **可组合**：管道、重定向、脚本，天然支持复杂工作流
* • ✅ **无界面**：不需要视觉理解，纯文本交互，稳定可靠
* • ✅ **可测试**：每一个命令都可被自动化测试验证

**这不是妥协，这是最优解** 🏆

---

## ⚙️ 核心技术：7 步全自动流水线

CLI-Anything 的核心是一个经过验证的 **7 阶段全自动流水线** 🔧

你把软件代码扔进去，出来的是一个可以直接发布的 Python CLI 包。

**整个过程无需人工介入** ✨

生成后的 CLI 长这样：

```
# 📦 安装到全局  
pip install -e ./gimp/agent-harness  
  
# 🌍 随处可用  
cli-anything-gimp --help  
cli-anything-gimp project new --width 1920 --height 1080 -o poster.json  
cli-anything-gimp --json layer add -n "Background" --type solid --color "#1a1a2e"  
  
# 💻 进入交互式 REPL  
cli-anything-gimp
```

---

## 🚀 一条命令，接入 7 大主流 Agent 框架

CLI-Anything 设计之初就是平台无关的 🎨

无论你用哪个 AI 编程工具，生成的 CLI 使用体验完全一致：

| 框架 | 支持状态 |
| --- | --- |
| 🤖 **Claude Code** | ✅ 原生支持 |
| 🔧 **GitHub Copilot** | ✅ 原生支持 |
| 🎭 **OpenClaw** | ✅ 原生支持 |
| ⚡ **Nanobot** | ✅ 原生支持 |
| 🧠 **Windsurf** | ✅ 原生支持 |
| 🌊 **Witsy** | ✅ 原生支持 |
| 🏢 **Kimi-IDE** | ✅ 原生支持 |

**一个输入，7 个平台同时支持** 🎯

---

## 📦 已覆盖的软件生态：10 大类别，40+ 款主流应用

这是最有说服力的部分 🎉 CLI-Anything 不是概念，是**已经真实可用的工具矩阵**：

### 🎨 创意设计

* • **GIMP** - 图像编辑
* • **Blender** - 3D 建模与渲染
* • **Inkscape** - 矢量图形设计
* • **Krita** - 数字绘画

### 🎬 音视频处理

* • **Audacity** - 音频编辑
* • **Shotcut** - 视频剪辑
* • **OBS Studio** - 直播推流

### 📊 生产力工具

* • **LibreOffice** - 办公套件
* • **Draw.io** - 流程图绘制
* • **Notion** - 知识管理

### 🤖 AI 内容生成

* • **Stable Diffusion WebUI** - AI 绘画
* • **ComfyUI** - 节点式 AI 工作流

> 📌 **所有生成的 CLI 都经过真实软件验证，1,508+ 测试用例，覆盖 11 款主流应用。**

---

## 🎯 CLI-Anything 能做什么？

### ✨ 全自动生成

无需人工干预，从源码到可发布 CLI，一键完成 ⚡

### ⚙️ 零侵入集成

不需要修改原软件代码，不需要 API 权限，不需要适配层 🔒

### 🔄 状态持久化

支持撤销/重做，有状态 REPL 会话，Agent 可以连续操作 📝

### 📊 结构化输出

每个命令内置 `--json` 参数，输出结构化数据供 Agent 消费 📈

### 🧪 完整测试覆盖

自动生成单元测试 + 端到端测试，确保生产可用 ✅

---

## 🎬 实测展示

### 🏭 专业级测试

在 **11 款复杂应用**上进行了实测，涵盖：

* • 🎨 创意工作流（图像编辑、3D 建模、矢量图形）
* • 📈 生产力工具（音频、办公、直播、视频剪辑）
* • 🤖 AI 内容生成

**这些软件此前对 AI Agent 来说几乎不可触及** 🚫

### ✅ 完整的 CLI 生成

每款软件都生成了完整的、可投产的 CLI 接口——

**不是 demo，而是保留全部功能的完整工具接入** 💪

### 📊 测试结果

全部 **1,527 项测试 100% 通过** 🎊：

* • ✅ 1,073 项单元测试
* • ✅ 435 项端到端测试
* • ✅ 19 项 Node.js 测试

---

## 🎯 核心设计原则

### 1️⃣ 真实软件集成

CLI 生成合法的项目文件（ODF、MLT XML、SVG），然后交给真实应用去渲染。

**我们做的是软件的结构化接口，而不是替代品** 🔄

### 2️⃣ 灵活的交互模式

每个 CLI 都支持两种模式：

* • 📝 **有状态 REPL** - 用于 Agent 交互会话
* • ⚡ **子命令模式** - 用于脚本和流水线

直接运行命令即进入 REPL 💻

### 3️⃣ 一致的使用体验

所有生成的 CLI 共享统一的 REPL 界面（`repl_skin.py`）：

* • 🎨 品牌横幅
* • 💫 风格化提示符
* • ⏮️ 命令历史
* • 📊 进度指示器
* • 📋 标准化格式

### 4️⃣ Agent 原生设计

* • 每个命令内置 `--json` 参数
* • 输出结构化数据供 Agent 消费
* • 可读的表格格式服务于交互调试
* • Agent 通过标准的 `--help` 和 `which` 命令发现能力

### 5️⃣ 零妥协的依赖策略

**真实软件是硬性要求**——没有兜底，没有降级。

后端缺失时测试直接失败（而非跳过），确保功能的真实性 💯

---

## 🌟 独特优势：和任何现有方案都不一样

| 特性 | CLI-Anything | 其他方案 |
| --- | --- | --- |
| 📝 **不需要源码** | ✅ 找不到源码就用开源替代品 | ❌ 必须有源码/API |
| 🔄 **状态持久化** | ✅ 支持撤销/重做 | ❌ 通常无状态 |
| 🧪 **自动生成测试** | ✅ 全自动生成 Benchmark | ❌ 手动编写测试 |
| 🎯 **真实应用调用** | ✅ 调用真实软件渲染 | ❌ 玩具实现或模拟 |
| 🔧 **无需适配层** | ✅ 零侵入 | ❌ 需要 API/适配层 |

---

## 🌐 CLI-Hub：社区驱动的 CLI 仓库

所有已验证的 CLI 统一收录在 CLI-Hub 🌍

🔗 **https://clianything.cc/**

### 快速安装

```
# OpenClaw 用户  
openclaw skills install cli-anything-hub  
  
# Nanobot 用户  
nanobot skills install cli-anything-hub
```

### 社区特性

* • 📂 按类别浏览，搜索你需要的软件
* • 🙋 社区贡献驱动
* • 📥 任何有 GUI 或 API 的软件都可以提交 PR

---

##

## 🎓 总结

CLI-Anything 解决的是一个**根本问题** 🔑：

> **AI Agent 和真实软件之间的最后一公里。**

它不改造 AI，不改造软件，只是给软件加了一层 AI 友好的 CLI 接口——

**而这层接口，是通过全自动流水线从源码自动生成的** 🤖

当所有专业软件都有了 AI 原生的 CLI 接口，AI Agent 能做的事，将从**"回答问题"**真正变成**"完成任务"** 🎯

本人已经实践过，要他操控Blender建模，但感觉还是差一点，不能直接要他建，应该先图生模，然后再扔给他细化，这样才是最优的方案

## 📚 资源链接

* • 🏠 **主仓库**：https://github.com/HKUDS/CLI-Anything
* • 🌐 **CLI-Hub**（可直接安装）：https://clianything.cc/
* • 📖 **中文文档**：https://github.com/HKUDS/CLI-Anything/blob/main/README\\_CN.md

---

> 💡 **本文纯属扯淡，如有雷同，我先瑞思拜**

> 📌 **本文参考资料均来自 CLI-Anything 官方 GitHub 仓库，测试数据基于官方公布内容。**

---

*—— 用 CLI 连接 AI 与软件的无限可能 🚀*

*[强烈推荐skill-OpenSpace：龙虾（openclaw）装上自我进化引擎，从笨手笨脚到独当一面 🦞（适用各种虾，workbuddy，Qclaw，JVSclaw等）](https://mp.weixin.qq.com/s?__biz=MzcwNzIwNjMxOA==&mid=2247484798&idx=1&sn=a8db8547d952e95757b9bf03ec3c6413&scene=21#wechat_redirect)*

*[🤖 当龙虾遇到爱马仕：OpenClaw vs Hermes Agent，从架构设计看他俩的核心差异与选型策略](https://mp.weixin.qq.com/s?__biz=MzcwNzIwNjMxOA==&mid=2247484799&idx=1&sn=2b9f98eb26073c2c851777cb433dfec6&scene=21#wechat_redirect)*

*[WorkBuddy 深度评测：新版本终于让AI Agent从“能说”到“会做”](https://mp.weixin.qq.com/s?__biz=MzcwNzIwNjMxOA==&mid=2247484768&idx=1&sn=e05f878f2083b5a1e3c549aca14e079b&scene=21#wechat_redirect)*

*[openclaw十大必装技能，"智障"进化成"智能"](https://mp.weixin.qq.com/s?__biz=MzcwNzIwNjMxOA==&mid=2247484516&idx=1&sn=27029568bbd3684430ecda5bf40ac8e3&scene=21#wechat_redirect)*

*[股票龙虾skills-装了这些技能后，我的openclaw变成了金融虾](https://mp.weixin.qq.com/s?__biz=MzcwNzIwNjMxOA==&mid=2247484468&idx=1&sn=0c3593dd36fe1cf548d88058e2db28f4&scene=21#wechat_redirect)*

*[装完这些skills，小龙虾变成澳洲大龙虾（适合OpenClaw，JVSclaw，workbuddy，kimiclaw等各衍生虾） 🦞](https://mp.weixin.qq.com/s?__biz=MzcwNzIwNjMxOA==&mid=2247484029&idx=1&sn=c725e8398d4eb3873eaf36edfe32ece1&scene=21#wechat_redirect)*

*[🦞 一个技术小白养龙虾的野路子生存指南---OpenClaw及workbuddy，jvsclaw，qclaw等衍生虾都适用](https://mp.weixin.qq.com/s?__biz=MzcwNzIwNjMxOA==&mid=2247483962&idx=1&sn=05d962c4eeece160c09b732cbe68829a&scene=21#wechat_redirect)*