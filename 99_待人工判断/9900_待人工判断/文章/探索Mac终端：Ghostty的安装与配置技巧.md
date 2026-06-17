---
title: 探索Mac终端：Ghostty的安装与配置技巧
author: 代码麻辣烫
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk0NjY0MTc0NA==&mid=2247499868&idx=1&sn=8c51a39a5fcfd494bc16d30a08350fe5&chksm=c23e4c4f908033bc869717ffaa08c4565311e10ee2a22daed957f5996974293a8a4fb566274d&mpshare=1&scene=24&srcid=04224qheG7FWt4sMdV0yEbxu&sharer_shareinfo=76094da98aa8852f405132d93a80cf84&sharer_shareinfo_first=76094da98aa8852f405132d93a80cf84#rd
---

## Ghostty：Mac用户的新宠终端工具

嘿，朋友们！今天我想和你们聊聊我最近发现的一个超棒的终端工具——Ghostty。如果你像我一样，经常和各种AI工具打交道，比如Codex、Gemini Cli，那你绝对不能错过这个神器。

### 为什么选择Ghostty？

我之前一直用的是Mac自带的终端，但说真的，在AI时代，它有点跟不上节奏了。比如在Claude Code里，想要优雅地换行输入，或者开个新的命令行窗口，都挺费劲的。Ghostty就像是给Mac终端打了一针强心剂，让一切都变得轻松起来。

Ghostty界面

### 安装Ghostty

安装Ghostty超简单，你可以直接在Codex App里找到它。如果安装后发现打不开，别急，去系统设置里调整一下隐私与安全选项，允许它运行就行。

隐私与安全设置

当然，如果你是命令行的忠实粉丝，也可以直接用Homebrew来安装：

```
brew search ghostty  
brew install --cask ghostty
```

或者，你可以直接去官网下载。

Ghostty官网下载

### 个性化配置Ghostty

安装好之后，你可以根据自己的喜好来配置Ghostty。比如设置主题、字体、窗口行为等等。我这里有个配置示例，你可以参考一下：

Ghostty配置界面

```
# Ghostty个性化配置  
theme: light:Catppuccin Latte,dark:Catppuccin Mocha  
background-opacity: 0.88  
font-family: "Maple Mono NF CN"  
# 更多个性化设置...
```

把这段配置复制到Ghostty的配置文件里，然后重启Ghostty，你的新终端就准备好了。

Ghostty配置示例

### 使用Ghostty

Ghostty的快捷键和窗口管理功能超级实用。比如，你可以用Cmd + D在一个标签页里打开两个命令行窗口，或者用Cmd + Shift + D来垂直分割窗口。这些快捷操作让你的工作效率直线上升。

Ghostty窗口管理

别忘了，为了让Ghostty看起来更棒，你可能需要安装一个特别的字体：

```
brew install --cask font-maple-mono-nf-cn
```

### 结语

好了，关于Ghostty的介绍就到这里。如果你正在寻找一个更现代、更强大的终端工具，Ghostty绝对值得一试。别忘了查看快捷键列表，它们能让你的操作更加得心应手。

Ghostty快捷键

希望这篇小分享能帮到你，我们下次再见！