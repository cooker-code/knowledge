---
title: OpenClaw架构-OpenClaw 安全模型：威胁边界、隔离策略与可审计执行控制
author: 彼岸流天
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5NDE1MzYxMg==&mid=2247484904&idx=1&sn=b3eb2190ad3cedc95be30a415c955a4c&chksm=c1ab3471fb431ccf65aa1dc310596652df8a4896a3bf036ff43d00db3348762c96f96ab94a13&mpshare=1&scene=24&srcid=0327zHX7SmXa3Ivo4dfKsTZ2&sharer_shareinfo=f57dc43115ea77a9176b895e5fd597df&sharer_shareinfo_first=f57dc43115ea77a9176b895e5fd597df#rd
---

## 摘要

OpenClaw 的安全模型面向自托管场景：在允许 Agent 执行必要能力（文件读写、命令执行、网络访问、API 调用）的同时，将高风险操作约束为**可隔离、可审批、可追溯**的事件。

* • **目标（Goals）**：降低 T1~T4（幻觉、注入、未授权入口、恶意扩展）导致的破坏性操作与数据外泄风险；在可用性与安全性之间提供分级策略与默认安全行为。
* • **非目标（Non-goals）**：不对抗本地攻击者（物理访问或已获取 SSH 权限，对应 T5）。
* • **关键控制面**：Sandbox（执行隔离与资源/网络/文件边界）、Tool Policy（按 Agent 实例最小权限）、`exec-approvals`（命令级审批与拒绝）、凭证隔离与日志脱敏、安全审计（JSONL）。
* • **默认推荐**：`sandbox.mode=non-main` + “职责拆分的 Tool Policy” + “默认 requireApproval 的 `exec-approvals`” + “出站白名单 + 元数据/内网/回连禁用”。

## 0. 适用范围与边界（Scope）

* • **适用范围**：OpenClaw Gateway/Agent 侧的工具调用安全控制（含 `exec`、文件访问、网络出站、MCP/Skill 扩展交互）。
* • **显式不覆盖**：本地攻击者（物理访问或已获取 SSH 权限，对应 T5）。在该前提下，防御重点为 T1~T4 的误操作与外部入口风险，并将“不可逆行为”收敛到人工审批与审计闭环内。

## 1. 威胁模型（Threat Model）

威胁分类如下（与安全控制面一一对应）：

* • **T1：模型幻觉**：误判用户意图导致误操作。
* • **T2：Prompt Injection（间接）**：网页/文件/扩展返回值携带诱导指令，影响模型决策与工具调用（“数据伪装成指令”）。
* • **T3：未授权用户（外部）**：未配对用户通过 Channel 或 API 触发会话、建立连接或诱导执行。
* • **T4：恶意 MCP Server / Skill**：第三方扩展窃取数据/凭证或伪造返回值引导后续操作。
* • **T5：本地攻击者**：不在防御目标内（自托管前提：操作者可信）。

## 2. 控制面分层（Control Planes）

OpenClaw 采用三层叠加控制：

* • **Sandbox**：决定执行位置（主机/沙盒）、文件可见范围与网络边界。
* • **Tool Policy**：按 Agent 实例配置工具 allow/deny（支持通配符），并按规则优先级进行匹配。
* • **exec-approvals**：对 `exec` 命令进行自动放行/需审批/拒绝（deny > requireApproval > autoApprove > 默认 requireApproval），并将结果写入审计日志。

下面各章节按“能否执行 → 在哪执行 → 执行到什么程度 → 如何追溯”的顺序展开。

## 3. Sandbox 分级策略（OFF / NON-MAIN / ALL）

* • **OFF**：主机直接执行；文件与网络基本不受限；主要依赖 `exec-approvals` 限制高危命令。
* • **NON-MAIN（默认推荐）**：主会话（CLI/WebSocket）主机执行；外部入口（Channel）沙盒执行，用于平衡可用性与外部风险。
* • **ALL**：全部会话在沙盒内执行；文件限制在 Workspace；网络白名单；适用于多人/企业部署。

## 4. 沙盒实现与降级

跨平台实现路径如下：

* • **macOS**：`sandbox-exec`（Seatbelt），通过 profile 限制文件与网络，并显式拒绝敏感路径（如 `~/.ssh`、`~/.gnupg`、OpenClaw `.env`）。
* • **Linux/WSL2**：优先 Docker，其次 bubblewrap（bwrap）；配置强调只挂载 workspace、资源限制、禁网或白名单网络、临时目录隔离等。
* • **降级策略**：当无可用沙盒实现时，降级为主机执行并输出警告，以显式暴露风险状态。

为保证“可评审”，建议在部署文档中明确沙盒决策的**确定性流程**（可复核）：

1. 1. 读取 `sandbox.mode`（`off` / `non-main` / `all`）。
2. 2. `off`：直接主机执行。
3. 3. `non-main`：按来源区分；CLI/WebSocket→主机执行，Channel→沙盒执行。
4. 4. `all`：统一进入沙盒执行。
5. 5. 沙盒执行时探测可用实现：macOS→`sandbox-exec`；Linux 有 Docker→`docker run`；无 Docker→`bwrap`；WSL2 复用 Linux。
6. 6. 若均不可用：降级为主机执行并产生显式告警；审计中应至少记录“发生降级/原因/最终执行位置”，便于事后复核（例如以 `model_fallback`/`config_change` 类事件或等价字段表达）。

## 5. Tool Policy（最小权限）

Tool Policy 以 Agent 实例为粒度控制工具权限。典型用法是按职责拆分：

* • `main`：允许全部工具
* • `research`：仅允许读取与检索类工具（例如 `web/browser/read/grep`）
* • `code-reviewer`：只读，拒绝 `exec/write/edit`

该机制的目标是减少单一 Agent 的权限面，降低误用与被滥用的概率。

下列片段用于将“最小权限”从概念落到可配置项（便于评审）：

```
{  
  "agents": {  
    "instances": {  
      "main": { "tools": { "allow": ["*"], "deny": [] } },  
      "research": { "tools": { "allow": ["web", "browser", "read", "grep"], "deny": [] } },  
      "code-reviewer": { "tools": { "allow": ["read", "glob", "grep"], "deny": ["exec", "write", "edit"] } }  
    }  
  }  
}
```

## 6. exec-approvals（命令审批）

`exec` 为高风险工具，采用三级审批机制：

* • **autoApprove**：低风险命令自动放行
* • **requireApproval**：潜在破坏性/外部访问/权限提升命令需要审批
* • **deny**：毁灭性命令直接拒绝

审批流程包括命令预处理与标准化、规则匹配、等待审批（含超时）、执行/拒绝、审计记录写入（`audit.jsonl`）。

为避免“描述正确但实现不可核对”，建议在文章中保留一个最小可读的配置样例（字段命名与实现保持一致）：

```
{  
  "exec-approvals": {  
    "autoApprove": ["ls *", "pwd", "whoami", "date", "git status", "git log *", "git diff *"],  
    "requireApproval": ["rm *", "mv *", "cp *", "git push *", "git commit *", "sudo *", "curl *", "wget *", "docker *"],  
    "deny": ["rm -rf /", "rm -rf /*", "rm -rf ~", "mkfs *", "dd if=*", "shutdown *", "reboot *"],  
    "approvalTimeout": 300000  
  }  
}
```

关键语义（建议显式写入评审材料）：

* • **默认行为**：未匹配任何列表 → `requireApproval`（默认收紧，用于对抗 T1）。
* • **优先级**：`deny` 高于 `requireApproval`，避免被更宽泛规则覆盖。
* • **链式/管道命令**：以整条命令按更严格规则处理，防止“前半段匹配 autoApprove、后半段绕过”。

## 7. 文件访问控制

在沙盒 ON（`non-main`/`all`）下，文件访问按 RW/RO/DENY 分层：

* • **RW**：Workspace、Agent 数据目录、临时目录
* • **RO**：系统工具与配置路径（如 `/usr`、`/bin`、`/etc` 等）
* • **DENY**：`~/.ssh`、`~/.gnupg`、OpenClaw `.env`、OAuth token 目录、其他 dotfiles、其他用户 HOME 等

实现中应确保：工具层（`read/write/edit`）也会对敏感路径进行拒绝检查，用于降低密钥与凭证误读/外泄风险。

为便于评审，该段建议补充两条可验证的约束声明：

* • **沙盒内进程不可见 `.env`**：敏感配置由 Gateway 进程读取并经内存传递，不注入子进程环境，降低 `/proc/pid/environ` 读取面。
* • **敏感路径 deny 为硬约束**：即使沙盒关闭，工具层路径检查仍应拒绝读取 `~/.ssh`/`~/.gnupg` 等敏感目录（用于降低 T2/T4 的外泄面）。

## 8. 网络安全（Gateway 端点与 Agent 出站）

* • **Gateway 端点**：默认仅绑定 `127.0.0.1:18789`；WebSocket/HTTP 使用 Bearer Token；CORS 限制；连接数与 rate limit 限制，用于降低 T3 风险。
* • **Agent 出站（沙盒 ON）**：白名单模式，仅允许必要域名；显式拒绝云元数据地址（如 `169.254.169.254`）、内网网段与回连 Gateway；并对 DNS rebinding 进行解析后 IP 校验。

建议保留一段最小网络策略样例（便于在评审中核对 allow/deny 是否覆盖关键风险面）：

```
{  
  "sandbox": {  
    "network": {  
      "allowOutbound": ["api.anthropic.com:443", "api.openai.com:443", "*.googleapis.com:443", "github.com:443", "registry.npmjs.org:443"],  
      "denyOutbound": ["169.254.169.254", "10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16", "127.0.0.1:18789"],  
      "allowInbound": false  
    }  
  }  
}
```

其中 `denyOutbound` 的安全动机应被显式说明（避免被误认为“随意收紧”）：

* • `169.254.169.254`：云元数据服务，常见凭证泄露入口。
* • 内网网段：避免 Agent 扫描/访问内部服务。
* • 回连 Gateway：避免自引用导致的策略绕过或循环调用。
* • DNS rebinding：解析后 IP 落入 deny 范围则拒绝连接。

## 9. Prompt Injection 纵深防御

纵深防御链路如下：

1. 1. **System Prompt 安全边界声明**
2. 2. **来源标记（Source Tagging）**：区分 `user_direct` 与 `web/file/mcp/skill` 等来源并降低其可信度
3. 3. **结构化包裹（XML）**：将工具返回以结构化标签包裹以区分“数据/指令”
4. 4. **exec-approvals 兜底**：对高危命令实施人工审批/拒绝

该链路的关键评审点是：系统是否将“数据/指令”在工程层面强制分离，而不是依赖模型自觉。来源标记与结构化包裹用于降低 T2 的有效性；`exec-approvals` 用于在工具层提供最后阻断点。

## 10. 凭证安全与审计

* • **凭证存储**：`.env` 与凭证目录权限控制（0600）；OAuth token 使用 AES-256-GCM；并强调避免将敏感信息注入子进程环境。
* • **日志脱敏**：对 `sk-*`、`Bearer` 等敏感串进行正则脱敏。
* • **安全审计**：以 JSON Lines 记录工具执行、审批、认证失败、配对事件与策略触发等，便于追溯与治理。

建议在发布级材料中保留一条最小审计事件样例，确保“可追溯”不仅停留在口头描述：

```
{  
  "event": "tool_execution",  
  "tool": "exec",  
  "channel": "telegram",  
  "sandbox": true,  
  "approval": { "required": true, "result": "approved", "latencyMs": 12000 },  
  "result": { "exitCode": 0, "success": true, "executionTimeMs": 3400 }  
}
```

## 11. 默认推荐配置（建议起点）

在个人自托管且启用外部 Channel 的场景，推荐以以下组合为起点：

* • **sandbox.mode**：`non-main`
* • **Agent 职责拆分**：将“研究/审阅”与“执行”角色拆分，并对非执行角色采用严格 Tool Policy
* • **exec-approvals**：将具备破坏性、外部访问或权限提升属性的命令归入 `requireApproval`；保留毁灭性命令 `deny`
* • **network allow/deny**：仅放行确有必要的外部域名；显式拒绝云元数据与内网段；禁止回连 Gateway

为降低“片段散落”的复制成本，可将上述建议收敛为一个最小可用的配置汇总（示例字段与本文一致，部署时按实际产品配置文件结构落位）：

```
{  
  "security": {  
    "sandbox": {  
      "mode": "non-main",  
      "implementation": "auto",  
      "network": {  
        "allowOutbound": ["api.anthropic.com:443", "api.openai.com:443", "*.googleapis.com:443", "github.com:443", "registry.npmjs.org:443"],  
        "denyOutbound": ["169.254.169.254", "10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16", "127.0.0.1:18789"],  
        "allowInbound": false  
      }  
    },  
    "exec-approvals": {  
      "autoApprove": ["ls *", "pwd", "whoami", "date", "git status", "git log *", "git diff *"],  
      "requireApproval": ["rm *", "mv *", "cp *", "git push *", "git commit *", "sudo *", "curl *", "wget *", "docker *"],  
      "deny": ["rm -rf /", "rm -rf /*", "rm -rf ~", "mkfs *", "dd if=*", "shutdown *", "reboot *"],  
      "approvalTimeout": 300000  
    }  
  }  
}
```

## 12. 风险与限制

* • 本地攻击者（T5）不在防御范围。
* • 沙盒可用性依赖平台能力；当发生降级时，风险状态上升且需运维侧显式确认。
* • 凭证保护策略在“可用性”与“强隔离”之间存在取舍，部署时需结合组织安全基线进行二次加固。

## 13. 关键默认值与位置（便于评审复核）

* • **Gateway 绑定**：`127.0.0.1:18789`
* • **审计日志路径**：`~/.openclaw/logs/audit.jsonl`
* • **审批超时**：`approvalTimeout = 300000ms`（示例为 5 分钟）
* • **出站关键 deny**：云元数据 `169.254.169.254`、内网网段 `10/8`、`172.16/12`、`192.168/16`、以及回连 `127.0.0.1:18789`
* • **敏感路径 deny（代表性）**：`~/.ssh`、`~/.gnupg`、OpenClaw `.env`、OAuth token 目录

## 参考文档

OpenClaw Gateway Security: `https://docs.openclaw.ai/gateway/security`  
OpenClaw Gateway Sandboxing: `https://docs.openclaw.ai/gateway/sandboxing`