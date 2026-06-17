---
title: [1125] Homebrew 5.0.0：迈向更快速、更现代的未来
author: oh my x
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzk4MTc0Ng==&mid=2247492640&idx=2&sn=0d861a47663296ead17fca80a9f28318&chksm=c1bfff397252e6305bc653a0b4ac45d777428713b9af9c04db37271c7014134a117e567bff61&mpshare=1&scene=24&srcid=1125Ui9rFGvEU250bZ2bESWT&sharer_shareinfo=828ab1f0eed322896697b4d830d1b2dd&sharer_shareinfo_first=828ab1f0eed322896697b4d830d1b2dd#rd
---

TLDR：

Homebrew 5.0.0 版本正式发布。本次更新是 Homebrew 发展史上的一个重要里程碑，核心变化包括：

1. **性能飞跃：**

   默认开启并行下载（`HOMEBREW_DOWNLOAD_CONCURRENCY=auto`），显著提升安装速度。
2. **平台升级：**

   Linux ARM64/AArch64 正式升级为 Tier 1 官方支持平台。
3. **Intel 平台日落：**

   明确了 macOS Intel 平台的淘汰时间表，将在 **2027 年 9 月**彻底移除支持。
4. **安全加强：**

   弃用了绕过 macOS Gatekeeper 安全机制的选项（如 `--no-quarantine`），并要求 Casks 必须进行代码签名。
5. **新功能：**

   `brew bundle`新增对 Go 语言包的支持；`brew info --sizes` 可显示包大小。

## Homebrew 5.0.0

知名 macOS 和 Linux 软件包管理器 Homebrew 近日宣布发布 5.0.0 版本。此次更新由核心维护者 MikeMcQuaid 宣布，带来了多项重大改进、平台支持升级以及对旧有架构的明确淘汰计划。

### 1. 性能革命：默认开启并行下载

在 5.0.0 版本中，Homebrew 最大的用户体验改进莫过于默认开启了下载并发功能（`HOMEBREW_DOWNLOAD_CONCURRENCY=auto`）。这意味着用户在安装多个软件包时，下载过程将自动并行进行，极大地缩短了等待时间。

虽然这一功能在 4.6.0 版本中已存在，但现在它成为了默认设置，并新增了并发下载的进度报告。如果用户遇到兼容性问题，可以通过设置 `HOMEBREW_DOWNLOAD_CONCURRENCY=1` 来禁用此功能。

### 2. 拥抱新架构：Linux ARM64 晋升 Tier 1

随着 ARM 架构在服务器和桌面领域的普及，Homebrew 正式将 **Linux ARM64/AArch64** 提升至 **Tier 1 官方支持级别**。这意味着 Homebrew 将为该平台提供最高级别的维护和支持，确保用户在如树莓派或基于 ARM 的云服务器上获得稳定可靠的体验。

此外，针对 Linux 平台，Homebrew 5.0.0 还改进了 Casks 的支持和可用性，并在 ARM64 Linux 上默认启用了 `CGO_ENABLED`。

### 3. 平台过渡：macOS Intel 淘汰时间表公布

Homebrew 明确了对旧版 macOS 和 Intel 架构的淘汰计划，以集中资源支持 Apple Silicon 和最新的 macOS 版本（包括官方支持 macOS 26 Tahoe）。

**关键时间节点：**

* **2026 年 9 月（或更晚）：**

  Homebrew 将不再支持 macOS Catalina 10.15 及更早版本。macOS Intel x86\_64 平台将降至 Tier 3，届时将停止 CI 支持，不再构建新的二进制包（bottles）。
* **2027 年 9 月（或更晚）：**

  Homebrew 将彻底停止在 macOS Big Sur 11 上的运行，并**完全移除对 Intel x86\_64 架构的支持**。

这一时间表标志着 Homebrew 正式进入全面拥抱 Apple Silicon 的时代。

### 4. 安全与现代化：弃用 Gatekeeper 绕过

为了与 macOS 的安全策略保持一致，Homebrew 5.0.0 弃用了允许用户绕过 macOS Gatekeeper 安全检查的标志，例如 `--no-quarantine` 和 `--quarantine`。

同时，Homebrew 宣布，**没有代码签名的 Casks 将被弃用**，并计划在 2026 年 9 月禁用所有无法通过 Gatekeeper 检查的 Casks。

在内部 API 方面，用户现在可以通过设置 `HOMEBREW_USE_INTERNAL_API` 来选择使用新的、更精简的 Homebrew JSON API，该 API 预计将在未来版本中成为默认设置。

### 5. 开发者与用户体验改进

新版本还带来了许多实用的功能和改进：

* **Go 包支持：**

  `brew bundle` 现在支持在 `Brewfile` 中安装 Go 语言包。
* **包大小查询：**

  新增 `brew info --sizes` 命令，可以显示每个 Formula 和 Cask 的大小。
* **搜索扩展：**

  `brew search --alpine` 现在可以搜索 Alpine Linux 软件包。
* **审核工具：**

  `brew audit` 新增对 Codeberg 仓库的在线检查支持。
* **社区政策：**

  Homebrew 放宽了接受新软件的标准，以鼓励更多项目加入。

**来源：**  
https://brew.sh/2025/11/12/homebrew-5.0.0/

---

---

🎉 **欢迎****加入我们的用户交流群** 🎉

在这里，你可以：  
📢 第一时间了解最新动态和活动信息  
🤝 结识同行，共享实战经验  
📚 探讨行业趋势，拓展人脉圈子

📌 **如何加入？**📷 扫描下方二维码，立即进群！

期待你的加入，一起交流成长！🚀