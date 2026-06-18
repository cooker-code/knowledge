> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 拳打 Claude Code，脚踢 Codex？新的 AI 编程“猛兽” Factory Droid 来了！
author: 老烨的AI工具实验室
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5ODM4ODY4MA==&mid=2247484001&idx=1&sn=f0407dd2115cbb5cdba39e0929c1c086&chksm=97def3cf0db82b4416a6f55c8f069a3dc69c505195b656691d3e08fad6ea25c0c6c27af8b2fb&mpshare=1&scene=24&srcid=1028AJsDNHOb1K9zKsarAh0C&sharer_shareinfo=b5134eb2b2c6514502885764fe562315&sharer_shareinfo_first=b5134eb2b2c6514502885764fe562315#rd
---

Factory Droid

# AI编程工具 #

开发圈疯狂刷屏

AI-POWERED DEV

国庆假期结束了，好好休息了一段时间（鸽了10天）。

好在这个假期，AI 圈还算风平浪静，没啥爆炸性的新闻，心心念念的Gemini 3也还在路上。

但就在这片平静之下，一个叫 Factory Droid 的家伙，正在国外开发圈疯狂刷屏。一开始我以为又是哪个新模型，深入一扒才发现。好家伙，

打了类固醇的Claude Code！

简单来说，它也是AI编程工具，但是它让本来就很强的模型，变得更强、更聪明、更会干活。那这家伙，到底猛在哪？

一、硬核数据：性能比原厂提升36%

不玩虚的，直接上全球最严苛的软件开发基准测试 Terminal-Bench 的数据。这个测试模拟的是真实开发场景，比如调试、代码现代化、云设施管理等，含金量极高。                         看明白了吗？最炸裂的一点是：同样使用 Claude Opus 4.1 内核，Factory Droid 的得分比 Anthropic 官方的 Claude Code 高出 15.6%，相对性能提升了整整 36%！

真.用敌人的矛，打爆敌人的盾。

二、真实场景：干活利索才是王道！

1. 打破工具墙，让 AI 融入真实工作流

全平台覆盖只是基础操作。Droid 强在深度集成，无缝衔接 CLI、IDE、Slack、GitHub、Figma、Jira、Linear 等。这意味着不只是程序员能用，产品经理、设计师都能在自己熟悉的工具里（比如 Slack 实时委托、Jira 任务分配）与 Droid 协作，这远超对手简单的 stdio 适配。

2. 拒绝“二选一”，模型自由才是真香！

成年人全都要，模型当然可以随便换。Droid 支持市面上几乎所有主流模型，更重要的是，它还支持接入本地模型和其他第三方 API 提供商。没错，这意味着强大的 GLM-4.6 也能轻松接入，实现真正的模型自由！（文末我附上GLM4.6的接入步骤）

3. 极致优化，Token 消耗暴降 90%！

这可能是最实在的优点。性能更强的同时，花销反而更少。yfinance 的 CTO 公开分享了他们从 Claude Code 切换到 Factory Droid 后的体验：每日的 Token 使用量从 1 亿（100M）个暴降到了 1000 万（10M）个。降本增效，效果显著。

总的来说，Factory Droid 就是一个“性能放大器”。它通过顶级的代理优化，让你手里的任意大模型（无论是 Claude、GPT 还是 GLM）都能爆发出远超其“官方工具”的实战能力，同时还更省钱、更灵活。

三、快速开始

注册账号

首先需要登录https://app.factory.ai/进行注册

安装Droid

Mac用户

```
#安装droid curl -fsSL https://app.factory.ai/cli | sh #进入你的项目目录 cd your-project #启动 droid
```

Windows 用户

```
#安装droid irm https://app.factory.ai/cli/windows | iex #设置环境变量（必须做） setx PATH "$Env:Path;$Env:USERPROFILE\bin" (运行后需重启命令行窗口才能生效) #进入你的项目目录 cd your-project #启动droid
```

配置自定义模型

配置`~/.factory/config.json`以windows为例:

```
#路径 C:\Users\你的用户名\.factory\config.json #新增配置 {         "custom_models":[                 {                         "model_display_name": "GLM-4.6",                             "model": "glm-4.6",                         "base_url": "https://open.bigmodel.cn/api/anthropic",             "api_key": "xxxxxx",                         "provider": "anthropic"                 }         ] }
```

其他高级用法

mcp，子agent等高级用法参见官方文档

https://docs.factory.ai/cli/getting-started/quickstart

搞定收工，Factory Droid 是不是你的菜，试过才知道！

现在注册还送4000 万的免费token，可以用来白嫖GPT-5-CODEX和Sonnect 4.5等顶尖模型，快去体验这个开发效率猛兽吧！！