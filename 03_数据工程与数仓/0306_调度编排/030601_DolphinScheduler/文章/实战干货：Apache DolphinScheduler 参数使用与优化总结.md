---
title: 实战干货：Apache DolphinScheduler 参数使用与优化总结
author: 海豚调度
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247524132&idx=1&sn=bbca9457b018859101b79c283d351ce9&chksm=c1d4affb4dcc2f0bb585b33171a5014c4d8e235459604b927a6d4272b3dd2d15f0b3f2a69a9f&mpshare=1&scene=24&srcid=1112jepDMn3fTMWirnKtROuE&sharer_shareinfo=c4783f1a5af0bbcab77df0adfa024de4&sharer_shareinfo_first=c4783f1a5af0bbcab77df0adfa024de4#rd
---

**点击蓝字 关注我们**

在使用 DolphinScheduler 进行数据调度开发的过程中，**参数的灵活运用**是提升任务复用性、动态化执行逻辑的关键。无论是日常跑批任务中的日期处理，还是复杂工作流中的上下游传参，掌握参数的正确使用方式，能极大提高开发效率与任务健壮性。

本文基于**海豚调度3.1.9版本**将系统梳理 DolphinScheduler 中各类参数的使用方法，涵盖**内置参数、衍生函数、日期计算、全局变量设置以及跨任务参数传递**等核心场景，助你彻底掌握参数使用的“正确姿势”。

**内置**

参数

DolphinScheduler 提供了几个常用的系统级内置参数，主要用于获取调度实例的执行时间。这些参数无需定义，直接引用即可。

### 基础参数

| 参数名 | 声明方式 | 含义 |
| --- | --- | --- |
| system.biz.date | ${system.biz.date} | 日常调度实例定时的定时时间前一天，格式为 yyyyMMdd |
| system.biz.curdate | ${system.biz.curdate} | 日常调度实例定时的定时时间，格式为 yyyyMMdd |
| system.datetime | ${system.datetime} | 日常调度实例定时的定时时间，格式为 yyyyMMddHHmmss |

* SHELL 案例

但是这种方式在SQL节点下 不适用，**SQL下使用衍生内置函数会比较方便**

## 

**衍生**

内置参数

## 

为解决 SQL 节点无法使用 `${}` 参数的问题，DolphinScheduler 提供了强大的 **`$[...]` 衍生函数语法**，支持任意格式的日期拼接与运算，**推荐在所有场景中优先使用**。

```
我们定义这种基准参数为 $[...] 格式的，$[yyyyMMddHHmmss] 是可以任意分解组合的，比如：$[yyyyMMdd], $[HHmmss], $[yyyy-MM-dd] 等
```

### 简单案例

* shell案例

结果如下：

  

* SQL案例

结果如下

### 日期月份增减变化

若需按月计算（考虑大小月、闰年），可使用 `add_months()` 函数

* shell 案例，获取前一个月日期

执行结果如下

* PG案例，下一个月

执行结果如下

### 其他时期增减

直接加减数字 在自定义格式后直接“+/-”数字

```
后 N 周：$[yyyyMMdd+7*N]  
前 N 周：$[yyyyMMdd-7*N]  
后 N 天：$[yyyyMMdd+N]  
前 N 天：$[yyyyMMdd-N]  
后 N 小时：$[HHmmss+N/24]  
前 N 小时：$[HHmmss-N/24]  
后 N 分钟：$[HHmmss+N/24/60]  
前 N 分钟：$[HHmmss-N/24/60]
```

* shell案例

执行结果如下

* SQL案例

执行结果如下：

TIPS ：一般建议使用衍生内置函数，即`$[]` 的方式，比较直观且通用。

## **单个任务节点** 中使用参数

**TIPS： 不要在别名的地方使用自定义参数，会触发BUG**

例如 下图，

**工作流**

全局参数

当多个任务需要使用相同的自定义变量（如环境标识、项目编号等），可通过**工作流全局参数**统一管理。

* 案例

下面三个节点用一个参数

shell案例

```
echo ${my_param}
```

SQL案例

```
SELECT name    
FROM test_datax_hive   
WHERE   
dt = DATE_FORMAT('$[yyyy-MM-dd-1]', 'yyyy-MM-dd')    
and name = ${my_param}
```

python案例

```
print('${my_param}')
```

在每个节点的地方不需要设置参数，只需要在工作流保存的时候设置参数即可。如下图。

## 工作流 参数传递

DolphinScheduler 支持在任务之间传递运行时生成的参数，适用于“上游查询结果 → 下游处理”的场景。

**只有SQL和SHELL任务可以往下传递参数。**

**SQL SHELL PYTHON可以接受上一个节点传递的参数。**

下面通过案例来演示实际操作。

### 1 SQL向下传递参数

在工作流中新建一个SQL节点。如下图。

1. 参数名要始终保持相同，这里使用my\_name作为参数
2. 自定义参数中也使用my\_name作为参数，选择OUT类型，用于往下传递

然后建SHELL类型任务，如下图

参数名保持相同，依旧是OUT类型。

再建一个PYTHON类型任务。如下图

参数名保持相同。

最后，把工作流连起来，如下图

最后看运行日志，来验证

SQL执行结果

SHELL 运行结果，参数被成功赋值

PYTHON运行结果，参数被成功赋值

### 2 SHELL向下传递参数

创建shell脚本，**参数名要一致**。

${setValue(key=value)} 的语句，key 为对应参数的 prop，value 为该参数的值。

在SQL节点接收参数

参数名保持一致

PYTHON任务中接收参数

执行结果

* SHELL

* SQL

* PYTHON

**参数使用**

总结

1. **IN 表示局部参数仅能在当前节点使用**
2. **OUT 表示局部参数可以向下游传递**。
3. 参数优先级：DolphinScheduler 参数的优先级从高到低为：`本地参数 > 上游任务传递的参数 > 全局参数`

**用户案例**

[天翼云](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247523837&idx=1&sn=e86291d08c937f0532eb32cdd89d62cf&scene=21#wechat_redirect)[Zoom](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247523741&idx=1&sn=358e27a66d278b8121ad6488d0496e86&scene=21#wechat_redirect)[网易邮箱](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247523424&idx=1&sn=897add93a8a4f2522dc283f4f29383ef&scene=21#wechat_redirect)

[每日互动](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522464&idx=1&sn=c2be472e02a5efdd8f34bf9fec85b206&scene=21#wechat_redirect)[惠生工程](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522888&idx=1&sn=7268afede2cb6e39585192487dd8c42b&scene=21#wechat_redirect)[作业帮](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522811&idx=1&sn=fd9815e0ec661bcb02c8f23d9f5a4693&scene=21#wechat_redirect)

[博世智驾](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522746&idx=1&sn=123081e566642559acdff0dcdb0d4a2f&scene=21#wechat_redirect) [蔚来汽车](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522480&idx=1&sn=b3bf2d7397cddb239d7e2187d3e6509b&chksm=c0f3c8d7f78441c13a08c9059d6d5511b95db0ad8ad1b1307e2ff6bf1259cc4653d022082796&scene=21#wechat_redirect)[长城汽车](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522240&idx=1&sn=ec3816d5967b51d03503c045ce096f34&chksm=c0f3c7a7f7844eb1df46c560f73ab74bf2cb63d8a1ad3e6c91b730287b49c2ee9acd0c8b7b0a&scene=21#wechat_redirect)

[集度](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522205&idx=1&sn=5bb295374a090a39059ff3f9ceb99d48&chksm=c0f3c7faf7844eecb800cafba9a9fc71e5a702e017cdaa3867002aa7daddb44953a535af5e12&scene=21#wechat_redirect)[长安汽车](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522172&idx=1&sn=a1087e48b6d6bc3cd744e599ca73f0a9&chksm=c0f3c71bf7844e0de2131ac25848d0527201ff8b3c8ca2b618cce9fbef15faf9571572682f88&scene=21#wechat_redirect)[思科网讯](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522089&idx=1&sn=7dbf20157a813882ac6079225372bf94&chksm=c0f3c74ef7844e580eeac890a69fddf7e6920207c123a3ebe28253fc1f97632d985e4835a245&scene=21#wechat_redirect)

食行[生鲜](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247521940&idx=1&sn=4c630d6b098a116d4ef46800309207fb&chksm=c0f3c6f3f7844fe52dbed6b907300fe8b3fa577b3714c2c0a49be1387a10377c92b5dd0509f5&scene=21#wechat_redirect)[联通医疗](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522050&idx=1&sn=f453aaf00d16a7d2dbeb6beaa8292597&chksm=c0f3c765f7844e731d33714059b2267aed730052b0f1462a2f97c83868894e9714c78a425320&scene=21#wechat_redirect)[联想](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522084&idx=1&sn=5579a2b3ce1238aa68735517fad6d939&chksm=c0f3c743f7844e55bda58f61c48378141cd87fed5127525fb707776addad1b5d4e46da3ffb54&scene=21#wechat_redirect)

[新网银行](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522033&idx=1&sn=a02f1db461bf2ee7df271b34e484a811&chksm=c0f3c696f7844f80f98793aee9f6efcd825ff998a43b6b3b777ab3711814e1b9601f7498987f&scene=21#wechat_redirect)唯品富邦[消费金融](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522218&idx=1&sn=ef94406e939a9a745b55cc738837f962&chksm=c0f3c7cdf7844edb18500ef088173f380f9d54b87edf3c4778329a4107e87e5c8cfdab315ea4&scene=21#wechat_redirect)

[自如](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522014&idx=1&sn=3308ce7bca93b2035d52d3640a38834d&chksm=c0f3c6b9f7844faf3dc774585ca16b89e8b52bf1dff20c6954edd2601437550a8b042465ee0a&scene=21#wechat_redirect)[有赞](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247521946&idx=1&sn=c205f5c8adecadada22c09abac0841c1&chksm=c0f3c6fdf7844febf7e3962c482c44dc4570e225fa9d0db4da589a4464ccd09d7d1bd29f9c9b&scene=21#wechat_redirect)[伊利](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522088&idx=1&sn=986c7050c428a7bc9717bbdab5a4f63f&chksm=c0f3c74ff7844e590a1eba962b504ff44dfebdc85441c6cca3566b5ee88f3ea28cd39b39c393&scene=21#wechat_redirect)[当贝大数据](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522042&idx=1&sn=e2e6a89387d58781893d83fbc6181707&chksm=c0f3c69df7844f8bd8c8b2ceb1ed4678f5b3f90204846f36d738107e49e9cdec2a8c9b1c6225&scene=21#wechat_redirect)

[珍岛集团](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522317&idx=1&sn=2090fab5cd17248ea555c97d650160f7&chksm=c0f3c86af784417ca9fb1742ca9210dd6610b640f08c8bacb5e1f88dc7513255065e31a38d60&scene=21#wechat_redirect)[传智教育](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522077&idx=1&sn=2590dcbf4ddffbb04801616d79b1b647&chksm=c0f3c77af7844e6c4c90fd8f7af997b4f2ee015d209b599043472cbaa0b6e9444b631bc203b5&scene=21#wechat_redirect)[Bigo](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247521942&idx=1&sn=bf16ca2c69ea4663e03779a145ffc9fe&chksm=c0f3c6f1f7844fe700ed91c4a4218a0232e31206ae74f748852dde32db2c08cd9bf68ee440e4&scene=21#wechat_redirect)

[YY直播](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522016&idx=1&sn=7850b746a21139d2d1b1e2b8d4be967c&chksm=c0f3c687f7844f914f18a6c41dbc5750e79326bddf006e79ff94221ba96974e121cac0564d69&scene=21#wechat_redirect)  [拈花云科](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522248&idx=1&sn=1bc58df2ca858b03c7be054bcd77b374&scene=21#wechat_redirect)[太美医疗](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522362&idx=1&sn=2af194c3bc2dfc03f17b08ea99e8cdea&chksm=c0f3c85df784414b5015a37346546d841138434d75252467c629009737b0eb0a2483aeb610e3&scene=21#wechat_redirect)

[Cisco Webex](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522442&idx=1&sn=c6fc64973008fd84898496b1e9a7a3eb&scene=21#wechat_redirect)[兴业证券](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522438&idx=1&sn=3c3d71e4788c1fdf75b8d903a8e36774&scene=21#wechat_redirect)

**迁移实战**

[Azkaban](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522046&idx=1&sn=e48aa66bab5e784958b4fa1e716e0f13&chksm=c0f3c699f7844f8f253b69da14be80d68c9feccfe927fcfd0e6a11c756669186b73aa8555df4&scene=21#wechat_redirect)   [Ooize](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522042&idx=1&sn=e2e6a89387d58781893d83fbc6181707&chksm=c0f3c69df7844f8bd8c8b2ceb1ed4678f5b3f90204846f36d738107e49e9cdec2a8c9b1c6225&scene=21#wechat_redirect)（当贝迁移案例）

[Airflow （有赞迁移案例）](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522478&idx=1&sn=d519310eecff8c78d3ffb9e67b5cb20b&chksm=c0f3c8c9f78441dfe238fd70560184ddbdc5b1bb1296935a44d655d9d4f2491442fd16609190&scene=21#wechat_redirect)

[Air2phin（迁移工具）](http://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522168&idx=1&sn=82a260336451e9c633b60f0f5fa77b79&chksm=c0f3c71ff7844e0951efa6116ce9dc9a5cd288a7d9046643642e6446e4bc0d9065cd56c0e592&scene=21#wechat_redirect)

[Airflow迁移](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522321&idx=1&sn=4f1cb9104c7c7247376a45fb319abced&source=41&scene=21#wechat_redirect)[实践](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522321&idx=1&sn=4f1cb9104c7c7247376a45fb319abced&source=41&scene=21#wechat_redirect)

**发版消息**

[Apache DolphinScheduler 3.2.2版本正式发布！](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522421&idx=1&sn=2cf5d31d5182198f872c4c1e70f48bc2&scene=21#wechat_redirect)

[Apache DolphinScheduler 3.2.1 版本发布：增强功能与安全性的全面升级](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247522347&idx=1&sn=89c72d5c7510824674154a47480075fe&scene=21#wechat_redirect)

[Apache DolphinScheduler 3.3.0 Alpha发布，功能增强与性能优化大升级！](https://mp.weixin.qq.com/s?__biz=MzkwNTg3MzY3Mg==&mid=2247523457&idx=1&sn=17c3bf447bd6ff40fefb93bf9d6abddf&scene=21#wechat_redirect)

**加入社区**

关注社区的方式有很多：

* **GitHub:** https://github.com/apache/dolphinscheduler
* **官网**：https://dolphinscheduler.apache.org/en-us
* **订阅开发者邮件**：dev@dolphinscheduler@apache.org（向邮箱发送任意内容，收到邮件后回复同意订阅即可）
* **X.com**：@DolphinSchedule
* **YouTube**：https://www.youtube.com/@apachedolphinscheduler
* **Slack**：https://join.slack.com/t/asf-dolphinscheduler/shared\_invite/zt-1cmrxsio1-nJHxRJa44jfkrNL\_Nsy9Qg

同样地，参与Apache DolphinScheduler 有非常多的参与贡献的方式，主要分为代码方式和非代码方式两种。

📂非代码方式包括：

完善文档、翻译文档；翻译技术性、实践性文章；投稿实践性、原理性文章；成为布道师；社区管理、答疑；会议分享；测试反馈；用户反馈等。

👩‍💻代码方式包括：

查找Bug；编写修复代码；开发新功能；提交代码贡献；参与代码审查等。

贡献第一个PR(文档、代码) 我们也希望是简单的，第一个PR用于熟悉提交的流程和社区协作以及感受社区的友好度。

**社区汇总了以下适合新手的问题列表**：https://github.com/apache/dolphinscheduler/pulls?q=is%3Apr+is%3Aopen+label%3A%22first+time+contributor%22

**优先级问题列表**：https://github.com/apache/dolphinscheduler/pulls?q=is%3Apr+is%3Aopen+label%3Apriority%3Ahigh

**如何参与贡献链接**：https://dolphinscheduler.apache.org/zh-cn/docs/3.2.2/%E8%B4%A1%E7%8C%AE%E6%8C%87%E5%8D%97\_menu/%E5%A6%82%E4%BD%95%E5%8F%82%E4%B8%8E\_menu

如果你❤️小海豚，就来为我点亮Star吧！

https://github.com/apache/dolphinscheduler

**你的好友秀秀子拍了拍你**

**并请你帮她点一下“分享”**