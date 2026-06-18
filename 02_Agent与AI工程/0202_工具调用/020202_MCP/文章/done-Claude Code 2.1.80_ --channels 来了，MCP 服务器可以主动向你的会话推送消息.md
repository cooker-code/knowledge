> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP异步通道与主动消息边界|MCP异步通道与主动消息边界]]、[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: Claude Code 2.1.80: --channels 来了，MCP 服务器可以主动向你的会话推送消息
author: AI灵感闪现
date:
url: https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490377&idx=1&sn=9c7ad1fd4b98c368848589a5373e5130&chksm=a7d54ac3cd608fe050939a8fc9ec568879ad3a94d9c5d4113df789c9740fa0dbef5f65ac8cf6&mpshare=1&scene=24&srcid=04194WZREP1MIGlTGhRZZXXJ&sharer_shareinfo=45b42aa7e4db3496619211ff307b4fb8&sharer_shareinfo_first=45b42aa7e4db3496619211ff307b4fb8#rd
---

> Claude Code 2.1.80 新增 --channels（研究预览），MCP 服务器可以主动推送消息到你的活跃会话。同时带来 effort frontmatter 让技能控制模型推理深度、rate\_limits 状态栏字段显示用量、settings 内联插件声明、修复 --resume 丢失并行工具结果、大仓库启动省 80MB 内存等 20+ 项更新。

Claude Code 的会话一直是一问一答：你说话，agent 回应。MCP 服务器扩展了这个模式，agent 可以调外部工具。但信息只往外流。agent 提问，服务器回答。

2.1.80 把反向通道打开了。`--channels`（研究预览）让 MCP 服务器可以主动向你正在运行的会话推送消息。一个只能被问到才说话的工具，学会了举手。

> **我是 AI灵感闪现，使用 OpenClaw 小龙虾 让 AI 自主管理工作和生活上的问题；使用 Claude Code + BMAD AI 驱动敏捷开发框架，让 AI 自主开发和交付软件来表达想法和灵感。****是 MoneyMind 省钱思维 App 和 HeartPetBond 心宠纽带 App 开发者。正在实践和分享让 AI 自主解决健康、生活、投资和等方面的问题。****我尽可能让 AI 自己完成从目标到交付以及演进的闭环，以最少的人为交互与监督，让 AI 自己跑流程。我只给 AI 想法或目标，全程不陪跑，让 AI 自主运行类似 Tesla FSD 自动驾驶。**

---

## 速览

| 领域 | 变化 |
| --- | --- |
| 通道 | `--channels` （研究预览）让 MCP 服务器主动推送消息到你的会话 |
| 技能 | `effort` frontmatter 让技能和斜杠命令覆盖模型 effort 级别 |
| 状态栏 | `rate_limits` 字段显示 Claude.ai 用量（5 小时和 7 天窗口） |
| 插件 | `source: 'settings'` 在 settings.json 中内联声明插件 |
| 插件提示 | 新增 CLI 工具使用检测，补充文件模式匹配 |
| 恢复 | 修复 `--resume` 丢失并行工具结果，不再显示 `[Tool result missing]` |
| 语音 | 修复 Cloudflare 机器人检测导致的 WebSocket 连接失败 |
| API | 修复通过 API 代理、Bedrock、Vertex 使用精细工具流式传输时的 400 错误 |
| 远程 | 修复 `/remote-control` 在网关和第三方部署中不应出现却出现的问题 |
| 沙盒 | 修复 `/sandbox` Tab 和方向键切换无响应 |
| UX | 大仓库 `@` 文件补全更快；改进 `/effort`、`/permissions` 导航 |
| 性能 | 25 万文件仓库启动省约 80MB 内存 |
| 设置 | 修复远程设置缓存导致托管设置启动时不生效 |
| 插件 | 插件安装提示简化为单条 `/plugin install` 命令 |

## 通道，双向了

MCP 一直是单行道。你的 agent 调用工具，服务器响应。如果服务器那边发生了什么事 -- CI 构建失败了、部署完成了、测试跑完了 -- 服务器没法告诉 agent。你得自己去问。

`--channels` 改变了这件事。MCP 服务器现在可以不等调用，主动向你的活跃会话推送消息。想想这意味着什么：

一个部署监控 MCP 服务器在看你的 staging 环境。agent 在写代码。部署挂了。服务器直接推送 "deploy failed: missing env var DATABASE\_URL on staging" 到你的会话。agent 看到了，检查配置，修了。

一个测试运行器 MCP 服务器在测试完成时推送结果。agent 不用轮询，收到通知就行。

一个团队沟通 MCP 服务器把相关的 Slack 消息或 PR review 评论推到你的编码会话里。

这是研究预览，API 可能会变。但方向已经明确：MCP 正在从工具调用协议变成双向消息层。

## 技能学会了自己定节奏

`effort` frontmatter 让技能和斜杠命令的作者在技能被调用时覆盖模型的 effort 级别。

以前 effort 是全局设置。复杂任务调高，简单任务调低，来回切。现在技能自己决定。一个代码格式化命令用最低 effort 跑，因为不需要深度推理。一个架构审查技能要求最高 effort，因为偷工减料会出烂分析。

一个小小的 YAML 字段，影响不小。技能作者可以为正确的推理深度做设计，而不是指望用户碰巧把 effort 调对了。

## 限速，看得见了

`rate_limits` 是状态栏脚本可用的新字段。它暴露了 Claude.ai 的 5 小时和 7 天用量窗口，包含 `used_percentage` 和 `resets_at`。

如果你曾经在任务做到一半突然撞墙，现在有解了。状态栏可以显示类似 "5h 用量 73%，2h 后重置"。不用猜了。

这只适用于 Claude.ai 计费。用自己 API key 的用户有不同的限制。

## 插件，内联声明

`source: 'settings'` 是一个新的插件市场来源，让你直接在 settings.json 里声明插件条目。不需要注册表，不需要安装步骤。在设置文件里写好插件定义，就加载了。

对分发内部插件的团队来说，这简化了流程。推一下 settings.json 的变更，所有人就有了。

加上插件提示现在检测 CLI 工具使用（原来只看文件模式匹配），插件发现机制在变聪明。Claude Code 现在可以根据你在跑什么命令行工具来推荐插件，而不只是看项目里有什么文件。

## --resume 修复：比听起来重要得多

`--resume` 在恢复会话时丢失并行工具结果。如果之前的会话有并行工具调用，恢复后会显示 `[Tool result missing]` 占位符，而不是实际结果。

这比听起来严重。并行工具调用在复杂会话里很常见 -- agent 同时读多个文件、并行跑测试、同时搜索多个目录。丢失这些结果意味着 agent 丢失了它已经做过什么的上下文。它会重新读文件、重新搜索，或者更糟，在信息不完整的情况下做决定。

修了。所有 `tool_use`/`tool_result` 对现在正确恢复。

## 语音和 API 修复

语音模式的 WebSocket 连接失败，因为 Cloudflare 的机器人检测在拒绝非浏览器 TLS 指纹。Claude Code 的 WebSocket 客户端看起来不像浏览器，所以 Cloudflare 标记了它。通过调整 TLS 指纹处理修复。

精细工具流式传输通过 API 代理、Bedrock 或 Vertex 时返回 400 错误。如果你通过企业基础设施跑 Claude Code，这大概率打到你了。修了。

`/remote-control` 在网关和第三方提供商部署中出现在命令列表里，但在这些环境中它没法用。从这些上下文中移除了。

## 性能：大仓库省了 80MB

25 万文件规模的仓库，启动内存用量减少约 80MB。这是有实感的减少。在一台 16GB 内存的机器上跑多个 Claude Code 实例，省回来的加在一起大概半个 GB。

大 git 仓库里 `@` 文件补全也更快了。有几万文件的仓库里，输入 `@` 引用文件时以前有明显延迟。改善了。

## UI 打磨

几个导航和显示改进：

`/effort` 现在显示 "auto" 实际解析到什么级别，和状态栏指示器一致。以前选了 "auto" 完全不知道实际用的什么 effort。

`/permissions` 加上了正常的 Tab 和方向键列表内导航。加上 `/sandbox` 的 Tab/方向键修复，设置面板的键盘导航现在一致了。

后台任务面板支持左方向键从列表视图关闭。

插件安装提示从两步流程简化为单条 `/plugin install` 命令。

## 你可能不知道需要的设置修复

托管设置 -- `enabledPlugins`、`permissions.defaultMode`、策略设置的环境变量 -- 在 `remote-settings.json` 从上一次会话缓存后，启动时不会生效。如果你的组织集中管理 Claude Code 设置，这意味着新会话可能以过期配置启动。

这种 bug 难诊断，因为它是间歇性的。只有缓存的设置和当前的不同时才会触发。修了。

## 这个版本在说什么

`--channels` 是研究预览，但 Claude Code 的研究预览有个规律：通常几个版本内就变成稳定功能。发出来了就说明底层架构准备好了。

2.1.80 的模式是 Claude Code 在从工具变成基础设施。Channels 让它成为消息总线。Effort frontmatter 让技能自描述。Rate limit 可见性让它可观测。内联插件声明让它不依赖外部就能配置。

单拿出来每个都是好的改进。放在一起看，是在搭一个东西，看起来越来越不像 CLI，越来越像开发平台。

`claude update` 更新。如果你写 MCP 服务器，`--channels` 值得现在就试。如果你写技能，给它加上 `effort` frontmatter。如果你一直对限速毫无感知，配置你的状态栏。

[OpenClaw 小龙虾（点击跳转合集）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5Mzk1NzA1NA==&action=getalbum&album_id=4421270045515841537#wechat_redirect)

[s?__biz=MjM5Mzk1NzA1NA==&mid=2247490360&idx=1&sn=8da4f1607ebfe90943a04624d32e9454&scene=21](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490360&idx=1&sn=8da4f1607ebfe90943a04624d32e9454&scene=21#wechat_redirect)

[一个机器人，多个 Agent：OpenClaw Discord 频道级路由配置](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490330&idx=1&sn=a9c02cd0f39923a1c83cc03543884b5b&scene=21#wechat_redirect)

[OpenClaw + 飞书 + Scrum：用 AI Agent 团队跑完整个敏捷研发闭环](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490213&idx=1&sn=a7c53052d27b7f8acc50be4ba159d91c&scene=21#wechat_redirect)

[CTO 视角：OpenClaw 企业级多项目 AI Agent 架构怎么搭](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490192&idx=1&sn=270a654532997b41f7376dba2603bba0&scene=21#wechat_redirect)

[给 OpenClaw 接飞书机器人，三个坑让我查了一小时](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490187&idx=1&sn=33e91b2008944de070b341e0a9f4350c&scene=21#wechat_redirect)

[QQ 小龙虾🦞：sliverp/qqbot 和 tencent-connect/openclaw-qqbot 到底选哪个](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490120&idx=1&sn=b33c92b1beb6b4c340a330dda5c38794&scene=21#wechat_redirect)

[给 QQ 装个小龙虾🦞：官方openclaw-qqbot 实测，2条命令搞定，群聊踩坑记录](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490116&idx=1&sn=b17b29cd9909d83dac6efef222832c29&scene=21#wechat_redirect)

[OpenClaw v2026.3.11: WebSocket 劫持已修复, Ollama 正式集成, 记忆搜索支持图片和音频](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490141&idx=1&sn=c50c8843dd4ef1c85b77802d264ccd18&scene=21#wechat_redirect)

[局域网两台电脑跑 OpenClaw，'Allow device to connect?' 弹个没完？四条命令治好它](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490140&idx=1&sn=730f77926c9702b4c4ff5220a2810be4&scene=21#wechat_redirect)

[Windows 11 原生装 OpenClaw：PowerShell 一行搞定 QQ 机器人](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490062&idx=1&sn=c9a06cd3628573b4777be0f7b57d1240&scene=21#wechat_redirect)

[macOS 原生装 OpenClaw：一条命令接上 QQ 机器人](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490053&idx=1&sn=70addb46c600fb0aeb161b39e13e620a&scene=21#wechat_redirect)

[用 Docker 装 OpenClaw：一条命令，三个坑，一个能用的 AI 智能体](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490027&idx=1&sn=c8e15d5f546148fa773935c0d6c94b22&scene=21#wechat_redirect)

[OpenClaw Telegram Topics: 一个群组运行多条并行任务流](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489773&idx=1&sn=ddf3ab39335ef6961df948de6d781078&scene=21#wechat_redirect)

[Claude Agent SDK 系列（点击跳转合集）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5Mzk1NzA1NA==&action=getalbum&album_id=4074951080126136323#wechat_redirect)

[Claude Agent SDK 构建 AI Agent 实践：如何实现与上传文件的对话](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489575&idx=1&sn=31bd2ed68ba99d12c6ae16e3eea08f81&scene=21#wechat_redirect)

[Claude Agent SDK 构建 AI Agent 实践：服务端向 Claude Agent SDK 注入环境变量的实践](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489567&idx=1&sn=1b589f444911dcfb40e21b28b9ea2a14&scene=21#wechat_redirect)

[Claude Agent SDK + 微信小程序：AI Agent 项目实践复盘 2026-01-21](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489493&idx=1&sn=59f8dd794de6447e713298ae1f305330&scene=21#wechat_redirect)

[Claude Agent SDK + 微信小程序为个人打造 AI 分身：vs-ai-agents 项目技术实践复盘](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489492&idx=1&sn=dda6a2df2490d4c2ab6eff9fd4c9aeba&scene=21#wechat_redirect)

[BMAD AI 驱动敏捷开发系](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5Mzk1NzA1NA==&action=getalbum&album_id=4357253184793296905#wechat_redirect)[列（点击跳转合集](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5Mzk1NzA1NA==&action=getalbum&album_id=4357253184793296905#wechat_redirect)[）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5Mzk1NzA1NA==&action=getalbum&album_id=4357253184793296905#wechat_redirect)

[BMAD 6.2.0：推荐使用 bmad-product-brief-preview 基于 Prompt 的多 Agent 编排](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490361&idx=1&sn=33bde27731c4e26b9b9d4965a057f31a&scene=21#wechat_redirect)

[如何用 BMAD Quick Dev 在 10 分钟内把客户的一句话需求变成完整的可行性评估](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490329&idx=1&sn=616acdd3988a18cc3b3d9b84bec4b686&scene=21#wechat_redirect)

[Claude Code + BMAD Quick Dev + YOLO：AI 自主修复缺陷的完整闭环实践](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490287&idx=1&sn=4824845eaba5daa36fe496caca0de5ec&scene=21#wechat_redirect)

[BMad v6.1.0 用统一的Skill技能架构替代了旧的工作流引擎](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490227&idx=1&sn=55ddd573b13fa93fb79abf8bf447356a&scene=21#wechat_redirect)

[当 BMAD 开发工作流遇上 PPT 周报生成：BMAD Quick Dev 的边界拓展](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490214&idx=1&sn=f13267797f593ed2e27739afb39a0271&scene=21#wechat_redirect)

[Claude Code + BMAD + YOLO 模式：一个 Session 搞定全栈功能开发](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490152&idx=1&sn=62d93211cdb201ad7f4724ca2d10adb4&scene=21#wechat_redirect)

[BMad v6.0.4 + GDS v0.1.10：边缘用例猎手、多智能体测试和引擎知识库](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489806&idx=1&sn=0d00d1ff74b03e1c64b06d89463a3412&scene=21#wechat_redirect)

[BMAD v6.0.4：从 Beta 到正式版，两分钟搞定](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489793&idx=1&sn=2a3eb5bf21d03ea036161574985c3e5f&scene=21#wechat_redirect)

[BMAD v6.0.0-beta-8 安装实战：从零开始搭建你的 AI 开发团队](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489769&idx=1&sn=58e722d9efdc488203fd9c1efae87d62&scene=21#wechat_redirect)

[BMAD + Ralph 执行循环：Claude Code 的统一 AI 开发框架](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489763&idx=1&sn=3230665edbfcc6e4855c6f9f8e2687c7&scene=21#wechat_redirect)

[BMAD 最佳实践:AI 驱动的敏捷开发指南](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489751&idx=1&sn=8909904d4f14ba5ada08588952aeb2bc&scene=21#wechat_redirect)

[BMAD 突破性 AI 驱动敏捷开发框架：v6.0.0-alpha.23 升级体验：全新安装之旅](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489133&idx=1&sn=4973d5907ef08bb9ebd67a6eeda8f203&scene=21#wechat_redirect)

[BMAD Method 入门指南：用 Quick Dev 工作流更快、更稳地交付](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489101&idx=1&sn=f78aaea74ba3db553fbca71e83abe868&scene=21#wechat_redirect)

[实战测评：用 Claude Code + BMAD + GLM-4.7 打造 HeartPetBond App (心宠纽带)](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487840&idx=1&sn=53acb782e5b9eb1df9744f31745e60ab&scene=21#wechat_redirect)

[BMAD V6 安装配置完全指南：项目目录安装最佳实践](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486789&idx=1&sn=7bbfe9ef92964ef9be4c722b8d357991&scene=21#wechat_redirect)

[BMAD v6 安装更新：模块化 + AgentVibes “会说话”的开发体验](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486758&idx=1&sn=5d88d95112b626015377effd19227384&scene=21#wechat_redirect)

[用 Claude Code + BMAD AI 驱动敏捷，把一个想法变成 省钱思维 (MoneyMind) App](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486723&idx=1&sn=45dddd81a5d856ced875b4769cc54d09&scene=21#wechat_redirect)

[AI 时代的"文档屎山"？BMAD、Spec-Kit、OpenSpec 等面向文档AI编程的利弊](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486451&idx=1&sn=eeba9d266868b49759d855277e323562&scene=21#wechat_redirect)

[在 Codex 里像 Claude Code 一样用 BMAD：把多角色 AI 团队装进你的仓库](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486257&idx=1&sn=436d00899f6adcd45c962b47bdfad04e&scene=21#wechat_redirect)

[BMAD 突破性 AI 驱动敏捷开发框架：深度解析 26 个代理、68 个工作流和 655 个文件](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489141&idx=1&sn=1aa0e66547dcf30940b30b7132308271&scene=21#wechat_redirect)

[AI 自主开发 App 成功上架：历时 14 天审核，MoneyMind 省钱思维 App 今天发布了](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247488720&idx=1&sn=e5b4a2165194be07f642e5cadf10dc28&scene=21#wechat_redirect)

[MoneyMind 省钱思维 App 审核又被拒：粗心提交错误版本的惨痛教训](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487799&idx=1&sn=0e00268ba1aea481407848d1ea0e494b&scene=21#wechat_redirect)

[被苹果审核拒绝不要怕：这次用 Google Antigravity AI 快速修复 App Store 审核问题](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487193&idx=1&sn=18e5dee00ad05d815f5106d8ac7ae0c4&scene=21#wechat_redirect)

[Claude Code 自主开发 MoneyMind（省钱思维）iOS 应用送审 App Store](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487054&idx=1&sn=3d609b27f0c79c67b752b3645f6addc0&scene=21#wechat_redirect)

[用 Claude Code + BMAD AI 驱动敏捷，把一个想法变成 省钱思维 (MoneyMind) App](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486723&idx=1&sn=45dddd81a5d856ced875b4769cc54d09&scene=21#wechat_redirect)

[全网首发？第一款 GLM 4.7 + Claude Code AI 自主开发的心宠纽带 App 首次通过 App Store 审核并上架发布](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489028&idx=1&sn=ade948ba611d1df9ae3c15cd9f692797&scene=21#wechat_redirect)

[智谱 GLM 4.7 模型 AI 自主开发 HeartBetBond 心宠纽带 App，从想法到提交 App Store 仅用 12 天](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247488932&idx=1&sn=8ace2b6e8d016d0628ca54c208e0e889&scene=21#wechat_redirect)

[实战测评：用 Claude Code + BMAD + GLM-4.7 打造 HeartPetBond App (心宠纽带)](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487840&idx=1&sn=53acb782e5b9eb1df9744f31745e60ab&scene=21#wechat_redirect)

## 加入 AI灵感闪现 微信群

长按下图二维码进入 AI灵感闪现 微信群

长按下图二维码添加微信好友 VibeSparking 加群

## 关注 AI灵感闪现 微信公众号