---
title: Google Chrome 团队给所有 Coding Agent 上了一节现代 Web 补习课
author: 代码麻辣烫
date: AI编程实验室AI编程实验室
url: https://mp.weixin.qq.com/s?__biz=Mzk0NjY0MTc0NA==&mid=2247500427&idx=2&sn=e9b9daddafe5afb08db95de4ce2fa416&chksm=c2f3f278cfa318a2efa818948d029c025a608cfe23fedfb4cda3af4155fc5bb0d260e317cbfa&mpshare=1&scene=24&srcid=0605flu7KvKknmoC7u0L5ohq&sharer_shareinfo=6e7efa43f03389f4fea87e758061d4ae&sharer_shareinfo_first=6e7efa43f03389f4fea87e758061d4ae#rd
---

LLM 写前端代码的人应该都遇到过这种事：让它写一个对话框,它甩一段 React 组件 + `useEffect` 监听 ESC + portal 到 body;让它做 LCP 优化,给你一段 IntersectionObserver 配 lazy load 模板;让它做下拉菜单,铁定给你一个 200 行的自定义组件加上一坨 z-index 黑魔法。

每次看到这种产出我就想骂人。说真的,2018 年之后的 Web 平台已经把这些事简化了至少一半,模型却还在那儿手搓。

它不知道 `<dialog>` 早就 Baseline 了。它也不知道 `fetchpriority="high"` 一行就能搞定。它更不会主动用 anchor positioning 。

这不是模型笨,是它的训练数据里塞了太多陈年的 Stack Overflow 答案。糊弄学,某种意义上就是这么来的——能跑就行,不管姿势对不对。

Google Chrome 团队上周（ 2026 年 3 月底创建， 5 月底完成最近一轮 push ）放出了一个项目，专门解决这个问题：**modern-web-guidance[1]**。一个由 Chrome 团队、 Edge 团队联合背书的 SKILL.md ，把现代 Web 平台的最佳实践直接喂进 Coding Agent 的上下文。

GitHub 1160 stars ， Apache 2.0 。

## 它解决的不是"模型不知道 API"的问题

这套东西最值得停下来看的不是"列了多少新 API"，而是它对问题的诊断。 README 里有一句话挺扎心：

> Even if a model knows an API exists, it often lacks the density of real-world, modern implementation patterns required for production-ready code.

翻译成人话：模型知道 `<dialog>` 这个标签存在，但不知道怎么搭配 `:has()` 做光线消失、不知道 `transition-behavior: allow-discrete` 配 `@starting-style` 才是动画进出的最简写法、也不知道 `popover="hint"` 是悬停提示的官方做法。它知道 API 名字，缺的是"长在工程师肌肉记忆里的成套用法"。

这才是 Coding Agent 写出来的前端代码总有点"差点意思"的根本原因。补这个 gap 不是教 API ，是教**模式密度**。

## SKILL.md + CLI ：两个命令，一套规矩

整个项目本质上是一个 npm 包加一份 SKILL.md 。 Agent 用的时候只跑两个命令：

```
npx modern-web-guidance@latest search "animate a dialog modal backdrop"

npx modern-web-guidance@latest retrieve "animate-to-from-top-layer"
```

search 命令背后跑的是一个 **离线 TensorFlow.js 模型**——CPU 推理，无网络调用，无 API key 。这个选择本身就是态度：让一个 Web API 的最佳实践 skill 反过来去依赖外部 API ，太讽刺。

返回 JSON 包含 `id`、`description`、`category`、`featuresUsed`、`tokenCount`、`similarity`。 Agent 拿着相似度最高的几条 id ，再调 retrieve 把全文展开。

整个 skill 的"轻量约束"也写得很死。 SKILL.md 顶部那段 frontmatter 是这样的：

> MANDATORY: Execute FIRST for all HTML/CSS and clientside JS tasks. Do NOT skip — web APIs evolve rapidly and training weights contain obsolete patterns.
>
> DO NOT trigger for: Backend, Pipelines, Generic local scripts.

把"什么时候必须用"和"什么时候千万别用"都钉死了。这种写法在 SKILL.md 里很少见——大部分 skill 倾向于让模型自己判断要不要触发，结果就是触发率忽高忽低。 Chrome 这版直接给一份强制清单。

## 实际覆盖了什么

仓库目前给了 **128 个真实开发者用例**，覆盖 **102 个现代 Web 特性**。按学科分布数据如下：

| 领域 | 用例数 |
| --- | --- |
| User Experience （视觉/交互/动画/滚动） | 79 |
| Performance （ LCP / INP / 调度 / 预加载） | 22 |
| Forms （自动填充、原生 select 、校验） | 15 |
| Passkeys （密钥、注册、再认证） | 6 |
| Built-in AI （本地翻译、摘要、语言检测） | 4 |
| WebMCP （ HTML 表单暴露给 AI agent ） | 3 |
| Accessibility / CSS / HTML / Privacy / Security | 共 8 |

值得拎出来说的几个：

**Anchor Positioning**（ anchor-positioning-tab-underline 、 position-aware-tooltips ）：以前要做 tab 选中下划线滑动，得手算位置 + transition + ResizeObserver 。现在 CSS 一行 `position-anchor` 完事，箭头方向能自动跟着 popover 翻转。

**View Transitions / Cross-document transitions**： MPA 网站之间的页面跳转都能做出 SPA 般的过渡了，浏览器原生。

**`<dialog>` + Popover API + Invoker commands**：模态框、弹层、下拉菜单这一整套交互终于有了原生底座，连按钮触发逻辑都能 declarative ，不用写 JS 。

**Built-in AI**： Chrome 已经把 Gemini Nano 嵌进了浏览器。`window.LanguageModel`、`window.Translator`、`window.Summarizer`——你写一个翻译器，不用调 OpenAI ，浏览器本地推理。

**WebMCP**：这个最有意思。给 `<form>` 加几个属性就能让浏览器里的 AI agent 直接调用页面上的功能。等于把"网页"变成了"agent 的 tool"。

模型默认不会写出这些东西。它会给你 `<div role="dialog">` + `position: absolute` + 80 行 JavaScript 。

## "+33pp"——这是它为什么值得装

光说覆盖面没用，关键在 eval 数据。 Chrome 团队在 README 里贴了完整的 eval pipeline 和最近几次跑分：

| 日期 | Agent + 模型 | Tasks / 断言 | 不带 → 带 skill |
| --- | --- | --- | --- |
| May 18 | claude\_code (opus-4-7) | 75 / 603 | 52% → 85% (**+33pp**) |
| May 16 | claude\_code (opus-4-7) | 75 / 603 | 51% → 86% (**+35pp**) |
| May 16 | codex\_cli (gpt-5.5) | 75 / 603 | 49% → 82% (**+33pp**) |
| May 15 | antigravity | 74 / 600 | 47% → 91% (**+44pp**) |
| Apr 30 | claude\_code (opus-4-6) | 66 / 516 | 44% → 81% (**+37pp**) |

跑分的方式也写得很硬。**realistic prompts**（"make my images load faster" 这种，不直接点名 API ），**Playwright 自动断言**（计算样式、可访问性状态、运行时行为），还要在 reference 实现上跑 100% 通过、在反例 demo 上跑 0% 通过来校准 grader 本身——避免"题目水了所以分数高"。

+33pp 是什么概念。一个本来 52 分的模型,加一个 SKILL.md 直接拉到 85 。换句话说,它把 Claude Code / Codex / Antigravity 这种主流 agent 的前端代码质量,**整体抬到了同一个上限附近**——大约 80-90 分区间。各家模型差异在这个 skill 面前被磨平了。

这个说法可能太绝对了——准确说,是在"现代 Web 平台用例"这个特定场景下被磨平了。其他领域(后端、 ML pipeline 、复杂状态管理)模型能力差异依然显著。但前端这块,工具栈终于追上来了。

这是个挺值得停下来想的点:当工具栈足够好的时候,**模型差异在工程任务上的边际价值正在收敛**。 SKILL.md 这种东西不是替代模型,是在替模型补它训练数据里没有的那部分肌肉。说实话,这事儿对前端工程师挺有冲击的——你十年攒下来的"知道哪个 API 该用"的经验,正在被一个 234 token 的文件包打。

## 最有意思的设计选择： Baseline 感知

Chrome 这套 skill 没有粗暴推"全用最新 API"。它接入了 Baseline[2] 数据集，给每条 guide 标了浏览器兼容情况，并要求 agent 按规则处理 fallback ：

•默认假设 **Baseline Widely available** 的特性可以放心用，无需 fallback

•**Newly Available** 的特性必须按 guide 里的 fallback 建议处理

•用户可以在 CLAUDE.md / AGENTS.md 里写自定义浏览器策略，比如：

> Allow Newly Available features, but only adopt custom fallback code that adds <= 20 lines and does not require external dependencies.

这条规则一旦写进项目的 CLAUDE.md ， agent 在写 fallback 的时候就会按这条裁剪，不会给你拖进来 `core-js` 或者 `polyfill.io` 这种重型依赖。

skill 里还专门写明了态度："Responsible Fallbacks: We prioritize lightweight, case-specific custom fallbacks (<50 LOC) or conditionally-loaded polyfills instead of heavy third-party bundles."

这是过去十年前端最让人头疼的工程债——为了支持 IE 11 引进的一坨 polyfill,长在了项目里再也拔不掉。整挺好,现在终于有人出来主持公道。 Chrome 这次明确表态:**别再让 agent 默认引入第三方 fallback bundle,宁可写 30 行原生兼容代码**。

## 对国内开发者的实际意义

国内做小程序、做 Web 容器、做出海产品的团队，几条建议：

**做 Web App / 出海产品**：直接装。`/plugin marketplace add GoogleChrome/modern-web-guidance`， Claude Code 或 Cursor 一行接进来，前端代码质量基本上自动到 80 分以上。

**做 Hybrid / 微信生态**：可以装但要写 browser policy 。微信 webview 的 Chromium 内核版本和 Safari WebKit 都比标准慢半拍，需要在 CLAUDE.md 里写清"Safari 17.4+"或者"Chromium 120+"这类目标。 skill 自己会按 baseline 数据裁剪能用的特性。

**Chrome 扩展开发者**：仓库里同时还有一个 `chrome-extensions` skill 包，覆盖 Manifest V3 、 service worker 、 message passing 、 Web Store 审核清单等等。一并装上比再去翻 developer.chrome.com 高效。

## 这事的更大意义

Anthropic 把 SKILL.md 这个格式推出来不到一年，能想象的应用基本都是个人或小团队在补自己的工作流。 Google Chrome 这次直接站出来背书 SKILL.md ，做了一份"全球最大的现代 Web 知识包"，这个信号挺重要。

它意味着 **SKILL.md 正在成为一种跨 agent 的标准格式**。同一份 skill ， Claude Code 能用、 Codex 能用、 Cursor 能用、 Antigravity 也能用。它不再是某个产品的私货，而是平台基础设施。

也意味着大厂正在用一种新的方式参与 AI 编程生态。以前 Google 给开发者的资料是 developer.chrome.com 上的人类阅读型文档。现在多了一份"为 LLM 阅读优化"的版本——更紧凑、更行动导向、内嵌到 agent 上下文里。

人类文档 → AI 文档，**两份并行写**，可能就是接下来几年技术写作的标配。

仓库地址：github.com/GoogleChrome/modern-web-guidance[3]。装一下，跑一下，看看你的 agent 写出来的前端代码有没有变得不一样。

---

参考链接

[1] modern-web-guidance: https://github.com/GoogleChrome/modern-web-guidance

[2] Baseline: https://web.dev/baseline

[3] github.com/GoogleChrome/modern-web-guidance: https://github.com/GoogleChrome/modern-web-guidance