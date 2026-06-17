---
title: 前端设计 SVG skills，让 AI 更擅长画图标
author: 树灰的AI笔记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxOTUwMTY5Mw==&mid=2247484582&idx=1&sn=05c98276ec21c81e696c3895ed0d8e3d&chksm=9a1fe10257f40b1922b9d5bdcc3160bb804c8b1159ee2a1650eddd6ce9f13229d50e1c4b43b9&mpshare=1&scene=24&srcid=0122Vk95BvwR2NizSpIjuquG&sharer_shareinfo=97763a3a5e8d864f214c8c591820c856&sharer_shareinfo_first=97763a3a5e8d864f214c8c591820c856#rd
---

之前用 AI 生成 spaceX 网站，虽然效果不错，但生成的图标不可用

今天我们用“svg-icon-generator”让 AI 画图标，看看能否锦上添花

先看前后对比，内容不变的情况下，第一张是之前生成的图标，第二张是用了 svg skill 生成的

【原版】

【用了 svg skill之后】

# SVG-skill 介绍

最开始我是从 skill 市场上找了一个，但是 skill 中只有简单的 markdown 介绍，核心就四句话

* 为 svg 图标生成器提供分步指导
* 遵循行业最佳实践和模式
* 生成可投入生产的代码和配置
* 根据通用标准验证输出

效果差强人意，正好我看到赛博禅心发了一篇用 Coze 生成公众号svg的文章，正好结合下

我将两者结合下，让 Claude 为我生成了一个新的 svg skill，生成的 svg 兼容网页和公众号

上面网页的效果就是用这个 skill 生成的，简介如下

```
WeChat SVG Interactive Skill 总结  
/.claude/skills/wechat-svg-interactive/SKILL.md  
  
  核心能力  
  | 点击交互：多步骤点击流程、按钮响应、状态切换  
  | SMIL 动画：原生 SVG 动画（微信唯一支持方式）  
  | Pattern 圆角：图片圆角裁剪的替代方案  
  | 预设模板：Mac Terminal、Cyberpunk、Minimal、Glassmorphism、Retro  
  
  ⚠️ 微信六大限制（关键）  
  
  | 限制              | 解决方案             |  
  |-------------------|----------------------|  
  | 不支持 JavaScript | 纯 SVG + SMIL        |  
  | 不支持 CSS 动画   | 用 <animate> 标签    |  
  | 不支持 id.click   | 用 begin="mousedown" |  
  | 不支持渐变颜色    | 全部使用纯色         |  
  | 定时动画不可靠    | 用点击触发代替延迟   |  
  | clipPath 图片溢出 | 用 pattern 填充方式  |  
  
  常用代码模式  
  
  <!-- 点击显示 -->  
  <animate attributeName="opacity" begin="mousedown" dur="0.3s" from="0" to="1"/>  
  
  <!-- 点击缩放 -->  
  <animateTransform attributeName="transform" type="scale"  
                     begin="mousedown" end="mouseup" from="1" to="0.95"/>  
  
  <!-- 圆角图片 -->  
  <pattern id="img" patternUnits="objectBoundingBox">  
    <image href="..." preserveAspectRatio="xMidYMid slice"/>  
  </pattern>  
  <rect fill="url(#img)" rx="50"/>  
  
  适用场景  
  
  - 营销活动卡片  
  - 抽奖转盘  
  - 多步骤教程  
  - 节日祝福  
  - 互动游戏
```

以上就是使用 svg skill 的过程，也传到我的 github 了，感兴趣的可以看看https://github.com/shuguan12/claudecode/tree/d0a884bd85e0d742aa185d4e1653ccb0dcf38bd3/skills

不过在这里，我更想给大家分享这个思路， skill 不在于多少，也不用天天盯着塞到收藏夹

而是能够根据个人需求，调试skill 效果、生产流程、迭代内容，真正为自己提效节省时间

---

欢迎关注～不求多不求快，慢慢更新实用技巧～