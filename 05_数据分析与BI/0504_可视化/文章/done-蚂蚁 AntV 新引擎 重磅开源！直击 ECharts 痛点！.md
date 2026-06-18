> 已吸收至：[[05_数据分析与BI/0504_可视化/0504_核心知识点/AntV与图表工程生态|AntV与图表工程生态]]
---
title: 蚂蚁 AntV 新引擎 重磅开源！直击 ECharts 痛点！
author: 前端开发爱好者
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzNTk3MjE2Ng==&mid=2247521484&idx=1&sn=8306ae2d5a5e1cb0139c130f5e2ba742&chksm=fb67389b631c23a5a4cec4e85faaeaed79891f7ff1d438c4a64fa3d77f357a835be86529281a&mpshare=1&scene=24&srcid=0107wWi5oVT2x3DmNvu1cfy4&sharer_shareinfo=ba35d0cd29cfd7a624956914241c7492&sharer_shareinfo_first=ba35d0cd29cfd7a624956914241c7492#rd
---

作为前端开发，**ECharts 用过的人不在少数**。

但是当你想画一些`信息图`、`组织架构图`、`流程图`或者`四象限分析图`的时候

**ECharts** 往往就显得有点吃力：`配置复杂`、`布局调不准`、`风格不统一`。

这时候，**AntV** 推出的 **Infographic** 就派上用场了。

## 什么是 AntV Infographic？

简单说，它是 **AntV 的新一代声明式信息图可视化引擎** 。

你只需要用一套清晰的语法描述你的信息结构，它就能把内容快速渲染成图，**不管是流程、层级还是对比分析**都可以处理得井井有条。

更重要的是，它天然支持 AI 生成和实时渲染，**内容可以边生成边看到效果**，大大降低了创作门槛。

## 核心亮点

* 🤖 **AI 友好**：语法和配置专门优化过，`AI` 能轻松理解信息结构，支持流式输出和渲染。
* 📦 **开箱即用**：内置近 `200` 个模板和可复用组件，几分钟就能搭出专业信息图。
* 🎨 **灵活主题**：`手绘`、`渐变`、`纹理`多种风格可选，也支持深度自定义。
* 🧑🏻‍💻 **内置编辑器**：生成的图可以直接在`编辑器`里`调整布局`、`文字和样式`，所见即所得。
* 📐 **高质量 SVG**：默认用 `SVG` 渲染，画质清晰，也方便二次编辑或嵌入系统。

## 快速上手（Vue / React 均可）

安装非常简单：

```
npm install @antv/infographic
# 或
pnpm add @antv/infographic
```

### JS 示例

```
import { Infographic } from '@antv/infographic';

const infographic = new Infographic({
  container: '#container',
  width: '100%',
  height: '100%',
  editable: true,
});

infographic.render(`
infographic list-row-simple-horizontal-arrow
data
  items
    - label Step 1
      desc Start
    - label Step 2
      desc In Progress
    - label Step 3
      desc Complete
`);
```

### React / Vue 示例

```
// React
<Infographic
  type="hierarchy"
  data={{
    title: '技术架构',
    items: [{ text: '前端' }, { text: '后端' }, { text: '数据层' }],
  }}
/>
```

```
<!-- Vue -->
<Infographic
  type="sequence"
  :data="{ items: ['需求分析', '设计', '开发', '上线'] }"
/>
```

**核心思想**：**你不是在调图表参数，而是在描述“内容结构”**。

## 流式渲染，AI 也能边生成边看

**Infographic** 支持 **边接收 AI 输出边渲染**，非常适合生成式 `AI 工作流`：

```
let buffer = '';
for (const chunk of chunks) {
  buffer += chunk;
  infographic.render(buffer);
}
```

## AI 整合与技能支持

**AntV** 为 **Infographic** 提供了丰富的 **AI** 集成能力，让自动化生成和人机协作更顺畅：

* **信息图生成器**：快速生成渲染用 `HTML` 文件
* **语法生成器**：根据描述生成`信息图语法`
* **结构创建器**：自定义`信息图布局`
* **项目创建器**：生成自定义`数据项`样式
* **模板更新器**：方便开发者维护和扩展模板库

## 模板丰富，直接套用

官方模板库覆盖了大部分使用场景：

👉 **官方示例**：`https://infographic.antv.vision/gallery`

**图表型**：适合展示简单数据

**对比型**：方案或指标对比

**层级型**：组织架构、系统分层

**列表型**：多信息点的结构化展示

**四象限型**：经典分析模型

**关系型**：模块依赖或关联说明

**顺序型**：流程、步骤或时间线

模板可直接使用，也可以微调样式，非常省时。

## 在线编辑器

如果不想写代码，也可以直接用**在线编辑器**：

👉 **在线编辑器**：`https://infographic.antv.vision/editor`

* `拖拽`、`修改`都能实时预览
* 非前端也能`快速上手`
* 支持 `AI` 自动生成和`人工编辑`结合

## AI Infographic

你只要把文字内容粘贴进去，`AI` 会自动理解语境，帮你生成匹配的信息图。

特别适合`写文章`、`做汇报`或`整理方案`，让创作效率直接提升。

## 项目地址

* **GitHub 地址**：`https://github.com/antvis/Infographic`
* **官网**：`https://infographic.antv.vision/`