---
title: Craft Agents：用 GUI 重做 Claude Code 的野心和底气
author: Git Trend
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwMzY1Mjg5MQ==&mid=2247486501&idx=1&sn=9fbc202e6ec6b1f986f5e5a0d8355d55&chksm=c1e715ec8c37ce58d467fe36add3072c4d99e1c9cc582e52f03c34d16876bf8259885e180fc5&mpshare=1&scene=24&srcid=0422ffvUqkM2IJsc2128p826&sharer_shareinfo=8c761cb82e2c022c30f47419236e166e&sharer_shareinfo_first=8c761cb82e2c022c30f47419236e166e#rd
---

> **项目卡片**
>
> * **项目**：Craft Agents[1]
> * **状态**：v0.8.9 / 4.3k Star / 近 30 天 16 次提交，周更节奏
> * **一句话判断**：如果把 Claude Code 换成一个带会话管理、Source 连接、自动化触发器的桌面 App，这就是答案

是什么，凭什么值得关注

Craft Agents 出自 Craft Docs 团队——就是做 Craft 文档编辑器的那个公司。2026 年 1 月开源，Apache 2.0，到现在 78 个 commit，版本号 v0.8.9。

他们想要一个非 CLI 的 Agent 工作环境。而且不是"想要"而已：团队内部的日常工作已经在用 Craft Agents 开发 Craft Agents 本身。这种狗粮式开发至少说明项目已经过了 demo 阶段。

技术栈：Claude Agent SDK 做主引擎，Pi SDK 覆盖 Google AI Studio / ChatGPT Plus / GitHub Copilot 等非 Anthropic 通道，Bun 做运行时，Electron 包桌面壳，React + shadcn/ui + Tailwind CSS v4 做 UI。

和 Claude Code、Cursor 的本质区别

把它理解为"带 GUI 的 Claude Code"太表面了。真正拉开差距的是三个架构决策：

1. **多会话 Inbox**——不是一个终端一条对话，而是类似邮件客户端的会话管理器，支持 Todo → In Progress → Needs Review → Done 的状态流，加标记和归档
2. **Source 系统**——把 MCP Server、REST API、本地文件系统统一成一种抽象，对话中用 `@` 实时引用，无需重启
3. **自动化触发器**——基于事件（标签变更、工具调用、定时器）自动创建 agent 会话并执行

这三点放在一起，Craft Agents 更像一个 Agent 操作系统，而不仅是编程助手。

Source 系统：三合一连接架构

Source 是 Craft Agents 最核心的差异化设计。它把所有外部连接统一成三种类型：

* **mcp**：标准 MCP Server，支持 stdio 和 SSE 两种传输方式
* **api**：REST API 直连，内置 Google（Gmail/Calendar/Drive/YouTube/Search Console）、Slack、Microsoft 等服务的 OAuth 流程
* **local**：本地文件系统，支持 Obsidian vault、Git 仓库等

每个 Source 以文件夹形式存储在 `~/.craft-agent/workspaces/{id}/sources/{slug}/` 下，包含 `config.json` 和 `guide.md`。agent 连接新 Source 时，会自动发现公开 API 和 MCP Server、读取文档、配置凭证——用户只需要说"add Linear as a source"。

凭证管理值得单独说。所有凭证存储在 `~/.craft-agent/credentials.enc`，AES-256-GCM 对称加密。密钥派生路径很有意思——它直接读硬件标识符（macOS 的 IOPlatformUUID、Windows 注册表的 MachineGuid、Linux 的 `/var/lib/dbus/machine-id`），通过 PBKDF2（10 万次迭代）生成加密密钥。凭证文件即使被拷走，换一台机器也解不开。

权限模型：三级防护

Craft Agents 的权限控制分三档，通过 `SHIFT+TAB` 实时切换：

| 模式 | 行为 |
| --- | --- |
| **Safe**（Explore） | 只读，屏蔽所有写操作 |
| **Ask**（Ask to Edit） | 默认模式，写操作需用户确认 |
| **Allow-All**（Auto） | 自动批准所有命令 |

在 Ask 模式下，系统实现了完整的 PreToolUse 管道：权限检查 → Source 阻断 → 前置条件校验 → 路径展开 → 配置校验 → 技能资格认证 → 用户确认决策。六步链式检查在 Claude、Codex、Copilot、Pi 四个后端间共享。

对于本地 MCP Server（stdio 传输），Craft Agents 会过滤敏感环境变量，防止 API Key 泄露到子进程。`ANTHROPIC_API_KEY`、`AWS_ACCESS_KEY_ID`、`GITHUB_TOKEN` 等全部在屏蔽名单中。

自动化：事件驱动的 Agent 工作流

这块是我觉得最被低估的功能。它支持两类事件源：

* **应用事件**：LabelAdd/Remove、PermissionModeChange、SessionStatusChange、SchedulerTick
* **Agent 事件**：PreToolUse、PostToolUse、UserPromptSubmit、SessionStart/End 等

触发后可执行两类动作：**Prompt**（创建新 agent 会话）和 **Webhook**（发送 HTTP 请求）。Prompt 动作支持 `@mentions` 引用 Source 和 Skill，`$CRAFT_LABEL`、`$CRAFT_SESSION_ID` 等环境变量自动展开。

配置示例——每周一到周五 9 点自动检查 GitHub 新 Issue：

```
{  
  "version": 2,  
  "automations": {  
    "SchedulerTick": [{  
      "cron": "0 9 * * 1-5",  
      "timezone": "America/New_York",  
      "labels": ["Scheduled"],  
      "actions": [{  
        "type": "prompt",  
        "prompt": "Check @github for new issues assigned to me"  
      }]  
    }]  
  }  
}
```

这套系统让 Craft Agents 从"你问它答"升级为"它自己跑"——配合 Headless 服务器模式，可以实现 7×24 自动化工作流。

Headless 服务器 + CLI：不绑桌面

Craft Agents 可以脱离 Electron 独立运行。服务端是一个纯 Bun 进程，通过 WebSocket RPC 暴露所有能力：

```
CRAFT_SERVER_TOKEN=$(openssl rand -hex 32) \  
  bun run packages/server/src/index.ts
```

桌面端可以连接远程服务器作为瘦客户端（`CRAFT_SERVER_URL` + `CRAFT_SERVER_TOKEN`），也可以通过 CLI 客户端操作：

```
# 自包含模式：启动服务器 → 发送 prompt → 退出  
craft-cli run "Summarize the README"  
  
# 连接已有服务器  
craft-cli --url ws://server:9100 --token <token> sessions  
craft-cli send <session-id> "What files changed?"
```

CLI 的 `run` 命令值得单独拎出来——它会自己拉起 headless server、创建 session、发 prompt、stream 回复、然后退出。一行命令在 CI/CD 管道里用上 agent 能力，不需要任何前置基础设施。

工程细节和值得注意的点

**多后端 Agent 架构**。Craft Agents 同时维护四个 Agent 后端（Claude、Codex、Copilot、Pi），共享 PreToolUse 管道但各自管理 SDK 生命周期。Claude 后端直接用 Claude Agent SDK，Pi 后端通过 `@mariozechner/pi-ai` 和 `@mariozechner/pi-coding-agent` 接入。

**大响应自动摘要**。MCP 工具响应超过 ~60KB 时，自动用 Claude Haiku 做意图感知的摘要压缩。它不是简单截断，而是在 MCP schema 里注入了 `_intent` 字段来保留摘要的上下文聚焦能力。这个设计比较精细。

**会话持久化用 JSONL**。每条消息一行 JSON，streaming 友好且可增量写入。路径处理做了跨平台适配（Windows ↔ Unix），支持会话在机器间迁移。

**多 LLM 连接**。一个 Workspace 可以配多个 LLM Provider（Anthropic、OpenAI、Google、OpenRouter、Ollama 等），每个 Session 可以指定不同的模型和 Provider。通过 custom base URL 还能接入任意 OpenAI 兼容端点。

**Deep Linking**。`craftagents://` 协议支持外部应用直接跳转到特定会话、Source、设置页面。

值得留意的几个问题

* **Bun 强依赖**。从构建到运行都绑定 Bun Runtime。团队环境如果还没 Bun，这是第一个门槛
* **Electron 壳**。`electron` ^39.2.7，标准 Electron 打包体积和内存开销都摆在那里
* **Pi SDK 闭源**。Google AI Studio / ChatGPT Plus / GitHub Copilot 三个通道依赖 `@mariozechner/pi-ai` 这个闭源 SDK，出了问题只能等上游修
* **版本 0.8.x**。78 个 commit 了但还没到 1.0，配置格式和 API 随时可能有 breaking change。用它做生产基础设施的话，要做好跟进升级的准备

值不值得试

**适合这些场景**：已经用 Claude Code 但受限于 CLI 交互方式；需要管理多个并发 agent 任务，单终端窗口不够用；想给非技术同事提供 agent 访问能力（GUI 比终端友好得多）；需要 7×24 自动化 agent 工作流（Headless 模式）。

**可以再等等**：团队标准化在 Node.js Runtime 上，或者需要和现有 IDE 深度集成。

4.3k Star、645 Fork、Apache 2.0，周更节奏稳定。在"Agent GUI"这个赛道上，它目前是最完整的开源方案。

---

如果这篇对你有用，建议点个关注。我会持续把 GitHub 上值得用的 AI 工具拆成「最短上手闭环 + 坑点清单 + 可复用配置」，让你少走弯路。

### 引用链接

[1]Craft Agents: *https://github.com/lukilabs/craft-agents-oss*