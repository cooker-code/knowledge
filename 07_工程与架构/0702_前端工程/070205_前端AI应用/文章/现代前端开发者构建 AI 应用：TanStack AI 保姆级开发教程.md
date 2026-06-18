---
title: 现代前端开发者构建 AI 应用：TanStack AI 保姆级开发教程
author: 前端AI行走
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU5MjYwMDgzNQ==&mid=2247487485&idx=1&sn=f2e600f85fc787b40d1af7f61b9ce666&chksm=ff574a19e06c27ad2202bdb937c03cc92f90eb5dcdd14162a97aa9c8f9ab53957f15110f7734&mpshare=1&scene=24&srcid=12270qKsnufzkWYGY2CJ5qnR&sharer_shareinfo=9763f0a2041fc61ee6ed6612a75ee0da&sharer_shareinfo_first=9763f0a2041fc61ee6ed6612a75ee0da#rd
---

# 前言：

TanStack AI 开发者是谁 ？如果你熟悉 React Query 或 TanStack Table，那么你一定对 TanStack 生态的高质量和开发者体验印象深刻。

**TanStack AI** 是由**TanStack**团队开发的一款跨平台、跨语言的 AI SDK。TanStack AI 是该生态中的最新成员，旨在为前端开发者提供一套**统一、类型安全且灵活**的 AI 应用开发工具。

本教程将手把手带你了解 TanStack AI 的核心概念、使用方法以及它为何如此特别。

## 1. 什么是 TanStack AI？

简单来说，TanStack AI 是一个 **SDK（软件开发工具包）**，它允许你在 JavaScript/TypeScript 应用中轻松集成各种 AI 模型（如 OpenAI、Anthropic、Gemini 等），而无需被锁定在某一家供应商的 API 中。

它就像是 AI 领域的 "通用适配器"，让你用一套代码就能驾驭不同的 AI 大脑。

## 2. 核心部分与传统开发的不同点

在 TanStack AI 出现之前，我们在前端集成 AI 通常有两种方式，但都有痛点：

| 方式 | 痛点 | TanStack AI 的解决方案 |
| --- | --- | --- |
| **直接使用官方 SDK** (如 `openai` npm 包) | 代码与特定厂商绑定，切换模型（如从 GPT-4 切到 Claude 3）需要重写大量代码。 | **多供应商支持** ：提供统一接口，切换模型只需改配置，无需改业务代码。 |
| **使用 LangChain 等后端重框架** | 概念复杂，学习曲线陡峭，且对前端开发者不够友好（通常偏向 Python 或 Node.js 后端逻辑）。 | **前端优先 & 全栈友好** ：专为 JS/TS 开发者设计，完美支持 React, Vue, Solid 等框架，且支持边缘计算 (Edge Runtime)。 |
| **手动处理流式数据 (Streaming)** | 处理 SSE (Server-Sent Events) 和打字机效果非常繁琐，容易出错。 | **内置流式支持** ：开箱即用的流式数据处理，轻松实现打字机效果。 |
| **缺乏类型安全** | 调用 AI 工具或解析返回的 JSON 时，经常遇到类型错误。 | **端到端类型安全** ：基于 TypeScript 和 Zod，确保输入输出完全符合预期。 |

## 3. 核心亮点 (Highlights) ✨

1. **供应商无关 (Vendor Agnostic)**:

* 想用 GPT-5.2？没问题。
* 想换成 Claude 4.5 Sonnet？一键切换。
* 想用本地的 Ollama？完全支持。
* **亮点**：你不再受制于单一 AI 公司的涨价或服务不稳定。

2. **极致的类型安全 (Type-Safety)**:

* 利用 TypeScript 和 Zod，TanStack AI 让你在编写代码时就能发现错误，而不是在运行时才崩溃。
* **亮点**：定义 AI 工具 (Tools) 时，参数类型自动推导，开发体验极佳。

3. **同构工具 (Isomorphic Tools)**:

* 定义的工具函数既可以在服务器端运行，也可以在客户端运行。
* **亮点**：支持 "Human in the loop"（人工介入），例如 AI 想要执行一个敏感操作（如删除文件），可以自动触发前端弹窗让用户确认。

4. **Headless UI (无头组件)**:

* 提供了处理聊天逻辑、状态管理的核心 Hook，但不绑定任何 UI 样式。
* **亮点**：你可以随意使用 Tailwind CSS、Material UI 或 Ant Design 来绘制聊天界面，完全掌控视觉效果。

5. **TanStack DevTools 集成**:

* **亮点**：拥有专门的调试面板，可以清晰地看到 AI 的思考过程、工具调用详情和数据流向，调试 AI 应用不再是黑盒。

## 4. 如何使用 (保姆级步骤)

假设我们使用 **React** 框架来构建一个简单的聊天应用。

### 第一步：安装

```
npm install @tanstack/ai @tanstack/ai-react @tanstack/ai-openai
```

### 第二步：配置 Provider (以 DeepSeek 为例)

DeepSeek 完美兼容 OpenAI 的 API 格式，所以我们可以直接使用 `createOpenAI` 但指向 DeepSeek 的地址。

```
import { createOpenAI } from '@tanstack/ai/providers/openai'  
  
const deepseek = createOpenAI({  
  baseURL: 'https://api.deepseek.com',  
  apiKey: process.env.DEEPSEEK_API_KEY,  
})
```

### 第三步：构建聊天界面 (使用 `useChat`)

这是最核心的部分。`useChat` 帮你管理了所有的对话历史、加载状态和输入处理。

```
import { useChat } from '@tanstack/react-ai'  
  
function ChatApp() {  
  const {   
    messages,    // 当前的对话列表  
    input,       // 输入框的值  
    handleInputChange, // 输入框变化处理函数  
    handleSubmit,      // 提交表单处理函数  
    status       // 当前状态: 'idle' | 'streaming' | 'submitting'  
  } = useChat({  
    // 这里配置你的后端 API 路径，TanStack AI 会自动处理请求  
    api: '/api/chat',   
    defaultModel: 'deepseek-chat',  
  })  
  
  return (  
    <div className="chat-container">  
      {/* 1. 消息列表展示 */}  
      <div className="messages">  
        {messages.map(m => (  
          <div key={m.id} className={`message ${m.role}`}>  
            <strong>{m.role === 'user' ? '你' : 'AI'}:</strong>  
            {m.content}  
          </div>  
        ))}  
      </div>  
  
      {/* 2. 输入框区域 */}  
      <form onSubmit={handleSubmit}>  
        <input  
          value={input}  
          onChange={handleInputChange}  
          placeholder="说点什么..."  
          disabled={status === 'streaming'}  
        />  
        <button type="submit" disabled={status === 'streaming'}>  
          发送  
        </button>  
      </form>  
    </div>  
  )  
}
```

### 第四步：后端处理 (API Route)

在你的后端 (如 Next.js API Route) 处理请求。

```
// app/api/chat/route.ts  
import { streamText } from'@tanstack/ai'  
import { deepseek } from'@/utils/ai-providers'// 刚才配置的  
  
exportasyncfunction POST(req: Request) {  
const { messages } = await req.json()  
  
// 核心魔法：streamText  
const result = await streamText({  
    model: deepseek.chat('deepseek-chat'),  
    messages,  
  })  
  
// 直接返回流式响应，前端 useChat 会自动解析  
return result.toAIStreamResponse()  
}
```

## 5. 进阶：使用工具 (Tools)

这是 AI 最强大的地方——让它能够执行代码或查询数据。

```
import { tool } from'@tanstack/ai'  
import { z } from'zod'  
  
// 定义一个工具：查询天气  
const weatherTool = tool({  
  description: '获取指定地点的当前天气',  
  parameters: z.object({  
    location: z.string().describe('城市名称，如 Beijing'),  
  }),  
  execute: async ({ location }) => {  
    // 这里写真实的 API 调用逻辑  
    const temp = 25// 假设获取到的温度  
    return { temperature: temp, condition: 'Sunny' }  
  },  
})  
  
// 在后端调用时加入工具  
streamText({  
  model: deepseek.chat('deepseek-chat'),  
  messages,  
  tools: {  
    weather: weatherTool, // 注册工具  
  },  
})
```

**效果**：当你问 "北京今天天气怎么样？" 时，AI 不会瞎编，而是会自动调用 `weatherTool`，获取真实数据后回答你。

## 6. 相关案例 (Use Cases)

1. **智能客服机器人 (Customer Support)**

* **场景**：用户在电商网站询问 "我的订单发货了吗？"。
* **实现**：使用 TanStack AI 连接数据库查询工具，AI 自动调用 `getOrderStatus` 工具查询订单状态并回复用户，全程无需人工介入。

2. **RAG 内部知识库 (Retrieval Augmented Generation)**

* **场景**：公司内部员工询问 "去年的报销政策是什么？"。
* **实现**：AI 先调用 "文档检索工具" 在公司 Wiki 中查找相关段落，然后基于检索到的内容生成准确的回答，避免 AI 产生幻觉。

3. **多模态应用 (Multimodal Apps)**

* **场景**：用户上传一张冰箱食材的照片，问 "我能做什么菜？"。
* **实现**：利用 TanStack AI 对多模态模型的支持 (如 DeepSeek-VL)，识别图片内容，并结合菜谱 API 推荐食谱。

4. **教育辅助工具 (Ed-Tech)**

* **场景**：学生做题遇到困难。
* **实现**：AI 扮演苏格拉底式导师，不直接给出答案，而是通过一步步提问引导学生自己思考，利用 Prompt Engineering 和状态管理控制教学节奏。

## 总结

TanStack AI 是现代前端开发者构建 AI 应用的**最佳切入点**。它剥离了复杂的底层细节，让你专注于业务逻辑和用户体验。

### 核心价值回顾

通过本教程，我们了解到 TanStack AI 的五大核心优势：

1. **供应商无关性** - 一套代码适配多个 AI 模型，告别厂商锁定
2. **类型安全** - TypeScript + Zod 确保端到端的类型安全
3. **同构工具** - 支持服务端和客户端无缝协作，实现 "Human in the loop"
4. **Headless 设计** - 完全掌控 UI，自由选择样式方案
5. **强大的调试能力** - DevTools 集成，让 AI 应用开发透明可观测

### 适用场景总结

TanStack AI 特别适合以下场景：

* **需要快速集成 AI 功能的前端项目** - 无论是聊天机器人、智能助手还是内容生成工具
* **多模型切换需求** - 需要在不同 AI 供应商之间灵活切换或 A/B 测试
* **类型安全要求高的项目** - 企业级应用需要严格的类型检查
* **需要工具调用 (Function Calling) 的应用** - 让 AI 能够执行真实操作，而不仅仅是对话
* **全栈应用** - Next.js、Remix 等框架的完美搭档，支持边缘计算

### 学习路径建议

如果你准备开始使用 TanStack AI，建议按以下顺序学习：

1. **基础阶段**：从简单的聊天应用开始，熟悉 `useChat` Hook 和 `streamText` 的基本用法
2. **进阶阶段**：学习工具 (Tools) 的定义和使用，实现 AI 与外部系统的交互
3. **高级阶段**：探索多模态支持、RAG 集成、自定义 Provider 等高级特性
4. **实战阶段**：将所学应用到实际项目中，如智能客服、知识库问答等场景

### 与其他方案的对比

| 特性 | TanStack AI | 直接使用官方 SDK | LangChain |
| --- | --- | --- | --- |
| **学习曲线** | ⭐⭐⭐ 平缓 | ⭐⭐⭐⭐ 简单 | ⭐⭐ 陡峭 |
| **类型安全** | ✅ 完整支持 | ⚠️ 部分支持 | ⚠️ 部分支持 |
| **供应商切换** | ✅ 无缝切换 | ❌ 需要重写 | ✅ 支持 |
| **前端友好度** | ✅ 专为前端设计 | ⚠️ 通用 SDK | ❌ 偏向后端 |
| **调试体验** | ✅ DevTools 支持 | ⚠️ 基础日志 | ⚠️ 基础日志 |
| **流式处理** | ✅ 开箱即用 | ⚠️ 需手动实现 | ✅ 支持 |

### 下一步行动

* 如果你想要**灵活**（随意换模型）；
* 如果你喜欢**TypeScript**（类型安全）；
* 如果你追求**开发体验**（DevTools 支持）；
* 如果你需要**快速上手**（简洁的 API 设计）；

那么，TanStack AI 绝对值得你尝试！

**立即开始**：

1. 访问 TanStack AI 官方文档
2. 在项目中运行 `npm install @tanstack/ai @tanstack/react-ai zod`
3. 按照本教程的步骤，构建你的第一个 AI 应用

**记住**：TanStack AI 的目标不是替代所有 AI 开发方案，而是为前端开发者提供一个**更优雅、更安全、更灵活**的选择。在 AI 应用开发的道路上，选择对的工具，往往比写更多的代码更重要。