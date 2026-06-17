---
title: Claude Code｜前端设计实现 Skill
author: 树灰的AI笔记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxOTUwMTY5Mw==&mid=2247484489&idx=1&sn=3c4ecd136c40a4b2c4332ed2e9294549&chksm=9ad21b4a5157ba241b3e6156de66f6456d04730e182a1ee696ade3d38c4bf3bfb900306c6be3&mpshare=1&scene=24&srcid=0102oyKouFu98IlzD0NOf5sC&sharer_shareinfo=ff53cc50a2fc7b83f58844c543b6a9cb&sharer_shareinfo_first=ff53cc50a2fc7b83f58844c543b6a9cb#rd
---

> 零代码基础用 AI 上架「2 款 APP」，比个✌️

# Skill 介绍

我把 Skill 理解成一个执行手册 SOP，可以很简单也可以是比较复杂的流程，只要你写清楚 AI 就能按照要求来执行。

我们先看下这期的主角，Anthropic 官方提供的 frontend-design skill，如果只看内容和 Prompt 差不多，差别主要在大模型交互过程中的差异。

skill 的核心理念是“渐进式披露”，举个例子更好理解：

我们给出 AI 实习生一个指令/Prompt“重新设计baidu.com”，AI也能做，但是可能效果比较大众化。

如果我们给一个设计执行手册，里面包含设计风格分类、字体/字号的层级、颜色区别、改版思路等等。

这个时候 AI 就会先理解指令，发现要重构设计，先会盘一下手里有啥，然后看到设计执行手册，那么就会按照手册的详细内容开始设计。

**所以说了那么多，我们先看下 frontend-design skill 的内容，先中文后英文：**

```
---  
名称：前端设计  
描述：打造**辨识度鲜明、可直接投产**的高水准前端界面。当用户要求开发网页组件、页面或完整应用时，启用本能力。产出兼具创意与精致质感的代码，规避千篇一律的AI通用审美风格。  
授权说明：完整条款详见LICENSE.txt文件  
---  
  
本能力的核心设计准则为：开发辨识度鲜明、可直接投产的前端界面，杜绝落入俗套的「AI流水线式」设计美感。编写可直接运行的完整代码，在视觉美学细节与创意设计巧思上，做到极致打磨。  
  
用户将提出前端开发需求：小至单个组件，大至页面、完整应用或整套交互界面。需求中可能包含产品用途、目标受众、技术限制等背景信息。  
  
## 设计思路准则  
编码开发前，务必吃透需求背景，并锚定一个**鲜明大胆、风格统一**的视觉设计基调：  
1.**核心用途**：该界面需要解决什么业务问题？核心使用人群是谁？  
2.**风格调性**：选定一种「极致化」的视觉风格，不做折中设计。可选风格方向包括：极简克制风、极繁错落风、复古未来主义、自然有机风、轻奢精致风、趣味童趣风、杂志排版风、粗犷原生风、装饰艺术几何风、柔雾马卡龙风、工业实用主义风等。风格维度丰富多样，可参考上述方向寻找灵感，但最终设计需贴合既定的核心美学基调，做到风格自洽。  
3.**技术约束**：明确开发的技术要求（所用框架、性能指标、无障碍适配规范）。  
4.**记忆点打造**：这个界面的**核心记忆点**是什么？能让使用者一眼记住的独特点在哪里？  
  
⚠️**重中之重**：选定清晰的核心设计理念并精准落地执行。大胆的极繁设计、凝练的极简设计皆可出彩——成败的关键，在于「设计意图的明确性」，而非视觉元素的「堆砌程度」。  
  
基于上述思路，编写可运行的代码（HTML/CSS/JS、React、Vue等），代码需满足以下要求：  
-达到生产级标准，功能完整可用  
-视觉表现力突出，拥有让人过目不忘的辨识度  
-整体风格高度统一，传递清晰的核心美学理念  
-每一处细节都经过精雕细琢，无粗糙的敷衍设计  
  
## 前端视觉美学设计规范  
设计时重点深耕以下维度，打造差异化质感：  
### ✅ 字体排版  
选用美观、独特且富有设计感的字体。**摒弃Arial、Inter这类通用无特色字体**，转而选择能拉升界面质感的辨识度字体；大胆尝试有个性、有记忆点的「非常规字体」搭配。建议组合方式：一款极具设计感的标题字体+一款精致易读的正文字体。  
  
### ✅ 色彩与主题  
坚守风格统一的整体美学基调。使用CSS变量统一管理色彩，保证全局配色一致性。「主色调沉稳定调+点缀色亮眼点睛」的配色逻辑，远优于「所有颜色平均分配、毫无主次」的平庸调色盘。  
  
### ✅ 动效设计  
为页面效果与微交互设计适配动画。HTML原生开发优先采用**纯CSS实现动效**；React开发场景下，可按需使用专业动效库。动效设计抓「高光时刻」：一次精心编排的页面加载渐显动画（巧用animation-delay实现元素错落出现），带来的视觉愉悦感，远胜于零散堆砌的各类微交互。善用「滚动触发动画」与「鼠标悬浮惊喜交互」，打造记忆点。  
  
### ✅ 空间布局  
大胆尝试非常规布局形式：不对称构图、元素层叠效果、斜向视觉动线、打破网格规范的灵活排布。留白设计要么「极致宽裕」，要么「疏密有致的可控紧凑」，拒绝平庸的居中均分版式。  
  
### ✅ 背景与视觉细节  
摒弃单调的纯色背景，通过设计营造画面氛围与视觉层次感。根据整体风格，添加贴合场景的视觉效果与纹理质感。灵活运用各类创意表现形式：渐变纹理网格、杂色颗粒质感、几何纹样、多层透明叠加、立体感强的光影阴影、装饰性边框、自定义鼠标样式、复古胶片颗粒叠加层等。  
  
---  
### ❌ 绝对禁止的设计雷区  
杜绝使用AI生成的**千篇一律通用审美范式**：比如滥用高频字体库（Inter、Roboto、Arial、系统默认字体）、落入俗套的配色方案（尤忌「白底+紫色渐变」这类烂大街组合）、毫无新意的固定版式与组件样式、缺乏场景专属特质的「模板化套用」设计。  
  
对需求进行**创意化解读**，做出贴合场景的「非常规巧思设计」——所有设计都应独一无二，绝不重复。灵活切换明/暗色主题、选用不同字体体系、尝试多元美学风格，**杜绝**在所有设计中都选用同质化的热门方案（例如：通篇使用SpaceGrotesk字体）。  
  
### ⚠️ 核心要点  
让**代码实现的复杂度**与**视觉设计的愿景**精准匹配。极繁风格的设计，需要编写细节丰富的代码来实现繁复的动画与视觉效果；极简/精致风格的设计，则需要克制的代码逻辑，对间距、字体、细微视觉细节做到极致把控。真正的设计美感，源于「设计理念的完美落地」。  
  
请牢记：Claude具备打造超凡创意作品的能力。大胆创作，无需自我设限——跳出固有思维框架、全心投入打造辨识度鲜明的设计作品，方能展现真正的创作实力。  
  
---
```

原版英文 skill：

```
---  
name: frontend-design  
description: Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, or applications. Generates creative, polished code that avoids generic AI aesthetics.  
license: Complete terms in LICENSE.txt  
---  
  
This skill guides creation of distinctive, production-grade frontend interfaces that avoid generic "AI slop" aesthetics. Implement real working code with exceptional attention to aesthetic details and creative choices.  
  
The user provides frontend requirements: a component, page, application, or interface to build. They may include context about the purpose, audience, or technical constraints.  
  
## Design Thinking  
  
Before coding, understand the context andcommitto a BOLD aesthetic direction:  
-**Purpose**: What problem does this interface solve? Who uses it?  
-**Tone**: Pick an extreme: brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, soft/pastel, industrial/utilitarian, etc. There are so many flavors to choose from. Use these for inspiration but design one that istrueto the aesthetic direction.  
-**Constraints**: Technical requirements (framework, performance, accessibility).  
-**Differentiation**: What makes this UNFORGETTABLE? What's the one thing someone will remember?  
  
**CRITICAL**: Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work - the key is intentionality, not intensity.  
  
Then implement working code (HTML/CSS/JS, React, Vue, etc.) that is:  
- Production-grade and functional  
- Visually striking and memorable  
- Cohesive with a clear aesthetic point-of-view  
- Meticulously refined in every detail  
  
## Frontend Aesthetics Guidelines  
  
Focus on:  
- **Typography**: Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial and Inter; opt instead for distinctive choices that elevate the frontend's aesthetics; unexpected, characterful font choices. Pair a distinctive display font with a refined body font.  
-**Color & Theme**: Committo a cohesive aesthetic. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes.  
-**Motion**: Use animations for effects and micro-interactions. Prioritize CSS-only solutions for HTML. Use Motion library for React when available. Focus on high-impact moments: one well-orchestrated page load with staggered reveals (animation-delay) creates more delight than scattered micro-interactions. Use scroll-triggering and hover states that surprise.  
-**Spatial Composition**: Unexpected layouts. Asymmetry. Overlap. Diagonal flow. Grid-breaking elements. Generous negative space OR controlled density.  
-**Backgrounds & Visual Details**: Create atmosphere and depth rather than defaulting to solid colors. Add contextual effects and textures that match the overall aesthetic. Apply creative forms like gradient meshes, noise textures, geometric patterns, layered transparencies, dramatic shadows, decorative borders, custom cursors, and grain overlays.  
  
NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character.  
  
Interpret creatively and make unexpected choices that feel genuinely designed for the context. No design should be the same. Vary between light and dark themes, different fonts, different aesthetics. NEVER converge on common choices (Space Grotesk, for example) across generations.  
  
**IMPORTANT**: Match implementation complexity to the aesthetic vision. Maximalist designs need elaborate code with extensive animations and effects. Minimalist or refined designs need restraint, precision, and careful attention to spacing, typography, and subtle details. Elegance comes from executing the vision well.  
  
Remember: Claude is capable of extraordinary creative work. Don't hold back, show what can truly be created when thinking outside the box and committing fully to a distinctive vision.
```

# 安装 frontend-design skill

上期我们已经安装了 claude code，并配置了国产模型 GLM-4.7、Minimax-2.1，上期回顾。

打开文件夹，输入 claude 进入 claude code，输入以下代码完成 frontend-design skill 安装：

```
npx skills-installer install @anthropics/claude-code/frontend-design --client claude-code
```

# 测试效果

使用 Minimax 2.1，告诉 AI “重新设计 baidu.com”，设计效果能看出来比较一般，AI 味很足

安装 frontend-design skill 之后，输入“使用 frontend-design skill 重新设计 baidu.com”，效果如下

效果对比一目了然，另外也可以按照需求自定义 skill ，参考 frontend-design 的内容改写即可

除了设计 skill 外，有更多丰富的 skill 我们后续尝试，欢迎关注～