> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: 适合很多公司和团队的 AI Coding 落地范式（三）
author: LV技术派
date:
url: https://mp.weixin.qq.com/s?__biz=MzI2MjM0MDE3MQ==&mid=2247484807&idx=1&sn=3e4c500867fcc74bf5fcb3268ea429a9&chksm=eb882fd747736524acc480191fab168c7efb7f29e31c957bcb22c530df223df2408f0775cf16&mpshare=1&scene=24&srcid=0116jrKD0IxAxcJwZILBt7fw&sharer_shareinfo=42a09c4a0b6eec113106c7feca1e5db6&sharer_shareinfo_first=42a09c4a0b6eec113106c7feca1e5db6#rd
---

大家好，我是 lv。

本文是「适合很多公司和团队的 AI Coding 落地范式」系列的第三篇。

前两篇见：

* [适合很多公司和团队的 AI Coding 落地范式（一）](https://mp.weixin.qq.com/s?__biz=MzI2MjM0MDE3MQ==&mid=2247484778&idx=1&sn=700a6012722bb18d42bb121c0df93067&scene=21#wechat_redirect)
* [适合很多公司和团队的 AI Coding 落地范式（二）](https://mp.weixin.qq.com/s?__biz=MzI2MjM0MDE3MQ==&mid=2247484792&idx=1&sn=bb061471c87fdec52dafc06a4afa60c0&scene=21#wechat_redirect)

在第一篇文章，我们把市面上的 AI Coding 产品划分为了两类：AI Coding 平台类产品 & AI Coding 基建类产品。

在第二篇，我们针对 AI Coding 平台类产品，探讨了适合很多公司和团队的 AI Coding 落地范式：AI Coding 平台工程化。

AI Coding 平台化工程可以让需求方（产品经理、设计师、运营人员等）直接生成符合公司设计规范 & 代码规范的代码。

第二篇的最后，我们也提出了一个问题：

AI Coding 平台化工程的**「缺点」**是什么？

**「缺点是」**：这种模式只适合特定场景：比如说邮件模版生成，营销页生成，而且主要是**「从 0 ～ 1」**的代码生成场景。

如果你所在的团队有大量类似场景的需求，那 AI Coding 平台再合适不过了，可以简单理解为它就相当于是**「AI 时代的低代码平台」**。

假设在一个从 **「1 ～ n」** 长期迭代的复杂项目中，需要基于已有模块进行功能迭代，基于 AI Coding 平台的工程化将会**「异常复杂」**。

因此，为了也能够在团队偏复杂场景下落地 AI Coding，我们本篇开始讨论：基于 AI Coding 基建的工程化落地范式。

## AI Coding 基建使用现状

市面上的 AI Coding 基建产品很多，比如 Cursor、Claude Code、Codex、Cline 等 ...

以 Cursor 和 Claude Code 为代表，支持的核心功能包括：Commands、Skills、Sub Agent、Mcp、Hooks 等。

> ❝
>
> 可参考 claude code 文档：https://code.claude.com/docs/zh-CN/sub-agents
>
> ❞

使用 AI Coding 基建以及以上核心功能，我们可以做到：

1. 针对非复用生码场景的 AI 辅助编码
2. 针对可复用生码场景的 AI 流程封装

第 1 种，大家遇到的场景不一样，没有可复用性借鉴的价值，主要是一些使用 ai coding 的技巧，如先 Plan 后 Coding、Spec Coding 等。

那我们在公司和团队内部主要就是做第 2 种，其实本质就是组合 Commands、Skills、sub agent、mcp、hooks 等能力，来实现：针对可复用生码场景的 AI 流程封装。

## AI Coding 基建工程化

很多公司和团队，在推广使用 AI Coding 基建的时候，可能会面临以下问题：

* **「团队协作」**：AI Rules 更新了，团队成员可能**「不知道」**，导致无法使用新的 AI Rules。
* **「规则重复」**：配置的 AI Rules 都是跟着项目走的，换一个同类型的项目，相同的生码场景需要新项目**「再配置一遍」**。
* **「工具割裂」**：不同小组可能混用不同的 AI Coding 基建，相同的生码场景无法在不同工具之间复用。（尤其是针对大型复杂团队来说）
* ...

为了让 AI Coding 基建能够更方便在团队内部落地使用，我们开始：**「AI Coding 基建工程化」**，产品形态为：**「Cli 工具」**。

Cli 工具主要包含两个核心功能：

1. **「统一给 AI Coding 基建提供外部数据」**

主要是通过 MCP Server 把公司内部的私有数据暴露给 AI Coding 基建，比如公司内部的私有组件库。

2. **「统一管理 AI Rules」**

针对公司内的不同项目 & 不同 AI Coding 基建提供不同的 AI Rules 初始化和更新能力。

比如在新项目的根目录下，执行: `cli init --cursor`，则生成针对这个项目配套 cursor 的生码 rules，执行 `cli update --cursor`，则更新这个项目配套 cursor 的生码 rules。

下面以实际的案例出发来聊聊 AI Coding 基建工程化的实现。

## Compoder CLi 案例

上一篇文章，我们主要展示的是 Compoder 作为 AI Coding 平台的一种产品形态，内部集成了 AI 生码的工作流 & 代码渲染器等。

其实，Compoder 还有一种产品形态：Cli 工具，这种产品形态代表的是 AI Coding 基建工程化的产物。

**「效果演示」**

我们同样以上一篇文章中的 Landing Page Codegen（基于私有组件生成规范代码的场景） 为例，来演示下 Compoder Cli 的效果。

1. 进入 landing page 的项目工程，在根目录执行：`compoder init`，根据 cli 提供的交互，选择 Landing Page Codegen，然后选择 Cursor 作为 AI Coding 基建

2. 打开 cursor chat，输入需求：`compoder landing page codegen 生成 https://github.com/IamLiuLv/compoder/ 的 landing page`

3. 等待代码生成完毕，查看效果

中间会调度对应的 mcp server 获取私有组件信息，然后生成代码。

**「实现原理」**

Compoder cli 做的事情就是：统一给 AI Coding 基建提供外部数据 & 统一管理 AI Rules。

**「统一给 AI Coding 基建提供外部数据」**

在 Compoder 内部，以 Landing Page Codegen 为例，我们管理了 Landing Page Codegen 的私有组件信息，比如组件名字、组件的描述信息、组件的 api 信息、组件的示例代码等。

为了让 AI Coding 基建能够访问到这些私有组件信息，我们使用 MCP Server 对外暴露 Compoder 内部管理的 Codegen 私有组件信息。

这个 MCP Server 对外暴露两个 tools：

* component-list: 获取 Landing Page Codegen 的私有组件列表，包括组件名字、以及组件的描述信息
* component-detail: 获取 Landing Page Codegen 指定组件的详细信息，主要包括组件的 api 信息、组件的示例代码等

这个 MCP server 的配置会在执行 `compoder init` 的时候，自动生成在项目中来，如我们选择的 AI Coding 基建是 Cursor，则会在项目中生成一个 .cursor/mcp.json 文件，内容如下：

假设在你公司团队内部，需要给 AI Coding 基建提供多种多样的外部数据，同样你可以通过这种工程化 Cli 来启动 MCP Server 对外暴露这些外部数据，使用 Tools、Resources、Prompts 等类型都可以，主流的 AI Coding 基建基本上都支持。

**「统一管理 AI Rules」**

在 Compoder 内部我们已经有了一套基于私有组件来生成代码的 workflow：设计组件 => 实现组件。

那怎么把这个流程在不同的 AI Coding 基建中执行呢？

不同的 AI Coding 基建，有不同的 Rules 格式要求，这里我们以 Claude Code 的 Skills 为例。

在执行 `compoder init` 的时候，我们选择 Claude Code 作为 AI Coding 基建，则会在项目中生成一个 .claude/skills/landing-page-codegen 文件夹，内容如下：

> ❝
>
> 这里的提示词细节就不展开讲了，感兴趣可以看：https://kyscj.xetslk.com/s/38PsyJ
>
> ❞

如果后续需要更新 AI Rules，则执行 `compoder update`，compoder cli 会根据 init 时候生成的 .compoderrc 文件中的配置，来更新对应的 AI Rules 文件。

如果你在公司内部，希望有一个统一的 AI Rules 管理方案，来针对不同场景 or 不同项目 or 不同 AI Coding 基建来 init & update AI Rules，可以参考 Compoder Cli 的这种实现方式。

---

以上，我们相当于学习了一套适合很多公司和团队的 AI Coding 落地范式：基于 AI Coding 基建搭建「符合公司内部研发流程」的工程化 Cli 工具。

这个工程化 Cli 主要用来：

* 统一给 AI Coding 基建提供外部数据
* 统一管理 AI Rules

你可以考虑在公司内部推广使用这套范式，来帮助团队成员更高效地使用 AI Coding 基建。

知道了这套范式后，我们再来思考下，哪些 Coding 场景可以用这套范式来实现，并在团队内部推广使用呢？

从岗位的信息流转来看：产品经理(prd) => 设计师(design) => 代码工程师(code)。

Compoder cli 代表的场景可以归纳为从产品经理（prd）直接 => 代码工程师（code）：**「prd to code」**，相当于把设计师和程序员的能力融入到了 compoder cli 生成的 ai rules 中。

但其实对于相对复杂一点的项目迭代来说，基本上需要配套设计师（design）的环节，设计师产出的设计稿是：**「从产品需求 -> 最终生产代码」** 一个很重要的桥梁。

就好比毛坯房装修，设计师产出的设计方案和装修方案，是确保装修效果符合业主预期的关键。

那从设计师（design）=> 代码工程师（code）：也就是**「design to code」**，如何在团队内部落地呢？

我在公司内部已实践落地了一年多的 **「AI 驱动的 design to code」** 方案，中间还成为 Design Engineer 角色，和设计师合作，直出设计稿对应的代码，交付给前端工程师进行集成。

目前实现了：基于设计稿生成多端（web 端、flutter app 端）、多技术栈（antd、mui、公司私有组件库, ...）的代码。

沉淀了一套比较通用的 design to code 方案，遇到其他的技术栈，也可以快速适配。

中间踩了很多坑，大家自己在尝试的过程中可能也遇到过的，比如：

* 使用市面上的诸如 figma mcp server 结合 Cursor、Claude Code 等 AI Coding 基建，但是还原度不高
* 怎么把设计稿节点映射为代码中的组件，尤其是私有组件场景
* 怎么把设计稿对应的静态资源引入到代码库，比如 image、svg 等
* 如何设计合理的 design to code agent workflow，保证生成的代码质量，适配团队不同项目、不同技术栈和不同的代码规范等。
* 如何尽可能自动化 design to code，保证人只需要审核关键节点即可。
* ...

不同公司团队的情况可能会不一样，有需求的伙伴，可以加我微信，1 v 1 咨询 & 陪跑落地。

我可以针对你实际情况来提供 design to code agent workflow 落地方案。

如果你也想知道 AI + 前端的更多可能性，探索 AI 时代下前端人员的转型之路，扫描下方二维码加我的微信，围观我的朋友圈，我会在圈内持续分享更多的 AI + 前端的前沿探索内容。