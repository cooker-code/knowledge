---
title: Vercel AI SDK：构建现代 Web AI 应用指南
author: 奇舞精选
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg4MTYwMzY1Mw==&mid=2247517602&idx=1&sn=c92e5dd68da3cf8d61eefcf5ad407192&chksm=ce24c008239b2ac22342b8546f30c7cfb0082d0347f2f603e46d602da7adac48bec32bb0296d&mpshare=1&scene=24&srcid=1210Ay4tW2WBh0kNbAxQqVI5&sharer_shareinfo=c188c32ba55dd03fc5aa6403ccfcc1d5&sharer_shareinfo_first=c188c32ba55dd03fc5aa6403ccfcc1d5#rd
---

> 本文作者为 360 奇舞团前端开发工程师

## 简介

随着大语言模型（LLM）越来越普及，将它们集成到 Web 应用、聊天机器人等场景中成为很多开发者的诉求。但不同模型提供商（DeepSeek、OpenAI、Anthropic、Google 等）在接口、请求方式、返回格式等方面差异很大。 此外，不同 Web 框架（React／Next.js、Vue／Nuxt、Node.js 等）也带来了兼容性和工程复杂性。

Vercel 推出的 Vercel AI SDK，旨在通过 **统一 API + 框架无关 UI 组件 + 多模型适配器** 的设计，将 “调用 LLM + 构建对话/生成 UI” 的复杂性封装起来，让开发者可以更专注于业务逻辑，而不必过多关注模型和框架差异。换句话说，Vercel AI SDK 是为“把 AI（LLM）接入现代 Web 应用” — 从原型到生产 — 提供一种标准、方便、高效的“中间层/基础设施”。如果你在做 ChatBot、知识库问答等项目时，Vercel AI SDK 能大幅降低上下游集成和工程复杂度。

当前，Vercel AI SDK 最新版本为 **AI SDK 6**，不过目前它处于 **Beta 阶段**。如果你不想用 Beta 版本，可以继续使用稳定版，即 AI SDK 5。

Vercel AI SDK 官网：https://ai-sdk.dev

Vercel AI SDK 开源地址：https://github.com/vercel/ai

## 核心模块

**AI SDK Core**

提供统一 API 用于调用语言模型／生成文本、结构化对象／数组、工具调用、流式响应等。通过统一接口屏蔽不同模型 Provider 的差异。SDK 本身不绑定任何模型，而是通过 Provider 机制适配众多模型提供商 (LLM)。如官方支持 DeepSeek、OpenAI、Anthropic、Google、xAI Grok 等，还支持自定义 Provider。

**AI SDK UI**

提供面向前端／前端框架 (React、Vue、Svelte、Angular 等) 的 hooks 或组件 (如 `useChat`, `useCompletion` 等)，方便构建聊天界面、生成式 UI、流式聊天界面、Agent UI 等。可以在多种前端架构中复用。

## 各库 (Package) 功能职责与适用场景

### `ai` — Core SDK

* Core 核心包，提供统一、框架无关的 API，用于调用任意兼容 Provider 的 LLM
* 支持基础用例：文本生成、结构化数据生成、流式响应 (streaming)
* 支持多模型，通过 Provider Adapter 机制，如 `@ai-sdk/openai`) 来适配各家模型，将它们在统一的接口下抽象。

**何时使用**`ai`：

* 你需要后端 / API route 等调用 LLM、生成文本、结构化输出、执行工具调用 (例如调用外部 API、数据库、文件系统等)，并将结果处理、存储、转发
* 构建 “Workflow / 后端服务 / CLI 工具 ” 等，不需要前端 UI
* 跨 Provider (DeepSeek / OpenAI / Google / Anthropic ) 编写通用逻辑，不想为每个提供商写独立处理。

**如何安装 / 依赖**：

```
npm install ai
```

### `@ai-sdk/openai` — OpenAI Provider 适配包

* “模型 Provider 适配包” — 用于当你希望使用 OpenAI (或兼容 OpenAI API) 作为 LLM Provider 时，把 OpenAI 整合进 Core；它实现了 Provider 接口。
* 提供对 OpenAI 规范功能的封装。

**何时使用**`@ai-sdk/openai`：

* 你计划用 OpenAI 的模型 (GPT-4, GPT-4o, GPT-5, 其他 OpenAI 模型) 作为你的 LLM。
* 你计划用 兼容 OpenAI 接口规范的模型作为你的 LLM。

**如何安装**：

```
npm install @ai-sdk/openai
```

同时你还需要 `ai` (core) 才能一起使用。

**示例**：

```
import { generateText } from 'ai';  
import { openai } from '@ai-sdk/openai';  
  
const { text } = await generateText({  
  model: openai('gpt-4o'),  
  prompt: '介绍一下 Vercel AI SDK',  
});
```

### `@ai-sdk/react` — React UI 层

* 为 React 应用 (包括基于 React 的框架，如 Next.js) 提供 UI 层抽象 (hooks / 组件)，方便在前端构建聊天 UI /生成 UI /流式 UI。
* 常见提供的 hook 包括 `useChat`, `useCompletion`, `useObject` 等 —— 它们封装了状态管理 (Messages 列表、用户输入、提交、Request/Response 状态)、UI 更新、与后端 (或由 `ai` 包提供的 API) 的交互流程。
* 支持与 `ai` + Provider 组合使用，通过前端 + 后端 构建完整的 ChatBot 应用。

**何时使用**`@ai-sdk/react`：

* 你在做前端项目 (SPA / 静态网页 / Next.js / React) 并希望直接给用户展示交互界面 (聊天框、对话、生成内容、流式输出等)。
* 想快速构建 UI，不用自己从头管理请求 / 响应 /消息列表 /状态 /loading /streaming/ UI 更新。

**如何安装**：

```
npm install @ai-sdk/react
```

\*\*示例 (React / Next.js)\*\*：

```
const { messages, sendMessage, status, error } = useChat({  
  transport: new TextStreamChatTransport({  
    api: "/api/chat",  
  }),  
});
```

后端（Next.js API route）使用 `ai` + `@ai-sdk/openai` 产生响应。

## 根据项目需求选择包

* 如果你的项目 **只需要后端 / 对 LLM 进行调用 / 生成文本** —— 只安装 `ai` + 对应 Provider (例如 `@ai-sdk/openai`)。
* 如果你的项目 **涉及前端 / 用户交互 / Chat UI /对话界面 /生成 UI** —— 那建议也加上 `@ai-sdk/react` (或视你前端框架为 `@ai-sdk/vue`, `@ai-sdk/svelte` 等) 来处理 UI/交互逻辑。
* `@ai-sdk/openai` (或其他 Provider 包) 是必须的，否则 `ai` 就不知道用哪家模型服务。

## 快速上手

以下是一个示例 "利用 Vercel AI SDK 构建简单聊天" 的实践。

### 步骤 1: 创建 Next.js 项目

```
# 创建一个新的 Next.js 项目  
npx create-next-app@latest ai-chatbot  
  
# 进入项目目录  
cd ai-chatbot
```

在创建过程中，选择以下选项：

* ✅ TypeScript
* ✅ ESLint
* ✅ Tailwind CSS
* ✅ App Router
* 🔘 src/ directory (可选)
* ✅ Turbopack (可选)

### 步骤 2: 安装 Vercel AI SDK 依赖

```
# 安装核心包和 OpenAI Provider  
npm install ai @ai-sdk/openai @ai-sdk/react  
  
# 或者使用其他 Provider，例如:  
# npm install ai @ai-sdk/anthropic @ai-sdk/react  # Anthropic Claude  
# npm install ai @ai-sdk/google @ai-sdk/react     # Google Gemini
```

### 步骤 3: 配置环境变量

在项目根目录创建 `.env.local` 文件（这里我们使用Kimi）：

```
# OpenAI API Key  
OPENAI_API_KEY=your_kimi_api_key_here  
  
# 如果使用兼容 OpenAI 的服务（如 Kimi），可以配置基础 URL  
OPENAI_BASE_URL=https://api.moonshot.cn/v1
```

### 步骤 4: 创建后端 API Route

创建文件 `app/api/chat/route.ts`：

```
import { openai } from'@ai-sdk/openai';  
import { streamText } from'ai';  
  
// 允许流式响应最多 30 秒  
exportconst maxDuration = 30;  
  
exportasyncfunction POST(req: Request) {  
// 从请求体中提取消息  
const { messages } = await req.json();  
  
// 调用 LLM 生成流式响应  
const result = streamText({  
    model: openai('gpt-4o'),  
    messages  
  });  
  
// 返回流式响应  
return result.toTextStreamResponse();  
}
```

**如果使用 Kimi（兼容 OpenAI API）**：

```
import { createOpenAI } from"@ai-sdk/openai";  
import { streamText, convertToModelMessages } from"ai";  
  
// 创建 Kimi Provider (Moonshot AI)  
const kimiClient = createOpenAI({  
  baseURL: "https://api.moonshot.cn/v1",  
  apiKey: process.env.KIMI_API_KEY,  
});  
  
exportconst maxDuration = 30;  
  
exportasyncfunction POST(req: Request) {  
const { messages } = await req.json();  
  
const result = streamText({  
    model: kimiClient.chat("moonshot-v1-8k"),  
    messages: convertToModelMessages(messages),  
  });  
return result.toTextStreamResponse();  
}
```

### 步骤 5: 创建前端聊天页面

修改 `app/page.tsx`：

```
"use client";  
  
import { useState } from"react";  
import { useChat } from"@ai-sdk/react";  
import { TextStreamChatTransport } from"ai";  
  
exportdefaultfunction ChatPage() {  
const [input, setInput] = useState("");  
const { messages, sendMessage, status, error } = useChat({  
    transport: new TextStreamChatTransport({  
      api: "/api/chat",  
    }),  
  });  
  
const isLoading = status === "streaming" || status === "submitted";  
  
return (  
    <main className="flex min-h-screen flex-col items-center justify-between p-8">  
      <div className="w-full max-w-2xl flex flex-col h-[80vh]">  
        <h1 className="text-3xl font-bold mb-6 text-center">AI 聊天助手</h1>  
  
        {/* 消息列表 */}  
        <div className="flex-1 overflow-y-auto mb-4 space-y-4 p-4 bg-gray-50 rounded-lg">  
          {messages.length === 0 && (  
            <div className="text-center text-gray-500 mt-10">  
              开始对话吧！向 AI 助手提问任何问题。  
            </div>  
          )}  
  
          {messages.map((message) => {  
            // 提取文本内容  
            const textContent = message.parts  
              ?.filter((part) => part.type === "text")  
              .map((part) => part.text)  
              .join("") || "";  
  
            return (  
              <div  
                key={message.id}  
                className={`flex ${  
                  message.role === "user" ? "justify-end" : "justify-start"  
                }`}  
              >  
                <div  
                  className={`max-w-[80%] rounded-lg px-4 py-2 ${  
                    message.role === "user"  
                      ? "bg-blue-500 text-white"  
                      : "bg-white text-gray-800 border border-gray-200"  
                  }`}  
                >  
                  <div className="text-xs font-semibold mb-1 opacity-70">  
                    {message.role === "user" ? "你" : "AI 助手"}  
                  </div>  
                  <div className="whitespace-pre-wrap">{textContent}</div>  
                </div>  
              </div>  
            );  
          })}  
  
          {isLoading && (  
            <div className="flex justify-start">  
              <div className="bg-white text-gray-800 border border-gray-200 rounded-lg px-4 py-2">  
                <div className="flex space-x-2">  
                  <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce"></div>  
                  <div  
                    className="w-2 h-2 bg-gray-400 rounded-full animate-bounce"  
                    style={{ animationDelay: "0.1s" }}  
                  ></div>  
                  <div  
                    className="w-2 h-2 bg-gray-400 rounded-full animate-bounce"  
                    style={{ animationDelay: "0.2s" }}  
                  ></div>  
                </div>  
              </div>  
            </div>  
          )}  
  
          {error && (  
            <div className="text-center text-red-500 p-4 bg-red-50 rounded-lg">  
              错误：{error.message}  
            </div>  
          )}  
        </div>  
        {/* 输入表单 */}  
        <form  
          onSubmit={(e) => {  
            e.preventDefault();  
            if (input.trim()) {  
              sendMessage({ text: input });  
              setInput("");  
            }  
          }}  
          className="flex gap-2"  
        >  
          <input  
            value={input}  
            onChange={(e) => setInput(e.target.value)}  
            placeholder="输入消息..."  
            disabled={isLoading}  
            className="flex-1 px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:bg-gray-100"  
          />  
          <button  
            type="submit"  
            disabled={isLoading || !input.trim()}  
            className="px-6 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 disabled:bg-gray-300 disabled:cursor-not-allowed transition-colors"  
          >  
            {isLoading ? "发送中..." : "发送"}  
          </button>  
        </form>  
      </div>  
    </main>  
  );  
}
```

### 步骤 6: 运行项目

```
# 启动开发服务器  
npm run dev
```

访问 `http://localhost:3000`，你就可以看到一个完整的 AI 聊天界面了！效果展示：

## 总结

Vercel AI SDK 是目前 Web / JavaScript / TypeScript 生态中，将大语言模型 (LLM) 与现代前端 + 后端框架 (Next.js, React, Vue, Svelte, Node.js, Serverless 等) 整合得最完整、最方便、最全面的开源 SDK 之一。

* 对于想快速构建聊天机器人、生成式应用、agent、文档分析、RAG 系统、多模态应用等的人来说，它大幅降低了技术门槛和工程成本；
* 对于需要支持多模型、多 provider、跨框架、跨运行时 (server / edge / frontend) 的项目，它提供了稳定、统一、可维护、可扩展的基础设施；
* 对于只是想在短时间内做原型 / MVP / demo，也非常友好。

当然，对于极度定制、高性能、特殊模型 / 极端场景，你仍可能需要绕开 SDK，自己构建更底层或更专门化的方案。如果你 **刚开始做 AI + Web 应用** —— Vercel AI SDK 是一个非常推荐的 “最好实践 / 起点 / 基础设施”。

-END -

**如果您关注前端+AI 相关领域可以扫码进群交流**

添加小编微信进群😊

## 关于奇舞团

奇舞团是 360 集团最大的大前端团队，非常重视人才培养，有工程师、讲师、翻译官、业务接口人、团队 Leader 等多种发展方向供员工选择，并辅以提供相应的技术力、专业力、通用力、领导力等培训课程。奇舞团以开放和求贤的心态欢迎各种优秀人才关注和加入奇舞团。