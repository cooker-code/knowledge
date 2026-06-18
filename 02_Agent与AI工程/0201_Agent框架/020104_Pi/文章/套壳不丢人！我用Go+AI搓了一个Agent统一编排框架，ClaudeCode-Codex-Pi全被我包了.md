---
title: 套壳不丢人！我用Go+AI搓了一个Agent统一编排框架，ClaudeCode-Codex-Pi全被我包了
author: 鸟窝聊技术
date: 鸟窝鸟窝
url: https://mp.weixin.qq.com/s?__biz=MzU2ODc4NzUxMg==&mid=2247490506&idx=1&sn=a53f94baf885045bbb679199a01cd853&chksm=fd39dee252db16167a4cc4d6bfd60cb7f45f4865a1f2fc43fdba1e5448bab6c07f5e29a93997&mpshare=1&scene=24&srcid=0605dAN1ENbbkAbKDirGITV1&sharer_shareinfo=64199303d0698a75b6cbc65a060352d6&sharer_shareinfo_first=64199303d0698a75b6cbc65a060352d6#rd
---

去年我还在折腾 langchain/langgraph 开发智能体，弄了个 langgraphgo 项目，把 langgraph 往 Go 生态圈里搬。那会儿网上做智能体的，十个有八个用 langchain/Crew AI。

一个阶段有一个阶段的玩法。

现在我看到了另一种路子：大家直接用 Claude Code、Codex、OpenCode、Pi 这些 coding agent "套壳"来实现智能体。

先说两个很多人搞混的点。

别觉得这些工具只能写代码。Claude Code、Codex 的架构走的是通用智能体模式，早就不止 coding 了。

也别把"套壳"当贬义词。Manus 刚火那阵，就有同事撇嘴说"这不就是 Claude 的套壳"。但你看，Claude、Codex、Antigravity 一个个都在推 SDK，巴不得你基于它们二次开发。牛顿怎么说的，站在巨人肩膀上不丢人。

百度厂内突然蹿红了一个叫 dodo 的应用比龙虾都火。就是靠"套壳"快速出产品原型，再慢慢长成大家离不开的工具。百度公众号和前几天的百度大会上也推了。

OpenClaw 是在 Pi coding agent 上搭起来的智能体产品，上半年火得不成样子。

这些东西免费（CC），有的还开源（Codex CLI、OpenCode、Pi），甚至设计了 harness engineering 这套东西。等于送你一个结实的产品基座，剩下的精力全押在产品和创意上。

去年我还在从零手搓智能体。今年，我负责的 LLM 训推故障分析产品已经改成跑在其中一个智能体上了。我只需要管诊断逻辑、知识库、对外 API——那些真正跟业务相关的事。

但问题来了：这么多智能体，到底用哪个？

小孩子才做选择题，成年人我都要。

那就自己动手。套一个壳。向上给统一调用层，向下接各种智能体。

于是 agent-wrapper 出来了。你可以拿它的命令行测试，也可以直接嵌到 Go 项目里，把精力放在你该忙的地方：扩大用户和赚钱。

> 有人会说 acp 不就能干这个？我实践下来 acp 不好用，从零搓了一个。具体分析见项目 README。

## 你到底要套什么

市面上的 Coding Agent CLI，单个拎出来都很能打——Claude Code 审代码，Codex 写脚本，Pi 做任务编排，OpenCode 跑自动化。但它们各自为政。五个操不同方言的装修师傅挤在一个工地，你每次想换人干活都得重新翻译需求。

胶水代码糊了一堆：启动子进程、解析各自的输出协议（NDJSON、SSE、JSONL、JSON-RPC……）、管生命周期、处理超时和错误、做重试和上下文压缩。全跟业务没关系，但不写系统就不稳。

agent-wrapper 干的就是这一层。一行代码注册 provider，一行代码切 agent：

```
registry := agentwrapper.NewRegistry()  
claude.RegisterIn(registry)  
codex.RegisterIn(registry)  
pi.RegisterIn(registry)  
  
agent, _ := registry.Get("codex", nil) // 切 Claude? 改成 "claude-code"  
orch := agentwrapper.NewOrchestrator(agent)  
result, _ := orch.RunSync(context.Background(), types.RunInput{  
    Prompt: "帮我重构这段代码",  
})  
fmt.Println(result.Text)
```

换 agent = 换一个字符串。胶水代码归零。

## 不只是包一层，是给 Agent 装上刹车

套壳没技术含量？我不认。

调一次 Agent 谁都会。管住它难得多。一个 AI Agent 在你的代码库里自由调用工具——写文件、删代码、执行命令——你敢不加约束？

agent-wrapper 的 Orchestrator 不是简单的 for 循环。内置了三道闸门（harness engineering 的标准做法）：

**审批拦截。** 每次 tool\_call 先过你的 ApprovalHandler。读文件放行，写文件拦住。按工具名、参数、上下文做决策，拒绝时注入合成 ToolResult，Agent 收到 "DENIED" 继续对话，不会崩：

```
orch := agentwrapper.NewOrchestrator(agent,  
    agentwrapper.WithApprovalHandler(func(ctx context.Context, call agentwrapper.ToolCall) (*agentwrapper.Decision, error) {  
switch call.Name {  
case"read", "ls", "grep":  
return &agentwrapper.Decision{Action: agentwrapper.ActionAllow}, nil  
default:  
return &agentwrapper.Decision{Action: agentwrapper.ActionDeny, Reason: "只读模式"}, nil  
        }  
    }),  
)
```

**预算控制。** Agent 烧 token 跟烧钱似的,听说网上某个团队一晚上干掉百万人民币的token。每个 turn 结束回调 BudgetHandler，超阈值直接掐断：

```
agentwrapper.WithBudgetHandler(func(ctx context.Context, usage types.TokenUsage)error {  
if usage.TotalTokens > 50000 {  
return fmt.Errorf("预算超支: %d tokens", usage.TotalTokens)  
    }  
returnnil  
}),
```

**上下文压缩 + 自动重试。** LLM 报 "context length exceeded"，Orchestrator 自动压消息历史（滑动窗口→摘要→链式策略，最多重试三次），调用方完全无感。

以上这三样，哪个 Agent CLI 原生都不给你。agent-wrapper 给了。

## 会话恢复：让 Agent 记住你的项目

Coding Agent 最烦的就是每次开新聊天都失忆。

agent-wrapper 支持 session resume。第一次 RunSync 拿回一个 SessionID，下次传进去，Agent 就知道你上次改了哪些文件、聊了什么架构、卡在哪个 bug 上：

```
// 第一轮  
r1, _ := orch.RunSync(ctx, types.RunInput{Prompt: "这个项目的目录结构是什么？"})  
fmt.Println(r1.SessionID) // ← 存起来  
  
// 下一轮，上下文全在  
r2, _ := orch.RunSync(ctx, types.RunInput{  
    Prompt:    "刚才你提到有一个潜在的性能问题，展开说说",  
    SessionID: r1.SessionID,  
})
```

CI 里跑重构，下班跑了一半，明天把 SessionID 传进去接着搞。跨天、跨进程、跨机器。

## 效果展示

一行命令就能跑：

```
# 流式输出（文本→stdout，元数据→stderr）  
agent-wrapper run --provider claude-code "解释这段代码"  
  
# JSON 聚合输出（CI/脚本友好）  
agent-wrapper run --provider codex "fix the bug" --json  
  
# 带审批+预算  
agent-wrapper run --provider claude-code "重构本项目" --approve-all --budget-tokens 50000  
  
# NDJSON 管道输出  
agent-wrapper run --provider claude-code "hello" --json --stream | jq .  
  
# 恢复会话  
agent-wrapper run --provider claude-code --session-id abc123 "继续"
```

但更核心的用法是当 Go 库用。go get 一下，import 进去，调 orch.Run。零额外进程，零协议开销。Claude Code、Codex、Pi、OpenCode，五个 provider，一套接口。

## 套壳的真正价值

套壳不丢人。

丢人的是换一次工具重写一遍胶水代码。丢人的是用着 AI Agent 还得手工管子进程生命周期。丢人的是一晚上 API 烧了几十美刀第二天看账单才后悔。

agent-wrapper 就是个壳。但解决的不是"怎么调一次 Agent"，是"怎么让 Agent 安全地、长期地、按预算地、跨 provider 为你工作"。把"玩一下"变成"能上线"。

项目地址：https://github.com/smallnest/agent-wrapper https://github.com/smallnest/agent-wrapper

## 最后说几句

大多数人用 Agent 还停留在"命令行输一个 prompt"。这没什么不对，但不够。

十年前我们从手写 SQL 走到满世界 ORM，从手动部署走到 k8s 编排。每一轮变化里，有人站在第一层骂套壳，有人站在第二层埋头套壳，还有人站在第三层——让所有想套壳的人套得更快。

Agent 时代，把壳套好，就是最好的手艺。