---
title: Flutter 3.38 和 Dart 3.10震撼发布
author: 盛云希
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5NjgzNDA4MA==&mid=2247489560&idx=1&sn=e851e52208904cc42439f6b888876065&chksm=918e9522c18ac3e4962bf9cf0e3d6c7fff4530f135a2dccea3c48aa519cd4aced9602bb3f1de&mpshare=1&scene=24&srcid=1125CiD3xFjYfJMY6l3vRE0R&sharer_shareinfo=d6f8bc1d87f2504ecde5fcd78ff55e68&sharer_shareinfo_first=d6f8bc1d87f2504ecde5fcd78ff55e68#rd
---

**Flutter 3.38 和 Dart 3.10 现已发布**，而且它们非常强大！这些版本包含大量可用性增强和开发者工具的改进。没错，其中也包括人工智能。

#### Dart 3.10 的亮点：更具表现力、更简洁、更健壮的代码。

* **点语法：**减少输入，提高编码效率。现在`MainAxisAlignment.start`，您只需输入 `.` 即可，无需再输入 `.`。`.start`访问点号简写(https://dart.dev/language/dot-shorthands)页面了解更多信息。

```
// 点语法Column(  mainAxisAlignment: .start,  crossAxisAlignment: .center,  children: [ /* ... */ ],),// 之前Column(  mainAxisAlignment: MainAxisAlignment.start,  crossAxisAlignment: CrossAxisAlignment.center,  children: [ /* … */ ],),
```

```
Padding(  padding: .all(8.0),  child: Text('Hello world'),),
```

* **构建钩子：**现已稳定！您可以直接将原生代码或原生资源打包到 Dart 包中。更多详情请访问钩子页面(https://dart.dev/tools/hooks)或观看构建钩子(https://www.youtube.com/watch?v=AxNF5dj8HWQ)视频。
* **还有更多！**新增了分析器插件系统(https://github.com/dart-lang/sdk/blob/main/pkg/analysis\_server\_plugin/doc/writing\_a\_plugin.md)。您可以使用此系统编写自定义分析规则，并利用 IDE 快速修复功能。使用新增的“已弃用”注解(https://api.dart.dev/stable/dart-core/Deprecated-class.html)可以弃用特定功能。

#### Flutter 3.38 的亮点：更完善的框架、更好的平台集成和更流畅的开发者工具。

* **网页功能增强：**我们提供了配置文件(https://docs.flutter.dev/platform-integration/web/web-dev-config-file)`flutter run`、代理支持和网页热重载(https://docs.flutter.dev/platform-integration/web/building#hot-reload-web)功能。因为谁有时间等待呢？
* **框架和用户界面：**

  `OverlayPortal`比以往任何时候都更加强大，预测性返回(https://developer.android.com/guide/navigation/predictive-back-gesture)手势现在已成为 Android 的默认功能。我们也在继续完善 Material Design 和 Cupertino 主题。
* **iOS 更新：全面支持 iOS 26/Xcode 26、命令行部署，以及**迁移 Flutter 应用(https://docs.flutter.dev/release/breaking-changes/uiscenedelegate#migration-guide-for-flutter-apps)的极其重要的指南。
* #### UIScene 生命周期迁移

  Flutter 3.38 包含了对苹果强制要求的UIScene 生命周期（https://developer.apple.com/documentation/technotes/tn3187-migrating-to-the-uikit-scene-based-life-cycle）的重要支持。这是继苹果在 WWDC25 大会上宣布“在 iOS 26 之后的版本中，任何使用最新 SDK 构建的 UIKit 应用都必须使用 UIScene 生命周期，否则将无法启动”之后的一项关键且积极的更新。

  为确保您的 iOS Flutter 应用程序在未来的 iOS 版本上保持兼容性并成功启动，需要进行迁移。

  **迁移 Flutter 应用程序**

  所有现有的 iOS Flutter 应用都必须迁移到新的生命周期（苍天呀，😭😭😭😭😭😭）。您可以通过两种方式完成此迁移：

  ```
  flutter config --enable-uiscene-migration
  ```

* **迁移 Flutter 插件**

  依赖应用程序生命周期事件的 Flutter 插件必须更新为使用 UIScene 生命周期事件。插件开发者应参考迁移指南（https://docs.flutter.dev/release/breaking-changes/uiscenedelegate#migration-guide-for-flutter-plugins）。未迁移的插件将在未来的版本中显示警告。

  **迁移嵌入式 Flutter（可选）**

  对于将 Flutter 嵌入原生宿主应用程序的项目，迁移是可选的，但强烈建议进行。使用“添加到应用程序迁移指南（https://docs.flutter.dev/release/breaking-changes/uiscenedelegate#migration-guide-for-adding-flutter-to-existing-app-add-to-app）”采用 Flutter 新的 UIScene API ，可以为您的插件启用场景生命周期事件，从而确保与 Flutter 生态系统的兼容性。

1. 手动迁移：请按照 Flutter网站（https://docs.flutter.dev/release/breaking-changes/uiscenedelegate#migration-guide-for-flutter-apps）上提供的手动迁移说明进行操作。自动迁移（实验性功能）：启用此实验性功能可自动处理迁移。此功能将在未来的版本中默认启用。运行以下命令：

* **Android 更新：** NDK r28 已发布，以兼容 16KB(https://developer.android.com/guide/practices/page-sizes) 页面大小；修复了一个主要的内存泄漏；并更新到 Java 17。
* #### 16KB 页面大小兼容性

  升级到 Flutter 3.38 是满足Google Play 16 KB （https://developer.android.com/guide/practices/page-sizes）页面大小兼容性要求的必要准备。从2025 年 11 月 1 日起，面向 Android 15 及更高版本的应用必须支持 16 KB 页面。此项变更可确保您的应用在高内存设备上正常运行，并带来性能提升，例如启动速度提升高达 30%。Flutter 3.38 将默认的 Android ndkVersion 更新为 NDK r28，这是原生代码实现 16 KB 页面大小支持的最低要求。

  #### 内存修复

  Flutter 3.38（https://github.com/flutter/flutter/issues/173770）修复了一个影响所有 Android 平台 Flutter 应用的严重内存泄漏问题。该问题（在 3.29.0 版本中引入）发生在 Activity 在退出时被销毁的情况下，无论是开发者设置中配置的销毁方式，还是由于内存不足而被系统强制终止的 Activity。

  #### Android 依赖项更新

  为您的应用找到合适的 Android 依赖项版本组合通常是一项挑战，这些依赖项包括 Gradle、Android Gradle 插件 (AGP)、Kotlin Gradle 插件 (KGP)、Java 等。对于 Flutter 3.38 版本，我们在持续集成 (CI) 环境中测试并确认了以下 Android 依赖项版本组合的兼容性：

  为了确保您的应用能够在不同的 Flutter 版本之间无缝运行，我们强烈建议您在构建文件中使用 Flutter SDK 提供的 API 级别变量。此版本配置的值如下：

+ `flutter.compileSdkVersion`（API 36）
+ `flutter.targetSdkVersion`（API 36）
+ `flutter.minSdkVersion`（API 24）或更高

+ **Java 17**：Flutter 3.38 中 Android 开发的最低版本要求。
+ **KGP 2.2.20**：该工具支持的最高Kotlin Gradle 插件版本。https://kotlinlang.org/docs/gradle-configure-project.html#apply-the-plugin
+ **AGP 8.11.1**：最新的 Android Gradle 插件版本，与 KGP 2.2.20兼容-https://kotlinlang.org/docs/gradle-configure-project.html#apply-the-plugin。
+ **Gradle 8.14**：此版本与所选的 Java、KGP 和 AGP 版本兼容。请注意，AGP 8.11.1 的最低版本要求为 Gradle 8.13。

* #### Web 开发配置文件

  该`flutter run`命令现在支持 Web 设置配置文件。您可以在`web_dev_config.yaml`项目根目录的文件中指定主机、端口、证书和标头信息。将该文件检入，以便团队中的每个人都能使用相同的设置进行调试。有关更多信息，请访问“设置 Web 开发配置文件-https://docs.flutter.dev/platform-integration/web/web-dev-config-file”。

  #### Web 开发代理设置

  除了现有的命令行参数外，Web 开发配置文件还支持新的代理设置。代理设置允许将对已配置路径的请求转发到另一台服务器。这使得开发能够连接到同一主机上动态端点的 Web 客户端变得更加容易。

  有关代理设置的详细信息，请参阅设置 Web 开发配置文件https://docs.flutter.dev/platform-integration/web/web-dev-config-file。

  #### 扩展了对网页热重载的支持

  现在，当您在浏览器中打开 Flutter 应用的链接并运行该应用时，状态热重载功能默认启用`-d web-server`。即使同时连接多个浏览器，此功能也能正常工作。

  与之前一样`-d chrome`，此功能可以使用标志暂时禁用`--no-web-experimental-hot-reload`。禁用此功能的功能将在未来的版本中移除，因此，如果您在开发流程中遇到问题，请使用 Dart 的Web 热重载问题模板（https://github.com/dart-lang/sdk/issues/new?template=5\_web\_hot\_reload.yml）提交错误报告。更多信息，请参阅Web 热重载文档（https://docs.flutter.dev/platform-integration/web/building#hot-reload-web）。

  ### 引擎

  #### 性能叠加层

  性能叠加层已重构，效率更高，在 Skia 和 Impeller 后端上的渲染时间均有所缩短。这意味着您可以以更少的开销获得更准确的性能数据。（#176364-https://github.com/flutter/flutter/pull/176364）

  #### **Vulkan 和 OpenGL ES**

  对 Vulkan 和 OpenGL ES 后端的诸多修复和改进提高了在更广泛设备上的稳定性和性能。这包括更好地处理管线缓存（#176322-https://github.com/flutter/flutter/pull/176322）、栅栏等待器（#173085-https://github.com/flutter/flutter/pull/173085）和图像布局过渡（#173884-https://github.com/flutter/flutter/pull/173884）。

  #### **渲染器统一**

  CanvasKit 和 Skwasm 渲染器的统一工作仍在继续。本次版本更新包含大量重构，以在两者之间共享更多代码，这将带来更一致的用户体验，并加快未来的开发速度（#174588-https://github.com/flutter/flutter/pull/174588）。

  #### 线程合并

  iOS 和 Android 系统中已移除选择退出线程合并的功能。欲了解更多信息，请观看精彩的线程合并视频-https://www.youtube.com/watch?v=miW7vCmQwnw。
* **工具：实验性**组件预览工具(https://docs.flutter.dev/tools/widget-previewer)和集成开发环境 (IDE)已进行重大更新。快来体验吧，感觉和SwiftUI很像了😄！
* **无障碍功能：**FLutter团队一直在努力让 Flutter 更具包容性。欢迎体验全新的`SliverSemantics(  
  https://api.flutter.dev/flutter/widgets/SliverSemantics-class.html)`组件和更完善的默认行为。

  ### 社区和生态系统聚焦

  在这个版本周期中，Flutter社区贡献了许多宝贵的内容。以下仅举几例：

  此版本重点介绍Kilian(https://github.com/schultek)和他的框架Jaspr(https://jaspr.site/)。Flutter的 Web 支持非常适合应用程序开发，而 Jaspr 则是一个基于 Dart 构建的传统 DOM（HTML/CSS）Web 框架。如果您需要一个基于 Dart 的网站解决方案，Jaspr 将是 Flutter Web 的绝佳补充。Flutter团队以至于将所有的文档基础架构——包括 dart.dev 和 docs.flutter.dev——都迁移到了这个平台上。强烈建议你们亲自体验一下(https://docs.jaspr.site/get\_started/quick\_start)！

* 还在等什么？喜欢尝鲜的小伙伴可以升级到 Flutter 3.38 和 Dart 3.10，去官网体验下！

+ Flutter 3.38 版本说明（移步官网）-https://docs.flutter.dev/release/release-notes
+ Dart 3.10 版本说明-https://github.com/dart-lang/sdk/blob/main/CHANGELOG.md
+ 深入探索人工智能应用的未来-https://www.youtube.com/playlist?list=PLjxrf2q8roU3ahJVrSgAnPjzkpGmL9Czl
+ 在Flutter AI Playground中体验-http://g.co/firebase/flutter-ai-playground
+ 加入flutter.dev/community-http://flutter.dev/community

* **Windows 显示属性：(https://github.com/9AZX)**现在您可以在 Windows 上获取详细的显示信息，包括显示器列表、显示尺寸、刷新率和 DPI。
* **新增便捷构造函数：**

  添加了一个`SliverGrid.list(https://github.com/ahmeddhus)`构造函数，为从组件列表创建网格提供了更简洁的 API。
* **增强手势处理：(https://github.com/houssemeddinefadhli81)小部件引入了一个**`onLongPressUp`回调`InkWell`，专门用于处理长按手势的释放。
* **更灵活的组件：(https://github.com/iamtoricool)**

  `maxCount`向构造函数添加了一个参数`Badge.count`，(https://github.com/rkishan516)向构造函数添加了一个瞬时变体`CupertinoSlidingSegmentedControl`。
* **重要修复：还有https://github.com/manu-sncf**和https://github.com/yiiim针对滚动行为的关键修复， https://github.com/romaingyh针对焦点问题的关键修复等等。
* 其他的重大变化请移步-https://docs.flutter.dev/release/breaking-changes