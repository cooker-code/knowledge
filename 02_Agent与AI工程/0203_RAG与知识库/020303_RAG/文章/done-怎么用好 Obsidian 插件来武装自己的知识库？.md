> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020303_RAG/020303_核心知识点/RAG生命周期与评估边界|RAG生命周期与评估边界]]
---
title: 怎么用好 Obsidian 插件来武装自己的知识库？
author: Next蔡蔡
date: 蔡蔡啊蔡蔡啊
url: https://mp.weixin.qq.com/s?__biz=MzIyMzk3MTEwNQ==&mid=2247488133&idx=1&sn=1df5c2a6508d1312c4d01e8268598b6b&chksm=e90b53f90e07d1f3160c6a6bf6a3fc4b98b5be66487513108c579075f9be9faf3050f4c9bf5b&mpshare=1&scene=24&srcid=0515VRMk7GcpHv2YCqQvkwZr&sharer_shareinfo=28128c9c9d7c3a1fc52b2fca829a2bc2&sharer_shareinfo_first=28128c9c9d7c3a1fc52b2fca829a2bc2#rd
---

哈喽各位精神股东们，我是Next蔡蔡！

之前分别分享了 [Obsidian 知识库的内容获取](https://mp.weixin.qq.com/s?__biz=MzIyMzk3MTEwNQ==&mid=2247488038&idx=1&sn=07005fb47403da8a317550d8f34ce8dc&scene=21#wechat_redirect)、[AI 接入](https://mp.weixin.qq.com/s?__biz=MzIyMzk3MTEwNQ==&mid=2247488072&idx=1&sn=082bf0ab21906e91d2c99e27e13a0566&scene=21#wechat_redirect)、[wiki 搭建](https://mp.weixin.qq.com/s?__biz=MzIyMzk3MTEwNQ==&mid=2247487993&idx=1&sn=1554f77b3ba2d13d38686efff7ec3296&scene=21#wechat_redirect)，以及[多端同步](https://mp.weixin.qq.com/s?__biz=MzIyMzk3MTEwNQ==&mid=2247488105&idx=1&sn=171f9919fbb516c0a18ba5479bda73a8&scene=21#wechat_redirect)，如果你细心的话，会发现有个东西贯穿其中，那就是插件。要知道，有无安装插件的 Obsidian 简直是两个样。

今天这篇文章就是分享我是怎么用好插件来武装 Obsidian 知识库的，分为两部分：

1） 我常用的 7 个 Obsidian 插件 & 为什么用它们

2）如何用 Claude Code 开发自己的 Obsidian 插件？

在准备发布这篇文章前，我突然发现就在昨天，Obsidian 官方发布了 Obsidian Community，你可以把它理解为 Obsidian 的插件与主题社区，不仅支持用户浏览，还支持开发者上去提交、认领和管理自己的插件。

我在文章最后也会补充介绍下它的使用。

一、7 个实用 Obsidian 插件

Obsidian 的插件系统有两套， 一套是官方的核心插件，另一套是第三方插件。

官方的核心插件就有 20 多个，不用安装，正常开启或关闭即可。新手可以先从核心插件开始熟悉，如果它们不满足需求，再去找可替代的第三方插件或考虑自己开发都不迟。这里就不过多展开。

今天主要介绍的是第三方插件，一共 7 个，都是我常用的，可以分为四类：

* **基础增强层**：BRAT（安装未上架插件）+ Git（知识管理和备份）
* **AI 增强层**：Claudian（让知识库真正“活”起来）+ Terminal（补充 Claudian 的不足）
* **导航与发现层**：Notebook Navigator（让知识可被高效检索和组织）
* **内容创作层**：Excalidraw（增强视觉思考）+ Local Images Plus（解决剪藏图片本地化问题）

不过在安装这些第三方插件之前，我们得先在“设置-第三方插件”中关闭安全模式，才能开启社区插件市场并完成后的搜索、安装和启用。

基础增强层：BRAT（安装未上架插件）

Obsidian 官方对第三方插件的上架审核非常严格，完整流程跑完可能得好几个月，所以部分开发者就没有将自己的插件上传到社区插件市场中，比如热门的 Claudian、Opencode 之前就没上架到 Obsidian 社区插件市场（最新消息：Claudian已经上架了）。

在没有 BRAT 插件之前，大家安装这类插件的方法比较麻烦：需要在对应插件的 GitHub 仓库找到并下载 main.js、manifest.json 和 styles.css 这三个文件，再去 ~/.obsidian/plugins 这个隐藏文件夹下创建相应文件夹。

这种方法对于非开发者来说，太麻烦了。

不仅得在一堆文件中找到 main.js、manifest.json 和 styles.css ，还要知道怎么找到隐藏文件夹。虽然现在已经能让 AI 帮忙执行这个流程，但前提是你已经安装了 IDE 和 CLI 这类 Coding 工具，这又是另一种麻烦。

这时候 BRAT 插件的优势就显现出来了。

只需要在社区插件市场安装 BRAT 插件，接着在 BRAT 插件的设置界面找到“Beta plugin list”版块，点击“Add beta plugin”；

在弹窗中添加我们安装的 beta 插件的 GitHub 链接，等识别链接后选择要安装的版本（一般选择 latest version 最新版本），最后点击“Add plugin”就搞定了。

是不是简单多了。

有了 BRAT 插件后，我们就可以安装任何不在社区插件市场中的插件了，包括我们后续自己开发的。

AI 增强层：Claudian & Terminal

之前在[怎么给 Obsidian 接入自己常用的 AI](https://mp.weixin.qq.com/s?__biz=MzIyMzk3MTEwNQ==&mid=2247488072&idx=1&sn=082bf0ab21906e91d2c99e27e13a0566&scene=21#wechat_redirect)这篇文章中有介绍过 Claudian 和 Terminal 这两个插件。

前者是 Claude Code 的 Obsidian 版本（底层基于 Claude Code CLI），后者则是用来运行 Claude Code CLI、Codex CLI 等工具，用来补足 Claudian 的缺陷。

关于它俩的安装，可以移步查看之前的文章，就不做赘述。

这里补充回答之前观众提出的一个问题：什么时候用 Claudian，什么时候用 Terminal + Coding CLI？

如果你每次操作的文件数量不多，那么大部分的知识管理任务都能用 Claudian 插件解决；

相反，如果涉及大量文件操作，比如把大量旧笔记迁移到新结构、批量修改上千条笔记的 frontmatter 等，使用 Claudian 插件比较容易出现卡慢的情况，这种就更建议去跑 Terminal + Coding CLI。

还有当你打算 Vibe Coding 自己的 Obsidian 插件，那也更建议去跑 Terminal + Coding CLI 或者 AI IDE，因为所有操作都 会更直观。

基础增强层：Git（版本管理和备份）

无论你采用 Claudian 还是 Terminal + Coding CLI，都存在文件被 AI 误操作找不回来的情况。但如果你提前用 Git 做好版本管理和备份，那就不用有这种后顾之忧。这也是我把基础增强层的 Git 放在 AI 增强层后面来介绍的原因。

关于怎么用 Github + Obsidian Git 插件进行版本管理和备份，可以查看之前这篇文章[如何找到最适合自己的 Obsidian 多端同步方案？](https://mp.weixin.qq.com/s?__biz=MzIyMzk3MTEwNQ==&mid=2247488105&idx=1&sn=171f9919fbb516c0a18ba5479bda73a8&scene=21#wechat_redirect)

我们只要完成了第一次备份，后续知识库有任何修改，都可以借助 Obsidian Git（开发者是 Vinzent）来实现自动提交和同步。对应调整这三个配置项就可以轻松实现：

* Auto commit-and-sync interval（minutes）：每隔多少分钟自动提交和同步一次；
* Auto commit-and-sync after stopping file edits：在停止编辑文件后自动提交和同步；
* Pullup on Start：当启动 Obsidian 时，会自动将 GitHub 的变动同步到本地仓库。

如果你决定长期使用 Obsidian，那么用 Git 做版本管理和备份是有必要的。

导航与发现层：Notebook Navigator（让知识可被高效检索和组织）

Obsidian 的默认文件资源管理器是比较原始的，只能看到文件名，没有内容预览；标签和文件夹是两个完全独立的面板，切换麻烦；如果是大型知识库，在里面找笔记会非常低效。

Notebook Navigator 就是为了解决这些问题开发出来的，当你安装并启用后，整个界面就会变成这样：

对于刚接触  Obsidian 不久的精神股东来说，可能不太能看出区别，我这么说吧：

最左侧的视图整合了默认界面的文件、标签、属性、预览和快捷操作。可以直接实现以前需要五六个插件才能搞定的能力。

比如“显示最近文件”这个能力，以前要通过安装 RecentFiles 这个第三方插件才能实现；

又比如“将标签作为文件夹，以标签的形式显示所有文件”，以前要用 TagFolder 插件才能实现；

还有“重命名、合并、切换和搜索标签”，以前则要借助 Tag Wrangler 插件……

是不是一下子就 get 到 Notebook Navigator 的价值了。

不仅如此，开启 Notebook Navigator 插件后还会多出中间这列视图：当我们点击左侧某个文件夹时，里面包含的文档就会以卡片形式在这列视图展示出来，不仅有标题，还有摘要、日期，甚至可以看到图片预览。

这样我们不用打开具体的笔记，就能快速判断大概内容。尤其是那些写了很久的笔记，一眼扫过去，就能迅速唤起记忆。

内容创作层：Excalidraw（增强视觉思考）

Excalidraw 你可能自己没用过，但一定在不少地方看过用它绘制的手绘风格的流程图、架构图、思维导图等。

这个插件不是把 Excalidraw 的“画图”能力简单搬过来，而是把 Excalidraw 深度集成到 Obsidian 中，让我们的知识库同时具备文字 + 视觉思维的能力。

比如 Excalidraw 插件创建的绘图文件就是 Markdown 文件，所以可以参与 Obsidian 的搜索、双链、图谱、Backlinks、标签、属性等。

**还可以做元素级双向链接**：在画布里可以直接把图形、文字框链接到 Obsidian 笔记、文件、块，甚至其他 Excalidraw 绘图。反过来，笔记里也可以嵌入绘图。

真正实现“笔记即画布、画布即笔记”的工作流。甚至有些人是为了用 Excalidraw 插件才用 Obsidian 的。

内容创作层：Local Images Plus（解决剪藏图片本地化问题）

之前在《如何把Obsidian Web Clipper价值最大化？》文章底下留言，怎么让剪藏插件实现图片离线下载。

我原本是用 Obsidian 官方的“附件+自定义快捷键”实现，就是在剪藏内容前，在软件中做好两步配置：

**第一步：选择图片默认存放路径。**在“设置-文件与链接”中配置“附件默认存放路径”，我选择是“指定的附件文件夹”；

**第二步：配置自定义快捷键。**在“设置-快捷键”中搜索“附件”，找到“下载当前文件内的所有附件”，给它配上自定义快捷键，比如 command+shift+D。

这样当你每次剪藏完文章，如果想要将图片保存到本地，就可以按自定义快捷键，文章对应的图片就会保存到你指定的附件文件夹中。

但上面的方法，用 Local Images Plus 这个插件就能解决，安装后你什么都不用管，每次剪藏完文章，插件就会自动给你处理图片下载。

如果你不想本地图片太占空间，可以在插件自定义是否要压缩，以及保存的图片质量等。

以上就是我常用的 7 个 Obsidian 插件，不多，但每个都在用。

* **基础增强层**：BRAT（安装未上架插件）+ Git（知识管理和备份）
* **AI 增强层**：Claudian（让知识库真正“活”起来）+ Terminal（补充 Claudian 的不足）
* **导航与发现层**：Notebook Navigator（让知识可被高效检索和组织）
* **内容创作层**：Excalidraw（增强视觉思考）+ Local Images Plus（解决剪藏图片本地化问题）

装 Obsidian 插件是一件很容易上头的事——看到一个功能心痒，冲动安装，然后就没有然后了。

比如社区很热门的 Templater、Dataview，以及 Style Settings 插件，我之前也安装过，后面因为没啥场景就基本没用。

第三方插件绝对不是越多越好，因为我们真正常用的插件并不多，装太多插件反而容易影响 Obsidian 的启动和运行速度。

我之前还追求过“All in One”，就是用很多插件把 Obsidian 打造出另一个工具的能力，比如任务管理，但实际体验后发现，这些插件其实都很难做到一些垂类工具的体验，所以后面也舍弃了。

我现在的做法就是：每装一个插件之前，先问自己两个问题：

**“我的具体使用场景是什么”，**找不到就不装；

**“这个插件有比垂类工具更好用吗”**，没有的话也不装。

这样留下来的，都是能真正能让我减少知识库管理摩擦、提升长期维护便利的插件。

二、如何用 Claude Code 开发自己的 Obsidian 插件？

Obsidian 社区插件市场上架的插件接近 4000，说多不多，说少也不少。由于每个人都有些知识管理的个性化需求，现有插件不一定都能满足。

比如我之前在找翻译插件时，发现很多都不是我想要的，甚至有些翻译插件已经一两年没更新维护了，要是将就用就会很难受。

这时候开发一个适合自己需求的个人插件，明显会更好。

以 Claude Code 为例（默认你们已经安装了），具体可以这么操作：

步骤一：梳理你的 Obsidian 插件需求

目前大部分 Coding Agent 对 Obsidian 插件的开发都不熟悉，所以我们最好给到它相应的官方文档，并且告诉它参考模板长怎样，最后将得到的 main.js、manifest.json 以及 styles.css  文件放到 .obsidian/plugins/<插件名称>/ 目录下即可。

```
附上提示词模板：{你的 Obsidian 插件需求：- 功能点 1：**- 功能点 2：**}插件的具体开发可以参考这两份官方文档：- Obsidian 插件创建文档：https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin- Obsidian 插件模板：https://github.com/obsidianmd/obsidian-sample-plugin
```

**步骤二：用 Superpowers skill 或 Plan Mode 完善需求**

简单/快速任务直接用 Claude Code 自带的 Plan Mode（目前基本所有 Coding Agent 都有自带的 Plan Mode）

中大型功能、复杂逻辑任务就用 Superpowers skill，用 /superpowers:brainstorming （即头脑风暴）帮你梳理清楚自己没想到的一些需求，接着用 /superpowers:writing-plans 写下计划。

Superpowers skill 🔗：https://github.com/obra/superpowers

步骤三：将 main.js、styles.css、manifest.json 放到你指定的 .obsidian 插件文件夹中

这一步有两种方法，一种是让 Claude Code 帮你创建插件文件夹，并且将开发好的 main.js、manifest.json 以及 styles.css 放到指定的插件文件夹中；

另一种是让 Claude Code 帮你上传到 GitHub 仓库，然后借助前面提到的 BRAT 插件进行安装启用。这种方法更方便后续要上架的用户。

步骤四：不断将插件调试到自己满意为止

这一步就是对插件反复调试到自己满意的程度，如果你用的是 Claude Code + 国产模型 的组合，建议接有视觉能力的模型，这样你就能通过截图反馈了。

步骤五（可选）：将插件上架到 Obsidian 社区

其实到步骤四，就已经能解决你对个性化插件的需求了。但如果你希望别人也能用上自己的插件，就可以跑上架流程。

相比非常严格的上架审核， Obsidian 插件的上架流程本身并不难。

就在昨天，Obsidian 官方发布了 Obsidian Community，你可以把它理解为 Obsidian 的插件与主题社区，大家不仅可以在这里按分类浏览社区的各类插件，还可以查看各种插件的健康评分，来决定是否安装；

🔗：https://community.obsidian.md/

如果你是插件的开发者，还可以上去提交、认领和管理自己的插件。

提交的方法很简单，注册个 Obsidian 账户然后登录 Obsidian Community，绑定你的 Github 仓库然后添加插件即可。

🔗：https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin

最后，做个简单的总结：

Obsidian 插件不用多，但一定要有，有限看官方核心插件或者第三方插件有没有适合自己的，避免重复造轮子。

如果现有的第三方插件没能解决我们的特定需求，那就用 AI 手搓一个，毕竟现在也不难。

以上就是今天的全部内容，谢谢你看到了这里。

我是Next蔡蔡，《白话AI编程》书籍作者，Skills蓝皮书、DeepSeek自学手册 开源教程作者，持续分享 AI 编程、AI Agent，以及我的 AI 学习思考。

我们下期见 ~