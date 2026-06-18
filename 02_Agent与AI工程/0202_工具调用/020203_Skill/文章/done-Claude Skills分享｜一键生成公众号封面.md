> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: Claude Skills分享｜一键生成公众号封面
author: 纯白精选
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzMzE2MzY1NA==&mid=2247489121&idx=1&sn=644cefd5808c7ad31ff04fabdbfbf36b&chksm=fbe22289e704a91c405cc32fb077e49463628e1c7096fb40ff803bce2fb5fe439e769040d45e&mpshare=1&scene=24&srcid=1113OfYbATsMml9snBmcaed6&sharer_shareinfo=9da53d772b36256f1d4693107a21b04b&sharer_shareinfo_first=9da53d772b36256f1d4693107a21b04b#rd
---

哈喽，大家好，我是纯白。

上篇分享了[《Claude Skills 完全指南：从入门到精通（附实战案例）》](https://mp.weixin.qq.com/s?__biz=MzUzMzE2MzY1NA==&mid=2247489095&idx=1&sn=9bae459792bb9891530e005aa157ca6a&scene=21#wechat_redirect)相信大家的Skills有了一定的了解。

今天通过日常如何生成公众号封面的例子，来进一步感受Claude Skills带来的便利。

之前使用提示词手动完成，现在通过Claude Skills就能自动完成这一过程。

同时相信大家会对mcp和skills的区别也有进一步的认识。

首先大家看一下三种方式的对比

|  |  |  |  |  |
| --- | --- | --- | --- | --- |
| 方案 | 特点 | 费用 | 操作 | 优劣势 |
| lovart.ai（收费服务） | 高质量底图生成 | 付费使用 | 平台直接生成 | ✅ 质量高，❌ 需付费 |
| 即梦官网 + Agent + HTML拼接（手动操作） | 免费积分 + 手动操作 | 可使用免费积分 | 复制提示词 → 调整参数 → 手动HTML拼接 | ✅ 成本低，❌ 步骤繁琐 |
| Claude Code Skills（完全自动化） | 一键生成，30秒完成 | 即梦免费积分 | 提供关键词和标题即可 | ✅ 完全自动化 + 成本低，❌ 无明显劣势 |

先忍不住展示一下成果，哈哈哈～～～

### lovart的方式

之前还有免费的积分可用，之后没有了。

### 即梦官网Agent

既然lovart也是调用即梦的模型，那同样的提示词在官网应该可以实现同样的效果。

即梦Agent的方式生成的html给出的图是占位符。你需要获取到地图的src源地址，然后在`https://html.inshub.cn/` 页面去替换。之后再去下载。

### 公众号封面生成器

有用大模型对中文字体输出的随性，所以生成封面的逻辑是底图+html渲染的方式。

那就可以自定义一个生成公众号封面的skills, 使用mcp生成底图，然后固定html输出去渲染，直接输出最终结果。

生成过程，可以看到调用公众号封面生成器 这个skill

节选mp-cover-generator skill的部分逻辑

```
.claude/skills/mp-cover-generator/SKILL.md
# 公众号封面生成器 Skill
```
---
name:"公众号封面生成器"
description:"根据主题和标题生成现代风格的公众号封面图，包含3D插画、渐变文字和副标题标签的完整HTML"
version:"2.0.0"
dependencies:
-"jimeng-image-generator (基于即梦AI的图像生成服务)"
-"HTML/CSS 知识"
-"Python 3.8+ 环境"
-"即梦AI Token (JIMENG_TOKEN)"
---

# 公众号封面生成器 Skill

## 🎯 核心任务

根据用户给定的**主题**、**标题**、**日期**和**作者名**，创建一个符合现代科技/生活方式调性的**公众号封面图**。最终产出物是一个**固定宽高比为21:9**的、包含内联样式的完整HTML文件。

## 🎨 视觉风格总览

***主题风格**:**可爱、圆润、简洁的3D插画风格(Cute,Soft,Minimalist3DIllustration)**。质感类似皮克斯动画或黏土定格动画，具有柔和的光影和玩具般的亲和力。
***色彩**:整体色调和谐、明快。背景为低饱和度的纯色或同色系渐变，以确保左侧文字有良好的可读性。
***构图**:严格的非对称式"右图左文"布局，视觉焦点清晰，主次分明。

## 何时使用此 Skill

当用户需要：
-创建公众号文章封面
-生成社交媒体横幅图
-制作带有标题和日期的宣传图
-需要3D插画风格的图文结合设计

关键触发词：公众号封面、封面图、banner、头图、宣传图

## MCP 配置

### jimeng-image-generator 设置

本技能使用jimeng-image-generatorMCP来生成图片。该MCP已经配置在项目的`.mcp.json`文件中：

```json
{
"mcpServers": {
    "jimeng-image-generator": {
      "command":"python",
      "args": ["xx/xx/mcp/jimeng_mcp/server.py"],
      "env": {
        "JIMENG_TOKEN":"your_jimeng_token_here"
      },
      "description":"基于即梦AI的图像生成服务，用于生成公众号封面底图"
    }
  }
}
```

### 环境要求

确保系统已安装：
-Python3.8+
-即梦AIToken(需要设置JIMENG_TOKEN环境变量)
-jimeng-image-generatorMCP所需依赖(fastmcp,requests等)

## 📐 生成规范与流程

### 第一部分：底图生成 (Agent 生图)

此区域用于生成无任何文字的、符合构图和风格规范的背景插图。

#### 尺寸与布局
***图片宽高比**:**21:9**(3024x1296像素)
***主体位置**:核心的3D视觉元素（如人物、设备、场景）**必须全部位于画面右侧的30%-40%区域内**
***留白区域**:画面**左侧60%-70%的区域必须是干净的留白**，可以是纯色或非常微妙的渐变，不能有任何干扰视觉的元素

#### 【极其重要】禁止的视觉元素
***严禁任何形式的文字**:生成的底图中绝对不能包含任何字母、数字或符号
***严禁边框**:图片不能有任何形式的内外边框

#### 插图生成要求

**内容**:插图需根据用户输入的**主题关键词**，象征性地创作一个场景。

**背景**:背景必须是简单的纯色或从左到右的同色系渐变(`subtlemonochromaticgradientbackground`)。

**【风格重点-严格遵守】**:

**核心风格**:
-✅**必须是可爱、圆润的卡通3D风格(Cute,rounded,cartoon3Dstyle)**
-✅所有模型都应有**柔和的边缘**和**哑光或黏土般的材质(softedges,matteorclay-likematerial)**
-✅整体感觉类似**皮克斯动画(Pixaranimationstyle)**或**玩具质感(toy-liketexture)**
-✅光照必须是明亮且柔和的(`brightandsoftlighting`)

**规避风格**:
-❌**严格禁止**生成以下风格：
-**霓虹/赛博朋克风(NOneon,NOcyberpunk)**
-**暗黑深沉风(NOdarkthemes)**
-**抽象科技线条风(NOabstracttechlines)**
-**玻璃质感或写实渲染风(NOglassmorphism,NOphotorealism)**

### 第二部分：文字层叠加 (Agent HTML实现)

此部分基于第一步生成的底图，通过绝对定位的方式精确添加文字层。

#### HTML结构
*创建一个主容器`div`，`position:relative;`
*底图作为`<img>`标签置于容器底层，`width:100%;display:block;`
*所有文字元素分别使用独立的`div`或`p`标签，并通过`position:absolute;`进行定位

#### 字体
优先使用流行的中文字体

#### 内容与样式 (基于父容器尺寸)

1.**日期(Date)**:
   *内容:获取当前星期和日期，格式:Fri.9.15
   *样式:`position:absolute;top:8%;left:6%;`字号`font-size:1.5vw;`(或等效的px值),颜色`color:#999999;`

2.**核心标题(Headline)**:
   *内容: [由用户指定，需拆分为两行]
   *样式:`position:absolute;top:18%;left:6%;`字号`font-size:4vw;`(或等效的px值),**加粗**`font-weight:bold;`,颜色`color:#000000;` 或 `#FFFFFF;` (需根据底图颜色自动判断对比度), 行高 `line-height: 1.2;`

3.**作者(Author)**:
   *内容:固定为：纯白精选
   *样式:`position:absolute;bottom:8%;left:6%;`字号`font-size:1.5vw;`(或等效的px值),颜色`color:#666666;`

## 💡 系统执行流程

### 第一步：收集必要信息

从用户处获取以下信息：
1.**主题关键词**：用于生成插画内容（必填）
2.**标题**：封面的核心标题文字（必填）

自动生成的信息：
-**日期**：自动获取当前星期和日期，格式为`Fri.11.11`
-**作者名**：固定为"纯白精选"

### 第二步：使用MCP生成底图

调用jimeng-image-generatorMCP的`generate_image`工具来生成底图。

#### MCP 调用示例

使用`generate_image`工具生成底图：

```python
# 调用 jimeng-image-generator MCP
generate_image(
    prompt="Createacute,rounded,cartoon3Dillustrationwith [主题关键词] theme. Style:Pixar-likeanimation,toy-liketexture,softedges,matte/clay-likematerials,brightandsoftlighting,vibrant colors. Main elements: [具体描述主题相关的物体和图标].NOpeople,NO characters. Composition:MainelementspositionedinRIGHT30-40%offrame.LEFT60-70%clean space with gradient background. Aspect ratio: 21:9 ultra-wide. IMPORTANT:NOtext,NOpeople,NOcharacters,NOneon/cyberpunk,NOdarkthemes,NOabstracttechlines,NOglassmorphism,NOphotorealism.",
    save_folder="/path/to/output/folder/",# 替换为实际的输出文件夹路径
)
```


### 第三步：构建 HTML 文件

基于生成的底图，创建包含文字层的HTML文件。

#### HTML 结构模板

```html
<!DOCTYPEhtml>
<htmllang="zh-CN">
<head>
    <metacharset="UTF-8">
    <metaname="viewport"content="width=device-width,initial-scale=1.0">
    <title>公众号封面</title>
    <style>
    ...
    </style>
</head>
<body>
    <divclass="cover-container">
        <!--背景图片-->
        <imgsrc="[生成的底图URL或base64]"alt="封面背景">

        <!--日期-->
        <divclass="date">[当前日期，格式:Fri.11.11]</div>

        <!--标题-->
        <h1class="headline">[用户指定的标题]</h1>

        <!--作者-->
        <divclass="author">纯白精选</div>
    </div>
</body>
</html>
```

#### 智能颜色决策

-自动检测底图左侧区域的亮度
-如果背景偏暗→标题颜色使用白色(`#FFFFFF`)
-如果背景偏亮→标题颜色使用深色(`#000000`)

### 第四步：输出最终文件

1.保存为独立的HTML文件（例如：`mp_cover_[日期].html`）
2.向用户展示预览效果
3.提供下载链接或完整代码

## 示例对话流程

**用户输入：**
```
生成一个关于AI的公众号封面，标题是"即梦无限画布"
```

**Assistant执行：**

1.**收集信息**
   -主题：AI/科技创新
   -标题：即梦无限画布
   -日期：Fri.11.11（自动获取）
   -作者：纯白精选（固定）

2.**使用MCP生成底图**
   -调用jimeng-image-generatorMCP的`generate_image`工具：
     ```python
     generate_image(
         prompt="Createacute,rounded,cartoon 3D illustration with AI/technology theme. Style:Pixar-likeanimation,toy-liketexture,softedges,matte/clay-likematerials,brightandsoftlighting,vibrant colors. Main elements:GlowingAIbrainiconwithfloatingskillbadges,lightbulbs,books,gearsandneuralnetworknodes.NOpeople,NO characters. Composition:ElementspositionedinRIGHT30-40%offrame.LEFT60-70%clean space with purple-blue gradient background. Aspect ratio: 21:9 ultra-wide. IMPORTANT:NOtext,NOpeople,NOcharacters,NOborders,NOneon/cyberpunk,NOdarkthemes,NOabstracttechlines,NOglassmorphism,NOphotorealism.",
         save_folder="/path/to/output/",
     )
     ```

3.**构建HTML**
   -创建包含底图和文字层的完整HTML文件
   -根据背景亮度自动调整文字颜色

4.**输出**
   -保存为`mp_cover_20251111.html`
   -向用户展示并提供下载

## 注意事项

-**图片尺寸配置**：严格使用3024x1296（21:9比例）
-**布局比例调整**：主体元素严格位于右侧30%-40%区间，左侧60%-70%必须保持干净留白
-确保生成的底图完全符合风格规范，特别是可爱圆润的卡通3D风格
-文字层的定位使用相对单位（vw/%），确保响应式
-标题如果过长，需要智能换行，保持在2-3行以内
-星期和日期格式必须为英文星期缩写格式（如Fri.11.11）
- 作者名固定为"纯白精选"，不接受用户自定义
```

### 相关链接

* 📝 完整提示词：「公众号可爱3D风格封面生成器」

https://prompt.inshub.cn/discover

* 🎨 html在线编辑：https://html.inshub.cn/

### 完整skills获取

回复"skills"获取完整SKILL.md内容及使用说明。

如果你觉得有用，别忘了点击右下角的「 分享 · 收藏 · 在看 · 赞 」噢