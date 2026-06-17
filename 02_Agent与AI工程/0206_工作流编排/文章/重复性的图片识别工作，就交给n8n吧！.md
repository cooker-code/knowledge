---
title: 重复性的图片识别工作，就交给n8n吧！
author: 鲤鱼冲啊
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247486011&idx=1&sn=4d2d8d328f92596a2f87bedfaf3739b2&chksm=c1211ae3537597bd12440d0c91b356a0dc3de6a0bae69f73d9582243fd4fb0489032ab9b5922&mpshare=1&scene=24&srcid=1122DUoHp79ULEBTjEhdQmWk&sharer_shareinfo=e693d6fa8a82343b5fe62f2ec5396a15&sharer_shareinfo_first=e693d6fa8a82343b5fe62f2ec5396a15#rd
---

发票、收据、行驶证、身份证...这些结构化的图片，是我们工作中”最熟悉的陌生人”。里面的信息很重要，但每次都要耗费数十分钟将内容整理到excel表格，费时费力还容易出错。

那么，今天的目标就是用n8n工作流实现自动识别图片中的文字并写入飞书多维表格。

①先说说经历过的坑

A、将图片二进制数据直接传给大模型会消耗海量token，AI Agent节点识别效果优于Basic LLM Chain节点

最开始是听gemini的，说图片不需要转base64，n8n可以直接搞定。采用表单提交图片，用code节点一次性将图片和提示词传给Basic LLM Chain节点，配合gemini-2.5-flash模型，结果是耗费海量的token还识别不准确。

接着尝试将Basic LLM Chain节点更换成AI Agent节点，识别变准确了。

但是！一张97kb图片直接耗费9万token！

一张1.16M图片耗费token超过117万，使用token多的gemini-2.0-flash模型也导致超限报错，实在是太离谱了！

token多的事先放一边，问了下gemini为什么更换成AI Agent节点后识别更准确，给出的解释是可能AI Agent更好的利用了模型内置的优化提示词。

B、用了两款ocr，效果不太理想

最开始想用腾讯云免费的，但后面看到条款，如果服务器不在国内，就会走海外流量，要收费，于是放弃。接着尝试了海外广受好评的ocr.space，每天有不少免费次数，但中文识别能力不行。最后尝试了最近热门的DeepSeek-OCR，硅基流动限时免费，但没找到对应的详细文档，按下图方法，识别效果也不理想。有谁知道如何使用效果好的，欢迎评论区告知下。

C、通过图片链接识别

用了一款免费的图床，将本地图片上传给图床，获取图片的在线链接，然后发送给大模型识别，结果gemini和豆包模型都会超时，只有gpt-4o识别成功，可能是因为这个免费图床最近访问速度变慢导致。

D、图片转base64，再发给大模型识别

以上方法都尝试不理想，那只剩下最后一种，就是先将图片转base64再识别了，让gemini给我方案，这次要求必须使用base64，结果，问题迎刃而解。。。

②最简版本

参考下图，用3个节点将本地图片识别出文字

文字正常识别出来，而且1.17M的图片只消耗了1518token！

第一个节点是form trigger（表格触发器），新版显示为n8n form，创建成功后显示为On form submission

点击上图n8n form后，选择triggers这个

配置参考如下，我这里的字段名称写的是img，后面一个节点的字段名也需要写为img，这样数据才能匹配上。

第二个节点可以搜索extract

找到base64，这个节点的作用是将图片转为base64编码格式

配置参考如下，修改下字段名就可以

第三个节点是http request，搜索http就可以找到，想更多了解的话，可以看下这篇：[n8n的“万能钥匙”！HTTP Request节点，让你连接全世界！](https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247485644&idx=1&sn=0089dce342e7c816d314b82587c2fafa&scene=21#wechat_redirect)

配置如下，这里用到了豆包模型，具体使用方法也可以参考上面这篇

图片识别的话，登录火山引擎后台，选择中间的控制台（注意不要点右上角控制台）

开通管理这里，随便哪个豆包大语言模型都可以，我这里用的是doubao-1-5-vision-pro-32k-250115，算是比较“过时”的版本了，但识别效果仍然不错。

上图右侧可以找到接入文档，找到base64部分

Curl这里就是对应http节点要填入的内容，下图框选处对应JSON部分写法

也就是json这里要填写的内容，可以点击箭头处展开

执行工作流传入图片后，左侧会显示对应数据，然后将对应的字段拖入json中，如果数据正确传入，会显示绿色

附上我这里的json代码，模型不一定要参考我的，可以用最新的Doubao-Seed-1.6相关版本，我这里节点名称修改过

```
{    "model": "doubao-1-5-vision-pro-32k-250115",    "messages": [        {            "role": "user",            "content": [                {                    "type": "image_url",                    "image_url": {                        "url": "data:{{ $('On form submission上传图片').item.json.img.mimetype }};base64,{{ $json.img }}"                    }                },                {                    "type": "text",                    "text": "请按顺序识别出所有文字,包含水印"                }            ]        }    ]}
```

来执行工作流看看效果，点击最底下的Execute workflow

会弹出表单，选择一张图片，比如这张发票（文章这里我做了模糊处理，自行测试的话可以找其他发票来识别）

点submit提交

点开Extract from File节点，可以看到左侧的图片转换成了右边的base64编码格式

点开http节点，可以看到右侧的content出现了对应文字，如果之前http的json处，我没有强调要识别水印，则“测试专用”这4个水印字不会识别出来，我强调后，就识别出来了。（注：这里的\n是换行的意思，可以在后续的code代码节点清洗掉）

接下来加大难度，尝试下识别这张横版的，里面还有手写字体，本身也有些模糊。

因为这张小票没有水印，所以我把提示词里面的要求识别水印去掉

好吧，效果不行

换成更强大的doubao-seed-1-6-vision-250815试试，这次好多了，但还是有一些识别错的，比如“黑糖碧根果仁”识别成“黑糖鮑勃滑溜溜”。

接下来换成gemini-2.5-flash，这次基本准确了，gemini确实牛！

关于gemini的配置可以参考这篇文档（https://ai.google.dev/gemini-api/docs/image-understanding?hl=zh-cn）

具体在n8n的http节点参考如下，gemini权限配置参考这篇：[Google AI Studio，让你拥有强大且免费的Gemini火力](https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247485387&idx=1&sn=19112b8f1b1904bc964f3aad18867090&scene=21#wechat_redirect)和这篇：[n8n中将数据写入Google Sheets，权限该如何配置？](https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247485279&idx=1&sn=95456219ec3449e12c55b97ec36076f8&scene=21#wechat_redirect)

json部分代码参考如下，我这里节点名称修改过

```
{  "contents": [    {      "parts": [        {          "inline_data": {            "mime_type": "{{ $('On form submission上传图片').item.json.img.mimetype }}",            "data": "{{ $json.img }}"          }        },        {          "text": "按顺序识别出所有文字，不是小票上的文字不要出现"        }      ]    }  ]}
```

文字识别出来了，但离实际运用还有距离，接下来完善下工作流。

③完善功能

篇幅有限，接下来就简单介绍下思路。

这是一个行驶证识别工作流，可以实现同时上传多张图片，逐个识别并将指定数据写入飞书多维表格，如果图片≥2M，则先压缩再识别（因为图片太大传输慢），图中右侧两个灰色节点可忽略（不同的模型，用来对比识别效果的）

自动写入效果如下

A、上传模块

就是一个form trigger节点

将Multiple Files选上，就可以同时上传多张图片了，

选择图片的时候，按住ctrl键，点击鼠标左键可以多选

如果担心工作流被其他人使用，可以参考下图设置账号密码，上传图片时可验证

工作流正式激活后（点击上方的Inactive，激活后会显示Active），将生产url复制到浏览器打开并保存，后面用这个链接就可以了

B、循环处理模块

这里用到2个节点，code和loop，code节点将图片转换为数组，传递给loop循环节点，loop节点从数组里循环取出图片进行处理

创建loop节点后，先将replace me节点删除

然后将loop节点的loop输出口和工作流最末尾节点连起来

code节点的代码由ai生成（我这里用的是gemini），最开始就是告诉ai，我需要用n8n实现同时上传多张图片，然后循环识别图片，就会给出对应方案，ai就会给出详细代码，但gemini刚开始给出的方案会频繁报错，可以把报错截图发给gemini参考，最终经过几轮折腾才搞定。详细代码见文末工作流。

loop节点这里比较简单，Batch Size这里设置1就可以，意思是每次只处理1个文件

最主要是接线要对

C、分流及压缩合并模块

由3个节点组成，if，edit image和merge，if节点将≥2M的图片传递给edit image节点进行压缩，＜2M的图片则不处理，最终汇合到merge节点输出

if节点参考下图设置，2M字节就是2的21次方，等于2097152

edit image这里对图片进行压缩，选择好图片格式Format后会出现Quality，可以填75~85这样，填75的时候，6M图片可以压缩到2M，但图片清晰度会下降一些，强力的大模型可以识别准确，OCR模型则识别准确率下降很多。

merge节点这里输入数填2，代表有两个数据输入源

D、识别模块

这里就和本文②相同了

E、数据清洗及写入表格模块

由code和feishu node节点组成

由于不同模型，不同提示词（http节点的json处设置），都会导致输出结果不同，比如这种，有的文字之间有\n，有的文字之间是空格

还有这种，我在提示词处强调文字之间都要有换行符\n

所以code节点的代码也要跟着变，要尽可能用提示词确保http节点输出的文字有固定规律（比如要求不需要显示多余注释文字，不要显示英文，所有文字之间都需要有换行符等等），code节点才好处理

这里code节点代码均由gemini生成

如果发现实际输出结果不对，则截图让gemini改代码，这里再次强调，因为大模型输出的随机性，所以需要在http节点的提示词处做好限制，等输出固定了，code才好处理，不然大模型一会输出这种样式，一会输出那种样式，code节点需要加入非常多判断才能处理，还不一定能解决。

关于本工作流code代码，可以在文末获取。

最后是飞书节点（需要安装社区节点n8n-nodes-feishu-lite），关于权限配置，参考[让AI帮你打工！n8n将AI资讯自动写入到你的飞书表格](https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247485195&idx=1&sn=2ae9e47bdcb5c57401b080ef973f304c&scene=21#wechat_redirect)

配置好多维表格字段名和权限后，参考下图连接飞书并填入表格token和id

json写法参考如下

最后附上工作流链接，https://pan.quark.cn/s/f7fd8cb67417

好了，今天的内容就到这里啦，从上传图片到自动写入多维表格，全程自动化，只需要喝杯咖啡，坐等结果！觉得有帮助的话，点赞收藏关注，我们下期见！