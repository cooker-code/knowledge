> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Unsloth API 来了，24GB 显存就能在 Claude Code、Codex、OpenClaw 里跑本地 Agent
author: Ai学习的老章
date: 老章很忙老章很忙
url: https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649013605&idx=1&sn=c403b633bf2e08a7785bc3a5c15b4361&chksm=86fa8a3ce31aa7b7ef70867cacb20964cd46036d2c9b6242736656af85318e7ca60ccdedf3f6&mpshare=1&scene=24&srcid=0509iwMkPHxO449IrUWw1YPj&sharer_shareinfo=311b65f5297f0614b35697234599b561&sharer_shareinfo_first=311b65f5297f0614b35697234599b561#rd
---

[Unsloth联手英伟达：3个优化让家用GPU训练LLM提速25%](https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649013460&idx=1&sn=ee7cca873d4f59841660cbd059806190&scene=21#wechat_redirect)

[DeepSeek-V4 蒸馏 Qwen3.5，只有 9B，本地能跑](https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649013440&idx=1&sn=191a51d8c136363dd72383cc1802eb7e&scene=21#wechat_redirect)

[Claude把手伸向了金融，开源10个金融Agent模板](https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649013310&idx=1&sn=41a882318a730a27e46846c35ddb84ea&scene=21#wechat_redirect)

今天聊点更有意思的：把 Claude Code、Codex、OpenClaw 这三个最火的 Agent 终端，全都接到本地 24GB 显存的开源大模型上跑

之前对接 Claude Code 的常规姿势，是把环境变量指到 DeepSeek、Kimi 这种云端 API，体验不错，但月底账单也是真不便宜

Unsloth 团队最近放了个大招——给自家 Studio 直接挂上了一个 **OpenAI 兼容 + Anthropic 兼容** 的双协议 API 端点，本地一条命令起服务，Claude Code、Codex、OpenClaw、OpenCode、Cursor、Cline 全都能直接接

24GB 内存（不论 Mac 统一内存还是 RTX 显卡）就能跑得起 Gemma 4 26B-A4B 或 Qwen3.6-27B，全程不联网，自己的代码不出本机

### 简介

先说说 Unsloth API 到底是啥

它其实是 Unsloth Studio（Unsloth 自家的本地推理 UI）开出来的一个对外 HTTP 端口，背后是 `llama.cpp` 的 `llama-server`，前面套了一层兼容层，**同一个端口同时讲两种话**

* `POST /v1/messages` —— Anthropic Messages API，给 Claude Code、Anthropic SDK、OpenClaw 用
* `POST /v1/chat/completions` 和 `/v1/responses` —— OpenAI 兼容，给 OpenAI SDK、Codex、OpenCode、Cursor、Continue、Cline、Open WebUI 用
* `GET /v1/models` —— 列出当前已加载的模型

认证方式跟 OpenAI 一模一样，请求头带 `Authorization: Bearer sk-unsloth-…`，启动后 key 在终端里直接打出来一份

Unsloth API 双协议示意

最关键的是它带了三个云端 API 才有的高级能力：

* **Self-healing tool calling**：模型偶尔会把工具参数写歪（少个引号、JSON 嵌套乱了），Unsloth 服务端会自动修一下再喂给客户端，工具调用成功率明显高一截
* **服务端代码执行**：在请求里加 `enable_tools: true` 和 `enabled_tools: ["python", "bash"]`，Bash / Python 直接在服务端沙箱里跑完把结果回流，跟 Claude artifacts 那个味儿差不多
* **Advanced Web Search**：模型能真去访问网页、读正文，不是只看一下 snippet

这几个能力以前是 Claude / OpenAI 这种闭源 API 的护城河，现在全本地化了

### 安装

整个链路就两件事：装 Unsloth Studio，再装你要用的 Agent CLI

**装 Unsloth Studio（一行）**

```
# macOS / Linux / WSL
curl -fsSL https://unsloth.ai/install.sh | sh

# Windows PowerShell
irm https://unsloth.ai/install.ps1 | iex
```

**加载一个 GGUF 模型并启动 API**

```
unsloth run unsloth/Qwen3.6-27B-GGUF
# 或者跑 Gemma 4
unsloth run unsloth/gemma-4-26B-A4B-it-GGUF
```

启动完终端会打印出两行很重要的东西，一行是端口（一般是 `http://localhost:8000` 或 `8888`），一行是 `sk-unsloth-...` 开头的 API key，这玩意儿只显示一次，记得马上存下来

也可以从 UI 里手动建：左下角头像 → Settings → API → 起个名字 → Create

### 硬件门槛

24GB 内存能跑哪些模型，Unsloth 官方给了一张实测表，老章挑两个最值得本地用的列出来：

| 模型 | 4-bit 推荐内存 | 适合谁 |
| --- | --- | --- |
| Gemma 4 26B-A4B（MoE） | 28–30 GB | M-series Mac 32GB 统一内存最稳 |
| Gemma 4 E4B（密集） | 9–12 GB | 8GB 显卡也能跑 |
| Qwen3.6-27B | 18 GB | 24GB 显存富裕 |
| Qwen3.6-35B-A3B（MoE） | 23 GB | 24GB 卡踩线，30GB 舒服 |

老章自己 M4 Pro 48GB 跑 Qwen3.6-27B Q4\_K\_XL，上下文 32K，吐字速度大概 25 tok/s，写代码完全够用

> ❝
>
> ⚠️ 提醒一下，CUDA 13.2 跑 GGUF 现在有 bug 会输出乱码，N 卡用户先用 13.1 或 12.x，NVIDIA 还在修

三家 CLI 接入对照

### 接入 Claude Code

**装 Claude Code**

```
curl -fsSL https://claude.ai/install.sh | bash
# 或 brew install --cask claude-code
```

**指向 Unsloth 端点**

```
export ANTHROPIC_BASE_URL="http://localhost:8888"
export ANTHROPIC_API_KEY="sk-unsloth-你的key"
```

**关掉那个让推理慢 90% 的坑**

这是个老章踩过的坑，Claude Code 最近会在每次请求前偷偷加一个 attribution header，header 一变 KV Cache 直接全废，推理速度掉 90%

`export CLAUDE_CODE_ATTRIBUTION_HEADER=0` 是没用的，必须写到配置文件里

```
cat > ~/.claude/settings.json <<'EOF'
{
  "env": {
    "CLAUDE_CODE_ATTRIBUTION_HEADER": "0"
  }
}
EOF
```

进项目目录跑 `claude`，再 `/model` 一下确认走的是本地模型，就齐活了

### 接入 Codex

Codex 现在只认 OpenAI Responses API（Chat Completions 已经在弃用路上），还好 Unsloth 在同一个端口上把 `/v1/responses` 也开了

**装 Codex**

```
brew install --cask codex
# 或 npm install -g @openai/codex
```

**配置 `~/.codex/config.toml`**

```
[model_providers.unsloth]
name = "unsloth"
base_url = "http://localhost:8888/v1"
wire_api = "responses"
env_key = "UNSLOTH_API_KEY"

[profiles.local]
model_provider = "unsloth"
model = "Qwen3.6-27B-GGUF"
```

```
export UNSLOTH_API_KEY="sk-unsloth-你的key"
codex --profile local
```

模型 ID 不知道写啥，直接 `curl http://localhost:8888/v1/models` 把 `id` 字段抄过去

### 接入 OpenClaw

OpenClaw 是个开源 Agent 终端，用 Anthropic Messages 协议跟模型说话，跟 Unsloth 是天作之合

**装 OpenClaw**

```
curl -fsSL https://openclaw.ai/install.sh | bash
```

**编辑 `~/.openclaw/openclaw.json`**

```
{
  "models": {
    "mode": "merge",
    "providers": {
      "unsloth": {
        "baseUrl": "http://localhost:8888/v1",
        "api": "anthropic-messages",
        "authHeader": true,
        "apiKey": "sk-unsloth-你的key",
        "models": [
          { "id": "Qwen3.6-27B-GGUF", "name": "Qwen3.6 本地" }
        ]
      }
    }
  }
}
```

注意 `baseUrl` 必须以 `/v1` 结尾，`api` 写 `anthropic-messages` 是告诉 OpenClaw 走 `/v1/messages` 这条路

### 实测体验

老章用 Qwen3.6-27B 在 Claude Code 里跑了一上午，几个真实感受写在这

**好的方面**

* 第一次 cold start 大概 30 秒，之后提示词响应秒回，本地跑不用等队列
* self-healing tool calls 真的有用，之前直接挂 llama.cpp 给 Claude Code 用，工具调用十次有三次参数 JSON 裂开，Unsloth 这边几乎没翻车
* 隐私这块踏实，公司项目敏感代码再也不用纠结要不要传出去
* 不用再盯账单了，电费怎么也比 API token 便宜

**不太好的地方**

* 27B 4-bit 比起 Claude Sonnet 4 / GPT-5 这种顶级模型，长链路任务（比如重构十几个文件）还是会糊，复杂任务老章建议拆小步喂
* 工具调用响应整体比云端慢一点，尤其是带 web search 的，本地浏览器抓页面就是慢
* Codex 现在一定要 `wire_api = "responses"`，老的 `chat` 模式已经不推荐，配错了会一直 400

**我的建议**

把它当 **日常副驾** 用，写脚手架、改 bug、跑测试、刷文档，本地模型完全够用，性能还稳定不限速

真要啃硬骨头（架构设计、跨多文件大重构），切回 Claude / GPT-5 这种顶级 API，按需混用最划算

### One More Thing

Unsloth 这一手在我看来意义挺大的——以前本地跑 Agent，瓶颈不在模型，而在 **协议生态**

llama.cpp 自己有 OpenAI 兼容端点，但 Claude Code 走的是 Anthropic 协议，两边对不上；想接 Claude Code 就得自己写代理层，门槛劝退

Unsloth 直接把 OpenAI 和 Anthropic 两个协议都喂在同一个端口上，再把 self-healing、tool calling、code execution、web search 这些原本要各家 SDK 各搞一遍的能力做成服务端默认开启

**装一次，三家 CLI 全通**，这才是本地 Agent 应该有的样子

如果你之前因为生态不全没真的把 Claude Code 接到本地用过，这次值得再试一遍

文档地址：unsloth.ai/docs/basics/api

**制作不易，如果这篇文章觉得对你有用，可否点个关注。给我个三连击：点赞、转发和在看。若可以再给我加个🌟，谢谢你看我的文章，我们下篇再见！**

[视频号做到6000粉，我每天只做一件事](https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649013460&idx=2&sn=e5687422fe9b1256fa5aeb58bd794cfe&scene=21#wechat_redirect)