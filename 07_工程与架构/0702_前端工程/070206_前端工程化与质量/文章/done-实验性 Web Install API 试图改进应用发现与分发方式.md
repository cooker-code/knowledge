> 已吸收至：[[07_工程与架构/0702_前端工程/070206_前端工程化与质量/070206_核心知识点/前端工程化工具链与质量门禁准则|前端工程化工具链与质量门禁准则]]、[[07_工程与架构/0702_前端工程/070206_前端工程化与质量/070206_知识地图|070206_前端工程化与质量知识地图]]

---
title: 实验性 Web Install API 试图改进应用发现与分发方式
author: InfoQ
date:
url: https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651280860&idx=4&sn=cec8e475643c2b7e94bc18364caa49d7&chksm=bca5c3e0209f6d358cb1bdf815b63d23e261e995580ec3955a31a31e6505dc68c59c19a2644c&mpshare=1&scene=24&srcid=0408fcbCeH5dkFdwHxc2YqdQ&sharer_shareinfo=8ec01948a40030c556c104a7530ddb90&sharer_shareinfo_first=8ec01948a40030c556c104a7530ddb90#rd
---

作者 | Bruno Couriol

译者 | 马可薇

一个新的实验性 Web Install API 目前已在微软 Edge 和 Chrome 中进入 Origin Trial（源试用）阶段。这个 API 允许开发者在应用内部的用户交互中，以编程方式触发 PWA 的安装提示。它的目标是简化软件的发现和分发流程，尤其适用于那些不知道浏览器地址栏里有安装图标，或者平时不使用应用商店的用户。

微软 Edge 团队发布的 官方说明文档 中解释了推出该 API 的动机：

> 终端用户目前并没有一种标准、跨平台的方式来获取应用。用户往往需要面对各种不一致、隐藏且专有的获取方式（比如自定义协议或应用商店）。
>
> Web Install API 的目标就是解决这个问题，提供一种开放、易用、标准化且跨平台的应用获取方式。

该提案的作者是微软 Edge 浏览器团队的 Diego Gonzalez。这也延续了微软过去对渐进式 Web 应用（PWA）的持续推动（例如 PWABuilder）。相比之下，苹果在 iOS 上对 PWA 的支持一直比较滞后，有时还会加入较多限制，这一情况过去也曾引起 欧盟方面的关注。

Web Install API 引入了一个新的 `navigator.install()` 方法。当应用安装成功时，它会返回一个 promise，并解析出一个 manifest\_id；如果用户拒绝安装（AbortError）或没有找到 manifest（DataError），则会报错。如果是在无痕 / 隐私模式下触发安装，这个 promise 也会被拒绝。

这个 Install API 的设计目标，是 替代或增强现有的`beforeinstallprompt` 事件，并让 PWA 可以通过直接的安装链接进行分发。

Diego Gonzalez 表示，目前多个浏览器厂商已经对推动这一提案的标准化和实现 表现出兴趣：

> 在 W3C WebApps 工作组范围内，Firefox、Safari 和 Chromium 已经同意推进“当前文档安装”的能力。同时也在讨论一种声明式的实现方式，因此在跨厂商层面是有进展的。不过，最终是否落地，还是取决于各个浏览器实现方。
>
> ……这也意味着在初期，我们只能安装当前浏览的页面，而跨站点安装仍在 WICG 中继续讨论。

目前这个 API 仍处于早期阶段，规范预计还会继续变化（比如跨站应用安装能力）。有兴趣测试并提供反馈的开发者，可以将 Chrome 或 Edge 升级到 143 以上版本；对于 139–142 版本，可以通过 about:flags 手动开启，或者参与 微软 Edge 的 Origin Trial。其他浏览器（如 Safari、Firefox）目前会忽略 `navigator.install`，继续使用各自已有的“添加到主屏幕”或“安装应用”流程。

声明：本文为 InfoQ 翻译，未经许可禁止转载。

**点击底部******阅读原文********访问 InfoQ 官网，获取更多精彩内容！****

今日好文推荐

[谷歌重磅开源Gemma 4！手机离线跑 Agent、还降内存，Qwen 被拉进正面对决](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651280694&idx=1&sn=7f775daf95e71304fd5ad7ac5fb1dcf4&scene=21#wechat_redirect)

[一个周末 + 1100 美元，干完 5 人 6 个月的活：Cloudflare 用 AI“复刻”Next.js，已跑进生产环境](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651280681&idx=1&sn=8cf3a59ecf58707f0546b7bc557797c7&scene=21#wechat_redirect)

[Claude Code 意外“开源”，51 万行源码曝光，但真正的秘密没有泄露](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651280544&idx=1&sn=9ae307e14adc5d554829569c3c3183e3&scene=21#wechat_redirect)

[Claude Code 过度设计，甚至不该给普通人用？OpenClaw 背后的Pi只留了 4 个工具](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651280184&idx=1&sn=4ba4c2ab4cb2bef777168e8a605e0dda&scene=21#wechat_redirect)