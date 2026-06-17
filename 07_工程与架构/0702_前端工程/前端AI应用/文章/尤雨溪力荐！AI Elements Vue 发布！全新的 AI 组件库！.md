---
title: 尤雨溪力荐！AI Elements Vue 发布！全新的 AI 组件库！
author: 前端开发爱好者
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUzNTk3MjE2Ng==&mid=2247520381&idx=1&sn=7d74786142556474034e825a0042fda8&chksm=fbac7610b4ef2a8f5b0ef63f8cd67f3f53c5bf2c7043ae21f79e4ca9638628adec8636e03984&mpshare=1&scene=24&srcid=1202J4emtwWViPufweQqS5Uv&sharer_shareinfo=511b3f9b065be949294440fafb93de9e&sharer_shareinfo_first=511b3f9b065be949294440fafb93de9e#rd
---

就在一天前，**尤雨溪**在推特上亲自推荐了 **AI Elements Vue**，这款专为 **Vue** 开发者打造的 `AI` 组件库迅速成为前端圈的热议焦点。

作为 `Vue` 官方生态的重要补充，**AI Elements Vue** 的发布标志着 **AI 原生应用开发**正式迈入 `Vue` 时代！

## 🧠 什么是 AI Elements？

**AI Elements** 最初由 **Vercel** 团队推出，是一个基于 **shadcn/ui** 的 React 组件库，专为构建 **AI** 原生应用而设计。

它提供了诸如`对话`、`消息`、`代码块`、`推理展示`、`工具调用`等丰富组件，极大简化了 **AI** 应用的开发流程。

## 🎉 AI Elements Vue 是什么？

**AI Elements Vue** 是由社区主导开发的 **Vue 版本移植项目**，并非 `Vercel` 官方出品，但获得了原作者的认可与支持。

它将 **AI Elements** 的核心能力带到了 **Vue** 生态，基于 **shadcn-vue** 构建，提供同样强大的 **AI** 组件支持。

## ✨ 核心特性

* 🧩 **专为 AI 设计**：提供聊天、推理、代码、工具调用等 `AI` 场景专用组件
* 🎨 **高度可定制**：所有组件都安装到本地，支持完全自定义样式与逻辑
* ⚡ **基于 shadcn-vue**：与 `shadcn-vue` 完美集成，一致的开发体验
* 🧱 **CLI 一键安装**：支持按需安装或全量安装，灵活高效
* 📦 **TypeScript 支持**：完全支持 `TS`，类型安全，开发更安心
* 🌍 **Nuxt 友好**：完美兼容 `Nuxt.js`，支持 `SSR` 场景

## 🚀 快速开始

### ✅ 前置要求

* **Node.js** ≥ `18`
* **Vue** 或 **Nuxt** 项目
* 已安装 **AI SDK**：·`https://ai-sdk.dev/`
* 已初始化 shadcn-vue（`npx shadcn-vue@latest init`）
* 已配置 **Tailwind CSS**（CSS Variables 模式）

### 📦 安装所有组件

```
npx ai-elements-vue@latest
```

### 🧩 安装单个组件

```
npx ai-elements-vue@latest add message  
npx ai-elements-vue@latest add conversation  
npx ai-elements-vue@latest add code-block
```

## 🧾 组件一览（部分）

| 组件名 | 状态 | 功能描述 |
| --- | --- | --- |
| `message` | ✅ | 单个聊天消息（支持头像） |
| `conversation` | ✅ | 聊天对话容器 |
| `response` | ✅ | AI 响应展示 |
| `prompt-input` | ✅ | 高级输入框（支持模型选择） |
| `code-block` | ✅ | 带复制功能的代码块 |
| `image` | ✅ | AI 图像展示 |
| `loader` | ✅ | 加载状态 |
| `suggestion` | ✅ | 快速操作建议 |
| `branch` | ✅ | 对话分支可视化 |
| `reasoning` | ❌ | 推理过程展示（开发中） |
| `tool` | ❌ | 工具调用可视化（开发中） |

## 🧪 示例代码

```
<template>  
  <Conversation>  
    <ConversationContent>  
      <Message v-for="(msg, index) in messages" :key="index" :from="msg.role">  
        <MessageContent>  
          <MessageResponse>{{ msg.content }}</MessageResponse>  
        </MessageContent>  
      </Message>  
    </ConversationContent>  
  </Conversation>  
</template>  
  
<script setup>  
import { useChat } from 'ai-sdk-vue'  
import {  
  Conversation,  
  ConversationContent,  
  Message,  
  MessageContent,  
  MessageResponse,  
} from '@/components/ai-elements'  
  
const { messages } = useChat()  
</script>
```

## 🧭 适用场景

* 🤖 AI 聊天机器人
* 💬 智能客服系统
* 🧠 AI 教育平台
* 🧑‍💻 编程助手
* 📄 文档问答系统
* 🎨 图像生成平台

## ✅ 总结

**AI Elements Vue** 的发布，填补了 **Vue** 生态在 AI 组件领域的空白。

它不仅继承了 **React** 版本的强大功能，还完美结合了 **Vue** 的响应式与组合式特性。

无论你是构建 **AI** 聊天应用、智能助手，还是 **AI** 教育平台，**AI Elements Vue** 都能让你 **更快、更优雅地实现目标**。

> 🎯 现在就用起来吧！只需一行命令，**AI** 应用触手可及！

```
npx ai-elements-vue@latest
```

## 🔗 相关链接

* **GitHub 仓库**：`https://github.com/vuepont/ai-elements-vue`
* **文档地址**：`https://ai-elements-vue.com`
* **原始 React 版本**：`https://github.com/vercel/ai-elements`