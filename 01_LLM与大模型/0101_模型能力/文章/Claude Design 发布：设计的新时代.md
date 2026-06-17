---
title: Claude Design 发布：设计的新时代
author: 赛博禅心
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNDQxOTU2MQ==&mid=2247515657&idx=1&sn=b5ec22b2fe2e05f40af6981b097c958b&chksm=c3cc70afbfc7d49bc493c8f78c44c00be3a65355b5c9f40766da30fa6e785bb3527ac217c60f&mpshare=1&scene=24&srcid=0418xROMsYbcg27k0co1JrSG&sharer_shareinfo=cf0c3b709be2123e921a94302471bf17&sharer_shareinfo_first=cf0c3b709be2123e921a94302471bf17#rd
---

PRODUCT

刚刚，Anthropic 发布 Claude Design，入口 `claude.ai/design`

只需要在输入框里输入：

帮我做一个暗色主题的交互图形，展示城市之间的文化流动。一个旋转的地球，城市之间用发光的弧线连起来

一句话输入，Claude 开始干活

Claude 开始干活。左侧把任务拆成 5 步：搭项目结构、做交互地球组件、加城市节点和连线、排版面和字体、加氛围光。右侧画布上，Every place has a story 的暗色着陆页从零长出来，地球转起来，`112 座城市`、`168 条航线`，一条一条连起来

Every place has a story，Claude 从零生成整个地球页面

可以拖着旋转，可以点击响应。真实能跑的界面

Brilliant 的产品设计师 Olivia Xu 说，他们最复杂的页面在别的工具里要写 **20 个以上** prompt 才能还原，Claude Design 里 **只要 2 个**

Datadog 的 PM 形容他们现在开会的样子：**「会还没散，原型已经能跑」**

## Claude 自己造调节面板

页面出来之后，点一下 Tweaks 按钮，输入：

给这个地球和连线加一套可调控件

Claude 把任务拆成 5 步，自己画面板

Claude 把活拆成 5 步：读现在的页面布局和组件、搭 tweaks 状态、做浮动面板 UI、加弧线和城市开关、加断点切换器。几秒之后，画布上出现一个悬浮的控件面板

Claude 自己设计的滑块面板

六个滑块：`Grid opacity 0.10`、`Grid density 10°`、`Country fill 0.07`、`Globe size 400`、`Rotation 11°/s`、`Tilt 21°`。下面还有 Outer rings 和 Back grid 的开关。这些参数没有在 prompt 里说过，是 Claude 自己判断「用户大概想调这些」

传统设计工具里，设计师定义 Design Token，下游拿到成品。Claude Design 里反过来，**Claude 主动把可变维度交出来**，让你拖

一句话做一个原型，Claude 自己决定哪些东西应该可调

## 再来一个例子

冥想 App：

做一个平静的手机冥想应用原型，字体要让人静下来，配色偏自然，版面干净

手机冥想 App 原型，自带调节面板

iPhone 上的冥想 App，Good morning Jared，今天的练习是 Forest Rain。Tweaks 面板里有七种主题色（Stone / Moss / Mist / Clay / Dusk / Sakura / Ink），字号可以拖

觉得主色调太亮，点 Comment 按钮，画个框，写一句「加一个深色模式」

一句评论，整个 App 翻成暗绿

整个 App 翻成暗绿，Tweaks 面板也同步变深色

## 文档也能直接排

只需要上传一个活动记录，对它说：

帮我把这个整理成一份好看的、能打印的欢迎册子，发给公司团建的人

活动记录变成 5 页打印册

5 页的横版册子出来了。封面大图、行程表、Big Sur 周边介绍、打包清单，统一的品牌调性

觉得封面图不对，点开 Comment 直接在图上画框，写「换成海岸线那张」

画框评论换图，右边还有精细到 CSS 级别的 Knobs 面板

旁边还有个叫 Knobs 的面板，比 Tweaks 更精细。Font 是 `Space Grotesk`，Size `48px`，Weight `700`，Line `50.4`，Tracking `-1px`。从字体、字号、字重、行高，到颜色、对齐、圆角、间距，一切 CSS 能调的都能调

## 顺便把 Deck 也做了

扔一个 `launch.doc` 进去，说：

把这些要点做成一份带品牌调性的 Deck

从产品要点到 8 页 Deck，自带数据可视化

8 页 Slide 从产品要点里长出来，serif 加 sans-serif 字体混搭，品牌色贯穿，数据页有动态柱状图、趋势线、三段版式。Traction 页上写着：启动以来接入 `3,200+` 工程团队，增长率 `78%`，平均评分 `4.9`。右边是去年 9 月到今年 4 月的增长曲线

## 做完了，能去哪

Export 菜单里有六条路，包括 Handoff 到 Claude Code

点 Export，菜单里六条路：项目 zip、PDF、PPTX、发到 Canva、独立 HTML、Handoff 到 Claude Code

选 Handoff to Claude Code，Claude Design 会把项目打包、生成一条 Fetch 指令、附上 readme，直接粘进本地 Claude Code 或者 Claude Code Web。工程那边拿到设计稿、设计意图、和原项目的组件约束

## 能拿它做什么

官方列了几类用法

设计师把静态稿转成可分享的交互原型，不用过 code review，也不用提 PR。PM 画出功能流程扔给 Claude Code 实现，或者交给设计师继续打磨。创始人和 AE 从粗略大纲出发，几分钟生成一份符合品牌的 Deck，导出 PPTX 或发到 Canva。市场做落地页、社媒素材、Campaign 视觉，再拉设计师收尾

设计师也可以一次铺出几个方向做选择

还有一类叫 Frontier Design，官方的原话是「任何人都能做带语音、视频、Shader、3D 和内置 AI 的代码原型」

## 底座和身份

Claude Design 出自 Anthropic Labs，底座是这周刚发布的 **Opus 4.7**

Anthropic Labs 1 月份正式扩展命名，专注孵化 Claude 的实验性产品形态。Instagram 联合创始人 Mike Krieger 从 CPO 挪过来领导 Labs，和 Ben Mann 一起带队。Claude Code、Cowork、MCP、Skills、Claude for Chrome，都是这种孵化模式的产物。Labs 挂名后，Cowork 是第一个 Research Preview，Claude Design 是第二个

## 工作流里的几个细节

Onboarding 阶段，Claude 读团队的代码库和设计文件，自动抽出一套 Design System。之后每个项目都会默认套用这套颜色、字体、组件。系统可以随时迭代，一个团队也可以维护多套

起点可以是文字描述，也可以上传图片和文档（DOCX、PPTX、XLSX），或者把 Claude 指到团队的代码库上。还有一个 Web Capture Tool，直接从网站抓元素，让原型看起来就是真实产品

协作走组织维度。文档可以设为私有，也可以给全组织开只读、评论或可编辑权限。多人同时打开时，会进入群聊式界面，大家一起跟 Claude 对话

## 三家早期客户

博客里放了三家的原文反馈

过去两年和 Anthropic 的合作让我们很享受，我们都专注于把复杂的事变简单。Canva 的使命一直是让所有人都能做设计，这意味着我们要把 Canva 带到想法开始的地方。从 Claude Design 里出来的稿子进到 Canva，就变成可编辑、可协作的设计，随时能改、能分享、能发布

Melanie Perkins，Canva CEO

Brilliant 的复杂交互和动画一直是原型里最痛的部分。我们最复杂的页面在别的工具里要 20 个以上的 prompt，用 Claude Design 只要 2 个。把设计意图打进 Claude Code 的 Handoff 里，从原型到生产的跳跃变得无缝

Olivia Xu，Brilliant 高级产品设计师

Claude Design 让我们的原型速度大幅加快。一场对话里，我们从粗糙想法做到能跑的原型。过去要一周 Brief、Mockup、多轮评审才能完成的事，现在一次对话搞定，结果还贴合品牌和设计规范

Aneesh Kethini，Datadog 产品经理

## 订阅和用量

Pro、Max、Team、Enterprise 订阅者都能用。Enterprise 默认关闭，管理员要去 Organization Settings 里打开

Claude Design 有 **独立的用量池**，和 Chat、Claude Code 分开计量，每周重置。超出后可以买 Extra Usage

Pro 适合偶尔出个东西。Max 5x 适合 PM、工程师偶尔做 Mockup。Max 20x 适合设计师和创意人员。Team 和 Enterprise 分 Standard 和 Premium 两种席位。Enterprise 如果走 Usage-Based 计费，Claude Design 按标准 API 费率计入账单，每人有一次性 credit 能用大约 20 个 prompt，7 月 17 日过期

## 在哪里能用

入口在 `claude.ai/design`

→ 官方公告：`anthropic.com/news/claude-design-anthropic-labs`

→ 演示视频：`youtube.com/watch?v=t_LBECIQQqs`

## 一个观察

Anthropic 现在的产品矩阵：Claude Code 管代码、Cowork 管桌面 Agent、Claude Design 管视觉。三条产品线并列，背后跑的都是 Opus 4.7

独立的用量池也说明，Anthropic 把 Claude Design 和 Claude Code 放到了 **同一级**

Labs 这套孵化机制上一次把 Claude Code 从实验推成主线，这一次轮到 Claude Design

至于下一个是什么....我们还不知道，或许是科学研究，或许是生物制药，也或许是其他...