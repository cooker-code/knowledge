---
title: Flutter 开发效率提升 200%：我的私藏工具库清单（附项目模板）
author: Flutter中文社区
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4MzM3MjEwNw==&mid=2447565068&idx=1&sn=540b85b0c1b030572ce2f7cf8a982be6&chksm=8a0ccd98df1975b37a0ade4bfa9e53c27dc9544569124d70f27059e1be43245745ad0cd75175&mpshare=1&scene=24&srcid=1125MGBA0SZsCgg40foy0Ch4&sharer_shareinfo=6bdda482c5e39ee7576bfb11ab9876fb&sharer_shareinfo_first=6bdda482c5e39ee7576bfb11ab9876fb#rd
---

很多刚接触 Flutter 的兄弟问我：“为什么我写个简单的界面要堆那么多代码？”、“屏幕适配怎么搞最快？”。 其实，Flutter 生态已经非常成熟，很多功能完全不需要我们手写。作为开发者，善用社区优秀的轮子，能让我们把精力集中在业务逻辑上。 今天「Flutter 中文社区」为大家整理了 7 款我个人项目中必装的“神级”库，每一个都能帮你节省大量开发时间。 **（文末有彩蛋，我把这些库封装成了一个开箱即用的通用模板，记得领取！）**

1. 屏幕适配王者：flutter\_screenutil

做移动端开发，最头疼的就是 UI 妹子给你发了一张 iPhone 的设计稿，结果你在 Android 各种奇葩分辨率上跑起来面目全非。

`flutter_screenutil` 是目前 Flutter 社区最成熟的屏幕适配方案。它能让你的程序在不同尺寸的屏幕上，展示出和设计稿一模一样的效果。

2. 网络请求标配：Dio

如果你还在用原生的 `HttpClient`，请立刻停止。Dio 是 Flutter 界的 OkHttp/Retrofit，功能极其强大。

3. 布局间距神器：Gap

还在到处写 `SizedBox(height: 10)` 吗？Flutter 官方力推的 Flex 布局虽然好用，但处理间距总觉得不够优雅。

`gap` 库完美解决了这个问题。它会自动判断你是处于 Row（水平）还是 Column（垂直）中，自动适配间距。

Dart

```
// 以前的写法Column(  children: [    Text('Hello'),    SizedBox(height: 16), // 你得记住这是垂直的    Text('World'),  ],)// 用 Gap 的写法Column(  children: [    Text('Hello'),    Gap(16), // 不用管横竖，自动识别    Text('World'),  ],)
```

4. 告别原生 Log：Logger

调试的时候满屏全是 `print`，想找报错信息简直是大海捞针。`logger` 库能输出漂亮、格式化、带颜色的日志。

5. 弹窗交互：BotToast

Flutter 自带的 Dialog 和 SnackBar 有时候限制太多，比如想在这个页面弹窗，但不想这就必须得有 Context。

`bot_toast` 是一个非侵入式的弹窗库，你可以在任何地方（甚至是网络请求的拦截器里）调用它，完全不需要 Context。

6. 本地数据持久化：shared\_preferences (配合 sp\_util)

保存用户的登录 Token、深色模式设置，杀掉 App 后还在，这就需要用到本地存储。虽然官方提供了插件，但建议配合简单的封装工具类使用。

7. 启动图标一键生成：flutter\_launcher\_icons

App 开发完了，要换图标。你需要去切 Android 的 mipmap 5套图，还要切 iOS 的 AppIcon... 太累了。

这个库只需要你在 `pubspec.yaml` 里配置一张高清大图，运行一行命令，它自动帮你生成所有平台需要的尺寸并替换。

🎁 粉丝福利（重点！）

看了这么多库，一个个去配置、写封装代码是不是很麻烦？

为了方便大家快速上手，我花费周末时间，基于以上推荐的库，搭建了一套【Flutter 企业级通用开发模板】。

**模板包含：✅ 已集成 Dio 并封装了全局拦截器与异常处理 ✅ 已配置 Screenutil 屏幕适配 ✅ 封装了通用的 Toast 工具类 ✅ 目录结构已按 MVVM 规范划分好**

**获取方式：此公众号回复："模板 " 即可获取链接**

拒绝重复造轮子，让我们一起高效开发！👇