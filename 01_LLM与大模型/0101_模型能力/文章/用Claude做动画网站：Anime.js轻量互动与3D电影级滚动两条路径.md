---
title: 用Claude做动画网站：Anime.js轻量互动与3D电影级滚动两条路径
author: 极客BIM设计工坊
date: 朗朗晴空朗朗晴空
url: https://mp.weixin.qq.com/s?__biz=MzI2MjA3ODk0OQ==&mid=2648117208&idx=1&sn=70c8cb261b044f1b2e6b63642b08d2a2&chksm=f3f978070e2004e3e3c9fc391769c3502bc7556ddcb51ebc4dca2aa9dd8776758e4f73e9698a&mpshare=1&scene=24&srcid=0515MQrAXLQxbztS4WjRNOl8&sharer_shareinfo=cb6bd4c292dc37e0b02016cd73260bd0&sharer_shareinfo_first=cb6bd4c292dc37e0b02016cd73260bd0#rd
---

长话短说

两条路径：Anime.js 做轻量互动站点，Three.js + GSAP 做电影级 3D 滚动体验。前置条件只有 Claude 访问权限 + Node.js，安装一个免费 npm 包就能开始。

2

路径

0

费用

v4

Anime.js

最近两条推文的讨论度很高：有人用 Claude 配合 **Anime.js** 做出了交互流畅的极简站点，也有人把 **Three.js + GSAP + ScrollTrigger** 组合出电影级别的滚动动画。这不是什么神秘黑魔法，核心思路就一条——让 Claude 生成动画代码，你来调参数。

下面一步一步写清楚。

前置条件

开始前确认三样东西：

● **Claude 访问权限** — Claude Pro ($20/月) 即可，Artifacts 功能在 Web 端默认开启

● **Node.js** — 运行 `node -v` 确认版本 ≥ 18

● **代码编辑器** — VS Code 足够

路径 A：用 Anime.js 做极简互动网站

**Anime.js** 是轻量级 JavaScript 动画引擎，npm 当前版本 4.4.1，GitHub 仓库 juliangarnier/anime 超过 54K stars。它支持 CSS 属性、SVG、DOM 属性和 JavaScript 对象的动画，gzip 后不到 10KB，适合做页面过渡、文字交错、悬停效果。

第一步：安装 Anime.js

```
mkdir my-anime-site && cd my-anime-site
npm init -y
npm install animejs
```

不想用 npm 的话，也可以直接用 CDN 引入一个 script 标签，适合单文件快速原型。

第二步：让 Claude 生成动画代码

打开 Claude Web 端 Artifacts，用这个提示词：

**提示词** 请用 Anime.js v4 创建一个极简风格的单页互动网站：1) 文字交错入场动画；2) 悬停按钮时放大+变色；3) SVG 图形跟随鼠标移动；4) 使用 inOutQuint 缓动；5) 保持界面干净，运动是唯一焦点。输出为单个 HTML 文件，用 CDN 引入 animejs。

Claude 会输出一个完整的 HTML 文件。Anime.js v4 的核心 API 是 `animate()`：

```
import { animate, stagger } from 'animejs';

animate('.item', {
  x: 320,
  rotate: { from: -180 },
  duration: 1250,
  delay: stagger(65, { from: 'center' }),
  ease: 'inOutQuint',
  loop: true,
  alternate: true
});
```

▸ `stagger()` 控制元素依次错开

▸ `ease` 选缓动曲线决定手感

▸ `timeline` 组合多个动画按时间线播放

**提示** 动画高级感的秘诀不在代码复杂度，而在缓动函数和时序。`inOutQuint` 和 `spring` 是出片率最高的两条曲线。

路径 B：Three.js + GSAP 做电影级 3D 滚动

这是推文中提到的进阶路径。三个库各司其职：**Three.js** 负责 3D 场景渲染，**GSAP** (GreenSock) 负责高性能动画时序，**ScrollTrigger** 插件让滚动驱动一切。

GSAP 2025 年被 Webflow 收购后免费开放，个人和商业使用均免费。

第一步：安装三个库

```
mkdir my-3d-site && cd my-3d-site
npm init -y
npm install three gsap
```

ScrollTrigger 包含在 GSAP 包中 (`gsap/ScrollTrigger`)，无需额外安装。

第二步：让 Claude 生成 3D 滚动场景

**提示词** 请用 Three.js + GSAP + ScrollTrigger 创建电影级滚动动画页面：1) 分层图像实现 3D 视差深度感；2) 滚动控制相机缩放和场景切换；3) 添加光照和阴影；4) 平滑缓动；5) 单个 HTML 文件，用 importmap 引入依赖。

核心代码结构：

```
import * as THREE from 'three';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

const cameraAnim = { x: 0, y: 0, z: 8 };

gsap.to(cameraAnim, {
  x: -20, y: 15, z: 0,
  scrollTrigger: {
    trigger: '.scene',
    scrub: 1,
    pin: true
  }
});
```

核心逻辑——**GSAP 管动画，Three.js 管渲染，ScrollTrigger 管触发**。Claude 理解这三个库各自的 API 分工，不会混在一起写。

**提示** 电影感关键是 `scrub` 参数——设为 1-3 的数字而非 true 会产生惯性拖尾效果，滚动停止后画面继续移动一秒再停下，这个手感是高端和低端的分水岭。

常见坑与避法

注意三个常见坑

**Anime.js v3 到 v4 破坏性升级** — v4 改用 ES 模块导入，不再使用全局 anime() 函数；Claude 输出旧语法需手动改成 import from

**Three.js 在移动端卡顿** — 初始化时设 renderer.setPixelRatio(Math.min(devicePixelRatio, 2)) 限制倍数

**GSAP 免费版许可** — GSAP Standard 免费，但向用户收费的产品需确认商业授权条款

部署上线

部署用 **Vite** 构建，推到 **Netlify**：

```
$ 安装 Vite
npm install -D vite
$ 开发预览
npx vite
$ 产物在 dist/ 目录
$ 拖到 Netlify Drop 即可部署
```

纯 CDN 方式 (单 HTML 文件) 更简单——拖到 Netlify Drop 即可，零配置。

适合谁 / 不适合谁

▸ **适合** — 做作品集站、产品落地页的人，不需要手写动画代码

▸ **适合** — 前端初学者，通过 Claude 生成的代码学习动画原理

▸ **不适合** — 需要复杂性能优化的大型项目（仍需专业前端工程师）

▸ **不适合** — 移动端优先且对低端机型兼容有严格要求的项目

三条可执行的下一步

● 花 15 分钟按路径 A 跑通 Anime.js 单页 demo，感受 stagger + 缓动效果

ˏ 准备好作品集静态图，按路径 B 让 Claude 生成 Three.js 视差背景

● 收藏 Anime.js v4 文档 (animejs.com/documentation) 和 GSAP 文档，调参数时对照查

SOURCES

Anime.js 官方文档 — animejs.com/documentation

Anime.js npm 包 — npmjs.com/package/animejs

GSAP ScrollTrigger + Three.js (Codrops) — tympanus.net/codrops/2025/11/19/how-to-build-cinematic-3d-scroll-experiences-with-gsap/

Claude + 3D 网站工作流 (MindStudio) — mindstudio.ai/blog/animated-3d-websites-claude-code-ai-video-generation/

推文来源 — x.com/Oluwaphilemon1/status/2052980072550727932 和 /status/2053773794288751009