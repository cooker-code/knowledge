---
title: Cursor 进阶教程：使用 OpenSkills 引入 Claude 官方技能
author: 前端AI行走
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU5MjYwMDgzNQ==&mid=2247487406&idx=1&sn=d1e949f1216b77988f61279e003510da&chksm=ffc31071ba066b4604e9914691d5923ea607bc9f5f0578d65644240e72f1f0057c678b708033&mpshare=1&scene=24&srcid=1212U9TxDrCdaFJCh1SwrOpt&sharer_shareinfo=46f37dc6c3f722e2e23db1a9c6e86432&sharer_shareinfo_first=46f37dc6c3f722e2e23db1a9c6e86432#rd
---

# 

前言：

Cursor/Trae/Qoder等AI编辑器虽然强大，但目前尚未原生集成类似Claude Code的"Skills"（技能）生态。

本教程将介绍一个开源项目**OpenSkills**，它能让你在Cursor、Trae、Qoder 等非 Claude Code 环境中，也能轻松使用 Anthropic 官方或自定义的 Skills。

主要以Cursor 编程工具为例子。

**核心原理**：

通过 `OpenSkills` 工具下载并管理 Skills，然后生成一份 `AGENTS.md` 指导文件。

Cursor/Trae/Qoder等AI助手会读取这份文件，再AI自我学习理解，从而学会如何调用这些工具。

---

## 准备工作：

## 

* **Node.js 环境：**

  确保你的电脑已安装 Node.js (因为需要用 npm)，建议起码 22.xx 版本或者最新版本。
* **终端**

  可以使用系统自带终端，也可以使用 Cursor 内置终端 ( `Ctrl+~`)。

---

## 

## 步骤一：安装 OpenSkills 工具

## 

我们需要先安装 `openskills` 这个命令行工具，它是管理 Skills 的管家。在终端输入以下命令进行全局安装：

```
npm i -g openskills
```

注意：这一步只需要做一次。

---

## 

## 步骤二：安装 Skills (以 Anthropic 官方库为例)

你可以选择将 Skills 安装到**当前项目**或**全局**。

### 方案 A：安装到当前项目 (推荐)

如果你只想让当前项目使用这些技能：

```
openskills install anthropics/skills
```

方案 B：安装到全局

如果你希望所有项目都能用：

```
openskills install anthropics/skills --global
```

操作提示：

运行命令后，OpenSkills 会克隆仓库。默认会全选所有 Skills。你可以通过 **空格键 (Space)** 来取消或勾选你具体想要的 Skills，然后按回车确认。

安装成功后，你会发现项目根目录下多了一个 `.claude/skills` 文件夹，里面就是下载好的技能代码。

得到类似结果如下：

*(进阶：你也可以安装其他第三方 Skills，只需将 `anthropics/skills` 替换为对应的 GitHub 仓库地址，例如 `openskills install 具体网站/custom-skills`)*

---

## 

## 步骤三：生成 AGENTS.md (关键步骤)

这一步是连接Skills 和 Cursor 的桥梁。我们需要生成一个 `AGENTS.md` 文件，告诉 Cursor 有哪些技能可用以及如何使用它们。

**(1)创建空文件**: 在项目根目录创建一个名为 `AGENTS.md` 的文件。

****(2)**同步配置**: 在终端运行：

```
openskills sync
```

****(3)**选择技能**: 运行命令后，再次通过空格键选择你想写入 `AGENTS.md` 的 Skills。

****(4)**完成**:确认后，OpenSkills会自动将技能的定义、使用方法写入 `AGENTS.md`。

**原理：Cursor的AI在回答问题时，会读取项目中的文件。当它看到**`AGENTS.md`时，就会理解：“哦，原来我有这些能力（比如搜索、画图等），如果用户需要，我可以按照文档里的格式去调用。”

---

## 

## 步骤四：在 Cursor/Trae 中调用 Skills

现在一切就绪，你可以像指挥官一样在 Cursor 的 Chat 或 Composer 中使用这些技能了。

**使用技巧**: 虽然 AI 可能会自动发现技能，但显式地告诉它调用哪个 Skill 效果更好。

### 示例 1：视频剪辑软件介绍页

> **Prompt**: "调用 frontend-design skills，用 HTML 开发一个视频剪辑软件的 SaaS 介绍页。以文件test-video.html 保存在当前项目"

### 

### 示例 2：个人博客原型

> **Prompt**: "调用 frontend-design skills，用 HTML 创建一个现代化的个人博客网站原型，包含首页、文章详情页、关于页面的完整博客。以文件test-my.html 保存在当前项目"

---

## 

## 

## 这是frontend-design文件里面的SKILL.md，里面其实就是提示词。内容信息如下：

## 

```
---name: frontend-designdescription: Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, artifacts, posters, or applications (examples include websites, landing pages, dashboards, React components, HTML/CSS layouts, or when styling/beautifying any web UI). Generates creative, polished code and UI design that avoids generic AI aesthetics.license: Complete terms in LICENSE.txt---This skill guides creation of distinctive, production-grade frontend interfaces that avoid generic "AI slop" aesthetics. Implement real working code with exceptional attention to aesthetic details and creative choices.The user provides frontend requirements: a component, page, application, or interface to build. They may include context about the purpose, audience, or technical constraints.## Design ThinkingBefore coding, understand the context and commit to a BOLD aesthetic direction:- **Purpose**: What problem does this interface solve? Who uses it?- **Tone**: Pick an extreme: brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, soft/pastel, industrial/utilitarian, etc. There are so many flavors to choose from. Use these for inspiration but design one that is true to the aesthetic direction.- **Constraints**: Technical requirements (framework, performance, accessibility).- **Differentiation**: What makes this UNFORGETTABLE? What's the one thing someone will remember?**CRITICAL**: Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work - the key is intentionality, not intensity.Then implement working code (HTML/CSS/JS, React, Vue, etc.) that is:- Production-grade and functional- Visually striking and memorable- Cohesive with a clear aesthetic point-of-view- Meticulously refined in every detail## Frontend Aesthetics GuidelinesFocus on:- **Typography**: Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial and Inter; opt instead for distinctive choices that elevate the frontend's aesthetics; unexpected, characterful font choices. Pair a distinctive display font with a refined body font.- **Color & Theme**: Commit to a cohesive aesthetic. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes.- **Motion**: Use animations for effects and micro-interactions. Prioritize CSS-only solutions for HTML. Use Motion library for React when available. Focus on high-impact moments: one well-orchestrated page load with staggered reveals (animation-delay) creates more delight than scattered micro-interactions. Use scroll-triggering and hover states that surprise.- **Spatial Composition**: Unexpected layouts. Asymmetry. Overlap. Diagonal flow. Grid-breaking elements. Generous negative space OR controlled density.- **Backgrounds & Visual Details**: Create atmosphere and depth rather than defaulting to solid colors. Add contextual effects and textures that match the overall aesthetic. Apply creative forms like gradient meshes, noise textures, geometric patterns, layered transparencies, dramatic shadows, decorative borders, custom cursors, and grain overlays.NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character.Interpret creatively and make unexpected choices that feel genuinely designed for the context. No design should be the same. Vary between light and dark themes, different fonts, different aesthetics. NEVER converge on common choices (Space Grotesk, for example) across generations.**IMPORTANT**: Match implementation complexity to the aesthetic vision. Maximalist designs need elaborate code with extensive animations and effects. Minimalist or refined designs need restraint, precision, and careful attention to spacing, typography, and subtle details. Elegance comes from executing the vision well.Remember: Claude is capable of extraordinary creative work. Don't hold back, show what can truly be created when thinking outside the box and committing fully to a distinctive vision.
```

## 翻译成的中文：

## 

```
---名称：前端设计描述：创建独特的、生产级的前端界面，具有高设计质量。当用户请求构建 Web 组件、页面、工件、海报或应用程序时 (例如网站、登录页面、仪表盘、React 组件、HTML/CSS 布局，或者在为任何 Web UI 设计样式 / 美化时), 可以使用这项技能。生成富有创意、优雅的代码和 UI 设计，避免通用的 AI 美学。License:LICENSE.txt 中的完整术语---这项技能指导创建独特的、生产级的前端界面，避免通用的 “AI 垃圾” 美学。实现真实的工作代码，特别注重美学细节和创意选择。用户提供前端需求：要构建的组件、页面、应用程序或界面。这些需求可能包括与目的、受众或技术约束相关的上下文。## 设计思维在编写代码之前，要理解上下文，并遵循 BOLD 的审美方向：目的：这个界面解决了什么问题？谁在使用它？**Tone**: 选择一个极端：极简主义、极大主义混沌、复古未来主义、有机 / 自然、奢华 / 精致、游戏 / 玩具般、编辑 / 杂志、极简主义 / 原始、装饰艺术 / 几何、柔和 / 粉彩、工业 / 实用等。有如此多的风格可供选择。使用这些作为灵感，但要设计一种符合审美方向的风格。限制：技术要求 (框架、性能、可访问性)。差异化：是什么让它令人难忘？有人会记住的唯一一件事是什么？批判性：选择一个清晰的概念方向并精确执行。大胆的极大主义和精致的极小主义都能发挥作用 —— 关键在于意图，而非强度。然后实现工作代码 (HTML/CSS/JS、React、Vue 等), 即：生产级和功能性视觉上令人印象深刻且难忘- 具有凝聚力和清晰的审美视角- 在每个细节上都精心打造## 前端美学指南重点关注：- **Typography**: 选择美观、独特且有趣的字体。避免 Arial 和 Inter 等通用字体；选择能够提升前端美学的独特选择；意想不到、富有特色的字体选择。将独特的显示字体与精致的主体字体相结合。- ** 色彩与主题 **: 致力于一致的美学。使用 CSS 变量以保持一致性。具有尖锐重音的主导色比羞怯、均匀分布的调色板更好。- **Motion**: 使用动画进行效果和微交互。为 HTML 优先考虑仅使用 CSS 的解决方案。可用时使用 Motion 库进行 React。专注于高影响力时刻：一个精心编排的页面加载和交错显示 (动画延迟) 比分散的微交互创造了更多乐趣。使用滚动触发和悬停状态来创造惊喜。空间构图：意想不到的布局。不对称。重叠。对角线流。网格破坏元素。丰富的负空间或可控密度。背景和视觉细节：创造氛围和深度，而不是默认使用纯色。添加符合整体美学的背景效果和纹理。应用创意形式，如渐变网格、噪声纹理、几何图案、分层透明度、戏剧性阴影、装饰边框、自定义光标和颗粒叠加。千万不要使用通用的人工智能生成的美学元素，如过度使用的字体系列 (Inter、Robo、Arial、系统字体)、陈旧的配色方案 (尤其是白色背景上的紫色渐变)、可预测的布局和组件模式，以及缺乏特定上下文特征的 cookie-cuter 设计。创造性地解读，做出意想不到的选择，让人感觉这些选择是真正为特定背景而设计的。没有设计应该是相同的。在光明和黑暗主题、不同字体、不同美学之间变化。永远不要在跨代的共同选择上趋同 (例如，Space Grotesk)。** 重要 **: 将实现复杂性与审美愿景相匹配。极简主义设计需要精心编写的代码，包含丰富的动画和效果。极简主义或精致设计需要克制、精确，并仔细关注间距、排版和细微的细节。优雅源于良好地执行愿景。记住：克劳德有能力进行非凡的创造性工作。不要保留，展示当你跳出固有思维框架，全身心投入到独特的愿景中时，真正能够创造出什么。许可证：专有许可证.LICENSE.txt 包含完整条款
```

## 

## 示例 2：网页转换成 PPT

## 

## 我看到了PPT这个skills, 于是我就让cursor 去将刚刚生成的网页转换成PPT

> 调用 pptx skills,将 test-video.html 保存成test-video.ppt 文件

## 结果如下：

## 

## 把网站拆分成几个网页，然后写了个脚本，执行脚本之后。得到PPT如下：

## 

## 从结论上看，Cursor 确实帮我完成了这项任务。不论效果好不好看可以后续继续优化的，只是它能够实现之前没有的技能。

## 

## 

## 专家心得：如何创建高质量 Skills

## 

## 如果你想自己编写 Skills，以下是来自网络上的几点建议：

## 

1. **从模仿开始**

   克隆官方 skills 仓库，让 AI 阅读它。AI 会通过官方的 `skill-creator` 快速学会如何创建一个规范的 Skill。
2. **标准化工作流**

   创建 Skill 的本质是将工作流标准化。你需要先在脑海中或文档里把工作流梳理清楚，不要试图偷懒。
3. **MVP (最小可行性产品) 思维**

   不要试图一次性完美。先做一个能跑通的 MVP Skill，发现问题后再迭代优化。多用 Git 管理版本。
4. **倒推法**

   对于复杂 Skill，不要试图一次性生成。可以先写出最终执行的脚本（比如 Python 脚本），测试无误后，再反向包装成 Skill 的接口定义。

---

## 

## 具体如何开发我暂未去实践过，后续有结论了再告诉大家如何开发。还是一些Skills 还是需要看下具体的文档说明，我们才知道这些Skills具体能够给我们哪些帮助。

## 

## 

## 总结

## 

通过 **OpenSkills** + **AGENTS.md** 的组合，我们成功打破了编辑器的限制，让 Cursor/Trae/Qoder也能拥有类似 Agent 的工具调用能力。可能具体的效果如何都不能打包票，至少可以使用其他的工具来实现我们开发人员的其他想法。

大家可以都去试试看！或者自己尝试着去写一个自己的AI Skills