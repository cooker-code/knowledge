> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor工程使用与上下文边界|Cursor工程使用与上下文边界]]
---
title: OpenCode：开源版Cursor横空出世，AI程序员从此不再卷
author: 红鱼AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzIxNTUxNDg4MA==&mid=2247484657&idx=1&sn=a1f8480bdca6bd78dc4580c0b9f06fb8&chksm=964064e74ef892b11b4f08dcbd7d50659441a1e6ccd8ba33de8036e8425f42fd0e3fc6ebbb0e&mpshare=1&scene=24&srcid=0106Nh88MDefX5LHdx1iVz5O&sharer_shareinfo=8558d113c14e12ee3bc612c536d44fbd&sharer_shareinfo_first=8558d113c14e12ee3bc612c536d44fbd#rd
---

先上GitHub项目地址：https://github.com/sst/opencode

再求一波关注（文末点击查看原文，更多项目和体验）：

## 前言：当AI遇上代码编辑器

最近科技圈最热闹的是什么？不是某个大厂又发了新手机，而是AI编程工具的爆发式增长。Cursor、GitHub Copilot、Cline这些工具轮番上阵，搞得咱们程序员既兴奋又焦虑——明天会不会就被AI取代了？

但今天要聊的这个项目，可能是目前最"硬核"的AI编程工具。不是因为它功能有多花哨，而是因为它把"开源"这个标签玩到了极致，而且架构设计堪称教科书级别。

## OpenCode是什么？

简单来说，OpenCode是一个完全开源的AI编程助手平台。它不是简单的代码补全工具，而是一个完整的AI驱动开发环境。最关键的是，它不像某些商业工具那样把核心功能锁在黑盒子里——所有代码、所有设计、所有逻辑，都在GitHub上公开可见。

项目的核心包括几个关键部分：

1. **多客户端支持**

   ：桌面应用、Web界面、VS Code插件、Zed编辑器集成
2. **Agent系统**

   ：Build Agent和Plan Agent协作完成复杂开发任务
3. **工具框架**

   ：丰富的内置工具和自定义工具能力
4. **LSP集成**

   ：完整的语言服务器协议支持
5. **MCP协议**

   ：Model Context Protocol，让AI能理解更复杂的上下文

## 怎么用起来？

别急，咱们一步一步来。

### 第一步：安装桌面应用

最推荐的方式是下载桌面应用。项目提供了多种安装方式，但咱们选最简单的：

```
# 如果你有Homebrew（macOS用户必备）brew install sst/opencode/opencode# 或者直接下载预编译的包# 访问GitHub Releases页面，找到对应系统的安装包
```

安装完成后，启动桌面应用，你会看到一个简洁但功能齐全的界面。没有花哨的动画，没有令人困惑的设置选项——就是纯粹的开发环境。

### 第二步：配置AI Provider

OpenCode本身不提供AI服务，它需要一个AI Provider来工作。支持主流的几种：

1. **OpenAI官方API**

   ：最直接的方式，但需要付费
2. **Anthropic Claude**

   ：目前很多开发者认为最懂代码的模型
3. **本地模型**

   ：通过配置自定义Provider，可以使用本地部署的模型

配置文件在你的项目目录下，创建一个`opencode.config.ts`：

```
import { defineConfig } from "opencode/config";export default defineConfig({  providers: {    openai: {      apiKey: process.env.OPENAI_API_KEY,      model: "gpt-4-turbo-preview"    },    anthropic: {      apiKey: process.env.ANTHROPIC_API_KEY,      model: "claude-3-opus-20240229"    }  },  defaultProvider: "openai"});
```

记得把API密钥存到环境变量里，别傻傻地硬编码在配置文件中。

### 第三步：开始会话

启动OpenCode后，打开你的项目目录。然后按下快捷键（默认是`Cmd+K`或`Ctrl+K`），会弹出对话界面。

这时候你可以开始和AI对话了。但OpenCode不是简单的聊天机器人，它理解代码上下文：

```
用户：在src/utils/目录下创建一个文件操作的工具函数库，包含读取、写入、删除文件的方法，要有类型定义
```

OpenCode会：

1. 分析你的项目结构
2. 创建合适的文件
3. 编写符合你项目风格的代码
4. 自动添加必要的类型定义
5. 甚至帮你写单元测试

### 第四步：使用Plan Agent做复杂任务

对于简单的代码生成，一个命令就够了。但对于复杂的多步骤任务，建议用Plan Agent。

Plan Agent的特点是"先计划，再执行"：

```
用户：重构现有的用户认证模块，迁移到JWT，同时更新所有相关的API端点
```

Plan Agent会：

1. 分析现有代码结构
2. 制定详细的迁移计划
3. 等待你确认计划
4. 按步骤执行每个任务
5. 在关键节点请求确认

这种设计避免了一次性修改太多代码导致的问题，也给你留了审查的机会。

### 第五步：自定义工具

OpenCode的强大之处在于可扩展。你可以添加自定义工具，让AI能执行特定任务。

在项目根目录创建`opencode/tools.ts`：

```
import { defineTool } from "opencode/tools";export const runTests = defineTool({  name: "run-tests",  description: "运行项目的测试套件",  execute: async () => {    const { exec } = await import("child_process");    return new Promise((resolve, reject) => {      exec("npm test", (error, stdout, stderr) => {        if (error) {          reject({ success: false, error: error.message });        } else {          resolve({ success: true, output: stdout });        }      });    });  }});
```

然后在配置中注册：

```
export default defineConfig({  tools: [runTests],  // ...其他配置});
```

现在AI就可以在需要时自动运行测试了。

## 高级用法

### 会话压缩

长时间的编程会话会产生大量上下文，OpenCode会自动压缩旧的会话历史，但只保留重要信息。这个过程是透明的，不需要你操心。

但如果想手动控制，可以在配置中设置：

```
export default defineConfig({  session: {    maxMessages: 100,    compressionThreshold: 50  }});
```

### LSP集成

OpenCode集成了LSP，这意味着它能提供真正的智能提示——不是简单的自动补全，而是基于语义理解的代码建议。

比如你输入一个函数调用，它不仅补全参数名，还会解释每个参数的作用。这是因为它真正理解了代码的语义，而不仅仅是做文本匹配。

### 快照系统

OpenCode会在关键时刻自动创建代码快照。这样即使AI的操作搞乱了代码，你也可以随时回滚。

快照保存在`.opencode/snapshots`目录下，每个快照都是一个完整的git commit。你也可以手动创建快照：

```
用户：创建一个当前状态的快照
```

## 实际应用场景

### 场景一：接手遗留代码

你刚接手一个三年没人维护的项目，代码混乱不堪。手动理解需要几周，但用OpenCode：

1. 让AI分析项目结构，生成架构文档
2. 询问特定模块的业务逻辑
3. 让AI找出潜在的bug
4. 逐步重构最混乱的部分

不是魔法，但比纯手动快得多。

### 场景二：快速原型开发

产品经理突然说"我们要加个功能"，要求下周一上线。传统流程是需求分析、设计、编码、测试，每个环节都要时间。

用OpenCode，你可以：

1. 快速生成原型代码
2. 让AI补充细节实现
3. 自动编写测试用例
4. 一边开发一边调整

一周时间，从想法到上线，不是不可能。

### 场景三：代码审查优化

作为技术负责人，你没有时间仔细review每一行代码。可以让OpenCode先做第一轮审查：

1. 找出潜在的bug和安全隐患
2. 识别代码异味
3. 建议性能优化
4. 检查类型安全

你只需要关注真正重要的部分，节省大量时间。

### 场景四：学习新技术

想学Rust但又不想啃厚书？让OpenCode作为你的导师：

1. 从简单的示例开始
2. 逐步增加复杂度
3. 让AI解释每个概念
4. 在实战中学习

比看书有趣，比看视频灵活。

## 写在最后

OpenCode不是要取代程序员，而是要成为程序员的超级助手。它处理繁琐的细节，让你专注于真正重要的问题。

开源的意义在于，所有人都能参与改进，所有人都能从中受益。如果你对AI编程感兴趣，不妨试试OpenCode——不仅好用，还能学到很多架构设计的知识。

GitHub项目地址：https://github.com/sst/opencode
项目文档地址：https://opencode.dev/docs
Discord社区地址：https://discord.gg/opencode

下次有人问你"AI会不会取代程序员"，你可以笑着说：不会，但会用AI工具的程序员会取代不会用的。

看到这里了，关注一波吧！

点击左下角查看原文，更多项目和体验