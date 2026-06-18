---
title: PyMOL 3.1 基础使用教程（二）
author: 一半科研一半诗
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU5MjE2MjA3Nw==&mid=2247483818&idx=1&sn=2b14f5b95058b68a8d2ab9f082d952ee&chksm=fff58db7e6831404e6a996e507b6441b075aae5f668479bcd438002f5964b683f3618c59a790&mpshare=1&scene=24&srcid=0110mlyCH8DeG8Xzda4d8c5E&sharer_shareinfo=ea7b3158c78aace20e531d4bca8a4864&sharer_shareinfo_first=ea7b3158c78aace20e531d4bca8a4864#rd
---

紧接上篇[PyMOL 3.1 基础使用教程（一）](https://mp.weixin.qq.com/s?__biz=MzU5MjE2MjA3Nw==&mid=2247483810&idx=1&sn=be753409c2404c1d254cb83d9cc248e5&scene=21#wechat_redirect)，蛋白复合物的进阶操作以PDB #1HEW 为例进行演示。

4. 对象列表面板功能介绍

1. 在对象列表面板处，我们能看到All，待分析蛋白名称（此处为1HEW）和（sele），（sele）是selection缩写，表示此刻正在选择的对象。需注意，左上角可以更改选择对象为atoms, residues, chains or molecules，选择的氨基酸残基也会在展示的序列中被高亮。若需选择多个连续的氨基酸残基，只需要左击并拖拽即可，或者逐个点击序列中的待选择氨基酸。
2. 在列表右侧，我们会看到A, S, H, L, C几个字母，它们可以完成对蛋白复合体的基本操作，比如，蛋白结构的不同显示，结构的对齐，水分子的删除，着色和标记特定氨基酸等一系列基础操作。

   |  |  |  |  |
   | --- | --- | --- | --- |
   | A | Action | **用于对目前对象的整体性操作，包括视角调整、相互作用显示，结构对齐，删除复制重命名，氢原子相关操作等功能。** | / |
   | S | Show | 用于蛋白，侧链，主链和配体等对象的不同风格展示。 | show+style：展示此种style，故多次使用此操作后将获得多种style的叠加展示；show as+style：将此前的style删除并展示此刻需要的style。 |
   | H | Hide | 隐藏Show展示的不同style, 主链，侧链，水，氢原子等。 | hide everything包括隐藏标签。 |
   | L | Label | 用于标记出感兴趣的氨基酸残基和原子等。 | / |
   | C | Color | 将蛋白复合体按不同方式（原子，链，二级结构，b-factors等）着色。 | / |
3. 每个对象或特定选择对象右侧的 A、S、H、L、C 操作仅作用于该对象（或该选择），而对 All 进行的相同操作，则会同时作用于对象列表中的所有对象。
4. 对象列表的显示/隐藏可以通过点击每个对象进行调整，绿点代表显示，灰点代表隐藏。
5. 下面我们试着用上述学习的功能完成以下的练习：

【End】

如果这篇内容对你有帮助，欢迎点个「关注」支持一下 ❤️

【往期精彩】

[PyMOL 3.1 基础使用教程（一）](https://mp.weixin.qq.com/s?__biz=MzU5MjE2MjA3Nw==&mid=2247483810&idx=1&sn=be753409c2404c1d254cb83d9cc248e5&scene=21#wechat_redirect)

[FigTree 系统发育树分析教程（二）](https://mp.weixin.qq.com/s?__biz=MzU5MjE2MjA3Nw==&mid=2247483798&idx=1&sn=d316ee7c5ad6011cd01d56dd902cf1fa&scene=21#wechat_redirect)

[FigTree 系统发育树分析教程（一）](https://mp.weixin.qq.com/s?__biz=MzU5MjE2MjA3Nw==&mid=2247483791&idx=1&sn=547e1692e2138a563d934654d1b95ab7&scene=21#wechat_redirect)

[ChimeraX 1.10 使用教程](https://mp.weixin.qq.com/s?__biz=MzU5MjE2MjA3Nw==&mid=2247483733&idx=1&sn=8a002e658ac29d08e930c679e66ed6f9&scene=21#wechat_redirect)系列