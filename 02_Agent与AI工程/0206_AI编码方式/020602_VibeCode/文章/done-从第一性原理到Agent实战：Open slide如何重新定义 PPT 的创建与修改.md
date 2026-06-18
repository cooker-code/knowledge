> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020602_VibeCode/020602_核心知识点/VibeCode边界与交付判断准则|VibeCode边界与交付判断准则]]
---
title: 从第一性原理到Agent实战：Open slide如何重新定义 PPT 的创建与修改
author: 查理的AI日记
date: 查理的AI日记查理的AI日记
url: https://mp.weixin.qq.com/s?__biz=MzY4ODMwNzI2MA==&mid=2247483674&idx=1&sn=84c9c720a392774adb0452e92044420c&chksm=f21c9a5051f63482ae1b9107123d9eff62dd116ef627492b3dd65b616a545ab999f72e9b7e5d&mpshare=1&scene=24&srcid=0511IqJwZXZZchW3FZgYfEfr&sharer_shareinfo=cc775fa73608b8e51301a8c3521cff17&sharer_shareinfo_first=cc775fa73608b8e51301a8c3521cff17#rd
---

# 前言：研究背景

最近一直在研究 AI Agent，把 Hermes、Claude Code 这些框架都调好了，也接入了大模型，但一直缺一个应用场景。

逛项目时看到了 open-slide，GitHub 上2800多 star，正好最近要做公司内部 AI 宣传汇报做个ppt。借这个现成的机会准备试一下。

在AI时代，保持持续输入的同时，一定要保证输出，信奉实践哲学。光知道不够，得真正用起来。

# Part 1：第一性原理解读

核心问题：open-slide 是怎么"写 PPT"的？

答：open-slide 不是生成 PPT 文件，而是你提供方向和大纲给Agent，AI Agent 去生成 React 代码，框架负责渲染。

一句话总结

> 如果把传统 PPT 工具比作"用画笔画画"，那 open-slide 就是"用 CSS 写网页"——你关注的是内容和结构，渲染交给框架。本质是通过你把你的想法给 Agent ，通过调用它这套框架进行前端的渲染，最终产生可以交互的一种演示成果。

# Part 2：认知框架（结构树思维）

要真正用好 open-slide，得先搞清楚它的"骨架"是怎么长的。这里分两个视角：

如果你要贡献代码或深入理解框架，源码仓库是这样组织的：

关键点：

core 是运行时，封装了 React、Vite、热重载等所有技术细节

cli 是脚手架工具，负责生成用户项目

demo 只是开发框架时的测试项目，不是用户要用的

用npx @open-slide/cli init my-slide初始化后，得到的是一个完整的工作区：

关键点：

你 99% 的时间只在slides//index.tsx里和 AI 对话

.claude/skills/里预置了 create-slide、apply-comments等技能

这是一个标准的 Vite + React 项目，可以部署到任何静态托管

themes/用来存放自定义设计系统

# Part 3：Agent实战

作为非开发者，前置几个概念要搞清楚：

| 工具 | 作用 | 关系 |
| --- | --- | --- |
| **Homebrew** | macOS 系统级包管理器 | 用来安装 Node.js、pnpm 等软件 |
| **Node.js** | JavaScript 运行时 | 是 pnpm 的运行基础，自带 npm |
| **pnpm** | Node.js 生态的包管理器 | 管理项目依赖，需要 Node.js 才能运行 |

**一句话总结**：Homebrew 装 Node.js，Node.js 是 pnpm 的运行基础，pnpm 管理 Node 项目依赖。

安装步骤：

```
#1. 安装 Homebrew（如果还没有）/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"#2. 用 Homebrew 安装 Node.jsbrew install node #3. 用 Homebrew 安装 pnpmbrew install pnpm#4. 验证安装node-v # 应该看到 v18+pnpm-v # 应该看到版本号
```

# Step 1：快速安装项目（3 分钟）

```
拉取官网对应npm包cd'对应文件夹'npx @open-slide/cli init my-slide
```

访问http://localhost:5173，看到幻灯片列表就算成功了。

# Step 2：让 AI 生成初稿（10 分钟）

打开新终端，以claude为例：

```
cd 'my-slide的路径'claude
```

然后在 Claude 里输入：

```
/create-slide Topic: AI 基础介绍（了解大语言模型、训练原理与应用场景）8-10 页，专业但友好的风格
```

AI 会反问几个问题：

主题细节：核心受众是谁？（管理层/技术团队/全员）

美学方向：偏科技感还是偏温暖？

是否需要动画：subtle 淡入还是动态转场？

回答完，AI 自动生成

slides//index.tsx。

# Step 3：Inspector 模式修改（15 分钟）

这是 open-slide 最酷的地方——点击即改。

形成了“生产-反馈-修改-生产”的正循环

操作流程：

浏览器访问http://localhost:5173/s/ai-training

点击右上角 "Inspect" 按钮

点击任意不满意的地方（标题、文字、颜色）

输入评论："把这个标题改成橙色" 或 "这里加一个副标题"

评论会被保存为 @slide-comment 标记

批量应用修改：

```
在终端运行/apply-comments
```

AI 会读取所有评论，批量应用到代码里。然后热重载，你直接看到效果。

循环这个流程：预览 → 评论 → 应用 → 再预览，直到满意。

# Step 4：导出和分享（2 分钟）

分享方式：

静态 HTML：直接拖拽dist/index.html到浏览器

部署上线：一键推 Vercel / Cloudflare Pages

PDF 导出：浏览器打印 → 另存为 PDF

全屏演示：按 F 键进入 Present 模式

完整流程耗时：约 30 分钟（从 0 到成品）

对比传统 PPT：找模板（10 分钟）+ 复制内容（30 分钟）+ 调整格式（40 分钟）+ 找配图（20 分钟）= 2 小时

# Part 4：适用对象

此项目需要快速制作、演示幻灯片的非专业技术用户打造，无需复杂配置与深度开发能力，开箱即用：

职场办公人群：白领、职场新人，快速制作工作汇报、项目总结、产品宣讲幻灯片；

学生群体：大学生、研究生，高效完成课程答辩、学术分享、毕业设计演示；

教师 / 培训师：快速生成教学课件、培训讲义，简化演示内容制作流程；

轻量演示需求者：无需专业 PPT 软件、追求简洁高效，仅需基础演示功能的用户；

技术入门者：前端新手、编程初学者，无需钻研源码，一键搭建专属演示项目。

总结：只要你需要做演示幻灯片，无论有无技术基础，都能轻松用它快速完成创作。

# Part 5：项目思考

写完这篇，我也在想：open-slide 代表的是未来，还是只是一个过渡方案？

我的答案：

open-slide 开创了一个很新颖的Agent 与 PPT 交互的本地化解决方案、交互范式。

它抓住了汇报的本质——AI Agent 不是替代人类，而是把人类从"操作软件"的劳动中解放出来，回归到"表达想法"的本质。在一些不是特别复杂的 PPT 上，可以快速出成果，而且整体风格唯美、极简。

短期内可能替代不了 PPT 很多复杂的功能，但是我认为一个事物发展到最后，一定是大道至简，AI最终的能力也会渗透到各种生态上的工具。

# 附录 A：核心优势总结

|  |  |
| --- | --- |
| 优势 | 说明 |
| **Agent-first** | 整个框架为 AI 生成优化，AI 是核心用户而非插件 |
| 代码即设计 | 设计决策是代码变量，可版本化、可复用、可自动化 |
| 无限扩展 | 本质是 React 代码，可嵌入实时图表、API 数据、自定义组件 |
| 版本控制 | Git 管理，每一行修改都有记录，支持 PR 流程 |
| Inspector 交互 | 点击即改，评论批量应用，迭代成本极低 |

# 附录 B：Skills 总结

1. **create-slide**

   负责整体流程：问问题 → 规划结构 → 写代码
2. **slide-authoring**

   是技术参考：Canvas 规范、Type scale、布局规则
3. **current-slide**

   定位当前页面：用户说 "this" 时自动读取位置
4. **apply-comments**

   批量应用评论：Inspector 模式的核心
5. **create-theme**

   生成主题文件：可复用的视觉规范

评论区聊聊：

你在 PPT 制作中遇到过什么坑？

下期想看什么 AI 工具实战？

你觉得 AI 会完全取代传统 PPT 工具吗？