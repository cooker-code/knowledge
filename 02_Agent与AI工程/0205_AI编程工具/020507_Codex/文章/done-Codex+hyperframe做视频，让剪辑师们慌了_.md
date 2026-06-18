> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020507_Codex/020507_核心知识点/Codex工程使用与沙箱边界|Codex工程使用与沙箱边界]]
---
title: Codex+hyperframe做视频，让剪辑师们慌了?
author: 鲸选AI
date: 点赞关注点赞关注
url: https://mp.weixin.qq.com/s?__biz=MzkyMjUxOTI0Mw==&mid=2247529594&idx=1&sn=59f388893998d481770ff49cebb33dd7&chksm=c0d1ab332f8eb1f9cc5480b4c52708e935a946727bde08ca4003ae00f6c43b6a02b0af52ba25&mpshare=1&scene=24&srcid=0513DgUpbwj1EaNRppaiD2s5&sharer_shareinfo=76cee3169905f48aabcd2a3a4d1108e5&sharer_shareinfo_first=76cee3169905f48aabcd2a3a4d1108e5#rd
---

这两天打开X，发现一个开源项目刷屏了——Hyperframes。GitHub上两天干了17.4k star，1.6k fork，Codex、Cursor、Claude Code的插件全线覆盖。

更重要的是，它打开了一条AI生成视频的新路线，不需要Seedance等视频大模型，而是用普通的语言模型就能生成可控的视频内容。直接上我们做的案例，大家看看效果怎么样。

通过案例可以看出，Hyperframes不是一个AI视频工具，它更像是给AI Agent量身定、基于 HTML 的视频渲染框架。它的核心理念特别直白——AI最擅长写的就是HTML，那为什么不让AI直接写HTML来生成视频？ 

# Hypereframes到底是什么

很多读者看到HTML，可能联想到我们最近做的HTML PPT案例集。

但大家千万不要先入为主这样想，不然你会认为，这不是Hyperframes做了个PPT录了个屏，就说自己能做视频了？！从而很难高看这个项目的未来发展前景。

简单说，Hyperframes是深圳一家出海数字人企业在2026年4月开发的项目，它未来做的通过代码、而不是图像元素构建视频。

不用碰PR、剪映等剪辑软件，让AI写一个HTML文件就行。

这个HTML文件本身就是完整的视频——什么时候出现什么画面、动画怎么做、背景音乐什么时候响、字幕怎么同步，全部写在HTML的属性里。最后一条命令转成MP4。

底层原理并不复杂。Hyperframes用Puppeteer驱动一个无头（不用登陆）Chrome，把你的HTML一帧一帧地截取下来，然后通过FFmpeg把这些帧串成视频，再叠上音频轨道。输出的结果是确定性的——同样的HTML，不管渲染多少次，出来的视频每一帧都一样。这对自动化流程来说非常重要，你不会遇到"怎么这次渲染和上次不一样"的问题。

### 跟Remotion到底有什么区别

说到HTML生成视频，绕不开Remotion。Remotion可以说是这个赛道的开创者，用React组件写视频。Hyperframes的README也很坦诚地标注了，哪些技术路线是从Remotion学来的，包括Chrome的启动参数、图像管道转FFmpeg流式传输、帧缓冲这些底层机制。但两者在最核心的地方分叉了——你用什么来写视频。

Remotion的选择是React组件。要做一个视频，得写TSX文件，意味着你得理解React的组件化思想——每一个画面是一个组件，每一段动画是一段逻辑，图片要import进来，样式要用styled-components或者内联style。最后还需要一个构建工具把你写的东西打包成JS bundle，再丢给Chrome渲染。

> 这中间多了一层概念转换：你得先想清楚视频的结构，然后翻译成React的组件树。

Hyperframes的选择就是纯粹的HTML加数据属性。你打开一个index.html，用data-composition-id定义这是哪个视频，用data-start和data-duration定义每个片段什么时间出现、持续多久，用data-track-index定义它在哪个轨道上（视频轨道、图片轨道、音频轨道）。

不需要构建工具，不需要打包，在浏览器里直接打开就能预览，预览的效果跟最终渲染一模一样。

这个差异对AI来说影响很大。AI写HTML和CSS的能力在过去一年里已经非常成熟了，你让任何一个主流代码模型写一个带动画的网页，它都能搞定。但如果让它写React组件，虽然也能写，但多了一步概念转换，出错概率更高，调试也更麻烦。Hyperframes说白了就是选了一条对AI最友好的路——直接写浏览器最原生理解的东西。

怎么安装

目前最方便的方式，是在 Codex 里直接安装官方插件。 方法一：Codex 插件安装 先安装 / 打开 Codex： https://chatgpt.com/codex/ 然后在 Codex 插件里搜索： HyperFrames by HeyGen

安装完成后，在对话框中直接使用： @hyperframes 就可以让 Codex 调用 HyperFrames 来生成视频。

方法二：Claude Code / OpenClaw / Cursor 等工具安装 如果你用的是 Claude Code、OpenClaw、Cursor、Gemini CLI 这一类 AI coding agent，可以用命令安装 skills： npx skills add heygen-com/hyperframes

> GitHub 地址： https://github.com/heygen-com/hyperframes

# 它到底能做什么

很明显hyperframes不适合做AI漫剧等赛道，具体适合做什么，我们来几个具体的例子。

你可以让AI把一个GitHub仓库的README变成一个讲解视频。在Claude Code里说"用/hyperframes看一下这个仓库 https://github.com/heygen-com/hyperframes，给我做个讲解视频"，它会自己读README，提取关键信息，按时间线组织成视频脚本，然后生成对应的HTML，最后渲染出视频。

你可以把一篇文章或者PDF变成一个短视频。说"用/hyperframes把这篇PDF做成一个45秒的演讲视频"，它会自动摘要内容，配上TTS朗读和字幕。

你可以把CSV数据变成一个动态图表比赛视频。说"用/hyperframes把这组CSV做成一个动画柱状图竞赛"，它就能生成一个数据驱动的可视化视频。

还有一个玩法，我觉得最能体现Hyperframes的思路。

你可以让AI在视频里模拟出一个真实的Codex界面，然后在这个界面上演出一系列操作——对话框里实时打字、点击发送、看到AI生成的PPT展示、切换到全屏播放。所有这些都在一个HTML文件里完成，用CSS动画和GSAP时间线驱动。最终渲染出来的视频看起来就像有人真的在操作Codex一样，但实际上一行剪辑软件都没碰。

如何使用： 

如果是在codex中使用： @remotion制作鲸格skills的宣传片，根据"D:\鲸格PPT-full-README.md"这个文档的内容

Claudecode中使用 /remotion制作鲸格skills的宣传片，根据"D:\鲸格PPT-full-README.md"这个文档的内容

视频质量很依赖提示词的精度。你描述得越细、越具体，出来的效果越好。但如果你只是说"做个好看的视频"，那大概率出来的东西很平淡。它不像image2那样一句话就能炸出一张大片级的画面，输出质量的上限取决于你输入的上限。

# 一个比较好用的提示词结构

我建议使用 HyperFrames 时，不要只写一句话。 

可以按下面这个结构来写：

```
 用 @HyperFrames 做一个视频。 【视频类型】 产品功能演示 / 工具介绍 / 教学短片 / 社媒短视频 【时长与画幅】 默认画幅，时长不超过 10 秒。 【核心目标】 展示这个 skills 的功能，让观众理解它能自动生成 PPT / 视频 / 演示内容。 开场展示 6 个 deck，做环绕动效。 平滑切换到真实感 Codex 白色首页。页面中央出现文字："我们该在鲸格 Skills 里做什么？"下方输入框模拟真人打字，输入：/鲸格skills 制作ppt"/鲸格skills" 要像插件标签一样显示，和普通输入文字明显区分。输入完成后点击发送，进入生成状态。画面中央出现 Codex 图标和鲸格 Skills 图标，做轻微跳动、闪烁、震动的生成中动效。 回到 Codex 聊天界面，出现一个小 PPT 演示框。鼠标从对话框滑动到 PPT 框并点击。镜头逐渐推进到 PPT 框内，让 PPT 演示占据主体面积。PPT 短短几页，设计要美观，信息展示合理。最后出现"鲸格 Skills"文字收尾。【风格要求】 整体拟真、高级，接近真实 Codex 产品界面。 界面切换使用平滑渐隐渐现。 所有动效清爽、自然，不要过度炫技。
```

最后： Hyperframes在这个时间点火起来，不是偶然。过去一年AI代码能力突飞猛进。模型写HTML和CSS的基本功已经极其扎实了。但问题在于，AI能写出很漂亮的网页，然后呢？网页的消费场景其实是有限的——不是每个HTML都需要有人去打开看。

Hyperframes相当于给AI的新技能找了一个全新的出口——做视频。互联网上对视频内容的需求远远大于对网页的需求。一条短视频的播放量可能是一篇文章阅读量的几十上百倍。所以这不是一个"用HTML写视频"的技术问题，而是一个"让AI的输出的内容能找到更大消费场景"的商业逻辑。

当然Hyperframes目前还处在早期，很多东西需要时间和社区去打磨。

未来可能的场景是，它会在非剪辑人群（技术、学生、创业者）中流行开来，这群人本身并不擅长剪辑，也没有很大的精力去拼接剪辑细碎的内容，正好有一种产出确定、效果OK的视频模式，于是自己输入一句话，就输出一个想要的视频了。

往期推荐：

[互联网“魔幻场”：AI正掀起一场复古运动](https://mp.weixin.qq.com/s?__biz=MzkyMjUxOTI0Mw==&mid=2247529573&idx=1&sn=9041d39161ca92ab8ff0fa0b6619f9fa&scene=21#wechat_redirect)



[AI学会梦里干活！Claude发布3大进化，Dario：单人公司10亿美金爆发](https://mp.weixin.qq.com/s?__biz=MzkyMjUxOTI0Mw==&mid=2247529551&idx=1&sn=79f99787f5508b12e129bdfac0c06ddf&scene=21#wechat_redirect)