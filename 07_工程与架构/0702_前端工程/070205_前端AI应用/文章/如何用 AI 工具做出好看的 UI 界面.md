---
title: 如何用 AI 工具做出好看的 UI 界面
author: 开发者山石
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk0NDYwMzc1NA==&mid=2247484732&idx=1&sn=75a259dac6dc28eb358c2894f363a61d&chksm=c207c7c6d7b6d157ff7c550749f887ccd6cfeecf0da3184d4a5f807c76c110638e7ed69dc7b1&mpshare=1&scene=24&srcid=1223cnChKp9jEOIZfQJa5b0N&sharer_shareinfo=ef646967da1d650ef67a20a89d19a117&sharer_shareinfo_first=ef646967da1d650ef67a20a89d19a117#rd
---

做出海产品的时候，需要不断得发外链来提高网站排名，当产品多了之后，怎么记录每个产品发了哪些外链，还需要发哪些外链成了一个比较麻烦的事情，最好有个工具来管理整个外链的发布情况。为了满足这个需求，最近打算 Vibe 一个小产品出来。

首先要解决的是前端设计问题，以前我做产品的时候，完全没有设计，画了个草稿之后就开始写代码了，或者从模板开始写代码。但是都已经快 2026 年了，我也想体验一下现在的 AI 的 UI 设计能力。我选择的工具是 Stitch，这是 Google 的产品，集成了目前风头正盛的 Gemini Pro 3 和 Nano Banana Pro，而且免费，没道理不试一下。

## 整体流程

以下过程不是一次性完成的，是经过多次探索出来的，文章后面会给出我做了哪些探索。

首先我用一大段提示词描述我的需求，这段提示词中包括：

1.

为什么要做这个网站

2.

这个网站有哪些功能

3.

我期望的交互逻辑是什么样的，中间描述 UI 的大体样子

要尽可能详细，把你能想到的都写出来。

```
平时我会做多个网站，需要不断发外链来提高网站的排名，我想做一个工具来记录每个网站外链发布情况。这个工具是一个网站，支持以下功能：  
  
- 支持邮箱密码登录，由于只有我一个人在使用，可以通过环境变量注入固定的邮箱密码，验证的时候匹配环境变量中的邮箱密码即可  
- 有个全局外链库，可以进行增、删、改、查，每个外链有几个属性：DR、URL、标题、分类、价值（高中低）、价格、备注等字段  
- 每新建一个项目，创建一个外链发布跟踪表，记录每一个全局外链是否提交状态，默认未提交，提交后需要手动更新为提交  
- 在全局外链库中新增一个外链，所有项目同步这个外链，初始为未提交  
- 每一天，每个项目，按照一定的排序规则（待定，可以自定义多种策略，默认按价值从高到低），自动选择 2 个外链（数量可配置），放到醒目位置，提醒我提交  
  
我期望的交互逻辑是：  
  
1. 打开网站，如果没有登录，展示登录页面，只需要输入邮箱密码即可  
2. 登录后，进入 Dashboard 页面，Dashboard 页面为左右布局，左边是侧边栏，右边是内容区  
3. 侧边栏包含：  
   - 外链库 (Library)  
   - 项目列表 (Projects), 二级目录是每个项目  
   - 底部功能区：  
     - 退出登录按钮  
     - 新建项目按钮  
4. 内容区：  
   1. Library: 分为 Header 和 Body 两部分，Header 包含新增外链按钮，Body 为表格形式  
   2. Project: 分为 Header 和 Body 两部分，Header 包含功能按钮如删除项目，Body 分为上下两块，上面为今日优先提交的外链列表，下面为所有外链列表
```

然后把这段提示词输入到的 Stitch 中，选择 Gemini 3 Pro 和 Web，官网地址：https://stitch.withgoogle.com/

就可以得到一个初步的设计方案了，然后可以通过 “变体” 进行修改。目前虽然 Stitch 支持了 Nano Banana Pro 进行图片标注修改，但是体验下来没有生成多个变体效果好。

原始登录页面

因为是个人项目，化繁为简之后的效果。输入提示词：因为这个项目是个人使用，不需要 Privacy 和 Item Of Service，也不需要忘记密码。

原始外链库页面

修改之后的效果，进行了多阶段修改，提示词包括：

1.

侧边栏登出按钮与 Create 按钮等宽，颜色不一样

2.

内容页的数据统计区域换成筛选功能栏

原始项目页面

修改后的项目页面，进行了多阶段修改，包括：

1.

选中 Library 页面和项目页面，项目页面的侧边栏与 Library 页面的侧边栏一致

2.

使用 Nano banana Pro 标注选中 Today’ Tasks 部分，要求改为列表

总体上这三个页面大体上已经满足了需求，更细节的地方需要在具体实现的时候再进行调整，已经相当满意了。

## 做过的尝试

### 1、简单提示词+草图

我一开始的提示词中没有交互逻辑那一块内容，只是进行了功能描述

```
平时我会做多个网站，需要不断发外链来提高网站的排名，我想做一个工具来记录每个网站外链发布情况。这个工具是一个网站，支持以下功能：  
  
- 支持邮箱密码登录，由于只有我一个人在使用，可以通过环境变量注入固定的邮箱密码，验证的时候匹配环境变量中的邮箱密码即可  
- 有个全局外链库，可以进行增、删、改、查，每个外链有几个属性：DR、URL、标题、分类、价值（高中低）、价格、备注等字段  
- 每新建一个项目，创建一个外链发布跟踪表，记录每一个全局外链是否提交状态，默认未提交，提交后需要手动更新为提交  
- 在全局外链库中新增一个外链，所有项目同步这个外链，初始为未提交  
- 每一天，每个项目，按照一定的排序规则（待定，可以自定义多种策略，默认按价值从高到低），自动选择 2 个外链（数量可配置），放到醒目位置，提醒我提交
```

然后我用 excalidraw 花了 3 个页面的草图

我把这个提示词和草图给到 Stitch，生成页面拟合了草图风格，但又没有完全拟合，有点不伦不类

不过还别说，第二个图的设计还挺有趣的。

所以 Stitch 中上传图片的意思是实际生效的 Demo 图。

### 2、简单提示词+Gemini补全

第二个尝试是把前面简单的提示词输入到 Gemini 中，让它帮我生成 Stitch 这种设计工具的提示词，它的提示词如下。

Create a comprehensive SEO Backlink Management Dashboard for a single user. The design should be clean, professional, and data-driven, similar to a modern SaaS platform using Tailwind CSS and shadcn/ui style.

Layout Structure:

•

A fixed sidebar for navigation: Dashboard (Today's Tasks), Global Link Library, Projects, and Settings.

•

A top header showing the current view title and a "Quick Add Link" button.

Key Pages to Design:

1.

Dashboard (Focus Mode):

•

Display "Daily Submission Goals" grouped by Project.

•

For each project, show 2 recommended backlink cards (highlighted).

•

Each card displays: Title, URL, DR (Domain Rating), Value (High/Med/Low), and a "Mark as Submitted" button.

2.

Global Link Library (Inventory):

•

A robust Data Table featuring columns: DR, URL, Title, Category, Difficulty (Badge), Value (Badge), Price, Wait Time, and Actions (Edit/Delete).

•

Include search and filter controls (by category, DR range, or value).

3.

Project Tracking Page:

•

A header showing project name and overall progress bar (Submitted vs. Total).

•

A list or table showing all links from the global library specifically for this project.

•

Status toggles for each link: "Not Started" vs. "Submitted".

Visual Style:

•

Use a neutral color palette (Slate/Gray) with a primary accent color (e.g., Indigo or Emerald).

•

Use Radix UI inspired components: Modals for adding links, Dropdowns for sorting strategies, and clear Badges for status/priority.

•

Ensure high information density but maintain readability.

这个方案的问题是，它补充了很多我不需要的细节，比如这个 Dashboard 页面，虽然看起来很漂亮，但是不是我想要的。

但是这份提示词启发了我，它在提示词中对每个页面进行了描述，或许我可以自己这么干。

### 3、复杂提示词 + Gemini 补全

第三次尝试，我写了最终那一版提示词，包含了交互逻辑，然后我把这份提示词给到了 Gemini，让它帮我生成提示词，其实效果还不错。但是我后面又试了下直接把提示词给到 Stitch，发现效果差不多，所以就把 Gemini 这一步给省了。

## 总结

总体体验下来，Stitch 的能力其实挺强的，对于小项目完全够用了。它的交互方式跟 Lovart 差不多，这可能是后面设计类的 Agent 的主要交互方式了。明年 AI 设计类工具应该会更好用，又补齐了独立开发的一块短板，非常期待。

接下来将会以这份 UI 为起始，使用 Claude Code 把整个产品 Vibe 出来，开发记录也会分享出来，欢迎关注。