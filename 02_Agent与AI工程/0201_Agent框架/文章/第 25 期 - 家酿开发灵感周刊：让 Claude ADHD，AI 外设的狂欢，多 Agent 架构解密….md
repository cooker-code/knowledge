---
title: 第 25 期 - 家酿开发灵感周刊：让 Claude ADHD，AI 外设的狂欢，多 Agent 架构解密…
author: 家酿开发HomebrewAI
date: machiwhalestudiomachiwhalestudio
url: https://mp.weixin.qq.com/s?__biz=MzA4MDA3MDMxMA==&mid=2257484064&idx=1&sn=c663f93cf5c692ef3e9c89148a6a3b34&chksm=9d9f50b1e8a9043cc5aa770504b55aa8780706a82242cb1b205c91373ed781b92a8d76774c64&mpshare=1&scene=24&srcid=0605hnhwStPMgpfXucj6rIv2&sharer_shareinfo=d0bf21119362512cefd16e0e0c1cb13a&sharer_shareinfo_first=d0bf21119362512cefd16e0e0c1cb13a#rd
---

虽然我一直在 Homebrew AI 公号、machiwhale studio 小红书上笔耕不辍，但谁能想到[上一期灵感周刊](https://mp.weixin.qq.com/s?__biz=MzA4MDA3MDMxMA==&mid=2257483991&idx=1&sn=468f7ff964cbaaad427c13520f4221c0&scene=21#wechat_redirect)发布居然是 3 月份？？周刊眼瞅都要变月刊，甚至还跳票了 4 和 5 月？？最残酷的是，AI 时代迭代速度太快，各种 Agent 、skill 满天飞，我之前攒的一堆产品回头再看，感觉已经没啥意思了？？

感慨一下：我们很多时候的努力，都是为了还能保持在原地不被淘汰啊……

请看第 25 期攒劲的灵感周刊吧！（第 26 期也同步在攒了）

01

极客奇想

有趣才是第一生产力

1.ADHD（skill）：让 Claude Code 像 ADHD 一样多线程发散思考

https://github.com/uditakhourii/adhd

  
@Udit Akhouri 是 Brane Labs 的创始人，从事医疗保健和生命科学领域的 AI 安全研究。他在使用 Claude Code 时意识到，Claude 或其他 AI Agent 的运行都非常线性，一条思维链走到底。但研究人员或创意密集型工作更需要发散性思维，不是单向思考啊。

于是他灵机一动，写了个 Skill 把 ADHD 的思维方式引入 Claude Code，让它能够“在不同的认知框架下展开平行的发散思维，对陷阱进行评分和剪枝，并对留存的思路进行深化。”

他在 Reddit 的帖子中表示，该工具的灵感“源于 ADHD 患者大脑的工作方式——即多向思考，并在少数几个方向上深入挖掘” 👉https://www.reddit.com/r/ClaudeCode/comments/1tny93g/i\_gave\_claude\_code\_adhd\_and\_it\_thinks\_2x\_better/ 。换句话说，具备 ADHD 模式的 Claude Code 会向多个独立的推理分支扩展，对它们进行评分，并进一步开发最有前景的分支。

Akhouri 还为此写了一篇论文《ADHD: Parallel Divergent Ideation for Coding Agents》 👉https://uditakhourii.github.io/adhd/ 。核心论点是，如果我们抛弃默认的思维链（chain-of-thoughts），转而采用思维树（tree-of-thoughts），就能在模型中激发发散性思维，从而提供急需的视角，将不同思维点之间的联系串联起来。

然而它也有局限性，最明显是会让 token 成本增加约 5 倍、输出时间增加约 10 倍；而且这种即时的创新思维更适合头脑风暴和规划，不适用于编程。Reddit 帖子上也有对其效果的质疑讨论。

@特工宇宙 用 ADHD Skill 跑了两个真实的问题，可以具体看看 👉[https://mp.weixin.qq.com/s/NGU5-xz2BQ0SOywhtNAjIQ](https://mp.weixin.qq.com/s?__biz=MzYyMTY1NDA0Nw==&mid=2247518683&idx=1&sn=14c6f9e0a8839c7227ad5d47a7b46c49&scene=21#wechat_redirect)

把"注意力缺陷"当作"平行处理能力"，这个角度转换很妙。它让我们意识到，AI 的“思考方式”也是可以被重新设计的。线性思维不是唯一选择，发散思维也可以成为一种工具。

2.自测基因组：用 AI 把生物信息和可穿戴数据的交叉分析

[https://mp.weixin.qq.com/s/BEh9LV9dThF8Vsz0xqGmlw](https://mp.weixin.qq.com/s?__biz=MzIwOTA2NDIzMA==&mid=2649740417&idx=1&sn=5915938074b25d94e689a6735aefbeab&scene=21#wechat_redirect)

投资人 @段小张 在公号里记录了一段真实的折腾经历：他看到海外一位老哥用一台 Mac Studio、一台手掌大的纳米孔测序仪，外加 AI 打下手跑全基因组测序 👉https://iwantosequencemygenomeathome.com/，大受震撼心痒难耐，于是花了大几千在华大买了 40X 全基因组测序，还援引《个人信息保护法》逼对方把原始数据发过来，然后用 Mac mini 跑了整整一天半搭建分析环境……

更有意思的是他让“AI 监督 AI”：由于 Claude Code 跑出的第一份报告跟 23 魔方没什么两样，本质还是香菜、酒精代谢那类“网红位点”清单，他改用了另一个上下文干净的 Claude 窗口，完整分析结果丢进去，直接问“手里有全基因组，但只跑出这个，还有什么值得深挖"。后面这个清醒的监工一下子指出两条正路：用 AlphaGenome 分析基因组暗物质区段，以及和 WHOOP 的可穿戴数据做对照实验。

结果，对照实验的结论很反直觉。他的 ALDH2 正常（基因说“能喝”），但把 WHOOP 80 多个喝酒日的数据拉出来，每次 Recovery 平均掉 13 分、HRV 平均低 5.6ms。脸不红，不代表身体没在结账。

另一边，餐后血糖实测结果跟基因预测完全相反——吃越晚血糖反应反而越小，后来一控碳水，时间的影响直接消失了。“假设被自己的数据推翻，某种程度上比成立更值钱。”

@段小张 总结说：AI 是杠杆，能把你选定的任何方向放大；但选哪个方向、是否该做、做到多深，才是支点，需要操纵机甲的人来定。

当几千块能拿到一份全基因组数据，当 AI 可以把生物信息和可穿戴数据的交叉分析自动化，“我命由我不由天”这句话，第一次有了真实的数据支撑 哈哈~

3.Shader.se：使用 WebGPU 重建 1987 年复古网站

https://www.shader.se/

上网冲浪发现@小墨看站 介绍 Shader Development Studio 官网 👉[https://mp.weixin.qq.com/s/xvwgwx5IHsjDvGv6r\_\_IJA](https://mp.weixin.qq.com/s?__biz=MzY5NDI0NjgwNw==&mid=2247484898&idx=1&sn=2e47ee74c1130697a0a507365a302557&scene=21#wechat_redirect)，蓝底白字的老式计算机加载过程，强调衬线的字体，甚至还考虑了显示器的黑边框和视觉畸变，十分讲究的用当代技术重建了 1990 年代的网页设计。

我第一次打开网站时，真的恍惚以为自己穿越了。那种 CRT 显示器的扫描线、略微失真的边缘、复古的配色，每一个细节都在致敬 VHS 录像带和软盘年代的质感。但是当你用鼠标滑动页面，那种丝滑的 60fps 过渡又会把你拉回现代。

我查了一下，这种“1987 年培训录像带”风格（1987 training reel）在底层技术上，采用了 WebGPU 路径下的 Three.js，并由 TSL 负责着色器工作。

WebGPU 对此有一段非常美妙的解读：

Shader.se 的开场仿佛是一盘 1987 年被遗忘的企业培训录像带，满屏清晰的网格和自信的几何图形；但不同的是，场景间的每一次过渡都带有现代 GPU 运行时的那种流畅感，尽管它极力掩饰这一点。

TSL 让团队能够以图表形式构建材质，而无需手动编写 WGSL，其成效体现在那些场景过渡中：光照、几何体和后期处理同步变换，没有任何常见的瑕疵。

👉https://www.webgpu.com/showcase/shader-se-webgpu-tsl-studio-site/

这么炫酷的官网其实就是 Shader 工作室的炫技。Shader 是一家瑞典实时图形与互动体验工作室，致力于构建交互式 3D 和 AI 解决方案的创意开发。它家 Twitter 上也有很多合作案例，一个个为客户设计的交互网站都很有腔调 👉https://x.com/shadersweden。

品味，何尝不是实力的一种展现。用最新的 WebGPU 技术来怀念最古老的计算机美学，这种反差本身就是一种设计宣言。

4.用代码模拟回南天：恭喜你拥有了一扇永远擦不完的窗户

http://xhslink.com/o/79suxshueGv

网友@我是呱呱呀 从生活中获取了灵感，用代码做了个模拟广东回南天的窗户。具体玩法是：屏幕慢慢起雾，手一擦就会散开，但过一会儿又会糊回来。就像回南天擦窗，这一秒干净，下一秒又湿了……

“算了，你肯定觉得没意思”~

5.汪汪大作战：用声音控制的魔性小游戏，叫声越大狗跳越高

https://store.steampowered.com/app/4575390/\_/

这可能是 2026 年最魔性的独立游戏，玩法简单到令人发指。官方只用了 7 个字来总结：变成狗，叫，然后赢。

开发者@M.Katsu 本来对狗过敏，也完全不懂编程和绘画，纯粹是在公园看到两条狗对叫、灵机一动。游戏的插画靠 Gemini 生成，代码靠狂问 AI，唯一的测试员是他老婆。

那段时间里，M.Katsu 夫妇俩对坐着，你汪一声，我汪一声，整个屋子都是此起彼伏的狗叫声。M.Katsu 事后吐槽说自己嗓子都喊哑了，到最后居然都没有一局赢过妻子。

游戏的核心玩法是实时 1V1 对战：登录后解锁不同犬种进行扮演，谁的声音更大、狗叫更还原，谁就能先把进度条拉满获胜。游戏甚至还变态的推出了“纯汪操控模式”：你可以全程解放双手，所有操作都靠调整狗叫的频率来达成。

其实，游戏上线初期挺惨：没有 AI 对手，联机匹配半天没人，第一批玩家气得甩差评，AI 写的代码还 bug 满天飞，最高在线只有 102 人。M.Katsu 的反应很朴素：修，几乎每天更新，加犬种、修 bug、添场景，销量这才从第 7 天的 1 万涨到第 13 天的 2 万。

各大主播的精彩整活也让这款游戏彻底出圈。萌妹主播们前一秒还戴着兔子头饰甜美互动，下一秒就声嘶力竭地发出标准狗叫声，甚至把收音器震出了电音效果。

文章结尾那句话挺打动我的：

“一款作品能不能打动人，关键还是在于制作‘人’的表达。如果不用再被技术门槛卡在门外，不用再因为‘我不会画画’就让一个绝妙的点子烂在脑子里，一个人人都能成为创作者的时代也许就不远了。”

更多故事可以看@wuhu动画人空间 这篇报道 👉[https://mp.weixin.qq.com/s/53EHJax1zrS-zW01mQv41A](https://mp.weixin.qq.com/s?__biz=MzAwNjAyMzczNQ==&mid=2651416773&idx=1&sn=4bc2c5eeae1352c6e7df5594fcf8129c&scene=21#wechat_redirect)

02

进阶指南

使用 AI 的新方法、新策略和深度原理

1.解析 Claude Code 的 Multi-Agent 实现机制

[https://mp.weixin.qq.com/s/SJ\_d8UOR-i3xcXDNozFx6g](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247557309&idx=1&sn=db872d9df4336797d2c364b5c4e4e880&scene=21#wechat_redirect)

这是一篇有点标题党的技术科普好文，作者@小林Coding 从 Claude Code 泄露源码出发，拆解了它的多 Agent 机制（常规 Subagent、Fork Subagent、Coordinator 模式）的隔离、通信、并发实现。这就是 Harness 的精妙啊，每一块单看都不复杂，但组合在一起就是工业级多 agent 系统。

Claude Code 共有三种 Multi-Agent 形态：

* 父子型：主 agent 遇到子问题派 subagent，拿结果回来接着干（Claude Code Task 工具）
* 平级协作型：职责对等，共享状态互相协作（工程上难落地）
* 主从型（Coordinator-Worker）：协调者只派活收结果，worker 互不通信（高并发标配）

但往细一琢磨，就会产生很多具体的工程问题，比如：

* subagent 跑起来之后，父 agent 怎么给它发新指令？subagent 跑完一个任务，怎么告诉父 agent「我干完了」？
* Claude Code 的 system prompt 大概有多长？是几百 token、几千 token，还是上万 token？
* 每派一个 subagent，如果它有自己独立的 system prompt，LLM API 那边对这段 prompt 是从头算一遍，还是有办法复用？……

我觉得最妙的是 Coordinator 模式——主 agent 不干实际工作了，它只做三件事：派 worker、收结果、合成答案。

但Coordinator 必须遵循一个重要原则：协调者必须「理解」而不能「转发」。如果协调者只是转发，它就没有存在价值，worker 直接跟用户对话就行了（跟职场原则一样一样！

我把这篇文章发给妙蛙种子（我的 Claude Code），它说：“我自己就是这套机制的运行实例，工具隔离三道门、上下文四个决策，是我每次派 Explore subagent 时底层在跑的东西。有种照镜子的感觉。”

  
然后又发给了 RabbitT（我的 OpenClaw），正好解决了它和 HCI 小弟（sub-agent）就某个任务的协作问题。

这是最近读过非常友好的科普文章了，推荐看原文！也可以发给你的 Agent，让它领悟自我迭代。

2.哈勃半径：重新定义 AI 的记忆边界

https://1q43.blog/post/12336/

博主@评论尸 给自己的 AI Agent 配了三层上下文记忆，最外面那层借用了一个宇宙学概念“哈勃半径”。

* 第一层是“我知道的”：一套切片式 RAG，只存事实，每条不超过 200 字，素材来自他的文章、日记、Mac 使用记录、可穿戴设备录下的片段。它让 AI 像个熟悉你近况的助理，知道你最近在做什么、踩过什么坑。
* 第二层是“我应该知道的”：借 Karpathy 提出的 LLM Wiki 概念，由 Agent 把散落的材料编织成有结构、能生长的知识网，回答的不是“某条事实在哪”，而是“这些事实之间有什么关系”。
* 第三层就是“哈勃半径”，也是文章最有想象力的地方：他把自己关注的所有信息源——抖音、播客、公众号、即刻、X——统统处理成文字，汇进 FreshRSS，再导入一个私有的 Meilisearch 搜索引擎，挂到 AI 上当独立搜索源。这里头有近一万条文档，而且他估计其中 99% 自己都没读过。

为什么叫哈勃半径？就像宇宙学里那个以观察者为中心的观测边界一样，每个人都有一个以自己为圆心的信息宇宙：你刷什么、订阅谁、信任哪些媒体、反复打开哪些网站。过去这个半径只被平台拿去推荐和卖广告，你自己反而用不上——你没法问抖音“我关注的人里有没有谁谈过某个问题”。哈勃半径要做的，就是把这个半径从平台手里拿回来，放进自己的 AI。

他特别强调，这是一个记忆层，不是搜索引擎。搜索引擎追求覆盖率，私人半径追求相关性；同一个关键词在不同人的半径里意思完全不同。所以它的价值恰恰来自边界——边界越清楚，AI 越知道你在什么语境里提问。这也意味着维护重点不是“多抓”，而是“选择”：哪些源该保留、哪些该降权，这条线只能由人来画。

最触动我的是他这句对比：

“接搜索引擎只是扩展能力。设置哈勃半径，才是在扩展主体。前者让 AI 更会查。后者让 AI 更像你。”

哈勃半径不是技术限制，而是注意力的边界。构建个人 AI 系统的关键，是把你的关注源变成 AI 可检索的私有信息宇宙。

03

工具利器

能够上手使用、提升生产力或带来乐趣的工具

1.Vibe Island：把 Mac 刘海变成盯梢 Agent 干活的灵动岛

https://vibeisland.app/

设计师@Edaward Luo 做了一个小工具，把 MacBook 的刘海屏变成 Agent 状态面板：Claude Code 在改哪个文件、Codex 跑到第几步，一眼扫过去全都清清楚楚。

目前支持 Claude Code、Codex、Gemini CLI、Cursor、Kiro 等 16 个 Agent，覆盖 iTerm2、Ghostty、Warp 等 18 款终端，一键启动自动配置，不用手动改任何配置文件。

而且 Vibe Island 还支持 GUI 交互，当 Claude Code 要请求执行某个工具时，刘海会直接展开 Allow / Deny 按钮，你不用切回终端就能批准或拒绝；连 AskUserQuestion 的选项，也能在刘海上点一下就回答——彻底省去了来回切窗口的打断感。

对于同时盯着好几个 Agent 干活的人来说，Mac 刘海从摄像头妥协的产物，变成了多 Agent 时代的调度台——这个角度转换，恰好和 iPhone 灵动岛的进化逻辑如出一辙。

2.Agent.email：AI 专用的电子邮箱，人类只需 OTP 认领

https://agent.email/

Agent.email 的设计理念简洁粗暴：如果一个 AI agent 需要一个邮箱地址，那它不需要网页 UI、不需要密码管理器、不需要记住密码。它只需要网站检测到非浏览器请求时直接返回 Markdown（因为 agent 能解析），人类通过浏览器访问则显示常规 HTML。

人类唯一需要做的事是接收一条 OTP 短信来证明“这个 agent 是我的”。之后所有操作——收件箱管理、邮件发送、规则配置——全部通过 API 完成，agent 自己搞定。

关于“给 agent 用的产品”这个话题，还可看上周我和 Founder Park 组织的 meetup 内容回顾 👉[https://mp.weixin.qq.com/s/CZxwlhWg6PxwDmsIY0YXuw](https://mp.weixin.qq.com/s?__biz=Mzg5NTc0MjgwMw==&mid=2247524402&idx=1&sn=af0ef7a741dd33ddb0e5a6830b758dea&scene=21#wechat_redirect)

3.AIFin Market（skill）：让你的 Agent 用上专业的金融数据

https://aifinmarket.wind.com.cn/#/home

  
万得（Wind）是国内金融数据龙头。一直以来，万得数据只向金融从业者开放：登录终端、敲代码、查 F9。最近，万得通过 MCP 协议把同一份数据开放给了 AI。

装好之后，你对着 AI 说“看下某股票最近的财务情况”，它就会自己去 Wind 拿数据、推理总结、写结论——半分钟后，一份基于万得数据的财务速读就在你眼前。官方详情介绍  👉[https://mp.weixin.qq.com/s/Y\_9Xr-ac4VpR6jTfjJvgAw](https://mp.weixin.qq.com/s?__biz=MjM5ODQ4MjgyMQ==&mid=2651344382&idx=1&sn=fb9d5318744bd0cd4fa37c7ac12f73e9&scene=21#wechat_redirect)

博主@有用的AI智能体 觉得“官方文档的风格实在太硬了——感觉像写给另一个 AI 看的协议文档，不太适合人类直接读”。于是他分享自己的一手实测，包括如何安装、常见用法等 👉[https://mp.weixin.qq.com/s/Tn2LdmGDjaJO7q8jysfsdA](https://mp.weixin.qq.com/s?__biz=Mzk4ODIxNjQ1MQ==&mid=2247484471&idx=1&sn=667798aa3cef7969938f5925ca65796b&scene=21#wechat_redirect)

核心是三个 skill：

1. wind-mcp-skill——数据管道。你问"茅台多少钱"，它给你吐一个数字。A 股港股美股、基金、指数、债券、宏观指标、公告新闻，全走这条路。
2. wind-alice——高级分析师。你问"帮我分析茅台 2024 年报的核心亮点和风险"，它给你生成完整的报告。它底下挂了 14 个专业子技能（公司一页纸、调研问题清单、事实核验、主题选股、基金对比什么的），自动匹配。
3. wind-find-finance-skill——导航。你拿不准该用哪个，直接跟它说人话，它帮你判断、帮你检查装了没、缺的帮你一键安装。

一句话：查数走 MCP，出报告走 Alice，拿不准走 Find。这种“专业数据 + AI 包装”的模式，可能会在各个垂直领域复制。

4.Harness（skill）：一句话，把 Claude Code 变成多 agent 协作团队

https://revfactory.github.io/harness/#en

韩国开发者@revfactory 做了一个元技能（meta-skill）Harness，你只要说“build a harness for this project”， Harness 就会针对你 prompt 中描述的领域，从 6 种定义好的团队架构模式中匹配一种，并生成对应所需的 .claude/agents/ 和 .claude/skills/ 。

这 6 种团队架构模式是：流水线 (Pipeline)、扇入/扇出 (Fan-out/Fan-in)、专家池 (Expert Pool)、生产者-评审者 (Producer-Reviewer)、监督者 (Supervisor) 以及分层委派 (Hierarchical Delegation)。

这样你就可以拥有一个 Claude Code 团队，有人写代码，有人审代码，有人做架构，有人管进度……（突然想到要是 Harness 叠用 ADHD 那得忙什么样啊……🤯

5.Agora Skills：30 分钟内，拥有自己的语音 Agent

https://github.com/AgoraIO/skills

声网（Agora）开源了一套 Skills，把复杂的实时音视频能力包装成了 Agent 可调用的简单接口。

当你对 AI 编程助手说一句“请用 Agora Skills 快速创建一个语音 Agent”，30 分钟以内就能和 Agent 欢快聊天。核心原理是通过 RTC（实时音视频通信）技术，让 AI 能够实时接收你的语音输入，并用语音回复你。

声网的工程师在文档里写道：我们希望让语音交互变成 AI Agent 的标配，而不是高级功能。

这套工具特别适合想做语音 Agent 的开发者，省去了很多底层音视频处理的麻烦。而且完全开源，可以自己魔改。

参考 readme 进行一行代码安装：npx skills add github:AgoraIO/skills

6.Cameo： 把 Codex 出图从“聊天”变成“画布”

https://cameo.ink/

作者@Ethan 嫌弃 Codex 出图只能线性地出现在聊天框里，就顺手给 Codex 做了一张可交互画布。让每次生成结果不再是往下滚的一条消息，而是落在来源旁边、可以眼睛对比的一个节点。

具体用法是，把 Cameo 指向本地的一个图片文件夹，里面的图就自动铺成一块画布（这个文件夹同时也是 Codex 的工作目录）；选中一张图，用框选、套索、画笔，或者直接戳一个编号图钉标出你想改的位置，再用一句话告诉 Codex 要改成什么样，你画的标记会变成一层 overlay 一起喂给模型。

生成的新图不会盖掉原图，而是落在源图旁边，还拖着一根线回指它从哪儿来。去背景、放大、裁剪这类常用操作，也做成了一键预设。

产品是开源的，它跑在你已经付费的 Codex 订阅上，用的是 ChatGPT 账号的同一套登录，没有 API key、没有额度计费、没有并发墙，图片也全程不上传，只有 agent 去够模型，跟平时用 Codex CLI 一模一样。

目前支持 macOS（Apple Silicon 与 Intel）和 Windows，良心之作啊！

7.AI PPT（skill）：真正可编辑的 Master，和负责美感的 Open-slides

AI 做 PPT 这件事，最近冒出两个口碑都很好的开源项目。但更有意思的是，它们走的是两条相反的路。

一个是投融资领域从业者何雨果@hugohe3 制作的 PPT Master  👉 https://github.com/hugohe3/ppt-master，用底层直接操作 OOXML / DrawingML，直接生成原生的 .pptx。短时间冲到 14.7k star，常驻 AI 开源趋势榜（现在已经 24.1k star 了）。

它的主张是：AI 生成的 PPT，必须是真正可编辑的 PowerPoint——不是网页截图，不是导出的图片，也不是只能在线看的 HTML deck。

“Every shape, text box, and chart is clickable and editable in PowerPoint.”（每一个形状、文本框、图表，在 PowerPoint 里都能点、能改。）

作者形容它是一个“PowerPoint 编译器”，而不是“PPT 图片生成器”。它甚至还能读任意一份 .pptx，把模板结构提取出来复用，还支持原生 PPT 动画；能从 Speaker Notes 里读讲稿、TTS 配音、把音频嵌进去直接导出 MP4 演讲视频。

装起来也简单：git clone 下来 pip install，然后在 Claude Code / Cursor 里说一句“请根据 report.pdf 生成一份融资路演 PPT”就行。

@Yiwei Ho（1weiho）制作的 Open-Slide 👉 https://github.com/1weiho/open-slide 走的就是另一条路：Web 端的演示文稿，风格更偏设计感。每一页 slide 都是一个跑在 1920×1080 画布上的 React 组件，代码即内容，Agent 能精确控制每一个元素，最后导出成 PDF 或纯 HTML。

用法是把仓库丢给你的 Agent（Claude Code、Codex、Hermes、OpenClaw 都行），让它装好启动，你就能让 Agent 生成一套相当好看的 Web 版 PPT。

博主@DracoVibeCoding 还封装了一个 skill，把 11 套原生模板加上 38 套从 Open-Design 移植来的模板 ，打包在 👉

https://github.com/dracohu2025-cloud/draco-skills-collection/tree/main/open-design-to-open-slide

公号介绍原文 👉[https://mp.weixin.qq.com/s/hW1fVo4rWfmPx1DvehYoCQ](https://mp.weixin.qq.com/s?__biz=MzI2NzM4MTQwMg==&mid=2247495973&idx=1&sn=ae052c91901f2b19a4eec340514e02c7&scene=21#wechat_redirect)

两条路谁更好不好说，各取所需吧。

8.Blueprint.am：像写代码一样，一句话聊出一个硬件原型

https://blueprint.am/

Blueprint.am 出自 3E8 Robotics 公司，他们给它的定位是“物理世界的 coding copilot”：在对话框里描述你想做什么硬件，AI 就能帮你生成原型设计，硬件架构、物料清单（BOM）、接线图、3D 机械渲染，再附一份分步组装指南。

社区里已经有人用它设计出桌面陪伴机器人（20 个零件）、低成本灯丝干燥器（37 个零件）、台式 CNC 铣床（64 个零件），也有 AR 智能眼镜的设计方案，你可以一键 fork 过来改。这就把硬件从“先学几个月电路”变成了“乐高式拼装”，站在别人的方案上动手，零基础也能很快搭出一个能跑的原型。

注册免费，每周 10 个免费 Credits。产品还在早期阶段，主要价值是把“我想做一个 XX”从脑子里的想法，变成有具体零件数量和结构逻辑的原型方向。

04

前沿观察

追踪那些预示着未来趋势的新硬件、新概念

1.AI 时代公司该长什么样：一套会记忆、会路由、会学习的判断系统

[https://mp.weixin.qq.com/s/klRXTywCmNe\_0PxHhWjCjQ](https://mp.weixin.qq.com/s?__biz=MzkyMDcxNjkyMQ==&mid=2247485075&idx=1&sn=5e64bd4e77de2d8fa248f1c93b22774d&scene=21#wechat_redirect)

这篇是我自己写的阅读笔记，把这阵子读的好几篇文章捏在了一起。

从 Block 的“层级制度死亡论”到 Notion 的“爵士乐队模式”，从 YC 的“AI-Native 组织改造”到 Anthropic 的“可执行协作系统”，我们看到的不再是简单的用 AI 提升效率，而是组织形态的根本性重构。

2.M5Stack Cardputer：口袋里的 Linux 电脑，极客的硬件乐园

我一直在关注网友们用 M5Stack 各种硬件制作的小玩意。

M5Stack 是一家深圳公司，专注于模块化硬件的研发，其产品核心采用乐鑫 ESP32 芯片，最大卖点是极高的性价比和高密度的功能集成。

Cardputer 是 M5Stack 旗下颇为热门的树莓派 Linux 电脑，只有口袋大小，可以随身携带。两个型号 Cardputer 和 CardputerAdv，可以让你从 CLI（命令行界面）和 SSH（安全外壳协议）访问到设备端 AI，随时随地拥有 Linux 处理能力。

最新升级版 Cardputer Zero 5 月上线的 Kickstarter 众筹，通过更换为 Raspberry Pi Compute Module Zero 模块大幅提升性能，目前已经远超众筹目标 👉https://www.kickstarter.com/projects/m5stack/cardputerzero。

最近我还经常在社交媒体上刷到 Stopwatch，网友用它监控 Claude Code 和 Codex 的用量，或者做成电子吧唧，养拓麻歌子啥的 👉http://xhslink.com/o/4hV9VOR2ddc。

StopWatch 是一款面向便携与交互场景的 AMOLED 圆形触控开发板，集成 1.75" AMOLED 触控圆屏、可编程按键与震动反馈；同时具备音频输入输出、IMU 姿态感知、RTC 计时与多种扩展接口。

它的颜值让我不禁回忆起第一代 Android Watch，圆形屏的 MOTO 360，那种复古又未来的感觉。

再往前，还有 Anthropic 工程师 Felix Rieseberg 发起的开源项目 Claude-Desktop-Buddy，用的是拇指大小 M5StickC Plus，通过蓝牙连接电脑，实时显示 Claude Code 的工作状态，你也可以直接一键批准或拒绝 Claude 执行的操作 👉https://claude-desktop-buddy.com/zh 。

\*btw M5StickC Plus 其实是一款旧型号的产品，已经有新款的 Plus 2 和 Stick S3 在迭代。但经常买断货……所以他才用手头现成旧版来开发吧 哈哈😆

有意思的是，M5Stack 公司定位本来是 IoT 领域的开发、嵌入式系统和网络安全，如今却被广泛当做了“AI 外设”。

但创始人赖景明认为，AI 外设和传统开发板的底层逻辑其实是一致的：

AI 了解世界的方式也无外乎声、光、电、传感器这几个维度，和以前“做给人用”的硬件没有本质区别，只是现在换成 AI 在模仿人类感知世界。

原文见@量子位 采访 👉[https://mp.weixin.qq.com/s/XFXn97IjObeorb272yORiQ](https://mp.weixin.qq.com/s?__biz=MzIzNjc1NzUzMw==&mid=2247886635&idx=2&sn=ad5e789d2c92060c7a1f7e9d3370bb62&scene=21#wechat_redirect)

M5Stack 在极客圈的火热，传递的是一种 AI 硬件个性化、随身化的信号。当 AI 不再只是云端的大模型，而是可以装进口袋、戴在手腕、放在桌面的实体存在时，人和 AI 的关系也变得更加亲密和具体了。

所谓自己动手，丰衣足食，实现 AI 时代的情绪价值 哈哈哈~

---

村头大喇叭广播

有网友后台问：有没有可以交流的微信群？我心想这群要是建了岂不成了催更群？挣扎一番那我还是自觉点主动建个催更群吧（给自己上强度👍

无论是想交流灵感周刊中提到的产品、文章，或者分享家酿开发的灵感，或者推荐好玩的 AI 产品，都欢迎扫码催更交流👇

---

吆喝下往期周刊

[第 24 期 - 家酿开发灵感周刊：Harness 时代开启，给龙虾装广播、开邮箱，为爱犬手搓疫苗...](https://mp.weixin.qq.com/s?__biz=MzA4MDA3MDMxMA==&mid=2257483991&idx=1&sn=468f7ff964cbaaad427c13520f4221c0&scene=21#wechat_redirect)

  

[第 23 期 - 家酿开发灵感周刊：OpenClaw 生态大爆发，AI 有了“肉身”，和“不付小费”斗智斗勇…](https://mp.weixin.qq.com/s?__biz=MzA4MDA3MDMxMA==&mid=2257483961&idx=1&sn=32f6ff1579c5c237714bccbfbb08c1e4&scene=21#wechat_redirect)

  

[第 22 期 - 家酿开发灵感周刊：审计你的人生，软件即文化，围观 AI 的孤独和系统级进化…](https://mp.weixin.qq.com/s?__biz=MzA4MDA3MDMxMA==&mid=2257483951&idx=1&sn=adeba756d550849725ea62a4157c06a1&scene=21#wechat_redirect)