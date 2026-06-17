---
title: MiniSim：MacOS 模拟器管理神器，一键启动 iOS/Android 模拟器
author: AI开源提效指南
date: AI开源提效指南AI开源提效指南
url: https://mp.weixin.qq.com/s?__biz=MzY5NzIxODM2MQ==&mid=2247484816&idx=1&sn=b0c35b6a2df5d9fa524eb6e8984086e9&chksm=f542e5308ca97661df988b79a9ba323977194ab09855e1c7e8a59bdbc5158086aaa9604e61b7&mpshare=1&scene=24&srcid=0526eTodDSu2e1p2shalfuOV&sharer_shareinfo=0bb691d3c28278ca1466bb20f568ef20&sharer_shareinfo_first=0bb691d3c28278ca1466bb20f568ef20#rd
---

大家好！这里是 AI开源提效指南！

今天给大家安利一款专为 Mac 用户打造的开源效率神器 —— **MiniSim**。

如果做过移动端的朋友应该知道，每次为了启动一个模拟器，是不是得先痛苦地等待 Xcode 或 Android Studio 慢吞吞加载完整个 IDE ?

MiniSim 是一款采用 Swift 和 AppKit 纯原生编写的 macOS 菜单栏小工具。它小巧、精致、几乎不占资源，却能化繁为简，让你在一个菜单里同时管理 iOS  与 Android 🤖 的所有模拟器。

| 特性 | 详情 |
| --- | --- |
| **开发语言** | Swift (100% 原生，流畅不卡顿) |
| **支持平台** | macOS |
| **开源协议** | MIT |
| **安装体积** | 极为轻量，内存占用微乎其微 |
| **获取方式** | Homebrew / GitHub Releases |

## 😫 痛点场景：你是否每天都在重复这些无效等待？

作为双端或跨平台（Flutter / React Native / Uni-app）移动开发者，这些场景你一定深有体会：

* ❌ **Xcode 启动太折腾：** 只想测个 iOS 样式，必须先打开 Xcode  顶栏选择设备  漫长等待，每次至少耗费 2 分钟。
* ❌ **Android Studio 同样抓狂：** 同样的流程在 Device Manager 里再熬一遍，双倍的等待，双倍的煎熬。
* ❌ **磁盘空间被吃光：** 想清理一些过期的、不用的模拟器，必须套娃进到 IDE 的多层设置深处去手动删除。
* ❌ **调试信息不好拿：** 想要个 UDID 或者 ADB ID，还要在终端里敲复杂的命令或者去 IDE 里翻找复制。

MiniSim 就是为了终结这些体验而生的。一个图标，秒开一切。

这个项目还在一直更新，对新版本也是支持的！

## ✨ 核心功能亮点

### 📱 iOS 模拟器管理

* **一键秒级启动：** 菜单栏直接点击任意 iOS 设备，无需感知 Xcode 的存在。
* **沙盒定位 (Show on disk)：** 右键即可**在 Finder 中直接打开该模拟器的实际磁盘路径**，方便清理缓存或调试数据。
* **调试信息复制：** 快速复制设备 UDID（真机测试/配置 Provisioning Profile 必备）或名称。

### 🤖 Android 模拟器管理

* **多模式启动：** 除了正常开启，还支持**冷启动（Cold Boot）**，直接解决模拟器卡死、系统报错等疑难杂症。
* **无音频启动 (No Audio)：** 🎧 **蓝牙耳机用户的神级福音！** 彻底解决由于模拟器占用音频通道导致 Mac 蓝牙耳机突然断连或音质降级的恶心 Bug。
* **无障碍集成 (a11y)：** 一键切换无障碍模式，方便测试应用的无障碍兼容性。
* **清理与删除：** 支持直接在列表中右键删除不再使用的 Android 虚拟设备（AVD），释放宝贵的 SSD 空间。

### 🔧 进阶自动化

* **自定义命令 (Custom Commands)：** 官方预留了强大的自定义扩展能力。你可以在设置中编写终端 Shell 脚本，MiniSim 提供了 、adbId 等动态变量。你可以用它做出“一键给指定模拟器装包”、“一键运行特定测试”等自动化工作流！
* **全局快捷键：** 默认通过 ⌥ Option + ⇧ Shift + e 即可随时随地在任何界面呼出设备菜单，全程无需鼠标。

## 📥 安装指南（超简单！）

### 方式一：Homebrew（强烈推荐）

打开你的终端，一行命令直接搞定：

```
brew install --cask minisim
```

### 方式二：手动下载

前往 GitHub 官方 Releases 页面 下载最新的 .zip 文件，拖入 Applications 文件夹即可。

> 💡 **小贴士：** 首次启动 MiniSim 时，它会贴心地弹出一个\*\*引导向导 (Setup Wizard)\*\*，自动去检测你 Mac 上的 Xcode 路径和 Android SDK 路径，你只需要一路点击确认，几乎实现零配置上手！

## 🎯 使用教程

### 基本使用

1. 启动 MiniSim（菜单栏出现图标）
2. 点击菜单栏图标，弹出设备列表
3. 选择要启动的模拟器，秒开！
4. 右键查看更多操作（复制 UDID、删除等）

### Raycast 集成

如果你使用 Raycast，可以直接安装 MiniSim 扩展：

* 安装 minisim Raycast Extension
* 在 Raycast 中搜索 `minisim` 即可快速操作。

### 全局快捷键

默认快捷键：`⌥ Option + ⇧ Shift + e`

随时呼出模拟器菜单，不用操作鼠标！

## 🎯 效率进阶：Raycast 深度集成

如果你是 Mac 效率神器 Raycast 的重度用户，MiniSim 简直是如虎添翼：

1. 你可以直接在浏览器中访问 MiniSim Raycast Extension。
2. 或者在支持富文本的平台直接点击这个一键安装命令：点击直接安装到 Raycast。
3. 安装后，不用抬起手去找菜单栏，直接唤醒 Raycast 输入 minisim 加上模拟器名字，回车即可启动！

---

## 🔍 技术底座

MiniSim 为什么能做到这么快、这么轻？因为它本身不包含任何笨重的模拟器核心，它只是一个优雅的系统命令行包装器（Wrapper）：

* **iOS 端：** 底层直接调用苹果官方的 xcrun simctl 工具。
* **Android 端：** 底层直接调用 Android SDK 目录下的 /tools/emulator。

> ⚠️ **前置条件：** 请确保你的 Mac 上已经正确安装了 Xcode 和 Android Studio/Android SDK，MiniSim 才能正确读取到你的设备列表。

## 📚 相关资源链接

```
GitHub 官方仓库: https://github.com/okwasniewski/MiniSim  
官方网站: https://www.minisim.app  
进阶指南: https://github.com/okwasniewski/MiniSim/blob/main/docs/custom_commands.md
```

如果你是一名每天和移动端开发打交道的工程师，赶快去试试 MiniSim 吧，这省下来的启动时间，多喝杯咖啡它不香吗？

免责声明：本文内容仅供学习交流，所述工具/方法请遵守相关平台服务条款及法律法规。如涉及第三方服务，请以官方最新政策为准。

---

**🎯****觉得这份工具干货有用？希望收到您的支持：**

* ⭐ 星标 / 置顶公众号，**第一时间解锁最新工具分享！**
* ✅ **点赞**「**推荐**」，让更多技术伙伴发现优质干货！
* 🔗 **转发**给团队小伙伴，一起高效提效！
* 💬 **底部留言区**，告诉我您想找的工具/项目方向！

**📬 长期追踪优质开源工具**

* 关注「**AI 开源提效指南**」｜日更开源神器，玩转技术提效！
* 回复 **【容器加速器】**，即刻开启你的高效探索之旅～