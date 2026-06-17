---
title: html_anything：推荐一个开源项目，以及如何把MD到HTML的产品化机会变成现实
author: 产品经理的AI分身
date: sunshinesunshine
url: https://mp.weixin.qq.com/s?__biz=Mzk2NDIzOTk2NQ==&mid=2247486251&idx=1&sn=349913aa77b1b61192ffe5284a3ee917&chksm=c525e261f433f61a263dd2daca1b11447c616f57dff3a9ed71eb658732431b0faf5113d191fa&mpshare=1&scene=24&srcid=0515Bdeg2UmM0GThiQ9Amly8&sharer_shareinfo=01fdc6367d163691b51e5921a1e064de&sharer_shareinfo_first=01fdc6367d163691b51e5921a1e064de#rd
---

5月8号，Claude Code团队的工程师Thariq发了一篇文章，标题叫 Using Claude Code: The Unreasonable Effectiveness of HTML。核心观点就一句话，他已经不在工作中用Markdown了，全部让AI直接输出HTML。这篇文章一天之内冲上了Hacker News榜首，X上引发了上万人讨论。

我当时也参与了这场讨论，提了一个观点，[Markdown是记忆，HTML是表达：AI时代的双轨架构](https://mp.weixin.qq.com/s?__biz=Mzk2NDIzOTk2NQ==&mid=2247485916&idx=1&sn=79a894d85d1a2b57d55a5f81a1cd87cd&scene=21#wechat_redirect)MD是数据内容层，HTML是展示交互层，两者之间蕴含着产品化的机会。不是谁替代谁的问题，而是中间缺一个足够好的「翻译层」。

没想到三天后，就有人把这个「翻译层」做出来了。

## 推荐一个我最近发现的开源项目

5月11号，一个叫html-anything的项目出现在GitHub上，来自nexu-io团队。到5月13号，三天迭代到v5，11项核心任务全部完成。

它做的事情一句话就能说清楚，把任意输入，不管是Markdown、CSV、Excel、JSON还是SQL，通过本地AI agent转换成一份「可交付」的HTML。生成完就是读者看到的样子，不需要再手动调。

零API Key，直接复用你已有的Claude或Cursor订阅，边际成本为零。一键发布到公众号、推特、知乎，或者下载HTML和PNG。

但让我真正觉得「这东西有用」的，不是这些技术特性，而是它75套模板覆盖的使用场景。我按自己的理解分了个类，你可以看看有没有击中你的需求。

**写文章做内容的人**

杂志长文、博客文章、读书笔记、数字指南、编辑墨水风格的文档。如果你经常写公众号长文或者技术博客，doc-kami-parchment这套暖羊皮纸模板会让你的文章看起来比纯白Markdown高一个量级。magazine-poster则是报纸风长图海报，巨字serif标题加双栏正文，开起来像一份印好的Sunday paper。

**做PPT做分享的人**

这块是重头戏，20套Keynote PPT skill。瑞士国际主义风格适合正式商务场合，冷静理性学院派。编辑墨水风格适合创意分享，纸感印刷质感。小红书粉彩风格适合年轻化受众。还有Replit风格、Hermes赛博朋克风格、XHS纯白风格等等。每套都附带了example.html，从repo里双击就能看效果。

**做社交媒体内容的人**

推特分享卡（1600x900）、小红书图文卡、Spotify Wrapped风格卡片、Reddit风格卡片。如果你需要给社交媒体做配图，这些模板可以直接把一段文字变成一张设计感十足的分享图。

**做产品和工程的人**

Web原型（SaaS落地页、Dashboard、技术文档页）、PM规格书、工程runbook、周报、会议纪要、看板。这些是日常工作中最高频的文档类型，每类都有对应的模板和设计系统约束。

**做数据报告的人**

数据可视化报告、财务报告、定价页对比表。如果你经常需要把数据变成可读的报告，这些模板能帮你把CSV或JSON直接灌进一套排版精良的HTML里。

**做视频和动效的人**

Hyperframes视频帧脚本，10套帧模板，符合heygen-com/hyperframes规范，可以直接交给Remotion渲染成mp4。还有VFX文字光标特效、故障艺术标题帧、品牌Logo收尾帧。如果你在做短视频或者产品发布视频，这些帧模板能省不少时间。

**找工作的人**

极简简历模板，A4尺寸，设计系统约束下的排版比Word简历好看几个档次。

**做营销的人**

营销海报、邮件营销模板、Waitlist页面、产品发布活动PPT、投资者Pitch Deck。

你看完这个列表，如果有一个场景让你觉得「正好需要」，那这个项目就是为你准备的。

## 它站在哪些巨人的肩膀上

让我特别感兴趣的是这个项目的「引用图谱」。它没有闭门造车，而是非常坦诚地站在四个开源项目的肩膀上，每个都精准地取了最核心的能力。

第一个是nexu-io/open-design，这是同一个团队做的更大型项目，GitHub上已经4万star了。html-anything直接复用了open-design的agent检测层、设计系统模型和SKILL.md协议。agent检测这块，启动时扫描你PATH上已安装的coding agent CLI，包括Claude Code、Cursor Agent、Codex、Gemini CLI、GitHub Copilot CLI、OpenCode、Qwen Coder、Aider这8个，甚至还会扫描/.local/bin、/.bun/bin这些GUI启动时会漏掉的目录。

第二个是mdnice/markdown-nice，也就是墨滴。做过公众号排版的人应该都熟悉，墨滴证明了juice内联CSS后粘贴到公众号可以零调整。html-anything的一键复制到公众号功能，底层就是走的这条路。

第三个是gcui-art/markdown-to-image，提供了iframe到高DPI PNG的截图导出路径。html-anything的一键导出PNG功能，用的就是这个方案。

第四个是alchaincyf/huashu-md-html，这个项目我之前没关注过，但它做的事情很有意思，是一套「反AI-slop」的设计纪律。具体来说就是中文优先字体栈、8px基线网格、对比度大于等于4.5、必须用真实数据。html-anything把这套纪律写进了每个SKILL.md的硬约束里。

除了这四个核心依赖，它还引用了op7418/guizang-ppt-skill的编辑墨水PPT灵感、1weiho/open-slide的1920x1080 agent-native画布思路、heygen-com/hyperframes的视频帧规范、remotion-dev/remotion的视频渲染能力。

你看看这个引用链，从设计系统到agent检测，从公众号排版到截图导出，从反AI话术到视频渲染，每一个都是对应领域里经过验证的方案。不是从零开始造轮子，而是把最好的轮子组装成了一辆车。

## 零API Key这个设计决策很聪明

从产品角度看，我觉得最聪明的决策是「零API Key」。

html-anything不要求你单独配置任何API密钥。它直接复用你已经在终端里登录好的CLI session。如果你之前在终端里跑过claude login或者cursor login，html-anything启动时就能检测到，直接用你的订阅额度干活。

边际成本为零。

这个设计看似简单，实际上解决了一个很现实的问题。现在各种AI工具都要你贴API Key，每个工具配一遍，管理起来很烦，而且还有安全风险。html-anything绕过了这个问题，把你已有的订阅变成基础设施。

调用方式也值得说一句。它通过spawn子进程调用本地CLI，用JSON-line协议做stdin/stdout通信，每个CLI一个轻量adapter。SSE流式传输，你能在浏览器里看着AI一行一行写HTML，不满意可以中途打断重发，不浪费一整次生成。

还有一个细节，iframe沙箱预览。生成的HTML跑在sandbox="allow-scripts allow-same-origin"的隔离环境里，Tailwind CDN和Google Fonts能正常加载，但cookie和localStorage跟宿主页面隔离，安全上做了该做的。

## 从MD到HTML，产品化的关键一步

回到我之前提的那个观点，MD是数据内容层，HTML是展示交互层。

这个判断到现在我依然觉得成立。Markdown的优势在于结构化、轻量、token效率高，它是给「写」和「处理」用的。HTML的优势在于排版自由、可交互、视觉表现力强，它是给「读」和「分享」用的。

过去的问题是，从MD到HTML这步转换太贵了。要么你手动写CSS调布局，要么你用模板引擎但灵活性有限。对于大多数非技术用户来说，这步根本走不通。

html-anything做的事情，就是用AI把这个转换成本压到了接近零。你写Markdown或者贴一段CSV，选个模板，按一下快捷键，AI帮你把内容灌进一套经过设计系统约束的HTML里，生成完就能发。

这让我想起一个类比。当年数码相机刚出来的时候，摄影师们也在争论，RAW格式才是专业选择，JPEG是给外行用的。但后来Lightroom这类工具出现了，一键调色一键出图，RAW到JPEG的转换成本被压到极低，JPEG才真正统治了大众摄影。

MD和HTML的关系有点像。不是HTML要取代MD，而是当MD到HTML的转换足够便宜的时候，HTML才会真正成为内容交付的默认格式。

## 75套模板背后的设计哲学

从产品经理的角度看，75套模板这个数字本身就是一个信号。

它说明这个团队不是在做一个小工具，而是在搭一个平台。模板覆盖了prototype、deck、frame、social、office、doc六种mode，每种mode下面又有多个scenario细分。比如office模式下面，PM规格书、工程runbook、财务报告、HR onboarding、发票、OKR、周报、会议纪要、看板，全都有对应的模板。

每套模板都遵循Claude Code的SKILL.md协议，带有扩展的frontmatter，包含mode、scenario、surface、preview、design\_system等元数据。也就是说，模板本身是机器可读的，AI agent可以根据用户输入自动匹配最合适的模板。

设计系统上，它用了paper/bone/ink/coral的调色方案，Inter Tight加Playfair Display的字体组合。从截图来看，视觉完成度确实不低，不是那种「能用但丑」的水平。

不过我也注意到，项目目前的技术栈是Next.js 16加React 19加Tailwind v4，相当激进。PROGRESS.md里记录的验证环境显示，Claude生成一份31KB的HTML大约需要80秒。这个速度对于长文档来说可能还需要优化。

## AI时代的速度让人感慨

说到这，我其实最想感叹的不是技术细节，而是这个项目诞生的速度。

5月8号，Thariq发了那篇讨论HTML vs Markdown的文章。5月11号，html-anything的第一个commit就出现了。到5月13号，三天时间，从v1迭代到v5，75套模板、8个agent适配、9类交付场景、一键发布到公众号/推特/知乎，全部跑通。

三天。

从一场行业讨论，到一个功能完整、设计精良的开源产品，只用了三天。

这个速度在传统软件开发模式下是不可想象的。但在这个case里，链条非常清晰，一个有洞察的想法，一个执行力强的团队，加上AI辅助编码，把一个从讨论到产品的周期压缩到了极致。

我之前说MD和HTML之间有产品化的机会，但说实话，我以为这个机会会被某个创业公司用半年时间做成一个SaaS产品。没想到一个开源团队用三天就给出了一个相当完整的答案。

这就是AI时代的特点。想法+行动力+AI，这个公式正在以前所未有的速度运转。你今天在推特上看到一个有趣的讨论，明天就能看到有人把讨论变成代码，后天就能看到一个可用的产品。

对于产品经理来说，这传递了一个信号，光有想法已经不够了，行动力的权重被无限放大。因为当你还在写PRD的时候，可能已经有人用AI把MVP做出来了。

## 我的判断

从产品形态来看，html-anything切了一个非常精准的切口。它没有试图做一个大而全的内容平台，而是聚焦在「agent时代的HTML编辑器」这一刀。本地优先、零API Key、复用已有订阅，这三个决策加在一起，让它几乎没有使用门槛。

从竞争格局来看，目前这个赛道还处于早期。mdnice墨滴解决了Markdown到公众号的排版问题，但它的核心还是Markdown输入。html-anything直接跳过了Markdown，让AI输出HTML，这是一个层级上的差异。而open-design作为同一个团队的大型项目，更多是面向设计系统和品牌网站的场景，html-anything则更聚焦在内容交付。

从趋势判断来看，我认为未来6到12个月，会出现更多类似的产品。因为底层逻辑是通的，AI的输出能力已经足够强，缺的就是把输出「包装」成读者想看的样子这一步。谁把这一步做得最便宜、最无缝，谁就抓住了这个窗口期。

我的判断很简单。html-anything这个项目值得你clone下来试试。不是因为它已经完美了，而是因为它指出了一个方向，MD到HTML之间的产品化机会，正在被一群执行力极强的开源开发者变成现实。

而这一切，从一场讨论到产品落地，只用了三天。