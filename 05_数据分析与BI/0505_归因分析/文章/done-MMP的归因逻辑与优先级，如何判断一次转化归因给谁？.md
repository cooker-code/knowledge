> 已吸收至：[[05_数据分析与BI/0505_归因分析/0505_核心知识点/广告归因与投放链路|广告归因与投放链路]]
---
title: MMP的归因逻辑与优先级，如何判断一次转化归因给谁？
author: 非线性增长-AgentGrowing
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzMjE1MzUxNw==&mid=2247483834&idx=1&sn=cac94a4b3031be36f66ecb34e3529e5b&chksm=fbaec28a147f4de1eeb3e7687e32795011abf548db098f4bde4cb6a16289ca4f558c72f8fb0c&mpshare=1&scene=24&srcid=0807TmpbuPFFFqoKYwxfwGeC&sharer_shareinfo=061d9ddad248c56ef7b839351a114f20&sharer_shareinfo_first=061d9ddad248c56ef7b839351a114f20#rd
---

**今天我们来聊一路MMP 是如何“判断归因”的，内容包括自归因渠道（SRN）与非自归因渠道的归因机制与多种归因方式的优先级排序**

**其实像Adjust、Appsflyer这种MMP 的归因系统，本质是一个参考了**多个维度的优先级匹配引擎**，根据用户的点击、展示、互动行为的时间和特征，逐步匹配并判断出最可信的来源。**

**MMP的判断流程是这样的：**

**对于非自归因渠道，例如 applovin 、unity 、 Mintegral等：**

1. **首先，用户完成一个目标行为（安装、first open等），触发了MMP的SDK，并上报给服务器**
2. **MMP 回溯广告点击与展示记录（通过click ID、IDFA、GAID等）**
3. **按照**预设优先级规则**进行匹配，判断归因结果**

**对于自归因渠道 SRN，例如Facebook、Google、ASA等：**

1. **这些自归因平台不会实时将广告点击数据传给 MMP，而是会在用户安装并首次打开APP时，**MMP 采集主动向这些平台发送设备标识（如 IDFA）询问“这个用户是不是你带来的呀？”****
2. ****然后平台在收到设备标识后进行内部比对，若命中并且在时间窗内，即告知 MMP“这用户是我的（这里之前提到的FB加入Engaged View逻辑后，就会将last click和Engaged View打包作为**last-touch一起回传**）无形之中归因范围扩大了一波****
3. ****如果多个平台都命中，则由 MMP 再基于最后点击时间、lookback window 决定归因顺序****

这些SAN 渠道的归因结果具有较强控制权，部分逻辑是“黑盒式”的。有时归因优先级由平台本身决定，不完全由 MMP 控制。

**那么接下来我们来看看MMP的归因优先级，下图是AF判定优先级的排序**

#### **权重由上到下依次是：**

####

* #### **Preload就是**预装：App预装在设备上，优先级最高，不需要用户行为。

  ####
* #### **Deterministic Engaged Click**（确定性活跃点击）用户**点击广告后进行了互动行为**，如安装、打开、注册等，且有明确ID匹配（如IDFA/GAID等）。精准度极高，是归因的首要参考。
* #### **Deterministic Click-through**（确定性点击）用户点击广告，但不一定互动，仅有点击记录，且能通过ID匹配。没有上一个“engaged”的优先级高。
* #### **Deterministic View-through**（确定性展示归因）用户**看到广告但没有点击**，之后安装了App，通过IDFA等能匹配到。

  ####
* #### **Probabilistic Engaged Click**（概率性活跃点击）：用户有“点击+互动”行为，但没有ID可用，通过设备指纹等技术进行匹配，精度稍低。
* #### **Probabilistic Click-through**（概率性点击）用户点击广告，但归因是通过设备指纹推测的。
* Probabilistic Engaged View（概率性活跃展示归因）用户**看过广告但没有点击**，并进行了某种“互动”行为（如观看一定时长、点击展开广告等），**之后转化，但由于缺乏设备ID（IDFA/GAID等），Appsflyer只能通过****设备指纹等概率模型**进行归因。

  ####
* #### **Deterministic View-through**（确定性展示归因），是说用户**看到广告但没有点击**，之后安装了App，通过IDFA等能匹配到。
* #### **Probabilistic View-through**（概率性展示归因）这个权重最低，因为没有ID，仅通过指纹或其他算法“猜测”用户看过广告后转化的情况。

其实并不复杂，所有 MMP 的归因优先级本质遵循同一个核心原则，即：ID确定性 > 用户行为深度 > 行为类型（点击 > 展示）

**如果觉得本篇内容对大家有帮助，请帮忙点点下方的赞、转发和推荐，谢谢大家~ 有问题也欢迎留言讨论**