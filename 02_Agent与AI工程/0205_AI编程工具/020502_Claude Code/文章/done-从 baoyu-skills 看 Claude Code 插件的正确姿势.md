> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 从 baoyu-skills 看 Claude Code 插件的正确姿势
author: 知识药丸
date:
url: https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649475496&idx=1&sn=58149e48cc440df1320dfe4b4da14e42&chksm=8f8d5c3db6215dd2c906cbece200218cde740ca5164d1217c5b5f35de20ae4c4bcb83060a28b&mpshare=1&scene=24&srcid=0122KTWFLi3JOY1XTSDKjMeS&sharer_shareinfo=2046ddef3dd46137429902bda98052f8&sharer_shareinfo_first=2046ddef3dd46137429902bda98052f8#rd
---

👀 最新、最有用的AI编程姿势，总来自「知识药丸」

最近在社区发现了一个很有意思的项目——baoyu-skills。这是**宝玉老师为 Claude Code 打造的技能集，专门用来解决"内容理解 → 生成视觉物料 → 导出压缩 → 自动发布"这一套完整的工作流。**

我花了点时间把这个项目的报告仔细研读了一遍，发现它不仅仅是个"能用"的工具，更重要的是，它展示了如何真正把 Claude Code 的插件机制用到极致。这篇笔记，我就从学习者的角度，聊聊这个项目里让我印象深刻的设计思路。

[《贾杰的AI编程秘籍》](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649474437&idx=1&sn=4ae1516afc0ec71b7638dd79daaae2cb&scene=21#wechat_redirect)付费合集，共10篇，现已完结。30元交个朋友，学不到真东西找我退钱；）

以及我的墨问合集《100个思维碎片》，1块钱100篇，与你探讨一些有意思的话题（文末有订阅方式

---

 

### Claude Code 本质上是什么?

在聊 baoyu-skills 之前，我们得先搞清楚 Claude Code 是个啥。

Claude Code 是 Anthropic 推出的命令行级 AI 编程助手。注意，它的定位不是"聊天工具"，而是一个能在你的代码仓库里执行高权限、可上下文感知的工程任务的 Agent。

你可以把它理解成一个"住在终端里的智能同事"——它能直接编辑文件、运行命令、创建提交，甚至通过 MCP(Model Context Protocol)连接 Google Drive、Slack 这些外部服务。更妙的是，它支持通过插件(Plugin)和技能(Skill)来扩展能力。

baoyu-skills 就是这样一套扩展能力的集合。

### 插件市场:不只是"技能包"

第一个让我眼前一亮的点是它的**插件市场设计**。

baoyu-skills 把所有能力按职责拆成了三大插件:

* • `content-skills`:内容生成与发布(小红书信息图、专业信息图、封面、幻灯片、漫画等)
* • `ai-generation-skills`:AI 生成后端(Gemini Web、通用图片生成)
* • `utility-skills`:工具链(X 转 Markdown、图片压缩)

这种拆分很聪明。用户可以按需安装，比如你只想做内容转换，就只装 `utility-skills`;如果要跑通从生成到发布的完整流程，就三个都装。这比把所有功能打成一个大包要灵活得多。

更关键的是，整个注册流程通过 `.claude-plugin/marketplace.json` 完成。这个文件定义了仓库元信息、插件列表、每个插件包含的技能。用户只需要一条命令:

```
/plugin marketplace add jimliu/baoyu-skills
```

就能把整个市场注册进来，然后在 Claude Code 里用 `/plugin` 命令浏览、安装想要的插件。这种"约定优于配置"的方式，让发布和使用都变得非常简单。

### "先固化理解，再生成"的工作流

第二个让我印象深刻的是它的**内容生成工作流**。

以小红书信息图为例，整个流程是这样的:

1. 1. **输入**:传入文件或直接粘贴内容
2. 2. **保存源内容**:写入输出目录的 `source-*.md`
3. 3. **内容分析**:输出 `analysis.md`，明确主题、受众、信息结构、视觉机会
4. 4. **生成多个大纲**:保留多份 `outline-style-*.md`，用于对比选择
5. 5. **一次性确认**:在单个步骤里让用户选定风格、版式、比例、语言
6. 6. **生成 prompts**:写入 `prompts/NN-...-[slug].md`，保证可重复生成
7. 7. **调用图像生成**:生成图片到输出目录
8. 8. **完成汇总**:输出路径、文件清单、关键参数

这个流程最妙的地方在于**把理解和生成分离**。

传统的 AI 工具往往是"一次性黑盒":你给个输入，它吐个输出，中间发生了什么你根本不知道。但 baoyu-skills 把整个过程拆成了多个可见、可回溯的步骤。每一步都有对应的文件落盘(`analysis.md`、`outline*.md`、`prompts/*.md`)，这意味着:

* • 你可以随时检查 AI 是怎么理解你的内容的
* • 你可以修改中间产物，让后续生成更符合预期
* • 你可以用同样的 prompt 重新生成，保证一致性

这种"先理解，再生成"的模式，把 AI 从一个"黑盒魔法"变成了一个"可控的工具链"。

### 二维组合系统:风格 × 版式

第三个亮点是**小红书信息图的二维系统**。

它把"如何生成一张信息图"拆成了两个维度:

* • **style(视觉美学)**:`cute`、`fresh`、`warm`、`bold`、`minimal`、`retro`、`pop`、`notion`、`chalkboard`
* • **layout(信息密度/结构)**:`sparse`、`balanced`、`dense`、`list`、`comparison`、`flow`

这两个维度可以自由组合。比如你可以要一个 `cute × dense` 的可爱风格、信息密集的布局，也可以要一个 `minimal × list` 的极简风格、列表式布局。

为什么这么设计?因为"风格"和"版式"是两个相对独立的维度。风格决定了视觉感受(色彩、字体、装饰元素)，版式决定了信息如何组织(单栏、双栏、流程图)。把它们分开，就可以用更少的配置覆盖更多的场景。

更聪明的是，它还会根据内容自动推荐合适的组合。比如对于数据对比类内容，它会推荐 `comparison` 版式;对于步骤教程类内容，它会推荐 `flow` 版式。这种"给建议但不强制"的设计，既保证了灵活性，又降低了使用门槛。

### 统一的 EXTEND.md:个性化覆盖层

第四个让我觉得很巧妙的设计是**EXTEND.md 机制**。

所有技能都支持通过 `EXTEND.md` 文件自定义。扩展路径有两个，按优先级检查:

1. 1. `.baoyu-skills/<skill-name>/EXTEND.md`(项目级:团队特定设置)
2. 2. `~/.baoyu-skills/<skill-name>/EXTEND.md`(用户级:个人偏好设置)

这个机制解决了一个很实际的问题:**如何在不修改源码的情况下，让每个人都能用自己喜欢的方式使用工具?**

比如你是个设计师，每次生成信息图都想用固定的品牌色、字体、水印。那你就可以在用户级 `EXTEND.md` 里写上这些偏好，以后每次生成，这些设置就会自动叠加到默认配置上。

又比如你在一个团队里，大家都用统一的视觉规范。那你就可以在项目级 `EXTEND.md` 里定义团队的"品牌风格"，所有人生成的内容都会保持一致。

这种"默认配置 + 个性化覆盖"的分层设计，让工具既有通用性，又有可定制性。

### 输出目录的命名约定:细节见功力

第五个值得说说的是**输出目录和文件命名的约定**。

每次执行都会创建独立目录:

```
<skill-suffix>/<topic-slug>/
```

比如 `xhs-images/ai-future/`，或者 `cover-image/quantum-computing/`。如果目录已存在，会追加时间戳避免冲突，比如 `ai-future-20260122-143052`。

图片文件名必须包含序号和可读 slug:

```
NN-{type}-[slug].png
```

比如 `01-cover-ai-future.png`、`02-content-key-benefits.png`、`04-slide-architecture-overview.png`。

为什么要这么严格?因为这些文件不只是"生成出来就完事了"，它们还要被后续的脚本(比如合成 PDF、合成 PPTX)按序号收集、排序、处理。如果命名不规范，整个工作流就乱了。

这种"先定义约定，再基于约定实现自动化"的思路，在 Unix 哲学里叫"文本流处理"。baoyu-skills 把同样的理念用到了 AI 生成的产物上，让原本"一次性"的生成结果，变成了可以被工具链串联起来的标准化数据。

### 浏览器的两个关键角色

第六个有意思的设计是**浏览器在两类场景中的作用**。

1. 1. **认证与会话**:`baoyu-danger-gemini-web` 通过浏览器 cookies 登录并缓存，用于访问逆向的 Gemini Web API。
2. 2. **发布自动化**:`baoyu-post-to-x`、`baoyu-post-to-wechat` 通过 CDP(Chrome DevTools Protocol)驱动 Chrome，完成网页端编辑器操作。

为什么要用浏览器?因为很多平台的 API 要么不开放，要么有严格的限制。但是网页端的编辑器是公开的，任何人都能用。通过 CDP 控制浏览器，就可以模拟人工操作，实现自动化。

这里还有个很细节的设计:**剪贴板脚本**(`copy-to-clipboard.ts`、`paste-from-clipboard.ts`)。为什么要用剪贴板?因为很多 Web 编辑器不支持直接插入图片的 DOM 操作，但它们都支持"粘贴"。所以脚本会先把图片复制到系统剪贴板，再触发"粘贴"事件，绕过了 API 的限制。

这种"用标准工具解决非标准问题"的思路，在工程实践里很常见，但能做得这么优雅的不多。

### 版本管理的 release 工作流

最后说说**版本管理**。

仓库内置了一个 release 工作流 skill(`.claude/skills/release-skills/SKILL.md`)，要求发布时必须完成:

1. 1. 更新 `CHANGELOG.md` 和 `CHANGELOG.zh.md`
2. 2. 更新 `.claude-plugin/marketplace.json` 的版本号(semver)
3. 3. 同步更新 `README.md` 和 `README.zh.md`
4. 4. 将上述修改一次性提交(单个 release commit)
5. 5. 创建版本 tag

这看起来是个"规范"，但它的妙处在于**把规范变成了可执行的 skill**。

你不需要去记这些步骤，也不需要每次手动检查。你只需要告诉 Claude Code:"我要发布新版本"，它就会按这个 skill 的流程执行。如果某个步骤没做，它会提醒你;如果某个步骤做错了，它会帮你修正。

这就是我说的"把人的经验，变成机器的行为"。

### 总结

学习 baoyu-skills 这个项目，让我对 Claude Code 的插件机制有了更深的理解。

它不是简单地"写几个脚本、打个包、发出去"，而是从头到尾都在思考:**如何让工具既强大又易用?如何让流程既灵活又可控?如何让配置既通用又可定制?**

这些设计思路，不仅适用于 Claude Code，也适用于任何需要"把复杂流程工具化"的场景。

如果你也在用 Claude Code，或者在思考如何构建更好的工具链，这个项目值得你花时间研究。它不是最酷炫的，但它可能是最实用的。

### 参考资料

* • baoyu-skills GitHub 仓库
* • Claude Code 官方文档
* • Claude Code 插件市场指南

 

---

 坚持创作不易，求个一键三连，谢谢你～❤️

以及「AI Coding技术交流群」，联系 ayqywx 我拉你进群，共同交流学习～