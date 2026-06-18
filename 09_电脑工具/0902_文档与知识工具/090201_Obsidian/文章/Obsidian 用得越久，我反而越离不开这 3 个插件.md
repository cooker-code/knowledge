---
title: Obsidian 用得越久，我反而越离不开这 3 个插件
author: 艾康的AI自留地
date: 艾康在路上艾康在路上
url: https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247492570&idx=1&sn=802fa6312c5fdfc7fba2adc645304a78&chksm=c0bd3428c65a61cdb486e80d638fa5375bbee4d2f2897ffa904d1d892653a6f37c815e032bfa&mpshare=1&scene=24&srcid=05167AMr1wNjZnFbNP3Iqa0A&sharer_shareinfo=c80b99c9cf36a1549e0da1cbb2da1264&sharer_shareinfo_first=c80b99c9cf36a1549e0da1cbb2da1264#rd
---

见字如面，我是艾康。

**点击关注**👆防止迷路。****

 

> 本文字数 2292，阅读大约需 4 分钟

不知道大家在用 Claude Code（或者其他 Agent）操作 Obsidian 笔记的时候，有没有遇到过这样的情况：

让它帮你整理一下笔记，结果一个命令执行失误，整个文件被清空了？🤡

或者让它批量处理几个文件，跑完一看，内容全没了？🤡

我就遇到过🤣，而且是一次性 20 个文件，全部清零。

那次是让 Claude Code 执行一个相对复杂的任务，结果命令执行失误，它直接把 20 个 Obsidian 文件全部重置了。

但最后我的笔记一字未失。

拯救我的不是 Obsidian 自带的功能，也不是同步功能的版本回滚，而是一个第三方插件。

随着 Agent 的能力不断增加，让 Agent 直接接管 Obsidian 已经成了很多人的日常。

让 Agent 帮我们改写笔记、整理文档、批量操作文件，大多数时候很顺，但意外是真的会发生。一旦没有提前给笔记做好兜底，等问题出现的那一刻，你只能干瞪眼。

所以今天想跟你分享我每天都在用的 3 个高频 Obsidian 插件。除了上面那个，另外两个分别处理图片和搜索。

#### Edit History：自动保存每篇笔记的所有编辑历史

简单说，Edit History 会自动记录笔记的每一次编辑历史，支持随时浏览、对比、恢复到之前的任意一个版本。

> GitHub 地址：https://github.com/antoniotejada/obsidian-edit-history

它的工作方式很简单：每次你保存笔记的时候，它就在后台默默记一份快照。

所有快照都压缩存储在本地一个叫 `.edtz` 的文件里，不上传任何地方，纯本地。

那次 20 个文件被清空之后，就是因为`.edtz` 文件存在，能直接找到最近一次编辑进行恢复，恢复出来的内容跟事故前一秒完全一致，一个字都没少。

**但 Obsidian 不是自带「文件恢复」吗？**

你可能会问：Obsidian 不是已经有一个叫「文件恢复」（File Recovery）的核心插件吗？为什么还要再装一个第三方？

我的判断是：两个都开，互为补充。

它们最关键的区别，是**保留时长**。

官方「文件恢复」可以设置快照间隔和保留时长，但保留时长是有上限的。一旦过了这个时间窗口，旧版本就被清掉了。

而 Edit History 是**永久保留**的。只要你不主动删 `.edtz` 文件，所有历史版本都在。

也就是说：

* • 官方核心插件 → 防止「最近几天」的误操作
* • Edit History → 防止「任何时候」的误操作

我之所以那 20 个文件能找回来，是因为事故之前的编辑记录全部都在。

在 AI 越来越深地参与文件操作的今天，给笔记加上一个兜底的配置，一次安装，一劳永逸。

#### Image Converter：图片处理一站式解决

> GitHub 地址：https://github.com/xryul/obsidian-image-converter

写笔记的时候，图片是绕不开的部分。截屏、贴图、保存网图，每天都在发生。

但默认情况下，Obsidian 处理图片的方式挺粗糙的：

* • 文件名是 `Pasted image 20260515203147.png` 这种又长又没意义的字符串
* • 图片直接散落在仓库根目录，时间长了一片混乱
* • 几张高分辨率截图叠加起来，仓库体积膨胀得很快

Image Converter 就是用来解决这些问题的工具。

它能干的事很多，列几个我用得最多的：

* • **自动转格式**：粘贴进来的图片，自动转成 WEBP（体积小、清晰度够），原图扔掉
* • **自动压缩**：可以指定 Quality 值（1-100），在体积和清晰度之间自己取舍

* • **自动重命名**：用变量模板，比如 `{noteName}-{date}`，告别那串无意义的字符
* • **自动归档**：按规则把图片放到指定文件夹，仓库根目录从此干净
* • **显示尺寸调整**：在编辑器里直接拖图片边缘，或者用 Cmd + 滚轮（Windows 用户用 Ctrl），所见即所得

我自己的设置就是这 5 个能力的组合。从粘贴的那一刻开始，图片就已经是处理好的状态，压缩过的、命名好的、归档好的。

除此之外，它还支持图片注释、裁剪、旋转、翻转、批量处理整个 vault 等更进阶的能力。这部分我用得不多，大家可以自行探索。

#### Quick Switcher++：让 Obsidian 从「搜文件」变成「搜一切」

> GitHub 地址：https://github.com/darlal/obsidian-switcher-plus

Obsidian 自带的 Quick Switcher 是个好东西，但它只能搜文件名。

可是在 Obsidian 里，你想跳的往往不是「文件」本身，而是某个章节标题、某个标签、某个反链、某个嵌入引用。这些它都搜不到。

Quick Switcher++ 是它的增强版。在继续使用 Cmd+O 快捷键的同时，给它加上了一堆模式前缀。

这个插件有一个比较反直觉的配置环节，必须先处理好，否则用着用着会很迷惑。

原生 Quick Switcher 必须保持启用（不能禁用），但要把 Cmd+O 重新绑给 **Quick Switcher++: Open in Standard Mode。**

因为这个插件并不会自动接管 Cmd+O。

如果你保留原生快捷键，那么输入 `#`、`@`、`>` 这些前缀字符的时候，原生 Switcher 只会把它当成普通搜索字符，根本识别不了模式。

具体操作：

1. 1. 打开 设置 → 快捷键
2. 2. 搜索 Quick Switcher++
3. 3. 找到 **Quick Switcher++: Open in Standard Mode**，绑定 Cmd+O
4. 4. 如果提示冲突，把原生 Quick Switcher 的 Cmd+O 解绑，或者换成别的键

配置好之后，按下 Cmd+O，如果能看到下面这个界面就说明成功启用 Quick Switcher++ 了。

开头输入不同的前缀字符，就进入不同的模式：

* • `#`：搜所有笔记的标题（heading）
* • `@`：进入文件后再搜符号：标题、标签、引用、嵌入、callout
* • `>`：命令模式，部分替代 Command Palette
* • `edt`：切换面板（如 `edt backlink` 直跳反链面板）
* • `~` / `^`：查找相关文件

其中 `@` 是我用得最多的。尤其是长文、卡片笔记、MOC 这类内容超多的笔记，先 Cmd+O 搜到文件，再按 `@`，瞬间跳到你要的章节，比手动滚来滚去快太多。

`~` 和 `^` 这两个功能非常像，都是查找跟某个文件相关的文件，区别在于**以谁为源**：

* • `~`：以当前 Switcher 候选项里高亮的那个文件为源
* • `^`：以当前编辑器正在编辑的文件为源

它默认会找三类相关文件：

* • Backlinks：哪些文件链接到了源文件
* • Outgoing links：源文件链接到了哪些文件
* • Disk location：和源文件在同一目录或目录树里的文件

另外在插件设置里，有两个配置选项分享一下：

* • **Preferred suggestion title source** 改成 `First H1 heading`：如果你习惯用第一个 H1 当真实标题（而不是用文件名），改完之后，搜索结果显示的就是 H1 而不是文件名，更直观。如果你用 frontmatter 的 title 管标题，也可以改成 frontmatter 来源
* • **标题搜索范围**：保留默认的 `Search all headings`，这样搜到的不只是第一行标题，所有层级的 heading 都能搜到

用熟了之后，Cmd+O 几乎可以替代 Obsidian 一大半的导航操作。

#### 写在最后

这 3 个插件（Edit History、Image Converter、Quick Switcher++）分别解决了笔记安全、图片处理、搜索导航这 3 件事，加起来基本覆盖了我每天用 Obsidian 的几个高频场景。

它们都不在新手必装插件清单里，但用上之后，说不定也会变成你每天都离不开的插件之一。

如果你刚开始用 Obsidian，还没装过插件，可以先看我之前那篇《[Obsidian 插件篇：告别选择困难症，新手看这篇就够了](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247490235&idx=1&sn=eaa38d410b8e83353a812acb6eb07825&scene=21#wechat_redirect)》，里面分享了 10 个我认为新手可以直接装上的。

两篇一起看，会更系统。

 

以上，就是本文全部内容，如果觉得这篇文章对你有启发，点赞、比心、分享三连就是对我最大的支持，谢谢～

往期推荐阅读：

•  [Obsidian 从入门到进阶合集](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&__biz=MzkxOTY1MjA1Ng==&scene=1&album_id=4151857975356817416&count=3&token=1752453856&lang=zh_CN#wechat_redirect)

• [AI把我推成“知名”博主后，我发现了一条产业链](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247490247&idx=1&sn=627e1ae7bd0d01ecd3099de9e54cc451&scene=21#wechat_redirect)

• [用 Gemini 解锁 YouTube 新用法，信息获取效率提升 10 倍](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247490776&idx=1&sn=d1e6b83578bffabd3d74791108cca706&scene=21#wechat_redirect)

• [有了 NotebookLM 后，还需要 Obsidian 吗？](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247490868&idx=1&sn=f1193dcb7fade0042c0514e1d6ed4998&scene=21#wechat_redirect)

• [我试了 NotebookLM 学习法后，彻底抛弃传统学习方式](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247490943&idx=1&sn=270b19200d4d2a34221d3060a2a5bc1e&scene=21#wechat_redirect)

• [NotebookLM 再次升级，来自谷歌的年终礼物](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247491016&idx=1&sn=4ae9bc14a7453c647fcda68c24488e7b&scene=21#wechat_redirect)

• [我用 NotebookLM 解锁 PPT 的 5 种玩法，实现了 PPT 自由](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247491035&idx=1&sn=553471c458a5131fff2a089549183385&scene=21#wechat_redirect)

• [AI 时代，你的上下文才是最值钱的资产](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247491293&idx=1&sn=c5714360a5c36f1cfe6b83ce0a146958&scene=21#wechat_redirect)

• [2026 年如何用好 AI，我发现这些能力更重要](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247491553&idx=1&sn=58d7cfe10cdfcc87b2b9e8fc7b46c63d&scene=21#wechat_redirect)

• [Openclaw 这么火，可你真的需要它吗？](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247491593&idx=1&sn=4f902da23c117eed62e570355f1cab5b&scene=21#wechat_redirect)

• [万物皆可命令行：AI 时代，软件正在长出第二套界面](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247492041&idx=1&sn=f9b3b841305b17f1e658efe584d0dc8c&scene=21#wechat_redirect)

• [全网都在抄 Karpathy 的知识库，但大多数人只学到了皮毛](https://mp.weixin.qq.com/s?__biz=MzkxOTY1MjA1Ng==&mid=2247492131&idx=1&sn=f650d6171269f383f4f6363e068aa48c&scene=21#wechat_redirect)