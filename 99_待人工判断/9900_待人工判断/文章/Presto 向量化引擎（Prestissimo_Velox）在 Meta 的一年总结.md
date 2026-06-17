---
title: Presto 向量化引擎（Prestissimo/Velox）在 Meta 的一年总结
author: 过往记忆大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=2650740477&idx=1&sn=646279734d1ce60e51fdc6b6dd817d55&chksm=887c0b8bbf0b829d6b7d948ac60a72ecd08a61159b29b324dd8a178c21d9be6b9aa576b25927&mpshare=1&scene=24&srcid=1207yIY9rGacCoCafflONgGO&sharer_shareinfo=8bc2fe39df08e56ab2b9650a2ceb49a8&sharer_shareinfo_first=8bc2fe39df08e56ab2b9650a2ceb49a8#rd
---

本文资料来自2023年12月06日举行的 PrestoCon 大会的题为《Prestissimo: A Year In, The Path to Veloxification》的 PPT，分享者  - Amit Dutta, Meta Platforms, Inc。

Prestissimo是一个雄心勃勃的项目，它使用 C++ 并利用开源执行引擎Velox重写了 Java 版本的Presto worker。在这次演讲中，简要介绍 Velox及其各种组件，并讨论它给 Presto 带来的好处。介绍了 Velox 集成到Presto (Prestissimo)中的高级设计。Prestissimo 已经在 Meta 的生产中运行了一年多，我们将利用这个机会深入研究我们从实验平台工作负载中获得的知识。我们希望我们在生产环境中运行Presto的c++ worker的经验和发现不仅会对那些希望将Presto的性能提升到一个新的水平的人感兴趣，也会对其他考虑veloxification的引擎感兴趣。