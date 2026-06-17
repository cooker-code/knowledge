---
title: n8n里如何循环指定次数？
author: RPA小站
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI4OTc4MDY3Nw==&mid=2247485754&idx=1&sn=f7ed4f3ea946102296d9d2b8a4370d5d&chksm=edf3583e90e60d5593529a2ae7b401ab2a1f0667664841d8df4e3db171ba956b7446481fcaa3&mpshare=1&scene=24&srcid=1106LhPLGuVom66M1I4YgbJn&sharer_shareinfo=9fd03b6c4eff31f4055b31e5902aab14&sharer_shareinfo_first=9fd03b6c4eff31f4055b31e5902aab14#rd
---

相信最近大家对n8n并不陌生了

这个开源的工作流自动化工具，支持本地部署，无需编程即可构建复杂自动化流程，大幅提升工作效率。

今天我来开始记录一些开发小技巧：如何在n8n里实现指定次数的循环？

下面是最基本的Loop Over Items循环控件

添加code代码，使其可以循环3次：

code里代码如下：

```
return Array(3).fill({})
```

那么，如何循环指定数据？

把code里的代码换成你要循环的数据即可：

示例代码如下：

```
const websites = [  
  { name: "Google", url: "https://www.google.com", category: "搜索引擎" },  
  { name: "GitHub", url: "https://github.com", category: "代码托管" },  
  { name: "Stack Overflow", url: "https://stackoverflow.com", category: "技术问答" },  
  { name: "MDN", url: "https://developer.mozilla.org", category: "开发文档" },  
  { name: "YouTube", url: "https://www.youtube.com", category: "视频平台" }  
];  
  
return websites.map(site => ({  
  json: site  
}));
```

下期继续更新