> 已吸收至：[[07_工程与架构/0702_前端工程/070204_Vue/070204_核心知识点/Vue项目工程化边界准则|Vue项目工程化边界准则]]、[[07_工程与架构/0702_前端工程/070204_Vue/070204_知识地图|070204_Vue知识地图]]

---
title: 用好腾讯开源的这两个前端框架，后端仔也能开发出好看的应用！
author: Java技术前线
date:
url: http://mp.weixin.qq.com/s?__biz=MzI3MjUxNzkxMw==&mid=2247491797&idx=1&sn=80e4b107bbeea591087d5e517c06db94&chksm=eb33fea3dc4477b5a9eed9a29b03955f2a85d20c1fc0e57c3a1dbf5380442f04a8eaf20a1c55&mpshare=1&scene=24&srcid=0613UR9PVgF5XQlS7RH3FXiR&sharer_sharetime=1686659444937&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

### ******点击关注公众号，Java干货****及时送达******👇****

今天推荐两个腾讯开源的前端框架，分别是 wujie（无界）和 Omi。

# wujie（无界）

无界微前端是一款基于 Web Components + iframe 微前端框架，具备成本低、速度快、原生隔离、功能强等一系列优点。

图片

Web Components 是一个浏览器原生支持的组件封装技术，可以有效隔离元素之间的样式，iframe 可以给子应用提供一个原生隔离的运行环境，相比自行构造的沙箱 iframe 提供了独立的 window、document、history、location，可以更好的和外部解耦。

无界微前端采用 webcomponent + iframe 的沙箱模式，在实现原生隔离的前提下比较完善的解决了上述问题。

### 特性

##### 成本低

* 主应用使用成本低
* 子应用适配成本低

##### 速度快

* 子应用首屏打开速度快
* 子应用运行速度快

##### 原生隔离

* css 样式通过 Web Components 可以做到严格的原生隔离
* js 运行在 iframe 中做到严格的原生隔离

##### 功能强大

* 支持子应用保活
* 支持子应用嵌套
* 支持多应用激活
* 支持应用共享
* 支持去中心化通信
* 支持生命周期钩子
* 支持插件系统
* 支持 vite 框架

##### 开源项目地址：

> * https://github.com/Tencent/wujie

# Omi

Omi 是一个前端跨框架跨平台框架。

图片

### 特性

* 跨框架，任何框架可以使用 Omi 自定义元素
* 提供桌面、移动和小程序整体解决方案
* 小巧的尺寸和高性能
* 基于 Shadow/Light Dom 设计
* 符合浏览器的发展趋势以及 API 设计理念
* Web Components + JSX/TSX 融合为一个框架 Omi
* JSX/TSX 是开发体验最棒(智能提示)、语法噪音最少、图灵完备的 UI 表达式，模板引擎不完备，模板字符串完备但是语法噪音太大
* 看看 Facebook React 和 Web Components 对比优势，Omi 融合了各自的优点，而且给开发者自由的选择喜爱的方式
* Shadow/Light DOM 与 Virtual DOM 融合，Omi 既使用了虚拟 DOM，也是使用真实 Shadow DOM，让视图更新更准确更迅速
* 局部 CSS 最佳解决方案(Shadow DOM)，社区为局部 CSS 折腾了不少框架和库(使用 js 或 json 写样式，如：Radium，jsxstyle，react-style；与 webpack 绑定使用生成独特的 className 文件名—类名—hash值，如：CSS Modules，Vue)，还有运行时注入scoped atrr 的方式，都是 hack 技术；Shadow DOM Style 是最完美的方案
* 对 custom elements 友好，通过字符串 '0'或者'false'传递 false，通过:和Omi.$传递任意复杂类型
* 增强了 CSS，支持 rpx 单位，基于 750 屏幕宽度

该项目里还给出了将近 20 个例子，比如：Omi 写的 MVP 架构的贪吃蛇游戏、Omi 钢琴、用 Markdown 生成静态网站文档等。

图片

##### 开源项目地址：

> * https://github.com/Tencent/omi

来源：TJ君

```
```
简单、漂亮、容易上手的开源 SAAS 多租户快速开发平台，已开源

你见过哪些目瞪口呆的 Java 代码技巧？

从3s到25ms！看看京东的接口优化技巧，确实很优雅！！

12个超好用的免费在线工具，大大提高生产力，建议收藏！

```
最近面试BAT，整理一份面试资料《Java面试BATJ通关手册》，覆盖了Java核心技术、JVM、Java并发、SSM、微服务、数据库、数据结构等等。

获取方式：点“在看”，关注公众号并回复 Java 领取，更多内容陆续奉上。
```

文章有帮助的话，在看，转发吧。

谢谢支持哟
```
```