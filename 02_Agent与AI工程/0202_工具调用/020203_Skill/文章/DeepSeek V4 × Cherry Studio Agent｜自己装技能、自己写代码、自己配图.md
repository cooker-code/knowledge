---
title: DeepSeek V4 × Cherry Studio Agent｜自己装技能、自己写代码、自己配图
author: Cherry Studio 全能AI工作站
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk3NTMxNDI0Mg==&mid=2247485547&idx=1&sn=73e31bc1ebf71ddccf4bad08529baa98&chksm=c54b112f21c02177243fce37d1c0d7e877de4b8d68fd7c31ef3a130a5e0ab903bab7dbd15fdf&mpshare=1&scene=24&srcid=0425RHisYRGsF6EJOY4VJkS8&sharer_shareinfo=eedd093a6100a711d284ed6ad0d43f75&sharer_shareinfo_first=eedd093a6100a711d284ed6ad0d43f75#rd
---

今天发现 **歸藏** 刚刚开源的杂志风 PPT Skill 很酷——一个能让 AI Agent 自动生成「电子杂志 × 电子墨水」风格网页 PPT 的技能，WebGL 流体背景、衬线排版、数据大字报，效果堪比专业设计师出品，看完立刻想试试。

正好，DeepSeek V4 也刚发布了。这次一口气出了两个版本：**V4-Flash** 又快又省，日常聊天随便用；**V4-Pro** 推理能力拉满，写代码、跑 Agent 都很稳。Cherry Studio 已经率先完成适配，更新到新版就能体验。

那我们就来做一个有意思的事：**用 DeepSeek V4-Pro 驱动 Cherry Studio Agent，让它自己安装 PPT Skill，然后自动生成一份 20 页杂志风网页 PPT。**全程只需要发两条消息。

先看成品效果：

左右滑动查看更多

DeepSeek V4 生成的杂志风 PPT 

没有用任何模板，没有手动画一笔。深色「靛蓝瓷」配色质感在线，标题是衬线字体，正文是无衬线，数据页自带大字报排版，架构页有流水线图示，甚至配图都是 AI 用代码画出来的——没给它图片素材，它自己生成了与内容匹配的矢量图形。

 

手把手教程

从零到成品，第一次用 Agent 也能走通

**❶**  配置 CherryIN，获取 V4 模型

打开 Cherry Studio，点击左下角「**设置**」，左侧选择「**模型服务**」，找到 **CherryIN** 并登录。

登录后点击「刷新」获取模型列表，勾选添加 **deepseek-v4-flash** 和 **deepseek-v4-pro**。

**❷**  进入「智能体」，选择 V4-Pro

点击左侧边栏「**智能体**」，打开一个 Agent，在顶部「**选择模型**」切换到 **deepseek-v4-pro**。

V4-Pro 是这次的旗舰推理模型，还支持 high / xhigh 两档深度推理模式，跑复杂任务更靠谱。

**❸**  让 Agent 自己安装「歸藏 PPT Skill」

这步是亮点——你不需要手动装任何东西，直接把下面这段话复制粘贴发给 Agent：

帮我安装 guizang-ppt-skill 这个 Claude Code skill。

请按下面步骤做：确保 ~/.claude/skills/ 目录存在（不存在就创建）；执行 git clone https://github.com/op7418/guizang-ppt-skill.git ~/.claude/skills/magazine-web-ppt；验证：ls ~/.claude/skills/magazine-web-ppt/ 应该看到 SKILL.md、assets/、references/ 三项；告诉我安装好了，之后我说"做一份杂志风 PPT"之类的话就会触发这个 skill。

Agent 会自动执行 git clone、创建目录、验证文件，全程不用你操心。这就是 Agent 的能力——不只会聊天，还能帮你跑命令、装插件。

这套 Skill 是 **歸藏**（@op7418）做的，审美非常在线——「电子杂志 x 电子墨水」风格的横向翻页网页，自带 WebGL 流体动效和衬线排版，效果堪比专业设计师出品。推荐去 GitHub 给他点个 ⭐️ Star~

github.com/op7418/guizang-ppt-skill

**❹**  发送 PPT 制作指令

安装好后，告诉 Agent 你想做什么 PPT 就行。比如我们发的是：

Cherry Studio 已经支持 DeepSeek V4 模型了，以 Cherry Studio 的口吻帮我做一份介绍 DeepSeek V4 新模型的 PPT，面向 AI 爱好者和 Cherry Studio 的用户，20 页，如果需要使用图片，请用公开的图片素材，图片直接放在 PPT 里。

然后等着就好~ Agent 会自动触发 PPT Skill，开始规划内容、设计页面、编写代码。

**❺**  等待生成，打开预览

整个过程大概几分钟。完成后 Agent 会告诉你文件路径，浏览器打开就能看到成品。支持键盘方向键翻页、鼠标滚轮滚动，按 ESC 还能进入缩略图全览模式。

**❻**  惊喜：没给图也自动配图了

最让我们惊喜的——即使没有提供任何图片素材，Agent 也会用 SVG 和 CSS 生成与内容匹配的装饰图形。讲架构时画流水线图，讲数据时做大字报，跟着内容走的、风格统一的视觉元素。

Agent 自动生成的数据可视化和架构图示

 

效果怎么样？

说实话，**超出预期**。

20 页内容完整、结构清晰。深色「靛蓝瓷」配色高级感拉满，标题用衬线字体（Playfair Display + 思源宋体），正文用无衬线体（思源黑体），层次分明。每页底部有页码导航，按 ESC 弹出缩略图全览，交互体验也没敷衍。

当然也有小瑕疵——个别页面存在文字重叠，是因为 20 页全自动生成时部分元素间距没有完全适配。但以「发两条消息就出来」的标准来看，这个完成度已经很能打了。

**拿来做团队分享、项目汇报、产品介绍，完全够用。**对细节有更高要求的话，还可以让 Agent 帮你微调排版。

 

顺便聊聊 DeepSeek V4

**V4-Flash** —— 速度快、成本低，适合日常对话、文案撰写、快速问答。响应很快，日常使用体验流畅。

**V4-Pro** —— 旗舰推理模型，擅长复杂任务。写代码、数据分析、跑 Agent 都很稳。支持 high 和 xhigh 两档深度推理模式，让模型花更多时间「想清楚」再回答。

Cherry Studio 已完成适配，在「设置 → 模型服务 → CherryIN」里刷新模型列表就能看到。

* 歸藏 PPT Skill 仓库 : github.com/op7418/guizang-ppt-skill
* CherryIN 提供加速域名、国际域名和备用域名三种连接方式
* 生成的 PPT 是 HTML 文件，浏览器打开就能演示

 

想试试？

更新 Cherry Studio，配好 Deepseek V4

发两条消息就能搞定。

你用 Agent 做过什么有意思的事情？欢迎评论区聊聊~