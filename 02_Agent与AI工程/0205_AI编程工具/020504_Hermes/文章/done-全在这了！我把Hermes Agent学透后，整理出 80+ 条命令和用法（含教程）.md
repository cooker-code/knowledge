> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: 全在这了！我把Hermes Agent学透后，整理出 80+ 条命令和用法（含教程）
author: AI立志传
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzNTE4ODc0OA==&mid=2247492390&idx=1&sn=0ab53c627800f9ca3aac72564e9b4cac&chksm=c33614ac59fb51812df421695f559e4858e7f01123e7d9ebd944a6ec9b294c0c3373789268ee&mpshare=1&scene=24&srcid=04232hS545bF4eCjMn7xsUjH&sharer_shareinfo=6f22f4098486a5a4d4a5a65443fda53a&sharer_shareinfo_first=6f22f4098486a5a4d4a5a65443fda53a#rd
---

— — —

```
# 新用户，一行装好
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 装完重载一下 shell
source ~/.bashrc   # zsh 用户换成 source ~/.zshrc

# 老用户，直接更新
hermes update
```

大家好，我是米汤！

下面的命令需要在终端里运行。

Mac 用户打开「终端」就行，Windows 用户需要先装 WSL2，装好之后打开 Ubuntu 终端再操作，文末避坑部分有详细步骤。

上篇介绍了 Hermes Agent 是什么以及如何接入个人微信👉[Hermes Agent可接入微信了！Github 55K Star，24小时不间断工作，和小龙虾搭伙当Agent牛马...（含教程）](https://mp.weixin.qq.com/s?__biz=MzkzNTE4ODc0OA==&mid=2247492375&idx=1&sn=154960aff27e1ee74cff387031ac4d3d&scene=21#wechat_redirect)

那么，除了和hermes聊天，还能干啥？

命令那么多，从哪里开始？

国内的模型能接吗？

这篇就把我使用AI辅助进行翻文档、翻源码整理出来的东西，

一次性全说清楚。

— — —

▍ 有多少东西

先说一下这篇的信息量。

Hermes 能接的AI模型，国内国外加起来超过 20 个，DeepSeek、Kimi、通义千问、智谱 GLM、MiniMax 全都支持，每个怎么配我都写了。

光命令就有 80 多条。

顶层的 CLI 命令 30 多个，进入聊天之后还有 50 多条斜杠指令，本文按「你想做什么」帮你分好类了，不用全记，收藏备查就好。

它有个技能商店，里面 644 个插件，覆盖 GitHub、写作、研究、机器学习等 16 个方向，一行命令装好。

它不是一个聊天框，是 9 个模块拼起来的完整系统，能说话、能自动化、能接浏览器、能定时跑任务。

我还翻了源码，整理出 10 条「为什么这样设计」的洞见，

放在文末，感兴趣的可以直接跳过去看。

— — —

▍ 第一类：启动和会话管理

最基础的一类，刚装好就会用到。

**你想启动 Hermes，就这个👇**

```
hermes
# 或者
hermes chat
```

**你想继续上次没聊完的对话👇**

```
hermes --continue
# 简写
hermes -c
```

退出的时候 Hermes 会打印一行恢复命令，下次直接复制粘贴就行。

**你想启动时就指定模型、工具集、技能👇**

新手跳过这块也没关系，进聊天之后用 `/model` 随时切换就行。这里是给想用命令行参数的朋友备查的。

```
# 指定模型
hermes chat -m deepseek/deepseek-chat

# 指定工具集
hermes chat -t web,debugging

# 预加载技能
hermes chat -s github-pr-workflow

# 单次提问，不进入聊天界面
hermes chat -q "帮我写一个 Python 爬虫"

# 安静模式，适合脚本调用，只输出最终回答
hermes chat -q "今天天气怎么样" --quiet

# 限制每轮最多工具调用次数（默认 90）
hermes chat --max-turns 30

# 在独立 Git worktree 里跑，适合多个 Agent 同时改同一个仓库
hermes chat --worktree

# 开启文件系统检查点，改坏了可以 /rollback 回来
hermes chat --checkpoints
```

**你想按名字找回某个会话👇**

```
hermes --resume 会话名
```

**你想管理历史会话，有一整套子命令👇**

```
hermes sessions list      # 列出所有历史会话
hermes sessions browse    # 交互式翻找，像文件管理器
hermes sessions rename ID 新名字   # 给会话改个好记的名字
hermes sessions export 文件名      # 导出成 JSONL 文件
hermes sessions delete ID          # 删掉某个会话
hermes sessions prune              # 批量清理旧会话
hermes sessions stats              # 看存储统计
```

有一个细节值得说。

Hermes 的所有对话都自动存在本地，格式是 SQLite 加 Markdown 双份，不会丢。

**好用的快捷键👇**

`Alt+Enter` — 多行输入换行，粘贴代码或写长提示词的时候用

Hermes 启动后的欢迎横幅

— — —

▍ 第二类：配置和模型切换

这类命令解决一个问题，随时换模型、改配置，不用重装。

**你想换一个模型👇**

```
hermes model
```

会弹出交互式菜单，上下键选，回车确认，10 秒搞定。

**国内可以用的模型，重点说一下👇**

| 提供商 | 支持的模型 | 配置方式 |
| --- | --- | --- |
| ------- | --------- | --------- |
| DeepSeek | DeepSeek-V3、R1 等 | 设置  `DEEPSEEK_API_KEY` |
| Kimi / Moonshot | 编码和聊天模型 | 设置 `KIMI_API_KEY` |
| 通义千问 / 阿里云 | Qwen 系列 | 设置  `DASHSCOPE_API_KEY` |
| 智谱 GLM / Z.AI | GLM 系列 | 设置 `GLM_API_KEY` |
| MiniMax | 国际端点 | 设置  `MINIMAX_API_KEY` |
| MiniMax 国内 | 国内端点 | 设置  `MINIMAX_CN_API_KEY` |

**国外提供商也都支持👇**

| 提供商 | 说明 | 配置方式 |
| --- | --- | --- |
| ------- | ----- | --------- |
| Nous Portal | 订阅制，零配置直接用 | `hermes login` |
| Anthropic | Claude 系列 | `hermes model` 认证或设置 API Key |
| OpenAI Codex | GPT 系列 | `hermes model` 设备码认证 |
| OpenRouter | 多提供商路由，一个 Key 用所有模型 | 设置 API Key |
| GitHub Copilot | GPT-5.x、Claude 等 | OAuth 或设置  `COPILOT_GITHUB_TOKEN` |
| Hugging Face | Qwen、DeepSeek 等 20+ 开源模型 | 设置 `HF_TOKEN` |
| Vercel AI Gateway | Vercel 路由 | 设置  `AI_GATEWAY_API_KEY` |

**自己有本地模型或私有端点也能接👇**

Ollama、vLLM、SGLang 这类 OpenAI 兼容的 API 都行，`hermes model` 里选「Custom Endpoint」，填 base URL 和 API Key，上下文窗口大小会自动检测。

不知道怎么设置环境变量的，最简单的方式是在 `~/.hermes/.env` 文件里加一行👇

```
DEEPSEEK_API_KEY=你的key
```

或者装好 Hermes 之后，直接告诉它「帮我配置 DeepSeek」，它会引导你一步步填好。

**你想看或者改配置👇**

```
hermes config              # 看当前所有配置
hermes config edit         # 用编辑器打开 config.yaml
hermes config set KEY VAL  # 改某一项，比如 hermes config set terminal.backend docker
hermes config path         # 看配置文件在哪
hermes config check        # 检查有没有缺失或过期的配置
hermes config migrate      # 升级配置格式
```

**你想走一遍完整的配置向导👇**

```
hermes setup
# 也可以只配某一块
hermes setup model
hermes setup gateway
hermes setup tools
```

**你想检查环境有没有问题👇**

```
hermes doctor
# 加 --fix 让它尝试自动修复
hermes doctor --fix
```

**其他常用的👇**

```
hermes status        # 看所有组件状态
hermes version       # 看版本号
hermes update        # 更新到最新版
hermes login         # OAuth 登录（Nous Portal、OpenAI Codex 等）
hermes logout        # 清除登录状态
```

**凭证池管理，多账号轮换用👇**

如果你有多个 API Key 想轮换使用，或者某个 Key 配额用完了自动切下一个，用这个。

普通用户一个 Key 就够，这块可以跳过。

```
hermes auth add              # 交互式添加凭证
hermes auth list             # 看凭证池里有哪些
hermes auth remove P INDEX   # 删掉某个
hermes auth reset PROVIDER   # 重置「配额耗尽」状态
```

**插件管理👇**

插件和技能不一样，插件是扩展 Hermes 底层能力的（比如接入新的记忆服务），技能是教它怎么做事的。

大多数人用不到插件，这块备查就好。

```
hermes plugins list      # 看已装的插件
hermes plugins install   # 装插件
hermes plugins remove    # 删插件
```

**外部记忆服务配置👇**

Hermes 默认把记忆存在本地文件里，如果你想接 Honcho（用户建模）或 Mem0（向量记忆）这类外部服务，用这个。

```
hermes memory setup    # 配置外部记忆服务
hermes memory status   # 看状态
hermes memory off      # 关掉外部记忆，回到本地
```

**MCP 服务器管理👇**

MCP（Model Context Protocol）是一个开放协议，让 Hermes 能接入外部工具服务器，比如 GitHub、文件系统、数据库等。

```
hermes mcp add NAME --url URL       # 添加一个 HTTP 类型的 MCP 服务器
hermes mcp add NAME --command CMD   # 添加一个命令行类型的
hermes mcp list                     # 看已配置的服务器
hermes mcp test NAME                # 测试连接
hermes mcp configure NAME           # 配置该服务器暴露哪些工具
hermes mcp remove NAME              # 删掉
hermes mcp serve                    # 把 Hermes 本身作为 MCP 服务器运行
```

配置文件里的写法长这样👇

```
# ~/.hermes/config.yaml
mcp_servers:
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "ghp_xxx"
```

**多实例隔离运行（Profiles）👇**

你想同时跑一个「写作助手」和一个「代码助手」，各自有独立的配置、记忆、技能，互不干扰，用 profile 隔离。刚开始用一个就够，等用熟了再考虑这个。

```
hermes profile list              # 看所有 profile
hermes profile create NAME       # 创建新 profile
hermes profile use NAME          # 设为默认
hermes profile show NAME         # 看详情
hermes profile rename A B        # 改名
hermes profile delete NAME       # 删掉
hermes profile export NAME       # 导出成 tar.gz
hermes profile import FILE       # 从压缩包导入
```

每个 profile 用 `HERMES_HOME` 环境变量隔离，独立的 config、记忆、技能、cron，互不干扰。

**日志和诊断👇**

```
hermes logs              # 看日志
hermes dump              # 导出安装和配置摘要，排错或提交 issue 时用
hermes insights          # 看使用统计和分析
hermes insights --days 7 # 只看最近 7 天
```

**Shell 自动补全👇**

装好之后可以开启命令自动补全，输入 `hermes` 按 Tab 就能提示子命令。把下面这行复制到终端运行一次就行，以后每次打开终端都会自动生效。

```
hermes completion bash >> ~/.bashrc   # bash 用户（大多数 Linux / WSL2）
hermes completion zsh >> ~/.zshrc     # zsh 用户（大多数 Mac）
```

运行完之后关掉终端重新打开，或者运行 `source ~/.bashrc` 让它立即生效。

**从 OpenClaw 迁移过来👇**

之前用过 OpenClaw 的朋友，Hermes 提供了迁移工具。

```
hermes claw migrate
```

**卸载👇**

```
hermes uninstall
```

— — —

▍ 第三类：技能和工具管理

这是 Hermes 最有意思的地方，也是和其他 AI 工具差距最大的地方。

Skills技能包是什么？

简单说，就是一个 Markdown 文件，里面写着「遇到这类任务，按这个流程做」。

Hermes 会在合适的时候自动加载对应的技能，不用你每次都重新解释。

更厉害的是，它会自己创建技能。

你让它做了一件复杂的事，用了 5 个以上的工具调用，它会自动把这个流程保存成技能，下次遇到类似的任务直接用，不用重新摸索。

**技能还支持条件激活。**

比如这个技能只在 Discord 里加载，或者只在当前目录有 `package.json` 的时候才激活，写在技能文件的头部就行👇

```
---
name: deploy-to-vercel
description: 把 Next.js 项目部署到 Vercel
platforms: [cli]
conditions: [file_exists:vercel.json]
---
```

`platforms` 控制在哪个平台加载，`conditions` 控制在什么情况下激活，不满足条件的技能不会出现在提示里，不占系统提示空间。

**自己写一个技能也很简单👇**

```
mkdir -p ~/.hermes/skills/my-custom/
```

然后在里面新建一个 `.md` 文件，写上 YAML 头部和正文步骤就行。写完重启 Hermes，或者等它自动刷新，就能用 `/技能名` 调用了。

**技能商店现在有多少东西👇**

644 个技能，4 个注册表，16 个分类。

Skills Hub 页面截图

代表性的分类举几个：

| 分类 | 代表技能 |
| --- | --- |
| ----- | --------- |
| AI Agents | claude-code、codex、hermes-agent、opencode |
| GitHub | github-pr-workflow、github-code-review、codebase-inspection、github-issues |
| MLOps | axolotl、dspy、gguf-quantization、unsloth、whisper、stable-diffusion、peft-fine-tuning |
| Productivity | notion、linear、google-workspace、ocr、powerpoint、nano-pdf |
| Research | arxiv、blogwatcher、llm-wiki、polymarket |
| Creative | ascii-art、manim-video、excalidraw、songwriting、p5js、ascii-video |
| Apple macOS | apple-notes、apple-reminders、findmy、imessage |
| Media | gif-search、youtube-content、heartmula、songsee |
| Gaming | minecraft-modpack-server、pokemon-player |
| DevOps | webhook-subscriptions |
| Data Science | jupyter-live-kernel |
| Note-taking | obsidian |
| Social Media | xitter |
| Smart Home | openhue |
| MCP | mcporter、native-mcp |
| Software Dev | plan、requesting-code-review、subagent-driven-development |

**你想找技能👇**

```
hermes skills browse  # 浏览所有技能
hermes skills search kubernetes # 搜索关键词
hermes skills search react --source skills-sh # 搜索社区目录
```

**你想装技能👇**

```
hermes skills install official/security/1password
hermes skills install skills-sh/vercel-labs/json-render/json-render-react --force
# --force 只在你审查过第三方技能之后再用
```

**其他技能管理命令👇**

```
hermes skills list       # 看已装的技能
hermes skills inspect ID # 先预览，不安装
hermes skills update     # 更新过时技能
hermes skills uninstall  # 卸载
hermes skills publish    # 发布自己写的技能
```

**工具管理👇**

```
hermes tools             # 交互式启用/禁用工具
hermes tools list        # 看所有工具和状态
hermes tools enable NAME
hermes tools disable NAME
hermes toolsets          # 看工具集分组
```

— — —

▍ 第四类：消息平台和定时任务

很多人不知道 Hermes 能接消息平台，

这是它和大多数 AI 工具最不一样的地方。

装好之后，你可以在 Telegram 里直接和它说话，它在服务器上跑，你不用开电脑。

支持的平台有 15 个以上，微信、Discord、飞书、钉钉、企微……

**你想接微信👇**

```
hermes gateway setup    # 交互式配置，跟着提示走
hermes gateway run      # 前台启动
hermes gateway install  # 安装成后台服务
hermes gateway start    # 启动服务
hermes gateway stop     # 停止
hermes gateway restart  # 重启
hermes gateway status   # 看状态
```

**你想让 Hermes 定时跑任务👇**

比如每天早上 9 点抓一下 Hacker News 的 AI 新闻，整理成摘要发到微信或飞书。

```
hermes cron create "09:00"   # 创建定时任务，跟着提示填提示词和投递平台
hermes cron list             # 看所有任务
hermes cron edit ID          # 改时间、提示词、投递方式
hermes cron pause ID         # 暂停
hermes cron resume ID        # 恢复
hermes cron run ID           # 立即触发一次
hermes cron remove ID        # 删除
hermes cron status           # 看调度器状态
```

时间格式很灵活，`'30m'`、`'every 2h'`、`'0 9  *'` 都行。

**你想接 Webhook👇**

```
hermes webhook subscribe 名字   # 创建一个 /webhooks/<名字> 路由
hermes webhook list
hermes webhook test 名字        # 发一次测试请求
hermes webhook remove 名字
```

— — —

▍ 语音模式

Hermes 支持语音输入和语音播报，需要单独安装。

```
pip install "hermes-agent[voice]"

# 可选，装了之后语音转文字在本地跑，不用联网
pip install faster-whisper
```

装好之后，进入聊天，用斜杠指令控制👇

```
/voice on    # 开启语音输入，按 Ctrl+B 开始录音
/voice tts   # 让 Hermes 朗读回复
/voice off   # 关掉
/voice status # 看当前状态
```

tg 和 Discord 里也支持语音，Discord 还能接语音频道，直接在频道里说话。

— — —

▍ 接入 IDE（ACP 编辑器集成）

这块是给用 VS Code、Zed、JetBrains 写代码的朋友的，不写代码的可以跳过。

Hermes 可以作为编辑器的 AI 助手运行，用的是 ACP 协议（Agent Client Protocol，可以理解成编辑器和 AI 之间的通用接口）。

```
pip install -e '.[acp]'
hermes acp
```

装好之后在编辑器里配置 ACP 服务器地址就行，和 Cursor、Copilot 的接入方式类似。

— — —

▍ 自定义 Agent 人格（SOUL.md）

Hermes 的默认语气和风格可以通过 `SOUL.md` 文件定义。

文件放在 `~/.hermes/SOUL.md`，每条消息发出前都会重新读取，改完不用重启。

写法很自由，比如👇

```
你是一个说话直接、偶尔用颜文字的助手。
你像一个什么都懂的朋友，不是一个客服机器人。
```

每个 profile 可以有自己的 SOUL.md，不同场景用不同人格。

— — —

▍ 第五类：会话内斜杠指令

进入 Hermes 聊天之后，输入 `/` 会弹出自动补全菜单。

这些是日常最高频的，按类整理好了。

**会话控制👇**

| 指令 | 说人话 | 频率 |
| --- | --- | --- |
| ----- | ------- | ----- |
| `/new` | 开一个全新对话 | 高 |
| `/retry` | 重试上一条，删掉上次回复重新发 | 高 |
| `/undo` | 撤销上一轮，连你的消息一起删 | 高 |
| `/history` | 看当前会话的完整对话历史 | 中 |
| `/compress` | 手动压缩上下文，省 token | 中 |
| `/background 提示词` | 把任务扔到后台跑，不打断当前对话 | 中 |
| `/btw 问题` | 临时提问，用当前上下文但不保存，不调工具 | 中 |
| `/queue 提示词` | 排队到下一轮再处理 | 中 |
| `/rollback` | 回滚文件改动，像时光机 | 中 |
| `/resume 名字` | 恢复某个命名会话 | 中 |
| `/stop` | 停掉所有后台进程 | 低 |
| `/title 名字` | 给当前会话起个好记的名字 | 低 |
| `/save` | 把对话保存到文件 | 低 |

**配置相关👇**

| 指令 | 说人话 | 频率 |
| --- | --- | --- |
| ----- | ------- | ----- |
| `/model` | 查看或切换当前模型 | 高 |
| `/yolo` | 切换是否跳过危险命令确认 | 中 |
| `/voice on` | 开语音输入 | 中 |
| `/voice tts` | 让 Hermes 朗读回复 | 中 |
| `/reasoning high` | 调高推理强度（none/low/medium/high/xhigh） | 中 |
| `/personality 名字` | 切换人格风格 | 低 |
| `/verbose` | 切换工具进度显示详细程度 | 低 |
| `/prompt 文本` | 查看或设置系统提示词 | 低 |
| `/skin 名字` | 换界面主题 | 低 |
| `/statusbar` | 开关顶部状态栏 | 低 |

**工具与技能👇**

| 指令 | 说人话 | 频率 |
| --- | --- | --- |
| ----- | ------- | ----- |
| `/skill 名字` | 把某个技能加载进当前会话 | 高 |
| `/tools` | 管理工具 | 中 |
| `/skills` | 搜索或安装技能 | 中 |
| `/browser connect` | 连接本地 Chrome 浏览器（CDP 协议） | 低 |
| `/paste` | 检查剪贴板里的图片并附加 | 低 |
| `/cron` | 管理定时任务 | 低 |
| `/reload-mcp` | 重新加载 MCP 服务器 | 低 |
| `/plugins` | 看已装的插件 | 低 |

**信息查看👇**

| 指令 | 说人话 | 频率 |
| --- | --- | --- |
| ----- | ------- | ----- |
| `/help` | 看所有命令 | 高 |
| `/usage` | 看 token 用了多少 | 中 |
| `/profile` | 看当前用的是哪个配置档案 | 低 |
| `/insights` | 看使用统计 | 低 |
| `/provider` | 看当前提供商信息 | 低 |

退出用 `/quit`，或者 `/exit`，或者 `/q`，三个都行。

**在 Telegram / Discord 等平台里，还有几个专属指令👇**

| 指令 | 说人话 |
| --- | --- |
| ----- | ------- |
| `/approve` | 批准 Hermes 要执行的危险命令 |
| `/deny` | 拒绝 |
| `/sethome` | 把当前聊天设为主频道，cron 任务默认推送到这里 |
| `/status` | 看当前会话信息 |
| `/commands` | 分页浏览所有命令 |
| `/update` | 在平台里直接更新 Hermes |

— — —

▍ 避坑 + 组合用法

几个真实踩过的坑。

**Windows 用户目前必须走 WSL2。**

不能直接在 PowerShell 或 CMD 里装，会报错。

先以管理员身份打开 PowerShell，

运行 `wsl --install`，重启电脑，打开 Ubuntu 终端，

再在 Ubuntu 里运行安装命令。

**`/rollback` 要提前开启。**

这个功能默认是关的，需要启动时加参数👇

```
hermes chat --checkpoints
```

开了之后每次文件改动前会自动存档，改坏了直接 `/rollback` 回来。

**`/yolo` 用之前想清楚。**

这个指令会跳过所有危险命令的确认提示，包括删文件、跑 SQL、执行脚本。

调试的时候方便，但别忘了关掉。

**几个好用的组合用法👇**

```
# 单次提问 + 指定模型，不进入聊天界面
hermes chat -q "帮我写一个 Python 爬虫" -m deepseek/deepseek-chat

# 恢复上次会话 + 预加载技能
hermes --continue -s github-pr-workflow

# 安静模式，适合脚本调用，只输出最终回答
hermes chat -q "今天天气怎么样" --quiet
```

`hermes gateway run` 加上 cron 任务，就是一套全自动定时推送系统，不用你盯着。

**想让 Hermes 在 Docker 里跑，更安全👇**

```
hermes config set terminal.backend docker
```

改完之后 Hermes 执行终端命令都在容器里，不会动你的本机环境。

也可以指定镜像👇

```
# ~/.hermes/config.yaml
terminal:
  backend: docker
  docker:
    image: python:3.11
    persistent: true
```

**想在脚本或应用里调用 Hermes👇**

这块是给开发者用的，想把 Hermes 嵌进自己的程序里。普通用户跳过就好。

```
pip install git+https://github.com/NousResearch/hermes-agent.git
```

```
from run_agent import AIAgent

agent = AIAgent()
result = agent.run_conversation("帮我分析这段代码")
print(result)
```

— — —

▍ 10 条源码洞见

翻了 Hermes 的源码之后，整理出 10 条觉得有意思的设计决策，给感兴趣的朋友看。

**1. 技能索引在系统提示里，全文在用户消息里。**

说人话就是，它只把技能的「目录」放在每次对话的背景里，技能的完整内容只在你真正用到的时候才加载进来。这样 AI 每次处理消息的成本能降 80%。

**2. 整个 Agent 循环是同步的，没有 asyncio。**

说人话就是，它选择了「一件事做完再做下一件」的方式，而不是「多件事同时做」。好处是代码更简单，别人更容易给它写插件。

**3. 工具是自注册的，不需要中心配置文件。**

说人话就是，加一个新工具，只需要写那个工具本身，不用去改任何「工具列表」文件，装上就能用。

**4. 技能是数据，不是代码。**

说人话就是，技能就是一个普通的文本文件，你可以直接用记事本打开改，Hermes 自己也能改，这才是真正的「自学习」。

**5. 记忆的目标是「减少未来的纠正」。**

说人话就是，它不是把所有聊天记录都存下来，而是只存那些「下次如果忘了你会纠正它」的东西，比如你的偏好、习惯、约定。

**6. 跨会话召回用的是 FTS5 全文搜索加 LLM 摘要。**

说人话就是，它能搜索你所有历史对话，找到相关内容，比你自己翻聊天记录快多了，而且不需要把所有历史都塞进当前对话里。

**7. 子 Agent 有工具限制和并发上限。**

说人话就是，它可以同时派出多个「小助手」并行干活，但每个小助手的权限是受限的，最多同时跑 3 个，防止失控。

**8. Gateway 会话 300 秒无活动自动触发记忆刷新。**

说人话就是，你在 Telegram 里和它聊完，5 分钟没动静，它会自动把这次对话的重要内容存进记忆，不用你手动保存。

**9. MCP 客户端有 2195 行。**

说人话就是，接外部工具这个功能很强大，但实现起来很复杂，出问题了比较难排查，这是目前最难用的部分。

**10. 每次文件改动前自动 git commit。**

说人话就是，它帮你改文件之前会先存一个备份，改坏了用 `/rollback` 可以一步步撤回来，这个设计是从 Claude Code 借鉴来的。

— — —

感谢朋友们的阅读，希望可以点赞、在看、留言~

想要一起长期探索 Hermes和龙虾这类AI Agent工具玩法的朋友，

可以在公众号后台私发「Agent」，我会拉你进群交流。

— — —

**参考来源**

https://hermes-agent.nousresearch.com/docs

Hermes Agent 官方文档

https://hermes-agent.nousresearch.com/docs/skills

Hermes Agent 官方 Skills 系统文档

https://github.com/NousResearch/hermes-agent

Hermes Agent 官方 GitHub 主仓库

#HermesAgent   #Agent   #AI工具   #OpenClaw   #AIAgent