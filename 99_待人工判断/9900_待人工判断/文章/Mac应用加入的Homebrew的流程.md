---
title: Mac应用加入的Homebrew的流程
author: 我有一计
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg2ODI0MTg1Mw==&mid=2247490500&idx=1&sn=510139930b3a8cf61324929309ae555e&chksm=cf991f6e8c68ee99a5176c2005602e82cb4af3934b1401367c4c3fbf5df0953f6dca5f645fc5&mpshare=1&scene=24&srcid=1103cbeFNDCArMrsomvZEBJI&sharer_shareinfo=ba3b90dc1b01814fa23b313159adf00f&sharer_shareinfo_first=ba3b90dc1b01814fa23b313159adf00f#rd
---

# 引言

这两天解决了 FreeTex 一个挂了几个月的 issue，那就是 Mac 版本的适配。

之前的 Mac 版本尝试用 CI 的方式去自动化构建，但一直存在兼容性问题。

现在该问题已得到解决，可以在 FreeTex 仓库中，找到最新版本的dmg下载链接。

仓库地址：https://github.com/zstar1003/FreeTex

该版本优先采用 mps 作为推理设备，在M系列的Mac上，推理速度有了大幅提升。

当然，本文的重点是更进一步，试图把安装包发布到 Homebrew 上，这样进一步方便用户安装。

# Homebrew 简介

Homebrew 是 macOS 上最流行的第三方包管理器，它不是 Apple 官方工具，而是开源社区项目。

Homebrew 本身不提供应用托管，它只是一个应用索引，方便用户直接可以通过命令行`brew install` 的方式去安装应用。

# Homebrew 接入步骤

下面是应用并入 Homebrew 的详细流程。

## 1. 创建dmg文件

macOS 上的应用通常是`.app`结构，为了方便应用分发，需要先将app打包成dmg。

这一步可以用 mac本身自带的`hdiutil`，也可以用第三方工具`create-dmg`，后者可以对DMG 窗口大小、位置、背景图进行修改，灵活性更高。

## 2. 托管dmg文件

由于 Homebrew 本身不提供文件托管，所以需要把dmg镜像托管到自己的服务器或者是github仓库，这里我上传到了github的Assets里面。

## 3. 填写 Homebrew 模板

Homebrew 有两套“安装脚本”体系：

| 类型 | 名字 | 用途 | 举例 |
| --- | --- | --- | --- |
| 命令行软件 | **Formula** （配方） | 编译/安装 CLI 工具 | `brew install git` |
| 图形软件/打包文件 | **Cask** （酒桶） | 安装 .app / .dmg / .pkg | `brew install --cask google-chrome` |

对于.dmg 文件，需要填写 Cask 模板，用下面的命令进行模板生成：

```
brew create --cask "https://github.com/zstar1003/FreeTex/releases/download/v1.0.0/FreeTex-Installer-1.0.0.dmg" --set-name FreeTex
```

这里的`cask`是下载链接，`set-name`是应用名称，参数不加也可以，只是加了之后会自动填到模板里。

这一步需要一定时间等待，因为第一次创建，它会把 Homebrew 整个仓库拷贝到`/opt/homebrew`路径下，相当于本地模拟了一个 Homebrew，方便下一步的测试。

等待完成后，会生成一个 Cask 模板，下面是我填完的信息的模板内容：

```
cask "freetex" do  
  version "1.0.0"  
  sha256 "20f784bbfee37ba75d6a7a40275ec09babf064a73e62f30dac6165db4d2d597b"  
  
  url "https://github.com/zstar1003/FreeTex/releases/download/v#{version}/FreeTex-Installer-#{version}.dmg"  
  name "FreeTex"  
  desc "Free intelligent formula recognition software"  
  homepage "https://xdxsb.top/FreeTex"  
  
  livecheck do  
    url :url  
    strategy :github_latest  
  end  
  
  depends_on macos: ">= :big_sur"  
  
  app "FreeTex.app"  
  
  zap trash: [  
    "~/Library/Application Support/FreeTex",  
    "~/Library/Caches/FreeTex",  
    "~/Library/Preferences/com.freetex.app.plist",  
    "~/Library/Saved Application State/com.freetex.app.savedState",  
  ]  
end
```

模板中，各参数的作用如下表所示：

| 项目 | 作用 |
| --- | --- |
| `cask` | 定义 GUI 软件安装脚本 |
| `version` | 版本 |
| `sha256` | 文件完整性校验 |
| `url` | 下载地址 |
| `name` | 应用名称 |
| `desc` | 描述 |
| `homepage` | 官网 |
| `livecheck` | 自动检查更新 |
| `depends_on` | 系统版本要求 |
| `app` | 指定要安装的 `.app` |
| `zap` | 卸载清理文件 |

## 4. 本地测试

下面进行本地测试，本地测试通过才能去合并到官方仓库。

首先临时设置环境变量：

```
export HOMEBREW_NO_AUTO_UPDATE=1  
export HOMEBREW_NO_INSTALL_FROM_API=1
```

这样是阻止 Homebrew 进行自动更新，因为在上一步已经把 Homebrew 整个库内容拷在了本地。

安装测试，这一步会自动根据 cask 模板文件去配置的地址进行下载，以确保下载地址没问题。

```
brew install --cask ./freetex.rb
```

卸载测试，这一步会将刚刚安装好的应用进行卸载，以确保模板中的卸载逻辑能正常运行。

```
brew uninstall freetex
```

规范性审核，这一步会进行模板格式的规范性审核，比如文件最后一行应该有空格，描述中不要有标点等之类的内容，它会自动审核并修复。

```
brew audit --new --cask freetex  
brew style --fix freetex
```

## 5. 合并到官方库

本地测试没问题之后，就可以去合并到官方仓库了。

仓库地址：https://github.com/Homebrew/homebrew-cask

首先fork该仓库，然后将自己的cask文件放进去，比如这个应用是以f开头，就放到`Casks/f`路径下。

放置完之后，提交PR，名称以`应用名 版本号`进行命名。

## 6. PR审核结果

PR 会进行两道审核：

第一道审核是系统自动检查该模板的语法规范、连通性等各种问题，测试不完全通过也是没关系的，比如dmg只适配了arm平台，对于intel平台的测试就是过不了。

第二道审核是人工审核，维护者速度很快，半天内就给了反馈：

这个PR没有被合并，因为打包时，我没有对这个应用进行签名，而 macOS 对 ARM 平台的要求是应用必须要有签名。

这个回复不仅速度快而且内容很具体，这让我想起了早年在国内酷安平台发布App的经历：等了数天，最后被拒，没有任何理由说明。这回应态度，高下立判。

# Homebrew 自建Tap

Homebrew 维护者还提到了另一种合并方式，他说，你可以自建一个Tap，这样他们很乐意去主动进行合并。

自建Tap，指的就是自己建一个 Homebrew 软件仓库，这样用户可以不走官方渠道。

下面我建了一个新仓库`homebrew-freetex`，它要求必须以`homebrew`开头命名。

然后把 Cask 模板放进去。

下面就可以用 homebrew 尝试安装：

```
brew tap zstar1003/homebrew-freetex  
brew install --cask freetex
```

这样是可以安装上的，但是打开出现弹窗：

原来 MacOS 对于未签名的app，用 homebrew 安装之后就会有这样的提示。

它明明可以显示“该应用未签名”之类的提示，但是却显示“已损坏”，让人第一眼看上去还以为应用有问题。

为什么 MacOS 这么执着于签名呢，美其名曰是为了用户安全，而实际上，签名需要注册开发者账号，并缴纳年费。

等我缴完“保护费”之后，再分享一下经验。