> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020507_Codex/020507_核心知识点/Codex工程使用与沙箱边界|Codex工程使用与沙箱边界]]
---
title: claude、codex工作状态灯被我开源了
author: 谁动了我的芝麻酱
date: yoyoyoyo
url: https://mp.weixin.qq.com/s?__biz=MzAwMDUzNzgwNg==&mid=2247483782&idx=1&sn=ddf82951a2e885fde8491bd26955e726&chksm=9b54f0aef976356c5be2491e6596df2dc146d7d2ac2ace2164d285b67eb5d457837e8dbf2aaa&mpshare=1&scene=24&srcid=0602wZuIBUVEBrP2nKyqdqLL&sharer_shareinfo=f3e7fe7c7d14b0746ac165768ccbb26a&sharer_shareinfo_first=f3e7fe7c7d14b0746ac165768ccbb26a#rd
---

一开始看到别人整的 claude 的状态红绿灯蛮有意思，就寻思着自己也搓一个。就开始找配件，准备买买买的时候，羞涩的钱包告诉我，可以先搞个不花钱的😂。

于是就有了写一个在状态栏展示claude、codex工作状态的红绿灯，解决我的最大的问题就是：有时候会同时干好几件事情，就会忘记去确认任务。等想起来的时候，都过去好久了。

装上这个状态灯之后，我就不用切换屏幕，抬眼一看，就知道它们工作得怎么样了。

我给他们定义了几个工作状态：

|  |  |  |
| --- | --- | --- |
| 颜色 | 状态 | 含义 |
| 灰色 | 空闲 | 没正在运行 |
| 绿色(闪烁) | 工作中 | 正在执行 |
| 黄色(闪烁) | 等待中 | 需要确认下一步操作 |
| 红色(闪烁) | 错误 | 出事了，快去终端看看 |
| 蓝色 | 完成 | 任务结束 |

安装方式有两种：

1. 可以从源码构建

```
git clone https://github.com/cuihuapeng/code-light-ai.gitcd code-light-aipnpm installpnpm tauri build
```

2. 也可以直接安装已编译好的

使用的话也比较简单

如果使用的是Claude 就点击 Setup Claude Hooks按钮进行初始化；如果是Codex 就点击 Setup Codex Hooks按钮进行初始化。

正常情况下他就可以愉快地工作了。

一些注意事项README.md 也有详细介绍：

```
https://github.com/cuihp/code-light-ai/blob/main/README_CN.md
```