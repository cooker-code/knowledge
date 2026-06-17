---
title: n8n的“超级智能门铃”，Webhook节点详解
author: 鲤鱼冲啊
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247485729&idx=1&sn=a2579ba925c6aff45bfd3712a26e32de&chksm=c1bdd8c527aef436a8bdf3d3fa44d344b8b9dccd3bb581985c4df6cf391b40b0f9f23f85ae32&mpshare=1&scene=24&srcid=1114bo3HmqKQuIrZf2X38JfA&sharer_shareinfo=62930caf36e278db705717163f3fd836&sharer_shareinfo_first=62930caf36e278db705717163f3fd836#rd
---

把n8n当成一个住在房子里的自动化管家，不定期有客人到访或快递送到，如果每隔几分钟就去门口看看，那太累了。这时候就轮到Webhook节点登场了，可以把它当成门口的超级智能门铃。普通的门铃只会响，而它可以把访客信息、快递原封不动的直接交给管家。

来看看怎么用吧！

①了解Webhook节点

创建工作流，搜索并添加Webhook节点

参数设置页面如下

先来看看HTTP Method，这里规定了外部应用访问的方法。GET表示“简单看看”（一个访客路过，看了眼你的门牌号），POST表示“提交资料或投递包裹”（访客登记信息，快递投递包裹给管家），PUT意思是替换掉旧数据，PATCH意思是修改一小部分数据，DELETE意思是删除数据。大部分Webhook场景都是使用POST。这里先设置为GET。

Webhook URLs是访问地址（相当于门牌号），这里分为Test（测试环境）和Production（生产/正式环境），选择Production时，需要激活工作流。这里先用Test。

仔细观察，可以发现Path就是Webhook URLs的最后一小截。这里将Path修改为formsubmit（注意：每个URL都必须是唯一的），Webhook URLs也会同步修改。

Authentication认证这里分为4种，默认是None（不需要认证），风险极低的操作，比如收集反馈意见可以用这个；Basic Auth（基本身份认证），外部应用在“按门铃”时，必须同时报上正确的用户名和密码。这是最常用最简单的一种安全措施；Header Auth（标头身份验证），不需要用户名和密码，但需要出示特定证件，这种方式也非常常见。JWT Auth是最复杂也是最安全的访问方式，一般人用不到。

Respond（响应）这个参数决定了n8n管家如何以及何时回复那个前来按门铃的“访客”（外部应用），Immediately（立刻马上），这个是默认选项，当门铃一响，n8n管家立刻对门外的快递员喊一声“收到啦”，然后再慢悠悠的处理包裹，优点是速度极快，缺点是只能回复一个固定信息，不能把工作流处理完之后的结果返回给对方，95%的情况会使用这项；

When Last Node Finishes（等最后一个节点完成后），门铃响后，管家先将包裹拿进来，然后处理包裹里面东西，当所有节点都运行完毕，管家把最后一个节点产生的结果递给等待的快递员，这种只有在需要当面拆封确认签收时才会用到（外部应用必须拿到你的结果才会进行下一步操作，如果耗时过长，可能会失败）；

Using 'Respond to Webhook' Node（使用“响应Webhook”节点），这种会在工作流中间某个位置，放一个专门的Respond to Webhook节点，当工作流运行到这个节点时，由这个节点回复快递员，这样可以尽快回复外部应用需要的关键信息，然后再继续处理其他耗时任务；

Streamin（流式传输），这是一种非常特殊的模式，管家不是一次性回复，而是每处理完一小步，就立刻告诉快递员一点结果（打字机逐字输出效果），小白几乎用不到这种方式。

Options这里有多个选项，一般不需要设置。

来看下，Allowed Origins (CORS)（允许的来源），规定了只有从哪些网站发来的请求，你的Webhook才接收；Field NameforBinaryData（二进制数据的字段名），发送方固定了文件字段名称，而你希望用另外一个名字来接收文件（比如图片、PDF）时才会用到；Ignore Bots（忽略机器人），可以尝试识别并忽略掉一些常见的网络爬虫请求；IP(s) Whitelist （IP地址白名单），只有指定IP发送的请求才会被接收；NoResponseBody（不要返回任何内容），只回复状态码，回复正文是空的；RawBody（接收原始数据），当收到的数据不是标准的JSON或表单格式时，开启这个，然后用后续节点，比如Code来手动解析；ResponseCode（自定义回复代码），只有当Respond响应模式是Immediately时才会生效，可以修改默认返回内容；ResponseHeaders（自定义回复标头），可以在回复中加入自定义的“证件”（Header），只有在对方系统要求你的回复必须包含特定的Header时才需要设置。

②模拟GET请求

参考如下设置

左侧点击Listen for event（开启网址监测）

复制上面网址到浏览器打开，可以看到如下提示，Workflow was started

返回节点，可以看到访问来源（访客）信息（访客用的是什么浏览器？操作系统？语言？等）

可以在网址后面加上参数，比如加上?name=zhangsan，再次访问看下，可以看到数据被传递进来了（查询name=zhangsan），但是这种通过网址传递数据的方法并不安全，只有在临时测试或者需要查询数据时会这么做，一般情况都是用POST传递。

接下来把HTTP Method改为POST，这时候想测试的话需要借助模拟请求工具，比如Postman、Apifox等，太重型了，所以接下来找个轻量化工具，打开网页就能用。

③模拟POST

先打开tally，https://tally.so，可以快速创建表单，并且发送请求。

登录后点击New form创建新表格

选择Use a tempate使用模板

随便找一个简单的，我这里用Contact form联系表格

选择Use this template使用此模板

可以修改模板，比如把Phone删了，然后点Publish发布

然后选择Integrations集成，选择Webhooks下面的Connect

填入n8n的Webhook节点Test网址，点击Save

绿色代表连接上了，右侧可以控制是否连接。

切换到Share，复制tally的表单域名

在浏览器打开，填入内容，并点击Submit提交（注意先把n8n的Listen监听打开）

提交后会返回Webook节点的默认回复。

来看看n8n这边Webhook的数据，可以看到，已经接收到了。

在n8n中添加搜索webhook节点时，还会出现一个Pespond to Webhook节点，这个节点该怎么使用呢？点赞收藏关注起来，我们下期再聊。