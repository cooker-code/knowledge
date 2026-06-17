---
title: Chrome 145：Arc 护城河，被一个 Flag 轻松抹平
author: 浮之静
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzNjE2NTI3NQ==&mid=2247491387&idx=1&sn=00f4a21b75506c7fddde3dad43859c2f&chksm=e94bc271a0412d5252a4feb307d604c96812214a3285f042d512bad6965a009535ad16ec1288&mpshare=1&scene=24&srcid=011872cwM0XDs3oRBmPynXnn&sharer_shareinfo=c610406bd6927f3bf7f8d21761fd5a17&sharer_shareinfo_first=c610406bd6927f3bf7f8d21761fd5a17#rd
---

你所谓的差异化可能只是巨头眼中的设置项...

## 关于浏览器系列我已经写过太多文章了，包括我自己也在做这方面探索： * [深度思考：聊聊 AI 发展趋势](https://mp.weixin.qq.com/s?__biz=MzIzNjE2NTI3NQ==&mid=2247491010&idx=1&sn=4a2dc0ef22832ea003bb0192b2f2f6d4&scene=21#wechat_redirect) * [浅谈 AI 浏览器](https://mp.weixin.qq.com/s?__biz=MzIzNjE2NTI3NQ==&mid=2247490854&idx=1&sn=8bc51140f9c8a476abc7170a1a700afd&scene=21#wechat_redirect) * [ChatGPT Atlas 发布，AI 浏览器大乱斗...](https://mp.weixin.qq.com/s?__biz=MzIzNjE2NTI3NQ==&mid=2247490809&idx=1&sn=4805e031fc03ba68b3d04057666387fe&scene=21#wechat_redirect) * [Noi v1.1.0 发布，Agent 能力初显](https://mp.weixin.qq.com/s?__biz=MzIzNjE2NTI3NQ==&mid=2247491359&idx=1&sn=c521759276d913e58421bbb77a36be7d&scene=21#wechat_redirect) * ...

## 背景

2026 年初，浏览器的竞争叙事出现了一个值得注意的拐点：**Chrome 145 Beta** 在 `chrome://flags/#vertical-tabs` 下引入了实验性的 **Vertical Tabs（垂直标签页）**。它还不是默认开启的“正式发布功能”，但它释放了一个很强的信号：曾经由 Arc、Edge、Vivaldi 等产品用来做差异化的交互形态，正在被 Chrome 这种“默认入口”吸收进主流基线。

同时，Chrome 作为全球份额占比最高的浏览器（StatCounter 2025 年 12 月的全球统计中，Chrome 约 71%），一旦把某种交互做成“系统级可选项”，UI 创新的战略空间就会被进一步挤压——不是 UI 不重要，而是**仅靠 UI 作为主要护城河**会越来越难。

与此同时，曾经以“重塑互联网计算机”为愿景的 The Browser Company（TBC）也经历了明显的战略转向：Arc 的“美学与交互叙事”逐渐让位于 Dia 的“智能与工作流叙事”（[Arc: 一个试图改变历史的浏览器](https://mp.weixin.qq.com/s?__biz=MzIzNjE2NTI3NQ==&mid=2247485449&idx=1&sn=5e2cd633b65087fad0176603d9f37b9d&scene=21#wechat_redirect)）。而这条路线在资本层面也被明确加码——Atlassian 宣布以约 6.1 亿美元收购 TBC，并把 Dia 描述为面向知识工作的 AI 浏览器入口。

## 界面范式收敛

过去三十年里，浏览器 UI 的主范式极其稳定：顶部水平标签栏几乎成了默认答案。但随着宽屏显示器普及与 SaaS 复杂度提升，“横向堆标签”的成本越来越高。Chrome 145 Beta 对垂直标签页的引入，可以视作对这种结构性压力的一次回应：把“更适合列表管理”的形态纳入主流选项，而不再只属于少数极客产品。

### 从边缘极客到主流标配

垂直标签页曾是 Power User 逃离 Chrome 的常见理由之一。Arc 通过把标签、书签与控制区整合到侧边栏，做出了一种更“工作台化”的体验；Edge、Vivaldi 也长期把垂直标签作为增强生产力的选项。

而在 **Chrome 145 Beta** 中，这个能力以实验 flag 的方式出现：

1. 打开 `chrome://flags/#vertical-tabs` → Enabled
2. 重启后，右键标签栏 → **Move tabs to the side**

启用后，你会得到一个层级更清晰、可配合 **Tab Groups** 的侧边标签栏，从而缓解“开几十个网页后只剩 favicon、标题不可读”的经典痛点。

这类动作常被拿来类比 **“Sherlocking”**：巨头把第三方的关键卖点收编进默认平台，从而降低用户迁移到小众产品的动机（同步、扩展生态、安全策略等“平台红利”依然集中在主浏览器）。这种“设计模式趋同”，会显著削弱 Arc 等挑战者在“第一眼体验”上的独占优势。

📌 Sherlocking 现象

Sherlocking（被 Sherlocked）说白了就是：**平台方把某个第三方 App 的核心卖点“系统内置化”**（预装、免费、深度整合），导致原 App **从“必装”变“可有可无”**，收入/增长被瞬间抽干。

它之所以叫这个名字，来自早期 Mac 的一个著名往事：第三方工具 Watson（Karelia）做了很受欢迎的“信息聚合/搜索通道”体验；随后 Apple 在 Sherlock 3（2002 左右）把非常相似的能力做进系统/自家工具里，引发开发者圈对“被抄+被内置”的强烈讨论，于是“被 Sherlocked” 就成了行话。

为什么这种事对第三方杀伤力特别大？核心在于**平台优势不是“功能”本身，而是分发与权限**：系统内置通常意味着默认触达、0 成本试用、设置入口/系统级入口、以及更深的系统 API/权限（甚至你拿不到的 private API），结果就是平台方哪怕只做 “80% 够用”的版本，也足够覆盖大多数用户需求。

需要补一点：Sherlocking 不等同于“平台永远在恶意抄袭”。有时平台只是自然演进（系统功能变强是正常趋势），而第三方仍能靠更专业的能力、细分场景、跨平台、企业需求活下来；但风险点在于：当你的产品价值几乎完全绑定在“一个可被系统吞掉的单点功能”上，就非常脆弱。

开发者面对 Sherlocking 常见的应对思路是：把护城河从“单一 feature” 转到**难被内置替代的东西**（专业工作流、行业深度、跨平台、生态集成、团队协作、数据与服务、品牌/社区），以及降低对单一平台的依赖。

英文一句话版：**Sherlocking：when a platform owner ships a built-in feature that makes a third-party app’s core value obsolete.**

### 宽屏时代之必然

这种回归并非偶然：现代显示器更“宽”，网页内容两侧常有空白，但纵向像素依旧宝贵。把标签列表放到侧边栏，本质上是在把“更适合纵向排列的对象”（任务/页面列表）放回到纵向区域里。Arc、Edge 的先行证明了它的可用性，而 Chrome 的跟进则意味着：优秀交互往往会从“创新”逐渐变成“基础设施”，并最终写入主流产品的默认能力池。

## 创新者窘境

如果说 Chrome 代表稳健的“基线进化”，那 Arc 曾代表激进的“体验重构”。Arc 的式微与 Dia 的登场，集中暴露了初创公司在巨头挤压下的两难：**做差异化容易不通用，做通用又容易失去差异化**。

### 审美疲劳 & 实用主义回归

Arc 曾凭借 “Spaces”、“Boosts” 与强烈的拟物化交互赢得设计师与开发者群体的追捧。但随着时间推移，复杂度会变成规模化的阻力：更多用户并不想“重新学习如何上网”，他们只想更快完成任务。

Dia 在设计哲学上更像一次“反向修正”：整体外观更克制，更强调 AI 的工作流价值而非 UI 的惊艳程度。对早期 Arc 核心用户而言，这种变化自然容易被评价为“没那么惊艳”“不温不火”；但从产品策略上，它更像是在争取更大规模人群的可用性与可复制性。

### Atlassian 收购

Dia 的定位变化与 Atlassian 的收购是同一条逻辑链：Atlassian 明确把 Dia 描述为“面向知识工作”的 AI 浏览器入口，并希望它与 Jira / Confluence 等协作产品形成闭环。

在 Dia 的叙事里，关键不再是“更漂亮的浏览器”，而是“参与工作流的浏览器”：**AI chat + skills + memory** 让浏览器能基于标签页上下文与会话状态提供持续帮助。

不然你的电脑可能就是下面这种状态，一大堆堆叠的 Tab，让人崩溃...

在这一语境下，Dia 的“无聊”外观反而更契合企业环境：企业 IT 更在意安全、可控、标准化与成本，而不是炫目的视觉效果。于是“少打扰、少意外、少迁移成本”，可能在企业侧是加分项，而不是缺点。

## Google 下沉式回击

当初创公司还在尝试从“浏览器 + 外部 LLM API”里找缝隙时，Google 的策略更像把能力往下压到两层：

1. **直接把 Gemini 放进 Chrome 的交互入口**（工具栏/侧边等），用“当前页面 + 多标签上下文”回答问题、总结内容、处理重复操作；
2. **把 Gemini Nano 变成浏览器内建的本地模型能力**，并通过 Built-in AI APIs（Summarizer / Translator / Writer / Rewriter / Prompt 等）让网页与扩展可以在本机调用 AI。

这意味着 Chrome 不只是“壳”，而正在变成 Gemini 的一个原生入口与运行时。

Google 在 2025 年交出的成绩单更像“模型 × 产品 × 分发”的联动：

* **Gemini in Chrome**：强调“不切 tab 的浏览内助手”，并利用打开的标签作为上下文；
* **Built-in AI APIs / Gemini Nano**：把摘要、翻译、写作等能力下沉到本地；
* **Nano Banana / Nano Banana Pro**：作为 Gemini 体系内的图像生成与编辑模型，在 Gemini 产品与相关渠道提供（而非“Chrome 默认右键菜单内置”）；
* **NotebookLM**：继续强化“资料驱动、可溯源”的研究与写作场景（作为生产力工具链的一环）；
* **Google Antigravity[1]**：更像一个 agent-first 的开发平台/IDE，而不是“Chrome 内置的编程按钮”；它强调多 agent 协作与任务编排。
* ...

如果把它对标 Dia 这类“主要依赖云端模型 API 的 AI 浏览器”，Google 的优势更结构性：**默认入口 + 本地推理 + 账号体系与产品矩阵**，在延迟、成本、隐私叙事与分发上更容易占上风——哪怕只做到 “80% 够用”。

### Aluminium OS

如果说 AI 功能是软实力，那么 “Aluminium OS” 更像 Google 的硬核布局。多家媒体与招聘/消息源指出，Google 正在推进一个 **Android 与 ChromeOS 更深度融合**的计划，代号 Aluminium，目标时间窗口指向 2026 年（Google's new 'Aluminium OS' project brings Android to PC: Here's what we know[2]）。

在这种设想里，Chrome（以及 Gemini）不再只是浏览器层的能力，而可能更接近系统级“默认接口”：更统一的账号与数据通路、更稳定的端侧推理、以及更深的系统 API 编排空间。对 Dia、Zen Browser 这类第三方应用而言，压力点在于：它们必须运行在平台规则之上，而平台自带能力天然拥有更强的默认分发与更深的系统整合空间。

## 从“浏览”到“代理”

2025 年浏览器叙事的核心趋势，是从“被动浏览（Passive Browsing）”向“代理式浏览（Agentic Browsing）”迁移：衡量“好用”的标准，逐渐从“界面是否惊艳”转向“能否替你把任务做完”。

### 技能（Skills）& 意图识别

Dia 的 “Skills” 更像这一趋势的早期形态：把常见意图（总结、规划、写作等）做成可复用的技能入口，并强调 “chat with your tabs”。

而 Google 的 Project Mariner[3] 则更偏“研究原型/更自动化”的方向：它明确以浏览器为起点探索人机交互的 agent 形态，并强调多任务自动化与“Teach and Repeat”（示范后复用）的能力。

### 新挑战：安全 & 隐私

浏览器一旦获得“自主点击与输入”的能力，安全风险会被放大。最典型的就是 **提示词注入（Prompt Injection）** 与 **Confused Deputy**：网页内容可以夹带对 agent 的“隐形指令”，诱导其执行越权操作或泄露信息。

应对策略上，两边路线差异很明显：

* Dia 在安全说明中明确提出：不把 URL 原样传给 LLM，并且让密码等敏感表单字段、不可逆按钮对 agent “不可见”。
* Google 则在 Chrome 的增强保护（Enhanced Protection）里引入端侧 Gemini Nano 来提升对诈骗/危险站点的识别与拦截能力。

这意味着未来浏览器的安全竞争，很可能会变成一场 “AI 攻防战”。

## 思考

Chrome 145 Beta 引入垂直标签页（即便仍是实验 flag），象征意义大于功能本身：它在提醒市场，**UI 形态的差异化会不断被主流吸收**，而竞争焦点正在上移到 AI 能力、端侧运行时与生态整合上。

* **URL 的消亡与 Intent 的兴起**：随着 agent 能力增强，用户可能更少“输入网址”，更多“输入需求”，浏览器逐渐靠近“搜索引擎 + 操作系统”的混合体。
* **企业与个人的分化**：个人市场更像流量入口与默认工具；企业市场更像付费工作台与数据/权限/治理的容器（Atlassian 收购 Dia 的逻辑在这里非常典型）。
* **交互模式的隐形化**：垂直标签页只是阶段性形态；更长远的想象是“任务界面主导、传统 UI 退居二线”，由 agent 在后台执行，用户更多审阅结果与确认关键操作。

对于用户而言，Arc 的“短暂绚烂”确实代表了一段高密度的交互创新；但 Chrome 145 与 Dia 的出现，也在宣告一个更工业化的阶段：效率、标准化、智能自动化成为主旋律。浏览器不再只是通向互联网的窗口，它正在成为“工作流本身”的承载层。

---

其实不光是 Dia，很多 AI 创业者都在面临大模型/平台公司的挤压（做 PDF 解析、PPT 生成、写作助手的都能感受到）。Noi 也一样：不支持常规浏览器插件、各种小 bug、生态与兼容性都很难和 Chrome 这种“默认入口”硬碰硬——所以 Noi 选择走另一条路（Noi = System + AI + Terminal + Agent）。

比如下面这张图，Noi 正在将一切操作命令化（资源、配置等）。命令化既方便程序自动化，也能为 agent 操作铺路。

除了一切命令化外，Noi 还在将一切资源渲染 Tab 化：在 Noi 中打开 Terminal，就像打开一个浏览器标签一样简单。

一句话来说：**一切皆可 Tab、一切皆可命令，Agent 正在路上…**

### References

[1]

**Antigravity:***https://antigravity.google*

[2]

**Google's new 'Aluminium OS' project brings Android to PC: Here's what we know:***https://www.androidauthority.com/aluminium-os-android-for-pcs-3619092*

[3]

**Project Mariner:***https://labs.google.com/mariner*