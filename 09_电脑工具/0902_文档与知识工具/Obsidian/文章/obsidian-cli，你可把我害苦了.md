---
title: obsidian-cli，你可把我害苦了
author: 农民伯伯吖
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzODkxMzA2OQ==&mid=2247484269&idx=1&sn=711584e0e03f1e2d385a51be67cc37e7&chksm=c3dfd2906b1300b65fcefc718668faafc42c962b8b2d571dfebb1b523a837df3b99fa1a9a293&mpshare=1&scene=24&srcid=0412cFyYYesSXOnGNeUvM8UP&sharer_shareinfo=870bdfc4a6882efc77e00475fdf8ba65&sharer_shareinfo_first=870bdfc4a6882efc77e00475fdf8ba65#rd
---

# Claudian 插件执行 obsidian property:set 命令返回 127 的问题排查

## 1 起因

这段时间我尝试将 Claude Code 的上下文管理机制引入 Obsidian，打算在 Obsidian 里通过 Claudian 插件搭建了一套 AI 驱动的知识管理工作流（和去年的 Copilot 提示词用法相比，用户体验和 Token 消耗都有明显升级）。

除了我自己设计的几个 skills 之外，我也用到了 kepano 的 obsidian-skill （坦白地说SKILL.md文档写得很粗糙，后来我自己做了一些优化）。  
一切都挺顺利，直到我想让 AI 自动修改笔记属性时，翻车了！

我的命令很简单：

```
obsidian property:set name="status" value="已完成" type=text path="01 记事本/04 精彩瞬间/2026年精彩瞬间.md"  
Exit code 127
```

网上查询之后才知道，127 的意思是 "command not found"。

我的第一反应是——这是不是 Claudian 插件的 bug？

## 2 第一轮排查：终端验证

要判断是不是插件的问题，最直接的方式就是绕过插件，手动在终端里跑同样的命令。

先在 PowerShell 里试：

```
PS C:\Users\农民伯伯> obsidian property:set name="status" value="已完成" type=text path="01 记事本/04 精彩瞬间/2026年精彩瞬间.md"  
Set status: 已完成
```

命令可以被正常执行。再到 cmd 里跑一遍，同样没问题。

同样的命令，终端里正常，Claudian 里 127，我几乎都要以为是Claudian 插件的 bug 了！

## 3 第二轮排查：冒号嫌疑

在一次和 AI 对话过程中，它提到论坛里有一个用户和我有类似的问题[1]：

> 运行带冒号的命令返回127；
>
> 运行不带冒号的命令可以正常被执行。

但对方的情况是因为Obsidian 在应用内更新时没有生成“Obsidian.com”文件导致的，跟我的情况不一样。

后来我在用 Claudian 的时候，AI自己运行了下面这条命令让我排除了冒号的嫌疑。

```
obsidian help property:set  ← 这条命令也有冒号，但在 claudian 里执行成功了
```

既然不是冒号的问题，那会不会是命令中存在中文字符的原因呢？

## 3 第三轮排查：中文字符

马上创建一个全英文的笔记验证一下看看

```
obsidian property:set name="status" value="done" type=text path="00 DASHBOARD/test.md"  
Exit code 127
```

还是 127，看来也不是中文字符的问题。

到这里我已经没有方向了：

* • √ 手动执行没问题
* • √ 冒号没问题
* • √ 中文没问题
* • × 在 Claudian 里带实际参数的 `property:set` 不行

## 4 转机

不死心的我又一次向 AI 咨询了这个问题，这次GPT-5.4建议我检查一下 Claudian 的执行环境，跑几个 `where` 命令看看各个工具的路径。

```
where obsidian    
where powershell    
where pwsh    
where node    
where python    
where jq    
where yq
```

结果返回了以下内容：

```
D:\Program Files\Obsidian\Obsidian.com          ← CLI 工具（正确的）  
D:\Program Files\Obsidian\Obsidian.exe          ← GUI 应用（错误的）
```

Claude Opus-4.6一眼就看出了问题所在——同一个目录下有两个 `obsidian`，一个是 CLI 工具（`.com`），一个是 GUI 应用（`.exe`）。Claudian 很可能调用了错误的那个。

它建议我在 Claudian 里试一下显式指定 `.com`：

```
Obsidian.com property:set name="status" value="done" type=text path="00 DASHBOARD/test.md"  
Set status: done
```

命令执行成功了！

## 5 真相

Obsidian 官方把 CLI 工具命名为 `Obsidian.com`，和 GUI 应用 `Obsidian.exe` 放在同一个目录下。

在 Windows 的 cmd 和 PowerShell 中，系统会按 `PATHEXT` 环境变量的优先级来决定执行哪个文件，顺序是 `.COM` 优先于 `.EXE`。所以我在终端里手动输入 `obsidian` 时，系统正确地选择了 `Obsidian.com`（CLI），一切正常。

但 Claudian 插件底层通过 Node.js 的 `child_process` 执行命令，而 Node.js 在 Windows 上并没有严格遵循 `PATHEXT` 的优先级顺序，导致实际调用了 `Obsidian.exe`（GUI 应用）。GUI 应用根本不认识 `property:set` 这个子命令，于是返回 127。

至于 `help` 命令为什么没问题？因为 GUI 和 CLI 恰好都能响应 `help`，所以一直没暴露这个问题。

这也解释了为什么手动执行和 Claudian 里的表现不一致——**不是命令不对，是同一个名字指向了不同的程序**。

## 6 解决方案

在 obsidian-cli 的SKILL.md中，把所有 `obsidian` 改成 `Obsidian.com`：

```
# 改之前  
obsidian property:set name="status" value="done" file="My Note"  
# 改之后  
obsidian.com property:set name="status" value="done" file="My Note"
```

一字改动，问题消失！

#### 引用链接

`[1]` 问题: *https://forum.obsidian.md/t/cli-windows-two-issues-1-in-app-updater-doesnt-generate-obsidian-com-2-git-bash-resolves-to-exe-instead-of-com/111908*

 

---

本文由农民伯伯 × 言舟(Claude) 协作完成  
*农民伯伯对本内容的准确性和价值观负责。*