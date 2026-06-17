---
title: IDEA 好用的插件 CodeGlance Pro
author: MyBatis
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA5ODgyNTk2OQ==&mid=2649791315&idx=1&sn=eed68d8bf6c32fc3683262daf2f2711e&chksm=888f98cdbff811dbbbbdc151a5a5e569079743644cbe99c4044b41f0406ba7c868ec633bdfb6&mpshare=1&scene=24&srcid=1201PsLTPbomipkMslt9xDBT&sharer_shareinfo=55f6ef7bb068b877579394f6bd88ec59&sharer_shareinfo_first=55f6ef7bb068b877579394f6bd88ec59#rd
---

在使用 IDEA 的过程中，想拖动滚动条时经常看不到滚动条的位置，而且滚动条特别窄也不方便点击，所以就想在 IDEA 中找一个插件能像 VS CODE 那种带预览的滚动条。

用一些关键词搜索后看到了几个 CodeGlance 名字的插件，其中一个带着 Pro，心里想着带 Pro 的会不会是需要收费的插件。

安装之后发现完全免费，代码也开源，赶紧私有方式import一份私有库，避免类似 MyBatis 插件的悲剧。

这个插件可以显示代码区域和日志区域的小地图，配置页面中还能配置这两部分的宽度等信息，使用起来非常方便。

在我打开的各种工程中没见到几千行甚至上万行的代码，使用非常流畅，没有卡顿。

尝试打开几万行的日志文件时就会把IDEA卡死，整个系统也会受到影响，同样的文件在VS CODE中打开和使用小地图拖动时很顺畅，不会卡。

为了避免卡死，可以设置 CodeGlance Pro的 Max lines count，改为一个较小的值例如2000，1000，500，当超出这个范围时就不会展示预览图，仍然是系统默认的滚动条。