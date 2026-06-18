---
title: Claude Code 不再空跑tools 提效 提效
author: 安全手札
date: c4bbagec4bbage
url: https://mp.weixin.qq.com/s?__biz=MzUzMzQ4OTk0OA==&mid=2247483872&idx=1&sn=99cc93e9290c393ffa0355ed2f810fdd&chksm=fb363109700aaff85c09275fd67a45372320e951c5650955a42be36f0738e954c9748c13052e&mpshare=1&scene=24&srcid=0509glnBbzGt1aOBVyHDMRQa&sharer_shareinfo=eb67704b15ee0d44da09906f66742dbc&sharer_shareinfo_first=eb67704b15ee0d44da09906f66742dbc#rd
---

> 基于源码逆向分析，从代码层面还原"Claude Code 真正需要什么" 为什么写这篇文章呢？是在日常使用 Claude Code 过程中，发现一些调用失败的情况，比如喜欢用gh rg等工具，但是环境没有安装，导致调用失败，所以决定写这篇文章，帮助大家更好地使用 Claude Code。
>
> 适用人群：想让 Claude Code 发挥 100% 能力的开发者

---

## 快速开始：一键安装 Prompt

**把下面这段话直接丢给 Claude Code，让它帮你自动检测并安装所有依赖。复制粘贴即可。文章也不用看了。**

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line帮我检查并安装 Claude Code 所需的全部依赖工具，按以下步骤执行：  
第一步：环境检测逐一检查以下工具是否已安装，列出已装/未装的清单：- git、ripgrep (rg)、gh (GitHub CLI)、tmux、poppler (pdftotext)- ast-grep (sg)、fd、jq、git-delta (delta)- typescript-language-server、tsc、pyright  
第二步：安装缺失工具根据当前操作系统（macOS 用 brew，Ubuntu/Debian 用 apt + 手动安装），安装所有缺失的工具：  
必装（缺一个 Claude Code 就会功能受限）：- git：版本控制基础，Claude Code 启动时依赖- ripgrep (rg)：Grep 工具的底层实现，不装则 Grep 工具直接失效- gh (GitHub CLI)：所有 GitHub PR/Issue/CI 操作的入口- tmux：多 Agent 并行模式（Swarm）和后台任务必须- poppler：PDF 文件读取支持  
推荐（代码搜索和处理效率提升）：- ast-grep (sg)：结构化代码搜索，精度远超纯文本搜索- fd：现代 find 替代，自动跳过 .gitignore- jq：JSON 命令行处理- git-delta：更好的 diff 显示  
语言服务器（按需安装，TypeScript 项目优先）：- npm install -g typescript typescript-language-server pyright  
第三步：配置验证- 运行 gh auth status 检查 GitHub CLI 是否已登录，未登录则执行 gh auth login- 逐一验证所有已安装工具的版本号- 输出最终的安装状态报告  
第四步：权限预配置检查 ~/.claude/settings.json 是否存在。如果不存在，创建以下配置来预授权常用只读命令（减少使用时频繁弹框确认）：  
{  "permissions": {    "allow": [      "Bash(git status)",      "Bash(git diff*)",      "Bash(git log*)",      "Bash(git show*)",      "Bash(git branch*)",      "Bash(git stash*)",      "Bash(gh pr*)",      "Bash(gh issue*)",      "Bash(gh repo*)",      "Bash(rg *)",      "Bash(fd *)",      "Bash(sg *)",      "Bash(jq *)",      "Bash(cat *)",      "Bash(ls *)",      "Bash(find *)",      "Bash(node --version)",      "Bash(npm run *)",      "Bash(pnpm *)",      "Bash(yarn *)"    ]  }}  
如果文件已存在，把上面的 allow 列表合并进去，不要覆盖已有配置。  
注意事项：- 安装过程中如果某个工具安装失败，跳过继续安装其余工具，最后统一报告失败项- 不要修改任何全局 git config（user.name/user.email 等）- gh auth login 需要交互操作，提示我手动完成即可
```

---

## 为什么要读这篇文章？

很多人装好 Claude Code 就直接用，觉得"能对话就行了"。但实际上，**一个未经配置的 Claude Code 就像一个没带工具箱的工程师** —— 他有能力，但什么也干不了。

这篇文章要解决三个核心问题：

### 问题一：为什么裸装 Claude Code 体验很差？

Claude Code 的架构本质是 **"大脑 + 工具"**。大脑是 Claude 模型本身，工具是它在你机器上能调用的外部命令。源码系统提示里写得很直白：

```
ounter(lineounter(lineounter(lineDo NOT use the Bash tool to run commands when a relevant dedicated tool is provided.Using dedicated tools allows the user to better understand and review your work.This is CRITICAL to assisting the user.
```

如果你没装 `ripgrep`，Claude Code 的 `Grep` 工具直接**无法工作**。没装 `gh`，所有 GitHub 操作要退化成手动拼 API 调用。没装 `poppler`，PDF 文件就是一个黑盒。每少装一个工具，Claude 的能力就少一块。

### 问题二：正确配置真的能省 Token、省钱吗？

**能，而且效果显著。** 这不是玄学，是源码级的机制保证：

**1. 专用工具的输出是结构化的、有上限的**

源码 `FileReadTool/limits.ts` 明确定义：

```
ounter(lineounter(lineounter(lineounter(line| limit         | default | on overflow     ||---------------|---------|-----------------|| maxSizeBytes  | 256 KB  | throws pre-read || maxTokens     | 25000   | throws post-read|
```

Read 工具最多返回 25,000 token 的内容，超出会报错让 Claude 用更精确的方式读取（指定行号范围）。而如果用 `cat` 命令通过 Bash 读文件，一个大文件轻松灌入几万 token 的原始输出，全部计入你的上下文窗口。

**2. 专用工具避免"Bash 绕路"的额外开销**

当 Claude 用 `Grep` 工具搜索代码时，它直接调用 `rg` 并以结构化 JSON 返回结果。但如果 `rg` 没装，Claude 只能退而求其次用 Bash 跑 `grep -r`，不仅更慢，输出也是杂乱的原始文本，需要更多 token 来理解。

**3. 权限预授权减少确认轮次**

每次 Claude 想执行一个命令，都需要一次 API 交互。源码中定义了大量"只读安全命令白名单"（`GIT_READ_ONLY_COMMANDS`、`RIPGREP_READ_ONLY_COMMANDS` 等），**这些命令不需要用户确认就能执行**。一次确认 = 一次额外的 API 轮次 = 额外的 input token 重发。装好工具 + 配好权限，可以把很多操作从"需要确认"降级为"静默执行"。

**4. Auto-Compact 机制与上下文管理**

源码里有精密的上下文管理系统。当 token 使用量接近上下文窗口限制时，会触发 auto-compact（自动压缩），把历史对话浓缩成摘要。**如果你的每次工具调用都产生大量冗余输出，compact 就会更频繁地触发**，每次 compact 本身也要消耗 token。减少单次工具输出量 → 减少 compact 频率 → 省 token。

**5. Microcompact：静默清理旧工具结果**

源码中还有一个 `microcompact` 机制 —— 在发送 API 请求前，自动清理掉旧的工具执行结果，只保留最近的几次。这意味着**即使之前的工具调用产生了大量输出，这些输出也会在后续轮次被清理掉**，避免上下文无限膨胀。专用工具的结构化输出天然更容易被高效 compact。

### 问题三：配置好能提升多少执行效率？

从源码角度看，效率提升来自三个维度：

**减少 API 轮次（最直接的效率提升）**

```
ounter(lineounter(line未配置：用户提问 → Claude 思考 → 尝试执行 → 权限确认弹框 → 用户确认 → 继续执行已配置：用户提问 → Claude 思考 → 直接执行（安全白名单内无需确认）
```

每省一次确认，就省一次完整的 API 往返。在一个复杂任务中，可能涉及几十次工具调用，差距会被放大。

**并行工具调用**

源码系统提示明确要求 Claude 尽量并行调用工具：

```
ounter(lineounter(lineounter(lineYou can call multiple tools in a single response.Make all independent tool calls in parallel.Maximize use of parallel tool calls where possible to increase efficiency.
```

专用工具支持并行执行（同时读多个文件、同时搜索多个目录），而 Bash 工具通常是串行的。工具越丰富，并行机会越多。

**子 Agent 隔离上下文**

源码中 `AgentTool` 的设计理念是：把复杂的子任务交给子 Agent 执行，**子 Agent 的工具输出不会污染主对话的上下文窗口**。这意味着一个搜索了 100 个文件的子 Agent，只需要把最终结论返回给主线程，而不是把 100 个文件的内容全塞进来。

```
ounter(lineounter(lineounter(lineCalling Agent without a subagent_type creates a fork, which runs in the backgroundand keeps its tool output out of your context — so you can keep chatting with the userwhile it works.
```

### 量化估算

| 场景 | 未配置 | 配置完善 | 节省 |
| --- | --- | --- | --- |
| 读一个 500 行文件 | Bash `cat` → ~2000 token 原始输出 | Read 工具 → 结构化输出 + 行号，可指定范围 | ~30-50% token |
| 搜索代码中的函数 | Bash `grep -r` → 大量冗余匹配 | Grep(rg) → 精确匹配 + 上下文行控制 | ~40-60% token |
| 查看 git 状态 | 需确认 → 额外一轮 API 往返 | 白名单内 → 零确认直接执行 | 省一整轮 API 调用 |
| 处理 PR + Issue | 手动拼 API URL → 多轮尝试 | `gh` 命令 → 一步到位 | 省 2-5 轮 API 调用 |
| 长会话（50+ 轮） | 频繁 compact，丢失上下文 | 输出精简，compact 间隔更长 | 上下文保持更完整 |

---

## 核心理念：Claude 有两种工具

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line专用工具（Dedicated Tools）  Read / Edit / Write / Glob / Grep / Agent ...  → Claude 优先使用，用户可以 review 每次操作  
Bash 工具（兜底）  → 没有专用工具时才用  → 源码原话：Do NOT use Bash when a relevant dedicated tool is provided. This is CRITICAL.
```

外部系统工具（如 git、gh、rg）是 Bash 工具的"弹药"。装得越全，Claude 能做的事越多。

理解这个架构至关重要：**专用工具走的是优化过的路径**（结构化输入输出、token 预算控制、结果可 review），**Bash 是兜底方案**（原始文本输出、没有 token 控制、用户难以 review）。你装的工具越完整，Claude 走优化路径的比例就越高。

---

## 一、必装工具

### 1. git（无需多说）

```
ounter(linegit --version
```

Claude Code 在启动时调用 `getIsGit()`、`getBranch()`、`findGitRoot()` 等数十个函数，整个工作流都建立在 git 之上。

**关键配置**：

```
ounter(lineounter(lineounter(line# Claude 会读取这些信息来跟随项目风格git config --global user.name "你的名字"git config --global user.email "your@email.com"
```

---

### 2. GitHub CLI（gh）：强烈推荐

```
ounter(lineounter(linebrew install ghgh auth login
```

**为什么必装？** 源码 `BashTool/prompt.ts` 明确写道：

```
ounter(lineounter(lineounter(lineUse the gh command via the Bash tool for ALL GitHub-related tasksincluding working with issues, pull requests, checks, and releases.If given a Github URL use the gh command to get the information needed.
```

Claude Code 在启动时通过 `getGhAuthStatus()` 检查：

* gh 是否已安装
* 是否已登录

**没有 gh**：Claude 处理 PR、Issue、CI 状态时会非常受限。

验证：

```
ounter(lineounter(linegh auth statusgh pr list  # 能正常输出就 OK
```

---

### 3. ripgrep（rg）：Grep 工具的实体

```
ounter(lineounter(linebrew install ripgreprg --version
```

**重要**：Claude Code 的 `Grep` 工具底层直接调用 `rg`。源码里有专门的 `RIPGREP_READ_ONLY_COMMANDS` 配置，把 ripgrep 纳入免权限确认的安全命令白名单。

没有 ripgrep，`Grep` 工具无法工作。

**对 Token 的影响**：Grep 工具支持 `output_mode`（content / files\_with\_matches / count）和 `head_limit` 参数，可以精确控制返回结果的粒度。而用 Bash 跑 `grep -r`，结果是不可控的原始文本流，可能一次灌入几万 token。仅这一个工具的差异，就能在代码搜索场景下节省 40-60% 的 token 消耗。

---

### 4. tmux：Swarm 多 Agent 模式必须

```
ounter(lineounter(lineounter(linebrew install tmux           # macOSsudo apt install tmux       # Ubuntu/Debiansudo dnf install tmux       # Fedora/RHEL
```

源码错误提示：

```
ounter(linereturn 'Install tmux with: brew install tmux'
```

**什么时候需要**：

* 使用 `--worktree` 隔离模式
* Swarm 模式（多个 Claude Agent 并行工作）
* 会话后台化（Shift+Esc）

---

### 5. poppler：PDF 阅读支持

```
ounter(lineounter(linebrew install poppler           # macOSsudo apt install poppler-utils  # Ubuntu
```

源码 `FileReadTool.ts`：

```
ounter(lineounter(lineounter(linePage extraction requires poppler-utils:install with `brew install poppler` on macOSor `apt-get install poppler-utils` on Debian/Ubuntu.
```

**没有 poppler**：Claude 无法读取 PDF 文件内容。

---

## 二、代码搜索工具套件

> 这套工具解决的核心问题：**让 Claude 用更少的 Token 找到更精确的信息**。搜索工具的质量直接决定了 Claude 在理解你代码库时的 Token 效率 —— 精准的搜索意味着更少的误报、更少的结果需要 Claude 去"阅读理解"，从而节省大量的上下文空间。

### ast-grep（sg）：结构化代码搜索

```
ounter(lineounter(linebrew install ast-grepsg --version
```

**与 ripgrep 的本质区别**：

```
ounter(lineounter(lineripgrep:  搜索文本字符串（可能误报注释、字符串内容）ast-grep: 搜索代码结构（精确匹配语法树）
```

实际场景对比：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 找所有 console.log 调用（不含注释和字符串里的）rg "console\.log"      # 有误报sg -p 'console.log($$$)' --lang js  # 精确  
# 找有两个参数的函数调用sg -p 'foo($A, $B)' --lang ts  # rg 无法做到  
# 结构化重构：把 var 改成 let（不动注释里的 var）sg -p 'var $NAME = $VAL' -r 'let $NAME = $VAL' --lang js
```

**用 ast-grep 写自定义 Lint 规则**：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# ~/.claude/rules/no-console.yamlid: no-console-loglanguage: TypeScriptrule:  pattern: console.log($$$)message: "请用 logger 替代 console.log"severity: error
```

```
ounter(linesg scan -r ~/.claude/rules/ ./src
```

---

### fd：现代 find 替代品

```
ounter(linebrew install fd
```

```
ounter(lineounter(lineounter(line# Claude 在 Glob 工具之外也会用 fdfd --extension ts --exclude node_modulesfd -t f -e tsx src/components/
```

比系统 find 快 3-5 倍，自动跳过 `.gitignore` 中的文件。

---

### jq：JSON 命令行处理

```
ounter(linebrew install jq
```

Claude 处理 API 返回、`package.json`、配置文件时会频繁用到：

```
ounter(lineounter(lineounter(line# 这类操作 Claude 会直接用 jqgh api repos/owner/repo/issues | jq '.[].title'cat package.json | jq '.dependencies'
```

---

## 三、语言服务器（LSP）

> LSP 解决的核心问题：**让 Claude 不再需要"猜"类型和引用关系**。没有 LSP 时，Claude 只能通过搜索文本来推断类型、查找定义、跟踪引用，这需要大量的搜索 → 阅读 → 推理的循环，每一步都消耗 Token。有了 LSP，一次调用就能获得精确的类型信息和跳转定义，相当于把 5-10 次搜索压缩成 1 次精确查询。

Claude Code 内置了 `LSPTool`，连接语言服务器来获取：

* 精准的类型信息
* 跳转到定义
* 引用查找
* 实时错误检测

### TypeScript / JavaScript

```
ounter(linenpm install -g typescript-language-server typescript
```

验证：

```
ounter(lineounter(linetypescript-language-server --versiontsc --version
```

### Python

```
ounter(lineounter(lineounter(lineounter(line# 推荐：pyright（源码中有专门的命令规格定义）npm install -g pyright# 或pip install pyright
```

源码里 `pyright` 有专门的 `PYRIGHT_READ_ONLY_COMMANDS` 配置，说明 Claude 可以直接调用类型检查。

### Go

```
ounter(lineounter(linebrew install gogo install golang.org/x/tools/gopls@latest
```

### Rust

```
ounter(lineounter(linecurl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | shrustup component add rust-analyzer
```

---

## 四、Git 体验增强

### git-delta：更好的 diff 显示

```
ounter(linebrew install git-delta
```

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# ~/.gitconfig[core]    pager = delta[delta]    navigate = true    side-by-side = true    line-numbers = true
```

Claude 查看 diff 时（`git diff`、`git show`）会更清晰。

### lazygit（可选）

```
ounter(linebrew install lazygit
```

当 Claude 输出 git 操作建议时，用 lazygit 可以更直观地验证和操作。

---

## 五、CLAUDE.md 配置

### 全局规则：`~/.claude/CLAUDE.md`

适合放你对所有项目通用的偏好：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 我的全局 Claude 规则  
## 代码风格- 用中文写注释- 函数不超过 50 行，超了就拆分- 不用 any 类型，找合适的类型替代  
## Git 规范- commit message 用中文，格式：feat/fix/refactor: 描述- 每个 commit 只做一件事  
## 禁止事项- 不要在生产代码里用 console.log- 不要用 == 用 ===- 不要用 var 用 const/let  
## 工具偏好- 搜索代码先用 Grep/Glob，不要用 Bash- 写新文件前先 Read 同类文件参考风格
```

### 项目规则：`项目根目录/CLAUDE.md`

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 项目名称 - Claude 规则  
## 项目技术栈- 前端：React 18 + TypeScript + Tailwind- 后端：Node.js + Fastify + Prisma- 测试：Vitest + Playwright  
## 重要约定- API 路由统一在 src/routes/ 下- 所有数据库操作通过 src/db/ 的封装- 错误处理用 src/utils/errors.ts 的工具函数  
## 运行命令- 开发：pnpm dev- 测试：pnpm test- 类型检查：pnpm typecheck- Lint：pnpm lint  
## 不要动的文件- .env.production（生产配置）- prisma/migrations/（数据库历史）
```

---

## 六、settings.json 权限预配置

提前配置常用命令的权限，减少频繁弹框打断。**这是降低 Token 消耗最立竿见影的配置之一。**

每次权限确认弹框，都意味着一次额外的 API 轮次：Claude 发出工具调用请求 → 等待用户确认 → 用户确认 → Claude 继续执行。在这个过程中，整个对话历史（所有 input token）都要重新发送一遍。如果一个任务涉及 20 次命令执行，每次都要确认，你实际上多支付了 20 次 input token 的费用。

路径：`~/.claude/settings.json`（全局）或 `.claude/settings.json`（项目）

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line{  "permissions": {    "allow": [      "Bash(git status)",      "Bash(git diff*)",      "Bash(git log*)",      "Bash(git show*)",      "Bash(git branch*)",      "Bash(git stash*)",      "Bash(gh pr*)",      "Bash(gh issue*)",      "Bash(gh repo*)",      "Bash(rg *)",      "Bash(fd *)",      "Bash(sg *)",      "Bash(jq *)",      "Bash(cat *)",      "Bash(ls *)",      "Bash(find *)",      "Bash(node --version)",      "Bash(npm run *)",      "Bash(pnpm *)",      "Bash(yarn *)"    ]  }}
```

**说明**：`Bash(git diff*)` 表示所有以 `git diff` 开头的命令都预授权，不需要逐一确认。

---

## 七、MCP 服务器扩展

> MCP 解决的核心问题：**让 Claude 直接与外部系统交互，而不是通过 Bash 间接操作**。比如操作数据库，没有 MCP 时 Claude 要通过 Bash 拼 SQL 命令，输出是原始文本；有了 SQLite/Postgres MCP，操作和结果都是结构化的，Token 消耗更低，错误率也更低。

MCP（Model Context Protocol）是 Claude 的工具扩展协议。配置在项目根目录的 `.mcp.json`：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line{  "mcpServers": {    "playwright": {      "command": "npx",      "args": ["@playwright/mcp@latest", "--headless"]    },    "sqlite": {      "command": "npx",      "args": ["@modelcontextprotocol/server-sqlite", "./db.sqlite"]    },    "filesystem": {      "command": "npx",      "args": ["@modelcontextprotocol/server-filesystem", "/path/to/dir"]    }  }}
```

**常用 MCP 服务器**：

| MCP 服务器 | 用途 | 安装 |
| --- | --- | --- |
| `@playwright/mcp` | 浏览器自动化、JS 页面抓取 | `npx @playwright/mcp@latest` |
| `@modelcontextprotocol/server-sqlite` | SQLite 数据库操作 | npm |
| `@modelcontextprotocol/server-postgres` | PostgreSQL 操作 | npm |
| `@modelcontextprotocol/server-github` | GitHub API 增强 | npm |
| `@modelcontextprotocol/server-filesystem` | 扩展文件系统访问范围 | npm |

---

## 八、Claude Code 源码揭示的内部安全白名单

> 理解白名单的意义：**每一个在白名单内的命令，都意味着 Claude 可以"免确认"直接执行**。从 Token 角度看，免确认 = 省一次 API 轮次 = 省一次 input token 重发。在一个有 50K token 对话历史的会话中，省一次确认就是省 50K input token。如果一个任务涉及 10 次 git 状态查询，这就是省 500K token。

以下命令在源码中被列为"只读安全命令"，Claude 执行时**无需用户确认**：

**Git 只读命令**（`GIT_READ_ONLY_COMMANDS`）：

```
ounter(lineounter(lineounter(linegit status, git log, git diff, git show, git branch,git tag, git remote, git stash list, git blame,git reflog, git shortlog, git describe, git rev-parse ...
```

**GitHub CLI 只读命令**（`GH_READ_ONLY_COMMANDS`）：

```
ounter(lineounter(linegh pr list/view/status, gh issue list/view,gh repo view, gh run list/view, gh release list ...
```

**Docker 只读命令**（`DOCKER_READ_ONLY_COMMANDS`）：

```
ounter(linedocker ps, docker images
```

**ripgrep**（`RIPGREP_READ_ONLY_COMMANDS`）：

```
ounter(linerg（所有搜索参数）
```

**pyright**（`PYRIGHT_READ_ONLY_COMMANDS`）：

```
ounter(linepyright（类型检查）
```

> 这意味着：只要你装好了这些工具，Claude 在做代码分析、查看历史、搜索内容时完全不会打断你。

---

## 九、一键安装脚本

### macOS

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line#!/bin/bashset -e  
echo "=== 安装 Claude Code 依赖工具 ==="  
# 检查 Homebrewif ! command -v brew &>/dev/null; then  echo "安装 Homebrew..."  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"fi  
echo "--- 必装工具 ---"brew install git ripgrep tmux poppler gh ast-grep fd jq git-delta  
echo "--- 登录 GitHub CLI ---"gh auth login  
echo "--- TypeScript LSP ---"npm install -g typescript typescript-language-server pyright  
echo "--- 安装 Playwright 浏览器（用于网页抓取）---"npx playwright install chromium  
echo ""echo "=== 验证安装 ==="echo -n "git: ";      git --versionecho -n "gh: ";       gh --version | head -1echo -n "rg: ";       rg --version | head -1echo -n "tmux: ";     tmux -Vecho -n "ast-grep: "; sg --versionecho -n "fd: ";       fd --versionecho -n "jq: ";       jq --versionecho -n "tsc: ";      tsc --versionecho ""echo "✓ 全部安装完成！"
```

### Ubuntu / Debian

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line#!/bin/bashset -e  
# 基础工具sudo apt updatesudo apt install -y git tmux poppler-utils jq fd-find  
# ripgrepcurl -LO https://github.com/BurntSushi/ripgrep/releases/latest/download/ripgrep_14.1.0-1_amd64.debsudo dpkg -i ripgrep_14.1.0-1_amd64.deb  
# GitHub CLItype -p curl >/dev/null || sudo apt install curl -ycurl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \  | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpgecho "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \  | sudo tee /etc/apt/sources.list.d/github-cli.listsudo apt update && sudo apt install gh -ygh auth login  
# ast-grepcargo install ast-grep  # 需要先装 Rust# 或下载预编译版本  
# Node.js 工具npm install -g typescript typescript-language-server pyright
```

---

## 十、工具重要度总览

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line★★★★★ 必须安装  git            版本控制基础  ripgrep (rg)   Grep 工具的实体，不装就失效  
★★★★☆ 强烈推荐  gh             GitHub 所有操作的入口  tmux           多 Agent / 后台任务必须  poppler        PDF 读取支持  
★★★☆☆ 代码搜索套件  ast-grep (sg)  结构化代码搜索/重构，精度远超 rg  fd             现代 find，自动跳过 gitignore  jq             JSON 处理必备  
★★☆☆☆ 语言服务器（按需）  typescript-language-server   TypeScript/JS 项目  pyright                      Python 项目（类型检查）  gopls                        Go 项目  rust-analyzer                Rust 项目  
★★☆☆☆ 体验增强  git-delta      漂亮的 diff 显示  lazygit        Git 可视化 TUI
```

---

## 附录：快速验证脚本

把这段存为 `check-claude-tools.sh`，随时运行检查：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line#!/bin/bash  
check() {  if command -v "$1" &>/dev/null; then    echo "✓ $1: $(command -v $1)"  else    echo "✗ $1: 未安装"  fi}  
echo "=== Claude Code 工具检查 ==="echo ""echo "--- 必须 ---"check gitcheck rgcheck ghcheck tmuxecho ""echo "--- 推荐 ---"check popplercheck sgcheck fdcheck jqecho ""echo "--- LSP ---"check typescript-language-servercheck tsccheck pyrightcheck goplscheck rust-analyzerecho ""echo "--- 体验增强 ---"check deltacheck lazygitecho ""  
# gh 认证状态if command -v gh &>/dev/null; then  if gh auth token &>/dev/null 2>&1; then    echo "✓ gh: 已认证登录"  else    echo "⚠ gh: 已安装但未登录，运行 gh auth login"  fifi
```

```
ounter(lineounter(linechmod +x check-claude-tools.sh./check-claude-tools.sh
```

---

## 附录 B：深入理解 Token 经济学

### Claude Code 的 Token 消耗结构

一次完整的 Claude Code 交互，Token 消耗分布大致如下：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line┌─────────────────────────────────────────────────┐│  System Prompt（系统提示）   ~3000-5000 token    │  ← 每轮都要发送，但有缓存├─────────────────────────────────────────────────┤│  Tool Definitions（工具定义）~2000-4000 token    │  ← 装的工具越多越大，但有缓存├─────────────────────────────────────────────────┤│  CLAUDE.md 规则              ~500-2000 token     │  ← 每轮都要发送，但有缓存├─────────────────────────────────────────────────┤│  对话历史                    动态增长             │  ← 这是主要消耗│  ├─ 用户消息                                     ││  ├─ Claude 回复                                  ││  └─ 工具调用结果             ← 这是关键变量       │  ← 占比最大、波动最大├─────────────────────────────────────────────────┤│  Output Token（Claude 输出） 每轮 200-2000       │└─────────────────────────────────────────────────┘
```

**工具调用结果**是唯一一个你能通过配置直接影响的变量。用 Read 工具读文件，最多 25,000 token；用 `cat` 通过 Bash 读文件，没有上限。用 Grep 工具搜索，可以控制 `head_limit`；用 `grep -r` 通过 Bash，输出完全不可控。

### Prompt Cache：省钱的核心机制

Claude API 有 prompt cache 机制 —— 如果你的请求前缀（system prompt + 工具定义 + CLAUDE.md）和上一次请求相同，这部分 token 走缓存读取，**成本只有正常 input token 的 10%**。

源码中的 `promptCacheBreakDetection.ts` 专门监控缓存命中率。**当你修改 settings.json 权限配置或 CLAUDE.md 时，可能会打破缓存前缀**，导致一次完整的缓存重建（约 5000-10000 token 的 cache\_creation）。所以：

* 配置改好之后尽量不要频繁调整
* System prompt 的静态部分和动态部分之间有明确的 `BOUNDARY MARKER`，保证动态内容变化不会影响静态部分的缓存

### Auto-Compact：上下文满了怎么办

当对话 token 接近上下文窗口限制时，Claude Code 会自动触发 compact（压缩）：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line上下文窗口 = 200K token（Sonnet）compact 阈值 ≈ 上下文窗口 - 20K（预留输出空间）  
当 token 使用 > 阈值 → 触发 auto-compact  → 调用模型对历史对话生成摘要  → 摘要替换原始对话  → 上下文从 180K 降到 ~10K  → 可以继续对话
```

问题在于：**每次 compact 本身就是一次完整的 API 调用**，要把所有历史对话作为 input 发给模型做总结。如果你的工具输出很臃肿，上下文膨胀更快，compact 更频繁，花的钱更多。

源码中还有更精细的 **microcompact** 机制，会在每次 API 调用前自动清理旧的工具结果（只保留最近几轮的输出）。这意味着那些产生了大量输出的 Bash 命令，它们的结果最终也会被清理掉 —— 但你已经为发送它们付过一次费了。

### 一个真实场景的 Token 对比

假设你让 Claude 在一个 TypeScript 项目中重构一个函数：

**未配置环境（工具缺失、无权限预授权）**：

| 步骤 | 操作 | Token 消耗 |
| --- | --- | --- |
| 1 | Claude 想搜索函数引用，没有 rg，用 `grep -r` | 需确认 +500，结果 +3000 |
| 2 | 搜索结果太多，Claude 需要过滤 | +2000 output |
| 3 | 读取每个文件（用 `cat`） | 每个文件 +2000，5 个文件 = +10000 |
| 4 | 确认 git status | 需确认 +500，结果 +200 |
| 5 | 编辑文件 | +1000 |
| 6 | 确认 git diff | 需确认 +500，结果 +1000 |
| **合计** |  | **~18,700 token** （工具结果部分） |

**配置完善的环境**：

| 步骤 | 操作 | Token 消耗 |
| --- | --- | --- |
| 1 | Grep 工具搜索函数引用（精确匹配 + head\_limit） | 结果 +800 |
| 2 | Read 工具读取相关文件（可指定行号范围） | 每个文件 +500，5 个文件 = +2500 |
| 3 | git status（白名单，无需确认） | 结果 +200 |
| 4 | FileEdit 工具编辑文件 | +500 |
| 5 | git diff（白名单，无需确认） | 结果 +800 |
| **合计** |  | **~4,800 token** （工具结果部分） |

**节省约 74% 的工具结果 Token，同时减少了 3 次权限确认轮次。**

---

## 总结：投入 30 分钟配置，换来持续的效率红利

配置 Claude Code 不是一次性的麻烦，而是一次投资。总结一下核心收益：

| 维度 | 不配置 | 配置完善 |
| --- | --- | --- |
| **Token 消耗** | 工具结果臃肿，上下文膨胀快 | 结构化输出 + token 上限控制，节省 30-70% |
| **API 轮次** | 频繁权限确认，每次多一轮往返 | 白名单静默执行，减少 50%+ 轮次 |
| **执行速度** | 串行 Bash，一个一个命令跑 | 专用工具并行执行，速度提升 2-3x |
| **上下文质量** | 原始文本输出，compact 频繁，信息丢失 | 精简输出，compact 间隔长，上下文更完整 |
| **能力范围** | 只能用 Bash 做所有事 | PDF 阅读、LSP 类型检查、结构化代码搜索全部解锁 |
| **可 Review 性** | Bash 输出难以审查 | 专用工具操作清晰可见，用户可以逐个确认 |

**花 30 分钟跑一遍安装脚本，每次使用都在省 Token、省时间、省心智负担。这是 ROI 最高的 30 分钟。**