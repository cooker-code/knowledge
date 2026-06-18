---
title: Codex 学会自己验 UI 了！切 viewport、走断点、截图留证，前端验收最耗人力的活被 agent 接走
author: 未来算子
date: AGIAGI
url: https://mp.weixin.qq.com/s?__biz=MzY5NzI4MDI5MQ==&mid=2247484102&idx=1&sn=9dca252630400a9d560e982c0242ad0d&chksm=f53f5bee6b109b40fb476715efe38727607a7132ef4fe356ed5080e46e7b5bc57abd52d1aa31&mpshare=1&scene=24&srcid=0517Jv1Z1lKmMbicGZGRyzgI&sharer_shareinfo=f26930b1f5af763b00c1d505f55b917d&sharer_shareinfo_first=f26930b1f5af763b00c1d505f55b917d#rd
---

导读  
【导读】OpenAI Codex 团队成员 James Sun 5 月 13 日在 X 上宣布：Codex 现在可以在 in-app browser 中切换不同 viewport 尺寸，控制 device toolbar，沿不同 breakpoints 点击验证，长流程还会自动截图留证。帖子获得超过 11 万次浏览、750+ 点赞。前端验收里最耗人力的响应式走查，开始交给 agent 了。

## 一个前端老痛点，被 agent 接走了

每个写过响应式布局的前端开发者，大概都经历过这种循环：

改完代码，开 dev server，拉宽浏览器，缩窄浏览器，再拉宽，点一遍交互，换个宽度再点一遍，截图，发给同事，等反馈，再改，再截图……

这段流程里最折磨人的部分，从来都不在写 CSS 上。反复切 viewport、反复走断点、反复截图确认——然后发现某个宽度下按钮被挤出了屏幕，这才是前端日常的真实消耗。

5 月 13 日，OpenAI Codex 团队的 James Sun 在 X 上发了一条帖，直接命中了这个痛点。

▲ James Sun 宣布 Codex in-app browser 支持 viewport 测试（11 万+ 浏览，750+ 点赞）

## 四层能力，远超"切个宽度"

James Sun 的原帖信息量很大，拆开来看有四层：

**第一层：真能切 viewport。**

> "Codex can now use the in-app browser to test your app at different viewport sizes!"

「Codex 现在可以用 in-app browser 在不同 viewport 尺寸下测试你的应用了！」

对 agent 来说，这一步意味着从"只在编辑器里改代码"，跨到了"打开浏览器、切换设备尺寸、看真实渲染结果"。

**第二层：切尺寸的同时还会走交互流程。**

> "It will control the device tool bar and click through your app at different breakpoints to validate & iterate."

「它会控制 device toolbar，在不同 breakpoints 间点击你的应用来验证和迭代。」

这才是关键——viewport 切换和交互测试被绑在了一起。它会在不同断点下真正点按钮、走流程、看页面有没有炸，比静态截图多出了整条交互验证链路。

**第三层：长流程自动截图，留证据给你复核。**

> "If it's a long run, Codex will take screenshots at key moments during testing and show them to you at the end of the turn, so you can verify its work."

「如果测试流程较长，Codex 会在关键时刻自动截图，并在回合结束时展示给你，让你验证它的工作。」

很多 agent 工具最大的问题，就是它告诉你"我检查过了"，但你根本没法看到它到底检查了什么。James Sun 这里强调的恰恰是：证据可见，结果可复核。

**第四层：隐藏浏览器禁用动画，测试提速 1-2 倍。**

> "To speed up testing, Codex can hide the IAB to disable animations, and accelerate testing by 1-2x."

「为了加速测试，Codex 可以隐藏 IAB 来禁用动画，把测试速度提升 1-2 倍。」

这个细节说明团队已经在做 agent 执行浏览器 QA 时的工程优化，已经跨过了"先跑通 demo 再说"的阶段。

## 底座在 4 月就铺好了：browser use 先行

viewport 测试并不是凭空冒出来的新东西。往回看一个月，OpenAI 在 4 月就已经把 in-app browser 和 browser use 的基础打好了。

▲ OpenAI 官方文档：in-app browser 的定位与能力边界

OpenAI 开发者文档明确写了以下能力定义：

> "The in-app browser gives you and Codex a shared view of rendered web pages inside a thread."

「in-app browser 在会话中为你和 Codex 提供一个共享的网页渲染视图。」

> "Use it for local development servers, file-backed previews, and public pages that don't require sign-in."

「适用于本地开发服务器、文件预览，以及不需要登录的公开页面。」

也就是说，OpenAI 先解决的是"agent 在受控浏览器环境里看页面、操作页面、交付可复核结果"这个基础设施问题。viewport 切换、breakpoint 走查、截图留证——全都建立在这个底座之上。

▲ OpenAI Codex changelog（2026-04-23）：browser use 正式进入更新日志

4 月 23 日的 changelog 也确认了这一点：Codex app 已经能让 agent 操作 in-app browser，用来点击 UI、复现视觉 bug、验证本地修复。5 月 13 日的 viewport 更新，是沿着这条线继续往前推。

## 一个月的定位升级：从"视觉反馈"到"验收执行器"

Ars Technica 在 4 月 17 日的报道里，把 in-app browser 描述为一种可以"观察 Codex 在 web 体验上做了什么"的能力——用户还可以像设计评审工具一样，在页面特定区域留下评论和指令。

▲ Ars Technica（2026-04-17）：把 in-app browser 放在"视觉反馈与协作"的定位上

一个月后，James Sun 把"不同 viewport / different breakpoints / key screenshots"单独拿出来讲。产品定位已经从"视觉审阅器"往"半自动前端验收工具"推进了一步。

The New Stack 在 5 月 7 日的评测中，也已经把 in-app browser 和 computer use、PR review 并列，当作 Codex 真实工作流升级的重要组成部分。外部开发者和媒体已经开始用真实代码库测试它，把它当真实工具来评估了。

## 社区反应：兴奋、追问，还有吐槽

**开发者最兴奋的点：agent 终于开始验 UI 了。**

X 帖下面的回复里，大量开发者在表达同一个意思：一个能打开网页的 agent 根本不够用，他们要的是能切断点、走流程、拍证据、最后让人快速复看的 agent。viewport bug 往往只有在真正切换尺寸时才会暴露，光靠 DOM 断言根本看不出来。

**工程细节被追问得很深。**

有开发者（Molty）马上问：它到底是按真实 CSS media queries 找断点，还是按固定宽度如 375/768/1280 测？前端开发者一眼就能看出，viewport 测试的核心价值，在于"切的是你项目真正的风险断点"。

James Sun 的回答很坦诚：

> "Hmm we don't prompt specifically to look at css media queries..."

「我们目前没有特意 prompt 让它去看 CSS media queries……」

他说模型可能自己能理解，但还需要进一步测试。这个回答一方面说明功能仍在演进，一方面也说明 OpenAI 愿意公开承认当前边界。

**社区已经开始提"配置化"和"并行化"需求。**

Brett Lamy 提到希望 viewports 能写进 config.toml，甚至让不同 iframe 各自跑 subagent 并行测试。James Sun 回应说团队确实需要把 browser use 的并行化做得更好，还抛出了"把保存 viewports 做成 skill"的想法。

这轮讨论的方向值得注意：开发者已经把这个功能当工作流基建看，开始想怎么把它嵌进自己的日常流程。

## 三个硬边界，不能忽略

**边界一：目前只支持本地页面。**

James Sun 在回复里明确说了：

> "Not yet! In app browser is targeted for local for now."

「还没有！in-app browser 目前针对的是本地场景。」

> "For now, Chrome for authenticated websites and in app browser for local. Will converge in the future!"

「目前的路线是：Chrome 处理需要登录的网站，in-app browser 处理本地开发。未来会合并。」

外部登录态网站、cookie 依赖场景、浏览器扩展——这些都还在边界之外。

**边界二：截图准确性曾出过问题。**

▲ GitHub issue #19429：社区报告 in-app browser 截图区域抓错

既然"关键时刻截图"是这次更新的核心卖点之一，那截图准不准就直接关系到整条验收证据链能不能成立。GitHub issue #19429 表明，社区确实遇到过 CSS pixel 与 DIP 坐标不一致导致截图区域错误的情况。虽然 issue 已关闭，但它指向了一个根本问题：只要截图坐标不稳定，agent 验收就可能看似完整、实际失真。

**边界三：真实开发场景比 demo 复杂得多。**

同线程里有人抱怨 browser preview 仍有体验问题——点击某些按钮时行为异常；有人追问文件上传、权限、多 tab、pop-out 窗口等场景的支持情况。这些声音都说明：这条产品线有明确价值，但离"前端 QA 全自动替代人工"还有相当距离。

## 前端验收的"最后一公里"，开始被 agent 接管

把 4 月以来的动作串起来看，OpenAI 做的事情有一条清晰的线：

* **terminal / local environment**

  负责运行和改动
* **in-app browser**

  负责看到真实页面
* **browser use**

  负责在页面里点、看、测
* **关键截图**

  负责把执行过程变成可复核证据
* **Chrome extension / computer use**

  负责把能力扩展到更复杂的外部环境

viewport testing 的意义在于，它让"浏览器"这个模块从辅助展示，升级成了真正的前端验收执行器。

响应式 bug 是最典型的"代码看起来没问题，但页面一切换尺寸就出事"的场景。传统自动化测试容易只覆盖 DOM 或 happy path，而忽略视觉层的断点问题。这次更新补的恰恰是这一层：device toolbar、different breakpoints、key screenshots。

Playwright 没有过时，手工 QA 也不会消失。但前端验收里那段"切 viewport、走断点、截图确认"的重复劳动，确实开始有了一个可对话、可编排、会留证据的 agent 协作者。

这个方向的下一步已经在社区讨论中浮现了：保存 viewports 配置、skill 化、并行化测试、从 local 走向 authenticated websites。

Codex 开始接手的，恰恰是前端验收里最机械、最费眼睛、最容易遗漏的那一段活。

---

— END —

— END —