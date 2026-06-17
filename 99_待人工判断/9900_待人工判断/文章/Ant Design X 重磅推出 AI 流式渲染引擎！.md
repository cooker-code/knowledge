---
title: Ant Design X 重磅推出 AI 流式渲染引擎！
author: 前端开发爱好者
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUzNTk3MjE2Ng==&mid=2247520788&idx=1&sn=89dc969904af80ae74781d63877dd24f&chksm=fb9419aa7677ee8c13e5f946f52d8271babc61991240f1f00c1758edbc48057a70aea5a4650b&mpshare=1&scene=24&srcid=1216lVrd96yYTadTFquLcvZp&sharer_shareinfo=9439aabe56be2664215222a3bbf34d5d&sharer_shareinfo_first=9439aabe56be2664215222a3bbf34d5d#rd
---

过去两年，**大模型**以周为单位迭代，**AI** 应用以**日**为单位**上线**。

`对话`、`编程`、`文档`、`报表`、`代码 review`……所有场景都在做同一件事——**让机器“边想边说”**。

传统 **Markdown** 渲染器“等全文、再渲染”的模式，在逐字蹦出的 **Token** 面前彻底失效；

**流式渲染**成了新一代 **AI** 产品的“刚需”。

## Ant Design X 给出答案：XMarkdown

**Ant Design** 团队把为「`通义千问`、`钉钉 AI`、`阿里云智能客服`」打磨多年的内核抽离出来，做成**零依赖、开箱即用**的 **React** 组件：

### 优势一览

* ⚡ **真·流式**  
  基于`分片` + `缓存补丁`，新 **Token** 只更新“最后一行”，`60 fps` 不闪屏
* 🔌 **插件生态**  
  `KaTeX` 公式、`Mermaid` 图表、`highlight` 代码块全部内置，一行引入
* 🎨 **极致可定制**  
  任意 `Markdown` 元素可替换成自定义 `React` 组件；主题变量一键换肤
* 🔐 **默认安全**  
  无 `dangerouslySetInnerHTML`，XSS 攻击面直接清零
* 📦 **轻量**  
  压缩后 `< 40 kB`，`CommonMark` + `GFM 100%` 兼容

### 2 分钟跑起来

```
npm i @ant-design/x-markdown
```

```
import { XMarkdown } from'@ant-design/x-markdown'  
  
function App() {  
const [chunk, setChunk] = useState('')  
  
// 逐字接收 AI 输出  
  useEffect(() => {  
    fetchEventSource('/api/ai-stream', {  
      onmessage(ev) { setChunk(c => c + ev.data) }  
    })  
  }, [])  
  
return<XMarkdown content={chunk} plugins={[Latex, Mermaid, Highlight]} />  
}
```

✅ 支持 `Next.js`、`Vite`、`Umi`、`Rspack` 等任意 **React** 技术栈。

## Vue 阵营怎么办？Element Plus X 同步跟进！

**Ant Design X** 目前仅支持 **React**，但**Vue 用户无需眼红**。

**Element Plus X** 推出同名组件 **XMarkdown**，**API** 与生态完全对齐 **Vue** 习惯：

### 优势再升级

* **高亮引擎**：`Shiki`（140+ 主题，可动态切换）
* **代码块**：`折叠`、`复制`、`全屏`、`放大`、`下载`、`主题换肤`**全套 Toolbar**
* **Mermaid**：`渐进式渲染` + `工具栏`（缩放、重置、下载）
* **公式**：`KaTeX` 行内 / 块级，增量排版
* **自定义标签**：可在 `Markdown` 里直接写 `<my-chart :data="xxx"/>` 并渲染成 Vue 组件
* **安全**：`HTML` 默认不解析，预览可选 **安全模式**

### 2 分钟跑起来

```
npm i element-plus-x
```

```
<template>  
  <x-markdown :markdown="stream" :themes="{ light: 'github-light', dark: 'dracula' }" />  
</template>  
  
<script setup>  
import { ref } from 'vue'  
import { XMarkdown } from 'element-plus-x'  
  
const stream = ref('')  
const evtSource = new EventSource('/api/ai-stream')  
evtSource.onmessage = e => stream.value += e.data  
</script>
```

✅ 兼容 `Nuxt`、`Vite`、`Webpack`、`Rspack`、纯 `CDN` 引入。

## 还有惊喜：markstream-vue —— 更轻量的第三选择

如果你只想“**最小包 + 最纯流式**”，尤雨溪亲自推荐的 **markstream-vue** 值得一看：

* 无 **UI** 框架依赖，**仅 807 kB**
* **Monaco Editor** 同款增量算法，**大代码块 60 fps**
* `Mermaid` / `KaTeX` 均做 **Token 级切片**，语法一够就渲染
* 支持在 `Markdown` 里直接嵌入 **任意 Vue 组件**
* 零配置，`<MarkdownRenderer :stream="chunk" />` 即用

### 1 分钟跑起来

```
npm i markstream-vue
```

```
<template>  
  <MarkdownRenderer :stream="chunk" />  
</template>  
  
<script setup>  
import { ref } from 'vue'  
import MarkdownRenderer from 'markstream-vue'  
  
const chunk = ref('')  
fetchEventSource('/api/ai-stream', {  
  onmessage(e) { chunk.value += e.data }  
})  
</script>
```

## 如何选型？一句话记住

* **React 项目** → 直接用 **Ant Design X XMarkdown**，插件全、主题省心
* **Vue 项目** → **Element Plus X XMarkdown**，Shiki 高亮、工具栏一条龙
* **Vue 但想极致轻量** → **markstream-vue**，**807 kB** 无依赖，纯流式更敏捷

**相关链接：**

* **Ant Design X XMarkdown**：`https://x.ant.design/x-markdowns/introduce-cn`
* **Element Plus X XMarkdown**：`https://element-plus-x.com/zh/components/xmarkdown/`
* **markstream-vue**：`https://markstream-vue.simonhe.me/`