---
title: 一文看懂Skills：读完50篇文章后，我安装了这些13个skills（附清单）
author: 闻思修AI手记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU4NDUyNTUwMQ==&mid=2247485344&idx=1&sn=14be057f819bcb3a41553127dd9c5852&chksm=fc89680d4d6b04bbf5aa09888a2deb8cb22d1b47eb27d9ea378e3084379c6f1e968206bb121e&mpshare=1&scene=24&srcid=0307Rru4juVYysRNhxf0lfaw&sharer_shareinfo=c8aa46e07350940a3143806b41c8e231&sharer_shareinfo_first=c8aa46e07350940a3143806b41c8e231#rd
---

Claude Skills这东西，火了好几个月了。

全网都在喊“Skills改变生产力”“Skills让效率起飞”，但我一直没动。主要是感觉时候没到。

我的习惯是：一项技术刚出来的时候不追，等它成熟到能比现有工具强十倍，再一次性研究透。

这可能会错过热点流量，不过没关系。追新工具这件事，太容易把自己累死了。

最近觉得时候到了。我把过去几个月收藏的50多篇文章翻出来，筛出20+篇精华，用Gemini、Claude、GPT、Manus分别跑了一轮Deep Research，综合成了一份完整报告。

这篇文章，就是研究之后的全部收获。

## Skills 到底是什么

一句话：**把你的流程性知识，变成可复用的能力包，让Agent随叫随到，稳定发挥。**

Skills本质就是一个文件夹。

文件夹里装的东西很清楚，如果你是职场人士：

* `SKILL.md` 相当于告诉你这个岗位是做什么的
* `scripts/` 是常用到的工具软件
* `references/` 是业务流程涉及的知识库
* `assets/` 是工作模板库

把这些东西打包放好后，让Claude干活的时候自己去查、去用、去执行。

你不用每次开新对话都从头交代一遍“我要什么格式”“我的风格”“按什么流程处理"等等。只需要调用这个skills就行。

这里有一个很容易被忽略的好处：**省Token**。

以往，在大模型的交互中，对话越长，模型越笨。且越耗费Token。

Token确实又寸土寸金（尤其像claude这种）。

而Skills呢把大量背景知识从对话中抽离出来。需要时按需调取，主对话窗口始终保持干净。这就间接省了不少Token。

## 什么时候该用 Skills

有一条判断标准特别好用：**任何你不想一遍遍重新解释的东西，都值得写成Skill。**

官方给了三类典型场景：

**第一类，组织级工作流。**品牌规范、法务合规流程、标准化文档模板。这些东西公司里每个人都要遵守，但每次找AI帮忙都得重新说一遍。此时写成Skill就能一劳永逸。

**第二类，专业领域经验。比如**Excel公式套路、数据分析流程、PDF处理方法、代码 标准、安全审计清单等。类似这些是你在某个领域摸爬滚打攒下来的经验。

**第三类，个人偏好和习惯。**你的笔记结构、代码风格、研究方法论。每个人干活都有自己的一套讲究，Skill能让AI记住你的偏好和风格。

根据这几个标准。我盘点了自己的习惯，并安装上了claude官方出的几个与自身工作贴近的skills。同时根据自己需求，又自制了几个skills。

## 怎么安装和创建 Skills

1、安装很简单，两种方式。

第一种，命令行直接装。直接告诉Claude："帮我安装这个skill，skill项目地址为 xxx"，它自己就能搞定。

第二种，把Skills文件夹拖到本地目录 `~/.claude/skills`，手动放进去。

2、创建Skills也不复杂。如果你不是程序员（比如我），最省事的办法是直接跟Claude说：

"我要创建 skill，一步步引导我"，它就会一步步带你走完流程，最后生成一个zip 包，上传安装即可。

进阶一点的做法是先安装官方出的`skill-creator`，当你提出需求的时候，让它调用这个skill帮你做设计。这样出来的Skill结构既规范又稳定。

## 把 GitHub 项目打包成 Skill：一个被低估的用法

这个思路来自卡兹克，我觉得是整个Skills生态里最值得关注的玩法。

**Skills最正确的用法，是将整个GitHub项目压缩成你自己的超级技能库。**

比如yt-dlp这个视频下载工具，打包成Skill之后，你只需要丢一个视频链接过去，Claude自动帮你下载。

比如Pake这个项目，可以把Web应用打包成轻量级桌面 APP。这些原本需要你去读文档、装环境、跑命令的事情，变成了一句话的事。

操作步骤并不复杂：

1. 复制目标 GitHub 项目链接
2. 给Claude Code提需求：“帮我把这个开源工具 https://... 打包成一个 Skill，实现xxx功能”
3. 先让claude开启计划模式做规划，再写Skill，稳定性会好很多
4. 如果一个skill在使用过程中遇到问题。你解决掉这个问题之后，记得告诉 claude：“把这些经验更新到这个skill里”。

第四步是关键。每次踩坑、每次修复，都喂回给Skill，它就会越来越好用。这就成为一个持续进化的skills了。

如果对这个部分感兴趣，可以搜索卡兹克这篇文章：《Skills的最正确用法，是将整个Github压缩成你自己的超级技能库》，应该会受到一些启发。

## Skills 不只是文档，还能跑代码

很多人以为Skills只是一堆提示词文件，其实不是。

Skills里还可以包含可执行脚本，比如Python脚本。

AI生成代码有个老问题：不稳定。

每次解决问题，它调用的库可能不一样。比如今天它用 `requests` 库，明天可能换成 `axios。`

又比如，同一个任务每次生成的代码也都不一样，调试成本很高。

但Skill里的脚本是你写好的、验证过的，逻辑固定，结果可预期。

同时，大量参考资料可以放在 `references/` 目录里，Claude 需要的时候自己去查，不占主对话的上下文。

这就像给员工配了一个资料柜，不用把所有文件都摊在桌面上。

## 五步框架：把工作流变成可进化的 Skill

这个方法论来自宝玉AI，观点比较激进但很有说服力：

**几乎所有能用workflow完成的AI任务，都可以用Agent + Skills实现。**

**我很认可这个观点。连lenny's newsletter里面赠送的n8n**（一种搭建流程的AI工具）**我都转手出去了。因为真的不需要了（[Lenny's Newsletter白送18个AI工具，我只激活了4个](https://mp.weixin.qq.com/s?__biz=MzU4NDUyNTUwMQ==&mid=2247485208&idx=1&sn=23a5b35f34e766127739a391cbfe99ff&scene=21#wechat_redirect)）。**

### 第一步：拆分

把工作流拆成单一职责的skill或subagent。每个模块只做一件事，做好一件事。

### 第二步：编排

在主skill里用自然语言描述整个流程。不需要写代码，像给同事交代任务一样说清楚就行。一个skill可以调用另一个skill，组合出复杂工作流。

### 第三步：存储

所有中间结果都保存成本地文件，而不是留在内存或上下文里。

### 第四步：分摊

Subagent之间只传文件路径，不传内容。这条规则很重要。直接把一大段内容塞给 subagent，上下文窗口很快就撑满了。

传路径的话，subagent会自己去读文件，上下文保持得很干净。

### 第五步：迭代

这是Agent + Skills相比传统workflow最大的优势——可以持续进化。

当skill的提示词你觉得不好的时候，可以让Claude帮你改。它甚至可以自己迭代 subagent的system prompt，实现自我进化。

这五步看似简单，但贯穿了一个核心思想：**拆得开、存得住、传得轻、改得动。**

## 实战案例

网络上实战案例太多了。看了一篇来自AI blew my mind的文章。整理了来自23个创作者的36个Skills应用场景。有兴趣可以按照下面标题搜索找来看。

## 一条最重要的认知

研究了这么多，最想说的其实是这一句：

**Skills一定是你本身实践过或者沉淀好的工作流，只是把它自动化。不是让AI帮你从零发明流程，而是把你已经验证过的流程固化下来。**

好的组织，会把经验变成可复用的 skill。好的个人，也一样。

这也是为什么同样用 Skills，有人效率翻倍，有人折腾半天没效果。差距不在工具，在于你自己有没有值得固化的东西。

## 最后我安装了哪些skills呢？

最终探索后安装了以下13个skills。供参考。这仅仅是第一次尝试装skills。后续应该会沉淀更多。欢迎交流。

1. `podcast-reader`：英文播客文字稿 → 结构化中文大纲（自制）
2. `github-to-skills`：自动将 GitHub 仓库转换为 AI Skills
3. `skill-manager`：Skills 生命周期管理
4. `obsidian-markdown`：Obsidian 风格 Markdown 创建与编辑（来自obsidian官网）
5. `pdf`：PDF 文件处理（读取、合并、分割、旋转）
6. `skill-evolution-manager`：根据反馈优化迭代现有 Skills
7. `skill-creator`：创建 Skills 的官方指南
8. `pptx`：PowerPoint 文件处理
9. `obsidian-bases`：Obsidian Bases 文件创建与编辑
10. `video-transcribe`：录制并转写视频音频为文字（自制）
11. `frontend-design`：生产级前端界面创建
12. `mcp-builder`：创建高质量 MCP（Model Context Protocol）服务器的指南
13. `json-canvas`：JSON Canvas 文件创建与编辑（来自obsidian官网）  
    以上5、7、8、11、12均来自anthropic官网；2、3、6来自卡兹克文章，文末资源清单附有有仓库链接

    图：我调用询问claude code现在安装了哪些skills。有些并不是真实的skills。总共是13个。

## 资源清单

最后整理一份实用资源，方便直接取用。  
**Skills 资源站：**

* **skills.sh**：精选 Skills 网站，支持 Claude Code、Cursor、GitHub Copilot、Windsurf、Codex、Gemini 等 16 种以上 AI 编程工具，一行命令即插即用
* **Anthropic 官方 GitHub**：github.com/anthropics/skills，学 Skills 从官方样例开始最靠谱
* **Skills Marketplace（SkillsMP）**：Claude、Codex、ChatGPT 的 Skills 市场

**Skills 管理三件套（卡兹克出品）：**

* `skill-creator` + `skill-evolution-manager` + `skill-manager`
* 解决 Skills 库的增删改查和迭代升级，实现全自动化管理
* 下载地址：github.com/KKKKhazix/Khazix-Skills

**官方延伸阅读：**

* *The Complete Guide to Building Skills for Claude*（Anthropic 官方权威指南）
* *Agent Skills - Claude API Docs*（开发者技术文档）
* agentskills.io（Agent Skills 开放标准官方网站）
* *36 Claude Skills Examples to Transform How You Work*（大量真实案例）

---

**推荐阅读**

1、 [告别Cursor低效编程：5个技巧让你事半功倍（附小白配置方法）](https://mp.weixin.qq.com/s?__biz=MzU4NDUyNTUwMQ==&mid=2247484476&idx=1&sn=d8d2fb797482bfef4019e3e54765c6ab&scene=21#wechat_redirect)

2、[谁懂啊为了让娃收心，竟然自己开发了个软件](https://mp.weixin.qq.com/s?__biz=MzU4NDUyNTUwMQ==&mid=2247485226&idx=1&sn=c638925f9546c6fbb6e1898cbb33ffea&scene=21#wechat_redirect)

3、 [腾讯ima 2.0发布：你的“第二大脑”来了？3个实战场景重塑工作流](https://mp.weixin.qq.com/s?__biz=MzU4NDUyNTUwMQ==&mid=2247484985&idx=1&sn=d7c6e6d94cdb91c287b80b55191a49f2&scene=21#wechat_redirect)

4、 [用Claude Code效率提升10倍后，依旧感谢被Cursor虐的五个月](https://mp.weixin.qq.com/s?__biz=MzU4NDUyNTUwMQ==&mid=2247484979&idx=1&sn=1d88c62fac6f23d59dc5aa6d4e3b27a8&scene=21#wechat_redirect)