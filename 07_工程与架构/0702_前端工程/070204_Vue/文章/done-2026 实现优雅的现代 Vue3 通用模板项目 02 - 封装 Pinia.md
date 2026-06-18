> 已吸收至：[[07_工程与架构/0702_前端工程/070204_Vue/070204_核心知识点/Vue项目工程化边界准则|Vue项目工程化边界准则]]、[[07_工程与架构/0702_前端工程/070204_Vue/070204_知识地图|070204_Vue知识地图]]

---
title: 2026 实现优雅的现代 Vue3 通用模板项目 02 - 封装 Pinia
author: 程序员优雅哥
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzNzMxODg4Mg==&mid=2247484993&idx=1&sn=6fbcab6e4f5ee1b2276d7ae0ccbb9b2a&chksm=c3fbbf423a0bed39080f68de529ba9e1056aee1b2674d66c5a99d53c2d1c5e45b4dd37016398&mpshare=1&scene=24&srcid=0113hE9SFKIH0hjTe1qgyPhw&sharer_shareinfo=af5ac5859c8c71855742ee68490197b9&sharer_shareinfo_first=af5ac5859c8c71855742ee68490197b9#rd
---

上一篇文章中，咱们对 create-vue 脚手架创建的项目进行了清理和重构，为后续的功能集成做好了准备。

虽然在项目创建时选择了集成 Pinia 作为状态管理方案，但在清理过程中删除了相关代码。

本文将以优雅的方式重新封装 Pinia，为模板项目添加强大的状态管理能力。

## 3 封装 Pinia

作为 Vue3 的官方状态管理库，与 Vuex 相比，Pinia 具有：

* 更简洁的 API
* 更好的 TypeScript 支持
* 更灵活的组合式 API 风格

封装 Pinia 只需两步：

1. 创建 Pinia 实例，并配置持久化插件
2. 将 Pinia 实例注册到 Vue 应用中

### 3.1 安装依赖

项目创建时已默认安装了 Pinia 核心依赖，我们只需添加状态持久化插件：

```
pnpm add pinia-plugin-persistedstate
```

使用该插件会自动将 Pinia 中的状态持久化到本地存储，避免页面刷新后状态丢失的问题。

### 3.2 封装 Pinia

在 `src/stores`目录下创建 `index.ts`文件，实现 Pinia 的封装和模块化注册：

```
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'
import type { App } from 'vue'

// 创建 Pinia 实例
const pinia = createPinia()

// 添加持久化插件
// 该插件会自动将状态保存到 localStorage，默认 key 为 store 名称
pinia.use(piniaPluginPersistedstate)

/**
 * 安装 Pinia
 * @paramapp Vue 应用实例
 * 采用模块化方法注册 Pinia
 */
export const installStore = (app: App) => {
  app.use(pinia)
}

export default pinia
```

在 `main.ts`中调用 `installStore`方法注册 Pinia：

```
import { createApp } from 'vue'
import App from './App.vue'
import { installRouter } from '@/router'
import { installStore } from '@/stores'

const app = createApp(App)
installRouter(app)
installStore(app)
app.mount('#app')
```

### 3.3 测试 Pinia

为了验证 Pinia 及持久化插件的配置效果，我们创建一个测试 store：

1. 在 `stores`目录下创建 `modules`子目录，用于组织不同业务模块的 store
2. 在 `modules`中创建 `demo.ts`文件：

```
import { defineStore } from 'pinia'
import { ref } from 'vue'

/**
 * 测试用的 Pinia Store
 * 使用组合式 API 风格编写，与 Vue3 的组合式 API 保持一致
 */
export const useDemoStore = defineStore(
'demoStore',// Store 唯一标识
()=>{
  // 状态定义
  const counter = ref<number>(0)

  // Action 定义（修改状态的方法）
  const increment = () => {
    counter.value++
  }

  // 返回需要暴露的状态和方法
  return{
    counter,
    increment,
  }
},
{
  persist:true,// 开启状态持久化
},
)
```

**代码说明**：

* 咱使用组合式 API 风格编写 Store，与 Vue3 的组合式 API  setup 保持一致的风格
* 通过 `persist: true`开启状态持久化，默认会将状态保存到 localStorage 中，可自定义状态保存的方式。
* 模块化组织 Store 文件 main.ts 的代码显得条理清晰

> 个人建议：既然在 Vue 文件中已经使用了组合式 API，在 Store 中也应采用相同风格，保持代码风格的一致。
>
> 千万别死抱着传统 vuex 的选项式写法不肯放手

在 `demo.vue`中添加测试代码，验证 Pinia 功能：

```
<template>
  <div class="demo">
  <h2>Pinia 测试</h2>
  <h3>计数器: {{ counter }}</h3>
  <button @click="demoStore.increment()"> 点击+1 </button>
  </div>
</template>

<script setup lang="ts">
import { useDemoStore } from '@/stores/modules/demo'
import { storeToRefs } from 'pinia'

// 初始化 Store 实例
const demoStore = useDemoStore()

// 使用 storeToRefs 解构状态，保持响应性
const { counter } = storeToRefs(demoStore)
</script>
```

**测试步骤**：

1. 启动项目并在浏览题访问 demo 页面
2. 界面显示的计数默认为 0
3. 点击按钮，计数器值会+1
4. 刷新页面，计数器值保持不变，刷新前是多少，刷新后依旧是多少（测试持久化功能）
5. 将 `persist`设置为 `false`，再次刷新页面，计数器会重置为 0

Pinia 及持久化插件已正确配置，可以正常管理和持久化应用状态。

关于 pinia 的更多使用方式，与三年前相比没啥区别，大家可以阅读官网或在优雅哥的历史文章中搜索。

## 总结

通过本文的封装，咱为模板项目集成了官方的状态管理库：

* 采用模块化方式封装 Pinia
* 集成持久化插件，解决页面刷新后状态丢失的问题
* 使用组合式 API 风格编写 Store，与 Vue3 的组合式 API 风格保持一致
* 使用模块化形式来组织 Store 文件

本文内容比较少，下一篇文章就上强度了：进入样式管理和图标集成的部分，为模板项目添加 UnoCSS 原子化样式和 SVG 图标支持，进一步实现模板项目的基础能力。

如果你对本文内容有任何疑问或建议，欢迎在评论区留言讨论。