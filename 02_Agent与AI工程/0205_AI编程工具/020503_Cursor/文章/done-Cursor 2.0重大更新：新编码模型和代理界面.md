> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor工程使用与上下文边界|Cursor工程使用与上下文边界]]
---
title: Cursor 2.0重大更新：新编码模型和代理界面
author: 未来的回响
date:
url: https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247484591&idx=1&sn=324f2df78b9afb28e93c6afad8f4660a&chksm=917b9bc8b34c4f8892173bfb235abc036b124ed770cd2294505d184318c4e35be6fbbd92dd57&mpshare=1&scene=24&srcid=1104OeUIhseFNMboqXSw1T7N&sharer_shareinfo=ead4e17f8c77c36592af455295a97646&sharer_shareinfo_first=ead4e17f8c77c36592af455295a97646#rd
---

Cursor于10.29发布了2.0大版本更新，更新了诸多功能包含了自研Composer模型、多代理、UI界面、语音模式等，这下可以在Cursor中直接单纯用嘴指挥AI干活儿了。

## **一、多代理**

在我们的新编辑器中管理代理，包括一个用于代理和计划的侧边栏。

在单个提示下并行运行多达八个代理。这使用 Git 工作树或远程机器来防止文件冲突。每个代理在其自己的隔离代码库副本中运行。

## **二、Composer**

推出我们的第一个代理式编码模型。Composer 是一个前沿模型，比类似智能水平的模型快 4 倍。

## **三、浏览器 (GA)**

在 1.7 版本中以 Beta 形式推出，代理的浏览器现在已进入 GA。我们为企业团队添加了额外支持，以在 2.0 中使用浏览器。

浏览器现在可以嵌入到编辑器中，包括强大的新工具来选择元素并将 DOM 信息转发给代理。了解更多关于使用浏览器的信息。

## **四、改进的代码审查**

现在更容易查看代理在多个文件中的所有更改，而无需在单个文件之间跳转。

## **五、沙盒终端 (GA)**

在 1.7 版本中以 Beta 形式推出，沙盒终端现在已进入 GA，适用于 macOS。我们现在默认在 macOS 上以 2.0 版本在安全沙盒中运行代理命令。

Shell命令（未被列入白名单的）将自动在沙盒中运行，具有对工作区的读/写访问权限，但无互联网访问权限。了解更多关于沙盒的信息。

## **六、团队命令**

在 Cursor 仪表板中为您的团队定义自定义命令和规则。

此上下文将自动应用于您的团队所有成员，而无需在编辑器中本地存储文件，并由团队管理员集中管理。

## **七、语音模式**

使用内置的语音到文本转换来控制代理。您还可以在设置中定义自定义提交关键词，以触发代理开始运行。

## **八、改进的性能**

Cursor 使用语言服务器协议 (LSP) 来实现特定语言的功能，如跳转到定义、悬停提示、诊断等。

我们大幅改进了所有语言的 LSP 加载和使用性能。这在与代理合作并查看差异时特别明显。

对于大型项目，Python 和 TypeScript LSP 现在默认更快，具有根据可用 RAM 动态配置的更高内存限制。

我们还修复了许多内存泄漏，并改进了整体内存使用。

## **九、后台计划模式**

使用一个模型创建计划，并使用另一个模型构建计划。您可以选择在前台或后台构建计划，甚至使用并行代理来生成多个计划以供审查。

## **十、可共享的团队命令**

与整个团队共享自定义规则、命令和提示。通过 Cursor 文档创建深度链接。

## **十一、改进的提示 UI**

文件和目录现在以内联药丸形式显示。我们还改进了带有标记上下文的提示复制/粘贴。

我们删除了上下文菜单中的许多显式项，包括 @Definitions、@Web、@Link、@Recent Changes、@Linter Errors 等。代理现在可以自行收集上下文，而无需在提示输入中手动附加。

## **十二、改进的代理框架**

我们大大改进了所有模型下代理工作的底层框架。这带来了显著的质量改进，特别是针对 GPT-5 Codex。

## **十三、云代理**

云代理现在提供 99.9% 的可靠性、即时启动，以及即将推出的新 UI。我们还改进了从编辑器发送代理到云端的体验。

## **十四、Cursor for Enterprise**

### **1、沙盒终端：管理员控制**

企业现在可以跨团队强制执行沙盒终端的标准设置。在团队级别配置沙盒可用性、Git 访问和网络访问。

### **2、钩子：云分发**

企业团队现在可以直接从 Web 仪表板分发钩子。管理员可以添加新钩子、保存草稿，并选择哪些钩子应用于哪些操作系统。

### **3、审计日志**

在 Cursor 中查看管理员事件的带时间戳日志：用户访问、设置更改、团队规则编辑和成员管理事件。

## **十五、其他更新**

### **改进 (8)**

* 优化了聊天渲染的文本解析。
* 增加了 TypeScript 语言服务器的默认内存。
* 简化了聊天内的文本渲染，以减少使用 LSP。
* 简化了工作树内的渲染，以使用更简单的 LSP。
* 简化了代理在读取文件时减少使用 LSP。
* 优化了 findFiles 性能，以有界并发批处理并发调用。
* 记事本已被弃用。
* 后台代理已被重命名为云代理。

### **补丁 (1)**

2.0.1-2.0.28：错误修复

**-END-**

**更多关于**AI工具、Cursor、MCP相关**的教程和资讯请持续关注后续分享！**

**本文完整版详见公众号：**未来的回响****

**文章精校版参见知识星球：**AI工具实战派****

**【限时开放】**欢迎加入**AI工具实战派**交流群一起学习进步～

**AI编程、AI运营、工具资料分享**请加入知识星球

**-推荐阅读-**

**【AI编程】**

* [Cursor+Gitlens再也不用担心频繁重建项目了](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247483693&idx=1&sn=cffa5e0542c912297387ee2637b98cee&scene=21#wechat_redirect)
* [使用Cursor时如何规避AI改坏代码——终极指南](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247483699&idx=1&sn=7fbd243a6357d8f15e526b2a3126af5a&scene=21#wechat_redirect)
* [Cursor编程bug反复改不好⁉️让AI用思维链整理思路](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247483705&idx=1&sn=7a3658d619cf882f4f2ccd1919d69f71&scene=21#wechat_redirect)
* [怎么从Cursor转到Claude Code？配合GLM-4.5的性价比AI编程指南](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247484341&idx=1&sn=d3d70b3522d175c9448a66f2fc0daef6&scene=21#wechat_redirect)

**【AI设计】**

* [AI设计对话指南（第一期）：一文掌握20个主流UI组件库，让AI秒懂你的设计意图！](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247483978&idx=1&sn=28e4853177c30b4bbadf1acb8ac276dc&scene=21#wechat_redirect)
* [AI设计对话指南（第二期）：19种UI设计风格速查手册](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247484010&idx=1&sn=5f4773a2ffb1df2b62adb01b167f84d8&scene=21#wechat_redirect)
* [AI设计对话指南（第三期）：UI设计提示词指南，减少与AI掰扯](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247484219&idx=1&sn=e4841d0987003f0d722fd3ff3a495a2b&scene=21#wechat_redirect)
* [如何使用Magic MCP生成好看的UI](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247483898&idx=1&sn=325483d6c635e779b6cc0f105daaf69e&scene=21#wechat_redirect)
* [如何使用Magic MCP生成好看的UI（第二期）](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247483920&idx=1&sn=29ae0739dff846d72c9792291aedc23d&scene=21#wechat_redirect)
* [如何在Cursor中使用Figma MCP自动进行产品原型设计](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247484042&idx=1&sn=769e7881f49ae4529025bdb434d55c1e&scene=21#wechat_redirect)

**【AI工具】**

* [【2025最全】12个顶级MCP服务器资源汇总，Cursor/Claude/AI开发者必备](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247483865&idx=1&sn=9998131d2ac940084fefbfb8224206fa&scene=21#wechat_redirect)
* [简单介绍一些常用的MCP服务器，快用来提高Cursor干活效率吧](https://mp.weixin.qq.com/s?__biz=MzA4Nzc3NzkzNQ==&mid=2247483825&idx=1&sn=a6138244d10a05c96520a1dffaff9528&scene=21#wechat_redirect)