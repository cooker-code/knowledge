> 已吸收至：[[03_数据工程与数仓/0308_元数据血缘与治理/030803_数据血缘/030803_核心知识点/血缘图谱与AI助手应用边界|血缘图谱与AI助手应用边界]]
---
title: 《从零实现 Agent 系统》连载 12｜治理平面：策略引擎、审批与审计血缘
author: IchbinDerek
date: 贵慜贵慜
url: https://mp.weixin.qq.com/s?__biz=MzU3NjgxMzE4NQ==&mid=2247487260&idx=1&sn=8ab417799a46c7182356569b711d39ce&chksm=fc1b6d44211986fe0e7396f5a3fe7eb6e3a5b1cbb4398a014a2858cd2f51d75b12c7daba1aa0&mpshare=1&scene=24&srcid=060295sBCM99g0u5jofFgvih&sharer_shareinfo=2509b8aae7d258edb81a9b157423d2fb&sharer_shareinfo_first=2509b8aae7d258edb81a9b157423d2fb#rd
---

*对照项目 **Agentium** 的背景：[Agentium 论文与开源项目介绍](https://mp.weixin.qq.com/s?__biz=MzU3NjgxMzE4NQ==&mid=2247487181&idx=1&sn=00d771beb9f049f97bdc73a16f66f352&scene=21#wechat_redirect)。本文图表及核心设计均来自开源项目Agentium，源码详见GitHub。*

[连载 05 《工具系统：契约、注册与安全默认》](https://mp.weixin.qq.com/s?__biz=MzU3NjgxMzE4NQ==&mid=2247487212&idx=1&sn=4ec582595ea4785b5d594779f81f6821&scene=21#wechat_redirect)的工具 execute 里提过「策略门禁」；[连载 08 《编排与工作流：从 Chat 到任务图》](https://mp.weixin.qq.com/s?__biz=MzU3NjgxMzE4NQ==&mid=2247487230&idx=1&sn=357588854bc418b543c4abed645fbfd1&scene=21#wechat_redirect)的 workflow 节点会 **AWAITING\_APPROVAL**。**本篇把治理平面摊开**：策略不是 if-else 散落在 handler 里，而是 **Policy as data**；人要插嘴走 **ApprovalGate**；每一次 allow/deny/审批/发布都要进 **AuditSink**，能按 `run_id` 串成血缘。高敏操作 **必须经过 gate**——不是模型「更谨慎」就够了。

---

## 治理平面管什么

与 **控制循环**（怎么转）、**协调层**（怎么排步骤）并列，**governance** 管三件事：

1. **策略（Policy）**：这次 tool / skill / 脚本路径，ALLOW、DENY 还是 REQUIRE\_APPROVAL？
2. **审批（Approval）**：REQUIRE\_APPROVAL 时谁批、批了怎么 resume、TTL 过了怎么办？
3. **审计（Audit / Lineage）**：不可变事件流，绑 `tenant_id`、`run_id`、`policy_version`、`rule_id`。

工具、通道出站、沙箱 deny、MCP  unsigned——凡影响安全与合规的决策，宜 **同一套 AuditRecord 形状** 追加，而不是各模块自定义日志格式。

---

## PolicyEngine：默认拒绝，规则可解释

策略文档（YAML/JSON）收成 **PolicyDocument**：

* **version**：活跃策略版本，写进审计；
* **default\_decision**：未命中规则时怎么办——企业场景常用 **DENY**；
* **rules**：有序列表，每条有 **id**、**decision**、**reason**，并按 **tools / roles / tenants / skills** 等维度匹配。

`decide_tool_call(context, tool_name)` 扫描规则，首条命中即返回 **Decision**（含 `rule_id` 方便 explainability）；否则回落 **default\_decision**。
还有 **decide\_skill\_use**、**decide\_skill\_script**——Skill 加载与脚本执行是独立维度，别只用 tool 规则硬套。

**Policy as data** 的好处：改 YAML、走发布流程即可，不必 redeploy 整包业务代码；坏处是 **规则冲突与顺序** 要文档化——通常「更具体的规则放前面，default deny 兜底」。

---

## 审批闸口：从 REQUIRE\_APPROVAL 到 resume

当 Decision 为 **REQUIRE\_APPROVAL** 时，ToolRegistry **不得**直接跑 handler：

1. 计算 **args\_hash**（参数摘要，防批 A 执行 B）；
2. **ApprovalGate.request\_approval** 生成 `approval_id`，状态 **PENDING**，可选 TTL 与 **resume\_state\_json**；
3. 抛 **ApprovalRequiredError**（带 `approval_id`）→ tool loop 或 workflow **挂起**（连载 03、08）；
4. 运维 / 用户通过 HTTP 或工单 **approve / reject**；
5. 批准后走 **execute\_after\_approval**——校验 approval 仍 pending、未过期、args 一致，再进 handler。

ApprovalRequest 跟踪：`run_id`、`tenant_id`、`tool_name`、`requested_by`、`expires_at`。过期任务应 **expire\_pending** 扫掉，避免僵尸 pending 占着 resume 口。

Workflow 节点的 HITL 与工具层 **同构**：都是「挂起 → 外部决策 → 带 id 恢复」，只是 pending 粒度不同。

---

## 审计血缘：append-only，按 run 回放

**AuditSink** 协议：`append(record)` + 可选 `query(run_id, tenant_id)`。
**AuditRecord** 含 `event_type`、`tenant_id`、`run_id`、`policy_version`、`payload`——例如 `tool_invoked`、`policy_denied`、`node_awaiting_approval`、`channel_delivered`、`sandbox_denied`。

原则：

* **只追加、不覆盖**——纠错用 compensating event，不改历史行；
* 持久化可用 **JSONL** 文件或 DB，测试用 in-memory；
* 与 **trace\_id**（RequestContext）对齐，方便和 telemetry  join。

「血缘」在这里指：从 ingress → policy → tool → artifact → outbound，同一 **run\_id** 能拉一条事件链，回答「谁准的、哪版策略、批了没有」。

---

## 领域包与策略发布：可回滚的变更单元

单文件 policy 不够用时，用 **Domain Pack** 打包分发：

* manifest（id、version、signer、各文件 checksum）；
* **policy.yaml** 追加规则；
* **sandbox\_profiles.yaml** 合并 capability 白名单（连载 09）；
* **dlp\_rules.yaml** 等扩展。

Loader **校验 checksum 再 merge**——manifest 声明的 hash 与文件字节不符则 **拒绝加载**，避免中间人篡改半套策略。

更大变更走 **PolicyReleaseManager**：

* 提交 **PolicyBundle**（version + policy\_document + **HMAC 签名**）；
* 验签失败 → fail-closed；
* 审批通过后 **canary 激活**（可按 tenant 灰度）；
* 保留 **previous\_release**，支持 **rollback** 到上一活跃版本。

这与[连载 11](https://mp.weixin.qq.com/s?__biz=MzU3NjgxMzE4NQ==&mid=2247487255&idx=1&sn=adf7dd37e2486f2f1a1ceab13a6a2072&scene=21#wechat_redirect) 的插件 fingerprint 同族：**治理变更也是版本化 artifact**，不是 ssh 上去改一行 YAML。

---

## 端到端：高敏工具过 gate（序列）

---

## 伪代码：evaluate 与 wait

```
function ExecuteTool(ctx, tool_name, args):

    decision := PolicyEngine.DecideToolCall(ctx, tool_name, args)
    Audit.Append("policy_evaluated", ctx, decision)

    if decision.type == DENY:
        raise PolicyDenied(decision.reason, decision.rule_id)

    if decision.type == REQUIRE_APPROVAL:
        hash := HashArgs(args)
        req := ApprovalGate.Request(ctx, tool_name, decision.reason, hash)
        Audit.Append("approval_requested", ctx, req.approval_id)
        raise ApprovalRequired(req.approval_id)

    return RunHandlerWithSandboxAndBudget(ctx, tool_name, args)


function ExecuteAfterApproval(ctx, tool_name, args, approval_id):

    req := ApprovalGate.Get(approval_id)
    if req.status != APPROVED or req.args_hash != HashArgs(args):
        raise PolicyDenied("approval mismatch or expired")
    Audit.Append("approval_consumed", ctx, approval_id)
    return RunHandlerWithSandboxAndBudget(ctx, tool_name, args)
```

---

## 与相邻连载的接缝

* **[连载 05](https://mp.weixin.qq.com/s?__biz=MzU3NjgxMzE4NQ==&mid=2247487212&idx=1&sn=4ec582595ea4785b5d594779f81f6821&scene=21#wechat_redirect)**：execute 管线第 2 步就是 policy；审批是 execute 的 **分支出口**。
* **[连载 08](https://mp.weixin.qq.com/s?__biz=MzU3NjgxMzE4NQ==&mid=2247487230&idx=1&sn=357588854bc418b543c4abed645fbfd1&scene=21#wechat_redirect)**：workflow `ApprovalRequiredError` → `pending_node` + resume。
* **[连载 09](https://mp.weixin.qq.com/s?__biz=MzU3NjgxMzE4NQ==&mid=2247487243&idx=1&sn=9b98ac21003c9e88fea16d02176ad71f&scene=21#wechat_redirect)**：domain pack 同时 merge **sandbox\_profiles**。
* **[连载 10](https://mp.weixin.qq.com/s?__biz=MzU3NjgxMzE4NQ==&mid=2247487248&idx=1&sn=9dbeb92b9f2e391fa2c2a1f1962773e2&scene=21#wechat_redirect)**：OutboundOrchestrator 的 block 也写 **AuditSink**。
* **连载 13**（下一篇）：注入/DLP/宪法 guard 是 **策略之外的纵深防御**，与 PolicyEngine **并联**，不是替代。

---

## 几句容易踩坑的地方

**default\_decision=ALLOW**——新工具未写规则就全网可跑。**审批无 args\_hash**——批邮件发信却执行删库。**审计只打日志不进 Sink**——合规问「谁批准的」答不出。**改 policy 不 bump version**——同 run 前后策略不一致无法回放。**Domain pack 不验 checksum**——供应链篡改。**Approval 无 TTL**——pending 永久占着 resume。**Policy 规则写死在 Python**——无法灰度与回滚。**只有 tool 规则没有 skill\_script 规则**——Skill 脚本成后门。

---

## 收束一下，下一篇讲什么

本篇应能画出 **高敏 tool 从 policy → approval → audit → handler** 的序列，说清 **default deny**、**ApprovalGate resume**、**AuditSink 血缘** 与 **domain pack / policy release 回滚**。

下一篇进入 **安全防线**：提示注入、社工、DLP、密钥扫描、宪法式 guard——与策略并联的纵深防御。（连载 13）