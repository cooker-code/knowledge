---
title: 使用Playwright MCP(模型上下文协议)和GitHub MCP进行AI驱动的端到端(E2E)测试
author: TesterHome社区
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkxMDM1NDQ0OA==&mid=2247522495&idx=1&sn=33f7d868ea35eb7fb08bd0c9c16f9dc7&chksm=c0b524bad21312b2ad7b957db8c149f03cf3e6faf79e38c924aefef7deaa5447354f562fe814&mpshare=1&scene=24&srcid=1016HfFk0rC6V3vzcp328PoT&sharer_shareinfo=d8df21320f2bf1e52f74b2b9ef6b192a&sharer_shareinfo_first=d8df21320f2bf1e52f74b2b9ef6b192a#rd
---

**整理编译｜**TesterHome社区

作者｜Kailash Pathak

**以下为作者观点：**

在快速发展的软件测试世界中，利用像模型上下文协议（MCP）服务器这样的 AI 驱动工具，可以改变我们构建和维护自动化框架的方式。本文将引导你使用 Playwright MCP 和 GitHub MCP 服务器进行端到端测试。

通过本地执行代码、通过 GitHub MCP 服务器将代码推送到 GitHub 以及在 GitHub Actions 中自动运行，这种方法通过让 AI 工具探索网站并生成结构化代码，同时集成版本控制和部署以实现可扩展性，最大限度地减少了手动编码。

问题陈述  
  
编写和维护端到端（E2E）测试自动化通常既耗时、重复又容易出错。开发人员需要花费大量精力进行设置、编写页面对象、将代码推送到仓库以及配置 CI/CD 流水线。  
  
我们提出的解决方案是使用 AI 驱动的 MCP 服务器（Playwright MCP + GitHub MCP）来：  
  
• 通过让 AI 生成结构化的 Playwright 测试代码（基于 POM）来减少手动编码。  
• 通过 GitHub MCP 自动化 DevOps 步骤，例如分支创建、代码推送和 CI/CD 集成。  
• 标准化工作流程，确保测试在本地和 GitHub Actions 中都能一致运行。  
  
前提条件  
  
• 已安装 Node.js 18+（用于 Playwright MCP）。  
• JS/TS  
• 一个 GitHub 账户，带有用于仓库访问的个人访问令牌（PAT）。  
• 一个与 MCP 兼容的 IDE，如 VS Code 或 Cursor，用于服务器集成。  
• GitHub Copilot 已设置完毕。  
  
为了自动化端到端（E2E）测试流程，让我们从设置 Playwright 和 GitHub MCP 服务器开始。  
  
使用 VS Code 设置 Playwright MCP  
  
要利用 Playwright MCP 的潜力，你需要在 VS Code 中对其进行配置，允许 AI 代理与 Playwright 管理的浏览器进行通信。以下是两种简单的安装和配置 MCP 的方法。  
  
方法1：通过 VS Code 终端快速设置  
  
最快的开始方法是通过 VS Code 的终端注册 Playwright MCP 服务器。此方法与平台无关，适用于 VS Code 的稳定版和 Insider 版。  
  
1. 启动 VS Code：  
▪打开 VS Code（稳定版）或 VS Code（Insider 版）。  
▪确保已安装 Node.js 和 npm，因为 MCP 依赖它们来执行。  
  
2. 打开终端：  
▪导航至"查看" > "终端"，或使用快捷键 Ctrl + ~（Windows/Linux）或 Cmd + ~（macOS）。  
  
对于 VS Code 稳定版，运行以下命令：

```
code --add-mcp '{"name":"playwright","command":"npx","args":["@playwright/mcp@latest"]}'
```

对于 VS Code Insider 版，运行以下命令：

```
code-insiders --add-mcp '{"name":"playwright","command":"npx","args":["@playwright/mcp@latest"]}'
```

3. 确认设置：

▪此命令将注册 MCP 服务器，使 VS Code 扩展（例如 GitHub Copilot）能够在需要浏览器自动化时自动启动它。  
▪通过触发 AI 驱动的任务（例如生成 Playwright 脚本）进行测试，确保服务器正确启动。  
  
方法2：在 settings.json 中自定义配置  
  
如需更多控制或定制设置，你可以在 VS Code 的 settings.json 文件中手动配置 Playwright MCP。此方法非常适合添加自定义参数或与特定工作流程集成。  
  
1. VS Code 设置：

▪转到"文件" > "首选项" > "设置"，或按 Ctrl + ,（Windows/Linux）或 Cmd + ,（macOS）。  
▪点击右上角的"打开设置（JSON）"图标以编辑 settings.json 文件。  
  
2. 添加 MCP 配置：

在根对象中插入以下 JSON 结构：

```
"mcp": {        "servers": {            "playwright": {                "command": "npx",                "args": [                    "@playwright/mcp@latest"                ]            },
```

以下是 Playwright MCP 服务器中可用的工具。

现在让我们看看如何在 VS Code 中设置 GitHub MCP。  
  
使用 VS Code 设置 GitHub MCP  
  
GitHub MCP 服务器是模型上下文协议（MCP）的一个实现，它使 AI 工具能够与 GitHub 的生态系统无缝交互。它充当 AI 驱动的应用程序和 GitHub API 之间的桥梁，允许你：  
  
• 自动化工作流程，例如创建分支、推送代码或触发 CI/CD 流水线。  
• 从仓库、问题或拉取请求中提取和管理数据。  
• 构建智能集成，使 AI 代理能够直接参与版本控制和项目管理任务。  
  
前提条件  
  
开始之前，请确保你已：  
• 在系统上安装并运行 Docker  
• 一个 GitHub PAT（个人访问令牌），具有你计划使用的 GitHub API 的适当权限  
  
步骤1：  
  
1.在 VS Code 中，点击"管理" → "设置"  
2.点击"编辑 settings.json"  
3.添加以下 .JSON 配置

以下是 GitHub 服务配置的详细信息：  
  
• command: "docker" → 运行 Docker 作为基础命令。  
• args → 传递给 Docker 的参数：  
◦ run → 运行容器。  
◦ -i → 保持标准输入（STDIN）打开（交互模式）。  
◦ --rm → 容器退出后将其删除。  
◦ -e GITHUB\_PERSONAL\_ACCESS\_TOKEN → 将环境变量传入容器。  
◦ mcp/github → 要运行的 Docker 镜像（一个 GitHub 客户端）。  
• env → 为容器定义环境变量：  
◦ GITHUB\_PERSONAL\_ACCESS\_TOKEN 设置为 ${input:github\_token}（一个占位符，在运行时会被你实际的 GitHub 令牌替换）。  
  
步骤2：  
  
1.打开链接 https://github.com/settings/personal-access-tokens  
2.从"设置" -> "开发者设置"创建令牌

一旦令牌（例如 input:github\_token）创建完成，它将显示在"精细令牌"子部分中。

你可以看到我们已经创建了 PAT，现在让我们看看如果尝试在不创建个人访问令牌的情况下推送代码会发生什么。  
  
不创建 PAT（个人访问令牌）的情况下推送代码  
  
当用户尝试在没有 PAT 的情况下推送代码时，会显示以下错误，用户无法将代码推送到 CI/CD GitHub。

Playwright 和 GitHub MCP 服务器的设置已经完成，下一步是创建测试用例并使用 GitHub MCP 服务器推送代码。  
  
使用 Playwright MCP 创建基本框架  
  
现在让我们创建一个基本的 Playwright 框架，其中包括使用 POM 的完整文件夹结构。  
  
在 VS Code 中使用 GitHub Copilot 运行以下提示：

```
Prompt：  
使用 Playwright 和 JavaScript 为以下步骤创建一个 POM 模型：1. 打开 https://www.saucedemo.com/2. 使用用户名和密码登录3. 将产品 "Sauce Labs Backpack" 添加到购物车4. 打开购物车5. 点击结账按钮6. 在名字、姓氏和邮编中填写随机数据7. 点击继续按钮8. 点击完成按钮9. 验证消息 "Thank you for your order!"
```

到目前为止，我们已经通过在 GitHub Copilot 中运行提示创建了测试用例。现在让我们使用提示创建分支。  
  
使用提示创建分支  
  
现在使用以下提示创建一个名为 e2eMCPTesting 的分支。  
  
运行以下提示：

```
Prompt：创建名为 "e2eMCPTesting" 的分支
```

我们已经使用提示创建了分支，但要将代码推送到 CI/CD，你需要手动创建分支，或者我们可以通过提供提示来创建。  
  
在 GitHub 中手动创建分支  
  
现在分支已经创建，下一步是登录 GitHub 并创建一个同名分支（例如 e2eMCPTesting），以便我们可以推送代码。

```
*** 如果你不想手动创建仓库，这是一个替代方法 ***1. 给出登录 GitHub.com 的提示，系统会要求你使用有效的凭据登录。2. 然后输入仓库名称，最终创建仓库。
```

分支已创建，下一步是将代码推送到 CI/CD。  
  
使用提示推送代码  
  
运行以下提示将代码推送到 CI/CD（GitHub）。

```
Prompt：将代码推送到远程仓库 URL https://github.com//e2eMCPTesting.git
```

在下面的截图中，你可以看到代码已推送到 GitHub。

代码已推送，下一步是执行测试用例。  
  
在 CI/CD（GitHub Action）中运行代码  
  
到目前为止，测试用例已经执行，代码已推送到 GitHub，下一步是在 GitHub Actions 中运行测试用例。  
  
现在在 Copilot 中运行以下提示，在 GitHub Action 中运行测试用例。

```
Prompt：在 GitHub Actions 中运行测试用例
```

在下面的截图中，你可以看到测试用例开始在 GitHub Action 中执行。

在下面的截图中，你可以看到测试用例在 GitHub Action 中成功执行。

在下面的截图中，你可以看到 .html 报告已成功生成。

打开报告并验证测试用例是否成功执行。

这样，使用 Playwright 和 GitHub MCP 服务器，我们可以自动化端到端场景。  
  
一些与 GitHub Action 通信的提示  
  
现在让我们用一些提示来测试 GitHub Action。  
  
Prompt-1

```
Prompt-1你能帮我检查一下在 GitHub Action 中，上次代码推送的所有测试用例是否都通过了吗？
```

在下面的截图中，你可以看到提示回答的结果相同。

Prompt-2

```
Prompt-2在 GitHub Action 中重新运行上一个任务。
```

Prompt-3

```
Prompt-3在 GitHub Action 中仅针对 webkit 浏览器运行测试用例。
```

在下面的截图中，你可以看到当我给出提示时，它首先自动更新了 .config 文件，然后推送代码并仅在 webkit 浏览器中运行测试用例。

在下面的截图中，在 GitHub Action 中，测试用例仅在 webkit 浏览器中执行。

使用 MCP 有很多好处，但是从安全角度来看，我们必须记住几个关键考虑因素。  
  
安全角度  
  
从安全角度来看，谨慎处理个人访问令牌（PAT）和仓库凭据至关重要。始终：  
  
• 保护凭据：始终创建具有最小权限范围的精细个人访问令牌（PAT），并将它们作为加密密钥存储在 GitHub Actions 中，而不是以明文形式存储在本地。  
• 审查AI输出：永远不要盲目推送 AI 生成的代码。检查测试是否有跳过的验证、不安全的选择器或可能削弱测试可靠性的配置。  
• 更新依赖项：保持 Playwright、Docker 镜像和 MCP 服务器的补丁更新，以避免将流水线暴露于已知漏洞。  
• 访问控制：限制谁可以触发 MCP 驱动的工作流程，并确保日志不会泄露敏感信息。  
  
总结  
  
AI 驱动的 MCP 服务器给我们思考端到端测试的方式带来了重大转变。开发人员不再需要花费数小时编写样板代码、设置仓库或手动配置 CI/CD，而是可以依靠 Playwright MCP 和 GitHub MCP 来处理重复的基础工作。  
  
这种方法并没有消除测试工程师的角色——反而增强了它。团队可以将精力集中在有意义的工作上，例如设计具有弹性的测试用例、解决复杂场景和提高整体测试覆盖率，而 AI 则负责日常的自动化工作。

原文链接：https://kailash-pathak.medium.com/ai-powered-e2e-testing-with-playwright-mcp-model-context-protocol-and-github-mcp-d5ead640e82c

**[招聘｜校招、社招都有，上海人工智能实验室--测试开发岗位招聘](https://mp.weixin.qq.com/s?__biz=MzkxMDM1NDQ0OA==&mid=2247522440&idx=1&sn=f822ddbc19ebaf45f5f5cb51ac3e1b4a&scene=21#wechat_redirect)**

**[大厂不再招测试？技术浪潮下测试相关职业的发展新局](https://mp.weixin.qq.com/s?__biz=MzkxMDM1NDQ0OA==&mid=2247522416&idx=1&sn=f540e6246f44175d66004678bd381b45&scene=21#wechat_redirect)**

**[自动化测试｜AI与智能策略结合，我如何构建一个自修复测试框架（开源）](https://mp.weixin.qq.com/s?__biz=MzkxMDM1NDQ0OA==&mid=2247522399&idx=1&sn=7893fb6cfd18a7c072065bf786bb63a4&scene=21#wechat_redirect)**

**[用户代理：TikTok多用户交互功能的测试](https://mp.weixin.qq.com/s?__biz=MzkxMDM1NDQ0OA==&mid=2247522362&idx=1&sn=345a78c5bf1942a15d8512d903d0d183&scene=21#wechat_redirect)**

**[为什么70%的测试自动化项目会失败](https://mp.weixin.qq.com/s?__biz=MzkxMDM1NDQ0OA==&mid=2247522345&idx=2&sn=010533707a0d8f2c4477ee624db209e3&scene=21#wechat_redirect)**

**[AI驱动测试：从辅助到自主的演进之路—— 有赞、携程、联通三大实践解析](https://mp.weixin.qq.com/s?__biz=MzkxMDM1NDQ0OA==&mid=2247522318&idx=1&sn=a0a0b81060b5de6ced2558b925759b82&scene=21#wechat_redirect)**