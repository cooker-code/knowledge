---
title: AI爬虫秘籍：打造高效llms.txt文件
author: 蜡笔小豆互联网
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYzNzE5OTcwMQ==&mid=2247483854&idx=1&sn=380b91acdd0b14ba9a4dc458328b35ec&chksm=f11f7a19f85cff0d3ab875cc01390686c9b04f69e387cd20327ce25a86a31ecea157d7daf5d1&mpshare=1&scene=24&srcid=1128MD4ljXLjwlaCp3bdajYU&sharer_shareinfo=d64fe1177fb5d49485648ab5d364befe&sharer_shareinfo_first=d64fe1177fb5d49485648ab5d364befe#rd
---

### 在AI技术飞速发展的今天，AI爬虫已成为获取和处理网络信息的重要工具。而llms.txt文件，作为一种专为AI爬虫和大型语言模型设计的“自述文件”新标准，显得尤为重要。本文将深入探讨llms.txt文件的用途和编写方法，帮助您创建高效的llms.txt文件，让您的网站内容更好地被AI理解和利用。

#### llms.txt文件是什么？

llms.txt文件类似于网站中的robots.txt，专门用于告知大型语言模型相关站点的使用协议和内容属性。它是一个Markdown文件，放在网站根目录下，通过提供结构化的内容和导航来增强AI的互动能力。llms.txt文件的主要作用包括提高AI理解准确性、增强AI回答质量、内容保护、版权控制、质量管控和商业策略实现。

#### 如何编写llms.txt文件？

1. **文件位置**：将llms.txt文件放置于网站根目录下，使其容易被AI爬虫发现。
2. **文件格式**：采用Markdown格式编写，确保结构清晰、易于阅读。
3. **内容结构**：包括但不限于以下几个部分：

* **站点介绍**：简要介绍网站的主题和内容。
* **内容概览**：概述网站的主要部分和内容类型。
* **导航链接**：提供网站重要页面的链接，方便AI爬虫快速定位。
* **版权声明**：明确内容的使用权限和版权信息。
* **更新频率**：告知内容的更新频率，帮助AI爬虫合理安排抓取时间。

4. **内容保护**：通过llms.txt文件，可以防止敏感或专有内容被AI系统未经授权学习使用。
5. **版权控制**：明确哪些内容可以合法用于AI训练，保护网站内容的合法权益。
6. **质量管控**：引导AI系统优先使用高质量内容，提高AI处理信息的准确性。
7. **商业策略**：通过选择性开放内容实现差异化竞争，提升网站在AI时代的竞争力。

**基本格式：**

# llms.txt - 针对大型语言模型的指导文件

User-agent: GPTBot

Allow: /public/

Disallow: /private/

Disallow: /admin/

User-agent: ChatGPT-User

Allow: /blog/

Allow: /articles/

Disallow: /user/

Disallow: /api/

User-agent: CCBot

Allow: /

Disallow: /private/

User-agent: \*

Allow: /public-content/

Disallow: /sensitive/

## 关键指令说明

* User-agent: 指定目标爬虫（GPTBot, ChatGPT-User, CCBot, Google-Extended等）
* Allow: 允许爬取的路径
* Disallow: 禁止爬取的路径
* Crawl-delay: 爬取延迟（秒）
* Contact: 网站管理员联系方式
* Policy: AI使用政策链接
* **Priority**: 内容优先级标记

#### 总结

llms.txt文件是AI时代下网站与AI爬虫、大型语言模型沟通的桥梁。通过精心编写llms.txt文件，我们可以让AI更准确地理解网站内容，提高AI的回答质量，同时也保护网站的内容安全和合法权益。希望本文能为您提供实用的指导，帮助您打造高效的llms.txt文件，让您的网站在AI时代中脱颖而出。