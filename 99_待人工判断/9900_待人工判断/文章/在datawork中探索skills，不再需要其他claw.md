---
title: 在datawork中探索skills，不再需要其他claw
author: 傲骄笔记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIyNDY4ODU0NQ==&mid=2247485116&idx=1&sn=94933ce5b0328b82a9652c3252991a88&chksm=e99a8ede77d7c79298b2b418fc9d88d1691110ed0ca25c0eb305b7498b03a7c25d408c96afbd&mpshare=1&scene=24&srcid=03250j3CZmGw4jPs6eiQoUof&sharer_shareinfo=a6ec1946c4b733bf13d9238b678f267d&sharer_shareinfo_first=a6ec1946c4b733bf13d9238b678f267d#rd
---

刻意紧急更新了一个版本，做了很多专门的优化，以方便大家在datawork中就能探索skills，而不用再去折腾其他claw。目前，测试效果很好。

主要更新内容如下：

* agent：改进了上下文压缩机制，现在，agent运行时，能在速度、成本、质量之间有更好的平衡；避免了因为压缩频率太高而出现无效循环的问题；已适配deepseek、kimi、xiaomi三家供应商
* web-chat：web工作台中的chat功能，已完成ui完成重构，现在界面更简洁；能展示思维链；无论在电脑端还是手机端，对话时切换或关掉页面，也不会导致中断
* PPT工作台：PPT工具系列更完整，现在操作更稳定

关于skills，早上刚刚做的测试，在datawork中，只给了一段简短的提示，agent就自行完成了skills技能的安装部署、下载，效果超出预期。

datawork主页：

https://publish.obsidian.md/xm/wiki/%E6%95%99%E7%A8%8B/datawork%EF%BC%9Aabout

2026.3.21