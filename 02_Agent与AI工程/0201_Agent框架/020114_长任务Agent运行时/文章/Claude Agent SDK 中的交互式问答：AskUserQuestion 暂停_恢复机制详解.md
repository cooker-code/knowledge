---
title: Claude Agent SDK 中的交互式问答：AskUserQuestion 暂停/恢复机制详解
author: AI贺贺
date: 艾贺艾贺
url: https://mp.weixin.qq.com/s?__biz=MzAxNjIzOTkyOQ==&mid=2449867812&idx=1&sn=86ed6f99da6e66f564044e61ba5ecd44&chksm=8dda16b3ed86406dbe99d476eb61084d6af0a1d11580d8fe1868f7b03ca43c028b97d8092ed8&mpshare=1&scene=24&srcid=0603VDSDrjlWoiAORTg9ZIIa&sharer_shareinfo=89497ad84bc976321cd1463295b06fcb&sharer_shareinfo_first=89497ad84bc976321cd1463295b06fcb#rd
---

> 在非交互式 AI Agent 中实现"暂停等待用户回答"的完整方案

## 官方参考文档

* Agent SDK — Permissions — 权限评估顺序、模式说明
* Agent SDK — User Input — `canUseTool` 回调、AskUserQuestion 处理
* Hooks Guide — PreToolUse Hook、defer/allow/deny

## 问题

Claude Agent SDK 的 `AskUserQuestion` 工具用于在对话中向用户提问。但在 Web API 场景下（SSE 流式响应、Agent Run），没有交互式终端——Claude 发出问题后，谁来回答？

核心挑战：**如何让一个正在运行的 HTTP 流式请求"暂停"，等待用户通过另一个 HTTP 请求提交答案，然后继续执行？**

## 解决方案概览

我们实现了两种模式：

| 模式 | 原理 | SSE 流 | 适用场景 |
| --- | --- | --- | --- |
| **Wait 模式** | Promise 阻塞 | 保持打开 | Chat 对话（单次长连接） |
| **Resume 模式** | Hook defer + 新会话恢复 | 关闭后重建 | Agent Run（需要断点恢复） |

---

AskUserQuestion 架构横幅图

image-20260414103732594

## 核心概念

### 1. Claude Agent SDK 的工具拦截层

SDK 提供两个时机拦截工具调用：

```
Claude 决定调用 AskUserQuestion  
        │  
        ▼  
  ┌─────────────────┐  
  │  PreToolUse Hook │  ← 脚本钩子（第 1 层，最先执行）  
  │  (优先级最高)    │     deny 可覆盖 bypassPermissions  
  │                  │     可以 allow / deny / defer  
  └────────┬────────┘  
           │ Hook 未 deny  
           ▼  
  ┌─────────────────┐  
  │  Permission Mode │  ← 权限模式（第 3 层）  
  │  + Deny/Allow    │     bypassPermissions 自动放行  
  └────────┬────────┘  
           │  
           ▼  
  ┌─────────────────┐  
  │  canUseTool      │  ← 代码回调（第 5 层，最后执行）  
  │  (优先级兜底)    │     AskUserQuestion 必定走到这里  
  │                  │     可以阻塞等待，updatedInput 替换工具输入  
  └────────┬────────┘  
           │  
           ▼  
     工具实际执行
```

**关键点：**

* `PreToolUse` Hook 是第一层防线，在所有权限检查之前执行。官方文档明确指出：Hook 返回 `deny` 即使在 `bypassPermissions` 模式下也会阻止工具执行。
* `canUseTool` 是最后一层，只有在前面所有层都没有做出决定时才会被调用。但因为 `AskUserQuestion` 的 `requiresUserInteraction()=true`，它**绕过了权限模式的自动放行**，确保一定会到达 `canUseTool`。
* `canUseTool` 的 `updatedInput` 优先级高于 Hook 的 `updatedInput`。两者不要同时用于 AskUserQuestion，否则会互相覆盖。

### 1.1 canUseTool 的真正含义

SDK 类型定义：

```
type CanUseTool = (  
  toolName: string,                             // 工具名：Read, Write, Bash, AskUserQuestion ...  
  input: Record<string, unknown>,               // 工具的输入参数  
  options: {  
    signal: AbortSignal                          // 取消信号  
    suggestions?: PermissionUpdate[]             // 权限更新建议  
    blockedPath?: string                         // 触发权限的文件路径  
    decisionReason?: string                      // 为什么触发权限检查  
    toolUseID: string                            // 本次工具调用的唯一 ID  
    agentID?: string                             // 子 agent ID（如果有）  
  }  
) => Promise<PermissionResult>  
  
type PermissionResult =  
  | { behavior: 'allow', updatedInput?: Record<string, unknown> }  // 放行，可修改输入  
  | { behavior: 'deny', message: string }                          // 拒绝，附上原因
```

**本质：它是 SDK 的"工具执行前权限检查 + 输入篡改"回调。** 返回 `allow` 放行，返回 `deny` 拒绝。放行时可以通过 `updatedInput` 替换工具的原始输入参数。

### 1.2 工具权限的完整评估顺序（官方）

根据 Agent SDK 官方文档 — How permissions are evaluated，当 Claude 请求使用工具时，SDK 按以下**固定顺序**评估权限：

```
Claude 决定调用工具（Read / Write / Bash / Edit / AskUserQuestion / ...）  
        │  
        ▼  
  ┌─────────────────────────────────────────────┐  
  │  第 1 层：PreToolUse Hooks                   │  
  │  脚本钩子，优先级最高                         │  
  │  deny  → 直接拒绝，后续所有层都不执行         │  
  │  allow → 继续（可附带 updatedInput）          │  
  │  defer → 暂停（Agent Run Resume 模式）        │  
  └──────────┬──────────────────────────────────┘  
             │ Hook 未 deny  
             ▼  
  ┌─────────────────────────────────────────────┐  
  │  第 2 层：Deny Rules（拒绝规则）              │  
  │  用户配置的显式拒绝列表                       │  
  └──────────┬──────────────────────────────────┘  
             │ 不在 deny 列表中  
             ▼  
  ┌─────────────────────────────────────────────┐  
  │  第 3 层：Permission Mode（权限模式）         │  
  │                                             │  
  │  bypassPermissions → 全部自动放行 ✅         │  
  │  plan             → 只读放行 ✅ 写操作拒绝 ❌│  
  │  default          → 危险操作需确认           │  
  │  dontAsk          → 未批准就拒绝             │  
  └──────────┬──────────────────────────────────┘  
             │ 权限模式认为"允许"  
             ▼  
  ┌─────────────────────────────────────────────┐  
  │  第 4 层：Allow Rules（允许规则）             │  
  │  用户配置的显式允许列表                       │  
  └──────────┬──────────────────────────────────┘  
             │ 允许规则匹配 / 未匹配  
             ▼  
  ┌─────────────────────────────────────────────┐  
  │  第 5 层：canUseTool 回调                    │  
  │  你提供的代码回调，最后兜底                   │  
  │  返回 allow（可附带 updatedInput）或 deny     │  
  └──────────┬──────────────────────────────────┘  
             │  
             ▼  
       工具实际执行
```

**官方原文：**

> *When Claude requests a tool, the SDK checks permissions in a specific order. It first runs hooks, then checks deny rules, followed by the active permission mode. Allow rules are checked next, and finally, if not resolved, the `canUseTool` callback is invoked.*
>
> — Agent SDK Permissions

**关键要点：**

1. **Hooks 最先执行**：`PreToolUse` Hook 在权限模式检查之前运行。Hook 返回 `deny` 会**直接阻止**工具执行，即使 `permissionMode` 设为 `bypassPermissions`。
2. **canUseTool 最后执行**：只有前面所有层都没有做出决定时，才会调用 `canUseTool`。在 `bypassPermissions` 模式下，大多数工具在第 3 层就被自动放行了，不会走到 `canUseTool`。
3. **AskUserQuestion 是例外**：见下文 1.3 节。

| permissionMode | 行为 | canUseTool 是否被调用 |
| --- | --- | --- |
| `bypassPermissions` | 所有工具自动放行 | 一般**不调用**（AskUserQuestion 例外，见 1.3） |
| `default` | 危险操作需要确认 | **只对需要确认的工具** 调用 |
| `plan` | 只读放行，写操作拒绝 | **对写操作** 调用 |
| `dontAsk` | 不提示，未批准就拒绝 | **对未批准的工具** 调用 |

### 1.3 为什么 bypassPermissions 下 AskUserQuestion 仍会触发 canUseTool

这是一个关键设计细节，也是整个方案成立的前提。

根据 1.2 节的评估顺序，`bypassPermissions` 模式下，大多数工具在第 3 层就被自动放行了，不会走到第 5 层的 `canUseTool`。

**但 `AskUserQuestion` 例外。** 官方文档明确说明：

> *Claude requests user input in two main scenarios: when it requires permission to use a tool that is not auto-approved, or when it needs to ask clarifying questions using the `AskUserQuestion` tool. **Both of these situations trigger your `canUseTool` callback**, which pauses the execution until a response is provided.*
>
> — Agent SDK User Input

也就是说，`AskUserQuestion` 工具本身有一个特殊标记——SDK 内部的 `requiresUserInteraction()` 方法：

| 工具 | `requiresUserInteraction()` | 含义 |
| --- | --- | --- |
| `AskUserQuestion` | **`true`** | 必须有用户交互才能执行 |
| `Read` | `false` | 不需要用户交互 |
| `Write` | `false` | 不需要用户交互 |
| `Bash` | `false` | 不需要用户交互 |
| `Edit` | `false` | 不需要用户交互 |
| 其他所有工具 | `false` | 不需要用户交互 |

SDK 的决策逻辑（从 `cli.js` 反编译简化）：

```
// SDK 内部对每个工具调用的决策流程  
if (permissionResult.behavior === 'allow'  
    && !tool.requiresUserInteraction?.()  
    && !queryOptions.requireCanUseTool) {  
// 普通工具 + bypassPermissions → 直接执行，跳过 canUseTool  
  executeTool(tool, input)  
} elseif (permissionResult.behavior === 'allow'  
           && (tool.requiresUserInteraction?.() || queryOptions.requireCanUseTool)) {  
// 特殊工具（如 AskUserQuestion）或 requireCanUseTool=true → 必须调用 canUseTool  
const canUseToolResult = await canUseToolCallback(toolName, input, options)  
if (canUseToolResult.behavior === 'allow') {  
    executeTool(tool, canUseToolResult.updatedInput ?? input)  
  } else {  
    // deny → 工具不执行，Claude 收到拒绝消息  
  }  
}
```

**这解释了三件事：**

1. **为什么 AskUserQuestion 能被拦截**：它的 `requiresUserInteraction()` 返回 `true`，无论 `permissionMode` 是什么，SDK 都会调用 `canUseTool` 回调。
2. **为什么其他工具不受影响**：`Read`/`Write`/`Bash` 等工具的 `requiresUserInteraction()` 返回 `false`，在 `bypassPermissions` 模式下直接执行，`canUseTool` 根本不会被调用。
3. **为什么我们的 canUseTool 对其他工具返回 deny 也不会出问题**：那段代码是防御性代码——在 `bypassPermissions` 模式下永远不会被执行到。

> **注意：** 还有一个 `requireCanUseTool` 查询级选项。如果设置为 `true`，所有工具都会走 `canUseTool` 回调，无论 `requiresUserInteraction()` 返回什么。我们**没有**使用这个选项。

> **官方参考：**Agent SDK Permissions | Agent SDK User Input | Hooks Guide

### 1.4 updatedInput 的作用

`canUseTool` 返回 `allow` 时可以通过 `updatedInput`**替换工具的原始输入**。这是 Wait 模式的核心：

```
Claude 发出:    { questions: [...], answers: {} }              ← 没有答案  
                                    │  
                              canUseTool 拦截  
                                    │  
                              await 等待用户回答  
                                    │  
                              updatedInput 替换  
                                    │  
Claude 收到:    { questions: [...], answers: { "问题": "用户答案" } }  ← 带答案
```

`updatedInput` 的优先级**高于** PreToolUse Hook 的 `updatedInput`，所以两者不要同时用于同一个工具。

### 2. async/await 暂停 ≠ 进程阻塞

Wait 模式的核心是一个 **Pending Promise**：

```
// canUseTool 回调中  
const answers = await new Promise((resolve) => {  
  // 把 resolve 存到 Map 里，等另一个 HTTP 请求来调用  
  pendingQuestions.set(questionId, { resolve })  
})  
  
// 返回答案给 SDK  
return { behavior: 'allow', updatedInput: { ...input, answers } }
```

当 Promise pending 时：

* 这个 async 函数**挂起**在 `await` 处
* Node.js **事件循环继续运行**，正常处理其他 HTTP 请求
* SSE 流的 `response.write()` 仍然有效
* 当另一个请求调用 `resolve(answers)` 时，async 函数恢复执行

```
事件循环视角：  
  
  ┌─ 处理 /api/chat/send 请求  
  │    ├─ SDK 消息循环（for await ...）  
  │    ├─ AskUserQuestion 触发 canUseTool  
  │    ├─ await new Promise(...) ← 挂起在这里  
  │    │  
  │    │  ┌─ 处理 /api/chat/questions/:id/answer 请求  
  │    │  │    └─ resolve(answers) ← 恢复上面的 Promise  
  │    │  │  
  │    │  └─ 继续其他 HTTP 请求...  
  │    │  
  │    ├─ Promise resolved，拿到 answers  
  │    ├─ 返回 { behavior: 'allow', updatedInput: ... }  
  │    └─ SDK 消息循环继续  
  └─ SSE 流正常结束
```

---

## Wait 模式实现（推荐）

### 后端架构

```
POST /api/chat/send (SSE 流)  
    │  
    ├─ query() 启动 Claude SDK  
    │   │  
    │   ├─ SDK 循环产出消息 → SSE 写入 response  
    │   │  
    │   ├─ Claude 调用 AskUserQuestion  
    │   │   └─ canUseTool 触发  
    │   │       ├─ 生成 questionId  
    │   │       ├─ SSE 写入 ask_user_question 事件  
    │   │       └─ 创建 Promise，await 等待  ← 这里暂停  
    │   │  
    │   │        ... 用户思考中，SSE 流保持打开 ...  
    │   │  
    │   │   POST /api/chat/questions/:id/answer  
    │   │       └─ pending.resolve(answers)  ← 这里恢复  
    │   │  
    │   ├─ 拿到 answers → 返回 allow + updatedInput  
    │   └─ SDK 继续 → SSE 写入后续消息  
    │  
    └─ SSE 流结束 [DONE]
```

### 关键代码

#### 1. canUseTool 工厂函数

```
// src/utils/canUseTool.ts  
  
function createInteractiveAskUserQuestionCanUseTool(  
  contextLabel: string,  
  handlers: {  
    onQuestion: (request: {  
      toolUseId?: string  
      input: AskUserQuestionInput  
      signal?: AbortSignal  
    }) => Promise<Record<string, string | string[]>>  
  }  
): CanUseTool {  
returnasync (toolName, input, options) => {  
    if (toolName !== 'AskUserQuestion') {  
      // ⚠️ 看似会拦截所有其他工具，实际上是死代码：  
      //  
      // 回顾 1.2 节的评估顺序：  
      //   Hooks → Deny Rules → Permission Mode → Allow Rules → canUseTool  
      //  
      // 在 bypassPermissions 模式下，Read/Write/Bash 等普通工具  
      // 在第 3 层 Permission Mode 就被自动放行了，根本不会走到第 5 层的 canUseTool。  
      //  
      // 只有 AskUserQuestion 因为 requiresUserInteraction()=true，  
      // 才会绕过第 3 层的自动放行，强制到达 canUseTool。  
      //  
      // 所以这段 deny 分支在 bypassPermissions 模式下永远不会被执行。  
      // 它只是防御性代码，防止未来 SDK 行为变化或 permissionMode 切换时出问题。  
      return { behavior: 'deny', message: '...' }  
    }  
  
    // 只有 AskUserQuestion 会走到这里  
    const normalizedInput = normalizeAskUserQuestionInput(input)  
    const answers = await handlers.onQuestion({  
      toolUseId: options.toolUseID,  
      input: normalizedInput,  
      signal: options.signal  
    })  
  
    return {  
      behavior: 'allow',  
      updatedInput: {  
        ...input,  
        questions: normalizedInput.questions,  
        answers: { ...normalizedInput.answers, ...answers }  
      }  
    }  
  }  
}
```

> **Q: 如果其他工具真的走到 canUseTool 会被拦截吗？**
>
> 会。但这只在以下条件**同时满足**时才可能发生：
>
> 1. `permissionMode` 不是 `bypassPermissions`（如 `default` 或 `dontAsk`）
> 2. 该工具没有被 Deny Rules 拒绝，也没有被 Allow Rules 放行
> 3. 该工具没有被 PreToolUse Hook 拦截
>
> 在我们的 Chat 端点中，使用的是 `bypassPermissions`，所以条件 1 就不满足——其他工具永远不会到达 canUseTool。
>
> 如果你的场景使用 `default` 模式，并且也配了这个 canUseTool，那其他工具确实会被 deny。这时应该改为：非 AskUserQuestion 工具返回 `{ behavior: 'allow' }` 放行，或者只对 AskUserQuestion 使用 canUseTool，其他工具交给 Hook 处理。

#### 2. Chat 端点的 handler

```
// src/server/createServer.ts — POST /api/chat/send  
  
const activeQuestionIds = new Set<string>()  
const abortController = new AbortController()  
  
const sdkQuery = query({  
  prompt,  
  options: {  
    // ... 其他配置  
    canUseTool: createInteractiveAskUserQuestionCanUseTool('server.chat', {  
      onQuestion: async ({ toolUseId, input, signal }) => {  
        const questionId = randomUUID()  
        activeQuestionIds.add(questionId)  
  
        // 1. 创建 pending 记录（resolve/reject 稍后填充）  
        pendingChatQuestions.set(questionId, {  
          id: questionId,  
          sessionId: currentSessionId,  
          questions: input.questions,  
          existingAnswers: input.answers,  
          resolve: () => {},  // 占位，下面立即覆盖  
          reject: () => {}  
        })  
  
        // 2. 通过 SSE 通知前端展示问题  
        writeSseEvent(response, {  
          type: 'ask_user_question',  
          questionId,  
          questions: input.questions,  
          // ...  
        })  
  
        // 3. 阻塞等待答案  
        returnnewPromise((resolve, reject) => {  
          const handleAbort = () => {  
            pendingChatQuestions.delete(questionId)  
            reject(newError('Aborted'))  
          }  
          signal?.addEventListener('abort', handleAbort, { once: true })  
  
          // 覆盖 pending 记录的 resolve/reject  
          const pending = pendingChatQuestions.get(questionId)!  
          pending.resolve = (answers) => {  
            signal?.removeEventListener('abort', handleAbort)  
            activeQuestionIds.delete(questionId)  
            resolve(answers)  
          }  
          pending.reject = (error) => {  
            signal?.removeEventListener('abort', handleAbort)  
            activeQuestionIds.delete(questionId)  
            reject(error)  
          }  
        })  
      }  
    }),  
    abortController  
  }  
})  
  
// SSE 流结束后清理所有未回答的问题  
request.on('close', () => {  
  abortController.abort()  
for (const id of activeQuestionIds) {  
    pendingChatQuestions.get(id)?.reject(newError('Stream closed'))  
    pendingChatQuestions.delete(id)  
  }  
})
```

#### 3. Answer 端点

```
// POST /api/chat/questions/:questionId/answer  
  
router.post('/api/chat/questions/:questionId/answer', async (request, response) => {  
const pending = pendingChatQuestions.get(request.params.questionId)  
if (!pending) {  
    return response.status(404).json({ error: 'Not found' })  
  }  
  
// 验证：每个问题都必须有答案  
const providedAnswers = collectProvidedAskUserQuestionAnswers(request.body.answers)  
const merged = { ...pending.existingAnswers, ...providedAnswers }  
const missing = pending.questions  
    .map(q => q.question)  
    .filter(q => !(q in merged))  
  
if (missing.length > 0) {  
    return response.status(400).json({ error: 'Missing answers', missingQuestions: missing })  
  }  
  
// 解除 Promise 阻塞  
  pendingChatQuestions.delete(request.params.questionId)  
  pending.resolve(merged)  
  
  response.status(202).json({ accepted: true, questionId: request.params.questionId })  
})
```

### 前端实现

```
// public/pages/chat.js — 简化版  
  
// 1. 发送消息，监听 SSE 流  
asyncfunction sendMessage(msg) {  
  chatState.streaming = true  
const response = await fetch('/api/chat/send', {  
    method: 'POST',  
    headers: { 'content-type': 'application/json' },  
    body: JSON.stringify({ cwd, message: msg, sessionId })  
  })  
  
const reader = response.body.getReader()  
const decoder = new TextDecoder()  
  
while (true) {  
    const { value, done } = await reader.read()  
    if (done) break  
  
    const lines = decoder.decode(value, { stream: true }).split('\n')  
    for (const line of lines) {  
      if (!line.startsWith('data: ')) continue  
      const event = JSON.parse(line.slice(6))  
  
      // 2. 收到 ask_user_question 事件 → 展示问题表单  
      if (event.type === 'ask_user_question') {  
        chatState.pendingQuestion = event  
        renderQuestionForm(event)  // 渲染 radio/checkbox/freeform  
      }  
  
      // 正常消息继续显示  
      chatState.messages.push(event)  
    }  
  }  
}  
  
// 3. 用户填写答案后提交  
asyncfunction submitPendingQuestion(form) {  
const questionId = form.dataset.questionId  
const answers = {}  
  
// 从表单收集答案（radio → string, checkbox → string[], textarea → string）  
for (const block of form.querySelectorAll('.chat-question-block')) {  
    const question = block.dataset.questionText  
    if (block.dataset.multiSelect === 'true') {  
      answers[question] = [...block.querySelectorAll('input[type="checkbox"]:checked')]  
        .map(input => input.value)  
    } else {  
      const radio = block.querySelector('input[type="radio"]:checked')  
      const text = block.querySelector('textarea, input[type="text"]')  
      answers[question] = radio?.value ?? text?.value ?? ''  
    }  
  }  
  
// 提交答案 → 后端 resolve Promise → SSE 流自动恢复  
await fetch(`/api/chat/questions/${questionId}/answer`, {  
    method: 'POST',  
    headers: { 'content-type': 'application/json' },  
    body: JSON.stringify({ answers })  
  })  
  
  chatState.pendingQuestion = null  
// SSE 流还在，Claude 会继续输出后续消息  
}
```

### 时序图

```
Frontend                    Backend                     Claude SDK  
   │                          │                            │  
   │── POST /chat/send ──────>│── query() ───────────────>│  
   │<── SSE: assistant ───────│<── stream events ─────────│  
   │<── SSE: assistant ───────│                            │  
   │                          │<── AskUserQuestion ────────│  
   │                          │   canUseTool fires          │  
   │                          │   Promise pending ⏳        │  
   │<── SSE: ask_user_question ─│                          │  
   │                          │                            │  
   │  [用户看到问题，填写答案]    │                            │  
   │                          │                            │  
   │── POST /questions/:id/answer ─>│ resolve(answers)     │  
   │<── 202 Accepted ─────────│   Promise resolved ✅      │  
   │                          │── allow + answers ────────>│  
   │<── SSE: assistant ───────│<── Claude continues ───────│  
   │<── SSE: result ──────────│<── done ───────────────────│  
   │<── [DONE] ───────────────│                            │
```

---

## Resume 模式实现（备选）

适用于 SSE 流不能长时间保持的场景。

### 原理

使用 `PreToolUse` Hook 返回 `defer`，SDK 产生 `stop_reason: 'tool_deferred'` 结果并停止。服务端关闭 SSE 流，保存问题。用户提交答案后，创建新的 SDK 会话恢复。

### 数据流

```
POST /api/agent/runs (SSE)  
    │  
    ├─ Claude 调用 AskUserQuestion  
    │   └─ PreToolUse Hook → { permissionDecision: 'defer' }  
    │       └─ SDK 返回 { stop_reason: 'tool_deferred' }  
    │           └─ SSE 发出 ask_user_question + done(awaiting_input)  
    │               └─ 连接关闭  
    │  
    ├─ POST /agent/runs/:runId/questions/:questionId/answer  
    │   └─ 存储答案，创建新的 run  
    │  
    └─ GET /agent/runs/:newRunId/stream (新 SSE)  
        └─ 新 run 恢复会话  
            └─ Hook 的 getResumeAnswers() 返回存好的答案  
                └─ { permissionDecision: 'allow', updatedInput: { answers } }  
                    └─ Claude 继续对话
```

### 关键代码

```
// Hook 工厂函数  
function createAskUserQuestionHooks(contextLabel, handlers) {  
return {  
    PreToolUse: [{  
      matcher: 'AskUserQuestion',  
      hooks: [async (hookInput) => {  
        const normalizedInput = normalizeAskUserQuestionInput(hookInput.tool_input)  
  
        // 检查是否有之前存好的答案（恢复场景）  
        const resumeAnswers = await handlers.getResumeAnswers?.({  
          toolUseId: hookInput.tool_use_id,  
          input: normalizedInput  
        })  
  
        if (resumeAnswers) {  
          // 有答案 → 允许执行  
          return {  
            continue: true,  
            hookSpecificOutput: {  
              permissionDecision: 'allow',  
              updatedInput: { ...normalizedInput, answers: resumeAnswers }  
            }  
          }  
        }  
  
        // 没有答案 → 暂停，通知前端  
        await handlers.onDefer({ toolUseId: hookInput.tool_use_id, input: normalizedInput })  
  
        return {  
          continue: true,  
          hookSpecificOutput: { permissionDecision: 'defer' }  
        }  
      }]  
    }]  
  }  
}
```

### 识别 deferred 结果

```
function isAskUserQuestionDeferredResult(message: any): boolean {  
  return message?.type === 'result'  
    && message?.stop_reason === 'tool_deferred'  
    && (!message?.deferred_tool_use?.name  
        || message.deferred_tool_use.name === 'AskUserQuestion')  
}
```

---

## 两种模式对比

| 维度 | Wait 模式 | Resume 模式 |
| --- | --- | --- |
| SSE 连接 | 保持打开，同一个流 | 关闭，需要新建 |
| SDK 会话 | 同一个 query() | 新建 query() 恢复 |
| 实现复杂度 | 低（Promise + resolve） | 高（defer + resume + 会话恢复） |
| 答案传递 | 直接 resolve Promise | 需要持久化答案，恢复时注入 |
| 适合场景 | 浏览器 Chat（长连接） | API Agent Run（可能断连） |
| 工具调用 | `canUseTool` | `PreToolUse` Hook |

---

## 实现清单

如果要在自己的项目中实现，按以下顺序：

### 后端

1. **定义 Pending Question 存储**

* `Map<string, { resolve, reject, questions, ... }>`
* 存在模块级变量或请求作用域中

2. **实现 canUseTool 工厂**

* 匹配 `AskUserQuestion` 工具名
* 创建 Promise，把 resolve/reject 存入 Map
* 通过 SSE 发送 `ask_user_question` 事件

3. **实现 Answer 端点**

* `POST /questions/:questionId/answer`
* 验证每个问题都有答案
* 调用 `pending.resolve(mergedAnswers)`

4. **清理机制**

* SSE 断开时 reject 所有 pending questions
* AbortSignal 处理（SDK 取消时清理）
* 定期清理过期的 pending 记录

### 前端

5. **SSE 事件处理**

* 监听 `ask_user_question` 事件
* 暂停渲染输入框，展示问题表单

6. **问题表单**

* Radio 按钮（单选）
* Checkbox（多选，multiSelect=true）
* 文本输入（自由回答）
* 提交到 Answer 端点

7. **状态管理**

* 提交答案后清除 `pendingQuestion`
* SSE 流仍在继续，后续消息正常渲染

### 输入规范化

8. **normalizeAskUserQuestionInput**

* 验证 question 文本非空
* 验证 options 的 label 非空
* 处理 partial/ malformed 输入

9. **collectProvidedAskUserQuestionAnswers**

* 支持 `string` 和 `string[]` 两种值类型
* 过滤空白值

---

## 常见陷阱

### 1. canUseTool 和 Hook 同时使用

**问题：**`canUseTool` 的 `updatedInput` 会覆盖 Hook 的 `updatedInput`。**解决：** 同一个工具不要同时配置两者。Wait 模式只用 `canUseTool`，Resume 模式只用 Hook。

### 2. toolUseId 匹配检查

**问题：** Resume 模式下，SDK 恢复会话时 AskUserQuestion 会生成新的 `tool_use_id`，和之前的不同。**解决：**`getResumeAnswers` 不要检查 `toolUseId`，或者只在同一会话内检查。

### 3. SSE 流关闭后未清理

**问题：** 客户端断开连接后，pending Promise 永远不 resolve，导致内存泄漏。**解决：** 监听 `request.on('close')`，reject 所有活跃的 pending questions。

### 4. 答案格式不匹配

**问题：** 前端提交的答案 key 必须和 question 文本完全一致（包括空格、标点）。**解决：** 前端从 SSE 事件的 `questions[].question` 取 key，不要硬编码。

---

## 边缘场景处理

### 场景 1：用户关闭浏览器 / 网络断开

**表现：** SSE 流断开，但用户没有提交答案。

**Wait 模式：**

```
浏览器关闭 → request 'close' 事件触发 → reject 所有 pending questions → SDK 收到错误 → 优雅结束
```

关键代码在 `createServer.ts` 的清理逻辑：

```
request.on('close', () => {  
  abortController.abort()          // 通知 SDK 取消  
  rejectActiveQuestions('...')     // reject 所有等待中的 Promise  
})
```

SDK 收到 abort 后会取消当前操作，SSE 流写入 error 事件后关闭。

**Resume 模式：**Run 进入 `awaiting_input` 状态，SSE 流正常关闭。问题记录保留在 `run.pendingQuestions` 中。用户可以稍后重新连接并提交答案。

### 场景 2：Claude 连续发出多次 AskUserQuestion

SDK 的消息循环是顺序的——一次只有一个工具在执行。所以不会出现两个 AskUserQuestion 同时 pending 的情况：

```
Claude 循环：  
  1. 调用 AskUserQuestion #1 → canUseTool → await Promise ... → 用户回答 → 继续  
  2. 处理上一步的结果  
  3. 调用 AskUserQuestion #2 → canUseTool → await Promise ... → 用户回答 → 继续
```

`activeQuestionIds` Set 记录当前活跃的问题，但同一时刻最多只有一个。

### 场景 3：定时任务（非交互式）遇到 AskUserQuestion

定时任务通过 `claudeAgentExecutor` 执行，没有 SSE 流和前端。这种场景下：

* 没有 `canUseTool` 回调（使用 `createDeniedPermissionCanUseTool`，对所有工具返回 deny）
* AskUserQuestion 被 deny → Claude 收到拒绝消息 → Claude 需要自己决定如何继续
* Claude 通常会绕过问题，用默认假设继续工作

**如果想让定时任务也能回答 AskUserQuestion：**可以提供一个自动回答的 `canUseTool`：

```
canUseTool: async (toolName, input) => {  
  if (toolName !== 'AskUserQuestion') return { behavior: 'deny', message: '...' }  
  // 自动选择第一个选项作为答案  
  const autoAnswers: Record<string, string> = {}  
  for (const q of input.questions) {  
    autoAnswers[q.question] = q.options?.[0]?.label ?? 'proceed'  
  }  
  return { behavior: 'allow', updatedInput: { ...input, answers: autoAnswers } }  
}
```

### 场景 4：AbortSignal 触发（SDK 主动取消）

当 SDK 发出 abort 信号时（如超时、父请求取消）：

```
SDK abort → options.signal 触发 'abort' 事件  
  → handleAbort() 执行  
    → 从 pendingQuestions Map 删除  
    → reject(new Error('Aborted'))  
  → canUseTool 的 Promise reject  
  → SDK 捕获错误，优雅结束
```

注意清理顺序：

1. 先从 Map 中删除（防止 answer 端点找到已 abort 的问题）
2. 再 reject Promise
3. 移除 abort 事件监听器

### 场景 5：Answer 端点收到已过期的 questionId

**表现：** 用户在页面停留很久后提交答案，但 SSE 流已经关闭，pending 记录已被清理。

**处理：**

```
const pending = pendingChatQuestions.get(request.params.questionId)  
if (!pending) {  
  return response.status(404).json({ error: 'Pending AskUserQuestion prompt not found' })  
}
```

前端收到 404 后应提示用户"问题已过期，请重新发送消息"。

### 场景 6：Placeholder-then-overwrite 模式

我们的代码中有一个看似奇怪的模式——先创建空的 resolve/reject，然后在 Promise 内部覆盖：

```
// 第一步：创建带空 resolve/reject 的记录  
pendingChatQuestions.set(questionId, {  
  resolve: () => {},   // 占位  
  reject: () => {}  
})  
  
// 第二步：在 Promise executor 内覆盖  
return new Promise((resolve, reject) => {  
  const pending = pendingChatQuestions.get(questionId)!  
  pending.resolve = (answers) => { resolve(answers) }  
  pending.reject = (error) => { reject(error) }  
})
```

**为什么这样做？** 因为 pending 记录需要在 Promise 创建**之前**就写入 Map（SSE 事件要引用 questionId），但 Promise 的 resolve/reject 只能在 Promise executor 内部获取。两者在同一个同步执行上下文中运行，所以不存在竞态条件。

### 场景 7：permissionMode 切换到 plan 模式

Plan 模式下，SDK 只允许只读工具（Read、Grep、Glob 等）。AskUserQuestion 仍然会触发 `canUseTool`（因为 `requiresUserInteraction()=true`），交互流程不变。

但注意：如果 `canUseTool` 对非 AskUserQuestion 工具返回 `deny`，在 plan 模式下这些工具本来就不会执行，所以 deny 也不会造成问题。

### 场景 8：并发多个 Chat 会话

每个 `POST /api/chat/send` 请求创建独立的：

* `activeQuestionIds` Set（请求作用域）
* `AbortController`（请求作用域）
* `pendingChatQuestions` Map 的条目（模块作用域，但 questionId 是 UUID，不会冲突）

多个会话可以并行运行，各自的 AskUserQuestion 互不干扰。Answer 端点通过 `questionId`（UUID）路由到正确的 pending Promise。

---

## 决策速查表

| 你要做什么 | 选择 |
| --- | --- |
| 浏览器 Chat 交互 | Wait 模式 + `canUseTool` |
| API Agent Run，需要断点恢复 | Resume 模式 + `PreToolUse` Hook |
| 定时任务，全自动 | 不配置交互回调，或自动选择默认答案 |
| 同时支持两种场景 | Agent Run 的 `askUserQuestionMode` 参数切换 |
| 在 `bypassPermissions` 下拦截 AskUserQuestion | 只配 `canUseTool`，不用 Hook |
| 在非 `bypassPermissions` 下拦截 AskUserQuestion | 可以用 Hook（defer），也可以用 `canUseTool` |