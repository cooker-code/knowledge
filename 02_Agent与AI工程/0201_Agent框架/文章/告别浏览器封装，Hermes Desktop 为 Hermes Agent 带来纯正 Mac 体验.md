---
title: 告别浏览器封装，Hermes Desktop 为 Hermes Agent 带来纯正 Mac 体验
author: 01进化论
date: 
url: https://mp.weixin.qq.com/s?__biz=MzcwNTA0NjUzNg==&mid=2247484305&idx=1&sn=d830ebcf9d99dcffe924ea67713a2f43&chksm=f597837072e8f579489bc3a48fa37034253d3d35401deb66070079d2800e51990c5e4c71e7c2&mpshare=1&scene=24&srcid=0413Hl2sonequIZwhPWZyhFu&sharer_shareinfo=f1ae5abd110ce12f478565a0b3762bb3&sharer_shareinfo_first=f1ae5abd110ce12f478565a0b3762bb3#rd
---

对于已经在使用强大的 Hermes Agent 的 Mac 用户来说，现在有了一款能让您的工作流更加流畅、更具原生体验的工具——Hermes Desktop。它并非一个简单的浏览器封装，而是一款专为 macOS 设计的本地应用程序，旨在将您最关心的工作流程无缝集成到一个窗口中。

Hermes Desktop 的核心设计理念是**克制与专注**。它不试图取代 Hermes 的命令行工作流，而是通过提供一个更快、更沉稳、更原生的 Mac 界面来增强它，同时保持其工作方式的透明性。

### 核心理念

* **直连 SSH**：直接通过 SSH 连接到您的 Hermes 主机，无需额外的网关 API。
* **单一事实来源**：始终将远程的 Hermes 主机作为唯一的数据来源，确保信息的一致性。
* **无本地镜像**：不会在您的 Mac 上镜像或缓存文件，避免了数据同步延迟和状态冲突的问题。
* **无远程依赖**：无需在远程主机上安装任何辅助服务或守护进程。

简单来说，Hermes Desktop 致力于让 Hermes 在 macOS 上的体验“宾至如归”，让您在享受原生应用便利性的同时，依然能感受到其背后强大而真实的工作机制。

## 功能预览

通过以下截图，您可以直观地感受到 Hermes Desktop 的核心界面，涵盖了会话、用量、技能和终端管理。

## 主要功能详解

Hermes Desktop 将 Hermes 的核心要素整合到了一个简洁高效的图形界面中：

* **原生 Mac 应用体验**：告别浏览器封装，享受流畅、快速、符合 macOS 设计规范的应用体验。
* **集成真实 SSH 终端**：内置一个功能完整的 SSH 终端，支持多标签页，让您随时可以在应用内执行命令。
* **核心文件安全编辑**：提供对 Hermes 关键配置文件的冲突感知编辑功能，包括：
  + `~/.hermes/memories/USER.md`
  + `~/.hermes/memories/MEMORY.md`
  + `~/.hermes/SOUL.md`
* **用量统计与分析**：直接从远程的 `~/.hermes/state.db` 读取数据，汇总展示输入/输出 Token 总量，并按模型进行分类统计。
* **技能 (Skills) 浏览器**：递归浏览并展示位于 `~/.hermes/skills/**/SKILL.md` 路径下的所有技能文件。
* **会话 (Session) 管理**：从远程 `~/.hermes/state.db` 数据库中读取、搜索和删除会话记录，操作实时同步。
* **广泛的兼容性**：只要目标设备上运行了 Hermes 并支持 SSH 访问，Hermes Desktop 就能连接。这包括：
  + 树莓派 (Raspberry Pi)
  + 另一台 Mac
  + VPS 或其他远程服务器
  + 通过 `ssh localhost` 连接到本机

## 开始使用

整个安装和配置过程非常简单，只需几分钟即可完成。

### 第一步：环境准备

在下载之前，请确保满足以下条件：

* **系统要求**：一台运行 macOS 14 或更新版本的 Mac。
* **SSH 访问**：您已经可以在 Mac 的终端中通过 SSH 无交互（即无需输入密码或确认主机密钥）地连接到 Hermes 主机。
* **主机密钥**：您至少已经在终端中接受过一次目标主机的 SSH 密钥。
* **网络路由**：您的 Mac 与 Hermes 主机之间有正常的网络连接，例如局域网、公网 IP、VPN 或 Tailscale。
* **远程环境**：Hermes 主机上已安装 `python3`。
* **数据路径**：Hermes 的数据位于远程用户的 `~/.hermes` 目录下。

**黄金法则**：如果以下命令可以在您的 Mac 终端中顺利执行，且不要求输入密码或确认主机密钥，那么 Hermes Desktop 也基本上准备就绪了。

```
ssh your-host
```

### 第二步：安装应用

1. 从项目的 GitHub Releases 页面下载 `HermesDesktop.app.zip`。
2. 双击解压 zip 文件。
3. 将 `HermesDesktop.app` 拖入您的 `Applications` (应用程序) 文件夹。
4. 打开应用。

**注意**：当前版本是为 Intel 和 Apple Silicon Mac 构建的通用应用，但尚未经过苹果公证。因此，首次启动时，macOS 可能会弹出安全警告。这是正常现象。如果遇到阻拦，请按以下步骤操作：

1. 在弹窗中点击 `取消`，不要选择 `移到废纸篓`。
2. 右键点击 `HermesDesktop.app` 图标，选择 `打开`。
3. 如有需要，前往 `系统设置` -> `隐私与安全性`，点击 `仍要打开`。

### 第三步：连接到您的 Hermes 主机

打开应用后，进入 `Connections` (连接) 标签页，创建一个新的连接配置。您有两种方式进行配置：

#### 方式一：使用 SSH 别名 (推荐)

这是最简单的方式。如果您在 Mac 的 `~/.ssh/config` 文件中配置了别名，例如：

```
Host hermes-home
 HostName vps.example.com
 User alex
```

那么在应用中，只需：

* 将 `SSH alias` 设置为 `hermes-home`。
* `Host`, `User`, `Port` 字段留空。

#### 方式二：直接填写主机信息

如果您通常使用 `ssh alex@vps.example.com` 这样的命令连接，那么在应用中：

* `Host or IP`: `vps.example.com`
* `User`: `alex`
* `Port`: `22` (或您的自定义 SSH 端口)

配置完成后，点击 `Test` (测试) 按钮进行连接预检。测试通过后，点击 `Use Host` (使用主机) 即可开始使用。`Test` 按钮会检查 SSH 目标是否可达、认证能否无交互通过以及远程环境中 `python3` 是否可用。

## 总结

Hermes Desktop 巧妙地平衡了原生体验与忠于本源之间的关系。它并非要隐藏 Hermes 的命令行本质，而是为其提供了一个优雅、高效的 macOS 图形化伴侣。如果您是一位在 Mac 上深度使用 Hermes 的开发者，这款应用无疑将极大提升您的工作效率和愉悦感。