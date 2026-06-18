> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020204_ToolCalling/020204_核心知识点/工具调用治理与协议边界|工具调用治理与协议边界]]
---
title: Vercel 又开源了个好玩的东西——给 AI 用的 Bash
author: 开源技术研习社
date:
url: https://mp.weixin.qq.com/s?chksm=c3234609f454cf1f4a7bfaee64f046a6b1d787d96cd20eb29e513ec97ac2f50cc0f440b157ce&exptype=unsubscribed_card_recommend_article_u2i_mainprocess_coarse_sort_tlfeeds&ranksessionid=1776085520_1&req_id=1776079567656149&mid=2247484247&sn=497020481fb39dbdc75668c2e70ce767&idx=1&__biz=Mzk0NDYwODMwNQ%3D%3D&scene=169&subscene=200&sessionid=1776085518&flutter_pos=9&clicktime=1776085535776&enterid=1776085535776&finder_biz_enter_id=5&key=daf9bdc5abc4e8d02a0cc1647351fff91371a02289edd85183d5ad3102d52c578bb86ffaacf51b9a4978204e08c3dd5f1bdd7db60afbadc3ad0735fe55c2137a4a547c6a05a69e8590f90016764c50ffe4a62cb3f7f9b1aad0d32f1d93eca27ace3c98de21a23cf01b52cfdebccff5bf08772140d7b74ab7af26e970192b6b3f&ascene=0&uin=MjAwNTI2NjEyMw%3D%3D&devicetype=OHOS-22&version=f3801037&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=CN&exportkey=n_ChQIAhIQK%2FRNDSy325ozMEZwDvIpBRLfAQIE97dBBAEAAAAAANsFJeijUZoAAAAOpnltbLcz9gKNyK89dVj0%2FKsjJ7v7EyBSPew%2BLGz1Bh0W%2FqoSSD76WDo%2FISk6cPc3HlKlLj4435sVAAjhgHRdDx9Zp6SZPkpqPf2%2BaAUmVmPpcX87GBtQrdyK%2BIhCVjYclYiPjY%2F%2FOi1ck6Ga1bzTuebhnbaps9RpFZYNerjXLAIiZjVLzK2V3UGqAoi54Zp53MQ626jh9uVCi2TNV5bC%2BOM7mDtUWmhCZXgf107VJw3iFM0GudqM043yIkV5bgbowyEhqs34iUw%3D&pass_ticket=Wm%2B8wY%2FrcGN%2BvuRLtIoJ0s%2FSqsrYZiRpd94S%2BL%2Fo31Fkl8mAXiIMGa0q%2BiNzX%2FW4&wx_header=3
---

最近 Vercel Labs 开源了个挺有意思的项目：just-bash。一个用 TypeScript 写的虚拟 bash 环境，跑在内存里，专门给 AI agent 用。

为什么需要这个？

AI 写代码、跑命令，最大的问题是什么？安全。

你敢让 AI 直接在你的服务器上执行 rm -rf 吗？不敢。

那怎么办？沙箱。

传统的沙箱方案：启动一个 Docker 容器，或者租一台虚拟机。安全是安全了，但成本高、启动慢、资源消耗大。

just-bash 走了另一条路：完全在内存里模拟一个 bash 环境。没有真正的容器，没有虚拟机，就是一个 JavaScript 进程，里面跑着虚拟的文件系统和命令行工具。

它能做什么？

支持大部分常用命令：

* 文件操作：ls、cat、cp、mv、mkdir、rm...
* 文本处理：grep、sed、awk、sort、uniq、jq...
* 甚至还能跑 Python 和 JavaScript（需要单独开启）

网络访问默认是关闭的，想开可以开，但要用白名单限制。

安全吗？

项目自己的定位很诚实：这不是 VM 级别的隔离。

它的威胁模型是：假设代码库能防住原型污染等 JS 层面的攻击，那就可以放心用。但如果你要跑不可信的二进制文件，还是老老实实用 Vercel Sandbox 或者 Docker。

谁会用这个？

* 做 AI agent 开发的：让 agent 在沙箱里试代码，不用担心搞坏系统
* 做代码执行平台的：在线 IDE、playground 之类
* 需要批量处理文本数据的：在 Node.js 里直接用 bash 工具链

一个有趣的细节

项目提供了好几种文件系统实现：

* 纯内存模式（默认）
* Overlay 模式：能读真实文件，但修改只留在内存里
* ReadWrite 模式：真的会写磁盘（慎用）
* Mountable 模式：组合多个文件系统

这个设计挺聪明的，不同场景用不同模式。

怎么用？

|  |
| --- |
| npm install just-bash |

 

|  |
| --- |
| import { Bash } from "just-bash";                    const bash = new Bash();                   await bash.exec('echo "Hello" > greeting.txt');                   const result = await bash.exec("cat greeting.txt");                   console.log(result.stdout); // "Hello\n" |

就这么简单。

值得一试吗？

如果你在做 AI 相关的项目，或者需要一个轻量的命令执行环境，可以看看。

项目还在 beta 阶段，issue 不多，正是提反馈的好时候。

GitHub 地址：github.com/vercel-labs/just-bash