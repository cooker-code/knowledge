---
title: 我做了一款 AI 编辑 word 的 skill，推荐给你试试，效果惊艳
author: AI干货家老明
date: 干货老明干货老明
url: https://mp.weixin.qq.com/s?__biz=MzY5MjE4ODg5MA==&mid=2247483913&idx=1&sn=5fa537d8d1d4fe7327da2708a71e51b8&chksm=f5bc51ecb0caede762fdd5ecb836ce830c66945f6f9d5a6ce6a5f16f661f3359a2a0c7940a7b&mpshare=1&scene=24&srcid=0515crFUgfN5h9L70vZG1ulg&sharer_shareinfo=4e66d88b4357af12d4e512f24239f076&sharer_shareinfo_first=4e66d88b4357af12d4e512f24239f076#rd
---

https://github.com/sgsss998/AI-Word-Skill

已开源在 github，需要自取。

效果示例：（左：不使用 sop；右：使用 sop）

为什么有价值、核心价值点在哪：

| 维度 | 价值 |
| --- | --- |
| **时间** | 少做一整轮“全篇重排”或“手工对齐到哭” |
| **质量** | 合同、纪要、公文、标书等场景下，**版式稳定≈专业度** |
| **可解释** | 出问题能对上 **OOXML / run / 样式** 的原因，不靠玄学 |
| **核心抓手** | **母版副本 + 尽量只动 `run.text`（必要时清空同段其余 run）+ 表格别漏** ——这是本仓库的技术立场 |

具体的技术实现路径本文就跳过了，全都写在 skill 的 md 里面了。

总之，这个 skill 最大的价值在于：省却了大量手工排版 word 的时间。

没什么其他别出心裁的花样，是一个非常朴素的sop。希望它能够帮到你。