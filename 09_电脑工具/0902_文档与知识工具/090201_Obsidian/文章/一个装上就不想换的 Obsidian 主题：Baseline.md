---
title: 一个装上就不想换的 Obsidian 主题：Baseline
author: 毕小烦
date: 
url: https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978805&idx=1&sn=2c7a8c09b17972665f56640f9ca2ebda&chksm=bc6b37650638cb5c2f9fd1e2732a27614f1919b7f8aa0c3b1a3d57e922405c6ad104aa903ea2&mpshare=1&scene=24&srcid=0419CklTBeOE99fsHW01WlG3&sharer_shareinfo=6552ae0625d3c6592675284b62535525&sharer_shareinfo_first=6552ae0625d3c6592675284b62535525#rd
---

> 好主题就一个标准：不碰任何设置就该好用，想改的时候又能改成完全不同的样子。

---

我折腾过很多 Obsidian 主题。装一个，调一会儿 Style Settings，觉得不对劲，卸掉，换下一个。调好的那些 CSS 类和自定义颜色跟着一起没了，下次从头来。

这么来回几次，我发现问题不在哪个主题不好。是大多数主题的设计让你卡在两个选择中间：

一种个性很强，但你想改点什么处处碰壁。作者在 CSS 里写死太多东西了。

另一种可调的东西确实多，可默认状态很糙，你不花几小时调 Style Settings 根本没法看。

选哪种都别扭。

**仓库最后长什么样，是主题作者定的，不是你。** Baseline 就是想把这件事交回给你。

作者 aaaaalexis 也是 Cupertino 的开发者。Cupertino 拿了 2024 年 Obsidian 官方 Gems of the Year 最佳主题奖，把原生 macOS 风格复刻到了 Obsidian 里。做 Baseline 的时候作者就一个想法，装上就好用，想改能改到面目全非。

这个主题的副标题是 *The baseline of your thoughts* （思维的基线）。它想做地基，而不是墙纸。

## 默认状态就很好

如果你之前用默认主题，装上 Baseline 以后的变化有点说不清。最先注意到的是标题字体和间距，但更大的变化不是某个具体细节，是整体感觉「对了」。

### 从 Minimal 迁过来，已有的配置基本不用重调

Minimal 是 Obsidian 社区下载量最高的主题，作者是 kepano（现在的 Obsidian CEO Steph Ango）。很多人在上面花了不少时间：`cards`、`wide`、`table-lines`、图片滤镜、笔记属性里那一堆 CSS 类。换主题最怕的就是这些东西全废了。

Baseline 兼容 Minimal 绝大多数 CSS 功能类。外观换了，你笔记里写好的类基本照用。如果你还调过 Minimal 的 Style Settings，作者给了一个 迁移工具，把原来的配置贴进去自动转成 Baseline 格式，省得重新调。

不过官方说的是「绝大多数」不是「全部」。你仓库里如果有特别冷门的类，最好先在空仓库里试一下。

### 配色、布局、工作区，都在一个主题里

Obsidian 主题生态各有各的强项：有人喜欢 Catppuccin 的配色，有人离不开 Border 的工作区布局，有人习惯了 Minimal 的卡片和图表。以前你为一个配色装一个主题，为一个布局再装一个，还得担心冲突。

Baseline 把这些都收了。Style Settings 里可以直接切：Minimal 的配色、AnuPpuccin 的 Catppuccin 调色、Sanctum 和 Tiniri 的配色、Border 的工作区、Iridium 的 Frame 布局、仿 Craft 风格的 Cupertino / Fusion 工作区，全在一个主题里。

### 手机上也不拉胯

很多桌面好看的主题一到手机就崩：间距挤、菜单错位、字体发虚。Baseline 针对手机单独调过，导航、菜单、触控区域大小都是按触屏来的，不是桌面的缩小版。

### 怎么切过去

**步骤 1：装**

设置 → 外观 → 主题 → 搜 Baseline → 安装启用。完事。

**步骤 2（Minimal 用户）：迁移配置**

打开 迁移工具 → 贴进你 Minimal 的 Style Settings 配置 → 复制转换结果 → 贴回 Obsidian 的 Style Settings。默认主题过来的跳过。

**步骤 3：装 Style Settings（想深度改才需要）**

设置 → 第三方插件 → 浏览 → 搜 Style Settings → 安装启用。

启用后设置里会多一个「Style Settings」面板，配色、字体、排版、布局都在里面。不知道从哪改起可以去 Baseline Marketplace 找个预设导进去看看效果。不装这个插件，默认状态也够了。

**步骤 4：单篇笔记单独控制**

笔记属性区加 `cssclasses` 字段，填上类名，只影响这一篇。

### 最后

我本来打算装上试试几天，然后就换回去。结果用到现在，回不去了。

---

这些文章你可能也感兴趣：

* [在 Obsidian 中，标题就是一切](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978768&idx=1&sn=e58582b057ae74825dda99f373aab9ae&scene=21#wechat_redirect)
* [Obsidian Web Clipper 1.3.0 阅读器：这才是阅读网页文章该有的样子](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978760&idx=1&sn=24e4b25552c9e9b81f5bf18be5e11785&scene=21#wechat_redirect)
* [Obsidian 无界面同步客户端（Headless Sync）公测，同步这件事，终于不用我管了](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978726&idx=1&sn=69fabdf431c9568162271569c4c33abc&scene=21#wechat_redirect)
* [我每天都很忙，但我不知道自己在想什么](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978714&idx=1&sn=3be49187cf663379504e70cabb53e33e&scene=21#wechat_redirect)
* [没完没了地折腾 Obsidian，本来就是一种正经爱好](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978709&idx=1&sn=24d9b0f61a462172d5bab9a086ae5ea1&scene=21#wechat_redirect)
* [你的Obsidian笔记，敢让 AI 看吗](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978689&idx=1&sn=809bf4b49eabb09f53ede9345d5054ae&scene=21#wechat_redirect)
* [你用 Obsidian 做过最奇特的事情是什么？](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978684&idx=1&sn=93bfb2a2cd56d6eca8a365e24b903083&scene=21#wechat_redirect)
* [Markdown 如何征服全世界](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978620&idx=1&sn=32bd3c0ea91487f832c080307c535d81&scene=21#wechat_redirect)
* [Obsidian 白用了！别再傻傻打标签，试试 " 笔记即标签 "](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978498&idx=1&sn=a71a099352671dc8a1fbf79527b70a42&scene=21#wechat_redirect)
* [为 Obsidian 新手架桥：我将 Apple Notes 的 “永恒笔记” 系统移植了过来](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978461&idx=1&sn=ccb6990de2ae8b5fe28e8fbd1d8522f4&scene=21#wechat_redirect)
* [扔掉文件夹！Obsidian 高手管理海量笔记的四种方案](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978460&idx=1&sn=5567b3f022b259024494d14c47fe3f52&scene=21#wechat_redirect)
* [我终于放弃在 Obsidian 里管任务了，你们都在用什么？](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978459&idx=1&sn=e3728e8c3a5ee58a742b9cb25192b03e&scene=21#wechat_redirect)
* [别再看教程了！Obsidian 真正的高效学习法，高手只强调这三步](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978456&idx=1&sn=ff7e745ec27c0b6ad9c41c92fdc44cb5&scene=21#wechat_redirect)
* [Obsidian 笔记一团乱麻？4 个方法，让每条笔记都能被找到](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978455&idx=1&sn=24eafb1e658fe7ab981c698e5be179a8&scene=21#wechat_redirect)
* [不在电脑前时，如何快速将灵感存入 Obsidian？](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978364&idx=1&sn=b6554d8fe4221d968b94a9647f542571&scene=21#wechat_redirect)
* [拒绝哑巴链接：如何在 Obsidian 中为笔记关联注入灵魂？](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978357&idx=1&sn=e93e76c74acd2860ca83ccf1d040f21c&scene=21#wechat_redirect)
* [Obsidian 有可能被 IDE 替代吗？为什么？](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978255&idx=1&sn=73568f45244d778f940809550bce2937&scene=21#wechat_redirect)
* [用 Obsidian 的朋友，你们是怎么搞定多端同步的？](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978225&idx=1&sn=67b95a387339b490ef5d4e3e09cf146b&scene=21#wechat_redirect)
* [给笔记 " 起绰号 "：一个极被低估的顶级思维习惯](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978204&idx=1&sn=74707c1d8099923f900cd80073e761a0&scene=21#wechat_redirect)
* [2025 年度盘点：Obsidian 用户心中的 " 年度最佳插件 "](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978200&idx=1&sn=ef6fadf7044b5294be76ef39a1246e85&scene=21#wechat_redirect)
* [Obsidian 中的几千条笔记到底要不要分类？你是图稳还是图快？](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978196&idx=1&sn=647931a88c9ce7d5f6801d797b934cfc&scene=21#wechat_redirect)
* [深度！如果 Obsidian 没了，你的笔记是不是全废了？](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978192&idx=1&sn=59c3ceef3ad49ac478a7d3c7a65be1c2&scene=21#wechat_redirect)
* [笔记是用来用的，不是用来供的](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978184&idx=1&sn=3171fc37e6cc6c4d5922ee046d703a86&scene=21#wechat_redirect)
* [你记的大部分笔记，本来就是用来忘的](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978180&idx=1&sn=11782bc3fd77af7fb97d8a9010f3b023&scene=21#wechat_redirect)
* [把笔记系统删到只剩 4 个文件夹后，我终于开始工作了](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978171&idx=1&sn=19671391c1c0bbb9989b73776bd25716&scene=21#wechat_redirect)
* [Obsidian 每日笔记，从相见恨晚到因爱生恨，到底该怎么写？](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978163&idx=1&sn=9d5807c77f422cc84fb46c281392b341&scene=21#wechat_redirect)
* [Obsidian 数据库，一个就够还是 90 个更好？](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978152&idx=1&sn=06473f9c964e952888d41a20060d931a&scene=21#wechat_redirect)
* [山姆·奥特曼：为什么我只用螺旋笔记本](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978088&idx=1&sn=c40bc0a74b2f0ee20288c109a6854f3e&token=214977009&lang=zh_CN&scene=21#wechat_redirect)
* [Typora 封神之路：全网最全插件、主题、模板和工具合集](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978082&idx=1&sn=5e81f029147705f84bb954f6400bb01d&scene=21#wechat_redirect)
* [为什么 Obsidian 用户“不再有换工具的需求”？一个笔记应用是如何打败所有对手的？](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650978015&idx=1&sn=f74c8db86f32b6939f8d4e016aa9095e&scene=21#wechat_redirect)
* [Obsidian CEO 提醒的 10 个 Obsidian 数据库的使用技巧](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650977890&idx=1&sn=b004d95bd3e5272ed93d7d72adc5e1db&scene=21#wechat_redirect)
* [新手使用 Obsidian 的 10 个核心原则](https://mp.weixin.qq.com/s?__biz=MjM5NTMzNzQxMw==&mid=2650977866&idx=1&sn=0ffa82f7a21165a9daf0f651fde5b662&scene=21#wechat_redirect)