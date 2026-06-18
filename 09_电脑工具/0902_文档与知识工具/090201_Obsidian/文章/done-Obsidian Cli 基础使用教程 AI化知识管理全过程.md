---
title: Obsidian Cli 基础使用教程 AI化知识管理全过程
author: Shiki实验室 用好工具延长生命
date:
url: https://mp.weixin.qq.com/s?__biz=MzU4MTQ4ODgyNg==&mid=2247497059&idx=1&sn=2a04e27bfcbff7d1ea77297edfe3a430&chksm=fc4fd455e21dddeb4c126da88def8c91261b7952c702f6abed47b9643a72d7c9ddda76f4bff7&mpshare=1&scene=24&srcid=0423F4eIcyKZk1OwSc3wvvfq&sharer_shareinfo=4e3c242823ed99726968fbb45fffa194&sharer_shareinfo_first=4e3c242823ed99726968fbb45fffa194#rd
---
> 已吸收至：[[09_电脑工具/0902_文档与知识工具/090201_Obsidian/090201_核心知识点/Obsidian知识工作流与AI插件边界|Obsidian 知识工作流与 AI 插件边界]]


> 字数 1502，阅读大约需 8 分钟

无论你用什么笔记软件，大概都经历过这样的时刻——

刚开始觉得这个工具太好用了，每天都在记。但写到第二年，库里已经几百篇笔记，旧的没标签、没分类、没属性，新的堆在收件箱里没归置。

想整理，但一想到要手动一条一条改，直接放弃😖。

而且因为笔记数量变多，无论用哪种方式管理，最后都用不起来！

这两个大问题，在 Obsidian 官方推出 CLI 工具、并且可以接入 AI 之后，有了完全不同的解法。

## **Obsidian CLI 是什么？**

很多人用 AI 辅助笔记管理的方式，是开两个窗口：一边Obsidian，一边是 AI 对话框，来回复制粘贴内容让 AI 帮你分析。

这种方式有个根本问题：AI 只是在帮你"想"，它并没有真正进入你的仓库。

Obsidian CLI 解决的就是这个底层硬伤。

Obsidian 官方在 v1.12 版本正式发布了命令行接口（CLI），现在我们能通过命令行实现关于Obsidian的所有操作，包括文件管理、标签、任务、模板、插件、版本历史等等等等。

因为现在的AI工具（Claude Code、Codex、Gemini CLI ）已经彻底渗透进了命令行接口，通他们就可以用自然语言，直接指挥 AI 通过命令行来进行仓库里操作——不是把笔记内容复制给它，是让AI进入你的库动手干活儿。

AI接管笔记的方法，彻底变天了。配置好了CLI工具之后，就可以实现：

### ✅ 快速建立知识管理结构

直接告诉 AI："帮我在仓库里建一套知识管理的文件夹结构。" 它会给你列出几个方案，你选一个，确认后自动创建好文件夹。

接着说："在每个文件夹下生成几个示例笔记。" 读书笔记模板、目标复盘框架、技能学习记录……三分钟之内全部到位。

### ✅ 批量整理旧笔记

如果库里有大量早期笔记没有打标签、没有分类，可以告诉 AI 按规则批量补上。AI 直接在仓库里操作，真正意义上的批量处理。

### ✅ 把常用操作存成指令，反复调用

每天复盘、每周周报、每月归档整理——这些重复性的操作，配置一次之后存成指令，下次直接调用就行，不需要重新描述需求。

## **配置全流程：新手也能跟着走完**

很多人听到"命令行"三个字就退缩了。但实际上，这套工具的配置门槛并没有想象中高。真正麻烦的只是中间可能会遇到一些报错，而这些报错，丢给任何一个免费 AI 基本都能解决。

### 第一步：开启 Obsidian CLI

把 Obsidian 更新到最新版本（v1.12 及以上），进入设置 → 通用 → 命令行界面，点击注册。整个过程是自动的，完成后 Obsidian 就会被注册到系统的 PATH 中，终端里就能使用 `obsidian` 命令了。

当然，我们肉眼是看不到这个注册过程的，但系统是知道的，所以这一步到此为止。

### 第二步：安装 Node.js 和 Claude Code

Claude Code 应该是目前大家都觉得最好用的 AI 命令行工具，所以我们就先安装它。安装它之前，需要先装好 Nodejs

访问官网https://nodejs.org/en/download 下载安装，完成后通过终端运行 `node -v` 和 `npm -v` ，如果出现了安装软件版本号，那么就是安装成功了。

安装好nodejs之后需要在终端输入下面的命令行，来安装claude code

```
npm install -g @anthropic-ai/claude-code
```

如果在国内没有网络，输入这个命令行到终端当中。

```
npm install -g @anthropic-ai/claude-code --registry=https://registry.npmmirror.com
```

安装 Claude Code 的过程中可能会遇到报错，太正常了！我经常和终端大战几百回合。

把错误截图发给任意一个 AI，让它告诉你怎么处理，多试几次不要气馁就好！

成功后用 `claude -v` 检查版本，如果出现了版本号说明安装成功。

### 第三步：替换成国内可用的 AI 模型

Claude Code 默认使用 Anthropic 的 API。一方面是贵、另一方面是国内网络环境容易封号。

更稳定的做法是换成国内的包月模型——比如 MiniMax或者Kimi都有包月的Coding Plan。我自己目前用的MiniMax 的 Coding Plan，29 元/月起，按对话次数计算，还比较划算。

用我的推荐链接购买能打9折！

https://platform.minimaxi.com/subscribe/coding-plan?code=33fXNwIebU&source=link

购买后拿到 API Key，从终端把模型和API key写入JSON文件，就能使用性价比更高的模型了～不怕一觉起来痛失几百万。

接下来我们要创建两个JSON文件并且修改配置，让Claude Code知道使用我们的包月模型，一步一步来操作。

我们先配置第一个settings.json文件。

打开终端，输入下面的指令创建一个设置文件：

```
mkdir -p ~/.claude && touch ~/.claude/settings.json
```

输入回车创建完成之后，不会出现任何新窗口，我们继续输入下面这一个指令来打开这个文件

```
open -a TextEdit ~/.claude/settings.json
```

把文件内容**整体替换**为（记得把 `你的Key` 换成真实的 MiniMax API Key，这里复制的时候一定要把代码做纯文本处理，比如把它复制到记事本再粘贴，也可以让AI他帮你彻底清理格式）：

```
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.minimaxi.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "你的Key",
    "API_TIMEOUT_MS": "3000000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
    "ANTHROPIC_MODEL": "MiniMax-M2.5",
    "ANTHROPIC_SMALL_FAST_MODEL": "MiniMax-M2.5",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "MiniMax-M2.5",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "MiniMax-M2.5",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "MiniMax-M2.5"
  }
}
```

这样第一个JSON文件就修改好了，关闭掉整个终端。我们再来创建和修改第二个.claude.json文件。

还是先输入下面一段指令然后回车，创建第二个JSON文件。

```
touch ~/.claude.json
```

输入之后仍然不会有新窗口打开，直接接着输入下面这一段指令打开刚刚创建的JSON文件

```
open -a TextEdit ~/.claude.json
```

在这个新窗口里加入下面这个字段（括号也要加入哈，但是不要覆盖其他内容）：

```
{
  "hasCompletedOnboarding": true
}
```

改完之后彻底关掉终端再打开，然后在终端输入claude，出现模型名称是MiniMax之后就设置成功了。



当然，出错也是常有的事，把错误截图发给AI，继续调整就好。

 

### 第四步 用Claude Code去安装其他CLI工具

配置好了Claude Code、安装好魔性之后，剩下的就简单了。因为我们可以用它去安装任何其他CLI工具！

直接跟它说话它就会自动安装，所以最重要的是，装好第一个！

## 装好之后，能做哪些事？

安装好之后，我们就可以使用最开始说到的那些功能：

### ✅ 快速建立知识管理结构

### ✅ 批量整理旧笔记

### ✅ 把常用操作存成指令，反复调用

Obsidian 本身已经是市面上逻辑最完整的笔记工具之一。而 CLI + AI 这一层，让它从一个"记录工具"变成了一个可以被自动化驱动的知识系统。

配置一次，受益的是你之后每一天的工作方式。

如果你也听说过Obsidian，但是不知道应该怎么上手，那么我的这本《上头Obsidian》应该非常适合你～

对于零基础的朋友，也可以通过系统课程打好基础，了解Obsidian的底层逻辑，结合现成的仓库和教程，就能在最短时间内建立一套真正高效的知识管理系统。

购买课程赠送纸质版操作手册，方便你随时查看～

感兴趣可以直接拍下，拍下之后别忘了叫我拉你进交流群哦！

另外承接各种知识管理相关咨询，欢迎链接❤️

好工具撬动生产力，生命活出两倍长度

效率提升、系统搭建、出海卖课

欢迎扫码加私人微信链接

我也在这个账号分享自己的出海和副业经历

感兴趣的朋友欢迎围观
