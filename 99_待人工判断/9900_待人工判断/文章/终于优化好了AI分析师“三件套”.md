---
title: 终于优化好了AI分析师“三件套”
author: 数字游牧日常
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxNjg0NDM2OQ==&mid=2658186331&idx=1&sn=918e07a53a15d0d0dd51229a47910178&chksm=817b9f9be44764d7bbeb84602d39d7e9ce82b33dbb5c781b07a2a0bcf169489f840f2e271408&mpshare=1&scene=24&srcid=11013V1jpc1NgMOrFQLL1J47&sharer_shareinfo=f5fcde6fdca7b2d09046ced2c91cd968&sharer_shareinfo_first=f5fcde6fdca7b2d09046ced2c91cd968#rd
---

经过长时间的尝试，终于放弃了大而全的努力，转而对于“AI分析师”的角色，用“三件套”来解决：Analyst（研究）；Slides（PPT演示）；Video（渲染）。

或许这样的方式更像是所谓AI原生，都在AI Studio的Build里完成。还为“三件套”的连通，使用Cloudflare做后台。

模型+Cloudflare，大概就是我设想里未来架构的全部，而经过一段时间的尝试，也初步验证了把除AI模型之外的部分全部搬移到Cloudflare上。当然，如果通过AI-Gateway，把模型API都wrap到Cloudflare上也是可行的。

巧合（不算巧合）的是下面截图里的例子，就是Cloudflare，NET。

Analyst

Slides

Video

有一个缺憾在这个架构下是很难解决了，因为AI Studio的Build的限制，我无法使用headless的ffmpeg来完成渲染，只能退而求其次选择了playwright的替代方案，视频质量相比录屏就有了明显的差距和不稳定。对了，Cloudflare支持docker了，下次有机会再做一套local的视频编辑和渲染组件扔到docker里去，就可以解决了。

不过，似乎意义也不很大。那就这样吧。

算是一项任务告一段落，但是，为什么to-do list还是越来越长了呢。