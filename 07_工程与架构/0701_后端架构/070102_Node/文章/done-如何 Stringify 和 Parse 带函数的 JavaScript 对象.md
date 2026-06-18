> 已吸收至：[[07_工程与架构/0701_后端架构/070102_Node/070102_核心知识点/Node架构实现路线|Node架构实现路线]]
---
title: 如何 Stringify 和 Parse 带函数的 JavaScript 对象
author: 数据可视化 AntV
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg3NTU4OTc3OA==&mid=2247489618&idx=1&sn=91c0794cd705714eea2615c0248a8930&chksm=cf3e61aff849e8b98dbde3137dc3068eb571389a2f3e583c1f5d7ed4792512a376d897bbf7fd&mpshare=1&scene=24&srcid=0803MwhSJDifT5lNQ6TKko1y&sharer_sharetime=1691019688461&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

作者：小小酥梨

将一个 JavaScript 对象转换为字符串可以通过 JSON.stringify 来完成，但是该方法有一个限制：会过滤掉 value 为**函数**的 key，导致没有办法通过 JSON.parse 来获得原始对象。本篇文章就介绍如何解决这个问题：如何 Stringify 和 Parse 带函数的 JavaScript 对象。

**Stringify&Parse**

**背景**

在介绍解决办法之前，首先简单介绍一下该问题的背景，有助于大家了解一个实际的应用场景。

在最新发布的 G2 5.0 中推出了一种全新的 API 形式：Spec API。该 API 是通过一个 JavaScript 对象去声明一个可视化图表。

然后调用 `chart.options(bar)` 去渲染图表，最后得到的可视化效果如下：

该 API 的优点之一就是：图表可以被**持续化存储，**也就是说可以把上面的 `bar` 对象转换为 `String` 保存下来，在需要使用的时候再进行解析。

而因为图表的声明本质上就是一个“带有函数的 JavaScript 对象”，所以就是要解决“如何 Stringify 和 Parse 带函数的 JavaScript 对象” 这个问题。

**JSON.stringify**

**存在问题**

了解了背景，我们通过一个简单的例子来再认识一下存在的问题。以如下的对象 `add` 为例：

当调用 `JSON.stringify`之后发现：`callback`这个字段已经被过滤掉了。

这样 `JSON.parse`就没有办法获得原始的对象。

所以我们需要定义两个新的方法 `stringify` 和 `parse`可以达到如下的效果。

**Stringify&Parse**

**思路**

查阅 MDN 发现 JSON.stringify 和 JSON.parse 都接受第二个参数，都会在返回最后结果之前对该对象的每一个 key 和 value 进行处理。类似一个钩子函数（Hook），可以让我们针对性的定制化 stringify 和 parse 的逻辑。

所以在当前这个场景中，只需要在 stringify 的过程中显示地将函数值转换成可以识别的字符串，然后在 parse 的时候识别该字符串，并且调用 `window.eval`将该字符串转换成函数实例即可。

**Stringify&Parse**

**解决方案**

将函数转换成字符串调用函数的 `function.toString`方法即可，同时为了将转换成字符串的函数和普通的字符串区分开来，我们将前者用标签 `<func>`包裹起来。当让这里标记的方式不是唯一的，只要能和后面的 parse 逻辑匹配即可。

使用该 `stringify`转换上述 add 对象得到如下的结果。可以发现 callback 字段已经正确得被保留下来了！

那么这之后就需要正确地解析该字符串了。这里我们只对可识别的字符串的值进行处理：获取函数代码，并且转换成函数对象实例返回。

最后运行代码，可以发现 `add.callback`可以正常被调用。

**Stringify&Parse**

**小结**

我们通过 `JSON.stringify`和 `JSON.parse` 的第二个参数解决了 Stringify 和 Parse 带函数的 JavaScript 对象问题，也为持续化存储 G2 5.0 Spec 提供了一个思路。

当然对于第二个问题还有别的解决思路：设计一套表达式语法。这可以使得整个 G2 的 Spec 可以完全变成一个 JSON 对象，比如如下。这就要求 G2 5.0 的 runtime 能正确的解析这套语法，这个也是 G2 5.0 未来的工作之一，感兴趣的小伙伴可以参与讨论和共建。

回顾

👉 [我们在可视化中遇到过的 Bad Case](http://mp.weixin.qq.com/s?__biz=Mzg3NTU4OTc3OA==&mid=2247489588&idx=1&sn=ef935cf74be846a8a33466fc5d1febda&chksm=cf3e61c9f849e8df4286125e79414d7d99e53d2af4132a2ac79431d36e01861cc9722bd243c3&scene=21#wechat_redirect)

👉 [用 G2 5.0 轻松绘制 heatmap 热力图](http://mp.weixin.qq.com/s?__biz=Mzg3NTU4OTc3OA==&mid=2247489405&idx=1&sn=65b14482800c9604e96a09f3681da0df&chksm=cf3e6e80f849e7961969996b44fd71e28434940795d020fee17d245b93278ee18d2716622caf&scene=21#wechat_redirect)

👉 [AntV L7 地理可视化 —— 小众但不...](http://mp.weixin.qq.com/s?__biz=Mzg3NTU4OTc3OA==&mid=2247489404&idx=1&sn=397186d90398a6dd2f9228ff8e350e4d&chksm=cf3e6e81f849e7979a23882091a269f2ebf618015e47a64ed78485eec3892855ac24dbb8f680&scene=21#wechat_redirect)

AntV 数据可视化官网

https://antv.antgroup.com/

AntV 数据可视化 GitHub

https://github.com/antvis

✨ 关注我们，了解更多~