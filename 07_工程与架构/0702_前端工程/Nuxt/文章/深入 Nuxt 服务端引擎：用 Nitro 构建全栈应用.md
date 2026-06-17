---
title: 深入 Nuxt 服务端引擎：用 Nitro 构建全栈应用
author: 文艺理科生Owen
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI0NzMyNTU0MA==&mid=2247483942&idx=1&sn=1f12179290c43fc6e8c5be305d1006f6&chksm=e87f753bbaaa0eab4e46a0341f92c7cc95eaff75f1b0d48e5bd20b5e2d479d59ff18870eb206&mpshare=1&scene=24&srcid=1203QdVkOPIp4q9oHhjYjxEA&sharer_shareinfo=5f3b9faa1dc7624263b1491213840441&sharer_shareinfo_first=5f3b9faa1dc7624263b1491213840441#rd
---

# 深入 Nuxt 服务端引擎：用 Nitro 构建全栈应用

> 摘要：当你的 Nuxt 应用不再满足于纯粹的前端渲染，而是渴望拥有自己的后端逻辑、处理数据、甚至连接数据库时，你是否认为需要再独立创建一个 Node.js 或 Go 项目？Nuxt 3 给出的答案是：完全不需要！借助其内置的服务端引擎——Nitro，你可以在同一个项目中，无缝地从前端开发者“进化”为全栈工程师。本文将带你深入 Nitro 的世界，从创建第一个 API 端点开始，直到构建一个完整的前后端分离的迷你应用。

---

## 引言

在完成了 Nuxt.js 入门 和 组件渲染进阶 的学习后，我们已经能熟练地构建高性能的 Vue 前端应用。但现代 Web 应用往往是“前后端一体”的。传统的开发模式需要我们维护两个独立的项目：一个前端项目（如 Nuxt）和一个后端项目（如 Express, Koa, NestJS）。

Nuxt 3 彻底改变了这一格局。它集成的 **Nitro** 服务端引擎，不仅仅是一个开发服务器，更是一个功能完备、性能卓越的后端框架。这意味着，你可以在 `server/` 目录下，用你熟悉的 TypeScript/JavaScript 语法，直接编写 API、连接数据库、处理业务逻辑。

本文的目标，就是带领已经熟悉 Nuxt 前端开发的你，迈出走向全栈的关键一步。读完本文，你将掌握：

* • 在 Nuxt 项目中创建 API 端点的核心方法。
* • Nitro 如何实现前后端的无缝、高效协作。
* • 利用服务端中间件处理通用逻辑（如认证、日志）。
* • 从零到一，构建一个功能完整的全栈应用。

## 1. 欢迎来到 Nitro 的世界：Nuxt 的全栈心脏

**Nitro** 是 Nuxt 的秘密武器。它是一个轻量级、高性能的 Web 服务器框架，专为现代 JavaScript 应用设计。

**它如何赋能 Nuxt？**

* • **零配置 API**：你只需在特定目录创建文件，Nitro 会自动将其注册为 API 路由。
* • **代码分割与摇树优化**：不仅对前端代码，Nitro 对后端代码同样进行优化，只打包被实际用到的代码，确保服务端启动快、体积小。
* • **跨平台部署**：Nitro 可以将你的应用打包成多种格式，轻松部署到 Node.js、Vercel、Netlify、Cloudflare Workers 等多种平台，无需修改代码。
* • **与前端无缝集成**：最强大的特性，我们稍后会详细探讨。

要开始使用 Nitro，你需要了解 `server/` 目录的核心约定：

* • `server/api/`: 存放你的 API 端点。例如 `server/api/hello.ts` 会被映射到 `/api/hello`。
* • `server/routes/`: 用于创建非 API 的服务端路由，比如生成 sitemap.xml 或 robots.txt。
* • `server/middleware/`: 存放服务端中间件，它们会在每个 API 请求到达处理器之前执行。

## 2. Nitro 初体验：你的第一个 API 端点

让我们卷起袖子，创建两个最基础的 API：一个 GET 请求和一个 POST 请求。

### 创建 GET 请求

在你的 Nuxt 项目中，创建文件 `server/api/hello.ts`：

```
// server/api/hello.ts  
export default defineEventHandler((event) => {  
  return {  
    message: 'Hello, Nitro!',  
    timestamp: new Date().toISOString()  
  }  
})
```

`defineEventHandler` 是 Nitro 提供的核心函数，用于定义请求处理器。现在，启动你的 Nuxt 开发服务器 (`npm run dev`)，然后在浏览器中访问 `http://localhost:3000/api/hello`，你将看到返回的 JSON 数据。就是这么简单！

### 创建 POST 请求

Nitro 会根据文件名自动推断请求方法。创建一个名为 `submit.post.ts` 的文件，它将只响应 POST 请求。

创建 `server/api/submit.post.ts`：

```
// server/api/submit.post.ts  
export default defineEventHandler(async (event) => {  
  // 使用 readBody 工具函数读取请求体  
  const body = await readBody(event)  
  
  return {  
    received: true,  
    data: body  
  }  
})
```

你可以使用 Postman 或 `curl` 来测试它：

```
curl -X POST -H "Content-Type: application/json" -d '''{"name": "Alice", "age": 30}''' http://localhost:3000/api/submit
```

服务器将返回你发送的数据，确认接收成功。

## 3. 前后端闭环：`useFetch` 的同构魔法

API 创建好了，前端组件如何调用呢？当然是使用 Nuxt 提供的 `useFetch` 组合式函数。

假设我们有一个页面 `pages/test-api.vue`：

```
<template>  
  <div>  
    <h1>API 测试</h1>  
    <button @click="fetchData">获取数据</button>  
    <pre v-if="data">{{ data }}</pre>  
  </div>  
</template>  
  
<script setup>  
const { data, pending, error, execute: fetchData } = useFetch('/api/hello', {  
  immediate: false // 设置为 false，不在组件加载时立即执行  
})  
</script>
```

`useFetch` 的真正魔力在于它的 **“同构（Isomorphic）”** 特性。这意味着它在不同环境下的行为是智能优化的：

1. 1. **服务端渲染 (SSR)**：当用户首次访问或刷新 `test-api` 页面时，Nuxt 服务器在渲染 HTML 的过程中遇到 `useFetch`。它不会发起一个真实的 HTTP 请求到 `localhost:3000`，而是**直接在内部调用 `server/api/hello.ts` 的函数**。这避免了网络开销，速度极快。
2. 2. **客户端导航 (CSR)**：当用户在应用内通过 `<NuxtLink>` 从其他页面跳转到 `test-api` 页面时，`useFetch` 会在浏览器中**发起一个真实的 `fetch` 请求**到 `/api/hello`。

这种智能切换对开发者是完全透明的，你只需要写一次 `useFetch`，就能同时享受到 SSR 的高性能和 CSR 的灵活性。

下面的流程图清晰地展示了这两种场景：

```
场景一：首次加载 (SSR)

直接在服务端内部调用函数

浏览器发起请求: GET /test-api

Nuxt 服务器

执行 pages/test-api.vue 的 setup

useFetch('/api/hello')

执行 server/api/hello.ts

返回数据

生成包含数据的完整 HTML

场景二：客户端导航

发起一个真实的 fetch 请求

用户点击链接

客户端 Vue

useFetch('/api/hello')

Nuxt 服务器 (Nitro)

执行 server/api/hello.ts

返回 JSON 数据

更新组件视图
```

## 4. 动态路由：处理 API 参数

静态 API 无法满足所有需求。如果想获取特定用户的信息，如 `/api/users/1`，就需要动态路由。

在 `server/api/` 目录下，使用方括号语法创建动态路由文件 `server/api/users/[id].ts`：

```
// server/api/users/[id].ts  
exportdefaultdefineEventHandler((event) => {  
// 从 event.context.params 中获取动态参数  
const userId = event.context.params.id  
  
// 模拟数据库查询  
const users = {  
    '1': { id: 1, name: 'Alice', email: 'alice@example.com' },  
    '2': { id: 2, name: 'Bob', email: 'bob@example.com' }  
  }  
  
if (!users[userId]) {  
    // 设置响应状态码并返回错误信息  
    setResponseStatus(event, 404)  
    return { error: 'User not found' }  
  }  
  
return users[userId]  
})
```

现在访问 `/api/users/1` 会返回 Alice 的信息，而访问 `/api/users/99` 则会返回 404 错误。

## 5. 服务端中间件：API 的“守卫”

当多个 API 需要共享某些逻辑时，比如日志记录、权限校验，为每个 API 单独编写这些代码会非常繁琐。这时，服务端中间件就派上用场了。

中间件是位于请求和最终处理器之间的“关卡”。一个请求从进入 Nitro 服务器到被 API 处理器响应，会依次通过所有定义的中间件。

```
API 处理器中间件2 (auth.ts)中间件1 (log.ts)Nitro 服务器客户端API 处理器中间件2 (auth.ts)中间件1 (log.ts)Nitro 服务器客户端alt[认证成功][认证失败]发起请求 (e.g., GET /api/admin/data)按顺序执行处理完毕, next()按顺序执行认证通过, next()分发给对应的 API 处理器返回处理结果响应 200 OK 和数据抛出错误或直接响应响应 401 Unauthorized
```

### 示例1：日志中间件

创建 `server/middleware/log.ts`。Nitro 会自动加载它。

```
// server/middleware/log.ts  
export default defineEventHandler((event) => {  
  console.log(`[${new Date().toLocaleTimeString()}] New request: ${getRequestURL(event)}`)  
})
```

现在，每一次对 API 的请求，都会在服务端的控制台打印出一条日志。

### 示例2：模拟认证中间件

假设我们想保护所有 `/api/admin/` 下的路由。

创建 `server/middleware/auth.ts`：

```
// server/middleware/auth.ts  
exportdefaultdefineEventHandler((event) => {  
const url = getRequestURL(event)  
  
// 只对 /api/admin/ 下的路由生效  
if (url.pathname.startsWith('/api/admin/')) {  
    // 从请求头中获取 Authorization  
    const token = getHeader(event, 'Authorization')  
      
    if (token !== 'Bearer my-secret-token') {  
      // 抛出一个错误，Nitro 会自动转换为 401 响应  
      throwcreateError({  
        statusCode: 401,  
        statusMessage: 'Unauthorized',  
      })  
    }  
    // 认证通过，可以在 event.context 中附加用户信息  
    event.context.user = { name: 'Admin' }  
  }  
})
```

这个中间件会检查请求头，如果认证失败，则直接中断请求并返回 401。如果成功，它还可以在 `event.context` 上附加信息，供后续的处理器使用。

## 6. 实战演练：构建一个迷你待办事项应用 (Todo List)

理论讲完了，让我们把所有知识点串联起来，构建一个完整的待办事项应用。

### 应用架构

我们的应用非常简单，前端一个 Vue 页面，后端提供四个 API 端点来对数据进行增删改查（CRUD）。

```
Nitro (服务端)

浏览器 (客户端)

useFetch 获取列表

useFetch 新增

useFetch 更新

useFetch 删除

Todos.vue 页面组件

GET /api/todos

POST /api/todos

PUT /api/todos/:id

DELETE /api/todos/:id

内存数据库
```

### 后端 API 实现

为了简化，我们直接用一个服务端的数组来模拟数据库。

**1. `server/api/todos/index.get.ts` (获取列表)**

```
// server/api/todos/index.get.ts  
import { todos } from './db' // 假设 db.ts 导出了 todos 数组  
  
export default defineEventHandler(() => {  
  return todos  
})
```

**2. `server/api/todos/index.post.ts` (新增)**

```
// server/api/todos/index.post.ts  
import { todos } from'./db'  
  
let idCounter = todos.length  
  
exportdefaultdefineEventHandler(async (event) => {  
const body = awaitreadBody(event)  
const newTodo = {  
    id: ++idCounter,  
    text: body.text,  
    completed: false  
  }  
  todos.push(newTodo)  
return newTodo  
})
```

**3. `server/api/todos/[id].put.ts` (更新)**

```
// server/api/todos/[id].put.ts  
import { todos } from'./db'  
  
exportdefaultdefineEventHandler(async (event) => {  
const id = Number(event.context.params.id)  
const body = awaitreadBody(event)  
const todo = todos.find(t => t.id === id)  
  
if (!todo) throwcreateError({ statusCode: 404, message: 'Todo not found' })  
  
  todo.completed = body.completed  
return todo  
})
```

**4. `server/api/todos/[id].delete.ts` (删除)**

```
// server/api/todos/[id].delete.ts  
import { todos, setTodos } from'./db'  
  
exportdefaultdefineEventHandler((event) => {  
const id = Number(event.context.params.id)  
const index = todos.findIndex(t => t.id === id)  
  
if (index === -1) throwcreateError({ statusCode: 404, message: 'Todo not found' })  
  
  todos.splice(index, 1)  
return { success: true }  
})
```

**5. `server/api/todos/db.ts` (模拟数据库)**

```
// server/api/todos/db.ts  
let todos = [  
  { id: 1, text: '学习 Nitro', completed: true },  
  { id: 2, text: '构建全栈应用', completed: false },  
]  
  
export { todos }
```

### 前端页面实现

创建 `pages/todos.vue`：

```
<template>  
  <div class="container">  
    <h1>待办事项</h1>  
    <div class="input-group">  
      <input v-model="newTodoText" @keyup.enter="addTodo" placeholder="添加新任务...">  
      <button @click="addTodo">添加</button>  
    </div>  
    <ul>  
      <li v-for="todo in todos" :key="todo.id" :class="{ completed: todo.completed }">  
        <span @click="toggleTodo(todo)">{{ todo.text }}</span>  
        <button class="delete-btn" @click="deleteTodo(todo.id)">×</button>  
      </li>  
    </ul>  
    <p v-if="pending">加载中...</p>  
    <p v-if="error" class="error">{{ error.message }}</p>  
  </div>  
</template>  
  
<script setup>  
import { ref } from 'vue'  
  
const newTodoText = ref('')  
  
// 获取待办事项列表  
const { data: todos, pending, error, refresh } = useFetch('/api/todos')  
  
// 添加新任务  
async function addTodo() {  
  if (!newTodoText.value.trim()) return  
  await $fetch('/api/todos', {  
    method: 'POST',  
    body: { text: newTodoText.value }  
  })  
  newTodoText.value = ''  
  await refresh() // 重新获取列表  
}  
  
// 切换任务状态  
async function toggleTodo(todo) {  
  await $fetch(`/api/todos/${todo.id}`, {  
    method: 'PUT',  
    body: { completed: !todo.completed }  
  })  
  await refresh()  
}  
  
// 删除任务  
async function deleteTodo(id) {  
  await $fetch(`/api/todos/${id}`, {  
    method: 'DELETE'  
  })  
  await refresh()  
}  
</script>  
  
<style scoped>  
.container { max-width: 600px; margin: 2rem auto; font-family: sans-serif; }  
.input-group { display: flex; margin-bottom: 1rem; }  
input { flex-grow: 1; padding: 0.5rem; border: 1px solid #ccc; }  
button { padding: 0.5rem 1rem; border: none; background-color: #00dc82; color: white; cursor: pointer; }  
ul { list-style: none; padding: 0; }  
li { display: flex; align-items: center; justify-content: space-between; padding: 0.5rem; border-bottom: 1px solid #eee; }  
li.completed span { text-decoration: line-through; color: #aaa; }  
li span { cursor: pointer; }  
.delete-btn { background: #ff6b6b; color: white; border: none; border-radius: 50%; width: 24px; height: 24px; cursor: pointer; }  
.error { color: red; }  
</style>
```

在这个前端组件中，我们使用了 `useFetch` 来获取初始数据，并利用其返回的 `refresh` 函数在增、删、改操作后高效地更新列表。对于非 GET 请求，我们使用了 Nuxt 提供的另一个便捷工具 `$fetch`，它的 API 与原生 `fetch` 类似，但同样享受同构调用等优化。

现在，访问 `http://localhost:3000/todos`，一个功能完备的全栈应用就运行起来了！

## 7. 总结与展望

通过本文的学习，我们揭开了 Nuxt 全栈能力的神秘面纱。你不再需要将自己局限于前端开发者的角色。借助 Nitro，你的 Nuxt 项目生来就具备了强大的后端能力。

我们回顾一下核心要点：

* • **`server/api` 目录是你的后端代码大本营。**
* • **`defineEventHandler` 是创建 API 的基本单元。**
* • **`useFetch` 和 `$fetch` 是连接前后端的桥梁，并内置了同构优化。**
* • **动态路由和中间件让你的后端逻辑更灵活、更健壮。**

这只是一个开始。基于今天所学，你可以继续探索更广阔的世界：

* • **连接真实数据库**：使用 Prisma 或 Drizzle ORM 在 Nitro 中操作 PostgreSQL, MySQL 等数据库。
* • **用户认证**：集成 Lucia Auth 或 Auth.js 等库，实现完整的用户注册和登录流程。
* • **文件上传**：处理 `multipart/form-data`，实现图片或文件上传功能。
* • **部署**：尝试将你的全栈 Nuxt 应用一键部署到 Vercel 或 Netlify。

希望这篇指南能为你打开一扇通往全栈开发的大门。现在，去你的 NuT项目里创建第一个 API 吧！

---

🌟 如果这篇指南对你有帮助，请点赞收藏，让更多人看到！

---

**P.S. 肝完这篇文章，希望你的技术栈又多了一块闪亮的徽章！如果觉得内容还不错，想不定期收到这类‘有点用’的硬核推送，不妨来我的公众号「文艺理科生Owen」订阅一下。别担心，这里没有风花雪月，只有代码、bug 和偶尔的灵光一闪。**