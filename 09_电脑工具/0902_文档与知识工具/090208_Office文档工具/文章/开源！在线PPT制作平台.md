---
title: 开源！在线PPT制作平台
author: 开源项目解读
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzk0OTYwMTE5Ng==&mid=2247486055&idx=1&sn=56c92c50c92ece3bbd06647b223b9c76&chksm=c2af4273625b47b427e9a18c53b0f0aaff87071240d466103cbf64ca9b72ba9b4e125763826c&mpshare=1&scene=24&srcid=1105DLyimpyDgJPCAL1tudxB&sharer_shareinfo=0e2f632b68ca5e74719b3e08420f33d2&sharer_shareinfo_first=0e2f632b68ca5e74719b3e08420f33d2#rd
---

大家好，我是开源项目解读小编，每天分享最重要的开源项目

主要分享GitHub上有趣、有意义、重要的项目

chatGPT出来后，PPT制作很火，有头部企业已经融资上亿。

今天分享开源的 在线PPT制作平台。

源代码：

http://www.gitpp.com/datasets/beautiful-ppt

用于创建简单多媒体演示的在线应用程序

# 数码幻灯片

一个用于创建演示文稿的简单应用程序。

它是根据 GNU AGPLv3 许可证发布的。除 Roboto Slab 和 Material Icons 字体（Apache 许可证版本 2.0）外，HKGrotesk、Montserrat、Quicksand、Lato、Open Sans、Source Sans Pro、League Gothic 和 News Cycle 字体（Sil Open Font 许可证 1.1）以及 Ubuntu 字体（Ubuntu字体许可证1.0)

### 准备并安装依赖项

```
npm install
```

### 启动开发服务器

```
npm run dev
```

### 环境变量（编译前在根目录创建的.env.生产文件）

```
AUTHORIZED_DOMAINS (* ou liste des domaines autorisés pour les requêtes POST et l'API, séparés par une virgule)  
VITE_DOMAIN (hôte de l'application, par exemple https://ladigitale.dev)  
VITE_FOLDER (dossier de l'application, par exemple /digislides/)  
VITE_PIXABAY_API_KEY (clé API Pixabay)
```

### 编译和缩小文件

```
npm run build
```

### API 需要 PHP 服务器

```
php -S 127.0.0.1:8000 (pour le développement uniquement)
```

### Apache 服务器的 .htaccess 配置（生产）

```
RewriteEngine on  
RewriteCond %{REQUEST_FILENAME} !-f  
RewriteCond %{REQUEST_FILENAME} !-d  
RewriteRule ^(.*)$ index.html
```

在线PPT制作平台之所以具有如此大的市场潜力，主要可以归结为以下几点原因：

### 1. 高效便捷的制作方式

* 打破时空限制：传统的PPT制作需要依赖特定的软件和电脑设备，而在线PPT制作平台打破了这一限制。只要有网络连接，用户可以在任何时间、任何地点通过电脑、平板或手机等设备轻松开始PPT的制作，极大地提高了工作效率和灵活性。
* 简化操作流程：在线PPT制作平台通常提供了一键生成、智能匹配等功能，用户只需简单的点击和选择即可快速生成专业的演示文稿，省去了传统软件繁琐的操作步骤。

### 2. 丰富的模板和素材资源

* 海量模板：在线PPT制作平台提供了大量的模板供用户选择，这些模板涵盖了各种行业和场景，如商务汇报、教育培训、产品展示等，满足了不同用户的需求。模板设计精美，风格多样，用户可以根据自身需求快速找到合适的模板。
* 多样化素材：除了模板外，平台还提供了丰富的图片、图标、图表等素材资源，用户可以轻松插入到PPT中，丰富演示文稿的内容，提升视觉效果。

### 3. 强大的编辑和自定义功能

* 灵活调整：在线PPT制作平台提供了丰富的编辑功能，如文字编辑、图片插入、图表制作等，用户可以根据自己的需求灵活调整PPT的布局、颜色、字体等样式，使演示文稿更加符合个人或团队的风格。
* 高度自定义：部分平台还支持用户自定义主题和风格，通过调整模板的细节元素，如背景、配色、字体等，实现个性化的PPT设计。

### 4. 实时协作与分享功能

* 团队协作：在线PPT制作平台通常具备实时协作功能，团队成员可以在同一份PPT上进行编辑和修改，实时查看对方的修改内容，提高了团队协作的效率。
* 便捷分享：用户可以将制作好的PPT一键分享给其他人，方便他们查看和讨论。同时，部分平台还支持将PPT导出为多种格式，如PDF、图片等，满足不同场景下的分享需求。

### 5. 市场需求旺盛

* 广泛的应用场景：PPT作为一种直观、高效的信息展示工具，广泛应用于各种商业场合，如企业内部汇报、项目路演、学术会议、产品发布等。随着企业对品牌形象和信息传递的要求越来越高，PPT制作的市场需求也在不断增加。
* 技能变现机会：对于熟练掌握PPT设计技巧的人来说，可以通过提供PPT制作服务来实现技能变现。在线PPT制作平台为这些专业人士提供了一个展示自己作品的平台，吸引了大量潜在客户的关注。

在线PPT制作平台以其高效便捷的制作方式、丰富的模板和素材资源、强大的编辑和自定义功能、实时协作与分享功能以及旺盛的市场需求等优势，展现出了巨大的市场潜力。

演示:https://ladigitale.dev/digislides/

源代码：

http://www.gitpp.com/datasets/beautiful-ppt

我们收集了GitHub上大量的开源项目，点击 **阅读原文** 查看更多学习项目