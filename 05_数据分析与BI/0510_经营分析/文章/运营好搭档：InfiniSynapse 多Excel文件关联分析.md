---
title: 运营好搭档：InfiniSynapse 多Excel文件关联分析
author: 祝威廉
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIyNzQyNzgxNQ==&mid=2247484994&idx=1&sn=dd79e4b734e250a53cf5e83d94ef8b97&chksm=e94b0e89b8de738dadf6bc37da03c6629a1ec387d9677278f65e6cf350206a5c53c0beea2974&mpshare=1&scene=24&srcid=1108N3llAaq1YT4YoOE3SJUr&sharer_shareinfo=ec82ac268ae68ca0bd778fb2a6a1b49d&sharer_shareinfo_first=ec82ac268ae68ca0bd778fb2a6a1b49d#rd
---

前言

昨天正式发了一篇文章 [开拓大模型的第二个杀手级应用场景：InfiniSynapse 正式发布](https://mp.weixin.qq.com/s?__biz=MzIyNzQyNzgxNQ==&mid=2247484979&idx=1&sn=43d016b08194c278ee36ba24f5b0aca2&scene=21#wechat_redirect) ,然后有人吐槽：

实际上我们官网的文档链接还是给了很多复杂案例的。不过今天我们来个简单的，方便大家理解产品。

我们知道运营的同学的数据来源主要包含两部分：

1. 公司的运营后台或者数据分析师

2. 各种Excel 数据、比如从第三方平台导出

如果希望把公司的数据和第三方的数据都进行关联统计分析，我知道很多运营同学都是手算的，非常麻烦。InfiniSynapse 支持多Excel 文件关联分析，所以只需要把各个平台的数据导出成 Excel,就可以关联统计了以及生成一些图表。

小案例展示

通过文件管理，我上传了四个Excel文件（电商数据，用户，产品以及消费记录），然后在数据源管理里，把这些Excel文件取了个名字。

现在，我可以让 InfinSynapse 对这四个Excel文件直接做一个宏观分析：

InfiniSynapse 会自动把 Excel 注册成表（也支持多sheet）：

接着InfiniSypase 在右侧Console 里会做一些任务规划，你可以实时看到规划和进度：

然后就得到了一个宏观概览：

这里，我追问了下这四个 Excel 到底都包含了哪些数据：

然后我提了一个细节需求：分析销售趋势和季节性模式， 最好给我一个可视化的图

这是最后的图表效果：

Bonus

前面演示的 Excel ,实际上也是我通过 InfiniSynapse 生成的，因为我原来手头的数据其实是csv,我只需要说一句话就可以完成转换的工作：

最后结果：

总结

实际上除了多Excel, 我们还能对接各种业务数据库和大数据计算引擎，并且可以对各种数据源做联合计算。运营同学实际上每天都要处理各种Excel统计运营效果，现在，他们可以不再需要手动统计数据，直接给 InfiniSynapse 这个分析师提需求，帮自己做分析了。

InfiniSynapse 提供了 Windows/Mac 客户端，

可以在官网 InfiniSynpase.com 下载使用