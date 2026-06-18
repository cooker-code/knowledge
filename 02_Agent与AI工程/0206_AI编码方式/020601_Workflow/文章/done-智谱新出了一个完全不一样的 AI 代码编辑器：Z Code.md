> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: 智谱新出了一个完全不一样的 AI 代码编辑器：Z Code
author: 读书不多
date:
url: https://mp.weixin.qq.com/s?__biz=MzIyMzU5Mzg2NQ==&mid=2247485355&idx=1&sn=8de5f9b7b5fea7d1723bf232645f5f62&chksm=e9cc0fca383065e580654c0073d480535f01aa5a20dc9abe619b17709e214f327a72ed6e97f2&mpshare=1&scene=24&srcid=1230JmjDvXee9ozRv6Yr2Re4&sharer_shareinfo=26d6f907ed4b84aa3c150ef1d4ab6fb3&sharer_shareinfo_first=26d6f907ed4b84aa3c150ef1d4ab6fb3#rd
---

智谱新出了一个完全不一样的 AI 代码编辑器：Z Code  https://zcode-ai.com

Z Code 界面

Z Code 是一款轻量级的 AI 代码编辑器，旨在解决命令行 AI 编程工具（如 Claude Code、Codex、Gemini等）操作门槛高的问题。它通过提供一个统一、友好的可视化桌面，将这些 Agent 的能力无缝集成，仅使用一个api key 就能丝滑切换体验多个Agent 编程工具。此外，还提供安全的文件版本管理、高效代码审查、任务 Agent、MCP 协议管理等一系列特性，为你打造一站式的 AI 辅助开发体验。

### 界面说明

Z Code 应用界面主要可分为4个区域：

1. 1. 顶部导航栏：管理窗口和文件夹
2. 2. 工具选择栏：切换内置的CLI编程工具
3. 3. 会话面板：展示用户和Agent的对话和Agent执行过程
4. 4. 预览面板：展开/隐藏右侧预览面板（代码审核、浏览器预览）

   Z Code 主界面布局

工具选择这里可以选择不同的 CLI 工具

CLI 工具选择

会话面板输入框里面，我们可以切换不同的模式和选择不同的模型。如果我们选择登录智谱账户，这里就会自动绑定并且出现 BigModel 的相关模型，前提是需要提前充值或者购买 GLM 的 Coding 套餐 [1]：https://www.bigmodel.cn/glm-coding?ic=N85GPAPFFP ，当然购买海外版也是一样的 ：https://z.ai/subscribe?ic=SAWILYNXG4

模式和模型选择

模型列表

点击 Manage Models 可以看到 添加自定义渠道的模型，比如说一些中转的 API 服务，比如 OpenRouter 这类服务。这样我可以在不同的 CLI 中方便地切换模型。但是下面这个默认的配置，并不能在 Claude Code 里面执行，需要设置为 anthropic 的消息格式。

自定义模型配置

举例来说 OpenRouter 里面配置 MiniMax2.1 可以参考如下配置，其他类似的中转同理。这样可以在 Z Code 里面准确的执行工具。

OpenRouter MiniMax 配置

剩下的其他内容就目前看中规中矩，完整的来说有以下的这些功能。

**多 CLI 集成**

* • 一个界面跑三种 CLI 工具，想用哪个切哪个
* • 支持自定义模型和中转 API，不用每个工具单独配置

**交互控制**

* • 四档权限：从每步都问到全自动执行，按需选择
* • 可以 @ 文件让 Agent 精准定位，有思考模式

**版本回溯**

* • 每轮对话自动存档，改坏了随时回滚
* • 内置 Git 面板，提交推送不用切终端

**辅助工具**

* • 文件树、终端面板、内置浏览器，前端预览方便
* • MCP 协议支持，可以扩展更多能力

Z Code 基本的功能都有了，还有不少问题，期待后续的优化和升级。Z Code 把非专业人员不方便入门和使用的命令行整合到了一个编辑器里面，这个想法太棒了，当然类似的 ACP 也可以做到，这个编辑器集成后把门槛进一步降低了，更多的人方便的去用 AI 和代码去实现自己的想法。

#### 引用链接

`[1]` Coding 套餐 : *https://www.bigmodel.cn/glm-coding?ic=N85GPAPFFP*