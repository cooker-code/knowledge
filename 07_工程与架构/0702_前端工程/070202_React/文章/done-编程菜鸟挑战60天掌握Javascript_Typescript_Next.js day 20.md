> 已吸收至：[[07_工程与架构/0702_前端工程/070202_React/070202_核心知识点/React架构与质量边界准则|React架构与质量边界准则]]、[[07_工程与架构/0702_前端工程/070202_React/070202_知识地图|070202_React知识地图]]

---
title: 编程菜鸟挑战60天掌握Javascript/Typescript/Next.js day 20
author: I.V. Epiphany
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5ODM1MjE2NQ==&mid=2247484056&idx=1&sn=6dbd42f7cc6d05dee70794ec438e4d18&chksm=975fc6b705dc65443c4c0d3fcaa1dcee641c68c8ec23aa6e9edef3b8bc3647ec1db16f1d2b4b&mpshare=1&scene=24&srcid=1128CIwvzOBA3fAt2ct4dEkj&sharer_shareinfo=05564b67525279fc46ea87481619bbe0&sharer_shareinfo_first=05564b67525279fc46ea87481619bbe0#rd
---

今天是从零学习Javascript/Typescript/Next.js的第20天。

以下是今天的学习内容：

291 CDN：content delivery network，内容发布网络，把内容缓存到全球各地的服务器节点，让用户从离自己最近的节点获取资源的技术。

292 static generation把网页变成可缓存的静态文件，CDN把这些静态文件在全球范围内快速送达

293 静态文件：static files，提前准备好、内容不会随用户访问而变化的文件。服务器只返回静态文件，无需运算/查数据库/动态生成，所以静态文件加载很快。常见的静态文件有HTML、CSS、JavaScript、图片、视频、字体等。

294 build：构建，把开发时的源代码打包、优化乃至生成静态文件的全过程，这一过程发生在部署之前（deploy，常见的部署网站有Vercel）。如果页面在用户访问时才生成，那不叫构建，叫服务器端渲染（server side rendering）或runtime rendering

295 浏览器能直接读取的语言有CSS、JavaScript、HTML三种（无需构建，所以构建也可以理解为把其他语言转化为HTML/CSS/JavaScript），HTML负责生成网页骨架结构，CSS负责网页样式外观，JavaScript负责和用户交互。

296 使用Next.js框架，网站会预渲染（pre-rendering），构建时直接生成静态HTML文件，无需JavaScript加载；但等到JavaScript加载之后（hydration），静态HTML文件被激活，页面上的react组件才有和用户的交互能力。不使用next.js框架的纯react.js网站默认不预渲染（除非自己实现），在加载JavaScript之前页面一片空白。

297 pre-rendering的两种方式：static generation/server-side rendering，区别在于为页面生成HTML文件的时机。Static generation在构建时生成HTML文件，文件会在以后的每次用户请求中反复使用；server-side rendering在每次用户请求时生成HTML文件

298 HTML JavaScript CSS都是浏览器原生支持的，无需编译无需打包无需转换，浏览器遇到HTML直接解析为DOM，遇到CSS直接解析为CSSOM，遇到JS直接运行。但真正浏览器零处理直接使用的只有HTML

299 尽可能多的使用static generation，但无法在用户请求之前渲染的页面（内容/数据更新频繁的页面）不能使用static generation，这时使用server-side rendering，虽慢一点但更新及时。或者不pre-rendering了，使用client-side rendering也行。

300 若static generation时需要fetch data，使用异步函数getStaticProps() （仅适用于pages router）