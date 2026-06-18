---
title: OpenClaw 速查表
author: 彼岸流天
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5NDE1MzYxMg==&mid=2247484898&idx=1&sn=0394535777617e2bc6ec58b589e665a8&chksm=c13d9e207703518d1f7f17c127961411537bc667194bc6dd45181992f6b6d195d16386496434&mpshare=1&scene=24&srcid=0326xGso1Gc9383zYzKk00hU&sharer_shareinfo=8bb535b2186fd3e3924793c086e3dcdf&sharer_shareinfo_first=8bb535b2186fd3e3924793c086e3dcdf#rd
---

你大概率遇到过：**网关又挂了、配对码过期了、文档翻了三层还没找到那条命令**——纯干货清单人人会收藏，但缺一张「遇事能秒翻」的桌面。这篇写给**已经决定自托管 OpenClaw**、又不想每次从搜索引擎重新拼关键词的人：读完你可以把浏览器关掉，靠本页搞定**端口与路径、完整 CLI（含 `tui` / `daemon` / `gateway` / `models` 等子命令索引）、架构名词、Skills/MCP 区别、排错起手式**。

下面是**并列模块**：从哪一部分读都行；要和长文教程配合用时，安装与踩坑请看同目录《OpenClaw 从安装到入门的完全指南》。

> **用途**：日常查阅命令、路径、概念与排错入口。  
> **架构细节**：见仓库 `architecture-design/openclaw/` 与 官方文档。  
> **分析对象**：openclaw/openclaw。**本文第 4 部分**对应文中标题「4. CLI 速查（细表）」；该部分据 **OpenClaw 2026.3.23-2**（`7ffe7e4`）本机 `openclaw --help` 与子命令 `--help` 交叉整理。升级 CLI 后请以本机 `--help` 与 官方 CLI 为准。

---

## 1. 一句话

**OpenClaw（小龙虾）**：自托管的个人智能体平台——在 Telegram / WhatsApp / Discord / Web 等入口用自然语言下指令，在你自己的机器上**执行**（文件、终端、浏览器、邮箱等），并把结果推回聊天窗口。  
口号：**The AI that actually does things**。

---

## 2. 架构一屏（谁干什么）

| 组件 | 角色 | 记住一句 |
| --- | --- | --- |
| **Gateway** | 唯一常驻 daemon，Channel + WebSocket + 配对 + 路由 | 「总机」：连谁、谁有权、消息往哪送 |
| **Agent Runtime** | 理解意图、组上下文、调模型、跑工具循环 | 「接线员 + 执行者」 |
| **Channel** | 各聊天平台适配，消息标准化进 Gateway | Telegram / WA / Discord / WebChat … |
| **Tool Engine** | 内置工具、Skills、MCP | exec / browser / web / Skills / MCP |
| **Model Provider** | 多厂商统一接口 + 降级 | Anthropic / OpenAI / Moonshot / Gemini / Ollama … |
| **Session + Memory** | 会话历史 + 长期记忆检索 | Session 短期；Memory 跨会话（sqlite-vec 等） |
| **Workspace + Prompt** | 工作目录与 AGENTS/SOUL/TOOLS 等 | 定规矩、性格、工具习惯 |
| **Security** | 沙盒、工具策略、审批、配对 | 渐进式收紧权限 |

默认控制面：**本机 WebSocket**，不默认暴露公网。

---

## 3. 默认端口与路径

| 项 | 值 |
| --- | --- |
| WebSocket API | `ws://127.0.0.1:18789/ws` |
| Dashboard / Web UI | `http://127.0.0.1:18789/` |
| Health | `http://127.0.0.1:18789/health` |
| Status | `http://127.0.0.1:18789/status` |
| WhatsApp Webhook（若用） | `http://127.0.0.1:18789/webhook/whatsapp` |
| 数据根目录 | `~/.openclaw/` |
| 主配置 | `~/.openclaw/openclaw.json` |
| 环境变量 / 密钥 | `~/.openclaw/.env` 、`credentials/` 等 |
| Workspace 默认 | `~/.openclaw/workspace/` （含 AGENTS.md、SOUL.md、TOOLS.md 等） |
| 网关 PID（常见） | `/tmp/openclaw-gateway.pid` |
| 日志（常见） | `~/.openclaw/logs/gateway.log` |

---

## 4. CLI 速查（细表）

命令形式：`openclaw [全局选项] <command> [子命令] [选项]`。带 `*` 的命令有子命令，务必 `openclaw <command> --help`。下文 **4.0**～**4.15** 为同级小节，文内交叉引用统一写作「见 **4.x**」。

### 4.0 全局选项（最常用）

| 选项 | 作用 |
| --- | --- |
| `-h, --help` | 帮助 |
| `-V, --version` | 版本号 |
| `--dev` | 开发配置：状态隔离到 `~/.openclaw-dev`，Gateway 默认端口 **19001**，衍生端口顺延 |
| `--profile <name>` | 命名配置：状态/配置隔离到 `~/.openclaw-<name>` |
| `--log-level <level>` | 全局日志：`silent` | `fatal` | `error` | `warn` | `info` | `debug` | `trace` |
| `--no-color` | 关闭 ANSI 颜色 |

示例：`openclaw --dev gateway run`（独立 dev 网关）。

### 4.1 一级命令索引（速览）

| 命令 | 一句话 |
| --- | --- |
| `tui` | **终端 UI** ，连 Gateway 里聊天（见 **4.2**） |
| `gateway *` | 运行 / 探活 / 安装服务 / RPC |
| `daemon *` | **Gateway 系统服务** （launchd/systemd/schtasks；CLI 标注为 *legacy alias*，与 `gateway` 服务子命令同类） |
| `agent` | 非交互 **一轮** Agent（可 `--deliver` 回渠道） |
| `dashboard` | 浏览器打开 Control UI（带当前 token） |
| `health` | 拉取运行中 Gateway 的 health |
| `onboard` / `setup` / `configure` | 向导 / 最小初始化 / 交互改配置 |
| `config *` | **非交互** get/set/unset/validate |
| `models *` | 模型列表、默认模型、auth profile、fallback |
| `channels *` / `message *` / `directory *` | 渠道账号 / 收发消息与频道操作 / 查 ID |
| `pairing *` | DM 配对 |
| `devices *` / `qr` | 设备配对与 token / iOS 扫码配对 |
| `nodes *` / `node *` | Gateway 管理的 Node（手机等）/ 无头 Node 宿主服务 |
| `sessions *` / `memory *` | 会话 / 记忆检索与索引 |
| `approvals *` | exec 审批（别名 `exec-approvals`） |
| `security *` / `secrets *` / `sandbox *` | 安全审计 / 密钥运行时 / Docker 沙盒 |
| `cron *` / `hooks *` / `plugins *` | 定时任务 / 内置 hooks / 插件 |
| `browser *` / `acp *` | 专用浏览器自动化 / ACP 桥 |
| `backup *` / `system *` / `webhooks *` / `dns *` | 备份 / 系统事件与 heartbeat / Webhook 辅助 / CoreDNS 广域发现 |
| `logs` / `doctor` / `docs` / `status` | 日志 tail / 体检 / 搜文档 / 渠道与会话摘要 |
| `skills *` / `agents *` | Skills / 多 Agent 实例 |
| `update *` / `reset` / `uninstall` | 升级 / 重置状态 / 卸载服务与数据 |
| `completion` / `clawbot *` | Shell 补全 / 旧版别名 |

### 4.2 `tui`：终端 UI

**前置**：Gateway 已运行（`gateway run` 或 `gateway start` / `daemon start`）。

```
openclaw tui
```

| 常用选项 | 说明 |
| --- | --- |
| `--session <key>` | 会话键，默认 `main` |
| `--message <text>` | 连接后自动发送一条 |
| `--history-limit <n>` | 载入历史条数，默认 `200` |
| `--thinking <level>` | 覆盖 thinking 等级 |
| `--timeout-ms <ms>` | Agent 超时（毫秒） |
| `--deliver` | 是否把助手回复**投递回渠道**（默认 false） |
| `--token` / `--password` | 网关鉴权（若配置为 token/password） |
| `--url <url>` | Gateway WebSocket URL（未配则用 `gateway.remote.url`） |

文档：https://docs.openclaw.ai/cli/tui

### 4.3 `gateway *` 与 `daemon *`

| `gateway` 子命令 | 说明 |
| --- | --- |
| `run` | 前台 WebSocket Gateway |
| `start` / `stop` / `restart` / `install` / `uninstall` | 系统服务生命周期 |
| `status` | 服务状态 + 探活 |
| `health` | 健康检查 |
| `probe` | 可达性 + 发现 + health + status **摘要** |
| `discover` | Bonjour 发现网关 |
| `call` | 调用 Gateway RPC（如 `gateway call health`） |
| `usage-cost` | 从会话日志汇总用量/成本 |

**`gateway run` 常见选项**：`--port`、`--bind`（`loopback` | `lan` | `tailnet` 等）、`--token`、`--force`（占端口时强杀旧进程）、`--tailscale`、`--verbose`、`--ws-log` …

**`daemon`**：`install` / `start` / `stop` / `restart` / `status` / `uninstall` —— 专注 Gateway **服务**；官方 CLI 说明其为 **legacy alias**，文档入口仍指向 gateway。

### 4.4 `agent` / `dashboard` / `health`

| 用法 | 说明 |
| --- | --- |
| `openclaw agent -m "…"` | 经 Gateway **一轮**；可加 `--agent`、`--session-id`、`--thinking`、`--json` |
| `openclaw agent --local -m "…"` | **本机嵌入式** 跑（当前 shell 需能访问模型 API） |
| `openclaw agent --deliver …` | 回复发回渠道；可配合 `-t` / `--to`（E.164）、`--reply-channel`、`--reply-to` |
| `openclaw dashboard` | 打开 Control UI |
| `openclaw health` | 请求 Gateway health（`--json` 便于脚本） |

### 4.5 `onboard` / `setup` / `configure` / `config`

| 命令 | 说明 |
| --- | --- |
| `openclaw onboard` | 全量向导；可加 `--install-daemon` |
| `openclaw setup` | 最小初始化 |
| `openclaw configure` | 交互配置；`--section`：`workspace` / `model` / `web` / `gateway` / `daemon` / `channels` / `skills` / `health` |
| `openclaw config` | 无子命令 → 常进入引导 |
| `openclaw config get <dot.path>` / `set` / `unset` / `validate` | 非交互；`set` 支持 `--ref-source env` 等引用密钥 |

### 4.6 `models *`

| 子命令 | 说明 |
| --- | --- |
| `list` / `status` | 列出模型 / 当前配置状态（根上也有 `--status-json`） |
| `set` / `set-image` | 默认对话模型 / 生图模型 |
| `auth` | 模型鉴权 profiles |
| `fallbacks` / `image-fallbacks` | 降级链 |
| `aliases` | 别名 |
| `scan` | 扫描 OpenRouter 等（工具+图能力） |

### 4.7 `channels *` / `message *` / `directory *`

**channels**：`list`、`add`、`remove`、`login`、`logout`、`status`（可加 `--deep`）、`capabilities`、`resolve`、`logs`。

**message**：`send`、`read`、`react`、`poll`、`thread`、Discord `search`、成员/权限等 —— 子命令很多，`message --help`。

**directory**：`self`、`peers`、`groups`（查联系人/群 ID）。

### 4.8 `pairing *` / `devices *` / `qr` / `nodes *` / `node *`

| 模块 | 要点 |
| --- | --- |
| `pairing list` | 待处理 DM 配对 |
| `pairing approve` | 支持 **单参数**`<code>` 或 **`--channel <ch> <code>`**（见 `pairing approve --help`） |
| `devices …` | `list` / `approve` / `reject` / `remove` / `revoke` / `rotate` |
| `qr` | iOS 配对 QR；`--remote`、`--token`、`--public-url` 等 |
| `nodes …` | `status` 、`invoke`、相机/投屏/通知等（移动端 Node） |
| `node …` | 无头 Node 宿主：`run` / `install` / `status` … |

### 4.9 `sessions *` / `memory *`

* • **sessions**：列表 + `--agent` / `--all-agents` / `--active <分钟>` / `--json`；子命令 `cleanup`。
* • **memory**：`search`（`--query`、`--max-results`）、`index`（`--force`）、`status`（`--deep`、`--json`）。

### 4.10 `approvals *` / `security *` / `secrets *` / `sandbox *`

* • `approvals`：`get` / `set` / `allowlist`
* • `security audit`：`--deep`、`--fix`、`--json`
* • `secrets`：`reload`、`audit`、`configure`、`apply`
* • `sandbox`：`list`、`explain`、`recreate`（可加 `--browser`、`--session`、`--agent`）

### 4.11 `cron *` / `hooks *` / `plugins *` / `system *` / `webhooks *` / `backup *`

* • **cron**：`list`、`add`、`edit`、`enable`、`disable`、`rm`、`run`、`runs`、`status`
* • **hooks**：`list`、`enable`、`disable`、`info`、`check`
* • **plugins**：`list`、`install`、`enable`、`disable`、`inspect`、`marketplace`、`update`、`uninstall`、`doctor`
* • **system**：`event`、`heartbeat`、`presence`
* • **webhooks**：如 `gmail`
* • **backup**：`create`、`verify`

### 4.12 `browser *` / `acp *`

* • **browser**：`start`/`stop`/`status`、`snapshot`、`navigate`、`click`、`screenshot`、`tabs`、`cookies`、`evaluate` …
* • **acp**：`acp client` 等（`--token`、`--url`、`--session`），见 https://docs.openclaw.ai/cli/acp

### 4.13 `dns *` 与网关发现

`dns setup`（CoreDNS + 广域发现）、`gateway discover`、`gateway probe`。

### 4.14 `logs` / `doctor` / `docs` / `completion` / `update` / `reset` / `uninstall`

| 命令 | 说明 |
| --- | --- |
| `logs` | `--follow` 、`--limit`、`--json`、`--token`、`--url` |
| `doctor` | 体检；支持 `--fix` / `--repair`、`--deep`、`--force`、`--non-interactive` 等（见 `doctor --help`） |
| `docs <query...>` | 搜在线文档 |
| `completion` | `-s zsh |
| `update` | 自更新；`update status` / `update wizard`；`--channel stable |
| `reset` | `--scope` 、`--dry-run` |
| `uninstall` | `--service` / `--state` / `--workspace` / `--all` 等 |

### 4.15 Skills / `agents *`

| 命令 | 说明 |
| --- | --- |
| `skills list` / `info` / `check` | 列出、查看详情、检查依赖是否就绪 |
| `skills search` / `install` / `update` | 搜 ClawHub、安装到当前 workspace、更新已装 Skills |
| `agents …` | 多 Agent、workspace、路由 —— `agents --help` |

以上 `skills` 子命令为 OpenClaw **运行时** 能力（与编辑器里的 Cursor Skill 不是同一套）。

**安装**：`npm install -g openclaw` 或官方脚本（Getting Started）。

---

## 5. 推荐首次闭环（最短路径）

1. 1. 安装 CLI → `openclaw onboard`（需要系统服务时加 `--install-daemon`）
2. 2. 配置模型与至少一个 Channel
3. 3. 启动 Gateway：`gateway run` 或 `gateway start`（亦可用 `daemon start`）
4. 4. 新用户 DM 配对 → `pairing approve …`（语法见 `pairing approve --help`）
5. 5. 验证任选：`openclaw tui`、`openclaw agent -m "你好"`、`openclaw dashboard`

---

## 6. Workspace 与 Prompt 文件（记名字即可）

| 文件 | 典型用途 |
| --- | --- |
| `AGENTS.md` | 运行规则、长期约定 |
| `SOUL.md` | 性格与行为准则 |
| `USER.md` | 用户画像、称呼偏好 |
| `TOOLS.md` | 工具使用习惯与本地约定 |
| `IDENTITY.md` | 身份与风格（可选） |
| `BOOT.md` | 启动层指令（可选） |
| `BOOTSTRAP.md` | 首次引导（一次性） |
| `MEMORY.md` / `memory/` | 可选的长期笔记型结构 |

Skills 安装后，工具说明会进入 **TOOLS.md** 等上下文；MCP 在配置中挂载后由 Tool Engine 动态发现。

---

## 7. 安全与沙盒（三档）

| 模式 | 含义 | 适用 |
| --- | --- | --- |
| **off** | 沙盒关闭，主机上高信任执行 | 单人、可信机器 |
| **non-main** | 主会话相对信任，其它会话沙盒 | **默认推荐** |
| **all** | 全局沙盒、权限最小化 | 多人 / 更严隔离 |

另需关注：**工具白名单 / 黑名单**、**危险操作审批（exec-approvals 等）**、**配对与权限**、**MCP/Skill 来源可信**。详见 Security、Sandboxing。

---

## 8. Session vs Memory（概念）

|  | Session | Memory |
| --- | --- | --- |
| **是什么** | 当前对话线程内的消息与工具轨迹 | 跨会话持久化的知识与偏好 |
| **典型存储** | `agents/<agentId>/sessions/<sessionId>/` 下 JSONL 等 | `memory/` + 向量检索后端（如 sqlite-vec） |
| **检索** | 随上下文窗口与修剪策略进模型 | 每轮按查询做语义 / 混合检索再注入 |

记忆条目有 **L0 / L1 / L2** 分层（摘要到全文）；可选用 **FSRS** 做复习调度。细节见 `03-session-memory.md`。

---

## 9. 工具扩展三条路

1. 1. **内置工具**：终端、读写、浏览器、网页搜索等
2. 2. **Skills**：npm 包形态，manifest 声明工具，配置启用
3. 3. **MCP Server**：标准协议，动态挂载工具

---

## 10. 常用常量（心里有数）

| 常量 | 典型值 | 含义 |
| --- | --- | --- |
| `MAX_TOOL_ROUNDS` | 20 | 单轮用户消息内工具循环上限 |
| `TOOL_TIMEOUT_MS` | 60000 | 单次工具超时（毫秒级，以实现为准） |
| `PAIRING_CODE_EXPIRY` | 约 10 分钟 | 配对码有效期 |
| `APPROVAL_TIMEOUT` | 约 5 分钟 | 审批等待超时 |

---

## 11. 排错速查

| 现象 | 先做 |
| --- | --- |
| 连不上 / 无响应 | `openclaw gateway status` 、`openclaw gateway probe`、`openclaw doctor`；TUI 连不上时加 `--token` 或检查 `gateway.remote.url` |
| 渠道收不到消息 | `openclaw status` 、`openclaw channels list`、查网关日志 |
| 权限 / 沙盒 / 命令被拒 | 看安全配置与审批提示；收紧前先理解 `non-main` vs `all` |
| 配置糊涂了 | `openclaw config` 、对照 `~/.openclaw/openclaw.json` |
| 彻底重来 | `openclaw reset` （会清状态，先备份） |

---

## 12. 官方与源码

* • 文档：https://docs.openclaw.ai/
* • 仓库：https://github.com/openclaw/openclaw
* • 架构概念：https://docs.openclaw.ai/concepts/architecture