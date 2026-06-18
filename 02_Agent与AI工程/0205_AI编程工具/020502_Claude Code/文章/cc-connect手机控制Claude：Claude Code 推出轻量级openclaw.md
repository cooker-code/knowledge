---
title: cc-connect手机控制Claude：Claude Code 推出轻量级openclaw
author: 顶尖架构技术栈
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwNzUzODAzNQ==&mid=2247488570&idx=1&sn=4a8094c2137f1660cb8353630c1e72af&chksm=c15ef2ee069937f48ad965058fca30a9bbf695ca8382a015cd23dd50df0a234a4d13687a2d99&mpshare=1&scene=24&srcid=0310uJeYFEnYCIgXyI6ofQW6&sharer_shareinfo=f575f3b0e6b82f0631644d7c93016a0b&sharer_shareinfo_first=f575f3b0e6b82f0631644d7c93016a0b#rd
---

大家好，我是**易安**，AI超级个体，大厂程序员二孩奶爸。

# cc-connect手机控制Claude：Claude Code 推出轻量级openclaw

## 写在前面

有没有这样的场景：你不在电脑旁，但脑子里突然有个想法想让 AI 帮你跑一跑——审查一段代码、搜一下技术资料、生成一份报告。但 Claude Code 这类工具只能在终端里用，手机上完全没法操作。

等你回到电脑前，思路可能早就散了。

开源工具 cc-connect 解决的就是这个问题。它把运行在你开发机上的 AI Agent 桥接到飞书、钉钉、Telegram 等日常聊天工具——从此在手机上发条消息，Claude Code 就能在远端帮你干活，结果直接推送回来。

更不寻常的是：它支持 7 个主流 AI Agent，9 个聊天平台，大部分平台不需要公网 IP，一个进程可以同时管理多个项目，甚至能在群聊里同时绑定多个 AI 机器人让它们互相协作。

---

## Claude Code 的能力越来越强，但一直把你钉在电脑前

Claude Code 这类 AI Agent 能做的事越来越多——读写文件、执行命令、跨文件重构、自主调试——本质上已经是一个能在你的项目里独立工作的"数字同事"。

但有一个问题：它只能在终端里跑。

你出门开会，它在电脑上等着。你睡觉，它在电脑上等着。哪怕你只是想让它帮你整理一份文档、跑一个脚本，也得先坐到电脑前打开终端。

真正发挥 AI Agent 价值的使用方式，应该是：你在任何地方，它在帮你干活。

cc-connect 做的就是这件事——把 Claude Code 接入你随时都在用的聊天工具，让它在你的开发机上后台运行，你随时用消息指挥它，结果推回到聊天窗口。

---

## cc-connect 是怎么工作的

架构很清晰：

```
         你（手机 / 电脑 / 平板）  
                    │  
    ┌───────────────┼───────────────┐  
    ▼               ▼               ▼  
   飞书           Slack         Telegram  ...9 个平台  
    │               │               │  
    └───────────────┼───────────────┘  
                    ▼  
              ┌────────────┐  
              │ cc-connect │  ← 你的开发机  
              └────────────┘  
              ┌─────┼─────┐  
              ▼     ▼     ▼  
         Claude  Gemini  Codex  ...7 个 Agent  
          Code    CLI   OpenCode / iFlow
```

cc-connect 跑在你的开发机上，同时监听聊天平台的消息和 AI Agent 的输出。你在飞书发一句"帮我看看 src/api.ts 有没有安全漏洞"，它收到消息转给 Claude Code，Claude Code 干完活把结果发回聊天窗口。

---

## 支持哪些 Agent 和平台

**当前已支持的 AI Agent：**

|  |
| --- |
|  |

| Agent | 状态 |
| --- | --- |
| Claude Code | ✅ |
| Codex (OpenAI) | ✅ |
| Cursor Agent | ✅ |
| Gemini CLI (Google) | ✅ |
| Qoder CLI | ✅ |
| OpenCode | ✅ |
| iFlow CLI | ✅ |
| Goose / Aider | 🔜 计划中 |
| Kimi Code / GLM Code | 🔭 探索中 |

**聊天平台：**

|  |
| --- |
|  |

| 平台 | 连接方式 | 需要公网 IP |
| --- | --- | --- |
| 飞书 | WebSocket 长连接 | 不需要 |
| 钉钉 | Stream 模式 | 不需要 |
| Telegram | Long Polling | 不需要 |
| Slack | Socket Mode | 不需要 |
| Discord | Gateway | 不需要 |
| 企业微信 | Webhook | 需要 |
| LINE | Webhook | 需要 |
| QQ（NapCat/OneBot） | WebSocket | 不需要 |
| QQ 官方机器人 | WebSocket | 不需要 |

大部分国内常用平台（飞书、钉钉、QQ）都是 WebSocket 长连接，不需要公网 IP，部署门槛很低。

---

## 核心功能详解

### 权限模式：手机也能切换

不同场景需要不同的 Agent 控制粒度。cc-connect 支持通过斜杠命令在聊天中直接切换：

```
/mode          # 查看当前模式  
/mode yolo     # 切换到全自动（所有工具调用自动通过）  
/mode default  # 切换回手动确认
```

以 Claude Code 为例：

|  |
| --- |
|  |

| 模式 | 行为 |
| --- | --- |
| `default` | 每次工具调用需要你确认 |
| `acceptEdits` | 文件编辑自动通过，其他需确认 |
| `plan` | 只做规划不执行，审批后再跑 |
| `bypassPermissions` （yolo） | 全自动，适合可信环境 |

在手机上发 `/mode yolo`，Claude Code 就进入全自动模式——不需要你逐步确认，适合你想让它在后台自主完成一批任务的场景。

### 定时任务：自然语言创建 cron

这是最日常的用法之一。直接跟 Claude Code 说：

> "每天早上 6 点帮我总结 GitHub trending"

Claude Code 会自动把这句话转成 `cc-connect cron add` 命令，创建定时任务，每天早上把结果推送到你的聊天窗口。

也可以手动创建：

```
/cron add 0 6 * * * 帮我收集 GitHub trending 并发送总结  
/cron list  
/cron del <id>
```

定时任务在 cc-connect 后台静默运行，不需要你守着。

### API Provider 管理：运行时切换，不重启

如果你同时使用多个 API 服务（Anthropic 直连、中转服务、AWS Bedrock 等），可以在聊天中随时切换，不需要重启 cc-connect：

```
/provider list              # 列出所有配置的 Provider  
/provider switch relay      # 切换到中转 Provider
```

配置方式：

```
[[projects.agent.providers]]  
name = "anthropic"  
api_key = "sk-ant-xxx"  
  
[[projects.agent.providers]]  
name = "relay"  
api_key = "sk-xxx"  
base_url = "https://api.relay-service.com"  
model = "claude-sonnet-4-20250514"
```

切换时自动重启 Agent 会话并加载新凭证，凭证通过环境变量注入子进程，不会写入本地配置文件。

### 多机器人中继：同一个群聊，多个 Agent 协作

可以在一个群聊里绑定多个 AI 机器人，让它们各自响应或相互协作：

```
/bind claudecode    # 绑定 Claude Code 机器人  
/bind gemini        # 绑定 Gemini CLI 机器人
```

绑定后，可以让 Claude Code 审查代码，再 @Gemini 提供第二意见——同一个群聊，不需要来回切换。

### Claude Code Router 集成

支持与 Claude Code Router 集成，根据任务类型自动路由到不同模型：

```
你：帮我重构这段代码       →  Router → DeepSeek（默认）  
你：思考一下这个架构决策   →  Router → DeepSeek Reasoner（思考模型）  
你：分析这个大型代码库     →  Router → Gemini Pro（长上下文）
```

配置 `router_url` 后，cc-connect 自动设置 `ANTHROPIC_BASE_URL` 指向 Router，其余对 Claude Code 透明。

### 语音消息直接发给 Agent

支持在飞书、Telegram、企业微信等平台直接发语音——cc-connect 调用 Whisper API（OpenAI 或 Groq）转录成文字，再转给 Agent 处理。需要安装 ffmpeg 做音频格式转换。

### 守护进程模式

```
cc-connect daemon install --config ~/.cc-connect/config.toml  
cc-connect daemon start  
cc-connect daemon status  
cc-connect daemon logs -f
```

支持 Linux systemd 和 macOS launchd，日志自动轮转，开机自启。

---

## 快速上手

**安装：**

```
npm install -g cc-connect
```

或者直接让 Claude Code 帮你安装——把这句话发给它：

```
请参考 https://raw.githubusercontent.com/chenhg5/cc-connect/refs/heads/main/INSTALL.md 帮我安装和配置 cc-connect
```

**配置（以飞书为例）：**

```
[[projects]]  
name = "my-backend"  
  
[projects.agent]  
type = "claudecode"  
  
[projects.agent.options]  
work_dir = "/path/to/project"  
mode = "default"  
  
[[projects.platforms]]  
type = "feishu"  
  
[projects.platforms.options]  
app_id = "cli_xxxx"  
app_secret = "xxxx"
```

**运行：**

```
cc-connect
```

各平台机器人创建步骤详见官方文档。

---

## Claude Code 是什么？

对刚接触的人来说，Claude Code 是 Anthropic 开发的 AI 编程 Agent。不是 Copilot 那种补全工具，是能在你的代码库里独立工作的 Agent——读写文件、执行 shell 命令、跨文件重构、自主调试，能完成完整的开发任务。

官方订阅方式：

|  |
| --- |
|  |

| 套餐 | 价格 | 适用场景 |
| --- | --- | --- |
| Claude Pro | $20/月 | 支持 Claude Code，有用量上限 |
| Claude Max | $100/月 | 更高用量，适合重度使用 |
| Anthropic API | 按 token 计费 | Sonnet 4.6：15 per 1M tokens |

不过对国内用户来说，官方订阅需要海外信用卡，网络环境也要折腾。如果嫌麻烦，可以看看 Code80，真实订阅账号转 API，换个 endpoint 就能在 cc-connect 或任何支持 Claude Code 的工具里直接用，体验跟官方一致。详情到官网了解：code.ai80.vip

---

## 常见问题

**Q：cc-connect 和 Claude Code /loop 有什么区别？**

A：两者互补。`/loop` 是让 Claude Code 本地持续循环执行任务；cc-connect 是让你从聊天 App 远程发起和控制任务。可以同时用——用 cc-connect 在手机上下达任务，用 `/loop` 让它循环跑完。

**Q：用飞书接入需要公网 IP 吗？**

A：不需要。飞书用的是 WebSocket 长连接，cc-connect 主动连飞书服务器，不需要外部访问你的机器。同样不需要公网 IP 的还有：钉钉、Telegram、Slack、Discord、QQ。需要公网 IP 的只有企业微信和 LINE（Webhook 模式）。

**Q：多个项目能同时管理吗？**

A：可以。一个 cc-connect 进程支持多个 `[[projects]]` 配置，每个项目对应独立的 Agent + 平台组合。比如后端项目用 Claude Code + 飞书，前端项目用 Codex + Telegram，同时跑。

**Q：支持切换到其他 AI Agent（比如 Gemini CLI）吗？**

A：支持。修改 `config.toml` 中的 `type` 字段即可，可选值：`claudecode`、`codex`、`cursor`、`gemini`、`qoder`、`opencode`、`iflow`，不需要重装。

**Q：国内开发者怎么接入 Claude Code？**

A：官方方式需要海外支付和网络。国内可以通过 Code80 来使用，在 cc-connect 的 `providers` 配置中填入对应的 `api_key` 和 `base_url`，切换 Provider 即可正常调用 Claude Code。

---

## 关于 code80

写这篇文章的时候想到一个问题——上面这些自定义指令、工作流配置、部署脚本，对我来说是近一年迭代的结果，但对刚接触 AI 编程的人来说门槛不低。

这也是我做 **code80****AI编程巴士** 的原因之一。**code80** 上会逐步整理这些工具和工作流的教程，包括可以直接复制使用的指令模板。如果你对 Claude Code 的自定义指令感兴趣，可以关注一下。

## 写在最后

Vibe Coding 不是拍脑袋写 prompt，而是**用工程化的思维管理 AI 编程的流程**。自定义指令是这个流程的骨架：

* • `/commit` 标准化了提交流程
* • `/upstream` 让分支同步和冲突处理变成了两分钟的事
* • `/progress-save` + `/progress-load` 解决了上下文断裂的问题
* • `/deploy` 把手动部署变成了一键操作
* • `/gitsync` 让多项目之间的代码同步不再遗漏
* • `/review` 和 `/bug-add` 保证了质量和经验积累
* • `/parallel-epic` 实现了多 Agent 并行开发

这些指令本身都是 markdown 文件，语法简单，十分钟就能写一个。但组合起来的效果是，你可以把精力集中在"要做什么"上，"怎么做"交给 Claude。

如果你也在用 AI 编程，欢迎交流，**微信：20133213**可以找到我。

**易安致力于为高T提供稳定可靠的纯血Claude,GPT,Gemini 模型服务，节省你们的时间，平均才0.5-0.6元一刀，而且是纯血帐号，无逆向，无倍率，性价比拉满。**