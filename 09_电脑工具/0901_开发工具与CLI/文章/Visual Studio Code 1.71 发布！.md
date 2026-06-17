---
title: Visual Studio Code 1.71 发布！
author: 玩转VS Code
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU1NjgwNTExNQ==&mid=2247494586&idx=1&sn=f6b91e4b7c7058a6c16c2b20434ecb20&chksm=fc3dd52ecb4a5c384892436c9c08d05e418c92f799fa8fcdb958278f1b11a90336e4ed2bc305&mpshare=1&scene=24&srcid=0903eTcamiGLznaDYeH1Tbsf&sharer_sharetime=1662177500170&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

出品 | OSC开源社区（ID：oschina2013)

Visual Studio Code 1.71 现已发布！

具体更新内容如下：

* 合并编辑器改进 - 文本和合并编辑器之间的转换更容易。

* 扩展的编解码器支持 - 帮助在 notebooks 和 webviews 中显示嵌入的音频和视频。

作为 VS Code 一部分提供的 FFmpeg 共享库以前只支持 `FLAC` 编解码器。在此版本中，该库已更新为支持以下编解码器和容器列表：

1. Vorbis
2. Flac
3. H.264
4. VP8
5. WAV
6. MP3
7. Ogg

* 文件重命名选择 - 按 F2 选择文件名、全名或文件扩展名。

* 新的 Code Action UI - 快速找到你正在寻找的 Code Action。

对 Code Action 控件进行了彻底的修改。现在不再是一个简单的 Code Actions 菜单，而是有一个自定义控件，可以更轻松地找到所需的 Code Action：

新控件还允许 VS Code 显示附加信息。例如，你现在可以将鼠标悬停在禁用的 Code Action 上以了解它们被禁用的原因：

* 终端更新 - Fish 和 Git Bash 的 shell 集成，新的平滑滚动。

+ 对 shell 集成进行了改进
+ 终端现在支持平滑滚动，它会在短时间内动画滚动，以帮助 n 在滚动后看到您的位置，类似于编辑器和列表。
+ 现在使用 kitty 终端首创的转义序列支持下划线样式和颜色。
+ 对终端渲染进行了几项改进

* Jupyter notebook 图像粘贴 - 在 notebook Markdown 单元格中粘贴和预览图像文件。

Jupyter 扩展现在允许用户将屏幕截图或图像文件粘贴到他们笔记本中的 Markdown 单元格中。目前仅支持 `image/png`mime 类型。要使用该功能，需添加 / 启用以下设置：

```
"ipynb.experimental.pasteImages.enabled": true"editor.experimental.pasteActions.enabled": true
```

* TypeScript 直播 - 在 YouTube 上观看 TS “速成课程” 或 “提示和技巧”。
* Live Preview 扩展 - Live Preview 现在支持 multi-root Web 项目。

* Markdown Language Server blog- 了解 Markdown 支持如何转移到语言服务器。

更多详情可查看官方公告：https://code.visualstudio.com/updates/v1\_71

**推荐阅读：**

* [全宇宙首本 VS Code 中文书，来了！](http://mp.weixin.qq.com/s?__biz=MzU1NjgwNTExNQ==&mid=2247485774&idx=1&sn=fba4ad7f8ad9bf339f9190f1d5fb6662&chksm=fc3e37dacb49beccf20cb3d7230a176802ea9401d52ca77e7cab003c7ce3896ca49bfb61e4a8&scene=21#wechat_redirect)
* [Code Runner for VS Code，下载量突破 4000 万！支持超过50种语言](https://mp.weixin.qq.com/s?__biz=MzIwODE4Nzg2NQ==&mid=2650557656&idx=1&sn=b769a31a839fb14215430deffc0cd774&scene=21#wechat_redirect)
* [微软也爱 Python！VS Code Python 全新发布！Jupyter Notebook 原生支持终于来了！](http://mp.weixin.qq.com/s?__biz=MzU1NjgwNTExNQ==&mid=2247484345&idx=1&sn=f541c502d69c4520a7ed50a141bb3386&chksm=fc3e3d2dcb49b43b668ab46c888e1433a0c177b92dd5adc91fa82766f42bd3c20beb07a731a9&scene=21#wechat_redirect)
* [微软也爱 Java！微软在 SpringOne 大会上宣布 Azure Spring Cloud 云服务！](http://mp.weixin.qq.com/s?__biz=MzU1NjgwNTExNQ==&mid=2247484340&idx=1&sn=6e03e88e42f326aeab6e8dfa535aad16&chksm=fc3e3d20cb49b4366c58a29a7c9cf27f795e9ac2014e89e46810c5fb6ec2ba4c7de7f5170c48&scene=21#wechat_redirect)
* [在微软（Microsoft）工作是怎样一番体验？](http://mp.weixin.qq.com/s?__biz=MzIwODE4Nzg2NQ==&mid=2650548708&idx=1&sn=389ac3e83d6c3cd7a54164581b0a3284&chksm=8f0e7152b879f844b80626296324dbf26ebefe82c48525d558617e6ef09a570bdb8d6033102f&scene=21#wechat_redirect)
* [微软内推，长期有效](http://mp.weixin.qq.com/s?__biz=MzIwODE4Nzg2NQ==&mid=2650548833&idx=1&sn=91c40a5133435a9b9b3d989e03af69e1&chksm=8f0e72d7b879fbc1afd382a36600552da15d4e90c9515f8109c701866d39e2ee8c0fafab16e1&scene=21#wechat_redirect)
* [代码编辑器横评：为什么 VS Code 能拔得头筹](http://mp.weixin.qq.com/s?__biz=MzU1NjgwNTExNQ==&mid=2247483834&idx=1&sn=c836662c60bf035e726bb82345453d4f&chksm=fc3e3f2ecb49b638bb58eb39df7646b0b180d43008f6bf9e9fadcf9390f79e8f4e0f7b4d3ec2&scene=21#wechat_redirect)
* [知否知否，VS Code 不止开源](http://mp.weixin.qq.com/s?__biz=MzU1NjgwNTExNQ==&mid=2247483756&idx=1&sn=d4faca9d8f69b67e6e1833431531c831&chksm=fc3e3ff8cb49b6eebe63083a2080d9d304d38604fbb608544249f5dda3311760f7efa6367c35&scene=21#wechat_redirect)
* [那些年，我们一起追的 VS Code](http://mp.weixin.qq.com/s?__biz=MzU1NjgwNTExNQ==&mid=2247483798&idx=1&sn=751467ecb2fc58d6aa7824ff123ef8cb&chksm=fc3e3f02cb49b6148ac69c87e24972da9b016a8257b0144ab565c20c9eb755cb2d673f233b00&scene=21#wechat_redirect)

**玩转VS Code**

VS Code **·** 编程开发 **·** 业界资讯