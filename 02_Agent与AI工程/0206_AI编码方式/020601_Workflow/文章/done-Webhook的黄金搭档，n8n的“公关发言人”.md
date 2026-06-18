> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: Webhook的黄金搭档，n8n的“公关发言人”
author: 鲤鱼冲啊
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247485753&idx=1&sn=836d2aa8dd1ae2cbdbe189d8d7d0c775&chksm=c190575557ceb8a8400ed32f4506c39f386abd77d6fb398a70a1fd9be70b2c0d734886b49fb3&mpshare=1&scene=24&srcid=1118zfZ45SDWElKj6E13kb8I&sharer_shareinfo=58ba1fe69a3362f7437f704dba412b02&sharer_shareinfo_first=58ba1fe69a3362f7437f704dba412b02#rd
---

接上期，如果说，Webhook节点是你的“前台接待员”，那么Respond to Webhook节点就是你的“公关发言人”。

没有Respond to Webhook节点时，接待员（Webhook）一收到访客包裹，就立马对访客说“收到了，再见！”（Immediately响应模式），访客走后，才开始处理包裹。

有了Respond to Webhook节点后，你给接待员（Webhook）下了一条指令：“有访客来，你先别回话，把包裹收下就行”（这里需要把Webhook节点的Respond模式修改为Using 'Respond to Webhook' Node）,当流程走到公关发言人（Respond to Webhook）处时，才由“发言人”正式对外回复。

接下来看看实例。

①自定义“谢谢你”页面

先添加好如下节点，Webhook，Set，Respond to Webhook，并连接起来。

Webhook节点按下图设置，HTTP Method选择POST，Respond模式选择Using 'Respond to Webhook' Node

Set节点按下图设置，可以先开启工作流监听，用Tally提交一个表单数据，这样左侧就有测试数据了，Mode选择Manual Mapping（手动映射），然后Fields to Set处选择Add Field，name填入“First name”，value处将左侧的value拖动过来即可。

运行Execute step（可以先把上一节点数据临时固定下），可以看到右侧出现了指定数据

来到第三个Respond to Webhook节点，Respond With选择JSON

Response Body填写

```
{"message": "谢谢你， {{ $json["First name"] }}"}
```

运行整个工作流测试下（记得取消数据固定），用Tally提交表单数据，结果还是出现固定返回信息

原来，是因为我们测试时通过Tally中转的缘故（如果直接用Postman等工具使用原网址，则不会出现这种情况），可以把Tally当成n8n前台的前台，Tally有自己的默认回复设置，这里只要把回复信息同步下就可以了。

在Tally点击左侧的Home

可以找到已发布的表格，点击Edit

可以看到第2页是Thank you页面，

修改为：谢谢你， @First name!，右上角点击Publish发布

再来试下，成功了！

②提供一个“查询API”

场景：当别人访问你的Webhook URL并提供一个商品名称时，你的n8n工作流就会去表格查询价格并返回给对方。

先在n8n创建一个数据表（参考[n8n重大更新：自带数据表，让你的自动化流程快到飞起！](https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247485539&idx=1&sn=1912e7abb1efbd8ed94ff74a9ef5f3e7&scene=21#wechat_redirect)），填入数据如下

创建节点如下，中间一个节点是Data table的Get row(s)

Webhook节点这次改为GET

Get row(s)节点参考如下配置

Respond to Webhook节点参考如下配置，Response Body这里填入

```
{"查询状态": "成功","价格": {{ $node["Get row(s)"].json.price }}}
```

执行工作流，把网址后面加上?name=orange，填入浏览器打开，可以看到提示查询成功，并给出价格

③提交一个“后台任务”并返回任务ID

场景：一个外部请求，让你的n8n去处理一个很耗时的任务（比如生成视频），这时候可以先立即回复一个“任务受理成功”的消息，并附上任务ID，让对方后续可以根据这个ID来查询进度。

这里只是提供一个思路，Webhook用POST，Respond to Webhook的JSON可以这样写

```
{"message": "视频生成中",  "taskId": "{{ $json.taskId }}"}
```

再用其他节点开始真正、耗时的生成视频。好处是实现了异步处理，用户体验佳。

好了，今天的内容就到这里，觉得这个“金牌公关”能让你的自动化流程“逼格”拉满的话，点赞、收藏、关注，这波“出场费”可不能少！还有更多好玩的内容，我们下期见。