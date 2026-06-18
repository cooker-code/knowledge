> 已吸收至：[[05_数据分析与BI/0505_归因分析/0505_核心知识点/广告归因与投放链路|广告归因与投放链路]]
---
title: 广告投放中的Cookie Mapping
author: 早起的码农
date:
url: https://mp.weixin.qq.com/s?__biz=MzI2ODYzMjkwMg==&mid=2247485186&idx=1&sn=159d6a7286b927bf8967a856b20cc3e9&chksm=eb31d05e371c65dc380d4f718a895b08430339173f3a41280c2f16a58762d9422b05a3c74bf8&mpshare=1&scene=24&srcid=0610qwNwjHRUIzY7IVi76PsI&sharer_shareinfo=ade031eec827d5197a031db87dc71165&sharer_shareinfo_first=ade031eec827d5197a031db87dc71165#rd
---

了解Cookie Mapping之前让我们先了解以下什么是Cookie。

## **01 什么是Cookie**

**HTTP cookie**，简称**cookie**，又称**数码存根**、“**网站／浏览＋魔饼／魔片**”等，是浏览网站时由网络服务器创建并由网页浏览器存放在用户计算机或其他设备的小文本文件。Cookie使Web服务器能在用户的设备存储状态信息（如添加到在线商店购物车中的商品）或跟踪用户的浏览活动（如点击特定按钮、登录或记录历史）。

第一方cookie 

指的是由网络用户访问的域创建的cookie。例如：当用户通过网络浏览器点击我博客xx.com 时，浏览器会在第一个页面中发送一个网页请求，这个过程需要用户直接与 xx.com 互动。这样网络浏览器随后就将此数据文件保存到“xx.com”域名下的用户计算机上。

大多数网页浏览器都支持第一方 cookies。

第三方cookie

是建立在别的域名不是你访问的域名（地址栏中的网址），比如：广告网络商就是最常见的第三方 cookies 的来源，他们用它们在多个网站上追踪用户的行为，当然这些活动可以用来调整广告。此外图像、 JavaScript 和 iframe 通常也会导致第三方 cookies 的生成。

虽然第三方Cookie是数字广告的基石，但用户不会像第一方cookie那样对第三方 cookies 那么友好了。GDPR、CCPA 等数据保护法规加强了对用户隐私的保护，许多人认为这是对他们隐私的侵犯，是对他们数字安全的威胁。浏览器厂商已经开始主导禁用第三方cookie, 例如，Safari（Apple）和 Firefox 已经默认屏蔽了第三方 Cookie。Google Chrome（市场份额最大）宣布将在 2025 年中全面停止支持第三方 Cookie（原计划是2024年，但已推迟）。

## **02 为什么要做Cookie Mapping**

在程序化广告系统中，不同平台（如 ADX、DSP、DMP）都只能在自己域下设置 cookie，所以**彼此之间看不到对方给用户打的 cookie ID**，而 Cookie Mapping 就是**打通这些用户 ID**的桥梁。大多数Cookie Mapping只是针对网站的，APP上的设备ID是唯一的，不需要通过映射或匹配去是识别用户。ADX（广告交易市场）：接到广告请求；DSP（需求方平台）：决定给谁投什么广告；用户访问页面，页面里加载了 ADX 的广告代码。

Cookie Mapping主要解决以下问题：

|  |  |
| --- | --- |
| 🎯 **用户定向投放** | DSP 必须识别用户，才能做兴趣定向、地域定向、retargeting 等 |

|  |  |
| --- | --- |
| 💰 **竞价决策** | DSP 是否出价、出多少钱，都依赖于是否识别出用户身份 |

|  |  |
| --- | --- |
| 🔄 **频次控制 / 去重** | 没有映射，DSP 无法知道这个用户是否已经投过广告 |

|  |  |
| --- | --- |
| 📊 **转化追踪** | 后期 attribution（归因）也依赖 Cookie ID 的统一 |

|  |  |
| --- | --- |
| 🤝 **多平台合作** | 数据管理平台（DMP）、CDP 也需要 ID mapping 来打通用户数据 |

从技术方面来看，各平台的Cookie彼此隔离（DSP无法直接读取ADX的Cookie），必须通过映射建立关联。每个平台生成独立的用户ID（如ADX用`adx_id`，DSP用`dsp_id`），需映射才能对齐。隐私合规要求，直接共享原始用户信息（如邮箱）违法，通过加密ID映射可合规实现数据协作

在广告投放的**Cookie Mapping（CM）**流程中，**ADX（广告交易平台）和DSP（需求方平台）**都可以作为发起方。

## **03 ADX发起Cookie Mapping**

这种模式Cookie Match Server在DSP端，Match Table在Adx端。Match Table没有匹配到且发起竞价请求的情况才会有Cookie  Mapping。

**流程：**

1. 用户访问媒体网站 → SSP 向 ADX 发送广告请求（携带`ssp_user_id`）
2. ADX 返回响应，**嵌入 DSP 的同步像素**（触发 DSP 写 Cookie）：

   <img src="https://dsp.com/sync?adx\_id=123&redirect=https://adx.com/match?dsp\_id=456"/>

 3. 用户浏览器加载 DSP 的像素：

* **DSP 在自己的域名下写入`dsp_user=456`**
* **`通过重定向将`**`dsp_id=456`回传给 ADX

4. ADX 存储`adx_id=123—dsp_id=456`的映射关系（CM Table）

5. 后续竞价时，ADX 在 RTB 请求中携带`adx_id=123`，DSP 通过 CM Table 匹配用户

**主流方式是**CM表ADX 存储****：ADX 维护全局映射表，供 DSP 查询。由于DSP将matched table托管给ADX，ADX将在request中直接发送映射后的cookie，交由DSP直接定向使用，能够节省DSP的检索时长，缩短响应时间。

## **04 DSP发起Cookie Mapping**

这种模式Cookie Match Server在ADX端，Match Table在DSP端。DSP从Adx接到的信息里面会有Adx userId，这个是AdX专门用于Cookie Mapping的一个ID，如果DSP的系统里面找不到这个ID，那么就需要发起Cookie Mapping。

1. DSP 检测到未知用户（如用户访问广告主落地页）
2. DSP**嵌入 ADX 的同步像素**：

   <img src="https://adx.com/sync?dsp\_id=789&redirect=https://dsp.com/match?adx\_id=101" />
3. ADX 在自己的cookie域名下写入`adx_user=101,` 返回自己的 Cookie（adx\_user=101）并重定向回 DSP
4. DSP 存储dsp\_id=789—adx\_id=101的映射关系
5. 后续 ADX 在竞价请求中发送adx\_id=101，DSP 可识别用户

**主流方式是CM表由DSP 存储，DSP 维护自己的映射表，不共享给 ADX。**

**Cookie Mapping的方式有好多种，不管哪种都是为打通媒体和多个广告投放组件之间的用户识别体系，让广告投放组件能识别同一用户，精准投放。随着第三方Cookie被淘汰，广告行业已经进入一个“后 Cookie 时代”。传统的 **Cookie Mapping**（基于第三方 cookie 的跨平台用户身份映射）逐渐失效，可以尝试用新的**替代技术和方法来完成用户的识别和ID同步：**统一身份标识（Universal ID）方案;浏览器端新机制（Google Privacy Sandbox）;第一方数据 + Server-to-Server（S2S）身份同步;隐私计算 & Clean Room 技术。**