---
title: AI+数据分析来了！Doris 让SQL直接调用DeepSeek、ChatGPT、Claude...
author: 一臻数据
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650488445&idx=1&sn=eddce927f9317611d47db8227e458ab5&chksm=f2b2f16eeb9fc832974321ba3c9d5ef2162035980b51c55a88dec94cea1d9adb0015c879face&mpshare=1&scene=24&srcid=0829eXuNvAUb6OdqSxDZPKK5&sharer_shareinfo=da2e0cd0dd582d7ea1aaab97d2033e2a&sharer_shareinfo_first=da2e0cd0dd582d7ea1aaab97d2033e2a#rd
---

见字如面，我是一臻 90后新手奶爸，探索Doris x AI

点击关注 👇 免费获取数字AI知识库

> ❝
>
> 昨晚十一点，老王正准备关电脑下班，突然收到数据产品经理小王的微信："老哥，明天老板要看用户评论的情感分析报告，咱们那几个小目标的Doris数据怎么办？找外包公司来不及了啊！" 
>
> 看着屏幕上密密麻麻的用户反馈数据，老王突然想起了`准备pre的Apache Doris 4.0版本`。
>
> " 别慌，"老王回复道，"可以找官方试试这个新玩意儿。" 打开电脑，敲下几行SQL代码，十分钟后，所有用户评论的情感分析结果就出来了。 
>
> 小王看到结果后直接发了个"`跪了`"的表情包：这就是 **Apache Doris 4.0 LLM函**数带来的魔力——让数据库直接拥有了AI的大脑？

## 当Doris SQL遇上大模型

说起来，数据分析师最头疼的事情是什么？

不是复杂的算法，不是海量的数据，而是那些看起来简单却让人抓狂的文本处理任务。

用户评论要分类、客服对话要提取关键信息、产品描述要做情感分析...以前这些活儿，要么写一堆复杂的正则表达式，要么调用各种第三方API，要么就是人工一条条处理。

现在不一样了。

Apache Doris 4.0直接把LLM的能力集成到了SQL函数里。你没听错，就是那些火遍全球的大语言模型技术（`DeepSeek、ChatGPT、Claude...`），现在可以直接在数据库里用了。

"这不就是个噱头吗？"老王的同事老李第一次听说这个功能时撇撇嘴，"数据库就好好做数据库的事，搞什么AI？"

直到上周他需要处理一批保险理赔数据，要从几亿条事故描述中筛选出赔付的案例。

以前这种活儿，他得写个`Python脚本，调用各种NLP库，折腾大半天`。

现在呢？`一条SQL搞定`：

```
SELECT  
    c.claim_id,  
    c.incident_description,  
    llm_classify(c.incident_description, ['交通事故', '人身意外', '财产损失', '其他']) AS incident_category  
FROM claims AS c  
JOIN policies AS p ON c.policy_id = p.policy_id  
WHERE  
    p.policy_type = '车险' AND LLM_FILTER(CONCAT('下列情形是否支持保险赔偿：', c.incident_description));
```

输出结果参考：

```
+----------+-----------------------------------------------------------------------------------------+-------------------+  
| claim_id | incident_description                                                                    | incident_category |  
+----------+-----------------------------------------------------------------------------------------+-------------------+  
|        1 | 昨天下午三点左右，我在东三环辅路开车时，与前车发生了追尾。                                        | 交通事故           |  
|        5 | 早上8点，在去上班的路上，发现车辆停放时被刮擦。                                                 | 财产损失           |  
+----------+-----------------
```

结果出来后，老李看着屏幕愣了半天，然后默默地把之前写的几百行Python代码删了。

"**真香**！"他说。

## 十个函数，解决90%的文本处理难题

Apache Doris 4.0 预测一口气推出了十个LLM函数，每一个都针对实际业务场景设计：

具体参考Doris官方文档：`https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/llm-functions/llm-function`

```
LLM_CLASSIFY: 信息分类  
LLM_EXTRACT: 提取信息  
LLM_FILTER：筛选信息  
LLM_FIXGRAMMAR： 修改病句  
LLM_GENERATE： 生成信息  
LLM_MASK： 掩盖敏感信息  
LLM_SENTIMENT： 情感分析  
LLM_SIMILARITY： 文本语义相似性比较  
LLM_SUMMARIZE： 文本摘要  
LLM_TRANSLATE： 翻译
```

LLM\_CLASSIFY负责分类，LLM\_EXTRACT提取信息，LLM\_SENTIMENT分析情感，LLM\_TRANSLATE做翻译...这些函数直接是给Doris装上了一个AI助手，专门处理那些让程序员头疼的文本任务。

最让老王印象深刻的是`LLM_SIMILARITY函数`。

上个月，老王合作的快递公司遇到了一个头疼问题：`N条用户评论堆在那里，客服主管想要快速找出那些真正不满意的客户，好及时处理投诉`。

传统的关键词搜索完全不靠谱——"服务态度不好"和"快递员很不耐烦"明明都是在抱怨服务，但系统就是识别不出来。

用了`LLM_SIMILARITY`后，这个问题瞬间解决：

```
-- 某家快递公司收到的评论表  
CREATETABLE user_comments (  
    id      INT,  
    commentVARCHAR(500)  
) DUPLICATEKEY(id)  
DISTRIBUTEDBYHASH(id) BUCKETS 10  
PROPERTIES (  
    "replication_num" = "1"  
);  
  
-- 按顾客语气情绪对评论进行排行时  
SELECTcomment,  
    LLM_SIMILARITY('resource_name', 'I am extremely dissatisfied with their service.', comment) AS score  
FROM user_comments ORDERBY score DESCLIMIT5;  
  
-- 查询结果大致如下：  
+-------------------------------------------------+-------+  
| comment                                         | score |  
+-------------------------------------------------+-------+  
| It arrived broken and I am really disappointed. |   7.5 |  
| Delivery was very slow and frustrating.         |   6.5 |  
| Not bad, but the packaging could be better.     |   3.5 |  
| It is fine, nothing special to mention.         |     3 |  
| Absolutely fantastic, highly recommend it.      |     1 |  
+-------------------------------------------------+-------+
```

他们设定一个标准的不满情绪模板"`I am extremely dissatisfied with their service`."，然后让系统计算每条评论与这个模板的语义相似度。

结果让人惊喜："It arrived broken and I am really disappointed."得分7.5，"Delivery was very slow and frustrating."得分6.5，而"Absolutely fantastic, highly recommend it."只有1分。

系统完美地理解了不同表达方式背后的相同情感。客服主管看到按相似度排序的结果后直接拍桌子："这比我们人工筛选还精准！现在我们可以优先处理那些真正愤怒的客户了。"

更有意思的是`LLM_MASK函数`。

近期，老王公司的法务部门急匆匆找到我："数据部的兄弟，咱们要向监管部门提交一批用户反馈数据，但里面有大量个人信息需要脱敏处理，你们能帮忙吗？"

他一看数据，头都大了——N条用户留言里，姓名、电话、邮箱、身份证号混杂在各种句子中，格式五花八门。以前遇到这种活儿，得写一堆正则表达式，还经常漏掉一些特殊格式。

现在有了LLM\_MASK函数，这个问题变得简单到让人不敢相信。只需先配置好AI资源，然后写了几行SQL：

```
-- name、age示例  
SET default_llm_resource = 'resource_name';  
SELECT LLM_MASK('Wccccat is a 20-year-old Doris community contributor.', ['name', 'age']) ASResult;  
  
+-----------------------------------------------------+  
| Result                                              |  
+-----------------------------------------------------+  
| [MASKED] is a [MASKED] Doris community contributor. |  
+-----------------------------------------------------+  
  
-- email、phone_num示例  
SELECT LLM_MASK('resource_name', 'My email is rarity@example.com and my phone is 123-456-7890',  
                ['email', 'phone_num']) ASRESULT  
       
+-----------------------------------------------+  
| RESULT                                        |  
+-----------------------------------------------+  
| My email is [MASKED] and my phone is [MASKED] |  
+-----------------------------------------------+      
```

离谱的是，连那些藏在句子中间、格式不规范的信息都逃不过它的"法眼"。

法务部门看到这些结果后直接竖起大拇指："这效率比我们人工检查还靠谱！而且格式这么标准，直接就能提交给监管部门了。"

## Doris让AI接入变得简单

Doris在设计这套LLM函数时，最nice的地方就是采用了**资源化管理**。

你可以把各种大模型服务——DeepSeek、ChatGPT、Claude等等——都配置成Doris的资源，然后在SQL中直接调用。

配置过程简单到让人怀疑：

```
-- 1、注册AI资源，deepseek-chat为例  
CREATERESOURCE'deepseek_example'  
PROPERTIES (      
    'type'='llm',      
    'llm.provider_type'='deepseek',      
    'llm.endpoint'='https://api.deepseek.com/chat/completions',   
    'llm.model_name' = 'deepseek-chat',      
    'llm.api_key' = 'xxxxx'  
);  
  
-- 2、设置默认资源  
SET default_llm_resource = 'deepseek_example';  
  
-- 3、直接在SQL中调用无需额外配置
```

就这么几行代码，你就把一个强大的AI模型接入到了数据库中。

想换个模型？改改配置就行。想同时用多个模型？创建多个资源。Doris会自动处理不同厂商API格式的差异，你完全不用操心底层的技术细节。

老王公司的数据师们对这个设计赞不绝口："`以前接入一个新的AI服务，要改代码、测试、部署，折腾好几天。现在几分钟就搞定，而且所有的API密钥都集中管理，安全性也有保障。`"

除此，Apache Doris 4.0的LLM函数让我重新思考了一个问题：**数据库的边界到底在哪里**？

传统观念里，数据库就是存储和查询数据的地方。计算逻辑放在应用层，AI处理放在专门的服务里，各司其职，井水不犯河水。

但现实业务中，数据处理往往是一个连续的流程——从`原始数据到清洗、转换、分析、再到最终的业务决策`。如果每个环节都要在不同的系统间传输数据，不仅效率低下，还容易出错。

Doris的做法是把AI能力直接下沉到数据层。数据在哪里，计算就在哪里，AI处理也在哪里。

这样做的好处是显而易见的：`减少数据传输、提高处理效率、简化系统架构`。更重要的是，它让数据分析师能够用熟悉的SQL语言来处理复杂的AI任务，大大降低了技术门槛。

当然，这种设计也带来了新的挑战。如何保证AI处理的稳定性？如何控制外部API调用的成本？如何处理不同模型的性能差异？Doris团队在这些方面都做了深入的思考和设计。

比如，内置的重试机制可以应对网络波动，资源配置可以灵活控制调用频率和成本，统一的接口屏蔽了不同模型的差异。

从某种意义上说，Apache Doris 4.0的LLM函数代表了数据库技术发展的一个新方向：`从单纯的数据存储和查询工具，向智能化演进`!

## 结语

回到文章开头的那个场景。当我们能用几行SQL就完成了N条用户评论的情感分析时，我意识到这不再是一个新功能的发布，而是数据处理方式的一次重塑。

在这个AI与数据深度融合的时代，Apache Doris正在重新定义数据库的可能性。而我们这些数据从业者，也正在见证和参与这场变革。

也许有一天，当我们回顾这段历史时，会发现Apache Doris 4.0的LLM函数是一个重要的转折点——**让AI真正走进了数据的世界，也让数据拥有了思考的能力**。

**完**

---

👇欢迎扫描下方二维码 👇 

备注 **666**免费领取资料  加入Doris官方群和PowerData数据社区❗️

#大数据 #开源 #OLAP #Doris #AI #人工智能