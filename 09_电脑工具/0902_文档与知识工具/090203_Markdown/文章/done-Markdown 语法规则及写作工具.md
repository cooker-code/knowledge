---
title: Markdown 语法规则及写作工具
author: 肆舆
date:
url: https://mp.weixin.qq.com/s?__biz=MzIwMTUyMTEwOA==&mid=2650257766&idx=1&sn=8f7f4a3ad905d9ec53fb7a2f662c2114&chksm=8f91cf15351909757cac90034615bc7ff486a521c324f5baf3512fc84cf5dc02e5eac0a48bf4&mpshare=1&scene=24&srcid=1218Be2Mq44kH5h3O1oUUGsI&sharer_shareinfo=9389aa4855deeb50ead1596f923ce186&sharer_shareinfo_first=9389aa4855deeb50ead1596f923ce186#rd
---
> 已吸收至：[[09_电脑工具/0902_文档与知识工具/090203_Markdown/090203_核心知识点/Markdown写作与渲染边界|Markdown 写作与渲染边界]]


# Markdown 语法规则及写作工具

Markdown 是一种轻量级标记语言，通过简单符号实现文本格式化。相对于 LaTeX 这类专业的排版语言，Markdown 更简单易用，适用于笔记，博客等场景。

Markdown 的语法规则较为简单，但不同的 Markdown 解析器对语法的支持可能有所不同，本文仅对基础语法进行简单的介绍，并介绍一些常用的 Markdown 写作工具。

## Markdown 语法规则

### 标题

使用`#`表示，数量对应级别。

```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

### 文本样式

* • 加粗：`**加粗文本**` 或 `__加粗文本__` → **加粗文本**
* • 斜体：`*斜体文本*` 或 `_斜体文本_` → 斜体文本
* • 粗斜体：`***粗斜体文本***` 或 `___粗斜体文本___` → **粗斜体文本**
* • 删除线：`~~删除线文本~~` →~~删除线文本~~
* • 转义字符：使用`\`输出特殊符号，例如：`\* 这不是斜体 \*` → \* 这不是斜体 \*
* • 引用：使用 `>` 表示：`> 引用文本` →
  > 引用文本
* • 分割线：三个或更多 \* 、 - 或 \_，例如 `***`，`---`，`___` →

---

### 列表

#### 无序列表

使用 `-`、`*` 或 `+`，示例：

```
- 项目1
- 项目2
  - 子项目
```

#### 有序列表

使用数字加点，示例：

```
1. 第一项
2. 第二项
  1. 子项
```

#### 任务列表

使用`- [x]` 和 `- [ ]`，示例：

```
- [x] 已完成任务
- [ ] 待办任务
```

### 图片

图片的格式可表示为`![显示文本](图片地址或链接)`，显示文本在图片无法加载时显示。可以使用本地图片地址，也可使用图床生成链接。

### 表格

使用 `|` 和 `-` 创建，使用 `:` 改变对齐方式，示例：

| 默认对齐 | 左对齐 | 居中对齐 | 右对齐 |
| --- | --- | --- | --- |
| 单元格内容 | 单元格内容 | 单元格内容 | 单元格内容 |

```
| 默认对齐 | 左对齐 | 居中对齐 | 右对齐 |
| --- | :--- | :---: | ---: |
| 单元格内容 | 单元格内容 | 单元格内容 | 单元格内容 |
```

### 代码

* • 行内代码：`` `inline code` `` → `inline code`
* • 代码块：三个反引号包裹，可指定语言，若需要在代码块内输入三个反引号，则代码块的界定符需比内部使用更多的反引号：

```
```python
print("Hello, Markdown!")
```
```

### 数学公式

支持 LaTeX 语法，需依赖 MathJax 或 KaTeX 渲染。

* • 行内公式：`$y = x^2$` →
* • 行间公式：`$$e^{i\theta} = \cos(\theta) + i\sin(\theta)$$` →
* • 在行间公式嵌入 LaTeX 公式环境，示例：

```
$$
\begin{equation}
  |x| = 
    \begin{cases}
      x & \text{if } x \geq 0 \\
      -x & \text{if } x < 0
    \end{cases}
\end{equation}
$$
```

* • 使用 `\tag{}` 命令为公式手动指定一个编号或标记，用于公式的引用和说明，`\tag*{}` 命令可以去除包围标签的圆括号，在引用公式时，标签需手动输入，示例：

```
$$
\begin{equation}
    \begin{split}
        e^x &= \sum_{n=0}^{\infty} \frac{x^n}{n!}  \\
        &= 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + \cdots + \frac{x^n}{n!} + \cdots
    \end{split}
\end{equation}
\tag*{1-1}
$$
```

### 链接

#### 行内链接

行内链接直接在行内定义链接的文本和地址。其格式可表示为`[显示文本](链接地址 "可选标题")`，`显示文本` 是页面上显示的文字，`链接地址` 是链接的实际地址，`"可选标题"`是鼠标悬停在链接上时出现的提示文字，可以省略。

示例：
`[微信官网](https://weixin.qq.com/ "微信官网")` → 微信官网[1]

#### 参考链接

当同一个链接在文章中出现多次时，使用这种方法可以方便统一管理。语法分为两部分：

* • 链接位置：在文中需要链接的地方写 `[链接文字][链接标识]`。
* • 链接定义：在文档的任意位置（通常放在末尾）写 `[链接标识]: 真实的链接地址`。

示例:

```
打开[微信公众平台][1]，可创建自己的公众号。

[1]:https://mp.weixin.qq.com/
```

#### 自动链接

用尖括号 `< >` 包裹网址或邮箱地址，Markdown 会自动将其转换为可点击的链接。

#### 脚注链接

脚注链接用于在文章底部添加解释或引用来源。语法分为两部分：

* • 添加注脚标记：在需要注释的文字后面加上 `[^注脚标识]`。
* • 定义注脚内容：在文档末尾（或其他位置）写 `[^注脚标识]: 注脚的解释内容`。

示例：

使用 Markdown[1]可以高效地写作，利用微信公众平台[2]可分享自己的文章。

`1.`HyperText Markup Language 超文本标记语言↩︎
`2.`微信公众平台: https://mp.weixin.qq.com/↩︎

```
使用 Markdown[^1] 可以高效地写作，利用微信公众平台[^2]可分享自己的文章。

[^1]:HyperText Markup Language 超文本标记语言
[^2]:微信公众平台: https://mp.weixin.qq.com/
```

#### 锚点链接

锚点链接用于在当前文档内部进行跳转，通常需要为标题定义一个标识符，然后用普通链接语法指向它。语法分两个部分：

* • 定义锚点：在标题后加上 {#自定义锚点名称}，例如：`## 章节标题 {#section1}`。
* • 链接到锚点：`[跳转到某一章](#section1)`。

### 复杂样式

Markdown 本身不支持复杂样式，但可通过嵌入 HTML 标签并结合 CSS 样式实现个性化排版，核心是利用 HTML 的 `style` 属性或 `<style>` 标签定义样式。

Markdown 已实现的文本样式:

* • 加粗文本：`<strong>` / `<b>`
* • 倾斜文本：`<em>` / `<i>`
* • 删除线：`<del>`
* • 分割线：`<hr>`
* • 行内代码：`<code>`
* • 代码块：`<pre>`
* • 行内引用：`<q>`
* • 引用块：`<blockquote>`

使用 HTML 标签可实现 Markdown 无法实现的文本样式，例如：

* • 上标：`<sup>上标</sup>`。`X<sup>2</sup>` → X2
* • 下标：`<sub>下标</sub>`。`Y<sub>2</sub>` → Y2
* • 下划线：`<u>下划线</u>` 或 `<ins>下划线</ins>` → 下划线
* • 高亮文本：`<mark>高亮文本</mark>` →高亮文本

使用 `<div>` 标签下的 `align` 属性可以设置文本对齐方式，文本对齐示例：

这段文本居左显示

这段文本居中显示

这段文本居右显示

```
<div align="left">这段文本居左显示</div>
<div align="center">这段文本居中显示</div>
<div align="right">这段文本居右显示</div>
```

使用 `<font>` 标签下的 `color` 属性可以设置文本颜色，`face` 属性可以设置文本字体，`size` 属性可以设置文本字号，文本颜色示例：
这是红色文字
这是黄色文字
这是绿色文字
这是蓝色文字

```
<font color="red">这是红色文字</font>
<font color="yellow">这是黄色文字</font>
<font color="#00FF00">这是绿色文字</font>
<font color="#0000FF">这是蓝色文字</font>
```

## Markdown 写作工具

### Visual Studio Code

Visual Studio Code 简称 VS Code，是一款由微软开发的免费、开源、跨平台的代码编辑器，兼具轻量、高效和高度可定制的特性。

1. 1. 下载
   访问 VS Code 官网[2] 下载对应操作系统（Windows, macOS, Linux）的安装程序。
2. 2. 安装
   运行安装程序，按照引导进行安装。
3. 3. 语言设置
   启动 VS Code，使用快捷键 `Ctrl+Shift+P` 打开命令面板，输入“Configure Display Language”或“Chinese”，安装中文语言包并重启。
4. 4. 插件安装
   在扩展界面搜索并安装对应插件。

* • `Markdown All in One` 插件支持 Markdown 语法高亮、自动补全等，
* • `Markdown Preview Enhanced` 插件提供强大的 Markdown 预览功能，支持数学公式、图表等，可导出 `html`、`pdf` 等多种格式。

5. 5. 创建项目
   在文件系统创建相应的项目文件夹，点击 `文件` → `打开文件夹`，选择创建的项目文件夹并打开。在项目文件夹下创建 `.md` 文件，并在文件中撰写所需内容。
6. 6. 预览
   在文件编辑界面右键单击，选择 `MPE:打开侧边预览` 即可预览 Markdown 内容。右键点击侧边预览窗口可以选择 `Preview Theme`、`Code Block Theme` 和 `Reveal.js Theme`。
7. 7. 导出

* • `html` 文件
  右键点击侧边预览窗口，选择 `Export` → `HTML` → `HTML (offline)` 可导出 `html` 文件。
* • `pdf` 文件
  右键点击侧边预览窗口，选择 `Export` → `Chrome (Puppeteer)` → `PDF` 可导出 `pdf` 文件。如需显示代码块背景，在 `Markdown Preview Enhanced` 插件设置选项里找到 `Markdown-preview-enhanced: Print Background`，勾选 `Whether to print background for file export or not`。
* • 另存为 `pdf` 文件
  右键点击侧边预览窗口，选择 `Open in Browser`，`Ctrl+P` 打印网页，选择 `另存为PDF`。通过勾选 `背景图形` 可显示代码块背景。布局、纸张大小、缩放、边距、页眉页脚可自行设置。

8. 8. 样式设置
    使用快捷键 `Ctrl+Shift+P` 打开命令面板，搜索 `Markdown Preview Enhanced: Customize CSS` ，带 `(Global)` 后缀的是设置全局样式，带 `(Workspace)` 后缀的是设置工作区样式。打开 `Markdown Preview Enhanced: Customize CSS (Global)`，通过填写代码可设置预览和打印的样式，如字体、字号、行距、背景颜色等。示例代码如下：

```
html body {
  font-family: "思源宋体 CN Medium", "Times New Roman";
  font-size: 16px;
  font-weight: normal;
  line-height: 1.6;
  color: #000000;
  background-color: #ffffff;
  overflow: initial;
  box-sizing: border-box;
  word-wrap: break-word;
}
```

1. 9. 代码自动换行
   设置代码自动换行是为了避免导出 `pdf` 文件时部分代码显示不全。打开 `Markdown Preview Enhanced: Customize CSS (Global)`，填写如下代码即可实现代码自动换行。

```
.markdown-preview.markdown-preview {
  
  // 自动换行：预览用
  pre, code {
    white-space: pre-wrap;
    word-break: break-word;
    overflow: hidden;
  }

  // 自动换行：打印用
  @media print {
    pre, code {
      white-space: pre-wrap;
      word-break: break-word;
      overflow: hidden;
    }
  }
}
```

### MarkText

MarkText 是一款专注于速度和可用性的开源 Markdown 编辑器，支持实时预览模式，还提供了源码模式、打字机模式和专注模式。

MarkText 可跨平台使用，适用于 Windows、macOS 和 Linux 三大主流桌面操作系统。可在 MarkText GitHub 主页[3] 下载。

### Doocs WeChat Markdown Editor

WeChat Markdown Editor 是一款专为微信公众号文章排版设计的开源在线编辑器，可以使用 Markdown 语法写作，然后一键生成样式美观、兼容微信公众号后台的图文内容，避免了手动排版的繁琐。支持 Markdown 语法、自定义主题样式、代码高亮、数学公式、Mermaid 图表、多图床等特性。

访问 WeChat Markdown Editor 官网[4]，可在线使用。访问 doocs/md GitHub 主页[5] ，可了解源代码自行部署。

#### 引用链接

`[1]` 微信官网: *https://weixin.qq.com/*
`[2]` VS Code 官网: *https://code.visualstudio.com/*
`[3]` MarkText GitHub 主页: *https://github.com/marktext/marktext/*
`[4]` WeChat Markdown Editor 官网: *https://md.doocs.org/*
`[5]` doocs/md GitHub 主页: *https://github.com/doocs/md/*
