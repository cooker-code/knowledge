---
title: Claude Code官方支持插件体系！一行命令 = Claude Code全家桶！
author: 我姚学AI
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk1Nzc3MTU5MA==&mid=2247485520&idx=1&sn=471879f3b0570d762e60ce9833049455&chksm=c214e7538b4465c958647adb6f85e927e224796d4a91b5a45360f71ca68f17251b4acd1e3aea&mpshare=1&scene=24&srcid=1014SqXzDAXWRnWg5WSGSzZb&sharer_shareinfo=0dae2a5300c218f47433b2c70e42c1d9&sharer_shareinfo_first=0dae2a5300c218f47433b2c70e42c1d9#rd
---

> 大家好，我是姚路行，一个爱搞AI的大厂程序员，也是一个90后奶爸

**关注下****方****公众******号******，回复【claude】免费****领取价值799的Claude Code超全学习资料**

作为一名日常使用 Claude Code 的开发者，当我看到 v2.0.12 版本推出插件功能时，第一反应是：这会不会又是一个鸡肋功能？

但当我花了一个下午时间深度体验后，我必须说：**这可能是 Claude Code有史以来最重要的更新**。

为什么这么说？让我带你从零开始，完整体验一遍这个功能。

## 插件到底能干嘛？

首先你要知道，Claude Code 有四种强大的扩展功能

1. **Slash Commands（斜杠命令）：输入 / 一键触发自定义操作**
2. **Subagents（子代理）：创建专属用途的AI代理**
3. **MCP Servers：让Claude Code能够连接外部工具和数据源**
4. **Hooks：在关键流程点插入自定义逻辑和行为**

虽然这四大功能很强，但依然有他的问题，不仅配置困难，而且分享困难，尽管你有很棒的工作流，想分享给同事还是要手把手的教。

### 插件功能的解决方案

**一句话总结：把所有配置打包成一个工具包，通过一条命令搞定安装。**

```
/plugin install <插件名>@<市场名>
```

更关键的是：**你可以随时启用或禁用插件**，需要时打开，不需要时关闭，不会让系统提示词变得又臭又长。

## 安装并使用插件

理论说完了，让我带你实际操作一遍如何安装并使用插件，Claude Code 的插件都是通过“插件市场”来分发的，所以首先要了解什么是插件市场。

插件市场其实就是一个插件的集合。简单说就是一个git仓库，里面放一个.claude-plugin/marketplace.json 文件，列出了所有可用的插件。

接下来步入正题，带你安装并使用插件，以官方的插件为例！

### Step 1：添加插件市场

在 Claude Code 中输入：

```
/plugin marketplace add anthropics/claude-code
```

回车后，系统提示市场添加成功。就这么简单，不到 5 秒钟。

### Step 2：浏览和安装插件

### 方式一：交互式菜单（推荐）

接下来输入：

```
/plugin
```

会弹出一个交互式菜单，有几个选项

选择“Browse and install plugins”，立刻看到了一长串插件列表：

每个插件都有清晰的描述，告诉你它能做什么，选择其中一个/多个安装。

### 方式二：命令式安装（快速安装）

```
/plugin install commit-commands@anthropics/claude-code
```

### Step 3：验证效果

重启后，我在项目目录里输入：

```
/commit-commands
```

就可以自动补全，看到这个插件里的命令了，棒！

## 管理插件

### 启用/禁用插件

不想让插件一直运行？可以随时禁用：

```
# 启用插件  
/pluginenable<插件名>@<市场名>  
  
# 禁用插件（不卸载）  
/plugindisable<插件名>@<市场名>
```

**关键点**：禁用的插件不会加载到系统提示词中，不会影响性能。

### 卸载插件

```
/pluginuninstall<插件名>@<市场名>
```

### 查看已安装的插件

```
/plugin
```

选择"Manage and uninstall plugins"，可以看到所有已安装的插件，以及它们的启用状态。

## 创建自己的插件（保姆级教程）

看到这里你可能会想：“这些插件看起来很复杂，我能做吗？”别担心，我带你走一遍，你会发现**真的很简单**。

### 目标：创建一个“问候插件”

我们要做一个简单的插件，提供一个/hello 命令，让 Claude 热情地向用户问好。

### Step 1：创建项目结构

```
# 创建测试市场  
mkdir test-marketplace  
cd test-marketplace  
  
# 创建插件目录  
mkdir my-first-plugin  
cd my-first-plugin
```

### Step 2：创建插件配置文件

```
# 创建.claude-plugin目录  
mkdir .claude-plugin  
  
# 创建plugin.json  
cat > .claude-plugin/plugin.json << 'EOF'  
{  
"name": "my-first-plugin",  
"description": "我的第一个问候插件",  
"version": "1.0.0",  
"author": {  
    "name": "我姚学AI"  
  }  
}  
EOF
```

这个文件是插件的"身份证"，定义了插件的基本信息。

### Step 3：添加自定义命令

```
# 创建commands目录  
mkdir commands  
  
# 创建hello命令  
cat > commands/hello.md << 'EOF'  
---  
description: 向用户打招呼  
---  
  
# Hello Command  
热情地向用户问好，并询问今天能提供什么帮助。让问候更个性化和鼓舞人心。  
EOF
```

**重点**：命令的定义就是一个 Markdown 文件，非常直观。

### Step 4：创建市场配置

```
# 回到test-marketplace目录  
cd ..  
  
# 创建市场的.claude-plugin目录  
mkdir .claude-plugin  
  
# 创建marketplace.json  
cat > .claude-plugin/marketplace.json << 'EOF'  
{  
"name": "test-marketplace",  
"owner": {  
    "name": "测试用户"  
  },  
"plugins": [  
    {  
      "name": "my-first-plugin",  
      "source": "./my-first-plugin",  
      "description": "我的第一个测试插件"  
    }  
  ]  
}  
EOF
```

### Step 5：安装并测试

```
# 从父目录启动Claude Code  
cd ..  
claude  
  
# 在Claude Code中添加测试市场  
/plugin marketplace add ./test-marketplace  
  
# 安装你的插件  
/plugin install my-first-plugin@test-marketplace  
  
# 选择“Install now”，然后重启Claude Code  
  
# 测试你的新命令  
/hello  
  
# 查看命令列表  
/help
```

### 成功了！你已经创建了第一个插件！

### 进阶：创建更复杂的插件

掌握了基础后，你可以创建更强大的插件，可以尝试创建一些自定义子代理、自定义Hooks以及引用指定的MCP Servers。

## 社区生态

如果关注我比较久的同学，应该知道我之前介绍了两个项目：

[9大领域！75+子代理！你想要的Claude Code子代理全都在这里！](https://mp.weixin.qq.com/s?__biz=Mzk1Nzc3MTU5MA==&mid=2247485254&idx=1&sn=cd670f95a61fa7a707683ad28fa40f11&scene=21#wechat_redirect)

[一行命令 = Claude Code全家桶！](https://mp.weixin.qq.com/s?__biz=Mzk1Nzc3MTU5MA==&mid=2247485433&idx=1&sn=b33fcda156eeedcb193281451b6a68ed&scene=21#wechat_redirect)

插件功能刚推出不久，他们就已经适配了Claude Code的插件系统，并且在社区开始爆发了，再加上官方插件仓库，地址如下：

### 1. `wshobson子代理库`

包含 80+ 专业子代理

```
地址：https://github.com/wshobson/agents
```

安装：

```
/plugin marketplace add wshobson/agents
```

### 2. davila7`插`件市场

包含：DevOps 自动化工具、文档自动生成、项目管理助手、测试套件

```
地址：https://github.com/davila7/claude-code-templates
```

安装：

```
/plugin marketplace add davila7/claude-code-templates
```

### 3. Anthropic 官方示例

安装官方市场：

```
/plugin marketplace add anthropics/claude-code
```

包含官方维护的示例插件，可以作为学习参考。

## 写在最后

坦白说，我刚看到插件功能的时候，觉得就是个“方便打包”的工具，没什么大不了的。

但真正用起来之后，我发现这个功能改变的不只是配置方式，**它改变的是整个协作模式**。

插件功能不仅可以随时开关，不会影响到系统的速度，而且可以多插件配置使用，非常有效。

并且插件功能是完全免费的，在 Terminal 和 VS Code 扩展都能使用，太棒了！

以前，每个人都是孤岛，各自摸索。

现在，我们可以站在彼此的肩膀上，用最好的工具、最佳的实践。

## 福利时间

关注我，后台回复【claude】即可领取价值 799 的 Claude Code 学习资料，祝大家能快速掌握并精通 Claude Code，加油哦！

## 往期优质文章

[Vibe Coding已死？Spec Coding当立！](https://mp.weixin.qq.com/s?__biz=Mzk1Nzc3MTU5MA==&mid=2247485473&idx=1&sn=40c7de5ade4747d1f4d010e38767ea9b&scene=21#wechat_redirect)

[重磅！Claude Sonnet 4.5 + Claude Code 2.0来袭！](https://mp.weixin.qq.com/s?__biz=Mzk1Nzc3MTU5MA==&mid=2247485467&idx=1&sn=6b0490d2e9d28f3eef2f3aa7d53e994e&scene=21#wechat_redirect)

[OpenAI官方推出Codex最佳实践！【附获取方法】](https://mp.weixin.qq.com/s?__biz=Mzk1Nzc3MTU5MA==&mid=2247485446&idx=1&sn=2b58b3f371f56e1dec669e3dfe2bc2f0&scene=21#wechat_redirect)

[一行命令 = Claude Code全家桶！](https://mp.weixin.qq.com/s?__biz=Mzk1Nzc3MTU5MA==&mid=2247485433&idx=1&sn=b33fcda156eeedcb193281451b6a68ed&scene=21#wechat_redirect)

[官方认证！Claude Code手机版来了！](https://mp.weixin.qq.com/s?__biz=Mzk1Nzc3MTU5MA==&mid=2247485415&idx=1&sn=421411b7cc4003f51878d1f49d862aef&scene=21#wechat_redirect)

## 一起学习