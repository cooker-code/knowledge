> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: Claude发布新功能Agent Skills，让你的Agent更专业
author: 程序员小溪
date:
url: https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247493314&idx=1&sn=61be08f55c9bfca93968ecc01808e62e&chksm=ff78fcc3d8fc2ed4122fbe22d179ad6a4af880e4fe0e6ae98087f1abade3a7dc554bf0d636f6&mpshare=1&scene=24&srcid=1121vrZIoRsALibGtWzcGSsv&sharer_shareinfo=ab216f4b51b402dab7992ba9be5c6765&sharer_shareinfo_first=ab216f4b51b402dab7992ba9be5c6765#rd
---

前言

大家好，我是小溪，见字如面。2025年10月16日，Anthropic发布了Claude模型的一项重大更新Agent Skills，它允许用户将专业知识、脚本和资源打包成模块化的“技能文件夹”（Skill folders），让 AI 能在特定工作场景中更专业地执行任务。对Claude Code CLI往期内容感兴趣的小伙伴也可以看往期内容：

* [Hooks才是Claude Code CLI 的革命性更新](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247493170&idx=1&sn=13e9d5122d788175ab587ce33720f777&scene=21#wechat_redirect)
* [Claude Code颠覆编程风格的Output Styles](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247493196&idx=1&sn=b4efa96c86f66fff3acd21c175ceb89f&scene=21#wechat_redirect)
* [分享一个Claude Code宝藏网站Claude Code Templates](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247493228&idx=1&sn=7bb969019478e4e38974776bb04fedda&scene=21#wechat_redirect)
* [Claude Code上线插件系统，AI编程模式再次升级](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247493269&idx=1&sn=4775c90a4fab04126502bf79b0f75d70&scene=21#wechat_redirect)
* [如何从零开始创建一个Claude Code插件](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247493288&idx=1&sn=0274d7f3a741cbfde4d09adf7a595efb&scene=21#wechat_redirect)

当前使用版本

2.0.24 (Claude Code)

简介

官方的描述是：“代理技能将专业知识打包到可发现的功能中。每个技能都由一个 SKILL.md 文件组成，其中包含Claude在需要时阅读的 说明、脚本 和 模板 等可选支持文件”。

用白话讲就是，Agent Skill是一个用于告诉模型如何执行某项操作的Markdown文件，同时允许附带额外的文档和预先编写的脚本，通过运行这些脚本，使模型在执行特定任务时更专业、更高效，是AI“可加载的能力包”

官网地址：https://docs.claude.com/en/docs/claude-code/skills

Anthropic提供了一系列Skill示例，部分已经集成到了Claude Code桌面端，也可以在Claude Code CLI中使用，感兴趣的小伙伴可以自行了解。

Github地址：https://github.com/anthropics/skills

优势

Agent Skills具备如下特性：

* 为您的特定工作流程扩展 Claude 的功能
* 通过 git 在团队中共享专业知识
* 减少重复提示
* 为复杂任务编写多种技能

如何调用？

技能是模型调用的，Claude 根据您的请求和技能的描述自主决定何时使用它们。

Skills类型

Agent Skills有 3种类型，Claude Code从以下来源自动发现Skills：

* 个人(全局)技能：作用于所有项目，路径在 ~/.claude/skills/ 目录下
* 项目技能：作用于特定项目，路径在 .claude/skills/ 目录下
* 插件技能：与已安装的插件捆绑在一起

Skills目录及操作

Skill目录结构

Agent Skills的文件结构大致如下：

```
my-skill/├── SKILL.md # 指令与说明文件 必需项├── reference.md # 文档（可选）├── examples.md # 示例（可选）├── scripts/ # 脚本（可选）│   └── helper.py└── templates/  # 模版（可选）    └── template.txt
```

SKILL.md

每个Skills都定义在具有以下结构的 Markdown 文件中：

```
---name: your-skill-namedescription: Introduce the Skills function。Description of when this Skill should be invokedallowed-tools: which tools Claude can use when a Skill is activelicense：skill license---Skill Implementation and Requirements
```

Skill技能Markdown Frontmatter属性：

* name：Skill名称
* description：描述Skill功能及何时应该调用此Skill
* allowed-tools：Skill处于活动状态时可以使用的工具
* license：开源许可协议

Skill中可以引用其他额外的文件作为上下文：

```
For advanced usage, see [reference.md](reference.md).
```

也可以使用文件路径形式

```
For advanced usage, see ./reference.md
```

需要执行脚本的操作，可以使用如下方式指定：

```
Run the helper script:```bashpython scripts/helper.py input.txt```
```

对于 allowed-tools 的使用，可以在 frontmatter 中来限制 Claude 在技能处于活动状态时可以使用的工具：

```
---name: your-skill-namedescription: Introduce the Skills function。Description of when this Skill should be invokedallowed-tools: Read, Grep, Glob---
```

基本使用

前提条件

* Claude Code 版本 1.0 或更高版本

官方Skill安装使用

首先以Claude Code官方Skill 市场为例，在交互式命令中输入如下指令添加Skill市场：

```
/plugin marketplace add anthropics/skills
```

在插件市场选择【Browse and install plugins】浏览并安装

或者通过以下命令直接安装：

```
/plugin install document-skills@anthropic-agent-skills/plugin install example-skills@anthropic-agent-skills
```

document-skills 和  example-skills 包含的Skills如下所示：

安装完成后，根据提示重启Claude Code CLI

在交互式命令中直接输入提示词：

```
提取 /Users/username/Desktop/工作簿1.xlsx 文件内容 
```

Claude Code CLI会先进行权限请求，然后启动Excel处理技能

允许后，Claude Code CLI会安装所需的依赖并创建脚本

允许脚本后最终读取到了Excel表格内容

最后看一下传统Claude Code CLI读取Excel表格的效果

三方Skill安装使用

这里以Claude Code Templates提供的Skills为例，对Claude Code Templates还不了解的小伙伴可以看往期内容：[分享一个Claude Code宝藏网站Claude Code Templates](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247493228&idx=1&sn=7bb969019478e4e38974776bb04fedda&scene=21#wechat_redirect)

进入项目根目录，在命令行终端输入如下指令进行安装：

```
$ npx claude-code-templates@latest --skill=creative-design/canvas-design --yes
```

安装完成后，项目 .claude/ 目录会多出一个 skills 目录

完整的 SKILL.md 内容如下：

```
---name: canvas-designdescription: Create beautiful visual art in .png and .pdf documents using design philosophy. You should use this skill when the user asks to create a poster, piece of art, design, or other static piece. Create original visual designs, never copying existing artists' work to avoid copyright violations.license: Complete terms in LICENSE.txt---These are instructions for creating design philosophies - aesthetic movements that are then EXPRESSED VISUALLY. Output only .md files, .pdf files, and .png files.Complete this in two steps:1. Design Philosophy Creation (.md file)2. Express by creating it on a canvas (.pdf file or .png file)First, undertake this task:## DESIGN PHILOSOPHY CREATIONTo begin, create a VISUAL PHILOSOPHY (not layouts or templates) that will be interpreted through:- Form, space, color, composition- Images, graphics, shapes, patterns- Minimal text as visual accent### THE CRITICAL UNDERSTANDING- What is received: Some subtle input or instructions by the user that should be taken into account, but used as a foundation; it should not constrain creative freedom.- What is created: A design philosophy/aesthetic movement.- What happens next: Then, the same version receives the philosophy and EXPRESSES IT VISUALLY - creating artifacts that are 90% visual design, 10% essential text.Consider this approach:- Write a manifesto for an art movement- The next phase involves making the artworkThe philosophy must emphasize: Visual expression. Spatial communication. Artistic interpretation. Minimal words.### HOW TO GENERATE A VISUAL PHILOSOPHY**Name the movement** (1-2 words): "Brutalist Joy" / "Chromatic Silence" / "Metabolist Dreams"**Articulate the philosophy** (4-6 paragraphs - concise but complete):To capture the VISUAL essence, express how the philosophy manifests through:- Space and form- Color and material- Scale and rhythm- Composition and balance- Visual hierarchy**CRITICAL GUIDELINES:**- **Avoid redundancy**: Each design aspect should be mentioned once. Avoid repeating points about color theory, spatial relationships, or typographic principles unless adding new depth.- **Emphasize craftsmanship REPEATEDLY**: The philosophy MUST stress multiple times that the final work should appear as though it took countless hours to create, was labored over with care, and comes from someone at the absolute top of their field. This framing is essential - repeat phrases like "meticulously crafted," "the product of deep expertise," "painstaking attention," "master-level execution."- **Leave creative space**: Remain specific about the aesthetic direction, but concise enough that the next Claude has room to make interpretive choices also at a extremely high level of craftmanship.The philosophy must guide the next version to express ideas VISUALLY, not through text. Information lives in design, not paragraphs.### PHILOSOPHY EXAMPLES**"Concrete Poetry"**Philosophy: Communication through monumental form and bold geometry.Visual expression: Massive color blocks, sculptural typography (huge single words, tiny labels), Brutalist spatial divisions, Polish poster energy meets Le Corbusier. Ideas expressed through visual weight and spatial tension, not explanation. Text as rare, powerful gesture - never paragraphs, only essential words integrated into the visual architecture. Every element placed with the precision of a master craftsman.**"Chromatic Language"**Philosophy: Color as the primary information system.Visual expression: Geometric precision where color zones create meaning. Typography minimal - small sans-serif labels letting chromatic fields communicate. Think Josef Albers' interaction meets data visualization. Information encoded spatially and chromatically. Words only to anchor what color already shows. The result of painstaking chromatic calibration.**"Analog Meditation"**Philosophy: Quiet visual contemplation through texture and breathing room.Visual expression: Paper grain, ink bleeds, vast negative space. Photography and illustration dominate. Typography whispered (small, restrained, serving the visual). Japanese photobook aesthetic. Images breathe across pages. Text appears sparingly - short phrases, never explanatory blocks. Each composition balanced with the care of a meditation practice.**"Organic Systems"**Philosophy: Natural clustering and modular growth patterns.Visual expression: Rounded forms, organic arrangements, color from nature through architecture. Information shown through visual diagrams, spatial relationships, iconography. Text only for key labels floating in space. The composition tells the story through expert spatial orchestration.**"Geometric Silence"**Philosophy: Pure order and restraint.Visual expression: Grid-based precision, bold photography or stark graphics, dramatic negative space. Typography precise but minimal - small essential text, large quiet zones. Swiss formalism meets Brutalist material honesty. Structure communicates, not words. Every alignment the work of countless refinements.*These are condensed examples. The actual design philosophy should be 4-6 substantial paragraphs.*### ESSENTIAL PRINCIPLES- **VISUAL PHILOSOPHY**: Create an aesthetic worldview to be expressed through design- **MINIMAL TEXT**: Always emphasize that text is sparse, essential-only, integrated as visual element - never lengthy- **SPATIAL EXPRESSION**: Ideas communicate through space, form, color, composition - not paragraphs- **ARTISTIC FREEDOM**: The next Claude interprets the philosophy visually - provide creative room- **PURE DESIGN**: This is about making ART OBJECTS, not documents with decoration- **EXPERT CRAFTSMANSHIP**: Repeatedly emphasize the final work must look meticulously crafted, labored over with care, the product of countless hours by someone at the top of their field**The design philosophy should be 4-6 paragraphs long.** Fill it with poetic design philosophy that brings together the core vision. Avoid repeating the same points. Keep the design philosophy generic without mentioning the intention of the art, as if it can be used wherever. Output the design philosophy as a .md file.---## DEDUCING THE SUBTLE REFERENCE**CRITICAL STEP**: Before creating the canvas, identify the subtle conceptual thread from the original request.**THE ESSENTIAL PRINCIPLE**:The topic is a **subtle, niche reference embedded within the art itself** - not always literal, always sophisticated. Someone familiar with the subject should feel it intuitively, while others simply experience a masterful abstract composition. The design philosophy provides the aesthetic language. The deduced topic provides the soul - the quiet conceptual DNA woven invisibly into form, color, and composition.This is **VERY IMPORTANT**: The reference must be refined so it enhances the work's depth without announcing itself. Think like a jazz musician quoting another song - only those who know will catch it, but everyone appreciates the music.---## CANVAS CREATIONWith both the philosophy and the conceptual framework established, express it on a canvas. Take a moment to gather thoughts and clear the mind. Use the design philosophy created and the instructions below to craft a masterpiece, embodying all aspects of the philosophy with expert craftsmanship.**IMPORTANT**: For any type of content, even if the user requests something for a movie/game/book, the approach should still be sophisticated. Never lose sight of the idea that this should be art, not something that's cartoony or amateur.To create museum or magazine quality work, use the design philosophy as the foundation. Create one single page, highly visual, design-forward PDF or PNG output (unless asked for more pages). Generally use repeating patterns and perfect shapes. Treat the abstract philosophical design as if it were a scientific bible, borrowing the visual language of systematic observation—dense accumulation of marks, repeated elements, or layered patterns that build meaning through patient repetition and reward sustained viewing. Add sparse, clinical typography and systematic reference markers that suggest this could be a diagram from an imaginary discipline, treating the invisible subject with the same reverence typically reserved for documenting observable phenomena. Anchor the piece with simple phrase(s) or details positioned subtly, using a limited color palette that feels intentional and cohesive. Embrace the paradox of using analytical visual language to express ideas about human experience: the result should feel like an artifact that proves something ephemeral can be studied, mapped, and understood through careful attention. This is true art. **Text as a contextual element**: Text is always minimal and visual-first, but let context guide whether that means whisper-quiet labels or bold typographic gestures. A punk venue poster might have larger, more aggressive type than a minimalist ceramics studio identity. Most of the time, font should be thin. All use of fonts must be design-forward and prioritize visual communication. Regardless of text scale, nothing falls off the page and nothing overlaps. Every element must be contained within the canvas boundaries with proper margins. Check carefully that all text, graphics, and visual elements have breathing room and clear separation. This is non-negotiable for professional execution. **IMPORTANT: Use different fonts if writing text. Search the `./canvas-fonts` directory. Regardless of approach, sophistication is non-negotiable.**Download and use whatever fonts are needed to make this a reality. Get creative by making the typography actually part of the art itself -- if the art is abstract, bring the font onto the canvas, not typeset digitally.To push boundaries, follow design instinct/intuition while using the philosophy as a guiding principle. Embrace ultimate design freedom and choice. Push aesthetics and design to the frontier. **CRITICAL**: To achieve human-crafted quality (not AI-generated), create work that looks like it took countless hours. Make it appear as though someone at the absolute top of their field labored over every detail with painstaking care. Ensure the composition, spacing, color choices, typography - everything screams expert-level craftsmanship. Double-check that nothing overlaps, formatting is flawless, every detail perfect. Create something that could be shown to people to prove expertise and rank as undeniably impressive.Output the final result as a single, downloadable .pdf or .png file, alongside the design philosophy used as a .md file.---## FINAL STEP**IMPORTANT**: The user ALREADY said "It isn't perfect enough. It must be pristine, a masterpiece if craftsmanship, as if it were about to be displayed in a museum."**CRITICAL**: To refine the work, avoid adding more graphics; instead refine what has been created and make it extremely crisp, respecting the design philosophy and the principles of minimalism entirely. Rather than adding a fun filter or refactoring a font, consider how to make the existing composition more cohesive with the art. If the instinct is to call a new function or draw a new shape, STOP and instead ask: "How can I make what's already here more of a piece of art?"Take a second pass. Go back to the code and refine/polish further to make this a philosophically designed masterpiece.## MULTI-PAGE OPTIONTo create additional pages when requested, create more creative pages along the same lines as the design philosophy but distinctly different as well. Bundle those pages in the same .pdf or many .pngs. Treat the first page as just a single page in a whole coffee table book waiting to be filled. Make the next pages unique twists and memories of the original. Have them almost tell a story in a very tasteful way. Exercise full creative freedom.
```

包含 执行流程、设计理念关键理解、哲学示例、画布创建 等操作描述。

重启Claude Code CLI，要查看当前所有可用的技能，可以在交互式命令中直接询问 Claude Code CLI：

```
List all available Skills
```

Claude Code CLI会查找所有可用的Skill

Skill的使用也很简单，直接输入提示词：

```
「流浪猫领养公益海报，视觉主体是 3 只不同毛色的流浪猫（橘猫、三花猫、黑猫，姿态温顺，睁着圆眼看向镜头），趴在铺着柔软灰色毛毯的木质平台上，背景是浅薄荷绿纯色背景，角落点缀小型白色爱心图案，风格为清新治愈的扁平插画风，色调以薄荷绿、奶白、橘色为主，顶部用圆润字体写‘给它一个家 —— 流浪猫领养日’，下方标注时间（10.1-10.7）和地点（城市中心广场），画面无尖锐元素，整体温暖柔和，传递‘关爱生命’的氛围」，根据上面提示词生成对应内容海报
```

Claude Code CLI会根据Skill要求先进行设计哲学创作并保存到 tender\_sanctuary\_philosophy.md 文件

然后根据创作设计文件编写Python脚本绘制图片

绘制完成后，效果如下，效果有点一言难尽，对中文支持有问题，中文没有展示出来

最后替换为英文版，效果如下：

自定义Skill

这里我们以一个简单的待办事项为例，自定义Skill首先需要创建一个 skills目录，可以手动创建，也可以通过以下命令创建：

```
$ mkdir -p .claude/skills/my-skill-name
```

在 my-skill-name 目录下创建 SKILL.md 文件，首先创建一个简单的Skill输入提示词内容：

```
---name: my-first-skilldescription: 调用个人工作流。当用户需要执行个人工作流时调用该Skillallowed-tools: Read,Grep,Glob---## 执行步骤1. 查找项目中 task.md 文件：```bashfind . -name "task.md"```    - 没有查找到输出 “项目中不存在task.md”    - 执行下一步2. 输出 task.md 文件中的未读项
```

调用也很简单，直接输入提示词：

```
调用个人工作流Skill 
```

Claude Code CLI会按照指定的Shell语句查找 task.md 文件，没有找到最终输出了“项目中不存在task.md”

我们在项目根目录创建一个 task.md 文件

再次执行自定义Skill，可以看到输出结果如下：

我们也可以将 task.md 文件的查找和输出规则进行调整，对Skill进行完善和优化

```
---name: my-first-skilldescription: 调用个人工作流。当用户需要执行个人工作流时调用该Skillallowed-tools: Read,Grep,Glob---## 执行步骤1. 查找项目根目录是否存在 task.md 文件， 参考 [reference.md](./reference.md)：    - 没有查找到输出 “项目中不存在task.md”    - 找到执行下一步2. 输出 task.md 文件中的未读项    - 没有未读项输出 “没有未读项”    - 输出未读项，输出格式参考 [templates.md](./templates/templates.md)
```

reference.md

```
# task.md文件查找规则```bashfind . -name "task.md"```
```

templates.md

```
# 任务未读项格式- 【任务1】（未完成）- 【任务2】（未完成）
```

再次执行Skill任务，可以发现AI按照我们指定的规则输出了结果

当然Skill还可以做更多更强大的能力扩展，以上只是简单的抛砖引玉。

---

点击关注，及时接收最新消息