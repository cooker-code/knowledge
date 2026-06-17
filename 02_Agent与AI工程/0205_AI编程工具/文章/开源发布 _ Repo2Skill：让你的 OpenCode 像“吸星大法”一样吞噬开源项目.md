---
title: 开源发布 | Repo2Skill：让你的 OpenCode 像“吸星大法”一样吞噬开源项目
author: 硅基工程师
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg4MzYwMzE1Mw==&mid=2247483754&idx=1&sn=35b2d8b720fb1f77ad70b2ea13073bd7&chksm=cea060df12463f934ed49e032b25a8e7f99c3380182b481dee5006aad25fe32dea799e678909&mpshare=1&scene=24&srcid=0127Yx3k3gNvXFBt3gyPYZC9&sharer_shareinfo=9823d5b3851c4cc0ba1cf25c805aa370&sharer_shareinfo_first=9823d5b3851c4cc0ba1cf25c805aa370#rd
---

# 懒惰是第一生产力

上一篇文章，我教大家如何配置opencode.json与oh-my-opencode.json。 很多同学后台留言：“博主，OpenCode 真是太香了，免费模型都用起来了，还有其他黑科技吗？”

兄弟们，GitHub 是什么？ 它是程序员的军火库。 里面躺着各种大神写好的牛逼工具：视频下载神器 yt-dlp、网站压力测试 locust、漏洞扫描 sqlmap...

但是，这些工具有个通病：参数太难记了！想下载个视频，你得记住 yt-dlp -f 'bestvideo+bestaudio' --merge-output-format mp4 ...如果不常用的工具，每次用之前还得查半天文档。

思路打开：如果我们把这些工具封装成 OpenCode 的Skill，会发生什么？ 你只需要对 AI 说：“下载这个视频，最高画质。” AI 会自动帮你拼接那些复杂的参数，并调用工具执行。

今天，我就手把手教大家，如何把一个 GitHub 开源项目，变成 AI 随叫随到的“skill技能”。

作为一名极客，我的原则是：如果一件事需要重复做两次，那就写个脚本自动完成它。

于是，这几天我熬夜肝出了一个“终极 Skill”。 它的功能只有一个：生孩子。它可以把 GitHub、GitLab、Gitee 上的任意开源项目，“吃”进去，然后自动“吐”出一个封装好的 OpenCode Skill。

我给它起名叫：Repo2Skill。

# 见证奇迹：30秒“吞噬” yt-dlp

为了演示它的威力，我找了个硬骨头：yt-dlp。 这是全球最强的视频下载工具，功能强大但参数极多，普通人根本记不住那些复杂的命令行参数。

以前，你想把它做成 Skill，需要：

阅读它的 5000 字文档。

搞清楚 Python API 怎么调。

手写 meta.json 定义参数。

现在，有了 Repo2Skill，你只需要做一件事：

在 OpenCode 终端输入一行命令：

```
使用repo2skill，帮我把这个开源工具 https://github.com/yt-dlp/yt-dlp 打包成一个Skill
```

然后，repo2skill就会开始开始分析yt-dlp这个项目，然后开始规划要怎么打包封装成一个Skill。

规划了一通以后，OpenCode就分析完了，向用户确认几个问题。

给出你的回答之后，然后它就会继续规划，最终给一个非常明确的计划，等待你同意之后，就会开始实施和测试。

30s左右整个skill就能完成了。

然后你就可以用这个skill去下载全球200+网站的视频啦！

这就是“降维打击”。

我没有写一行代码，没有看一行文档，我就拥有了 yt-dlp 的全部能力。

# 开源：送给所有测试人和极客

我是做测试出身的，深知工具生态的重要性。 这么好用的东西，藏着掖着没意思。

我决定把它开源！

项目地址： 👉 https://github.com/zhangyanxs/repo2skill

目前支持特性：

✅ 支持 GitHub / GitLab / Gitee 链接

✅ 8 个 API 镜像自动轮换

✅ 官方 API + 代理支持

✅ 无 token 60 次/小时,有 token 5000 次/小时

✅ 完整的仓库结构分析

✅ 适配 GLM / Gemini / Claude 模型

# 怎么玩？

确保你已经安装了 OpenCode。

把我的Repo2Skill作为一个普通 Skill 安装进去（这是你唯一需要手动安装的 Skill，以后它就能帮你生万物）。

找一个你喜欢的开源库（比如 sqlmap, faker, requests）。

输入命令，开始套娃！