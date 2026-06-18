> 已吸收至：[[07_工程与架构/0702_前端工程/070204_Vue/070204_核心知识点/Vue项目工程化边界准则|Vue项目工程化边界准则]]、[[07_工程与架构/0702_前端工程/070204_Vue/070204_知识地图|070204_Vue知识地图]]

---
title: Vue3 应该这样使用 SCSS、UnoCSS - 2026 实现优雅的现代 Vue3 通用模板系列之3
author: 程序员优雅哥
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzNzMxODg4Mg==&mid=2247485013&idx=1&sn=2214310f85314e8021da4ce9affd36e9&chksm=c31302f955432e70e93e593aa23938bca350a43043ce546c4660c44d81a737362a9e270b3f9d&mpshare=1&scene=24&srcid=0115nsMnKqzPVkbx8vOX7Uy8&sharer_shareinfo=37142aa165165f9084b08d80cc43f674&sharer_shareinfo_first=37142aa165165f9084b08d80cc43f674#rd
---

通过前面两篇文章，通用模板项目已经完成了：

* 创建项目
* 清理目录文件
* 集成 Pinia 状态管理及状态持久化能力

本文将聚焦于样式管理，为模板项目的视觉效果打下基础。

## 4 优雅的使用 CSS 预处理器 - SCSS

CSS 预处理器可以显著提升样式开发效率，支持变量、循环、嵌套、混合等高级特性，这是使用原生 CSS 无法实现的。

CSS 预处理器有很多，语法和功能都大同小异，大家可以根据自己的喜好和所在企业的习惯自行选择即可，优雅哥这里依旧使用 SCSS。

Vite 内置了对 `scss`、`sass`、`less`、`styl`和 `stylus`这些预处理器的支持，无需安装额外的 Vite 插件，只需安装相应的预处理器依赖即可。

### 4.1 安装依赖

SCSS 的依赖库为 Sass，安装为开发依赖：

```
pnpm add sass -D
```

### 4.2 样式文件组织

CSS 架构是开发过程中常被忽略的环节：Vue、TS 代码可能写得很优雅，但到 CSS 便草草了事。

优雅哥在三年多前分享的组件库脚手架系列文章中，已经介绍过常见的 CSS 架构模式，如 OOCSS、ACSS、BEM、SMACSS、ITCSS 等，这里不再赘述。

为了保持样式管理的清晰性和便捷性，咱们这里就只划分为三层：

**settings**

样式变量的定义，如颜色值、尺寸、字体大小等基础变量。

在 UI 设计中有一个很厉害的专业术语：设计令牌，即 Design Tokens。 这一层，也可以叫定义设计令牌。

**base**

全局通用的定制化样式，如：

* 覆盖 Element Plus、VXE Table 的默认样式
* 设置 html、body 的样式

**components**:

自定义的组件的样式，每个组件自行维护，不需要抽取到公共目录中。

这一层采用 `ACSS`模式，具体实现使用 UnoCSS。

在 `src`中按照如下结构建立目录和文件：

```
src/
  |- assets/
      |- scss/
         |- base/
            |- index.scss
         |- settings/
            |- _color.scss
            |- _size.scss
            |- index.scss
         |- index.scss
```

### 4.3 实现 settings 层

依次实现 settings 层的三个文件：

1. `scss/settings/_color.scss`

存放 SCSS 和 CSS 颜色变量。

首先定义一系列 SCSS 变量，包括：品牌与状态色、文字、边框、背景颜色； 然后利用 SCSS 的可编程的特点生成不同色阶的 CSS 颜色变量；最后定义一套暗黑主题的变量值。

```
@use "sass:color";

// 品牌与状态色
$wm-color-primary:#1C9399;
$wm-color-success:#3DBE7D;
$wm-color-warning:#FFB74D;
$wm-color-danger:#F1524C;
$wm-color-info:#5E7CE0;

// 中性色 - 文字
$wm-color-text-primary:#303133;
$wm-color-text-regular:#606266;
$wm-color-text-secondary:#909399;
$wm-color-text-placeholder:#A8ABB2;
$wm-color-text-disabled:#C0C4CC;

// 中性色 - 边框
$wm-border-color-base:#DCDFE6;
$wm-border-color-light:#E4E7ED;

// 中性色 - 背景
$wm-bg-color-page:#F5F7FA;
$wm-bg-color-base:#FFFFFF;

// 默认主题 白天模式
:root {
// 品牌/状态色及其色阶
@each $type,$color in (primary,$wm-color-primary),
(success,$wm-color-success),
(danger,$wm-color-danger),
(warning,$wm-color-warning),
(info,$wm-color-info){

--wm-color-#{$type}:#{$color};

// 生成浅色变体 (混合白色)
@each $i in (1,2,3,5,7,8,9){
--wm-color-#{$type}-light-#{$i}: #{color.mix(#fff,$color,$i*10%)};
}

// 生成深色变体 (混合黑色)
@each $i in (1,2){
--wm-color-#{$type}-dark-#{$i}: #{color.mix(#000,$color,$i*10%)};
}
}

// 文字
--wm-color-text-primary:#{$wm-color-text-primary};
--wm-color-text-regular:#{$wm-color-text-regular};
--wm-color-text-secondary:#{$wm-color-text-secondary};
--wm-color-text-placeholder:#{$wm-color-text-placeholder};

// 边框
--wm-border-color-base:#{$wm-border-color-base};
--wm-border-color-light:#{$wm-border-color-light};

// 背景
--wm-bg-color-base:#{$wm-bg-color-base};
--wm-bg-color-page:#{$wm-bg-color-page};
}

// 深色主题覆盖
[data-theme="dark"] {
--wm-color-text-primary:rgba(255,255,255,0.95);
--wm-color-text-regular:rgba(255,255,255,0.75);
--wm-color-text-secondary:rgba(255,255,255,0.55);
--wm-color-text-placeholder:rgba(255,255,255,0.35);

--wm-bg-color-page:#121212;
--wm-bg-color-base:#1a1a1a;
--wm-border-color-base:rgba(255,255,255,0.12);
}
```

上面这段代码是不是比较优雅？

* 使用 `@use "sass:color"`导入 sass 的颜色模块
* 通过 `@each`循环批量生成 CSS 变量，减少重复代码
* 使用 `color.mix()`函数生成不同亮度的颜色色阶

> **注意**：Sass 升级后 API 有变化：
>
> 1. 不再推荐使用 `@import`，而是使用 `@use`导入模块
> 2. 颜色混合函数从 `mix()`变为 `color.mix()`
>
> 启动服务时的控制台警告信息应认真处理，它们通常提示潜在问题或过时用法。千万不要看见警告信息，觉得不是报错，就不处理！

2. `scss/settings/_size.scss`

存放 SCSS 和 CSS 尺寸相关变量。

```
// 间距系统 (以8px为基准)
$wm-spacing-base:8px;

:root {
--wm-spacing-xs: #{$wm-spacing-base*0.5};// 4px
--wm-spacing-sm: #{$wm-spacing-base*1};// 8px
--wm-spacing-md: #{$wm-spacing-base*1.5};// 12px
--wm-spacing-lg: #{$wm-spacing-base*2};// 16px
--wm-spacing-xl: #{$wm-spacing-base*3};// 24px
--wm-spacing-2xl: #{$wm-spacing-base*4};// 32px
}

// ===== 字体系统 =====
:root {
--wm-font-size-xs:12px;
--wm-font-size-sm:14px;
--wm-font-size-base:16px;
--wm-font-size-lg:18px;
--wm-font-size-xl:20px;
--wm-font-size-2xl:24px;
--wm-font-size-3xl:30px;

--wm-font-weight-normal:400;
--wm-font-weight-medium:500;
--wm-font-weight-semibold:600;

--wm-line-height-tight:1.25;
--wm-line-height-normal:1.5;
}

// ===== 圆角 =====
:root {
--wm-border-radius-sm:4px;
--wm-border-radius-base:6px;
--wm-border-radius-lg:8px;
--wm-border-radius-full:9999px;
}
```

3. `scss/settings/index.scss`

导出前两个文件，作为 settings 的入口：

```
@use"./color";
@use"./size";
```

### 4.4 实现 base 层

由于目前咱们没有引入第三方组件库，现在无需设置第三方组件库样式。

故 base 层目录只有一个入口文件：

`scss/base/index.scss`设置 body 样式

```
body {
margin:0;
background-color:var(--wm-bg-color-page);
color:var(--wm-color-text-primary);
font-family: -apple-system, BlinkMacSystemFont,'Segoe UI', Roboto,'Helvetica Neue', Arial, sans-serif;
}
```

若后续需要覆盖如 Element Plus 的全局样式时，在这个目录添加文件，并在 index.scss 中导入即可。

### 4.5 统一样式入口

`assets/scss/index.scss`:

全局样式入口文件，分别导入 settings 和 base：

```
// 导入变量文件
@use"./settings";
@use"./base";
```

### 4.6 统一资源管理

为了保持 `main.ts`的简洁，创建一个专门的资源管理模块，统一处理所有静态资源的导入：

1. 在 `src`目录下创建 `plugins`目录
2. 在该目录中创建 `assets.ts`文件：

`src/plugins/assets.ts`:

```
import'@/assets/scss/index.scss'

/**
 * 空方法，通过调用该方法，导入上面 import 中的样式和资源
 */
export const installAssets = () => {}
```

在 `main.ts`中调用：

```
// ...
import { installAssets } from '@/plugins/assets'

// ...

installAssets()

// ...
```

这时候再看看浏览器调试面板，是否出现了前面定义的 CSS 变量？

CSS 变量

## 5 集成 UnoCSS

UnoCSS 是一个高性能、可定制的原子化 CSS 框架，具体的介绍之前也分享过对应文章，这里快速进行集成。

### 5.1 安装依赖

安装 UnoCSS 及其相关依赖：

```
pnpm add unocss -D
```

### 5.2 配置 Vite 插件

在 `vite.config.ts`文件中添加 UnoCSS 插件：

```
// ...
import UnoCSS from 'unocss/vite'// 导入 UnoCSS 插件


export default defineConfig({
   plugins:[
// ...
UnoCSS()
],
// ...
})
```

### 5.3 配置 UnoCSS

在根目录下创建 `uno.config.ts`文件，配置 UnoCSS，主要配置如下内容： 1、预设配置：使用 Tailwind 预设 2、转换器配置：支持 @apply、分组 3、主题配置：前面定义的 CSS 变量映射到 UnoCSS 4、规则：为 primary, success 等颜色生成工具类 5、快捷类：定义了两个样式快捷类名

```
import { defineConfig, presetWind4, transformerDirectives, transformerVariantGroup } from 'unocss'

exportdefaultdefineConfig({
    presets:[
presetWind4({
            preflights:{ reset:true},
}),
],

    transformers:[
// 启用 @apply 指令支持
transformerDirectives(),
// 启用 `hover:(bg-gray-400 text-white)` 类写法
transformerVariantGroup(),
],

    theme:{
        colors:{
// 品牌与状态色
            primary:'var(--wm-color-primary)',
            success:'var(--wm-color-success)',
            warning:'var(--wm-color-warning)',
            danger:'var(--wm-color-danger)',
            info:'var(--wm-color-info)',
// 文字色
'text-primary':'var(--wm-color-text-primary)',
'text-regular':'var(--wm-color-text-regular)',
'text-secondary':'var(--wm-color-text-secondary)',
// 背景色
'bg-page':'var(--wm-bg-color-page)',
'bg-base':'var(--wm-bg-color-base)',
},
        spacing:{
            xs:'var(--wm-spacing-xs)',
            sm:'var(--wm-spacing-sm)',
            md:'var(--wm-spacing-md)',
            lg:'var(--wm-spacing-lg)',
            xl:'var(--wm-spacing-xl)',
'2xl':'var(--wm-spacing-2xl)',
},
        fontSize:{
            xs:'var(--wm-font-size-xs)',
            sm:'var(--wm-font-size-sm)',
            base:'var(--wm-font-size-base)',
            lg:'var(--wm-font-size-lg)',
            xl:'var(--wm-font-size-xl)',
'2xl':'var(--wm-font-size-2xl)',
'3xl':'var(--wm-font-size-3xl)',
},
        borderRadius:{
            sm:'var(--wm-border-radius-sm)',
            base:'var(--wm-border-radius-base)',
            lg:'var(--wm-border-radius-lg)',
            full:'var(--wm-border-radius-full)',
},
},

    rules:[
...['primary','success','warning','danger','info'].flatMap((color)=>[
[`bg-${color}-light-3`,{'background-color':`var(--wm-color-${color}-light-3)`}],
[`bg-${color}-light-5`,{'background-color':`var(--wm-color-${color}-light-5)`}],
[`bg-${color}-light-7`,{'background-color':`var(--wm-color-${color}-light-7)`}],
[`bg-${color}-light-9`,{'background-color':`var(--wm-color-${color}-light-9)`}],
[`text-${color}-dark-2`,{ color:`var(--wm-color-${color}-dark-2)`}],
]),
],

    shortcuts:{
// 布局
'flex-center':'flex justify-center items-center',
'flex-col-center':'flex flex-col justify-center items-center',
},
})
```

在以前的版本中，还需要手动添加样式重置（安装 @unocss/reset, 并引入需要的样式重置文件）

但咱使用了 Tailwind 4 作为预设，可以通过配置 `preflights: { reset: true }`来开启样式重置，故这里无需安装其他依赖了。

**引入 UnoCSS**： 在 `src/plugins/assets.ts`文件中添加 UnoCSS 的引入：

```
<template>
<div>
  <divclass="demo">Hello world</div>
  <divclass="text-xs text-primary">测试文字</div>
  <divclass="text-sm text-success">测试文字</div>
  <divclass="text-md text-info">测试文字</div>
  <divclass="text-base text-warning">测试文字</div>
  <divclass="text-lg text-danger">测试文字</div>
  <divclass="text-xl text-text-primary">测试文字</div>
  <divclass="text-2xl text-text-regular">测试文字</div>
  <divclass="text-3xl text-text-secondary">测试文字</div>

  <buttonclass="mt-xl px-md py-sm border rounded-base"@click="toggleTheme">
      切换深色/浅色模式
  </button>
</div>
</template>

<script setup lang="ts">
const toggleTheme= () => {
const html = document.documentElement
const currentTheme = html.getAttribute('data-theme')
  html.setAttribute('data-theme', currentTheme ==='dark'?'light':'dark')
}
</script>
<style scoped lang="scss">
.demo{
@apply text-xl text-primary bg-success-light-7;
}
</style>
```

启动服务，测试页面效果及切换主题的效果：白天暗黑

## 结语

通过本文的配置，我们为模板项目添加了完整的样式管理系统：

* **SCSS 预处理器**

  实现了模块化的样式组织，通过变量和循环生成丰富的 CSS 颜色系统和尺寸系统，支持深色模式
* **UnoCSS 原子化 CSS**

  集成了高性能的原子化 CSS 框架，提供丰富的实用类，结合自定义主题和快捷类，显著提升开发效率
* **统一资源管理**

  创建了专门的资源管理模块，保持 main.ts 的简洁性

这些配置为模板项目的视觉设计和开发效率奠定了坚实基础。通过 SCSS 变量和 UnoCSS 的结合使用，我们构建了一个灵活、可维护的样式系统，支持快速开发和主题定制。

在下一篇文章中，我们将介绍图标相关的封装。

本文的相关 scss 代码看着比较混乱，如果您需要 SCSS 源码，可在留言区留下您的邮箱。如果你对本文内容有任何疑问或建议，欢迎在评论区留言讨论。我们下期再见！